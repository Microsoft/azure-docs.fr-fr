---
title: Récupération d’urgence Azure Service Fabric
description: Azure Service Fabric offre des fonctionnalités permettant de gérer les urgences. Cet article décrit les types d’incidents qui peuvent se produire et comment les gérer.
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 9c258d8d0a7aa26c96ab4f64017770ebdd153e60
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86257512"
---
# <a name="disaster-recovery-in-azure-service-fabric"></a>Récupération d’urgence dans Azure Service Fabric
Pour fournir une haute disponibilité, il est essentiel que les services puissent survivre à tous les types d’échecs. Ceci est particulièrement important pour les échecs inattendus et incontrôlables. 

Cet article décrit certains modes d’échec courants qui peuvent aboutir à une situation critique s’ils ne sont pas modélisés et gérés correctement. Il traite également des atténuations de risques et des actions requises en cas d’incident. L’objectif est de limiter ou d’éliminer le risque de temps d’arrêt ou de perte de données en cas de défaillances, planifiées ou non.

## <a name="avoiding-disaster"></a>Éviter les incidents
L’objectif principal d’Azure Service Fabric est de vous aider à modéliser votre environnement et vos services de sorte que les échecs courants ne se transforment pas en incidents. 

En général, il existe deux types de scénarios d’incident/échec :
- Défaillances matérielles et logicielles
- Défaillances opérationnelles

### <a name="hardware-and-software-faults"></a>Défaillances matérielles et logicielles
Les défaillances matérielles et logicielles sont imprévisibles. Le moyen le plus simple de survivre aux défaillances est d’exécuter plusieurs copies du service au-delà des limites des défaillances matérielles ou logicielles. 

Par exemple, si votre service s’exécute sur une seule machine, l’échec de cette machine est une situation d’urgence pour le service. Un moyen simple d’éviter ce type d’incident consiste à garantir que le service s’exécute sur plusieurs ordinateurs. Des tests sont également nécessaires pour garantir que la défaillance d’une machine ne va pas perturber le service en cours d’exécution. La planification de la capacité garantit qu’une instance de remplacement peut être créée ailleurs et que la réduction de la capacité ne va pas surcharger les services restants. 

Le même modèle fonctionne, quel que soit le type d’échec que vous cherchez à éviter. Par exemple, si vous cherchez surtout à éviter les défaillances d’un réseau SAN, vous pouvez exécuter le service sur plusieurs réseaux SAN. Si vous cherchez surtout à éviter la perte d’un rack de serveurs, vous pouvez exécuter le service sur plusieurs racks. Si vous cherchez surtout à éviter la perte de centres de données, vous pouvez exécuter le service sur plusieurs régions Azure, plusieurs zones de disponibilité Azure, ou sur vos propres centres de données. 

Lorsqu’un service couvre plusieurs instances physiques (machines, racks, centres de données, régions), vous êtes toujours susceptible de subir des défaillances simultanées. Toutefois, les défaillances uniques, voire multiples, d’un type particulier (par exemple, une seule machine virtuelle ou une liaison réseau défaillante) sont gérées automatiquement et ne sont donc plus des « incidents ». 

Service Fabric fournit des mécanismes pour le développement du cluster et gère la restauration des nœuds et services ayant échoué. Service Fabric permet également d’exécuter de nombreuses instances de vos services pour empêcher des défaillances non planifiées d’aboutir à des situations d’urgence réelles.

Dans certains cas, il n’est pas possible d’effectuer un déploiement suffisamment étendu pour couvrir toutes les défaillances. Par exemple, un tel déploiement peut nécessiter davantage de ressources matérielles dans lesquelles vous n’êtes pas disposé à investir, après évaluation des risques de défaillances. Lorsque vous utilisez des applications distribuées, des tronçons de communication ou des coûts de réplication d’état supplémentaires vers des sites distants peuvent provoquer une latence inacceptable. Votre décision doit être prise en fonction de chaque application. 

Pour les défaillances logicielles en particulier, la défaillance peut se situer au niveau du service que vous essayez de mettre à l’échelle. Dans ce cas, l’utilisation de copies supplémentaires ne permet pas d’empêcher l’incident, car la condition d’échec concerne toutes les instances.

### <a name="operational-faults"></a>Défaillances opérationnelles
Même si votre service est exécuté dans le monde entier avec de nombreuses redondances, les situations d’urgence sont toujours possibles. Par exemple, une personne peut reconfigurer le nom DNS du service ou le supprimer par inadvertance. 

Par exemple, imaginons que vous disposiez d’un service Service Fabric avec état, et que quelqu’un ait supprimé ce service par inadvertance. À moins que d’autres mesures d’atténuation des risques ne soient en place, le service et son état ont désormais disparu. Ces incidents opérationnels (de type « Désolé… ») nécessitent des mesures d’atténuation des risques et une procédure de récupération différentes de celles employées pour les défaillances inattendues. 

Les méthodes recommandées pour éviter ce type de défaillances opérationnelles sont les suivantes :
- Restreindre l’accès opérationnel à l’environnement.
- Réaliser un audit strict des opérations dangereuses.
- Imposer l’automatisation, empêcher les modifications manuelles ou hors bande et valider les modifications par rapport à l’environnement avant de les appliquer.
- Faire en sorte que les opérations de destruction soient réversibles. Les opérations réversibles ne sont pas appliquées immédiatement ou peuvent être annulées dans un certain délai.

Service Fabric fournit des mécanismes pour éviter les défaillances opérationnelles, tels que le contrôle d’accès [en fonction du rôle](service-fabric-cluster-security-roles.md) pour les opérations de cluster. Toutefois, pour éviter la plupart de ces défaillances, des efforts d’organisation et des systèmes supplémentaires sont nécessaires. Service Fabric fournit des mécanismes permettant de surmonter les défaillances opérationnelles, notamment [la sauvegarde et la restauration des services avec état](service-fabric-backuprestoreservice-quickstart-azurecluster.md).

## <a name="managing-failures"></a>Gestion des défaillances
L’objectif de Service Fabric est la gestion automatique des défaillances. Cependant, la gestion de certains types de défaillances requiert du code supplémentaire. Certains types de défaillances _ne doivent pas_ être gérés automatiquement, pour des raisons de sécurité et de continuité opérationnelle. 

### <a name="handling-single-failures"></a>Gestion des défaillances uniques
Les machines uniques peuvent connaître une défaillance pour diverses raisons. L’origine de la défaillance est parfois matérielle, comme dans le cas d’une défaillance de l’alimentation ou du matériel réseau. Certaines défaillances peuvent se situer au niveau du logiciel. Il s’agit notamment de défaillances du système d’exploitation ou du service. Service Fabric détecte automatiquement ce type de défaillance, notamment lorsqu’une machine est isolée des autres machines à cause de problèmes de réseau.

Quel que soit le type du service, si vous n’exécutez qu’une seule instance de ce service et que son code échoue pour une raison quelconque, le service connaîtra un temps d’arrêt. 

Pour gérer les défaillances uniques, le plus simple est de faire en sorte que vos services soient exécutés sur plusieurs nœuds par défaut. Pour les services sans état, assurez-vous que `InstanceCount` est supérieur à 1. Pour les services avec état, il est recommandé d’affecter une valeur égale à 3 à `TargetReplicaSetSize` et à `MinReplicaSetSize`. Le fait d’exécuter plusieurs copies du code de votre service permet au service de gérer automatiquement les défaillances uniques. 

### <a name="handling-coordinated-failures"></a>Gestion des défaillances multiples
Des défaillances multiples dans un cluster peuvent être dues à des défaillances, planifiées ou non, ou à des modifications de l’infrastructure, ou bien à des modifications logicielles planifiées. Service Fabric modélise les zones d’infrastructure qui rencontrent des défaillances multiples comme des *domaines d’erreur*. Les zones qui subissent des modifications logicielles coordonnées sont modélisées comme des *domaines de mise à niveau*. Pour plus d’informations sur les domaines d’erreur, les domaines de mise à niveau et la topologie de cluster, consultez [Décrire un cluster Service Fabric à l’aide du gestionnaire des ressources clusters](service-fabric-cluster-resource-manager-cluster-description.md).

Par défaut, Service Fabric prend en compte les domaines d’erreur et de mise à niveau lorsqu’il planifie où vos services doivent s’exécuter. Par défaut, Service Fabric tente de faire en sorte que vos services s’exécutent sur plusieurs domaines d’erreur et de mise à niveau. Ainsi, si des modifications planifiées ou non planifiées se produisent, vos services restent disponibles. 

Par exemple, supposons que la défaillance d’une source d’alimentation provoque la défaillance simultanée de toutes les machines d’un rack. Si plusieurs copies du service sont exécutées, la perte de plusieurs machines en raison d’une défaillance de domaine d’erreur aboutit à la défaillance unique d’un service. C’est pourquoi la gestion des domaines d’erreur et de mise à niveau est essentielle pour garantir la haute disponibilité de vos services. 

Lorsque vous exécutez Service Fabric dans Azure, les domaines d’erreur et de mise à niveau sont gérés automatiquement. Ce n’est pas forcément le cas dans d’autres environnements. Si vous créez vos propres clusters locaux, veillez à mapper et à planifier correctement la disposition du domaine d’erreur.

Les domaines de mise à niveau sont utiles pour modéliser les zones qui vont en même temps faire l’objet une mise à niveau logicielle. Pour cette raison, les domaines de mise à niveau définissent souvent les limites du logiciel pendant les mises à niveau planifiées. Les mises à niveau de Service Fabric et de vos services suivent le même modèle. Pour plus d’informations sur le déploiement des mises à niveau, les domaines de mise à niveau et le modèle d’intégrité Service Fabric qui permet d’empêcher des modifications non souhaitées d’affecter le cluster et votre service, consultez :

 - [Mise à niveau de l’application](service-fabric-application-upgrade.md)
 - [Tutoriel sur la mise à niveau des applications](service-fabric-application-upgrade-tutorial.md)
 - [Modèle d’intégrité Service Fabric](service-fabric-health-introduction.md)

Vous pouvez visualiser la disposition de votre cluster à l’aide de la carte de cluster fournie dans [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) :

<center>

![Nœuds répartis dans les domaines d’erreur et visibles dans Service Fabric Explorer][sfx-cluster-map]
</center>

> [!NOTE]
> La modélisation des zones de défaillance, la propagation des mises à niveau, l’exécution de plusieurs instances du code et de l’état de votre service, les règles de sélection élective pour garantir l’exécution de vos services sur plusieurs domaines d’erreur et de mise à niveau, et la surveillance intégrée de l’intégrité ne constituent que *quelques exemples* des fonctionnalités fournies par Service Fabric pour empêcher les problèmes et les défaillances opérationnels de se transformer en incidents. 
>

### <a name="handling-simultaneous-hardware-or-software-failures"></a>Gestion des défaillances matérielles ou logicielles simultanées
Nous parlons de défaillances uniques. Comme vous pouvez le voir, elles sont faciles à gérer, aussi bien pour les services avec que sans état. Il vous suffit pour cela d’exécuter plusieurs copies du code (et de l’état) sur les différents domaines d’erreur et de mise à niveau. 

Les défaillances multiples, simultanées et aléatoires sont également possibles. Ces défaillances sont davantage susceptibles d’aboutir à un temps d’arrêt ou à un véritable incident.


#### <a name="stateless-services"></a>Service sans état
Le nombre d’instances pour un service sans état indique le nombre d’instances souhaité à exécuter. En cas d’échec d’une ou de la totalité des instances, Service Fabric répond en créant automatiquement des instances de remplacement sur d’autres nœuds. Service Fabric continue à créer des remplacements jusqu’à ce que le service atteigne de nouveau le nombre d’instances souhaité.

Par exemple, supposons que le service sans état a une valeur `InstanceCount` de -1. Cette valeur signifie qu’une instance doit être en cours d’exécution sur chaque nœud du cluster. Si certaines de ces instances échouent, Service Fabric détecte que ce service n’est pas à l’état souhaité et essaie de créer les instances sur les nœuds où elles sont manquantes.

#### <a name="stateful-services"></a>Services avec état
Il existe deux types de service avec état :
- Service avec état persistant.
- Service avec état non persistant. (Variables stockées en mémoire.)

La récupération après une défaillance d’un service avec état dépend du type du service avec état, du nombre de réplicas détenus par le service et du nombre de réplicas ayant échoué.

Dans un service avec état, les données entrantes sont répliquées entre les réplicas (primaire et secondaires actifs). Si une majorité des réplicas reçoivent les données, celles-ci sont considérées comme étant validées par le *quorum*. (Pour cinq réplicas, trois sont un quorum.) Cela signifie qu’à tout moment, il y aura au moins un quorum de réplicas avec les données les plus récentes. Si des réplicas échouent (par exemple, deux sur cinq), nous pouvons utiliser la valeur du quorum pour calculer si nous pouvons les récupérer. (Étant donné que les trois réplicas restants sur cinq sont toujours opérationnels, il est garanti qu’au moins un réplica aura des données complètes.)

En cas de défaillance d’un quorum de réplicas, la partition est déclarée comme étant en état de *perte de quorum*. Supposons qu’une partition a cinq réplicas, ce qui signifie qu’au moins trois sont assurés d’avoir des données complètes. En cas d’échec d’un quorum (trois sur cinq) des réplicas), Service Fabric ne peut pas déterminer si les réplicas restants (deux sur cinq) disposent de suffisamment de données pour restaurer la partition. Dans les cas où Service Fabric détecte une perte de quorum, son comportement par défaut consiste à empêcher les écritures supplémentaires sur la partition, à déclarer une perte de quorum et à attendre la restauration d’un quorum de réplicas.

Pour déterminer si un incident s’est produit pour un service avec état et pour gérer ensuite cet incident, effectuez les trois étapes suivantes :

1. Déterminez s’il y a eu perte de quorum, ou non.
   
   Une perte de quorum est déclarée quand la majorité des réplicas d’un service avec état sont défaillants en même temps.
2. Déterminez si la perte de quorum est permanente ou non.
   
   La plupart du temps, ces défaillances sont passagères. Les processus sont redémarrés, les nœuds sont redémarrés, les machines virtuelles sont relancées et les partitions réseau sont réparées. Cependant, il arrive que les défaillances soient permanentes. Les échecs sont permanents ou non, selon que l’état du service avec état est conservé ou qu’il l’est uniquement en mémoire : 
   
   - Pour les services non persistants, la défaillance d’un quorum ou de plusieurs réplicas aboutit _immédiatement_ à une perte de quorum permanente. Quand Service Fabric détecte une perte de quorum dans un service avec état non persistant, il passe immédiatement à l’étape 3 en déclarant une perte (potentielle) de données. La déclaration de perte de données est logique. En effet, Service Fabric sait qu’il est inutile d’attendre le retour des réplicas. Même s’ils sont récupérés, les données sont perdues en raison de la nature non persistante du service.
   - Pour les services persistants avec état, quand une défaillance de quorum ou de plusieurs réplicas se produit, Service Fabric se met à attendre le retour des réplicas et la restauration du quorum. Cela entraîne une interruption de service pour les _opérations d’écriture_ dans la partition (ou « jeu de réplicas ») concernée. Cependant, les opérations de lecture sont toujours possibles, avec toutefois une moins bonne garantie de cohérence. Par défaut, Service Fabric attend *indéfiniment* la restauration du quorum, car le fait de continuer sans attendre la restauration peut entraîner une perte de données et présente d’autres risques. Service Fabric ne passe donc pas à l’étape suivante tant qu’un administrateur ne prend aucune mesure pour déclarer une perte de données.
3. Déterminer si des données sont perdues et restaurer à partir de sauvegardes.

   Si la perte de quorum a été déclarée (automatiquement ou par le biais d’une action d’administration), Service Fabric et les services déterminent alors si des données ont été réellement perdues. À ce stade, Service Fabric sait également que les autres réplicas ne pourront pas être restaurés. C’est la décision que nous avons prise lorsque nous avons arrêté d’attendre la résolution de la perte de quorum. Le meilleur plan d’action pour le service est généralement de tout stopper et d’attendre une intervention de l’administrateur.
   
   Lorsque Service Fabric appelle la méthode `OnDataLossAsync`, c’est toujours dans le cas d’une perte de données _supposée_. Service Fabric vérifie que cet appel est remis au _meilleur_ réplica restant. Il s’agira du réplica qui a le plus progressé. 
   
   Nous parlons toujours de perte de données _supposée_, car il est possible que le réplica restant ait exactement le même état que celui du réplica principal au moment de la perte du quorum. Toutefois, sans cet état auquel le comparer, il n’existe aucun moyen pour Service Fabric ou les opérateurs de s’en assurer.     
   
   Qu’est-ce que fait une implémentation classique de la méthode `OnDataLossAsync` ?
   1. L’implémentation enregistre que `OnDataLossAsync` a été déclenché, puis elle lance les alertes administratives nécessaires.
   1. En général, l’implémentation s’interrompt et attend que l’administrateur ait pris une décision et entreprenne les actions manuelles nécessaires. En effet, même si des sauvegardes sont disponibles, elles doivent d’abord être préparées. 
   
      Par exemple, si deux services coordonnent des informations, ces sauvegardes devront peut-être être modifiées afin de garantir qu’après la restauration, les informations qui importent le plus pour ces deux services sont cohérentes. 
   1. Souvent, le service implique d’autres données de télémétrie. Ces métadonnées peuvent être contenues dans d’autres services ou dans des journaux d’activité. Si nécessaire, ces informations peuvent être utilisées pour déterminer si des appels reçus et traités sur le réplica principal ne sont pas présents dans la sauvegarde ou n’ont pas été répliqués sur un réplica donné. Ces appels devront peut-être être relus ou ajoutés à la sauvegarde pour permettre la restauration.  
   1. L’implémentation compare l’état du réplica restant à celui contenu dans les sauvegardes disponibles. Si vous utilisez des collections fiables Service Fabric, vous disposez [d’outils et de processus](service-fabric-reliable-services-backup-restore.md) pour le faire. L’objectif est de voir si l’état au sein du réplica est suffisant et quels éléments manquent à la sauvegarde.
   1. Une fois la comparaison effectuée et la restauration terminée (si nécessaire), le code du service doit retourner la valeur **true** si des modifications de l’état ont été apportées. Si le réplica détermine qu’il s’agit de la meilleure copie disponible de l’état et n’apporte aucune modification, le code retourne la valeur **false**. 
   
      La valeur **true** indique que les _autres_ réplicas restants peuvent maintenant ne plus être cohérents avec celui-ci. Ils sont donc supprimés et reconstruits à partir de ce réplica. Une valeur **false** indique qu’aucune modification d’état n’a été apportée, et donc, que les autres réplicas peuvent être conservés tels quels. 

Il est extrêmement important que les auteurs du service testent les scénarios de défaillance et de perte de données potentielle avant de déployer les services dans l’environnement de production. Pour vous protéger contre le risque de perte, il est important de régulièrement [sauvegarder l’état](service-fabric-reliable-services-backup-restore.md) de vos services avec état dans un magasin géoredondant. 

Vous devez également être sûr que vous avez la possibilité de restaurer l’état. Étant donné que des sauvegardes de nombreux services différents sont effectuées à des moments différents, vous devez être sûr qu’après la restauration, vos services ont une vision cohérente des uns des autres. 

Par exemple, prenons le cas d’un service qui génère un numéro et le stocke, puis l’envoie à un autre service qui le stocke également. Après une restauration, vous pouvez vous rendre compte que le deuxième service contient le numéro, mais pas le premier service, parce que sa sauvegarde n’a pas inclus cette opération.

Si vous découvrez que les réplicas restants ne suffisent pas pour continuer dans un scénario de perte, et si vous ne pouvez pas reconstruire l’état du service à partir des données de télémétrie, c’est la fréquence de vos sauvegardes qui va déterminer votre meilleur objectif de point de récupération (RPO). Service Fabric fournit de nombreux outils pour tester divers scénarios d’échec, y compris la perte de quorum permanente et la perte de données qui nécessitent une restauration à partir d’une sauvegarde. Ces scénarios sont inclus dans les outils de testabilité de Service Fabric, qui sont gérés par le service d’analyse des erreurs. Pour plus d’informations sur ces outils et modèles, consultez [Présentation du service d’analyse des erreurs](service-fabric-testability-overview.md). 

> [!NOTE]
> Les services système peuvent également subir une perte de quorum. L’impact est spécifique au service en question. Par exemple, une perte de quorum dans le service d’affectation de noms affecte la résolution de noms, tandis qu’une perte de quorum dans le service Failover Manager bloque la création de services et les basculements. 
> 
> Les services système de Service Fabric suivent le même modèle que vos services de gestion des états, mais nous vous déconseillons d’essayer de les faire passer de l’état de perte de quorum à celui de perte de données potentielle. Au lieu de cela, nous vous recommandons d’[obtenir de l’aide](service-fabric-support.md) pour trouver une solution ciblée adaptée à votre situation. Il est généralement préférable d’attendre le retour des réplicas défaillants.
>

#### <a name="troubleshooting-quorum-loss"></a>Résolution des problèmes liés à la perte de quorum

Les réplicas peuvent être interrompus par intermittence en raison d’une défaillance passagère. Patientez un certain temps, car Service Fabric essaie de les récupérer. Si les réplicas ont été inactifs pendant une durée supérieure à la durée attendue, prenez ces mesures de résolution des problèmes :
- Les réplicas sont peut-être bloqués. Consultez les rapports d’intégrité au niveau des réplicas et vos journaux des applications. Collectez les vidages sur incident et prenez les mesures nécessaires pour effectuer la récupération.
- Le processus de réplica ne répond peut-être plus. Inspectez vos journaux des applications pour vérifier ce point. Collectez le vidage des processus, puis stoppez le processus qui ne répond pas. Service Fabric crée un processus de remplacement et essaie de récupérer le réplica.
- Les nœuds qui hébergent les réplicas sont peut-être inactifs. Redémarrez la machine virtuelle sous-jacente pour récupérer les nœuds.

Il n’est parfois pas possible de récupérer des réplicas. Ce sera le cas, par exemple, si les lecteurs sont défaillants ou si les ordinateurs ne répondent pas. Dans ce cas, Service Fabric doit être invité à ne pas attendre la récupération du réplica.

N’utilisez *pas* ces méthodes si la perte de données potentielle est incompatible avec la mise en ligne du service. Dans ce cas, tous les efforts doivent être faits pour récupérer les machines physiques.

Les actions suivantes peuvent entraîner une perte de données. Vérifiez avant de les mettre en œuvre.
   
> [!NOTE]
> L’utilisation de ces méthodes autrement que pour cibler certaines partitions n’est _jamais_ sûre. 
>

- Utilisez `Repair-ServiceFabricPartition -PartitionId` ou `System.Fabric.FabricClient.ClusterManagementClient.RecoverPartitionAsync(Guid partitionId)` l’API. Cette API permet de spécifier l’ID de la partition à faire passer de l’état de perte de quorum à celui de perte de données potentielle.
- Si votre cluster rencontre des défaillances fréquentes qui font passer les services à l’état de perte de quorum et qu’_une perte de données potentielle est acceptable_, la spécification d’une valeur [QuorumLossWaitDuration](/powershell/module/servicefabric/update-servicefabricservice?view=azureservicefabricps) appropriée peut aider vos services à récupérer automatiquement. Service Fabric attend pendant la durée `QuorumLossWaitDuration` indiquée (par défaut, durée infinie) avant d’effectuer la récupération. Nous ne recommandons *pas* cette méthode, car elle peut entraîner des pertes de données inattendues.

## <a name="availability-of-the-service-fabric-cluster"></a>Disponibilité du cluster Service Fabric
En général, le cluster Service Fabric est un environnement hautement distribué sans point de défaillance unique. Une défaillance au niveau d’un nœud n’entraîne pas de problèmes de disponibilité ou de fiabilité pour le cluster. Ceci est principalement dû au fait que les services système de Service Fabric suivent les instructions données plus haut. Autrement dit, ils s’exécutent toujours avec au moins trois réplicas par défaut, et les services système exécutés sans état s’exécutent sur tous les nœuds. 

Les couches sous-jacentes Service Fabric de détection du réseau et des défaillances sont entièrement distribuées. La plupart des services système peuvent être reconstruits à partir des métadonnées dans le cluster, ou sinon, savent comment resynchroniser leur état dans d’autres emplacements. La disponibilité du cluster peut être compromise si les services système connaissent une perte de quorum comme celle décrite précédemment. Dans ce cas, vous n’êtes peut-être pas en mesure d’effectuer certaines opérations sur le cluster, comme démarrer une mise à niveau ou déployer de nouveaux services. Toutefois, le cluster reste disponible. 

Les services sur un cluster en cours d’exécution continueront à s’exécuter dans ces conditions, sauf s’ils ont besoin que les opérations d’écriture continuent de fonctionner dans les services système. Par exemple, si Failover Manager est en perte de quorum, tous les services continuent à s’exécuter. Cependant, tous les services qui échouent ne peuvent pas redémarrer automatiquement, car cela nécessite l’implication de Failover Manager. 

### <a name="failures-of-a-datacenter-or-an-azure-region"></a>Défaillance d’un centre de données ou d’une région Azure
Dans de rares cas, un centre de données physique peut devenir temporairement indisponible suite à une perte d’alimentation ou de connectivité réseau. Dans ce cas, les clusters et services Service Fabric du centre de données ou de la région Azure ne sont plus disponibles. Toutefois, _vos données sont conservées_. 

Pour les clusters exécutés dans Azure, vous pouvez afficher les mises à jour concernant les pannes sur la [page d’état Azure][azure-status-dashboard]. En cas de destruction partielle ou intégrale d’un centre de données physique (ce qui est très peu probable), les clusters Service Fabric qui y sont hébergés et les services qu’ils contiennent peuvent être perdus. Cette perte comprend les états non sauvegardés en dehors du centre de données ou de la région.

Deux stratégies permettent de surmonter la défaillance permanente ou prolongée d’un centre de données ou d’une région : 

- Exécuter des clusters Service Fabric distincts dans plusieurs régions de ce type et utiliser un mécanisme de basculement et de restauration automatiques entre ces environnements. Ce type de modèle multicluster actif/actif ou actif/passif nécessite du code de gestion et d’opération supplémentaire. Ce modèle nécessite également la coordination des sauvegardes des services d’un centre de données ou d’une région afin qu’ils soient disponibles dans d’autres centres de données ou régions quand l’un d’eux échoue. 
- Exécuter un cluster Service Fabric qui s’étend sur plusieurs centres de données ou régions. Pour cette stratégie, la configuration minimale prise en charge est de trois régions ou centres de données. Il est recommandé d’utiliser cinq régions ou centres de données. 
  
  Ce modèle nécessite une topologie de cluster plus complexe. Toutefois, l’avantage est que la défaillance d’un centre de données ou d’une région passe de l’état d’incident à celui de défaillance normale. Ces défaillances peuvent être gérées par les mécanismes qui fonctionnent pour les clusters d’une même région. Les domaines d’erreur, les domaines de mise à niveau et les règles de sélection élective de Service Fabric garantissent que les charges de travail sont distribuées de manière à survivre aux défaillances normales. 
  
  Pour plus d’informations sur les stratégies qui peuvent aider à utiliser les services dans ce type de cluster, consultez [Stratégies de sélection élective pour les services Service Fabric](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md).

### <a name="random-failures-that-lead-to-cluster-failures"></a>Défaillances aléatoires qui entraînent des défaillances de cluster
Service Fabric utilise le concept de *nœuds initiaux*. Il s’agit de nœuds qui permettent de maintenir la disponibilité du cluster sous-jacent. 

Ils vous aident à garantir que le cluster reste opérationnel en établissant des baux avec d’autres nœuds et en servant de tiebreaker pour certains types de défaillances. Si des défaillances aléatoires suppriment la majorité des nœuds initiaux du cluster et que ceux-ci ne sont pas restaurés rapidement, votre cluster est automatiquement arrêté. Le cluster échoue alors. 

Dans Azure, le fournisseur de ressources Service Fabric gère les configurations de cluster Service Fabric. Par défaut, le fournisseur de ressources distribue les nœuds initiaux dans les domaines d’erreur et de mise à niveau pour le *type de nœud principal*. Si le type de nœud principal a une durabilité Silver ou Gold, lorsque vous supprimez un nœud initial (en le mettant à l’échelle dans votre type de nœud principal ou en le supprimant manuellement), le cluster tente de promouvoir un autre nœud non initial à partir de la capacité disponible du type de nœud principal. Cette tentative échoue si la capacité disponible est inférieure à celle requise par le niveau de fiabilité de votre cluster pour votre type de nœud principal.

Dans les clusters Service Fabric autonomes et Azure, le type de nœud principal est celui qui exécute les nœuds initiaux. Lorsque vous définissez un type de nœud principal, Service Fabric utilise automatiquement le nombre de nœuds fournis en créant jusqu’à neuf nœuds initiaux et sept réplicas de chaque service système. Si un ensemble de défaillances aléatoires supprime simultanément une majorité de ces réplicas, les services système passent à l’état de perte de quorum. Si la majorité des nœuds initiaux est perdue, le cluster est arrêté rapidement après.

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment simuler différentes défaillances à l’aide de [l’infrastructure de testabilité](service-fabric-testability-overview.md).
- Consultez les autres ressources de récupération d’urgence et de haute disponibilité. Microsoft a publié de nombreuses rubriques d’aide sur ces sujets. Bien que certains de ces documents fassent référence à des techniques propres à d’autres produits, ils contiennent un grand nombre de meilleures pratiques générales que vous pouvez appliquer dans le contexte de Service Fabric :
  - [Liste de contrôle de disponibilité](/azure/architecture/checklist/resiliency-per-service)
  - [Exécution d’un exercice de récupération d’urgence](../azure-sql/database/disaster-recovery-drills.md)
  - [Récupération d’urgence et haute disponibilité pour les applications Azure][dr-ha-guide]
- Découvrez les [options de support de Service Fabric](service-fabric-support.md).


<!-- External links -->

[repair-partition-ps]: /windows/win32/perfctrs/specifying-a-counter-path
[azure-status-dashboard]:https://azure.microsoft.com/status/
[azure-regions]: https://azure.microsoft.com/regions/
[dr-ha-guide]: /previous-versions/azure/dn251004(v=azure.100)


<!-- Images -->

[sfx-cluster-map]: ./media/service-fabric-disaster-recovery/sfx-clustermap.png
