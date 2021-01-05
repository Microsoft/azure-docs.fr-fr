---
title: Envoyer des notifications Push à Android avec Azure Notification Hubs et le SDK Firebase version 1.0.0-preview1
description: Dans ce tutoriel, vous allez découvrir comment utiliser Azure Notification Hubs et Google Firebase Cloud Messaging pour envoyer des notifications Push à des appareils Android (version 1.0.0-preview1).
author: sethmanheim
ms.author: sethm
ms.date: 5/28/2020
ms.topic: tutorial
ms.service: notification-hubs
ms.reviewer: thsomasu
ms.lastreviewed: 05/27/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 63841bd603373d0fb325bcf82511ce3fb07b4136
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017251"
---
# <a name="tutorial-send-push-notifications-to-android-devices-using-firebase-sdk-version-100-preview1"></a>Tutoriel : Envoyer des notifications Push à des appareils Android à l’aide du SDK Firebase version 1.0.0-preview1

Ce tutoriel montre comment utiliser Azure Notification Hubs et la version mise à jour du SDK Firebase Cloud Messaging (FCM) (version 1.0.0-preview1) pour envoyer des notifications Push à une application Android. Dans ce tutoriel, vous allez créer une application Android vide qui reçoit des notifications Push à l’aide de Firebase Cloud Messaging (FCM).

Vous pouvez télécharger le code complet pour ce tutoriel à partir de [GitHub](https://github.com/Azure/azure-notificationhubs-android/tree/v1-preview/notification-hubs-test-app-refresh).

Ce didacticiel couvre les étapes suivantes :

- Créer un projet Android Studio.
- Créer un projet qui prend en charge Firebase Cloud Messaging.
- Créer un hub de notification.
- Connecter votre application au hub.
- Tester l'application.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous avez besoin d'un compte Azure actif. Si vous ne possédez pas de compte, vous pouvez créer un compte d’évaluation gratuit en quelques minutes. Pour plus d’informations, consultez [Essai gratuit Azure](https://azure.microsoft.com/free/).

Vous devez également disposer des éléments suivants :

- La dernière version de [Android Studio](https://go.microsoft.com/fwlink/?LinkId=389797) est recommandée.
- Le niveau d’API 16 constitue la prise en charge minimale.

## <a name="create-an-android-studio-project"></a>Créer un projet Android Studio

La première étape consiste à créer un projet dans Android Studio :

1. Lancez Android Studio.

2. Sélectionnez **File** (Fichier), puis **New** (Nouveau), puis **New Project** (Nouveau projet).

3. Dans la page **Choose your project** (Choisir votre projet), sélectionnez **Empty Activity** (Activité vide), puis sélectionnez **Next** (Suivant).

4. Dans la page **Configure your project** (Configurer votre projet), effectuez les actions suivantes :
   1. Entrez un nom pour l’application.
   2. Spécifiez un emplacement où enregistrer les fichiers du projet.
   3. Sélectionnez **Terminer**.

   :::image type="content" source="media/android-sdk/configure-project.png" alt-text="Configurer un projet":::

## <a name="create-a-firebase-project-that-supports-fcm"></a>Créer un projet qui prend en charge FCM

1. Connectez-vous à la [console Firebase](https://firebase.google.com/console/). Créer un nouveau projet Firebase si vous n’en avez pas encore.

2. Une fois le projet créé, sélectionnez **Add Firebase to your Android app** (Ajouter Firebase à votre application Android).

   :::image type="content" source="media/android-sdk/get-started.png" alt-text="Ajouter Firebase":::

3. Dans la page **Add Firebase to your Android app** (Ajouter Firebase à votre application Android), procédez comme suit :

   1. Pour le **Nom de package Android**, copiez la valeur de **applicationId** dans le fichier **build.gradle** de votre application. Dans cet exemple, il s’agit de `com.fabrikam.fcmtutorial1app`.

      :::image type="content" source="media/android-sdk/specify-package-name-fcm-settings.png" alt-text="Spécifier le nom du package":::

   2. Sélectionnez **Inscrire une application**.

4. Sélectionnez **Télécharger google-services.json**, enregistrez le fichier dans le dossier **app** de votre projet, puis sélectionnez **Suivant**.

   :::image type="content" source="media/android-sdk/download-google-service-button.png" alt-text="Télécharger le service Google":::

5. Dans la console Firebase, sélectionnez la roue dentée associée à votre projet. Ensuite, sélectionnez **Project Settings** (Paramètres du projet).

   :::image type="content" source="media/android-sdk/notification-hubs-firebase-console-project-settings.png" alt-text="Paramètres du projet":::

6. Si vous n’avez pas encore téléchargé le fichier **google-services.json** dans le dossier **application** de votre projet Android Studio, vous pouvez le faire à partir de cette page.

7. Basculez vers l’onglet **Cloud Messaging** (Messagerie cloud).

8. Copiez et enregistrez la **clé du serveur** pour une utilisation ultérieure. Vous utilisez cette valeur pour configurer votre hub.

## <a name="configure-a-notification-hub"></a>Configurer un hub de notification

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Sélectionnez **Tous les services** dans le menu de gauche, puis sélectionnez **Notification Hubs** dans la section **Mobile**. Sélectionnez l’icône d’étoile en regard du nom du service pour l’ajouter à la section **FAVORIS** dans le menu de gauche. Après avoir ajouté **Notification Hubs** aux **FAVORIS**, sélectionnez-le dans le menu de gauche.

3. Dans la page **Notification Hubs**, sélectionnez **Ajouter** dans la barre d’outils.

   :::image type="content" source="media/android-sdk/add-hub.png" alt-text="Ajouter un hub":::

4. Dans la page **Notification Hubs**, effectuez les actions suivantes :

   1. Entrez un nom dans **Notification Hub**.

   2. Entrez un nom dans **Créer un espace de noms**. Un espace de noms contient un ou plusieurs hubs.

   3. Sélectionnez une valeur dans la zone déroulante **Emplacement**. Cette valeur spécifie l’emplacement où vous voulez créer le hub.

   4. Sélectionnez un groupe de ressources existant dans **Groupe de ressources** ou créez-en un.

   5. Sélectionnez **Create** (Créer).

      :::image type="content" source="media/android-sdk/create-hub.png" alt-text="Créer un hub":::

5. Sélectionnez **Notifications** (icône de cloche), puis **Accéder à la ressource**. Vous pouvez également actualiser la liste dans la page **Notification Hubs** et sélectionner votre hub.

   :::image type="content" source="media/android-sdk/notification-hubs.png" alt-text="Sélectionner un hub":::

6. Sélectionnez **Stratégies d’accès** dans la liste. Notez que deux chaînes de connexion sont disponibles. Vous en aurez besoin pour gérer les notifications Push.

   :::image type="content" source="media/android-sdk/access-policies.png" alt-text="Stratégies d’accès":::

   > [!IMPORTANT]
   > N’utilisez pas la stratégie **DefaultFullSharedAccessSignature** dans votre application. Cette stratégie doit être utilisée uniquement dans le back-end de l’application.

## <a name="configure-firebase-cloud-messaging-settings-for-the-hub"></a>Configurer les paramètres de Firebase Cloud Messaging pour le hub

1. Dans le volet gauche, sous **Settings** (Paramètres), sélectionnez **Google (GCM/FCM)** .

2. Entrez la **clé serveur** pour le projet FCM que vous avez enregistré précédemment.

3. Dans la barre d’outils, sélectionnez **Enregistrer**.

   :::image type="content" source="media/android-sdk/fcm-server-key.png" alt-text="Clé serveur":::

4. Le portail Azure affiche un message indiquant que le hub a été correctement mis à jour. Le bouton **Enregistrer** est désactivé.

Votre hub de notification est désormais configuré pour fonctionner avec Firebase Cloud Messaging. Vous disposez également des chaînes de connexion nécessaires pour envoyer des notifications à un appareil et inscrire une application pour recevoir des notifications.

## <a name="connect-your-app-to-the-notification-hub"></a>Connexion de votre application au hub de notification

### <a name="add-google-play-services-to-the-project"></a>Ajout de services Google Play au projet

1. Dans Android Studio, sélectionnez **Outils** dans le menu, puis **SDK Manager**.

2. Sélectionnez la version cible du Kit de développement logiciel (SDK) Android qui est utilisé dans votre projet. Puis sélectionnez **Afficher les détails du package**.

   :::image type="content" source="media/android-sdk/notification-hubs-android-studio-sdk-manager.png" alt-text="Gestionnaire SDK":::

3. Sélectionnez **API Google**, si elles ne sont pas déjà installées.

   :::image type="content" source="media/android-sdk/google-apis-selected.png" alt-text="API":::

4. Basculez vers l’onglet **SDK Tools**. Si vous n’avez pas déjà installé un service Google Play, sélectionnez **Google Play Services** comme indiqué dans l’image suivante. Sélectionnez ensuite **Appliquer** pour procéder à l’installation. Notez le chemin du Kit de développement logiciel (SDK). Vous l’utiliserez à une étape suivante.

   :::image type="content" source="media/android-sdk/google-play-services-selected.png" alt-text="Google Play Services":::

5. Si la boîte de dialogue de **confirmation de la modification** s’affiche, sélectionnez **OK**. Le programme d’installation des composants installe les composants demandés. Sélectionnez **Terminer** une fois que les composants sont installés.

6. Sélectionnez **OK** pour fermer la boîte de dialogue **Paramètres pour les nouveaux projets**.

### <a name="add-azure-notification-hubs-libraries"></a>Ajoutez des bibliothèques Azure Notification Hubs

1. Dans la section dependencies du fichier **Build.Gradle** de l’application, ajoutez les lignes suivantes :

   ```gradle
   implementation 'com.microsoft.azure:notification-hubs-android-sdk:1.0.0-preview1@aar'
   implementation 'androidx.appcompat:appcompat:1.0.0'

   implementation 'com.google.firebase:firebase-messaging:20.1.5'

   implementation 'com.android.volley:volley:1.1.1'
   ```

2. Ajoutez le dépôt suivant après la section dependencies :

   ```gradle
   repositories {
      maven {
         url "https://dl.bintray.com/microsoftazuremobile/SDK"
      }
   }
   ```

### <a name="add-google-firebase-support"></a>Ajouter la prise en charge de Google Firebase

1. Ajoutez le plug-in suivant à la fin du fichier s’il n’y figure pas encore.

   ```gradle
   apply plugin: 'com.google.gms.google-services'
   ```

2. Sélectionnez **Synchroniser maintenant** dans la barre d’outils.

### <a name="add-code"></a>Ajout de code

1. Créez un objet **NotificationHubListener**, qui gère l’interception des messages provenant d’Azure Notification Hubs.

   ```csharp
   public class CustomNotificationListener implements NotificationHubListener {

      @override

      public void onNotificationReceived(Context context, NotificationMessage message) {

      /* The following notification properties are available. */

      String title = message.getTitle();
      String message = message.getMessage();
      Map<String, String> data = message.getData();

      if (message != null) {
         Log.d(TAG, "Message Notification Title: " + title);
         Log.d(TAG, "Message Notification Body: " + message);
         }
      }
   }
   ```

2. Dans la méthode `OnCreate` de la classe `MainActivity`, ajoutez le code suivant pour démarrer le processus d’initialisation de Notification Hubs quand l’activité est créée :

   ```java
   @Override
   protected void onCreate(Bundle savedInstanceState) {

      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity\_main);
      NotificationHub.setListener(new CustomNotificationListener());
      NotificationHub.initialize(this.getApplication(), “Connection-String”, "Hub Name");

   }
   ```

3. Dans la barre de menus de Android Studio, sélectionnez **Build** (Générer), puis **Rebuild Project** (Regénérer le projet) pour vérifier que votre code ne contient pas d’erreur. Si vous recevez une erreur concernant l’icône **ic_launcher**, supprimez l’instruction suivante du fichier AndroidManifest.xml :

   ```xml
   android:icon="@mipmap/ic_launcher"
   ```

4. Vérifiez que vous disposez d’un appareil virtuel pour l’exécution de l’application. Si vous n’en avez pas, ajoutez-en un de la façon suivante :

   1. :::image type="content" source="media/android-sdk/open-device-manager.png" alt-text="Gestionnaire d’appareils":::
   2. :::image type="content" source="media/android-sdk/your-virtual-devices.png" alt-text="Appareils virtuels":::
   3. Exécutez l’application sur l’appareil sélectionné, puis vérifiez que son inscription s’effectue correctement auprès du hub.

      :::image type="content" source="media/android-sdk/device-registration.png" alt-text="Enregistrement de l’appareil":::

      > [!NOTE]
      > L’inscription peut échouer lors du lancement initial, jusqu’à ce que la méthode `onTokenRefresh()` du service d’ID d’instance soit appelée. Une actualisation doit lancer une inscription réussie auprès du hub de notification.

## <a name="send-a-test-notification"></a>Envoyer une notification de test

Vous pouvez envoyer des notifications Push à votre hub de notification à partir du [portail Azure](https://portal.azure.com/), comme suit :

1. Sur le portail Azure, dans la page Hub de notification de votre hub, à la section **Résolution des problèmes**, sélectionnez **Envoi de test**.

2. Dans **Plateformes**, sélectionnez **Android**.

3. Sélectionnez **Envoyer**. Vous ne voyez pas encore de notification sur l’appareil Android parce que vous n’avez pas exécuté l’application mobile sur celui-ci. Après avoir exécuté l’application mobile, sélectionnez de nouveau le bouton **Envoyer** pour voir le message de notification.

4. Consultez le résultat de l’opération dans la liste affichée en bas de la page du portail.

   :::image type="content" source="media/android-sdk/notification-hubs-test-send.png" alt-text="Envoyer une notification de test":::

5. Vous voyez le message de notification sur votre appareil.

Les notifications Push sont normalement envoyées dans un service back-end comme Mobile Apps ou ASP.NET à l’aide d’une bibliothèque compatible. Si aucune bibliothèque n’est disponible pour votre backend, vous pouvez également utiliser l’API REST directement pour envoyer des messages de notification.

## <a name="run-the-mobile-app-on-emulator"></a>Exécuter l’application mobile sur un émulateur

Avant de tester les notifications Push dans un émulateur, vérifiez que votre image d’émulateur prend en charge le niveau d’API Google que vous avez choisi pour votre application. Si votre image ne prend pas en charge les API Google natives, vous pouvez obtenir une exception **SERVICE_NOT_AVAILABLE**.

Vérifiez aussi que vous avez ajouté votre compte Google à l’émulateur en cours d’exécution sous **Paramètres** > **Comptes**. Dans le cas contraire, vos tentatives d’inscription auprès de FCM risquent d’entraîner la levée d’une exception **AUTHENTICATION_FAILED**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez utilisé Firebase Cloud Messaging pour diffuser des notifications à tous les appareils Android inscrits auprès du service. Pour savoir comment envoyer des notifications Push à des appareils spécifiques, passez au didacticiel suivant :

> [!div class="nextstepaction"]
>[Tutoriel : Envoyer des notifications à des utilisateurs spécifiques](push-notifications-android-specific-users-firebase-cloud-messaging.md)

La liste suivante indique certains autres tutoriels relatif à l’envoi de notifications :

- Azure Mobile Apps : pour découvrir un exemple de la procédure d’envoi de notifications à partir d’un serveur back-end Mobile Apps intégré à Notification Hubs, consultez [Ajout de notifications Push à votre application iOS](/previous-versions/azure/app-service-mobile/app-service-mobile-ios-get-started-push).

- ASP.NET : [Utiliser Notification Hubs pour envoyer des notifications Push à des utilisateurs](notification-hubs-aspnet-backend-ios-apple-apns-notification.md).

- Kit de développement logiciel (SDK) Java pour Azure Notification Hubs : consultez [Utilisation de Notification Hubs à partir de Java](notification-hubs-java-push-notification-tutorial.md) pour envoyer des notifications depuis Java. Il a été testé dans Eclipse pour le développement Android.

- PHP : [Utilisation de Notification Hubs à partir de PHP](notification-hubs-php-push-notification-tutorial.md).