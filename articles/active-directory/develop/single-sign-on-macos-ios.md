---
title: Configurer l’authentification unique sur macOS et iOS
titleSuffix: Microsoft identity platform
description: Découvrez comment configurer l’authentification unique sur macOS et iOS.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 02/03/2020
ms.author: marsma
ms.reviewer: ''
ms.custom: aaddev
ms.openlocfilehash: 25389348476552298ddb947ccb59acb8b3d5bc57
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80881246"
---
# <a name="how-to-configure-sso-on-macos-and-ios"></a>Procédure : Configurer l’authentification unique sur macOS et iOS

Microsoft Authentication Library (MSAL) pour macOS et iOS prend en charge l’authentification unique entre les applications et les navigateurs macOS/iOS. Cet article aborde les scénarios d’authentification unique suivants :

- [Authentification unique silencieuse entre plusieurs applications](#silent-sso-between-apps)

Ce type d’authentification unique fonctionne entre plusieurs applications distribuées par le même développeur Apple. Elle fournit une authentification unique silencieuse (c’est-à-dire que l’utilisateur n’est pas invité à fournir des informations d’identification) en lisant des jetons d’actualisation écrits par d’autres applications à partir du trousseau et en les échangeant contre des jetons d’accès en mode silencieux.  

- [Authentification unique par le biais d’un répartiteur d’authentification](#sso-through-authentication-broker-on-ios)

> [!IMPORTANT]
> Ce flux n’est pas disponible sur macOS.

Microsoft fournit des applications, appelées répartiteurs, qui activent l’authentification unique entre les applications de différents fournisseurs tant que l’appareil mobile est inscrit auprès d’Azure Active Directory (AAD). Ce type d’authentification unique nécessite l’installation d’une application de répartition sur l’appareil de l’utilisateur.

- **Authentification unique entre MSAL et Safari**

L’authentification unique est effectuée par le biais de la classe [ASWebAuthenticationSession](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession?language=objc). Elle utilise l’état de connexion existant à partir d’autres applications et du navigateur Safari. Elle n’est pas limitée aux applications distribuées par le même développeur Apple, mais elle nécessite une interaction de l’utilisateur.

Si vous utilisez la vue web par défaut dans votre application pour connecter des utilisateurs, vous aurez une authentification unique automatique entre les applications basées sur MSAL et Safari. Pour en savoir plus sur les vues web pris en charge par MSAL, consultez [Personnaliser les navigateurs et les vues web](customize-webviews.md).

> [!IMPORTANT]
> Ce type d’authentification unique n’est actuellement pas disponible sur macOS. MSAL sur macOS prend uniquement en charge WKWebView, qui n’offre pas de prise en charge de l’authentification unique avec Safari. 

- **Authentification unique silencieuse entre les applications macOS/iOS ADAL et MSAL**

MSAL Objective-C prend en charge la migration et l’authentification unique avec les applications ADAL Objective-C. Les applications doivent être distribuées par le même développeur Apple.

Pour des instructions sur l’authentification unique entre les applications ADAL et MSAL, consultez [Authentification unique entre les applications ADAL et MSAL sur macOS et iOS](sso-between-adal-msal-apps-macos-ios.md).

## <a name="silent-sso-between-apps"></a>Authentification unique silencieuse entre les applications

MSAL prend en charge le partage de l’authentification unique par le biais des groupes d’accès au trousseau iOS.

Pour activer l’authentification unique parmi vos applications, vous devez effectuer les étapes suivantes, qui sont expliquées plus en détail ci-dessous :

1. Vérifiez que l’ensemble de vos applications utilisent le même ID client ou ID d’application.
1. Vérifiez que l’ensemble de vos applications partagent le même certificat de signature fourni par Apple, de manière à ce que vous puissiez partager les trousseaux.
1. Demandez la même éligibilité de trousseau pour l’ensemble de vos applications.
1. Signalez aux SDK MSAL le trousseau partagé que vous souhaitez que nous utilisions s’il est différent du trousseau par défaut.

### <a name="use-the-same-client-id-and-application-id"></a>Utiliser les mêmes ID client et ID d’application

Pour que la plateforme d’identités Microsoft sache quelles applications peuvent partager des jetons, ces applications doivent partager le même ID de client ou d’application. Il s’agit de l’identificateur unique qui vous a été fourni lorsque vous avez inscrit votre première application dans le portail.

La plateforme d’identités Microsoft distingue les applications qui utilisent le même ID d’application par leurs **URI de redirection**. Chaque application peut posséder plusieurs URI de redirection inscrits sur le portail d’intégration. Chacune des applications de votre suite présente un URI de redirection propre. Par exemple :

URI de redirection App1 : `msauth.com.contoso.mytestapp1://auth`  
URI de redirection App2 : `msauth.com.contoso.mytestapp2://auth`  
URI de redirection App3 : `msauth.com.contoso.mytestapp3://auth`  

> [!IMPORTANT]
> Le format des URI de redirection doit être compatible avec le format pris en charge par MSAL, qui est documenté dans [Spécifications du format d’URI de redirection MSAL](redirect-uris-ios.md#msal-redirect-uri-format-requirements).

### <a name="setup-keychain-sharing-between-applications"></a>Créer le partage de trousseau entre les applications

Pour activer le partage de trousseau, consultez l’article d’Apple sur l’[ajout de fonctionnalités](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AddingCapabilities/AddingCapabilities.html). L’essentiel est de déterminer le nom que vous souhaitez attribuer à votre trousseau et d’ajouter cette fonctionnalité à toutes vos applications qui seront concernées par l’authentification unique.

Une fois les droits configurés correctement, vous verrez un fichier `entitlements.plist` dans le répertoire de votre projet qui contient du code semblable à celui-ci :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>keychain-access-groups</key>
    <array>
        <string>$(AppIdentifierPrefix)com.myapp.mytestapp</string>
        <string>$(AppIdentifierPrefix)com.myapp.mycache</string>
    </array>
</dict>
</plist>
```

#### <a name="add-a-new-keychain-group"></a>Ajouter un nouveau groupe de trousseaux

Ajoutez un nouveau groupe de trousseaux aux **fonctionnalités** de votre projet. Le groupe de trousseaux doit être :
* `com.microsoft.adalcache` sur iOS 
* `com.microsoft.identity.universalstorage` sur macOS

![exemple de trousseau](media/single-sign-on-macos-ios/keychain-example.png)

Pour plus d’informations, consultez [Groupes de trousseaux](howto-v2-keychain-objc.md).

## <a name="configure-the-application-object"></a>Configurer l’objet d’application

Une fois que vous avez activé le droit au trousseau dans chacune de vos applications et que vous êtes prêt à utiliser l’authentification unique, configurez `MSALPublicClientApplication` avec votre groupe d’accès au trousseau comme dans l’exemple suivant :

Objective-C :

```objc
NSError *error = nil;
MSALPublicClientApplicationConfig *configuration = [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<my-client-id>"];
configuration.cacheConfig.keychainSharingGroup = @"my.keychain.group";
    
MSALPublicClientApplication *application = [[MSALPublicClientApplication alloc] initWithConfiguration:configuration error:&error];
```

Swift :

```swift
let config = MSALPublicClientApplicationConfig(clientId: "<my-client-id>")
config.cacheConfig.keychainSharingGroup = "my.keychain.group"

do {
   let application = try MSALPublicClientApplication(configuration: config)
  // continue on with application
} catch let error as NSError {
  // handle error here
}
```

> [!WARNING]
> Quand vous partagez un trousseau entre vos applications, n’importe quelle application peut supprimer des utilisateurs ou même l’ensemble des jetons de votre application.
> Cela aura un impact particulièrement sévère si vous avez des applications qui s’appuient sur les jetons pour exécuter des tâches d’arrière-plan.
> Le partage du trousseau demande de votre part une grande précaution quand votre application utilise les opérations de suppression du SDK Microsoft Identity.

Et voilà ! Le SDK partage désormais les informations d’identification dans l’ensemble de vos applications. La liste des comptes sera également partagée entre toutes les instances d’application.

## <a name="sso-through-authentication-broker-on-ios"></a>Authentification unique par le biais d’un répartiteur d’authentification sur iOS

MSAL fournit la prise en charge de l’authentification répartie avec Microsoft Authenticator. Microsoft Authenticator fournit l’authentification unique pour les appareils inscrits auprès d’AAD, et aide également votre application à respecter les stratégies d’accès conditionnel.

Les étapes suivantes expliquent comment activer l’authentification unique à l’aide d’un répartiteur d’authentification pour votre application :

1. Inscrivez un format d’URI de redirection compatible avec le répartiteur pour l’application dans le fichier Info.plist de votre application. Le format d’URI de redirection compatible avec le répartiteur est `msauth.<app.bundle.id>://auth`. Remplacez « <app.bundle.id> » par l’ID de bundle de votre application. Par exemple :

    ```xml
    <key>CFBundleURLSchemes</key>
    <array>
        <string>msauth.<app.bundle.id></string>
    </array>
    ```

1. Ajoutez les schémas suivants au fichier Info.plist de votre application sous `LSApplicationQueriesSchemes` :

    ```xml
    <key>LSApplicationQueriesSchemes</key>
    <array>
         <string>msauthv2</string>
         <string>msauthv3</string>
    </array>
    ```

1. Ajoutez le code suivant à votre fichier `AppDelegate.m` pour gérer les rappels :

    Objective-C :
    
    ```objc
    - (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *,id> *)options
    {
        return [MSALPublicClientApplication handleMSALResponse:url sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
    }
    ```
    
    Swift :
    
    ```swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
    }
    ```
    
**Si vous utilisez Xcode 11**, vous devez placer le rappel MSAL dans le fichier `SceneDelegate`.
Si vous prenez en charge à la fois UISceneDelegate et UIApplicationDelegate pour assurer la compatibilité avec une version plus ancienne d’iOS, le rappel MSAL doit être placé dans les deux fichiers.

Objective-C :

```objc
 - (void)scene:(UIScene *)scene openURLContexts:(NSSet<UIOpenURLContext *> *)URLContexts
 {
     UIOpenURLContext *context = URLContexts.anyObject;
     NSURL *url = context.URL;
     NSString *sourceApplication = context.options.sourceApplication;
     
     [MSALPublicClientApplication handleMSALResponse:url sourceApplication:sourceApplication];
 }
```

Swift :

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        
        guard let urlContext = URLContexts.first else {
            return
        }
        
        let url = urlContext.url
        let sourceApp = urlContext.options.sourceApplication
        
        MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApp)
    }
```

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [Flux d’authentification et scénarios d’applications](authentication-flows-app-scenarios.md)