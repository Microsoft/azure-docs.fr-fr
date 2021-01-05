---
title: 'Démarrage rapide : Utiliser un pool SQL serverless'
description: Dans ce guide de démarrage rapide, vous allez voir et découvrir à quel point il est facile d’interroger différents types de fichiers en utilisant le pool SQL serverless.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql
ms.date: 04/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: e75bab664bdd0d6a4ac898905226c9956f62b4ff
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96450510"
---
# <a name="quickstart-use-serverless-sql-pool"></a>Démarrage rapide : Utiliser un pool SQL serverless

Le pool SQL serverless Synapse est un service de requête serverless qui vous permet d’exécuter des requêtes SQL sur des fichiers placés dans le Stockage Azure. Dans ce guide de démarrage rapide, vous allez apprendre à interroger différents types de fichiers en utilisant le pool SQL serverless. Les formats pris en charge sont répertoriés dans [OPENROWSET](sql/develop-openrowset.md).

Ce démarrage rapide montre comment interroger : Fichiers CSV, Apache Parquet et JSON.

## <a name="prerequisites"></a>Prérequis

Choisissez le client SQL avec lequel vous souhaitez émettre les requêtes :

- [Azure Synapse Studio](quickstart-synapse-studio.md) est un outil web que vous pouvez utiliser pour parcourir les fichiers dans le stockage et créer des requêtes SQL.
- [Azure Data Studio](sql/get-started-azure-data-studio.md) est un outil client qui vous permet d’exécuter des requêtes SQL et des notebooks sur votre base de données à la demande.
- [SQL Server Management Studio](sql/get-started-ssms.md) est un outil client qui vous permet d’exécuter des requêtes SQL sur votre base de données à la demande.

Paramètres pour ce guide de démarrage rapide :

| Paramètre                                 | Description                                                   |
| ----------------------------------------- | ------------------------------------------------------------- |
| Adresse du point de terminaison du service de pool SQL serverless    | Utilisée comme nom de serveur                                   |
| Région du point de terminaison du service de pool SQL serverless     | Utilisée pour déterminer le stockage que nous allons utiliser dans les exemples |
| Nom d’utilisateur et mot de passe pour l’accès au point de terminaison | Utilisés pour accéder au point de terminaison                               |
| Base de données utilisée pour créer des vues         | Base de données utilisée comme point de départ dans les exemples       |

## <a name="first-time-setup"></a>Première configuration

Avant d’utiliser les exemples :

- Créer une base de données pour vos vues (au cas où vous souhaiteriez utiliser des vues)
- Créer les informations d’identification que devra utiliser le pool SQL serverless pour accéder aux fichiers dans le stockage

### <a name="create-database"></a>Créer une base de données

Créez votre propre base de données à des fins de démonstration. Vous utiliserez cette base de données pour créer vos vues et pour les exemples de requêtes fournis dans cet article.

> [!NOTE]
> Les bases de données sont utilisées uniquement pour les métadonnées de vue, et non pour les données réelles.
>Notez le nom de base de données que vous utilisez, car vous en aurez besoin plus loin dans le Guide de démarrage rapide.

Utilisez la requête suivante, en remplaçant `mydbname` par le nom de votre choix :

```sql
CREATE DATABASE mydbname
```

### <a name="create-data-source"></a>Créer une source de données

Pour exécuter des requêtes à l’aide du pool SQL serverless, créez une source de données que le pool peut utiliser pour accéder aux fichiers dans le stockage.
Exécutez l’extrait de code suivant pour créer une source de données utilisée dans les exemples de cette section :

```sql
-- create master key that will protect the credentials:
CREATE MASTER KEY ENCRYPTION BY PASSWORD = <enter very strong password here>

-- create credentials for containers in our demo storage account
CREATE DATABASE SCOPED CREDENTIAL sqlondemand
WITH IDENTITY='SHARED ACCESS SIGNATURE',  
SECRET = 'sv=2018-03-28&ss=bf&srt=sco&sp=rl&st=2019-10-14T12%3A10%3A25Z&se=2061-12-31T12%3A10%3A00Z&sig=KlSU2ullCscyTS0An0nozEpo4tO5JAgGBvw%2FJX2lguw%3D'
GO
CREATE EXTERNAL DATA SOURCE SqlOnDemandDemo WITH (
    LOCATION = 'https://sqlondemandstorage.blob.core.windows.net',
    CREDENTIAL = sqlondemand
);
```

## <a name="query-csv-files"></a>Interroger des fichiers CSV

L’image suivante est un aperçu du fichier à interroger :

![10 premières lignes du fichier CSV sans en-tête, avec caractères de nouvelle ligne de style Windows.](./sql/media/query-single-csv-file/population.png)

La requête suivante montre comment lire un fichier CSV qui ne contient pas de ligne d’en-tête et comprend des caractères de nouvelle ligne de style Windows et des colonnes délimitées par des virgules :

```sql
SELECT TOP 10 *
FROM OPENROWSET
  (
      BULK 'csv/population/*.csv',
      DATA_SOURCE = 'SqlOnDemandDemo',
      FORMAT = 'CSV', PARSER_VERSION = '2.0'
  )
WITH
  (
      country_code VARCHAR (5)
    , country_name VARCHAR (100)
    , year smallint
    , population bigint
  ) AS r
WHERE
  country_name = 'Luxembourg' AND year = 2017
```

Vous pouvez spécifier le schéma au moment de la compilation de la requête.
Pour plus d’exemples, consultez la procédure indiquant comment [interroger des fichiers CSV](sql/query-single-csv-file.md).

## <a name="query-parquet-files"></a>Interroger des fichiers Parquet

L’exemple suivant montre les fonctionnalités d’inférence automatique du schéma pour l’interrogation des fichiers Parquet. Il retourne le nombre de lignes en septembre 2017 sans spécifier de schéma.

> [!NOTE]
> Vous n’avez pas besoin de spécifier de colonnes dans la clause `OPENROWSET WITH` lors de la lecture de fichiers Parquet. Dans ce cas, le pool SQL serverless utilise les métadonnées dans le fichier Parquet et lie les colonnes par nom.

```sql
SELECT COUNT_BIG(*)
FROM OPENROWSET
  (
      BULK 'parquet/taxi/year=2017/month=9/*.parquet',
      DATA_SOURCE = 'SqlOnDemandDemo',
      FORMAT='PARQUET'
  ) AS nyc
```

Pour plus d’informations sur l’interrogation des fichiers Parquet, cliquez [ici](sql/query-parquet-files.md).

## <a name="query-json-files"></a>Interroger des fichiers JSON

### <a name="json-sample-file"></a>Exemple de fichier JSON

Les fichiers sont stockés dans un dossier *books* au sein d’un conteneur *json*, et contiennent une entrée de livre unique avec la structure suivante :

```json
{  
   "_id":"ahokw88",
   "type":"Book",
   "title":"The AWK Programming Language",
   "year":"1988",
   "publisher":"Addison-Wesley",
   "authors":[  
      "Alfred V. Aho",
      "Brian W. Kernighan",
      "Peter J. Weinberger"
   ],
   "source":"DBLP"
}
```

### <a name="query-json-files"></a>Interroger des fichiers JSON

La requête suivante montre comment utiliser [JSON_VALUE](/sql/t-sql/functions/json-value-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) pour récupérer des valeurs scalaires (titre, éditeur) à partir d’un livre intitulé *Probabilistic and Statistical Methods in Cryptology, An Introduction by Selected articles* :

```sql
SELECT
    JSON_VALUE(jsonContent, '$.title') AS title
  , JSON_VALUE(jsonContent, '$.publisher') as publisher
  , jsonContent
FROM OPENROWSET
  (
      BULK 'json/books/*.json',
      DATA_SOURCE = 'SqlOnDemandDemo'
    , FORMAT='CSV'
    , FIELDTERMINATOR ='0x0b'
    , FIELDQUOTE = '0x0b'
    , ROWTERMINATOR = '0x0b'
  )
WITH
  ( jsonContent varchar(8000) ) AS [r]
WHERE
  JSON_VALUE(jsonContent, '$.title') = 'Probabilistic and Statistical Methods in Cryptology, An Introduction by Selected Topics'
```

> [!IMPORTANT]
> Nous lisons l’intégralité du fichier JSON sous la forme d’une seule ligne/colonne. Ainsi, FIELDTERMINATOR, FIELDQUOTE et ROWTERMINATOR sont définis sur 0x0B, car nous ne pensons pas les trouver dans le fichier.

## <a name="next-steps"></a>Étapes suivantes

Vous êtes maintenant prêt à poursuivre avec les articles suivants :

- [Interroger un fichier CSV](sql/query-single-csv-file.md)
- [Interroger des dossiers et plusieurs fichiers CSV](sql/query-folders-multiple-csv-files.md)
- [Interroger des fichiers spécifiques](sql/query-specific-files.md)
- [Interroger des fichiers Parquet](sql/query-parquet-files.md)
- [Interroger des types imbriqués Parquet](sql/query-parquet-nested-types.md)
- [Interroger des fichiers JSON](sql/query-json-files.md)
- [Création et utilisation de vues](sql/create-use-views.md)
- [Création et utilisation de tables externes](sql/create-use-external-tables.md)
- [Conserver le résultat d’une requête dans le stockage Azure](sql/create-external-table-as-select.md)
- [Interroger un fichier CSV](sql/query-single-csv-file.md)
