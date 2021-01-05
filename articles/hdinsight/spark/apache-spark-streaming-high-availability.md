---
title: Travaux Spark Streaming hautement disponibles dans YARN - Azure HDInsight
description: Guide pratique pour configurer Apache Spark Streaming pour un scénario de haute disponibilité dans Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/29/2019
ms.openlocfilehash: 2ec0bf460a73f95e18e2e9221e8cbd8d4e14ff77
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86086209"
---
# <a name="create-high-availability-apache-spark-streaming-jobs-with-yarn"></a>Créer des tâches Apache Spark Streaming à haute disponibilité avec YARN

[Apache Spark](https://spark.apache.org/) Streaming permet d’implémenter des applications scalables à haut débit et à tolérance de panne pour le traitement des flux de données. Vous pouvez connecter des applications Spark Streaming sur un cluster HDInsight Spark à différentes sortes de sources de données, comme Azure Event Hubs, Azure IoT Hub, [Apache Kafka](https://kafka.apache.org/), [Apache Flume](https://flume.apache.org/), Twitter, [ZeroMQ](http://zeromq.org/) ou des sockets TCP bruts, ou en surveillant les modifications du système de fichiers [Apache Hadoop HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html). Spark Streaming prend en charge la tolérance de panne avec la garantie qu’un événement donné n’est traité qu’une seule fois, même en cas de défaillance d’un nœud.

Spark Streaming crée des travaux à exécution longue durant lesquels vous pouvez appliquer des transformations aux données, puis transmettre les résultats à des systèmes de fichiers, des bases de données, des tableaux de bord et la console. Spark Streaming traite des micro-lots de données, en recueillant d’abord un lot d’événements sur un intervalle de temps défini. Ensuite, ce lot est transféré pour le traitement et la sortie. Les intervalles des lots sont généralement définis en fractions de seconde.

![Spark Streaming](./media/apache-spark-streaming-high-availability/apache-spark-streaming.png)

## <a name="dstreams"></a>DStreams

Spark Streaming représente un flux continu de données utilisant un *flux de données discrétisé* (DStream). Ce DStream peut être créé à partir de sources d’entrée telles que Kafka ou Event Hubs, ou en appliquant des transformations à un autre DStream. Quand un événement arrive à votre application Spark Streaming, il est stocké de manière fiable. Autrement dit, les données d’événement sont répliquées afin que plusieurs nœuds en aient une copie. Cela garantit que l’arrêt d’un nœud n’entraînera pas la perte de votre événement.

Le cœur Spark utilise des *jeux de données distribués résilients* (RDD, Resilient Distributed Datasets). Les RDD distribuent les données parmi plusieurs nœuds du cluster, où chaque nœud conserve généralement ses données entièrement en mémoire pour des performances optimales. Chaque RDD représente les événements recueillis pendant un intervalle de lot. Quand l’intervalle de lot s’est écoulé, Spark Streaming génère un nouveau RDD contenant toutes les données dans cet intervalle. Cet ensemble continu de RDD est recueilli dans un DStream. Une application Spark Streaming traite les données stockées dans le RDD de chaque lot.

![Spark DStream](./media/apache-spark-streaming-high-availability/apache-spark-dstream.png)

## <a name="spark-structured-streaming-jobs"></a>Travaux Spark Structured Streaming

Spark Structured Streaming a été introduit dans Spark 2.0 comme moteur d’analyse pour une utilisation sur des données de streaming structurées. Spark Structured Streaming utilise les API du moteur de traitement par lot SparkSQL. Comme Spark Streaming, Spark Structured Streaming exécute ses calculs sur des micro-lots de données qui arrivent en continu. Spark Structured Streaming représente un flux de données sous forme de Table d’entrée avec un nombre de lignes illimité. Autrement dit, la Table d’entrée continue à croître à mesure que de nouvelles données arrivent. Cette Table d’entrée est traitée en continu par une longue requête, et les résultats sont écrits dans une Table de sortie.

![Spark Structured Streaming](./media/apache-spark-streaming-high-availability/structured-streaming.png)

Dans Structured Streaming, les données arrivent au système et sont ingérées immédiatement dans la Table d’entrée. Vous écrivez des requêtes qui effectuent des opérations sur cette Table d’entrée. La sortie de requête génère une autre table, appelée Table de résultats. La Table de résultats contient les résultats de votre requête, à partir desquels vous récupérez des données à envoyer à un magasin de données externe tel qu’une base de données relationnelle. *L’intervalle de déclenchement* définit quand les données de la Table d’entrée sont traitées. Par défaut, Structured Streaming traite les données dès qu’elles arrivent. Toutefois, vous pouvez également configurer le déclencheur pour qu’il s’exécute durant un intervalle plus long, afin que les données de streaming soient traitées dans des lots basés sur le temps. Les données de la table de résultats peuvent être actualisées systématiquement avec les nouvelles données, afin que la table contienne toutes les données de sortie générées depuis le début de la requête de streaming (*mode Complet*), ou la table peut contenir uniquement les nouvelles données qui ont été générées depuis la dernière exécution de la requête (*mode Ajout*).

## <a name="create-fault-tolerant-spark-streaming-jobs"></a>Créer des travaux Spark Streaming à tolérance de panne

Pour créer un environnement à haute disponibilité pour vos travaux Spark Streaming, vous devez avant tout développer chacun de vos travaux en vue de la récupération automatique en cas de défaillance. Ces travaux à récupération automatique tolèrent les pannes.

Les RDD ont plusieurs propriétés qui facilitent l’exécution des travaux Spark Streaming à haute disponibilité et à tolérance de panne :

* Les lots de données d’entrée stockés dans des RDD en tant que DStream sont répliqués automatiquement en mémoire à des fins de tolérance de panne.
* Les données perdues à cause de l’échec d’un worker peuvent être recalculées à partir des données d’entrée répliquées sur différents workers, tant que ces nœuds worker sont disponibles.
* La récupération rapide après incident peut se produire en moins d’une seconde, car la récupération à partir d’erreurs/délais a lieu par le biais du calcul en mémoire.

### <a name="exactly-once-semantics-with-spark-streaming"></a>Sémantique du traitement unique avec Spark Streaming

Pour créer une application qui ne traite chaque événement qu’une seule fois, réfléchissez à la façon dont tous les points de défaillance système redémarrent après un problème, et à la façon dont vous pouvez éviter la perte de données. La sémantique du traitement unique nécessite qu’aucune donnée ne soit perdue à aucun point, et que le traitement des messages puisse être redémarré, quel que soit le point où il s’est arrêté. Consultez [Créer des travaux Spark Streaming avec traitement unique des événements](apache-spark-streaming-exactly-once.md).

## <a name="spark-streaming-and-apache-hadoop-yarn"></a>Spark Streaming et Apache Hadoop YARN

Dans HDInsight, le travail de cluster est coordonné par *Yet Another Resource Negotiator* (YARN). La conception d’une haute disponibilité pour Spark Streaming inclut des techniques pour Spark Streaming et également pour les composants YARN.  Vous trouverez ci-dessous un exemple de configuration utilisant YARN.

![Architecture YARN](./media/apache-spark-streaming-high-availability/hdi-yarn-architecture.png)

Les sections suivantes décrivent les considérations en matière de conception pour cette configuration.

### <a name="plan-for-failures"></a>Planifier en vue des défaillances

Pour créer une configuration YARN pour la haute disponibilité, vous devez planifier en vue d’une défaillance possible de l’exécuteur ou du pilote. Certains travaux Spark Streaming impliquent également des exigences en matière de garantie de données qui nécessitent une installation et une configuration supplémentaires. Par exemple, une application de streaming peut avoir une exigence professionnelle de garantie de perte de données nulle malgré les erreurs qui se produisent dans le système de streaming hôte ou le cluster HDInsight.

Si un **exécuteur** échoue, ses tâches et ses récepteurs sont redémarrés automatiquement par Spark. Aucune modification de configuration n’est nécessaire.

Toutefois, si un **pilote** échoue, tous ses exécuteurs associés échouent, et tous les blocs reçus et les résultats de calculs sont perdus. Pour redémarrer un pilote qui s’est arrêté, utilisez des *points de contrôle DStream* comme décrit dans [Créer des travaux Spark Streaming avec traitement unique des événements](apache-spark-streaming-exactly-once.md#use-checkpoints-for-drivers). Les points de contrôle DStream enregistrent régulièrement le *graphe orienté acyclique* (DAG) des DStreams vers un stockage à tolérance de panne tel que le Stockage Azure.  Ils permettent à Spark Structured Streaming de redémarrer le pilote ayant échoué à partir des informations de point de contrôle.  Ce redémarrage de pilote lance de nouveaux exécuteurs et redémarre également des récepteurs.

Pour récupérer des pilotes avec des points de contrôle DStream

* Configurez le redémarrage automatique de pilote sur YARN avec le paramètre de configuration `yarn.resourcemanager.am.max-attempts`.
* Définissez un répertoire de points de contrôle dans un système de fichiers compatible HDFS avec `streamingContext.checkpoint(hdfsDirectory)`.
* Restructurez le code source pour utiliser des points de contrôle pour la récupération, par exemple :

    ```scala
        def creatingFunc() : StreamingContext = {
            val context = new StreamingContext(...)
            val lines = KafkaUtils.createStream(...)
            val words = lines.flatMap(...)
            ...
            context.checkpoint(hdfsDir)
        }

        val context = StreamingContext.getOrCreate(hdfsDir, creatingFunc)
        context.start()
    ```

* Configurez la récupération des données perdues en activant le journal WAL (write-ahead log) avec `sparkConf.set("spark.streaming.receiver.writeAheadLog.enable","true")`, et désactivez la réplication en mémoire pour les DStreams d’entrée avec `StorageLevel.MEMORY_AND_DISK_SER`.

Pour résumer, grâce aux points de contrôle, au journal WAL et à des récepteurs fiables, vous pourrez bénéficier d’une récupération des données « au moins une fois » :

* Exactement une fois, tant que les données reçues ne sont pas perdues et que les sorties sont idempotent ou transactionnelles.
* Exactement une fois, avec la nouvelle approche Kafka Direct qui utilise Kafka comme journal répliqué au lieu d’utiliser des récepteurs ou des journaux WAL.

### <a name="typical-concerns-for-high-availability"></a>Difficultés classiques concernant la haute disponibilité

* Il est plus difficile de surveiller des travaux de streaming que des travaux de traitement par lots. Les travaux Spark Streaming sont généralement longs, et YARN n’effectue l’agrégation des journaux d’activité que lorsqu’un travail est terminé.  Les points de contrôle Spark sont perdus pendant les mises à niveau d’application ou de Spark, et vous devrez effacer le répertoire de points de contrôle durant une mise à niveau.

* Configurez votre mode de cluster YARN pour exécuter les pilotes même en cas d’échec d’un client. Pour configurer le redémarrage automatique des pilotes :

    ```
    spark.yarn.maxAppAttempts = 2
    spark.yarn.am.attemptFailuresValidityInterval=1h
    ```

* Spark et l’interface utilisateur de Spark Streaming ont un système de métriques configurable. Vous pouvez également utiliser des bibliothèques supplémentaires, telles que Graphite/Grafana, pour télécharger des métriques de tableau de bord comme le nombre d’enregistrements traités, l’utilisation de la mémoire/du catalogue global sur le pilote et les exécuteurs, le retard total, l’utilisation du cluster, etc. Dans Structured Streaming version 2.1 ou ultérieure, vous pouvez utiliser `StreamingQueryListener` pour collecter des métriques supplémentaires.

* Vous devez segmenter les longs travaux.  Quand une application Spark Streaming est soumise au cluster, la file d’attente YARN où le travail s’exécute doit être définie. Vous pouvez utiliser un [Planificateur de capacité YARN](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/CapacityScheduler.html) pour envoyer des longs travaux vers des files d’attente distinctes.

* Arrêtez correctement votre application de streaming. Si vos offsets sont connus et que tout l’état de l’application est stocké en externe, vous pouvez arrêter par programmation votre application de streaming à l’emplacement approprié. Une technique consiste à utiliser des « crochets de thread » dans Spark, en vérifiant la présence d’un indicateur externe chaque *n* secondes. Vous pouvez également utiliser un *fichier marqueur* créé sur HDFS lors du démarrage de l’application, puis supprimé quand vous souhaitez l’arrêter. Pour l’approche faisant appel à un fichier marqueur, utilisez un thread distinct dans votre application Spark qui appelle du code semblable à celui-ci :

    ```scala
    streamingContext.stop(stopSparkContext = true, stopGracefully = true)
    // to be able to recover on restart, store all offsets in an external database
    ```

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble d’Apache Spark Streaming](apache-spark-streaming-overview.md)
* [Créer des travaux Apache Spark Streaming avec traitement unique des événements](apache-spark-streaming-exactly-once.md)
* [Tâches Apache Spark Streaming longues sur YARN](https://mkuthan.github.io/blog/2016/09/30/spark-streaming-on-yarn/)
* [Structured Streaming: Fault Tolerant Semantics (Structured Streaming : Sémantique de tolérance de panne)](https://spark.apache.org/docs/2.1.0/structured-streaming-programming-guide.html#fault-tolerance-semantics)
* [Discretized Streams: A Fault-Tolerant Model for Scalable Stream Processing (Flux discrétisés : Modèle à tolérance de panne pour le traitement de flux scalables)](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2012/EECS-2012-259.pdf)
