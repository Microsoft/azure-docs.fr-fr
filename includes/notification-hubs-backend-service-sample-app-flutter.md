---
author: mikeparker104
ms.author: miparker
ms.date: 07/07/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: 2f8cd5e554980225850780cc474106c3dc69288a
ms.sourcegitcommit: 5cace04239f5efef4c1eed78144191a8b7d7fee8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/08/2020
ms.locfileid: "86156296"
---
### <a name="create-the-flutter-solution"></a>Créer la solution Flutter

1. Ouvrez une nouvelle instance de **Visual Studio Code**.

1. Ouvrez la **palette de commandes** (**Maj** + **Cmd** + **P**).

1. Sélectionnez la commande **Flutter : Nouveau projet**, puis appuyez sur **Entrée**.

1. Entrez *push_demo* comme **Nom de projet**, puis sélectionnez un **Emplacement de projet**.

1. Quand vous y êtes invité, choisissez **Obtenir des packages**.

1. Faites un **Ctrl** + **clic** sur le dossier **kotlin** (sous **app** > **src** > **main**), puis choisissez **Révéler dans le Finder**. Ensuite, renommez les dossiers enfants (sous le dossier **kotlin**) en ```com```, ```<your_organization>``` et ```pushdemo``` respectivement.

    > [!NOTE]
    > Quand vous utilisez le modèle **Visual Studio Code**, ces dossiers sont par défaut **com**, **example**, **<project_name>** . À titre indicatif, si **mobcat** est utilisé comme **organisation**, la structure des dossiers doit apparaître comme suit :
    >
    > - kotlin
    >     - com
    >         - mobcat
    >             - pushdemo

1. De retour dans **Visual Studio Code**, remplacez la valeur **applicationId** dans **android** > **app** > **build.gradle** par `com.<your_organization>.pushdemo`.

    > [!NOTE]
    > Remplacez l’espace réservé *<your_organization>* par le nom de votre organisation. Par exemple, avec **mobcat** comme organisation, **Nom du package** a la valeur *com.mobcat.pushdemo*.

1. Mettez à jour l’attribut **package** dans les fichiers **AndroidManifest.xml**, sous **src** > **debug**, **src** > **main** et **src** > **profile** respectivement. Vérifiez que les valeurs correspondent à l’**applicationId** utilisé à l’étape précédente.

    ```xml
    <manifest
        xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.<your_organization>.pushdemo>">
        ...
    </manifest>
    ```

1. Remplacez l’attribut ```android:label``` dans le fichier **AndroidManifest.xml** sous **src** > **main** par *PushDemo*. Ensuite, ajoutez l’attribut ```android:allowBackup``` directement sous ```android:label``` et définissez-le avec la valeur **false**.

    ```xml
    <application
        android:name="io.flutter.app.FlutterApplication"
        android:label="PushDemo"
        android:allowBackup="false"
        android:icon="@mipmap/ic_launcher">
        ...
    </application>
    ```

1. Ouvrez le fichier **build.gradle** de niveau application (**android** > **app** > **build.gradle**), puis mettez à jour *compileSdkVersion* (dans la section **android**) pour utiliser l’API **29**. Ensuite, remplacez les valeurs *minSdkVersion* et *targetSdkVersion* (dans la section **defaultConfig**), par **26** et **29** respectivement.

    > [!NOTE]
    > Seuls les appareils qui exécutent les **niveaux d’API 26 et supérieurs** sont pris en charge dans le cadre de ce tutoriel, mais vous pouvez l’étendre pour prendre en charge les appareils exécutant des versions antérieures.

1. Faites un **Ctrl** + **clic** sur le dossier **ios**, puis choisissez **Ouvrir dans Xcode**.

1. Dans **Xcode**, cliquez sur **Runner** (projet **xcodeproj** en haut, pas le dossier). Sélectionnez ensuite la cible **Runner**, puis l’onglet **General**. Avec la configuration de build **All** sélectionnée, remplacez **Bundle Identifier** par `com.<your_organization>.PushDemo`.

    > [!NOTE]
    > Remplacez l’espace réservé *<your_organization>* par le nom de votre organisation. Par exemple, avec **mobcat** comme organisation, **Bundle Identifier** a la valeur *com.mobcat.PushDemo*.

1. Cliquez sur **Info.plist**, puis remplacez la valeur de **Bundle name** par **PushDemo**.

1. Fermez **Xcode** et revenez à **Visual Studio Code**.

1. De retour dans **Visual Studio Code**, ouvrez **pubspec.yaml**, ajoutez les **Packages Dart** [http](https://pub.dev/packages/http) et [flutter_secure_storage](https://pub.dev/packages/flutter_secure_storage) comme dépendances. Ensuite, enregistrez le fichier et cliquez sur **Obtenir des packages** quand vous y êtes invité.

    ```yaml
    dependencies:
      flutter:
        sdk: flutter

      http: ^0.12.1
      flutter_secure_storage: ^3.3.3
    ```

1. Dans **Terminal**, passez au dossier **ios** (pour votre projet Flutter). Ensuite, exécutez la commande [**pod install**](https://guides.cocoapods.org/using/getting-started.html#installation) pour installer de nouveaux pods (requis par le package [flutter_secure_storage](https://pub.dev/packages/flutter_secure_storage)).

1. Faites un **Ctrl** + **clic** sur le dossier **lib**, puis choisissez **Nouveau fichier** dans le menu avec *main_page.dart* comme nom de fichier. Ajoutez ensuite le code suivant.

    ```kotlin
    import 'package:flutter/material.dart';

    class MainPage extends StatefulWidget {
      @override
      _MainPageState createState() => _MainPageState();
    }

    class _MainPageState extends State<MainPage> {
      @override
      Widget build(BuildContext context) {
        return Scaffold(
          body: SafeArea(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: <Widget>[],
            )
          )
        );
      }
    }
    ```

1. Dans **main.dart**, remplacez le code basé sur un modèle par ce qui suit.

    ```dart
    import 'package:flutter/material.dart';
    import 'package:push_demo/main_page.dart';

    final navigatorKey = GlobalKey<NavigatorState>();

    void main() => runApp(MaterialApp(home: MainPage(), navigatorKey: navigatorKey));
    ```

1. Dans **Terminal**, générez et exécutez l’application sur chaque plateforme cible pour vérifier que l’application basée sur un modèle s’exécute sur vos appareils. Vérifiez que les appareils pris en charge sont connectés.

    ```bash
    flutter run
    ```

### <a name="implement-the-cross-platform-components"></a>Implémenter les composants multiplateforme

1. Faites un **Ctrl** + **clic** sur le dossier **lib**, puis choisissez **Nouveau dossier** dans le menu avec *models* comme **Nom de dossier**.

1. Faites un **Ctrl** + **clic** sur le dossier **models**, puis choisissez **Nouveau fichier** dans le menu avec *device_installation.dart* comme nom de fichier. Ajoutez ensuite le code suivant.

    ```dart
    class DeviceInstallation {
        final String deviceId;
        final String platform;
        final String token;
        final List<String> tags;

        DeviceInstallation(this.deviceId, this.platform, this.token, this.tags);

        DeviceInstallation.fromJson(Map<String, dynamic> json)
          : deviceId = json['installationId'],
            platform = json['platform'],
            token = json['pushChannel'],
            tags = json['tags'];

        Map<String, dynamic> toJson() =>
        {
          'installationId': deviceId,
          'platform': platform,
          'pushChannel': token,
          'tags': tags,
        };
    }
    ```

1. Ajoutez un nouveau fichier au dossier **models** appelé *push_demo_action.dart* pour définir l’énumération des actions prises en charge dans cet exemple.

    ```dart
    enum PushDemoAction {
      actionA,
      actionB,
    }
    ```

1. Ajoutez un nouveau dossier au projet appelé **services**, puis ajoutez un nouveau fichier appelé *device_installation_service.dart* avec l’implémentation suivante.

    ```dart
    import 'package:flutter/services.dart';

    class DeviceInstallationService {
      static const deviceInstallation = const MethodChannel('com.<your_organization>.pushdemo/deviceinstallation');
      static const String getDeviceIdChannelMethod = "getDeviceId";
      static const String getDeviceTokenChannelMethod = "getDeviceToken";
      static const String getDevicePlatformChannelMethod = "getDevicePlatform";

      Future<String> getDeviceId() {
        return deviceInstallation.invokeMethod(getDeviceIdChannelMethod);
      }

      Future<String> getDeviceToken() {
        return deviceInstallation.invokeMethod(getDeviceTokenChannelMethod);
      }

      Future<String> getDevicePlatform() {
        return deviceInstallation.invokeMethod(getDevicePlatformChannelMethod);
      }
    }
    ```

    > [!NOTE]
    > Remplacez l’espace réservé *<your_organization>* par le nom de votre organisation. Par exemple, avec **mobcat** comme organisation, [MethodChannel](https://api.flutter.dev/flutter/services/MethodChannel-class.html) a pour nom *com.mobcat.pushdemo/deviceinstallation*.
    >
    > Cette classe encapsule l’utilisation de la plateforme native sous-jacente pour acquérir les détails de l’installation de l’appareil requis. [MethodChannel](https://api.flutter.dev/flutter/services/MethodChannel-class.html) facilite la communication asynchrone bidirectionnelle avec les plateformes natives sous-jacentes. L’équivalent spécifique à la plateforme de ce canal sera créé dans les étapes ultérieures.

1. Ajoutez un autre fichier à ce dossier appelé *notification_action_service.dart* avec l’implémentation suivante.

    ```dart
    import 'package:flutter/services.dart';
    import 'dart:async';
    import 'package:push_demo/models/push_demo_action.dart';

    class NotificationActionService {
      static const notificationAction =
          const MethodChannel('com.<your_organization>.pushdemo/notificationaction');
      static const String triggerActionChannelMethod = "triggerAction";
      static const String getLaunchActionChannelMethod = "getLaunchAction";

      final actionMappings = {
        'action_a' : PushDemoAction.actionA,
        'action_b' : PushDemoAction.actionB
      };

      final actionTriggeredController = StreamController.broadcast();

      NotificationActionService() {
        notificationAction
            .setMethodCallHandler(handleNotificationActionCall);
      }

      Stream get actionTriggered => actionTriggeredController.stream;

      Future<void> triggerAction({action: String}) async {

        if (!actionMappings.containsKey(action)) {
          return;
        }

        actionTriggeredController.add(actionMappings[action]);
      }

      Future<void> checkLaunchAction() async {
        final launchAction = await notificationAction.invokeMethod(getLaunchActionChannelMethod) as String;

        if (launchAction != null) {
          triggerAction(action: launchAction);
        }
      }

      Future<void> handleNotificationActionCall(MethodCall call) async {
        switch (call.method) {
          case triggerActionChannelMethod:
            return triggerAction(action: call.arguments as String);
          default:
            throw MissingPluginException();
            break;
        }
      }
    }
    ```

    > [!NOTE]
    > Ce mécanisme simple permet de centraliser la gestion des actions de notification pour qu’elles puissent être gérées sur plusieurs plateformes à l’aide d’une énumération fortement typée. Le service permet à la plateforme native sous-jacente de déclencher une action si une telle action est spécifiée dans la charge utile de notification. Il permet également au code commun de vérifier de manière rétrospective si une action a été spécifiée durant le lancement de l’application quand Flutter est prêt à la traiter. C’est notamment le cas si vous lancez l’application en appuyant sur une notification dans le centre de notifications.

1. Ajoutez un nouveau fichier au dossier **services** appelé *notification_registration_service.dart* avec l’implémentation suivante.

    ```dart
    import 'dart:convert';
    import 'package:flutter/services.dart';
    import 'package:http/http.dart' as http;
    import 'package:push_demo/services/device_installation_service.dart';
    import 'package:push_demo/models/device_installation.dart';
    import 'package:flutter_secure_storage/flutter_secure_storage.dart';

    class NotificationRegistrationService {
      static const notificationRegistration =
          const MethodChannel('com.<your_organization>.pushdemo/notificationregistration');

      static const String refreshRegistrationChannelMethod = "refreshRegistration";
      static const String installationsEndpoint = "api/notifications/installations";
      static const String cachedDeviceTokenKey = "cached_device_token";
      static const String cachedTagsKey = "cached_tags";

      final deviceInstallationService = DeviceInstallationService();
      final secureStorage = FlutterSecureStorage();

      String baseApiUrl;
      String apikey;

      NotificationRegistrationService(this.baseApiUrl, this.apikey) {
        notificationRegistration
            .setMethodCallHandler(handleNotificationRegistrationCall);
      }

      String get installationsUrl => "$baseApiUrl$installationsEndpoint";

      Future<void> deregisterDevice() async {
        final cachedToken = await secureStorage.read(key: cachedDeviceTokenKey);
        final serializedTags = await secureStorage.read(key: cachedTagsKey);

        if (cachedToken == null || serializedTags == null) {
          return;
        }

        var deviceId = await deviceInstallationService.getDeviceId();

        if (deviceId.isEmpty) {
          throw "Unable to resolve an ID for the device.";
        }

        var response = await http
            .delete("$installationsUrl/$deviceId", headers: {"apikey": apikey});

        if (response.statusCode != 200) {
          throw "Deregister request failed: ${response.reasonPhrase}";
        }

        await secureStorage.delete(key: cachedDeviceTokenKey);
        await secureStorage.delete(key: cachedTagsKey);
      }

      Future<void> registerDevice(List<String> tags) async {
        try {
          final deviceId = await deviceInstallationService.getDeviceId();
          final platform = await deviceInstallationService.getDevicePlatform();
          final token = await deviceInstallationService.getDeviceToken();

          final deviceInstallation =
              DeviceInstallation(deviceId, platform, token, tags);

          final response = await http.put(installationsUrl,
              body: jsonEncode(deviceInstallation),
              headers: {"apikey": apikey, "Content-Type": "application/json"});

          if (response.statusCode != 200) {
            throw "Register request failed: ${response.reasonPhrase}";
          }

          final serializedTags = jsonEncode(tags);

          await secureStorage.write(key: cachedDeviceTokenKey, value: token);
          await secureStorage.write(key: cachedTagsKey, value: serializedTags);
        } on PlatformException catch (e) {
          throw e.message;
        } catch (e) {
          throw "Unable to register device: $e";
        }
      }

      Future<void> refreshRegistration() async {
        final currentToken = await deviceInstallationService.getDeviceToken();
        final cachedToken = await secureStorage.read(key: cachedDeviceTokenKey);
        final serializedTags = await secureStorage.read(key: cachedTagsKey);

        if (currentToken == null ||
            cachedToken == null ||
            serializedTags == null ||
            currentToken == cachedToken) {
          return;
        }

        final tags = jsonDecode(serializedTags);

        return registerDevice(tags);
      }

      Future<void> handleNotificationRegistrationCall(MethodCall call) async {
        switch (call.method) {
          case refreshRegistrationChannelMethod:
            return refreshRegistration();
          default:
            throw MissingPluginException();
            break;
        }
      }
    }
    ```

    > [!NOTE]
    > Cette classe encapsule l’utilisation de **DeviceInstallationService** et des demandes au service back-end pour effectuer les actions d’inscription, de désinscription et d’actualisation de l’inscription requises. L’argument **apiKey** n’est obligatoire que si vous avez choisi d’effectuer la section [Authentifier les clients à l’aide d’une clé API](#authenticate-clients-using-an-api-key-optional).

1. Ajoutez un nouveau fichier au dossier **lib** appelé *config.dart* avec l’implémentation suivante.

    ```dart
    class Config {
      static String apiKey = "API_KEY";
      static String backendServiceEndpoint = "BACKEND_SERVICE_ENDPOINT";
    }
    ```

    > [!NOTE]
    > Il s’agit d’un moyen simple de définir des secrets d’application. Remplacez les valeurs d’espace réservé par vos propres valeurs. Vous devez en avoir pris note quand vous avez créé le service back-end. L’**URL de l’application API** doit être `https://<api_app_name>.azurewebsites.net/`. Le membre **apiKey** n’est obligatoire que si vous avez choisi d’effectuer la section [Authentifier les clients à l’aide d’une clé API](#authenticate-clients-using-an-api-key-optional).
    >
    > Veillez à ajouter cela à votre fichier gitignore pour éviter de valider ces secrets dans le contrôle de code source.

### <a name="implement-the-cross-platform-ui"></a>Implémenter l’interface utilisateur multiplateforme

1. Dans **main_page.dart**, remplacez la fonction **build** par ce qui suit.

    ```dart
    @override
    Widget build(BuildContext context) {
    return Scaffold(
        body: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 20.0, vertical: 40.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.end,
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: <Widget>[
              FlatButton(
                child: Text("Register"),
                onPressed: registerButtonClicked,
              ),
              FlatButton(
                child: Text("Deregister"),
                onPressed: deregisterButtonClicked,
              ),
            ],
          ),
        ),
      );
    }
    ```

1. Ajoutez les importations requises en haut du fichier **main_page.dart**.

    ```dart
    import 'package:push_demo/services/notification_registration_service.dart';
    import 'config.dart';
    ```

1. Ajoutez un champ à la classe **_MainPageState** pour stocker une référence à **NotificationRegistrationService**.

    ```dart
    final notificationRegistrationService = NotificationRegistrationService(Config.backendServiceEndpoint, Config.apiKey);
    ```

1. Au niveau de la classe **_MainPageState**, implémentez les gestionnaires d’événements pour les événements **onPressed** des boutons **Register** et **Deregister**. Appelez les méthodes **Register**/**Deregister** correspondantes, puis affichez une alerte pour indiquer le résultat.

    ```dart
    void registerButtonClicked() async {
        try {
          await notificationRegistrationService.registerDevice(List<String>());
          await showAlert(message: "Device registered");
        }
        catch (e) {
          await showAlert(message: e);
        }
      }

      void deregisterButtonClicked() async {
        try {
          await notificationRegistrationService.deregisterDevice();
          await showAlert(message: "Device deregistered");
        }
        catch (e) {
          await showAlert(message: e);
        }
      }

      Future<void> showAlert({ message: String }) async {
        return showDialog<void>(
          context: context,
          barrierDismissible: false,
          builder: (BuildContext context) {
            return AlertDialog(
              title: Text('PushDemo'),
              content: SingleChildScrollView(
                child: ListBody(
                  children: <Widget>[
                    Text(message),
                  ],
                ),
              ),
              actions: <Widget>[
                FlatButton(
                  child: Text('OK'),
                  onPressed: () {
                    Navigator.of(context).pop();
                  },
                ),
              ],
            );
          },
        );
      }
    ```

1. Maintenant, dans **main.dart**, vérifiez que les importations suivantes sont présentes en haut du fichier.

    ```dart
    import 'package:flutter/material.dart';
    import 'package:push_demo/models/push_demo_action.dart';
    import 'package:push_demo/services/notification_action_service.dart';
    import 'package:push_demo/main_page.dart';
    ```

1. Déclarez une variable pour stocker la référence à une instance de **NotificationActionService**, puis initialisez-la.

    ```dart
    final notificationActionService = NotificationActionService();
    ```

1. Ajoutez des fonctions pour gérer l’affichage d’une alerte quand une action est déclenchée.

    ```dart
    void notificationActionTriggered(PushDemoAction action) {
      showActionAlert(message: "${action.toString().split(".")[1]} action received");
    }

    Future<void> showActionAlert({ message: String }) async {
      return showDialog<void>(
        context: navigatorKey.currentState.overlay.context,
        barrierDismissible: false,
        builder: (BuildContext context) {
          return AlertDialog(
            title: Text('PushDemo'),
            content: SingleChildScrollView(
              child: ListBody(
                children: <Widget>[
                  Text(message),
                ],
              ),
            ),
            actions: <Widget>[
              FlatButton(
                child: Text('OK'),
                onPressed: () {
                  Navigator.of(context).pop();
                },
              ),
            ],
          );
        },
      );
    }
    ```

1. Mettez à jour la fonction **main** pour observer le flux **NotificationActionService** **actionTriggered** et recherchez les actions capturées pendant le lancement de l’application.

    ```dart
    void main() async {
      runApp(MaterialApp(home: MainPage(), navigatorKey: navigatorKey,));
      notificationActionService.actionTriggered.listen((event) { notificationActionTriggered(event as PushDemoAction); });
      await notificationActionService.checkLaunchAction();
    }
    ```

    > [!NOTE]
    > Il s’agit simplement d’illustrer la réception et la propagation des actions de notification Push. En règle générale, celles-ci sont gérées en mode silencieux, par exemple en accédant à une vue spécifique ou en actualisant certaines données plutôt qu’en affichant une alerte en l’occurrence.
