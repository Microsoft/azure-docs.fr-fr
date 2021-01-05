---
title: Concevoir un processus ELT au lieu d’un processus ETL
description: Implémentez des stratégies flexibles de chargement de données pour les pools SQL dédiés au sein d’Azure Synapse Analytics.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/20/2020
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: 64ba24eb0eab581310122908fc05d1d671ac1d40
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96531571"
---
# <a name="data-loading-strategies-for-dedicated-sql-pool-in-azure-synapse-analytics"></a>Stratégies de chargement de données pour un pool SQL dédié dans Azure Synapse Analytics

Les pools SQL dédiés SMP traditionnels utilisent un processus ELT (Extract, Load, and Transform – Extraire, charger et transformer) pour le chargement des données. Synapse SQL, au sein d’Azure Synapse Analytics, utilise une architecture de traitement de requêtes distribuées qui tire parti de la scalabilité et de la flexibilité des ressources de calcul et de stockage.

L’utilisation d’un processus ELT s’appuie sur ses capacités de traitement de requêtes distribuées intégrées et d’éliminer les ressources nécessaires à la transformation des données avant le chargement.

Bien que les pools SQL dédiés prennent en charge de nombreuses méthodes de chargement, notamment des options SQL Server bien connues, comme [bcp](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) et l’[API SqlBulkCopy](/dotnet/api/system.data.sqlclient.sqlbulkcopy?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json), le moyen le plus rapide et le plus scalable pour charger des données est d’utiliser des tables externes PolyBase et l’[instruction COPY](/sql/t-sql/statements/copy-into-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

Avec PolyBase et l’instruction COPY, vous pouvez accéder à des données externes stockées dans Stockage Blob Azure ou dans Azure Data Lake Store via le langage T-SQL. Pour une plus grande flexibilité lors du chargement, nous vous recommandons d’utiliser l’instruction COPY.


## <a name="what-is-elt"></a>ELT, qu’est-ce que ça veut dire ?

ELT (Extract, Load, and Transform – Extraire, charger et transformer) est un processus par lequel des données sont extraites d’un système source, chargées dans un pool SQL dédié, puis transformées.

Les étapes de base pour implémenter ELT sont les suivantes :

1. Extrayez les données sources dans des fichiers texte.
2. Placez les données dans le stockage Blob Azure ou Azure Data Lake Store.
3. Préparez les données pour le chargement.
4. Chargez les données dans des tables de mise en lots avec PolyBase ou la commande COPY.
5. Transformez les données.
6. Insérez les données dans des tables de production.

Pour suivre un tutoriel sur le chargement, consultez [Chargement de données à partir du Stockage Blob Azure](load-data-from-azure-blob-storage-using-polybase.md).

## <a name="1-extract-the-source-data-into-text-files"></a>1. Extraire les données sources dans des fichiers texte

L’extraction des données à partir de votre système source dépend de l’emplacement de stockage. L’objectif est de déplacer les données vers des fichiers texte délimités ou CSV pris en charge.

### <a name="supported-file-formats"></a>Formats de fichiers pris en charge

Avec PolyBase et l’instruction COPY, vous pouvez charger des données à partir de fichiers texte ou CSV encodés au format UTF-8 et UTF-16. En plus des fichiers texte délimités ou CSV, il charge des données à partir de formats de fichiers Hadoop, comme ORC et Parquet. PolyBase et l’instruction COPY peuvent aussi charger des données à partir de fichiers compressés Gzip et Snappy.

Le format ASCII étendu de largeur fixe et les formats imbriqués, comme WinZip ou XML, ne sont pas pris en charge. Si vous exportez à partir de SQL Server, vous pouvez utiliser l’[outil en ligne de commande bcp](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour exporter les données dans des fichiers texte délimités.

## <a name="2-land-the-data-into-azure-blob-storage-or-azure-data-lake-store"></a>2. Placer les données dans le stockage Blob Azure ou Azure Data Lake Store

Pour charger les données dans Stockage Azure, vous pouvez les déplacer dans [Stockage Blob Azure](../../storage/blobs/storage-blobs-introduction.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) ou dans [Azure Data Lake Store Gen2](../../data-lake-store/data-lake-store-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Quel que soit l’emplacement choisi, les données doivent être stockées dans des fichiers texte. PolyBase et l’instruction COPY peuvent charger depuis l’un ou l’autre emplacement.

Voici des outils et services que vous pouvez utiliser pour déplacer des données dans le stockage Azure.

- Le service [Azure ExpressRoute](../../expressroute/expressroute-introduction.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) améliore le débit, les performances et la prévisibilité du réseau. ExpressRoute est un service qui achemine vos données via une connexion privée dédiée vers Azure. Les connexions ExpressRoute n’acheminent pas vos données via le réseau Internet public. Elles offrent davantage de fiabilité, des vitesses supérieures, des latences inférieures et une sécurité renforcée par rapport aux connexions publiques sur Internet.
- L’[utilitaire AZCopy](../../storage/common/storage-choose-data-transfer-solution.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) déplace les données vers le stockage Azure via l’Internet public. Il fonctionne si la taille de vos données ne dépasse pas les 10 To. Pour effectuer des chargements réguliers avec AZCopy, testez la vitesse du réseau pour voir si elle est acceptable.
- [Azure Data Factory (ADF)](../../data-factory/introduction.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) dispose d’une passerelle que vous pouvez installer sur votre serveur local. Ensuite, vous pouvez créer un pipeline pour déplacer des données à partir de votre serveur local vers le stockage Azure. Pour utiliser Data Factory avec des pools SQL dédiés, consultez [Chargement de données pour des pools SQL dédiés](../../data-factory/load-azure-sql-data-warehouse.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

## <a name="3-prepare-the-data-for-loading"></a>3. Préparer les données pour le chargement

Avant de pouvoir charger les données de votre compte de stockage, vous devrez peut-être les préparer et les nettoyer. Vous pouvez préparer vos données pendant qu’elles sont dans la source, pendant que vous exportez les données dans des fichiers texte ou une fois que les données se trouvent dans le stockage Azure.  Plus vous les préparez tôt, plus ce sera facile.  

### <a name="define-the-tables"></a>Définir les tables

Vous devez d’abord définir la ou les tables que vous chargez dans votre pool SQL dédié lors de l’utilisation de l’instruction COPY.

Si vous utilisez PolyBase, vous devez définir des tables externes dans votre pool SQL dédié avant le chargement. PolyBase utilise des tables externes pour définir les données dans le stockage Azure et y accéder. Une table externe est similaire à une vue de base de données. La table externe contient le schéma de table et pointe vers les données stockées en dehors du pool SQL dédié.

La définition des tables externes implique de spécifier la source des données, le format des fichiers texte et les définitions de la table. Les articles de référence sur la syntaxe T-SQL dont vous aurez besoin sont les suivants :

- [CREATE EXTERNAL DATA SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [CREATE EXTERNAL FILE FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [CREATE EXTERNAL TABLE](/sql/t-sql/statements/create-external-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

Utilisez le mappage de type de données SQL suivant lors du chargement des fichiers Parquet :

|                         Type Parquet                         |   Type logique Parquet (annotation)   |  Type de données SQL   |
| :----------------------------------------------------------: | :-----------------------------------: | :--------------: |
|                           BOOLEAN                            |                                       |       bit        |
|                     BINARY / BYTE_ARRAY                      |                                       |    varbinary     |
|                            DOUBLE                            |                                       |      float       |
|                            FLOAT                             |                                       |       real       |
|                            INT32                             |                                       |       int        |
|                            INT64                             |                                       |      bigint      |
|                            INT96                             |                                       |    datetime2     |
|                     FIXED_LEN_BYTE_ARRAY                     |                                       |      binary      |
|                            BINARY                            |                 UTF8                  |     NVARCHAR     |
|                            BINARY                            |                STRING                 |     NVARCHAR     |
|                            BINARY                            |                 ENUM                  |     NVARCHAR     |
|                            BINARY                            |                 UUID                  | UNIQUEIDENTIFIER |
|                            BINARY                            |                DECIMAL                |     Décimal      |
|                            BINARY                            |                 JSON                  |  nvarchar(MAX)   |
|                            BINARY                            |                 BSON                  |  varbinary(max)  |
|                     FIXED_LEN_BYTE_ARRAY                     |                DECIMAL                |     Décimal      |
|                          BYTE_ARRAY                          |               INTERVAL                |  varchar(max),   |
|                            INT32                             |             INT(8, true)              |     SMALLINT     |
|                            INT32                             |            INT(16, true)            |     SMALLINT     |
|                            INT32                             |             INT(32, true)             |       int        |
|                            INT32                             |            INT(8, false)            |     TINYINT      |
|                            INT32                             |            INT(16, false)             |       int        |
|                            INT32                             |           INT(32, false)            |      bigint      |
|                            INT32                             |                 DATE                  |       Date       |
|                            INT32                             |                DECIMAL                |     Décimal      |
|                            INT32                             |            TIME (MILLIS )             |       time       |
|                            INT64                             |            INT(64, true)            |      bigint      |
|                            INT64                             |           INT(64, false )            |  decimal(20,0)   |
|                            INT64                             |                DECIMAL                |     Décimal      |
|                            INT64                             |         TIME (MILLIS)                 |       time       |
|                            INT64                             | TIMESTAMP   (MILLIS)                  |    datetime2     |
| [Type complexe](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fgithub.com%2Fapache%2Fparquet-format%2Fblob%2Fmaster%2FLogicalTypes.md%23lists&data=02\|01\|kevin%40microsoft.com\|19f74d93f5ca45a6b73c08d7d7f5f111\|72f988bf86f141af91ab2d7cd011db47\|1\|0\|637215323617803168&sdata=6Luk047sK26ijTzfvKMYc%2FNu%2Fz0AlLCX8lKKTI%2F8B5o%3D&reserved=0) |                 Liste                  |   varchar(max)   |
| [Type complexe](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fgithub.com%2Fapache%2Fparquet-format%2Fblob%2Fmaster%2FLogicalTypes.md%23maps&data=02\|01\|kevin%40microsoft.com\|19f74d93f5ca45a6b73c08d7d7f5f111\|72f988bf86f141af91ab2d7cd011db47\|1\|0\|637215323617803168&sdata=FiThqXxjgmZBVRyigHzfh5V7Z%2BPZHjud2IkUUM43I7o%3D&reserved=0) |                  MAP                  |   varchar(max)   |

>[!IMPORTANT] 
>- Les pools dédiés SQL ne prennent actuellement pas en charge les types de données Parquet avec précision MICROS ou NANOS. 
>- Vous pouvez rencontrer l’erreur suivante si les types ne correspondent pas entre Parquet et SQL ou si vous avez des types de données Parquet non pris en charge : **« HdfsBridge::recordReaderFillBuffer - Unexpected error encountered filling record reader buffer: ClassCastException: ... »**
>- Le chargement d’une valeur en dehors de la plage 0-127 dans une colonne tinyint pour le format de fichier ORC et Parquet n’est pas pris en charge.

Pour un exemple de création d’objets externes, consultez[Créer des tables externes](https://docs.microsoft.com/azure/synapse-analytics/sql/develop-tables-external-tables?tabs=sql-pool).

### <a name="format-text-files"></a>Formater les fichiers texte

Si vous utilisez PolyBase, les objets externes définis doivent aligner les lignes des fichiers texte avec la table externe et la définition du format de fichier. Les données de chaque ligne du fichier texte doivent être alignées avec la définition de la table.
Pour formater les fichiers texte :

- Si vos données proviennent d’une source non relationnelle, vous devez les transformer en lignes et en colonnes. Que les données proviennent d’une source relationnelle ou non relationnelle, elles doivent être transformées pour être alignées avec les définitions des colonnes pour la table dans laquelle vous souhaitez les charger.
- Formatez les données dans le fichier texte pour les aligner avec les types de colonnes et de données dans la table de destination. Un décalage entre les types de données dans les fichiers texte externes et la table du pool SQL dédié cause le rejet des lignes lors du chargement.
- Séparez les champs dans le fichier texte à l’aide d’une marque de fin.  Assurez-vous d’utiliser un caractère ou une séquence de caractères qui ne se trouve pas dans votre source de données. Utilisez la marque de fin que vous avez spécifiée avec l’instruction [CREATE EXTERNAL FILE FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="4-load-the-data-using-polybase-or-the-copy-statement"></a>4. Charger les données à l’aide de PolyBase ou de l’instruction COPY

Il est recommandé de charger des données dans une table de mise en lots. Les tables de mise en lots vous permettent de gérer les erreurs sans interférer avec les tables de production. Une table de mise en lots vous donne également la possibilité d’utiliser l’architecture de traitement parallèle de pool SQL dédié pour transformer les données avant de les insérer dans des tables de production.

### <a name="options-for-loading"></a>Options de chargement

Pour charger des données, vous pouvez utiliser l’une des options de chargement suivantes :

- L’[instruction COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest) est l’utilitaire de chargement recommandé, car il vous permet de charger des données de manière fluide et flexible. L’instruction dispose de nombreuses fonctionnalités de chargement supplémentaires non fournies par PolyBase. 
- [Polybase avec T-SQL](load-data-from-azure-blob-storage-using-polybase.md) vous oblige à définir des objets de données externes.
- [PolyBase et l’instruction COPY avec Azure Data Factory (ADF)](../../data-factory/load-azure-sql-data-warehouse.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) est un autre outil d’orchestration.  Il définit un pipeline et planifie les travaux.
- [Polybase avec SSIS](/sql/integration-services/load-data-to-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) fonctionne bien lorsque vos données sources se trouvent dans SQL Server. SSIS définit le mappage de la table « source vers destination » et orchestre aussi le chargement. Si vous disposez déjà de packages SSIS, vous pouvez modifier les packages pour travailler avec le nouvel entrepôt de données de destination.
- [PolyBase avec Azure Databricks](../../azure-databricks/databricks-extract-load-sql-data-warehouse.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) transfère les données d’une table vers une trame de données Databricks et/ou écrit des données d’une trame de données Databricks dans une table à l’aide de la technologie PolyBase.

### <a name="other-loading-options"></a>Autres options de chargement

En plus de PolyBase et de l’instruction COPY, vous pouvez utiliser [bcp](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) ou l’[API SqlBulkCopy](/dotnet/api/system.data.sqlclient.sqlbulkcopy?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). L’outil bcp charge directement dans la base de données sans passer par Stockage Blob Azure et est destiné uniquement aux petits chargements.

> [!NOTE]
> Les performances de chargement de ces options sont beaucoup plus lentes qu’avec PolyBase et l’instruction COPY.

## <a name="5-transform-the-data"></a>5. Transformer les données

Pendant que les données se trouvent dans la table de mise en lots, effectuez les transformations requises par votre charge de travail. Déplacez ensuite les données dans une table de production.

## <a name="6-insert-the-data-into-production-tables"></a>6. Insérer les données dans des tables de production

L’instruction INSERT INTO... SELECT déplace les données depuis la table de mise en lots vers la table permanente.

Lorsque vous concevez un processus ETL, commencez par exécuter le processus sur un petit échantillon. Essayez d’extraire 1 000 lignes de la table dans un fichier, déplacez-le vers Azure, puis essayez de le charger dans une table de mise en lots.

## <a name="partner-loading-solutions"></a>Solutions de chargement des partenaires

La plupart de nos partenaires proposent des solutions de chargement. Pour en savoir plus, consultez la liste de nos [partenaires de solutions](sql-data-warehouse-partner-business-intelligence.md).

## <a name="next-steps"></a>Étapes suivantes

Pour le chargement des instructions, consultez [Guidance for loading data](guidance-for-loading-data.md) (Conseils relatifs au chargement des données).
