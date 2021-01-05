---
title: Comprendre les kits IoT Azure SDK | Microsoft Docs
description: Guide du développeur - informations et liens vers divers kits Azure IoT device et service SDK que vous pouvez utiliser pour créer des applications d’appareil et des applications principales.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/14/2020
ms.custom:
- mqtt
- 'Role: IoT Device'
- 'Role: Cloud Development'
ms.openlocfilehash: 9921ed500f831f2a002335fe83eae8f1bb47ecd4
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97095336"
---
# <a name="understand-and-use-azure-iot-hub-sdks"></a>Comprendre et utiliser les kits Azure IoT Hub SDK

Il existe deux catégories de kits de développement logiciel (SDK) permettant de travailler avec IoT Hub :

* Les kits SDK **IoT Hub Device** vous permettent de générer des applications qui s’exécutent sur vos appareils IoT à l’aide d’un client d’appareil ou d’un client de module. Ces applications envoient des données de télémétrie à votre hub IoT et reçoivent le cas échéant des messages ou des mises à jour de tâche, de méthode ou de jumeau de ce hub IoT.  Vous pouvez également utiliser un client de module afin de créer des [modules](../iot-edge/iot-edge-modules.md) pour le [runtime Azure IoT Edge](../iot-edge/about-iot-edge.md).

* Les kits SDK **IoT Hub Service** vous permettent de créer des applications back-end pour gérer votre hub IoT et, si vous le souhaitez, d’envoyer des messages, de planifier des tâches, d’appeler des méthodes directes ou d’envoyer les mises à jour de propriétés souhaitées à vos modules ou appareils IoT.

De plus, nous fournissons également un ensemble de SDK pour travailler avec le [Service Device Provisioning](../iot-dps/about-iot-dps.md).

* Les **SDK Device Provisioning** vous permettent de créer des applications qui s’exécutent sur vos appareils IoT pour communiquer avec le Service Device Provisioning.

* Les **SDK Service Provisioning** vous permettent de créer des applications back-end pour gérer vos inscriptions dans le Service Device Provisioning.

Découvrez les [avantages du développement avec les Kits de développement logiciel (SDK) Azure IoT](https://azure.microsoft.com/blog/benefits-of-using-the-azure-iot-sdks-in-your-azure-iot-solution/).

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="os-platform-and-hardware-compatibility"></a>Compatibilité des plateformes de système d’exploitation et du matériel

Les plateformes prises en charge pour les Kits de développement logiciel (SDK) se trouvent dans le document [Prise en charge des plateformes par les Kits de développement logiciel (SDK) Azure IoT](iot-hub-device-sdk-platform-support.md).

Pour plus d’informations sur la compatibilité des Kits de développement logiciel (SDK) avec un matériel particulier, consultez le [catalogue d’appareils Azure Certified pour IoT](https://catalog.azureiotsolutions.com/) ou un référentiel spécifique.

## <a name="azure-iot-hub-device-sdks"></a>Kits Azure IoT Hub Device SDK

Les kits Microsoft Azure IoT device SDK contiennent du code qui facilite la création d’applications qui se connectent aux services Azure IoT Hub et sont gérés par eux.

Kit Azure IoT device SDK pour .NET : 

* Téléchargement à partir de [NuGet](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/).  L’espace de noms est Microsoft.Azure.Devices.Clients, qui contient des clients IoT Hub Device (DeviceClient, ModuleClient).
* [Code source](https://github.com/Azure/azure-iot-sdk-csharp)
* [Informations de référence sur l'API](/dotnet/api/microsoft.azure.devices?view=azure-dotnet)
* [Informations de référence sur les modules](/dotnet/api/microsoft.azure.devices.client.moduleclient?view=azure-dotnet)


Azure IoT Hub device SDK pour Embedded C (ANSI C - C99) :
* [Générer le kit SDK Embedded C](https://github.com/Azure/azure-sdk-for-c/tree/master/sdk/docs/iot#build)
* [Code source](https://github.com/Azure/azure-sdk-for-c)
* [Graphique des tailles](https://github.com/Azure/azure-sdk-for-c/tree/master/sdk/docs/iot#size-chart) pour des appareils avec contraintes.
* [Informations de référence sur l'API](https://azuresdkdocs.blob.core.windows.net/$web/dotnet/Azure.Identity/1.0.0/api/index.html)


Azure IoT Hub device SDK pour C (ANSI C - C99) :

* installation à partir d’[apt-get, MBED, Arduino IDE ou iOS](https://github.com/Azure/azure-iot-sdk-c/blob/master/readme.md#packages-and-libraries)
* [Code source](https://github.com/Azure/azure-iot-sdk-c)
* [Compiler le kit Device SDK C](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/readme.md#compiling-the-c-device-sdk)
* [Informations de référence sur l'API](/azure/iot-hub/iot-c-sdk-ref/)
* [Informations de référence sur les modules](/azure/iot-hub/iot-c-sdk-ref/iothub-module-client-h)
* [Portage du kit SDK C vers d’autres plateformes](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md)
* [Documentation du développeur](https://github.com/Azure/azure-iot-sdk-c/tree/master/doc) pour plus d’informations sur la compilation croisée, la procédure pour bien démarrer avec différentes plateformes, et ainsi de suite.
* [Informations sur la consommation de ressources du Kit de développement logiciel (SDK) Azure IoT Hub C](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/c_sdk_resource_information.md)

Kit Azure IoT device SDK pour Java :

* Ajout au projet [Maven](https://github.com/Azure/azure-iot-sdk-java/blob/master/doc/java-devbox-setup.md#for-the-device-sdk)
* [Code source](https://github.com/Azure/azure-iot-sdk-java)
* [Informations de référence sur l'API](/java/api/com.microsoft.azure.sdk.iot.device)
* [Informations de référence sur les modules](/java/api/com.microsoft.azure.sdk.iot.device.moduleclient?view=azure-java-stable)

Kit Azure IoT device SDK pour Node.js :

* Installation à partir de [npm](https://www.npmjs.com/package/azure-iot-device)
* [Code source](https://github.com/Azure/azure-iot-sdk-node)
* [Informations de référence sur l'API](/javascript/api/azure-iot-device/?view=azure-iot-typescript-latest)
* [Informations de référence sur les modules](/javascript/api/azure-iot-device/moduleclient?view=azure-node-latest)

Kit Azure IoT device SDK pour Python :

* Installation à partir de [pip](https://pypi.org/project/azure-iot-device/)
* [Code source](https://github.com/Azure/azure-iot-sdk-python)
* [Informations de référence sur l'API](/python/api/azure-iot-device)

Kit Azure IoT Hub device SDK pour iOS :

* Installation à partir de [CocoaPod](https://cocoapods.org/pods/AzureIoTHubClient)
* [Exemples](https://github.com/Azure-Samples/azure-iot-samples-ios)
* Informations de référence sur l’API : voir [Informations de référence sur l’API C](/azure/iot-hub/iot-c-sdk-ref/)

## <a name="azure-iot-hub-service-sdks"></a>Kits Azure IoT Hub Service SDK

Les kits Azure IoT service SDK contiennent du code pour faciliter la création d’applications qui interagissent directement avec IoT Hub pour gérer les appareils et la sécurité.

Kit Azure IoT Hub service SDK pour .NET :

* Téléchargement à partir de [NuGet](https://www.nuget.org/packages/Microsoft.Azure.Devices/).  L’espace de noms est Microsoft.Azure.Devices, qui contient des clients IoT Hub Service (RegistryManager, ServiceClients).
* [Code source](https://github.com/Azure/azure-iot-sdk-csharp)
* [Informations de référence sur l'API](/dotnet/api/microsoft.azure.devices)

Kit Azure IoT Hub service SDK pour Java :

* Ajout au projet [Maven](https://github.com/Azure/azure-iot-sdk-java/blob/master/doc/java-devbox-setup.md#for-the-service-sdk)
* [Code source](https://github.com/Azure/azure-iot-sdk-java)
* [Informations de référence sur l'API](/java/api/com.microsoft.azure.sdk.iot.service)

Kit Azure IoT Hub service SDK pour Node.js :

* Téléchargement à partir de [npm](https://www.npmjs.com/package/azure-iothub)
* [Code source](https://github.com/Azure/azure-iot-sdk-node)
* [Informations de référence sur l'API](/javascript/api/azure-iothub/?view=azure-iot-typescript-latest)

Kit Azure IoT Hub service SDK pour Python :

* Téléchargement à partir de [pip](https://pypi.python.org/pypi/azure-iot-hub/)
* [Code source](https://github.com/Azure/azure-iot-sdk-python/tree/master)
* [Informations de référence sur l'API](/python/api/azure-iot-hub)

Kit Azure IoT Hub service SDK pourC :

Le kit Azure IoT service SDK pour C n’est plus en cours de développement actif.
Nous allons continuer à résoudre les bogues critiques tels que les plantages, l’altération des données et les failles de sécurité. Toutefois, nous n’ajouterons aucune nouvelle fonctionnalité, et nous ne résoudrons aucun bogue non critique.

La prise en charge du kit Azure IoT service SDK est disponible dans les langages de niveau supérieur ([C#](https://github.com/Azure/azure-iot-sdk-csharp), [Java](https://github.com/Azure/azure-iot-sdk-java), [Node](https://github.com/Azure/azure-iot-sdk-node), [Python](https://github.com/Azure/azure-iot-sdk-python)).

* Téléchargement à partir de [apt-get, MBED, Arduino IDE ou NuGet](https://github.com/Azure/azure-iot-sdk-c/blob/master/readme.md)
* [Code source](https://github.com/Azure/azure-iot-sdk-c)

Kit Azure IoT Hub service SDK pour iOS :

* Installation à partir de [CocoaPod](https://cocoapods.org/pods/AzureIoTHubServiceClient)
* [Exemples](https://github.com/Azure-Samples/azure-iot-samples-ios)

> [!NOTE]
> Consultez les fichiers lisez-moi dans les dépôts GitHub pour plus d’informations sur l’utilisation du langage et des gestionnaires de packages spécifiques à la plateforme pour installer les fichiers binaires et dépendances sur votre ordinateur de développement.

## <a name="microsoft-azure-provisioning-sdks"></a>Kits SDK de provisionnement Microsoft Azure

Les **Kits de développement logiciel (SDK) de provisionnement Microsoft Azure** vous permettent d’ajouter des appareils à votre hub IoT à l’aide du [Service Device Provisioning](../iot-dps/about-iot-dps.md).

Kits de développement logiciel (SDK) de services et d’appareils d’approvisionnement Azure pour C# :

* téléchargement à partir de [Device SDK](https://www.nuget.org/packages/Microsoft.Azure.Devices.Provisioning.Client/) et [Service SDK](https://www.nuget.org/packages/Microsoft.Azure.Devices.Provisioning.Service/) à partir de NuGet.
* [Code source](https://github.com/Azure/azure-iot-sdk-csharp/)
* [Informations de référence sur l'API](/dotnet/api/microsoft.azure.devices.provisioning.client?view=azure-dotnet)

Kits de développement logiciel (SDK) de services et d’appareils d’approvisionnement Azure pour C :

* installation à partir d’[apt-get, MBED, Arduino IDE ou iOS](https://github.com/Azure/azure-iot-sdk-c/blob/master/readme.md#packages-and-libraries)
* [Code source](https://github.com/Azure/azure-iot-sdk-c/blob/master/provisioning_client)
* [Informations de référence sur l'API](/azure/iot-hub/iot-c-sdk-ref/)

Kits de développement logiciel (SDK) de services et d’appareils d’approvisionnement Azure pour Java :

* Ajout au projet [Maven](https://github.com/Azure/azure-iot-sdk-java/blob/master/doc/java-devbox-setup.md#for-the-service-sdk)
* [Code source](https://github.com/Azure/azure-iot-sdk-java/blob/master/provisioning)
* [Informations de référence sur l'API](/java/api/com.microsoft.azure.sdk.iot.provisioning.device?view=azure-java-stable)

Kits de développement logiciel (SDK) de services et d’appareils d’approvisionnement Azure pour Node.js :

* [Code source](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning)
* [Informations de référence sur l'API](/javascript/api/overview/azure/iothubdeviceprovisioning?view=azure-node-latest)
* Téléchargement de [Device SDK](https://badge.fury.io/js/azure-iot-provisioning-device) et [Service SDK](https://badge.fury.io/js/azure-iot-provisioning-service) à partir de npm

Kits de développement logiciel (SDK) de services et d’appareils d’approvisionnement Azure pour Python :

* [Code source](https://github.com/Azure/azure-iot-sdk-python)
* Téléchargement de [Device SDK](https://pypi.org/project/azure-iot-device/) et [Service SDK](https://pypi.org/project/azure-iothub-provisioningserviceclient/) à partir de pip

## <a name="next-steps"></a>Étapes suivantes

Les kits de développement logiciel IoT d’Azure fournissent également un ensemble d’outils pour faciliter le développement :

* [iothub-diagnostic](https://github.com/Azure/iothub-diagnostics) : outil de ligne de commande multiplateforme permettant de diagnostiquer les problèmes liés à la connexion avec IoT Hub.
* [azure-iot-explorer](https://github.com/Azure/azure-iot-explorer) : application de bureau multiplateforme qui se connecte à votre hub IoT pour ajouter/gérer/communiquer avec des appareils IoT.

Documentation pertinente relative au développement à l’aide des kit Azure IoT SDK :

* Découvrez comment [gérer la connectivité et la messagerie fiable](iot-hub-reliability-features-in-sdks.md) à l’aide des kits IoT Hub SDK.
* Découvrez comment [développer pour des plateformes mobiles](iot-hub-how-to-develop-for-mobile-devices.md) telles qu’iOS et Android.
* [Prise en charge des plateformes par le kit de développement logiciel (SDK) Azure IoT](iot-hub-device-sdk-platform-support.md)

Les autres rubriques de référence de ce Guide du développeur IoT Hub comprennent :

* [Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md)
* [Langage de requête IoT Hub pour les jumeaux d’appareil, les travaux et le routage des messages](iot-hub-devguide-query-language.md)
* [Quotas et limitation](iot-hub-devguide-quotas-throttling.md)
* [Prise en charge de MQTT dans IoT Hub](iot-hub-mqtt-support.md)
* [Informations de référence sur l’API REST d’IoT Hub](/rest/api/iothub/)
