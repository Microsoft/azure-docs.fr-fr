---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/19/2020
ms.openlocfilehash: 7658b5a51c9e24e5530114ebca7455a0ed3ea097
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95511353"
---
## <a name="model-id-announcement"></a>Annonce de l’ID de modèle

Pour annoncer l’ID de modèle, l’appareil doit l’inclure dans les informations de connexion :

```python
device_client = IoTHubDeviceClient.create_from_symmetric_key(
    symmetric_key=symmetric_key,
    hostname=registration_result.registration_state.assigned_hub,
    device_id=registration_result.registration_state.device_id,
    product_info=model_id,
)
```

> [!TIP]
> Pour les modules et IoT Edge, utilisez `IoTHubModuleClient` à la place de `IoTHubDeviceClient`.

## <a name="dps-payload"></a>Charge utile DPS

Les appareils qui utilisent [Device Provisioning Service (DPS)](../articles/iot-dps/about-iot-dps.md) peuvent inclure le `modelId` à utiliser pendant le processus de provisionnement à l’aide de la charge utile JSON suivante.

```json
{
    "modelId" : "dtmi:com:example:Thermostat;1"
}
```

## <a name="implement-telemetry-properties-and-commands"></a>Implémentation des données de télémétrie, des propriétés et des commandes

Comme le décrit la section [Présentation des composants dans les modèles IoT Plug-and-Play](../articles/iot-pnp/concepts-components.md), les générateurs d’appareils doivent décider s’ils souhaitent utiliser des composants pour décrire leurs appareils, auquel cas ces derniers devront suivre les règles décrites dans cette section.

### <a name="telemetry"></a>Télémétrie

Aucun composant par défaut ne nécessite une propriété spéciale.

Lorsque vous utilisez des composants imbriqués, les appareils doivent définir une propriété de message avec le nom du composant :

```python
async def send_telemetry_from_temp_controller(device_client, telemetry_msg, component_name=None):
    msg = Message(json.dumps(telemetry_msg))
    msg.content_encoding = "utf-8"
    msg.content_type = "application/json"
    if component_name:
        msg.custom_properties["$.sub"] = component_name
    await device_client.send_message(msg)
}
```

### <a name="read-only-properties"></a>Propriétés en lecture seule

Le signalement d’une propriété à partir du composant par défaut ne nécessite pas de construction spéciale :

```python
await device_client.patch_twin_reported_properties({"maxTempSinceLastReboot": 38.7})
```

Le jumeau d’appareil est mis à jour avec la propriété rapportée suivante :

```json
{
  "reported": {
      "maxTempSinceLastReboot" : 38.7
  }
}
```

Lorsque vous utilisez des composants imbriqués, les propriétés doivent être créées dans le nom du composant :

```python
inner_dict = {}
inner_dict["targetTemperature"] = 38.7
inner_dict["__t"] = "c"
prop_dict = {}
prop_dict["thermostat1"] = inner_dict

await device_client.patch_twin_reported_properties(prop_dict)
```

Le jumeau d’appareil est mis à jour avec la propriété rapportée suivante :

```json
{
  "reported": {
    "thermostat1" : {  
      "__t" : "c",  
      "maxTempSinceLastReboot" : 38.7
     }
  }
}
```

### <a name="writable-properties"></a>Propriétés accessibles en écriture

Ces propriétés peuvent être définies par l’appareil ou mises à jour par la solution. Si la solution met à jour une propriété, le client reçoit une notification sous la forme d’un rappel dans `IoTHubDeviceClient` ou `IoTHubModuleClient`. Pour respecter les conventions IoT Plug-and-Play, l’appareil doit informer le service que la propriété a bien été reçue.

#### <a name="report-a-writable-property"></a>Signalement d’une propriété accessible en écriture

Quand un appareil rapporte une propriété accessible en écriture, il doit inclure les valeurs `ack` définies dans les conventions.

Pour signaler une propriété accessible en écriture à partir du composant par défaut :

```python
prop_dict = {}
prop_dict["targetTemperature"] = {
    "ac": 200,
    "ad": "reported default value",
    "av": 0,
    "value": 23.2
}

await device_client.patch_twin_reported_properties(prop_dict)
```

Le jumeau d’appareil est mis à jour avec la propriété rapportée suivante :

```json
{
  "reported": {
    "targetTemperature": {
      "value": 23.2,
      "ac": 200,
      "av": 0,
      "ad": "reported default value"
    }
  }
}
```

Pour signaler une propriété accessible en écriture à partir d’un composant imbriqué, le jumeau doit inclure un marqueur :

```python
inner_dict = {}
inner_dict["targetTemperature"] = {
    "ac": 200,
    "ad": "reported default value",
    "av": 0,
    "value": 23.2
}
inner_dict["__t"] = "c"
prop_dict = {}
prop_dict["thermostat1"] = inner_dict

await device_client.patch_twin_reported_properties(prop_dict)
```

Le jumeau d’appareil est mis à jour avec la propriété rapportée suivante :

```json
{
  "reported": {
    "thermostat1": {
      "__t" : "c",
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 0,
          "ad": "complete"
      }
    }
  }
}
```

#### <a name="subscribe-to-desired-property-updates"></a>Abonnement aux mises à jour de propriétés souhaitées

Les services peuvent mettre à jour les propriétés souhaitées qui déclenchent une notification sur les appareils connectés. Cette notification inclut les propriétés souhaitées mises à jour, y compris le numéro de version identifiant la mise à jour. Les appareils doivent répondre avec le même message `ack` que les propriétés rapportées.

Un composant par défaut considère la propriété unique et crée la propriété `ack` rapportée avec la version reçue :

```python
async def execute_property_listener(device_client):
    ignore_keys = ["__t", "$version"]
    while True:
        patch = await device_client.receive_twin_desired_properties_patch()  # blocking call

        version = patch["$version"]
        prop_dict = {}

        for prop_name, prop_value in patch.items():
            if prop_name in ignore_keys:
                continue
            else:
                prop_dict[prop_name] = {
                    "ac": 200,
                    "ad": "Successfully executed patch",
                    "av": version,
                    "value": prop_value,
                }

        await device_client.patch_twin_reported_properties(prop_dict)
```

Le jumeau d’appareil montre la propriété dans les sections desired et reported :

```json
{
  "desired" : {
    "targetTemperature": 23.2,
    "$version" : 3
  },
  "reported": {
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
  }
}
```

Un composant imbriqué reçoit les propriétés souhaitées encapsulées avec le nom du composant et doivent renvoyer la propriété rapportée `ack` :

```python
def create_reported_properties_from_desired(patch):
    ignore_keys = ["__t", "$version"]
    component_prefix = list(patch.keys())[0]
    values = patch[component_prefix]

    version = patch["$version"]
    inner_dict = {}

    for prop_name, prop_value in values.items():
        if prop_name in ignore_keys:
            continue
        else:
            inner_dict["ac"] = 200
            inner_dict["ad"] = "Successfully executed patch"
            inner_dict["av"] = version
            inner_dict["value"] = prop_value
            values[prop_name] = inner_dict

    properties_dict = dict()
    if component_prefix:
        properties_dict[component_prefix] = values
    else:
        properties_dict = values

    return properties_dict

async def execute_property_listener(device_client):
    while True:
        patch = await device_client.receive_twin_desired_properties_patch()  # blocking call
        properties_dict = create_reported_properties_from_desired(patch)

        await device_client.patch_twin_reported_properties(properties_dict)
```

Le jumeau d’appareil des composants montre les sections desired et reported de la façon suivante :

```json
{
  "desired" : {
    "thermostat1" : {
        "__t" : "c",
        "targetTemperature": 23.2,
    }
    "$version" : 3
  },
  "reported": {
    "thermostat1" : {
        "__t" : "c",
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
    }
  }
}
```

### <a name="commands"></a>Commandes

Un composant par défaut reçoit le nom de la commande telle qu’elle a été appelée par le service.

Un composant imbriqué reçoit le nom de la commande avec, pour préfixe, le nom du composant et le séparateur `*`.

```python
command_request = await device_client.receive_method_request("thermostat1*reboot")
```

#### <a name="request-and-response-payloads"></a>Charge utile de demande et de réponse

Les commandes utilisent des types pour définir leur charge utile de demande et de réponse. Un appareil doit désérialiser le paramètre d’entrée entrant et sérialiser la réponse. L’exemple suivant montre comment implémenter une commande avec des types complexes définis dans les charges utiles :

```json
{
  "@type": "Command",
  "name": "getMaxMinReport",
  "displayName": "Get Max-Min report.",
  "description": "This command returns the max, min and average temperature from the specified time to the current time.",
  "request": {
    "name": "since",
    "displayName": "Since",
    "description": "Period to return the max-min report.",
    "schema": "dateTime"
  },
  "response": {
    "name" : "tempReport",
    "displayName": "Temperature Report",
    "schema": {
      "@type": "Object",
      "fields": [
        {
          "name": "maxTemp",
          "displayName": "Max temperature",
          "schema": "double"
        },
        {
          "name": "minTemp",
          "displayName": "Min temperature",
          "schema": "double"
        },
        {
          "name" : "avgTemp",
          "displayName": "Average Temperature",
          "schema": "double"
        },
        {
          "name" : "startTime",
          "displayName": "Start Time",
          "schema": "dateTime"
        },
        {
          "name" : "endTime",
          "displayName": "End Time",
          "schema": "dateTime"
        }
      ]
    }
  }
}
```

Les extraits de code suivants montrent comment un appareil implémente cette définition de commande, notamment les types utilisés pour permettre la sérialisation et la désérialisation :

```python
def create_max_min_report_response(values):
    response_dict = {
        "maxTemp": max_temp,
        "minTemp": min_temp,
        "avgTemp": sum(avg_temp_list) / moving_window_size,
        "startTime": (datetime.now() - timedelta(0, moving_window_size * 8)).isoformat(),
        "endTime": datetime.now().isoformat(),
    }
    # serialize response dictionary into a JSON formatted str
    response_payload = json.dumps(response_dict, default=lambda o: o.__dict__, sort_keys=True)
    return response_payload
```

> [!Tip]
> Le nom des demandes et des réponses n’est pas présent dans les charges utiles sérialisées transmises sur le réseau.
