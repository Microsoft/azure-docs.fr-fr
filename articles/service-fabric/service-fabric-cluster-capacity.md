---
title: Considérations en matière de planification de la capacité du cluster Service Fabric
description: Les types de nœuds, la durabilité, la fiabilité et d’autres éléments à prendre en compte lors de la planification de votre cluster Service Fabric.
ms.topic: conceptual
ms.date: 05/21/2020
ms.author: pepogors
ms.openlocfilehash: 731dcfdf25efc4b2f44669dacd8a400037ed47f4
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576330"
---
# <a name="service-fabric-cluster-capacity-planning-considerations"></a>Considérations en matière de planification de la capacité du cluster Service Fabric

La planification de la capacité du cluster est importante pour chaque environnement de production Service Fabric. Éléments clés à prendre en compte :

* **Nombre initial et propriétés des types de nœuds *de cluster***

* Niveau de ***durabilité* de chaque type de nœud**, qui détermine les privilèges de la VM au sein de l’infrastructure Azure

* Niveau de ***fiabilité* du cluster**, qui détermine la stabilité des services système Service Fabric et de la fonction globale du cluster

Cet article vous guidera à travers les points de décision significatifs pour chacun de ces domaines.

## <a name="initial-number-and-properties-of-cluster-node-types"></a>Nombre initial et propriétés des types de nœuds de cluster

Un *type de nœud* définit la taille, le nombre et les propriétés d’un ensemble de nœuds (machines virtuelles) du cluster. Chaque type de nœud qui est défini dans un cluster Service Fabric est mappé à un [groupe de machines virtuelles identiques distinct](../virtual-machine-scale-sets/overview.md).

Comme chaque type de nœud est un groupe identique distinct, il peut faire l’objet d’une montée ou descente en puissance de manière indépendante, avoir différents jeux de ports ouverts et présenter différentes métriques de capacité. Pour plus d’informations sur la relation entre les types de nœuds et les groupes de machines virtuelles identiques, consultez [Types de nœuds de cluster Service Fabric](service-fabric-cluster-nodetypes.md).

Chaque cluster requiert un **type de nœud principal**, qui exécute des services système essentiels qui fournissent des fonctionnalités de plateforme Service Fabric. Bien qu’il soit possible d’utiliser également des types de nœuds principaux pour exécuter vos applications, il est recommandé de les dédier uniquement à l’exécution des services système.

**Les types de nœuds non principaux** peuvent être utilisés pour définir des rôles d’application (comme les services en *frontaux* et *principaux*) et pour isoler physiquement les services au sein d’un cluster. Les clusters Service Fabric peuvent avoir zéro, un ou plusieurs types de nœuds non principaux.

Le type de nœud principal est configuré à l’aide de l’attribut `isPrimary` sous la définition du type de nœud dans le modèle de déploiement Azure Resource Manager. Pour obtenir la liste complète des propriétés de type de nœud, consultez l’[Objet NodeTypeDescription](/azure/templates/microsoft.servicefabric/clusters#nodetypedescription-object). Par exemple, ouvrez un fichier *AzureDeploy.json* dans [Exemples de cluster Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/) et *Rechercher sur la page* pour rechercher l’objet `nodeTypes`.

### <a name="node-type-planning-considerations"></a>Considérations relatives à la planification du type de nœud

Le nombre de types de nœuds initiaux dépend de l’objectif de votre cluster et des applications et services qui s’y exécutent. Considérez les questions suivantes :

* ***Votre application inclut-elle plusieurs services ? Si oui, ces services doivent-ils être publics ou accessibles sur Internet ?** _

    Les applications standard contiennent un service de passerelle frontale qui reçoit des entrées d’un client, et un ou plusieurs services principaux qui communiquent avec les services frontaux avec des réseaux entre les différents services. Ces cas requièrent généralement trois types de nœuds : un type de nœud principal et deux types de nœuds non principaux (un pour le service frontal et un pour le service principal).

_ ***Vos services (qui composent votre application) ont-ils des besoins d’infrastructure différents tels qu’une RAM plus volumineuse ou des cycles processeur plus élevés ?** _

    Often, front-end service can run on smaller VMs (VM sizes like D2) that have ports open to the internet.  Computationally intensive back-end services might need to run on larger VMs (with VM sizes like D4, D6, D15) that are not internet-facing. Defining different node types for these services allow you to make more efficient and secure use of underlying Service Fabric VMs, and enables them to scale them independently. For more on estimating the amount of resources you'll need, see [Capacity planning for Service Fabric applications](service-fabric-capacity-planning.md)

_ ***L’un de vos services d’application devra-t-il évoluer au-delà de 100 nœuds ?** _

    A single node type can't reliably scale beyond 100 nodes per virtual machine scale set for Service Fabric applications. Running more than 100 nodes requires additional virtual machine scale sets (and therefore additional node types).

_ ***Votre cluster sera-t-il déployé sur plusieurs Zones de disponibilité ?** _

    Service Fabric supports clusters that span across [Availability Zones](../availability-zones/az-overview.md) by deploying node types that are pinned to specific zones, ensuring high-availability of your applications. Availability Zones require additional node type planning and minimum requirements. For details, see [Recommended topology for primary node type of Service Fabric clusters spanning across Availability Zones](service-fabric-cross-availability-zones.md#recommended-topology-for-primary-node-type-of-azure-service-fabric-clusters-spanning-across-availability-zones). 

Lorsque vous déterminez le nombre et les propriétés des types de nœuds pour la création initiale de votre cluster, gardez à l'esprit que vous pouvez toujours ajouter, modifier ou supprimer des types de nœuds (non principaux) une fois que votre cluster est déployé. [Les types de nœuds principaux peuvent également être modifiés](service-fabric-scale-up-primary-node-type.md) dans les clusters en cours d’exécution (bien que de telles opérations exigent beaucoup de planification et de prudence dans les environnements de production).

Une autre considération pour les propriétés de votre type de nœud est le niveau de durabilité, qui détermine les privilèges que les VM d'un type de nœud ont au sein de l'infrastructure Azure. Utilisez la taille des VM que vous choisissez pour votre cluster et le nombre d'instances que vous attribuez aux différents types de nœuds pour vous aider à déterminer le niveau de durabilité approprié pour chacun de vos types de nœuds, comme décrit ci-après.

## <a name="durability-characteristics-of-the-cluster"></a>Caractéristiques de durabilité du cluster

Le _niveau de durabilité* désigne les privilèges que vos VM Service Fabric ont avec l'infrastructure Azure sous-jacente. Ce privilège permet à Service Fabric de suspendre toute demande de l’infrastructure au niveau de la machine virtuelle (par exemple, un redémarrage, une réinitialisation ou une migration) qui influe sur la configuration requise du quorum Service Fabric pour les services système et vos services avec état.

> [!IMPORTANT]
> Le niveau de durabilité est défini par type de nœud. Si rien n'est spécifié, le niveau *Bronze* sera utilisé, mais il ne prévoit pas de mise à niveau automatique du système d'exploitation. Les niveaux *Argent* ou *Or* sont recommandés pour les charges de travail de production.

Le tableau ci-dessous présente les niveaux de durabilité de Service Fabric, leurs exigences et leurs prix.

| Niveau de durabilité  | Nombre minimal de machines virtuelles exigées | Tailles des machines virtuelles prises en charge                                                                  | Mises à jour que vous apportez à votre groupe de machines virtuelles identiques                               | Mises à jour et maintenance lancées par Azure                                                              | 
| ---------------- |  ----------------------------  | ---------------------------------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Or             | 5                              | Tailles des nœuds complets dédiées à un seul client (par exemple, L32s, GS5, G5, DS15_v2, D15_v2) | Peuvent être différées jusqu’à ce qu’elles soient approuvées par le cluster Service Fabric | Peuvent être suspendues pendant 2 heures par domaine de mise à niveau pour accorder du temps supplémentaire pour la récupération de réplicas à partir d’échecs précédents |
| Argent           | 5                              | Machines virtuelles avec un seul cœur ou plus avec un disque SSD local d’au moins 50 Go                      | Peuvent être différées jusqu’à ce qu’elles soient approuvées par le cluster Service Fabric | Ne peuvent pas être différées pour quelque durée que ce soit                                                    |
| Bronze          | 1                              | Machines virtuelles avec un disque SSD local d’au moins 50 Go                                              | Ne sont pas différées par le cluster Service Fabric           | Ne peuvent pas être différées pour quelque durée que ce soit                                                    |

> [!WARNING]
> Avec la durabilité Bronze, la mise à jour automatique de l'image du système d'exploitation n'est pas disponible. Bien que l'[application Patch Orchestration](service-fabric-patch-orchestration-application.md) (destinée uniquement aux clusters hébergés non-Azure) *ne soit pas recommandée* pour les niveaux de durabilité Argent ou supérieurs, il s’agit de votre seule option pour automatiser les mises à jour Windows en ce qui concerne les domaines de mise à niveau de Service Fabric.

> [!IMPORTANT]
> Quel que soit le niveau de durabilité, la [désallocation](/rest/api/compute/virtualmachinescalesets/deallocate) d’un groupe de machines virtuelles identiques a pour effet de détruire le cluster.

### <a name="bronze"></a>Bronze

Les types de nœuds s’exécutant avec un niveau de durabilité Bronze n’obtiennent aucun privilège. Cela signifie que les travaux d’infrastructure qui affectent vos charges de travail avec état ne seront ni arrêtés ni différés. Utilisez le niveau de durabilité Bronze pour les types de nœuds qui exécutent uniquement des charges de travail sans état. Pour les charges de travail de production, il est recommandé d’exécuter Silver ou un niveau supérieur.

### <a name="silver-and-gold"></a>Argent et Or

Utilisez les niveaux de durabilité Argent ou Or pour tous les types de nœuds qui hébergent des services avec état dont vous prévoyez une diminution fréquente de la taille des instances, si vous préférez que les opérations de déploiement soient retardées et que la capacité soit réduite au profit d’une simplification du processus. Les scénarios de scale-out ne doivent pas affecter votre choix du niveau de durabilité.

#### <a name="advantages"></a>Avantages

* Réduit le nombre d’étapes requises dans une opération de diminution de la taille des instances (les étapes de désactivation du nœud et Remove-ServiceFabricNodeState sont appelées automatiquement).
* Réduit le risque de perte de données résultant d’une opération de changement de taille de machine virtuelle et d’opérations de l’infrastructure Azure.

#### <a name="disadvantages"></a>Inconvénients

* Des déploiements sur les groupes de machines virtuelles identiques et d’autres ressources Azure associées peuvent être différés, dépasser le délai d’attente ou être entièrement bloqués par des problèmes au sein de votre cluster ou au niveau de l’infrastructure.
* Augmente le nombre d’[événements du cycle de vie d’un réplica](service-fabric-reliable-services-lifecycle.md) (par exemple, les permutations principales) en raison de l’automatisation des désactivations de nœud durant les opérations de l’infrastructure Azure.
* Met des nœuds hors service pendant les mises à jour logicielles de la plateforme Azure ou la maintenance du matériel. Vous pouvez voir l’état En cours de désactivation/Désactivé des nœuds pendant ces activités. La capacité de votre cluster est temporairement réduite, mais cela ne devrait pas avoir d’impact sur sa disponibilité ou sur celle de vos applications.

#### <a name="best-practices-for-silver-and-gold-durability-node-types"></a>Bonnes pratiques pour les types de nœuds de durabilité Argent et Or

Suivez ces recommandations pour la gestion des types de nœuds avec durabilité Argent ou Or :

* Veillez à la permanence de l’intégrité de votre cluster et de vos applications, et assurez-vous que les applications répondent à tous les [événements de cycle de vie de réplica de service](service-fabric-reliable-services-lifecycle.md) (par exemple, le blocage de la création d’un réplica) en temps opportun.
* Adoptez des méthodes plus sûres pour la modification des tailles de machine virtuelle (augmentation ou réduction d’échelle). La modification de la taille de la machine virtuelle sur un groupe complet nécessite une planification minutieuse et de la prudence. Pour plus d’informations, consultez [Effectuer un scale-up d’un type de nœud Service Fabric](service-fabric-scale-up-primary-node-type.md)
* Conservez au minimum cinq nœuds pour tout groupe de machines virtuelles identiques sur lequel le niveau de durabilité Gold ou Silver est activé. Votre cluster entrera en état d'erreur si vous passez en dessous de ce seuil, et vous devrez nettoyer manuellement l'état (`Remove-ServiceFabricNodeState`) des nœuds supprimés.
* Chaque groupe de machines virtuelles identiques avec le niveau de durabilité Silver ou Gold doit être mappé à son propre type de nœud dans le cluster Service Fabric. Le mappage de plusieurs groupes de machines virtuelles identiques à un type de nœud unique empêche le fonctionnement correct de la coordination entre le cluster Service Fabric et l’infrastructure Azure.
* Ne supprimez pas d’instances de machine virtuelle aléatoires. Utilisez toujours la fonctionnalité de scale-in du groupe de machines virtuelles identiques. La suppression d’instances de machine virtuelle aléatoires risque de créer des déséquilibres au sein de l’instance de machine virtuelle répartie sur les [domaines de mise à niveau](service-fabric-cluster-resource-manager-cluster-description.md#upgrade-domains) et sur les [domaines d’erreur](service-fabric-cluster-resource-manager-cluster-description.md#fault-domains). Ce déséquilibre peut nuire à la capacité du système à équilibrer correctement la charge entre les instances de service/réplicas de service.
* Si vous utilisez la fonctionnalité Mise à l’échelle automatique, définissez les règles de façon à ce que le scale-in (suppression d’instances de machine virtuelle) soit effectué nœud après nœud. La descente en puissance de plusieurs instances en même temps présente des risques.
* En cas de suppression ou de désallocation de machines virtuelles sur le type de nœud principal, ne réduisez jamais le nombre de machines virtuelles d’allouées en dessous de ce qu’exige le niveau de fiabilité. Ces opérations sont bloquées pour une durée indéterminée dans un groupe identique avec un niveau de durabilité Silver ou Gold.

### <a name="changing-durability-levels"></a>Modification des niveaux de durabilité

Dans le cadre de certaines contraintes, le niveau de durabilité du type de nœud peut être ajusté :

* Les types de nœuds avec des niveaux de durabilité Argent ou Or ne peuvent pas être rétrogradés à Bronze.
* La mise à niveau de Bronze vers Argent ou Or peut prendre quelques heures.
* Lorsque vous modifiez le niveau de durabilité, veillez à le mettre à jour à la fois dans la configuration d’extension Service Fabric de votre ressource de groupe de machines virtuelles identiques et dans la définition du type de nœud dans votre ressource de cluster Service Fabric. Ces valeurs doivent correspondre.

Une autre considération à prendre en compte lorsque la planification de la capacité est le niveau de fiabilité de votre cluster, qui détermine la stabilité des services système et de votre cluster global, comme décrit dans la section suivante.

## <a name="reliability-characteristics-of-the-cluster"></a>Caractéristiques de fiabilité du cluster

Le *niveau de fiabilité* du cluster détermine le nombre de réplicas des services système en cours d’exécution sur le type de nœud principal du cluster. Plus le nombre de réplicas est grand, plus les services système sont fiables (et par conséquent, le cluster dans son ensemble).

> [!IMPORTANT]
> Le niveau de fiabilité est défini au niveau du cluster et détermine le nombre minimal de nœuds du type de nœud principal. Les charges de travail de production nécessitent un niveau de fiabilité Argent (supérieur ou égal à cinq nœuds) ou supérieur.  

Le niveau de fiabilité peut avoir les valeurs suivantes :

* **Platine** : les services système s’exécutent avec un nombre de jeux de réplicas cible égal à neuf
* **Or** : les services système s’exécutent avec un nombre de jeux de réplicas cible égal à sept
* **Argent** : les services système s’exécutent avec un nombre de jeux de réplicas cible égal à cinq
* **Bronze** : les services système s’exécutent avec un nombre de jeux de réplicas cible égal à trois

Voici la recommandation sur le choix du niveau de fiabilité. Le nombre de nœuds initiaux est également défini sur le nombre minimal de nœuds d’un niveau de fiabilité.


| **Nombre de nœuds** | **Niveau de fiabilité** |
| --- | --- |
| 1 | *Ne spécifiez pas le paramètre `reliabilityLevel` : le système le calcule.* |
| 3 | Bronze |
| 5 ou 6| Argent |
| 7 ou 8 | Or |
| 9 et plus | Platinum |

Lorsque vous augmentez ou réduisez la taille de votre cluster (la somme des instances de machine virtuelle dans tous les types de nœuds), pensez à mettre à jour la fiabilité de votre cluster d’un niveau à l’autre. Cette opération déclenche les mises à niveau de cluster nécessaires pour modifier le nombre de jeux de réplicas des services système. Attendez que la mise à niveau en cours se termine avant d’apporter d’autres modifications au cluster, comme l’ajout de nœuds.  Vous pouvez surveiller la progression de la mise à niveau avec Service Fabric Explorer ou en exécutant [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade?view=azureservicefabricps)

### <a name="capacity-planning-for-reliability"></a>Planification des capacités de fiabilité

Les besoins en capacité de votre cluster sont déterminés par vos exigences spécifiques en matière de charge de travail et de fiabilité. Cette section fournit des conseils généraux pour vous aider à prendre en main la planification de la capacité.

#### <a name="virtual-machine-sizing"></a>Tailles des machines virtuelles

**Pour les charges de travail de production, la taille de référence de machine virtuelle recommandée est [Standard D2_V2](../virtual-machines/dv2-dsv2-series.md) (ou équivalent) avec un minimum de 50 Go de mémoire sur disque SSD local 2 cœurs et 4 Gio de mémoire.** Il est recommandé d’utiliser un disque SSD local de 50 Go au minimum, mais certaines charges de travail (comme celles qui exécutent des conteneurs Windows) nécessitent des disques plus volumineux. Lorsque vous choisissez d’autres [tailles de machines virtuelles](../virtual-machines/sizes-general.md) pour les charges de travail de production, gardez à l’esprit les contraintes suivantes :

- Les tailles de machine virtuelle de base partielle comme Standard A0 ne sont pas prises en charge.
- La *série-A* de tailles de machines virtuelles n’est pas prise en charge pour des raisons de performances.
- Les machines virtuelles de faible priorité ne sont pas prises en charge.

#### <a name="primary-node-type"></a>Type de nœud principal

Les **charges de travail de production** sur Azure nécessitent au minimum cinq nœuds principaux (instances de machine virtuelle) et un niveau de fiabilité Argent. Il est recommandé de dédier le type de nœud principal des clusters aux services système et d’utiliser des contraintes de placement pour déployer votre application sur des types de nœuds secondaires.

Les **charges de travail de test** dans Azure peuvent exécuter au minimum un ou trois nœuds principaux. Pour configurer un cluster à un nœud, assurez-vous que le paramètre `reliabilityLevel` est complètement omis dans votre modèle Resource Manager (spécifier une valeur de chaîne vide pour `reliabilityLevel` n’est pas suffisant). Si vous configurez le cluster à un nœud via le portail Azure, la configuration est automatiquement prise en charge.

> [!WARNING]
> Les clusters à un nœud s’exécutent avec une configuration spéciale sans fiabilité et lorsque la mise à l’échelle n’est pas prise en charge.

#### <a name="non-primary-node-types"></a>Types de nœuds non principaux

Le nombre minimal de nœuds pour un type de nœud non principal dépend du [niveau de durabilité](#durability-characteristics-of-the-cluster) spécifique au type de nœud. Vous devez planifier le nombre de nœuds (et le niveau de durabilité) en fonction du nombre de réplicas des applications ou des services que vous souhaitez exécuter pour le type de nœud, et selon que la charge de travail est avec ou sans état. Gardez à l’esprit que vous pouvez augmenter ou diminuer le nombre de machines virtuelles dans un type de nœud à chaque fois que vous avez déployé le cluster.

##### <a name="stateful-workloads"></a>Charges de travail avec état

Pour les charges de travail de production avec état basées sur les [collections ou les acteurs fiables](service-fabric-choose-framework.md) Service Fabric, un nombre de réplicas cibles minimum de cinq est recommandé. Cela signifie qu’à l’état stationnaire, vous vous retrouvez avec un réplica (issu d’un jeu de réplicas) dans chaque domaine d’erreur et de mise à niveau. En général, utilisez le niveau de fiabilité que vous avez défini pour les services système comme guide pour le nombre de réplicas que vous utilisez pour vos services avec état.

##### <a name="stateless-workloads"></a>Charges de travail sans état

Pour les charges de travail de production sans état, la taille minimale prise en charge pour le type de nœud non principal est de trois pour maintenir le quorum, mais une taille de nœud de type cinq est recommandée.

## <a name="next-steps"></a>Étapes suivantes

Avant de configurer votre cluster, passez en revue les `Not Allowed` [stratégies de mise à niveau de cluster](service-fabric-cluster-fabric-settings.md) pour limiter la nécessité de recréer votre cluster ultérieurement en raison des paramètres de configuration du système non modifiables.

Pour plus d’informations sur la planification d’un cluster, consultez :

* [Planification et mise à l’échelle de la capacité de calcul](service-fabric-best-practices-capacity-scaling.md)
* [Planification de la capacité pour les applications Service Fabric](service-fabric-capacity-planning.md)
* [Planification de la récupération d’urgence](service-fabric-disaster-recovery.md)

<!--Image references-->
[SystemServices]: ./media/service-fabric-cluster-capacity/SystemServices.png
