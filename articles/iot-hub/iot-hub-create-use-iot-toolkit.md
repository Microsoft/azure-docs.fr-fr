---
title: Créer un hub Azure IoT avec Azure IoT Tools pour VS Code | Microsoft Docs
description: Découvrez comment utiliser Azure IoT Tools pour Visual Studio Code afin de créer un hub Azure IoT dans un groupe de ressources.
author: formulahendry
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/04/2019
ms.author: junhan
ms.openlocfilehash: 668087ae596688e86b7b84a16bc5c0fd0f9fcef6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75912248"
---
# <a name="create-an-iot-hub-using-the-azure-iot-tools-for-visual-studio-code"></a>Créer un hub Azure IoT avec Azure IoT Tools pour Visual Studio Code

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

Cet article montre comment utiliser [Azure IoT Tools pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) afin de créer un hub Azure IoT. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des éléments suivants :

- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

- [Visual Studio Code](https://code.visualstudio.com/)

- [Outils Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) pour Visual Studio Code.

## <a name="create-an-iot-hub"></a>Créer un hub IoT

1. Dans Visual Studio Code, ouvrez la vue **Explorateur**.

2. En bas de l’Explorateur, développez la section **Appareils Azure IoT Hub**. 

   ![Développer Appareils Azure IoT Hub](./media/iot-hub-create-use-iot-toolkit/azure-iot-hub-devices.png)

3. Cliquez sur **...** dans l’en-tête de section **Appareils Azure IoT Hub**. Si vous ne voyez pas les points de suspension, pointez sur l’en-tête. 

4. Choisissez **Créer un hub IoT**.

5. Une fenêtre contextuelle s’affiche en bas à droite pour vous permettre de vous connecter à Azure pour la première fois.

6. Sélectionnez un abonnement Azure. 

7. Sélectionnez un groupe de ressources.

8. Sélectionnez un emplacement.

9. Sélectionnez un niveau tarifaire.

10. Entrez un nom global unique pour votre hub IoT.

11. Attendez quelques minutes que le hub IoT soit créé.

## <a name="next-steps"></a>Étapes suivantes

Nous avons maintenant déployé un hub Azure IoT avec Azure IoT Tools pour Visual Studio Code. Pour en savoir plus, consultez les articles suivants :

* [Utiliser Azure IoT Tools pour Visual Studio Code afin d’envoyer et de recevoir des messages entre votre appareil et un hub IoT](iot-hub-vscode-iot-toolkit-cloud-device-messaging.md).

* [Utiliser Azure IoT Tools pour Visual Studio Code pour la gestion des appareils Azure IoT Hub](iot-hub-device-management-iot-toolkit.md)

* [Consultez la page wiki Azure IoT Hub pour VS Code](https://github.com/microsoft/vscode-azure-iot-toolkit/wiki).
