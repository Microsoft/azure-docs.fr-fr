---
title: "Démarrage rapide : Recevoir des événements à l'aide d'Apache Storm - Azure Event Hubs"
description: 'Démarrage rapide : Cet article fournit des informations à propos de l’utilisation d’Apache Storm pour recevoir des événements depuis Azure Event Hubs.'
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: 099a3ff1ad57616de0d7929d787b82e9d55c9e00
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "87087575"
---
# <a name="quickstart-receive-events-from-event-hubs-using-apache-storm"></a>Démarrage rapide : Recevoir des événements d’Event Hubs avec Apache Storm

[Apache Storm](https://storm.incubator.apache.org) est un système de calcul distribué en temps réel qui simplifie de façon fiable le traitement de vastes flux de données. Cette section illustre comment utiliser une commande « spout » Storm Azure Event Hubs pour recevoir des événements provenant d'Event Hubs. À l'aide d'Apache Storm, vous pouvez fractionner des événements entre plusieurs processus hébergés dans des nœuds différents. L'intégration d'Event Hubs à Storm simplifie la consommation d'événements grâce au contrôle transparent de sa progression via l'installation de Storm Zookeeper et à la gestion des points de contrôle permanents et des réceptions en parallèle d'Event Hubs.

Pour plus d’informations sur les modèles de réception Event Hubs, consultez la page [Vue d’ensemble d’Event Hubs][Event Hubs overview].

## <a name="prerequisites"></a>Prérequis
Avant de commencer avec le démarrage rapide, **créez un espace de noms Event Hubs et un Event Hub**. Utilisez le [portail Azure](https://portal.azure.com) pour créer un espace de noms de type Event Hubs et obtenez les informations de gestion nécessaires à votre application pour communiquer avec le hub d’événements. Pour créer un espace de noms et un hub d’événements, suivez la procédure décrite dans [cet article](event-hubs-create.md). 

## <a name="create-project-and-add-code"></a>Créer le projet et ajouter du code

Ce didacticiel utilise une installation [HDInsight Storm][HDInsight Storm] , fournie avec la commande « spout » Event Hubs déjà disponible.

1. Suivez la procédure [Prise en main de Storm avec HDInsight](../hdinsight/storm/apache-storm-overview.md) pour créer un cluster HDInsight et le connecter par le biais du Bureau à distance.
2. Copiez le fichier `%STORM_HOME%\examples\eventhubspout\eventhubs-storm-spout-0.9-jar-with-dependencies.jar` dans votre environnement de développement local. Il contient la commande events-storm-spout.
3. Utilisez la commande suivante pour installer le package dans le magasin Maven local. Vous pouvez ainsi l'ajouter en tant que référence dans le projet Storm à une étape ultérieure.

    ```shell
    mvn install:install-file -Dfile=target\eventhubs-storm-spout-0.9-jar-with-dependencies.jar -DgroupId=com.microsoft.eventhubs -DartifactId=eventhubs-storm-spout -Dversion=0.9 -Dpackaging=jar
    ```
4. Dans Eclipse, créez un projet Maven (cliquez sur **Fichier**, **Nouveau**, puis **Projet**).
   
    ![Fichier -> Nouveau -> Projet][12]
5. Sélectionnez l’option **Utiliser l’emplacement d’espace de travail par défaut**, puis cliquez sur **Suivant**.
6. Sélectionnez l’archétype **maven-archetype-quickstart**, puis cliquez sur **Suivant**.
7. Insérez un **GroupId** et un **ArtifactId**, puis cliquez sur **Terminer**.
8. Dans **pom.xml**, ajoutez les dépendances ci-après dans le nœud `<dependency>`.

    ```xml  
    <dependency>
        <groupId>org.apache.storm</groupId>
        <artifactId>storm-core</artifactId>
        <version>0.9.2-incubating</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>com.microsoft.eventhubs</groupId>
        <artifactId>eventhubs-storm-spout</artifactId>
        <version>0.9</version>
    </dependency>
    <dependency>
        <groupId>com.netflix.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>1.3.3</version>
        <exclusions>
            <exclusion>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
            </exclusion>
            <exclusion>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
            </exclusion>
        </exclusions>
        <scope>provided</scope>
    </dependency>
    ```

9. Dans le dossier **src**, créez un fichier nommé **Config.properties**, puis copiez-y le contenu ci-après en remplaçant les valeurs `receive rule key` et `event hub name` :

    ```java
    eventhubspout.username = ReceiveRule
    eventhubspout.password = {receive rule key}
    eventhubspout.namespace = ioteventhub-ns
    eventhubspout.entitypath = {event hub name}
    eventhubspout.partitions.count = 16
       
    # if not provided, will use storm's zookeeper settings
    # zookeeper.connectionstring=localhost:2181
       
    eventhubspout.checkpoint.interval = 10
    eventhub.receiver.credits = 10
    ```
    La valeur d' **eventhub.receiver.credits** détermine le nombre d'événements traités par lot avant leur introduction dans le pipeline Storm. Par souci de simplicité, cet exemple définit cette valeur à 10. En production, il doit généralement être défini sur des valeurs plus élevées ; par exemple 1024.
10. Créez une classe nommée **LoggerBolt** en utilisant le code suivant :
    
    ```java
    import java.util.Map;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import backtype.storm.task.OutputCollector;
    import backtype.storm.task.TopologyContext;
    import backtype.storm.topology.OutputFieldsDeclarer;
    import backtype.storm.topology.base.BaseRichBolt;
    import backtype.storm.tuple.Tuple;
    
    public class LoggerBolt extends BaseRichBolt {
        private OutputCollector collector;
        private static final Logger logger = LoggerFactory
                  .getLogger(LoggerBolt.class);
    
        @Override
        public void execute(Tuple tuple) {
            String value = tuple.getString(0);
            logger.info("Tuple value: " + value);
   
            collector.ack(tuple);
        }
   
        @Override
        public void prepare(Map map, TopologyContext context, OutputCollector collector) {
            this.collector = collector;
            this.count = 0;
        }
        
        @Override
        public void declareOutputFields(OutputFieldsDeclarer declarer) {
            // no output fields
        }
    
    }
    ```
    
    Cette commande « bolt » de Storm enregistre le contenu des événements reçus. Cette fonction peut facilement être étendue pour stocker des tuples dans un service de stockage. L’[exemple HDInsight Storm avec Event Hub] utilise cette même approche pour stocker des données dans le Stockage Azure et Power BI.
11. Créez une classe appelée **LogTopology** en utilisant le code suivant :
    
    ```java
    import java.io.FileReader;
    import java.util.Properties;
    import backtype.storm.Config;
    import backtype.storm.LocalCluster;
    import backtype.storm.StormSubmitter;
    import backtype.storm.generated.StormTopology;
    import backtype.storm.topology.TopologyBuilder;
    import com.microsoft.eventhubs.samples.EventCount;
    import com.microsoft.eventhubs.spout.EventHubSpout;
    import com.microsoft.eventhubs.spout.EventHubSpoutConfig;
        
    public class LogTopology {
        protected EventHubSpoutConfig spoutConfig;
        protected int numWorkers;
        
        protected void readEHConfig(String[] args) throws Exception {
            Properties properties = new Properties();
            if (args.length > 1) {
                properties.load(new FileReader(args[1]));
            } else {
                properties.load(EventCount.class.getClassLoader()
                        .getResourceAsStream("Config.properties"));
            }
        
            String username = properties.getProperty("eventhubspout.username");
            String password = properties.getProperty("eventhubspout.password");
            String namespaceName = properties
                    .getProperty("eventhubspout.namespace");
            String entityPath = properties.getProperty("eventhubspout.entitypath");
            String zkEndpointAddress = properties
                    .getProperty("zookeeper.connectionstring"); // opt
            int partitionCount = Integer.parseInt(properties
                    .getProperty("eventhubspout.partitions.count"));
            int checkpointIntervalInSeconds = Integer.parseInt(properties
                    .getProperty("eventhubspout.checkpoint.interval"));
            int receiverCredits = Integer.parseInt(properties
                    .getProperty("eventhub.receiver.credits")); // prefetch count
                                                                // (opt)
            System.out.println("Eventhub spout config: ");
            System.out.println("  partition count: " + partitionCount);
            System.out.println("  checkpoint interval: "
                    + checkpointIntervalInSeconds);
            System.out.println("  receiver credits: " + receiverCredits);
     
            spoutConfig = new EventHubSpoutConfig(username, password,
                    namespaceName, entityPath, partitionCount, zkEndpointAddress,
                    checkpointIntervalInSeconds, receiverCredits);
        
            // set the number of workers to be the same as partition number.
            // the idea is to have a spout and a logger bolt co-exist in one
            // worker to avoid shuffling messages across workers in storm cluster.
            numWorkers = spoutConfig.getPartitionCount();
        
            if (args.length > 0) {
                // set topology name so that sample Trident topology can use it as
                // stream name.
                spoutConfig.setTopologyName(args[0]);
            }
        }
        
        protected StormTopology buildTopology() {
            TopologyBuilder topologyBuilder = new TopologyBuilder();
       
            EventHubSpout eventHubSpout = new EventHubSpout(spoutConfig);
            topologyBuilder.setSpout("EventHubsSpout", eventHubSpout,
                    spoutConfig.getPartitionCount()).setNumTasks(
                    spoutConfig.getPartitionCount());
            topologyBuilder
                    .setBolt("LoggerBolt", new LoggerBolt(),
                            spoutConfig.getPartitionCount())
                    .localOrShuffleGrouping("EventHubsSpout")
                    .setNumTasks(spoutConfig.getPartitionCount());
            return topologyBuilder.createTopology();
        }
        
        protected void runScenario(String[] args) throws Exception {
            boolean runLocal = true;
            readEHConfig(args);
            StormTopology topology = buildTopology();
            Config config = new Config();
            config.setDebug(false);
        
            if (runLocal) {
                config.setMaxTaskParallelism(2);
                LocalCluster localCluster = new LocalCluster();
                localCluster.submitTopology("test", config, topology);
                Thread.sleep(5000000);
                localCluster.shutdown();
            } else {
                config.setNumWorkers(numWorkers);
                StormSubmitter.submitTopology(args[0], config, topology);
            }
        }
        
        public static void main(String[] args) throws Exception {
            LogTopology topology = new LogTopology();
            topology.runScenario(args);
        }
    }
    ```

    Cette classe crée une commande « spout » Event Hubs en utilisant les propriétés du fichier de configuration pour l’instancier. Il est important de noter que cet exemple crée autant de tâches « spout » que le nombre de partitions dans le concentrateur d'événements, afin d'utiliser le parallélisme maximal autorisé par ce concentrateur d'événements.

## <a name="next-steps"></a>Étapes suivantes
Vous pouvez en apprendre plus sur Event Hubs en consultant les liens suivants :

* [Vue d’ensemble d’Event Hubs][Event Hubs overview]
* [Créer un concentrateur d’événements](event-hubs-create.md)
* [FAQ sur les hubs d’événements](event-hubs-faq.md)

<!-- Links -->
[Event Hubs overview]: ./event-hubs-about.md
[HDInsight Storm]: ../hdinsight/storm/apache-storm-overview.md
[exemple HDInsight Storm avec Event Hub]: https://github.com/Azure-Samples/hdinsight-java-storm-eventhub

<!-- Images -->

[12]: ./media/event-hubs-get-started-receive-storm/create-storm1.png
