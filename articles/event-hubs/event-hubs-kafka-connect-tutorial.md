---
title: Intégrer à Apache Kafka Connect - Azure Event Hubs | Microsoft Docs
description: Cet article fournit des informations sur l’utilisation de Kafka Connect avec Azure Event Hubs pour Kafka.
ms.topic: how-to
ms.date: 06/23/2020
ms.openlocfilehash: d37d2465d9389a0bcfaabdec32bad0c86846cfb2
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92369537"
---
# <a name="integrate-apache-kafka-connect-support-on-azure-event-hubs-preview"></a>Intégrer la prise en charge Apache Kafka Connect à Azure Event Hubs (préversion)
Si l’ingestion des besoins métier augmente, les exigences augmentent également pour ingérer différents récepteurs et sources externes. [Apache Kafka Connect](https://kafka.apache.org/documentation/#connect) fournit cette infrastructure pour se connecter et importer/exporter des données depuis/vers n’importe quel système externe, tel que MySQL, HDFS, et système de fichiers via un cluster Kafka. Ce tutoriel vous montre comment utiliser l’infrastructure Kafka Connect avec des Event Hubs.

Ce tutoriel vous guide tout au long de l’intégration de Kafka Connect avec un hub d’événements et du déploiement des connecteurs FileStreamSource et FileStreamSink de base. Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire. Bien que ces connecteurs ne soient pas destinés à une utilisation en production, ils présentent un scénario Kafka Connect de bout en bout où Azure Event Hubs agit comme un répartiteur Kafka.

> [!NOTE]
> Cet exemple est disponible sur [GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/connect).

Dans ce tutoriel, vous effectuez les étapes suivantes :

> [!div class="checklist"]
> * Créer un espace de noms Event Hubs
> * Cloner l’exemple de projet
> * Configurer Kafka Connect pour Event Hubs
> * Exécuter Kafka Connect
> * Créer des connecteurs

## <a name="prerequisites"></a>Prérequis
Pour suivre cette procédure pas à pas, vérifiez que les conditions préalables ci-dessous sont bien remplies :

- Abonnement Azure. Si vous n’en avez pas, [créez un compte gratuit](https://azure.microsoft.com/free/).
- [Git](https://www.git-scm.com/downloads)
- Linux/MacOS
- Version de Kafka (version 1.1.1, version 2.11 de Scala), disponible depuis [kafka.apache.org](https://kafka.apache.org/downloads#1.1.1)
- Lisez l’article d’introduction [Event Hubs pour Apache Kafka](./event-hubs-for-kafka-ecosystem-overview.md)

## <a name="create-an-event-hubs-namespace"></a>Créer un espace de noms Event Hubs
Un espace de noms Event Hubs est requis pour échanger avec tout service Event Hubs. Pour obtenir des instructions sur la création d'un espace de noms et d'un Event Hub, consultez [Créer un Event Hub](event-hubs-create.md). Obtenez la chaîne de connexion Event Hubs et le nom de domaine complet (FQDN) pour une utilisation ultérieure. Pour obtenir des instructions, consultez [Obtenir une chaîne de connexion Event Hubs](event-hubs-get-connection-string.md). 

## <a name="clone-the-example-project"></a>Cloner l’exemple de projet
Clonez le référentiel Azure Event Hubs et accédez au sous-dossier Tutoriels/Se connecter : 

```
git clone https://github.com/Azure/azure-event-hubs-for-kafka.git
cd azure-event-hubs-for-kafka/tutorials/connect
```

## <a name="configure-kafka-connect-for-event-hubs"></a>Configurer Kafka Connect pour Event Hubs
Une reconfiguration minimale est nécessaire pour rediriger le débit Kafka Connect de Kafka vers Event Hubs.  L’exemple `connect-distributed.properties` suivant montre comment configurer Connect pour s’authentifier et communiquer avec le point de terminaison Kafka sur Event Hubs :

```properties
bootstrap.servers={YOUR.EVENTHUBS.FQDN}:9093 # e.g. namespace.servicebus.windows.net:9093
group.id=connect-cluster-group

# connect internal topic names, auto-created if not exists
config.storage.topic=connect-cluster-configs
offset.storage.topic=connect-cluster-offsets
status.storage.topic=connect-cluster-status

# internal topic replication factors - auto 3x replication in Azure Storage
config.storage.replication.factor=1
offset.storage.replication.factor=1
status.storage.replication.factor=1

rest.advertised.host.name=connect
offset.flush.interval.ms=10000

key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
internal.key.converter=org.apache.kafka.connect.json.JsonConverter
internal.value.converter=org.apache.kafka.connect.json.JsonConverter

internal.key.converter.schemas.enable=false
internal.value.converter.schemas.enable=false

# required EH Kafka security settings
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";

producer.security.protocol=SASL_SSL
producer.sasl.mechanism=PLAIN
producer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";

consumer.security.protocol=SASL_SSL
consumer.sasl.mechanism=PLAIN
consumer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";

plugin.path={KAFKA.DIRECTORY}/libs # path to the libs directory within the Kafka release
```

> [!IMPORTANT]
> Remplacez `{YOUR.EVENTHUBS.CONNECTION.STRING}` par la chaîne de connexion de votre espace de noms Event Hubs. Pour savoir comment obtenir la chaîne de connexion, consultez [Obtenir une chaîne de connexion Event Hubs](event-hubs-get-connection-string.md). Voici un exemple de configuration : `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`


## <a name="run-kafka-connect"></a>Exécuter Kafka Connect

Dans cette étape, un Worker Kafka Connect est démarré en local en mode distribué, à l’aide d’Event Hubs pour maintenir l’état du cluster.

1. Enregistrez le fichier `connect-distributed.properties` ci-dessus en local.  Veillez à remplacer toutes les valeurs entre accolades.
2. Accédez à l’emplacement de la version de Kafka sur votre ordinateur.
4. Exécutez `./bin/connect-distributed.sh /PATH/TO/connect-distributed.properties`.  L’API REST du Worker Connect est prêt pour l’interaction lorsque vous voyez `'INFO Finished starting connectors and tasks'`. 

> [!NOTE]
> Kafka Connect utilise l’API Kafka AdminClient pour créer automatiquement des rubriques avec les configurations recommandées, y compris le compactage. Une vérification rapide de l’espace de noms dans le portail Azure révèle que les rubriques internes du Worker Connect ont été créées automatiquement.
>
>Les rubriques internes Kafka Connect **doivent utiliser le compactage**.  L’équipe Event Hubs n’est pas responsable de la résolution des configurations incorrectes si les rubriques Connect internes ne sont pas correctement configurées.

### <a name="create-connectors"></a>Créer des connecteurs
Cette section vous guide tout au long de l’ajout des connecteurs FileStreamSource et FileStreamSink. 

1. Créez un répertoire pour les fichiers de données d’entrée et de sortie.
    ```bash
    mkdir ~/connect-quickstart
    ```

2. Créez deux fichiers : un avec les données de départ à partir duquel le connecteur FileStreamSource lit, et un autre dans lequel notre connecteur FileStreamSink écrit.
    ```bash
    seq 1000 > ~/connect-quickstart/input.txt
    touch ~/connect-quickstart/output.txt
    ```

3. Créez un connecteur FileStreamSource.  Veillez à remplacer les accolades par le chemin d’accès de votre répertoire de base.
    ```bash
    curl -s -X POST -H "Content-Type: application/json" --data '{"name": "file-source","config": {"connector.class":"org.apache.kafka.connect.file.FileStreamSourceConnector","tasks.max":"1","topic":"connect-quickstart","file": "{YOUR/HOME/PATH}/connect-quickstart/input.txt"}}' http://localhost:8083/connectors
    ```
    Vous devriez voir l’Event Hubs `connect-quickstart` sur votre instance Event Hubs après l’exécution de la commande ci-dessus.
4. Vérifiez l’état du connecteur source.
    ```bash
    curl -s http://localhost:8083/connectors/file-source/status
    ```
    Si vous le souhaitez, vous pouvez utiliser l’[Explorateur Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/releases) pour vérifier que les événements sont arrivés dans la rubrique `connect-quickstart`.

5. Créez un connecteur FileStreamSink.  Veillez de nouveau à remplacer les accolades par le chemin d’accès de votre répertoire de base.
    ```bash
    curl -X POST -H "Content-Type: application/json" --data '{"name": "file-sink", "config": {"connector.class":"org.apache.kafka.connect.file.FileStreamSinkConnector", "tasks.max":"1", "topics":"connect-quickstart", "file": "{YOUR/HOME/PATH}/connect-quickstart/output.txt"}}' http://localhost:8083/connectors
    ```
 
6. Vérifiez l’état du connecteur récepteur.
    ```bash
    curl -s http://localhost:8083/connectors/file-sink/status
    ```

7. Vérifiez que les données ont été répliquées entre les fichiers et que les données sont identiques entre les deux fichiers.
    ```bash
    # read the file
    cat ~/connect-quickstart/output.txt
    # diff the input and output files
    diff ~/connect-quickstart/input.txt ~/connect-quickstart/output.txt
    ```

### <a name="cleanup"></a>Nettoyage
Kafka Connect crée des rubriques Event Hub pour stocker les configurations, les décalages et l’état qui persistent même après l’arrêt du cluster Connect. À moins que cette persistance ne soit souhaitée, il est recommandé de supprimer ces rubriques. Vous pouvez également supprimer l’Event Hub `connect-quickstart` qui a été créé au cours de cette procédure pas à pas.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations sur Event Hubs pour Kafka, consultez les articles suivants :  

- [Mettre en miroir un broker Kafka dans un hub d’événements](event-hubs-kafka-mirror-maker-tutorial.md)
- [Connecter Apache Spark à un hub d’événements](event-hubs-kafka-spark-tutorial.md)
- [Connecter Apache Flink à un hub d’événements](event-hubs-kafka-flink-tutorial.md)
- [Explorer des exemples sur GitHub](https://github.com/Azure/azure-event-hubs-for-kafka)
- [Connecter Akka Streams à un hub d’événements](event-hubs-kafka-akka-streams-tutorial.md)
- [Guide du développeur Apache Kafka pour Azure Event Hubs](apache-kafka-developer-guide.md)
