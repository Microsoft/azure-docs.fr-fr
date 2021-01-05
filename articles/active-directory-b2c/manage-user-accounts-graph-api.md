---
title: Gérer les utilisateurs avec l’API Microsoft Graph
titleSuffix: Azure AD B2C
description: Découvrez comment gérer les utilisateurs dans un locataire Azure AD B2C en appelant l’API Microsoft Graph et en utilisant une identité d’application pour automatiser le processus.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/03/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 6abc3316e18fc70a2969bc220fd75e10e10f0e6e
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507776"
---
# <a name="manage-azure-ad-b2c-user-accounts-with-microsoft-graph"></a>Gérer les comptes d’utilisateur Azure AD B2C avec Microsoft Graph

Microsoft Graph vous permet de gérer les comptes d’utilisateur dans votre annuaire Azure AD B2C en fournissant des méthodes de création, de lecture, de mise à jour et de suppression dans l’API Microsoft Graph. Vous pouvez migrer un magasin d’utilisateurs existant vers un locataire Azure AD B2C et effectuer d’autres opérations de gestion des comptes d’utilisateur en appelant l’API Microsoft Graph.

Dans les sections qui suivent, les principaux aspects de la gestion des utilisateurs Azure AD B2C avec l’API Microsoft Graph sont présentés. Les opérations, les types et les propriétés de l’API Microsoft Graph présentés ici constituent une partie de ce qui figure dans la documentation de référence sur l’API Microsoft Graph.

## <a name="register-a-management-application"></a>Inscrire une application de gestion

Avant qu’une application ou un script de gestion des utilisateurs que vous écrivez puisse interagir avec les ressources de votre locataire Azure AD B2C, vous avez besoin d’une inscription d’application qui accorde les autorisations nécessaires.

Suivez les étapes de cet article de procédure pour créer une inscription d’application que votre application de gestion peut utiliser :

[Gérer Azure AD B2C avec Microsoft Graph](microsoft-graph-get-started.md)

## <a name="user-management-microsoft-graph-operations"></a>Opérations Microsoft Graph pour la gestion des utilisateurs

Les opérations suivantes de gestion des utilisateurs sont disponibles dans l’[API Microsoft Graph](/graph/api/resources/user) :

- [Obtenir une liste d’utilisateurs](/graph/api/user-list)
- [Créer un utilisateur](/graph/api/user-post-users)
- [Obtenir un utilisateur](/graph/api/user-get)
- [Mettre à jour un utilisateur](/graph/api/user-update)
- [Suppression d’un utilisateur](/graph/api/user-delete)

## <a name="user-properties"></a>Propriétés de l’utilisateur

### <a name="display-name-property"></a>Propriété de nom d’affichage

`displayName` est le nom à afficher dans la gestion des utilisateurs du portail Azure pour l’utilisateur et dans le jeton d’accès qu’Azure AD B2C retourne à l’application. Cette propriété est requise.

### <a name="identities-property"></a>Propriété Identities

Un compte de client, qui peut être un consommateur, un partenaire ou un citoyen, peut être associé à ces types d’identité :

- Identité **locale** : Le nom d’utilisateur et le mot de passe sont stockés localement dans l’annuaire Azure AD B2C. Nous faisons souvent référence à ces identités en parlant de « comptes locaux ».
- Identités **fédérées** : Également appelées comptes *sociaux* ou d’*entreprise*. L’identité de l’utilisateur est managée par un fournisseur d’identité fédéré, comme Facebook, Microsoft, ADFS ou Salesforce.

Un utilisateur disposant d’un compte client peut se connecter avec plusieurs identités. Par exemple, le nom d’utilisateur, l’adresse e-mail, l’ID d’employé, l’ID national, etc. Un même compte peut avoir plusieurs identités, à la fois locales et de réseaux sociaux, avec le même mot de passe.

Dans l’API Microsoft Graph, les identités locales et fédérées sont stockées dans l’attribut `identities` de l’utilisateur, qui est de type [objectIdentity][graph-objectIdentity]. La collection `identities` représente un ensemble d’identités utilisées pour se connecter à un compte d’utilisateur. Cette collection permet à l’utilisateur de se connecter au compte d’utilisateur avec une de ses identités associées.

| Propriété   | Type |Description|
|:---------------|:--------|:----------|
|signInType|string| Spécifie les types de connexion utilisateur dans votre annuaire. Pour le compte local : `emailAddress`, `emailAddress1`, `emailAddress2`, `emailAddress3`, `userName` ou tout autre type de votre choix. Le compte social doit être défini sur `federated`.|
|émetteur|string|Spécifie l’émetteur de l’identité. Pour les comptes locaux (où  **signInType** n’est pas `federated`), cette propriété est le nom de domaine par défaut du locataire B2C local, par exemple `contoso.onmicrosoft.com`. Pour l’identité de réseaux sociaux (où **signInType** est `federated`), la valeur est le nom de l’émetteur, par exemple `facebook.com`|
|issuerAssignedId|string|Spécifie l’identificateur unique affecté à l’utilisateur par l’émetteur. La combinaison de **issuer** et **issuerAssignedId** doit être unique au sein de votre locataire. Pour un compte local, quand **signInType** est défini sur `emailAddress` ou sur `userName`, il représente le nom de connexion pour l’utilisateur.<br>Quand **signInType** est défini sur : <ul><li>`emailAddress` (ou commence par `emailAddress`, comme `emailAddress1`), **issuerAssignedId** doit être une adresse e-mail valide</li><li>`userName` (ou toute autre valeur), **issuerAssignedId** doit être une [partie locale valide d’une adresse e-mail](https://tools.ietf.org/html/rfc3696#section-3)</li><li>`federated`, **issuerAssignedId** représente l’identificateur unique du compte fédéré</li></ul>|

Propriété **Identities** avec une identité de compte local comprenant un nom de connexion, une adresse e-mail comme identifiant de connexion et une identité sociale. 

 ```json
 "identities": [
     {
       "signInType": "userName",
       "issuer": "contoso.onmicrosoft.com",
       "issuerAssignedId": "johnsmith"
     },
     {
       "signInType": "emailAddress",
       "issuer": "contoso.onmicrosoft.com",
       "issuerAssignedId": "jsmith@yahoo.com"
     },
     {
       "signInType": "federated",
       "issuer": "facebook.com",
       "issuerAssignedId": "5eecb0cd"
     }
   ]
 ```

Pour les identités fédérées, en fonction du fournisseur d’identité, **issuerAssignedId** est une valeur unique pour un utilisateur donné par application ou par compte de développement. Configurez la stratégie Azure AD B2C avec le même ID d’application que celui précédemment attribué par le fournisseur d’identité sociale ou une autre application au sein du même compte de développement.

### <a name="password-profile-property"></a>Propriété de profil de mot de passe

Pour une identité locale, la propriété **passwordProfile** est obligatoire et contient le mot de passe de l’utilisateur. La propriété `forceChangePasswordNextSignIn` doit être définie sur `false`.

Pour une identité (de réseaux sociaux) fédérée, la propriété **passwordProfile** n’est pas nécessaire.

```json
"passwordProfile" : {
    "password": "password-value",
    "forceChangePasswordNextSignIn": false
  }
```

### <a name="password-policy-property"></a>Propriété de stratégie de mot de passe

La stratégie de mot de passe Azure AD B2C (pour les comptes locaux) est basée sur la stratégie de [force de mot de passe fort](../active-directory/authentication/concept-sspr-policy.md) d’Azure Active Directory. Les stratégies de réinitialisation du mot de passe, d’inscription ou de connexion Azure AD B2C utilisent des mots de passe forts et qui n’expirent pas.

Dans les scénarios de migration d’utilisateurs, si les comptes que vous voulez migrer ont un mot de passe moins fort que les [règles de mot de passe fort](../active-directory/authentication/concept-sspr-policy.md) d’Azure AD B2C, vous pouvez désactiver l’exigence d’un mot de passe fort. Pour modifier la stratégie de mot de passe par défaut, définissez la propriété `passwordPolicies` sur `DisableStrongPassword`. Par exemple, vous pouvez modifier la demande de création d’utilisateur comme suit :

```json
"passwordPolicies": "DisablePasswordExpiration, DisableStrongPassword"
```

### <a name="extension-properties"></a>Propriétés d’extension

Toutes les applications orientées client ont des exigences uniques concernant les informations à recueillir. Votre locataire Azure AD B2C est fourni avec un ensemble intégré d’informations stockées dans des propriétés, comme le prénom, le nom, la localité et le code postal. Avec Azure AD B2C, vous pouvez étendre l’ensemble de propriétés stockées dans chaque compte client. Pour plus d'informations sur la définition d'attributs personnalisés, consultez [Attributs personnalisés](user-flow-custom-attributes.md).

L’API Microsoft Graph prend en charge la création et la mise à jour d’un utilisateur avec des attributs d’extension. Les attributs d’extension dans l’API Graph sont nommés d’après la convention `extension_ApplicationClientID_attributename`, où `ApplicationClientID` est l’**ID d’application (client)** de l’application `b2c-extensions-app` (accessible dans **Inscriptions d’applications** > **Toutes les applications** dans le portail Azure). Notez que l’**ID d’application (client)** tel qu’il est représenté dans le nom de l’attribut d’extension ne comprend aucun trait d’union. Par exemple :

```json
"extension_831374b3bd5041bfaa54263ec9e050fc_loyaltyNumber": "212342"
```

## <a name="code-sample-how-to-programmatically-manage-user-accounts"></a>Exemple de code : Comment gérer les comptes d’utilisateurs par programmation

Cet exemple de code est une application console .NET Core qui utilise le [SDK Microsoft Graph](/graph/sdks/sdks-overview) pour interagir avec l’API Microsoft Graph. Son code illustre comment appeler l’API pour gérer par programmation les utilisateurs dans un locataire Azure AD B2C.
Vous pouvez [télécharger l’archive de l’exemple](https://github.com/Azure-Samples/ms-identity-dotnetcore-b2c-account-management/archive/master.zip) (*.zip), [parcourir le dépôt](https://github.com/Azure-Samples/ms-identity-dotnetcore-b2c-account-management) sur GitHub ou cloner le dépôt :

```cmd
git clone https://github.com/Azure-Samples/ms-identity-dotnetcore-b2c-account-management.git
```

Une fois que vous avez obtenu l’exemple de code, configurez-le pour votre environnement, puis générez le projet :

1. Ouvrez le projet dans [Visual Studio](https://visualstudio.microsoft.com) ou dans [Visual Studio Code](https://code.visualstudio.com).
1. Ouvrez `src/appsettings.json`.
1. Dans la section `appSettings`, remplacez `your-b2c-tenant` par le nom de votre locataire, et `Application (client) ID` et `Client secret` par les valeurs de l’inscription de votre application de gestion (consultez la section [Inscrire une application de gestion](#register-a-management-application) de cet article).
1. Ouvrez une fenêtre de console dans votre clone local du dépôt, accédez au répertoire `src`, puis générez le projet :
    ```console
    cd src
    dotnet build
    ```
1. Exécutez l’application avec la commande `dotnet` :

    ```console
    dotnet bin/Debug/netcoreapp3.1/b2c-ms-graph.dll
    ```

L’application affiche une liste de commandes que vous pouvez exécuter. Par exemple, obtenir tous les utilisateurs, obtenir un seul utilisateur, supprimer un utilisateur, mettre à jour le mot de passe d’un utilisateur et importer en bloc.

### <a name="code-discussion"></a>Considérations sur le code

L’exemple de code utilise le kit [SDK Microsoft Graph](/graph/sdks/sdks-overview), qui est conçu pour simplifier la création d’applications qualitatives, efficaces et résilientes qui accèdent à Microsoft Graph.

Toute requête envoyée à l’API Microsoft Graph nécessite un jeton d’accès pour l’authentification. La solution utilise le package NuGet [Microsoft.Graph.Auth](https://www.nuget.org/packages/Microsoft.Graph.Auth/) qui fournit un wrapper basé sur un scénario d’authentification de la bibliothèque d’authentification Microsoft (MSAL) à utiliser avec le SDK Microsoft Graph.

La méthode `RunAsync` dans le fichier _Program.cs_ :

1. Lit les paramètres d’application à partir du fichier _appsettings.json_
1. Initialise le fournisseur d’authentification en utilisant le flux d’[octroi d’informations d’identification du client OAuth 2.0](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md). Avec le flux d’octroi d’informations d’identification du client, l’application est en mesure d’obtenir un jeton d’accès pour appeler l’API Microsoft Graph.
1. Configure le client du service Microsoft Graph avec le fournisseur d’authentification :

    ```csharp
    // Read application settings from appsettings.json (tenant ID, app ID, client secret, etc.)
    AppSettings config = AppSettingsFile.ReadFromJsonFile();

    // Initialize the client credential auth provider
    IConfidentialClientApplication confidentialClientApplication = ConfidentialClientApplicationBuilder
        .Create(config.AppId)
        .WithTenantId(config.TenantId)
        .WithClientSecret(config.ClientSecret)
        .Build();
    ClientCredentialProvider authProvider = new ClientCredentialProvider(confidentialClientApplication);

    // Set up the Microsoft Graph service client with client credentials
    GraphServiceClient graphClient = new GraphServiceClient(authProvider);
    ```

Le *GraphServiceClient* initialisée est ensuite utilisé dans _UserService.cs_ pour effectuer les opérations de gestion des utilisateurs. Par exemple, pour obtenir une liste des comptes d’utilisateur dans le locataire :

```csharp
public static async Task ListUsers(GraphServiceClient graphClient)
{
    Console.WriteLine("Getting list of users...");

    // Get all users (one page)
    var result = await graphClient.Users
        .Request()
        .Select(e => new
        {
            e.DisplayName,
            e.Id,
            e.Identities
        })
        .GetAsync();

    foreach (var user in result.CurrentPage)
    {
        Console.WriteLine(JsonConvert.SerializeObject(user));
    }
}
```

[Faire des appels d’API en utilisant le SDK Microsoft Graph](/graph/sdks/create-requests) contient des informations sur la façon de lire et d’écrire des informations à partir de Microsoft Graph, d’utiliser `$select` pour contrôler les propriétés retournées, de fournir des paramètres de requête personnalisés, et d’utiliser les paramètres de requête `$filter` et `$orderBy`.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir un index complet des opérations de l’API Microsoft Graph prises en charge pour les ressources Azure AD B2C, consultez [Opérations Microsoft Graph disponibles pour Azure AD B2C](microsoft-graph-operations.md).

<!-- LINK -->

[graph-objectIdentity]: /graph/api/resources/objectidentity
[graph-user]: (https://docs.microsoft.com/graph/api/resources/user)