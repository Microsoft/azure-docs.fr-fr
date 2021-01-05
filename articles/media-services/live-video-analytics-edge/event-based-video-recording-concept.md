---
title: Enregistrement de vidéo basé sur les événements – Azure
description: Un enregistrement vidéo basé sur les événements est un enregistrement vidéo qui a été déclenché par un événement. L’événement en question peut provenir du traitement du signal vidéo lui-même (par exemple, la détection de mouvement) ou d’une source indépendante (par exemple, l’ouverture d’une porte).  Certains cas d’usage relatifs à l’enregistrement de vidéo basé sur les événements sont décrits dans cet article.
ms.topic: conceptual
ms.date: 05/27/2020
ms.openlocfilehash: 6a5f4873b2cfef8d9a6594916d82cd30a3bc1cc2
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97401597"
---
# <a name="event-based-video-recording"></a>Enregistrement de vidéo basé sur les événements  
 
## <a name="suggested-pre-reading"></a>Lecture préalable suggérée  

* [Enregistrement de vidéo continu](continuous-video-recording-concept.md)
* [Lecture du contenu enregistré](video-playback-concept.md)
* [Concept de graphe multimédia](media-graph-concept.md)

## <a name="overview"></a>Vue d’ensemble 

Un enregistrement vidéo basé sur les événements est un enregistrement vidéo qui a été déclenché par un événement. L’événement en question peut provenir du traitement du signal vidéo lui-même (par exemple, la détection de mouvement) ou d’une source indépendante (par exemple, l’ouverture d’une porte). 

Vous pouvez enregistrer une vidéo (déclenchée par un événement) à partir d’une caméra CCTV vers une ressource Media Services à l’aide d’un graphe multimédia constitué d’un nœud [source RTSP](media-graph-concept.md#rtsp-source), d’un nœud [récepteur de ressources](media-graph-concept.md#asset-sink) et d’autres nœuds, comme indiqué dans les cas d’utilisation ci-dessous. Vous pouvez configurer le nœud [récepteur de ressources](media-graph-concept.md#asset-sink) pour qu’il génère de nouvelles ressources chaque fois qu’un événement se produit, de sorte que la vidéo qui correspond à chaque événement se trouve dans sa propre ressource. Vous pouvez également choisir d’utiliser une seule ressource pour conserver la vidéo de tous les événements. 

Comme alternative au nœud récepteur de ressources, vous pouvez utiliser un nœud [récepteur de fichiers](media-graph-concept.md#file-sink) pour capturer des extraits de vidéo (liés à un événement qui vous intéresse) à un emplacement spécifié sur le système de fichiers local sur le périphérique. 

Certains cas d’usage relatifs à l’enregistrement de vidéo basé sur les événements sont décrits dans cet article.

### <a name="video-recording-based-on-motion-detection"></a>Enregistrement de vidéo basé sur la détection de mouvement  

Dans ce cas d’usage, vous pouvez enregistrer des clips vidéo uniquement lorsqu’un mouvement est détecté dans la vidéo et être alerté quand un tel clip vidéo est généré. Cela pourrait être pertinent dans les scénarios de sécurité commerciale impliquant la protection des infrastructures critiques. En générant des alertes et en enregistrant la vidéo lorsqu’un mouvement inattendu est détecté, vous pouvez améliorer l’efficacité de l’opérateur humain, car une action est nécessaire uniquement lorsque l’alerte est générée.

Le diagramme ci-dessous montre une représentation graphique d’un graphe multimédia qui résout ce cas d’usage. La représentation JSON de la topologie d’un tel graphe multimédia peut être trouvée [ici](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/evr-motion-assets/topology.json).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording/motion-detection.svg" alt-text="Enregistrement de vidéo basé sur la détection de mouvement":::

Dans le diagramme, le nœud source RTSP capture le flux vidéo en direct à partir de la caméra et le remet à un nœud [processeur de détection de mouvement](media-graph-concept.md#motion-detection-processor). Lors de la détection de mouvement dans la vidéo en direct, le nœud processeur de détection de mouvement génère un événement qui accède au nœud [processeur de porte de signal](media-graph-concept.md#signal-gate-processor), ainsi qu’au nœud récepteur de messages IoT Hub. Ce dernier envoie les événements au hub IoT Edge, à partir duquel ils peuvent être acheminés vers d’autres destinations pour déclencher des alertes. 

Un événement du nœud détecteur de mouvement déclenchera le nœud processeur de porte de signal, et il laissera le flux vidéo en direct du nœud source RTSP passer au nœud récepteur de ressources. En l’absence de tels événements de détection de mouvement ultérieurs, la porte se fermera après un laps de temps défini. Cela détermine la durée de la vidéo enregistrée.

### <a name="video-recording-based-on-events-from-other-sources"></a>Enregistrement de vidéo basé sur les événements provenant d’autres sources  

Dans ce cas d’usage, les signaux d’un autre capteur IoT peuvent être utilisés pour déclencher l’enregistrement de la vidéo. Le diagramme ci-dessous montre une représentation graphique d’un graphe multimédia qui résout ce cas d’usage. La représentation JSON de la topologie d’un tel graphe multimédia peut être trouvée [ici](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/evr-hubMessage-files/topology.json).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording/other-sources.svg" alt-text="Enregistrement de vidéo basé sur les événements provenant d’autres sources":::

Dans le diagramme, le capteur externe envoie des événements au hub IoT Edge. Les événements sont ensuite acheminés vers le nœud processeur de porte de signal via le nœud [source de messages IoT Hub](media-graph-concept.md#iot-hub-message-source). Le comportement du nœud processeur de porte de signal est le même que dans le cas d’usage précédent : il s’ouvre et laisse le flux vidéo en direct passer du nœud source RTSP au nœud récepteur de fichiers (ou au nœud récepteur de ressources) lorsqu’il est déclenché par l’événement externe. 

Si vous utilisez un nœud récepteur de fichiers, la vidéo sera enregistrée dans le système de fichiers local sur le périphérique. Sinon, si vous utilisez un nœud récepteur de ressources, la vidéo sera enregistrée dans une ressource.

### <a name="video-recording-based-on-an-external-inferencing-module"></a>Enregistrement de vidéo basé sur un module d’inférence externe 

Dans ce cas d’usage, vous pouvez enregistrer des clips vidéo basés sur le signal d’un système logique externe. Un exemple de ce cas d’usage peut être l’enregistrement d’un clip vidéo uniquement lorsqu’un camion est détecté dans le flux vidéo du trafic sur une autoroute. Le diagramme ci-dessous montre une représentation graphique d’un graphe multimédia qui résout ce cas d’usage. La représentation JSON de la topologie d’un tel graphe multimédia peut être trouvée [ici](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/evr-hubMessage-assets/topology.json).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/event-based-video-recording/external-inferencing-module.svg" alt-text="Enregistrement de vidéo basé sur un module d’inférence externe":::

Dans le diagramme, le nœud source RTSP capture le flux vidéo en direct à partir de la caméra et le remet à deux branches : l’une possède un nœud [processeur de porte de signal](media-graph-concept.md#signal-gate-processor), et l’autre utilise un nœud [d’extension HTTP](media-graph-concept.md) pour envoyer des données à un module logique externe. Le nœud d’extension HTTP permet au graphe multimédia d’envoyer des trames d’image (au format JPEG, BMP ou PNG) à un service d’inférence externe via REST. Généralement, ce chemin de signal ne peut prendre en charge que des fréquences d’images faibles (< 5 fps). Vous pouvez utiliser le nœud de processeur d’extension HTTP pour réduire la fréquence d’images de la vidéo allant vers le modèle d’inférence externe.

Les résultats du service d’inférence externe sont récupérés par le nœud d’extension HTTP et relayés au hub IoT Edge via le nœud récepteur de messages IoT Hub, où ils peuvent être traités par le module logique externe. Si le service d’inférence est capable de détecter des véhicules, par exemple, le module logique peut rechercher un véhicule spécifique, tel qu’un bus ou un camion. Lorsque le module logique détecte l’objet d’intérêt, il peut déclencher le nœud processeur de porte de signal en envoyant un événement via le hub IoT Edge au nœud source des messages IoT Hub dans le graphique. La sortie de la porte de signal est affichée pour atteindre un nœud récepteur de fichiers ou un nœud récepteur de ressources. Dans le premier cas, la vidéo sera enregistrée dans le système de fichiers local sur le périphérique. Dans le dernier cas, la vidéo sera enregistrée dans une ressource.

Cet exemple peut être amélioré à l’aide d’un processeur de détecteur de mouvement avant le nœud de processeur d’extension HTTP. Cela permettra de réduire la charge du service d’inférence, par exemple la nuit lorsqu’il peut y avoir de longues périodes sans aucun véhicule sur la route. 

## <a name="next-steps"></a>Étapes suivantes

[Didacticiel : enregistrement de vidéo basé sur les événements](event-based-video-recording-tutorial.md)
