---
title: Gestion des appareils Azure IoT avec Azure IoT Tools pour VS Code
description: Utilisez Azure IoT Tools pour Visual Studio Code pour la gestion des appareils Azure IoT Hub, avec les méthodes directes et les options de gestion des propriétés souhaitées du jumeau.
author: formulahendry
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/04/2019
ms.author: junhan
ms.openlocfilehash: d85e0e967dd802a77ccbc11b884d7a9f2891524d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81688100"
---
# <a name="use-azure-iot-tools-for-visual-studio-code-for-azure-iot-hub-device-management"></a>Utiliser Azure IoT Tools pour Visual Studio Code pour la gestion des appareils Azure IoT Hub

![Diagramme de bout en bout](media/iot-hub-get-started-e2e-diagram/2.png)

[Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) est une extension Visual Studio Code qui facilite la gestion d’IoT Hub et le développement des applications IoT. Il est fourni avec des options de gestion que vous pouvez utiliser pour effectuer diverses tâches.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

| Option de gestion          | Tâche                    |
|----------------------------|--------------------------------|
| Méthodes directes             | Faites agir un appareil comme commençant/arrêtant d’envoyer des messages ou comme redémarrant l’appareil.                                        |
| Lire le jumeau d’appareil           | Obtenez l’état signalé d’un appareil. Par exemple, l’appareil signale que le voyant clignote maintenant.                                    |
| Mettre à jour le jumeau d’appareil         | Mettez un appareil dans certains états, par exemple en réglant un voyant sur le vert ou en définissant l’intervalle d’envoi de télémétrie sur 30 minutes.         |
| Messages Cloud vers appareil   | Envoyez des notifications à un appareil. Par exemple, « Forte probabilité de pluie aujourd'hui. N’oubliez pas de prendre un parapluie avec vous. »              |

Pour plus d’explications sur les différences et des conseils sur l’utilisation de ces options, consultez [l’aide sur la communication appareil-à-cloud](iot-hub-devguide-d2c-guidance.md) et [l’aide sur la communication cloud-à-appareil](iot-hub-devguide-c2d-guidance.md).

Les représentations d’appareil sont des documents JSON qui stockent des informations sur l’état des appareils (métadonnées, configurations et conditions). IoT Hub conserve une représentation d’appareil pour chaque appareil que vous y connectez. Pour plus d’informations sur les représentations d’appareil, consultez [Prise en main des représentations d’appareils](iot-hub-node-node-twin-getstarted.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="what-you-learn"></a>Contenu

Vous allez apprendre à utiliser Azure IoT Tools pour Visual Studio Code avec différentes options de gestion sur votre ordinateur de développement.

## <a name="what-you-do"></a>Procédure

Exécuter Azure IoT Tools pour Visual Studio Code avec diverses options de gestion.

## <a name="what-you-need"></a>Ce dont vous avez besoin

* Un abonnement Azure actif.
* Une instance Azure IoT Hub associée à votre abonnement.
* [Visual Studio Code](https://code.visualstudio.com/)
* [Azure IoT Tools pour VS Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) ou copiez cette URL et collez-la dans une fenêtre de navigateur : `vscode:extension/vsciot-vscode.azure-iot-tools`.

## <a name="sign-in-to-access-your-iot-hub"></a>Se connecter pour accéder à votre hub ioT

1. En mode **Explorateur** dans Visual Studio Code, développez la section **Appareils Azure IoT Hub** dans le coin inférieur gauche.

2. Cliquez sur **Sélectionner IoT Hub** dans le menu contextuel.

3. Une fenêtre contextuelle s’affiche en bas à droite pour vous permettre de vous connecter à Azure pour la première fois.

4. Une fois que vous êtes connecté, votre liste d’abonnements Azure s’affiche. Sélectionnez Abonnement Azure et IoT Hub.

5. La liste des appareils apparaît quelques secondes plus tard sous l’onglet **Appareils Azure IoT Hub**.

   > [!Note]
   > Vous pouvez également effectuer la configuration en choisissant **Définir la chaîne de connexion IoT Hub**. Entrez la chaîne de connexion de stratégie **iothubowner** IoT Hub à laquelle se connecte votre appareil IoT dans la fenêtre contextuelle.

## <a name="direct-methods"></a>Méthodes directes

1. Cliquez avec le bouton droit sur votre appareil et sélectionnez **Appeler une méthode directe**. 

2. Entrez le nom de la méthode et la charge utile dans la zone d’entrée.

3. Les résultats seront affichés dans la vue **SORTIE** > **Azure IoT Hub**.

## <a name="read-device-twin"></a>Lire le jumeau d’appareil

1. Cliquez avec le bouton droit sur votre appareil et sélectionnez **Modifier le jumeau d’appareil**. 

2. Un fichier **azure-iot-device-twin.json** s’ouvre avec le contenu du jumeau d’appareil.

## <a name="update-device-twin"></a>Mettre à jour le jumeau d’appareil

1. Modifiez les **étiquettes** ou le champ **properties.desired**.

2. Cliquez avec le bouton droit sur le fichier **azure-iot-device-twin.json**.

3. Sélectionnez **Mettre à jour le jumeau d’appareil** pour mettre à jour le jumeau d’appareil.

## <a name="send-cloud-to-device-messages"></a>Envoi de messages cloud vers appareil

Pour envoyer un message à partir de votre instance IoT Hub sur votre appareil, procédez comme suit :
 
1. Cliquez avec le bouton droit sur votre appareil et sélectionnez **Envoyer un message cloud-à-appareil**. 

2. Entrez le message dans la zone de saisie.

3. Les résultats seront affichés dans la vue **SORTIE** > **Azure IoT Hub**.

## <a name="next-steps"></a>Étapes suivantes

Vous avez découvert comment utiliser Azure IoT Tools pour Visual Studio Code avec diverses options de gestion.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
