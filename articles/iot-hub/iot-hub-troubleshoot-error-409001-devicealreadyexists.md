---
title: Résolution de l'erreur Azure IoT Hub 409001 DeviceAlreadyExists
description: Correction de l'erreur 409001 DeviceAlreadyExists
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: troubleshooting
ms.date: 01/30/2020
ms.author: jlian
ms.openlocfilehash: 93ab2ecc8e820c461a7c79082ac1d50c24f0ba8b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76960290"
---
# <a name="409001-devicealreadyexists"></a>409001 DeviceAlreadyExists

Cet article décrit les causes et solutions des erreurs **409001 DeviceAlreadyExists**.

## <a name="symptoms"></a>Symptômes

Lorsque vous tentez d'inscrire un appareil dans IoT Hub, la requête échoue et vous recevez l'erreur **409001 DeviceAlreadyExists**.

## <a name="cause"></a>Cause :

Un autre appareil possède déjà la même identité dans IoT Hub. 

## <a name="solution"></a>Solution

Utilisez une autre identité d'appareil et réessayez.