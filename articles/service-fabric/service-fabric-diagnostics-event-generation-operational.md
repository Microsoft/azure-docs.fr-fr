---
title: Liste des événements Azure Service Fabric
description: Liste complète des événements fournis par Azure Service Fabric pour faciliter la surveillance des clusters.
author: srrengar
ms.topic: reference
ms.date: 2/25/2019
ms.author: srrengar
ms.openlocfilehash: e69b407bc7d58a83616daa44272ec008ccff9fad
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85846644"
---
# <a name="list-of-service-fabric-events"></a>Liste des événements Service Fabric 

Service Fabric expose un ensemble primaire d’événements de cluster pour vous informer de l’état de votre cluster en tant qu’[événements Service Fabric](service-fabric-diagnostics-events.md). Ils sont basés sur les actions effectuées par Service Fabric sur vos nœuds et votre cluster ou les décisions de gestion effectuées par un opérateur/propriétaire de cluster. Ces événements peuvent être accessibles de plusieurs façons, notamment en configurant les [journaux Azure Monitor avec votre cluster](service-fabric-diagnostics-oms-setup.md) ou en interrogeant l’[EventStore](service-fabric-diagnostics-eventstore.md). Sur les ordinateurs Windows, ces événements sont enregistrés dans l’EventLog afin de pouvoir voir les événements Service Fabric dans l’observateur d’événements. 

Voici certaines caractéristiques de ces événements
* Chaque événement est lié à une entité spécifique dans le cluster, par exemple une application, un service, un nœud ou un réplica.
* Chaque événement contient un ensemble de champs communs : EventInstanceId, EventName et Category.
* Chaque événement contient des champs qui lient l’événement à l’entité à laquelle il est associé. Par exemple, l’événement ApplicationCreated a des champs qui identifient le nom de l’application créée.
* Les événements sont structurés de telle sorte qu’ils peuvent être consommés avec divers outils pour une analyse approfondie. De plus, les détails pertinents d’un événement sont définis en tant que propriétés distinctes par opposition à une longue chaîne. 
* Les événements sont écrits par différents sous-systèmes de Service Fabric et sont identifiés par Source(Task) ci-dessous. Vous trouverez d’autres informations sur ces sous-systèmes dans [Architecture de Service Fabric](service-fabric-architecture.md) et [Présentation technique de Service Fabric](service-fabric-technical-overview.md).

Voici une liste de ces événements Service Fabric organisés par entité.

## <a name="cluster-events"></a>Événements de cluster

**Événements de mise à niveau des clusters**

Vous trouverez plus de détails sur les mises à niveau de cluster [ici](service-fabric-cluster-upgrade-windows-server.md).

| EventId | Nom | Category | Description |Source (tâche) | Level | 
| --- | --- | --- | --- | --- | --- | 
| 29627 | ClusterUpgradeStarted | Mettre à niveau | Une mise à niveau de cluster a commencé | CM | Informationnel |
| 29628 | ClusterUpgradeCompleted | Mettre à niveau | Une mise à niveau de cluster est terminée | CM | Informationnel | 
| 29629 | ClusterUpgradeRollbackStarted | Mettre à niveau | La restauration d’une mise à niveau de cluster a démarré  | CM | Avertissement | 
| 29630 | ClusterUpgradeRollbackCompleted | Mettre à niveau | La restauration d’une mise à niveau de cluster est terminée | CM | Avertissement | 
| 29631 | ClusterUpgradeDomainCompleted | Mettre à niveau | Un domaine de mise à niveau a terminé la mise à niveau pendant une mise à niveau de cluster | CM | Informationnel | 

## <a name="node-events"></a>Événements de nœud

**Événements du cycle de vie des nœuds** 

| EventId | Nom | Category | Description |Source (tâche) | Level |
| --- | --- | ---| --- | --- | --- | 
| 18602 | NodeDeactivateCompleted | StateTransition | La désactivation d’un nœud est terminée | FM | Informationnel | 
| 18603 | NodeUp | StateTransition | Le cluster a détecté qu’un nœud avait démarré | FM | Informationnel | 
| 18604 | NodeDown | StateTransition | Le cluster a détecté qu’un nœud s’est fermé. Lors du redémarrage du nœud, vous verrez un événement NodeDown suivi d’un événement NodeUp |  FM | Error | 
| 18605 | NodeAddedToCluster | StateTransition |  Un nouveau nœud a été ajouté au cluster et Service Fabric peut déployer des applications sur ce nœud | FM | Informationnel | 
| 18606 | NodeRemovedFromCluster | StateTransition |  Un nœud a été supprimé du cluster. Service Fabric ne déploie plus d’applications sur ce nœud | FM | Informationnel | 
| 18607 | NodeDeactivateStarted | StateTransition |  La désactivation d’un nœud a commencé | FM | Informationnel | 
| 25621 | NodeOpenSucceeded | StateTransition |  Un nœud a démarré correctement | FabricNode | Informationnel | 
| 25622 | NodeOpenFailed | StateTransition |  Un nœud n’est pas arrivé à démarrer et à joindre l’anneau | FabricNode | Error | 
| 25624 | NodeClosed | StateTransition |  Un nœud s’est fermé avec succès | FabricNode | Informationnel | 
| 25626 | NodeAborted | StateTransition |  Un nœud s’est arrêté de manière anormale | FabricNode | Error | 

## <a name="application-events"></a>Événements liés aux applications

**Événements du cycle de vie des applications**

| EventId | Nom | Category | Description |Source (tâche) | Level | 
| --- | --- | --- | --- | --- | --- | 
| 29620 | ApplicationCreated | LifeCycle | Une application a été créée | CM | Informationnel | 
| 29625 | ApplicationDeleted | LifeCycle | Une application existante a été supprimée | CM | Informationnel | 
| 23083 | ApplicationProcessExited | LifeCycle | Un processus au sein d’une application s’est terminé | Hébergement | Informationnel | 

**Événements de mise à niveau des applications**

Vous trouverez plus de détails sur les mises à niveau d’application [ici](service-fabric-application-upgrade.md).

| EventId | Nom | Category | Description |Source (tâche) | Level | 
| --- | --- | ---| --- | --- | --- | 
| 29621 | ApplicationUpgradeStarted | Mettre à niveau | Une mise à niveau d’application a commencé | CM | Informationnel | 
| 29622 | ApplicationUpgradeCompleted | Mettre à niveau | Une mise à niveau d’application s’est terminée | CM | Informationnel | 
| 29623 | ApplicationUpgradeRollbackStarted | Mettre à niveau | La restauration d’une mise à niveau d’application a démarré |CM | Avertissement | 
| 29624 | ApplicationUpgradeRollbackCompleted | Mettre à niveau | La restauration d’une mise à niveau d’application s’est terminée | CM | Avertissement | 
| 29626 | ApplicationUpgradeDomainCompleted | Mettre à niveau | Un domaine de mise à niveau a terminé la mise à niveau pendant une mise à niveau d’application | CM | Informationnel | 

## <a name="service-events"></a>Événements de service

**Événements du cycle de vie du service**

| EventId | Nom | Category | Description |Source (tâche) | Level | 
| --- | --- | ---| --- | --- | --- |
| 18657 | ServiceCreated | LifeCycle | Un service a été créé | FM | Informationnel | 
| 18658 | ServiceDeleted | LifeCycle | Un service existant a été supprimé | FM | Informationnel | 

## <a name="partition-events"></a>Événements de partition

**Événements de déplacement de partition**

| EventId | Nom | Category | Description |Source (tâche) | Level | 
| --- | --- | ---| --- | --- | --- |
| 18940 | PartitionReconfigured | LifeCycle | La reconfiguration d’une partition s’est terminée | RA | Informationnel | 

## <a name="replica-events"></a>Événements de réplica

**Événements de cycle de vie du réplica**

| EventId | Nom | Category | Description |Source (tâche) | Level |
| --- | --- | ---| --- | --- | --- |
| 61701 | ReliableDictionaryOpened | LifeCycle | Le dictionnaire fiable s’est ouvert | DistributedDictionary | Informationnel |
| 61702 | ReliableDictionaryClosed | LifeCycle | Le dictionnaire fiable s’est fermé | DistributedDictionary | Informationnel |
| 61703 | ReliableDictionaryCheckpointRecovered | LifeCycle | Le dictionnaire fiable a récupéré son point de contrôle | DistributedDictionary | Informationnel |
| 61704 | ReliableDictionaryCheckpointFilesSent | LifeCycle | Le réplica a envoyé les fichiers de point de contrôle du dictionnaire fiable | DistributedDictionary | Informationnel |
| 61705 | ReliableDictionaryCheckpointFilesReceived | LifeCycle | Le réplica a reçu les fichiers de point de contrôle du dictionnaire fiable | DistributedDictionary | Informationnel |
| 61963 | ReliableQueueOpened | LifeCycle | La file d’attente fiable s’est ouverte | DistributedQueue | Informationnel |
| 61964 | ReliableQueueClosed | LifeCycle | La file d’attente fiable s’est fermée | DistributedQueue | Informationnel |
| 61965 | ReliableQueueCheckpointRecovered | LifeCycle | La file d’attente fiable a récupéré son point de contrôle | DistributedQueue | Informationnel |
| 61966 | ReliableQueueCheckpointFilesSent | LifeCycle | Le réplica a envoyé les fichiers de point de contrôle de la file d’attente fiable | DistributedQueue | Informationnel |
| 63647 | ReliableQueueCheckpointFilesReceived | LifeCycle | Le réplica a reçu les fichiers de point de contrôle de la file d’attente fiable | DistributedQueue | Informationnel |
| 63648 | ReliableConcurrentQueueOpened | LifeCycle | La file d’attente simultanée fiable s’est ouverte | ReliableConcurrentQueue | Informationnel |
| 63649 | ReliableConcurrentQueueClosed | LifeCycle | La file d’attente simultanée fiable s’est fermée | ReliableConcurrentQueue | Informationnel |
| 63650 | ReliableConcurrentQueueCheckpointRecovered | LifeCycle | La file d’attente simultanée fiable a récupéré son point de contrôle | ReliableConcurrentQueue | Informationnel |
| 61687 | TStoreError | Échec | La collection fiable a reçu une erreur inattendue | TStore | Error |
| 63831 | PrimaryFullCopyInitiated | LifeCycle | Le réplica principal a lancé une copie complète | TReplicator | Informationnel |
| 63832 | PrimaryPartialCopyInitiated | LifeCycle | Le réplica principal a lancé une copie partielle | TReplicator | Informationnel |
| 16831 | BuildIdleReplicaStarted | LifeCycle | Le réplica principal a lancé la création d’un réplica inactif | Réplication | Informationnel |
| 16832 | BuildIdleReplicaCompleted | LifeCycle | Le réplica principal a terminé la création d’un réplica inactif | Réplication | Informationnel |
| 16833 | BuildIdleReplicaFailed | LifeCycle | Le réplica principal n’a pas réussi à créer un réplica inactif | Réplication | Avertissement |
| 16834 | PrimaryReplicationQueueFull | Intégrité | La file d’attente de réplication du réplica principal est pleine | Réplication | Avertissement |
| 16835 | PrimaryReplicationQueueWarning | Intégrité | La file d’attente de réplication du réplica principal est presque pleine | Réplication | Avertissement |
| 16836 | PrimaryReplicationQueueWarningMitigated | Intégrité | La file d’attente de réplication du réplica principal est correcte | Réplication | Informationnel |
| 16837 | SecondaryReplicationQueueFull | Intégrité | La file d’attente de réplication du réplica secondaire est pleine | Réplication | Avertissement |
| 16838 | SecondaryReplicationQueueWarning | Intégrité | La file d’attente de réplication du réplica secondaire est presque pleine | Réplication | Avertissement |
| 16839 | SecondaryReplicationQueueWarningMitigated | Intégrité | La file d’attente de réplication du réplica secondaire est correcte | Réplication | Informationnel |
| 16840 | PrimaryFaultedSlowSecondary | Intégrité | Le réplica principal n’a pas réussi à créer un réplica secondaire lent | Réplication | Avertissement |
| 16841 | ReplicatorFaulted | Intégrité | Le réplica a subi une défaillance | Réplication | Avertissement |

## <a name="container-events"></a>Événements de conteneur

**Événements du cycle de vie des conteneurs** 

| EventId | Nom | Description |Source (tâche) | Level | Version |
| --- | --- | ---| --- | --- | --- |
| 23074 | ContainerActivated | Un conteneur a démarré | Hébergement | Informationnel | 1 |
| 23075 | ContainerDeactivated | Un conteneur s’est arrêté | Hébergement | Informationnel | 1 |
| 23082 | ContainerExited | Un conteneur s’est arrêté : vérifiez l’indicateur UnexpectedTermination | Hébergement | Informationnel | 1 |

## <a name="health-reports"></a>Rapports d'intégrité

Le [modèle d’intégrité Service Fabric](service-fabric-health-introduction.md) fournit une évaluation et des rapports d’intégrité complets, flexibles et extensibles. À partir de Service Fabric version 6.2, les données d’intégrité sont écrites en tant qu’événements de plateforme pour fournir des enregistrements historiques d’intégrité. Pour limiter le volume d’événements d’intégrité, nous écrivons uniquement les éléments suivants en tant qu’événements Service Fabric :

* Tous les rapports d’intégrité `Error` ou `Warning`
* Les rapports d’intégrité `Ok` lors des transitions
* Quand un événement d’intégrité `Error` ou `Warning` expire. Cela peut servir à déterminer la durée pendant laquelle une entité n’a pas été saine

**Événements du rapport d’intégrité des clusters**

| EventId | Nom | Description |Source (tâche) | Level | Version |
| --- | --- | --- | --- | --- | --- |
| 54428 | ClusterNewHealthReport | Un nouveau rapport d’intégrité de cluster est disponible | HM | Informationnel | 1 |
| 54437 | ClusterHealthReportExpired | Un rapport d’intégrité de cluster existant a expiré | HM | Informationnel | 1 |

**Événements du rapport d’intégrité des nœuds**

| EventId | Nom | Description |Source (tâche) | Level | Version |
| --- | --- | ---| --- | --- | --- |
| 54423 | NodeNewHealthReport | Un nouveau rapport d’intégrité de nœud est disponible | HM | Informationnel | 1 |
| 54432 | NodeHealthReportExpired | Un rapport d’intégrité de nœud existant a expiré | HM | Informationnel | 1 |

**Événements du rapport d’intégrité des applications**

| EventId | Nom | Description |Source (tâche) | Level | Version |
| --- | --- | ---| --- | --- | --- |
| 54425 | ApplicationNewHealthReport | Un rapport d’intégrité d’application a été créé. Il concerne les applications non déployées. | HM | Informationnel | 1 |
| 54426 | DeployedApplicationNewHealthReport | Un rapport d’intégrité d’application déployée a été créé | HM | Informationnel | 1 |
| 54427 | DeployedServicePackageNewHealthReport | Un rapport d’intégrité de service déployée a été créé | HM | Informationnel | 1 |
| 54434 | ApplicationHealthReportExpired | Un rapport d’intégrité d’application existant a expiré | HM | Informationnel | 1 |
| 54435 | DeployedApplicationHealthReportExpired | Un rapport d’intégrité d’application déployée existant a expiré | HM | Informationnel | 1 |
| 54436 | DeployedServicePackageHealthReportExpired | Un rapport d’intégrité de service déployé existant a expiré | HM | Informationnel | 1 |

**Événements du rapport d’intégrité du service**

| EventId | Nom | Description |Source (tâche) | Level | Version |
| --- | --- | ---| --- | --- | --- |
| 54424 | ServiceNewHealthReport | Un rapport d’intégrité de service a été créé | HM | Informationnel | 1 |
| 54433 | ServiceHealthReportExpired | Un rapport d’intégrité de service existant a expiré | HM | Informationnel | 1 |

**Événements du rapport d’intégrité des partitions**

| EventId | Nom | Description |Source (tâche) | Level | Version |
| --- | --- | ---| --- | --- | --- |
| 54422 | PartitionNewHealthReport | Un rapport d’intégrité de partition a été créé | HM | Informationnel | 1 |
| 54431 | PartitionHealthReportExpired | Un rapport d’intégrité de partition existant a expiré | HM | Informationnel | 1 |

**Événements du rapport d’intégrité de réplica**

| EventId | Nom | Description |Source (tâche) | Level | Version |
| --- | --- | ---| --- | --- | --- |
| 54429 | StatefulReplicaNewHealthReport | Un rapport d’intégrité de réplica avec état a été créé | HM | Informationnel | 1 |
| 54430 | StatelessInstanceNewHealthReport | Un rapport d’intégrité d’instance sans état a été créé | HM | Informationnel | 1 |
| 54438 | StatefulReplicaHealthReportExpired | Un rapport d’intégrité de réplica avec état existant a expiré | HM | Informationnel | 1 |
| 54439 | StatelessInstanceHealthReportExpired | Un rapport d’intégrité d’instance sans état existant a expiré | HM | Informationnel | 1 |

## <a name="chaos-testing-events"></a>Événements de test par le chaos 

**Événements de session de test par le chaos**

| EventId | Nom | Description |Source (tâche) | Level | Version |
| --- | --- | ---| --- | --- | --- |
| 50021 | ChaosStarted | Une session de test par le chaos a démarré | Testabilité | Informationnel | 1 |
| 50023 | ChaosStopped | Une session de test par le chaos s’est arrêtée | Testabilité | Informationnel | 1 |

**Événements du nœud de chaos**

| EventId | Nom | Description |Source (tâche) | Level | Version |
| --- | --- | ---| --- | --- | --- |
| 50033 | ChaosNodeRestartScheduled | Un nœud est planifié pour redémarrer dans le cadre d’une session de test par le chaos | Testabilité | Informationnel | 1 |
| 50087 | ChaosNodeRestartCompleted | Un nœud a fini de redémarrer dans le cadre d’une session de test par le chaos | Testabilité | Informationnel | 1 |

**Événements liés aux applications de chaos**

| EventId | Nom | Description |Source (tâche) | Level | Version |
| --- | --- | ---| --- | --- | --- |
| 50053 | ChaosCodePackageRestartScheduled | Un redémarrage de package de code a été planifié pendant une session de test par le chaos | Testabilité | Informationnel | 1 |
| 50101 | ChaosCodePackageRestartCompleted | Un redémarrage de package de code s’est terminé pendant une session de test par le chaos | Testabilité | Informationnel | 1 |

**Événements de partition de chaos**

| EventId | Nom | Description |Source (tâche) | Level | Version |
| --- | --- | ---| --- | --- | --- |
| 50069 | ChaosPartitionPrimaryMoveScheduled | Une partition principale est planifiée pour redémarrer dans le cadre d’une session de test par le chaos | Testabilité | Informationnel | 1 |
| 50077 | ChaosPartitionSecondaryMoveScheduled | Une partition secondaire est planifiée pour redémarrer dans le cadre d’une session de test par le chaos | Testabilité | Informationnel | 1 |
| 65003 | PartitionPrimaryMoveAnalysis | Une analyse plus approfondie du déplacement de la partition principale est disponible | Testabilité | Informationnel | 1 |

**Événements de réplica de chaos**

| EventId | Nom | Description |Source (tâche) | Level | Version |
| --- | --- | ---| --- | --- | --- |
| 50047 | ChaosReplicaRestartScheduled | Un redémarrage de réplica a été planifié pendant une session de test par le chaos | Testabilité | Informationnel | 1 |
| 50051 | ChaosReplicaRemovalScheduled | Une suppression de réplica a été planifiée pendant une session de test par le chaos | Testabilité | Informationnel | 1 |
| 50093 | ChaosReplicaRemovalCompleted | Une suppression de réplica s’est terminée pendant une session de test par le chaos | Testabilité | Informationnel | 1 |

## <a name="other-events"></a>Autres événements

**Événements de corrélation**

| EventId | Nom | Description |Source (tâche) | Level | Version |
| --- | --- | ---| --- | --- | --- |
| 65011 | CorrelationOperational | Une corrélation a été détectée | Testabilité | Informationnel | 1 |

## <a name="events-prior-to-version-62"></a>Événements antérieurs à la version 6.2

Voici la liste complète des événements fournis par Service Fabric avant la version 6.2.

| EventId | Nom | Source (tâche) | Level |
| --- | --- | --- | --- |
| 25620 | NodeOpening | FabricNode | Informationnel |
| 25621 | NodeOpenedSuccess | FabricNode | Informationnel |
| 25622 | NodeOpenedFailed | FabricNode | Informationnel |
| 25623 | NodeClosing | FabricNode | Informationnel |
| 25624 | NodeClosed | FabricNode | Informationnel |
| 25625 | NodeAborting | FabricNode | Informationnel |
| 25626 | NodeAborted | FabricNode | Informationnel |
| 29627 | ClusterUpgradeStart | CM | Informationnel |
| 29628 | ClusterUpgradeComplete | CM | Informationnel |
| 29629 | ClusterUpgradeRollback | CM | Informationnel |
| 29630 | ClusterUpgradeRollbackComplete | CM | Informationnel |
| 29631 | ClusterUpgradeDomainComplete | CM | Informationnel |
| 23074 | ContainerActivated | Hébergement | Informationnel |
| 23075 | ContainerDeactivated | Hébergement | Informationnel |
| 29620 | ApplicationCreated | CM | Informationnel |
| 29621 | ApplicationUpgradeStart | CM | Informationnel |
| 29622 | ApplicationUpgradeComplete | CM | Informationnel |
| 29623 | ApplicationUpgradeRollback | CM | Informationnel |
| 29624 | ApplicationUpgradeRollbackComplete | CM | Informationnel |
| 29625 | ApplicationDeleted | CM | Informationnel |
| 29626 | ApplicationUpgradeDomainComplete | CM | Informationnel |
| 18566 | ServiceCreated | FM | Informationnel |
| 18567 | ServiceDeleted | FM | Informationnel |

## <a name="next-steps"></a>Étapes suivantes

* Obtenir une vue d’ensemble des [diagnostics dans Service Fabric](service-fabric-diagnostics-overview.md)
* En savoir plus sur le service EventStore dans [Vue d’ensemble du service EventStore Service Fabric](service-fabric-diagnostics-eventstore.md)
* Modification de votre configuration [Diagnostics Azure](service-fabric-diagnostics-event-aggregation-wad.md) pour collecter davantage de journaux
* [Configuration d’Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md) pour voir les journaux d’activité de votre canal opérationnel
