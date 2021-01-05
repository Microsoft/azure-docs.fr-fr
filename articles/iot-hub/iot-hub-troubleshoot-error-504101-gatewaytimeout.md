---
title: Résolution de l’erreur Azure IoT Hub 504101 GatewayTimeout
description: Comprendre comment corriger l’erreur 504101 GatewayTimeout
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: troubleshooting
ms.date: 01/30/2020
ms.author: jlian
ms.custom: amqp
ms.openlocfilehash: 373acc30ed652a7f540e840dfad5eeeda65ca179
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81759559"
---
# <a name="504101-gatewaytimeout"></a>504101 GatewayTimeout

Cet article décrit les causes et solutions des erreurs **504101 GatewayTimeout**.

## <a name="symptoms"></a>Symptômes

Lorsque vous tentez d’appeler une méthode directe à partir de l’IoT Hub sur un appareil, la demande échoue en générant l’erreur **504101 GatewayTimeout**.

## <a name="cause"></a>Cause :

### <a name="cause-1"></a>Cause 1

L’IoT Hub a rencontré une erreur et n’a pas pu vérifier que l’exécution de la méthode directe s’était terminée avant l’expiration du délai d’attente.

### <a name="cause-2"></a>Cause 2

Lors de l’utilisation d’une version antérieure du Kit de développement logiciel (SDK) C# Azure IoT (< 1.19.0), le lien AMQP entre l’appareil et IoT Hub peut être abandonné en mode silencieux en raison d’un bogue.

## <a name="solution"></a>Solution

### <a name="solution-1"></a>Solution 1

Faites une nouvelle tentative.

### <a name="solution-2"></a>Solution 2

Opérez une mise à niveau vers la dernière version du Kit de développement logiciel (SDK) C# Azure IoT.