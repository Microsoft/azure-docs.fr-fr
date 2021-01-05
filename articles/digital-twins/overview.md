---
title: Qu’est-ce qu’Azure Digital Twins ?
titleSuffix: Azure Digital Twins
description: Présentation des opérations qu’il est possible de réaliser avec Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 3/12/2020
ms.topic: overview
ms.service: digital-twins
ms.openlocfilehash: 49b7bae1a0cc224b92f292b891fae210f2cffa4e
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97400605"
---
# <a name="what-is-azure-digital-twins"></a>Qu’est-ce qu’Azure Digital Twins ?

**Azure Digital Twins** est une offre de plateforme en tant que service (PaaS) qui permet la création de graphes de connaissances basés sur des modèles numériques d’environnements entiers. Il peut s’agir de bâtiments, d’usines, de batteries de serveurs, de réseaux énergétiques, de chemins de fer, de stades, etc., voire de villes entières. Ces modèles numériques peuvent être utilisés pour obtenir des insights qui permettent d’améliorer les produits, d’optimiser les opérations, de réduire les coûts et de fournir des expériences client exceptionnelles.

Tirez parti de votre expertise sur Azure Digital Twins pour créer des solutions personnalisées et connectées qui :
* Modélisent tous les environnements et donnent vie aux jumeaux numériques de manière scalable et sécurisée
* Connectent des ressources telles que des appareils IoT et des systèmes d’entreprise existants
* Utilisent un système d’événements robuste pour créer une logique métier dynamique et un traitement des données
* S’intègrent aux services Azure (données, analytiques et IA) pour vous aider à se souvenir du passé et à prédire l’avenir

## <a name="azure-digital-twins-capabilities"></a>Fonctionnalités Azure Digital Twins

Voici un résumé des fonctionnalités fournies par Azure Digital Twins.

### <a name="open-modeling-language"></a>Langage de modélisation ouvert

Dans Azure Digital Twins, vous définissez les entités numériques qui représentent les personnes, les lieux et les objets de votre environnement physique à l’aide de types de jumeaux personnalisés appelés [**modèles**](concepts-models.md). 

Vous pouvez voir ces définitions de modèle comme un vocabulaire spécialisé décrivant votre activité. Pour une solution de gestion de bâtiments par exemple, vous pouvez définir des modèles tels que « immeuble », « étage » et « ascenseur ». Vous pouvez ensuite créer des **jumeaux numériques** en fonction de ces modèles pour représenter votre environnement spécifique.

Les modèles sont définis dans un langage de type JSON appelé [DTDL (Digital Twins Definition Language)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md). Ils décrivent les jumeaux par rapport à leurs propriétés d’état, événements de télémétrie, commandes, composants et relations.
* Les modèles définissent les **relations**  sémantiques entre vos entités afin que vous puissiez connecter vos jumeaux dans un graphe de connaissances qui reflète leurs interactions. Vous pouvez considérer les modèles comme des noms dans une description de votre monde, et les relations comme des verbes.
* Vous pouvez également spécialiser des jumeaux à l’aide de l’héritage de modèle. Un modèle peut hériter d’un autre modèle.

Le langage DTDL est utilisé pour les modèles de données dans d’autres services Azure IoT, notamment [IoT Plug-and-Play](../iot-pnp/overview-iot-plug-and-play.md) et [Time Series Insights](../time-series-insights/overview-what-is-tsi.md). Il vous permet de garder votre solution Azure Digital Twins connectée et compatible avec d’autres parties de l’écosystème Azure.

### <a name="live-execution-environment"></a>Environnement d’exécution en direct

Les modèles numériques dans Azure Digital Twins sont des représentations dynamiques et à jour du monde réel. À l’aide des relations dans vos modèles DTDL personnalisés, vous allez connecter les représentations dans un **graphe dynamique** représentant votre environnement.

Vous pouvez afficher une visualisation de votre graphe Azure Digital Twins à l’aide d’un exemple d’application, l’[**explorateur Azure Digital Twins**](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/).

Voici à quoi ressemble l’exemple de visualisation :

:::image type="content" source="media/includes/azure-digital-twins-explorer.png" alt-text="Capture d’écran de l’exemple d’application Explorateur Azure Digital Twins, montrant un graphe des nœuds représentant des jumeaux numériques" lightbox="media/includes/azure-digital-twins-explorer.png":::

Azure Digital Twins fournit un **système d’événements** riche pour maintenir ce graphe à jour avec le traitement des données et la logique métier. Vous pouvez connecter des ressources de calcul externes, telles qu’[Azure Functions](../azure-functions/functions-overview.md), pour piloter le traitement des données de manière flexible et personnalisée.

Vous pouvez également extraire des insights à partir de l’environnement d’exécution dynamique, à l’aide de la puissance **API de requête** d’Azure Digital Twins. L’API vous permet d’interroger des conditions de recherche enrichies, notamment des valeurs de propriété, des relations, des propriétés de relation, des informations de modèle, etc. Vous pouvez également combiner des requêtes, en recueillant un large éventail d’insights sur votre environnement et en répondant aux questions personnalisées qui vous intéressent.

### <a name="input-from-iot-and-business-systems"></a>Entrée des systèmes d’entreprise et IoT

Pour maintenir l’environnement d’exécution dynamique d’Azure Digital Twins à jour avec le monde réel, vous pouvez utiliser [IoT Hub](../iot-hub/about-iot-hub.md) pour connecter votre solution à des appareils IoT et IoT Edge. Ces appareils managés par un hub sont représentés dans le cadre de votre graphe de jumeaux et fournissent les données qui déterminent votre modèle.

Vous pouvez créer un nouvel IoT Hub à cet effet avec Azure Digital Twins, ou connecter un IoT Hub existant avec les appareils qu’il gère déjà.

Vous pouvez également piloter Azure Digital Twins à partir d’autres sources de données, à l’aide de l’API REST ou de connecteurs vers d’autres services tels que [Logic Apps](../logic-apps/logic-apps-overview.md).

### <a name="output-to-tsi-storage-and-analytics"></a>Sortie vers Time Series Insights, Stockage et Analytics

Les données de votre modèle Azure Digital Twins peuvent être routées vers les services Azure en aval pour des analyses ou du stockage supplémentaires. Cela est possible via des **itinéraires d’événements**, qui utilisent [Event Hub](../event-hubs/event-hubs-about.md), [Event Grid](../event-grid/overview.md)ou [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) pour piloter les flux de données souhaités.

Voici quelques opérations que vous pouvez effectuer avec les itinéraires d’événements :
* Stockage des données Azure Digital Twins dans [Azure Data Lake](../storage/blobs/data-lake-storage-introduction.md)
* Analyse des données Azure Digital Twins avec [Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md), ou d’autres outils d’analytique données de Microsoft
* Intégration de workflows plus volumineux avec Logic Apps
* Connexion d’Azure Digital Twins à Time Series Insights pour suivre l’historique des séries chronologiques de chaque jumeau
* Alignement d’un modèle de série chronologique dans Time Series Insights avec une source dans Azure Digital Twins

Il s’agit d’une autre façon pour Azure Digital Twins de se connecter à une plus grande solution et de prendre en charge vos besoins personnalisés pour continuer à travailler avec ces insights.

## <a name="azure-digital-twins-in-a-solution-context"></a>Azure Digital Twins dans le contexte d’une solution

Azure Digital Twins est couramment utilisé en association avec d’autres services Azure dans le cadre d’une solution IoT de plus grande envergure. 

Une solution complète utilisant Azure Digital Twins peut contenir les éléments suivants :
* Instance de service Azure Digital Twins. Celle-ci stocke vos modèles de jumeau et votre graphe de jumeaux avec son état, et orchestre le traitement des événements.
* Une ou plusieurs applications clientes qui pilotent l’instance Azure Digital Twins en configurant des modèles, en créant une topologie et en extrayant des insights à partir du graphe de jumeaux.
* Une ou plusieurs ressources de calcul externes pour traiter les événements générés par Azure Digital Twins ou les sources de données connectées telles que les appareils. Une façon courante de fournir des ressources de calcul consiste à utiliser [Azure Functions](../azure-functions/functions-overview.md).
* Un IoT Hub pour fournir des fonctionnalités de gestion des appareils et de flux de données IoT.
* Services en aval pour gérer des tâches telles que l’intégration de workflow (par exemple, [Logic Apps](../logic-apps/logic-apps-overview.md), le stockage froid, l’intégration de séries chronologiques ou l’analytique).

Le schéma suivant montre où Azure Digital Twins se trouve dans le contexte d’une solution Azure IoT plus volumineuse.

:::image type="content" source="media/overview/solution-context.png" alt-text="Schéma montrant des sources d’entrée, des services de sortie et une communication bidirectionnelle avec les applications clientes et les ressources de calcul externes." border="false" lightbox="media/overview/solution-context.png":::

## <a name="service-limits"></a>Limites du service

Pour obtenir la liste des limites Azure Digital Twins, consultez [*Référence : limites de service*](reference-service-limits.md).

## <a name="next-steps"></a>Étapes suivantes

Si vous avez utilisé la première préversion d’Azure Digital Twins (octobre 2018), découvrez ce qui a changé : [*Vue d’ensemble : Différences par rapport à la première version*](overview-differences.md).

Pour savoir comment utiliser Azure Digital Twins, vous pouvez également consulter le guide de démarrage rapide suivant : [*Démarrage rapide : Explorer un exemple de scénario*](quickstart-adt-explorer.md).