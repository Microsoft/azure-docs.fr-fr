---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/24/2020
ms.openlocfilehash: 16cba46d9c0f6a933965366d82eb8e21c4ab1ca6
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033879"
---
## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes décrites dans cet article, vous avez besoin de l’application Azure IoT Central suivante créée à l’aide du modèle **Application personnalisée**. Pour plus d’informations, consultez [Créer une application](../articles/iot-central/core/quick-deploy-iot-central.md). L’application doit avoir été créée à partir le 14 juillet 2020 ou après.

Vous pouvez exécuter ce tutoriel sur Linux ou Windows. Dans ce tutoriel, les commandes de l’interpréteur de commandes suivent la convention Linux des séparateurs de chemin « `/` ». Si vous suivez ce guide sur Windows, veillez à remplacer les séparateurs par « `\` ».

Les prérequis diffèrent selon le système d’exploitation :

### <a name="linux"></a>Linux

Ce tutoriel suppose que vous utilisez Ubuntu Linux. Les étapes de ce didacticiel ont été testées à l’aide d’Ubuntu 18.04.

Pour suivre ce tutoriel sur Linux, installez les logiciels suivants dans votre environnement Linux local :

Installez **GCC**, **Git**, **cmake** et toutes les dépendances nécessaires à l’aide de la commande `apt-get` :

```sh
sudo apt-get update
sudo apt-get install -y git cmake build-essential curl libcurl4-openssl-dev libssl-dev uuid-dev
```

Vérifiez que la version de `cmake` est supérieure à **2.8.12** et que la version de **GCC** est supérieure à **4.4.7**.

```sh
cmake --version
gcc --version
```

### <a name="windows"></a>Windows

Pour effectuer ce tutoriel sur Windows, vous devez installer les logiciels suivants sur votre environnement Windows local :

* [Visual Studio (Community, Professional ou Enterprise)](https://visualstudio.microsoft.com/downloads/) : veillez à inclure la charge de travail **Développement Desktop en C++** quand vous [installez](/cpp/build/vscpp-step-0-installation?preserve-view=true&view=vs-2019) Visual Studio.
* [Git](https://git-scm.com/download/).
* [CMake](https://cmake.org/download/).

## <a name="download-the-code"></a>Téléchargement du code

Dans ce guide de démarrage rapide, vous préparez un environnement de développement pour cloner et générer le kit Azure IoT Hub Device C SDK.

Ouvrez une invite de commandes dans le répertoire de votre choix. Exécutez la commande suivante pour cloner le dépôt GitHub [Azure IoT C SDKs and Libraries](https://github.com/Azure/azure-iot-sdk-c) à cet emplacement :

```cmd\bash
git clone https://github.com/Azure/azure-iot-sdk-c.git
cd azure-iot-sdk-c
git submodule update --init
```

Attendez-vous à ce que cette opération prenne plusieurs minutes.

## <a name="review-the-code"></a>Vérifier le code

Dans la copie du SDK Microsoft Azure IoT pour C que vous avez téléchargée précédemment, ouvrez le fichier *azure-iot-sdk-c/iothub_client/samples/pnp/pnp_simple_thermostat/pnp_simple_thermostat.c* dans un éditeur de texte.

Quand vous exécutez l’exemple pour vous connecter à IoT Central, il utilise le service Device Provisioning (DPS) pour inscrire l’appareil et générer une chaîne de connexion. L’exemple récupère les informations de connexion DPS dont il a besoin à partir de l’environnement de ligne de commande.

La fonction `main` appelle d’abord `CreateAndConfigureDeviceClientHandleForPnP` pour :

* Définir l’ID du modèle `dtmi:com:example:Thermostat;1`. IoT Central utilise l’ID de modèle pour identifier ou générer le modèle d’appareil correspondant à cet appareil. Pour en savoir plus, consultez [Associer un appareil à un modèle d’appareil](../articles/iot-central/core/concepts-get-connected.md#associate-a-device-with-a-device-template).
* Utiliser DPS pour provisionner et inscrire l’appareil.
* Créer un gestionnaire de commandes pour la commande `getMaxMinReport`.
* Créer un gestionnaire de mise à jour de propriétés pour la propriété `targetTemperature` accessible en écriture.
* Créer un handle de client d’appareil et le connecter à votre application IoT Central.

La fonction `main` démarre ensuite un thread pour envoyer régulièrement des données de télémétrie.

```c
int main(void)
{
  IOTHUB_DEVICE_CLIENT_LL_HANDLE deviceClientLL = NULL;

  if (GetConnectionSettingsFromEnvironment() == false)
  {
    LogError("Cannot read required environment variable(s)");
  }
  else if (BuildUtcTimeFromCurrentTime(g_ProgramStartTime, sizeof(g_ProgramStartTime)) == false)
  {
    LogError("Unable to output the program start time");
  }
  else if ((deviceClientLL = CreateAndConfigureDeviceClientHandleForPnP()) == NULL)
  {
    LogError("Failed creating IotHub device client");
  }
  else
  {
    LogInfo("Successfully created device client handle.  Hit Control-C to exit program\n");

    int numberOfIterations = 0;
    SendMaxTemperatureSinceReboot(deviceClientLL);

    while (true)
    {
      // Wake up periodically to poll.  Even if we do not plan on sending telemetry, we still need to poll periodically in order to process
      // incoming requests from the server and to do connection keep alives.
      if ((numberOfIterations % g_sendTelemetryPollInterval) == 0)
      {
        Thermostat_SendCurrentTemperature(deviceClientLL);
      }

      IoTHubDeviceClient_LL_DoWork(deviceClientLL);
      ThreadAPI_Sleep(g_sleepBetweenPollsMs);
      numberOfIterations++;
    }

      // Clean up the iothub sdk handle
      IoTHubDeviceClient_LL_Destroy(deviceClientLL);
      // Free all the IoT SDK subsystem
      IoTHub_Deinit();
  }

  return 0;
}
```

La fonction `Thermostat_SendCurrentTemperature` montre comment l’appareil envoie les données de télémétrie de température à IoT Central :

```c
void Thermostat_SendCurrentTemperature(IOTHUB_DEVICE_CLIENT_LL_HANDLE deviceClientLL) 
{
  IOTHUB_MESSAGE_HANDLE messageHandle = NULL;
  IOTHUB_CLIENT_RESULT iothubResult;

  char temperatureStringBuffer[32];

  if (snprintf(temperatureStringBuffer, sizeof(temperatureStringBuffer), g_temperatureTelemetryBodyFormat, g_currentTemperature) < 0)
  {
    LogError("snprintf of current temperature telemetry failed");
  }
  else if ((messageHandle = IoTHubMessage_CreateFromString(temperatureStringBuffer)) == NULL)
  {
    LogError("IoTHubMessage_CreateFromString failed");
  }
  else if ((iothubResult = IoTHubDeviceClient_LL_SendEventAsync(deviceClientLL, messageHandle, NULL, NULL)) != IOTHUB_CLIENT_OK)
  {
    LogError("Unable to send telemetry message, error=%d", iothubResult);
  }

  IoTHubMessage_Destroy(messageHandle);
}
```

La fonction `SendMaxTemperatureSinceReboot` envoie une mise à jour de propriété `maxTempSinceLastReboot` à IoT Central :

```c
static void SendMaxTemperatureSinceReboot(IOTHUB_DEVICE_CLIENT_LL_HANDLE deviceClientLL)
{
  IOTHUB_CLIENT_RESULT iothubClientResult;
  char maxTemperatureSinceRebootProperty[256];

  if (snprintf(maxTemperatureSinceRebootProperty, sizeof(maxTemperatureSinceRebootProperty), g_maxTemperatureSinceRebootFormat, g_maxTemperature) < 0)
  {
      LogError("snprintf building maxTemperature failed");
  }
  else if ((iothubClientResult = IoTHubDeviceClient_LL_SendReportedState(deviceClientLL, (const unsigned char*)maxTemperatureSinceRebootProperty, strlen(maxTemperatureSinceRebootProperty), NULL, NULL)) != IOTHUB_CLIENT_OK)
  {
      LogError("Unable to send reported state for maximum temperature.  Error=%d", iothubClientResult);
  }
  else
  {
      LogInfo("Sending maxTempSinceReboot property");
  }
}
```

La fonction `Thermostat_DeviceTwinCallback` gère les mises à jour de propriétés accessibles en écriture à partir d’IoT Central :

```c
static void Thermostat_DeviceTwinCallback(DEVICE_TWIN_UPDATE_STATE updateState, const unsigned char* payload, size_t size, void* userContextCallback)
{
  // The device handle associated with this request is passed as the context, since we will need to send reported events back.
  IOTHUB_DEVICE_CLIENT_LL_HANDLE deviceClientLL = (IOTHUB_DEVICE_CLIENT_LL_HANDLE)userContextCallback;

  char* jsonStr = NULL;
  JSON_Value* rootValue = NULL;
  JSON_Object* desiredObject;
  JSON_Value* versionValue = NULL;
  JSON_Value* targetTemperatureValue = NULL;

  LogInfo("DeviceTwin callback invoked");

  if ((jsonStr = CopyTwinPayloadToString(payload, size)) == NULL)
  {
    LogError("Unable to allocate twin buffer");
  }
  else if ((rootValue = json_parse_string(jsonStr)) == NULL)
  {
    LogError("Unable to parse twin JSON");
  }
  else if ((desiredObject = GetDesiredJson(updateState, rootValue)) == NULL)
  {
    LogError("Cannot retrieve desired JSON object");
  }
  else if ((targetTemperatureValue = json_object_get_value(desiredObject, g_JSONTargetTemperature)) == NULL)
  {
    LogInfo("JSON property %s not specified.  This is NOT an error as the server doesn't need to set this, but there is no further action to take.", g_JSONTargetTemperature);
  }
  else if ((versionValue = json_object_get_value(desiredObject, g_IoTHubTwinDesiredVersion)) == NULL)
  {
    // The $version does need to be set in *any* legitimate twin desired document.  Its absence suggests 
    // something is fundamentally wrong with how we've received the twin and we should not proceed.
    LogError("Cannot retrieve field %s for twin.  The underlying IoTHub device twin protocol (NOT the service solution directly) should have specified this.", g_IoTHubTwinDesiredVersion);
  }
  else if (json_value_get_type(versionValue) != JSONNumber)
  {
    // The $version must be a number (and in practice an int) A non-numerical value indicates 
    // something is fundamentally wrong with how we've received the twin and we should not proceed.
    LogError("JSON field %s is not a number but must be", g_IoTHubTwinDesiredVersion);
  }
  else if (json_value_get_type(targetTemperatureValue) != JSONNumber)
  {
    LogError("JSON field %s is not a number", g_JSONTargetTemperature);
  }
  else
  {
    double targetTemperature = json_value_get_number(targetTemperatureValue);
    int version = (int)json_value_get_number(versionValue);

    LogInfo("Received targetTemperature = %f", targetTemperature);

    bool maxTempUpdated = false;
    UpdateTemperatureAndStatistics(targetTemperature, &maxTempUpdated);

    // The device needs to let the service know that it has received the targetTemperature desired property.
    SendTargetTemperatureReport(deviceClientLL, targetTemperature, g_statusSuccess, version, g_temperaturePropertyResponseDescription);

    if (maxTempUpdated)
    {
      // If the Maximum temperature has been updated, we also report this as a property.
      SendMaxTemperatureSinceReboot(deviceClientLL);
    }
  }

  json_value_free(rootValue);
  free(jsonStr);
}
```

La fonction `Thermostat_DeviceMethodCallback` gère les commandes appelées à partir d’IoT Central :

```c
static int Thermostat_DeviceMethodCallback(const char* methodName, const unsigned char* payload, size_t size, unsigned char** response, size_t* responseSize, void* userContextCallback)
{
  (void)userContextCallback;

  char* jsonStr = NULL;
  JSON_Value* rootValue = NULL;
  const char* sinceStr;
  int result;

  LogInfo("Device method %s arrived", methodName);

  *response = NULL;
  *responseSize = 0;

  if (strcmp(methodName, g_getMaxMinReport) != 0)
  {
    LogError("Method name %s is not supported on this component", methodName);
    result = g_statusNotFoundStatus;
  }
  else if ((jsonStr = CopyTwinPayloadToString(payload, size)) == NULL)
  {
    LogError("Unable to allocate twin buffer");
    result = g_statusInternalError;
  }
  else if ((rootValue = json_parse_string(jsonStr)) == NULL)
  {
    LogError("Unable to parse twin JSON");
    result = g_statusBadFormat;
  }
  // See caveats section in ../readme.md; we don't actually respect this sinceStr to keep the sample simple,
  // but want to demonstrate how to parse out in any case.
  else if ((sinceStr = json_value_get_string(rootValue)) == NULL)
  {
    LogError("Cannot retrieve since value");
    result = g_statusBadFormat;
  }
  else if (BuildMaxMinCommandResponse(response, responseSize) == false)
  {
    LogError("Unable to build response");
    result = g_statusInternalError;
  }
  else
  {
    LogInfo("Returning success from command request");
    result = g_statusSuccess;
  }

  if (*response == NULL)
  {
    SetEmptyCommandResponse(response, responseSize, &result);
  }

  json_value_free(rootValue);
  free(jsonStr);

  return result;
}
```

## <a name="build-the-code"></a>Générer le code

Vous utilisez le kit de développement logiciel (SDK) de l’appareil pour créer l’exemple de code inclus :

1. Créez un sous-répertoire _cmake_ dans le dossier racine du SDK d’appareil, puis accédez à ce dossier :

    ```cmd\bash
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    ```

1. Exécutez les commandes suivantes pour générer le SDK et les exemples :

    ```cmd\bash
    cmake -Duse_prov_client=ON -Dhsm_type_symm_key=ON -Drun_e2e_tests=OFF ..
    cmake --build .
    ```

## <a name="get-connection-information"></a>Obtenir des informations de connexion

[!INCLUDE [iot-central-connection-configuration](iot-central-connection-configuration.md)]

## <a name="run-the-code"></a>Exécuter le code

Pour exécuter l’exemple d’application, ouvrez un environnement de ligne de commande, puis accédez au dossier *azure-iot-sdk-c\cmake*.

[!INCLUDE [iot-central-connection-environment](iot-central-connection-environment.md)]

Pour exécuter l’exemple :

```bash
# Bash
cd iothub_client/samples/pnp/pnp_simple_thermostat/
./pnp_simple_thermostat
```

```cmd
REM Windows
cd iothub_client\samples\pnp\pnp_simple_thermostat\Debug
.\pnp_simple_thermostat.exe
```

La sortie suivante montre l’inscription de l’appareil et la connexion à IoT Central. L’exemple commence à envoyer la télémétrie :

```output
Info: Initiating DPS client to retrieve IoT Hub connection information
-> 11:53:07 CONNECT | VER: 4 | KEEPALIVE: 0 | FLAGS: 194 | USERNAME: 0ne001BB295/registrations/sample-device-01/api-version=2019-03-31&ClientVersion=1.3.9 | PWD: XXXX | CLEAN: 1
<- 11:53:08 CONNACK | SESSION_PRESENT: false | RETURN_CODE: 0x0
-> 11:53:09 SUBSCRIBE | PACKET_ID: 1 | TOPIC_NAME: $dps/registrations/res/# | QOS: 1
<- 11:53:10 SUBACK | PACKET_ID: 1 | RETURN_CODE: 1
-> 11:53:10 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_MOST_ONCE | TOPIC_NAME: $dps/registrations/PUT/iotdps-register/?$rid=1 | PAYLOAD_LEN: 91
<- 11:53:11 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_MOST_ONCE | TOPIC_NAME: $dps/registrations/res/202/?$rid=1&retry-after=3 | PACKET_ID: 2 | PAYLOAD_LEN: 94
-> 11:53:12 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_MOST_ONCE | TOPIC_NAME: $dps/registrations/GET/iotdps-get-operationstatus/?$rid=2&operationId=4.2f792ade0a5c3e68.e123be1d-3b1e-4874-813a-5612ae586979 | PAYLOAD_LEN: 91
<- 11:53:13 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_MOST_ONCE | TOPIC_NAME: $dps/registrations/res/200/?$rid=2 | PACKET_ID: 2 | PAYLOAD_LEN: 478
Info: Provisioning callback indicates success.  iothubUri=iotc-5d1e8352-cb4d-4e0b-ac00-37704ffa1f2a.azure-devices.net, deviceId=sample-device-01
-> 11:53:13 DISCONNECT
Info: DPS successfully registered.  Continuing on to creation of IoTHub device client handle.
Info: Successfully created device client handle.  Hit Control-C to exit program

Info: Sending maxTempSinceReboot property
-> 11:53:14 CONNECT | VER: 4 | KEEPALIVE: 240 | FLAGS: 192 | USERNAME: iotc-5d1e8352-cb4d-4e0b-ac00-37704ffa1f2a.azure-devices.net/sample-device-01/?api-version=2020-09-30&DeviceClientType=iothubclient%2f1.3.9%20(native%3b%20WindowsProduct%3a0x00000004%206.2%3b%20x64%3b%20%7bC85D6F43-30FF-4647-BF03-226E8A2943FD%7d)&model-id=dtmi%3acom%3aexample%3aThermostat%3b1 | PWD: XXXX | CLEAN: 0
<- 11:53:14 CONNACK | SESSION_PRESENT: false | RETURN_CODE: 0x0
-> 11:53:15 SUBSCRIBE | PACKET_ID: 2 | TOPIC_NAME: $iothub/twin/res/# | QOS: 0 | TOPIC_NAME: $iothub/methods/POST/# | QOS: 0
<- 11:53:15 SUBACK | PACKET_ID: 2 | RETURN_CODE: 0 | RETURN_CODE: 0
-> 11:53:15 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_MOST_ONCE | TOPIC_NAME: $iothub/twin/GET/?$rid=3
-> 11:53:15 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_MOST_ONCE | TOPIC_NAME: $iothub/twin/PATCH/properties/reported/?$rid=4 | PAYLOAD_LEN: 32
-> 11:53:15 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_LEAST_ONCE | TOPIC_NAME: devices/sample-device-01/messages/events/ | PACKET_ID: 5 | PAYLOAD_LEN: 21
<- 11:53:15 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: QOS_VALUE_INVALID | TOPIC_NAME: $iothub/twin/res/204/?$rid=4&$version=2 | PAYLOAD_LEN: 0
<- 11:53:15 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: QOS_VALUE_INVALID | TOPIC_NAME: $iothub/twin/res/200/?$rid=3 | PAYLOAD_LEN: 82
Info: DeviceTwin callback invoked
Info: JSON property targetTemperature not specified.  This is NOT an error as the server doesn't need to set this, but there is no further action to take.
<- 11:53:15 PUBACK | PACKET_ID: 5
-> 11:53:15 SUBSCRIBE | PACKET_ID: 6 | TOPIC_NAME: $iothub/twin/PATCH/properties/desired/# | QOS: 0
<- 11:53:16 SUBACK | PACKET_ID: 6 | RETURN_CODE: 0
-> 11:54:20 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_LEAST_ONCE | TOPIC_NAME: devices/sample-device-01/messages/events/ | PACKET_ID: 7 | PAYLOAD_LEN: 21
<- 11:54:21 PUBACK | PACKET_ID: 7
```

[!INCLUDE [iot-central-monitor-thermostat](iot-central-monitor-thermostat.md)]

Vous pouvez voir comment l’appareil répond aux commandes et aux mises à jour de propriétés :

```output
<- 11:56:34 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: QOS_VALUE_INVALID | TOPIC_NAME: $iothub/methods/POST/getMaxMinReport/?$rid=1 | PAYLOAD_LEN: 26
Info: Device method getMaxMinReport arrived
Info: Response=<{"maxTemp":22.00,"minTemp":22.00,"avgTemp":22.00,"startTime":"2020-11-25T11:53:05Z","endTime":"2020-11-25T11:56:34Z"}>
Info: Returning success from command request
-> 11:56:34 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_MOST_ONCE | TOPIC_NAME: $iothub/methods/res/200/?$rid=1 | PAYLOAD_LEN: 117

...

<- 11:57:24 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: QOS_VALUE_INVALID | TOPIC_NAME: $iothub/twin/PATCH/properties/desired/?$version=2 | PAYLOAD_LEN: 37
Info: DeviceTwin callback invoked
Info: Received targetTemperature = 56.000000
Info: Sending maxTempSinceReboot property
Info: Sending maxTempSinceReboot property
-> 11:57:24 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_MOST_ONCE | TOPIC_NAME: $iothub/twin/PATCH/properties/reported/?$rid=11 | PAYLOAD_LEN: 68
-> 11:57:24 PUBLISH | IS_DUP: false | RETAIN: 0 | QOS: DELIVER_AT_MOST_ONCE | TOPIC_NAME: $iothub/twin/PATCH/properties/reported/?$rid=12 | PAYLOAD_LEN: 32
```
