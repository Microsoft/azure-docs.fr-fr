---
title: Présentation d’Apache Storm - Azure HDInsight
description: Apache Storm vous permet de traiter des flux de données en temps réel. Azure HDInsight vous permet de créer facilement des clusters Storm dans le cloud Azure. Avec Visual Studio, vous pouvez créer des solutions Storm en C#, puis les déployer sur vos clusters Storm HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: overview
ms.custom: hdinsightactive,hdiseo17may2017,seoapr2020
ms.date: 04/20/2020
ms.openlocfilehash: 97b1466e6ac1f2c2dfb931655b64b0f9937ba21d
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "82183260"
---
# <a name="what-is-apache-storm-on-azure-hdinsight"></a>Présentation d’Apache Storm sur Azure HDInsight

[Apache Storm](https://storm.apache.org/) est un système de calcul distribué, à tolérance de panne et open source. Vous pouvez utiliser Storm pour traiter des flux de données en temps réel avec [Apache Hadoop](../hadoop/apache-hadoop-introduction.md). Les solutions Storm peuvent également permettre un traitement garanti des données, ainsi que la possibilité de relire les données dont le traitement a échoué une première fois.

## <a name="why-use-apache-storm-on-hdinsight"></a>Pourquoi utiliser Apache Storm sur HDInsight ?

Le logiciel Storm sur HDInsight offre les fonctionnalités suivantes :

* __Contrat de niveau de service à 99 % sur le temps d’activité de Storm__ : Storm sur HDInsight bénéficie d’une assistance complète et continue. Storm sur HDInsight possède également un contrat SLA garantissant un taux de disponibilité de 99,9 pour cent. Cela signifie que Microsoft garantit une disponibilité de la connectivité externe du cluster Storm au moins 99,9 pour cent du temps. Pour plus d’informations, consultez le [support Azure](https://azure.microsoft.com/support/options/). Consultez également le document [Informations sur le contrat SLA pour HDInsight](https://azure.microsoft.com/support/legal/sla/hdinsight/v1_0/).

* Il prend en charge une personnalisation aisée via l’exécution de scripts sur un cluster Storm pendant ou après la création. Pour en savoir plus, voir [Personnaliser des clusters HDInsight à l’aide d’une d’action de script](../hdinsight-hadoop-customize-cluster-linux.md).

* **Créer des solutions dans plusieurs langues** : Vous pouvez écrire des composants de Storm dans la langue de votre choix, comme Java, C# et Python.

    * Intègre Visual Studio avec HDInsight pour le développement, la gestion et la surveillance des topologies C#. Pour en savoir plus, voir [Développement de topologies C# pour Apache Storm sur HDInsight à l’aide des outils Hadoop pour Visual Studio](apache-storm-develop-csharp-visual-studio-topology.md).

    * Prise en charge de l’interface Java Trident. Vous pouvez créer des topologies Storm qui prennent en charge « exactement une fois » le traitement des messages, la persistance de magasin de données « transactionnels » et un ensemble d’opérations d’analyses courantes de flux de données.

* **Mise à l’échelle dynamique** : Vous pouvez ajouter ou supprimer les nœuds Worker sans nuire à l’exécution des topologies Storm. Désactivez puis réactivez les topologies en cours d’exécution afin de tirer parti des nouveaux nœuds ajoutés avec les opérations de mise à l’échelle.

* **Créer des pipelines de streaming à l’aide de plusieurs services Azure** : Storm sur HDInsight s’intègre à d’autres services Azure, tels que Event Hubs, SQL Database, Stockage Azure et Azure Data Lake Storage. Pour un exemple de solution s’intégrant avec des services Azure, voir [Traiter des événements de Azure Event Hubs avec Apache Storm sur HDInsight](https://github.com/Azure-Samples/hdinsight-java-storm-eventhub).

Pour obtenir la liste des entreprises qui utilisent Apache Storm pour leurs solutions d’analyse en temps réel, voir [Entreprises utilisant Apache Storm](https://storm.apache.org/Powered-By.html).

Pour commencer avec Storm, consultez [Créer et surveiller une topologie Apache Storm dans Azure HDInsight](apache-storm-quickstart.md).

## <a name="how-does-apache-storm-work"></a>Fonctionnement d’Apache Storm

Storm exécute des topologies au lieu des travaux [Apache Hadoop MapReduce](https://hadoop.apache.org/docs/r1.2.1/mapred_tutorial.html) que vous connaissez peut-être. Les topologies Storm sont constituées de plusieurs composants organisés dans un graphe orienté acyclique (DAG). Flux de données entre les composants dans le graphe. Chaque composant consomme un ou plusieurs flux de données, et peut émettre éventuellement un ou plusieurs flux. Le diagramme suivant illustre les flux de données entre les composants dans une topologie de base des statistiques :

![Exemple d’organisation des composants dans une topologie Storm](./media/apache-storm-overview/example-apache-storm-topology-diagram.png)

* Les composants Spout importent les données dans une topologie. Ils émettent un ou plusieurs flux dans la topologie.

* Les composants Bolt utilisent les flux émis à partir des spouts ou des autres bolts. Les bolts peuvent éventuellement émettre des flux vers la topologie. Ils sont également responsables de l’écriture des données dans les services externes ou le stockage, comme HDFS, Kafka ou HBase.

## <a name="reliability"></a>Fiabilité

Apache Storm garantit que chaque message entrant est toujours entièrement traité, même si l’analyse des données est répartie sur des centaines de nœuds.

Le nœud Nimbus fournit des fonctionnalités similaires à celles d’Apache Hadoop JobTracker. Nimbus attribue des tâches à d’autres nœuds dans un cluster par le biais d’Apache ZooKeeper. Les nœuds Zookeeper permettent de coordonner un cluster et aident à la communication entre Nimbus et le processus Superviseur sur les nœuds Worker. Si un nœud de traitement tombe en panne, le nœud Nimbus est informé et affecte la tâche et les données associées à un autre nœud.

Par défaut, un cluster Apache Storm est configuré pour n’avoir qu’un seul nœud Nimbus. Storm sur HDInsight fournit deux nœuds Nimbus. Si le nœud principal tombe en panne, le cluster Storm bascule vers le nœud secondaire tandis que le nœud principal est récupéré. Le schéma suivant illustre la configuration du flux de tâches pour Storm sur HDInsight :

![Schéma de Nimbus, de Zookeeper et de Superviseur](./media/apache-storm-overview/storm-diagram-nimbus.png)

## <a name="ease-of-use"></a>Simplicité d'utilisation

|Utilisation |Description |
|---|---|
|Connectivité Secure Shell (SSH)|vous pouvez accéder aux nœuds principaux de votre cluster Storm via Internet, à l’aide de SSH. Vous pouvez exécuter des commandes directement sur votre cluster à l’aide de SSH. Pour en savoir plus, voir [Utilisation de SSH avec Hadoop Linux sur HDInsight depuis Linux, Unix ou OS X](../hdinsight-hadoop-linux-use-ssh-unix.md).|
|Connectivité web|tous les clusters HDInsight fournissent l’interface utilisateur web Ambari. Vous pouvez surveiller, configurer et gérer facilement des services sur votre cluster à l’aide de l’interface utilisateur web Ambari. Les clusters Storm fournissent également l’interface utilisateur Storm. Vous pouvez surveiller et gérer des topologies Storm en cours d’exécution à partir de votre navigateur à l’aide de l’interface utilisateur Storm. Pour en savoir plus, consultez les documents [Manage HDInsight using the Apache Ambari Web UI (Gérer HDInsight à l’aide de l’interface utilisateur web Ambari)](../hdinsight-hadoop-manage-ambari.md) et [Surveillance et gestion à l’aide de l’interface utilisateur Apache Storm](apache-storm-deploy-monitor-topology-linux.md#monitor-and-manage-a-topology-using-the-storm-ui).|
|Azure PowerShell et Azure CLI|PowerShell et Azure CLI fournissent des utilitaires en ligne de commande que vous pouvez utiliser avec HDInsight et d’autres services Azure, à partir de votre système client.|
|Intégration de Visual Studio|Azure Data Lake Tools pour Visual Studio inclut des modèles de projet pour la création de topologies Storm en C# à l’aide du framework SCP.NET. Data Lake Tools fournit des outils permettant de déployer, de surveiller et de gérer des solutions avec Storm sur HDInsight. Pour en savoir plus, voir [Développement de topologies C# pour Apache Storm sur HDInsight à l’aide des outils Hadoop pour Visual Studio](apache-storm-develop-csharp-visual-studio-topology.md).|

## <a name="integration-with-other-azure-services"></a>Intégration avec d’autres services Azure

* __Azure Data Lake Storage__ : consultez [Utilisation d’Azure Data Lake Storage avec Apache Storm sur HDInsight](apache-storm-write-data-lake-store.md).

* __Event Hubs__ : pour obtenir un exemple d’utilisation d’Event Hubs avec un cluster Storm, consultez les exemples suivants :

    * [Traitement des événements Azure Event Hubs avec Apache Storm sur HDInsight (Java)](https://github.com/Azure-Samples/hdinsight-java-storm-eventhub)

    * [Traitement des événements Azure Event Hubs avec Apache Storm sur HDInsight (C#)](apache-storm-develop-csharp-event-hub-topology.md)

* __SQL Database__, __Cosmos DB__, __Event Hubs__ et __HBase__ : des exemples de modèles sont inclus dans Data Lake Tools pour Visual Studio. Pour plus d’informations, voir [Développement d’une topologies C# pour Apache Storm sur HDInsight](apache-storm-develop-csharp-visual-studio-topology.md).

## <a name="apache-storm-use-cases"></a>Cas d’utilisation d’Apache Storm

Voici quelques scénarios courants dans lesquels vous pouvez utiliser Storm sur HDInsight :

* Internet des objets (IoT)
* Détection des fraudes
* Analyse des réseaux sociaux
* Extraction, transformation et chargement (ETL)
* Analyse du réseau
* Recherche
* Engagement mobile

Pour plus d‘informations sur les scénarios réels, consultez le document [Comment les entreprises utilisent-elles Apache Storm ?](https://storm.apache.org/Powered-By.html).

## <a name="development"></a>Développement

Les développeurs .NET peuvent concevoir et implémenter des topologies en C# à l’aide de Data Lake Tools pour Visual Studio. Vous pouvez également créer des topologies hybrides qui utilisent des composants Java et C#. Pour plus d‘informations, consultez la rubrique [Développer des topologies C# pour Apache Storm sur HDInsight en utilisant Visual Studio](apache-storm-develop-csharp-visual-studio-topology.md).

Vous pouvez également développer des solutions Java à l’aide de l’IDE de votre choix. Pour plus d’informations, voir [Développer des topologies Java pour Apache Storm sur HDInsight](apache-storm-develop-java-topology.md).

Python peut également être utilisé pour développer des composants Storm. Pour plus d’informations, voir [Développer des topologies Apache Storm à l’aide de Python sur HDInsight](apache-storm-develop-python-topology.md).

## <a name="common-development-patterns"></a>Modèles de développement courants

### <a name="guaranteed-message-processing"></a>Traitement de message garanti

Apache Storm peut fournir différents niveaux de traitement de message garanti. Par exemple, une application Storm de base garantit un traitement « Une fois au minimum », tandis que Trident peut garantir un traitement « Exactement une fois ». Consultez [Guarantees on data processing](https://storm.apache.org/about/guarantees-data-processing.html) sur le site apache.org.

### <a name="ibasicbolt"></a>IBasicBolt

Le modèle de lecture d’une entrée tuple (qui émet zéro, un ou plusieurs tuples) puis de confirmation de l’entrée tuple dès la fin de la méthode d’exécution est courant. Storm fournit l’interface [IBasicBolt](https://storm.apache.org/releases/current/javadocs/org/apache/storm/topology/IBasicBolt.html)pour automatiser ce modèle.

### <a name="joins"></a>Jointures

La façon dont les flux de données sont joints varie entre les applications. Vous pouvez par exemple joindre chaque tuple provenant de plusieurs flux dans un nouveau flux, ou joindre uniquement des lots de tuples pour une fenêtre spécifique. Dans les deux cas, la jonction peut être effectuée à l’aide de [fieldsGrouping](https://storm.apache.org/releases/current/javadocs/org/apache/storm/topology/InputDeclarer.html#fieldsGrouping-java.lang.String-org.apache.storm.tuple.Fields-). Le regroupement de champs constitue une façon de définir comment les tuples sont acheminés vers les bolts.

Dans l’exemple Java suivant, fieldsGrouping est utilisé pour acheminer les tuples qui proviennent des composants « 1 », « 2 » et « 3 » vers le bolt MyJoiner :

```java
builder.setBolt("join", new MyJoiner(), parallelism) .fieldsGrouping("1", new Fields("joinfield1", "joinfield2")) .fieldsGrouping("2", new Fields("joinfield1", "joinfield2")) .fieldsGrouping("3", new Fields("joinfield1", "joinfield2"));
```

### <a name="batches"></a>Lots

Apache Storm fournit un mécanisme de minutage interne appelé « tuple de graduation ». Vous pouvez définir la fréquence à laquelle un tuple de graduation est émis dans votre topologie.

Pour obtenir un exemple d’utilisation d’un tick tuple à partir d’un composant C#, consultez le fichier [PartialBoltCount.cs](https://github.com/hdinsight/hdinsight-storm-examples/blob/3b2c960549cac122e8874931df4801f0934fffa7/EventCountExample/EventCountTopology/src/main/java/com/microsoft/hdinsight/storm/examples/PartialCountBolt.java).

### <a name="caches"></a>Caches

La mise en cache en mémoire est souvent utilisée pour accélérer le traitement, car elle garde en mémoire les éléments les plus utilisés. Dans la mesure où une topologie est distribuée entre plusieurs nœuds et plusieurs processus au sein de chaque nœud, vous devez envisager d’utiliser [fieldsGrouping](https://storm.apache.org/releases/current/javadocs/org/apache/storm/topology/InputDeclarer.html#fieldsGrouping-java.lang.String-org.apache.storm.tuple.Fields-). Utilisez `fieldsGrouping` pour garantir que les tuples qui contiennent les champs utilisés pour la recherche dans le cache sont toujours acheminés vers le même processus. Cette fonctionnalité de groupage permet d’éviter la duplication des entrées de cache à travers les processus.

### <a name="stream-top-n"></a>Diffuser les N premiers

Lorsque votre topologie dépend du calcul d’une valeur « N premiers », calculez cette valeur en parallèle. Puis, fusionnez le résultat de ces calculs dans une valeur globale. Cette opération peut être effectuée à l’aide de l’élément [fieldsGrouping](https://storm.apache.org/releases/current/javadocs/org/apache/storm/topology/InputDeclarer.html#fieldsGrouping-java.lang.String-org.apache.storm.tuple.Fields-) pour acheminer les données par champ pour un traitement parallèle. Vous pouvez ensuite acheminer les données vers un bolt qui détermine globalement la valeur « N premiers ».

Pour obtenir un exemple de calcul pour une valeur « N premiers », voir l’exemple [RollingTopWords](https://github.com/apache/storm/blob/master/examples/storm-starter/src/jvm/org/apache/storm/starter/RollingTopWords.java).

## <a name="logging"></a>Journalisation

Storm utilise Apache Log4j 2 pour journaliser les informations. Par défaut, une grande quantité de données est journalisée, et il peut donc être difficile d’effectuer un tri parmi toutes ces informations. Vous pouvez inclure un fichier de configuration de journalisation dans votre topologie Storm pour contrôler le comportement de journalisation.

Pour découvrir un exemple de topologie indiquant comment configurer la journalisation, consultez l’exemple de [comptage du nombre de mots basé sur Java](apache-storm-develop-java-topology.md) pour Storm sur HDInsight.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les solutions d‘analyse en temps réel avec Apache Storm dans HDInsight :

* [Créer et surveiller une topologie Apache Storm dans Azure HDInsight à votre projet](apache-storm-quickstart.md)
* [Exemples de topologies pour Apache Storm dans HDInsight](apache-storm-example-topology.md)
