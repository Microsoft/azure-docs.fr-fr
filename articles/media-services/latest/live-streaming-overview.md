---
title: Vue d’ensemble du streaming en direct avec Azure Media Services v3 | Microsoft Docs
description: Cet article offre une vue d’ensemble du streaming en direct à l’aide d’Azure Media Services v3.
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
ms.openlocfilehash: dc08e23b6ef040ba5833ffa97f71d8cd1da3bb8a
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92019798"
---
# <a name="live-streaming-with-azure-media-services-v3"></a>Streaming en direct avec Azure Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Azure Media Services vous permet de transmettre des événements en direct auprès de vos clients dans le cloud Azure. Pour cela, vous avez besoin des éléments suivants :  

- Une caméra, pour capturer l’événement en direct.<br/>Pour obtenir des idées de configuration, consultez [Simple and portable event video gear setup]( https://link.medium.com/KNTtiN6IeT).

    Si vous n’avez pas accès à une caméra, des outils tels que [Telestream Wirecast](https://www.telestream.net/wirecast/overview.htm) peuvent être utilisés pour générer un flux en direct à partir d’un fichier vidéo.
- Un encodeur vidéo live, pour convertir les signaux de la caméra (ou d’un autre appareil, comme un ordinateur portable) en flux de contribution qui sera ensuite envoyé à Media Services. Le flux de contribution peut inclure des signaux de publicité, tels que les marqueurs SCTE-35.<br/>Pour obtenir une liste d’encodeurs de streaming live recommandés, consultez [Encodeurs de streaming live](recommended-on-premises-live-encoders.md). Consultez également ce billet de blog : [Live streaming production with OBS](https://link.medium.com/ttuwHpaJeT).
- Des composants dans Media Services, pour ingérer, prévisualiser, empaqueter, enregistrer, chiffrer et diffuser l’événement en direct auprès de vos clients, ou dans un CDN en vue d’une diffusion ultérieure.

Pour les clients qui souhaitent distribuer du contenu à un large public sur Internet, il est conseillé d’activer le CDN sur le [point de terminaison de streaming](streaming-endpoint-concept.md).

Cet article offre une vue d’ensemble et des conseils relatifs au streaming en direct avec Media Services ainsi que des liens vers d’autres articles pertinents.
 
> [!NOTE]
> Vous pouvez utiliser le [portail Azure](https://portal.azure.com/) pour gérer les [événements en direct](live-events-outputs-concept.md) v3, voir des [actifs multimédias](assets-concept.md) v3 et obtenir des informations sur l’accès aux API. Pour toutes les autres tâches de gestion (par exemple les transformations et travaux), utilisez l’[API REST](/rest/api/media/), l’[interface de ligne de commande](/cli/azure/ams) ou l’un des [SDK](media-services-apis-overview.md#sdks) pris en charge.

## <a name="dynamic-packaging-and-delivery"></a>Empaquetage dynamique et distribution

Avec Media Services, vous pouvez tirer parti de l’[empaquetage dynamique](dynamic-packaging-overview.md), qui vous permet de prévisualiser et de diffuser vos streams en direct aux [formats MPEG DASH, HLS et Smooth Streaming](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming) à partir du flux de contribution envoyé au service. Vos clients peuvent alors lire le flux en direct au moyen de n’importe quel lecteur compatible avec HLS, DASH ou Smooth Streaming. Vous pouvez utiliser le lecteur multimédia [Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) dans vos applications web ou mobiles afin de transmettre votre flux dans un de ces protocoles.

## <a name="dynamic-encryption"></a>Chiffrement dynamique

Le chiffrement dynamique permet de chiffrer dynamiquement votre contenu en direct ou à la demande avec AES-128 ou un des trois systèmes principaux de gestion des droits numériques (DRM) : Microsoft PlayReady, Google Widevine et Apple FairPlay. Media Services fournit également un service de distribution de clés AES et licences (PlayReady, Widevine et FairPlay) DRM aux clients autorisés. Pour plus d’informations, consultez [Chiffrement dynamique](content-protection-overview.md).

> [!NOTE]
> Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="dynamic-filtering"></a>Filtrage dynamique

Le filtrage dynamique permet de contrôler le nombre de pistes, de formats, de vitesses de transmission et de fenêtres de temps de présentation envoyés aux lecteurs. Pour plus d’informations, consultez [Filtres et manifestes dynamiques](filters-dynamic-manifest-overview.md).

## <a name="live-event-types"></a>Types d’événements en direct

Les [événements en direct](/rest/api/media/liveevents) sont chargés de la réception et du traitement des flux vidéo en direct. Un événement en direct peut être défini sur *Pass-through* (un encodeur live local envoie un flux à vitesse de transmission multiple) ou sur *Live Encoding* (un encodeur live local envoie un flux à vitesse de transmission unique). Pour plus d’informations sur le streaming en direct dans Media Services v3, consultez [Événements en direct et sorties en direct](live-events-outputs-concept.md).

### <a name="pass-through"></a>Requête directe

![Diagramme montrant comment les flux vidéo et audio d’un événement en direct Pass-through sont ingérés et traités.](./media/live-streaming/pass-through.svg)

Quand vous utilisez l’**événement en direct** de type pass-through, vous chargez l’encodeur live local de générer un flux vidéo à vitesse de transmission multiple et d’envoyer ce flux comme flux de contribution à l’événement en direct (à l’aide du protocole RTMP ou MP4 fragmenté d’entrée). L’événement en direct envoie ensuite les flux vidéo entrants au packager dynamique (point de terminaison de streaming) sans transcodage supplémentaire. Une transmission LiveEvent est optimisée pour les événements en direct de longue durée ou le streaming en direct linéaire sans interruption (24 heures sur 24, 365 jours par an). 

### <a name="live-encoding"></a>Encodage en direct  

![encodage en temps réel](./media/live-streaming/live-encoding.svg)

Quand vous utilisez l’encodage cloud avec Media Services, vous configurez votre encodeur live local pour qu’il envoie un flux vidéo à une seule vitesse de transmission comme flux de contribution (jusqu’à 32 Mbit/s au total) à l’événement en direct (à l’aide du protocole RTMP ou MP4 fragmenté). L’événement en direct transcode le flux de débit unique entrant en [flux vidéo à différents débits](https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming) avec différentes résolutions pour améliorer la diffusion et la rendre disponible pour les appareils de lecture par le biais de protocoles standard du secteur tels que MPEG-DASH, Apple HTTP Live Streaming (HLS) et Microsoft Smooth Streaming. 

### <a name="live-transcription-preview"></a>Transcription en direct (préversion)

Transcription en direct est une fonctionnalité que vous pouvez utiliser avec les événements en direct qui sont des encodages à transfert direct ou en temps réel. Pour en savoir plus, consultez [Transcription en direct](live-transcription.md). Lorsque cette fonctionnalité est activée, le service utilise la fonctionnalité de [reconnaissance vocale](../../cognitive-services/speech-service/speech-to-text.md) de Cognitive Services pour transcrire les paroles du fichier audio entrant en texte. Ce texte, accompagné de fichiers vidéo et audio dans les protocoles MPEG-DASH et HLS, est ensuite mis à disposition pour diffusion.

> [!NOTE]
> Actuellement, la transcription en direct est disponible en tant que fonctionnalité d’évaluation dans la région USA Ouest 2.

## <a name="live-streaming-workflow"></a>Workflow de streaming en direct

Pour comprendre le flux de travail de streaming en direct dans Media Services v3, vous devez commencer par évaluer et comprendre les concepts suivants : 

- [Points de terminaison de streaming](streaming-endpoint-concept.md)
- [Événements en direct et sorties en direct](live-events-outputs-concept.md)
- [Localisateurs de Streaming](streaming-locators-concept.md)

### <a name="general-steps"></a>Étapes générales

1. Dans votre compte Media Services, vérifiez que le **point de terminaison de streaming** (origine) est en cours d’exécution. 
2. Créez un [événement en direct](live-events-outputs-concept.md). <br/>Lors de la création de l’événement, vous pouvez spécifier qu’il démarre automatiquement. Sinon, lancez-le dès que vous souhaitez commencer le streaming.<br/> Lorsque le démarrage automatique est défini sur true, l’événement en direct démarre juste après sa création. La facturation commence donc dès que son exécution démarre. Vous devez appeler explicitement la commande Stop sur la ressource de l’événement en direct pour arrêter toute facturation supplémentaire. Pour plus d’informations, consultez [États et facturation des événements en direct](live-event-states-billing.md).
3. Récupérez la ou les URL ingérées et configurez votre encodeur local afin qu’il utilise cette URL pour envoyer le flux de contribution.<br/>Voir [Encodeurs live recommandés](recommended-on-premises-live-encoders.md).
4. Récupérez l’URL d’aperçu et utilisez-la pour vérifier que l’entrée de l’encodeur est bien reçue.
5. Créez un objet **asset** (actif multimédia). 

    Chaque sortie en direct est associée à un actif multimédia, qu’elle utilise pour enregistrer la vidéo dans le conteneur de stockage d’objets blob Azure associé. 
6. Créez une **sortie en direct** et utilisez le nom de l’actif multimédia que vous avez créé afin que le flux puisse être archivé dans l’actif multimédia.

    Les sorties en direct démarrent dès leur création et s’arrêtent à leur suppression. Quand vous supprimez la sortie en direct, vous ne supprimez pas l’actif multimédia sous-jacent ni le contenu de celui-ci.
7. Créez un **localisateur de streaming** avec les [types intégrés de la stratégie de streaming](streaming-policy-concept.md).

    Pour publier la sortie en temps réel, vous devez créer un localisateur de streaming pour la ressource associée. 
8. Listez les chemins dans le **localisateur de streaming** pour récupérer les URL à utiliser (elles sont déterministes).
9. Récupérez le nom d’hôte du **point de terminaison de streaming** (origine) à partir duquel vous souhaitez effectuer le streaming.
10. Combinez l’URL de l’étape 8 avec le nom d’hôte de l’étape 9 pour obtenir l’URL complète.
11. Si vous ne souhaitez plus afficher votre **événement en direct**, arrêtez le streaming de l’événement et supprimez le **localisateur de streaming**.
12. Si vous avez terminé de diffuser en continu les événements et que vous voulez nettoyer les ressources configurées précédemment, suivez la procédure ci-dessous.

    * Arrêtez d’envoyer le flux à partir de l’encodeur.
    * Arrêtez l’événement en temps réel. Une fois l’événement en direct arrêté, aucuns frais ne sont encourus. Lorsque vous devez le redémarrer, il possède la même URL de réception. Vous n’avez donc pas besoin de reconfigurer votre encodeur.
    * Vous pouvez arrêter votre point de terminaison de diffusion en continu, sauf si vous souhaitez continuer à fournir l’archive de votre événement en direct en tant que flux à la demande. Si l’événement en direct est dans l’état Arrêté, aucun frais n’est encouru.

La ressource sur laquelle la sortie dynamique est archivée devient automatiquement une ressource à la demande lorsque la sortie dynamique est supprimée. Vous devez supprimer toutes les sorties dynamiques avant de pouvoir arrêter un événement en direct. Vous pouvez éventuellement utiliser un indicateur [removeOutputsOnStop](/rest/api/media/liveevents/stop#request-body) pour supprimer automatiquement les sorties dynamiques à l’arrêt. 

> [!TIP]
> Consultez le [tutoriel sur le streaming en direct](stream-live-tutorial-with-api.md), l’article examine le code qui implémente les étapes décrites ci-dessus.

## <a name="other-important-articles"></a>Autres articles importants

- [Encodeurs live recommandés](recommended-on-premises-live-encoders.md)
- [Utiliser un magnétoscope numérique cloud](live-event-cloud-dvr.md)
- [Comparaison des fonctionnalités des types d’événements en direct](live-event-types-comparison.md)
- [États et facturation](live-event-states-billing.md)
- [Latence](live-event-latency.md)

## <a name="frequently-asked-questions"></a>Forum aux questions

Consultez l’article [Foire aux questions](frequently-asked-questions.md#live-streaming).

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="next-steps"></a>Étapes suivantes

* [Guide de démarrage rapide sur le streaming en direct](live-events-wirecast-quickstart.md)
* [Didacticiel sur le streaming en direct](stream-live-tutorial-with-api.md)
* [Conseils de migration pour le passage de Media Services v2 à Media Services v3](migrate-from-v2-to-v3.md)