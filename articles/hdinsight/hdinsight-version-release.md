---
title: Vue d’ensemble d’HDInsight 4.0 - Azure
description: Comparer les fonctionnalités, les limitations et les recommandations de mise à niveau entre HDInsight 3.6 et HDInsight 4.0.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: hrasheed
ms.service: hdinsight
ms.topic: conceptual
ms.date: 08/21/2020
ms.openlocfilehash: 2fd7a3e512b79651fdcf6a6ac0c14822361fc263
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350192"
---
# <a name="azure-hdinsight-40-overview"></a>Vue d’ensemble d’Azure HDInsight 4.0

Azure HDInsight est l'un des services les plus populaires parmi les clients d'entreprise pour Apache Hadoop et Apache Spark. HDInsight 4.0 est une distribution cloud des composants Apache Hadoop. Cet article fournit des informations sur les dernières versions d’Azure HDInsight et sur la mise à niveau.

## <a name="whats-new-in-hdinsight-40"></a>Nouveautés de HDInsight 4.0

### <a name="apache-hive-30-and-low-latency-analytical-processing"></a>Apache Hive 3.0 et traitement analytique à faible latence

Le traitement Apache Hive LLAP (Low-Latency Analytical Processing) utilise des serveurs de requêtes persistants et la mise en cache en mémoire. Ce processus permet d'obtenir rapidement des résultats SQL sur les données d’un stockage cloud distant. Hive LLAP utilise un ensemble persistant de démons qui exécutent des fragments de requêtes Hive. L’exécution de requêtes sur LLAP est le même que pour Hive sans LLAP, sauf que les tâches de travail s’exécutent dans des démons LLAP au lieu de conteneurs.

Voici certains avantages de Hive LLAP :

* Capacité à effectuer des analyses SQL approfondies sans sacrifier les performances et l'adaptabilité. Par exemple les jointures complexes, les sous-requêtes, les fonctions de fenêtrage, le tri, les fonctions définies par l'utilisateur et les agrégations complexes.

* Requêtes interactives sur les données du même stockage que celui où sont préparées les données, vous n’avez donc pas besoin de déplacer les données du stockage vers un autre moteur de traitement analytique.

* La mise en cache des résultats des requêtes permet de réutiliser les résultats des requêtes déjà calculées. Ce cache permet d'économiser le temps et les ressources consacrés à l'exécution des tâches de cluster requises pour la requête.

### <a name="hive-dynamic-materialized-views"></a>Vues matérialisées dynamiques Hive

Hive prend désormais en charge les vues matérialisées dynamiques, ou le précalcul des totaux pertinents. Les vues accélèrent le traitement des requêtes dans les entrepôts de données. Les vues matérialisées peuvent être stockées nativement dans Hive et utiliser l’accélération LLAP de manière fluide.

### <a name="hive-transactional-tables"></a>Tables transactionnelles Hive

HDI 4.0 inclut Apache Hive 3. Hive 3 nécessite une conformité ACID (atomicité, cohérence, isolation, durabilité) pour les tables transactionnelles qui résident dans l’entrepôt Hive. Les tables de conformité ACID et les données de table sont accessibles et gérées par Hive. Les données des tables CRUD (créer, récupérer, mettre à jour et supprimer) doivent être au format de fichier ORC (Optimized Row Column). Les tables à insertion uniquement prennent en charge tous les formats de fichier.

* ACID v2 améliore les performances au niveau du format de stockage et du moteur d’exécution.

* ACID est activé par défaut pour autoriser la prise en charge complète des mises à jour de données.

* Les capacités ACID améliorées vous permettent d’effectuer des mises à jour et des suppressions au niveau des lignes.

* Aucune surcharge de performances.

* Aucun compartimentage nécessaire.

* Spark peut lire et écrire dans des tables ACID Hive via le connecteur d’entrepôt Hive.

Découvrez plus en détail [Apache Hive 3.0](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.0.0/hive-overview/content/hive_whats_new_in_this_release_hive.html).

### <a name="apache-spark"></a>Apache Spark

Apache Spark obtient des tables pouvant être mises à jour et des transactions ACID avec le connecteur d’entrepôt Hive. Le connecteur d’entrepôt Hive vous permet d’inscrire des tables transactionnelles Hive comme des tables externes dans Spark pour accéder à toutes les fonctionnalités transactionnelles. Les versions précédentes prenaient uniquement en charge la manipulation de partitions de table. Le connecteur d’entrepôt Hive prend aussi en charge les trames de données de streaming.  Ce processus diffuse en continu les lectures et les écritures dans des tables Hive de streaming et transactionnelles à partir de Spark.

Les exécuteurs Spark peuvent se connecter directement aux démons Hive LLAP pour récupérer et mettre à jour les données de façon transactionnelle, ce qui permet à Hive de garder le contrôle des données.

Apache Spark sur HDInsight 4.0 prend en charge les scénarios suivants :

* Entraîner le modèle Machine Learning sur la même table transactionnelle utilisée pour la création de rapports.
* Utiliser des transactions ACID pour ajouter de manière sécurisée des colonnes de Spark ML dans une table Hive.
* Exécuter un travail de streaming Spark sur le flux de modification à partir d’une table de streaming Hive.
* Créer des fichiers ORC directement à partir d’un travail Spark Structured Streaming.

Ce n’est plus un souci si vous essayez accidentellement d’accéder à des tables transactionnelles Hive directement à partir de Spark. Cela entraîne des résultats incohérents, des données en double ou une altération des données. Dans HDInsight 4.0, les tables Spark et les tables Hive sont conservées dans des metastores séparés. Utilisez le connecteur d’entrepôt de données Hive pour inscrire explicitement des tables transactionnelles Hive comme des tables externes Spark.

Découvrez plus en détail [Apache Spark](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.0.0/spark-overview/content/analyzing_data_with_apache_spark.html).

### <a name="apache-oozie"></a>Apache Oozie

Apache Oozie 4.3.1 est inclus dans HDI 4.0 avec les changements suivants :

* Oozie n’exécute plus d’actions Hive. Hive CLI a été supprimé et remplacé par BeeLine.

* Vous pouvez exclure les dépendances indésirables de la bibliothèque de partage en ajoutant un modèle d’exclusion dans votre fichier **job.properties**.

Découvrez plus en détail [Apache Oozie](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.0.0/release-notes/content/patch_oozie.html).

## <a name="how-to-upgrade-to-hdinsight-40"></a>Comment effectuer une mise à niveau vers HDInsight 4.0

Testez de manière approfondie vos composants avant d’implémenter la dernière version dans un environnement de production. HDInsight 4.0 vous permet de lancer le processus de mise à niveau. HDInsight 3.6 est l'option par défaut pour prévenir les accidents.

Il n’existe aucun chemin de mise à niveau des versions précédentes de HDInsight vers HDInsight 4.0 pris en charge. Comme les formats de données des metastores et des objets blob ont changé, HDInsight 4.0 n’est pas compatible avec les versions précédentes. Il est important de bien séparer votre nouvel environnement HDInsight 4.0 de votre environnement de production actuel. Si vous déployez HDInsight 4.0 sur votre environnement actuel, votre metastore sera mis à niveau de manière irréversible.  

## <a name="limitations"></a>Limites

* HDInsight 4.0 ne prend pas en charge MapReduce pour Apache Hive. Utilisez plutôt Apache Tez. Découvrez plus en détail [Apache Tez](https://tez.apache.org/).
* HDInsight 4.0 ne prend pas en charge Apache Storm.
* HDInsight 4,0 ne prend pas en charge le type de cluster ML services.
* La vue Hive est uniquement disponible sur les clusters HDInsight 4.0 dotés d’un numéro de version supérieur ou égal à 4.1. Ce numéro de version est disponible dans Administrateur Ambari -> Versions.
* L’interpréteur de commandes d’Apache Zeppelin n’est pas pris en charge dans les clusters Spark et Interactive Query.
* Vous ne pouvez pas *désactiver* LLAP sur un cluster Spark-LLAP. Vous pouvez uniquement désactiver LLAP.
* Azure Data Lake Storage Gen2 ne peut pas enregistrer les blocs-notes Jupyter dans un cluster Spark.
* Apache Pig s’exécute par défaut sur Tez, mais vous pouvez le remplacer par MapReduce.
* L’intégration de Spark SQL Ranger pour la sécurité des lignes et des colonnes est déconseillée.
* Spark 2.4 et Kafka 2.1 sont disponibles dans HDInsight 4.0. Par conséquent, Spark 2.3 et Kafka  1.1 ne sont plus pris en charge. Nous vous recommandons d’utiliser Spark 2.4 et Kafka 2.1 et versions ultérieures dans HDInsight 4.0.

## <a name="next-steps"></a>Étapes suivantes

* [Guide de migration HBase](./hbase/apache-hbase-migrate-new-version.md)
* [Guide de migration Hive](./interactive-query/apache-hive-migrate-workloads.md)
* [Guide de migration Kafka](./kafka/migrate-versions.md)
* [Guide de migration Spark](./spark/migrate-versions.md)
* [Documentation Azure HDInsight](index.yml)
* [Notes de publication](hdinsight-release-notes.md)