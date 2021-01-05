---
title: 'Tutoriel : Exemple Alertes visuelles IoT'
titleSuffix: Azure Cognitive Services
description: Dans ce tutoriel, vous utilisez Custom Vision avec un appareil IoT pour reconnaître et signaler des états visuels à partir du flux vidéo d’une caméra.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: tutorial
ms.date: 11/23/2020
ms.author: pafarley
ms.openlocfilehash: c6405e2fcddef9ae3228ede76dfa57f7542164c8
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96020175"
---
# <a name="tutorial-use-custom-vision-with-an-iot-device-to-report-visual-states"></a>Tutoriel : Utiliser Custom Vision avec un appareil IoT pour signaler des états visuels

Cet exemple d’application montre comment utiliser Custom Vision pour effectuer l’apprentissage d’un appareil avec une caméra pour détecter les états visuels. Vous pouvez exécuter ce scénario de détection sur un appareil IoT à l’aide d’un modèle ONNX exporté.

Un état visuel décrit le contenu d’une image : une salle vide ou une salle où des personnes sont présentes, une allée vide ou une allée avec camion, et ainsi de suite. Dans l’image ci-dessous, vous pouvez voir que l’application détecte quand une banane ou une pomme est placée devant la caméra.

![Animation d’une IU reconnaissant un fruit devant la caméra](./media/iot-visual-alerts-tutorial/scoring.gif)

Ce didacticiel vous explique comment :
> [!div class="checklist"]
> * Configurer l’exemple d’application pour utiliser vos propres ressources Custom Vision et IoT Hub
> * Utiliser l’application pour effectuer l’apprentissage de votre projet Custom Vision
> * Utiliser l’application pour noter les nouvelles images en temps réel et envoyer les résultats à Azure

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services) avant de commencer. 

## <a name="prerequisites"></a>Prérequis

* [!INCLUDE [create-resources](includes/create-resources.md)]
    > [!IMPORTANT]
    > Ce projet doit être un projet de classification d’images **Compact**, car nous allons exporter le modèle vers ONNX ultérieurement.
* Vous devez également [créer une ressource IoT Hub](https://ms.portal.azure.com/#create/Microsoft.IotHub) sur Azure.
* [Visual Studio 2015 ou ultérieur](https://www.visualstudio.com/downloads/)
* Ou un appareil IoT exécutant Windows 10 IoT Core version 17763 ou une version ultérieure. Vous pouvez également exécuter l’application directement à partir de votre ordinateur.
   * Pour Raspberry Pi 2 et 3, vous pouvez configurer Windows 10 directement à partir de l’application Tableau de bord IoT. Pour les autres appareils, comme DrangonBoard, vous devez les flasher à l’aide de la [méthode eMMC](/windows/iot-core/tutorials/quickstarter/devicesetup#flashing-with-emmc-for-dragonboard-410c-other-qualcomm-devices). Si vous avez besoin d’aide pour configurer un nouvel appareil, consultez [Configuration de votre appareil](/windows/iot-core/tutorials/quickstarter/devicesetup) dans la documentation de Windows IoT.

## <a name="about-the-visual-alerts-app"></a>À propos de l’application Alertes visuelles

L’application Alertes visuelles IoT s’exécute en boucle continue, en alternant entre quatre états différents :

* **Aucun modèle** : État de non-fonctionnement. L’application se met en veille continuellement pendant une seconde et vérifie la caméra.
* **Capture des images d’apprentissage** : Dans cet état, l’application capture une image et la charge en tant qu’image d’apprentissage dans le projet Custom Vision cible. L’application se met ensuite en veille pendant 500 ms et répète l’opération jusqu’à ce que le nombre cible défini d’images soit capturé. Elle déclenche ensuite l’apprentissage du modèle Custom Vision.
* **En attente du modèle entraîné** : Dans cet état, l’application appelle l’API Custom Vision chaque seconde pour vérifier si le projet cible contient une itération formée. Lorsqu’elle en trouve une, elle télécharge le modèle ONNX correspondant dans un fichier local et passe à l’état **Scoring**.
* **Scoring** : Dans cet état, l’application utilise Windows ML pour évaluer une image unique de la caméra par rapport au modèle ONNX local. La classification de l’image qui en résulte est affichée à l’écran et envoyée sous forme de message à IoT Hub. L’application se met ensuite en veille pendant une seconde avant de faire le scoring d’une nouvelle image.

## <a name="examine-the-code-structure"></a>Examiner la structure du code

Les fichiers suivants gèrent la fonctionnalité principale de l’application.

| Fichier | Description |
|-------------|-------------|
| [MainPage.xaml](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/blob/master/IoTVisualAlerts/MainPage.xaml) | Ce fichier définit l’interface utilisateur XAML. Il héberge le contrôle de la caméra web et contient les étiquettes utilisées pour les mises à jour de l’état.|
| [MainPage.xaml.cs](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/blob/master/IoTVisualAlerts/MainPage.xaml.cs) | Ce code contrôle le comportement de l’interface utilisateur XAML. Il contient le code de traitement de l’ordinateur d’état.|
| [CustomVision\CustomVisionServiceWrapper.cs](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/blob/master/IoTVisualAlerts/CustomVision/CustomVisionServiceWrapper.cs) | Cette classe est un wrapper qui gère l’intégration avec le service Custom Vision.|
| [CustomVision\CustomVisionONNXModel.cs](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/blob/master/IoTVisualAlerts/CustomVision/CustomVisionONNXModel.cs) | Cette classe est un wrapper qui gère l’intégration avec Windows ML pour le chargement du modèle ONNX et le scoring des images par rapport à celui-ci.|
| [IoTHub\IotHubWrapper.cs](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/blob/master/IoTVisualAlerts/IoTHub/IoTHubWrapper.cs) | Cette classe est un wrapper qui gère l’intégration avec IoT Hub pour charger les résultats de scoring dans Azure.|

## <a name="set-up-the-visual-alerts-app"></a>Configurer l’application Alertes visuelles

Procédez comme suit pour exécuter l’application Alertes visuelles IoT sur votre ordinateur ou votre appareil IoT.

1. Clonez ou téléchargez [l’exemple IoTVisualAlerts](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/tree/master/IoTVisualAlerts) sur GitHub.
1. Ouvrez la solution _IoTVisualAlerts.sln_ dans Visual Studio.
1. Intégrez votre projet Custom Vision :
    1. Dans le script _CustomVision\CustomVisionServiceWrapper.cs_, mettez à jour la variable `ApiKey` à l’aide de votre clé d’apprentissage.
    1. Ensuite, mettez à jour la variable `Endpoint` avec l’URL de point de terminaison associée à votre clé.
    1. Mettez à jour la variable `targetCVSProjectGuid` avec l’ID correspondant du projet Custom Vision que vous souhaitez utiliser. 
1. Configurez la ressource IoT Hub :
    1. Dans le script _IoTHub\IotHubWrapper.cs_, mettez à jour la variable `s_connectionString` avec la chaîne de connexion appropriée pour votre appareil. 
    1. Dans le portail Azure, chargez votre instance IoT Hub, cliquez sur **Appareils IoT** sous **Explorateurs**, sélectionnez votre appareil cible (ou créez-en un si nécessaire) et recherchez la chaîne de connexion sous **Chaîne de connexion principale**. La chaîne contient le nom de votre IoT Hub, l’ID de l’appareil et la clé d’accès partagé ; son format est le suivant : `{your iot hub name}.azure-devices.net;DeviceId={your device id};SharedAccessKey={your access key}`.

## <a name="run-the-app"></a>Exécuter l’application

Si vous exécutez l’application sur votre ordinateur, sélectionnez **Ordinateur local** pour l’appareil cible dans Visual Studio, puis sélectionnez **x64** ou **x86** pour la plateforme cible. Ensuite, appuyez sur F5 pour exécuter le programme. L’application doit démarrer et afficher le flux en direct à partir de la caméra et un message d’état.

Si vous effectuez le déploiement sur un appareil IoT avec un processeur ARM, vous devez sélectionner **ARM** comme plateforme cible et **Ordinateur distant** comme appareil cible. Indiquez l’adresse IP de votre appareil lorsque vous y êtes invité (elle doit se trouver sur le même réseau que votre ordinateur). Vous pouvez récupérer l’adresse IP à partir de l’application Windows IoT par défaut une fois que vous avez démarré l’appareil et que vous l’avez connecté au réseau. Appuyez sur F5 pour exécuter le programme.

Quand vous exécutez l’application pour la première fois, elle n’a aucune connaissance des états visuels. Elle affiche un message d’état indiquant qu’aucun modèle n’est disponible. 

## <a name="capture-training-images"></a>Capturer les images d’apprentissage

Pour configurer un modèle, vous devez placer l’application dans l’état **Capture des images d’apprentissage**. Exécutez l’une des étapes suivantes :
* Si vous exécutez l’application sur un PC, utilisez le bouton situé dans l’angle supérieur droit de l’interface utilisateur.
* Si vous exécutez l’application sur un appareil IoT, appelez la méthode `EnterLearningMode` sur l’appareil via IoT Hub. Vous pouvez l’appeler par le biais de l’entrée de l’appareil dans le menu IoT Hub du portail Azure ou à l’aide d’un outil comme [Explorateur d'appareils IoT Hub](https://github.com/Azure/azure-iot-sdk-csharp).
 
Lorsque l’application entre dans l’état **Capture des images d’apprentissage**, elle capture environ deux images par seconde jusqu’à ce qu’elle ait atteint le nombre cible d’images. Par défaut, le nombre cible est de 30 images, mais vous pouvez définir ce paramètre en passant le nombre souhaité en tant qu’argument à la méthode `EnterLearningMode` IoT Hub. 

Pendant que l’application capture des images, vous devez exposer la caméra aux types d’états visuels que vous souhaitez détecter (par exemple, une salle vide, une salle où des personnes sont présentes, un bureau vide, un bureau avec un camion jouet, etc.).

## <a name="train-the-custom-vision-model"></a>Effectuer l’apprentissage du modèle Custom Vision

Une fois que l’application a terminé la capture des images, elle les charge et bascule sur l’état **En attente du modèle entraîné**. À ce stade, vous devez accéder au [site web Custom Vision](https://www.customvision.ai/) et générer un modèle basé sur les nouvelles images d’entraînement. L’animation suivante montre un exemple de ce processus.

![Animation : étiquetage de plusieurs images de bananes](./media/iot-visual-alerts-tutorial/labeling.gif)

Pour répéter ce processus avec votre propre scénario :

1. Connectez-vous au [site web Custom Vision](http://customvision.ai).
1. Recherchez votre projet cible, qui doit maintenant contenir toutes les images d’apprentissage chargées par l’application.
1. Pour chaque état visuel que vous souhaitez identifier, sélectionnez les images appropriées et appliquez manuellement l’étiquette.
    * Par exemple, si votre objectif est de faire la distinction entre une salle vide et une salle où des personnes se trouvent, nous vous recommandons d’étiqueter cinq images ou plus avec des personnes en tant que nouvelle classe **People** (Personnes) et d’étiqueter au moins cinq images où il n’y a pas de personnes avec l’étiquette **Negative** (Négatif). Cela permet au modèle de faire la différence entre les deux états.
    * Autre exemple : si votre objectif est d’estimer le taux de remplissage d’une étagère, vous pouvez utiliser des étiquettes comme **EmptyShelf** (Étagère vide), **PartiallyFullShelf** (Étagère partiellement remplie) et **FullShelf** (Étagère remplie).
1. Lorsque vous avez terminé, sélectionnez le bouton **Former**.
1. Une fois l’apprentissage terminé, l’application détecte qu’une itération formée est disponible. Elle démarrera le processus d’exportation du modèle entraîné vers ONNX et de son téléchargement sur l’appareil.

## <a name="use-the-trained-model"></a>Utiliser le modèle entraîné

Une fois que l’application a téléchargé le modèle entraîné, elle passe sur l’état **Scoring** et démarre le scoring des images de la caméra dans une boucle continue.

Pour chaque image capturée, l’application affiche l’étiquette supérieure à l’écran. Si elle ne reconnaît pas l’état visuel, elle affiche **No Matches** (Aucune correspondance). L’application envoie également ces messages à IoT Hub, et si une classe est détectée, le message inclut l’étiquette, le niveau de confiance et une propriété appelée `detectedClassAlert`, qui peut être utilisée par les clients IoT Hub qui souhaitent effectuer un routage rapide des messages en fonction des propriétés.

En outre, l’exemple utilise une [bibliothèque Sense HAT](https://github.com/emmellsoft/RPi.SenseHat) pour détecter s’il s’exécute sur un appareil Raspberry Pi avec une unité Sense HAT, afin de pouvoir l’utiliser comme affichage de sortie en définissant tous les voyants d’affichage sur Rouge lorsqu’il détecte une classe et sur Blanc lorsqu’il ne détecte rien.

## <a name="reuse-the-app"></a>Réutiliser l’application

Si vous souhaitez rétablir l’état d’origine de l’application, cliquez sur le bouton situé dans l’angle supérieur droit de l’interface utilisateur ou appelez la méthode `DeleteCurrentModel` via IoT Hub.

À tout moment, vous pouvez répéter l’étape de chargement des images d’apprentissage en cliquant sur le bouton de l’interface utilisateur en haut à droite ou en appelant à nouveau la méthode `EnterLearningMode`.

Si vous exécutez l’application sur un appareil et que vous devez récupérer l’adresse IP à nouveau (pour établir une connexion à distance via le [client distant Windows IoT](https://www.microsoft.com/p/windows-iot-remote-client/9nblggh5mnxz#activetab=pivot:overviewtab), par exemple), vous pouvez appeler la méthode `GetIpAddress` via IoT Hub.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Supprimez votre projet Custom Vision si vous ne voulez plus le conserver. Sur le [site web Custom Vision](https://customvision.ai), accédez à **Projects** (Projets) et sélectionnez la corbeille sous votre nouveau projet.

![Capture d’écran d’un panneau étiqueté My New Project (Nouveau projet) avec une icône de corbeille](./media/csharp-tutorial/delete_project.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez configuré et exécuté une application qui détecte les informations d’état visuel sur un appareil IoT et envoie les résultats à IoT Hub. Explorez ensuite le code source ou effectuez l’une des modifications suggérées ci-dessous.

> [!div class="nextstepaction"]
> [Exemple IoTVisualAlerts (GitHub)](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates/tree/master/IoTVisualAlerts)

* Ajoutez une méthode IoT Hub pour basculer l’application directement vers l’état **En attente du modèle entraîné**. De cette façon, vous pouvez effectuer l’apprentissage du modèle avec des images qui ne sont pas capturées par l’appareil lui-même, puis envoyer (push) le nouveau modèle vers l’appareil sur commande.
* Suivez le tutoriel [Visualiser les données de capteur en temps réel](../../iot-hub/iot-hub-live-data-visualization-in-power-bi.md) pour créer un tableau de bord Power BI afin de visualiser les alertes IoT Hub envoyées par l’exemple.
* Suivez le tutoriel [Surveillance à distance IoT](../../iot-hub/iot-hub-monitoring-notifications-with-azure-logic-apps.md) pour créer une application logique qui répond aux alertes IoT Hub lorsque des états visuels sont détectés.