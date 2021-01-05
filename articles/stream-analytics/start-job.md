---
title: Procédure de démarrage d’un travail Azure Stream Analytics
description: Cet article explique comment démarrer un travail Stream Analytics à partir du portail Azure, de PowerShell et de Visual Studio.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 04/03/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f89b23d0c20aafeeb07a744545208f6056efa2c9
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130814"
---
# <a name="how-to-start-an-azure-stream-analytics-job"></a>Procédure de démarrage d’un travail Azure Stream Analytics

Vous pouvez démarrer votre travail Azure Stream Analytics à partir du portail Azure, de PowerShell ou Visual Studio. Lorsque vous démarrez un travail, vous sélectionnez l’heure à laquelle le travail doit commencer à créer la sortie. Le portail Azure, Visual Studio et PowerShell ont différentes méthodes pour définir l’heure de début. Ces méthodes sont décrites ci-dessous.

## <a name="start-options"></a>Options de démarrage
Les trois options suivantes sont disponibles pour démarrer un travail. Notez que toutes les heures mentionnées ci-dessous sont celles spécifiées dans [TIMESTAMP BY](/stream-analytics-query/timestamp-by-azure-stream-analytics). Si TIMESTAMP BY n’est pas spécifié, l’heure d’arrivée sera utilisée.
* **Maintenant**  : rend le point de départ du flux d’événements de sortie identique à celui du démarrage du travail. Si un opérateur temporel est utilisé (par exemple, fenêtre de temps, LAG ou JOIN), Azure Stream Analytics consulte automatiquement les données de la source d’entrée. Par exemple, si vous démarrez un travail « Maintenant » et si votre requête utilise une fenêtre bascule de 5 minutes, Azure Stream Analytics recherche des données datant de 5 minutes auparavant dans l’entrée.
Le premier événement de sortie possible aura alors un horodatage identique ou postérieur à l’heure actuelle, et ASA garantit que tous les événements d’entrée qui peuvent logiquement contribuer à la sortie ont été pris en compte. Par exemple, aucune agrégation fenêtrée partielle n’est générée. Il s’agit toujours de la valeur agrégée complète.

* **Personnalisé**  : vous pouvez choisir le point de départ de la sortie. Comme avec l’option **Maintenant** , Azure Stream Analytics lit automatiquement les données avant cette heure si un opérateur temporel est utilisé. 

* **Lors du dernier arrêt**  : cette option est disponible lorsque le travail a été démarré précédemment, mais a été arrêté manuellement ou a échoué. Lorsque vous choisissez cette option, Azure Stream Analytics utilise la dernière heure de sortie pour redémarrer le travail. Par conséquent, aucune donnée n’est perdue. Comme avec les options précédentes, Azure Stream Analytics lit automatiquement les données avant cette heure si un opérateur temporel est utilisé. Étant donné que plusieurs partitions d’entrée peuvent avoir des heures différentes, la première heure d’arrêt de toutes les partitions est utilisée. Ainsi, des doublons peuvent se produire dans la sortie. Pou plus d’informations sur le traitement en une seule fois, consultez la page [Garanties de remise d’événement](/stream-analytics-query/event-delivery-guarantees-azure-stream-analytics).


## <a name="azure-portal"></a>Portail Azure

Accédez à votre travail dans le portail Azure et sélectionnez **Démarrer** sur la page de présentation. Choisissez l’ **heure de début de sortie du travail** , puis sélectionnez **Démarrer**.

Choisissez une des options pour l’ **heure de début de sortie du travail**. Les options sont *Maintenant* , *Personnalisé* et, si le travail a été exécuté précédemment, *Lors du dernier arrêt*. Pour plus d’informations sur ces options, voir plus haut.

## <a name="visual-studio"></a>Visual Studio

Dans la vue du travail, cliquez sur le bouton à la flèche verte pour démarrer le travail. Définissez le **mode de démarrage de la sortie du travail**  et sélectionnez **Démarrer**. L’état du travail devient **En cours d’exécution**.

Il existe trois options pour le **mode de démarrage de la sortie du travail**  : *JobStartTime* , *CustomTime* et *LastOutputEventTime*. Si cette propriété est absente, la valeur par défaut est *JobStartTime*. Pour plus d’informations sur ces options, voir plus haut.


## <a name="powershell"></a>PowerShell

Utilisez la cmdlet suivante pour démarrer votre travail à l’aide de PowerShell :

```powershell
Start-AzStreamAnalyticsJob `
  -ResourceGroupName $resourceGroup `
  -Name $jobName `
  -OutputStartMode 'JobStartTime'
```

Il existe trois solutions pour **OutputStartMode**  : *JobStartTime* , *CustomTime* et *LastOutputEventTime*. Si cette propriété est absente, la valeur par défaut est *JobStartTime*. Pour plus d’informations sur ces options, voir plus haut.

Pour plus d’informations sur la cmdlet `Start-AzStreamAnalyitcsJob`, consultez les informations de référence sur [Start-AzStreamAnalyticsJob référence](/powershell/module/az.streamanalytics/start-azstreamanalyticsjob).

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md)
* [Démarrage rapide : Créer un travail Stream Analytics à l’aide d’Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Démarrage rapide : créer une tâche Stream Analytics à l’aide des outils Azure Stream Analytics pour Visual Studio](stream-analytics-quick-create-vs.md)