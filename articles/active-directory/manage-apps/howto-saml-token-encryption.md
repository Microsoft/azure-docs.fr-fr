---
title: Chiffrement des jetons SAML dans Azure Active Directory
description: Découvrez comment configurer le chiffrement des jetons SAML Azure Active Directory.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/13/2020
ms.author: kenwith
ms.reviewer: paulgarn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 820e2cb0d422597f0e649e6934fd8bb11c1521db
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997475"
---
# <a name="how-to-configure-azure-ad-saml-token-encryption"></a>Procédure : Configurer le chiffrement de jeton SAML Azure AD

> [!NOTE]
> Le chiffrement des jetons est une fonctionnalité Azure Active Directory (Azure AD) Premium. Pour en savoir plus sur les éditions, les fonctionnalités et les tarifs d’Azure AD, consultez [Tarification Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).

Le chiffrement des jetons SAML permet d’utiliser des assertions SAML chiffrées avec une application qui le prend en charge. Quand il est configuré pour une application, Azure AD chiffre les assertions SAML qu’il émet pour celle-ci à l’aide de la clé publique obtenue à partir d’un certificat stocké dans Azure AD. L’application doit utiliser la clé privée correspondante pour déchiffrer le jeton avant que celui-ci ne puisse être utilisé comme preuve d’authentification pour l’utilisateur connecté.

Le chiffrement des assertions SAML entre Azure AD et l’application fournit une garantie supplémentaire que le contenu du jeton ne peut pas être intercepté et que des données personnelles ou professionnelles ne peuvent pas être compromises.

Même sans le chiffrement des jetons, les jetons SAML Azure AD ne sont jamais transmis en clair sur le réseau. Azure AD exige que les échanges de demande/réponse de jeton s’effectuent via des canaux chiffrés HTTPS/TLS afin que les communications entre le fournisseur d’identité, le navigateur et l’application se déroulent sur des liaisons chiffrées. Considérons la valeur de chiffrement des jetons dans votre situation par rapport à la surcharge de gestion de certificats supplémentaires.   

Pour configurer le chiffrement des jetons, vous devez charger un fichier de certificat X.509 qui contient la clé publique dans l’objet d’application Azure AD qui représente l’application. Pour obtenir le certificat X.509, vous pouvez le télécharger à partir de l’application elle-même ou l’obtenir auprès du fournisseur de l’application dans les cas où ce dernier fournit des clés de chiffrement. Dans les cas où l’application s’attend à ce que vous fournissiez une clé privée, il peut être créé à l’aide d’outils de chiffrement, la partie clé privée étant chargée dans le magasin de clés de l’application et le certificat de clé publique correspondant étant chargé dans Azure AD.

Azure AD utilise AES-256 pour chiffrer les données d’assertion SAML.

## <a name="configure-saml-token-encryption"></a>Configurer le chiffrement des jetons SAML

Pour configurer le chiffrement des jetons SAML, procédez comme suit :

1. Obtenez un certificat de clé publique correspondant à une clé privée qui est configurée dans l’application.

    Créez une paire de clés asymétriques à utiliser pour le chiffrement. Par ailleurs, si l’application fournit une clé publique à utiliser pour le chiffrement, suivez les instructions de l’application pour télécharger le certificat X.509.

    La clé publique doit être stockée dans un fichier de certificat X.509 au format .cer.

    Si l’application utilise une clé que vous créez pour votre instance, suivez les instructions fournies par votre application pour l’installation de la clé privée que l’application utilisera pour déchiffrer les jetons à partir de votre locataire Azure AD.

1. Ajoutez le certificat à la configuration de l’application dans Azure AD.

### <a name="to-configure-token-encryption-in-the-azure-portal"></a>Pour configurer le chiffrement des jetons dans le portail Azure

Vous pouvez ajouter le certificat public à la configuration de votre application dans le portail Azure.

1. Accédez au [portail Azure](https://portal.azure.com).

1. Accédez au panneau **Azure Active Directory > Applications d’entreprise**, puis sélectionnez l’application pour laquelle vous voulez configurer le chiffrement des jetons.

1. Dans la page de l’application, sélectionnez **Chiffrement des jetons**.

    ![Option de chiffrement des jetons dans le portail Azure](./media/howto-saml-token-encryption/token-encryption-option-small.png)

    > [!NOTE]
    > L’option **Chiffrement des jetons** est disponible uniquement pour les applications SAML qui ont été configurées à partir du panneau **Applications d’entreprise** dans le portail Azure, à partir de la Galerie d’applications ou d’une application hors galerie. Pour les autres applications, cette option de menu est désactivée. Pour les applications inscrites par le biais de l’expérience **Inscriptions d’applications** dans le portail Azure, vous pouvez configurer le chiffrement pour les jetons SAML avec le manifeste d’application à l’aide de Microsoft Graph ou de PowerShell.

1. Sur la page **Chiffrement des jetons**, sélectionnez **Importer un certificat** pour importer le fichier .cer qui contient votre certificat public X.509.

    ![Importer le fichier .cer qui contient le certificat X.509](./media/howto-saml-token-encryption/import-certificate-small.png)

1. Une fois le certificat importé et la clé privée configurée pour une utilisation côté application, activez le chiffrement en sélectionnant les points de suspension ( **...** ) en regard de l’état de l’empreinte numérique, puis sélectionnez **Activer le chiffrement des jetons** dans les options du menu déroulant.

1. Sélectionnez **Oui** pour confirmer l’activation du certificat de chiffrement de jeton.

1. Vérifiez que les assertions SAML émises pour l’application sont chiffrées.

### <a name="to-deactivate-token-encryption-in-the-azure-portal"></a>Pour désactiver le chiffrement des jetons dans le portail Azure

1. Dans le portail Azure, accédez à **Azure Active Directory > Applications d’entreprise**, puis sélectionnez l’application pour laquelle le chiffrement des jetons SAML est activé.

1. Dans la page de l’application, sélectionnez **Chiffrement des jetons**, recherchez le certificat, puis sélectionnez l’option **...** pour afficher le menu déroulant.

1. Sélectionnez **Désactiver le chiffrement des jetons**.

## <a name="configure-saml-token-encryption-using-graph-api-powershell-or-app-manifest"></a>Configurer le chiffrement des jetons SAML à l’aide de l’API Graph, de PowerShell ou d’un manifeste d’application

Les certificats de chiffrement sont stockés sur l’objet d’application dans Azure AD avec une balise d’utilisation `encrypt`. Vous pouvez configurer plusieurs certificats de chiffrement. Celui qui est actif pour le chiffrement des jetons est identifié par l’attribut `tokenEncryptionKeyID`.

Vous avez besoin de l’ID d’objet de l’application pour configurer le chiffrement des jetons à l’aide de l’API Microsoft Graph ou de PowerShell. Vous pouvez trouver cette valeur programmatiquement ou en accédant à la page **Propriétés** de l’application dans le portail Azure et en notant la valeur **ID objet**.

Quand vous configurez une valeur keyCredential à l’aide de Graph, de PowerShell ou dans le manifeste d’application, vous devez générer un GUID à utiliser pour l’ID de la clé (keyId).

### <a name="to-configure-token-encryption-using-microsoft-graph"></a>Pour configurer le chiffrement des jetons à l’aide de Microsoft Graph

1. Mettez à jour la valeur `keyCredentials` de l’application avec un certificat X.509 pour le chiffrement. L’exemple suivant vous montre comment procéder.

    ```
    Patch https://graph.microsoft.com/beta/applications/<application objectid>

    { 
       "keyCredentials":[ 
          { 
             "type":"AsymmetricX509Cert","usage":"Encrypt",
             "keyId":"fdf8c5d8-f727-43fd-beaf-0f1521cf3d35",    (Use a GUID generator to obtain a value for the keyId)
             "key": "MIICADCCAW2gAwIBAgIQ5j9/b+n2Q4pDvQUCcy3…"  (Base64Encoded .cer file)
          }
        ]
    }
    ```

1. Identifiez le certificat de chiffrement qui est actif pour le chiffrement des jetons. L’exemple suivant vous montre comment procéder.

    ```
    Patch https://graph.microsoft.com/beta/applications/<application objectid> 

    { 
       "tokenEncryptionKeyId":"fdf8c5d8-f727-43fd-beaf-0f1521cf3d35" (The keyId of the keyCredentials entry to use)
    }
    ```

### <a name="to-configure-token-encryption-using-powershell"></a>Pour configurer le chiffrement des jetons à l’aide de PowerShell

1. Utilisez le dernier module Azure AD PowerShell pour vous connecter à votre locataire.

1. Définissez les paramètres de chiffrement de jeton à l’aide de la commande **[Set-AzureApplication](/powershell/module/azuread/set-azureadapplication?view=azureadps-2.0-preview)** .

    ```
    Set-AzureADApplication -ObjectId <ApplicationObjectId> -KeyCredentials "<KeyCredentialsObject>"  -TokenEncryptionKeyId <keyID>
    ```

1. Lisez les paramètres de chiffrement de jeton à l’aide des commandes suivantes.

    ```powershell
    $app=Get-AzureADApplication -ObjectId <ApplicationObjectId>
    $app.KeyCredentials
    $app.TokenEncryptionKeyId
    ```

### <a name="to-configure-token-encryption-using-the-application-manifest"></a>Pour configurer le chiffrement des jetons à l’aide du manifeste d’application

1. À partir du portail Azure, accédez à **Azure Active Directory > Inscriptions des applications**.

1. Sélectionnez **Toutes les applications** dans la liste déroulante pour afficher toutes les applications, puis sélectionnez l’application d’entreprise que vous voulez configurer.

1. Dans la page de l’application, sélectionnez **Manifeste** pour modifier le [manifeste d’application](../develop/reference-app-manifest.md).

1. Définissez la valeur pour l’attribut `tokenEncryptionKeyId`.

    L’exemple suivant illustre un manifeste d’application configuré avec deux certificats de chiffrement, le deuxième étant sélectionné comme celui étant actif à l’aide de tokenEnryptionKeyId.

    ```json
    { 
      "id": "3cca40e2-367e-45a5-8440-ed94edd6cc35",
      "accessTokenAcceptedVersion": null,
      "allowPublicClient": false,
      "appId": "cb2df8fb-63c4-4c35-bba5-3d659dd81bf1",
      "appRoles": [],
      "oauth2AllowUrlPathMatching": false,
      "createdDateTime": "2017-12-15T02:10:56Z",
      "groupMembershipClaims": "SecurityGroup",
      "informationalUrls": { 
         "termsOfService": null, 
         "support": null, 
         "privacy": null, 
         "marketing": null 
      },
      "identifierUris": [ 
        "https://testapp"
      ],
      "keyCredentials": [ 
        { 
          "customKeyIdentifier": "Tog/O1Hv1LtdsbPU5nPphbMduD=", 
          "endDate": "2039-12-31T23:59:59Z", 
          "keyId": "8be4cb65-59d9-404a-a6f5-3d3fb4030351", 
          "startDate": "2018-10-25T21:42:18Z", 
          "type": "AsymmetricX509Cert", 
          "usage": "Encrypt", 
          "value": <Base64EncodedKeyFile> 
          "displayName": "CN=SAMLEncryptTest" 
        }, 
        {
          "customKeyIdentifier": "U5nPphbMduDmr3c9Q3p0msqp6eEI=",
          "endDate": "2039-12-31T23:59:59Z", 
          "keyId": "6b9c6e80-d251-43f3-9910-9f1f0be2e851",
          "startDate": "2018-10-25T21:42:18Z", 
          "type": "AsymmetricX509Cert", 
          "usage": "Encrypt", 
          "value": <Base64EncodedKeyFile> 
          "displayName": "CN=SAMLEncryptTest2" 
        } 
      ], 
      "knownClientApplications": [], 
      "logoUrl": null, 
      "logoutUrl": null, 
      "name": "Test SAML Application", 
      "oauth2AllowIdTokenImplicitFlow": true, 
      "oauth2AllowImplicitFlow": false, 
      "oauth2Permissions": [], 
      "oauth2RequirePostResponse": false, 
      "orgRestrictions": [], 
      "parentalControlSettings": { 
         "countriesBlockedForMinors": [], 
         "legalAgeGroupRule": "Allow" 
        }, 
      "passwordCredentials": [], 
      "preAuthorizedApplications": [], 
      "publisherDomain": null, 
      "replyUrlsWithType": [], 
      "requiredResourceAccess": [], 
      "samlMetadataUrl": null, 
      "signInUrl": "https://127.0.0.1:444/applications/default.aspx?metadata=customappsso|ISV9.1|primary|z" 
      "signInAudience": "AzureADMyOrg",
      "tags": [], 
      "tokenEncryptionKeyId": "6b9c6e80-d251-43f3-9910-9f1f0be2e851" 
    }  
    ```

## <a name="next-steps"></a>Étapes suivantes

* Découvrir l’[utilisation du protocole SAML par Azure AD](../develop/active-directory-saml-protocol-reference.md)
* Découvrir le format, les caractéristiques de sécurité et le contenu des [jetons SAML dans Azure AD](../develop/reference-saml-tokens.md)