---
title: Comment gérer les jumeaux numériques IoT Plug-and-Play
description: Comment gérer un appareil IoT Plug-and-Play à l’aide d’API de jumeau numérique
author: prashmo
ms.author: prashmo
ms.date: 07/20/2020
ms.topic: how-to
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 51bd073b6f84f3378b91091e04f2f278ea2f49b6
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95495267"
---
# <a name="manage-iot-plug-and-play-digital-twins"></a>Gérer les jumeaux numériques IoT Plug-and-Play

IoT Plug-and-Play prend en charge les opérations **Obtenir le jumeau numérique** et **Mettre à jour un jumeau numérique** pour gérer des représentations numériques. Vous pouvez utiliser les [API REST](/rest/api/iothub/service/digitaltwin) ou l’un des [Kits de développement logiciel (SDK) du service](libraries-sdks.md).

Au moment de la rédaction de cet article, la version d’API de jumeau numérique est `2020-09-30`.

## <a name="update-a-digital-twin"></a>Mettre à jour un jumeau numérique

Un appareil IoT Plug-and-Play implémente un modèle décrit par [Digital Twins Definition Language v2 (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl). Les développeurs de solutions peuvent utiliser l’**API de mise à jour d’un jumeau numérique** pour mettre à jour l’état du composant et les propriétés du jumeau numérique.

L’appareil IoT Plug-and-Play utilisé comme exemple dans cet article implémente le [modèle Contrôleur de température](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) avec les composants [Thermostat](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json).

L’extrait de code suivant montre la réponse à une requête **Obtenir le jumeau numérique** formatée comme un objet JSON. Pour en savoir plus sur le format du jumeau numérique, consultez [Comprendre les jumeaux numériques IoT Plug-and-Play](./concepts-digital-twin.md#digital-twin-json-format) :

```json
{
    "$dtId": "sample-device",
    "serialNumber": "alwinexlepaho8329",
    "thermostat1": {
        "maxTempSinceLastReboot": 25.3,
        "targetTemperature": 20.4,
        "$metadata": {
            "targetTemperature": {
                "desiredValue": 20.4,
                "desiredVersion": 4,
                "ackVersion": 4,
                "ackCode": 200,
                "ackDescription": "Successfully executed patch",
                "lastUpdateTime": "2020-07-17T06:11:04.9309159Z"
            },
            "maxTempSinceLastReboot": {
                "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
            }
        }
    },
    "$metadata": {
        "$model": "dtmi:com:example:TemperatureController;1",
        "serialNumber": {
            "lastUpdateTime": "2020-07-17T06:10:31.9609233Z"
        }
    }
}
```

Les jumeaux numériques vous permettent de mettre à jour l’intégralité d’un composant ou d’une propriété à l’aide d’un document [JSON Patch](http://jsonpatch.com/).

Par exemple, vous pouvez mettre à jour la propriété `targetTemperature` comme suit :

```json
[
    {
        "op": "add",
        "path": "/thermostat1/targetTemperature",
        "value": 21.4
    }
]
```

La mise à jour précédente définit la valeur souhaitée d’une propriété dans le niveau du composant correspondant `$metadata` comme indiqué dans l’extrait de code suivant. IoT Hub met à jour la version souhaitée de la propriété :

```json
"thermostat1": {
    "targetTemperature": 20.4,
    "$metadata": {
        "targetTemperature": {
            "desiredValue": 21.4,
            "desiredVersion": 5,
            "ackVersion": 4,
            "ackCode": 200,
            "ackDescription": "Successfully executed patch",
            "lastUpdateTime": "2020-07-17T06:11:04.9309159Z"
        }
    }
}
```

### <a name="add-replace-or-remove-a-component"></a>Ajouter, remplacer ou supprimer un composant

Les opérations au niveau du composant requièrent un marqueur `$metadata` d’objet vide dans la valeur.

Une opération d’ajout ou de remplacement de composant définit les valeurs souhaitées de toutes les propriétés fournies. Elle efface également les valeurs souhaitées pour toutes les propriétés accessibles en écriture non fournies avec la mise à jour.

La suppression d’un composant efface les valeurs souhaitées de toutes les propriétés accessibles en écriture existantes. Un appareil finit par synchroniser cette suppression et arrête de signaler les propriétés individuelles. Le composant est ensuite supprimé du jumeau numérique.

L’exemple JSON Patch suivant montre comment ajouter, remplacer ou supprimer un composant :

```json
[
    {
        "op": "add",
        "path": "/thermostat1",
        "value": {
            "targetTemperature": 21.4,
            "anotherWritableProperty": 42,
            "$metadata": {}
        }
    },
    {
        "op": "replace",
        "path": "/thermostat1",
        "value": {
            "targetTemperature": 21.4,
            "$metadata": {}
        }
    },
    {
        "op": "remove",
        "path": "/thermostat2"
    }
]
```

### <a name="add-replace-or-remove-a-property"></a>Ajouter, remplacer ou supprimer une propriété

Une opération d’ajout ou de remplacement définit la valeur souhaitée d’une propriété. L’appareil peut synchroniser l’état et signaler une mise à jour de la valeur, ainsi qu’un code `ack`, une version et une description.

La suppression d’une propriété efface la valeur souhaitée de la propriété si elle est définie. L’appareil peut alors cesser de signaler cette propriété et celle-ci est supprimée du composant. Si cette propriété est la dernière dans le composant, le composant est également supprimé.

L’exemple JSON Patch suivant montre comment ajouter, remplacer ou supprimer une propriété dans un composant :

```json
[
    {
        "op": "add",
        "path": "/thermostat1/targetTemperature",
        "value": 21.4
    },
    {
        "op": "replace",
        "path": "/thermostat1/anotherWritableProperty",
        "value": 42
    },
    {
        "op": "remove",
        "path": "/thermostat2/targetTemperature",
    }
]
```

### <a name="rules-for-setting-the-desired-value-of-a-digital-twin-property"></a>Règles de définition de la valeur souhaitée d’une propriété de jumeau numérique

**Nom**

Le nom d’un composant ou d’une propriété doit être un nom DTDL v2 valide.

Caractères autorisés : de a à z, de A à Z, de 0 à 9 (hors premier caractère) et trait de soulignement (hors premier et dernier caractère).

Un nom peut contenir entre 1 et 64 caractères.

**Valeur de la propriété**

La valeur doit être une [propriété DTDL v2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#property) valide.

Tous les types primitifs sont pris en charge. Parmi les types complexes, les enums, les mappages et les objets sont pris en charge. Pour plus d’informations, consultez [Schémas DTDL v2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#schemas).

Les propriétés ne prennent pas en charge les tableaux ni les schémas complexes avec un tableau.

Une profondeur maximale de cinq niveaux est prise en charge pour un objet complexe.

Tous les noms de champs dans un objet complexe doivent être des noms DTDL v2 valides.

Toutes les clés de mappage doivent être des noms DTDL v2 valides.

## <a name="troubleshoot-update-digital-twin-api-errors"></a>Résoudre les erreurs liées à l’API de mise à jour du jumeau numérique

L’API de jumeau numérique génère le message d’erreur générique suivant :

`ErrorCode:ArgumentInvalid;'{propertyName}' exists within the device twin and is not digital twin conformant property. Please refer to aka.ms/dtpatch to update this to be conformant.`

Si vous voyez cette erreur, assurez-vous que le correctif de mise à jour respecte les [règles de définition de la valeur souhaitée d’une propriété de jumeau numérique](#rules-for-setting-the-desired-value-of-a-digital-twin-property)

Lorsque vous mettez à jour un composant, assurez-vous que le [marqueur $metadata d’objet vide](#add-replace-or-remove-a-component) est défini.

Les mises à jour peuvent échouer si les valeurs rapportées d’un appareil ne sont pas conformes aux [conventions IoT Plug-and-Play](./concepts-convention.md#writable-properties).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert les jumeaux numériques, voici quelques ressources supplémentaires :

- [Interagir avec un appareil à partir de votre solution](quickstart-service.md)
- [API REST de jumeau numérique IoT](/rest/api/iothub/service/digitaltwin)
- [Explorateur Azure IoT](howto-use-iot-explorer.md)