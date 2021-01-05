---
title: Traduire des protocoles modbus avec des passerelles - Azure IoT Edge | Microsoft Docs
description: Autoriser les appareils utilisant Modbus TCP à communiquer avec Azure IoT Hub en créant un appareil de passerelle IoT Edge
author: kgremban
manager: philmea
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 11/19/2019
ms.author: kgremban
ms.openlocfilehash: d07a1d1ab0d3b1f4315c09f1c403126139b34612
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92043899"
---
# <a name="connect-modbus-tcp-devices-through-an-iot-edge-device-gateway"></a>Connecter des appareils Modbus TCP via une passerelle d’appareils IoT Edge

Si vous souhaitez connecter des appareils IoT utilisant les protocoles Modbus TCP ou RTU à un hub IoT Azure, vous pouvez utiliser un appareil IoT Edge comme passerelle. Le périphérique de passerelle lit les données à partir de vos appareils Modbus, puis les communique dans le cloud en utilisant un protocole pris en charge.

![Les appareils Modbus se connectent à IoT Hub via une passerelle IoT Edge](./media/deploy-modbus-gateway/diagram.png)

Cet article vous explique comment créer votre propre image conteneur pour un module Modbus (vous pouvez également utiliser un exemple prédéfini), avant de la déployer sur l’appareil IoT Edge qui sera votre passerelle.

Cet article suppose que vous utilisez le protocole Modbus TCP. Pour plus d’informations sur la façon de configurer le module pour prendre en charge le protocole Modbus RTU, consultez le projet [Module Modbus Azure IoT Edge](https://github.com/Azure/iot-edge-modbus) sur GitHub.

## <a name="prerequisites"></a>Prérequis

* Un appareil Azure IoT Edge. Pour une procédure détaillée montrant comment en configurer un, consultez [Déployer Azure IoT Edge sur Windows](quickstart.md) ou [Linux](quickstart-linux.md).
* Chaîne de connexion de clé primaire de l’appareil IoT Edge.
* Un appareil Modbus physique ou simulé prenant en charge Modbus TCP. Vous devez connaître son adresse IPv4.

## <a name="prepare-a-modbus-container"></a>Préparer un conteneur Modbus

Si vous souhaitez tester la fonctionnalité de passerelle Modbus, Microsoft propose un module d’exemple que vous pouvez utiliser. Vous pouvez accéder au module à partir de la Place de marché Azure, de [Modbus](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft_iot.edge-modbus?tab=Overview) ou avec l’URI d’image `mcr.microsoft.com/azureiotedge/modbus:1.0`.

Si vous souhaitez créer votre propre module et le personnaliser en fonction de votre environnement, un projet open source [Module Modbus Azure IoT Edge](https://github.com/Azure/iot-edge-modbus) existe sur GitHub. Suivez les instructions du projet pour créer votre propre image conteneur. Pour créer une image de conteneur, reportez-vous aux rubriques [Développer des modules C# dans Visual Studio](./how-to-visual-studio-develop-module.md) ou [Développer des modules dans Visual Studio Code](how-to-vs-code-develop-module.md). Ces articles fournissent des instructions sur la création de modules et sur la publication des images conteneur sur un registre.

## <a name="try-the-solution"></a>Essayer la solution

Cette section est consacrée au déploiement de l’exemple de module Modbus de Microsoft sur votre appareil IoT Edge.

1. Accédez à votre IoT Hub sur le [portail Azure](https://portal.azure.com/).

2. Accédez à **IoT Edge** et cliquez sur votre appareil IoT Edge.

3. Sélectionnez **Définir des modules**.

4. Dans la section **Modules IoT Edge**, ajoutez le module Modbus :

   1. Cliquez sur la liste déroulante **Ajouter** et sélectionnez **Module Place de marché**.
   2. Recherchez `Modbus`, puis sélectionnez le **Module Modbus TCP** de Microsoft.
   3. Le module est automatiquement configuré pour votre IoT Hub et apparaît dans la liste des modules IoT Edge. Les itinéraires sont également configurés automatiquement. Sélectionnez **Revoir + créer**.
   4. Passez en revue le manifeste de déploiement et sélectionnez **Créer**.

5. Sélectionnez le module Modbus `ModbusTCPModule` dans la liste et sélectionnez l’onglet **Paramètres de jumeau de module**. Le JSON requis pour les propriétés de représentation du module souhaitées est automatiquement rempli.

6. Recherchez la propriété **SlaveConnection** dans le JSON et définissez sa valeur sur l’adresse IPv4 de votre appareil Modbus.

7. Sélectionnez **Update**.

8. Sélectionnez **Vérifier + Créer**, vérifiez le déploiement, puis sélectionnez **Créer**.

9. Revenez à la page de détails de l’appareil et sélectionnez **Actualiser**. Vous devez voir le nouveau module `ModbusTCPModule` en cours d’exécution avec le runtime IoT Edge.

## <a name="view-data"></a>Afficher les données

Afficher les données provenant du module Modbus :

```cmd/sh
iotedge logs modbus
```

Vous pouvez afficher les données de télémétrie envoyées par l’appareil à l’aide de l’[extension Azure IoT Hub pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit) (anciennement l’extension Azure IoT Toolkit).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la façon dont les appareils IoT Edge peuvent agir en tant que passerelles, consultez [Créer un appareil IoT Edge agissant comme une passerelle transparente](./how-to-create-transparent-gateway.md).
* Pour plus d’informations sur le fonctionnement des modules IoT Edge, consultez [Comprendre les modules Azure IoT Edge](iot-edge-modules.md).