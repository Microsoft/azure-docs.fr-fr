---
title: Créer une application MapReduce Java pour Apache Hadoop - Azure HDInsight
description: Découvrez comment utiliser Apache Maven pour créer une application MapReduce basée sur Java, puis l’exécuter avec Hadoop sur Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017, devx-track-java
ms.date: 01/16/2020
ms.openlocfilehash: 8b80616806f846951ebbaffcb2bf31ad77b96266
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996030"
---
# <a name="develop-java-mapreduce-programs-for-apache-hadoop-on-hdinsight"></a>Développer des programmes MapReduce Java pour Apache Hadoop sur HDInsight

Découvrez comment utiliser Apache Maven pour créer une application MapReduce basée sur Java, puis l’exécuter avec Apache Hadoop sur Azure HDInsight.

## <a name="prerequisites"></a>Prérequis

* [Kit de développeur Java (JDK), version 8](/azure/developer/java/fundamentals/java-jdk-long-term-support).

* [Apache Maven](https://maven.apache.org/download.cgi) correctement [installé](https://maven.apache.org/install.html) en fonction d’Apache.  Maven est un système de génération de projet pour les projets Java.

## <a name="configure-development-environment"></a>Configurer l’environnement de développement

L’environnement utilisé pour cet article était un ordinateur exécutant Windows 10. Les commandes ont été exécutées dans une invite de commandes, et les différents fichiers ont été modifiés avec le bloc-notes. Modifiez en conséquence pour votre environnement.

Dans une invite de commandes, entrez les commandes suivantes afin de créer un environnement de travail :

```cmd
IF NOT EXIST C:\HDI MKDIR C:\HDI
cd C:\HDI
```

## <a name="create-a-maven-project"></a>Création d’un projet Maven

1. Entrez la commande suivante pour créer un projet Maven nommé **wordcountjava** :

   ```bash
   mvn archetype:generate -DgroupId=org.apache.hadoop.examples -DartifactId=wordcountjava -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   ```

    Cette commande crée un répertoire du nom spécifié par le paramètre `artifactID` (**wordcountjava** dans cet exemple). Le répertoire contient les éléments suivants :

    * `pom.xml` - Le [modèle d’objet du projet (POM)](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html) qui contient les informations et la configuration utilisées pour générer le projet.
    * src\main\java\org\apache\hadoop\examples: contient votre code d’application.
    * src\test\java\org\apache\hadoop\examples: contient des tests pour votre application.

1. Supprimez l’exemple de code généré. Supprimez les fichiers d’application et de test générés `AppTest.java` et `App.java` en entrant les commandes ci-dessous :

    ```cmd
    cd wordcountjava
    DEL src\main\java\org\apache\hadoop\examples\App.java
    DEL src\test\java\org\apache\hadoop\examples\AppTest.java
    ```

## <a name="update-the-project-object-model"></a>Mise à jour du modèle d'objet du projet

Pour bénéficier d’une référence complète du fichier pom.xml, accédez à l’adresse https://maven.apache.org/pom.html. Ouvrez `pom.xml` en entrant la commande ci-dessous :

```cmd
notepad pom.xml
```

### <a name="add-dependencies"></a>Ajout de dépendances

Dans `pom.xml`, ajoutez le texte suivant dans la section `<dependencies>` :

```xml
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-mapreduce-examples</artifactId>
    <version>2.7.3</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-mapreduce-client-common</artifactId>
    <version>2.7.3</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>2.7.3</version>
    <scope>provided</scope>
</dependency>
```

Cela définit les bibliothèques nécessaires (figurant dans &lt;artifactId\>) avec une version spécifique (figurant dans &lt;version\>). Au moment de la compilation, ces dépendances sont téléchargées à partir du référentiel Maven par défaut. Vous pouvez utiliser la [recherche du référentiel Maven](https://search.maven.org/#artifactdetails%7Corg.apache.hadoop%7Chadoop-mapreduce-examples%7C2.5.1%7Cjar) pour en afficher davantage.

`<scope>provided</scope>` indique à Maven que ces dépendances ne doivent pas être fournies avec l'application, car elles sont fournies par le cluster HDInsight au moment de l'exécution.

> [!IMPORTANT]
> La version utilisée doit correspondre à la version de Hadoop présente sur votre cluster. Pour plus d’informations sur les versions, voir [Contrôle de version des composants HDInsight](../hdinsight-component-versioning.md).

### <a name="build-configuration"></a>Configuration de build

Les plug-ins Maven permettent de personnaliser les étapes de génération du projet, Cette section est utilisée pour ajouter des plug-ins, des ressources et d’autres options de configuration de build.

Ajoutez le code suivant au fichier `pom.xml`, puis enregistrez celui-ci. Ce texte doit être contenu entre les balises `<project>...</project>` dans le fichier, par exemple entre `</dependencies>` et `</project>`.

```xml
<build>
    <plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>2.3</version>
        <configuration>
        <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ApacheLicenseResourceTransformer">
            </transformer>
        </transformers>
        </configuration>
        <executions>
        <execution>
            <phase>package</phase>
                <goals>
                <goal>shade</goal>
                </goals>
        </execution>
        </executions>
        </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.6.1</version>
        <configuration>
        <source>1.8</source>
        <target>1.8</target>
        </configuration>
    </plugin>
    </plugins>
</build>
```

Cette section configure également les plug-ins Apache Maven Compiler et Apache Maven Shade. Le plug-in compiler est utilisé pour compiler la topologie. Le plug-in shade est utilisé pour empêcher la duplication de licence dans le package JAR généré par Maven. Ce plug-in est utilisé pour prévenir l’erreur relative aux fichiers de licence dupliqués lors de l’exécution sur le cluster HDInsight. Le fait d’utiliser le plug-in maven-shade-plugin avec l’implémentation `ApacheLicenseResourceTransformer` permet d’éviter cette erreur.

Le plug-in maven-shade-plugin produit également un uber jar contenant toutes les dépendances requises par l'application.

Enregistrez le fichier `pom.xml`.

## <a name="create-the-mapreduce-application"></a>Création de l’application MapReduce

1. Entrez la commande ci-dessous pour créer et ouvrir un nouveau fichier `WordCount.java`. Sélectionnez **Oui** à l'invite pour créer un fichier.

    ```cmd
    notepad src\main\java\org\apache\hadoop\examples\WordCount.java
    ```

2. Copiez et collez ensuite le code Java ci-dessous dans le nouveau fichier. Fermez le fichier.

    ```java
    package org.apache.hadoop.examples;

    import java.io.IOException;
    import java.util.StringTokenizer;
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Job;
    import org.apache.hadoop.mapreduce.Mapper;
    import org.apache.hadoop.mapreduce.Reducer;
    import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
    import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
    import org.apache.hadoop.util.GenericOptionsParser;

    public class WordCount {

        public static class TokenizerMapper
            extends Mapper<Object, Text, Text, IntWritable>{

        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context
                        ) throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
            word.set(itr.nextToken());
            context.write(word, one);
            }
        }
    }

    public static class IntSumReducer
            extends Reducer<Text,IntWritable,Text,IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values,
                            Context context
                            ) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
            sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
        if (otherArgs.length != 2) {
            System.err.println("Usage: wordcount <in> <out>");
            System.exit(2);
        }
        Job job = new Job(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
        }
    }
    ```

    Notez que le nom du package est `org.apache.hadoop.examples` et que le nom de la classe est `WordCount`. Vous utilisez ces noms lors de l’envoi de la tâche MapReduce.

## <a name="build-and-package-the-application"></a>Génération et package de l'application

À partir du répertoire `wordcountjava`, utilisez la commande suivante pour générer un fichier jar contenant l’application :

```cmd
mvn clean package
```

Cette commande nettoie les artefacts de build précédents, télécharge toute dépendance non encore installée, puis génère et met l’application en package.

Une fois la commande exécutée, le répertoire `wordcountjava/target` contient un fichier nommé `wordcountjava-1.0-SNAPSHOT.jar`.

> [!NOTE]
> Le fichier `wordcountjava-1.0-SNAPSHOT.jar` est un uberjar, qui contient non seulement la tâche WordCount, mais également les dépendances nécessaires à la tâche au moment de l'exécution.

## <a name="upload-the-jar-and-run-jobs-ssh"></a>Téléchargement du fichier JAR et exécution des travaux (SSH)

Les étapes suivantes utilisent `scp` pour copier le fichier jar sur le nœud principal d’Apache HBase sur le cluster HDInsight. La commande `ssh` est ensuite utilisée pour se connecter au cluster et exécuter l’exemple directement sur le nœud principal.

1. Chargez le fichier jar dans le cluster. Remplacez `CLUSTERNAME` par le nom de votre cluster HDInsight, puis entrez la commande suivante :

    ```cmd
    scp target/wordcountjava-1.0-SNAPSHOT.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:
    ```

1. Connectez-vous au cluster. Remplacez `CLUSTERNAME` par le nom de votre cluster HDInsight, puis entrez la commande suivante :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. À partir de la session SSH, utilisez la commande suivante pour exécuter l’application MapReduce :

   ```bash
   yarn jar wordcountjava-1.0-SNAPSHOT.jar org.apache.hadoop.examples.WordCount /example/data/gutenberg/davinci.txt /example/data/wordcountout
   ```

    Cette commande démarre l’application WordCount MapReduce. Le fichier d’entrée est `/example/data/gutenberg/davinci.txt`, et le répertoire de sortie est `/example/data/wordcountout`. Les fichiers d’entrée et de sortie sont stockés dans le stockage par défaut du cluster.

1. Une fois la tâche terminée, utilisez la commande suivante pour afficher les résultats générés :

   ```bash
   hdfs dfs -cat /example/data/wordcountout/*
   ```

    Vous devez recevoir une liste de mots et de nombres, avec des valeurs similaires au texte suivant :

    ```output
    zeal    1
    zelus   1
    zenith  2
    ```

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à développer une tâche MapReduce Java. Consultez les documents suivants pour découvrir les autres façons de travailler avec HDInsight.

* [Utilisation d’Apache Hive avec HDInsight](hdinsight-use-hive.md)
* [Utilisation de MapReduce avec HDInsight](hdinsight-use-mapreduce.md)
* [Centre de développement Java](https://azure.microsoft.com/develop/java/)