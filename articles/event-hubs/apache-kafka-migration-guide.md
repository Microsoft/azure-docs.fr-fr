---
title: Migrer vers Azure Event Hubs pour Apache Kafka
description: Cet article explique comment migrer des clients d’Apache Kafka vers Azure Event Hubs.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: d9f3775a85df5a881c2c38566628e4e1d4d8c40e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90061442"
---
# <a name="migrate-to-azure-event-hubs-for-apache-kafka-ecosystems"></a>Migrer vers Azure Event Hubs pour les écosystèmes Apache Kafka
Azure Event Hubs expose un point de terminaison Apache Kafka, qui vous permet de vous connecter à Event Hubs en utilisant le protocole Kafka. En apportant des modifications minimales à votre application Kafka existante, vous pouvez vous connecter à Azure Event Hubs et profiter des avantages de l'écosystème Azure. Event Hubs fonctionne avec un grand nombre de vos applications Kafka existantes, dont MirrorMaker. Pour plus d’informations, consultez [Event Hubs pour Apache Kafka](event-hubs-for-kafka-ecosystem-overview.md)

## <a name="pre-migration"></a>Prémigration 

### <a name="create-an-azure-account"></a>Création d'un compte Azure
Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.

### <a name="create-an-event-hubs-namespace"></a>Créer un espace de noms Event Hubs
Suivez les instructions pas à pas de l'article [Créer un Event Hub](event-hubs-create.md) pour créer un espace de noms Event Hubs et un Event Hub. 

### <a name="connection-string"></a>Chaîne de connexion
Suivez les étapes de l'article [Obtenir la chaîne de connexion à partir du portail](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Et notez la chaîne de connexion en vue d'une utilisation ultérieure. 

### <a name="fully-qualified-domain-name-fqdn"></a>Nom de domaine complet (FQDN)
Vous pouvez également avoir besoin du FQDN qui pointe vers votre espace de noms Event Hub. Vous trouverez ce FQDN dans votre chaîne de connexion :

`Endpoint=sb://`**`mynamespace.servicebus.windows.net`**`/;SharedAccessKeyName=XXXXXX;SharedAccessKey=XXXXXX`

Si votre espace de noms Event Hubs est déployé sur un cloud non public, votre nom de domaine peut être différent (par exemple, \*.servicebus.chinacloudapi.cn, \*.servicebus.usgovcloudapi.net ou \*.servicebus.cloudapi.de).

## <a name="migration"></a>Migration 

### <a name="update-your-kafka-client-configuration"></a>Mettre à jour la configuration de votre client Kafka

Pour vous connecter à un Event Hub compatible avec Kafka, vous devez mettre à jour les configurations du client Kafka. Si vous avez du mal à trouver le vôtre, essayez de rechercher où `bootstrap.servers` est défini dans votre application.

Insérez les configurations suivantes là où cela s'avère nécessaire dans votre application. Veillez à mettre à jour les valeurs `bootstrap.servers` et `sasl.jaas.config` pour diriger le client vers votre point de terminaison Event Hubs Kafka avec l'authentification qui convient. 

```
bootstrap.servers={MYNAMESPACE}.servicebus.windows.net:9093
request.timeout.ms=60000
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{CONNECTION STRING TO YOUR NAMESPACE}";
``` 

Si `sasl.jaas.config` n'est pas une configuration prise en charge dans votre infrastructure, recherchez les configurations utilisées pour définir le nom d'utilisateur et le mot de passe SASL, et utilisez-les à la place. Définissez le nom d'utilisateur sur `$ConnectionString` et le mot de passe sur votre chaîne de connexion Event Hubs.

## <a name="post-migration"></a>Postmigration
Exécutez votre application Kafka qui envoie des événements au Event Hub. Vérifiez ensuite que cet Event Hub reçoit les événements à l'aide du portail Azure. Sur la page **Présentation** de votre espace de noms Event Hubs, accédez à l'affichage **Messages** de la section **Métriques**. Actualisez la page pour mettre à jour le graphique. Cela peut prendre quelques secondes pour indiquer que les messages ont été reçus. 

[![Vérifier que le hub d’événements a reçu les messages](./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png)](./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png#lightbox)


## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur Event Hubs et sur Event Hubs pour Kafka, consultez les articles suivants :  

- [Guide de dépannage Apache Kafka pour Event Hubs](apache-kafka-troubleshooting-guide.md)
- [Forum aux questions (FAQ) - Event Hubs pour Apache Kafka](apache-kafka-frequently-asked-questions.md)
- [Guide du développeur Apache Kafka pour Azure Event Hubs](apache-kafka-developer-guide.md)
- [Configurations recommandées](apache-kafka-configurations.md)