---
title: Communiquer avec une application d’appareil en C# avec des flux d'appareil IoT Hub
description: Dans ce guide de démarrage rapide, vous exécutez deux exemples d’applications C# qui communiquent par le biais d’un flux d’appareil établi par le biais d’IoT Hub.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: quickstart
ms.custom: mvc, devx-track-azurecli
ms.date: 03/14/2019
ms.author: robinsh
ms.openlocfilehash: 8a8e6f12a8709935a6fa9551261c56fc8f2aabbd
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94832056"
---
# <a name="quickstart-communicate-to-a-device-application-in-c-via-iot-hub-device-streams-preview"></a>Démarrage rapide : Communiquer avec une application d’appareil en C# par le biais de flux d’appareil IoT Hub (préversion)

[!INCLUDE [iot-hub-quickstarts-3-selector](../../includes/iot-hub-quickstarts-3-selector.md)]

Azure IoT Hub prend actuellement en charge les flux d’appareil en tant que [fonctionnalité d’évaluation](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Les [flux d’appareil IoT Hub](./iot-hub-device-streams-overview.md) permettent aux applications de service et d’appareil de communiquer de manière sécurisée à travers des pare-feux. Ce guide de démarrage rapide implique deux applications C# qui tirent parti de flux d’appareil pour envoyer des données dans les deux sens (écho).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Pour le moment, la préversion des flux d’appareil est uniquement prise en charge pour les hubs IoT créés dans les régions suivantes :
  * USA Centre
  * EUAP USA Centre
  * Europe Nord
  * Asie Sud-Est

* Les deux exemples d’applications que vous exécutez dans ce guide de démarrage rapide sont écrits en C#. Votre machine de développement doit disposer du SDK .NET Core 2.1.0 ou version ultérieure.

    Téléchargez le [Kit SDK .NET Core pour plusieurs plateformes sur .NET](https://www.microsoft.com/net/download/all).

    Vérifiez la version actuelle de C# sur votre machine de développement à l’aide de la commande suivante :

    ```
    dotnet --version
    ```

* [Téléchargez les exemples C# Azure IOT](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip) et extrayez l’archive ZIP. Vous en avez besoin à la fois côté appareil et côté service.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Inscrire un appareil

Un appareil doit être inscrit dans votre hub IoT pour pouvoir se connecter. Dans cette section, vous utilisez Azure Cloud Shell pour inscrire un appareil simulé.

1. Pour créer l’identité d’appareil, exécutez la commande suivante dans Cloud Shell :

   > [!NOTE]
   > * Remplacez l’espace réservé *YourIoTHubName* par le nom que vous avez choisi pour votre hub IoT.
   > * Pour le nom de l’appareil que vous êtes en train d’inscrire, nous vous recommandons d’utiliser *MyDevice* comme indiqué. Si vous choisissez un autre nom pour votre appareil, utilisez-le pour l’ensemble de cet article et mettez à jour le nom de l’appareil dans les exemples d’application avant de les exécuter.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyDevice
    ```

1. Pour obtenir la *chaîne de connexion d’appareil* pour celui que vous venez d’inscrire, exécutez la commande suivante dans Cloud Shell :

   > [!NOTE]
   > Remplacez l’espace réservé *YourIoTHubName* par le nom que vous avez choisi pour votre hub IoT.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id MyDevice --output table
    ```

    Notez la chaîne de connexion d’appareil retournée ; vous en aurez besoin plus loin dans ce guide de démarrage rapide. Cela ressemble à l’exemple suivant :

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyDevice;SharedAccessKey={YourSharedAccessKey}`

3. Vous avez également besoin de la *chaîne de connexion du service* de votre hub IoT pour permettre à l’application côté service de se connecter à votre hub IoT et d’établir un flux d’appareil. La commande suivante récupère cette valeur de votre hub IoT :

   > [!NOTE]
   > Remplacez l’espace réservé *YourIoTHubName* par le nom que vous avez choisi pour votre hub IoT.

    ```azurecli-interactive
    az iot hub show-connection-string --policy-name service --name {YourIoTHubName} --output table
    ```

    Notez la chaîne de connexion de service retournée ; vous en aurez besoin plus loin dans ce guide de démarrage rapide. Cela ressemble à l’exemple suivant :

   `"HostName={YourIoTHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}"`

## <a name="communicate-between-the-device-and-the-service-via-device-streams"></a>Communiquer entre appareil et service par le biais de flux d’appareil

Dans cette section, vous exécutez l’application côté appareil et l’application côté service et communiquez entre les deux.

### <a name="run-the-service-side-application"></a>Exécuter l’application côté service

Dans une fenêtre de terminal locale, accédez au répertoire `iot-hub/Quickstarts/device-streams-echo/service` dans votre dossier de projet décompressé. Gardez à portée de main les informations suivantes :

| Nom du paramètre | Valeur du paramètre |
|----------------|-----------------|
| `ServiceConnectionString` | Chaîne de connexion de service de votre hub IoT. |
| `MyDevice` | Identificateur de l’appareil que vous avez créé précédemment. |

Compilez et exécutez le code avec les commandes suivantes :

```
cd ./iot-hub/Quickstarts/device-streams-echo/service/

# Build the application
dotnet build

# Run the application
# In Linux or macOS
dotnet run "{ServiceConnectionString}" "MyDevice"

# In Windows
dotnet run {ServiceConnectionString} MyDevice
```
L’application attendra que l’application de l’appareil soit disponible.

> [!NOTE]
> Un délai d’expiration se produit si l’application côté appareil ne répond pas dans les temps.

### <a name="run-the-device-side-application"></a>Exécuter l’application côté appareil

Dans une autre fenêtre de terminal locale, accédez au répertoire `iot-hub/Quickstarts/device-streams-echo/device` dans votre dossier de projet décompressé. Gardez à portée de main les informations suivantes :

| Nom du paramètre | Valeur du paramètre |
|----------------|-----------------|
| `DeviceConnectionString` | Chaîne de connexion d’appareil de votre hub IoT. |

Compilez et exécutez le code avec les commandes suivantes :

```
cd ./iot-hub/Quickstarts/device-streams-echo/device/

# Build the application
dotnet build

# Run the application
# In Linux or macOS
dotnet run "{DeviceConnectionString}"

# In Windows
dotnet run {DeviceConnectionString}
```

À la fin de la dernière étape, l’application côté service lance un flux sur votre appareil. Une fois le flux établi, l’application envoie un tampon de chaîne au service sur le flux de données. Dans cet exemple, l’application côté service renvoie simplement les mêmes données à l’appareil, ce qui démontre une communication bidirectionnelle réussie entre les deux applications.

Sortie de console côté appareil :

![Sortie de console côté appareil](./media/quickstart-device-streams-echo-csharp/device-console-output.png)

Sortie de console côté service :

![Sortie de console côté service](./media/quickstart-device-streams-echo-csharp/service-console-output.png)

Le trafic envoyé sur le flux est traité par tunnel par le biais du hub IoT au lieu d’être envoyé directement. Les avantages sont détaillés dans [Avantages des flux d'appareils](./iot-hub-device-streams-overview.md#benefits).

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [iot-hub-quickstarts-clean-up-resources-device-streams](../../includes/iot-hub-quickstarts-clean-up-resources-device-streams.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez configuré un hub IoT, inscrit un appareil, établi un flux d’appareil entre des applications C# côté appareil et côté service, puis utilisé le flux pour envoyer des données dans les deux sens entre les applications.

Pour en savoir plus sur les flux d’appareil, consultez :

> [!div class="nextstepaction"]
> [Vue d’ensemble des flux d’appareil](./iot-hub-device-streams-overview.md)
