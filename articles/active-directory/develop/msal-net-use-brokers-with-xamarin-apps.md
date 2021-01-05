---
title: Utiliser des répartiteurs avec Xamarin iOS et Android | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment configurer des applications Xamarin iOS capables d’utiliser Microsoft Authenticator et Microsoft Authentication Library pour .NET (MSAL.NET). Découvrez aussi comment migrer d’Azure AD Authentication Library pour .NET (ADAL.NET) vers Microsoft Authentication Library pour .NET (MSAL.NET).
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
ms.openlocfilehash: 7fa13a328a55b0e9eaa546e70bf0711f4f011cf1
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96173428"
---
# <a name="use-microsoft-authenticator-or-intune-company-portal-on-xamarin-applications"></a>Utiliser Microsoft Authenticator ou le Portail d’entreprise Intune sur des applications Xamarin

Sur Android et iOS, des répartiteurs tels que Microsoft Authenticator et le Portail d’entreprise Microsoft Intune pour Android permettent d’effectuer les opérations suivantes :

- **Authentification unique (SSO)**  : les utilisateurs n’ont pas besoin de se connecter à chaque application.
- **Identification de l’appareil** : le répartiteur accède au certificat de l’appareil. Ce certificat est créé sur l’appareil quand celui-ci est joint à l’espace de travail.
- **Vérification de l’identification de l’application** : quand une application appelle le répartiteur, elle transmet son URL de redirection. Le répartiteur vérifie alors l’URL.

Pour activer l’une de ces fonctionnalités, utilisez le paramètre `WithBroker()` lorsque vous appelez la méthode `PublicClientApplicationBuilder.CreateApplication`. Par défaut, le paramètre `.WithBroker()` est défini sur true.

La configuration de l’authentification répartie dans la bibliothèque d’authentification Microsoft pour .NET (MSAL.NET) varie selon la plateforme :

* [Applications iOS](#brokered-authentication-for-ios)
* [Applications Android](#brokered-authentication-for-android)

## <a name="brokered-authentication-for-ios"></a>Authentification répartie pour iOS

Suivez les étapes ci-dessous pour permettre à votre application Xamarin.iOS de communiquer avec l’application [Microsoft Authenticator](https://itunes.apple.com/us/app/microsoft-authenticator/id983156458). Si vous ciblez iOS 13, vous pouvez vous renseigner sur le [changement cassant d’API d’Apple](./msal-net-xamarin-ios-considerations.md).

### <a name="step-1-enable-broker-support"></a>Étape 1 : Activer la prise en charge du répartiteur

Vous devez activer la prise en charge du répartiteur pour chaque instance de `PublicClientApplication`. La prise en charge est désactivée par défaut. Si vous créez `PublicClientApplication` avec `PublicClientApplicationBuilder`, utilisez le paramètre `WithBroker()` comme dans l’exemple suivant. Par défaut, le paramètre `WithBroker()` est défini sur true.

```csharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithReplyUri(redirectUriOnIos) // $"msauth.{Bundle.Id}://auth" (see step 6 below)
                .Build();
```

### <a name="step-2-enable-keychain-access"></a>Étape 2 : Activer l’accès au trousseau

Pour activer l’accès au trousseau, vous devez avoir défini un groupe d’accès au trousseau pour votre application. Vous pouvez utiliser l’API `WithIosKeychainSecurityGroup()` pour définir votre groupe d’accès au trousseau lorsque vous créez votre application :

```csharp
var builder = PublicClientApplicationBuilder
     .Create(ClientId)
     .WithIosKeychainSecurityGroup("com.microsoft.adalcache")
     .Build();
```

Pour plus d'informations, consultez [Activer l’accès au trousseau](msal-net-xamarin-ios-considerations.md#enable-keychain-access).

### <a name="step-3-update-appdelegate-to-handle-the-callback"></a>Étape 3 : Mettre à jour AppDelegate pour gérer le rappel

Quand MSAL.NET appelle le répartiteur, ce dernier rappelle votre application via la méthode `OpenUrl` de la classe `AppDelegate`. Comme MSAL attend la réponse du répartiteur, votre application doit coopérer pour rappeler MSAL.NET. Pour permettre cette coopération, mettez à jour le fichier *AppDelegate.cs* pour remplacer la méthode suivante.

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

Cette méthode est appelée à chaque démarrage de l’application. Elle permet de traiter la réponse du répartiteur et d’accomplir le processus d’authentification lancé par MSAL.NET.

### <a name="step-4-set-uiviewcontroller"></a>Étape 4 : Définir UIViewController()

Toujours dans le fichier *AppDelegate.cs*, vous devez définir une fenêtre d’objet. Vous n’avez généralement pas besoin de définir la fenêtre d’objet pour Xamarin iOS, mais vous avez besoin d’une fenêtre d’objet pour envoyer des réponses au répartiteur et recevoir des réponses.

Pour configurer la fenêtre d’objet :

1. Dans le fichier *AppDelegate.cs*, définissez `App.RootViewController` sur un nouveau `UIViewController()`. Cette affectation garantit que l’appel au répartiteur comprend `UIViewController`. Si ce paramètre n’est pas correctement affecté, vous risquez de recevoir cette erreur :

      `"uiviewcontroller_required_for_ios_broker":"UIViewController is null, so MSAL.NET cannot invoke the iOS broker. See https://aka.ms/msal-net-ios-broker"`

1. Dans l’appel `AcquireTokenInteractive`, utilisez `.WithParentActivityOrWindow(App.RootViewController)`, puis passez la référence à la fenêtre d’objet que vous allez utiliser.

    Dans *App.cs* :

    ```csharp
       public static object RootViewController { get; set; }
    ```

    Dans *AppDelegate.cs* :

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

### <a name="step-5-register-a-url-scheme"></a>Étape 5 : Inscrire un schéma d’URL

MSAL.NET utilise des URL pour appeler le répartiteur, avant de retourner la réponse du répartiteur à votre application. Pour effectuer l’aller-retour, inscrivez un schéma d’URL pour votre application dans le fichier *Info.plist*.

Le nom `CFBundleURLSchemes` doit inclure le préfixe `msauth.`, suivi de `CFBundleURLName`.

Dans le schéma d’URL, `BundleId` identifie de manière unique l’application : `$"msauth.(BundleId)"`. Par conséquent, si `BundleId` est `com.yourcompany.xforms`, le schéma d’URL est `msauth.com.yourcompany.xforms`.

> [!NOTE]
> Ce schéma d’URL fait alors partie de l’URI de redirection qui identifie de manière unique votre application lorsqu’elle reçoit la réponse du répartiteur.

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

### <a name="step-6-add-the-broker-identifier-to-the-lsapplicationqueriesschemes-section"></a>Étape 6 : Ajouter l’identificateur du répartiteur à la section LSApplicationQueriesSchemes

MSAL utilise `–canOpenURL:` pour vérifier si le répartiteur est installé sur l’appareil. Dans iOS 9, Apple a verrouillé les schémas qu’une application peut interroger.

Ajoutez `msauthv2` à la section `LSApplicationQueriesSchemes` du fichier *Info.plist*, comme dans l’exemple suivant :

```XML
<key>LSApplicationQueriesSchemes</key>
    <array>
      <string>msauthv2</string>
      <string>msauthv3</string>
    </array>
```

### <a name="step-7-add-a-redirect-uri-to-your-app-registration"></a>Étape 7 : Ajouter un URI de redirection à l’inscription de votre application

Si vous utilisez le répartiteur, votre URI de redirection doit remplir une exigence supplémentaire. Le format de l'URI de redirection _doit_ être le suivant :

```csharp
$"msauth.{BundleId}://auth"
```

Voici un exemple :

```csharp
public static string redirectUriOnIos = "msauth.com.yourcompany.XForms://auth";
```

Vous remarquerez que l’URI de redirection correspond au nom `CFBundleURLSchemes` que vous avez indiqué dans le fichier *Info.plist*.

Ajoutez l’URI de redirection à l’inscription de l’application dans le [portail Azure](https://portal.azure.com). Pour générer un URI de redirection correctement mis en forme, utilisez les **inscriptions d’applications** dans le portail Azure pour générer l’URI de redirection répartie à partir de l’ID d’offre groupée.

**Pour générer l’URI de redirection :**

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez **Azure Active Directory** > **Inscriptions d’applications** > votre application inscrite.
1. Sélectionnez **Authentification** > **Ajouter une plateforme** > **iOS / macOS**.
1. Entrez votre ID d’offre groupée, puis sélectionnez **Configurer**.

    Copiez l’URI de redirection généré qui apparaît dans la zone de texte **URI de redirection** à inclure dans votre code :

    :::image type="content" source="media/msal-net-use-brokers-with-xamarin-apps/portal-01-ios-platform-settings.png" alt-text="Paramètres de plateforme iOS avec URI de redirection généré dans le portail Azure":::
1. Sélectionnez **Terminé** pour terminer la génération de l’URI de redirection.

## <a name="brokered-authentication-for-android"></a>Authentification répartie pour Android

### <a name="step-1-enable-broker-support"></a>Étape 1 : Activer la prise en charge du répartiteur

La prise en charge du répartiteur est activée par `PublicClientApplication`. Elle est désactivée par défaut. Utilisez le paramètre `WithBroker()` (défini sur true par défaut) lors de la création de `IPublicClientApplication` via `PublicClientApplicationBuilder`.

```csharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithRedirectUri(redirectUriOnAndroid) // See step #4
                .Build();
```

### <a name="step-2-update-appdelegate-to-handle-the-callback"></a>Étape 2 : Mettre à jour AppDelegate pour gérer le rappel

Quand MSAL.NET appelle le répartiteur, ce dernier rappelle à son tour votre application avec la méthode `OnActivityResult()`. Étant donné que MSAL attend la réponse du répartiteur, votre application doit router le résultat vers MSAL.NET.

Routez le résultat vers la méthode `SetAuthenticationContinuationEventArgs(int requestCode, Result resultCode, Intent data)` en substituant la méthode `OnActivityResult()` comme indiqué ici :

```csharp
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
   base.OnActivityResult(requestCode, resultCode, data);
   AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(requestCode, resultCode, data);
}
```

Cette méthode est appelée chaque fois que l’application de répartiteur est lancée et permet de traiter la réponse du répartiteur et d’accomplir le processus d’authentification démarré par MSAL.NET.

### <a name="step-3-set-an-activity"></a>Étape 3 : Définir une activité

Pour activer l’authentification répartie, définissez une activité afin que MSAL puisse envoyer la réponse au répartiteur et recevoir sa réponse. Pour ce faire, fournissez l’activité (généralement, `MainActivity`) à `WithParentActivityOrWindow(object parent)`, l’objet parent.

Par exemple, dans l’appel à `AcquireTokenInteractive()` :

```csharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow((Activity)context))
             .ExecuteAsync();
```

### <a name="step-4-add-a-redirect-uri-to-your-app-registration"></a>Étape 4 : Ajouter un URI de redirection à l’inscription de votre application

MSAL utilise des URL pour appeler le répartiteur, avant de retourner à votre application. Pour effectuer cet aller-retour, vous devez inscrire un **URI de redirection** pour votre application à l’aide du [portail Azure](https://portal.azure.com).

Le format de l’URI de redirection pour votre application dépend du certificat utilisé pour signer l’APK. Par exemple :

```
msauth://com.microsoft.xforms.testApp/hgbUYHVBYUTvuvT&Y6tr554365466=
```

La dernière partie de l’URI, `hgbUYHVBYUTvuvT&Y6tr554365466=`, correspond à la version encodée au format Base64 de la signature utilisée pour l’APK. Lors du développement de votre application dans Visual Studio, si vous déboguez votre code sans signer l’APK avec un certificat spécifique, Visual Studio signe l’APK pour vous à des fins de débogage. Lorsque Visual Studio signe l’APK pour vous de cette manière, il lui donne une signature unique pour l’ordinateur sur lequel il est basé. Par conséquent, chaque fois que vous créez votre application sur un autre ordinateur, vous devez mettre à jour l’URI de redirection dans le code de l’application et dans l’inscription de l’application sur le portail Azure afin de vous authentifier auprès de MSAL.

Lors du débogage, vous pouvez rencontrer une exception MSAL (ou un message du journal) indiquant que l’URI de redirection fourni est incorrect. **Le message d’exception ou de journal indique également l’URI de redirection que vous devez utiliser** avec l’ordinateur sur lequel vous effectuez le débogage. Vous pouvez utiliser l’URI de redirection fourni pour poursuivre le développement de votre application à condition de mettre à jour l’URI de redirection dans le code et d’ajouter l’URI de redirection fourni à l’inscription de l’application dans le portail Azure.

Une fois que vous êtes prêt à finaliser votre code, mettez à jour l’URI de redirection dans le code et l’inscription de l’application dans le portail Azure afin d’utiliser la signature du certificat avec laquelle vous signez l’APK.

Dans la pratique, cela signifie que vous devez envisager d’ajouter un URI de redirection pour chaque membre de votre équipe de développement, *plus* un URI de redirection pour la version signée de production de l’APK.

Vous pouvez calculer cette signature vous-même, d’une façon similaire à celle utilisée par MSAL :

```csharp
   private string GetRedirectUriForBroker()
   {
      string packageName = Application.Context.PackageName;
      string signatureDigest = this.GetCurrentSignatureForPackage(packageName);
      if (!string.IsNullOrEmpty(signatureDigest))
      {
            return string.Format(CultureInfo.InvariantCulture, "{0}://{1}/{2}", RedirectUriScheme,
               packageName.ToLowerInvariant(), signatureDigest);
      }

      return string.Empty;
   }

   private string GetCurrentSignatureForPackage(string packageName)
   {
            PackageInfo info = Application.Context.PackageManager.GetPackageInfo(packageName,
               PackageInfoFlags.Signatures);
            if (info != null && info.Signatures != null && info.Signatures.Count > 0)
            {
               // First available signature. Applications can be signed with multiple signatures.
               // The order of Signatures is not guaranteed.
               Signature signature = info.Signatures[0];
               MessageDigest md = MessageDigest.GetInstance("SHA");
               md.Update(signature.ToByteArray());
               return Convert.ToBase64String(md.Digest(), Base64FormattingOptions.None);
               // Server side needs to register all other tags. ADAL will
               // send one of them.
            }
   }
```

Vous avez également la possibilité d’acquérir la signature pour votre package en utilisant **keytool** avec les commandes suivantes :

* Windows :
    ```console
    keytool.exe -list -v -keystore "%LocalAppData%\Xamarin\Mono for Android\debug.keystore" -alias androiddebugkey -storepass android -keypass android
    ````
* macOS :
    ```console
    keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64
    ````

### <a name="step-5-optional-fall-back-to-the-system-browser"></a>Étape 5 (facultatif) : Revenir au navigateur système

Si MSAL est configuré pour utiliser le répartiteur mais que celui-ci n’est pas installé, MSAL revient à l’utilisation d’une vue web (un navigateur). MSAL essaie de s’authentifier à l’aide du navigateur système par défaut sur l’appareil, ce qui échoue car l’URI de redirection est configuré pour le répartiteur et que le navigateur système ne sait pas comment l’utiliser pour revenir à MSAL. Pour éviter cet échec, vous pouvez configurer un *filtre d’intention* avec l’URI de redirection du répartiteur que vous avez utilisé à l’étape 4.

Modifiez le manifeste de votre application pour ajouter ce filtre d’intention :

```xml
<!-- NOTE the SLASH (required) that prefixes the signature value in the path attribute.
     The signature value is the Base64-encoded signature discussed above. -->
<intent-filter>
      <data android:scheme="msauth"
                    android:host="Package Name"
                    android:path="/Package Signature"/>
```

Par exemple, si vous avez un URI de redirection `msauth://com.microsoft.xforms.testApp/hgbUYHVBYUTvuvT&Y6tr554365466=`, votre manifeste doit ressembler à l’extrait de code XML suivant.

La barre oblique (`/`) devant la signature dans la valeur `android:path` est **requise**.

```xml
<!-- NOTE the SLASH (required) that prefixes the signature value in the path attribute.
     The signature value is the Base64-encoded signature discussed above. -->
<intent-filter>
      <data android:scheme="msauth"
                    android:host="com.microsoft.xforms.testApp"
                    android:path="/hgbUYHVBYUTvuvT&Y6tr554365466="/>
```

Comme alternative, vous pouvez configurer MSAL pour revenir au navigateur incorporé, qui ne repose pas sur un URI de redirection :

```csharp
.WithUseEmbeddedWebUi(true)
```

## <a name="troubleshooting-tips-for-android-brokered-authentication"></a>Conseils de dépannage pour l’authentification répartie Android

Voici quelques conseils pour éviter les problèmes quand vous implémentez l’authentification répartie sur Android :

- **Redirect URI** (URI de redirection) – Ajoutez un URI de redirection à l’inscription de votre application dans le [portail Azure](https://portal.azure.com/). Un URI de redirection manquant ou incorrect est un problème rencontré couramment par les développeurs.
- **Broker version** (Version du répartiteur) – Installez la version minimale requise des applications de répartiteur. L’une de ces deux applications peut être utilisée pour l’authentification répartie sur Android.
  - [Portail d’entreprise Intune](https://play.google.com/store/apps/details?id=com.microsoft.windowsintune.companyportal) (version 5.0.4689.0 ou ultérieure)
  - [Microsoft Authenticator](https://play.google.com/store/apps/details?id=com.azure.authenticator) (version 6.2001.0140 ou ultérieure).
- **Broker precedence** (Priorité du répartiteur) – MSAL communique avec le *premier répartiteur installé* sur l’appareil lorsque plusieurs répartiteurs sont installés.

    Exemple : Si vous commencez par installer Microsoft Authenticator, puis installez le portail d’entreprise Intune, l’authentification répartie se produit *uniquement* sur Microsoft Authenticator.
- **Logs** (Journaux) – Si vous rencontrez un problème avec l’authentification répartie, l’affichage des journaux du répartiteur peut vous aider à diagnostiquer la cause.
  - Affichez les journaux de Microsoft Authenticator :

    1. Sélectionnez le bouton de menu en haut à droite de l’application.
    1. Sélectionnez **Aide** > **Envoyer des journaux** > **Afficher les journaux**.
    1. Sélectionnez **Copier tout** pour copier les journaux du répartiteur dans le presse-papiers de l’appareil.

    La meilleure façon de déboguer ces journaux consiste à les envoyer par courrier électronique à vous-même et à les afficher sur votre ordinateur de développement. Il peut s’avérer plus facile d’analyser les journaux sur votre ordinateur plutôt que sur l’appareil lui-même. Vous pouvez également utiliser un éditeur de test sur Android pour enregistrer les journaux dans un fichier texte, puis utiliser un câble USB pour copier le fichier sur un ordinateur.

  - Affichez les journaux du portail d’entreprise Intune :

    1. Sélectionnez le bouton de menu en haut à gauche de l’application.
    1. Sélectionnez **Paramètres** > **Données de diagnostic**.
    1. Sélectionnez **Copier les journaux** pour copier les journaux du répartiteur vers la carte SD de l’appareil.
    1. Connectez l’appareil à un ordinateur à l’aide d’un câble USB pour afficher les journaux sur votre ordinateur de développement.

    Une fois que vous avez les journaux, vous pouvez les rechercher pour vos tentatives d’authentification via l’ID de corrélation. L’ID de corrélation est attaché à chaque demande d’authentification. Pour rechercher les erreurs retournées par le point de terminaison d’authentification de la plateforme d’identité Microsoft, recherchez `AADSTS`.

## <a name="next-steps"></a>Étapes suivantes

Découvrez les [considérations relatives à l’utilisation de la plateforme Windows universelle avec MSAL.NET](msal-net-uwp-considerations.md).
