---
title: Se connecter avec votre application Apache Spark - Azure Event Hubs | Microsoft Docs
description: Cet article fournit des informations sur l’utilisation d’Apache Spark avec Azure Event Hubs pour Kafka.
ms.topic: how-to
ms.date: 06/23/2020
ms.openlocfilehash: 84184ed3dffee97863b93c592d1cd577df313605
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913736"
---
# <a name="connect-your-apache-spark-application-with-azure-event-hubs"></a>Connecter votre application Apache Spark avec Azure Event Hubs
Ce tutoriel vous montre comment connecter votre application Spark à Event Hubs pour un streaming en temps réel. Cette intégration permet la diffusion en continu sans avoir à modifier vos clients de protocole ni exécuter vos propres clusters Zookeeper ou Kafka. Ce tutoriel nécessite Apache Spark v2.4+ et Apache Kafka v2.0+.

> [!NOTE]
> Cet exemple est disponible sur [GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/spark/).

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Créer un espace de noms Event Hubs
> * Cloner l’exemple de projet
> * Exécuter Spark
> * Lire à partir d’Event Hubs pour Kafka
> * Écrire dans Event Hubs pour Kafka

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel, vérifiez que vous possédez :
-   Abonnement Azure. Si vous n’en avez pas, [créez un compte gratuit](https://azure.microsoft.com/free/).
-   [Apache Spark v2.4](https://spark.apache.org/downloads.html)
-   [Apache Kafka v2.0]( https://kafka.apache.org/20/documentation.html)
-   [Git](https://www.git-scm.com/downloads)

> [!NOTE]
> L’adaptateur Spark-Kafka a été mis à jour pour pouvoir prendre en charge Kafka v2.0 et Spark v2.4. Dans des versions précédentes de Spark, l’adaptateur prenait en charge les versions Kafka v0.10 et antérieures, mais s’appuyaient particulièrement sur les API Kafka v0.10. Comme Event Hubs pour Kafka ne prend pas en charge Kafka v0.10, les adaptateurs Spark-Kafka des versions Spark antérieures à v2.4 ne sont pas pris en charge par Event Hubs pour les écosystèmes Kafka.


## <a name="create-an-event-hubs-namespace"></a>Créer un espace de noms Event Hubs
Un espace de noms Event Hubs est requis pour échanger avec tout service Event Hubs. Pour obtenir des instructions sur la création d'un espace de noms et d'un Event Hub, consultez [Créer un Event Hub](event-hubs-create.md). Obtenez la chaîne de connexion Event Hubs et le nom de domaine complet (FQDN) pour une utilisation ultérieure. Pour obtenir des instructions, consultez [Obtenir une chaîne de connexion Event Hubs](event-hubs-get-connection-string.md). 

## <a name="clone-the-example-project"></a>Cloner l’exemple de projet
Clonez le référentiel Azure Event Hubs et accédez au sous-dossier `tutorials/spark` :

```bash
git clone https://github.com/Azure/azure-event-hubs-for-kafka.git
cd azure-event-hubs-for-kafka/tutorials/spark
```

## <a name="read-from-event-hubs-for-kafka"></a>Lire à partir d’Event Hubs pour Kafka
Après quelques modifications de la configuration, vous pouvez commencer à lire à partir d’Event Hubs pour Kafka. Mettez à jour **BOOTSTRAP_SERVERS** et **EH_SASL** avec les informations de votre espace de noms. Vous pouvez maintenant commencer la diffusion en continu avec Event Hubs, comme vous le feriez avec Kafka. Pour obtenir l’exemple de code complet, consultez le fichier sparkConsumer.scala sur GitHub. 

```scala
//Read from your Event Hub!
val df = spark.readStream
    .format("kafka")
    .option("subscribe", TOPIC)
    .option("kafka.bootstrap.servers", BOOTSTRAP_SERVERS)
    .option("kafka.sasl.mechanism", "PLAIN")
    .option("kafka.security.protocol", "SASL_SSL")
    .option("kafka.sasl.jaas.config", EH_SASL)
    .option("kafka.request.timeout.ms", "60000")
    .option("kafka.session.timeout.ms", "30000")
    .option("kafka.group.id", GROUP_ID)
    .option("failOnDataLoss", "true")
    .load()

//Use dataframe like normal (in this example, write to console)
val df_write = df.writeStream
    .outputMode("append")
    .format("console")
    .start()
```

## <a name="write-to-event-hubs-for-kafka"></a>Écrire dans Event Hubs pour Kafka
Vous pouvez aussi écrire dans Event Hubs, comme vous le feriez dans Kafka. N’oubliez pas de mettre à jour votre configuration pour modifier **BOOTSTRAP_SERVERS** et **EH_SASL** avec les informations de votre espace de noms Event Hubs.  Pour obtenir l’exemple de code complet, consultez le fichier sparkProducer.scala sur GitHub. 

```scala
df = /**Dataframe**/

//Write to your Event Hub!
df.writeStream
    .format("kafka")
    .option("topic", TOPIC)
    .option("kafka.bootstrap.servers", BOOTSTRAP_SERVERS)
    .option("kafka.sasl.mechanism", "PLAIN")
    .option("kafka.security.protocol", "SASL_SSL")
    .option("kafka.sasl.jaas.config", EH_SASL)
    .option("checkpointLocation", "./checkpoint")
    .start()
```



## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur Event Hubs et sur Event Hubs pour Kafka, consultez les articles suivants :  

- [Mettre en miroir un broker Kafka dans un hub d’événements](event-hubs-kafka-mirror-maker-tutorial.md)
- [Connecter Apache Flink à un hub d’événements](event-hubs-kafka-flink-tutorial.md)
- [Intégrer Kafka Connect à un hub d’événements](event-hubs-kafka-connect-tutorial.md)
- [Explorer des exemples sur GitHub](https://github.com/Azure/azure-event-hubs-for-kafka)
- [Connecter Akka Streams à un hub d’événements](event-hubs-kafka-akka-streams-tutorial.md)
- [Guide du développeur Apache Kafka pour Azure Event Hubs](apache-kafka-developer-guide.md)

