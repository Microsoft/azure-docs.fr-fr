---
title: Optimiser la configuration du cluster Apache Spark - Azure HDInsight
description: Découvrez comment configurer votre cluster Apache Spark pour améliorer le débit sur Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 08/21/2020
ms.custom: contperf-fy21q1
ms.openlocfilehash: 80abc052bc8fa731a1cea0d3ecb523aedb46cf05
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97031779"
---
# <a name="cluster-configuration-optimization-for-apache-spark"></a>Optimisation de la configuration de cluster pour Apache Spark

Cet article explique comment optimiser la configuration de votre cluster Apache Spark pour obtenir de meilleures performances sur Azure HDInsight.

## <a name="overview"></a>Vue d’ensemble

En fonction de votre charge de travail de cluster Spark, vous pouvez déterminer qu’une configuration Spark autre que la configuration par défaut entraînerait une optimisation de l’exécution des travaux Spark.  Effectuez des tests d’évaluation avec des exemples de charges de travail afin de valider les configurations de cluster différentes des configurations par défaut.

Voici quelques paramètres courants que vous pouvez ajuster :

|Paramètre |Description |
|---|---|
|--num-executors|Définit le nombre approprié d’Exécuteurs.|
|--executor-cores|Définit le nombre de cœurs de chaque exécuteur. En règle générale, vous devez avoir des exécuteurs de taille moyenne, car d’autres processus consomment une partie de la mémoire disponible.|
|--executor-memory|Définit la taille de la mémoire pour chaque Exécuteur, ce qui contrôle la taille de segment de mémoire sur YARN. Conservez de la mémoire pour la surcharge d’exécution.|

## <a name="select-the-correct-executor-size"></a>Sélectionner la taille d’exécuteur correcte

Lors du choix de votre configuration d’exécuteur, n’oubliez pas de prendre en compte la surcharge de garbage collection Java.

* Facteurs permettant de réduire la taille de l’exécuteur :
    1. Réduisez la taille de segment de mémoire jusqu’à une valeur inférieure à 32 Go afin de maintenir la surcharge de garbage collection à moins de 10 %.
    2. Réduisez le nombre de cœurs afin de maintenir la surcharge de garbage collection à moins de 10 %.

* Facteurs permettant d’augmenter la taille de l’exécuteur :
    1. Réduisez la surcharge de communication entre les exécuteurs.
    2. Réduisez le nombre de connexions ouvertes entre les exécuteurs (N2) sur les clusters de grande taille (> 100 exécuteurs).
    3. Augmentez la taille de segment de mémoire afin de prendre en compte les tâches utilisant beaucoup de mémoire.
    4. Facultatif : réduisez la surcharge de mémoire par exécuteur.
    5. Facultatif : augmentez l’utilisation et la concurrence en surabonnant l’UC.

En règle générale, lors de la sélection de la taille d’Exécuteur :

1. Commencez avec 30 Go par exécuteur et distribuez les cœurs disponibles.
2. Augmentez le nombre de cœurs d’exécuteur pour les clusters de grande taille (> 100 exécuteurs).
3. Modifiez la taille en fonction des tests d’évaluation et de facteurs tels que la surcharge de garbage collection.

Lors de l’exécution de requêtes simultanées, tenez compte des points suivants :

1. Commencez avec 30 Go par exécuteur et tous les cœurs.
2. Créez plusieurs applications Spark parallèles en surabonnant les UC (amélioration de la latence d’environ 30 %).
3. Distribuez les requêtes parmi les applications parallèles.
4. Modifiez la taille en fonction des tests d’évaluation et de facteurs tels que la surcharge de garbage collection.

Pour plus d’informations sur l’utilisation d’Ambari pour configurer des exécuteurs, consultez [Paramètres Apache Spark : exécuteurs Spark](apache-spark-settings.md#configuring-spark-executors).

Surveillez les performances des requêtes afin de détecter les valeurs hors norme ou autres problèmes de performances en examinant l’affichage de la chronologie, ainsi que le graphique SQL, les statistiques des travaux, et ainsi de suite. Pour plus d’informations sur le débogage des travaux Spark à l’aide de YARN et du serveur d’historique Spark, consultez [Déboguer des travaux Apache Spark en cours d’exécution sur Azure HDInsight](apache-spark-job-debugging.md). Pour obtenir des conseils sur l’utilisation du serveur de chronologie YARN, consultez [Accéder aux journaux d’activité d’applications YARN Apache Hadoop](../hdinsight-hadoop-access-yarn-app-logs-linux.md).

Parfois, un ou plusieurs exécuteurs sont plus lents que les autres, et l’exécution des tâches prend beaucoup plus de temps. Cette lenteur se produit fréquemment sur les clusters de grande taille (> 30 nœuds). Dans ce cas, répartissez le travail parmi un plus grand nombre de tâches afin que le planificateur puisse compenser cette lenteur. Par exemple, faites en sorte d’avoir au moins deux fois plus de tâches que le nombre de cœurs d’exécuteur dans l’application. Vous pouvez également activer l’exécution spéculative des tâches avec `conf: spark.speculation = true`.

## <a name="next-steps"></a>Étapes suivantes

* [Optimiser le traitement de données pour Apache Spark](optimize-cluster-configuration.md)
* [Optimiser le stockage des données pour Apache Spark](optimize-data-storage.md)
* [Optimiser l’utilisation de la mémoire pour Apache Spark](optimize-memory-usage.md)
