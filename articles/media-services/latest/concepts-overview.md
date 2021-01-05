---
title: Terminologie et concepts de Media Services
titleSuffix: Azure Media Services
description: En savoir plus sur la terminologie et les concepts d’Azure Media Services.
services: media-servicesgit
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: 897df7f07a349457c49929b69993c75345f52241
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92017809"
---
# <a name="media-services-terminology-and-concepts"></a>Terminologie et concepts de Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Cet article présente brièvement les concepts et la terminologie d’Azure Media Services. Il contient également des liens qui renvoient à des articles présentant une explication détaillée des concepts et des fonctionnalités de Media Services v3.

Il est recommandé d’examiner les concepts fondamentaux décrits dans ces rubriques avant de commencer le développement.

> [!NOTE]
> Actuellement, vous pouvez utiliser le [Portail Azure](https://portal.azure.com/) pour gérer les [événements en direct](live-events-outputs-concept.md) Media Services v3, afficher (mais pas gérer) les [éléments multimédias](assets-concept.md) v3 et [obtenir des informations sur l’accès aux API](./access-api-howto.md).
> Pour toutes les autres tâches de gestion (par exemple, les [transformations et travaux](transforms-jobs-concept.md) et la [protection de contenu](content-protection-overview.md)), utilisez l’[API REST](/rest/api/media/accountfilters), l’interface [CLI](/cli/azure/ams) ou l’un des [kits de développement logiciel (SDK)](media-services-apis-overview.md#sdks) pris en charge.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="media-services-v3-terminology"></a>Terminologie Media Services v3

|Terme|Description|
|---|---|
|Événement en direct|Un **événement en direct** représente un pipeline d’ingestion, de transcodage (éventuellement) et d’empaquetage des streams en direct des fichiers vidéo et audio et des métadonnées en temps réel.<br/><br/>Pour les clients migrant depuis les API Media Services v2, **l’événement en direct** remplace l’entité **Channel** dans la version 2. Pour plus d’informations, consultez l’article [Conseils de migration pour le passage de Media Services v2 à Media Services v3](migrate-from-v2-to-v3.md).|
|Point de terminaison de streaming/empaquetage/origine|Un **point de terminaison de streaming** représente un empaquetage dynamique (juste-à-temps) et un service d’origine qui permet de transmettre votre contenu en direct et à la demande directement à une application de lecteur cliente. Il utilise un des protocoles courants de diffusion multimédia en continu (HLS ou DASH). En outre, le **point de terminaison de streaming** offre un chiffrement dynamique (juste-à-temps) aux systèmes de gestion des droits numériques (DRM) leaders sur le marché.<br/><br/>Dans le secteur de la diffusion multimédia en continu, ce service est communément appelé **packager** ou **origine**.  Autres termes courants de ce secteur pour cette capacité : JITP (packager juste-à-temps) ou JITE (chiffrement juste-à-temps).

## <a name="media-services-v3-concepts"></a>Concepts Media Services v3

|Concepts|Description|Liens|
|---|---|---|
|Éléments multimédias et téléchargement de contenu|Pour commencer à gérer, chiffrer, coder, analyser et diffuser en continu du contenu multimédia dans Azure, vous devez créer un compte Media Services, puis charger vos fichiers numériques dans **Éléments multimédias**.|[Chargement et stockage cloud](storage-account-concept.md)<br/><br/>[Concept des éléments multimédias](assets-concept.md)|
|Encodage du contenu|Une fois que vous avez chargé vos fichiers multimédias numériques haute définition dans Éléments multimédias, vous pouvez les encoder dans des formats pouvant être lus sur un large choix de navigateurs et d’appareils. <br/><br/>Pour encoder du contenu avec Media Services v3, vous devez créer des **transformations** et des **travaux**.|[Transformations et travaux](transforms-jobs-concept.md)<br/><br/>[Encodage avec Media Services](encoding-concept.md)|
|Analyse du contenu (Video Indexer)|Media Services v3 vous permet d’extraire les insights de vos fichiers vidéo et audio à l’aide de présélections Media Services v3. Pour analyser votre contenu à l'aide de présélections Media Services v3, vous devez créer des **transformations** et des **travaux**.<br/><br/>Si vous souhaitez des informations plus détaillées, utilisez directement [Video Indexer](../video-indexer/index.yml).|[Analyse des fichiers vidéo et audio](analyzing-video-audio-files-concept.md)|
|Empaquetage et remise|Une fois que votre contenu est encodé, vous pouvez tirer parti de **l’empaquetage dynamique**. Dans Media Services, un **point de terminaison de streaming** est le service d’empaquetage dynamique utilisé pour distribuer du contenu multimédia aux lecteurs clients. Pour rendre les vidéos dans la ressource de sortie disponibles en lecture pour les clients, vous devez créer un **localisateur de streaming**, puis générer des URL de diffusion en continu. <br/><br/>Lors de la création du **localisateur de streaming**, vous devez spécifier une **stratégie de streaming** en plus du nom de l’élément multimédia. Les **stratégies de streaming** vous permettent de définir des protocoles de streaming et des options de chiffrement (le cas échéant) pour vos **localisateurs de streaming**. L’empaquetage dynamique est utilisé que le streaming de contenu soit effectué en direct ou à la demande. <br/><br/>Vous pouvez utiliser les **manifestes dynamiques** de Media Services pour diffuser en continu uniquement un rendu spécifique ou des sous-clips de votre vidéo.|[mise en package dynamique](dynamic-packaging-overview.md)<br/><br/>[Points de terminaison de streaming](streaming-endpoint-concept.md)<br/><br/>[Localisateurs de diffusion en continu](streaming-locators-concept.md)<br/><br/>[Stratégies de diffusion en continu](streaming-policy-concept.md)<br/><br/>[Manifestes dynamiques](filters-dynamic-manifest-overview.md)<br/><br/>[Filtres](filters-concept.md)|
|Protection du contenu|Media Services vous permet de transmettre votre contenu en direct ou à la demande chiffré dynamiquement avec la norme Advanced Encryption Standard (AES-128) ou l’un des trois principaux systèmes DRM : Microsoft PlayReady, Google Widevine et Apple FairPlay. Media Services fournit également un service de distribution de clés AES et licences (PlayReady, Widevine et FairPlay) DRM aux clients autorisés. <br/><br/>Si vous spécifiez des options de chiffrement sur votre flux, créez la **stratégie de clé de contenu**, puis associez-la à votre **localisateur de streaming**. La **stratégie de clé de contenu** vous permet de configurer le mode de remise de la clé de contenu aux clients finals.<br/><br/> Essayez de réutiliser des stratégies chaque fois que les mêmes options sont nécessaires.| [Stratégies de clé de contenu](content-key-policy-concept.md)<br/><br/>[Protection du contenu](content-protection-overview.md)|
|Vidéo en flux continu|Media Services vous permet de transmettre des événements en direct auprès de vos clients dans le cloud Azure. Les **événements en direct** sont chargés de la réception et du traitement des flux vidéo en direct. Quand vous créez un **événement en direct**, un point de terminaison d’entrée est également créé. Vous pouvez utiliser ce point de terminaison pour envoyer un signal en direct à partir d’un encodeur à distance. Une fois que le flux transite dans **l’événement en direct**, vous pouvez commencer l’événement de streaming en créant un **élément multimédia**, une **sortie en direct** et un **localisateur de streaming**. La **sortie en direct** archive le flux dans **l’élément multimédia** et le met à la disposition des observateurs via le **point de terminaison de streaming**. Un événement en direct peut être défini sur *Pass-through* (un encodeur live local envoie un flux à vitesse de transmission multiple) ou sur *Live Encoding* (un encodeur live local envoie un flux à vitesse de transmission unique). |[Vue d’ensemble du streaming en direct](live-streaming-overview.md)<br/><br/>[Événements en direct et sorties en direct](live-events-outputs-concept.md)|
|Surveillance avec Event Grid|Pour afficher la progression du travail, utilisez **Event Grid**. Media Services émet aussi les types d’événements en direct. Avec Event Grid, vos applications peuvent écouter les événements de presque tous les services Azure ou de toute source personnalisée, et y réagir. |[Traitement des événements Event Grid](reacting-to-media-services-events.md)<br/><br/>[Schémas](media-services-event-schemas.md)|
|Surveillance avec Azure Monitor|Surveillez les métriques et les journaux de diagnostic qui vous aident à comprendre le fonctionnement de vos applications avec Azure Monitor.|[Métriques et journaux de diagnostic](media-services-metrics-diagnostic-logs.md)<br/><br/>[Schémas des journaux de diagnostic](media-services-diagnostic-logs-schema.md)|
|Clients Player|Vous pouvez utiliser Lecteur multimédia Azure pour lire des contenus multimédias diffusés en continu par Media Services sur une grande variété de navigateurs et d’appareils. Lecteur multimédia Azure exploite les normes du secteur, telles que HTML5, Media Source Extensions (MSE) et Encrypted Media Extensions (EME), pour offrir une expérience enrichie de diffusion en continu adaptative. |[Présentation du Lecteur multimédia Azure](use-azure-media-player.md)|

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="next-steps"></a>Étapes suivantes

* [Encoder le fichier distant et diffuser la vidéo – REST](stream-files-tutorial-with-rest.md)
* [Encoder le fichier chargé et diffuser la vidéo – REST](stream-files-tutorial-with-api.md)
* [Diffuser en direct - .NET](stream-live-tutorial-with-api.md)
* [Analyser votre vidéo - .NET](analyze-videos-tutorial-with-api.md)
* [Chiffrement dynamique AES-128 - .NET](protect-with-aes128.md)
* [Chiffrer dynamiquement avec multi-DRM - .NET](protect-with-drm.md)