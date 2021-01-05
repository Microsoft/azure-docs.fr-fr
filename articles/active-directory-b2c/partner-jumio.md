---
title: Tutoriel pour configurer Azure Active Directory B2C avec Jumio
titleSuffix: Azure AD B2C
description: Dans ce tutoriel, vous configurez Azure Active Directory B2C avec Jumio pour la vérification automatisée des identités, visant à protéger les données client.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/20/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 66ec0d4b09dc983eb898d63d45b3dd7cab291c4c
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96928662"
---
# <a name="tutorial-for-configuring-jumio-with-azure-active-directory-b2c"></a>Tutoriel de configuration de Jumio avec Azure Active Directory B2C

Dans cet exemple de tutoriel, nous fournissons des conseils sur la façon d’intégrer Azure Active Directory B2C (Azure AD B2C) avec [Jumio](https://www.jumio.com/). Jumio est un service de vérification d’identité, qui permet de vérifier de façon automatisée l’identité en temps réel afin de protéger les données client.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous avez :

- Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).

- Un [locataire Azure AD B2C](./tutorial-create-tenant.md) lié à votre abonnement Azure.

## <a name="scenario-description"></a>Description du scénario

L’intégration de Jumio inclut les composants suivants :

- Azure AD B2C : serveur d’autorisation chargé de vérifier les informations d’identification de l’utilisateur. Également connu sous le nom de fournisseur d’identité.

- Jumio : service qui réceptionne les détails d’identité fournis par l’utilisateur et les vérifie.

- API REST intermédiaire : API qui implémente l’intégration entre Azure AD B2C et le service Jumio.

- Stockage Blob Azure : service qui fournit des fichiers d’interface utilisateur personnalisés aux stratégies Azure AD B2C.

Le diagramme d’architecture suivant illustre l’implémentation.

![Schéma de l’architecture d’une intégration d’Azure AD B2C avec Jumio.](./media/partner-jumio/jumio-architecture-diagram.png)

|Étape | Description |
|:-----| :-----------|
| 1. | L’utilisateur arrive sur une page pour se connecter ou s’inscrire afin de créer un compte. Azure AD B2C collecte les attributs de l’utilisateur.
| 2. | Azure AD B2C appelle l’API de couche intermédiaire et lui transmet les attributs de l’utilisateur.
| 3. | L’API de couche intermédiaire collecte les attributs de l’utilisateur et les convertit en un format utilisable par l’API Jumio. Ensuite, elle envoie les attributs à Jumio.
| 4. | Jumio consomme ces informations et les traite, puis renvoie le résultat à l’API de couche intermédiaire.
| 5. | L’API de couche intermédiaire traite les informations et renvoie les informations pertinentes à Azure AD B2C.
| 6. | Azure AD B2C reçoit les informations renvoyées par l’API de couche intermédiaire. Si elles montrent une réponse d’échec, un message d’erreur est affiché à l’utilisateur. Si elles montrent une réponse de réussite, l’utilisateur est authentifié et inscrit dans l’annuaire.

## <a name="sign-up-with-jumio"></a>S’inscrire auprès de Jumio

Pour créer un compte Jumio, contactez [Jumio](https://www.jumio.com/contact/).

## <a name="configure-azure-ad-b2c-with-jumio"></a>Configurer Azure AD B2C avec Jumio

Après avoir créé un compte Jumio, vous utilisez le compte pour configurer Azure AD B2C. Les sections suivantes décrivent ce processus dans l’ordre des opérations.

### <a name="deploy-the-api"></a>Déployer l’API

Déployez le [code d’API](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Jumio/API/Jumio.Api) fourni sur un service Azure. Vous pouvez publier le code à partir de Visual Studio, en suivant [ces instructions](/visualstudio/deployment/quickstart-deploy-to-azure).

>[!NOTE]
>Vous aurez besoin de l’URL du service déployé pour configurer Azure AD avec les paramètres requis.

### <a name="deploy-the-client-certificate"></a>Déployer le certificat client

1. Un certificat client contribue à protéger l’appel de l’API Jumio. Créez un certificat auto-signé à l’aide de l’exemple de code PowerShell suivant :

   ``` PowerShell
   $cert = New-SelfSignedCertificate -Type Custom -Subject "CN=Demo-SigningCertificate" -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.3") -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -NotAfter (Get-Date).AddYears(2) -CertStoreLocation "Cert:\CurrentUser\My"
   $cert.Thumbprint
   $pwdText = "Your password"
   $pwd = ConvertTo-SecureString -String $pwdText -Force -AsPlainText
   Export-PfxCertificate -Cert $Cert -FilePath "{your-local-path}\Demo-SigningCertificate.pfx" -Password $pwd.

   ```

   Le certificat est ensuite exporté à l’emplacement spécifié pour ``{your-local-path}``.

3. Importez le certificat dans Azure App Service en suivant les instructions fournies dans [cet article](../app-service/configure-ssl-certificate.md#upload-a-private-certificate).

### <a name="create-a-signingencryption-key"></a>Créer une clé de signature/chiffrement

Créez une chaîne aléatoire de plus de 64 caractères contenant uniquement des lettres et des chiffres.

Par exemple : ``C9CB44D98642A7062A0D39B94B6CDC1E54276F2E7CFFBF44288CEE73C08A8A65``

Utilisez le script PowerShell suivant pour créer cette chaîne :

```PowerShell
-join ((0x30..0x39) + ( 0x41..0x5A) + ( 0x61..0x7A) + ( 65..90 ) | Get-Random -Count 64  | % {[char]$_})

```

### <a name="configure-the-api"></a>Configurer l’API

Vous pouvez [configurer les paramètres d’application dans Azure App Service](../app-service/configure-common.md#configure-app-settings). Cette méthode permet de configurer ces paramètres de façon sécurisée sans les archiver dans un référentiel. Vous devrez fournir les paramètres suivants à l’API REST :

| Paramètres de l’application | Source | Notes |
| :-------- | :------------| :-----------|
|JumioSettings:AuthUsername | Configuration du compte Jumio |     |
|JumioSettings:AuthPassword | Configuration du compte Jumio |     |
|AppSettings:SigningCertThumbprint|Empreinte du certificat auto-signé créé|  |
|AppSettings:IdTokenSigningKey| Clé de signature créée à l’aide de PowerShell | |
| AppSettings:IdTokenEncryptionKey |Clé de chiffrement créée à l’aide de PowerShell
| AppSettings:IdTokenIssuer | Émetteur à utiliser pour le jeton JWT (une valeur GUID est recommandée) |
| AppSettings:IdTokenAudience  | Audience à utiliser pour le jeton JWT (une valeur GUID est recommandée) |
|AppSettings:BaseRedirectUrl | URL de base de la stratégie Azure AD B2C | https://{your-tenant-name}.b2clogin.com/{your-application-id}|
| WEBSITE_LOAD_CERTIFICATES| Empreinte du certificat auto-signé créé |

### <a name="deploy-the-ui"></a>Déployer l’interface utilisateur

1. Configurer un [conteneur de stockage blob dans votre compte de stockage](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container).

2. Stockez les fichiers d’interface utilisateur du [dossier d’interface utilisateur](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Jumio/UI) dans votre conteneur d’objets blob.

#### <a name="update-ui-files"></a>Mettre à jour les fichiers d’interface utilisateur

1. Dans les fichiers d’interface utilisateur, accédez au dossier [ocean_blue](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Jumio/UI/ocean_blue).

2. Ouvrez chaque fichier HTML.

3. Recherchez et remplacez `{your-ui-blob-container-url}` par l’URL de votre conteneur d’objets blob.

4. Recherchez et remplacez `{your-intermediate-api-url}` par l’URL du service d’application API intermédiaire.

>[!NOTE]
> Une bonne pratique consiste à ajouter une notification de consentement sur la page de collection d’attributs. Informez les utilisateurs que les informations seront envoyées à des services tiers à des fins de vérification d’identité.

### <a name="configure-the-azure-ad-b2c-policy"></a>Configurer la stratégie Azure AD B2C

1. Accédez à la [stratégie d’Azure AD B2C](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Jumio/Policies) dans le dossier stratégies.

2. Suivez [cet article](./custom-policy-get-started.md?tabs=applications#custom-policy-starter-pack) pour télécharger le [pack de démarrage LocalAccounts](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/LocalAccounts).

3. Configurez la stratégie pour le locataire Azure AD B2C.

>[!NOTE]
>Mettez à jour les stratégies fournies en fonction de votre locataire spécifique.

## <a name="test-the-user-flow"></a>Tester le flux utilisateur

1. Ouvrez le locataire Azure AD B2C. Sous **Stratégies**, sélectionnez **Identity Experience Framework**.

2. Sélectionnez votre **SignUpSignIn** créé précédemment.

3. Sélectionnez **Exécuter le flux utilisateur**, puis :

   a. Pour **Application**, sélectionnez l’application inscrite (JWT dans cet exemple).

   b. Pour **URL de réponse**, sélectionnez l’**URL de redirection**.

   c. Sélectionnez **Exécuter le flux utilisateur**.

4. Suivez le processus d’inscription et créez un compte.

5. Le service Jumio est appelé pendant le flux, après la création de l’attribut utilisateur. Si le flux est incomplet, vérifiez que l’utilisateur n’est pas enregistré dans l’annuaire.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Stratégies personnalisées dans Azure AD B2C](./custom-policy-overview.md)

- [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](./custom-policy-get-started.md?tabs=applications)
