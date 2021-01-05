---
title: Résolution de l’erreur Azure IoT Hub 404103 DeviceNotOnline
description: Comprendre comment corriger l’erreur 404103 DeviceNotOnline
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: troubleshooting
ms.date: 01/30/2020
ms.author: jlian
ms.openlocfilehash: e648428f924cfc33421c8591c41f7ac85b05a033
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76960298"
---
# <a name="404103-devicenotonline"></a>404103 DeviceNotOnline

Cet article décrit les causes et solutions des erreurs **404103 DeviceNotFound**.

## <a name="symptoms"></a>Symptômes

Une méthode directe d’accès à un appareil échoue en générant l’erreur **404103 DeviceNotOnline**, même si l’appareil est en ligne. 

## <a name="cause"></a>Cause :

Si vous savez que l’appareil est en ligne et continuez à recevoir l’erreur, cela est probablement dû au fait que le rappel de méthode directe n’est pas inscrit sur l’appareil.

## <a name="solution"></a>Solution

Pour configurer correctement votre appareil pour les rappels de méthode directe, voir [Gérer une méthode directe sur un appareil](iot-hub-devguide-direct-methods.md#handle-a-direct-method-on-a-device).