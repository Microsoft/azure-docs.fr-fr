---
title: Architecture de référence pour une solution de gestion des déchets créée avec Azure IoT Central | Microsoft Docs
description: Découvrez les concepts qui se rapportent à une solution de gestion des déchets conçue avec Azure IoT Central.
author: miriambrus
ms.author: miriamb
ms.date: 12/11/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 94098a1a2a9ac1b4889ad165a65c8b9d50d734dd
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97586803"
---
# <a name="connected-waste-monitoring-reference-architecture"></a>Architecture de référence d’une solution connectée de suivi des déchets 

Vous pouvez créer des solutions de gestion des déchets connectées à l’aide du **modèle d’application Azure IoT Central** en tant qu’application IoT de démarrage. Cet article fournit des conseils d’ordre général sur l’architecture de référence d’une solution de bout en bout. 

![Architecture de gestion des déchets connectée](./media/concepts-connectedwastemanagement-architecture/concepts-connectedwastemanagement-architecture1.png)


Concepts :

1. Appareils et connectivité  
1. IoT Central 
1. Extensibilité et intégrations
1. Applications métier

Intéressons-nous aux composants clés qui sont généralement impliqués dans une solution de suivi de la consommation d’eau.

## <a name="devices-and-connectivity"></a>Appareils et connectivité 
Les appareils utilisés dans les environnements ouverts, comme les conteneurs de déchets, peuvent généralement être connectés via des réseaux LPWAN, via un opérateur réseau tiers. Pour ces types d’appareils, vous pouvez utiliser [Azure IoT Central Device Bridge](../core/howto-build-iotc-device-bridge.md) pour envoyer les données de votre appareil à votre application IoT dans Azure IoT Central. Vous pouvez également utiliser des passerelles d’appareil compatibles IP et capables de se connecter directement à IoT Central.

## <a name="iot-central"></a>IoT Central 
Azure IoT Central est une plateforme d’application IoT qui vous permet de commencer rapidement votre solution IoT. Vous pouvez personnaliser et intégrer votre solution à des services tiers.
Une fois que vous avez connecté vos systèmes d’eau intelligents à IoT Central, vous pouvez contrôler les appareils, effectuer le suivi et déclencher des alertes, accéder à l’interface utilisateur à laquelle est intégré RBAC, accéder aux tableaux de bord d’insights configurables et accéder aux options d’extensibilité. 

## <a name="extensibility-and-integrations"></a>Extensibilité et intégrations
Vous pouvez étendre votre application IoT dans IoT Central et si vous le souhaitez :
* Transformer et intégrer vos données IoT en vue d’une analytique avancée, par exemple en entraînant des modèles Machine Learning via l’exportation continue de données à partir d’une application IoT Central
* Automatiser des workflows dans d’autres systèmes en déclenchant des actions via Power Automate ou des webhooks à partir d’une application IoT Central
* Accéder par programmation à votre application IoT dans IoT Central via des API IoT Central

## <a name="business-applications"></a>Applications métier 
Les données IoT peuvent être utilisées pour alimenter diverses applications métier au sein d’un service de gestion des déchets. Pour savoir comment connecter votre application de gestion des déchets connectée IoT Central à Field Services, suivez le tutoriel qui explique [comment intégrer une application à Dynamics 365 Field Services](./how-to-configure-connected-field-services.md). 

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [créer une application de gestion des déchets connectée](./tutorial-connected-waste-management.md) IoT Central
* Apprenez-en davantage sur les [modèles IoT Central Government](./overview-iot-central-government.md)
* Pour plus d’informations sur IoT Central, consultez la [vue d’ensemble d’IoT Central](../core/overview-iot-central.md)
