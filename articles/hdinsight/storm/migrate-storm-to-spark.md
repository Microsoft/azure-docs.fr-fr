---
title: Migrer Azure HDInsight 3.6 Apache Storm vers HDInsight 4.0 Apache Spark
description: Découvrez les différences et le flux de migration pour la migration des charges de travail Apache Storm vers Spark Streaming ou Spark Structured Streaming.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 01/16/2019
ms.openlocfilehash: e1262a4699bc42cb5b9a4398be2254854c5d5ff2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86081194"
---
# <a name="migrate-azure-hdinsight-36-apache-storm-to-hdinsight-40-apache-spark"></a>Migrer Azure HDInsight 3.6 Apache Storm vers HDInsight 4.0 Apache Spark

Ce document décrit comment migrer des charges de travail Apache Storm de HDInsight 3.6 vers HDInsight 4.0. HDInsight 4.0 ne prend pas en charge le type de cluster Apache Storm, et vous devrez migrer vers une autre plateforme de données de streaming. Deux options appropriées sont Apache Spark Streaming et Spark Structured Streaming. Ce document décrit les différences entre ces plateformes et recommande un flux de travail pour la migration des charges de travail Apache Storm.

## <a name="storm-migration-paths-in-hdinsight"></a>Chemins de migration Storm dans HDInsight

Si vous souhaitez migrer à partir d’Apache Storm sur HDInsight 3.6, vous avez plusieurs options :

* Spark Streaming sur HDInsight 4.0
* Spark Structured Streaming sur HDInsight 4.0
* Azure Stream Analytics

Ce document fournit un guide pour la migration d’Apache Storm vers Spark Streaming et Spark Structured Streaming.

> [!div class="mx-imgBorder"]
> ![Chemin de migration HDInsight Storm](./media/migrate-storm-to-spark/storm-migration-path.png)

## <a name="comparison-between-apache-storm-and-spark-streaming-spark-structured-streaming"></a>Comparaison entre Apache Storm et Spark Streaming, Spark Structured Streaming

Apache Storm peut fournir différents niveaux de traitement de message garanti. Par exemple, une application Storm de base peut garantir un traitement « Une fois au minimum », tandis que [Trident](https://storm.apache.org/releases/current/Trident-API-Overview.html) peut garantir un traitement « Exactement une fois ». Spark Streaming et Spark Structured Streaming garantissent que tout événement d’entrée est traité une seule fois, même en cas de défaillance d’un nœud. Storm a un modèle qui traite chaque événement unique, et vous pouvez également utiliser le modèle Micro-Batch avec Trident. Spark Streaming et Spark Structured Streaming fournissent un modèle de traitement Micro-Batch.

|  |Storm |Streaming Spark | Spark Structured Streaming|
|---|---|---|---|
|**Garantie de traitement des événements**|Au moins une fois <br> Une seule fois (Trident) |[Une seule fois](https://spark.apache.org/docs/latest/streaming-programming-guide.html)|[Une seule fois](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)|
|**Modèle de traitement**|Temps réel <br> Micro-Batch  (Trident) |Micro-Batch |Micro-Batch |
|**Prise en charge de l’heure de l’événement**|[Oui](https://storm.apache.org/releases/2.0.0/Windowing.html)|Non|[Oui](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)|
|**Langues**|Java, etc.|Scala, Java, Python|Python, R, Scala, Java, SQL|

### <a name="spark-streaming-vs-spark-structured-streaming"></a>Spark Streaming vs Spark Structured Streaming

Spark Structured Streaming remplace Spark Streaming (DStreams). Structured Streaming continuera à bénéficier d’améliorations et de maintenance, contrairement à DStreams qui sera proposé uniquement en mode maintenance. **Remarque : Liens nécessaire pour insister sur ce point**. Structured Streaming n’offre pas autant de fonctionnalités que DStreams pour les sources et les récepteurs pris en charge par défaut. Veillez donc à bien évaluer vos besoins pour choisir l’option de traitement de flux Spark qui vous convient le mieux.

## <a name="streaming-single-event-processing-vs-micro-batch-processing"></a>Traitement en streaming (événement unique) vs traitement Micro-Batch

Storm fournit un modèle qui traite chaque événement unique. Cela signifie que tous les enregistrements entrants sont traités dès leur arrivée. Les applications Spark Streaming doivent attendre quelques fractions de seconde avant de pouvoir collecter chaque micro-batch d’événements et envoyer ce lot pour traitement. En revanche, une application basée sur les événements traite chaque événement immédiatement. La latence de Spark Streaming est généralement inférieure à quelques secondes. Les avantages de l’approche par micro-lots sont un traitement plus efficace des données et des calculs d’agrégation plus simples.

> [!div class="mx-imgBorder"]
> ![streaming et traitement micro-batch](./media/migrate-storm-to-spark/streaming-and-micro-batch-processing.png)

## <a name="storm-architecture-and-components"></a>Architecture et composants de Storm

Les topologies Storm sont constituées de plusieurs composants organisés dans un graphe orienté acyclique (DAG). Flux de données entre les composants dans le graphe. Chaque composant consomme un ou plusieurs flux de données, et peut émettre éventuellement un ou plusieurs flux.

|Composant |Description |
|---|---|
|Spout|Intègre des données dans une topologie. Ils émettent un ou plusieurs flux dans la topologie.|
|Bolt|Consomme des flux émis à partir de spouts ou d’autres bolts. Les bolts peuvent éventuellement émettre des flux vers la topologie. Ils sont également responsables de l’écriture des données dans les services externes ou le stockage, comme HDFS, Kafka ou HBase.|

> [!div class="mx-imgBorder"]
> ![interaction des composants Storm](./media/migrate-storm-to-spark/apache-storm-components.png)

Storm est constitué des trois démons suivants, qui assurent le fonctionnement du cluster Storm.

|Daemon |Description |
|---|---|
|Nimbus|Tout comme Hadoop JobTracker, il est chargé de la distribution du code dans l‘ensemble du cluster, de l’attribution des tâches aux machines et de la supervision des défaillances.|
|Zookeeper|Utilisé pour la coordination de cluster.|
|Supervisor|Écoute le travail affecté à sa machine, et démarre et arrête les processus Worker en fonction des directives de Nimbus. Chaque processus Worker exécute un sous-ensemble d’une topologie. La logique d’application de l’utilisateur (Spouts et Bolt) s’exécute ici.|

> [!div class="mx-imgBorder"]
> ![démons nimbus, zookeeper et supervisor](./media/migrate-storm-to-spark/nimbus-zookeeper-supervisor.png)

## <a name="spark-streaming-architecture-and-components"></a>Architecture et composants de Spark Streaming

Les étapes suivantes constituent un résumé de la coordination entre les composants dans Spark Streaming (DStreams) et Spark Structured Streaming :

* Quand Spark Streaming est lancé, le pilote lance la tâche dans l’exécuteur.
* L’exécuteur reçoit un flux à partir d’une source de données de streaming.
* Quand l’exécuteur reçoit des flux de données, il les divise en blocs et les conserve en mémoire.
* Les blocs de données sont répliqués vers d’autres exécuteurs.
* Les données traitées sont ensuite stockées dans le magasin de données cible.

> [!div class="mx-imgBorder"]
> ![chemin de streaming spark vers la sortie](./media/migrate-storm-to-spark/spark-streaming-to-output.png)

## <a name="spark-streaming-dstream-workflow"></a>Workflow Spark Streaming (DStream)

Au fur et à mesure que chacun de ces intervalles s’écoule, un nouveau RDD contenant toutes les données de cet intervalle est produit. Les ensembles continus de RDD sont collectés dans un DStream. Par exemple, si l’intervalle de traitement par lots est d’une seconde, votre DStream émet chaque seconde un lot incluant un RDD qui contient toutes les données ingérées durant cette seconde. Lors du traitement du DStream, l’événement de température apparaît dans l’un de ces lots. Une application Spark Streaming traite les lots qui contiennent les événements et agit en fin de compte sur les données stockées dans chaque RDD.

> [!div class="mx-imgBorder"]
> ![lots de traitement spark streaming](./media/migrate-storm-to-spark/spark-streaming-batches.png)

Pour plus d’informations sur les différentes transformations disponibles avec Spark Streaming, consultez [Transformations on DStreams](https://spark.apache.org/docs/latest/streaming-programming-guide.html#transformations-on-dstreams).

## <a name="spark-structured-streaming"></a>Spark Structured Streaming

Spark Structured Streaming représente un flux de données sous la forme d’une table qui n’est pas liée en profondeur. La table continue à croître à mesure que de nouvelles données arrivent. Cette table d’entrée est traitée en continu par une longue requête, et les résultats sont écrits dans une table de sortie.

Dans Structured Streaming, les données arrivent au système et sont ingérées immédiatement dans une table d’entrée. Vous écrivez des requêtes (à l’aide des API DataFrame et DataSet)qui effectuent des opérations sur cette table d’entrée.

Le résultat de la requête génère une *table de résultats* qui contient les résultats de votre requête. Vous pouvez extraire des données à partir de la table de résultats pour un magasin de données externe, tel qu’une base de données relationnelle.

Le moment où les données de la table d’entrée sont traitées est contrôlé par l’intervalle de déclenchement. Par défaut, l’intervalle de déclencheur est défini sur zéro, ce qui signifie que Structured Streaming tente de traiter les données dès qu’elles arrivent. Concrètement, dès que Structured Streaming a fini l’exécution de la requête précédente, il démarre un autre flux de traitement sur toutes les nouvelles données reçues. Vous pouvez configurer le déclencheur pour qu’il s’exécute à un intervalle défini, afin que les données de diffusion en continu soient traitées dans des lots basés sur le temps.

> [!div class="mx-imgBorder"]
> ![traitement des données dans structured streaming](./media/migrate-storm-to-spark/structured-streaming-data-processing.png)

> [!div class="mx-imgBorder"]
> ![modèle de programmation pour structured streaming](./media/migrate-storm-to-spark/structured-streaming-model.png)

## <a name="general-migration-flow"></a>Flux de migration général

Le flux de migration recommandé de Storm à Spark suppose l’architecture initiale suivante :

* Kafka est utilisé comme source de données de streaming
* Kafka et Storm sont déployés sur le même réseau virtuel
* Les données traitées par Storm sont écrites dans un récepteur de données, tel que Stockage Azure ou Azure Data Lake Storage Gen2.

    > [!div class="mx-imgBorder"]
    > ![diagramme de l’environnement actuel présumé](./media/migrate-storm-to-spark/presumed-current-environment.png)

Pour migrer votre application de Storm vers l’une des API de streaming Spark, effectuez les étapes suivantes :

1. **Déployez un nouveau cluster.** Déployez un nouveau cluster Spark HDInsight 4.0 sur le même réseau virtuel, déployez votre application Spark Streaming ou Spark Structured Streaming sur ce dernier et testez-la de manière approfondie.

    > [!div class="mx-imgBorder"]
    > ![nouveau déploiement spark dans HDInsight](./media/migrate-storm-to-spark/new-spark-deployment.png)

1. **Arrêtez de consommer sur l’ancien cluster Storm.** Dans le cluster Storm existant, arrêtez de consommer des données de la source de données de streaming et attendez que les données aient été écrites sur le récepteur cible.

    > [!div class="mx-imgBorder"]
    > ![arrêtez de consommer sur le cluster actif](./media/migrate-storm-to-spark/stop-consuming-current-cluster.png)

1. **Commencez à consommer sur le nouveau cluster Spark.** Commencez le streaming de données à partir d’un cluster Spark HDInsight 4.0 nouvellement déployé. À ce stade, le processus est pris en charge en consommant à partir de l’offset Kafka le plus récent.

    > [!div class="mx-imgBorder"]
    > ![commencez à consommer sur le nouveau cluster](./media/migrate-storm-to-spark/start-consuming-new-cluster.png)

1. **Supprimez l’ancien cluster en fonction des besoins.** Une fois que le basculement est terminé et que tout fonctionne correctement, supprimez l’ancien cluster Storm HDInsight 3.6 en fonction des besoins.

    > [!div class="mx-imgBorder"]
    > ![supprimez les anciens clusters HDInsight en fonction des besoins](./media/migrate-storm-to-spark/remove-old-clusters1.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Storm, Spark Streaming et Spark Structured Streaming, consultez les documents suivants :

* [Vue d’ensemble d’Apache Spark Streaming](../spark/apache-spark-streaming-overview.md)
* [Présentation d’Apache Spark Structured Streaming](../spark/apache-spark-structured-streaming-overview.md)
