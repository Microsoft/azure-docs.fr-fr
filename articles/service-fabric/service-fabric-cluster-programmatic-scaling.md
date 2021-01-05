---
title: Mise à l’échelle d’Azure Service Fabric par programmation
description: Faire évoluer un cluster Azure Service Fabric par programmation, selon des déclencheurs personnalisés
author: mjrousos
ms.topic: conceptual
ms.date: 01/23/2018
ms.author: mikerou
ms.custom: devx-track-csharp
ms.openlocfilehash: 9f39bd874c1f5a1be42ca1c88e6ea2fe8df22f87
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94648206"
---
# <a name="scale-a-service-fabric-cluster-programmatically"></a>Mettre à l’échelle un cluster Service Fabric par programmation 

Les clusters Service Fabric s’exécutant dans Azure s’appuient sur les groupes de machines virtuelles identiques.  La [mise à l’échelle du cluster](./service-fabric-cluster-scale-in-out.md) décrit comment les clusters Service Fabric peuvent être mis à l’échelle manuellement ou selon des règles de mise à l’échelle automatique. Cet article décrit comment gérer les informations d’identification et monter ou réduire en puissance un cluster à l’aide du Kit de développement logiciel (SDK) Azure Fluent, qui est un scénario plus avancé. Pour obtenir une vue d’ensemble, consultez les [méthodes de programmation de coordination des opérations de mise à l’échelle Azure](service-fabric-cluster-scaling.md#programmatic-scaling). 


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="manage-credentials"></a>Gérer les informations d’identification
Un service chargé de gérer la mise à l’échelle doit pouvoir accéder aux ressources d’un groupe de machines virtuelles identiques, sans ouverture de session interactive. L’accès au cluster Service Fabric est simple si le service de mise à l’échelle modifie sa propre application Service Fabric, mais des informations d’identification sont nécessaires pour accéder au groupe identique. Pour vous connecter, vous pouvez utiliser un [principal de service](/cli/azure/create-an-azure-service-principal-azure-cli) créé avec [Azure CLI](https://github.com/azure/azure-cli).

Pour créer un principal de service, procédez comme suit :

1. Connectez-vous à Azure CLI (`az login`) en tant qu’utilisateur disposant d’un accès au groupe de machines virtuelles identiques
2. Créez le principal de service avec `az ad sp create-for-rbac`
    1. Notez l’ID d’application (appelé ID client ailleurs), le nom, le mot de passe et le client en vue d’une utilisation ultérieure.
    2. Vous aurez également besoin de votre ID d’abonnement, que vous pouvez afficher avec `az account list`.

La bibliothèque de calcul Fluent peut se connecter avec ces informations d’identification de la façon suivante (les principaux types Azure Fluent, par exemple `IAzure`, se trouvent dans le package [Microsoft.Azure.Management.Fluent](https://www.nuget.org/packages/Microsoft.Azure.Management.Fluent/)) :

```csharp
var credentials = new AzureCredentials(new ServicePrincipalLoginInformation {
                ClientId = AzureClientId,
                ClientSecret = 
                AzureClientKey }, AzureTenantId, AzureEnvironment.AzureGlobalCloud);
IAzure AzureClient = Azure.Authenticate(credentials).WithSubscription(AzureSubscriptionId);

if (AzureClient?.SubscriptionId == AzureSubscriptionId)
{
    ServiceEventSource.Current.ServiceMessage(Context, "Successfully logged into Azure");
}
else
{
    ServiceEventSource.Current.ServiceMessage(Context, "ERROR: Failed to login to Azure");
}
```

Une fois celle-ci connectée, le nombre d’instances de groupes identiques peut être interrogée avec `AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId).Capacity`.

## <a name="scaling-out"></a>Montée en charge
Le Kit de développement logiciel (SDK) Azure Compute permet d’ajouter des instances au groupe de machines virtuelles identiques, moyennant quelques appels seulement.

```csharp
var scaleSet = AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId);
var newCapacity = (int)Math.Min(MaximumNodeCount, scaleSet.Capacity + 1);
scaleSet.Update().WithCapacity(newCapacity).Apply(); 
``` 

Il est également possible de gérer la taille des groupes de machines virtuelles identiques avec des applets de commande PowerShell. [`Get-AzVmss`](/powershell/module/az.compute/get-azvmss) peut récupérer l’objet groupe de machines virtuelles identiques. La capacité actuelle peut être connue par le biais de la propriété `.sku.capacity`. Une fois la capacité fixée à la valeur souhaitée, il est possible de mettre à jour le groupe de machines virtuelles identiques dans Azure avec la commande [`Update-AzVmss`](/powershell/module/az.compute/update-azvmss).

Lorsque vous ajoutez manuellement un nœud, l’ajout d’une instance de groupe identique doit suffire pour démarrer un nouveau nœud Service Fabric, car le modèle de groupe identique contient des extensions pour ajouter automatiquement des instances au cluster Service Fabric. 

## <a name="scaling-in"></a>Mise à l'échelle

La mise à l’échelle est similaire à la montée en charge. Les modifications d’un groupe de machines virtuelles identiques sont pratiquement identiques. Mais, comme nous l’avons déjà vu, Service Fabric ne nettoie automatiquement que les nœuds supprimés ayant une durabilité de niveau or ou argent. Par conséquent, dans le cas d’une mise à l’échelle avec une durabilité de niveau bronze, il est nécessaire d’interagir avec le cluster Service Fabric pour arrêter le nœud à supprimer, puis de supprimer son état.

La préparation du nœud à l’arrêt consiste à trouver le nœud à supprimer (la dernière instance de groupe de machines virtuelles identiques ajoutée) et à le désactiver. Les instances de groupe de machines virtuelles identiques sont numérotées dans l’ordre dans lequel elles sont ajoutées. Ainsi, vous pouvez trouver les nœuds les plus récents en comparant le suffixe du numéro dans les noms des nœuds (qui correspondent aux noms des instances de groupe de machines virtuelles identiques sous-jacents). 

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

Une fois le nœud à supprimer trouvé, il peut être désactivé et supprimé à l’aide de la même instance `FabricClient` et de l’instance `IAzure` précédente.

```csharp
var scaleSet = AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId);

// Remove the node from the Service Fabric cluster
ServiceEventSource.Current.ServiceMessage(Context, $"Disabling node {mostRecentLiveNode.NodeName}");
await client.ClusterManager.DeactivateNodeAsync(mostRecentLiveNode.NodeName, NodeDeactivationIntent.RemoveNode);

// Wait (up to a timeout) for the node to gracefully shutdown
var timeout = TimeSpan.FromMinutes(5);
var waitStart = DateTime.Now;
while ((mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Up || mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Disabling) &&
        DateTime.Now - waitStart < timeout)
{
    mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync()).FirstOrDefault(n => n.NodeName == mostRecentLiveNode.NodeName);
    await Task.Delay(10 * 1000);
}

// Decrement VMSS capacity
var newCapacity = (int)Math.Max(MinimumNodeCount, scaleSet.Capacity - 1); // Check min count 

scaleSet.Update().WithCapacity(newCapacity).Apply(); 
```

Comme pour la montée en charge, des cmdlets e PowerShell de modification de la capacité des groupes de machines virtuelles identiques peuvent également être utilisées ici si une approche par scripts est préférable. Une fois l’instance de machine virtuelle supprimée, l’état du nœud Service Fabric peut être supprimé.

```csharp
await client.ClusterManager.RemoveNodeStateAsync(mostRecentLiveNode.NodeName);
```

## <a name="next-steps"></a>Étapes suivantes

Pour commencer à implémenter votre propre logique de mise à l’échelle automatique, vous devez vous familiariser avec les API utiles et les concepts suivants :

- [Augmenter ou diminuer la taille des instances d’un cluster Service Fabric à l’aide de règles de mise à l’échelle automatique](./service-fabric-cluster-scale-in-out.md)
- [Bibliothèques Azure Management pour .NET](https://github.com/Azure/azure-libraries-for-net) (utiles pour interagir avec les groupes de machines virtuelles identiques sous-jacents d’un cluster Service Fabric)
- [System.Fabric.FabricClient](/dotnet/api/system.fabric.fabricclient) (utile pour interagir avec un cluster Service Fabric et ses nœuds)
