---
title: Planification et mise à l’échelle de la capacité pour Azure Service Fabric
description: Meilleures pratiques en matière de planification et de mise à l'échelle des clusters et applications Service Fabric
author: peterpogorski
ms.topic: conceptual
ms.date: 04/25/2019
ms.author: pepogors
ms.custom: devx-track-csharp
ms.openlocfilehash: 96cd460ddfea863eb27a1087ff59f3b87acf65d8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90531302"
---
# <a name="capacity-planning-and-scaling-for-azure-service-fabric"></a>Planification et mise à l’échelle de la capacité pour Azure Service Fabric

Avant de créer un cluster Azure Service Fabric ou de mettre à l’échelle les ressources de calcul hébergeant votre cluster, il convient d’en planifier la capacité. Pour plus d’informations sur la planification de la capacité, consultez [Planification de la capacité des clusters Service Fabric](./service-fabric-cluster-capacity.md). Pour plus de meilleures pratiques sur la mise à l’échelle du cluster, consultez [Considérations relatives à l’extensibilité de Service Fabric](/azure/architecture/reference-architectures/microservices/service-fabric#scalability-considerations).

En plus de prendre en compte les caractéristiques de type de nœud et de cluster, prévoyez que les opérations de mise à l’échelle d’un environnement de production prendront plus d’une heure. Cette considération est vraie quel que soit le nombre de machines virtuelles que vous ajoutez.

## <a name="autoscaling"></a>Mise à l’échelle automatique
Vous devez effectuer les opérations de mise à l’échelle via des modèles Azure Resource Manager, car il est recommandé de traiter [les configurations de ressources en tant que code](./service-fabric-best-practices-infrastructure-as-code.md). 

L’utilisation de la mise à l’échelle automatique via des groupes de machines virtuelles identiques empêchera votre modèle Resource Manager versionné de définir avec précision le nombre d’instance des groupes de machines virtuelles identiques. Une définition non précise augmente les risques d’opérations de mise à l’échelle non désirées lors de futurs déploiements. En règle générale, vous devez utiliser la mise à l’échelle automatique si :

* le déploiement de vos modèles Resource Manager avec une capacité appropriée déclarée ne prend pas en charge votre cas d’utilisation.
     
   Outre la mise à l’échelle manuelle, vous pouvez configurer un [pipeline d’intégration et livraison continue dans les Services Azure DevOps à l’aide de projets de déploiement de groupe de ressources Azure](../azure-resource-manager/templates/add-template-to-azure-pipelines.md). Ce pipeline est couramment déclenché par une application logique qui utilise des métriques de performances de machine virtuelle interrogées depuis l’[API REST Azure Monitor](../azure-monitor/platform/rest-api-walkthrough.md). Le pipeline met automatiquement à l’échelle les mesures souhaitées de manière efficace, tout en optimisant les modèles Resource Manager.
* Vous devez mettre à l’échelle horizontale uniquement un nœud de groupe de machines virtuelles identiques à la fois.
   
   Pour effectuer un scale-out de trois nœuds ou plus à la fois, vous devez [effectuer le scale-out d’un cluster Service Fabric en ajoutant un groupe de machines virtuelles identiques](virtual-machine-scale-set-scale-node-type-scale-out.md). Il est plus sûr d’effectuer un scale-in et un scale-out de groupe de machines virtuelles identiques à l’horizontale, un nœud à la fois.
* Vous bénéficiez d’une fiabilité Silver ou supérieure pour votre cluster Service Fabric, ainsi qu’une durabilité Silver ou supérieure pour tous les groupes identiques, où vous configurez des règles de mise à l’échelle automatique.
  
   La capacité minimale pour les règles de mise à l’échelle doit être égale ou supérieure à cinq instances de machine virtuelle. Elle doit également être égale ou supérieure au minimum de votre niveau de fiabilité pour votre type de nœud principal.

> [!NOTE]
> L’infrastructure de service avec état Service Fabric : /System/InfastructureService/<NODE_TYPE_NAME> s’exécute sur chaque type de nœud au niveau de durabilité Silver ou une version ultérieure. Il s’agit du seul service système qui est pris en charge pour s’exécuter dans Azure sur tous vos types de nœud de clusters.

> [!IMPORTANT]
> La mise à l’échelle automatique Service Fabric prend en charge les [configurations de mise à l’échelle](../virtual-machine-scale-sets/virtual-machine-scale-sets-scale-in-policy.md) des groupes de machines virtuelles identiques `Default` et `NewestVM`.

## <a name="vertical-scaling-considerations"></a>Considérations relatives à la mise à l’échelle verticale

La [mise à l’échelle verticale](./virtual-machine-scale-set-scale-node-type-scale-out.md) d’un type de nœud dans Azure Service Fabric requiert un certain nombre d’étapes et de considérations. Par exemple :

* Le cluster doit être intègre avant sa mise à l’échelle, à défaut de quoi, vous le déstabiliserez davantage.
* Le niveau de durabilité Silver ou supérieur est requis pour tous les types de nœuds Service Fabric Cluster hébergeant des services avec état.

> [!NOTE]
> Le type de nœud principal qui héberge des services système Service Fabric avec état doit présenter un niveau de durabilité Silver ou supérieur. Une fois le niveau Silver activé, les opérations de cluster telles que les mises à niveau, l’ajout ou la suppression de nœuds, etc. seront plus lentes car le système privilégie la sécurité des données par rapport à la vitesse des opérations.

La mise à l’échelle verticale d’un groupe de machines virtuelles identiques est une opération destructrice. À la place, mettez à l’échelle de façon horizontale votre cluster en ajoutant un nouveau groupe identique avec la référence SKU souhaitée. Ensuite, migrez vos services vers votre référence SKU souhaitée pour effectuer une opération de mise à l’échelle verticale sûre. Modifier la référence SKU d’une ressource de groupe de machines virtuelles identiques est une opération destructrice, car cela réinitialise vos hôtes et supprime tout état persistant localement.

Votre cluster utilise les [propriétés de nœud et contraintes de placement](./service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints) de Service Fabric pour savoir où héberger vos services d’applications. Lorsque vous mettez à l’échelle verticalement votre type de nœud principal, déclarez des valeurs de propriété identiques pour `"nodeTypeRef"`. Vous pouvez trouver ces valeurs dans l’extension de Service Fabric pour les groupes de machines virtuelles identiques. 

L’extrait suivant d’un modèle Resource Manager présente les propriétés que vous allez déclarer. Il a la même valeur pour les groupes identiques auxquels vous effectuez la mise à l’échelle, et il est pris en charge uniquement comme un service avec état temporaire de votre cluster.

```json
"settings": {
   "nodeTypeRef": ["[parameters('primaryNodetypeName')]"]
}
```

> [!NOTE]
> Ne laissez pas votre cluster s’exécuter avec plusieurs groupes de machines virtuelles identiques utilisant la même propriété `nodeTypeRef` au-delà du laps de temps nécessaire à l’opération de mise à l’échelle verticale.
>
> Validez toujours les opérations dans des environnements de test avant de modifier l’environnement de production. Par défaut, les services système de cluster Service Fabric présentent une contrainte de placement pour cibler uniquement le type de nœud principal.

Une fois les propriétés de nœud et les contraintes de placement déclarées, procédez comme suit sur une instance de machine virtuelle à la fois. Ceci permet d’arrêter correctement les services système (et vos services avec état) sur l’instance de machine virtuelle que vous supprimez pendant que de nouveaux réplicas sont créés ailleurs.

1. À partir de PowerShell, exécutez `Disable-ServiceFabricNode` avec l’intention `RemoveNode` pour désactiver le nœud que vous vous apprêtez à supprimer. Supprimez le type de nœud présentant le nombre le plus élevé. Par exemple, en présence d’un cluster à six nœuds, supprimez l’instance de machine virtuelle « MyNodeType_5 ».
2. Exécutez `Get-ServiceFabricNode` pour vous assurer que le nœud a bien été désactivé. Si ce n’est pas le cas, patientez jusqu'à ce que le nœud soit désactivé. Cette opération peut prendre plusieurs heures pour chaque nœud. Ne continuez pas tant que le nœud n'a pas été désactivé.
3. Réduisez une à une le nombre de machines virtuelles dans ce type de nœud. L'instance de machine virtuelle la plus élevée va à présent être supprimée.
4. Répétez les étapes 1 à 3 selon vos besoins, mais n’effectuez jamais de scale-in du nombre d’instances sur les types de nœuds principaux inférieur à ce que garantit le niveau de fiabilité. Pour obtenir la liste des instances recommandées, consultez [Planification de la capacité du cluster Service Fabric](./service-fabric-cluster-capacity.md).
5. Une fois que toutes les machines virtuelles ont disparu (représentées comme « Down ») fabric:/System/InfrastructureService/[nom du nœud] indique un état d’erreur. Ensuite, vous pouvez mettre à jour la ressource de cluster pour supprimer le type de nœud. Vous pouvez utiliser le déploiement du modèle ARM ou modifier la ressource de cluster par le biais d’[Azure Resource Manager](https://resources.azure.com). Cela démarre une mise à niveau du cluster qui supprimera le service fabric:/System/InfrastructureService/[type du nœud] qui est en état d’erreur.
 6. Après cela, vous pouvez supprimer le groupe de machines virtuelles identiques, mais vous verrez toujours les nœuds comme « Down » depuis la vue de Service Fabric Explorer. La dernière étape consiste à les effacer avec la commande `Remove-ServiceFabricNodeState`.

## <a name="horizontal-scaling"></a>Mise à l’échelle horizontale

Vous pouvez effectuer une mise à l’échelle horizontale [manuellement](./service-fabric-cluster-scale-in-out.md) ou [par programmation](./service-fabric-cluster-programmatic-scaling.md).

> [!NOTE]
> Si vous mettez à l’échelle un type de nœud doté d’une durabilité Silver ou Gold, le processus sera lent.

### <a name="scaling-out"></a>Montée en charge

Effectuez un scale-out d’un cluster Service Fabric en augmentant le nombre d’instances d’un groupe de machines virtuelles identiques donné. Ce scale-out peut se faire par programmation, à l'aide de `AzureClient` et de l'ID du groupe de machines virtuelles souhaité.

```csharp
var scaleSet = AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId);
var newCapacity = (int)Math.Min(MaximumNodeCount, scaleSet.Capacity + 1);
scaleSet.Update().WithCapacity(newCapacity).Apply(); 
```

Pour effectuer un scale-out manuel, mettez à jour la capacité dans la propriété de référence SKU de la ressource [Groupe de machines virtuelles identiques](/rest/api/compute/virtualmachinescalesets/createorupdate#virtualmachinescalesetosprofile) souhaitée.

```json
"sku": {
    "name": "[parameters('vmNodeType0Size')]",
    "capacity": "[parameters('nt0InstanceCount')]",
    "tier": "Standard"
}
```

### <a name="scaling-in"></a>Mise à l'échelle

La mise à l'échelle requiert une plus grande attention que la montée en charge. Par exemple :

* Les services système Service Fabric s’exécutent sur le type de nœud principal de votre cluster. Vous ne devez jamais effectuer l’arrêt ou le scale-in du nombre d’instances pour ce type de nœud de façon à avoir moins d’instances que ce que garantit le niveau de fiabilité. 
* Pour un service avec état, un certain nombre de nœuds doivent toujours fonctionner afin de maintenir la disponibilité et de conserver l’état de votre service. Au minimum, le nombre de nœuds doit être égal au nombre de jeux de réplicas cibles du service ou de la partition.

Pour effectuer un scale-in manuel, procédez comme suit :

1. À partir de PowerShell, exécutez `Disable-ServiceFabricNode` avec l’intention `RemoveNode` pour désactiver le nœud que vous vous apprêtez à supprimer. Supprimez le type de nœud présentant le nombre le plus élevé. Par exemple, en présence d’un cluster à six nœuds, supprimez l’instance de machine virtuelle « MyNodeType_5 ».
2. Exécutez `Get-ServiceFabricNode` pour vous assurer que le nœud a bien été désactivé. Si ce n’est pas le cas, patientez jusqu'à ce que le nœud soit désactivé. Cette opération peut prendre plusieurs heures pour chaque nœud. Ne continuez pas tant que le nœud n'a pas été désactivé.
3. Réduisez une à une le nombre de machines virtuelles dans ce type de nœud. L'instance de machine virtuelle la plus élevée va à présent être supprimée.
4. Répétez les étapes 1 à 3 selon vos besoins, jusqu’à approvisionner la capacité souhaitée. N’effectuez jamais de scale-in du nombre d’instances sur les types de nœuds principaux de façon à avoir moins que ce que garantit le niveau de fiabilité. Pour obtenir la liste des instances recommandées, consultez [Planification de la capacité du cluster Service Fabric](./service-fabric-cluster-capacity.md).

Pour effectuer un scale-in manuel, mettez à jour la capacité dans la propriété de référence SKU de la ressource [Groupe de machines virtuelles identiques](/rest/api/compute/virtualmachinescalesets/createorupdate#virtualmachinescalesetosprofile) souhaitée.

```json
"sku": {
    "name": "[parameters('vmNodeType0Size')]",
    "capacity": "[parameters('nt0InstanceCount')]",
    "tier": "Standard"
}
```

Vous devez préparer le nœud à l’arrêt pour effectuer un scale-in programmatiquement. Recherchez le nœud à supprimer (le nœud de l’instance la plus élevée). Par exemple :

```csharp
using (var client = new FabricClient())
{
    var mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync())
        .Where(n => n.NodeType.Equals(NodeTypeToScale, StringComparison.OrdinalIgnoreCase))
        .Where(n => n.NodeStatus == System.Fabric.Query.NodeStatus.Up)
        .OrderByDescending(n =>
        {
            var instanceIdIndex = n.NodeName.LastIndexOf("_");
            var instanceIdString = n.NodeName.Substring(instanceIdIndex + 1);
            return int.Parse(instanceIdString);
        })
        .FirstOrDefault();
```

Désactivez et supprimez le nœud en utilisant la même instance `FabricClient` (`client` dans le cas présent) et la même instance de nœud (`instanceIdString` dans le cas présent) utilisés dans le code précédent :

```csharp
var scaleSet = AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId);

// Remove the node from the Service Fabric cluster
ServiceEventSource.Current.ServiceMessage(Context, $"Disabling node {mostRecentLiveNode.NodeName}");
await client.ClusterManager.DeactivateNodeAsync(mostRecentLiveNode.NodeName, NodeDeactivationIntent.RemoveNode);

// Wait (up to a timeout) for the node to gracefully shut down
var timeout = TimeSpan.FromMinutes(5);
var waitStart = DateTime.Now;
while ((mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Up || mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Disabling) &&
        DateTime.Now - waitStart < timeout)
{
    mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync()).FirstOrDefault(n => n.NodeName == mostRecentLiveNode.NodeName);
    await Task.Delay(10 * 1000);
}

// Decrement virtual machine scale set capacity
var newCapacity = (int)Math.Max(MinimumNodeCount, scaleSet.Capacity - 1); // Check min count 

scaleSet.Update().WithCapacity(newCapacity).Apply();
```

> [!NOTE]
> Lorsque vous effectuez un scale-in d’un cluster, vous voyez l’instance de machine virtuelle/nœud supprimée qui s’affiche dans un état non intègre dans Service Fabric Explorer. Pour obtenir une explication de ce comportement, consultez [Comportements que vous pouvez observer dans Service Fabric Explorer](./service-fabric-cluster-scale-in-out.md#behaviors-you-may-observe-in-service-fabric-explorer). Vous pouvez :
> * Appelez la commande [Remove-ServiceFabricNodeState](/powershell/module/servicefabric/remove-servicefabricnodestate?view=azureservicefabricps&preserve-view=true) avec le nom de nœud approprié.
> * Déployez l’[application auxiliaire mise à l’échelle automatique Service Fabric](https://github.com/Azure/service-fabric-autoscale-helper/) sur votre cluster. Cette application permet de s’assurer que les nœuds diminués en puissance sont effacés de Service Fabric Explorer.

## <a name="reliability-levels"></a>Niveaux de fiabilité

Le [niveau de fiabilité](./service-fabric-cluster-capacity.md) est une propriété de votre ressource de cluster Service Fabric. Il ne peut pas être configuré différemment pour différents types de nœuds. Il contrôle le facteur de réplication des services système du cluster, et correspond à un paramètre au niveau de la ressource de cluster. 

Le niveau de fiabilité détermine le nombre minimal de nœuds que doit contenir votre type de nœud principal. Le niveau de fiabilité peut avoir les valeurs suivantes :

* Platinum : Exécutez les services système avec un nombre de jeux de réplicas cible égal à sept et neuf nœuds initiaux.
* Gold : Exécutez les services système avec un nombre de jeux de réplicas cible égal à sept et sept nœuds initiaux.
* Silver : Exécutez les services système avec un nombre de jeux de réplicas cible égal à cinq et cinq nœuds initiaux.
* Bronze : Exécutez les services système avec un nombre de jeux de réplicas cible égal à trois et trois nœuds initiaux.

Le niveau de fiabilité minimal recommandé est Silver.

Le niveau de fiabilité est défini dans la section des propriétés de la [ressource Microsoft.servicefabric/clusters](/azure/templates/microsoft.servicefabric/2018-02-01/clusters), comme suit :

```json
"properties":{
    "reliabilityLevel": "Silver"
}
```

## <a name="durability-levels"></a>Niveaux de durabilité

> [!WARNING]
> Les types de nœud s’exécutant avec un niveau de durabilité Bronze n’obtiennent _aucun privilège_. Les travaux d’infrastructure qui affectent vos charges de travail sans état ne seront ni arrêtés ni différés, ce qui peut affecter vos charges de travail. 
>
> Utilisez le niveau de durabilité Bronze uniquement pour les types de nœuds qui exécutent des charges de travail sans état. Pour les charges de travail de production, exécutez un niveau Silver ou supérieur pour garantir la cohérence de l’état. Sélectionnez la fiabilité qui convient en fonction des éléments contenus dans la [documentation relative à la planification de la capacité](./service-fabric-cluster-capacity.md).

Le niveau de durabilité doit être défini dans deux ressources. L’une est le profil d’extension de la [ressource de groupe de machines virtuelles identiques](/rest/api/compute/virtualmachinescalesets/createorupdate#virtualmachinescalesetosprofile) :

```json
"extensionProfile": {
    "extensions":          {
        "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
        "properties": {
            "settings": {
                "durabilityLevel": "Bronze"
            }
        }
    }
}
```

L’autre est sous `nodeTypes` dans la ressource [Microsoft.ServiceFabric/clusters](/azure/templates/microsoft.servicefabric/2018-02-01/clusters) : 

```json
"nodeTypes": [
    {
        "name": "[variables('vmNodeType0Name')]",
        "durabilityLevel": "Bronze"
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

* Créez un cluster sur des machines virtuelles ou des ordinateurs exécutant Windows Server : [Création de clusters Service Fabric pour Windows Server](service-fabric-cluster-creation-for-windows-server.md).
* Créez un cluster sur des machines virtuelles ou des ordinateurs exécutant Linux : [Créer un cluster Linux](service-fabric-cluster-creation-via-portal.md).
* Découvrez les [options de support de Service Fabric](service-fabric-support.md).

[Image1]: ./media/service-fabric-best-practices/generate-common-name-cert-portal.png