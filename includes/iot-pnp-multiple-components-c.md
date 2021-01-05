---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 190e2144e92ffd77c098b4bf9205de03e55af7df
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95511260"
---
Ce tutoriel vous montre comment créer un exemple d'application d'appareil IoT Plug-and-Play avec des composants, comment le connecter à votre hub IoT, et comment utiliser l'outil Explorateur Azure IoT pour voir les informations qu'il envoie au hub. L’exemple d’application est écrit en C et est inclus dans le Azure IoT device SDK pour C. Un générateur de solutions peut utiliser l’outil Explorateur Azure IoT pour comprendre les fonctionnalités d’un appareil IoT Plug-and-Play sans avoir à afficher de code d’appareil.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Vous pouvez suivre ce tutoriel sur Linux ou Windows. Dans ce tutoriel, les commandes de l’interpréteur de commandes suivent la convention Linux des séparateurs de chemin « `/` ». Si vous suivez ce guide sur Windows, veillez à remplacer les séparateurs par « `\` ».

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

Si vous avez suivi le [guide de démarrage rapide : Connecter un exemple d'application d'appareil IoT Plug-and-Play s'exécutant sur Linux ou Windows à IoT Hub (C)](../articles/iot-pnp/quickstart-connect-device.md), vous avez déjà téléchargé le code.

Dans ce tutoriel, vous préparez un environnement de développement qui vous permet de cloner et générer le kit SDK Azure IoT Hub Device C.

Ouvrez une invite de commandes dans un dossier de votre choix. Exécutez la commande suivante pour cloner le dépôt GitHub [Azure IoT C SDKs and Libraries](https://github.com/Azure/azure-iot-sdk-c) à cet emplacement :

```cmd/bash
git clone https://github.com/Azure/azure-iot-sdk-c.git
cd azure-iot-sdk-c
git submodule update --init
```

Attendez-vous à ce que cette opération prenne plusieurs minutes.

## <a name="build-and-run-the-code"></a>Générer et exécuter le code

Vous pouvez générer et exécuter le code à l’aide de Visual Studio ou de `cmake` sur la ligne de commande.

### <a name="use-visual-studio"></a>Utiliser Visual Studio

1. Ouvrez le dossier racine du dépôt cloné. Après quelques secondes, la prise en charge de **CMake** dans Visual Studio crée tout ce dont vous avez besoin pour exécuter et déboguer le projet.
1. Une fois que Visual Studio est prêt, dans l’**Explorateur de solutions**, accédez à l’exemple *iothub_client/samples/pnp/pnp_temperature_controller/* .
1. Cliquez avec le bouton droit sur le fichier *pnp_temperature_controller.c*, puis sélectionnez **Ajouter une configuration Debug**. Sélectionnez **Par défaut**.
1. Visual Studio ouvre le fichier *launch.vs.json*. Modifiez ce fichier comme indiqué dans l’extrait suivant pour définir les variables d’environnement nécessaires. Vous avez noté l’ID d’étendue et la clé primaire d’inscription lorsque vous avez terminé [Configurer votre environnement pour les guides de démarrages rapides et tutoriels IoT Plug-and-Play](../articles/iot-pnp/set-up-environment.md) :

    ```json
    {
      "version": "0.2.1",
      "defaults": {},
      "configurations": [
        {
          "type": "default",
          "project": "iothub_client\\samples\\pnp\\pnp_temperature_controller\\pnp_temperature_controller.c",
          "projectTarget": "",
          "name": "pnp_temperature_controller.c",
          "env": {
            "IOTHUB_DEVICE_SECURITY_TYPE": "DPS",
            "IOTHUB_DEVICE_DPS_ID_SCOPE": "<Your ID scope>",
            "IOTHUB_DEVICE_DPS_DEVICE_ID": "my-pnp-device",
            "IOTHUB_DEVICE_DPS_DEVICE_KEY": "<Your enrollment primary key>"
          }
        }
      ]
    }
    ```

1. Cliquez avec le bouton droit sur le fichier *pnp_temperature_controller.c*, puis sélectionnez **Définir comme élément de démarrage**.
1. Pour tracer l’exécution du code dans Visual Studio, ajoutez un point d’arrêt à la fonction `main` dans le fichier *pnp_temperature_controller.c*.
1. Vous pouvez à présent exécuter et déboguer l’exemple à partir du menu **Déboguer**.

L’appareil, désormais prêt à recevoir des commandes et des mises à jour de propriétés, a commencé à envoyer des données de télémétrie au hub. Laissez l’exemple s’exécuter pendant que vous effectuez les étapes suivantes.

### <a name="use-cmake-at-the-command-line"></a>Utiliser `cmake` sur la ligne de commande

Pour créer l'exemple :

1. Créez un sous-dossier _cmake_ dans le dossier racine du kit SDK de l’appareil cloné, puis accédez à ce dossier :

    ```cmd/bash
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    ```

1. Exécutez les commandes suivantes pour produire et générer les fichiers projet du kit SDK et des exemples :

    ```cmd/bash
    cmake ..
    cmake --build .
    ```

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Pour en savoir plus sur l’exemple de configuration, consultez l’[exemple de fichier Lisez-moi](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/samples/pnp/readme.md).

Pour exécuter l’exemple :

1. Dans le dossier _cmake_, accédez au dossier qui contient le fichier exécutable et exécutez-le :

    ```bash
    # Bash
    cd iothub_client/samples/pnp/pnp_temperature_controller/
    ./pnp_temperature_controller
    ```

    ```cmd
    REM Windows
    cd iothub_client\samples\pnp\pnp_temperature_controller\Debug
    pnp_temperature_controller.exe
    ```

L’appareil, désormais prêt à recevoir des commandes et des mises à jour de propriétés, a commencé à envoyer des données de télémétrie au hub. Laissez l’exemple s’exécuter pendant que vous effectuez les étapes suivantes.

### <a name="use-the-azure-iot-explorer-to-validate-the-code"></a>Utiliser l’Explorateur Azure IoT pour valider le code

Une fois l’exemple de client d’appareil démarré, utilisez l’outil Explorateur Azure IoT pour vérifier qu’il fonctionne.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Vérifier le code

Cet exemple implémente un appareil contrôleur de température IoT Plug-and-Play. Cet exemple implémente un modèle avec [plusieurs composants](../articles/iot-pnp/concepts-components.md). Le [fichier de modèle DTDL (Digital Twins Definition Language) pour l’appareil de contrôle de température](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) définit la télémétrie, les propriétés et les commandes implémentées par l’appareil.

### <a name="iot-plug-and-play-helper-functions"></a>Fonctions d’assistance IoT Plug-and-Play

Pour cet exemple, le code utilise des fonctions d’assistance du dossier */common* :

*pnp_device_client_ll* contient la méthode de connexion pour IoT Plug-and-Play avec `model-id` inclus en tant que paramètre : `PnP_CreateDeviceClientLLHandle`.

*pnp_protocol* : contient les fonctions d’assistance IoT Plug-and-Play :

* `PnP_CreateReportedProperty`
* `PnP_CreateReportedPropertyWithStatus`
* `PnP_ParseCommandName`
* `PnP_CreateTelemetryMessageHandle`
* `PnP_ProcessTwinData`
* `PnP_CopyPayloadToString`
* `PnP_CreateDeviceClientLLHandle_ViaDps`

Ces fonctions d’assistance sont suffisamment génériques pour être utilisées dans votre propre projet. Cet exemple les utilise dans les trois fichiers qui correspondent à chaque composant du modèle :

* *pnp_deviceinfo_component*
* *pnp_temperature_controller*
* *pnp_thermostat_component*

Par exemple, dans le fichier *pnp_deviceinfo_component*, la fonction `SendReportedPropertyForDeviceInformation` utilise deux des fonctions d’assistance :

```c
if ((jsonToSend = PnP_CreateReportedProperty(componentName, propertyName, propertyValue)) == NULL)
{
    LogError("Unable to build reported property response for propertyName=%s, propertyValue=%s", propertyName, propertyValue);
}
else
{
    const char* jsonToSendStr = STRING_c_str(jsonToSend);
    size_t jsonToSendStrLen = strlen(jsonToSendStr);

    if ((iothubClientResult = IoTHubDeviceClient_LL_SendReportedState(deviceClientLL, (const unsigned char*)jsonToSendStr, jsonToSendStrLen, NULL, NULL)) != IOTHUB_CLIENT_OK)
    {
        LogError("Unable to send reported state for property=%s, error=%d", propertyName, iothubClientResult);
    }
    else
    {
        LogInfo("Sending device information property to IoTHub.  propertyName=%s, propertyValue=%s", propertyName, propertyValue);
    }
}
```

Chaque composant de l’exemple suit ce modèle.

### <a name="code-flow"></a>Flux de code

La fonction `main` initialise la connexion et envoie l’ID du modèle :

```c
deviceClient = CreateDeviceClientAndAllocateComponents();
```

Le code utilise `PnP_CreateDeviceClientLLHandle` pour se connecter au hub IoT, définir `modelId` en tant qu’option et configurer la méthode d’appareil ainsi que les gestionnaires de rappels de jumeaux d’appareil pour les méthodes directes et les mises à jour des jumeaux d’appareil :

```c
g_pnpDeviceConfiguration.deviceMethodCallback = PnP_TempControlComponent_DeviceMethodCallback;
g_pnpDeviceConfiguration.deviceTwinCallback = PnP_TempControlComponent_DeviceTwinCallback;
g_pnpDeviceConfiguration.modelId = g_temperatureControllerModelId;
...

deviceClient = PnP_CreateDeviceClientLLHandle(&g_pnpDeviceConfiguration);
```

`&g_pnpDeviceConfiguration` contient également les informations de connexion. La variable d’environnement **IOTHUB_DEVICE_SECURITY_TYPE** détermine si l’exemple utilise une chaîne de connexion ou le service de provisionnement des appareils pour se connecter au hub IoT.

Quand l’appareil envoie un ID de modèle, il devient un appareil IoT Plug-and-Play.

Une fois les gestionnaires de rappels en place, l’appareil réagit aux mises à jour des jumeaux et aux appels de méthodes directes :

* Pour le rappel des jumeaux d’appareil, `PnP_TempControlComponent_DeviceTwinCallback` appelle la fonction `PnP_ProcessTwinData` pour traiter les données. `PnP_ProcessTwinData` utilise le *modèle visiteur* pour analyser le contenu JSON et consulter chaque propriété, en appelant `PnP_TempControlComponent_ApplicationPropertyCallback` sur chaque élément.

* Pour le rappel des commandes, la fonction `PnP_TempControlComponent_DeviceMethodCallback` utilise la fonction d’assistance afin d’analyser les noms de commandes et de composants :

    ```c
    PnP_ParseCommandName(methodName, &componentName, &componentNameSize, &pnpCommandName);
    ```

    La fonction `PnP_TempControlComponent_DeviceMethodCallback` appelle ensuite la commande sur le composant :

    ```c
    LogInfo("Received PnP command for component=%.*s, command=%s", (int)componentNameSize, componentName, pnpCommandName);
    if (strncmp((const char*)componentName, g_thermostatComponent1Name, g_thermostatComponent1Size) == 0)
    {
        result = PnP_ThermostatComponent_ProcessCommand(g_thermostatHandle1, pnpCommandName, rootValue, response, responseSize);
    }
    else if (strncmp((const char*)componentName, g_thermostatComponent2Name, g_thermostatComponent2Size) == 0)
    {
        result = PnP_ThermostatComponent_ProcessCommand(g_thermostatHandle2, pnpCommandName, rootValue, response, responseSize);
    }
    else
    {
        LogError("PnP component=%.*s is not supported by TemperatureController", (int)componentNameSize, componentName);
        result = PNP_STATUS_NOT_FOUND;
    }
    ```

La fonction `main` initialise les propriétés en lecture seule envoyées au hub IoT :

```c
PnP_TempControlComponent_ReportSerialNumber_Property(deviceClient);
PnP_DeviceInfoComponent_Report_All_Properties(g_deviceInfoComponentName, deviceClient);
PnP_TempControlComponent_Report_MaxTempSinceLastReboot_Property(g_thermostatHandle1, deviceClient);
PnP_TempControlComponent_Report_MaxTempSinceLastReboot_Property(g_thermostatHandle2, deviceClient);
```

La fonction `main` entre dans une boucle pour mettre à jour les données d’événement et de télémétrie pour chaque composant :

```c
while (true)
{
    PnP_TempControlComponent_SendWorkingSet(deviceClient);
    PnP_ThermostatComponent_SendTelemetry(g_thermostatHandle1, deviceClient);
    PnP_ThermostatComponent_SendTelemetry(g_thermostatHandle2, deviceClient);
}
```

La fonction `PnP_ThermostatComponent_SendTelemetry` vous montre comment utiliser le struct `PNP_THERMOSTAT_COMPONENT`. L’exemple utilise ce struct pour stocker des informations sur les deux thermostats dans le contrôleur de température. Le code utilise la fonction `PnP_CreateTelemetryMessageHandle` pour préparer le message et l’envoyer :

```c
messageHandle = PnP_CreateTelemetryMessageHandle(pnpThermostatComponent->componentName, temperatureStringBuffer);
...
iothubResult = IoTHubDeviceClient_LL_SendEventAsync(deviceClientLL, messageHandle, NULL, NULL);
```

Pour finir, la fonction `main` détruit les différents composants et ferme la connexion au hub.

[!INCLUDE [iot-pnp-clean-resources.md](iot-pnp-clean-resources.md)]
