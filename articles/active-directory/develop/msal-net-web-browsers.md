---
title: Utilisation de navigateurs web (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: En savoir plus sur les considérations spécifiques lors de l’utilisation de Xamarin Android avec la bibliothèque d’authentification Microsoft pour .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/18/2020
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: bf51f39a789b91a4cb0b88eb8bb1f2989bec7358
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88165819"
---
# <a name="using-web-browsers-msalnet"></a>Utilisation de navigateurs web (MSAL.NET)

Des navigateurs web sont nécessaires pour l’authentification interactive. Par défaut, MSAL.NET prend en charge le [navigateur web du système](#system-web-browser-on-xamarinios-xamarinandroid) sur Xamarin.iOS et Xamarin.Android. Par contre, [vous pouvez aussi activer le navigateur web intégré](#enable-embedded-webviews-on-ios-and-android) en fonction de vos exigences (UX, besoin de l’authentification unique (SSO), sécurité) dans les applications [Xamarin.iOS](#choosing-between-embedded-web-browser-or-system-browser-on-xamarinios) et [Xamarin.Android](#detecting-the-presence-of-custom-tabs-on-xamarinandroid). Et vous pouvez même [choisir de façon dynamique](#detecting-the-presence-of-custom-tabs-on-xamarinandroid) le navigateur web à utiliser en fonction de la présence de Chrome, ou d’un navigateur prenant en charge les onglets personnalisés de Chrome dans Android. Dans les applications de bureau .NET Core, MSAL.NET prend uniquement en charge le navigateur du système.

## <a name="web-browsers-in-msalnet"></a>Navigateurs web dans MSAL.NET

### <a name="interaction-happens-in-a-web-browser"></a>Les interactions s'effectuent dans un navigateur web

Il est important de comprendre que lors de l'acquisition d'un jeton de manière interactive, le contenu de la boîte de dialogue n'est pas fourni par la bibliothèque, mais par le service d'émission de jeton de sécurité (STS). Le point de terminaison d’authentification retourne du code HTML et JavaScript qui contrôle l’interaction, ce qui est restitué dans un navigateur web ou un contrôle web. Autoriser le STS pour gérer l’interaction HTML présente de nombreux avantages :

- Le mot de passe (si un a été tapé) n’est jamais stocké par l’application, ni par la bibliothèque d’authentification.
- Permet les redirections vers d’autres fournisseurs d’identité (par exemple, une connexion avec un compte professionnel ou scolaire, ou un compte personnel avec MSAL, ou avec un compte social par Azure AD B2C).
- Permet au STS de contrôler l’accès conditionnel, par exemple, en obligeant l’utilisateur à passer par l’[authentification multifacteur (MFA)](../authentication/concept-mfa-howitworks.md) pendant la phase d’authentification (entrée d’un code confidentiel Windows Hello, ou réponse à un appel sur son téléphone ou sur une application d’authentification sur son téléphone). Lorsque l’authentification multifacteur demandée n’est pas encore configurée, l’utilisateur peut la configurer le moment venu dans la même boîte de dialogue.  L’utilisateur indique son numéro de téléphone mobile et reçoit des instructions pour installer une application d’authentification et lire un code QR permettant d’ajouter son compte. Cette interaction pilotée par serveur est une excellente pratique !
- Permet à l’utilisateur de modifier son mot de passe dans la même boîte de dialogue lorsque le mot de passe a expiré (en fournissant des champs supplémentaires pour l’ancien mot de passe et le nouveau mot de passe).
- Permet de personnaliser la marque du locataire, ou l’application (images) contrôlée par le propriétaire de l’application / l’administrateur de locataires Azure AD.
- Permet aux utilisateurs de donner leur consentement pour laisser l’application accéder aux ressources / étendues en leur nom, juste après l’authentification.

### <a name="embedded-vs-system-web-ui"></a>Interface utilisateur web incorporée ou système

MSAL.NET est une bibliothèque multi-infrastructures qui dispose d’un code spécifique à l’infrastructure pour héberger un navigateur dans un contrôle d’interface utilisateur (par exemple, sur .NET Classic, elle utilise WinForms, sur Xamarin, elle utilise des contrôles mobiles natifs, etc.). Ce contrôle est appelé interface utilisateur web `embedded`. MSAL.NET peut également lancer le navigateur du système d'exploitation.

En règle générale, il est recommandé d'utiliser la plateforme par défaut, qui correspond généralement au navigateur du système. Le navigateur du système mémorise mieux les utilisateurs déjà connectés. Si nécessaire, utilisez `WithUseEmbeddedWebView(bool)` pour modifier ce comportement

### <a name="at-a-glance"></a>Aperçu

| Framework        | Embedded | Système | Default |
| ------------- |-------------| -----| ----- |
| .NET Classic     | Oui | Oui^ | Embedded |
| .NET Core     | Non | Oui^ | Système |
| .NET Standard | Non | Oui^ | Système |
| UWP | Oui | Non | Embedded |
| Xamarin.Android | Oui | Oui  | Système |
| Xamarin.iOS | Oui | Oui  | Système |
| Xamarin.Mac| Oui | Non | Embedded |

^ Requiert l'URI de redirection « http://localhost  »

## <a name="system-web-browser-on-xamarinios-xamarinandroid"></a>Navigateur web du système sur Xamarin.iOS, Xamarin.Android

Par défaut, MSAL.NET prend en charge le navigateur web du système sur Xamarin.iOS, Xamarin.Android et .NET Core. Pour toutes les plateformes qui fournissent l’interface utilisateur (autrement dit, pas .NET Core), une boîte de dialogue est fournie par la bibliothèque intégrant un contrôle de navigateur web. MSAL.NET utilise également un affichage web intégré pour .NET Desktop et WAB pour la plateforme Windows universelle (UWP). Toutefois, MSAL.NET se sert par défaut du **navigateur web du système** pour les applications Xamarin iOS et Xamarin Android. Sur iOS, son choix peut même se porter sur l’affichage web à utiliser en fonction de la version du système d’exploitation (iOS12, iOS11 et versions antérieures).

L’utilisation du navigateur système présente l’énorme avantage de partager l’état SSO avec d’autres applications et applications web, sans avoir besoin d’un répartiteur (portail d’entreprise / Authenticator). Le navigateur du système a été utilisé, par défaut, dans MSAL.NET pour les plateformes Xamarin iOS et Xamarin Android car, sur ces plateformes, le navigateur web du système occupe tout l'écran et l'expérience utilisateur est meilleure. L'affichage web du système n'est pas différent de celui d'une boîte de dialogue. Sur iOS, en revanche, l’utilisateur peut être amené à devoir donner son consentement pour que le navigateur puisse rappeler l’application, ce qui peut s’avérer ennuyeux.

## <a name="system-browser-experience-on-net-core"></a>Expérience relative au navigateur du système sur .NET Core

Sur .NET Core, MSAL.NET lance le navigateur du système sous forme de processus distinct. MSAL.NET n'a aucun contrôle sur ce navigateur, mais au terme de l'authentification de l'utilisateur, la page web est redirigée de manière à ce que MSAL.NET puisse intercepter l'URI.

Vous pouvez également configurer les applications écrites pour .NET Classic de manière à ce qu'elles utilisent ce navigateur, en spécifiant

```csharp
await pca.AcquireTokenInteractive(s_scopes)
         .WithUseEmbeddedWebView(false)
```

MSAL.NET ne peut pas détecter si l'utilisateur s'absente ou ferme simplement le navigateur. Les applications utilisant cette technique sont invitées à définir un délai d'expiration (via `CancellationToken`). Nous recommandons un délai d'expiration d'au moins quelques minutes pour tenir compte des cas où l'utilisateur est invité à changer de mot de passe ou à procéder à une authentification multifacteur.

### <a name="how-to-use-the-default-os-browser"></a>Utiliser le navigateur par défaut du système d'exploitation

MSAL.NET doit écouter sur `http://localhost:port` et intercepter le code envoyé par AAD lorsque l'utilisateur a fini de s'authentifier (voir [Code d'autorisation](v2-oauth2-auth-code-flow.md) pour plus de détails).

Pour activer le navigateur du système :

1. Lors de l'inscription de l'application, configurez `http://localhost` comme URI de redirection (non prise en charge par B2C).
2. Lorsque vous générez votre application PublicClientApplication, spécifiez l'URI de redirection suivante :

```csharp
IPublicClientApplication pca = PublicClientApplicationBuilder
                            .Create("<CLIENT_ID>")
                             // or use a known port if you wish "http://localhost:1234"
                            .WithRedirectUri("http://localhost")  
                            .Build();
```

> [!Note]
> Si vous configurez `http://localhost`, MSAL.NET trouvera en interne un port ouvert aléatoire et l'utilisera.

### <a name="linux-and-mac"></a>Linux et MAC

Sous Linux, MSAL.NET ouvre le navigateur du système d'exploitation par défaut à l'aide de l'outil xdg-open. Pour y remédier, exécutez l’outil depuis un terminal, par exemple, `xdg-open "https://www.bing.com"`. Sous Mac, le navigateur est ouvert en appelant `open <url>`.

### <a name="customizing-the-experience"></a>Personnalisation de l'expérience

> [!NOTE]
> La personnalisation est disponible dans MSAL.NET 4.1.0 ou version ultérieure.

MSAL.NET peut répondre avec un message HTTP lors de la réception d'un jeton ou en cas d'erreur. Vous pouvez afficher un message HTML ou effectuer une redirection vers l'URL de votre choix :

```csharp
var options = new SystemWebViewOptions() 
{
    HtmlMessageError = "<p> An error occured: {0}. Details {1}</p>",
    BrowserRedirectSuccess = new Uri("https://www.microsoft.com");
}

await pca.AcquireTokenInteractive(s_scopes)
         .WithUseEmbeddedWebView(false)
         .WithSystemWebViewOptions(options)
         .ExecuteAsync();
```

### <a name="opening-a-specific-browser-experimental"></a>Ouverture d'un navigateur spécifique (expérimental)

Vous pouvez personnaliser la façon dont MSAL.NET ouvre le navigateur. Par exemple, au lieu d'utiliser le navigateur par défaut, vous pouvez forcer l'ouverture d'un navigateur spécifique :

```csharp
var options = new SystemWebViewOptions() 
{
    OpenBrowserAsync = SystemWebViewOptions.OpenWithEdgeBrowserAsync
}
```

### <a name="uwp-doesnt-use-the-system-webview"></a>UWP n'utilise pas System Webview

Pour les applications de bureau, cependant, lancer un affichage web du système entraîne une expérience utilisateur de moins bonne qualité, car l’utilisateur voit le navigateur, dans lequel il peut déjà avoir d’autres onglets ouverts. Ainsi, lorsque l’authentification a lieu, les utilisateurs obtiennent une page leur demandant de fermer cette fenêtre. Si l'utilisateur n'y prend pas garde, il peut fermer l'ensemble du processus (y compris les autres onglets, qui ne sont pas liés à l'authentification). Exploiter le navigateur du système sur un appareil de bureau impliquerait également d’ouvrir des ports locaux et de se mettre à l’écoute sur ces ports, ce qui peut nécessiter des autorisations avancées pour l’application. Vous, en tant que développeur, utilisateur ou administrateur, pouvez être réticent face à cette exigence.

## <a name="enable-embedded-webviews-on-ios-and-android"></a>Activer les affichages web incorporés sous iOS et Android

Vous pouvez également activer des affichages web incorporés dans les applications Xamarin.iOS et Xamarin.Android. À compter de MSAL.NET 2.0.0-preview, MSAL.NET prend également en charge l’utilisation de l’option d’affichage web **incorporé**. Pour ADAL.NET, l’affichage web incorporé est la seule option prise en charge.

En tant que développeur qui utilisez MSAL.NET ciblant Xamarin, vous avez le choix entre des affichages web incorporés ou des navigateurs de système. C’est à vous de choisir selon l’expérience utilisateur et les préoccupations en matière de sécurité que vous voulez cibler.

Actuellement, MSAL.NET ne prend pas en charge les répartiteurs iOS et Android. Par conséquent, si vous devez fournir l’authentification unique (SSO), la solution du navigateur système peut demeurer préférable. La prise en charge des répartiteurs avec le navigateur web intégré se trouve sur le backlog MSAL.NET.

### <a name="differences-between-embedded-webview-and-system-browser"></a>Différences entre l’affichage web incorporé et le navigateur du système
Il existe quelques différences visuelles entre la vue web incorporée et le navigateur du système dans MSAL.NET.

**Connexion interactive avec MSAL.NET utilisant l’affichage web incorporé :**

![affichage incorporé](media/msal-net-web-browsers/embedded-webview.png)

**Connexion interactive avec MSAL.NET utilisant le navigateur du système :**

![Navigateur du système](media/msal-net-web-browsers/system-browser.png)

### <a name="developer-options"></a>Options pour développeurs

En tant que développeur utilisant MSAL.NET, vous disposez de plusieurs options pour afficher la boîte de dialogue interactive à partir de STS :

- **Navigateur système.** Le navigateur du système est défini par défaut dans la bibliothèque. Si vous utilisez Android, consultez les [navigateurs de système](msal-net-system-browser-android-considerations.md) pour plus d’informations sur les navigateurs pris en charge pour l’authentification. Lorsque vous utilisez le navigateur système dans Android, nous vous conseillons de préférer l’appareil qui dispose d’un navigateur prenant en charge les onglets personnalisés de Chrome.  Sinon, l’authentification peut échouer.
- **Affichage web incorporé.** Pour utiliser uniquement la vue web incorporée dans MSAL.NET, le générateur de paramètres `AcquireTokenInteractively` contient une méthode `WithUseEmbeddedWebView()`.

    iOS

    ```csharp
    AuthenticationResult authResult;
    authResult = app.AcquireTokenInteractively(scopes)
                    .WithUseEmbeddedWebView(useEmbeddedWebview)
                    .ExecuteAsync();
    ```

    Android :

    ```csharp
    authResult = app.AcquireTokenInteractively(scopes)
                .WithParentActivityOrWindow(activity)
                .WithUseEmbeddedWebView(useEmbeddedWebview)
                .ExecuteAsync();
    ```

#### <a name="choosing-between-embedded-web-browser-or-system-browser-on-xamarinios"></a>Choix entre le navigateur web incorporé et le navigateur du système sur Xamarin.iOS

Dans le fichier `AppDelegate.cs` de votre application iOS, vous pouvez initialiser la valeur de `ParentWindow` sur `null`. Ce n’est pas utilisé dans iOS

```csharp
App.ParentWindow = null; // no UI parent on iOS
```

#### <a name="choosing-between-embedded-web-browser-or-system-browser-on-xamarinandroid"></a>Choix entre le navigateur web incorporé et le navigateur du système sur Xamarin.Android

Dans le fichier `MainActivity.cs` de votre application Android, vous pouvez définir l'activité parente afin que le résultat de l'authentification y soit renvoyé :

```csharp
 App.ParentWindow = this;
```

Ensuite, dans le fichier `MainPage.xaml.cs` :

```csharp
authResult = await App.PCA.AcquireTokenInteractive(App.Scopes)
                      .WithParentActivityOrWindow(App.ParentWindow)
                      .WithUseEmbeddedWebView(true)
                      .ExecuteAsync();
```

#### <a name="detecting-the-presence-of-custom-tabs-on-xamarinandroid"></a>Détection de la présence des onglets personnalisés sur Xamarin.Android

Si vous souhaitez utiliser le navigateur web du système pour activer l’authentification unique avec les applications qui s’exécutent dans le navigateur, mais que vous êtes soucieux par rapport à l’expérience utilisateur des appareils Android qui ne disposent pas de navigateur avec prise en charge des onglets personnalisés, vous avez la possibilité de décider en appelant la méthode `IsSystemWebViewAvailable()` dans `IPublicClientApplication`. Cette méthode renvoie `true` si PackageManager détecte des onglets personnalisés, et `false` s'ils ne sont pas détectés sur l'appareil.

Selon la valeur retournée par cette méthode, et vos exigences, vous pouvez prendre une décision :

- Vous pouvez retourner un message d’erreur personnalisé à l’utilisateur. Par exemple : « Veuillez installer Chrome pour poursuivre l’authentification » OU
- Vous pouvez revenir à l’option de l’affichage web incorporé et lancer l’interface utilisateur en tant que vue web incorporée.

Le code ci-dessous montre l’option d’affichage web incorporé :

```csharp
bool useSystemBrowser = app.IsSystemWebviewAvailable();

authResult = await App.PCA.AcquireTokenInteractive(App.Scopes)
                      .WithParentActivityOrWindow(App.ParentWindow)
                      .WithUseEmbeddedWebView(!useSystemBrowser)
                      .ExecuteAsync();
```

#### <a name="net-core-doesnt-support-interactive-authentication-with-an-embedded-browser"></a>.NET Core ne prend pas en charge l'authentification interactive auprès d'un navigateur incorporé

Pour .NET Core, l'acquisition interactive de jetons ne peut s'effectuer que via le navigateur web du système, et non avec les affichages web incorporés. En effet, .NET Core ne fournit pas encore d'interface utilisateur.
Si vous souhaitez personnaliser l'expérience de navigation avec le navigateur web du système, vous pouvez implémenter l'interface [IWithCustomUI](scenario-desktop-acquire-token.md#withcustomwebui) et même fournir votre propre navigateur.
