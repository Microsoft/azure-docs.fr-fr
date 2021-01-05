---
author: areddish
ms.author: areddish
ms.service: cognitive-services
ms.date: 10/26/2020
ms.custom: devx-track-js
ms.openlocfilehash: 3942d4e7316c9617e1d2f61820cfa91b42d4b883
ms.sourcegitcommit: 9706bee6962f673f14c2dc9366fde59012549649
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94625440"
---
Ce guide fournit des instructions et un exemple de code pour vous aider à commencer à utiliser la bibliothèque de client Custom Vision pour Node.js afin de générer un modèle de classification d’images. Vous allez créer un projet, ajouter des étiquettes, entraîner le projet et utiliser l’URL de point de terminaison de prédiction du projet pour le tester programmatiquement. Utilisez cet exemple comme modèle pour générer votre propre application de reconnaissance d’image.

> [!NOTE]
> Si vous souhaitez créer et entraîner un modèle de classification _sans_ écrire de code, consultez à la place les [instructions basées sur le navigateur](../../getting-started-build-a-classifier.md).

Utilisez la bibliothèque de client Custom Vision pour .NET pour :

* Créer un projet de service Custom Vision
* Ajouter des balises au projet
* Charger et étiqueter des images
* Entraîner le projet
* Publier l’itération actuelle
* Tester le point de terminaison de prédiction

Documentation de référence [(entraînement)](/javascript/api/@azure/cognitiveservices-customvision-training/?view=azure-node-latest) [(prédiction)](/javascript/api/@azure/cognitiveservices-customvision-prediction/?view=azure-node-latest) | Code source de la bibliothèque [(entraînement)](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-customvision-training) [(prédiction)](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-customvision-prediction) | Package (npm) [(entraînement)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-training) [(prédiction)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-prediction) | [Exemples](/samples/browse/?products=azure&terms=custom%20vision&languages=javascript)

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* Version actuelle de [Node.js](https://nodejs.org/)
* Une fois que vous avez votre abonnement Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesCustomVision"  title="Créer une ressource Custom Vision"  target="_blank">créez une ressource Custom Vision <span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le portail Azure pour créer une ressource d’entraînement et de prédiction et obtenir vos clés et votre point de terminaison. Attendez qu’elle se déploie, puis cliquez sur le bouton **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison des ressources que vous créez pour connecter votre application Custom Vision. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide.
    * Vous pouvez utiliser le niveau tarifaire Gratuit (`F0`) pour tester le service, puis passer par la suite à un niveau payant pour la production.

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-nodejs-application"></a>Création d’une application Node.js

Dans une fenêtre de console (telle que cmd, PowerShell ou bash), créez un répertoire pour votre application et accédez-y. 

```console
mkdir myapp && cd myapp
```

Exécutez la commande `npm init` pour créer une application de nœud avec un fichier `package.json`. 

```console
npm init
```

### <a name="install-the-client-library"></a>Installer la bibliothèque de client

Pour écrire une application d’analyse d’image avec Custom Vision pour Node.js, vous avez besoin des packages NPM Custom Vision. Pour les installer, exécutez la commande suivante dans PowerShell :

```shell
npm install @azure/cognitiveservices-customvision-training
npm install @azure/cognitiveservices-customvision-prediction
```

Le fichier `package.json` de votre application sera mis à jour avec les dépendances.

Créez un fichier nommé `index.js` et importez les bibliothèques suivantes :

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_imports)]


> [!TIP]
> Vous voulez voir l’intégralité du fichier de code de démarrage rapide à la fois ? Vous le trouverez sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js), qui contient les exemples de code utilisés dans ce guide de démarrage rapide.

Créez des variables pour le point de terminaison et les clés Azure de votre ressource. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_creds)]

> [!IMPORTANT]
> Accédez au portail Azure. Si la ressource [nom du produit] que vous avez créée dans la section **Prérequis** a été déployée, cliquez sur le bouton **Accéder à la ressource** sous **Étapes suivantes**. La clé et le point de terminaison se trouvent dans la page **Clé et point de terminaison** de la ressource, sous **Gestion des ressources**. 
>
> N’oubliez pas de supprimer la clé de votre code une fois que vous avez terminé, et ne la postez jamais publiquement. Pour la production, envisagez d’utiliser une méthode de stockage et d’accès sécurisée pour vos informations d’identification. Pour plus d’informations, consultez l’article sur la [sécurité](../../../cognitive-services-security.md) de Cognitive Services.

Ajoutez également des champs pour le nom de votre projet et un paramètre de délai d’attente pour les appels asynchrones.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_vars)]


## <a name="object-model"></a>Modèle objet

|Nom|Description|
|---|---|
|[TrainingAPIClient](/javascript/api/@azure/cognitiveservices-customvision-training/trainingapiclient?view=azure-node-latest) | Cette classe gère la création, l’entraînement et la publication de vos modèles. |
|[PredictionAPIClient](/javascript/api/@azure/cognitiveservices-customvision-prediction/predictionapiclient?view=azure-node-latest)| Cette classe gère l’interrogation de vos modèles pour les prédictions de classification d’images.|
|[Prédiction](/javascript/api/@azure/cognitiveservices-customvision-prediction/prediction?view=azure-node-latest)| Cette interface définit une prédiction unique sur une seule image. Elle comprend des propriétés pour l’ID et le nom de l’objet ainsi qu’un score de confiance.|

## <a name="code-examples"></a>Exemples de code

Ces extraits de code vous montrent comment effectuer les tâches suivantes avec la bibliothèque de client Custom Vision pour JavaScript :

* [Authentifier le client](#authenticate-the-client)
* [Créer un projet de service Custom Vision](#create-a-new-custom-vision-project)
* [Ajouter des balises au projet](#add-tags-to-the-project)
* [Charger et étiqueter des images](#upload-and-tag-images)
* [Entraîner le projet](#train-the-project)
* [Publier l’itération actuelle](#publish-the-current-iteration)
* [Tester le point de terminaison de prédiction](#test-the-prediction-endpoint)

## <a name="authenticate-the-client"></a>Authentifier le client

Instanciez les objets clients avec le point de terminaison et la clé. Créez un objet **ApiKeyCredentials** avec votre clé et utilisez-le avec votre point de terminaison pour créer un objet [TrainingAPIClient](/javascript/api/@azure/cognitiveservices-customvision-training/trainingapiclient?view=azure-node-latest) et un objet [PredictionAPIClient](/javascript/api/@azure/cognitiveservices-customvision-prediction/predictionapiclient?view=azure-node-latest).

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_auth)]


## <a name="create-a-new-custom-vision-project"></a>Créer un projet de service Custom Vision

Démarrez une nouvelle fonction destinée à contenir tous vos appels de fonction Custom Vision. Ajoutez le code suivant pour créer un projet de service Custom Vision.


[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_create)]


## <a name="add-tags-to-the-project"></a>Ajouter des balises au projet

Pour créer des étiquettes de classification dans votre projet, ajoutez le code suivant à votre fonction :

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_tags)]


## <a name="upload-and-tag-images"></a>Charger et étiqueter des images

Tout d’abord, téléchargez les exemples d’images pour ce projet. Enregistrez le contenu du [dossier des exemples d’images](https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/CustomVision/ImageClassification/Images) sur votre appareil local.

Pour ajouter les exemples d’images au projet, insérez le code suivant après la création de mots clés. Ce code charge chaque image avec la balise correspondante.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_upload)]

> [!IMPORTANT]
> Vous devrez changer le chemin des images (`sampleDataRoot`) en fonction de l’emplacement à partir duquel vous avez téléchargé le dépôt d’exemples du kit SDK Python pour Cognitive Services.

## <a name="train-the-project"></a>Entraîner le projet

Ce code crée la première itération du modèle de prédiction. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_train)]

## <a name="publish-the-current-iteration"></a>Publier l’itération actuelle

Ce code publie l’itération entraînée sur le point de terminaison de prédiction. Le nom donné à l’itération publiée peut être utilisé pour envoyer des requêtes de prédiction. Les itérations ne sont pas disponibles sur le point de terminaison de prédiction tant qu’elles n’ont pas été publiées.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_publish)]


## <a name="test-the-prediction-endpoint"></a>Tester le point de terminaison de prédiction

Pour envoyer une image au point de terminaison de prédiction et récupérer la prédiction, ajoutez le code suivant à votre fonction. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_test)]

Ensuite, fermez votre fonction Custom Vision et appelez-la.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_function_close)]


## <a name="run-the-application"></a>Exécution de l'application

Exécutez l’application avec la commande `node` de votre fichier de démarrage rapide.

```console
node index.js
```

Le résultat de l’application ressemble au texte suivant :

```console
Creating project...
Adding images...
Training...
Training started...
Training status: Training
Training status: Training
Training status: Training
Training status: Completed
Results:
         Hemlock: 94.97%
         Japanese Cherry: 0.01%
```

Vous pouvez ensuite vérifier que l’image test (qui se trouve dans **<sampleDataRoot>/Test/** ) est étiquetée de façon appropriée. Vous pouvez aussi revenir sur le [site web Custom Vision](https://customvision.ai) et consulter l’état actuel de votre nouveau projet.

[!INCLUDE [clean-ic-project](../../includes/clean-ic-project.md)]

## <a name="next-steps"></a>Étapes suivantes

Vous savez maintenant comment effectuer la détection d’objet dans le code. Cet exemple exécute une seule itération d’entraînement, mais vous aurez souvent besoin d’entraîner et de tester votre modèle à plusieurs reprises pour le rendre plus précis.

> [!div class="nextstepaction"]
> [Tester et réentraîner un modèle](../../test-your-model.md)

* Qu’est-ce que Custom Vision ?
* Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js).
* [Documentation de référence du SDK (entraînement)](/javascript/api/@azure/cognitiveservices-customvision-training/?view=azure-node-latest)
* [Documentation de référence du SDK (prédiction)](/javascript/api/@azure/cognitiveservices-customvision-prediction/?view=azure-node-latest)
