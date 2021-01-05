---
title: Interroger un stockage de données avec un pool SQL serverless
description: Cet article explique comment interroger Stockage Azure en utilisant la ressource de pool SQL serverless au sein d’Azure Synapse Analytics.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: sql
ms.date: 04/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 967250cf29d1f0248f296cb545a764bd8e611773
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462657"
---
# <a name="query-storage-files-with-serverless-sql-pool-in-azure-synapse-analytics"></a>Interroger des fichiers de stockage avec un pool SQL serverless dans Azure Synapse Analytics

Un pool SQL serverless vous permet d’interroger des données dans votre lac de données. Ce service offre une surface d’exposition de requête T-SQL qui prend en charge les requêtes de données semi-structurées et non structurées. Pour l’interrogation, les aspects T-SQL suivants sont pris en charge :

- Surface d’exposition [SELECT](/sql/t-sql/queries/select-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) complète, y compris la majorité des [fonctions et opérateurs SQL](overview-features.md).
- CREATE EXTERNAL TABLE AS SELECT ([CETAS](develop-tables-cetas.md)) crée une [table externe](develop-tables-external-tables.md), puis exporte, en parallèle, les résultats d’une instruction SELECT Transact-SQL vers le stockage Azure.

Pour plus d’informations sur ce qui est pris en charge ou non, consultez l’article [Vue d’ensemble du pool SQL serverless](on-demand-workspace-overview.md) ou les articles suivants :
- [Développer un accès au stockage](develop-storage-files-overview.md), pour apprendre à utiliser une [table externe](develop-tables-external-tables.md) et la [fonction OPENROWSET](develop-openrowset.md) afin de lire des données à partir du stockage.
- [Contrôler l’accès au stockage](develop-storage-files-storage-access-control.md), pour voir comment permettre à Synapse SQL d’accéder au stockage à l’aide de l’authentification SAS ou de l’identité managée de l’espace de travail.

## <a name="overview"></a>Vue d’ensemble

Pour faciliter la prise en charge de l’interrogation sur place des données qui se trouvent dans des fichiers de Stockage Azure, le pool SQL serverless utilise la fonction [OPENROWSET](develop-openrowset.md) avec des fonctionnalités supplémentaires :

- [Interroger plusieurs fichiers ou dossiers](#query-multiple-files-or-folders)
- [Format de fichier PARQUET](#query-parquet-files)
- [Interroger un texte CSV et délimité (marque de fin de champ, marque de fin de ligne, caractère d’échappement)](#query-csv-files)
- [Lire un sous-ensemble choisi de colonnes](#read-a-chosen-subset-of-columns)
- [Inférence de schéma](#schema-inference)
- [Fonction filename](#filename-function)
- [Fonction filepath](#filepath-function)
- [Utilisation de types complexes et de structures de données imbriquées ou répétées](#work-with-complex-types-and-nested-or-repeated-data-structures)

## <a name="query-parquet-files"></a>Interroger des fichiers PARQUET

Pour interroger les données sources Parquet, utilisez FORMAT = 'PARQUET'

```syntaxsql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net//mycontainer/mysubfolder/data.parquet', FORMAT = 'PARQUET') 
WITH (C1 int, C2 varchar(20), C3 varchar(max)) as rows
```

Pour obtenir des exemples d’utilisation, consultez l’article [Interroger des fichiers Parquet](query-parquet-files.md).

## <a name="query-csv-files"></a>Interroger des fichiers CSV

Pour interroger les données sources CSV, utilisez FORMAT = 'CSV'. Vous pouvez spécifier le schéma du fichier CSV dans le cadre de la fonction `OPENROWSET` quand vous interrogez des fichiers CSV :

```sql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net/mycontainer/mysubfolder/data.csv', FORMAT = 'CSV', PARSER_VERSION='2.0') 
WITH (C1 int, C2 varchar(20), C3 varchar(max)) as rows
```

Vous pouvez utiliser des options supplémentaires pour ajuster les règles d’analyse au format CSV personnalisé :
- ESCAPE_CHAR = 'char' Spécifie le caractère dans le fichier qui est utilisé pour se placer lui-même dans une séquence d’échappement ainsi que toutes les valeurs de délimiteur dans le fichier. Si le caractère d’échappement est suivi d’une valeur autre que lui-même, ou que l’une des valeurs de délimiteur, le caractère d’échappement est supprimé lors de la lecture de la valeur.
Le paramètre ESCAPE_CHAR est appliqué, que FIELDQUOTE soit ou non activé. Il ne sera pas utilisé comme caractère d’échappement devant le caractère de délimitation. Le caractère de délimitation doit être placé dans une séquence d’échappement avec un autre caractère de délimitation. Le caractère de délimitation peut apparaître dans la valeur de colonne seulement si la valeur est encapsulée avec des caractères de délimitation.
- FIELDTERMINATOR = 'field_terminator' Spécifie la marque de fin de champ à utiliser. La virgule («  **,**  ») est la marque de fin de champ par défaut.
- ROWTERMINATOR = 'row_terminator' Spécifie la marque de fin de ligne à utiliser. Par défaut, il s’agit d’un caractère de nouvelle ligne :  **\r\n**.

## <a name="file-schema"></a>Schéma de fichier

Le langage SQL dans Synapse SQL vous permet de définir le schéma du fichier dans le cadre de la fonction `OPENROWSET` et de lire la totalité ou un sous-ensemble des colonnes, ou il tente de déterminer automatiquement les types de colonne à partir du fichier à l’aide de l’inférence de schéma.

### <a name="read-a-chosen-subset-of-columns"></a>Lire un sous-ensemble choisi de colonnes

Pour spécifier les colonnes que vous souhaitez lire, vous pouvez fournir une clause WITH facultative dans votre instruction `OPENROWSET`.

- S’il y a des fichiers de données CSV, pour lire toutes les colonnes, indiquez les noms des colonnes et leur type de données. Si vous souhaitez un sous-ensemble de colonnes, utilisez des nombres ordinaux pour sélectionner les colonnes des fichiers de données d’origine par ordinal. Les colonnes sont ainsi liées par la désignation ordinale.
- S’il existe des fichiers de données Parquet, fournissez des noms de colonne qui correspondent aux noms des colonnes dans les fichiers de données d’origine. Les colonnes sont alors liées par nom.

```sql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net/mycontainer/mysubfolder/data.parquet', FORMAT = 'PARQUET') 
WITH (
      C1 int, 
      C2 varchar(20),
      C3 varchar(max)
) as rows
```

Pour chaque colonne, vous devez spécifier le nom et le type de colonne dans la clause `WITH`.
Pour obtenir des exemples, reportez-vous à [Lire des fichiers CSV sans spécifier toutes les colonnes](query-single-csv-file.md#return-a-subset-of-columns).

## <a name="schema-inference"></a>Inférence de schéma

En omettant la clause WITH dans l’instruction `OPENROWSET`, vous pouvez demander au service de détecter automatiquement (déduire) le schéma à partir des fichiers sous-jacents.

> [!NOTE]
> Cette possibilité ne fonctionne actuellement que pour le format de fichier PARQUET.

```sql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net/mycontainer/mysubfolder/data.parquet', FORMAT = 'PARQUET') 
```

Assurez-vous que des [types de données déduits appropriés](best-practices-sql-on-demand.md#check-inferred-data-types) sont utilisés pour des performances optimales. 

## <a name="query-multiple-files-or-folders"></a>Interroger plusieurs fichiers ou dossiers

Pour exécuter une requête T-SQL sur un ensemble de fichiers, à l’intérieur d’un dossier ou d’un ensemble de dossiers, tout en les traitant en tant qu’entité ou ensemble de lignes unique, fournissez un chemin de dossier ou de modèle (à l’aide de caractères génériques) sur un ensemble de fichiers ou de dossiers.

Les règles suivantes s’appliquent :

- Les modèles peuvent apparaître comme faisant partie d’un chemin de répertoire, ou dans un nom de fichier.
- Plusieurs modèles peuvent apparaître dans la même étape de répertoire ou le même nom de fichier.
- S’il y a plusieurs caractères génériques, les fichiers présents dans tous les chemins correspondants sont inclus dans le jeu de fichiers obtenu.

```sql
SELECT * FROM
OPENROWSET( BULK N'https://myaccount.dfs.core.windows.net/myroot/*/mysubfolder/*.parquet', FORMAT = 'PARQUET' ) as rows
```

Pour obtenir des exemples d’utilisation, reportez-vous à l’article [Interroger des dossiers et plusieurs fichiers](query-folders-multiple-csv-files.md).

## <a name="file-metadata-functions"></a>Fonctions de métadonnées de fichier

### <a name="filename-function"></a>Fonction filename

Cette fonction retourne le nom du fichier d’où provient la ligne. 

Pour interroger des fichiers spécifiques, lisez la section Filename dans l’article [Interroger des fichiers spécifiques](query-specific-files.md#filename).

Le type de données de retour est nvarchar (1024). Pour des performances optimales, castez toujours le résultat de la fonction filename en un type de données approprié. Si vous utilisez le type de données caractères, assurez-vous que la longueur appropriée est utilisée.

### <a name="filepath-function"></a>Fonction filepath

Cette fonction retourne un chemin complet ou une partie de chemin :

- En cas d’appel sans paramètre, elle retourne le chemin complet du fichier d’où est issue une ligne.
- En cas d’appel avec paramètre, elle retourne une partie du chemin qui correspond au caractère générique, à la position spécifiée dans le paramètre. Par exemple, la valeur de paramètre 1 retourne une partie du chemin qui correspond au premier caractère générique.

Pour plus d’informations, consultez la section Filepath dans l’article [Interroger des fichiers spécifiques](query-specific-files.md#filepath).

Le type de données de retour est nvarchar (1024). Pour des performances optimales, castez toujours le résultat de la fonction filepath en un type de données approprié. Si vous utilisez le type de données caractères, assurez-vous que la longueur appropriée est utilisée.

## <a name="work-with-complex-types-and-nested-or-repeated-data-structures"></a>Utiliser des types complexes et des structures de données imbriquées ou répétées

Pour faciliter la prise en charge des données stockées dans des types de données imbriqués ou répétés, comme dans des fichiers [Parquet](https://github.com/apache/parquet-format/blob/master/LogicalTypes.md#nested-types), le pool SQL serverless comprend les extensions suivantes.

#### <a name="project-nested-or-repeated-data"></a>Projeter des données imbriquées ou répétées

Pour projeter des données, exécutez une instruction SELECT sur le fichier Parquet qui contient des colonnes de types de données imbriquées. En sortie, les valeurs imbriquées sont sérialisées en JSON et retournées sous la forme d’un type de données SQL varchar(8000).

```sql
    SELECT * FROM
    OPENROWSET
    (   BULK 'unstructured_data_path' ,
        FORMAT = 'PARQUET' )
    [AS alias]
```

Pour plus d’informations, reportez-vous à la section Projeter des données imbriquées ou répétées dans l’article [Interroger des types imbriqués Parquet](query-parquet-nested-types.md#project-nested-or-repeated-data).

#### <a name="access-elements-from-nested-columns"></a>Accéder aux éléments à partir de colonnes imbriquées

Pour accéder aux éléments imbriqués à partir d’une colonne imbriquée, telle que Struct, utilisez la « notation par points » pour concaténer les noms de champ dans le chemin. Indiquez le chemin en tant que column_name dans la clause WITH de la fonction `OPENROWSET`.

L’exemple de fragment de syntaxe est le suivant :

```syntaxsql
    OPENROWSET
    (   BULK 'unstructured_data_path' ,
        FORMAT = 'PARQUET' )
    WITH ({'column_name' 'column_type',})
    [AS alias]
    'column_name' ::= '[field_name.] field_name'
```

Par défaut, la fonction `OPENROWSET` associe le chemin et le nom de champ de la source aux noms de colonne fournis dans la clause WITH. Les éléments contenus à des niveaux d’imbrication différents dans le même fichier Parquet source sont accessibles via la clause WITH.

**Valeurs retournées**

- La fonction retourne une valeur scalaire, telle que int, decimal et varchar, à partir de l’élément spécifié, et dans le chemin spécifié, pour tous les types Parquet qui ne sont pas dans le groupe Type imbriqué.
- Si le chemin pointe vers un élément qui est d’un type imbriqué, la fonction retourne un fragment JSON à partir de l’élément du plus haut niveau dans le chemin spécifié. Le fragment JSON est de type varchar(8000).
- Si la propriété est introuvable au niveau de l’élément column_name spécifié, la fonction retourne une erreur.
- Si la propriété est introuvable au niveau de l’élément column_name précisé, en fonction du [mode de chemin](/sql/relational-databases/json/json-path-expressions-sql-server?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest#PATHMODE), la fonction retourne une erreur en mode strict, ou retourne la valeur Null en mode lax.

Pour obtenir des exemples de requêtes, consultez la section Accéder aux éléments à partir de colonnes imbriquées dans l’article [Interroger des types imbriqués Parquet](query-parquet-nested-types.md#read-properties-from-nested-object-columns).

#### <a name="access-elements-from-repeated-columns"></a>Accéder aux éléments à partir de colonnes répétées

Pour accéder aux éléments d’une colonne répétée, par exemple un élément d’un tableau ou d’un plan, utilisez la fonction [JSON_VALUE](/sql/t-sql/functions/json-value-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) pour chaque élément scalaire que vous devez projeter, et fournissez les informations suivantes :

- Colonne imbriquée ou répétée, en tant que premier paramètre
- Un [chemin JSON](/sql/relational-databases/json/json-path-expressions-sql-server?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) qui spécifie la propriété ou l’élément auquel accéder, en tant que second paramètre

Pour accéder à des éléments non scalaires à partir d’une colonne répétée, utilisez la fonction [JSON_QUERY](/sql/t-sql/functions/json-query-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) pour chaque élément non scalaire que vous devez projeter, et fournissez les informations suivantes :

- Colonne imbriquée ou répétée, en tant que premier paramètre
- Un [chemin JSON](/sql/relational-databases/json/json-path-expressions-sql-server?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) qui spécifie la propriété ou l’élément auquel accéder, en tant que second paramètre

Consultez le fragment de syntaxe ci-dessous :

```syntaxsql
    SELECT
       { JSON_VALUE (column_name, path_to_sub_element), }
       { JSON_QUERY (column_name [ , path_to_sub_element ]), )
    FROM
    OPENROWSET
    (   BULK 'unstructured_data_path' ,
        FORMAT = 'PARQUET' )
    [AS alias]
```

Vous trouverez des exemples de requêtes permettant d’accéder à des éléments depuis les colonnes répétées dans l’article [Interroger des types imbriqués Parquet](query-parquet-nested-types.md#access-elements-from-repeated-columns).

## <a name="query-samples"></a>Exemples de requête

Vous pouvez en découvrir plus sur l’interrogation des différents types de données à l’aide des exemples de requêtes.

### <a name="tools"></a>Outils

Les outils dont vous avez besoin pour émettre des requêtes :
    - Azure Synapse Studio 
    - Azure Data Studio
    - SQL Server Management Studio

### <a name="demo-setup"></a>Configuration de la démonstration

La première étape consiste à **créer la base de données** dans laquelle seront exécutées les requêtes. Ensuite, vous initialiserez les objets en exécutant le [script d’installation](https://github.com/Azure-Samples/Synapse/blob/master/SQL/Samples/LdwSample/SampleDB.sql) sur cette base de données. 

Ce script d’installation crée les sources de données, les informations d’identification limitées à la base de données et les formats de fichiers externes utilisés pour la lecture des données dans ces exemples.

> [!NOTE]
> Les bases de données sont utilisées uniquement pour voir les métadonnées, et non pour les données réelles.  Notez le nom de la base de données que vous utilisez, vous en aurez besoin plus tard.

```sql
CREATE DATABASE mydbname;
```

### <a name="provided-demo-data"></a>Données de démonstration fournies

Les données de démonstration contiennent les jeux de données suivants :

- Taxi de New York - Enregistrements de trajet de taxi jaune - partie du jeu de données publiques de New York au format CSV et Parquet
- Jeu de données Population au format CSV
- Exemples de fichiers Parquet avec colonnes imbriquées
- Livres au format JSON

| Chemin d’accès du dossier                                                  | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| /csv/                                                        | Dossier parent des données au format CSV                         |
| /csv/population/<br />/csv/population-unix/<br />/csv/population-unix-hdr/<br />/csv/population-unix-hdr-escape<br />/csv/population-unix-hdr-quoted | Dossiers contenant les fichiers de données Population dans différents formats CSV. |
| /csv/taxi/                                                   | Dossier contenant les fichiers de données publiques de New York au format CSV              |
| /parquet/                                                    | Dossier parent des données au format Parquet                     |
| /parquet/taxi                                                | Fichiers de données publiques de New York au format Parquet, partitionnés par année et par mois à l’aide du schéma de partitionnement Hive/Hadoop. |
| /parquet/nested/                                             | Exemples de fichiers Parquet avec colonnes imbriquées                     |
| /json/                                                       | Dossier parent des données au format JSON                        |
| /json/books/                                                 | Fichiers JSON avec des données de livres                                   |


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interrogation de différents types de fichiers ainsi que sur la création et l’utilisation de vues, consultez les articles suivants :

- [Interroger des fichiers CSV](query-single-csv-file.md)
- [Interroger des fichiers Parquet](query-parquet-files.md)
- [Interroger des fichiers JSON](query-json-files.md)
- [Interroger des valeurs imbriquées](query-parquet-nested-types.md)
- [Interroger des dossiers et plusieurs fichiers CSV](query-folders-multiple-csv-files.md)
- [Utiliser des métadonnées de fichier dans les requêtes](query-specific-files.md)
- [Créer et utiliser des vues](create-use-views.md)
