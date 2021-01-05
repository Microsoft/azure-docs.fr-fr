---
title: Utiliser C# avec MapReduce sur Hadoop dans HDInsight - Azure
description: Découvrez comment utiliser C# pour créer des solutions MapReduce avec Apache Hadoop dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, seoapr2020, devx-track-csharp
ms.date: 04/28/2020
ms.openlocfilehash: 4153b612b4b15883f28517d806acf6b340a94e49
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92543047"
---
# <a name="use-c-with-mapreduce-streaming-on-apache-hadoop-in-hdinsight"></a>Utiliser C# avec streaming MapReduce sur Apache Hadoop dans HDInsight

Découvrez comment utiliser C# pour créer une solution MapReduce dans HDInsight.

Le streaming Apache Hadoop vous permet d’exécuter des tâches MapReduce à l’aide d’un script ou d’un exécutable. Ici, .NET est utilisé pour implémenter le mappeur et le raccord de réduction pour une solution de comptage de mots.

## <a name="net-on-hdinsight"></a>.NET sur HDInsight

Les clusters HDInsight utilisent [Mono (https://mono-project.com)](https://mono-project.com) pour exécuter les applications .NET. La version 4.2.1 de Mono est incluse dans la version 3.6 de HDInsight. Pour plus d’informations sur la version de Mono incluse dans HDInsight, consultez [Composants Apache Hadoop disponibles avec différentes versions de HDInsight](../hdinsight-component-versioning.md#apache-components-available-with-different-hdinsight-versions).

Pour plus d’informations sur la compatibilité Mono avec les versions de .NET Framework, consultez [Compatibilité Mono](https://www.mono-project.com/docs/about-mono/compatibility/).

## <a name="how-hadoop-streaming-works"></a>Fonctionnement de la diffusion en continu Hadoop

Le processus de base utilisé pour la diffusion en continu dans ce document est la suivante :

1. Hadoop transmet des données vers le mappeur ( *mapper.exe* dans cet exemple) sur STDIN.
2. Le mappeur traite les données et émet des paires clé/valeur séparées par des tabulations sur STDOUT.
3. La sortie est lue par Hadoop, puis transmise au raccord de réduction ( *reducer.exe* dans cet exemple) sur STDIN.
4. Le raccord de réduction lit les paires clé/valeur séparées par des tabulations, traite les données, puis émet le résultat sous forme de paires clé/valeur séparées par des tabulations sur STDOUT.
5. La sortie est lue par Hadoop et écrite dans le répertoire de sortie.

Pour plus d’informations sur la diffusion en continu, consultez l’article [Diffusion en continu Hadoop](https://hadoop.apache.org/docs/r2.7.1/hadoop-streaming/HadoopStreaming.html).

## <a name="prerequisites"></a>Prérequis

* Visual Studio.

* Des connaissances en écriture et en génération de code C# qui cible .NET Framework 4.5.

* Permet de charger des fichiers .exe dans le cluster. Les étapes de ce document utilisent Data Lake Tools pour Visual Studio pour télécharger les fichiers vers le stockage principal pour le cluster.

* Si vous utilisez PowerShell, vous aurez besoin du [module Az](/powershell/azure/).

* Un cluster Apache Hadoop sur HDInsight. Consultez [Bien démarrer avec HDInsight sur Linux](../hadoop/apache-hadoop-linux-tutorial-get-started.md).

* Le schéma d'URI de votre principal espace de stockage de clusters. Ce schéma serait `wasb://` pour Stockage Azure, `abfs://` pour Azure Data Lake Storage Gen2 ou `adl://` pour Azure Data Lake Storage Gen1. Si l’option de transfert sécurisé était activée pour Stockage Azure ou Data Lake Storage Gen2, l’URI serait `wasbs://` ou `abfss://`, respectivement.

## <a name="create-the-mapper"></a>Créer le mappeur

Dans Visual Studio, créez une nouvelle application console .NET Framework nommée *mapper* . Utilisez le code suivant pour l’application :

```csharp
using System;
using System.Text.RegularExpressions;

namespace mapper
{
    class Program
    {
        static void Main(string[] args)
        {
            string line;
            //Hadoop passes data to the mapper on STDIN
            while((line = Console.ReadLine()) != null)
            {
                // We only want words, so strip out punctuation, numbers, etc.
                var onlyText = Regex.Replace(line, @"\.|;|:|,|[0-9]|'", "");
                // Split at whitespace.
                var words = Regex.Matches(onlyText, @"[\w]+");
                // Loop over the words
                foreach(var word in words)
                {
                    //Emit tab-delimited key/value pairs.
                    //In this case, a word and a count of 1.
                    Console.WriteLine("{0}\t1",word);
                }
            }
        }
    }
}
```

Après avoir créé l’application, générez-la pour produire le fichier */bin/Debug/mapper.exe* dans le répertoire du projet.

## <a name="create-the-reducer"></a>Créer le raccord de réduction

Dans Visual Studio, créez une nouvelle application console .NET Framework nommée *reducer* . Utilisez le code suivant pour l’application :

```csharp
using System;
using System.Collections.Generic;

namespace reducer
{
    class Program
    {
        static void Main(string[] args)
        {
            //Dictionary for holding a count of words
            Dictionary<string, int> words = new Dictionary<string, int>();

            string line;
            //Read from STDIN
            while ((line = Console.ReadLine()) != null)
            {
                // Data from Hadoop is tab-delimited key/value pairs
                var sArr = line.Split('\t');
                // Get the word
                string word = sArr[0];
                // Get the count
                int count = Convert.ToInt32(sArr[1]);

                //Do we already have a count for the word?
                if(words.ContainsKey(word))
                {
                    //If so, increment the count
                    words[word] += count;
                } else
                {
                    //Add the key to the collection
                    words.Add(word, count);
                }
            }
            //Finally, emit each word and count
            foreach (var word in words)
            {
                //Emit tab-delimited key/value pairs.
                //In this case, a word and a count of 1.
                Console.WriteLine("{0}\t{1}", word.Key, word.Value);
            }
        }
    }
}
```

Après avoir créé l’application, générez-la pour produire le fichier */bin/Debug/reducer.exe* dans le répertoire du projet.

## <a name="upload-to-storage"></a>Téléchargement vers le stockage

Ensuite, vous devez charger les applications *mapper* et *reducer* sur le stockage HDInsight.

1. Dans Visual Studio, sélectionnez **Affichage** > **Explorateur de serveurs** .

1. Cliquez avec le bouton droit sur **Azure** , sélectionnez **Se connecter à un abonnement Microsoft Azure...** , puis terminez le processus de connexion.

1. Développez le cluster HDInsight sur lequel vous souhaitez déployer cette application. Une entrée avec le texte **(compte de stockage par défaut)** est répertoriée.

   ![Compte de stockage, cluster HDInsight, explorateur de serveurs, Visual Studio](./media/apache-hadoop-dotnet-csharp-mapreduce-streaming/hdinsight-storage-account.png)

   * Si l’entrée **(compte de stockage par défaut)** peut être développée, vous utilisez un **compte de stockage Azure** en tant que stockage par défaut pour le cluster. Pour afficher les fichiers sur le stockage par défaut pour le cluster, développez l’entrée, puis double-cliquez sur le **(conteneur par défaut)** .

   * Si l’entrée **(compte de stockage par défaut)** ne peut pas être développée, vous utilisez **Azure Data Lake Storage** en tant que stockage par défaut pour le cluster. Pour afficher les fichiers sur le stockage par défaut pour le cluster, double-cliquez sur l’entrée **(compte de stockage par défaut)** .

1. Pour charger les fichiers .exe, appliquez l’une des méthodes suivantes :

    * Si vous utilisez un **compte de stockage Azure** , sélectionnez l’icône **Télécharger un objet Blob** .

        ![Icône de chargement de HDInsight pour le mappeur, Visual Studio](./media/apache-hadoop-dotnet-csharp-mapreduce-streaming/hdinsight-upload-icon.png)

        Dans la boîte de dialogue **Télécharger un nouveau fichier** , sous **Nom de fichier** , sélectionnez **Parcourir** . Dans la boîte de dialogue **Télécharger un objet Blob** , accédez au dossier *bin\debug* pour le projet *mapper* , puis sélectionnez le fichier *mapper.exe* . Enfin, sélectionnez **Ouvrir** , puis **OK** pour terminer le téléchargement.

    * Pour **Azure Data Lake Storage** , cliquez avec le bouton droit sur une zone vide de la liste des fichiers, puis sélectionnez **Charger** . Enfin, sélectionnez le fichier *mapper.exe* et cliquez sur **Ouvrir** .

    Une fois que le chargement de *mapper.exe* est terminé, répétez le processus de chargement pour le fichier *reducer.exe* .

## <a name="run-a-job-using-an-ssh-session"></a>Exécuter un travail : Utilisation d’une session SSH

La procédure suivante décrit comment exécuter un travail MapReduce à l’aide d’une session SSH :

1. Utilisez la [commande ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Exécutez une des commandes suivantes pour démarrer la tâche MapReduce :

   * Si le stockage par défaut est **Stockage Azure** :

        ```bash
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar \
            -files wasbs:///mapper.exe,wasbs:///reducer.exe \
            -mapper mapper.exe \
            -reducer reducer.exe \
            -input /example/data/gutenberg/davinci.txt \
            -output /example/wordcountout
        ```

    * Si le stockage par défaut est **Data Lake Storage Gen1** :

        ```bash
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar \
            -files adl:///mapper.exe,adl:///reducer.exe \
            -mapper mapper.exe \
            -reducer reducer.exe \
            -input /example/data/gutenberg/davinci.txt \
            -output /example/wordcountout
        ```

   * Si le stockage par défaut est **Data Lake Storage Gen2** :

        ```bash
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar \
            -files abfs:///mapper.exe,abfs:///reducer.exe \
            -mapper mapper.exe \
            -reducer reducer.exe \
            -input /example/data/gutenberg/davinci.txt \
            -output /example/wordcountout
        ```

   La liste suivante fournit la signification de chaque paramètre et option :

   |Paramètre | Description |
   |---|---|
   |hadoop-streaming.jar|Indique le fichier jar contenant la fonctionnalité MapReduce de streaming.|
   |-files|Indique les fichiers *mapper.exe* et *reducer.exe* pour ce travail. La déclaration de protocole `wasbs:///`, `adl:///` ou `abfs:///` devant chaque fichier correspond au chemin d’accès à la racine du stockage par défaut pour le cluster.|
   |-mapper|Indique le fichier qui implémente le mappeur.|
   |-reducer|Indique le fichier qui implémente le raccord de réduction.|
   |-input|Indique les données d’entrée.|
   |-output|Indique le répertoire de sortie.|

1. Une fois le travail MapReduce terminé, utilisez la commande suivante pour afficher les résultats :

   ```bash
   hdfs dfs -text /example/wordcountout/part-00000
   ```

   Le texte suivant est un exemple de données renvoyées par cette commande :

   ```output
   you     1128
   young   38
   younger 1
   youngest        1
   your    338
   yours   4
   yourself        34
   yourselves      3
   youth   17
   ```

## <a name="run-a-job-using-powershell"></a>Exécuter un travail : Utilisation de PowerShell

Utilisez le script PowerShell suivant pour exécuter une tâche MapReduce et télécharger les résultats.

[!code-powershell[main](../../../powershell_scripts/hdinsight/use-csharp-mapreduce/use-csharp-mapreduce.ps1?range=5-87)]

Ce script vous invite à entrer le nom et le mot de passe du compte de connexion du cluster, ainsi que le nom du cluster HDInsight. Une fois que le travail est terminé, la sortie est téléchargée sous la forme d’un fichier nommé *output.txt* . Le texte suivant constitue un exemple des données contenues dans le fichier `output.txt` :

```output
you     1128
young   38
younger 1
youngest        1
your    338
yours   4
yourself        34
yourselves      3
youth   17
```

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser MapReduce dans Apache Hadoop sur HDInsight](hdinsight-use-mapreduce.md)
* [Utiliser une fonction C# définie par l’utilisateur avec Apache Hive et Apache Pig](apache-hadoop-hive-pig-udf-dotnet-csharp.md)
* [Développer des programmes Java MapReduce](apache-hadoop-develop-deploy-java-mapreduce-linux.md)