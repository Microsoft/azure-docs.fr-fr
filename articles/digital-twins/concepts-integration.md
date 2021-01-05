---
title: Intégration à d’autres services
titleSuffix: Azure Digital Twins
description: Comprendre les exigences d’entrée et de sortie lors du déploiement d’Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 3/16/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 118b02ab694d27dbe4e13cbfa1a617a56b052772
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92043066"
---
# <a name="integrate-azure-digital-twins-with-other-services"></a>Intégrer Azure Digital Twins avec d’autres services

Azure Digital Twins est généralement utilisé avec d’autres services pour créer des solutions flexibles et connectées qui utilisent vos données de diverses manières.

À l’aide d’[**itinéraires d’événements**](concepts-route-events.md), Azure Digital Twins peut recevoir des données de services en amont, comme [IoT Hub](../iot-hub/about-iot-hub.md) ou [Logic Apps](../logic-apps/logic-apps-overview.md), qui sont utilisés pour fournir de la télémétrie et des notifications. 

Azure Digital Twins peut également acheminer des données vers des services en aval, comme [Azure Maps](../azure-maps/about-azure-maps.md) et [Time Series Insights](../time-series-insights/overview-what-is-tsi.md), à des fins de stockage, d’intégration de flux de travail, d’analytique et autres. 

## <a name="data-ingress"></a>Entrée de données

Azure Digital Twins peut être piloté par des données et des événements issus d’un service quel qu’il soit ([IoT Hub](../iot-hub/about-iot-hub.md), [Logic Apps](../logic-apps/logic-apps-overview.md)), votre propre service personnalisé, etc. Cela vous permet de collecter des données de télémétrie à partir d’appareils physiques dans votre environnement, puis de les traiter à l’aide du graphique Azure Digital Twins dans le cloud.

Au lieu d’un IoT Hub intégré en arrière-plan, Azure Digital Twins vous permet d’utiliser vos propres IoT Hub pour les utiliser avec le service. Vous pouvez utiliser un IoT Hub existant actuellement en production ou en déployer un nouveau à cet effet. Vous bénéficiez ainsi d’un accès complet à toutes les fonctionnalités de gestion d’appareils d’IoT Hub.

Pour ingérer des données issues d’une source dans Azure Digital Twins, utilisez une [**fonction Azure**](../azure-functions/functions-overview.md). Pour plus d’informations sur ce modèle, consultez [*Guide pratique : Ingérer la télémétrie depuis IoT Hub*](how-to-ingest-iot-hub-data.md) ou essayez-le dans le document Azure Digital Twins [*Tutoriel : Connecter une solution de bout en bout*](tutorial-end-to-end.md). 

Vous pouvez également découvrir comment connecter Azure Digital Twins à un déclencheur Logic Apps dans [*Guide pratique : Intégrer à Logic Apps*](how-to-integrate-logic-apps.md).

## <a name="data-egress-services"></a>Services de sortie des données

Azure Digital Twins peut envoyer des données à des **points de terminaison** connectés. Les points de terminaison pris en charge peuvent être :
* [Concentrateur d’événements](../event-hubs/event-hubs-about.md)
* [Event Grid](../event-grid/overview.md)
* [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md)

Les points de terminaison sont attachés au service Azure Digital Twins à l’aide d’API de gestion ou du portail Azure. Apprenez-en davantage sur la façon d’attacher un point de terminaison à Azure Digital Twins dans [*Guide pratique : Gérer les points de terminaison et les routes*](how-to-manage-routes-apis-cli.md).

Il existe de nombreux autres services vers lesquels vous pouvez finalement diriger vos données, tels que [Stockage Azure](../storage/common/storage-introduction.md), [Azure Maps](../azure-maps/about-azure-maps.md) ou [Time Series Insights](../time-series-insights/overview-what-is-tsi.md). Pour envoyer vos données à de tels services, attachez le service de destination à un point de terminaison.

Par exemple, si vous utilisez également Azure Maps et souhaitez mettre en corrélation l’emplacement avec votre [graphique de jumeau](concepts-twins-graph.md) Azure Digital Twins, vous pouvez utiliser Azure Functions avec Event Grid pour établir la communication entre tous les services au sein de votre déploiement. Pour en savoir plus, consultez [*Guide pratique : Utiliser Azure Digital Twins pour mettre à jour un plan intérieur Azure Maps*](how-to-integrate-maps.md)

Vous pouvez également apprendre à acheminer des données de la même façon vers Time Series Insights dans [*Guide pratique : Intégrer à Time Series Insights*](how-to-integrate-time-series-insights.md).

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur les points de terminaison et le routage des événements vers des services externes :
* [*Concepts : routage des événements Azure Digital Twins*](concepts-route-events.md)

Découvrez comment configurer Azure Digital Twins pour ingérer des données à partir d’IoT Hub :
* [*Guide pratique : Ingérer la télémétrie depuis IoT Hub*](how-to-ingest-iot-hub-data.md)