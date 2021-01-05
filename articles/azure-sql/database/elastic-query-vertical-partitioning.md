---
title: Interroger plusieurs bases de données cloud de schémas différents
description: configuration de requêtes de bases de données croisées sur les partitions verticales
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: MladjoA
ms.author: mlandzic
ms.reviewer: sstein
ms.date: 01/25/2019
ms.openlocfilehash: c507a4c618713ba83d25b9defa918092db1a3c8e
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792087"
---
# <a name="query-across-cloud-databases-with-different-schemas-preview"></a>Interroger des bases de données cloud de schémas différents (version préliminaire)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

![Requête sur plusieurs tables dans des bases de données différentes][1]

Les bases de données partitionnées verticalement utilisent différents ensembles de tables sur différentes bases de données. Cela signifie que le schéma est différent sur des bases de données différentes. Par exemple, toutes les tables d’inventaire se trouvent sur une base de données alors que toutes les tables liées à la comptabilité se trouvent dans une seconde base de données.

## <a name="prerequisites"></a>Prérequis

* L’utilisateur doit posséder l’autorisation ALTER ANY EXTERNAL DATA SOURCE. Cette autorisation est incluse dans l’autorisation ALTER DATABASE.
* Les autorisations ALTER ANY EXTERNAL DATA SOURCE sont nécessaires pour faire référence à la source de données sous-jacente.

## <a name="overview"></a>Vue d’ensemble

> [!NOTE]
> Contrairement au partitionnement horizontal, ces instructions DDL ne dépendent pas de la définition d’une couche de données avec un mappage de partition via la bibliothèque client de base de données élastique.
>

1. [CREATE MASTER KEY](/sql/t-sql/statements/create-master-key-transact-sql)
2. [CREATE DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/create-database-scoped-credential-transact-sql)
3. [CREATE EXTERNAL DATA SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql)
4. [CREATE EXTERNAL TABLE](/sql/t-sql/statements/create-external-table-transact-sql)

## <a name="create-database-scoped-master-key-and-credentials"></a>Créer la clé principale et les informations d’identification de la base de données

Les informations d'identification sont utilisées par la requête élastique pour se connecter à vos bases de données distantes.  

```sql
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'master_key_password';
CREATE DATABASE SCOPED CREDENTIAL <credential_name>  WITH IDENTITY = '<username>',  
SECRET = '<password>'
[;]
```

> [!NOTE]
> Veillez à ce que `<username>` ne contienne pas de suffixe **"\@servername"** .

## <a name="create-external-data-sources"></a>Créer des sources de données externes

Syntaxe :

<External_Data_Source> ::= CREATE EXTERNAL DATA SOURCE <data_source_name> WITH (TYPE = RDBMS, LOCATION = ’<fully_qualified_server_name>’, DATABASE_NAME = ‘<remote_database_name>’,  
    CREDENTIAL = <credential_name> ) [;]

> [!IMPORTANT]
> Le paramètre TYPE doit être défini sur **RDBMS** .

### <a name="example"></a>Exemple

L’exemple suivant illustre l’utilisation de l’instruction CREATE pour les sources de données externes.

```sql
CREATE EXTERNAL DATA SOURCE RemoteReferenceData
   WITH
      (
         TYPE=RDBMS,
         LOCATION='myserver.database.windows.net',
         DATABASE_NAME='ReferenceData',
         CREDENTIAL= SqlUser
      );
```

Pour récupérer la liste des sources de données externes actuelles :

```sql
select * from sys.external_data_sources;
```

### <a name="external-tables"></a>Tables externes

Syntaxe :

CREATE EXTERNAL TABLE [ database_name . [ schema_name ] . | schema_name . ] table_name  
    ( { <column_definition> } [ ,...n ]) { WITH ( <rdbms_external_table_options> ) } )[;]

<rdbms_external_table_options> ::= DATA_SOURCE = <External_Data_Source>, [ SCHEMA_NAME = N'nonescaped_schema_name',] [ OBJECT_NAME = N'nonescaped_object_name',]

### <a name="example"></a>Exemple

```sql
CREATE EXTERNAL TABLE [dbo].[customer]
   (
      [c_id] int NOT NULL,
      [c_firstname] nvarchar(256) NULL,
      [c_lastname] nvarchar(256) NOT NULL,
      [street] nvarchar(256) NOT NULL,
      [city] nvarchar(256) NOT NULL,
      [state] nvarchar(20) NULL,
      DATA_SOURCE = RemoteReferenceData
   );
```

L’exemple suivant illustre comment récupérer la liste des tables externes à partir de la base de données en cours :

```sql
select * from sys.external_tables;
```

### <a name="remarks"></a>Notes

La requête élastique étend la syntaxe de la table externe existante pour définir des tables externes qui utilisent des sources de données externes de type SGBDR. Une définition de table externe pour le partitionnement vertical couvre les aspects suivants :

* **Schéma**  : le DDL de table externe définit un schéma que vos requêtes peuvent utiliser. Le schéma fourni dans votre définition de la table externe doit correspondre au schéma des tables appartenant à la base de données externe sur lesquelles sont stockées les données réelles.
* **Référence de base de données distante**  : le DDL de table externe fait référence à une source de données externe. La source de données externe spécifie le nom du serveur et le nom de la base de données distante dans lesquels sont stockées les données réelles du tableau.

La syntaxe permettant de créer des tables externes à l’aide de sources de données externes comme indiqué dans la section précédente est la suivante :

La clause DATA_SOURCE définit la source de données externe (par exemple, la base de données distante en cas de partitionnement horizontal) utilisée pour la table externe.  

Les clauses SCHEMA_NAME et OBJECT_NAME offrent la possibilité de mapper une définition de table externe sur une table dans un autre schéma base de données, ou sur une table portant un autre nom, respectivement. Cela s’avère utile si vous souhaitez définir une table externe sur une vue de catalogue ou une vue de gestion dynamique sur votre base de données distante, ou dans toute autre situation dans laquelle le nom de table distant est déjà utilisé en local.  

L’instruction DDL suivante supprime une définition de table externe existante du catalogue local. Elle n’affecte pas la base de données distante.

```sql
DROP EXTERNAL TABLE [ [ schema_name ] . | schema_name. ] table_name[;]  
```

**Autorisations pour CREATE/DROP EXTERNAL TABLE**  : les autorisations ALTER ANY EXTERNAL DATA SOURCE sont nécessaires au DDL de table externe, lequel est aussi nécessaire pour faire référence à la source de données sous-jacente.  

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Les utilisateurs ayant accès à la table externe acquièrent un accès automatique aux tables distantes sous-jacentes avec les informations d’identification fournies dans la définition de source de données externe. Vous devez gérer l’accès à la table externe avec beaucoup d’attention pour éviter une élévation de privilèges non souhaitée par le biais d’informations d’identification de la source de données externe. Les autorisations SQL standard permettent de GRANT (OCTROYER) ou de REVOKE (RÉVOQUER) l’accès à une table externe comme s’il s’agissait d’une table normale.  

## <a name="example-querying-vertically-partitioned-databases"></a>Exemple : interrogation de bases de données partitionnées verticalement

La requête suivante effectue une jointure tridirectionnelle entre les deux tables locales pour les commandes, et les lignes de commande et la table distante pour les clients. Voici un exemple de cas d’utilisation de données de référence pour requête élastique :

```sql
    SELECT
     c_id as customer,
     c_lastname as customer_name,
     count(*) as cnt_orderline,
     max(ol_quantity) as max_quantity,
     avg(ol_amount) as avg_amount,
     min(ol_delivery_d) as min_deliv_date
    FROM customer
    JOIN orders
    ON c_id = o_c_id
    JOIN  order_line
    ON o_id = ol_o_id and o_c_id = ol_c_id
    WHERE c_id = 100
```

## <a name="stored-procedure-for-remote-t-sql-execution-sp_execute_remote"></a>Procédure stockée pour l’exécution de T-SQL à distance : sp\_execute_remote

La requête élastique introduit également une procédure stockée qui offre un accès direct à la base de données distante. La procédure stockée est appelée [sp\_execute\_remote](/sql/relational-databases/system-stored-procedures/sp-execute-remote-azure-sql-database) et peut être utilisée pour exécuter le code T-SQL ou les procédures stockées distantes sur la base de données distante. Les paramètres suivants sont pris en compte :

* Nom de la source de données (nvarchar) : nom de la source de données externe de type SGBDR.
* Requête (nvarchar) : requête T-SQL à exécuter sur la base de données distante.
* Déclaration de paramètre (nvarchar) - facultatif : chaîne contenant des définitions de type de données pour les paramètres utilisés dans le paramètre de requête (comme sp_executesql).
* Liste de valeurs de paramètre - facultatif : liste de valeurs de paramètre séparées par des virgules (comme sp_executesql).

sp\_execute\_remote utilise la source de données externe fournie dans les paramètres d’appel pour exécuter l’instruction T-SQL donnée sur la base de données distante. Il utilise les informations d’identification de la source de données externe pour se connecter à la base de données distante.  

Exemple :

```sql
    EXEC sp_execute_remote
        N'MyExtSrc',
        N'select count(w_id) as foo from warehouse'
```

## <a name="connectivity-for-tools"></a>Connectivité des outils

Vous pouvez utiliser des chaînes de connexion SQL Server standard pour connecter votre BI et vos outils décisionnels et d’intégration de données aux bases de données sur le serveur dont la requête élastique est activée et les tables externes définies. Assurez-vous que SQL Server est pris en charge comme source de données pour votre outil. Reportez-vous ensuite à la requête de base de données élastique et à ses tables externes comme s’il s’agissait de n’importe quelle autre base de données SQL Server à laquelle vous vous connectez avec votre outil.

## <a name="best-practices"></a>Meilleures pratiques

* Assurez-vous que la base de données du point de terminaison de requête élastique est autorisée à accéder à la base de données distante en autorisant l’accès des services Azure dans sa configuration de pare-feu Azure SQL Database. Vérifiez également que les informations d’identification fournies dans la définition de source de données externe peuvent se connecter à la base de données distante et qu’elles bénéficient des autorisations d’accès à la table distante.  
* Une requête élastique est mieux adaptée aux requêtes dont la plus grande partie du calcul peut être effectuée sur les bases de données distantes. De manière générale, vous obtenez les meilleures performances de requête avec des prédicats de filtres sélectifs pouvant être évalués sur les bases de données ou des jointures distantes pouvant être exécutées en totalité sur la base de données distante. D’autres modèles de requête peuvent nécessiter le chargement de grandes quantités de données de la base de données distante et s’exécuter de façon médiocre.

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation des requêtes élastiques, consultez [Présentation des requêtes élastiques](elastic-query-overview.md).
* Pour le didacticiel sur le partitionnement vertical, consultez [Prise en main des requêtes de bases de données croisées (partitionnement vertical)](elastic-query-getting-started-vertical.md).
* Pour commencer le didacticiel sur le partitionnement horizontal (sharding), consultez [Prise en main des requêtes élastiques pour le partitionnement horizontal (sharding)](elastic-query-getting-started.md).
* Pour la syntaxe et des exemples de requêtes pour des données partitionnées horizontalement, consultez [Requêtes sur des données partitionnées horizontalement](elastic-query-horizontal-partitioning.md)
* Consultez [sp\_execute \_remote](/sql/relational-databases/system-stored-procedures/sp-execute-remote-azure-sql-database) pour une procédure stockée qui exécute une instruction Transact-SQL sur une seule base de données Azure SQL distante ou un ensemble de bases de données servant de partitions dans un schéma de partitionnement horizontal.

<!--Image references-->
[1]: ./media/elastic-query-vertical-partitioning/verticalpartitioning.png

<!--anchors-->