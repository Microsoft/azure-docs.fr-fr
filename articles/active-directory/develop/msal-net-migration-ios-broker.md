---
title: Migrer des applications Xamarin avec des répartiteurs vers MSAL.NET
titleSuffix: Microsoft identity platform
description: Découvrez comment migrer d’ADAL.NET vers MSAL.NET des applications Xamarin iOS qui utilisent Microsoft Authenticator.
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/08/2019
ms.author: jmprieur
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: b4eff5910ff5230902d497b55b2afbe6d605365a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89177429"
---
# <a name="migrate-ios-applications-that-use-microsoft-authenticator-from-adalnet-to-msalnet"></a>Migrer des applications iOS d’ADAL.NET vers MSAL.NET en utilisant Microsoft Authenticator

Vous utilisez la bibliothèque d’authentification Azure Active Directory pour .NET (ADAL.NET) et le répartiteur iOS. Il est maintenant temps de migrer vers la [bibliothèque d'authentification Microsoft](msal-overview.md) pour .NET (MSAL.NET) qui prend en charge le répartiteur sur iOS depuis la version 4.3.

Par où démarrer ? Cet article vous aide à migrer votre application Xamarin iOS d'ADAL vers MSAL.

## <a name="prerequisites"></a>Prérequis
Cet article part du principe que vous disposez déjà d’une application Xamarin iOS intégrée avec le répartiteur iOS. Si ce n’est pas le cas, passez directement à MSAL.NET pour commencer à implémenter le répartiteur. Pour des informations sur l’appel du répartiteur iOS dans MSAL.NET avec une nouvelle application, consultez [cette documentation](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Leveraging-the-broker-on-iOS#why-use-brokers-on-xamarinios-and-xamarinandroid-applications).

## <a name="background"></a>Arrière-plan

### <a name="what-are-brokers"></a>Que sont les répartiteurs ?

Les répartiteurs sont des applications fournies par Microsoft sur Android et iOS. (Voir l'application [Microsoft Authenticator](https://www.microsoft.com/p/microsoft-authenticator/9nblgggzmcj6) sur iOS et Android, ainsi que l’application Portail d'entreprise Intune sur Android.)

Ils permettent d’effectuer les opérations suivantes :

- Authentification unique.
- Identification d’appareil requise par certaines [stratégies d’accès conditionnel](../conditional-access/overview.md). Pour plus d’informations, consultez [Gestion des appareils](../conditional-access/concept-conditional-access-conditions.md#device-platforms).
- Vérification de l’identification de l'application, également requise dans certains scénarios d’entreprise. Pour plus d'informations, consultez [Gestion des applications mobiles Intune (GAM)](/intune/mam-faq).

## <a name="migrate-from-adal-to-msal"></a>Migrer d’ADAL vers MSAL

### <a name="step-1-enable-the-broker"></a>Étape 1 : Activer le répartiteur

<table>
<tr><td>Code ADAL actuel :</td><td>Contrepartie MSAL :</td></tr>
<tr><td>
Dans ADAL.NET, la prise en charge du répartiteur était activée pour chaque contexte d’authentification. Elle est désactivée par défaut. Vous deviez définir un

indicateur `useBroker` sur la valeur true dans le constructeur `PlatformParameters` pour appeler le répartiteur :

```csharp
public PlatformParameters(
        UIViewController callerViewController,
        bool useBroker)
```
Par ailleurs, dans le code spécifique de la plateforme de cet exemple, dans le convertisseur de page pour iOS, vous devez définir l’indicateur `useBroker`
sur true :
```csharp
page.BrokerParameters = new PlatformParameters(
          this,
          true,
          PromptBehavior.SelectAccount);
```

Ensuite, incluez les paramètres dans l’appel d’acquisition de jeton :
```csharp
 AuthenticationResult result =
                    await
                        AuthContext.AcquireTokenAsync(
                              Resource,
                              ClientId,
                              new Uri(RedirectURI),
                              platformParameters)
                              .ConfigureAwait(false);
```

</td><td>
Dans MSAL.NET, la prise en charge du répartiteur est activée pour chaque application cliente publique (PublicClientApplication). Elle est désactivée par défaut. Pour l’activer, utilisez le

le paramètre `WithBroker()` (défini sur true par défaut) pour appeler répartiteur :

```csharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithReplyUri(redirectUriOnIos)
                .Build();
```
Dans l’appel d’acquisition de jeton :
```csharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow(App.RootViewController)
             .ExecuteAsync();
```
</table>

### <a name="step-2-set-a-uiviewcontroller"></a>Étape 2 : Définir un UIViewController()
Dans ADAL.NET, vous avez transmis un UIViewController dans le cadre de `PlatformParameters`. (Voir l'exemple de l'étape 1.) Dans MSAL.NET, pour offrir plus de flexibilité aux développeurs, une fenêtre d’objet est utilisée, mais n’est pas obligatoire dans l’utilisation normale d’iOS. Pour pouvoir utiliser le répartiteur, définissez la fenêtre d’objet pour qu’elle envoie et reçoive des réponses du répartiteur.
<table>
<tr><td>Code ADAL actuel :</td><td>Contrepartie MSAL :</td></tr>
<tr><td>
Un UIViewController est transmis dans

`PlatformParameters` dans la plateforme spécifique à iOS.

```csharp
page.BrokerParameters = new PlatformParameters(
          this,
          true,
          PromptBehavior.SelectAccount);
```
</td><td>
Dans MSAL.NET, vous devez effectuer deux opérations pour définir la fenêtre d’objet pour iOS :

1. Dans `AppDelegate.cs`, définissez `App.RootViewController` sur un nouveau `UIViewController()`.
Cette affectation vérifie l’existence d’un UIViewController avec l’appel au répartiteur. S’il n’est pas défini correctement, vous pouvez recevoir cette erreur : `"uiviewcontroller_required_for_ios_broker":"UIViewController is null, so MSAL.NET cannot invoke the iOS broker. See https://aka.ms/msal-net-ios-broker"`
1. Dans l’appel AcquireTokenInteractive, utilisez `.WithParentActivityOrWindow(App.RootViewController)` et transmettez la référence à la fenêtre d’objet que vous allez utiliser.

**Par exemple :**

Dans `App.cs` :
```csharp
   public static object RootViewController { get; set; }
```
Dans `AppDelegate.cs` :
```csharp
   LoadApplication(new App());
   App.RootViewController = new UIViewController();
```
Dans l’appel d’acquisition de jeton :
```csharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow(App.RootViewController)
             .ExecuteAsync();
```

</table>

### <a name="step-3-update-appdelegate-to-handle-the-callback"></a>Étape 3 : Mettre à jour AppDelegate pour gérer le rappel
ADAL et MSAL appellent le répartiteur, puis celui-ci rappelle votre application par le biais de la méthode `OpenUrl` de la classe `AppDelegate`. Pour plus d’informations, consultez [cette documentation](msal-net-use-brokers-with-xamarin-apps.md#step-3-update-appdelegate-to-handle-the-callback).

Il n'existe ici aucune différence entre ADAL.NET et MSAL.NET.

### <a name="step-4-register-a-url-scheme"></a>Étape 4 : Inscrire un schéma d’URL
ADAL.NET et MSAL.NET utilisent des URL pour appeler le répartiteur, avant de retourner la réponse du répartiteur à l’application. Inscrivez le modèle d’URL dans le fichier `Info.plist` de votre application comme suit :

<table>
<tr><td>Code ADAL actuel :</td><td>Contrepartie MSAL :</td></tr>
<tr><td>
Le modèle d’URL est propre à votre application.
</td><td>
Les

`CFBundleURLSchemes` doit inclure

`msauth.`

en tant que préfixe, suivi de votre `CFBundleURLName`.

Par exemple : `$"msauth.(BundleId")`

```csharp
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

> [!NOTE]
> Ce schéma d’URL fait partie intégrante de l’URI de redirection utilisé pour identifier de manière unique l’application lorsqu’elle reçoit la réponse du répartiteur.

</table>

### <a name="step-5-add-the-broker-identifier-to-the-lsapplicationqueriesschemes-section"></a>Étape 5 : Ajouter l’identificateur du répartiteur à la section LSApplicationQueriesSchemes

ADAL.NET et MSAL.NET utilisent `-canOpenURL:` pour vérifier si le répartiteur est installé sur l’appareil. Ajoutez l’identificateur correct pour le répartiteur iOS à la section LSApplicationQueriesSchemes du fichier info.plist comme suit :

<table>
<tr><td>Code ADAL actuel :</td><td>Contrepartie MSAL :</td></tr>
<tr><td>
Utilisations

`msauth`


```csharp
<key>LSApplicationQueriesSchemes</key>
<array>
     <string>msauth</string>
</array>
```
</td><td>
Utilisations

`msauthv2`


```csharp
<key>LSApplicationQueriesSchemes</key>
<array>
     <string>msauthv2</string>
     <string>msauthv3</string>
</array>
```
</table>

### <a name="step-6-register-your-redirect-uri-in-the-azure-portal"></a>Étape 6 : Inscrire votre URI de redirection dans le portail Azure

ADAL.NET et MSAL.NET ajoutent tous deux une exigence supplémentaire à l'URI de redirection lorsqu'il cible le répartiteur. Enregistrez l’URI de redirection avec votre application sur le portail Azure.
<table>
<tr><td>Code ADAL actuel :</td><td>Contrepartie MSAL :</td></tr>
<tr><td>

`"<app-scheme>://<your.bundle.id>"`

Exemple :

`mytestiosapp://com.mycompany.myapp`
</td><td>

`$"msauth.{BundleId}://auth"`

Exemple :

`public static string redirectUriOnIos = "msauth.com.yourcompany.XForms://auth"; `

</table>

Pour plus d’informations sur l’inscription de l’URI de redirection dans le Portail Azure, consultez [Étape 7 : Ajouter un URI de redirection à l’inscription de votre application](msal-net-use-brokers-with-xamarin-apps.md#step-7-add-a-redirect-uri-to-your-app-registration).

### <a name="step-7-set-the-entitlementsplist"></a>**Étape 7 : Définir les droits Entitlements.plist**

Activer l’accès au trousseau dans le fichier de*Entitlements.plist* :

```xml
 <key>keychain-access-groups</key>
    <array>
      <string>$(AppIdentifierPrefix)com.microsoft.adalcache</string>
    </array>
```

Pour plus d’informations sur l’activation de l’accès au trousseau, consultez [Activer l’accès au trousseau](msal-net-xamarin-ios-considerations.md#enable-keychain-access).

## <a name="next-steps"></a>Étapes suivantes

Découvrez les [considérations spécifiques de Xamarin iOS avec MSAL.NET](msal-net-xamarin-ios-considerations.md).