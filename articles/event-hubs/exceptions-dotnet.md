---
title: Azure Event Hubs – Exceptions .NET
description: Cet article donne la liste des exceptions de messages .NET Azure Event Hubs et les actions suggérées.
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
manager: timlt
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.custom: seodec18
ms.date: 09/23/2020
ms.author: shvija
ms.openlocfilehash: 6a1d7c969d31033ae9d00d212cc8f1a45abbeda1
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91339042"
---
# <a name="eventhubsexception---net"></a>EventHubsException – .NET
Une **EventHubsException** est déclenchée lorsqu’une opération propre à Event Hubs a provoqué un problème, y compris des erreurs au sein du service et propres au client. 

## <a name="exception-information"></a>Informations sur l’exception
L’exception contient les informations contextuelles suivantes pour vous aider à comprendre le contexte de l’erreur et sa gravité relative. 

- **IsTransient** : identifie si l’exception est considérée comme récupérable ou non. Dans le cas où elle a été jugée temporaire, la stratégie de nouvelles tentatives appropriée a déjà été appliquée et les nouvelles tentatives ont échoué.
- **Motif** : donne un ensemble de raisons connues de l’échec qui permettent de catégoriser et de clarifier la cause racine. Ces raisons visent à permettre l’application du filtrage des exceptions et d’autres logiques dans le cas où l’inspection du texte d’un message d’exception ne serait pas idéale. Voici quelques-unes des principales raisons de l’échec :
    - **Client fermé** : se produit lorsqu’un client Event Hubs a déjà été fermé ou supprimé. Nous vous recommandons de vérifier, dans le code de l’application, que les objets de la bibliothèque de client Event Hubs sont créés et fermés dans l’étendue prévue.
    - **Délai d’expiration du service** : indique que le service Event Hubs n’a pas répondu à une opération dans le laps de temps attendu. Ce problème peut être dû à un problème temporaire de réseau ou de service. On ne sait pas si le service Event Hubs a pu effectuer la demande, car l’état n’est pas connu. Il est recommandé d’essayer de vérifier l’état actuel et de réessayer si nécessaire.
    - **Quota dépassé** : indique qu’il y a trop d’opérations de lecture actives pour un seul groupe de consommateurs. Cette limite dépend du niveau de l’espace de noms Event Hubs. Il peut être nécessaire de passer à un niveau supérieur. Une autre solution consiste à créer des groupes de consommateurs supplémentaires et à veiller à ce que le nombre de lectures des clients consommateurs respecte la limite pour chaque groupe. Pour plus d’informations, consultez [Quotas et limites Azure Event Hubs](event-hubs-quotas.md).
    - **Taille des messages dépassée** : les données d’événement présentent une taille maximale autorisée pour un événement individuel et un lot d’événements. Sont comprises les données de l’événement ainsi que les métadonnées associées et la surcharge du système. Pour résoudre cette erreur, réduisez le nombre d’événements envoyés dans un lot ou la taille des données incluses dans le message. Les limites de taille étant sujettes à modification, consultez [Quotas et limites Azure Event Hubs](event-hubs-quotas.md) pour plus de précisions.
    - **Client déconnecté** : un client consommateur a été déconnecté de l’instance Event Hubs par le service Event Hubs. Cette situation se produit généralement lorsqu’un consommateur d’un niveau de propriétaire supérieur revendique la propriété d’un jumelage entre une partition et un groupe de consommateurs.
    - **Ressource introuvable** : le service Event Hubs n’a pas trouvé une certaine ressource (par exemple, Event Hub, groupe de consommateurs ou partition). Elle a peut-être été supprimée. Un problème a également pu se produire avec le service Event Hubs proprement dit.

## <a name="handling-exceptions"></a>Gestion des exceptions
Il existe plusieurs façons de réagir à une raison d’échec pour **EventHubException**. L’une d’elles consiste à appliquer une clause de filtre d’exception dans le bloc catch.

```csharp
try
{
    // Read events using the consumer client
}
catch (EventHubsException ex) where 
    (ex.Reason == EventHubsException.FailureReason.ConsumerDisconnected)
{
    // Take action based on a consumer being disconnected
}
```

## <a name="next-steps"></a>Étapes suivantes
D’autres exceptions sont documentées dans [l’article hérité](event-hubs-messaging-exceptions.md). Certaines d’entre elles s’appliquent uniquement à la bibliothèque de client .NET Event Hubs héritée.