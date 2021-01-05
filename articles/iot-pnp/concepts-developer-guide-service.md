---
title: Guide du développeur du service - IoT Plug-and-Play | Microsoft Docs
description: Description d’IoT Plug-and-Play pour les développeurs de service
author: dominicbetts
ms.author: dobett
ms.date: 10/01/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
zone_pivot_groups: programming-languages-set-ten
ms.openlocfilehash: df913716ff34a61e5bde4c0771ea8b7599db3d30
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92521374"
---
# <a name="iot-plug-and-play-service-developer-guide"></a>Guide du développeur du service IoT Plug-and-Play

IoT Plug-and-Play permet de créer des smart devices qui publient leurs fonctionnalités sur les applications Azure IoT. Les appareils IoT Plug-and-Play n’ont pas besoin de configuration manuelle quand un client les connecte à des applications compatibles avec IoT Plug-and-Play.

IoT Plug-and-Play vous permet d’utiliser les appareils qui ont annoncé leur ID de modèle auprès de votre hub IoT. Par exemple, vous pouvez accéder directement aux propriétés et aux commandes d’un appareil.

Pour utiliser un appareil IoT Plug-and-Play connecté à votre hub IoT, utilisez l’un des kits de développement logiciel (SDK) du service IoT :

## <a name="service-sdks"></a>Kits de développement logiciel (SDK) IoT Service

Utilisez les kits de développement logiciel (SDK) de service Azure IoT dans votre solution pour interagir avec les appareils et les modules. Par exemple, vous pouvez utiliser les kits de développement logiciel (SDK) de service pour lire et mettre à jour les propriétés de jumeau numérique et appeler des commandes. Les langages pris en charge incluent C# , Java, Node.js et Python.

Les kits de développement logiciel (SDK) de service vous permettent d’accéder aux informations des appareils à partir d’une solution, telle qu’une application de bureau ou web. Les kits de développement logiciel (SDK) de service incluent deux espaces de noms et des modèles d’objet que vous pouvez utiliser pour récupérer l’ID de modèle :

- Client du service IoT Hub. Ce service expose l’ID de modèle en tant que propriété de jumeau d’appareil.

- Client Digital Twins. La nouvelle API Digital Twins opère sur des constructions de modèle en [langage DTDL (Digital Twins Definition Language)](concepts-digital-twin.md), telles que les composants, les propriétés et les commandes. Les API de jumeaux numériques permettent aux développeurs de solutions de créer plus facilement des solutions IoT Plug-and-Play.

:::zone pivot="programming-language-csharp"

[!INCLUDE [iot-pnp-service-devguide-csharp](../../includes/iot-pnp-service-devguide-csharp.md)]

:::zone-end

:::zone pivot="programming-language-java"

[!INCLUDE [iot-pnp-service-devguide-java](../../includes/iot-pnp-service-devguide-java.md)]

:::zone-end

:::zone pivot="programming-language-javascript"

[!INCLUDE [iot-pnp-service-devguide-node](../../includes/iot-pnp-service-devguide-node.md)]

:::zone-end

:::zone pivot="programming-language-python"

[!INCLUDE [iot-pnp-service-devguide-python](../../includes/iot-pnp-service-devguide-python.md)]

:::zone-end

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert la modélisation d’appareil, voici quelques ressources supplémentaires :

- [Langage DTDL (Digital Twins Definition Language)](https://github.com/Azure/opendigitaltwins-dtdl)
- [SDK d’appareils C](/azure/iot-hub/iot-c-sdk-ref/)
- [API REST IoT](/rest/api/iothub/device)
- [Composants de modèle](./concepts-components.md)
- [Installer et utiliser les outils de création DTDL](howto-use-dtdl-authoring-tools.md)