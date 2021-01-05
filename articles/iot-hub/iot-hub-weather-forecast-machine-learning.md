---
title: Prévisions météo à l’aide d’Azure Machine Learning studio (classique) avec des données IoT Hub
description: Utilisez Azure Machine Learning studio (classique) pour prédire le risque de pluie sur la base des données de température et d’humidité collectées par votre hub IoT à partir d’un capteur.
author: robinsh
manager: philmea
keywords: Prévisions météo avec Machine Learning
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 09/16/2020
ms.author: robinsh
ms.openlocfilehash: ab9e122ba0b2b50203a2d66ae14f03f3b6300f96
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452343"
---
# <a name="weather-forecast-using-the-sensor-data-from-your-iot-hub-in-azure-machine-learning-studio-classic"></a>Prévision météo à l’aide des données de capteur d’un hub IoT Hub dans Azure Machine Learning studio (classique)

![Diagramme de bout en bout](media/iot-hub-get-started-e2e-diagram/6.png)

[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

L’apprentissage automatique (Machine Learning) utilise des ordinateurs pour exécuter des modèles prédictifs qui apprennent à partir de données existantes afin de prévoir les tendances, résultats et comportements futurs. Azure Machine Learning Studio (classique) est un service d’analyse prédictive cloud qui permet de créer et de déployer rapidement des modèles prédictifs sous forme de solutions d’analyse.

## <a name="what-you-learn"></a>Contenu

Vous allez apprendre à utiliser Azure Machine Learning studio (classique) pour effectuer des prévisions météo (risque de pluie) à l’aide des données de température et d’humidité de votre hub IoT Azure. Les chances de sortie sont la sortie d’un modèle de prévision météo préparé. Le modèle est basé sur des données historiques pour prévoir les chances de pluie en fonction de la température et de l’humidité.

## <a name="what-you-do"></a>Procédure

- Déployez le modèle de prévision météo comme un service web.
- Préparation de votre instance IoT Hub pour l’accès aux données via l’ajout d’un groupe de consommateurs.
- Créez un travail Stream Analytics et configurez-le pour :
  - Lire les données de température et d’humidité en temps réel à partir de votre IoT Hub.
  - Appelez le service web pour obtenir les chances de pluie.
  - Enregistrer les résultats dans un stockage d'objets blob Azure.
- Utilisez Microsoft Azure Storage Explorer pour afficher les prévisions météorologiques.

## <a name="what-you-need"></a>Ce dont vous avez besoin

- Suivre le tutoriel [Simulateur en ligne Raspberry Pi](iot-hub-raspberry-pi-web-simulator-get-started.md) ou un des tutoriels de l’appareil, par exemple [Raspberry Pi avec node.js](iot-hub-raspberry-pi-kit-node-get-started.md). Ceux-ci couvrent les exigences suivantes :
  - Un abonnement Azure actif.
  - Une instance Azure IoT Hub associée à votre abonnement.
  - Une application cliente qui envoie des messages à votre instance Azure IoT Hub.
- Un compte [Azure Machine Learning Studio (classique)](https://studio.azureml.net/).
- Un [compte de stockage Azure](../storage/common/storage-account-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#types-of-storage-accounts). Un **compte universel v2** est idéal, mais n’importe quel compte de stockage Azure prenant en charge le Stockage Blob Azure fait également l’affaire.

> [!Note]
> Cet article utilise Azure Stream Analytics et plusieurs autres services payants. Des frais supplémentaires sont occasionnés dans Azure Stream Analytics lorsque les données doivent être transférées entre différentes régions Azure. C’est pourquoi il est judicieux de s’assurer que votre groupe de ressources, votre hub IoT et votre compte de stockage Azure, ainsi que l’espace de travail Machine Learning studio (classique) et le travail Azure Stream Analytics ajoutés dans la suite de ce tutoriel, se trouvent tous dans la même région Azure. Pour connaître la prise en charge régionale d’Azure Machine Learning studio (classique) et d’autres services Azure, consultez la page [Disponibilité des produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?products=machine-learning-studio&regions=all).

## <a name="deploy-the-weather-prediction-model-as-a-web-service"></a>Déployer le modèle de prévision météo comme un service web

Dans cette section, vous obtenez le modèle de prédiction météorologique de la bibliothèque Azure AI. Vous ajoutez ensuite un module R-script au modèle pour nettoyer les données de température et d’humidité. Enfin, vous déployez le modèle en tant que service web prédictif.

### <a name="get-the-weather-prediction-model"></a>Obtenir le modèle de prédiction météorologique

Dans cette section, vous obtenez le modèle de prédiction météorologique d’Azure AI Gallery, et vous l’ouvrez dans Azure Machine Learning Studio (classique).

1. Accédez à la [page de modèle de prévision météo](https://gallery.cortanaintelligence.com/Experiment/Weather-prediction-model-1).

   ![Ouvrir la page du modèle de prédiction météorologique dans Azure AI Gallery](media/iot-hub-weather-forecast-machine-learning/weather-prediction-model-in-azure-ai-gallery.png)

1. Sélectionnez **Ouvrir dans studio (classique)** pour ouvrir le modèle dans Microsoft Azure Machine Learning studio (classique). Sélectionnez une région près de votre hub IoT et l’espace de travail approprié dans la fenêtre contextuelle **Copier l’expérience à partir de la galerie**.

   ![Ouvrir le modèle de prédiction météorologique dans Azure Machine Learning Studio (classique)](media/iot-hub-weather-forecast-machine-learning/open-ml-studio.png)

### <a name="add-an-r-script-module-to-clean-temperature-and-humidity-data"></a>Ajouter un module de script R pour nettoyer les données de température et d’humidité

Pour que le modèle se comporte correctement, les données de température et d’humidité doivent être convertibles en données numériques. Dans cette section, vous ajoutez un module de script R au modèle de prédiction météorologique pour supprimer toutes les lignes contenant des valeurs de données de température ou d’humidité non convertibles en valeurs numériques.

1. Sur le côté gauche de la fenêtre Azure Machine Learning studio (classique), sélectionnez la flèche pour développer le panneau Outils. Entrez « Exécuter » dans la zone de recherche. Sélectionnez le module **Exécuter un script R**.

   ![Sélectionner le module Exécuter un script R](media/iot-hub-weather-forecast-machine-learning/select-r-script-module.png)

1. Faites glisser le module **Exécuter un script R** près du module **Clean Missing Data** (Nettoyer les données manquantes) et du module **Exécuter un script R** existant sur le diagramme. Supprimez la connexion entre les modules **Clean Missing Data** et **Exécuter un script R**, puis connectez les entrées et les sorties du nouveau module comme indiqué.

   ![Ajouter un module Exécuter un script R](media/iot-hub-weather-forecast-machine-learning/add-r-script-module.png)

1. Sélectionnez le nouveau module **Exécuter un script R** pour ouvrir sa fenêtre Propriétés. Copiez le code suivant, puis collez-le dans la zone **Script R**.

   ```r
   # Map 1-based optional input ports to variables
   data <- maml.mapInputPort(1) # class: data.frame

   data$temperature <- as.numeric(as.character(data$temperature))
   data$humidity <- as.numeric(as.character(data$humidity))

   completedata <- data[complete.cases(data), ]

   maml.mapOutputPort('completedata')

   ```

   Une fois que vous avez terminé, la fenêtre Propriétés doit ressembler à ce qui suit :

   ![Ajouter du code au module Exécuter un script R](media/iot-hub-weather-forecast-machine-learning/add-code-to-module.png)

### <a name="deploy-predictive-web-service"></a>Déployer un service web prédictif

Dans cette section, vous validez le modèle, vous configurez un service web prédictif basé sur ce modèle, puis vous déployez le service web.

1. Sélectionnez **Exécuter** pour valider les étapes du modèle. Cette étape peut prendre quelques minutes.

   ![Exécuter l’expérience pour valider les étapes](media/iot-hub-weather-forecast-machine-learning/run-experiment.png)

1. Sélectionnez **CONFIGURER LE SERVICE WEB** > **Service web prédictif**. Le diagramme d’expérience prédictive s’ouvre.

   ![Déployer le modèle de prédiction météorologique dans Azure Machine Learning Studio (classique)](media/iot-hub-weather-forecast-machine-learning/predictive-experiment.png)

1. Dans le diagramme de l’expérience prédictive, supprimez la connexion entre le module **Entrée du service web** et le module **Sélectionner des colonnes dans le jeu de données** en haut. Faites glisser ensuite le module **Web service input** près du module **Score Model** (Modèle de score), et connectez-le comme indiqué :

   ![Connecter deux modules dans Azure Machine Learning Studio (classique)](media/iot-hub-weather-forecast-machine-learning/connect-modules-azure-machine-learning-studio.png)

1. Sélectionnez **EXÉCUTER** pour valider les étapes du modèle.

1. Sélectionnez **DÉPLOYER LE SERVICE WEB** pour déployer le modèle en tant que service web.

1. Sur le tableau de bord du modèle, téléchargez le classeur **Excel 2010 ou version antérieure** pour **DEMANDE/RÉPONSE**.

   > [!Note]
   > Veillez à télécharger le **classeur Excel 2010 ou version antérieure**, même si vous exécutez une version ultérieure d’Excel sur votre ordinateur.

   ![Téléchargement du classeur Excel pour le point de terminaison DEMANDE/RÉPONSE](media/iot-hub-weather-forecast-machine-learning/download-workbook.png)

1. Ouvrez le classeur Excel, notez **l’URL DU SERVICE WEB** et la **CLÉ D’ACCÈS**.

[!INCLUDE [iot-hub-get-started-create-consumer-group](../../includes/iot-hub-get-started-create-consumer-group.md)]

## <a name="create-configure-and-run-a-stream-analytics-job"></a>Créer, configurer et exécuter un travail Stream Analytics

### <a name="create-a-stream-analytics-job"></a>Création d’un travail Stream Analytics

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez **Créer une ressource**. Tapez « travail Stream Analytics » dans la zone de recherche, puis sélectionnez **Travail Stream Analytics** dans la liste déroulante des résultats. Lorsque le volet **Travail Stream Analytics** s’ouvre, sélectionnez **Créer**.
1. Saisissez les informations ci-après concernant le travail.

   **Nom du travail** : Nom du travail. Le nom doit être globalement unique.

   **Abonnement**: sélectionnez votre abonnement s’il est différent de l’abonnement par défaut.

   **Groupe de ressources** : utilisez le même groupe de ressources que celui de votre hub IoT.

   **Emplacement** : utilisez le même emplacement que votre groupe de ressources.

   Laissez la valeur par défaut dans tous les autres champs.

   ![Créer un travail Stream Analytics dans Azure](media/iot-hub-weather-forecast-machine-learning/create-stream-analytics-job.png)

1. Sélectionnez **Create** (Créer).

### <a name="add-an-input-to-the-stream-analytics-job"></a>Ajouter une entrée à la tâche Stream Analytics

1. Ouvrez le travail Stream Analytics.
1. Sous **Topologie de la tâche**, sélectionnez **Entrées**.
1. Dans le volet **Entrées**, sélectionnez **Ajouter une entrée de flux**, puis sélectionnez **IoT Hub** dans la liste déroulante. Dans le volet **Nouvelle entrée**, choisissez **Sélectionnez le hub IoT dans vos abonnements**, puis entrez les informations suivantes :

   **Alias d’entrée** : alias unique de l’entrée.

   **Abonnement**: sélectionnez votre abonnement s’il est différent de l’abonnement par défaut.

   **Hub IoT** : sélectionner le hub IoT dans votre abonnement.

   **Nom de la stratégie d'accès partagé** : sélectionnez un **service**. (Vous pouvez également utiliser **iothubowner**.)

   **Groupe de consommateurs** : sélectionnez le groupe de consommateurs que vous avez créé.

   Laissez la valeur par défaut dans tous les autres champs.

   ![Ajouter une entrée à un travail Stream Analytics dans Azure](media/iot-hub-weather-forecast-machine-learning/add-input-stream-analytics-job.png)

1. Sélectionnez **Enregistrer**.

### <a name="add-an-output-to-the-stream-analytics-job"></a>Ajouter une sortie à la tâche Stream Analytics

1. Sous **Topologie de la tâche**, sélectionnez **Sorties**.
1. Dans le volet **Sorties**, sélectionnez **Ajouter**, puis sélectionnez **Stockage Blob/Data Lake Storage** dans la liste déroulante. Dans le volet **Nouvelle sortie**, choisissez **Sélectionnez le stockage dans vos abonnements**, puis entrez les informations suivantes :

   **Alias de sortie** : alias unique de la sortie.

   **Abonnement**: sélectionnez votre abonnement s’il est différent de l’abonnement par défaut.

   **Compte de stockage** : Le compte de stockage pour votre stockage d’objets blob. Vous pouvez utiliser un compte de stockage existant ou en créer un nouveau.

   **Conteneur** : le conteneur dans lequel l’objet blob est enregistré. Vous pouvez créer un conteneur ou utiliser un conteneur existant.

   **Format de sérialisation de l’événement** : Sélectionnez **CSV**.

   ![Ajouter une sortie à un travail Stream Analytics dans Azure](media/iot-hub-weather-forecast-machine-learning/add-output-stream-analytics-job.png)

1. Sélectionnez **Enregistrer**.

### <a name="add-a-function-to-the-stream-analytics-job-to-call-the-web-service-you-deployed"></a>Ajouter une fonction au travail Stream Analytics pour appeler le service web que vous avez déployé

1. Sous **Topologie du travail**, sélectionnez **Fonctions**.
1. Dans le volet **Fonctions**, sélectionnez **Ajouter**, puis sélectionnez **Azure ML studio** dans la liste déroulante. (Veillez à sélectionner **Azure ML studio** et non **Azure ML service**.) Dans le volet **Nouvelle fonction**, choisissez **Fournir manuellement les paramètres de fonction Azure Machine Learning** et entrez les informations suivantes :

   **Alias de la fonction** : entrez `machinelearning`.

   **URL** : entrez l’URL DU SERVICE WEB que vous avez notée à partir du classeur Excel.

   **Clé** : entrez la CLÉ D’ACCÈS que vous avez notée à partir du classeur Excel.

   ![Ajouter une requête à un travail Stream Analytics dans Azure](media/iot-hub-weather-forecast-machine-learning/add-function-stream-analytics-job.png)

1. Sélectionnez **Enregistrer**.

### <a name="configure-the-query-of-the-stream-analytics-job"></a>Configurer la requête de la tâche Stream Analytics

1. Sous **Topologie de la tâche**, sélectionnez **Requête**.
1. Remplacez le code existant par le code suivant :

   ```sql
   WITH machinelearning AS (
      SELECT EventEnqueuedUtcTime, temperature, humidity, machinelearning(temperature, humidity) as result from [YourInputAlias]
   )
   Select System.Timestamp time, CAST (result.[temperature] AS FLOAT) AS temperature, CAST (result.[humidity] AS FLOAT) AS humidity, CAST (result.[scored probabilities] AS FLOAT ) AS 'probabalities of rain'
   Into [YourOutputAlias]
   From machinelearning
   ```

   Remplacez `[YourInputAlias]` par l’alias d’entrée du travail.

   Remplacez `[YourOutputAlias]` par l’alias de sortie du travail.

1. Sélectionnez **Enregistrer la requête**.

> [!Note]
> Si vous sélectionnez **Tester la requête**, le message suivant s’affiche : Il n’est pas possible de tester les requêtes avec des fonctions de Machine Learning. Modifiez la requête et réessayez. Vous pouvez ignorer ce message sans risque et sélectionner **OK** pour le fermer. Veillez à enregistrer la requête avant de passer à la section suivante.

### <a name="run-the-stream-analytics-job"></a>Exécuter la tâche Stream Analytics

Dans le travail Stream Analytics, sélectionnez **Vue d’ensemble** dans le volet gauche. Ensuite, sélectionnez **Démarrer** > **Maintenant** > **Démarrer**. Une fois la tâche lancée, l’état correspondant passe de **Arrêté** à **Exécution**.

![Exécuter la tâche Stream Analytics](media/iot-hub-weather-forecast-machine-learning/run-stream-analytics-job.png)

## <a name="use-microsoft-azure-storage-explorer-to-view-the-weather-forecast"></a>Utiliser Microsoft Azure Storage Explorer pour afficher les prévisions météorologiques

Exécutez l’application cliente pour commencer la collecte et l’envoi de données de température et d’humidité à votre IoT Hub. Pour chaque message que votre IoT Hub reçoit, le travail Stream Analytics appelle le service web de prévision météorologique pour produire les chances de pluie. Le résultat est ensuite enregistré dans votre stockage d’objets blob Azure. Azure Storage Explorer est un outil que vous pouvez utiliser pour afficher le résultat.

1. [Téléchargez et installez Microsoft Azure Storage Explorer](https://storageexplorer.com/).
1. Ouvrez l’Explorateur de stockage Azure.
1. Connectez-vous à votre compte Azure.
1. Sélectionnez votre abonnement.
1. Sélectionnez votre abonnement > **Comptes de stockage** > votre compte de stockage > **Conteneurs d’objets blob** > votre conteneur.
1. Téléchargez un fichier .csv pour voir le résultat. La dernière colonne enregistre les chances de pluie.

   ![Résultats de prévisions météo avec Azure Machine Learning studio (classique)](media/iot-hub-weather-forecast-machine-learning/weather-forecast-result.png)

## <a name="summary"></a>Résumé

Vous avez utilisé Azure Machine Learning studio (classique) pour connaître le risque de pluie sur la base des données de température et d’humidité reçues par votre hub IoT.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]