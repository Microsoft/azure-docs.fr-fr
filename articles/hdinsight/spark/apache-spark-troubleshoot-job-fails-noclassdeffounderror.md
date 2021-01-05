---
title: NoClassDefFoundError – Apache Spark avec des données Apache Kafka dans Azure HDInsight
description: Le travail de diffusion en continu Apache Spark qui lit des données à partir d’un cluster Apache Kafka échoue en générant une erreur NoClassDefFoundError dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/29/2019
ms.openlocfilehash: c523dbc4612ebfcebf3923900b31f3a2a7b0c5c6
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93287769"
---
# <a name="apache-spark-streaming-job-that-reads-apache-kafka-data-fails-with-noclassdeffounderror-in-hdinsight"></a>Le travail de diffusion en continu Apache Spark qui lit des données Apache Kafka échoue en générant une erreur NoClassDefFoundError dans Azure HDInsight

Cet article décrit la procédure à suivre pour résoudre les problèmes rencontrés lors de l’utilisation de composants Apache Spark dans des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Le cluster Apache Spark exécute une tâche de diffusion en continu Spark qui lit les données à partir d’un cluster Apache Kafka. La tâche de diffusion en continu Spark échoue si la compression du flux Kafka est activée. Dans ce cas, l’application Yarn de diffusion en continu Spark, application_1525986016285_0193, échoue en raison de l’erreur suivante :

```
18/05/17 20:01:33 WARN YarnAllocator: Container marked as failed: container_e25_1525986016285_0193_01_000032 on host: wn87-Scaled.2ajnsmlgqdsutaqydyzfzii3le.cx.internal.cloudapp.net. Exit status: 50. Diagnostics: Exception from container-launch.
Container id: container_e25_1525986016285_0193_01_000032
Exit code: 50
Stack trace: ExitCodeException exitCode=50: 
 at org.apache.hadoop.util.Shell.runCommand(Shell.java:944)
```

## <a name="cause"></a>Cause :

Cette erreur peut être due à la spécification d’une version du fichier jar `spark-streaming-kafka` autre que celle du cluster Kafka que vous exécutez.

Par exemple, si vous exécutez un cluster Kafka version 0.10.1, la commande suivante génère une erreur :

```
spark-submit \
--packages org.apache.spark:spark-streaming-kafka-0-8_2.11:2.2.0
--conf spark.executor.instances=16 \
...
~/Kafka_Spark_SQL.py <bootstrap server details>
```

## <a name="resolution"></a>Résolution

Utilisez la commande Spark-submit avec l’option `–packages` et assurez-vous que la version du fichier jar spark-streaming-kafka est identique à celle du cluster Kafka que vous exécutez.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]