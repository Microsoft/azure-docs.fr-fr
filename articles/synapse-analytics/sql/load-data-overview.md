---
title: Concevoir une stratégie de chargement de données PolyBase pour un pool SQL dédié
description: Au lieu d’ETL, concevez un processus d’extraction, de chargement et de transformation (ELT) pour charger les données avec un pool SQL dédié.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: a57abd080bdbbaefbe07258a2b241c093dc8c441
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93308746"
---
# <a name="design-a-polybase-data-loading-strategy-for-dedicated-sql-pool-in-azure-synapse-analytics"></a>Concevoir une stratégie de chargement de données PolyBase pour un pool SQL dédié dans Azure Synapse Analytics

Les entrepôts de données SMP traditionnels utilisent un processus d’extraction, transformation et chargement (ETL) pour le chargement des données. Un pool Azure SQL est une architecture de traitement massivement parallèle (MPP) qui tire parti de la scalabilité et de la flexibilité des ressources de calcul et de stockage. L'utilisation d'un processus ELT permet de tirer parti des capacités de traitement de requêtes distribuées intégrées et d'éliminer les ressources nécessaires à la transformation des données avant le chargement.

Bien que le pool SQL prenne en charge de nombreuses méthodes de chargement, notamment des options non-Polybase telles que les API BCP et SQL BulkCopy, le moyen le plus rapide et le plus scalable de charger des données consiste à utiliser PolyBase.  PolyBase est une technologie qui accède aux données externes stockées dans Stockage Blob Azure ou Azure Data Lake Store par le biais du langage T-SQL.

> [!VIDEO https://www.youtube.com/embed/l9-wP7OdhDk]

## <a name="extract-load-and-transform-elt"></a>Extraire, charger et transformer (ELT)

ELT (Extract, Load, and Transform - Extraire, charger et transformer) est un processus par lequel des données sont extraites d’un système source, chargées dans un entrepôt de données, puis transformées.

Les étapes de base pour implémenter un processus ELT PolyBase pour un pool SQL dédié sont les suivantes :

1. Extrayez les données sources dans des fichiers texte.
2. Placez les données dans le stockage Blob Azure ou Azure Data Lake Store.
3. Préparez les données pour le chargement.
4. Chargez les données dans les tables de mise en lots du pool SQL dédié à l’aide de PolyBase.
5. Transformez les données.
6. Insérez les données dans des tables de production.

Pour suivre un tutoriel sur le chargement, consultez l’article [Utiliser PolyBase pour charger des données du Stockage Blob Azure dans Azure Synapse Analytics](../sql-data-warehouse/load-data-from-azure-blob-storage-using-polybase.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

Pour en savoir plus, voir l’article de blog [Loading patterns](https://blogs.msdn.microsoft.com/sqlcat/20../../azure-sql-data-warehouse-loading-patterns-and-strategies/) (Modèles de charge).

## <a name="1-extract-the-source-data-into-text-files"></a>1. Extraire les données sources dans des fichiers texte

L’extraction des données à partir de votre système source dépend de l’emplacement de stockage.  L’objectif est de déplacer les données vers des fichiers texte délimités pris en charge par PolyBase.

### <a name="polybase-external-file-formats"></a>Formats des fichiers externes PolyBase

PolyBase charge les données à partir de fichiers texte encodés avec UTF-8 et UTF-16. En plus des fichiers texte délimités, il charge des données à partir des fichiers aux formats Hadoop, RC, ORC et Parquet. PolyBase peut également charger des données à partir de fichiers compressés Gzip et Snappy. PolyBase ne prend actuellement pas en charge le codage ASCII étendu, le format de largeur fixe et les formats imbriqués tels que WinZip, JSON et XML.

Si vous exportez des données à partir de SQL Server, vous pouvez utiliser l’[outil en ligne de commande bcp](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) pour exporter les données dans des fichiers texte délimités. Le mappage du type de données Parquet à Azure Synapse Analytics est le suivant :

| **Type de données Parquet** |                      **Type de données SQL**                       |
| :-------------------: | :----------------------------------------------------------: |
|        TINYINT        |                           TINYINT                            |
|       SMALLINT        |                           SMALLINT                           |
|          int          |                             int                              |
|        bigint         |                            bigint                            |
|        boolean        |                             bit                              |
|        double         |                            float                             |
|         float         |                             real                             |
|        double         |                            money                             |
|        double         |                          SMALLMONEY                          |
|        string         |                            NCHAR                             |
|        string         |                           NVARCHAR                           |
|        string         |                             char                             |
|        string         |                           varchar                            |
|        binary         |                            binary                            |
|        binary         |                          varbinary                           |
|       timestamp       |                             Date                             |
|       timestamp       |                        smalldatetime                         |
|       timestamp       |                          datetime2                           |
|       timestamp       |                           DATETIME                           |
|       timestamp       |                             time                             |
|       Date            |                             Date                             |
|        Décimal        |                            Décimal                           |

## <a name="2-land-the-data-into-azure-blob-storage-or-azure-data-lake-store"></a>2. Placer les données dans le stockage Blob Azure ou Azure Data Lake Store

Pour charger les données dans le stockage Azure, vous pouvez les déplacer dans le [stockage Blob Azure](../../storage/blobs/storage-blobs-introduction.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) ou dans [Azure Data Lake Store](../../data-lake-store/data-lake-store-overview.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json). Quel que soit l’emplacement choisi, les données doivent être stockées dans des fichiers texte. PolyBase peut effectuer le chargement à partir des deux emplacements.

Voici des outils et services que vous pouvez utiliser pour déplacer des données dans le stockage Azure.

- Le service [Azure ExpressRoute](../../expressroute/expressroute-introduction.md) améliore le débit, les performances et la prévisibilité du réseau. ExpressRoute est un service qui achemine vos données via une connexion privée dédiée vers Azure. Les connexions ExpressRoute n’acheminent pas vos données via le réseau Internet public. Elles offrent davantage de fiabilité, des vitesses supérieures, des latences inférieures et une sécurité renforcée par rapport aux connexions publiques sur Internet.
- L’[utilitaire AZCopy](../../storage/common/storage-use-azcopy-v10.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) déplace les données vers le stockage Azure via l’Internet public. Il fonctionne si la taille de vos données ne dépasse pas les 10 To. Pour effectuer des chargements réguliers avec AZCopy, testez la vitesse du réseau pour voir si elle est acceptable.
- [Azure Data Factory (ADF)](../../data-factory/introduction.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) dispose d’une passerelle que vous pouvez installer sur votre serveur local. Ensuite, vous pouvez créer un pipeline pour déplacer des données à partir de votre serveur local vers le stockage Azure. Pour utiliser Data Factory avec le pool SQL dédié, consultez [Charger des données dans un pool SQL dédié](../../data-factory/load-azure-sql-data-warehouse.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="3-prepare-the-data-for-loading"></a>3. Préparer les données pour le chargement

Avant de pouvoir charger les données de votre compte de stockage dans le pool SQL dédié, vous devrez peut-être les préparer et les nettoyer. Vous pouvez préparer vos données pendant qu’elles sont dans la source, pendant que vous exportez les données dans des fichiers texte ou une fois que les données se trouvent dans le stockage Azure.  Plus vous les préparez tôt, plus ce sera facile.  

### <a name="define-external-tables"></a>Définir des tables externes

Avant de pouvoir charger des données, vous devez définir des tables externes dans votre entrepôt de données. PolyBase utilise des tables externes pour définir les données dans le stockage Azure et y accéder. Une table externe est similaire à une vue de base de données. La table externe contient le schéma de table et pointe vers les données stockées en dehors de l’entrepôt de données.

La définition des tables externes implique de spécifier la source des données, le format des fichiers texte et les définitions de la table. Voici les rubriques sur la syntaxe T-SQL dont vous aurez besoin :

- [CREATE EXTERNAL DATA SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest)
- [CREATE EXTERNAL FILE FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest)
- [CREATE EXTERNAL TABLE](/sql/t-sql/statements/create-external-table-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest)

### <a name="format-text-files"></a>Formater les fichiers texte

Une fois les objets externes définis, vous devez aligner les lignes des fichiers texte avec la table externe et la définition du format de fichier. Les données de chaque ligne du fichier texte doivent être alignées avec la définition de la table.
Pour formater les fichiers texte :

- Si vos données proviennent d’une source non relationnelle, vous devez les transformer en lignes et en colonnes. Que les données proviennent d’une source relationnelle ou non relationnelle, elles doivent être transformées pour être alignées avec les définitions des colonnes pour la table dans laquelle vous souhaitez les charger.
- Formatez les données dans le fichier texte pour les aligner avec les types de colonnes et de données dans la table de destination du pool SQL. Un décalage entre les types de données dans les fichiers texte externes et la table de l’entrepôt de données cause le rejet des lignes lors du chargement.
- Séparez les champs dans le fichier texte à l’aide d’une marque de fin.  Assurez-vous d’utiliser un caractère ou une séquence de caractères qui ne se trouve pas dans votre source de données. Utilisez la marque de fin que vous avez spécifiée avec l’instruction [CREATE EXTERNAL FILE FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="4-load-the-data-into-dedicated-sql-pool-staging-tables-using-polybase"></a>4. Charger les données dans les tables de mise en lots du pool SQL dédié à l’aide de PolyBase

Il est recommandé de charger des données dans une table de mise en lots. Les tables de mise en lots vous permettent de gérer les erreurs sans interférer avec les tables de production. Une table de mise en lots vous donne également la possibilité d'utiliser les capacités de traitement des requêtes distribuées intégrées au pool SQL pour transformer les données avant de les insérer dans des tables de production.

### <a name="options-for-loading-with-polybase"></a>Options de chargement avec PolyBase

Pour charger des données avec PolyBase, vous pouvez utiliser l’une des options de chargement suivantes :

- [PolyBase avec T-SQL](../sql-data-warehouse/load-data-from-azure-blob-storage-using-polybase.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) fonctionne bien lorsque vos données se trouvent dans le stockage Blob Azure ou dans Azure Data Lake Store. Il vous offre un contrôle optimal sur le processus de chargement, mais nécessite également que vous définissiez des objets de données externes. Les autres méthodes définissent ces objets en arrière-plan pendant que vous mappez les tables sources vers les tables de destination.  Pour orchestrer les chargements T-SQL, vous pouvez utiliser Azure Data Factory, SSIS ou les fonctions Azure.
- [Polybase avec SSIS](/sql/integration-services/load-data-to-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) fonctionne bien quand vos données sources se trouvent dans SQL Server. SSIS définit le mappage de la table « source vers destination » et orchestre aussi le chargement. Si vous disposez déjà de packages SSIS, vous pouvez modifier les packages pour travailler avec le nouvel entrepôt de données de destination.
- [PolyBase avec Azure Data Factory (ADF)](../../data-factory/load-azure-sql-data-warehouse.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) est un autre outil d’orchestration.  Il définit un pipeline et planifie les travaux.
- [PolyBase avec Azure Databricks](../../azure-databricks/databricks-extract-load-sql-data-warehouse.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) transfère les données d’une table Azure Synapse Analytics vers une trame de données Databricks et/ou écrit des données d’une trame de données Databricks dans une table Azure Synapse Analytics à l’aide de la technologie PolyBase.

### <a name="non-polybase-loading-options"></a>Options de chargement non-PolyBase

Si vos données ne sont pas compatibles avec PolyBase, vous pouvez utiliser l’outil [bcp](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) ou l’[API SQLBulkCopy](/dotnet/api/system.data.sqlclient.sqlbulkcopy?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json). L’outil bcp charge directement dans le pool SQL dédié sans passer par Stockage Blob Azure et est destiné uniquement aux petits chargements. Notez que les performances de chargement de ces options sont beaucoup plus lentes qu’avec PolyBase.

## <a name="5-transform-the-data"></a>5. Transformer les données

Pendant que les données se trouvent dans la table de mise en lots, effectuez les transformations requises par votre charge de travail. Déplacez ensuite les données dans une table de production.

## <a name="6-insert-the-data-into-production-tables"></a>6. Insérer les données dans des tables de production

L’instruction INSERT INTO... SELECT déplace les données depuis la table de mise en lots vers la table permanente.

Lorsque vous concevez un processus ETL, commencez par exécuter le processus sur un petit échantillon. Essayez d’extraire 1 000 lignes de la table dans un fichier, déplacez-le vers Azure, puis essayez de le charger dans une table de mise en lots.

## <a name="partner-loading-solutions"></a>Solutions de chargement des partenaires

La plupart de nos partenaires proposent des solutions de chargement. Pour en savoir plus, consultez la liste de nos [partenaires de solutions](../sql-data-warehouse/sql-data-warehouse-partner-business-intelligence.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des instructions sur le chargement, consultez [Meilleures pratiques de chargement de données](data-loading-best-practices.md).
