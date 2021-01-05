---
title: Envoyer des notifications Push vers Android avec Azure Notification Hubs et Google Cloud Messaging | Microsoft Docs
description: Dans ce didacticiel, vous découvrirez comment utiliser Azure Notification Hubs et Google Firebase Cloud Messaging pour envoyer des notifications Push à des appareils Android.
services: notification-hubs
documentationcenter: android
keywords: notifications push,notification push,notification push android
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: tutorial
ms.custom: mvc, devx-track-java
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: 8c97710202a448c613ab685932cb335bbaed4953
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87832652"
---
# <a name="tutorial-send-push-notifications-to-android-devices-by-using-azure-notification-hubs-and-google-cloud-messaging-deprecated"></a>Tutoriel : Envoyer des notifications Push aux appareils Android via Azure Notification Hubs et Google Cloud Messaging (déconseillé)

[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

> [!WARNING]
> À compter du 10 avril 2018, Google a déconseillé Google Cloud Messaging (GCM). Le serveur GCM et les API clientes sont déconseillés et seront supprimés dès le 29 mai 2019. Pour plus d’informations, consultez [GCM et FCM dans la foire aux questions](https://developers.google.com/cloud-messaging/faq).

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel montre comment utiliser Azure Notification Hubs pour envoyer des notifications Push vers une application Android.
Vous allez créer une application Android vide qui reçoit des notifications Push à l’aide de Google Cloud Messaging (GCM).

> [!IMPORTANT]
> Google Cloud Messaging (GCM) est obsolète et sera supprimé [prochainement](https://developers.google.com/cloud-messaging/faq).

> [!IMPORTANT]
> Cette rubrique explique les notifications Push avec Google Cloud Messaging (GCM). Si vous utilisez encore Firebase Cloud Messaging (FCM) de Google, consultez [Sending push notifications to Android with Azure Notification Hubs and FCM](notification-hubs-android-push-notification-google-fcm-get-started.md)(Envoi de notifications Push vers Android avec Azure Notification Hubs et FCM).

Le code complet de ce didacticiel peut être téléchargé depuis GitHub [ici](https://github.com/Azure/azure-notificationhubs-android/tree/master/FCMTutorialApp).

Dans ce tutoriel, vous allez effectuer les actions suivantes :

> [!div class="checklist"]
> * Création d’un projet prenant en charge Google Cloud Messaging
> * Création d’un hub de notifications
> * Connexion de votre application au hub de notification
> * Test de l'application

## <a name="prerequisites"></a>Prérequis

* **Abonnement Azure**. Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.
* [Android Studio](https://go.microsoft.com/fwlink/?LinkId=389797).

## <a name="creating-a-project-that-supports-google-cloud-messaging"></a>Création d’un projet prenant en charge Google Cloud Messaging

[!INCLUDE [mobile-services-enable-Google-cloud-messaging](../../includes/mobile-services-enable-google-cloud-messaging.md)]

## <a name="create-a-notification-hub"></a>Création d’un hub de notifications

[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-gcm-setting-for-the-notification-hub"></a>Configurer un paramètre GCM pour le hub de notifications

1. Sélectionnez **Google (GCM)** dans **PARAMÈTRES DE NOTIFICATION**.
2. Entrez la valeur de **Clé API** que vous avez obtenue à partir de la console Google Cloud.
3. Sélectionnez **Enregistrer** dans la barre d’outils.

    ![Azure Notification Hubs - Google (GCM)](./media/notification-hubs-android-get-started/notification-hubs-gcm-api.png)

Votre hub de notification est à présent configuré pour GCM, et vous disposez des chaînes de connexion vous permettant d’inscrire votre application pour la réception et l’envoi de notifications Push.

## <a name="connect-your-app-to-the-notification-hub"></a><a id="connecting-app"></a>Connexion de votre application au hub de notification

### <a name="create-a-new-android-project"></a>Créer un projet Android

1. Dans Android Studio, démarrez un nouveau projet Android Studio.

   ![Android Studio - nouveau projet][13]
2. Choisissez le format **Phone and Tablet** (Téléphone et tablette) et le **Minimum SDK** (SDK minimal) que vous voulez prendre en charge. Cliquez ensuite sur **Suivant**.

   ![Android Studio - workflow de création de projet][14]
3. Choisissez **Empty Activity** (Activité vide) comme activité principale, cliquez sur **Next** (Suivant), puis sur **Finish** (Terminer).

### <a name="add-google-play-services-to-the-project"></a>Ajout de services Google Play au projet

[!INCLUDE [Add Play Services](../../includes/notification-hubs-android-studio-add-google-play-services.md)]

### <a name="adding-azure-notification-hubs-libraries"></a>Ajout de bibliothèques de concentrateurs de notification Azure

1. Dans le fichier `Build.Gradle` de **l’application**, dans la section **dépendances**, ajoutez les lignes suivantes.

    ```gradle
    implementation 'com.microsoft.azure:notification-hubs-android-sdk:0.6@aar'
    implementation 'com.microsoft.azure:azure-notifications-handler:1.0.1@aar'
    ```

2. Ajoutez le référentiel suivant après la section **dépendances** .

    ```gradle
    repositories {
        maven {
            url "https://dl.bintray.com/microsoftazuremobile/SDK"
        }
    }
    ```

### <a name="updating-the-projects-androidmanifestxml"></a>Mise à jour du fichier AndroidManifest.xml du projet

1. Pour prendre en charge GCM, implémentez un service d’écoute d’ID d’instance dans le code utilisé afin [d’obtenir des jetons d’inscription](https://developers.google.com/cloud-messaging/) à l’aide de [l’API d’ID d’instance Google](https://developers.google.com/instance-id/). Dans ce didacticiel, le nom de la classe est `MyInstanceIDService`.

    Ajoutez la définition de service suivante au fichier AndroidManifest.xml, dans la balise `<application>` . Remplacez l’espace réservé `<your package>` par le nom de votre package actuel, qui apparaît en haut du fichier `AndroidManifest.xml`.
  
    ```xml
    <service android:name="<your package>.MyInstanceIDService" android:exported="false">
        <intent-filter>
            <action android:name="com.google.android.gms.iid.InstanceID"/>
        </intent-filter>
    </service>
    ```

2. Une fois que l’application a reçu le jeton d’inscription GCM de la part de l’API d’ID d’instance, elle l’utilise pour [s’inscrire auprès d’Azure Notification Hub](notification-hubs-push-notification-registration-management.md). L’inscription en arrière-plan est effectuée à l’aide d’un élément `IntentService` nommé `RegistrationIntentService`. Ce service gère [l’actualisation du jeton d’inscription GCM](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens).

    Ajoutez la définition de service suivante au fichier AndroidManifest.xml, dans la balise `<application>` . Remplacez l’espace réservé `<your package>` par le nom de votre package actuel, qui apparaît en haut du fichier `AndroidManifest.xml`.

    ```xml
    <service
        android:name="<your package>.RegistrationIntentService"
        android:exported="false">
    </service>
    ```

3. Définissez le destinataire qui recevra les notifications. Ajoutez la définition de destinataire suivante au fichier AndroidManifest.xml, dans la balise `<application>` . Remplacez l’espace réservé `<your package>` par le nom de votre package actuel, qui apparaît en haut du fichier `AndroidManifest.xml`.

    ```xml
    <receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
        android:permission="com.google.android.c2dm.permission.SEND">
        <intent-filter>
            <action android:name="com.google.android.c2dm.intent.RECEIVE" />
            <category android:name="<your package name>" />
        </intent-filter>
    </receiver>
    ```

4. Ajoutez les autorisations GCM nécessaires suivantes sous la balise `<application>`. Remplacez `<your package>` par le nom du package qui apparaît en haut du fichier `AndroidManifest.xml`.

    Pour plus d'informations sur ces autorisations, consultez la rubrique [Configurer une application cliente GCM pour Android](https://developers.google.com/cloud-messaging/).

    ```xml
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    <uses-permission android:name="android.permission.WAKE_LOCK"/>
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

    <permission android:name="<your package>.permission.C2D_MESSAGE" android:protectionLevel="signature" />
    <uses-permission android:name="<your package>.permission.C2D_MESSAGE"/>
    ```

### <a name="adding-code"></a>Ajout de code

1. Dans la vue du projet, développez **app** > **src** > **main** > **java**. Cliquez avec le bouton droit sur le dossier de votre package sous **java**, cliquez sur **Nouveau**, puis sur **Classe Java**. Ajoutez une nouvelle classe nommée `NotificationSettings`.

    ![Android Studio - nouvelle classe Java][6]

    Mettez à jour les trois espaces réservés dans le code suivant pour la classe `NotificationSettings` :

   * `SenderId`: numéro de projet que vous avez obtenu précédemment dans la [console Google Cloud](https://cloud.google.com/console).
   * `HubListenConnectionString`: chaîne de connexion **DefaultListenAccessSignature** de votre hub. Vous pouvez copier cette chaîne de connexion en cliquant sur **Stratégies d’accès** dans la page **Paramètres** de votre hub dans le [Azure portal].
   * `HubName`: utilisez le nom de votre hub de notification qui s’affiche dans la page Hub du [Azure portal].

     `NotificationSettings` code :

     ```java
     public class NotificationSettings {
        public static String SenderId = "<Your project number>";
        public static String HubName = "<Your HubName>";
        public static String HubListenConnectionString = "<Your default listen connection string>";
     }
     ```

2. Ajoutez une nouvelle classe nommée `MyInstanceIDService`. Cette classe est l’implémentation de service d’écouteur d’ID d’instance.

    Le code de cette classe appelle `IntentService` pour [actualiser le jeton GCM](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens) en arrière-plan.

    ```java
    import android.content.Intent;
    import android.util.Log;
    import com.google.android.gms.iid.InstanceIDListenerService;

    public class MyInstanceIDService extends InstanceIDListenerService {

        private static final String TAG = "MyInstanceIDService";

        @Override
        public void onTokenRefresh() {

            Log.i(TAG, "Refreshing GCM Registration Token");

            Intent intent = new Intent(this, RegistrationIntentService.class);
            startService(intent);
        }
    };
    ```

3. Ajoutez à votre projet une nouvelle classe nommée `RegistrationIntentService`. Cette classe implémente `IntentService` qui gère [l’actualisation du jeton GCM](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens) et [l’inscription au hub de notification](notification-hubs-push-notification-registration-management.md).

    Utilisez le code suivant pour cette classe.

    ```java
    import android.app.IntentService;
    import android.content.Intent;
    import android.content.SharedPreferences;
    import android.preference.PreferenceManager;
    import android.util.Log;

    import com.google.android.gms.gcm.GoogleCloudMessaging;
    import com.google.android.gms.iid.InstanceID;
    import com.microsoft.windowsazure.messaging.NotificationHub;

    public class RegistrationIntentService extends IntentService {

        private static final String TAG = "RegIntentService";

        private NotificationHub hub;

        public RegistrationIntentService() {
            super(TAG);
        }

        @Override
        protected void onHandleIntent(Intent intent) {
            SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
            String resultString = null;
            String regID = null;

            try {
                InstanceID instanceID = InstanceID.getInstance(this);
                String token = instanceID.getToken(NotificationSettings.SenderId,
                        GoogleCloudMessaging.INSTANCE_ID_SCOPE);
                Log.i(TAG, "Got GCM Registration Token: " + token);

                // Storing the registration id that indicates whether the generated token has been
                // sent to your server. If it is not stored, send the token to your server,
                // otherwise your server should have already received the token.
                if ((regID=sharedPreferences.getString("registrationID", null)) == null) {
                    NotificationHub hub = new NotificationHub(NotificationSettings.HubName,
                            NotificationSettings.HubListenConnectionString, this);
                    Log.i(TAG, "Attempting to register with NH using token : " + token);

                    regID = hub.register(token).getRegistrationId();

                    // If you want to use tags...
                    // Refer to : https://azure.microsoft.com/documentation/articles/notification-hubs-routing-tag-expressions/
                    // regID = hub.register(token, "tag1", "tag2").getRegistrationId();

                    resultString = "Registered Successfully - RegId : " + regID;
                    Log.i(TAG, resultString);
                    sharedPreferences.edit().putString("registrationID", regID ).apply();
                } else {
                    resultString = "Previously Registered Successfully - RegId : " + regID;
                }
            } catch (Exception e) {
                Log.e(TAG, resultString="Failed to complete token refresh", e);
                // If an exception happens while fetching the new token or updating the registration data
                // on a third-party server, this ensures that we'll attempt the update at a later time.
            }

            // Notify UI that registration has completed.
            if (MainActivity.isVisible) {
                MainActivity.mainActivity.ToastNotify(resultString);
            }
        }
    }
    ```

4. Dans votre classe `MainActivity`, ajoutez les instructions `import` suivantes au début de la classe.

    ```java
    import com.google.android.gms.common.ConnectionResult;
    import com.google.android.gms.common.GoogleApiAvailability;
    import com.google.android.gms.gcm.*;
    import com.microsoft.windowsazure.notifications.NotificationsManager;
    import android.util.Log;
    import android.widget.TextView;
    import android.widget.Toast;
    import android.content.Intent;
    ```

5. Ajoutez les membres privés suivants dans la partie supérieure de la classe. Ce code [vérifie la disponibilité de Google Play Services comme recommandé par Google](https://developers.google.com/android/guides/setup#ensure_devices_have_the_google_play_services_apk).

    ```java
    public static MainActivity mainActivity;
    public static Boolean isVisible = false;
    private GoogleCloudMessaging gcm;
    private static final int PLAY_SERVICES_RESOLUTION_REQUEST = 9000;
    private static final String TAG = "MainActivity";
    ```

6. Dans votre classe `MainActivity` , ajoutez la méthode suivante à la disponibilité des services Google Play.

    ```java
    /**
        * Check the device to make sure it has the Google Play Services APK. If
        * it doesn't, display a dialog that allows users to download the APK from
        * the Google Play Store or enable it in the device's system settings.
        */
    private boolean checkPlayServices() {
        GoogleApiAvailability apiAvailability = GoogleApiAvailability.getInstance();
        int resultCode = apiAvailability.isGooglePlayServicesAvailable(this);
        if (resultCode != ConnectionResult.SUCCESS) {
            if (apiAvailability.isUserResolvableError(resultCode)) {
                apiAvailability.getErrorDialog(this, resultCode, PLAY_SERVICES_RESOLUTION_REQUEST)
                        .show();
            } else {
                Log.i(TAG, "This device is not supported by Google Play Services.");
                ToastNotify("This device is not supported by Google Play Services.");
                finish();
            }
            return false;
        }
        return true;
    }
    ```

7. Dans votre classe `MainActivity`, ajoutez le code suivant pour rechercher Google Play Services avant d’appeler votre `IntentService` pour obtenir votre jeton d’inscription GCM et vous inscrire auprès de votre hub de notification.

    ```java
    public void registerWithNotificationHubs()
    {
        Log.i(TAG, " Registering with Notification Hubs");

        if (checkPlayServices()) {
            // Start IntentService to register this application with GCM.
            Intent intent = new Intent(this, RegistrationIntentService.class);
            startService(intent);
        }
    }
    ```

8. Dans la méthode `OnCreate` de la classe `MainActivity`, ajoutez le code suivant pour lancer le processus d’inscription lorsque l’activité est créée.

    ```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mainActivity = this;
        NotificationsManager.handleNotifications(this, NotificationSettings.SenderId, MyHandler.class);
        registerWithNotificationHubs();
    }
    ```

9. Ajoutez ces méthodes supplémentaires à `MainActivity` pour vérifier l’état de l’application et afficher un rapport dans votre application.

    ```java
    @Override
    protected void onStart() {
        super.onStart();
        isVisible = true;
    }

    @Override
    protected void onPause() {
        super.onPause();
        isVisible = false;
    }

    @Override
    protected void onResume() {
        super.onResume();
        isVisible = true;
    }

    @Override
    protected void onStop() {
        super.onStop();
        isVisible = false;
    }

    public void ToastNotify(final String notificationMessage) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Toast.makeText(MainActivity.this, notificationMessage, Toast.LENGTH_LONG).show();
                TextView helloText = (TextView) findViewById(R.id.text_hello);
                helloText.setText(notificationMessage);
            }
        });
    }
    ```

10. La méthode `ToastNotify` utilise le contrôle *« Hello World »* `TextView` pour afficher en permanence un rapport d’état et des notifications dans l’application. Dans votre disposition activity_main.xml, ajoutez l’ID suivant pour ce contrôle.

    ```xml
    android:id="@+id/text_hello"
    ```

11. Ajoutez une sous-classe pour le destinataire qui est défini dans le fichier AndroidManifest.xml. Ajoutez à votre projet une autre nouvelle classe nommée `MyHandler`.

12. Ajoutez les instructions d’importation suivantes au-dessus de `MyHandler.java` :

    ```java
    import android.app.NotificationManager;
    import android.app.PendingIntent;
    import android.content.Context;
    import android.content.Intent;
    import android.os.Bundle;
    import android.support.v4.app.NotificationCompat;
    import com.microsoft.windowsazure.notifications.NotificationsHandler;
    import android.net.Uri;
    import android.media.RingtoneManager;
    ```

13. Ajoutez le code suivant pour la classe `MyHandler` afin d’en faire une sous-classe de `com.microsoft.windowsazure.notifications.NotificationsHandler`.

    Comme ce code remplace la méthode `OnReceive`, le gestionnaire rapporte les notifications reçues. Le gestionnaire envoie également la notification Push au gestionnaire de notifications Android en utilisant la méthode `sendNotification()` . La méthode `sendNotification()` doit être exécutée quand l’application n’est pas en cours d’exécution et qu’une notification est reçue.

    ```java
    public class MyHandler extends NotificationsHandler {
        public static final int NOTIFICATION_ID = 1;
        private NotificationManager mNotificationManager;
        NotificationCompat.Builder builder;
        Context ctx;

        @Override
        public void onReceive(Context context, Bundle bundle) {
            ctx = context;
            String nhMessage = bundle.getString("message");
            sendNotification(nhMessage);
            if (MainActivity.isVisible) {
                MainActivity.mainActivity.ToastNotify(nhMessage);
            }
        }

        private void sendNotification(String msg) {

            Intent intent = new Intent(ctx, MainActivity.class);
            intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);

            mNotificationManager = (NotificationManager)
                    ctx.getSystemService(Context.NOTIFICATION_SERVICE);

            PendingIntent contentIntent = PendingIntent.getActivity(ctx, 0,
                    intent, PendingIntent.FLAG_ONE_SHOT);

            Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
            NotificationCompat.Builder mBuilder =
                    new NotificationCompat.Builder(ctx)
                            .setSmallIcon(R.mipmap.ic_launcher)
                            .setContentTitle("Notification Hub Demo")
                            .setStyle(new NotificationCompat.BigTextStyle()
                                    .bigText(msg))
                            .setSound(defaultSoundUri)
                            .setContentText(msg);

            mBuilder.setContentIntent(contentIntent);
            mNotificationManager.notify(NOTIFICATION_ID, mBuilder.build());
        }
    }
    ```

14. Dans Android Studio, sur la barre de menus, cliquez sur **Build** > **Rebuild Project** pour vérifier que votre code ne contient aucune erreur.

## <a name="testing-your-app"></a>Test de votre application

### <a name="run-the-mobile-application"></a>Exécuter l’application mobile

1. Exécutez l’application et notez qu’un ID d’inscription apparaît quand l’inscription réussit.

      ![Tests sur Android - Inscription de canal][18]
2. Entrez le message de notification à envoyer à tous les appareils Android inscrits auprès du hub.

      ![Tests sur Android - envoi d’un message][19]

3. Appuyez sur **Envoyer une notification**. Tous les appareils sur lesquels l’application est en cours d’exécution affichent une instance `AlertDialog` comportant le message de notification Push. Les appareils sur lesquels l’application n’est pas en cours d’exécution, mais qui ont déjà été inscrits aux notifications Push, reçoivent une notification dans le gestionnaire de notifications Android. Vous pouvez afficher les messages de notification en effectuant un balayage vers le bas depuis l’angle supérieur gauche.

      ![Tests sur Android - notifications][21]

### <a name="test-send-push-notifications-from-the-azure-portal"></a>Tester l’envoi de notifications Push à partir du portail Azure

Vous pouvez tester la réception des notifications Push dans votre application en les envoyant via le [Azure portal].

1. Sélectionnez **Envoi de test** dans la section **Résolution des problèmes**.
2. Pour les **plateformes**, sélectionnez **Android**.
3. Sélectionnez **Envoyer** pour envoyer la notification test.
4. Vérifiez que le message de notification s’affiche sur l’appareil Android.

    ![Azure Notification Hubs - Test d’envoi](./media/notification-hubs-android-get-started/notification-hubs-test-send.png)

[!INCLUDE [notification-hubs-sending-notifications-from-the-portal](../../includes/notification-hubs-sending-notifications-from-the-portal.md)]

#### <a name="push-notifications-in-the-emulator"></a>Notifications Push dans l’émulateur

Si vous souhaitez effectuer les tests de notifications Push sur un émulateur, vérifiez que votre image d’émulateur prend en charge le niveau d’API Google que vous avez choisi pour votre application. Si votre image ne prend pas en charge les API Google natives, l’exception **SERVICE\_NOT\_AVAILABLE** est levée.

De plus, assurez-vous que vous avez ajouté votre compte Google à l’émulateur en cours d’exécution sous **Paramètres** > **Comptes**. Sinon, vos tentatives d’inscription auprès de GCM peuvent entraîner la levée de l’exception **AUTHENTICATION\_FAILED**.

## <a name="optional-send-push-notifications-directly-from-the-app"></a>(Facultatif) Envoyer des notifications Push directement depuis l’application

En règle générale, vous devez envoyer des notifications à l'aide d'un serveur principal. Dans la plupart des cas, vous chercherez peut-être à envoyer des notifications Push directement à partir de l’application cliente. Cette section explique comment envoyer des notifications à partir du client avec [l’API REST d’Azure Notification Hub](/previous-versions/azure/reference/dn223264(v=azure.100)).

1. Dans la vue de projet Android Studio, développez **App** > **src** > **main** > **res** > **layout**. Ouvrez le fichier de disposition `activity_main.xml` et cliquez sur l’onglet **Texte** pour mettre à jour le texte du fichier. Mettez-le à jour avec le code suivant, qui ajoute de nouveaux contrôles `Button` et `EditText` pour l’envoi de messages de notification Push au hub de notification. Ajoutez ce code en bas, juste avant `</RelativeLayout>`.

    ```xml
    <Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/send_button"
    android:id="@+id/sendbutton"
    android:layout_centerVertical="true"
    android:layout_centerHorizontal="true"
    android:onClick="sendNotificationButtonOnClick" />

    <EditText
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:id="@+id/editTextNotificationMessage"
    android:layout_above="@+id/sendbutton"
    android:layout_centerHorizontal="true"
    android:layout_marginBottom="42dp"
    android:hint="@string/notification_message_hint" />
    ```

2. Dans la vue de projet Android Studio, développez **App** > **src** > **main** > **res** > **values**. Ouvrez le fichier `strings.xml` et ajoutez les valeurs de chaîne référencées par les nouveaux contrôles `Button` et `EditText`. Ajoutez les lignes suivantes en bas du fichier, juste avant `</resources>`.

    ```xml
    <string name="send_button">Send Notification</string>
    <string name="notification_message_hint">Enter notification message text</string>
    ```

3. Dans votre fichier `NotificationSetting.java`, ajoutez le paramètre suivant à la classe `NotificationSettings`.

    Mettez à jour `HubFullAccess` avec la chaîne de connexion **DefaultFullSharedAccessSignature** correspondant à votre hub. Vous pouvez copier cette chaîne de connexion à partir du [Azure portal] en cliquant sur **Stratégies d’accès** dans la page **Paramètres** de votre hub de notification.

    ```java
    public static String HubFullAccess = "<Enter Your DefaultFullSharedAccess Connection string>";
    ```

4. Ajoutez les instructions `import` suivantes au début du fichier `MainActivity.java`.

    ```java
    import java.io.BufferedOutputStream;
    import java.io.BufferedReader;
    import java.io.InputStreamReader;
    import java.io.OutputStream;
    import java.net.HttpURLConnection;
    import java.net.URL;
    import java.net.URLEncoder;
    import javax.crypto.Mac;
    import javax.crypto.spec.SecretKeySpec;
    import android.util.Base64;
    import android.view.View;
    import android.widget.EditText;
    ```

5. Dans votre fichier `MainActivity.java`, ajoutez les membres suivants au-dessus de la classe `MainActivity`.

    ```java
    private String HubEndpoint = null;
    private String HubSasKeyName = null;
    private String HubSasKeyValue = null;
    ```

6. Créez un jeton SAP (signature d'accès partagé) pour authentifier une requête POST d'envoi de messages à votre hub de notification. Analysez les données clés de la chaîne de connexion, puis créez le jeton SaS, comme indiqué sur la page [Concepts courants](/previous-versions/azure/reference/dn495627(v=azure.100)) des informations de référence sur l’API REST. Le code suivant est un exemple d'implémentation.

    Dans `MainActivity.java`, ajoutez la méthode suivante à la classe `MainActivity` pour analyser votre chaîne de connexion.

    ```java
    /**
        * Example code from https://msdn.microsoft.com/library/azure/dn495627.aspx
        * to parse the connection string so a SaS authentication token can be
        * constructed.
        *
        * @param connectionString This must be the DefaultFullSharedAccess connection
        *                         string for this example.
        */
    private void ParseConnectionString(String connectionString)
    {
        String[] parts = connectionString.split(";");
        if (parts.length != 3)
            throw new RuntimeException("Error parsing connection string: "
                    + connectionString);

        for (int i = 0; i < parts.length; i++) {
            if (parts[i].startsWith("Endpoint")) {
                this.HubEndpoint = "https" + parts[i].substring(11);
            } else if (parts[i].startsWith("SharedAccessKeyName")) {
                this.HubSasKeyName = parts[i].substring(20);
            } else if (parts[i].startsWith("SharedAccessKey")) {
                this.HubSasKeyValue = parts[i].substring(16);
            }
        }
    }
    ```

7. Dans `MainActivity.java`, ajoutez la méthode suivante à la classe `MainActivity` pour créer un jeton d’authentification SaS.

    ```java
    /**
        * Example code from https://msdn.microsoft.com/library/azure/dn495627.aspx to
        * construct a SaS token from the access key to authenticate a request.
        *
        * @param uri The unencoded resource URI string for this operation. The resource
        *            URI is the full URI of the Service Bus resource to which access is
        *            claimed. For example,
        *            "http://<namespace>.servicebus.windows.net/<hubName>"
        */
    private String generateSasToken(String uri) {

        String targetUri;
        String token = null;
        try {
            targetUri = URLEncoder
                    .encode(uri.toString().toLowerCase(), "UTF-8")
                    .toLowerCase();

            long expiresOnDate = System.currentTimeMillis();
            int expiresInMins = 60; // 1 hour
            expiresOnDate += expiresInMins * 60 * 1000;
            long expires = expiresOnDate / 1000;
            String toSign = targetUri + "\n" + expires;

            // Get an hmac_sha1 key from the raw key bytes
            byte[] keyBytes = HubSasKeyValue.getBytes("UTF-8");
            SecretKeySpec signingKey = new SecretKeySpec(keyBytes, "HmacSHA256");

            // Get an hmac_sha1 Mac instance and initialize with the signing key
            Mac mac = Mac.getInstance("HmacSHA256");
            mac.init(signingKey);

            // Compute the hmac on input data bytes
            byte[] rawHmac = mac.doFinal(toSign.getBytes("UTF-8"));

            // Using android.util.Base64 for Android Studio instead of
            // Apache commons codec
            String signature = URLEncoder.encode(
                    Base64.encodeToString(rawHmac, Base64.NO_WRAP).toString(), "UTF-8");

            // Construct authorization string
            token = "SharedAccessSignature sr=" + targetUri + "&sig="
                    + signature + "&se=" + expires + "&skn=" + HubSasKeyName;
        } catch (Exception e) {
            if (isVisible) {
                ToastNotify("Exception Generating SaS : " + e.getMessage().toString());
            }
        }

        return token;
    }
    ```

8. Dans `MainActivity.java`, ajoutez la méthode suivante à la classe `MainActivity` pour gérer le clic sur le bouton **Envoyer une notification** et envoyer le message de notification Push au hub à l’aide de l’API REST intégrée.

    ```java
    /**
        * Send Notification button click handler. This method parses the
        * DefaultFullSharedAccess connection string and generates a SaS token. The
        * token is added to the Authorization header on the POST request to the
        * notification hub. The text in the editTextNotificationMessage control
        * is added as the JSON body for the request to add a GCM message to the hub.
        *
        * @param v
        */
    public void sendNotificationButtonOnClick(View v) {
        EditText notificationText = (EditText) findViewById(R.id.editTextNotificationMessage);
        final String json = "{\"data\":{\"message\":\"" + notificationText.getText().toString() + "\"}}";

        new Thread()
        {
            public void run()
            {
                try
                {
                    // Based on reference documentation...
                    // https://msdn.microsoft.com/library/azure/dn223273.aspx
                    ParseConnectionString(NotificationSettings.HubFullAccess);
                    URL url = new URL(HubEndpoint + NotificationSettings.HubName +
                            "/messages/?api-version=2015-01");

                    HttpURLConnection urlConnection = (HttpURLConnection)url.openConnection();

                    try {
                        // POST request
                        urlConnection.setDoOutput(true);

                        // Authenticate the POST request with the SaS token
                        urlConnection.setRequestProperty("Authorization",
                            generateSasToken(url.toString()));

                        // Notification format should be GCM
                        urlConnection.setRequestProperty("ServiceBusNotification-Format", "gcm");

                        // Include any tags
                        // Example below targets 3 specific tags
                        // Refer to : https://azure.microsoft.com/documentation/articles/notification-hubs-routing-tag-expressions/
                        // urlConnection.setRequestProperty("ServiceBusNotification-Tags",
                        //        "tag1 || tag2 || tag3");

                        // Send notification message
                        urlConnection.setFixedLengthStreamingMode(json.length());
                        OutputStream bodyStream = new BufferedOutputStream(urlConnection.getOutputStream());
                        bodyStream.write(json.getBytes());
                        bodyStream.close();

                        // Get response
                        urlConnection.connect();
                        int responseCode = urlConnection.getResponseCode();
                        if ((responseCode != 200) && (responseCode != 201)) {
                            BufferedReader br = new BufferedReader(new InputStreamReader((urlConnection.getErrorStream())));
                            String line;
                            StringBuilder builder = new StringBuilder("Send Notification returned " +
                                    responseCode + " : ")  ;
                            while ((line = br.readLine()) != null) {
                                builder.append(line);
                            }

                            ToastNotify(builder.toString());
                        }
                    } finally {
                        urlConnection.disconnect();
                    }
                }
                catch(Exception e)
                {
                    if (isVisible) {
                        ToastNotify("Exception Sending Notification : " + e.getMessage().toString());
                    }
                }
            }
        }.start();
    }
    ```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez envoyé des notifications de diffusion à tous vos appareils Android inscrits auprès du serveur principal. Pour découvrir comment envoyer des notifications Push à des appareils Android spécifiques, passez au didacticiel suivant :  

 > [!div class="nextstepaction"]
 > [Notifications Push vers des appareils spécifiques](notification-hubs-aspnet-backend-android-xplat-segmented-gcm-push-notification.md)

<!-- Images. -->
[6]: ./media/notification-hubs-android-get-started/notification-hub-android-new-class.png
[12]: ./media/notification-hubs-android-get-started/notification-hub-connection-strings.png
[13]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-new-project.png
[14]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-choose-form-factor.png
[15]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app4.png
[16]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app5.png
[17]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app6.png
[18]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-registered.png
[19]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-set-message.png
[20]: ./media/notification-hubs-android-get-started/notification-hub-create-console-app.png
[21]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-received-message.png
[22]: ./media/notification-hubs-android-get-started/notification-hub-scheduler1.png
[23]: ./media/notification-hubs-android-get-started/notification-hub-scheduler2.png
[29]: ./media/mobile-services-android-get-started-push/mobile-eclipse-import-Play-library.png
[30]: ./media/notification-hubs-android-get-started/notification-hubs-debug-hub-gcm.png
[31]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-add-ui.png

<!-- URLs. -->
[Get started with push notifications in Mobile Services]: ../mobile-services-javascript-backend-android-get-started-push.md
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[Referencing a library project]: https://go.microsoft.com/fwlink/?LinkId=389800
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Use Notification Hubs to push notifications to users]: notification-hubs-aspnet-backend-gcm-android-push-to-user-google-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-aspnet-backend-android-xplat-segmented-gcm-push-notification.md
[Azure portal]: https://portal.azure.com
