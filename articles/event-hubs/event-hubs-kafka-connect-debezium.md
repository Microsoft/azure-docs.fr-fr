---
title: Intégrer Apache Kafka Connect sur Azure Event Hubs (version préliminaire) avec Debezium pour la capture des changements de données
description: Cet article fournit des informations sur l’utilisation de Debezium avec Azure Event Hubs pour Kafka.
ms.topic: how-to
author: abhirockzz
ms.author: abhishgu
ms.date: 08/11/2020
ms.openlocfilehash: ae3ef2e1f35be432558769c512845543867ef27a
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505407"
---
# <a name="integrate-apache-kafka-connect-support-on-azure-event-hubs-preview-with-debezium-for-change-data-capture"></a>Intégrer la prise en charge d’Apache Kafka Connect sur Azure Event Hubs (version préliminaire) avec Debezium pour la capture des changements de données

**La capture des changements de données (CDC)** est une technique utilisée pour effectuer le suivi des modifications au niveau des lignes dans les tables de base de données en réponse à des opérations de création, de mise à jour et de suppression. [Debezium](https://debezium.io/) est une plateforme distribuée qui s’appuie sur les fonctionnalités de capture des changements de données disponibles dans différentes bases de données (par exemple, le [décodage logique dans PostgreSQL](https://www.postgresql.org/docs/current/static/logicaldecoding-explanation.html)). Il fournit un ensemble de [connecteurs Kafka Connect](https://debezium.io/documentation/reference/1.2/connectors/index.html) qui utilisent les modifications au niveau des lignes dans la ou les tables de base de données et les convertissent en flux d’événements qui sont ensuite envoyés à [Apache Kafka](https://kafka.apache.org/).

Ce didacticiel vous guide tout au long de la configuration d’un système basé sur la capture des changements de données sur Azure à l’aide d' [Azure Event Hubs](./event-hubs-about.md?WT.mc_id=devto-blog-abhishgu) (pour Kafka), d’[Azure DB pour PostgreSQL](../postgresql/overview.md) et de Debezium. Il utilise le [connecteur Debezium PostgreSQL](https://debezium.io/documentation/reference/1.2/connectors/postgresql.html) pour diffuser en continu les modifications de base de données de PostgreSQL vers les rubriques Kafka dans Azure Event Hubs

> [!NOTE]
> Cet article contient des références au terme *liste verte*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

Dans ce tutoriel, vous effectuez les étapes suivantes :

> [!div class="checklist"]
> * Créer un espace de noms Event Hubs
> * Installer et configurer Azure Database pour PostgreSQL
> * Configurer et exécuter Kafka Connect avec le connecteur Debezium PostgreSQL
> * Tester la capture des changements de données
> * (Facultatif) Utiliser des événements de changement de données avec un connecteur `FileStreamSink`

## <a name="pre-requisites"></a>Conditions préalables
Pour effectuer cette procédure, vous aurez besoin des éléments suivants :

- Abonnement Azure. Si vous n’en avez pas, [créez un compte gratuit](https://azure.microsoft.com/free/).
- Linux/MacOS
- Version de Kafka (version 1.1.1, version 2.11 de Scala), disponible depuis [kafka.apache.org](https://kafka.apache.org/downloads#1.1.1)
- Lisez l’article d’introduction [Event Hubs pour Apache Kafka](./event-hubs-for-kafka-ecosystem-overview.md)

## <a name="create-an-event-hubs-namespace"></a>Créer un espace de noms Event Hubs
Un espace de noms Event Hubs est requis pour échanger avec tout service Event Hubs. Pour obtenir des instructions sur la création d'un espace de noms et d'un Event Hub, consultez [Créer un Event Hub](event-hubs-create.md). Obtenez la chaîne de connexion Event Hubs et le nom de domaine complet (FQDN) pour une utilisation ultérieure. Pour obtenir des instructions, consultez [Obtenir une chaîne de connexion Event Hubs](event-hubs-get-connection-string.md). 

## <a name="setup-and-configure-azure-database-for-postgresql"></a>Installer et configurer Azure Database pour PostgreSQL
[Azure Database pour PostgreSQL](../postgresql/overview.md) est un service de base de données relationnelle basé sur la version de la communauté du moteur de base de données PostgreSQL open source, et il est disponible en deux options de déploiement : Serveur unique et Hyperscale (Citus). [Procédez comme suit](../postgresql/quickstart-create-server-database-portal.md) pour créer un serveur Azure Database pour PostgreSQL à l’aide du Portail Azure. 

## <a name="setup-and-run-kafka-connect"></a>Configurer et exécuter Kafka Connect
Cette section aborde les sujets suivants :

- Installation du connecteur Debezium
- Configuration de Kafka Connect pour Event Hubs
- Démarrer le cluster Kafka Connect avec le connecteur Debezium

### <a name="download-and-setup-debezium-connector"></a>Télécharger et configurer le connecteur Debezium
Suivez les dernières instructions de la [documentation Debezium](https://debezium.io/documentation/reference/1.2/connectors/postgresql.html#postgresql-deploying-a-connector) pour télécharger et configurer le connecteur.

- Téléchargez l’archive du plug-in du connecteur. Par exemple, pour télécharger la version `1.2.0` du connecteur, utilisez ce lien : https://repo1.maven.org/maven2/io/debezium/debezium-connector-postgres/1.2.0.Final/debezium-connector-postgres-1.2.0.Final-plugin.tar.gz
- Extrayez les fichiers JAR et copiez-les dans le [plugin.path de Kafka Connect](https://kafka.apache.org/documentation/#connectconfigs).


### <a name="configure-kafka-connect-for-event-hubs"></a>Configurer Kafka Connect pour Event Hubs
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
> Remplacez `{YOUR.EVENTHUBS.CONNECTION.STRING}` par la chaîne de connexion de votre espace de noms Event Hubs. Pour savoir comment obtenir la chaîne de connexion, consultez [Obtenir une chaîne de connexion Event Hubs](event-hubs-get-connection-string.md). Voici un exemple de configuration : `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`.


### <a name="run-kafka-connect"></a>Exécuter Kafka Connect
Dans cette étape, un Worker Kafka Connect est démarré en local en mode distribué, à l’aide d’Event Hubs pour maintenir l’état du cluster.

1. Enregistrez le fichier `connect-distributed.properties` ci-dessus en local.  Veillez à remplacer toutes les valeurs entre accolades.
2. Accédez à l’emplacement de la version de Kafka sur votre ordinateur.
3. Exécutez `./bin/connect-distributed.sh /PATH/TO/connect-distributed.properties` et attendez que le cluster démarre.

> [!NOTE]
> Kafka Connect utilise l’API Kafka AdminClient pour créer automatiquement des rubriques avec les configurations recommandées, y compris le compactage. Une vérification rapide de l’espace de noms dans le portail Azure révèle que les rubriques internes du Worker Connect ont été créées automatiquement.
>
> Les rubriques internes Kafka Connect **doivent utiliser le compactage**.  L’équipe Event Hubs n’est pas responsable de la résolution des configurations incorrectes si les rubriques Connect internes ne sont pas correctement configurées.

### <a name="configure-and-start-the-debezium-postgresql-source-connector"></a>Configurer et démarrer le connecteur Debezium PostgreSQL source

Créez un fichier de configuration (`pg-source-connector.json`) pour le connecteur source PostgreSQL : remplacez les valeurs comme pour votre instance Azure PostgreSQL.

```json
{
    "name": "todo-connector",
    "config": {
        "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
        "database.hostname": "<replace with Azure PostgreSQL instance name>.postgres.database.azure.com",
        "database.port": "5432",
        "database.user": "<replace with database user name>",
        "database.password": "<replace with database password>",
        "database.dbname": "postgres",
        "database.server.name": "my-server",
        "plugin.name": "wal2json",
        "table.whitelist": "public.todos"
    }
}
```

> [!TIP]
> L’attribut `database.server.name` est un nom logique qui identifie et fournit un espace de noms pour le serveur de base de données ou le cluster PostgreSQL en cours d’analyse. Pour obtenir des informations détaillées, consultez [Documentation de Debezium](https://debezium.io/documentation/reference/1.2/connectors/postgresql.html#postgresql-property-database-server-name)

Pour créer une instance du connecteur, utilisez le point de terminaison de l’API REST Kafka Connect :

```bash
curl -X POST -H "Content-Type: application/json" --data @pg-source-connector.json http://localhost:8083/connectors
```

Pour vérifier l’état du connecteur :

```bash
curl -s http://localhost:8083/connectors/todo-connector/status
```

## <a name="test-change-data-capture"></a>Tester la capture des changements de données
Pour voir la capture des changements de données en action, vous devrez créer/mettre à jour/supprimer des enregistrements dans la base de données Azure PostgreSQL.

Commencez par vous connecter à votre base de données Azure PostgreSQL (l’exemple ci-dessous utilise [psql](https://www.postgresql.org/docs/12/app-psql.html))

```bash
psql -h <POSTGRES_INSTANCE_NAME>.postgres.database.azure.com -p 5432 -U <POSTGRES_USER_NAME> -W -d <POSTGRES_DB_NAME> --set=sslmode=require

e.g. 

psql -h my-postgres.postgres.database.azure.com -p 5432 -U testuser@my-postgres -W -d postgres --set=sslmode=require
```

**Créer une table et insérer des enregistrements**

```sql
CREATE TABLE todos (id SERIAL, description VARCHAR(50), todo_status VARCHAR(10), PRIMARY KEY(id));

INSERT INTO todos (description, todo_status) VALUES ('setup postgresql on azure', 'complete');
INSERT INTO todos (description, todo_status) VALUES ('setup kafka connect', 'complete');
INSERT INTO todos (description, todo_status) VALUES ('configure and install connector', 'in-progress');
INSERT INTO todos (description, todo_status) VALUES ('start connector', 'pending');
```

Le connecteur doit maintenant entrer en action et envoyer des événements de changement des données à une rubrique Event Hubs avec le `my-server.public.todos` suivant, en supposant que vous avez `my-server` comme valeur pour `database.server.name` et que `public.todos` est la table dont vous effectuez le suivi des modifications (comme dans la configuration de `table.whitelist`).

**Consulter la rubrique Check Event Hubs**

Examinons le contenu de la rubrique pour s’assurer que tout fonctionne comme prévu. L’exemple ci-dessous utilise [`kafkacat`](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/kafkacat), mais vous pouvez également [créer un contrôle serveur consommateur à l’aide de l’une des options répertoriées ici](apache-kafka-developer-guide.md)

Créez un fichier nommé `kafkacat.conf` avec le contenu suivant :

```
metadata.broker.list=<enter event hubs namespace>.servicebus.windows.net:9093
security.protocol=SASL_SSL
sasl.mechanisms=PLAIN
sasl.username=$ConnectionString
sasl.password=<enter event hubs connection string>
```

> [!NOTE]
> Mettez à jour les attributs `metadata.broker.list` et `sasl.password` dans `kafkacat.conf` en fonction des informations d’Event Hubs. 

Dans un autre terminal, démarrez un contrôle serveur consommateur :

```bash
export KAFKACAT_CONFIG=kafkacat.conf
export BROKER=<enter event hubs namespace>.servicebus.windows.net:9093
export TOPIC=my-server.public.todos

kafkacat -b $BROKER -t $TOPIC -o beginning
```

Vous devez voir les charges utiles JSON représentant les événements de changement des données générés dans PostgreSQL en réponse aux lignes que vous venez d’ajouter à la table `todos`. Voici un extrait de la charge utile :


```json
{
    "schema": {...},
    "payload": {
        "before": null,
        "after": {
            "id": 1,
            "description": "setup postgresql on azure",
            "todo_status": "complete"
        },
        "source": {
            "version": "1.2.0.Final",
            "connector": "postgresql",
            "name": "fullfillment",
            "ts_ms": 1593018069944,
            "snapshot": "last",
            "db": "postgres",
            "schema": "public",
            "table": "todos",
            "txId": 602,
            "lsn": 184579736,
            "xmin": null
        },
        "op": "c",
        "ts_ms": 1593018069947,
        "transaction": null
    }
```

L’événement se compose de `payload` , ainsi que de son `schema` (omis par souci de concision). Dans la section `payload`, notez comment l’opération de création (`"op": "c"`) est représentée : `"before": null` signifie qu’il s’agit d’une nouvelle ligne insérée (`INSERT`), `after` fournit des valeurs pour les colonnes de la ligne, `source` fournit les métadonnées de l’instance PostgreSQL à partir desquelles cet événement a été récupéré, etc.

Vous pouvez également essayer les mêmes opérations de mise à jour ou de suppression, et examiner les événements de changement des données. Par exemple, pour mettre à jour l’état de la tâche pour `configure and install connector` (en supposant que son `id` est `3`) :

```sql
UPDATE todos SET todo_status = 'complete' WHERE id = 3;
```

## <a name="optional-install-filestreamsink-connector"></a>(Facultatif) Installer le connecteur FileStreamSink
Maintenant que toutes les modifications apportées à la table `todos` sont capturées dans la rubrique Event Hubs, nous allons utiliser le connecteur FileStreamSink (disponible par défaut dans Kafka Connect) pour utiliser ces événements.

Créez un fichier de configuration (`file-sink-connector.json`) pour le connecteur : remplacez l’attribut `file` par votre système de fichiers.

```json
{
    "name": "cdc-file-sink",
    "config": {
        "connector.class": "org.apache.kafka.connect.file.FileStreamSinkConnector",
        "tasks.max": "1",
        "topics": "my-server.public.todos",
        "file": "<enter full path to file e.g. /Users/foo/todos-cdc.txt>"
    }
}
```

Pour créer le connecteur et vérifier son état :

```bash
curl -X POST -H "Content-Type: application/json" --data @file-sink-connector.json http://localhost:8083/connectors

curl http://localhost:8083/connectors/cdc-file-sink/status
```

Insérez/mettez à jour/supprimez des enregistrements de base de données et surveillez les enregistrements dans le fichier récepteur de sortie configuré :

```bash
tail -f /Users/foo/todos-cdc.txt
```


## <a name="cleanup"></a>Nettoyage
Kafka Connect crée des rubriques Event Hub pour stocker les configurations, les décalages et l’état qui persistent même après l’arrêt du cluster Connect. À moins que cette persistance ne soit souhaitée, il est recommandé de supprimer ces rubriques. Vous pouvez également supprimer l’Event Hub `my-server.public.todos` qui a été créé au cours de cette procédure pas à pas.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations sur Event Hubs pour Kafka, consultez les articles suivants :  

- [Mettre en miroir un broker Kafka dans un hub d’événements](event-hubs-kafka-mirror-maker-tutorial.md)
- [Connecter Apache Spark à un hub d’événements](event-hubs-kafka-spark-tutorial.md)
- [Connecter Apache Flink à un hub d’événements](event-hubs-kafka-flink-tutorial.md)
- [Explorer des exemples sur GitHub](https://github.com/Azure/azure-event-hubs-for-kafka)
- [Connecter Akka Streams à un hub d’événements](event-hubs-kafka-akka-streams-tutorial.md)
- [Guide du développeur Apache Kafka pour Azure Event Hubs](apache-kafka-developer-guide.md)