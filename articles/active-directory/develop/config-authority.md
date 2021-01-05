---
title: Configurer des fournisseurs d’identité (MSAL iOS/macOS) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment utiliser différentes autorités, telles que B2C, des clouds souverains et des utilisateurs invités, avec MSAL pour iOS et macOS.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 08/28/2019
ms.author: marsma
ms.reviewer: oldalton
ms.custom: aaddev
ms.openlocfilehash: d8a176fff0da932d0fafd40b9ab895b635acc5f6
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96169441"
---
# <a name="how-to-configure-msal-for-ios-and-macos-to-use-different-identity-providers"></a>Procédure : configurer MSAL pour iOS et macOS afin d’utiliser différents fournisseurs d’identité

Cet article montre comment configurer votre application MSAL (Microsoft Authentication Library) pour iOS et macOS pour différentes autorités, telles qu’Azure Active Directory (Azure AD), B2C (Business-to-Consumer), clouds souverains et utilisateurs invités.  Tout au long de cet article, vous pouvez généralement considérer une autorité comme étant un fournisseur d’identité.

## <a name="default-authority-configuration"></a>Configuration de l’autorité par défaut

`MSALPublicClientApplication` est configuré avec `https://login.microsoftonline.com/common` comme URL d’autorité par défaut, laquelle convient à la plupart des scénarios Azure Active Directory (AAD). À moins d’implémenter des scénarios avancés tels que des clouds nationaux ou d’utiliser B2C, vous n’aurez pas besoin de la changer.

> [!NOTE]
> L’authentification moderne avec les services de fédération Active Directory (AD FS) comme fournisseur d’identité n’est pas prise en charge (voir [Scénarios ADFS pour les développeurs](/windows-server/identity/ad-fs/overview/ad-fs-openid-connect-oauth-flows-scenarios) pour plus d’informations). AD FS est pris en charge par le biais de la fédération.

## <a name="change-the-default-authority"></a>Changer l’autorité par défaut

Dans certains scénarios, tels que B2C (Business-to-Consumer), vous devrez peut-être changer l’autorité par défaut.

### <a name="b2c"></a>B2C

Pour fonctionner avec B2C, [Microsoft Authentication Library (MSAL)](reference-v2-libraries.md) nécessite une configuration d’autorité différente. MSAL reconnaît un format d’URL d’autorité B2C par elle-même. Le format d’autorité B2C reconnu est `https://<host>/tfp/<tenant>/<policy>`, par exemple `https://login.microsoftonline.com/tfp/contoso.onmicrosoft.com/B2C_1_SignInPolicy`. Toutefois, vous pouvez également utiliser toute autre URL d’autorité B2C prise en charge en déclarant l’autorité en tant qu’autorité B2C de manière explicite.

Pour prendre en charge un format d’URL arbitraire pour B2C, `MSALB2CAuthority` peut être défini avec une URL arbitraire, comme suit :

Objective-C
```objc
NSURL *authorityURL = [NSURL URLWithString:@"arbitrary URL"];
MSALB2CAuthority *b2cAuthority = [[MSALB2CAuthority alloc] initWithURL:authorityURL
                                                                     error:&b2cAuthorityError];
```
Swift
```swift
guard let authorityURL = URL(string: "arbitrary URL") else {
    // Handle error
    return
}
let b2cAuthority = try MSALB2CAuthority(url: authorityURL)
```

Toutes les autorités B2C qui n’utilisent pas le format d’autorité B2C par défaut doivent être déclarées en tant qu’autorités connues.

Ajoutez chaque autorité B2C différente à la liste des autorités connues même si les autorités diffèrent uniquement en terme de stratégie.

Objective-C
```objc
MSALPublicClientApplicationConfig *b2cApplicationConfig = [[MSALPublicClientApplicationConfig alloc]
                                                               initWithClientId:@"your-client-id"
                                                               redirectUri:@"your-redirect-uri"
                                                               authority:b2cAuthority];
b2cApplicationConfig.knownAuthorities = @[b2cAuthority];
```
Swift
```swift
let b2cApplicationConfig = MSALPublicClientApplicationConfig(clientId: "your-client-id", redirectUri: "your-redirect-uri", authority: b2cAuthority)
b2cApplicationConfig.knownAuthorities = [b2cAuthority]
```

Quand votre application demande une nouvelle stratégie, l’URL d’autorité doit être changée, car elle est différente pour chaque stratégie. 

Pour configurer une application B2C, définissez `@property MSALAuthority *authority` avec une instance de `MSALB2CAuthority` dans `MSALPublicClientApplicationConfig` avant de créer `MSALPublicClientApplication`, comme suit :

Objective-C
```ObjC
    // Create B2C authority URL
    NSURL *authorityURL = [NSURL URLWithString:@"https://login.microsoftonline.com/tfp/contoso.onmicrosoft.com/B2C_1_SignInPolicy"];
    
    MSALB2CAuthority *b2cAuthority = [[MSALB2CAuthority alloc] initWithURL:authorityURL
                                                                     error:&b2cAuthorityError];
    if (!b2cAuthority)
    {
        // Handle error
        return;
    }
    
    // Create MSALPublicClientApplication configuration
    MSALPublicClientApplicationConfig *b2cApplicationConfig = [[MSALPublicClientApplicationConfig alloc]
                                                                   initWithClientId:@"your-client-id"
                                                                   redirectUri:@"your-redirect-uri"
                                                                   authority:b2cAuthority];

    // Initialize MSALPublicClientApplication
    MSALPublicClientApplication *b2cApplication =
    [[MSALPublicClientApplication alloc] initWithConfiguration:b2cApplicationConfig error:&error];
    
    if (!b2cApplication)
    {
        // Handle error
        return;
    }
```
Swift
```swift
do{
    // Create B2C authority URL
    guard let authorityURL = URL(string: "https://login.microsoftonline.com/tfp/contoso.onmicrosoft.com/B2C_1_SignInPolicy") else {
        // Handle error
        return
    }
    let b2cAuthority = try MSALB2CAuthority(url: authorityURL)

    // Create MSALPublicClientApplication configuration
    let b2cApplicationConfig = MSALPublicClientApplicationConfig(clientId: "your-client-id", redirectUri: "your-redirect-uri", authority: b2cAuthority)

    // Initialize MSALPublicClientApplication
    let b2cApplication = try MSALPublicClientApplication(configuration: b2cApplicationConfig)
} catch {
    // Handle error
}
```

### <a name="sovereign-clouds"></a>Clouds souverains

Si votre application s’exécute dans un cloud souverain, vous devrez peut-être changer l’URL d’autorité dans le `MSALPublicClientApplication`. L’exemple suivant définit l’URL d’autorité de façon à ce qu’elle fonctionne avec le cloud AAD allemand :

Objective-C
```objc
    NSURL *authorityURL = [NSURL URLWithString:@"https://login.microsoftonline.de/common"];
    MSALAuthority *sovereignAuthority = [MSALAuthority authorityWithURL:authorityURL error:&authorityError];
    
    if (!sovereignAuthority)
    {
        // Handle error
        return;
    }
    
    MSALPublicClientApplicationConfig *applicationConfig = [[MSALPublicClientApplicationConfig alloc]
                                                               initWithClientId:@"your-client-id"
                                                               redirectUri:@"your-redirect-uri"
                                                               authority:sovereignAuthority];
    
    
    MSALPublicClientApplication *sovereignApplication = [[MSALPublicClientApplication alloc] initWithConfiguration:applicationConfig error:&error];
    
    
    if (!sovereignApplication)
    {
        // Handle error
        return;
    }
```
Swift
```swift
do{
    guard let authorityURL = URL(string: "https://login.microsoftonline.de/common") else {
        //Handle error
        return
    }
    let sovereignAuthority = try MSALAuthority(url: authorityURL)
            
    let applicationConfig = MSALPublicClientApplicationConfig(clientId: "your-client-id", redirectUri: "your-redirect-uri", authority: sovereignAuthority)
            
    let sovereignApplication = try MSALPublicClientApplication(configuration: applicationConfig)
} catch {
    // Handle error
}
```

Vous devrez peut-être passer différentes étendues à chaque cloud souverain. Les étendues à envoyer dépendent de la ressource que vous utilisez. Par exemple, vous pourriez utiliser `"https://graph.microsoft.com/user.read"` dans le cloud mondial et `"https://graph.microsoft.de/user.read"` dans le cloud allemand.

### <a name="signing-a-user-into-a-specific-tenant"></a>Connexion d’un utilisateur à un locataire spécifique

Quand l’URL d’autorité est définie sur `"login.microsoftonline.com/common"`, l’utilisateur est connecté à son locataire d’hébergement. Toutefois, certaines applications peuvent avoir besoin de connecter l’utilisateur à un autre locataire, et certaines applications fonctionnent uniquement avec un seul locataire.

Pour connecter l’utilisateur à un locataire spécifique, configurez `MSALPublicClientApplication` avec une autorité spécifique. Par exemple :

`https://login.microsoftonline.com/469fdeb4-d4fd-4fde-991e-308a78e4bea4`

L’exemple suivant montre comment connecter un utilisateur à un locataire spécifique :

Objective-C
```objc
    NSURL *authorityURL = [NSURL URLWithString:@"https://login.microsoftonline.com/469fdeb4-d4fd-4fde-991e-308a78e4bea4"];
    MSALAADAuthority *tenantedAuthority = [[MSALAADAuthority alloc] initWithURL:authorityURL error:&authorityError];
    
    if (!tenantedAuthority)
    {
        // Handle error
        return;
    }
    
    MSALPublicClientApplicationConfig *applicationConfig = [[MSALPublicClientApplicationConfig alloc]
                                                               initWithClientId:@"your-client-id"
                                                               redirectUri:@"your-redirect-uri"
                                                               authority:tenantedAuthority];
    
    MSALPublicClientApplication *application =
    [[MSALPublicClientApplication alloc] initWithConfiguration:applicationConfig error:&error];
    
    if (!application)
    {
        // Handle error
        return;
    }
```
Swift
```swift
do{
    guard let authorityURL = URL(string: "https://login.microsoftonline.com/469fdeb4-d4fd-4fde-991e-308a78e4bea4") else {
        //Handle error
        return
    }    
    let tenantedAuthority = try MSALAADAuthority(url: authorityURL)
            
    let applicationConfig = MSALPublicClientApplicationConfig(clientId: "your-client-id", redirectUri: "your-redirect-uri", authority: tenantedAuthority)
            
    let application = try MSALPublicClientApplication(configuration: applicationConfig)
} catch {
    // Handle error
}
```

## <a name="supported-authorities"></a>Autorités prises en charge

### <a name="msalauthority"></a>MSALAuthority

La classe `MSALAuthority` est la classe abstraite de base pour les classes d’autorité MSAL. N’essayez pas de créer une instance de celle-ci en utilisant `alloc` ou `new`. Au lieu de cela, créez l’une de ses sous-classes directement (`MSALAADAuthority`, `MSALB2CAuthority`) ou utilisez la méthode de fabrique `authorityWithURL:error:` pour créer des sous-classes à l’aide d’une URL d’autorité.

Utilisez la propriété `url` pour obtenir une URL d’autorité normalisée. Les paramètres et les composants ou fragments de chemin supplémentaires qui ne font pas partie de l’autorité ne seront pas dans l’URL d’autorité normalisée retournée.

Voici les sous-classes de `MSALAuthority` que vous pouvez instancier en fonction de l’autorité que vous souhaitez utiliser.

### <a name="msalaadauthority"></a>MSALAADAuthority

`MSALAADAuthority` représente une autorité AAD. L’URL d’autorité doit être au format suivant, où `<port>` est facultatif : `https://<host>:<port>/<tenant>`

### <a name="msalb2cauthority"></a>MSALB2CAuthority

`MSALB2CAuthority` représente une autorité B2C. Par défaut, l’URL d’autorité B2C doit être au format suivant, où `<port>` est facultatif : `https://<host>:<port>/tfp/<tenant>/<policy>`. Toutefois, MSAL prend également en charge d’autres formats d’autorité B2C arbitraires.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [Flux d’authentification et scénarios d’applications](authentication-flows-app-scenarios.md)