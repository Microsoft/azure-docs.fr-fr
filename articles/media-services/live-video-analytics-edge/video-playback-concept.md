---
title: Lecture de vidéo - Azure
description: Espace réservé
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 17b1f93c18dfb013916c4c0d3756fb97a73e2675
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87042968"
---
# <a name="video-playback"></a>Lecture de vidéo 

## <a name="suggested-pre-reading"></a>Lecture préalable suggérée 

* [Présentation de Live Video Analytics sur IoT Edge](overview.md)
* [Terminologie de Live Video Analytics sur IoT Edge](terminology.md)
* [Concept de graphe multimédia](media-graph-concept.md)

## <a name="overview"></a>Vue d’ensemble  

Vous pouvez utiliser des [graphes multimédias](media-graph-concept.md) pour enregistrer des vidéos dans un [actif multimédia](terminology.md#asset) Azure Media Services. Dans ce document, vous pouvez découvrir les étapes à suivre pour lire un actif multimédia à l’aide des fonctionnalités de streaming existantes d’Azure Media Services.

## <a name="streaming-endpoint"></a>point de terminaison de diffusion en continu 

Vous pouvez utiliser Azure Media Services pour [diffuser](terminology.md#streaming) l’actif multimédia sur des lecteurs vidéo en utilisant des protocoles de streaming multimédia HTTP, comme HTTP Live Streaming (HLS) et MPEG-DASH. Cette conversion d’un contenu enregistré en formats de streaming est gérée par un [point de terminaison de streaming](../latest/streaming-endpoint-concept.md), qui est une ressource que vous devez provisionner dans votre compte Azure Media Services.

## <a name="streaming-policy"></a>Stratégie de streaming 

Azure Media Services vous propose différentes méthodes pour sécuriser vos flux vidéo, comme décrit dans l’article [Protéger votre contenu à l’aide du chiffrement dynamique de Media Services](../latest/content-protection-overview.md). Les options élémentaires de protection du contenu sont les suivantes :

* **Streaming en clair** : aucun chiffrement n’est appliqué pendant le streaming.
* **Utiliser Advanced Encryption Standard (AES-128)**  : implémentez une méthode pour remettre les clés de déchiffrement de la vidéo uniquement aux utilisateurs authentifiés.
* **Utiliser des systèmes de gestion des droits numériques (DRM)**  : contrôlez l’utilisation, la modification et la distribution de vidéos sur des appareils qui appliquent ces stratégies.

Pour assurer la protection du contenu, vous pouvez définir et créer une [stratégie de streaming](../latest/streaming-policy-concept.md) dans votre compte Media Services, puis l’utiliser pour le streaming de tous les actifs multimédias (en supposant que tous les flux ont les mêmes critères de sécurité). Vous pouvez également utiliser l’une des stratégies prédéfinies (par exemple, Predefined_ClearStreamingOnly).

## <a name="streaming-locator"></a>Localisateur de streaming  

Une fois que vous avez démarré un point de terminaison de streaming dans votre compte Media Services et que vous avez une stratégie de streaming définie, vous pouvez diffuser des médias enregistrés à partir d’un actif multimédia par le biais des protocoles HLS ou DASH. Les lecteurs web et les applications mobiles ont besoin d’une URL pointant vers ce flux HLS ou DASH. Vous pouvez générer cette URL à l’aide du [localisateur de streaming](../latest/streaming-locators-concept.md). Comme expliqué dans cet article et illustré par l’exemple cité dans [Créer un localisateur de streaming et générer des URL](../latest/create-streaming-locator-build-url.md), l’URL de streaming se compose à partir du point de terminaison de streaming, de la stratégie de streaming et du localisateur de streaming.

## <a name="content-recorded-using-file-sink"></a>Contenu enregistré à l’aide du récepteur de fichiers  

Comme décrit dans [Récepteur de fichiers de graphe multimédia](media-graph-concept.md#file-sink), vous pouvez utiliser des graphes multimédias pour enregistrer des vidéos dans le système de fichiers local de l’appareil périphérique en utilisant un récepteur de fichiers dans votre graphe multimédia. Le récepteur de fichiers génère des fichiers [MP4](https://developer.mozilla.org/docs/Web/Media/Formats/Containers#MP4) et vous pouvez utiliser l’élément [&lt;vidéo&gt;](https://developer.mozilla.org/docs/Web/HTML/Element/video) HTML5 pour lire ce contenu. 

## <a name="next-steps"></a>Étapes suivantes

[Azure IoT Edge](../../iot-edge/index.yml)
<!--
## Next steps

[Playback recording](playback-recording-how-to.md)
-->
