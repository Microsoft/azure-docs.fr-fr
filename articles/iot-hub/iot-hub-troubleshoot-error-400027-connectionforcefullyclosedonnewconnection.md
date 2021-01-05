---
title: Résolution de l’erreur 400027 ConnectionForcefullyClosedOnNewConnection dans Azure IoT Hub
description: Comprendre comment corriger l’erreur 400027 ConnectionForcefullyClosedOnNewConnection
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: troubleshooting
ms.date: 01/30/2020
ms.author: jlian
ms.custom:
- mqtt
- fasttrack-edit
- iot
ms.openlocfilehash: e5d1dc345c72d77be6172fb9c3a10eb2f38d186a
ms.sourcegitcommit: 4bee52a3601b226cfc4e6eac71c1cb3b4b0eafe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94506326"
---
# <a name="400027-connectionforcefullyclosedonnewconnection"></a>400027 ConnectionForcefullyClosedOnNewConnection

Cet article décrit les causes et solutions des erreurs **400027 ConnectionForcefullyClosedOnNewConnection**.

## <a name="symptoms"></a>Symptômes

Votre appareil est déconnecté avec l’erreur **Communication_Error**, le motif indiqué par **ConnectionStatusChangeReason** étant l’utilisation du SDK .NET et du type de transport MQTT.

Votre opération sur le jumeau appareil-à-cloud (par exemple, lecture ou correction de propriétés signalées) ou votre appel de méthode directe échouent avec le code d’erreur **400027**.

## <a name="cause"></a>Cause

Un autre client a créé une connexion à IoT Hub à l’aide des mêmes informations d’identification. L’IoT Hub a donc fermé la connexion précédente. L’IoT Hub ne permet pas à plusieurs clients de se connecter à l’aide du même ensemble d’informations d’identification.

## <a name="solution"></a>Solution

Assurez-vous que chaque client se connecte à l’IoT Hub à l’aide de sa propre identité.