---
title: Haute disponibilité et fiabilité
description: En savoir plus sur la haute disponibilité et la fiabilité dans Azure Scheduler
services: scheduler
ms.service: scheduler
author: derek1ee
ms.author: deli
ms.reviewer: klam, estfan
ms.topic: article
ms.date: 08/16/2016
ms.openlocfilehash: 98a6672af7e74fdd0732f3ba03264d2f674eb44f
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92368143"
---
# <a name="high-availability-and-reliability-for-azure-scheduler"></a>Haute disponibilité et fiabilité pour Azure Scheluler

> [!IMPORTANT]
> [Azure Logic Apps](../logic-apps/logic-apps-overview.md) remplace Azure Scheduler, qui est en phase de [mise hors service](../scheduler/migrate-from-scheduler-to-logic-apps.md#retire-date). Pour poursuivre les travaux que vous avez configurés dans Scheduler, veuillez [migrer vers Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md) dès que possible. 
>
> Scheduler n’est plus disponible dans le portail Azure, mais l’[API REST](/rest/api/scheduler) et les [applets de commande Azure Scheduler PowerShell](scheduler-powershell-reference.md) restent disponibles pour vous permettre de gérer vos travaux et collections de travaux.

Azure Scheduler assure à la fois la [haute disponibilité](/azure/architecture/framework/#resiliency) et la fiabilité pour vos travaux. Pour plus d’informations, consultez [Contrat SLA pour Scheduler](https://azure.microsoft.com/support/legal/sla/scheduler).

## <a name="high-availability"></a>Haute disponibilité

Azure Scheduler est [hautement disponible] et utilise le déploiement de service géo-redondant et la réplication de travail géo-régionale.

### <a name="geo-redundant-service-deployment"></a>Déploiement de service géo-redondant

Azure Scheduler est disponible dans presque [toutes les régions géographiques actuellement prises en charge par Azure](https://azure.microsoft.com/global-infrastructure/regions/#services). Ainsi, si un centre de données Azure situé dans une région hébergée devient indisponible, vous pouvez continuer à utiliser Azure Scheduler, car les fonctionnalités de basculement du service vous permettent d’accéder à Scheduler via un autre centre de données.

### <a name="geo-regional-job-replication"></a>Réplication de travail géo-régionale

Vos propres travaux dans Azure Scheduler sont répliqués entre les régions Azure. Ainsi, en cas de panne dans une région, Azure Scheduler bascule et garantit l’exécution de vos travaux à partir d'un autre centre de données dans la région géographique associée.

Par exemple, si vous créez un travail dans la région USA Centre Sud, Azure Scheduler réplique automatiquement ce travail dans la région USA Centre Nord. Si une défaillance se produit dans la région USA Centre Sud, Azure Scheduler exécute le travail dans la région USA Centre Nord. 

![Réplication de travail géo-régionale](./media/scheduler-high-availability-reliability/scheduler-high-availability-reliability-image1.png)

Azure Scheduler garantit également que vos données resteront dans la même région géographique, mais élargie, au cas où une défaillance se produirait dans Azure. Par conséquent, vous n’êtes pas tenu de dupliquer vos travaux lorsque vous souhaitez simplement bénéficier d’une haute disponibilité. Azure Scheduler fournit automatiquement la haute disponibilité pour vos travaux.

## <a name="reliability"></a>Fiabilité

Azure Scheduler garantit sa propre haute disponibilité mais adopte une approche différente pour les travaux créés par l'utilisateur. Par exemple, supposons que votre travail appelle un point de terminaison HTTP qui n'est pas disponible. Azure Scheduler tente d’exécuter votre travail avec succès en vous offrant des méthodes alternatives pour gérer les défaillances : 

* Configurez des stratégies de nouvelle tentative.
* Configurez d’autres points de terminaison.

<a name="retry-policies"></a>

### <a name="retry-policies"></a>Stratégies de nouvelle tentative

Azure Scheduler vous permet de configurer des stratégies de nouvelle tentative. Si un travail échoue, Scheduler tente par défaut de l'exécuter encore quatre fois, à des intervalles de 30 secondes. Vous pouvez rendre cette stratégie de nouvelle tentative plus agressive, en spécifiant 10 nouvelles tentatives à 30 secondes d’intervalle, ou moins agressive, avec deux nouvelles tentatives à un jour d’intervalle.

Par exemple, supposons que vous créez un travail hebdomadaire qui appelle un point de terminaison HTTP. Si le point de terminaison HTTP devient indisponible pendant quelques heures lorsque votre travail s’exécute, vous ne voudrez vraisemblablement pas attendre une semaine pour réexécuter le travail, ce qui est le cas puisque la stratégie de nouvelle tentative par défaut ne fonctionne pas dans ce cas. Par conséquent, vous pouvez modifier la stratégie de nouvelle tentative standard pour que les nouvelles tentatives se produisent, par exemple, toutes les trois heures, plutôt que toutes les 30 secondes. 

Pour découvrir comment configurer une stratégie de nouvelle tentative, consultez [retryPolicy](scheduler-concepts-terms.md#retrypolicy).

### <a name="alternate-endpoints"></a>Autres points de terminaison

Si votre travail Azure Scheduler appelle un point de terminaison inaccessible, même après avoir appliqué la stratégie de nouvelle tentative, Scheduler bascule vers un autre point de terminaison qui peut gérer ces erreurs. Par conséquent, si vous configurez ce point de terminaison, Scheduler appelle ce point de terminaison, ce qui rend vos propres travaux hautement disponibles en cas de défaillances.

Par exemple, le diagramme suivant montre comment Scheduler suit la stratégie de nouvelle tentative lors de l’appel d’un service web à New York. Si les nouvelles tentatives échouent, Scheduler recherche un autre point de terminaison. Si le point de terminaison existe, Scheduler commence à envoyer des demandes à l’autre point de terminaison. La même stratégie de nouvelle tentative s'applique à l'action d'origine et à l'autre action.

![Comportement de Scheduler avec la stratégie de nouvelle tentative et l’autre point de terminaison](./media/scheduler-high-availability-reliability/scheduler-high-availability-reliability-image2.png)

Le type d’action pour l’autre action peut différer de l’action d’origine. Par exemple, bien que l’action d’origine appelle un point de terminaison HTTP, l’autre action peut consigner les erreurs à l’aide d’une action de file d’attente de stockage, de file d’attente Service Bus ou de rubrique Service Bus.

Pour découvrir comment configurer un autre point de terminaison, consultez [errorAction](scheduler-concepts-terms.md#error-action).

## <a name="next-steps"></a>Étapes suivantes

* [Concepts, terminologie et hiérarchie des entités](scheduler-concepts-terms.md)
* [Informations de référence sur l’API REST d’Azure Scheluler](/rest/api/scheduler)
* [Informations de référence sur les applets de commande PowerShell d’Azure Scheluler](scheduler-powershell-reference.md)
* [Limites, quotas, valeurs par défaut et codes d’erreur](scheduler-limits-defaults-errors.md)