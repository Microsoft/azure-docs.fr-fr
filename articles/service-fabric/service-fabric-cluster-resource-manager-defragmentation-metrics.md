---
title: Défragmentation des métriques dans Azure Service Fabric
description: Découvrez l’utilisation de la défragmentation, ou de l’empaquetage, en tant que stratégie pour les métriques dans Service Fabric. Cette technique est utile pour les services très volumineux.
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: bba459be4408f4a4bc438bb33b0570a91e84f2cd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75563358"
---
# <a name="defragmentation-of-metrics-and-load-in-service-fabric"></a>Défragmentation des mesures et de la charge dans Service Fabric
La stratégie par défaut de Service Fabric Cluster Resource Manager pour la gestion des mesures de charge dans le cluster consiste à répartir la charge. S’assurer que les nœuds sont utilisés de façon uniforme évite les points chauds et les points froids qui entraînent des problèmes de contention et gaspillage des ressources. La distribution des charges de travail dans le cluster est également la configuration la plus sûre afin de surmonter les défaillances, car elle permet de s’assurer qu’une défaillance n’affecte pas un trop grand pourcentage d’une charge de travail donnée. 

Service Fabric Cluster Resource Manager prend en charge une stratégie différente pour gérer la charge : la défragmentation. La défragmentation signifie qu’au lieu d’essayer de répartir l’utilisation d’une mesure dans le cluster, nous la consolidons. La consolidation n’est qu’une inversion de la stratégie d’équilibrage par défaut. Au lieu de réduire l’écart type moyen de la charge de mesure, Cluster Resource Manager tente de l’augmenter.

## <a name="when-to-use-defragmentation"></a>Quand utiliser la défragmentation
La distribution de la charge dans le cluster consomme des ressources sur chaque nœud. Certaines charges de travail créent des services qui sont particulièrement volumineux et qui consomment la plus grande partie voire la totalité d’un nœud. Dans ces cas, lorsque des charges de travail importantes sont créées, l’espace disponible sur un nœud peut s’avérer insuffisant pour les exécuter. Les charges de travail volumineuses ne posent pas de problème dans Service Fabric. Dans ces cas, Cluster Resource Manager détermine ce dont il a besoin pour réorganiser le cluster afin de faire de la place pour cette charge de travail volumineuse. Toutefois, en attendant, cette charge de travail doit attendre d’être planifiée dans le cluster.

S’il y a de nombreux services et états à déplacer, le déplacement de cette charge de travail dans le cluster peut prendre beaucoup de temps. Cette situation survient davantage si d’autres charges de travail dans le cluster sont volumineuses et que leur réorganisation prend plus de temps. L’équipe Service Fabric a mesuré les temps de création dans des simulations de ce scénario. Nous avons constaté que la création de services volumineux prend beaucoup plus de temps dès que l’utilisation de clusters se situe entre 30 et 50 %. Pour gérer ce scénario, nous avons introduit la défragmentation comme stratégie d’équilibrage. Nous avons découvert que pour les charges de travail volumineuses, notamment celles pour lesquelles le temps de création était important, la défragmentation aide réellement à la planification de ces nouvelles charges de travail dans le cluster.

Vous pouvez configurer des mesures de défragmentation pour que Cluster Resource Manager essaie de manière proactive de condenser la charge des services sur un nombre de nœuds plus réduit. Cela permet de garantir qu’il y a presque toujours de la place pour les services volumineux, sans avoir à réorganiser le cluster. Ne pas avoir à réorganiser le cluster permet de créer rapidement de grandes charges de travail.

La plupart des utilisateurs n’ont pas besoin de la défragmentation. Les services sont en règle générale de petite taille et par conséquent, il n’est pas difficile de leur trouver de la place dans le cluster. Lorsqu’une réorganisation est possible, elle se déroule rapidement car la plupart des services sont petits et peuvent être déplacés rapidement et en parallèle. Mais si vos services sont volumineux et qu’ils doivent être créés rapidement, la stratégie de défragmentation est faite pour vous. Examinons maintenant les compromis qu’implique l’utilisation de la défragmentation. 

## <a name="defragmentation-tradeoffs"></a>Compromis concernant la défragmentation
La défragmentation peut augmenter la probabilité que les défaillances aient un impact car plus de services sont exécutés sur des nœuds défaillants. La défragmentation peut également augmenter les coûts, étant donné que les ressources du cluster doivent être gardées en réserve, en attendant la création de charges de travail volumineuses.

Le diagramme suivant offre une représentation visuelle de deux clusters, l’un défragmenté et l’autre non. 

<center>

![Comparaison entre clusters équilibrés et défragmentés][Image1]
</center>

Dans le cas équilibré, tenez compte du nombre de mouvements qui seraient nécessaires pour placer un des plus grands objets de service. Dans le cluster défragmenté, la charge de travail volumineuse peut être placée sur les nœuds 4 ou 5 sans avoir à attendre le déplacement d’autres services.

## <a name="defragmentation-pros-and-cons"></a>Avantages et inconvénients de la défragmentation
Quelles sont donc ces autres compromis ? Voici un tableau succinct des éléments à considérer :

| Avantages de la défragmentation | Inconvénients de la défragmentation |
| --- | --- |
| Permet la création plus rapide de services de grande taille |Concentre la charge dans un plus petit nombre de nœuds, ce qui augmente la contention |
| Permet de diminuer le déplacement des données lors de la création |Les défaillances peuvent avoir un impact sur d’autres services et provoquer une plus grande désinscription |
| Permet de décrire en détail les exigences et la récupération d’espace |Configuration générale plus complexe de la gestion des ressources |

Vous pouvez combiner des mesures défragmentés et normales dans le même cluster. Cluster Resource Manager tente de consolider autant que possible les mesures de défragmentation tout en répartissant les autres. Les résultats obtenus en combinant des stratégies de défragmentation et d’équilibrage dépendent de plusieurs facteurs, notamment :
  - le nombre de mesures d’équilibrage par rapport au nombre de mesures de défragmentation
  - si un service utilise ces deux types de mesures 
  - les poids des mesures
  - les charges actuelles des mesures
  
L’expérimentation est nécessaire pour déterminer la configuration exacte nécessaire. Nous vous recommandons d’effectuer une mesure minutieuse de vos charges de travail avant d’activer les mesures de défragmentation en production. Cela est particulièrement vrai lorsque vous combinez des mesures de défragmentation et d’équilibrage dans le même service. 

## <a name="configuring-defragmentation-metrics"></a>Configuration de mesures de défragmentation
La configuration des mesures de défragmentation est une décision globale au niveau du cluster et des mesures individuelles peuvent être sélectionnées pour la défragmentation. Les extraits de configuration suivants montrent comment configurer des mesures pour la défragmentation. Dans ce cas, « Metric1 » est configuré comme une mesure de défragmentation, tandis que « Metric2 » continuera d’être équilibrée normalement. 

ClusterManifest.xml :

```xml
<Section Name="DefragmentationMetrics">
    <Parameter Name="Metric1" Value="true" />
    <Parameter Name="Metric2" Value="false" />
</Section>
```

via ClusterConfig.json pour les déploiements autonomes ou Template.json pour les clusters hébergés sur Azure :

```json
"fabricSettings": [
  {
    "name": "DefragmentationMetrics",
    "parameters": [
      {
          "name": "Metric1",
          "value": "true"
      },
      {
          "name": "Metric2",
          "value": "false"
      }
    ]
  }
]
```


## <a name="next-steps"></a>Étapes suivantes
- Cluster Resource Manager comporte de nombreuses options permettant de décrire le cluster. Pour en savoir plus sur celles-ci, consultez cet article sur la [description d’un cluster Service Fabric](service-fabric-cluster-resource-manager-cluster-description.md).
- Les métriques représentent la façon dont Service Fabric Cluster Resource Manager gère la consommation et la capacité du cluster. Pour en savoir plus sur ces mesures et la façon de les configurer, consultez [cet article](service-fabric-cluster-resource-manager-metrics.md)

[Image1]:./media/service-fabric-cluster-resource-manager-defragmentation-metrics/balancing-defrag-compared.png
