---
title: Limitations de débit de l’ingestion de diffusion en continu – Azure Time Series Insights Gen2 | Microsoft Docs
description: Découvrez les limites de débit d’entrée dans Azure Time Series Insights Gen2.
author: lyrana
ms.author: lyhughes
manager: dpalled
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 10/26/2020
ms.custom: seodec18
ms.openlocfilehash: c8be18049e6ae74a198f5885a46b70df581e0cd7
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96187457"
---
# <a name="streaming-ingestion-throughput-limits"></a>Limites de débit d’ingestion de diffusion en continu

Voici les limitations d’entrée des données de diffusion en continu d’Azure Time Series Insights Gen2.

> [!TIP]
> Pour obtenir une liste complète de toutes les limites, consultez [Planifier votre environnement Azure Time Series Insights Gen2](./how-to-plan-your-environment.md#review-azure-time-series-insights-gen2-limits).

## <a name="per-environment-limitations"></a>Limitations par environnement

En général, les taux d’entrée sont considérés comme le facteur du nombre d’appareils qui se trouvent dans votre organisation, de la fréquence d’émission des événements et de la taille de chaque événement :

* **Nombre d’appareils** × **Fréquence d’émission d’événements** × **Taille de chaque événement**.

Par défaut, Azure Time Series Insights Gen2 peut ingérer des données entrantes à un débit allant **jusqu’à 1 mégaoctet par seconde (Mbit/s) par environnement Azure Time Series Insights Gen2**. Il existe des limitations supplémentaires [par partition de hub](./concepts-streaming-ingress-throughput-limits.md#hub-partitions-and-per-partition-limits).

> [!TIP]
>
> * Une prise en charge de l’environnement pour des vitesses d’ingestion jusqu’à 8 Mbits/s peut être fournie sur demande.
> * Contactez-nous si vous avez besoin d’un débit supérieur en soumettant un ticket de support via le portail Azure.

* **Exemple 1 :**

    Contoso Shipping est doté de 100 000 appareils qui émettent un événement trois fois par minute. La taille d’un événement est de 200 octets. Un IoT Hub comportant quatre partitions est utilisé en tant que source d’événements Azure Time Series Insights Gen2.

  * Le taux d’ingestion de leur environnement Azure Time Series Insights Gen2 est le suivant : **100 000 appareils * 200 octets/événement * (3/60 événements/s) = 1 Mbit/s**.
    * En supposant que les partitions sont équilibrées, le taux d’ingestion par partition est de 0,25 Mbits/s.
    * Le débit d’ingestion de Contoso Shipping s’inscrit dans les limitations de mise à l’échelle.

* **Exemple 2 :**

    Contoso Fleet Analytics compte 40 000 appareils qui émettent un événement à chaque seconde. Un Event Hub comportant deux partitions est utilisé comme source d’événements Azure Time Series Insights Gen2. La taille d’un événement est de 200 octets.

  * Le taux d’ingestion de l’environnement est le suivant : **40 000 appareils * 200 octets/événement * 1 événement/s = 8 Mbit/s**.
    * En supposant que les partitions sont équilibrées, leur taux par partition est de 4 Mbits/s.
    * Le taux d'ingestion de Contoso Fleet Analytics dépasse les limites de l'environnement et des partitions. Une demande peut être adressée à Azure Time Series Insights Gen2 via le portail Azure pour augmenter le taux d’ingestion de l’environnement, et un Event Hub comportant davantage de partitions peut être créé afin de respecter les limites.

## <a name="hub-partitions-and-per-partition-limits"></a>Partitions de hub et limites par partition

Lors de la planification de votre environnement Azure Time Series Insights Gen2, il est important de tenir compte de la configuration des sources de l’événement que vous comptez connecter à Azure Time Series Insights Gen2. IoT Hub et Event Hubs utilisent des partitions afin d’activer la mise à l’échelle horizontale pour le traitement des événements.

Une *partition* est une séquence ordonnée d’événements conservée dans un hub. Le nombre de partitions est défini lors de la phase de création du hub et ne peut pas être modifié.

Pour les meilleures pratiques de partitionnement d’Event Hubs, voir [De combien de partitions ai-je besoin ?](../event-hubs/event-hubs-faq.md#how-many-partitions-do-i-need)

> [!NOTE]
> La plupart des IoT Hubs utilisés avec Azure Time Series Insights Gen2 n’ont besoin que de quatre partitions.

Que vous créiez un hub pour votre environnement Azure Time Series Insights Gen2 ou que vous utilisiez un hub existant, vous devez calculer votre débit d’ingestion par partition pour déterminer s’il s’inscrit dans les limites.

Azure Time Series Insights Gen2 a actuellement une **limite générale par partition de 0,5 Mbits/s**.

### <a name="iot-hub-specific-considerations"></a>Considérations spécifiques à IoT Hub

Lors de la création d’un appareil dans IoT Hub, cet appareil est définitivement attribué à une partition. IoT Hub est ainsi en mesure de garantir l’ordre des événements (puisque l’affectation ne change jamais).

Une affectation de partition fixe a également une incidence sur les instances Azure Time Series Insights Gen2 qui ingèrent les données envoyées à partir de l’IoT Hub en aval. Lorsque des messages provenant de plusieurs appareils sont transférés au hub à l’aide du même ID de passerelle, ils peuvent arriver dans la même partition au même moment, ce qui peut entraîner un dépassement des limites d’échelle par partition.

**Impact** :

* Si une partition unique subit un taux d’ingestion soutenu supérieur à la limite, il est possible qu’Azure Time Series Insights Gen2 ne synchronise pas toutes les données de télémétrie des appareils avant que la période de conservation des données IoT Hub soit dépassée. Par conséquent, des données envoyées peuvent se perdre si les limites d’ingestion sont régulièrement dépassées.

Pour limiter ce risque, nous vous recommandons d’adopter les meilleures pratiques suivantes :

* Calculez vos taux d’ingestion par environnement et par partition avant de déployer votre solution.
* Assurez-vous que la charge de vos appareils IoT Hub est équilibrée dans toute la mesure du possible.

> [!IMPORTANT]
> Pour les environnements qui utilisent IoT Hub comme source d’événements, calculez le débit d’ingestion en vous appuyant sur le nombre d’appareils hub en cours d’utilisation pour que le débit tombe bien sous la limite de 0,5 Mbit/s par partition.
>
> * Même si plusieurs événements se produisent simultanément, la limite n’est pas dépassée.

  ![Diagramme de partitions IoT Hub](media/concepts-ingress-overview/iot-hub-partiton-diagram.png)

Pour en savoir plus sur l’optimisation du débit et des partitions du hub, reportez-vous aux ressources suivantes :

* [Mise à l’échelle d’IoT Hub](../iot-hub/iot-hub-scaling.md)
* [Mise à l’échelle d’Event Hub](../event-hubs/event-hubs-scalability.md#throughput-units)
* [Partitions Event Hub](../event-hubs/event-hubs-features.md#partitions)

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur le [stockage](./concepts-storage.md) des données