---
title: API du processeur de flux de modification .NET Azure Cosmos DB
description: Découvrez l’API et le kit SDK du processeur de flux de modification, notamment les dates de lancement, les dates de suppression et les modifications apportées entre chaque version du kit SDK du processeur de flux de modification .NET.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.openlocfilehash: ff861b5a58de4f108e49e52e6f09c71d7dd678ac
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097190"
---
# <a name="net-change-feed-processor-sdk-download-and-release-notes"></a>Kit de développement logiciel (SDK) du processeur de flux de modification .NET Téléchargement et notes de publication
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
>
> * [Kit de développement logiciel (SDK) .NET v3](sql-api-sdk-dotnet-standard.md)
> * [SDK .NET v2](sql-api-sdk-dotnet.md)
> * [SDK .NET Core v2](sql-api-sdk-dotnet-core.md)
> * [SDK .NET Change Feed v2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.JS](sql-api-sdk-node.md)
> * [SDK Java v4](sql-api-sdk-java-v4.md)
> * [SDK Java Async v2](sql-api-sdk-async-java.md)
> * [SDK Java Sync v2](sql-api-sdk-java.md)
> * [Spring Data v2](sql-api-sdk-java-spring-v2.md)
> * [Spring Data v3](sql-api-sdk-java-spring-v3.md)
> * [Spark Connector](sql-api-sdk-java-spark.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api
> * [Fournisseur de ressources REST](/rest/api
> * [SQL](sql-api-query-reference.md)
> * [Exécuteur en bloc – .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Exécuteur en bloc – Java](sql-api-sdk-bulk-executor-java.md)

|   |   |
|---|---|
|**Téléchargement du Kit de développement logiciel (SDK)**|[NuGet](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB.ChangeFeedProcessor/)|
|**Documentation de l’API**|[Documentation de référence de l’API de la bibliothèque du processeur de flux de modification](/dotnet/api/microsoft.azure.documents.changefeedprocessor?view=azure-dotnet&preserve-view=true)|
|**Prise en main**|[Prise en main du kit SDK du processeur de flux de modification .NET](change-feed.md)|
|**Infrastructure actuellement prise en charge**| [Microsoft .NET Framework 4.5](https://www.microsoft.com/download/details.aspx?id=30653)</br> [Microsoft .NET Core](https://www.microsoft.com/net/download/core) |

> [!NOTE]
> Si vous utilisez un processeur de flux de modification, consultez la version 3.x la plus récente du [Kit de développement logiciel (SDK) .NET](change-feed-processor.md), qui a intégré le flux de modification au Kit de développement logiciel (SDK). 

## <a name="release-notes"></a>Notes de publication

### <a name="v2-builds"></a>Builds V2

### <a name="232"></a><a id="2.3.2"></a>2.3.2
* Ajout de la compatibilité du magasin de baux avec le SDK V3 qui active les chemins de migration à chaud. Une application peut migrer vers le kit SDK V3 et migrer à nouveau vers la bibliothèque du processeur Modifier le flux sans perdre aucun état.

### <a name="231"></a><a id="2.3.1"></a>2.3.1
* Correction du cas où le motif de clôture `FeedProcessing.ChangeFeedObserverCloseReason.Unknown` était envoyé à `FeedProcessing.IChangeFeedObserver.CloseAsync` si la partition était introuvable ou si le réplica cible n’était pas à jour avec la session de lecture. Désormais, les motifs de clôture `FeedProcessing.ChangeFeedObserverCloseReason.ResourceGone` et `FeedProcessing.ChangeFeedObserverCloseReason.ReadSessionNotAvailable` sont utilisées dans ces cas-là.
* Ajout du nouveau motif de clôture `FeedProcessing.ChangeFeedObserverCloseReason.ReadSessionNotAvailable` envoyé pour fermer l’observateur de flux de modification lorsque le réplica cible n’est pas à jour avec la session de lecture.

### <a name="230"></a><a id="2.3.0"></a>2.3.0
* Ajout d’une nouvelle méthode `ChangeFeedProcessorBuilder.WithCheckpointPartitionProcessorFactory` et de l’interface publique correspondante `ICheckpointPartitionProcessorFactory`. Cela permet à une implémentation de l’interface `IPartitionProcessor` d’utiliser un mécanisme de point de contrôle intégré. La nouvelle fabrique est semblable à l’`IPartitionProcessorFactory` existante, sauf que sa méthode `Create` prend également le paramètre `ILeaseCheckpointer`.
* Une seule des deux méthodes, `ChangeFeedProcessorBuilder.WithPartitionProcessorFactory` ou `ChangeFeedProcessorBuilder.WithCheckpointPartitionProcessorFactory`, peut être utilisée pour la même instance de `ChangeFeedProcessorBuilder`.

### <a name="228"></a><a id="2.2.8"></a>2.2.8
* Amélioration de la stabilité et de la capacité de diagnostic :
  * Ajout de la possibilité de détecter lorsque la lecture du flux de modification prend beaucoup de temps. Lorsqu’elle est plus longue que la valeur spécifiée par la propriété `ChangeFeedProcessorOptions.ChangeFeedTimeout`, les actions suivantes sont effectuées :
    * L’opération de lecture du flux de modification sur la partition problématique est abandonnée.
    * L’instance du processeur de flux de modification abandonne la propriété du bail problématique. Le bail supprimé sera récupéré au cours de la prochaine étape d’acquisition de bail effectuée ou non par la même instance de processeur de flux de modification. Ainsi, la lecture du flux de modification recommencera.
    * Un problème est signalé au moniteur d’intégrité. Le moniteur d’intégrité par défaut envoie tous les problèmes signalés au journal des traces.
  * Ajout d’une nouvelle propriété publique : `ChangeFeedProcessorOptions.ChangeFeedTimeout`. La valeur par défaut de cette propriété est de 10 min.
  * Ajout d’une nouvelle valeur enum publique : `Monitoring.MonitoredOperation.ReadChangeFeed`. Lorsque la valeur de `HealthMonitoringRecord.Operation` est définie sur `Monitoring.MonitoredOperation.ReadChangeFeed`, cela indique que le problème d’intégrité est lié à la lecture du flux de modification.

### <a name="227"></a><a id="2.2.7"></a>2.2.7
* La stratégie d’équilibrage de charge améliorée pour le scénario lors de l’obtention de tous les baux prend plus de temps que l’intervalle d’expiration de bail, par exemple, en raison de problèmes réseau :
  * Dans ce scénario, l’algorithme d’équilibrage de charge considérait de façon erronée que les baux avaient expiré, ce qui entraînait un vol de baux des propriétaires actifs. Cela pouvait déclencher un rééquilibrage inutile d’un grand nombre de baux.
  * Ce problème est résolu dans cette version, en évitant les nouvelles tentatives en cas de conflit lors de l’acquisition d’un bail expiré que le propriétaire n’a pas changé et le report de l’acquisition du bail expiré à l’itération d’équilibrage de charge suivante.

### <a name="226"></a><a id="2.2.6"></a>2.2.6
* Gestion améliorée des exceptions de l’Observateur.
* Informations plus détaillées sur les erreurs de l’Observateur :
  * Lorsqu’un Observateur est fermé en raison d'une exception levée par ProcessChangesAsync, CloseAsync reçoit alors le paramètre du motif qui est défini sur ChangeFeedObserverCloseReason.ObserverError.
  * Ajout de traces pour identifier les erreurs dans le code utilisateur d’un Observateur.

### <a name="225"></a><a id="2.2.5"></a>2.2.5
* Ajout de la prise en charge de la gestion des collections fractionnées qui utilisent un débit de base de données partagé.
  * Cette version corrige un problème qui peut se produire dans les collections fractionnées utilisant un débit de base de données partagé lorsque le fractionnement entraîne un rééquilibrage des partitions avec la création d’une seule plage de clés de partition enfants au lieu de deux. Dans ce cas, le processeur de flux de modification peut se bloquer lors de la suppression du bail pour l’ancienne plage de clés de partition et ne pas créer de nouveaux baux. Ce problème a été résolu dans cette version.

### <a name="224"></a><a id="2.2.4"></a>2.2.4
* Ajout de la nouvelle propriété ChangeFeedProcessorOptions.StartContinuation pour prendre en charge le démarrage du flux de modification à partir d’un jeton de continuation de requête. Elle est utilisée uniquement quand la collection de baux est vide ou quand aucune propriété ContinuationToken n’est définie pour un bail. Pour les baux de la collection dont la propriété ContinuationToken est définie, ContinuationToken est utilisée et ChangeFeedProcessorOptions.StartContinuation est ignorée.

### <a name="223"></a><a id="2.2.3"></a>2.2.3
* Ajout de la prise en charge de l’utilisation d’un magasin personnalisé afin de rendre persistants les jetons de continuation par partition.
  * Par exemple, vous pouvez utiliser un magasin de baux personnalisé pour persister une collection de baux Azure Cosmos DB partitionnée de n’importe quelle manière personnalisée.
  * Les magasins de baux personnalisés peuvent utiliser le nouveau point d’extensibilité ChangeFeedProcessorBuilder.WithLeaseStoreManager(ILeaseStoreManager) et la nouvelle interface publique ILeaseStoreManager.
  * Refactorisation de l’interface ILeaseManager en plusieurs interfaces de rôle.
* Changement cassant mineur : suppression du point d’extensibilité ChangeFeedProcessorBuilder.WithLeaseManager(ILeaseManager). Utilisez ChangeFeedProcessorBuilder.WithLeaseStoreManager(ILeaseStoreManager) à la place.

### <a name="222"></a><a id="2.2.2"></a>2.2.2
* Cette version corrige un problème qui se produit durant le traitement d’un fractionnement dans la collection supervisée et durant l’utilisation d’une collection de baux partitionnée. Lors du traitement d’un bail pour une partition fractionnée, le bail correspondant à cette partition ne peut pas être supprimé. Ce problème a été résolu dans cette version.

### <a name="221"></a><a id="2.2.1"></a>2.2.1
* Calcul de l’estimateur fixe pour les comptes avec plusieurs régions d’écriture et le nouveau format de jeton de session.

### <a name="220"></a><a id="2.2.0"></a>2.2.0
* Ajout de la prise en charge des collections de baux partitionnées. La clé de partition doit être définie comme /id.
* Changement cassant mineur : les méthodes de l’interface IChangeFeedDocumentClient et la classe ChangeFeedDocumentClient ont été modifiées pour inclure les paramètres RequestOptions et CancellationToken. IChangeFeedDocumentClient est un point d’extensibilité avancée qui vous permet d’offrir une implémentation personnalisée du client du document à utiliser le processeur de flux de modification, par exemple, décorer DocumentClient et intercepter tous les appels correspondants pour renforcer le suivi, la gestion des erreurs, etc. Avec cette mise à jour, le code qui implémente IChangeFeedDocumentClient devra être modifié pour inclure les nouveaux paramètres dans l’implémentation.
* Améliorations mineures des diagnostics.

### <a name="210"></a><a id="2.1.0"></a>2.1.0
* Ajout de la nouvelle API, Task&lt;IReadOnlyList&lt;RemainingPartitionWork&gt;&gt; IRemainingWorkEstimator.GetEstimatedRemainingWorkPerPartitionAsync(). Elle peut être utilisée pour estimer un travail pour chaque partition.
* Prend en charge le kit de développement logiciel (SDK) Microsoft.Azure.DocumentDB 2.0. Nécessite Microsoft.Azure.DocumentDB 2.0 ou version ultérieure.

### <a name="206"></a><a id="2.0.6"></a>2.0.6
* Ajout de la propriété publique ChangeFeedEventHost.HostName pour assurer la compatibilité avec la version 1.

### <a name="205"></a><a id="2.0.5"></a>2.0.5
* Condition de concurrence qui se produit pendant un fractionnement de partition. La condition de concurrence peut conduire à l’acquisition d’un bail et à sa perte immédiate pendant le fractionnement de partition, entraînant une contention. Le problème de condition de concurrence est résolu avec cette version.

### <a name="204"></a><a id="2.0.4"></a>2.0.4
* Kit SDK GA

### <a name="203-prerelease"></a><a id="2.0.3-prerelease"></a>2.0.3-prerelease
* Les problèmes suivants ont été résolus :
  * En cas de fractionnement de la partition, un double traitement des documents modifiés avant le fractionnement peut se produire.
  * L’API GetEstimatedRemainingWork a retourné 0 quand aucun bail n’était présent dans la collection de baux.

* Les exceptions suivantes sont rendues publiques. Les extensions qui implémentent IPartitionProcessor peuvent lever ces exceptions.
  * Microsoft.Azure.Documents.ChangeFeedProcessor.Exceptions.LeaseLostException. 
  * Microsoft.Azure.Documents.ChangeFeedProcessor.Exceptions.PartitionException. 
  * Microsoft.Azure.Documents.ChangeFeedProcessor.Exceptions.PartitionNotFoundException.
  * Microsoft.Azure.Documents.ChangeFeedProcessor.Exceptions.PartitionSplitException. 

### <a name="202-prerelease"></a><a id="2.0.2-prerelease"></a>2.0.2-prerelease
* Évolutions mineures de l’API :
  * Suppression de ChangeFeedProcessorOptions.IsAutoCheckpointEnabled qui a été marqué comme étant obsolète.

### <a name="201-prerelease"></a><a id="2.0.1-prerelease"></a>2.0.1-prerelease
* Améliorations sur le plan de la stabilité :
  * Meilleure gestion de l’initialisation du magasin de baux. Quand le magasin de baux est vide, une seule instance de processeur peut l’initialiser. Pendant ce temps, les autres attendent.
  * Renouvellement/libération de bail plus stable/efficace. Le renouvellement et la libération d’un bail au niveau d’une partition sont indépendants du renouvellement des autres. Dans v1, il s’agissait d’une opération séquentielle qui portait sur toutes les partitions.
* Nouvelle API v2 :
  * Modèle de générateur pour la construction flexible du processeur : la classe ChangeFeedProcessorBuilder.
    * Peut prendre n’importe quelle combinaison de paramètres.
    * Peut prendre une instance DocumentClient pour la surveillance et/ou la collection de baux (non disponible dans v1).
  * IChangeFeedObserver.ProcessChangesAsync accepte désormais CancellationToken.
  * IRemainingWorkEstimator : l’estimateur de travail restant peut être utilisé indépendamment du processeur.
  * Nouveaux points d’extensibilité :
    * IPartitionLoadBalancingStrategy : pour un équilibrage de charge de partitions personnalisé entre des instances du processeur.
    * ILease, ILeaseManager : pour la gestion de baux personnalisée.
    * IPartitionProcessor : pour les modifications de traitement personnalisées sur une partition.
* Journalisation : utilise la bibliothèque [LibLog](https://github.com/damianh/LibLog).
* 100 % compatible avec l’API v1.
* Nouvelle base de code.
* Compatible avec les versions 1.21.1 et ultérieures du [SDK SQL .NET](sql-api-sdk-dotnet.md).

### <a name="v1-builds"></a>Builds V1

### <a name="133"></a><a id="1.3.3"></a>1.3.3
* Journalisation supplémentaire ajoutée.
* Réparation d’une fuite DocumentClient lors de l’appel du travail en attente plusieurs fois.

### <a name="132"></a><a id="1.3.2"></a>1.3.2
* Correctifs inclus dans l’estimation de travail en attente.

### <a name="131"></a><a id="1.3.1"></a>1.3.1
* Améliorations de la stabilité.
  * Correctif pour résoudre le problème lié aux tâches annulées susceptibles de provoquer l’arrêt des observateurs dans certaines partitions.
* Prise en charge des points de contrôle manuels.
* Compatible avec les versions 1.21 et version ultérieure du [Kit de développement logiciel (SDK) SQL .NET](sql-api-sdk-dotnet.md).

### <a name="120"></a><a id="1.2.0"></a>1.2.0
* Ajout de la prise en charge de .NET Standard 2.0. Le package prend désormais en charge les monikers d’infrastructure `netstandard2.0` et `net451`.
* Compatible avec les versions 1.17.0 et supérieures du [Kit de développement logiciel (SDK) SQL .NET](sql-api-sdk-dotnet.md).
* Compatible avec les versions 1.5.1 et supérieures du [Kit de développement logiciel (SDK) principal SQL .NET](sql-api-sdk-dotnet-core.md).

### <a name="111"></a><a id="1.1.1"></a>1.1.1
* Résout un problème relatif au calcul de l’estimation du travail restant lorsque le flux de modification était vide ou qu’aucun travail n’était en attente.
* Compatible avec les versions 1.13.2 et supérieures du [Kit de développement logiciel (SDK) SQL .NET](sql-api-sdk-dotnet.md).

### <a name="110"></a><a id="1.1.0"></a>1.1.0
* Ajout d’une méthode pour obtenir une estimation du travail restant à traiter dans le flux de modification.
* Compatible avec les versions 1.13.2 et supérieures du [Kit de développement logiciel (SDK) SQL .NET](sql-api-sdk-dotnet.md).

### <a name="100"></a><a id="1.0.0"></a>1.0.0
* Kit SDK GA
* Compatible avec les versions 1.14.1 et inférieures du [Kit de développement logiciel (SDK) SQL .NET](sql-api-sdk-dotnet.md).

## <a name="release--retirement-dates"></a>Dates de lancement et de suppression

Microsoft fournira une notification au moins **12 mois** avant le retrait d’un Kit de développement logiciel (SDK) pour faciliter la transition vers une version plus récente/prise en charge. Les nouvelles fonctionnalités et fonctions, et les optimisations sont uniquement ajoutées au Kit SDK actuel. Par conséquent, il est recommandé de toujours passer à la dernière version du SDK dès que possible.

> [!WARNING]
> Après le 31 août 2022, Azure Cosmos DB n’apportera plus de correctifs de bogues, n’ajoutera plus de nouvelles fonctionnalités et ne fournira plus de support aux versions 1.x du Kit SDK .NET Core ou .NET Azure Cosmos DB pour l’API SQL. Si vous préférez ne pas effectuer la mise à niveau, les requêtes envoyées depuis la version 1.x du Kit de développement logiciel (SDK) continueront à être traitées par le service Azure Cosmos DB.

<br/>

| Version | Date de sortie | Date de suppression |
| --- | --- | --- |
| [2.3.2](#2.3.2) |11 août 2020 |--- |
| [2.3.1](#2.3.1) |30 juillet 2020 |--- |
| [2.3.0](#2.3.0) |2 avril 2020 |--- |
| [2.2.8](#2.2.8) |28 octobre 2019 |--- |
| [2.2.7](#2.2.7) |14 mai 2019 |--- |
| [2.2.6](#2.2.6) |29 janvier 2019 |--- |
| [2.2.5](#2.2.5) |13 décembre 2018 |--- |
| [2.2.4](#2.2.4) |29 novembre 2018 |--- |
| [2.2.3](#2.2.3) |19 novembre 2018 |--- |
| [2.2.2](#2.2.2) |31 octobre 2018 |--- |
| [2.2.1](#2.2.1) |24 octobre 2018 |--- |
| [1.3.3](#1.3.3) |8 mai 2018 |--- |
| [1.3.2](#1.3.2) |18 avril 2018 |--- |
| [1.3.1](#1.3.1) |13 mars 2018 |--- |
| [1.2.0](#1.2.0) |31 octobre 2017 |--- |
| [1.1.1](#1.1.1) |29 août 2017 |--- |
| [1.1.0](#1.1.0) |13 août 2017 |--- |
| [1.0.0](#1.0.0) |7 juillet 2017 |--- |

## <a name="faq"></a>Questions fréquentes (FAQ)

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>Voir aussi

Pour en savoir plus sur Cosmos DB, consultez la page du service [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).
