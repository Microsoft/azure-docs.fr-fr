---
title: Fichier include
description: Fichier include
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/28/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 24a07109fc8f4d6ebd283dee7ee00107f0eb49b7
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95559575"
---
Vous pouvez utiliser l’API REST [Get Endpoint Health](/rest/api/iothub/iothubresource/getendpointhealth#iothubresource_getendpointhealth) pour obtenir l’état d’intégrité des points de terminaison. Nous vous recommandons d’utiliser les [métriques de routage IoT Hub](../articles/iot-hub/monitor-iot-hub-reference.md#routing-metrics) associées à la latence de routage des messages pour identifier et déboguer des erreurs lorsque le point de terminaison est inactif ou n’est pas sain, car nous nous attendons à ce que la latence soit plus élevée lorsque le point de terminaison se trouve dans l’un des états indiqués ci-dessous. Pour en savoir plus sur l’utilisation des métriques IoT Hub, consultez [Surveiller IoT Hub](../articles/iot-hub/monitor-iot-hub.md).

|État d’intégrité|Description|
|---|---|
|healthy|Le point de terminaison accepte des messages comme prévu.|
|unhealthy|Le point de terminaison n’accepte pas les messages et IoT Hub effectue une nouvelle tentative d’envoyer des messages à ce point de terminaison.|
|unknown|IoT Hub n’a pas tenté de remettre des messages à ce point de terminaison.|
|degraded|Le point de terminaison accepte les messages plus lentement que prévu ou récupère d’un état non sain.|
|dead|IoT Hub ne remet plus de messages à ce point de terminaison. Les nouvelles tentatives d’envoi de messages à ce point de terminaison ont échoué.|