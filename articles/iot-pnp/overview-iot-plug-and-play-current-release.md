---
title: Version actuelle de IoT Plug-and-Play | Microsoft Docs
description: Découvrez ce qui est inclus dans la version actuelle de IoT Plug-and-Play.
author: dominicbetts
ms.author: dobett
ms.date: 10/01/2020
ms.topic: overview
ms.custom: mvc
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 5e9d9e762568c01d31f0c3a29be809a5f90b6b44
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421649"
---
# <a name="what-is-in-the-current-iot-plug-and-play-release"></a>Contenu de la version actuelle de IoT Plug-and-Play

Cet article récapitule les outils, kits de développement logiciel (SDK) et API qui prennent en charge la version actuelle de IoT Plug-and-Play. Les numéros de version indiqués reflètent le numéro de version au moment où IoT Plug-and-Play est mis à la disposition générale. Les numéros de version peuvent augmenter par incréments après la sortie de la version actuelle.

## <a name="modeling-language"></a>Langage de modélisation

[Langage DTDL (Digital Twins Definition Language) v2](https://github.com/Azure/opendigitaltwins-dtdl).

Pour en savoir plus sur le fonctionnement des appareils IoT Plug-and-Play avec DTDL, consultez [Conventions IoT Plug and Play](concepts-convention.md).

## <a name="tools-and-utilities"></a>Outils et utilitaires

- Explorateur Azure IoT 0.12.0.

    Pour plus d’informations, consultez [Installer et utiliser l’explorateur Azure IoT](howto-use-iot-explorer.md).

- Extension VS Code 1.0.0.

    Pour plus d’informations, consultez [Installer et utiliser les outils de création DTDL](howto-use-dtdl-authoring-tools.md).

- Extension Visual Studio 2019 1.0.0.

    Pour plus d’informations, consultez [Installer et utiliser les outils de création DTDL](howto-use-dtdl-authoring-tools.md).

- Azure IoT Azure CLI 0.10.0.

    L’extension Azure IoT comprend des commandes pour faciliter la certification des appareils. Consultez `az iot product -h`.

## <a name="libraries-and-sdks"></a>Bibliothèques et kits SDK

Pour en savoir plus sur les bibliothèques et les Kits de développement logiciel (SDK), consultez [Kits de développement logiciel (SDK) Microsoft pour IoT Plug-and-Play](libraries-sdks.md).

- SDK d’appareil C [vcpkg 1.3.9](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/setting_up_vcpkg.md)
- SDK d’appareil Embedded C [GitHub](https://github.com/Azure/azure-sdk-for-c/)
- SDK d’appareil .NET [NuGet 1.31.0](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client)
- SDK d’appareil Java [Maven 1.26.0](https://mvnrepository.com/artifact/com.microsoft.azure.sdk.iot/iot-device-client)
- SDK d’appareil Python [pip 2.3.0](https://pypi.org/project/azure-iot-device/)
- SDK d’appareil Node.js [npm 1.17.2](https://www.npmjs.com/package/azure-iot-device)
- .NET - Service IoT Hub [NuGet 1.27.1](https://www.nuget.org/packages/Microsoft.Azure.Devices )
- Java - Service IoT Hub[Maven 1.26.0](https://mvnrepository.com/artifact/com.microsoft.azure.sdk.iot/iot-service-client/1.26.0)
- Node.js - Service IoT Hub [npm 1.13.0](https://www.npmjs.com/package/azure-iothub)
- Python - Service IoT Hub/Digital Twins [pip 2.2.3](https://pypi.org/project/azure-iot-hub)
- Analyseur de modèle DTDL [NuGet](https://www.nuget.org/packages/Microsoft.Azure.DigitalTwins.Parser).

## <a name="rest-apis"></a>API REST

API REST [2020-09-30](/rest/api/iothub).

Pour plus d’informations, consultez [Guide du développeur IoT Plug-and-Play](concepts-developer-guide-service.md).

## <a name="iot-hub"></a>IoT Hub

IoT Plug-and-Play est pris en charge par IoT Hub dans toutes les régions. IoT Plug-and-Play n’est pris en charge que par les hubs IoT standard ou gratuits.

## <a name="announcements"></a>Annonces

Pour obtenir les annonces actuelles et précédentes sur IoT Plug-and-Play, consultez les billets de blog suivants :

- [Actualisation de la préversion publique (publiée le 29 août 2020)](https://techcommunity.microsoft.com/t5/internet-of-things/add-quot-plug-and-play-quot-to-your-iot-solutions/ba-p/1548531)
- [Préparez et certifiez vos appareils pour IoT Plug-and-Play (publié le 26 août 2020)](https://azure.microsoft.com/blog/prepare-and-certify-your-devices-for-iot-plug-and-play/)
- [IoT Plug-and-Play est maintenant disponible en préversion (publié le 22 août 2019)](https://azure.microsoft.com/blog/iot-plug-and-play-is-now-available-in-preview/)
- [Créer avec Azure IoT Central et IoT Plug-and-Play (publié le 7 mai 2019)](https://azure.microsoft.com/blog/build-with-azure-iot-central-and-iot-plug-and-play/)