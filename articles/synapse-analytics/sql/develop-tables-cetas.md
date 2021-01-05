---
title: Instruction CREATE EXTERNAL TABLE AS SELECT (CETAS) dans Synapse SQL
description: Utiliser l'instruction CREATE EXTERNAL TABLE AS SELECT (CETAS) avec Synapse SQL
services: synapse-analytics
author: filippopovic
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 09/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: 8ffb3a0948267ea40a5d0511de63a80ad23584d1
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96454667"
---
# <a name="cetas-with-synapse-sql"></a>CETAS avec Synapse SQL

Vous pouvez utiliser CREATE EXTERNAL TABLE AS SELECT (CETAS) dans le pool SQL dédié ou dans le pool SQL serverless pour effectuer les tâches suivantes :  

- Créer une table externe
- Exporter, en parallèle, les résultats d'une instruction Transact-SQL SELECT vers :

  - Hadoop
  - Azure Storage Blob
  - Azure Data Lake Storage Gen2

## <a name="cetas-in-dedicated-sql-pool"></a>CETAS dans un pool SQL dédié

Pour plus d’informations sur le pool SQL dédié, et sur l’utilisation et la syntaxe de CETAS, consultez l’article [CREATE EXTERNAL TABLE AS SELECT](/sql/t-sql/statements/create-external-table-as-select-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true). Pour obtenir des conseils sur CTAS avec un pool SQL dédié, consultez l’article [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="cetas-in-serverless-sql-pool"></a>CETAS dans un pool SQL serverless

Lors de l’utilisation d’un pool SQL serverless, CETAS est utilisé pour créer une table externe et exporter les résultats de la requête vers Stockage Blob Azure ou Azure Data Lake Storage Gen2.

## <a name="syntax"></a>Syntaxe

```syntaxsql
CREATE EXTERNAL TABLE [ [database_name  . [ schema_name ] . ] | schema_name . ] table_name
    WITH (
        LOCATION = 'path_to_folder',  
        DATA_SOURCE = external_data_source_name,  
        FILE_FORMAT = external_file_format_name  
)
    AS <select_statement>  
[;]

<select_statement> ::=  
    [ WITH <common_table_expression> [ ,...n ] ]  
    SELECT <select_criteria>
```

## <a name="arguments"></a>Arguments

*[ [ *database_name* . [ *schema_name* ] . ] | *schema_name* . ] *table_name**

Nom (composé d’une à trois parties) de la table à créer. Pour une table externe, un pool SQL serverless stocke seulement les métadonnées de la table. Aucune donnée réelle n’est déplacée ni stockée dans un pool SQL serverless.

LOCATION = *'path_to_folder'*

Spécifie l’emplacement dans lequel écrire les résultats de l’instruction SELECT exécutée sur la source de données externes. Le dossier racine est l’emplacement de données qui est spécifié dans la source de données externe. L’emplacement doit pointer vers un dossier et se terminer par un /. Exemple : aggregated_data/

DATA_SOURCE = *external_data_source_name*

Spécifie le nom de l’objet de source de données externe qui contient l’emplacement où les données externes vont être stockées. Pour créer une source de données externe, utilisez [CREATE EXTERNAL DATA SOURCE (Transact-SQL)](develop-tables-external-tables.md#create-external-data-source).

FILE_FORMAT = *external_file_format_name*

Spécifie le nom de l’objet de format de fichier externe qui contient le format du fichier de données externe. Pour créer un format de fichier externe, utilisez [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](develop-tables-external-tables.md#create-external-file-format). Seuls les formats de fichiers externes avec FORMAT_TYPE=PARQUET et FORMAT_TYPE=DELIMITEDTEXT sont actuellement pris en charge. La compression GZip pour le format DELIMITEDTEXT n’est pas prise en charge.

WITH *<common_table_expression>*

Spécifie un jeu de résultats nommé temporaire, désigné par le terme d'expression de table commune (CTE, Common Table Expression). Pour plus d’informations, consultez [WITH common_table_expression (Transact-SQL)](/sql/t-sql/queries/with-common-table-expression-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

SELECT <select_criteria>

Remplit la nouvelle table avec les résultats d’une instruction SELECT. *select_criteria* correspond au corps de l’instruction SELECT qui détermine les données qui sont copiées dans la nouvelle table. Pour plus d’informations sur les instructions SELECT, consultez [SELECT (Transact-SQL)](/sql/t-sql/queries/select-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

> [!NOTE]
> La clause ORDER BY de SELECT n'est pas prise en charge pour CETAS.

## <a name="permissions"></a>Autorisations

Pour que CETAS fonctionne, vous devez disposer d'autorisations permettant d'afficher le contenu des dossiers et permettant d'écrire dans le dossier LOCATION.

## <a name="examples"></a>Exemples

Ces exemples utilisent CETAS pour enregistrer la population totale agrégée par année et par état dans le dossier aggregated_data qui se trouve dans la source de données population_ds.

Cet exemple s’appuie sur les informations d’identification, la source de données et le format de fichier externe créés précédemment. Reportez-vous au document [tables externes](develop-tables-external-tables.md). Pour enregistrer les résultats d’une requête dans un autre dossier d’une même source de données, modifiez l’argument LOCATION. 

Pour enregistrer les résultats dans un autre compte de stockage, créez une autre source de données pour l’argument DATA_SOURCE.

> [!NOTE]
> Les exemples qui suivent utilisent un compte Azure Open Data Storage public. Ce compte est en lecture seule. Pour exécuter ces requêtes, vous devez fournir la source de données pour laquelle vous disposez d’autorisations en écriture.

```sql
-- use CETAS to export select statement with OPENROWSET result to  storage
CREATE EXTERNAL TABLE population_by_year_state
WITH (
    LOCATION = 'aggregated_data/',
    DATA_SOURCE = population_ds,  
    FILE_FORMAT = census_file_format
)  
AS
SELECT decennialTime, stateName, SUM(population) AS population
FROM
    OPENROWSET(BULK 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer/release/us_population_county/year=*/*.parquet',
    FORMAT='PARQUET') AS [r]
GROUP BY decennialTime, stateName
GO

-- you can query the newly created external table
SELECT * FROM population_by_year_state
```

L’exemple suivant utilise une table externe comme source pour CETAS. Il s’appuie sur les informations d’identification, la source de données, le format de fichier externe et la table externe créés précédemment. Reportez-vous au document [tables externes](develop-tables-external-tables.md).

```sql
-- use CETAS with select from external table
CREATE EXTERNAL TABLE population_by_year_state
WITH (
    LOCATION = 'aggregated_data/',
    DATA_SOURCE = population_ds,  
    FILE_FORMAT = census_file_format
)  
AS
SELECT decennialTime, stateName, SUM(population) AS population
FROM census_external_table
GROUP BY decennialTime, stateName
GO

-- you can query the newly created external table
SELECT * FROM population_by_year_state
```

## <a name="supported-data-types"></a>Types de données pris en charge

CETAS peut être utilisé pour stocker les jeux de résultats avec les types de données SQL suivants :

- binary
- varbinary
- char
- varchar
- NCHAR
- NVARCHAR
- smalldate
- Date
- DATETIME
- datetime2
- datetimeoffset
- time
- Décimal
- numeric
- float
- real
- bigint
- TINYINT
- SMALLINT
- int
- bigint
- bit
- money
- SMALLMONEY
- UNIQUEIDENTIFIER

> [!NOTE]
> Les LOB supérieurs à 1 Mo ne peuvent pas être utilisés avec CETAS.

## <a name="next-steps"></a>Étapes suivantes

Essayez d'interroger des [Tables externes Apache Spark pour Azure Synapse](develop-storage-files-spark-tables.md).
