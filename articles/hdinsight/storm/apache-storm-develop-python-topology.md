---
title: Apache Storm avec composants Python - Azure HDInsight
description: Découvrez comment créer une topologie Apache Storm qui utilise des composants Python dans Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017, devx-track-python
ms.date: 12/16/2019
ms.openlocfilehash: e62e2583f8a4d3872a1342f0f702a1913f733e7f
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545461"
---
# <a name="develop-apache-storm-topologies-using-python-on-hdinsight"></a>Développer des topologies Storm Apache à l’aide de Python sur HDInsight

Découvrez comment créer une topologie [Apache Storm](https://storm.apache.org/) qui utilise des composants Python. Apache Storm prend en charge plusieurs langages et vous permet de combiner des composants de plusieurs langages dans une même topologie. Le framework [Flux](https://storm.apache.org/releases/current/flux.html) (datant de Storm 0.10.0) permet de créer facilement des solutions qui utilisent des composants Python.

> [!IMPORTANT]  
> Les informations contenues dans ce document ont été testées avec Storm sur HDInsight 3.6.

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache Storm sur HDInsight. Consultez la section [Création de clusters Apache Hadoop à l’aide du portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md) et sélectionnez **Storm** dans le champ **Type de cluster** .

* Un environnement de développement Storm local (facultatif). Un environnement Storm local n’est nécessaire que si vous souhaitez exécuter la topologie localement. Pour plus d’informations, consultez la page [Configurer un environnement de développement](https://storm.apache.org/releases/current/Setting-up-development-environment.html).

* [Python 2.7 ou version ultérieure](https://www.python.org/downloads/).

* [Kit de développeur Java (JDK), version 8](/azure/developer/java/fundamentals/java-jdk-long-term-support).

* [Apache Maven](https://maven.apache.org/download.cgi) correctement [installé](https://maven.apache.org/install.html) en fonction d’Apache.  Maven est un système de génération de projet pour les projets Java.

## <a name="storm-multi-language-support"></a>Prise en charge multi-langage de Storm

Apache Storm est conçu pour fonctionner avec des composants écrits dans n’importe quel langage de programmation. Les composants doivent comprendre comment travailler avec la définition Thrift pour Storm. Pour Python, un module est fourni dans le cadre du projet Apache Storm pour vous permettre de communiquer facilement avec Storm. Vous trouverez ce module à l’adresse [https://github.com/apache/storm/blob/master/storm-multilang/python/src/main/resources/resources/storm.py](https://github.com/apache/storm/blob/master/storm-multilang/python/src/main/resources/resources/storm.py).

Storm est un processus Java qui s’exécute sur une machine virtuelle Java (JVM). Les composants écrits dans d’autres langages sont exécutés en tant que sous-processus. Storm communique avec ces sous-processus à l’aide de messages JSON envoyées par le biais de stdin/stdout. Vous trouverez plus de détails sur la communication entre les composants dans la documentation relative au [Protocole multi-langage](https://storm.apache.org/releases/current/Multilang-protocol.html).

## <a name="python-with-the-flux-framework"></a>Python avec l’infrastructure Flux

L’infrastructure Flux permet de définir des topologies Storm séparément des composants. L’infrastructure de Flux utilise YAML pour définir la topologie Storm. Le texte suivant est un exemple de manière de référencer un composant Python dans le document YAML :

```yaml
# Spout definitions
spouts:
  - id: "sentence-spout"
    className: "org.apache.storm.flux.wrappers.spouts.FluxShellSpout"
    constructorArgs:
      # Command line
      - ["python", "sentencespout.py"]
      # Output field(s)
      - ["sentence"]
    # parallelism hint
    parallelism: 1
```

La classe `FluxShellSpout` est utilisée pour démarrer le script `sentencespout.py` qui implémente le Spout.

Flux s’attend à ce que les scripts Python se trouvent dans le répertoire `/resources` du fichier jar contenant la topologie. Cet exemple stocke donc les scripts Python dans le répertoire `/multilang/resources`. `pom.xml` inclut ce fichier selon le code XML suivant :

```xml
<!-- include the Python components -->
<resource>
    <directory>${basedir}/multilang</directory>
    <filtering>false</filtering>
</resource>
```

Comme mentionné précédemment, il existe un fichier `storm.py` qui implémente la définition Thrift pour Storm. L’infrastructure Flux inclut `storm.py` automatiquement lors de la génération du projet : vous n’avez donc pas besoin de vous en occuper.

## <a name="build-the-project"></a>Créer le projet

1. Téléchargez le projet à partir de [https://github.com/Azure-Samples/hdinsight-python-storm-wordcount](https://github.com/Azure-Samples/hdinsight-python-storm-wordcount).

1. Ouvrez une invite de commandes et accédez à la racine du projet : `hdinsight-python-storm-wordcount-master`. Entrez la commande suivante :

    ```cmd
    mvn clean compile package
    ```

    Cette commande crée un fichier `target/WordCount-1.0-SNAPSHOT.jar` qui contient la topologie compilée.

## <a name="run-the-storm-topology-on-hdinsight"></a>Exécuter la topologie Storm sur HDInsight

1. Utilisez la [commande ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) pour copier le fichier `WordCount-1.0-SNAPSHOT.jar` sur votre cluster Storm dans HDInsight. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    scp target/WordCount-1.0-SNAPSHOT.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:
    ```

1. Une fois le fichier chargé, connectez-vous au cluster via SSH :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Dans la session SSH, utilisez la commande suivante pour démarrer la topologie sur le cluster :

    ```bash
    storm jar WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux -r -R /topology.yaml
    ```

    Une fois démarrée, la topologie Storm s’exécute jusqu’à ce qu’elle soit arrêtée.

1. Utilisez l’interface utilisateur Storm pour afficher la topologie sur le cluster. L’interface utilisateur Storm se trouve à l’adresse `https://CLUSTERNAME.azurehdinsight.net/stormui`. Remplacez `CLUSTERNAME` par le nom de votre cluster.

1. Arrêtez la topologie Storm. Utilisez la commande suivante pour arrêter la topologie sur le cluster :

    ```bash
    storm kill wordcount
    ```

    Vous pouvez également utiliser l’interface utilisateur Storm. Sous **Actions de topologie** pour la topologie, sélectionnez **Supprimer** .

## <a name="run-the-topology-locally"></a>Exécuter la topologie localement

Pour exécuter la topologie localement, utilisez la commande suivante :

```bash
storm jar WordCount-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux -l -R /topology.yaml
```

> [!NOTE]  
> Cette commande requiert un environnement de développement Storm local. Pour plus d’informations, consultez la page [Configurer un environnement de développement](https://storm.apache.org/releases/current/Setting-up-development-environment.html).

Une fois démarrée, la topologie émet des informations de ce type sur la console locale :

```output
24302 [Thread-25-sentence-spout-executor[4 4]] INFO  o.a.s.s.ShellSpout - ShellLog pid:2436, name:sentence-spout Emiting the cow jumped over the moon
24302 [Thread-30] INFO  o.a.s.t.ShellBolt - ShellLog pid:2438, name:splitter-bolt Emitting the
24302 [Thread-28] INFO  o.a.s.t.ShellBolt - ShellLog pid:2437, name:counter-bolt Emitting years:160
24302 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=the, count=599}
24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=seven, count=302}
24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=dwarfs, count=143}
24303 [Thread-25-sentence-spout-executor[4 4]] INFO  o.a.s.s.ShellSpout - ShellLog pid:2436, name:sentence-spout Emiting the cow jumped over the moon
24303 [Thread-30] INFO  o.a.s.t.ShellBolt - ShellLog pid:2438, name:splitter-bolt Emitting cow
24303 [Thread-17-log-executor[3 3]] INFO  o.a.s.f.w.b.LogInfoBolt - {word=four, count=160}
```

Pour arrêter la topologie, appuyez sur __Ctrl+C__ .

## <a name="next-steps"></a>Étapes suivantes

Consultez les documents suivants pour découvrir d’autres façons de travailler avec HDInsight : [Comment utiliser des fonctions définies par l’utilisateur (UDF) Python dans Apache Pig et Apache Hive](../hadoop/python-udf-hdinsight.md).