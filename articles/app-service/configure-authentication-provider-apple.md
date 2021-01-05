---
title: Configurer Se connecter avec Apple (préversion)
description: Découvrez comment configurer l’authentification Se connecter avec Apple pour en faire un fournisseur d’identité pour votre application App Service ou Azure Functions.
ms.topic: article
ms.date: 11/19/2020
ms.reviewer: mikarmar
ms.openlocfilehash: b77e0613f502d003b5e4651e34be4cadbd4209a9
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602940"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-sign-in-using-a-sign-in-with-apple-provider-preview"></a>Configurer l’application App Service ou Azure Functions pour se connecter à l’aide d’un fournisseur Se connecter avec Apple (préversion)

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

Cet article vous montre comment configurer Azure App Service ou Azure Functions pour utiliser Se connecter avec Apple comme fournisseur d’authentification. 

Pour effectuer la procédure décrite dans cet article, vous devez être inscrit au programme de développement d’Apple. Pour vous inscrire au programme de développement d’Apple, rendez-vous à l’adresse [developer.apple.com/programs/enroll](https://developer.apple.com/programs/enroll/).

> [!CAUTION]
> L’activation de Se connecter avec Apple désactive la gestion de la fonctionnalité d’authentification/d’autorisation App Service pour votre application via certains clients, comme le portail Azure, Azure CLI et Azure PowerShell. La fonctionnalité s’appuie sur une nouvelle surface d’API qui, dans la préversion, n’est pas encore prise en compte dans toutes les expériences de gestion.

[comment]: <Remove this caution block when V2 becomes available for use.> 

## <a name="create-an-application-in-the-apple-developer-portal"></a><a name="createApplication"> </a>Créer une application dans le portail Apple Developer
Vous devez créer un ID d’application et un ID de service dans le portail Apple Developer.

1. Sur le portail Apple Developer, accédez à **Certificates, Identifiers & Profiles** (Certificats, identificateurs et profils).
2. Sous l’onglet **Identifiers** (Identificateurs), sélectionnez le bouton **(+)** .
3. Dans la page **Register a New Identifier** (Inscrire un nouvel identificateur), choisissez **App IDs** (ID d’application), puis cliquez sur **Continue** (Continuer). (Les ID d’application incluent un ou plusieurs ID de services.) ![Inscription d’un nouvel identificateur d’application dans le portail Apple Developer](media/configure-authentication-provider-apple/apple-register-app.jpg)
4. Sur la page **Register an App ID** (Inscrire un ID d’application), fournissez une description et un ID de pack, puis sélectionnez **Sign in with Apple** (Se connecter avec Apple) dans la liste des capacités. Sélectionnez **Continuer**. Prenez notre de votre **App ID Prefix (Team ID)** [préfixe d’ID d’application (ID d’équipe)] à cette étape, vous en aurez besoin plus tard.
![Configuration d’un nouvel identificateur d’application dans le portail Apple Developer](media/configure-authentication-provider-apple/apple-configure-app.jpg)
5. Passez en revue les informations d’inscription de l’application et sélectionnez **Register** (S’inscrire).
6. Ici encore, sous l’onglet **Identifiers** (Identificateurs), sélectionnez le bouton **(+)** .
![Création d’un nouvel identificateur de services dans le portail Apple Developer](media/configure-authentication-provider-apple/apple-new-app.jpg) 
7. Dans la page **Register a New Identifier** (Inscrire un nouvel identificateur), choisissez **Services IDs** (ID de services), puis cliquez sur **Continue** (Continuer).
![Inscription d’un nouvel identificateur de services dans le portail Apple Developer](media/configure-authentication-provider-apple/apple-register-service.jpg)
8. Sur la page **Register a Services ID** (Inscrire un ID de services), fournissez une description et un identificateur. La description correspond à ce qui sera montré à l’utilisateur sur l’écran de consentement. L’identificateur sera votre ID client utilisé pour configurer le fournisseur Apple avec votre application App Service. Ensuite, sélectionnez **Configurer**.
![Fournir une description et un identificateur](media/configure-authentication-provider-apple/apple-configure-service-1.jpg)
9. Dans la fenêtre contextuelle, définissez l’ID d’application principal sur l’ID d’application que vous avez créé précédemment. Spécifiez le domaine de votre application dans la section Domaines. Pour l’URL de retour, utilisez l’URL `<app-url>/.auth/login/apple/callback`. Par exemple : `https://contoso.azurewebsites.net/.auth/login/apple/callback`. Sélectionnez ensuite **Add** (Ajouter) et **Save** (Enregistrer).
![Spécification du domaine et de l’URL de retour pour l’inscription](media/configure-authentication-provider-apple/apple-configure-service-2.jpg)
10. Passez en revue les informations d’inscription du service et sélectionnez **Save** (Enregistrer).

## <a name="generate-the-client-secret"></a><a name="generateClientSecret"> </a>Générer la clé secrète client
Apple exige que les développeurs d’applications créent et signent un jeton JWT comme valeur de clé secrète client. Pour générer ce secret, commencez par générer et télécharger une clé privée à courbe elliptique à partir du portail Apple Developer. Utilisez ensuite cette clé pour [signer un JWT](#sign-the-client-secret-jwt) avec une [charge utile spécifique](#structure-the-client-secret-jwt).

### <a name="create-and-download-the-private-key"></a>Créer et télécharger la clé privée
1. Sous l’onglet **Keys** (Clés) du portail Apple Developer, choisissez **Create a key** (Créer une clé) ou sélectionnez le bouton **(+)** .
2. Dans la page **Inscrire une nouvelle clé**, donnez un nom à la clé, cochez la case à côté de **Se connecter avec Apple**, puis sélectionnez **Configurer**.
3. Sur la page **Configure Key** (Configurer la clé), liez la clé à l’ID d’application principal que vous avez créé précédemment et sélectionnez **Save** (Enregistrer).
4. Terminez la création de la clé en confirmant les informations et en sélectionnant **Continue**(Continuer), puis en vérifiant les informations et en sélectionnant **Register** (Enregistrer).
5. Sur la page **Download Your Key** (Télécharger votre clé), téléchargez la clé. Elle sera téléchargée sous la forme d’un fichier `.p8` (PKCS#8). Vous utiliserez le contenu du fichier pour signer le jeton JWT de la clé secrète client.

### <a name="structure-the-client-secret-jwt"></a>Structurer le jeton JWT de la clé secrète client
Apple exige que la clé secrète client soit l’encodage Base64 d’un jeton JWT. Le jeton JWT décodé doit avoir une charge utile structurée comme dans l’exemple suivant :
```json
{
  "alg": "ES256",
  "kid": "URKEYID001",
}.{
  "sub": "com.yourcompany.app1",
  "nbf": 1560203207,
  "exp": 1560289607,
  "iss": "ABC123DEFG",
  "aud": "https://appleid.apple.com"
}.[Signature]
```
- **sub** : L’ID client Apple (également l’ID de service)
- **iss** : Votre ID d’équipe Apple Developer
- **aud** : Apple reçoit le jeton, il s’agit donc du public visé
- **exp** : Pas plus de six mois après **nbf**

La version encodée en base64 de la charge utile ci-dessus ressemble à ceci : ```eyJhbGciOiJFUzI1NiIsImtpZCI6IlVSS0VZSUQwMDEifQ.eyJzdWIiOiJjb20ueW91cmNvbXBhbnkuYXBwMSIsIm5iZiI6MTU2MDIwMzIwNywiZXhwIjoxNTYwMjg5NjA3LCJpc3MiOiJBQkMxMjNERUZHIiwiYXVkIjoiaHR0cHM6Ly9hcHBsZWlkLmFwcGxlLmNvbSJ9.ABSXELWuTbgqfrIUz7bLi6nXvkXAz5O8vt0jB2dSHTQTib1x1DSP4__4UrlKI-pdzNg1sgeocolPNTmDKazO8-BHAZCsdeeTNlgFEzBytIpMKFfVEQbEtGRkam5IeclUK7S9oOva4EK4jV4VmgDrr-LGWWO3TaAxAvy3_ZoKohvFFkVG```

_Remarque : Apple n’accepte pas les jetons JWT de clé secrète client dont la date d’expiration se situe plus de six mois après la date de création (ou nbf). Cela signifie que vous devrez alterner votre clé secrète client au minimum tous les six mois._

Pour plus d’informations sur la génération et la validation des jetons, consultez la [documentation d’Apple destinée aux développeurs](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens). 

### <a name="sign-the-client-secret-jwt"></a>Signer le jeton JWT de la clé secrète client
Vous allez utiliser le fichier `.p8` téléchargé précédemment pour signer le JWT de la clé secrète client. Ce fichier est un [fichier PCKS#8](https://en.wikipedia.org/wiki/PKCS_8) qui contient la clé de signature privée au format PEM. De nombreuses bibliothèques peuvent créer et signer le JWT pour vous. 

Différents types de bibliothèques open source sont disponibles en ligne pour la création et la signature des jetons JWT. Pour plus d’informations sur la génération de jetons JWT, rendez-vous à l’adresse jwt.io. Par exemple, l’une des façons de générer la clé secrète client consiste à importer le [package NuGet Microsoft.IdentityModel.Tokens](https://www.nuget.org/packages/Microsoft.IdentityModel.Tokens/) et à exécuter une petite quantité de code C# indiquée ci-dessous.

```csharp
using Microsoft.IdentityModel.Tokens;

public static string GetAppleClientSecret(string teamId, string clientId, string keyId, string p8key)
{
    string audience = "https://appleid.apple.com";

    string issuer = teamId;
    string subject = clientId;
    string kid = keyId;

    IList<Claim> claims = new List<Claim> {
        new Claim ("sub", subject)
    };

    CngKey cngKey = CngKey.Import(Convert.FromBase64String(p8key), CngKeyBlobFormat.Pkcs8PrivateBlob);

    SigningCredentials signingCred = new SigningCredentials(
        new ECDsaSecurityKey(new ECDsaCng(cngKey)),
        SecurityAlgorithms.EcdsaSha256
    );

    JwtSecurityToken token = new JwtSecurityToken(
        issuer,
        audience,
        claims,
        DateTime.Now,
        DateTime.Now.AddDays(180),
        signingCred
    );
    token.Header.Add("kid", kid);
    token.Header.Remove("typ");

    JwtSecurityTokenHandler tokenHandler = new JwtSecurityTokenHandler();

    return tokenHandler.WriteToken(token);
}
```
- **teamId** : Votre ID d’équipe Apple Developer
- **clientId** : L’ID client Apple (également l’ID de service)
- **p8key** : Clé au format PEM. Vous pouvez obtenir la clé en ouvrant le fichier `.p8` dans un éditeur de texte et en copiant tout ce qui se trouve entre `-----BEGIN PRIVATE KEY-----` et `-----END PRIVATE KEY-----`, sans sauts de ligne.
- **keyId** : ID de la clé téléchargée

Ce jeton retourné est la valeur de la clé secrète client que vous allez utiliser pour configurer le fournisseur Apple.

> [!IMPORTANT]
> La clé secrète client est une information d'identification de sécurité importante. Ne partagez cette clé secrète avec personne et ne la distribuez pas dans une application cliente.
>

Ajoutez le secret client en tant que [paramètre d’application](./configure-common.md#configure-app-settings) pour l’application, en utilisant un nom de votre choix pour le paramètre. Prenez note de ce nom pour une utilisation ultérieure.

## <a name="add-provider-information-to-your-application"></a><a name="configure"> </a>Ajouter des informations sur le fournisseur à votre application

> [!NOTE]
> La configuration requise est dans un nouveau format d’API, actuellement pris en charge uniquement par la [configuration basée sur fichier (préversion)](.\app-service-authentication-how-to.md#config-file). Vous devrez suivre les étapes ci-dessous à l’aide d’un fichier de ce type.

Cette section vous guide tout au long de la mise à jour de la configuration pour inclure votre nouveau fournisseur d’identité. Voici un exemple de configuration.

1. Dans l’objet `identityProviders`, ajoutez un objet `apple` s’il n’en existe pas déjà un.
2. Attribuez un objet à cette clé avec un objet `registration` dans celui-ci, et éventuellement un objet `login` :
    
    ```json
    "apple" : {
       "registration" : {
            "clientId": "<client id>",
            "clientSecretSettingName": "APP_SETTING_CONTAINING_APPLE_CLIENT_SECRET" 
        },
       "login": {
             "scopes": []
       }
    }
    ```
    a. Dans l’objet `registration`, définissez le `clientId` sur l’ID client que vous avez collecté.
    
    b. Dans l’objet `registration`, définissez `clientSecretSettingName` sur le nom du paramètre d’application où vous avez stocké la clé secrète client.
    
    c. Dans l’objet `login`, vous pouvez choisir de définir le tableau `scopes` pour qu’il inclue une liste d’étendues utilisées lors de l’authentification auprès d’Apple, telles que « name » et « email ». Si des étendues sont configurées, elles sont demandées explicitement sur l’écran de consentement quand les utilisateurs se connectent pour la première fois.

Une fois cette configuration définie, vous êtes prêt à utiliser votre fournisseur Apple pour l’authentification dans votre application.

Une configuration complète peut ressembler à l’exemple suivant (où le paramètre APPLE_GENERATED_CLIENT_SECRET pointe vers un paramètre d’application contenant un JWT généré) :

```json
{
    "platform": {
        "enabled": true
    },
    "globalValidation": {
        "redirectToProvider": "apple",
        "unauthenticatedClientAction": "RedirectToLoginPage"
    },
    "identityProviders": {
        "apple": {
            "registration": {
                "clientId": "com.contoso.example.client",
                "clientSecretSettingName": "APPLE_GENERATED_CLIENT_SECRET"
            },
            "login": {
                "scopes": []
            }
        }
    },
    "login": {
        "tokenStore": {
            "enabled": true
        }
    }     
}
```

## <a name="next-steps"></a><a name="related-content"> </a>Étapes suivantes

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]
