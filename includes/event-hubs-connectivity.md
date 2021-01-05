---
title: Fichier Include
description: Fichier Include
services: event-hubs
author: spelluru
ms.service: event-hubs
ms.topic: include
ms.date: 11/19/2020
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: dac82692c76d9d36b1f25d7b93b5c3a2e2400672
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002787"
---
### <a name="what-ports-do-i-need-to-open-on-the-firewall"></a>Quels ports du pare-feu dois-je ouvrir ? 
Vous pouvez utiliser les protocoles suivants avec Azure Event Hubs pour envoyer et recevoir des événements :

- Advanced Message Queuing Protocol 1.0 (AMQP)
- Hypertext Transfer Protocol 1.1 avec TLS (HTTPS)
- Apache Kafka

Consultez le tableau suivant pour savoir quels ports de sortie vous devez ouvrir afin d’utiliser ces protocoles dans le but de communiquer avec Azure Event Hubs. 

| Protocol | Ports | Détails | 
| -------- | ----- | ------- | 
| AMQP | 5671 et 5672 | Consultez le [Guide du protocole AMQP](../articles/service-bus-messaging/service-bus-amqp-protocol-guide.md) | 
| HTTPS | 443 | Ce port est utilisé pour l’API HTTP/REST et pour AMQP sur WebSockets. |
| Kafka | 9093 | Voir [Utiliser Azure Event Hubs à partir d’applications Apache Kafka](../articles/event-hubs/event-hubs-for-kafka-ecosystem-overview.md)

Le port HTTPS est requis pour la communication sortante quand AMQP est utilisé sur le port 5671, car plusieurs opérations de gestion effectuées par les kits SDK clients et l’acquisition de jetons à partir d’Azure Active Directory (le cas échéant) s’exécutent sur HTTPS. 

Les kits SDK officiels Azure utilisent généralement le protocole AMQP pour envoyer et recevoir des messages d’Event Hubs. L’option de protocole AMQP sur WebSockets s’exécute sur le port TCP 443, tout comme l’API HTTP, mais son fonctionnement est identique au mode AMQP classique. Cette option présente une latence de connexion initiale plus élevée en raison d’allers-retours supplémentaires pour l’établissement d’une liaison et d’une surcharge légèrement plus importante en compensation du partage du port HTTPS. Si ce mode est sélectionné, le port TCP 443 s’avère suffisant à des fins de communication. Les options suivantes permettent de sélectionner le mode AMQP classique ou le mode AMQP WebSockets :

| Language | Option   |
| -------- | ----- |
| .NET     | Propriété [EventHubConnectionOptions.TransportType](/dotnet/api/azure.messaging.eventhubs.eventhubconnectionoptions.transporttype?view=azure-dotnet&preserve-view=true) avec [EventHubsTransportType.AmqpTcp](/dotnet/api/azure.messaging.eventhubs.eventhubstransporttype?view=azure-dotnet&preserve-view=true) ou [EventHubsTransportType.AmqpWebSockets](/dotnet/api/azure.messaging.eventhubs.eventhubstransporttype?view=azure-dotnet&preserve-view=true) |
| Java     | [com.microsoft.azure.eventhubs.EventProcessorClientBuilder.transporttype](/java/api/com.azure.messaging.eventhubs.eventprocessorclientbuilder.transporttype?view=azure-java-stable&preserve-view=true) avec [AmqpTransportType.AMQP](/java/api/com.azure.core.amqp.amqptransporttype?view=azure-java-stable&preserve-view=true) ou [AmqpTransportType.AMQP_WEB_SOCKETS](/java/api/com.azure.core.amqp.amqptransporttype?view=azure-java-stable&preserve-view=true) |
| Nœud  | [EventHubConsumerClientOptions](/javascript/api/@azure/event-hubs/eventhubconsumerclientoptions?view=azure-node-latest&preserve-view=true) possède une propriété `webSocketOptions`. |
| Python | [EventHubConsumerClient.transport_type](/python/api/azure-eventhub/azure.eventhub.eventhubconsumerclient?view=azure-python&preserve-view=true) avec [TransportType.Amqp](/python/api/azure-eventhub/azure.eventhub.transporttype?view=azure-python) ou [TransportType.AmqpOverWebSocket](/python/api/azure-eventhub/azure.eventhub.transporttype?view=azure-python&preserve-view=true) |

### <a name="what-ip-addresses-do-i-need-to-allow"></a>Quelles adresses IP dois-je autoriser ?
Lorsque vous utilisez Azure, vous devez parfois autoriser des plages d’adresses IP ou des URL spécifiques dans votre pare-feu ou proxy d’entreprise pour accéder à tous les services Azure que vous utilisez ou essayez d’utiliser. Vérifiez que le trafic est autorisé sur les adresses IP utilisées par Event Hubs. Pour les adresses IP utilisées par Azure Event Hubs : consultez le document [Plages d’adresses IP Azure et balises de service – Cloud public](https://www.microsoft.com/download/details.aspx?id=56519).

Vérifiez également que l’adresse IP de votre espace de noms est autorisée. Pour trouver les adresses IP à autoriser pour vos connexions, procédez comme suit :

1. Exécutez la commande suivante depuis une invite de commandes : 

    ```
    nslookup <YourNamespaceName>.servicebus.windows.net
    ```
2. Notez l’adresse IP renvoyée dans `Non-authoritative answer`. 

Si vous utilisez la **redondance de zone** pour votre espace de noms, vous devez suivre quelques étapes supplémentaires : 

1. Tout d’abord, exécutez nslookup sur l’espace de noms.

    ```
    nslookup <yournamespace>.servicebus.windows.net
    ```
2. Notez le nom dans la section **Réponse ne faisant pas autorité**, qui se présente dans l’un des formats suivants : 

    ```
    <name>-s1.cloudapp.net
    <name>-s2.cloudapp.net
    <name>-s3.cloudapp.net
    ```
3. Exécutez nslookup pour chacun d’eux avec des suffixes s1, s2 et s3 pour obtenir les adresses IP des 3 instances en cours d’exécution dans 3 zones de disponibilité. 

    > [!NOTE]
    > L’adresse IP retournée par la commande `nslookup` n’est pas une adresse IP statique. Toutefois, elle reste constante jusqu’à ce que le déploiement sous-jacent soit supprimé ou déplacé vers un autre cluster.

### <a name="where-can-i-find-client-ip-sending-or-receiving-messages-to-my-namespace"></a>Où puis-je trouver l’adresse IP cliente qui échange des messages avec mon espace de noms ?
Tout d’abord, activez le [Filtrage IP](../articles/event-hubs/event-hubs-ip-filtering.md) sur l’espace de noms. 

Activez ensuite les journaux de diagnostic pour [Événements de connexion au réseau virtuel Event Hubs](../articles/event-hubs/event-hubs-diagnostic-logs.md#event-hubs-virtual-network-connection-event-schema) en suivant les instructions dans [Activer les journaux de diagnostic](../articles/event-hubs/event-hubs-diagnostic-logs.md#enable-diagnostic-logs). Vous verrez l’adresse IP pour laquelle la connexion est refusée.

```json
{
    "SubscriptionId": "0000000-0000-0000-0000-000000000000",
    "NamespaceName": "namespace-name",
    "IPAddress": "1.2.3.4",
    "Action": "Deny Connection",
    "Reason": "IPAddress doesn't belong to a subnet with Service Endpoint enabled.",
    "Count": "65",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name",
    "Category": "EventHubVNetConnectionEvent"
}
```

> [!IMPORTANT]
> Les journaux de réseau virtuel ne sont générés que si l’espace de noms autorise l’accès provenant **d’adresses IP spécifiques** (règles de filtre d’adresse IP). Si vous souhaitez obtenir des journaux de réseau virtuel pour suivre l’adresse IP des clients qui se connectent à l’espace de noms Event Hubs sans pour autant restreindre l’accès à votre espace de noms à l’aide de ces fonctionnalités, vous pouvez appliquer la solution de contournement suivante : activez le filtrage d’adresse IP et ajoutez la plage IPv4 adressable totale (1.0.0.0/1-255.0.0.0/1). Event Hubs ne prend pas en charge les plages d’adresses IPv6. 
