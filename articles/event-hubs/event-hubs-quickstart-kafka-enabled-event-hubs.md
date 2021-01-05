---
title: 'Démarrage rapide : Diffusion de données en continu avec Azure Event Hubs en utilisant le protocole Kafka'
description: 'Démarrage rapide : Cet article fournit des informations sur la façon d’effectuer une diffusion en continu dans Azure Event Hubs à l’aide du protocole Kafka et d’API.'
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: 2020534a3984453bcd6eff7ad0f5c02d9e7a29ff
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92368347"
---
# <a name="quickstart-data-streaming-with-event-hubs-using-the-kafka-protocol"></a>Démarrage rapide : Streaming de données avec Event Hubs en utilisant le protocole Kafka
Ce guide de démarrage rapide montre comment diffuser en streaming des données dans Event Hubs, sans changer vos protocoles clients ni exécuter vos propres clusters. Vous allez voir comment utiliser vos producteurs et vos consommateurs pour communiquer avec Event Hubs, avec seulement un changement de configuration dans vos applications. 

> [!NOTE]
> Cet exemple est disponible sur [GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/java).

## <a name="prerequisites"></a>Prérequis

Pour suivre ce démarrage rapide, vérifiez que vous avez :

* Lisez l’article [Event Hubs pour Apache Kafka](event-hubs-for-kafka-ecosystem-overview.md).
* Un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.
* [Java Development Kit (JDK) 1.7+](/azure/developer/java/fundamentals/java-jdk-long-term-support).
* [Téléchargez](https://maven.apache.org/download.cgi) et [installez](https://maven.apache.org/install.html) une archive binaire Maven.
* [Git](https://www.git-scm.com/)


## <a name="create-an-event-hubs-namespace"></a>Créer un espace de noms Event Hubs
Quand vous créez un espace de noms Event Hubs de niveau **Standard** , le point de terminaison Kafka pour l’espace de noms est automatiquement activé. Vous pouvez alors transmettre les événements de vos applications qui utilisent le protocole Kafka vers l’espace de noms Event Hubs de niveau Standard. Suivez les instructions pas à pas de l’article [Créer un hub d’événements avec le portail Azure](event-hubs-create.md) pour créer un espace de noms Event Hubs de niveau **Standard**. 

> [!NOTE]
> Event Hubs pour Kafka est disponible uniquement avec le niveau **Standard** et les niveaux **dédiés**. Le niveau **De base** ne prend pas en charge Kafka sur Event Hubs.

## <a name="send-and-receive-messages-with-kafka-in-event-hubs"></a>Envoyer et recevoir des messages avec Kafka dans Event Hubs

1. Clonez le [dépôt Azure Event Hubs pour Kafka](https://github.com/Azure/azure-event-hubs-for-kafka).

2. Accédez à `azure-event-hubs-for-kafka/quickstart/java/producer`.

3. Mettez à jour les détails de configuration pour le producteur dans `src/main/resources/producer.config` comme suit :

    **TLS/SSL :**

    ```xml
    bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";
    ```
    
    > [!IMPORTANT]
    > Remplacez `{YOUR.EVENTHUBS.CONNECTION.STRING}` par la chaîne de connexion de votre espace de noms Event Hubs. Pour savoir comment obtenir la chaîne de connexion, consultez [Obtenir une chaîne de connexion Event Hubs](event-hubs-get-connection-string.md). Voici un exemple de configuration : `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`.

    **OAuth**  :

    ```xml
    bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=OAUTHBEARER
    sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required;
    sasl.login.callback.handler.class=CustomAuthenticateCallbackHandler;
    ```    

    Vous trouverez le code source de la classe du gestionnaire d’exemples CustomAuthenticateCallbackHandler sur GitHub [ici](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth/java/appsecret/producer/src/main/java).
4. Exécutez le code du producteur et diffusez en streaming les événements dans Event Hubs :
   
    ```shell
    mvn clean package
    mvn exec:java -Dexec.mainClass="TestProducer"                                    
    ```
    
5. Accédez à `azure-event-hubs-for-kafka/quickstart/java/consumer`.

6. Mettez à jour les détails de configuration pour le consommateur dans `src/main/resources/consumer.config` comme suit :
   
    **TLS/SSL :**

    ```xml
    bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";
    ```

    > [!IMPORTANT]
    > Remplacez `{YOUR.EVENTHUBS.CONNECTION.STRING}` par la chaîne de connexion de votre espace de noms Event Hubs. Pour savoir comment obtenir la chaîne de connexion, consultez [Obtenir une chaîne de connexion Event Hubs](event-hubs-get-connection-string.md). Voici un exemple de configuration : `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`.

    **OAuth**  :

    ```xml
    bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
    security.protocol=SASL_SSL
    sasl.mechanism=OAUTHBEARER
    sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required;
    sasl.login.callback.handler.class=CustomAuthenticateCallbackHandler;
    ``` 

    Vous trouverez le code source de la classe du gestionnaire d’exemples CustomAuthenticateCallbackHandler sur GitHub [ici](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth/java/appsecret/consumer/src/main/java).

    Vous trouverez [ici](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth) tous les exemples OAuth applicables à Event Hubs pour Kafka.
7. Exécutez le code du consommateur et traitez les événements depuis le hub d’événements à l’aide de vos clients Kafka :

    ```java
    mvn clean package
    mvn exec:java -Dexec.mainClass="TestConsumer"                                    
    ```

Si votre cluster Kafka Event Hubs a des événements, vous devez maintenant commencer à les recevoir du consommateur.

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez appris à diffuser en streaming dans Event Hubs sans changer vos clients de protocole ni exécuter vos propres clusters. Pour en savoir plus, consultez [Guide du développeur Apache Kafka pour Azure Event Hubs](apache-kafka-developer-guide.md).