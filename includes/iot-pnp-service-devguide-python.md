---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 10/20/2020
ms.openlocfilehash: bef7807c0df580a6763a69619cdaa3d9d29f72e6
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92521375"
---
Les ressources suivantes sont également disponibles :

- [Documentation de référence du Kit de développement logiciel (SDK) Python](/python/api/azure-iot-hub/azure.iot.hub?preserve-view=true&view=azure-python)
- [Exemples de clients de service](https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-hub/samples/iothub_registry_manager_method_sample.py)
- [Exemples Digital Twins](https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-hub/samples/get_digital_twin_sample.py)

## <a name="iot-hub-service-client-examples"></a>Exemples de clients du service IoT Hub

Cette section présente des exemples Python utilisant le client de service IoT Hub et les classes **IoTHubRegistryManager** et **CloudToDeviceMethod**. Vous utilisez la classe **IoTHubRegistryManager** pour interagir avec l’état de l’appareil à l’aide de jumeaux d’appareil. Vous pouvez également utiliser la classe **IoTHubRegistryManager** pour [interroger les inscriptions d’appareils](../articles/iot-hub/iot-hub-devguide-query-language.md) dans votre IoT Hub. Vous utilisez la classe **CloudToDeviceMethod** pour appeler des commandes sur l’appareil. Le modèle [DTDL](../articles/iot-pnp/concepts-digital-twin.md) pour l’appareil définit la télémétrie, les propriétés et les commandes que l’appareil implémente. Dans les extraits de code, la variable `device_id` contient l’ID d’appareil de l’appareil IoT Plug-and-Play inscrit auprès de votre hub IoT.

### <a name="get-the-device-twin-and-model-id"></a>Récupérer le jumeau d’appareil et l’ID de modèle

Pour récupérer le jumeau d’appareil et l’ID de modèle de l’appareil IoT Plug-and-Play connecté à votre hub IoT :

```python
from azure.iot.hub import IoTHubRegistryManager
from azure.iot.hub.models import Twin, TwinProperties

iothub_registry_manager = IoTHubRegistryManager(iothub_connection_str)

# ...

twin = iothub_registry_manager.get_twin(device_id)
print("The device twin is: ")
print("")
print(twin)
print("")

additional_props = twin.additional_properties
if "modelId" in additional_props:
    print("The Model ID for this device is:")
    print(additional_props["modelId"])
    print("")
```

### <a name="update-device-twin"></a>Mettre à jour le jumeau d’appareil

L’extrait de code suivant montre comment mettre à jour la propriété `targetTemperature` sur un appareil. L’exemple montre comment vous devez obtenir `etag` du jumeau avant de le mettre à jour. La propriété est définie dans le composant par défaut de l’appareil :

```python
iothub_registry_manager = IoTHubRegistryManager(iothub_connection_str)

twin = iothub_registry_manager.get_twin(device_id)

twin_patch = Twin()

twin_patch.properties = TwinProperties(
    desired={"targetTemperature": 42}
)
updated_twin = iothub_registry_manager.update_twin(device_id, twin_patch, twin.etag)
```

L’extrait suivant montre comment mettre à jour la propriété `targetTemperature` sur un composant. L’exemple montre comment vous devez obtenir `ETag` du jumeau avant de le mettre à jour. La propriété est définie dans le composant **Thermostat1** :

```python
iothub_registry_manager = IoTHubRegistryManager(iothub_connection_str)

twin = iothub_registry_manager.get_twin(device_id)

twin_patch = Twin()

twin_patch.properties = TwinProperties(
    desired={ "thermostat1": {
        "__t": "c",
        "targetTemperature": 42}
    }
)
updated_twin = iothub_registry_manager.update_twin(device_id, twin_patch, twin.etag)
```

Pour une propriété dans un composant, le correctif de la propriété se présente comme dans l’exemple suivant :

```json
{
"thermostat1":
  {
    "__t": "c",
    "targetTemperature": 20
  }
}
```

### <a name="call-command"></a>Commande Call

L’extrait suivant montre comment appeler la commande `getMaxMinReport` définie dans un composant par défaut :

```python
from azure.iot.hub import IoTHubRegistryManager
from azure.iot.hub.models import CloudToDeviceMethod

# ...

iothub_registry_manager = IoTHubRegistryManager(iothub_connection_str)

method_payload = datetime.datetime.now() - datetime.timedelta(minutes=2)
device_method = CloudToDeviceMethod(method_name="getMaxMinReport", payload=method_payload)
result = iothub_registry_manager.invoke_device_method(device_id, device_method)
print(result.payload)
```

L’extrait suivant montre comment appeler la commande `getMaxMinReport` sur un composant. La commande est définie dans le composant **thermostat1**  :

```python
from azure.iot.hub import IoTHubRegistryManager
from azure.iot.hub.models import CloudToDeviceMethod

# ...

iothub_registry_manager = IoTHubRegistryManager(iothub_connection_str)

method_payload = datetime.datetime.now() - datetime.timedelta(minutes=2)
device_method = CloudToDeviceMethod(method_name="thermostat1*getMaxMinReport", payload=method_payload)
result = iothub_registry_manager.invoke_device_method(device_id, device_method)
print(result.payload)
```

## <a name="iot-hub-digital-twin-examples"></a>Exemples de jumeaux numériques IoT Hub

Vous utilisez la classe **DigitalTwinClient** pour interagir avec l’état de l’appareil à l’aide de jumeaux numériques. Le modèle [DTDL](../articles/iot-pnp/concepts-digital-twin.md) pour l’appareil définit la télémétrie, les propriétés et les commandes que l’appareil implémente.

La variable `device_id` contient l’ID d’appareil de l’appareil IoT Plug-and-Play inscrit auprès de votre hub IoT.

### <a name="get-the-digital-twin-and-model-id"></a>Récupérer le jumeau numérique et l’ID de modèle

Pour récupérer le jumeau numérique et l’ID de modèle de l’appareil IoT Plug-and-Play connecté à votre hub IoT :

```python
from azure.iot.hub import DigitalTwinClient

digital_twin_client = DigitalTwinClient(iothub_connection_str)

digital_twin = digital_twin_client.get_digital_twin(device_id)
if digital_twin:
    print(digital_twin)
    print("Model Id: " + digital_twin["$metadata"]["$model"])
else:
    print("No digital_twin found")
```

### <a name="update-digital-twin"></a>Mettre à jour le jumeau numérique

L’extrait de code suivant montre comment mettre à jour la propriété `targetTemperature` sur un appareil. La propriété est définie dans le composant par défaut de l’appareil :

```python
from azure.iot.hub import DigitalTwinClient

digital_twin_client = DigitalTwinClient(iothub_connection_str)

patch = [{"op": "add", "path": "/targetTemperature", "value": 42}]
digital_twin_client.update_digital_twin(device_id, patch)
```

L’extrait suivant montre comment mettre à jour la propriété `targetTemperature` sur un composant. La propriété est définie dans le composant **Thermostat1** :

```python
from azure.iot.hub import DigitalTwinClient

digital_twin_client = DigitalTwinClient(iothub_connection_str)

patch = [{"op": "add", "path": "/targetTemperature", "value": 42}]
digital_twin_client.update_digital_twin(device_id, patch)
```

### <a name="call-command"></a>Commande Call

L’extrait suivant montre comment appeler la commande `getMaxMinReport` définie dans un composant par défaut :

```python
from azure.iot.hub import DigitalTwinClient

payload = datetime.datetime.now() - datetime.timedelta(minutes=2)

connect_timeout_in_seconds = 3
response_timeout_in_seconds = 7


digital_twin_client = DigitalTwinClient(iothub_connection_str)

invoke_command_result = digital_twin_client.invoke_command(
    device_id, "getMaxMinReport", payload, connect_timeout_in_seconds, response_timeout_in_seconds
)
if invoke_command_result:
    print(invoke_command_result)
else:
    print("No invoke_command_result found")
```

L’extrait suivant montre comment appeler la commande `getMaxMinReport` sur un composant. La commande est définie dans le composant **thermostat1**  :

```python
from azure.iot.hub import DigitalTwinClient

payload = datetime.datetime.now() - datetime.timedelta(minutes=2)

connect_timeout_in_seconds = 3
response_timeout_in_seconds = 7


digital_twin_client = DigitalTwinClient(iothub_connection_str)

invoke_command_result = digital_twin_client.invoke_component_command(
    device_id, "thermostat1", "getMaxMinReport", payload, connect_timeout_in_seconds, response_timeout_in_seconds
)
if invoke_command_result:
    print(invoke_command_result)
else:
    print("No invoke_command_result found")
```

## <a name="read-device-telemetry"></a>Lire la télémétrie d’appareil

Les appareils IoT Plug-and-Play envoient la télémétrie définie dans le modèle DTDL à IoT Hub. Par défaut, IoT Hub achemine la télémétrie vers un point de terminaison Event Hubs où vous pouvez l’utiliser. Pour en savoir plus, consultez [Utiliser le routage des messages IoT Hub pour envoyer des messages appareil-à-cloud à différents points de terminaison](../articles/iot-hub/iot-hub-devguide-messages-d2c.md).

L’extrait de code suivant montre comment lire la télémétrie du point de terminaison Event Hubs par défaut. Le code de cet extrait provient du démarrage rapide IoT Hub [Envoyer des données de télémétrie d’un appareil à un hub IoT et les lire depuis ce hub avec une application back-end](../articles/iot-hub/quickstart-send-telemetry-python.md) :

```python
import asyncio
from azure.eventhub import TransportType
from azure.eventhub.aio import EventHubConsumerClient

# Define callbacks to process events
async def on_event_batch(partition_context, events):
    for event in events:
        print("Received event from partition: {}.".format(partition_context.partition_id))
        print("Telemetry received: ", event.body_as_str())
        print("Properties (set by device): ", event.properties)
        print("System properties (set by IoT Hub): ", event.system_properties)
        print()
    await partition_context.update_checkpoint()

async def on_error(partition_context, error):
    # ...

loop = asyncio.get_event_loop()
client = EventHubConsumerClient.from_connection_string(
    conn_str=CONNECTION_STR,
    consumer_group="$default",
)

try:
    loop.run_until_complete(client.receive_batch(on_event_batch=on_event_batch, on_error=on_error))
except KeyboardInterrupt:
    print("Receiving has stopped.")
finally:
    loop.run_until_complete(client.close())
    loop.stop()
```

La sortie suivante du code précédent montre la télémétrie de température envoyée par l’appareil IoT Plug-and-Play **Thermostat** sans composant qui possède uniquement le composant par défaut. La propriété système `dt-dataschema` affiche l’ID de modèle :

```cmd/sh
Received event from partition: 1.
Telemetry received:  {"temperature": 12}
Properties (set by device):  None
System properties (set by IoT Hub):  {b'content-type': b'application/json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-connection-auth-method': b'{"scope":"device","type":"sas","issuer":"iothub","acceptingIpFilterRule":null}', b'iothub-connection-auth-generation-id': b'637388855582764406', b'iothub-enqueuedtime': 1603288810715, b'iothub-message-source': b'Telemetry', b'dt-dataschema': b'dtmi:com:example:Thermostat;1', b'x-opt-sequence-number': 13280, b'x-opt-offset': b'12890070640', b'x-opt-enqueued-time': 1603288810824}
```

La sortie suivante du code précédent montre la télémétrie de température envoyée par l’appareil IoT Plug-and-Play **TemperatureController** à plusieurs composants. La propriété système `dt-subject` affiche le nom du composant qui a envoyé la télémétrie. Dans cet exemple, les deux composants sont `thermostat1` et `thermostat2` tels qu’ils sont définis dans le modèle DTDL. La propriété système `dt-dataschema` affiche l’ID de modèle :

```cmd/sh
Received event from partition: 1.
Telemetry received:  {"temperature": 45}
Properties (set by device):  None
System properties (set by IoT Hub):  {b'content-type': b'application/json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-connection-auth-method': b'{"scope":"device","type":"sas","issuer":"iothub","acceptingIpFilterRule":null}', b'iothub-connection-auth-generation-id': b'637388858939631652', b'iothub-enqueuedtime': 1603289127844, b'iothub-message-source': b'Telemetry', b'dt-subject': b'thermostat1', b'dt-dataschema': b'dtmi:com:example:TemperatureController;1', b'x-opt-sequence-number': 13328, b'x-opt-offset': b'12890095440', b'x-opt-enqueued-time': 1603289128001}

Received event from partition: 1.
Telemetry received:  {"temperature": 49}
Properties (set by device):  None
System properties (set by IoT Hub):  {b'content-type': b'application/json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-connection-auth-method': b'{"scope":"device","type":"sas","issuer":"iothub","acceptingIpFilterRule":null}', b'iothub-connection-auth-generation-id': b'637388858939631652', b'iothub-enqueuedtime': 1603289133017, b'iothub-message-source': b'Telemetry', b'dt-subject': b'thermostat2', b'dt-dataschema': b'dtmi:com:example:TemperatureController;1', b'x-opt-sequence-number': 13329, b'x-opt-offset': b'12890095928', b'x-opt-enqueued-time': 1603289133173}
```

## <a name="read-device-twin-change-notifications"></a>Lire les notifications de changement du jumeau d’appareil

Vous pouvez configurer IoT Hub pour générer des notifications de changement du jumeau d’appareil à acheminer vers un point de terminaison pris en charge. Pour en savoir plus, consultez [Utiliser le routage des messages IoT Hub pour envoyer des messages appareil-à-cloud à différents points de terminaison > Événements autres que les événements de télémétrie](../articles/iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events).

Le code présenté dans l’extrait de code Python précédent génère la sortie suivante quand IoT Hub génère des notifications de changement du jumeau d’appareil pour un appareil thermostat sans composant. Les propriétés de l’application `iothub-message-schema` et `opType` vous fournissent des informations sur le type de notification de changement :

```cmd/sh
Received event from partition: 1.
Telemetry received:  {"version":3,"properties":{"reported":{"maxTempSinceLastReboot":10.96,"$metadata":{"$lastUpdated":"2020-10-21T14:10:42.4171263Z","maxTempSinceLastReboot":{"$lastUpdated":"2020-10-21T14:10:42.4171263Z"}},"$version":2}}}
Properties (set by device):  {b'hubName': b'my-pnp-hub', b'deviceId': b'my-pnp-device', b'operationTimestamp': b'2020-10-21T14:10:42.4171263+00:00', b'iothub-message-schema': b'twinChangeNotification', b'opType': b'updateTwin'}
System properties (set by IoT Hub):  {b'user-id': b'my-pnp-hub\x81\x0e\xa4\x7f', b'correlation-id': b'12104ced5402', b'content-type': b'application/json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-enqueuedtime': 1603289442519, b'iothub-message-source': b'twinChangeEvents', b'x-opt-sequence-number': 13332, b'x-opt-offset': b'12890097392', b'x-opt-enqueued-time': 1603289442738}
```

Le code présenté dans l’extrait de code Python précédent génère la sortie suivante quand IoT Hub génère des notifications de changement du jumeau d’appareil pour un appareil avec des composants. Cet exemple montre la sortie quand un appareil capteur de température avec un composant thermostat génère des notifications. Les propriétés de l’application `iothub-message-schema` et `opType` vous fournissent des informations sur le type de notification de changement :

```cmd/sh
Received event from partition: 1.
Telemetry received:  {"version":4,"properties":{"reported":{"thermostat1":{"maxTempSinceLastReboot":98.34,"__t":"c"},"$metadata":{"$lastUpdated":"2020-10-21T14:13:39.36491Z","thermostat1":{"$lastUpdated":"2020-10-21T14:13:39.36491Z","maxTempSinceLastReboot":{"$lastUpdated":"2020-10-21T14:13:39.36491Z"},"__t":{"$lastUpdated":"2020-10-21T14:13:39.36491Z"}}},"$version":3}}}
Properties (set by device):  {b'hubName': b'my-pnp-hub', b'deviceId': b'my-pnp-device', b'operationTimestamp': b'2020-10-21T14:13:39.3649100+00:00', b'iothub-message-schema': b'twinChangeNotification', b'opType': b'updateTwin'}
System properties (set by IoT Hub):  {b'user-id': b'my-pnp-hub', b'correlation-id': b'1210b664ab83', b'content-type': b'application/json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-enqueuedtime': 1603289619481, b'iothub-message-source': b'twinChangeEvents', b'x-opt-sequence-number': 13341, b'x-opt-offset': b'12890102216', b'x-opt-enqueued-time': 1603289619668}
```

## <a name="read-digital-twin-change-notifications"></a>Lire les notifications de changement du jumeau numérique

Vous pouvez configurer IoT Hub pour générer des notifications de changement du jumeau numérique à acheminer vers un point de terminaison pris en charge. Pour en savoir plus, consultez [Utiliser le routage des messages IoT Hub pour envoyer des messages appareil-à-cloud à différents points de terminaison > Événements autres que les événements de télémétrie](../articles/iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events).

Le code présenté dans l’extrait de code Python précédent génère la sortie suivante quand IoT Hub génère des notifications de changement du jumeau numérique pour un appareil thermostat sans composant. Les propriétés de l’application `iothub-message-schema` et `opType` vous fournissent des informations sur le type de notification de changement :

```cmd/sh
Received event from partition: 1.
Telemetry received:  [{"op":"add","path":"/$metadata/maxTempSinceLastReboot","value":{"lastUpdateTime":"2020-10-21T14:10:42.4171263Z"}},{"op":"add","path":"/maxTempSinceLastReboot","value":10.96}]
Properties (set by device):  {b'hubName': b'my-pnp-hub', b'deviceId': b'my-pnp-device', b'operationTimestamp': b'2020-10-21T14:10:42.4171263+00:00', b'iothub-message-schema': b'digitalTwinChangeNotification', b'opType': b'updateTwin'}
System properties (set by IoT Hub):  {b'user-id': b'my-pnp-hub\x81\x0e\xa4\x7f', b'correlation-id': b'12104ced5402', b'content-type': b'application/json-patch+json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-enqueuedtime': 1603289442519, b'iothub-message-source': b'digitalTwinChangeEvents', b'x-opt-sequence-number': 13333, b'x-opt-offset': b'12890098024', b'x-opt-enqueued-time': 1603289442738}
```

Le code présenté dans l’extrait de code Python précédent génère la sortie suivante quand IoT Hub génère des notifications de changement du jumeau numérique pour un appareil avec des composants. Cet exemple montre la sortie quand un appareil capteur de température avec un composant thermostat génère des notifications. Les propriétés de l’application `iothub-message-schema` et `opType` vous fournissent des informations sur le type de notification de changement :

```cmd/sh
Received event from partition: 1.
Telemetry received:  [{"op":"add","path":"/thermostat1","value":{"$metadata":{"maxTempSinceLastReboot":{"lastUpdateTime":"2020-10-21T14:13:39.36491Z"}},"maxTempSinceLastReboot":98.34}}]
Properties (set by device):  {b'hubName': b'my-pnp-hub', b'deviceId': b'my-pnp-device', b'operationTimestamp': b'2020-10-21T14:13:39.3649100+00:00', b'iothub-message-schema': b'digitalTwinChangeNotification', b'opType': b'updateTwin'}
System properties (set by IoT Hub):  {b'user-id': b'my-pnp-hub', b'correlation-id': b'1210b664ab83', b'content-type': b'application/json-patch+json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-enqueuedtime': 1603289619481, b'iothub-message-source': b'digitalTwinChangeEvents', b'x-opt-sequence-number': 13342, b'x-opt-offset': b'12890102984', b'x-opt-enqueued-time': 1603289619668}
```
