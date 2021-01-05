---
title: Interroger des fichiers Parquet à l’aide d’un pool SQL serverless
description: Cet article explique comment interroger des fichiers Parquet à l’aide du pool SQL serverless.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 05/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: cce4c6aff986c2e8c3d879d962714e13f6b2e7ae
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97694677"
---
# <a name="query-parquet-files-using-serverless-sql-pool-in-azure-synapse-analytics"></a>Interroger des fichiers Parquet à l’aide d’un pool SQL serverless dans Azure Synapse Analytics

Cet article explique comment écrire une requête à l’aide d’un pool SQL serverless pour lire des fichiers Parquet.

## <a name="quickstart-example"></a>Exemple de démarrage rapide

La fonction `OPENROWSET` vous permet de lire le contenu d’un fichier Parquet en fournissant l’URL de votre fichier.

### <a name="read-parquet-file"></a>Lire un fichier Parquet

Le moyen le plus simple d’afficher le contenu de votre fichier `PARQUET` consiste à fournir l’URL du fichier à la fonction `OPENROWSET` et à spécifier le `FORMAT` Parquet. Si le fichier est disponible publiquement ou si votre identité Azure AD peut y accéder, vous devriez pouvoir voir le contenu du fichier à l’aide d’une requête comme celle montrée dans l’exemple suivant :

```sql
select top 10 *
from openrowset(
    bulk 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases/latest/ecdc_cases.parquet',
    format = 'parquet') as rows
```

Assurez-vous que vous pouvez accéder à ce fichier. Si votre fichier est protégé par une clé SAP ou une identité Azure personnalisée, vous devez configurer les [informations d’identification au niveau du serveur pour la connexion SQL](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#server-scoped-credential).

> [!IMPORTANT]
> Assurez-vous que vous utilisez un classement de base de données UTF-8 (par exemple `Latin1_General_100_BIN2_UTF8`), car les valeurs de chaîne dans les fichiers PARQUET sont encodées à l’aide d’un encodage UTF-8.
> Une incompatibilité entre l’encodage de texte dans le fichier PARQUET et le classement peut entraîner des erreurs de conversion inattendues.
> Vous pouvez facilement modifier le classement par défaut de la base de données actuelle à l’aide de l’instruction T-SQL suivante : `alter database current collate Latin1_General_100_BIN2_UTF8`

### <a name="data-source-usage"></a>Utilisation d’une source de données

L’exemple précédent utilise le chemin complet du fichier. Vous pouvez également créer une source de données externe avec l’emplacement qui pointe vers le dossier racine du stockage et utiliser cette source de données et le chemin relatif du fichier dans la fonction `OPENROWSET` :

```sql
create external data source covid
with ( location = 'https://pandemicdatalake.blob.core.windows.net/public/curated/covid-19/ecdc_cases' );
go
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.parquet',
        data_source = 'covid',
        format = 'parquet'
    ) as rows
```

Si une source de données est protégée par une clé SAP ou une identité personnalisée, vous pouvez configurer la [source de données avec des informations d’identification dans l’étendue de la base de données](develop-storage-files-storage-access-control.md?tabs=shared-access-signature#database-scoped-credential).

### <a name="explicitly-specify-schema"></a>Spécifier explicitement le schéma

`OPENROWSET` vous permet de spécifier explicitement les colonnes que vous souhaitez lire à partir du fichier à l’aide de la clause `WITH` :

```sql
select top 10 *
from openrowset(
        bulk 'latest/ecdc_cases.parquet',
        data_source = 'covid',
        format = 'parquet'
    ) with ( date_rep date, cases int, geo_id varchar(6) ) as rows
```

> [!IMPORTANT]
> Veillez à spécifier explicitement un classement UTF-8 (par exemple `Latin1_General_100_BIN2_UTF8`) pour toutes les colonnes de chaîne dans la clause `WITH`, ou définissez un classement UTF-8 au niveau de la base de données.
> Une incompatibilité entre l’encodage de texte dans le fichier et le classement de colonne de chaîne peut entraîner des erreurs de conversion inattendues.
> Vous pouvez facilement modifier le classement par défaut de la base de données actuelle à l’aide de l’instruction T-SQL suivante : `alter database current collate Latin1_General_100_BIN2_UTF8`
> Vous pouvez facilement définir le classement sur les types de colonne à l’aide de la définition suivante : `geo_id varchar(6) collate Latin1_General_100_BIN2_UTF8`

Dans les sections suivantes, vous pouvez voir comment interroger différents types de fichiers Parquet.

## <a name="prerequisites"></a>Prérequis

La première étape consiste à **créer une base de données** avec une source de données qui fait référence au compte de stockage [NYC Yellow Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/). Ensuite, initialisez les objets en exécutant le [script d’installation](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) sur cette base de données. Ce script d’installation crée les sources de données, les informations d’identification délimitées à la base de données et les formats de fichiers externes utilisés dans ces exemples.

## <a name="dataset"></a>Dataset

Le jeu de données [NYC Yellow Taxi](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/) est utilisé dans cet exemple. Vous pouvez interroger les fichiers Parquet de la même façon que vous [lisez des fichiers CSV](query-parquet-files.md). La seule différence réside dans le fait que le paramètre `FILEFORMAT` doit être défini sur `PARQUET`. Les exemples de cet article montrent les spécificités de lecture des fichiers Parquet.

## <a name="query-set-of-parquet-files"></a>Ensemble de requêtes de fichiers Parquet

Vous pouvez spécifier uniquement les colonnes intéressantes lorsque vous interrogez des fichiers Parquet.

```sql
SELECT
        YEAR(tpepPickupDateTime),
        passengerCount,
        COUNT(*) AS cnt
FROM  
    OPENROWSET(
        BULK 'puYear=2018/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) WITH (
        tpepPickupDateTime DATETIME2,
        passengerCount INT
    ) AS nyc
GROUP BY
    passengerCount,
    YEAR(tpepPickupDateTime)
ORDER BY
    YEAR(tpepPickupDateTime),
    passengerCount;
```

## <a name="automatic-schema-inference"></a>Inférence de schéma automatique

Vous n’avez pas besoin d'utiliser la clause OPENROWSET WITH lors de la lecture de fichiers Parquet. Les noms de colonnes et les types de données sont automatiquement lus à partir des fichiers Parquet.

L’exemple ci-dessous montre les fonctionnalités d’inférence automatique du schéma des fichiers Parquet. Il renvoie le nombre de lignes en septembre 2017 sans spécifier de schéma.

> [!NOTE]
> Vous n’avez pas besoin de spécifier de colonnes dans la clause OPENROWSET WITH lors de la lecture de fichiers Parquet. Dans ce cas, le service de requête du pool SQL serverless utilise les métadonnées dans le fichier Parquet et lie les colonnes par nom.

```sql
SELECT TOP 10 *
FROM  
    OPENROWSET(
        BULK 'puYear=2018/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) AS nyc
```

### <a name="query-partitioned-data"></a>Interroger des données partitionnées

Le jeu de données fourni dans cet exemple est divisé (partitionné) en sous-dossiers distincts. Utilisez la fonction filepath pour cibler des partitions spécifiques. Cet exemple montre les montants par année, par mois et par type de paiement pour les trois premiers mois de 2017.

> [!NOTE]
> La requête du pool SQL serverless est compatible avec le schéma de partitionnement Hive/Hadoop.

```sql
SELECT
        YEAR(tpepPickupDateTime),
        passengerCount,
        COUNT(*) AS cnt
FROM  
    OPENROWSET(
        BULK 'puYear=*/puMonth=*/*.snappy.parquet',
        DATA_SOURCE = 'YellowTaxi',
        FORMAT='PARQUET'
    ) nyc
WHERE
    nyc.filepath(1) = 2017
    AND nyc.filepath(2) IN (1, 2, 3)
    AND tpepPickupDateTime BETWEEN CAST('1/1/2017' AS datetime) AND CAST('3/31/2017' AS datetime)
GROUP BY
    passengerCount,
    YEAR(tpepPickupDateTime)
ORDER BY
    YEAR(tpepPickupDateTime),
    passengerCount;
```

## <a name="type-mapping"></a>Mappage des types

Pour le mappage de type Parquet en type SQL natif, sélectionnez [Mappage de type pour Parquet](develop-openrowset.md#type-mapping-for-parquet).

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant pour savoir comment [interroger des types imbriqués Parquet](query-parquet-nested-types.md).
