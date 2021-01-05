---
title: Supervision et journalisation - Azure
description: Cet article fournit une vue d’ensemble de la supervision et de la journalisation dans Live Video Analytics sur IoT Edge.
ms.topic: reference
ms.date: 04/27/2020
ms.openlocfilehash: ef00517fc61ac532bdd99c1e887dfd93d56a8c4f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89567552"
---
# <a name="monitoring-and-logging"></a>Surveillance et journalisation

Dans cet article, vous allez découvrir comment recevoir des événements du module Live Video Analytics sur IoT Edge à des fins de supervision à distance. 

Vous allez également découvrir comment contrôler les journaux générés par le module.

## <a name="taxonomy-of-events"></a>Taxonomie des événements

Live Video Analytics sur IoT Edge émet des événements ou des données de télémétrie selon la taxonomie suivante.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/telemetry-schema/taxonomy.png" alt-text="Taxonomie des événements&quot;:::

* Opérationnel : événements générés dans le cadre des actions effectuées par un utilisateur, ou lors de l’exécution d’un [graphe multimédia](media-graph-concept.md).
   
   * Volume : censé être faible (quelques occurrences par minute, voire même un débit inférieur).
   * Exemples :

      Enregistrement démarré (ci-dessous), enregistrement arrêté
      
      ```
      {
        &quot;body&quot;: {
          &quot;outputType&quot;: &quot;assetName&quot;,
          &quot;outputLocation&quot;: &quot;sampleAssetFromEVR-LVAEdge-20200512T233309Z&quot;
        },
        &quot;applicationProperties&quot;: {
          &quot;topic&quot;: &quot;/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>&quot;,
          &quot;subject&quot;: &quot;/graphInstances/Sample-Graph-2/sinks/assetSink&quot;,
          &quot;eventType&quot;: &quot;Microsoft.Media.Graph.Operational.RecordingStarted&quot;,
          &quot;eventTime&quot;: &quot;2020-05-12T23:33:10.392Z&quot;,
          &quot;dataVersion&quot;: &quot;1.0"
        }
      }
      ```
* Diagnostics : événements permettant de diagnostiquer des erreurs et/ou des problèmes de performances.

   * Volume : peut être élevé (plusieurs occurrences par minute).
   * Exemples :
   
      Informations RTSP [SDP](https://en.wikipedia.org/wiki/Session_Description_Protocol) (ci-dessous) ou intervalles dans le flux vidéo entrant.

      ```
      {
        "body": {
          "sdp": "SDP:\nv=0\r\no=- 1589326384077235 1 IN IP4 XXX.XX.XX.XXX\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/lots_015.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.04.12\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-73.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/lots_015.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=640028;sprop-parameter-sets=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\r\na=control:track1\r\n"
        },
        "applicationProperties": {
          "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
          "subject": "/graphInstances/Sample-Graph-2/sources/rtspSource",
          "eventType": "Microsoft.Media.Graph.Diagnostics.MediaSessionEstablished",
          "eventTime": "2020-05-12T23:33:04.077Z",
          "dataVersion": "1.0"
        }
      }
      ```
* Analytique : événements générés dans le cadre de l’analyse vidéo.

   * Volume : peut être élevé (plusieurs occurrences par minute ou fréquence plus élevée).
   * Exemples :
      
      Mouvement détecté (ci-dessous), résultat de l’inférence.

   ```      
   {
     "body": {
       "timestamp": 143039375044290,
       "inferences": [
         {
           "type": "motion",
           "motion": {
             "box": {
               "l": 0.48954,
               "t": 0.140741,
               "w": 0.075,
               "h": 0.058824
             }
           }
         }
       ]
     },
     "applicationProperties": {
       "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
       "subject": "/graphInstances/Sample-Graph-2/processors/md",
       "eventType": "Microsoft.Media.Graph.Analytics.Inference",
       "eventTime": "2020-05-12T23:33:09.381Z",
       "dataVersion": "1.0"
     }
   }
   ```

Les événements émis par le module sont envoyés au [hub IoT Edge](../../iot-edge/iot-edge-runtime.md#iot-edge-hub), et depuis ce dernier, ils peuvent être routés vers d’autres destinations. 

### <a name="timestamps-in-analytic-events"></a>Timestamps des événements analytiques

Comme indiqué ci-dessus, un timestamp est associé aux événements générés dans le cadre de l'analyse vidéo. Si vous avez [enregistré la vidéo en direct](video-recording-concept.md) dans le cadre de votre topologie graphique, cet horodatage vous permet de trouver où un événement particulier s’est produit dans la vidéo enregistrée. Les instructions suivantes expliquent comment mapper le timestamp d'un événement analytique avec la chronologie de la vidéo enregistrée dans un [élément multimédia Azure Media Service](terminology.md#asset).

Tout d'abord, extrayez la valeur `eventTime`. Utilisez cette valeur dans un [filtre d'intervalle de temps](playback-recordings-how-to.md#time-range-filters) pour extraire une portion appropriée de l'enregistrement. Par exemple, vous souhaitez peut-être extraire une vidéo qui commence 30 secondes avant `eventTime` et se termine 30 secondes après. Avec l'exemple ci-dessus, où `eventTime` correspond à 2020-05-12T23:33:09.381Z, une demande de manifeste HLS pour la fenêtre +/- 30s ressemblerait à ceci :

```
https://{hostname-here}/{locatorGUID}/content.ism/manifest(format=m3u8-aapl,startTime=2020-05-12T23:32:39Z,endTime=2020-05-12T23:33:39Z).m3u8
```

L'URL ci-dessus renverrait une [playlist principale](https://developer.apple.com/documentation/http_live_streaming/example_playlists_for_http_live_streaming) contenant les URL des playlists multimédias. La playlist multimédia contiendrait des entrées semblables aux suivantes :

```
...
#EXTINF:3.103011,no-desc
Fragments(video=143039375031270,format=m3u8-aapl)
...
```
Dans ce qui précède, l'entrée indique qu'un fragment vidéo est disponible et qu'il commence à une valeur de timestamp de `143039375031270`. La valeur `timestamp` de l'événement analytique utilise la même échelle de temps que la playlist multimédia, et peut être utilisée pour identifier le fragment vidéo pertinent et rechercher l'image qui convient.

Pour plus d'informations, vous pouvez lire l'un des nombreux [articles](https://www.bing.com/search?q=frame+accurate+seeking+in+HLS) disponibles sur la recherche précise d'images dans HLS.

## <a name="controlling-events"></a>Contrôle des événements

Vous pouvez utiliser les propriétés de jumeau de module suivantes, comme décrit dans [Schéma JSON de jumeau de module](module-twin-configuration-schema.md), pour contrôler les événements opérationnels et de diagnostic publiés par le module Live Video Analytics sur IoT Edge.

`diagnosticsEventsOutputName` : inclut et fournit (le cas échéant) la valeur de cette propriété, en vue d’obtenir des événements de diagnostic en provenance du module. Omettez-la ou laissez-la vide pour empêcher le module de publier des événements de diagnostic.
   
`operationalEventsOutputName` : inclut et fournit (le cas échéant) la valeur de cette propriété, en vue d’obtenir des événements opérationnels en provenance du module. Omettez-la ou laissez-la vide pour empêcher le module de publier des événements opérationnels.
   
Les événements analytiques sont générés par des nœuds comme le processeur de détection de mouvement ou le processeur d’extension HTTP, et le récepteur du hub IoT est utilisé pour les envoyer au hub IoT Edge. 

Vous pouvez contrôler le [routage de tous les événements ci-dessus](../../iot-edge/module-composition.md#declare-routes) par le biais d’une propriété voulue du jumeau de module $edgeHub (dans le manifeste de déploiement) :

```
 "$edgeHub": {
   "properties.desired": {
     "schemaVersion": "1.0",
     "routes": {
       "moduleToHub": "FROM /messages/modules/lvaEdge/outputs/* INTO $upstream"
     },
     "storeAndForwardConfiguration": {
       "timeToLiveSecs": 7200
     }
   }
 }
```

Dans l’exemple ci-dessus, lvaEdge correspond au nom du module Live Video Analytics sur IoT Edge, et la règle de routage suit le schéma défini dans les [déclarations de routes](../../iot-edge/module-composition.md#declare-routes).

> [!NOTE]
> Pour veiller à ce que les événements analytiques atteignent le hub IoT Edge, il doit y avoir un nœud récepteur du hub IoT en aval d’un nœud de processeur de détection de mouvement et/ou d’un nœud de processeur d’extension HTTP.

## <a name="event-schema"></a>Schéma d’événement

Les événements proviennent de l’appareil Edge et peuvent être utilisés sur cet appareil ou dans le cloud. Les événements générés par Live Video Analytics sur IoT Edge sont conformes au [modèle de messagerie en streaming](../../iot-hub/iot-hub-devguide-messages-construct.md) établi par Azure IoT Hub, avec des propriétés système, des propriétés d’application et un corps.

### <a name="summary"></a>Résumé

Chaque événement, lorsqu’il est observé par le biais du hub IoT, possède un ensemble de propriétés communes comme décrit ci-dessous.

|Propriété   |Type de propriété| Type de données   |Description|
|---|---|---|---|
|message-id |système |guid|  ID d’événement unique.|
|topic| applicationProperty |string|    Chemin Azure Resource Manager du compte Media Services.|
|subject|   applicationProperty |string|    Sous-chemin de l’entité émettrice de l’événement.|
|eventTime| applicationProperty|    string| Heure à laquelle l’événement a été généré.|
|eventType| applicationProperty |string|    Identificateur du type d’événement (voir ci-dessous).|
|body|body  |object|    Données d’événement particulières.|
|dataVersion    |applicationProperty|   string  |{Majeure}.{Mineure}|

### <a name="properties"></a>Propriétés

#### <a name="message-id"></a>message-id

Identificateur global unique (GUID) de l’événement

#### <a name="topic"></a>topic

Représente le compte Azure Media Services associé au graphe.

`/subscriptions/{subId}/resourceGroups/{rgName}/providers/Microsoft.Media/mediaServices/{accountName}`

#### <a name="subject"></a>subject

Entité qui émet l’événement :

`/graphInstances/{graphInstanceName}`<br/>
`/graphInstances/{graphInstanceName}/sources/{sourceName}`<br/>
`/graphInstances/{graphInstanceName}/processors/{processorName}`<br/>
`/graphInstances/{graphInstanceName}/sinks/{sinkName}`

La propriété subject permet aux événements génériques d’être mappés à leur module de génération. Par exemple, en cas de nom d’utilisateur ou mot de passe RTSP non valide, l’événement généré est `Microsoft.Media.Graph.Diagnostics.ProtocolError` sur le nœud `/graphInstances/myGraph/sources/myRtspSource`.

#### <a name="event-types"></a>Types d’événements

Les types d’événements sont attribués à un espace de noms selon le schéma suivant :

`Microsoft.Media.Graph.{EventClass}.{EventType}`

#### <a name="event-classes"></a>Classes d’événements

|Nom de la classe|Description|
|---|---|
|Analytics  |Événements générés dans le cadre de l’analyse du contenu.|
|Diagnostics    |Événements qui facilitent le diagnostic des problèmes et niveaux de performance.|
|En fonctionnement    |Événements générés dans le cadre des opérations des ressources.|

Les types d’événements sont propres à chaque classe d’événement.

Exemples :

* Microsoft.Media.Graph.Analytics.Inference
* Microsoft.Media.Graph.Diagnostics.AuthorizationError
* Microsoft.Media.Graph.Operational.GraphInstanceStarted

### <a name="event-time"></a>Heure de l’événement

L’heure de l’événement est décrite dans la chaîne ISO8601 et correspond à l’heure à laquelle l’événement s’est produit.

## <a name="logging"></a>Journalisation

Comme avec d’autres modules IoT Edge, vous pouvez également [examiner les journaux de conteneur](../../iot-edge/troubleshoot.md#check-container-logs-for-issues) sur l’appareil Edge. Les informations écrites dans les journaux peuvent être contrôlées par les [propriétés de jumeau de module](module-twin-configuration-schema.md) suivantes :

* logLevel

   * Les valeurs autorisées sont Détaillé, Informations, Avertissement, Erreur, Aucun.
   * La valeur par défaut est Informations : les journaux contiennent les messages d’erreur, d’avertissement et d’information.
   * Si vous définissez la valeur sur Avertissement, les journaux contiennent les messages d’erreur et d’avertissement.
   * Si vous définissez la valeur sur Erreur, les journaux ne contiennent que les messages d’erreur.
   * Si vous définissez la valeur sur Aucun, aucun journal n’est généré (cela n’est pas recommandé).
   * Vous devez utiliser la valeur Détaillé uniquement si vous avez besoin de partager des journaux avec le support Azure pour diagnostiquer un problème.
* logCategories

   * Liste séparée par des virgules d’un ou plusieurs des éléments suivants : Application, Events, MediaPipeline.
   * Valeur par défaut : Application, Events.
   * Application : il s’agit d’informations élémentaires provenant du module, comme des messages de démarrage du module, des erreurs d’environnement et des appels de méthode directs.
   * Events : il s’agit de tous les événements décrits précédemment dans cet article.
   * MediaPipeline : il s’agit de journaux détaillés qui peuvent donner des insights lors de la résolution des problèmes, comme des difficultés d’établissement d’une connexion avec une caméra compatible RTSP.
   
### <a name="generating-debug-logs"></a>Génération des journaux de débogage

Dans certains cas, vous aurez peut-être besoin de générer des journaux plus détaillés que ceux décrits ci-dessus, pour aider le support Azure à résoudre un problème. Deux étapes sont nécessaires pour cela.

Tout d’abord, vous [liez le stockage du module au stockage de l’appareil](../../iot-edge/how-to-access-host-storage-from-module.md#link-module-storage-to-device-storage) par le biais de createOptions. Si vous examinez un [modèle de manifeste de déploiement](https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp/blob/master/src/edge/deployment.template.json) à partir des démarrages rapides, vous voyez :

```
"createOptions": {
   …
   "Binds": [
     "/var/local/mediaservices/:/var/lib/azuremediaservices/"
   ]
 }
```

L’exemple ci-dessus permet au module Edge d’écrire des journaux sur le chemin de stockage (de l’appareil) « /var/local/mediaservices/ ». Si vous ajoutez la propriété voulue suivante au module :

`"debugLogsDirectory": "/var/lib/azuremediaservices/debuglogs/",`

Alors le module écrit les journaux de débogage dans un format binaire sur le chemin de stockage (de l’appareil) /var/local/mediaservices/debuglogs/, que vous pouvez partager avec le support Azure.

## <a name="faq"></a>Questions fréquentes (FAQ)

[FAQ](faq.md#monitoring-and-metrics)

## <a name="next-steps"></a>Étapes suivantes

[Enregistrement vidéo en continu](continuous-video-recording-tutorial.md)
