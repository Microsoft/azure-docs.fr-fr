---
title: 'Tutoriel : API de producteur et de consommateur Apache Kafka – Azure HDInsight'
description: Découvrez comment utiliser les API de consommateur et de producteur Apache Kafka avec Kafka sur HDInsight. Dans ce didacticiel, vous allez apprendre à utiliser ces API avec Kafka sur HDInsight à partir d’une application Java.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 05/19/2020
ms.openlocfilehash: b942fb321d2bceef64930bea0c660f66747508b6
ms.sourcegitcommit: 3e8058f0c075f8ce34a6da8db92ae006cc64151a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92629304"
---
# <a name="tutorial-use-the-apache-kafka-producer-and-consumer-apis"></a>Tutoriel : Utiliser les API de producteur et de consommateur Apache Kafka

Découvrez comment utiliser les API de consommateur et de producteur Apache Kafka avec Kafka sur HDInsight.

L’API de producteur Kafka permet aux applications d’envoyer des flux de données au cluster Kafka. L’API de consommateur Kafka permet aux applications de lire des flux de données à partir du cluster.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Prérequis
> * Comprendre le code
> * Générer et déployer l’application
> * Exécuter l’application sur le cluster

Pour plus d’informations sur les API, consultez la documentation Apache sur l’[API de producteur](https://kafka.apache.org/documentation/#producerapi) et l’[API de consommateur](https://kafka.apache.org/documentation/#consumerapi).

## <a name="prerequisites"></a>Prérequis

* Apache Kafka sur un cluster HDInsight. Pour découvrir comment créer un cluster, consultez [Démarrer avec Apache Kafka sur HDInsight](apache-kafka-get-started.md).
* [Kit de développeur Java (JDK) version 8](/azure/developer/java/fundamentals/java-jdk-long-term-support) ou un équivalent, tel qu’OpenJDK.
* [Apache Maven](https://maven.apache.org/download.cgi) correctement [installé](https://maven.apache.org/install.html) en fonction d’Apache.  Maven est un système de génération de projet pour les projets Java.
* Un client SSH tel que Putty. Pour plus d’informations, consultez [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="understand-the-code"></a>Comprendre le code

L’exemple d’application se trouve sur [https://github.com/Azure-Samples/hdinsight-kafka-java-get-started](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started), dans le sous-répertoire `Producer-Consumer`. Si vous utilisez un cluster Kafka où est activé le **Pack Sécurité Entreprise** , vous devez utiliser la version de l’application qui est située dans le sous-répertoire `DomainJoined-Producer-Consumer`.

L’application se compose principalement de quatre fichiers :
* `pom.xml`: ce fichier définit les dépendances du projet, la version Java et les méthodes d’empaquetage.
* `Producer.java`: ce fichier envoie des phrases aléatoires à Kafka à l’aide de l’API de producteur.
* `Consumer.java`: ce fichier utilise l’API de consommateur pour lire les données de Kafka et les émettre dans STDOUT.
* `AdminClientWrapper.java`: Ce fichier utilise l’API d’administration pour créer, décrire et supprimer des rubriques Kafka.
* `Run.java`: interface de ligne de commande utilisée pour exécuter le code producteur et consommateur.

### <a name="pomxml"></a>Pom.xml

Les points importants à comprendre dans le fichier `pom.xml` sont les suivants :

* Dépendances : ce projet repose sur les API de producteur et de consommateur Kafka, fournies par le package `kafka-clients`. Le code XML suivant définit cette dépendance :

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

    * `maven-compiler-plugin`: permet de définir la version Java utilisée par le projet à la version 8. Il s’agit de la version de Java utilisée par HDInsight 3.6.
    * `maven-shade-plugin`: permet de générer un fichier uber jar qui contient cette application, ainsi que toutes les dépendances. Il permet aussi de définir le point d’entrée de l’application, afin que vous puissiez exécuter directement le fichier JAR sans spécifier la classe principale.

### <a name="producerjava"></a>Producer.java

Le producteur communique avec les hôtes du répartiteur Kafka (nœuds de travail) et envoie des données dans une rubrique Kafka. L’extrait de code suivant provient du fichier [Producer.java](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Producer-Consumer/src/main/java/com/microsoft/example/Producer.java) dans le [dépôt GitHub](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started), et montre comment définir les propriétés du producteur. Pour les clusters Sécurité Entreprise, une propriété supplémentaire doit être ajoutée "properties.setProperty(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SASL_PLAINTEXT");"

```java
Properties properties = new Properties();
// Set the brokers (bootstrap servers)
properties.setProperty("bootstrap.servers", brokers);
// Set how to serialize key/value pairs
properties.setProperty("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
properties.setProperty("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
KafkaProducer<String, String> producer = new KafkaProducer<>(properties);
```

### <a name="consumerjava"></a>Consumer.java

Le consommateur communique avec les hôtes de répartition Kafka (nœuds de travail) et lit les enregistrements dans une boucle. L’extrait de code suivant provenant du fichier [Consumer.java](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Producer-Consumer/src/main/java/com/microsoft/example/Consumer.java) définit les propriétés du consommateur. Pour les clusters Sécurité Entreprise, une propriété supplémentaire doit être ajoutée "properties.setProperty(CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, "SASL_PLAINTEXT");"

```java
KafkaConsumer<String, String> consumer;
// Configure the consumer
Properties properties = new Properties();
// Point it to the brokers
properties.setProperty("bootstrap.servers", brokers);
// Set the consumer group (all consumers must belong to a group).
properties.setProperty("group.id", groupId);
// Set how to serialize key/value pairs
properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
// When a group is first created, it has no offset stored to start reading from. This tells it to start
// with the earliest record in the stream.
properties.setProperty("auto.offset.reset","earliest");

consumer = new KafkaConsumer<>(properties);
```

Dans ce code, le consommateur est configuré pour procéder à la lecture à partir du début de la rubrique (`auto.offset.reset` a la valeur `earliest`.)

### <a name="runjava"></a>Run.java

Le fichier [Run.java](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Producer-Consumer/src/main/java/com/microsoft/example/Run.java) fournit une interface de ligne de commande qui exécute le code producteur ou consommateur. Vous devez fournir les informations relatives à l’hôte de répartition Kafka en tant que paramètre. Vous pouvez également inclure une valeur d’ID de groupe, utilisée par le processus consommateur. Si vous créez plusieurs instances de consommateur en utilisant le même ID de groupe, elles équilibrent la charge de lecture à partir de la rubrique.

## <a name="build-and-deploy-the-example"></a>Générer et déployer l’exemple

### <a name="use-pre-built-jar-files"></a>Utiliser des fichiers JAR prédéfinis

Téléchargez les fichiers jar à partir de l’[Exemple Kafka bien démarrer avec Azure](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/tree/master/Prebuilt-Jars). Si le **Pack Sécurité Entreprise** est activé dans votre cluster, utilisez kafka-producer-consumer-esp.jar. Utilisez la commande ci-dessous pour copier les fichiers jar dans votre cluster.

```cmd
scp kafka-producer-consumer*.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:kafka-producer-consumer.jar
```

### <a name="build-the-jar-files-from-code"></a>Générer les fichiers JAR à partir du code

Si vous souhaitez ignorer cette étape, vous pouvez télécharger les fichiers jar prédéfinis à partir du sous-répertoire `Prebuilt-Jars`. Téléchargez kafka-producer-consumer.jar. Si le **Pack Sécurité Entreprise** est activé dans votre cluster, utilisez kafka-producer-consumer-esp.jar. Effectuez l’étape 3 pour copier le fichier jar dans votre cluster HDInsight.

1. Téléchargez et extrayez les exemples à partir de [https://github.com/Azure-Samples/hdinsight-kafka-java-get-started](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started).

2. Configurez votre répertoire actuel sur l’emplacement du répertoire `hdinsight-kafka-java-get-started\Producer-Consumer`. Si vous utilisez un cluster Kafka où est activé le **Pack Sécurité Entreprise** , vous devez définir l’emplacement sur le sous-répertoire `DomainJoined-Producer-Consumer`. Exécutez la commande suivante pour générer l’application :

    ```cmd
    mvn clean package
    ```

    Cette commande crée un répertoire nommé `target`, qui contient un fichier nommé `kafka-producer-consumer-1.0-SNAPSHOT.jar`. Pour les clusters avec Pack Sécurité Entreprise activé, le fichier sera `kafka-producer-consumer-esp-1.0-SNAPSHOT.jar`

3. Remplacez `sshuser` par l’utilisateur SSH de votre cluster, puis remplacez `CLUSTERNAME` par le nom de votre cluster. Entrez la commande suivante pour copier le fichier `kafka-producer-consumer-1.0-SNAPSHOT.jar` dans votre cluster HDInsight. Lorsque vous y êtes invité, entrez le mot de passe de l’utilisateur SSH.

    ```cmd
    scp ./target/kafka-producer-consumer*.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:kafka-producer-consumer.jar
    ```

## <a name="run-the-example"></a><a id="run"></a> Exécuter l’exemple

1. Remplacez `sshuser` par l’utilisateur SSH de votre cluster, puis remplacez `CLUSTERNAME` par le nom de votre cluster. Ouvrez une connexion SSH au cluster en entrant la commande suivante. Si vous y êtes invité, entrez le mot de passe du compte d’utilisateur SSH.

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Pour obtenir les hôtes du répartiteur Kafka, remplacez les valeurs `<clustername>` et `<password>` de la commande suivante, puis exécutez-la. Utilisez la même casse pour `<clustername>`, comme indiqué dans le portail Azure. Remplacez `<password>` par le mot de passe de connexion du cluster, puis exécutez :

    ```bash
    sudo apt -y install jq
    export clusterName='<clustername>'
    export password='<password>'
    export KAFKABROKERS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2);
    ```

    > [!Note]  
    > Cette commande nécessite un accès à Ambari. Si votre cluster se trouve derrière un groupe de sécurité réseau, exécutez cette commande à partir d’un ordinateur qui peut accéder à Ambari.

1. Pour créer la rubrique Kafka, `myTest`, entrez la commande suivante :

    ```bash
    java -jar kafka-producer-consumer.jar create myTest $KAFKABROKERS
    ```

1. Pour exécuter le producteur et écrire des données dans la rubrique, utilisez la commande suivante :

    ```bash
    java -jar kafka-producer-consumer.jar producer myTest $KAFKABROKERS
    ```

1. Une fois que le producteur a terminé, utilisez la commande suivante pour lire à partir de la rubrique :

    ```bash
    java -jar kafka-producer-consumer.jar consumer myTest $KAFKABROKERS
    scp ./target/kafka-producer-consumer*.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.net:kafka-producer-consumer.jar
    ```

    Les enregistrements lus et le nombre d’enregistrements s’affichent.

1. Utilisez __Ctrl + C__ pour quitter le consommateur.

### <a name="multiple-consumers"></a>Consommateurs multiples

Les consommateurs Kafka utilisent un groupe de consommateurs lors de la lecture des enregistrements. L’utilisation du même groupe avec plusieurs consommateurs permet des lectures à charge équilibrée à partir d’une rubrique. Chaque consommateur dans le groupe reçoit une partie des enregistrements.

L’application consommatrice accepte un paramètre utilisé comme ID de groupe. Par exemple, la commande suivante démarre un consommateur à l’aide de l’ID de groupe `myGroup` :

```bash
java -jar kafka-producer-consumer.jar consumer myTest $KAFKABROKERS myGroup
```

Utilisez __Ctrl + C__ pour quitter le consommateur.

Pour voir ce processus en action, utilisez la commande suivante :

```bash
tmux new-session 'java -jar kafka-producer-consumer.jar consumer myTest $KAFKABROKERS myGroup' \
\; split-window -h 'java -jar kafka-producer-consumer.jar consumer myTest $KAFKABROKERS myGroup' \
\; attach
```

Cette commande utilise `tmux` pour fractionner le terminal en deux colonnes. Un consommateur est lancé dans chaque colonne, avec la même valeur d’ID de groupe. Une fois que les consommateurs ont terminé la lecture, notez que chacun a lu uniquement une partie des enregistrements. Appuyez deux fois sur __Ctrl+C__ pour quitter `tmux`.

La consommation par les clients au sein du même groupe est gérée par le biais des partitions de la rubrique. Dans cet exemple de code, la rubrique `test` créée précédemment possède huit partitions. Si vous lancez huit consommateurs, chaque consommateur lit les enregistrements à partir d’une seule partition pour la rubrique.

> [!IMPORTANT]  
> Il ne peut pas y avoir plus d’instances de consommateurs dans un groupe de consommateurs que de partitions. Dans cet exemple, un groupe de consommateurs peut contenir jusqu’à huit consommateurs puisque c’est le nombre de partitions de la rubrique. Vous pouvez également disposer de plusieurs groupes de consommateurs, chacun ne dépassant pas huit consommateurs.

Les enregistrements stockés dans Kafka le sont dans l’ordre de réception au sein d’une partition. Pour obtenir la livraison chronologique des enregistrements *dans une partition* , créez un groupe de consommateurs où le nombre d’instances de consommateurs correspond au nombre de partitions. Pour obtenir la livraison chronologique des enregistrements *dans la rubrique* , créez un groupe de consommateurs avec une seule instance de consommateur.

## <a name="common-issues-faced"></a>Problèmes courants rencontrés

1. **La création de rubriques échoue** Si le Pack Sécurité Entreprise est activé sur votre cluster, utilisez les [fichiers JAR prédéfinis pour le producteur et le consommateur](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Prebuilt-Jars/kafka-producer-consumer-esp.jar). Le fichier jar du Pack Sécurité Entreprise peut être généré à partir du code dans le [sous-répertoire `DomainJoined-Producer-Consumer`](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/tree/master/DomainJoined-Producer-Consumer). Les propriétés producteur et consommateur sont dotées d’une propriété supplémentaire `CommonClientConfigs.SECURITY_PROTOCOL_CONFIG` pour les clusters dont le Pack Sécurité Entreprise est activé.

2. **Échec dans les clusters dont le Pack Sécurité Entreprise est activé**  : Si les opérations de production et de consommation échouent et que vous utilisez un cluster dont le Pack Sécurité Entreprise est activé, vérifiez que l’utilisateur `kafka` est présent dans toutes les stratégies Ranger. S’il ne l’est pas, ajoutez-le à toutes les stratégies Ranger.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer les ressources créées par ce didacticiel, vous pouvez supprimer le groupe de ressources. La suppression du groupe de ressources efface également le cluster HDInsight associé et d’autres ressources liées au groupe de ressources.

Pour supprimer le groupe de ressources à l’aide du portail Azure :

1. Sur le portail Azure, développez le menu de gauche pour ouvrir le menu des services, et sélectionnez __Groupes de ressources__ pour afficher la liste de vos groupes de ressources.
2. Recherchez le groupe de ressources à supprimer, puis faites un clic droit sur le bouton __Plus__ (...) se trouvant à droite de la liste.
3. Sélectionnez __Supprimer le groupe de ressources__ et confirmez.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez découvert comment utiliser les API de consommateur et de producteur Apache Kafka avec Kafka sur HDInsight. Consultez les articles suivants pour en savoir plus sur l’utilisation de Kafka :

* [Utiliser le proxy REST Kafka](rest-proxy.md)
* [Analyser les journaux d’activité Apache Kafka](apache-kafka-log-analytics-operations-management.md)