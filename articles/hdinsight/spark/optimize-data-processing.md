---
title: Optimiser le traitement de données pour Apache Spark – Azure HDInsight
description: Découvrez comment choisir les opérations les plus efficaces pour traiter vos données sur Apache Spark avec Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/20/2020
ms.custom: contperf-fy21q1
ms.openlocfilehash: 3b85f6238bec4ef85d724a2fc48ea5988c3fceb2
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97031723"
---
# <a name="data-processing-optimization-for-apache-spark"></a>Optimisation du traitement de données pour Apache Spark

Cet article explique comment optimiser la configuration de votre cluster Apache Spark pour obtenir de meilleures performances sur Azure HDInsight.

## <a name="overview"></a>Vue d’ensemble

Si vous avez des travaux lents sur une jointure ou une lecture aléatoire, la cause est probablement l’*asymétrie des données*. L’asymétrie des données désigne une asymétrie dans vos données de travail. Par exemple, un travail de mappage peut prendre 20 secondes, mais l’exécution d’un travail où les données sont jointes ou assignées de manière aléatoire prend des heures. Pour corriger l’asymétrie des données, vous devez convertir l’intégralité de la clé en valeur salt, ou utiliser une *valeur salt isolée* pour uniquement un sous-ensemble de clés. Si vous utilisez une valeur salt isolée, vous devez effectuer un filtrage supplémentaire afin d’isoler votre sous-ensemble de clés salt dans les jointures de mappage. Une autre option consiste à introduire une colonne de compartiment et à pré-agréger d’abord dans les compartiments.

Un autre facteur pouvant ralentir les jointures est le type de jointure. Par défaut, Spark utilise le type de jointure `SortMerge`. Ce type de jointure convient parfaitement aux grands jeux de données, mais il est coûteux en terme de calcul, car il doit d’abord trier les côtés gauche et droit des données avant de les fusionner.

Une jointure `Broadcast` est particulièrement adaptée aux petits jeux de données, ou quand un côté de la jointure est beaucoup plus petit que l’autre. Ce type de jointure diffuse un côté à tous les exécuteurs. Il exige donc davantage de mémoire pour les diffusions en général.

Vous pouvez changer le type de jointure dans votre configuration en définissant `spark.sql.autoBroadcastJoinThreshold`, ou vous pouvez définir un indicateur de jointure à l’aide des API DataFrame (`dataframe.join(broadcast(df2))`).

```scala
// Option 1
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 1*1024*1024*1024)

// Option 2
val df1 = spark.table("FactTableA")
val df2 = spark.table("dimMP")
df1.join(broadcast(df2), Seq("PK")).
    createOrReplaceTempView("V_JOIN")

sql("SELECT col1, col2 FROM V_JOIN")
```

Si vous utilisez des tables compartimentées, vous disposez d’un troisième type de jointure : `Merge`. Un jeu de données correctement prépartitionné et prétrié ignore la phase de tri coûteuse d’une jointure `SortMerge`.

L’ordre des jointures est important, en particulier dans les requêtes plus complexes. Commencez par les jointures les plus sélectives. Dans la mesure du possible, déplacez également les jointures qui augmentent le nombre de lignes après les agrégations.

Pour gérer le parallélisme en cas de jointures cartésiennes, vous pouvez ajouter des structures imbriquées, le fenêtrage, et éventuellement ignorer une ou plusieurs étapes dans votre travail Spark.

## <a name="optimize-job-execution"></a>Optimiser l’exécution des travaux

* Effectuez une mise en cache si nécessaire. Par exemple, si vous utilisez les données à deux reprises, mettez-les en cache.
* Diffusez les variables vers tous les exécuteurs. Les variables ne sont sérialisées qu’une seule fois, ce qui accélère les recherches.
* Utilisez le pool de threads sur le pilote. Cela permet d’accélérer de nombreuses tâches.

Surveillez régulièrement vos travaux en cours d’exécution afin de détecter tout problème de performances. Si vous avez besoin de plus de détails pour résoudre certains problèmes, utilisez l’un des outils de profilage de performances suivants :

* [Intel PAL Tool](https://github.com/intel-hadoop/PAT) surveille l’utilisation de la bande passante par le réseau, le stockage et l’UC.
* [Oracle Java 8 Mission Control](https://www.oracle.com/technetwork/java/javaseproducts/mission-control/java-mission-control-1998576.html) profile le code d’exécuteur et Spark.

Le moteur Tungsten joue un rôle essentiel pour les performances des requêtes Spark 2.x. Il dépend de la génération de code à l’échelle globale. Dans certains cas, la génération de code à l’échelle globale peut être désactivée. Par exemple, si vous utilisez un type non mutable (`string`) dans l’expression d’agrégation, `SortAggregate` s’affiche à la place de `HashAggregate`. Par exemple, pour obtenir de meilleures performances, essayez ce qui suit puis réactivez la génération de code :

```sql
MAX(AMOUNT) -> MAX(cast(AMOUNT as DOUBLE))
```

## <a name="next-steps"></a>Étapes suivantes

* [Optimiser le stockage des données pour Apache Spark](optimize-data-storage.md)
* [Optimiser l’utilisation de la mémoire pour Apache Spark](optimize-memory-usage.md)
* [Optimiser la configuration de cluster pour Apache Spark](optimize-cluster-configuration.md)
