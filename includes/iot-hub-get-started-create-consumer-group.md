---
author: robinsh
manager: philmea
ms.author: robinsh
ms.topic: include
ms.date: 05/20/2019
ms.openlocfilehash: fbb4e53e0047b9768a70c01aecfb7f31ae213b3f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96025632"
---
## <a name="add-a-consumer-group-to-your-iot-hub"></a>Ajouter un groupe de consommateurs à votre instance IoT Hub

Les [Groupes de consommateurs](../articles/event-hubs/event-hubs-features.md#event-consumers) fournissent des vues indépendantes dans le flux d’événements qui permettent aux applications et services Azure de consommer indépendamment des données à partir du même point de terminaison Event Hub. Dans cette section, vous ajoutez un groupe de consommateurs à votre point de terminaison intégré utilisé ultérieurement dans ce didacticiel pour extraire les données du point de terminaison.

Pour ajouter un groupe de consommateurs à votre instance IoT Hub, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), ouvrez votre instance IoT Hub.

2. Dans le volet gauche, cliquez sur **Points de terminaison prédéfinis**, sélectionnez **Événements** dans le volet droit, puis entrez un nom sous **Groupes de consommateurs**. Sélectionnez **Enregistrer**.

   ![Créer un groupe de consommateurs dans votre IoT Hub](./media/iot-hub-get-started-create-consumer-group/iot-hub-create-consumer-group-azure.png)