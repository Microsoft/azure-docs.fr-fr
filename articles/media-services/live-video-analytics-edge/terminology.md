---
title: Terminologie de Live Video Analytics sur IoT Edge - Azure
description: Cet article fournit une vue d’ensemble de la terminologie de Live Video Analytics sur IoT Edge.
ms.topic: conceptual
ms.date: 05/30/2020
ms.openlocfilehash: e3a77b69adf2241a4af2652db4edb6673a63b4f0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88690610"
---
# <a name="terminology"></a>Terminologie

Cet article fournit une vue d’ensemble de la terminologie liée à [Live Video Analytics sur IoT Edge](overview.md).

## <a name="azure-media-services"></a>Azure Media Services

Azure Media Services est une plateforme multimédia cloud qui vous permet de créer des solutions multimédias. Vous pouvez en savoir plus à son sujet dans la documentation [Azure Media Services](../latest/media-services-overview.md).

## <a name="asset"></a>Asset

Un [actif multimédia](../latest/assets-concept.md) est une entité dans Azure Media Services qui est mappée à un conteneur d’objets blob dans le compte de stockage Azure associé à un compte Media Services. Tous les fichiers associés à un actif multimédia sont stockés en tant qu’objets blob dans ce conteneur tandis que Media Services contient les métadonnées (par exemple, un nom, une description, une heure de création) associées à l’actif multimédia.

Live Video Analytics sur IoT Edge peut créer des actifs multimédias et/ou ajouter des données à des actifs multimédias existants. Ainsi, les scénarios d’enregistrement et lecture vidéo en continu et basé sur des événements (avec capture vidéo sur l’appareil périphérique, enregistrement sur Azure Media Services et lecture par le biais des fonctionnalités de streaming Azure Media Services existantes) sont possibles.

## <a name="grpc"></a>gRPC

[gRPC](https://grpc.io/docs/guides/) est un Framework d’appel de procédure distante (RPC) indépendant du langage et de haute performance. Il utilise des schémas structurés basés sur une session via [Protocol Buffers 3](https://developers.google.com/protocol-buffers/docs/proto3) en tant que format d’échange de messages sous-jacent pour la communication.

## <a name="media-graph"></a>Graphe multimédia

Un [graphe multimédia](media-graph-concept.md) vous permet de définir l’emplacement à partir duquel le contenu multimédia doit être capturé, la manière dont il doit être traité et où les résultats doivent être remis. Il vous permet de définir un graphe constitué de sources, processeurs et nœuds récepteurs. Il offre ainsi la possibilité de créer des applications d’analytique vidéo dynamiques. Le concept du graphe multimédia est abordé plus en détail dans la page conceptuelle correspondante.

## <a name="recording"></a>Enregistrement

Dans le contexte d’un système de gestion vidéo pour les caméras de sécurité, l’enregistrement vidéo fait référence au processus de capture des vidéos à partir des caméras et à leur stockage dans un ou plusieurs fichiers afin de les regarder ultérieurement sur des applications mobiles et des navigateurs. L’enregistrement vidéo peut être classé dans la catégorie de l’[enregistrement vidéo en continu](continuous-video-recording-concept.md) et l’[enregistrement vidéo basé sur des événements](event-based-video-recording-concept.md). Ces catégories sont expliquées plus en détail dans la page conceptuelle sur l’[enregistrement vidéo](video-recording-concept.md).

## <a name="rtsp"></a>RTSP

[RTSP](https://tools.ietf.org/html/rfc2326) (Real-Time Streaming Protocol) fait référence au protocole de streaming en temps réel. Il s’agit d’un protocole au niveau de l’application pour contrôler la remise des données avec des propriétés en temps réel. Le protocole RTSP fournit un cadre extensible qui permet une remise contrôlée à la demande de données en temps réel, comme des données audio et vidéo. 

## <a name="streaming"></a>Diffusion en continu

Si vous avez regardé une vidéo sur un appareil mobile à partir de services comme Netflix, YouTube, etc., vous avez expérimenté le streaming vidéo. La lecture commence peu après avoir appuyé sur « Lecture » (si vous disposez d’une bande passante suffisante) et vous pouvez effectuer une recherche dans la chronologie de la vidéo. Avec le streaming, l’idée est de fournir uniquement la partie de la vidéo en cours de visionnage et de laisser l’utilisateur démarrer la vidéo alors que les données sont encore en cours de transfert depuis un serveur vers le client de lecture. Dans le contexte d’Azure Media Services, le [streaming](https://en.wikipedia.org/wiki/Streaming_media) fait référence au processus de remise de contenu multimédia depuis [Azure Media Services](../azure-media-player/azure-media-player-overview.md) vers un client de streaming (par exemple, le Lecteur multimédia Azure). Vous pouvez utiliser Azure Media Services pour diffuser en streaming des vidéos sur des clients à l’aide de protocoles de streaming multimédia HTTP, comme [HTTP Live Streaming (TLS)](https://developer.apple.com/streaming/) et [MPEG-DASH](https://dashif.org/about/). HLS est pris en charge par le Lecteur multimédia Azure et des lecteurs web comme [JW Player](https://www.jwplayer.com/), [hls.js](https://github.com/video-dev/hls.js/), [VideoJS](https://videojs.com/), [Shaka Player de Google](https://github.com/google/shaka-player), et vous pouvez effectuer un rendu en mode natif dans des applications mobiles avec [Exoplayer](https://github.com/google/ExoPlayer) d’Android et [AV Foundation](https://developer.apple.com/av-foundation/) d’iOS. MPEG-DASH est également pris en charge par le Lecteur multimédia Azure. [Retrouvez la liste des clients dans cette page](https://dashif.org/clients/). 

En utilisant des [graphes multimédias](#media-graph) pour enregistrer des vidéos dans un actif multimédia dans Azure Media Services, vous pouvez utiliser la fonctionnalité de streaming de Media Services pour diffuser des flux vidéo en HLS et DASH. Pour en savoir plus à ce sujet, consultez l’article sur la [lecture vidéo](video-playback-concept.md).

## <a name="vms"></a>VMS

[VMS](https://en.wikipedia.org/wiki/Video_management_system) (Video Management System) fait référence à un système de gestion vidéo. De tels systèmes sont utilisés pour configurer et contrôler des caméras de vidéosurveillance, ainsi que pour capturer et enregistrer des vidéos à partir de ces caméras. Ces systèmes fournissent également des applications clientes pour lire les vidéos enregistrées.

## <a name="next-steps"></a>Étapes suivantes

[Graphes multimédias](media-graph-concept.md)
