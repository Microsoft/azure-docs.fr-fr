---
title: Équilibrer votre cluster Azure Service Fabric
description: Présentation de l’équilibrage de votre cluster avec Service Fabric Cluster Resource Manager.
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 192aca589c3b1e660667dbe8377afe7802b56f17
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93146192"
---
# <a name="balancing-your-service-fabric-cluster"></a>Équilibrage de votre cluster Service Fabric
Service Fabric Cluster Resource Manager permet de modifier la charge dynamique, de réagir aux ajouts ou aux suppressions de nœuds ou de services. Il corrige également automatiquement les violations de contrainte et rééquilibre de façon proactive le cluster. Mais à quelle fréquence ces actions sont-elles effectuées, et quel en est l’élément déclencheur ?

Il existe trois différentes catégories de tâches exécutées par Cluster Resource Manager. Il s'agit de :

1. Positionnement : cette étape a trait au placement des réplicas avec état ou des instances sans état manquants. Le positionnement inclut non seulement les nouveaux services, mais aussi les réplicas avec état ou les instances sans état qui ont échoué. La suppression et l’annulation des réplicas ou des instances sont gérées ici.
2. Vérifications de contrainte : cette étape recherche les violations des différentes contraintes (règles) de placement au sein du système et apporte les corrections nécessaires. Le non-dépassement de la capacité des nœuds et le respect des contraintes de positionnement d’un service sont des exemples de règles.
3. Équilibrage : cette étape vérifie si un rééquilibrage est nécessaire en fonction du niveau d’équilibrage désiré qui a été configuré pour différentes mesures. Si c’est le cas, elle tente de trouver une disposition plus équilibrée dans le cluster.

## <a name="configuring-cluster-resource-manager-timers"></a>Configuration des minuteurs Cluster Resource Manager
Le premier ensemble de contrôles autour d’équilibrage sont un ensemble de minuteurs. Ces minuteurs déterminent la fréquence à laquelle Cluster Resource Manager examine le cluster et mène des actions correctives.

Chacun de ces différents types de corrections offerts par Cluster Resource Manager est contrôlé par un minuteur différent qui détermine sa fréquence. Lorsque chaque minuteur se déclenche, la tâche est planifiée. Par défaut, Resource Manager :

* analyse son état et applique des mises à jour (comme l’enregistrement de l’arrêt d’un nœud) tous les dixièmes de seconde ;
* définit l’indicateur de contrôle du placement chaque seconde ;
* définit l’indicateur de contrôle de la contrainte chaque seconde
* définit l’indicateur d’équilibrage toutes les cinq secondes.

Voici des exemples de configuration qui définit ces minuteurs :

ClusterManifest.xml :

``` xml
        <Section Name="PlacementAndLoadBalancing">
            <Parameter Name="PLBRefreshGap" Value="0.1" />
            <Parameter Name="MinPlacementInterval" Value="1.0" />
            <Parameter Name="MinConstraintCheckInterval" Value="1.0" />
            <Parameter Name="MinLoadBalancingInterval" Value="5.0" />
        </Section>
```

via ClusterConfig.json pour les déploiements autonomes ou Template.json pour les clusters hébergés sur Azure :

```json
"fabricSettings": [
  {
    "name": "PlacementAndLoadBalancing",
    "parameters": [
      {
          "name": "PLBRefreshGap",
          "value": "0.10"
      },
      {
          "name": "MinPlacementInterval",
          "value": "1.0"
      },
      {
          "name": "MinConstraintCheckInterval",
          "value": "1.0"
      },
      {
          "name": "MinLoadBalancingInterval",
          "value": "5.0"
      }
    ]
  }
]
```

À l’heure actuelle, Cluster Resource Manager effectue uniquement l’une de ces actions à la fois, séquentiellement. C’est pourquoi nous faisons référence à ces minuteurs en tant qu’« intervalles minimaux » et les actions effectuées lorsque les minuteurs sont déclenchés représentent des « indicateurs de paramètre ». Par exemple, Cluster Resource Manager traite les demandes de création de services en attente avant d’équilibrer le cluster. Comme en témoignent les intervalles de temps par défaut spécifiés, Cluster Resource Manager analyse fréquemment ce qu’il a à faire. Par conséquent, les modifications apportées à la fin de chaque étape sont généralement mineures. L’apport fréquent de modifications mineures rend Cluster Resource Manager réactif aux événements qui se produisent dans le cluster. Les compteurs par défaut assurent en partie un traitement par lot, car un grand nombre d’événements du même type ont tendance à se produire simultanément. 

Par exemple, lorsque des nœuds échouent, ils se produisent simultanément dans des domaines d’erreur entiers. Tous ces échecs sont capturés à la prochaine mise à jour de l’état après *PLBRefreshGap*. Les corrections sont déterminées lorsque les prochaines opérations de placement, de contrôle des contraintes et d’équilibrage sont exécutées. Par défaut, Cluster Resource Manager n’analyse pas les modifications qui ont eu lieu sur plusieurs heures dans le cluster et n’essaie pas de les traiter toutes en même temps. En effet, cela entraînerait des pics d’activité momentanés.

Cluster Resource Manager a également besoin d’informations supplémentaires pour déterminer si le cluster est déséquilibré. Nous disposons pour cela de deux autres éléments de configuration : les *seuils d’équilibrage* et les *seuils d’activité*.

## <a name="balancing-thresholds"></a>Seuils d’équilibrage
Un seuil d’équilibrage est le principal contrôle utilisé pour déclencher le rééquilibrage. Le seuil d’équilibrage pour une métrique est un _ratio_. Si la charge d’une mesure sur le nœud le plus chargé, divisée par la quantité de charge sur le nœud le moins chargé, dépasse la valeur *BalancingThreshold* de cette mesure, le cluster est considéré comme déséquilibré. L’équilibrage est alors déclenché lorsque Cluster Resource Manager effectue sa vérification suivante. Le minuteur *MinLoadBalancingInterval* détermine la fréquence à laquelle Cluster Resource Manager doit vérifier si un rééquilibrage est nécessaire. Cette vérification ne signifie pas que quelque chose se produit. 

Les seuils d’équilibrage sont définis par métrique dans le cadre de la définition du cluster. Pour plus d’informations sur les mesures, consultez [cet article](service-fabric-cluster-resource-manager-metrics.md).

ClusterManifest.xml

```xml
    <Section Name="MetricBalancingThresholds">
      <Parameter Name="MetricName1" Value="2"/>
      <Parameter Name="MetricName2" Value="3.5"/>
    </Section>
```

via ClusterConfig.json pour les déploiements autonomes ou Template.json pour les clusters hébergés sur Azure :

```json
"fabricSettings": [
  {
    "name": "MetricBalancingThresholds",
    "parameters": [
      {
          "name": "MetricName1",
          "value": "2"
      },
      {
          "name": "MetricName2",
          "value": "3.5"
      }
    ]
  }
]
```

<center>

![Exemple de seuil d’équilibrage][Image1]
</center>

Dans cet exemple, chaque service utilise une unité d’une mesure donnée. Dans l’exemple du haut, la charge maximale sur un nœud est égale à 5 et la valeur minimale à 2. Supposons que le seuil d’équilibrage pour cette mesure soit égal à 3. Étant donné que le ratio dans le cluster est de 5/2 = 2,5, soit une valeur inférieure au seuil d’équilibrage de 3, le cluster est équilibré. Aucun équilibrage n’est déclenché lorsque Cluster Resource Manager effectue sa vérification.

Dans l’exemple du bas, la charge maximale sur un nœud est égale à 10 et la valeur minimale à 2, ce qui donne un ratio de 5. Cette valeur étant supérieure au seuil d’équilibrage de 3 spécifié pour cette mesure, l’exécution d’un rééquilibrage sera planifiée lors du prochain déclenchement du minuteur d’équilibrage. Dans une telle situation, une partie de la charge est généralement répartie sur le nœud Node3. Comme Service Fabric Cluster Resource Manager utilise une approche prudente, une partie de la charge peut également être répartie sur le nœud Node2. 

<center>

![Actions de l’exemple de seuil d’équilibrage][Image2]
</center>

> [!NOTE]
> « L’équilibrage » adopte deux stratégies différentes pour gérer la charge de votre cluster. La stratégie par défaut utilisée par Cluster Resource Manager consiste à répartir la charge sur les nœuds du cluster. L’autre stratégie est la [défragmentation](service-fabric-cluster-resource-manager-defragmentation-metrics.md). La défragmentation se produit en même temps que l’équilibrage. Les stratégies d’équilibrage et de défragmentation peuvent être utilisées pour différentes mesures au sein du même cluster. Un service peut avoir à la fois des mesures d’équilibrage et de défragmentation. Pour les mesures de défragmentation, le ratio des charges dans le cluster déclenche un rééquilibrage lorsqu’il se situe _sous_ le seuil d’équilibrage. 
>

L’obtention d’une valeur inférieure au seuil d’équilibrage ne constitue pas un objectif explicite. Les seuils d’équilibrage ne sont qu’un *déclencheur*. Lors de l’équilibrage, Cluster Resource Manager détermine les améliorations possibles, le cas échéant. Une analyse d’équilibrage ne signifie pas que des données sont déplacées. Parfois, le cluster est déséquilibré mais comporte trop de contraintes pour être corrigé. Les améliorations peuvent également nécessiter des mouvements trop [coûteux](service-fabric-cluster-resource-manager-movement-cost.md).

## <a name="activity-thresholds"></a>seuils d’activité
Parfois, même si les nœuds sont relativement déséquilibrés, la quantité *totale* de charge dans le cluster est faible. La charge insuffisante peut correspondre à une baisse temporaire ou être due au fait qu’il s’agit d’un nouveau cluster qui vient juste d’être amorcé. Dans les deux cas, il n’est pas forcément judicieux de passer du temps à équilibrer le cluster, car les gains seront minimes. Si le cluster a fait l’objet d’un équilibrage, vous consommerez des ressources réseau et de calcul pour déplacer des éléments sans que cela ait un impact *réel*. Pour éviter ces mouvements inutiles, il existe un autre contrôle appelé « seuil d’activité ». Les seuils d’activité vous permettent de spécifier une limite inférieure absolue pour l’activité. Lorsqu’aucun nœud ne dépasse ce seuil, l’équilibrage n’est pas déclenché, même si le seuil d’équilibrage est atteint.

Supposons que nous conservons notre seuil d’équilibrage de 3 pour cette mesure. Supposons également que nous avons un seuil d’activité de 1 536. Dans le premier cas, bien que le cluster soit déséquilibré selon le seuil d’équilibrage, comme aucun nœud n’atteint le seuil d’activité, il ne se passe rien. Dans l’exemple du bas, le nœud Node1 dépasse le seuil d’activité. Comme le seuil d’équilibrage et le seuil d’activité pour la mesure sont tous deux dépassés, un équilibrage est planifié. Prenons pour exemple le schéma suivant : 

<center>

![Exemple de seuil d’activité][Image3]
</center>

Au même titre que les seuils d’équilibrage, les seuils d’activité sont définis par métrique par l’intermédiaire de la définition du cluster :

ClusterManifest.xml

``` xml
    <Section Name="MetricActivityThresholds">
      <Parameter Name="Memory" Value="1536"/>
    </Section>
```

via ClusterConfig.json pour les déploiements autonomes ou Template.json pour les clusters hébergés sur Azure :

```json
"fabricSettings": [
  {
    "name": "MetricActivityThresholds",
    "parameters": [
      {
          "name": "Memory",
          "value": "1536"
      }
    ]
  }
]
```

Les seuils d’équilibrage et d’activité sont tous deux liés à une mesure spécifique : l’équilibrage n’est déclenché que si le seuil d’équilibrage et le seuil d’activité sont dépassés pour la même mesure.

> [!NOTE]
> Lorsqu’elle n’est pas spécifiée, la valeur du seuil d’équilibrage d’une métrique est de 1, et la valeur du seuil d’activité est de 0. Cela signifie que le gestionnaire de ressources du cluster va tenter de maintenir le parfait équilibre de cette métrique pour toutes les charges. Si vous utilisez des métriques personnalisées, nous vous recommandons de définir explicitement vos propres seuils d’équilibrage et d’activité. 
>

## <a name="balancing-services-together"></a>Équilibrage de plusieurs services en même temps
La détermination de l’état de déséquilibre du cluster est une décision qui porte sur l’ensemble du cluster. Cependant, la procédure suivie pour corriger un déséquilibre consiste à déplacer des instances et des réplicas de service individuels. Logique, n’est-ce pas ? Si la mémoire est empilée sur un nœud, plusieurs réplicas ou instances peuvent être impliqués. Pour corriger le déséquilibre, il peut donc être nécessaire de déplacer les réplicas avec état ou les instances sans état qui utilisent la mesure déséquilibrée.

Occasionnellement pourtant, un service qui n’était pas déséquilibré lui-même est déplacé (voir la discussion sur les poids locaux et globaux plus haut). Pourquoi un service serait-il déplacé si toutes les mesures de ce service ont été équilibrées ? Examinons un exemple :

- Prenons par exemple quatre services : Service1, Service2, Service3 et Service4. 
- Service1 signale les mesures Metric1 et Metric2. 
- Service2 signale les mesures Metric2 et Metric3. 
- Service3 signale les mesures Metric3 et Metric4.
- Service4 signale la mesure Metric99. 

Vous voyez certainement où je veux en venir : il s’agit d’une chaîne ! Nous n’avons pas vraiment quatre services indépendants, mais plutôt trois services qui sont liés et un qui est indépendant.

<center>

![Schéma qui montre comment équilibrer plusieurs services en même temps.][Image4]
</center>

En raison de cette chaîne, il est donc possible qu’un déséquilibre dans les mesures 1 à 4 provoque le déplacement de réplicas ou d’instances appartenant aux services 1 à 3. Nous savons également qu’un déséquilibre de la mesure Metric1, Metric2 ou Metric3 ne peut pas provoquer de déplacements pour le service Service4. Cela n’aurait aucun intérêt, puisque le déplacement de réplicas ou d’instances appartenant au service Service4 n’a pas la moindre incidence sur l’équilibre de la mesure Metric1, Metric2 ou Metric3.

Cluster Resource Manager identifie automatiquement les services associés. Ajouter, supprimer ou modifier les mesures des services peut avoir un impact sur leurs relations. Par exemple, entre deux équilibrages, la mesure Metric2 peut avoir été mise à jour pour être supprimée de la configuration du service Service2. La chaîne entre Service1 et Service2 est alors rompue. Au lieu de deux groupes de services liés, vous en avez à présent trois :

<center>

![Schéma qui montre que le Gestionnaire des ressources clusters identifie les services associés.][Image5]
</center>

## <a name="next-steps"></a>Étapes suivantes
* Les métriques représentent la façon dont Service Fabric Cluster Resource Manager gère la consommation et la capacité du cluster. Pour en savoir plus sur ces mesures et la façon de les configurer, consultez [cet article](service-fabric-cluster-resource-manager-metrics.md)
* Le coût du mouvement est une façon de signaler à Cluster Resource Manager que certains services sont plus coûteux à déplacer que d’autres. Pour plus d’informations sur le coût lié aux déplacements, reportez-vous à [cet article](service-fabric-cluster-resource-manager-movement-cost.md).
* Cluster Resource Manager a plusieurs limitations que vous pouvez configurer pour ralentir l’évolution dans le cluster. Elles ne sont normalement pas nécessaires mais, si vous en avez besoin, vous pouvez en savoir plus sur ces limitations [ici](service-fabric-cluster-resource-manager-advanced-throttling.md)
* Le Gestionnaire des ressources clusters peut reconnaître et gérer le sous-clustering (situation qui se présente parfois en cas de recours à des contraintes de placement et à un équilibrage). Pour connaître l’impact potentiel du sous-clustering sur l’équilibrage et apprendre à le gérer, cliquez [ici](cluster-resource-manager-subclustering.md).

[Image1]:./media/service-fabric-cluster-resource-manager-balancing/cluster-resrouce-manager-balancing-thresholds.png
[Image2]:./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-threshold-triggered-results.png
[Image3]:./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-activity-thresholds.png
[Image4]:./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-services-together1.png
[Image5]:./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-services-together2.png
