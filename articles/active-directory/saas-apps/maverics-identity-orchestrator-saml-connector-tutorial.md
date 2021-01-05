---
title: 'Tutoriel : Intégrer l’authentification unique Azure Active Directory à Maverics Identity Orchestrator SAML Connector | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et Maverics Identity Orchestrator SAML Connector.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/12/2020
ms.author: jeedes
ms.openlocfilehash: a4bfe2b87f3f2242189a78d9a31a89d82720fd37
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96862069"
---
# <a name="tutorial-integrate-azure-ad-single-sign-on-with-maverics-identity-orchestrator-saml-connector"></a>Tutoriel : Intégrer l’authentification unique Azure AD avec Maverics Identity Orchestrator SAML Connector

Strata offre un moyen simple d’intégrer des applications locales à Azure Active Directory (Azure AD) pour l’authentification et le contrôle d’accès.

Cet article vous montre comment configurer Maverics Identity Orchestrator pour :
* Migrer de manière incrémentielle les utilisateurs d’un système d’identité local vers Azure AD lors de la connexion à une application locale héritée
* Router les demandes de connexion à partir d’un produit de gestion de l’accès web hérité, tel que CA SiteMinder ou Oracle Access Manager, vers Azure AD.
* Authentifier l’utilisateur auprès des applications locales qui sont protégées à l’aide d’en-têtes HTTP ou de cookies de session propriétaires après l’authentification de l’utilisateur sur Azure AD.

Strata fournit un logiciel que vous pouvez déployer localement ou dans le cloud. Il sert à des fins de découverte, de connexion et d’orchestration des fournisseurs d’identité afin de créer une gestion distribuée des identités pour les entreprises hybrides et multiclouds.

Ce tutoriel montre comment migrer une application web locale protégée par un produit de gestion de l’accès web hérité (SiteMinder) afin d’utiliser Azure AD pour l’authentification et le contrôle d’accès. Les étapes de base sont les suivantes :
1. Installez Maverics Identity Orchestrator.
2. Inscrire votre application d’entreprise auprès d’Azure AD et la configurer afin qu’elle utilise Maverics Azure AD SAML Zero Code Connector pour l’authentification unique SAML.
3. Intégrez Maverics à SiteMinder et au magasin d’utilisateurs LDAP (Lightweight Directory Access Protocol).
4. Configurer un coffre de clés Azure et configurer Maverics pour l’utiliser en tant que fournisseur de gestion des secrets.
5. Illustrer la migration d’utilisateurs et l’abstraction de session à l’aide de Maverics pour fournir l’accès à une application web Java locale.

Pour obtenir des instructions d’installation et de configuration supplémentaires, consultez le [Site web de Strata](https://www.strata.io).

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
- Un abonnement Maverics Identity Orchestrator SAML Connector pour lequel l’authentification unique est activée. Pour obtenir le logiciel Maverics, contactez l’[équipe commerciale Strata](mailto:sales@strata.io).

## <a name="install-maverics-identity-orchestrator"></a>Installer Maverics Identity Orchestrator

Pour vous familiariser avec l’installation de Maverics Identity Orchestrator, reportez-vous aux [instructions d’installation](https://www.strata.io).

### <a name="system-requirements"></a>Configuration système requise
* Systèmes d’exploitation pris en charge
  * RHEL 7+
  * CentOS 7+

* Les dépendances
  * systemd

### <a name="installation"></a>Installation

1. Obtenez le dernier package du gestionnaire de package Maverics RedHat (RPM). Copiez le package sur le système sur lequel vous souhaitez installer le logiciel Maverics.

2. Installez le package Maverics, en substituant le nom de votre fichier à `maverics.rpm`.

    `sudo rpm -Uvf maverics.rpm`

3. Une fois Maverics installé, il s’exécute en tant que service sous `systemd`. Pour vérifier que le service est en cours d’exécution, exécutez la commande suivante :

    `sudo systemctl status maverics`

Par défaut, Maverics est installé dans le répertoire */usr/local/bin*.

Après l’installation de Maverics, le fichier *maverics.yaml* par défaut est créé dans le répertoire */etc/maverics*. Avant que vous ne modifiiez votre configuration pour inclure des `workflows` et des `connectors`, votre fichier de configuration se présente comme suit :

```yaml
# © Strata Identity Inc. 2020. All Rights Reserved. Patents Pending.

version: 0.1
listenAddress: ":7474"
```
## <a name="configuration-options"></a>Options de configuration
### <a name="version"></a>Version
Le champ `version` déclare la version du fichier de configuration qui est utilisée. Si la version n’est pas spécifiée, la version de configuration la plus récente sera utilisée.

```yaml
version: 0.1
```
### <a name="listenaddress"></a>listenAddress
`listenAddress` déclare l’adresse qu’Orchestrator doit écouter. Si la section hôte de l’adresse est vide, Orchestrator doit écouter toutes les adresses IP de monodiffusion et anycast disponibles du système local. Si la section du port de l’adresse est vide, un numéro de port est automatiquement choisi.

```yaml
listenAddress: ":453"
```
### <a name="tls"></a>TLS

Le champ `tls` déclare un mappage des objets Transport Layer Security (TLS). Les objets TLS peuvent être utilisés par les connecteurs ainsi que par le serveur Orchestrator. Pour toutes les options TLS disponibles, consultez la documentation du package de `transport`.

Microsoft Azure nécessite une communication sur TLS lorsque vous utilisez l’authentification unique basée sur SAML. Pour plus d’informations sur la génération de certificats, consultez [Chiffrons un site web](https://letsencrypt.org/getting-started/).

La clé `maverics` est réservée au serveur Orchestrator. Toutes les autres clés sont disponibles et peuvent être utilisées pour injecter un objet TLS dans un connecteur donné.

```yaml
tls:
  maverics:
    certFile: /etc/maverics/maverics.cert
    keyFile: /etc/maverics/maverics.key
```  
### <a name="include-files"></a>Fichiers Include

Vous pouvez définir `connectors` et `workflows` dans leurs propres fichiers de configuration et les référencer dans le fichier de *maverics.yaml* à l’aide de `includeFiles`, conformément à l’exemple suivant :

```yaml
includeFiles:
  - workflow/sessionAbstraction.yaml
  - connector/AzureAD-saml.yaml
  - connector/siteminder.yaml
  ```

Ce didacticiel utilise un seul fichier de configuration *maverics.yaml*.

## <a name="use-azure-key-vault-as-your-secrets-provider"></a>Utiliser Azure Key Vault comme fournisseur de secrets

### <a name="manage-secrets"></a>Gérer les secrets

Pour charger les secrets, Maverics peut s’intégrer à différentes solutions de gestion des secrets. Les intégrations actuelles incluent un fichier, Hashicorp Vault et Azure Key Vault. Si aucune solution de gestion des secrets n’est spécifiée, Maverics charge par défaut les secrets en texte brut à partir de *maverics.yaml*.

Pour déclarer une valeur comme une clé secrète dans un fichier de configuration *maverics.yaml*, placez le secret entre crochets angulaires :

  ```yaml
  connectors:
  - name: AzureAD
    type: AzureAD
    apiToken: <AzureADAPIToken>
    oauthClientID: <AzureADOAuthClientID>
    oauthClientSecret: <AzureADOAuthClientSecret>
  ```

### <a name="load-secrets-from-a-file"></a>Charger des secrets à partir d’un fichier

1. Pour charger des secrets à partir d’un fichier, ajoutez la variable d’environnement `MAVERICS_SECRET_PROVIDER` dans le fichier */etc/maverics/maverics.env* à l’aide de :

   `MAVERICS_SECRET_PROVIDER=secretfile:///<PATH TO SECRETS FILE>`

2. Redémarrez le service Maverics en exécutant :

   `sudo systemctl restart maverics`

Le contenu du fichier *secrets.yaml* peut être rempli avec un nombre quelconque de `secrets`.

```yaml
secrets:
  AzureADAPIToken: aReallyGoodToken
  AzureADOAuthClientID: aReallyUniqueID
  AzureADOAuthClientSecret: aReallyGoodSecret
```
### <a name="set-up-an-azure-key-vault"></a>Configurer un coffre de clés Azure

Vous pouvez configurer un coffre de clés Azure à l’aide du portail Azure ou d’Azure CLI.

**Utiliser le portail Azure**
1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. [Créez un coffre de clés](../../key-vault/general/quick-create-portal.md).
1. [Ajoutez les secrets au coffre de clés](../../key-vault/secrets/quick-create-portal.md#add-a-secret-to-key-vault).
1. [Inscrivez une application auprès d’Azure AD](../develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal).
1. [Autoriser une application à utiliser un secret](../../key-vault/secrets/quick-create-portal.md#add-a-secret-to-key-vault).

**Utilisation de l’interface de ligne de commande Microsoft Azure**

1. Ouvrez [Azure CLI](/cli/azure/install-azure-cli), puis entrez la commande suivante :

    ```shell
    az login
    ```

1. Créez un coffre de clés à l’aide de la commande suivante :
    ```shell
    az keyvault create --name "[VAULT_NAME]" --resource-group "[RESOURCE_GROUP]" --location "[REGION]"
    ```

1. Ajoutez les secrets au coffre de clés en exécutant la commande suivante :
    ```shell
    az keyvault secret set --vault-name "[VAULT_NAME]" --name "[SECRET_NAME]" --value "[SECRET_VALUE]"
    ```

1. Inscrivez une application avec Azure AD en exécutant la commande suivante :
    ```shell
    az ad sp create-for-rbac -n "MavericsKeyVault" --skip-assignment > azure-credentials.json
    ```

1. Autorisez une application à utiliser un secret en exécutant la commande suivante :
    ```shell
    az keyvault set-policy --name "[VAULT_NAME]" --spn [APPID] --secret-permissions list get
    #APPID can be found in the azure-credentials.json
    generated in the previous step
    ```

1. Pour charger des secrets à partir de votre coffre de clés Azure, définissez la variable d’environnement `MAVERICS_SECRET_PROVIDER` dans le fichier */etc/maverics/maverics.env* à l’aide des informations d’identification trouvées dans le fichier *azure-credentials.json*, au format suivant :
 
   `MAVERICS_SECRET_PROVIDER='azurekeyvault://<KEYVAULT NAME>.vault.azure.net?clientID=<APPID>&clientSecret=<PASSWORD>&tenantID=<TENANT>'`

1. Redémarrez le service Maverics : `sudo systemctl restart maverics`

## <a name="configure-your-application-in-azure-ad-for-saml-based-sso"></a>Configurer votre application dans Azure AD pour l’authentification unique SAML

1. Dans votre locataire Azure AD, accédez à **Applications d’entreprise**, recherchez **Maverics Identity Orchestrator SAML Connector**, puis sélectionnez-le.

1. Dans la page « Maverics Identity Orchestrator SAML Connector », volet **Propriétés**, définissez **Assignation requise de l’utilisateur ?** sur **Non** afin que l’application fonctionne pour les utilisateurs récemment migrés.

1. Dans le volet **Vue d’ensemble** d’Identity Orchestrator SAML Connector, sélectionnez **Configurer l’authentification unique**, puis **SAML**.

1. Dans Maverics Identity Orchestrator SAML Connector, accédez au volet **Authentification basée sur SAML** et modifiez la **Configuration SAML de base** en sélectionnant le bouton **Modifier** (icône de crayon).

   ![Capture d’écran du bouton Modifier la configuration SAML de base.](common/edit-urls.png)

1. Entrez l’**ID d’entité** en saisissant une URL au format suivant : `https://<SUBDOMAIN>.maverics.org`. L’ID d’entité doit être unique dans les applications du locataire. Enregistrez la valeur entrée ici pour l’inclure dans la configuration de Maverics.

1. Entrez l’**URL de réponse** au format suivant : `https://<AZURECOMPANY.COM>/<MY_APP>/`. 

1. Entrez l’**URL de connexion**, au format suivant : `https://<AZURE-COMPANY.COM>/<MY_APP>/<LOGIN PAGE>`. 

1. Sélectionnez **Enregistrer**.

1. Dans la section **Certificat de signature SAML**, cliquez sur le bouton **Copier** pour copier l’**URL des métadonnées de fédération d’application**, puis enregistrez-la sur votre ordinateur.

    ![Capture d’écran du bouton de copie Certificat de signature SAML.](common/copy-metadataurl.png)

## <a name="configure-maverics-identity-orchestrator-azure-ad-saml-connector"></a>Configurer les informations de Maverics Identity Orchestrator Azure AD SAML Connector

Maverics Identity Orchestrator Azure AD Connector prend en charge OpenID Connect et SAML Connect. Pour configurer le connecteur, procédez comme suit : 

1. Pour activer l’authentification unique SAML, définissez `authType: saml`.

1. Créez la valeur pour `samlMetadataURL` au format suivant : `samlMetadataURL:https://login.microsoftonline.com/<TENANT ID>/federationmetadata/2007-06/federationmetadata.xml?appid=<APP ID>`.

1. Définissez l’URL vers laquelle doit être redirigé l’utilisateur dans votre application une fois qu’il s’est connecté avec ses informations d’identification Azure. Utilisez le format suivant : `samlRedirectURL: https://<AZURECOMPANY.COM>/<MY_APP>`.

1. Copiez la valeur à partir de l’EntityID précédemment configuré : `samlEntityID: https://<SUBDOMAIN>.maverics.org`.

1. Copiez la valeur de l’URL de réponse qu’Azure AD doit utiliser pour envoyer la réponse SAML en mode POST : `samlConsumerServiceURL: https://<AZURE-COMPANY.COM>/<MY_APP>`.

1. Générez une clé de signature JSON Web Token (JWT), qui est utilisée pour protéger les informations de session Maverics Identity Orchestrator, à l’aide de l’[outil OpenSSL](https://www.openssl.org/source/) :

    ```shell 
    openssl rand 64 | base64
    ```
1. Copiez la réponse dans la propriété de configuration `jwtSigningKey` : `jwtSigningKey: TBHPvTtu6NUqU84H3Q45grcv9WDJLHgTioqRhB8QGiVzghKlu1mHgP1QHVTAZZjzLlTBmQwgsSoWxGHRcT4Bcw==`.

## <a name="attributes-and-attribute-mapping"></a>Attributs et mappage d’attributs
Le mappage d’attributs sert à définir le mappage d’attributs utilisateur à partir d’un annuaire d’utilisateurs local source dans le locataire Azure AD une fois l’utilisateur provisionné.

Les attributs déterminent les données utilisateur qui peuvent être retournées à une application dans une revendication, transmises dans des cookies de session ou transmises à l’application dans des variables d’en-tête HTTP.

## <a name="configure-the-maverics-identity-orchestrator-azure-ad-saml-connector-yaml-file"></a>Configurer le fichier YAML de Maverics Identity Orchestrator Azure AD SAML Connector

La configuration de Maverics Identity Orchestrator Azure AD Connector se présente comme suit :

```yaml
- name: AzureAD
  type: azure
  authType: saml
  samlMetadataURL: https://login.microsoftonline.com/<TENANT ID>/federationmetadata/2007-06/federationmetadata.xml?appid=<APP ID>
  samlRedirectURL: https://<AZURECOMPANY.COM>/<MY_APP>
  samlConsumerServiceURL: https://<AZURE-COMPANY.COM>/<MY_APP>
  jwtSigningKey: <SIGNING KEY>
  samlEntityID: https://<SUBDOMAIN>.maverics.org
  attributeMapping:
    displayName: username
    mailNickname: givenName
    givenName: givenName
    surname: sn
    userPrincipalName: mail
    password: password
```

## <a name="migrate-users-to-an-azure-ad-tenant"></a>Migrer des utilisateurs vers un locataire Azure AD

Suivez cette configuration pour migrer de façon incrémentielle des utilisateurs à partir d’un produit de gestion de l’accès web, tel que CA SiteMinder, Oracle Access Manager, ou IBM Tivoli. Vous pouvez également les migrer à partir d’un répertoire LDAP (Lightweight Directory Access Protocol) ou d’une base de données SQL.

### <a name="configure-your-application-permissions-in-azure-ad-to-create-users"></a>Configurer vos autorisations d’application dans Azure AD pour créer des utilisateurs

1. Dans votre locataire Azure AD, accédez à `App registrations` et sélectionnez l’application **Maverics Identity Orchestrator SAML Connector**.

1. Dans la page **Maverics Identity Orchestrator SAML Connector | Certificats et secrets**, sélectionnez `New client secret` puis sélectionnez l’option expiration. Sélectionnez le bouton **Copier** pour copier le secret et enregistrez-le sur votre ordinateur.

1. Dans le volet **Maverics Identity Orchestrator SAML Connector | Autorisations d’API**, sélectionnez **Ajouter une autorisation** puis, dans le volet **Demander des autorisations d’API**, sélectionnez **Microsoft Graph** et **Autorisations d’application**. 

1. Dans l’écran suivant, sélectionnez **user.ReadWrite.All**, puis sélectionnez **Ajouter des autorisations**. 

1. Revenez au volet **API autorisées** et sélectionnez **Accorder un consentement d’administrateur**.

### <a name="configure-the-maverics-identity-orchestrator-saml-connector-yaml-file-for-user-migration"></a>Configurer le fichier YAML de Maverics Identity Orchestrator SAML Connector pour la migration d’utilisateurs

Pour activer le workflow de migration utilisateur, ajoutez ces propriétés supplémentaires au fichier de configuration :
1. Entrez l’**URL Azure Graph** au format suivant : `graphURL: https://graph.microsoft.com`.
1. Entrez l’**URL du jeton OAuth** au format suivant : `oauthTokenURL: https://login.microsoftonline.com/<TENANT ID>/federationmetadata/2007-06/federationmetadata.xml?appid=<APP ID>`.
1. Entrez la clé secrète client précédemment générée au format suivant : `oauthClientSecret: <CLIENT SECRET>`.


Le fichier configuration final de Maverics Identity Orchestrator Azure AD Connector se présente comme suit :

```yaml
- name: AzureAD
  type: azure
  authType: saml
  samlMetadataURL: https://login.microsoftonline.com/<TENANT ID>/federationmetadata/2007-06/federationmetadata.xml?appid=<APP ID>
  samlRedirectURL: https://<AZURECOMPANY.COM>/<MY_APP>
  samlConsumerServiceURL: https://<AZURE-COMPANY.COM>/<MY_APP>
  jwtSigningKey: TBHPvTtu6NUqU84H3Q45grcv9WDJLHgTioqRhB8QGiVzghKlu1mHgP1QHVTAZZjzLlTBmQwgsSoWxGHRcT4Bcw==
  samlEntityID: https://<SUBDOMAIN>.maverics.org
  graphURL: https://graph.microsoft.com
  oauthTokenURL: https://login.microsoftonline.com/<TENANT ID>/oauth2/v2.0/token
  oauthClientID: <APP ID>
  oauthClientSecret: <NEW CLIENT SECRET>
  attributeMapping:
    displayName: username
    mailNickname: givenName
    givenName: givenName
    surname: sn
    userPrincipalName: mail
    password: password
```

### <a name="configure-maverics-zero-code-connector-for-siteminder"></a>Configurer Maverics Zero Code Connector pour SiteMinder

Vous utilisez le connecteur SiteMinder pour migrer des utilisateurs vers un locataire Azure AD. Vous consignez les utilisateurs dans des applications locales héritées qui sont protégées par SiteMinder à l’aide des identités et des informations d’identification Azure AD nouvellement créées.

Pour ce tutoriel, SiteMinder a été configuré pour protéger l’application héritée en utilisant l’authentification basée sur les formulaires et le cookie `SMSESSION`. Pour une intégration à une application qui consomme des informations d’authentification et de session via des en-têtes HTTP, vous devez ajouter la configuration d’émulation d’en-tête au connecteur.

Cet exemple mappe l’attribut `username` à l’en-tête HTTP `SM_USER` :

```yaml
  headers:
    SM_USER: username
```

Définissez `proxyPass` sur la destination vers laquelle les demandes sont proxysées. En général, cet emplacement est l’hôte de l’application protégée.

`loginPage` doit correspondre à l’URL du formulaire de connexion utilisé par SiteMinder lors de la redirection des utilisateurs à des fins d’authentification.

```yaml
connectors:
- name: siteminder-login-form
  type: siteminder
  loginType: form
  loginPage: /siteminderagent/forms/login.fcc
  proxyPass: http://host.company.com
```

### <a name="configure-maverics-zero-code-connector-for-ldap"></a>Configurer Maverics Zero Code Connector pour LDAP

Quand les applications sont protégées par un produit de gestion de l’accès web (WAM) tel que SiteMinder, les identités et les attributs des utilisateurs sont généralement stockés dans un annuaire LDAP.

Cette configuration de connecteur montre comment se connecter à l’annuaire LDAP. Le connecteur est configuré en tant que magasin d’utilisateurs pour SiteMinder afin que les informations de profil utilisateur appropriées puissent être collectées pendant le workflow de migration et qu’un utilisateur correspondant puisse être créé dans Azure AD.

* `baseDN` spécifie à quel endroit effectuer une recherche LDAP dans l’annuaire.

* `url` désigne l’adresse et le port du serveur LDAP auquel se connecter.

* `serviceAccountUsername` est le nom d’utilisateur utilisé pour se connecter au serveur LDAP, généralement exprimé sous la forme d’un nom unique de liaison (par exemple `CN=Directory Manager`).

* `serviceAccountPassword` est le mot de passe utilisé pour se connecter au serveur LDAP. Cette valeur est stockée dans l’instance Azure Key Vault déjà configurée.  

* `userAttributes` définit la liste d’attributs liés à l’utilisateur à interroger. Ces attributs sont ensuite mappés à des attributs Azure AD correspondants.

```yaml
- name: company-ldap
  type: ldap
  url: "ldap://ldap.company.com:389"
  baseDN: ou=People,o=company,c=US
  serviceAccountUsername: uid=admin,ou=Admins,o=company,c=US
  serviceAccountPassword: <vaulted-password>
  userAttributes:
    - uid
    - cn
    - givenName
    - sn
    - mail
    - mobile
```

### <a name="configure-the-migration-workflow"></a>Configurer le workflow de migration

La configuration du workflow de migration détermine la manière dont Maverics migre les utilisateurs de SiteMinder ou LDAP vers Azure AD.

Ce workflow :
- Utilise le connecteur SiteMinder pour proxyser la connexion SiteMinder. Les informations d’identification de l’utilisateur sont validées par le biais de l’authentification SiteMinder, puis transmises aux étapes suivantes du workflow.
- Récupère les attributs de profil utilisateur du magasin d’utilisateurs SiteMinder.
- Envoie une demande à l’API Microsoft Graph pour créer l’utilisateur dans votre locataire Azure AD.

Pour configurer le workflow de migration, procédez comme suit :

1. Donnez un nom au workflow, par exemple (par exemple **SiteMinder to Azure AD Migration** (Migration depuis SiteMinder vers Azure AD)).
1. Spécifiez `endpoint`, chemin HTTP sur lequel le workflow est exposé, qui déclenche les `actions` de ce workflow en réponse aux demandes. Le `endpoint` correspond généralement à l’application qui est proxysée (par exemple, `/my_app`). La valeur doit inclure des barres obliques de début et de fin.
1. Ajoutez les `actions` appropriées au workflow.

   a. Définissez la méthode `login` pour le connecteur SiteMinder. La valeur connector doit correspondre à la valeur name dans la configuration du connecteur.

   b. Définissez la méthode `getprofile` pour le connecteur LDAP.

   c.  Définissez la méthode `createuser` pour le connecteur Azure AD.

    ```yaml
      workflows:
      - name: SiteMinder to Azure AD Migration
        endpoint: /my_app/
        actions:
        - connector: siteminder-login-form
          method: login
        - connector: company-ldap
          method: getprofile
        - connector: AzureAD
          method: createuser
    ```
### <a name="verify-the-migration-workflow"></a>Vérifier le workflow de migration

1. Si le service Maverics n’est pas déjà en cours d’exécution, démarrez-le en exécutant la commande suivante :  

   `sudo systemctl start maverics`

1. Accédez à l’URL de connexion proxysée : `http://host.company.com/my_app`.
1. Fournissez les informations d’identification utilisateur permettant de se connecter à l’application alors que celle-ci est protégée par SiteMinder.
4. Accédez à **Accueil** > **Utilisateurs | Tous les utilisateurs** pour vérifier que l’utilisateur est créé dans le locataire Azure AD.  

### <a name="configure-the-session-abstraction-workflow"></a>Configurer le workflow d’abstraction de session

Le workflow d’abstraction de session déplace l’authentification et le contrôle d’accès de l’application web locale héritée vers le locataire Azure AD.

Le connecteur Azure utilise la méthode `login` pour rediriger l’utilisateur vers l’URL de connexion, en supposant qu’il n’existe aucune session.

Une fois l’authentification terminée, le jeton de session créé en tant que résultat est passé à Maverics. La méthode `emulate` du connecteur SiteMinder est utilisée pour émuler la session basée sur les cookies et/ou la session basée sur l’en-tête, puis décorer la demande avec les attributs supplémentaires requis par l’application.

1. Donnez un nom au workflow, par exemple (par exemple **SiteMinder Session Abstraction** (Abstraction de session SiteMinder)).
1. Spécifiez `endpoint`, qui correspond à l’application proxysée. La valeur doit inclure des barres obliques de début et de fin (par exemple `/my_app/`).
1. Ajoutez les `actions` appropriées au workflow.

   a. Définissez la méthode `login` pour le connecteur Azure. La valeur `connector` doit correspondre à la valeur `name` dans la configuration du connecteur.

   b. Définissez la méthode `emulate` pour le connecteur SiteMinder.

     ```yaml
      - name: SiteMinder Session Abstraction
        endpoint: /my_app/
        actions:
      - connector: azure
        method: login
      - connector: siteminder-login-form
        method: emulate
     ```
### <a name="verify-the-session-abstraction-workflow"></a>Vérifier le workflow de l’abstraction de session

1. Accédez à l’URL de l’application proxysée : `https://<AZURECOMPANY.COM>/<MY_APP>`. 
    
    Vous êtes redirigé vers la page de connexion proxysée.

1. Entrez les informations d’identification de l’utilisateur Azure AD.

   Vous devriez être redirigé vers l’application comme si vous étiez authentifié directement par SiteMinder.
