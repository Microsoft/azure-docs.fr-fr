---
title: Résolution de l’erreur Azure IoT Hub 403002 IoTHubQuotaExceeded
description: Comprendre comment corriger l’erreur 403002 IoTHubQuotaExceeded
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: troubleshooting
ms.date: 01/30/2020
ms.author: jlian
ms.openlocfilehash: 8312c3267e826088f34b3bffe1520703eec00bdc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76960390"
---
# <a name="403002-iothubquotaexceeded"></a>403002 IoTHubQuotaExceeded

Cet article décrit les causes et solutions des erreurs **403002 IoTHubQuotaExceeded**.

## <a name="symptoms"></a>Symptômes

Toutes les demandes que vous adressez à IoT Hub échouent avec l’erreur **403002 IoTHubQuotaExceeded**. La liste d’appareils du hub IoT ne se charge pas dans le Portail Azure.

## <a name="cause"></a>Cause :

Le quota quotidien des messages pour le hub IoT est dépassé. 

## <a name="solution"></a>Solution

[Mettez à niveau ou augmentez le nombre d’unités sur le hub IoT](iot-hub-upgrade.md) ou attendez le jour UTC suivant que le quota quotidien soit actualisé.

## <a name="next-steps"></a>Étapes suivantes

* Pour comprendre la façon dont les opérations sont comptabilisées dans le quota, telles que les requêtes de jumeaux et les méthodes directes, consultez [Comprendre la tarification d’IoT Hub](iot-hub-devguide-pricing.md#charges-per-operation)
* Pour configurer la surveillance de l’utilisation quotidienne du quota, configurez une alerte avec la métrique *Nombre total de messages utilisés*. Pour obtenir des instructions pas à pas, consultez [Configurer des métriques et des alertes avec IoT Hub](tutorial-use-metrics-and-diags.md#set-up-metrics)