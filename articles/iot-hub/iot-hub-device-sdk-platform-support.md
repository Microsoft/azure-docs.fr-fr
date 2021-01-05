---
title: Prise en charge des plateformes par les Kits de développement logiciel (SDK) d’appareils Azure IoT | Microsoft Docs
description: Les kits SDK d’appareils open source sont disponibles sur GitHub en C, .NET (C#), Java, Node.js et Python pour connecter des appareils à Azure IoT Hub et au service DPS de provisionnement d’appareils.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 10/08/2019
ms.author: robinsh
ms.openlocfilehash: 5df36cdb3c02fd4c813625c4869c21581f009b02
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92152215"
---
# <a name="azure-iot-device-sdks-platform-support"></a>Prise en charge des plateformes par les kits Azure IoT device SDK

Microsoft travaille continuellement au développement des appareils Azure IoT Hub. Microsoft publie des kits Device SDK open source sur GitHub pour vous aider à connecter des appareils à Azure IoT Hub et au service Device Provisioning. Les kits Device SDK sont disponibles dans les langages C, .NET (C#), Java, Node.js et Python. Microsoft teste chaque SDK afin de vérifier s’il s’exécute dans l’une des configurations prises en charge qui sont détaillées dans la section [SDK Microsoft et prise en charge des plateformes d’appareils](#microsoft-sdks-and-device-platform-support).

En plus des kits Device SDK, Microsoft propose aux clients et aux développeurs d’autres moyens de connecter leurs appareils à Azure IoT :

* Microsoft collabore avec plusieurs sociétés partenaires afin de les aider à publier des kits de développement basés sur le SDK Azure IoT en C pour leurs plateformes matérielles.

* Microsoft collabore avec des partenaires approuvés Microsoft afin de fournir un ensemble étendu d’appareils qui ont été testés et certifiés pour Azure IoT. Pour obtenir la liste actuelle de ces appareils, consultez le [Catalogue d’appareils Azure Certified pour IoT](https://catalog.azureiotsolutions.com/).

* Microsoft fournit une couche d’abstraction de plateforme dans le kit Azure IoT Hub Device SDK en C, afin d’aider les développeurs à porter facilement le SDK vers leur plateforme. Pour plus d’informations, consultez le [guide de portage pour les SDK en C](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md).

Cette rubrique fournit des informations sur les kits SDK Microsoft et les configurations de plateforme qu’ils prennent en charge, ainsi que des informations sur les autres options listées plus haut.

## <a name="microsoft-sdks-and-device-platform-support"></a>SDK Microsoft et prise en charge des plateformes d’appareils

Microsoft publie des kits SDK open source sur GitHub pour les langages suivants : C, .NET (C#), Node.js, Java et Python. Les kits SDK et leurs dépendances sont listés dans cette section. Les kits SDK sont pris en charge sur toutes les plateformes d’appareils qui sont compatibles avec ces dépendances.

Pour chacun des kits SDK listés, Microsoft :

* Crée et exécute en continu des tests de bout en bout sur la branche master du SDK dans GitHub, et ce, sur différentes plateformes connues.  Pour que les tests couvrent différentes versions du compilateur, nous les effectuons généralement sur la version LTS la plus récente et sur la version la plus utilisée.

* Fournit des instructions d’installation ou des packages d’installation, si nécessaire.

* Prend entièrement en charge les kits SDK sur GitHub avec du code open source, un chemin pour les contributions client et l’engagement de l’équipe produit concernant les problèmes GitHub.

### <a name="c-sdk"></a>Kit de développement logiciel (SDK) C

Le kit [Azure IoT Hub device SDK en C](https://github.com/Azure/azure-iot-sdk-c) a été testé avec les configurations suivantes et les prend donc en charge.

| Système d''exploitation                  | Bibliothèque TLS                  | Autres conditions requises                                                                     |
|---------------------|------------------------------|---------------------------------------------------------------------------------------------|
| Linux               | OpenSSL, WolfSSL ou BearSSL | Sockets Berkeley</br></br>POSIX (Portable Operating System Interface)                       |
| iOS 12.2            | OpenSSL                      | XCode émulé dans OSX 10.13.4                                                               |
| Famille Windows 10   | SChannel                     |                                                                                             |
| Mbed OS 5.4         | Mbed TLS 2                   | [DevKit IoT MXChip](https://microsoft.github.io/azure-iot-developer-kit/)                  |
| Système d’exploitation Azure Sphere     | WolfSSL                      | [Azure Sphere MT3620](https://azure.microsoft.com/services/azure-sphere/get-started/) |
| Arduino             | BearSSL                      | [ESP32 ou ESP8266](https://github.com/Azure/azure-iot-arduino#simple-sample-instructions) 

### <a name="python-sdk"></a>Kit de développement logiciel (SDK) Python

Le kit [Azure IoT Hub device SDK en Python](https://github.com/Azure/azure-iot-sdk-python) a été testé avec les configurations suivantes et les prend donc en charge.

| Système d''exploitation                  | Compilateur                          |
|---------------------|-----------------------------------|
| Linux               | Python 2.7.*, 3.5 ou version ultérieure |
| macOS High Sierra   | Python 2.7.*, 3.5 ou version ultérieure |
| Famille Windows 10   | Python 2.7.*, 3.5 ou version ultérieure |

Seul Python version 3.5.3 ou ultérieure prend en charge les API asynchrones, nous vous recommandons d’utiliser la version 3.7 ou une version ultérieure.

### <a name="net-sdk"></a>Kit de développement logiciel (SDK) .NET

Le kit [Azure IoT Hub device SDK en .NET (C#)](https://github.com/Azure/azure-iot-sdk-csharp) a été testé avec les configurations suivantes et les prend donc en charge.

| Système d''exploitation                                   | standard                                                   |
|--------------------------------------|------------------------------------------------------------|
| Linux                                | .NET Core 2.1                                              |
| Références SKU Windows 10 Desktop et Server   | .NET Core 2.1, .NET Framework 4.5.1 ou .NET Framework 4.7 |

Le kit de développement logiciel (SDK) .NET peut également être utilisé avec l’[agent Smart Device Azure](https://github.com/ms-iot/azure-client-tools/blob/master/docs/device-agent/device-agent.md) ou [un service NT personnalisé qui peut utiliser RPC pour communiquer avec les applications UWP](/samples/microsoft/windows-iotcore-samples/ntservice-rpc/).

### <a name="nodejs-sdk"></a>Kit de développement logiciel (SDK) Node.js

Le kit [Azure IoT Hub device SDK en Node.js](https://github.com/Azure/azure-iot-sdk-node) a été testé avec les configurations suivantes et les prend donc en charge.

| Système d''exploitation                  | Version de nœud    |
|---------------------|-----------------|
| Linux               | LTS et actuel |
| Famille Windows 10   | LTS et actuel |

### <a name="java-sdk"></a>Kit de développement logiciel (SDK) Java

Le kit [Azure IoT Hub device SDK en Java](https://github.com/Azure/azure-iot-sdk-java) a été testé avec les configurations suivantes et les prend donc en charge.

| Système d''exploitation                     | Version de Java |
|------------------------|--------------|
| API Android 28         | Java 8       |
| Linux x64             | Java 8       |
| Famille Windows 10 x64  | Java 8       |

## <a name="partner-supported-development-kits"></a>Kits de développement pris en charge par les partenaires

Microsoft travaille avec différents partenaires en vue de fournir des kits de développement pour plusieurs architectures de microprocesseur. Ces partenaires ont effectué le portage du SDK Azure IoT en C vers leur plateforme. Les partenaires sont chargés de créer la couche d’abstraction de plateforme du SDK et d’effectuer sa maintenance. Microsoft collabore avec les partenaires pour fournir une prise en charge étendue.

| Partenaire             | Appareils                            | Lien                     | Support |
|---------------------|------------------------------------|--------------------------|---------|
| Espressif           | ESP32 <br/> ESP8266                              | [Esp-azure](https://github.com/espressif/esp-azure)                | [GitHub](https://github.com/espressif/esp-azure)  
| Qualcomm            | Modem LTE IoT Qualcomm MDM9206     | [LTE Qualcomm pour le Kit de développement logiciel (SDK) IoT](https://developer.qualcomm.com/software/lte-iot-sdk) | [Forum](https://developer.qualcomm.com/forums/software/lte-iot-sdk)   |
| ST Microelectronics | Série STM32L4 <br/> Série STM32F4 <br/>  Série STM32F7 <br/>  Kit de découverte STM32L4 pour nœud IoT    | [X-CUBE-AZURE](https://www.st.com/en/embedded-software/x-cube-azure.html) <br/>  <br/> [P-NUCLEO-AZURE](https://www.st.com/content/st_com/en/products/evaluation-tools/solution-evaluation-tools/communication-and-connectivity-solution-eval-boards/p-nucleo-azure1.html) <br/> [FP-CLD-AZURE](https://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32-ode-function-pack-sw/fp-cld-azure1.html)            | [Support](https://www.st.com/content/st_com/en/support/support-home.html)
| Texas Instruments   | CC3220SF Launchpad </br> CC3220S Launchpad </br> CC3235SF Launchpad </br> CC3235S LaunchPad </br> MSP432E4 Launchpad | [Plug-in Azure IoT pour SimpleLink](https://github.com/TexasInstruments/azure-iot-pal-simplelink) | [Forum TI E2E](https://e2e.ti.com) <br/> [Forum TI E2E pour CC3220](https://e2e.ti.com/support/wireless_connectivity/simplelink_wifi_cc31xx_cc32xx/) <br/> [Forum TI E2E pour MSP432E4](https://e2e.ti.com/support/microcontrollers/msp430/) |

## <a name="porting-the-microsoft-azure-iot-c-sdk"></a>Portage du SDK Microsoft Azure IoT en C

Si la plateforme de votre appareil n’est pas mentionnée dans l’une des sections précédentes, vous pouvez effectuer le portage du SDK Azure IoT en C. Le portage du SDK en C implique principalement l’implémentation de la couche d’abstraction de plateforme du SDK. La couche d’abstraction de plateforme définit des primitives qui « lient » votre appareil aux fonctions de niveau supérieur du SDK. Pour plus d’informations, consultez [Instructions de portage](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md).

## <a name="microsoft-partners-and-certified-azure-iot-devices"></a>Partenaires Microsoft et appareils Azure Certified pour IoT

Microsoft collabore avec différents partenaires en vue d’étendre le nombre d’appareils testés et certifiés pour Azure IoT.

* Pour découvrir les appareils Azure Certified pour IoT, consultez le [Catalogue d’appareils Azure Certified pour IoT](https://catalog.azureiotsolutions.com/).

* Pour en savoir plus sur l’écosystème Azure Certified pour IoT, consultez [Rejoindre l’écosystème Certified pour IoT](https://catalog.azureiotsolutions.com/register).

## <a name="connecting-to-iot-hub-without-an-sdk"></a>Connexion à IoT Hub sans SDK

Si vous n’êtes pas en mesure d’utiliser l’un des kits Azure IoT device SDK, vous pouvez vous connecter directement à IoT Hub à l’aide d’un protocole de communication pris en charge. Pour plus d’informations, consultez [Développer sans Kit de développement logiciel (SDK) Azure IoT](iot-hub-devguide-no-sdk.md).

## <a name="support-and-other-resources"></a>Prise en charge et autres ressources

Si vous rencontrez des problèmes lors de l’utilisation des kits Azure IoT device SDK, vous pouvez obtenir de l’aide de différentes façons, résumées ci-dessous. Pour obtenir des informations complètes sur toutes vos options de support, consultez [Options d’aide et de support Azure IoT](../iot-fundamentals/iot-support-help.md). 

**Signaler des bogues** : les bogues des kits Device SDK peuvent être signalés dans la page des problèmes du projet GitHub concerné. Les correctifs apportés au projet sont rapidement intégrés aux mises à jour du produit.

* [Problèmes concernant le SDK Azure IoT Hub C](https://github.com/Azure/azure-iot-sdk-c/issues)

* [Problèmes concernant le SDK Azure IoT Hub .NET (C#)](https://github.com/Azure/azure-iot-sdk-csharp/issues)

* [Problèmes concernant le SDK Azure IoT Hub Java](https://github.com/Azure/azure-iot-sdk-java/issues)

* [Problèmes concernant le SDK Azure IoT Hub Node.js](https://github.com/Azure/azure-iot-sdk-node/issues)

* [Problèmes concernant le SDK Azure IoT Hub Python](https://github.com/Azure/azure-iot-sdk-python/issues)

**Questions techniques** : vous pouvez poser des questions techniques à la fois sur [Microsoft Q&A](/answers/topics/azure-iot-sdk.html) et [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-iot-sdk) à l’aide de la balise *azure-iot-sdk*.

**Équipe de support technique Microsoft** : les utilisateurs qui ont un [plan de support](https://azure.microsoft.com/support/plans/) peuvent solliciter l’équipe de support technique Microsoft en créant une demande de support directement à partir du [portail Azure](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

**Demandes de fonctionnalités** : vous pouvez suivre vos demandes de fonctionnalités Azure IoT dans la [page User Voice](https://feedback.azure.com/forums/321918-azure-iot) du produit concerné.

## <a name="next-steps"></a>Étapes suivantes

* [Kits de développement logiciel (SDK) de services et d’appareils](iot-hub-devguide-sdks.md)
* [Guide de portage](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md)