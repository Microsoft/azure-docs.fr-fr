---
title: Interroger des fichiers JSON à l’aide d’un pool SQL serverless
description: Cette section explique comment lire des fichiers JSON à l’aide d’un pool SQL serverless dans Azure Synapse Analytics.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 5703e4ac3d4a545041c7d258f0b1f0e7c79a33b5
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462611"
---
# <a name="query-json-files-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Interroger des fichiers JSON à l’aide d’un pool SQL serverless dans Azure Synapse Analytics

Cet article explique comment écrire une requête à l’aide d’un pool SQL serverless dans Azure Synapse Analytics. L’objectif de la requête est de lire des fichiers JSON avec [OPENROWSET](develop-openrowset.md). 
- Fichiers JSON standard où plusieurs documents JSON sont stockés sous la forme d’un tableau JSON.
- Fichiers JSON délimités par des lignes où les documents JSON sont séparés par un caractère de nouvelle ligne. Les extensions courantes pour ces types de fichiers sont `jsonl`, `ldjson` et `ndjson`.

## <a name="read-json-documents"></a>Lire les documents JSON

Le moyen le plus simple d’afficher le contenu de votre fichier JSON consiste à fournir l’URL du fichier à la fonction `OPENROWSET`, à spécifier le `FORMAT` CSV et à définir des valeurs `0x0b` pour `fieldterminator` et `fieldquote`. Si vous devez lire des fichiers JSON délimités par des lignes, cela est suffisant. Si vous avez un fichier JSON classique, vous devez définir des valeurs `0x0b` pour `rowterminator`. La fonction `OPENROWSET` analyse JSON et retourne tous les documents au format suivant :

| doc |
| --- |
|{"date_rep":"2020-07-24","day":24,"month":7,"year":2020,"cases":3,"deaths":0,"geo_id":"AF"}|
|{"date_rep":"2020-07-25","day":25,"month":7,"year":2020,"cases":7,"deaths":0,"geo_id":"AF"}|
|{"date_rep":"2020-07-26","day":26,"month":7,"year":2020,"cases":4,"deaths":0,"geo_id":"AF"}|
|{"date_rep":"2020-07-27","day":27,"month":7,"year":2020,"cases":8,"deaths":0,"geo_id":"AF"}|

Si le fichier est disponible publiquement ou si votre identité Azure AD peut y accéder, vous devriez voir le contenu du fichier à l’aide d’une requête comme celle montrée dans les exemples suivants.

### <a name="read-json-files"></a>Lire des fichiers JSON

L’exemple de requête suivant lit des fichiers JSON et des fichiers JSON et les fichiers JSON délimités par des lignes et retourne chaque document sous la forme d’une ligne distincte.

```sql
select top 10 *
from openrowset(
        bulk 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.jsonl',
        format = 'csv',
        fieldterminator ='0x0b',
        fieldquote = '0x0b'
    ) with (doc nvarchar(max)) as rows
go
select top 10 *
from openrowset(
        bulk 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.json',
        format = 'csv',
        fieldterminator ='0x0b',
        fieldquote = '0x0b',
        rowterminator = '0x0b' --> You need to override rowterminator to read classic JSON
    ) with (doc nvarchar(max)) as rows
```

Le document JSON de l’exemple de requête précédent comprend un tableau d’objets. La requête retourne chaque objet sous la forme d’une ligne distincte dans le jeu de résultats. Assurez-vous que vous pouvez accéder à ce fichier. Si votre fichier est protégé par une clé SAS ou une identité personnalisée, vous devez configurer les [informations d’identification au niveau du serveur pour la connexion SQL](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#server-scoped-credential). 

### <a name="data-source-usage"></a>Utilisation d’une source de données

L’exemple précédent utilise le chemin complet du fichier. Vous pouvez également créer une source de données externe avec l’emplacement qui pointe vers le dossier racine du stockage et utiliser cette source de données et le chemin relatif du fichier dans la fonction `OPENROWSET` :

```sql
create external data source covid
with ( location = 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases' );
go
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.jsonl',
        data_source = 'covid',
        format = 'csv',
        fieldterminator ='0x0b',
        fieldquote = '0x0b'
    ) with (doc nvarchar(max)) as rows
go
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.json',
        data_source = 'covid',
        format = 'csv',
        fieldterminator ='0x0b',
        fieldquote = '0x0b',
        rowterminator = '0x0b' --> You need to override rowterminator to read classic JSON
    ) with (doc nvarchar(max)) as rows
```

Si une source de données est protégée par une clé SAS ou une identité personnalisée, vous pouvez configurer la [source de données avec des informations d’identification dans l’étendue de la base de données](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#database-scoped-credential).

Dans les sections suivantes, vous pouvez voir comment interroger différents types de fichiers JSON.

## <a name="parse-json-documents"></a>Analyser des documents JSON

Les requêtes des exemples précédents renvoient chaque document JSON sous la forme d’une chaîne unique dans une ligne distincte du jeu de résultats. Vous pouvez utiliser les fonctions `JSON_VALUE` et `OPENJSON` pour analyser les valeurs des documents JSON et les retourner en tant que valeurs relationnelles, comme le montre l’exemple suivant :

| date\_rep | cas | geo\_id |
| --- | --- | --- |
| 2020-07-24 | 3 | AF |
| 2020-07-25 | 7 | AF |
| 2020-07-26 | 4 | AF |
| 2020-07-27 | 8| AF |

### <a name="sample-json-document"></a>Exemple de document JSON

Les exemples de requêtes lisent les fichiers *json* contenant des documents avec la structure suivante :

```json
{
    "date_rep":"2020-07-24",
    "day":24,"month":7,"year":2020,
    "cases":13,"deaths":0,
    "countries_and_territories":"Afghanistan",
    "geo_id":"AF",
    "country_territory_code":"AFG",
    "continent_exp":"Asia",
    "load_date":"2020-07-25 00:05:14",
    "iso_country":"AF"
}
```

> [!NOTE]
> Si ces documents sont stockés sous forme de fichiers JSON délimités par des lignes, vous devez définir `FIELDTERMINATOR` et `FIELDQUOTE` sur 0x0b. Si vous avez un format JSON standard, vous devez définir `ROWTERMINATOR` sur 0x0b.

### <a name="query-json-files-using-json_value"></a>Interroger des fichiers JSON à l’aide de JSON_VALUE

La requête ci-dessous vous montre comment utiliser [JSON_VALUE](/sql/t-sql/functions/json-value-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) pour récupérer des valeurs scalaires (titre, éditeur) dans des document JSON :

```sql
select
    JSON_VALUE(doc, '$.date_rep') AS date_reported,
    JSON_VALUE(doc, '$.countries_and_territories') AS country,
    JSON_VALUE(doc, '$.cases') as cases,
    doc
from openrowset(
        bulk 'latest/ecdc_cases.jsonl',
        data_source = 'covid',
        format = 'csv',
        fieldterminator ='0x0b',
        fieldquote = '0x0b'
    ) with (doc nvarchar(max)) as rows
order by JSON_VALUE(doc, '$.geo_id') desc
```

### <a name="query-json-files-using-openjson"></a>Interroger des fichiers JSON à l’aide de OPENJSON

La requête suivante utilise [OPENJSON](/sql/t-sql/functions/openjson-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest). Elle récupère les statistiques de la COVID signalées en Serbie :

```sql
select
    *
from openrowset(
        bulk 'latest/ecdc_cases.jsonl',
        data_source = 'covid',
        format = 'csv',
        fieldterminator ='0x0b',
        fieldquote = '0x0b'
    ) with (doc nvarchar(max)) as rows
    cross apply openjson (doc)
        with (  date_rep datetime2,
                cases int,
                fatal int '$.deaths',
                country varchar(100) '$.countries_and_territories')
where country = 'Serbia'
order by country, date_rep desc;
```

## <a name="next-steps"></a>Étapes suivantes

Les articles suivants de cette série vont vous montrer comment exécuter les tâches suivantes :

- [Interrogation de plusieurs dossiers et fichiers](query-folders-multiple-csv-files.md)
- [Créer et utiliser des vues](create-use-views.md)
