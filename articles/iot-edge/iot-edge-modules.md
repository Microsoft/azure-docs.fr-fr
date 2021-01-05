---
title: Découvrez l’exécution de logique par les modules sur vos appareils - Azure IoT Edge | Microsoft Docs
description: Les modules Azure IoT Edge sont des unités de logique en conteneur qui peuvent être déployées et gérées à distance afin de vous permettre d’exécuter une logique métier sur les appareils IoT Edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 03/21/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: devx-track-csharp
ms.openlocfilehash: 9feef9ec04bcc44bc09d55202b72ada4adc94cf5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89017054"
---
# <a name="understand-azure-iot-edge-modules"></a>Présentation des modules Azure IoT Edge

Azure IoT Edge vous permet de déployer et de gérer la logique métier sur la périphérie sous la forme de *modules*. Les modules Azure IoT Edge sont la plus petite unité de calcul gérée par IoT Edge. Ils peuvent contenir des services Azure (par exemple Azure Stream Analytics) ou votre propre code spécifique à la solution. Pour comprendre comment les modules sont développés, déployés et gérés, considérez les quatre éléments conceptuels d’un module :

* Une **image de module** est un package contenant le logiciel qui définit un module.
* Une **instance de module** est l’unité de calcul spécifique exécutant l’image de module sur un appareil IoT Edge. L’instance de module est démarrée par le runtime IoT Edge.
* Une **identité de module** est une information (notamment des informations d’identification de sécurité) stockée dans IoT Hub et associée à chaque instance de module.
* Un **jumeau de module** est un document JSON stocké dans IoT Hub qui contient les informations d’état relatives à une instance de module, notamment des métadonnées, des configurations et des conditions.

## <a name="module-images-and-instances"></a>Instances et images de modules

Les images de modules IoT Edge contiennent des applications qui tirent parti de la gestion, de la sécurité et des fonctionnalités de communication du runtime IoT Edge. Vous pouvez développer vos propres images de modules ou en exporter une à partir d’un service Azure pris en charge, tel qu’Azure Stream Analytics.
Les images existent dans le cloud et peuvent être mises à jour, modifiées et déployées dans différentes solutions. Par exemple, un module qui utilise l’apprentissage automatique pour prédire la sortie d’une ligne de production existe en tant qu’image distincte d’un module qui utilise la vision informatique pour contrôler un drone.

Chaque fois qu’une image de module est déployée sur un appareil et démarrée par le runtime IoT Edge, une nouvelle instance de ce module est créée. Deux appareils dans différentes parties du monde peuvent utiliser la même image de module. Toutefois, chaque appareil a sa propre instance de module lorsque le module est démarré sur l’appareil.

![Diagramme - Images de modules dans le cloud, instances de modules sur les appareils](./media/iot-edge-modules/image_instance.png)

Dans l’implémentation, les images de modules existent en tant qu’images de conteneurs dans un référentiel tandis que les instances de modules sont des conteneurs sur des appareils.

<!--
As use cases for Azure IoT Edge grow, new types of module images and instances will be created. For example, resource constrained devices cannot run containers so may require module images that exist as dynamic link libraries and instances that are executables. 
-->

## <a name="module-identities"></a>Identités de modules

Quand une nouvelle instance de module est créée par le runtime IoT Edge, elle est associée à une identité de module correspondante. L’identité de module est stockée dans IoT Hub et utilisée comme étendue d’adressage et de sécurité pour toutes les communications cloud et locales pour cette instance de module.

L’identité associée à une instance de module dépend de l’identité de l’appareil sur lequel l’instance s’exécute et du nom que vous donnez à ce module dans votre solution. Par exemple, si vous nommez `insight` un module qui utilise Azure Stream Analytics et que vous le déployez sur un appareil nommé `Hannover01`, le runtime IoT Edge crée une identité de module correspondante nommée `/devices/Hannover01/modules/insight`.

Il est clair que dans les scénarios où vous avez besoin de déployer une image de module plusieurs fois sur le même appareil, vous pouvez déployer la même image plusieurs fois avec des noms différents.

![Diagramme - Les identités de modules sont uniques sur les appareils et entre les appareils](./media/iot-edge-modules/identity.png)

## <a name="module-twins"></a>Jumeaux de module

Chaque instance de module a aussi un jumeau de module correspondant que vous pouvez utiliser pour configurer l’instance de module. L’instance et le jumeau sont associés par le biais de l’identité de module.

Un jumeau de module est un document JSON qui stocke des propriétés de configuration et des informations sur un module. Ce concept s’apparente au concept de [jumeau d’appareil](../iot-hub/iot-hub-devguide-device-twins.md) dans IoT Hub. La structure d’un jumeau de module est identique à celle d’un jumeau d’appareil. Les API utilisées pour interagir avec les deux types de jumeaux sont identiques. La seule différence entre les deux est l’identité utilisée pour instancier le SDK client.

```csharp
// Create a ModuleClient object. This ModuleClient will act on behalf of a
// module since it is created with a module’s connection string instead
// of a device connection string.
ModuleClient client = new ModuleClient.CreateFromEnvironmentAsync(settings);
await client.OpenAsync();

// Get the module twin
Twin twin = await client.GetTwinAsync();
```

## <a name="offline-capabilities"></a>Fonctionnalités hors ligne

Les modules Azure IoT Edge peuvent fonctionner en mode hors connexion indéfiniment après la première synchronisation avec IoT Hub. Les appareils IoT Edge peuvent également étendre cette fonctionnalité hors connexion avec d’autres appareils IoT. Pour plus d’informations, consultez [Introduction aux fonctionnalités hors connexion étendues pour les appareils, modules et appareils enfants IoT Edge](offline-capabilities.md).

## <a name="next-steps"></a>Étapes suivantes

* [Comprendre les conditions requises et les outils de développement de modules IoT Edge](module-development.md)
* [Présentation du runtime Azure IoT Edge et de son architecture](iot-edge-runtime.md)
