---
title: Utilisation d’une bibliothèque cliente de la base de données élastique avec Entity Framework
description: Utilisez la bibliothèque cliente de la base de données élastique et Entity Framework pour le codage de bases de données
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/04/2019
ms.openlocfilehash: 48d43cb2d3c51194d0708a2b9b739a0ee87843d0
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92793396"
---
# <a name="elastic-database-client-library-with-entity-framework"></a>Bibliothèque cliente de la base de données élastique avec Entity Framework
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Ce document présente les modifications d'une application Entity Framework requises pour l'intégration avec les [outils des bases de données élastiques](elastic-scale-introduction.md). L’objectif est de composer une [gestion de carte de partitions](elastic-scale-shard-map-management.md) et un [routage dépendant des données](elastic-scale-data-dependent-routing.md) avec l’approche Entity Framework **Code First** . Le didacticiel [Code First pour une nouvelle base de données](/ef/ef6/modeling/code-first/workflows/new-database) pour Entity Framework sert d’exemple tout au long de ce document. L’exemple de code qui accompagne ce document fait partie de l’ensemble d’échantillons des outils de base de données élastique figurant parmi les exemples de code Visual Studio.

## <a name="downloading-and-running-the-sample-code"></a>Téléchargement et exécution de l'exemple de code

Pour télécharger le code utilisé dans cet article :

* Visual Studio 2012 ou une version ultérieure est nécessaire.
* Téléchargez l'[exemple Outils des bases de données élastiques pour SQL Azure - Intégration Entity Framework](https://github.com/Azure/elastic-db-tools/). Décompressez l’exemple à l’emplacement de votre choix.
* Démarrez Visual Studio.
* Dans Visual Studio, sélectionnez Fichier -> Ouvrir un projet/une solution.
* Dans la boîte de dialogue **Ouvrir un projet** , accédez à l’exemple que vous avez téléchargé, puis sélectionnez **EntityFrameworkCodeFirst.sln** pour ouvrir l’exemple.

Pour exécuter l'exemple, vous devez créer trois bases de données vides dans Azure SQL Database :

* Une base de données du gestionnaire des cartes de partitions
* Une base de données nommée Shard 1
* Une base de données nommée Shard 2

Une fois ces bases de données créées, remplissez les espaces réservés dans le fichier **Program.cs** avec le nom de votre serveur, les noms des bases de données et vos informations d’identification pour vous connecter à ces dernières. Créez la solution dans Visual Studio. Visual Studio télécharge les packages NuGet nécessaires pour la bibliothèque cliente de base de données élastique, Entity Framework et la gestion des erreurs temporaires dans le cadre du processus de génération. Assurez-vous que la restauration des packages NuGet est activée pour votre solution. Vous pouvez activer ce paramètre en cliquant sur le fichier de la solution dans l'Explorateur de solutions Visual Studio.

## <a name="entity-framework-workflows"></a>Flux de travail Entity Framework

Les développeurs d'Entity Framework s'appuient sur l'un des quatre flux de travail suivants pour créer des applications et garantir la persistance pour les objets d'application :

* **Code First (Nouvelle base de données)**  : le développeur EF crée le modèle dans le code de l’application dont EF se sert pour générer la base de données.
* **Code First (Base de données existante)**  : le développeur laisse EF générer le code d’application du modèle à partir d’une base de données existante.
* **Model First**  : le développeur crée le modèle dans le concepteur EF, puis EF crée la base de données à partir de ce modèle.
* **Database First**  : le développeur utilise les outils EF pour déduire le modèle à partir d’une base de données existante.

Toutes ces approches s'appuient sur la classe DbContext pour gérer en toute transparence les connexions et le schéma de base de données d'une application. Les différents constructeurs de la classe de base DbContext permettent différents niveaux de contrôle sur la création de la connexion, l’amorçage de la base de données et la création du schéma. Les problèmes surviennent principalement en raison du fait que la gestion des connexions de base de données fournie par Entity Framework interfère avec les fonctionnalités de gestion des connexions des interfaces de routage dépendant des données fournies par la bibliothèque cliente de base de données élastique.

## <a name="elastic-database-tools-assumptions"></a>Hypothèses des outils de base de données élastique

Vous trouverez les définitions des termes évoqués ici dans la page [Glossaire des outils de base de données élastique](elastic-scale-glossary.md).

La bibliothèque cliente de base de données permet de définir des partitions pour les données de votre application. Ces partitions sont nommées shardlets. Les shardlets sont identifiés par une clé de partitionnement et sont mappés vers des bases de données spécifiques. Une application peut avoir autant de bases de données que nécessaire et distribuer les shardlets pour fournir suffisamment de capacité ou de performances selon les besoins de l'entreprise. Le mappage des valeurs de clé de partitionnement vers les bases de données est stocké par une carte de partitions fournie par les API clientes de la base de données élastique. Cette fonctionnalité s’appelle la **gestion des cartes de partitions** , ou GCP. La carte de partitions sert également de service Broker de connexion de base de données pour les demandes transportant une clé de partitionnement. Cette fonctionnalité est appelée **routage dépendant des données** .

Le gestionnaire des cartes de partitions empêche tout affichage incohérent des données shardlet pouvant perturber les utilisateurs lors des opérations de gestion de shardlet simultanées (par exemple, le déplacement des données d'une partition à l'autre). Pour ce faire, la partition gérée par la bibliothèque cliente mappe dans le service Broker les connexions de base de données pour une application. Ainsi, la fonctionnalité de carte de partitions peut automatiquement arrêter une connexion de base de données si des opérations de gestion de partition peuvent affecter le shardlet pour lequel la connexion a été créée. Cette approche doit s'intégrer à certaines fonctionnalités d'Entity Framework, telles que la création de connexions à partir d'une connexion existante pour vérifier l'existence de la base de données. Nous constatons qu'en général, les constructeurs DbContext standard fonctionnent uniquement de façon fiable pour les connexions de base de données fermées pouvant être clonées en toute sécurité pour Entity Framework. Le principe de conception de la base de données élastique consiste plutôt à utiliser uniquement le service Broker sur les connexions ouvertes. On pourrait penser que la fermeture d'une connexion demandée par la bibliothèque cliente avant son transfert vers le DbContext EF peut résoudre ce problème. Cependant, si l’on ferme la connexion et que l’on s’appuie sur Entity Framework pour la rouvrir, la bibliothèque n’effectue pas les contrôles de validation et de cohérence. Par contre, la fonctionnalité de migrations d'Entity Framework utilise ces connexions pour gérer le schéma de base de données sous-jacent de façon transparente pour l'application. L’idéal est de conserver et de combiner toutes ces fonctionnalités de la bibliothèque cliente de base de données élastique et d’Entity Framework dans la même application. La section suivante décrit en détail ces propriétés et les éléments requis.

## <a name="requirements"></a>Spécifications

Lors de l’utilisation des API de la bibliothèque cliente de base de données élastique et des API Entity Framework, vous souhaitez conserver les propriétés suivantes :

* **Scale-out**  : pour ajouter ou supprimer des bases de données de la couche Données de l’application partitionnée en fonction des besoins de capacité de l’application. Cela revient à contrôler la création et la suppression des bases de données et à utiliser les API du gestionnaire des cartes de partitions de la base de données élastique pour gérer les bases de données et les mappages des shardlets.
* **Cohérence** : l’application utilise le partitionnement et les fonctionnalités de routage dépendant des données de la bibliothèque cliente. Pour éviter d'obtenir des résultats de requêtes incorrects ou altérés, les connections sont demandées via le gestionnaire des cartes de partitions. Cela maintient également la validation et la cohérence.
* **Code First**  : pour conserver le côté pratique du paradigme Code First d’EF. Dans Code First, les classes de l'application sont mappées en toute transparence vers les structures de base de données sous-jacentes. Le code d’application interagit avec les propriétés DbSet qui masquent la plupart des aspects impliqués dans le traitement de base de données sous-jacent.
* **Schéma**  : Entity Framework gère la création initiale des schémas de base de données ainsi que leur évolution ultérieure au fil des migrations. En gardant ces capacités, il est facile d'adapter votre application à mesure que les données évoluent.

Le guide suivant indique comment répondre à ces impératifs pour les applications Code First à l'aide des outils de base de données élastique.

## <a name="data-dependent-routing-using-ef-dbcontext"></a>Routage dépendant des données avec la classe Entity Framework DbContext

Les connexions de base de données avec Entity Framework sont généralement gérées via des classes secondaires de **DbContext** . Créez ces sous-classes en procédant à une dérivation à partir de **DbContext** . À partir d’ici, vous allez définir vos **DbSets** qui implémentent les collections reposant sur la base de données d’objets CLR pour votre application. Dans le contexte du routage dépendant des données, vous pouvez identifier plusieurs propriétés utiles qui ne s’appliquent pas nécessairement à d’autres scénarios d’application Code First Entity Framework :

* La base de données existe déjà et a été enregistrée dans la carte de partitions de la base de données élastique.
* Le schéma de l’application a déjà été déployé vers la base de données (voir ci-après).
* Les connexions de routage dépendant des données vers la base de données sont demandées par la carte de partitions.

Pour intégrer **DbContexts** à un routage dépendant des données pour la montée en charge :

1. Créez des connexions de bases de données physiques via les interfaces clientes de base de données élastique du gestionnaire de cartes de partitions.
2. Encapsulez la connexion avec la sous-classe **DbContext**
3. Transmettez la connexion aux classes de base **DbContext** pour vous assurer que tous les traitements côté EF se produisent également.

L'exemple de code suivant illustre cette approche. Ce code est également contenu dans le projet Visual Studio qui accompagne cet article.

```csharp
public class ElasticScaleContext<T> : DbContext
{
public DbSet<Blog> Blogs { get; set; }
...

    // C'tor for data-dependent routing. This call opens a validated connection
    // routed to the proper shard by the shard map manager.
    // Note that the base class c'tor call fails for an open connection
    // if migrations need to be done and SQL credentials are used. This is the reason for the
    // separation of c'tors into the data-dependent routing case (this c'tor) and the internal c'tor for new shards.
    public ElasticScaleContext(ShardMap shardMap, T shardingKey, string connectionStr)
        : base(CreateDDRConnection(shardMap, shardingKey, connectionStr),
        true /* contextOwnsConnection */)
    {
    }

    // Only static methods are allowed in calls into base class c'tors.
    private static DbConnection CreateDDRConnection(
    ShardMap shardMap,
    T shardingKey,
    string connectionStr)
    {
        // No initialization
        Database.SetInitializer<ElasticScaleContext<T>>(null);

        // Ask shard map to broker a validated connection for the given key
        SqlConnection conn = shardMap.OpenConnectionForKey<T>
                            (shardingKey, connectionStr, ConnectionOptions.Validate);
        return conn;
    }
```

## <a name="main-points"></a>Points principaux

* Un nouveau constructeur remplace le constructeur par défaut dans la classe secondaire de DbContext
* Le nouveau constructeur accepte les arguments exigés pour le routage dépendant des données via la bibliothèque cliente de base de données élastique :
  
  * la carte de partitions pour accéder aux interfaces de routage dépendant des données,
  * la clé de partition pour identifier le shardlet,
  * une chaîne de connexion contenant les informations d’identification pour la connexion de routage dépendant des données vers la partition.
* L’appel au constructeur de classe de base prend un détour par une méthode statique qui effectue toutes les étapes nécessaires pour le routage dépendant des données.
  
  * Il utilise l’appel OpenConnectionForKey des interfaces clientes de base de données élastique sur la carte de partitions pour établir une connexion ouverte.
  * La carte de partitions crée la connexion ouverte vers la partition hébergeant le shardlet pour la clé de partitionnement donnée.
  * Cette connexion ouverte est renvoyée vers le constructeur de classe de base de DbContext pour indiquer que cette connexion doit être utilisée par EF au lieu de laisser EF créer une connexion automatiquement. Ainsi, la connexion a été marquée par l’API cliente de base de données élastique pour garantir la cohérence au cours des opérations de gestion de carte de partitions.

Utilisez le nouveau constructeur pour votre classe secondaire DbContext au lieu du constructeur par défaut dans votre code. Voici un exemple :

```csharp
// Create and save a new blog.

Console.Write("Enter a name for a new blog: ");
var name = Console.ReadLine();

using (var db = new ElasticScaleContext<int>(
                        sharding.ShardMap,  
                        tenantId1,  
                        connStrBldr.ConnectionString))
{
    var blog = new Blog { Name = name };
    db.Blogs.Add(blog);
    db.SaveChanges();

    // Display all Blogs for tenant 1
    var query = from b in db.Blogs
                orderby b.Name
                select b;
    …
}
```

Le nouveau constructeur ouvre la connexion vers la partition qui héberge les données pour le shardlet identifié par la valeur de **tenantid1** . Le code du bloc **using** ne change pas pour accéder au **DbSet** pour les blogs utilisant EF sur la partition pour **tenantid1** . Cela modifie la sémantique du code dans le bloc using de sorte que toutes les opérations de base de données sont adaptées à la seule partition où **tenantid1** est conservé. Par exemple, une requête LINQ sur les blogs **DbSet** ne renvoie que des blogs stockés sur la partition actuelle, mais pas ceux stockés sur d'autres partitions.  

### <a name="transient-faults-handling"></a>Gestion des erreurs temporaires

L’équipe d’aide de Microsoft a publié l’article [Bloc d’application de gestion des erreurs temporaires](/previous-versions/msp-n-p/dn440719(v=pandp.60)). La bibliothèque est utilisée avec la bibliothèque cliente d’infrastructure élastique conjointement à EF. Toutefois, assurez-vous que toute exception temporaire renvoie à un emplacement où vous pouvez garantir que le nouveau constructeur est utilisé après une erreur temporaire afin que toute nouvelle tentative de connexion soit effectuée à l’aide des constructeurs que vous avez modifiés. Sinon, une connexion à la partition appropriée n'est pas garantie et il n'est pas certain que la connexion soit maintenue, lorsque des modifications de la carte de partitions surviennent.

L'exemple de code suivant explique comment utiliser une stratégie de récupération SQL autour des nouveaux constructeurs de sous-classe **DbContext** :

```csharp
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() =>
{
    using (var db = new ElasticScaleContext<int>(
                            sharding.ShardMap,  
                            tenantId1,  
                            connStrBldr.ConnectionString))
        {
                var blog = new Blog { Name = name };
                db.Blogs.Add(blog);
                db.SaveChanges();
        …
        }
    });
```

**SqlDatabaseUtils.SqlRetryPolicy** du code ci-dessus est définie comme une **SqlDatabaseTransientErrorDetectionStrategy** avec un nombre de tentatives de 10 et un délai d’attente de 5 secondes entre chaque tentative. Cette approche est similaire aux conseils pour EF et les transactions initiées par l'utilisateur (voir [Limitations des nouvelles tentatives des stratégies d'exécution (à partir d'Entity Framework 6)](/ef/ef6/fundamentals/connection-resiliency/retry-logic). Les deux situations nécessitent que le programme d’application contrôle l’étendue dans laquelle l’exception transitoire retourne : afin de rouvrir la transaction ou (comme indiqué) de recréer le contexte à partir du constructeur approprié qui utilise les bibliothèques clientes de base de données élastique.

La nécessité de contrôler l'endroit où les exceptions temporaires nous amènent exclut également l'utilisation de la **SqlAzureExecutionStrategy** intégrée, fournie avec Entity Framework. **SqlAzureExecutionStrategy** rouvrirait une connexion, mais n’utiliserait pas **OpenConnectionForKey** , et passerait donc outre toute la validation effectuée au sein de l’appel **OpenConnectionForKey** . Au lieu de cela, l'exemple de code utilise la **DefaultExecutionStrategy** intégrée, également fournie avec Entity Framework. Contrairement à **SqlAzureExecutionStrategy** , elle fonctionne correctement avec la stratégie de nouvelle tentative de la gestion des erreurs temporaires. La stratégie d'exécution est définie dans la classe **ElasticScaleDbConfiguration** . Notez que nous avons décidé de ne pas utiliser **DefaultSqlExecutionStrategy** , car il propose d’utiliser **SqlAzureExecutionStrategy** en cas d’exceptions temporaires, ce qui entraînerait le comportement incorrect indiqué précédemment. Pour plus d’informations sur les différentes stratégies de nouvelle tentative et Entity Framework, consultez la rubrique [Résilience des connexions dans Entity Framework](/ef/ef6/fundamentals/connection-resiliency/retry-logic).

#### <a name="constructor-rewrites"></a>Réécritures de constructeur

Les exemples de code ci-dessus illustrent les réécritures de constructeur par défaut nécessaires pour votre application afin d’utiliser le routage dépendant des données avec Entity Framework. Le tableau suivant généralise cette approche aux autres constructeurs.

| Constructeur en cours | Constructeur réécrit pour les données | Constructeur de base | Notes |
| --- | --- | --- | --- |
| MyContext() |ElasticScaleContext(ShardMap, TKey) |DbContext(DbConnection, bool) |La connexion doit dépendre de la carte de partitions et de la clé de routage dépendant des données. Vous devez contourner la création de connexion automatique par EF pour utiliser la carte de partitions à la place pour répartir la connexion. |
| MyContext(string) |ElasticScaleContext(ShardMap, TKey) |DbContext(DbConnection, bool) |La connexion dépend de la carte de partitions et de la clé de routage dépendant des données. Un nom de base de données fixe ou une chaîne de connexion ne fonctionne pas, car ils contournent alors la validation par la carte de partitions. |
| MyContext(DbCompiledModel) |ElasticScaleContext(ShardMap, TKey, DbCompiledModel) |DbContext(DbConnection, DbCompiledModel, bool) |La connexion est créée avec le modèle fourni pour la carte de partitions et la clé de partitionnement données. Le modèle compilé est transmis au c’tor de base. |
| MyContext(DbConnection, bool) |ElasticScaleContext(ShardMap, TKey, bool) |DbContext(DbConnection, bool) |La connexion doit être déduite de la carte de partitions et de la clé. Elle ne peut pas être fournie comme entrée (sauf si cette entrée utilisait déjà la carte de partitions et la clé). La valeur booléenne est transmise. |
| MyContext(string, DbCompiledModel) |ElasticScaleContext(ShardMap, TKey, DbCompiledModel) |DbContext(DbConnection, DbCompiledModel, bool) |La connexion doit être déduite de la carte de partitions et de la clé. Elle ne peut pas être fournie comme entrée (sauf si cette entrée utilisait la carte de partitions et la clé). Le modèle compilé est transmis. |
| MyContext(ObjectContext, bool) |ElasticScaleContext(ShardMap, TKey, ObjectContext, bool) |DbContext(ObjectContext, bool) |Le nouveau constructeur doit s'assurer que toute connexion dans ObjectContext transmise en tant qu'entrée est réacheminée vers une connexion gérée par une infrastructure élastique. Ce document ne contient pas de présentation détaillée d’ObjectContext. |
| MyContext(DbConnection, DbCompiledModel, bool) |ElasticScaleContext(ShardMap, TKey, DbCompiledModel, bool) |DbContext(DbConnection, DbCompiledModel, bool); |La connexion doit être déduite de la carte de partitions et de la clé. Elle ne peut pas être fournie comme entrée (sauf si cette entrée utilisait déjà la carte de partitions et la clé). Les valeurs Model et Boolean sont transmises au constructeur de classe de base. |

## <a name="shard-schema-deployment-through-ef-migrations"></a>Déploiement de schéma de partition via des migrations Entity Framework

La gestion de schéma automatique est un avantage fourni par Entity Framework. Dans le contexte d’applications utilisant des outils de base de données élastique, vous souhaitez conserver cette fonctionnalité de provisionnement automatique du schéma pour les partitions récemment créées en cas d’ajout de bases de données à l’application partitionnée. Le cas d'usage principal correspond à l'augmentation de la capacité de la couche Données des applications partitionnées en utilisant EF. Le fait de s'appuyer sur les fonctionnalités d'Entity Framework pour la gestion des schémas permet de réduire l'effort d'administration de base de données grâce à une application partitionnée basée sur Entity Framework.

Le déploiement de schéma via des migrations EF fonctionne mieux sur des **connexions non ouvertes** . Ce comportement diffère de celui du scénario de routage dépendant des données qui s’appuie sur la connexion ouverte fournie par l’API cliente de la base de données élastique. Une autre différence se situe au niveau de l’exigence de cohérence : même s’il est tout indiqué de vérifier la cohérence pour toutes les connexions de routage dépendant des données afin de vous protéger contre la manipulation simultanée de cartes de partitions, ce problème ne concerne pas le déploiement de schéma initial vers une nouvelle base de données qui n’est pas encore inscrite dans la carte de partitions et qui n’a pas encore été allouée pour contenir des shardlets. Vous pouvez donc vous reposer sur des connexions de base de données standard pour ce scénario, et non sur le routage dépendant des données.  

Cela mène vers une approche où le déploiement de schéma via des migrations Entity Framework est étroitement lié à l'inscription de la nouvelle base de données en tant que partition dans la carte de partitions de l'application. Ceci nécessite les conditions préalables suivantes :

* La base de données a déjà été créée.
* La base de données est vide, elle ne contient ni schéma, ni données utilisateur.
* Il n’est pas encore possible d’accéder à la base de données via les API clientes de base de données élastique pour le routage dépendant des données.

Avec ces conditions préalables appliquées, vous pouvez créer un **SqlConnection** standard non ouvert pour lancer des migrations Entity Framework pour le déploiement de schéma. L'exemple de code suivant illustre cette approche.

```csharp
// Enter a new shard - i.e. an empty database - to the shard map, allocate a first tenant to it  
// and kick off EF initialization of the database to deploy schema

public void RegisterNewShard(string server, string database, string connStr, int key)
{

    Shard shard = this.ShardMap.CreateShard(new ShardLocation(server, database));

    SqlConnectionStringBuilder connStrBldr = new SqlConnectionStringBuilder(connStr);
    connStrBldr.DataSource = server;
    connStrBldr.InitialCatalog = database;

    // Go into a DbContext to trigger migrations and schema deployment for the new shard.
    // This requires an un-opened connection.
    using (var db = new ElasticScaleContext<int>(connStrBldr.ConnectionString))
    {
        // Run a query to engage EF migrations
        (from b in db.Blogs
            select b).Count();
    }

    // Register the mapping of the tenant to the shard in the shard map.
    // After this step, data-dependent routing on the shard map can be used

    this.ShardMap.CreatePointMapping(key, shard);
}
```

Cet exemple illustre la méthode **RegisterNewShard** qui enregistre la partition dans la carte de partitions, déploie le schéma via des migrations Entity Framework et stocke le mappage d'une clé de partitionnement pour la partition. Elle s’appuie sur un constructeur de la sous-classe **DbContext** ( **ElasticScaleContext** dans l’exemple) qui récupère une chaîne de connexion SQL en tant qu’entrée. Le code de ce constructeur est simple, comme le montre l'exemple suivant :

```csharp
// C'tor to deploy schema and migrations to a new shard
protected internal ElasticScaleContext(string connectionString)
    : base(SetInitializerForConnection(connectionString))
{
}

// Only static methods are allowed in calls into base class c'tors
private static string SetInitializerForConnection(string connectionString)
{
    // You want existence checks so that the schema can get deployed
    Database.SetInitializer<ElasticScaleContext<T>>(
new CreateDatabaseIfNotExists<ElasticScaleContext<T>>());

    return connectionString;
}
```

Vous avez peut-être utilisé la version du constructeur héritée de la classe de base. Mais le code doit garantir que l'initialiseur par défaut pour Entity Framework est utilisé lors de la connexion. D'où le bref détour par la méthode statique avant l'appel vers le constructeur de classe de base avec la chaîne de connexion. Notez que l'inscription des partitions doit s'exécuter dans un domaine d'application ou un processus différent pour vous assurer que les paramètres de l'initialiseur d'Entity Framework n'entrent pas en conflit.

## <a name="limitations"></a>Limites

Les approches décrites dans ce document entraînent quelques limitations :

* Les applications Entity Framework utilisant **LocalDb** en premier doivent migrer vers une base de données SQL Server standard avant d'utiliser la bibliothèque cliente de base de données élastique. La montée en charge d’une application via le partitionnement avec l’infrastructure élastique n’est pas possible avec **LocalDb** . Notez que le développement peut toujours utiliser **LocalDb** .
* Toutes les modifications apportées à l’application qui impliquent les modifications de schéma de base de données doivent passer par des migrations Entity Framework sur toutes les partitions. L'exemple de code pour ce document ne montre pas comment procéder. Envisagez d’utiliser la commande Update-Database avec un paramètre ConnectionString pour effectuer une itération sur toutes les partitions. Vous pouvez également extraire le script T-SQL pour la migration en attente à l’aide d’une commande Update-Database avec l’option -Script, puis appliquer le script T-SQL à vos partitions.  
* Nous partons du principe que tous les traitements de base de données d’une demande donnée sont contenus dans une seule partition, identifiée par la clé de partitionnement fournie par la demande. Cependant, cette hypothèse n'est pas toujours vraie. Par exemple, lorsqu'il n'est pas possible de proposer une clé de partitionnement. Pour résoudre ce problème, les bibliothèques clientes fournissent la classe **MultiShardQuery** qui implémente une abstraction de connexion pour l’interrogation sur plusieurs partitions. Ce document ne permet pas d'apprendre à utiliser la **MultiShardQuery** conjointement à Entity Framework.

## <a name="conclusion"></a>Conclusion

En suivant les procédures décrites dans ce document, les applications Entity Framework peuvent utiliser les fonctionnalités de la bibliothèque cliente de base de données élastique pour le routage dépendant des données en refactorisant des constructeurs des sous-classes **DbContext** utilisés dans l’application Entity Framework. Cela limite les modifications requises aux emplacements où les classes **DbContext** existent déjà. De plus, les applications Entity Framework peuvent continuer à bénéficier du déploiement de schéma automatique en combinant les procédures qui appellent les migrations Entity Framework nécessaires à l'enregistrement de nouvelles partitions et mappages dans la carte de partitions.

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-use-entity-framework-applications-visual-studio/sample.png