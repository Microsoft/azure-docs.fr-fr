---
title: Équilibrage des métriques sous-clusterisées
description: Effet des contraintes de placement sur l’équilibrage et comment le gérer
author: nipavlo
ms.topic: conceptual
ms.date: 03/15/2020
ms.author: nipavlo
ms.openlocfilehash: 7f571a851e4da147240c524b742bcd652bc54181
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82183112"
---
# <a name="balancing-of-subclustered-metrics"></a>Équilibrage des métriques sous-clusterisées

## <a name="what-is-subclustering"></a>Qu’est-ce que le sous-clustering ?

Le sous-clustering se produit quand des services avec des contraintes de placement différentes ont une métrique commune et qu’ils signalent de la charge pour celle-ci. Si la charge signalée par les services diffère de manière significative, la charge totale sur les nœuds a un écart type important, ce qui semblerait indiquer que le cluster est déséquilibré, même s’il a le meilleur équilibre possible.

## <a name="how-subclustering-affects-load-balancing"></a>Dans quelle mesure le sous-clustering affecte l’équilibrage de charge ?

Si la charge signalée par les services sur des nœuds différents diffère significativement, il est possible qu’il y ait un déséquilibre important, alors qu’il n’en est rien. En outre, si le faux déséquilibre provoqué par le sous-clustering est plus grand que le déséquilibre réel, l’algorithme d’équilibrage Resource Manager risque de produire à tort un équilibre non optimal dans le cluster.

Supposons, par exemple, que nous ayons quatre services et qu’ils signalent tous une charge pour la métrique Metric1 :

* Le service A a une contrainte de placement « NodeType==Frontend » et signale une charge de 10.
* Le service B a une contrainte de placement « NodeType==Frontend » et signale une charge de 10.
* Le service C a une contrainte de placement « NodeType==Backend » et signale une charge de 100.
* Le service D a une contrainte de placement « NodeType==Backend » et signale une charge de 100.
* De plus, nous avons quatre nœuds. Deux d’entre eux ont NodeType défini comme « Frontend » et les deux autres sont « Backend ».

Et nous avons le placement suivant :

<center>

![Exemple de placement sous-clusterisé][Image1]
</center>

Le cluster peut paraître non équilibré ; nous avons une charge importante sur les nœuds 3 et 4, mais ce placement crée le meilleur équilibre possible dans ce cas.

Resource Manager peut reconnaître les situations de sous-clustering et, dans presque tous les cas, il peut produire un équilibre optimal pour la situation donnée.

Dans certains cas exceptionnels, quand Resource Manager ne parvient pas à équilibrer de façon optimale une métrique sous-clusterisée, il détecte toujours le sous-clustering et génère un rapport d’intégrité pour vous conseiller de résoudre le problème.

## <a name="types-of-subclustering-and-how-they-are-handled"></a>Types de sous-clustering et comment ils sont gérés

Les cas de sous-clustering peuvent être classifiés en trois catégories différentes. La catégorie d’une situation de sous-clustering spécifique détermine la façon dont elle est gérée par Resource Manager.

### <a name="first-category--flat-subclustering-with-disjoint-node-groups"></a>Première catégorie : sous-clustering plat avec groupes de nœuds disjoints

Cette catégorie est la forme la plus simple de sous-clustering dans laquelle les nœuds peuvent être séparés en différents groupes et chaque service ne peut être placé que sur des nœuds de l’un de ces groupes. Chaque nœud appartient à un groupe et à un seul. La situation décrite ci-dessus appartient à cette catégorie, comme la plupart des cas de sous-clustering. 

Pour les situations appartenant à cette catégorie, Resource Manager peut produire l’équilibre optimal et aucune autre intervention n’est nécessaire.

### <a name="second-category--subclustering-with-hierarchical-node-groups"></a>Deuxième catégorie : sous-clustering avec groupes de nœuds hiérarchiques

Cette situation se produit quand un groupe de nœuds autorisés pour un service est un sous-ensemble du groupe de nœuds autorisés pour un autre service. L’exemple le plus courant de cette situation est quand un service a une contrainte de placement définie et qu’un autre service n’a pas de contrainte de placement et peut être placé sur n’importe quel nœud.

Exemple :

* Service A : aucune contrainte de placement
* Service B : contrainte de placement « NodeType==Frontend »
* Service C : contrainte de placement « NodeType==Backend »

Cette configuration crée une relation sous-ensemble/sur-ensemble entre les groupes de nœuds pour différents services.

<center>

![Sous-clusters avec relation sous-ensemble/sur-ensemble][Image2]
</center>

Dans cette situation, il y a une probabilité qu’un équilibre non optimal soit effectué.

Resource Manager reconnaît cette situation et produit un rapport d’intégrité qui vous conseille de scinder le service A en deux services : le service A1, qui peut être placé sur les nœuds Frontend, et le service A2, qui peut être placé sur les nœuds Backend. Cela nous ramène à la situation de la première catégorie qui peut être équilibrée de manière optimale.

### <a name="third-category--subclustering-with-partial-overlap-between-node-sets"></a>Troisième catégorie : sous-clustering avec chevauchement partiel entre les collections de nœuds

Cette situation se produit quand il existe un chevauchement partiel entre des collections de nœuds sur lesquels certains services peuvent être placés.

Par exemple, si nous avons une propriété de nœud appelée NodeColor et que nous avons trois nœuds :

* Nœud 1 : NodeColor=Red
* Nœud 2 : NodeColor=Blue
* Nœud 3 : NodeColor=Green

Et nous avons deux services :

* Service A : contrainte de placement « Color==Red || Color==Blue »
* Service B : contrainte de placement « Color==Blue || Color==Green »

Ainsi, le service A peut être placé sur les nœuds 1 et 2 et le service B peut être placé sur les nœuds 2 et 3.

Dans cette situation, il y a une probabilité qu’un équilibre non optimal soit effectué.

Resource Manager reconnaît cette situation et produit un rapport d’intégrité qui vous conseille de scinder certains des services.

Dans ce cas, Resource Manager n’est pas en mesure de formuler une proposition de scission des services, car plusieurs scissions peuvent être effectuées et il n’existe aucun moyen d’estimer le meilleur mode de scission.

## <a name="configuring-subclustering"></a>Configuration du sous-clustering

Le comportement de Resource Manager vis-à-vis du sous-clustering peut être modifié par le biais des paramètres de configuration suivants :
* SubclusteringEnabled : ce paramètre détermine si Resource Manager prend en compte le sous-clustering lors de l’équilibrage de charge. Si ce paramètre est désactivé, Resource Manager ignore le sous-clustering et tente d’obtenir un équilibre optimal sur un niveau global. La valeur par défaut de ce paramètre est false.
* SubclusteringReportingPolicy : ce paramètre détermine la façon dont Resource Manager émet des rapports d’intégrité pour le sous-clustering hiérarchique et avec chevauchement partiel. La valeur zéro signifie que les rapports d’intégrité sur le sous-clustering sont désactivés, la valeur « 1 » signifie que des rapports d’intégrité d’avertissement sont générés pour les cas de sous-clustering non optimaux, tandis que la valeur « 2 » génère des rapports d’intégrité « OK ». La valeur par défaut de ce paramètre est 1.

ClusterManifest.xml :

``` xml
        <Section Name="PlacementAndLoadBalancing">
            <Parameter Name="SubclusteringEnabled" Value="true" />
            <Parameter Name="SubclusteringReportingPolicy" Value="1" />
        </Section>
```

via ClusterConfig.json pour les déploiements autonomes ou Template.json pour les clusters hébergés sur Azure :

```json
"fabricSettings": [
  {
    "name": "PlacementAndLoadBalancing",
    "parameters": [
      {
          "name": "SubclusteringEnabled",
          "value": "true"
      },
      {
          "name": "SubclusteringReportingPolicy",
          "value": "1"
      },
    ]
  }
]
```

## <a name="next-steps"></a>Étapes suivantes
* Pour en savoir plus sur la façon dont Cluster Resource Manager gère et équilibre la charge du cluster, consultez l’article sur [l’équilibrage de la charge](service-fabric-cluster-resource-manager-balancing.md)
* Pour savoir comment vos services peuvent être contraints à n’être placés que sur certains nœuds, consultez [Propriétés de nœud et contraintes de placement](service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints).

[Image1]:./media/cluster-resource-manager-subclustering/subclustered-placement.png
[Image2]:./media/cluster-resource-manager-subclustering/subset-superset-nodes.png
