---
title: Présentation d’Apache Hadoop et de MapReduce - Azure HDInsight
description: Présentation de HDInsight ainsi que de la pile de technologies et des composants Apache Hadoop.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: overview
ms.custom: hdinsightactive,hdiseo17may2017,mvc,seodec18
ms.date: 02/27/2020
ms.openlocfilehash: ef1914499765beff9913f9735cf55736135f9d96
ms.sourcegitcommit: 693df7d78dfd5393a28bf1508e3e7487e2132293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92899633"
---
# <a name="what-is-apache-hadoop-in-azure-hdinsight"></a>Qu’est-ce qu’Apache Hadoop dans Azure HDInsight ?

[Apache Hadoop](https://hadoop.apache.org/) était l’infrastructure open source d’origine de traitement et d’analyse distribués des jeux de données volumineuses sur des clusters. L’écosystème Hadoop comprend des logiciels et des utilitaires associés, notamment Apache Hive, Apache HBase, Spark, Kafka et bien d’autres encore.

Azure HDInsight est un service cloud d’analytique complètement managé, exhaustif et open source pour les entreprises. Le type de cluster Apache Hadoop dans Azure HDInsight vous permet d’utiliser [Apache Hadoop Distributed File System (HDFS)](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html), la gestion des ressources [Apache Hadoop YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html) ainsi qu’un modèle de programmation [MapReduce](https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html) simple pour traiter et analyser des lots de données en parallèle.  Les clusters Hadoop dans HDInsight sont compatibles avec [Stockage Blob Azure](../../storage/common/storage-introduction.md), [Azure Data Lake Storage Gen1](../../data-lake-store/data-lake-store-overview.md) et [Azure Data Lake Storage Gen2](../../storage/blobs/data-lake-storage-introduction.md).

Pour voir les piles de composants de technologie Hadoop disponibles sur HDInsight, consultez [Composants et versions disponibles avec HDInsight](../hdinsight-component-versioning.md). Pour plus d’informations sur Hadoop dans HDInsight, consultez la rubrique [Page de fonctionnalités Azure pour HDInsight](https://azure.microsoft.com/services/hdinsight/).

## <a name="what-is-mapreduce"></a>Présentation de MapReduce

[Apache Hadoop MapReduce](https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html) est une infrastructure logicielle qui permet d’écrire des tâches traitant d’importantes quantités de données. Les données d’entrée sont fractionnées en blocs indépendants. Chaque bloc est traité en parallèle sur les nœuds de votre cluster. Une tâche MapReduce se compose de deux fonctions :

* **Mappeur** : il consomme les données d'entrée, les analyse (généralement avec les opérations de tri et de filtre) et émet des tuples (paires clé-valeur)

* **Raccord de réduction** : il consomme les tuples émis par le Mappeur et effectue une opération de synthèse qui crée un résultat plus petit, combiné à partir des données du Mappeur

Un exemple de tâche MapReduce de comptage de mots de base est illustré dans le diagramme suivant :

 ![HDI.WordCountDiagram](./media/apache-hadoop-introduction/hdi-word-count-diagram.gif)

Le résultat de ce travail indique le nombre d’occurrences de chaque mot dans le texte.

* Le mappeur prend chaque ligne du texte saisi en tant qu'entrée, et la divise en mots. Il émet une paire clé/valeur pour chaque occurrence de mot ou si le mot est suivi d’un 1. Le résultat est trié avant d'être envoyé au raccord de réduction.
* Ce dernier calcule la somme du compte de mots, puis émet une seule paire clé/valeur contenant le mot défini, suivi par la somme de ses occurrences.

MapReduce peut être implémenté dans plusieurs langues. Java est l'implémentation la plus courante. Ce langage est utilisé à titre d’exemple dans ce document.

## <a name="development-languages"></a>Langues de développement

Les langages ou frameworks basés sur Java et la Machine virtuelle Java peuvent être directement exécutés comme une [tâche MapReduce](..//hadoop/submit-apache-hadoop-jobs-programmatically.md). L’exemple utilisé dans ce document est une application Java MapReduce. Les langages autres que Java, comme C#, Python ou des exécutables autonomes, doivent utiliser la **diffusion en continu Hadoop**.

La diffusion en continu Hadoop communique avec le mappeur et le raccord de réduction via STDIN et STDOUT. Le mappeur et le raccord de réduction lisent les données ligne par ligne depuis STDIN et écrivent la sortie dans STDOUT. Chaque ligne lue ou émise par le mappeur et le raccord de réduction doit être au format d’une paire clé / valeur, délimitée par un caractère de tabulation :

`[key]\t[value]`

Pour plus d’informations, consultez [Diffusion en continu Hadoop](https://hadoop.apache.org/docs/current/hadoop-streaming/HadoopStreaming.html).

Pour obtenir des exemples d’utilisation de diffusion en continu Hadoop avec HDInsight, consultez le document suivant :

* [Développement de tâches MapReduce C#](apache-hadoop-dotnet-csharp-mapreduce-streaming.md)

## <a name="where-do-i-start"></a>Par où commencer ?

* [Démarrage rapide : créer un cluster Apache Hadoop dans Azure HDInsight à l’aide du portail Azure](../hadoop/apache-hadoop-linux-create-cluster-get-started-portal.md)
* [Tutoriel : Envoyer des tâches Apache Hadoop dans HDInsight](../hadoop/submit-apache-hadoop-jobs-programmatically.md)
* [Développer des programmes MapReduce Java pour Apache Hadoop sur HDInsight](../hadoop/apache-hadoop-develop-deploy-java-mapreduce-linux.md)
* [Utiliser Apache Hive comme un outil d’extraction, de transformation et de chargement (ETL)](../hadoop/apache-hadoop-using-apache-hive-as-an-etl-tool.md)
* [Extraire, transformer et charger (ETL) à l’échelle](../hadoop/apache-hadoop-etl-at-scale.md)
* [Rendre un pipeline d’analytique de données opérationnel](../hdinsight-operationalize-data-pipeline.md)

## <a name="next-steps"></a>Étapes suivantes

* [Créer un cluster Apache Hadoop dans HDInsight à l’aide du portail](../hadoop/apache-hadoop-linux-create-cluster-get-started-portal.md)
* [Créer un cluster Apache Hadoop dans HDInsight à l’aide d’un modèle ARM](../hadoop/apache-hadoop-linux-tutorial-get-started.md)