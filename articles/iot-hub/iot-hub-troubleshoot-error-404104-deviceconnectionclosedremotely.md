---
title: Résolution de l’erreur Azure IoT Hub 404104 DeviceConnectionClosedRemotely
description: Comprendre comment résoudre l’erreur Azure IoT Hub 404104 DeviceConnectionClosedRemotely
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: troubleshooting
ms.date: 01/30/2020
ms.author: jlian
ms.custom: mqtt
ms.openlocfilehash: 673a76417739fa59a91979cca7c6807a584868f0
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92538253"
---
# <a name="404104-deviceconnectionclosedremotely"></a>404104 DeviceConnectionClosedRemotely

Cet article décrit les causes et solutions des erreurs **404104 DeviceConnectionClosedRemotely**.

## <a name="symptoms"></a>Symptômes

### <a name="symptom-1"></a>Symptôme 1

Les appareils se déconnectent à intervalles réguliers (toutes les 65 minutes, par exemple) et **404104 DeviceConnectionClosedRemotely** apparaît dans les journaux de ressources IoT Hub. Parfois, **401003 IoTHubUnauthorized** s'affiche, de même qu'un événement de connexion d'appareil réussie moins d'une minute plus tard.

### <a name="symptom-2"></a>Symptôme 2

Les appareils se déconnectent de façon aléatoire et **404104 DeviceConnectionClosedRemotely** apparaît dans les journaux de ressources IoT Hub.

### <a name="symptom-3"></a>Symptôme 3

De nombreux appareils se déconnectent en même temps et vous constatez une baisse au niveau des [métriques des appareils connectés (connectedDeviceCount)](monitor-iot-hub-reference.md). De plus, les journaux Azure Monitor présentent un plus grand nombre d'erreurs **404104 DeviceConnectionClosedRemotely** et d'[erreurs internes 500xxx](iot-hub-troubleshoot-error-500xxx-internal-errors.md).

## <a name="causes"></a>Causes

### <a name="cause-1"></a>Cause 1

Le [jeton SAS utilisé pour se connecter à IoT Hub](iot-hub-devguide-security.md#security-tokens) a expiré et dès lors, IoT Hub déconnecte l’appareil. La connexion est rétablie une fois le jeton actualisé par l’appareil. Par exemple, [le jeton SAS expire toutes les heures par défaut pour le kit de développement logiciel (SDK) C](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-authentication), ce qui peut entraîner des déconnexions régulières.

Pour plus d’informations, consultez [401003 IoTHubUnauthorized - Cause](iot-hub-troubleshoot-error-401003-iothubunauthorized.md#cause-1).

### <a name="cause-2"></a>Cause 2

Voici quelques possibilités :

- L’appareil a perdu la connectivité réseau sous-jacente plus longtemps que [MQTT keep-alive](iot-hub-mqtt-support.md#default-keep-alive-timeout), ce qui se traduit par un délai d’inactivité distant. Le paramètre MQTT keep-alive peut être différent selon l’appareil.

- L’appareil a envoyé une réinitialisation au niveau TCP/IP, mais pas au niveau de l'application `MQTT DISCONNECT`. En fait, l’appareil a brusquement interrompu la connexion socket sous-jacente. Ce problème relève parfois de bogues dans les versions antérieures du kit de développement logiciel (SDK) Azure IoT.

- L'application côté appareil s'est bloquée.

### <a name="cause-3"></a>Cause 3

IoT Hub peut rencontrer un problème temporaire. Voir [Cause d’erreur de serveur interne à IoT Hub](iot-hub-troubleshoot-error-500xxx-internal-errors.md#cause).

## <a name="solutions"></a>Solutions

### <a name="solution-1"></a>Solution 1

Consultez [401003 IoTHubUnauthorized - Solution 1](iot-hub-troubleshoot-error-401003-iothubunauthorized.md#solution-1)

### <a name="solution-2"></a>Solution 2

- Assurez-vous que l’appareil dispose d'une bonne connectivité à IoT Hub en [testant la connexion](tutorial-connectivity.md). Si le réseau n’est pas fiable ou intermittent, nous vous déconseillons d'augmenter la valeur keep-alive car cela pourrait entraîner une détection plus longue (via les alertes Azure Monitor, par exemple). 

- Utiliser les dernières versions des [kits de développement logiciel (SDK) IoT](iot-hub-devguide-sdks.md).

### <a name="solution-3"></a>Solution 3

Voir [Solutions des erreurs de serveur internes à IoT Hub](iot-hub-troubleshoot-error-500xxx-internal-errors.md#solution).

## <a name="next-steps"></a>Étapes suivantes

Nous vous recommandons d’utiliser les kits Azure IoT device SDK pour gérer les connexions de manière fiable. Pour plus d'informations, consultez [Gérer la connectivité et la messagerie fiable à l’aide des kits de développement logiciel (SDK) d’appareil Azure IoT Hub](iot-hub-reliability-features-in-sdks.md)