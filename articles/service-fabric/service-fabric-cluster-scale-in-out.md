---
title: Effectuer un scale-in ou un scale-out d’un cluster Service Fabric
description: Augmentez ou diminuez la taille des instances d’un cluster Service Fabric pour répondre à la demande en définissant des règles de mise à l’échelle automatique pour chaque type de nœud/groupe de machines virtuelles identiques. Ajouter ou supprimer des nœuds d’un cluster Service Fabric
ms.topic: conceptual
ms.date: 03/12/2019
ms.openlocfilehash: 6ee04c73b75d6b335e450ff816c51f0a3089b918
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94409958"
---
# <a name="scale-a-cluster-in-or-out"></a>Mettre à l’échelle un cluster

> [!WARNING]
> Lisez cette section avant d’effectuer une mise à l’échelle

La mise à l’échelle des ressources de calcul pour provisionner la charge de travail de votre application nécessite une planification intentionnelle. Elle durera généralement plus d’une heure dans un environnement de production et vous devrez comprendre votre charge de travail et le contexte commercial. En fait, si vous n’avez jamais effectué cette opération auparavant, il est recommandé de commencer par lire et bien comprendre les [points à prendre en compte dans la planification des capacités du cluster Service Fabric](service-fabric-cluster-capacity.md) avant de poursuivre ce document. Cette recommandation vise à éviter les problèmes non intentionnels de LiveSite. Il est également recommandé de bien tester les opérations que vous choisissez d’effectuer dans un environnement hors production. Vous pouvez à tout moment [signaler des problèmes de production ou demander un support payant pour Azure](service-fabric-support.md#report-production-issues-or-request-paid-support-for-azure). Pour les ingénieurs affectés à ces opérations ayant un contexte approprié, cet article décrira les opérations de mise à l’échelle. Toutefois, vous devez déterminer et bien comprendre les opérations appropriées à votre cas d’utilisation, telles que les ressources à mettre à l’échelle (CPU, stockage, mémoire), le sens de mise à l’échelle (vertical ou horizontal) et les opérations à effectuer (déploiement de modèles de ressources, portail, PowerShell/CLI).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="scale-a-service-fabric-cluster-in-or-out-using-auto-scale-rules-or-manually"></a>Augmenter ou diminuer la taille des instances d’un cluster Service Fabric à l’aide de règles de mise à l’échelle automatique ou manuellement
Les jeux de mise à l’échelle de machine virtuelle sont des ressources de calcul Azure que vous pouvez utiliser pour déployer et gérer une collection de machines virtuelles en tant que jeu. Chaque type de nœud qui est défini dans un cluster Service Fabric est configuré en tant que groupe de machines virtuelles identiques distinct. Chaque type de nœud peut ensuite faire l’objet d’une augmentation ou d’une diminution de la taille des instances de manière indépendante, avoir différents jeux de ports ouverts et présenter différentes métriques de capacité. Pour en savoir plus, consultez le document portant sur les [types de nœuds Service Fabric](service-fabric-cluster-nodetypes.md). Étant donné que les types de nœuds Service Fabric de votre cluster sont constitués de groupes de machines virtuelles identiques sur le serveur principal, vous devez définir des règles de mise à l’échelle automatique pour chaque type de nœud/groupe de machines virtuelles identiques.

> [!NOTE]
> Votre abonnement doit avoir suffisamment de cœurs pour ajouter les nouvelles machines virtuelles qui composeront ce cluster. Il n’existe actuellement aucune validation de modèle. Vous obtenez donc une erreur liée au temps de déploiement si l’une des limites de quota est atteinte. De même, un type de nœud unique ne peut tout simplement pas dépasser 100 nœuds par VMSS. Il se peut que vous ayez besoin d’ajouter des VMSS pour atteindre l’échelle ciblée, et la mise à l’échelle automatique ne peut pas ajouter des VMSS de façon automatique. L’ajout de VMSS sur place, dans un cluster activé, est une tâche difficile qui conduit fréquemment les utilisateurs à provisionner de nouveaux clusters avec les types de nœuds appropriés qui ont été provisionnés lors de la création ; [planifier la capacité du cluster](./service-fabric-cluster-capacity.md) en conséquence. 
> 
> 

## <a name="choose-the-node-typevirtual-machine-scale-set-to-scale"></a>Choisir le type de nœud/groupe de machines virtuelles identiques à mettre à l’échelle
Actuellement, vous n’avez pas la possibilité de spécifier les règles de mise à l’échelle automatique pour les groupes de machines virtuelles identiques à l’aide du portail pour créer un cluster Service Fabric. Nous allons donc utiliser Azure PowerShell (1.0+) pour lister les types de nœuds et leur ajouter des règles de mise à l’échelle automatique.

Pour obtenir la liste des groupes de machines virtuelles identiques qui composent votre cluster, exécutez les cmdlets suivantes :

```powershell
Get-AzResource -ResourceGroupName <RGname> -ResourceType Microsoft.Compute/VirtualMachineScaleSets

Get-AzVmss -ResourceGroupName <RGname> -VMScaleSetName <virtual machine scale set name>
```

## <a name="set-auto-scale-rules-for-the-node-typevirtual-machine-scale-set"></a>Définir des règles de mise à l’échelle automatique pour le type de nœud/groupe de machines virtuelles identiques
Si votre cluster comporte plusieurs types de nœud, vous devez effectuer cette opération pour chaque type de nœud/groupe de machines virtuelles identiques dont vous voulez augmenter ou diminuer la taille des instances. Tenez compte du nombre de nœuds dont vous devez disposer avant de configurer la mise à l’échelle automatique. Le nombre de nœuds minimal dont vous devez disposer pour le type de nœud principal est déterminé par le niveau de fiabilité que vous avez choisi. En savoir plus sur les [niveaux de fiabilité](service-fabric-cluster-capacity.md).

> [!NOTE]
> Un scale-in du type de nœud principal en deçà du nombre minimal peut rendre le cluster instable, voire même entraîner son interruption. Cela peut entraîner une perte de données pour vos applications et pour les services système.
> 
> 

Actuellement, la fonctionnalité de mise à l’échelle automatique ne dépend pas des charges que vos applications peuvent signaler à Service Fabric. Par conséquent, pour le moment, la mise à l’échelle automatique que vous obtenez dépend uniquement des compteurs de performances émis par chaque instance du groupe de machines virtuelles identiques.  

Suivez ces instructions afin de [configurer la mise à l’échelle automatique pour chaque groupe de machines virtuelles identiques](../virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview.md).

> [!NOTE]
> Dans un scénario de scale-in, à moins que votre type de nœud ne possède un [niveau de durabilité][durability] Gold ou Silver, vous devez appeler l’[applet de commande Remove-ServiceFabricNodeState](/powershell/module/servicefabric/remove-servicefabricnodestate) avec le nom de nœud approprié. Pour la durabilité Bronze, il n’est pas recommandé d’effectuer un scale-in de plus d’un nœud à la fois.
> 

## <a name="manually-add-vms-to-a-node-typevirtual-machine-scale-set"></a>Ajouter manuellement des machines virtuelles à un type de nœud/groupe de machines virtuelles identiques

Lorsque vous effectuez un scale-out, vous ajoutez des instances de machines virtuelles au groupe identique. Ces instances deviennent les nœuds que Service Fabric utilise. Service Fabric sait quand des instances sont ajoutées au groupe identique et réagit automatiquement. 

> [!NOTE]
> L’ajout de machines virtuelles prend du temps, donc ne vous attendez pas à ce que les ajouts soient instantanés. Envisagez d’ajouter de la capacité bien à l’avance, car vous devez compter plus de 10 minutes avant que la capacité de la machine virtuelle soit disponible pour placer les réplicas/instances de service.
> 

### <a name="add-vms-using-a-template"></a>Ajouter de machines virtuelles à l’aide d’un modèle
Suivez les exemples/instructions dans la [galerie de modèles de démarrage rapide](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-scale-existing) pour modifier le nombre de machines virtuelles dans chaque type de nœud. 

### <a name="add-vms-using-powershell-or-cli-commands"></a>Ajouter des machines virtuelles à l’aide des commandes PowerShell ou CLI
Le code suivant obtient un groupe identique par son nom et augmente de 1 sa **capacité**.

```powershell
$scaleset = Get-AzVmss -ResourceGroupName SFCLUSTERTUTORIALGROUP -VMScaleSetName nt1vm
$scaleset.Sku.Capacity += 1

Update-AzVmss -ResourceGroupName $scaleset.ResourceGroupName -VMScaleSetName $scaleset.Name -VirtualMachineScaleSet $scaleset
```

Ce code définit la capacité avec la valeur 6.

```azurecli
# Get the name of the node with
az vmss list-instances -n nt1vm -g sfclustertutorialgroup --query [*].name

# Use the name to scale
az vmss scale -g sfclustertutorialgroup -n nt1vm --new-capacity 6
```

## <a name="manually-remove-vms-from-a-node-typevirtual-machine-scale-set"></a>Supprimer manuellement des machines virtuelles d’un type de nœud/groupe de machines virtuelles identiques
Lorsque vous effectuez un scale-in d’un type de nœud, vous supprimez les instances de machine virtuelle du groupe identique. Si le type de nœud dispose d’un niveau de durabilité Bronze, Service Fabric n’est pas informé des événements et signale qu’un nœud a disparu. Service Fabric signale donc que l’état du cluster n’est pas sain. Pour éviter cet état incorrect, vous devez supprimer le nœud du cluster et l’état du nœud de manière explicite.

Les services système Service Fabric s’exécutent sur le type de nœud principal de votre cluster. Lorsque vous effectuez un scale-in du type de nœud principal, n’effectuez jamais de scale-in du nombre d’instances à moins de ce que garantit le [niveau de fiabilité](service-fabric-cluster-capacity.md). 
 
Pour un service avec état, un certain nombre de nœuds doivent toujours fonctionner afin de maintenir la disponibilité et de conserver l’état de votre service. Au minimum, le nombre de nœuds doit être égal au nombre de jeux de réplicas cibles du service/de la partition.

### <a name="remove-the-service-fabric-node"></a>Supprimer le nœud Service Fabric

Les étapes pour supprimer manuellement un état de nœud s’appliquent uniquement aux types de nœuds avec un niveau de durabilité *Bronze*.  Pour les niveaux de durabilité *Silver* et *Gold*, ces étapes sont effectuées automatiquement par la plateforme. Pour plus d’informations sur la durabilité, consultez la [Planification de la capacité des clusters Service Fabric][durability].

>[!NOTE]
> Conservez au minimum cinq nœuds pour tout groupe de machines virtuelles identiques sur lequel le niveau de durabilité Gold ou Silver est activé. Votre cluster entrera en état d'erreur si vous passez en dessous de ce seuil et vous devrez nettoyer manuellement les nœuds supprimés.

Pour que les nœuds du cluster soient toujours répartis uniformément entre les domaines d’erreur et de mise à jour, et ainsi permettre leur utilisation homogène, le dernier nœud créé doit être supprimé en premier. En d’autres termes, les nœuds doivent être supprimés dans l’ordre inverse de leur création. Le dernier nœud créé est celui contenant la plus grande valeur de propriété `virtual machine scale set InstanceId`. Les exemples de code ci-dessous renvoient le dernier nœud créé.

```powershell
Get-ServiceFabricNode | Sort-Object NodeInstanceId -Descending | Select-Object -First 1
```

```shell
sfctl node list --query "sort_by(items[*], &name)[-1]"
```

Le cluster Service Fabric doit être informé de la suppression de ce nœud. Pour cela, trois étapes sont nécessaires :

1. Désactivez le nœud afin qu’il ne soit plus utilisé pour la réplication de données.  
PowerShell : `Disable-ServiceFabricNode`  
sfctl : `sfctl node disable`

2. Arrêtez le nœud afin que le runtime Service Fabric soit arrêté correctement et que votre application reçoive une requête de fin d’exécution.  
PowerShell : `Start-ServiceFabricNodeTransition -Stop`  
sfctl : `sfctl node transition --node-transition-type Stop`

2. Supprimez le nœud du cluster.  
PowerShell : `Remove-ServiceFabricNodeState`  
sfctl : `sfctl node remove-state`

Une fois ces trois étapes effectuées, vous pouvez supprimer le nœud du groupe identique. Si vous utilisez un niveau de durabilité autre que le niveau [Bronze][durability], ces étapes sont effectuées automatiquement lorsque vous supprimez une instance de groupe identique.

Le bloc de code suivant obtient le dernier nœud créé, le désactive, l’arrête et le supprime du cluster.

```powershell
#### After you've connected.....
# Get the node that was created last
$node = Get-ServiceFabricNode | Sort-Object { $_.NodeName.Substring($_.NodeName.LastIndexOf('_') + 1) } -Descending | Select-Object -First 1

# Node details for the disable/stop process
$nodename = $node.NodeName
$nodeid = $node.NodeInstanceId

$loopTimeout = 10

# Run disable logic
Disable-ServiceFabricNode -NodeName $nodename -Intent RemoveNode -TimeoutSec 300 -Force

$state = Get-ServiceFabricNode | Where-Object NodeName -eq $nodename | Select-Object -ExpandProperty NodeStatus

while (($state -ne [System.Fabric.Query.NodeStatus]::Disabled) -and ($loopTimeout -ne 0))
{
    Start-Sleep 5
    $loopTimeout -= 1
    $state = Get-ServiceFabricNode | Where-Object NodeName -eq $nodename | Select-Object -ExpandProperty NodeStatus
    Write-Host "Checking state... $state found"
}

# Exit if the node was unable to be disabled
if ($state -ne [System.Fabric.Query.NodeStatus]::Disabled)
{
    Write-Error "Disable failed with state $state"
}
else
{
    # Stop node
    $stopid = New-Guid
    Start-ServiceFabricNodeTransition -Stop -OperationId $stopid -NodeName $nodename -NodeInstanceId $nodeid -StopDurationInSeconds 300

    $state = (Get-ServiceFabricNodeTransitionProgress -OperationId $stopid).State
    $loopTimeout = 10

    # Watch the transaction
    while (($state -eq [System.Fabric.TestCommandProgressState]::Running) -and ($loopTimeout -ne 0))
    {
        Start-Sleep 5
        $state = (Get-ServiceFabricNodeTransitionProgress -OperationId $stopid).State
        Write-Host "Checking state... $state found"
    }

    if ($state -ne [System.Fabric.TestCommandProgressState]::Completed)
    {
        Write-Error "Stop transaction failed with $state"
    }
    else
    {
        # Remove the node from the cluster
        Remove-ServiceFabricNodeState -NodeName $nodename -TimeoutSec 300 -Force
    }
}
```

Dans le code **sfctl** ci-dessous, la commande suivante permet d’obtenir le nom du dernier nœud créé (**node-name**) :`sfctl node list --query "sort_by(items[*], &name)[-1].name"`

```shell
# Inform the node that it is going to be removed
sfctl node disable --node-name _nt1vm_5 --deactivation-intent 4 -t 300

# Stop the node using a random guid as our operation id
sfctl node transition --node-instance-id 131541348482680775 --node-name _nt1vm_5 --node-transition-type Stop --operation-id c17bb4c5-9f6c-4eef-950f-3d03e1fef6fc --stop-duration-in-seconds 14400 -t 300

# Remove the node from the cluster
sfctl node remove-state --node-name _nt1vm_5
```

> [!TIP]
> Utilisez les requêtes **sfctl** suivantes pour vérifier l’état de chaque étape
>
> **Vérifier l’état de la désactivation**
> `sfctl node list --query "sort_by(items[*], &name)[-1].nodeDeactivationInfo"`
>
> **Vérifier l’état de l’arrêt**
> `sfctl node list --query "sort_by(items[*], &name)[-1].isStopped"`
>

### <a name="scale-in-the-scale-set"></a>Effectuer un scale-in du groupe identique

Maintenant que le nœud Service Fabric a été supprimé du cluster, vous pouvez diminuer la taille des instances du groupe de machines virtuelles identiques. Dans l’exemple ci-dessous, la capacité du groupe identique est diminuée de 1.

```powershell
$scaleset = Get-AzVmss -ResourceGroupName SFCLUSTERTUTORIALGROUP -VMScaleSetName nt1vm
$scaleset.Sku.Capacity -= 1

Update-AzVmss -ResourceGroupName SFCLUSTERTUTORIALGROUP -VMScaleSetName nt1vm -VirtualMachineScaleSet $scaleset
```

Ce code définit la capacité sur la valeur 5.

```azurecli
# Get the name of the node with
az vmss list-instances -n nt1vm -g sfclustertutorialgroup --query [*].name

# Use the name to scale
az vmss scale -g sfclustertutorialgroup -n nt1vm --new-capacity 5
```

## <a name="behaviors-you-may-observe-in-service-fabric-explorer"></a>Comportements que vous pouvez observer dans Service Fabric Explorer
Lorsque vous effectuez un scale-out d’un cluster, Service Fabric Explorer reflète le nombre de nœuds (instances de groupe de machines virtuelles identiques) qui font partie du cluster.  Toutefois, lorsque vous effectuez un scale-in d’un cluster, vous voyez le nœud/l’instance de machine virtuelle supprimé affiché dans un état non intègre, sauf si vous appelez la [commande Remove-ServiceFabricNodeState](/powershell/module/servicefabric/remove-servicefabricnodestate) avec le nom de nœud approprié.   

Voici l’explication de ce comportement.

Les nœuds répertoriés dans Service Fabric Explorer reflètent ce que savent les services système Service Fabric (FM spécifiquement) quant au nombre de nœuds que possède/possédait le cluster. Lorsque vous effectuez un scale-in du groupe de machines virtuelles identiques, la machine virtuelle est supprimée mais le service système FM croit toujours que le nœud (mappé à la machine virtuelle qui a été supprimée) reviendra. Par conséquent, Service Fabric Explorer continue d’afficher ce nœud (même si l’état d’intégrité peut être inconnu ou un état d’erreur).

Pour vous assurer qu’un nœud est supprimé lorsqu’une machine virtuelle est supprimée, vous avez deux possibilités :

1. Choisissez un niveau de durabilité Gold ou Silver pour les types de nœuds de votre cluster, ce qui assure l’intégration de l’infrastructure. Ceci supprime automatiquement les nœuds de l’état de nos services système (FM) lors du scale-in.
Consultez les [détails sur les niveaux de durabilité ici](service-fabric-cluster-capacity.md)

> [!NOTE]
> Conservez au minimum cinq nœuds pour tout groupe de machines virtuelles identiques sur lequel le niveau de durabilité Gold ou Silver est activé. Votre cluster entrera en état d'erreur si vous passez en dessous de ce seuil et vous devrez nettoyer manuellement les nœuds supprimés.

2. Après le scale-in de l’instance de machine virtuelle, vous devez appeler l’[applet de commande Remove-ServiceFabricNodeState](/powershell/module/servicefabric/remove-servicefabricnodestate).

> [!NOTE]
> Les clusters Service Fabric nécessitent un certain nombre de nœuds actifs en permanence pour maintenir la disponibilité et préserver l’état, situation appelée « conservation du quorum ». Il est donc généralement déconseillé d’arrêter toutes les machines du cluster, sauf si vous avez d’abord effectué une [sauvegarde complète de votre état](service-fabric-reliable-services-backup-restore.md).
> 
> 

## <a name="next-steps"></a>Étapes suivantes
Lisez les documents suivants pour en savoir plus sur la planification de la capacité du cluster, la mise à niveau d’un cluster et le partitionnement des services :

* [Planification de la capacité de votre cluster](service-fabric-cluster-capacity.md)
* [Mise à niveau des clusters](service-fabric-cluster-upgrade.md)
* [Partitionnement des services avec état pour une mise à l’échelle maximale](service-fabric-concepts-partitioning.md)

<!--Image references-->
[BrowseServiceFabricClusterResource]: ./media/service-fabric-cluster-scale-in-out/BrowseServiceFabricClusterResource.png
[ClusterResources]: ./media/service-fabric-cluster-scale-in-out/ClusterResources.png

[durability]: service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster
