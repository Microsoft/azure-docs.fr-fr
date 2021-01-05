---
title: Fichier include
description: Fichier include
services: service-bus-messaging
author: axisc
ms.service: service-bus-messaging
ms.topic: include
ms.date: 6/9/2020
ms.author: aschhab
ms.custom: include file
ms.openlocfilehash: 9030080d0b8c8e032cb2992a62275efcdb04aabc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87798135"
---
Le tableau suivant présente les fonctionnalités Java Message Service (JMS) actuellement prises en charge par Azure Service Bus. Il présente également des fonctionnalités non prises en charge.


| Fonctionnalité | API |Statut |
|---|---|---|
| Files d’attente   | <ul> <li> JMSContext.createQueue( String queueName) </li> </ul>| **Pris en charge** |
| Rubriques   | <ul> <li> JMSContext.createTopic( String topicName) </li> </ul>| **Pris en charge** |
| Files d’attente temporaires |<ul> <li> JMSContext.createTemporaryQueue() </li> </ul>| **Pris en charge** |
| Rubriques temporaires |<ul> <li> JMSContext.createTemporaryTopic() </li> </ul>| **Pris en charge** |
| Producteur de messages /<br/> JMSProducer |<ul> <li> JMSContext.createProducer() </li> </ul>| **Pris en charge** |
| Explorateurs de files d'attente |<ul> <li> JMSContext.createBrowser(Queue queue) </li> <li> JMSContext.createBrowser(Queue queue, String messageSelector) </li> </ul> | **Pris en charge** |
| Consommateur de messages/ <br/> JMSConsumer | <ul> <li> JMSContext.createConsumer( Destination destination) </li> <li> JMSContext.createConsumer( Destination destination, String messageSelector) </li> <li> JMSContext.createConsumer( Destination destination, String messageSelector, boolean noLocal)</li> </ul>  <br/> noLocal n'est actuellement pas pris en charge | **Pris en charge** |
| Abonnements durables partagés | <ul> <li> JMSContext.createSharedDurableConsumer(Topic topic, String name) </li> <li> JMSContext.createSharedDurableConsumer(Topic topic, String name, String messageSelector) </li> </ul>| **Pris en charge**|
| Abonnements durables non partagés | <ul> <li> JMSContext.createDurableConsumer(Topic topic, String name) </li> <li> createDurableConsumer(Topic topic, String name, String messageSelector, boolean noLocal) </li> </ul> <br/> noLocal n'est actuellement pas pris en charge et doit être défini sur false | **Pris en charge** |
| Abonnements non durables partagés |<ul> <li> JMSContext.createSharedConsumer(Topic topic, String sharedSubscriptionName) </li> <li> JMSContext.createSharedConsumer(Topic topic, String sharedSubscriptionName, String messageSelector) </li> </ul> | **Pris en charge** |
| Abonnements non durables non partagés |<ul> <li> JMSContext.createConsumer(Destination destination) </li> <li> JMSContext.createConsumer( Destination destination, String messageSelector) </li> <li> JMSContext.createConsumer( Destination destination, String messageSelector, boolean noLocal) </li> </ul> <br/> noLocal n'est actuellement pas pris en charge et doit être défini sur false | **Pris en charge** |
| Sélecteurs de messages | dépend du consommateur créé | **Pris en charge** |
| Délai de livraison (messages planifiés) | <ul> <li> JMSProducer.setDeliveryDelay( long deliveryDelay) </li> </ul>|**Pris en charge**|
| Message créé |<ul> <li> JMSContext.createMessage() </li> <li> JMSContext.createBytesMessage() </li> <li> JMSContext.createMapMessage() </li> <li> JMSContext.createObjectMessage( Serializable object) </li> <li> JMSContext.createStreamMessage() </li> <li> JMSContext.createTextMessage() </li> <li> JMSContext.createTextMessage( String text) </li> </ul>| **Pris en charge** |
| Transactions distribuées || Non pris en charge |
