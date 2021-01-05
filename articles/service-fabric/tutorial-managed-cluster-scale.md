---
title: Effectuer le scale-out d’un cluster Service Fabric managé (préversion)
description: Dans ce tutoriel, découvrez comment effectuer le scale-out d’un type de nœud d’un cluster Service Fabric managé.
ms.topic: tutorial
ms.date: 09/28/2020
ms.openlocfilehash: 01b299744d462496296884211eff08b7a9c64687
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92316136"
---
# <a name="tutorial-scale-out-a-service-fabric-managed-cluster-preview"></a>Tutoriel : Effectuer le scale-out d’un cluster Service Fabric managé (préversion)

Dans cette série de didacticiels, nous allons aborder les points suivants :

> [!div class="checklist"]
> * [Comment déployer un cluster Service Fabric managé.](tutorial-managed-cluster-deploy.md)
> * Comment effectuer le scale-out d’un cluster Service Fabric managé
> * [Comment ajouter et supprimer des types de nœud dans un cluster Service Fabric managé](tutorial-managed-cluster-add-remove-node-type.md)
> * [Comment déployer une application sur un cluster Service Fabric managé](tutorial-managed-cluster-deploy-app.md)

Cette partie de la série explique comment :

> [!div class="checklist"]
> * Mettre à l’échelle un nœud de cluster Service Fabric managé

## <a name="prerequisites"></a>Prérequis

* Un cluster Service Fabric managé (consultez [*Déployer un cluster managé*](tutorial-managed-cluster-deploy.md)).
* [Azure PowerShell 4.7.0](/powershell/azure/release-notes-azureps?preserve-view=true&view=azps-4.7.0#azservicefabric) ou ultérieur (consultez [*Installer Azure PowerShell*](/powershell/azure/install-az-ps?preserve-view=true&view=azps-4.7.0)).

## <a name="scale-a-service-fabric-managed-cluster"></a>Mettre à l’échelle un cluster Service Fabric managé
Modifiez le nombre d’instances pour augmenter ou diminuer le nombre de nœuds sur le type de nœud que vous souhaitez mettre à l’échelle. Vous pouvez trouver les noms des types de nœuds dans le modèle Azure Resource Manager (modèle ARM) à partir de votre déploiement de cluster ou dans Service Fabric Explorer.  

> [!NOTE]
> Si le type de nœud est « Principal », vous ne pourrez pas descendre en dessous de 3 nœuds pour un cluster de référence SKU « De base » et de 5 nœuds pour un cluster de référence SKU « Standard ».

```powershell
$resourceGroup = "myResourceGroup"
$clusterName = "mysfcluster"
$nodeTypeName = "FE"
$instanceCount = "7"

Set-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName -name $nodeTypeName -InstanceCount $instanceCount -Verbose
```

Le cluster commencera à se mettre à niveau automatiquement et, après quelques minutes, vous verrez les nœuds supplémentaires.

## <a name="next-steps"></a>Étapes suivantes

Dans cette étape, nous avons mis à l’échelle un type de nœud sur un cluster Service Fabric managé. Pour plus d’informations sur l’ajout et la suppression de types de nœud, consultez :

> [!div class="nextstepaction"]
> [Ajouter et supprimer des types de nœud dans un cluster Service Fabric managé](tutorial-managed-cluster-add-remove-node-type.md)