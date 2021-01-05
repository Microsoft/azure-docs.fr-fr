---
title: Fichier include
description: Fichier include
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.date: 09/01/2020
ms.topic: include
ms.custom: include file, devx-track-js, cog-serv-seo-aug-2020
ms.openlocfilehash: 0c3a02461d8ad4bc8470383244dad7b86b47f405
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96303687"
---
Utilisez les bibliothèques de client LUIS (Language Understanding) pour Node.js pour :

* Créer une application
* Ajouter une intention, une entité issue du Machine Learning, avec un exemple d'énoncé
* Entraîner et publier une application.
* Interroger un runtime de prédiction

[Documentation de référence](/javascript/api/@azure/cognitiveservices-luis-authoring/?view=azure-node-latest) |  [Création](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-luis-authoring) et [Prédiction](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-luis-runtime) - Code source de la bibliothèque | [Création](https://www.npmjs.com/package/@azure/cognitiveservices-luis-authoring) et [Prédiction](https://www.npmjs.com/package/@azure/cognitiveservices-luis-runtime) - NPM | [Exemples](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/LUIS/sdk-3x/index.js)

## <a name="prerequisites"></a>Prérequis

* [Node.JS](https://nodejs.org)
* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services)
* Une fois en possession de votre abonnement Azure, [créez une ressource de création LUIS](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesLUISAllInOne) sur le portail Azure pour obtenir vos clé et point de terminaison. Attendez qu’elle se déploie, puis cliquez sur le bouton **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous [créez](../luis-how-to-azure-subscription.md#create-luis-resources-in-the-azure-portal) pour connecter votre application à la création LUIS. Vous collerez votre clé et votre point de terminaison dans le code ci-dessous plus loin dans le guide de démarrage rapide. Vous pouvez utiliser le niveau tarifaire gratuit (`F0`) pour tester le service.

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-javascript-application"></a>Créer une application JavaScript

1. Dans une fenêtre de console, créez un nouveau répertoire pour votre application et déplacez-vous dans ce répertoire.

    ```console
    mkdir quickstart-sdk && cd quickstart-sdk
    ```

1. Initialisez le répertoire en tant qu'application JavaScript en créant un fichier `package.json`.

    ```console
    npm init -y
    ```

1. Créez un fichier nommé `index.js` pour votre code JavaScript.

    ```console
    touch index.js
    ```

### <a name="install-the-npm-libraries"></a>Installer les bibliothèques NPM

Dans le répertoire de l'application, installez les dépendances à l'aide des commandes suivantes, exécutées une ligne à la fois :

```console
npm install @azure/ms-rest-js
npm install @azure/cognitiveservices-luis-authoring
npm install @azure/cognitiveservices-luis-runtime
```

Votre `package.json` doit se présenter comme suit :

```json
{
  "name": "quickstart-sdk",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@azure/cognitiveservices-luis-authoring": "^4.0.0-preview.3",
    "@azure/cognitiveservices-luis-runtime": "^5.0.0",
    "@azure/ms-rest-js": "^2.0.8"
  }
}
```

## <a name="authoring-object-model"></a>Modèle d'objet de création

Le client de création Language Understanding (LUIS) est un objet [LUISAuthoringClient](/javascript/api/@azure/cognitiveservices-luis-authoring/luisauthoringclient?view=azure-node-latest) qui s’authentifie auprès d’Azure et contient votre clé de création.

## <a name="code-examples-for-authoring"></a>Exemples de code pour la création

Une fois le client créé, utilisez-le pour accéder aux fonctionnalités, notamment :

* Applications : [add](/javascript/api/@azure/cognitiveservices-luis-authoring/apps?view=azure-node-latest#add-applicationcreateobject--msrest-requestoptionsbase-), [delete](/javascript/api/@azure/cognitiveservices-luis-authoring/apps?view=azure-node-latest#deletemethod-string--models-appsdeletemethodoptionalparams-), [publish](/javascript/api/@azure/cognitiveservices-luis-authoring/apps?view=azure-node-latest#publish-string--applicationpublishobject--msrest-requestoptionsbase-)
* Exemples d’énoncés : [ajouter par lot](/javascript/api/@azure/cognitiveservices-luis-authoring/examples?view=azure-node-latest#batch-string--string--examplelabelobject----msrest-requestoptionsbase-), [supprimer par ID](/javascript/api/@azure/cognitiveservices-luis-authoring/examples?view=azure-node-latest#deletemethod-string--string--number--msrest-requestoptionsbase-)
* Fonctionnalités : gérer les [listes d’expressions](/javascript/api/@azure/cognitiveservices-luis-authoring/features?view=azure-node-latest#addphraselist-string--string--phraselistcreateobject--msrest-requestoptionsbase-)
* Modèle : gérer les [intentions](/javascript/api/@azure/cognitiveservices-luis-authoring/model?view=azure-node-latest#addintent-string--string--modelcreateobject--msrest-requestoptionsbase-) et les entités
* Modèle : gérer les [modèles](/javascript/api/@azure/cognitiveservices-luis-authoring/pattern?view=azure-node-latest#addpattern-string--string--patternrulecreateobject--msrest-requestoptionsbase-)
* Entraînement : [entraînez](/javascript/api/@azure/cognitiveservices-luis-authoring/train?view=azure-node-latest#trainversion-string--string--msrest-requestoptionsbase-) l’application et interrogez l’[état de l’entraînement](/javascript/api/@azure/cognitiveservices-luis-authoring/train?view=azure-node-latest#getstatus-string--string--msrest-requestoptionsbase-)
* [Versions](/javascript/api/@azure/cognitiveservices-luis-authoring/versions?view=azure-node-latest) : gérer avec clonage, exportation et suppression

## <a name="prediction-object-model"></a>Modèle d'objet de prédiction

Le client de création Language Understanding (LUIS) est un objet [LUISAuthoringClient](/javascript/api/@azure/cognitiveservices-luis-runtime/luisruntimeclient?view=azure-node-latest) qui s’authentifie auprès d’Azure et contient votre clé de création.

## <a name="code-examples-for-prediction-runtime"></a>Exemples de code pour le runtime de prédiction

Une fois le client créé, utilisez-le pour accéder aux fonctionnalités, notamment :

* [Prédiction](/javascript/api/@azure/cognitiveservices-luis-runtime/predictionoperations?view=azure-node-latest#getslotprediction-string--string--predictionrequest--models-predictiongetslotpredictionoptionalparams-) par emplacement de `staging` ou de `production`
* [Prédiction par version](/javascript/api/@azure/cognitiveservices-luis-runtime/predictionoperations?view=azure-node-latest#getversionprediction-string--string--predictionrequest--models-predictiongetversionpredictionoptionalparams-)

[!INCLUDE [Bookmark links to same article](sdk-code-examples.md)]

## <a name="add-the-dependencies"></a>Ajouter les dépendances

Ouvrez le fichier `index.js` dans votre éditeur ou IDE préféré, puis ajoutez les dépendances suivantes.

[!code-javascript[Add NPM libraries to code file](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=Dependencies)]

## <a name="add-boilerplate-code"></a>Ajouter du code réutilisable

1. Ajoutez la méthode `quickstart` et son appel. Cette méthode contient la majeure partie du code restant. Cette méthode est appelée à la fin du fichier.

    ```javascript
    const quickstart = async () => {

        // add calls here


    }
    quickstart()
        .then(result => console.log("Done"))
        .catch(err => {
            console.log(`Error: ${err}`)
            })
    ```

1. Sauf indication contraire, ajoutez le code restant dans la méthode de démarrage rapide.

## <a name="create-variables-for-the-app"></a>Créer des variables pour l'application

Créez deux ensembles de variables : le premier ensemble que vous modifiez, le deuxième ensemble qui reste tel qu’il apparaît dans l’exemple de code. 

1. Créez des variables pour stocker votre clé de création et les noms de vos ressources.

    [!code-javascript[Variables you need to change](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=VariablesYouChange)]

1. Créez des variables pour stocker vos points de terminaison, le nom de l'application, la version et le nom de l'intention.

    [!code-javascript[Variables you don't need to change](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=VariablesYouDontNeedToChangeChange)]

## <a name="authenticate-the-client"></a>Authentifier le client

Créez un objet [CognitiveServicesCredentials](/javascript/api/@azure/ms-rest-js/apikeycredentials?view=azure-node-latest) avec votre clé et utilisez-le avec votre point de terminaison pour créer un objet [LUISAuthoringClient](/javascript/api/@azure/cognitiveservices-luis-authoring/luisauthoringclient?view=azure-node-latest).

[!code-javascript[Authenticate the client](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringCreateClient)]

## <a name="create-a-luis-app"></a>Créer une application LUIS

Une application LUIS stocke le modèle de traitement en langage naturel contenant les intentions, les entités et les exemples d'énoncés.

Créez une méthode [add](/javascript/api/@azure/cognitiveservices-luis-authoring/apps?view=azure-node-latest) pour l’objet [AppsOperation](/javascript/api/@azure/cognitiveservices-luis-authoring/apps?view=azure-node-latest) afin de créer l’application. Le nom et la culture de la langue sont des propriétés obligatoires.

[!code-javascript[Create a LUIS app](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringCreateApplication)]


## <a name="create-intent-for-the-app"></a>Créer une intention pour l’application
L’objet principal dans un modèle d’application LUIS est l’intention. L’intention s’aligne sur un regroupement d’_intentions_ d’énoncés utilisateur. Un utilisateur peut poser une question ou émettre un énoncé en souhaitant obtenir une réponse _prévue_ particulière d’un bot (ou d’une autre application cliente). Réserver un billet d’avion, demander quelle est la météo dans une ville de destination et demander des informations de contact pour un service client sont des exemples d’intentions.

Utilisez la méthode [model.add_intent](/javascript/api/@azure/cognitiveservices-luis-authoring/model?view=azure-node-latest#addintent-string--string--modelcreateobject--msrest-requestoptionsbase-) avec le nom de l’intention unique, puis transmettez l’ID de l’application, l’ID de version et le nom de la nouvelle intention.

La valeur `intentName` est codée en dur dans `OrderPizzaIntent` dans le cadre des variables de la section [Créer des variables pour l'application](#create-variables-for-the-app).

[!code-javascript[Create intent for the app](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AddIntent)]

## <a name="create-entities-for-the-app"></a>Créer des entités pour l’application

Bien que les entités ne soient pas obligatoires, elles sont présentes dans la plupart des applications. L’entité extrait des informations à partir de l’énoncé utilisateur, qui sont nécessaires pour répondre à l’intention de l’utilisateur. Il existe plusieurs types d’entités [prédéfinies](/javascript/api/@azure/cognitiveservices-luis-authoring/model?view=azure-node-latest#addcustomprebuiltentity-string--string--prebuiltdomainmodelcreateobject--msrest-requestoptionsbase-) et personnalisées, chacune avec leurs propres modèles DTO (Data Transformation Object).  Les entités prédéfinies courantes à ajouter à votre application incluent [number](../luis-reference-prebuilt-number.md), [datetimeV2](../luis-reference-prebuilt-datetimev2.md), [geographyV2](../luis-reference-prebuilt-geographyv2.md) et [ordinal](../luis-reference-prebuilt-ordinal.md).

Il est important de savoir que les entités ne sont pas marquées avec une intention. Elles peuvent s’appliquer à de nombreuses intentions. Seuls les exemples d’énoncés utilisateur sont marqués pour une intention unique spécifique.

Les méthodes de création pour les entités font partie de la classe [Model](/javascript/api/@azure/cognitiveservices-luis-authoring/model?view=azure-node-latest). Chaque type d’entité possède son propre modèle DTO (Data Transformation Object).

Le code de création d'entité crée une entité de Machine Learning avec des sous-entités et des fonctionnalités appliquées aux sous-entités `Quantity`.

:::image type="content" source="../media/quickstart-sdk/machine-learned-entity.png" alt-text="Capture d’écran partielle du portail montrant l’entité créée : une entité de machine learning avec des sous-entités et des fonctionnalités appliquées aux sous-entités « Quantity ».":::

[!code-javascript[Create entities for the app](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringAddEntities)]

Placez la méthode suivante au-dessus de la méthode `quickstart` pour trouver l’ID de la sous-entité Quantity, afin d’attribuer les fonctionnalités à cette sous-entité.

[!code-javascript[Find subentity id](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringSortModelObject)]

## <a name="add-example-utterance-to-intent"></a>Ajouter un exemple d’énoncé à une intention

Pour déterminer l’intention d’un énoncé et extraire des entités, l’application a besoin d’exemples d’énoncés. Les exemples doivent cibler une intention spécifique et unique, et doivent marquer toutes les entités personnalisées. Les entités prédéfinies n’ont pas besoin d’être marquées.

Ajoutez des exemples d’énoncés en créant une liste d’objets [ExampleLabelObject](/javascript/api/@azure/cognitiveservices-luis-authoring/examplelabelobject?view=azure-node-latest), un objet pour chaque exemple d’énoncé. Chaque exemple doit marquer toutes les entités avec un dictionnaire de paires nom/valeur de nom d’entité et de valeur d’entité. La valeur de l’entité doit être exactement telle qu’elle apparaît dans le texte de l’exemple d’énoncé.

:::image type="content" source="../media/quickstart-sdk/labeled-example-machine-learned-entity.png" alt-text="Capture d’écran partielle montrant l’exemple d’énoncé étiqueté dans le portail.":::

Appelez [examples.add](/javascript/api/@azure/cognitiveservices-luis-authoring/examples?view=azure-node-lates) avec l'ID de l'application, l'ID de version et l'exemple.

[!code-javascript[Add example utterance to intent](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=AuthoringAddLabeledExamples)]

## <a name="train-the-app"></a>Effectuer l’apprentissage de l’application

Une fois le modèle créé, l’application LUIS doit être entraînée pour cette version du modèle. Un modèle entraîné peut être utilisé dans un [conteneur](../luis-container-howto.md) ou [publié](../luis-how-to-publish-app.md) dans les emplacements intermédiaires ou produits.

La méthode [train.trainVersion](/javascript/api/@azure/cognitiveservices-luis-authoring/train?view=azure-node-latest#trainversion-string--string--msrest-requestoptionsbase-) a besoin de l’ID d’application et de l’ID de version.

Un modèle très petit, comme l’illustre ce guide de démarrage rapide, sera entraîné très rapidement. Pour les applications de niveau production, l’entraînement de l’application doit inclure un appel d’interrogation à la méthode [get_status](/javascript/api/@azure/cognitiveservices-luis-authoring/train?view=azure-node-latest#getstatus-string--string--msrest-requestoptionsbase-) pour déterminer si l’entraînement a réussi. La réponse est une liste d’objets [ModelTrainingInfo](/javascript/api/@azure/cognitiveservices-luis-authoring/modeltraininginfo?view=azure-node-latest) avec un état distinct pour chaque objet. Tous les objets doivent réussir pour que l’entraînement soit considéré comme terminé.

[!code-javascript[Train the app](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=TrainAppVersion)]

## <a name="publish-app-to-production-slot"></a>Publier une application dans l’emplacement de production

Publiez l’application LUIS à l’aide de la méthode [app.publish](/javascript/api/@azure/cognitiveservices-luis-authoring/apps?view=azure-node-latest#publish-string--applicationpublishobject--msrest-requestoptionsbase-). Cela permet de publier la version entraînée actuelle à l’emplacement spécifié au point de terminaison. Votre application cliente utilise ce point de terminaison afin d’envoyer des énoncés utilisateur pour la prédiction de l’intention et l’extraction d’entité.

[!code-javascript[Publish app to production slot](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=PublishVersion)]


## <a name="authenticate-the-prediction-runtime-client"></a>Authentifier le client du runtime de prédiction

Utilisez un objet msRest.ApiKeyCredentials avec votre clé, puis utilisez-le avec votre point de terminaison pour créer un objet [LUIS.LUISRuntimeClient](/javascript/api/@azure/cognitiveservices-luis-runtime/luisruntimeclient?view=azure-node-latest).

[!INCLUDE [Caution about using authoring key](caution-authoring-key.md)]

[!code-javascript [Authenticate the prediction runtime client](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=PredictionCreateClient)]

## <a name="get-prediction-from-runtime"></a>Obtenir une prédiction du runtime

Ajoutez le code suivant pour créer la requête à adresser au runtime de prédiction. L’énoncé utilisateur fait partie de l’objet [predictionRequest](/javascript/api/@azure/cognitiveservices-luis-runtime/predictionrequest?view=azure-node-latest).

La méthode **[luisRuntimeClient.prediction.getSlotPrediction](/javascript/api/@azure/cognitiveservices-luis-runtime/predictionoperations?view=azure-node-latest#getslotprediction-string--string--predictionrequest--models-predictiongetslotpredictionoptionalparams-)** nécessite plusieurs paramètres tels que l’ID de l’application, le nom de l’emplacement et l’objet de requête de prédiction pour traiter la requête. Les autres options que sont verbose (mode détaillé), showAllIntents (afficher toutes les intentions) et log (journal) sont facultatives.

[!code-javascript [Get prediction from runtime](~/cognitive-services-quickstart-code/javascript/LUIS/sdk-3x/index.js?name=QueryPredictionEndpoint)]

[!INCLUDE [Prediction JSON response](sdk-json.md)]

## <a name="run-the-application"></a>Exécution de l'application

Exécutez l’application avec la commande `node index.js` de votre fichier de démarrage rapide.

```console
node index.js
```