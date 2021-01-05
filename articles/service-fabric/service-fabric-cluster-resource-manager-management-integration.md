---
title: Cluster Resource Manager - Intégration de la gestion
description: Vue d’ensemble des points d’intégration entre Cluster Resource Manager et Service Fabric Management.
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: ae80ac5833e90164fc4ff92010fd1830ae932cd2
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92174043"
---
# <a name="cluster-resource-manager-integration-with-service-fabric-cluster-management"></a>Intégration de Cluster Resource Manager à la gestion de cluster Service Fabric
Service Fabric Cluster Resource Manager n’actionne pas les mises à niveau dans Service Fabric, mais il y participe. La première manière dont Cluster Resource Manager aide à la gestion consiste à suivre l’état souhaité du cluster et des services qu’il contient. Cluster Resource Manager envoie des rapports d’intégrité lorsqu’il ne peut pas placer le cluster dans la configuration souhaitée. Par exemple, si la capacité est insuffisante, Cluster Resource Manager envoie des avertissements d’intégrité et des erreurs indiquant le problème. Le fonctionnement des mises à niveau est un autre composant de l’intégration. Cluster Resource Manager modifie légèrement son comportement pendant les mises à niveau.  

## <a name="health-integration"></a>Intégration de l’intégrité
Cluster Resource Manager assure un suivi permanent des règles que vous avez définies pour le placement de vos services. Il surveille aussi la capacité restante pour chaque métrique au niveau des nœuds et du cluster dans son ensemble. S’il ne peut pas respecter ces règles ou si la capacité est insuffisante, des erreurs et des avertissements d’intégrité sont émis. Par exemple, si la capacité d’un nœud est dépassée, Cluster Resource Manager tente de résoudre la situation en déplaçant des services. S’il ne peut pas résoudre le problème, il émet un avertissement d’intégrité indiquant pour quel nœud la capacité a été dépassée, ainsi que les mesures concernées.

Les violations des contraintes de placement constituent un autre exemple des avertissements d’intégrité de Resource Manager. Par exemple, si vous avez défini une contrainte de placement (comme `“NodeColor == Blue”`) et que Resource Manager détecte une violation de cette contrainte, il émet un avertissement d’intégrité. Cela vaut pour les contraintes personnalisées et les contraintes par défaut (comme les contraintes de domaine d’erreur et de domaine de mise à niveau).

Voici un exemple de rapport d’intégrité. Dans ce cas, le rapport d’intégrité concerne l’une des partitions de service du système. Le message d’intégrité indique que les réplicas de cette partition sont temporairement empaquetés dans des domaines de mise à niveau trop peu nombreux.

```posh
PS C:\Users\User > Get-ServiceFabricPartitionHealth -PartitionId '00000000-0000-0000-0000-000000000001'


PartitionId           : 00000000-0000-0000-0000-000000000001
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.PLB', Property='ReplicaConstraintViolation_UpgradeDomain', HealthState='Warning', ConsiderWarningAsError=false.

ReplicaHealthStates   :
                        ReplicaId             : 130766528804733380
                        AggregatedHealthState : Ok

                        ReplicaId             : 130766528804577821
                        AggregatedHealthState : Ok

                        ReplicaId             : 130766528854889931
                        AggregatedHealthState : Ok

                        ReplicaId             : 130766528804577822
                        AggregatedHealthState : Ok

                        ReplicaId             : 130837073190680024
                        AggregatedHealthState : Ok

HealthEvents          :
                        SourceId              : System.PLB
                        Property              : ReplicaConstraintViolation_UpgradeDomain
                        HealthState           : Warning
                        SequenceNumber        : 130837100116930204
                        SentAt                : 8/10/2015 7:53:31 PM
                        ReceivedAt            : 8/10/2015 7:53:33 PM
                        TTL                   : 00:01:05
                        Description           : The Load Balancer has detected a Constraint Violation for this Replica: fabric:/System/FailoverManagerService Secondary Partition 00000000-0000-0000-0000-000000000001 is
                        violating the Constraint: UpgradeDomain Details: UpgradeDomain ID -- 4, Replica on NodeName -- Node.8 Currently Upgrading -- false Distribution Policy -- Packing
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Ok->Warning = 8/10/2015 7:13:02 PM, LastError = 1/1/0001 12:00:00 AM
```

Voici ce que nous apprend ce message d’intégrité :

1. Tous les réplicas proprement dits sont sains ; Chacun possède `AggregatedHealthState : Ok`.
2. La contrainte de distribution de domaine de mise à niveau n’est actuellement pas respectée. Cela signifie qu’un domaine de mise à niveau particulier compte plus de réplicas que prévu pour cette partition.
3. L’identité du nœud qui contient le réplica à l’origine de la violation. Dans ce cas, il s’agit du nœud nommé *Node.8* .
4. Si cette partition fait actuellement l’objet d’une mise à niveau (« Currently Upgrading -- false »).
5. La stratégie de distribution pour ce service : « Distribution Policy -- Packing ». Ceci est régi par la [stratégie de positionnement](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md#requiring-replica-distribution-and-disallowing-packing) `RequireDomainDistribution`. *Packing* indique que, dans ce cas, DomainDistribution n’était _pas_ obligatoire. Nous savons donc que la stratégie de positionnement n’a pas été spécifiée pour ce service. 
6. La date et l’heure du rapport (10/08/2015 19:13:02)

Ces informations alimentent les alertes qui se déclenchent en production pour vous informer qu’un problème est survenu. Elles servent aussi à détecter et à mettre un terme aux mises à niveau incorrectes. Dans ce cas, nous aimerions savoir pourquoi Resource Manager a décidé de placer les réplicas dans le domaine de mise à niveau. En règle générale, l’empaquetage est temporaire et peut résulter, par exemple, de l’arrêt des nœuds dans les autres domaines de mise à niveau.

Supposons que Cluster Resource Manager tente de placer certains services, mais qu’aucune solution ne fonctionne. Quand les services ne peuvent pas être placés, c’est généralement pour l’une des raisons suivantes :

1. Une situation temporaire empêche de placer correctement l’instance ou le réplica de ce service.
2. Les conditions de placement du service ne peuvent pas être satisfaites.

En pareil cas, les rapports d’intégrité de Cluster Resource Manager vous aident à déterminer pourquoi le service ne peut pas être placé. C’est ce que nous appelons une séquence d’élimination de contrainte. Au cours de ce processus, le système passe en revue les contraintes configurées affectant le service et les enregistrements qu’elles éliminent. Lorsque les services ne peuvent pas être placés, vous pouvez ainsi voir quels nœuds ont été éliminés et pour quelle raison.

## <a name="constraint-types"></a>Types de contrainte
Examinons les différentes contraintes qui apparaissent dans ces rapports d’intégrité. Les messages de contrôle d’intégrité liés à ces contraintes s’affichent quand les réplicas ne peuvent pas être placés.

* **ReplicaExclusionStatic** et **ReplicaExclusionDynamic**  : ces contraintes indiquent qu’une solution a été rejetée, car deux objets de service de la même partition auraient dû être placés sur le même nœud. Cette opération n’est pas autorisée, car une défaillance de ce nœud aurait un trop fort impact sur cette partition. ReplicaExclusionStatic et ReplicaExclusionDynamic appliquent pratiquement la même règle et les différences importent peu. Si vous constatez qu’une séquence d’élimination de contrainte contient la contrainte ReplicaExclusionDynamic ou ReplicaExclusionStatic, Cluster Resource Manager considère qu’il n’y a pas suffisamment de nœuds. Les solutions restantes doivent utiliser ces placements non valides, qui ne sont pas autorisés. Les autres contraintes de la séquence indiquent généralement pourquoi les nœuds sont éliminés en premier lieu.
* **PlacementConstraint**  : si vous voyez ce message, cela signifie que certains nœuds ne correspondant pas aux contraintes de placement du service ont été éliminés. Nous avons suivi les contraintes de placement actuellement configurées dans le cadre de ce message. Ceci est normal si une contrainte de placement est définie. Cependant, si la contrainte de placement entraîne à tort l’élimination d’un trop grand nombre de nœuds, c’est de cette façon que vous le remarquerez.
* **NodeCapacity**  : cette contrainte signifie que Cluster Resource Manager n’a pas pu placer les réplicas sur les nœuds indiqués, car ils auraient été mis en surcapacité.
* **Affinity**  : cette contrainte indique que les réplicas n’ont pas été placés sur les nœuds affectés pour ne pas enfreindre la contrainte d’affinité. Plus d’informations sur la contrainte Affinity, lisez [cet article](service-fabric-cluster-resource-manager-advanced-placement-rules-affinity.md)
* **FaultDomain** et **UpgradeDomain**  : cette contrainte élimine des nœuds si le placement du réplica sur les nœuds indiqués se traduit par une compression dans un domaine de mise à niveau ou d’erreur spécifique. Plusieurs exemples décrivant cette contrainte sont présentés dans la rubrique [Contraintes des domaines d’erreur et de mise à niveau, et comportement résultant](service-fabric-cluster-resource-manager-cluster-description.md)
* **PreferredLocation**  : cette contrainte ne doit normalement pas retirer des nœuds de la solution, car elle s’exécute par défaut en tant qu’optimisation. La contrainte d’emplacement par défaut est aussi présente pendant les mises à niveau. À ces occasions, elle déplace les services à l’emplacement d’origine qui était le leur au moment où la mise à niveau a démarré.

## <a name="blocklisting-nodes"></a>Mise des nœuds en liste de blocage
Cluster Resource Manager peut aussi afficher un message de contrôle d’intégrité quand des nœuds sont mis en liste de blocage. La mise des nœuds en liste de blocage peut être considérée comme une contrainte provisoire qui s’applique automatiquement. Les nœuds sont mis en liste de blocage quand ils subissent des défaillances à répétition pendant le lacement d’instances de ce type de service. Les nœuds sont mis en liste de blocage par type de service, si bien qu’un nœud peut être placé dans une liste de blocage pour un type de service, mais pas pour un autre. 

Vous verrez que la mise en liste de blocage intervient souvent pendant le développement : certains bogues provoquent l’arrêt de votre hôte de service au démarrage, et Service Fabric tente à plusieurs reprises de créer l’hôte de service, mais l’erreur se répète. Après plusieurs tentatives, le nœud est mis en liste de blocage et Cluster Resource Manager essaie de créer le service à un autre emplacement. Si cette erreur se répète sur plusieurs nœuds, il est possible que tous les nœuds du cluster se retrouvent dans la liste de blocage. La mise en liste de blocage peut aussi concerner un trop grand nombre de nœuds, au point qu’il n’en reste pas suffisamment pour lancer le service à l’échelle souhaitée. Il est fort probable dans ce cas que Cluster Resource Manager affiche des erreurs ou des avertissements supplémentaires pour indiquer que le service présente moins de réplicas ou d’instances que prévu, ainsi que des messages de contrôle d’intégrité décrivant la défaillance à l’origine de la mise en liste de blocage.

La mise en liste de blocage n’est pas une condition permanente. Après quelques minutes, le nœud est retiré de la liste de blocage et Service Fabric réactive les services sur ce nœud. En revanche, si le service continue d’échouer, le nœud est remis en liste de blocage pour ce type de service. 

### <a name="constraint-priorities"></a>Priorités de contrainte

> [!WARNING]
> Il n’est pas recommandé de modifier les priorités des contraintes. De plus, cela peut avoir des répercutions défavorables sur votre cluster. Vous trouverez ci-dessous des informations fournies à titre indicatif sur les priorités des contraintes par défaut et leur comportement. 
>

Face à toutes ces contraintes, vous vous dites peut-être : « Ce qui compte avant tout dans mon système, ce sont les contraintes de domaine d’erreur. Pour être certain que la contrainte de domaine d’erreur va bien être respectée, je vais enfreindre les autres contraintes ».

Les contraintes peuvent être configurées avec différents niveaux de priorité, Ces règles sont les suivantes :

   - « stricte » (0)
   - « souple » (1)
   - « optimisation » (2)
   - « désactivé » (-1). 
   
Par défaut, la plupart des contraintes sont configurées comme des contraintes strictes.

Il n’est pas courant de modifier la priorité des contraintes. Dans certaines circonstances, la modification des priorités des contraintes était une nécessité, généralement pour contourner un bogue ou un comportement qui avait un impact sur l’environnement. En règle générale, la flexibilité de l’infrastructure de priorité de contrainte a bien fonctionné, mais elle n’est pas souvent nécessaire. La plupart du temps, les priorités par défaut sont conservées. 

Les niveaux de priorité ne signifient pas qu’une contrainte donnée _sera_ enfreinte, ni qu’elle sera toujours respectée. Les priorités des contraintes définissent l’ordre dans lequel les contraintes sont appliquées. Les priorités permettent de définir des compromis quand il est impossible de satisfaire toutes les contraintes. En règle générale, si rien de particulier ne se passe dans l’environnement, les contraintes peuvent toutes être satisfaites. Les conflits de contraintes ou la présence d’un grand nombre de défaillances simultanées sont des exemples de scénarios qui peuvent aboutir à des violations de contraintes.

Dans des situations avancées, vous pouvez modifier les priorités des contraintes. Supposons, par exemple, que vous vouliez faire en sorte que la contrainte Affinity soit enfreinte pour résoudre des problèmes de capacité de nœuds. Pour ce faire, vous pouvez définir la priorité de la contrainte Affinity sur « souple » (1) et laisser la contrainte Capacity sur « stricte » (0).

Les valeurs de priorité par défaut associées aux différentes contraintes sont indiquées dans le fichier config suivant :

ClusterManifest.xml

```xml
        <Section Name="PlacementAndLoadBalancing">
            <Parameter Name="PlacementConstraintPriority" Value="0" />
            <Parameter Name="CapacityConstraintPriority" Value="0" />
            <Parameter Name="AffinityConstraintPriority" Value="0" />
            <Parameter Name="FaultDomainConstraintPriority" Value="0" />
            <Parameter Name="UpgradeDomainConstraintPriority" Value="1" />
            <Parameter Name="PreferredLocationConstraintPriority" Value="2" />
        </Section>
```

via ClusterConfig.json pour les déploiements autonomes ou Template.json pour les clusters hébergés sur Azure :

```json
"fabricSettings": [
  {
    "name": "PlacementAndLoadBalancing",
    "parameters": [
      {
          "name": "PlacementConstraintPriority",
          "value": "0"
      },
      {
          "name": "CapacityConstraintPriority",
          "value": "0"
      },
      {
          "name": "AffinityConstraintPriority",
          "value": "0"
      },
      {
          "name": "FaultDomainConstraintPriority",
          "value": "0"
      },
      {
          "name": "UpgradeDomainConstraintPriority",
          "value": "1"
      },
      {
          "name": "PreferredLocationConstraintPriority",
          "value": "2"
      }
    ]
  }
]
```

## <a name="fault-domain-and-upgrade-domain-constraints"></a>Contraintes de domaine d’erreur et de domaine de mise à niveau
Cluster Resource Manager vise à maintenir une répartition harmonieuse des services entre les domaines d’erreur et de mise à niveau. Pour cela, une contrainte est intégrée dans le moteur de Cluster Resource Manager. Pour plus d’informations sur leur utilisation et leur comportement spécifique, consultez l’article relatif à la [configuration de cluster](service-fabric-cluster-resource-manager-cluster-description.md#fault-and-upgrade-domain-constraints-and-resulting-behavior).

Cluster Resource Manager peut être amené à empaqueter deux réplicas dans un domaine de mise à niveau pour gérer les mises à niveau, les défaillances ou d’autres violations de contraintes. Normalement, l’empaquetage dans des domaines d’erreur ou de mise à niveau intervient uniquement quand plusieurs défaillances ou évolutions se produisent dans le système et empêchent un positionnement correct. Si vous voulez empêcher l’empaquetage même dans ces situations, vous pouvez utiliser la  [stratégie de positionnement](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md#requiring-replica-distribution-and-disallowing-packing)`RequireDomainDistribution`. Notez que cela peut avoir des effets secondaires et nuire à la disponibilité et à la fiabilité du service. Pensez-y avant toute décision dans ce sens.

Si l’environnement est correctement configuré, toutes les contraintes sont entièrement respectées, même pendant les mises à niveau. L’essentiel est que Cluster Resource Manager surveille vos contraintes. Quand il détecte une violation, il le signale immédiatement et essaie de corriger le problème.

## <a name="the-preferred-location-constraint"></a>La contrainte d’emplacement par défaut
La contrainte PreferredLocation est un peu différente, car elle remplit deux fonctions différentes. D’une part, cette contrainte est utilisée pendant les mises à niveau d’applications. Cluster Resource Manager gère automatiquement cette contrainte pendant les mises à niveau. Elle permet de vérifier que les réplicas retrouvent leur emplacement initial à l’issue des mises à niveau. D’autre part, la contrainte PreferredLocation est utilisée avec la [`PreferredPrimaryDomain` stratégie de positionnement](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md). Il s’agit dans les deux cas d’optimisations. De ce fait, la contrainte PreferredLocation est la seule contrainte à être définie par défaut sur « optimisation ».

## <a name="upgrades"></a>Mises à niveau
Cluster Resource Manager est également utile durant les mises à niveau d’applications et de cluster, où il exécute deux tâches :

* vérifier que les règles du cluster ne sont pas compromises ;
* essayer de faire en sorte que la mise à niveau se déroule sans problème.

### <a name="keep-enforcing-the-rules"></a>Application continue des règles
L’élément principal à retenir est que les règles (les contraintes strictes, comme les contraintes de placement et les capacités) sont toujours appliquées pendant les mises à niveau. Les contraintes de placement s’assurent que vos charges de travail s’exécutent uniquement où elles y sont autorisées, notamment pendant les mises à niveau. Quand les services sont très ralentis, les mises à niveau peuvent prendre plus de temps. Quand le service ou son nœud est arrêté pour une mise à jour, plusieurs destinations possibles peuvent être proposées.

### <a name="smart-replacements"></a>Remplacements actifs
Lorsqu’une mise à niveau démarre, Resource Manager prend une capture instantanée de l’organisation actuelle du cluster. À mesure que chaque domaine de mise à niveau aboutit, il tente de rétablir l’organisation initiale des services qui figuraient dans ce domaine de mise à niveau. De cette manière, il existe tout au plus deux transitions possibles pour un service pendant la mise à niveau. Il y a une sortie du nœud affecté et un retour dans celui-ci. Le fait de rétablir le cluster ou le service tel qu’il était avant la mise à niveau garantit également que la mise à niveau n’affecte pas la disposition du cluster. 

### <a name="reduced-churn"></a>Évolution limitée
Pendant les mises à niveau, Cluster Resource Manager désactive aussi l’équilibrage. Le fait de suspendre l’équilibrage empêche les réactions inutiles envers la mise à niveau, comme le déplacement de services vers des nœuds qui ont été vidés pour la mise à niveau. Si la mise à niveau en question est une mise à niveau du cluster, alors l’ensemble du cluster n’est pas équilibré au cours de la mise à niveau. La vérification des contraintes reste active, seul le mouvement basé sur l’équilibrage proactif des métriques est désactivé.

### <a name="buffered-capacity--upgrade"></a>Capacité mise en mémoire tampon et mise à niveau
En règle générale, vous voulez que la mise à niveau arrive à son terme même si le cluster est ralenti ou proche de la saturation. La gestion de la capacité du cluster est encore plus importante pendant les mises à niveau qu’en temps normal. Selon le nombre de domaines de mise à niveau, entre 5 et 20 pour cent de la capacité doit être migrée pendant le déploiement de la mise à niveau sur le cluster. Ce travail doit aller quelque part. C’est dans ce contexte que la notion de [capacité mise en mémoire tampon](service-fabric-cluster-resource-manager-cluster-description.md#node-buffer-and-overbooking-capacity) prend tout son sens. La capacité mise en mémoire tampon est respectée en phase de fonctionnement normal. Cluster Resource Manager peut remplir les nœuds jusqu’à leur capacité totale (en consommant la mémoire tampon) au cours des mises à niveau, si nécessaire.

## <a name="next-steps"></a>Étapes suivantes
* Commencez au début pour [obtenir une présentation de Service Fabric Cluster Resource Manager](service-fabric-cluster-resource-manager-introduction.md)
