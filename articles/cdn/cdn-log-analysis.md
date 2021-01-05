---
title: Analyse des modèles d’utilisation CDN Azure
description: Cet article décrit les différents types de rapports d’analyse disponibles pour les produits Azure CDN.
services: cdn
author: asudbring
manager: KumudD
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/30/2020
ms.author: allensu
ms.openlocfilehash: 2c593cda7761ce1defa6bdb31e0dbb528b9e5eca
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483986"
---
# <a name="analyze-azure-cdn-usage-patterns"></a>Analyse des modèles d’utilisation CDN Azure

Après avoir activé le CDN pour votre application, vous pouvez surveiller l’utilisation du CDN, vérifier l’intégrité de votre application et corriger les éventuels problèmes. Azure CDN fournit ces fonctionnalités sous les formes suivantes : 

## <a name="raw-logs-for-azure-cdn-from-microsoft"></a>Journaux bruts pour Azure CDN de Microsoft
Avec un profil Microsoft standard, vous pouvez activer les journaux bruts et choisir de diffuser les journaux dans :

* Stockage Azure
* Hubs d'événements
* Azure Log Analytics

Avec Azure Log Analytics vous pouvez afficher les métriques de surveillance et configurer des alertes. 

Pour en savoir plus, voir [Journaux bruts HTTP Azure CDN](monitoring-and-access-log.md).


## <a name="core-analytics-via-azure-diagnostic-logs"></a>Analytique principale via les journaux de diagnostic Azure

L’analytique principale est disponible pour les points de terminaison CDN à tous les niveaux tarifaires. Les journaux de diagnostic Azure permettent d’exporter l’analytique principale vers un stockage Azure, des hubs d’événements ou des journaux Azure Monitor. Les journaux Azure Monitor offrent une solution avec des graphiques configurables par l’utilisateur et personnalisables. Pour plus d’informations sur les journaux de diagnostic Azure, consultez [Journaux de diagnostic Azure](cdn-azure-diagnostic-logs.md).

## <a name="verizon-core-reports"></a>Rapports principaux de Verizon

Les profils **CDN Azure Standard fourni par Verizon** ou **CDN Azure Premium fourni par Verizon** fournissent des rapports principaux. Vous pouvez afficher les rapports principaux dans le portail supplémentaire Verizon. Les rapports principaux de Verizon sont accessibles par le biais de l’option **Gérer** du portail Azure. Ils offrent toute une variété de graphiques et de vues. Pour plus d’informations, consultez [Rapports principaux de Verizon](cdn-analyze-usage-patterns.md).

## <a name="verizon-custom-reports"></a>Rapports personnalisés de Verizon

Les profils **CDN Azure Standard fourni par Verizon** ou **CDN Azure Premium fourni par Verizon** fournissent des rapports personnalisés. Vous pouvez afficher les rapports personnalisés dans le portail supplémentaire Verizon. Les rapports personnalisés de Verizon sont accessibles par le biais de l’option **Gérer** du portail Azure. 

Les rapports personnalisés affichent le nombre d’accès ou les données transférées pour chaque CNAME de périphérie. Les données sont groupées par code de réponse HTTP ou état du cache sur une période de temps. Pour plus d’informations, consultez [Rapports personnalisés de Verizon](cdn-verizon-custom-reports.md).

## <a name="azure-cdn-premium-from-verizon-reports"></a>Rapports CDN Azure Premium fourni par Verizon

Avec **Azure CDN Premium de Verizon**, vous pouvez également accéder aux rapports suivants :
   * [Rapports HTTP avancés](cdn-advanced-http-reports.md)
   * [Statistiques en temps réel](cdn-real-time-stats.md)
   * [Performances de nœuds de périphérie Azure CDN](cdn-edge-performance.md)

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez appris les différentes options pour les rapports d’analyse pour Azure CDN.

Pour plus d’informations sur Azure CDN et les autres services Azure mentionnés dans cet article, consultez :

* [Présentation d’Azure CDN](cdn-overview.md)
* [Journaux bruts HTTP Azure CDN](monitoring-and-access-log.md)
