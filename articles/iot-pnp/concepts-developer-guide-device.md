---
title: Guide du développeur d’appareils (C) – IoT Plug-and-Play | Microsoft Docs
description: Description d’IoT Plug-and-Play pour les développeurs utilisant des appareils C
author: rido-min
ms.author: rmpablos
ms.date: 11/19/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
zone_pivot_groups: programming-languages-set-twenty-six
ms.openlocfilehash: 339ae9a0a799a0a03f1fcf37dadf2e1ffa724243
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97028680"
---
# <a name="iot-plug-and-play-device-developer-guide"></a>Guide du développeur d’appareils IoT Plug-and-Play

IoT Plug-and-Play permet de créer des smart devices qui publient leurs fonctionnalités sur les applications Azure IoT. Les appareils IoT Plug-and-Play n’ont pas besoin de configuration manuelle quand un client les connecte à des applications compatibles avec IoT Plug-and-Play.

Un smart device peut être implémenté directement. Utilisez les [modules](../iot-hub/iot-hub-devguide-module-twins.md) ou les [modules IoT Edge](../iot-edge/about-iot-edge.md).

Ce guide décrit les étapes de base nécessaires pour créer un appareil, un module ou un module IoT Edge qui suit les [conventions IoT Plug-and-Play](../iot-pnp/concepts-convention.md).

Pour créer un appareil, un module ou un module IoT Edge IoT Plug and Play, procédez comme suit :

1. Vérifiez que votre appareil utilise le protocole MQTT ou MQTT sur WebSockets pour se connecter à Azure IoT Hub.
1. Créez un modèle [DTDL (Digital Twins Definition Language)](https://github.com/Azure/opendigitaltwins-dtdl) pour décrire votre appareil. Pour plus d’informations, consultez [Présentation des composants dans les modèles IoT Plug-and-Play](concepts-components.md).
1. Mettez à jour votre appareil ou module de façon à annoncer `model-id` dans le cadre de la connexion de l’appareil.
1. Implémentez les données de télémétrie, les propriétés et les commandes suivant les [conventions IoT Plug-and-Play](concepts-convention.md).

Une fois que l’implémentation de votre appareil ou module est prête, utilisez [Azure IoT Explorer](howto-use-iot-explorer.md) pour confirmer que l’appareil respecte les conventions IoT Plug-and-Play.

:::zone pivot="programming-language-ansi-c"

[!INCLUDE [iot-pnp-device-devguide-c](../../includes/iot-pnp-device-devguide-c.md)]

:::zone-end

:::zone pivot="programming-language-csharp"

[!INCLUDE [iot-pnp-device-devguide-csharp](../../includes/iot-pnp-device-devguide-csharp.md)]

:::zone-end

:::zone pivot="programming-language-java"

[!INCLUDE [iot-pnp-device-devguide-java](../../includes/iot-pnp-device-devguide-java.md)]

:::zone-end

:::zone pivot="programming-language-javascript"

[!INCLUDE [iot-pnp-device-devguide-node](../../includes/iot-pnp-device-devguide-node.md)]

:::zone-end

:::zone pivot="programming-language-python"

[!INCLUDE [iot-pnp-device-devguide-python](../../includes/iot-pnp-device-devguide-python.md)]

:::zone-end

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous maîtrisez le développement d’appareil IoT Plug-and-Play, voici quelques ressources supplémentaires :

- [Langage DTDL (Digital Twins Definition Language)](https://github.com/Azure/opendigitaltwins-dtdl)
- [SDK d’appareils C](/azure/iot-hub/iot-c-sdk-ref/)
- [API REST IoT](/rest/api/iothub/device)
- [Composants de modèle](concepts-components.md)
- [Installer et utiliser les outils de création DTDL](howto-use-dtdl-authoring-tools.md)
- [Guide du développeur du service IoT Plug-and-Play](concepts-developer-guide-service.md)