---
author: mikeparker104
ms.author: miparker
ms.date: 07/07/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: 11bd05231aa3b5b336400d2434e63e57807a7e69
ms.sourcegitcommit: 5cace04239f5efef4c1eed78144191a8b7d7fee8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/08/2020
ms.locfileid: "86156295"
---
### <a name="add-the-google-services-json-file"></a>Ajout du fichier JSON Google Services

1. Faites un **Ctrl** + **clic** sur le dossier **android**, puis choisissez **Ouvrir dans Android Studio**. Passez ensuite en mode **Projet** (si ce n’est pas déjà fait).

1. Recherchez le fichier *google-services.json* que vous avez téléchargé précédemment lors de la configuration du projet **PushDemo** dans la [console Firebase](https://console.firebase.google.com). Ensuite, faites-le glisser dans le répertoire racine du module **app** (**android** > **android** > **app**).

### <a name="configure-build-settings-and-permissions"></a>Configurer les paramètres de build et les autorisations

1. Passez du mode **Projet** au mode **Android**.

1. Ouvrez **AndroidManifest.xml**, puis ajoutez les autorisations **INTERNET** et **READ_PHONE_STATE** après l’élément **application** avant la balise de fermeture **</manifest>** .

    ```xml
    <manifest>
        <application>...</application>
        <uses-permission android:name="android.permission.INTERNET" />
        <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    </manifest>
    ```

### <a name="add-the-firebase-sdks"></a>Ajouter les kits SDK Firebase

1. Dans **Android Studio**, ouvrez le fichier **build.gradle** au niveau du projet (**Gradle Scripts** > **build.gradle (Project: android)** ). Vérifiez quer le classpath « com.google.gms:google-services » se trouve dans le nœud ``buildscript`` > **dependencies**.

    ```json
    buildscript {

      repositories {
        // Check that you have the following line (if not, add it):
        google()  // Google's Maven repository
      }

      dependencies {
        // ...

        // Add the following line:
        classpath 'com.google.gms:google-services:4.3.3'  // Google Services plugin
      }
    }

    allprojects {
      // ...

      repositories {
        // Check that you have the following line (if not, add it):
        google()  // Google's Maven repository
        // ...
      }
    }
    ```

    > [!NOTE]
    > Veillez à référencer la dernière version, conformément aux instructions fournies dans la [console Firebase](https://console.firebase.google.com) au moment de la création du **projet Android**.

1. Dans le fichier **build.gradle** au niveau de l’application (**Gradle Scripts** > **build.gradle (Module: app)** ), appliquez le **plug-in Google Services Gradle**. Appliquez le plug-in juste au-dessus du nœud **android**.

    ```json
    // ...

    // Add the following line:
    apply plugin: 'com.google.gms.google-services'  // Google Services plugin

    android {
      // ...
    }
    ```

1. Dans le même fichier, dans le nœud **dependencies**, ajoutez la dépendance pour la bibliothèque Android **Cloud Messaging**.

    ```json
    dependencies {
        // ...
        implementation 'com.google.firebase:firebase-messaging:20.2.0'
    }
    ```

    > [!NOTE]
    > Veillez à référencer la dernière version, conformément à la [documentation sur le client Android Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/android/client).

1. Enregistrez les changements, puis cliquez sur le bouton **Sync Now** (dans l’invite de la barre d’outils) ou sur **Sync Project with Gradle Files**.

### <a name="handle-push-notifications-for-android"></a>Gérer les notifications Push pour Android

1. Dans **Android Studio**, faites un **Ctrl** + **clic** sur le dossier du package **com.<your_organization>.pushdemo** (**app** > **src** > **main** > **kotlin**), puis choisissez **Package** dans le menu **New** menu. Entrez **services** comme nom, puis appuyez sur **Entrée**.

1. Faites un **Ctrl** + **clic** dans le dossier **services**, puis choisissez **Kotlin File/Class** dans le menu **New**. Entrez **DeviceInstallationService** comme nom, puis appuyez sur **Entrée**.

1. Implémentez **DeviceInstallationService** à l’aide du code suivant.

    ```kotlin
    package com.<your_organization>.pushdemo.services

    import android.annotation.SuppressLint
    import android.content.Context
    import android.provider.Settings.Secure
    import com.google.android.gms.common.ConnectionResult
    import com.google.android.gms.common.GoogleApiAvailability
    import io.flutter.embedding.engine.FlutterEngine
    import io.flutter.plugin.common.MethodCall
    import io.flutter.plugin.common.MethodChannel

    @SuppressLint("HardwareIds")
    class DeviceInstallationService {

        companion object {
            const val DEVICE_INSTALLATION_CHANNEL = "com.<your_organization>.pushdemo/deviceinstallation"
            const val GET_DEVICE_ID = "getDeviceId"
            const val GET_DEVICE_TOKEN = "getDeviceToken"
            const val GET_DEVICE_PLATFORM = "getDevicePlatform"
        }

        private var context: Context
        private var deviceInstallationChannel : MethodChannel

        val playServicesAvailable
            get() = GoogleApiAvailability.getInstance().isGooglePlayServicesAvailable(context) == ConnectionResult.SUCCESS

        constructor(context: Context, flutterEngine: FlutterEngine) {
            this.context = context
            deviceInstallationChannel = MethodChannel(flutterEngine.dartExecutor.binaryMessenger, DEVICE_INSTALLATION_CHANNEL)
            deviceInstallationChannel.setMethodCallHandler { call, result -> handleDeviceInstallationCall(call, result) }
        }

        fun getDeviceId() : String
            = Secure.getString(context.applicationContext.contentResolver, Secure.ANDROID_ID)

        fun getDeviceToken() : String {
            if(!playServicesAvailable) {
                throw Exception(getPlayServicesError())
            }

            // TODO: Revisit once we have created the PushNotificationsFirebaseMessagingService
            val token = "Placeholder_Get_Value_From_FirebaseMessagingService_Implementation"

            if (token.isNullOrBlank()) {
                throw Exception("Unable to resolve token for FCM.")
            }

            return token
        }

        fun getDevicePlatform() : String = "fcm"

        private fun handleDeviceInstallationCall(call: MethodCall, result: MethodChannel.Result) {
            when (call.method) {
                GET_DEVICE_ID -> {
                    result.success(getDeviceId())
                }
                GET_DEVICE_TOKEN -> {
                    getDeviceToken(result)
                }
                GET_DEVICE_PLATFORM -> {
                    result.success(getDevicePlatform())
                }
                else -> {
                    result.notImplemented()
                }
            }
        }

        private fun getDeviceToken(result: MethodChannel.Result) {
            try {
                val token = getDeviceToken()
                result.success(token)
            }
            catch (e: Exception) {
                result.error("ERROR", e.message, e)
            }
        }

        private fun getPlayServicesError(): String {
            val resultCode = GoogleApiAvailability.getInstance().isGooglePlayServicesAvailable(context)

            if (resultCode != ConnectionResult.SUCCESS) {
                return if (GoogleApiAvailability.getInstance().isUserResolvableError(resultCode)){
                    GoogleApiAvailability.getInstance().getErrorString(resultCode)
                } else {
                    "This device is not supported"
                }
            }

            return "An error occurred preventing the use of push notifications"
        }
    }
    ```

    > [!NOTE]
    > Cette classe implémente l’équivalent propre à la plateforme du canal `com.<your_organization>.pushdemo/deviceinstallation`. Cela a été défini dans la partie Flutter de l’application dans **DeviceInstallationService.dart**. Dans ce cas, les appels sont effectués du code commun à l’hôte natif. Veillez à remplacer chaque instance de **<your_organization>** par votre propre organisation.
    >
    > Cette classe fournit un ID unique (à l’aide de [Secure.AndroidId](https://developer.android.com/reference/android/provider/Settings.Secure#ANDROID_ID)) dans le cadre de la charge utile de l’inscription au hub de notification.

1. Ajoutez un autre élément de type **Kotlin File/Class** au dossier **services** appelé *NotificationRegistrationService*, puis ajoutez le code suivant.

    ```kotlin
    package com.<your_organization>.pushdemo.services

    import io.flutter.embedding.engine.FlutterEngine
    import io.flutter.plugin.common.MethodChannel

    class NotificationRegistrationService {

        companion object {
            const val NOTIFICATION_REGISTRATION_CHANNEL = "com.<your_organization>.pushdemo/notificationregistration"
            const val REFRESH_REGISTRATION = "refreshRegistration"
        }

        private var notificationRegistrationChannel : MethodChannel

        constructor(flutterEngine: FlutterEngine) {
            notificationRegistrationChannel = MethodChannel(flutterEngine.dartExecutor.binaryMessenger, NotificationRegistrationService.NOTIFICATION_REGISTRATION_CHANNEL)
        }

        fun refreshRegistration() {
            notificationRegistrationChannel.invokeMethod(REFRESH_REGISTRATION, null)
        }
    }
    ```

    > [!NOTE]
    > Cette classe implémente l’équivalent propre à la plateforme pour le canal `com.<your_organization>.pushdemo/notificationregistration`. Cela a été défini dans la partie Flutter de l’application dans **NotificationRegistrationService.dart**. Dans ce cas, les appels sont effectués de l’hôte natif au code commun. Là encore, veillez à remplacer chaque instance de **<your_organization>** par votre propre organisation.

1. Ajoutez un autre élément de type **Kotlin File/Class** au dossier **services** appelé *NotificationActionService*, puis ajoutez le code suivant.

    ```kotlin
    package com.<your_organization>.pushdemo.services

    import io.flutter.embedding.engine.FlutterEngine
    import io.flutter.plugin.common.MethodCall
    import io.flutter.plugin.common.MethodChannel

    class NotificationActionService {
        companion object {
            const val NOTIFICATION_ACTION_CHANNEL = "com.<your_organization>.pushdemo/notificationaction"
            const val TRIGGER_ACTION = "triggerAction"
            const val GET_LAUNCH_ACTION = "getLaunchAction"
        }

        private var notificationActionChannel : MethodChannel
        var launchAction : String? = null

        constructor(flutterEngine: FlutterEngine) {
            notificationActionChannel = MethodChannel(flutterEngine.dartExecutor.binaryMessenger, NotificationActionService.NOTIFICATION_ACTION_CHANNEL)
            notificationActionChannel.setMethodCallHandler { call, result -> handleNotificationActionCall(call, result) }
        }

        fun triggerAction(action: String) {
            notificationActionChannel.invokeMethod(NotificationActionService.TRIGGER_ACTION, action)
        }

        private fun handleNotificationActionCall(call: MethodCall, result: MethodChannel.Result) {
            when (call.method) {
                NotificationActionService.GET_LAUNCH_ACTION -> {
                    result.success(launchAction)
                }
                else -> {
                    result.notImplemented()
                }
            }
        }
    }
    ```

    > [!NOTE]
    > Cette classe implémente l’équivalent propre à la plateforme pour le canal `com.<your_organization>.pushdemo/notificationaction`. Cela a été défini dans la partie Flutter de l’application dans **NotificationActionService.dart**. Les appels peuvent être effectués dans les deux sens. Veillez à remplacer chaque instance de **<your_organization>** par votre propre organisation.

1. Ajoutez un nouvel élément de type **Kotlin File/Class** au package **com.<your_organization>.pushdemo** appelé *PushNotificationsFirebaseMessagingService*, puis implémentez-le à l’aide du code suivant.

    ```kotlin
    package com.<your_organization>.pushdemo

    import android.os.Handler
    import android.os.Looper
    import com.google.firebase.messaging.FirebaseMessagingService
    import com.google.firebase.messaging.RemoteMessage
    import com.<your_organization>.pushdemo.services.NotificationActionService
    import com.<your_organization>.pushdemo.services.NotificationRegistrationService

    class PushNotificationsFirebaseMessagingService : FirebaseMessagingService() {

        companion object {
            var token : String? = null
            var notificationRegistrationService : NotificationRegistrationService? = null
            var notificationActionService : NotificationActionService? = null
        }

        override fun onNewToken(token: String) {
            PushNotificationsFirebaseMessagingService.token = token
            notificationRegistrationService?.refreshRegistration()
        }

        override fun onMessageReceived(message: RemoteMessage) {
            message.data.let {
                Handler(Looper.getMainLooper()).post {
                    notificationActionService?.triggerAction(it.getOrDefault("action", null))
                }
            }
        }
    }
    ```

    > [!NOTE]
    > Cette classe est chargée de gérer les notifications quand l’application s’exécute au premier plan. Elle appelle de manière conditionnelle **triggerAction** sur **NotificationActionService** si une action est incluse dans la charge utile de notification qui est reçue dans **onMessageReceived**. Elle appelle également **refreshRegistration** sur **NotificationRegistrationService** quand le jeton **Firebase** est regénéré en remplaçant la fonction **onNewToken**.
    >
    > Encore une fois, veillez à remplacer chaque instance de **<your_organization>** par votre propre organisation.

1. Dans **AndroidManifest.xml** (**app** > **src** > **main**), ajoutez **PushNotificationsFirebaseMessagingService** en bas de l’élément **application** avec le filtre d’intention `com.google.firebase.MESSAGING_EVENT`.

    ```xml
    <manifest>
        <application>
            <!-- EXISTING MANIFEST CONTENT -->
             <service
                android:name="com.<your_organization>.pushdemo.PushNotificationsFirebaseMessagingService"
                android:exported="false">
                <intent-filter>
                    <action android:name="com.google.firebase.MESSAGING_EVENT" />
                </intent-filter>
            </service>
        </application>
    </manifest>
    ```

1. De retour dans **DeviceInstallationService**, vérifiez que les importations suivantes sont présentes en haut du fichier.

    ```kotlin
    package com.<your_organization>.pushdemo
    import com.<your_organization>.pushdemo.services.PushNotificationsFirebaseMessagingService
    ```

    > [!NOTE]
    > Remplacez **<your_organization>** par votre propre organisation.

1. Changez le texte de l’espace réservé *Placeholder_Get_Value_From_FirebaseMessagingService_Implementation* pour récupérer la valeur du jeton de **PushNotificationFirebaseMessagingService**.

    ```kotlin
    fun getDeviceToken() : String {
        if(!playServicesAvailable) {
            throw Exception(getPlayServicesError())
        }

        // Get token from the PushNotificationsFirebaseMessagingService.token field.
        val token = PushNotificationsFirebaseMessagingService.token

        if (token.isNullOrBlank()) {
            throw Exception("Unable to resolve token for FCM.")
        }

        return token
    }
    ```

1. Dans **MainActivity**, vérifiez que les importations suivantes sont présentes en haut du fichier.

    ```kotlin
    package com.<your_organization>.pushdemo

    import android.content.Intent
    import android.os.Bundle
    import com.google.android.gms.tasks.OnCompleteListener
    import com.google.firebase.iid.FirebaseInstanceId
    import com.<your_organization>.pushdemo.services.DeviceInstallationService
    import com.<your_organization>.pushdemo.services.NotificationActionService
    import com.<your_organization>.pushdemo.services.NotificationRegistrationService
    import io.flutter.embedding.android.FlutterActivity
    ```

    > [!NOTE]
    > Remplacez **<your_organization>** par votre propre organisation.

1. Ajoutez une variable pour stocker une référence à **DeviceInstallationService**.

    ```kotlin
    private lateinit var deviceInstallationService: DeviceInstallationService
    ```

1. Ajoutez une fonction appelée **processNotificationActions** pour vérifier si une intention (**Intent**) a une valeur supplémentaire nommée **action**. Déclenchez cette action de manière conditionnelle ou stockez-la en vue d’une utilisation ultérieure si l’action est traitée pendant le lancement de l’application.

    ```kotlin
     private fun processNotificationActions(intent: Intent, launchAction: Boolean = false) {
        if (intent.hasExtra("action")) {
            var action = intent.getStringExtra("action");

            if (action.isNotEmpty()) {
                if (launchAction) {
                    PushNotificationsFirebaseMessagingService.notificationActionService?.launchAction = action
                }
                else {
                    PushNotificationsFirebaseMessagingService.notificationActionService?.triggerAction(action)
                }
            }
        }
    }
    ```

1. Remplacez la fonction **onNewIntent** pour appeler **processNotificationActions**.

    ```kotlin
    override fun onNewIntent(intent: Intent) {
        super.onNewIntent(intent)
        processNotificationActions(intent)
    }
    ```

    > [!NOTE]
    > Dans la mesure où **LaunchMode** pour **MainActivity** a la valeur **SingleTop**, une intention (**Intent**) sera envoyée à l’instance **Activity** par le biais de la fonction **onNewIntent**, plutôt que de la fonction **onCreate**. Vous devez donc gérer une intention (**Intent**) entrante dans les fonctions **onCreate** et **onNewIntent**.

1. Substituez la fonction **onCreate** en définissant **deviceInstallationService** avec une nouvelle instance de **DeviceInstallationService**.

    ```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        flutterEngine?.let {
            deviceInstallationService = DeviceInstallationService(context, it)
        }
    }
    ```

1. Définissez les propriétés **notificationActionService** et **notificationRegistrationService** avec **PushNotificationFirebaseMessagingServices**.

    ```kotlin
    flutterEngine?.let {
      deviceInstallationService = DeviceInstallationService(context, it)
      PushNotificationsFirebaseMessagingService.notificationActionService = NotificationActionService(it)
      PushNotificationsFirebaseMessagingService.notificationRegistrationService = NotificationRegistrationService(it)
    }
    ```

1. Dans la même fonction, appelez de manière conditionnelle **FirebaseInstanceId.getInstance().instanceId**. Implémentez **OnCompleteListener** pour définir la valeur résultante du **jeton** avec **PushNotificationFirebaseMessagingService** avant d’appeler **refreshRegistration**.

    ```kotlin
    if(deviceInstallationService?.playServicesAvailable) {
        FirebaseInstanceId.getInstance().instanceId
            .addOnCompleteListener(OnCompleteListener { task ->
                if (!task.isSuccessful)
                    return@OnCompleteListener

                PushNotificationsFirebaseMessagingService.token = task.result?.token
                PushNotificationsFirebaseMessagingService.notificationRegistrationService?.refreshRegistration()
            })
    }
    ```

1. Toujours dans **onCreate**, appelez **processNotificationActions** à la fin de la fonction. Utilisez *true* comme argument *launchAction* pour indiquer que cette action est traitée pendant le lancement de l’application.

    ```kotlin
    processNotificationActions(this.intent, true)
    ```

> [!NOTE]
> Pour continuer à recevoir des notifications Push, vous devez réinscrire l’application chaque fois que vous l’exécutez et l’arrêtez à partir d’une session de débogage.
