---
title: Conseils sur les performances Azure Cosmos DB pour le kit SDK .NET v3
description: Découvrez les options de configuration côté client favorisant l’amélioration des performances d’Azure Cosmos DB avec le kit SDK .NET v3.
author: j82w
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/13/2020
ms.author: jawilley
ms.custom: devx-track-dotnet, contperf-fy21q2
ms.openlocfilehash: f503f132794f6d04b587a78b8f838acba26f9ac3
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032012"
---
# <a name="performance-tips-for-azure-cosmos-db-and-net"></a>Conseils sur les performances pour Azure Cosmos DB et .NET
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Kit de développement logiciel (SDK) .NET v3](performance-tips-dotnet-sdk-v3-sql.md)
> * [Kit SDK .NET v2](performance-tips.md)
> * [Kit SDK Java v4](performance-tips-java-sdk-v4-sql.md)
> * [SDK Java Async v2](performance-tips-async-java.md)
> * [SDK Java Sync v2](performance-tips-java.md)

Azure Cosmos DB est une base de données distribuée rapide et flexible, qui peut être mise à l’échelle en toute transparence avec des niveaux de latence et de débit garantis. Vous n’avez pas à apporter de modifications d’architecture majeures ou écrire de code complexe pour mettre à l’échelle votre base de données avec Azure Cosmos DB. La réduction et l’augmentation de l’échelle est aussi simple que le passage d’un appel d’API. Pour en savoir plus, consultez [Approvisionner le débit sur un conteneur](how-to-provision-container-throughput.md) ou [Approvisionner le débit sur une base de données](how-to-provision-database-throughput.md). 

Comme Azure Cosmos DB est accessible via des appels réseau, vous pouvez apporter des optimisations côté client pour atteindre des performances de pointe quand vous utilisez le [kit SDK SQL .NET](sql-api-sdk-dotnet-standard.md).

Si vous essayez d’améliorer les performances de votre base de données, envisagez les options présentées dans les sections suivantes.

## <a name="hosting-recommendations"></a>Recommandations relatives à l’hébergement

**Pour les charges de travail nécessitant de nombreuses requêtes, utilisez Windows 64 bits plutôt que les processus hôte Linux ou Windows 32 bits**

Nous recommandons les processus hôte Windows 64 bits pour améliorer les performances. Le Kit de développement logiciel (SDK) SQL intègre un fichier ServiceInterop.dll natif pour analyser et optimiser les requêtes localement. ServiceInterop.dll est uniquement pris en charge sur la plateforme Windows x64. 

Pour Linux et les autres plateformes non prises en charge où ServiceInterop.dll n’est pas disponible, il procède à un appel réseau supplémentaire à destination de la passerelle afin d'obtenir la requête optimisée. 

Les quatre types d’application listés ici utilisent un traitement 32 bits des processus hôte par défaut. Pour passer à un traitement 64 bits des processus hôte pour votre type d’application, procédez comme suit :

- **Pour les applications exécutables** : Dans la fenêtre **Propriétés du projet**, dans le volet **Build**, définissez la [plateforme cible](/visualstudio/ide/how-to-configure-projects-to-target-platforms?preserve-view=true&view=vs-2019) sur **x64**.

- **Pour les projets de test basés sur VSTest** : Dans le menu Visual Studio **Test**, sélectionnez **Test** > **Paramètres de test**, puis définissez **Architecture de processeur par défaut** sur **X64**.

- **Pour les applications web ASP.NET déployées localement** : Sélectionnez **Outils** > **Options** > **Projets et solutions** > **Projets web**, puis sélectionnez **Utiliser la version 64 bits d’IIS Express pour les sites et les projets web**.

- **Pour les applications web ASP.NET déployées sur Azure** : Dans le portail Azure, dans **Paramètres de l’application**, sélectionnez la plateforme **64 bits**.

> [!NOTE] 
> Par défaut, les nouveaux projets Visual Studio sont définis sur **Any CPU**. Nous vous recommandons de définir votre projet sur **x64** afin qu’il ne passe pas à **x86**. Un projet défini sur **N’importe quelle UC** peut facilement basculer vers **x86** si une dépendance x86 exclusive est ajoutée.<br/>
> Le fichier ServiceInterop.dll doit se trouver dans le dossier à partir duquel la bibliothèque DLL du kit SDK s’exécute. Cela ne doit être un problème que si vous copiez manuellement des DLL ou si vous avez des systèmes de génération ou de déploiement personnalisés.
    
**Activer le garbage collection (GC) côté serveur**

Réduire la fréquence de garbage collection peut aider dans certains cas. Dans .NET, définissez [gcServer](/dotnet/core/run-time-config/garbage-collector#flavors-of-garbage-collection) sur `true`.

**Effectuer une montée en charge de votre charge de travail cliente**

Si vous effectuez des tests à des niveaux de débit élevés ou à des débits supérieurs à 50 000 unités de requête par seconde (RU/s), l’application cliente peut devenir un goulet d’étranglement pour la charge de travail. Cela est dû au fait que l’ordinateur peut atteindre un seuil d’utilisation du processeur ou du réseau. Si vous atteignez ce point, vous pouvez continuer à augmenter le compte Azure Cosmos DB en augmentant la taille des instances de vos applications clientes sur plusieurs serveurs.

> [!NOTE] 
> Une utilisation élevée de l'UC peut entraîner une latence accrue et des exceptions en termes de délai d’expiration des requêtes.

## <a name="networking"></a>Mise en réseau
<a id="direct-connection"></a>

**Stratégie de connexion : Utiliser le mode de connexion directe**

Le mode de connexion par défaut du Kit de développement logiciel (SDK) .NET v3 est le mode direct. Vous configurez le mode de connexion lorsque vous créez l’instance `CosmosClient` dans `CosmosClientOptions`.  Pour en savoir plus sur les différentes options de connectivité, consultez l’article sur les [modes de connectivité](sql-sdk-connection-modes.md).

```csharp
string connectionString = "<your-account-connection-string>";
CosmosClient client = new CosmosClient(connectionString,
new CosmosClientOptions
{
    ConnectionMode = ConnectionMode.Gateway // ConnectionMode.Direct is the default
});
```

**Épuisement des ports éphémères**

Si vous êtes confronté à un volume de connexion élevé ou à une utilisation élevée des ports sur vos instances, vérifiez d’abord que vos instances clientes sont des singletons. En d’autres termes, les instances clientes doivent être uniques pour la durée de vie de l’application.

Lorsqu’il s’exécute sur le protocole TCP, le client optimise la latence en utilisant les connexions à long terme. Cela diffère du protocole HTTPS, qui met fin aux connexions après deux minutes d’inactivité.

Dans les scénarios où vous disposez de peu d’accès, si vous remarquez un nombre de connexions supérieur par rapport au mode passerelle, vous pouvez :

* Configurer la propriété [CosmosClientOptions.PortReuseMode](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.portreusemode) sur `PrivatePortPool` (en vigueur avec la version 4.6.1 ou ultérieure du framework et .NET Core version 2.0 ou ultérieure). Cette propriété permet au kit SDK d’utiliser un petit pool de ports éphémères pour différents points de terminaison de destination Azure Cosmos DB.
* Configurer la propriété [CosmosClientOptions.IdleConnectionTimeout](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.idletcpconnectiontimeout) afin qu’elle soit supérieure ou égale à 10 minutes. Les valeurs recommandées sont comprises entre 20 minutes et 24 heures.

<a id="same-region"></a>

**Pour des performances optimales, colocalisez les clients dans la même région Azure**

Dans la mesure du possible, placez toutes les applications qui appellent Azure Cosmos DB dans la même région que la base de données Azure Cosmos DB. Voici une comparaison approximative : des appels à Azure Cosmos DB dans une même région s’effectuent en 1 à 2 millisecondes (ms), mais la latence entre la côte ouest et la côte est des États-Unis est supérieure à 50 ms. Cette latence peut varier d’une requête à l’autre, en fonction de l’itinéraire utilisé par la requête lorsqu’elle passe du client à la limite du centre de données Azure. 

Vous pouvez obtenir la latence la plus faible possible en veillant à ce que l’application appelante soit située dans la même région Azure que le point de terminaison Azure Cosmos DB configuré. Pour obtenir la liste des régions disponibles, voir [Régions Azure](https://azure.microsoft.com/regions/#services).

:::image type="content" source="./media/performance-tips/same-region.png" alt-text="Colocalisez les clients dans la même région Azure" border="false":::

   <a id="increase-threads"></a>

**Augmenter le nombre de threads/tâches**

Étant donné que les appels à Azure Cosmos DB sont effectués sur le réseau, vous devrez peut-être modifier le degré de concurrence de vos requêtes, afin que l’application cliente attende un temps minimal entre les requêtes. Par exemple, si vous utilisez la [bibliothèque parallèle de tâches](/dotnet/standard/parallel-programming/task-parallel-library-tpl) .NET, créez des centaines de tâches de lecture ou d’écriture dans Azure Cosmos DB.

**Activer la mise en réseau accélérée**
 
Pour réduire la latence et l’instabilité du processeur, nous vous recommandons d’activer la mise en réseau accélérée sur vos machines virtuelles clientes. Pour plus d’informations, consultez [Créer une machine virtuelle Windows avec mise en réseau accélérée](../virtual-network/create-vm-accelerated-networking-powershell.md) ou [Créer une machine virtuelle Linux avec mise en réseau accélérée](../virtual-network/create-vm-accelerated-networking-cli.md).

## <a name="sdk-usage"></a>Utilisation du Kit de développement logiciel (SDK)

**Installation du kit de développement logiciel (SDK) le plus récent**

Les SDK Azure Cosmos DB sont constamment améliorés pour fournir des performances optimales. Pour déterminer le kit SDK le plus récent et passer en revue les améliorations, consultez [SDK Azure Cosmos DB](sql-api-sdk-dotnet-standard.md).

**Utiliser les API de flux**

Le [Kit de développement logiciel (SDK) .NET V3](https://github.com/Azure/azure-cosmos-dotnet-v3) contient des API de flux qui peuvent recevoir et retourner des données sans sérialisation. 

Les applications intermédiaires qui ne consomment pas directement les réponses du Kit de développement logiciel (SDK), mais qui les relayent vers d’autres couches Application, peuvent tirer parti des API de flux. Pour obtenir des exemples sur la gestion des flux, consultez les exemples de [gestion d’élément](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement).

**Utiliser un client Azure Cosmos DB singleton pour la durée de vie de votre application**

Chaque instance de `CosmosClient` est thread-safe et effectue une gestion des connexions efficace et une mise en cache d’adresses efficace en mode direct. Pour permettre une gestion efficace des connexions et améliorer les performances du client SDK, nous vous recommandons d’utiliser une seule instance par `AppDomain` pour la durée de vie de l’application.

Lorsque vous travaillez sur Azure Functions, les instances doivent également suivre les [instructions](../azure-functions/manage-connections.md#static-clients) existantes et gérer une seule instance.

<a id="max-connection"></a>

**Désactiver la réponse de contenu sur les opérations d’écriture**

Pour les charges de travail qui ont des charges utiles de création intensives, attribuez à l’option de requête `EnableContentResponseOnWrite` la valeur `false`. Le service ne renvoie plus la ressource créée ou mise à jour au kit de développement logiciel (SDK). Normalement, comme l’application détient l’objet en cours de création, le service n’a pas besoin de le retourner. Les valeurs d’en-tête sont toujours accessibles, comme des frais de requête. La désactivation de la réponse de contenu peut améliorer les performances, car le kit SDK n’a plus besoin d’allouer de la mémoire ni de sérialiser le corps de la réponse. Cela réduit également l’utilisation de la bande passante réseau pour améliorer encore les performances.  

```csharp
ItemRequestOptions requestOptions = new ItemRequestOptions() { EnableContentResponseOnWrite = false };
ItemResponse<Book> itemResponse = await this.container.CreateItemAsync<Book>(book, new PartitionKey(book.pk), requestOptions);
// Resource will be null
itemResponse.Resource
```

**Activer l’exécution en bloc pour optimiser le débit plutôt que la latence**

Activez l’*exécution en bloc* pour les scénarios où la charge de travail requiert un débit très élevé et où la latence n’est pas aussi importante. Pour plus d’informations sur l’activation de l’exécution en bloc et sur les scénarios où elle devrait être utilisée, consultez [Introduction à la prise en charge de l’exécution en bloc](https://devblogs.microsoft.com/cosmosdb/introducing-bulk-support-in-the-net-sdk).

**Augmenter System.Net MaxConnections par hôte lors de l’utilisation du mode passerelle**

Les requêtes d’Azure Cosmos DB sont effectuées via le protocole HTTPS/REST lorsque vous utilisez le mode passerelle. Elles sont soumises à la limite de connexion par défaut par nom d’hôte ou adresse IP. Vous devrez peut-être définir `MaxConnections` sur une valeur plus élevée (de 100 à 1 000) afin que la bibliothèque cliente puisse utiliser plusieurs connexions simultanées à Azure Cosmos DB. Dans le Kit de développement logiciel (SDK) .NET 1.8.0 et versions ultérieures, la valeur par défaut de [ServicePointManager. DefaultConnectionLimit](/dotnet/api/system.net.servicepointmanager.defaultconnectionlimit) est 50. Pour modifier cette valeur, vous pouvez définir [`Documents.Client.ConnectionPolicy.MaxConnectionLimit`](/dotnet/api/microsoft.azure.documents.client.connectionpolicy.maxconnectionlimit) sur une valeur supérieure.

**Paramétrer des requêtes parallèles pour les collections partitionnées**

Le Kit de développement logiciel (SDK) SQL .NET prend en charge les requêtes parallèles, qui permettent d’interroger un conteneur partitionné en parallèle. Pour plus d’informations, voir les [exemples de code](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/Queries/Program.cs) concernant l’utilisation des kits SDK. Les requêtes parallèles sont conçues pour améliorer la latence et le débit des requêtes par rapport à leur équivalent série. 

Les requêtes parallèles fournissent deux paramètres que vous pouvez paramétrer en fonction de vos besoins : 

- **MaxConcurrency** : Contrôle le nombre maximal de partitions qui peuvent être interrogées en parallèle.

   La requête parallèle fonctionne en interrogeant plusieurs partitions en parallèle. Mais, les données d’une partition individuelle sont extraites en série dans le cadre de la requête. La définition de `MaxConcurrency` dans [SDK V3](https://github.com/Azure/azure-cosmos-dotnet-v3) sur le nombre de partitions augmente les chances de résultats de la requête, sous réserve que toutes les autres conditions système restent inchangées. Si vous ne connaissez pas le nombre de partitions, vous pouvez définir le degré de parallélisme sur un nombre élevé. Le système choisit la valeur minimale (nombre de partitions, entrée fournie par l’utilisateur) comme degré de parallélisme.

    Les requêtes parallèles produisent de meilleurs résultats si les données sont réparties de manière homogène entre toutes les partitions. Si la collection est partitionnée de sorte que toutes les données retournées par une requête, ou une grande partie d’entre elles, sont concentrées sur quelques partitions (une partition dans le pire des cas), ces partitions vont limiter les performances de la requête.
   
- **MaxBufferedItemCount** : Contrôle le nombre de résultats prérécupérés.

   Une requête parallèle est conçue pour pré-extraire les résultats pendant que le lot de résultats actuel est en cours de traitement par le client. Cette pré-récupération permet d’améliorer la latence globale d’une requête. Le paramètre `MaxBufferedItemCount` limite le nombre de résultats pré-récupérés (fetch). Définissez `MaxBufferedItemCount` sur le nombre attendu de résultats retournés (ou un nombre plus élevé) pour permettre à la requête de recevoir le maximum d’avantages de la pré-récupération (fetch).

   La pré-récupération (fetch) fonctionne de la même façon, quel que soit le degré de parallélisme, et il existe une seule mémoire tampon pour les données de toutes les partitions.  

**Implémentation d’interruption à des intervalles de RetryAfter**

Lors du test de performances, vous devez augmenter la charge jusqu’à une limite d’un petit nombre de requêtes. Si les requêtes sont limitées, l’application cliente doit interrompre la limitation pour l’intervalle de nouvelle tentative spécifié sur le serveur. Le respect de l’interruption contribue à garantir un temps d’attente minimal entre les différentes tentatives. 

Pour plus d’informations, consultez la page [RetryAfter](/dotnet/api/microsoft.azure.cosmos.cosmosexception.retryafter?preserve-view=true&view=azure-dotnet#Microsoft_Azure_Cosmos_CosmosException_RetryAfter).
    
Un mécanisme permet de consigner des informations de diagnostic supplémentaires et de résoudre les problèmes de latence, comme indiqué dans l’exemple suivant. Vous pouvez consigner la chaîne de diagnostic de requêtes ayant une latence de lecture supérieure. La chaîne de diagnostic capturée vous aide à comprendre combien de fois vous avez reçu une erreur *429* pour une requête donnée.

```csharp
ItemResponse<Book> readItemResponse = await this.cosmosContainer.ReadItemAsync<Book>("ItemId", new PartitionKey("PartitionKeyValue"));
readItemResponse.Diagnostics.ToString(); 
```

**Augmenter le nombre de threads/tâches**

Consultez [Augmenter le nombre de threads/tâches](#increase-threads) dans la section Mise en réseau de cet article.

## <a name="indexing-policy"></a>Stratégie d’indexation
 
**Exclusion des chemins d’accès inutilisés de l’indexation pour des écritures plus rapides**

La stratégie d’indexation d’Azure Cosmos DB vous permet également de spécifier les chemins de document à inclure ou exclure lors de l’indexation en utilisant les chemins d’accès d’indexation (IndexingPolicy.IncludedPaths et IndexingPolicy.ExcludedPaths). 

Indexer uniquement les chemins dont vous avez besoin vous permet d’améliorer les performances d’écriture, de réduire les frais liés aux unités de requête dans le cadre des opérations d’écriture et de réduire le stockage des index pour les scénarios dans lesquels les modèles de requête sont connus au préalable. Cela est dû au fait que les coûts d’indexation correspondent directement au nombre de chemins d’accès uniques indexés. Par exemple, le code suivant montre comment exclure une section entière des documents (appelée sous-arborescence) de l’indexation à l’aide du caractère générique « * » :

```csharp
var containerProperties = new ContainerProperties(id: "excludedPathCollection", partitionKeyPath: "/pk" );
containerProperties.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
containerProperties.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/nonIndexedContent/*");
Container container = await this.cosmosDatabase.CreateContainerAsync(containerProperties);
```

Pour plus d’informations, consultez [Stratégies d’indexation d’Azure Cosmos DB](index-policy.md).

## <a name="throughput"></a>Débit
<a id="measure-rus"></a>

**RU/s**

Azure Cosmos DB offre un ensemble complet d’opérations de base de données. Ces opérations incluent les requêtes hiérarchiques et relationnelles avec les fichiers UDF (Universal Disk Format), les procédures stockées et les déclencheurs, qui fonctionnent tous au niveau des documents d’une collection de bases de données. 

Les coûts associés à chacune de ces opérations varient en fonction du processeur, des E/S et de la mémoire nécessaires à l’exécution de l’opération. Plutôt que de vous soucier de la gestion des ressources matérielles, vous pouvez considérer une unité de requête (RU) comme une mesure unique des ressources nécessaires à l’exécution des diverses opérations de base de données et au traitement d’une requête d’application.

Le débit est approvisionné en fonction du nombre d’[unités de requête](request-units.md) défini pour chaque conteneur. La consommation d’unités de requête est évaluée sous la forme d’un débit d’unités par seconde. Les applications qui dépassent le taux d’unités de requête configuré pour le conteneur associé sont limitées jusqu’à ce que le taux soit inférieur au niveau configuré pour le conteneur. Si votre application requiert un niveau de débit plus élevé, vous pouvez augmenter le débit en approvisionnant des unités de requête supplémentaires.

La complexité d’une requête a une incidence sur le nombre d’unités de requête consommées pour une opération. Le nombre de prédicats, la nature des prédicats, le nombre de fichiers UDF et la taille du jeu de données source ont tous une influence sur le coût des opérations de requête.

Pour mesurer les frais de l’opération (création, mise à jour ou suppression), inspectez l’en-tête [x-ms-request-charge](/rest/api/cosmos-db/common-cosmosdb-rest-response-headers) (ou la propriété `RequestCharge` équivalente dans `ResourceResponse\<T>` ou `FeedResponse\<T>` dans le SDK .NET) afin de déterminer le nombre d’unités de requête consommées par les opérations :

```csharp
// Measure the performance (Request Units) of writes
ItemResponse<Book> response = await container.CreateItemAsync<Book>(myBook, new PartitionKey(myBook.PkValue));
Console.WriteLine("Insert of item consumed {0} request units", response.RequestCharge);
// Measure the performance (Request Units) of queries
FeedIterator<Book> queryable = container.GetItemQueryIterator<ToDoActivity>(queryString);
while (queryable.HasMoreResults)
    {
        FeedResponse<Book> queryResponse = await queryable.ExecuteNextAsync<Book>();
        Console.WriteLine("Query batch consumed {0} request units", queryResponse.RequestCharge);
    }
```             

Les frais de requête retournés dans cet en-tête correspondent à une fraction de votre débit provisionné (à savoir 2 000 RU/s). Par exemple, si la requête ci-dessus renvoie 1 000 documents de 1 Ko, le coût de l’opération est de 1 000. Ainsi, en une seconde, le serveur honore uniquement deux requêtes de ce type avant de limiter le taux des requêtes ultérieures. Pour plus d’informations, consultez [Unités de requête](request-units.md) et la [calculatrice d’unités de requête](https://www.documentdb.com/capacityplanner).
<a id="429"></a>

**Gestion de la limite de taux/du taux de requête trop importants**

Lorsqu’un client tente de dépasser le débit réservé pour un compte, les performances au niveau du serveur ne sont pas affectées et la capacité de débit n’est pas utilisée au-delà du niveau réservé. Le serveur met fin à la requête de manière préventive avec RequestRateTooLarge (code d’état HTTP 429). Il renvoie un en-tête [x-ms-retry-after-ms](/rest/api/cosmos-db/common-cosmosdb-rest-response-headers) qui indique la durée, en millisecondes, pendant laquelle l’utilisateur doit attendre avant de retenter la requête.

```xml
    HTTP Status 429,
    Status Line: RequestRateTooLarge
    x-ms-retry-after-ms :100
```

Les kits de développement logiciel (SDK) interceptent tous implicitement cette réponse, respectent l’en-tête retry-after spécifiée par le serveur, puis relancent la requête. La tentative suivante réussira toujours, sauf si plusieurs clients accèdent simultanément à votre compte.

Si plusieurs de vos clients opèrent simultanément et systématiquement au-delà du taux de requête, le nombre de nouvelles tentatives par défaut actuellement défini sur 9 en interne par le client ne suffira peut-être pas. Dans ce cas, le client envoie à l’application une exception CosmosException avec le code d’état 429. 

Vous pouvez modifier le nombre de nouvelles tentatives par défaut en définissant les `RetryOptions` sur l’instance `CosmosClientOptions`. Par défaut, l’exception CosmosException avec le code d’état 429 est retournée après un temps d’attente cumulé de 30 secondes si la requête continue à fonctionner au-dessus du taux de requête. Cette erreur est renvoyée même lorsque le nombre actuel de nouvelles tentatives est inférieur au nombre maximal de nouvelles tentatives, que la valeur actuelle soit la valeur par défaut 9 ou une valeur définie par l’utilisateur.

Le comportement de nouvelle tentative automatisée contribue à améliorer la résilience et la convivialité pour la plupart des applications. Mais cela peut ne pas être le meilleur comportement lorsque vous effectuez des benchmarks des performances, en particulier lorsque vous mesurez la latence. La latence client observée atteindra un pic si l’expérience atteint la limite de serveur et oblige le kit de développement logiciel (SDK) client à effectuer une nouvelle tentative en silence. Pour éviter des pics de latence lors d’expériences de performances, mesurez les frais retournés par chaque opération et assurez-vous que les requêtes restent sous le taux de requêtes réservé. 

Pour plus d’informations, consultez [Unités de requête](request-units.md).

**Pour un débit plus élevé, concevez en prévision des documents plus petits**

Les frais de requête (à savoir, le coût de traitement des requêtes) d’une opération donnée sont directement liés à la taille du document. Les opérations sur les documents volumineux coûtent plus cher que les opérations sur les petits documents.

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir un exemple d’application permettant d’évaluer Azure Cosmos DB lors de scénarios hautes performances sur quelques ordinateurs clients, consultez [Test des performances et de la mise à l’échelle avec Azure Cosmos DB](performance-testing.md).

Pour en savoir plus sur la conception de votre application pour une mise à l’échelle et de hautes performances, consultez [Partitionnement, clés de partition et mise à l’échelle dans Cosmos DB](partitioning-overview.md).