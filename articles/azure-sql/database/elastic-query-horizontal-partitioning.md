---
title: Créer des rapports sur des bases de données cloud montées en charge (scale out)
description: comment configurer des requêtes élastiques sur les partitions horizontales
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: MladjoA
ms.author: mlandzic
ms.reviewer: sstein
ms.date: 01/03/2019
ms.openlocfilehash: 148c4828309738a18dbda5fd35ea634e8384bfde
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792104"
---
# <a name="reporting-across-scaled-out-cloud-databases-preview"></a>Création de rapports sur des bases de données cloud mises à l’échelle (version préliminaire)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

![Requête sur plusieurs partitions][1]

Les bases de données partitionnées répartissent des lignes sur une mise à l’échelle vers la couche données. Le schéma est identique sur toutes les bases de données participantes, également connu sous le terme partitionnement horizontal. En utilisant une requête élastique, vous pouvez créer des rapports qui couvrent toutes les bases de données d’une base de données partitionnée.

Pour un démarrage rapide, consultez [Créer des rapports sur des bases de données cloud avec scale-out](elastic-query-getting-started.md).

Pour les bases de données non partitionnées, consultez [Interroger plusieurs bases de données cloud avec différents schémas](elastic-query-vertical-partitioning.md).

## <a name="prerequisites"></a>Prérequis

* Créez une carte de partitions à l’aide d’une bibliothèque de base de données élastique cliente. Consultez la rubrique [Gestion des cartes de partitions](elastic-scale-shard-map-management.md). Ou utilisez l’exemple d’application de la rubrique [Prise en main des outils de base de données élastiques](elastic-scale-get-started.md).
* Vous pouvez également consulter la rubrique [Migrer des bases de données existantes vers des bases de données mises à l’échelle](elastic-convert-to-use-elastic-tools.md).
* L’utilisateur doit posséder l’autorisation ALTER ANY EXTERNAL DATA SOURCE. Cette autorisation est incluse dans l’autorisation ALTER DATABASE.
* Les autorisations ALTER ANY EXTERNAL DATA SOURCE sont nécessaires pour faire référence à la source de données sous-jacente.

## <a name="overview"></a>Vue d’ensemble

Ces instructions créent une représentation des métadonnées de votre couche de données partitionnées dans la base de données de requête élastique.

1. [CREATE MASTER KEY](/sql/t-sql/statements/create-master-key-transact-sql)
2. [CREATE DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/create-database-scoped-credential-transact-sql)
3. [CREATE EXTERNAL DATA SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql)
4. [CREATE EXTERNAL TABLE](/sql/t-sql/statements/create-external-table-transact-sql)

## <a name="11-create-database-scoped-master-key-and-credentials"></a>1.1 Créer la clé principale et les informations d’identification de la base de données

Les informations d'identification sont utilisées par la requête élastique pour se connecter à vos bases de données distantes.  

```sql
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'password';
CREATE DATABASE SCOPED CREDENTIAL <credential_name>  WITH IDENTITY = '<username>',  
SECRET = '<password>'
[;]
```

> [!NOTE]
> Vérifiez que *« \<username\> »* ne contient pas le suffixe *« \@servername »* .

## <a name="12-create-external-data-sources"></a>1.2 Créer des sources de données externes

Syntaxe :

```sql
<External_Data_Source> ::=
    CREATE EXTERNAL DATA SOURCE <data_source_name> WITH
        (TYPE = SHARD_MAP_MANAGER,
                   LOCATION = '<fully_qualified_server_name>',
        DATABASE_NAME = ‘<shardmap_database_name>',
        CREDENTIAL = <credential_name>,
        SHARD_MAP_NAME = ‘<shardmapname>’
               ) [;]
```

### <a name="example"></a>Exemple

```sql
CREATE EXTERNAL DATA SOURCE MyExtSrc
WITH
(
    TYPE=SHARD_MAP_MANAGER,
    LOCATION='myserver.database.windows.net',
    DATABASE_NAME='ShardMapDatabase',
    CREDENTIAL= SMMUser,
    SHARD_MAP_NAME='ShardMap'
);
```

Récupérez la liste des sources de données externes actuelles :

```sql
select * from sys.external_data_sources;
```

La source de données externe fait référence au mappage de partitions. Une requête élastique utilise ensuite la source de données externe et le mappage de la partition sous-jacents pour énumérer les bases de données qui interviennent dans les couches de données.
Les mêmes informations d’identification sont utilisées pour lire la carte de partitions et accéder aux données présentes sur les partitions pendant le traitement d’une requête élastique.

## <a name="13-create-external-tables"></a>1.3 Créer des tables externes

Syntaxe :  

```sql
CREATE EXTERNAL TABLE [ database_name . [ schema_name ] . | schema_name. ] table_name  
    ( { <column_definition> } [ ,...n ])
    { WITH ( <sharded_external_table_options> ) }
) [;]  

<sharded_external_table_options> ::=
  DATA_SOURCE = <External_Data_Source>,
  [ SCHEMA_NAME = N'nonescaped_schema_name',]
  [ OBJECT_NAME = N'nonescaped_object_name',]
  DISTRIBUTION = SHARDED(<sharding_column_name>) | REPLICATED |ROUND_ROBIN
```

**Exemple**

```sql
CREATE EXTERNAL TABLE [dbo].[order_line](
     [ol_o_id] int NOT NULL,
     [ol_d_id] tinyint NOT NULL,
     [ol_w_id] int NOT NULL,
     [ol_number] tinyint NOT NULL,
     [ol_i_id] int NOT NULL,
     [ol_delivery_d] datetime NOT NULL,
     [ol_amount] smallmoney NOT NULL,
     [ol_supply_w_id] int NOT NULL,
     [ol_quantity] smallint NOT NULL,
      [ol_dist_info] char(24) NOT NULL
)

WITH
(
    DATA_SOURCE = MyExtSrc,
     SCHEMA_NAME = 'orders',
     OBJECT_NAME = 'order_details',
    DISTRIBUTION=SHARDED(ol_w_id)
);
```

Récupérer la liste des tables externes à partir de la base de données en cours :

```sql
SELECT * from sys.external_tables;
```

Pour supprimer des bases de données externes :

```sql
DROP EXTERNAL TABLE [ database_name . [ schema_name ] . | schema_name. ] table_name[;]
```

### <a name="remarks"></a>Notes

La clause DATA\_SOURCE définit la source de données externe (une carte de partitions dans le cas d’un partitionnement horizontal) qui est utilisée pour la table externe.  

Les clauses SCHEMA\_NAME et OBJECT\_NAME mappent la définition de table externe à une table dans un schéma différent. En cas d’omission, le schéma de l’objet distant est supposé de type « dbo » et son nom est supposé être identique au nom de la table externe en cours de définition. Ceci est particulièrement utile si le nom de votre table distante est déjà utilisé dans la base de données dans laquelle vous souhaitez créer la table externe. Par exemple, vous souhaitez définir une table externe pour obtenir une vue agrégée des affichages de catalogue ou de vues de gestion dynamiques sur la couche des données mise à l’échelle. Dans la mesure où les affichages catalogue et les vues de gestion dynamique existent déjà localement, vous ne pouvez pas utiliser leur nom pour la définition de la table externe. Vous devez utiliser un autre nom et le nom de la vue de catalogue ou de la vue de gestion dynamique dans les clauses SCHEMA\_NAME et/ou OBJECT\_NAME. (Voir l’exemple ci-dessous.)

La clause DISTRIBUTION spécifie la distribution des données utilisée pour cette table. Le processeur de requêtes utilise les informations fournies dans la clause DISTRIBUTION pour créer les plans de requête les plus efficaces.

1. **SHARDED** signifie que les données sont partitionnées horizontalement entre les bases de données. La clé de partitionnement pour la distribution des données figure dans le paramètre **<nom_colonne_partitionnement>** .
2. **REPLICATED** signifie que des copies identiques de la table sont présentes sur chaque base de données. La responsabilité de vous assurer que les réplicas sont identiques d’une base de données à l’autre vous incombe.
3. **ROUND\_ROBIN** signifie que la table est partitionnée horizontalement à l’aide d’une méthode de distribution liée à l’application.

**Référence de la couche Données**  : le DDL de table externe fait référence à une source de données externe. La source de données externe spécifie une carte de partitions qui fournit à la table externe les informations nécessaires à la localisation de toutes les bases de données de votre couche de données.

### <a name="security-considerations"></a>Considérations relatives à la sécurité

Les utilisateurs ayant accès à la table externe acquièrent un accès automatique aux tables distantes sous-jacentes avec les informations d’identification fournies dans la définition de source de données externe. Évitez une élévation de privilèges non souhaitée par le biais d’informations d'identification de la source de données externe. Utilisez GRANT ou REVOKE pour une table externe, comme s’il s’agissait d’une table standard.  

Une fois votre table externe et votre source de données externe définies, vous pouvez utiliser l’ensemble T-SQL complet sur vos tables externes.

## <a name="example-querying-horizontal-partitioned-databases"></a>Exemple : interrogation de bases de données partitionnées horizontales

La requête suivante effectue une jonction tridirectionnelle entre les entrepôts, les commandes et les lignes de commande, et utilise plusieurs agrégats et un filtre sélectif. Elle suppose (1) un partitionnement horizontal (sharding) et (2) que les entrepôts, les commandes et lignes de commande sont partitionnées par la colonne d’ID de l’entrepôt et que la requête élastique peut placer des jointures sur les partitions et traiter la partie coûteuse de la requête sur les partitions en parallèle.

```sql
    select  
         w_id as warehouse,
         o_c_id as customer,
         count(*) as cnt_orderline,
         max(ol_quantity) as max_quantity,
         avg(ol_amount) as avg_amount,
         min(ol_delivery_d) as min_deliv_date
    from warehouse
    join orders
    on w_id = o_w_id
    join order_line
    on o_id = ol_o_id and o_w_id = ol_w_id
    where w_id > 100 and w_id < 200
    group by w_id, o_c_id
```

## <a name="stored-procedure-for-remote-t-sql-execution-sp_execute_remote"></a>Procédure stockée pour l’exécution de T-SQL à distance : sp\_execute_remote

La requête élastique introduit également une procédure stockée qui offre un accès direct aux partitions. La procédure stockée est appelée [sp\_execute\_remote](/sql/relational-databases/system-stored-procedures/sp-execute-remote-azure-sql-database) et peut être utilisée pour exécuter le code T-SQL ou les procédures stockées distantes sur des bases de données distantes. Les paramètres suivants sont pris en compte :

* Nom de la source de données (nvarchar) : nom de la source de données externe de type SGBDR.
* Requête (nvarchar) : requête T-SQL à exécuter sur chaque partition.
* Déclaration de paramètre (nvarchar) - facultatif : chaîne contenant des définitions de type de données pour les paramètres utilisés dans le paramètre de requête (comme sp_executesql).
* Liste de valeurs de paramètre - facultatif : liste de valeurs de paramètre séparées par des virgules (comme sp_executesql).

sp\_execute\_remote utilise la source de données externe fournie dans les paramètres d’appel pour exécuter l’instruction T-SQL donnée sur toutes les bases de données distantes. Il utilise les informations d’identification de la source de données externe pour se connecter à la base de données shardmap et aux bases de données distantes.  

Exemple :

```sql
    EXEC sp_execute_remote
        N'MyExtSrc',
        N'select count(w_id) as foo from warehouse'
```

## <a name="connectivity-for-tools"></a>Connectivité des outils

Utilisez des chaînes de connexion SQL Server standard pour connecter votre application, votre décisionnel et vos outils d’intégration des données à la base de données avec vos définitions de table externe. Assurez-vous que SQL Server est pris en charge comme source de données pour votre outil. Référencez la base de données de requête élastique comme n’importe quelle autre base de données SQL Server connectée à l’outil et utilisez des tables externes à partir de votre outil ou votre application comme s’il s’agissait de tables locales.

## <a name="best-practices"></a>Meilleures pratiques

* Vérifiez que la base de données du point de terminaison de requête élastique est autorisée à accéder à la base de données de carte de partitions et à toutes les partitions via les pare-feu de base de données SQL.  
* Validez ou appliquez la distribution de données définie par la table externe. Si la distribution réelle des données est différente de la distribution spécifiée dans la définition de votre table, vos requêtes peuvent donner des résultats inattendus.
* La requête élastique n’effectue pas d’élimination de partition lorsque les prédicats de clé de partitionnement permettent d’exclure en toute sécurité certaines partitions du traitement.
* Une requête élastique est mieux adaptée aux requêtes dont la plus grande partie du calcul peut être effectuée sur les partitions. De manière générale, vous obtenez les meilleures performances de requête avec des prédicats de filtres sélectifs pouvant être évalués sur des partitions ou des jonctions via les clés de partitionnement qui peuvent être effectuées de manière alignée sur toutes les partitions. D’autres modèles de requête peuvent nécessiter le chargement de grandes quantités de données dans le nœud principal, à partir des partitions, ce qui peut nuire aux performances.

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation des requêtes élastiques, consultez [Présentation des requêtes élastiques](elastic-query-overview.md).
* Pour le didacticiel sur le partitionnement vertical, consultez [Prise en main des requêtes de bases de données croisées (partitionnement vertical)](elastic-query-getting-started-vertical.md).
* Pour la syntaxe et des exemples de requêtes pour des données partitionnées verticalement, consultez [Requêtes sur des données partitionnées verticalement](elastic-query-vertical-partitioning.md)
* Pour commencer le didacticiel sur le partitionnement horizontal (sharding), consultez [Prise en main des requêtes élastiques pour le partitionnement horizontal (sharding)](elastic-query-getting-started.md).
* Consultez [sp\_execute \_remote](/sql/relational-databases/system-stored-procedures/sp-execute-remote-azure-sql-database) pour une procédure stockée qui exécute une instruction Transact-SQL sur une seule base de données Azure SQL distante ou un ensemble de bases de données servant de partitions dans un schéma de partitionnement horizontal.

<!--Image references-->
[1]: ./media/elastic-query-horizontal-partitioning/horizontalpartitioning.png
<!--anchors-->