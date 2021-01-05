---
title: Découvrez Apache Spark pour les développeurs U-SQL d’Azure Data Lake Analytics.
description: Cet article décrit les concepts d’Apache Spark pour vous aider à comprendre les différences entre les développeurs U-SQL.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.custom: understand-apache-spark-for-usql-developers
ms.date: 10/15/2019
ms.openlocfilehash: a66a82a6d2a5bb1f534ed211091793b2ab4d2a28
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221092"
---
# <a name="understand-apache-spark-for-u-sql-developers"></a>Comprendre Apache Spark pour les développeurs U-SQL

Microsoft prend en charge plusieurs services d’analyse, tels que [Azure Databricks](/azure/databricks/scenarios/what-is-azure-databricks) et [Azure HDInsight](../hdinsight/hdinsight-overview.md), ainsi que Azure Data Lake Analytics. Les développeurs nous font savoir qu’ils ont une nette préférence pour les solutions open source lorsqu’ils créent des pipelines d’analyse. Pour aider les développeurs U-SQL à comprendre Apache Spark et la façon dont vous pouvez transformer vos scripts U-SQL en Apache Spark, nous avons créé ce guide.

Il comprend un certain nombre d’étapes que vous pouvez effectuer, ainsi que plusieurs alternatives.

## <a name="steps-to-transform-u-sql-to-apache-spark"></a>Étapes de transformation de U-SQL en Apache Spark

1. Transformez vos pipelines d’orchestration de travail.

   Si vous utilisez [Azure Data Factory](../data-factory/introduction.md) pour orchestrer vos scripts Azure Data Lake Analytics, vous devrez les ajuster pour orchestrer les nouveaux programmes Spark.
2. Comprendre les différences en matière de gestion de données pour U-SQL et Spark

   Si vous souhaitez déplacer vos données de [Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md) vers [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md), vous devez copier les données de fichier et les données maintenues de catalogue. Notez que Azure Data Lake Analytics prend en charge uniquement Azure Data Lake Storage Gen1. Voir [Comprendre les formats de données Spark](understand-spark-data-formats.md)
3. Transformez vos scripts U-SQL en Spark

   Avant de transformer vos scripts U-SQL, vous devez choisir un service d’analyse. Voici quelques-uns des services de calcul disponibles :
      - [Flux de données Azure Data Factory](../data-factory/concepts-data-flow-overview.md) Les mappages de flux de données sont des transformations de données visuellement conçues qui permettent aux ingénieurs de données de développer une logique de transformation de données graphique sans écrire de code. Bien qu’ils ne soient pas adaptés pour exécuter du code utilisateur complexe, ils peuvent facilement représenter des transformations de flux de données de type SQL traditionnelles
      - [Azure HDInsight Hive](../hdinsight/hadoop/apache-hadoop-using-apache-hive-as-an-etl-tool.md) Apache Hive sur HDInsight est adapté aux opérations d’extraction, de transformation et de chargement (ETL). Cela signifie que vous allez convertir vos scripts U-SQL en Apache Hive.
      - Moteurs Apache Spark tels que [Azure HDInsight Spark](../hdinsight/spark/apache-spark-overview.md) ou [Azure Databricks](/azure/databricks/scenarios/what-is-azure-databricks) Cela signifie que vous allez traduire vos scripts U-SQL en Spark. Pour plus d’informations, consultez [Comprendre les formats de données Spark](understand-spark-data-formats.md)

> [!CAUTION]
> [Azure Databricks](/azure/databricks/scenarios/what-is-azure-databricks) et [Azure HDInsight Spark](../hdinsight/spark/apache-spark-overview.md) sont des services de cluster et non des travaux serverless comme Azure Data Lake Analytics. Vous devez réfléchir à la façon d’approvisionner les clusters pour obtenir le ratio coût/performances approprié et comment gérer leur durée de vie pour réduire vos coûts. Ces services ont des caractéristiques de performances différentes du code utilisateur écrit en .NET, vous devez donc écrire des wrappers ou réécrire votre code dans un langage pris en charge. Pour plus d’informations, consultez [Understand Spark data formats](understand-spark-data-formats.md) (Comprendre les formats de données Spark), [Understand Apache Spark code concepts for U-SQL developers](understand-spark-code-concepts.md) (Comprendre les concepts de code Apache Spark pour les développeurs U-SQL), [.Net for Apache Spark](https://dotnet.microsoft.com/apps/data/spark) (.NET pour Apache Spark)

## <a name="next-steps"></a>Étapes suivantes

- [Comprendre les formats de données Spark pour les développeurs U-SQL](understand-spark-data-formats.md)
- [Comprendre les concepts de code Spark pour les développeurs U-SQL](understand-spark-code-concepts.md)
- [Mettre à niveau vos solutions d’analytique de Big Data d’Azure Data Lake Storage Gen1 vers Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-migrate-gen1-to-gen2.md)
- [.NET pour Apache Spark](/dotnet/spark/what-is-apache-spark-dotnet)
- [Transformer des données à l’aide d’une activité Hadoop Hive dans Azure Data Factory](../data-factory/transform-data-using-hadoop-hive.md)
- [Transformer des données à l’aide d’une activité Spark dans Azure Data Factory](../data-factory/transform-data-using-spark.md)
- [Présentation d’Apache Spark dans Azure HDInsight](../hdinsight/spark/apache-spark-overview.md)