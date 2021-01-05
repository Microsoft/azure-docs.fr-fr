---
title: 'Tutoriel : Utiliser l’API de flux Apache Kafka - Azure HDInsight '
description: Didacticiel - Découvrez comment utiliser l’API de flux Apache Kafka avec Kafka sur HDInsight. Cette API vous permet d’effectuer un traitement de flux entre des rubriques dans Kafka.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.custom: hdinsightactive
ms.date: 03/20/2020
ms.openlocfilehash: 7a5982cb96891b3c0724007aa98527a07daba0fe
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92532562"
---
# <a name="tutorial-use-apache-kafka-streams-api-in-azure-hdinsight"></a>Tutoriel : Utiliser l’API de flux Apache Kafka - Azure HDInsight

Découvrez comment créer une application qui utilise l’API Apache Kafka Streams et comment l’exécuter avec Kafka sur HDInsight.

L’application utilisée dans ce didacticiel est un comptage de diffusion en continu. Elle lit les données texte à partir d’une rubrique Kafka, extrait des mots spécifiques, puis stocke le mot et le nombre associé dans une autre rubrique Kafka.

Le traitement des flux Kafka est souvent effectué à l’aide d’Apache Spark ou d’Apache Storm. Kafka version 1.1.0 (dans HDInsight 3.5 et 3.6) a introduit l’API de flux Kafka. Cette API permet de transformer des flux de données entre des rubriques d’entrée et de sortie. Dans certains cas, cela peut être une alternative à la création d’une solution de streaming Spark ou Storm.

Pour plus d’informations sur les flux Kafka, consultez la documentation [d’introduction aux flux](https://kafka.apache.org/10/documentation/streams/) sur le site Apache.org.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Comprendre le code
> * Générer et déployer l’application
> * Configurer les rubriques Kafka
> * Exécuter le code

## <a name="prerequisites"></a>Prérequis

* Un cluster Kafka sur HDInsight 3.6. Pour découvrir comment créer un cluster Kafka sur HDInsight, consultez le document [Démarrer avec Apache Kafka sur HDInsight](apache-kafka-get-started.md).

* Effectuez les étapes présentées dans le document [API de producteur et de consommateur Apache Kafka](apache-kafka-producer-consumer-api.md). Les étapes de ce document utilisent l’exemple d’application et les rubriques créés dans ce didacticiel.

* [Kit de développeur Java (JDK) version 8](/azure/developer/java/fundamentals/java-jdk-long-term-support) ou un équivalent, tel qu’OpenJDK.

* [Apache Maven](https://maven.apache.org/download.cgi) correctement [installé](https://maven.apache.org/install.html) en fonction d’Apache.  Maven est un système de génération de projet pour les projets Java.

* Un client SSH. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="understand-the-code"></a>Comprendre le code

L’exemple d’application se trouve sur [https://github.com/Azure-Samples/hdinsight-kafka-java-get-started](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started), dans le sous-répertoire `Streaming`. L’application se compose de deux fichiers :

* `pom.xml`: ce fichier définit les dépendances du projet, la version Java et les méthodes d’empaquetage.
* `Stream.java`: ce fichier implémente la logique de streaming.

### <a name="pomxml"></a>Pom.xml

Les points importants à comprendre dans le fichier `pom.xml` sont les suivants :

* Dépendances : ce projet repose sur l’API Kafka Streams, fournie par le package `kafka-clients`. Le code XML suivant définit cette dépendance :

    ```xml
    <!-- Kafka client for producer/consumer operations -->
    <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>${kafka.version}</version>
    </dependency>
    ```

    L’entrée `${kafka.version}` est déclarée dans la section `<properties>..</properties>` de `pom.xml` et elle est configurée pour la version Kafka du cluster HDInsight.

* Plug-ins : les plug-ins Maven fournissent diverses fonctionnalités. Dans ce projet, les plug-ins suivants sont utilisés :

    * `maven-compiler-plugin`: permet de définir la version Java utilisée par le projet à la version 8. Java 8 est la version requise par HDInsight 3.6.
    * `maven-shade-plugin`: permet de générer un fichier uber jar qui contient cette application ainsi que toutes les dépendances. Il permet aussi de définir le point d’entrée de l’application, si bien que vous pouvez exécuter directement le fichier Jar sans spécifier la classe principale.

### <a name="streamjava"></a>Stream.java

Le fichier [Stream.java](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Streaming/src/main/java/com/microsoft/example/Stream.java) utilise l’API Streams pour implémenter une application de comptage de mots. Il lit les données d’une rubrique Kafka nommée `test` et écrit les nombres de mots dans une rubrique nommée `wordcounts`.

Le code suivant définit l’application de comptage de mots :

```java
package com.microsoft.example;

import org.apache.kafka.common.serialization.Serde;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.KeyValue;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.KStreamBuilder;

import java.util.Arrays;
import java.util.Properties;

public class Stream
{
    public static void main( String[] args ) {
        Properties streamsConfig = new Properties();
        // The name must be unique on the Kafka cluster
        streamsConfig.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-example");
        // Brokers
        streamsConfig.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, args[0]);
        // SerDes for key and values
        streamsConfig.put(StreamsConfig.KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
        streamsConfig.put(StreamsConfig.VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());

        // Serdes for the word and count
        Serde<String> stringSerde = Serdes.String();
        Serde<Long> longSerde = Serdes.Long();

        KStreamBuilder builder = new KStreamBuilder();
        KStream<String, String> sentences = builder.stream(stringSerde, stringSerde, "test");
        KStream<String, Long> wordCounts = sentences
                .flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))
                .map((key, word) -> new KeyValue<>(word, word))
                .countByKey("Counts")
                .toStream();
        wordCounts.to(stringSerde, longSerde, "wordcounts");

        KafkaStreams streams = new KafkaStreams(builder, streamsConfig);
        streams.start();

        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
}
```

## <a name="build-and-deploy-the-example"></a>Générer et déployer l’exemple

Pour générer et déployer le projet dans votre cluster Kafka sur HDInsight, procédez comme suit :

1. Définissez votre répertoire actif en fonction de l’emplacement du répertoire `hdinsight-kafka-java-get-started-master\Streaming`, puis utilisez la commande suivante pour créer un package jar :

    ```cmd
    mvn clean package
    ```

    Cette commande crée le package dans `target/kafka-streaming-1.0-SNAPSHOT.jar`.

2. Remplacez `sshuser` par l’utilisateur SSH de votre cluster, puis remplacez `clustername` par le nom de votre cluster. Utilisez la commande suivante pour copier le fichier `kafka-streaming-1.0-SNAPSHOT.jar` dans votre cluster HDInsight. Si vous y êtes invité, entrez le mot de passe du compte d’utilisateur SSH.

    ```cmd
    scp ./target/kafka-streaming-1.0-SNAPSHOT.jar sshuser@clustername-ssh.azurehdinsight.net:kafka-streaming.jar
    ```

## <a name="create-apache-kafka-topics"></a>Créer des rubriques Apache Kafka

1. Remplacez `sshuser` par l’utilisateur SSH de votre cluster, puis remplacez `CLUSTERNAME` par le nom de votre cluster. Ouvrez une connexion SSH au cluster en entrant la commande suivante. Si vous y êtes invité, entrez le mot de passe du compte d’utilisateur SSH.

    ```bash
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

2. Installez [jq](https://stedolan.github.io/jq/), un processeur JSON en ligne de commande. À partir de la connexion SSH ouverte, entrez la commande suivante pour installer `jq` :

    ```bash
    sudo apt -y install jq
    ```

3. Configurez une variable de mot de passe. Remplacez `PASSWORD` par le mot de passe de connexion du cluster, puis entrez la commande :

    ```bash
    export password='PASSWORD'
    ```

4. Extrayez le nom du cluster avec la bonne casse. La casse réelle du nom du cluster peut être différente de la casse attendue, suivant la façon dont le cluster a été créé. Cette commande obtient la casse réelle, puis la stocke dans une variable. Entrez la commande suivante :

    ```bash
    export clusterName=$(curl -u admin:$password -sS -G "http://headnodehost:8080/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')
    ```

    > [!Note]  
    > Si vous effectuez ce processus de l’extérieur du cluster, la procédure pour stocker le nom du cluster est différente. Récupérez le nom du cluster en minuscules à partir du portail Azure. Ensuite, remplacez le nom du cluster par `<clustername>` dans la commande suivante, puis exécutez-la : `export clusterName='<clustername>'`.  

5. Pour obtenir les hôtes de répartition Kafka et les hôtes Apache ZooKeeper, utilisez les commandes suivantes. Lorsque vous y êtes invité, entrez le mot de passe du compte de connexion (admin) au cluster.

    ```bash
    export KAFKAZKHOSTS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2);

    export KAFKABROKERS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2);
    ```

    > [!Note]  
    > Ces commandes nécessitent un accès à Ambari. Si votre cluster se trouve derrière un groupe de sécurité réseau, exécutez ces commandes à partir d’un ordinateur qui peut accéder à Ambari.

6. Pour créer les rubriques utilisées par l’opération de diffusion en continu, utilisez les commandes suivantes :

    > [!NOTE]  
    > Vous recevrez peut-être un message d’erreur indiquant que la rubrique `test` existe déjà. Ce n’est pas grave : cette rubrique a peut-être été créée dans le didacticiel dédié aux API de producteur et de consommateur.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic test --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic wordcounts --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic RekeyedIntermediateTopic --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic wordcount-example-Counts-changelog --zookeeper $KAFKAZKHOSTS
    ```

    Les rubriques sont utilisées pour les opérations suivantes :

   * `test`: cette rubrique indique où les enregistrements sont reçus. L’application de diffusion en continu procède à la lecture à partir d’ici.
   * `wordcounts`: cette rubrique correspond à l’emplacement où l’application de streaming stocke sa sortie.
   * `RekeyedIntermediateTopic`: cette rubrique est utilisée pour repartitionner les données à mesure que le comptage est mis à jour par l’opérateur `countByKey`.
   * `wordcount-example-Counts-changelog`: cette rubrique est un magasin d’état utilisé par l’opération `countByKey`.

    Kafka sur HDInsight peut également être configuré pour créer des rubriques automatiquement. Pour plus d’informations, consultez le document [Configure automatic topic creation (Configurer la création automatique de rubriques)](apache-kafka-auto-create-topics.md).

## <a name="run-the-code"></a>Exécuter le code

1. Pour lancer l’application de diffusion en continu en arrière-plan, utilisez la commande suivante :

    ```bash
    java -jar kafka-streaming.jar $KAFKABROKERS $KAFKAZKHOSTS &
    ```

    Vous recevrez peut-être un avertissement concernant Apache log4j. Vous pouvez l’ignorer.

2. Pour envoyer des enregistrements à la rubrique `test`, utilisez la commande suivante afin de lancer l’application de producteur :

    ```bash
    java -jar kafka-producer-consumer.jar producer test $KAFKABROKERS
    ```

3. Une fois le producteur terminé, utilisez la commande suivante pour afficher les informations stockées dans la rubrique `wordcounts` :

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $KAFKABROKERS --topic wordcounts --formatter kafka.tools.DefaultMessageFormatter --property print.key=true --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer --from-beginning
    ```

    Les paramètres `--property` indiquent au consommateur de console qu’il faut imprimer la clé (mot) et le nombre (valeur). Ce paramètre configure également le désérialiseur à utiliser lors de la lecture de ces valeurs à partir de Kafka.

    Le résultat ressemble au texte suivant :

    ```output
    dwarfs  13635
    ago     13664
    snow    13636
    dwarfs  13636
    ago     13665
    a       13803
    ago     13666
    a       13804
    ago     13667
    ago     13668
    jumped  13640
    jumped  13641
    ```

    Le paramètre `--from-beginning` configure le lancement du consommateur au début des enregistrements stockés dans la rubrique. Le nombre est incrémenté chaque fois qu’un mot est rencontré. Par conséquent, la rubrique contient plusieurs entrées pour chaque mot, affichant un nombre croissant.

4. Utilisez __Ctrl + C__ pour quitter le producteur. Continuez à utiliser __Ctrl + C__ pour quitter l’application et le consommateur.

5. Pour supprimer les rubriques utilisées par l’opération de streaming, utilisez les commandes suivantes :

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic test --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic wordcounts --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic RekeyedIntermediateTopic --zookeeper $KAFKAZKHOSTS
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic wordcount-example-Counts-changelog --zookeeper $KAFKAZKHOSTS
    ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer les ressources créées par ce didacticiel, vous pouvez supprimer le groupe de ressources. La suppression du groupe de ressources efface également le cluster HDInsight associé et d’autres ressources liées au groupe de ressources.

Pour supprimer le groupe de ressources à l’aide du portail Azure :

1. Sur le portail Azure, développez le menu de gauche pour ouvrir le menu des services, et sélectionnez __Groupes de ressources__ pour afficher la liste de vos groupes de ressources.
2. Recherchez le groupe de ressources à supprimer, puis faites un clic droit sur le bouton __Plus__ (...) se trouvant à droite de la liste.
3. Sélectionnez __Supprimer le groupe de ressources__ et confirmez.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez vu comment utiliser l’API Apache Kafka Streams avec Kafka sur HDInsight. Consultez les articles suivants pour en savoir plus sur l’utilisation de Kafka.

> [!div class="nextstepaction"]
> [Analyser les journaux d’activité Apache Kafka](apache-kafka-log-analytics-operations-management.md)