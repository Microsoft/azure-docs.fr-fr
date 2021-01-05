---
title: Bibliothèques et Kits de développement logiciel (SDK) IoT Plug-and-Play
description: Informations sur les bibliothèques d’appareils et de services disponibles pour développer des solutions IoT Plug-and-Play.
author: rido-min
ms.author: rmpablos
ms.date: 07/22/2020
ms.topic: reference
ms.service: iot-pnp
services: iot-pnp
ms.custom: mvc
ms.openlocfilehash: 3ff01455c56dd259ea64878f912eb3f2431547e7
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95495250"
---
# <a name="microsoft-sdks-for-iot-plug-and-play"></a>Kits SDK Microsoft pour IoT Plug-and-Play

Les bibliothèques et Kits de développement logiciel (SDK) IoT Plug-and-Play permettent aux développeurs de créer des solutions IoT à l’aide d’un large éventail de langages de programmation sur plusieurs plateformes. Le tableau suivant contient des liens vers des exemples et des guides de démarrage rapide pour aider votre prise en main :

## <a name="device-sdks"></a>Kits de développement logiciel (SDK) d’appareil

| Langage | Package | Dépôt de code | Exemples | Démarrage rapide | Informations de référence |
|---|---|---|---|---|---|
| C - Appareil | [vcpkg 1.3.9](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/setting_up_vcpkg.md) | [GitHub](https://github.com/Azure/azure-iot-sdk-c) | [Exemples](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples/pnp) | [Connexion à IoT Hub](quickstart-connect-device.md) | [Référence](/azure/iot-hub/iot-c-sdk-ref/) |
| .NET - Appareil | [NuGet 1.31.0](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client) | [GitHub](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/) | [Exemples](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/iot-hub/Samples/device/PnpDeviceSamples) | [Connexion à IoT Hub](quickstart-connect-device.md) | [Référence](/dotnet/api/microsoft.azure.devices.client?preserve-view=true&view=azure-dotnet) |
| Java - Appareil | [Maven 1.26.0](https://mvnrepository.com/artifact/com.microsoft.azure.sdk.iot/iot-device-client) | [GitHub](https://github.com/Azure/azure-iot-sdk-java/tree/master/) | [Exemples](https://github.com/Azure/azure-iot-sdk-java/tree/master/device/iot-device-samples/pnp-device-sample) | [Connexion à IoT Hub](quickstart-connect-device.md) | [Référence](/java/api/com.microsoft.azure.sdk.iot.device?preserve-view=true&view=azure-java-stable) |
| Python - Appareil | [pip 2.3.0](https://pypi.org/project/azure-iot-device/) | [GitHub](https://github.com/Azure/azure-iot-sdk-python/tree/master/) | [Exemples](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/pnp) | [Connexion à IoT Hub](quickstart-connect-device.md) | [Référence](/python/api/azure-iot-device/azure.iot.device?preserve-view=true&view=azure-python) |
| Node - Appareil | [npm 1.17.2](https://www.npmjs.com/package/azure-iot-device)  | [GitHub](https://github.com/Azure/azure-iot-sdk-node/tree/master/) | [Exemples](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples/pnp) | [Connexion à IoT Hub](quickstart-connect-device.md) | [Référence](/javascript/api/azure-iot-device/?preserve-view=true&view=azure-node-latest) |
| Embedded C - Appareil | N/A | [GitHub](https://github.com/Azure/azure-sdk-for-c/)| [Exemples](howto-use-embedded-c.md#samples) | [Utilisation d’Embedded C](howto-use-embedded-c.md) | N/A

## <a name="service-sdks"></a>Kits de développement logiciel (SDK) IoT Service

| Plateforme  | Package | Dépôt de code | Exemples | Démarrage rapide | Informations de référence |
|---|---|---|---|---|---|
| .NET - service IoT Hub | [NuGet 1.27.1](https://www.nuget.org/packages/Microsoft.Azure.Devices ) | [GitHub](https://github.com/Azure/azure-iot-sdk-csharp) | [Exemples](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/iot-hub/Samples/service/PnpServiceSamples) | NON APPLICABLE | [Référence](/dotnet/api/microsoft.azure.devices?preserve-view=true&view=azure-dotnet) |
| Java - service IoT Hub | [Maven 1.26.0](https://mvnrepository.com/artifact/com.microsoft.azure.sdk.iot/iot-service-client/1.26.0) | [GitHub](https://github.com/Azure/azure-iot-sdk-java) | [Exemples](https://github.com/Azure/azure-iot-sdk-java/tree/master/service/iot-service-samples/pnp-service-sample) | NON APPLICABLE | [Référence](/java/api/com.microsoft.azure.sdk.iot.service?preserve-view=true&view=azure-java-stable) |
| Node - service IoT Hub | [npm 1.13.0](https://www.npmjs.com/package/azure-iothub) | [GitHub](https://github.com/Azure/azure-iot-sdk-node) | [Exemples](https://github.com/Azure/azure-iot-sdk-node/tree/master/service/samples) | NON APPLICABLE | [Référence](/javascript/api/azure-iothub/?preserve-view=true&view=azure-node-latest) |
| Python - service Digital Twins | [pip 2.2.3](https://pypi.org/project/azure-iot-hub) | [GitHub](https://github.com/Azure/azure-iot-sdk-python) | [Exemples](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-hub/samples) | [Interagir avec l’API IoT Hub Digital Twins](quickstart-service.md) | N/A |
| Node - service Digital Twins | [npm 1.13.0](https://www.npmjs.com/package/azure-iot-digitaltwins-service) | [GitHub](https://github.com/Azure/azure-iot-sdk-node) | [Exemples](https://github.com/Azure/azure-iot-sdk-node/tree/master/service/samples/javascript) | [Interagir avec l’API IoT Hub Digital Twins](quickstart-service.md) | N/A |

## <a name="next-steps"></a>Étapes suivantes

Pour tester les kits SDK et les bibliothèques, consultez le [Guide du développeur](concepts-developer-guide-device.md), ainsi que les [guides de démarrage rapide d’appareil](quickstart-connect-device.md) et les [guides de démarrage rapide de service](quickstart-service.md).