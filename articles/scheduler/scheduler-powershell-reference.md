---
title: Référence de cmdlets PowerShell
description: En savoir plus sur les applets de commande PowerShell pour Azure Scheduler
services: scheduler
ms.service: scheduler
author: derek1ee
ms.author: deli
ms.reviewer: klam, estfan
ms.topic: article
ms.date: 08/18/2016
ms.openlocfilehash: ccc9f709348d961e49bced00946658a6997837c0
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92368109"
---
# <a name="powershell-cmdlets-reference-for-azure-scheduler"></a>Informations de référence sur les applets de commande PowerShell pour Azure Scheluler

> [!IMPORTANT]
> [Azure Logic Apps](../logic-apps/logic-apps-overview.md) remplace Azure Scheduler, qui est en phase de [mise hors service](../scheduler/migrate-from-scheduler-to-logic-apps.md#retire-date). Pour poursuivre les travaux que vous avez configurés dans Scheduler, veuillez [migrer vers Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md) dès que possible. 
>
> Scheduler n’est plus disponible dans le portail Azure, mais l’[API REST](/rest/api/scheduler) et les [applets de commande Azure Scheduler PowerShell](scheduler-powershell-reference.md) restent disponibles pour vous permettre de gérer vos travaux et collections de travaux.

Vous pouvez utiliser les applets de commande PowerShell pour créer des scripts afin de créer et de gérer des travaux et des collections de travaux Scheduler. Cet article répertorie les principales applets de commande PowerShell pour Azure Scheduler avec des liens vers leurs articles de référence. Pour installer Azure PowerShell pour votre abonnement Azure, consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/). Pour plus d’informations sur les [applets de commande Azure Resource Manager](/powershell/azure/), consultez [Utilisation d’Azure PowerShell avec Azure Resource Manager](../azure-resource-manager/management/manage-resources-powershell.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

| Applet de commande | Description |
|--------|-------------|
| [Disable-AzSchedulerJobCollection](/powershell/module/azurerm.scheduler/disable-azurermschedulerjobcollection) |Désactive une collection de travaux. |
| [Enable-AzureRmSchedulerJobCollection](/powershell/module/azurerm.scheduler/enable-azurermschedulerjobcollection) |Active une collection de travaux. |
| [Get-AzSchedulerJob](/powershell/module/azurerm.scheduler/get-azurermschedulerjob) |Récupère les travaux du Planificateur. |
| [Get-AzSchedulerJobCollection](/powershell/module/azurerm.scheduler/get-azurermschedulerjobcollection) |Récupère les collections de travaux. |
| [Get-AzSchedulerJobHistory](/powershell/module/azurerm.scheduler/get-azurermschedulerjobhistory) |Affiche l’historique des travaux. |
| [New-AzSchedulerHttpJob](/powershell/module/azurerm.scheduler/new-azurermschedulerhttpjob) |Crée un travail HTTP. |
| [New-AzSchedulerJobCollection](/powershell/module/azurerm.scheduler/new-azurermschedulerjobcollection) |Crée une collection de travaux. |
| [New-AzSchedulerServiceBusQueueJob](/powershell/module/azurerm.scheduler/new-azurermschedulerservicebusqueuejob) | Crée un travail de file d’attente Service Bus. |
| [New-AzSchedulerServiceBusTopicJob](/powershell/module/azurerm.scheduler/new-azurermschedulerservicebustopicjob) |Crée un travail de rubrique Service Bus. |
| [New-AzSchedulerStorageQueueJob](/powershell/module/azurerm.scheduler/new-azurermschedulerstoragequeuejob) |Crée un travail de file d’attente de stockage. |
| [Remove-AzSchedulerJob](/powershell/module/azurerm.scheduler/remove-azurermschedulerjob) |Supprime un travail du Planificateur. |
| [Remove-AzSchedulerJobCollection](/powershell/module/azurerm.scheduler/remove-azurermschedulerjobcollection) |Supprime une collection de travaux. |
| [Set-AzSchedulerHttpJob](/powershell/module/azurerm.scheduler/set-azurermschedulerhttpjob) |Modifie un travail HTTP du Planificateur. |
| [Set-AzSchedulerJobCollection](/powershell/module/azurerm.scheduler/set-azurermschedulerjobcollection) |Modifie une collection de travaux. |
| [Set-AzSchedulerServiceBusQueueJob](/powershell/module/azurerm.scheduler/set-azurermschedulerservicebusqueuejob) |Modifie un travail de file d’attente Service Bus. |
| [Set-AzSchedulerServiceBusTopicJob](/powershell/module/azurerm.scheduler/set-azurermschedulerservicebustopicjob) |Modifie un travail de rubrique Service Bus. |
| [Set-AzSchedulerStorageQueueJob](/powershell/module/azurerm.scheduler/set-azurermschedulerstoragequeuejob) |Modifie un travail de file d’attente de stockage. |
||| 

Pour plus d’informations, vous pouvez exécuter les applets de commande suivantes : 

```text
Get-Help <cmdlet name> -Detailed
Get-Help <cmdlet name> -Examples
Get-Help <cmdlet name> -Full
```

## <a name="next-steps"></a>Étapes suivantes

* [Concepts, terminologie et hiérarchie d’entités d’Azure Scheduler](scheduler-concepts-terms.md)
* [Limites, valeurs par défaut et codes d’erreur d’Azure Scheluler](scheduler-limits-defaults-errors.md)
* [Informations de référence sur l’API REST d’Azure Scheluler](/rest/api/scheduler)