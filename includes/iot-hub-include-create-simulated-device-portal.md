---
title: Fichier Include
description: Fichier Include
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 02/26/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 8d8f8021925ac9c9e427dd6571ecaa1a30c85497
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177685"
---
<!-- This is the instructions for creating a simulated device you can use for testing routing.-->

Ensuite, créez une identité d’appareil et enregistrez sa clé pour une utilisation ultérieure. Cette identité d’appareil est utilisée par l’application de simulation pour envoyer des messages à IoT Hub. Cette fonctionnalité n’est pas disponible dans PowerShell ou quand vous utilisez un modèle Azure Resource Manager. Les étapes suivantes vous expliquent comment créer l’appareil simulé à partir du [portail Azure](https://portal.azure.com).

1. Ouvrez le [portail Azure](https://portal.azure.com) et connectez-vous à votre compte Azure.

2. Sélectionnez **Groupes de ressources**, puis choisissez votre groupe de ressources. Ce didacticiel utilise **ContosoResources**.

3. Dans la liste des ressources, sélectionnez votre hub IoT. Ce didacticiel utilise **ContosoTestHub**. Sélectionnez **Appareils IoT** dans le volet Hub.

4. Sélectionnez **Ajouter**. Dans le volet Ajouter un appareil, indiquez l’ID d’appareil. Ce didacticiel utilise **Contoso-Test-Device**. N’indiquez pas de clés et cochez l’option **Générer automatiquement les clés**. Vérifiez que l’option **Connecter l’appareil à IoT Hub** est activée. Sélectionnez **Enregistrer**.

   ![Écran d’ajout d’appareil](./media/iot-hub-include-create-simulated-device-portal/add-device.png)

5. Maintenant que l’appareil est créé, sélectionnez-le pour voir les clés générées. Sélectionnez l’icône de copie de la clé primaire et enregistrez-la quelque part (dans le Bloc-notes par exemple) pour l’utiliser lors de la phase de test de ce tutoriel.

   ![Détails de l’appareil, notamment les clés](./media/iot-hub-include-create-simulated-device-portal/device-details.png)