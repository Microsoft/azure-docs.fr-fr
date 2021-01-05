---
title: Supprimer un cluster Service Fabric dans Azure
description: Dans ce didacticiel, vous allez découvrir comment supprimer un cluster Service Fabric hébergé par Azure et toutes ses ressources. Vous pouvez supprimer le groupe de ressources contenant le cluster ou supprimer des ressources de manière sélective.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: aa5b09b627db6bd3a3d573c32e7e68a359e2c04d
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92738996"
---
# <a name="tutorial-remove-a-service-fabric-cluster-running-in-azure"></a>Didacticiel : Supprimer un cluster Service Fabric dans Azure

Cinquième d’une série, ce tutoriel montre comment supprimer un cluster Service Fabric exécuté dans Azure. Pour supprimer complètement un cluster Service Fabric, vous devez également supprimer les ressources utilisées par le cluster. Vous pouvez procéder de deux façons : supprimer le groupe de ressources dans lequel est situé le cluster (ce qui supprime la ressource de cluster et toutes les autres ressources du groupe de ressources) ou supprimer spécifiquement la ressource de cluster et les ressources associées (mais pas d’autres ressources du groupe de ressources).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Supprimer un groupe de ressources et toutes ses ressources
> * Supprimer des ressources de manière sélective à partir d’un groupe de ressources

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * Créer un [cluster Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) sécurisé sur Azure à l’aide d’un modèle
> * [Superviser un cluster](service-fabric-tutorial-monitor-cluster.md)
> * [Mettre à l’échelle un cluster](service-fabric-tutorial-scale-cluster.md)
> * [Mettre à niveau le runtime d’un cluster](service-fabric-tutorial-upgrade-cluster.md)
> * Suppression d'un cluster


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Installez [Azure PowerShell](/powershell/azure//install-az-ps) ou [Azure CLI](/cli/azure/install-azure-cli).
* Créer un [cluster Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) sécurisé sur Azure

## <a name="delete-the-resource-group-containing-the-service-fabric-cluster"></a>Supprimer le groupe de ressources contenant le cluster Service Fabric
Le plus simple pour supprimer le cluster et toutes les ressources qu’il consomme consiste à supprimer le groupe de ressources.

Connectez-vous à Azure et sélectionnez l’ID d’abonnement avec lequel vous souhaitez supprimer le cluster.  Vous pouvez trouver votre ID d’abonnement en vous connectant au [portail Azure](https://portal.azure.com). Supprimez le groupe de ressources et toutes les ressources de cluster à l’aide de l’applet de commande [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) ou de la commande [az group delete](/cli/azure/group?view=azure-cli-latest).

```powershell
Connect-AzAccount
Set-AzContext -SubscriptionId <guid>
$groupname = "sfclustertutorialgroup"
Remove-AzResourceGroup -Name $groupname -Force
```

```azurecli
az login
az account set --subscription <guid>
ResourceGroupName="sfclustertutorialgroup"
az group delete --name $ResourceGroupName
```

## <a name="selectively-delete-the-cluster-resource-and-the-associated-resources"></a>Supprimer de manière sélective la ressource de cluster et les ressources associées
Si votre groupe de ressources comporte uniquement des ressources qui sont liées au cluster Service Fabric à supprimer, il est plus facile de supprimer l’ensemble du groupe de ressources. Si vous souhaitez supprimer de manière sélective les ressources dans votre groupe de ressources et conserver des ressources qui ne sont pas associées au cluster, procédez comme suit.

Répertoriez les ressources dans le groupe de ressources :

```powershell
Connect-AzAccount
Set-AzContext -SubscriptionId <guid>
$groupname = "sfclustertutorialgroup"
Get-AzResource -ResourceGroupName $groupname | ft
```

```azurecli
az login
az account set --subscription <guid>
ResourceGroupName="sfclustertutorialgroup"
az resource list --resource-group $ResourceGroupName
```

Pour chacune des ressources à supprimer, exécutez le script suivant :

```powershell
Remove-AzResource -ResourceName "<name of the Resource>" -ResourceType "<Resource Type>" -ResourceGroupName $groupname -Force
```

```azurecli
az resource delete --name "<name of the Resource>" --resource-type "<Resource Type>" --resource-group $ResourceGroupName
```

Pour supprimer la ressource de cluster, exécutez le script suivant :

```powershell
Remove-AzResource -ResourceName "<name of the Resource>" -ResourceType "Microsoft.ServiceFabric/clusters" -ResourceGroupName $groupname -Force
```

```azurecli
az resource delete --name "<name of the Resource>" --resource-type "Microsoft.ServiceFabric/clusters" --resource-group $ResourceGroupName
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Supprimer un groupe de ressources et toutes ses ressources
> * Supprimer des ressources de manière sélective à partir d’un groupe de ressources

Maintenant que vous avez terminé ce didacticiel, essayez ce qui suit :
* Découvrez comment inspecter et gérer un cluster Service Fabric à l’aide de [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md).
* Découvrez comment [appliquer des correctifs au système d’exploitation Windows](service-fabric-patch-orchestration-application.md) du cluster.
* Découvrez comment agréger et collecter des événements pour les [clusters Windows](service-fabric-diagnostics-event-aggregation-wad.md) et [configurer Log Analytics](service-fabric-diagnostics-oms-setup.md) pour superviser les événements de cluster.
