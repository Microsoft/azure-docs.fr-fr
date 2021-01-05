---
title: Gestion de l’accès réseau public pour le hub IoT Azure
description: Documentation sur la désactivation et l’activation de l’accès au réseau public pour le hub IoT
author: jlian
ms.author: jlian
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 07/01/2020
ms.openlocfilehash: c82f98df8fb79fa10f2e30b219c1a02bb646e2de
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85937447"
---
# <a name="managing-public-network-access-for-your-iot-hub"></a>Gestion de l’accès réseau public pour votre hub IoT

Pour limiter l’accès au seul [point de terminaison privé pour votre hub IoT dans votre réseau virtuel](virtual-network-support.md), désactivez l’accès au réseau public. Pour ce faire, utilisez le portail Azure ou l’API `publicNetworkAccess`. 

## <a name="turn-off-public-network-access-using-azure-portal"></a>Désactiver l’accès au réseau public à l’aide du portail Azure

1. Visitez le [portail Azure](https://portal.azure.com)
2. Accédez à votre hub IoT.
3. Sélectionnez **Mise en réseau** dans le menu Place de marché.
4. Sous « Autoriser l’accès du réseau public à », sélectionnez **Désactivé**
5. Sélectionnez **Enregistrer**.

:::image type="content" source="media/iot-hub-publicnetworkaccess/turn-off-public-network-access.png" alt-text="Image qui montre le portail Azure où désactiver l’accès au réseau public" lightbox="media/iot-hub-publicnetworkaccess/turn-off-public-network-access.png":::

Pour activer l’accès au réseau public, sélectionnez **Activé**, puis **Enregistrer**.

## <a name="ip-filter"></a>Filtre IP 

Si l’accès au réseau public est désactivé, toutes les règles de [filtre IP](iot-hub-ip-filtering.md) sont ignorées. Cela est dû au fait que toutes les adresses IP de l’Internet public sont bloquées. Pour utiliser le filtre IP, utilisez l’option **Plage d’adresses IP sélectionnées**.

## <a name="bug-fix-with-built-in-event-hub-compatible-endpoint"></a>Résolution de bogue via un point de terminaison compatible avec Event Hub intégré

Il existe un bogue dans IoT Hub où le [point de terminaison intégré compatible avec Event Hub](iot-hub-devguide-messages-read-builtin.md) continue d’être accessible via l’Internet public alors que l’accès du réseau public à IoT Hub est désactivé. Pour en savoir plus et nous contacter concernant ce bogue, consultez [Désactiver l’accès du réseau public pour IoT Hub désactive l’accès au point de terminaison Event Hub intégré](https://azure.microsoft.com/updates/iot-hub-public-network-access-bug-fix).