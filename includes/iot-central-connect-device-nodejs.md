---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/24/2020
ms.openlocfilehash: 6a6baa14d7521f4a85350af7b08b5fcbe82ddf6b
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033834"
---
## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes de cet article, vous avez besoin des éléments suivants :

* Une application Azure IoT Central créée avec le modèle **Application personnalisée**. Pour plus d’informations, consultez [Créer une application](../articles/iot-central/core/quick-deploy-iot-central.md). L’application doit avoir été créée à partir le 14 juillet 2020 ou après.
* Une machine de développement où [Node.js](https://nodejs.org/) version 6 ou ultérieure est installé. Vous pouvez exécuter `node --version` sur la ligne de commande pour vérifier la version. Les instructions de ce tutoriel supposent que vous exécutez la commande **node** à l’invite de commandes Windows. Toutefois, vous pouvez utiliser Node.js sur de nombreux autres systèmes d’exploitation.
* Une copie locale du dépôt GitHub [SDK Microsoft Azure IoT pour Node.js](https://github.com/Azure/azure-iot-sdk-node) qui contient l’exemple de code. Utilisez ce lien pour télécharger une copie du dépôt : [Télécharger le fichier ZIP](https://github.com/Azure/azure-iot-sdk-node/archive/master.zip). Décompressez ensuite le fichier dans un emplacement approprié de votre ordinateur local.

## <a name="review-the-code"></a>Vérifier le code

Dans la copie du SDK Microsoft Azure IoT pour Node.js que vous avez téléchargée précédemment, ouvrez le fichier *azure-iot-sdk-node/device/samples/pnp/simple_thermostat.js* dans un éditeur de texte.

Quand vous exécutez l’exemple pour vous connecter à IoT Central, il utilise le service Device Provisioning (DPS) pour inscrire l’appareil et générer une chaîne de connexion. L’exemple récupère les informations de connexion DPS dont il a besoin à partir de l’environnement de ligne de commande.

La méthode `main` :

* Crée un objet `client` et définit l’ID de modèle `dtmi:com:example:Thermostat;1` avant l’ouverture de la connexion. IoT Central utilise l’ID de modèle pour identifier ou générer le modèle d’appareil correspondant à cet appareil. Pour en savoir plus, consultez [Associer un appareil à un modèle d’appareil](../articles/iot-central/core/concepts-get-connected.md#associate-a-device-with-a-device-template).
* Crée un gestionnaire de commandes.
* Démarre une boucle pour envoyer des données de télémétrie de température toutes les 10 secondes.
* Envoie la propriété `maxTempSinceLastReboot` à IoT Central. IoT Central ignore la propriété `serialNumber`, car elle ne fait pas partie du modèle d’appareil.
* Crée un gestionnaire de propriétés accessible en écriture.

```javascript
async function main() {

  // ...

  // fromConnectionString must specify a transport, coming from any transport package.
  const client = Client.fromConnectionString(deviceConnectionString, Protocol);

  let resultTwin;
  try {
    // Add the modelId here
    await client.setOptions(modelIdObject);
    await client.open();

    client.onDeviceMethod(commandMaxMinReport, commandHandler);

    // Send Telemetry every 10 secs
    let index = 0;
    intervalToken = setInterval(() => {
      sendTelemetry(client, index).catch((err) => console.log('error', err.toString()));
      index += 1;
    }, telemetrySendInterval);

    // attach a standard input exit listener
    attachExitHandler(client);

    // Deal with twin
    try {
      resultTwin = await client.getTwin();
      const patchRoot = createReportPropPatch({ serialNumber: deviceSerialNum });
      const patchThermostat = createReportPropPatch({
        maxTempSinceLastReboot: deviceTemperatureSensor.getMaxTemperatureValue()
      });

      // the below things can only happen once the twin is there
      updateComponentReportedProperties(resultTwin, patchRoot);
      updateComponentReportedProperties(resultTwin, patchThermostat);

      // Setup the handler for desired properties
      desiredPropertyPatchHandler(resultTwin);

    } catch (err) {
      console.error('could not retrieve twin or report twin properties\n' + err.toString());
    }
  } catch (err) {
    console.error('could not connect Plug and Play client or could not attach interval function for telemetry\n' + err.toString());
  }
}
```

La fonction `provisionDevice` montre comment l’appareil utilise le service DPS pour s’inscrire et se connecter à IoT Central. La charge utile inclut l’ID de modèle qu’IoT Central utilise pour [associer l’appareil à un modèle d’appareil](../articles/iot-central/core/concepts-get-connected.md#associate-a-device-with-a-device-template) :

```javascript
async function provisionDevice(payload) {
  var provSecurityClient = new SymmetricKeySecurityClient(registrationId, symmetricKey);
  var provisioningClient = ProvisioningDeviceClient.create(provisioningHost, idScope, new ProvProtocol(), provSecurityClient);

  if (!!(payload)) {
    provisioningClient.setProvisioningPayload(payload);
  }

  try {
    let result = await provisioningClient.register();
    deviceConnectionString = 'HostName=' + result.assignedHub + ';DeviceId=' + result.deviceId + ';SharedAccessKey=' + symmetricKey;
  } catch (err) {
    console.error("error registering device: " + err.toString());
  }
}
```

La fonction `sendTelemetry` montre comment l’appareil envoie les données de télémétrie de température à IoT Central. La méthode `getCurrentTemperatureObject` retourne un objet semblable à `{ temperature: 45.6 }` :

```javascript
async function sendTelemetry(deviceClient, index) {
  console.log('Sending telemetry message %d...', index);
  const msg = new Message(
    JSON.stringify(
      deviceTemperatureSensor.updateSensor().getCurrentTemperatureObject()
    )
  );
  msg.contentType = 'application/json';
  msg.contentEncoding = 'utf-8';
  await deviceClient.sendEvent(msg);
}
```

La méthode `main` utilise les deux méthodes suivantes pour envoyer la propriété `maxTempSinceLastReboot` à IoT Central. La méthode `main` appelle `createReportPropPatch` avec un objet semblable à `{maxTempSinceLastReboot: 80.9}` :

```javascript
const createReportPropPatch = (propertiesToReport) => {
  let patch;
  patch = { };
  patch = propertiesToReport;
  return patch;
};

const updateComponentReportedProperties = (deviceTwin, patch) => {
  deviceTwin.properties.reported.update(patch, function (err) {
    if (err) throw err;
    console.log('Properties have been reported for component');
  });
};
```

La méthode `main` utilise les deux méthodes suivantes pour gérer les mises à jour de la propriété accessible en écriture de _température cible_ à partir d’IoT Central. Notez comment `propertyUpdateHandle` génère la réponse avec la version et le code d’état :

```javascript
const desiredPropertyPatchHandler = (deviceTwin) => {
  deviceTwin.on('properties.desired', (delta) => {
    const versionProperty = delta.$version;

    Object.entries(delta).forEach(([propertyName, propertyValue]) => {
      if (propertyName !== '$version') {
        propertyUpdateHandler(deviceTwin, propertyName, null, propertyValue, versionProperty);
      }
    });
  });
};

const propertyUpdateHandler = (deviceTwin, propertyName, reportedValue, desiredValue, version) => {
  console.log('Received an update for property: ' + propertyName + ' with value: ' + JSON.stringify(desiredValue));
  const patch = createReportPropPatch(
    { [propertyName]:
      {
        'value': desiredValue,
        'ac': 200,
        'ad': 'Successfully executed patch for ' + propertyName,
        'av': version
      }
    });
  updateComponentReportedProperties(deviceTwin, patch);
  console.log('updated the property');
};
```

La méthode `main` utilise les deux méthodes suivantes pour gérer les appels à la commande `getMaxMinReport`. La méthode `getMaxMinReportObject` génère le rapport sous la forme d’un objet JSON :

```javascript
const commandHandler = async (request, response) => {
  switch (request.methodName) {
  case commandMaxMinReport: {
    console.log('MaxMinReport ' + request.payload);
    await sendCommandResponse(request, response, 200, deviceTemperatureSensor.getMaxMinReportObject());
    break;
  }
  default:
    await sendCommandResponse(request, response, 404, 'unknown method');
    break;
  }
};

const sendCommandResponse = async (request, response, status, payload) => {
  try {
    await response.send(status, payload);
    console.log('Response to method \'' + request.methodName +
              '\' sent successfully.' );
  } catch (err) {
    console.error('An error ocurred when sending a method response:\n' +
              err.toString());
  }
};
```

## <a name="get-connection-information"></a>Obtenir des informations de connexion

[!INCLUDE [iot-central-connection-configuration](iot-central-connection-configuration.md)]

## <a name="run-the-code"></a>Exécuter le code

Pour exécuter l’exemple d’application, ouvrez un environnement de ligne de commande, puis accédez au dossier *azure-iot-sdk-node/device/samples/pnp* qui contient l’exemple de fichier *simple_thermostat.js*.

[!INCLUDE [iot-central-connection-environment](iot-central-connection-environment.md)]

Installez les packages nécessaires :

```cmd/sh
npm install
```

Exécutez l’exemple :

```cmd/sh
node simple_thermostat.js
```

La sortie suivante montre l’inscription de l’appareil et la connexion à IoT Central. L’exemple envoie ensuite la propriété `maxTempSinceLastReboot` avant de commencer à envoyer des données de télémétrie :

```output
registration succeeded
assigned hub=iotc-.......azure-devices.net
deviceId=sample-device-01
payload=undefined
Connecting using connection string HostName=iotc-........azure-devices.net;DeviceId=sample-device-01;SharedAccessKey=Ci....=
Enabling the commands on the client
Please enter q or Q to exit sample.
The following properties will be updated for root interface:
{ maxTempSinceLastReboot: 55.20309427428496 }
Properties have been reported for component
Sending telemetry message 0...
Sending telemetry message 1...
Sending telemetry message 2...
Sending telemetry message 3...
```

[!INCLUDE [iot-central-monitor-thermostat](iot-central-monitor-thermostat.md)]

Vous pouvez voir comment l’appareil répond aux commandes et aux mises à jour de propriétés :

```output
MaxMinReport 2020-10-15T12:00:00.000Z
Response to method 'getMaxMinReport' sent successfully.

...

Received an update for property: targetTemperature with value: {"value":86.3}
The following properties will be updated for root interface:
{
  targetTemperature: {
    value: { value: 86.3 },
    ac: 200,
    ad: 'Successfully executed patch for targetTemperature',
    av: 2
  }
}
```
