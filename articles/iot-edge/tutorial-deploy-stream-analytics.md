---
title: Tutoriel - Stream Analytics en périphérie à l’aide d’Azure IoT Edge
description: Dans ce tutoriel, vous déployez Azure Stream Analytics en tant que module dans un appareil IoT Edge
author: kgremban
ms.author: kgremban
ms.date: 07/29/2020
ms.topic: tutorial
ms.service: iot-edge
ms.custom: mvc
ms.openlocfilehash: 61779527d4b855f4327ad4b77a1e22207a94b8c0
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92048370"
---
# <a name="tutorial-deploy-azure-stream-analytics-as-an-iot-edge-module"></a>Tutoriel : Déployer Azure Stream Analytics en tant que module IoT Edge

De nombreuses solutions IoT utilisent les services d’analytique pour obtenir des insights sur les données envoyées par les appareils IoT au fur et à mesure de leur arrivée dans le cloud. Avec Azure IoT Edge, vous pouvez utiliser cette logique [Azure Stream Analytics](../stream-analytics/index.yml) et la transposer à l’appareil lui-même. Lors du traitement des flux de données de télémétrie en périphérie, vous pouvez réduire la quantité de données chargées et réduire le temps nécessaire pour réagir aux informations actionnables.

Azure IoT Edge et Azure Stream Analytics sont intégrés pour simplifier le développement de vos charges de travail. Vous pouvez créer un travail Azure Stream Analytics dans le portail Azure, puis le déployer en tant que module IoT Edge sans code supplémentaire.  

Azure Stream Analytics offre une syntaxe de requête très structurée pour l’analyse des données, à la fois dans le cloud et sur les appareils IoT Edge. Pour plus d’informations, consultez la [documentation Azure Stream Analytics](../stream-analytics/stream-analytics-edge.md).

Le module Stream Analytics de ce didacticiel calcule la température moyenne sur une fenêtre propagée de 30 secondes. Lorsque cette moyenne atteint 70, le module envoie les données et alerte l’appareil afin de prendre des mesures. Dans ce cas, cette mesure consiste à réinitialiser le capteur de température simulée. Dans un environnement de production, vous pouvez utiliser cette fonctionnalité pour arrêter un ordinateur ou prendre des mesures préventives quand la température atteint des niveaux dangereux.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
>
> * Créer un travail Azure Stream Analytics pour traiter les données à la périphérie.
> * Connecter le nouveau travail Azure Stream Analytics à d’autres modules IoT Edge.
> * Déployer le travail Azure Stream Analytics sur un appareil IoT Edge depuis le portail Azure.

<center>

![Diagramme - Architecture du tutoriel : indexer et déployer un travail ASA](./media/tutorial-deploy-stream-analytics/asa-architecture.png)
</center>

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Un appareil Azure IoT Edge :

* Vous pouvez utiliser une machine virtuelle Azure en tant qu’appareil IoT Edge. Pour cela, suivez les étapes du guide de démarrage rapide pour les appareils [Linux](quickstart-linux.md) ou [Windows](quickstart.md).

Ressources cloud :

* Un niveau gratuit ou standard [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) dans Azure.

## <a name="create-an-azure-stream-analytics-job"></a>Créer un travail Azure Stream Analytics

Dans cette section, vous allez créer un travail Azure Stream Analytics qui effectuera les étapes suivantes :

* Recevoir les données envoyées par votre appareil IoT Edge
* Interroger les données de télémétrie pour connaître les valeurs situées en dehors d’une plage définie
* Prendre des mesures sur l’appareil IoT Edge en fonction des résultats de la requête

### <a name="create-a-storage-account"></a>Créez un compte de stockage.

Lorsque vous créez un travail Azure Stream Analytics pour s’exécuter sur un appareil IoT Edge, il doit être stocké de manière à pouvoir être appelé à partir de l’appareil. Vous pouvez utiliser un compte Stockage Azure existant ou en créer un maintenant.

1. Dans le portail Azure, accédez à **Créer une ressource** > **Stockage** > **Compte de stockage**.

1. Fournissez les valeurs suivantes pour créer votre compte de stockage :

   | Champ | Valeur |
   | ----- | ----- |
   | Abonnement | Choisissez le même abonnement que votre IoT Hub. |
   | Resource group | Nous vous recommandons d’utiliser le même groupe de ressources pour toutes vos ressources de test dans le cadre des démarrages rapides et tutoriels IoT Edge. Par exemple, utilisez **IoTEdgeResources**. |
   | Nom | Fournissez un nom unique pour votre compte de stockage. |
   | Emplacement | Choisissez un emplacement proche de vous. |

1. Conservez les valeurs par défaut des autres champs, puis sélectionnez **Vérifier + créer**.

1. Passez en revue vos paramètres, puis sélectionnez **Créer**.

### <a name="create-a-new-job"></a>Créer une nouvelle tâche

1. Dans le portail Azure, accédez à **Créer une ressource** > **Internet des objets** > **Tâche Stream Analytics**.

1. Fournissez les valeurs suivantes pour créer votre travail :

   | Champ | Valeur |
   | ----- | ----- |
   | Nom du travail | Fournissez un nom pour votre travail. Par exemple, **IoTEdgeJob** |
   | Abonnement | Choisissez le même abonnement que votre IoT Hub. |
   | Resource group | Nous vous recommandons d’utiliser le même groupe de ressources pour toutes les ressources de test que vous créez dans le cadre des démarrages rapides et didacticiels IoT Edge. Par exemple, utilisez **IoTEdgeResources**. |
   | Emplacement | Choisissez un emplacement proche de vous. |
   | Environnement d’hébergement | Sélectionnez **Edge**. |

1. Sélectionnez **Create** (Créer).

### <a name="configure-your-job"></a>Configurer votre travail

Lorsque votre travail Stream Analytics est créé dans le portail Azure, vous pouvez le configurer avec une entrée, une sortie et une requête à exécuter sur les données qui le traverse.

À l’aide des trois éléments d’entrée, de sortie et de requête, cette section crée un travail qui reçoit des données de température de l’appareil IoT Edge. Il analyse ces données dans une fenêtre de 30 secondes consécutives. Si la température moyenne de cette fenêtre dépasse 70 degrés, une alerte est envoyée à l’appareil IoT Edge. Vous allez spécifier exactement d’où les données proviennent et où elles passent dans la section suivante lorsque vous déployez le travail.  

1. Accédez à votre travail Stream Analytics dans le portail Azure.

1. Sous **Topologie de la tâche**, sélectionnez **Entrées** puis **Ajouter une entrée de flux**.

   ![Azure Stream Analytics - Ajouter une entrée](./media/tutorial-deploy-stream-analytics/asa-input.png)

1. Choisissez **Edge Hub** dans la liste déroulante.

1. Dans le volet **Nouvelle entrée**, saisissez **température** comme alias d’entrée.

1. Conservez les valeurs par défaut pour les autres champs et sélectionnez **Enregistrer**.

1. Sous **Topologie de la tâche**, ouvrez **Sorties**, puis sélectionnez **Ajouter**.

   ![Azure Stream Analytics - Ajouter une sortie](./media/tutorial-deploy-stream-analytics/asa-output.png)

1. Choisissez **Edge Hub** dans la liste déroulante.

1. Dans le volet **Nouvelle sortie**, saisissez **alerte** comme alias de sortie.

1. Conservez les valeurs par défaut pour les autres champs et sélectionnez **Enregistrer**.

1. Sous **Topologie de la tâche**, sélectionnez **Requête**.

1. Remplacez le texte par défaut par la requête suivante. Le code SQL envoie une commande de réinitialisation à la sortie d’alerte si la température moyenne de la machine dans une fenêtre de 30 secondes atteint 70 degrés. La commande de réinitialisation a été préprogrammée dans le capteur comme une action qui peut être effectuée.

    ```sql
    SELECT  
        'reset' AS command
    INTO
       alert
    FROM
       temperature TIMESTAMP BY timeCreated
    GROUP BY TumblingWindow(second,30)
    HAVING Avg(machine.temperature) > 70
    ```

1. Sélectionnez **Enregistrer la requête**.

### <a name="configure-iot-edge-settings"></a>Configurer les paramètres IoT Edge

Pour préparer votre travail Stream Analytics à déployer sur un appareil IoT Edge, vous devez associer le travail à un conteneur dans un compte de stockage. Lorsque vous passez au déploiement de votre travail, la définition du travail est exportée vers le conteneur de stockage.

1. Sous **Configurer**, sélectionnez **Paramètres du compte de stockage**, puis **Ajouter un compte de stockage**.

   ![Azure Stream Analytics - Ajouter un compte de stockage](./media/tutorial-deploy-stream-analytics/add-storage-account.png)

1. Dans le menu déroulant, sélectionnez le **compte de stockage** que vous avez créé au début de ce tutoriel.

1. Pour le champ **Conteneur**, sélectionnez **Créer nouveau** et fournissez un nom pour le conteneur de stockage.

1. Sélectionnez **Enregistrer**.

## <a name="deploy-the-job"></a>Déployer la tâche

Vous êtes désormais prêt à déployer le travail Azure Stream Analytics sur votre appareil IoT Edge.

Dans cette section, vous utilisez l’Assistant **Définir des modules** dans le portail Azure pour créer un *manifeste de déploiement*. Un manifeste de déploiement est un fichier JSON qui décrit tous les modules qui seront déployés sur un appareil, les registres de conteneurs qui stockent les images de modules, comment les modules doivent être gérés et comment les modules peuvent communiquer entre eux. Votre appareil IoT Edge récupère son manifeste de déploiement d’IoT Hub, puis utilise les informations qu’il contient pour déployer et configurer tous les modules assignés.

Pour ce didacticiel, vous déployez deux modules. Le premier est **SimulatedTemperatureSensor**, un module qui simule un capteur de température et d’humidité. Le second est votre travail Stream Analytics. Le module de capteur fournit le flux de données que votre requête de travail analysera.

1. Accédez à votre hub IoT dans le portail Azure.

1. Accédez à **IoT Edge**, puis ouvrez la page de détails de votre appareil IoT Edge.

1. Sélectionnez **Définir des modules**.  

1. Si vous avez déployé le module SimulatedTemperatureSensor sur cet appareil, celui-ci peut se remplir automatiquement. Sinon, ajoutez le module en procédant comme suit :

   1. Cliquez sur **Ajouter** et sélectionnez **Module IoT Edge**.
   1. Pour le nom, tapez **SimulatedTemperatureSensor**.
   1. Pour l’URI de l’image, entrez **mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0**.
   1. Laissez les autres paramètres inchangés et sélectionnez **Ajouter**.

1. Ajoutez votre travail Azure Stream Analytics Edge en suivant les étapes :

   1. Cliquez sur **Ajouter** et sélectionnez **Module Azure Stream Analytics**.
   1. Sélectionnez votre abonnement et le travail Azure Stream Analytics Edge que vous avez créé.
   1. Sélectionnez **Enregistrer**.

   Une fois que vous avez enregistré vos modifications, les détails de votre tâche Stream Analytics sont publiés dans le conteneur de stockage que vous avez créé.

1. Lorsque le module Stream Analytics est ajouté à la liste des modules, sélectionnez son nom pour voir comment il est structuré et mettez à jour ses paramètres dans la page **Mettre à jour le module IoT Edge**.

   L’onglet **Paramètres du module** contient l’**URI d’image** qui pointe vers une image Azure Stream Analytics standard. Cette image est utilisée pour chaque module Stream Analytics qui est déployé sur un appareil IoT Edge.

   L’onglet **Paramètres de jumeau de module** présente le JSON qui définit la propriété Azure Stream Analytics (ASA) appelée **ASAJobInfo**. La valeur de cette propriété pointe vers la définition du travail dans votre conteneur de stockage. Cette propriété correspond à la configuration de l’image Stream Analytics avec les informations de votre tâche.

   Par défaut, le module Stream Analytics prend le nom de la tâche sur laquelle il est basé. Vous pouvez modifier le nom du module dans cette page si vous le souhaitez, mais cela n’est pas nécessaire.

1. Sélectionnez **Mettre à jour** ou **Annuler**.

1. Notez le nom de votre module Stream Analytics. Vous en aurez besoin à l’étape suivante. Ensuite, sélectionnez **Suivant : Routes** pour continuer.

1. Sous l’onglet **Routes**, vous définissez la manière dont les messages sont transmis entre les modules et le hub IoT. Les messages sont construits à l’aide de paires nom/valeur. Remplacez le nom et les valeurs `route` et `upstream` par défaut par les paires indiquées dans le tableau suivant, les paires nom/valeur suivantes, en remplaçant les instances de _{moduleName}_ par le nom de votre module Azure Stream Analytics.

    | Nom | Valeur |
    | --- | --- |
    | `telemetryToCloud` | `FROM /messages/modules/SimulatedTemperatureSensor/* INTO $upstream` |
    | `alertsToCloud` | `FROM /messages/modules/{moduleName}/* INTO $upstream` |
    | `alertsToReset` | `FROM /messages/modules/{moduleName}/* INTO BrokeredEndpoint("/modules/SimulatedTemperatureSensor/inputs/control")` |
    | `telemetryToAsa` | `FROM /messages/modules/SimulatedTemperatureSensor/* INTO BrokeredEndpoint("/modules/{moduleName}/inputs/temperature")`|

    Les itinéraires que vous déclarez ici définissent le flux de données via l’appareil IoT Edge. Les données de télémétrie de SimulatedTemperatureSensor sont envoyées à IoT Hub et à l’entrée **temperature** qui a été configurée dans la tâche Stream Analytics. Les messages de sortie d’**alerte** sont envoyés à IoT Hub et au module SimulatedTemperatureSensor pour déclencher la commande de réinitialisation.

1. Sélectionnez **Suivant : Vérifier + créer**.

1. À l’étape **Vérifier + créer**, vous voyez que les informations que vous avez fournies dans l’Assistant ont été converties en manifeste de déploiement JSON. Une fois que vous avez passé en revue le manifeste, sélectionnez **Créer**.

1. Vous êtes redirigé vers la page des détails de l’appareil. Sélectionnez **Actualiser**.  

    Vous devez voir le nouveau module Stream Analytics s’exécuter avec le module de l’agent IoT Edge et le module du hub IoT Edge. Quelques minutes peuvent être nécessaires pour que les informations atteignent votre appareil IoT Edge, puis pour que les nouveaux modules démarrent. Si vous ne voyez pas les modules s’exécuter immédiatement, continuez à actualiser la page.

    ![SimulatedTemperatureSensor et module ASA signalés par l’appareil](./media/tutorial-deploy-stream-analytics/module-output2.png)

## <a name="view-data"></a>Afficher les données

Vous pouvez désormais aller sur votre appareil IoT Edge pour consulter les interactions entre les modules Azure Stream Analytics et SimulatedTemperatureSensor.

1. Vérifiez que tous les modules sont en cours d’exécution dans Docker :

   ```cmd/sh
   iotedge list  
   ```

1. Affichez tous les journaux d’activité système et les données des métriques. Utilisez le nom du module Stream Analytics :

   ```cmd/sh
   iotedge logs -f {moduleName}  
   ```

1. Voyez comment la commande de réinitialisation affecte SimulatedTemperatureSensor en consultant les journaux du capteur :

   ```cmd/sh
   iotedge logs SimulatedTemperatureSensor
   ```

   Vous pouvez voir la température de l’ordinateur augmenter progressivement jusqu’à atteindre 70 degrés pendant 30 secondes. Le module Stream Analytics déclenche alors une réinitialisation qui fait redescendre la température de l’ordinateur à 21.

   ![Réinitialiser la sortie de commande dans les journaux d’activité du module](./media/tutorial-deploy-stream-analytics/docker_log.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de passer à l’article recommandé suivant, vous pouvez conserver les ressources et configurations que vous avez créées afin de les réutiliser. Vous pouvez également continuer à utiliser le même appareil IoT Edge comme appareil de test.

Sinon, vous pouvez supprimer les ressources Azure et les configurations locales que vous avez utilisées dans cet article pour éviter les frais.

[!INCLUDE [iot-edge-clean-up-cloud-resources](../../includes/iot-edge-clean-up-cloud-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez configuré un travail Azure Stream Analytics pour qu’il analyse les données de votre appareil IoT Edge. Ensuite, vous avez chargé ce module Azure Stream Analytics sur votre appareil IoT Edge pour traiter et réagir localement à l’augmentation de température, et pour envoyer le flux de données agrégées dans le cloud. Vous pouvez passer à d’autres didacticiels pour savoir comment Azure IoT Edge peut trouver d’autres solutions pour votre entreprise.

> [!div class="nextstepaction"]
> [Déployer un modèle Azure Machine Learning en tant que module](tutorial-deploy-machine-learning.md)