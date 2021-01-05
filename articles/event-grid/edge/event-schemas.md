---
title: Schémas d’événements – Azure Event Grid IoT Edge | Microsoft Docs
description: Schémas d’événements dans Event Grid sur IoT Edge.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: ea36c40f2038d016afb0c45944a98d4d90df6240
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86171565"
---
# <a name="event-schemas"></a>Schémas d’événement

Le module Event Grid accepte et remet les événements au format JSON. Trois schémas sont actuellement pris en charge par Event Grid :

* **EventGridSchema**
* **CustomSchema**
* **CloudEventSchema**

Vous pouvez configurer le schéma auquel un serveur de publication doit se conformer pendant la création de la rubrique. S’il n’est pas spécifié, la valeur par défaut est **EventGridSchema**. Les événements non conformes au schéma attendu seront rejetés.

Les abonnés peuvent également configurer le schéma dans lequel ils souhaitent recevoir les événements. S’il n’est pas spécifié, la valeur par défaut est le schéma de la rubrique.
Actuellement, le schéma de remise aux abonnés doit correspondre au schéma d’entrée de la rubrique. 

## <a name="eventgrid-schema"></a>Schéma EventGrid

Le schéma Event Grid se compose d’un ensemble de propriétés requises auxquelles une entité de publication doit se conformer. Chaque serveur de publication doit remplir les champs de niveau supérieur.

```json
[
  {
    "topic": string,
    "subject": string,
    "id": string,
    "eventType": string,
    "eventTime": string,
    "data":{
      object-unique-to-each-publisher
    },
    "dataVersion": string,
    "metadataVersion": string
  }
]
```

### <a name="eventgrid-schema-properties"></a>Propriétés des schémas Event Grid

Tous les événements contiennent les données de niveau supérieur suivantes :

| Propriété | Type | Obligatoire | Description |
| -------- | ---- | ----------- |-----------
| topic | string | Non | Doit correspondre à la rubrique sur laquelle il est publié. S’il n’est pas spécifié, Event Grid le remplit avec le nom de la rubrique sur laquelle il est publié. |
| subject | string | Oui | Chemin de l’objet de l’événement, défini par le serveur de publication. |
| eventType | string | Oui | Type d’événement de cette source d’événements, par exemple, BlobCreated. |
| eventTime | string | Oui | L’heure à quelle l’événement est généré selon l’heure UTC du fournisseur. |
| id | string | Non | Identificateur unique de l’événement. |
| data | object | Non | Permet de capturer les données d’événement propres à l’entité de publication. |
| dataVersion | string | Oui | Version du schéma de l’objet de données. Le serveur de publication définit la version du schéma. |
| metadataVersion | string | Non | Version du schéma des métadonnées d’événement. Event Grid définit le schéma des propriétés de niveau supérieur. Event Grid fournit cette valeur. |

### <a name="example--eventgrid-schema-event"></a>Exemple – Événement de schéma EventGrid

```json
[
  {
    "id": "1807",
    "eventType": "recordInserted",
    "subject": "myapp/vehicles/motorcycles",
    "eventTime": "2017-08-10T21:03:07+00:00",
    "data": {
      "make": "Ducati",
      "model": "Monster"
    },
    "dataVersion": "1.0"
  }
]
```

## <a name="customevent-schema"></a>Schéma CustomEvent

Dans un schéma personnalisé, il n’existe aucune propriété obligatoire appliquée comme le schéma EventGrid. L’entité de publication peut contrôler entièrement le schéma d’événement, ce qui offre une flexibilité maximale et donne accès à des scénarios dans lesquels un système basé sur les événements est déjà en place et qui consistent à réutiliser des événements existants ou à éviter d’être lié à un schéma spécifique.

### <a name="custom-schema-properties"></a>Propriétés des schémas personnalisés

Aucune propriété obligatoire. C’est à l’entité de publication de déterminer la charge utile.

### <a name="example--custom-schema-event"></a>Exemple – Événement de schéma personnalisé

```json
[
  {
    "eventdata": {
      "make": "Ducati",
      "model": "Monster"
    }
  }
]
```

## <a name="cloudevent-schema"></a>Schéma CloudEvents

En plus des schémas ci-dessus, Azure Event Grid prend nativement en charge les événements dans le [schéma JSON CloudEvents](https://github.com/cloudevents/spec/blob/master/json-format.md). CloudEvents est une spécification ouverte de description des données d’événement. Elle simplifie l’interopérabilité en proposant un schéma d’événement commun pour la publication et la consommation des événements. Elle fait partie de la [CNCF](https://www.cncf.io/) ; la version actuellement disponible est 1.0-rc1.

### <a name="cloudevent-schema-properties"></a>Propriétés des schémas CloudEvents

Pour connaître les propriétés d’enveloppe obligatoires, voir [Spécification CloudEvents](https://github.com/cloudevents/spec/blob/master/json-format.md#3-envelope).

### <a name="example--cloud-event"></a>Exemple – CloudEvents
```json
[{
       "id": "1807",
       "type": "recordInserted",
       "source": "myapp/vehicles/motorcycles",
       "time": "2017-08-10T21:03:07+00:00",
       "datacontenttype": "application/json",
       "data": {
            "make": "Ducati",
            "model": "Monster"
        },
        "dataVersion": "1.0",
        "specVersion": "1.0-rc1"
}]
```
