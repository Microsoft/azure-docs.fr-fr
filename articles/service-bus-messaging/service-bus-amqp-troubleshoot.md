---
title: Résoudre des erreurs AMQP dans Azure Service Bus | Microsoft Docs
description: Fournit une liste des erreurs AMQP qui peuvent s’afficher lors de l’utilisation d’Azure Service Bus, ainsi que la cause de ces erreurs.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 88b10940e0b910f50e6ccf7f8c53134fa7f0ba2f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88064347"
---
# <a name="amqp-errors-in-azure-service-bus"></a>Erreurs AMQP dans Azure Service Bus
Cet article énumère certaines des erreurs qui s’affichent lors de l’utilisation d’AMQP avec Azure Service Bus. Il s’agit de comportements standard du service. Vous pouvez les éviter en établissant des appels d’envoi ou de réception sur la connexion/liaison, ce qui a pour effet de recréer celles-ci automatiquement.

## <a name="link-is-closed"></a>La liaison est fermée 
L’erreur suivante s’affiche lorsque la connexion et la liaison AMQP sont actives mais qu’aucun appel (d’envoi ou de réception) n’est établi à l’aide de la liaison pendant 10 minutes. La liaison est alors fermée. Mais la connexion est toujours ouverte.

```
amqp:link:detach-forced:The link 'G2:7223832:user.tenant0.cud_00000000000-0000-0000-0000-00000000000000' is force detached by the broker due to errors occurred in publisher(link164614). Detach origin: AmqpMessagePublisher.IdleTimerExpired: Idle timeout: 00:10:00. TrackingId:00000000000000000000000000000000000000_G2_B3, SystemTracker:mynamespace:Topic:MyTopic, Timestamp:2/16/2018 11:10:40 PM
```

## <a name="connection-is-closed"></a>La connexion est fermée
L’erreur suivante s’affiche sur la connexion AMQP lorsque toutes les liaisons de celle-ci ont été fermées parce qu’il n’y a eu aucune activité et qu’aucun nouveau lien n’a été créé en l’espace de 5 minutes.

```
Error{condition=amqp:connection:forced, description='The connection was inactive for more than the allowed 300000 milliseconds and is closed by container 'LinkTracker'. TrackingId:00000000000000000000000000000000000_G21, SystemTracker:gateway5, Timestamp:2019-03-06T17:32:00', info=null}
```

## <a name="link-is-not-created"></a>La liaison n’est pas créée 
Cette erreur s’affiche quand une nouvelle connexion AMQP est créée, mais qu’aucune liaison n’est créée dans la minute suivant la création de la connexion AMQP.

```
Error{condition=amqp:connection:forced, description='The connection was inactive for more than the allowed 60000 milliseconds and is closed by container 'LinkTracker'. TrackingId:0000000000000000000000000000000000000_G21, SystemTracker:gateway5, Timestamp:2019-03-06T18:41:51', info=null}
```

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur AMQP et Service Bus, consultez les liens suivants :

* [Vue d’ensemble du protocole AMQP de Service Bus]
* [Guide du protocole AMQP 1.0]
* [AMQP in Service Bus for Windows Server (AMQP dans Service Bus pour Windows Server)]

[Vue d’ensemble du protocole AMQP de Service Bus]: service-bus-amqp-overview.md
[Guide du protocole AMQP 1.0]: service-bus-amqp-protocol-guide.md
[AMQP in Service Bus for Windows Server (AMQP dans Service Bus pour Windows Server)]: /previous-versions/service-bus-archive/dn282144(v=azure.100)