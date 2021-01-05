---
title: Configurer des applications mobiles appelant des API web | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment configurer le code de votre application mobile pour appeler une API Web
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 06/16/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: a18a36b8583f8534b2a2e643e5c155dc7a2d65e2
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94444059"
---
# <a name="configure-a-mobile-app-that-calls-web-apis"></a>Configurer une application mobile qui appelle des API web

Après avoir créé votre application, vous allez apprendre à configurer le code à l’aide des paramètres d’inscription de l’application. Les applications mobiles présentent quelques complexités liées à l’ajustement dans leur infrastructure de création.

## <a name="find-msal-support-for-mobile-apps"></a>Rechercher la prise en charge de MSAL pour les applications mobiles

Les types MSAL (Microsoft Authentication Library) suivants prennent en charge les applications mobiles.

MSAL | Description
------------ | ----------
![MSAL.NET](media/sample-v2-code/logo_NET.png) <br/> MSAL.NET  | Utilisé pour développer des applications portables. MSAL.NET prend en charge les plateformes suivantes pour la création d’une application mobile : Plateforme Windows universelle (UWP), Xamarin.iOS et Xamarin.Android.
![MSAL.iOS](media/sample-v2-code/logo_iOS.png) <br/> MSAL.iOS | Utilisé pour développer des applications iOS natives avec Objective-C ou Swift.
![MSAL.Android](media/sample-v2-code/logo_android.png) <br/> MSAL.Android | Utilisé pour développer des applications Android natives dans Java pour Android.

## <a name="instantiate-the-application"></a>Instancier l’application

### <a name="android"></a>Android

Les applications mobiles utilisent la classe `PublicClientApplication`. Voici comment l’instancier :

```Java
PublicClientApplication sampleApp = new PublicClientApplication(
                    this.getApplicationContext(),
                    R.raw.auth_config);
```

### <a name="ios"></a>iOS

Les applications mobiles sur iOS doivent instancier la classe `MSALPublicClientApplication`. Pour instancier la classe, utilisez le code suivant.

```objc
NSError *msalError = nil;

MSALPublicClientApplicationConfig *config = [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"];
MSALPublicClientApplication *application = [[MSALPublicClientApplication alloc] initWithConfiguration:config error:&msalError];
```

```swift
let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>")
if let application = try? MSALPublicClientApplication(configuration: config){ /* Use application */}
```

Des [propriétés MSALPublicClientApplicationConfig supplémentaires](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALPublicClientApplicationConfig.html#/Configuration%20options) peuvent remplacer l’autorité par défaut, spécifier un URI de redirection ou modifier le comportement de mise en cache des jetons MSAL.

### <a name="xamarin-or-uwp"></a>Xamarin ou UWP

Cette section explique comment instancier l’application pour des applications Xamarin.iOS, Xamarin.Android et UWP.

#### <a name="instantiate-the-application"></a>Instancier l’application

Dans Xamarin ou UWP, le moyen le plus simple d’instancier l’application consiste à utiliser le code suivant. Dans ce code, `ClientId` est le GUID de votre application inscrite.

```csharp
var app = PublicClientApplicationBuilder.Create(clientId)
                                        .Build();
```

Des méthodes `With<Parameter>` supplémentaires définissent le parent de l’interface utilisateur, remplacent l’autorité par défaut, spécifient un nom et une version de client pour la télémétrie, spécifient un URI de redirection et spécifient la fabrique HTTP à utiliser. La fabrique HTTP peut être utilisée, par exemple, pour gérer les proxies et pour spécifier la télémétrie et la journalisation.

Les sections ci-dessous fournissent plus d’informations sur l’instanciation de l’application.

##### <a name="specify-the-parent-ui-window-or-activity"></a>Spécifier l’interface utilisateur, la fenêtre ou l’activité parente

Sur Android, vous devez transmettre l’activité parente avant d’effectuer l’authentification interactive. Sur iOS, lorsque vous utilisez un répartiteur, vous devez transmettre `ViewController`. De la même façon sur UWP, vous pouvez transmettre la fenêtre parente. Vous la transmettez lorsque vous acquérez le jeton. Toutefois, lorsque vous créez l’application, vous pouvez également spécifier un rappel comme délégué qui retourne `UIParent`.

```csharp
IPublicClientApplication application = PublicClientApplicationBuilder.Create(clientId)
  .ParentActivityOrWindowFunc(() => parentUi)
  .Build();
```

Sur Android, nous vous recommandons d’utiliser [`CurrentActivityPlugin`](https://github.com/jamesmontemagno/CurrentActivityPlugin). Le code de générateur `PublicClientApplication` qui en résulte ressemble à l’exemple suivant :

```csharp
// Requires MSAL.NET 4.2 or above
var pca = PublicClientApplicationBuilder
  .Create("<your-client-id-here>")
  .WithParentActivityOrWindow(() => CrossCurrentActivity.Current)
  .Build();
```

##### <a name="find-more-app-building-parameters"></a>Rechercher d’autres paramètres de création d’applications

Pour obtenir la liste de toutes les méthodes disponibles sur `PublicClientApplicationBuilder`, consultez la [liste des méthodes](/dotnet/api/microsoft.identity.client.publicclientapplicationbuilder#methods).

Pour obtenir une description de toutes les options exposées dans `PublicClientApplicationOptions`, consultez la [documentation de référence](/dotnet/api/microsoft.identity.client.publicclientapplicationoptions).

## <a name="tasks-for-xamarin-ios"></a>Tâches pour Xamarin iOS

Si vous utilisez MSAL.NET sur Xamarin iOS, effectuez les tâches suivantes.

* [Écraser et implémenter la fonction `OpenUrl` dans `AppDelegate`](msal-net-xamarin-ios-considerations.md#implement-openurl)
* [Activer les groupes de trousseau](msal-net-xamarin-ios-considerations.md#enable-keychain-access)
* [Activer le partage du cache de jeton](msal-net-xamarin-ios-considerations.md#enable-token-cache-sharing-across-ios-applications)
* [Activer l’accès au trousseau](msal-net-xamarin-ios-considerations.md#enable-keychain-access)

Pour plus d’informations, consultez [Considérations relatives à Xamarin iOS](msal-net-xamarin-ios-considerations.md).

## <a name="tasks-for-msal-for-ios-and-macos"></a>Tâches pour MSAL pour iOS et macOS

Ces tâches sont nécessaires lorsque vous utilisez MSAL pour iOS et macOS :

* [Implémenter le rappel`openURL`](#brokered-authentication-for-msal-for-ios-and-macos)
* [Activer les groupes d’accès au trousseau](howto-v2-keychain-objc.md)
* [Personnaliser les navigateurs et les vues web](customize-webviews.md)

## <a name="tasks-for-xamarinandroid"></a>Tâches pour Xamarin.Android

Si vous utilisez Xamarin.Android, effectuez les tâches suivantes :

- [S’assurer que le contrôle revient à la bibliothèque MSAL une fois la partie interactive du flux d’authentification terminée](msal-net-xamarin-android-considerations.md#ensure-that-control-returns-to-msal)
- [Mettre à jour le manifeste Android](msal-net-xamarin-android-considerations.md#update-the-android-manifest)
- [Utiliser la vue web incorporée (facultatif)](msal-net-xamarin-android-considerations.md#use-the-embedded-web-view-optional)
- [Détecter un problème le cas échéant](msal-net-xamarin-android-considerations.md#troubleshooting)

Pour plus d’informations, consultez [Considérations relatives à Xamarin.Android](msal-net-xamarin-android-considerations.md).

Pour plus d’informations concernant les navigateurs sur Android, consultez [Considérations spécifiques à Xamarin.Android lors de l’utilisation de MSAL.NET](msal-net-system-browser-android-considerations.md).

#### <a name="tasks-for-uwp"></a>Tâches pour UWP

Sur UWP, vous pouvez utiliser des réseaux d’entreprise. Les sections suivantes décrivent les tâches que vous devez effectuer dans le scénario d’entreprise.

Pour plus d’informations, consultez [Considérations spécifiques à UWP lors de l’utilisation de MSAL.NET](msal-net-uwp-considerations.md).

## <a name="configure-the-application-to-use-the-broker"></a>Configurer l’application pour utiliser le répartiteur

Sur Android et iOS, les répartiteurs activent :

- **Authentification unique (SSO)**  : Vous pouvez utiliser la SSO pour les appareils qui sont inscrits auprès d’Azure Active Directory (Azure AD). Lorsque vous utilisez la SSO, vos utilisateurs n’ont pas besoin de se connecter à chaque application.
- **Identification des appareils** : Ce paramètre active les stratégies d’accès conditionnel qui sont liées aux appareils Azure AD. Le processus d’authentification utilise le certificat de l’appareil qui a été créé lorsque l’appareil a été joint à l’espace de travail.
- **Vérification de l’identification de l’application** : Lorsqu’une application appelle le répartiteur, elle transmet son URL de redirection. Le répartiteur la vérifie ensuite.

### <a name="enable-the-broker-on-xamarin"></a>Activer le répartiteur sur Xamarin

Pour activer le répartiteur sur Xamarin, utilisez le paramètre `WithBroker()` lorsque vous appelez la méthode `PublicClientApplicationBuilder.CreateApplication`. Par défaut, `.WithBroker()` est défini sur true.

Pour activer l’authentification répartie pour Xamarin.iOS, suivez les étapes de la [section Xamarin.iOS](#enable-brokered-authentication-for-xamarin-ios) dans cet article.

### <a name="enable-the-broker-for-msal-for-android"></a>Activer le répartiteur pour MSAL pour Android

Pour plus d’informations sur l’activation d’un répartiteur sur Android, consultez [Authentification répartie sur Android](msal-android-single-sign-on.md).

### <a name="enable-the-broker-for-msal-for-ios-and-macos"></a>Activer le répartiteur pour MSAL pour iOS et macOS

L’authentification répartie est activée par défaut pour les scénarios Azure AD dans MSAL pour iOS et macOS.

Les sections suivantes fournissent des instructions pour configurer votre application pour la prise en charge de l’authentification répartie pour MSAL pour Xamarin.iOS ou MSAL pour iOS et macOS. Certaines étapes diffèrent dans les deux ensembles d’instructions.

### <a name="enable-brokered-authentication-for-xamarin-ios"></a>Activer l’authentification répartie pour Xamarin iOS

Suivez les étapes de cette section pour permettre à votre application Xamarin.iOS de communiquer avec l’application [Microsoft Authenticator](https://itunes.apple.com/us/app/microsoft-authenticator/id983156458).

#### <a name="step-1-enable-broker-support"></a>Étape 1 : Activer la prise en charge du répartiteur

La prise en charge du répartiteur est désactivée par défaut. Vous l’activez pour une classe de `PublicClientApplication` individuelle. Utilisez le paramètre `WithBroker()` lorsque vous créez la classe `PublicClientApplication` par le biais de `PublicClientApplicationBuilder`. Par défaut, le paramètre `WithBroker()` est défini sur true.

```csharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithReplyUri(redirectUriOnIos) // $"msauth.{Bundle.Id}://auth" (see step 6 below)
                .Build();
```

#### <a name="step-2-update-appdelegate-to-handle-the-callback"></a>Étape 2 : Mettre à jour AppDelegate pour gérer le rappel

Quand MSAL.NET appelle le répartiteur, ce dernier rappelle votre application. Il la rappelle à l’aide de la méthode `AppDelegate.OpenUrl`. MSAL attendant la réponse du répartiteur, votre application doit coopérer pour rappeler MSAL.NET. Vous configurez ce comportement en mettant à jour le fichier `AppDelegate.cs` pour écraser la méthode, comme le montre le code suivant.

```csharp
public override bool OpenUrl(UIApplication app, NSUrl url,
                             string sourceApplication,
                             NSObject annotation)
{
 if (AuthenticationContinuationHelper.IsBrokerResponse(sourceApplication))
 {
  AuthenticationContinuationHelper.SetBrokerContinuationEventArgs(url);
  return true;
 }
 else if (!AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url))
 {
  return false;
 }
 return true;
}
```

Cette méthode est appelée chaque fois que l'application est lancée. Elle permet de traiter la réponse du répartiteur et de terminer le processus d’authentification que MSAL.NET a démarré.

#### <a name="step-3-set-a-uiviewcontroller"></a>Étape 3 : Définir un UIViewController()

Pour Xamarin iOS, vous n’êtes normalement pas tenu de définir la fenêtre d’objet. Mais dans ce cas, vous devez la définir afin de pouvoir envoyer et recevoir des réponses d’un répartiteur. Dans `AppDelegate.cs`, vous devez définir un paramètre `ViewController` pour définir une fenêtre d’objet.

Pour définir la fenêtre de l’objet, procédez comme suit :

1. Dans `AppDelegate.cs`, affectez la valeur `App.RootViewController` à un nouveau `UIViewController()`. Ce paramètre permet de s’assurer que l’appel au répartiteur comprend `UIViewController`. S’il n’est pas défini correctement, vous pouvez recevoir cette erreur :

    `"uiviewcontroller_required_for_ios_broker":"UIViewController is null, so MSAL.NET cannot invoke the iOS broker. See https://aka.ms/msal-net-ios-broker."`

1. Lors de l’appel `AcquireTokenInteractive`, utilisez `.WithParentActivityOrWindow(App.RootViewController)`. Transmettez la référence à la fenêtre de l’objet que vous allez utiliser. Voici un exemple :

    Dans `App.cs` :
    ```csharp
       public static object RootViewController { get; set; }
    ```
    Dans `AppDelegate.cs` :
    ```csharp
       LoadApplication(new App());
       App.RootViewController = new UIViewController();
    ```
    Dans l’appel `AcquireToken` :
    ```csharp
    result = await app.AcquireTokenInteractive(scopes)
                 .WithParentActivityOrWindow(App.RootViewController)
                 .ExecuteAsync();
    ```

#### <a name="step-4-register-a-url-scheme"></a>Étape 4 : Inscrire un schéma d’URL

MSAL.NET utilise des URL pour appeler le répartiteur, avant de retourner la réponse du répartiteur à votre application. Pour terminer l’aller-retour, inscrivez un schéma d’URL de votre application dans le fichier `Info.plist`.

Pour inscrire le schéma d’URL de votre application, procédez comme suit :

1. Ajoutez le préfixe `msauth` à `CFBundleURLSchemes`.
1. Ajoutez `CFBundleURLName` à la fin. Suivez ce modèle :

   `$"msauth.(BundleId)"`

   Ici, `BundleId` identifie de façon unique votre appareil. Par exemple, si `BundleId` est `yourcompany.xforms`, votre schéma d’URL est `msauth.com.yourcompany.xforms`.

   > [!NOTE]
   > Ce schéma d’URL fera partie de l’URI de redirection qui identifie de façon unique votre application lorsqu’il reçoit la réponse du répartiteur.

   ```XML
    <key>CFBundleURLTypes</key>
       <array>
         <dict>
           <key>CFBundleTypeRole</key>
           <string>Editor</string>
           <key>CFBundleURLName</key>
           <string>com.yourcompany.xforms</string>
           <key>CFBundleURLSchemes</key>
           <array>
             <string>msauth.com.yourcompany.xforms</string>
           </array>
         </dict>
       </array>
   ```

#### <a name="step-5-add-to-the-lsapplicationqueriesschemes-section"></a>Étape 5 : Ajouter à la section LSApplicationQueriesSchemes

MSAL utilise `–canOpenURL:` pour vérifier si le répartiteur est installé sur l’appareil. Dans iOS 9, Apple a verrouillé les schémas qu’une application peut interroger.

Ajoutez `msauthv2` à la section `LSApplicationQueriesSchemes` du fichier `Info.plist`, comme dans l’exemple de code suivant :

```XML
<key>LSApplicationQueriesSchemes</key>
    <array>
      <string>msauthv2</string>
    </array>
```

### <a name="brokered-authentication-for-msal-for-ios-and-macos"></a>Authentification répartie pour MSAL pour iOS et macOS

L’authentification répartie est activée par défaut pour les scénarios Azure AD.

#### <a name="step-1-update-appdelegate-to-handle-the-callback"></a>Étape 1 : Mettre à jour AppDelegate pour gérer le rappel

Quand MSAL pour iOS et macOS appelle le répartiteur, celui-ci rappelle votre application via la méthode `openURL`. MSAL attendant la réponse du répartiteur, votre application doit coopérer pour rappeler MSAL. Configurez cette capacité en mettant à jour le fichier `AppDelegate.m` pour écraser la méthode, comme le montre le code suivant.

```objc
- (BOOL)application:(UIApplication *)app
            openURL:(NSURL *)url
            options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
{
    return [MSALPublicClientApplication handleMSALResponse:url
                                         sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
}
```

```swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {

        guard let sourceApplication = options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String else {
            return false
        }

        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApplication)
    }
```

> [!NOTE]
> Si vous avez adopté `UISceneDelegate` sur iOS 13 ou une version ultérieure, placez le rappel MSAL dans le `scene:openURLContexts:` de `UISceneDelegate` à la place. MSAL `handleMSALResponse:sourceApplication:` ne doit être appelé qu’une seule fois pour chaque URL.
>
> Pour plus d’informations, consultez la [documentation d’Apple](https://developer.apple.com/documentation/uikit/uiscenedelegate/3238059-scene?language=objc).

#### <a name="step-2-register-a-url-scheme"></a>Étape 2 : Inscrire un schéma d’URL

MSAL pour iOS et macOS utilise des URL pour appeler le répartiteur, avant de retourner la réponse du répartiteur à votre application. Pour terminer l’aller-retour, inscrivez un schéma d’URL pour votre application dans le fichier `Info.plist`.

Pour inscrire un schéma pour votre application :

1. Ajoutez le préfixe `msauth` à votre modèle d’URL personnalisée.

1. Ajoutez l’identificateur de votre offre groupée à la fin de votre schéma. Suivez ce modèle :

   `$"msauth.(BundleId)"`

   Ici, `BundleId` identifie de façon unique votre appareil. Par exemple, si `BundleId` est `yourcompany.xforms`, votre schéma d’URL est `msauth.com.yourcompany.xforms`.

   > [!NOTE]
   > Ce schéma d’URL fera partie de l’URI de redirection qui identifie de façon unique votre application lorsqu’il reçoit la réponse du répartiteur. Assurez-vous que l’URI de redirection au format `msauth.(BundleId)://auth` est inscrit pour votre application dans le [Portail Azure](https://portal.azure.com).

   ```XML
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

#### <a name="step-3-add-lsapplicationqueriesschemes"></a>Étape 3 : Ajouter LSApplicationQueriesSchemes

Ajoutez `LSApplicationQueriesSchemes` pour autoriser les appels à l’application Microsoft Authenticator, si elle est installée.

> [!NOTE]
> Le schéma `msauthv3` est nécessaire lorsque votre application est compilée à l’aide de Xcode 11 et versions ultérieures.

Voici un exemple d’ajout de `LSApplicationQueriesSchemes` :

```XML
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>msauthv2</string>
  <string>msauthv3</string>
</array>
```

### <a name="brokered-authentication-for-xamarinandroid"></a>Authentification répartie pour Xamarin.Android

Pour plus d’informations sur l’activation d’un répartiteur sur Android, consultez [Authentification répartie sur Xamarin.Android](msal-net-use-brokers-with-xamarin-apps.md#brokered-authentication-for-android).

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Acquisition d’un jeton](scenario-mobile-acquire-token.md).