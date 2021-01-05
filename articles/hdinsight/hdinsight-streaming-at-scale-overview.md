---
title: Streaming à l’échelle dans Azure HDInsight
description: Comment utiliser le streaming de données avec des clusters Apache scalables dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 12/17/2019
ms.openlocfilehash: a04ce77c7e81a3a73b87eaf5790b383dece35d86
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022715"
---
# <a name="streaming-at-scale-in-hdinsight"></a>Diffusion en continu à l’échelle dans HDInsight

Les solutions Big Data en temps réel agissent sur des données qui sont en mouvement. En général, ces données sont le plus précieuses au moment de leur arrivée. Si le flux de données entrant devient plus grand que ce qui peut être traité à ce moment-là, vous devrez peut-être limiter les ressources. En guise d’alternative, un cluster HDInsight peut monter en puissance pour répondre aux besoins de votre solution de streaming en ajoutant des nœuds à la demande.

Dans une application de streaming, une ou plusieurs sources de données génèrent des événements (parfois plusieurs millions par seconde) qui doivent être ingérés rapidement sans supprimer aucune information utile. Les événements entrants sont traités avec la *mise en mémoire tampon du flux*, également appelée *mise en file d’attente des événements*, par un service comme [Apache Kafka](kafka/apache-kafka-introduction.md) ou [Event Hubs](https://azure.microsoft.com/services/event-hubs/). Après avoir collecté les événements, vous pouvez analyser les données à l’aide d’un système d’analytique en temps réel dans la couche de *traitement du flux*, tel que [Apache Storm](storm/apache-storm-overview.md) ou [Apache Spark Streaming](spark/apache-spark-streaming-overview.md). Les données traitées peuvent être stockées dans des systèmes de stockage à long terme, comme [Azure Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage/), et affichées en temps réel sur un tableau de bord de décisionnel, tel que [Power BI](https://powerbi.microsoft.com), Tableau ou une page web personnalisée.

![Modèles de streaming Azure HDInsight](./media/hdinsight-streaming-at-scale-overview/HDInsight-streaming-patterns.png)

## <a name="apache-kafka"></a>Apache Kafka

Apache Kafka fournit un service de mise en file d’attente des messages à débit élevé et à faible latence. Il fait désormais partie de la suite Apache d’Open Source Software (OSS). Kafka utilise un modèle de messagerie avec publication et abonnement, et il stocke des flux de données partitionnées en toute sécurité dans un cluster distribué et répliqué. Kafka se met à l’échelle de manière linéaire à mesure que le débit augmente.

Pour plus d’informations, consultez [Présentation d’Apache Kafka sur HDInsight](kafka/apache-kafka-introduction.md).

## <a name="apache-storm"></a>Apache Storm

Apache Storm est un système de calcul distribué, open source et tolérant aux pannes. Il est optimisé pour le traitement de flux de données en temps réel avec Hadoop. L’unité de base des données pour un événement entrant est un Tuple, qui est un ensemble immuable de paires clé/valeur. Une séquence illimitée de ces Tuples forme un Flux, qui est provient d’un Spout. Le Spout encapsule une source de données de streaming (par exemple Kafka) et émet des Tuples. Une topologie Storm est une séquence de transformations sur ces flux.

Pour plus d’informations, consultez [Présentation d’Apache Storm sur Azure HDInsight](storm/apache-storm-overview.md).

## <a name="spark-streaming"></a>Spark Streaming

Spark Streaming est une extension de Spark qui vous permet de réutiliser le même code que celui utilisé pour le traitement par lots. Vous pouvez combiner des requêtes interactives et par lots dans la même application. Contrairement à Storm, Spark Streaming fournit une sémantique de traitement avec état de type « exactement une fois ». En cas d’utilisation avec [l’API directe Kafka](https://spark.apache.org/docs/latest/streaming-kafka-integration.html), qui garantit que toutes les données Kafka sont reçues par Spark Streaming exactement une fois, il est possible d’obtenir des garanties de bout en bout « exactement une fois ». L’une des forces de Spark Streaming réside dans ses fonctionnalités de tolérance des pannes, qui permettent de récupérer rapidement les nœuds défectueux quand plusieurs nœuds sont utilisés au sein du cluster.

Pour plus d’informations, consultez [Présentation d’Apache Spark Streaming](./spark/apache-spark-streaming-overview.md).

## <a name="scaling-a-cluster"></a>Mise à l’échelle d’un cluster

Bien que vous puissiez spécifier le nombre de nœuds dans votre cluster lors de sa création, vous pourriez souhaiter augmenter ou réduire la taille du cluster pour l’adapter à la charge de travail. Tous les clusters HDInsight vous permettent de [changer le nombre de nœuds du cluster](hdinsight-administer-use-portal-linux.md#scale-clusters). Les clusters Spark peuvent être supprimés sans perte de données, puisque toutes les données sont stockées dans Stockage Azure ou Data Lake Storage.

Les technologies de découplage offrent des avantages. Par exemple, Kafka est une technologie de mise en mémoire tampon des événements. Elle consomme donc de très nombreuses E/S et n’a pas besoin de beaucoup de puissance de traitement. En comparaison, les processeurs de flux tels que Spark Streaming nécessitent beaucoup de ressources système et des machines virtuelles plus puissantes. En découplant ces technologies dans des clusters différents, vous pouvez les mettre à l’échelle indépendamment tout en exploitant au mieux les machines virtuelles.

### <a name="scale-the-stream-buffering-layer"></a>Mettre à l’échelle la couche de mise en mémoire tampon du flux

Les technologies de mise en mémoire tampon du flux Event Hubs and Kafka utilisent toutes deux des partitions, et les consommateurs lisent à partir de ces partitions. La mise à l’échelle du débit d’entrée nécessite l’augmentation du nombre de partitions, et l’ajout de partitions accroît le parallélisme. Dans Event Hubs, le nombre de partitions ne peut pas être modifié après le déploiement. Il est donc important de commencer en gardant l’échelle cible à l’esprit. Avec Kafka, il est possible [d’ajouter des partitions](https://kafka.apache.org/documentation.html#basic_ops_cluster_expansion), même pendant que Kafka traite des données. Kafka fournit un outil pour réattribuer des partitions, `kafka-reassign-partitions.sh`. HDInsight fournit un [outil de rééquilibrage de réplica de partition](https://github.com/hdinsight/hdinsight-kafka-tools), `rebalance_rackaware.py`. Cet outil de rééquilibrage appelle l’outil `kafka-reassign-partitions.sh` de telle sorte que chaque réplica soit dans un domaine d’erreur et un domaine de mise à jour distinct, ce qui accroît la tolérance de panne et permet à Kafka de prendre en charge les racks.

### <a name="scale-the-stream-processing-layer"></a>Mettre à l’échelle la couche de traitement du flux

Apache Storm et Spark Streaming prennent tous deux en charge l’ajout de nœuds worker à leurs clusters, même pendant le traitement des données.

Pour tirer parti des nouveaux nœuds ajoutés par le biais de la mise à l’échelle de Storm, vous devez rééquilibrer les topologies Storm démarrées avant l’augmentation de la taille du cluster. Vous pouvez effectuer ce rééquilibrage à l’aide de l’interface utilisateur web de Storm ou de son interface CLI. Pour plus d’informations, consultez la [documentation d’Apache Storm](https://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html).

Apache Spark utilise trois principaux paramètres pour la configuration de son environnement, en fonction des spécifications de l’application : `spark.executor.instances`, `spark.executor.cores` et `spark.executor.memory`. Un *exécuteur* est un processus lancé pour une application Spark. Un exécuteur s’exécute sur le nœud worker et est responsable de l’exécution des tâches de l’application. Le nombre d’exécuteurs par défaut et les tailles d’exécuteur de chaque cluster sont calculés en fonction du nombre de nœuds worker et de leur taille. Ces nombres sont stockés dans le fichier `spark-defaults.conf` sur chaque nœud principal du cluster.

Ces trois paramètres peuvent être configurés au niveau du cluster (pour toutes les applications qui s’exécutent sur le cluster), et peuvent aussi être spécifiés pour chaque application. Pour plus d’informations, consultez [Gérer les ressources des clusters Apache Spark](spark/apache-spark-resource-manager.md).

## <a name="next-steps"></a>Étapes suivantes

* [Créer et surveiller une topologie Apache Storm dans Azure HDInsight à votre projet](storm/apache-storm-quickstart.md)
* [Exemples de topologies pour Apache Storm dans HDInsight](storm/apache-storm-example-topology.md)
* [Présentation d’Apache Spark sur HDInsight](spark/apache-spark-overview.md)
* [Démarrer avec Apache Kafka sur HDInsight](kafka/apache-kafka-get-started.md)