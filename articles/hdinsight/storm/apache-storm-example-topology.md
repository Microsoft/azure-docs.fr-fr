---
title: Exemples de topologies Apache Storm sur Azure HDInsight
description: Liste d’exemples de topologies Storm créées et testées avec Apache Storm sur HDInsight, y compris les topologies C# et Java de base et l’utilisation d’Event Hubs.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/27/2019
ms.openlocfilehash: cfc68b144eac16681f162b3574a82351397ed37c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86086855"
---
# <a name="example-apache-storm-topologies-and-components-for-apache-storm-on-hdinsight"></a>Exemples de topologies et de composants Apache Storm pour Apache Storm sur HDInsight

Voici une liste d’exemples créés et gérés par Microsoft à utiliser avec [Apache Storm](https://storm.apache.org/) dans HDInsight. Ces exemples couvrent une multitude de sujets, depuis la création de topologies de base en C# et Java jusqu’à l’utilisation de services Azure comme Event Hubs, Cosmos DB, SQL Database, [Apache HBase](https://hbase.apache.org/) sur HDInsight et le stockage Azure. Certains exemples montrent également comment utiliser des technologies non Azure, ou même non Microsoft, telles que SignalR et Socket.IO.

| Description | Illustre le | Langage/structure |
|:--- |:--- |:--- |
| [Écrire dans Azure Data Lake Storage à partir d’Apache Storm](apache-storm-write-data-lake-store.md) |Écriture dans Azure Data Lake Storage |Java |
| [Source de spout et bolt Event Hub](https://github.com/apache/storm/tree/master/external/storm-eventhubs) |Source de spout et bolt Event Hub |Java |
| [Développement de topologies basées sur Java pour Apache Storm dans HDInsight][5797064f] |Maven |Java |
| [Développement de topologies C# pour Apache Storm dans HDInsight à l'aide de Visual Studio][16fce2d1] |Outils HDInsight pour Visual Studio |C#, Java |
| [Traitement des événements Azure Event Hubs avec Apache Storm sur HDInsight (C#)][844d1d81] |Event Hubs |C# et Java |
| [Traitement des événements Azure Event Hubs avec Storm sur HDInsight (Java)](https://github.com/Azure-Samples/hdinsight-java-storm-eventhub) |Event Hubs |Java |
| [Traiter des données de capteurs de véhicules issues d’Event Hubs avec Apache Storm sur HDInsight][246ee964] |Event Hubs, Cosmos DB, Azure Storage Blob (WASB) |C#, Java |
| [Extraction, transformation et chargement (ETL) à partir d’Azure Event Hubs dans Apache HBase, à l’aide d’Apache Storm sur HDInsight][b4b68194] |Event Hubs, HBase |C# |
| [Modèle de projet de topologie Storm C# pour l’utilisation des services Azure à partir d’Apache Storm sur HDInsight][ce0c02a2] |Event Hubs, Cosmos DB, SQL Database, HBase, SignalR |C#, Java |
| [Tests d’extensibilité pour la lecture de données à partir d’Azure Event Hubs à l’aide d’Apache Storm sur HDInsight][d6c540e3] |Débit des messages, Event Hubs, SQL Database |C#, Java |
| [Utiliser Apache Kafka avec Apache Storm sur HDInsight](../hdinsight-apache-storm-with-kafka.md) | Lecture et écriture Apache Storm dans Apache Kafka | Java |

> [!WARNING]  
> Les exemples C# de cette liste ont été initialement créés et testés avec HDInsight de basés sur Windows ; ils peuvent ne pas fonctionner correctement avec des clusters HDInsight sur Linux. Les clusters basés sur Linux utilisent Mono pour exécuter le code .NET, et peuvent avoir des problèmes de compatibilité avec les frameworks et les packages utilisés dans l’exemple.
>
> Linux est le seul système d’exploitation utilisé sur HDInsight version 3.4 ou supérieure.

## <a name="python-only"></a>Python uniquement

Voir [Utiliser Python avec Apache Storm sur HDInsight](apache-storm-develop-python-topology.md) pour obtenir un exemple de composants Python avec une topologie Flux.

## <a name="next-steps"></a>Étapes suivantes

* [Créer et surveiller une topologie Apache Storm dans Azure HDInsight à votre projet](./apache-storm-quickstart.md)
* [Découvrez comment déployer et gérer des topologies Apache Storm avec Apache Storm sur HDInsight][6eb0d3b8]

[6eb0d3b8]:apache-storm-deploy-monitor-topology-linux.md "Découvrez comment déployer et gérer des topologies à l’aide du tableau de bord web d’Apache Storm, de l’interface utilisateur Storm ou des outils HDInsight pour Visual Studio."
[16fce2d1]:apache-storm-develop-csharp-visual-studio-topology.md "Découvrez comment créer une topologie Storm en C# à l’aide des outils HDInsight pour Visual Studio."
[5797064f]:apache-storm-develop-java-topology.md "Découvrez comment créer des topologies Storm dans Java à l’aide de Maven en créant une topologie de comptage de mots de base."
[844d1d81]:apache-storm-develop-csharp-event-hub-topology.md "Découvrez comment lire et écrire des données à partir d’Azure Event Hubs avec Storm sur HDInsight."
[246ee964]: https://github.com/hdinsight/hdinsight-storm-examples/blob/master/IotExample/README.md "Découvrez comment utiliser une topologie Storm pour lire des messages à partir d’Azure Event Hubs, lire des documents à partir d’Azure Cosmos DB pour le référencement des données et enregistrer des données dans le stockage Azure."
[d6c540e3]: https://github.com/hdinsight/hdinsight-storm-examples/blob/master/EventCountExample "Plusieurs topologies pour illustrer le débit lors de la lecture à partir d’Azure Event Hubs et du stockage dans SQL Database à l’aide d’Apache Storm sur HDInsight."
[b4b68194]: https://github.com/hdinsight/hdinsight-storm-examples/blob/master/RealTimeETLExample "Découvrez comment lire des données à partir d’Azure Event Hubs, les agréger et les transformer, puis les stocker dans HBase sur HDInsight."
[ce0c02a2]: https://github.com/hdinsight/hdinsight-storm-examples/tree/master/templates/HDInsightStormExamples "Ce projet contient des modèles de spouts, bolts et topologies permettant d’interagir avec divers services Azure tels que Event Hubs, Cosmos DB et SQL Database."
