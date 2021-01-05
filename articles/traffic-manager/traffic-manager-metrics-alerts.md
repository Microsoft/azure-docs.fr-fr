---
title: Métriques et alertes dans Azure Traffic Manager
description: Dans cet article, découvrez les métriques et alertes disponibles pour Traffic Manager dans Azure.
services: traffic-manager
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/11/2018
ms.author: duau
ms.openlocfilehash: 997e2e97161e6db7cdca04c1b79f5149a2e4c409
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96003529"
---
# <a name="traffic-manager-metrics-and-alerts"></a>Métriques et alertes Traffic Manager

Traffic Manager vous offre un équilibrage de charge basé sur DNS qui inclut plusieurs méthodes de routage et des options de surveillance des points de terminaison. Cet article décrit les métriques et alertes associées disponibles pour les clients. 

## <a name="metrics-available-in-traffic-manager"></a>Métriques disponibles dans Traffic Manager 

Traffic Manager fournit les métriques suivantes, selon le profil, que les clients peuvent utiliser pour comprendre leur utilisation de Traffic Manager et connaître l’état de leurs points de terminaison sous ce profil.  

### <a name="queries-by-endpoint-returned"></a>Requêtes par point de terminaison renvoyé
Utilisez [cette métrique](../azure-monitor/platform/metrics-supported.md) pour afficher le nombre de requêtes traitées par un profil Traffic Manager sur une période spécifiée. Vous pouvez également afficher les mêmes informations au niveau d’un point de terminaison et déterminer ainsi le nombre de fois où un point de terminaison a été retourné dans les réponses à une requête à partir de Traffic Manager.

Dans l’exemple suivant, la figure 1 affiche toutes les réponses à une requête renvoyées par le profil Traffic Manager. 

  
![Vue agrégée de toutes les requêtes](./media/traffic-manager-metrics-alerts/traffic-manager-metrics-queries-aggregate-view.png)

*Figure 1 : Vue agrégée de toutes les requêtes*
  
La figure 2 affiche les mêmes informations, mais réparties cette fois par point de terminaison. Par conséquent, vous pouvez voir le volume des réponses aux requêtes dans lequel un point de terminaison spécifique a été retourné.

![Métriques Traffic Manager - mode fractionné du volume de requêtes par point de terminaison](./media/traffic-manager-metrics-alerts/traffic-manager-metrics-query-volume-per-endpoint.png)

*Figure 2 : me mode fractionné avec volume de requêtes indiqué par point de terminaison retourné*

## <a name="endpoint-status-by-endpoint"></a>État du point de terminaison par point de terminaison
Utilisez [cette métrique](../azure-monitor/platform/metrics-supported.md#microsoftnetworktrafficmanagerprofiles) pour évaluer l’état d’intégrité des points de terminaison dans le profil. Elle accepte deux valeurs :
 - utilisez **1** si le point de terminaison est activé.
 - utilisez **0** si le point de terminaison est arrêté.

Cette métrique peut être affichée comme une valeur d’agrégation représentant l’état de toutes les métriques (figure 3), ou elle peut être fractionnée (voir figure 4) pour afficher l’état de points de terminaison spécifiques. Dans le cas le premier cas, si le niveau d’agrégation est sélectionné comme **Avg**, la valeur de cette métrique est la moyenne arithmétique de l’état de tous les points de terminaison. Par exemple, si un profil comporte deux points de terminaison et qu’un seul est sain, cette métrique a une valeur de **0,50**, comme indiqué dans la figure 3. 


![Métriques Traffic Manager - mode composite de l’état du point de terminaison](./media/traffic-manager-metrics-alerts/traffic-manager-metrics-endpoint-status-composite-view.png)

*Figure 3 : mode composite de la métrique d’état du point de terminaison – agrégation « Avg » sélectionnée*


![Métriques Traffic Manager - mode fractionné de l’état du point de terminaison](./media/traffic-manager-metrics-alerts/traffic-manager-metrics-endpoint-status-split-view.png)

*Figure 4 : mode fractionné des métriques d’état du point de terminaison*

Vous pouvez utiliser ces métriques via [le service Azure Monitor](../azure-monitor/platform/metrics-supported.md) du portail, l’[API REST](https://docs.microsoft.com/rest/api/monitor/), l’[interface de ligne de commande Azure CLI](https://docs.microsoft.com/cli/azure/monitor) et [Azure PowerShell](https://docs.microsoft.com/powershell/module/az.applicationinsights), ou dans la section des métriques du portail Traffic Manager.

## <a name="alerts-on-traffic-manager-metrics"></a>Alertes sur les métriques Traffic Manager
En plus du traitement et de l’affichage des métriques à partir de Traffic Manager, Azure Monitor permet aux clients de configurer et de recevoir des alertes associées à ces métriques. Vous pouvez choisir les conditions à remplir dans ces métriques pour qu’une alerte se produise, la fréquence à laquelle ces conditions doivent être surveillées et comment les alertes doivent vous être envoyées. Pour plus d’informations, consultez la [documentation sur les alertes Azure Monitor](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md).

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur le [service Azure Monitor](../azure-monitor/platform/metrics-supported.md)
- Découvrez comment [créer un graphique à l’aide d’Azure Monitor](../azure-monitor/platform/metrics-getting-started.md#create-your-first-metric-chart)
