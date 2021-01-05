---
title: 'Service Fabric Cluster Resource Manager : Coût du déplacement'
description: En savoir plus sur le coût du déplacement pour les services Service Fabric, et sur la façon dont il peut être spécifié pour répondre à n’importe quel besoin architectural, y compris une configuration dynamique.
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.custom: devx-track-csharp
ms.openlocfilehash: 0fdcfb02851d56ed996ae4bf32671ab545782733
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89005341"
---
# <a name="service-movement-cost"></a>Coût du déplacement de services
L’un des facteurs que Service Fabric Cluster Resource Manager prend en compte pour déterminer les modifications à apporter à un cluster est leur coût. La notion de « coût » est mise en balance avec la capacité d’amélioration du cluster. Le coût est pris en compte lors du déplacement de services à des fins d’équilibrage, de défragmentation ou autres. L’objectif est de répondre aux exigences en limitant les perturbations et le coût.

Le déplacement de services coûte au minimum du temps processeur et de la bande passante réseau. Pour les services avec état, il suppose de copier l’état de ces services, ce qui consomme davantage de mémoire et de disque. Réduire le coût des solutions fournies par Azure Service Fabric Cluster Resource Manager permet de s’assurer que les ressources du cluster ne sont pas exploitées inutilement. Mais il ne faut pas non plus ignorer les solutions qui pourraient améliorer considérablement l’allocation de ressources dans le cluster.

Cluster Resource Manager dispose de deux modes de calcul et de limitation des coûts lorsqu’il gère le cluster. Le premier mécanisme consiste simplement à compter chacun des déplacements qui seraient effectués. Si deux solutions présentant à peu près le même solde (score) sont générées, Cluster Resource Manager préfère celle qui affiche le coût le plus bas (nombre total de déplacements).

Cette stratégie fonctionne bien. Mais, comme avec les charges par défaut ou statiques, il est peu probable dans n’importe quel système complexe que tous les déplacements soient égaux. Certains sont susceptibles d’être beaucoup plus coûteux.

## <a name="setting-move-costs"></a>Définir les coûts de déplacement 
Vous pouvez spécifier le coût par défaut de déplacement d’un service lors de sa création :

PowerShell :

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -DefaultMoveCost Medium
```

C# : 

```csharp
FabricClient fabricClient = new FabricClient();
StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
//set up the rest of the ServiceDescription
serviceDescription.DefaultMoveCost = MoveCost.Medium;
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

Vous pouvez également spécifier ou mettre à jour dynamiquement MoveCost pour un service après sa création : 

PowerShell : 

```posh
Update-ServiceFabricService -Stateful -ServiceName "fabric:/AppName/ServiceName" -DefaultMoveCost High
```

C# :

```csharp
StatefulServiceUpdateDescription updateDescription = new StatefulServiceUpdateDescription();
updateDescription.DefaultMoveCost = MoveCost.High;
await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/AppName/ServiceName"), updateDescription);
```

## <a name="dynamically-specifying-move-cost-on-a-per-replica-basis"></a>Spécifier dynamiquement le coût de déplacement pour chaque réplica

Les extraits de code précédents permettent tous de spécifier MoveCost pour la totalité d’un service d’un seul coup en dehors du service proprement dit. Toutefois, le coût de déplacement prend toute son utilité lorsque, pour un objet de service donné, il change au cours de sa vie. Étant donné que ce sont probablement les services eux-mêmes qui sont les mieux placés pour savoir combien coûte leur déplacement à un instant t, il existe une API qui permet aux services de signaler leur propre coût de déplacement à l’exécution. 

C# :

```csharp
this.Partition.ReportMoveCost(MoveCost.Medium);
```

## <a name="impact-of-move-cost"></a>Impact du coût de déplacement
MoveCost comporte cinq niveaux : Zero, Low, Medium, High et VeryHigh. Les règles suivantes s’appliquent :

* À l’exception de Zero et VeryHigh, ces niveaux MoveCost sont liés les uns aux autres. 
* Un coût de déplacement de zéro signifie que le déplacement est gratuit et ne doit pas compter dans le score de la solution.
* Le fait de régler votre coût de déplacement sur High ou VeryHigh ne permet *pas* de garantir que le réplica ne sera *jamais* déplacé.
* Les réplicas avec un coût de déplacement VeryHigh sont déplacés uniquement si une violation de contrainte existant dans le cluster ne peut pas être résolue d’une autre façon (même si elle nécessite de déplacer de nombreux autres réplicas pour résoudre la violation)



<center>

![Coût du déplacement comme facteur de sélection des réplicas pour les déplacements][Image1]
</center>

MoveCost vous permet de rechercher les solutions qui provoquent globalement le moins de perturbations et qui sont les plus faciles à mettre en place tout en obtenant un équilibre équivalent. La notion de coût d’un service peut être liée à beaucoup de choses. Les facteurs les plus courants pour calculer le coût de déplacement sont :

- La quantité d’état ou de données que le service doit déplacer.
- Le coût de déconnexion des clients. Le déplacement d’un réplica principal coûte généralement plus cher que celui d’un réplica secondaire.
- Le coût d’interruption d’une opération en cours. Certaines opérations au niveau du magasin de données ou en réponse à un appel du client sont coûteuses. Après un certain point, il est préférable de ne pas les arrêter si vous n’êtes pas obligé de le faire. Par conséquent, pendant l’opération, vous augmentez le coût du déplacement de cet objet de service afin de réduire la probabilité qu’il se déplace. Lorsque l’opération se termine, vous rétablissez le coût normal.

> [!IMPORTANT]
> L’utilisation du coût de déplacement VeryHigh doit être soigneusement envisagée, car elle restreint de manière significative la capacité du Gestionnaire des ressources clusters à trouver une solution de placement globalement optimale dans le cluster. Les réplicas avec un coût de déplacement VeryHigh sont déplacés uniquement si une violation de contrainte existant dans le cluster ne peut pas être résolue d’une autre façon (même si elle nécessite de déplacer de nombreux autres réplicas pour résoudre la violation)

## <a name="enabling-move-cost-in-your-cluster"></a>Activer le coût de déplacement dans un cluster
Pour que les coûts de déplacement pris en compte soient aussi précis que possible, MoveCost doit être activé dans le cluster. Sans ce paramètre, le mode par défaut, qui consiste à compter les déplacements, est utilisé pour calculer MoveCost, et les rapports MoveCost sont ignorés.


ClusterManifest.xml :

``` xml
        <Section Name="PlacementAndLoadBalancing">
            <Parameter Name="UseMoveCostReports" Value="true" />
        </Section>
```

via ClusterConfig.json pour les déploiements autonomes ou Template.json pour les clusters hébergés sur Azure :

```json
"fabricSettings": [
  {
    "name": "PlacementAndLoadBalancing",
    "parameters": [
      {
          "name": "UseMoveCostReports",
          "value": "true"
      }
    ]
  }
]
```

## <a name="next-steps"></a>Étapes suivantes
- Le Gestionnaire des ressources de cluster Service Fabric utilise des métriques pour gérer la consommation et la capacité du cluster. Pour en savoir plus sur les métriques et leur configuration, consultez [Gestion de la charge et de la consommation des ressources dans Service Fabric avec des métriques](service-fabric-cluster-resource-manager-metrics.md).
- Pour en savoir plus sur la façon dont le Gestionnaire des ressources de cluster gère et équilibre la charge du cluster, consultez [Équilibrage de votre cluster Service Fabric](service-fabric-cluster-resource-manager-balancing.md).

[Image1]:./media/service-fabric-cluster-resource-manager-movement-cost/service-most-cost-example.png
