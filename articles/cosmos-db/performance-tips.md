---
title: Conseils sur les performances Azure Cosmos DB pour le kit SDK .NET v2
description: Découvrez les options de configuration côté client permettant d’améliorer les performances d’Azure Cosmos DB pour le kit .NET v2.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/13/2020
ms.author: sngun
ms.custom: devx-track-dotnet, contperf-fy21q2
ms.openlocfilehash: 47e20e89c8eaef59b9acd6cf7e31244afd4bcf60
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359045"
---
# <a name="performance-tips-for-azure-cosmos-db-and-net-sdk-v2"></a>Conseils sur les performances pour Azure Cosmos DB et le kit SDK .NET v2
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Kit de développement logiciel (SDK) .NET v3](performance-tips-dotnet-sdk-v3-sql.md)
> * [Kit SDK .NET v2](performance-tips.md)
> * [Kit SDK Java v4](performance-tips-java-sdk-v4-sql.md)
> * [SDK Java Async v2](performance-tips-async-java.md)
> * [SDK Java Sync v2](performance-tips-java.md)

Azure Cosmos DB est une base de données distribuée rapide et flexible qui peut être mise à l’échelle en toute transparence avec une latence et un débit garantis. Vous n’avez pas à apporter de modifications d’architecture majeures ou écrire de code complexe pour mettre à l’échelle votre base de données avec Azure Cosmos DB. La réduction et l’augmentation de l’échelle est aussi simple que le passage d’un appel d’API. Pour en savoir plus, voir [Approvisionner le débit d’un conteneur](how-to-provision-container-throughput.md) ou [Approvisionner le débit d’une base de données](how-to-provision-database-throughput.md). Toutefois, étant donné qu’Azure Cosmos DB est accessible par le biais d’appels réseau, vous pouvez apporter des optimisations côté client de manière à atteindre des performances de pointe quand vous utilisez le [Kit de développement logiciel (SDK) SQL .NET](sql-api-sdk-dotnet-standard.md).

Par conséquent, si vous essayez d’améliorer les performances de votre base de données, envisagez les options suivantes :

## <a name="upgrade-to-the-net-v3-sdk"></a>Mise à niveau vers le kit SDK .NET v3

Le kit SDK [.NET v3](https://github.com/Azure/azure-cosmos-dotnet-v3) est sorti. Si vous utilisez le kit SDK .NET v3, reportez-vous au [guide des performances .NET v3](performance-tips-dotnet-sdk-v3-sql.md) pour obtenir les informations suivantes :

- Mode TCP direct par défaut
- Prise en charge de l’API Stream
- Prise en charge d’un sérialiseur personnalisé pour autoriser l’utilisation de System.Text.JSON
- Prise en charge intégrée du traitement par lots et en bloc

## <a name="hosting-recommendations"></a>Recommandations relatives à l’hébergement

**Pour les charges de travail nécessitant de nombreuses requêtes, utilisez Windows 64 bits plutôt que les processus hôte Linux ou Windows 32 bits**

Nous recommandons les processus hôte Windows 64 bits pour améliorer les performances. Le Kit de développement logiciel (SDK) SQL intègre un fichier ServiceInterop.dll natif pour analyser et optimiser les requêtes localement. ServiceInterop.dll est uniquement pris en charge sur la plateforme Windows x64. Pour Linux et les autres plateformes non prises en charge où ServiceInterop.dll n’est pas disponible, il procède à un appel réseau supplémentaire à destination de la passerelle afin d'obtenir la requête optimisée. Les types d’applications suivants utilisent les processus hôte 32 bits par défaut. Pour modifier les processus hôte en traitement 64 bits, procédez comme suit, selon le type de votre application :

- Pour les applications exécutables, vous pouvez modifier les processus hôte en définissant la [plateforme cible](/visualstudio/ide/how-to-configure-projects-to-target-platforms?preserve-view=true&view=vs-2019) sur **x64** dans la fenêtre **Propriétés du projet**, sous l’onglet **Générer**.

- Pour les projets basés sur VSTest, vous pouvez modifier les processus hôte en sélectionnant **Test** > **Paramètres de test** > **Default Processor Architecture as X64** (Définir l’architecture de processeur par défaut sur X64), à partir du menu **Visual Studio Test**.

- Pour les applications web ASP.NET déployées localement, vous pouvez modifier les processus hôte en sélectionnant **Utiliser la version 64 bits d’IIS Express pour les sites et les projets Web**, sous **Outils** > **Options** > **Projects and Solutions (Projets et solutions)**  > **Projets Web**.

- Pour les applications Web ASP.NET déployées sur Azure, vous pouvez modifier les processus hôte en sélectionnant la plateforme **64 bits** dans les **paramètres d’application** dans le Portail Azure.

> [!NOTE] 
> Par défaut, les nouveaux projets Visual Studio sont définis sur **Any CPU**. Nous vous recommandons de définir votre projet sur **x64** afin qu’il ne passe pas à **x86**. Un projet défini sur **Any CPU** peut facilement basculer vers **x86** si une dépendance est ajoutée à x86 uniquement.<br/>
> ServiceInterop.dll doit se trouver dans le dossier à partir duquel le fichier DLL du Kit de développement logiciel (SDK) s’exécute. Cela ne doit être un problème que si vous copiez manuellement des DLL ou si vous avez des systèmes de génération/déploiement personnalisés.
    
**Activer garbage collection (GC) côté serveur**

Réduire la fréquence de garbage collection peut aider dans certains cas. Dans .NET, définissez [gcServer](/dotnet/framework/configure-apps/file-schema/runtime/gcserver-element) sur `true`.

**Effectuer une montée en charge de votre charge de travail cliente**

Si vous effectuez des tests à des niveaux de débit élevé (plus de 50 000 RU/s), l’application cliente peut devenir un goulet d’étranglement en raison du plafonnement sur l’utilisation du processeur ou du réseau. Si vous atteignez ce point, vous pouvez continuer à augmenter le compte Azure Cosmos DB en augmentant la taille des instances de vos applications clientes sur plusieurs serveurs.

> [!NOTE] 
> Une utilisation élevée de l'UC peut entraîner une latence accrue et des exceptions en termes de délai d’expiration des requêtes.

## <a name="networking"></a><a id="networking"></a>Mise en réseau

**Stratégie de connexion : Utiliser le mode de connexion directe**

Le mode de connexion par défaut du kit de développement logiciel (SDK) .NET V2 est la passerelle. Vous configurez le mode de connexion pendant la construction de l’instance `DocumentClient` à l’aide du paramètre `ConnectionPolicy`. Si vous utilisez le mode direct, vous devez également définir le `Protocol` à l’aide du paramètre `ConnectionPolicy`. Pour en savoir plus sur les différentes options de connectivité, consultez l’article sur les [modes de connectivité](sql-sdk-connection-modes.md).

```csharp
Uri serviceEndpoint = new Uri("https://contoso.documents.net");
string authKey = "your authKey from the Azure portal";
DocumentClient client = new DocumentClient(serviceEndpoint, authKey,
new ConnectionPolicy
{
   ConnectionMode = ConnectionMode.Direct, // ConnectionMode.Gateway is the default
   ConnectionProtocol = Protocol.Tcp
});
```

**Épuisement des ports éphémères**

Si vous êtes confronté à un volume de connexion élevé ou à une utilisation élevée des ports sur vos instances, vérifiez d’abord que vos instances clientes sont des singletons. En d’autres termes, les instances clientes doivent être uniques pour la durée de vie de l’application.

En cas d’exécution sur le protocole TCP, le client optimise la latence en utilisant des connexions à long terme par opposition au protocole HTTPS, qui met fin aux connexions après 2 minutes d’inactivité.

Dans les scénarios où vous disposez de peu d’accès et que vous remarquez un nombre de connexions supérieur par rapport au mode de passerelle, vous pouvez :

* Configurez la propriété [ConnectionPolicy.PortReuseMode](/dotnet/api/microsoft.azure.documents.client.connectionpolicy.portreusemode) sur `PrivatePortPool` (effective avec la version du framework > = 4.6.1 et la version .NET Core > = 2,0) : Cette propriété permet au kit SDK d’utiliser un petit pool de ports éphémères pour différents points de terminaison de destination Azure Cosmos DB.
* Configurez la propriété [ConnectionPolicy.Idleconnectiontimeout,](/dotnet/api/microsoft.azure.documents.client.connectionpolicy.idletcpconnectiontimeout) afin qu’elle soit supérieure ou égale à 10 minutes. Les valeurs recommandées sont comprises entre 20 minutes et 24 heures.

**Appel d’OpenAsync pour éviter la latence de démarrage lors de la première requête**

Par défaut, la première requête a une latence plus élevée, car elle doit extraire la table de routage d’adresses. Lorsque vous utilisez le [Kit de développement logiciel (SDK) V2](sql-api-sdk-dotnet.md), appelez `OpenAsync()` une fois pendant l’initialisation afin d’éviter cette latence de démarrage lors de la première requête. L’appel ressemble à ceci : `await client.OpenAsync();`

> [!NOTE]
> `OpenAsync` génère des requêtes afin d’obtenir la table de routage d’adresses pour tous les conteneurs du compte. Pour les comptes qui ont de nombreux conteneurs, mais dont l’application accède à un sous-ensemble, `OpenAsync` génère un volume de trafic inutile, ce qui ralentit l’initialisation. Ainsi, l’utilisation de `OpenAsync` peut s’avérer superflue dans ce scénario, car elle ralentit le démarrage de l’application.

**Pour des performances optimales, colocaliser les clients dans la même région Azure**

Dans la mesure du possible, placez toutes les applications qui appellent Azure Cosmos DB dans la même région que la base de données Azure Cosmos DB. Voici une comparaison approximative : les appels à Azure Cosmos DB dans la même région s’effectuent en 1 à 2 ms, mais la latence entre les côtes Ouest et Est des États-Unis est supérieure à 50 ms. Cette latence peut varier d’une requête à l’autre, en fonction de l’itinéraire utilisé par la requête lorsqu’elle passe du client à la limite du centre de données Azure. Vous pouvez obtenir la latence la plus faible possible en veillant à ce que l’application appelante soit située dans la même région Azure que le point de terminaison Azure Cosmos DB configuré. Pour obtenir la liste des régions disponibles, voir [Régions Azure](https://azure.microsoft.com/regions/#services).

:::image type="content" source="./media/performance-tips/same-region.png" alt-text="Stratégie de connexion Azure Cosmos DB" border="false":::

**Augmenter le nombre de threads/tâches**
<a id="increase-threads"></a>

Étant donné que les appels à Azure Cosmos DB sont effectués sur le réseau, vous devrez peut-être modifier le degré de parallélisme de vos requêtes, afin que l’application cliente attende un temps minimal entre les requêtes. Par exemple, si vous utilisez la [bibliothèque parallèle de tâches](/dotnet/standard/parallel-programming/task-parallel-library-tpl) .NET, créez des centaines de tâches de lecture ou d’écriture dans Azure Cosmos DB.

**Activer la mise en réseau accélérée**
 
Pour réduire la latence et l’instabilité du processeur, nous vous recommandons d’activer la mise en réseau accélérée sur les machines virtuelles clientes. Consultez les articles [Créer une machine virtuelle Windows avec mise en réseau accélérée](../virtual-network/create-vm-accelerated-networking-powershell.md) ou [Créer une machine virtuelle Linux avec mise en réseau accélérée](../virtual-network/create-vm-accelerated-networking-cli.md).

## <a name="sdk-usage"></a>Utilisation du Kit de développement logiciel (SDK)

**Installation du kit de développement logiciel (SDK) le plus récent**

Les SDK Azure Cosmos DB sont constamment améliorés pour fournir des performances optimales. Consultez les pages du [SDK Azure Cosmos DB](sql-api-sdk-dotnet-standard.md) pour déterminer quel est le SDK le plus récent et passer en revue les améliorations.

**Utiliser un client Azure Cosmos DB singleton pour la durée de vie de votre application**

Chaque instance de `DocumentClient` est thread-safe et effectue une gestion des connexions efficace et une mise en cache d’adresses quand le mode direct est sélectionné. Pour permettre une gestion efficace des connexions et améliorer les performances du client SDK, nous vous recommandons d’utiliser une seule instance par `AppDomain` pour la durée de vie de l’application.

**Augmentation de System.Net MaxConnections par hôte lors de l’utilisation du mode passerelle**

Les requêtes d’Azure Cosmos DB sont effectuées via le protocole HTTPS/REST lorsque vous utilisez le mode passerelle. Elles sont soumises à la limite de connexion par défaut par nom d’hôte ou adresse IP. Vous devrez peut-être définir `MaxConnections` sur une valeur plus élevée (100 à 1 000) afin que la bibliothèque cliente puisse utiliser plusieurs connexions simultanées à Azure Cosmos DB. Dans le Kit de développement logiciel (SDK) .NET 1.8.0 et versions ultérieures, la valeur par défaut de [ServicePointManager. DefaultConnectionLimit](/dotnet/api/system.net.servicepointmanager.defaultconnectionlimit) est 50. Pour modifier cette valeur, vous pouvez définir [Documents.Client.ConnectionPolicy.MaxConnectionLimit](/dotnet/api/microsoft.azure.documents.client.connectionpolicy.maxconnectionlimit) sur une valeur supérieure.

**Paramétrer des requêtes parallèles pour les collections partitionnées**

Le Kit de développement logiciel (SDK) SQL .NET version 1.9.0 et ultérieure prend en charge les requêtes parallèles, qui permettent d’interroger une collection partitionnée en parallèle. Pour plus d’informations, voir les [exemples de code](https://github.com/Azure/azure-cosmos-dotnet-v2/blob/master/samples/code-samples/Queries/Program.cs) concernant l’utilisation des kits SDK. Les requêtes parallèles sont conçues pour améliorer la latence et le débit des requêtes par rapport à leur équivalent série. Les requêtes parallèles fournissent deux paramètres que vous pouvez paramétrer en fonction de vos besoins : 
- `MaxDegreeOfParallelism` contrôle le nombre maximal de partitions qui peuvent être interrogées en parallèle. 
- `MaxBufferedItemCount` contrôle le nombre de résultats pré-récupérés.

**_Ajustement du degré de parallélisme_* _

La requête parallèle fonctionne en interrogeant plusieurs partitions en parallèle. Mais, les données d’une partition individuelle sont extraites en série dans le cadre de la requête. La définition de `MaxDegreeOfParallelism` dans [SDK V2](sql-api-sdk-dotnet.md) sur le nombre de partitions augmente les chances de résultats de la requête, sous réserve que toutes les autres conditions système restent inchangées. Si vous ne connaissez pas le nombre de partitions, vous pouvez définir le degré de parallélisme sur un nombre élevé. Le système choisit la valeur minimale (nombre de partitions, entrée fournie par l’utilisateur) comme degré de parallélisme.

Les requêtes parallèles produisent de meilleurs résultats si les données sont réparties de manière homogène entre toutes les partitions. Si la collection est partitionnée de sorte que toutes les données retournées par une requête, ou une grande partie d’entre elles, sont concentrées sur quelques partitions (une partition dans le pire des cas), ces partitions vont limiter les performances de la requête.

_*_Tuning MaxBufferedItemCount_*_
    
Une requête parallèle est conçue pour pré-extraire les résultats pendant que le lot de résultats actuel est en cours de traitement par le client. Cette pré-récupération permet d’améliorer la latence globale d’une requête. Le paramètre `MaxBufferedItemCount` limite le nombre de résultats pré-récupérés (fetch). Définissez `MaxBufferedItemCount` sur le nombre attendu de résultats retournés (ou un nombre plus élevé) pour permettre à la requête de recevoir le maximum d’avantages de la pré-récupération (fetch).

La pré-récupération (fetch) fonctionne de la même façon, quel que soit le degré de parallélisme, et il existe une seule mémoire tampon pour les données de toutes les partitions.  

_ *Implémentation d’interruption à des intervalles de RetryAfter**

Lors du test de performances, vous devez augmenter la charge jusqu’à une limite d’un petit nombre de requêtes. Si les requêtes sont limitées, l’application cliente doit s’interrompre à la limitation pour l’intervalle de nouvelle tentative spécifié sur le serveur. Le respect de l’interruption garantit un temps d’attente minimal entre chaque tentative. 

La prise en charge des stratégies de nouvelle tentative est incluse dans ces Kits de développement logiciel (SDK) :
- Versions 1.8.0 et ultérieures du [Kit de développement logiciel (SDK) .NET pour SQL](sql-api-sdk-dotnet.md) et le [Kit de développement logiciel (SDK) Java pour SQL](sql-api-sdk-java.md)
- Versions 1.9.0 et ultérieures du [Kit de développement logiciel (SDK) Node.js pour SQL](sql-api-sdk-node.md) et le [Kit de développement logiciel (SDK) Python pour SQL](sql-api-sdk-python.md)
- Toutes les versions prises en charge des Kits de développement logiciel (SDK) [.NET Core](sql-api-sdk-dotnet-core.md) 

Pour plus d’informations, consultez la page [RetryAfter](/dotnet/api/microsoft.azure.documents.documentclientexception.retryafter).
    
Dans la version 1.19 ou version ultérieure du Kit de développement logiciel (SDK) .NET, un mécanisme permet de consigner des informations de diagnostic supplémentaires et de résoudre les problèmes de latence, comme indiqué dans l’exemple suivant. Vous pouvez consigner la chaîne de diagnostic de requêtes ayant une latence de lecture supérieure. La chaîne de diagnostic capturée vous permet de connaître le nombre de fois où vous avez reçu des erreurs 429 pour une requête donnée.

```csharp
ResourceResponse<Document> readDocument = await this.readClient.ReadDocumentAsync(oldDocuments[i].SelfLink);
readDocument.RequestDiagnosticsString 
```

**Mise en cache d’URI de document pour une latence de lecture plus faible**

Effectuez une mise en cache des URI de document dès que possible pour garantir la meilleure lecture. Vous devez définir la logique pour mettre en cache l’ID de ressource lorsque vous créez une ressource. Les recherches par ID de ressource sont plus rapides que les recherches par nom. La mise en cache de ces valeurs améliore donc les performances.

**Réglage de la taille de la page des flux de lecture/requêtes pour de meilleures performances**

Lorsque vous effectuez une lecture groupée de documents à l'aide de la fonctionnalité de flux de lecture (par exemple, `ReadDocumentFeedAsync`) ou lorsque vous émettez une requête SQL, les résultats sont retournés de façon segmentée si le jeu de résultats est trop volumineux. Par défaut, les résultats sont retournés dans des segments de 100 éléments ou de 1 Mo, selon la limite atteinte en premier.

Afin de réduire le nombre de boucles réseau nécessaires pour récupérer tous les résultats applicables, vous pouvez augmenter la taille de la page en utilisant [x-ms-max-item-count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) pour demander jusqu’à 1 000 en-têtes. Lorsque vous avez besoin d’afficher uniquement quelques résultats (par exemple, si votre interface utilisateur ou API d’application retourne seulement 10 résultats à la fois), vous pouvez également réduire la taille de la page à 10 résultats, afin de baisser le débit consommé pour les lectures et requêtes.

> [!NOTE] 
> La propriété `maxItemCount` ne doit pas être utilisée uniquement pour la pagination. Son utilisation principale consiste à améliorer les performances des requêtes en réduisant le nombre maximal d’éléments retournés dans une seule page.  

Vous pouvez également définir la taille de la page à l’aide des SDK Azure Cosmos DB disponibles. La propriété [MaxItemCount](/dotnet/api/microsoft.azure.documents.client.feedoptions.maxitemcount?view=azure-dotnet&preserve-view=true) dans `FeedOptions` vous permet de définir le nombre maximal d’éléments à retourner dans l’opération d’énumération. Lorsque la propriété `maxItemCount` est définie sur -1, le Kit de développement logiciel (SDK) recherche automatiquement la valeur optimale en fonction de la taille du document. Par exemple :
    
```csharp
IQueryable<dynamic> authorResults = client.CreateDocumentQuery(documentCollection.SelfLink, "SELECT p.Author FROM Pages p WHERE p.Title = 'About Seattle'", new FeedOptions { MaxItemCount = 1000 });
```
    
Lorsqu’une requête est exécutée, les données qui en résultent sont envoyées dans un paquet TCP. Si vous spécifiez une valeur trop faible pour `maxItemCount`, le nombre d’allers-retours requis pour envoyer les données dans le paquet TCP est élevé, ce qui affecte les performances. Par conséquent, si vous ne savez pas quelle valeur définir pour la propriété `maxItemCount`, il est préférable d’affecter la valeur -1 et permettre au kit de développement logiciel (SDK) de choisir la valeur par défaut.

**Augmenter le nombre de threads/tâches**

Consultez [Augmenter le nombre de threads/tâches](#increase-threads) dans la section Mise en réseau de cet article.

## <a name="indexing-policy"></a>Stratégie d’indexation
 
**Exclusion des chemins d’accès inutilisés de l’indexation pour des écritures plus rapides**

La stratégie d’indexation d’Azure Cosmos DB vous permet également de spécifier les chemins d’accès de document à inclure ou exclure de l’indexation en utilisant les chemins d’accès d’indexation (IndexingPolicy.IncludedPaths et IndexingPolicy.ExcludedPaths). Les chemins d’accès d’indexation peuvent améliorer les performances d’écriture et réduire le stockage des index pour les scénarios dans lesquels les modèles de requête sont connus au préalable. Cela est dû au fait que les coûts d’indexation correspondent directement au nombre de chemins d’accès uniques indexés. Par exemple, ce code montre comment exclure une section entière des documents (appelée sous-arborescence) de l’indexation à l’aide du caractère générique « * » :

```csharp
var collection = new DocumentCollection { Id = "excludedPathCollection" };
collection.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
collection.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/nonIndexedContent/*");
collection = await client.CreateDocumentCollectionAsync(UriFactory.CreateDatabaseUri("db"), collection);
```

Pour plus d’informations, consultez [Stratégies d’indexation d’Azure Cosmos DB](index-policy.md).

## <a name="throughput"></a><a id="measure-rus"></a>Débit

**Mesurer et optimiser de façon à réduire l’utilisation des unités de requête par seconde**

Azure Cosmos DB offre un ensemble complet d’opérations de base de données. Ces opérations incluent les requêtes hiérarchiques et relationnelles avec les fonctions définies par l’utilisateur, les procédures stockées et les déclencheurs, qui fonctionnent tous au niveau des documents d’une collection de base de données. Le coût associé à chacune de ces opérations varie en fonction du processeur, des E/S et de la mémoire nécessaires à l’exécution de l’opération. Plutôt que de vous soucier de la gestion des ressources matérielles, vous pouvez considérer une unité de requête (RU) comme une mesure unique des ressources nécessaires à l’exécution des opérations de base de données et à la réponse à la requête de l’application.

Le débit est approvisionné en fonction du nombre d’[unités de requête](request-units.md) défini pour chaque conteneur. La consommation d’unités de requête est évaluée en fonction d’un taux par seconde. Les applications qui dépassent le taux d’unités de requête configuré pour le conteneur associé sont limitées jusqu’à ce que le taux soit inférieur au niveau configuré pour le conteneur. Si votre application requiert un niveau de débit plus élevé, vous pouvez augmenter le débit en approvisionnant des unités de requête supplémentaires.

La complexité d’une requête a une incidence sur le nombre d’unités de requête consommées pour une opération. Le nombre de prédicats, la nature des prédicats, le nombre de fonctions définies par l’utilisateur et la taille du jeu de données sources ont tous une influence sur le coût des opérations de requête.

Pour mesurer les frais de l’opération (création, mise à jour ou suppression), inspectez l’en-tête [x-ms-request-charge](/rest/api/cosmos-db/common-cosmosdb-rest-response-headers) (ou la propriété `RequestCharge` équivalente dans `ResourceResponse\<T>` ou `FeedResponse\<T>` dans le SDK .NET) afin de déterminer le nombre d’unités de requête consommées par les opérations :

```csharp
// Measure the performance (Request Units) of writes
ResourceResponse<Document> response = await client.CreateDocumentAsync(collectionSelfLink, myDocument);
Console.WriteLine("Insert of document consumed {0} request units", response.RequestCharge);
// Measure the performance (Request Units) of queries
IDocumentQuery<dynamic> queryable = client.CreateDocumentQuery(collectionSelfLink, queryString).AsDocumentQuery();
while (queryable.HasMoreResults)
    {
        FeedResponse<dynamic> queryResponse = await queryable.ExecuteNextAsync<dynamic>();
        Console.WriteLine("Query batch consumed {0} request units", queryResponse.RequestCharge);
    }
```             

Les frais de la requête retournée dans cet en-tête correspondent à une fraction du débit configuré (c’est-à-dire 2 000 RU/seconde). Par exemple, si la requête ci-dessus renvoie 1 000 documents de 1 Ko, le coût de l’opération est de 1 000. Ainsi, en une seconde, le serveur honore uniquement deux requêtes de ce type avant de limiter le taux des requêtes ultérieures. Pour plus d’informations, consultez [Unités de requête](request-units.md) et la [calculatrice d’unités de requête](https://www.documentdb.com/capacityplanner).
<a id="429"></a>

**Gestion de la limite de taux/du taux de requête trop importants**

Lorsqu’un client tente de dépasser le débit réservé pour un compte, les performances au niveau du serveur ne sont pas affectées et la capacité de débit n’est pas utilisée au-delà du niveau réservé. Le serveur met fin à la requête de manière préventive avec RequestRateTooLarge (code d’état HTTP 429). Il renvoie un en-tête [x-ms-retry-after-ms](/rest/api/cosmos-db/common-cosmosdb-rest-response-headers) qui indique la durée, en millisecondes, pendant laquelle l’utilisateur doit attendre avant de retenter la requête.

```http
HTTP Status 429,
Status Line: RequestRateTooLarge
x-ms-retry-after-ms :100
```

Les kits de développement logiciel (SDK) interceptent tous implicitement cette réponse, respectent l’en-tête retry-after spécifiée par le serveur, puis relancent la requête. La tentative suivante réussira toujours, sauf si plusieurs clients accèdent simultanément à votre compte.

Si plusieurs de vos clients opèrent simultanément et systématiquement au-delà du taux de requête, le nombre de nouvelles tentatives par défaut actuellement défini sur 9 en interne par le client ne suffira peut-être pas. Dans ce cas, le client envoie à l’application une DocumentClientException avec le code d’état 429. 

Vous pouvez modifier le nombre de nouvelles tentatives par défaut en définissant les `RetryOptions` sur l’instance `ConnectionPolicy`. Par défaut, la DocumentClientException avec le code d’état 429 est retournée après un temps d’attente cumulé de 30 secondes si la requête continue à fonctionner au-dessus du taux de requête. Cette erreur est renvoyée même lorsque le nombre actuel de nouvelles tentatives est inférieur au nombre maximal de nouvelles tentatives, que la valeur actuelle soit la valeur par défaut 9 ou une valeur définie par l’utilisateur.

Le comportement de nouvelle tentative automatisée contribue à améliorer la résilience et la convivialité pour la plupart des applications. Mais cela peut ne pas être le meilleur comportement lorsque vous effectuez des benchmarks des performances, en particulier lorsque vous mesurez la latence. La latence client observée atteindra un pic si l’expérience atteint la limite de serveur et oblige le kit de développement logiciel (SDK) client à effectuer une nouvelle tentative en silence. Pour éviter des pics de latence lors des expériences de performances, mesurez la charge renvoyée par chaque opération et assurez-vous que les requêtes fonctionnent en dessous du taux de requête réservé. Pour plus d’informations, consultez [Unités de requête](request-units.md).

**Pour un débit plus élevé, concevez en prévision des documents plus petits**

Les frais de requête (à savoir, le coût de traitement de requête) d’une opération donnée sont directement liés à la taille du document. Les opérations sur les documents volumineux coûtent plus cher que les opérations sur les petits documents.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir un exemple d’application permettant d’évaluer Azure Cosmos DB lors de scénarios hautes performances sur quelques ordinateurs clients, consultez [Test des performances et de la mise à l’échelle avec Azure Cosmos DB](performance-testing.md).

Pour en savoir plus sur la conception de votre application pour une mise à l’échelle et de hautes performances, consultez [Partitionnement, clés de partition et mise à l’échelle dans Cosmos DB](partitioning-overview.md).