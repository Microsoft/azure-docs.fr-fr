---
title: Interroger les abonnements Azure Event Grid
description: Cet article explique comment répertorier les abonnements Event Grid que comprend votre abonnement Azure. Vous devez fournir des paramètres différents selon le type d’abonnement.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 3d700f543bc5e3c7add2a346c10acf975e1c2462
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86120447"
---
# <a name="query-event-grid-subscriptions"></a>Interroger les abonnements Event Grid 

Cet article explique comment lister les abonnements Event Grid que comprend votre abonnement Azure. Pour interroger vos abonnements Event Grid existants, il est important de connaître les différents types d’abonnements. Vous devez fournir des paramètres différents selon le type d’abonnement que vous souhaitez obtenir.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="resource-groups-and-azure-subscriptions"></a>Groupes de ressources et abonnements Azure

Les groupes de ressources et les abonnements Azure ne sont pas des ressources Azure. Par conséquent, les abonnements Event Grid à des groupes de ressources ou à des abonnements Azure ne présentent pas les mêmes propriétés que les abonnements Event Grid à des ressources Azure. Les abonnements Event Grid à des groupes de ressources ou à des abonnements Azure sont considérés comme globaux.

Pour obtenir les abonnements Event Grid d’un abonnement Azure et de ses groupes de ressources, vous n’avez pas besoin de fournir de paramètres. Veillez simplement à sélectionner l’abonnement Azure que vous souhaitez interroger. Les exemples suivants ne permettent pas d’obtenir les abonnements Event Grid de rubriques personnalisées ou de ressources Azure.

Pour l’interface de ligne de commande Azure, consultez :

```azurecli-interactive
az account set -s "My Azure Subscription"
az eventgrid event-subscription list
```

Pour PowerShell, utilisez la commande suivante :

```azurepowershell-interactive
Set-AzContext -Subscription "My Azure Subscription"
Get-AzEventGridSubscription
```

Pour obtenir les abonnements Event Grid d’un abonnement Azure, indiquez le type de rubrique **Microsoft.Resources.Subscriptions**.

Pour l’interface de ligne de commande Azure, consultez :

```azurecli-interactive
az eventgrid event-subscription list --topic-type-name "Microsoft.Resources.Subscriptions" --location global
```

Pour PowerShell, utilisez la commande suivante :

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicTypeName "Microsoft.Resources.Subscriptions"
```

Pour obtenir les abonnements Event Grid de tous les groupes de ressources que comprend un abonnement Azure, indiquez le type de rubrique **Microsoft.Resources.ResourceGroups**.

Pour l’interface de ligne de commande Azure, consultez :

```azurecli-interactive
az eventgrid event-subscription list --topic-type-name "Microsoft.Resources.ResourceGroups" --location global
```

Pour PowerShell, utilisez la commande suivante :

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicTypeName "Microsoft.Resources.ResourceGroups"
```

Pour obtenir les abonnements Event Grid d’un groupe de ressources spécifique, indiquez le nom du groupe de ressources en tant que paramètre.

Pour l’interface de ligne de commande Azure, consultez :

```azurecli-interactive
az eventgrid event-subscription list --resource-group myResourceGroup --location global
```

Pour PowerShell, utilisez la commande suivante :

```azurepowershell-interactive
Get-AzEventGridSubscription -ResourceGroupName myResourceGroup
```

## <a name="custom-topics-and-azure-resources"></a>Rubriques personnalisées et ressources Azure

Les rubriques personnalisées Event Grid sont des ressources Azure. Par conséquent, l’interrogation des abonnements Event Grid de rubriques personnalisées se fait de la même manière que pour les autres ressources (un compte de stockage Blob, par exemple). Pour obtenir les abonnements Event Grid de rubriques personnalisées, vous devez fournir des paramètres qui identifient la ressource ou l’emplacement de la ressource. Il n’est pas possible d’interroger les abonnements Event Grid de l’ensemble des ressources de votre abonnement Azure.

Pour obtenir les abonnements Event Grid des rubriques personnalisées et des autres ressources situées à un emplacement, indiquez le nom de celui-ci.

Pour l’interface de ligne de commande Azure, consultez :

```azurecli-interactive
az eventgrid event-subscription list --location westus2
```

Pour PowerShell, utilisez la commande suivante :

```azurepowershell-interactive
Get-AzEventGridSubscription -Location westus2
```

Pour obtenir les abonnements à des rubriques personnalisées pour un emplacement, indiquez l’emplacement en question et le type de rubrique **Microsoft.EventGrid.Topics**.

Pour l’interface de ligne de commande Azure, consultez :

```azurecli-interactive
az eventgrid event-subscription list --topic-type-name "Microsoft.EventGrid.Topics" --location "westus2"
```

Pour PowerShell, utilisez la commande suivante :

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicTypeName "Microsoft.EventGrid.Topics" -Location westus2
```

Pour obtenir les abonnements à des comptes de stockage pour un emplacement, indiquez l’emplacement en question et le type de rubrique **Microsoft.Storage.StorageAccounts**.

Pour l’interface de ligne de commande Azure, consultez :

```azurecli-interactive
az eventgrid event-subscription list --topic-type "Microsoft.Storage.StorageAccounts" --location westus2
```

Pour PowerShell, utilisez la commande suivante :

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicTypeName "Microsoft.Storage.StorageAccounts" -Location westus2
```

Pour obtenir les abonnements Event Grid d’une rubrique personnalisée, indiquez son nom et le nom de son groupe de ressources.

Pour l’interface de ligne de commande Azure, consultez :

```azurecli-interactive
az eventgrid event-subscription list --topic-name myCustomTopic --resource-group myResourceGroup
```

Pour PowerShell, utilisez la commande suivante :

```azurepowershell-interactive
Get-AzEventGridSubscription -TopicName myCustomTopic -ResourceGroupName myResourceGroup
```

Pour obtenir les abonnements Event Grid d’une ressource spécifique, indiquez son ID.

Pour l’interface de ligne de commande Azure, consultez :

```azurecli-interactive
resourceid=$(az resource show -n mystorage -g myResourceGroup --resource-type "Microsoft.Storage/storageaccounts" --query id --output tsv)
az eventgrid event-subscription list --resource-id $resourceid
```

Pour PowerShell, utilisez la commande suivante :

```azurepowershell-interactive
$resourceid = (Get-AzResource -Name mystorage -ResourceGroupName myResourceGroup).ResourceId
Get-AzEventGridSubscription -ResourceId $resourceid
```

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la remise d’événements et sur les nouvelles tentatives, consultez [Remise et nouvelle tentative de remise de messages avec Azure Grid](delivery-and-retry.md).
* Pour une présentation d’Event Grid, consultez [À propos d’Event Grid](overview.md).
* Pour une prise en main rapide d’Event Grid, consultez [Créer et acheminer des événements personnalisés avec Azure Event Grid](custom-event-quickstart.md).
