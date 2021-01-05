---
title: Service Fabric Cluster Resource Manager - Groupes d’applications
description: Vue d’ensemble de la fonctionnalité de groupe d’applications dans Service Fabric Cluster Resource Manager
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 988c7ce52125800c16aa785d5b1458604a927ecd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75452151"
---
# <a name="introduction-to-application-groups"></a>Présentation des groupes d’applications
Service Fabric Cluster Resource Manager gère habituellement les ressources de cluster en répartissant la charge (représentée par les [métriques](service-fabric-cluster-resource-manager-metrics.md)) uniformément dans le cluster. Service Fabric gère la capacité des nœuds du cluster et le cluster dans son ensemble via la [capacité](service-fabric-cluster-resource-manager-cluster-description.md). Les mesures et la capacité sont idéales pour divers types de charges de travail, mais les modèles qui utilisent beaucoup d’instances d’application Service Fabric différentes imposent parfois des conditions supplémentaires. Par exemple, vous pouvez :

- Réserver une certaine capacité sur les nœuds du cluster pour les services au sein d’une instance d’application nommée
- Limiter le nombre total de nœuds que les services au sein d’une instance d’application nommée exécutent (au lieu de les répartir sur l’ensemble du cluster)
- Définir des capacités sur l’instance d’application nommée afin de limiter le nombre de services ou la consommation totale des ressources des services qu’elle renferme

Pour répondre à ces exigences, Service Fabric Cluster Resource Manager prend en charge une fonctionnalité appelée Groupes d’applications.

## <a name="limiting-the-maximum-number-of-nodes"></a>Limitation du nombre maximal de nœuds
Le cas d’utilisation le plus simple de la capacité d’application est lorsqu’une instance d’application doit être limitée à un certain nombre de nœuds au maximum. Cela consolide tous les services au sein de cette instance d’application sur un nombre défini d’ordinateurs. La consolidation est utile lorsque vous tentez de prédire ou de limiter l’utilisation des ressources physiques par les services au sein de cette instance de l’application nommée. 

L’image suivante montre une instance d’application avec et sans un nombre maximal de nœuds définis :

<center>

![Instance d’application définissant le nombre maximal de nœuds][Image1]
</center>

Dans l’exemple de gauche, aucun nombre maximal de nœuds n’est défini pour l’application, qui inclut trois services. Cluster Resource Manager a réparti tous les réplicas sur six nœuds disponibles afin d’optimiser l’équilibre au sein du cluster (comportement par défaut). Dans l’exemple de droite, nous voyons la même application limitée à trois nœuds.

Le paramètre qui contrôle ce comportement est appelé MaximumNodes. Ce paramètre peut être défini lors de la création de l’application, ou mis à jour pour une instance d’application qui était déjà en cours d’exécution.

PowerShell

``` posh
New-ServiceFabricApplication -ApplicationName fabric:/AppName -ApplicationTypeName AppType1 -ApplicationTypeVersion 1.0.0.0 -MaximumNodes 3
Update-ServiceFabricApplication –ApplicationName fabric:/AppName –MaximumNodes 5
```

C#

``` csharp
ApplicationDescription ad = new ApplicationDescription();
ad.ApplicationName = new Uri("fabric:/AppName");
ad.ApplicationTypeName = "AppType1";
ad.ApplicationTypeVersion = "1.0.0.0";
ad.MaximumNodes = 3;
await fc.ApplicationManager.CreateApplicationAsync(ad);

ApplicationUpdateDescription adUpdate = new ApplicationUpdateDescription(new Uri("fabric:/AppName"));
adUpdate.MaximumNodes = 5;
await fc.ApplicationManager.UpdateApplicationAsync(adUpdate);

```

Dans cet ensemble de nœuds, Cluster Resource Manager ne garantit pas que certains objets de service seront positionnés ensemble ou que certains nœuds seront utilisés.

## <a name="application-metrics-load-and-capacity"></a>Mesures, charge et capacité d’application
Les groupes d’applications vous permettent également de définir les mesures associées à une instance d’application nommée donnée, ainsi que la capacité de cette instance d’application par rapport à ces mesures. Les mesures d’application vous permettent de suivre, réserver et limiter la consommation des ressources des services à l’intérieur de cette instance d’application.

Pour chaque mesure d’application, il existe deux valeurs qui peuvent être définies :

- **Capacité totale de l’application** : représente la capacité totale de l’application pour une mesure donnée. Cluster Resource Manager interdit la création de nouveaux services dans cette instance d’application qui provoqueraient le dépassement de cette valeur par la charge totale. Par exemple, supposons que l’instance d’application dispose d’une capacité de 10 et a déjà une charge de cinq. La création d’un service avec une charge totale par défaut de 10 est interdite.
- **Capacité maximale du nœud** : indique la charge totale maximale pour l’application sur un seul nœud. Si la charge dépasse cette capacité, Cluster Resource Manager déplace les réplicas vers d’autres nœuds afin que la charge diminue.


PowerShell :

``` posh
New-ServiceFabricApplication -ApplicationName fabric:/AppName -ApplicationTypeName AppType1 -ApplicationTypeVersion 1.0.0.0 -Metrics @("MetricName:Metric1,MaximumNodeCapacity:100,MaximumApplicationCapacity:1000")
```

C# :

``` csharp
ApplicationDescription ad = new ApplicationDescription();
ad.ApplicationName = new Uri("fabric:/AppName");
ad.ApplicationTypeName = "AppType1";
ad.ApplicationTypeVersion = "1.0.0.0";

var appMetric = new ApplicationMetricDescription();
appMetric.Name = "Metric1";
appMetric.TotalApplicationCapacity = 1000;
appMetric.MaximumNodeCapacity = 100;
ad.Metrics.Add(appMetric);
await fc.ApplicationManager.CreateApplicationAsync(ad);
```

## <a name="reserving-capacity"></a>Capacité réservée
Une autre utilisation courante pour les groupes d’applications consiste à s’assurer que les ressources au sein du cluster sont réservées à une instance d’application donnée. L’espace est toujours réservé lors de la création de l’instance d’application.

La réservation d’espace dans le cluster pour l’application se produit immédiatement, même lorsque :
- l’instance d’application est créée mais ne contient aucun service
- le nombre de services au sein de l’instance d’application change à chaque fois 
- les services existent mais n’utilisent pas les ressources 

La réservation de ressources pour une instance d’application implique de spécifier deux paramètres supplémentaires : *MinimumNodes* et *NodeReservationCapacity*

- **MinimumNodes** : définit le nombre minimal de nœuds sur lesquels l’instance d’application devrait s’exécuter.  
- **NodeReservationCapacity** : ce paramètre est défini pour chaque mesure de l’application. La valeur correspond à la quantité de cette mesure réservée à l’application sur n’importe quel nœud où s’exécutent les services de cette application.

La combinaison des paramètres **MinimumNodes** et **NodeReservationCapacity** garantit une réservation de charge minimale pour l’application au sein du cluster. S’il y a moins de capacité restante dans le cluster que la réservation totale requise, la création de l’application échoue. 

Voyons un exemple de réservation de capacité :

<center>

![Instances d’application définissant la capacité de réserve][Image2]
</center>

Dans l’exemple de gauche, aucune capacité n’est définie pour les applications. Cluster Resource Manager équilibre tout conformément aux règles normales.

Dans l’exemple de droite, supposons que Application1 a été créée avec les paramètres suivants :

- MinimumNodes défini sur deux
- Une mesure d’application définie avec
  - NodeReservationCapacity sur 20

PowerShell

 ``` posh
 New-ServiceFabricApplication -ApplicationName fabric:/AppName -ApplicationTypeName AppType1 -ApplicationTypeVersion 1.0.0.0 -MinimumNodes 2 -Metrics @("MetricName:Metric1,NodeReservationCapacity:20")
 ```

C#

 ``` csharp
ApplicationDescription ad = new ApplicationDescription();
ad.ApplicationName = new Uri("fabric:/AppName");
ad.ApplicationTypeName = "AppType1";
ad.ApplicationTypeVersion = "1.0.0.0";
ad.MinimumNodes = 2;

var appMetric = new ApplicationMetricDescription();
appMetric.Name = "Metric1";
appMetric.NodeReservationCapacity = 20;

ad.Metrics.Add(appMetric);

await fc.ApplicationManager.CreateApplicationAsync(ad);
```

Service Fabric réserve une capacité sur deux nœuds pour Application1 et n’autorise pas les services de Application2 à utiliser cette capacité, même si aucune charge n’est consommée en même temps par les services à l’intérieur de Application1. Cette capacité d’application réservée est considérée comme consommée et déduite de la capacité restante sur ce nœud et dans le cluster.  La réservation est immédiatement déduite de la capacité de cluster restante, mais la consommation réservée est déduite de la capacité d’un nœud spécifique uniquement lorsqu’au moins un objet service est placé sur celui-ci. Cette dernière réservation permet plus de flexibilité et une meilleure utilisation des ressources dans la mesure où les ressources sont réservées uniquement sur des nœuds lorsque cela est nécessaire.

## <a name="obtaining-the-application-load-information"></a>Obtention des informations sur la charge d’application
Pour chaque application pour laquelle une capacité d’application a été définie pour une ou plusieurs mesures, vous pouvez obtenir les informations sur la charge totale signalée par les réplicas de ses services.

PowerShell :

``` posh
Get-ServiceFabricApplicationLoadInformation –ApplicationName fabric:/MyApplication1
```

C#

``` csharp
var v = await fc.QueryManager.GetApplicationLoadInformationAsync("fabric:/MyApplication1");
var metrics = v.ApplicationLoadMetricInformation;
foreach (ApplicationLoadMetricInformation metric in metrics)
{
    Console.WriteLine(metric.ApplicationCapacity);  //total capacity for this metric in this application instance
    Console.WriteLine(metric.ReservationCapacity);  //reserved capacity for this metric in this application instance
    Console.WriteLine(metric.ApplicationLoad);  //current load for this metric in this application instance
}
```

La requête ApplicationLoad renvoie des informations de base sur la capacité d’application spécifiée pour l’application. Ces informations comprennent les informations sur les nombres minimum et maximum de nœuds, et le nombre de nœuds actuellement occupés par l’application. Elles comprennent également des informations sur chaque mesure de charge d’application, notamment :

* Nom de la métrique : nom attribué à la métrique.
* Capacité de réserve : capacité du cluster qui est réservée dans le cluster pour cette application.
* Charge de l’application : charge totale des réplicas enfants de cette application.
* Capacité d’application : valeur maximale autorisée pour la charge de l’application.

## <a name="removing-application-capacity"></a>Suppression de la capacité d’application
Une fois les paramètres de capacité d’application définis pour une application, vous pouvez les supprimer à l’aide d’API de mise à jour de l’application ou d’applets de commande PowerShell. Par exemple :

``` posh
Update-ServiceFabricApplication –Name fabric:/MyApplication1 –RemoveApplicationCapacity

```

Cette commande supprime tous les paramètres de gestion de la capacité d’application de l’instance d’application. Cela inclut MinimumNodes, MaximumNodes et les mesures de l’application, le cas échéant. L’effet de la commande est immédiat. Une fois cette commande terminée, Cluster Resource Manager utilise le comportement par défaut pour la gestion des applications. Les paramètres de capacité d’application peuvent être spécifiés à nouveau via `Update-ServiceFabricApplication`/`System.Fabric.FabricClient.ApplicationManagementClient.UpdateApplicationAsync()`.

### <a name="restrictions-on-application-capacity"></a>Restrictions sur la capacité d’application
Il existe plusieurs restrictions à respecter concernant les paramètres de capacité d’application. En cas d’erreurs de validation, aucune modification n’est appliquée.

- Tous les paramètres entiers doivent être des nombres positifs.
- La valeur du paramètre MinimumNodes ne doit jamais être supérieure à celle du paramètre MaximumNodes.
- Si les capacités d’une mesure de charge sont définies, elles doivent respecter les règles suivantes :
  - La capacité de réservation de nœud ne doit pas être supérieure à la capacité maximale de nœud. Par exemple, vous ne pouvez pas limiter la capacité de la mesure « UC » sur le nœud à deux unités et réserver trois unités sur chaque nœud.
  - Si le paramètre MaximumNodes est spécifié, le résultat du produit de sa valeur par la capacité maximale de nœud ne doit pas être supérieur à la capacité totale d’application. Par exemple, supposons que la capacité maximale de nœud pour les mesures de charge « UC » est définie sur huit. Supposons également que vous définissez le nombre maximal de nœuds sur 10. Dans ce cas, la capacité totale de l’application doit être supérieure à 80 pour cette mesure de charge.

Les restrictions sont appliquées lors de la création de l’application et des mises à jour.

## <a name="how-not-to-use-application-capacity"></a>Cas dans lesquels la capacité d’application ne doit pas être utilisée
- N’essayez pas d’utiliser les fonctionnalités de groupe d’applications pour contraindre l’application à un sous-ensemble _spécifique_ de nœuds. En d’autres termes, vous pouvez spécifier que l’application peut exécuter au maximum cinq nœuds, mais vous ne pouvez pas déterminer les cinq nœuds dans le cluster. Vous pouvez contraindre une application à des nœuds spécifiques à l’aide des contraintes de placement pour les services.
- N’essayez pas d’utiliser la capacité d’application pour vous assurer que deux services d’une même application sont placés sur les mêmes nœuds. Utilisez plutôt l’[affinité](service-fabric-cluster-resource-manager-advanced-placement-rules-affinity.md) ou les [contraintes de placement](service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints).

## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur la configuration des services, consultez la rubrique [En savoir plus sur la configuration des services](service-fabric-cluster-resource-manager-configure-services.md)
- Pour en savoir plus sur la façon dont Cluster Resource Manager gère et équilibre la charge du cluster, consultez l’article sur [l’équilibrage de la charge](service-fabric-cluster-resource-manager-balancing.md)
- Commencez au début pour [obtenir une présentation de Service Fabric Cluster Resource Manager](service-fabric-cluster-resource-manager-introduction.md)
- Pour plus d’informations sur le fonctionnement des mesures en général, consultez les informations sur les [mesures de charge Service Fabric](service-fabric-cluster-resource-manager-metrics.md)
- Cluster Resource Manager comporte de nombreuses options permettant de décrire le cluster. Pour en savoir plus sur celles-ci, consultez cet article sur la [description d’un cluster Service Fabric](service-fabric-cluster-resource-manager-cluster-description.md).

[Image1]:./media/service-fabric-cluster-resource-manager-application-groups/application-groups-max-nodes.png
[Image2]:./media/service-fabric-cluster-resource-manager-application-groups/application-groups-reserved-capacity.png
