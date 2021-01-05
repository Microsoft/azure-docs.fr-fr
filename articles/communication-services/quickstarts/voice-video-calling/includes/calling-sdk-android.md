---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 9/1/2020
ms.author: mikben
ms.openlocfilehash: c015561e66d77e6df352e601bf1a67da5996d4d5
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94915505"
---
## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Une ressource Communication Services déployée. [Créez une ressource Communication Services](../../create-communication-resource.md).
- Un `User Access Token` pour activer le client d’appel. Pour en savoir plus, consultez [Comment obtenir un `User Access Token`](../../access-tokens.md)
- Facultatif : Suivez le guide de démarrage rapide pour [prendre en main l’ajout des appels à votre application](../getting-started-with-calling.md)

## <a name="setting-up"></a>Configuration

### <a name="install-the-package"></a>Installer le package

<!-- TODO: update with instructions on how to download, install and add package to project -->
Recherchez votre niveau de projet build.gradle et veillez à ajouter `mavenCentral()` à la liste des référentiels sous `buildscript` et `allprojects`.
```groovy
buildscript {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```

```groovy
allprojects {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```
Ensuite, dans votre niveau de module build.gradle, ajoutez les lignes suivantes à la section dépendances.

```groovy
dependencies {
    ...
    implementation 'com.azure.android:azure-communication-calling:1.0.0-beta.2'
    ...
}

```

## <a name="object-model"></a>Modèle objet

Les classes et les interfaces suivantes gèrent certaines des principales fonctionnalités de la bibliothèque de client Appel Azure Communication Services :

| Nom                                  | Description                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| CallClient| CallClient est le point d’entrée principal de la bibliothèque de client Appel.|
| CallAgent | CallAgent sert à démarrer et à gérer les appels. |
| CommunicationUserCredential | CommunicationUserCredential est utilisé comme informations d’identification du jeton pour instancier CallAgent.|

## <a name="initialize-the-callclient-create-a-callagent-and-access-the-devicemanager"></a>Initialiser CallClient, créer CallAgent et accéder à DeviceManager

Pour créer une instance `CallAgent`, vous devez appeler la méthode `createCallAgent` sur une instance `CallClient`. Cette opération renvoie un objet d’instance `CallAgent` de manière asynchrone.
La méthode `createCallAgent` prend un `CommunicationUserCredential` comme argument, qui encapsule un [jeton d’accès](../../access-tokens.md).
Pour accéder au `DeviceManager`, une instance callAgent doit être créée en premier lieu, puis vous pouvez utiliser la méthode `CallClient.getDeviceManager` pour obtenir le DeviceManager.

```java
String userToken = '<user token>';
CallClient callClient = new CallClient();
CommunicationUserCredential tokenCredential = new CommunicationUserCredential(userToken);
android.content.Context appContext = this.getApplicationContext(); // From within an Activity for instance
CallAgent callAgent = await callClient.createCallAgent((appContext, tokenCredential).get();
DeviceManage deviceManager = await callClient.getDeviceManager().get();
```

## <a name="place-an-outgoing-call-and-join-a-group-call"></a>Passer un appel sortant et joindre un appel de groupe

Pour créer et démarrer un appel, vous devez appeler la méthode `CallAgent.call()` et fournir le `Identifier` du ou des appelé(s).
Pour rejoindre un appel de groupe, vous devez appeler la méthode `CallAgent.join()` et fournir l’ID de groupe. Les ID de groupe doivent être au format GUID ou UUID.

La création et le démarrage de l’appel sont synchrones. L’instance d’appel vous permet de vous abonner à tous les événements de l’appel.

### <a name="place-a-11-call-to-a-user"></a>Passer un appel 1:1 à un utilisateur
Pour passer un appel à un autre utilisateur de Communication Services, appelez la méthode `call` sur `callAgent` et transmettez un objet avec la clé `communicationUserId`.
```java
StartCallOptions startCallOptions = new StartCallOptions();
Context appContext = this.getApplicationContext();
CommunicationUser acsUserId = new CommunicationUser(<USER_ID>);
CommunicationUser participants[] = new CommunicationUser[]{ acsUserId };
call oneToOneCall = callAgent.call(appContext, participants, startCallOptions);
```

### <a name="place-a-1n-call-with-users-and-pstn"></a>Passer un appel 1:n avec des utilisateurs et un RTC
> [!WARNING]
> Les appels RTC ne sont pas disponibles actuellement. Pour passer un appel 1:n à un utilisateur et un numéro RTC, vous devez spécifier le numéro de téléphone de l’appelé.
Votre ressource Communication Services doit être configurée de manière à autoriser l’appel RTC :
```java
CommunicationUser acsUser1 = new CommunicationUser(<USER_ID>);
PhoneNumber acsUser2 = new PhoneNumber("<PHONE_NUMBER>");
CommunicationIdentifier participants[] = new CommunicationIdentifier[]{ acsUser1, acsUser2 };
StartCallOptions startCallOptions = new StartCallOptions();
Context appContext = this.getApplicationContext();
Call groupCall = callAgent.call(participants, startCallOptions);
```

### <a name="place-a-11-call-with-video-camera"></a>Passer un appel 1:1 avec une caméra vidéo
> [!WARNING]
> Actuellement, un seul flux vidéo local sortant est pris en charge. Pour passer un appel avec vidéo, vous devez énumérer les caméras locales à l’aide de l’API `deviceManager` `getCameraList`.
Une fois la caméra sélectionnée, utilisez-la pour créer une instance `LocalVideoStream` et la transmettre à `videoOptions` en tant qu’élément dans le tableau `localVideoStream` vers une méthode `call`.
Une fois l’appel connecté, il commence automatiquement à envoyer un flux vidéo aux autres participants à partir de la caméra sélectionnée.

> [!NOTE]
> Pour des raisons de confidentialité, la vidéo ne sera pas partagée lors de l’appel si elle n’est pas visionnée localement.
Pour plus d’informations, consultez [Aperçu de la caméra locale](#local-camera-preview).
```java
Context appContext = this.getApplicationContext();
VideoDeviceInfo desiredCamera = callClient.getDeviceManager().get().getCameraList().get(0);
LocalVideoStream currentVideoStream = new LocalVideoStream(desiredCamera, appContext);
VideoOptions videoOptions = new VideoOptions(currentVideoStream);

// Render a local preview of video so the user knows that their video is being shared
Renderer previewRenderer = new Renderer(currentVideoStream, appContext);
View uiView = previewRenderer.createView(new RenderingOptions(ScalingMode.Fit));
// Attach the uiView to a viewable location on the app at this point
layout.addView(uiView);

CommunicationUser[] participants = new CommunicationUser[]{ new CommunicationUser("<acs user id>") };
StartCallOptions startCallOptions = new StartCallOptions();
startCallOptions.setVideoOptions(videoOptions);
Call call = callAgent.call(context, participants, startCallOptions);
```

### <a name="join-a-group-call"></a>Rejoindre un appel de groupe
Pour démarrer un nouvel appel de groupe ou rejoindre un appel de groupe en cours, vous devez appeler la méthode « join » et transmettre un objet avec une propriété `groupId`. La valeur doit être un GUID.
```java
Context appContext = this.getApplicationContext();
GroupCallContext groupCallContext = new groupCallContext("<GUID>");
JoinCallOptions joinCallOptions = new JoinCallOptions();

call = callAgent.join(context, groupCallContext, joinCallOptions);
```

## <a name="push-notifications"></a>Notifications Push

### <a name="overview"></a>Vue d’ensemble
Les notifications Push mobiles sont les notifications contextuelles que vous recevez sur des appareils mobiles. Pour l’appel, nous utilisons des notifications Push VoIP (Voice over Internet Protocol). Nous effectuons l’inscription aux notifications push, nous traitons les notifications push, puis annulons l’inscription aux notifications push.

### <a name="prerequisites"></a>Prérequis

Un compte Firebase sur lequel Cloud Messaging (FCM) est activé et avec votre service Firebase Cloud Messaging connecté à une instance Azure Notification Hub. Pour plus d’informations, consultez [Notifications Communication Services](../../../concepts/notifications.md).
De plus, ce tutoriel suppose que vous utilisez Android Studio version 3.6 ou supérieure pour générer votre application.

Un jeu d’autorisations pour l’application Android est requis pour pouvoir recevoir des messages de notification de la part de Firebase Cloud Messaging. Dans votre fichier `AndroidManifest.xml`, ajoutez le jeu d’autorisations suivant juste après *<manifest ...>* ou sous la balise *</application>*

```XML
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
```

### <a name="register-for-push-notifications"></a>Inscription aux notifications Push

Pour s’inscrire aux notifications Push, l’application doit appeler `registerPushNotification()` sur une instance *CallAgent* avec un jeton d’inscription d’appareil.

Pour obtenir le jeton d’inscription de l’appareil, ajoutez la bibliothèque de clients Firebase au fichier *build.gradle* de votre module d’application en ajoutant les lignes suivantes à la section `dependencies` si elles n’y figurent pas encore :

```
    // Add the client library for Firebase Cloud Messaging
    implementation 'com.google.firebase:firebase-core:16.0.8'
    implementation 'com.google.firebase:firebase-messaging:20.2.4'
```

Dans le fichier *build.gradle* de votre niveau de projet, ajoutez l’élément suivant dans la section `dependencies` s’il n’y figure pas encore.

```
    classpath 'com.google.gms:google-services:4.3.3'
```

Ajoutez le plug-in suivant au début du fichier s’il n’y figure pas encore :

```
apply plugin: 'com.google.gms.google-services'
```

Sélectionnez *Synchroniser maintenant* dans la barre d’outils. Ajoutez l’extrait de code suivant pour obtenir le jeton d’inscription de l’appareil généré par la bibliothèque cliente de Firebase Cloud Messaging pour l’instance de l’application cliente. Veillez à ajouter les importations ci-dessous à l’en-tête de l’activité principale de l’instance. Celles-ci sont nécessaires pour que l’extrait de code récupère le jeton :

```
import com.google.android.gms.tasks.OnCompleteListener;
import com.google.android.gms.tasks.Task;
import com.google.firebase.iid.FirebaseInstanceId;
import com.google.firebase.iid.InstanceIdResult;
```

Ajoutez cet extrait de code pour récupérer le jeton :

```
        FirebaseInstanceId.getInstance().getInstanceId()
                .addOnCompleteListener(new OnCompleteListener<InstanceIdResult>() {
                    @Override
                    public void onComplete(@NonNull Task<InstanceIdResult> task) {
                        if (!task.isSuccessful()) {
                            Log.w("PushNotification", "getInstanceId failed", task.getException());
                            return;
                        }

                        // Get new Instance ID token
                        String deviceToken = task.getResult().getToken();
                        // Log
                        Log.d("PushNotification", "Device Registration token retrieved successfully");
                    }
                });
```
Inscrivez le jeton d’inscription d’appareil auprès de la bibliothèque de client Calling Services pour les notifications Push des appels entrants :

```java
String deviceRegistrationToken = "<Device Token from previous section>";
try {
    callAgent.registerPushNotification(deviceRegistrationToken).get();
}
catch(Exception e) {
    System.out.println("Something went wrong while registering for Incoming Calls Push Notifications.")
}
```

### <a name="push-notification-handling"></a>Gestion des notifications Push

Pour recevoir les notifications Push d’appel entrant, appelez *handlePushNotification()* sur une instance *CallAgent* avec une charge utile.

Pour obtenir la charge utile depuis Firebase Cloud Messaging, commencez par créer un nouveau Service (Fichier > Nouveau > Service > Service) qui étend la classe de la bibliothèque de client Firebase *FirebaseMessagingService* et remplacez la méthode `onMessageReceived`. Cette méthode est le gestionnaire d’événements appelé lorsque Firebase Cloud Messaging remet la notification Push à l’application.

```java
public class MyFirebaseMessagingService extends FirebaseMessagingService {
    private java.util.Map<String, String> pushNotificationMessageDataFromFCM;

    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        // Check if message contains a notification payload.
        if (remoteMessage.getNotification() != null) {
            Log.d("PushNotification", "Message Notification Body: " + remoteMessage.getNotification().getBody());
        }
        else {
            pushNotificationMessageDataFromFCM = remoteMessage.getData();
        }
    }
}
```
Ajoutez la définition de service suivante au fichier `AndroidManifest.xml`, dans la balise <application> :

```
        <service
            android:name=".MyFirebaseMessagingService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
```

- Une fois la charge utile récupérée, elle peut être transmise à la bibliothèque de client *Communication Services* pour être gérée en appelant la méthode *handlePushNotification* sur une instance *CallAgent*. Une instance `CallAgent` est créée en appelant la méthode `createCallAgent(...)` sur la classe `CallClient`.

```java
try {
    callAgent.handlePushNotification(pushNotificationMessageDataFromFCM).get();
}
catch(Exception e) {
    System.out.println("Something went wrong while handling the Incoming Calls Push Notifications.");
}
```

Lorsque la gestion du message de notification Push est réussie et que tous les gestionnaires d’événements sont correctement inscrits, l’application sonne.

### <a name="unregister-push-notifications"></a>Désinscription des notifications Push

Les applications peuvent annuler l’inscription aux notifications Push à tout moment. Appelez la méthode `unregisterPushNotification()` sur callAgent pour procéder à la désinscription.

```java
try {
    callAgent.unregisterPushNotifications().get();
}
catch(Exception e) {
    System.out.println("Something went wrong while un-registering for all Incoming Calls Push Notifications.")
}
```

## <a name="call-management"></a>Gestion des appels
Vous pouvez accéder à toutes les propriétés et effectuer différentes opérations lors d’un appel pour gérer les paramètres liés à la vidéo et à l’audio.

### <a name="call-properties"></a>Propriétés d’appel

Obtenir l’ID unique de cet appel :

```java
String callId = call.getCallId();
```

Pour en savoir plus sur les autres participants de l’appel, examinez la collection `remoteParticipant` sur l’instance `call` :

```java
List<RemoteParticipant> remoteParticipants = call.getRemoteParticipants();
```

L’identité de l’appelant si l’appel est entrant :

```java
CommunicationIdentifier callerId = call.getCallerId();
```

Obtenir l’état de l’appel : 

```java
CallState callState = call.getState();
```

Retourne une chaîne représentant l’état actuel d’un appel :
* « Aucun » - état initial de l’appel
* « Entrant » - indique que l’appel est entrant, il doit être accepté ou rejeté
* « Connexion en cours » - état de transition initial une fois l’appel passé ou accepté
* « Sonnerie » - pour un appel sortant, indique que l’appel sonne pour les participants distants ; il est « entrant » de leur côté
* « EarlyMedia » – indique un état dans lequel une annonce est lue avant la connexion de l’appel
* « Connecté » – l’appel est connecté
* « En attente » – l’appel est mis en attente, aucun média ne circule entre le point de terminaison local et les participants distants
* « Déconnexion » – état de transition avant que l’appel passe à l’état « Déconnecté »
* « Déconnecté » – état d’appel final


Pour connaître la raison de l’arrêt de l’appel, examinez la propriété `callEndReason`. Il contient le code/sous-code : 

```java
CallEndReason callEndReason = call.getCallEndReason();
int code = callEndReason.getCode();
int subCode = callEndReason.getSubCode();
```

Pour voir si l’appel en cours est un appel entrant, inspectez la propriété `isIncoming` :

```java
boolean isIncoming = call.getIsIncoming();
```

Pour voir si le microphone actuel est désactivé, inspectez la propriété `muted` :

```java
boolean muted = call.getIsMicrophoneMuted();
```

Pour inspecter les flux vidéo actifs, vérifiez la collection `localVideoStreams` :

```java
List<LocalVideoStream> localVideoStreams = call.getLocalVideoStreams();
```

### <a name="mute-and-unmute"></a>Activer et désactiver le son

Pour activer ou désactiver le son du point de terminaison local, vous pouvez utiliser les API asynchrones `mute` et `unmute` :

```java
call.mute().get();
call.unmute().get();
```

### <a name="start-and-stop-sending-local-video"></a>Démarrer et arrêter l’envoi de vidéo locale

Pour démarrer une vidéo, vous devez énumérer les caméras à l’aide de l’API `getCameraList` sur l’objet `deviceManager`. Cela crée une nouvelle instance de `LocalVideoStream` transmettant la caméra souhaitée, et en la transmettant dans l’API `startVideo` comme argument :

```java
VideoDeviceInfo desiredCamera = <get-video-device>;
Context appContext = this.getApplicationContext();
currentVideoStream = new LocalVideoStream(desiredCamera, appContext);
videoOptions = new VideoOptions(currentVideoStream);
Future startVideoFuture = call.startVideo(currentVideoStream);
startVideoFuture.get();
```

Une fois que vous avez commencé à envoyer une vidéo, une instance `LocalVideoStream` est ajoutée à la collection `localVideoStreams` sur l’instance d’appel.

```java
currentVideoStream == call.getLocalVideoStreams().get(0);
```

Pour arrêter la vidéo locale, transmettez l’instance `localVideoStream` disponible dans la collection `localVideoStreams` :

```java
call.stopVideo(localVideoStream).get();
```

Vous pouvez basculer vers une autre caméra pendant l’envoi d’une vidéo en appelant `switchSource` sur une instance `localVideoStream` :
```java
localVideoStream.switchSource(source).get();
```

## <a name="remote-participants-management"></a>Gestion des participants distants

Tous les participants distants sont représentés par le type `RemoteParticipant` et sont disponibles via la collection `remoteParticipants` sur une instance d’appel.

### <a name="list-participants-in-a-call"></a>Répertorier les participants à un appel
La collection `remoteParticipants` renvoie une liste de participants distants dans un appel donné :
```java
List<RemoteParticipant> remoteParticipants = call.getRemoteParticipants(); // [remoteParticipant, remoteParticipant....]
```

### <a name="remote-participant-properties"></a>Propriétés du participant distant
Tout participant distant donné dispose d’un ensemble de propriétés et de collections qui lui sont associées :

* Obtenez l’identificateur de ce participant distant.
L’identité est l’un des types d’identificateurs
```java
CommunicationIdentifier participantIdentity = remoteParticipant.getIdentifier();
```

* Obtenez l’état de ce participant distant.
```java
ParticipantState state = remoteParticipant.getState();
```
Il peut s’agir de l’un des états suivants :
* « Inactif » – état initial
* « Connexion » – état transitoire pendant que le participant se connecte à l’appel
* « Connecté » – le participant est connecté à l’appel
* « En attente » – le participant est en attente
* « EarlyMedia » – une annonce est lue avant que le participant soit connecté à l’appel
* « Déconnecté » – état final ; le participant est déconnecté de l’appel


* Pour savoir pourquoi un participant a quitté l’appel, inspectez la propriété `callEndReason` :
```java
CallEndReason callEndReason = remoteParticipant.getCallEndReason();
```

* Pour vérifier si le son de ce participant distant est activé ou non, inspectez la propriété `isMuted` :
```java
boolean isParticipantMuted = remoteParticipant.getIsMuted();
```

* Pour vérifier si ce participant parle ou non, inspectez la propriété `isSpeaking` :
```java
boolean isParticipantSpeaking = remoteParticipant.getIsSpeaking();
```

* Pour inspecter tous les flux vidéo qu’un participant donné envoie dans cet appel, vérifiez la collection `videoStreams` :
```java
List<RemoteVideoStream> videoStreams = remoteParticipant.getVideoStreams(); // [RemoteVideoStream, RemoteVideoStream, ...]
```


### <a name="add-a-participant-to-a-call"></a>Ajouter un participant à un appel

Pour ajouter un participant à un appel (un utilisateur ou un numéro de téléphone), vous pouvez appeler `addParticipant`. Cette opération renvoie de façon synchrone l’instance de participant distant.

```java
const acsUser = new CommunicationUser("<acs user id>");
const acsPhone = new PhoneNumber("<phone number>");
RemoteParticipant remoteParticipant1 = call.addParticipant(acsUser);
RemoteParticipant remoteParticipant2 = call.addParticipant(acsPhone);
```

### <a name="remove-participant-from-a-call"></a>Supprimer un participant d’un appel
Pour supprimer un participant d’un appel (un utilisateur ou un numéro de téléphone), vous pouvez appeler `removeParticipant`.
Le problème sera résolu de façon asynchrone une fois le participant supprimé de l’appel.
Le participant sera également supprimé de la collection `remoteParticipants`.
```java
RemoteParticipant remoteParticipant = call.getParticipants().get(0);
call.removeParticipant(acsUser).get();
call.removeParticipant(acsPhone).get();
```

## <a name="render-remote-participant-video-streams"></a>Afficher les flux vidéo des participants distants
Pour répertorier les flux vidéo et les flux de partage d’écran des participants distants, inspectez les collections `videoStreams` :
```java
RemoteParticipant remoteParticipant = call.getRemoteParticipants().get(0);
RemoteVideoStream remoteParticipantStream = remoteParticipant.getVideoStreams().get(0);
MediaStreamType streamType = remoteParticipantStream.getType(); // of type MediaStreamType.Video or MediaStreamType.ScreenSharing
```
 
Pour afficher le `RemoteVideoStream` d’un participant distant, vous devez vous abonner à un événement `OnVideoStreamsUpdated`.

Dans l’événement, la modification de la propriété `isAvailable` sur true indique que le participant distant envoie actuellement un flux. Une fois cette opération effectuée, créez une nouvelle instance d’un(e) `Renderer`, puis créez un nouveau/une nouvelle `RendererView` à l’aide de l’API `createView` asynchrone et joignez `view.target` n’importe où dans l’interface utilisateur de votre application.

Chaque fois que la disponibilité d’un flux distant change, vous pouvez choisir de détruire le convertisseur tout entier ou un `RendererView` spécifique, ou de les conserver, mais cela entraîne l’affichage d’une image vidéo vide.

```java
Renderer remoteVideoRenderer = new Renderer(remoteParticipantStream, appContext);
View uiView = remoteVideoRenderer.createView(new RenderingOptions(ScalingMode.Fit));
layout.addView(uiView);

remoteParticipant.addOnVideoStreamsUpdatedListener(e -> onRemoteParticipantVideoStreamsUpdated(p, e));

void onRemoteParticipantVideoStreamsUpdated(RemoteParticipant participant, RemoteVideoStreamsEvent args) {
    for(RemoteVideoStream stream : args.getAddedRemoteVideoStreams()) {
        if(stream.getIsAvailable()) {
            startRenderingVideo();
        } else {
            renderer.dispose();
        }
    }
}
```

### <a name="remote-video-stream-properties"></a>Propriétés du flux vidéo distant
Un flux vidéo distant possède plusieurs propriétés

* `Id` - ID d’un flux vidéo distant
```java
int id = remoteVideoStream.getId();
```

* `MediaStreamType` – peut être « Video » ou « ScreenSharing »
```java
MediaStreamType type = remoteVideoStream.getType();
```

* `isAvailable` – indique si le point de terminaison du participant distant envoie activement un flux
```java
boolean availability = remoteVideoStream.getIsAvailable();
```

### <a name="renderer-methods-and-properties"></a>Méthodes et propriétés du convertisseur
Objet du convertisseur suivant les API

* Créez une instance `RendererView` qui peut être jointe ultérieurement dans l’interface utilisateur de l’application pour afficher le flux vidéo distant.
```java
// Create a view for a video stream
renderer.createView()
```
* Supprimez le convertisseur et tous les `RendererView` associés à ce convertisseur. À appeler lorsque vous avez supprimé toutes les vues associées de l’interface utilisateur.
```java
renderer.dispose()
```

* `StreamSize` - taille (largeur/hauteur) d’un flux vidéo distant
```java
StreamSize renderStreamSize = remoteVideoStream.getSize();
int width = renderStreamSize.getWidth();
int height = renderStreamSize.getHeight();
```


### <a name="rendererview-methods-and-properties"></a>Méthodes et propriétés de RendererView
Lors de la création d’un `RendererView`, vous pouvez spécifier les propriétés `scalingMode` et `mirrored` qui s’appliquent à cette vue : Le mode de mise à l’échelle peut être « Stretch », « Crop » ou « Fit ». Si `mirrored` est défini sur `true`, le flux rendu est retourné verticalement.

```java
Renderer remoteVideoRenderer = new Renderer(remoteVideoStream, appContext);
RendererView rendererView = remoteVideoRenderer.createView(new RenderingOptions(ScalingMode.Fit));
```

Le RendererView créé peut alors être attaché à l’interface utilisateur de l’application à l’aide de l’extrait de code suivant :
```java
layout.addView(rendererView);
```

Vous pouvez modifier le mode de mise à l’échelle ultérieurement en appelant l’API `updateScalingMode` sur l’objet de RendererView avec l’un des arguments suivants : « ScalingMode.Stretch », « ScalingMode.Crop » ou « ScalingMode.Fit ».
```java
// Update the scale mode for this view.
rendererView.updateScalingMode(ScalingMode.Crop)
```


## <a name="device-management"></a>Gestion des appareils

`DeviceManager` vous permet d’énumérer les appareils locaux utilisables dans un appel pour transmettre vos flux audio/vidéo. Il vous permet également de demander à un utilisateur l’autorisation d’accéder à son microphone et à sa caméra à l’aide de l’API de navigateur natif.

Vous pouvez accéder au `deviceManager` en appelant la méthode `callClient.getDeviceManager()`.
> [!WARNING]
> Actuellement, un objet `callAgent` doit être préalablement instancié pour avoir accès à DeviceManager

```java
DeviceManager deviceManager = callClient.getDeviceManager().get();
```

### <a name="enumerate-local-devices"></a>Énumérer les appareils locaux

Pour accéder aux appareils locaux, vous pouvez utiliser les méthodes d’énumération sur le gestionnaire d’appareils. L’énumération est une action synchrone.

```java
//  Get a list of available video devices for use.
List<VideoDeviceInfo> localCameras = deviceManager.getCameraList(); // [VideoDeviceInfo, VideoDeviceInfo...]

// Get a list of available microphone devices for use.
List<AudioDeviceInfo> localMicrophones = deviceManager.getMicrophoneList(); // [AudioDeviceInfo, AudioDeviceInfo...]

// Get a list of available speaker devices for use.
List<AudioDeviceInfo> localSpeakers = deviceManager.getSpeakerList(); // [AudioDeviceInfo, AudioDeviceInfo...]
```

### <a name="set-default-microphonespeaker"></a>Définir le microphone/haut-parleur par défaut

Le gestionnaire d’appareils vous permet de définir un appareil par défaut qui sera utilisé lors du démarrage d’un appel.
Si les valeurs par défaut du client ne sont pas définies, Azure Communication Services revient aux valeurs par défaut du système d’exploitation.

```java

// Get the microphone device that is being used.
AudioDeviceInfo defaultMicrophone = deviceManager.getMicrophoneList().get(0);

// Set the microphone device to use.
deviceManager.setMicrophone(defaultMicrophone);

// Get the speaker device that is being used.
AudioDeviceInfo defaultSpeaker = deviceManager.getSpeakerList().get(0);

// Set the speaker device to use.
deviceManager.setSpeaker(defaultSpeaker);
```

### <a name="local-camera-preview"></a>Aperçu de la caméra locale

Vous pouvez utiliser `DeviceManager` et `Renderer` pour commencer à afficher le flux de votre caméra locale. Ce flux n’est pas envoyé à d’autres participants ; il s’agit d’un flux d’aperçu local. Cette action est asynchrone.

```java
VideoDeviceInfo videoDevice = <get-video-device>;
Context appContext = this.getApplicationContext();
currentVideoStream = new LocalVideoStream(videoDevice, appContext);
videoOptions = new VideoOptions(currentVideoStream);

Renderer previewRenderer = new Renderer(currentVideoStream, appContext);
View uiView = previewRenderer.createView(new RenderingOptions(ScalingMode.Fit));

// Attach the uiView to a viewable location on the app at this point
layout.addView(uiView);
```

## <a name="eventing-model"></a>Modèle d’événements
Vous pouvez vous abonner à la plupart des propriétés et collections pour être averti quand des valeurs changent.

### <a name="properties"></a>Propriétés
Pour vous abonner à des événements `property changed` :

```java
// subscribe
PropertyChangedListener callStateChangeListener = new PropertyChangedListener()
{
    @Override
    public void onPropertyChanged(PropertyChangedEvent args)
    {
        Log.d("The call state has changed.");
    }
}
call.addOnCallStateChangedListener(callStateChangeListener);

//unsubscribe
call.removeOnCallStateChangedListener(callStateChangeListener);
```

### <a name="collections"></a>Collections
Pour vous abonner à des événements `collection updated` :

```java
LocalVideoStreamsChangedListener localVideoStreamsChangedListener = new LocalVideoStreamsChangedListener()
{
    @Override
    public void onLocalVideoStreamsUpdated(LocalVideoStreamsEvent localVideoStreamsEventArgs) {
        Log.d(localVideoStreamsEventArgs.getAddedStreams().size());
        Log.d(localVideoStreamsEventArgs.getRemovedStreams().size());
    }
}
call.addOnLocalVideoStreamsChangedListener(localVideoStreamsChangedListener);
// To unsubscribe
call.removeOnLocalVideoStreamsChangedListener(localVideoStreamsChangedListener);
```