---
ms.openlocfilehash: 870b5b54c4a5afb39b43063bd00d3cb73e8ee0f9
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88682122"
---
Quand vous exécutez le graphe multimédia, les résultats du nœud processeur de détection de mouvement passe par le nœud récepteur IoT Hub pour atteindre le hub IoT. Les messages qui s’affichent dans la fenêtre **SORTIE** de Visual Studio Code contiennent une section `body` et une section `applicationProperties`. Pour plus d’informations, consultez [Créer et lire des messages IoT Hub](../../../../../iot-hub/iot-hub-devguide-messages-construct.md).

Dans les messages suivants, le module Live Video Analytics définit les propriétés de l’application et le contenu du corps.

### <a name="mediasessionestablished-event"></a>Événement MediaSessionEstablished

Quand un graphe multimédia est instancié, le nœud source RTSP tente de se connecter au serveur RTSP qui s’exécute sur le conteneur rtspsim-live555. Si la connexion réussit, l’événement suivant est affiché.

```
[IoTHubMonitor] [05:37:21 AM] Message received from [lva-sample-device/lvaEdge]:
{  
"body": {
"sdp": "SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 xxx.xxx.xxx.xxx\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets={SPS}\r\na=control:track1\r\n"  
},  
"applicationProperties": {  
    "dataVersion": "1.0",  
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",  
    "subject": "/graphInstances/Sample-Graph-1/sources/rtspSource",  
    "eventType": "Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished",  
    "eventTime": "2020-05-21T05:37:21.398Z",
    }  
}
```

Dans la sortie précédente : 

* Le message est un événement de diagnostic, `MediaSessionEstablished`. Il indique que le nœud source RTSP (l’objet) a établi une connexion au simulateur RTSP et a commencé à recevoir un flux en direct (simulé).
* Dans `applicationProperties`, `subject` fait référence au nœud présent dans la topologie de graphe à partir duquel le message a été généré. Dans ce cas, le message provient du nœud source RTSP.
* Dans `applicationProperties`, `eventType` indique que cet événement est un événement de diagnostic.
* La valeur `eventTime` correspond à l’heure à laquelle l’événement s’est produit.
* La section `body` contient des données relatives à l’événement de diagnostic. Dans ce cas, les données comprennent les détails du [protocole SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="recordingstarted-event"></a>Événement RecordingStarted

Quand un mouvement est détecté, le nœud processeur de porte de signal est activé et le nœud récepteur de fichiers dans le graphe multimédia commence à écrire un fichier MP4. Le nœud récepteur de fichiers envoie un événement opérationnel. Le `type` est défini sur `motion` pour indiquer qu’il s’agit d’un résultat du processeur de détection de mouvement. La valeur `eventTime` est l’heure UTC à laquelle le mouvement s’est produit. Pour plus d’informations sur ce processus, consultez la section [Vue d’ensemble](#overview) de ce guide de démarrage rapide.

Voici un exemple de ce message :

```
[IoTHubMonitor] [05:37:27 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "outputType": "filePath",
    "outputLocation": "/var/media/sampleFilesFromEVR-filesinkOutput-20200521T053726Z.mp4"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",  
    "subject": "/graphInstances/Sample-Graph-1/sinks/fileSink",
    "eventType": "Microsoft.Media.Graph.Operational.RecordingStarted",
    "eventTime": "2020-05-21T05:37:27.713Z",
    "dataVersion": "1.0"
  }
}
```

Dans le message précédent : 

* Dans `applicationProperties`, `subject` fait référence au nœud présent dans le graphe multimédia à partir duquel le message a été généré. Dans ce cas, le message provient du nœud récepteur de fichiers.
* Dans `applicationProperties`, `eventType` indique que cet événement est opérationnel.
* La valeur `eventTime` correspond à l’heure à laquelle l’événement s’est produit. Cette heure est 5 à 6 secondes après l’événement `MediaSessionEstablished` et après le début du flux vidéo. Cette heure correspond au repère des 5 à 6 secondes quand la [voiture a commencé à se déplacer](#review-the-sample-video) dans l’aire de stationnement.
* La section `body` contient des données relatives à l’événement opérationnel. Dans ce cas, les données comprennent `outputType` et `outputLocation`.
* La variable `outputType` indique que ces informations concernent le chemin du fichier.
* La valeur `outputLocation` est l’emplacement du fichier MP4 dans le module de périphérie.

### <a name="recordingstopped-and-recordingavailable-events"></a>Événements RecordingStopped et RecordingAvailable

Si vous examinez les propriétés du nœud processeur de porte de signal dans la [topologie de graphe](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/evr-motion-files/topology.json), vous remarquez que les durées d’activation sont définies sur 5 secondes. Ainsi, 5 secondes environ après la réception de l’événement `RecordingStarted`, vous obtenez :

* Un événement `RecordingStopped`, indiquant que l’enregistrement s’est arrêté.
* Un événement `RecordingAvailable`, indiquant que le fichier MP4 peut maintenant être visualisé.

Les deux événements sont généralement émis à quelques secondes d’intervalle.
