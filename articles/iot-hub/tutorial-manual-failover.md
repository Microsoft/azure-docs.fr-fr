---
title: Basculement manuel d’un Azure IoT hub | Microsoft Docs
description: Découvrez comment effectuer un basculement manuel de votre hub IoT vers une autre région, comment vérifier s’il est réussi, puis comment le retourner à la région d’origine et le revérifier.
author: robinsh
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 07/24/2019
ms.author: robinsh
ms.custom:
- mvc
- mqtt
ms.openlocfilehash: 69a0795b9c299b5113c39ce2c4556573f730e4b7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89013943"
---
# <a name="tutorial-perform-manual-failover-for-an-iot-hub"></a>Tutoriel : Effectuer un basculement manuel pour un hub IoT

Basculement manuel est une fonctionnalité du service IoT Hub qui permet aux clients le[basculement](https://en.wikipedia.org/wiki/Failover) des opérations de leur hub à partir d’une région primaire vers la région Azure associée géographiquement correspondante. Basculement manuel est possible en cas de sinistre régional ou une panne de service étendue. Vous pouvez également effectuer un basculement planifié pour tester vos capacités de récupération d’urgence, mais nous vous recommandons d’utiliser un IoT hub de test plutôt que celui en cours d’exécution en production. La fonctionnalité de basculement manuel est proposée aux clients sans coût supplémentaire pour les hubs IoT créés après le 18 mai 2017.

Dans ce didacticiel, vous allez effectuer les tâches suivantes :

> [!div class="checklist"]
> * Créez un IoT hub, à l’aide du portail Azure. 
> * Effectuer un basculement. 
> * Consultez le hub en cours d’exécution dans l’emplacement secondaire.
> * Effectuer une restauration automatique pour ramener les opérations d’IoT hub à l’emplacement principal. 
> * Confirmer que le hub s’exécute correctement dans l’emplacement approprié.

Pour plus d’informations sur le basculement manuel et le basculement initié par Microsoft avec IoT Hub, consultez [de récupération d’urgence inter-région](iot-hub-ha-dr.md#cross-region-dr).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

* Assurez-vous que le port 8883 est ouvert dans votre pare-feu. L'exemple d’appareil de ce tutoriel utilise le protocole MQTT qui communique sur le port 8883. Dans certains environnements réseau professionnels et scolaires, ce port peut être bloqué. Pour plus d’informations sur les différentes façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Créer un hub IoT

1. Connectez-vous au [portail Azure](https://portal.azure.com). 

2. Cliquez sur **+ Créer une ressource** et sélectionnez **Internet des objets**, puis **IoT Hub**.

   ![Capture d’écran montrant la création d’un IoT hub](./media/tutorial-manual-failover/create-hub-01.png)

3. Sélectionnez l’onglet **Fonctions de base**. Complétez les champs suivants.

    **Abonnement** : sélectionnez l’abonnement Azure que vous souhaitez utiliser.

    **Groupe de ressources** : cliquez sur **Créer nouveau**  et spécifiez **ManIFailRG** pour le nom du groupe de ressources.

    **Région** : sélectionnez une région proche de vous. Ce didacticiel utilise `West US 2`. Un basculement ne peut être effectué entre des régions Azure géographiquement associées. La région géographiquement associée à USA Ouest 2 est WestCentralUS.
    
   **Nom de IoT Hub** : spécifiez un nom pour votre IoT Hub. Le nom du hub doit être globalement unique. 

   ![Capture d’écran montrant le volet d’informations de base pour la création d’un IoT Hub](./media/tutorial-manual-failover/create-hub-02-basics.png)

   Cliquez sur **Vérifier + créer**. (Il utilise les valeurs par défaut pour la taille et échelle.) 

4. Passez en revue les informations, puis cliquez sur **Créer** pour créer l’IoT hub. 

   ![Capture d’écran montrant l’étape finale pour la création d’un IoT Hub](./media/tutorial-manual-failover/create-hub-03-create.png)

## <a name="perform-a-manual-failover"></a>Effectuer un basculement manuel

Notez qu’il existe une limite de deux basculements et deux restaurations par jour pour un IoT Hub.

1. Cliquez sur **Groupes de ressources** et sélectionnez le groupe de ressources **ManlFailRG**. Dans la liste des ressources, cliquez sur votre IoT Hub. 

1. Sous **Paramètres** dans le volet IoT Hub, cliquez sur **Basculement**.

   ![Capture d’écran montrant le volet de propriétés de IoT Hub](./media/tutorial-manual-failover/trigger-failover-01.png)

1. Dans le volet Basculement manuel sont affichés l’**Emplacement actuel** et l’**Emplacement de basculement**. L’emplacement actuel indique toujours l’emplacement où le hub est actuellement actif. L’emplacement de basculement est la [région Azure géographiquement associée](../best-practices-availability-paired-regions.md) standard qui est associé à l’emplacement actuel. Vous ne pouvez pas modifier les valeurs de l’emplacement. Pour ce tutoriel, l’emplacement actuel est `West US 2` et l’emplacement de basculement est `West Central US`.

   ![Capture d’écran montrant le volet de basculement manuel](./media/tutorial-manual-failover/trigger-failover-02.png)

1. En haut du volet Basculement manuel, cliquez sur **Démarrer le basculement**. 

1. Dans le volet de confirmation, spécifiez le nom de votre hub IoT pour vérifier que c’est bien celui que vous souhaitez basculer. Ensuite, pour lancer le basculement, cliquez sur **Basculement**.

   La quantité de temps nécessaire pour effectuer le basculement manuel est proportionnelle au nombre d’appareils qui sont inscrits pour votre hub. Par exemple, si vous avez 100 000 appareils,cela peut prendre 15 minutes, mais si vous avez cinq millions d’appareils, cela peut prendre une heure ou plus.

   ![Capture d’écran montrant le volet de confirmation](./media/tutorial-manual-failover/trigger-failover-03-confirm.png)

   Pendant l’exécution du processus de basculement manuel, une bannière s’affiche pour vous indiquer qu’un basculement manuel est en cours. 

   ![Capture d’écran montrant le volet de Basculement manuel en cours](./media/tutorial-manual-failover/trigger-failover-04-in-progress.png)

   Si vous fermez le volet IoT Hub et que vous le rouvrez en cliquant dessus dans le volet Groupe de ressources, une bannière s’affiche pour vous indiquer que le hub est en cours de basculement manuel. 

   ![Capture d’écran montrant le basculement de hub IoT en cours](./media/tutorial-manual-failover/trigger-failover-05-hub-inactive.png)

   Une fois terminé, les régions actuelle et de basculement dans la page Basculement manuel sont permutées, et le hub est à nouveau actif. Dans cet exemple, l’emplacement actuel est maintenant `WestCentralUS` et l’emplacement de basculement est maintenant `West US 2`. 

   ![Capture d’écran montrant que le basculement est terminé](./media/tutorial-manual-failover/trigger-failover-06-finished.png)

   La page de vue d’ensemble affiche également une bannière indiquant que le basculement est terminé et que le hub IoT s’exécute dans `West Central US`.

   ![Capture d’écran montrant que le basculement est terminé dans la page de vue d’ensemble](./media/tutorial-manual-failover/trigger-failover-06-finished-overview.png)


## <a name="perform-a-failback"></a>Effectuer une restauration automatique 

Une fois que vous avez effectué un basculement manuel, vous pouvez basculer des opérations de hub vers la région primaire d’origine, il s’agit d’une restauration automatique. Si vous venez d’exécuter un basculement, vous devez patienter environ une heure avant de pouvoir déposer une requête de restauration automatique. Si vous essayez d’effectuer la restauration automatique dans un délai plus court, un message d’erreur s’affichera.

Une restauration automatique est effectuée à l’instar d’un basculement manuel. Voici les étapes : 

1. Pour effectuer une restauration automatique, retournez dans le volet de Iot Hub pour votre Iot Hub.

2. Sous **Paramètres** dans le volet IoT Hub, cliquez sur **Basculement**. 

3. En haut du volet Basculement manuel, cliquez sur **Démarrer le basculement**. 

4. Dans le volet de confirmation, spécifiez le nom de votre hub IoT pour vérifier que c’est bien celui que vous souhaitez restaurer automatiquement. Pour ensuite lancer la restauration automatique, cliquez sur OK. 

   ![Capture d’écran de la requête de restauration manuelle](./media/tutorial-manual-failover/trigger-failover-03-confirm.png)

   Les bannières sont affichées comme expliqué dans la section Effectuer un basculement. Une fois la restauration automatique terminée, il affiche à nouveau `West US 2` comme emplacement actuel et `West Central US` comme emplacement de basculement, comme défini à l’origine.

## <a name="clean-up-resources"></a>Nettoyer les ressources 

Pour supprimer toutes les ressources que vous avez créées, supprimez le groupe de ressources. Cette opération supprime toutes les ressources contenues dans le groupe. Dans le cas présent, l’Iot hub et le groupe de ressources sont supprimés. 

1. Cliquez sur **Groupes de ressources**. 

2. Recherchez et sélectionnez le groupe de ressource **ManlFailRG**. Cliquez sur le compte pour l’ouvrir. 

3. Cliquez sur **Supprimer le groupe de ressources**. Lorsque vous y êtes invité, entrez le nom du groupe de ressources et cliquez sur **Supprimer** pour confirmer. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris comment configurer et effectuer un basculement manuel, et comment déposer une requête de restauration automatique en effectuant les tâches suivantes :

> [!div class="checklist"]
> * Créez un IoT hub, à l’aide du portail Azure. 
> * Effectuer un basculement. 
> * Consultez le hub en cours d’exécution dans l’emplacement secondaire.
> * Effectuer une restauration automatique pour ramener les opérations d’IoT hub à l’emplacement principal. 
> * Confirmer que le hub s’exécute correctement dans l’emplacement approprié.

Passez au didacticiel suivant pour découvrir comment gérer l’état d’un appareil IoT. 

> [!div class="nextstepaction"]
> [Gérer l’état d’un appareil IoT](tutorial-device-twins.md)
