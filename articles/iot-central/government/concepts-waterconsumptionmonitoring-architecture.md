---
title: Architecture de référence pour une solution de suivi de la consommation d’eau créée avec Azure IoT Central | Microsoft Docs
description: Découvrez les concepts qui se rapportent à une solution de suivi de la consommation d’eau conçue avec Azure IoT Central.
author: miriambrus
ms.author: miriamb
ms.date: 12/11/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.openlocfilehash: a291a32c8fd635c9124a7e1f997b6c676c8c0bd4
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97586786"
---
# <a name="water-consumption-monitoring-reference-architecture"></a>Architecture de référence d’une solution de suivi de la consommation d’eau 

Vous pouvez créer des solutions de suivi de la consommation d’eau avec le **modèle d’application Azure IoT Central** en tant qu’application IoT de démarrage. Cet article fournit des conseils d’ordre général sur l’architecture de référence d’une solution de bout en bout. 

![Architecture du suivi de la consommation d’eau](./media/concepts-waterconsumptionmonitoring-architecture/concepts-waterconsumptionmonitoring-architecture1.png)

Concepts :

1. Appareils et connectivité  
1. IoT Central 
1. Extensibilité et intégrations
1. Applications métier

Intéressons-nous aux composants clés qui sont généralement impliqués dans une solution de suivi de la consommation d’eau.

## <a name="devices-and-connectivity"></a>Appareils et connectivité 
Cette section nomme généralement « systèmes d’eau intelligents » les appareils utilisés pour le suivi de la qualité de l’eau ou le suivi de la consommation d’eau. Les systèmes d’eau intelligents peuvent être des débitmètres, des moniteurs de qualité de l’eau, des vannes intelligentes, des détecteurs de fuite, etc.

Les appareils utilisés dans les systèmes d’eau intelligents sont généralement connectés via des réseaux LPWAN, via un opérateur réseau tiers. Pour ces types d’appareils, vous pouvez utiliser [Azure IoT Central Device Bridge](../core/howto-build-iotc-device-bridge.md) pour envoyer les données de votre appareil à votre application IoT dans Azure IoT Central. Vous pouvez également utiliser des passerelles d’appareil compatibles IP et capables de se connecter directement à IoT Central.

## <a name="iot-central"></a>IoT Central 
Azure IoT Central est une plateforme d’application IoT qui vous permet de commencer rapidement votre solution IoT. Vous pouvez personnaliser et intégrer votre solution à des services tiers.
Une fois que vous avez connecté vos systèmes d’eau intelligents à IoT Central, vous pouvez contrôler les appareils, effectuer le suivi et déclencher des alertes, accéder à l’interface utilisateur à laquelle est intégré RBAC, accéder aux tableaux de bord d’insights configurables et accéder aux options d’extensibilité. 


## <a name="extensibility-and-integrations"></a>Extensibilité et intégrations
Vous pouvez étendre votre application IoT dans IoT Central et si vous le souhaitez :
* Transformer et intégrer vos données IoT en vue d’une analytique avancée, par exemple en entraînant des modèles Machine Learning via l’exportation continue de données à partir d’une application IoT Central
* Automatiser des workflows dans d’autres systèmes en déclenchant des actions via Power Automate ou des webhooks à partir d’une application IoT Central
* Accéder programmatiquement à votre application IoT dans IoT Central via des API IoT Central

## <a name="business-applications"></a>Applications métier 
Les données IoT peuvent être utilisées pour alimenter différentes sortes d’applications métier au sein d’une solution. Pour savoir comment connecter votre application de suivi de la consommation d’eau IoT Central à Field Services, suivez le tutoriel qui explique [comment intégrer une application à Dynamics 365 Field Services](./how-to-configure-connected-field-services.md). 


## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [créer une application IoT Central pour le suivi de la consommation d’eau](./tutorial-water-consumption-monitoring.md)
* Apprenez-en davantage sur les [modèles IoT Central Government](./overview-iot-central-government.md)
* Pour plus d’informations sur IoT Central, consultez la [vue d’ensemble d’IoT Central](../core/overview-iot-central.md)
