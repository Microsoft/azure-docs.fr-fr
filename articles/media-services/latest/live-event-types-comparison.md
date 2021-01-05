---
title: Types d’événements en temps réel Azure Media Services | Microsoft Docs
description: Dans Azure Media Services, un événement en direct peut être *Pass-through* ou *Live Encoding*. Cet article contient un tableau détaillé qui compare des types d’événements en direct.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: c79d45cfac22f41f05071b619c444e7b7ab7956a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89397302"
---
# <a name="live-event-types-comparison"></a>Comparaison des types d’événements en direct

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Dans Azure Media Services, un [événement en direct](/rest/api/media/liveevents) peut être défini sur *Pass-through* (un encodeur live local envoie un flux à vitesse de transmission multiple) ou sur *Live Encoding* (un encodeur live local envoie un flux à vitesse de transmission unique). 

Cet article compare les fonctionnalités des types d’événements en direct.

## <a name="types-comparison"></a>Comparaison des types 

Le tableau suivant compare les fonctionnalités des types d’événements en direct. Durant la création, les types sont définis à l’aide de [LiveEventEncodingType](/rest/api/media/liveevents/create#liveeventencodingtype) :

* **LiveEventEncodingType.None** - Un encodeur live local envoie un flux à débits multiples. Les flux reçus transitent par l’Événement en temps réel sans traitement supplémentaire. Également appelé événement en direct pass-through.
* **LiveEventEncodingType.Standard** - Un encodeur live local envoie un flux à débit unique à l’Événement en direct, puis Media Services crée des flux à débits multiples. Si la résolution du flux de contribution est de 720p ou plus, la présélection **Default720p** encode un jeu de 6 paires résolution/débits (plus d’informations plus loin dans cet article).
* **LiveEventEncodingType.Premium1080p** - Un encodeur live local envoie un flux à débit unique à l’Événement en direct, puis Media Services crée des flux à débits multiples. La présélection Default1080p spécifie le jeu de sortie des paires résolution/débits (plus d’informations plus loin dans cet article). 

| Fonctionnalité | Événement en direct pass-through | Événement en direct 1080p standard ou premium |
| --- | --- | --- |
| L’entrée à débit binaire unique est encodée en plusieurs débits binaires dans le cloud |Non |Oui |
| Résolution vidéo maximale pour les flux de contribution |4K (4096x2160 à 60 images/s) |1080p (1920 x 1088 à 30 images/s)|
| Couches maximales recommandées dans le flux de contribution|Jusqu’à 12|1 audio|
| Couches maximales dans la sortie| Comme dans l’entrée|Jusqu’à 6 (voir les présélections système ci-dessous)|
| Bande passante agrégée maximale du flux de contribution|60 Mbits/s|N/A|
| Débit binaire maximal pour une seule couche dans la contribution |20 Mbits/s|20 Mbits/s|
| Prise en charge de pistes audio multilingues|Oui|Non|
| Codecs vidéo d’entrée pris en charge |H.264/AVC et H.265/HEVC|H.264/AVC|
| Codecs vidéo de sortie pris en charge|Comme dans l’entrée|H.264/AVC|
| Profondeur de bit de la vidéo, entrée et sortie, prises en charge|Jusqu’à 10 bits notamment HDR 10/HLG|8 bits|
| Codecs audio d’entrée pris en charge|AAC-LC, HE-AAC v1, HE-AAC v2|AAC-LC, HE-AAC v1, HE-AAC v2|
| Codecs audio de sortie pris en charge|Comme dans l’entrée|AAC-LC|
| Résolution vidéo maximale de la vidéo de sortie|Comme dans l’entrée|Standard - 720p, Premium1080p - 1080p|
| Fréquence d’images maximale de la vidéo d’entrée|60 images/seconde|Standard ou Premium1080p - 30 images/seconde|
| Protocoles d’entrée|RTMP, MP4 fragmenté (Smooth Streaming)|RTMP, MP4 fragmenté (Smooth Streaming)|
| Price|Consultez la [page de tarification](https://azure.microsoft.com/pricing/details/media-services/) et cliquez sur l’onglet « Vidéo en direct »|Consultez la [page de tarification](https://azure.microsoft.com/pricing/details/media-services/) et cliquez sur l’onglet « Vidéo en direct »|
| Durée maximale| 24 hrs x 365 jours, direct linéaire | 24 heures x 365 jours, linéaire en direct (préversion)|
| Capacité à transmettre directement les données des sous-titres CEA 608/708 intégrées|Oui|Oui|
| Possibilité d’activer la transcription en direct|Oui|Oui|
| Prise en charge de l’insertion d’ardoises|Non|Non|
| Prise en charge de la signalisation des annonces via l’API| Non|Non|
| Prise en charge de la signalisation des annonces via des messages SCTE-35 intrabande|Oui|Oui|
| Capacité de récupération suite à de brèves interruptions du flux de contribution|Oui|Partiel|
| Prise en charge des groupes d’images d’entrée non uniformes|Oui|Non – l’entrée doit avoir une durée GOP fixe|
| Prise en charge de l’entrée à fréquence d’images variable|Oui|Non. L’entrée doit avoir une fréquence d’images fixe. Les variations mineures sont tolérées, par exemple pendant les scènes à mouvement élevé. Mais le flux de contribution ne peut pas faire baisser la fréquence d’images (par exemple, à 15 images par seconde).|
| Auto-fermeture de l’événement en direct en cas de perte du flux d’entrée|Non|Après 12 heures si aucune sortie en temps réel n’est en cours d’exécution|

## <a name="system-presets"></a>Présélections système

La présélection [presetName](/rest/api/media/liveevents/create#liveeventencoding) détermine les résolutions et débits de la sortie émanant de l’encodeur live. Si vous utilisez un encodeur live **Standard** (LiveEventEncodingType.Standard), la présélection *Default720p* spécifie un jeu de 6 paires résolution/débit, comme présenté ci-dessous. Autrement, si vous utilisez un encodeur live **Premium1080p** (LiveEventEncodingType.Premium1080p), la présélection *Default1080p* spécifie le jeu de sortie des paires résolution/débit.

> [!NOTE]
> Vous ne pouvez pas appliquer la présélection Default1080p à un événement réel si elle a été configurée pour l’encodage live Standard. Vous obtiendrez une erreur. Vous obtiendrez également une erreur si vous essayez d’appliquer la présélection Default720p à un encodeur live Premium1080p.

### <a name="output-video-streams-for-default720p"></a>Flux de sortie vidéo pour la présélection Default720p

Si la résolution du flux de contribution est de 720p ou plus, la présélection **Default720p** encode le flux dans les 6 couches suivantes. Dans le tableau ci-dessous, le débit est exprimé en Kbits/s, MaxFPS représente cette fréquence d’images maximale autorisée (en images par seconde), Profile représente le profil H.264 utilisé.

| Bitrate | Largeur | Hauteur | IPS max. | Profil |
| --- | --- | --- | --- | --- |
| 3 500 |1 280 |720 |30 |Élevé |
| 2 200 |960 |540 |30 |Élevé |
| 1 350 |704 |396 |30 |Élevé |
| 850 |512 |288 |30 |Élevé |
| 550 |384 |216 |30 |Élevé |
| 200 |340 |192 |30 |Élevé |

> [!NOTE]
> Si vous avez besoin de personnaliser la présélection d’encodage live, ouvrez un ticket de support sur le Portail Azure. Vous devez spécifier la table souhaitée pour la résolution vidéo et les débits. La personnalisation du débit d’encodage audio n’est pas prise en charge. Vérifiez qu’il n’existe qu’une seule couche à 720p, et au maximum six couches. Spécifiez également que vous demandez une présélection.

### <a name="output-video-streams-for-default1080p"></a>Flux de sortie vidéo pour la présélection Default1080p

Si la résolution du flux de contribution est de 1080p, la présélection **Default1080p** encode le flux dans les 6 couches suivantes.

| Bitrate | Largeur | Hauteur | IPS max. | Profil |
| --- | --- | --- | --- | --- |
| 5500 |1920 |1080 |30 |Élevé |
| 3000 |1 280 |720 |30 |Élevé |
| 1 600 |960 |540 |30 |Élevé |
| 800 |640 |360 |30 |Élevé |
| 400 |480 |270 |30 |Élevé |
| 200 |320 |180 |30 |Élevé |

> [!NOTE]
> Si vous avez besoin de personnaliser la présélection d’encodage live, ouvrez un ticket de support sur le Portail Azure. Vous devez spécifier la table de résolution et de débits binaires souhaitée. Vérifiez qu’il n’existe qu’une seule couche à 1080p, et au maximum 6 couches. Spécifiez également que vous demandez une présélection pour un encodeur live Premium1080p. Les valeurs des débits et des résolutions spécifiques peuvent être ajustées au fil du temps.

### <a name="output-audio-stream-for-default720p-and-default1080p"></a>Flux de sortie audio pour Default720p et Default1080p

Pour les présélections *Default720p* et *Default1080p*, l’audio est encodé en Stéréo AAC-LC à 128 Kbits/s. Le taux d’échantillonnage suit celui de la piste audio dans le flux de contribution.

## <a name="implicit-properties-of-the-live-encoder"></a>Propriétés implicites de l’encodeur live

La section précédente décrit les propriétés de l’encodeur live qui peuvent être contrôlées de manière explicite, via la présélection, comme le nombre de couches, les résolutions et les débits. Cette section clarifie les propriétés implicites.

### <a name="group-of-pictures-gop-duration"></a>Durée d’un groupe d’images (GOP)

L’encodeur live suit la structure d’un [groupe d’images](https://en.wikipedia.org/wiki/Group_of_pictures) de la contribution de flux, ce qui signifie que les couches de sortie auront la même durée que le groupe d’images. Par conséquent, il est recommandé de configurer l’encodeur local pour produire un flux de contribution ayant corrigé la durée du groupe d’images (généralement de 2 secondes). Cela garantit que les flux HLS et MPEG DASH sortants du service ont également résolu les durées du groupe d’images. La plupart des appareils devraient être en mesure de tolérer de légères variations dans les durées du groupe d’images.

### <a name="frame-rate"></a>Fréquence d’images

L’encodeur live suit également les durées des trames vidéo individuelles dans le flux de contribution, ce qui signifie que les couches de sortie auront des images de mêmes durées. Par conséquent, il est recommandé de configurer l’encodeur local pour produire un flux de contribution ayant une fréquence d’images fixe (30 images par seconde au maximum). Cela garantit que les flux HLS et MPEG DASH sortants du service ont également résolu les durées de la fréquence d’images. La plupart des appareils devraient être en mesure de tolérer de légères variations dans les fréquences d’images, mais rien ne garantit que l’encodeur live produira une sortie lue correctement. Votre encodeur live local ne doit pas supprimer des images (par exemple, dans des conditions de batterie faible) ni faire varier la fréquence d’images en aucune façon.

### <a name="resolution-of-contribution-feed-and-output-layers"></a>Résolution du flux de contribution et des couches de sortie

L’encodeur live est configuré pour éviter de convertir le flux de contribution. Par conséquent, la résolution maximale des couches de sortie ne dépasse pas celle du flux de contribution.

Par exemple, si vous envoyez un flux de contribution à 720p pour un événement en direct configuré pour un encodage live Default1080p, la sortie aura uniquement 5 couches, en partant de 720p à 3 Mbits/s, jusqu’à 1080p à 200 Kbits/s. Ou bien si vous envoyez un flux de contribution à 360p dans un événement en direct pour un encodage live Standard, la sortie contiendra 3 couches (aux résolutions 288p, 216p et 192p). Dans le cas dégénéré, si vous envoyez un flux de contribution de 160 x 90 pixels dans un encodeur live Standard, la sortie contiendra une couche à la résolution 160x90 au même débit que celui du flux de contribution.

### <a name="bitrate-of-contribution-feed-and-output-layers"></a>Débit du flux de contribution et des couches de sortie

L’encodeur live est configuré pour respecter les paramètres de débit de la présélection, peu importe le débit du flux de contribution. Par conséquent, le débit des couches de sortie peut dépasser celui du flux de contribution. Par exemple, si vous envoyez un flux de contribution d’une résolution de 720p à 1 Mbits/s, les couches de sortie restent identiques à ceux de la [table](live-event-types-comparison.md#output-video-streams-for-default720p) ci-dessus.

## <a name="next-steps"></a>Étapes suivantes

[Vue d’ensemble du streaming en direct](live-streaming-overview.md)
