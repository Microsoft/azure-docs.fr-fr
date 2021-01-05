---
title: Réaction aux événements de paires clé-valeur dans Azure App Configuration
description: Utilisez Azure Event Grid pour vous abonner aux événements de configuration d’application, qui permettent aux applications de réagir aux modifications des valeurs de clé sans nécessiter de code complexe.
services: azure-app-configuration,event-grid
author: jimmyca
ms.author: jimmyca
ms.date: 02/20/2020
ms.topic: article
ms.service: azure-app-configuration
ms.openlocfilehash: 640be797b2653f9e6c969306b7e2b99393b99c39
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92078202"
---
# <a name="reacting-to-azure-app-configuration-events"></a>Réaction aux événements Azure App Configuration

Les événements Azure App Configuration permettent aux applications de réagir aux modifications au niveau des paires clé-valeur. La méthode utilisée n’exige pas de faire appel à du code complexe ou à des services d’interrogation coûteux et inefficaces. Au lieu de cela, les événements sont envoyés (push) via [Azure Event Grid](https://azure.microsoft.com/services/event-grid/) aux abonnés, comme [Azure Functions](https://azure.microsoft.com/services/functions/), [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/), ou même à votre propre écouteur HTTP personnalisé. Vous paierez uniquement pour ce que vous utiliserez.

Les événements Azure App Configuration sont envoyés à Azure Event Grid qui fournit des services de livraison fiables à vos applications via des stratégies enrichies de nouvelle tentative et de livraison de lettres mortes. Pour plus d’informations, consultez [Distribution et nouvelle tentative de distribution de messages avec Azure Grid](../event-grid/delivery-and-retry.md).

Les scénarios courants d’événement App Configuration comprennent l’actualisation des configurations d’application, le déclenchement de déploiements et tout workflow axé sur la configuration. Lorsque les modifications sont peu fréquentes, mais que votre scénario requiert une réactivité immédiate, une architecture basée sur des événements peut être particulièrement efficace.

Examinons [Acheminer des événements Azure App Configuration vers un point de terminaison Web personnalisé - CLI](./howto-app-configuration-event.md) pour obtenir un exemple rapide. 

![Modèle de Event Grid](./media/event-grid-functional-model.png)

## <a name="available-azure-app-configuration-events"></a>Événements Azure App Configuration disponibles
Event Grid utilise les [abonnements aux événements](../event-grid/concepts.md#event-subscriptions) pour acheminer les messages d’événements vers les abonnés. Les abonnements aux événements Azure App Configuration peuvent inclure deux types d’événements :  

> |Nom de l'événement|Description|
> |----------|-----------|
> |`Microsoft.AppConfiguration.KeyValueModified`|Émis lorsqu’une paire clé-valeur est créée ou remplacée|
> |`Microsoft.AppConfiguration.KeyValueDeleted`|Émis lorsqu’une paire clé-valeur est supprimée|

## <a name="event-schema"></a>Schéma d’événement
Les événements Azure App Configuration contiennent toutes les informations dont vous avez besoin pour répondre aux modifications de vos données. Vous pouvez identifier un événement App Configuration, car la propriété eventType commence par « Microsoft.AppConfiguration ». Plus d’informations sur l’utilisation des propriétés d’événement Event Grid sont documentées dans [schéma d’événement Event Grid](../event-grid/event-schema.md).  

> |Propriété|Type|Description|
> |-------------------|------------------------|-----------------------------------------------------------------------|
> |topic|string|ID Azure Resource Manager complet de la configuration d’application qui émet l’événement.|
> |subject|string|L’URI de la paire clé-valeur qui est le sujet de l’événement.|
> |eventTime|string|La date et l’heure de l’événement sont générées au format ISO 8601.|
> |eventType|string|« Microsoft.AppConfiguration.KeyValueModified » ou « Microsoft.AppConfiguration.KeyValueDeleted ».|
> |Id|string|Un identificateur unique de cet événement.|
> |dataVersion|string|Version du schéma de l’objet de données.|
> |metadataVersion|string|Version du schéma des propriétés de niveau supérieur.|
> |data|object|Collecte des données d’événement spécifiques d’Azure App Configuration|
> |data.key|string|La clé de la paire clé-valeur qui a été modifiée ou supprimée.|
> |data.label|string|Le cas échéant, l’étiquette de la paire clé-valeur qui a été modifiée ou supprimée.|
> |data.etag|string|Pour le paramètre `KeyValueModified`, l’etag de la nouvelle paire clé-valeur. Pour le paramètre `KeyValueDeleted`, l’etag de la paire clé-valeur qui a été supprimée.|

Voici un exemple d’événement KeyValueModified :
```json
[{
  "id": "84e17ea4-66db-4b54-8050-df8f7763f87b",
  "topic": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/microsoft.appconfiguration/configurationstores/contoso",
  "subject": "https://contoso.azconfig.io/kv/Foo?label=FizzBuzz",
  "data": {
    "key": "Foo",
    "label": "FizzBuzz",
    "etag": "FnUExLaj2moIi4tJX9AXn9sakm0"
  },
  "eventType": "Microsoft.AppConfiguration.KeyValueModified",
  "eventTime": "2019-05-31T20:05:03Z",
  "dataVersion": "1",
  "metadataVersion": "1"
}]

```

Pour plus d’informations, consultez [Schéma d’événements Azure App Configuration](../event-grid/event-schema-app-configuration.md).

## <a name="practices-for-consuming-events"></a>Pratiques pour la consommation d’événements
Les applications qui gèrent les événements App Configuration doivent suivre les pratiques recommandées :
> [!div class="checklist"]
> * Plusieurs abonnements peuvent être configurés pour router les événements vers le même gestionnaire d’événements. Il ne faut donc pas supposer que les événements proviennent d’une source particulière. Vérifiez l’objet du message pour connaître l’instance App Configuration qui envoie l’événement.
> * Vérifiez l’eventType, et ne supposez pas que tous les événements reçus seront du type que vous attendez.
> * Utilisez les champs etag pour vérifier si vos informations sur les objets sont encore à jour.  
> * Utilisez les champs de séquence pour comprendre l’ordre des événements concernant un objet en particulier.
> * Le champ Objet vous permet d’accéder à la paire clé-valeur qui a été modifiée.


## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur Event Grid et essayer les événements Azure App Configuration :

- [À propos d’Event Grid](../event-grid/overview.md)
- [Router des événements Azure App Configuration vers un point de terminaison Web personnalisé](./howto-app-configuration-event.md)