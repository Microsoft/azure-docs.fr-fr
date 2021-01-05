---
title: Configurer le trousseau
titleSuffix: Microsoft identity platform
description: Découvrez comment configurer le trousseau afin que votre application puisse y mettre en cache des jetons.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 08/28/2019
ms.author: marsma
ms.reviewer: oldalton
ms.custom: aaddev
ms.openlocfilehash: 06e197a6e445c7dc1179be696318905f2132ee36
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85477734"
---
# <a name="configure-keychain"></a>Configurer le trousseau

Quand la bibliothèque [MSAL (Microsoft Authentication Library) pour iOS et macOS](msal-overview.md) se connecte à un utilisateur, ou actualise un jeton, elle tente de mettre en cache les jetons dans le trousseau. En mettant en cache les jetons dans le trousseau, MSAL peut fournir une authentification unique silencieuse entre plusieurs applications distribuées par le même développeur Apple. L’authentification unique est effectuée par le biais de la fonctionnalité de groupes d’accès au trousseau. Pour plus d’informations, consultez la [documentation sur les éléments de trousseau](https://developer.apple.com/documentation/security/keychain_services/keychain_items/sharing_access_to_keychain_items_among_a_collection_of_apps?language=objc) d’Apple.

Cet article explique comment configurer des droits d’application afin que MSAL puisse écrire des jetons mis en cache dans le trousseau iOS et macOS.

## <a name="default-keychain-access-group"></a>Groupe d’accès au trousseau par défaut

### <a name="ios"></a>iOS

MSAL sur iOS utilise le groupe d’accès `com.microsoft.adalcache` par défaut. Il s’agit du groupe d’accès partagé utilisé par les kits SDK MSAL et ADAL (Azure AD Authentication Library) et qui garantit la meilleure expérience d’authentification unique entre plusieurs applications du même éditeur.

Sur iOS, ajoutez le groupe de trousseau `com.microsoft.adalcache` au droit de votre application dans XCode sous **Paramètres du project** > **Capacités** > **Partage de trousseau**

### <a name="macos"></a>macOS

MSAL sur macOS utilise le groupe d’accès `com.microsoft.identity.universalstorage` par défaut.

En raison des limitations du trousseau macOS, l’`access group` de MSAL ne se traduit pas directement par l’attribut de groupe d’accès au trousseau (voir [kSecAttrAccessGroup](https://developer.apple.com/documentation/security/ksecattraccessgroup?language=objc)) sur macOS 10.14 et antérieur. Toutefois, il se comporte de la même manière du point de vue de l’authentification unique en s’assurant que plusieurs applications distribuées par le même développeur Apple peuvent bénéficier de l’authentification unique sans assistance.

Sur macOS 10.15 (macOS Catalina) et ultérieur, MSAL utilise l’attribut de groupe d’accès au trousseau pour obtenir une authentification unique silencieuse, de manière semblable à iOS.

## <a name="custom-keychain-access-group"></a>Groupe d’accès au trousseau personnalisé

Si vous souhaitez utiliser un autre groupe d’accès au trousseau, vous pouvez transmettre votre groupe personnalisé lors de la création de `MSALPublicClientApplicationConfig` avant de créer `MSALPublicClientApplication`, comme suit :

# <a name="objective-c"></a>[Objective-C](#tab/objc)

```objc
MSALPublicClientApplicationConfig *config = [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"your-client-id"
                                                                                            redirectUri:@"your-redirect-uri"
                                                                                              authority:nil];
    
config.cacheConfig.keychainSharingGroup = @"custom-group";
    
MSALPublicClientApplication *application = [[MSALPublicClientApplication alloc] initWithConfiguration:config error:nil];
    
// Now call acquiretoken. 
// Tokens will be saved into the "custom-group" access group
// and only shared with other applications declaring the same access group
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
let config = MSALPublicClientApplicationConfig(clientId: "your-client-id",
                                            redirectUri: "your-redirect-uri",
                                              authority: nil)
config.cacheConfig.keychainSharingGroup = "custom-group"
        
do {
  let application = try MSALPublicClientApplication(configuration: config)
  // continue on with application          
} catch let error as NSError {
  // handle error here
}       
```

---

## <a name="disable-keychain-sharing"></a>Désactiver le partage de trousseau

Si vous ne souhaitez pas partager l’état de l’authentification unique entre plusieurs applications, ou ne pas utiliser de groupe d’accès au trousseau, désactivez le partage de trousseau en transmettant l’ID de bundle d’applications comme keychainGroup :

# <a name="objective-c"></a>[Objective-C](#tab/objc)

```objc
config.cacheConfig.keychainSharingGroup = [[NSBundle mainBundle] bundleIdentifier];
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
if let bundleIdentifier = Bundle.main.bundleIdentifier {
    config.cacheConfig.keychainSharingGroup = bundleIdentifier
}
```

---

## <a name="handle--34018-error-failed-to-set-item-into-keychain"></a>Gérer l’erreur -34018 (échec de la définition de l’élément dans le trousseau)

L’erreur -34018 signifie normalement que le trousseau n’a pas été configuré correctement. Vérifiez que le groupe d’accès au trousseau qui a été configuré dans MSAL correspond à celui configuré dans les droits.

## <a name="ensure-your-application-is-properly-signed"></a>Vérifiez que votre application est signée correctement.

Sur macOS, les applications peuvent s’exécuter sans être signées par le développeur. Bien que la plupart des fonctionnalités de MSAL continueront à fonctionner, l’authentification unique par le biais du trousseau d’accès exige que l’application soit signée. Si vous êtes confronté à plusieurs invites de trousseau, vérifiez que la signature de votre application est valide.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur les groupes d’accès au trousseau dans l’article d’Apple intitulé [Sharing Access to Keychain Items Among a Collection of Apps](https://developer.apple.com/documentation/security/keychain_services/keychain_items/sharing_access_to_keychain_items_among_a_collection_of_apps?language=objc).
