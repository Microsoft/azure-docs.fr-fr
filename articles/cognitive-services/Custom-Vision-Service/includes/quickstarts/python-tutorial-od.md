---
author: PatrickFarley
ms.author: pafarley
ms.service: cognitive-services
ms.date: 10/25/2020
ms.openlocfilehash: b35b383eb93b0c451fcb98c66538a4335480a649
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021277"
---
Bien démarrer avec la bibliothèque de client Custom Vision pour Python Suivez les étapes ci-après pour installer le package et essayer l’exemple de code visant à créer un modèle de détection d’objets. Vous allez créer un projet, ajouter des étiquettes, entraîner le projet et utiliser l’URL de point de terminaison de prédiction du projet pour le tester programmatiquement. Utilisez cet exemple comme modèle pour générer votre propre application de reconnaissance d’image.

> [!NOTE]
> Si vous souhaitez créer et entraîner un modèle de détection d’objets _sans_ écrire de code, consultez à la place les [instructions basées sur le navigateur](../../get-started-build-detector.md).

Utilisez la bibliothèque de client Custom Vision pour Python pour :

* Créer un projet de service Custom Vision
* Ajouter des balises au projet
* Charger et étiqueter des images
* Entraîner le projet
* Publier l’itération actuelle
* Tester le point de terminaison de prédiction

[Documentation de référence](/python/api/overview/azure/cognitiveservices/customvision?view=azure-python) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-vision-customvision/azure/cognitiveservices/vision/customvision) | [Package (PyPI)](https://pypi.org/project/azure-cognitiveservices-vision-customvision/) | [Exemples](/samples/browse/?languages=python&products=azure&term=vision&terms=vision)

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* [Python 3.x](https://www.python.org/)
* Une fois que vous avez votre abonnement Azure, <a href="https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_customvision#create/Microsoft.CognitiveServicesCustomVision"  title="Créer une ressource Custom Vision"  target="_blank">créez une ressource Custom Vision <span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le portail Azure pour créer une ressource d’entraînement et de prédiction et obtenir vos clés et votre point de terminaison. Attendez qu’elle se déploie, puis cliquez sur le bouton **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison des ressources que vous créez pour connecter votre application Custom Vision. Vous collerez vos clés et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.

## <a name="setting-up"></a>Configuration

### <a name="install-the-client-library"></a>Installer la bibliothèque de client

Pour écrire une application d’analyse d’image avec Custom Vision pour Python, vous avez besoin de la bibliothèque de client Custom Vision. Après l’installation de Python, exécutez la commande suivante dans PowerShell ou dans une fenêtre de console :

```powershell
pip install azure-cognitiveservices-vision-customvision
```

### <a name="create-a-new-python-application"></a>Créer une application Python

Créez un fichier Python et importez les bibliothèques suivantes.

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_imports)]

> [!TIP]
> Vous voulez voir l’intégralité du fichier de code de démarrage rapide à la fois ? Vous le trouverez sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py), qui contient les exemples de code utilisés dans ce guide de démarrage rapide.

Créez des variables pour le point de terminaison et les clés d’abonnement Azure de votre ressource.

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_creds)]


> [!IMPORTANT]
> Accédez au portail Azure. Si les ressources Custom Vision que vous avez créées dans la section **Prérequis** ont été déployées, cliquez sur le bouton **Accéder à la ressource** sous **Étapes suivantes**. Les clés et le point de terminaison se trouvent dans les pages **Clé et point de terminaison** des ressources, sous **Gestion des ressources**. Vous devez vous procurer vos clés d’entraînement et de prédiction.
>
> Vous trouverez la valeur de l’ID de ressource de prédiction sous l’onglet **Vue d’ensemble** de la ressource, listé comme **ID d’abonnement**.
>
> N’oubliez pas de supprimer les clés de votre code une fois que vous avez terminé, et ne les postez jamais publiquement. Pour la production, envisagez d’utiliser une méthode de stockage et d’accès sécurisée pour vos informations d’identification. Pour plus d’informations, consultez l’article sur la [sécurité](../../../../cognitive-services/cognitive-services-security.md) de Cognitive Services.

## <a name="object-model"></a>Modèle objet

|Nom|Description|
|---|---|
|[CustomVisionTrainingClient](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.training.customvisiontrainingclient?view=azure-python) | Cette classe gère la création, l’entraînement et la publication de vos modèles. |
|[CustomVisionPredictionClient](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.prediction.customvisionpredictionclient?view=azure-python)| Cette classe gère l’interrogation de vos modèles pour les prédictions de détection d’objets.|
|[ImagePrediction](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.prediction.models.imageprediction?view=azure-python)| Cette classe définit une prédiction d’objet unique sur une seule image. Elle comprend des propriétés pour l’ID et le nom de l’objet, la position du rectangle englobant de l’objet et un score de confiance.|

## <a name="code-examples"></a>Exemples de code

Ces extraits de code vous montrent comment effectuer les opérations suivantes avec la bibliothèque de client Custom Vision pour Python :

* [Authentifier le client](#authenticate-the-client)
* [Créer un projet de service Custom Vision](#create-a-new-custom-vision-project)
* [Ajouter des balises au projet](#add-tags-to-the-project)
* [Charger et étiqueter des images](#upload-and-tag-images)
* [Entraîner le projet](#train-the-project)
* [Publier l’itération actuelle](#publish-the-current-iteration)
* [Tester le point de terminaison de prédiction](#test-the-prediction-endpoint)

## <a name="authenticate-the-client"></a>Authentifier le client

Instanciez un client d’entraînement et de prédiction à l’aide de votre point de terminaison et de vos clés. Créez des objets **ApiKeyServiceClientCredentials** avec vos clés et utilisez-les avec votre point de terminaison pour créer un objet [CustomVisionTrainingClient](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.training.customvisiontrainingclient?view=azure-python) et un objet [CustomVisionPredictionClient](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.prediction.customvisionpredictionclient?view=azure-python).

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_auth)]


## <a name="create-a-new-custom-vision-project"></a>Créer un projet de service Custom Vision

Pour créer un nouveau projet Service Vision personnalisée, ajoutez le code suivant à votre script. 

Consultez la méthode [create_project](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.training.operations.customvisiontrainingclientoperationsmixin?view=azure-python#create-project-name--description-none--domain-id-none--classification-type-none--target-export-platforms-none--custom-headers-none--raw-false----operation-config-&preserve-view=true) pour spécifier d’autres options quand vous créez votre projet (procédure expliquée dans le guide du portail web [Build a detector](../../get-started-build-detector.md) [Créer un détecteur]).  

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_create)]


## <a name="add-tags-to-the-project"></a>Ajouter des balises au projet

Pour créer des étiquettes d’objet dans votre projet, ajoutez le code suivant :

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_tags)]


## <a name="upload-and-tag-images"></a>Charger et étiqueter des images

Tout d’abord, téléchargez les exemples d’images pour ce projet. Enregistrez le contenu du [dossier des exemples d’images](https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/CustomVision/ObjectDetection/Images) sur votre appareil local.

Lorsque vous appliquez des balises à des images dans des projets de détection d’objet, vous devez préciser la région de chaque objet balisé avec des coordonnées normalisées. Le code suivant associe chaque exemple d’image à sa région balisée. Les régions spécifient le niveau du bit dans des coordonnées normalisées, et ces dernières sont données dans l’ordre : gauche, haut, largeur et hauteur.

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_tagging)]

> [!NOTE]
> Si vous n’avez pas d’utilitaire permettant d’effectuer des opérations de glisser-déposer pour marquer les coordonnées des régions, vous pouvez utiliser l’IU web sur [Customvision.ai](https://www.customvision.ai/). Dans cet exemple, les coordonnées sont déjà fournies.

Utilisez alors cette carte d’associations pour charger chaque exemple d’image avec ses coordonnées de région (vous pouvez charger jusqu’à 64 images dans un même lot). Ajoutez le code ci-dessous.


[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_upload)]

> [!NOTE]
> Vous devrez changer le chemin des images en fonction de l’emplacement à partir duquel vous avez téléchargé le dépôt d’exemples du kit SDK Python pour Cognitive Services.

## <a name="train-the-project"></a>Entraîner le projet

Ce code crée la première itération du modèle de prédiction. 

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_train)]

> [!TIP]
> Entraîner avec des étiquettes sélectionnées
>
> Vous pouvez, si vous le souhaitez, effectuer l’entraînement sur un sous-ensemble de vos étiquettes appliquées. Vous pouvez procéder de la sorte si vous n’avez pas encore suffisamment appliqué certaines étiquettes, contrairement à d’autres. Dans l’appel **[train_project](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.training.operations.customvisiontrainingclientoperationsmixin?view=azure-python#train-project-project-id--training-type-none--reserved-budget-in-hours-0--force-train-false--notification-email-address-none--selected-tags-none--custom-headers-none--raw-false----operation-config-&preserve-view=true)** , définissez le paramètre facultatif *selected_tags* sur une liste de chaînes d’ID des étiquettes que vous souhaitez utiliser. Le modèle effectue l’entraînement pour ne reconnaître que les étiquettes présentes dans cette liste.

## <a name="publish-the-current-iteration"></a>Publier l’itération actuelle

Les itérations ne sont pas disponibles sur le point de terminaison de prédiction tant qu’elles n’ont pas été publiées. Le code suivant rend l’itération actuelle du modèle disponible pour l’interrogation. 

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_publish)]

## <a name="test-the-prediction-endpoint"></a>Tester le point de terminaison de prédiction

Pour envoyer une image au point de terminaison de prédiction et récupérer la prédiction, ajoutez le code suivant à la fin du fichier :

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py?name=snippet_test)]


## <a name="run-the-application"></a>Exécution de l'application

Exécutez *CustomVisionQuickstart.py*.

```powershell
python CustomVisionQuickstart.py
```

La sortie de l’application doit apparaître dans la console. Vous pouvez alors vérifier que l’image test (qui se trouve dans **<emplacement_de_base_des_images>/images/Test**) est étiquetée de façon appropriée et que la région de détection est correcte. Vous pouvez aussi revenir sur le [site web Custom Vision](https://customvision.ai) et consulter l’état actuel de votre nouveau projet.

[!INCLUDE [clean-od-project](../../includes/clean-od-project.md)]

## <a name="next-steps"></a>Étapes suivantes

Vous avez à présent effectué chaque étape du processus de détection d’objet dans le code. Cet exemple exécute une seule itération d’entraînement, mais vous aurez souvent besoin d’entraîner et de tester votre modèle à plusieurs reprises pour le rendre plus précis. Le guide suivant traite de la classification d’images, mais ses principes sont identiques à la détection d’objet.

> [!div class="nextstepaction"]
> [Tester et réentraîner un modèle](../../test-your-model.md)

* Qu’est-ce que Custom Vision ?
* Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/CustomVision/ObjectDetection/CustomVisionQuickstart.py).
* [Documentation de référence du SDK](/python/api/overview/azure/cognitiveservices/customvision?view=azure-python)
