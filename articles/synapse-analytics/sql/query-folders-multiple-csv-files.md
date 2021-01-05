---
title: Interroger plusieurs dossiers et fichiers à l’aide d’un pool SQL serverless
description: Un pool SQL serverless prend en charge la lecture de plusieurs fichiers ou dossiers en utilisant des caractères génériques similaires à ceux utilisés dans le système d’exploitation Windows.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 04/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 83c4d88e1a87f6b546e26dd55da338a36f16ebe4
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462621"
---
# <a name="query-folders-and-multiple-files"></a>Interroger des dossiers et plusieurs fichiers  

Cet article explique comment écrire une requête à l’aide d’un pool SQL serverless dans Azure Synapse Analytics.

Un pool SQL serverless prend en charge la lecture de plusieurs fichiers ou dossiers en utilisant des caractères génériques similaires à ceux utilisés dans le système d’exploitation Windows. Toutefois, il offre davantage de flexibilité, car l’utilisation de plusieurs caractères génériques est autorisée.

## <a name="prerequisites"></a>Prérequis

La première étape consiste à **créer la base de données** dans laquelle seront exécutées les requêtes. Ensuite, initialisez les objets en exécutant le [script d’installation](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) sur cette base de données. Ce script crée les sources de données, les informations d'identification étendues à la base de données et les formats de fichiers externes utilisés dans ces exemples.

Vous allez utiliser le dossier *csv/taxi* pour suivre les exemples de requêtes. Il contient les données d’enregistrements de courses des taxis jaunes de New York couvrant la période de juillet 2016 à juin 2018. Les fichiers du dossier *csv/taxi* sont nommés d'après l'année et le mois en utilisant le modèle suivant : yellow_tripdata_<year>-<month>.csv

## <a name="read-all-files-in-folder"></a>Lire tous les fichiers dans le dossier

L’exemple ci-dessous lit tous les fichiers de données NYC Yellow Taxi figurant dans le dossier *csv/taxi*, et retourne le nombre total de passagers et de trajets par an. Il montre également l’utilisation de fonctions d’agrégation.

```sql
SELECT 
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
        BULK 'csv/taxi/*.csv',
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        pickup_datetime DATETIME2 2, 
        passenger_count INT 4
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Tous les fichiers accessibles avec la fonction OPENROWSET seule doivent présenter la même structure (c’est-à-dire les mêmes nombre de colonnes et types de données).

### <a name="read-subset-of-files-in-folder"></a>Lire un sous-ensemble de fichiers dans le dossier

L’exemple ci-dessous lit les fichiers de données NYC Yellow Taxi pour 2017 dans le dossier *csv/taxi* en utilisant un caractère générique, et retourne le montant total par type de paiement.

```sql
SELECT 
    payment_type,  
    SUM(fare_amount) AS fare_total
FROM OPENROWSET(
        BULK 'csv/taxi/yellow_tripdata_2017-*.csv',
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        payment_type INT 10,
        fare_amount FLOAT 11
    ) AS nyc
GROUP BY payment_type
ORDER BY payment_type;
```

> [!NOTE]
> Tous les fichiers accessibles avec la fonction OPENROWSET seule doivent présenter la même structure (c’est-à-dire les mêmes nombre de colonnes et types de données).

## <a name="read-folders"></a>Lire les dossiers

Le chemin d’accès que vous fournissez à la fonction OPENROWSET peut également être celui d’un dossier. Les sections suivantes incluent le types de requêtes suivants.

### <a name="read-all-files-from-specific-folder"></a>Lire tous les fichiers d’un dossier spécifique

Vous pouvez lire tous les fichiers figurant dans un dossier en utilisant le caractère générique de niveau fichier, comme indiqué dans [Lire tous les fichiers dans le dossier](#read-all-files-in-folder). Il existe cependant un moyen d’interroger un dossier et de consommer tous les fichiers qu’il contient.

Si le chemin d’accès fourni dans la fonction OPENROWSET pointe vers un dossier, tous les fichiers figurant dans celui-ci sont utilisés comme source pour votre requête. La requête suivante lit tous les fichiers figurant dans le dossier *csv/taxi*.

> [!NOTE]
> Notez la présence de la barre oblique (/) à la fin du chemin d’accès dans la requête ci-dessous. Elle indique qu’il s’agit d’un dossier. En cas d’omission de la barre oblique (/), la requête cible un fichier nommé *taxi* à la place.

```sql
SELECT
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
        BULK 'csv/taxi/',
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
        pickup_datetime DATETIME2, 
        dropoff_datetime DATETIME2,
        passenger_count INT,
        trip_distance FLOAT,
        rate_code INT,
        store_and_fwd_flag VARCHAR(100) COLLATE Latin1_General_BIN2,
        pickup_location_id INT,
        dropoff_location_id INT,
        payment_type INT,
        fare_amount FLOAT,
        extra FLOAT,
        mta_tax FLOAT,
        tip_amount FLOAT,
        tolls_amount FLOAT,
        improvement_surcharge FLOAT,
        total_amount FLOAT
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Tous les fichiers accessibles avec la fonction OPENROWSET seule doivent présenter la même structure (c’est-à-dire les mêmes nombre de colonnes et types de données).

### <a name="read-all-files-from-multiple-folders"></a>Lire tous les fichiers de plusieurs dossiers

Vous pouvez lire tous les fichiers figurant dans plusieurs dossiers en utilisant un caractère générique. La requête suivante lit tous les fichiers de tous les dossiers figurant dans le dossier *csv*, dont les noms commencent par *t* et se terminent par *i*.

> [!NOTE]
> Notez la présence de la barre oblique (/) à la fin du chemin d’accès dans la requête ci-dessous. Elle indique qu’il s’agit d’un dossier. En cas d’omission de la barre oblique (/), la requête cible les fichiers nommés *t&ast;i* à la place.

```sql
SELECT
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
        BULK 'csv/t*i/', 
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
        pickup_datetime DATETIME2, 
        dropoff_datetime DATETIME2,
        passenger_count INT,
        trip_distance FLOAT,
        rate_code INT,
        store_and_fwd_flag VARCHAR(100) COLLATE Latin1_General_BIN2,
        pickup_location_id INT,
        dropoff_location_id INT,
        payment_type INT,
        fare_amount FLOAT,
        extra FLOAT,
        mta_tax FLOAT,
        tip_amount FLOAT,
        tolls_amount FLOAT,
        improvement_surcharge FLOAT,
        total_amount FLOAT
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Tous les fichiers accessibles avec la fonction OPENROWSET seule doivent présenter la même structure (c’est-à-dire les mêmes nombre de colonnes et types de données).

Étant donné qu’un seul dossier correspond aux critères, le résultat de la requête est le même que celui obtenu avec l’option [Lire tous les fichiers dans le dossier](#read-all-files-in-folder).

## <a name="traverse-folders-recursively"></a>Parcourir les dossiers de manière récursive

Un pool SQL serverless peut parcourir les dossiers de manière récursive si vous spécifiez /** à la fin du chemin d’accès. La requête suivante lit tous les fichiers de tous les dossiers et sous-dossiers figurant dans le dossier *csv*.

```sql
SELECT
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
        BULK 'csv/taxi/**', 
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
        pickup_datetime DATETIME2, 
        dropoff_datetime DATETIME2,
        passenger_count INT,
        trip_distance FLOAT,
        rate_code INT,
        store_and_fwd_flag VARCHAR(100) COLLATE Latin1_General_BIN2,
        pickup_location_id INT,
        dropoff_location_id INT,
        payment_type INT,
        fare_amount FLOAT,
        extra FLOAT,
        mta_tax FLOAT,
        tip_amount FLOAT,
        tolls_amount FLOAT,
        improvement_surcharge FLOAT,
        total_amount FLOAT
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Tous les fichiers accessibles avec la fonction OPENROWSET seule doivent présenter la même structure (c’est-à-dire les mêmes nombre de colonnes et types de données).

## <a name="multiple-wildcards"></a>Plusieurs caractères génériques

Vous pouvez utiliser plusieurs caractères génériques sur différents niveaux de chemin d’accès. Par exemple, vous pouvez enrichir la requête précédente afin de lire uniquement les fichiers contenant des données de 2017 dans tous les dossiers dont le nom commence par *t* et se termine par *i*.

> [!NOTE]
> Notez la présence de la barre oblique (/) à la fin du chemin d’accès dans la requête ci-dessous. Elle indique qu’il s’agit d’un dossier. En cas d’omission de la barre oblique (/), la requête cible les fichiers nommés *t&ast;i* à la place.
> Le nombre maximal de caractères génériques par requête est limité à 10.

```sql
SELECT
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
        BULK 'csv/t*i/yellow_tripdata_2017-*.csv',
        DATA_SOURCE = 'sqlondemanddemo',
        FORMAT = 'CSV', PARSER_VERSION = '2.0',
        FIRSTROW = 2
    )
    WITH (
        vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
        pickup_datetime DATETIME2, 
        dropoff_datetime DATETIME2,
        passenger_count INT,
        trip_distance FLOAT,
        rate_code INT,
        store_and_fwd_flag VARCHAR(100) COLLATE Latin1_General_BIN2,
        pickup_location_id INT,
        dropoff_location_id INT,
        payment_type INT,
        fare_amount FLOAT,
        extra FLOAT,
        mta_tax FLOAT,
        tip_amount FLOAT,
        tolls_amount FLOAT,
        improvement_surcharge FLOAT,
        total_amount FLOAT
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Tous les fichiers accessibles avec la fonction OPENROWSET seule doivent présenter la même structure (c’est-à-dire les mêmes nombre de colonnes et types de données).

Étant donné qu’un seul dossier correspond aux critères, le résultat de la requête est le même que celui obtenu avec les options [Lire un sous-ensemble de fichiers dans un dossier](#read-subset-of-files-in-folder) et [Lire tous les fichiers d’un dossier spécifique](#read-all-files-from-specific-folder). Des scénarios d’utilisation de caractères génériques plus complexes sont traités dans [Interroger des fichiers Parquet](query-parquet-files.md).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez l’article [Interroger des fichiers spécifiques](query-specific-files.md).
