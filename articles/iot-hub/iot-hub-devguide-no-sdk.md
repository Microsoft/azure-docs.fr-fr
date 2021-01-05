---
title: Développer avec un kit de développement logiciel (SDK) Azure IoT | Microsoft Docs
description: 'Guide du développeur : informations et liens vers des rubriques que vous pouvez utiliser pour créer des applications d’appareil et des applications back-end sans utiliser de kit de développement logiciel (SDK) Azure IoT.'
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 10/12/2020
ms.custom:
- mqtt
- amqp
- 'Role: IoT Device'
- 'Role: Cloud Development'
ms.openlocfilehash: ae8b01522a248b8b1dbdd255a9fcd55f16cf2369
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461716"
---
# <a name="develop-without-using-an-azure-iot-hub-sdk"></a>Développer sans utiliser de kit de développement logiciel (SDK) Azure IoT Hub

Cette rubrique fournit des informations et des liens utiles pour les développeurs qui souhaitent développer des applications d’appareil ou back-end sans utiliser les kits de développement logiciel (SDK) Azure IoT.

Microsoft recommande vivement d’utiliser un kit de développement logiciel (SDK) Azure IoT. Les kits de développement logiciel (SDK) Azure IoT d’appareil et service sont publiés sur de nombreuses plateformes populaires. Les kits de développement logiciel (SDK) fournissent une couche pratique qui gère la majeure partie de la complexité du protocole de communication sous-jacent, y compris la connexion et la reconnexion des appareils, et la stratégie de nouvelles tentatives. Les kits de développement logiciel (SDK) sont régulièrement mis à jour pour fournir les fonctionnalités les plus récentes exposées par IoT Hub, ainsi que les mises à jour de sécurité. L’utilisation des kits de développement logiciel (SDK) peut vous aider à réduire le temps de développement et le temps consacrés à la maintenance du code. Pour en savoir plus sur les kits de développement logiciel (SDK) Azure IoT, consultez [Kits de développement logiciel (SDK) Azure IoT d’appareil et de service](iot-hub-devguide-sdks.md). Pour plus d’informations sur les avantages de l’utilisation d’un kit de développement logiciel (SDK) Azure IoT, consultez les [Avantages de l’utilisation des kits de développement logiciel (SDK) Azure IoT et pièges à éviter si vous ne ](https://azure.microsoft.com/en-us/blog/benefits-of-using-the-azure-iot-sdks-in-your-azure-iot-solution/) billet de blog.

Même si IoT Hub prend en charge AMQP, AMQP sur WebSockets, HTTPS, MQTT et MQTT sur WebSockets pour la communication avec les appareils, nous vous recommandons d’utiliser MQTT si votre appareil le prend en charge.

## <a name="development-prerequisites"></a>Conditions préalables au développement

Avant de commencer le développement, vous devez avoir une connaissance approfondie de IoT Hub et des fonctionnalités que vous souhaitez que votre appareil ou application back-end implémente. Voici une liste très abrégée des sujets que vous devez connaître :

* Assurez-vous de bien comprendre les points de terminaison exposés par IoT Hub et les protocoles pris en charge sur chaque point de terminaison. Pour plus d’informations, voir [points de terminaison de IoT Hub](iot-hub-devguide-endpoints.md).

* Lorsqu’un choix de protocole est impliqué pour les applications d’appareil, nous vous recommandons vivement d’utiliser MQTT. Avant de choisir un protocole, veillez à bien comprendre les limitations imposées par chaque. Pour en savoir plus, voir [Choisir un protocole de communication](iot-hub-devguide-protocols.md).

* Pour comprendre l’authentification avec IoT Hub, consultez [contrôler l’accès à IoT Hub](iot-hub-devguide-security.md).

[!INCLUDE [iot-hub-include-x509-ca-signed-support-note](../../includes/iot-hub-include-x509-ca-signed-support-note.md)]

## <a name="help-on-different-protocols"></a>Aide sur différents protocoles

Pour obtenir de l’aide sur les protocoles suivants sans un kit de développement logiciel (SDK) Azure IoT :

* Les applications d’appareil ou back-end sur **AMQP**, consultez [Support AMQP](iot-hub-amqp-support.md).

* Pour les applications d’appareil sur **MQTT**, consultez [Support MQTT](iot-hub-mqtt-support.md). La majeure partie de cette rubrique traite directement via le protocole MQTT. Elle contient également des informations sur l’utilisation du [référentiel d’échantillons IoT MQTT](https://github.com/Azure-Samples/IoTMQTTSample). Ce référentiel contient des échantillons C qui utilisent la bibliothèque Mosquitto Eclipse pour envoyer des messages à IoT Hub.

* Les applications de périphérique ou back-end sur **HTTPS**, consultez les [API REST Azure IoT Hub](/rest/api/iothub/). N’oubliez pas, comme indiqué dans les [Conditions préalables au développement](#development-prerequisites), que vous ne pouvez pas utiliser l’authentification de l’autorité de certification X.509 avec HTTPS.

Pour les appareils, nous vous recommandons fortement d’utiliser MQTT si votre appareil le prend en charge.

## <a name="next-steps"></a>Étapes suivantes

* [Support MQTT](iot-hub-mqtt-support.md)