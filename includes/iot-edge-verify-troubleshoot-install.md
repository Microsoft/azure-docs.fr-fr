---
title: Après l’installation et l’approvisionnement, vérifier la réussite et résoudre les problèmes
description: Fichier include
services: iot-edge
author: kgremban
ms.service: iot-edge
ms.topic: include
ms.date: 10/06/2020
ms.author: kgremban
ms.custom: include file
ms.openlocfilehash: 295769e5d4924e529b296dbb0b9d405ee197c1db
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91979200"
---
## <a name="verify-successful-setup"></a>Vérifier la réussite de l’installation

Vérifiez l’état du service IoT Edge. Il doit être listé comme étant en cours d’exécution.  

# <a name="linux"></a>[Linux](#tab/linux)

```bash
systemctl status iotedge
```

# <a name="windows"></a>[Windows](#tab/windows)

```powershell
Get-Service iotedge
```

---

Consultez les journaux d’activité de service.

# <a name="linux"></a>[Linux](#tab/linux)

```bash
journalctl -u iotedge --no-pager --no-full
```

# <a name="windows"></a>[Windows](#tab/windows)

Si vous venez tout juste d’installer le runtime IoT Edge, vous pouvez voir une liste d’erreurs de la période entre l’exécution de la commande **Deploy-IoTEdge** et de la commande **Initialize-IoTEdge**. Ces erreurs sont attendues, car le service tente de démarrer avant d’être configuré.

```powershell
. {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
```

---

Lancez l’[outil de résolution des problèmes](../articles/iot-edge/troubleshoot.md#run-the-check-command) pour vérifier les erreurs de configuration et de mise en réseau les plus courantes.

```powershell
iotedge check
```

Tant que vous n’avez pas déployé votre premier module sur IoT Edge sur votre appareil, le module système **$edgeHub** n’est pas déployé sur l’appareil. Par conséquent, la vérification automatisée renverra une erreur pour la vérification de la connectivité `Edge Hub can bind to ports on host`. Vous pouvez ignorer cette erreur, sauf si elle se produit après le déploiement d’un module sur l’appareil.

Enfin, listez les modules en cours d’exécution :

```powershell
iotedge list
```

Après une nouvelle installation, le seul module que vous devez voir en cours d’exécution est **edgeAgent**.

## <a name="tips-and-troubleshooting"></a>Conseils et résolution des problèmes

Sur les appareils aux ressources limitées, il est vivement recommandé d’affecter la valeur *false* à la variable d’environnement *OptimizeForPerformance*, conformément aux instructions figurant dans le [guide de résolution des problèmes](../articles/iot-edge/troubleshoot.md).

Si votre appareil ne peut pas se connecter à IoT Hub et que votre réseau comporte un serveur proxy, effectuez les étapes décrites dans [Configurer votre appareil IoT Edge pour communiquer via un serveur proxy](../articles/iot-edge/how-to-configure-proxy-support.md).

# <a name="linux"></a>[Linux](#tab/linux)

Sur les appareils Linux, vous avez besoin de privilèges élevés pour exécuter les commandes `iotedge`. Après l’installation du runtime, déconnectez-vous de votre machine et reconnectez-vous pour mettre à jour vos autorisations automatiquement. À ce moment-là, utilisez `sudo` pour exécuter des commandes avec des privilèges élevés.

# <a name="windows"></a>[Windows](#tab/windows)

Sur les appareils Windows exécutant des conteneurs Windows, le moteur de conteneur Moby a été installé dans le cadre d’IoT Edge. Le moteur Moby a été conçu pour s’exécuter en parallèle avec le Bureau Docker. Vous pouvez utiliser des commandes `docker` si vous souhaitez interagir directement avec les conteneurs sur votre appareil. Toutefois, vous devez spécifiquement cibler le moteur Moby dans le cas où le Bureau Docker est également installé sur l’appareil.

Par exemple, pour répertorier toutes les images Docker, exécutez la commande suivante :

```powershell
docker images
```

Pour répertorier toutes les images Moby, modifiez la même commande en ajoutant un pointeur vers le moteur Moby :

```powershell
docker -H npipe:////./pipe/iotedge_moby_engine images
```

Le moteur d’URI est répertorié dans la sortie du script d’installation, ou vous pouvez le trouver dans la section des paramètres du runtime du conteneur pour le fichier config.yaml.

![moby_runtime uri dans config.yaml](./media/iot-edge-verify-troubleshoot-install/moby-runtime-uri.png)

---
