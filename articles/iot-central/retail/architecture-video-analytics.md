---
title: Détection de mouvement et d’objet de l’analytique vidéo Azure IoT Central | Microsoft Docs
description: Apprenez à créer une application IoT Central à l’aide du modèle d’application d’analytique vidéo pour la détection d’objets et de mouvements dans IoT Central. Ce modèle utilise l’analytique vidéo en direct et des caméras connectées.
author: KishorIoT
ms.author: nandab
ms.date: 07/27/2020
ms.topic: conceptual
ms.service: iot-central
ms.subservice: iot-central-retail
services: iot-central
ms.openlocfilehash: 48808f762536390287bae40e8af3849da20b81c2
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91874286"
---
# <a name="video-analytics---object-and-motion-detection-application-architecture"></a>Architecture de l’application Analytique vidéo – Détection d’objets et de mouvements

Le modèle d’application **Analytique vidéo – Détection d’objets et de mouvements** vous permet de créer des solutions IoT, notamment des capacités d’analytique vidéo en direct.

:::image type="content" source="media/architecture-video-analytics/architecture.png" alt-text="Diagramme de la vue d’ensemble des composants d’analytique vidéo pour la détection d’objets et de mouvements.":::

Les principaux composants de la solution d’analytique vidéo sont les suivants :

## <a name="live-video-analytics-lva"></a>Analytique vidéo en direct (LVA)

LVA fournit une plateforme vous permettant de créer des applications vidéo intelligentes qui s’étendent à la périphérie et au cloud. La plateforme vous permet de créer des applications vidéo intelligentes qui s’étendent à la périphérie et au cloud. La plateforme offre la possibilité de capturer, d’enregistrer, d’analyser des vidéos en direct et de publier les résultats, tels qu’une analytique vidéo, dans les services Azure. Les services Azure peuvent s’exécuter dans le cloud ou à la périphérie. La plateforme peut être utilisée pour améliorer les solutions IoT avec analyse vidéo.

Pour plus d’informations, consultez [Live Video Analytics](https://github.com/Azure/live-video-analytics) sur GitHub.

## <a name="iot-edge-lva-gateway-module"></a>Module de passerelle LVA IoT Edge

Le module de passerelle LVA IoT Edge instancie les caméras en tant que nouveaux appareils et les connecte directement à IoT Central à l’aide du Kit de développement logiciel (SDK) du client d’appareil IoT.

Dans cette implémentation de référence, les appareils se connectent à la solution à l’aide de clés symétriques à partir de la périphérie. Pour plus d’informations sur la connectivité des appareils, consultez [Se connecter à Azure IoT Central](../core/concepts-get-connected.md).

## <a name="media-graph"></a>Graphe multimédia

Le graphe multimédia vous permet de définir l’emplacement à partir duquel capturer le média, comment le traiter et où livrer les résultats. Pour configurer le graphe multimédia, vous devez connecter des composants ou des nœuds de la manière souhaitée. Pour plus d’informations, consultez la [page relative au graphe multimédia](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph) sur GitHub.

## <a name="next-steps"></a>Étapes suivantes

L’étape suivante suggérée consiste à découvrir [comment déployer une application IoT Central à l’aide du modèle d’application d’analytique vidéo pour la détection d’objets et de mouvements](tutorial-video-analytics-deploy.md).
