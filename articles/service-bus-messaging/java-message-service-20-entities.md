---
title: Messagerie Azure Service Bus-Entités de service de messagerie Java
description: Cet article fournit une vue d’ensemble des entités de messagerie Azure Service Bus via l’API de service de messagerie Java.
ms.topic: article
ms.date: 07/20/2020
ms.openlocfilehash: 1a7fe3d6355146ccf0fce50266a6f3b8da5231b3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87801244"
---
# <a name="java-message-service-jms-20-entities-preview"></a>Entités de Java Message Service (JMS) 2.0 (préversion)

Les applications clientes se connectant à Azure Service Bus Premium et utilisant la [bibliothèque JMS Azure Service Bus](https://search.maven.org/artifact/com.microsoft.azure/azure-servicebus-jms) peuvent tirer parti des entités ci-dessous.

## <a name="queues"></a>Files d’attente

Les files d’attente dans JMS sont sémantiquement comparables aux [files d’attente Service Bus](service-bus-queues-topics-subscriptions.md#queues) traditionnelles.

Pour créer une file d’attente, utilisez les méthodes ci-dessous dans la classe `JMSContext` –

```java
Queue createQueue(String queueName)
```

## <a name="topics"></a>Rubriques

Les rubriques dans JMS sont sémantiquement comparables aux [files d’attente Service Bus](service-bus-queues-topics-subscriptions.md#topics-and-subscriptions) traditionnelles.

Pour créer une rubrique, utilisez les méthodes ci-dessous dans la classe `JMSContext` –

```java
Topic createTopic(String topicName)
```

## <a name="temporary-queues"></a>Files d’attente temporaires

Lorsqu’une application cliente requiert une entité temporaire qui existe pendant la durée de vie de l’application, elle peut utiliser des files d’attente temporaires. Elles sont utilisées dans le modèle de [ réponse à la demande](https://www.enterpriseintegrationpatterns.com/patterns/messaging/RequestReply.html).

Pour créer une file d’attente temporaire, utilisez les méthodes ci-dessous dans la classe `JMSContext` –

```java
TemporaryQueue createTemporaryQueue()
```

## <a name="temporary-topics"></a>Rubriques temporaires

Tout comme les files d’attente temporaires, des rubriques temporaires permettent la publication/l’abonnement via une entité temporaire qui existe pendant la durée de vie de l’application.

Pour créer une rubrique temporaire, utilisez les méthodes ci-dessous dans la classe `JMSContext` –

```java
TemporaryTopic createTemporaryTopic()
```

## <a name="java-message-service-jms-subscriptions"></a>JMS (Java Message Service) et abonnements

Bien que qu’elles soient sémantiquement similaires aux [Abonnements ](service-bus-queues-topics-subscriptions.md#topics-and-subscriptions)(qui existent sur une rubrique et activent la sémantique de publication/d’abonnement), les spécifications de service de message Java présentent les concepts d’attributs **partagés**, **non partagés**, **Durables** et **Non durables** pour un abonnement donné.

> [!NOTE]
> Les abonnements ci-dessous sont disponibles dans le niveau Azure Service Bus Premium pour la préversion des applications clientes se connectant à Azure Service Bus à l’aide de la [bibliothèque JMS Azure Service Bus](https://search.maven.org/artifact/com.microsoft.azure/azure-servicebus-jms).
>
> Pour la préversion publique, ces abonnements ne peuvent pas être créés à l’aide du portail Azure.
>

### <a name="shared-durable-subscriptions"></a>Abonnements durables partagés

Un abonnement durable partagé est utilisé lorsque tous les messages publiés sur une rubrique doivent être reçus et traités par une application, que l’application consomme ou non activement l’abonnement à tout moment.

Étant donné qu’il s’agit d’un abonnement partagé, toute application authentifiée pour recevoir des messages de Service Bus peut en recevoir de l’abonnement.

Pour créer un abonnement durable partagé, utilisez les méthodes ci-dessous sur la classe `JMSContext` –

```java
JMSConsumer createSharedDurableConsumer(Topic topic, String name)

JMSConsumer createSharedDurableConsumer(Topic topic, String name, String messageSelector)
```

L’abonnement durable partagé continue à exister, sauf s’il est supprimé à l’aide de la méthode `unsubscribe` sur la classe `JMSContext`.

```java
void unsubscribe(String name)
```

### <a name="unshared-durable-subscriptions"></a>Abonnements durables non partagés

Tout comme un abonnement durable partagé, un abonnement durable partagé est utilisé lorsque tous les messages publiés sur une rubrique doivent être reçus et traités par une application, que l’application consomme ou non activement l’abonnement à tout moment.

Toutefois, étant donné qu’il s’agit d’un abonnement non partagé, seule l’application qui a créé l’abonnement peut recevoir des messages de ce dernier.

Pour créer un abonnement durable non partagé, utilisez les méthodes ci-dessous à partir de la classe `JMSContext` – 

```java
JMSConsumer createDurableConsumer(Topic topic, String name)

JMSConsumer createDurableConsumer(Topic topic, String name, String messageSelector, boolean noLocal)
```

> [!NOTE]
> La fonctionnalité `noLocal` n’est actuellement pas prise en charge et est ignorée.
>

L’abonnement durable non partagé continue à exister, sauf s’il est supprimé à l’aide de la méthode `unsubscribe` sur la classe `JMSContext`.

```java
void unsubscribe(String name)
```

### <a name="shared-non-durable-subscriptions"></a>Abonnements non durables partagés

Un abonnement partagé non durable est utilisé lorsque plusieurs applications clientes doivent recevoir et traiter des messages d’un seul abonnement, uniquement jusqu’à ce qu’ils les consomment/les reçoivent activement.

Étant donné que l’abonnement n’est pas durable, il n’est pas persistant. Les messages ne sont pas reçus par cet abonnement lorsqu’il n’y a aucun consommateur actif.

Pour créer un abonnement partagé non durable, créez un `JmsConsumer` comme indiqué dans les méthodes ci-dessous de la classe `JMSContext` –

```java
JMSConsumer createSharedConsumer(Topic topic, String sharedSubscriptionName)

JMSConsumer createSharedConsumer(Topic topic, String sharedSubscriptionName, String messageSelector)
```

L’abonnement partagé non durable continue d’exister jusqu’à ce qu’il y ait des consommateurs actifs qui le reçoivent.

### <a name="unshared-non-durable-subscriptions"></a>Abonnements non durables non partagés

Un abonnement non partagé non durable est utilisé lorsque l’application cliente doit recevoir et traiter des messages d’un seul abonnement, uniquement jusqu’à ce qu’elle les consomme/les reçoive activement. Un seul consommateur peut exister sur cet abonnement, à savoir le client qui a créé l’abonnement.

Étant donné que l’abonnement n’est pas durable, il n’est pas persistant. Les messages ne sont pas reçus par cet abonnement lorsqu’il n’y a aucun consommateur actif.

Pour créer un abonnement non-partagé non durable, créez un `JMSConsumer` comme indiqué dans les méthodes ci-dessous de la classe `JMSContext` –

```java
JMSConsumer createConsumer(Destination destination)

JMSConsumer createConsumer(Destination destination, String messageSelector)

JMSConsumer createConsumer(Destination destination, String messageSelector, boolean noLocal)
```

> [!NOTE]
> La fonctionnalité `noLocal` n’est actuellement pas prise en charge et est ignorée.
>

L’abonnement non partagé non durable continue d’exister jusqu’à ce qu’il y ait un consommateur actif qui le reçoit.

### <a name="message-selectors"></a>Sélecteurs de messages

À l’instar des **filtres et actions** qui existent pour les abonnements Service Bus standard, les **sélecteurs de messages** existent pour les abonnements JMS.

Les sélecteurs de messages peuvent être configurés sur chacun des abonnements JMS et exister sous la forme d’une condition de filtre sur les propriétés d’en-tête de message. Seuls les messages avec des propriétés d’en-tête correspondant à l’expression du sélecteur de messages sont remis. La valeur Null ou une chaîne vide indique qu’il n’existe aucun sélecteur de message pour l’abonnement/le consommateur JMS.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations et des exemples d’utilisation de la messagerie Service Bus, consultez les rubriques avancées suivantes :

* [Présentation de la messagerie Service Bus](service-bus-messaging-overview.md)
* [Utiliser l’API Java Message Service 2.0 avec Azure Service Bus Premium (Version préliminaire)](how-to-use-java-message-service-20.md)



