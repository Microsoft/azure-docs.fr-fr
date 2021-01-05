---
title: Guide du développeur Apache Kafka pour Event Hubs
description: Cet article fournit des liens vers des articles qui décrivent comment intégrer vos applications Kafka à Azure Event Hubs.
ms.date: 06/23/2020
ms.topic: article
ms.openlocfilehash: 757db4faf2a933e366bc818d5cf6aab04d0b08ba
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90061731"
---
# <a name="apache-kafka-developer-guide-for-azure-event-hubs"></a>Guide du développeur Apache Kafka pour Azure Event Hubs
Cet article fournit des liens vers des articles qui décrivent comment intégrer vos applications Apache Kafka à Azure Event Hubs. 

## <a name="overview"></a>Vue d’ensemble
Event Hubs fournit un point de terminaison Kafka qui peut être utilisé par vos applications Kafka pour éviter d’exécuter votre propre cluster Kafka. Event Hubs fonctionne avec un grand nombre de vos applications Kafka existantes. Pour plus d’informations, consultez [Event Hubs pour Apache Kafka](event-hubs-for-kafka-ecosystem-overview.md)

## <a name="quickstarts"></a>Démarrages rapides
Vous trouverez des démarrages rapides dans GitHub et dans ce jeu de contenus pour vous aider à assimiler rapidement Event Hubs pour Kafka.

### <a name="quickstarts-in-github"></a>Démarrages rapides dans GitHub
Consultez les démarrages rapides suivants dans le référentiel **azure-event-hubs-for-kafka** : 

| Langage/structure du client | Description | 
| ------------------------- | ----------- | 
| [.NET](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/dotnet) | <p>Ce guide de démarrage rapide montre comment créer un point de terminaison Event Hubs Kafka et s’y connecter à l’aide d'un exemple de producteur et de consommateur écrit en C# à l’aide de .NET Core 2.0.</p><p>Cet exemple est basé sur le [client .NET Apache Kafka de Confluent](https://github.com/confluentinc/confluent-kafka-dotnet), modifié en vue d’une utilisation avec Event Hubs pour Kafka.</p> | 
| [Java](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/java) | Ce guide de démarrage rapide montre comment créer un point de terminaison Event Hubs Kafka et s’y connecter à l’aide d'un exemple de producteur et de consommateur écrit en Java. |
| [Node.JS](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/node) | <p>Ce guide de démarrage rapide montre comment créer un point de terminaison Event Hubs Kafka et s’y connecter à l’aide d'un exemple de producteur et de consommateur écrit en Node.</p><p>Cet exemple utilise la bibliothèque [node-rdkafka](https://github.com/Blizzard/node-rdkafka). </p>| 
| [Python](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/python) | <p>Ce guide de démarrage rapide montre comment créer un point de terminaison Event Hubs Kafka et s’y connecter à l’aide d'un exemple de producteur et de consommateur écrit en Python.</p><p>Cet exemple est basé sur le [client Python Apache Kafka de Confluent](https://github.com/confluentinc/confluent-kafka-python), modifié en vue d’une utilisation avec Event Hubs pour Kafka.</p>|
| [Go](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/go) | <p>Ce guide de démarrage rapide montre comment créer un point de terminaison Event Hubs Kafka et s’y connecter à l’aide d'un exemple de producteur et de consommateur écrit en Go.</p><p>Cet exemple est basé sur le [client Golang Apache Kafka de Confluent](https://github.com/confluentinc/confluent-kafka-go), modifié en vue d’une utilisation avec Event Hubs pour Kafka.</p>| 
| [Sarama kafka Go](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/go-sarama-client) | Ce guide de démarrage rapide montre comment créer un point de terminaison Event Hubs Kafka et s’y connecter à l’aide d'un exemple de producteur et de consommateur écrit en Go grâce à la bibliothèque du [client Sarama Kafka](https://github.com/Shopify/sarama). |
| [Kafka](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/kafka-cli) | Ce guide de démarrage rapide montre comment créer un point de terminaison Event Hubs Kafka et s’y connecter à l’aide de l’interface de ligne de commande fournie avec la distribution Apache Kafka.| 
| [Kafkacat](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/kafkacat) | kafkacat est un modèle de consommateur et de producteur en ligne de commande non-JVM basé sur librdkafka, populaire en raison de sa vitesse et de sa petite taille. Ce guide de démarrage rapide contient un exemple de configuration et plusieurs exemples simples de commandes kafkacat. | 
 
### <a name="quickstarts-in-docs"></a>Démarrages rapides dans DOCS
Consultez le démarrage rapide : [Streaming de données avec Event Hubs en utilisant le protocole Kafka](event-hubs-quickstart-kafka-enabled-event-hubs.md) inclus dans ce jeu de contenus, qui fournit des instructions pas à pas sur la façon d’effectuer un streaming dans Event Hubs. Vous allez voir comment utiliser vos producteurs et vos consommateurs pour communiquer avec Event Hubs, avec seulement un changement de configuration dans vos applications. 


## <a name="tutorials"></a>Tutoriels 

### <a name="tutorials-in-github"></a>Tutoriels dans GitHub
Consultez les tutoriels suivants sur GitHub :

| Didacticiel | Description | 
| ------------------------- | ----------- | 
| [Akka](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/akka/java) | Ce tutoriel montre comment connecter Akka Streams à des hubs d’événements prenant en charge Kafka sans modifier vos protocoles clients ni exécuter vos propres clusters. Il existe deux tutoriels distincts qui utilisent les langages de programmation **Java** et **Scala**. | 
| [Connexion](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/connect) | Ce document vous guide tout au long de l’intégration de Kafka Connect à Azure Event Hubs et du déploiement des connecteurs FileStreamSource et FileStreamSink de base. Bien que ces connecteurs ne soient pas destinés à une utilisation en production, ils présentent un scénario Kafka Connect de bout en bout où Azure Event Hubs se fait passer pour un répartiteur Kafka.| 
| [Filebeat](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/filebeat) | Ce document vous guidera tout au long de l’intégration de Filebeat et d’Event Hubs via la sortie Kafka de Filebeat. | 
| [Flink](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/flink) | Ce tutoriel montre comment connecter Apache Flink à des Event Hubs prenant en charge Kafka sans modifier vos protocoles clients ni exécuter vos propres clusters. | 
| [FluentD](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/fluentd) | Ce document vous guidera tout au long de l’intégration de Fluentd et d’Event Hubs à l’aide du plug-in de sortie `out_kafka` pour Fluentd. |
| [Interop](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/interop) | Ce tutoriel vous montre comment échanger des événements entre consommateurs et producteurs à l’aide de différents protocoles. |
| [Logstash](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/logstash) | Ce tutoriel vous guide tout au long de l’intégration de Logstash à des Event Hubs prenant en charge Kafka à l’aide de plug-ins d’entrée/sortie Logstash Kafka. | 
| [MirrorMaker](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/mirror-maker) | Ce tutoriel montre comment un Event Hub et Kafka MirrorMaker peuvent intégrer un pipeline Kafka existant dans Azure en mettant en miroir le flux d’entrée Kafka dans le service Event Hubs. |
| [NiFi](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/nifi) | Ce tutoriel montre comment connecter Apache NiFi à un espace de noms Event Hubs. | 
| [OAuth](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth) | Les guides de démarrage rapide vous montrent comment créer un point de terminaison Event Hubs Kafka et s’y connecter à l’aide d'un exemple de producteur et de consommateur écrit en Go et dans des langages de programmation Java. |
| [Schema Registry de Confluent](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/schema-registry) | Ce tutoriel vous guide tout au long de l’intégration de Schema Registry et d’Event Hubs pour Kafka. | 
| [Spark](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/spark) | Ce tutoriel montre comment connecter votre application Spark à un hub d’événements sans modifier vos protocoles clients ni exécuter vos propres clusters Kafka. | 

### <a name="tutorials-in-docs"></a>Tutoriels dans DOCS
Consultez également le tutoriel : [Processus Apache Kafka pour les événements Event Hubs à l’aide de Stream Analytics](event-hubs-kafka-stream-analytics.md) dans ce jeu de contenus, qui montre comment diffuser en continu des données dans Event Hubs et les traiter avec Azure Stream Analytics.

## <a name="how-to-guides"></a>Guides pratiques
Consultez les guides pratiques suivants dans notre documentation :

| Article | Description | 
| ------- | ----------- | 
| [Mettre en miroir un broker Kafka dans un hub d’événements](event-hubs-kafka-mirror-maker-tutorial.md) | Montre comment mettre en miroir un répartiteur Kafka dans un hub d’événements à l’aide de Kafka MirrorMaker. |
| [Connecter Apache Spark à un hub d’événements](event-hubs-kafka-spark-tutorial.md) | Vous montre comment connecter votre application Spark à Event Hubs pour un streaming en temps réel. |
| [Connecter Apache Flink à un hub d’événements](event-hubs-kafka-flink-tutorial.md) | Vous montre comment connecter Apache Flink à un Event Hub sans modifier vos protocoles clients ni exécuter vos propres clusters. |
| [Intégrer Apache Kafka Connect à un hub d’événements (préversion)](event-hubs-kafka-connect-tutorial.md) | Vous guide tout au long de l’intégration de Kafka Connect avec un hub d’événements et du déploiement des connecteurs FileStreamSource et FileStreamSink de base. |
| [Connecter Akka Streams à un hub d’événements](event-hubs-kafka-akka-streams-tutorial.md) | Vous montre comment connecter Akka Streams à un Event Hub sans modifier vos protocoles clients ni exécuter vos propres clusters. |
| [Utiliser Spring Boot Starter pour Apache Kafka avec Azure Event Hubs](/azure/developer/java/spring-framework/configure-spring-cloud-stream-binder-java-app-kafka-azure-event-hub) | Explique comment configurer un Stream Binder Spring Cloud basé sur Java développé avec l’initialiseur Spring Boot pour utiliser Apache Kafka avec Azure Event Hubs. |

## <a name="next-steps"></a>Étapes suivantes
Passez en revue les exemples du référentiel GitHub [azure-event-hubs-for-kafka](https://github.com/Azure/azure-event-hubs-for-kafka) dans les dossiers des guides de démarrage rapide et des tutoriels.

Consultez également les articles suivants :

- [Guide de dépannage Apache Kafka pour Event Hubs](apache-kafka-troubleshooting-guide.md)
- [Forum aux questions (FAQ) - Event Hubs pour Apache Kafka](apache-kafka-frequently-asked-questions.md)
- [Guide de migration Apache Kafka pour Event Hubs](apache-kafka-migration-guide.md)



