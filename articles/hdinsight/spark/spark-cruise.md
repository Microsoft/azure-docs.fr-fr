---
title: Utiliser SparkCruise sur Azure HDInsight pour accélérer les requêtes Apache Spark
description: Découvrez comment utiliser la plateforme d’optimisation SparkCruise pour améliorer l’efficacité des requêtes Apache Spark.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 07/27/2020
ms.openlocfilehash: 1a73b4707f83d6a23dffc20d95aa7b8a0fa465b3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88649055"
---
# <a name="sparkcruise-on-azure-hdinsight"></a>SparkCruise sur Azure HDInsight

Ce document traite de la fonctionnalité Azure HDInsight *SparkCruise*, qui réutilise automatiquement les calculs Apache Spark pour améliorer l’efficacité des requêtes.

## <a name="overview"></a>Vue d’ensemble

Les requêtes que vous exécutez sur une plateforme d’analyse telle qu’Apache Spark sont décomposées dans un plan de requête qui contient des sous-requêtes plus petites. Ces sous-requêtes peuvent apparaître à plusieurs reprises dans les plans de requête de plusieurs requêtes. À chaque fois que cela se produit, elles sont réexécutées pour retourner les résultats. Toutefois, la réexécution de la même requête peut être inefficace et entraîner des coûts de calcul inutiles.

*SparkCruise* est une plateforme d’optimisation de la charge de travail qui peut réutiliser des calculs communs, diminuant ainsi le temps global d’exécution des requêtes et les coûts de transfert de données. La plateforme utilise le concept de *vue matérialisée*, qui est une requête dont les résultats sont stockés dans un format précalculé. Ces résultats peuvent ensuite être réutilisés lorsque la requête elle-même réapparaît ultérieurement, au lieu de recalculer les résultats.

## <a name="setup-and-installation"></a>Configuration et installation

SparkCruise est disponible sur tous les clusters HDInsight 4.0 avec Spark 2.3 ou 2.4. Les fichiers bibliothèques SparkCruise sont installés dans le répertoire `/opt/peregrine/` de votre cluster HDInsight. Pour fonctionner correctement, *SparkCruise* requiert les propriétés de configuration suivantes, qui sont définies par défaut.

* `spark.sql.queryExecutionListeners` est définie sur `com.microsoft.peregrine.spark.listeners.PlanLogListener`, ce qui active la journalisation des plans de requête.
* `spark.sql.extensions` est définie sur `com.microsoft.peregrine.spark.extensions.SparkExtensionsHdi`, ce qui active les règles de l’optimiseur pour la matérialisation et la réutilisation en ligne.

## <a name="computation-reuse-in-spark-sql"></a>Réutilisation des calculs dans Spark SQL

L’exemple de scénario suivant illustre l’utilisation de *SparkCruise* pour optimiser les requêtes Apache Spark. 

1. Utilisez SSH dans le nœud principal de votre cluster Spark.
1. Tapez `spark-shell`.
1. Exécutez l’extrait de code suivant, qui exécute quelques requêtes de base à l’aide d’exemples de données sur le cluster.

    ```scala
    spark.sql("select count(*) from hivesampletable").collect
    spark.sql("select count(*) from hivesampletable").collect
    spark.sql("select distinct market from hivesampletable where querytime like '11%'").show
    spark.sql("select distinct state, country from hivesampletable where querytime like '11%'").show
    :quit
    ```
1. Utilisez l’outil d’analyse des requêtes *SparkCruise* pour analyser les plans de requête des requêtes précédentes, qui sont stockées dans les journaux des applications Spark. 

    ```
    sudo /opt/peregrine/analyze/peregrine.sh analyze views
    ```

1. Affichez la sortie du processus d’analyse, qui est un fichier de commentaires. Ce fichier de commentaires contient des annotations pour les futures requêtes Spark SQL. 

    ```
    sudo /opt/peregrine/analyze/peregrine.sh show
    ```

La commande `analyze` analyse les plans de requête et crée une représentation tabulaire de la charge de travail. Vous pouvez interroger cette table de charge de travail à l’aide du notebook *WorkloadInsights* inclus dans le référentiel [Exemples SparkCruise HDInsight](https://github.com/Azure-Samples/azure-sparkcruise-samples). Ensuite, la commande `views` identifie des expressions de sous-plan communes et sélectionne celles qui sont intéressantes pour une matérialisation et une réutilisation ultérieures. La sortie est un fichier de commentaires contenant des annotations pour les futures requêtes Spark SQL. 

La commande `show` affiche une sortie semblable au texte suivant :

```bash
Feedback file -->

1593761760087311271 Materialize /peregrine/views/1593761760087311271
1593761760087311271 Reuse /peregrine/views/1593761760087311271
18446744073621796959 Materialize /peregrine/views/18446744073621796959
18446744073621796959 Reuse /peregrine/views/18446744073621796959
11259615723090744908 Materialize /peregrine/views/11259615723090744908
11259615723090744908 Reuse /peregrine/views/11259615723090744908
9409467400931056980 Materialize /peregrine/views/9409467400931056980
9409467400931056980 Reuse /peregrine/views/9409467400931056980

Materialized subexpressions -->

Found 4 items
-rw-r--r--   1 sshuser sshuser     113445 2020-07-24 16:46 /peregrine/views/logical_ir.csv
-rw-r--r--   1 sshuser sshuser     169458 2020-07-24 16:46 /peregrine/views/physical_ir.csv
-rw-r--r--   1 sshuser sshuser      25730 2020-07-24 16:46 /peregrine/views/views.csv
-rw-r--r--   1 sshuser sshuser        536 2020-07-24 16:46 /peregrine/views/views.stp
```

Le fichier de commentaires contient des entrées au format suivant : `subplan-identifier [Materialize|Reuse] input/path/to/action`. Dans cet exemple, il existe deux signatures uniques : une qui représente les deux premières requêtes répétées et la seconde qui représente le prédicat de filtre dans les deux dernières requêtes. Grâce à ce fichier de commentaires, les requêtes suivantes, lorsqu’elles seront soumises à nouveau, se matérialiseront et réutiliseront automatiquement les sous-plans communs. 

Pour tester les optimisations, exécutez une autre série d’exemples de requêtes.

1. Tapez `spark-shell`.
1. Exécutez l’extrait de code suivant.

    ```scala
    spark.sql("select count(*) from hivesampletable").collect
    spark.sql("select count(*) from hivesampletable").collect
    spark.sql("select distinct state, country from hivesampletable where querytime like '12%'").show
    spark.sql("select distinct market from hivesampletable where querytime like '12%'").show
    :quit
    ```

1. Affichez de nouveau le fichier de commentaires pour voir les signatures des requêtes qui ont été réutilisées.

    ```
    sudo /opt/peregrine/analyze/peregrine.sh show
    ```

La commande `show` donne une sortie semblable au texte suivant :

```bash
Feedback file -->

1593761760087311271 Materialize /peregrine/views/1593761760087311271
1593761760087311271 Reuse /peregrine/views/1593761760087311271
18446744073621796959 Materialize /peregrine/views/18446744073621796959
18446744073621796959 Reuse /peregrine/views/18446744073621796959
11259615723090744908 Materialize /peregrine/views/11259615723090744908
11259615723090744908 Reuse /peregrine/views/11259615723090744908
9409467400931056980 Materialize /peregrine/views/9409467400931056980
9409467400931056980 Reuse /peregrine/views/9409467400931056980

Materialized subexpressions -->

Found 8 items
drwxr-xr-x   - root root          0 2020-07-24 17:21 /peregrine/views/11259615723090744908
drwxr-xr-x   - root root          0 2020-07-24 17:21 /peregrine/views/1593761760087311271
drwxr-xr-x   - root root          0 2020-07-24 17:22 /peregrine/views/18446744073621796959
drwxr-xr-x   - root root          0 2020-07-24 17:21 /peregrine/views/9409467400931056980
-rw-r--r--   1 root root     113445 2020-07-24 16:46 /peregrine/views/logical_ir.csv
-rw-r--r--   1 root root     169458 2020-07-24 16:46 /peregrine/views/physical_ir.csv
-rw-r--r--   1 root root      25730 2020-07-24 16:46 /peregrine/views/views.csv
-rw-r--r--   1 root root        536 2020-07-24 16:46 /peregrine/views/views.stp

```

Bien que la valeur littérale de la requête soit passée de `'11%'` à `'12%'`, *SparkCruise* peut toujours faire correspondre les requêtes précédentes aux nouvelles requêtes avec de légères variations telles que l’évolution des valeurs littérales et des versions de jeux de données. S’il existe des modifications majeures apportées à une requête, vous pouvez réexécuter l’outil d’analyse pour identifier les nouvelles requêtes qui peuvent être réutilisées.

En arrière-plan, *SparkCruise* déclenche une sous-requête pour matérialiser le sous-plan sélectionné à partir de la première requête qui le contient. Les requêtes ultérieures peuvent lire directement les sous-plans matérialisés au lieu de les recalculer. Dans cette charge de travail, les sous-plans seront matérialisés en ligne par la première et la troisième requête. Nous pouvons observer le changement de plan des requêtes après la matérialisation des sous-plans communs.

Vous pouvez voir que quatre nouvelles sous-expressions matérialisées sont désormais disponibles pour être réutilisées dans les requêtes suivantes.

## <a name="clean-up"></a>Nettoyage

Les fichiers de commentaires, les sous-plans matérialisés et les journaux de requêtes sont conservés dans des sessions Spark. Pour supprimer ces fichiers, exécutez la commande suivante :

```bash
sudo /opt/peregrine/analyze/peregrine.sh clean
```

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser le notebook Insights sur la charge de travail pour déterminer les avantages de SparkCruise](https://github.com/Azure-Samples/azure-sparkcruise-samples/tree/main/SparkCruise)
* [Améliorer les performances des charges de travail Apache Spark à l’aide d’Azure HDInsight IO Cache](apache-spark-improve-performance-iocache.md)
* [Optimiser les travaux Apache Spark dans Azure HDInsight](./apache-spark-perf.md)
* [SparkCruise: Handsfree Computation Reuse in Spark](https://people.cs.umass.edu/~aroy/sparkcruise-vldb19.pdf)
* [Instructions Apache Spark sur Azure HDInsight](./spark-best-practices.md)
