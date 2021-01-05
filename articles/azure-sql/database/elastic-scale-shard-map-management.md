---
title: Effectuer un scale-out d’une base de données
description: Utilisation de ShardMapManager, la bibliothèque cliente de base de données élastique
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: e23b94c850c6ec326c2f4ad034e1fefc158087a5
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92793447"
---
# <a name="scale-out-databases-with-the-shard-map-manager"></a>Effectuer un scale-out des bases de données avec le Gestionnaire de cartes de partitions
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Pour facilement effectuer un scale-out des bases de données sur Azure SQL Database, utilisez un Gestionnaire de cartes de partitions. Le Gestionnaire de cartes de partitions est une base de données spéciale qui gère les informations de mappage global sur toutes les partitions (bases de données) dans un ensemble de partitions. Les métadonnées permettent à une application de se connecter à la base de données qui convient en fonction de la valeur de la **clé de partitionnement** . En outre, chaque partition de l’ensemble contient les cartes qui suivent les données de partitions locales (appelées **shardlets** ).

![Gestion des cartes de partitions](./media/elastic-scale-shard-map-management/glossary.png)

Il est essentiel de comprendre comment ces mappages sont construits pour la gestion du mappage de partition. Utilisez pour cela la classe ShardMapManager ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager) située dans la [bibliothèque cliente de base de données élastique](elastic-database-client-library.md) pour gérer les cartes de partitions.  

## <a name="shard-maps-and-shard-mappings"></a>Cartes de partitions et mappages de partitions

Pour chaque partition, vous devez sélectionner le type de carte de partitions à créer. Votre choix dépend de l’architecture de la base de données :

1. Client unique par base de données  
2. Plusieurs clients par base de données (deux types) :
   1. Mappage de liste
   2. Mappage de plage

Pour un modèle monolocataire, créez une carte de partitions de **mappage de liste** . Le modèle à un seul client attribue une base de données par client. Il s’agit d’un modèle efficace pour les développeurs SaaS, car il simplifie la gestion.

![Mappage de liste][1]

Le modèle multi-locataire affecte plusieurs locataires à une seule base de données (et vous pouvez distribuer des groupes de locataires sur plusieurs bases de données). Utilisez ce modèle lorsque vous pensez que chaque client va avoir de faibles besoins en termes de données. Dans ce modèle, attribuez une plage de clients à une base de données à l’aide du **mappage de plage** .

![Mappage de plage][2]

Vous pouvez également implémenter un modèle de base de données multi-locataire à l’aide d’un *mappage de liste* pour affecter plusieurs locataires à une base de données individuelle. Par exemple, DB1 est utilisée pour stocker les informations d’ID client 1 et 5 et DB2 stocke les données pour les clients 7 et 10.

![Plusieurs clients sur une base de données unique][3]

### <a name="supported-types-for-sharding-keys"></a>Types pris en charge pour les clés de partitionnement

L’infrastructure élastique prend en charge les types suivants en tant que clés de partitionnement :

| .NET | Java |
| --- | --- |
| entier |entier |
| long |long |
| guid |uuid |
| byte[]  |byte[] |
| DATETIME | timestamp |
| intervalle de temps | duration|
| datetimeoffset |offsetdatetime |

### <a name="list-and-range-shard-maps"></a>Liste et cartes de partitions de plage

Vous pouvez construire des cartes de partition en utilisant des **listes de valeurs de clés de partitionnement individuelles** ou des **plages de valeurs de clés de partitionnement** .

### <a name="list-shard-maps"></a>Cartes de partition de liste

Les **partitions** contiennent des **shardlets** (micro-partitions) et le mappage de ces shardlets en partitions est tenu à jour par une carte de partitions. Une **carte de partitions de liste** est une association entre les valeurs de clés individuelles identifiant les shardlets et les bases de données qui servent de partitions.  **mappages de liste** sont explicites et plusieurs valeurs de clés peuvent être mappées à la même base de données. Par exemple, la valeur de clé 1 est mappée vers la base de données A, et les valeurs de clés 3 et 6 sont toutes deux mappées vers la base de données B.

| Clé | Emplacement de partition |
| --- | --- |
| 1 |Database_A |
| 3 |Database_B |
| 4 |Database_C |
| 6 |Database_B |
| ... |... |

### <a name="range-shard-maps"></a>Cartes de partitions de plage

Dans une **carte de partitions de plage** , la plage de clé est décrite par une paire **[valeur inférieure, valeur supérieure)** , où la *valeur inférieure* correspond à la clé minimale de la plage, tandis que la *valeur supérieure* correspond à la première valeur supérieure à la plage.

Par exemple, **[0, 100)** inclut tous les entiers égaux ou supérieurs à 0 et inférieurs à 100. Notez que plusieurs plages peuvent pointer vers la même base de données et que les plages disjointes sont prises en charge (par exemple, [100,200) et [400,600) pointent vers la base de données C dans l’exemple suivant.)

| Clé | Emplacement de partition |
| --- | --- |
| [1,50) |Database_A |
| [50,100) |Database_B |
| [100,200) |Database_C |
| [400,600) |Database_C |
| ... |... |

Chacune des tables présentées ci-dessus correspond à un exemple conceptuel d'un objet **ShardMap** . Chaque ligne correspond à un exemple simplifié d’un objet **PointMapping** (pour la carte de partitions de liste) ou **RangeMapping** (pour la carte de partitions de plage).

## <a name="shard-map-manager"></a>Gestionnaire des cartes de partitions

Dans la bibliothèque cliente, le gestionnaire des cartes de partitions correspond à une collection de cartes de partitions. Les données gérées par une instance **ShardMapManager** sont conservées à trois emplacements :

1. **Carte de partitions globale (GSM)**  : vous spécifiez une base de données en tant que référentiel pour l’ensemble de ses mappages et cartes de partitions. Les tables spéciales et les procédures stockées sont automatiquement créées pour gérer les informations. Il s'agit généralement d'une petite base de données à laquelle vous pouvez accéder rapidement, et elle ne doit pas être utilisée pour d'autres besoins de l'application. Les tables sont stockées dans un schéma spécifique nommé **__ShardManagement** .
2. **Carte de partitions locale (LSM)**  : chaque base de données que vous définissez en tant que partition est modifiée pour contenir plusieurs petites tables et des procédures stockées spéciales qui contiennent et gèrent les informations de carte de partitions propres à cette partition. Ces informations sont redondantes avec les informations contenues dans la GSM et elles permettent aux applications de valider les informations de carte de partitions mises en cache sans placer aucune charge sur la GSM. L'application utilise la LSM pour déterminer la validité d'un mappage mis en cache. Les tables correspondant à la LSM de chaque partition sont également stockées dans le schéma **__ShardManagement** .
3. **Cache d’application**  : chaque instance d’application accédant à un objet **ShardMapManager** conserve un cache local en mémoire de ses mappages. Elle stocke les informations de routage récupérées récemment.

## <a name="constructing-a-shardmapmanager"></a>Construction d'un objet ShardMapManager

Un objet **ShardMapManager** est construit à l’aide d’un modèle factory ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanagerfactory), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory)). La méthode **ShardMapManagerFactory.GetSqlShardMapManager** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanagerfactory.getsqlshardmapmanager), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager)) récupère des informations d’identification (incluant le nom du serveur et le nom de base de données contenant la GSM) sous la forme d’un objet **ConnectionString** et retourne l’instance d’un objet **ShardMapManager** .  

**Remaque :** l’objet **ShardMapManager** doit être instancié une seule fois par domaine d’application, dans le code d’initialisation d’une application. La création d’instances supplémentaires de ShardMapManager dans le même domaine d’application entraîne une augmentation de l’utilisation de la mémoire et du processeur par l’application. Un **ShardMapManager** peut contenir n’importe quel nombre de cartes de partitions. Si une carte de partitions peut suffire à de nombreuses applications, il arrive que différents ensembles de bases de données soient utilisés pour différents schémas ou pour des objectifs uniques. Lorsque cela se produit, il est préférable d'employer plusieurs cartes de partitions.

Dans ce code, une application tente d’ouvrir un objet **ShardMapManager** existant avec la méthode TryGetSqlShardMapManager ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanagerfactory.trygetsqlshardmapmanager), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager). Si aucun objet représentant un **ShardMapManager** (GSM) global ne se trouve dans la base de données, la bibliothèque cliente les crée à l’aide de la méthode CreateSqlShardMapManager ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanagerfactory.createsqlshardmapmanager), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.createsqlshardmapmanager)).

```Java
// Try to get a reference to the Shard Map Manager in the shardMapManager database.
// If it doesn't already exist, then create it.
ShardMapManager shardMapManager = null;
boolean shardMapManagerExists = ShardMapManagerFactory.tryGetSqlShardMapManager(shardMapManagerConnectionString,ShardMapManagerLoadPolicy.Lazy, refShardMapManager);
shardMapManager = refShardMapManager.argValue;

if (shardMapManagerExists) {
    ConsoleUtils.writeInfo("Shard Map %s already exists", shardMapManager);
}
else {
    // The Shard Map Manager does not exist, so create it
    shardMapManager = ShardMapManagerFactory.createSqlShardMapManager(shardMapManagerConnectionString);
    ConsoleUtils.writeInfo("Created Shard Map %s", shardMapManager);
}
```

```csharp
// Try to get a reference to the Shard Map Manager via the Shard Map Manager database.  
// If it doesn't already exist, then create it.
ShardMapManager shardMapManager;
bool shardMapManagerExists = ShardMapManagerFactory.TryGetSqlShardMapManager(
                                        connectionString,
                                        ShardMapManagerLoadPolicy.Lazy,
                                        out shardMapManager);

if (shardMapManagerExists)
{
    Console.WriteLine("Shard Map Manager already exists");
}
else
{
    // Create the Shard Map Manager.
    ShardMapManagerFactory.CreateSqlShardMapManager(connectionString);
    Console.WriteLine("Created SqlShardMapManager");

    shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager(
            connectionString,
            ShardMapManagerLoadPolicy.Lazy);

// The connectionString contains server name, database name, and admin credentials for privileges on both the GSM and the shards themselves.
}
```

Pour la version .NET, vous pouvez utiliser PowerShell pour créer un Gestionnaire de cartes de partitions. Un exemple est disponible [ici](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db).

## <a name="get-a-rangeshardmap-or-listshardmap"></a>Obtenir un objet RangeShardMap ou ListShardMap

Après avoir créé un gestionnaire de cartes de partitions, vous pouvez obtenir RangeShardMap ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) ou ListShardMap ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.listshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.listshardmap-1)) à l’aide de la méthode TryGetRangeShardMap ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.trygetrangeshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetrangeshardmap)), TryGetListShardMap ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.trygetlistshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetlistshardmap)), ou GetShardMap ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.getshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.getshardmap)).

```Java
// Creates a new Range Shard Map with the specified name, or gets the Range Shard Map if it already exists.
static <T> RangeShardMap<T> createOrGetRangeShardMap(ShardMapManager shardMapManager,
            String shardMapName,
            ShardKeyType keyType) {
    // Try to get a reference to the Shard Map.
    ReferenceObjectHelper<RangeShardMap<T>> refRangeShardMap = new ReferenceObjectHelper<>(null);
    boolean isGetSuccess = shardMapManager.tryGetRangeShardMap(shardMapName, keyType, refRangeShardMap);
    RangeShardMap<T> shardMap = refRangeShardMap.argValue;

    if (isGetSuccess && shardMap != null) {
        ConsoleUtils.writeInfo("Shard Map %1$s already exists", shardMap.getName());
    }
    else {
        // The Shard Map does not exist, so create it
        try {
            shardMap = shardMapManager.createRangeShardMap(shardMapName, keyType);
        }
        catch (Exception e) {
            e.printStackTrace();
        }
        ConsoleUtils.writeInfo("Created Shard Map %1$s", shardMap.getName());
    }

    return shardMap;
}
```

```csharp
// Creates a new Range Shard Map with the specified name, or gets the Range Shard Map if it already exists.
public static RangeShardMap<T> CreateOrGetRangeShardMap<T>(ShardMapManager shardMapManager, string shardMapName)
{
    // Try to get a reference to the Shard Map.
    RangeShardMap<T> shardMap;
    bool shardMapExists = shardMapManager.TryGetRangeShardMap(shardMapName, out shardMap);

    if (shardMapExists)
    {
        ConsoleUtils.WriteInfo("Shard Map {0} already exists", shardMap.Name);
    }
    else
    {
        // The Shard Map does not exist, so create it
        shardMap = shardMapManager.CreateRangeShardMap<T>(shardMapName);
        ConsoleUtils.WriteInfo("Created Shard Map {0}", shardMap.Name);
    }

    return shardMap;
}
```

### <a name="shard-map-administration-credentials"></a>Informations d'identification de l'administration de carte de partitions

Les applications d'administration et de manipulation de cartes de partitions diffèrent des applications acheminant des connexions grâce aux cartes de partitions.

Pour administrer des cartes de partitions (ajouter ou modifier des partitions, cartes de partitions, mappages de partitions, etc.), vous devez instancier l’objet **ShardMapManager** en utilisant des **informations d’identification dotées de privilèges de lecture/écriture sur la base de données GSM et sur chaque base de données servant de partition** . Les informations d'identification doivent autoriser l'écriture sur les tables dans la CPG et la CPL à chaque saisie ou modification des informations de carte de partitions, ainsi qu'à chaque création d'une table CPL sur une nouvelle partition.  

Consultez [Informations d’identification utilisées pour accéder à la bibliothèque cliente de la base de données élastique](elastic-scale-manage-credentials.md).

### <a name="only-metadata-affected"></a>Seules les métadonnées sont affectées

Les méthodes utilisées pour remplir ou modifier les données d'un objet **ShardMapManager** n'affectent pas les données utilisateur stockées dans les partitions. Par exemple, des méthodes telles que **CreateShard** , **DeleteShard** ou **UpdateMapping** affectent uniquement les métadonnées de carte de partitions. affectent uniquement les métadonnées de carte de partitions. Elles ne suppriment pas, n’ajoutent pas ou ni ne modifient les données utilisateur contenues dans les partitions.  (L’outil de **fractionnement/fusion** inclus avec les outils de base de données élastique utilise ces applications avec l’orchestration des mouvements de données réels entre les partitions.) Consultez [Mise à l’échelle utilisant l’outil de fractionnement et de fusion de bases de données élastiques](elastic-scale-overview-split-and-merge.md).

## <a name="data-dependent-routing"></a>Routage dépendant des données

Le Gestionnaire de cartes de partitions est utilisé dans les applications devant se connecter à une base de données pour exploiter leurs données. Ces connexions doivent être associées à la base de données correcte. Cette opération est nommée **Routage dépendant des données** . Pour ces applications, instanciez un objet Gestionnaire des cartes de partitions à partir de la fabrique à l'aide des informations d'identification ayant un accès en lecture seule sur la base de données CPG. Les requêtes de connexion individuelles ultérieures fournissent les informations d'identification requises pour la connexion à la base de données de partitions adéquate.

Ces applications (utilisant **ShardMapManager** ouvert avec des informations d’identification en lecture seule) ne peuvent pas modifier les cartes ni les mappages. Pour cela, vous devez créer des applications d'administration ou des scripts PowerShell qui fourniront des informations d'identification dotées de privilèges élevés comme indiqué précédemment. Consultez [Informations d’identification utilisées pour accéder à la bibliothèque cliente de la base de données élastique](elastic-scale-manage-credentials.md).

Pour plus d’informations, consultez [Routage dépendant des données](elastic-scale-data-dependent-routing.md).

## <a name="modifying-a-shard-map"></a>Modification d'une carte de partitions

Vous disposez de plusieurs méthodes pour modifier une carte de partitions. Toutes les méthodes suivantes modifient les métadonnées décrivant les partitions et leurs mappages, mais elles ne modifient pas physiquement les données dans les partitions et ne créent ni ne suppriment les bases de données réelles.  Il est possible que certaines opérations de carte de partitions décrites ci-dessous requièrent une coordination à l’aide d’actions d’administration qui déplacent physiquement les données ou qui ajoutent et suppriment des bases de données servant de partitions.

Ces méthodes fonctionnent ensemble en tant que blocs de construction disponibles pour la modification de la distribution globale des données dans votre environnement de base de données partitionnée.  

* Pour ajouter ou supprimer des partitions : utilisez les méthodes **CreateShard** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.shardmap.createshard), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.createshard)) et **DeleteShard** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.shardmap.deleteshard), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.deleteshard)) de la classe Shardmap ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.shardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap)).
  
    Le serveur et la base de données représentant la partition cible doivent déjà exister pour pouvoir exécuter ces opérations. Ces méthodes n’ont pas d’incidence sur les bases de données elles-mêmes. Elles affectent uniquement les métadonnées de la carte de partitions.
* Pour créer ou supprimer des points ou des plages qui sont mappés aux partitions, utilisez les méthodes **CreateRangeMapping** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.createrangemapping), [.NET](/previous-versions/azure/dn841993(v=azure.100))) et **DeleteMapping** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.deletemapping), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) de la classe RangeShardMapping ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)), et la méthode **CreatePointMapping** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.listshardmap.createpointmapping), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.listshardmap-1)) de la classe ListShardMap ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.listshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.listshardmap-1)).
  
    Vous pouvez mapper différents points ou plages vers la même partition. Ces méthodes affectent uniquement les métadonnées. Elles n’affectent pas les données qui peuvent déjà être présentes dans les partitions. Si vous devez supprimer des données de la base de données par souci de cohérence avec les opérations **DeleteMapping** , effectuez ces opérations séparément, mais en utilisant ces méthodes.  
* Pour fractionner des plages existantes en deux ou pour fusionner des plages adjacentes en une seule : utilisez **SplitMapping** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.splitmapping), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) et **MergeMappings** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.mergemappings), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)).  
  
    Notez que les opérations de fractionnement/fusion **ne changent pas la partition sur laquelle les valeurs de clé sont mappées** . Un fractionnement divise une plage existante en deux parties, qui sont chacune mappées vers la même partition. Une fusion s'applique sur deux plages adjacentes qui sont déjà mappées vers la même partition, afin d'en faire une plage unique.  Le déplacement des points ou des plages entre des partitions doit être coordonné à l’aide de l’objet **UpdateMapping** , conjointement au déplacement des données réelles.  Vous pouvez utiliser le service de **fractionnement/fusion** , compris dans les outils de base de données élastique, pour coordonner les modifications de la carte de partitions avec le mouvement des données, lorsque celui-ci est nécessaire.
* Pour remapper (ou déplacer) des points ou des plages vers différentes partitions, utilisez **UpdateMapping** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.updatemapping), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)).  
  
    Comme il est possible que des données doivent être déplacées d’une partition à une autre afin d’être cohérentes avec les opérations **UpdateMapping** , vous devez effectuer ces déplacements séparément tout en utilisant ces méthodes.

* Pour mettre des mappages en ligne et hors connexion, utilisez **MarkMappingOffline** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.markmappingoffline), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) et **MarkMappingOnline** ( [Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.markmappingonline), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) afin de contrôler l’état en ligne d’un mappage.
  
    Certaines opérations des mappages de partitions sont autorisées uniquement lorsque l’état d’un mappage est « hors connexion », notamment **UpdateMapping** et **DeleteMapping** . Quand un mappage est hors connexion, une requête dépendant des données basée sur une clé incluse dans ce mappage retourne une erreur. En outre, lorsqu'une plage est d'abord mise hors connexion, toutes les connexions vers la partition concernée sont supprimées automatiquement afin d'éviter des résultats incohérents ou incomplets pour les requêtes émises vers les plages en cours de modification.

Les mappages sont des objets immuables dans .NET.  Toutes les méthodes ci-dessus qui modifient les mappages invalident également les références à ceux-ci dans votre code. Pour faciliter l’exécution des séquences d'opérations qui modifient l'état d'un mappage, toutes les méthodes qui modifient un mappage renvoient une nouvelle référence de mappage, permettant aux opérations d’être chaînées. Par exemple, pour supprimer un mappage existant dans shardmap sm qui contient la clé 25, vous pouvez exécuter les éléments suivants :

```
    sm.DeleteMapping(sm.MarkMappingOffline(sm.GetMappingForKey(25)));
```

## <a name="adding-a-shard"></a>Ajout d'une partition

Les applications ont souvent besoin d’ajouter de nouvelles partitions pour gérer des données attendues de nouvelles clés ou plages de clés, pour une carte de partitions qui existe déjà. Par exemple, une application partitionnée par un ID de client peut requérir l'approvisionnement d'une nouvelle partition pour un nouveau client, ou des données partitionnées mensuellement peuvent requérir l'approvisionnement d'une nouvelle partition avant le début de chaque mois.

Si la nouvelle plage de valeurs de clé n’appartient pas déjà à un mappage existant et qu’aucun déplacement de données n’est nécessaire, il est facile d’ajouter la nouvelle partition et d’associer la nouvelle clé ou plage à cette partition. Pour plus d’informations sur l’ajout de nouvelles partitions, consultez [Ajout d’une nouvelle partition](elastic-scale-add-a-shard.md).

Cependant, pour les scénarios requérant le déplacement de données, l’outil de fusion/fractionnement est requis pour orchestrer le déplacement des données entre les partitions conjointement aux mises à jour nécessaires de la carte de partitions. Pour plus d’informations sur l’utilisation de l’outil de fractionnement/fusion, consultez [Présentation du service de fractionnement/fusion](elastic-scale-overview-split-and-merge.md).

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/elastic-scale-shard-map-management/listmapping.png
[2]: ./media/elastic-scale-shard-map-management/rangemapping.png
[3]: ./media/elastic-scale-shard-map-management/multipleonsingledb.png