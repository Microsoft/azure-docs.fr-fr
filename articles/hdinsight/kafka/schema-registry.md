---
title: Apache Kafka avec Confluent Schema Registry dans Azure HDInsight
description: Déployez un cluster Kafka géré par HDInsight avec un nœud de périphérie à l’intérieur d’un réseau virtuel, puis installez Confluent Schema Registry sur le nœud de périphérie.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: hrasheed
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/18/2020
ms.openlocfilehash: bb46bc18469638416ff76f84516498e0076c85fd
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95500320"
---
# <a name="apache-kafka-with-confluent-schema-registry-in-azure-hdinsight"></a>Apache Kafka avec Confluent Schema Registry dans Azure HDInsight

Le Registre de schémas Kafka fournit des sérialiseurs qui se connectent à des clients Apache Kafka qui gèrent le stockage et la récupération de schéma de message pour les messages Kafka envoyés au format Avro. Il s’agissait d’un projet OSS de Confluent, mais il est désormais sous [licence de la communauté Confluent](https://www.confluent.io/blog/license-changes-confluent-platform/). Le Registre de schémas sert en outre les intérêts suivants :

* Stocker et récupérer des schémas pour les producteurs et consommateurs.
* Appliquer la compatibilité ascendante/descendante/complète sur les rubriques.
* Réduire la taille de la charge utile envoyée à Kafka.

Dans un cluster Kafka géré par HDInsight, le Registre de schémas est généralement déployé sur un nœud de périphérie pour permettre la séparation du calcul des nœuds principaux.

Vous trouverez ci-dessous une architecture représentative de la façon dont le Registre de schémas est déployé sur un cluster HDInsight. Le Registre de schémas expose en mode natif une API REST pour les opérations qu’il contient.  Les producteurs et consommateurs peuvent interagir avec le Registre de schémas à partir du réseau virtuel ou à l’aide du [proxy REST Kafka](rest-proxy.md).

![Registre de schémas Kafka HDInsight](./media/schema-registry/pic1.png)

## <a name="deploy-an-hdinsight-managed-kafka-with-confluent-schema-registry"></a>Déployer une instance Kafka gérée par HDInsight avec Confluent Schema Registry

Dans cette section, nous déployons un cluster Kafka géré par HDInsight avec un nœud de périphérie à l’intérieur d’un réseau virtuel, puis installons Confluent Schema Registry sur le nœud de périphérie.  

1. Sélectionnez le bouton **Déployer sur Azure** ci-dessous pour vous connecter à Azure et ouvrir le modèle Resource Manager.

    [![Déployer sur Azure](./media/schema-registry/hdi-deploy-to-azure1.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Farnabganguly%2FKafkaschemaregistry%2Fmaster%2Fazuredeploy.json)

1. Dans le modèle de déploiement personnalisé, renseignez les champs comme indiqué ci-dessous :

    |Propriété |Description |
    |---|---|
    |Abonnement|Dans la liste déroulante, sélectionnez l’abonnement Azure utilisé pour le cluster.|
    |Resource group|Dans la liste déroulante, sélectionnez votre groupe de ressources existant ou **Créer**.|
    |Région|Dans la liste déroulante, sélectionnez une région dans laquelle le cluster est créé.|
    |Nom du cluster|Entrez un nom globalement unique. Vous pouvez aussi conserver le nom par défaut.|
    |Nom d’utilisateur de connexion au cluster|Indiquez le nom d’utilisateur, la valeur par défaut est **Administrateur**.|
    |Mot de passe de connexion au cluster|Indiquez le mot de passe.|
    |Nom d’utilisateur SSH|Indiquez le nom d’utilisateur. La valeur par défaut est **sshuser**.|
    |Mot de passe SSH|Indiquez le mot de passe.|

    Laissez les autres champs tels quels. Sélectionnez **Vérifier + créer** pour continuer.

1. Passez en revue les détails du déploiement, puis sélectionnez **Créer** pour initialiser le déploiement. Le déploiement peut prendre jusqu’à 45 minutes.

## <a name="configure-the-confluent-schema-registry"></a>Configurer Confluent Schema Registry

Dans cette section, nous allons configurer le Registre de schémas Kafka Confluent que nous avons installé sur le nœud de périphérie. Confluent Schema Registry se trouve dans `/etc/schema-registry/schema-registry.properties`, et les mécanismes permettant de démarrer et d’arrêter les exécutables du service se trouvent dans le dossier `/usr/bin/`.

Le Registre de schémas doit savoir que le service Zookeeper peut interagir avec le cluster Kafka HDInsight. Suivez les étapes ci-dessous pour connaître les détails du quorum Zookeeper.

1. Utilisez la [commande ssh](../hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre nœud de périphérie de cluster. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@schema-registry.CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Configurez une variable de mot de passe. Remplacez PASSWORD par le mot de passe de connexion du cluster, puis entrez la commande :

    ```bash
    export password='PASSWORD'
    ```

1. Extrayez le nom du cluster avec la bonne casse. Exécutez la commande suivante :

    ```bash
    export clusterName=$(curl -u admin:$password -sS -G "http://headnodehost:8080/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')
    ```

1. Extrayez les hôtes Zookeeper Kafka. Exécutez la commande suivante :

    ```bash
    export KAFKAZKHOSTS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2);

    echo $KAFKAZKHOSTS
    ```

    Prenez note de cette valeur, car elle sera utilisée ultérieurement.

1. Extrayez les hôtes du répartiteur Kafka. Exécutez la commande suivante :

    ```bash
    export KAFKABROKERS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2);

    echo $KAFKABROKERS
    ```

1. Ouvrez les fichiers de propriétés du Registre de schémas en mode d’édition. Exécutez la commande suivante :

    ```bash
    sudo nano /etc/schema-registry/schema-registry.properties
    ```

    1. Mettez à jour la valeur de `kafkastore.connection.url` avec la chaîne Zookeeper identifiée précédemment.
    1. Mettez à jour la valeur pour `debug` sur `true`.

    Le fichier de propriétés ressemble maintenant à ce qui suit :  

    ```
    listeners=http://0.0.0.0:8081
    kafkastore.connection.url=zk1-ag4kaf.q2hwzr1xkxjuvobkaagmjjkhta.gx.internal.cloudapp.net:2181,zk2-ag4kaf.q2hwzr1xkxjuvobkaagmjjkhta.gx.internal.cloudapp.net:2181
    kafkastore.topic=_schemas
    debug=true
    ```

1. Pour enregistrer le fichier, appuyez sur **Ctrl+X**, sur **O**, puis sur **Entrée**.

1. Démarrez le Registre de schémas et faites-le pointer pour utiliser le fichier de propriétés du Registre de schémas mis à jour. Exécutez les commandes suivantes :

    ```bash
    cd /bin
    sudo schema-registry-start /etc/schema-registry/schema-registry.properties
    ```

1. Une fois le Registre de schémas exécuté dans une session SSH, lancez une autre fenêtre SSH.  Inscrivez une nouvelle version d’un schéma sous l’objet « Kafka-key » et notez la sortie :

    ```cmd
    curl -X POST -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
        --data '{"schema": "{\"type\": \"string\"}"}'
    ```

    ```output
       HTTP/1.1 200 OK
    Date: Sun, 22 Mar 2020 16:33:04 GMT
    Content-Type: application/vnd.schemaregistry.v1+json
    Content-Length: 9
    Server: Jetty(9.2.24.v20180105)
    ```

1. Inscrivez une nouvelle version d’un schéma sous l’objet « Kafka-value » et notez la sortie :

    ```cmd
    curl -X POST -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
        --data '{"schema": "{\"type\": \"string\"}"}' \
    ```

    ```output
    HTTP/1.1 200 OK
    Date: Sun, 22 Mar 2020 16:34:18 GMT
    Content-Type: application/vnd.schemaregistry.v1+json
    Content-Length: 9
    Server: Jetty(9.2.24.v20180105)
    ```

1. Répertoriez tous les sujets et vérifiez la sortie :

    ```cmd
    curl -X GET -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
        http://localhost:8081/subjects
    ```

    ```output
    HTTP/1.1 200 OK
    Date: Sun, 22 Mar 2020 16:34:39 GMT
    Content-Type: application/vnd.schemaregistry.v1+json
    Content-Length: 27
    Server: Jetty(9.2.24.v20180105)

    ["Kafka-value","Kafka-key"]
    ```

Vous souhaiterez peut-être essayer les autres [commandes avancées répertoriées ici](https://docs.confluent.io/1.0/schema-registry/docs/intro.html#quickstart).

## <a name="send-and-consume-avro-data-from-kafka-using-schema-registry"></a>Envoyer et consommer des données Avro à partir de Kafka à l’aide du Registre de schémas

Dans cette section, nous allons lire les données d’une entrée standard et les écrire dans une rubrique Kafka dans un certain format. Nous allons ensuite lire les données des rubriques Kafka à l’aide d’un consommateur avec un formateur Avro pour transformer ces données en format lisible.

1. Créez une rubrique Kafka `agkafkaschemareg`.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic agkafkaschemareg --zookeeper $KAFKAZKHOSTS
    ```

1. Utilisez le **Producteur de console Kafka Avro** pour créer un schéma, assigner le schéma à la rubrique et commencer à envoyer des données à la rubrique au format Avro. Assurez-vous que la rubrique Kafka de l’étape précédente est correctement créée et que **$KAFKABROKERS** contient une valeur.

    Le schéma que nous envoyons est une clé : Paire de valeurs

    ```
    Value
    {
      "type": "record",
      "name": "example_schema",
      "namespace": "com.example",
      "fields": [
        {
          "name": "cust_id",
          "type": "int",
          "doc": "Id of the customer account"
        },
        {
          "name": "year",
          "type": "int",
          "doc": "year of expense"
        },
        {
          "name": "expenses",
          "type": {
            "type": "array",
            "items": "float"
          },
          "doc": "Expenses for the year"
        }
      ],
      "doc:": "A basic schema for storing messages"
    }
    ```

    Utilisez la commande ci-dessous pour démarrer le **Producteur de console Kafka Avro** :

    ```bash
    /usr/bin/kafka-avro-console-producer     --broker-list $KAFKABROKERS     --topic agkafkaschemareg     --property parse.key=true --property key.schema='{"type" : "int", "name" : "id"}'     --property value.schema='{ "type" : "record", "name" : "example_schema", "namespace" : "com.example", "fields" : [ { "name" : "cust_id", "type" : "int", "doc" : "Id of the customer account" }, { "name" : "year", "type" : "int", "doc" : "year of expense" }, { "name" : "expenses", "type" : {"type": "array", "items": "float"}, "doc" : "Expenses for the year" } ], "doc:" : "A basic schema for storing messages" }'
    ```

1. Lorsque le producteur est prêt à accepter des messages, commencez à envoyer les messages au format de schéma Avro prédéfini. Utilisez la touche Tab pour créer un espacement entre la clé et la valeur.

    ```
    1 TAB {"cust_id":1313131, "year":2012, "expenses":[1313.13, 2424.24]}
    2 TAB {"cust_id":3535353, "year":2011, "expenses":[761.35, 92.18, 14.41]}
    3 TAB {"cust_id":7979797, "year":2011, "expenses":[4489.00]}
    ```

1. Essayez d’entrer des données hors schéma aléatoires dans le producteur de console pour voir comment le producteur autorise à présent toutes les données qui ne sont pas conformes au schéma Avro prédéfini.

    ```
    1       {"cust_id":1313131, "year":2012, "expenses":[1313.13, 2424.24]}
    2       {"cust_id":1313131,"cust_age":34 "year":2012, "expenses":[1313.13, 2424.24,34.212]}
    org.apache.kafka.common.errors.SerializationException: Error deserializing json {"cust_id":1313131,"cust_age":34 "year":2012, "expenses":[1313.13, 2424.24,34.212]} to Avro of schema {"type":"record","name":"example_schema","namespace":"com.example","fields":[{"name":"cust_id","type":"int","doc":"Id of the customer account"},{"name":"year","type":"int","doc":"year of expense"},{"name":"expenses","type":{"type":"array","items":"float"},"doc":"Expenses for the year"}],"doc:":"A basic schema for storing messages"}
    Caused by: org.codehaus.jackson.JsonParseException: Unexpected character ('"' (code 34)): was expecting comma to separate OBJECT entries
     at [Source: java.io.StringReader@3561c410; line: 1, column: 35]
            at org.codehaus.jackson.JsonParser._constructError(JsonParser.java:1433)
            at org.codehaus.jackson.impl.JsonParserMinimalBase._reportError(JsonParserMinimalBase.java:521)
            at org.codehaus.jackson.impl.JsonParserMinimalBase._reportUnexpectedChar(JsonParserMinimalBase.java:442)
            at org.codehaus.jackson.impl.ReaderBasedParser.nextToken(ReaderBasedParser.java:406)
            at org.apache.avro.io.JsonDecoder.getVaueAsTree(JsonDecoder.java:549)
            at org.apache.avro.io.JsonDecoder.doAction(JsonDecoder.java:474)
            at org.apache.avro.io.parsing.Parser.advance(Parser.java:88)
            at org.apache.avro.io.JsonDecoder.advance(JsonDecoder.java:139)
            at org.apache.avro.io.JsonDecoder.readInt(JsonDecoder.java:166)
            at org.apache.avro.io.ValidatingDecoder.readInt(ValidatingDecoder.java:83)
            at org.apache.avro.generic.GenericDatumReader.readInt(GenericDatumReader.java:511)
            at org.apache.avro.generic.GenericDatumReader.readWithoutConversion(GenericDatumReader.java:182)
            at org.apache.avro.generic.GenericDatumReader.read(GenericDatumReader.java:152)
            at org.apache.avro.generic.GenericDatumReader.readField(GenericDatumReader.java:240)
            at org.apache.avro.generic.GenericDatumReader.readRecord(GenericDatumReader.java:230)
            at org.apache.avro.generic.GenericDatumReader.readWithoutConversion(GenericDatumReader.java:174)
            at org.apache.avro.generic.GenericDatumReader.read(GenericDatumReader.java:152)
            at org.apache.avro.generic.GenericDatumReader.read(GenericDatumReader.java:144)
            at io.confluent.kafka.formatter.AvroMessageReader.jsonToAvro(AvroMessageReader.java:213)
            at io.confluent.kafka.formatter.AvroMessageReader.readMessage(AvroMessageReader.java:200)
            at kafka.tools.ConsoleProducer$.main(ConsoleProducer.scala:59)
            at kafka.tools.ConsoleProducer.main(ConsoleProducer.scala)
    
    ```

1. Sur un autre écran, démarrez le consommateur de console Kafka Avro

    ```bash
    sudo /usr/bin/kafka-avro-console-consumer --bootstrap-server $KAFKABROKERS --topic agkafkaschemareg --from-beginning
    ```

    Vous devriez commencer à voir la sortie ci-dessous :

    ```output
    {"cust_id":1313131,"year":2012,"expenses":[1313.13,2424.24]}
    {"cust_id":7979797,"year":2011,"expenses":[4489.0]}
    {"cust_id":3535353,"year":2011,"expenses":[761.35,92.18,14.41]}
    ```

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser les API de producteur et de consommateur Apache Kafka](apache-kafka-producer-consumer-api.md)