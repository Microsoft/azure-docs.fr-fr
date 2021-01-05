---
title: Journaux Azure Monitor pour Apache Kafka - Azure HDInsight
description: Découvrez comment utiliser les journaux Azure Monitor pour analyser les journaux d’activité du cluster Apache Kafka sur Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 02/17/2020
ms.openlocfilehash: 5128ac7608dfce08471f7a7f97ed28a7971e62b2
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92534445"
---
# <a name="analyze-logs-for-apache-kafka-on-hdinsight"></a>Analyser les journaux d’activité pour Apache Kafka sur HDInsight

Découvrez comment utiliser les journaux Azure Monitor pour analyser les journaux d’activité générés par Apache Kafka sur HDInsight.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="logs-location"></a>Emplacement des journaux

Les journaux Apache Kafka du cluster se trouvent sous `/var/log/kafka`. Les journaux Kafka ne sont pas enregistrés ni conservés sur les cycles de vie des clusters, et ce, que des disques managés soient utilisés ou non. Le tableau suivant présente les journaux disponibles.

|Journal |Description |
|---|---|
|kafka.out|stdout et stderr du processus Kafka. Vous trouverez les journaux de démarrage et d’arrêt Kafka dans ce fichier.|
|server.log|Journal principal du serveur Kafka. Tous les journaux du répartiteur Kafka se trouvent ici.|
|controller.log|Journaux de contrôleur si le répartiteur agit comme contrôleur.|
|statechange.log|Tous les événements de changement d’état des répartiteurs sont journalisés dans ce fichier.|
|kafka-gc.log|Statistiques de nettoyage de la mémoire Kafka.|

## <a name="enable-azure-monitor-logs-for-apache-kafka"></a>Activer les journaux Azure Monitor pour Apache Kafka

Les étapes permettant d’activer les journaux Azure Monitor pour HDInsight sont les mêmes pour tous les clusters HDInsight. Utilisez les liens suivants pour apprendre à créer et à configurer les services requis :

1. Créez un espace de travail Log Analytics. Pour plus d’informations, consultez le document [Journaux d’activité dans Azure Monitor](../../azure-monitor/platform/data-platform-logs.md).

2. Créez un cluster Kafka dans HDInsight. Pour plus d’informations, consultez le document [Démarrer avec Apache Kafka dans HDInsight](apache-kafka-get-started.md).

3. Configurez le cluster Kafka pour utiliser les journaux Azure Monitor. Pour plus d’informations, consultez le document [Use Azure Monitor logs to monitor HDInsight](../hdinsight-hadoop-oms-log-analytics-tutorial.md) (Utiliser les journaux Azure Monitor pour surveiller HDInsight).

> [!IMPORTANT]  
> Un délai de 20 minutes peut s’écouler avant que les données ne soient disponibles pour les journaux Azure Monitor.

## <a name="query-logs"></a>Journaux d’activité des requêtes

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez votre espace de travail Log Analytics.

2. Dans le menu de gauche, sous **Général** , sélectionnez **Journaux d’activité** . À ce stade, vous pouvez rechercher les données collectées par Kafka. Entrez une requête dans la fenêtre de requête, puis sélectionnez **Exécuter** . Voici quelques exemples de recherches :

* Utilisation du disque :

    ```kusto
    Perf
    | where ObjectName == "Logical Disk" and CounterName == "Free Megabytes" and InstanceName == "_Total" and ((Computer startswith_cs "hn" and Computer contains_cs "-") or (Computer startswith_cs "wn" and Computer contains_cs "-")) 
    | summarize AggregatedValue = avg(CounterValue) by Computer, bin(TimeGenerated, 1h)
    ```

* Utilisation de l’UC :

    ```kusto
    Perf 
    | where CounterName == "% Processor Time" and InstanceName == "_Total" and ((Computer startswith_cs "hn" and Computer contains_cs "-") or (Computer startswith_cs "wn" and Computer contains_cs "-")) 
    | summarize AggregatedValue = avg(CounterValue) by Computer, bin(TimeGenerated, 1h)
    ```

* Messages entrants par seconde : (Remplacez `your_kafka_cluster_name` par le nom de votre cluster.)

    ```kusto
    metrics_kafka_CL 
    | where ClusterName_s == "your_kafka_cluster_name" and InstanceName_s == "kafka-BrokerTopicMetrics-MessagesInPerSec-Count" 
    | summarize AggregatedValue = avg(kafka_BrokerTopicMetrics_MessagesInPerSec_Count_value_d) by HostName_s, bin(TimeGenerated, 1h)
    ```

* Octets entrants par seconde : (Remplacez `wn0-kafka` par un nom d’hôte du nœud Worker.)

    ```kusto
    metrics_kafka_CL 
    | where HostName_s == "wn0-kafka" and InstanceName_s == "kafka-BrokerTopicMetrics-BytesInPerSec-Count" 
    | summarize AggregatedValue = avg(kafka_BrokerTopicMetrics_BytesInPerSec_Count_value_d) by bin(TimeGenerated, 1h)
    ```

* Octets sortants par seconde : (Remplacez `your_kafka_cluster_name` par le nom de votre cluster.)

    ```kusto
    metrics_kafka_CL 
    | where ClusterName_s == "your_kafka_cluster_name" and InstanceName_s == "kafka-BrokerTopicMetrics-BytesOutPerSec-Count" 
    | summarize AggregatedValue = avg(kafka_BrokerTopicMetrics_BytesOutPerSec_Count_value_d) by bin(TimeGenerated, 1h)
    ```

    Vous pouvez également entrer `*` pour rechercher tous les types consignés. Actuellement, les fichiers journaux d’activité suivants sont disponibles pour les requêtes :

    | Type de journal | Description |
    | ---- | ---- |
    | log\_kafkaserver\_CL | Kafka broker server.log |
    | log\_kafkacontroller\_CL | Kafka broker controller.log |
    | metrics\_kafka\_CL | Mesures JMX Kafka |

    ![Apache Kafka - Utilisation de l’UC dans Log Analytics](./media/apache-kafka-log-analytics-operations-management/apache-kafka-cpu-usage.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure Monitor, consultez [Vue d’ensemble d’Azure Monitor](../../azure-monitor/overview.md) et [Interroger les journaux Azure Monitor pour surveiller les clusters HDInsight](../hdinsight-hadoop-oms-log-analytics-use-queries.md).

Pour plus d’informations sur l’utilisation d’Apache Kafka, consultez les documents suivants :

* [Mettre en miroir Apache Kafka entre plusieurs clusters HDInsight](apache-kafka-mirroring.md)
* [Augmenter l’échelle d’Apache Kafka sur HDInsight](apache-kafka-scalability.md)
* [Utiliser le streaming Apache Spark (DStreams) avec Apache Kafka](../hdinsight-apache-spark-with-kafka.md)
* [Utiliser Apache Spark Structured Streaming avec Apache Kafka](../hdinsight-apache-kafka-spark-structured-streaming.md)