---
title: Azure IoT Edge et Azure IoT Central | Microsoft Docs
description: Découvrez comment utiliser Azure IoT Edge avec une application IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 12/19/2020
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.custom:
- device-developer
- iot-edge
ms.openlocfilehash: 9a7c886ba4dd6e7ab4bd62700f5437855a16a5ad
ms.sourcegitcommit: ab829133ee7f024f9364cd731e9b14edbe96b496
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/28/2020
ms.locfileid: "97796565"
---
# <a name="connect-azure-iot-edge-devices-to-an-azure-iot-central-application"></a>Connecter des appareils Azure IoT Edge à une application Azure IoT Central

*Cet article s’applique aux créateurs de solutions et aux développeurs d’appareils.*

IoT Edge est constitué de trois composants :

* Les **modules IoT Edge** sont des conteneurs qui exécutent les services Azure, les services partenaire ou votre propre code. Ils sont déployés sur des appareils IoT Edge et s’exécutent localement sur ces appareils.
* Le **runtime IoT Edge** s’exécute sur chaque appareil IoT Edge et gère les modules déployés sur chaque appareil.
* Une **interface basée sur le cloud** permet de superviser et de gérer des appareils IoT Edge à distance. IoT Central est l’interface cloud.

Un appareil **Azure IoT Edge** peut être un appareil de passerelle avec, en aval, des appareils qui se connectent à l’appareil IoT Edge. Cet article partage plus d’informations sur les modèles de connectivité des appareils en aval.

Un **modèle d’appareil** définit les fonctionnalités de votre appareil et de vos modules IoT Edge. Parmi ces fonctionnalités figurent l’envoi des données de télémétrie par le module, les propriétés de module et les commandes auxquelles un module répond.

## <a name="downstream-device-relationships-with-a-gateway-and-modules"></a>Relations des appareils en aval avec une passerelle et des modules

Les appareils en aval peuvent se connecter à un appareil de passerelle IoT Edge via le module `$edgeHub`. Cet appareil IoT Edge devient une passerelle transparente dans ce scénario.

![Diagramme de la passerelle transparente](./media/concepts-iot-edge/gateway-transparent.png)

Les appareils en aval peuvent également se connecter à un appareil de passerelle IoT Edge via un module personnalisé. Dans le scénario suivant, les appareils en aval se connectent via un module personnalisé Modbus.

![Diagramme de la connexion via un module personnalisé](./media/concepts-iot-edge/gateway-module.png)

Le diagramme suivant illustre la connexion à un appareil de passerelle IoT Edge via les deux types de modules (personnalisé et `$edgeHub`).  

![Diagramme de connexion via les deux modules de connexion](./media/concepts-iot-edge/gateway-module-transparent.png)

Enfin, les appareils en aval peuvent se connecter à un appareil de passerelle IoT Edge via plusieurs modules personnalisés. Le diagramme suivant montre des appareils en aval qui se connectent via un module personnalisé Modbus, un module personnalisé BLE et le module `$edgeHub`. 

![Diagramme de connexion par le biais de plusieurs modules personnalisés](./media/concepts-iot-edge/gateway-module2-transparent.png)

## <a name="deployment-manifests-and-device-templates"></a>Manifestes de déploiement et modèles d’appareil

Dans IoT Edge, vous pouvez déployer et gérer la logique métier sous forme de modules. Les modules IoT Edge sont la plus petite unité de calcul gérée par IoT Edge. Ils peuvent contenir des services Azure (par exemple Azure Stream Analytics) ou du code propre à votre solution. Pour comprendre comment les modules sont développés, déployés et gérés, consultez [Modules IoT Edge](../../iot-edge/iot-edge-modules.md).

À un niveau élevé, un manifeste de déploiement est une liste des jumeaux de module configurés avec leurs propriétés souhaitées. Un manifeste de déploiement indique à un appareil IoT Edge (ou à un groupe d’appareils) les modules à installer et comment les configurer. Les manifestes de déploiement incluent les propriétés souhaitées pour chaque jumeau de module. Les appareils IoT Edge rapportent les propriétés indiquées pour chaque module.

Utilisez Visual Studio Code pour créer un manifeste de déploiement. Pour en savoir plus, consultez [Azure IoT Edge pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge).

Dans Azure IoT Central, vous pouvez importer un manifeste de déploiement pour créer un modèle d’appareil. L’organigramme suivant montre le cycle de vie d’un manifeste de déploiement dans IoT Central.

![Organigramme du cycle de vie d’un manifeste de déploiement](./media/concepts-iot-edge/dmflow.png)

IoT Central modélise ainsi un appareil IoT Edge :

* À chaque modèle d’appareil IoT Edge correspond un modèle d’appareil.
* Un modèle de capacité de module est généré pour chaque module personnalisé listé dans le manifeste de déploiement.
* Une relation est établie entre chaque modèle de capacité de module et un modèle d’appareil.
* Un modèle de capacité de module implémente des interfaces de module.
* Chaque interface de module contient des données de télémétrie, des propriétés et des commandes.

![Diagramme de la modélisation d’IoT Edge](./media/concepts-iot-edge/edgemodelling.png)

## <a name="iot-edge-gateway-devices"></a>Appareils de passerelle IoT Edge

Si vous avez sélectionné un appareil IoT Edge comme appareil de passerelle, vous pouvez ajouter des relations en aval aux modèles d’appareil pour les appareils que vous souhaitez connecter à l’appareil de passerelle.

## <a name="next-steps"></a>Étapes suivantes

Si vous êtes un développeur d’appareils, l’étape suivante suggérée consiste à en apprendre plus sur les [types d’appareils de passerelle dans IoT Central](./tutorial-define-gateway-device-type.md).
