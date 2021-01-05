---
title: Résoudre les problèmes avec les rapports d’intégrité du système
description: Décrit les rapports d’intégrité envoyés par les composants Azure Service Fabric et leur utilisation pour la résolution des problèmes de cluster ou d’application.
author: georgewallace
ms.topic: conceptual
ms.date: 2/28/2018
ms.author: gwallace
ms.openlocfilehash: 8e60ac5065c2f9543a641daf4f62299c00c61fc8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000655"
---
# <a name="use-system-health-reports-to-troubleshoot"></a>Utiliser les rapports d’intégrité du système pour la résolution des problèmes
Les composants Azure Service Fabric fournissent des rapports d’intégrité du système prêts à l’emploi pour toutes les entités du cluster. Le [magasin d’intégrité](service-fabric-health-introduction.md#health-store) crée et supprime des entités en fonction des rapports du système. Il les organise au sein d’une hiérarchie qui tient compte des interactions entre les entités.

> [!NOTE]
> Pour mieux comprendre les concepts relatifs à l’intégrité, consultez la documentation relative au [modèle d’intégrité du Service Fabric](service-fabric-health-introduction.md).
> 
> 

Les rapports d’intégrité du système procurent une visibilité sur les fonctionnalités du cluster et des applications, et signalent les problèmes. Pour les applications et services, les rapports d’intégrité du système vérifient que les entités sont implémentées et qu’elles se comportent correctement du point de vue de Service Fabric. Les rapports ne fournissent aucune information sur l’intégrité de la logique métier du service ni sur la détection des processus qui ne répondent pas. Les services utilisateur peuvent enrichir les données d’intégrité avec des informations spécifiques à leur logique.

> [!NOTE]
> Les rapports d’intégrité envoyés par les agents de surveillance de l’utilisateur sont visibles uniquement *après* que les composants système ont créé une entité. Lorsqu’une entité est supprimée, le magasin d’intégrité élimine automatiquement l’ensemble des rapports d’intégrité qui lui sont associés. Il en est de même lorsqu’une instance de cette entité est créée. Par exemple, lorsqu’une instance de réplica de service persistant et avec état est créée. Tous les rapports associés à l’ancienne instance sont supprimés et éliminés du magasin.
> 
> 

Les rapports sur les composants système sont identifiés par la source, qui commence par le préfixe « **System.** » . Les pilotes de surveillance ne peuvent pas utiliser le même préfixe pour leurs sources, car les rapports dotés de paramètres non valides sont rejetés.

Examinons certains rapports du système et tâchons d’identifier les éléments déclencheurs et d’apprendre à corriger les problèmes potentiels qu’ils représentent.

> [!NOTE]
> Service Fabric continue d’ajouter des rapports sur les domaines d’intérêt qui améliorent la visibilité sur les évènements associés au cluster et aux applications. Les rapports existants peuvent être améliorés en offrant plus de détails pour vous aider à résoudre le problème plus rapidement.
> 
> 

## <a name="cluster-system-health-reports"></a>Rapports d’intégrité du système sur le cluster
L’entité d’intégrité du cluster est créée automatiquement dans le magasin d’intégrité. Si tout fonctionne correctement, elle ne présente pas de rapport système.

### <a name="neighborhood-loss"></a>Perte de voisinage
**System.Federation** indique une erreur s’il détecte une perte de voisinage. Le rapport est issu de nœuds individuels ; l’ID de nœud est inclus dans le nom de propriété. Si un voisinage est perdu dans l’ensemble de l’anneau de Service Fabric, deux événements (qui représentent les deux limites du rapport d’intervalle) sont généralement signalés. Si plusieurs voisinages sont perdus, d’autres d’événements ont lieu.

Le rapport spécifie le délai d’expiration du bail global comme durée de vie. Il est renvoyé lorsque la moitié de la durée de vie est atteinte, tant que la condition reste active. L’événement arrivé à expiration est automatiquement supprimé. Le comportement de suppression à expiration garantit le nettoyage approprié du rapport dans le magasin d’intégrité, même si le nœud de création de rapports est arrêté.

* **SourceId** : System.Federation
* **Property** : commence par **Neighborhood** et inclut des informations sur le nœud.
* **Étapes suivantes** : examinez la raison de la perte du voisinage. Par exemple, vérifiez la communication entre les nœuds des clusters.

### <a name="rebuild"></a>Reconstruire

Le service Failover Manager (FM) gère les informations sur les nœuds des clusters. Lorsque FM perd ses données et passe en perte de données, il ne peut pas être sûr qu’il dispose des informations les plus récentes concernant les nœuds du cluster. Dans ce cas, le système passe par une regénération, et System.FM recueille les données de tous les nœuds du cluster afin de rétablir son état. Parfois, en raison de problèmes liés au réseau ou aux nœuds, il peut arriver que la regénération soit bloquée. Cela peut également se produire avec le service Failover Manager Master (FMM). Le service FMM est un service système sans état qui assure le suivi de toutes les instances du service FM qui se trouvent dans le cluster. Le nœud principal du service FMM est toujours le nœud dont l’ID est le plus proche de 0. Si ce nœud est supprimé, une regénération est déclenchée.
Lorsque l’une des conditions précédentes se produit, **System.FM** ou **System.FMM** la signalent via un rapport d’erreurs. La regénération peut se bloquer lors de l’une des deux phases suivantes :

* **Attente de la diffusion** : FM/FMM attend la réponse des autres nœuds au message de diffusion.

  * **Étapes suivantes** : vérifiez qu’il n’y a pas de problèmes de connexion réseau entre les nœuds.
* **Attente des nœuds** : FM/FMM a déjà reçu une réponse de diffusion des autres nœuds et attend la réponse de certains nœuds. Le rapport d’intégrité répertorie les nœuds pour lesquels FM/FMM attend une réponse.
   * **Étapes suivantes** : vérifiez la connexion réseau entre FM/FMM et les nœuds répertoriés. Examinez chacun des nœuds répertoriés pour vérifier qu’ils ne comportent pas d’autres problèmes.

* **SourceID** : System.FM ou System.FMM
* **Property** : Rebuild (Regénération).
* **Étapes suivantes** : vérifiez la connexion réseau entre les nœuds, ainsi que l’état de tous les nœuds qui sont répertoriés dans la description du rapport d’intégrité.

### <a name="seed-node-status"></a>État du nœud seed
**System.FM** émet un avertissement de niveau de cluster si certains nœuds seed sont défectueux. Les nœuds seed permettent de maintenir la disponibilité du cluster sous-jacent. Ces nœuds vous aident à garantir que le cluster reste actif en établissant des baux avec d’autres nœuds et en servant de tiebreaker pour certains types de défaillances réseau. Si la majorité des nœuds seed du cluster sont supprimés et ne sont pas restaurés, le cluster est automatiquement arrêté. 

Un nœud seed est non sain si son état de nœud est Arrêté, Supprimé ou Inconnu.
Le rapport d’avertissement relatif à l’état du nœud seed répertorie tous les nœuds seed non sains et fournit des informations détaillées les concernant.

* **SourceID** : System.FM
* **Property** : SeedNodeStatus
* **Étapes suivantes** : si cet avertissement s’affiche dans le cluster, suivez les instructions ci-dessous pour résoudre le problème : Pour un cluster exécutant Service Fabric 6.5 ou une version ultérieure : Pour un cluster Service Fabric sur Azure, Service Fabric tente automatiquement de transformer un nœud seed qui vient de s’arrêter en un nœud non seed. Pour que cela se produise, assurez-vous que le nombre de nœuds non seed dans le type de nœud principal est supérieur ou égal au nombre de nœuds seed avec l’état Arrêté. Le cas échéant, ajoutez des nœuds supplémentaires au type de nœud principal.
En fonction de l’état du cluster, cela peut prendre un certain temps pour résoudre le problème. Après cela, le rapport d’avertissement est automatiquement effacé.

Pour un cluster Service Fabric autonome, tous les nœuds seed doivent être sains pour effacer le rapport d’avertissement. Selon la raison pour laquelle les nœuds seed sont non sains, différentes actions sont possibles : si le nœud seed est arrêté, les utilisateurs doivent le restaurer ; si le nœud seed est supprimé ou inconnu, il [doit être supprimé du cluster](./service-fabric-cluster-windows-server-add-remove-nodes.md).
Le rapport d’avertissement est automatiquement effacé lorsque tous les nœuds seed sont sains.

Pour un cluster exécutant une version antérieure à Service Fabric 6.5 : Dans ce cas, le rapport d’avertissement doit être effacé manuellement. **Les utilisateurs doivent s’assurer que tous les nœuds seed sont sains avant d’effacer le rapport** : si le nœud seed est arrêté, les utilisateurs doivent le restaurer ; si le nœud seed est supprimé ou inconnu, il doit être supprimé du cluster.
Une fois que tous les nœuds seed sont sains, utilisez la commande suivante dans PowerShell pour [effacer le rapport d’avertissement](/powershell/module/servicefabric/send-servicefabricclusterhealthreport) :

```powershell
PS C:\> Send-ServiceFabricClusterHealthReport -SourceId "System.FM" -HealthProperty "SeedNodeStatus" -HealthState OK

## Node system health reports
System.FM, which represents the Failover Manager service, is the authority that manages information about cluster nodes. Each node should have one report from System.FM showing its state. The node entities are removed when the node state is removed. For more information, see [RemoveNodeStateAsync](/dotnet/api/system.fabric.fabricclient.clustermanagementclient.removenodestateasync).

### Node up/down
System.FM reports as OK when the node joins the ring (it's up and running). It reports an error when the node departs the ring (it's down, either for upgrading or simply because it has failed). The health hierarchy built by the health store acts on deployed entities in correlation with System.FM node reports. It considers the node a virtual parent of all deployed entities. The deployed entities on that node are exposed through queries if the node is reported as up by System.FM, with the same instance as the instance associated with the entities. When System.FM reports that the node is down or restarted, as a new instance, the health store automatically cleans up the deployed entities that can exist only on the down node or on the previous instance of the node.

* **SourceId**: System.FM
* **Property**: State.
* **Next steps**: If the node is down for an upgrade, it should come back up after it's been upgraded. In this case, the health state should switch back to OK. If the node doesn't come back or it fails, the problem needs more investigation.

The following example shows the System.FM event with a health state of OK for node up:

```powershell
PS C:\> Get-ServiceFabricNodeHealth  _Node_0

NodeName              : _Node_0
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 8
                        SentAt                : 7/14/2017 4:54:51 PM
                        ReceivedAt            : 7/14/2017 4:55:14 PM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
```


### <a name="certificate-expiration"></a>Expiration du certificat
**System.FabricNode** indique un avertissement lorsque les certificats utilisés par le nœud sont sur le point d’arriver à expiration. Chaque nœud comporte trois certificats : **Certificate_cluster**, **Certificate_server** et **Certificate_default_client**. Lorsque la date d’expiration est à au moins deux semaines, l’état d’intégrité du rapport est OK. Si elle a lieu dans les deux semaines qui suivent, le type de rapport est un avertissement. La durée de vie de ces événements est infinie, et ils sont supprimés lorsqu’un nœud quitte un cluster.

* **SourceId** : System.FabricNode
* **Property** : commence par **Certificate** et contient des informations supplémentaires sur le type de certificat.
* **Étapes suivantes** : mettez à jour les certificats sur le point d’arriver à expiration.

### <a name="load-capacity-violation"></a>Violation de capacité de charge
L’équilibrage de charge de Service Fabric indique un avertissement quand il détecte une violation de la capacité du nœud.

* **SourceId** : System.PLB
* **Property** : commence par **Capacity**.
* **Étapes suivantes** : contrôlez les mesures fournies et examinez la capacité actuelle sur le nœud.

### <a name="node-capacity-mismatch-for-resource-governance-metrics"></a>Incompatibilité des capacités de nœud pour les mesures de gouvernance des ressources
System.Hosting transmet un avertissement si les capacités de nœud définies dans le manifeste de cluster sont supérieures aux capacités de nœud réelles pour les mesures de gouvernance des ressources (cœurs du processeur et mémoire). Un rapport d’intégrité s’affiche lorsque le premier package de service qui utilise la [gouvernance des ressources](service-fabric-resource-governance.md) s’inscrit sur un nœud spécifié.

* **SourceId** : System.Hosting
* **Property** : **ResourceGovernance**.
* **Étapes suivantes** : cela peut poser un problème, car les packages de service régissant ne sont pas appliqués comme prévu et la [gouvernance des ressources](service-fabric-resource-governance.md) ne fonctionne pas correctement. Mettez à jour le manifeste de cluster en indiquant les capacités de nœud appropriées pour ces mesures ou ne les spécifiez pas et laissez Service Fabric détecter automatiquement les ressources disponibles.

## <a name="application-system-health-reports"></a>Rapports d’intégrité du système sur les applications
System.CM, qui représente le service Cluster Manager, est l’autorité qui gère les informations sur une application.

### <a name="state"></a>State
System.CM consigne la valeur OK lorsque l’application a été créée ou mise à jour. Il informe le magasin d’intégrité lorsque l’application est supprimée afin qu’elle puisse en être retirée.

* **SourceId** : System.CM
* **Property** : State (État).
* **Étapes suivantes** : si l’application a été créée ou mise à jour, elle doit inclure le rapport d’intégrité du gestionnaire du cluster. Sinon, vérifiez l’état de l’application en effectuant une requête. Par exemple, utilisez la cmdlet PowerShell **Get-ServiceFabricApplication -ApplicationName** *applicationName*.

L’exemple suivant représente l’événement d’état sur l’application **fabric:/WordCount** :

```powershell
PS C:\> Get-ServiceFabricApplicationHealth fabric:/WordCount -ServicesFilter None -DeployedApplicationsFilter None -ExcludeHealthStatistics

ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Ok
ServiceHealthStates             : None
DeployedApplicationHealthStates : None
HealthEvents                    : 
                                  SourceId              : System.CM
                                  Property              : State
                                  HealthState           : Ok
                                  SequenceNumber        : 282
                                  SentAt                : 7/13/2017 5:57:05 PM
                                  ReceivedAt            : 7/14/2017 4:55:10 PM
                                  TTL                   : Infinite
                                  Description           : Application has been created.
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : Error->Ok = 7/13/2017 5:57:05 PM, LastWarning = 1/1/0001 12:00:00 AM
```

## <a name="service-system-health-reports"></a>Rapports d’intégrité du système sur les services
System.FM, qui représente le service Failover Manager, est l’autorité qui gère les informations sur les services.

### <a name="state"></a>State
System.FM consigne la valeur OK lorsque le service a été créé. Il supprime l’entité du magasin d’intégrité lorsque le service est supprimé.

* **SourceId** : System.FM
* **Property** : State (État).

L’exemple suivant représente l’événement d’état sur le service **fabric:/WordCount/WordCountWebService** :

```powershell
PS C:\> Get-ServiceFabricServiceHealth fabric:/WordCount/WordCountWebService -ExcludeHealthStatistics


ServiceName           : fabric:/WordCount/WordCountWebService
AggregatedHealthState : Ok
PartitionHealthStates : 
                        PartitionId           : 8bbcd03a-3a53-47ec-a5f1-9b77f73c53b2
                        AggregatedHealthState : Ok
                        
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 14
                        SentAt                : 7/13/2017 5:57:05 PM
                        ReceivedAt            : 7/14/2017 4:55:10 PM
                        TTL                   : Infinite
                        Description           : Service has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="service-correlation-error"></a>Erreur de corrélation de services
**System.PLB** signale une erreur lorsqu’il détecte que la mise à jour d’un service est liée à un autre service qui crée une chaîne d’affinités. Le rapport est effacé lorsque la mise à jour est réussie.

* **SourceId** : System.PLB
* **Property** : **ServiceDescription**.
* **Étapes suivantes** : vérifiez les descriptions de service en corrélation.

## <a name="partition-system-health-reports"></a>Rapports d’intégrité du système sur les partitions
System.FM, qui représente le service Failover Manager, est l’autorité qui gère les informations sur les partitions de service.

### <a name="state"></a>State
System.FM consigne la valeur OK lorsque la partition créée est intègre. Il élimine l’entité du magasin d’intégrité lorsque la partition est supprimée.

Si la partition présente une valeur inférieure au nombre minimal de réplicas, une erreur est signalée. Si la partition présente une valeur supérieure au nombre minimum de réplicas, mais inférieure au nombre cible de réplicas, un avertissement est signalé. Si la partition subit une perte de quorum, System.FM indique une erreur.

Les autres événements notables incluent un avertissement quand la reconfiguration et la génération prennent plus de temps que prévu. Les délais impartis pour la génération et la reconfiguration sont configurables en fonction des scénarios de service. Par exemple, si un service présente un état défini en téraoctet, par exemple Azure SQL Database, la génération prendra davantage de temps que celle d’un service affichant un état d’un volume moindre.

* **SourceId** : System.FM
* **Property** : State (État).
* **Étapes suivantes** : si l’état d’intégrité n’affiche pas OK, il est possible que certains réplicas n’aient pas été correctement créés, ouverts ou promus comme réplicas principaux ou secondaires. 

Si la description fait état d’une perte de quorum, l’examen du rapport d’intégrité détaillé pour les réplicas qui sont arrêtés et la remise en service de ceux-ci peuvent vous permettre de remettre en ligne la partition.

Si la description indique une partition bloquée lors de la [reconfiguration](service-fabric-concepts-reconfiguration.md), le rapport d’intégrité sur le réplica principal fournit des informations supplémentaires.

Pour les autres rapports d’intégrité System.FM, il y aurait des rapports sur les réplicas ou la partition ou le service à partir d’autres composants système. 

Les exemples ci-dessous décrivent certains de ces rapports. 

L’exemple suivant représente une partition saine :

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountWebService | Get-ServiceFabricPartitionHealth -ExcludeHealthStatistics -ReplicasFilter None

PartitionId           : 8bbcd03a-3a53-47ec-a5f1-9b77f73c53b2
AggregatedHealthState : Ok
ReplicaHealthStates   : None
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 70
                        SentAt                : 7/13/2017 5:57:05 PM
                        ReceivedAt            : 7/14/2017 4:55:10 PM
                        TTL                   : Infinite
                        Description           : Partition is healthy.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

L’exemple suivant représente l’intégrité d’une partition qui présente un nombre de réplicas inférieur à la valeur cible. L’étape suivante consiste à obtenir la description de partition indiquant comment elle est configurée : La valeur de **MinReplicaSetSize** est de trois et celle de **TargetReplicaSetSize** est de sept. Récupérez ensuite le nombre de nœuds dans le cluster. Ici, il est égal à cinq. Par conséquent, dans ce cas, il n’est pas possible de placer deux réplicas, car le nombre cible de réplicas est supérieur au nombre de nœuds disponibles.

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricPartitionHealth -ReplicasFilter None -ExcludeHealthStatistics


PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.
                        
ReplicaHealthStates   : None
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 123
                        SentAt                : 7/14/2017 4:55:39 PM
                        ReceivedAt            : 7/14/2017 4:55:44 PM
                        TTL                   : Infinite
                        Description           : Partition is below target replica or instance count.
                        fabric:/WordCount/WordCountService 7 2 af2e3e44-a8f8-45ac-9f31-4093eb897600
                          N/S Ready _Node_2 131444422260002646
                          N/S Ready _Node_4 131444422293113678
                          N/S Ready _Node_3 131444422293113679
                          N/S Ready _Node_1 131444422293118720
                          N/P Ready _Node_0 131444422293118721
                          (Showing 5 out of 5 replicas. Total available replicas: 5)
                        
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 7/14/2017 4:55:44 PM, LastOk = 1/1/0001 12:00:00 AM
                        
                        SourceId              : System.PLB
                        Property              : ServiceReplicaUnplacedHealth_Secondary_af2e3e44-a8f8-45ac-9f31-4093eb897600
                        HealthState           : Warning
                        SequenceNumber        : 131445250939703027
                        SentAt                : 7/14/2017 4:58:13 PM
                        ReceivedAt            : 7/14/2017 4:58:14 PM
                        TTL                   : 00:01:05
                        Description           : The Load Balancer was unable to find a placement for one or more of the Service's Replicas:
                        Secondary replica could not be placed due to the following constraints and properties:  
                        TargetReplicaSetSize: 7
                        Placement Constraint: N/A
                        Parent Service: N/A
                        
                        Constraint Elimination Sequence:
                        Existing Secondary Replicas eliminated 4 possible node(s) for placement -- 1/5 node(s) remain.
                        Existing Primary Replica eliminated 1 possible node(s) for placement -- 0/5 node(s) remain.
                        
                        Nodes Eliminated By Constraints:
                        
                        Existing Secondary Replicas -- Nodes with Partition's Existing Secondary Replicas/Instances:
                        --
                        FaultDomain:fd:/4 NodeName:_Node_4 NodeType:NodeType4 UpgradeDomain:4 UpgradeDomain: ud:/4 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/3 NodeName:_Node_3 NodeType:NodeType3 UpgradeDomain:3 UpgradeDomain: ud:/3 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/2 NodeName:_Node_2 NodeType:NodeType2 UpgradeDomain:2 UpgradeDomain: ud:/2 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/1 NodeName:_Node_1 NodeType:NodeType1 UpgradeDomain:1 UpgradeDomain: ud:/1 Deactivation Intent/Status: None/None
                        
                        Existing Primary Replica -- Nodes with Partition's Existing Primary Replica or Secondary Replicas:
                        --
                        FaultDomain:fd:/0 NodeName:_Node_0 NodeType:NodeType0 UpgradeDomain:0 UpgradeDomain: ud:/0 Deactivation Intent/Status: None/None
                        
                        
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 7/14/2017 4:56:14 PM, LastOk = 1/1/0001 12:00:00 AM

PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | select MinReplicaSetSize,TargetReplicaSetSize

MinReplicaSetSize TargetReplicaSetSize
----------------- --------------------
                2                    7                        

PS C:\> @(Get-ServiceFabricNode).Count
5
```

L’exemple suivant montre l’intégrité d’une partition qui est bloquée dans une reconfiguration parce que l’utilisateur ne respecte pas le jeton d’annulation dans la méthode **RunAsync**. L’examen du rapport d’intégrité de tout réplica marqué comme Principal (P) permet d’explorer plus en profondeur le problème.

```powershell
PS C:\utilities\ServiceFabricExplorer\ClientPackage\lib> Get-ServiceFabricPartitionHealth 0e40fd81-284d-4be4-a665-13bc5a6607ec -ExcludeHealthStatistics 


PartitionId           : 0e40fd81-284d-4be4-a665-13bc5a6607ec
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                                               
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 7
                        SentAt                : 8/27/2017 3:43:09 AM
                        ReceivedAt            : 8/27/2017 3:43:32 AM
                        TTL                   : Infinite
                        Description           : Partition reconfiguration is taking longer than expected.
                        fabric:/app/test1 3 1 0e40fd81-284d-4be4-a665-13bc5a6607ec
                          P/S Ready Node1 131482789658160654
                          S/P Ready Node2 131482789688598467
                          S/S Ready Node3 131482789688598468
                          (Showing 3 out of 3 replicas. Total available replicas: 3)                        
                        
                        For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Ok->Warning = 8/27/2017 3:43:32 AM, LastError = 1/1/0001 12:00:00 AM
```
Ce rapport d’intégrité affiche l’état des réplicas de la partition en cours de reconfiguration : 

```
  P/S Ready Node1 131482789658160654
  S/P Ready Node2 131482789688598467
  S/S Ready Node3 131482789688598468
```

Pour chaque réplica, le rapport d’intégrité contient :
- Rôle de configuration précédente
- Rôle de configuration actuelle
- [État du réplica](service-fabric-concepts-replica-lifecycle.md)
- Nœud sur lequel le réplica est en cours d’exécution
- ID du réplica

Pour notre exemple, d’autres examens sont nécessaires. Examinez l’intégrité de chaque réplica individuel, en commençant par les réplicas marqués comme `Primary` et `Secondary` (131482789658160654 et 131482789688598467) dans l’exemple précédent.

### <a name="replica-constraint-violation"></a>Violation des contraintes de réplicas
**System.PLB** indique un avertissement s’il détecte une violation des contraintes de réplicas et qu’il ne peut pas placer tous les réplicas de la partition. Les détails du rapport montrent quelles contraintes et quelles propriétés empêchent le placement des réplicas.

* **SourceId** : System.PLB
* **Property** : commence par **ReplicaConstraintViolation**.

## <a name="replica-system-health-reports"></a>Rapports d’intégrité du système sur les réplicas
**System.RA**, qui représente le composant Reconfiguration Agent, est l’autorité de l’état des réplicas.

### <a name="state"></a>State
System.RA indique la valeur OK lorsque le réplica a été créé.

* **SourceId** : System.RA
* **Property** : State (État).

L’exemple suivant représente un réplica sain :

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricReplica | where {$_.ReplicaRole -eq "Primary"} | Get-ServiceFabricReplicaHealth

PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
ReplicaId             : 131444422293118721
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131445248920273536
                        SentAt                : 7/14/2017 4:54:52 PM
                        ReceivedAt            : 7/14/2017 4:55:13 PM
                        TTL                   : Infinite
                        Description           : Replica has been created._Node_0
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/14/2017 4:55:13 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="replicaopenstatus-replicaclosestatus-replicachangerolestatus"></a>ReplicaOpenStatus, ReplicaCloseStatus, ReplicaChangeRoleStatus
Cette propriété est utilisée pour indiquer les avertissements ou les défaillances lors de la tentative d’ouverture d’un réplica, de la fermeture d’un réplica ou de la transition d’un réplica d’un rôle à un autre. Pour en savoir plus, consultez [Réplicas et instances](service-fabric-concepts-replica-lifecycle.md). Les défaillances peuvent être des exceptions déclenchées par les appels d’API ou les incidents du processus hôte de service pendant cette période. Pour les défaillances dues aux appels d’API à partir de code C#, Service Fabric ajoute l’exception et l’arborescence des appels de procédure au rapport d’intégrité.

Ces avertissements d’intégrité sont déclenchés après avoir essayé l’action localement un certain nombre de fois (en fonction de la stratégie). Service Fabric retente l’action jusqu’à un seuil maximal. Une fois ce seuil maximal atteint, il peut essayer d’agir pour corriger la situation. Cette tentative risque de supprimer ces avertissements, car l’action effectuée sur ce nœud est abandonnée. Par exemple, si un réplica ne parvient pas à s’ouvrir sur un nœud, Service Fabric déclenche un avertissement d’intégrité. Si après plusieurs tentatives, le réplica ne parvient pas à s’ouvrir, Service Fabric résout le problème lui-même. Cette action peut impliquer d’effectuer la même opération sur un autre nœud. Cette tentative entraîne la suppression de l’avertissement déclenché pour ce réplica. 

* **SourceId** : System.RA
* **Property** : **ReplicaOpenStatus**, **ReplicaCloseStatus** et **ReplicaChangeRoleStatus**.
* **Étapes suivantes** : examinez le code de service ou les vidages sur incident pour identifier la cause de l’échec de l’opération.

L’exemple suivant montre l’intégrité d’un réplica qui déclenche `TargetInvocationException` à partir de sa méthode open. La description contient le point de défaillance (**IStatefulServiceReplica.Open**), le type d’exception (**TargetInvocationException**) et l’arborescence des appels de procédure.

```powershell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId 337cf1df-6cab-4825-99a9-7595090c0b1b -ReplicaOrInstanceId 131483509874784794


PartitionId           : 337cf1df-6cab-4825-99a9-7595090c0b1b
ReplicaId             : 131483509874784794
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='ReplicaOpenStatus', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : ReplicaOpenStatus
                        HealthState           : Warning
                        SequenceNumber        : 131483510001453159
                        SentAt                : 8/27/2017 11:43:20 PM
                        ReceivedAt            : 8/27/2017 11:43:21 PM
                        TTL                   : Infinite
                        Description           : Replica had multiple failures during open on _Node_0 API call: IStatefulServiceReplica.Open(); Error = System.Reflection.TargetInvocationException (-2146232828)
Exception has been thrown by the target of an invocation.
   at Microsoft.ServiceFabric.Replicator.RecoveryManager.d__31.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.LoggingReplicator.d__137.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.DynamicStateManager.d__109.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.TransactionalReplicator.d__79.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.StatefulServiceReplica.d__21.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Services.Runtime.StatefulServiceReplicaAdapter.d__0.MoveNext()

    For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/27/2017 11:43:21 PM, LastOk = 1/1/0001 12:00:00 AM                        
```

L’exemple suivant montre un réplica qui est constamment en panne lors de la fermeture :

```powershell
C:>Get-ServiceFabricReplicaHealth -PartitionId dcafb6b7-9446-425c-8b90-b3fdf3859e64 -ReplicaOrInstanceId 131483565548493142


PartitionId           : dcafb6b7-9446-425c-8b90-b3fdf3859e64
ReplicaId             : 131483565548493142
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='ReplicaCloseStatus', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : ReplicaCloseStatus
                        HealthState           : Warning
                        SequenceNumber        : 131483565611258984
                        SentAt                : 8/28/2017 1:16:01 AM
                        ReceivedAt            : 8/28/2017 1:16:03 AM
                        TTL                   : Infinite
                        Description           : Replica had multiple failures during close on _Node_1. The application 
                        host has crashed.
                        
                        For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 1:16:03 AM, LastOk = 1/1/0001 12:00:00 AM
```

### <a name="reconfiguration"></a>Reconfiguration
Cette propriété est utilisée pour indiquer le moment où un réplica effectuant une [reconfiguration](service-fabric-concepts-reconfiguration.md) détecte que la reconfiguration est bloquée. Ce rapport d’intégrité peut se trouver sur le réplica dont le rôle actuel est principal, sauf dans les cas d’une reconfiguration principale d’échange, où il peut être sur le réplica qui est rétrogradé de principal à secondaire actif.

La reconfiguration peut être bloquée pour l’une des raisons suivantes :

- Une action sur le réplica local (le même réplica que celui effectuant la reconfiguration) ne se termine pas. Dans ce cas, l’examen des rapports d’intégrité sur ce réplica à partir d’autres composants (System.RAP ou System.RE) peut fournir des informations supplémentaires.

- Une action ne se termine pas sur un réplica distant. Les réplicas pour lesquels des actions sont en attente sont affichés dans le rapport d’intégrité. Une analyse approfondie doit être effectuée sur les rapports d’intégrité pour ces réplicas à distance. Il peut également y avoir des problèmes de communication entre ce nœud et le nœud distant.

Dans de rares cas, la reconfiguration peut se bloquer en raison de problèmes de communication ou d’autres problèmes entre ce nœud et le service Failover Manager.

* **SourceId** : System.RA
* **Property** : Reconfiguration.
* **Étapes suivantes** : examinez les réplicas locaux ou distants en fonction de la description du rapport d’intégrité.

L’exemple suivant montre un rapport d’intégrité dans lequel une reconfiguration est bloquée sur le réplica local. Dans cet exemple, la cause de ce blocage s’explique par le fait qu’un service ne respecte pas le jeton d’annulation.

```powershell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId 9a0cedee-464c-4603-abbc-1cf57c4454f3 -ReplicaOrInstanceId 131483600074836703


PartitionId           : 9a0cedee-464c-4603-abbc-1cf57c4454f3
ReplicaId             : 131483600074836703
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='Reconfiguration', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : Reconfiguration
                        HealthState           : Warning
                        SequenceNumber        : 131483600309264482
                        SentAt                : 8/28/2017 2:13:50 AM
                        ReceivedAt            : 8/28/2017 2:13:57 AM
                        TTL                   : Infinite
                        Description           : Reconfiguration is stuck. Waiting for response from the local replica
                        
                        For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 2:13:57 AM, LastOk = 1/1/0001 12:00:00 AM
```

L’exemple suivant montre un rapport d’intégrité dans lequel une reconfiguration est bloquée et attend une réponse de deux réplicas distants. Dans cet exemple, la partition comprend trois réplicas, dont le réplica principal. 

```Powershell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId  579d50c6-d670-4d25-af70-d706e4bc19a2 -ReplicaOrInstanceId 131483956274977415


PartitionId           : 579d50c6-d670-4d25-af70-d706e4bc19a2
ReplicaId             : 131483956274977415
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='Reconfiguration', HealthState='Warning', ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : Reconfiguration
                        HealthState           : Warning
                        SequenceNumber        : 131483960376212469
                        SentAt                : 8/28/2017 12:13:57 PM
                        ReceivedAt            : 8/28/2017 12:14:07 PM
                        TTL                   : Infinite
                        Description           : Reconfiguration is stuck. Waiting for response from 2 replicas
                        
                        Pending Replicas: 
                        P/I Down 40 131483956244554282
                        S/S Down 20 131483956274972403
                        
                        For more information see: https://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 12:07:37 PM, LastOk = 1/1/0001 12:00:00 AM
```

Ce rapport d’intégrité montre que la reconfiguration est bloquée et attend une réponse de deux réplicas : 

```
    P/I Down 40 131483956244554282
    S/S Down 20 131483956274972403
```

Pour chaque réplica, les informations suivantes sont fournies :
- Rôle de configuration précédente
- Rôle de configuration actuelle
- [État du réplica](service-fabric-concepts-replica-lifecycle.md)
- ID du nœud
- ID du réplica

Pour débloquer la reconfiguration :
- Les réplicas **down** doivent être réactivés. 
- Les réplicas **inbuild** doivent terminer la génération et passer à l’état Prêt.

### <a name="slow-service-api-call"></a>Appel lent d’API de service
**System.RAP** et **System.Replicator** indiquent un avertissement si un appel de code de service utilisateur prend plus de temps que la durée configurée. L’avertissement est effacé à l’exécution de l’appel.

* **SourceId** : System.RAP ou System.Replicator
* **Property** : nom de l’API lente. La description fournit plus de détails sur le délai de mise en attente de l’API.
* **Étapes suivantes** : recherchez pourquoi l’appel prend plus de temps que prévu.

L’exemple suivant montre l’événement d’intégrité à partir de System.RAP pour un service fiable qui ne respecte pas le jeton d’annulation dans **RunAsync** :

```powershell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId 5f6060fb-096f-45e4-8c3d-c26444d8dd10 -ReplicaOrInstanceId 131483966141404693


PartitionId           : 5f6060fb-096f-45e4-8c3d-c26444d8dd10
ReplicaId             : 131483966141404693
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='Reconfiguration', HealthState='Warning', ConsiderWarningAsError=false.
                        
HealthEvents          :                         
                        SourceId              : System.RAP
                        Property              : IStatefulServiceReplica.ChangeRole(S)Duration
                        HealthState           : Warning
                        SequenceNumber        : 131483966663476570
                        SentAt                : 8/28/2017 12:24:26 PM
                        ReceivedAt            : 8/28/2017 12:24:56 PM
                        TTL                   : Infinite
                        Description           : The api IStatefulServiceReplica.ChangeRole(S) on _Node_1 is stuck. Start Time (UTC): 2017-08-28 12:23:56.347.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 12:24:56 PM, LastOk = 1/1/0001 12:00:00 AM
                        
```

La propriété et le texte indiquent l’API qui est bloquée. Les étapes ultérieures à suivre pour les diverses API bloquées sont différentes. Toute API sur *IStatefulServiceReplica* ou *IStatelessServiceInstance* correspond généralement à un bogue dans le code de service. La section suivante décrit comment cela se traduit dans le [modèle Reliable Services](service-fabric-reliable-services-lifecycle.md) :

- **IStatefulServiceReplica.Open** : cet avertissement indique qu’un appel à `CreateServiceInstanceListeners` ou `ICommunicationListener.OpenAsync`, ou s’il a été remplacé, à `OnOpenAsync`, est bloqué.

- **IStatefulServiceReplica.Close** et **IStatefulServiceReplica.Abort** : la plupart du temps, un service ne respecte ne pas le jeton d’annulation transmis à `RunAsync`. Cela peut également indiquer que `ICommunicationListener.CloseAsync`, ou si remplacé, `OnCloseAsync` est bloqué.

- **IStatefulServiceReplica.ChangeRole(S)** et **IStatefulServiceReplica.ChangeRole(N)** : la plupart du temps, un service ne respecte ne pas le jeton d’annulation transmis à `RunAsync`. Dans ce scénario, la meilleure solution consiste à redémarrer le réplica.

- **IStatefulServiceReplica.ChangeRole(P)** : la plupart du temps, le service n’a pas renvoyé de tâche à partir de `RunAsync`.

D’autres appels d’API qui peuvent être bloqués se trouvent dans l’interface **IReplicator**. Par exemple :

- **IReplicator.CatchupReplicaSet** : cet avertissement indique une chose parmi deux possibilités. Il n’y a pas assez de réplicas. Pour voir si c’est le cas, regardez le statut de réplica des réplicas se trouvant dans la partition ou dans le rapport d’intégrité de System.FM pour une reconfiguration bloquée. Soit que les réplicas n’accusent pas réception des opérations. La cmdlet PowerShell `Get-ServiceFabricDeployedReplicaDetail` peut être utilisée pour déterminer la progression de tous les réplicas. Le problème se situe dans les réplicas dont la valeur `LastAppliedReplicationSequenceNumber` se trouve derrière la valeur `CommittedSequenceNumber` du réplica principal.

- **IReplicator.BuildReplica(\<Remote ReplicaId>)**  : cet avertissement indique un problème dans le processus de génération. Pour en savoir plus, consultez [Réplicas et instances](service-fabric-concepts-replica-lifecycle.md). Cela peut être dû à une configuration incorrecte de l’adresse du réplicateur. Pour plus d’informations, consultez [Configuration des services fiables (Reliable Services) avec état](service-fabric-reliable-services-configuration.md) et [Spécifier des ressources dans un manifeste de service](service-fabric-service-manifest-resources.md). Il peut également s’agir d’un problème sur le nœud distant.

### <a name="replicator-system-health-reports"></a>Rapports d’intégrité du système sur le réplicateur
**File d’attente de réplication complète :** 
**System.Replicator** indique un avertissement lorsque la file d’attente de réplication est pleine. Sur le réplica principal, la file d’attente de réplication se remplit généralement en raison de la lenteur d’un ou de plusieurs réplicas secondaires à accuser réception des opérations. Sur le rôle secondaire, cela se produit habituellement lorsque le service prend trop de temps pour appliquer les opérations. L’avertissement est effacé une fois que la file d’attente n’est plus pleine.

* **SourceId** : System.Replicator
* **Property** : **PrimaryReplicationQueueStatus** ou **SecondaryReplicationQueueStatus** en fonction du rôle du réplica.
* **Étapes suivantes** : si le rapport se trouve sur le réplica principal, vérifiez la connexion entre les nœuds dans le cluster. Si toutes les connexions sont intègres, il se peut qu’il y ait au moins un réplica secondaire lent avec une latence de disque élevée pour appliquer les opérations. Si le rapport se trouve sur le réplica secondaire, vérifiez d’abord l’utilisation et les performances du disque sur le nœud. Vérifiez ensuite la connexion sortante du nœud lent vers le nœud principal.

**RemoteReplicatorConnectionStatus :** 
**System.Replicator** sur le réplica principal signale un avertissement lorsque la connexion à un duplicateur secondaire (distant) n’est pas intègre. L’adresse du duplicateur distant figure dans le message du rapport, ce qui permet de détecter facilement si la configuration incorrecte a été transmise ou s’il y a des problèmes réseau entre les duplicateurs.

* **SourceId** : System.Replicator
* **Property** : **RemoteReplicatorConnectionStatus**.
* **Étapes suivantes** : vérifiez le message d’erreur et assurez-vous que l’adresse du réplicateur distant est correctement configurée. Par exemple, si le réplicateur distant est ouvert avec l’adresse d’écoute « localhost », il n’est pas joignable depuis l’extérieur. Si l’adresse semble correcte, vérifiez la connexion entre le nœud principal et l’adresse distante pour rechercher d’éventuels problèmes de réseau.

### <a name="replication-queue-full"></a>File d’attente de réplication complète
**System.Replicator** indique un avertissement lorsque la file d’attente de réplication est pleine. Sur le réplica principal, la file d’attente de réplication se remplit généralement en raison de la lenteur d’un ou de plusieurs réplicas secondaires à accuser réception des opérations. Sur le rôle secondaire, cela se produit habituellement lorsque le service prend trop de temps pour appliquer les opérations. L’avertissement est effacé une fois que la file d’attente n’est plus pleine.

* **SourceId** : System.Replicator
* **Property** : **PrimaryReplicationQueueStatus** ou **SecondaryReplicationQueueStatus** en fonction du rôle du réplica.

### <a name="slow-naming-operations"></a>Opérations de nommage lentes
**System.NamingService** signale l’intégrité sur son réplica principal quand une opération de nommage prend trop de temps. [CreateServiceAsync](/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync) et [DeleteServiceAsync](/dotnet/api/system.fabric.fabricclient.servicemanagementclient.deleteserviceasync) sont des exemples d’opérations de nommage. D’autres méthodes se trouvent sous FabricClient. Par exemple, dans les [méthodes de gestion de service](/dotnet/api/system.fabric.fabricclient.servicemanagementclient) ou les [méthodes de gestion de propriété](/dotnet/api/system.fabric.fabricclient.propertymanagementclient).

> [!NOTE]
> Le service de nom résout les noms de service dans un emplacement du cluster. Les utilisateurs peuvent l’utiliser pour gérer les noms et propriétés de service. Il s’agit d’un service persistant partitionné Service Fabric. L’une des partitions représente le *propriétaire de l’autorité*, qui contient des métadonnées sur tous les noms et les services Service Fabric. Les noms Service Fabric sont mappés à des partitions différentes, appelées partitions *Propriétaire du nom*. Ainsi, le service est extensible. En savoir plus sur le [service de nommage](service-fabric-architecture.md).
> 
> 

Quand une opération de nommage prend plus longtemps que prévu, elle est marquée avec un avertissement sur le réplica principal de la partition de service de nommage qui effectue l’opération. Si l’opération se termine avec succès, l’avertissement est effacé. Si l’opération se termine avec une erreur, le rapport d’intégrité inclut des détails sur l’erreur.

* **SourceId** : System.NamingService
* **Property** : commence par le préfixe « **Duration_** » et identifie l’opération lente et le nom Service Fabric sur lequel l’opération est appliquée. Par exemple, si la création de service au niveau du nom **fabric:/MyApp/MyService** prend trop de temps, la propriété est **Duration_AOCreateService.fabric:/MyApp/MyService**. « AO » pointe vers le rôle de la partition de nommage pour ce nom et cette opération.
* **Étapes suivantes** : vérifiez pourquoi l’opération de nommage échoue. Chaque opération peut avoir différentes causes principales. Par exemple, le service de suppression peut être bloqué. Le service peut être bloqué car l’hôte d’application se bloque constamment sur un nœud à cause d’un bogue utilisateur dans le code de service.

L’exemple suivant illustre une opération de création de service. L’opération a duré plus longtemps que la durée configurée. « AO »réessaie et envoie le travail à « NO ». « NO » a terminé la dernière opération avec TIMEOUT. Dans ce cas, le même réplica est principal pour les rôles « AO » et « NO ».

```powershell
PartitionId           : 00000000-0000-0000-0000-000000001000
ReplicaId             : 131064359253133577
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.NamingService', Property='Duration_AOCreateService.fabric:/MyApp/MyService', HealthState='Warning', ConsiderWarningAsError=false.

HealthEvents          :
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131064359308715535
                        SentAt                : 4/29/2016 8:38:50 PM
                        ReceivedAt            : 4/29/2016 8:39:08 PM
                        TTL                   : Infinite
                        Description           : Replica has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 4/29/2016 8:39:08 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.NamingService
                        Property              : Duration_AOCreateService.fabric:/MyApp/MyService
                        HealthState           : Warning
                        SequenceNumber        : 131064359526778775
                        SentAt                : 4/29/2016 8:39:12 PM
                        ReceivedAt            : 4/29/2016 8:39:38 PM
                        TTL                   : 00:05:00
                        Description           : The AOCreateService started at 2016-04-29 20:39:08.677 is taking longer than 30.000.
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 4/29/2016 8:39:38 PM, LastOk = 1/1/0001 12:00:00 AM

                        SourceId              : System.NamingService
                        Property              : Duration_NOCreateService.fabric:/MyApp/MyService
                        HealthState           : Warning
                        SequenceNumber        : 131064360657607311
                        SentAt                : 4/29/2016 8:41:05 PM
                        ReceivedAt            : 4/29/2016 8:41:08 PM
                        TTL                   : 00:00:15
                        Description           : The NOCreateService started at 2016-04-29 20:39:08.689 completed with FABRIC_E_TIMEOUT in more than 30.000.
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 4/29/2016 8:39:38 PM, LastOk = 1/1/0001 12:00:00 AM
```

## <a name="deployedapplication-system-health-reports"></a>Rapports d’intégrité du système sur les applications déployées
**System.Hosting** est l’autorité régnant sur les entités déployées.

### <a name="activation"></a>Activation
System.Hosting consigne la valeur OK lorsqu’une application a été activée sur le nœud. Dans le cas contraire, il indique une erreur.

* **SourceId** : System.Hosting
* **Property** : **Activation**, inclut la version de déploiement.
* **Étapes suivantes** : si l’application est non saine, recherchez la raison de l’échec de l’activation.

L’exemple suivant représente une activation réussie :

```powershell
PS C:\> Get-ServiceFabricDeployedApplicationHealth -NodeName _Node_1 -ApplicationName fabric:/WordCount -ExcludeHealthStatistics

ApplicationName                    : fabric:/WordCount
NodeName                           : _Node_1
AggregatedHealthState              : Ok
DeployedServicePackageHealthStates : 
                                     ServiceManifestName   : WordCountServicePkg
                                     ServicePackageActivationId : 
                                     NodeName              : _Node_1
                                     AggregatedHealthState : Ok
                                     
HealthEvents                       : 
                                     SourceId              : System.Hosting
                                     Property              : Activation
                                     HealthState           : Ok
                                     SequenceNumber        : 131445249083836329
                                     SentAt                : 7/14/2017 4:55:08 PM
                                     ReceivedAt            : 7/14/2017 4:55:14 PM
                                     TTL                   : Infinite
                                     Description           : The application was activated successfully.
                                     RemoveWhenExpired     : False
                                     IsExpired             : False
                                     Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="download"></a>Téléchargement
System.Hosting indique une erreur en cas d’échec du téléchargement du package d’application.

* **SourceId** : System.Hosting
* **Property** : **Download** (Télécharger), inclut la version de déploiement.
* **Étapes suivantes** : recherchez la raison de l’échec du téléchargement sur le nœud.

## <a name="deployedservicepackage-system-health-reports"></a>Rapports d’intégrité du système sur le package de service déployé
**System.Hosting** est l’autorité régnant sur les entités déployées.

### <a name="service-package-activation"></a>Activation du package de service
System.Hosting consigne la valeur OK si l’activation du package de service sur le nœud est réussie. Dans le cas contraire, il indique une erreur.

* **SourceId** : System.Hosting
* **Property** : Activation.
* **Étapes suivantes** : examinez la raison de l’échec de l’activation.

### <a name="code-package-activation"></a>Activation du package de code
System.Hosting indique la valeur OK pour chaque package de code en cas de réussite de l’activation. En cas d’échec de l’activation, il indique un avertissement conformément à la configuration. Si l’activation de **CodePackage** échoue, ou s’il se termine avec une erreur supérieure à la valeur **CodePackageHealthErrorThreshold** configurée, System.Hosting indique une erreur. Si un package de service contient plusieurs packages de code, un rapport d’activation est généré pour chacun d’entre eux.

* **SourceId** : System.Hosting
* **Property** : utilise le préfixe **CodePackageActivation** et contient le nom du package de code et le point d’entrée sous la forme *CodePackageActivation:NomDuPackageDeCode:SetupEntryPoint/PointDEntrée*. Par exemple, **CodePackageActivation:Code:SetupEntryPoint**.

### <a name="service-type-registration"></a>Inscription du type de service
System.Hosting indique la valeur OK si le type de service a été inscrit correctement. Il indique une erreur si l’inscription n’a pas été effectuée à temps, conformément à la configuration via **ServiceTypeRegistrationTimeout**. Si le runtime est fermé, le type de service n’est pas inscrit à partir du nœud et Hosting signale un avertissement.

* **SourceId** : System.Hosting
* **Property** : utilise le préfixe **ServiceTypeRegistration** et contient le nom du type de service. Par exemple, **ServiceTypeRegistration:FileStoreServiceType**.

L’exemple suivant représente un package de service déployé sain :

```powershell
PS C:\> Get-ServiceFabricDeployedServicePackageHealth -NodeName _Node_1 -ApplicationName fabric:/WordCount -ServiceManifestName WordCountServicePkg


ApplicationName            : fabric:/WordCount
ServiceManifestName        : WordCountServicePkg
ServicePackageActivationId : 
NodeName                   : _Node_1
AggregatedHealthState      : Ok
HealthEvents               : 
                             SourceId              : System.Hosting
                             Property              : Activation
                             HealthState           : Ok
                             SequenceNumber        : 131445249084026346
                             SentAt                : 7/14/2017 4:55:08 PM
                             ReceivedAt            : 7/14/2017 4:55:14 PM
                             TTL                   : Infinite
                             Description           : The ServicePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
                             
                             SourceId              : System.Hosting
                             Property              : CodePackageActivation:Code:EntryPoint
                             HealthState           : Ok
                             SequenceNumber        : 131445249084306362
                             SentAt                : 7/14/2017 4:55:08 PM
                             ReceivedAt            : 7/14/2017 4:55:14 PM
                             TTL                   : Infinite
                             Description           : The CodePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
                             
                             SourceId              : System.Hosting
                             Property              : ServiceTypeRegistration:WordCountServiceType
                             HealthState           : Ok
                             SequenceNumber        : 131445249088096842
                             SentAt                : 7/14/2017 4:55:08 PM
                             ReceivedAt            : 7/14/2017 4:55:14 PM
                             TTL                   : Infinite
                             Description           : The ServiceType was registered successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="download"></a>Téléchargement
System.Hosting indique une erreur en cas d’échec du téléchargement du package de service.

* **SourceId** : System.Hosting
* **Property** : **Download** (Télécharger), inclut la version de déploiement.
* **Étapes suivantes** : recherchez la raison de l’échec du téléchargement sur le nœud.

### <a name="upgrade-validation"></a>Validation de mise à niveau
System.Hosting indique une erreur en cas d’échec de la validation lors la mise à niveau ou en cas d’échec de la mise à niveau sur le nœud.

* **SourceId** : System.Hosting
* **Property** : utilise le préfixe **FabricUpgradeValidation** et contient la version de la mise à niveau.
* **Description** : désigne l’erreur rencontrée.

### <a name="undefined-node-capacity-for-resource-governance-metrics"></a>Capacités de nœud non définies pour les mesures de gouvernance des ressources
System.Hosting transmet un avertissement si les capacités de nœud ne sont pas définies dans le manifeste de cluster et si la configuration de la détection automatique est désactivée. Service Fabric émet un avertissement d’intégrité si le package de service qui utilise la [gouvernance des ressources](service-fabric-resource-governance.md) s’inscrit sur un nœud spécifié.

* **SourceId** : System.Hosting
* **Property** : **ResourceGovernance**.
* **Étapes suivantes** : la meilleure façon de résoudre ce problème consiste à modifier le manifeste de cluster pour activer la détection automatique des ressources disponibles. Une autre méthode consiste à mettre à jour le manifeste de cluster en indiquant les capacités de nœud appropriées pour ces mesures.

## <a name="next-steps"></a>Étapes suivantes
* [Affichage rapports d’intégrité de Service Fabric](service-fabric-view-entities-aggregated-health.md)

* [Comment signaler et contrôler l’intégrité du service](service-fabric-diagnostics-how-to-report-and-check-service-health.md)

* [Surveiller et diagnostiquer les services localement](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

* [Mise à niveau des applications Service Fabric](service-fabric-application-upgrade.md)
