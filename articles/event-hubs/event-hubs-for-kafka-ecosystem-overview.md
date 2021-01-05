---
title: Utiliser un hub d’événements à partir de l’application Apache Kafka - Azure Event Hubs | Microsoft Docs
description: Cet article fournit des informations sur la prise en charge d’Apache Kafka par Azure Event Hubs.
ms.topic: article
ms.date: 09/25/2020
ms.openlocfilehash: d9aa8af30d5ef5e1a985e4d73a9d4a8921ac7d45
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92369588"
---
# <a name="use-azure-event-hubs-from-apache-kafka-applications"></a>Utiliser Azure Event Hubs à partir d’applications Apache Kafka
Event Hubs fournit un point de terminaison compatible avec les API de producteur et de consommateur Apache Kafka® que la plupart des applications clientes Apache Kafka existantes peuvent utiliser comme alternative à l'exécution de votre propre cluster Apache Kafka. Event Hubs prend en charge les clients d'API de producteur et de consommateur Apache Kafka à partir de la version 1.0.

> [!VIDEO https://www.youtube.com/embed/UE1WgB96_fc]

## <a name="what-does-event-hubs-for-kafka-provide"></a>Que fournit Event Hubs pour Kafka ?

La fonctionnalité Event Hubs pour Apache Kafka fournit au-dessus d'Azure Event Hubs une tête de protocole compatible avec les clients Apache Kafka créés pour les versions 1.0 et ultérieures du serveur Apache Kafka, et prend à la fois en charge la lecture et l'écriture sur les hubs d'événements, qui équivalent aux rubriques d'Apache Kafka. 

Vous avez souvent la possibilité d'utiliser le point de terminaison Event Hubs Kafka à partir de vos applications, sans modifier le code par rapport à votre configuration Kafka existante, et vous contenter de modifier la configuration : Mettez à jour la chaîne de connexion dans les configurations pour qu'elle pointe vers le point de terminaison Kafka exposé par votre hub d'événements au lieu de pointer vers votre cluster Kafka. Vous pouvez ensuite commencer à diffuser des événements en continu à partir de vos applications qui utilisent le protocole Kafka dans Event Hubs. 

D'un point de vue conceptuel, Kafka et Event Hubs sont très semblables : ce sont tous les deux des journaux partitionnés conçus pour la diffusion de données en continu, le client contrôlant la partie à lire dans le journal conservé. Le tableau suivant mappe les concepts entre Kafka et Event Hubs.

### <a name="kafka-and-event-hub-conceptual-mapping"></a>Mappage des concepts Kafka et Event Hubs

| Concept Kafka | Concept Event Hubs|
| --- | --- |
| Cluster | Espace de noms |
| Rubrique | Event Hub |
| Partition | Partition|
| Groupe de consommateurs | Groupe de consommateurs |
| Offset | Offset|

### <a name="key-differences-between-apache-kafka-and-event-hubs"></a>Principales différences entre Apache Kafka et Event Hubs

[Apache Kafka](https://kafka.apache.org/) est un logiciel que vous devez généralement installer pour l'utiliser, tandis qu'Event Hubs est un service natif Cloud complètement managé. Il n'y a aucun serveur, disque ou réseau à gérer et à surveiller, ni aucun répartiteur à configurer. Vous créez un espace de noms, qui est un point de terminaison avec un nom de domaine complet, puis vous créez des hubs d'événements (rubriques) dans cet espace de noms. 

Pour plus d’informations sur les espaces de noms et Event Hubs, consultez [Fonctionnalités Event Hubs](event-hubs-features.md#namespace). En tant que service cloud, Event Hubs utilise une seule adresse IP virtuelle stable comme point de terminaison, les clients n’ont donc pas besoin de connaître les répartiteurs ou machines dans un cluster. Même si Event Hubs implémente le même protocole, cette différence signifie que l'ensemble du trafic Kafka de toutes les partitions est acheminé de manière prévisible via ce point de terminaison ; l'accès à un pare-feu n'est donc pas nécessaire pour les répartiteurs d'un cluster.   

Dans Event Hubs, la mise à l'échelle dépend du nombre d'unités de débit que vous achetez, chaque unité de débit vous donnant droit à 1 mégaoctet par seconde, ou à 1 000 événements par seconde en entrée et au double de ce volume en sortie. Lorsque vous atteignez la limite de débit, Event Hubs peut automatiquement effectuer un scale-up des unités de débit si vous utilisez la fonction [Majoration automatique](event-hubs-auto-inflate.md) ; cette fonctionnalité est également compatible avec la prise en charge du protocole Apache Kafka.  

### <a name="is-apache-kafka-the-right-solution-for-your-workload"></a>Apache Kafka convient-il à votre charge de travail ?

Après avoir créé des applications à l'aide d'Apache Kafka, il vous sera également utile de savoir qu'Azure Event Hubs fait partie d'un ensemble de services qui englobe [Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) et [Azure Event Grid](../event-grid/overview.md). 

Certains fournisseurs de distributions commerciales d'Apache Kafka présentent Apache Kafka comme un guichet unique pour tous vos besoins en matière de plateforme de messagerie. Mais en réalité, Apache Kafka n'implémente pas, par exemple, le modèle de file d'attente [consommateur simultané](/azure/architecture/patterns/competing-consumers), ne prend pas en charge le mode [publication/abonnement](/azure/architecture/patterns/publisher-subscriber) à un niveau qui permet aux abonnés d'accéder aux messages entrants sur la base de règles évaluées par le serveur autres que les décalages simples, et il ne dispose d'aucune fonctionnalité pour suivre le cycle de vie d'un travail initié par un message ou pour mettre les messages défectueux de côté dans une file d'attente de lettres mortes, or ces fonctionnalités sont toutes fondamentales pour de nombreux scénarios de messagerie d'entreprise.

Pour comprendre les différences entre les modèles et déterminer quel modèle est le mieux couvert par tel ou tel service, consultez l'aide relative aux [Options de messagerie asynchrone dans Azure](/azure/architecture/guide/technology-choices/messaging). En tant qu'utilisateur d'Apache Kafka, vous constaterez peut-être que les chemins de communication que vous avez jusqu'à présent réalisés avec Kafka peuvent être réalisés avec une complexité de base bien moindre et des fonctionnalités encore plus puissantes en utilisant Event Grid ou Service Bus. 

Si vous avez besoin de fonctionnalités Apache Kafka spécifiques non disponibles via l'interface d'Event Hubs pour Apache Kafka, ou si votre modèle d'implémentation dépasse les [quotas d'Event Hubs](event-hubs-quotas.md), vous pouvez également exécuter un [cluster Apache Kafka natif dans Azure HDInsight](../hdinsight/kafka/apache-kafka-introduction.md).  

## <a name="security-and-authentication"></a>Sécurité et authentification
Chaque fois que vous publiez ou utilisez des événements à partir d'une instance Event Hubs pour Kafka, votre client tente d'accéder aux ressources Event Hubs. Vous voulez assurer l'accès aux ressources à l’aide d’une entité autorisée. Lorsque vous utilisez le protocole Apache Kafka avec vos clients, vous pouvez utiliser des mécanismes SASL pour définir votre configuration à des fins d’authentification et de chiffrement. L’utilisation d'Event Hubs pour Kafka nécessite le chiffrement TLS (car toutes les données en transit avec Event Hubs sont chiffrées avec le protocole TLS). Pour ce faire, vous pouvez spécifier l'option SASL_SSL dans votre fichier de configuration. 

Azure Event Hubs propose plusieurs options afin d'autoriser l’accès à vos ressources sécurisées. 

- OAuth 2.0
- Signature d’accès partagé (SAP)

#### <a name="oauth-20"></a>OAuth 2.0
Event Hubs s'intègre à Azure Active Directory (Azure AD), qui fournit un serveur d'autorisation centralisé compatible avec **OAuth 2.0**. Avec Azure AD, vous pouvez utiliser le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour accorder des autorisations affinées à vos identités clientes. Vous pouvez utiliser cette fonctionnalité avec vos clients Kafka en spécifiant **SASL_SSL** pour le protocole et **OAUTHBEARER** pour le mécanisme. Pour plus d’informations sur les niveaux et les rôles Azure pour limiter l’accès, consultez [Autoriser l’accès avec Azure AD](authorize-access-azure-active-directory.md).

```properties
bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
security.protocol=SASL_SSL
sasl.mechanism=OAUTHBEARER
sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required;
sasl.login.callback.handler.class=CustomAuthenticateCallbackHandler;
```

#### <a name="shared-access-signature-sas"></a>Signature d’accès partagé (SAS)
Event Hubs fournit également les **signatures d'accès partagé (SAP)** à des fins d'accès délégué aux ressources Event Hubs pour Kafka. Autoriser l’accès à l’aide d’un mécanisme basé sur les jetons OAuth 2.0 offre une sécurité et une facilité d'utilisation supérieures par rapport aux SAP. Grâce aux rôles intégrés, il n'est plus nécessaire de recourir à une autorisation basée sur une liste de contrôle d’accès, qui doit être gérée par l’utilisateur. Vous pouvez utiliser cette fonctionnalité avec vos clients Kafka en spécifiant **SASL_SSL** pour le protocole et **PLAIN** pour le mécanisme. 

```properties
bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";
```

> [!IMPORTANT]
> Remplacez `{YOUR.EVENTHUBS.CONNECTION.STRING}` par la chaîne de connexion de votre espace de noms Event Hubs. Pour savoir comment obtenir la chaîne de connexion, consultez [Obtenir une chaîne de connexion Event Hubs](event-hubs-get-connection-string.md). Voici un exemple de configuration : `sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXXXXXXXXXXXXXXX";`

> [!NOTE]
> Si vous utilisez l’authentification SAS avec les clients Kafka, les connexions établies ne sont pas déconnectées lorsque la clé SAS est regénérée. 


#### <a name="samples"></a>Exemples 
Pour consulter un **tutoriel** avec des instructions pas à pas afin de créer un Event Hub et y accéder à l’aide d’une signature d’accès partagé ou OAuth, consultez [Démarrage rapide : Streaming de données avec Event Hubs en utilisant le protocole Kafka](event-hubs-quickstart-kafka-enabled-event-hubs.md).

Pour plus d' **exemples** montrant comment utiliser OAuth avec Event Hubs pour Kafka, consultez les [exemples sur GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth).

## <a name="other-event-hubs-features"></a>Autres fonctionnalités Event Hubs 

La fonctionnalité Event Hubs pour Apache Kafka est l'un des trois protocoles simultanément disponibles sur Azure Event Hubs, en plus de HTTP et d'AMQP. Vous pouvez écrire avec l'un de ces protocoles et lire avec un autre, afin de permettre à vos producteurs Apache Kafka actuels de continuer à publier via Apache Kafka, mais votre lecteur peut bénéficier de l'intégration native à l'interface AMQP d'Event Hubs, comme Azure Stream Analytics ou Azure Functions. À l'inverse, vous pouvez facilement intégrer Azure Event Hubs dans les réseaux de routage AMQP en tant que point de terminaison cible, tout en lisant les données par le biais d'intégrations Apache Kafka.  

En outre, les fonctionnalités Event Hubs telles que [Capture](event-hubs-capture-overview.md), qui permet un archivage à long terme extrêmement économique via Stockage Blob Azure et Azure Data Lake Storage, et [Géo-reprise d'activité après sinistre](event-hubs-geo-dr.md) sont également compatibles avec Event Hubs pour Kafka.

## <a name="apache-kafka-feature-differences"></a>Différences entre les fonctionnalités d'Apache Kafka 

L'objectif d'Event Hubs pour Apache Kafka est de fournir un accès aux fonctionnalités d'Azure Event Hub aux applications qui sont verrouillées dans l'API Apache Kafka et qui, autrement, devraient être soutenues par un cluster Apache Kafka. 

Comme expliqué [ci-dessus](#is-apache-kafka-the-right-solution-for-your-workload), l'ensemble de services de messagerie Azure offre une couverture enrichie et robuste pour une multitude de scénarios de messagerie, et bien que les fonctionnalités suivantes ne soient actuellement pas prises en charge par le support d'Event Hubs pour l'API Apache Kafka, nous indiquons où et comment accéder à la fonctionnalité souhaitée.

### <a name="transactions"></a>Transactions

[Azure Service Bus](../service-bus-messaging/service-bus-transactions.md) offre une prise en charge robuste des transactions qui permet de recevoir des messages et des sessions tout en envoyant des messages sortants issus du traitement des messages à différentes entités cibles sous la protection de la cohérence d'une transaction. L'ensemble de fonctionnalités permet non seulement de ne traiter qu'une seule fois chacun des messages d'une séquence, mais évite également qu'un autre consommateur retraite par inadvertance les mêmes messages, comme ce serait le cas avec Apache Kafka. Service Bus est le service recommandé pour les charges de travail de messages transactionnels.

### <a name="compression"></a>Compression

La fonctionnalité de [compression](https://cwiki.apache.org/confluence/display/KAFKA/Compression) côté client d'Apache Kafka compresse un lot de plusieurs messages en un seul message côté producteur et décompresse le lot côté consommateur. Le répartiteur Apache Kafka traite le lot comme un message spécial.

Cette fonctionnalité est fondamentalement en contradiction avec le modèle multiprotocole d'Azure Event Hubs, qui permet aux messages, même ceux envoyés par lots, d'être récupérés individuellement auprès du répartiteur et via n'importe quel protocole. 

La charge utile de tout événement Event Hub est un flux d'octets, et le contenu peut être compressé avec l'algorithme de votre choix. Le format d'encodage Apache Avro prend en charge la compression en mode natif.

### <a name="log-compaction"></a>Compactage des journaux

La fonctionnalité de compactage des journaux Apache Kafka permet de supprimer d'une partition tous les enregistrements à l'exception du dernier de chaque clé, ce qui transforme une rubrique Apache Kafka en magasin de paires clé-valeur où la dernière valeur ajoutée remplace la précédente. Le modèle de magasin de paires clé-valeur, même avec des mises à jour fréquentes, est bien mieux pris en charge par les services de base de données comme [Azure Cosmos DB](../cosmos-db/introduction.md).

La fonctionnalité de compactage des journaux est utilisée par les frameworks clients Kafka Connect et Kafka Streams.

### <a name="kafka-streams"></a>Kafka Streams

Kafka Streams est une bibliothèque de client dédiée à l'analyse des flux. Elle fait partie du projet open source Apache Kafka mais est distincte du répartiteur de flux d'événements Apache Kafka. 

Généralement, les clients Azure Event Hubs demandent la prise en charge de Kafka Streams parce qu'ils s'intéressent au produit « ksqlDB » de Confluent. « ksqlDB » est un projet propriétaire à source partagée qui est [concédé sous licence](https://github.com/confluentinc/ksql/blob/master/LICENSE) de manière à ce qu'aucun fournisseur « proposant des logiciels en tant que service, une plateforme en tant que service, une infrastructure en tant que service ou d'autres services en ligne similaires en concurrence avec les produits ou services Confluent » ne soit autorisé à utiliser ou à offrir la prise en charge « ksqlDB ». En pratique, si vous utilisez ksqlDB, vous devez exploiter Kafka vous-même ou utiliser les offres cloud de Confluent. Les termes du contrat de licence peuvent également s'appliquer aux clients Azure qui offrent des services à des fins exclues par la licence.

Autonome et non doté de ksqlDB, Kafka Streams possède moins de fonctionnalités que de nombreux frameworks et services alternatifs, dont la plupart disposent d'interfaces SQL intégrées de diffusion en continu, et qui s'intègrent toutes à Azure Event Hubs :

- [Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md)
- [Azure Synapse Analytics (via Event Hubs Capture)](../event-grid/event-grid-event-hubs-integration.md)
- [Azure Databricks](/azure/databricks/scenarios/databricks-stream-from-eventhubs)
- [Apache Samza](https://samza.apache.org/learn/documentation/latest/connectors/eventhubs)
- [Apache Storm](event-hubs-storm-getstarted-receive.md)
- [Apache Spark](event-hubs-kafka-spark-tutorial.md)
- [Apache Flink](event-hubs-kafka-flink-tutorial.md)
- [Akka Streams](event-hubs-kafka-akka-streams-tutorial.md)

Les services et frameworks répertoriés peuvent généralement acquérir des flux d'événements et des données de référence issus de sources diverses par le biais d'adaptateurs. Kafka Streams ne peut acquérir des données qu'à partir d'Apache Kafka ; vos projets d'analyse sont donc verrouillés dans Apache Kafka. Pour utiliser des données issues d'autres sources, vous devez d'abord importer ces données dans Apache Kafka à l'aide du framework Kafka Connect.

Si vous souhaitez utiliser le framework Kafka Streams sur Azure, [Apache Kafka sur HDInsight](../hdinsight/kafka/apache-kafka-introduction.md) vous fournira cette option. Apache Kafka sur HDInsight offre un contrôle total sur tous les aspects de la configuration d'Apache Kafka, tout en étant entièrement intégré à divers aspects de la plateforme Azure, de la sélection élective du domaine d'erreur/de mise à jour à l'isolement réseau, en passant par l'intégration de la surveillance. 

## <a name="next-steps"></a>Étapes suivantes
Cet article a présenté une introduction à Event Hubs pour Kafka. Pour en savoir plus, consultez [Guide du développeur Apache Kafka pour Azure Event Hubs](apache-kafka-developer-guide.md).