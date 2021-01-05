---
title: Fichier include
description: Fichier include
services: digital-twins
ms.service: digital-twins
ms.topic: include
ms.date: 07/09/2020
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.custom: include file
ms.openlocfilehash: e584b6eff16636f0657c586f6c630dbf8bbb99b2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96027423"
---
Ce qui suit résume les limites clés dans Azure Time Series Insights Gen1.

### <a name="sku-ingress-rates-and-capacities"></a>Capacités et débits d’entrée de la référence (SKU)

Les vitesses et capacités d’entrée des références (SKU) S1 et S2 offrent une certaine souplesse lors de la configuration d’un nouvel environnement Azure Time Series Insights. La capacité de votre référence SKU indique le taux d’entrée quotidien en fonction du nombre d’événements ou d’octets stockés, selon le cas. Notez que les entrées sont mesurées *par minute* et que la **limitation** est appliquée à l’aide de l’algorithme du seau à jetons. L’entrée est mesurée en blocs de 1 Ko. Par exemple, un événement réel de 0,8 Ko représente un événement, tandis qu’un événement de 2,6 Ko est compté comme trois événements.

| Capacité de la référence (SKU) S1 | Débit d’entrée | Capacité de stockage maximale
| --- | --- | --- |
| 1 | 1 Go (1 millions d’événements) par jour | 30 Go (30 millions d’événements) par mois |
| 10 | 10 Go (10 millions d’événements) par jour | 300 Go (300 millions d’événements) par mois |

| Capacité de la référence (SKU) S2 | Débit d’entrée | Capacité de stockage maximale
| --- | --- | --- |
| 1 | 10 Go (10 millions d’événements) par jour | 300 Go (300 millions d’événements) par mois |
| 10 | 100 Go (100 millions d’événements) par jour | 3 To (3 milliards d’événements) par mois |

> [!NOTE]
> Les capacités sont mises à l’échelle de façon linéaire. Par conséquent, une référence SKU S1 avec la capacité 2 prend en charge 2 Go (2 millions) d’événements par débit d’entrée par jour et 60 Go (60 millions d’événements) par mois.

Les environnements de référence (SKU) S2 prennent en charge beaucoup plus d’événements par mois et ont une capacité d’entrée sensiblement supérieure.

| SKU  | Nombre d’événements par mois  | Nombre d’événements par minute | Taille d’événement par minute  |
|---------|---------|---------|---------|---------|
| S1     |   30 millions   |  720    |  720 Ko   |
 |S2     |   300 millions   | 7 200   | 7 200 Ko  |

### <a name="property-limits"></a>Limites de propriétés

Les limites de propriétés Gen1 dépendent de l’environnement de référence (SKU) sélectionné. Les propriétés d’événements fournies ont des colonnes JSON, CSV et de graphique qui peuvent être affichées dans [l’Explorateur Azure Time Series Insights](../articles/time-series-insights/time-series-quickstart.md).

| SKU | Propriétés maximales |
| --- | --- |
| S1 | 600 propriétés (colonnes) |
| S2 | 800 propriétés (colonnes) |

### <a name="event-sources"></a>Sources d’événement

Au maximum deux sources d’événements par instance sont prise en charge.

* Découvrez comment [ajouter une source Event Hub](../articles/time-series-insights/how-to-ingest-data-event-hub.md).
* Configurez [une source de hub IoT](../articles/time-series-insights/how-to-ingest-data-iot-hub.md).

### <a name="api-limits"></a>Limites de l’API

Les limites de l’API REST pour Azure Time Series Insights Gen1 sont spécifiées dans la [documentation de référence sur l’API REST](/rest/api/time-series-insights/dataaccess(preview)/query/getavailability).