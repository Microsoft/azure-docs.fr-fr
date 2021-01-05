---
title: Azure App Configuration en tant que source Event Grid
description: Cet article explique comment utiliser Azure App Configuration en tant que source d’événement Event Grid. Il fournit le schéma et des liens vers des articles de procédure et des tutoriels.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: bdd077c291bd1e1c441217740daf39c8bcaad732
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86116996"
---
# <a name="azure-app-configuration-as-an-event-grid-source"></a>Azure App Configuration en tant que source Event Grid
Cet article fournit les propriétés et le schéma des événements Azure App Configuration. Pour une présentation des schémas d’événements, consultez [Schéma d’événements Azure Event Grid](event-schema.md). Cet article fournit également une liste de guides de démarrage rapide et de tutoriels permettant d’utiliser Azure App Configuration en tant que source d’événement.

## <a name="event-grid-event-schema"></a>Schéma d’événement Event Grid

### <a name="available-event-types"></a>Types d’événement disponibles

Azure App Configuration propose les types d’événements suivants :

| Type d'événement | Description |
| ---------- | ----------- |
| Microsoft.AppConfiguration.KeyValueModified | Émis lorsqu’une valeur de clé est créée ou remplacée. |
| Microsoft.AppConfiguration.KeyValueDeleted | Émis lorsqu’une valeur de clé est supprimée. |

### <a name="example-event"></a>Exemple d’événement

L’exemple suivant montre le schéma d’un événement de valeur de clé modifié : 

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

Le schéma d’un événement de valeur de clé supprimé est similaire : 

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
  "eventType": "Microsoft.AppConfiguration.KeyValueDeleted",
  "eventTime": "2019-05-31T20:05:03Z",
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```
 
### <a name="event-properties"></a>Propriétés d’événement

Un événement contient les données générales suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| topic | string | Chemin d’accès complet à la source de l’événement. Ce champ n’est pas modifiable. Event Grid fournit cette valeur. |
| subject | string | Chemin de l’objet de l’événement, défini par le serveur de publication. |
| eventType | string | Un des types d’événements inscrits pour cette source d’événement. |
| eventTime | string | L’heure à quelle l’événement est généré selon l’heure UTC du fournisseur. |
| id | string | Identificateur unique de l’événement. |
| data | object | Données de l’événement App Configuration. |
| dataVersion | string | Version du schéma de l’objet de données. Le serveur de publication définit la version du schéma. |
| metadataVersion | string | Version du schéma des métadonnées d’événement. Event Grid définit le schéma des propriétés de niveau supérieur. Event Grid fournit cette valeur. |

L’objet de données comporte les propriétés suivantes :

| Propriété | Type | Description |
| -------- | ---- | ----------- |
| key | string | La clé de la paire clé-valeur qui a été modifiée ou supprimée. |
| label | string | Le cas échéant, l’étiquette de la paire clé-valeur qui a été modifiée ou supprimée. |
| etag | string | Pour le paramètre `KeyValueModified`, l’etag de la nouvelle paire clé-valeur. Pour le paramètre `KeyValueDeleted`, l’etag de la paire clé-valeur qui a été supprimée. |

## <a name="tutorials-and-how-tos"></a>Tutoriels et articles de procédures

|Intitulé | Description |
|---------|---------|
| [Réagir aux événements Azure App Configuration à l’aide d’Event Grid](../azure-app-configuration/concept-app-configuration-event.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Vue d’ensemble de l’intégration d’Azure App Configuration avec Event Grid. |
| [Démarrage rapide : Router des événements Azure App Configuration vers un point de terminaison web personnalisé avec Azure CLI](../azure-app-configuration/howto-app-configuration-event.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Montre comment utiliser Azure CLI pour envoyer des événements Azure App Configuration à un webhook. |

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation d’Azure Event Grid, consultez [Présentation d’Event Grid](overview.md).
* Pour plus d’informations sur la création d’un abonnement Azure Event Grid, consultez [Schéma d’abonnement à Event Grid](subscription-creation-schema.md).
* Pour découvrir les événements Azure App Configuration et savoir comment les utiliser, voir [Démarrage rapide : Router des événements Azure App Configuration vers un point de terminaison web avec Azure CLI](../azure-app-configuration/howto-app-configuration-event.md?toc=%2fazure%2fevent-grid%2ftoc.json). 