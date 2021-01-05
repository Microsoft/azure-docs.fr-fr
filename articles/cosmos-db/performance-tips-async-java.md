---
title: Conseils sur les performances pour le Kit de développement logiciel (SDK) Java asynchrone v2 Azure Cosmos DB
description: Découvrez les options de configuration clientes disponibles afin d’améliorer les performances de la base de données Azure Cosmos pour le Kit de développement logiciel (SDK) Java asynchrone v2
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: how-to
ms.date: 05/11/2020
ms.author: anfeldma
ms.custom: devx-track-java, contperf-fy21q2
ms.openlocfilehash: bd009ae4909c8cb016a31323294df3a359eb7c51
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033661"
---
# <a name="performance-tips-for-azure-cosmos-db-async-java-sdk-v2"></a>Conseils sur les performances pour le Kit de développement logiciel (SDK) Java asynchrone v2 Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Kit de développement logiciel (SDK) Java v4](performance-tips-java-sdk-v4-sql.md)
> * [SDK Java Async v2](performance-tips-async-java.md)
> * [SDK Java Sync v2](performance-tips-java.md)
> * [Kit de développement logiciel (SDK) .NET v3](performance-tips-dotnet-sdk-v3-sql.md)
> * [Kit SDK .NET v2](performance-tips.md)


> [!IMPORTANT]  
> Il ne s’agit *pas* du Kit de développement logiciel (SDK) Java pour Azure Cosmos DB le plus récent. Vous devez mettre à niveau votre projet vers le [Kit de développement logiciel (SDK) Java v4 Azure Cosmos DB](sql-api-sdk-java-v4.md), puis lire le [guide sur les performances](performance-tips-java-sdk-v4-sql.md) du Kit de développement logiciel (SDK) Java v4 Azure Cosmos DB. Pour la mise à niveau, suivez les instructions fournies dans les guides [Migrer vers le Kit de développement logiciel (SDK) Java v4 Azure Cosmos DB](migrate-java-v4-sdk.md) et [Reactor vs RxJava](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/reactor-rxjava-guide.md). 
> 
> Ces conseils en matière de performances concernent uniquement le Kit de développement logiciel (SDK) Java asynchrone v2 Azure Cosmos DB. Pour plus d’informations, consultez les [Notes de publication](sql-api-sdk-async-java.md), le [Référentiel Maven](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb) du Kit de développement logiciel (SDK) Java asynchrone v2 Azure Cosmos DB et le [guide de résolution des problèmes](troubleshoot-java-async-sdk.md) du Kit de développement logiciel (SDK) Java asynchrone v2 Azure Cosmos DB.
>

Azure Cosmos DB est une base de données distribuée rapide et flexible qui peut être mise à l’échelle en toute transparence avec une latence et un débit garantis. Vous n’avez pas à apporter de modifications d’architecture majeures ou écrire de code complexe pour mettre à l’échelle votre base de données avec Azure Cosmos DB. La réduction et l’augmentation de l’échelle est aussi simple que le passage d’un appel d’API ou de Kit de développement logiciel (SDK). Toutefois, étant donné qu’Azure Cosmos DB est accessible via des appels réseau, vous pouvez apporter des optimisations côté client de manière à atteindre des performances de pointe quand vous utilisez le [Kit de développement logiciel (SDK) Java synchrone v2 Azure Cosmos DB](sql-api-sdk-async-java.md).

Si vous vous demandez comment améliorer les performances de votre base de données, lisez ce qui suit :

## <a name="networking"></a>Mise en réseau

* **Mode de connexion : Utiliser le mode direct**
    
  La façon dont un client se connecte à Azure Cosmos DB a des conséquences importantes sur les performances, notamment en termes de latence côté client. *ConnectionMode* est un paramètre de configuration clé disponible pour la configuration de la *ConnectionPolicy* du client. Pour le Kit de développement logiciel (SDK) Java asynchrone Azure Cosmos DB v2, les deux modes de connexion disponibles sont :  
      
  * [Passerelle (par défaut)](/java/api/com.microsoft.azure.cosmosdb.connectionmode)  
  * [Direct](/java/api/com.microsoft.azure.cosmosdb.connectionmode)
  
  Le mode Passerelle est pris en charge sur toutes les plateformes du SDK et c’est l’option configurée par défaut. Si vos applications s’exécutent dans un réseau d’entreprise comportant des restrictions de pare-feu strictes, le mode Passerelle est idéal, car il utilise le port HTTPS standard et un seul point de terminaison.   Cela implique toutefois un compromis en matière de performances : ce mode fait intervenir un tronçon réseau supplémentaire chaque fois que des données sont lues ou écrites dans Azure Cosmos DB. Pour cette raison, le mode Direct offre de meilleures performances grâce à un moins grand nombre de tronçons réseau.
  
  *ConnectionMode* est configuré pendant la construction de l’instance *DocumentClient* avec le paramètre *ConnectionPolicy*.

### <a name="async-java-sdk-v2-maven-commicrosoftazureazure-cosmosdb"></a><a id="asyncjava2-connectionpolicy"></a>Kit de développement logiciel (SDK) Java asynchrone v2 (Maven com.microsoft.azure::azure-cosmosdb)

```java
    public ConnectionPolicy getConnectionPolicy() {
        ConnectionPolicy policy = new ConnectionPolicy();
        policy.setConnectionMode(ConnectionMode.Direct);
        policy.setMaxPoolSize(1000);
        return policy;
    }

    ConnectionPolicy connectionPolicy = new ConnectionPolicy();
    DocumentClient client = new DocumentClient(HOST, MASTER_KEY, connectionPolicy, null);
```

* **Colocalisation des clients dans la même région Azure pour les performances**

  Dans la mesure du possible, placez toutes les applications appelant Azure Cosmos DB dans la même région que la base de données Azure Cosmos. Pour une comparaison approximative, les appels à Azure Cosmos DB dans la même région s’effectuent en 1 à 2 ms, mais la latence entre les côtes Ouest et Est des États-Unis est supérieure à 50 ms. Cette latence peut probablement varier d’une requête à l’autre, en fonction de l’itinéraire utilisé par la requête lorsqu’elle passe du client à la limite du centre de données Azure. Pour obtenir la latence la plus faible possible, l’application appelante doit être située dans la même région Azure que le point de terminaison Azure Cosmos DB configuré. Pour obtenir la liste des régions disponibles, voir [Régions Azure](https://azure.microsoft.com/regions/#services).

  :::image type="content" source="./media/performance-tips/same-region.png" alt-text="Illustration de la stratégie de connexion Azure Cosmos DB" border="false":::

## <a name="sdk-usage"></a>Utilisation du kit de développement logiciel (SDK)

* **Installation du kit de développement logiciel (SDK) le plus récent**

  Les SDK Azure Cosmos DB sont constamment améliorés pour fournir des performances optimales. Consultez les pages [Notes de publication](sql-api-sdk-async-java.md) du Kit de développement logiciel (SDK) Java asynchrone Azure Cosmos DB v2 pour déterminer quel est le SDK le plus récent et passer en revue les améliorations.

* **Utiliser un client Azure Cosmos DB singleton pour la durée de vie de votre application**

  Chaque instance AsyncDocumentClient est thread-safe et effectue une gestion des connexions efficace et une mise en cache d’adresses. Pour permettre une gestion des connexions efficace et améliorer les performances par AsyncDocumentClient, nous vous recommandons d’utiliser une seule instance de AsyncDocumentClient par AppDomain pour la durée de vie de l’application.

* **Paramètre ConnectionPolicy**

  Par défaut, les demandes Cosmos DB en mode direct sont effectuées sur TCP lors de l’utilisation du SDK Java asynchrone v2 Azure Cosmos DB. En interne, le SDK utilise une architecture spéciale en mode direct pour gérer dynamiquement les ressources réseau et obtenir les meilleures performances.

  Dans le SDK Java asynchrone v2 Azure Cosmos DB, le mode direct est le meilleur choix pour améliorer les performances des bases de données avec la plupart des charges de travail. 

  * ***Vue d’ensemble du mode direct** _

  :::image type="content" source="./media/performance-tips-async-java/rntbdtransportclient.png" alt-text="Illustration de l’architecture du mode direct" border="false":::
  
  L’architecture côté client utilisée en mode direct permet une utilisation prévisible du réseau et un accès multiplexé aux réplicas Azure Cosmos DB. Le diagramme ci-dessus montre comment le mode direct route les demandes des clients vers les réplicas dans le back-end Cosmos DB. L’architecture du mode direct alloue jusqu’à 10 _ *canaux** côté client par réplica de base de données. Un canal est une connexion TCP précédée d’une mémoire tampon des requêtes, d’une profondeur de 30 requêtes. Les canaux appartenant à un réplica sont alloués dynamiquement en fonction des besoins par le **point de terminaison de service** du réplica. Quand l’utilisateur émet une requête en mode direct, **TransportClient** l’achemine vers le point de terminaison de service approprié en fonction de la clé de partition. La **file d’attente des requêtes** met en mémoire tampon les requêtes avant le point de terminaison de service.

  * ***Options de configuration de ConnectionPolicy pour le mode direct** _

    Pour commencer, utilisez les paramètres de configuration recommandés ci-dessous. Si vous rencontrez des problèmes sur ce point particulier, contactez l’[équipe Azure Cosmos DB](mailto:CosmosDBPerformanceSupport@service.microsoft.com).

    Si vous utilisez Azure Cosmos DB comme base de données de référence (c’est-à-dire si la base de données est utilisée pour de nombreuses opérations de lecture de point et quelques opérations d’écriture), il peut être acceptable de définir _idleEndpointTimeout* sur 0 (c’est-à-dire pas de délai d’expiration).


    | Option de configuration       | Default    |
    | :------------------:       | :-----:    |
    | bufferPageSize             | 8 192       |
    | connectionTimeout          | « PT1M »     |
    | idleChannelTimeout         | "PT0S"     |
    | idleEndpointTimeout        | "PT1M10S"  |
    | maxBufferCapacity          | 8388608    |
    | maxChannelsPerEndpoint     | 10         |
    | maxRequestsPerChannel      | 30         |
    | receiveHangDetectionTime   | "PT1M5S"   |
    | requestExpiryInterval      | "PT5S"     |
    | requestTimeout             | « PT1M »     |
    | requestTimerResolution     | "PT0.5S"   |
    | sendHangDetectionTime      | "PT10S"    |
    | shutdownTimeout            | "PT15S"    |

* ***Conseils de programmation pour le mode direct** _

  Consultez l’article de référence [Résolution des problèmes](troubleshoot-java-async-sdk.md) du kit de développement logiciel (SDK) Azure Cosmos DB Async Java v2 pour résoudre tous les problèmes liés au kit SDK.
  
  Voici quelques conseils de programmation importants lors de l’utilisation du mode direct :
  
  _ **Utilisez le multithreading dans votre application pour bénéficier d’un transfert de données TCP efficace** : après avoir effectué une requête, votre application doit s’abonner pour recevoir les données sur un autre thread. Sinon, une opération « semi-duplex » imprévue est effectuée et les requêtes suivantes sont bloquées en attendant la réponse de la requête précédente.
  
  * **Exécutez les charges de travail nécessitant beaucoup de ressources système sur un thread dédié** : pour des raisons similaires à celles du conseil précédent, il est conseillé de placer les opérations comme le traitement de données complexes dans un thread distinct. Une requête qui extrait des données d’un autre magasin de données (par exemple, si le thread utilise simultanément des magasins de données Azure Cosmos DB et Spark) risque de subir une plus grande latence ; nous vous recommandons de créer un thread supplémentaire qui attend une réponse de l’autre magasin de données.
  
    * Les E/S du réseau sous-jacent dans le kit SDK Azure Cosmos DB Async Java v2 sont gérées par Netty ; consultez ces [conseils pour éviter les modèles de codage qui bloquent les threads d’E/S de Netty](troubleshoot-java-async-sdk.md#invalid-coding-pattern-blocking-netty-io-thread).
  
  * **Modélisation des données** - Le contrat SLA d’Azure Cosmos DB suppose que la taille des documents est inférieure à 1 Ko. L’optimisation de votre modèle de données et le recours à une programmation qui favorise les petites tailles de documents entraînent généralement une diminution de la latence. Si vous avez besoin de stocker et de récupérer des documents d’une taille supérieure à 1 Ko, nous vous recommandons de lier les documents à des données dans le Stockage Blob Azure.

* **Paramétrage des requêtes parallèles pour les collections partitionnées**

  Le Kit de développement logiciel (SDK) Java asynchrone v2 Azure Cosmos DB prend en charge les requêtes parallèles, qui vous permettent d’interroger une collection partitionnée en parallèle. Pour plus d’informations, voir les [exemples de code](https://github.com/Azure/azure-cosmosdb-java/tree/master/examples/src/test/java/com/microsoft/azure/cosmosdb/rx/examples) concernant l’utilisation des kits SDK. Les requêtes parallèles sont conçues pour améliorer la latence des requêtes et le débit sur leur équivalent série.

  * ***Optimisation de setMaxDegreeOfParallelism\:** _
    
    Les requêtes parallèles interrogent plusieurs partitions en parallèle. Les données d’une collection partitionnée individuelle sont toutefois extraites en série dans le cadre de la requête. Utilisez donc le paramètre setMaxDegreeOfParallelism pour définir le nombre de partitions qui augmente les chances de résultats de la requête, sous réserve que toutes les autres conditions système restent inchangées. Si vous ne connaissez pas le nombre de partitions, vous pouvez utiliser le paramètre setMaxDegreeOfParallelism pour définir un nombre élevé, et le système sélectionne le minimum (nombre de partitions, entrée fournie par l’utilisateur) comme degré maximal de parallélisme.

    Il est important de noter que les requêtes parallèles produisent de meilleurs résultats si les données sont réparties de manière homogène entre toutes les partitions. Si la collection est partitionnée de telle façon que toutes les données retournées par une requête, ou une grande partie d’entre elles, sont concentrées sur quelques partitions (une partition dans le pire des cas), les performances de la requête sont altérées par ces partitions.

  _ ***Optimisation de setMaxBufferedItemCount\:** _
    
    Une requête parallèle est conçue pour pré-extraire les résultats pendant que le lot de résultats actuel est en cours de traitement par le client. La pré-extraction permet d’améliorer la latence globale d’une requête. setMaxBufferedItemCount limite le nombre de résultats pré-extraits. Définir le paramètre setMaxBufferedItemCount sur le nombre de résultats retournés attendu (ou un nombre plus élevé) permet à la requête d’optimiser la pré-extraction.

    La pré-extraction fonctionne de la même façon, quel que soit le paramètre MaxDegreeOfParallelism, et il existe une seule mémoire tampon pour les données de toutes les partitions.

_ **Implémentation de l’interruption à intervalles définis par getRetryAfterInMilliseconds**

  Lors du test de performances, vous devez augmenter la charge jusqu’à une limite d’un petit nombre de requêtes. En cas de limitation, l’application client doit s’interrompre pour l’intervalle de nouvelle tentative spécifié sur le serveur. Le respect de l’interruption garantit un temps d’attente minimal entre chaque tentative.

* **Effectuer un scale-out de votre charge de travail cliente**

  Si vous effectuez des tests à des niveaux de débit élevé (> 50 000 RU/s), l’application cliente peut devenir un goulet d’étranglement en raison du plafonnement sur l’utilisation du processeur ou du réseau. Si vous atteignez ce point, vous pouvez continuer à augmenter le compte Azure Cosmos DB en augmentant la taille des instances de vos applications clientes sur plusieurs serveurs.

* **Utilisation de l’adressage en fonction du nom**

  Pour éviter de récupérer les ID de ressource de toutes les ressources utilisées pour construire le lien, utilisez l’adressage en fonction du nom dans lequel les liens ont pour format `dbs/MyDatabaseId/colls/MyCollectionId/docs/MyDocumentId`, et non SelfLinks (\_self) dans lequel les liens ont pour format `dbs/<database_rid>/colls/<collection_rid>/docs/<document_rid>`. En outre, ces ressources étant recréés (éventuellement avec le même nom), leur mise en cache peut s’avérer superflue.

* **Réglage de la taille de la page des flux de lecture/requêtes pour de meilleures performances**

  Pendant une lecture groupée de documents à l’aide de la fonctionnalité de flux de lecture (par exemple, readDocuments) ou l’émission d’une requête SQL, les résultats sont retournés de façon segmentée si le jeu de résultats est trop grand. Par défaut, les résultats sont retournés dans des segments de 100 éléments ou de 1 Mo, selon la limite atteinte en premier.

  Afin de réduire le nombre de boucles réseau nécessaires pour récupérer tous les résultats applicables, vous pouvez augmenter la taille de la page à 1000 résultats à l’aide de l’en-tête de requête [x-ms-max-item-count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers). Si vous avez besoin d’afficher uniquement quelques résultats, (par exemple, si votre interface utilisateur ou API d’application retourne seulement 10 résultats à la fois), vous pouvez également réduire la taille de la page à 10 résultats, afin de baisser le débit consommé pour les lectures et requêtes.

  Vous pouvez également définir la taille de la page à l’aide de la méthode setMaxItemCount.

* **Utilisation du Scheduler approprié (éviter le vol de threads Netty E/S Eventloop)**

  Le Kit de développement logiciel (SDK) Java asynchrone v2 Azure Cosmos DB utilise [Netty](https://netty.io/) pour les E/S non bloquantes. Le Kit de développement logiciel (SDK) utilise un nombre fixe de threads d’E/S netty eventloop (autant de cœurs de processeur présents sur votre machine) pour l’exécution d’opérations d’E/S. L’Observable retourné par l’API émet le résultat sur l’un des threads netty d’eventloop d’E/S partagés. Il est donc important de ne pas bloquer les threads netty eventloop d’E/S partagés. Un travail intensif de l’UC ou le blocage de l’opération sur le thread netty eventloop d’E/S peut provoquer un interblocage ou réduire considérablement le débit du Kit de développement logiciel (SDK).

  Par exemple, le code suivant exécute un travail intensif de l’UC sur le thread netty eventloop d’E/S :

  **Kit SDK Java Async v2 (Maven com.microsoft.azure::azure-cosmosdb)**

  ```java
    Observable<ResourceResponse<Document>> createDocObs = asyncDocumentClient.createDocument(
      collectionLink, document, null, true);

    createDocObs.subscribe(
      resourceResponse -> {
        //this is executed on eventloop IO netty thread.
        //the eventloop thread is shared and is meant to return back quickly.
        //
        // DON'T do this on eventloop IO netty thread.
        veryCpuIntensiveWork();
      });
  ```

  Une fois le résultat reçu, si vous souhaitez effectuer un travail intensif de l’UC sur le résultat, vous devez éviter de le faire sur le thread netty eventloop d’E/S. Vous pouvez fournir à la place votre propre Scheduler pour fournir votre propre thread pour l’exécution de votre travail.

  **Kit SDK Java Async v2 (Maven com.microsoft.azure::azure-cosmosdb)**

  ```java
    import rx.schedulers;

    Observable<ResourceResponse<Document>> createDocObs = asyncDocumentClient.createDocument(
      collectionLink, document, null, true);

    createDocObs.subscribeOn(Schedulers.computation())
    subscribe(
      resourceResponse -> {
        // this is executed on threads provided by Scheduler.computation()
        // Schedulers.computation() should be used only when:
        //   1. The work is cpu intensive 
        //   2. You are not doing blocking IO, thread sleep, etc. in this thread against other resources.
        veryCpuIntensiveWork();
      });
  ```

  En fonction de votre type de travail, vous devez utiliser le Scheduler RxJava existant approprié pour votre travail. Lire ici [``Schedulers``](http://reactivex.io/RxJava/1.x/javadoc/rx/schedulers/Schedulers.html).

  Pour plus d’informations, consultez la [page GitHub](https://github.com/Azure/azure-cosmosdb-java) concernant le Kit de développement logiciel (SDK) Java asynchrone v2 Azure Cosmos DB.

* **Désactiver la journalisation de netty**

    La journalisation de la bibliothèque netty produit beaucoup d’informations et doit être désactivée (la suppression de la connexion dans la configuration peut être insuffisante) afin d’éviter des coûts de processeur supplémentaires. Si vous n’êtes pas en mode débogage, désactivez la journalisation de netty en même temps. Par conséquent, si vous utilisez log4j pour supprimer les coûts supplémentaires de l’UC induits par ``org.apache.log4j.Category.callAppenders()`` de netty, ajoutez la ligne suivante à votre codebase :

    ```java
    org.apache.log4j.Logger.getLogger("io.netty").setLevel(org.apache.log4j.Level.OFF);
    ```

 * **Limite des ressources des fichiers ouverts du système d’exploitation**
 
    Certains systèmes Linux (par exemple Red Hat) ont une limite maximale du nombre de fichiers ouverts et donc du nombre total de connexions. Exécutez la commande suivante pour afficher les limites actuelles :

    ```bash
    ulimit -a
    ```

    Le nombre de fichiers ouverts (nofile) doit être assez grand pour avoir suffisamment d’espace pour la taille de vos pools de connexion configurés et autres fichiers ouverts par le système d’exploitation. Il peut être modifié pour permettre une plus grande taille des pools de connexion.

    Ouvrir le fichier limits.conf :

    ```bash
    vim /etc/security/limits.conf
    ```
    
    Ajoutez/modifiez les lignes suivantes :

    ```
    * - nofile 100000
    ```

## <a name="indexing-policy"></a>Stratégie d’indexation
 
* **Exclusion des chemins d’accès inutilisés de l’indexation pour des écritures plus rapides**

    La stratégie d’indexation d’Azure Cosmos DB vous permet de spécifier les chemins d’accès au document à inclure ou exclure de l’indexation en tirant parti des chemins d’accès d’indexation (setIncludedPaths et setExcludedPaths). L’utilisation des chemins d’accès d’indexation peut offrir des performances d’écriture améliorées et réduire le stockage d’index pour les scénarios dans lesquels les modèles de requête sont connus d’avance, puisque les coûts d’indexation sont directement liés au nombre de chemins d’accès uniques indexés. Par exemple, le code suivant montre comment exclure de l’indexation une section entière des documents (également appelée sous-arborescence) avec le caractère générique « * ».

    ### <a name="async-java-sdk-v2-maven-commicrosoftazureazure-cosmosdb"></a><a id="asyncjava2-indexing"></a>Kit de développement logiciel (SDK) Java asynchrone v2 (Maven com.microsoft.azure::azure-cosmosdb)

    ```Java
    Index numberIndex = Index.Range(DataType.Number);
    numberIndex.set("precision", -1);
    indexes.add(numberIndex);
    includedPath.setIndexes(indexes);
    includedPaths.add(includedPath);
    indexingPolicy.setIncludedPaths(includedPaths);
    collectionDefinition.setIndexingPolicy(indexingPolicy);
    ```

    Pour plus d’informations, consultez [Stratégies d’indexation d’Azure Cosmos DB](./index-policy.md).

## <a name="throughput"></a><a id="measure-rus"></a>Débit

* **Mesure et réglage pour réduire l’utilisation d’unités de requête par seconde**

    Azure Cosmos DB propose un riche ensemble d’opérations de base de données, dont les requêtes hiérarchiques et relationnelles avec les fonctions définies par l’utilisateur, les procédures stockées et les déclencheurs, qui fonctionnent toutes au niveau des documents d’une collection de base de données. Le coût associé à chacune de ces opérations varie en fonction du processeur, des E/S et de la mémoire nécessaires à l’exécution de l’opération. Plutôt que de vous soucier de la gestion des ressources matérielles, vous pouvez considérer une unité de demande comme une mesure unique des ressources nécessaires à l’exécution des opérations de base de données et à la réponse à la requête de l’application.

    Le débit est provisionné en fonction du nombre [d’unités de requête](request-units.md) défini pour chaque conteneur. La consommation d’unités de requête est évaluée en fonction d’un taux par seconde. Les applications qui dépassent le taux d’unités de requête configuré pour le conteneur associé sont limitées jusqu’à ce que le taux soit inférieur au niveau configuré pour le conteneur. Si votre application requiert un niveau de débit plus élevé, vous pouvez augmenter le débit en provisionnant des unités de requête supplémentaires.

    La complexité d’une requête a un impact sur le nombre d’unités de requête consommées pour une opération. Le nombre de prédicats, la nature des prédicats, le nombre de fonctions définies par l’utilisateur et la taille du jeu de données sources ont tous une influence sur le coût des opérations de requête.

    Pour mesurer la surcharge de toute opération (création, mise à jour ou suppression), inspectez l’en-tête [x-ms-request-charge](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) afin de mesurer le nombre d’unités de requête consommées par ces opérations. Vous pouvez également examiner la propriété RequestCharge équivalente dans ResourceResponse\<T> ou FeedResponse\<T>.

    ### <a name="async-java-sdk-v2-maven-commicrosoftazureazure-cosmosdb"></a><a id="asyncjava2-requestcharge"></a>Kit de développement logiciel (SDK) Java asynchrone v2 (Maven com.microsoft.azure::azure-cosmosdb)

    ```Java
    ResourceResponse<Document> response = asyncClient.createDocument(collectionLink, documentDefinition, null,
                                                     false).toBlocking.single();
    response.getRequestCharge();
    ```

    Les frais de la requête retournée dans cet en-tête correspondent à une fraction du débit provisionné. Par exemple, si 2 000 RU/seconde sont provisionnées et que la requête ci-dessus retourne 1000 documents de 1 Ko, le coût de l’opération est de 1000. Par conséquent, en une seconde, le serveur honore uniquement deux requêtes avant de limiter le taux de requêtes suivantes. Pour plus d’informations, consultez [Unités de requête](request-units.md) et la [calculatrice d’unités de requête](https://www.documentdb.com/capacityplanner).

* **Gestion de la limite de taux/du taux de requête trop importants**

    Lorsqu’un client tente de dépasser le débit réservé pour un compte, les performances au niveau du serveur ne sont pas affectées et la capacité de débit n’est pas utilisée au-delà du niveau réservé. Le serveur met fin à la requête de manière préventive avec RequestRateTooLarge (code d’état HTTP 429) et il retourne l’en-tête [x-ms-retry-after-ms](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) indiquant la durée, en millisecondes, pendant laquelle l’utilisateur doit attendre avant de réessayer.

   ```xml
   HTTP Status 429,
   Status Line: RequestRateTooLarge
   x-ms-retry-after-ms :100
   ```

    Les kits de développement logiciel (SDK) interceptent tous implicitement cette réponse, respectent l’en-tête retry-after spécifiée par le serveur, puis relancent la requête. La tentative suivante réussira toujours, sauf si plusieurs clients accèdent simultanément à votre compte.

    Si plusieurs clients opèrent simultanément au-delà du taux de requête, le nombre de nouvelles tentatives par défaut actuellement défini sur 9 en interne par le client peut ne pas suffire. Dans ce cas, le client envoie une exception DocumentClientException avec le code d’état 429 à l’application. Vous pouvez modifier le nombre de nouvelles tentatives par défaut en utilisant setRetryOptions sur l’instance ConnectionPolicy. Par défaut, la DocumentClientException avec le code d’état 429 est retournée après un temps d’attente cumulé de 30 secondes si la requête continue à fonctionner au-dessus du taux de requête. Cela se produit même lorsque le nombre de nouvelles tentatives actuel est inférieur au nombre maximal de nouvelles tentatives, qu’il s’agisse de la valeur par défaut de 9 ou d’une valeur définie par l’utilisateur.

    Alors que le comportement de nouvelle tentative automatique permet d’améliorer la résilience et la facilité d’utilisation pour la plupart des applications, il peut se révéler contradictoire lors de l’exécution de tests de performances, en particulier lors de la mesure de la latence. La latence client observée atteindra un pic si l’expérience atteint la limite de serveur et oblige le kit de développement logiciel (SDK) client à effectuer une nouvelle tentative en silence. Pour éviter des pics de latence lors des expériences de performances, mesurez la charge renvoyée par chaque opération et assurez-vous que les requêtes fonctionnent en dessous du taux de requête réservé. Pour plus d’informations, consultez [Unités de requête](request-units.md).

* **Conception de documents plus petits pour un débit plus élevé**

    Les frais de requête (le coût de traitement de requête) d’une opération donnée sont directement liés à la taille du document. Des opérations sur des documents volumineux coûtent plus cher que des opérations sur de petits documents.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la conception de votre application pour une mise à l’échelle et de hautes performances, consultez [Partitionnement, clés de partition et mise à l’échelle dans Cosmos DB](partitioning-overview.md).