---
title: Exécuter des exemples MapReduce Apache Hadoop dans HDInsight - Azure
description: Bien démarrer avec des exemples MapReduce dans des fichiers jar inclus dans HDInsight. Utilisez le protocole SSH pour vous connecter au cluster, puis utilisez la commande Hadoop pour exécuter des exemples de travaux.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 12/12/2019
ms.openlocfilehash: cc0918f3d9739a214e682d7faa460b6cc519cb0b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86207840"
---
# <a name="run-the-mapreduce-examples-included-in-hdinsight"></a>Exécuter les exemples MapReduce inclus dans HDInsight

[!INCLUDE [samples-selector](../../../includes/hdinsight-run-samples-selector.md)]

Découvrez comment exécuter les exemples MapReduce inclus à Apache Hadoop dans HDInsight.

## <a name="prerequisites"></a>Conditions préalables requises

* Un cluster Apache Hadoop sur HDInsight. Consultez [Bien démarrer avec HDInsight sur Linux](./apache-hadoop-linux-tutorial-get-started.md).

* Un client SSH. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="the-mapreduce-examples"></a>Exemples MapReduce

Les exemples se trouvent sur le cluster HDInsight dans `/usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar`. Le code source de ces exemples est inclus dans le cluster HDInsight dans `/usr/hdp/current/hadoop-client/src/hadoop-mapreduce-project/hadoop-mapreduce-examples`.

Les exemples suivants sont contenus dans cette archive :

|Exemple |Description |
|---|---|
|aggregatewordcount|Compte les Mots dans les fichiers d’entrée.|
|aggregatewordhist|Calcule l’histogramme des Mots dans les fichiers d’entrée.|
|bbp|Utilise Bailey-Borwein-Plouffe pour calculer le nombre exact de chiffres de Pi.|
|dbcount|Compte les journaux d’activité de consultation de pages stockés dans une base de données.|
|distbbp|Utilise une formule de type BBP pour calculer le nombre exact de bits de Pi.|
|grep|Compte les correspondances regex dans l’entrée.|
|join|Effectue une jointure sur des jeux de données triés, à partition égale.|
|multifilewc|Compte les Mots de plusieurs fichiers.|
|pentomino|Programme de disposition de vignettes permettant de trouver des solutions aux problèmes de pentomino.|
|pi|Estime la valeur de Pi à l’aide de la méthode quasi Monte-Carlo.|
|randomtextwriter|Écrit 10 Go de données textuelles aléatoires par nœud.|
|randomwriter|Écrit 10 Go de données aléatoires par nœud.|
|secondarysort|Définit un tri secondaire lors de la phase de réduction.|
|sort|Trie les données écrites par l’enregistreur aléatoire.|
|sudoku|Solveur de sudoku.|
|teragen|Générateur de données pour le programme terasort.|
|terasort|Exécute le terasort.|
|teravalidate|Vérification des résultats du programme terasort.|
|wordcount|Compte les Mots dans les fichiers d’entrée.|
|wordmean|Compte la longueur Moyenne des Mots dans les fichiers d’entrée.|
|wordmedian|Compte la longueur médiane des Mots dans les fichiers d’entrée.|
|wordstandarddeviation|Compte l’écart type de la longueur des Mots dans les fichiers d’entrée.|

## <a name="run-the-wordcount-example"></a>Exécuter l’exemple wordcount

1. Connectez-vous à HDInsight à l’aide de SSH. Remplacez `CLUSTER` par le nom de votre cluster, puis entrez la commande suivante :

    ```cmd
    ssh sshuser@CLUSTER-ssh.azurehdinsight.net
    ```

2. Dans la session SSH, utilisez la commande suivante pour répertorier les exemples :

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar
    ```

    Cette commande génère la liste des exemples de la section précédente de ce document.

3. Utilisez la commande suivante pour obtenir de l’aide sur un exemple spécifique. Dans ce cas, l’exemple **wordcount** :

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount
    ```

    Le message suivant s’affiche :

    ```output
    Usage: wordcount <in> [<in>...] <out>
    ```

    Ce message indique que vous pouvez fournir plusieurs chemins d’accès d’entrée pour les documents sources. Le chemin d’accès final correspond à l’emplacement de stockage de la sortie (nombre de mots dans les documents sources).

4. Pour compter tous les mots figurant dans les carnets de Léonard de Vinci qui sont fournis comme exemples de données avec votre cluster, utilisez les éléments suivants :

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount /example/data/gutenberg/davinci.txt /example/data/davinciwordcount
    ```

    L’entrée de ce travail est lue à partir de `/example/data/gutenberg/davinci.txt`. La sortie de cet exemple est stockée dans `/example/data/davinciwordcount`. Les deux chemins d’accès sont situés sur le stockage par défaut du cluster, et non du système de fichiers local.

   > [!NOTE]  
   > Comme indiqué dans l’aide de l’exemple wordcount, vous pouvez également spécifier plusieurs fichiers d’entrée. Par exemple, `hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount /example/data/gutenberg/davinci.txt /example/data/gutenberg/ulysses.txt /example/data/twowordcount` compte les mots figurant dans les fichiers davinci.txt et ulysses.txt.

5. Une fois la tâche terminée, utilisez la commande suivante pour afficher le résultat généré :

    ```bash
    hdfs dfs -cat /example/data/davinciwordcount/*
    ```

    Cette commande concatène tous les fichiers de sortie générés par le travail. Elle affiche la sortie dans la console. Le résultat ressemble au texte suivant :

    ```output
    zum     1
    zur     1
    zwanzig 1
    zweite  1
    ```

    Chaque ligne représente un mot et le nombre d’occurrences dans les données d’entrée.

## <a name="the-sudoku-example"></a>L’exemple de Sudoku

[Sudoku](https://en.wikipedia.org/wiki/Sudoku) est un puzzle logique composé de 3 x 3 grilles. Certaines cellules de la grille comportent des numéros, tandis que d’autres sont vides, l’objectif consiste à résoudre les cellules vides. Le lien précédent comporte davantage d’informations sur le casse-tête, mais l’objectif de cet exemple consiste à résoudre les cellules vides. Par conséquent, notre entrée doit être un fichier au format suivant :

* Neuf lignes de neuf colonnes
* Chaque colonne peut contenir un nombre ou `?` (qui indique une cellule vide)
* Les cellules sont séparées par un espace

La façon de construire des puzzles Sudoku est particulière dans la mesure où vous ne pouvez pas répéter un nombre dans une colonne ou une ligne. Il existe un exemple sur le cluster HDInsight qui est construit correctement. Il se trouve dans `/usr/hdp/*/hadoop/src/hadoop-mapreduce-project/hadoop-mapreduce-examples/src/main/java/org/apache/hadoop/examples/dancing/puzzle1.dta` et contient le texte suivant :

```output
8 5 ? 3 9 ? ? ? ?
? ? 2 ? ? ? ? ? ?
? ? 6 ? 1 ? ? ? 2
? ? 4 ? ? 3 ? 5 9
? ? 8 9 ? 1 4 ? ?
3 2 ? 4 ? ? 8 ? ?
9 ? ? ? 8 ? 5 ? ?
? ? ? ? ? ? 2 ? ?
? ? ? ? 4 5 ? 7 8
```

Pour exécuter cet exemple de problème dans l’exemple de Sudoku, utilisez la commande suivante :

```bash
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar sudoku /usr/hdp/*/hadoop/src/hadoop-mapreduce-project/hadoop-mapreduce-examples/src/main/java/org/apache/hadoop/examples/dancing/puzzle1.dta
```

Les résultats sont les suivants :

```output
8 5 1 3 9 2 6 4 7
4 3 2 6 7 8 1 9 5
7 9 6 5 1 4 3 8 2
6 1 4 8 2 3 7 5 9
5 7 8 9 6 1 4 2 3
3 2 9 4 5 7 8 1 6
9 4 7 2 8 6 5 3 1
1 8 5 7 3 9 2 6 4
2 6 3 1 4 5 9 7 8
```

## <a name="pi--example"></a>Exemple de Pi (π)

L’exemple de Pi utilise une méthode statistique (quasi-Monte-Carlo) pour estimer la valeur de Pi. Les points sont placés de manière aléatoire dans un carré unitaire. Le carré contient également un cercle. La probabilité que les points tombent dans le cercle est égale à l’aire du cercle, pi/4. La valeur de pi peut être estimée à partir de la valeur de 4R. R est le rapport entre le nombre de points situés à l’intérieur du cercle et le nombre total de points situés à l’intérieur du carré. Plus l'échantillon de points est grand, plus l'estimation est précise.

Utilisez la commande suivante pour exécuter cet exemple : Cette commande utilise 16 mappages, avec pour chacun 10 000 000 exemples, pour estimer la valeur de Pi :

```bash
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar pi 16 10000000
```

La valeur retournée par cette commande doit être semblable à **3,14159155000000000000**. Pour référence, voici les 10 chiffres de Pi après la virgule : 3,1415926535.

## <a name="10-gb-graysort-example"></a>Exemple GraySort de 10 Go

GraySort est un tri de benchmark. La mesure est le taux de tri (To/minute) obtenu lors du tri de très grandes quantités de données, en général au minimum 100 To.

Cet exemple utilise seulement 10 Go de données afin de pouvoir être exécuté relativement rapidement. Il utilise les applications MapReduce développées par Owen O'Malley et Arun Murthy. Ces applications ont remporté en 2009 le benchmark du tri de téraoctets (« Daytona ») annuel universel général avec un taux de 0,578 To/min (100 To en 173 minutes). Pour plus d'informations à ce sujet et sur d'autres benchmarks de tri, consultez le site [Sort Benchmark](https://sortbenchmark.org/).

Cet exemple utilise trois ensembles de programmes MapReduce :

* **TeraGen**: programme MapReduce qui génère des lignes de données à trier

* **TeraSort**: échantillonne les données d'entrée et utilise MapReduce pour trier les données en une commande totale.

    TeraSort est un tri MapReduce standard, à l’exception d’un partitionneur personnalisé. Le partitionneur utilise une liste triée des clés N-1 échantillonnée qui définissent la plage de clés pour chaque réduction. Plus particulièrement, toutes les clés semblables à cet échantillon [i-1] <= key < sample[i] sont envoyées pour réduire i. Le partitionneur garantit que les sorties de réduction i sont toutes inférieures aux sorties de réduction i+1.

* **TeraValidate**: programme MapReduce qui valide le tri global de la sortie

    Il crée un mappage par fichier dans le répertoire de sortie et chaque mappage assure que chaque clé est inférieure ou égale à la précédente. La fonction de mappage génère des enregistrements de la première et de la dernière clés de chaque fichier. La fonction de réduction veille à ce que la première clé du fichier i soit supérieure à la dernière clé du fichier i-1. Un problème est signalé comme une sortie de la phase de réduction avec les clés dans le désordre.

Utilisez les étapes suivantes pour générer des données, trier, puis valider la sortie :

1. Générez 10 Go de données, qui sont stockées dans le stockage par défaut du cluster HDInsight à l’emplacement `/example/data/10GB-sort-input` :

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teragen -Dmapred.map.tasks=50 100000000 /example/data/10GB-sort-input
    ```

    `-Dmapred.map.tasks` indique à Hadoop le nombre de tâches de mappage à utiliser pour cette tâche. Les deux derniers paramètres demandent au travail de créer 10 Go de données et de les stocker sous `/example/data/10GB-sort-input`.

2. Exécutez la commande suivante pour trier les données :

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar terasort -Dmapred.map.tasks=50 -Dmapred.reduce.tasks=25 /example/data/10GB-sort-input /example/data/10GB-sort-output
    ```

    `-Dmapred.reduce.tasks` indique à Hadoop le nombre de tâches de réduction à utiliser pour cette tâche. Les deux derniers paramètres sont simplement les emplacements d’entrée et de sortie des données.

3. Pour valider les données générées par le tri, utilisez les éléments suivants :

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teravalidate -Dmapred.map.tasks=50 -Dmapred.reduce.tasks=25 /example/data/10GB-sort-output /example/data/10GB-sort-validate
    ```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à exécuter les exemples inclus avec les clusters HDInsight sous Linux. Pour des didacticiels sur l’utilisation de Pig, Hive et MapReduce avec HDInsight, consultez les rubriques suivantes :

* [Utiliser Apache Hive avec Apache Hadoop sur HDInsight](hdinsight-use-hive.md)
* [Utiliser MapReduce avec Apache Hadoop sur HDInsight](hdinsight-use-mapreduce.md)
