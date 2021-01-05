---
title: Tutoriel - Utiliser MQTT pour créer un client d’appareil Azure IoT Plug-and-Play | Microsoft Docs
description: Tutoriel - Utiliser le protocole MQTT directement pour créer un client d’appareil IoT Plug-and-Play sans utiliser les kits Azure IoT Device SDK
author: ericmitt
ms.author: ericmitt
ms.date: 05/13/2020
ms.topic: tutorial
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 6852b0532b23e46c7b986926b21cd0b7e9f9736d
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421377"
---
# <a name="tutorial---use-mqtt-to-develop-an-iot-plug-and-play-device-client"></a>Tutoriel - Utiliser MQTT pour développer un client d’appareil IoT Plug-and-Play

Dans la mesure du possible, vous devez utiliser l’un des kits Azure IoT Device SDK pour créer des clients d’appareil IoT Plug-and-Play. Toutefois, dans certains scénarios, par exemple quand la mémoire de l’appareil utilisé est restreinte, vous devrez peut-être utiliser une bibliothèque MQTT pour communiquer avec votre hub IoT.

L’exemple de ce tutoriel utilise la bibliothèque MQTT [Eclipse Mosquitto](http://mosquitto.org/) et Visual Studio. Les étapes de ce tutoriel supposent que vous utilisez Windows sur votre ordinateur de développement.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-pnp-prerequisites](../../includes/iot-pnp-prerequisites.md)]

Pour effectuer ce tutoriel sur Windows, vous devez installer les logiciels suivants sur votre environnement Windows local :

* [Visual Studio (Community, Professional ou Enterprise)](https://visualstudio.microsoft.com/downloads/) : veillez à inclure la charge de travail **Développement Desktop en C++** quand vous [installez](/cpp/build/vscpp-step-0-installation?preserve-view=true&view=vs-2019) Visual Studio.
* [Git](https://git-scm.com/download/)
* [CMake](https://cmake.org/download/)

Utilisez l’*explorateur Azure IoT* pour ajouter un nouvel appareil appelé à votre hub IoT. Vous avez configuré votre hub IoT et l’explorateur Azure IoT lorsque vous avez terminé [Configurer votre environnement pour les guides de démarrages rapides et tutoriels IoT Plug-and-Play](set-up-environment.md) :

1. Lancez l’outil **explorateur Azure IoT**.
1. Sur la page **Hubs IoT**, sélectionnez **Afficher les appareils de ce hub**.
1. Sur la page **Appareils**, sélectionnez **+Nouveau**.
1. Créez un appareil appelé *my-mqtt-device* qui utilise une clé symétrique générée automatiquement.
1. Sur la page **Identité de l’appareil**, développez **Chaîne de connexion avec jeton SAS**.
1. Choisissez la **Clé primaire** à utiliser comme **Clé symétrique**, définissez le délai d’expiration sur 60 minutes, puis sélectionnez **Générer**.
1. Copiez la **chaîne de connexion avec jeton SAS** générée. Vous l’utiliserez plus loin dans le didacticiel.

## <a name="clone-sample-repo"></a>Cloner l’exemple de dépôt

Utilisez la commande suivante pour cloner l’exemple de dépôt à un emplacement approprié sur votre ordinateur local :

```cmd
git clone https://github.com/Azure-Samples/IoTMQTTSample.git
```

## <a name="install-mqtt-library"></a>Installer la bibliothèque MQTT

Utilisez l’outil `vcpkg` pour télécharger et générer la bibliothèque Eclipse Mosquitto.

Utilisez la commande suivante pour cloner le dépôt **Vcpkg** à un emplacement approprié sur votre ordinateur local :

```cmd
git clone https://github.com/Microsoft/vcpkg.git
```

Utilisez les commandes suivantes pour préparer l’environnement `vcpkg`. Vous avez besoin d’une invite de commandes avec élévation de privilèges quand vous exécutez `vcpkg integrate install` :

```cmd
cd vcpkg
.\bootstrap-vcpkg.bat
.\vcpkg integrate install
```

Utilisez la commande suivante pour télécharger et générer la bibliothèque Eclipse Mosquitto :

```cmd
.\vcpkg install mosquitto:x64-windows
```

## <a name="migrate-the-sample-to-iot-plug-and-play"></a>Migrer l’exemple vers IoT Plug-and-Play

### <a name="review-the-non-iot-plug-and-play-sample-code"></a>Examiner l’exemple de code non-IoT Plug-and-Play

Mettez à jour le code avec les détails de votre hub IoT et de votre appareil avant de le générer et de l’exécuter.

Pour voir l’exemple de code dans Visual Studio, ouvrez le fichier solution *MQTTWin32.sln* situé dans le dossier *IoTMQTTSample\src\Windows*.

Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet **TelemetryMQTTWin32**, puis sélectionnez **Définir comme projet de démarrage**.

Dans le projet **TelemetryMQTTWin32**, ouvrez le fichier source **MQTT_Mosquitto.cpp**. Mettez à jour les définitions d’informations de connexion avec les détails de l’appareil que vous avez notés précédemment. Remplacez les espaces réservés du jeton pour :

* l’identificateur `IOTHUBNAME` par le nom de votre hub IoT.
* l’identificateur `DEVICEID` par `my-mqtt-device`.
* l’identificateur `PWD` par la partie correcte de la chaîne de connexion du jeton SAS que vous avez générée pour l’appareil. Utilisez la partie de la chaîne de connexion de `SharedAccessSignature sr=` jusqu’à la fin.

Vérifiez que le code fonctionne correctement en démarrant l’explorateur Azure IoT et en commençant à écouter la télémétrie.

Exécutez l’application (Ctrl+F5). Au bout de quelques secondes, vous voyez une sortie ressemblant à ce qui suit :

:::image type="content" source="media/tutorial-use-mqtt/mqtt-sample-output.png" alt-text="Sortie de l’exemple d’application MQTT":::

Dans l’explorateur Azure IoT, vous pouvez voir que l’appareil n’est pas un appareil IoT Plug-and-Play :

:::image type="content" source="media/tutorial-use-mqtt/non-pnp-iot-explorer.png" alt-text="Appareil non-IoT Plug-and-Play dans l’explorateur Azure IoT":::

### <a name="make-the-device-an-iot-plug-and-play-device"></a>Convertir l’appareil en appareil IoT Plug-and-Play

Un appareil IoT Plug-and-Play doit respecter un ensemble de conventions simples. Si un appareil envoie un ID de modèle quand il se connecte, il devient un appareil IoT Plug-and-Play.

Dans cet exemple, vous ajoutez un ID de modèle au paquet de connexion MQTT. Vous transmettez l’ID de modèle comme paramètre de chaîne de requête dans `USERNAME` et remplacez l’`api-version` par `2020-09-30` :

```c
// computed Host Username and Topic
//#define USERNAME IOTHUBNAME ".azure-devices.net/" DEVICEID "/?api-version=2018-06-30"
#define USERNAME IOTHUBNAME ".azure-devices.net/" DEVICEID "/?api-version=2020-09-30&model-id=dtmi:com:example:Thermostat;1"
#define PORT 8883
#define HOST IOTHUBNAME //".azure-devices.net"
#define TOPIC "devices/" DEVICEID "/messages/events/"
```

Regénérez l’exemple et exécutez-le.

Le jumeau d’appareil inclut désormais l’ID de modèle :

:::image type="content" source="media/tutorial-use-mqtt/model-id-iot-explorer.png" alt-text="Voir l’ID de modèle dans l’explorateur Azure IoT":::

Vous pouvez maintenant naviguer dans le composant IoT Plug-and-Play :

:::image type="content" source="media/tutorial-use-mqtt/components-iot-explorer.png" alt-text="Voir les composants dans l’explorateur Azure IoT":::

Vous pouvez maintenant modifier votre code d’appareil pour implémenter la télémétrie, les propriétés et les commandes définies dans votre modèle. Pour voir un exemple d’implémentation de l’appareil à thermostat avec la bibliothèque Mosquitto, consultez [Using MQTT PnP with Azure IoTHub without the IoT SDK on Windows](https://github.com/Azure-Samples/IoTMQTTSample/tree/master/src/Windows/PnPMQTTWin32) (Utilisation de MQTT PnP avec Azure IoT Hub sans le SDK IoT sur Windows) sur GitHub.

> [!NOTE]
>Le client utilise le fichier de certificat racine `IoTHubRootCA_Baltimore.pem` pour vérifier l’identité du hub IoT auquel il se connecte.

### <a name="mqtt-topics"></a>Rubriques MQTT

Les définitions suivantes s’appliquent aux rubriques MQTT utilisées par l’appareil pour envoyer des informations au hub IoT. Pour en savoir plus, consultez [Communication avec votre IoT Hub à l’aide du protocole MQTT](../iot-hub/iot-hub-mqtt-support.md) :

* `DEVICE_CAPABILITIES_MESSAGE` définit la rubrique utilisée par l’appareil pour signaler les interfaces qu’il implémente.
* `DEVICETWIN_PATCH_MESSAGE` définit la rubrique utilisée par l’appareil pour signaler les mises à jour de propriétés au hub IoT.
* `DEVICE_TELEMETRY_MESSAGE` définit la rubrique utilisée par l’appareil pour envoyer les données de télémétrie au hub IoT.

Pour plus d’informations sur MQTT, consultez le dépôt GitHub [MQTT Samples for Azure IoT](https://github.com/Azure-Samples/IoTMQTTSample/) (Exemples MQTT pour Azure IoT).
  
## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à modifier un client d’appareil MQTT pour qu’il respecte les conventions IoT Plug-and-Play. Pour en savoir plus sur la préversion d’IoT Plug-and-Play, consultez :

> [!div class="nextstepaction"]
> [Architecture](concepts-architecture.md)

Pour en savoir plus sur la prise en charge du protocole MQTT par IoT Hub, consultez :

> [!div class="nextstepaction"]
> [Communication avec votre IoT Hub à l’aide du protocole MQTT](../iot-hub/iot-hub-mqtt-support.md)