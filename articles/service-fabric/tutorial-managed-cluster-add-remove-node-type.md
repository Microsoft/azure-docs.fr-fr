---
title: Ajouter et supprimer des types de nœud liés à un cluster Service Fabric managé (préversion)
description: Dans ce tutoriel, découvrez comment ajouter et supprimer des types de nœud liés à un cluster Service Fabric managé.
ms.topic: tutorial
ms.date: 09/28/2020
ms.openlocfilehash: 51cc83b4accae5f2791ce378e30f6fa692446b1c
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92316206"
---
# <a name="tutorial-add-and-remove-node-types-from-a-service-fabric-managed-cluster-preview"></a>Tutoriel : Ajouter et supprimer des types de nœud liés à un cluster Service Fabric managé (préversion)

Dans cette série de tutoriels, nous allons aborder les points suivants :

> [!div class="checklist"]
> * [Comment déployer un cluster Service Fabric managé](tutorial-managed-cluster-deploy.md)
> * [Comment effectuer le scale-out d’un cluster Service Fabric managé](tutorial-managed-cluster-scale.md)
> * Comment ajouter et supprimer des nœuds dans un cluster Service Fabric managé
> * [Comment déployer une application sur un cluster Service Fabric managé](tutorial-managed-cluster-deploy-app.md)

Cette partie de la série explique comment :

> [!div class="checklist"]
> * Ajouter un type de nœud à un cluster Service Fabric managé
> * Supprimer un type de nœud d’un cluster Service Fabric managé

## <a name="prerequisites"></a>Prérequis

* Un cluster Service Fabric managé (consultez [*Déployer un cluster managé*](tutorial-managed-cluster-deploy.md)).
* [Azure PowerShell 4.7.0](/powershell/azure/release-notes-azureps?preserve-view=true&view=azps-4.7.0#azservicefabric) ou ultérieur (consultez [*Installer Azure PowerShell*](/powershell/azure/install-az-ps?preserve-view=true&view=azps-4.7.0)).

## <a name="add-a-node-type-to-a-service-fabric-managed-cluster"></a>Ajouter un type de nœud à un cluster Service Fabric managé

Vous pouvez ajouter un type de nœud à un cluster Service Fabric managé à l’aide d’un modèle Azure Resource Manager, de PowerShell ou de l’interface CLI. Dans ce tutoriel, nous allons ajouter un type de nœud à l’aide d’Azure PowerShell.

Pour créer un type de nœud, vous devez définir trois propriétés :

* **Nom du type de nœud** : Nom unique issu d’un des types de nœud existants du cluster.
* **Nombre d’instances** : Nombre initial de nœuds du nouveau type de nœud.
* **Taille de la machine virtuelle** : Référence SKU de machine virtuelle pour les nœuds. Si elle n’est pas spécifiée, la valeur par défaut *Standard_D2* est utilisée.

> [!NOTE]
> Si le type de nœud à ajouter correspond au premier ou au seul type de nœud du cluster, la propriété Principal doit être utilisée.

```powershell
$resourceGroup = "myResourceGroup"
$clusterName = "mysfcluster"
$nodeTypeName = "nt2"
$vmSize = "Standard_D2_v2"

New-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName -Name $nodeTypeName -InstanceCount 3 -vmSize $vmSize
```

## <a name="remove-a-node-type-from-a-service-fabric-managed-cluster"></a>Supprimer un type de nœud d’un cluster Service Fabric managé

Pour supprimer un type de nœud d’un cluster Service Fabric managé, vous devez utiliser PowerShell ou l’interface CLI. Dans ce tutoriel, nous allons supprimer un type de nœud à l’aide d’Azure PowerShell.

> [!NOTE]
> Il n’est pas possible de supprimer un type de nœud principal s’il s’agit du seul type de nœud principal du cluster.  

Pour supprimer un type de nœud :

```powershell
$resourceGroup = "myResourceGroup"
$clusterName = "myCluster"
$nodeTypeName = "nt2"

Remove-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName  -Name $nodeTypeName
```

## <a name="next-steps"></a>Étapes suivantes

 Dans cette section, nous avons ajouté et supprimé des types de nœuds. Pour savoir comment déployer une application sur un cluster Service Fabric managé, consultez :

> [!div class="nextstepaction"]
> [Déployer une application sur un cluster Service Fabric managé](tutorial-managed-cluster-deploy-app.md)