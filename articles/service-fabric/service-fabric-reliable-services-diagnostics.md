---
title: Diagnostics pour Reliable Services avec état dans Azure Service Fabric
description: Fonctionnalité de diagnostic pour Reliable Services avec état dans Azure Service Fabric
ms.topic: conceptual
ms.date: 8/24/2018
ms.openlocfilehash: 5a3831dd4f8d5402980fac3daf8c35d9884c852d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91840759"
---
# <a name="diagnostic-functionality-for-stateful-reliable-services"></a>Fonctionnalité de diagnostic pour Reliable Services avec état
La classe StatefulServiceBase de Reliable Services avec état dans Azure Service Fabric émet des événements [EventSource](/dotnet/api/system.diagnostics.tracing.eventsource?view=netcore-3.1) qui peuvent être utilisés pour déboguer le service, fournir des informations sur le fonctionnement du runtime et vous aider lors du dépannage.

## <a name="eventsource-events"></a>Événements EventSource
Le nom EventSource de la classe StatefulServiceBase de Reliable Services avec état est « Microsoft-ServiceFabric-Services ». Les événements issus de cette source d’événements s’affichent dans la fenêtre [Événements de diagnostics](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md#view-service-fabric-system-events-in-visual-studio) lorsque le service est en cours de [débogage dans Visual Studio](service-fabric-debugging-your-application.md).

[PerfView](https://www.microsoft.com/download/details.aspx?id=28567), [Diagnostics Azure](../cloud-services/cloud-services-dotnet-diagnostics.md) et [Microsoft TraceEvent Library](https://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.TraceEvent) sont des exemples d’outils et de technologies permettant de collecter et/ou d’afficher des événements EventSource.

## <a name="events"></a>Événements
| Nom d'événement | ID de l’événement | Level | Description de l'événement |
| --- | --- | --- | --- |
| StatefulRunAsyncInvocation |1 |Informationnel |Émis lorsque la tâche de service RunAsync est démarrée |
| StatefulRunAsyncCancellation |2 |Informationnel |Émis lorsque la tâche de service RunAsync est annulée |
| StatefulRunAsyncCompletion |3 |Informationnel |Émis lorsque la tâche de service RunAsync est terminée |
| StatefulRunAsyncSlowCancellation |4 |Avertissement |Émis lorsque la tâche de service RunAsync prend trop de temps pour terminer l'annulation |
| StatefulRunAsyncFailure |5 |Error |Émis lorsque la tâche de service RunAsync renvoie une exception |

## <a name="interpret-events"></a>Interprétation des événements
Les événements StatefulRunAsyncInvocation, StatefulRunAsyncCompletion et StatefulRunAsyncCancellation sont utiles à l’enregistreur du service pour comprendre le cycle de vie d’un service, ainsi que le minutage lorsqu’un service démarre, est annulé ou se termine. Cette information peut être utile lors du débogage de problèmes du service ou pour comprendre le cycle de vie du service.

Les enregistreurs de service doivent prêter attention aux événements StatefulRunAsyncSlowCancellation et StatefulRunAsyncFailure qui indiquent des problèmes liés au service.

StatefulRunAsyncFailure est émis à chaque fois que la tâche RunAsync() du service lève une exception. En général, une exception levée indique une erreur ou un bogue dans le service. En outre, l’exception entraîne l’échec du service, qui est alors déplacé vers un autre nœud. Cette opération peut s’avérer coûteuse et peut retarder les requêtes entrantes pendant le déplacement du service. Les enregistreurs de service doivent déterminer la cause de l’exception et, si possible, la résoudre.

StatefulRunAsyncSlowCancellation est émis à chaque fois qu’une requête d’annulation de la tâche RunAsync prend plus de quatre secondes. Lorsqu’un service prend trop de temps pour terminer l’annulation, cela a un impact sur la capacité du service à être redémarré rapidement sur un autre nœud. Ce scénario peut avoir un impact sur la disponibilité globale du service.

## <a name="performance-counters"></a>Compteurs de performance
Le runtime Reliable Services définit les catégories de compteurs de performances suivantes :

| Category | Description |
| --- | --- |
| Réplicateur transactionnel Service Fabric |Compteurs spécifiques au réplicateur transactionnel Azure Service Fabric |
| Service Fabric TStore |Compteurs spécifiques à Azure Service Fabric TStore |

Le réplicateur transactionnel Service Fabric est utilisé par le [Gestionnaire d’état fiable](./service-fabric-reliable-services-introduction.md) pour répliquer les transactions au sein d’un ensemble donné de [réplicas](service-fabric-concepts-replica-lifecycle.md).

Service Fabric TStore est un composant utilisé dans les [Collections fiables](./service-fabric-reliable-services-introduction.md) pour stocker et récupérer des paires clé-valeur.

L'application [Analyseur de performances Windows](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc749249(v=ws.11)) , disponible par défaut dans le système d'exploitation Windows, peut être utilisée pour collecter et afficher les données de compteur de performances. [Diagnostics Azure](../cloud-services/cloud-services-dotnet-diagnostics.md) est une autre option pour collecter les données de compteur de performances et les télécharger dans les tables Azure.

### <a name="performance-counter-instance-names"></a>Noms d'instance de compteur de performances
Un cluster avec un grand nombre de services fiables ou de partitions de services fiables disposera d'un grand nombre d’instances de compteur de performances du réplicateur transactionnel. Cela est également le cas pour les compteurs de performances TStore, mais est aussi multiplié par le nombre de Files d’attente fiables et Dictionnaires fiables utilisés. Les noms d’instance de compteur de performances peuvent aider à identifier la [partition](service-fabric-concepts-partitioning.md), le réplica de service et, dans le cas de TStore, le fournisseur d’état auxquels l’instance de compteur de performances est associée.

#### <a name="service-fabric-transactional-replicator-category"></a>Catégorie de réplicateur transactionnel Service Fabric
Pour la catégorie `Service Fabric Transactional Replicator`, les noms d'instance de compteur ont le format suivant :

`ServiceFabricPartitionId:ServiceFabricReplicaId`

*ServiceFabricPartitionId* est la représentation sous forme de chaîne de l'ID de partition Service Fabric associée à l'instance de compteur de performances. L'ID de partition est un GUID et sa représentation sous forme de chaîne est générée via [`Guid.ToString`](/dotnet/api/system.guid.tostring?view=netcore-3.1#System_Guid_ToString_System_String_) avec le spécificateur de format « D ».

*ServiceFabricReplicaId* est l’ID associé à un réplica donné d’un service fiable. L’ID de réplica est inclus dans le nom de l’instance de compteur de performances pour garantir son unicité et éviter tout conflit avec d’autres instances de compteur de performances générées par la même partition. Vous trouverez [ici](service-fabric-concepts-replica-lifecycle.md) d’autres détails sur les réplicas et leur rôle dans les services fiables.

Le nom d’instance de compteur suivant est typique d’un compteur de la catégorie `Service Fabric Transactional Replicator` :

`00d0126d-3e36-4d68-98da-cc4f7195d85e:131652217797162571`

Dans l’exemple précédent, `00d0126d-3e36-4d68-98da-cc4f7195d85e` est la représentation sous forme de chaîne de l’ID de partition de Service Fabric et `131652217797162571` est l’ID du réplica.

#### <a name="service-fabric-tstore-category"></a>Catégorie Service Fabric TStore
Pour la catégorie `Service Fabric TStore`, les noms d'instance de compteur ont le format suivant :

`ServiceFabricPartitionId:ServiceFabricReplicaId:StateProviderId_PerformanceCounterInstanceDifferentiator_StateProviderName`

*ServiceFabricPartitionId* est la représentation sous forme de chaîne de l'ID de partition Service Fabric associée à l'instance de compteur de performances. L'ID de partition est un GUID et sa représentation sous forme de chaîne est générée via [`Guid.ToString`](/dotnet/api/system.guid.tostring?view=netcore-3.1#System_Guid_ToString_System_String_) avec le spécificateur de format « D ».

*ServiceFabricReplicaId* est l’ID associé à un réplica donné d’un service fiable. L’ID de réplica est inclus dans le nom de l’instance de compteur de performances pour garantir son unicité et éviter tout conflit avec d’autres instances de compteur de performances générées par la même partition. Vous trouverez [ici](service-fabric-concepts-replica-lifecycle.md) d’autres détails sur les réplicas et leur rôle dans les services fiables.

*StateProviderId* est l’ID associé à un fournisseur d’état au sein d’un service fiable. L’ID du fournisseur d’état est inclus dans le nom de l’instance de compteur de performances pour différencier un TStore d’un autre.

*PerformanceCounterInstanceDifferentiator* est un ID de différenciation associé à une instance de compteur de performances au sein d’un fournisseur d’état. Cet élément différenciateur est inclus dans le nom de l’instance de compteur de performances pour garantir son unicité et éviter tout conflit avec d’autres instances de compteur de performances générées par le même fournisseur d’état.

*StateProviderName* est le nom associé à un fournisseur d’état au sein d’un service fiable. Le nom du fournisseur d’état est inclus dans le nom de l’instance de compteur de performance pour permettre aux utilisateurs d’identifier facilement l’état qu’il fournit.

Le nom d’instance de compteur suivant est typique d’un compteur de la catégorie `Service Fabric TStore` :

`00d0126d-3e36-4d68-98da-cc4f7195d85e:131652217797162571:142652217797162571_1337_urn:MyReliableDictionary/dataStore`

Dans l’exemple précédent, `00d0126d-3e36-4d68-98da-cc4f7195d85e` est la représentation sous forme de chaîne de l’ID de partition Service Fabric, `131652217797162571` est l’ID de réplica, `142652217797162571` est l’ID de fournisseur d’état et `1337` est l’élément différenciateur d’instance de compteur de performances. `urn:MyReliableDictionary/dataStore` est le nom du fournisseur d’état qui stocke les données pour la collection nommée `urn:MyReliableDictionary`.

### <a name="transactional-replicator-performance-counters"></a>Compteurs de performance du réplicateur transactionnel

Le runtime Reliable Services émet les événements suivants dans la catégorie `Service Fabric Transactional Replicator`

 Nom du compteur | Description |
| --- | --- |
| Opérations BeginTransaction/s | Nombre de nouvelles transactions d’écriture créées par seconde.|
| Opérations de transaction/s | Nombre d’opérations d’ajout/de mise à jour/de suppression effectuée sur des collections fiables par seconde.|
| Octets du journal vidé/s | Nombre d'octets vidés sur le disque chaque seconde par le réplicateur transactionnel |
| Opérations limitées/s | Nombre d'opérations rejetées chaque seconde par le réplicateur transactionnel en raison de la limitation. |
| Avg. Transaction en ms/validation | Latence de validation moyenne par transaction en millisecondes |
| Avg. Latence de vidage (ms) | Durée moyenne des opérations de vidage disque initiées par le réplicateur transactionnel en millisecondes |

### <a name="tstore-performance-counters"></a>Compteurs de performances TStore

Le runtime Reliable Services émet les événements suivants dans la catégorie `Service Fabric TStore`

 Nom du compteur | Description |
| --- | --- |
| Nombre d’éléments | Nombre d’éléments dans le magasin.|
| Taille du disque | Taille totale de disque, en octets, des fichiers de point de contrôle du magasin.|
| Octets écrits/s dans le fichier de point de contrôle | Nombre d’octets écrits par seconde pour le fichier de point de contrôle le plus récent.|
| Octets disque transférés/s sur la copie | Nombre d’octets disque lus (sur le réplica principal) ou écrits (sur un réplica secondaire) par seconde lors d’une copie de magasin.|

## <a name="next-steps"></a>Étapes suivantes
[Fournisseurs EventSource dans PerfView](/archive/blogs/vancem/introduction-tutorial-logging-etw-events-in-c-system-diagnostics-tracing-eventsource)
