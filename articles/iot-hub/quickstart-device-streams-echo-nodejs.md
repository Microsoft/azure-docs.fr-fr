---
title: Communiquer avec une application d’appareil en Node.js avec des flux d'appareil Azure IoT Hub
description: Dans ce guide de démarrage rapide, vous allez exécuter une application côté service Node.js qui communique avec un appareil IoT par le biais d’un flux d’appareil.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: quickstart
ms.custom: mvc, devx-track-js, devx-track-azurecli
ms.date: 03/14/2019
ms.author: robinsh
ms.openlocfilehash: 678955970f3eeb87a10c43cd43effc3464db7794
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94832005"
---
# <a name="quickstart-communicate-to-a-device-application-in-nodejs-via-iot-hub-device-streams-preview"></a>Démarrage rapide : Communiquer avec une application d’appareil dans Node.js par le biais de flux d’appareil IoT Hub (préversion)

[!INCLUDE [iot-hub-quickstarts-3-selector](../../includes/iot-hub-quickstarts-3-selector.md)]

Dans ce guide de démarrage rapide, vous exécutez une application côté service et vous configurez la communication entre un appareil et un service à l’aide de flux d’appareil. Les flux d’appareil Azure IoT Hub permettent aux applications de service et d’appareil de communiquer de manière sécurisée à travers des pare-feux. Dans la préversion publique, le SDK Node.js prend uniquement en charge les flux d’appareil côté service. Par conséquent, le présent guide de démarrage rapide couvre uniquement les instructions permettant d’exécuter l’application côté service.

## <a name="prerequisites"></a>Prérequis

* Avoir suivi le guide de démarrage rapide [Communiquer avec des applications d’appareil en C par le biais de flux d’appareil IoT Hub](./quickstart-device-streams-echo-c.md) ou [Communiquer avec des applications d’appareil en C# par le biais de flux d’appareil IoT Hub](./quickstart-device-streams-echo-csharp.md).

* Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

* [Node.js 10+](https://nodejs.org).

    Vous pouvez vérifier la version actuelle de Node.js sur votre machine de développement à l’aide de la commande suivante :

    ```cmd/sh
    node --version
    ```

* [Un exemple de projet Node.js](https://github.com/Azure-Samples/azure-iot-samples-node/archive/streams-preview.zip).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

Microsoft Azure IoT Hub prend actuellement en charge les flux d’appareil en tant que [fonctionnalité d’évaluation](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

> [!IMPORTANT]
> Pour le moment, la préversion des flux d’appareil est uniquement prise en charge pour les hubs IoT créés dans les régions suivantes :
>
> * USA Centre
> * EUAP USA Centre
> * Europe Nord
> * Asie Sud-Est

## <a name="create-an-iot-hub"></a>Créer un hub IoT

Si vous avez terminé le [Démarrage rapide : envoyer des données de télémétrie à partir d’un appareil à un IoT Hub](quickstart-send-telemetry-node.md) précédent, vous pouvez ignorer cette étape.

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Inscrire un appareil

Si vous avez terminé le [Démarrage rapide : envoyer des données de télémétrie à partir d’un appareil à un IoT Hub](quickstart-send-telemetry-node.md) précédent, vous pouvez ignorer cette étape.

Un appareil doit être inscrit dans votre hub IoT pour pouvoir se connecter. Dans ce démarrage rapide, vous utilisez Azure Cloud Shell pour inscrire un appareil simulé.

1. Exécutez les commandes suivantes dans Azure Cloud Shell pour créer l’identité d’appareil.

   **YourIoTHubName** : Remplacez l’espace réservé ci-dessous par le nom que vous avez choisi pour votre hub IoT.

   **MyDevice** : il s’agit du nom de l’appareil que vous inscrivez. Nous vous recommandons d’utiliser **MyDevice** comme indiqué. Si vous choisissez un autre nom pour votre appareil, vous devez utiliser ce nom tout au long de cet article et mettre à jour le nom de l’appareil dans les exemples d’application avant de les exécuter.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyDevice
    ```

2. Vous avez également besoin d’une *chaîne de connexion de service* pour activer l’application back-end afin de vous connecter à votre IoT Hub et récupérer les messages. La commande suivante récupère la chaîne de connexion de service correspondant à votre hub IoT :

    **YourIoTHubName** : Remplacez l’espace réservé ci-dessous par le nom que vous avez choisi pour votre hub IoT.

    ```azurecli-interactive
    az iot hub show-connection-string --policy-name service --name {YourIoTHubName} --output table
    ```

    Notez la chaîne de connexion de service retournée ; vous en aurez besoin plus loin dans ce guide de démarrage rapide. Cela ressemble à l’exemple suivant :

   `"HostName={YourIoTHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}"`

## <a name="communicate-between-device-and-service-via-device-streams"></a>Communiquer entre appareil et service par le biais de flux d’appareil

Dans cette section, vous exécutez l’application côté appareil et l’application côté service et communiquez entre les deux.

### <a name="run-the-device-side-application"></a>Exécuter l’application côté appareil

Comme mentionné précédemment, le SDK Node.js IoT Hub prend uniquement en charge des flux d’appareil côté service. Pour une application côté appareil, utilisez l’un des programmes d’appareil complémentaires disponibles dans ces guides de démarrage rapide :

* [Communiquer avec des applications d’appareil en C via les flux d'appareils IoT Hub](./quickstart-device-streams-echo-c.md)

* [Communiquer avec des applications d’appareil en C# via les flux d'appareils IoT Hub](./quickstart-device-streams-echo-csharp.md)

Vérifiez que l’application côté appareil est en cours d’exécution avant de passer à l’étape suivante.

### <a name="run-the-service-side-application"></a>Exécuter l’application côté service

L’application Node.js côté service utilisée dans ce guide de démarrage rapide présente les fonctionnalités suivantes :

* Crée un flux d’appareil vers un appareil IoT.
* Lit une entrée à partir de la ligne de commande et l’envoie à l’application d’appareil, qui la renvoie à son tour.

Le code illustre le processus de lancement d’un flux d’appareil ainsi que la façon de l’utiliser pour envoyer et recevoir des données.

Si l’application côté appareil est en cours d’exécution, effectuez les étapes ci-dessous dans une fenêtre de terminal locale pour exécuter l’application côté service dans Node.js :

* Indiquez les informations d’identification de votre service et l’ID de l’appareil sous forme de variables d’environnement.
 
   ```cmd/sh
   # In Linux
   export IOTHUB_CONNECTION_STRING="{ServiceConnectionString}"
   export STREAMING_TARGET_DEVICE="MyDevice"

   # In Windows
   SET IOTHUB_CONNECTION_STRING={ServiceConnectionString}
   SET STREAMING_TARGET_DEVICE=MyDevice
   ```
  
   Modifiez l’espace réservé ServiceConnectionString pour qu’il corresponde à votre chaîne de connexion de service, et **MyDevice** pour qu’il corresponde à votre ID d’appareil si vous lui avez donné un nom différent.

* Accédez à `Quickstarts/device-streams-service` dans votre dossier de projet décompressé, puis exécutez l’exemple en utilisant Node.

   ```cmd/sh
   cd azure-iot-samples-node-streams-preview/iot-hub/Quickstarts/device-streams-service
    
   # Install the preview service SDK, and other dependencies
   npm install azure-iothub@streams-preview
   npm install

   node echo.js
   ```

À la fin de la dernière étape, le programme côté service lance un flux à votre appareil et, une fois établi, envoie un tampon de chaîne au service sur le flux. Dans cet exemple, le programme côté service lit simplement l’`stdin` sur le terminal et l’envoie à l’appareil, qui la renvoie à son tour. Cet exemple montre une communication bidirectionnelle réussie entre les deux applications.

![Sortie de la console côté service](./media/quickstart-device-streams-echo-nodejs/service-console-output.png)

Vous pouvez ensuite mettre fin au programme en appuyant de nouveau sur Entrée.

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [iot-hub-quickstarts-clean-up-resources-device-streams](../../includes/iot-hub-quickstarts-clean-up-resources-device-streams.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez configuré un hub IoT, inscrit un appareil, établi un flux d’appareil entre des applications côté appareil et côté service, puis utilisé le flux pour envoyer des données dans les deux sens entre les applications.

Utilisez les liens ci-dessous pour en savoir plus sur les flux d’appareils :

> [!div class="nextstepaction"]
> [Vue d’ensemble des flux d’appareil](./iot-hub-device-streams-overview.md) 
