---
title: Spark Streaming et traitement unique des événements – Azure HDInsight
description: Comment configurer Apache Spark Streaming pour traiter un événement une seule fois.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 11/15/2018
ms.openlocfilehash: 8e0037f6aea4aef53efc192066027e0a0143bda1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86086175"
---
# <a name="create-apache-spark-streaming-jobs-with-exactly-once-event-processing"></a>Créer des tâches Apache Spark Streaming avec traitement unique des événements

Les applications de traitement de flux adoptent différentes approches quant à la façon dont elles gèrent le retraitement des messages après une panne dans le système :

* Au moins une fois : chaque message est traité, mais il peut l’être plusieurs fois.
* Au plus une fois : chaque message peut être traité ou non. Si un message est traité, il ne l’est qu’une seule fois.
* Une seule fois : chaque message est traité, mais une seule fois.

Cet article explique comment configurer Spark Streaming afin de traiter les événements une seule fois.

## <a name="exactly-once-semantics-with-apache-spark-streaming"></a>Sémantique du traitement unique avec Apache Spark Streaming

Commencez par analyser comment tous les points de défaillance du système redémarrent après un problème et comment vous pouvez éviter une perte de données. Une application Spark Streaming a :

* Une source d’entrée.
* Un ou plusieurs processus récepteur qui extraient des données de la source d’entrée.
* Des tâches qui traitent les données.
* Un récepteur de sortie.
* Un processus pilote qui gère le travail à long terme.

La sémantique du traitement unique nécessite qu’aucune donnée ne soit perdue à aucun point, et que le traitement des messages puisse être redémarré, quel que soit l’endroit où la défaillance se produit.

### <a name="replayable-sources"></a>Sources pouvant être relues

La source à partir de laquelle votre application Spark Streaming lit vos événements doit *pouvoir être relue*. Cela signifie que si le message est récupéré, mais qu’ensuite le système connait une défaillance avant d’avoir pu conserver ou traiter le message, la source doit refournir le même message.

Dans Azure, Azure Event Hubs et [Apache Kafka](https://kafka.apache.org/) sur HDInsight fournissent des sources pouvant être relues. Parmi les autres sources pouvant être relues figurent les systèmes de fichiers à tolérance de panne, comme [Apache Hadoop HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html), les objets Blob du stockage Azure ou Azure Data Lake Storage, où toutes les données sont conservées indéfiniment, et où vous pouvez relire ces données dans leur intégralité, à tout moment.

### <a name="reliable-receivers"></a>Récepteurs fiables

Dans Spark Streaming, des sources telles qu’Event Hubs et Kafka ont des *récepteurs fiables* ; chaque récepteur effectue le suivi de sa progression de la lecture de la source. Un récepteur fiable conserve son état dans un stockage à tolérance de panne, soit dans [Apache ZooKeeper](https://zookeeper.apache.org/), soit dans des points de contrôle Spark Streaming écrits dans HDFS. Si un récepteur de ce type échoue, puis est redémarré, il peut reprendre là où il s’était arrêté.

### <a name="use-the-write-ahead-log"></a>Utiliser le journal WAL (write-ahead log)

Spark Streaming prend en charge l’utilisation d’un journal WAL (write-ahead log), où chaque événement reçu est écrit dans le répertoire des points de contrôle de Spark dans un stockage à tolérance de panne, puis stocké dans un jeu de données distribué résilient (RDD). Dans Azure, le stockage à tolérance de panne est HDFS, il repose sur le stockage Azure ou Azure Data Lake Storage. Dans votre application Spark Streaming, vous pouvez activer le journal WAL (write-ahead log) pour tous les récepteurs en définissant le paramètre de configuration `spark.streaming.receiver.writeAheadLog.enable` sur `true`. Le journal WAL (write-ahead log) fournit une tolérance de panne pour les échecs du pilote et des Exécuteurs.

Pour les Workers exécutant des tâches sur les données d’événement, chaque RDD est par définition à la fois répliqué et distribué sur plusieurs Workers. Si une tâche échoue en raison d’une panne du Worker en cours d’exécution, elle est redémarrée sur un autre Worker qui a un réplica des données d’événement ; ainsi, l’événement n’est pas perdu.

### <a name="use-checkpoints-for-drivers"></a>Utiliser des points de contrôle pour les pilotes

Les pilotes de travail doivent pouvoir être redémarrés. Si le pilote exécutant votre application Spark Streaming tombe en panne, il entraîne tous les récepteurs en cours d’exécution, les tâches et tout RDD stockant des données d’événement. Dans ce cas, vous devez être en mesure d’enregistrer la progression du travail afin de pouvoir le reprendre ultérieurement. Pour ce faire, vous pouvez contrôler régulièrement le graphe orienté acyclique (DAG) du flux discrétisé dans le stockage à tolérance de panne. Les métadonnées du DAG incluent la configuration utilisée pour créer l’application de streaming, les opérations qui définissent l’application et tous les lots éventuellement en attente d’exécution. Ces métadonnées permettent de redémarrer un pilote défectueux à partir des informations de points de contrôle. Quand le pilote redémarre, il lance de nouveaux récepteurs qui eux-mêmes récupèrent les données d’événement dans les RDD à partir du journal WAL (write-ahead log).

Vous activez les points de contrôle dans Spark Streaming en deux étapes.

1. Dans l’objet StreamingContext, configurez le chemin du stockage des points de contrôle :

    ```Scala
    val ssc = new StreamingContext(spark, Seconds(1))
    ssc.checkpoint("/path/to/checkpoints")
    ```

    Dans HDInsight, ces points de contrôle doivent être enregistrés dans le stockage par défaut associé à votre cluster (Stockage Azure ou Azure Data Lake Storage).

2. Ensuite, spécifiez un intervalle de point de contrôle (en secondes) sur le flux discrétisé. À chaque intervalle, les données d’état dérivées de l’événement d’entrée sont rendues persistantes dans le stockage. Les données d’état persistantes peuvent réduire le calcul nécessaire à la regénération de l’état à partir de l’événement source.

    ```Scala
    val lines = ssc.socketTextStream("hostname", 9999)
    lines.checkpoint(30)
    ssc.start()
    ssc.awaitTermination()
    ```

### <a name="use-idempotent-sinks"></a>Utiliser des récepteurs idempotents

Le récepteur de destination dans lequel votre travail écrit les résultats doit être en mesure de gérer les situations où il reçoit un même résultat plusieurs fois. Le récepteur doit être en mesure de détecter ces résultats en double et de les ignorer. Un récepteur *idempotent* peut être appelé plusieurs fois avec les mêmes données sans aucun changement d’état.

Vous pouvez créer des récepteurs idempotents en implémentant une logique qui d’abord vérifie l’existence du résultat entrant dans la banque de données. Si le résultat existe déjà, l’écriture doit réussir du point de vue de votre travail Spark, mais en réalité votre banque de données a ignoré les données en double. Si le résultat n’existe pas, le récepteur doit insérer ce nouveau résultat dans son stockage.

Par exemple, vous pouvez utiliser une procédure stockée avec Azure SQL Database qui insère des événements dans une table. Cette procédure stockée recherche l’événement à partir de champs clés, puis n’insère l’enregistrement dans la table que si elle ne trouve aucun événement correspondant.

Un autre exemple consiste à utiliser un système de fichiers partitionné, comme les objets blob du stockage Azure ou Azure Data Lake Storage. Dans ce cas, votre logique de récepteur n’a pas besoin de vérifier l’existence d’un fichier. Si le fichier représentant l’événement existe, il est simplement remplacé par les mêmes données. Sinon, un fichier est créé dans le chemin calculé.

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble d’Apache Spark Streaming](apache-spark-streaming-overview.md)
* [Création de tâches Apache Spark Streaming hautement disponibles dans Apache Hadoop YARN](apache-spark-streaming-high-availability.md)
