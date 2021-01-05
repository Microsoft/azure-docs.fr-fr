---
author: alexeystrakh
ms.author: alstrakh
ms.date: 06/11/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: 9330488debb0b7611fdd63656e4c555566c749dd
ms.sourcegitcommit: bcb962e74ee5302d0b9242b1ee006f769a94cfb8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/07/2020
ms.locfileid: "86060437"
---
### <a name="configure-required-android-packages"></a>Configurer les packages Android requis

Le package est [automatiquement lié](https://github.com/react-native-community/cli/blob/master/docs/autolinking.md) lors de la génération de l’application. Quelques étapes supplémentaires sont nécessaires pour terminer le processus de configuration.

### <a name="configure-android-manifest"></a>Configurer le manifeste Android

Dans votre fichier « android/app/src/main/AndroidManifest.xml », vérifiez le nom du package, les autorisations et les services requis. Assurez-vous que vous avez inscrit les récepteurs `RNPushNotificationPublisher` et `RNPushNotificationBootEventReceiver`, et inscrit le service `RNPushNotificationListenerService`. Les métadonnées de notification peuvent être utilisées pour personnaliser l’apparence de vos notifications Push.

  ```xml
  <manifest xmlns:android="http://schemas.android.com/apk/res/android"
      package="YOUR_PACKAGE_NAME">

        <uses-permission android:name="android.permission.INTERNET" />
        <uses-permission android:name="android.permission.WAKE_LOCK" />
        <uses-permission android:name="android.permission.VIBRATE" />
        <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

        <application
          android:name=".MainApplication"
          android:label="@string/app_name"
          android:usesCleartextTraffic="true"
          android:icon="@mipmap/ic_launcher"
          android:roundIcon="@mipmap/ic_launcher_round"
          android:allowBackup="false"
          android:theme="@style/AppTheme">

          <meta-data  android:name="com.dieam.reactnativepushnotification.notification_channel_name"
                      android:value="PushDemo Channel"/>
          <meta-data  android:name="com.dieam.reactnativepushnotification.notification_channel_description"
                      android:value="PushDemo Channel Description"/>
          <meta-data  android:name="com.dieam.reactnativepushnotification.notification_foreground"
                      android:value="true"/>
          <meta-data  android:name="com.dieam.reactnativepushnotification.notification_color"
                      android:resource="@android:color/white"/>

          <receiver android:name="com.dieam.reactnativepushnotification.modules.RNPushNotificationPublisher" />
          <receiver android:name="com.dieam.reactnativepushnotification.modules.RNPushNotificationBootEventReceiver">
              <intent-filter>
                  <action android:name="android.intent.action.BOOT_COMPLETED" />
              </intent-filter>
          </receiver>

          <service
              android:name="com.dieam.reactnativepushnotification.modules.RNPushNotificationListenerService"
              android:exported="false" >
              <intent-filter>
                  <action android:name="com.google.firebase.MESSAGING_EVENT" />
              </intent-filter>
          </service>

          <activity
            android:name=".MainActivity"
            android:label="@string/app_name"
            android:configChanges="keyboard|keyboardHidden|orientation|screenSize|uiMode"
            android:launchMode="singleTask"
            android:windowSoftInputMode="adjustResize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
          </activity>
          <activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />
        </application>

  </manifest>
  ```

### <a name="configure-google-services"></a>Configurer les services Google

Dans « android/app/build.gradle », inscrivez les services Google :

```gradle
dependencies {
  ...
  implementation 'com.google.firebase:firebase-analytics:17.3.0'
  ...
}

apply plugin: 'com.google.gms.google-services'
```

Copiez le fichier « google-services.json » que vous avez téléchargé lors de l’installation de FCM dans le dossier de projet « android/app/ ».

### <a name="handle-push-notifications-for-android"></a>Gérer les notifications Push pour Android

Vous avez configuré le service `RNPushNotificationListenerService` existant pour gérer les notifications Push Android entrantes. Ce service a été inscrit précédemment dans le manifeste de l’application. Il traite les notifications entrantes et les transmet par proxy à la partie React Native multiplateforme. Aucune étape supplémentaire n’est nécessaire.
