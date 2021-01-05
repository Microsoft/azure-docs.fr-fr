---
title: Service EventStore Azure Service Fabric
description: Découvrez le service EventStore d’Azure Service Fabric, un Moyen de comprendre et de surveiller l’état d’un cluster ou de charges de travail à tout Moment.
author: srrengar
ms.topic: conceptual
ms.date: 6/6/2019
ms.author: srrengar
ms.openlocfilehash: 0b6258e160794eaf7d0c05775f8fd3b796e3ba47
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91354817"
---
# <a name="eventstore-overview"></a>Vue d’ensemble d’EventStore

>[!NOTE]
>Depuis Service Fabric version 6.4, les API EventStore sont réservées aux clusters Windows s’exécutant sur Azure uniquement. Nous travaillons au portage de cette fonctionnalité vers Linux et vers nos clusters autonomes.

## <a name="overview"></a>Vue d’ensemble

Introduit dans la version 6.2, le service EventStore est une option de supervision de Service Fabric. EventStore offre un moyen de comprendre l’état de votre cluster ou de vos charges de travail à un moment donné dans le temps.
EventStore est un service Service Fabric avec état qui conserve les événements du cluster. Les événements sont exposés via Service Fabric Explorer, REST et les API. EventStore interroge le cluster directement pour obtenir des données de diagnostic sur une entité de votre cluster et doit être utilisé pour aider à :

* Diagnostiquer les problèmes de développement ou de test, ou lorsque vous utilisez peut-être un pipeline de surveillance.
* Vérifier que les actions de gestion que vous entreprenez sur votre cluster sont traitées correctement
* Obtenir un « instantané » de la façon dont interagit Service Fabric avec une entité en particulier

![Capture d'écran représentant l'onglet ÉVÉNEMENTS du volet Nœuds et affichant plusieurs événements, dont un événement NodeDown.](media/service-fabric-diagnostics-eventstore/eventstore.png)

Pour obtenir la liste complète des événements disponibles dans le service EventStore, consultez l’article relatif aux [événements Service Fabric](service-fabric-diagnostics-event-generation-operational.md).

>[!NOTE]
>Depuis Service Fabric version 6.4, les API et l’expérience utilisateur EventStore sont mis à la disposition générale pour les clusters Azure Windows. Nous travaillons au portage de cette fonctionnalité vers Linux et vers nos clusters autonomes.

Vous pouvez interroger le service EventStore à propos des événements qui sont disponibles pour chaque entité et type d’entité du cluster. Cela signifie que vous pouvez rechercher des événements aux niveaux suivants :
* Cluster : événements spécifiques au cluster lui-même (ex : mise à niveau du cluster)
* Nœuds : événements à tous les niveaux de nœuds
* Nœud : événements spécifiques à un nœud identifié avec `nodeName`
* Applications : événements à tous les niveaux des applications
* Application : événements spécifiques à une application identifiée avec `applicationId`
* Services : événements de tous les services des clusters
* Service : événements d’un service spécifique identifié avec `serviceId`
* Partitions : événements de toutes les partitions
* Partition : événements d’une partition spécifique identifiée avec `partitionId`
* Réplicas de partition : événements de l’ensemble des réplicas / instances d’une partition spécifique identifiée avec `partitionId`
* Réplica de partition : événements d’un réplica/d’une instance spécifique identifié(e) avec `replicaId` et `partitionId`

Pour en savoir plus sur l’API, consultez les [Informations de référence sur les API EventStore](/rest/api/servicefabric/sfclient-index-eventsstore).

Le service EventStore a également la possibilité de mettre en corrélation les événements du cluster. En examinant les événements écrits en même temps à partir de différentes entités dont les conséquences peuvent être mutuelles, le service EventStore est en mesure de lier ces événements pour aider à identifier les causes des activités du cluster. Par exemple, si l’une de vos applications n’est plus saine sans aucune modification forcée, le service EventStore examine également d’autres événements exposés par la plateforme et peut les mettre en corrélation avec un événement `Error` ou `Warning`. Cela contribue à accélérer la détection des défaillances et l’analyse des causes racines.

## <a name="enable-eventstore-on-your-cluster"></a>Activer EventStore sur votre cluster

### <a name="local-cluster"></a>Cluster local

Dans le [fabricSettings.json de votre cluster](service-fabric-cluster-fabric-settings.md), ajoutez EventStoreService en tant que composant additionnel et mettez à niveau le cluster.

```json
    "addOnFeatures": [
        "EventStoreService"
    ],
```

### <a name="azure-cluster-version-65"></a>Cluster Azure 6.5 et versions ultérieures
Si votre cluster Azure est mis à niveau vers la version 6.5 ou une version ultérieure, EventStore est automatiquement activé sur votre cluster. Pour refuser, vous devez mettre à jour votre modèle de cluster comme suit :

* Utilisez la version d’API datée du `2019-03-01` ou une version ultérieure 
* Ajoutez le code suivant à votre section Propriétés dans votre cluster
  ```json  
    "fabricSettings": [
      …
    ],
    "eventStoreServiceEnabled": false
  ```

### <a name="azure-cluster-version-64"></a>Cluster Azure 6.4

Si vous utilisez la version 6.4, vous pouvez modifier votre modèle Azure Resource Manager pour activer le service EventStore. Pour cela, effectuez une [mise à niveau de la configuration de cluster](service-fabric-cluster-config-upgrade-azure.md) et ajoutez le code suivant. Vous pouvez utiliser le paramètre PlacementConstraints pour placer les réplicas du service EventStore sur un NodeType spécifique, par exemple, un NodeType dédié pour les services système. La section `upgradeDescription` configure la mise à niveau de configuration pour déclencher un redémarrage sur les nœuds. Vous pouvez supprimer la section dans une autre mise à jour.

```json
    "fabricSettings": [
          …
          …
          …,
         {
            "name": "EventStoreService",
            "parameters": [
              {
                "name": "TargetReplicaSetSize",
                "value": "3"
              },
              {
                "name": "MinReplicaSetSize",
                "value": "1"
              },
              {
                "name": "PlacementConstraints",
                "value": "(NodeType==<node_type_name_here>)"
              }
            ]
          }
        ],
        "upgradeDescription": {
          "forceRestart": true,
          "upgradeReplicaSetCheckTimeout": "10675199.02:48:05.4775807",
          "healthCheckWaitDuration": "00:01:00",
          "healthCheckStableDuration": "00:01:00",
          "healthCheckRetryTimeout": "00:5:00",
          "upgradeTimeout": "1:00:00",
          "upgradeDomainTimeout": "00:10:00",
          "healthPolicy": {
            "maxPercentUnhealthyNodes": 100,
            "maxPercentUnhealthyApplications": 100
          },
          "deltaHealthPolicy": {
            "maxPercentDeltaUnhealthyNodes": 0,
            "maxPercentUpgradeDomainDeltaUnhealthyNodes": 0,
            "maxPercentDeltaUnhealthyApplications": 0
          }
        }
```


## <a name="next-steps"></a>Étapes suivantes
* Bien démarrer avec l’API EventStore API - [Utilisation des API EventStore dans les clusters Azure Service Fabric](service-fabric-diagnostics-eventstore-query.md)
* En savoir plus sur la liste des événements proposés par EventStore - [Événements Service Fabric](service-fabric-diagnostics-event-generation-operational.md)
* Vue d’ensemble de la surveillance et des diagnostics de Service Fabric : [Monitoring et diagnostics pour Azure Service Fabric](service-fabric-diagnostics-overview.md)
* Afficher la liste complète des appels d’API - [Informations de référence sur les API REST EventStore](/rest/api/servicefabric/sfclient-index-eventsstore)
* Pour plus d’informations sur la surveillance du cluster, consultez [Monitoring du cluster et de la plateforme](service-fabric-diagnostics-event-generation-infra.md).
