---
title: Visualisation des données en temps réel depuis Azure IoT Hub – Power BI
description: Power BI permet d’afficher des données sur les températures et l’humidité collectées par le capteur et envoyées à votre instance Azure IoT Hub.
author: robinsh
keywords: visualisation de données en temps réel, visualisation de données en direct, visualisation de données de capteurs
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 6/08/2020
ms.author: robinsh
ms.openlocfilehash: 6d10d0202e9e4c813cffe6373acfb5200ebb3266
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92146781"
---
# <a name="visualize-real-time-sensor-data-from-azure-iot-hub-using-power-bi"></a>Visualiser des données de capteur en temps réel depuis Azure IoT Hub, à l’aide de Power BI

![Diagramme de bout en bout](./media/iot-hub-live-data-visualization-in-power-bi/end-to-end-diagram.png)

[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

## <a name="what-you-learn"></a>Contenu

Vous apprenez à visualiser les données de capteur en temps réel que votre instance Azure IoT Hub reçoit par l’intermédiaire de Power BI. Si vous souhaitez essayer de visualiser les données dans votre hub IoT avec une application web, consultez [Utiliser une application web pour visualiser les données de capteur en temps réel à partir d’Azure IoT Hub](iot-hub-live-data-visualization-in-web-apps.md).

## <a name="what-you-do"></a>Procédure

* Préparation de votre instance IoT Hub pour l’accès aux données via l’ajout d’un groupe de consommateurs.

* Création, configuration et exécution d’une tâche Stream Analytics pour le transfert de données de votre instance IoT Hub vers votre compte Power BI.

* Création et publication d’un rapport Power BI pour visualiser les données.

## <a name="what-you-need"></a>Ce dont vous avez besoin

* Suivre le tutoriel [Simulateur en ligne Raspberry Pi](iot-hub-raspberry-pi-web-simulator-get-started.md) ou un des tutoriels de l’appareil, par exemple [Raspberry Pi avec node.js](iot-hub-raspberry-pi-kit-node-get-started.md). Ces articles demandent les éléments suivants :
  
  * Un abonnement Azure actif.
  * Une instance Azure IoT Hub associée à votre abonnement.
  * Une application cliente qui envoie des messages à votre instance Azure IoT Hub.

* Un compte Microsoft Power BI. ([Essayez Power BI gratuitement](https://powerbi.microsoft.com/)).

[!INCLUDE [iot-hub-get-started-create-consumer-group](../../includes/iot-hub-get-started-create-consumer-group.md)]

## <a name="create-configure-and-run-a-stream-analytics-job"></a>Créer, configurer et exécuter un travail Stream Analytics

Commençons par la création d’une tâche Stream Analytics. Une fois la tâche créée, vous définissez les entrées, les sorties et la requête utilisée pour récupérer les données.

### <a name="create-a-stream-analytics-job"></a>Création d’un travail Stream Analytics

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** > **Internet des objets** > **Tâche Stream Analytics**.

2. Saisissez les informations ci-après concernant le travail.

   **Nom du travail** : Nom du travail. Le nom doit être globalement unique.

   **Groupe de ressources** : utilisez le même groupe de ressources que celui de votre hub IoT.

   **Emplacement** : utilisez le même emplacement que votre groupe de ressources.

   ![Créer un travail Stream Analytics dans Azure](./media/iot-hub-live-data-visualization-in-power-bi/create-stream-analytics-job.png)

3. Sélectionnez **Create** (Créer).

### <a name="add-an-input-to-the-stream-analytics-job"></a>Ajouter une entrée à la tâche Stream Analytics

1. Ouvrez le travail Stream Analytics.

2. Sous **Topologie de la tâche**, sélectionnez **Entrées**.

3. Dans le volet **Entrées**, sélectionnez **Ajouter une entrée de flux**, puis sélectionnez **IoT Hub** dans la liste déroulante. Dans le volet de la nouvelle entrée, indiquez les informations suivantes :

   **Alias d’entrée** : indiquez un alias unique pour l’entrée.

   **Sélectionner un hub IoT dans vos abonnements** : sélectionnez cette case d’option.

   **Abonnement**: Sélectionnez l’abonnement Azure que vous utilisez pour ce tutoriel.

   **Hub IoT** : sélectionnez le hub IoT que vous utilisez pour ce tutoriel.

   **Point de terminaison** : sélectionnez **Messagerie**.

   **Nom de la stratégie d'accès partagé** : sélectionnez le nom de la stratégie d’accès partagé que vous souhaitez voir utiliser par le travail Stream Analytics. Pour ce tutoriel, vous pouvez sélectionner *service*. La stratégie *service* est créée par défaut sur les nouveaux hubs IoT ; elle accorde l’autorisation d’envoyer et de recevoir sur des points de terminaison côté cloud qui sont exposés par le hub IoT. Pour plus d’informations, consultez [Contrôle d’accès et autorisations](iot-hub-devguide-security.md#access-control-and-permissions).

   **Clé de la stratégie d’accès partagé** : ce champ est automatiquement renseigné par rapport à votre sélection pour le nom de stratégie d’accès partagé.

   **Groupe de consommateurs** : sélectionnez le groupe de consommateurs que vous avez créé précédemment.

   Laissez tous les autres champs sur leurs valeurs par défaut.

   ![Ajouter une entrée à un travail Stream Analytics dans Azure](./media/iot-hub-live-data-visualization-in-power-bi/add-input-to-stream-analytics-job.png)

4. Sélectionnez **Enregistrer**.

### <a name="add-an-output-to-the-stream-analytics-job"></a>Ajouter une sortie à la tâche Stream Analytics

1. Sous **Topologie de la tâche**, sélectionnez **Sorties**.

2. Dans le volet **Sorties**, sélectionnez **Ajouter** et **Power BI**.

3. Dans le volet **Power BI - Nouvelle sortie**, sélectionnez **Autoriser** et suivez les invites pour vous connecter à votre compte Power BI.

4. Une fois connecté à Power BI, entrez les informations suivantes :

   **Alias de sortie** : alias unique de la sortie.

   **Espace de travail de groupe** : sélectionnez l’espace de travail de votre groupe cible.

   **Nom du jeu de données** : entrez un nom de jeu de données.

   **Nom de la table** : Saisissez le nom de la table.

   **Mode d'authentification** : Conservez la valeur par défaut.

   ![Ajouter une sortie à un travail Stream Analytics dans Azure](./media/iot-hub-live-data-visualization-in-power-bi/add-output-to-stream-analytics-job.png)

5. Sélectionnez **Enregistrer**.

### <a name="configure-the-query-of-the-stream-analytics-job"></a>Configurer la requête de la tâche Stream Analytics

1. Sous **Topologie de la tâche**, sélectionnez **Requête**.

2. Remplacez `[YourInputAlias]` par l’alias d’entrée du travail.

3. Remplacez `[YourOutputAlias]` par l’alias de sortie du travail.

   ![Ajouter une requête à un travail Stream Analytics dans Azure](./media/iot-hub-live-data-visualization-in-power-bi/add-query-to-stream-analytics-job.png)

4. Sélectionnez **Enregistrer la requête**.

### <a name="run-the-stream-analytics-job"></a>Exécuter la tâche Stream Analytics

Dans la tâche Stream Analytics, sélectionnez **Vue d’ensemble**, puis **Démarrer** > **Maintenant** > **Démarrer**. Une fois la tâche lancée, l’état correspondant passe de **Arrêté** à **Exécution**.

![Exécuter un travail Stream Analytics dans Azure](./media/iot-hub-live-data-visualization-in-power-bi/run-stream-analytics-job.png)

## <a name="create-and-publish-a-power-bi-report-to-visualize-the-data"></a>Créer et publier un rapport Power BI pour visualiser les données

Les étapes suivantes vous montrent comment créer et publier un rapport à l’aide du service Power BI. Vous pouvez suivre ces étapes, avec quelques modifications, si vous souhaitez utiliser la « nouvelle apparence » dans Power BI. Pour comprendre les différences et savoir comment naviguer dans la « nouvelle apparence », consultez [La « nouvelle apparence » du service Power BI](/power-bi/consumer/service-new-look).

1. Vérifiez que l’exemple d’application s’exécute correctement sur votre appareil. Si ce n’est pas le cas, vous pouvez consulter les didacticiels sous [Configurer votre appareil](./iot-hub-raspberry-pi-kit-node-get-started.md).

2. Connectez-vous à votre compte [Power BI](https://powerbi.microsoft.com/en-us/).

3. Sélectionnez l’espace de travail que vous avez utilisé, **Mon espace de travail**.

4. Sélectionnez **Jeux de données**.

   Vous devez voir apparaître le jeu de données que vous avez indiqué lors de la création de la sortie du travail Stream Analytics.

5. Pour le jeu de données que vous avez créé, sélectionnez **Ajouter un rapport** (première icône à droite du nom du jeu de données).

   ![Créer un rapport Microsoft Power BI](./media/iot-hub-live-data-visualization-in-power-bi/power-bi-create-report.png)

6. Créez un graphique en courbes pour afficher la température en temps réel et au fil du temps.

   1. Dans le volet **Visualisations** de la page de création de rapport, sélectionnez l’icône du graphique en courbes pour ajouter un graphique de ce type.

   2. Sur le volet **Champs**, développez la table que vous avez indiquée lors de la création de la sortie du travail Stream Analytics.

   3. Faites glisser **EventEnqueuedUtcTime** vers **Axe** dans le volet **Visualisations**.

   4. Faites glisser **Température** vers **Valeurs**.

      Un graphique en courbes est créé. L’axe des abscisses affiche la date et l’heure du fuseau horaire UTC. Quant à l’axe des ordonnées, il affiche la température fournie par le capteur.

      ![Ajouter un graphique en courbes sur la température dans un rapport Microsoft Power BI](./media/iot-hub-live-data-visualization-in-power-bi/power-bi-add-temperature.png)

7. Créez un autre graphique en courbes pour afficher l’humidité en temps réel, au fil du temps. Pour ce faire, cliquez sur une partie vide du canevas et suivez la procédure ci-dessus pour placer **EventEnqueuedUtcTime** sur l’axe des abscisses et **Humidité** sur l’axe des ordonnées.

   ![Ajouter un graphique en courbes sur l’humidité dans un rapport Microsoft Power BI](./media/iot-hub-live-data-visualization-in-power-bi/power-bi-add-humidity.png)

8. Sélectionnez **Enregistrer** pour enregistrer le rapport.

9. Sélectionnez **Rapports** dans le volet de gauche, puis sélectionnez le rapport que vous venez de créer.

10. Sélectionnez **Fichier** > **Publier sur le web**.

    ![Sélectionnez publier sur le web pour le rapport de Microsoft Power BI](./media/iot-hub-live-data-visualization-in-power-bi/power-bi-select-publish-to-web.png)

    > [!NOTE]
    > Si vous recevez une notification pour contacter votre administrateur, vous devrez peut-être le faire pour pouvoir créer du code incorporé. La création de code incorporé doit être activée avant que vous puissiez effectuer cette étape.
    >
    > ![Notification vous invitant à contacter votre administrateur](./media/iot-hub-live-data-visualization-in-power-bi/contact-admin.png)

11. Sélectionnez **Créer un code incorporé**, puis sélectionnez **Publier**.

Vous obtenez le lien d’accès au rapport, que vous pouvez partager avec les utilisateurs pour leur permettre d’y accéder, ainsi qu’un extrait de code, qui permet d’intégrer le rapport dans votre blog ou site web.

![Publier un rapport Microsoft Power BI](./media/iot-hub-live-data-visualization-in-power-bi/power-bi-web-output.png)

Microsoft propose également des [applications mobiles Power BI](https://powerbi.microsoft.com/en-us/documentation/powerbi-power-bi-apps-for-mobile-devices/), qui permettent d’afficher les tableaux de bord et rapports Power BI sur votre appareil mobile et d’interagir avec eux.

## <a name="next-steps"></a>Étapes suivantes

Vous avez correctement utilisé Power BI pour visualiser les données de capteur en temps réel, à partir de votre instance Azure IoT Hub.

Pour découvrir une autre façon de visualiser des données depuis Azure IoT Hub, voir [Utiliser une application web pour visualiser les données de capteur en temps réel à partir d’Azure IoT Hub ](iot-hub-live-data-visualization-in-web-apps.md).

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]