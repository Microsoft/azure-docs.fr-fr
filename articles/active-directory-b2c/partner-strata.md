---
title: Tutoriel pour configurer Azure Active Directory B2C avec Strata
titleSuffix: Azure AD B2C
description: Dans ce tutoriel, découvrez comment intégrer l’authentification Azure AD B2C avec whoIam pour la vérification de l’utilisateur
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/25/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: bddc4c64feb31f78bed482bbd729ab1c4b8e676e
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96171413"
---
# <a name="tutorial-for-extending-azure-ad-b2c-to-protect-on-premises-applications-using-strata"></a>Tutoriel sur l’extension d’Azure AD B2C afin de protéger des applications locales à l’aide de Strata

Ce tutoriel explique comment intégrer Azure Active Directory (AD) B2C avec le connecteur [Maverics Identity Orchestrator](https://www.strata.io/maverics-identity-orchestrator/) de Strata.
Le connecteur Maverics Identity Orchestrator étend Azure AD B2C afin de protéger des applications locales. Il se connecte à n’importe quel système d’identité, migre en toute transparence les utilisateurs et les informations d’identification, synchronise les stratégies et les configurations, et réduit la charge d’authentification et de gestion des sessions. Strata permet aux entreprises de passer rapidement d’une configuration héritée à Azure AD B2C sans réécrire d’applications. La solution offre les avantages suivants :

- **Authentification unique (SSO) du client auprès d’applications hybrides locales** : Azure AD B2C prend en charge l’authentification unique du client avec Maverics Identity Orchestrator. Les utilisateurs se connectent avec leurs comptes hébergés dans Azure AD B2C ou le fournisseur d’identité sociale (IdP). Maverics étend l’authentification unique aux applications qui étaient précédemment sécurisées par des systèmes d’identité hérités tels que Symantec SiteMinder.

- **Extension de l’authentification unique basée sur des normes aux applications sans réécriture de celles-ci** : Utilisez Azure AD B2C pour gérer l’accès des utilisateurs et activer l’authentification unique avec des connecteurs OIDC ou SAML Maverics Identity Orchestrator.

- **Facilité de configuration** : Azure AD B2C fournit une interface utilisateur pas-à-pas simple pour connecter des connecteurs OIDC ou SAML Maverics Identity Orchestrator à Azure AD B2C.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez des éléments suivants :

- Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).

- Un [locataire Azure AD B2C](./tutorial-create-tenant.md) lié à votre abonnement Azure.

- Une instance d’[Azure Key Vault](https://azure.microsoft.com/services/key-vault/) pour stocker les secrets qu’utilise Maverics Identity Orchestrator. Celle-ci est utilisée pour se connecter à Azure AD B2C ou à d’autres fournisseurs d’attributs tels qu’un annuaire ou une base de données LDAP (Lightweight Directory Access Protocol).

- Une instance de [Maverics Identity Orchestrator](https://www.strata.io/maverics-identity-orchestrator/) installée et en cours d’exécution sur une machine virtuelle Azure ou sur le serveur local de votre choix. Pour plus d’informations sur la façon d’obtenir le logiciel et d’accéder à la documentation relative à l’installation et à la configuration, contactez [Strata](https://www.strata.io/contact/).

- Une application locale que vous allez faire passer d’un système d’identité hérité à Azure AD B2C.

## <a name="scenario-description"></a>Description du scénario

L’intégration de la solution Maverics de Strata inclut les composants suivants :

- **Azure AD B2C** : serveur d’autorisation chargé de vérifier les informations d’identification de l’utilisateur. Des utilisateurs authentifiés peuvent accéder aux applications locales à l’aide d’un compte local stocké dans l’annuaire Azure AD B2C.

- **Fournisseur externe d’identité sociale ou d’entreprise** : peut être n’importe quel fournisseur OpenID Connect, Facebook, Google ou GitHub. Consultez les informations sur l’utilisation de [fournisseurs d’identité externes](./technical-overview.md#external-identity-providers) avec Azure AD B2C.  

- **Maverics Identity Orchestrator de Strata** : le service qui orchestre l’authentification de l’utilisateur et transmet en toute transparence l’identité aux applications via des en-têtes HTTP.

Le diagramme d’architecture suivant illustre l’implémentation.

![Image montrant l’architecture d’une intégration Azure AD B2C avec Strata Maverics pour permettre l’accès à des applications hybrides](./media/partner-strata/strata-architecture-diagram.png)

| Étapes | Description |
|:-------|:---------------|
| 1. | L’utilisateur effectue une demande d’accès à l’application hébergée localement. Maverics Identity Orchestrator transmet la demande de l’utilisateur à l’application.|
| 2. | L’application Orchestrator vérifie l’état d’authentification de l’utilisateur. S’il ne reçoit pas de jeton de session ou si le jeton de session fourni n’est pas valide, il achemine l’utilisateur vers Azure AD B2C à des fins d’authentification.|
| 3. | Azure AD B2C envoie la demande d’authentification au fournisseur d’identité sociale configuré.|
| 4. | Le fournisseur d’identité demande à l’utilisateur ses informations d’identification. Selon le fournisseur d’identité, l’utilisateur peut être tenu d’effectuer une authentification multifacteur (MFA).|
| 5. | Le fournisseur d’identité renvoie la réponse d’authentification à Azure AD B2C. Si vous le souhaitez, l’utilisateur peut créer un compte local dans l’annuaire Azure AD B2C au cours de cette étape.|
| 6. | Azure AD B2C envoie la demande de l’utilisateur au point de terminaison spécifié lors de l’inscription de l’application Orchestrator dans le locataire Azure AD B2C.|
| 7. | L’application Orchestrator évalue les stratégies d’accès et calcule les valeurs d’attribut à inclure dans les en-têtes HTTP transférés à l’application. Au cours de cette étape, l’application Orchestrator peut appeler des fournisseurs d’attributs supplémentaires afin de récupérer les informations nécessaires pour définir correctement les valeurs d’en-tête. L’application Orchestrator définit les valeurs d’en-tête et envoie la demande à l’application.|
| 8. | L’utilisateur est maintenant authentifié et a accès à l’application.|

## <a name="get-maverics-identity-orchestrator"></a>Obtenir Maverics Identity Orchestrator
Pour obtenir le logiciel que vous allez utiliser pour intégrer votre application locale héritée avec Azure AD B2C, contactez [Strata](https://www.strata.io/contact/). Une fois que vous avez obtenu le logiciel, suivez les étapes ci-dessous pour déterminer les conditions préalables spécifiques de l’application Orchestrator, et exécutez les étapes d’installation et de configuration requises.

## <a name="configure-your-azure-ad-b2c-tenant"></a>Configurer votre locataire Azure AD B2C

1. **Inscrivez votre application**

   a. [Inscrivez l’application Orchestrator en tant qu’application](./tutorial-register-applications.md?tabs=app-reg-ga) dans le locataire Azure AD B2C.
   >[!Note]
   >Vous aurez besoin du nom et de l’identificateur du locataire, de l’ID client, de la clé secrète client, des revendications configurées et de l’URI de redirection ultérieurement pour configurer votre instance Orchestrator.

   b. Accordez des autorisations d’API MS Graph à vos applications. Votre application aura besoin des autorisations suivantes : `offline_access`, `openid`.

   c. Ajoutez un URI de redirection pour votre application. Cet URI correspond au paramètre `oauthRedirectURL` de la configuration du connecteur Azure AD B2C de votre application Orchestrator, par exemple `https://example.com/oidc-endpoint`.

2. **Créez un flux utilisateur** : créez un [flux utilisateur d’inscription et de connexion](./tutorial-create-user-flows.md).

3. **Ajoutez un fournisseur d’identité** : choisissez de connecter votre utilisateur avec un compte local ou un [fournisseur d’identité](./tutorial-add-identity-providers.md) sociale ou d’entreprise.

4. **Définissez les attributs utilisateur** : définissez les attributs à collecter lors de l’inscription.

5. **Spécifiez les revendications d’application** : spécifiez les attributs à retourner à l’application via votre instance Orchestrator. L’application Orchestrator utilise des attributs extraits de revendications retournées par Azure AD B2C, et peut récupérer des attributs supplémentaires à partir d’autres systèmes d’identité connectés, tels que des annuaires et bases de données LDAP. Ces attributs sont définis dans des en-têtes HTTP et envoyés à l’application locale en amont.

## <a name="configure-maverics-identity-orchestrator"></a>Configurer Maverics Identity Orchestrator

Dans les sections suivantes, nous allons vous guider tout au long des étapes nécessaires à la configuration de votre instance Orchestrator. Pour obtenir de l’aide et de la documentation supplémentaires, contactez [Strata](https://www.strata.io/contact/).

### <a name="maverics-identity-orchestrator-server-requirements"></a>Configuration requise pour le serveur Maverics Identity Orchestrator

Vous pouvez exécuter votre instance Orchestrator sur n’importe quel serveur, qu’il soit local ou dans une infrastructure de cloud public d’un fournisseur tel que Azure, AWS ou GCP.

- Système d’exploitation : REHL 7.7 ou version ultérieure, CentOS 7+

- Disque : 10 Go (petit)

- Mémoire : 16 Go

- Ports : 22 (SSH/SCP), 443, 80

- Accès racine pour les tâches d’installation/d’administration

- Maverics Identity Orchestrator s’exécute en tant qu’utilisateur `maverics` sous `systemd`

- Sortie réseau à partir du serveur hébergeant Maverics Identity Orchestrator avec la possibilité d’atteindre votre locataire Azure AD.

### <a name="install-maverics-identity-orchestrator"></a>Installer Maverics Identity Orchestrator

1. Obtenez le package RPM Maverics le plus récent. Placez le package sur le système sur lequel vous souhaitez installer Maverics. Si vous copiez le fichier sur un hôte distant, [SCP](https://www.ssh.com/ssh/scp/) est un outil précieux.

2. Pour installer le package Maverics, exécutez la commande suivante en remplaçant `maverics.rpm` par votre nom de fichier.

   `sudo rpm -Uvf maverics.rpm`

   Par défaut, Maverics est installé dans le répertoire `/usr/local/bin`.

3. Une fois Maverics installé, il s’exécute en tant que service sous `systemd`.  Pour vérifier que le service Maverics est en cours d’exécution, exécutez la commande suivante :

   `sudo service maverics status`

  Si l’installation de l’application Orchestrator s’est déroulée correctement, un message similaire à celui-ci doit s’afficher :

```
Redirecting to /bin/systemctl status maverics.service
  maverics.service - Maverics
  Loaded: loaded (/etc/systemd/system/maverics.service; enabled; vendor preset: disabled)
  Active: active (running) since Thu 2020-08-13 16:48:01 UTC; 24h ago
  Main PID: 330772 (maverics)
  Tasks: 5 (limit: 11389)
  Memory: 14.0M
  CGroup: /system.slice/maverics.service
          └─330772 /usr/local/bin/maverics --config /etc/maverics/maverics.yaml
  ```

4. Si le service Maverics ne parvient pas à démarrer, exécutez la commande suivante pour examiner le problème :

   `journalctl --unit=maverics.service --reverse`

   L’entrée de journal la plus récente apparaît au début de la sortie.

Une fois Maverics installé, le fichier `maverics.yaml` par défaut est créé dans le répertoire `/etc/maverics`.

Configurez votre Orchestrator pour protéger l’application. Procédez à l’intégration avec Azure AD B2C, stockez et récupérez des secrets à partir d’[Azure Key Vault](https://azure.microsoft.com/services/key-vault/?OCID=AID2100131_SEM_bf7bdd52c7b91367064882c1ce4d83a9:G:s&ef_id=bf7bdd52c7b91367064882c1ce4d83a9:G:s&msclkid=bf7bdd52c7b91367064882c1ce4d83a9). Définissez l’emplacement à partir duquel l’application Orchestrator doit lire sa configuration.

### <a name="supply-configuration-using-environment-variables"></a>Configuration de l’approvisionnement à l’aide de variables d’environnement

Fournissez une configuration à vos instances Orchestrator via des variables d’environnement.

`MAVERICS_CONFIG`

Cette variable d’environnement indique à l’instance Orchestrator les fichiers de configuration YAML à utiliser et l’emplacement où les trouver au démarrage ou au redémarrage. Définissez la variable d’environnement dans `/etc/maverics/maverics.env`.

### <a name="create-the-orchestrators-tls-configuration"></a>Créer la configuration TLS de l’application Orchestrator

Le champ `tls` de votre fichier `maverics.yaml` déclare les configurations de sécurité de la couche de transport que votre instance Orchestrator utilisera. Les connecteurs peuvent utiliser des objets TLS et le serveur Orchestrator.

La clé `maverics` est réservée au serveur Orchestrator. Toutes les autres clés sont disponibles et peuvent être utilisées pour injecter un objet TLS dans un connecteur donné.

```yaml
tls:
  maverics:
    certFile: /etc/maverics/maverics.cert
    keyFile: /etc/maverics/maverics.key
```

### <a name="configure-the-azure-ad-b2c-connector"></a>Configurer le connecteur Azure AD B2C

Les orchestrateurs utilisent des connecteurs pour s’intégrer avec les fournisseurs d’authentification et d’attributs. Dans ce cas, cette App Gateway d’Orchestrator utilise le connecteur Azure AD B2C comme fournisseur d’authentification et d’attributs. Azure AD B2C utilise le fournisseur d’identité sociale pour l’authentification, puis agit en tant que fournisseur d’attributs pour l’application Orchestrator, en passant des attributs dans des revendications définies dans des en-têtes HTTP.  

La configuration de ce connecteur correspond à l’application inscrite dans le locataire Azure AD B2C.

1. Copiez l’ID client, le secret et l’URI de redirection de la configuration de votre application vers votre locataire.

2. Attribuez un nom à votre connecteur, indiqué ici comme `azureADB2C`, et définissez le connecteur `type` sur `azure`. Notez le nom du connecteur, car cette valeur est utilisée dans d’autres paramètres de configuration ci-dessous.

3. Pour cette intégration, la valeur `authType` doit être définie sur `oidc`.

4. Définissez l’ID client que vous avez copié à l’étape 1 comme valeur du paramètre `oauthClientID`.

5. Définissez la clé secrète client que vous avez copiée à l’étape 1 comme valeur du paramètre `oauthClientSecret`.

6. Définissez l’URI de redirection que vous avez copié à l’étape 1 comme valeur du paramètre `oauthRedirectURL` .

7. Le connecteur OIDC Azure AD B2C utilise le point de terminaison OIDC bien connu pour découvrir les métadonnées, y compris les URL et les clés de signature. Définissez la valeur de `oidcWellKnownURL` sur le point de terminaison de votre locataire.

```yaml
connectors:
  name: azureADB2C
  type: azure
  oidcWellKnownURL: https://<tenant name>.b2clogin.com/<tenant name>.onmicrosoft.com/B2C_1_login/v2.0/.well-known/openid-configuration
  oauthRedirectURL: https://example.com/oidc-endpoint
  oauthClientID: <azureADB2CClientID>
  oauthClientSecret: <azureADB2CClientSecret>
  authType: oidc
```

### <a name="define-azure-ad-b2c-as-your-authentication-provider"></a>Définir Azure AD B2C comme fournisseur d’authentification

Un fournisseur d’authentification détermine comment effectuer l’authentification pour un utilisateur qui n’a pas présenté de session valide dans le cadre de la demande de ressource d’application. La configuration dans votre locataire Azure AD B2C détermine comment demander à un utilisateur ses informations d’identification et appliquer des stratégies d’authentification supplémentaires. Par exemple, pour exiger un deuxième facteur afin de terminer le processus d’authentification, et décider des revendications à retourner à l’App Gateway d’Orchestrator une fois l’authentification réussie.

La valeur de `authProvider` doit correspondre à la valeur `name` de votre connecteur.

```yaml
authProvider: azureADB2C
```

### <a name="protect-your-on-premises-app-with-an-orchestrator-app-gateway"></a>Protéger votre application locale avec une App Gateway d’Orchestrator

La configuration d’App Gateway de l’Orchestrator déclare comment Azure AD B2C doit protéger votre application, et comment les utilisateurs doivent accéder à l’application.

1. Créez un nom pour votre App Gateway. Vous pouvez utiliser un nom convivial ou un nom d’hôte complet comme identificateur pour votre application.

2. Définissez `location`. L’exemple ci-dessous utilise la racine de l’application `/`, mais il peut s’agir de n’importe quel chemin d’accès d’URL de votre application.

3. Définissez l’application protégée dans `upstream` à l’aide de la convention host:port : `https://example.com:8080`.

4. Définissez les valeurs des pages d’erreur et de non-autorisation.

5. Définissez les noms d’en-tête HTTP et les valeurs d’attribut qui doivent être fournis à l’application pour établir l’authentification et contrôler l’accès à l’application. Les noms d’en-tête sont arbitraires et correspondent généralement à la configuration de l’application. Les valeurs d’attribut sont associées à des espaces de noms par le connecteur qui les fournit. Dans l’exemple ci-dessous, les valeurs retournées à partir d’Azure AD B2C sont précédées du nom du connecteur `azureADB2C`, où le suffixe est le nom de l’attribut contenant la valeur requise, par exemple `given_name`.

6. Définissez les stratégies à évaluer et à appliquer. Trois actions sont définies : `allowUnauthenticated`, `allowAnyAuthenticated`et `allowIfAny`. Chaque action est associée à une `resource` et la stratégie est évaluée pour cette `resource`.

>[!NOTE]
>`headers` et `policies` utilisent des extensions de service JavaScript ou GoLang pour implémenter une logique arbitraire qui améliore considérablement les fonctionnalités par défaut.

```yaml
appgateways:
  - name: Sonar
    location: /
    upstream: https://example.com:8080
    errorPage: https://example.com:8080/sonar/error
    unauthorizedPage: https://example.com:8080/sonar/accessdenied

    headers:
      SM_USER: azureADB2C.sub
      firstname: azureADB2C.given_name
      lastname: azureADB2C.family_name

    policies:
      - resource: ~ \.(jpg|png|ico|svg)
        allowUnauthenticated: true
      - resource: /
        allowAnyAuthenticated: true
      - resource: /sonar/daily_deals
        allowIfAny:
          azureADB2C.customAttribute: Rewards Member
```

### <a name="use-azure-key-vault-as-your-secrets-provider"></a>Utiliser Azure Key Vault comme fournisseur de secrets

Il est important de sécuriser les secrets que votre application Orchestrator utilise pour se connecter à Azure AD B2C et à tout autre système d’identité. Par défaut, Maverics charge les secrets en texte brut en dehors de `maverics.yaml` mais, dans ce tutoriel, vous allez utiliser Azure Key Vault comme fournisseur de secrets.

Suivez les instructions pour [créer un Key Vault](../key-vault/secrets/quick-create-portal.md) que votre instance Orchestrator utilisera comme fournisseur de secrets. Ajoutez vos secrets à votre coffre et prenez note du `SECRET NAME` donné à chaque secret. Par exemple, `AzureADB2CClientSecret`.

Pour déclarer une valeur en tant que secret dans un fichier de configuration `maverics.yaml`, placez le secret entre crochets angulaires :

```yaml
connectors:
  - name: AzureADB2C
    type: azure
    oauthClientID: <AzureADB2CClientID>
    oauthClientSecret: <AzureADB2CClientSecret>
```

La valeur spécifiée entre crochets angulaires doit correspondre au `SECRET NAME` donnée au secret dans votre Azure Key Vault.

Pour charger des secrets à partir d’Azure Key Vault, définissez la variable d’environnement `MAVERICS_SECRET_PROVIDER` dans le fichier `/etc/maverics/maverics.env`, avec les informations d’identification se trouvant dans le fichier azure-credentials.json, en utilisant le modèle suivant :

`MAVERICS_SECRET_PROVIDER='azurekeyvault://<KEYVAULT NAME>.vault.azure.net?clientID=<APPID>&clientSecret=<PASSWORD>&tenantID=<TENANT>'`

### <a name="put-everything-together"></a>Regrouper tous les éléments

Voici comment la configuration de l’application Orchestrator s’affiche lorsque vous terminez les configurations décrites ci-dessus.

```yaml
version: 0.4.2
listenAddress: ":443"
tls:
  maverics:
    certFile: certs/maverics.crt
    keyFile: certs/maverics.key

authProvider: azureADB2C

connectors:
  - name: azureADB2C
    type: azure
    oidcWellKnownURL: https://<tenant name>.b2clogin.com/<tenant name>.onmicrosoft.com/B2C_1_login/v2.0/.well-known/openid-configuration
    oauthRedirectURL: https://example.com/oidc-endpoint
    oauthClientID: <azureADB2CClientID>
    oauthClientSecret: <azureADB2CClientSecret>
    authType: oidc

appgateways:
  - name: Sonar
    location: /
    upstream: http://example.com:8080
    errorPage: http://example.com:8080/sonar/accessdenied
    unauthorizedPage: http://example.com:8080/sonar/accessdenied

    headers:
      SM_USER: azureADB2C.sub
      firstname: azureADB2C.given_name
      lastname: azureADB2C.family_name

    policies:
      - resource: ~ \.(jpg|png|ico|svg)
        allowUnauthenticated: true
      - resource: /
        allowAnyAuthenticated: true
      - resource: /sonar/daily_deals
        allowIfAny:
          azureADB2C.customAttribute: Rewards Member
```

## <a name="test-the-flow"></a>Tester le flux

1. Accédez à l’URL de l’application locale, `https://example.com/sonar/dashboard`.

2. L’application Orchestrator doit effectuer une redirection vers la page que vous avez configurée dans votre flux utilisateur.

3. Sélectionnez le fournisseur d’identité dans la liste affichée sur la page.

4. Une fois que vous êtes redirigé vers le fournisseur d’identité, entrez les informations d’identification demandées, y compris un jeton MFA, le cas échéant.

5. Une fois l’authentification réussie, vous devez être redirigé vers Azure AD B2C, qui transfère la demande d’application à l’URI de redirection de l’application Orchestrator.

6. L’application Orchestrator évalue les stratégies, calcule les en-têtes et envoie l’utilisateur à l’application en amont.  

7. Vous devez voir l’application demandée.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Stratégies personnalisées dans Azure AD B2C](./custom-policy-overview.md)

- [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](./custom-policy-get-started.md?tabs=applications)