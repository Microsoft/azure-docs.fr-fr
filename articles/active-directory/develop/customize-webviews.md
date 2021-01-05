---
title: Personnaliser les navigateurs et les vues web (MSAL iOS/macOS) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment personnaliser l’expérience de navigateur iOS/macOS MSAL pour la connexion des utilisateurs.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 08/28/2019
ms.author: marsma
ms.reviewer: oldalton
ms.custom: aaddev
ms.openlocfilehash: a8486ec87b5198231a33b1dab382ba457c8c8066
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85478125"
---
# <a name="how-to-customize-browsers-and-webviews-for-iosmacos"></a>Procédure : personnaliser les navigateurs et les vues web pour iOS/macOS

Un navigateur web est nécessaire pour l’authentification interactive. Sur iOS et macOS 10.15+, la bibliothèque MSAL (Microsoft Authentication Library) utilise le navigateur web système par défaut (qui peut apparaître au-dessus de votre application) afin d’effectuer une authentification interactive pour la connexion des utilisateurs. L’utilisation du navigateur système présente l’avantage de partager l’état d’authentification unique (SSO) avec d’autres applications et avec les applications web.

Vous pouvez changer l’expérience en personnalisant la configuration avec d’autres options d’affichage du contenu web, telles que :

Pour iOS uniquement :

- [SFAuthenticationSession](https://developer.apple.com/documentation/safariservices/sfauthenticationsession?language=objc) 
- [SFSafariViewController](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller?language=objc)

Pour iOS et macOS :

- [ASWebAuthenticationSession](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession?language=objc)
- [WKWebView](https://developer.apple.com/documentation/webkit/wkwebview?language=objc).

MSAL pour macOS prend uniquement en charge `WKWebView` sur les versions plus anciennes du système d’exploitation. `ASWebAuthenticationSession` est uniquement pris en charge sur macOS 10,15 et versions ultérieures. 

## <a name="system-browsers"></a>Navigateurs système

Pour iOS, `ASWebAuthenticationSession`, `SFAuthenticationSession` et `SFSafariViewController` sont considérés comme des navigateurs système. Pour macOS, seul `ASWebAuthenticationSession` est disponible. En général, les navigateurs système partagent des cookies et d’autres données de sites web avec l’application de navigateur Safari.

Par défaut, MSAL détecte de manière dynamique la version d’iOS et sélectionne le navigateur système recommandé disponible sur cette version. Sur iOS 12 +, il s’agit de `ASWebAuthenticationSession`. 

### <a name="default-configuration-for-ios"></a>Configuration par défaut pour iOS

| Version | un navigateur Web |
|:-------------:|:-------------:|
| iOS 12+ | ASWebAuthenticationSession |
| iOS 11 | SFAuthenticationSession |
| iOS 10 | SFSafariViewController |

### <a name="default-configuration-for-macos"></a>Configuration par défaut pour macOS

| Version | un navigateur Web |
|:-------------:|:-------------:|
| macOS 10.15+ | ASWebAuthenticationSession |
| autres versions | WKWebView |

Les développeurs peuvent également sélectionner un autre navigateur système pour les applications MSAL :

- `SFAuthenticationSession` est la version iOS 11 de `ASWebAuthenticationSession`.
- `SFSafariViewController` est plus général, fournit une interface pour naviguer sur le web et peut également être utilisé à des fins de connexion. Dans iOS 9 et 10, les cookies et autres données de sites web sont partagés avec Safari, mais pas dans iOS 11 et ultérieur.

## <a name="in-app-browser"></a>Navigateur dans l’application

[WKWebView](https://developer.apple.com/documentation/webkit/wkwebview) est un navigateur dans l’application qui affiche le contenu web. Il ne partage pas de cookies ou de données de sites web avec d’autres instances de **WKWebView**, ni avec le navigateur Safari. WKWebView est un navigateur multiplateforme qui est disponible pour iOS et macOS.

## <a name="cookie-sharing-and-single-sign-on-sso-implications"></a>Implications en termes de partage des cookies et d’authentification unique

Le navigateur que vous utilisez a un impact sur l’expérience d’authentification unique, en raison de la façon dont il partage les cookies. Les tableaux suivants récapitulent les expériences d’authentification unique par navigateur.

| Technology    | Type de navigateur  | Disponibilité iOS | Disponibilité macOS | Partage des cookies et d’autres données  | Disponibilité MSAL | SSO |
|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|-------------:|
| [ASWebAuthenticationSession](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession) | Système | iOS12 et ultérieur | macOS 10.15 et ultérieur | Oui | iOS et macOS 10.15+ | avec instances de Safari
| [SFAuthenticationSession](https://developer.apple.com/documentation/safariservices/sfauthenticationsession) | Système | iOS11 et ultérieur | N/A | Oui | iOS uniquement |  avec instances de Safari
| [SFSafariViewController](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller) | Système | iOS11 et ultérieur | N/A | Non | iOS uniquement | Non**
| **SFSafariViewController** | Système | iOS10 | N/A | Oui | iOS uniquement |  avec instances de Safari
| **WKWebView**  | Dans l’application | iOS8 et ultérieur | macOS 10.10 et ultérieur | Non | iOS et macOS | Non**

** Pour que l’authentification unique fonctionne, les jetons doivent être partagés entre les applications. Cela nécessite un cache de jetons ou une application répartiteur, comme Microsoft Authenticator pour iOS.

## <a name="change-the-default-browser-for-the-request"></a>Changer le navigateur par défaut pour la requête

Vous pouvez utiliser un navigateur dans l’application ou un navigateur système spécifique en fonction de vos besoins d’expérience utilisateur, en changeant la propriété suivante dans `MSALWebviewParameters` :

```objc
@property (nonatomic) MSALWebviewType webviewType;
```

## <a name="change-per-interactive-request"></a>Changement interactif en fonction de la requête

Chaque requête peut être configurée pour remplacer le navigateur par défaut en changeant la propriété `MSALInteractiveTokenParameters.webviewParameters.webviewType` avant de la transmettre à l’API `acquireTokenWithParameters:completionBlock:`.

De plus, MSAL prend en charge la transmission d’un `WKWebView` personnalisé en définissant la propriété `MSALInteractiveTokenParameters.webviewParameters.customWebView`.

Par exemple :

Objective-C
```objc
UIViewController *myParentController = ...;
WKWebView *myCustomWebView = ...;
MSALWebviewParameters *webViewParameters = [[MSALWebviewParameters alloc] initWithAuthPresentationViewController:myParentController];
webViewParameters.webviewType = MSALWebviewTypeWKWebView;
webViewParameters.customWebview = myCustomWebView;
MSALInteractiveTokenParameters *interactiveParameters = [[MSALInteractiveTokenParameters alloc] initWithScopes:@[@"myscope"] webviewParameters:webViewParameters];
    
[app acquireTokenWithParameters:interactiveParameters completionBlock:completionBlock];
```
Swift
```swift
let myParentController: UIViewController = ...
let myCustomWebView: WKWebView = ...
let webViewParameters = MSALWebviewParameters(authPresentationViewController: myParentController)
webViewParameters.webviewType = MSALWebviewType.wkWebView
webViewParameters.customWebview = myCustomWebView
let interactiveParameters = MSALInteractiveTokenParameters(scopes: ["myscope"], webviewParameters: webViewParameters)

app.acquireToken(with: interactiveParameters, completionBlock: completionBlock)
```

Si vous utilisez une vue web personnalisée, des notifications sont utilisées pour indiquer l’état du contenu web affiché, par exemple :

```objc
/*! Fired at the start of a resource load in the webview. The URL of the load, if available, will be in the @"url" key in the userInfo dictionary */
extern NSString *MSALWebAuthDidStartLoadNotification;

/*! Fired when a resource finishes loading in the webview. */
extern NSString *MSALWebAuthDidFinishLoadNotification;

/*! Fired when web authentication fails due to reasons originating from the network. Look at the @"error" key in the userInfo dictionary for more details.*/
extern NSString *MSALWebAuthDidFailNotification;

/*! Fired when authentication finishes */
extern NSString *MSALWebAuthDidCompleteNotification;

/*! Fired before ADAL invokes the broker app */
extern NSString *MSALWebAuthWillSwitchToBrokerApp;
```

### <a name="options"></a>Options

Tous les types de navigateurs web pris en charge par MSAL sont déclarés dans l’[énumération MSALWebviewType](https://github.com/AzureAD/microsoft-authentication-library-for-objc/blob/master/MSAL/src/public/MSALDefinitions.h#L47).

```objc
typedef NS_ENUM(NSInteger, MSALWebviewType)
{
    /**
     For iOS 11 and up, uses AuthenticationSession (ASWebAuthenticationSession or SFAuthenticationSession).
     For older versions, with AuthenticationSession not being available, uses SafariViewController.
     For macOS 10.15 and above uses ASWebAuthenticationSession
     For older macOS versions uses WKWebView
     */
    MSALWebviewTypeDefault,
    
    /** Use ASWebAuthenticationSession where available.
     On older iOS versions uses SFAuthenticationSession
     Doesn't allow any other webview type, so if either of these are not present, fails the request*/
    MSALWebviewTypeAuthenticationSession,
    
#if TARGET_OS_IPHONE
    
    /** Use SFSafariViewController for all versions. */
    MSALWebviewTypeSafariViewController,
    
#endif
    /** Use WKWebView */
    MSALWebviewTypeWKWebView,
};
```

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [Flux d’authentification et scénarios d’applications](authentication-flows-app-scenarios.md)
