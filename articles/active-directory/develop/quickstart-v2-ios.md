---
title: 'Démarrage rapide : Ajouter la connexion Microsoft à une application iOS ou macOS | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce guide de démarrage rapide, découvrez comment une application iOS ou macOS peut connecter des utilisateurs, obtenir un jeton d’accès auprès de la plateforme d’identités Microsoft et appeler l’API Microsoft Graph.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 09/24/2019
ms.author: marsma
ms.reviewer: jmprieur, saeeda
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started, languages:iOS
ms.openlocfilehash: d1a3965fef6966f70a829cd66d6ce10a01d7af98
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97030890"
---
# <a name="quickstart-sign-in-users-and-call-the-microsoft-graph-api-from-an-ios-or-macos-app"></a>Démarrage rapide : Connecter des utilisateurs et appeler l’API Microsoft Graph à partir d’une application iOS ou macOS

Dans ce guide de démarrage rapide, vous téléchargez et exécutez un exemple de code qui montre comment une application iOS ou macOS native peut connecter des utilisateurs et obtenir un jeton d’accès pour appeler l’API Microsoft Graph.

Ce guide de démarrage rapide s’applique aux applications iOS et macOS. Certaines étapes sont nécessaires uniquement pour les applications iOS et seront indiquées comme telles.

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* XCode 10+
* iOS 10+
* macOS 10.12+

## <a name="how-the-sample-works"></a>Fonctionnement de l’exemple

![Fonctionnement de l’exemple d’application généré par ce guide de démarrage rapide](media/quickstart-v2-ios/ios-intro.svg)

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-app"></a>Inscrire et télécharger votre application de démarrage rapide
> Vous disposez de deux options pour démarrer votre application de démarrage rapide :
> * [Express] [Option 1 : Inscrire et configurer automatiquement votre application, puis télécharger votre exemple de code](#option-1-register-and-auto-configure-your-app-and-then-download-the-code-sample)
> * [Manuel] [Option 2 : Inscrire et configurer manuellement vos application et exemple de code](#option-2-register-and-manually-configure-your-application-and-code-sample)
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-the-code-sample"></a>Option 1 : Inscrire et configurer automatiquement votre application, puis télécharger l’exemple de code
> #### <a name="step-1-register-your-application"></a>Étape 1 : Inscrivez votre application
> Pour inscrire votre application :
> 1. Accédez au nouveau volet [Portail Azure - Inscriptions des applications](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/IosQuickstartPage/sourceType/docs).
> 1. Entrez un nom pour votre application, puis sélectionnez **Inscrire**.
> 1. Suivez les instructions pour télécharger et configurer automatiquement votre nouvelle application en un seul clic.
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>Option n°2 : Inscrire et configurer manuellement vos application et exemple de code
>
> #### <a name="step-1-register-your-application"></a>Étape 1 : Inscrivez votre application
> Pour inscrire votre application et ajouter manuellement les informations d’inscription de l’application à votre solution, procédez comme suit :
>
> 1. Connectez-vous au [portail Azure](https://portal.azure.com).
> 1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
> 1. Recherchez et sélectionnez **Azure Active Directory**.    
> 1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.
> 1. Entrez un **nom** pour votre application. Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
> 1. Sélectionnez **Inscription**.
> 1. Sous **Gérer**, sélectionnez **Authentification** > **Ajouter une plateforme** > **iOS**.
> 1. Entrez l'**identificateur d'offre groupée** de votre application. L’identificateur de bundle est une chaîne unique qui identifie de façon unique votre application, par exemple `com.<yourname>.identitysample.MSALMacOS`. Prenez note de la valeur que vous utilisez. Notez que la configuration iOS s’applique également aux applications macOS.
> 1. Sélectionnez **Configurer** et enregistrez les détails de la **configuration MSAL** pour une utilisation ultérieure dans ce guide de démarrage rapide.
> 1. Sélectionnez **Terminé**.

> [!div renderon="portal" class="sxs-lookup"]
>
> #### <a name="step-1-configure-your-application"></a>Étape 1 : Configuration de votre application
> Pour que l'exemple de code de ce guide de démarrage rapide fonctionne, vous devez ajouter un URI de redirection compatible avec le répartiteur d'authentification.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Apporter cette modification pour moi]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Déjà configuré](media/quickstart-v2-ios/green-check.png) Votre application est configurée avec ces attributs
>
> #### <a name="step-2-download-the-sample-project"></a>Étape 2 : Téléchargement de l’exemple de projet
> > [!div id="autoupdate_ios" class="nextstepaction"]
> > [Télécharger l’exemple de code pour iOS]()
>
> > [!div id="autoupdate_macos" class="nextstepaction"]
> > [Télécharger l’exemple de code pour macOS]()
> [!div renderon="docs"]
> #### <a name="step-2-download-the-sample-project"></a>Étape 2 : Téléchargement de l’exemple de projet
>
> - [Télécharger l’exemple de code pour iOS](https://github.com/Azure-Samples/active-directory-ios-swift-native-v2/archive/master.zip)
> - [Télécharger l’exemple de code pour macOS](https://github.com/Azure-Samples/active-directory-macOS-swift-native-v2/archive/master.zip)

#### <a name="step-3-install-dependencies"></a>Étape 3 : Installer des dépendances

Dans une fenêtre de terminal, accédez au dossier à l’aide de l’exemple de code téléchargé et exécutez `pod install` pour installer la bibliothèque MSAL la plus récente.

> [!div renderon="portal" class="sxs-lookup"]
> #### <a name="step-4-your-app-is-configured-and-ready-to-run"></a>Étape 4 : Votre application est configurée et prête à être exécutée
> Nous avons configuré votre projet avec les valeurs des propriétés de votre application et il est prêt à être exécuté.
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`
>
> [!div renderon="docs"]
>#### <a name="step-4-configure-your-project"></a>Étape 4 : Configurer votre projet
> Si vous avez sélectionné l’option 1 ci-dessus, vous pouvez ignorer ces étapes.
> 1. Extrayez le fichier zip et ouvrez le projet dans XCode.
> 1. Modifiez **ViewController.swift** et remplacez la ligne commençant par « let kClientID » par l’extrait de code suivant. N’oubliez pas de mettre à jour la valeur de `kClientID` avec l’ID client que vous avez enregistré lors de l’inscription de votre application dans le portail, plus haut dans ce guide de démarrage rapide :
>    ```swift
>    let kClientID = "Enter_the_Application_Id_Here"
>    ```
> 1. Si vous créez une application pour les [clouds nationaux Azure AD](/graph/deployments#app-registration-and-token-service-root-endpoints), remplacez la ligne commençant par « let kGraphEndpoint » et « let kAuthority » par des points de terminaison corrects. Pour un accès global, utilisez les valeurs par défaut :
>     ```swift
>     let kGraphEndpoint = "https://graph.microsoft.com/"
>     let kAuthority = "https://login.microsoftonline.com/common"
>     ```
> 1. D’autres points de terminaison sont décrits [ici](/graph/deployments#app-registration-and-token-service-root-endpoints). Par exemple, pour exécuter le guide de démarrage rapide avec Azure AD Allemagne, utilisez la commande suivante :
>     ```swift
>     let kGraphEndpoint = "https://graph.microsoft.de/"
>     let kAuthority = "https://login.microsoftonline.de/common"
>     ```
> 1. Ouvrez les paramètres du projet. Dans la section **Identité**, entrez l’**Identificateur de bundle** que vous avez indiqué dans le portail.
> 1. Cliquez avec le bouton droit sur **Info.plist** et sélectionnez **Ouvrir en tant que** > **Code source**.
> 1. Sous le nœud racine du dictionnaire, remplacez `Enter_the_bundle_Id_Here` par l’**_ID de bundle_* _ que vous avez utilisé dans le portail.
>
>    ```xml
>    <key>CFBundleURLTypes</key>
>    <array>
>       <dict>
>          <key>CFBundleURLSchemes</key>
>          <array>
>             <string>msauth.Enter_the_Bundle_Id_Here</string>
>          </array>
>       </dict>
>    </array>
>
>    ```
> 1. Générez et exécutez l'application !

## <a name="more-information"></a>Informations complémentaires

Parcourez ces sections pour en savoir plus sur ce démarrage rapide.

### <a name="get-msal"></a>Obtenir MSAL

MSAL ([MSAL.framework](https://github.com/AzureAD/microsoft-authentication-library-for-objc)) est la bibliothèque utilisée pour connecter les utilisateurs et demander des jetons permettant d'accéder à une API protégée par la plateforme d'identités Microsoft. Vous pouvez ajouter MSAL à votre application en suivant le processus ci-après :

```
$ vi Podfile

```
Ajoutez ce qui suit à ce podfile (avec la cible de votre projet) :

```
use_frameworks!

target 'MSALiOS' do
   pod 'MSAL'
end

```

Exécutez la commande d’installation de CocoaPods :

```pod install```

### <a name="initialize-msal"></a>Initialiser MSAL

Vous pouvez ajouter la référence de MSAL en ajoutant le code suivant :

```swift
import MSAL
```

Ensuite, initialisez MSAL à l’aide du code suivant :

```swift
let authority = try MSALAADAuthority(url: URL(string: kAuthority)!)

let msalConfiguration = MSALPublicClientApplicationConfig(clientId: kClientID, redirectUri: nil, authority: authority)
self.applicationContext = try MSALPublicClientApplication(configuration: msalConfiguration)
```

> |Où : | Description |
> |---------|---------|
> | `clientId` | ID d’application de l’application inscrite dans _portal.azure.com* |
> | `authority` | Point de terminaison de la plateforme d’identités Microsoft. Dans la plupart des cas, ce sera `https://login.microsoftonline.com/common` |
> | `redirectUri` | URI de redirection de l'application. Vous pouvez passer « nil » pour utiliser la valeur par défaut, ou votre URI de redirection personnalisé. |

### <a name="for-ios-only-additional-app-requirements"></a>Pour iOS uniquement, conditions supplémentaires pour les applications

Votre application doit également comporter ce qui suit dans la propriété `AppDelegate`. Cela permet au kit SDK MSAL de gérer la réponse des jetons à partir de l’application du répartiteur d’authentification, au moment de l’authentification.

 ```swift
 func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {

        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
    }

 ```

> [!NOTE]
> Sur iOS 13+, si vous adoptez `UISceneDelegate` au lieu de `UIApplicationDelegate`, placez ce code dans le rappel `scene:openURLContexts:` à la place (consultez la [documentation Apple](https://developer.apple.com/documentation/uikit/uiscenedelegate/3238059-scene?language=objc)).
> Si vous prenez en charge à la fois UISceneDelegate et UIApplicationDelegate pour assurer la compatibilité avec une version plus ancienne d’iOS, le rappel MSAL doit être placé aux deux endroits.

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

Enfin, votre application doit comporter une entrée `LSApplicationQueriesSchemes` dans la liste ***Info.plist** _ en plus des `CFBundleURLTypes`. Ceci est inclus dans l'exemple fourni.

   ```xml
   <key>LSApplicationQueriesSchemes</key>
   <array>
      <string>msauthv2</string>
      <string>msauthv3</string>
   </array>
   ```

### <a name="sign-in-users--request-tokens"></a>Connexion des utilisateurs et demandes de jetons

MSAL utilise deux méthodes pour acquérir des jetons : `acquireToken` et `acquireTokenSilent`.

#### <a name="acquiretoken-get-a-token-interactively"></a>acquireToken : Obtenir un jeton de manière interactive

Certaines situations nécessitent l'interaction des utilisateurs avec la Plateforme d'identités Microsoft. Dans ce cas, l'utilisateur final peut être amené à sélectionner son compte, à saisir ses informations d'identification ou à accepter les autorisations relatives à votre application. Par exemple,

_ Lorsque des utilisateurs se connectent pour la première fois à l’application
* Si un utilisateur réinitialise son mot de passe, il doit entrer ses informations d’identification
* Lorsque votre application demande l'accès à une ressource pour la première fois
* Lorsque l'authentification multifacteur ou d'autres stratégies d'accès conditionnel sont requises

```swift
let parameters = MSALInteractiveTokenParameters(scopes: kScopes, webviewParameters: self.webViewParamaters!)
self.applicationContext!.acquireToken(with: parameters) { (result, error) in /* Add your handling logic */}
```

> |Où :| Description |
> |---------|---------|
> | `scopes` | Contient les étendues demandées (c’est-à-dire `[ "user.read" ]` pour Microsoft Graph ou `[ "<Application ID URL>/scope" ]` pour les API web personnalisées (`api://<Application ID>/access_as_user`) |

#### <a name="acquiretokensilent-get-an-access-token-silently"></a>acquireTokenSilent : Obtenir un jeton d’accès en mode silencieux

Les applications ne doivent pas demander aux utilisateurs de se connecter chaque fois qu'elles ont besoin d'un jeton. Si l'utilisateur est déjà connecté, cette méthode permet aux applications demander des jetons en mode silencieux.

```swift
self.applicationContext!.getCurrentAccount(with: nil) { (currentAccount, previousAccount, error) in

   guard let account = currentAccount else {
      return
   }

   let silentParams = MSALSilentTokenParameters(scopes: self.kScopes, account: account)
   self.applicationContext!.acquireTokenSilent(with: silentParams) { (result, error) in /* Add your handling logic */}
}
```

> |Où : | Description |
> |---------|---------|
> | `scopes` | Contient les étendues demandées (c’est-à-dire `[ "user.read" ]` pour Microsoft Graph ou `[ "<Application ID URL>/scope" ]` pour les API web personnalisées (`api://<Application ID>/access_as_user`) |
> | `account` | Compte pour lequel un jeton est demandé. Ce guide de démarrage rapide concerne une application monocompte. Si vous souhaitez créer une application multicompte, vous devez définir une logique permettant d’identifier le compte à utiliser pour les demandes de jetons à l’aide de `accountsFromDeviceForParameters:completionBlock:` et de transmettre le `accountIdentifier` correct. |

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Étapes suivantes

Passez au tutoriel pas à pas dans lequel vous créez une application iOS ou macOS qui obtient un jeton d’accès auprès de la plateforme d’identités Microsoft et l’utilise pour appeler l’API Microsoft Graph.

> [!div class="nextstepaction"]
> [Tutoriel : Connecter des utilisateurs et appeler Microsoft Graph à partir d’une application iOS ou macOS](tutorial-v2-ios.md)
