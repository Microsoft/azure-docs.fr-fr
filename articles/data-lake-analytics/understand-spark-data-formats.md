---
title: Découvrez les formats de données Apache Spark pour les développeurs U-SQL d’Azure Data Lake Analytics.
description: Cet article décrit les concepts d’Apache Spark pour aider les développeurs U-SQL à comprendre les différences entre les formats de données U-SQL et Spark.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.custom: understand-apache-spark-data-formats
ms.date: 01/31/2019
ms.openlocfilehash: 399914186ce9de62ef46b682c8d4a6e51426cc26
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221109"
---
# <a name="understand-differences-between-u-sql-and-spark-data-formats"></a>Comprendre les différences entre les formats de données U-SQL et Spark

Si vous souhaitez utiliser [Azure Databricks](/azure/databricks/scenarios/what-is-azure-databricks) ou [Azure HDInsight Spark](../hdinsight/spark/apache-spark-overview.md), nous vous recommandons de migrer vos données d’[Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md) vers [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md).

En plus de déplacer vos fichiers, vous souhaiterez également rendre vos données, stockées dans des tables U-SQL, accessibles à Spark.

## <a name="move-data-stored-in-azure-data-lake-storage-gen1-files"></a>Déplacer des données stockées dans des fichiers Azure Data Lake Storage Gen1

Les données stockées dans des fichiers peuvent être déplacées de différentes manières :

- Écrivez un pipeline [Azure Data Factory](../data-factory/introduction.md) pour copier les données d’un compte [Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md) vers un compte [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md).
- Écrivez un travail Spark qui lit les données du compte [Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md) et les écrit dans le compte [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md). Selon votre cas d’utilisation, vous souhaiterez peut-être l’écrire dans un autre format, Parquet par exemple, si vous n’avez pas besoin de conserver le format de fichier d’origine.

Nous vous recommandons de consulter l’article [Mettre à niveau vos solutions d’analytique de Big Data d’Azure Data Lake Storage Gen1 vers Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-migrate-gen1-to-gen2.md)

## <a name="move-data-stored-in-u-sql-tables"></a>Déplacer des données stockées dans des tables U-SQL

Les tables U-SQL ne sont pas reconnues par Spark. Si vous disposez de données stockées dans des tables U-SQL, vous exécuterez un travail U-SQL qui extrait les données de la table et les enregistre dans un format reconnu par Spark. Le format le plus approprié consiste à créer un jeu de fichiers Parquet suivant la disposition de dossier du metastore Hive.

La sortie peut être obtenue en U-SQL avec le générateur de sortie Parquet intégré et à l’aide du partitionnement de sortie dynamique avec des jeux de fichiers pour créer les dossiers de partition. [Traiter plus de fichiers que jamais et utiliser parquet](/archive/blogs/azuredatalake/process-more-files-than-ever-and-use-parquet-with-azure-data-lake-analytics) fournit un exemple de création de ces données Spark consommables.

Après cette transformation, vous copiez les données comme indiqué dans le chapitre [Déplacer des données stockées dans des fichiers Azure Data Lake Storage Gen1](#move-data-stored-in-azure-data-lake-storage-gen1-files).

## <a name="caveats"></a>Mises en garde

- Sémantique des données - Lorsque vous copiez des fichiers, la copie s’exécute au niveau de l’octet. Ainsi, les mêmes données doivent apparaître dans le compte [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md). Notez toutefois que Spark peut interpréter certains caractères différemment. Par exemple, il peut utiliser une valeur par défaut différente pour un séparateur de lignes dans un fichier CSV.
    En outre, si vous copiez des données typées (à partir de tables), Parquet et Spark peuvent avoir une précision et une échelle différentes pour certaines des valeurs typées (par exemple, float) et peuvent traiter les valeurs Null différemment. Par exemple, U-SQL comprend la sémantique C# pour les valeurs Null, tandis que Spark comprend une logique à trois valeurs pour les valeurs Null.

- Les tables U-SQL d’organisation des données (partitionnement) fournissent un partitionnement à deux niveaux. Le niveau externe (`PARTITIONED BY`) est par valeur et mappe principalement dans le schéma de partitionnement Hive/Spark à l’aide de hiérarchies de dossiers. Vous devez vous assurer que les valeurs Null sont mappées avec le dossier approprié. Le niveau interne (`DISTRIBUTED BY`) dans U-SQL offre 4 schémas de distribution : tourniquet (round robin), plage, hachage et hachage direct.
    Les tables Hive/Spark prennent uniquement en charge le partitionnement de valeur ou le partitionnement de hachage, à l’aide d’une fonction de hachage différente de U-SQL. Lorsque vous générez vos données de table U-SQL, vous ne pourrez probablement mapper que le partitionnement de valeur pour Spark et devrez peut-être effectuer un réglage supplémentaire de la disposition de vos données en fonction de vos requêtes Spark finales.

## <a name="next-steps"></a>Étapes suivantes

- [Comprendre les concepts de code Spark pour les développeurs U-SQL](understand-spark-code-concepts.md)
- [Mettre à niveau vos solutions d’analytique de Big Data d’Azure Data Lake Storage Gen1 vers Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-migrate-gen1-to-gen2.md)
- [.NET pour Apache Spark](/dotnet/spark/what-is-apache-spark-dotnet)
- [Transformer des données à l’aide d’une activité Spark dans Azure Data Factory](../data-factory/transform-data-using-spark.md)
- [Transformer des données à l’aide d’une activité Hadoop Hive dans Azure Data Factory](../data-factory/transform-data-using-hadoop-hive.md)
- [Présentation d’Apache Spark dans Azure HDInsight](../hdinsight/spark/apache-spark-overview.md)