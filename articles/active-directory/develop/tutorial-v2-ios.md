---
title: 'Tutoriel : Créer une application iOS ou macOS qui utilise la plateforme d’identités Microsoft pour l’authentification | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce tutoriel, vous allez créer une application iOS ou macOS qui utilise la plateforme d’identités Microsoft pour connecter les utilisateurs et obtenir un jeton d’accès pour appeler l’API Microsoft Graph en leur nom.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 09/18/2020
ms.author: marsma
ms.reviewer: oldalton
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: f04a8aa96b51ac9330e4302c3afcc48f7d305b39
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507708"
---
# <a name="tutorial-sign-in-users-and-call-microsoft-graph-from-an-ios-or-macos-app"></a>Tutoriel : Connecter des utilisateurs et appeler Microsoft Graph à partir d’une application iOS ou macOS

Dans ce tutoriel, vous allez créer une application iOS ou macOS qui s’intègre à la plateforme d’identités Microsoft afin de connecter des utilisateurs et obtenir un jeton d’accès pour appeler l’API Microsoft Graph.

À la fin de ce guide, votre application acceptera les connexions de comptes Microsoft personnels (y compris outlook.com, live.com et d’autres) et de comptes professionnels ou scolaires de n’importe quelle entreprise ou organisation utilisant Azure Active Directory. Ce tutoriel est applicable aux applications iOS et macOS. Certaines étapes varient d’une plateforme à l’autre.

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Créer un projet d’application iOS ou macOS dans *Xcode*.
> * Inscrire l'application sur le portail Azure
> * Ajouter du code pour prendre en charge la connexion et la déconnexion des utilisateurs
> * Ajouter du code pour appeler l’API Microsoft Graph.
> * Test de l'application

## <a name="prerequisites"></a>Prérequis

- [Xcode 11.x+](https://developer.apple.com/xcode/)

## <a name="how-tutorial-app-works"></a>Fonctionnement de l’application du tutoriel

![Fonctionnement de l’exemple d’application généré par ce tutoriel](../../../includes/media/active-directory-develop-guidedsetup-ios-introduction/iosintro.svg)

L’application utilisée dans ce tutoriel peut connecter des utilisateurs et obtenir des données de Microsoft Graph en leur nom. Ces données sont accessibles par le biais d’une API protégée (l’API Microsoft Graph dans ce cas précis) qui nécessite une autorisation et est protégée par la plateforme d’identités Microsoft.

Plus précisément :

* Votre application connectera l’utilisateur via un navigateur ou Microsoft Authenticator.
* L’utilisateur final acceptera les autorisations que votre application a demandées.
* Votre application émet un jeton d’accès pour l’API Microsoft Graph.
* Le jeton d’accès est inclus dans la requête HTTP adressée à l’API web.
* La réponse Microsoft Graph est traitée.

Cet exemple utilise la bibliothèque d’authentification Microsoft (MSAL) pour implémenter l’authentification. MSAL renouvelle automatiquement les jetons, fournit une authentification unique entre les autres applications de l’appareil et gère les comptes.

Si vous souhaitez télécharger une version complète de l’application que vous avez générée dans ce tutoriel, vous trouverez les deux versions sur GitHub :

- [Exemple de code iOS](https://github.com/Azure-Samples/active-directory-ios-swift-native-v2/) (GitHub)
- [Exemple de code macOS](https://github.com/Azure-Samples/active-directory-macOS-swift-native-v2/) (GitHub)

## <a name="create-a-new-project"></a>Création d'un projet

1. Ouvrez Xcode et sélectionnez **Create a new Xcode project** (Créer un projet Xcode).
2. Pour les applications iOS, sélectionnez **iOS** > **Application avec affichage unique**, puis **Suivant**.
3. Pour les applications macOS, sélectionnez **macOS** > **Application Cocoa**, puis **Suivant**.
4. Spécifiez un nom de produit.
5. Choisissez **Swift** comme **Langage**, puis sélectionnez **Suivant**.
6. Sélectionnez un dossier où créer votre application, puis sélectionnez **Créer**.

## <a name="register-your-application"></a>Inscrivez votre application

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.
1. Entrez un **nom** pour votre application. Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
1. Sélectionnez **Comptes dans un annuaire d’organisation (tout annuaire Azure AD - Multilocataire) et comptes Microsoft personnels (par exemple, Skype, Xbox)** sous **Types de comptes pris en charge**.
1. Sélectionnez **Inscription**.
1. Sous **Gérer**, sélectionnez **Authentification** > **Ajouter une plateforme** > **iOS/macOS**.
1. Entrez l’ID de bundle de votre projet. Si vous avez téléchargé le code, cette valeur est `com.microsoft.identitysample.MSALiOS`. Si vous créez votre propre projet, sélectionnez-le dans Xcode et ouvrez l’onglet **Général**. L’identificateur de bundle apparaît dans la section **Identité**.
1. Sélectionnez **Configurer** et enregistrez la **Configuration MSAL** qui apparaît dans la page **Configuration MSAL** pour pouvoir l’entrer plus tard quand vous devrez configurer votre application. 
1. Sélectionnez **Terminé**.

## <a name="add-msal"></a>Ajouter MSAL

Choisissez l’une des méthodes suivantes pour installer la bibliothèque MSAL dans votre application :

### <a name="cocoapods"></a>CocoaPods

1. Si vous utilisez [CocoaPods](https://cocoapods.org/), installez `MSAL` en commençant par créer un fichier vide nommé `podfile` dans le même dossier que le fichier `.xcodeproj` de votre projet. Ajoutez la ligne suivante à `podfile` :

   ```
   use_frameworks!

   target '<your-target-here>' do
      pod 'MSAL'
   end
   ```

2. Remplacez `<your-target-here>` par le nom de votre projet.
3. Dans une fenêtre de terminal, accédez au dossier qui contient le `podfile` que vous avez créé et exécutez `pod install` pour installer la bibliothèque MSAL.
4. Fermez Xcode et ouvrez `<your project name>.xcworkspace` pour recharger le projet dans Xcode.

### <a name="carthage"></a>Carthage

Si vous utilisez [Carthage](https://github.com/Carthage/Carthage), installez `MSAL` en l’ajoutant à votre `Cartfile` :

```
github "AzureAD/microsoft-authentication-library-for-objc" "master"
```

À partir d’une fenêtre de terminal, dans le même répertoire que le `Cartfile` mis à jour, exécutez la commande suivante pour que Carthage mette à jour les dépendances dans votre projet.

iOS :

```bash
carthage update --platform iOS
```

macOS :

```bash
carthage update --platform macOS
```

### <a name="manually"></a>Manuellement

Vous pouvez également utiliser Git Submodule ou consulter la dernière version pour l’utiliser comme framework dans votre application.

## <a name="add-your-app-registration"></a>Ajouter votre inscription d’application

Ensuite, nous allons ajouter votre inscription d’application à votre code.

Tout d’abord, ajoutez l’instruction import suivante en haut des fichiers `ViewController.swift` ainsi que `AppDelegate.swift` ou `SceneDelegate.swift` :

```swift
import MSAL
```

Ajoutez ensuite le code suivant à `ViewController.swift` avant `viewDidLoad()` :

```swift
// Update the below to your client ID you received in the portal. The below is for running the demo only
let kClientID = "Your_Application_Id_Here"
let kGraphEndpoint = "https://graph.microsoft.com/" // the Microsoft Graph endpoint
let kAuthority = "https://login.microsoftonline.com/common" // this authority allows a personal Microsoft account and a work or school account in any organization's Azure AD tenant to sign in

let kScopes: [String] = ["user.read"] // request permission to read the profile of the signed-in user

var accessToken = String()
var applicationContext : MSALPublicClientApplication?
var webViewParameters : MSALWebviewParameters?
var currentAccount: MSALAccount?
```

La seule valeur que vous devez modifier ci-dessus est la valeur attribuée à `kClientID` comme [ID d’application](./developer-glossary.md#application-id-client-id). Cette valeur fait partie des données de configuration MSAL que vous avez enregistrées au début de ce tutoriel pour inscrire l’application dans le portail Azure.

## <a name="configure-xcode-project-settings"></a>Configurer les paramètres d’un projet Xcode

Ajoutez un nouveau groupe de trousseaux à **Signature et fonctionnalités** pour votre projet. Le groupe de trousseaux doit être `com.microsoft.adalcache` sur iOS et `com.microsoft.identity.universalstorage` sur macOS.

![Interface utilisateur Xcode montrant comment le groupe de trousseaux doit être configuré](../../../includes/media/active-directory-develop-guidedsetup-ios-introduction/iosintro-keychainShare.png)

## <a name="for-ios-only-configure-url-schemes"></a>Pour iOS uniquement, configurer des schémas d’URL

Lors de cette étape, vous allez inscrire `CFBundleURLSchemes` afin que l’utilisateur puisse être redirigé vers l’application après la connexion. Notez que `LSApplicationQueriesSchemes` permet également à votre application d’utiliser Microsoft Authenticator.

Dans Xcode, ouvrez `Info.plist` en tant que fichier de code source et ajoutez ce qui suit dans la section `<dict>`. Remplacez `[BUNDLE_ID]` par la valeur que vous avez utilisée dans le portail Azure. Si vous avez téléchargé le code, l’identificateur de bundle est `com.microsoft.identitysample.MSALiOS`. Si vous créez votre propre projet, sélectionnez-le dans Xcode et ouvrez l’onglet **Général**. L’identificateur de bundle apparaît dans la section **Identité**.

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
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>msauthv2</string>
    <string>msauthv3</string>
</array>
```

## <a name="for-macos-only-configure-app-sandbox"></a>Pour macOS uniquement, configurer App Sandbox

1. Accédez à vos paramètres de projet Xcode > **Onglet Capacités** > **App Sandbox**.
2. Cochez la case **Connexions sortantes (client)** .

## <a name="create-your-apps-ui"></a>Créer l’interface utilisateur de votre application

À présent, créez une interface utilisateur qui comprend un bouton pour appeler l’API Microsoft Graph, un autre pour se déconnecter, et une vue texte pour afficher une sortie en ajoutant le code suivant à la classe `ViewController` :

### <a name="ios-ui"></a>Interface utilisateur iOS

```swift
var loggingText: UITextView!
var signOutButton: UIButton!
var callGraphButton: UIButton!
var usernameLabel: UILabel!

func initUI() {

    usernameLabel = UILabel()
    usernameLabel.translatesAutoresizingMaskIntoConstraints = false
    usernameLabel.text = ""
    usernameLabel.textColor = .darkGray
    usernameLabel.textAlignment = .right

    self.view.addSubview(usernameLabel)

    usernameLabel.topAnchor.constraint(equalTo: view.topAnchor, constant: 50.0).isActive = true
    usernameLabel.rightAnchor.constraint(equalTo: view.rightAnchor, constant: -10.0).isActive = true
    usernameLabel.widthAnchor.constraint(equalToConstant: 300.0).isActive = true
    usernameLabel.heightAnchor.constraint(equalToConstant: 50.0).isActive = true

    // Add call Graph button
    callGraphButton  = UIButton()
    callGraphButton.translatesAutoresizingMaskIntoConstraints = false
    callGraphButton.setTitle("Call Microsoft Graph API", for: .normal)
    callGraphButton.setTitleColor(.blue, for: .normal)
    callGraphButton.addTarget(self, action: #selector(callGraphAPI(_:)), for: .touchUpInside)
    self.view.addSubview(callGraphButton)

    callGraphButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    callGraphButton.topAnchor.constraint(equalTo: view.topAnchor, constant: 120.0).isActive = true
    callGraphButton.widthAnchor.constraint(equalToConstant: 300.0).isActive = true
    callGraphButton.heightAnchor.constraint(equalToConstant: 50.0).isActive = true

    // Add sign out button
    signOutButton = UIButton()
    signOutButton.translatesAutoresizingMaskIntoConstraints = false
    signOutButton.setTitle("Sign Out", for: .normal)
    signOutButton.setTitleColor(.blue, for: .normal)
    signOutButton.setTitleColor(.gray, for: .disabled)
    signOutButton.addTarget(self, action: #selector(signOut(_:)), for: .touchUpInside)
    self.view.addSubview(signOutButton)

    signOutButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    signOutButton.topAnchor.constraint(equalTo: callGraphButton.bottomAnchor, constant: 10.0).isActive = true
    signOutButton.widthAnchor.constraint(equalToConstant: 150.0).isActive = true
    signOutButton.heightAnchor.constraint(equalToConstant: 50.0).isActive = true

    let deviceModeButton = UIButton()
    deviceModeButton.translatesAutoresizingMaskIntoConstraints = false
    deviceModeButton.setTitle("Get device info", for: .normal);
    deviceModeButton.setTitleColor(.blue, for: .normal);
    deviceModeButton.addTarget(self, action: #selector(getDeviceMode(_:)), for: .touchUpInside)
    self.view.addSubview(deviceModeButton)

    deviceModeButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    deviceModeButton.topAnchor.constraint(equalTo: signOutButton.bottomAnchor, constant: 10.0).isActive = true
    deviceModeButton.widthAnchor.constraint(equalToConstant: 150.0).isActive = true
    deviceModeButton.heightAnchor.constraint(equalToConstant: 50.0).isActive = true

    // Add logging textfield
    loggingText = UITextView()
    loggingText.isUserInteractionEnabled = false
    loggingText.translatesAutoresizingMaskIntoConstraints = false

    self.view.addSubview(loggingText)

    loggingText.topAnchor.constraint(equalTo: deviceModeButton.bottomAnchor, constant: 10.0).isActive = true
    loggingText.leftAnchor.constraint(equalTo: self.view.leftAnchor, constant: 10.0).isActive = true
    loggingText.rightAnchor.constraint(equalTo: self.view.rightAnchor, constant: -10.0).isActive = true
    loggingText.bottomAnchor.constraint(equalTo: self.view.bottomAnchor, constant: 10.0).isActive = true
}

func platformViewDidLoadSetup() {

    NotificationCenter.default.addObserver(self,
                        selector: #selector(appCameToForeGround(notification:)),
                        name: UIApplication.willEnterForegroundNotification,
                        object: nil)

}

@objc func appCameToForeGround(notification: Notification) {
    self.loadCurrentAccount()
}

```

### <a name="macos-ui"></a>Interface utilisateur macOS

```swift

var callGraphButton: NSButton!
var loggingText: NSTextView!
var signOutButton: NSButton!

var usernameLabel: NSTextField!

func initUI() {

    usernameLabel = NSTextField()
    usernameLabel.translatesAutoresizingMaskIntoConstraints = false
    usernameLabel.stringValue = ""
    usernameLabel.isEditable = false
    usernameLabel.isBezeled = false
    self.view.addSubview(usernameLabel)

    usernameLabel.topAnchor.constraint(equalTo: view.topAnchor, constant: 30.0).isActive = true
    usernameLabel.rightAnchor.constraint(equalTo: view.rightAnchor, constant: -10.0).isActive = true

    // Add call Graph button
    callGraphButton  = NSButton()
    callGraphButton.translatesAutoresizingMaskIntoConstraints = false
    callGraphButton.title = "Call Microsoft Graph API"
    callGraphButton.target = self
    callGraphButton.action = #selector(callGraphAPI(_:))
    callGraphButton.bezelStyle = .rounded
    self.view.addSubview(callGraphButton)

    callGraphButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    callGraphButton.topAnchor.constraint(equalTo: view.topAnchor, constant: 50.0).isActive = true
    callGraphButton.heightAnchor.constraint(equalToConstant: 34.0).isActive = true

    // Add sign out button
    signOutButton = NSButton()
    signOutButton.translatesAutoresizingMaskIntoConstraints = false
    signOutButton.title = "Sign Out"
    signOutButton.target = self
    signOutButton.action = #selector(signOut(_:))
    signOutButton.bezelStyle = .texturedRounded
    self.view.addSubview(signOutButton)

    signOutButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    signOutButton.topAnchor.constraint(equalTo: callGraphButton.bottomAnchor, constant: 10.0).isActive = true
    signOutButton.heightAnchor.constraint(equalToConstant: 34.0).isActive = true
    signOutButton.isEnabled = false

    // Add logging textfield
    loggingText = NSTextView()
    loggingText.translatesAutoresizingMaskIntoConstraints = false

    self.view.addSubview(loggingText)

    loggingText.topAnchor.constraint(equalTo: signOutButton.bottomAnchor, constant: 10.0).isActive = true
    loggingText.leftAnchor.constraint(equalTo: self.view.leftAnchor, constant: 10.0).isActive = true
    loggingText.rightAnchor.constraint(equalTo: self.view.rightAnchor, constant: -10.0).isActive = true
    loggingText.bottomAnchor.constraint(equalTo: self.view.bottomAnchor, constant: -10.0).isActive = true
    loggingText.widthAnchor.constraint(equalToConstant: 500.0).isActive = true
    loggingText.heightAnchor.constraint(equalToConstant: 300.0).isActive = true
}

func platformViewDidLoadSetup() {}

```

Ensuite, à l’intérieur de la classe `ViewController`, remplacez la méthode `viewDidLoad()` par :

```swift
    override func viewDidLoad() {

        super.viewDidLoad()

        initUI()

        do {
            try self.initMSAL()
        } catch let error {
            self.updateLogging(text: "Unable to create Application Context \(error)")
        }

        self.loadCurrentAccount()
        self.platformViewDidLoadSetup()
    }
```

## <a name="use-msal"></a>Utiliser MSAL

### <a name="initialize-msal"></a>Initialiser MSAL

Ajoutez la méthode `initMSAL` suivante à la classe `ViewController` :

```swift
    func initMSAL() throws {

        guard let authorityURL = URL(string: kAuthority) else {
            self.updateLogging(text: "Unable to create authority URL")
            return
        }

        let authority = try MSALAADAuthority(url: authorityURL)

        let msalConfiguration = MSALPublicClientApplicationConfig(clientId: kClientID, redirectUri: nil, authority: authority)
        self.applicationContext = try MSALPublicClientApplication(configuration: msalConfiguration)
        self.initWebViewParams()
    }
```

Ajoutez ce qui suit après la méthode `ViewController` dans la classe `initMSAL`.

### <a name="ios-code"></a>Code iOS :

```swift
func initWebViewParams() {
        self.webViewParameters = MSALWebviewParameters(authPresentationViewController: self)
    }
```

### <a name="macos-code"></a>Code macOS :

```swift
func initWebViewParams() {
        self.webViewParameters = MSALWebviewParameters()
    }
```

### <a name="for-ios-only-handle-the-sign-in-callback"></a>Pour iOS uniquement, gérer le rappel de connexion

Ouvrez le fichier `AppDelegate.swift` . Pour gérer le rappel après la connexion, ajoutez `MSALPublicClientApplication.handleMSALResponse` à la classe `appDelegate` comme suit :

```swift
// Inside AppDelegate...
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {

        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
}

```

**Si vous utilisez Xcode 11**, vous devez placer le rappel MSAL dans `SceneDelegate.swift`.
Si vous prenez en charge à la fois UISceneDelegate et UIApplicationDelegate pour assurer la compatibilité avec une version plus ancienne d’iOS, le rappel MSAL doit être placé dans les deux fichiers.

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

#### <a name="acquire-tokens"></a>Acquérir des jetons

Nous pouvons maintenant implémenter la logique de traitement de l’interface utilisateur de l’application et obtenir des jetons de manière interactive par le biais de MSAL.

MSAL expose deux méthodes principales pour obtenir des jetons : `acquireTokenSilently()` et `acquireTokenInteractively()` :

- `acquireTokenSilently()` tente de connecter un utilisateur et d’obtenir des jetons sans aucune intervention de l’utilisateur tant qu’un compte est présent. `acquireTokenSilently()` nécessite de fournir un `MSALAccount` valide qui peut être récupéré en utilisant une des API d’énumération de comptes MSAL. Cet exemple utilise `applicationContext.getCurrentAccount(with: msalParameters, completionBlock: {})` pour récupérer le compte actuel.

- `acquireTokenInteractively()` affiche toujours l’interface utilisateur lors de la tentative de connexion de l’utilisateur. Elle peut utiliser des cookies de session dans le navigateur ou un compte dans Microsoft Authenticator pour fournir une expérience d’authentification unique interactive.

Ajoutez le code suivant à la classe `ViewController` :

```swift
    func getGraphEndpoint() -> String {
        return kGraphEndpoint.hasSuffix("/") ? (kGraphEndpoint + "v1.0/me/") : (kGraphEndpoint + "/v1.0/me/");
    }

    @objc func callGraphAPI(_ sender: AnyObject) {

        self.loadCurrentAccount { (account) in

            guard let currentAccount = account else {

                // We check to see if we have a current logged in account.
                // If we don't, then we need to sign someone in.
                self.acquireTokenInteractively()
                return
            }

            self.acquireTokenSilently(currentAccount)
        }
    }

    typealias AccountCompletion = (MSALAccount?) -> Void

    func loadCurrentAccount(completion: AccountCompletion? = nil) {

        guard let applicationContext = self.applicationContext else { return }

        let msalParameters = MSALParameters()
        msalParameters.completionBlockQueue = DispatchQueue.main

        applicationContext.getCurrentAccount(with: msalParameters, completionBlock: { (currentAccount, previousAccount, error) in

            if let error = error {
                self.updateLogging(text: "Couldn't query current account with error: \(error)")
                return
            }

            if let currentAccount = currentAccount {

                self.updateLogging(text: "Found a signed in account \(String(describing: currentAccount.username)). Updating data for that account...")

                self.updateCurrentAccount(account: currentAccount)

                if let completion = completion {
                    completion(self.currentAccount)
                }

                return
            }

            self.updateLogging(text: "Account signed out. Updating UX")
            self.accessToken = ""
            self.updateCurrentAccount(account: nil)

            if let completion = completion {
                completion(nil)
            }
        })
    }
```

#### <a name="get-a-token-interactively"></a>Obtenir un jeton de manière interactive

L’extrait de code suivant obtient un jeton pour la première fois en créant un objet `MSALInteractiveTokenParameters` et en appelant `acquireToken`. Ensuite, vous allez ajouter du code qui :

1. Crée `MSALInteractiveTokenParameters` avec des étendues.
2. Appelle `acquireToken()` avec les paramètres créés.
3. Gère les erreurs. Pour plus d’informations, consultez le [guide de gestion des erreurs MSAL pour iOS et macOS](msal-error-handling-ios.md).
4. Gère le cas de réussite.

Ajoutez le code suivant à la classe `ViewController` .

```swift
func acquireTokenInteractively() {

    guard let applicationContext = self.applicationContext else { return }
    guard let webViewParameters = self.webViewParameters else { return }

    // #1
    let parameters = MSALInteractiveTokenParameters(scopes: kScopes, webviewParameters: webViewParameters)
    parameters.promptType = .selectAccount

    // #2
    applicationContext.acquireToken(with: parameters) { (result, error) in

        // #3
        if let error = error {

            self.updateLogging(text: "Could not acquire token: \(error)")
            return
        }

        guard let result = result else {

            self.updateLogging(text: "Could not acquire token: No result returned")
            return
        }

        // #4
        self.accessToken = result.accessToken
        self.updateLogging(text: "Access token is \(self.accessToken)")
        self.updateCurrentAccount(account: result.account)
        self.getContentWithToken()
    }
}
```

La propriété `promptType` de `MSALInteractiveTokenParameters` configure le comportement des invites d’authentification et de consentement. Les valeurs suivantes sont admises :

- `.promptIfNecessary` (par défaut) - Des invites sont envoyées à l’utilisateur uniquement en cas de besoin. L’expérience d’authentification unique est déterminée par la présence de cookies dans la vue web et le type de compte. Si plusieurs utilisateurs sont connectés, l’expérience de sélection de compte est présentée. *Il s’agit du comportement par défaut*.
- `.selectAccount` - Si aucun utilisateur n’est spécifié, la vue web d’authentification liste les comptes actuellement connectés parmi lesquels l’utilisateur peut faire son choix.
- `.login` - Oblige l’utilisateur à s’authentifier dans la vue web. Si vous spécifiez cette valeur, un seul compte peut être connecté à la fois.
- `.consent` - Oblige l’utilisateur à donner son consentement à l’ensemble actuel d’étendues pour la demande.

#### <a name="get-a-token-silently"></a>Obtenir un jeton en mode silencieux

Pour obtenir un jeton mis à jour en mode silencieux, ajoutez le code suivant à la classe `ViewController`. Il crée un objet `MSALSilentTokenParameters` et appelle `acquireTokenSilent()` :

```swift

    func acquireTokenSilently(_ account : MSALAccount!) {

        guard let applicationContext = self.applicationContext else { return }

        /**

         Acquire a token for an existing account silently

         - forScopes:           Permissions you want included in the access token received
         in the result in the completionBlock. Not all scopes are
         guaranteed to be included in the access token returned.
         - account:             An account object that we retrieved from the application object before that the
         authentication flow will be locked down to.
         - completionBlock:     The completion block that will be called when the authentication
         flow completes, or encounters an error.
         */

        let parameters = MSALSilentTokenParameters(scopes: kScopes, account: account)

        applicationContext.acquireTokenSilent(with: parameters) { (result, error) in

            if let error = error {

                let nsError = error as NSError

                // interactionRequired means we need to ask the user to sign-in. This usually happens
                // when the user's Refresh Token is expired or if the user has changed their password
                // among other possible reasons.

                if (nsError.domain == MSALErrorDomain) {

                    if (nsError.code == MSALError.interactionRequired.rawValue) {

                        DispatchQueue.main.async {
                            self.acquireTokenInteractively()
                        }
                        return
                    }
                }

                self.updateLogging(text: "Could not acquire token silently: \(error)")
                return
            }

            guard let result = result else {

                self.updateLogging(text: "Could not acquire token: No result returned")
                return
            }

            self.accessToken = result.accessToken
            self.updateLogging(text: "Refreshed Access token is \(self.accessToken)")
            self.updateSignOutButton(enabled: true)
            self.getContentWithToken()
        }
    }
```

### <a name="call-the-microsoft-graph-api"></a>Appeler l’API Microsoft Graph

Après avoir obtenu un jeton, votre application peut l’utiliser dans l’en-tête HTTP pour envoyer une requête autorisée à Microsoft Graph :

| clé d’en-tête    | value                 |
| ------------- | --------------------- |
| Autorisation | Porteur \<access-token> |

Ajoutez le code suivant à la classe `ViewController` :

```swift
    func getContentWithToken() {

        // Specify the Graph API endpoint
        let graphURI = getGraphEndpoint()
        let url = URL(string: graphURI)
        var request = URLRequest(url: url!)

        // Set the Authorization header for the request. We use Bearer tokens, so we specify Bearer + the token we got from the result
        request.setValue("Bearer \(self.accessToken)", forHTTPHeaderField: "Authorization")

        URLSession.shared.dataTask(with: request) { data, response, error in

            if let error = error {
                self.updateLogging(text: "Couldn't get graph result: \(error)")
                return
            }

            guard let result = try? JSONSerialization.jsonObject(with: data!, options: []) else {

                self.updateLogging(text: "Couldn't deserialize result JSON")
                return
            }

            self.updateLogging(text: "Result from Graph: \(result))")

            }.resume()
    }
```

Pour en savoir plus sur l’API Microsoft Graph, consultez [Microsoft Graph API](https://graph.microsoft.com).

### <a name="use-msal-for-sign-out"></a>Utiliser MSAL pour se déconnecter

Ajoutez ensuite la prise en charge de la déconnexion.

> [!Important]
> La déconnexion de MSAL entraîne la suppression de toutes les informations connues sur un utilisateur dans l’application ainsi que la suppression d’une session active sur son appareil quand c’est autorisé par la configuration de l’appareil. Vous pouvez aussi déconnecter l’utilisateur du navigateur.

Pour ajouter une fonctionnalité de déconnexion, ajoutez le code suivant dans la classe `ViewController`.

```swift
@objc func signOut(_ sender: AnyObject) {

        guard let applicationContext = self.applicationContext else { return }

        guard let account = self.currentAccount else { return }

        do {

            /**
             Removes all tokens from the cache for this application for the provided account

             - account:    The account to remove from the cache
             */

            let signoutParameters = MSALSignoutParameters(webviewParameters: self.webViewParameters!)
            signoutParameters.signoutFromBrowser = false // set this to true if you also want to signout from browser or webview

            applicationContext.signout(with: account, signoutParameters: signoutParameters, completionBlock: {(success, error) in

                if let error = error {
                    self.updateLogging(text: "Couldn't sign out account with error: \(error)")
                    return
                }

                self.updateLogging(text: "Sign out completed successfully")
                self.accessToken = ""
                self.updateCurrentAccount(account: nil)
            })

        }
    }
```

### <a name="enable-token-caching"></a>Activer la mise en cache des jetons

Par défaut, MSAL met en cache les jetons de votre application dans le trousseau iOS ou macOS.

Pour activer la mise en cache des jetons

1. Vérifiez que votre application est signée correctement.
1. Accédez à vos paramètres de projet Xcode > **onglet Fonctionnalités** > **Activer le partage de trousseau**.
1. Sélectionnez **+** et entrez l’un des **Groupes de trousseaux** suivants :
    - iOS : `com.microsoft.adalcache`
    - MacOS : `com.microsoft.identity.universalstorage`

### <a name="add-helper-methods"></a>Ajouter des méthodes d’assistance
Ajoutez les méthodes d’assistance suivantes à la classe `ViewController` pour compléter l’exemple.

### <a name="ios-ui"></a>Interface utilisateur iOS :

``` swift

    func updateLogging(text : String) {

        if Thread.isMainThread {
            self.loggingText.text = text
        } else {
            DispatchQueue.main.async {
                self.loggingText.text = text
            }
        }
    }

    func updateSignOutButton(enabled : Bool) {
        if Thread.isMainThread {
            self.signOutButton.isEnabled = enabled
        } else {
            DispatchQueue.main.async {
                self.signOutButton.isEnabled = enabled
            }
        }
    }

    func updateAccountLabel() {

        guard let currentAccount = self.currentAccount else {
            self.usernameLabel.text = "Signed out"
            return
        }

        self.usernameLabel.text = currentAccount.username
    }

    func updateCurrentAccount(account: MSALAccount?) {
        self.currentAccount = account
        self.updateAccountLabel()
        self.updateSignOutButton(enabled: account != nil)
    }
```

### <a name="macos-ui"></a>Interface utilisateur macOS :

```swift
    func updateLogging(text : String) {

        if Thread.isMainThread {
            self.loggingText.string = text
        } else {
            DispatchQueue.main.async {
                self.loggingText.string = text
            }
        }
    }

    func updateSignOutButton(enabled : Bool) {
        if Thread.isMainThread {
            self.signOutButton.isEnabled = enabled
        } else {
            DispatchQueue.main.async {
                self.signOutButton.isEnabled = enabled
            }
        }
    }

     func updateAccountLabel() {

         guard let currentAccount = self.currentAccount else {
            self.usernameLabel.stringValue = "Signed out"
            return
        }

        self.usernameLabel.stringValue = currentAccount.username ?? ""
        self.usernameLabel.sizeToFit()
     }

     func updateCurrentAccount(account: MSALAccount?) {
        self.currentAccount = account
        self.updateAccountLabel()
        self.updateSignOutButton(enabled: account != nil)
    }
```

### <a name="for-ios-only-get-additional-device-information"></a>Pour iOS uniquement, obtenir des informations supplémentaires sur l’appareil

Utilisez le code suivant pour lire la configuration actuelle de l’appareil, notamment si l’appareil est configuré comme étant partagé :

```swift
    @objc func getDeviceMode(_ sender: AnyObject) {

        if #available(iOS 13.0, *) {
            self.applicationContext?.getDeviceInformation(with: nil, completionBlock: { (deviceInformation, error) in

                guard let deviceInfo = deviceInformation else {
                    self.updateLogging(text: "Device info not returned. Error: \(String(describing: error))")
                    return
                }

                let isSharedDevice = deviceInfo.deviceMode == .shared
                let modeString = isSharedDevice ? "shared" : "private"
                self.updateLogging(text: "Received device info. Device is in the \(modeString) mode.")
            })
        } else {
            self.updateLogging(text: "Running on older iOS. GetDeviceInformation API is unavailable.")
        }
    }
```

### <a name="multi-account-applications"></a>Applications multicompte

Cette application est adaptée à un scénario de compte unique. MSAL prend également en charge les scénarios multicomptes. Toutefois, cela nécessite du travail supplémentaire de la part des applications. Vous devrez créer l’interface utilisateur pour aider les utilisateurs à sélectionner le compte qu’ils souhaitent utiliser pour chaque action qui nécessite des jetons. Sinon, votre application peut implémenter une approche heuristique pour sélectionner le compte à utiliser en interrogeant tous les comptes auprès de MSAL. Par exemple, consultez l’[API](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALPublicClientApplication.html#/c:objc(cs)MSALPublicClientApplication(im)accountsFromDeviceForParameters:completionBlock:) `accountsFromDeviceForParameters:completionBlock:`

## <a name="test-your-app"></a>Test de l'application

Générez et déployez l’application sur un appareil de test ou un simulateur. Vous devez pouvoir vous connecter et obtenir des jetons pour les comptes Azure AD ou les comptes personnels Microsoft.

La première fois qu’un utilisateur se connecte à votre application, Microsoft Identity l’invitera à accepter les autorisations demandées. Bien que la plupart des utilisateurs puissent donner leur accord, certains locataires Azure AD ont désactivé le consentement de l’utilisateur, ce qui oblige les administrateurs à donner leur consentement au nom de tous les utilisateurs. Pour permettre la prise en charge de ce scénario, inscrivez les étendues de votre application sur le portail Azure.

Une fois que vous êtes connecté, l’application affiche les données retournées par le point de terminaison `/me` Microsoft Graph.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur la création d’applications mobiles qui appellent des API web protégées dans notre série de scénarios en plusieurs parties.

> [!div class="nextstepaction"]
> [Scénario : Application mobile appelant des API web](scenario-mobile-overview.md)
