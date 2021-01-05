---
title: Utilisation des propriétés dans une solution Azure IoT Central
description: Découvrez comment utiliser les propriétés en lecture seule et accessibles en écriture dans une solution Azure IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 11/06/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: a517f7a796b6543c8d60f0d1ebdba16afa0bc4b7
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96751425"
---
# <a name="use-properties-in-an-azure-iot-central-solution"></a>Utilisation des propriétés dans une solution Azure IoT Central

Cet article explique comment utiliser les propriétés de l’appareil qui sont définies dans un modèle d’appareil dans votre application Azure IoT Central.

Les propriétés représentent des valeurs à un moment donné. Par exemple, un appareil peut utiliser une propriété pour indiquer la température cible qu’il tente d’atteindre. Par défaut, les propriétés de l’appareil sont en lecture seule dans IoT Central. Les propriétés accessibles en écriture vous permettent de synchroniser l’état entre votre appareil et votre application Azure IoT Central.

Vous pouvez également définir des propriétés de cloud dans une application Azure IoT Central. Les valeurs de propriétés de cloud ne sont jamais échangées avec un appareil et ne sont pas concernées par cet article.

## <a name="define-your-properties"></a>Définir vos propriétés

Les propriétés sont des champs de données qui représentent l’état de votre appareil. Utilisez les propriétés pour représenter l’état durable de l’appareil, par exemple l’état marche/arrêt d’un appareil. Les propriétés peuvent également représenter des propriétés d’appareil de base, telles que la version logicielle de l’appareil. Les propriétés sont déclarées en lecture seule ou accessibles en écriture.

La capture d’écran suivante montre une définition de propriété dans l’application Azure IoT Central.

:::image type="content" source="media/howto-use-properties/property-definition.png" alt-text="Capture d’écran montrant une définition de propriété dans une application Azure IoT Central.":::

Le tableau suivant décrit les paramètres de configuration d’une fonctionnalité de données de propriété.

| Champ           | Description                                                                                                                                                                                                                        |
|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Nom complet    | Nom complet de la valeur de propriété utilisée sur les tableaux de bord et les formulaires.                                                                                                                                                              |
| Nom            | Nom de la propriété. Azure IoT Central génère une valeur pour ce champ à partir du nom d’affichage, mais vous pouvez choisir votre propre valeur si nécessaire. Ce champ doit être alphanumérique.                                                 |
| Type de fonctionnalité | Propriété.                                                                                                                                                                                                                          |
| Type sémantique   | Type sémantique de la propriété, telles que la température, l’état ou l’événement. Le choix du type sémantique détermine lequel des champs suivants est disponible.                                                                       |
| schéma          | Type de données de propriété, tel que double, chaîne ou vecteur. Les options disponibles sont déterminées par le type sémantique. Le schéma n’est pas disponible pour les types sémantiques d’événement et d’état.                                               |
| Accessible en écriture       | Si la propriété n’est pas inscriptible, l’appareil peut signaler des valeurs de propriété à Azure IoT Central. Si la propriété est inscriptible, l’appareil peut signaler des valeurs de propriété à Azure IoT Central. Azure IoT Central peut alors envoyer des mises à jour de propriétés à l’appareil. |
| severity        | Disponible uniquement pour le type sémantique d’événement. Les gravités sont **Erreur**, **Information** ou **Avertissement**.                                                                                                                         |
| Valeurs d’état    | Disponible uniquement pour le type sémantique d’état. Définissez les valeurs d’état possibles, chacune ayant un nom d’affichage, un nom, un type d’énumération et une valeur.                                                                                   |
| Unité            | Unité pour la valeur des propriétés, telles que **mph**, **%** ou **&deg;C**.                                                                                                                                                              |
| Unité d'affichage    | Unité d’affichage à utiliser sur les tableaux de bord et les formulaires.                                                                                                                                                                                    |
| Commentaire         | Commentaires sur la fonctionnalité de propriété.                                                                                                                                                                                        |
| Description     | Description de la fonctionnalité de propriété.                                                                                                                                                                                          |

Les propriétés peuvent également être définies dans une interface dans un modèle d’appareil comme indiqué ici :

``` json
{
  "@type": [
    "Property",
    "Temperature"
  ],
  "name": "targetTemperature",
  "schema": "double",
  "displayName": "Target Temperature",
  "description": "Allows to remotely specify the desired target temperature.",
  "unit" : "degreeCelsius",
  "writable": true
},
{
  "@type": [
    "Property",
    "Temperature"
  ],
  "name": "maxTempSinceLastReboot",
  "schema": "double",
  "unit" : "degreeCelsius",
  "displayName": "Max temperature since last reboot.",
  "description": "Returns the max temperature since last device reboot."
}
```

Cet exemple montre deux propriétés. Ces propriétés sont liées à la définition de propriété dans l’interface utilisateur :

* `@type` spécifie le type de capacité : `Property`. L’exemple précédent montre également le type sémantique `Temperature` pour les deux propriétés.
* `name` pour la propriété.
* `schema` spécifie le type de données pour la propriété. Cette valeur peut être un type primitif, tel que double, entier, valeur booléenne ou chaîne. Les types d’objets complexes et les mappages sont également pris en charge.
* `writable`. Par défaut, les propriétés sont en lecture seule. Vous pouvez marquer une propriété comme étant accessible en écriture, à l’aide de ce champ.

Des champs facultatifs, tels que le nom d’affichage et la description, vous permettent d’ajouter des détails à l’interface et aux fonctionnalités.

Lorsque vous créez une propriété, vous pouvez spécifier des types de schéma complexes tels que **Objet** et **Enum**.

![Capture d’écran montrant comment ajouter une fonctionnalité.](./media/howto-use-properties/property.png)

Lorsque vous sélectionnez le **Schéma** complexe, par exemple **Objet**, vous devez également définir l’objet.

:::image type="content" source="media/howto-use-properties/object.png" alt-text="Capture d’écran montrant comment définir un objet":::

Le code suivant illustre la définition d’un type de propriété Objet. Cet objet a deux champs dont les types sont « string » et « integer ».

``` json
{
  "@type": "Property",
  "displayName": {
    "en": "ObjectProperty"
  },
  "name": "ObjectProperty",
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

## <a name="implement-read-only-properties"></a>Implémenter des propriétés en lecture seule

Par défaut, les propriétés sont en lecture seule. Les propriétés en lecture seule permettent à un appareil de signaler des mises à jour de valeurs de propriété à votre application Azure IoT Central. Votre application Azure IoT Central ne peut pas définir la valeur d’une propriété en lecture seule.

Azure IoT Central utilise les jumeaux d’appareil pour synchroniser les valeurs de propriété entre l’appareil et l’application Azure IoT Central. Les valeurs de propriété d’appareil utilisent les propriétés signalées du jumeau d’appareil. Pour plus d’informations, consultez la documentation relative aux [jumeaux d’appareil](../../iot-hub/tutorial-device-twins.md).

L’extrait de code suivant d’un modèle d’appareil montre la définition d’un type de propriété en lecture seule :

``` json
{
  "name": "model",
  "displayName": "Device model",
  "schema": "string",
  "comment": "Device model name or ID. Ex. Surface Book 2."
}
```

Les mises à jour de propriété sont envoyées par un appareil sous la forme d’une charge utile JSON. Pour plus d’informations, consultez [Charges utiles](./concepts-telemetry-properties-commands.md).

Vous pouvez utiliser l’Azure IoT device SDK pour envoyer une mise à jour de propriété à votre application Azure IoT Central.

Les propriétés de jumeaux d’appareil peuvent être envoyées à votre application Azure IoT Central à l’aide de la fonction suivante :

``` javascript
hubClient.getTwin((err, twin) => {
    const properties = {
        model: 'environmentalSensor1.0'
    };
    twin.properties.reported.update(properties, (err) => {
        console.log(err ? `error: ${err.toString()}` : `status: success` )
    });
});
```

Cet article utilise Node.js pour des raisons de simplicité. Pour obtenir des informations complètes sur les exemples d’application d’appareil, consultez le tutoriel [Créer et connecter une application cliente à votre application Azure IoT Central](tutorial-connect-device.md).

La vue suivante dans l’application Azure IoT Central affiche les propriétés que vous pouvez voir. La vue définit automatiquement la propriété **Modèle d’appareil** comme une _propriété d’appareil en lecture seule_.

:::image type="content" source="media/howto-use-properties/read-only.png" alt-text="Capture d’écran montrant la vue d’une propriété en lecture seule":::

## <a name="implement-writable-properties"></a>Implémenter des propriétés accessibles en écriture

Les propriétés accessibles en écriture sont définies par un opérateur dans l’application Azure IoT Central sur un formulaire. Azure IoT Central envoie la propriété à l’appareil. Azure IoT Central attend un accusé de réception de la part de l’appareil.

L’extrait de code suivant d’un modèle d’appareil illustre la définition d’un type de propriété accessible en écriture :

``` json
{
  "@type": "Property",
  "displayName": "Brightness Level",
  "description": "The brightness level for the light on the device. Can be specified as 1 (high), 2 (medium), 3 (low)",
  "name": "brightness",
  "writable": true,
  "schema": "long"
}
```

Pour définir et gérer les propriétés accessibles en écriture auxquelles votre appareil répond, vous pouvez utiliser le code suivant :

``` javascript
hubClient.getTwin((err, twin) => {
    twin.on('properties.desired.brightness', function(desiredBrightness) {
        console.log( `Received setting: ${desiredBrightness.value}` );
        var patch = {
            brightness: {
                value: desiredBrightness.value,
                ad: 'success',
                ac: 200,
                av: desiredBrightness.$version
            }
        }
        twin.properties.reported.update(patch, (err) => {
            console.log(err ? `error: ${err.toString()}` : `status: success` )
        });
    });
});
```

Le message de réponse doit inclure les champs `ac` et `av`. Le champ `ad` est facultatif. Pour obtenir des exemples, consultez les extraits de code suivants :

* `ac` est un champ numérique qui utilise les valeurs du tableau suivant.
* `av` est le numéro de version envoyé à l’appareil.
* `ad` est une description de chaîne d’option.

| Valeur | Étiquette | Description |
| ----- | ----- | ----------- |
| `'ac': 200` | Completed | L’opération de modification de la propriété a été effectuée. |
| `'ac': 202` ou `'ac': 201` | Pending | L’opération de modification de la propriété est en attente ou en cours. |
| `'ac': 4xx` | Erreur | La modification de la propriété demandée n’est pas valide ou a rencontré une erreur. |
| `'ac': 5xx` | Error | L’appareil a rencontré une erreur inattendue lors du traitement de la modification demandée. |

Pour plus d’informations sur les jumeaux d'appareil, consultez [Configurer vos appareils à partir d’un service backend](../../iot-hub/tutorial-device-twins.md).

Quand l’opérateur définit une propriété inscriptible dans l’application Azure IoT Central, l’application utilise une propriété souhaitée du jumeau d’appareil pour envoyer la valeur à l’appareil. L’appareil répond alors en utilisant une propriété signalée du jumeau d’appareil. Quand Azure IoT Central reçoit la valeur de la propriété rapportée, il met à jour l’affichage des propriétés avec l’état **Accepté**.

L’affichage suivant montre les propriétés accessibles en écriture. Lorsque vous entrez la valeur et sélectionnez **Enregistrer**, l’état initial est **En attente**. Lorsque l’appareil accepte la modification, l’état passe à **Accepté**.

![Capture d’écran montrant l’état En attente.](./media/howto-use-properties/status-pending.png)

![Capture d’écran montrant la propriété Accepté.](./media/howto-use-properties/accepted.png)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à utiliser les propriétés dans votre application Azure IoT Central, consultez :

* [Payloads](concepts-telemetry-properties-commands.md)
* [Créer et connecter une application cliente à votre application Azure IoT Central](tutorial-connect-device.md)