---
title: Initialiser des applications clientes MSAL.NET | Azure
titleSuffix: Microsoft identity platform
description: En savoir plus sur l’initialisation d’applications clientes publiques et confidentielles à l’aide de la bibliothèque d’authentification Microsoft pour .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/18/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 5ec419be5c7549553788d009f09fa3e0fb8655e4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91258283"
---
# <a name="initialize-client-applications-using-msalnet"></a>Initialiser des applications clientes avec MSAL.NET
Cet article décrit l’initialisation d’applications clientes publiques et confidentielles à l’aide de la bibliothèque d’authentification Microsoft pour .NET (MSAL.NET).  Pour en savoir plus sur les types d’applications clientes et les options de configuration d’application, consultez la [vue d’ensemble](msal-client-applications.md).

Avec MSAL.NET 3.x, la méthode recommandée pour instancier une application consiste à utiliser les générateurs d’applications : `PublicClientApplicationBuilder` et `ConfidentialClientApplicationBuilder`. Ils offrent un mécanisme puissant permettant de configurer l’application à partir du code, ou d’un fichier de configuration ou même en combinant ces deux approches.

## <a name="prerequisites"></a>Prérequis
Avant d’initialiser une application, vous devez d’abord l’[inscrire](quickstart-register-app.md) afin que votre application puisse être intégrée à la plate-forme d’identité Microsoft.  Après l’inscription, vous aurez peut-être besoin des informations suivantes (que vous trouverez sur le Portail Azure) :

- L’ID client (une chaîne représentant un GUID)
- L’URL du fournisseur d’identité (l’instance) et l’audience de connexion pour votre application. Ces deux paramètres représentent collectivement l’autorité.
- L’ID locataire si vous écrivez une application métier uniquement pour votre organisation (également nommée application à locataire unique).
- Le secret d’application (chaîne secrète client) ou le certificat (de type X509Certificate2) s’il s’agit d’une application cliente confidentielle.
- Pour les applications web et parfois pour les applications clientes publiques (en particulier lorsque votre application doit utiliser un répartiteur), vous devez également avoir défini l’URI de redirection auquel le fournisseur d’identité recontactera votre application avec les jetons de sécurité.

## <a name="ways-to-initialize-applications"></a>Méthodes d’initialisation des applications
Il existe différentes manières d’instancier des applications clientes.

### <a name="initializing-a-public-client-application-from-code"></a>Initialisation d’une application cliente publique public à partir de code

Le code suivant instancie une application cliente publique en connectant les utilisateurs dans le cloud public Microsoft Azure, avec leur compte professionnel et scolaire ou leur compte Microsoft personnel.

```csharp
IPublicClientApplication app = PublicClientApplicationBuilder.Create(clientId)
    .Build();
```

### <a name="initializing-a-confidential-client-application-from-code"></a>Initialisation d’une application cliente confidentielle à partir de code

De la même manière, le code suivant instancie une application confidentielle (une application web se trouvant sur `https://myapp.azurewebsites.net`) en gérant les jetons des utilisateurs du cloud public Microsoft Azure, avec leur compte professionnel et scolaire ou leur compte Microsoft personnel. L’application est identifiée par le fournisseur d’identité en partageant une clé secrète client :

```csharp
string redirectUri = "https://myapp.azurewebsites.net";
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithClientSecret(clientSecret)
    .WithRedirectUri(redirectUri )
    .Build();
```

Comme vous le savez peut-être, en production, plutôt que d’utiliser une clé secrète client, vous souhaiterez peut-être partager un certificat avec Azure AD. Le code serait alors le suivant :

```csharp
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithCertificate(certificate)
    .WithRedirectUri(redirectUri )
    .Build();
```

### <a name="initializing-a-public-client-application-from-configuration-options"></a>Initialisation d’une application cliente publique public à partir d’options de configuration

Le code suivant instancie une application cliente publique à partir d’un objet de configuration, qui pourrait être renseigné par programmation ou lu à partir d’un fichier de configuration :

```csharp
PublicClientApplicationOptions options = GetOptions(); // your own method
IPublicClientApplication app = PublicClientApplicationBuilder.CreateWithApplicationOptions(options)
    .Build();
```

### <a name="initializing-a-confidential-client-application-from-configuration-options"></a>Initialisation d’une application cliente confidentielle à partir d’options de configuration

Le même type de modèle s’applique aux applications clientes confidentielles. Vous pouvez également ajouter d’autres paramètres à l’aide de modificateurs `.WithXXX` (ici un certificat).

```csharp
ConfidentialClientApplicationOptions options = GetOptions(); // your own method
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(options)
    .WithCertificate(certificate)
    .Build();
```

## <a name="builder-modifiers"></a>Modificateurs de générateur

Dans les extraits de code à l’aide de générateurs d’applications, un certain nombre de méthodes `.With` peuvent être appliquées en tant que modificateurs (par exemple, `.WithCertificate` et `.WithRedirectUri`). 

### <a name="modifiers-common-to-public-and-confidential-client-applications"></a>Modificateurs communs aux applications clientes publiques et confidentielles

Les modificateurs que vous pouvez définir sur un client public ou un générateur d’applications clientes confidentielles sont les suivants :

|Modificateur | Description|
|--------- | --------- |
|`.WithAuthority()` 7 remplace | Définit l’autorité par défaut de l’application sur une autorité Azure AD, avec la possibilité de choisir le cloud Azure, l’audience, le locataire (ID de locataire ou nom de domaine), ou de fournir directement l’URI de l’autorité.|
|`.WithAdfsAuthority(string)` | Définit l’autorité par défaut de l’application sur une autorité ADFS.|
|`.WithB2CAuthority(string)` | Définit l’autorité par défaut de l’application sur une autorité Azure AD B2C.|
|`.WithClientId(string)` | Remplace l’ID client.|
|`.WithComponent(string)` | Définit le nom de la bibliothèque à l’aide de MSAL.NET (pour des raisons de télémétrie). |
|`.WithDebugLoggingCallback()` | Si elle est appelée, l’application appelle `Debug.Write` en activant simplement les traces de débogage. Pour plus d’informations, voir [Journalisation](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/logging).|
|`.WithExtraQueryParameters(IDictionary<string,string> eqp)` | Définissez les paramètres de requête supplémentaires au niveau de l’application qui seront envoyés dans toutes les demandes d’authentification. Cela est substituable à chaque niveau de la méthode d’acquisition de jeton (avec le même `.WithExtraQueryParameters pattern`).|
|`.WithHttpClientFactory(IMsalHttpClientFactory httpClientFactory)` | Permet des scénarios avancés, tels que la configuration d’un proxy HTTP, ou forcer MSAL à utiliser un HttpClient particulier (par exemple, dans des applications web ASP.NET Core/API).|
|`.WithLogging()` | Si elle est appelée, l’application effectue un rappel à l’aide de traces de débogage. Pour plus d’informations, voir [Journalisation](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/logging).|
|`.WithRedirectUri(string redirectUri)` | Remplace l’URI de redirection par défaut. Pour des applications clientes publiques, cela sera utile pour les scénarios impliquant le répartiteur.|
|`.WithTelemetry(TelemetryCallback telemetryCallback)` | Définit le délégué utilisé pour envoyer la télémétrie.|
|`.WithTenantId(string tenantId)` | Remplace l’ID de locataire, ou la description du locataire.|

### <a name="modifiers-specific-to-xamarinios-applications"></a>Modificateurs spécifiques des applications Xamarin.iOS

Les modificateurs que vous pouvez définir sur un générateur d’applications clientes publiques sur Xamarin.iOS sont les suivants :

|Modificateur | Description|
|--------- | --------- |
|`.WithIosKeychainSecurityGroup()` | **Xamarin.iOS uniquement** : Définit le groupe de sécurité de la chaîne de clés iOS (pour la persistance du cache).|

### <a name="modifiers-specific-to-confidential-client-applications"></a>Modificateurs spécifiques aux applications clientes confidentielles

Les modificateurs que vous pouvez définir sur un générateur d’applications clientes confidentielles sont les suivants :

|Modificateur | Description|
|--------- | --------- |
|`.WithCertificate(X509Certificate2 certificate)` | Définit le certificat identifiant l’application avec Azure AD.|
|`.WithClientSecret(string clientSecret)` | Définit la clé secrète client (mot de passe de l’application) identifiant l’application avec Azure AD.|

Ces modificateurs s’excluent mutuellement. Si vous fournissez les deux, MSAL génère une exception explicite.

### <a name="example-of-usage-of-modifiers"></a>Exemple d’utilisation des modificateurs

Supposons qu’il s’agisse d’une application coeur de métier dédiée uniquement à votre organisation.  Vous pouvez écrire ce qui suit :

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
        .Build();
```

Le fait que la programmation des clouds nationaux est désormais simplifiée est intéressant. Si vous souhaitez que votre application soit mutualisée dans un cloud national, vous pouvez écrire, par exemple :

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAuthority(AzureCloudInstance.AzureUsGovernment, AadAuthorityAudience.AzureAdMultipleOrgs)
        .Build();
```

Il existe également un remplacement pour ADFS (ADFS 2019 n’est pas pris en charge actuellement) :
```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAdfsAuthority("https://consoso.com/adfs")
        .Build();
```

Enfin, si vous êtes un développeur Azure AD B2C, vous pouvez spécifier votre client comme suit :

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithB2CAuthority("https://fabrikamb2c.b2clogin.com/tfp/{tenant}/{PolicySignInSignUp}")
        .Build();
```
