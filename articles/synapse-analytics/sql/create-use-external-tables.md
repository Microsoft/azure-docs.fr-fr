---
title: Créer et utiliser des tables externes dans le pool SQL serverless
description: Dans cette section, vous allez découvrir comment créer et utiliser des tables externes dans le pool SQL serverless.
services: synapse-analytics
author: vvasic-msft
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: vvasic
ms.reviewer: jrasnick
ms.openlocfilehash: 5dcd4b7b76752ca5396fc68afc8d4c8e4e1edca5
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462299"
---
# <a name="create-and-use-external-tables-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Créer et utiliser des tables externes avec le pool SQL serverless dans Azure Synapse Analytics

Dans cette section, vous allez découvrir comment créer et utiliser des [tables externes](develop-tables-external-tables.md) dans le pool SQL serverless. Les tables externes sont utiles quand vous souhaitez contrôler l’accès aux données externes dans un pool SQL serverless et si vous voulez utiliser des outils comme Power BI en combinaison avec un pool SQL serverless. Les tables externes peuvent accéder à deux types de stockage :
- le stockage public où les utilisateurs accèdent aux fichiers de stockage public ;
- le stockage protégé où les utilisateurs accèdent aux fichiers de stockage à l’aide d’informations d’identification SAP, d’une identité Azure AD ou de l’identité managée d’un espace de travail Synapse.

## <a name="prerequisites"></a>Prérequis

La première étape consiste à créer la base de données dans laquelle les tables seront créées. Ensuite, initialisez les objets en exécutant le [script d’installation](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) sur cette base de données. Ce script crée les objets suivants qui sont utilisés dans cet exemple :
- DATABASE SCOPED CREDENTIAL `sqlondemand` qui permet d’accéder au compte de stockage Azure `https://sqlondemandstorage.blob.core.windows.net` protégé par SAP.

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL [sqlondemand]
    WITH IDENTITY='SHARED ACCESS SIGNATURE',  
    SECRET = 'sv=2018-03-28&ss=bf&srt=sco&sp=rl&st=2019-10-14T12%3A10%3A25Z&se=2061-12-31T12%3A10%3A00Z&sig=KlSU2ullCscyTS0An0nozEpo4tO5JAgGBvw%2FJX2lguw%3D'
    ```

- EXTERNAL DATA SOURCE `sqlondemanddemo` qui fait référence au compte de stockage de démonstration protégé par clé SAP et EXTERNAL DATA SOURCE `YellowTaxi` qui fait référence au compte de stockage Azure en disponibilité publique à l’emplacement `https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/`.

    ```sql
    CREATE EXTERNAL DATA SOURCE SqlOnDemandDemo WITH (
        LOCATION = 'https://sqlondemandstorage.blob.core.windows.net',
        CREDENTIAL = sqlondemand
    );
    GO
    CREATE EXTERNAL DATA SOURCE YellowTaxi
    WITH ( LOCATION = 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/')
    ```

- FILE FORMAT `QuotedCSVWithHeaderFormat` et FILE FORMAT `ParquetFormat` qui décrivent les types de fichiers CSV et Parquet.

    ```sql
    CREATE EXTERNAL FILE FORMAT QuotedCsvWithHeaderFormat
    WITH (  
        FORMAT_TYPE = DELIMITEDTEXT,
        FORMAT_OPTIONS ( FIELD_TERMINATOR = ',', STRING_DELIMITER = '"', FIRST_ROW = 2   )
    );
    GO
    CREATE EXTERNAL FILE FORMAT ParquetFormat WITH (  FORMAT_TYPE = PARQUET );
    ```

Les requêtes de cet article seront exécutées sur votre exemple de base de données et utiliseront ces objets. 

## <a name="create-an-external-table-on-protected-data"></a>Création d’une table externe sur des données protégées

Il est possible de créer des tables externes qui accèdent aux données d’un compte de stockage Azure autorisant l’accès aux utilisateurs munis d’une identité Azure AD ou d’une clé SAP. Vous pouvez créer des tables externes de la même façon que vous créez habituellement des tables externes SQL Server. 

La requête suivante crée une table externe qui lit le fichier *population.csv* à partir du compte de stockage Azure de démonstration Synapse SQL auquel il est fait référence à l’aide de la source de données `sqlondemanddemo` et qui est protégé avec les informations d’identification étendues à la base de données appelées `sqlondemand`. 

La source de données et les informations d’identification étendues à la base de données sont créées dans le [script d’installation](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql).

> [!NOTE]
> Modifiez la première ligne de la requête, c’est-à-dire [mydbname], afin d’utiliser la base de données que vous avez créée. 

```sql
USE [mydbname];
GO
CREATE EXTERNAL TABLE populationExternalTable
(
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
    [year] smallint,
    [population] bigint
)
WITH (
    LOCATION = 'csv/population/population.csv',
    DATA_SOURCE = sqlondemanddemo,
    FILE_FORMAT = QuotedCSVWithHeaderFormat
);
```

## <a name="create-an-external-table-on-public-data"></a>Création d’une table externe sur des données publiques

Il est possible de créer des tables externes qui lisent les données de fichiers placés sur le stockage Azure en disponibilité publique. Ce [script d’installation](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) crée la source de données externe publique et la définition de format de fichier Parquet qui sont utilisées dans la requête suivante :

```sql
CREATE EXTERNAL TABLE Taxi (
     vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
     pickup_datetime DATETIME2, 
     dropoff_datetime DATETIME2,
     passenger_count INT,
     trip_distance FLOAT,
     fare_amount FLOAT,
     tip_amount FLOAT,
     tolls_amount FLOAT,
     total_amount FLOAT
) WITH (
         LOCATION = 'puYear=*/puMonth=*/*.parquet',
         DATA_SOURCE = YellowTaxi,
         FILE_FORMAT = ParquetFormat
);
```
## <a name="use-an-external-table"></a>Utilisation d’une table externe

Il est possible d’utiliser des [tables externes](develop-tables-external-tables.md) dans les requêtes de la même façon que dans les requêtes SQL Server.

La requête suivante illustre ce point avec la table externe *population* créée dans la section précédente. Elle retourne les noms des pays/régions avec leur population en 2019, dans l’ordre décroissant.

> [!NOTE]
> Modifiez la première ligne de la requête, c’est-à-dire [mydbname], afin d’utiliser la base de données que vous avez créée.

```sql
USE [mydbname];
GO

SELECT
    country_name, population
FROM populationExternalTable
WHERE
    [year] = 2019
ORDER BY
    [population] DESC;
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le stockage des résultats d’une requête dans le stockage, consultez l’article [Stocker les résultats d’une requête dans le stockage](../sql/create-external-table-as-select.md).
