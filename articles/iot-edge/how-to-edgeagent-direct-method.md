---
title: Méthodes directes edgeAgent intégrées - Azure IoT Edge
description: Surveiller et gérer un déploiement IoT Edge à l’aide de méthodes directes intégrées dans le module de runtime de l’agent IoT Edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 03/02/2020
ms.topic: conceptual
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 42c79526288fb7e05959ac60cddc6f468656ffd4
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91972541"
---
# <a name="communicate-with-edgeagent-using-built-in-direct-methods"></a>Communiquer avec edgeAgent à l’aide de méthodes directes intégrées

Surveillez et gérez les déploiements d’IoT Edge à l’aide des méthodes directes incluses dans le module de l’agent IoT Edge. Les méthodes directes sont implémentées sur l’appareil, puis peuvent être appelées à partir du cloud. L’agent IoT Edge comprend des méthodes directes qui vous aident à surveiller et gérer vos appareils IoT Edge à distance.

Pour plus d’informations sur les méthodes directes, leur utilisation et la façon de les implémenter dans vos propres modules, consultez [Comprendre et appeler des méthodes directes à partir d’IoT Hub](../iot-hub/iot-hub-devguide-direct-methods.md).

Les noms de ces méthodes directes sont gérés sans tenir compte de la casse.

## <a name="ping"></a>Ping

La méthode **ping** est utile pour vérifier si IoT Edge s’exécute sur un appareil, ou si l’appareil dispose d’une connexion ouverte à IoT Hub. Utilisez cette méthode directe pour envoyer une requête ping à l'agent IoT Edge et obtenir son état. Une commande ping réussie retourne une charge utile vide et **"status": 200**.

Par exemple :

```azurecli
az iot hub invoke-module-method --method-name 'ping' -n <hub name> -d <device name> -m '$edgeAgent'
```

Dans le portail Azure, appelez la méthode avec le nom de la méthode `ping` et une charge utile JSON vide `{}`.

![Appeler la méthode directe « ping » dans le portail Azure](./media/how-to-edgeagent-direct-method/ping-direct-method.png)

## <a name="restart-module"></a>Redémarrer le module

La méthode **RestartModule** permet la gestion à distance des modules qui s’exécutent sur un appareil IoT Edge. Si un module signale un état d’échec ou un autre comportement anormal, vous pouvez déclencher l’agent IoT Edge pour le redémarrer. Une commande de redémarrage réussie renvoie une charge utile vide et **"status": 200**.

La méthode RestartModule est disponible dans IoT Edge version 1.0.9 et versions ultérieures. 

Vous pouvez utiliser la méthode directe RestartModule sur n’importe quel module s’exécutant sur un appareil IoT Edge, y compris le module edgeAgent lui-même. Toutefois, si vous utilisez cette méthode directe pour arrêter edgeAgent, vous ne recevrez pas de résultat de réussite, car la connexion est interrompue lors du redémarrage du module.

Par exemple :

```azurecli
az iot hub invoke-module-method --method-name 'RestartModule' -n <hub name> -d <device name> -m '$edgeAgent' --method-payload \
'
    {
        "schemaVersion": "1.0",
        "id": "<module name>"
    }
'
```

Dans le portail Azure, appelez la méthode avec le nom de la méthode `RestartModule` et la charge utile JSON suivante :

```json
{
    "schemaVersion": "1.0",
    "id": "<module name>"
}
```

![Appeler la méthode directe ’RestartModule’ dans le portail Azure](./media/how-to-edgeagent-direct-method/restartmodule-direct-method.png)

## <a name="diagnostic-direct-methods"></a>Méthodes directes de diagnostic

* [GetModuleLogs](how-to-retrieve-iot-edge-logs.md#retrieve-module-logs) : Récupérez les journaux de module en les intégrant dans la réponse de la méthode directe.
* [UploadModuleLogs](how-to-retrieve-iot-edge-logs.md#upload-module-logs) : Récupérez les journaux de module et chargez-les dans le Stockage Blob Azure.
* [UploadSupportBundle](how-to-retrieve-iot-edge-logs.md#upload-support-bundle-diagnostics) : Récupérez les journaux de module à l’aide d’un bundle de support et chargez un fichier zip dans le Stockage Blob Azure.
* [GetTaskStatus](how-to-retrieve-iot-edge-logs.md#get-upload-request-status) : Vérifiez l’état d’une requête de chargement de journaux ou de bundle de support.

Ces méthodes directes de diagnostic sont disponibles à partir de la version 1.0.10.

## <a name="next-steps"></a>Étapes suivantes

[Propriétés des jumeaux de module de l’agent IoT Edge et du hub IoT Edge](module-edgeagent-edgehub.md)
