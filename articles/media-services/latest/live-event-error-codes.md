---
title: Codes d'erreur des événements en direct Azure Media Services | Microsoft Docs
description: Cet article répertorie les codes d'erreur des événements en direct.
author: IngridAtMicrosoft
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: error-reference
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 13277c3d46d5b68aa5705699f45a2bf9e97a78d8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89291532"
---
# <a name="media-services-live-event-error-codes"></a>Codes d'erreur des événements en direct Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Les tableaux de cette section répertorient les codes d’erreur [Événement en temps réel](live-events-outputs-concept.md).

## <a name="liveeventconnectionrejected"></a>LiveEventConnectionRejected

Lorsque vous vous abonnez aux événements [Event Grid](../../event-grid/index.yml) pour un événement en direct, il se peut que vous voyiez l’une des erreurs suivantes de l’événement [LiveEventConnectionRejected](media-services-event-schemas.md#liveeventconnectionrejected).

| Code de résultat | Description |
| ----------- | ----------- |
| MPE_RTMP_APPID_AUTH_FAILURE | URL d’ingestion incorrecte |
| MPE_INGEST_ENCODER_CONNECTION_DENIED | L’adresse IP de l’encodeur n’est pas présente dans la liste d’adresses IP autorisées configurée |
| MPE_INGEST_RTMP_SETDATAFRAME_NOT_RECEIVED | L’encodeur RTMP n’a pas envoyé la commande setDataFrame. |
| MPE_INGEST_CODEC_NOT_SUPPORTED | Le codec spécifié n’est pas pris en charge. |
| MPE_INGEST_DESCRIPTION_INFO_NOT_RECEIVED |Les informations de description de média n’ont pas été reçues avant la remise des données de média réelles.|
| MPE_INGEST_MEDIA_QUALITIES_EXCEEDED |Le nombre de qualités pour le type d’audio ou de vidéo dépasse la limite maximale autorisée.|
| MPE_INGEST_BITRATE_AGGREGATED_EXCEEDED |Le débit binaire total entrant dans un service d’événement ou de canal en direct dépasse la limite maximale autorisée.|
| MPE_RTMP_FLV_TAG_TIMESTAMP_INVALID | Le timestamp du FLVTag vidéo ou audio n’est pas valide à partir de l’encodeur RTMP. |
| MPE_INGEST_FRAMERATE_EXCEEDED | Les flux entrants avec taux de trames ingérés par l’encodeur ont dépassé le nombre maximal autorisé de 30 T/s pour l’encodage d’événements/canaux en direct.|
| MPE_INGEST_VIDEO_RESOLUTION_NOT_SUPPORTED | Les flux entrants ingérés par l’encodeur ont dépassé les résolutions autorisées suivantes : 1920 x 1088 pour l'encodage des canaux/événements en direct et 4096 x 2160 pour le transfert des canaux/événements en direct.|
| MPE_INGEST_RTMP_TOO_LARGE_UNPROCESSED_FLV | L’événement en direct a reçu une grande quantité de données audio à la fois, ou une grande quantité de données vidéo sans images clés. Nous avons déconnecté l’encodeur afin de lui permettre de réessayer avec des données correctes. |

## <a name="liveeventencoderdisconnected"></a>LiveEventEncoderDisconnected

Il se peut que vous voyiez l’une des erreurs suivantes de l’événement [LiveEventEncoderDisconnected](media-services-event-schemas.md#liveeventencoderdisconnected).

|Code de résultat|Description|
|---|---|
|MPE_RTMP_SESSION_IDLE_TIMEOUT|La session RTMP a expiré après avoir été inactive pendant le temps maximal autorisé.|
|MPE_RTMP_FLV_TAG_TIMESTAMP_INVALID|Le timestamp du FLVTag vidéo ou audio n’est pas valide à partir de l’encodeur RTMP.|
|MPE_CAPACITY_LIMIT_REACHED|L’encodeur envoie les données trop vite.|
|Codes d’erreur inconnus|Ces codes d’erreur peuvent varier de l’erreur de mémoire à des entrées de table de hachage en double.|


## <a name="see-also"></a>Voir aussi

[Codes d'erreur du point de terminaison de streaming (origine)](streaming-endpoint-error-codes.md)

## <a name="next-steps"></a>Étapes suivantes

[Tutoriel : Diffuser en direct avec Media Services](stream-live-tutorial-with-api.md)
