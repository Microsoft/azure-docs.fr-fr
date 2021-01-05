---
title: Importer des énoncés à l’aide de Node.js - LUIS
titleSuffix: Azure Cognitive Services
description: Découvrez comment créer une application LUIS par programmation à partir de données préexistantes au format CSV à l’aide de l’API de création LUIS.
services: cognitive-services
manager: nitinme
ms.custom: seodec18, devx-track-js
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 09/05/2019
ms.openlocfilehash: 58eb92f4d0bc3de4671ca2ece14a178a876e4a6b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91541037"
---
# <a name="build-a-luis-app-programmatically-using-nodejs"></a>Créer une application LUIS par programmation à l’aide de Node.js

LUIS fournit une API de programmation qui fait tout ce que le site web [LUIS](luis-reference-regions.md) fait. Ceci permet de gagner du temps lorsque vous avez des données préexistantes et qu’il serait plus rapide de créer une application LUIS par programmation que d’entrer manuellement des informations.

[!INCLUDE [Waiting for LUIS portal refresh](./includes/wait-v3-upgrade.md)]

## <a name="prerequisites"></a>Conditions préalables requises

* Connectez-vous au site web [LUIS](luis-reference-regions.md) et recherchez votre [clé de création](luis-how-to-azure-subscription.md#authoring-key) dans les paramètres du compte. Cette clé vous permet d’appeler les API de création.
* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer.
* Cet article commence par un volume partagé de cluster pour les fichiers journaux d’une société fictive de requêtes d’utilisateurs. Téléchargez-le [ici](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/IoT.csv).
* Installez la dernière version de Node.js avec NPM. Téléchargez-la [ici](https://nodejs.org/en/download/).
* **[Recommandé]**  Visual Studio Code pour IntelliSense et le débogage, à télécharger gratuitement [ici](https://code.visualstudio.com/).

Tout le code utilisé dans cet article est disponible dans le [dépôt GitHub d’exemples Azure Language Understanding](https://github.com/Azure-Samples/cognitive-services-language-understanding/tree/master/examples/build-app-programmatically-csv).

## <a name="map-preexisting-data-to-intents-and-entities"></a>Mapper des données préexistantes à des intentions et à des entités
Même si vous envisagez un système qui n’a pas été créé avec LUIS, s’il contient des données textuelles mappées à différentes choses que les utilisateurs souhaitent faire, vous pouvez parvenir à mapper des catégories existantes d’entrées d’utilisateurs à des intentions dans LUIS. Si vous pouvez identifier des mots ou des expressions importantes dans ce que les utilisateurs disent, ces mots peuvent être mappés à des entités.

Ouvrez le fichier [`IoT.csv`](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/IoT.csv). Il contient un journal des requêtes d’utilisateurs à un service de domotique hypothétique, y compris la façon dont elles ont été classées, ce que l’utilisateur a dit, et certaines colonnes dont des informations utiles sont tirées.

![Fichier CSV de données préexistantes](./media/luis-tutorial-node-import-utterances-csv/csv.png)

Vous voyez que la colonne **RequestType** pourrait représenter des intentions et que la colonne **Request** montre un exemple d’énoncé. Les autres champs pourraient être des entités s’ils sont présents dans l’énoncé. Puisque vous avez des intentions, des entités et des exemples d’énoncés, vous remplissez les conditions préalables pour créer un exemple d’application simple.

## <a name="steps-to-generate-a-luis-app-from-non-luis-data"></a>Étapes pour générer une application LUIS à partir de données externes à LUIS
Pour créer une nouvelle application LUIS à partir du fichier CSV :

* Analysez les données du fichier CSV :
    * Convertissez-le dans un format que vous pouvez charger sur LUIS à l’aide de l’API de création.
    * Dans les données analysées, recueillez des informations sur les intentions et les entités.
* Effectuez des appels d’API de création à :
    * Créez l'application.
    * Ajoutez des intentions et des entités collectées à partir des données analysées.
    * Une fois que vous avez créé l’application LUIS, vous pouvez ajouter les exemples d’énoncés à partir des données analysées.

Vous pouvez voir ce flux de programme dans la dernière partie du fichier `index.js`. Copiez ou [téléchargez](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/index.js) ce code et enregistrez-le dans `index.js`.

   [!code-javascript[Node.js code for calling the steps to build a LUIS app](~/samples-luis/examples/build-app-programmatically-csv/index.js)]


## <a name="parse-the-csv"></a>Analyser les volumes partagés de cluster

Les entrées des colonnes qui contiennent les énoncés dans le volume partagé de cluster doivent être analysées dans un format JSON que LUIS peut comprendre. Ce format JSON doit contenir un champ `intentName` qui identifie l’intention de l’énoncé. Il doit également contenir un champ `entityLabels`, qui peut être vide s’il n’y a aucune entité dans l’énoncé.

Par exemple, l’entrée « Allumer les lumières » est mappée à ce fichier JSON :

```json
        {
            "text": "Turn on the lights",
            "intentName": "TurnOn",
            "entityLabels": [
                {
                    "entityName": "Operation",
                    "startCharIndex": 5,
                    "endCharIndex": 6
                },
                {
                    "entityName": "Device",
                    "startCharIndex": 12,
                    "endCharIndex": 17
                }
            ]
        }
```

Dans cet exemple, le `intentName` provient de la requête de l’utilisateur sous l’en-tête de colonne **Requête** dans le fichier CSV, et le `entityName` provient des autres colonnes avec les informations sur la clé. Par exemple, s’il y a une entrée pour **Opération** ou pour **Appareil** et que ce string apparaît également dans la requête proprement dite, elle peut être étiquetée comme entité. Le code suivant illustre ce processus d’analyse. Vous pouvez le copier ou le [télécharger](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_parse.js) et l’enregistrer sous `_parse.js`.

   [!code-javascript[Node.js code for parsing a CSV file to extract intents, entities, and labeled utterances](~/samples-luis/examples/build-app-programmatically-csv/_parse.js)]



## <a name="create-the-luis-app"></a>Créer l’application LUIS
Une fois que les données ont été analysées dans JSON, ajoutez-les à une application LUIS. Le code suivant crée l’application LUIS. Copiez-le ou [téléchargez-le](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_create.js), puis enregistrez-le dans `_create.js`.

   [!code-javascript[Node.js code for creating a LUIS app](~/samples-luis/examples/build-app-programmatically-csv/_create.js)]


## <a name="add-intents"></a>Ajouter des intentions
Une fois que vous avez une application, vous devez lui assigner des intentions. Le code suivant crée l’application LUIS. Copiez-le ou [téléchargez-le](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_intents.js), puis enregistrez-le dans `_intents.js`.

   [!code-javascript[Node.js code for creating a series of intents](~/samples-luis/examples/build-app-programmatically-csv/_intents.js)]


## <a name="add-entities"></a>Ajouter des entités
Le code suivant ajoute les entités à l’application LUIS. Copiez-le ou [téléchargez-le](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_entities.js), puis enregistrez-le dans `_entities.js`.

   [!code-javascript[Node.js code for creating entities](~/samples-luis/examples/build-app-programmatically-csv/_entities.js)]



## <a name="add-utterances"></a>Ajouter des énoncés
Une fois que les entités et les intentions ont été définies dans l’application LUIS, vous pouvez ajouter les énoncés. Le code suivant utilise l’API [Utterances_AddBatch](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09), qui vous permet d’ajouter jusqu'à 100 énoncés à la fois.  Copiez-le ou [téléchargez-le](https://github.com/Azure-Samples/cognitive-services-language-understanding/blob/master/examples/build-app-programmatically-csv/_upload.js), puis enregistrez-le dans `_upload.js`.

   [!code-javascript[Node.js code for adding utterances](~/samples-luis/examples/build-app-programmatically-csv/_upload.js)]


## <a name="run-the-code"></a>Exécuter le code


### <a name="install-nodejs-dependencies"></a>Installer les dépendances de Node.js
Installez les dépendances de Node.js à partir de NPM dans le terminal/la ligne de commande.

```console
> npm install
```

### <a name="change-configuration-settings"></a>Modifier les paramètres de configuration
Pour pouvoir utiliser cette application, vous devez modifier les valeurs dans le fichier index.js en fonction de votre propre clé de point de terminaison et indiquer le nom que vous souhaitez donner à l’application. Vous pouvez également définir la culture de l’application ou modifier le numéro de version.

Ouvrez le fichier index.js et modifiez ces valeurs en haut du fichier.


```javascript
// Change these values
const LUIS_programmaticKey = "YOUR_AUTHORING_KEY";
const LUIS_appName = "Sample App";
const LUIS_appCulture = "en-us";
const LUIS_versionId = "0.1";
```

### <a name="run-the-script"></a>Exécuter le script
Exécutez le script à partir d’un terminal/d’une ligne de commande Node.js.

```console
> node index.js
```

or

```console
> npm start
```

### <a name="application-progress"></a>Progression de l’application
Pendant que l’application est en cours d’exécution, la ligne de commande indique la progression. La sortie de la ligne de commande inclut le format des réponses de LUIS.

```console
> node index.js
intents: ["TurnOn","TurnOff","Dim","Other"]
entities: ["Operation","Device","Room"]
parse done
JSON file should contain utterances. Next step is to create an app with the intents and entities it found.
Called createApp, created app with ID 314b306c-0033-4e09-92ab-94fe5ed158a2
Called addIntents for intent named TurnOn.
Called addIntents for intent named TurnOff.
Called addIntents for intent named Dim.
Called addIntents for intent named Other.
Results of all calls to addIntent = [{"response":"e7eaf224-8c61-44ed-a6b0-2ab4dc56f1d0"},{"response":"a8a17efd-f01c-488d-ad44-a31a818cf7d7"},{"response":"bc7c32fc-14a0-4b72-bad4-d345d807f965"},{"response":"727a8d73-cd3b-4096-bc8d-d7cfba12eb44"}]
called addEntity for entity named Operation.
called addEntity for entity named Device.
called addEntity for entity named Room.
Results of all calls to addEntity= [{"response":"6a7e914f-911d-4c6c-a5bc-377afdce4390"},{"response":"56c35237-593d-47f6-9d01-2912fa488760"},{"response":"f1dd440c-2ce3-4a20-a817-a57273f169f3"}]
retrying add examples...

Results of add utterances = [{"response":[{"value":{"UtteranceText":"turn on the lights","ExampleId":-67649},"hasError":false},{"value":{"UtteranceText":"turn the heat on","ExampleId":-69067},"hasError":false},{"value":{"UtteranceText":"switch on the kitchen fan","ExampleId":-3395901},"hasError":false},{"value":{"UtteranceText":"turn off bedroom lights","ExampleId":-85402},"hasError":false},{"value":{"UtteranceText":"turn off air conditioning","ExampleId":-8991572},"hasError":false},{"value":{"UtteranceText":"kill the lights","ExampleId":-70124},"hasError":false},{"value":{"UtteranceText":"dim the lights","ExampleId":-174358},"hasError":false},{"value":{"UtteranceText":"hi how are you","ExampleId":-143722},"hasError":false},{"value":{"UtteranceText":"answer the phone","ExampleId":-69939},"hasError":false},{"value":{"UtteranceText":"are you there","ExampleId":-149588},"hasError":false},{"value":{"UtteranceText":"help","ExampleId":-81949},"hasError":false},{"value":{"UtteranceText":"testing the circuit","ExampleId":-11548708},"hasError":false}]}]
upload done
```




## <a name="open-the-luis-app"></a>Ouvrir l’application LUIS
Une fois le script terminé, vous pouvez vous connecter à [LUIS](luis-reference-regions.md) et voir l’application LUIS que vous avez créée sous **Mes applications**. Vous devriez voir les énoncés que vous avez ajoutés sous les intentions **TurnOn**, **TurnOff** et **None**.

![Intention TurnOn](./media/luis-tutorial-node-import-utterances-csv/imported-utterances-661.png)


## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tester votre application et effectuez son apprentissage sur le site web de LUIS](luis-interactive-test.md)

## <a name="additional-resources"></a>Ressources supplémentaires

Cet exemple d’application utilise les API suivantes de LUIS :
- [créer une application](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c36)
- [ajouter des intentions](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0c)
- [ajouter des entités](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0e)
- [ajouter des énoncés](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09)
