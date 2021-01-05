---
title: 'Testabilité : Communication de service'
description: La communication service à service constitue un point d’intégration critique d’une application Service Fabric. Cet article aborde les problématiques de conception et les techniques de test.
ms.topic: conceptual
ms.date: 11/02/2017
ms.openlocfilehash: fbf889422d69d031d777acd80cf3becfb3184568
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575922"
---
# <a name="service-fabric-testability-scenarios-service-communication"></a>Scénarios de testabilité Service Fabric Communication de service
Les microservices et les styles architecturaux orientés services émergent naturellement dans Azure Service Fabric. Dans ces types d’architectures distribuées, les applications de microservices compartimentés sont généralement composées de plusieurs services qui interagissent entre eux. Même dans le cas le plus simple, vous disposez habituellement d’au moins un service web sans état et d’un service de stockage de données avec état qui communiquent.

La communication de service à service constitue un point d’intégration critique d’une application, car chaque service expose une API distante aux autres services. L’utilisation d’un ensemble de limites d’API impliquant un trafic d’E/S nécessite bien souvent une attention particulière, secondée par de solides méthodes de test et de validation.

Lorsque ces limites de services sont liées au sein d’un système distribué, de nombreux aspects sont à prendre en compte :

* *Protocole de transfert*. Allez-vous utiliser le protocole HTTP, pour une interopérabilité améliorée, ou un protocole binaire personnalisé favorisant un débit optimal ?
* *Gestion des erreurs*. Comment les erreurs permanentes et temporaires sont-elles traitées ? Que se passe-t-il quand un service se déplace vers un autre nœud ?
* *Délais d’attente et latence*. Dans les applications multiniveaux, comment chaque couche de service traite-t-elle la latence de bout en bout, au sein de la pile jusqu’à l’utilisateur ?

Vous utilisez l’un des composants de communication de service intégrés de Service Fabric ? Vous développez le vôtre ? Pour garantir la résilience de votre application, il est primordial de tester l’interaction entre vos services.

## <a name="prepare-for-services-to-move"></a>Préparation des services à déplacer
Les instances de service peuvent se déplacer au fil du temps. Ce déplacement est particulièrement avéré quand elles sont configurées avec des mesures de charge dédiées à l’équilibrage personnalisé optimal des ressources. Service Fabric déplace vos instances de service à des fins de disponibilité maximale, même pendant les mises à niveau, les basculements, les augmentations de la taille des instances et d’autres situations ponctuant le cycle de vie d’un système distribué.

Quand les services se déplacent dans le cluster, vos clients et les autres services doivent envisager deux scénarios d’interaction avec ces derniers :

* L’instance de service ou le réplica de partition ont subi un déplacement depuis votre dernière interaction. Cette configuration, qui fait partie du cycle de vie normal du service, se produit naturellement au cours de la vie d’une application.
* L’instance de service ou le réplica de partition est en cours de déplacement. Le basculement d’un service entre deux nœuds se produit très rapidement dans Service Fabric, mais vous pouvez constater un délai de mise à disposition en cas de démarrage lent du composant de communication de votre service.

Pour bénéficier d’un système pleinement fonctionnel, il est nécessaire de gérer ces scénarios de manière appropriée. Pour cela, n’oubliez pas que :

* Chaque service pouvant faire l’objet d’une connexion présente une *adresse* d’écoute (par exemple, HTTP ou WebSockets). Quand une instance ou partition de service se déplace, le point de terminaison de son adresse change. (Elle se déplace vers un autre nœud dont l’adresse IP est différente.) Si vous utilisez des composants de communication intégrés, ils traitent pour vous la nouvelle résolution des adresses de service.
* Vous pourrez observer une augmentation temporaire de la latence du service étant donné que l’instance de service redémarre son écouteur. Cette latence dépend de la vitesse à laquelle le service ouvre l’écouteur une fois l’instance de service déplacée.
* Toutes les connexions existantes doivent être fermées, puis rouvertes une fois que le service s’ouvre sur un nouveau nœud. Un arrêt ou redémarrage approprié du nœud laisse suffisamment de temps aux connexions existantes pour s’arrêter correctement.

### <a name="test-it-move-service-instances"></a>Test : Déplacement des instances de service
À l’aide des outils de testabilité de Service Fabric, vous pouvez établir un scénario test afin d’évaluer ces situations dans des contextes différents :

1. Déplacez un réplica principal de service avec état.
   
    Le réplica principal d’une partition de service avec état peut être déplacé pour diverses raisons. Appliquez ce scénario pour cibler le réplica principal d’une partition spécifique et examiner la réaction de vos services à ce déplacement, dans un cadre très strict.
   
    ```powershell
   
    PS > Move-ServiceFabricPrimaryReplica -PartitionId 6faa4ffa-521a-44e9-8351-dfca0f7e0466 -ServiceName fabric:/MyApplication/MyService
   
    ```
2. Arrêtez un nœud.
   
    Quand un nœud est arrêté, Service Fabric déplace l’ensemble des instances ou partitions de service positionnées sur ce nœud vers l’un des autres nœuds disponibles dans le cluster. Utilisez ce scénario pour tester une configuration où un nœud de votre cluster est perdu et l’ensemble de vos instances et réplicas de service sur ce nœud doivent être déplacés.
   
    Vous pouvez arrêter un nœud à l’aide de l’applet de commande PowerShell **Stop-ServiceFabricNode** :
   
    ```powershell
   
    PS > Stop-ServiceFabricNode -NodeName Node_1
   
    ```

## <a name="maintain-service-availability"></a>Maintenir la disponibilité du service
En tant que plateforme, Service Fabric est conçu pour assurer la haute disponibilité de vos services. Mais dans des cas extrêmes, les problèmes liés à l’infrastructure sous-jacente peuvent quand même entraîner une indisponibilité. Il est important de tester ces scénarios également.

Les services avec état utilisent un système avec quorum pour répliquer l’état à des fins de haute disponibilité. Cela signifie qu’un quorum de réplicas doit être disponible pour l’exécution des opérations d’écriture. Dans de rares cas, comme celui d’une défaillance matérielle étendue, aucun quorum de réplicas ne peut être disponible. Le cas échéant, vous ne pourrez pas exécuter d’opérations d’écriture, mais disposerez des opérations de lecture.

### <a name="test-it-write-operation-unavailability"></a>Test : Écriture de la non-disponibilité des opérations
En utilisant les outils de testabilité de Service Fabric, vous pouvez injecter une erreur qui entraîne une perte de quorum en guise de test. Même si ce scénario est rare, les clients et les entités qui dépendent d’un service avec état doivent néanmoins s’y préparer. Que faire quand les requêtes d’écriture sur le service avec état sont impossibles ? Il est également essentiel que le service avec état ait conscience de cette possibilité, et qu’il en fasse part de manière adaptée aux appelants.

Vous pouvez provoquer une perte de quorum à l’aide de l’applet de commande PowerShell **Invoke-ServiceFabricPartitionQuorumLoss** :

```powershell

PS > Invoke-ServiceFabricPartitionQuorumLoss -ServiceName fabric:/Myapplication/MyService -QuorumLossMode QuorumReplicas -QuorumLossDurationInSeconds 20

```

Dans cet exemple, nous avons défini `QuorumLossMode` sur `QuorumReplicas` pour indiquer que nous voulons provoquer une perte de quorum sans retirer tous les réplicas. Ainsi, les opérations de lecture sont toujours possibles. Pour tester ce scénario avec l’intégralité de la partition indisponible, définissez ce commutateur sur `AllReplicas`.

## <a name="next-steps"></a>Étapes suivantes
[En savoir plus sur les actions de testabilité](service-fabric-testability-actions.md)

[En savoir plus sur les scénarios de testabilité](service-fabric-testability-scenarios.md)

