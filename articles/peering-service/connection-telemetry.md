---
title: Télémétrie des connexions à Azure Peering Service
description: Découvrez la télémétrie des connexions à Microsoft Azure Peering Service
services: peering-service
author: derekolo
ms.service: peering-service
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.date: 05/18/2020
ms.author: derekol
ms.openlocfilehash: 0e54b20cc6aefa69226c24f557ae02166dc4b4b6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84871299"
---
# <a name="peering-service-connection-telemetry"></a>Télémétrie des connexions Peering Service

La télémétrie des connexions fournit des insights collectés au sujet de la connectivité entre l’endroit où se trouve le client et le réseau Microsoft. Les clients peuvent obtenir une télémétrie des connexions à Azure Peering Service en inscrivant la connexion dans le portail Azure. Cette fonctionnalité offre une sécurité de préfixe et des Insights sur les performances du réseau.


La télémétrie des connexions se compose des étendues suivantes :

### <a name="latency-measurement"></a>Mesure de la latence

 La latence est mesurée du client au PoP Microsoft Edge pour les préfixes inscrits dans le Peering Service.

### <a name="route-prefix-monitoring-and-protection"></a>Surveillance et protection des préfixes d’itinéraire

Les chemins de routage sont surveillés pour toute activité suspecte qui est ensuite capturée dans des journaux d’événements. Par exemple, des journaux d’événements sont créés pour certains de ces facteurs :

- Détournements de préfixe
- Retrait de préfixe
- Fuites d’itinéraires

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la connexion Peering Service, consultez [Connexion Peering Service](connection.md).
- Pour intégrer une connexion Peering Service, consultez [intégration d’un modèle de Peering Service](onboarding-model.md).
- Pour mesurer les données de télémétrie, consultez [Mesurer les données de télémétrie des connexions](measure-connection-telemetry.md).
