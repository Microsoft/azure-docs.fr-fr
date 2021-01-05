---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 10/20/2020
ms.openlocfilehash: 7bf32de017a5f8ad19eb044ae7dbcdc2eaa96ca5
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92521378"
---
Les ressources suivantes sont également disponibles :

- [Documentation de référence du Kit de développement logiciel (SDK) Node.js](/javascript/api/azure-iothub?preserve-view=true&view=azure-node-latest)
- [Exemples de clients de service](https://github.com/Azure/azure-iot-sdk-node/blob/master/service/samples/javascript/twin.js)
- [Exemples Digital Twins](https://github.com/Azure/azure-iot-sdk-node/blob/master/service/samples/javascript/get_digital_twin.js)

## <a name="iot-hub-service-client-examples"></a>Exemples de clients du service IoT Hub

Cette section présente des exemples JavaScript utilisant le client du service IoT Hub et les classes **Registry** et **Client**. Vous utilisez la classe **Registry** pour interagir avec l’état de l’appareil à l’aide de jumeaux d’appareil. Vous pouvez également utiliser la classe **Registry** pour [interroger les inscriptions d’appareils](../articles/iot-hub/iot-hub-devguide-query-language.md) dans votre IoT Hub. Vous utilisez la classe **Client** pour appeler des commandes sur l’appareil. Le modèle [DTDL](../articles/iot-pnp/concepts-digital-twin.md) pour l’appareil définit la télémétrie, les propriétés et les commandes que l’appareil implémente. Dans les extraits de code, la variable `deviceId` contient l’ID d’appareil de l’appareil IoT Plug-and-Play inscrit auprès de votre hub IoT.

### <a name="get-the-device-twin-and-model-id"></a>Récupérer le jumeau d’appareil et l’ID de modèle

Pour récupérer le jumeau d’appareil et l’ID de modèle de l’appareil IoT Plug-and-Play connecté à votre hub IoT :

```javascript
var Registry = require('azure-iothub').Registry;

// ...

var registry = Registry.fromConnectionString(connectionString);
registry.getTwin(deviceId, function(err, twin) {
  if (err) {
    console.error(err.message);
  } else {
    console.log('Model Id: ' + twin.modelId);
    console.log(JSON.stringify(twin, null, 2));
  }
}
```

### <a name="update-device-twin"></a>Mettre à jour le jumeau d’appareil

L’extrait de code suivant montre comment mettre à jour la propriété `targetTemperature` sur un appareil. L’exemple montre comment vous devez obtenir le jumeau avant de le mettre à jour. La propriété est définie dans le composant par défaut de l’appareil :

```javascript
var Registry = require('azure-iothub').Registry;
var registry = Registry.fromConnectionString(connectionString);

registry.getTwin(deviceId, function(err, twin) {
  if (err) {
    console.error(err.message);
  } else {
    var twinPatch = {
      properties: {
        desired: {
          targetTemperature: 42
        }
      }
    };
    twin.update(twinPatch, function(err, twin) {
      if (err) {
        console.error(err.message);
      } else {
        console.log(JSON.stringify(twin, null, 2));
      }
    }
  }
}
```

L’extrait suivant montre comment mettre à jour la propriété `targetTemperature` sur un composant. L’exemple montre comment vous devez obtenir le jumeau avant de le mettre à jour. La propriété est définie dans le composant **thermostat1**  :

```javascript
var Registry = require('azure-iothub').Registry;
var registry = Registry.fromConnectionString(connectionString);

registry.getTwin(deviceId, function(err, twin) {
  if (err) {
    console.error(err.message);
  } else {
    var twinPatch = {
      properties: {
        desired: {
          thermostat1:
          {
            __t: "c",
            targetTemperature: 45
          }
        }
      }
    };
    twin.update(twinPatch, function(err, twin) {
      if (err) {
        console.error(err.message);
      } else {
        console.log(JSON.stringify(twin, null, 2));
      }
    }
  }
}
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

```javascript
var Client = require('azure-iothub').Client;

// ...

var client = Client.fromConnectionString(connectionString);

var methodParams = {
  methodName: "getMaxMinReport",
  payload: new Date().getMinutes -2,
  responseTimeoutInSeconds: 15
};

client.invokeDeviceMethod(deviceId, methodParams, function (err, result) {
  if (err) {
    console.error('Failed to invoke method \'' + methodParams.methodName + '\': ' + err.message);
  } else {
    console.log(methodParams.methodName + ' on ' + deviceId + ':');
    console.log(JSON.stringify(result, null, 2));
  }
});
```

L’extrait suivant montre comment appeler la commande `getMaxMinReport` sur un composant. La commande est définie dans le composant **thermostat1**  :

```javascript
var Client = require('azure-iothub').Client;

// ...

var client = Client.fromConnectionString(connectionString);

var methodParams = {
  methodName: "thermostat1*getMaxMinReport",
  payload: new Date().getMinutes -2,
  responseTimeoutInSeconds: 15
};

client.invokeDeviceMethod(deviceId, methodParams, function (err, result) {
  if (err) {
    console.error('Failed to invoke method \'' + methodParams.methodName + '\': ' + err.message);
  } else {
    console.log(methodParams.methodName + ' on ' + deviceId + ':');
    console.log(JSON.stringify(result, null, 2));
  }
});
```

## <a name="iot-hub-digital-twin-examples"></a>Exemples de jumeaux numériques IoT Hub

Vous utilisez la classe **DigitalTwinClient** pour interagir avec l’état de l’appareil à l’aide de jumeaux numériques. Le modèle [DTDL](../articles/iot-pnp/concepts-digital-twin.md) pour l’appareil définit la télémétrie, les propriétés et les commandes que l’appareil implémente.

Cette section présente des exemples JavaScript utilisant l’API Digital Twins.

La variable `digitalTwinId` contient l’ID d’appareil de l’appareil IoT Plug-and-Play inscrit auprès de votre hub IoT.

### <a name="get-the-digital-twin-and-model-id"></a>Récupérer le jumeau numérique et l’ID de modèle

Pour récupérer le jumeau numérique et l’ID de modèle de l’appareil IoT Plug-and-Play connecté à votre hub IoT :

```javascript
const IoTHubTokenCredentials = require('azure-iothub').IoTHubTokenCredentials;
const DigitalTwinClient = require('azure-iothub').DigitalTwinClient;
const { inspect } = require('util');

// ...

const credentials = new IoTHubTokenCredentials(connectionString);
const digitalTwinClient = new DigitalTwinClient(credentials);

const digitalTwin = await digitalTwinClient.getDigitalTwin(digitalTwinId);

console.log(inspect(digitalTwin));
console.log('Model Id: ' + inspect(digitalTwin.$metadata.$model));
```

### <a name="update-digital-twin"></a>Mettre à jour le jumeau numérique

L’extrait de code suivant montre comment mettre à jour la propriété `targetTemperature` sur un appareil. La propriété est définie dans le composant par défaut de l’appareil :

```javascript
const IoTHubTokenCredentials = require('azure-iothub').IoTHubTokenCredentials;
const DigitalTwinClient = require('azure-iothub').DigitalTwinClient;

// ...

const credentials = new IoTHubTokenCredentials(connString);
const digitalTwinClient = new DigitalTwinClient(credentials);

const patch = [{
  op: 'add',
  path: '/targetTemperature',
  value: 42
}];
await digitalTwinClient.updateDigitalTwin(digitalTwinId, patch);
```

L’extrait suivant montre comment mettre à jour la propriété `targetTemperature` sur un composant. La propriété est définie dans le composant **thermostat1**  :

```javascript
const IoTHubTokenCredentials = require('azure-iothub').IoTHubTokenCredentials;
const DigitalTwinClient = require('azure-iothub').DigitalTwinClient;

// ...

const credentials = new IoTHubTokenCredentials(connString);
const digitalTwinClient = new DigitalTwinClient(credentials);

const patch = [{
  op: 'add',
  path: '/thermostat1/targetTemperature',
  value: 42
}];
await digitalTwinClient.updateDigitalTwin(digitalTwinId, patch);
```

### <a name="call-command"></a>Commande Call

L’extrait suivant montre comment appeler la commande `getMaxMinReport` définie dans un composant par défaut :

```javascript
const IoTHubTokenCredentials = require('azure-iothub').IoTHubTokenCredentials;
const DigitalTwinClient = require('azure-iothub').DigitalTwinClient;
const { inspect } = require('util');

// ...

const commandPayload = new Date().getMinutes -2;

const credentials = new IoTHubTokenCredentials(connectionString);
const digitalTwinClient = new DigitalTwinClient(credentials);

const options = {
  connectTimeoutInSeconds: 30,
  responseTimeoutInSeconds: 40
};
const commandResponse = await digitalTwinClient.invokeCommand(digitalTwinId, "getMaxMinReport", commandPayload, options);

console.log(inspect(commandResponse));
```

L’extrait suivant montre comment appeler la commande `getMaxMinReport` sur un composant. La commande est définie dans le composant **thermostat1**  :

```javascript
const IoTHubTokenCredentials = require('azure-iothub').IoTHubTokenCredentials;
const DigitalTwinClient = require('azure-iothub').DigitalTwinClient;
const { inspect } = require('util');

// ...

const commandPayload = new Date().getMinutes -2;

const credentials = new IoTHubTokenCredentials(connectionString);
const digitalTwinClient = new DigitalTwinClient(credentials);

const options = {
  connectTimeoutInSeconds: 30,
  responseTimeoutInSeconds: 40
};
const commandResponse = await digitalTwinClient.invokeComponentCommand(digitalTwinId, "thermostat1", "getMaxMinReport", commandPayload, options);

console.log(inspect(commandResponse));
```

## <a name="read-device-telemetry"></a>Lire la télémétrie d’appareil

Les appareils IoT Plug-and-Play envoient la télémétrie définie dans le modèle DTDL à IoT Hub. Par défaut, IoT Hub achemine la télémétrie vers un point de terminaison Event Hubs où vous pouvez l’utiliser. Pour en savoir plus, consultez [Utiliser le routage des messages IoT Hub pour envoyer des messages appareil-à-cloud à différents points de terminaison](../articles/iot-hub/iot-hub-devguide-messages-d2c.md).

L’extrait de code suivant montre comment lire la télémétrie du point de terminaison Event Hubs par défaut. Le code de cet extrait provient du démarrage rapide IoT Hub [Envoyer des données de télémétrie d’un appareil à un hub IoT et les lire depuis ce hub avec une application back-end](../articles/iot-hub/quickstart-send-telemetry-node.md) :

```javascript
const { EventHubConsumerClient } = require("@azure/event-hubs");

var printError = function (err) {
  console.log(err.message);
};

var printMessages = function (messages) {
  for (const message of messages) {
    console.log("Telemetry received: ");
    console.log(JSON.stringify(message.body));
    console.log("Properties (set by device): ");
    console.log(JSON.stringify(message.properties));
    console.log("System properties (set by IoT Hub): ");
    console.log(JSON.stringify(message.systemProperties));
    console.log("");
  }
};

// ...

const clientOptions = {};

const consumerClient = new EventHubConsumerClient("$Default", connectionString, clientOptions);

consumerClient.subscribe({
  processEvents: printMessages,
  processError: printError,
});
```

La sortie suivante du code précédent montre la télémétrie de température envoyée par l’appareil IoT Plug-and-Play **TemperatureController** à plusieurs composants. La propriété système `dt-subject` affiche le nom du composant qui a envoyé la télémétrie. Dans cet exemple, les deux composants sont `thermostat1` et `thermostat2` tels qu’ils sont définis dans le modèle DTDL. La propriété système `dt-dataschema` affiche l’ID de modèle :

```cmd/sh
Telemetry received:
{"temperature":68.77370855171125}
Properties (set by device):
undefined
System properties (set by IoT Hub):
{"iothub-connection-device-id":"my-pnp-device","iothub-connection-auth-method":"{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}","iothub-connection-auth-generation-id":"637388034455888246","iothub-enqueuedtime":1603206669320,"iothub-message-source":"Telemetry","dt-subject":"thermostat1","dt-dataschema":"dtmi:com:example:TemperatureController;1","contentType":"application/json","contentEncoding":"utf-8"}

Telemetry received:
{"temperature":30.833394506549226}
Properties (set by device):
undefined
System properties (set by IoT Hub):
{"iothub-connection-device-id":"my-pnp-device","iothub-connection-auth-method":"{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}","iothub-connection-auth-generation-id":"637388034455888246","iothub-enqueuedtime":1603206665835,"iothub-message-source":"Telemetry","dt-subject":"thermostat2","dt-dataschema":"dtmi:com:example:TemperatureController;1","contentType":"application/json","contentEncoding":"utf-8"}
```

## <a name="read-device-twin-change-notifications"></a>Lire les notifications de changement du jumeau d’appareil

Vous pouvez configurer IoT Hub pour générer des notifications de changement du jumeau d’appareil à acheminer vers un point de terminaison pris en charge. Pour en savoir plus, consultez [Utiliser le routage des messages IoT Hub pour envoyer des messages appareil-à-cloud à différents points de terminaison > Événements autres que les événements de télémétrie](../articles/iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events).

Le code présenté dans l’extrait de code JavaScript précédent génère la sortie suivante quand IoT Hub génère des notifications de changement du jumeau d’appareil pour un appareil thermostat sans composant. Les propriétés de l’application `iothub-message-schema` et `opType` vous fournissent des informations sur le type de notification de changement :

```cmd/sh
Telemetry received:
{"version":4,"properties":{"reported":{"maxTempSinceLastReboot":42.1415152639582,"$metadata":{"$lastUpdated":"2020-10-21T10:01:40.1281138Z","maxTempSinceLastReboot":{"$lastUpdated":"2020-10-21T10:01:40.1281138Z"}},"$version":3}}}
Properties (set by device):
{"hubName":"my-pnp-hub","deviceId":"my-pnp-device","operationTimestamp":"2020-10-21T10:01:40.1281138+00:00","iothub-message-schema":"twinChangeNotification","opType":"updateTwin"}
System properties (set by IoT Hub):
{"iothub-connection-device-id":"my-pnp-device","iothub-enqueuedtime":1603274500282,"iothub-message-source":"twinChangeEvents","userId":{"type":"Buffer","data":[109,121,45,112,110,112,45,104,117,98]},"correlationId":"11ed82d13f50","contentType":"application/json","contentEncoding":"utf-8"}
```

Le code présenté dans l’extrait de code JavaScript précédent génère la sortie suivante quand IoT Hub génère des notifications de changement du jumeau d’appareil pour un appareil avec des composants. Cet exemple montre la sortie quand un appareil capteur de température avec un composant thermostat génère des notifications. Les propriétés de l’application `iothub-message-schema` et `opType` vous fournissent des informations sur le type de notification de changement :

```cmd/sh
Telemetry received:
{"version":4,"properties":{"reported":{"thermostat1":{"maxTempSinceLastReboot":3.5592971602417913,"__t":"c"},"$metadata":{"$lastUpdated":"2020-10-21T10:07:51.8284866Z","thermostat1":{"$lastUpdated":"2020-10-21T10:07:51.8284866Z","maxTempSinceLastReboot":{"$lastUpdated":"2020-10-21T10:07:51.8284866Z"},"__t":{"$lastUpdated":"2020-10-21T10:07:51.8284866Z"}}},"$version":3}}}
Properties (set by device):
{"hubName":"my-pnp-hub","deviceId":"my-pnp-device","operationTimestamp":"2020-10-21T10:07:51.8284866+00:00","iothub-message-schema":"twinChangeNotification","opType":"updateTwin"}
System properties (set by IoT Hub):
{"iothub-connection-device-id":"my-pnp-device","iothub-enqueuedtime":1603274871951,"iothub-message-source":"twinChangeEvents","userId":{"type":"Buffer","data":[109,121,45,112,110,112,45,104,117,98]},"correlationId":"11ee605b195f","contentType":"application/json","contentEncoding":"utf-8"}
```

## <a name="read-digital-twin-change-notifications"></a>Lire les notifications de changement du jumeau numérique

Vous pouvez configurer IoT Hub pour générer des notifications de changement du jumeau numérique à acheminer vers un point de terminaison pris en charge. Pour en savoir plus, consultez [Utiliser le routage des messages IoT Hub pour envoyer des messages appareil-à-cloud à différents points de terminaison > Événements autres que les événements de télémétrie](../articles/iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events).

Le code présenté dans l’extrait de code JavaScript précédent génère la sortie suivante quand IoT Hub génère des notifications de changement du jumeau numérique pour un appareil thermostat sans composant. Les propriétés de l’application `iothub-message-schema` et `opType` vous fournissent des informations sur le type de notification de changement :

```cmd/sh
Telemetry received:
[{"op":"add","path":"/$metadata/maxTempSinceLastReboot","value":{"lastUpdateTime":"2020-10-21T10:01:40.1281138Z"}},{"op":"add","path":"/maxTempSinceLastReboot","value":42.1415152639582}]
Properties (set by device):
{"hubName":"my-pnp-hub","deviceId":"my-pnp-device","operationTimestamp":"2020-10-21T10:01:40.1281138+00:00","iothub-message-schema":"digitalTwinChangeNotification","opType":"updateTwin"}
System properties (set by IoT Hub):
{"iothub-connection-device-id":"my-pnp-device","iothub-enqueuedtime":1603274500282,"iothub-message-source":"digitalTwinChangeEvents","userId":{"type":"Buffer","data":[109,121,45,112,110,112,45,104,117,98]},"correlationId":"11ed82d13f50","contentType":"application/json-patch+json","contentEncoding":"utf-8"}
```

Le code présenté dans l’extrait de code JavaScript précédent génère la sortie suivante quand IoT Hub génère des notifications de changement du jumeau numérique pour un appareil avec des composants. Cet exemple montre la sortie quand un appareil capteur de température avec un composant thermostat génère des notifications. Les propriétés de l’application `iothub-message-schema` et `opType` vous fournissent des informations sur le type de notification de changement :

```cmd/sh
Telemetry received:
[{"op":"add","path":"/thermostat1","value":{"$metadata":{"maxTempSinceLastReboot":{"lastUpdateTime":"2020-10-21T10:07:51.8284866Z"}},"maxTempSinceLastReboot":3.5592971602417913}}]
Properties (set by device):
{"hubName":"my-pnp-hub","deviceId":"my-pnp-device","operationTimestamp":"2020-10-21T10:07:51.8284866+00:00","iothub-message-schema":"digitalTwinChangeNotification","opType":"updateTwin"}
System properties (set by IoT Hub):
{"iothub-connection-device-id":"my-pnp-device","iothub-enqueuedtime":1603274871951,"iothub-message-source":"digitalTwinChangeEvents","userId":{"type":"Buffer","data":[109,121,45,112,110,112,45,104,117,98]},"correlationId":"11ee605b195f","contentType":"application/json-patch+json","contentEncoding":"utf-8"}
```
