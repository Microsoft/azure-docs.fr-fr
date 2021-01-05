---
title: Protéger les applications Azure Service Bus contre les pannes et les sinistres
description: Cet article présente des techniques permettant de protéger les applications contre une éventuelle défaillance d'Azure Service Bus.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 4f3ff89e3ec59ad4445ab0b7ee7eeb45d18fa3b8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88065622"
---
# <a name="best-practices-for-insulating-applications-against-service-bus-outages-and-disasters"></a>Meilleures pratiques pour protéger les applications contre les pannes de Service Bus et les sinistres

Les applications stratégiques doivent fonctionner en permanence, même en cas de pannes non planifiées ou de sinistres. Cette rubrique décrit les techniques que vous pouvez utiliser pour protéger les applications Service Bus contre une panne de service ou un sinistre potentiel.

Une panne est définie comme l'indisponibilité temporaire d'Azure Service Bus. La panne peut affecter certains composants de Service Bus, comme une banque de messagerie ou même le centre de données entier. Une fois le problème résolu, Service Bus redevient disponible. En règle générale, une panne n'entraîne pas la perte de messages ou autres données. L'indisponibilité d'une banque de messagerie spécifique constitue un exemple de défaillance d'un composant. Une panne d'alimentation ou un commutateur réseau défaillant constituent des exemples de panne au niveau du centre de données. Une panne peut durer de quelques minutes à plusieurs jours.

Un sinistre est défini comme la perte définitive d'une unité d'échelle ou d'un centre de données Service Bus. Le centre de données peut devenir disponible à nouveau ou pas. En général, un sinistre provoque la perte d'une partie ou de l'ensemble des messages ou autres données. Les incendies, les inondations ou les tremblements de terre sont des exemples de sinistres.

## <a name="protecting-against-outages-and-disasters---service-bus-premium"></a>Protection contre les pannes et les sinistres - Service Bus Premium
Les concepts liés à la haute disponibilité et à la reprise d’activité sont intégrés directement au niveau Premium d’Azure Service Bus, aussi bien dans la même région (par le biais des zones de disponibilité) que dans des régions différentes (par le biais de la géo-reprise d’activité après sinistre).

### <a name="geo-disaster-recovery"></a>Géo-reprise d’activité après sinistre

Service Bus Premium prend en charge la géo-reprise d’activité après sinistre au niveau de l’espace de noms. Pour plus d’informations, consultez [Géorécupération d’urgence Azure Service Bus](service-bus-geo-dr.md). La fonctionnalité de récupération d’urgence, disponible pour la [référence SKU Premium](service-bus-premium-messaging.md) uniquement, implémente la récupération d’urgence des métadonnées, en s’appuyant sur les espaces de noms de récupération d’urgence principal et secondaire.

### <a name="availability-zones"></a>Zones de disponibilité

La référence SKU de Service Bus Premium prend en charge les [zones de disponibilité](../availability-zones/az-overview.md), en fournissant des emplacements isolés des pannes au sein d’une même région Azure. Service Bus gère trois copies de la banque de messagerie (1 copie principale et 2 secondaires). Service Bus synchronise les opérations relatives aux données et à la gestion sur les trois copies. Si la copie principale échoue, l’une des copies secondaires devient la copie principale, sans temps d’arrêt ressenti. Si les applications constatent des déconnexions temporaires de Service Bus, la logique de nouvelle tentative dans le SDK se reconnectera automatiquement à Service Bus. 

> [!NOTE]
> Dans Azure Service Bus Premium, la prise en charge des zones de disponibilité est fournie uniquement pour les [régions Azure](../availability-zones/az-region.md) qui comprennent déjà des zones de disponibilité.

Vous pouvez activer les Zones de disponibilité sur les nouveaux espaces de noms uniquement, à l’aide du portail Azure. Service Bus ne prend pas en charge la migration des espaces de noms existants. Vous ne pouvez pas désactiver la redondance de zone après l’avoir activée sur votre espace de noms.

![1][]


## <a name="protecting-against-outages-and-disasters---service-bus-standard"></a>Protection contre les pannes et les sinistres - Service Bus Standard
Pour obtenir une résilience contre les pannes du centre de données lors de l’utilisation du niveau tarifaire de messagerie standard, Service Bus propose deux approches : la réplication *active* et la réplication *passive*. Pour chaque approche, si une file d’attente ou une rubrique donnée doit rester accessible en cas de panne du centre de données, vous pouvez la créer dans les deux espaces de noms. Les deux entités peuvent avoir le même nom. Par exemple, une file d’attente principale peut être accessible sous **contosoPrimary.servicebus.windows.net/myQueue**, alors que son homologue secondaire peut être accessible sous **contosoSecondary.servicebus.windows.net/myQueue**.

>[!NOTE]
> La **réplication active** et la **réplication passive** sont des solutions à usage général et non des fonctionnalités propres à Service Bus. La logique de réplication (c’est-à-dire l’envoi à deux espaces de noms différents) se trouve dans les applications de l’expéditeur. Le destinataire doit avoir une logique personnalisée pour la détection des doublons.

Si l'application ne nécessite pas une communication permanente entre l'expéditeur et le destinataire, l'application peut implémenter une file d'attente côté client durable pour empêcher la perte de messages et protéger l'expéditeur contre toute erreur Service Bus temporaire.

### <a name="active-replication"></a>Réplication active
La réplication active utilise des entités dans les deux espaces de noms pour chaque opération. Tout client qui envoie un message envoie deux copies du même message. La première copie est envoyée à l’entité principale (par exemple, **contosoPrimary.servicebus.windows.net/sales**) et la deuxième copie du message est envoyée à l’entité secondaire (par exemple, **contosoSecondary.servicebus.windows.net/sales**).

Un client reçoit des messages des deux files d'attente. Le destinataire traite la première copie d'un message et la deuxième copie est supprimée. Pour supprimer les messages en double, l'expéditeur doit attribuer un identificateur unique à chaque message. Les deux copies du message doivent avoir le même identificateur. Vous pouvez utiliser les propriétés [BrokeredMessage.MessageId][BrokeredMessage.MessageId] ou [BrokeredMessage.Label][BrokeredMessage.Label] ou une propriété personnalisée, pour baliser le message. Le destinataire doit conserver une liste des messages qu'il a déjà reçus.

L’exemple de [géo-réplication avec le niveau Standard de Service Bus][Geo-replication with Service Bus Standard Tier] illustre la réplication active des entités de messagerie.

> [!NOTE]
> L'approche de réplication active double le nombre d'opérations ; par conséquent, cette approche peut mener à un coût plus élevé.
> 
> 

### <a name="passive-replication"></a>Réplication passive
En cas d'absence de panne, la réplication passive n'utilise qu'une seule des deux entités de messagerie. Un client envoie le message à l'entité active. Si l'opération sur l'entité active échoue avec un code d'erreur qui indique que le centre de données qui héberge l'entité active n'est pas disponible, le client envoie une copie du message à l'entité de sauvegarde. À ce stade, les entités active et de sauvegarde permutent leurs rôles : le client en charge de l'envoi considère l'ancienne entité active comme nouvelle entité de sauvegarde, et l'ancienne entité de sauvegarde comme nouvelle entité active. Si les deux opérations d'envoi échouent, les rôles des deux entités restent inchangés et une erreur est renvoyée.

Un client reçoit des messages des deux files d'attente. Étant donné qu'il est possible que le destinataire reçoive deux copies du même message, le destinataire doit supprimer les messages en double. Vous pouvez supprimer les doublons de la même façon que celle décrite pour la réplication active.

En général, la réplication passive est moins onéreuse que la réplication active, car dans la plupart des cas, une seule opération est effectuée. La latence, le débit et le coût monétaire sont identiques au scénario de la non réplication.

Lorsque vous utilisez la réplication passive, les messages peuvent être perdus ou reçus deux fois, dans les scénarios suivants :

* **Retard ou perte de message** : Supposons que l’expéditeur a envoyé avec succès un message m1 à la file d’attente principale, et qu’ensuite la file d'attente devient indisponible avant que le destinataire ne reçoive m1. L'expéditeur envoie un message ultérieur m2 à la file d'attente secondaire. Si la file d'attente principale est temporairement indisponible, le destinataire reçoit m1 lorsque la file d'attente est à nouveau disponible. En cas de sinistre, le destinataire peut ne jamais recevoir m1.
* **Réception de doublons** : Supposons que l'expéditeur envoie un message m à la file d'attente principale. Service Bus traite m avec succès mais ne parvient pas à envoyer une réponse. Après expiration de l'opération d'envoi, l'expéditeur envoie une copie identique de m à la file d'attente secondaire. Si le destinataire peut recevoir la première copie de m avant que la file d'attente principale ne devienne indisponible, le destinataire reçoit les deux copies de m approximativement au même moment. Si le destinataire ne peut pas recevoir la première copie de m avant que la file d'attente principale ne devienne indisponible, le destinataire ne reçoit initialement que la deuxième copie de m, mais reçoit ensuite une deuxième copie de m lorsque la file d'attente principale devient disponible.

L’exemple de [géo-réplication avec le niveau Standard de Service Bus][Geo-replication with Service Bus Standard Tier] illustre la réplication passive des entités de messagerie.

## <a name="protecting-relay-endpoints-against-datacenter-outages-or-disasters"></a>Protection des points de terminaison de relais contre les pannes ou les sinistres du centre de données
La géo-réplication des points de terminaison d’[Azure Relay](../azure-relay/relay-what-is-it.md) permet à un service qui expose un point de terminaison de relais d’être accessible en cas de pannes de Service Bus. Pour obtenir la géo-réplication, le service doit créer deux points de terminaison de relais dans des espaces de noms différents. Les espaces de noms doivent résider dans des centres de données différents et les deux points de terminaison doivent avoir des noms différents. Par exemple, un point de terminaison principal peut être accessible sous **contosoPrimary.servicebus.windows.net/myPrimaryService**, alors que son homologue secondaire peut être accessible sous **contosoSecondary.servicebus.windows.net/mySecondaryService**.

Le service écoute alors sur les deux points de terminaison et un client peut appeler le service par le biais de n'importe quel point de terminaison. Une application cliente choisit de façon aléatoire l'un des relais en tant que point de terminaison principal et envoie sa requête au point de terminaison actif. Si l'opération échoue avec un code d'erreur, cette défaillance indique que le point de terminaison de relais n'est pas disponible. L'application ouvre un canal vers le point de terminaison de sauvegarde et soumet la requête à nouveau. À ce stade, les points de terminaison actif et de sauvegarde permutent leurs rôles : l'application cliente considère l'ancien point de terminaison actif comme nouveau point de terminaison de sauvegarde, et l'ancien point de terminaison de sauvegarde comme nouveau point de terminaison actif. Si les deux opérations d'envoi échouent, les rôles des deux entités restent inchangés et une erreur est renvoyée.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d'informations sur la récupération d'urgence, consultez les articles suivants :

* [Géorécupération d’urgence Azure Service Bus](service-bus-geo-dr.md)
* [Continuité de l’activité Azure SQL Database][Azure SQL Database Business Continuity]
* [Conception d’applications résilientes pour Azure][Azure resiliency technical guidance]

[Service Bus Authentication]: service-bus-authentication-and-authorization.md
[Partitioned messaging entities]: service-bus-partitioning.md
[Asynchronous messaging patterns and high availability]: service-bus-async-messaging.md#failure-of-service-bus-within-an-azure-datacenter
[BrokeredMessage.MessageId]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
[BrokeredMessage.Label]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
[Geo-replication with Service Bus Standard Tier]: https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoReplication
[Azure SQL Database Business Continuity]:../azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview.md
[Azure resiliency technical guidance]: /azure/architecture/resiliency

[1]: ./media/service-bus-outages-disasters/az.png