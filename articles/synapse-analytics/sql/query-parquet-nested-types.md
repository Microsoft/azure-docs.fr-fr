---
title: Interroger des types imbriqués Parquet à l’aide d’un pool SQL serverless
description: Cet article vous explique comment interroger des types imbriqués Parquet à l’aide du pool SQL serverless.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 91f612ba7f19deb739dbb6004e275ea044a5a3d3
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462566"
---
# <a name="query-nested-types-in-parquet-and-json-files-by-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Interroger des types imbriqués dans des fichiers Parquet et JSON à l’aide d’un pool SQL serverless dans Azure Synapse Analytics

Cet article explique comment écrire une requête à l’aide d’un pool SQL serverless dans Azure Synapse Analytics. La requête va lire des types imbriqués Parquet.
Les types imbriqués sont des structures complexes qui représentent des objets ou des tableaux. Les types imbriqués peuvent être stockés dans : 
- [Parquet](query-parquet-files.md) où vous pouvez avoir plusieurs colonnes complexes qui contiennent des tableaux et des objets.
- [Fichiers JSON](query-json-files.md) hiérarchiques où vous pouvez lire un document JSON complexe en tant que colonne unique.
- Collections Azure Cosmos DB (actuellement en préversion publique contrôlée) où chaque document peut contenir des propriétés imbriquées complexes.

Le pool SQL serverless met en forme tous les types imbriqués en tant qu’objets et tableaux JSON. Vous pouvez [extraire ou modifier des objets complexes à l’aide de fonctions JSON](https://docs.microsoft.com/sql/relational-databases/json/validate-query-and-change-json-data-with-built-in-functions-sql-server) ou [analyser des données JSON à l’aide de la fonction OPENJSON](https://docs.microsoft.com/sql/relational-databases/json/convert-json-data-to-rows-and-columns-with-openjson-sql-server). 

Voici un exemple de requête qui extrait des valeurs scalaires et d’objets du fichier de jeu de données JSON [COVID-19 Open Research Dataset](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/) qui contient des objets imbriqués : 

```sql
SELECT
    title = JSON_VALUE(doc, '$.metadata.title'),
    first_author = JSON_QUERY(doc, '$.metadata.authors[0]'),
    first_author_name = JSON_VALUE(doc, '$.metadata.authors[0].first'),
    complex_object = doc
FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/covid19temp/comm_use_subset/pdf_json/000b7d1517ceebb34e1e3e817695b6de03e2fa78.json',
        FORMAT='CSV', FIELDTERMINATOR ='0x0b', FIELDQUOTE = '0x0b', ROWTERMINATOR = '0x0b'
    )
    WITH ( doc varchar(MAX) ) AS docs;
```

La fonction `JSON_VALUE` retourne une valeur scalaire à partir du champ dans le chemin d’accès spécifié. La fonction `JSON_QUERY` retourne un objet au format JSON à partir du champ dans le chemin d’accès spécifié.

> [!IMPORTANT]
> Cet exemple utilise un fichier à partir du jeu de données COVID-19 Open Research Dataset. [Consultez la licence et la structure des données ici](https://azure.microsoft.com/services/open-datasets/catalog/covid-19-open-research/).

## <a name="prerequisites"></a>Prérequis

La première étape consiste à créer une base de données dans laquelle la source des données sera créée. Ensuite, vous allez initialiser les objets en exécutant un [script d’installation](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) sur la base de données. Le script crée les sources de données, les informations d'identification étendues à la base de données et les formats de fichiers externes utilisés dans les exemples.

## <a name="project-nested-or-repeated-data"></a>Projeter des données imbriquées ou répétées

Un fichier Parquet peut avoir plusieurs colonnes avec des types complexes. Les valeurs de ces colonnes sont mises en forme en tant que texte JSON et retournées en tant que colonnes VARCHAR. La requête suivante lit le fichier structExample.parquet et montre comment lire les valeurs des colonnes imbriquées : 

```sql
SELECT
    DateStruct, TimeStruct, TimestampStruct, DecimalStruct, FloatStruct
FROM
    OPENROWSET(
        BULK 'parquet/nested/structExample.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    )
    WITH (
        DateStruct VARCHAR(8000),
        TimeStruct VARCHAR(8000),
        TimestampStruct VARCHAR(8000),
        DecimalStruct VARCHAR(8000),
        FloatStruct VARCHAR(8000)
    ) AS [r];
```

Cette requête donne le résultat suivant. Le contenu de chaque objet imbriqué est retourné sous forme de texte JSON.

| DateStruct    | TimeStruct    | TimestampStruct   | DecimalStruct | FloatStruct |
| --- | --- | --- | --- | --- |
|{"Date":"2009-04-25"}| {"Time":"20:51:54.3598000"}|    {"Timestamp":"5501-04-08 12:13:57.4821000"}|    {"Decimal":11143412.25350}| {"Float":0.5}|
|{"Date":"1916-04-29"}| {"Time":"00:16:04.6778000"}|    {"Timestamp":"1990-06-30 20:50:52.6828000"}|    {"Decimal":1963545.62800}|  {"Float":-2.125}|

La requête suivante lit le fichier justSimpleArray.parquet. Il projette toutes les colonnes du fichier Parquet, y compris les données imbriquées et répétées.

```sql
SELECT
    SimpleArray
FROM
    OPENROWSET(
        BULK 'parquet/nested/justSimpleArray.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) AS [r];
```

Cette requête retournera le résultat suivant :

|SimpleArray|
| --- |
|[11,12,13]|
|[21,22,23]|

## <a name="read-properties-from-nested-object-columns"></a>Lire les propriétés à partir des colonnes d’objets imbriqués

La fonction `JSON_VALUE` vous permet de retourner des valeurs à partir de colonnes au format texte JSON :

```sql
SELECT
    title = JSON_VALUE(complex_column, '$.metadata.title'),
    first_author_name = JSON_VALUE(complex_column, '$.metadata.authors[0].first'),
    body_text = JSON_VALUE(complex_column, '$.body_text.text'),
    complex_column
FROM
    OPENROWSET( BULK 'https://azureopendatastorage.blob.core.windows.net/covid19temp/comm_use_subset/pdf_json/000b7d1517ceebb34e1e3e817695b6de03e2fa78.json',
                FORMAT='CSV', FIELDTERMINATOR ='0x0b', FIELDQUOTE = '0x0b', ROWTERMINATOR = '0x0b' ) WITH ( complex_column varchar(MAX) ) AS docs;
```

Le résultat est présenté dans le tableau suivant :

|title  | first_author_name | body_text | complex_column |
| --- | --- | --- | --- |
| Informations supplémentaires Un éco-épidémiolo... | Julien   | - Figure S1 : Phylogénie de... | `{    "paper_id": "000b7d1517ceebb34e1e3e817695b6de03e2fa78",    "metadata": {        "title": "Supplementary Information An eco-epidemiological study of Morbilli-related paramyxovirus infection in Madagascar bats reveals host-switching as the dominant macro-evolutionary mechanism",        "authors": [            {                "first": "Julien"` |

Contrairement aux fichiers JSON, qui, dans la plupart des cas, retournent une seule colonne contenant un objet JSON complexe, les fichiers Parquet peuvent avoir plusieurs colonnes complexes. Vous pouvez lire les propriétés de la colonne imbriquée à l’aide de la fonction `JSON_VALUE` sur chaque colonne. `OPENROWSET` vous permet de spécifier directement les chemins d’accès des propriétés imbriquées dans une clause `WITH`. Vous pouvez définir les chemins en tant que nom d’une colonne, ou vous pouvez ajouter une [expression JSON](https://docs.microsoft.com/sql/relational-databases/json/json-path-expressions-sql-server) après le type de colonne.

La requête suivante lit le fichier structExample.parquet et montre comment exposer les éléments d’une colonne imbriquée. Il y a deux façons de référencer une valeur imbriquée :
- En spécifiant l’expression de chemin de valeur imbriquée après la spécification de type.
- En mettant en forme du nom de colonne en tant que chemin imbriqué à l’aide de «.» pour référencer les champs.

```sql
SELECT
    *
FROM
    OPENROWSET(
        BULK 'parquet/nested/structExample.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    )
    WITH (
        [DateValue] DATE '$.DateStruct.Date',
        [TimeStruct.Time] TIME,
        [TimestampStruct.Timestamp] DATETIME2,
        DecimalValue DECIMAL(18, 5) '$.DecimalStruct.Decimal',
        [FloatStruct.Float] FLOAT
    ) AS [r];
```

## <a name="access-elements-from-repeated-columns"></a>Accéder aux éléments à partir de colonnes répétées

La requête suivante lit le fichier justSimpleArray.parquet et utilise [JSON_VALUE](/sql/t-sql/functions/json-value-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) pour récupérer un élément scalaire à partir d’une colonne répétée, comme un tableau ou une carte :

```sql
SELECT
    *,
    JSON_VALUE(SimpleArray, '$[0]') AS FirstElement,
    JSON_VALUE(SimpleArray, '$[1]') AS SecondElement,
    JSON_VALUE(SimpleArray, '$[2]') AS ThirdElement
FROM
    OPENROWSET(
        BULK 'parquet/nested/justSimpleArray.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) AS [r];
```

Voici le résultat :

|SimpleArray    | FirstElement  | SecondElement | ThirdElement |
| --- | --- | --- | --- |
| [11,12,13] | 11   | 12 | 13 |
| [21,22,23] | 21   | 22 | 23 |

## <a name="access-sub-objects-from-complex-columns"></a>Accéder à des sous-objets à partir de colonnes complexes

La requête suivante lit le fichier mapExample.parquet et utilise [JSON_QUERY](/sql/t-sql/functions/json-query-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) pour récupérer un élément non scalaire à partir d’une colonne répétée, telle qu’un tableau ou une carte :

```sql
SELECT
    MapOfPersons,
    JSON_QUERY(MapOfPersons, '$."John Doe"') AS [John]
FROM
    OPENROWSET(
        BULK 'parquet/nested/mapExample.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) AS [r];
```

Vous pouvez également référencer explicitement les colonnes que vous souhaitez renvoyer dans une clause `WITH` :

```sql
SELECT DocId,
    MapOfPersons,
    JSON_QUERY(MapOfPersons, '$."John Doe"') AS [John]
FROM
    OPENROWSET(
        BULK 'parquet/nested/mapExample.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) 
    WITH (DocId bigint, MapOfPersons VARCHAR(max)) AS [r];
```

La structure `MapOfPersons` est retournée en tant que colonne VARCHAR et mise en forme en tant que chaîne JSON.

## <a name="project-values-from-repeated-columns"></a>Projeter des valeurs à partir de colonnes répétées

Si vous avez un tableau de valeurs scalaires (par exemple `[1,2,3]`) dans certaines colonnes, vous pouvez facilement les développer et les joindre à la ligne principale à l’aide de ce script :

```sql
SELECT
    SimpleArray, Element
FROM
    OPENROWSET(
        BULK 'parquet/nested/justSimpleArray.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) AS arrays
    CROSS APPLY OPENJSON (SimpleArray) WITH (Element int '$') as array_values
```

## <a name="next-steps"></a>Étapes suivantes

L’article suivant vous indiquera comment [interroger des fichiers JSON](query-json-files.md).
