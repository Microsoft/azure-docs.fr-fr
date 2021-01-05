---
author: mikeparker104
ms.author: miparker
ms.date: 07/07/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: 9a8b76c0887cff498edd90aedf5cc6eb674fc7e5
ms.sourcegitcommit: 5cace04239f5efef4c1eed78144191a8b7d7fee8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/08/2020
ms.locfileid: "86156292"
---
### <a name="configure-the-runner-target-and-infoplist"></a>Configurer la cible Runner et Info.plist

1. Dans **Visual Studio Code**, faites un **Ctrl** + **clic** sur le dossier **ios**, puis choisissez **Ouvrir dans Xcode**.

1. Dans **Xcode**, cliquez sur **Runner** (projet **xcodeproj** situé en haut, pas le dossier), puis sélectionnez la cible **Runner** et **Signing & Capabilities**. Après avoir sélectionné la configuration de build **All**, choisissez votre compte de développeur pour **Team**. Vérifiez ensuite que l’option « Automatically manage signing » est activée et que votre certificat de signature et votre profil de provisionnement sont automatiquement sélectionnés.

    > [!NOTE]
    > Si vous ne voyez pas la nouvelle valeur de profil d’approvisionnement, essayez d’actualiser les profils de l’identité de signature. Pour cela, sélectionnez **Xcode** > **Preferences** > **Account**, puis sélectionnez le bouton **Download Manual Profiles** pour télécharger les profils.

1. Cliquez sur **+ Capability**, puis recherchez **Push Notifications**. **Double-cliquez** sur **Push Notifications** pour ajouter cette fonctionnalité.

1. Ouvrez **Info.plist** et définissez **Minimum system version** avec la valeur **13.0**.

    > [!NOTE]
    > Seuls les appareils qui exécutent **iOS 13.0 ou une version ultérieure** sont pris en charge dans ce tutoriel, mais vous pouvez l’étendre pour prendre en charge les appareils exécutant des versions antérieures.

1. Ouvrez **Runner.entitlements** et vérifiez que le paramètre **APS Environment** indique **development**.

### <a name="handle-push-notifications-for-ios"></a>Gérer les notifications Push pour iOS

1. Faites un **Ctrl** + **clic** sur le dossier **Runner** (dans le projet Runner), puis choisissez **New Group** avec **Services** comme nom.

1. Faites un **Ctrl** + **clic** sur le dossier **Services**, puis choisissez **New File**. Ensuite, choisissez **Swift File**, puis cliquez sur **Next**. Spécifiez **DeviceInstallationService** comme nom, puis cliquez sur **Create**.

1. Implémentez **DeviceInstallationService.swift** à l’aide du code suivant.

    ```swift
    import Foundation

    class DeviceInstallationService {

        enum DeviceRegistrationError: Error {
            case notificationSupport(message: String)
        }

        var token : Data? = nil

        let DEVICE_INSTALLATION_CHANNEL = "com.<your_organization>.pushdemo/deviceinstallation"
        let GET_DEVICE_ID = "getDeviceId"
        let GET_DEVICE_TOKEN = "getDeviceToken"
        let GET_DEVICE_PLATFORM = "getDevicePlatform"

        private let deviceInstallationChannel : FlutterMethodChannel

        var notificationsSupported : Bool {
            get {
                if #available(iOS 13.0, *) {
                    return true
                }
                else {
                    return false
                }
            }
        }

        init(withBinaryMessenger binaryMessenger : FlutterBinaryMessenger) {
            deviceInstallationChannel = FlutterMethodChannel(name: DEVICE_INSTALLATION_CHANNEL, binaryMessenger: binaryMessenger)
            deviceInstallationChannel.setMethodCallHandler(handleDeviceInstallationCall)
        }

        func getDeviceId() -> String {
            return UIDevice.current.identifierForVendor!.description
        }

        func getDeviceToken() throws -> String {
            if(!notificationsSupported) {
                let notificationSupportError = getNotificationsSupportError()
                throw DeviceRegistrationError.notificationSupport(message: notificationSupportError)
            }

            if (token == nil) {
                throw DeviceRegistrationError.notificationSupport(message: "Unable to resolve token for APNS.")
            }

            return token!.reduce("", {$0 + String(format: "%02X", $1)})
        }

        func getDevicePlatform() -> String {
            return "apns"
        }

        private func handleDeviceInstallationCall(call: FlutterMethodCall, result: @escaping FlutterResult) {
            switch call.method {
            case GET_DEVICE_ID:
                result(getDeviceId())
            case GET_DEVICE_TOKEN:
                getDeviceToken(result: result)
            case GET_DEVICE_PLATFORM:
                result(getDevicePlatform())
            default:
                result(FlutterMethodNotImplemented)
            }
        }

        private func getDeviceToken(result: @escaping FlutterResult) {
            do {
                let token = try getDeviceToken()
                result(token)
            }
            catch let error {
                result(FlutterError(code: "UNAVAILABLE", message: error.localizedDescription, details: nil))
            }
        }

        private func getNotificationsSupportError() -> String {

            if (!notificationsSupported) {
                return "This app only supports notifications on iOS 13.0 and above. You are running \(UIDevice.current.systemVersion)"
            }

            return "An error occurred preventing the use of push notifications."
        }
    }
    ```

    > [!NOTE]
    > Cette classe implémente l’équivalent propre à la plateforme du canal `com.<your_organization>.pushdemo/deviceinstallation`. Cela a été défini dans la partie Flutter de l’application dans **DeviceInstallationService.dart**. Dans ce cas, les appels sont effectués du code commun à l’hôte natif. Veillez à remplacer chaque instance de **<your_organization>** par votre propre organisation.
    >
    > Cette classe fournit un ID unique (à l’aide de la valeur [UIDevice.identifierForVendor](https://developer.apple.com/documentation/uikit/uidevice/1620059-identifierforvendor)) dans le cadre de la charge utile de l’inscription au hub de notification.

1. Ajoutez un autre élément de type **Swift File** au dossier **Services** appelé *NotificationRegistrationService*, puis ajoutez le code suivant.

    ```swift
    import Foundation

    class NotificationRegistrationService {

        let NOTIFICATION_REGISTRATION_CHANNEL = "com.<your_organization>.pushdemo/notificationregistration"
        let REFRESH_REGISTRATION = "refreshRegistration"

        private let notificationRegistrationChannel : FlutterMethodChannel

        init(withBinaryMessenger binaryMessenger : FlutterBinaryMessenger) {
           notificationRegistrationChannel = FlutterMethodChannel(name: NOTIFICATION_REGISTRATION_CHANNEL, binaryMessenger: binaryMessenger)
        }

        func refreshRegistration() {
            notificationRegistrationChannel.invokeMethod(REFRESH_REGISTRATION, arguments: nil)
        }
    }
    ```

    > [!NOTE]
    > Cette classe implémente l’équivalent propre à la plateforme pour le canal `com.<your_organization>.pushdemo/notificationregistration`. Cela a été défini dans la partie Flutter de l’application dans **NotificationRegistrationService.dart**. Dans ce cas, les appels sont effectués de l’hôte natif au code commun. Là encore, veillez à remplacer chaque instance de **<your_organization>** par votre propre organisation.

1. Ajoutez un autre élément de type **Swift File** au dossier **Services** appelé *NotificationActionService*, puis ajoutez le code suivant.

    ```swift
    import Foundation

    class NotificationActionService {

        let NOTIFICATION_ACTION_CHANNEL = "com.<your_organization>.pushdemo/notificationaction"
        let TRIGGER_ACTION = "triggerAction"
        let GET_LAUNCH_ACTION = "getLaunchAction"

        private let notificationActionChannel: FlutterMethodChannel

        var launchAction: String? = nil

        init(withBinaryMessenger binaryMessenger: FlutterBinaryMessenger) {
            notificationActionChannel = FlutterMethodChannel(name: NOTIFICATION_ACTION_CHANNEL, binaryMessenger: binaryMessenger)
            notificationActionChannel.setMethodCallHandler(handleNotificationActionCall)
        }

        func triggerAction(action: String) {
           notificationActionChannel.invokeMethod(TRIGGER_ACTION, arguments: action)
        }

        private func handleNotificationActionCall(call: FlutterMethodCall, result: @escaping FlutterResult) {
            switch call.method {
            case GET_LAUNCH_ACTION:
                result(launchAction)
            default:
                result(FlutterMethodNotImplemented)
            }
        }
    }
    ```

    > [!NOTE]
    > Cette classe implémente l’équivalent propre à la plateforme pour le canal `com.<your_organization>.pushdemo/notificationaction`. Cela a été défini dans la partie Flutter de l’application dans **NotificationActionService.dart**. Les appels peuvent être effectués dans les deux sens. Veillez à remplacer chaque instance de **<your_organization>** par votre propre organisation.

1. Dans **AppDelegate.swift**, ajoutez des variables pour stocker une référence aux services que vous avez créés précédemment.

    ```swift
    var deviceInstallationService : DeviceInstallationService?
    var notificationRegistrationService : NotificationRegistrationService?
    var notificationActionService : NotificationActionService?
    ```

1. Ajoutez une fonction appelée **processNotificationActions** pour traiter les données de notification. Déclenchez cette action de manière conditionnelle ou stockez-la en vue d’une utilisation ultérieure si l’action est traitée pendant le lancement de l’application.

    ```swift
    func processNotificationActions(userInfo: [AnyHashable : Any], launchAction: Bool = false) {
        if let action = userInfo["action"] as? String {
            if (launchAction) {
                notificationActionService?.launchAction = action
            }
            else {
                notificationActionService?.triggerAction(action: action)
            }
        }
    }
    ```

1. Substituez la fonction **didRegisterForRemoteNotificationsWithDeviceToken** en définissant la valeur du **jeton** pour **DeviceInstallationService**. Ensuite, appelez **refreshRegistration** sur **NotificationRegistrationService**.

    ```swift
    override func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
      deviceInstallationService?.token = deviceToken
      notificationRegistrationService?.refreshRegistration()
    }
    ```

1. Substituez la méthode **didReceiveRemoteNotification** en passant l’argument **userInfo** à la fonction **processNotificationActions**.

    ```swift
    override func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable : Any]) {
        processNotificationActions(userInfo: userInfo)
    }
    ```

1. Substituez la fonction **didFailToRegisterForRemoteNotificationsWithError** pour consigner l’erreur.

    ```swift
    override func application(_ application: UIApplication, didFailToRegisterForRemoteNotificationsWithError error: Error) {
        print(error);
    }
    ```

    > [!NOTE]
    > Cette valeur s’apparente à un espace réservé. Vous pouvez implémenter une journalisation et une gestion des erreurs appropriées pour les scénarios de production.

1. Dans **didFinishLaunchingWithOptions**, instanciez les variables **deviceInstallationService**, **notificationRegistrationService** et **notificationActionService**.

    ```swift
    let controller : FlutterViewController = window?.rootViewController as! FlutterViewController

    deviceInstallationService = DeviceInstallationService(withBinaryMessenger: controller.binaryMessenger)
    notificationRegistrationService = NotificationRegistrationService(withBinaryMessenger: controller.binaryMessenger)
    notificationActionService = NotificationActionService(withBinaryMessenger: controller.binaryMessenger)
    ```

1. Dans la même fonction, demandez une autorisation de manière conditionnelle et ajoutez l’inscription aux notifications distantes.

    ```swift
    if #available(iOS 13.0, *) {
      UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) {
          (granted, error) in

          if (granted)
          {
              DispatchQueue.main.async {
                  let pushSettings = UIUserNotificationSettings(types: [.alert, .sound, .badge], categories: nil)
                  application.registerUserNotificationSettings(pushSettings)
                  application.registerForRemoteNotifications()
              }
          }
      }
    }
    ```

1. Si **launchOptions** contient la clé **remoteNotification**, appelez **processNotificationActions** à la fin de la fonction **didFinishLaunchingWithOptions**. Passez l’objet **userInfo** résultant et utilisez *true* pour l’argument *launchAction*. Une valeur *true* indique que l’action est en cours de traitement pendant le lancement de l’application.

    ```swift
    if let userInfo = launchOptions?[.remoteNotification] as? [AnyHashable : Any] {
        processNotificationActions(userInfo: userInfo, launchAction: true)
    }
    ```
