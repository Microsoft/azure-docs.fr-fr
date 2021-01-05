---
title: Authentification basée sur un en-tête avec PingAccess pour Proxy d’application d’Azure AD
description: Publiez des applications avec PingAccess et Application Proxy pour prendre en charge l’authentification basée sur un en-tête.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 10/24/2019
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: e09bb0b07112a962b709c380c48f2a656c16097b
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94663230"
---
# <a name="header-based-authentication-for-single-sign-on-with-application-proxy-and-pingaccess"></a>Authentification basée sur l’en-tête pour une authentification unique avec le proxy d’application et PingAccess

Le proxy d’application Azure Active Directory (Azure AD) a conclu un partenariat avec PingAccess afin que vos clients Azure AD puissent accéder à un plus grand nombre de vos applications. PingAccess fournit une autre option au-delà de l’[authentification unique intégrée basée sur l’en-tête](application-proxy-configure-single-sign-on-with-headers.md).

## <a name="whats-pingaccess-for-azure-ad"></a>Présentation de PingAccess pour Azure AD

Avec PingAccess pour Azure AD, vous pouvez donner aux utilisateurs l’accès aux applications qui utilisent des en-têtes pour l’authentification et ils peuvent s’y connecter avec l’authentification unique (SSO). Le proxy d’application traite ces applications comme n’importe quelle autre application, en utilisant Azure AD pour authentifier l’accès, puis pour faire transiter le trafic via le service de connecteur. PingAccess est situé devant les applications et convertit le jeton d’accès issu d’Azure AD en en-tête. L’application reçoit alors l’authentification dans le format qu’elle peut lire.

Vos utilisateurs ne voient pas de différence quand ils se connectent pour utiliser vos applications d’entreprise. Ils pourront toujours travailler depuis n’importe où et sur n’importe quel appareil. Les connecteurs de proxy d’application dirigent le trafic distant vers toutes les applications, quel que soit leur type d’authentification, si bien qu’ils continuent à équilibrer automatiquement les charges.

## <a name="how-do-i-get-access"></a>Comment puis-je avoir accès ?

Comme ce scénario est issu d’un partenariat entre Azure Active Directory et PingAccess, vous avez besoin de licences pour les deux services. Toutefois, les abonnements Azure Active Directory Premium incluent une licence PingAccess de base qui couvre jusqu’à 20 applications. Si vous devez publier plus de 20 applications basées sur un en-tête, vous pouvez acheter une licence supplémentaire auprès de PingAccess.

Pour plus d’informations, consultez la page [Éditions d’Azure Active Directory](../fundamentals/active-directory-whatis.md).

## <a name="publish-your-application-in-azure"></a>Publier votre application dans Azure

Cet article s’adresse aux personnes qui souhaitent publier une application avec ce scénario pour la première fois. En plus de détailler la procédure de publication, il vous guide pour une bonne prise en main du proxy d’application et de PingAccess. Si vous avez déjà configuré les deux services mais souhaitez bénéficier d’un rappel sur la procédure de publication, passez à la section [Ajouter votre application dans Azure AD avec le proxy d’application](#add-your-application-to-azure-ad-with-application-proxy).

> [!NOTE]
> Comme ce cas de figure repose sur un partenariat entre Azure AD et PingAccess, certaines de ces instructions figurent sur le site Ping Identity.

### <a name="install-an-application-proxy-connector"></a>Installer un connecteur Application Proxy

Si vous avez activé le proxy d’application et déjà activé et installé un connecteur, vous pouvez ignorer cette section et passer à [Ajouter votre application dans Azure AD avec le proxy d’application](#add-your-application-to-azure-ad-with-application-proxy).

Le connecteur de proxy d’application est un service Windows Server qui dirige le trafic de vos employés distants vers vos applications publiées. Pour obtenir des instructions d’installation plus détaillées, consultez [Tutoriel : Ajouter une application locale pour un accès à distance via le service Proxy d’application d’Azure Active Directory](application-proxy-add-on-premises-application.md).

1. Connectez-vous au [portail Azure Active Directory](https://aad.portal.azure.com/) en tant qu’administrateur d’application. La page du **Centre d’administration Azure Active Directory** s’affiche.
1. Sélectionnez **Azure Active Directory** > **Proxy d’application** > **Télécharger le service de connecteur**. La page **Téléchargement du connecteur Proxy d’application** s’affiche.

   ![Téléchargement du connecteur Proxy d’application](./media/application-proxy-configure-single-sign-on-with-ping-access/application-proxy-connector-download.png)

1. Suivez les instructions d'installation.

Le téléchargement du connecteur doit activer automatiquement le proxy d’application pour votre annuaire, mais si ce n’est pas le cas, vous pouvez sélectionner **Activer le proxy d’application**.

### <a name="add-your-application-to-azure-ad-with-application-proxy"></a>Ajouter votre application dans Azure AD avec le proxy d’application

Vous devez effectuer deux actions dans le portail Azure. Tout d’abord, vous devez publier votre application avec le proxy d’application. Ensuite, vous devez collecter certaines informations sur l’application que vous pouvez utiliser pendant la procédure PingAccess.

#### <a name="publish-your-application"></a>Publication de votre application

Vous devez commencer par publier votre application. Cette action implique :

- l’ajout de votre application en local à Azure AD
- l’affectation d’un utilisateur pour tester l’application et le choix de l’authentification unique basée sur l’en-tête
- la configuration des URL de redirection de l’application
- l’octroi d’autorisations pour les utilisateurs et d’autres applications dans le but d’utiliser votre application en local

Pour publier votre propre application en local :

1. Si vous ne l’avez pas fait dans la dernière section, connectez-vous au [portail Azure Active Directory](https://aad.portal.azure.com/) en tant qu’administrateur d’application.
1. Sélectionnez **Applications d’entreprise** > **Nouvelle application** > **Ajouter une application locale**. La page **Ajouter votre propre application locale** s’affiche.

   ![Ajouter votre propre application locale](./media/application-proxy-configure-single-sign-on-with-ping-access/add-your-own-on-premises-application.png)
1. Entrez les informations concernant votre nouvelle application dans les champs obligatoires. Utilisez les instructions ci-dessous pour les paramètres.

   > [!NOTE]
   > Pour obtenir une description plus détaillée de cette étape, consultez [Ajouter une application locale à Azure AD](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad).

   1. **URL interne** : normalement, vous devez indiquer l’URL de la page de connexion de l’application lorsque vous êtes sur le réseau d’entreprise. Pour ce scénario, le connecteur doit traiter le proxy PingAccess en tant que première page de l’application. Utilisez le format suivant : `https://<host name of your PingAccess server>:<port>`. Le port par défaut est 3000, mais vous pouvez le configurer dans PingAccess.

      > [!WARNING]
      > Pour ce type d’authentification unique, l’URL interne doit utiliser `https` et ne peut pas utiliser `http`. En outre, il existe une contrainte lors de la configuration d’une application selon laquelle deux applications ne doivent pas avoir la même URL interne, car cela permet à Proxy d’application de les distinguer.

   1. **Méthode de pré-authentification** : Choisissez **Azure Active Directory**.
   1. **Traduire l’URL dans les en-têtes** : Choisissez **Non**.

   > [!NOTE]
   > S’il s’agit de votre première application, utilisez le port 3000 pour démarrer et revenir afin de mettre à jour ce paramètre si vous modifiez votre configuration PingAccess. Pour les applications suivantes, le port doit correspondre à l’écouteur que vous avez configuré dans PingAccess. Découvrez plus en détail les [écouteurs dans PingAccess](https://support.pingidentity.com/s/document-item?bundleId=pingaccess-52&topicId=reference/ui/pa_c_Listeners.html).

1. Sélectionnez **Ajouter**. La page de vue d’ensemble de la nouvelle application s’affiche.

À présent, affectez un utilisateur pour le test de l’application et choisissez l’authentification unique basée sur l’en-tête :

1. Dans la barre latérale de l’application, sélectionnez **Utilisateurs et groupes** > **Ajouter un utilisateur** > **Utilisateurs et groupes (\<Number> Sélectionné)** . La liste des utilisateurs et des groupes apparaît pour que vous en choisissiez un membre.

   ![Affiche la liste des utilisateurs et des groupes](./media/application-proxy-configure-single-sign-on-with-ping-access/users-and-groups.png)

1. Sélectionnez un utilisateur pour le test de l’application, puis sélectionnez **Sélectionner**. Vérifiez que ce compte de test a accès à l’application locale.
1. Sélectionnez **Attribuer**.
1. Dans la barre latérale de l’application, sélectionnez **Authentification unique** > **Basée sur l’en-tête**.

   > [!TIP]
   > S’il s’agit votre première authentification unique basée sur un en-tête, vous devez installer PingAccess. Pour vous assurer que votre abonnement Azure est automatiquement associé à votre installation PingAccess, utilisez le lien de cette page d’authentification unique afin de télécharger PingAccess. Vous pouvez ouvrir le site de téléchargement maintenant ou revenir sur cette page ultérieurement.

   ![Montre l’écran d’authentification unique basée sur l’en-tête et PingAccess](./media/application-proxy-configure-single-sign-on-with-ping-access/sso-header.png)

1. Sélectionnez **Enregistrer**.

Ensuite, vérifiez que votre URL de redirection est définie sur votre URL externe :

1. À partir de la barre latérale **Centre d’administration Azure Active Directory**, sélectionnez **Azure Active Directory** > **Inscriptions des applications**. La liste des applications apparaît.
1. Sélectionnez votre application.
1. Sélectionnez le lien en regard de **URI de redirection**, qui indique le nombre d’URI de redirection configurés pour les clients web et publics. La page **\<application name> - Authentification** s’affiche.
1. Vérifiez si l’URL externe que vous avez affectée à votre application précédemment se trouve dans la liste **URI de redirection**. Si ce n’est pas le cas, ajoutez maintenant l’URL externe, en utilisant un type d’URI de redirection **Web**, puis sélectionnez **Enregistrer**.

En plus de l’URL externe, un point de terminaison d’autorisation d’Azure Active Directory sur l’URL externe doit être ajouté à la liste des URI de redirection.

`https://*.msappproxy.net/pa/oidc/cb`
`https://*.msappproxy.net/`

Enfin, configurez votre application locale afin que les utilisateurs aient un accès en lecture et les autres applications un accès en lecture/écriture :

1. À partir de la barre latérale **Inscriptions des applications** pour votre application, sélectionnez **API autorisées** > **Ajouter une autorisation** > **API Microsoft** > **Microsoft Graph**. La page **Demander des autorisations d’API** pour **Microsoft Graph** s’affiche et elle contient les API pour Windows Azure Active Directory.

   ![Page Demander des autorisations d’API](./media/application-proxy-configure-single-sign-on-with-ping-access/required-permissions.png)

1. Sélectionnez **Autorisations déléguées** > **Utilisateur** > **User.Read**.
1. Sélectionnez **Autorisations d’application** > **Application** > **Application.ReadWrite.All**.
1. Sélectionnez **Ajouter des autorisations**.
1. Dans la page **API autorisées**, sélectionnez **Accorder un consentement d’administrateur pour \<your directory name>** .

#### <a name="collect-information-for-the-pingaccess-steps"></a>Collecter les informations pour la procédure PingAccess

Vous devez collecter ces trois informations (tous les GUID) pour configurer votre application avec PingAccess :

| Nom du champ Azure AD | Nom du champ PingAccess | Format de données |
| --- | --- | --- |
| **ID d’application (client)** | **ID client** | GUID |
| **ID de l’annuaire (locataire)** | **Émetteur** | GUID |
| `PingAccess key` | **Clé secrète client** | Chaîne aléatoire |

Pour collecter ces informations :

1. À partir de la barre latérale **Centre d’administration Azure Active Directory**, sélectionnez **Azure Active Directory** > **Inscriptions des applications**. La liste des applications apparaît.
1. Sélectionnez votre application. La page **Inscriptions des applications** de votre application apparaît.

   ![Vue d’ensemble de l’inscription pour une application](./media/application-proxy-configure-single-sign-on-with-ping-access/registration-overview-for-an-application.png)

1. À côté de la valeur **ID d’application (client)** , sélectionnez l’icône **Copier dans le Presse-papiers**, puis copiez et enregistrez-la. Vous allez spécifier cette valeur ultérieurement en tant qu’ID client de PingAccess.
1. À côté de la valeur **ID de l’annuaire (locataire)** , sélectionnez également **Copier dans le Presse-papiers**, puis copiez et enregistrez-la. Vous allez spécifier cette valeur ultérieurement en tant qu’émetteur de PingAccess.
1. À partir de la barre latérale de la page **Inscriptions des applications** pour votre application, sélectionnez **Certificats et secrets** > **Nouveau secret client**. La page **Ajouter un secret client** apparaît.

   ![Page Ajouter un secret client](./media/application-proxy-configure-single-sign-on-with-ping-access/add-a-client-secret.png)

1. Dans **Description**, tapez `PingAccess key`.
1. Sous **Expire**, choisissez comment définir la clé de PingAccess : **Dans 1 an**, **Dans 2 ans** ou **Jamais**.
1. Sélectionnez **Ajouter**. La clé de PingAccess s’affiche dans la table des secrets client, avec une chaîne aléatoire qui est automatiquement renseignée dans le champ **VALUE**.
1. À côté du champ **VALUE** de la clé PingAccess, sélectionnez l’icône **Copier dans le Presse-papiers**, puis copiez et enregistrez-la. Vous allez spécifier cette valeur ultérieurement en tant que secret client de PingAccess.

**Mettez à jour le champ `acceptMappedClaims` :**

1. Connectez-vous au [portail Azure Active Directory](https://aad.portal.azure.com/) en tant qu’administrateur d’application.
1. Sélectionnez **Azure Active Directory** > **Inscriptions des applications**. La liste des applications apparaît.
1. Sélectionnez votre application.
1. À partir de la barre latérale de la page **Inscriptions des applications** pour votre application, sélectionnez **Manifeste**. Le code JSON de manifeste pour l’inscription de votre application s’affiche.
1. Recherchez le champ `acceptMappedClaims` et remplacez la valeur par `True`.
1. Sélectionnez **Enregistrer**.

### <a name="use-of-optional-claims-optional"></a>Utilisation de revendications facultatives (facultatif)

Les revendications facultatives vous permettent d’ajouter des revendications standard mais non incluses par défaut qu’ont tous les utilisateurs et locataires. Vous pouvez configurer des revendications facultatives pour votre application en modifiant le manifeste de l’application. Pour plus d’informations, consultez l’article [Manifeste d’application Azure Active Directory](../develop/reference-app-manifest.md)

Exemple d’insertion d’adresse e-mail dans l’élément access_token que PingAccess consommera :
```
    "optionalClaims": {
        "idToken": [],
        "accessToken": [
            {
                "name": "email",
                "source": null,
                "essential": false,
                "additionalProperties": []
            }
        ],
        "saml2Token": []
    },
```

### <a name="use-of-claims-mapping-policy-optional"></a>Utilisation d’une stratégie de mappage de revendications (facultatif)

[Stratégie de mappage de revendications (préversion)](../develop/active-directory-claims-mapping.md#claims-mapping-policy-properties) pour les attributs qui n’existent pas dans Azure AD. Le mappage de revendications vous permet de migrer les anciennes applications locales vers le cloud en ajoutant des revendications personnalisées supplémentaires qui sont soutenues par vos objets ADFS ou utilisateur.

Pour que votre application utilise une revendication personnalisée et comprenne des champs supplémentaires, [créez une stratégie de mappage de revendications personnalisées et attribuez-la à l’application](../develop/active-directory-claims-mapping.md#claims-mapping-policy-assignment).

> [!NOTE]
> Pour utiliser une revendication personnalisée, vous devez également disposer d’une stratégie personnalisée définie et affectée à l’application. Cette stratégie doit inclure tous les attributs personnalisés nécessaires.
>
> Vous pouvez effectuer la définition et l’affectation de la stratégie via PowerShell ou Microsoft Graph. Si vous utilisez PowerShell, vous devez d’abord utiliser `New-AzureADPolicy`, puis l’attribuer à l’application avec `Add-AzureADServicePrincipalPolicy`. Pour plus d’informations, consultez [Attribution de stratégie de mappage de revendications](../develop/active-directory-claims-mapping.md#claims-mapping-policy-assignment).

Exemple :
```powershell
$pol = New-AzureADPolicy -Definition @('{"ClaimsMappingPolicy":{"Version":1,"IncludeBasicClaimSet":"true", "ClaimsSchema": [{"Source":"user","ID":"employeeid","JwtClaimType":"employeeid"}]}}') -DisplayName "AdditionalClaims" -Type "ClaimsMappingPolicy"

Add-AzureADServicePrincipalPolicy -Id "<<The object Id of the Enterprise Application you published in the previous step, which requires this claim>>" -RefObjectId $pol.Id
```

### <a name="enable-pingaccess-to-use-custom-claims"></a>Activer PingAccess pour utiliser les revendications personnalisées

L’activation de PingAccess pour utiliser les revendications personnalisées est facultative, mais requise si vous pensez que l’application consommera des revendications supplémentaires.

Lorsque vous allez configurer PingAccess à l’étape suivante, la session Web que vous allez créer (Paramètres -> Accès -> Sessions Web) doit avoir l’option **Request Profile (Demander un profil)** désélectionnée et l’option **Refresh User Attributes (Actualiser les attributs utilisateur)** définie sur la valeur **Non**.

## <a name="download-pingaccess-and-configure-your-application"></a>Télécharger PingAccess et configurer votre application

Maintenant que vous avez terminé toutes les étapes de configuration d’Azure Active Directory, vous pouvez passer à la configuration de PingAccess.

La procédure détaillée concernant la partie PingAccess de ce scénario continue dans la documentation de Ping Identity. Suivez les instructions fournies dans [Configurer PingAccess pour Azure AD afin de protéger les applications publiées à l’aide du proxy d'application Microsoft Azure Active Directory](https://support.pingidentity.com/s/document-item?bundleId=pingaccess-52&topicId=agents/azure/pa_c_PAAzureSolutionOverview.html) sur le site web de Ping Identity et téléchargez la [dernière version de PingAccess](https://www.pingidentity.com/en/lp/azure-download.html?).

Ces étapes vous aideront à installer PingAccess et à configurer un compte PingAccess (si vous n’en avez pas encore). Ensuite, pour créer une connexion Azure AD OpenID Connect (OIDC), vous configurez un fournisseur de jetons avec la valeur d’**ID de l’annuaire (locataire)** que vous avez copiée à partir du portail Azure AD. Ensuite, pour créer une session Web sur PingAccess, vous utilisez les valeurs **ID d’application (client)** et `PingAccess key`. Enfin, vous pouvez configurer le mappage d’identité et créer un hôte virtuel, le site et l’application.

### <a name="test-your-application"></a>Tester votre application

Une fois toutes ces étapes effectuées, votre application doit être opérationnelle. Pour la tester, ouvrez un navigateur et accédez à l’URL externe que vous avez créée lors de la publication de l’application dans Azure. Connectez-vous au compte de test que vous avez attribué à l’application.

## <a name="next-steps"></a>Étapes suivantes

- [Configurer PingAccess pour Azure AD afin de protéger les applications publiées à l’aide du proxy d'application Microsoft Azure Active Directory](https://docs.pingidentity.com/bundle/pingaccess-60/page/jep1564006742933.html)
- [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](what-is-single-sign-on.md)
- [Résoudre les problèmes de proxy d’application et les messages d’erreur](application-proxy-troubleshoot.md)