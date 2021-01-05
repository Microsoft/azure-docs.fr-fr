---
title: Charges utiles de télémétrie, de propriétés et de commandes dans Azure IoT Central | Microsoft Docs
description: Les modèles d’appareil Azure IoT Central vous permettent de spécifier la données de télémétrie, les propriétés et les commandes qu’un appareil doit implémenter. Comprendre le format des données qu’un appareil peut échanger avec IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 12/19/2020
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.custom: device-developer
ms.openlocfilehash: c29af68433f29d7bdd363bedfa6d36316b952f4c
ms.sourcegitcommit: ab829133ee7f024f9364cd731e9b14edbe96b496
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/28/2020
ms.locfileid: "97795341"
---
# <a name="telemetry-property-and-command-payloads"></a>Charges utiles de télémétrie, de propriétés et de commandes

_Cet article s’applique aux développeurs d’appareils._

Un modèle d’appareil dans Azure IoT Central est un blueprint qui définit ce qui suit :

* Les données de télémétrie qu’un appareil envoie à IoT Central.
* Les propriétés qu’un appareil synchronise avec IoT Central.
* Les commandes qu’IoT Central appelle sur un appareil.

Cet article décrit, pour les développeurs d’appareils, les charges utiles JSON que les appareils envoient et reçoivent pour les données de télémétrie, les propriétés et les commandes définies dans un modèle d’appareil.

L’article ne décrit pas tous les types possibles de charge utile de télémétrie, de propriété et de commande, mais les exemples illustrent tous les types de clés.

Chaque exemple montre un extrait de code du modèle d’appareil qui définit le type et des exemples de charges utiles JSON pour illustrer comment l’appareil doit interagir avec l’application IoT Central.

> [!NOTE]
> IoT Central accepte un JSON valide, mais il ne peut être utilisé pour les visualisations que s’il correspond à une définition du modèle d’appareil. Pour exporter des données qui ne correspondent pas à une définition, consultez [Exporter des données IoT vers des destinations dans Azure](howto-export-data.md).

Le fichier JSON qui définit le modèle d’appareil utilise le [langage DTDL (Digital Twin Definition Language) v2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md).

Pour obtenir un exemple de code d’appareil illustrant l’utilisation de certaines de ces charges utiles, consultez le tutoriel [Créer et connecter une application cliente à votre application Azure IoT Central](tutorial-connect-device.md).

## <a name="view-raw-data"></a>Voir les données brutes

IoT Central vous permet d’afficher les données brutes qu’un appareil envoie à une application. Cette vue est utile pour résoudre les problèmes liés à la charge utile envoyée à partir d’un appareil. Pour afficher les données brutes envoyées par un appareil :

1. Accédez à l’appareil à partir de la page **Appareils**.

1. Sélectionnez l’onglet **Données brutes** :

    :::image type="content" source="media/concepts-telemetry-properties-commands/raw-data.png" alt-text="Vue Données brutes":::

    Dans cette vue, vous pouvez sélectionner les colonnes à afficher et définir une plage de temps à afficher. La colonne **Données démodélisées** affiche les données de l’appareil qui ne correspondent à aucune définition de propriété ou de télémétrie dans le modèle d’appareil.

## <a name="telemetry"></a>Télémétrie

### <a name="primitive-types"></a>Types primitifs

Cette section présente des exemples de types de données de télémétrie primitifs qu’un appareil transmet à une application IoT Central.

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de télémétrie `boolean` :

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "BooleanTelemetry"
  },
  "name": "BooleanTelemetry",
  "schema": "boolean"
}
```

Un client d’appareil doit envoyer la télémétrie au format JSON comme dans l’exemple suivant :

```json
{ "BooleanTelemetry": true }
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de télémétrie `string` :

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "StringTelemetry"
  },
  "name": "StringTelemetry",
  "schema": "string"
}
```

Un client d’appareil doit envoyer la télémétrie au format JSON comme dans l’exemple suivant :

```json
{ "StringTelemetry": "A string value - could be a URL" }
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de télémétrie `integer` :

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "IntegerTelemetry"
  },
  "name": "IntegerTelemetry",
  "schema": "integer"
}

```

Un client d’appareil doit envoyer la télémétrie au format JSON comme dans l’exemple suivant :

```json
{ "IntegerTelemetry": 23 }
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de télémétrie `double` :

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "DoubleTelemetry"
  },
  "name": "DoubleTelemetry",
  "schema": "double"
}
```

Un client d’appareil doit envoyer la télémétrie au format JSON comme dans l’exemple suivant :

```json
{ "DoubleTelemetry": 56.78 }
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de télémétrie `dateTime` :

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "DateTimeTelemetry"
  },
  "name": "DateTimeTelemetry",
  "schema": "dateTime"
}
```

Un client d’appareil doit envoyer la télémétrie au format JSON comme dans l’exemple suivant – les types `DateTime` doivent être au format ISO 8061 :

```json
{ "DateTimeTelemetry": "2020-08-30T19:16:13.853Z" }
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de télémétrie `duration` :

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "DurationTelemetry"
  },
  "name": "DurationTelemetry",
  "schema": "duration"
}
```

Un client d’appareil doit envoyer la télémétrie au format JSON comme dans l’exemple suivant – les durées doivent être au format ISO 8601 :

```json
{ "DurationTelemetry": "PT10H24M6.169083011336625S" }
```

### <a name="complex-types"></a>Types complexes

Cette section présente des exemples de types de données de télémétrie complexes qu’un appareil transmet à une application IoT Central.

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de télémétrie `geopoint` :

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "GeopointTelemetry"
  },
  "name": "GeopointTelemetry",
  "schema": "geopoint"
}
```

Un client d’appareil doit envoyer la télémétrie au format JSON comme dans l’exemple suivant. IoT Central affiche la valeur sous la forme d’une épingle sur une carte :

```json
{
  "GeopointTelemetry": {
    "lat": 47.64263,
    "lon": -122.13035,
    "alt": 0
  }
}
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de télémétrie `Enum` :

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "EnumTelemetry"
  },
  "name": "EnumTelemetry",
  "schema": {
    "@type": "Enum",
    "displayName": {
      "en": "Enum"
    },
    "valueSchema": "integer",
    "enumValues": [
      {
        "displayName": {
          "en": "Item1"
        },
        "enumValue": 0,
        "name": "Item1"
      },
      {
        "displayName": {
          "en": "Item2"
        },
        "enumValue": 1,
        "name": "Item2"
      },
      {
        "displayName": {
          "en": "Item3"
        },
        "enumValue": 2,
        "name": "Item3"
      }
    ]
  }
}
```

Un client d’appareil doit envoyer la télémétrie au format JSON comme dans l’exemple suivant. Les valeurs possibles sont `0`, `1` et `2`. Elles s’affichent dans IoT Central en tant que `Item1`, `Item2` et `Item3` :

```json
{ "EnumTelemetry": 1 }
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de télémétrie `Object`. Cet objet a trois champs avec des types `dateTime`, `integer` et `Enum` :

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "ObjectTelemetry"
  },
  "name": "ObjectTelemetry",
  "schema": {
    "@type": "Object",
    "displayName": {
      "en": "Object"
    },
    "fields": [
      {
        "displayName": {
          "en": "Property1"
        },
        "name": "Property1",
        "schema": "dateTime"
      },
      {
        "displayName": {
          "en": "Property2"
        },
        "name": "Property2",
        "schema": "integer"
      },
      {
        "displayName": {
          "en": "Property3"
        },
        "name": "Property3",
        "schema": {
          "@type": "Enum",
          "displayName": {
            "en": "Enum"
          },
          "valueSchema": "integer",
          "enumValues": [
            {
              "displayName": {
                "en": "Item1"
              },
              "enumValue": 0,
              "name": "Item1"
            },
            {
              "displayName": {
                "en": "Item2"
              },
              "enumValue": 1,
              "name": "Item2"
            },
            {
              "displayName": {
                "en": "Item3"
              },
              "enumValue": 2,
              "name": "Item3"
            }
          ]
        }
      }
    ]
  }
}
```

Un client d’appareil doit envoyer la télémétrie au format JSON comme dans l’exemple suivant. Les types `DateTime` doivent être conformes à ISO 8061. Les valeurs possibles pour `Property3` sont `0` et `1`. Elles s’affichent dans IoT Central en tant que `Item1`, `Item2` et `Item3` :

```json
{
  "ObjectTelemetry": {
      "Property1": "2020-09-09T03:36:46.195Z",
      "Property2": 37,
      "Property3": 2
  }
}
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de télémétrie `vector` :

```json
{
  "@type": "Telemetry",
  "displayName": {
    "en": "VectorTelemetry"
  },
  "name": "VectorTelemetry",
  "schema": "vector"
}
```

Un client d’appareil doit envoyer la télémétrie au format JSON comme dans l’exemple suivant :

```json
{
  "VectorTelemetry": {
    "x": 74.72395045538597,
    "y": 74.72395045538597,
    "z": 74.72395045538597
  }
}
```

### <a name="event-and-state-types"></a>Types d’événements et d’états

Cette section présente des exemples d’événements et d’états de télémétrie qu’un appareil transmet à une application IoT Central.

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type d'événement `integer` :

```json
{
  "@type": [
    "Telemetry",
    "Event"
  ],
  "displayName": {
    "en": "IntegerEvent"
  },
  "name": "IntegerEvent",
  "schema": "integer"
}
```

Un client d’appareil doit envoyer les données de l’événement au format JSON comme dans l’exemple suivant :

```json
{ "IntegerEvent": 74 }
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type d'état `integer` :

```json
{
  "@type": [
    "Telemetry",
    "State"
  ],
  "displayName": {
    "en": "IntegerState"
  },
  "name": "IntegerState",
  "schema": {
    "@type": "Enum",
    "valueSchema": "integer",
    "enumValues": [
      {
        "displayName": {
          "en": "Level1"
        },
        "enumValue": 1,
        "name": "Level1"
      },
      {
        "displayName": {
          "en": "Level2"
        },
        "enumValue": 2,
        "name": "Level2"
      },
      {
        "displayName": {
          "en": "Level3"
        },
        "enumValue": 3,
        "name": "Level3"
      }
    ]
  }
}
```

Un client d’appareil doit envoyer l’état au format JSON comme dans l’exemple suivant. Les valeurs d’état entières possibles sont `1`, `2` ou `3` :

```json
{ "IntegerState": 2 }
```

## <a name="properties"></a>Propriétés

> [!NOTE]
> Les formats de charge utile pour les propriétés s’appliquent aux applications créées le ou après le 14/07/2020.

### <a name="primitive-types"></a>Types primitifs

Cette section présente des exemples de types de données de propriétés primitifs qu’un appareil envoie à une application IoT Central.

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de propriété `boolean` :

```json
{
  "@type": "Property",
  "displayName": {
    "en": "BooleanProperty"
  },
  "name": "BooleanProperty",
  "schema": "boolean",
  "writable": false
}
```

Un client d’appareil doit envoyer une charge utile JSON ressemblant à l’exemple suivant en tant que propriété signalée dans le jumeau d'appareil :

```json
{ "BooleanProperty": false }
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de propriété `boolean` :

```json
{
  "@type": "Property",
  "displayName": {
    "en": "LongProperty"
  },
  "name": "LongProperty",
  "schema": "long",
  "writable": false
}
```

Un client d’appareil doit envoyer une charge utile JSON ressemblant à l’exemple suivant en tant que propriété signalée dans le jumeau d'appareil :

```json
{ "LongProperty": 439 }
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de propriété `date` :

```json
{
  "@type": "Property",
  "displayName": {
    "en": "DateProperty"
  },
  "name": "DateProperty",
  "schema": "date",
  "writable": false
}
```

Un client d’appareil doit envoyer une charge utile JSON ressemblant à l’exemple suivant en tant que propriété signalée dans le jumeau d'appareil. Les types `Date` doivent être conformes à ISO 8061 :

```json
{ "DateProperty": "2020-05-17" }
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de propriété `duration` :

```json
{
  "@type": "Property",
  "displayName": {
    "en": "DurationProperty"
  },
  "name": "DurationProperty",
  "schema": "duration",
  "writable": false
}
```

Un client d’appareil doit envoyer une charge utile JSON ressemblant à l’exemple suivant en tant que propriété signalée dans le jumeau d'appareil – les durées doivent être conformes à la durée selon ISO 8601 :

```json
{ "DurationProperty": "PT10H24M6.169083011336625S" }
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de propriété `float` :

```json
{
  "@type": "Property",
  "displayName": {
    "en": "FloatProperty"
  },
  "name": "FloatProperty",
  "schema": "float",
  "writable": false
}
```

Un client d’appareil doit envoyer une charge utile JSON ressemblant à l’exemple suivant en tant que propriété signalée dans le jumeau d'appareil :

```json
{ "FloatProperty": 1.9 }
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de propriété `string` :

```json
{
  "@type": "Property",
  "displayName": {
    "en": "StringProperty"
  },
  "name": "StringProperty",
  "schema": "string",
  "writable": false
}
```

Un client d’appareil doit envoyer une charge utile JSON ressemblant à l’exemple suivant en tant que propriété signalée dans le jumeau d'appareil :

```json
{ "StringProperty": "A string value - could be a URL" }
```

### <a name="complex-types"></a>Types complexes

Cette section présente des exemples de types de données de propriétés complexes qu’un appareil envoie à une application IoT Central.

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de propriété `geopoint` :

```json
{
  "@type": "Property",
  "displayName": {
    "en": "GeopointProperty"
  },
  "name": "GeopointProperty",
  "schema": "geopoint",
  "writable": false
}
```

Un client d’appareil doit envoyer une charge utile JSON ressemblant à l’exemple suivant en tant que propriété signalée dans le jumeau d'appareil :

```json
{
  "GeopointProperty": {
    "lat": 47.64263,
    "lon": -122.13035,
    "alt": 0
  }
}
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de propriété `Enum` :

```json
{
  "@type": "Property",
  "displayName": {
    "en": "EnumProperty"
  },
  "name": "EnumProperty",
  "writable": false,
  "schema": {
    "@type": "Enum",
    "displayName": {
      "en": "Enum"
    },
    "valueSchema": "integer",
    "enumValues": [
      {
        "displayName": {
          "en": "Item1"
        },
        "enumValue": 0,
        "name": "Item1"
      },
      {
        "displayName": {
          "en": "Item2"
        },
        "enumValue": 1,
        "name": "Item2"
      },
      {
        "displayName": {
          "en": "Item3"
        },
        "enumValue": 2,
        "name": "Item3"
      }
    ]
  }
}
```

Un client d’appareil doit envoyer une charge utile JSON ressemblant à l’exemple suivant en tant que propriété signalée dans le jumeau d'appareil. Les valeurs possibles sont `0` et `1`. Elles s’affichent dans IoT Central en tant que `Item1`, `Item2` et `Item3` :

```json
{ "EnumProperty": 1 }
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de propriété `Object`. Cet objet a deux champs avec des types `string` et `integer` :

```json
{
  "@type": "Property",
  "displayName": {
    "en": "ObjectProperty"
  },
  "name": "ObjectProperty",
  "writable": false,
  "schema": {
    "@type": "Object",
    "displayName": {
      "en": "Object"
    },
    "fields": [
      {
        "displayName": {
          "en": "Field1"
        },
        "name": "Field1",
        "schema": "integer"
      },
      {
        "displayName": {
          "en": "Field2"
        },
        "name": "Field2",
        "schema": "string"
      }
    ]
  }
}
```

Un client d’appareil doit envoyer une charge utile JSON ressemblant à l’exemple suivant en tant que propriété signalée dans le jumeau d'appareil :

```json
{
  "ObjectProperty": {
    "Field1": 37,
    "Field2": "A string value"
  }
}
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de propriété `vector` :

```json
{
  "@type": "Property",
  "displayName": {
    "en": "VectorProperty"
  },
  "name": "VectorProperty",
  "schema": "vector",
  "writable": false
}
```

Un client d’appareil doit envoyer une charge utile JSON ressemblant à l’exemple suivant en tant que propriété signalée dans le jumeau d'appareil :

```json
{
  "VectorProperty": {
    "x": 74.72395045538597,
    "y": 74.72395045538597,
    "z": 74.72395045538597
  }
}
```

### <a name="writeable-property-types"></a>Types de propriétés modifiables

Cette section présente des exemples de types de propriétés modifiables qu’un appareil reçoit d’une application IoT Central.

IoT Central attend une réponse de l’appareil pour les mises à jour des propriétés modifiables. Le message de réponse doit inclure les champs `ac` et `av`. Le champ `ad` est facultatif. Pour obtenir des exemples, consultez les extraits de code suivants.

`ac` est un champ numérique qui utilise les valeurs du tableau suivant :

| Valeur | Étiquette | Description |
| ----- | ----- | ----------- |
| `'ac': 200` | Completed | L’opération de modification de la propriété a été effectuée. |
| `'ac': 202` ou `'ac': 201` | Pending | L’opération de modification de la propriété est en attente ou en cours |
| `'ac': 4xx` | Error | La modification de la propriété demandée n’est pas valide ou a rencontré une erreur |
| `'ac': 5xx` | Error | L’appareil a rencontré une erreur inattendue lors du traitement de la modification demandée. |

`av` est le numéro de version envoyé à l’appareil.

`ad` est une description de chaîne d’option.

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de propriété modifiable `string` :

```json
{
  "@type": "Property",
  "displayName": {
    "en": "StringPropertyWritable"
  },
  "name": "StringPropertyWritable",
  "writable": true,
  "schema": "string"
}
```

L’appareil reçoit la charge utile suivante d’IoT Central :

```json
{  
  "StringPropertyWritable": "A string from IoT Central", "$version": 7
}
```

L’appareil doit envoyer la charge utile JSON suivante à IoT Central une fois qu’il a traité la mise à jour. Ce message comprend le numéro de version de la mise à jour d’origine reçue d’IoT Central. Lorsqu’IoT Central reçoit ce message, il marque la propriété comme  **synchronisée** dans l’interface utilisateur :

```json
{
  "StringPropertyWritable": {
    "value": "A string from IoT Central",
    "ac": 200,
    "ad": "completed",
    "av": 7
  }
}
```

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de propriété modifiable `Enum` :

```json
{
  "@type": "Property",
  "displayName": {
    "en": "EnumPropertyWritable"
  },
  "name": "EnumPropertyWritable",
  "writable": true,
  "schema": {
    "@type": "Enum",
    "displayName": {
      "en": "Enum"
    },
    "valueSchema": "integer",
    "enumValues": [
      {
        "displayName": {
          "en": "Item1"
        },
        "enumValue": 0,
        "name": "Item1"
      },
      {
        "displayName": {
          "en": "Item2"
        },
        "enumValue": 1,
        "name": "Item2"
      },
      {
        "displayName": {
          "en": "Item3"
        },
        "enumValue": 2,
        "name": "Item3"
      }
    ]
  }
}
```

L’appareil reçoit la charge utile suivante d’IoT Central :

```json
{  
  "EnumPropertyWritable":  1 , "$version": 10
}
```

L’appareil doit envoyer la charge utile JSON suivante à IoT Central une fois qu’il a traité la mise à jour. Ce message comprend le numéro de version de la mise à jour d’origine reçue d’IoT Central. Lorsqu’IoT Central reçoit ce message, il marque la propriété comme  **synchronisée** dans l’interface utilisateur :

```json
{
  "EnumPropertyWritable": {
    "value": 1,
    "ac": 200,
    "ad": "completed",
    "av": 10
  }
}
```

## <a name="commands"></a>Commandes

> [!NOTE]
> Dans l’interface utilisateur web IoT Central, vous pouvez sélectionner l’option **Mettre en file d'attente si hors connexion** pour une commande. Ce paramètre n’est pas inclus si vous exportez un modèle ou une interface à partir du modèle d’appareil.

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’une commande qui n’a pas de paramètres et n’attend pas que l’appareil retourne quoi que ce soit :

```json
{
  "@type": "Command",
  "displayName": {
    "en": "CommandBasic"
  },
  "name": "CommandBasic"
}
```

L’appareil reçoit une charge utile vide dans la requête et doit retourner une charge utile vide dans la réponse avec un code de réponse HTTP `200` pour indiquer la réussite de l’opération.

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’une commande qui a un paramètre entier et attend que l’appareil retourne une valeur entière :

```json
{
  "@type": "Command",
  "request": {
    "@type": "CommandPayload",
    "displayName": {
      "en": "RequestParam"
    },
    "name": "RequestParam",
    "schema": "integer"
  },
  "response": {
    "@type": "CommandPayload",
    "displayName": {
      "en": "ResponseParam"
    },
    "name": "ResponseParam",
    "schema": "integer"
  },
  "displayName": {
    "en": "CommandSimple"
  },
  "name": "CommandSimple"
}
```

L’appareil reçoit une valeur entière en tant que charge utile de la requête. L’appareil doit retourner une valeur entière en tant que charge utile de réponse avec un code de réponse HTTP `200` pour indiquer la réussite.

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’une commande qui a un paramètre objet et attend que l’appareil retourne un objet. Dans cet exemple, les deux objets ont des champs des types entier et chaîne :

```json
{
  "@type": "Command",
  "request": {
    "@type": "CommandPayload",
    "displayName": {
      "en": "RequestParam"
    },
    "name": "RequestParam",
    "schema": {
      "@type": "Object",
      "displayName": {
        "en": "Object"
      },
      "fields": [
        {
          "displayName": {
            "en": "Field1"
          },
          "name": "Field1",
          "schema": "integer"
        },
        {
          "displayName": {
            "en": "Field2"
          },
          "name": "Field2",
          "schema": "string"
        }
      ]
    }
  },
  "response": {
    "@type": "CommandPayload",
    "displayName": {
      "en": "ResponseParam"
    },
    "name": "ResponseParam",
    "schema": {
      "@type": "Object",
      "displayName": {
        "en": "Object"
      },
      "fields": [
        {
          "displayName": {
            "en": "Field1"
          },
          "name": "Field1",
          "schema": "integer"
        },
        {
          "displayName": {
            "en": "Field2"
          },
          "name": "Field2",
          "schema": "string"
        }
      ]
    }
  },
  "displayName": {
    "en": "CommandComplex"
  },
  "name": "CommandComplex"
}
```

L’extrait de code suivant montre un exemple de charge utile de requête envoyée à l’appareil :

```json
{ "Field1": 56, "Field2": "A string value" }
```

L’extrait de code suivant montre un exemple de charge utile de réponse envoyée par l’appareil. Utilisez un code de réponse HTTP `200` pour indiquer la réussite :

```json
{ "Field1": 87, "Field2": "Another string value" }
```

### <a name="long-running-commands"></a>Commandes de longue durée

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’une commande. La commande a un paramètre entier et attend que l’appareil retourne une valeur entière :

```json
{
  "@type": "Command",
  "request": {
    "@type": "CommandPayload",
    "displayName": {
      "en": "RequestParam"
    },
    "name": "RequestParam",
    "schema": "integer"
  },
  "response": {
    "@type": "CommandPayload",
    "displayName": {
      "en": "ResponseParam"
    },
    "name": "ResponseParam",
    "schema": "integer"
  },
  "displayName": {
    "en": "LongRunningCommandSimple"
  },
  "name": "LongRunningCommandSimple"
}
```

L’appareil reçoit une valeur entière en tant que charge utile de la requête. Si l’appareil a besoin de temps pour traiter cette commande, il doit retourner une charge utile de réponse vide avec un code de réponse HTTP `202` pour indiquer que l’appareil a accepté la requête de traitement.

Une fois que l’appareil a fini de traiter la requête, il doit envoyer à IoT Central une propriété qui ressemble à l’exemple suivant. Le nom de la propriété doit être identique au nom de la commande :

```json
{
  "LongRunningCommandSimple": {
    "value": 87
  }
}
```

## <a name="next-steps"></a>Étapes suivantes

En tant que développeur d’appareils, à présent que vous avez appris ce que sont les modèles d’appareils, nous vous suggérons de lire [Se connecter à Azure IoT Central](./concepts-get-connected.md) pour en savoir plus sur l’inscription des appareils auprès d’IoT Central et sur la manière dont IoT Central sécurise les connexions des appareils.
