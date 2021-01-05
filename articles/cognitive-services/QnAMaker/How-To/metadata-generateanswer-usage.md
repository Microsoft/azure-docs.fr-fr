---
title: Métadonnées avec l’API GenerateAnswer - QnA Maker
titleSuffix: Azure Cognitive Services
description: Avec QnA Maker, vous pouvez ajouter des métadonnées, sous la forme de paires clé-valeur, à vos paires question/réponse. Vous pouvez filtrer des résultats de requêtes d’utilisateurs et stocker des informations supplémentaires utilisables dans des conversations de suivi.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: c250868c9d470ee85f765f693aff3e21320fc45e
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96346186"
---
# <a name="get-an-answer-with-the-generateanswer-api-and-metadata"></a>Obtenir une réponse avec l’API GenerateAnswer et des métadonnées

Pour obtenir la réponse prédite à la question d’un utilisateur, utilisez l’API GenerateAnswer. Lorsque vous publiez une Base de connaissances, vous pouvez voir des informations sur l’utilisation de cette API dans la page **Publier**. Vous pouvez également configurer l’API pour filtrer les réponses en fonction des balises de métadonnées, et tester la Base de connaissances à partir du point de terminaison avec le paramètre de chaîne de requête de test.

Avec QnA Maker, vous pouvez ajouter des métadonnées, sous la forme de paires clé-valeur, à vos paires question/réponse. Vous pouvez ensuite utilise ces informations pour filtrer des résultats de requêtes d’utilisateurs et pour stocker des informations supplémentaires utilisables dans des conversations de suivi. Pour plus d’informations, consultez [Base de connaissances](../index.yml).

<a name="qna-entity"></a>

## <a name="store-questions-and-answers-with-a-qna-entity"></a>Stocker des questions et réponses avec une entité QnA

Il est important de bien comprendre de quelle façon QnA Maker stocke les données des questions et réponses. L’illustration suivante représente une entité QnA :

![Illustration d’une entité QnA](../media/qnamaker-how-to-metadata-usage/qna-entity.png)

Chaque entité QnA a un ID unique et persistant. Vous pouvez utiliser l’ID pour mettre à jour une entité QnA particulière.

<a name="generateanswer-api"></a>

## <a name="get-answer-predictions-with-the-generateanswer-api"></a>Obtenir des prédictions de réponse avec l’API GenerateAnswer

Utilisez l’[API GenerateAnswer](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer) dans votre bot ou application pour rechercher dans votre Base de connaissances une réponse à une question d’un utilisateur et obtenir la meilleure correspondance parmi les paires question/réponse.

<a name="generateanswer-endpoint"></a>

## <a name="publish-to-get-generateanswer-endpoint"></a>Publier pour obtenir un point de terminaison GenerateAnswer

Après avoir publié votre Base de connaissances à partir du [portail QnA Maker](https://www.qnamaker.ai), ou à l’aide de [l’API](/rest/api/cognitiveservices/qnamaker/knowledgebase/publish), vous pouvez obtenir les détails de votre point de terminaison GenerateAnswer.

Pour obtenir les détails de votre point de terminaison :
1. Connectez-vous à [https://www.qnamaker.ai](https://www.qnamaker.ai).
1. Dans **My knowledge bases** (Mes bases de connaissances), sélectionnez **View Code** (Afficher le code) sur la ligne de votre base de connaissances.
    ![Capture d’écran de My knowledge bases](../media/qnamaker-how-to-metadata-usage/my-knowledge-bases.png)
1. Examinez les détails de votre point de terminaison GenerateAnswer.

    # <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

    ![Capture d’écran des détails du point de terminaison](../media/qnamaker-how-to-metadata-usage/view-code.png)

    # <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

    ![Capture d’écran des détails du point de terminaison managés](../media/qnamaker-how-to-metadata-usage/view-code-managed.png)

    ---

Vous pouvez également obtenir les détails de votre point de terminaison à partir de l’onglet **Settings** (Paramètres) de votre base de connaissances.

<a name="generateanswer-request"></a>

## <a name="generateanswer-request-configuration"></a>Configuration de requête GenerateAnswer

Vous appelez GenerateAnswer à l’aide d’une requête HTTP POST. Pour obtenir un exemple de code qui montre comment appeler GenerateAnswer, consultez les [guides de démarrage rapide](../quickstarts/quickstart-sdk.md#generate-an-answer-from-the-knowledge-base).

La requête POST utilise :

* Obligatoirement des [paramètres d’URI](/rest/api/cognitiveservices/qnamakerruntime/runtime/train#uri-parameters)
* Obligatoirement une propriété d’en-tête, `Authorization`, pour la sécurité
* Obligatoirement des [propriétés de corps](/rest/api/cognitiveservices/qnamakerruntime/runtime/train#feedbackrecorddto).

Le format de l’URL GenerateAnswer est le suivant :

```
https://{QnA-Maker-endpoint}/knowledgebases/{knowledge-base-ID}/generateAnswer
```

N’oubliez pas de définir la propriété d’en-tête HTTP de `Authorization` avec une valeur de la chaîne `EndpointKey` et une espace final, puis la clé de point de terminaison figurant sur la page **Paramètres**.

Un exemple de corps JSON ressemble à ceci :

```json
{
    "question": "qna maker and luis",
    "top": 6,
    "isTest": true,
    "scoreThreshold": 30,
    "rankerType": "" // values: QuestionOnly
    "strictFilters": [
    {
        "name": "category",
        "value": "api"
    }],
    "userId": "sd53lsY="
}
```

En savoir plus sur [rankerType](../concepts/best-practices.md#choosing-ranker-type).

Le JSON précédent a demandé uniquement des réponses à 30 % ou plus par rapport au score de seuil.

<a name="generateanswer-response"></a>

## <a name="generateanswer-response-properties"></a>Propriétés de réponse GenerateAnswer

La [réponse](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer#successful-query) est un objet JSON incluant toutes les informations nécessaires pour afficher la réponse et l’intersection suivante dans la conversation, si elle est disponible.

```json
{
    "answers": [
        {
            "score": 38.54820341616869,
            "Id": 20,
            "answer": "There is no direct integration of LUIS with QnA Maker. But, in your bot code, you can use LUIS and QnA Maker together. [View a sample bot](https://github.com/Microsoft/BotBuilder-CognitiveServices/tree/master/Node/samples/QnAMaker/QnAWithLUIS)",
            "source": "Custom Editorial",
            "questions": [
                "How can I integrate LUIS with QnA Maker?"
            ],
            "metadata": [
                {
                    "name": "category",
                    "value": "api"
                }
            ]
        }
    ]
}
```

Le JSON précédent a fourni une réponse avec un score de 38,5 %.

## <a name="use-qna-maker-with-a-bot-in-c"></a>Utiliser QnA Maker avec un bot en C#

Bot Framework donne accès aux propriétés de QnA Maker avec l’[API getAnswer ](/dotnet/api/microsoft.bot.builder.ai.qna.qnamaker.getanswersasync?preserve-view=true&view=botbuilder-dotnet-stable#Microsoft_Bot_Builder_AI_QnA_QnAMaker_GetAnswersAsync_Microsoft_Bot_Builder_ITurnContext_Microsoft_Bot_Builder_AI_QnA_QnAMakerOptions_System_Collections_Generic_Dictionary_System_String_System_String__System_Collections_Generic_Dictionary_System_String_System_Double__):

```csharp
using Microsoft.Bot.Builder.AI.QnA;
var metadata = new Microsoft.Bot.Builder.AI.QnA.Metadata();
var qnaOptions = new QnAMakerOptions();

metadata.Name = Constants.MetadataName.Intent;
metadata.Value = topIntent;
qnaOptions.StrictFilters = new Microsoft.Bot.Builder.AI.QnA.Metadata[] { metadata };
qnaOptions.Top = Constants.DefaultTop;
qnaOptions.ScoreThreshold = 0.3F;
var response = await _services.QnAServices[QnAMakerKey].GetAnswersAsync(turnContext, qnaOptions);
```

Le JSON précédent a demandé uniquement des réponses à 30 % ou plus par rapport au score de seuil.

## <a name="use-qna-maker-with-a-bot-in-nodejs"></a>Utiliser QnA Maker avec un bot en Node.js

Bot Framework donne accès aux propriétés de QnA Maker avec l’[API getAnswer ](/javascript/api/botbuilder-ai/qnamaker?preserve-view=true&view=botbuilder-ts-latest#generateanswer-string---undefined--number--number-):

```javascript
const { QnAMaker } = require('botbuilder-ai');
this.qnaMaker = new QnAMaker(endpoint);

// Default QnAMakerOptions
var qnaMakerOptions = {
    ScoreThreshold: 0.30,
    Top: 3
};
var qnaResults = await this.qnaMaker.getAnswers(stepContext.context, qnaMakerOptions);
```

Le JSON précédent a demandé uniquement des réponses à 30 % ou plus par rapport au score de seuil.

<a name="metadata-example"></a>

## <a name="use-metadata-to-filter-answers-by-custom-metadata-tags"></a>Utiliser des métadonnées pour filtrer les réponses sur des balises de métadonnées personnalisées

L’ajout de métadonnées permet de filtrer les réponses sur ces balises de métadonnées. Ajoutez la colonne de métadonnées à partir du menu **Options d’affichage**. Ajoutez des métadonnées à votre Vase de connaissances en sélectionnant l’icône **+** des métadonnées pour ajouter une paire de métadonnées. Cette paire se compose d’une clé et une valeur.

![Capture d’écran de l’ajout de métadonnées](../media/qnamaker-how-to-metadata-usage/add-metadata.png)

<a name="filter-results-with-strictfilters-for-metadata-tags"></a>

## <a name="filter-results-with-strictfilters-for-metadata-tags"></a>Filtrer les résultats avec strictFilters pour les balises de métadonnées

Imaginez que la question de l’utilisateur soit « Quand ferme cet hôtel ? », où l’intention a implicitement trait au restaurant « Paradise ».

Étant donné que les résultats sont requis uniquement pour le restaurant « Paradise », vous pouvez définir un filtre dans l’appel GenerateAnswer sur les métadonnées du nom du restaurant. L’exemple suivant illustre cela :

```json
{
    "question": "When does this hotel close?",
    "top": 1,
    "strictFilters": [ { "name": "restaurant", "value": "paradise"}]
}
```

### <a name="logical-and-by-default"></a>AND logique par défaut

Pour combiner plusieurs filtres de métadonnées dans la requête, ajoutez les filtres de métadonnées supplémentaires au tableau de la propriété `strictFilters`. Par défaut, les valeurs sont combinées logiquement (AND). Une combinaison logique exige que tous les filtres correspondent aux paires de question/réponse pour que la paire soit retournée dans la réponse.

Cela équivaut à utiliser la propriété `strictFiltersCompoundOperationType` avec la valeur `AND`.

### <a name="logical-or-using-strictfilterscompoundoperationtype-property"></a>OR logique en utilisant la propriété strictFiltersCompoundOperationType

Quand plusieurs filtres de métadonnées sont combinés, si vous êtes seulement intéressé par la correspondance d’un ou quelques filtres, utilisez la propriété `strictFiltersCompoundOperationType` avec la valeur `OR`.

Votre base de connaissances retourne alors des réponses quand un filtre correspond, mais ne retourne pas de réponses dépourvues de métadonnées.

```json
{
    "question": "When do facilities in this hotel close?",
    "top": 1,
    "strictFilters": [
      { "name": "type","value": "restaurant"},
      { "name": "type", "value": "bar"},
      { "name": "type", "value": "poolbar"}
    ],
    "strictFiltersCompoundOperationType": "OR"
}
```

### <a name="metadata-examples-in-quickstarts"></a>Exemples de métadonnées dans les guides de démarrage rapide

Apprenez-en plus sur les métadonnées dans le guide de démarrage rapide du portail QnA Maker pour les métadonnées :
* [Création – Ajouter des métadonnées à une paire QnA](../quickstarts/add-question-metadata-portal.md#add-metadata-to-filter-the-answers)
* [Prédiction de requête – Filtrer les réponses par métadonnées](../quickstarts/get-answer-from-knowledge-base-using-url-tool.md)

<a name="keep-context"></a>

## <a name="use-question-and-answer-results-to-keep-conversation-context"></a>Utilisez les résultats de questions et réponses pour conserver le contexte de la conversation

La réponse à GenerateAnswer contient les informations de métadonnées correspondantes de la paire question/réponse trouvée. Vous pouvez utiliser ces informations pour stocker le contexte de la conversation précédente en vue d’une utilisation dans des conversations ultérieures.

```json
{
    "answers": [
        {
            "questions": [
                "What is the closing time?"
            ],
            "answer": "10.30 PM",
            "score": 100,
            "id": 1,
            "source": "Editorial",
            "metadata": [
                {
                    "name": "restaurant",
                    "value": "paradise"
                },
                {
                    "name": "location",
                    "value": "secunderabad"
                }
            ]
        }
    ]
}
```

## <a name="match-questions-only-by-text"></a>Faire correspondre les questions uniquement, par texte

Par défaut, QnA Maker recherche dans les questions et les réponses. Pour rechercher dans les questions uniquement afin de générer une réponse, utilisez le paramètre `RankerType=QuestionOnly` dans le corps POST de la requête GenerateAnswer.

Vous pouvez rechercher dans la Base de connaissances publiée à l’aide de `isTest=false`, ou dans la Base de connaissances de test à l’aide de `isTest=true`.

```json
{
  "question": "Hi",
  "top": 30,
  "isTest": true,
  "RankerType":"QuestionOnly"
}
```

## <a name="common-http-errors"></a>Erreurs HTTP courantes

|Code|Explication|
|:--|--|
|2xx|Succès|
|400|Les paramètres de la requête sont incorrects, ce qui signifie que les paramètres obligatoires sont manquants, incorrects ou trop grands|
|400|Le corps de la requête est incorrect, ce qui signifie que l’objet JSON est manquant, mal formé ou trop grand|
|401|Clé non valide|
|403|Interdit : vous n’avez pas les autorisations appropriées|
|404|La base de connaissances n’existe pas|
|410|Cette API est déconseillée et n’est plus disponible.|

## <a name="next-steps"></a>Étapes suivantes

La page **Publier** fournit aussi des informations pour [générer une réponse](../Quickstarts/get-answer-from-knowledge-base-using-url-tool.md) avec Postman ou cURL.

> [!div class="nextstepaction"]
> [Obtenir une analyse de votre base de connaissances](../how-to/get-analytics-knowledge-base.md)