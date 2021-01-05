---
title: Résolution de l’erreur Azure IoT Hub 429001 ThrottlingException
description: Comprendre comment corriger l’erreur 429001 ThrottlingException
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: troubleshooting
ms.date: 01/30/2020
ms.author: jlian
ms.openlocfilehash: d2f12a6982886eeaa375151c5b8a73acc573aab9
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545359"
---
# <a name="429001-throttlingexception"></a>429001 ThrottlingException

Cet article décrit les causes et solutions des erreurs **429001 ThrottlingException** .

## <a name="symptoms"></a>Symptômes

Les demandes que vous adressez à IoT Hub échouent avec l’erreur **429001 ThrottlingException** .

## <a name="cause"></a>Cause :

Les [limites de limitation](./iot-hub-devguide-quotas-throttling.md) IoT Hub ont été dépassées pour l’opération demandée.

## <a name="solution"></a>Solution

Vérifiez si vous avez atteint la limite de limitation en comparant votre métrique de *tentatives d’envoi de messages de télémétrie* aux limites spécifiées plus haut. Vous pouvez également vérifier la métrique *Nombre d’erreurs de limitation* . Pour plus d’informations sur ces métriques, consultez [Métriques de télémétrie d’appareil](monitor-iot-hub-reference.md#device-telemetry-metrics). Pour plus d’informations sur l’utilisation des métriques pour vous aider à surveiller votre IoT Hub, consultez [Surveiller IoT Hub](monitor-iot-hub.md).

IoT Hub ne retourne l’erreur 429 ThrottlingException qu’après que la limite a été violée pendant un laps de temps trop long. Cela a pour but de ne pas supprimer vos messages si votre IoT Hub reçoit du trafic en rafales. Dans le même temps, IoT Hub traite les messages en fonction du taux de limitation de l’opération, qui peut être lent en cas de trafic intense dans le backlog. Pour plus d’informations, voir [Régulation de flux IoT Hub](./iot-hub-devguide-quotas-throttling.md#traffic-shaping).

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez des limites de quota ou de limitation, envisagez d’[augmenter la taille des instances de votre IoT Hub](./iot-hub-scaling.md).