---
title: Conseils sur les performances pour le SDK Java v4 Azure Cosmos DB
description: Découvrir les options de configuration clientes afin d’améliorer les performances de la base de données Azure Cosmos pour le SDK Java v4
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: how-to
ms.date: 10/13/2020
ms.author: anfeldma
ms.custom: devx-track-java, contperf-fy21q2
ms.openlocfilehash: 79f8c868b68cba1cff3e99e88e989fcc4d2a3df2
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029020"
---
# <a name="performance-tips-for-azure-cosmos-db-java-sdk-v4"></a>Conseils sur les performances pour le SDK Java v4 Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [SDK Java v4](performance-tips-java-sdk-v4-sql.md)
> * [SDK Java Async v2](performance-tips-async-java.md)
> * [SDK Java Sync v2](performance-tips-java.md)
> * [Kit de développement logiciel (SDK) .NET v3](performance-tips-dotnet-sdk-v3-sql.md)
> * [SDK .NET v2](performance-tips.md)
> 

> [!IMPORTANT]  
> Ces conseils en matière de performances concernent uniquement le SDK Java v4 Azure Cosmos DB. Pour plus d’informations, consultez les [notes de publication](sql-api-sdk-java-v4.md) du SDK Java v4 Azure Cosmos DB, le [dépôt Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos) et le [guide de dépannage](troubleshoot-java-sdk-v4-sql.md) du SDK Java v4 Azure Cosmos DB. Si vous utilisez actuellement une version antérieure à v4, consultez le [guide de migration vers le SDK Java v4 Azure Cosmos DB](migrate-java-v4-sdk.md) pour obtenir de l’aide sur la mise à niveau.
>

Azure Cosmos DB est une base de données distribuée rapide et flexible qui peut être mise à l’échelle en toute transparence avec une latence et un débit garantis. Vous n’avez pas à apporter de modifications d’architecture majeures ou écrire de code complexe pour mettre à l’échelle votre base de données avec Azure Cosmos DB. La réduction et l’augmentation de l’échelle est aussi simple que le passage d’un appel d’API ou de Kit de développement logiciel (SDK). Toutefois, étant donné qu’Azure Cosmos DB est accessible au moyen d’appels réseau, vous pouvez apporter des optimisations côté client de manière à atteindre des performances de pointe quand vous utilisez le SDK Java v4 Azure Cosmos DB.

Si vous vous demandez comment améliorer les performances de votre base de données, lisez ce qui suit :

## <a name="networking"></a>Mise en réseau

* **Mode de connexion : Utiliser le mode direct**
<a id="direct-connection"></a>
    
    Le mode de connexion par défaut du kit de développement logiciel (SDK) Java est le mode direct. Vous pouvez configurer le mode de connexion dans le générateur de clients à l’aide de la méthode *directMode()* ou *gatewayMode()* , comme indiqué ci-dessous. Pour configurer l’un ou l’autre de ces modes avec les paramètres par défaut, appelez l’une des méthodes sans arguments. Sinon, transmettez une instance de classe de paramètres de configuration comme argument (*DirectConnectionConfig* pour *directMode()* ,  *GatewayConnectionConfig* pour *gatewayMode()* .). Pour en savoir plus sur les différentes options de connectivité, consultez l’article sur les [modes de connectivité](sql-sdk-connection-modes.md).
    
    ### <a name="java-v4-sdk"></a><a id="override-default-consistency-javav4"></a> SDK Java V4

    # <a name="async"></a>[Async](#tab/api-async)

    API Async du Kit de développement logiciel (SDK) Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceClientConnectionModeAsync)]

    # <a name="sync"></a>[Synchronisation](#tab/api-sync)

    API Sync du Kit de développement logiciel (SDK) Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=PerformanceClientConnectionModeSync)]

    --- 

    La méthode *directMode()* présente une substitution supplémentaire, pour la raison suivante. Les opérations de plan de contrôle telles que CRUD de base de données et de conteneur utilisent *toujours* le mode passerelle. Lorsque l’utilisateur a configuré le mode direct pour les opérations de plan de données, les opérations de plan de contrôle utilisent les paramètres du mode passerelle par défaut. Cela convient à la plupart des utilisateurs. Toutefois, les utilisateurs qui souhaitent opter pour le mode direct pour les opérations de plan de données et de réglage des paramètres du mode de passerelle de plan de contrôle peuvent utiliser la substitution *directMode()* ci-dessous :

    ### <a name="java-v4-sdk"></a><a id="override-default-consistency-javav4"></a> SDK Java V4

    # <a name="async"></a>[Async](#tab/api-async)

    API Async du Kit de développement logiciel (SDK) Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceClientDirectOverrideAsync)]

    # <a name="sync"></a>[Synchronisation](#tab/api-sync)

    API Sync du Kit de développement logiciel (SDK) Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=PerformanceClientDirectOverrideSync)]

    --- 

<a name="collocate-clients"></a>
* **Colocaliser les clients dans la même région Azure pour de meilleures performances** <a id="same-region"></a>.

    Dans la mesure du possible, placez toutes les applications appelant Azure Cosmos DB dans la même région que la base de données Azure Cosmos. Pour une comparaison approximative, les appels à Azure Cosmos DB dans la même région s’effectuent en 1 à 2 ms, mais la latence entre les côtes Ouest et Est des États-Unis est supérieure à 50 ms. Cette latence peut probablement varier d’une requête à l’autre, en fonction de l’itinéraire utilisé par la requête lorsqu’elle passe du client à la limite du centre de données Azure. Pour obtenir la latence la plus faible possible, l’application appelante doit être située dans la même région Azure que le point de terminaison Azure Cosmos DB configuré. Pour obtenir la liste des régions disponibles, voir [Régions Azure](https://azure.microsoft.com/regions/#services).

    :::image type="content" source="./media/performance-tips/same-region.png" alt-text="Illustration de la stratégie de connexion Azure Cosmos DB" border="false":::

    Une application qui interagit avec un compte Azure Cosmos DB multirégion doit configurer des [emplacements préférés](tutorial-global-distribution-sql-api.md#preferred-locations) pour que les demandes soient transmises à une région colocalisée.

* **Activez l’accélération réseau sur votre machine virtuelle Azure pour réduire la latence.**

Nous vous recommandons de suivre les instructions relatives à l’activation de l’accélération réseau dans votre machine virtuelle Azure [Windows (instructions ici)](../virtual-network/create-vm-accelerated-networking-powershell.md) ou [Linux (instructions ici)](../virtual-network/create-vm-accelerated-networking-cli.md) pour optimiser les performances.

Sans accélération réseau, les E/S qui transitent entre votre machine virtuelle Azure et d’autres ressources Azure peuvent être inutilement routées par le biais d’un hôte et d’un commutateur virtuel situés entre la machine virtuelle et sa carte réseau. Le fait d’avoir l’hôte et le commutateur virtuel inline dans le chemin de données entraîne non seulement une augmentation de la latence et de l’instabilité dans le canal de communication, mais aussi le vol des cycles processeur de la machine virtuelle. L’accélération réseau offre plusieurs avantages : la machine virtuelle interagit directement avec la carte réseau sans intermédiaires, les détails de la stratégie réseau précédemment gérés par l’hôte et le commutateur virtuel sont désormais gérés dans le matériel au niveau de la carte réseau, et l’hôte et le commutateur virtuel sont contournés. L’activation de l’accélération réseau se traduit généralement par une latence plus faible et plus *cohérente*, un débit plus élevé et une utilisation réduite du processeur.

Limitations : l’accélération réseau doit être prise en charge sur le système d’exploitation de la machine virtuelle et ne peut être activée que si la machine virtuelle est arrêtée et libérée. La machine virtuelle ne peut pas être déployée avec Azure Resource Manager.

Pour plus d’informations, consultez les instructions propres à [Windows](../virtual-network/create-vm-accelerated-networking-powershell.md) et à [Linux](../virtual-network/create-vm-accelerated-networking-cli.md).

## <a name="sdk-usage"></a>Utilisation du Kit de développement logiciel (SDK)
* **Installation du kit de développement logiciel (SDK) le plus récent**

    Les SDK Azure Cosmos DB sont constamment améliorés pour fournir des performances optimales. Consultez les pages du [SDK Azure Cosmos DB](sql-api-sdk-async-java.md) pour déterminer quel est le SDK le plus récent et passer en revue les améliorations.

* **Utiliser un client Azure Cosmos DB singleton pour la durée de vie de votre application**

    Chaque instance du client Azure Cosmos DB est thread-safe et effectue une gestion des connexions efficace et une mise en cache d’adresses. Pour permettre une gestion des connexions efficace par le client Azure Cosmos DB et améliorer ses performances, nous vous recommandons d’utiliser une seule instance du client Azure Cosmos DB par AppDomain pour la durée de vie de l’application.

   <a id="max-connection"></a>

* **Utiliser le niveau de cohérence le plus bas requis pour votre application**

    Quand vous créez un *CosmosClient*, la cohérence par défaut utilisée est *Session* si elle n’est pas explicitement définie. Si la cohérence *Session* n’est pas requise par votre logique d’application, attribuez *Eventual* à *Consistency*. Remarque : Il est recommandé d’utiliser au moins une cohérence *Session* dans les applications utilisant le processeur de flux de modification Azure Cosmos DB.

* **Utiliser l’API Async pour maximiser le débit provisionné**

    Le SDK Java v4 Azure Cosmos DB regroupe deux API : Sync et Async. En gros, l’API Async implémente la fonctionnalité du SDK, tandis que l’API Sync est un wrapper dynamique qui effectue des appels de blocage à l’API Async. Cela vient trancher avec l’ancien SDK Java v2 Async Azure Cosmos DB, qui était uniquement asynchrone, et avec l’ancien SDK Java v2 Sync Azure Cosmos DB, qui était uniquement synchrone et dont l’implémentation était entièrement séparée. 
    
    Le choix de l’API est déterminé lors de l’initialisation du client : un *CosmosAsyncClient* prend en charge l’API Async alors qu’un *CosmosClient* prend en charge l’API Sync. 
    
    L’API Async implémente des E/S non bloquantes et constitue le meilleur choix si votre objectif est de maximiser le débit lors de l’émission de demandes à destination d’Azure Cosmos DB. 
    
    L’utilisation de l’API Sync peut être un bon choix si vous voulez une API qui se bloque lors de la réponse à chaque demande, ou si l’opération synchrone est le paradigme dominant dans votre application. Par exemple, vous pouvez utiliser l’API Sync quand vous persistez des données sur Azure Cosmos DB dans une application de microservices, à condition toutefois que le débit ne soit pas critique.  
    
    Sachez simplement que le débit de l’API Sync se dégrade avec l’augmentation du temps de réponse aux demandes, tandis que l’API Async peut saturer les capacités de bande passante totale de votre matériel. 
    
    La colocalisation géographique peut vous faire bénéficier d’un débit plus élevé et plus cohérent lors de l’utilisation de l’API Sync (consultez [Colocaliser des clients dans la même région Azure pour les performances](#collocate-clients)), mais elle n’est pas censée dépasser le débit possible de l’API Async.

    Certains utilisateurs peuvent également ne pas connaître [Project Reactor](https://projectreactor.io/), le framework Reactive Streams utilisé pour implémenter l’API Async du SDK Java v4 Azure Cosmos DB. Si cela pose problème, nous vous recommandons de lire notre [guide de présentation des modèles Reactor](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/reactor-pattern-guide.md), puis de consulter cette [introduction à la programmation réactive](https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Intro) pour vous familiariser avec ces outils. Si vous avez déjà utilisé Azure Cosmos DB avec une interface Async et le SDK Java v2 Async Azure Cosmos DB, vous connaissez peut-être [ReactiveX](http://reactivex.io/)/[RxJava](https://github.com/ReactiveX/RxJava), mais ne savez pas ce qui a changé dans Project Reactor. Dans ce cas, consultez notre [guide sur Reactor et RxJava](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/reactor-rxjava-guide.md) pour en savoir plus.

    Les extraits de code suivants montrent comment initialiser votre client Azure Cosmos DB pour l’API Async ou l’API Sync, respectivement :

    ### <a name="java-v4-sdk"></a><a id="override-default-consistency-javav4"></a> SDK Java V4

    # <a name="async"></a>[Async](#tab/api-async)

    API Async du Kit de développement logiciel (SDK) Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceClientAsync)]

    # <a name="sync"></a>[Synchronisation](#tab/api-sync)

    API Sync du Kit de développement logiciel (SDK) Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=PerformanceClientSync)]

    --- 

* **Paramètre ConnectionPolicy**

    Par défaut, les demandes Cosmos DB en mode direct sont effectuées sur TCP lors de l’utilisation du SDK Java v4 Azure Cosmos DB. En interne, le mode direct utilise une architecture spéciale pour gérer dynamiquement les ressources réseau et obtenir de meilleures performances.

    Dans le SDK Java v4 Azure Cosmos DB, le mode direct est le meilleur choix pour améliorer les performances des bases de données avec la plupart des charges de travail. 

    * ***Vue d’ensemble du mode direct** _

        :::image type="content" source="./media/performance-tips-async-java/rntbdtransportclient.png" alt-text="Illustration de l’architecture du mode direct" border="false":::

        L’architecture côté client utilisée en mode direct permet une utilisation prévisible du réseau et un accès multiplexé aux réplicas Azure Cosmos DB. Le diagramme ci-dessus montre comment le mode direct route les demandes des clients vers les réplicas dans le back-end Cosmos DB. L’architecture du mode direct alloue jusqu’à 10 _ *canaux** côté client par réplica de base de données. Un canal est une connexion TCP précédée d’une mémoire tampon des requêtes, qui correspond à une profondeur de 30 requêtes. Les canaux appartenant à un réplica sont alloués dynamiquement en fonction des besoins du **point de terminaison de service** du réplica. Quand l’utilisateur émet une requête en mode direct, **TransportClient** route la requête vers le point de terminaison de service approprié en fonction de la clé de partition. La **file d’attente des requêtes** met en mémoire tampon les requêtes avant le point de terminaison de service.

    * ***Options de configuration du mode direct** _

        Si vous souhaitez opter pour un comportement de mode direct non défini par défaut, créez une instance _DirectConnectionConfig* et personnalisez ses propriétés, puis transmettez l’instance des propriétés personnalisées à la méthode *directMode()* dans le générateur de clients Azure Cosmos DB.

        Ces paramètres de configuration contrôlent le comportement de l’architecture du mode direct sous-jacent dont il est question précédemment.

        Pour commencer, utilisez les paramètres de configuration recommandés ci-dessous. Ces options *DirectConnectionConfig* correspondent à des paramètres de configuration avancés susceptibles d’affecter les performances du kit de développement logiciel (SDK) de manière inattendue. Nous recommandons donc aux utilisateurs de ne pas les modifier, sauf s’ils ont une très bonne connaissance des compromis et que cela est absolument nécessaire. Si vous rencontrez des problèmes sur ce point particulier, contactez l’[équipe Azure Cosmos DB](mailto:CosmosDBPerformanceSupport@service.microsoft.com).

        | Option de configuration       | Default   |
        | :------------------:       | :-----:   |
        | idleConnectionTimeout      | "PT0"     |
        | maxConnectionsPerEndpoint  | "130"     |
        | connectTimeout             | "PT5S"    |
        | idleEndpointTimeout        | "PT1H"    |
        | maxRequestsPerConnection   | "30"      |

* **Paramétrage des requêtes parallèles pour les collections partitionnées**

    Le SDK Java v4 Azure Cosmos DB prend en charge les requêtes parallèles, ce qui vous permet d’interroger une collection partitionnée en parallèle. Pour plus d’informations, consultez les [exemples de code](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples) concernant l’utilisation du SDK Java v4 Azure Cosmos DB. Les requêtes parallèles sont conçues pour améliorer la latence des requêtes et le débit sur leur équivalent série.

    * ***Optimisation de setMaxDegreeOfParallelism\:** _
    
        Les requêtes parallèles interrogent plusieurs partitions en parallèle. Les données d’une collection partitionnée individuelle sont toutefois extraites en série dans le cadre de la requête. Utilisez donc le paramètre setMaxDegreeOfParallelism pour définir le nombre de partitions qui augmente les chances de résultats de la requête, sous réserve que toutes les autres conditions système restent inchangées. Si vous ne connaissez pas le nombre de partitions, vous pouvez utiliser le paramètre setMaxDegreeOfParallelism pour définir un nombre élevé, et le système sélectionne le minimum (nombre de partitions, entrée fournie par l’utilisateur) comme degré maximal de parallélisme.

        Il est important de noter que les requêtes parallèles produisent de meilleurs résultats si les données sont réparties de manière homogène entre toutes les partitions. Si la collection est partitionnée de telle façon que toutes les données retournées par une requête, ou une grande partie d’entre elles, sont concentrées sur quelques partitions (une partition dans le pire des cas), les performances de la requête sont altérées par ces partitions.

    _ ***Optimisation de setMaxBufferedItemCount\:** _
    
        Parallel query is designed to pre-fetch results while the current batch of results is being processed by the client. The pre-fetching helps in overall latency improvement of a query. setMaxBufferedItemCount limits the number of pre-fetched results. Setting setMaxBufferedItemCount to the expected number of results returned (or a higher number) enables the query to receive maximum benefit from pre-fetching.

        Pre-fetching works the same way irrespective of the MaxDegreeOfParallelism, and there is a single buffer for the data from all partitions.

_ **Effectuer une montée en charge de votre charge de travail cliente**

    If you are testing at high throughput levels, the client application may become the bottleneck due to the machine capping out on CPU or network utilization. If you reach this point, you can continue to push the Azure Cosmos DB account further by scaling out your client applications across multiple servers.

    A good rule of thumb is not to exceed >50% CPU utilization on any given server, to keep latency low.

   <a id="tune-page-size"></a>

* **Réglage de la taille de la page des flux de lecture/requêtes pour de meilleures performances**

    Pendant une lecture groupée de documents à l’aide de la fonctionnalité de flux de lecture (par exemple, *readItems*) ou l’émission d’une requête SQL (*queryItems*), les résultats sont retournés de façon segmentée si le jeu de résultats est trop volumineux. Par défaut, les résultats sont retournés dans des segments de 100 éléments ou de 1 Mo, selon la limite atteinte en premier.

    Supposons que votre application émette une requête pour Azure Cosmos DB et que votre application nécessite l’ensemble complet des résultats de la requête pour effectuer sa tâche. Pour réduire le nombre de boucles réseau nécessaires pour récupérer tous les résultats applicables, vous pouvez augmenter la taille de la page en ajustant le champ de l’en-tête de demande [x-ms-max-item-count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers). 

    * Pour les requêtes à partition unique, le fait d’ajuster la valeur du champ [x-ms-max-item-count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) à -1 (taille de page non limitée) maximise la latence en minimisant le nombre de pages de réponse aux requêtes : le jeu de résultats complet est donc retourné dans une seule page ou, si la requête dépasse le délai d’expiration, dans le nombre minimum de pages possible. Cela permet de gagner du temps lors des allers-retours des requêtes.
    
    * Pour les requêtes sur plusieurs partitions, vous risquez de surcharger le SDK avec des tailles de page ingérables si vous définissez le champ [x-ms-max-item-count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) avec la valeur -1 et supprimez la taille de page limite. Dans le cas de partitions multiples, nous vous recommandons d’attribuer à la taille de page limite une valeur élevée mais finie, comme 1 000, pour réduire la latence. 
    
    Dans certaines applications, vous n’aurez peut-être pas besoin de l’ensemble complet des résultats de la requête. Si vous avez besoin d’afficher uniquement quelques résultats (par exemple, si votre interface utilisateur ou API d’application retourne seulement 10 résultats à la fois), vous pouvez également réduire la taille de la page à 10 résultats afin de baisser le débit consommé pour les lectures et les requêtes.

    Vous pouvez également définir l’argument de taille de page préférée de la méthode *byPage* au lieu de modifier directement le champ d’en-tête REST. Gardez à l’esprit que [x-ms-max-item-count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) et l’argument de taille de page préférée de *byPage* définissent uniquement une limite supérieure pour la taille de page ; il ne s’agit pas d’une exigence absolue. Donc, pour diverses raisons, Azure Cosmos DB peut retourner des pages qui sont plus petites que votre taille de page préférée. 

* **Utilisation du Scheduler approprié (éviter le vol de threads Netty E/S Eventloop)**

    Les fonctionnalités asynchrones du SDK Java Azure Cosmos DB sont basées sur des E/S non bloquantes [netty](https://netty.io/). Le Kit de développement logiciel (SDK) utilise un nombre fixe de threads d’E/S netty eventloop (autant de cœurs de processeur présents sur votre machine) pour l’exécution d’opérations d’E/S. Le Flux retourné par l’API émet le résultat sur l’un des threads netty eventloop d’E/S partagés. Il est donc important de ne pas bloquer les threads netty eventloop d’E/S partagés. Un travail intensif de l’UC ou le blocage de l’opération sur le thread netty eventloop d’E/S peut provoquer un interblocage ou réduire considérablement le débit du Kit de développement logiciel (SDK).

    Par exemple, le code suivant exécute un travail intensif de l’UC sur le thread netty eventloop d’E/S :
    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-noscheduler"></a>API Async du SDK Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceNeedsSchedulerAsync)]

    Une fois le résultat reçu, si vous souhaitez effectuer un travail intensif de l’UC sur le résultat, vous devez éviter de le faire sur le thread netty eventloop d’E/S. Vous pouvez fournir à la place votre propre Scheduler pour fournir votre propre thread pour l’exécution de votre travail, comme indiqué ci-dessous (nécessite `import reactor.core.scheduler.Schedulers`).

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-scheduler"></a>API Async du SDK Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceAddSchedulerAsync)]

    En fonction de votre type de travail, vous devez utiliser le Scheduler Reactor existant approprié pour votre travail. Lire ici [``Schedulers``](https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/Schedulers.html).

    Pour plus d’informations sur le SDK Java v4 Azure Cosmos DB, consultez le [répertoire Cosmos DB du SDK Azure pour le dépôt unique Java sur GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-cosmos).

* **Optimiser les paramètres de journalisation dans votre application**

    Pour diverses raisons, vous souhaiterez ou devrez peut-être ajouter la journalisation dans un thread qui génère un débit de demande élevé. Si votre objectif est de saturer complètement le débit provisionné d’un conteneur avec les requêtes générées par ce thread, les optimisations de la journalisation peuvent améliorer considérablement les performances.

    * ***Configurer un enregistreur asynchrone** _

        La latence d’un enregistreur d’événements synchrone prend nécessairement en compte le calcul de latence globale de votre thread générateur de demandes. Un enregistreur d’événements asynchrone, tel que [log4j2](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flogging.apache.org%2Flog4j%2Flog4j-2.3%2Fmanual%2Fasync.html&data=02%7C01%7CCosmosDBPerformanceInternal%40service.microsoft.com%7C36fd15dea8384bfe9b6b08d7c0cf2113%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637189868158267433&sdata=%2B9xfJ%2BWE%2F0CyKRPu9AmXkUrT3d3uNA9GdmwvalV3EOg%3D&reserved=0), est recommandé pour découpler la surcharge de journalisation de vos threads d’application hautes performances.

    _***Désactiver la journalisation de netty** _

        Netty library logging is chatty and needs to be turned off (suppressing sign in the configuration may not be enough) to avoid additional CPU costs. If you are not in debugging mode, disable netty's logging altogether. So if you are using log4j to remove the additional CPU costs incurred by ``org.apache.log4j.Category.callAppenders()`` from netty add the following line to your codebase:

        ```java
        org.apache.log4j.Logger.getLogger("io.netty").setLevel(org.apache.log4j.Level.OFF);
        ```

 _ **Limite des ressources des fichiers ouverts du système d’exploitation**
 
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

* **Spécifier la clé de partition dans les écritures de point**

    Pour améliorer les performances des écritures de point, spécifiez la clé de partition de l’élément dans l’appel d’API de l’écriture de point, comme indiqué ci-dessous :

    # <a name="async"></a>[Async](#tab/api-async)

    API Async du Kit de développement logiciel (SDK) Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceNoPKAsync)]

    # <a name="sync"></a>[Synchronisation](#tab/api-sync)

    API Sync du Kit de développement logiciel (SDK) Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=PerformanceNoPKSync)]

    --- 

    plutôt que de fournir uniquement l’instance de l’élément, comme indiqué ci-dessous :

    # <a name="async"></a>[Async](#tab/api-async)

    API Async du Kit de développement logiciel (SDK) Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceAddPKAsync)]

    # <a name="sync"></a>[Synchronisation](#tab/api-sync)

    API Sync du Kit de développement logiciel (SDK) Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=PerformanceAddPKSync)]

    --- 

    Ce dernier est pris en charge, mais augmente la latence de votre application. Le SDK doit analyser l’élément et extraire la clé de partition.

## <a name="indexing-policy"></a>Stratégie d’indexation
 
* **Exclusion des chemins d’accès inutilisés de l’indexation pour des écritures plus rapides**

    La stratégie d’indexation d’Azure Cosmos DB vous permet de spécifier les chemins d’accès au document à inclure ou exclure de l’indexation en tirant parti des chemins d’accès d’indexation (setIncludedPaths et setExcludedPaths). L’utilisation des chemins d’accès d’indexation peut offrir des performances d’écriture améliorées et réduire le stockage d’index pour les scénarios dans lesquels les modèles de requête sont connus d’avance, puisque les coûts d’indexation sont directement liés au nombre de chemins d’accès uniques indexés. Par exemple, le code suivant montre comment inclure et exclure de l’indexation une section entière des documents (également appelée sous-arborescence) avec le caractère générique « * ».

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos"></a><a id="java4-indexing"></a>Kit de développement logiciel (SDK) Java V4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateIndexingAsync)]

    Pour plus d’informations, consultez [Stratégies d’indexation d’Azure Cosmos DB](index-policy.md).

## <a name="throughput"></a>Débit
<a id="measure-rus"></a>

* **Mesure et réglage pour réduire l’utilisation d’unités de requête par seconde**

    Azure Cosmos DB propose un riche ensemble d’opérations de base de données, dont les requêtes hiérarchiques et relationnelles avec les fonctions définies par l’utilisateur, les procédures stockées et les déclencheurs, qui fonctionnent toutes au niveau des documents d’une collection de base de données. Le coût associé à chacune de ces opérations varie en fonction du processeur, des E/S et de la mémoire nécessaires à l’exécution de l’opération. Plutôt que de vous soucier de la gestion des ressources matérielles, vous pouvez considérer une unité de demande comme une mesure unique des ressources nécessaires à l’exécution des opérations de base de données et à la réponse à la requête de l’application.

    Le débit est provisionné en fonction du nombre [d’unités de requête](request-units.md) défini pour chaque conteneur. La consommation d’unités de requête est évaluée en fonction d’un taux par seconde. Les applications qui dépassent le taux d’unités de requête configuré pour le conteneur associé sont limitées jusqu’à ce que le taux soit inférieur au niveau configuré pour le conteneur. Si votre application requiert un niveau de débit plus élevé, vous pouvez augmenter le débit en provisionnant des unités de requête supplémentaires.

    La complexité d’une requête a un impact sur le nombre d’unités de requête consommées pour une opération. Le nombre de prédicats, la nature des prédicats, le nombre de fonctions définies par l’utilisateur et la taille du jeu de données sources ont tous une influence sur le coût des opérations de requête.

    Pour mesurer la surcharge de toute opération (création, mise à jour ou suppression), inspectez l’en-tête [x-ms-request-charge](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) afin de mesurer le nombre d’unités de requête consommées par ces opérations. Vous pouvez également examiner la propriété RequestCharge équivalente dans ResourceResponse\<T> ou FeedResponse\<T>.

    # <a name="async"></a>[Async](#tab/api-async)

    API Async du Kit de développement logiciel (SDK) Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=PerformanceRequestChargeAsync)]

    # <a name="sync"></a>[Synchronisation](#tab/api-sync)

    API Sync du Kit de développement logiciel (SDK) Java v4 (Maven com.azure::azure-cosmos)

    [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=PerformanceRequestChargeSync)]

    --- 

    Les frais de la requête retournée dans cet en-tête correspondent à une fraction du débit provisionné. Par exemple, si 2 000 RU/seconde sont provisionnées et que la requête ci-dessus retourne 1000 documents de 1 Ko, le coût de l’opération est de 1000. Par conséquent, en une seconde, le serveur honore uniquement deux requêtes avant de limiter le taux de requêtes suivantes. Pour plus d’informations, consultez [Unités de requête](request-units.md) et la [calculatrice d’unités de requête](https://www.documentdb.com/capacityplanner).

<a id="429"></a>
* **Gestion de la limite de taux/du taux de requête trop importants**

    Lorsqu’un client tente de dépasser le débit réservé pour un compte, les performances au niveau du serveur ne sont pas affectées et la capacité de débit n’est pas utilisée au-delà du niveau réservé. Le serveur met fin à la requête de manière préventive avec RequestRateTooLarge (code d’état HTTP 429) et il retourne l’en-tête [x-ms-retry-after-ms](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) indiquant la durée, en millisecondes, pendant laquelle l’utilisateur doit attendre avant de réessayer.

    ```xml
        HTTP Status 429,
        Status Line: RequestRateTooLarge
        x-ms-retry-after-ms :100
    ```

    Les kits de développement logiciel (SDK) interceptent tous implicitement cette réponse, respectent l’en-tête retry-after spécifiée par le serveur, puis relancent la requête. La tentative suivante réussira toujours, sauf si plusieurs clients accèdent simultanément à votre compte.

    Si plusieurs clients opèrent simultanément au-delà du taux de requête, le nombre de nouvelles tentatives par défaut actuellement défini sur 9 en interne par le client peut ne pas suffire. Dans ce cas, le client lève une exception *CosmosClientException* avec le code d’état 429 à l’application. Vous pouvez modifier le nombre de nouvelles tentatives par défaut en utilisant setRetryOptions sur l’instance ConnectionPolicy. Par défaut, l’exception *CosmosClientException* avec le code d’état 429 est retournée après un temps d’attente cumulé de 30 secondes si la requête continue à fonctionner au-dessus du taux de requête. Cela se produit même lorsque le nombre de nouvelles tentatives actuel est inférieur au nombre maximal de nouvelles tentatives, qu’il s’agisse de la valeur par défaut de 9 ou d’une valeur définie par l’utilisateur.

    Alors que le comportement de nouvelle tentative automatique permet d’améliorer la résilience et la facilité d’utilisation pour la plupart des applications, il peut se révéler contradictoire lors de l’exécution de tests de performances, en particulier lors de la mesure de la latence. La latence client observée atteindra un pic si l’expérience atteint la limite de serveur et oblige le kit de développement logiciel (SDK) client à effectuer une nouvelle tentative en silence. Pour éviter des pics de latence lors des expériences de performances, mesurez la charge renvoyée par chaque opération et assurez-vous que les requêtes fonctionnent en dessous du taux de requête réservé. Pour plus d’informations, consultez [Unités de requête](request-units.md).

* **Conception de documents plus petits pour un débit plus élevé**

    Les frais de requête (le coût de traitement de requête) d’une opération donnée sont directement liés à la taille du document. Des opérations sur des documents volumineux coûtent plus cher que des opérations sur de petits documents. Dans l’idéal, concevez votre application et vos workflows pour que la taille de votre élément soit d’environ 1 Ko (ou d’un ordre ou d’une magnitude similaire). Pour les applications sensibles à la latence, évitez les éléments volumineux comme les documents de plusieurs mégaoctets qui ralentissent votre application.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la conception de votre application pour une mise à l’échelle et de hautes performances, consultez [Partitionnement, clés de partition et mise à l’échelle dans Cosmos DB](partitioning-overview.md).
