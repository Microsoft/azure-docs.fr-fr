---
title: Mappages de régions d’Azure Monitor pour conteneurs
description: Cet article décrit les mappages de région pris en charge entre Azure Monitor pour conteneurs, l’espace de travail Log Analytics et les métriques personnalisées.
ms.topic: conceptual
ms.date: 09/22/2020
ms.custom: references_regions
ms.openlocfilehash: 9e3b8635c70dfdf33b0a062be80c948cd77923cc
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91272903"
---
# <a name="region-mappings-supported-by-azure-monitor-for-containers"></a>Mappages de régions pris en charge par Azure Monitor pour conteneurs

 Lorsque vous activez Azure Monitor pour conteneurs, seules certaines régions sont prises en charge pour la liaison d’un espace de travail Log Analytics et d’un cluster AKS, ainsi que pour la collecte de métriques personnalisées envoyées à Azure Monitor.

## <a name="log-analytics-workspace-supported-mappings"></a>Mappages pris en charge sur l’espace de travail Log Analytics

Les régions AKS prises en charge sont répertoriées dans [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service). L’espace de travail Log Analytics doit se trouver dans la même région, à l’exception des régions répertoriées dans le tableau suivant. Pour les mises à jour, consultez les [notes de publication AKS](https://github.com/Azure/AKS/releases).


|**Région du cluster AKS** | **Région de l’espace de travail Log Analytics** |
|-----------------------|------------------------------------|
|**Afrique** | |
|SouthAfricaNorth |WestEurope |
|SouthAfricaWest |WestEurope |
|**Australie** | |
|AustraliaCentral2 |AustraliaCentral |
|**Brésil** | |
|BrazilSouth | SouthCentralUS |
|**Canada** ||
|CanadaEast |CanadaCentral |
|**Europe** | |
|FranceSouth |FranceCentral |
|UKWest |RoyaumeUniSud |
|**Inde** | |
|SouthIndia |CentralIndia |
|WestIndia |CentralIndia |
|**Japon** | |
|JapanWest |JaponEst |
|**Corée** | |
|KoreaSouth |KoreaCentral |
|**États-Unis** | |
|WestCentralUS<sup>1</sup>|EastUS<sup>1</sup>|


<sup>1</sup> En raison de restrictions de capacité, la région n’est pas disponible lors de la création de ressources. Cela comprend les espaces de travail Log Analytics. Toutefois, les ressources liées préexistantes dans la région doivent continuer à fonctionner.

## <a name="custom-metrics-supported-regions"></a>Régions prises en charge su les métriques personnalisées

La collecte de métriques des pods et nœuds de cluster Azure Kubernetes Service (AKS) est prise en charge dans le cadre de la publication en tant que métriques personnalisées dans les [régions Azure](../platform/metrics-custom-overview.md#supported-regions) suivantes.

## <a name="next-steps"></a>Étapes suivantes

Pour commencer la surveillance de votre cluster AKS, consultez [Comment activer Azure Monitor pour les conteneurs](container-insights-onboard.md) pour comprendre les exigences et les méthodes disponibles pour activer la surveillance.  
