---
title: Utiliser des métadonnées de fichier dans les requêtes
description: La fonction OPENROWSET fournit des informations de fichier et de chemin d’accès sur chaque fichier utilisé dans la requête pour filtrer ou analyser des données en fonction du nom de fichier ou du chemin d’accès du dossier.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: d24ae1f42c685589309506b2d5e0eab157b2bc42
ms.sourcegitcommit: 5e2f5efba1957ba40bd951c3dcad42f4a00734ff
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96299613"
---
# <a name="use-file-metadata-in-serverless-sql-pool-queries"></a>Utiliser les métadonnées de fichier dans les requêtes de pool SQL serverless

Le pool SQL serverless peut traiter plusieurs fichiers et dossiers, comme cela est décrit dans l’article [Interroger plusieurs dossiers et fichiers](query-folders-multiple-csv-files.md). Dans cet article, vous allez apprendre à utiliser des informations de métadonnées sur des noms de fichiers et de dossiers dans les requêtes.

Vous devrez peut-être parfois savoir quelle source de fichier ou de dossier est corrélée à une ligne spécifique dans le jeu de résultats.

Vous pouvez utiliser les fonctions `filepath` et `filename` pour retourner les noms ou le chemin d’accès des fichiers dans le jeu de résultats. Vous pouvez les utiliser pour filtrer des données sur le nom de ou le chemin d’accès du dossier. Ces fonctions sont décrites dans la section syntaxe des fonctions [filename](query-data-storage.md#filename-function) et [filepath](query-data-storage.md#filepath-function). Dans les sections suivantes, vous trouverez de courtes descriptions ainsi que des exemples.

## <a name="prerequisites"></a>Prérequis

La première étape consiste à **créer une base de données** avec une source de données qui fait référence au compte de stockage. Ensuite, initialisez les objets en exécutant le [script d’installation](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) sur cette base de données. Ce script d’installation crée les sources de données, les informations d’identification délimitées à la base de données et les formats de fichiers externes utilisés dans ces exemples.

## <a name="functions"></a>Fonctions

### <a name="filename"></a>Nom de fichier

Cette fonction retourne le nom du fichier sont provient la ligne.

L’exemple suivant lit les fichiers de données NYC Yellow Taxi pour les trois derniers mois de 2017, et retourne le nombre de courses par fichier. La partie OPENROWSET de la requête spécifie les fichiers à lire.

```sql
SELECT
    nyc.filename() AS [filename]
    ,COUNT_BIG(*) AS [rows]
FROM  
    OPENROWSET(
        BULK 'parquet/taxi/year=2017/month=9/*.parquet',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT='PARQUET'
    ) nyc
GROUP BY nyc.filename();
```

L’exemple suivant montre comment utiliser *filename()* dans la clause WHERE pour filtrer les fichiers à lire. Il accède au dossier entier dans la partie OPENROWSET de la requête et filtre les fichiers dans la clause WHERE.

Vos résultats seront les mêmes que dans l’exemple précédent.

```sql
SELECT
    r.filename() AS [filename]
    ,COUNT_BIG(*) AS [rows]
FROM OPENROWSET(
    BULK 'csv/taxi/yellow_tripdata_2017-*.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV',
        PARSER_VERSION = '2.0',
        FIRSTROW = 2) 
        WITH (C1 varchar(200) ) AS [r]
WHERE
    r.filename() IN ('yellow_tripdata_2017-10.csv', 'yellow_tripdata_2017-11.csv', 'yellow_tripdata_2017-12.csv')
GROUP BY
    r.filename()
ORDER BY
    [filename];
```

### <a name="filepath"></a>Filepath

La fonction filepath retourne un chemin d’accès complet ou partiel :

- En cas d’appel sans paramètre, elle retourne le chemin complet du fichier dont provient la ligne. Lorsque DATA_SOURCE est utilisé dans OPENROWSET, il retourne le chemin relatif à DATA_SOURCE. 
- En cas d’appel avec paramètre, elle retourne une partie du chemin qui correspond au caractère générique occupant la position spécifiée dans le paramètre. Par exemple, la valeur de paramètre 1 retourne une partie du chemin qui correspond au premier caractère générique.

L’exemple suivant lit les fichiers de données NYC Yellow Taxi pour les trois derniers mois de 2017. Elle retourne le nombre de courses par chemin d’accès de fichier. La partie OPENROWSET de la requête spécifie les fichiers à lire.

```sql
SELECT
    r.filepath() AS filepath
    ,COUNT_BIG(*) AS [rows]
FROM OPENROWSET(
        BULK 'csv/taxi/yellow_tripdata_2017-1*.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV',
        PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        vendor_id INT
    ) AS [r]
GROUP BY
    r.filepath()
ORDER BY
    filepath;
```

L’exemple suivant montre comment utiliser *filepath()* dans la clause WHERE pour filtrer les fichiers à lire.

Vous pouvez utiliser les caractères génériques dans la partie OPENROWSET de la requête et filtrer les fichiers dans la clause WHERE. Vos résultats seront les mêmes que dans l’exemple précédent.

```sql
SELECT
    r.filepath() AS filepath
    ,r.filepath(1) AS [year]
    ,r.filepath(2) AS [month]
    ,COUNT_BIG(*) AS [rows]
FROM OPENROWSET(
        BULK 'csv/taxi/yellow_tripdata_*-*.csv',
        DATA_SOURCE = 'SqlOnDemandDemo',
        FORMAT = 'CSV',
        PARSER_VERSION = '2.0',        
        FIRSTROW = 2
    )
WITH (
    vendor_id INT
) AS [r]
WHERE
    r.filepath(1) IN ('2017')
    AND r.filepath(2) IN ('10', '11', '12')
GROUP BY
    r.filepath()
    ,r.filepath(1)
    ,r.filepath(2)
ORDER BY
    filepath;
```

## <a name="next-steps"></a>Étapes suivantes

L’article suivant explique comment [interroger des fichiers Parquet](query-parquet-files.md).
