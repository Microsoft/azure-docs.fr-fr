---
title: Erreur RequestBodyTooLarge de l’application Apache Spark - Azure HDInsight
description: NativeAzureFileSystem ... RequestBodyTooLarge s’affiche dans le journal de l’application de diffusion en continu Apache Spark dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/29/2019
ms.openlocfilehash: 38d6e5bfea1ae7ad4eead3a3f614007d31f0a7cb
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93287927"
---
# <a name="nativeazurefilesystemrequestbodytoolarge-appear-in-apache-spark-streaming-app-log-in-hdinsight"></a>« NativeAzureFileSystem...RequestBodyTooLarge » s’affiche dans le journal de l'application de diffusion en continu Apache Spark dans HDInsight

Cet article décrit la procédure à suivre pour résoudre les problèmes rencontrés lors de l’utilisation de composants Apache Spark dans des clusters Azure HDInsight.

## <a name="issue"></a>Problème

L’erreur `NativeAzureFileSystem ... RequestBodyTooLarge` s’affiche dans le journal du pilote d'une application de diffusion en continu Apache Spark.

## <a name="cause"></a>Cause

Votre fichier journal des événements Spark a probablement atteint la limite de longueur de fichier pour WASB.

Dans Spark 2.3, chaque application Spark génère un fichier journal des événements Spark. Le fichier journal des événements Spark pour une application de diffusion en continu Spark continue d'augmenter tant que l'application est exécutée. Actuellement, un fichier sur WASB présente une limite de bloc de 50000, et la taille de bloc par défaut est de 4 Mo. Ainsi, dans la configuration par défaut, la taille maximale des fichiers est de 195 Go. Toutefois, le Stockage Azure a augmenté la taille de bloc maximale à 100 Mo, ce qui a repoussé la limite des fichiers à 4,75 To. Pour plus d’informations, consultez [Objectifs de performance et de scalabilité pour le stockage d’objets blob](../../storage/blobs/scalability-targets.md).

## <a name="resolution"></a>Résolution

Trois solutions sont disponibles pour cette erreur :

* Augmentez la taille de bloc jusqu’à 100 Mo. Dans l’interface utilisateur Ambari, modifiez la propriété de configuration HDFS `fs.azure.write.request.size` (ou créez-la dans la section `Custom core-site`). Définissez la propriété sur une plus grande valeur, par exemple : 33554432. Enregistrez la configuration mise à jour et redémarrez les composants concernés.

* Arrêtez et renvoyez régulièrement le travail spark-streaming.

* Utilisez HDFS pour stocker les journaux des événements Spark. L’utilisation de HDFS à des fins de stockage peut entraîner une perte de données liées aux événements Spark lors de la mise à l’échelle du cluster ou les mises à niveau Azure.

    1. Modifiez `spark.eventlog.dir` et `spark.history.fs.logDirectory` via l’interface utilisateur Ambari :

        ```
        spark.eventlog.dir = hdfs://mycluster/hdp/spark2-events
        spark.history.fs.logDirectory = "hdfs://mycluster/hdp/spark2-events"
        ```

    1. Créez des répertoires sur HDFS :

        ```
        hadoop fs -mkdir -p hdfs://mycluster/hdp/spark2-events
        hadoop fs -chown -R spark:hadoop hdfs://mycluster/hdp
        hadoop fs -chmod -R 777 hdfs://mycluster/hdp/spark2-events
        hadoop fs -chmod -R o+t hdfs://mycluster/hdp/spark2-events
        ```

    1. Redémarrez tous les services concernés via l’interface utilisateur Ambari.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]