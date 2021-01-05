---
title: Conteneurs et services de mise à l’échelle automatique Azure Service Fabric
description: Azure Service Fabric vous permet de définir des stratégies de mise à l’échelle automatique pour les services et les conteneurs.
author: radicmilos
ms.topic: conceptual
ms.date: 04/17/2018
ms.author: miradic
ms.custom: devx-track-csharp
ms.openlocfilehash: 060bb9dcdd504846c76ab4c782b2857fdddfa394
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91354800"
---
# <a name="introduction-to-auto-scaling"></a>Introduction à la mise à l’échelle automatique
La mise à l’échelle automatique est une fonctionnalité supplémentaire de Service Fabric pour mettre dynamiquement à l’échelle vos services selon la charge qu’ils présentent ou leur utilisation des ressources. La mise à l’échelle automatique offre une extensibilité idéale et permet le provisionnement d’instances ou de partitions supplémentaires de votre service à la demande. Tout le processus de mise à l’échelle automatique est automatisé et fluide, et après avoir configuré vos stratégies sur un service, aucune opération de mise à l’échelle manuelle n’est nécessaire au niveau du service. La mise à l’échelle automatique peut être activée au moment de la création du service ou à tout moment par la mise à jour du service.

La charge d’un service particulier qui varie au fil du temps correspond à un scénario courant dans lequel la mise à l’échelle automatique peut s’avérer utile. Par exemple, un service comme une passerelle peut être mis à l’échelle en fonction de la quantité de ressources nécessaires pour gérer les demandes entrantes. Examinons un exemple de ce à quoi ces règles de mise à l’échelle peuvent ressembler :
* Si toutes les instances de ma passerelle utilisent plus de deux cœurs en moyenne, augmentez la taille du service de passerelle en ajoutant une autre instance. Effectuez cette opération toutes les heures, mais sans jamais avoir plus de sept instances au total.
* Si toutes les instances de ma passerelle utilisent moins d’un demi-cœur en moyenne, diminuez la taille du service en supprimant une instance. Effectuez cette opération toutes les heures, mais sans jamais avoir moins de trois instances au total.

La mise à l’échelle automatique est prise en charge pour les conteneurs et les services Service Fabric standard. Pour que vous puissiez utiliser la mise à l’échelle automatique, il est nécessaire que le runtime Service Fabric version 6.2 ou ultérieure soit en cours d’exécution. 

Le reste de cet article décrit les stratégies de mise à l’échelle ainsi que les façons d’activer ou de désactiver la mise à l’échelle automatique, et fournit des exemples d’utilisation de cette fonctionnalité.

## <a name="describing-auto-scaling"></a>Description de la mise à l’échelle automatique
Des stratégies de mise à l’échelle automatique peuvent être définies pour chaque service dans un cluster Service Fabric. Chaque stratégie de mise à l’échelle se compose de deux parties :
* Le **déclencheur de la mise à l’échelle** décrit quand la mise à l’échelle du service est effectuée. Les conditions qui sont définies dans le déclencheur sont vérifiées régulièrement pour déterminer si un service doit être mis à l’échelle ou non.

* Le **mécanisme de mise à l’échelle** décrit comment la mise à l’échelle est effectuée quand elle est déclenchée. Le mécanisme est uniquement appliqué quand les conditions définies dans le déclencheur sont remplies.

Tous les déclencheurs qui sont actuellement pris en charge fonctionnent avec des [métriques de charge logique](service-fabric-cluster-resource-manager-metrics.md) ou avec des métriques physiques comme l’utilisation du processeur ou de la mémoire. Dans les deux cas, Service Fabric surveille la charge signalée pour la métrique et évalue le déclencheur régulièrement pour déterminer si une mise à l’échelle est nécessaire.

Deux mécanismes sont pris en charge pour la mise à l’échelle automatique. Le premier est destiné aux services sans état ou aux conteneurs dans lesquels la mise à l’échelle automatique est effectuée en ajoutant ou supprimant des [instances](service-fabric-concepts-replica-lifecycle.md). Pour les services avec et sans état, la mise à l’échelle automatique peut également être effectuée en ajoutant ou supprimant des [partitions](service-fabric-concepts-partitioning.md) nommées du service.

> [!NOTE]
> Pour l’instant, une seule stratégie de mise à l’échelle est prise en charge par service, et un seul déclencheur de mise à l’échelle est pris en charge par stratégie de mise à l’échelle.

## <a name="average-partition-load-trigger-with-instance-based-scaling"></a>Déclencheur de charge moyenne de partitions avec mise à l’échelle basée sur les instances
Le premier type de déclencheur est basé sur la charge d’instances dans une partition de service sans état. Les charges de métriques sont d’abord lissées afin d’obtenir la charge pour chaque instance d’une partition, puis la moyenne de ces valeurs est calculée pour toutes les instances de la partition. Trois facteurs déterminent quand le service est mis à l’échelle :

* Le _seuil inférieur de charge_ est une valeur qui détermine le moment où la taille du service est **diminuée**. Si la charge moyenne de toutes les instances des partitions est inférieure à cette valeur, la taille du service est diminuée.
* Le _seuil supérieur de charge_ est une valeur qui détermine le moment où la taille du service est **augmentée**. Si la charge moyenne de toutes les instances de la partition est supérieure à cette valeur, la taille du service est augmentée.
* _L’intervalle de mise à l’échelle_ détermine la fréquence à laquelle le déclencheur est vérifié. Une fois que le déclencheur est vérifié, le mécanisme est appliqué si une mise à l’échelle est nécessaire. Si la mise à l’échelle n’est pas nécessaire, aucune action n’est entreprise. Dans les deux cas, aucune nouvelle vérification du déclencheur n’est effectuée avant l’expiration suivante de l’intervalle de mise à l’échelle.

Ce déclencheur peut être utilisé uniquement avec les services sans état (conteneurs ou services Service Fabric sans état). Quand un service a plusieurs partitions, le déclencheur est évalué séparément pour chaque partition et le mécanisme spécifié est appliqué à chacune de manière indépendante. Par conséquent, dans ce cas, il est possible que la taille de certaines partitions du service soit augmentée, que la taille d’autres soit diminuée, et que la taille d’autres encore ne soit pas du tout mise à l’échelle en même temps, en fonction de leur charge.

Le seul mécanisme qui peut être utilisé avec ce déclencheur est PartitionInstanceCountScaleMechanism. Trois facteurs déterminent la façon dont ce mécanisme est appliqué :
* _Scale Increment_ (Incrément d’échelle) détermine le nombre d’instances ajoutées ou supprimées quand le mécanisme est déclenché.
* _Nombre maximum d’instances_ définit la limite supérieure de la mise à l’échelle. Si le nombre d’instances de la partition atteint cette limite, la taille du service n’est pas augmentée, quelle que soit la charge. Il est possible d’omettre cette limite en spécifiant la valeur -1 et, dans ce cas, la taille du service est augmentée autant que possible (la limite correspond au nombre de nœuds qui sont disponibles dans le cluster).
* _Nombre minimum d’instances_ définit la limite inférieure de la mise à l’échelle. Si le nombre d’instances de la partition atteint cette limite, la taille du service n’est pas diminuée, quelle que soit la charge.

## <a name="setting-auto-scaling-policy-for-instance-based-scaling"></a>Définition de la stratégie de mise à l'échelle automatique pour la mise à l'échelle basée sur les instances

### <a name="using-application-manifest"></a>Utilisation du manifeste de l’application
``` xml
<LoadMetrics>
<LoadMetric Name="MetricB" Weight="High"/>
</LoadMetrics>
<ServiceScalingPolicies>
<ScalingPolicy>
    <AveragePartitionLoadScalingTrigger MetricName="MetricB" LowerLoadThreshold="1" UpperLoadThreshold="2" ScaleIntervalInSeconds="100"/>
    <InstanceCountScalingMechanism MinInstanceCount="3" MaxInstanceCount="4" ScaleIncrement="1"/>
</ScalingPolicy>
</ServiceScalingPolicies>
```
### <a name="using-c-apis"></a>Utilisation des API C#
```csharp
FabricClient fabricClient = new FabricClient();
StatelessServiceDescription serviceDescription = new StatelessServiceDescription();
//set up the rest of the ServiceDescription
AveragePartitionLoadScalingTrigger trigger = new AveragePartitionLoadScalingTrigger();
PartitionInstanceCountScaleMechanism mechanism = new PartitionInstanceCountScaleMechanism();
mechanism.MaxInstanceCount = 3;
mechanism.MinInstanceCount = 1;
mechanism.ScaleIncrement = 1;
trigger.MetricName = "servicefabric:/_CpuCores";
trigger.ScaleInterval = TimeSpan.FromMinutes(20);
trigger.LowerLoadThreshold = 1.0;
trigger.UpperLoadThreshold = 2.0;
ScalingPolicyDescription policy = new ScalingPolicyDescription(mechanism, trigger);
serviceDescription.ScalingPolicies.Add(policy);
//as we are using scaling on a resource this must be exclusive service
//also resource monitor service needs to be enabled
serviceDescription.ServicePackageActivationMode = ServicePackageActivationMode.ExclusiveProcess
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```
### <a name="using-powershell"></a>Utilisation de PowerShell
```posh
$mechanism = New-Object -TypeName System.Fabric.Description.PartitionInstanceCountScaleMechanism
$mechanism.MinInstanceCount = 1
$mechanism.MaxInstanceCount = 6
$mechanism.ScaleIncrement = 2
$trigger = New-Object -TypeName System.Fabric.Description.AveragePartitionLoadScalingTrigger
$trigger.MetricName = "servicefabric:/_CpuCores"
$trigger.LowerLoadThreshold = 0.3
$trigger.UpperLoadThreshold = 0.8
$trigger.ScaleInterval = New-TimeSpan -Minutes 10
$scalingpolicy = New-Object -TypeName System.Fabric.Description.ScalingPolicyDescription
$scalingpolicy.ScalingMechanism = $mechanism
$scalingpolicy.ScalingTrigger = $trigger
$scalingpolicies = New-Object 'System.Collections.Generic.List[System.Fabric.Description.ScalingPolicyDescription]'
$scalingpolicies.Add($scalingpolicy)
#as we are using scaling on a resource this must be exclusive service
#also resource monitor service needs to be enabled
Update-ServiceFabricService -Stateless -ServiceName "fabric:/AppName/ServiceName" -ScalingPolicies $scalingpolicies
```

## <a name="average-service-load-trigger-with-partition-based-scaling"></a>Déclencheur de charge moyenne de services avec mise à l’échelle basée sur les partitions
Le second déclencheur est basé sur la charge de toutes les partitions d’un service. Les charges de métriques sont d’abord lissées afin d’obtenir la charge pour chaque réplica ou instance d’une partition. Pour les services avec état, la charge de la partition est considérée comme étant la charge du réplica principal alors que, pour les services sans état, la charge de la partition est la charge moyenne de toutes les instances de la partition. La moyenne de ces valeurs est calculée pour toutes les partitions du service, et cette valeur est utilisée pour déclencher la mise à l’échelle automatique. Comme pour le mécanisme précédent, trois facteurs déterminent quand le service est mis à l’échelle :

* Le _seuil inférieur de charge_ est une valeur qui détermine le moment où la taille du service est **diminuée**. Si la charge moyenne de toutes les partitions du service est inférieure à cette valeur, la taille du service est diminuée.
* Le _seuil supérieur de charge_ est une valeur qui détermine le moment où la taille du service est **augmentée**. Si la charge moyenne de toutes les partitions du service est supérieure à cette valeur, la taille du service est augmentée.
* _L’intervalle de mise à l’échelle_ détermine la fréquence à laquelle le déclencheur est vérifié. Une fois que le déclencheur est vérifié, le mécanisme est appliqué si une mise à l’échelle est nécessaire. Si la mise à l’échelle n’est pas nécessaire, aucune action n’est entreprise. Dans les deux cas, aucune nouvelle vérification du déclencheur n’est effectuée avant l’expiration suivante de l’intervalle de mise à l’échelle.

Ce déclencheur peut être utilisé à la fois avec les services avec et sans état. Le seul mécanisme qui peut être utilisé avec ce déclencheur est AddRemoveIncrementalNamedPartitionScalingMechanism. Quand la taille du service est augmentée, une nouvelle partition est ajoutée et quand la taille du service est diminuée, l’une des partitions existantes est supprimée. Il existe des restrictions qui sont vérifiées quand le service est créé ou mis à jour et la création/mise à jour du service échoue si ces conditions ne sont pas remplies :
* Le schéma de partition nommé doit être utilisé pour le service.
* Les noms de partition doivent être des nombres entiers consécutifs, comme « 0 », « 1 », etc.
* Le nom de la première partition doit être « 0 ».

Par exemple, si un service est initialement créé avec trois partitions, la seule possibilité valable pour les noms de partition est « 0 », « 1 » et « 2 ».

L’opération de mise à l’échelle automatique réelle qui est effectuée respecte aussi ce schéma de nommage :
* Si les partitions actives du service sont nommées « 0 », « 1 » et « 2 », la partition qui est ajoutée pour l’augmentation de la taille est nommée « 3 ».
* Si les partitions actives du service sont nommées « 0 », « 1 » et « 2 », la partition qui est supprimée pour la diminution de la taille est la partition nommée « 2 ».

Comme avec le mécanisme qui utilise la mise à l’échelle en ajoutant ou supprimant des instances, trois paramètres déterminent la façon dont ce mécanisme est appliqué :
* _Scale Increment_ (Incrément d’échelle) détermine le nombre de partitions ajoutées ou supprimées quand le mécanisme est déclenché.
* _Nombre maximal de partitions_ définit la limite supérieure de la mise à l’échelle. Si le nombre de partitions du service atteint cette limite, la taille du service n’est pas augmentée, quelle que soit la charge. Il est possible d’omettre cette limite en spécifiant la valeur -1 et, dans ce cas, la taille du service est augmentée autant que possible (la limite correspond à la capacité réelle du cluster).
* _Nombre minimum d’instances_ définit la limite inférieure de la mise à l’échelle. Si le nombre de partitions du service atteint cette limite, la taille du service n’est pas diminuée, quelle que soit la charge.

> [!WARNING] 
> Quand le mécanisme AddRemoveIncrementalNamedPartitionScalingMechanism est utilisé avec des services avec état, Service Fabric ajoute ou supprime des partitions **sans notification ni avertissement**. Le repartitionnement des données n’est pas effectué lorsque le mécanisme de mise à l’échelle est déclenché. Dans le cas d’une opération de scale-out, les nouvelles partitions sont vides alors que dans le cas d’une opération de scale-in, la **partition est supprimée avec toutes les données qu’elle contient**.

## <a name="setting-auto-scaling-policy-for-partition-based-scaling"></a>Définition de la stratégie de mise à l'échelle automatique pour la mise à l'échelle basée sur les partitions

### <a name="using-application-manifest"></a>Utilisation du manifeste de l’application
``` xml
<NamedPartition>
    <Partition Name="0" />
</NamedPartition>
<ServiceScalingPolicies>
    <ScalingPolicy>
        <AverageServiceLoadScalingTrigger MetricName="servicefabric:/_MemoryInMB" LowerLoadThreshold="300" UpperLoadThreshold="500" ScaleIntervalInSeconds="600"/>
        <AddRemoveIncrementalNamedPartitionScalingMechanism MinPartitionCount="1" MaxPartitionCount="3" ScaleIncrement="1"/>
    </ScalingPolicy>
</ServiceScalingPolicies>
```
### <a name="using-c-apis"></a>Utilisation des API C#
```csharp
FabricClient fabricClient = new FabricClient();
StatefulServiceUpdateDescription serviceUpdate = new StatefulServiceUpdateDescription();
AveragePartitionLoadScalingTrigger trigger = new AverageServiceLoadScalingTrigger();
PartitionInstanceCountScaleMechanism mechanism = new AddRemoveIncrementalNamedPartitionScalingMechanism();
mechanism.MaxPartitionCount = 4;
mechanism.MinPartitionCount = 1;
mechanism.ScaleIncrement = 1;
//expecting that the service already has metric NumberOfConnections
trigger.MetricName = "NumberOfConnections";
trigger.ScaleInterval = TimeSpan.FromMinutes(15);
trigger.LowerLoadThreshold = 10000;
trigger.UpperLoadThreshold = 20000;
ScalingPolicyDescription policy = new ScalingPolicyDescription(mechanism, trigger);
serviceUpdate.ScalingPolicies = new List<ScalingPolicyDescription>;
serviceUpdate.ScalingPolicies.Add(policy);
await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/AppName/ServiceName"), serviceUpdate);
```
### <a name="using-powershell"></a>Utilisation de PowerShell
```posh
$mechanism = New-Object -TypeName System.Fabric.Description.AddRemoveIncrementalNamedPartitionScalingMechanism
$mechanism.MinPartitionCount = 1
$mechanism.MaxPartitionCount = 3
$mechanism.ScaleIncrement = 2
$trigger = New-Object -TypeName System.Fabric.Description.AverageServiceLoadScalingTrigger
$trigger.MetricName = "servicefabric:/_MemoryInMB"
$trigger.LowerLoadThreshold = 5000
$trigger.UpperLoadThreshold = 10000
$trigger.ScaleInterval = New-TimeSpan -Minutes 25
$scalingpolicy = New-Object -TypeName System.Fabric.Description.ScalingPolicyDescription
$scalingpolicy.ScalingMechanism = $mechanism
$scalingpolicy.ScalingTrigger = $trigger
$scalingpolicies = New-Object 'System.Collections.Generic.List[System.Fabric.Description.ScalingPolicyDescription]'
$scalingpolicies.Add($scalingpolicy)
#as we are using scaling on a resource this must be exclusive service
#also resource monitor service needs to be enabled
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -TargetReplicaSetSize 3 -MinReplicaSetSize 2 -HasPersistedState true -PartitionNames @("0","1") -ServicePackageActivationMode ExclusiveProcess -ScalingPolicies $scalingpolicies
```

## <a name="auto-scaling-based-on-resources"></a>Mise à l’échelle automatique basée sur les ressources

Pour permettre au service du moniteur de ressource d’effectuer la mise à l’échelle basée sur des ressources réelles

``` json
"fabricSettings": [
...      
],
"addonFeatures": [
    "ResourceMonitorService"
],
```
Deux métriques représentent les ressources physiques réelles. L’une d’elles est servicefabric:/_CpuCores qui représente l’utilisation réelle du processeur (ainsi, 0,5 correspond à un demi-cœur) et l’autre est servicefabric:/_MemoryInMB qui représente l’utilisation de la mémoire en Mo.
ResourceMonitorService est chargé du suivi de l’utilisation du processeur et de la mémoire des services utilisateur. Ce service applique la moyenne mobile pondérée pour tenir compte des pics à durée de vie limitée potentiels. La surveillance des ressources est prise en charge pour les applications placées et non placées dans des conteneurs sur Windows et pour celles en conteneur sur Linux. La mise à l’échelle automatique sur les ressources est activée uniquement pour les services activés dans le [modèle à processus exclusif](service-fabric-hosting-model.md#exclusive-process-model).

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur la [scalabilité des applications](service-fabric-concepts-scalability.md).
