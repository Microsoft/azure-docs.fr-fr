---
title: Qu’est-ce que Live Video Analytics sur IoT Edge - Azure
description: Cette rubrique fournit une vue d’ensemble de Live Video Analytics sur IoT Edge. La plateforme offre les fonctionnalités que vous pouvez utiliser pour améliorer vos solutions IoT. Par exemple, capturez, enregistrez, analysez des vidéos en direct et publiez les résultats (vidéo et/ou analyses vidéo) dans les services Azure.
ms.topic: overview
ms.date: 05/27/2020
ms.openlocfilehash: b9b8726c5e8ad6850e05aeee48fccabee703080e
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "90904353"
---
# <a name="what-is-live-video-analytics-on-iot-edge-preview"></a>Qu’est-ce que Live Video Analytics sur IoT Edge ? (préversion)

Live Video Analytics sur IoT Edge fournit une plateforme pour créer des applications vidéo intelligentes qui s’étendent à la périphérie et au cloud. Cette plateforme offre la possibilité de capturer, d’enregistrer et d’analyser des vidéos en direct, ainsi que de publier les résultats (vidéo et/ou analytique vidéo) sur des services Azure (dans le cloud et/ou la périphérie). La plateforme peut être utilisée pour améliorer les solutions IoT avec analyse vidéo. Live Video Analytics sur la fonctionnalité IoT Edge peut être associé à d’autres modules Azure IoT Edge, tels que Stream Analytics sur IoT Edge, Cognitive Services sur IoT Edge, ainsi que les services Azure dans le cloud, tels que Media Services, Event Hub, Cognitive Services, etc. pour créer des applicables hybrides efficaces (par exemple, edge + cloud).

Live Video Analytics sur IoT Edge est conçue pour être une plateforme extensible. Elle vous permet de connecter différents modules de périphérie d’analyse vidéo (par exemple, des conteneurs Cognitive Services, des modules de périphérie personnalisés que vous avez générés avec des modèles Machine Learning open source ou des modèles personnalisés entraînés avec vos propres données) et de les utiliser pour analyser la vidéo en direct sans vous soucier de la complexité liée à la création et à l’exécution d’un pipeline vidéo en direct.

## <a name="accelerate-iot-solutions-development"></a>Accélérer le développement de solutions IoT 

Les solutions IoT qui combinent l’analytique vidéo avec des signaux d’autres capteurs IoT et/ou des données métier peuvent vous aider à automatiser ou semi-automatiser des décisions métier, et ainsi améliorer la productivité. Live Video Analytics sur IoT Edge vous permet de développer des solutions de ce type plus rapidement. Vous pouvez vous concentrer sur la création des modules d’analyse vidéo et de la logique propre à votre entreprise, et laisser le soin à la plateforme de masquer les complexités liées à la gestion et à l’exécution d’un pipeline vidéo.

Avec Live Video Analytics sur IoT Edge, vous pouvez continuer à utiliser vos [caméras de télésurveillance](https://en.wikipedia.org/wiki/Closed-circuit_television_camera) avec vos [systèmes de gestion vidéo existants](https://en.wikipedia.org/wiki/Video_management_system), et créer des applications d’analytique vidéo de manière indépendante. Vous pouvez utiliser Live Video Analytics sur IoT Edge conjointement avec les kits de ressources et les SDK Vision par ordinateur pour créer des solutions IoT de pointe. Le diagramme ci-dessous illustre ceci.

![Développez des solutions IoT avec Live Video Analytics sur IoT Edge](./media/overview/product-diagram.svg)

## <a name="supported-environments"></a>Environnements pris en charge

Les environnements Linux x86 -64 et ARM64 sont pris en charge.
> [!NOTE]
> La prise en charge des appareils Linux ARM64 est disponible dans les versions builds`1.0.4` et ultérieures.
> La prise en charge de l’exécution d’Azure IoT Edge Runtime sur des appareils ARM64 est incluse dans la [version préliminaire publique](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="get-started"></a>Bien démarrer

Lisez les articles conceptuels suivants, puis essayez un démarrage rapide pour exécuter la détection de mouvement sur un flux vidéo en direct.

### <a name="concepts"></a>Concepts

* [Graphe multimédia](media-graph-concept.md)
* [Enregistrement de vidéo](video-recording-concept.md)
* [Lecture de vidéo](video-playback-concept.md)
* [Enregistrement de vidéo continu](continuous-video-recording-concept.md)
* [Enregistrement de vidéo basé sur les événements](event-based-video-recording-concept.md)
* [Live Video Analytics sans enregistrement de vidéo](analyze-live-video-concept.md)

## <a name="next-steps"></a>Étapes suivantes

* Suivez le tutoriel [Démarrage rapide : Exécutez Live Video Analytics avec votre propre article ](use-your-model-quickstart.md)de modèle pour voir comment vous pouvez exécuter la détection de mouvement sur un flux vidéo en direct.
* Passer en revue la [terminologie](terminology.md)
* Découvrez [le matériel open source Live Video Analytics](https://github.com/Azure/live-video-analytics)

