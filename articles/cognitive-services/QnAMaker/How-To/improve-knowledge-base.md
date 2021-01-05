---
title: Questions suggérées pour l’apprentissage actif – QnA Maker
description: Améliorez la qualité de votre base de connaissances avec l'apprentissage actif. Examinez, acceptez ou refusez, ajoutez sans supprimer ou modifier les questions existantes.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 04/06/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: a77083c721328a8df9facd2f4b320b863aa03c68
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352403"
---
# <a name="accept-active-learning-suggested-questions-in-the-knowledge-base"></a>Accepter des questions suggérées d’apprentissage actif dans la base de connaissances


<a name="accept-an-active-learning-suggestion-in-the-knowledge-base"></a>

L’apprentissage actif modifie la base de connaissances ou Search Service après approbation de la suggestion, puis elle est enregistrée et formée. Si vous approuvez la suggestion, elle est ajoutée en tant que question de substitution.

## <a name="turn-on-active-learning"></a>Activation de l’apprentissage actif

Pour consulter les questions suggérées, vous devez [activer l'apprentissage actif](../concepts/active-learning-suggestions.md) pour votre ressource QnA Maker.

## <a name="view-suggested-questions"></a>Afficher les questions suggérées

1. Pour afficher les suggestions de questions, sur la page **Modifier** de la base de connaissances, sélectionnez **Afficher les options**, puis **Show active learning suggestions** (Afficher les suggestions d’apprentissage actif).

    [![Dans la section Modifier du portail, sélectionnez l’option d’affichage des suggestions afin de voir les nouvelles alternatives de question d’apprentissage actif.](../media/improve-knowledge-base/show-suggestions-button.png)](../media/improve-knowledge-base/show-suggestions-button.png#lightbox)

1. Filtrez la base de connaissances contenant des paires de questions-réponses pour afficher uniquement les suggestions en sélectionnant **Filter by Suggestions** (Filtrer par suggestions).

    [![Utilisez le filtre par suggestions pour afficher uniquement les alternatives de question suggérées d’apprentissage actif.](../media/improve-knowledge-base/filter-by-suggestions.png)](../media/improve-knowledge-base/filter-by-suggestions.png#lightbox)

1. Chaque paire de questions-réponses suggère de nouvelles alternatives de question avec une coche, `✔`, pour les accepter ou avec un symbole `x` pour rejeter les suggestions. Sélectionnez la coche pour ajouter la question.

    [![Sélectionnez ou rejeter les alternatives de question suggérées d’apprentissage actif en sélectionnant la coche verte ou le symbole de suppression rouge.](../media/improve-knowledge-base/accept-active-learning-suggestions-small.png)](../media/improve-knowledge-base/accept-active-learning-suggestions.png#lightbox)

    Vous pouvez ajouter ou supprimer _toutes les suggestions_ en sélectionnant **Ajouter tout** ou **Rejeter tout** dans la barre d’outils contextuelle.

1. Sélectionnez **Save and Train** (Enregistrer et effectuer l’apprentissage) pour enregistrer les modifications apportées à la base de connaissances.

1. Sélectionnez **Publier** pour que les modifications soient disponibles dans [l’API GenerateAnswer](metadata-generateanswer-usage.md#generateanswer-request-configuration).

    Lorsque 5 requêtes similaires ou plus sont ordonnées en cluster, toutes les 30 minutes, QnA Maker suggère les questions alternatives pour que vous les acceptiez ou les refusiez.


<a name="#score-proximity-between-knowledge-base-questions"></a>

### <a name="architectural-flow-for-using-generateanswer-and-train-apis-from-a-bot"></a>Flux architectural pour utiliser les API Train et GenerateAnswer à partir d’un bot

Un bot ou une autre application cliente doivent utiliser le flux architectural suivant pour utiliser l’apprentissage actif :

* Le bot [obtient la réponse de la base de connaissances](#use-the-top-property-in-the-generateanswer-request-to-get-several-matching-answers) avec l’API GenerateAnswer, à l’aide de la propriété `top` pour obtenir un certain nombre de réponses.
* Le bot détermine des commentaires explicites :
    * À l’aide de votre [logique métier personnalisée](#use-the-score-property-along-with-business-logic-to-get-list-of-answers-to-show-user), filtrez les scores faibles.
    * Dans le bot ou l’application cliente, affichez la liste de réponses possibles pour l’utilisateur et obtenez la réponse sélectionnée par l’utilisateur.
* Le bot [envoie la réponse sélectionnée à QnA Maker](#bot-framework-sample-code) avec [l’API Train](#train-api).


### <a name="use-the-top-property-in-the-generateanswer-request-to-get-several-matching-answers"></a>Utilisation de la propriété principale dans la requête GenerateAnswer pour obtenir plusieurs réponses correspondantes

Lorsque vous soumettez une question à QnA Maker pour obtenir une réponse, la propriété `top` du corps JSON définit le nombre de réponses à renvoyer.

```json
{
    "question": "wi-fi",
    "isTest": false,
    "top": 3
}
```

### <a name="use-the-score-property-along-with-business-logic-to-get-list-of-answers-to-show-user"></a>Utiliser la propriété de score, ainsi que la logique métier pour obtenir la liste de réponses à afficher pour l’utilisateur

Lorsque l’application cliente (par exemple, un bot conversationnel) reçoit la réponse, les 3 questions principales sont renvoyées. Utilisez la propriété `score` pour analyser la proximité entre les scores. Cette plage de proximité est déterminée par votre logique métier.

```json
{
    "answers": [
        {
            "questions": [
                "Wi-Fi Direct Status Indicator"
            ],
            "answer": "**Wi-Fi Direct Status Indicator**\n\nStatus bar icons indicate your current Wi-Fi Direct connection status:  \n\nWhen your device is connected to another device using Wi-Fi Direct, '$  \n\n+ *+ ' Wi-Fi Direct is displayed in the Status bar.",
            "score": 74.21,
            "id": 607,
            "source": "Bugbash KB.pdf",
            "metadata": []
        },
        {
            "questions": [
                "Wi-Fi - Connections"
            ],
            "answer": "**Wi-Fi**\n\nWi-Fi is a term used for certain types of Wireless Local Area Networks (WLAN). Wi-Fi communication requires access to a wireless Access Point (AP).",
            "score": 74.15,
            "id": 599,
            "source": "Bugbash KB.pdf",
            "metadata": []
        },
        {
            "questions": [
                "Turn Wi-Fi On or Off"
            ],
            "answer": "**Turn Wi-Fi On or Off**\n\nTurning Wi-Fi on makes your device able to discover and connect to compatible in-range wireless APs.  \n\n1.  From a Home screen, tap ::: Apps > e Settings .\n2.  Tap Connections > Wi-Fi , and then tap On/Off to turn Wi-Fi on or off.",
            "score": 69.99,
            "id": 600,
            "source": "Bugbash KB.pdf",
            "metadata": []
        }
    ]
}
```

## <a name="client-application-follow-up-when-questions-have-similar-scores"></a>Suivi de l’application cliente lorsque les questions présentent des scores similaires

Votre application cliente affiche les questions avec une option permettant à l’utilisateur de sélectionner _la seule question_ qui représente le mieux son intention.

Une fois que l’utilisateur sélectionne l’une des questions existantes, l’application cliente envoie le choix de l’utilisateur en tant que commentaires à l’aide de l’API Train de QnA Maker. Ces commentaires terminent la boucle de commentaires de l’apprentissage actif.

## <a name="train-api"></a>API Train

Les commentaires de l’apprentissage actif sont envoyés à QnA Maker avec la requête POST de l’API Train. La signature d’API est :

```http
POST https://<QnA-Maker-resource-name>.azurewebsites.net/qnamaker/knowledgebases/<knowledge-base-ID>/train
Authorization: EndpointKey <endpoint-key>
Content-Type: application/json
{"feedbackRecords": [{"userId": "1","userQuestion": "<question-text>","qnaId": 1}]}
```

|Propriété de requête HTTP|Nom|Type|Objectif|
|--|--|--|--|
|Paramètre d’itinéraire d’URL|ID de base de connaissances|string|GUID de votre base de connaissances.|
|Sous-domaine personnalisé|Nom de la ressource QnAMaker|string|Le nom de la ressource est utilisé en tant que sous-domaine personnalisé pour votre service QnA Maker. Cela se trouve sur la page Paramètres une fois la base de connaissances publiée. Il est indiqué en tant que `host`.|
|En-tête|Content-Type|string|type de média du corps envoyé à l’API. La valeur par défaut est: `application/json`|
|En-tête|Autorisation|string|clé de votre point de terminaison (EndpointKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx).|
|Corps de publication|Objet JSON|JSON|Commentaires de formation|

Le corps JSON a plusieurs paramètres :

|Propriété du corps JSON|Type|Objectif|
|--|--|--|--|
|`feedbackRecords`|tableau|Liste des commentaires.|
|`userId`|string|ID utilisateur de la personne acceptant les questions suggérées. Le choix du format d’ID utilisateur dépend de vous. Par exemple, une adresse e-mail peut être un ID utilisateur valide dans votre architecture. facultatif.|
|`userQuestion`|string|Texte exact de la requête de l’utilisateur. Obligatoire.|
|`qnaID`|nombre|ID de la question, trouvée dans la [réponse GenerateAnswer](metadata-generateanswer-usage.md#generateanswer-response-properties). |

Un exemple de corps JSON ressemble à ceci :

```json
{
    "feedbackRecords": [
        {
            "userId": "1",
            "userQuestion": "<question-text>",
            "qnaId": 1
        }
    ]
}
```

Une réponse réussie renvoie un état 204 et aucun corps de réponse JSON.

### <a name="batch-many-feedback-records-into-a-single-call"></a>Traiter autant d’enregistrements de commentaires que possible dans un seul appel

Dans l’application côté client, par exemple un bot, vous pouvez stocker les données, puis envoyer un grand nombre d’enregistrements dans un seul corps JSON dans le tableau `feedbackRecords`.

Un exemple de corps JSON ressemble à ceci :

```json
{
    "feedbackRecords": [
        {
            "userId": "1",
            "userQuestion": "How do I ...",
            "qnaId": 1
        },
        {
            "userId": "2",
            "userQuestion": "Where is ...",
            "qnaId": 40
        },
        {
            "userId": "3",
            "userQuestion": "When do I ...",
            "qnaId": 33
        }
    ]
}
```



<a name="active-learning-is-saved-in-the-exported-apps-tsv-file"></a>

## <a name="bot-framework-sample-code"></a>Exemple de code d’infrastructure de bot

Votre code d’infrastructure de bot doit appeler l’API Train si la requête de l’utilisateur doit être utilisée pour l’apprentissage actif. Deux éléments de code doivent être écrits :

* Déterminer si la requête doit être utilisée pour l’apprentissage actif
* Renvoyer la requête à l’API Train de QnA Maker pour l’apprentissage actif

Dans [l’exemple de bot Azure](https://github.com/microsoft/BotBuilder-Samples), ces deux activités ont été programmées.

### <a name="example-c-code-for-train-api-with-bot-framework-4x"></a>Exemple de code C# pour l’API Train avec Bot Framework 4.x

Le code suivant illustre comment renvoyer des informations à QnA Maker avec l’API Train.

```csharp
public class FeedbackRecords
{
    // <summary>
    /// List of feedback records
    /// </summary>
    [JsonProperty("feedbackRecords")]
    public FeedbackRecord[] Records { get; set; }
}

/// <summary>
/// Active learning feedback record
/// </summary>
public class FeedbackRecord
{
    /// <summary>
    /// User id
    /// </summary>
    public string UserId { get; set; }

    /// <summary>
    /// User question
    /// </summary>
    public string UserQuestion { get; set; }

    /// <summary>
    /// QnA Id
    /// </summary>
    public int QnaId { get; set; }
}

/// <summary>
/// Method to call REST-based QnAMaker Train API for Active Learning
/// </summary>
/// <param name="endpoint">Endpoint URI of the runtime</param>
/// <param name="FeedbackRecords">Feedback records train API</param>
/// <param name="kbId">Knowledgebase Id</param>
/// <param name="key">Endpoint key</param>
/// <param name="cancellationToken"> Cancellation token</param>
public async static void CallTrain(string endpoint, FeedbackRecords feedbackRecords, string kbId, string key, CancellationToken cancellationToken)
{
    var uri = endpoint + "/knowledgebases/" + kbId + "/train/";

    using (var client = new HttpClient())
    {
        using (var request = new HttpRequestMessage())
        {
            request.Method = HttpMethod.Post;
            request.RequestUri = new Uri(uri);
            request.Content = new StringContent(JsonConvert.SerializeObject(feedbackRecords), Encoding.UTF8, "application/json");
            request.Headers.Add("Authorization", "EndpointKey " + key);

            var response = await client.SendAsync(request, cancellationToken);
            await response.Content.ReadAsStringAsync();
        }
    }
}
```

### <a name="example-nodejs-code-for-train-api-with-bot-framework-4x"></a>Exemple de code Node.js pour l’API Train avec Bot Framework 4.x

Le code suivant illustre comment renvoyer des informations à QnA Maker avec l’API Train.

```javascript
async callTrain(stepContext){

    var trainResponses = stepContext.values[this.qnaData];
    var currentQuery = stepContext.values[this.currentQuery];

    if(trainResponses.length > 1){
        var reply = stepContext.context.activity.text;
        var qnaResults = trainResponses.filter(r => r.questions[0] == reply);

        if(qnaResults.length > 0){

            stepContext.values[this.qnaData] = qnaResults;

            var feedbackRecords = {
                FeedbackRecords:[
                    {
                        UserId:stepContext.context.activity.id,
                        UserQuestion: currentQuery,
                        QnaId: qnaResults[0].id
                    }
                ]
            };

            // Call Active Learning Train API
            this.activeLearningHelper.callTrain(this.qnaMaker.endpoint.host, feedbackRecords, this.qnaMaker.endpoint.knowledgeBaseId, this.qnaMaker.endpoint.endpointKey);

            return await stepContext.next(qnaResults);
        }
        else{

            return await stepContext.endDialog();
        }
    }

    return await stepContext.next(stepContext.result);
}
```

## <a name="active-learning-is-saved-in-the-exported-knowledge-base"></a>L’apprentissage actif est enregistré dans la base de connaissances exportée

Lorsque l’apprentissage actif est activé pour votre application et que vous exportez l’application, la colonne `SuggestedQuestions` dans le fichier tsv conserve les données d’apprentissage actif.

La colonne `SuggestedQuestions` est un objet JSON d’informations des commentaires implicites, `autosuggested`, et explicites, `usersuggested`. Un exemple de cet objet JSON pour une seule question soumise par l’utilisateur de `help` est :

```JSON
[
    {
        "clusterHead": "help",
        "totalAutoSuggestedCount": 1,
        "totalUserSuggestedCount": 0,
        "alternateQuestionList": [
            {
                "question": "help",
                "autoSuggestedCount": 1,
                "userSuggestedCount": 0
            }
        ]
    }
]
```

Lorsque vous réimportez cette application, l’apprentissage actif continue à collecter des informations et à recommander des suggestions pour votre base de connaissances.



## <a name="best-practices"></a>Meilleures pratiques

Pour prendre connaissance des meilleures pratiques lors de l’utilisation de l’apprentissage actif, consultez la rubrique [Bonnes pratiques](../Concepts/best-practices.md#active-learning).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Utiliser des métadonnées avec l’API GenerateAnswer](metadata-generateanswer-usage.md)