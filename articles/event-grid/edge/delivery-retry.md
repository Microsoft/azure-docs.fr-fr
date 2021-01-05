---
title: Livraison et nouvelle tentative – Azure Event Grid sur IoT Edge | Microsoft Docs
description: Livraison et nouvelle tentative dans Event Grid sur IoT Edge.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: aa0b3a05fb26f6be951b697145d7b22e03b7792d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86171616"
---
# <a name="delivery-and-retry"></a>Livraison et nouvelle tentative

Event Grid assure une distribution fiable. Il tente de livrer chaque message au moins une fois immédiatement pour chaque abonnement correspondant. Si le point de terminaison d’un abonné n’accuse pas réception d’un événement ou en cas de défaillance, Event Grid effectue une nouvelle tentative de livraison conformément à une **planification** et à une **stratégie de nouvelles tentatives** fixes.  Par défaut, le module Event Grid fournit un événement à la fois à l’abonné. La charge utile est cependant un tableau avec un seul événement. Vous pouvez faire en sorte que le module fournisse plusieurs événements à la fois en activant la fonctionnalité de traitement par lot des sorties. Pour plus de détails sur cette fonctionnalité, voir [Traitement par lot des sorties](delivery-output-batching.md).  

> [!IMPORTANT]
>Il n’existe aucune pris en charge de la persistance pour des données d’événement. Cela signifie que le redéploiement ou le redémarrage du module Event Grid entraîne la perte des événements non encore livrés.

## <a name="retry-schedule"></a>Planification des nouvelles tentatives

Event Grid attend une réponse pendant jusqu’à 60 secondes après la livraison d’un message. Si le point de terminaison de l’abonné n’accuse pas réception de la réponse, le message est placé dans l’une de nos files d’attente d’interruption en vue de nouvelles tentatives.

Il existe deux files d’attente d’interruption préconfigurées qui déterminent la planification d’une nouvelle tentative. Il s'agit de :

| Planifier | Description |
| ---------| ------------ |
| 1 minute | La livraison des messages qui se terminent ici est tentée à chaque minute.
| 10 minutes | La livraison des messages qui se terminent ici est tentée toutes les 10 minutes.

### <a name="how-it-works"></a>Fonctionnement

1. Le message arrive dans le module Event Grid. Une tentative de livraison immédiate est effectuée.
1. Si la livraison échoue, le message est placé dans une file d’attente de 1 minute à l’issue de laquelle sa livraison est retentée.
1. Si la livraison échoue, le message est placé dans une file d’attente de 10 minutes à l’issue desquelles sa livraison est retentée toutes les 10 minutes.
1. Les livraisons sont tentées jusqu’à ce que la livraison réussisse ou que les limites de stratégie de nouvelles tentatives soient atteintes.

## <a name="retry-policy-limits"></a>Limites de stratégie de nouvelles tentatives

Deux configurations déterminent la stratégie de nouvelles tentatives. Il s'agit de :

* Nombre maximal de tentatives
* Durée de vie (TTL) de l’événement

Un événement est abandonné si l’une des limites de la stratégie de nouvelles tentatives est atteinte. La planification des nouvelles tentatives est décrite dans la section Planification des nouvelles tentatives. La configuration de ces limites peut être effectuée pour tous les abonnés ou par abonnement. La section suivante décrit chacune d’elles de façon plus détaillées.

## <a name="configuring-defaults-for-all-subscribers"></a>Configuration des valeurs par défaut pour tous les abonnés

Il existe deux propriétés, `brokers__defaultMaxDeliveryAttempts` et `broker__defaultEventTimeToLiveInSeconds`, qui peuvent être configurées dans le cadre du déploiement d’Event Grid, qui contrôlent les valeurs par défaut de la stratégie de nouvelles tentatives pour tous les abonnés.

| Nom de la propriété | Description |
| ---------------- | ------------ |
| `broker__defaultMaxDeliveryAttempts` | Nombre maximal de tentatives de livraison d’un événement. Valeur par défaut : 30.
| `broker__defaultEventTimeToLiveInSeconds` | Durée de vie de l’événement exprimée en secondes, après laquelle un événement est supprimé s’il n’a pas été livré. Valeur par défaut : **7 200** secondes

## <a name="configuring-defaults-per-subscriber"></a>Configuration des valeurs par défaut par abonné

Vous pouvez également spécifier des limites de stratégie de nouvelles tentatives pour chaque abonnement.
Pour plus d’informations sur la façon de configurer les valeurs par défaut par abonné, voir la [documentation sur l’API](api.md). Les valeurs par défaut de niveau d’abonnement remplacent les configurations au niveau du module.

## <a name="examples"></a>Exemples

L’exemple suivant configure une stratégie de nouvelles tentatives dans le module Event Grid avec maxNumberOfAttempts = 3 et TTL de l’événement de 30 minutes

```json
{
  "Env": [
    "broker__defaultMaxDeliveryAttempts=3",
    "broker__defaultEventTimeToLiveInSeconds=1800"
  ],
  "HostConfig": {
    "PortBindings": {
      "4438/tcp": [
        {
          "HostPort": "4438"
        }
      ]
    }
  }
}
```

L’exemple suivant configure un abonnement Webhook avec maxNumberOfAttempts = 3 et TTL de l’événement de 30 minutes

```json
{
 "properties": {
  "destination": {
   "endpointType": "WebHook",
   "properties": {
    "endpointUrl": "<your_webhook_url>",
    "eventDeliverySchema": "eventgridschema"
   }
  },
  "retryPolicy": {
   "eventExpiryInMinutes": 30,
   "maxDeliveryAttempts": 3
  }
 }
}
```
