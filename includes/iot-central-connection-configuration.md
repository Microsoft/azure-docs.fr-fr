---
title: Fichier include
description: Fichier include
services: iot-central
author: dominicbetts
ms.service: iot-central
ms.topic: include
ms.date: 11/03/2020
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 2b7881e7fa1ccbcec1325b2af0a570c86b0585a8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017483"
---
Quand vous exécutez l’exemple d’application d’appareil plus loin dans ce tutoriel, vous avez besoin des valeurs de configuration suivantes :

* Étendue de l’ID : dans votre application IoT Central, accédez à **Administration > Connexion de l’appareil**. Notez la valeur **Étendue de l’ID**.
* Clé primaire du groupe : Dans votre application IoT Central, accédez à **Administration > Connexion de l’appareil > SAS-IoT-Devices**. Notez la valeur **Clé primaire** de la signature d’accès partagé.

Utilisez Cloud Shell pour générer une clé d’appareil à partir de la clé SAS de groupe que vous venez de récupérer :

```azurecli-interactive
az extension add --name azure-iot
az iot central device compute-device-key  --device-id sample-device-01 --pk <the group SAS primary key value>
```

Notez la clé d’appareil générée. Vous l’utiliserez ultérieurement dans ce tutoriel.
