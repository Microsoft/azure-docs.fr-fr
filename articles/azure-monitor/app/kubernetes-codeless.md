---
title: Surveiller des applications sur Azure Kubernetes service (AKS) avec Application Insights – Azure Monitor | Microsoft Docs
description: Azure Monitor s’intègre sans difficulté avec votre application s’exécutant sur Kubernetes, et vous permet d’épingler les problèmes liés à vos applications en un rien de temps.
ms.topic: conceptual
author: MS-jgol
ms.author: jgol
ms.date: 05/13/2020
ms.openlocfilehash: 52bd6d2a98e5126ff2463de1ef99da03ca555567
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87075300"
---
# <a name="zero-instrumentation-application-monitoring-for-kubernetes---azure-monitor-application-insights"></a>Surveillance d’application sans instrumentation pour Kubernetes – Azure Monitor Application Insights

> [!IMPORTANT]
>  Actuellement, vous pouvez activer la surveillance de vos applications Java s’exécutant sur Kubernetes sans instrumentation de votre code, en utilisant l’[agent autonome Java](./java-in-process-agent.md). Pendant que la solution permettant d’activer en toute transparence la surveillance des applications est en construction pour d’autres langages, utilisez les kits de développement logiciel (SDK) pour surveiller vos applications s’exécutant sur AKS : [ASP.NET Core](./asp-net-core.md), [ASP.NET](./asp-net.md), [Node.js](./nodejs.md), [JavaScript](./javascript.md) et [Python](./opencensus-python.md).

## <a name="application-monitoring-without-instrumenting-the-code"></a>Surveillance d’applications sans instrumentation du code
Actuellement, seul Java vous permet d’activer la surveillance d’applications sans instrumentation du code. Pour surveiller des applications dans d’autres langages, utilisez les kits de développement logiciel (SDK). 

## <a name="java"></a>Java
Une fois activé, l’agent Java collecte automatiquement une multitude de demandes, dépendances, journaux et métriques à partir des bibliothèques et infrastructures les plus utilisées.

Suivez [les instructions détaillées](./java-in-process-agent.md) pour surveiller vos applications Java qui s’exécutent dans des applications Kubernetes ainsi que dans d’autres environnements. 

## <a name="other-languages"></a>Autres langages

Pour les applications dans d’autres langages, nous vous recommandons actuellement d’utiliser les kits de développement logiciel (SDK) :
* [ASP.NET Core](./asp-net-core.md)
* [ASP.NET](./asp-net.md)
* [Node.JS](./nodejs.md) 
* [JavaScript](./javascript.md)
* [Python](./opencensus-python.md)

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [Azure Monitor](../overview.md) et [Application Insights](./app-insights-overview.md)
* Obtenir une vue d’ensemble du [Suivi distribué](./distributed-tracing.md) et découvrir ce que la [cartographie d’application](./app-map.md?tabs=net) peut faire pour vous
