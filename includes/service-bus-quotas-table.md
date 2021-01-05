---
title: Fichier include
description: Fichier include
services: service-bus-messaging
author: spelluru
ms.service: service-bus-messaging
ms.topic: include
ms.date: 07/15/2020
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 764d75872eb1bad0647235cd70f9984ebd7789bd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96025473"
---
Le tableau suivant répertorie les informations de quota propres à la messagerie Azure Service Bus. Pour plus d’informations sur la tarification et d’autres quotas pour Service Bus, voir la [Tarification Service Bus](https://azure.microsoft.com/pricing/details/service-bus/) .

| Nom du quota | Étendue | Notes | Valeur |
| --- | --- | --- | --- |
| Nombre maximal d’espaces de noms de base/standard par abonnement Azure |Espace de noms |Les demandes suivantes d’espaces de noms de base/standard supplémentaires sont rejetées par le portail Azure. |100|
| Nombre maximal d’espaces de noms premium par abonnement Azure |Espace de noms |Les demandes suivantes d’espaces de noms premium supplémentaires sont rejetées par le portail. |100 |
| Taille de la file d’attente ou de la rubrique |Entité |Définie lors de la création de la file d’attente/rubrique. <br/><br/> Les messages entrants suivants sont rejetés et le code appelant reçoit une exception. |1, 2, 3, 4 ou 5 Go.<br /><br />Dans la référence SKU Premium, ainsi que la référence SKU Standard avec le [partitionnement](../articles/service-bus-messaging/service-bus-partitioning.md) activé, la taille maximale de file d’attente/rubrique est de 80 Go. |
| Nombre de connexions simultanées sur un espace de noms |Espace de noms |Les demandes suivantes de connexions supplémentaires sont rejetées et le code appelant reçoit une exception. Les opérations REST ne sont pas comptées parmi les connexions TCP simultanées. |Messagerie Net : 1 000.<br /><br />AMQP : 5 000. |
| Nombre de demandes de réception simultanées sur une entité de file d’attente/rubrique/abonnement |Entité |Les demandes de réception suivantes sont rejetées et le code appelant reçoit une exception. Ce quota s’applique au nombre combiné d’opérations de réception simultanées sur tous les abonnements à une rubrique. |5 000 |
| Nombre de rubriques/files d’attente par espace de noms |Espace de noms |Les requêtes suivantes de création de rubrique ou de file d’attente dans l’espace de noms sont rejetées. Par conséquent, en cas de configuration via le [portail Azure][Azure portal], un message d’erreur est généré. Si elle est appelée à partir de l’API de gestion, une exception est reçue par le code appelant. |10 000 pour le niveau De base ou Standard. Le nombre total de rubriques et de files d’attente dans un espace de noms doit être inférieur ou égal à 10 000. <br/><br/>Pour le niveau Premium, 1 000 par unité de messagerie. La limite maximale est 4 000. |
| Nombre de [rubriques/files d’attente partitionnées](../articles/service-bus-messaging/service-bus-partitioning.md) par espace de noms |Espace de noms |Les requêtes suivantes de création de rubrique ou de file d’attente partitionnée dans l’espace de noms sont rejetées. Par conséquent, en cas de configuration via le [portail Azure][Azure portal], un message d’erreur est généré. Si elle est appelée à partir de l’API de gestion, l’exception **QuotaExceededException** est reçue par le code appelant. |Niveaux de base et standard : 100.<br/><br/>Les entités partitionnées ne sont pas prises en charge dans le niveau [Premium](../articles/service-bus-messaging/service-bus-premium-messaging.md).<br/><br />Chaque file d’attente ou rubrique partitionnée est comptabilisée dans le quota de 1 000 entités par espace de noms. |
| Taille maximale de tout chemin d’entité de messagerie : file d’attente ou rubrique |Entité |- |260 caractères. |
| Taille maximale de tout nom d’entité de messagerie : espace de noms, abonnement ou règle d’abonnement |Entité |- |50 caractères. |
| Taille maximale de l’[ID d’un message](/dotnet/api/microsoft.azure.servicebus.message.messageid) | Entité |- | 128 |
| Taille maximale de l’[ID de session](/dotnet/api/microsoft.azure.servicebus.message.sessionid) d’un message | Entité |- | 128 |
| Taille de message d’une entité de file d’attente/rubrique/abonnement |Entité |Les messages entrants dont la taille dépasse ces quotas sont rejetés et le code appelant reçoit une exception. |Taille de message maximale : 256 Ko pour le [niveau Standard](../articles/service-bus-messaging/service-bus-premium-messaging.md), 1 Mo pour le [niveau Premium](../articles/service-bus-messaging/service-bus-premium-messaging.md). <br /><br />En raison d’une surcharge du système, cette limite est inférieure à ces valeurs.<br /><br />Taille d’en-tête maximale : 64 Ko.<br /><br />Nombre maximal de propriétés d’en-tête dans le conteneur de propriétés : **byte/int.MaxValue**.<br /><br />Taille maximale d’une propriété dans le conteneur de propriétés : pas de limite explicite. Cette valeur est limitée par la taille d’en-tête maximale. |
| Taille de propriété de message d’une entité de file d’attente/rubrique/abonnement |Entité | L’exception `SerializationException` est générée. |La taille de propriété de message maximale pour chaque propriété est de 32 000. La taille cumulée de toutes les propriétés ne peut pas dépasser 64 000. La limite s’applique à l’intégralité de l’en-tête du paramètre [Brokered Message](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage), qui inclut les propriétés de l’utilisateur ainsi que les propriétés système telles que [Sequence Number](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.sequencenumber), [Label](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.label), [Message ID](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.messageid). |
| Nombre d’abonnements par rubrique |Entité |Les requêtes suivantes de création d’abonnements supplémentaires à la rubrique sont rejetées. Par conséquent, en cas de configuration via le portail, un message d’erreur est affiché. Si elle est appelée à partir de l’API de gestion, une exception est reçue par le code appelant. |2 000 par rubrique pour le niveau Standard et le niveau Premium. |
| Nombre de filtres SQL par rubrique |Entité |Les requêtes suivantes de création de filtres supplémentaires sur la rubrique sont rejetées et le code appelant reçoit une exception. |2 000 |
| Nombre de filtres de corrélation par rubrique |Entité |Les requêtes suivantes de création de filtres supplémentaires sur la rubrique sont rejetées et le code appelant reçoit une exception. |100 000 |
| Taille d’actions/filtres SQL |Espace de noms |Les requêtes suivantes de création de filtres supplémentaires sont rejetées et le code appelant reçoit une exception. |Longueur maximale de la chaîne de condition de filtre : 1 024 (1 Ko).<br /><br />Longueur maximale de la chaîne d’action de règle : 1 024 (1 Ko).<br /><br />Nombre maximal d’expressions par action de règle : 32. |
| Nombre de règles [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) par espace de noms, file d’attente ou rubrique |Entité, espace de noms |Les requêtes suivantes de création de règles supplémentaires sont rejetées et le code appelant reçoit une exception. |Nombre maximal de règles par type d’entité : 12. <br /><br /> Les règles qui sont configurées sur un espace de noms Service Bus s’appliquent à tous les types : files d’attente, rubriques. |
| Nombre de messages par transaction | Transaction | Les messages entrants supplémentaires sont rejetés et une exception indiquant « Impossible d’envoyer plus de 100 messages dans une transaction unique » est reçue par le code appelant. | 100 <br /><br /> Pour les deux operations **Send()** et **SendAsync()** . |
| Nombre de règles de réseau virtuel et de filtre IP | Espace de noms | &nbsp; | 128 | 

[Azure portal]: https://portal.azure.com