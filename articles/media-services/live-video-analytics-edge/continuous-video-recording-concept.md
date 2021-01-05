---
title: Enregistrement de vidéo en continu – Azure
description: L’enregistrement de vidéo continu (CVR) fait référence au processus d’enregistrement vidéo continu à partir d’une source vidéo. Cette rubrique explique ce qu’est le CVR.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 04f09f1968e647c57ba0913a9e7f9e601d045771
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89566690"
---
# <a name="continuous-video-recording"></a>Enregistrement de vidéo continu  

## <a name="suggested-pre-reading"></a>Lecture préalable suggérée  

* [Concept de graphe multimédia](media-graph-concept.md)
* [Concept d’enregistrement vidéo](video-recording-concept.md)

## <a name="overview"></a>Vue d’ensemble

L’enregistrement de vidéo continu (CVR) fait référence au processus d’enregistrement vidéo continu à partir d’une source vidéo. Live Video Analytics sur IoT Edge prend en charge l’enregistrement de vidéos en continu, 24 h sur 24 et 7 jours sur 7, à partir d’une caméra CCTV via un [graphe multimédia](media-graph-concept.md) constitué d’un nœud source RTSP et d’un nœud récepteur de ressources. Le diagramme ci-dessous montre une représentation graphique d’un tel graphe multimédia. La représentation JSON de la [topologie de graphe](media-graph-concept.md?branch=release-preview-media-services-lva#media-graph-topologies-and-instances) d’un tel graphe multimédia peut être trouvée [ici](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph/topologies/cvr-asset).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/continuous-video-recording/continuous-video-recording-overview.svg" alt-text="Enregistrement de vidéo continu":::

Le graphe multimédia représenté ci-dessus peut être exécuté sur un périphérique, le récepteur de ressources enregistrant la vidéo sur une [ressource](terminology.md#asset) Azure Media Services. La vidéo sera enregistrée aussi longtemps que le graphe multimédia sera activé. Étant donné que la vidéo est enregistrée en tant que ressource, elle peut être lue à l’aide des capacités de diffusion en continu existantes de Media Services. Pour plus d’informations, consultez [Lecture du contenu enregistré](video-playback-concept.md).

## <a name="resilient-recording"></a>Enregistrement résilient

Live Video Analytics sur IoT Edge prend en charge le fonctionnement dans des conditions de réseau imparfaites, où le périphérique peut parfois perdre la connectivité avec le cloud ou subir une baisse de la bande passante disponible. Pour tenir compte de cela, la vidéo de la source est enregistrée localement dans un cache et est automatiquement synchronisée avec la ressource de manière périodique. Si vous examinez la [topologie de graphe JSON](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph/topologies/cvr-asset/topology.json), vous verrez que les propriétés suivantes sont définies :

```
"segmentLength": "PT30S",
"localMediaCacheMaximumSizeMiB": "2048",
"localMediaCachePath": "/var/lib/azuremediaservices/tmp/",
```

Les deux dernières propriétés sont pertinentes pour l’enregistrement résilient (toutes deux sont également des propriétés requises pour un nœud récepteur de ressources). La propriété localMediaCachePath indique au récepteur de ressources d’utiliser ce chemin d’accès au dossier pour mettre en cache les données multimédia avant de les charger dans la ressource. Vous pouvez consulter [cet article](../../iot-edge/how-to-access-host-storage-from-module.md) pour comprendre comment le module Edge peut utiliser le stockage local de votre appareil. La propriété localMediaCacheMaximumSizeMiB définit la quantité d’espace disque que le récepteur de ressources peut utiliser comme cache (1 Mio = 1024 * 1024 octets). 

Si votre module Edge perd la connectivité pendant une très longue période et que le contenu stocké dans le dossier cache atteint la valeur localMediaCacheMaximumSizeMiB, le récepteur de ressources commencera à supprimer les données du cache, en commençant par les données les plus anciennes. Par exemple, si l’appareil a perdu la connexion à 10 h et que le cache atteint la limite maximale à 18 h, le récepteur de ressources commence à supprimer les données enregistrées à 10 h. 

Lorsque la connectivité réseau est restaurée, le récepteur de ressources commence à charger à partir du cache, en commençant une fois encore par les données les plus anciennes. Dans l’exemple ci-dessus, supposons que 5 minutes de vidéo doivent être supprimées du cache au moment où la connectivité est rétablie (disons à 18 h 02), alors le récepteur de ressources commencera à charger à partir de 10 h 05.

Si vous examinez ultérieurement la ressource à l’aide de [ces API](playback-recordings-how-to.md), vous verrez qu’il y a un intervalle dans la ressource à peu près entre 10 h et 10 h 05.

## <a name="segmented-recording"></a>Enregistrement segmenté  

Comme indiqué ci-dessus, le nœud récepteur de ressources enregistre la vidéo dans un cache local et charge régulièrement la vidéo dans le cloud. La propriété segmentLength (indiquée dans la section ci-dessus) vous permet de contrôler le coût des transactions d’écriture associé à l’écriture des données dans votre compte de stockage où la ressource est enregistrée. Par exemple, si vous augmentez la période de chargement de 30 secondes à 5 minutes, le nombre de transactions de stockage diminuera d’un facteur de 10 (5*60/30).

La propriété segmentLength garantit que le module Edge charge la vidéo au maximum une fois par secondes segmentLength. Cette propriété a une valeur minimale de 30 secondes (il s’agit également de la valeur par défaut) et peut être augmentée par incréments de 30 secondes jusqu’à un maximum de 5 minutes.

> [!NOTE]
> Consultez l’article [Lecture des enregistrements](playback-recordings-how-to.md) pour connaître l’effet de segmentLength sur la lecture.

## <a name="see-also"></a>Voir aussi

* [Enregistrement de vidéo basé sur les événements](event-based-video-recording-concept.md)
* [Lecture du contenu enregistré](video-playback-concept.md)

## <a name="next-steps"></a>Étapes suivantes

[Didacticiel : enregistrement de vidéo en continu](continuous-video-recording-tutorial.md)
