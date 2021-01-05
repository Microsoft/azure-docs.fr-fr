---
title: Utiliser IoT Plug-and-Play sur des appareils limités | Microsoft Docs
description: Découvrez comment vous pouvez implémenter IoT Plug-and-Play sur des appareils limités à l’aide du kit de développement logiciel (SDK) pour Embedded C ou Azure RTOS.
author: elhorton
ms.author: elhorton
ms.date: 09/23/2020
ms.topic: how-to
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: d61ca10612a0935f8483745d164835d7498280c0
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92042811"
---
# <a name="implement-iot-plug-and-play-on-constrained-devices"></a>Implémenter IoT Plug-and-Play sur des appareils limités

Si vous développez pour des *appareils limités*, vous pouvez utiliser IoT Plug-and-Play avec le [kit de développement logiciel (SDK) Azure pour les bibliothèques clientes Embedded C](https://aka.ms/embeddedcsdk) ou avec [Azure RTOS](/azure/rtos/overview-rtos). Cet article contient des liens et des ressources pour ces scénarios limités.

## <a name="use-the-sdk-for-embedded-c"></a>Utiliser le kit de développement logiciel (SDK) pour Embedded C

Le kit de développement logiciel (SDK) pour Embedded C offre une solution légère pour connecter des appareils limités aux services Azure IoT, y compris à l’aide des [conventions d’IoT Plug-and-Play](concepts-convention.md). Les liens suivants incluent des exemples d’appareil réel et à des fins éducatives et de débogage.

### <a name="use-a-real-device"></a>Utiliser un appareil réel

Pour obtenir un didacticiel complet de bout en bout sur l’utilisation du kit de développement logiciel (SDK) pour Embedded C, le service Device Provisioning et IoT Plug-and-Play sur un appareil réel, consultez [Reprogrammer PIC-IoT WX Development Board pour se connecter à Azure via le service IoT Hub Device Provisioning](https://github.com/Azure-Samples/Microchip-PIC-IoT-Wx).

### <a name="introductory-samples"></a>Exemple d’introduction

Le référentiel du kit de développement logiciel (SDK) pour Embedded C contient [plusieurs exemples](https://github.com/Azure/azure-sdk-for-c/tree/master/sdk/samples/iot#iot-hub-plug-and-play-sample) qui vous montrent comment utiliser IoT Plug-and-Play :

> [!NOTE]
> L’exécution de ces exemples est présentée sur Windows et Linux à des fins éducatives et de débogage. Dans un scénario de production, les exemples sont destinés aux appareils limités uniquement.

- [Exemple de thermostat avec le kit de développement logiciel (SDK) pour Embedded C](https://github.com/Azure/azure-sdk-for-c/blob/master/sdk/samples/iot/paho_iot_hub_pnp_sample.c)

- [Exemple de contrôleur de température avec le kit de développement logiciel (SDK) pour Embedded C](https://github.com/Azure/azure-sdk-for-c/blob/master/sdk/samples/iot/paho_iot_hub_pnp_component_sample.c)

## <a name="using-azure-rtos"></a>Utilisation d’Azure RTOS

Azure RTOS inclut une couche légère qui ajoute une connectivité native aux services cloud Azure IoT. Cette couche offre un mécanisme simple pour connecter des appareils limités à Azure IoT tout en utilisant les fonctionnalités avancées d’Azure RTOS. Pour en savoir plus, consultez [Qu’est-ce que Microsoft Azure RTOS](/azure/rtos/overview-rtos).

### <a name="toolchains"></a>Chaînes d’outils

Les exemples d’Azure RTOS sont proposés avec différents types de combinaisons d’IDE et de chaîne d’outils, par exemple :

- IAR : IDE [Embedded Workbench](https://www.iar.com/iar-embedded-workbench/) d’IAR
- GCC/CMake : Créer sur le système de build [CMake](https://cmake.org/) open source la [chaîne d’outils GNU Arm Embedded](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm).
- MCUExpresso : [IDE MCUXpresso](https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/mcuxpresso-integrated-development-environment-ide:MCUXpresso-IDE) de NXP
- STM32Cube : [IDE STM32Cube](https://www.st.com/en/development-tools/stm32cubeide.html) de STMicroelectronic
- MPLAB : [IDE MPLAB X](https://www.microchip.com/mplab/mplab-x-ide) de Microchip

### <a name="samples"></a>Exemples

Pour obtenir des exemples qui vous montrent comment démarrer sur différents appareils avec Azure RTOS et IoT Plug-and-Play, consultez le tableau suivant :

Fabricant | Appareil | Exemples |
| --- | --- | --- |
| Microchip | [ATSAME54-XPRO](https://www.microchip.com/developmenttools/productdetails/atsame54-xpro) | [GCC/CMake](https://github.com/azure-rtos/getting-started/tree/master/Microchip/ATSAME54-XPRO) • [IAR](https://aka.ms/azrtos-sample/e54-iar) • [MPLAB](https://aka.ms/azrtos-sample/e54-mplab)
| MXCHIP | [AZ3166](https://aka.ms/iot-devkit) | [GCC/CMake](https://github.com/azure-rtos/getting-started/tree/master/MXChip/AZ3166)
| NXP | [MIMXRT1060-EVK](https://www.nxp.com/design/development-boards/i-mx-evaluation-and-development-boards/mimxrt1060-evk-i-mx-rt1060-evaluation-kit:MIMXRT1060-EVK) | [GCC/CMake](https://github.com/azure-rtos/getting-started/tree/master/NXP/MIMXRT1060-EVK) • [IAR](https://aka.ms/azrtos-sample/rt1060-iar) • [MCUXpresso](https://aka.ms/azrtos-sample/rt1060-mcuxpresso)
| STMicroelectronics | [32F746GDISCOVERY](https://www.st.com/en/evaluation-tools/32f746gdiscovery.html) | [IAR](https://aka.ms/azrtos-sample/f746g-iar) • [STM32Cube](https://aka.ms/azrtos-sample/f746g-cubeide)
| STMicroelectronics | [B-L475E-IOT01](https://www.st.com/en/evaluation-tools/b-l475e-iot01a.html) | [GCC/CMake](https://github.com/azure-rtos/getting-started/tree/master/STMicroelectronics/STM32L4_L4%2B) • [IAR](https://aka.ms/azrtos-sample/l4s5-iar) • [STM32Cube](https://aka.ms/azrtos-sample/l4s5-cubeide)
| STMicroelectronics | [B-L4S5I-IOT01](https://www.st.com/en/evaluation-tools/b-l4s5i-iot01a.html) | [GCC/CMake](https://github.com/azure-rtos/getting-started/tree/master/STMicroelectronics/STM32L4_L4%2B) • [IAR](https://aka.ms/azrtos-sample/l4s5-iar) • [STM32Cube](https://aka.ms/azrtos-sample/l4s5-cubeide)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert les options d’implémentation d’IoT Plug-and-Play sur des appareils limités, l’étape suivante suggérée consiste à en savoir plus sur les [conventions d’IoT Plug-and-Play](concepts-convention.md).