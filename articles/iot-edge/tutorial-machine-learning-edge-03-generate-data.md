---
title: 'Tutoriel : Générer des données d’appareil simulé - Machine Learning sur Azure IoT Edge'
description: 'Tutoriel : Créer des appareils virtuels qui génèrent des données de télémétrie simulées utilisables par la suite pour entraîner un modèle Machine Learning.'
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 1/20/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: eef5e60b06eedb1fb07c57aa2e369dd3830fcad5
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932402"
---
# <a name="tutorial-generate-simulated-device-data"></a>Tutoriel : Générer des données d’appareil simulées

Dans cet article, nous utilisons des données d’entraînement de Machine Learning pour simuler un appareil qui envoie des données de télémétrie à Azure IoT Hub. Comme indiqué dans l’introduction, ce tutoriel utilise le [Turbofan engine degradation simulation data set](https://c3.nasa.gov/dashlink/resources/139/) (jeu de données de simulation de dégradation d’un turboréacteur à double flux) pour simuler les données d’un ensemble de moteurs d’avion à des fins d’entraînement et de test.

Dans notre scénario expérimental, nous savons que :

* Les données se composent de plusieurs séries chronologiques multidimensionnelles.
* Chaque jeu de données est divisé en sous-ensembles d’entraînement et de test.
* Chaque série chronologique provient d’un moteur différent.
* Chaque moteur a un niveau d’usure initial différent et des variations de fabrication.

Pour ce tutoriel, nous utilisons le sous-ensemble de données d’entraînement d’un seul jeu de données (FD003).

En réalité, chaque moteur est un appareil IoT indépendant. Comme nous supposons que vous n’avez pas de groupe de turboréacteurs connectés à Internet sous la main, nous utilisons un logiciel pour les remplacer.

Le simulateur est un programme C# qui utilise les API IoT Hub pour inscrire par programmation des appareils virtuels dans IoT Hub. Ensuite, nous lisons les données de chaque appareil à partir du sous-ensemble de données fournies par la NASA et les envoyons à votre hub IoT à l’aide d’un appareil IoT simulé. Tout le code de cette partie du tutoriel se trouve dans le répertoire DeviceHarness du dépôt.

Le projet DeviceHarness est un projet .NET Core écrit en C# divisé en quatre classes :

* **Program :** Point d’entrée permettant au responsable d’exécution de gérer l’entrée utilisateur et la coordination générale.
* **TrainingFileManager** : chargé de la lecture et de l’analyse du fichier de données sélectionné.
* **CycleData** : représente une seule ligne de données dans un fichier convertie en message.
* **TurbofanDevice** : chargé de créer un appareil IoT qui correspond à un seul appareil (série chronologique) dans les données et de transmettre les données à IoT Hub.

L’exécution des tâches décrites dans cet article doit prendre environ 20 minutes.

L’équivalent du travail de cette étape dans le monde réel est généralement exécutée par des développeurs d’appareil et des développeurs du cloud.

Dans cette section du tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Intégrer un projet externe à votre environnement de développement.
> * Utiliser l’exemple de projet DeviceHarness pour générer des données d’appareil IoT simulé.
> * Examiner les données générées dans votre hub IoT.

## <a name="prerequisites"></a>Prérequis

Cet article fait partie d’une série décrivant l’utilisation d’Azure Machine Learning sur IoT Edge. Chaque article de la série s’appuie sur le travail réalisé dans le cadre de l’article précédent. Si vous êtes tombé directement sur cet article, reportez-vous au [premier article](tutorial-machine-learning-edge-01-intro.md) de la série.

## <a name="configure-visual-studio-code-and-build-deviceharness-project"></a>Configurer Visual Studio Code et générer le projet DeviceHarness

1. Ouvrez une session Bureau à distance sur votre machine virtuelle de développement.

1. Dans Visual Studio Code, ouvrez le dossier `C:\source\IoTEdgeAndMlSample\DeviceHarness`.

1. Comme vous utilisez ces extensions sur cette machine pour la première fois, certaines extensions se mettent à jour et installent leurs dépendances. Vous pouvez être invité à mettre à jour l’extension. Dans ce cas, sélectionnez **Recharger la fenêtre**.

   Si des erreurs OmniSharp apparaissent dans la fenêtre de sortie, vous devez désinstaller l’extension C#.

1. Vous êtes inviter à ajouter les ressources dont DeviceHarness a besoin. Sélectionnez **Oui** pour les ajouter.

   * La notification peut prendre quelques secondes à s’afficher.
   * Si vous avez manqué cette notification, consultez l’icône de cloche en bas à droite.

   ![Fenêtre contextuelle de l’extension VS Code](media/tutorial-machine-learning-edge-03-generate-data/add-required-assets.png)

1. Sélectionnez **Restaurer** pour restaurer les dépendances de package.

   ![Invite de restauration de VS Code](media/tutorial-machine-learning-edge-03-generate-data/restore-package-dependencies.png)

   Si vous ne recevez pas ces notifications, fermez Visual Studio Code, supprimez les répertoires bin et obj dans `C:\source\IoTEdgeAndMlSample\DeviceHarness`, ouvrez Visual Studio Code, puis rouvrez le dossier DeviceHarness.

1. Vérifiez que votre environnement est correctement configuré en déclenchant une build : **Ctrl** + **Maj** + **B** ou **Terminal** > **Exécuter la tâche de génération**.

1. Vous êtes invité à sélectionner la tâche de build à exécuter. Sélectionnez **Build**.

1. La build s’exécute et génère un message de réussite.

   ![Message de sortie de réussite de la build](media/tutorial-machine-learning-edge-03-generate-data/build-success.png)

1. Vous pouvez définir cette build comme build par défaut en sélectionnant **Terminal** > **Configurer une tâche de build par défaut...** et en choisissant **Build** à partir de l’invite.

## <a name="connect-to-iot-hub-and-run-deviceharness"></a>Se connecter à IoT Hub et exécuter DeviceHarness

Maintenant que nous avons la génération de projet, connectez-vous à votre hub IoT pour accéder à la chaîne de connexion et superviser la progression de la génération des données.

### <a name="sign-in-to-azure-in-visual-studio-code"></a>Se connecter à Azure dans Visual Studio Code

1. Connectez-vous à votre abonnement Azure dans Visual Studio Code en ouvrant la palette de commandes : `Ctrl + Shift + P` ou **Afficher** > **Palette de commandes**.

1. Recherchez la commande **Azure : Se connecter**.

   Une fenêtre de navigateur s’ouvre et vous invite à entrer vos informations d’identification. Dès que vous êtes redirigé vers une page de réussite, vous pouvez fermer le navigateur.

### <a name="connect-to-your-iot-hub-and-retrieve-hub-connection-string"></a>Se connecter à votre hub IoT et récupérer la chaîne de connexion du hub

1. Dans la section inférieure de l’Explorateur Visual Studio Code, sélectionnez le cadre **Azure IoT Hub** pour le développer.

1. Dans le cadre développé, cliquez sur **Sélectionner un hub IoT**.

1. Quand vous y êtes invité, sélectionnez votre abonnement Azure, puis votre hub IoT.

1. Cliquez sur **...** à droite de **Azure IoT Hub** pour accéder à d’autres actions. Sélectionnez **Copier une chaîne de connexion IoT Hub**.

   ![Copier une chaîne de connexion IoT Hub](media/tutorial-machine-learning-edge-03-generate-data/copy-hub-connection-string.png)

### <a name="run-the-deviceharness-project"></a>Exécuter le projet DeviceHarness

1. Sélectionnez **Afficher** > **Terminal** pour ouvrir le terminal Visual Studio Code.

   Si vous ne voyez pas d’invite, appuyez sur Entrée.

1. Entrez `dotnet run` dans le terminal.

1. Quand vous êtes invité à entrer la chaîne de connexion IoT Hub, collez la chaîne de connexion copiée dans la section précédente.

1. Dans le cadre **Appareils Azure IoT Hub**, cliquez sur le bouton Actualiser.

   ![Actualiser la liste des appareils IoT Hub](media/tutorial-machine-learning-edge-03-generate-data/refresh-hub-device-list.png)

1. Notez que les appareils sont ajoutés au hub IoT et qu’ils s’affichent en vert pour indiquer qu’ils envoient des données. Après l’envoi de messages au hub IoT, les appareils se déconnectent et apparaissent en bleu.

1. Vous pouvez voir les messages envoyés au hub en cliquant avec le bouton droit sur un appareil et en sélectionnant **Démarrer la supervision du point de terminaison d’événement intégré**. Les messages s’affichent dans le volet de sortie dans Visual Studio Code.

1. Arrêtez la supervision en cliquant dans le volet de sortie **Azure IoT Hub** et choisissez **Arrêter la supervision du point de terminaison d’événement intégré**.

1. Laissez l’application s’exécuter jusqu’à la fin, c’est-à-dire pendant quelques minutes.

## <a name="check-iot-hub-for-activity"></a>Rechercher une activité sur le hub IoT

Les données envoyées par DeviceHarness ont été acheminées à votre hub IoT, ce que vous pouvez vérifier dans le portail Azure.

1. Ouvrez le [portail Azure](https://portal.azure.com/) et accédez au hub IoT créé pour ce tutoriel.

1. Dans le menu du volet de gauche, sous **Supervision**, sélectionnez **Métriques**.

1. Dans la page définition du graphique, cliquez sur la liste déroulante **Métrique**, faites défiler la liste et sélectionnez **Routing: data delivered to storage** (Routage : données remises au stockage). Le graphique doit montrer le pic correspondant au routage des données vers le stockage.

   ![Graphique montrant le pic correspondant au routage des données vers le stockage](media/tutorial-machine-learning-edge-03-generate-data/iot-hub-usage.png)

## <a name="validate-data-in-azure-storage"></a>Valider les données dans le Stockage Azure

Les données que nous venons d’envoyer à votre hub IoT ont été routées vers le conteneur de stockage que nous avons créé dans l’article précédent. Examinons les données dans notre compte de stockage.

1. Dans le portail Azure, accédez à votre compte de stockage.

1. Dans le navigateur du compte de stockage, sélectionnez **Explorateur Stockage (préversion)** .

1. Dans l’explorateur de stockage, sélectionnez **Conteneurs d’objets blob**, puis `devicedata`.

1. Dans le volet de contenu, cliquez sur le dossier avec le nom du hub IoT suivi de l’année, du mois, du jour et de l’heure. Vous voyez plusieurs dossiers correspondant aux minutes auxquelles les données ont été écrites.

   ![Voir les dossiers dans le stockage blob](media/tutorial-machine-learning-edge-03-generate-data/confirm-data-storage-results.png)

1. Cliquez sur un de ces dossiers pour rechercher les fichiers de données étiquetés **00** et **01** correspondant à la partition.

1. Les fichiers sont écrits au format [Avro](https://avro.apache.org/). Double-cliquez sur l’un de ces fichiers pour ouvrir un autre onglet de navigateur et afficher partiellement les données. Si vous êtes invité à ouvrir le fichier dans un programme, vous pouvez choisir VS Code pour qu’il s’affiche correctement.

1. Vous n’avez pas besoin d’essayer de lire ou d’interpréter les données pour le moment, ce sera l’objet de l’article suivant.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Ce tutoriel fait partie d’un ensemble où chaque article s’appuie sur le travail effectué dans les articles précédents. Ne nettoyez pas les ressources avant d’avoir terminé le dernier tutoriel.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, nous avons utilisé un projet .NET Core pour créer un ensemble d’appareils IoT virtuels et envoyer des données par le biais de ces appareils à notre hub IoT et dans un conteneur Stockage Azure. Ce projet simule un scénario réel dans lequel des appareils IoT physiques envoient des données à un hub IoT, puis dans un stockage organisé. Ces données comprennent des relevés de capteur, des paramètres opérationnels, des signaux et modes d’échec, etc. Une fois que nous avons collecté suffisamment de données, nous les utilisons pour entraîner des modèles qui prédisent la durée de vie restante (RUL) de l’appareil. Ce modèle Machine Learning est décrit dans le prochain article.

Passez à l’article suivant pour entraîner un modèle Machine Learning avec les données.

> [!div class="nextstepaction"]
> [Entraîner et déployer un modèle Azure Machine Learning](tutorial-machine-learning-edge-04-train-model.md)
