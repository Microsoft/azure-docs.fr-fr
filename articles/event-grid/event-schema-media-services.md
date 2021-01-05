---
title: Azure Media Services comme source Event Grid
description: Décrit les propriétés qui sont fournies pour les événements Media Services avec Azure Event Grid
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: c1c5953cae7364131eefcec97d3375404c85e963
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015211"
---
# <a name="azure-media-services-as-an-event-grid-source"></a>Azure Media Services comme source Event Grid

Cet article fournit les schémas et les propriétés pour les événements Media Services.

## <a name="job-related-event-types"></a>Types d’événements associés au travail

Media Services émet les types d’événements associés au **travail** décrits ci-dessous. Il existe deux catégories d’événements associés au **travail** : « Supervision des changements d’état du travail » et « Supervision des changements d’état de la sortie du travail ». 

Vous pouvez vous inscrire à tous les événements en vous abonnant à l’événement JobStateChange. Vous pouvez aussi choisir de vous abonner uniquement à des événements spécifiques (par exemple, aux états finaux que sont JobErrored, JobFinished et JobCanceled).   

### <a name="monitoring-job-state-changes"></a>Supervision des changements d’état du travail

| Type d'événement | Description |
| ---------- | ----------- |
| Microsoft.Media.JobStateChange| Obtient un événement pour tous les changements d’état du travail. |
| Microsoft.Media.JobScheduled| Obtient un événement quand un travail passe à l’état Planifié. |
| Microsoft.Media.JobProcessing| Obtient un événement quand un travail passe à l’état En cours de traitement. |
| Microsoft.Media.JobCanceling| Obtient un événement quand un travail passe à l’état Annulation. |
| Microsoft.Media.JobFinished| Obtient un événement quand un travail passe à l’état Terminé. Il s’agit d’un état final qui inclut les sorties du travail.|
| Microsoft.Media.JobCanceled| Obtient un événement quand un travail passe à l’état Annulé. Il s’agit d’un état final qui inclut les sorties du travail.|
| Microsoft.Media.JobErrored| Obtient un événement quand un travail passe à l’état Erreur. Il s’agit d’un état final qui inclut les sorties du travail.|

Consultez les [exemples de schémas](#event-schema-examples) qui suivent.

### <a name="monitoring-job-output-state-changes"></a>Supervision des changements d’état de sortie du travail

Un travail peut contenir plusieurs sorties de travail (si vous avez configuré la transformation pour avoir plusieurs sorties de travail). Si vous souhaitez suivre les détails de la sortie individuelle d’un travail, écoutez l’événement de modification de la sortie d’un travail.

Chaque **travail** sera à un niveau supérieur à **JobOutput**, ce qui entraîne le déclenchement des événements de sortie de travail dans un travail correspondant. 

Les messages d’erreur dans `JobFinished`, `JobCanceled`, `JobError` sortent les résultats agrégés pour chaque sortie de travail, lorsque toutes sont terminées. En revanche, les événements de sortie du travail se déclenchent à mesure que chaque tâche se termine. Par exemple, si vous avez une sortie d’encodage, suivie d’une sortie Video Analytics, vous obtiendriez deux événements déclenchant des événements de sortie de travail avant que l’événement JobFinished final ne se déclenche avec les données agrégées.

| Type d'événement | Description |
| ---------- | ----------- |
| Microsoft.Media.JobOutputStateChange| Obtient un événement pour tous les changements d’état de sortie du travail. |
| Microsoft.Media.JobOutputScheduled| Obtient un événement quand la sortie du travail passe à l’état Planifié. |
| Microsoft.Media.JobOutputProcessing| Obtient un événement quand la sortie du travail passe à l’état En cours de traitement. |
| Microsoft.Media.JobOutputCanceling| Obtient un événement quand la sortie du travail passe à l’état Annulation.|
| Microsoft.Media.JobOutputFinished| Obtient un événement quand la sortie du travail passe à l’état Terminé.|
| Microsoft.Media.JobOutputCanceled| Obtient un événement quand la sortie du travail passe à l’état Annulé.|
| Microsoft.Media.JobOutputErrored| Obtient un événement quand la sortie du travail passe à l’état Erreur.|

Consultez les [exemples de schémas](#event-schema-examples) qui suivent.

### <a name="monitoring-job-output-progress"></a>Surveillance de la progression de la sortie des travaux

| Type d'événement | Description |
| ---------- | ----------- |
| Microsoft.Media.JobOutputProgress| Cet événement reflète la progression du traitement des travaux de 0 à 100 %. Le service tente d’envoyer un événement s’il y a eu une augmentation de 5 % ou plus de la valeur de progression ou si plus de 30 secondes se sont écoulées depuis le dernier événement (pulsation). Il n’est pas garanti que la valeur de progression commence à 0 % ni qu’elle atteigne 100 %, ni encore qu’elle augmente à un taux constant dans le temps. Cet événement ne doit pas être utilisé pour déterminer que le traitement a été effectué – vous devez utiliser à la place les événements de changement d’état.|

Consultez les [exemples de schémas](#event-schema-examples) qui suivent.

## <a name="live-event-types"></a>Types d’événements en direct

Media Services émet aussi les types d’événements **en direct** décrits ci-dessous. Il existe deux catégories d’événements **en direct** : les événements de flux de données et les événements de piste. 

### <a name="stream-level-events"></a>Événements de flux de données

Les événements de flux sont déclenchés par le flux de données ou la connexion. Chaque événement possède un paramètre `StreamId` qui identifie la connexion ou le flux. Chaque flux de données ou connexion dispose d’une ou de plusieurs pistes de différents types. Par exemple, une même connexion provenant d’un encodeur peut avoir une seule piste audio et quatre pistes vidéo. Les types d’événements de flux de données sont :

| Type d'événement | Description |
| ---------- | ----------- |
| Microsoft.Media.LiveEventConnectionRejected | La tentative de connexion de l’encodeur est rejetée. |
| Microsoft.Media.LiveEventEncoderConnected | L’encodeur établit une connexion avec l’événement en direct. |
| Microsoft.Media.LiveEventEncoderDisconnected | L’encodeur se déconnecte. |

Consultez les [exemples de schémas](#event-schema-examples) qui suivent.

### <a name="track-level-events"></a>Événements de piste

Les événements de piste sont déclenchés en fonction de la piste. 

> [!NOTE]
> Tous les événements de niveau suivi sont déclenchés après qu’un encodeur live est connecté.

Les types d’événements de niveau suivi sont :

| Type d'événement | Description |
| ---------- | ----------- |
| Microsoft.Media.LiveEventIncomingDataChunkDropped | Le serveur multimédia supprime le bloc de données, car il est trop tard ou les timestamps se chevauchent (le timestamp du nouveau bloc de données est inférieur à l’heure de fin du bloc de données précédent). |
| Microsoft.Media.LiveEventIncomingStreamReceived | Le serveur multimédia reçoit le premier bloc de données pour chaque piste dans le flux ou la connexion. |
| Microsoft.Media.LiveEventIncomingStreamsOutOfSync | Le serveur multimédia détecte des flux audio et vidéo qui ne sont pas synchronisés. Ce type d’événement doit être utilisé comme un avertissement, car l’expérience utilisateur peut ne pas être affectée. |
| Microsoft.Media.LiveEventIncomingVideoStreamsOutOfSync | Le serveur multimédia détecte que l’un des deux flux vidéo provenant de l’encodeur externe n’est pas synchronisé. Ce type d’événement doit être utilisé comme un avertissement, car l’expérience utilisateur peut ne pas être affectée. |
| Microsoft.Media.LiveEventIngestHeartbeat | Publié toutes les 20 secondes pour chaque piste pendant l’événement en direct. Fournit un résumé de l’intégrité d’ingestion.<br/><br/>Une l’encodeur a été initialement connecté, l’événement de pulsation continue d’émettre toutes les 20 secondes, que l’encodeur soit encore connecté ou non. |
| Microsoft.Media.LiveEventTrackDiscontinuityDetected | Le serveur multimédia détecte une discontinuité dans la piste entrante. |

Consultez les [exemples de schémas](#event-schema-examples) qui suivent.

## <a name="event-schema-examples"></a>Exemples de schéma d’événement

### <a name="jobstatechange"></a>JobStateChange

L’exemple suivant montre le schéma d’un événement **JobStateChange** : 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
    "eventType": "Microsoft.Media.JobStateChange",
    "eventTime": "2018-04-20T21:26:13.8978772",
    "id": "b9d38923-9210-4c2b-958f-0054467d4dd7",
    "data": {
      "previousState": "Processing",
      "state": "Finished"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| previousState | string | L’état du travail avant l’événement. |
| state | string | Le nouvel état de la tâche notifié dans cet événement. Par exemple, « Planifié : le travail est prêt à démarrer » ou « Terminé : le travail est terminé ».|

Où l’état du travail peut prendre l’une des valeurs suivantes : *En file d’attente*, *Planifié*, *Traitement*, *Terminé*, *Erreur*, *Annulé*, *Annulation en cours*

> [!NOTE]
> *En file d’attente* est présent uniquement dans la propriété **previousState**, mais pas dans la propriété **state**.

### <a name="jobscheduled-jobprocessing-jobcanceling"></a>JobScheduled, JobProcessing, JobCanceling

Pour chaque changement d’état de travail non final (par exemple, JobScheduled, JobProcessing, JobCanceling), l’exemple de schéma se présente comme suit :

```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "eventType": "Microsoft.Media.JobProcessing",
  "eventTime": "2018-10-12T16:12:18.0839935",
  "id": "a0a6efc8-f647-4fc2-be73-861fa25ba2db",
  "data": {
    "previousState": "Scheduled",
    "state": "Processing",
    "correlationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

### <a name="jobfinished-jobcanceled-joberrored"></a>JobFinished, JobCanceled, JobErrored

Pour chaque changement d’état de travail final (par exemple, JobFinished, JobCanceled, JobErrored), l’exemple de schéma se présente comme suit :

```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "eventType": "Microsoft.Media.JobFinished",
  "eventTime": "2018-10-12T16:25:56.4115495",
  "id": "9e07e83a-dd6e-466b-a62f-27521b216f2a",
  "data": {
    "outputs": [
      {
        "@odata.type": "#Microsoft.Media.JobOutputAsset",
        "assetName": "output-7640689F",
        "error": null,
        "label": "VideoAnalyzerPreset_0",
        "progress": 100,
        "state": "Finished"
      }
    ],
    "previousState": "Processing",
    "state": "Finished",
    "correlationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| outputs | Array | Obtient les sorties du travail.|

### <a name="joboutputstatechange"></a>JobOutputStateChange

L’exemple suivant montre le schéma de l’événement **JobOutputStateChange** :

```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "eventType": "Microsoft.Media.JobOutputStateChange",
  "eventTime": "2018-10-12T16:25:56.0242854",
  "id": "dde85f46-b459-4775-b5c7-befe8e32cf90",
  "data": {
    "previousState": "Processing",
    "output": {
      "@odata.type": "#Microsoft.Media.JobOutputAsset",
      "assetName": "output-7640689F",
      "error": null,
      "label": "VideoAnalyzerPreset_0",
      "progress": 100,
      "state": "Finished"
    },
    "jobCorrelationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

### <a name="joboutputscheduled-joboutputprocessing-joboutputfinished-joboutputcanceling-joboutputcanceled-joboutputerrored"></a>JobOutputScheduled, JobOutputProcessing, JobOutputFinished, JobOutputCanceling, JobOutputCanceled, JobOutputErrored

Pour chaque changement d’état de sortie du travail (JobOutput), l’exemple de schéma se présente comme suit :

```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "eventType": "Microsoft.Media.JobOutputProcessing",
  "eventTime": "2018-10-12T16:12:18.0061141",
  "id": "f1fd5338-1b6c-4e31-83c9-cd7c88d2aedb",
  "data": {
    "previousState": "Scheduled",
    "output": {
      "@odata.type": "#Microsoft.Media.JobOutputAsset",
      "assetName": "output-7640689F",
      "error": null,
      "label": "VideoAnalyzerPreset_0",
      "progress": 0,
      "state": "Processing"
    },
    "jobCorrelationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```
### <a name="joboutputprogress"></a>JobOutputProgress

L’exemple de schéma ressemble à ce qui suit :

 ```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/belohGroup/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/job-5AB6DE32",
  "eventType": "Microsoft.Media.JobOutputProgress",
  "eventTime": "2018-12-10T18:20:12.1514867",
  "id": "00000000-0000-0000-0000-000000000000",
  "data": {
    "jobCorrelationData": {
      "TestKey1": "TestValue1",
      "testKey2": "testValue2"
    },
    "label": "VideoAnalyzerPreset_0",
    "progress": 86
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

### <a name="liveeventconnectionrejected"></a>LiveEventConnectionRejected

L’exemple suivant montre le schéma d’un événement **LiveEventConnectionRejected** : 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/MyLiveEvent1",
    "eventType": "Microsoft.Media.LiveEventConnectionRejected",
    "eventTime": "2018-01-16T01:57:26.005121Z",
    "id": "b303db59-d5c1-47eb-927a-3650875fded1",
    "data": { 
      "streamId":"Mystream1",
      "ingestUrl": "http://abc.ingest.isml",
      "encoderIp": "118.238.251.xxx",
      "encoderPort": 52859,
      "resultCode": "MPE_INGEST_CODEC_NOT_SUPPORTED"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| streamId | string | Identificateur du flux de données ou de la connexion. L’encodeur ou le client est chargé d’ajouter cet ID dans l’URL d’ingestion. |  
| ingestUrl | string | URL d’ingestion fournie par l’événement en direct. |  
| encoderIp | string | Adresse IP de l’encodeur. |
| encoderPort | string | Port de l’encodeur dont provient ce flux. |
| resultCode | string | Motif de rejet de la connexion. Les codes de résultat sont répertoriés dans le tableau suivant. |

Vous pouvez trouver les codes de résultat d’erreur dans [Codes d’erreur d’événement en direct](../media-services/latest/live-event-error-codes.md).

### <a name="liveeventencoderconnected"></a>LiveEventEncoderConnected

L’exemple suivant montre le schéma d’un événement **LiveEventEncoderConnected** : 

```json
[
  { 
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventEncoderConnected",
    "eventTime": "2018-08-07T23:08:09.1710643",
    "id": "<id>",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml",
      "streamId": "15864-stream0",
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| streamId | string | Identificateur du flux de données ou de la connexion. L’encodeur ou le client est chargé d’ajouter cet ID dans l’URL d’ingestion. |
| ingestUrl | string | URL d’ingestion fournie par l’événement en direct. |
| encoderIp | string | Adresse IP de l’encodeur. |
| encoderPort | string | Port de l’encodeur dont provient ce flux. |

### <a name="liveeventencoderdisconnected"></a>LiveEventEncoderDisconnected

L’exemple suivant montre le schéma d’un événement **LiveEventEncoderDisconnected** : 

```json
[
  { 
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventEncoderDisconnected",
    "eventTime": "2018-08-07T23:08:09.1710872",
    "id": "<id>",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml",
      "streamId": "15864-stream0",
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485",
      "resultCode": "S_OK"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| streamId | string | Identificateur du flux de données ou de la connexion. L’encodeur ou le client est chargé d’ajouter cet ID dans l’URL d’ingestion. |  
| ingestUrl | string | URL d’ingestion fournie par l’événement en direct. |  
| encoderIp | string | Adresse IP de l’encodeur. |
| encoderPort | string | Port de l’encodeur dont provient ce flux. |
| resultCode | string | Motif de déconnexion de l’encodeur. Il peut s’agir d’une déconnexion normale ou due à une erreur. Les codes de résultat sont répertoriés dans le tableau suivant. |

Vous pouvez trouver les codes de résultat d’erreur dans [Codes d’erreur d’événement en direct](../media-services/latest/live-event-error-codes.md).

Les codes de résultat de déconnexion normale sont :

| Code de résultat | Description |
| ----------- | ----------- |
| S_OK | Encodeur déconnecté avec succès. |
| MPE_CLIENT_TERMINATED_SESSION | Encodeur déconnecté (RTMP). |
| MPE_CLIENT_DISCONNECTED | Encodeur déconnecté (FMP4). |
| MPI_REST_API_CHANNEL_RESET | La commande de réinitialisation du canal est reçue. |
| MPI_REST_API_CHANNEL_STOP | La commande d’arrêt du canal est reçue. |
| MPI_REST_API_CHANNEL_STOP | Canal en cours de maintenance. |
| MPI_STREAM_HIT_EOF | Le flux EOF est envoyé par l’encodeur. |

### <a name="liveeventincomingdatachunkdropped"></a>LiveEventIncomingDataChunkDropped

L’exemple suivant montre le schéma d’un événement **LiveEventIncomingDataChunkDropped** : 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/MyLiveEvent1",
    "eventType": "Microsoft.Media.LiveEventIncomingDataChunkDropped",
    "eventTime": "2018-01-16T01:57:26.005121Z",
    "id": "03da9c10-fde7-48e1-80d8-49936f2c3e7d",
    "data": { 
      "trackType": "Video",
      "trackName": "Video",
      "bitrate": 300000,
      "timestamp": 36656620000,
      "timescale": 10000000,
      "resultCode": "FragmentDrop_OverlapTimestamp"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| trackType | string | Type de la piste (Audio/Vidéo). |
| trackName | string | Nom de la piste. |
| bitrate | entier | Débit binaire de la piste. |
| timestamp | string | Timestamp du bloc de données supprimé. |
| timescale | string | Échelle de temps du timestamp. |
| resultCode | string | Motif de suppression du bloc de données. **FragmentDrop_OverlapTimestamp** ou **FragmentDrop_NonIncreasingTimestamp**. |

### <a name="liveeventincomingstreamreceived"></a>LiveEventIncomingStreamReceived

L’exemple suivant montre le schéma d’un événement **LiveEventIncomingStreamReceived** : 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventIncomingStreamReceived",
    "eventTime": "2018-08-07T23:08:10.5069288Z",
    "id": "7f939a08-320c-47e7-8250-43dcfc04ab4d",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml/Streams(15864-stream0)15864-stream0",
      "trackType": "video",
      "trackName": "video",
      "bitrate": 2962000,
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485",
      "timestamp": "15336831655032322",
      "duration": "20000000",
      "timescale": "10000000"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| trackType | string | Type de la piste (Audio/Vidéo). |
| trackName | string | Nom de la piste (fourni par l’encodeur ou, dans le cas du RTMP, généré par le serveur selon le format *TypePiste_DébitBinaire*). |
| bitrate | entier | Débit binaire de la piste. |
| ingestUrl | string | URL d’ingestion fournie par l’événement en direct. |
| encoderIp | string  | Adresse IP de l’encodeur. |
| encoderPort | string | Port de l’encodeur dont provient ce flux. |
| timestamp | string | Premier timestamp du bloc de données reçu. |
| timescale | string | Échelle de temps dans laquelle le timestamp est représenté. |

### <a name="liveeventincomingstreamsoutofsync"></a>LiveEventIncomingStreamsOutOfSync

L’exemple suivant montre le schéma d’un événement **LiveEventIncomingStreamsOutOfSync** : 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventIncomingStreamsOutOfSync",
    "eventTime": "2018-08-10T02:26:20.6269183Z",
    "id": "b9d38923-9210-4c2b-958f-0054467d4dd7",
    "data": {
      "minLastTimestamp": "319996",
      "typeOfStreamWithMinLastTimestamp": "Audio",
      "maxLastTimestamp": "366000",
      "typeOfStreamWithMaxLastTimestamp": "Video",
      "timescaleOfMinLastTimestamp": "10000000", 
      "timescaleOfMaxLastTimestamp": "10000000"       
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| minLastTimestamp | string | Minimum des derniers timestamps parmi toutes les pistes (audio ou vidéo). |
| typeOfTrackWithMinLastTimestamp | string | Type de la piste (audio ou vidéo) avec dernier timestamp minimum. |
| maxLastTimestamp | string | Maximum de tous les timestamps parmi toutes les pistes (audio ou vidéo). |
| typeOfTrackWithMaxLastTimestamp | string | Type de la piste (audio ou vidéo) avec le dernier timestamp maximum. |
| timescaleOfMinLastTimestamp| string | Obtient l’échelle de temps dans laquelle « MinLastTimestamp » est représenté.|
| timescaleOfMaxLastTimestamp| string | Obtient l’échelle de temps dans laquelle « MaxLastTimestamp » est représenté.|

### <a name="liveeventincomingvideostreamsoutofsync"></a>LiveEventIncomingVideoStreamsOutOfSync

L’exemple suivant montre le schéma d’un événement **LiveEventIncomingVideoStreamsOutOfSync** : 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/LiveEvent1",
    "eventType": "Microsoft.Media.LiveEventIncomingVideoStreamsOutOfSync",
    "eventTime": "2018-01-16T01:57:26.005121Z",
    "id": "6dd4d862-d442-40a0-b9f3-fc14bcf6d750",
    "data": {
      "firstTimestamp": "2162058216",
      "firstDuration": "2000",
      "secondTimestamp": "2162057216",
      "secondDuration": "2000",
      "timescale": "10000000"      
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| firstTimestamp | string | Timestamp reçu pour l’une des pistes/l’un des niveaux de qualité de type vidéo. |
| firstDuration | string | Durée du bloc de données avec le premier timestamp. |
| secondTimestamp | string  | Timestamp reçu pour une autre piste/un autre niveau de qualité de type vidéo. |
| secondDuration | string | Durée du bloc de données avec un deuxième timestamp. |
| timescale | string | Échelle de temps des timestamps et de la durée.|

### <a name="liveeventingestheartbeat"></a>LiveEventIngestHeartbeat

L’exemple suivant montre le schéma d’un événement **LiveEventIngestHeartbeat** : 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventIngestHeartbeat",
    "eventTime": "2018-08-07T23:17:57.4610506",
    "id": "7f450938-491f-41e1-b06f-c6cd3965d786",
    "data": {
      "trackType": "audio",
      "trackName": "audio",
      "bitrate": 160000,
      "incomingBitrate": 155903,
      "lastTimestamp": "15336837535253637",
      "timescale": "10000000",
      "overlapCount": 0,
      "discontinuityCount": 0,
      "nonincreasingCount": 0,
      "unexpectedBitrate": false,
      "state": "Running",
      "healthy": true
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| trackType | string | Type de la piste (Audio/Vidéo). |
| trackName | string | Nom de la piste (fourni par l’encodeur ou, dans le cas du RTMP, généré par le serveur selon le format *TypePiste_DébitBinaire*). |
| bitrate | entier | Débit binaire de la piste. |
| incomingBitrate | entier | Débit binaire calculé et basé sur des blocs de données provenant de l’encodeur. |
| lastTimestamp | string | Timestamp le plus récent reçu pour une piste dans les 20 dernières secondes. |
| timescale | string | Échelle de temps dans laquelle les timestamps sont exprimés. |
| overlapCount | entier | Nombre de blocs de données avec des timestamps qui se chevauchent au cours des 20 dernières secondes. |
| discontinuityCount | entier | Nombre de discontinuités observées dans les dernières 20 secondes. |
| nonIncreasingCount | entier | Nombre de blocs de données présentant des timestamps passés dans les dernières 20 secondes. |
| unexpectedBitrate | bool | Indique si les débits binaires attendus et réels diffèrent de plus de la valeur maximale autorisée au cours des 20 dernières secondes. La valeur est true si et seulement si incomingBitrate >= 2* bitrate OR incomingBitrate <= bitrate/2 OR IncomingBitrate = 0. |
| state | string | État de l’événement en direct. |
| healthy | bool | Indique si l’ingestion est intègre en fonction des nombres et des indicateurs. Healthy a la valeur true si overlapCount = 0 && discontinuityCount = 0 && nonIncreasingCount = 0 && unexpectedBitrate = false. |

### <a name="liveeventtrackdiscontinuitydetected"></a>LiveEventTrackDiscontinuityDetected

L’exemple suivant montre le schéma d’un événement **LiveEventTrackDiscontinuityDetected** : 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventTrackDiscontinuityDetected",
    "eventTime": "2018-08-07T23:18:06.1270405Z",
    "id": "5f4c510d-5be7-4bef-baf0-64b828be9c9b",
    "data": {
      "trackName": "video",
      "previousTimestamp": "15336837615032322",
      "trackType": "video",
      "bitrate": 2962000,
      "newTimestamp": "15336837619774273",
      "discontinuityGap": "575284",
      "timescale": "10000000"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| trackType | string | Type de la piste (Audio/Vidéo). |
| trackName | string | Nom de la piste (fourni par l’encodeur ou, dans le cas du RTMP, généré par le serveur selon le format *TypePiste_DébitBinaire*). |
| bitrate | entier | Débit binaire de la piste. |
| previousTimestamp | string | Timestamp du fragment précédent. |
| newTimestamp | string | Timestamp du fragment actuel. |
| discontinuityGap | string | Écart entre les deux timestamps ci-dessus. |
| timescale | string | Échelle de temps dans laquelle les écarts de timestamp et de discontinuité sont représentés. |

### <a name="common-event-properties"></a>Propriétés d’événements courantes

Un événement contient les données générales suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| topic | string | Rubrique EventGrid. Cette propriété comporte l’ID de ressource pour le compte Media Services. |
| subject | string | Chemin d’accès à la ressource pour le canal Media Services sous le compte Media Services. La concaténation de la rubrique et de l’objet indique l’ID de ressource pour le travail. |
| eventType | string | Un des types d’événements inscrits pour cette source d’événement. Par exemple, « Microsoft.Media.JobStateChange ». |
| eventTime | string | L’heure à quelle l’événement est généré selon l’heure UTC du fournisseur. |
| id | string | Identificateur unique de l’événement. |
| data | object | Données d’événement Media Services. |
| dataVersion | string | Version du schéma de l’objet de données. Le serveur de publication définit la version du schéma. |
| metadataVersion | string | Version du schéma des métadonnées d’événement. Event Grid définit le schéma des propriétés de niveau supérieur. Event Grid fournit cette valeur. |

## <a name="next-steps"></a>Étapes suivantes

[S’inscrire à des événements de changement d’état d’un travail](../media-services/latest/job-state-events-cli-how-to.md)

## <a name="see-also"></a>Voir aussi

- [Kit de développement logiciel (SDK) .NET EventGrid qui inclut les événements Media Services](https://www.nuget.org/packages/Microsoft.Azure.EventGrid/)
- [Définitions d’événements Media Services](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/eventgrid/data-plane/Microsoft.Media/stable/2018-01-01/MediaServices.json)
- [Codes d’erreur d’événement en direct](../media-services/latest/live-event-error-codes.md)
