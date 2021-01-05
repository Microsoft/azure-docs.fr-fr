---
title: Migrer des charges de travail Apache Spark 2.1 ou 2.2 vers les versions 2.3 ou 2.4 – Azure HDInsight
description: Découvrez comment migrer Apache Spark 2.1 et 2.2 vers les versions 2.3 ou 2.4.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: hrasheed
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/20/2020
ms.openlocfilehash: 1ce9b0faa6636d1318871cc9ef66cfbe47908265
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89504976"
---
# <a name="migrate-apache-spark-21-and-22-workloads-to-23-and-24"></a>Migrer des charges de travail Apache Spark 2.1 et 2.2 vers les versions 2.3 et 2.4

Ce document explique comment migrer des charges de travail Apache Spark sur Spark 2.1 et 2.2 vers les versions 2.3 ou 2.4.

Comme indiqué dans les [notes de publication](../hdinsight-release-notes.md#upcoming-changes), à partir du 1er juillet 2020, les configurations de cluster suivantes ne seront pas prises en charge et les clients ne seront pas en mesure de créer de nouveaux clusters avec celles-ci :
 - Spark 2.1 et 2.2 dans un cluster Spark HDInsight 3.6
 - Spark 2.3 dans un cluster Spark HDInsight 4.0

Les clusters existants dans ces configurations s’exécuteront tels quels sans le support de Microsoft. Si vous utilisez Spark 2.1 ou 2.2 sur HDInsight 3.6, passez à Spark 2.3 sur HDInsight 3.6 d’ici le 30 juin 2020 pour éviter une éventuelle interruption du système ou du support. Si vous utilisez Spark 2.3 sur un cluster HDInsight 4.0, passez à Spark 2.4 sur HDInsight 4.0 d’ici le 30 juin 2020 pour éviter une éventuelle interruption du système ou du support.

Pour obtenir des informations générales sur la migration d’un cluster HDInsight 3.6 vers la version 4.0, consultez [Effectuer la migration d’un cluster HDInsight vers une version plus récente](../hdinsight-upgrade-cluster.md). Pour obtenir des informations générales sur la migration vers une version plus récente d’Apache Spark, consultez [Apache Spark : Stratégie de contrôle de version](https://spark.apache.org/versioning-policy.html).

## <a name="guidance-on-spark-version-upgrades-on-hdinsight"></a>Conseils sur les mises à niveau de la version Spark sur HDInsight

| Scénario de mise à niveau | Mechanism | Points importants à prendre en compte | Intégration de Spark/Hive |
|------------------|-----------|--------------------|------------------------|
|Spark 2.1 HDInsight 3.6 vers Spark 2.3 HDInsight 3.6| Recréer des clusters avec Spark 2.3 HDInsight | Consultez les articles suivants : <br> [Apache Spark : Mise à niveau de Spark SQL 2.2 vers 2.3](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-22-to-23) <br><br> [Apache Spark : Mise à niveau de Spark SQL 2.1 vers 2.2](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-21-to-22) | Aucune modification |
|Spark 2.2 HDInsight 3.6 vers Spark 2.3 HDInsight 3.6 | Recréer des clusters avec Spark 2.3 HDInsight | Consultez les articles suivants : <br> [Apache Spark : Mise à niveau de Spark SQL 2.2 vers 2.3](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-22-to-23) | Aucune modification |
| Spark 2.1 HDInsight 3.6 vers Spark 2.4 HDInsight 4.0 | Recréer des clusters avec HDInsight 4.0 et Spark 2.4 | Consultez les articles suivants : <br> [Apache Spark : Mise à niveau de Spark SQL 2.3 vers 2.4](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-23-to-24) <br><br> [Apache Spark : Mise à niveau de Spark SQL 2.2 vers 2.3](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-22-to-23) <br><br> [Apache Spark : Mise à niveau de Spark SQL 2.1 vers 2.2](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-21-to-22) | L’intégration de Spark et de Hive a changé dans HDInsight 4.0. <br><br> Dans HDInsight 4.0, Spark et Hive utilisent des catalogues indépendants pour accéder aux tables SparkSQL ou Hive. Les tables créées par Spark résident dans le catalogue Spark. Les tables créées par Hive résident dans le catalogue Hive. Ce comportement est différent de HDInsight 3.6, où Hive et Spark partageaient le même catalogue. L’intégration de Hive et Spark à HDInsight 4.0 s’appuie sur le connecteur Hive Warehouse Connector (HWC). Le connecteur HWC sert de pont entre Spark et Hive. En savoir plus sur Hive Warehouse Connector. <br> Dans HDInsight 4.0, si vous souhaitez partager le metastore entre Hive et Spark, vous pouvez le faire en remplaçant la propriété metastore.catalog.default par hive dans votre cluster Spark. Vous pouvez trouver cette propriété dans Ambari Advanced spark2-hive-site-override. Il est important de comprendre que le partage du metastore ne fonctionne que pour les tables Hive externes. Cela ne fonctionnera pas si vous avez des tables Hive internes/gérées ou des tables ACID. <br><br>Pour plus d’informations, consultez [Migrer des charges de travail Azure HDInsight 3.6 Hive vers HDInsight 4.0](../interactive-query/apache-hive-migrate-workloads.md).<br><br> |
| Spark 2.2 HDInsight 3.6 vers Spark 2.4 HDInsight 4.0 | Recréer des clusters avec HDInsight 4.0 et Spark 2.4 | Consultez les articles suivants : <br> [Apache Spark : Mise à niveau de Spark SQL 2.3 vers 2.4](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-23-to-24) <br><br> [Apache Spark : Mise à niveau de Spark SQL 2.2 vers 2.3](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-22-to-23) | L’intégration de Spark et de Hive a changé dans HDInsight 4.0. <br><br> Dans HDInsight 4.0, Spark et Hive utilisent des catalogues indépendants pour accéder aux tables SparkSQL ou Hive. Les tables créées par Spark résident dans le catalogue Spark. Les tables créées par Hive résident dans le catalogue Hive. Ce comportement est différent de HDInsight 3.6, où Hive et Spark partageaient le même catalogue. L’intégration de Hive et Spark à HDInsight 4.0 s’appuie sur le connecteur Hive Warehouse Connector (HWC). Le connecteur HWC sert de pont entre Spark et Hive. En savoir plus sur Hive Warehouse Connector. <br> Dans HDInsight 4.0, si vous souhaitez partager le metastore entre Hive et Spark, vous pouvez le faire en remplaçant la propriété metastore.catalog.default par hive dans votre cluster Spark. Vous pouvez trouver cette propriété dans Ambari Advanced spark2-hive-site-override. Il est important de comprendre que le partage du metastore ne fonctionne que pour les tables Hive externes. Cela ne fonctionnera pas si vous avez des tables Hive internes/gérées ou des tables ACID. <br><br>Pour plus d’informations, consultez [Migrer des charges de travail Azure HDInsight 3.6 Hive vers HDInsight 4.0](../interactive-query/apache-hive-migrate-workloads.md).|

## <a name="next-steps"></a>Étapes suivantes

* [Effectuer la migration d’un cluster HDInsight vers une version plus récente](../hdinsight-upgrade-cluster.md)
* [Migrer des charges de travail Azure HDInsight 3.6 Hive vers HDInsight 4.0](../interactive-query/apache-hive-migrate-workloads.md)
