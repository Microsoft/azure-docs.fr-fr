---
title: MapReduce et connexion SSH avec Apache Hadoop – HDInsight
description: Apprenez à utiliser le protocole SSH pour exécuter des tâches MapReduce avec Apache Hadoop sur HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/10/2020
ms.openlocfilehash: 2736d0cfe514252e36ba6d7e0d71efe09da15aae
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86076298"
---
# <a name="use-mapreduce-with-apache-hadoop-on-hdinsight-with-ssh"></a>Utilisation de MapReduce avec Apache Hadoop sur HDInsight avec SSH

[!INCLUDE [mapreduce-selector](../../../includes/hdinsight-selector-use-mapreduce.md)]

Découvrez comment soumettre des tâches MapReduce à partir d’une connexion SSH (Secure Shell) vers HDInsight.

> [!NOTE]
> Si vous connaissez déjà l’utilisation de serveurs Apache Hadoop basés sur Linux, mais pas HDInsight, consultez la rubrique [Informations sur l’utilisation de HDInsight sur Linux](../hdinsight-hadoop-linux-information.md).

## <a name="prerequisites"></a>Conditions préalables requises

Un cluster Apache Hadoop sur HDInsight. Consultez [Créer des clusters Apache Hadoop à l’aide du Portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md).

## <a name="use-hadoop-commands"></a>Utilisation de commandes Hadoop

1. Utilisez la [commande ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Une fois connecté au cluster HDInsight, utilisez la commande suivante pour lancer une tâche MapReduce :

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount /example/data/gutenberg/davinci.txt /example/data/WordCountOutput
    ```

    Cette commande démarre la classe `wordcount`, qui est contenue dans le fichier `hadoop-mapreduce-examples.jar`. Il utilise le document `/example/data/gutenberg/davinci.txt` comme entrée. La sortie est stockée dans `/example/data/WordCountOutput`.

    > [!NOTE]
    > Pour plus d’informations sur ce travail MapReduce et pour des exemples de données, consultez [Utiliser MapReduce dans Apache Hadoop sur HDInsight](hdinsight-use-mapreduce.md).

    La tâche émet des informations lors de son traitement, avant de renvoyer des informations semblables au texte suivant lorsqu’elle est terminée :

    ```output
    File Input Format Counters
    Bytes Read=1395666
    File Output Format Counters
    Bytes Written=337623
    ```

1. Une fois la tâche terminée, utilisez la commande suivante pour afficher les fichiers sortants :

    ```bash
    hdfs dfs -ls /example/data/WordCountOutput
    ```

    Cette commande affiche deux fichiers, `_SUCCESS` et `part-r-00000`. Le fichier `part-r-00000` contient le résultat pour cette tâche.

    > [!NOTE]  
    > Certaines tâches MapReduce peuvent fractionner les résultats sur plusieurs fichiers **part-r-#####** . Dans ce cas, utilisez le suffixe ##### pour indiquer l’ordre des fichiers.

1. Pour afficher la sortie, utilisez la commande suivante :

    ```bash
    hdfs dfs -cat /example/data/WordCountOutput/part-r-00000
    ```

    Cette commande affiche une liste des mots contenus dans le fichier **wasb://example/data/gutenberg/davinci.txt**, ainsi que le nombre d’occurrences de chaque mot. Le texte suivant est un exemple des données contenues dans le fichier :

    ```output
    wreathed        3
    wreathing       1
    wreaths         1
    wrecked         3
    wrenching       1
    wretched        6
    wriggling       1
    ```

## <a name="next-steps"></a>Étapes suivantes

Comme vous pouvez le voir, les commandes Hadoop fournissent un moyen facile pour exécuter des tâches MapReduce sur un cluster HDInsight avant d’afficher le résultat de la tâche. Pour plus d’informations sur d’autres méthodes de travail avec Hadoop sur HDInsight :

* [Utilisation de MapReduce dans Hadoop HDInsight](hdinsight-use-mapreduce.md)
* [Utiliser Apache Hive avec Apache Hadoop sur HDInsight](hdinsight-use-hive.md)
