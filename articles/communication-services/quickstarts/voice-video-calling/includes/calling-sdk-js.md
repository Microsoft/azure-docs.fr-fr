---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 9/1/2020
ms.author: mikben
ms.openlocfilehash: ff9eca855269597477bc42a319c99c886576d92c
ms.sourcegitcommit: 0dcafc8436a0fe3ba12cb82384d6b69c9a6b9536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94482602"
---
## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Ressource Communication Services déployée. [Créez une ressource Communication Services](../../create-communication-resource.md).
- `User Access Token` pour activer le client d’appel. Pour en savoir plus, consultez [Comment obtenir un `User Access Token`](../../access-tokens.md)
- Facultatif : Suivez le démarrage rapide pour [prendre en main l’ajout de l’appel à votre application](../getting-started-with-calling.md)

## <a name="setting-up"></a>Configuration

### <a name="install-the-client-library"></a>Installer la bibliothèque de client

Utilisez la commande `npm install` pour installer les bibliothèques clientes Azure Communication Services d’appel et communes pour JavaScript.

```console
npm install @azure/communication-common --save

npm install @azure/communication-calling --save

```

## <a name="object-model"></a>Modèle objet

Les classes et les interfaces suivantes gèrent certaines des principales fonctionnalités de la bibliothèque de client Appel Azure Communication Services :

| Nom                             | Description                                                                                                                                 |
| ---------------------------------| ------------------------------------------------------------------------------------------------------------------------------------------- |
| CallClient                       | CallClient est le point d’entrée principal de la bibliothèque de client Appel.                                                                       |
| CallAgent                        | CallAgent sert à démarrer et à gérer les appels.                                                                                            |
| AzureCommunicationUserCredential | La classe AzureCommunicationUserCredential implémente l’interface CommunicationUserCredential qui est utilisée pour instancier le CallAgent. |


## <a name="initialize-the-callclient-create-callagent-and-access-devicemanager"></a>Initialiser CallClient, créer CallAgent et accéder à DeviceManager

Instanciez une nouvelle instance de `CallClient` . Vous pouvez le configurer à l’aide d’options personnalisées comme une instance d’enregistreur d’événements.
Une fois un `CallClient` instancié, vous pouvez créer une instance `CallAgent` en appelant la méthode `createCallAgent` sur l’instance `CallClient`. Cette opération renvoie un objet d’instance `CallAgent` de manière asynchrone.
La méthode `createCallAgent` prend un `CommunicationUserCredential` en tant qu’argument, qui accepte un [jeton d’accès utilisateur](../../access-tokens.md).
Pour accéder à `DeviceManager`, une instance callAgent doit d’abord être créée. Vous pouvez ensuite utiliser la méthode `getDeviceManager` sur l’instance `CallClient` pour obtenir DeviceManager.

```js
const userToken = '<user token>';
callClient = new CallClient(options);
const tokenCredential = new AzureCommunicationUserCredential(userToken);
const callAgent = await callClient.createCallAgent(tokenCredential);
const deviceManager = await callClient.getDeviceManager()
```

## <a name="place-an-outgoing-call"></a>Passer un appel sortant

Pour créer et démarrer un appel, vous devez utiliser l’une des API sur CallAgent et fournir un utilisateur que vous avez créé par le biais de la bibliothèque cliente d’administration de Communication Services.

La création et le démarrage de l’appel sont synchrones. L’instance d’appel vous permet de vous abonner à des événements d’appel.

## <a name="place-a-11-call-to-a-user-or-a-1n-call-with-users-and-pstn"></a>Passer un appel 1:1 à un utilisateur ou un appel 1:n avec plusieurs utilisateurs et RTC

Pour passer un appel à un autre utilisateur de Communication Services, appelez la méthode `call` sur `callAgent`, et transmettez le CommunicationUser que vous avez [créé avec la bibliothèque d’administration de Communication Services](../../access-tokens.md).

```js
const oneToOneCall = callAgent.call([CommunicationUser]);
```

### <a name="place-a-1n-call-with-users-and-pstn"></a>Passer un appel 1:n avec des utilisateurs et un RTC

Pour passer un appel 1 : n à un utilisateur et un numéro RTPC, vous devez spécifier un CommunicationUser et un numéro de téléphone pour les deux appelés.

Votre ressource Communication Services doit être configurée pour autoriser l’appel RTC.
```js

const userCallee = { communicationUserId: <ACS_USER_ID> }
const pstnCallee = { phoneNumber: <PHONE_NUMBER>};
const groupCall = callAgent.call([userCallee, pstnCallee], placeCallOptions);

```

### <a name="place-a-11-call-with-video-camera"></a>Passer un appel 1:1 avec une caméra vidéo
> [!WARNING]
> Il ne peut actuellement y avoir qu’un seul flux vidéo local sortant.
Pour passer un appel vidéo, vous devez énumérer des caméras locales à l’aide de l’API `getCameraList` de DeviceManager.
Une fois la caméra souhaitée sélectionnée, utilisez-la pour construire une instance `LocalVideoStream` et passer celle-ci dans `videoOptions` en tant qu’élément du tableau `localVideoStream` à la méthode `call`.
Une fois votre appel connecté, il commence automatiquement à envoyer un flux vidéo aux autres participants à partir de la caméra sélectionnée
```js
const deviceManager = await callClient.getDeviceManager();
const videoDeviceInfo = deviceManager.getCameraList()[0];
localVideoStream = new LocalVideoStream(videoDeviceInfo);
const placeCallOptions = {videoOptions: {localVideoStreams:[localVideoStream]}};
const call = callAgent.call(['acsUserId'], placeCallOptions);

```

### <a name="join-a-group-call"></a>Rejoindre un appel de groupe
Pour démarrer un nouvel appel de groupe ou rejoindre un appel de groupe, utilisez la méthode « join » et transmettez un objet avec une propriété `groupId`. La valeur doit être un GUID.
```js

const context = { groupId: <GUID>}
const call = callAgent.join(context);

```

## <a name="call-management"></a>Gestion des appels

Vous pouvez accéder à toutes les propriétés et effectuer différentes opérations lors d’un appel pour gérer les paramètres liés à la vidéo et à l’audio.

### <a name="call-properties"></a>Propriétés d’appel
* Obtenez l’ID unique (string) de cet appel.
```js

const callId: string = call.id;

```

* Pour en savoir plus sur les autres participants à l’appel, examinez la collection `remoteParticipant` sur l’instance `call`. Le tableau contient la liste des objets `RemoteParticipant`.
```js
const remoteParticipants = call.remoteParticipants;
```

* L’identité de l’appelant si l’appel est entrant. L’identité correspond à l’un des types `Identifier`
```js

const callerIdentity = call.callerIdentity;

```

* Obtenir l’état de l’appel.
```js

const callState = call.state;

```
Retourne une chaîne représentant l’état actuel d’un appel :
* « Aucun » – état initial de l’appel
* « Entrant » – indique qu’un appel est entrant ; il doit être accepté ou rejeté
* « Connexion en cours » – état de transition initial une fois l’appel passé ou accepté
* « Sonnerie » – pour un appel sortant, indique que l’appel sonne pour les participants distants, qu’il est « entrant » de leur côté
* « EarlyMedia » – indique un état dans lequel une annonce est lue avant la connexion de l’appel
* « Connecté » – l’appel est connecté
* « En attente » – l’appel est mis en attente, aucun média ne circule entre le point de terminaison local et les participants distants
* « Déconnexion » – état de transition avant que l’appel passe à l’état « Déconnecté »
* « Déconnecté » – état d’appel final.
   * Si la connexion réseau est perdue, l’état passe à « Déconnecté » au bout de 2 minutes environ.


* Pour connaître la raison pour laquelle un appel donné s’est terminé, inspectez la propriété `callEndReason`.
```js

const callEndReason = call.callEndReason;
// callEndReason.code (number) code associated with the reason
// callEndReason.subCode (number) subCode associated with the reason
```

* Pour savoir si l’appel en cours est un appel entrant, inspectez la propriété `isIncoming`. Elle retourne un `Boolean`.
```js
const isIncoming = call.isIncoming;
```

*  Pour vérifier si le microphone actuel est désactivé, inspectez la propriété `muted`. Elle retourne un `Boolean`.
```js

const muted = call.isMicrophoneMuted;

```

* Pour voir si le flux de partage d’écran est envoyé à partir d’un point de terminaison donné, vérifiez la propriété `isScreenSharingOn`. Elle retourne un `Boolean`.
```js

const isScreenSharingOn = call.isScreenSharingOn;

```

* Pour inspecter les flux vidéo actifs, vérifiez la collection `localVideoStreams`. Elle contient des objets `LocalVideoStream`.
```js

const localVideoStreams = call.localVideoStreams;

```

### <a name="mute-and-unmute"></a>Activer et désactiver le son

Pour activer ou désactiver le son du point de terminaison local, vous pouvez utiliser les API asynchrones `mute` et `unmute` :

```js

//mute local device 
await call.mute();

//unmute local device 
await call.unmute();

```

### <a name="start-and-stop-sending-local-video"></a>Démarrer et arrêter l’envoi de vidéo locale


Pour démarrer une vidéo, vous devez énumérer les caméras à l’aide de la méthode `getCameraList` sur l’objet `deviceManager`. Créez ensuite une instance de `LocalVideoStream` transmettant la caméra souhaitée dans la méthode `startVideo` en tant qu’argument :


```js
const localVideoStream = new LocalVideoStream(videoDeviceInfo);
await call.startVideo(localVideoStream);

```

Une fois que vous avez commencé à envoyer une vidéo, une instance `LocalVideoStream`est ajoutée à la collection `localVideoStreams` sur une instance d’appel.

```js

call.localVideoStreams[0] === localVideoStream;

```

Pour arrêter la vidéo locale, transmettez l’instance `localVideoStream` disponible dans la collection `localVideoStreams` :

```js

await call.stopVideo(localVideoStream);

```

Vous pouvez basculer vers une autre caméra pendant l’envoi d’une vidéo en appelant `switchSource` sur une instance `localVideoStream` :

```js
const source callClient.getDeviceManager().getCameraList()[1];
localVideoStream.switchSource(source);

```
### <a name="faq"></a>Questions fréquentes (FAQ)
 * Si la connectivité réseau est perdue, l’état de l’appel passe-t-il à « Déconnecté » ?
    * Oui, si la connexion réseau est perdue pendant plus de 2 minutes, l’appel passe à l’état Déconnecté et l’appel prend fin.

## <a name="remote-participants-management"></a>Gestion des participants distants

Tous les participants distants sont représentés par le type `RemoteParticipant`, et disponibles via la collection `remoteParticipants` sur une instance d’appel.

### <a name="list-participants-in-a-call"></a>Répertorier les participants à un appel
La collection `remoteParticipants` renvoie une liste de participants distants dans un appel donné :

```js

call.remoteParticipants; // [remoteParticipant, remoteParticipant....]

```

### <a name="remote-participant-properties"></a>Propriétés du participant distant
Un participant distant dispose d’un ensemble de propriétés et de collections qui lui sont associés

* Obtenez l’identificateur de ce participant distant.
L’identité est l’un des types d’identificateurs :
```js
const identifier = remoteParticipant.identifier;
//It can be one of:
// { communicationUserId: '<ACS_USER_ID'> } - object representing ACS User
// { phoneNumber: '<E.164>' } - object representing phone number in E.164 format
```

* Obtenez l’état de ce participant distant.
```js

const state = remoteParticipant.state;
```
Il peut s’agir de l’un des états suivants :
* « Inactif » – état initial
* « Connexion » – état transitoire pendant que le participant se connecte à l’appel
* « Connecté » – le participant est connecté à l’appel
* « En attente » – le participant est en attente
* « EarlyMedia » – une annonce est lue avant que le participant soit connecté à l’appel
* « Déconnecté » – état final ; le participant est déconnecté de l’appel.
   * Si un participant à distance perd sa connectivité réseau, son état passe à « Déconnecté » après environ 2 minutes.

Pour savoir pourquoi le participant a quitté l’appel, inspectez lz propriété `callEndReason` :
```js

const callEndReason = remoteParticipant.callEndReason;
// callEndReason.code (number) code associated with the reason
// callEndReason.subCode (number) subCode associated with the reason
```

* Pour vérifier si ce participant distant est muet ou non, inspectez la propriété `isMuted`. Elle retourne un `Boolean`.
```js
const isMuted = remoteParticipant.isMuted;
```

* Pour vérifier si ce participant distant parle ou non, inspectez la propriété `isSpeaking`. Elle retourne un `Boolean`.
```js

const isSpeaking = remoteParticipant.isSpeaking;

```

* Pour inspecter tous les flux vidéo qu’un participant donné envoie dans cet appel, vérifiez la collection `videoStreams`. Elle contient des objets `RemoteVideoStream`.
```js

const videoStreams = remoteParticipant.videoStreams; // [RemoteVideoStream, ...]

```


### <a name="add-a-participant-to-a-call"></a>Ajouter un participant à un appel

Pour ajouter un participant à un appel (un utilisateur ou un numéro de téléphone), vous pouvez appeler `addParticipant`.
Fournir l’un des types d’identificateurs.
Cette opération retourne de façon synchrone l’instance de participant distant.

```js
const userIdentifier = { communicationUserId: <ACS_USER_ID> };
const pstnIdentifier = { phoneNumber: <PHONE_NUMBER>}
const remoteParticipant = call.addParticipant(userIdentifier);
const remoteParticipant = call.addParticipant(pstnIdentifier);
```

### <a name="remove-participant-from-a-call"></a>Supprimer un participant d’un appel

Pour supprimer un participant d’un appel (un utilisateur ou un numéro de téléphone), vous pouvez appeler `removeParticipant`.
Vous devez passer l’un des types « Identificateur », ce qui sera résolu de façon asynchrone une fois le participant supprimé de l’appel.
Le participant sera également supprimé de la collection `remoteParticipants`.

```js
const userIdentifier = { communicationUserId: <ACS_USER_ID> };
const pstnIdentifier = { phoneNumber: <PHONE_NUMBER>}
await call.removeParticipant(userIdentifier);
await call.removeParticipant(pstnIdentifier);
```

## <a name="render-remote-participant-video-streams"></a>Afficher les flux vidéo des participants distants

Pour répertorier les flux vidéo et les flux de partage d’écran des participants distants, inspectez les collections `videoStreams` :

```js
const remoteVideoStream: RemoteVideoStream = call.remoteParticipants[0].videoStreams[0];
const streamType: MediaStreamType = remoteVideoStream.type;
```
 
Pour afficher un `RemoteVideoStream`, vous devez vous abonner à un événement `isAvailableChanged`.
Si la propriété `isAvailable` prend la valeur `true`, un participant distant envoie un flux.
Quand cela se produit, créez une instance de `Renderer`, puis une nouvelle instance `RendererView` à l’aide de la méthode `createView` asynchrone.  Vous pouvez ensuite attacher `view.target` à un élément d’interface utilisateur quelconque.
Chaque fois que la disponibilité d’un flux distant change, vous pouvez choisir de détruire le convertisseur tout entier ou un `RendererView` spécifique, ou de les conserver, mais cela entraîne l’affichage d’une image vidéo vide.

```js
let renderer: Renderer;
const displayVideo = () => {
    renderer = new Renderer(remoteParticipantStream);
    const view = await renderer.createView();
    htmlElement.appendChild(view.target);
}
remoteParticipantStream.on('availabilityChanged', async () => {
    if (remoteParticipantStream.isAvailable) {
        displayVideo();
    } else {
        renderer.dispose();
    }
});
if (remoteParticipantStream.isAvailable) {
    displayVideo();
}
```

### <a name="remote-video-stream-properties"></a>Propriétés du flux vidéo distant
Les flux vidéo distants ont les propriétés suivantes :

* `Id` - ID d’un flux vidéo distant
```js
const id: number = remoteVideoStream.id;
```

* `StreamSize` - taille (largeur/hauteur) d’un flux vidéo distant
```js
const size: {width: number; height: number} = remoteVideoStream.size;
```

* `MediaStreamType` – peut être « video » ou « ScreenSharing »
```js
const type: MediaStreamType = remoteVideoStream.type;
```
* `isAvailable` – indique si le point de terminaison du participant distant envoie activement un flux
```js
const type: boolean = remoteVideoStream.isAvailable;
```

### <a name="renderer-methods-and-properties"></a>Méthodes et propriétés de convertisseur

* Créez une instance `RendererView` qui peut être jointe ultérieurement dans l’interface utilisateur de l’application pour afficher le flux vidéo distant.
```js
renderer.createView()
```

* Supprimez le convertisseur et toutes les instances `RendererView` associées.
```js
renderer.dispose()
```


### <a name="rendererview-methods-and-properties"></a>Méthodes et propriétés de RendererView
Lorsque vous créez un `RendererView`, vous pouvez spécifier les propriétés `scalingMode` et `mirrored`.
Le mode de mise à l’échelle peut être « Stretch », « Crop » ou « Fit ». Si `Mirrored` est spécifié, le flux rendu est retourné verticalement.

```js
const rendererView: RendererView = renderer.createView({ scalingMode, mirrored });
```
Toute instance `RendererView` donnée a une propriété `target` qui représente la surface du rendu. Cette propriété doit être jointe à l’interface utilisateur de l’application :
```js
document.body.appendChild(rendererView.target);
```

Vous pouvez mettre à jour ultérieurement le mode de mise à l’échelle en appelant la méthode `updateScalingMode`.
```js
view.updateScalingMode('Crop')
```
### <a name="faq"></a>Questions fréquentes (FAQ)
* Si un participant à distance perd sa connexion réseau, son état passe-t-il à « Déconnecté » ?
    * Oui, si un participant à distance perd sa connexion réseau pendant plus de 2 minutes, son état passe à Déconnecté et il est supprimé de l’appel.
## <a name="device-management"></a>Gestion des appareils

`DeviceManager` vous permet d’énumérer les appareils locaux utilisables dans un appel pour transmettre vos flux audio/vidéo. Il vous permet également de demander à un utilisateur l’autorisation d’accéder à son microphone et à sa caméra à l’aide de l’API de navigateur natif.

Vous pouvez accéder au `deviceManager` en appelant la méthode `callClient.getDeviceManager()`.
> [!WARNING]
> Actuellement, un objet `callAgent` doit être préalablement instancié pour avoir accès à DeviceManager

```js

const deviceManager = await callClient.getDeviceManager();

```

### <a name="enumerate-local-devices"></a>Énumérer les appareils locaux

Pour accéder aux appareils locaux, vous pouvez utiliser les méthodes d’énumération sur le gestionnaire d’appareils. L’énumération est une action synchrone.

```js

//  Get a list of available video devices for use.
const localCameras = deviceManager.getCameraList(); // [VideoDeviceInfo, VideoDeviceInfo...]

// Get a list of available microphone devices for use.
const localMicrophones = deviceManager.getMicrophoneList(); // [AudioDeviceInfo, AudioDeviceInfo...]

// Get a list of available speaker devices for use.
const localSpeakers = deviceManager.getSpeakerList(); // [AudioDeviceInfo, AudioDeviceInfo...]

```

### <a name="set-default-microphonespeaker"></a>Définir le microphone/haut-parleur par défaut

Le gestionnaire d’appareils vous permet de définir un appareil par défaut qui sera utilisé lors du démarrage d’un appel.
Si les valeurs par défaut du client ne sont pas définies, Azure Communication Services revient aux valeurs par défaut du système d’exploitation.

```js

// Get the microphone device that is being used.
const defaultMicrophone = deviceManager.getMicrophone();

// Set the microphone device to use.
await deviceMicrophone.setMicrophone(AudioDeviceInfo);

// Get the speaker device that is being used.
const defaultSpeaker = deviceManager.getSpeaker();

// Set the speaker device to use.
await deviceManager.setSpeaker(AudioDeviceInfo);

```

### <a name="local-camera-preview"></a>Aperçu de la caméra locale

Vous pouvez utiliser `DeviceManager` et `Renderer` pour commencer à afficher le flux de votre caméra locale. Ce flux n’est pas envoyé à d’autres participants ; il s’agit d’un flux d’aperçu local. Cette action est asynchrone.

```js
const localVideoDevice = deviceManager().getCameraList()[0];
const localCameraStream = new LocalVideoStream(localVideoDevice);
const renderer = new Renderer(localCameraStream);
const view = await renderer.createView();
document.body.appendChild(view.target);

```

### <a name="request-permission-to-cameramicrophone"></a>Demander l’autorisation d’accès à une caméra ou un microphone

Demandez à un utilisateur d’accorder l’autorisation d’accéder à sa caméra ou à son microphone comme suit :

```js
const result = await deviceManager.askDevicePermission(audio: true, video: true);
```
Cela sera résolu de façon asynchrone avec un objet indiquant si les autorisations `audio` et `video` ont été accordées :
```js
console.log(result.audio);
console.log(result.video);
```

Vous pouvez inspecter l’état d’autorisation actuel d’un type donné en appelant `getPermissionState` :

```js

const result = deviceManager.getPermissionState('Microphone'); // for microphone permission state
const result = deviceManager.getPermissionState('Camera'); // for camera permission state

console.log(result); // 'Granted' | 'Denied' | 'Prompt' | 'Unknown';

```

## <a name="eventing-model"></a>Modèle d’événements

Vous pouvez vous abonner à la plupart des propriétés et collections pour être averti quand des valeurs changent.

### <a name="properties"></a>Propriétés
Pour vous abonner à des événements `property changed` :

```js

const eventHandler = () => {
    // check current value of a property, value is not passed to callback
    console.log(object.property);
};
object.on('propertyNameChanged',eventHandler);

// To unsubscribe:

object.off('propertyNameChanged',eventHandler);

```

### <a name="collections"></a>Collections
Pour vous abonner à des événements `collection updated` :

```js

const eventHandler = (e) => {
    // check added elements
    console.log(e.added);
    // check removed elements
    console.log(e.removed);
};
object.on('collectionNameUpdated',eventHandler);

// To unsubscribe:

object.off('collectionNameUpdated',eventHandler);

```
