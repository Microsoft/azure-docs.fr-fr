---
title: Utiliser des URI de redirection avec MSAL (iOS/macOS) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez les différences entre Microsoft Authentication Library pour ObjectiveC (MSAL pour iOS et macOS) et Azure AD Authentication Library pour ObjectiveC (ADAL.ObjC), et comment procéder à une migration entre elles.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 08/28/2019
ms.author: marsma
ms.reviewer: jak
ms.custom: aaddev
ms.openlocfilehash: 95bd7b5ac325ef5484bd01284c46489acb919a32
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85830347"
---
# <a name="using-redirect-uris-with-the-microsoft-authentication-library-for-ios-and-macos"></a>Utilisation des URI de redirection avec la bibliothèque d’authentification Microsoft pour iOS et macOS

Lorsqu’un utilisateur s’authentifie, Azure Active Directory (Azure AD) envoie le jeton à l’application par le biais de l’URI de redirection inscrit auprès de l’application Azure AD.

La bibliothèque d’authentification Microsoft (MSAL) exige que l’URI de redirection soit inscrit dans un format spécifique auprès de l’application Azure AD. MSAL utilise un URI de redirection par défaut, si vous n’en spécifiez aucun. Le format est `msauth.[Your_Bundle_Id]://auth`.

Le format URI de redirection par défaut fonctionne pour la plupart des applications et des scénarios, y compris pour l’authentification répartie et la vue web du système. Utilisez le format par défaut chaque fois que cela est possible.

Toutefois, vous devrez peut-être changer l’URI de redirection pour les scénarios avancés, comme décrit ci-dessous.

## <a name="scenarios-that-require-a-different-redirect-uri"></a>Scénarios nécessitant un URI de redirection différent

### <a name="cross-app-single-sign-on-sso"></a>Authentification unique (SSO) entre applications

Pour que la plateforme d’identités Microsoft partage des jetons entre les applications, chaque application doit disposer du même ID client ou ID d’application. Il s’agit de l’identificateur unique fourni lorsque vous avez inscrit votre application sur le portail (et non pas l’ID de bundle d’application que vous inscrivez pour chaque application auprès d’Apple).

Les URI de redirection doivent être différents pour chaque application iOS. Le service d’identité Microsoft peut ainsi identifier de façon unique les différentes applications qui partagent un ID d’application. Chaque application peut avoir plusieurs URI de redirection inscrits sur le portail Azure. Chacune des applications de votre suite présente un URI de redirection propre. Par exemple :

Si nous considérons l’inscription d’application suivante dans le portail Azure :

* ID client : `ABCDE-12345` (il s’agit d’un ID client unique)
* RedirectUris : `msauth.com.contoso.app1://auth`, `msauth.com.contoso.app2://auth`, `msauth.com.contoso.app3://auth`

App1 utilise la redirection `msauth.com.contoso.app1://auth`.\
App2 utilise `msauth.com.contoso.app2://auth`.\
App3 utilise `msauth.com.contoso.app1://auth`.

### <a name="migrating-from-adal-to-msal"></a>Migration d’ADAL vers MSAL

Lorsque vous avez migré vers MSAL du code qui utilisait la bibliothèque d’authentification Active Directory (ADAL), vous disposiez peut-être déjà d’un URI de redirection configuré pour votre application. Vous pouvez continuer à utiliser le même URI de redirection tant que votre application ADAL est configurée pour prendre en charge des scénarios répartis, et que votre URI de redirection répond aux exigences du format d’URI de redirection MSAL.

## <a name="msal-redirect-uri-format-requirements"></a>Conditions requises du format d’URI de redirection MSAL

* L’URI de redirection MSAL doit se présenter dans le format `<scheme>://host`

    selon lequel `<scheme>` est une chaîne unique qui identifie votre application. Elle est principalement basée sur l’identificateur de bundle de votre application pour garantir l’unicité. Par exemple, si l’ID de bundle de votre application est `com.contoso.myapp`, votre URI de redirection doit se présenter dans le format : `msauth.com.contoso.myapp://auth`.

    Si vous effectuez une migration à partir d’ADAL, votre URI de redirection aura probablement le format suivant : `<scheme>://[Your_Bundle_Id]`, dans lequel `scheme` est une chaîne unique. Ce format continue de fonctionner lorsque vous utilisez MSAL.

* `<scheme>` doit être inscrit dans le fichier Info.plist de votre application sous `CFBundleURLTypes > CFBundleURLSchemes`.  Dans cet exemple, Info.plist a été ouvert en tant que code source :

    ```xml
    <key>CFBundleURLTypes</key>
    <array>
        <dict>
            <key>CFBundleURLSchemes</key>
            <array>
                <string>msauth.[BUNDLE_ID]</string>
            </array>
        </dict>
    </array>
    ```

MSAL vérifie si votre URI de redirection s’inscrit correctement et retourne une erreur si ce n’est pas le cas.
    
* Si vous souhaitez utiliser des liens universels en tant qu’URI de redirection, `<scheme>` doit être `https` et n’a pas besoin d’être déclaré dans `CFBundleURLSchemes`. À la place, configurez l’application et le domaine suivant les instructions d’Apple qui figurent dans la page [Universal Links for Developers](https://developer.apple.com/ios/universal-links/) (Liens universels pour les développeurs) et appelez la méthode `handleMSALResponse:sourceApplication:` de `MSALPublicClientApplication` lorsque votre application est ouverte au moyen d’un lien universel.

## <a name="use-a-custom-redirect-uri"></a>Utiliser un URI de redirection personnalisé

Pour utiliser un URI de redirection personnalisé, transmettez le paramètre `redirectUri` à `MSALPublicClientApplicationConfig`, et cet objet à `MSALPublicClientApplication` lorsque vous initialisez l’objet. Si l’URI de redirection n’est pas valide, l’initialiseur retourne `nil` et définit `redirectURIError` à l’aide d’informations supplémentaires.  Par exemple :

Objective-C :

```objc
MSALPublicClientApplicationConfig *config =
        [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"your-client-id"
                                                        redirectUri:@"your-redirect-uri"
                                                        authority:authority];
NSError *redirectURIError;
MSALPublicClientApplication *application =
        [[MSALPublicClientApplication alloc] initWithConfiguration:config error:&redirectURIError];
```

Swift :

```swift
let config = MSALPublicClientApplicationConfig(clientId: "your-client-id",
                                            redirectUri: "your-redirect-uri",
                                              authority: authority)
do {
  let application = try MSALPublicClientApplication(configuration: config)
  // continue on with application          
} catch let error as NSError {
  // handle error here
}       
```



## <a name="handle-the-url-opened-event"></a>Gérer l’événement d’URL ouverte

Votre application doit appeler MSAL lorsqu’elle reçoit une réponse par le biais de modèles d’URL ou de liens universels. Appelez la méthode `handleMSALResponse:sourceApplication:` de `MSALPublicClientApplication` lorsque votre application est ouverte. Voici un exemple de schémas personnalisés :

Objective-C :

```objc
- (BOOL)application:(UIApplication *)app
            openURL:(NSURL *)url
            options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
{
    return [MSALPublicClientApplication handleMSALResponse:url 
                                         sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
}
```

Swift :

```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
}
```



## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [Flux d’authentification et scénarios d’applications](authentication-flows-app-scenarios.md)
