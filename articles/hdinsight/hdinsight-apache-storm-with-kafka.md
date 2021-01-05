---
title: 'Tutoriel : Apache Storm avec Apache Kafka – Azure HDInsight'
description: Découvrez comment créer un pipeline de diffusion en continu à l’aide d’Apache Storm et Apache Kafka sur HDInsight. Dans ce didacticiel, vous utilisez les composants KafkaBolt et KafkaSpout pour transmettre des données à partir de Kafka.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: tutorial
ms.date: 06/25/2019
ms.openlocfilehash: 6c600c4cfe96b849786664aa878ec1f84407da5b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85963527"
---
# <a name="tutorial-use-apache-storm-with-apache-kafka-on-hdinsight"></a>Tutoriel : Utiliser Apache Storm avec Apache Kafka sur HDInsight

Ce tutoriel montre comment utiliser une topologie [Apache Storm](https://storm.apache.org/) pour lire et écrire des données avec [Apache Kafka](https://kafka.apache.org/) sur HDInsight. Ce tutoriel montre également comment conserver des données dans le stockage [Apache Hadoop HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html) sur le cluster Storm.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Storm et Kafka
> * Présentation du code
> * Créer des clusters Kafka et Storm
> * Génération de la topologie
> * Configurer la topologie
> * Créer la rubrique Kafka
> * Lancer les topologies
> * Arrêt des topologies
> * Nettoyer les ressources

## <a name="prerequisites"></a>Prérequis

* Connaissances sur la création des rubriques Kafka. Pour plus d’informations, consultez le document [Démarrer avec Apache Kafka sur HDInsight](./kafka/apache-kafka-get-started.md).

* Connaissances sur la conception et le déploiement de solutions Storm (topologies). Plus précisément, les topologies qui utilisent le framework [Flux Apache Storm](https://storm.apache.org/releases/current/flux.html). Pour plus d’informations, consultez le document [Créer une topologie Apache Storm en Java](./storm/apache-storm-develop-java-topology.md).

* [Java JDK 1.8](https://www.oracle.com/technetwork/pt/java/javase/downloads/jdk8-downloads-2133151.html) ou version ultérieure. HDInsight 3.5 ou les versions ultérieures requièrent Java 8.

* [Maven 3.x](https://maven.apache.org/download.cgi)

* Un client SSH (vous avez besoin des commandes `ssh` et `scp`) : pour plus d’informations, voir [Utilisation de SSH avec HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

Les variables d’environnement suivantes peuvent être définies lors de l’installation de Java et du Kit de développeur Java (JDK) sur votre station de travail de développement. Toutefois, vous devez vérifier qu’elles existent et qu’elles contiennent les valeurs correctes pour votre système.

* `JAVA_HOME` : doit pointer vers le répertoire d’installation du Kit de développeur Java (JDK).
* `PATH` : doit contenir les chemins d’accès suivants :
  
    * `JAVA_HOME` (ou le chemin équivalent).
    * `JAVA_HOME\bin` (ou le chemin équivalent).
    * Répertoire d’installation de Maven.

> [!IMPORTANT]  
> Les étapes décrites dans ce document nécessitent un groupe de ressources Azure contenant à la fois un Storm sur HDInsight et un Kafka sur un cluster HDInsight. Ces clusters sont tous deux situés dans un réseau virtuel Azure, ce qui permet au cluster Storm de communiquer directement avec le cluster Kafka.
> 
> Pour des raisons pratiques, ce document renvoie à un modèle permettant de créer toutes les ressources Azure requises. 
>
> Pour plus d’informations sur l’utilisation de HDInsight dans un réseau virtuel, consultez le document [Planifier un réseau virtuel pour HDInsight](hdinsight-plan-virtual-network-deployment.md).

## <a name="storm-and-kafka"></a>Storm et Kafka

Apache Storm fournit plusieurs composants pour l’utilisation d’Apache Kafka. Les composants suivants sont utilisés dans ce didacticiel :

* `org.apache.storm.kafka.KafkaSpout`: ce composant lit les données à partir de Kafka. Ce composant s’appuie sur les composants suivants :

    * `org.apache.storm.kafka.SpoutConfig`: fournit une configuration pour le composant Spout.

    * `org.apache.storm.spout.SchemeAsMultiScheme` et `org.apache.storm.kafka.StringScheme` : comment les données issues de Kafka sont transformées en tuple Storm.

* `org.apache.storm.kafka.bolt.KafkaBolt`: ce composant écrit des données dans Kafka. Ce composant s’appuie sur les composants suivants :

    * `org.apache.storm.kafka.bolt.selector.DefaultTopicSelector`: décrit la rubrique qui reçoit les écritures.

    * `org.apache.kafka.common.serialization.StringSerializer`: configure Bolt pour sérialiser les données en tant que valeur de chaîne.

    * `org.apache.storm.kafka.bolt.mapper.FieldNameBasedTupleToKafkaMapper`: effectue un mappage à partir de la structure de données tuple utilisée dans la topologie Storm vers les champs stockés dans Kafka.

Ces composants sont disponibles dans le package `org.apache.storm : storm-kafka`. Utilisez la version du package qui correspond à la version Storm. Pour HDInsight 3.6, la version Storm est 1.1.0.
Vous avez également besoin du package `org.apache.kafka : kafka_2.10` qui contient les composants Kafka supplémentaires. Utilisez la version du package qui correspond à la version Kafka. Pour HDInsight 3.6, la version de Kafka est 1.1.1.

Le code XML suivant correspond à la déclaration de dépendance dans `pom.xml` pour un projet [Apache Maven](https://maven.apache.org/) :

```xml
<!-- Storm components for talking to Kafka -->
<dependency>
    <groupId>org.apache.storm</groupId>
    <artifactId>storm-kafka</artifactId>
    <version>1.1.0</version>
</dependency>
<!-- needs to be the same Kafka version as used on your cluster -->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka_2.10</artifactId>
    <version>1.1.1</version>
    <!-- Exclude components that are loaded from the Storm cluster at runtime -->
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## <a name="understanding-the-code"></a>Présentation du code

Le code utilisé dans ce document est disponible à l’adresse [https://github.com/Azure-Samples/hdinsight-storm-java-kafka](https://github.com/Azure-Samples/hdinsight-storm-java-kafka).

Deux topologies sont fournies avec ce didacticiel :

* Kafka-writer : génère des phrases aléatoires et les stocke dans Kafka.

* Kafka-reader : lit les données à partir de Kafka, puis les stocke dans le magasin de fichier HDFS pour le cluster Storm.

    > [!WARNING]  
    > Pour activer le fonctionnement de Storm avec le stockage HDFS utilisé par HDInsight, une action de script est requise. Le script installe plusieurs fichiers jar sur le chemin d’accès `extlib` de Storm. Le modèle de ce didacticiel utilise automatiquement le script lors de la création du cluster.
    >
    > Si vous n’utilisez pas le modèle de ce document pour créer le cluster Storm, vous devez appliquer manuellement l’action de script à votre cluster.
    >
    > L’action de script se trouve dans [https://hdiconfigactions.blob.core.windows.net/linuxstormextlibv01/stormextlib.sh](https://hdiconfigactions.blob.core.windows.net/linuxstormextlibv01/stormextlib.sh) et elle est appliquée aux nœuds de superviseur et Nimbus du cluster. Pour plus d’informations sur l’utilisation des actions de script, consultez le document [Personnaliser des clusters HDInsight à l’aide d’actions de script](hdinsight-hadoop-customize-cluster-linux.md).

Les topologies sont définies à l’aide de [Flux](https://storm.apache.org/releases/current/flux.html). Flux a été introduit dans Storm 0.10.x et vous permet de séparer la configuration de topologie du code. Pour les topologies qui utilisent l’infrastructure Flux, la topologie est définie dans un fichier YAML. Le fichier YAML peut être inclus dans le cadre de la topologie. Il peut également s’agir d’un fichier autonome utilisé lorsque vous envoyez la topologie. Flux prend également en charge la substitution des variables au moment de l’exécution, ce qui est utilisé dans cet exemple.

Les paramètres suivants sont définis au moment de l’exécution pour les topologies suivantes :

* `${kafka.topic}`: nom de la rubrique Kafka dans laquelle les topologies lisent et écrivent.

* `${kafka.broker.hosts}`: hôtes sur lesquels les répartiteurs Kafka s’exécutent. Les informations sur les répartiteurs sont utilisées par le KafkaBolt lors de l’écriture sur Kafka.

* `${kafka.zookeeper.hosts}`: hôtes sur lesquels Zookeeper s’exécute dans le cluster Kafka.

* `${hdfs.url}`: URL de système de fichiers pour le composant HDFSBolt. Indique si les données sont écrites sur un compte Stockage Azure ou Azure Data Lake Storage.

* `${hdfs.write.dir}`: répertoire dans lequel les données sont écrites.

Pour plus d’informations sur les topologies des flux, consultez [https://storm.apache.org/releases/current/flux.html](https://storm.apache.org/releases/current/flux.html).

### <a name="kafka-writer"></a>Kafka-writer

Dans la topologie Kafka-writer, le composant Bolt Kafka accepte deux valeurs de chaîne comme paramètres. Ces paramètres indiquent les champs de tuple envoyés par Bolt à Kafka en tant que valeurs __clé__ et __message__. La clé permet de partitionner les données dans Kafka. Le message correspond aux données stockées.

Dans cet exemple, le composant `com.microsoft.example.SentenceSpout` émet un tuple qui contient deux champs : `key` et `message`. Le composant Bolt Kafka extrait ces champs et envoie les données qui y figurent vers Kafka.

Les champs ne doivent pas nécessairement utiliser les noms `key` et `message`. Ces noms sont utilisés dans ce projet pour faciliter la compréhension du mappage.

Le YAML suivant correspond à la définition du composant Kafka-writer :

```yaml
# kafka-writer
---

# topology definition
# name to be used when submitting
name: "kafka-writer"

# Components - constructors, property setters, and builder arguments.
# Currently, components must be declared in the order they are referenced
components:
  # Topic selector for KafkaBolt
  - id: "topicSelector"
    className: "org.apache.storm.kafka.bolt.selector.DefaultTopicSelector"
    constructorArgs:
      - "${kafka.topic}"

  # Mapper for KafkaBolt
  - id: "kafkaMapper"
    className: "org.apache.storm.kafka.bolt.mapper.FieldNameBasedTupleToKafkaMapper"
    constructorArgs:
      - "key"
      - "message"

  # Producer properties for KafkaBolt
  - id: "producerProperties"
    className: "java.util.Properties"
    configMethods:
      - name: "put"
        args:
          - "bootstrap.servers"
          - "${kafka.broker.hosts}"
      - name: "put"
        args:
          - "acks"
          - "1"
      - name: "put"
        args:
          - "key.serializer"
          - "org.apache.kafka.common.serialization.StringSerializer"
      - name: "put"
        args:
          - "value.serializer"
          - "org.apache.kafka.common.serialization.StringSerializer"
 

# Topology configuration
config:
  topology.workers: 2

# Spout definitions
spouts:
  - id: "sentence-spout"
    className: "com.microsoft.example.SentenceSpout"
    parallelism: 8

# Bolt definitions
bolts:
  - id: "kafka-bolt"
    className: "org.apache.storm.kafka.bolt.KafkaBolt"
    parallelism: 8
    configMethods:
    - name: "withProducerProperties"
      args: [ref: "producerProperties"]
    - name: "withTopicSelector"
      args: [ref: "topicSelector"]
    - name: "withTupleToKafkaMapper"
      args: [ref: "kafkaMapper"]

# Stream definitions

streams:
  - name: "spout --> kafka" # Streams data from the sentence spout to the Kafka bolt
    from: "sentence-spout"
    to: "kafka-bolt"
    grouping:
      type: SHUFFLE
```

### <a name="kafka-reader"></a>Kafka-reader

Dans la topologie Kafka-reader, le composant Spout lit les données de Kafka en tant que valeurs de chaîne. Les données sont ensuite écrites dans le journal Storm par le composant de journalisation et dans le système de fichiers HDFS pour le cluster Storm par le composant Bolt HDFS.

```yaml
# kafka-reader
---

# topology definition
# name to be used when submitting
name: "kafka-reader"

# Components - constructors, property setters, and builder arguments.
# Currently, components must be declared in the order they are referenced
components:
  # Convert data from Kafka into string tuples in storm
  - id: "stringScheme"
    className: "org.apache.storm.kafka.StringScheme"
  - id: "stringMultiScheme"
    className: "org.apache.storm.spout.SchemeAsMultiScheme"
    constructorArgs:
      - ref: "stringScheme"

  - id: "zkHosts"
    className: "org.apache.storm.kafka.ZkHosts"
    constructorArgs:
      - "${kafka.zookeeper.hosts}"

  # Spout configuration
  - id: "spoutConfig"
    className: "org.apache.storm.kafka.SpoutConfig"
    constructorArgs:
      # brokerHosts
      - ref: "zkHosts"
      # topic
      - "${kafka.topic}"
      # zkRoot
      - ""
      # id
      - "readerid"
    properties:
      - name: "scheme"
        ref: "stringMultiScheme"

    # How often to sync files to HDFS; every 1000 tuples.
  - id: "syncPolicy"
    className: "org.apache.storm.hdfs.bolt.sync.CountSyncPolicy"
    constructorArgs:
      - 1

  # Rotate files when they hit 5 MB
  - id: "rotationPolicy"
    className: "org.apache.storm.hdfs.bolt.rotation.FileSizeRotationPolicy"
    constructorArgs:
      - 5
      - "KB"

  # File format; read the directory from filters at run time, and use a .txt extension when writing.
  - id: "fileNameFormat"
    className: "org.apache.storm.hdfs.bolt.format.DefaultFileNameFormat"
    configMethods:
      - name: "withPath"
        args: ["${hdfs.write.dir}"]
      - name: "withExtension"
        args: [".txt"]

  # Internal file format; fields delimited by `|`.
  - id: "recordFormat"
    className: "org.apache.storm.hdfs.bolt.format.DelimitedRecordFormat"
    configMethods:
      - name: "withFieldDelimiter"
        args: ["|"]

# Topology configuration
config:
  topology.workers: 2

# Spout definitions
spouts:
  - id: "kafka-spout"
    className: "org.apache.storm.kafka.KafkaSpout"
    constructorArgs:
      - ref: "spoutConfig"
    # Set to the number of partitions for the topic
    parallelism: 8

# Bolt definitions
bolts:
  - id: "logger-bolt"
    className: "com.microsoft.example.LoggerBolt"
    parallelism: 1
  
  - id: "hdfs-bolt"
    className: "org.apache.storm.hdfs.bolt.HdfsBolt"
    configMethods:
      - name: "withConfigKey"
        args: ["hdfs.config"]
      - name: "withFsUrl"
        args: ["${hdfs.url}"]
      - name: "withFileNameFormat"
        args: [ref: "fileNameFormat"]
      - name: "withRecordFormat"
        args: [ref: "recordFormat"]
      - name: "withRotationPolicy"
        args: [ref: "rotationPolicy"]
      - name: "withSyncPolicy"
        args: [ref: "syncPolicy"]
    parallelism: 1

# Stream definitions

streams:
  # Stream data to log
  - name: "kafka --> log" # name isn't used (placeholder for logging, UI, etc.)
    from: "kafka-spout"
    to: "logger-bolt"
    grouping:
      type: SHUFFLE
  
  # stream data to file
  - name: "kafka --> hdfs"
    from: "kafka-spout"
    to: "hdfs-bolt"
    grouping:
      type: SHUFFLE
```

### <a name="property-substitutions"></a>Substitutions de propriété

Le projet contient un fichier nommé `dev.properties` qui est utilisé pour transmettre des paramètres utilisés par les topologies. Il définit les propriétés suivantes :

| Fichier dev.properties | Description |
| --- | --- |
| `kafka.zookeeper.hosts` | Hôtes [Apache ZooKeeper](https://zookeeper.apache.org/) pour le cluster Kafka. |
| `kafka.broker.hosts` | Hôtes du répartiteur Kafka (nœuds Worker). |
| `kafka.topic` | Rubrique Kafka utilisée par les topologies. |
| `hdfs.write.dir` | Répertoire dans lequel la topologie Kafka-reader écrit. |
| `hdfs.url` | Système de fichiers utilisé par le cluster Storm. Pour les comptes de stockage Azure, utilisez la valeur `wasb://`. Pour Azure Data Lake Storage Gen2, utilisez la valeur `abfs://`. Pour Azure Data Lake Storage Gen1, utilisez la valeur `adl://`. |

## <a name="create-the-clusters"></a>Création des clusters

Apache Kafka sur HDInsight ne donne pas accès aux répartiteurs Kafka via l’internet public. Tout ce qui utilise Kafka doit se trouver sur le même réseau virtuel Azure. Dans ce didacticiel, les clusters Kafka et Storm se trouvent sur le même réseau virtuel Azure. 

Le diagramme suivant illustre les flux de communication entre Storm et Kafka :

![Diagramme des clusters Storm et Kafka dans un réseau virtuel Azure](./media/hdinsight-apache-storm-with-kafka/apache-storm-kafka-vnet.png)

> [!NOTE]  
> Les autres services sur le cluster, tels que SSH et [Apache Ambari](https://ambari.apache.org/), sont accessibles via Internet. Pour plus d’informations sur les ports publics disponibles avec HDInsight, consultez [Ports et URI utilisés par HDInsight](hdinsight-hadoop-port-settings-for-services.md).

Pour créer un réseau virtuel Azure puis les clusters Kafka et Storm qu’il contient, procédez comme suit :

1. Utilisez le bouton suivant pour vous connecter à Azure et ouvrir le modèle dans le portail Azure.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fhdinsight-storm-java-kafka%2Fmaster%2Fcreate-kafka-storm-clusters-in-vnet.json" target="_blank"><img src="./media/hdinsight-apache-storm-with-kafka/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

    Le modèle Azure Resource Manager se trouve dans **https://github.com/Azure-Samples/hdinsight-storm-java-kafka/blob/master/create-kafka-storm-clusters-in-vnet.json** . Il crée les ressources suivantes :

    * Groupe de ressources Azure
    * Réseau virtuel Azure
    * Compte de Stockage Azure
    * Kafka sur HDInsight version 3.6 (trois nœuds worker)
    * Storm sur HDInsight version 3.6 (trois nœuds worker)

   > [!WARNING]  
   > Pour garantir la disponibilité de Kafka sur HDInsight, votre cluster doit contenir au moins trois nœuds Worker. Ce modèle crée un cluster Kafka qui contient trois nœuds Worker.

2. Utilisez les instructions suivantes pour remplir les entrées de la section **Déploiement personnalisé** :

   1. Utilisez les informations suivantes pour renseigner les entrées dans la section **Modèle personnalisé** :

      | Paramètre | Valeur |
      | --- | --- |
      | Abonnement | Votre abonnement Azure |
      | Resource group | Le groupe de ressources qui contient les ressources. |
      | Emplacement | La région Azure dans laquelle sont créées les ressources. |
      | Nom du cluster Kafka | Le nom du cluster Kafka. |
      | Nom du cluster Storm | Nom du cluster Storm. |
      | Nom d’utilisateur de connexion au cluster | Le nom d’utilisateur administrateur pour l’accès aux clusters. |
      | Mot de passe de connexion au cluster | Le mot de passe d’utilisateur administrateur pour l’accès aux clusters. |
      | Nom d’utilisateur SSH | L’utilisateur SSH à créer pour l’accès aux clusters. |
      | Mot de passe SSH | Le mot de passe de l’utilisateur SSH. |
   
      ![Image des paramètres de modèle](./media/hdinsight-apache-storm-with-kafka/storm-kafka-template.png)

3. Passez en revue les **termes et conditions**, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.

4. Pour finir, cochez **Épingler au tableau de bord**, puis sélectionnez **Acheter**.

> [!NOTE]  
> La création des clusters peut prendre jusqu’à 20 minutes.

## <a name="build-the-topology"></a>Génération de la topologie

1. Dans votre environnement de développement, téléchargez le projet sur [https://github.com/Azure-Samples/hdinsight-storm-java-kafka](https://github.com/Azure-Samples/hdinsight-storm-java-kafka), ouvrez une interface de ligne de commande et accédez à l’emplacement auquel vous avez téléchargé le projet.

2. À partir du répertoire **hdinsight-storm-java-kafka**, utilisez la commande suivante pour compiler le projet et créer un package de déploiement :

   ```bash
   mvn clean package
   ```

    Le processus de création de package crée un fichier nommé `KafkaTopology-1.0-SNAPSHOT.jar` dans le répertoire `target`.

3. Utilisez la commande suivante pour copier le package sur votre Storm dans votre cluster HDInsight. Remplacez `sshuser` par le nom d’utilisateur SSH pour le cluster. Remplacez `stormclustername` par le nom du cluster __Storm__.

   ```bash
   scp ./target/KafkaTopology-1.0-SNAPSHOT.jar sshuser@stormclustername-ssh.azurehdinsight.net:KafkaTopology-1.0-SNAPSHOT.jar
   ```

    À l'invite, saisissez le mot de passe que vous avez utilisé lors de la création du cluster.

## <a name="configure-the-topology"></a>Configurer la topologie

1. Pour découvrir les hôtes du répartiteur Kafka pour **Kafka** sur le cluster HDInsight, utilisez l’une des méthodes suivantes :

    ```powershell
    $creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"
    $clusterName = Read-Host -Prompt "Enter the Kafka cluster name"
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER" `
        -Credential $creds `
        -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $brokerHosts = $respObj.host_components.HostRoles.host_name[0..1]
    ($brokerHosts -join ":9092,") + ":9092"
    ```

    > [!IMPORTANT]  
    > L’exemple de Bash suivant suppose que `$CLUSTERNAME` contient le nom du cluster __Kafka__. Il suppose également que la version 1.5 de [jq](https://stedolan.github.io/jq/) (ou une version ultérieure) est installée. Lorsque vous y êtes invité, entrez le mot de passe du compte de connexion au cluster.

    ```bash
    curl -su admin -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER" | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2
    ```

    Les informations renvoyées sont similaire au texte suivant :

    ```output
    wn0-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092,wn1-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092
     ```

    > [!IMPORTANT]  
    > Votre cluster pouvant disposer de plus de deux hôtes de répartiteur, il n’est pas nécessaire de fournir une liste complète des hôtes aux clients. Un ou deux sont suffisants.

2. Pour découvrir les hôtes du répartiteur Zookeeper pour __Kafka__ sur le cluster HDInsight, utilisez l’une des méthodes suivantes :

    ```powershell
    $creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"
    $clusterName = Read-Host -Prompt "Enter the Kafka cluster name"
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" `
        -Credential $creds `
        -UseBasicParsing
    $respObj = ConvertFrom-Json $resp.Content
    $zookeeperHosts = $respObj.host_components.HostRoles.host_name[0..1]
    ($zookeeperHosts -join ":2181,") + ":2181"
    ```

    > [!IMPORTANT]  
    > L’exemple de Bash suivant suppose que `$CLUSTERNAME` contient le nom du cluster __Kafka__. Il suppose également que [jq](https://stedolan.github.io/jq/) est installé. Lorsque vous y êtes invité, entrez le mot de passe du compte de connexion au cluster.

    ```bash
    curl -su admin -G "https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2
    ```

    Les informations renvoyées sont similaire au texte suivant :

    ```output
    zk0-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181,zk2-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181
    ```

    > [!IMPORTANT]  
    > Puisqu’il existe plus de deux nœuds Zookeeper, il n’est pas nécessaire de fournir une liste complète des hôtes aux clients. Un ou deux sont suffisants.

    Enregistrez cette valeur, car vous l’utiliserez plus tard.

3. Modifiez le fichier `dev.properties` à la racine du projet. Ajoutez les informations des hôtes du répartiteur et de Zookeeper pour le cluster __Kafka__ aux lignes correspondantes dans ce fichier. L’exemple suivant est configuré à l’aide des exemples de valeurs de l’étape précédente :

    ```bash
    kafka.zookeeper.hosts: zk0-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181,zk2-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:2181
    kafka.broker.hosts: wn0-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092,wn1-kafka.53qqkiavjsoeloiq3y1naf4hzc.ex.internal.cloudapp.net:9092
    kafka.topic: stormtopic
    ```

    > [!IMPORTANT]  
    > L’entrée `hdfs.url` est configurée pour un cluster qui utilise un compte de stockage Azure. Pour utiliser cette topologie avec un cluster Storm recourant à Data Lake Storage, remplacez la valeur `wasb` par `adl`.

4. Enregistrez le fichier `dev.properties`, puis utilisez la commande suivante pour le charger sur le cluster **Storm** :

     ```bash
    scp dev.properties USERNAME@BASENAME-ssh.azurehdinsight.net:dev.properties
    ```

    Remplacez **USERNAME** par le nom d’utilisateur SSH pour le cluster. Remplacez **BASENAME** par le nom de base que vous avez utilisé lors de la création du cluster.

## <a name="create-the-kafka-topic"></a>Créer la rubrique Kafka

Kafka stocke les données dans une _rubrique_. Vous devez créer la rubrique avant de commencer les topologies Storm. Pour créer la topologie, procédez comme suit :

1. Connectez-vous au cluster __Kafka__ par le biais de SSH à l’aide de la commande suivante. Remplacez `sshuser` par le nom d’utilisateur SSH que vous avez utilisé lors de la création du cluster. Remplacez `kafkaclustername` par le nom du cluster Kafka :

    ```bash
    ssh sshuser@kafkaclustername-ssh.azurehdinsight.net
    ```

    À l'invite, saisissez le mot de passe que vous avez utilisé lors de la création du cluster.
   
    Pour plus d’informations, consultez [Utiliser SSH avec HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

2. Pour créer la rubrique Kafka, utilisez la commande suivante. Remplacez `$KAFKAZKHOSTS` par les informations d’hôte ZooKeeper que vous avez utilisées lors de la configuration de la topologie :

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic stormtopic --zookeeper $KAFKAZKHOSTS
    ```

    Cette commande se connecte à ZooKeeper pour le cluster Kafka et crée une rubrique nommée `stormtopic`. Cette rubrique est utilisée par les topologies Storm.

## <a name="start-the-writer"></a>Démarrer l’enregistreur

1. Utilisez ce qui suit pour vous connecter au cluster **Storm** à l’aide de SSH. Remplacez `sshuser` par le nom d’utilisateur SSH que vous avez utilisé lors de la création du cluster. Remplacez `stormclustername` par le nom du cluster Storm :

    ```bash
    ssh sshuser@stormclustername-ssh.azurehdinsight.net
    ```

    À l'invite, saisissez le mot de passe que vous avez utilisé lors de la création du cluster.
   
    Pour plus d’informations, consultez [Utiliser SSH avec HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

2. Dans la connexion SSH sur le cluster Storm, utilisez la commande suivante pour démarrer la topologie d’enregistreur :

    ```bash
    storm jar KafkaTopology-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /writer.yaml --filter dev.properties
    ```

    Les paramètres utilisés avec cette commande sont les suivants :

    * `org.apache.storm.flux.Flux`: utilisez Flux pour configurer et exécuter cette topologie.

    * `--remote`: envoyez la topologie à Nimbus. La topologie est distribuée sur les nœuds de travail dans le cluster.

    * `-R /writer.yaml`: utilisez le fichier `writer.yaml` pour configurer la topologie. `-R` indique que cette ressource est incluse dans le fichier jar. Il se trouve à la racine du fichier jar, donc `/writer.yaml` est le chemin d’accès à celui-ci.

    * `--filter`: complétez les entrées de la topologie `writer.yaml` à l’aide des valeurs figurant dans le fichier `dev.properties`. Par exemple, la valeur de l’entrée `kafka.topic` dans le fichier est utilisée pour remplacer l’entrée `${kafka.topic}` dans la définition de la topologie.

## <a name="start-the-reader"></a>Démarrer le lecteur

1. Dans la session SSH sur le cluster Storm, utilisez la commande suivante pour démarrer la topologie de lecteur :

   ```bash
   storm jar KafkaTopology-1.0-SNAPSHOT.jar org.apache.storm.flux.Flux --remote -R /reader.yaml --filter dev.properties
   ```

2. Patientez une minute, puis utilisez la commande suivante pour afficher les fichiers créés par la topologie du lecteur :

    ```bash
    hdfs dfs -ls /stormdata
    ```

    Le résultat ressemble au texte suivant :

    ```output
    Found 173 items
      -rw-r--r--   1 storm supergroup       5137 2018-04-09 19:00 /stormdata/hdfs-bolt-4-0-1523300453088.txt
      -rw-r--r--   1 storm supergroup       5128 2018-04-09 19:00 /stormdata/hdfs-bolt-4-1-1523300453624.txt
      -rw-r--r--   1 storm supergroup       5131 2018-04-09 19:00 /stormdata/hdfs-bolt-4-10-1523300455170.txt
      ...
    ```

3. Pour afficher le contenu du fichier, utilisez la commande suivante. Remplacez `filename.txt` par le nom d’un fichier :

    ```bash
    hdfs dfs -cat /stormdata/filename.txt
    ```

    Le texte suivant constitue un exemple de contenu de fichier :

    > score de quatre et sept ans auparavant
    >
    > blanche neige et les sept nains
    >
    > je suis à deux avec la nature
    >
    > blanche neige et les sept nains
    >
    > je suis à deux avec la nature
    >
    > score de quatre et sept ans auparavant
    >
    > une pomme chaque matin éloigne le docteur

## <a name="stop-the-topologies"></a>Arrêt des topologies

Dans une session SSH sur le cluster Storm, utilisez la commande suivante pour arrêter les topologies Storm :

  ```bash
  storm kill kafka-writer
  storm kill kafka-reader
  ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer les ressources créées par ce didacticiel, vous pouvez supprimer le groupe de ressources. La suppression du groupe de ressources efface également le cluster HDInsight associé et d’autres ressources liées au groupe de ressources.

Pour supprimer le groupe de ressources à l’aide du portail Azure :

1. Sur le portail Azure, développez le menu de gauche pour ouvrir le menu des services, et sélectionnez __Groupes de ressources__ pour afficher la liste de vos groupes de ressources.
2. Recherchez le groupe de ressources à supprimer, puis faites un clic droit sur le bouton __Plus__ (...) se trouvant à droite de la liste.
3. Sélectionnez __Supprimer le groupe de ressources__ et confirmez.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à utiliser une topologie [Apache Storm](https://storm.apache.org/) pour écrire dans Kafka et lire à partir d’[Apache Kafka](https://kafka.apache.org/) sur HDInsight. Vous avez également appris à stocker des données dans le stockage [Apache Hadoop HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html) utilisé par HDInsight.

> [!div class="nextstepaction"]
> [Utiliser une API de producteur et de consommateur Apache Kafka](kafka/apache-kafka-producer-consumer-api.md)
