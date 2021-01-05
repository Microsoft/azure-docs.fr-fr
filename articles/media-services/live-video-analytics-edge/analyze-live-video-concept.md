---
title: 'Analyse de la vidéo en direct sans enregistrement : Azure'
description: Un graphique multimédia peut être utilisé pour extraire les analyses d’un flux vidéo en direct, sans avoir à les enregistrer en périphérie ou dans le cloud en toute simplicité. Cet article aborde ce concept.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 25a7cadc47603b726542fa391d441e1fbca78908
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97398966"
---
# <a name="analyzing-live-video-without-any-recording"></a>Analyse de la vidéo en direct sans enregistrement

## <a name="suggested-pre-reading"></a>Lecture préalable suggérée 

* [Concept de graphique multimédia](media-graph-concept.md)
* [Enregistrement de vidéo basé sur les événements](event-based-video-recording-concept.md)

## <a name="overview"></a>Vue d’ensemble  

Vous pouvez utiliser les graphiques multimédias pour analyser des vidéos en direct, sans en enregistrer les portions dans un fichier ou une ressource. Les graphiques multimédias ci-dessous sont similaires à ceux de l’article [Enregistrement de vidéo basé sur les événements](event-based-video-recording-concept.md), mais sans nœud récepteur de ressource ou de fichier.

### <a name="motion-detection"></a>Détection de mouvement

Le graphique multimédia présenté ci-dessous se compose d’un nœud de [source RTSP](media-graph-concept.md#rtsp-source), d’un nœud de [processeur de détection de mouvement](media-graph-concept.md#motion-detection-processor) et d’un nœud [de réception de messages du hub IoT](media-graph-concept.md#iot-hub-message-sink). La représentation JSON de la topologie d’un tel graphique multimédia peut être trouvée [ici](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/motion-detection/topology.json). Ce graphique vous permet de détecter le mouvement dans les flux vidéo en direct entrants et de transmettre les événements à d’autres applications et services via le nœud récepteur de messages du IoT Hub. Les applications ou services externes peuvent déclencher une alerte ou envoyer une notification aux personnes appropriées.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video/motion-detection.svg" alt-text="Live Video Analytics basé sur la détection de mouvement":::

### <a name="analyzing-video-using-a-custom-vision-model"></a>Analyse de vidéos à l’aide d’un modèle Custom Vision 

Le graphique multimédia présenté ci-dessous vous permet d’analyser un flux vidéo en direct à l’aide d’un modèle Custom Vision empaqueté dans un module distinct. La représentation JSON de la topologie d’un tel graphique multimédia peut être trouvée [ici](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/httpExtension/topology.json). Vous pouvez voir des exemples sur l’habillage des modèles dans des modules IoT Edge qui s’exécutent en tant que service d’inférence [ici](https://github.com/Azure/live-video-analytics/tree/master/utilities/video-analysis).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video/motion-detected-frames.svg" alt-text="Analyse vidéo en direct basée sur un module d’inférence externe":::

Dans ce graphique multimédia, l’entrée vidéo de la source RTSP est envoyée à un [nœud de processeur d’extension HTTP](media-graph-concept.md#http-extension-processor), qui envoie des images (au format JPEG, BMP ou PNG) à un service d’inférence externe sur REST. Les résultats du service d’inférence externe sont récupérés par le nœud d’extension HTTP et relayés à IoT Edge Hub via le nœud de IoT Hub de réception des messages. Ce type de graphique multimédia peut être utilisé pour créer des solutions pour divers scénarios, tels que la compréhension de la distribution de séries chronologiques de véhicules à l’intersection, la compréhension du modèle de trafic des consommateurs dans un magasin de vente au détail, etc.
>[!TIP]
> Vous pouvez gérer la fréquence d’images dans le nœud de processeur d’extension HTTP à l’aide du champ `samplingOptions` avant de l’envoyer en aval.

Cet exemple peut être amélioré en utilisant un processeur de détecteur de mouvement avant le nœud de processeur d’extension HTTP. Cela réduira la charge sur le service d’inférence, car il est utilisé uniquement lorsqu’il y a une activité de mouvement dans la vidéo.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/analyze-live-video/custom-model.svg" alt-text="Analyse vidéo en direct basée sur les trames détectées par un module d’inférence externe":::

## <a name="next-steps"></a>Étapes suivantes

[Enregistrement vidéo en continu](continuous-video-recording-concept.md)
