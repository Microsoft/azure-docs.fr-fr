---
title: Migrer des applications JMS (Java Message Service) d’Apache ActiveMQ vers Azure Service Bus | Microsoft Docs
description: Cet article explique comment migrer des applications JMS existantes qui interagissent avec Apache ActiveMQ pour interagir avec Azure Service Bus.
services: service-bus-messaging
documentationcenter: ''
author: axisc
manager: timlt
editor: spelluru
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/07/2020
ms.author: aschhab
ms.custom: devx-track-java
ms.openlocfilehash: b8408dde86d1902cf5b4899c4783c9dd185449ee
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92515744"
---
# <a name="migrate-existing-java-message-service-jms-20-applications-from-apache-activemq-to-azure-service-bus"></a>Migrer des applications JMS (Java Message Service) 2.0 existantes d’Apache ActiveMQ vers Azure Service Bus

Cet article explique comment modifier une application Java Message Service (JMS) 2.0 qui interagit avec un répartiteur JMS afin qu’elle interagisse avec Azure Service Bus à la place. En particulier, l’article traite de la migration à partir d’Apache ActiveMQ ou d’Amazon MQ.

Azure Service Bus prend en charge les charges de travail de la plateforme Java 2, Édition Entreprise et Spring qui utilisent l’API JMS 2.0 sur AMQP (Advanced Message Queueing Protocol).

## <a name="before-you-start"></a>Avant de commencer

### <a name="differences-between-azure-service-bus-and-apache-activemq"></a>Différences entre Azure Service Bus et Apache ActiveMQ

Azure Service Bus et Apache ActiveMQ sont tous deux des répartiteurs de messages, qui fonctionnent en tant que fournisseurs JMS avec lesquels les applications clientes échangent des messages. Ils activent tous les deux la sémantique point-à-point avec des files d’attente et la sémantique de publication-abonnement avec des rubriques et des abonnements. 

Même dans ce cas, il existe des différences entre les deux, comme le montre le tableau suivant :

| Category | ActiveMQ | Azure Service Bus |
| --- | --- | --- |
| Hiérarchisation des applications | Monolithe en cluster | À deux niveaux <br> (passerelle + back-end) |
| Prise en charge de protocole | <ul> <li>AMQP</li> <li> STOMP </li> <li> OpenWire </li> </ul> | AMQP |
| Mode de provisionnement | <ul> <li> IaaS (Infrastructure as a service), localement </li> <li> Amazon MQ (plateforme managée en tant que service) </li> | Plateforme managée en tant que service (PaaS) |
| Taille des messages | Configurable par le client | 1 Mo (niveau Premium) |
| Haute disponibilité | Managée par le client | Managée par la plateforme |
| Récupération d'urgence | Managée par le client | Managée par la plateforme | 

### <a name="current-supported-and-unsupported-features"></a>Fonctionnalités prises en charge et non prises en charge actuellement

[!INCLUDE [service-bus-jms-features-list](../../includes/service-bus-jms-feature-list.md)]

### <a name="considerations"></a>Considérations

La nature à deux niveaux d’Azure Service Bus offre différentes fonctionnalités de continuité d’activité (haute disponibilité et reprise d’activité). Toutefois, certaines considérations sont à prendre en compte quand vous utilisez des fonctionnalités JMS.

#### <a name="service-upgrades"></a>Mises à niveau de service

En cas de mises à niveau et de redémarrages de Service Bus, les files d’attente ou les rubriques temporaires sont supprimées. Si votre application est sensible à la perte de données dans les files d’attente ou les rubriques temporaires, n’utilisez pas de files d’attente ou de rubriques temporaires. Utilisez à la place des files d’attente, des rubriques et des abonnements durables.

#### <a name="data-migration"></a>Migration des données

Dans le cadre de la migration et de la modification de vos applications clientes pour qu’elles interagissent avec Azure Service Bus, les données conservées dans ActiveMQ ne sont pas migrées vers Service Bus. Une application personnalisée peut vous être nécessaire pour vider les files d’attente, les rubriques et les abonnements ActiveMQ, puis relire les messages dans les files d’attente, les rubriques et les abonnements de Service Bus.

#### <a name="authentication-and-authorization"></a>Authentification et autorisation

Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) soutenu par Azure Active Directory est le mécanisme d’authentification à privilégier pour Service Bus. Toutefois, étant donné qu’Azure RBAC, ou l’authentification basée sur les revendications, n’est pas pris en charge par Apache QPID JMS, vous devez utiliser des clés SAS pour l’authentification.

## <a name="pre-migration"></a>Prémigration

### <a name="version-check"></a>Vérification de la version

Vous utilisez les composants et versions suivants lors de l’écriture des applications JMS : 

| Composant | Version |
|---|---|
| API JMS (Java Message Service) | 1.1 ou ultérieure |
| Protocole AMQP | 1.0 |

### <a name="ensure-that-amqp-ports-are-open"></a>Vérifier que les ports AMQP sont ouverts

Service Bus prend en charge la communication par le biais du protocole AMQP. À cet effet, activez la communication sur les ports 5671 (AMQP) et 443 (TCP). Selon l’emplacement où les applications clientes sont hébergées, vous pouvez avoir besoin d’un ticket de support pour autoriser la communication sur ces ports.

> [!IMPORTANT]
> Service Bus prend uniquement en charge le protocole AMQP 1.0.

### <a name="set-up-enterprise-configurations"></a>Élaborer des configurations d’entreprise

Service Bus offre différentes fonctionnalités de haute disponibilité et de sécurité de l’entreprise. Pour plus d'informations, consultez les pages suivantes : 

  * [Points de terminaison de service de réseau virtuel](service-bus-service-endpoints.md)
  * [Pare-feu](service-bus-ip-filtering.md)
  * [Chiffrement côté service avec clé gérée par le client (BYOK)](configure-customer-managed-key.md)
  * [Points de terminaison privés](private-link-service.md)
  * [Authentification et autorisation](service-bus-authentication-and-authorization.md)

### <a name="monitoring-alerts-and-tracing"></a>Supervision, alertes et suivi

Pour chaque espace de noms Service Bus, vous publiez des métriques sur Azure Monitor. Vous pouvez utiliser ces métriques à des fins d’alerte et de mise à l’échelle dynamique des ressources allouées à l’espace de noms.

Pour plus d’informations sur les différentes métriques et sur la configuration d’alertes sur celles-ci, consultez [Métriques Service Bus dans Azure Monitor](service-bus-metrics-azure-monitor.md). Vous pouvez également consulter les articles sur le [suivi côté client des opérations de données](service-bus-end-to-end-tracing.md) et la [journalisation opérationnelle/des diagnostics pour les opérations de gestion](service-bus-diagnostic-logs.md).

### <a name="metrics---new-relic"></a>Métriques - New Relic

Vous pouvez établir une correspondance entre les métriques ActiveMQ et les métriques Azure Service Bus. Consultez les informations suivantes sur le site New Relic :

  * [Métriques New Relic ActiveMQ/Amazon MQ](https://docs.newrelic.com/docs/integrations/amazon-integrations/aws-integrations-list/aws-mq-integration)
  * [Métriques New Relic Azure Service Bus](https://docs.newrelic.com/docs/integrations/microsoft-azure-integrations/azure-integrations-list/azure-service-bus-monitoring-integration)

> [!NOTE]
> New Relic n’offre pas d’intégration fluide et directe à ActiveMQ, mais des métriques sont disponibles pour Amazon MQ. Amazon MQ étant dérivé d’ActiveMQ, le tableau suivant mappe les métriques New Relic d’AmazonMQ à Azure Service Bus.
>

|Regroupement des métriques| Métrique Amazon MQ/ActiveMQ | Métrique Azure Service Bus |
|------------|---------------------------|--------------------------|
|Service Broker|`CpuUtilization`|`CPUXNS`|
|Service Broker|`MemoryUsage`|`WSXNS`|
|Service Broker|`CurrentConnectionsCount`|`activeConnections`|
|Service Broker|`EstablishedConnectionsCount`|`activeConnections` + `connectionsClosed`|
|Service Broker|`InactiveDurableTopicSubscribersCount`|Utiliser les métriques des abonnements|
|Service Broker|`TotalMessageCount`|Utiliser le niveau file d’attente/rubrique/abonnement `activeMessages`|
|File d’attente/Rubrique|`EnqueueCount`|`incomingMessages`|
|File d’attente/Rubrique|`DequeueCount`|`outgoingMessages`|
|File d'attente|`QueueSize`|`sizeBytes`|



## <a name="migration"></a>Migration

Pour migrer votre application JMS 2.0 existante afin d’interagir avec Service Bus, suivez les étapes décrites dans les sections suivantes.

### <a name="export-the-topology-from-activemq-and-create-the-entities-in-service-bus-optional"></a>Exporter la topologie à partir d’ActiveMQ et créer les entités dans Service Bus (facultatif)

Pour que les applications clientes puissent se connecter de façon fluide à Service Bus, migrez la topologie (y compris les files d’attente, les rubriques et les abonnements) d’Apache ActiveMQ vers Service Bus.

> [!NOTE]
> Pour les applications JMS, vous créez les files d’attente, les rubriques et les abonnements en tant qu’opération d’exécution. La plupart des fournisseurs JMS (courtiers de messages) vous donnent la possibilité de les créer au moment de l’exécution. C’est pourquoi cette étape d’exportation est considérée comme facultative. Pour être sûr que votre application dispose des autorisations nécessaires pour créer la topologie au moment de l’exécution, utilisez la chaîne de connexion avec les autorisations `Manage` SAS.

Pour ce faire :

1. Utilisez les [outils en ligne de commande ActiveMQ](https://activemq.apache.org/activemq-command-line-tools-reference) pour exporter la topologie.
1. Recréez la même topologie à l’aide d’un [modèle Azure Resource Manager](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).
1. Exécutez le modèle Azure Resource Manager.


### <a name="import-the-maven-dependency-for-service-bus-jms-implementation"></a>Importer la dépendance Maven pour l’implémentation JMS Service Bus

Pour garantir une connectivité fluide avec Service Bus, ajoutez le package `azure-servicebus-jms` en tant que dépendance au fichier Maven `pom.xml`, comme suit :

```xml
<dependencies>
...
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-servicebus-jms</artifactId>
    </dependency>
...
</dependencies>
```

### <a name="application-server-configuration-changes"></a>Modifications de configuration du serveur d’applications

Cette partie est personnalisée pour le serveur d’applications qui héberge vos applications clientes se connectant à ActiveMQ.

#### <a name="tomcat"></a>Tomcat

Ici, vous commencez par la configuration propre à ActiveMQ comme indiqué dans le fichier `/META-INF/context.xml`.

```XML
<Context antiJARLocking="true">
    <Resource
        name="jms/ConnectionFactory"
        auth="Container"
        type="org.apache.activemq.ActiveMQConnectionFactory"
        description="JMS Connection Factory"
        factory="org.apache.activemq.jndi.JNDIReferenceFactory"
        brokerURL="tcp://localhost:61616"
        brokerName="LocalActiveMQBroker"
        useEmbeddedBroker="false"/>

    <Resource name="jms/topic/MyTopic"
        auth="Container"
        type="org.apache.activemq.command.ActiveMQTopic"
        factory="org.apache.activemq.jndi.JNDIReferenceFactory"
        physicalName="MY.TEST.FOO"/>
    <Resource name="jms/queue/MyQueue"
        auth="Container"
        type="org.apache.activemq.command.ActiveMQQueue"
        factory="org.apache.activemq.jndi.JNDIReferenceFactory"
        physicalName="MY.TEST.FOO.QUEUE"/>
</Context>
```

Vous l’adaptez pour pointer vers Service Bus, comme suit :

```xml
<Context antiJARLocking="true">
    <Resource
        name="jms/ConnectionFactory"
        auth="Container"
        type="com.microsoft.azure.servicebus.jms.ServiceBusJmsConnectionFactory"
        description="JMS Connection Factory"
        factory="org.apache.qpid.jms.jndi.JNDIReferenceFactory"
        connectionString="<INSERT YOUR SERVICE BUS CONNECTION STRING HERE>"/>

    <Resource name="jms/topic/MyTopic"
        auth="Container"
        type="org.apache.qpid.jms.JmsTopic"
        factory="org.apache.qpid.jms.jndi.JNDIReferenceFactory"
        physicalName="MY.TEST.FOO"/>
    <Resource name="jms/queue/MyQueue"
        auth="Container"
        type="org.apache.qpid.jms.JmsQueue"
        factory="org.apache.qpid.jms.jndi.JNDIReferenceFactory"
        physicalName="MY.TEST.FOO.QUEUE"/>
</Context>
```

#### <a name="spring-applications"></a>Applications Spring

##### <a name="update-the-applicationproperties-file"></a>Mettre à jour le fichier `application.properties`

Si vous utilisez une application Spring Boot pour vous connecter à ActiveMQ, supprimez les propriétés spécifiques à ActiveMQ du fichier `application.properties`.

```properties
spring.activemq.broker-url=<ACTIVEMQ BROKER URL>
spring.activemq.user=<ACTIVEMQ USERNAME>
spring.activemq.password=<ACTIVEMQ PASSWORD>
```

Ensuite, ajoutez les propriétés propres à Service Bus au fichier `application.properties`.

```properties
azure.servicebus.connection-string=Endpoint=myEndpoint;SharedAccessKeyName=mySharedAccessKeyName;SharedAccessKey=mySharedAccessKey
```

##### <a name="replace-activemqconnectionfactory-with-servicebusjmsconnectionfactory"></a>Remplacer `ActiveMQConnectionFactory` par `ServiceBusJmsConnectionFactory`

L’étape suivante consiste à remplacer l’instance de `ActiveMQConnectionFactory` par `ServiceBusJmsConnectionFactory`.

> [!NOTE] 
> Les modifications de code réelles sont propres à l’application et à la façon dont les dépendances sont gérées, mais l’exemple suivant fournit des conseils sur ce qui doit être changé.
>

Auparavant, vous deviez instancier un objet d’`ActiveMQConnectionFactory`, comme suit :

```java

String BROKER_URL = "<URL of the hosted ActiveMQ broker>";
ConnectionFactory factory = new ActiveMQConnectionFactory(BROKER_URL);

Connection connection = factory.createConnection();
connection.start();

```

À présent, vous instanciez un objet de `ServiceBusJmsConnectionFactory`, comme suit :

```java

ServiceBusJmsConnectionFactorySettings settings = new ServiceBusJmsConnectionFactorySettings();
String SERVICE_BUS_CONNECTION_STRING = "<Service Bus Connection string>";

ConnectionFactory factory = new ServiceBusJmsConnectionFactory(SERVICE_BUS_CONNECTION_STRING, settings);

Connection connection = factory.createConnection();
connection.start();

```

## <a name="post-migration"></a>Postmigration

Maintenant que vous avez modifié l’application pour qu’elle commence à envoyer et à recevoir des messages de Service Bus, vous devez vérifier qu’elle fonctionne comme prévu. Quand cette opération est terminée, vous pouvez affiner et moderniser votre pile d’application.

## <a name="next-steps"></a>Étapes suivantes

Utilisez [Spring Boot Starter pour Azure Service Bus JMS](/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-azure-service-bus) afin de bénéficier d’une intégration fluide à Service Bus.

Pour plus d’informations sur la messagerie Service Bus et JMS, consultez :

* [JMS avec Service Bus](service-bus-java-how-to-use-jms-api-amqp.md)
* [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md)
* [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)