---
title: Optimisation des performances pour les clusters Kafka HDInsight Apache
description: Fournit une vue d’ensemble des techniques pour optimiser les charges de travail Apache Kafka sur Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 12/19/2019
ms.openlocfilehash: 752068af531c4a0ecc832d266f88105c14452ecb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75494912"
---
# <a name="performance-optimization-for-apache-kafka-hdinsight-clusters"></a>Optimisation des performances pour les clusters Kafka HDInsight Apache

Cet article contient des suggestions pour optimiser les performances de vos charges de travail Apache Kafka dans HDInsight. L’accent est mis sur l’optimisation de la configuration du producteur et du répartiteur. Il existe différentes façons de mesurer les performances, et les optimisations que vous appliquez dépendent de vos besoins.

## <a name="architecture-overview"></a>Vue d’ensemble de l’architecture

Les rubriques Kafka sont utilisées pour organiser des enregistrements. Les enregistrements sont produits par des producteurs et utilisés par des consommateurs. Les producteurs envoient des enregistrements aux répartiteurs Kafka, qui stockent alors les données. Chaque nœud Worker dans votre cluster HDInsight est un répartiteur Kafka.

Les rubriques partitionnent les enregistrements entre les répartiteurs. Au cours de l’utilisation des enregistrements, il est possible d’utiliser jusqu’à un consommateur par partition pour effectuer un traitement parallèle des données.

La réplication est utilisée pour dupliquer des partitions entre les nœuds. Ceci protège contre les pannes des nœuds (du répartiteur). Une seule partition parmi le groupe de réplicas est désignée comme partition « leader ». Le trafic de producteur est acheminé vers le leader de chaque nœud, en utilisant l’état géré par ZooKeeper.

## <a name="identify-your-scenario"></a>Identifier votre scénario

Les performances d’Apache Kafka ont deux aspects principaux : le débit et la latence. Le débit est vitesse de traitement maximale des données de votre application. Un débit plus élevé est généralement préférable. La latence est le temps nécessaire pour que les données soient stockées ou récupérées. Une latence plus faible est généralement préférable. Trouver le juste équilibre entre le débit, la latence et le coût de l’infrastructure de l’application peut être difficile. Vos exigences en matière de performances vont probablement correspondre à une des trois situations courantes suivantes, selon que vous avez besoin d’un débit élevé, d’une latence faible ou des deux :

* Débit élevé, latence faible. Ce scénario nécessite à la fois un débit élevé et une latence faible (environ 100 millisecondes). La supervision de la disponibilité des services est un exemple de ce type d’application.
* Débit élevé, latence élevée. Ce scénario nécessite un débit élevé (environ 1,5 Gbits/s), mais peut tolérer une latence plus élevée (< 250 ms). L’ingestion de données de télémétrie pour des processus en quasi temps réel, comme des applications de sécurité et de détection des intrusions, est un exemple de ce type d’application.
* Débit faible, latence faible. Ce scénario nécessite une latence faible (< 10 ms) pour le traitement en temps réel, mais peut tolérer un débit inférieur. Les vérifications de l’orthographe et de la grammaire en ligne sont un exemple de ce type d’application.

## <a name="producer-configurations"></a>Configurations du producteur

Les sections suivantes sont consacrées à certaines des propriétés de configuration les plus importantes pour optimiser les performances de vos producteurs Kafka. Pour obtenir une explication détaillée de toutes les propriétés de configuration, consultez la [documentation Apache Kafka sur les configurations de producteur](https://kafka.apache.org/documentation/#producerconfigs).

### <a name="batch-size"></a>Taille du lot

Les producteurs Apache Kafka assemblent des groupes de messages (appelés lots) qui sont envoyés en tant qu’unité à stocker dans une même partition de stockage. La taille de lot représente le nombre d’octets qui doivent être présents avant que ce groupe soit transmis. Augmenter la valeur du paramètre `batch.size` peut augmenter le débit, car il réduit la surcharge du traitement due aux demandes du réseau et des E/S. Avec une charge faible, une taille de lot augmentée peut augmenter la latence des envois de Kafka, car le producteur attend qu’un lot soit prêt. Avec une charge importante, il est recommandé d’augmenter la taille de lot pour améliorer le débit et la latence.

### <a name="producer-required-acknowledgments"></a>Accusés de réception requis par le producteur

La configuration des accusés de réception (`acks`) requis par le producteur détermine le nombre d’accusés de réception requis par la partition « leader » avant qu’une demande d’écriture soit considérée comme terminée. Ce paramètre affecte la fiabilité des données et il peut prendre les valeurs suivantes : `0`, `1` ou `-1`. La valeur `-1` signifie qu’un accusé de réception doit être reçu de tous les réplicas avant que l’écriture soit terminée. Définir cette valeur sur `acks = -1` offre de meilleures garanties contre la perte de données, mais elle résulte également en une latence plus élevée et un débit inférieur. Si les spécifications de votre application demandent un débit plus élevé, essayez en définissant `acks = 0` ou `acks = 1`. Gardez à l’esprit que ne pas accuser réception pour tous les réplicas peut réduire la fiabilité des données.

### <a name="compression"></a>Compression

Un producteur Kafka peut être configuré de façon à compresser les messages avant de les envoyer aux répartiteurs. Le paramètre `compression.type` spécifie le codec de compression à utiliser. Les codecs de compression pris en charge sont « gzip », « snappy » et « lz4 ». La compression est utile et doit être envisagée si la capacité du disque est limitée.

Parmi les deux codecs de compression couramment utilisés, `gzip` et `snappy`, `gzip` a un taux de compression plus élevé, ce qui entraîne une utilisation inférieure du disque, au prix d’une charge plus élevée pour le processeur. Le codec `snappy` offre une compression inférieure, avec moins de charge pour le processeur. Vous pouvez décider du codec à utiliser en fonction des limitations du disque du répartiteur ou du processeur du producteur. `gzip` peut compresser les données à une vitesse cinq fois supérieure à celle de `snappy`.

L’utilisation de la compression des données va augmenter le nombre d’enregistrements qui peuvent être stockés sur un disque. Elle peut également augmenter la charge du processeur dans les cas où les formats de compression utilisés par le producteur et par le répartiteur ne correspondent pas. Les données doivent en effet être compressées avant l’envoi et décompressées avant leur traitement.

## <a name="broker-settings"></a>Paramètres du répartiteur

Les sections suivantes sont consacrées à certaines des propriétés les plus importantes pour optimiser les performances de vos répartiteurs Kafka. Pour obtenir une explication détaillée de tous les paramètres des répartiteurs, consultez la [documentation Apache Kafka sur les configurations de répartiteur](https://kafka.apache.org/documentation/#producerconfigs).

### <a name="number-of-disks"></a>Nombre de disques

Les disques de stockage sont sujets à des limitations des IOPS (opérations d’entrée/sortie par seconde) et des octets lus/écrits par seconde. Lors de la création de partitions, Kafka stocke chaque nouvelle partition sur le disque ayant le moins de partitions existantes, de façon à les équilibrer entre les disques disponibles. En dépit de cette stratégie de stockage, lors du traitement de centaines de partitions de réplica sur chaque disque, Kafka peut facilement saturer le débit des disques disponibles. Le compromis se fait ici entre le débit et de coût. Si votre application nécessite un débit supérieur, créer un cluster avec plusieurs disques managés par répartiteur. HDInsight ne prend actuellement pas en charge l’ajout de disques managés à un cluster en cours d’exécution. Pour plus d’informations sur la façon de configurer le nombre de disques managés, consultez [Configurer le stockage et la scalabilité pour Apache Kafka sur HDInsight](apache-kafka-scalability.md). Vous devez bien comprendre les implications en termes de coût de l’augmentation de l’espace de stockage pour les nœuds de votre cluster.

### <a name="number-of-topics-and-partitions"></a>Nombre de rubriques et de partitions

Les producteurs Kafka écrivent dans des rubriques. Les consommateurs Kafka lisent dans des rubriques. Une rubrique est associée à un journal, qui est une structure de données sur le disque. Kafka ajoute les enregistrements provenant d’un ou plusieurs producteurs à la fin du journal d’une rubrique. Un journal de rubrique se compose de nombreuses partitions qui sont réparties en plusieurs fichiers. Ces fichiers sont à leur tour répartis entre plusieurs nœuds de cluster Kafka. Les consommateurs lisent dans des rubriques Kafka à leur rythme, et ils peuvent choisir leur position (décalage) dans le journal de la rubrique.

Chaque partition Kafka est un fichier journal sur le système, et les threads des producteurs peuvent écrire simultanément dans plusieurs journaux. De même, comme chaque thread de consommateur lit les messages à partir d’une seule partition, la consommation à partir de plusieurs partitions est également gérée en parallèle.

L’augmentation de la densité de partitions (le nombre de partitions par répartiteur) ajoute une charge liée aux opérations de métadonnées et aux demandes/réponses par partition entre la partition « leader » et celles qui la suivent. Même en l’absence de données en transition, les réplicas de partition continuent d’extraire des données des « leaders », ce qui aboutit à un traitement supplémentaire pour envoyer et recevoir des demandes sur le réseau.

Pour les clusters Apache Kafka 1.1 et ultérieurs dans HDInsight, nous vous recommandons d’avoir un maximum de 1 000 partitions par répartiteur, y compris les réplicas. L’augmentation du nombre de partitions par répartiteur diminue le débit et peut également entraîner l’indisponibilité de rubriques. Pour plus d’informations sur la prise en charge des partitions dans Kafka, consultez [le billet de blog Apache Kafka officiel sur l’augmentation du nombre de partitions prises en charge dans la version 1.1.0](https://blogs.apache.org/kafka/entry/apache-kafka-supports-more-partitions). Pour plus d’informations sur la modification des rubriques, consultez [Apache Kafka : modification des rubriques](https://kafka.apache.org/documentation/#basic_ops_modify_topic).

### <a name="number-of-replicas"></a>Nombre de réplicas

Un facteur de réplication plus élevé entraîne des demandes supplémentaires entre la partition « leader » et celles qui la suivent. Par conséquent, un facteur de réplication supérieur consomme plus de disque et de processeur pour gérer les demandes supplémentaires, en augmentant la latence des écritures et en diminuant le débit.

Nous vous recommandons d’utiliser au moins une réplication 3x pour Kafka dans Azure HDInsight. La plupart des régions Azure ont trois domaines d’erreur, mais dans les régions avec seulement deux domaines d’erreur, les utilisateurs doivent utiliser une réplication 4x.

Pour plus d’informations sur la réplication, consultez [Apache Kafka : réplication](https://kafka.apache.org/documentation/#replication) et [Apache Kafka : augmentation du facteur de réplication](https://kafka.apache.org/documentation/#basic_ops_increase_replication_factor).

## <a name="next-steps"></a>Étapes suivantes

* [Traitement de billions d’événements par jour avec Apache Kafka sur Azure](https://azure.microsoft.com/blog/processing-trillions-of-events-per-day-with-apache-kafka-on-azure/)
* [Présentation d’Apache Kafka sur HDInsight](apache-kafka-introduction.md)
