---
title: Optimiser l’utilisation de la mémoire dans Apache Spark - Azure HDInsight
description: Découvre comment optimiser l’utilisation de la mémoire dans Apache Spark sur Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/20/2020
ms.custom: contperf-fy21q1
ms.openlocfilehash: ed10648fb115ae0dc12300f3e48db165fcb4c215
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97028153"
---
# <a name="memory-usage-optimization-for-apache-spark"></a>Optimisation de l’utilisation de la mémoire pour Apache Spark

Cet article explique comment optimiser la gestion de la mémoire de votre cluster Apache Spark pour obtenir de meilleures performances sur Azure HDInsight.

## <a name="overview"></a>Vue d’ensemble

Spark fonctionne en plaçant les données en mémoire. La gestion des ressources en mémoire est donc un aspect clé de l’optimisation de l’exécution des travaux Spark.  Vous pouvez appliquer plusieurs techniques pour utiliser efficacement la mémoire de votre cluster.

* Privilégiez les partitions de données de petite taille, et prenez en compte la taille, les types et la distribution des données dans votre stratégie de partitionnement.
* Utilisez la sérialisation [`Kryo data serialization`](https://github.com/EsotericSoftware/kryo), plus récente et plus efficace, au lieu de la sérialisation Java par défaut.
* Privilégiez l’utilisation de YARN, car il sépare `spark-submit` en lot.
* Surveillez et affinez les paramètres de configuration de Spark.

Pour référence, la structure de la mémoire Spark et certains des principaux paramètres mémoire de l’exécuteur sont illustrés dans l’image suivante.

## <a name="spark-memory-considerations"></a>Considérations relatives à la mémoire Spark

Si vous utilisez Apache Hadoop YARN, la plateforme contrôle la mémoire utilisée par tous les conteneurs sur chaque nœud Spark.  Le diagramme suivant montre les objets clés et leurs relations.

![Gestion de la mémoire Spark avec YARN](./media/apache-spark-perf/apache-yarn-spark-memory.png)

Pour éviter les messages « Mémoire insuffisante », essayez les solutions suivantes :

* Passez en revue les lectures aléatoires de gestion DAG. Réduisez par la réduction côté mappage, prépartitionnez (ou compartimentez) les données sources, optimisez les lectures aléatoires uniques, et réduisez la quantité de données envoyées.
* Privilégiez `ReduceByKey` (avec sa limite de mémoire fixe) à `GroupByKey`, qui fournit des agrégations, le fenêtrage et d’autres fonctions, mais qui n’a pas de limite de mémoire.
* Privilégiez `TreeReduce`, qui effectue plus de travail sur les partitions ou les exécuteurs, à `Reduce`, qui effectue tout le travail sur le pilote.
* Utilisez des DataFrames plutôt que des objets RDD de niveau inférieur.
* Créez des ComplexTypes qui encapsulent des actions, telles que « N premiers », différentes agrégations ou opérations de fenêtrage.

Pour obtenir des étapes supplémentaires de résolution des problèmes, consultez [Exceptions OutOfMemoryError pour Apache Spark dans Azure HDInsight](apache-spark-troubleshoot-outofmemory.md).

## <a name="next-steps"></a>Étapes suivantes

* [Optimiser le traitement de données pour Apache Spark](optimize-cluster-configuration.md)
* [Optimiser le stockage des données pour Apache Spark](optimize-data-storage.md)
* [Optimiser la configuration de cluster pour Apache Spark](optimize-cluster-configuration.md)
