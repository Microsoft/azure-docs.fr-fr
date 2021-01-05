---
title: Utilisation d’Akka Streams pour Apache Kafka - Azure Event Hubs | Microsoft Docs
description: Cet article fournit des informations sur la connexion d’Akka Streams à un hub d’événements Azure
ms.topic: how-to
ms.date: 06/23/2020
ms.openlocfilehash: 92ab927189329493696c70b61ffc7f11cad22a66
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92369571"
---
# <a name="using-akka-streams-with-event-hubs-for-apache-kafka"></a>Utilisation d’Akka Streams avec Event Hubs pour Apache Kafka

Ce tutoriel vous montre comment connecter Akka Streams grâce à la prise en charge d’Event Hubs pour Apache Kafka sans modifier vos protocoles clients ni exécuter vos propres clusters. 

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Créer un espace de noms Event Hubs
> * Cloner l’exemple de projet
> * Exécuter le producteur d’Akka Streams 
> * Exécuter le consommateur d’Akka Streams

> [!NOTE]
> Cet exemple est disponible sur [GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/akka/java).

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vérifiez que les conditions préalables ci-dessous sont bien remplies :

* Lisez l’article [Event Hubs pour Apache Kafka](event-hubs-for-kafka-ecosystem-overview.md). 
* Un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.
* [Java Development Kit (JDK) 1.8+](/azure/developer/java/fundamentals/java-jdk-long-term-support)
    * Sur Ubuntu, exécutez `apt-get install default-jdk` pour installer le JDK.
    * Veillez à définir la variable d’environnement JAVA_HOME pour qu’elle pointe vers le dossier dans lequel le JDK est installé.
* [Téléchargé](https://maven.apache.org/download.cgi) et [installé](https://maven.apache.org/install.html) une archive binaire Maven.
    * Sur Ubuntu, vous pouvez exécuter `apt-get install maven` pour installer Maven.
* [Git](https://www.git-scm.com/downloads)
    * Sur Ubuntu, vous pouvez exécuter `sudo apt-get install git` pour installer Git.

## <a name="create-an-event-hubs-namespace"></a>Créer un espace de noms Event Hubs

Un espace de noms Event Hubs est requis pour échanger avec tout service Event Hubs. Pour plus d’informations, consultez [Créer un hub d’événements](event-hubs-create.md). Veillez à copier la chaîne de connexion Event Hubs pour une utilisation ultérieure.

## <a name="clone-the-example-project"></a>Cloner l’exemple de projet

Maintenant que vous disposez d’une chaîne de connexion Event Hubs, clonez le dépôt Azure Event Hubs pour Kafka, puis accédez au sous-dossier `akka` :

```shell
git clone https://github.com/Azure/azure-event-hubs-for-kafka.git
cd azure-event-hubs-for-kafka/tutorials/akka/java
```

## <a name="run-akka-streams-producer"></a>Exécuter le producteur d’Akka Streams

À l’aide de l’exemple du producteur de flux Akka fourni, envoyez des messages au service Event Hubs.

### <a name="provide-an-event-hubs-kafka-endpoint"></a>Fournir un point de terminaison Event Hubs Kafka

#### <a name="producer-applicationconf"></a>Producer application.conf

Mettez à jour des valeurs `bootstrap.servers` et `sasl.jaas.config` dans`producer/src/main/resources/application.conf` pour diriger le producteur vers le point de terminaison Event Hubs Kafka avec l’authentification correcte.

```xml
akka.kafka.producer {
    #Akka Kafka producer properties can be defined here


    # Properties defined by org.apache.kafka.clients.producer.ProducerConfig
    # can be defined in this configuration section.
    kafka-clients {
        bootstrap.servers="{YOUR.EVENTHUBS.FQDN}:9093"
        sasl.mechanism=PLAIN
        security.protocol=SASL_SSL
        sasl.jaas.config="org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"{YOUR.EVENTHUBS.CONNECTION.STRING}\";"
    }
}
```

> [!IMPORTANT]
> Remplacez `{YOUR.EVENTHUBS.CONNECTION.STRING}` par la chaîne de connexion de votre espace de noms Event Hubs. Pour savoir comment obtenir la chaîne de connexion, consultez [Obtenir une chaîne de connexion Event Hubs](event-hubs-get-connection-string.md). Voici un exemple de configuration : `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`


### <a name="run-producer-from-the-command-line"></a>Exécuter le producteur depuis la ligne de commande

Pour exécuter le producteur depuis la ligne de commande, générez le fichier JAR, puis exécutez depuis Maven (ou générez le fichier JAR avec Maven, puis exécutez dans Java en ajoutant le ou les fichiers JAR Kafka au paramètre classpath) :

```shell
mvn clean package
mvn exec:java -Dexec.mainClass="AkkaTestProducer"
```

le producteur va maintenant commencer à envoyer des événement à l’Event Hub au niveau de la rubrique `test` et imprimer les événements dans stdout.

## <a name="run-akka-streams-consumer"></a>Exécuter le consommateur d’Akka Streams

À l’aide de l’exemple de contrôle serveur consommateur fourni, recevez des messages à partir de l’Event Hub.

### <a name="provide-an-event-hubs-kafka-endpoint"></a>Fournir un point de terminaison Event Hubs Kafka

#### <a name="consumer-applicationconf"></a>Consumer application.conf

Mettez à jour des valeurs `bootstrap.servers` et `sasl.jaas.config` dans`consumer/src/main/resources/application.conf` pour diriger le contrôle serveur consommateur vers le point de terminaison Event Hubs Kafka avec l’authentification correcte.

```xml
akka.kafka.consumer {
    #Akka Kafka consumer properties defined here
    wakeup-timeout=60s

    # Properties defined by org.apache.kafka.clients.consumer.ConsumerConfig
    # defined in this configuration section.
    kafka-clients {
       request.timeout.ms=60000
       group.id=akka-example-consumer

       bootstrap.servers="{YOUR.EVENTHUBS.FQDN}:9093"
       sasl.mechanism=PLAIN
       security.protocol=SASL_SSL
       sasl.jaas.config="org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"{YOUR.EVENTHUBS.CONNECTION.STRING}\";"
    }
}
```

> [!IMPORTANT]
> Remplacez `{YOUR.EVENTHUBS.CONNECTION.STRING}` par la chaîne de connexion de votre espace de noms Event Hubs. Pour savoir comment obtenir la chaîne de connexion, consultez [Obtenir une chaîne de connexion Event Hubs](event-hubs-get-connection-string.md). Voici un exemple de configuration : `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`


### <a name="run-consumer-from-the-command-line"></a>Exécuter le contrôle serveur consommateur depuis la ligne de commande

Pour exécuter le contrôle serveur consommateur depuis la ligne de commande, générez le fichier JAR, puis exécutez depuis Maven (ou générez le fichier JAR avec Maven, puis exécutez dans Java en ajoutant le ou les fichiers JAR Kafka au paramètre classpath) :

```shell
mvn clean package
mvn exec:java -Dexec.mainClass="AkkaTestConsumer"
```

Si l’Event Hub contient des événements (par exemple, si votre producteur est également en cours d’exécution), le contrôle serveur consommateur commence maintenant à recevoir les événements provenant de la rubrique `test`. 

Consultez le [Guide Kafka de flux Akka](https://doc.akka.io/docs/akka-stream-kafka/current/home.html) pour plus d’informations sur les flux Akka.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d'informations sur Event Hubs pour Kafka, consultez les articles suivants :  

- [Mettre en miroir un broker Kafka dans un hub d’événements](event-hubs-kafka-mirror-maker-tutorial.md)
- [Connecter Apache Spark à un hub d’événements](event-hubs-kafka-spark-tutorial.md)
- [Connecter Apache Flink à un hub d’événements](event-hubs-kafka-flink-tutorial.md)
- [Intégrer Kafka Connect à un hub d’événements](event-hubs-kafka-connect-tutorial.md)
- [Explorer des exemples sur GitHub](https://github.com/Azure/azure-event-hubs-for-kafka)
- [Guide du développeur Apache Kafka pour Azure Event Hubs](apache-kafka-developer-guide.md)