---
title: Interroger une base de connaissances – QnA Maker
description: Une base de connaissances doit être publiée. Une fois publiée, la base de connaissances est interrogée au point de terminaison de prédiction du runtime à l’aide de l’API generateAnswer.
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: d8f986299edee46bf5cace7a9f4c805c29b3ce0c
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96346203"
---
# <a name="query-the-knowledge-base-for-answers"></a>Interroger la base de connaissances pour obtenir des réponses

Une base de connaissances doit être publiée. Une fois publiée, la base de connaissances est interrogée au point de terminaison de prédiction du runtime à l’aide de l’API generateAnswer. La requête comprend le texte de la question et d’autres paramètres pour aider QnA Maker à sélectionner la meilleure correspondance possible à une réponse.

## <a name="how-qna-maker-processes-a-user-query-to-select-the-best-answer"></a>Comment QnA Maker traite une requête de l’utilisateur pour sélectionner la meilleure réponse

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Le base de connaissances QnA Maker entraînée et [publiée](../quickstarts/create-publish-knowledge-base.md#publish-the-knowledge-base) reçoit une requête de l’utilisateur, à partir d’un robot ou d’une autre application cliente, au niveau de l’[API GenerateAnswer](../how-to/metadata-generateanswer-usage.md). Le diagramme suivant illustre le processus enclenché lors de la réception de la requête de l’utilisateur.

![Processus de modèle de classement par ordre de priorité pour une requête de l’utilisateur](../media/qnamaker-concepts-knowledgebase/ranker-v1.png)

### <a name="ranker-process"></a>Processus de classement

Le processus est expliqué dans le tableau suivant.

|Étape|Objectif|
|--|--|
|1|L’application cliente envoie la requête de l’utilisateur vers l’[API GenerateAnswer](../how-to/metadata-generateanswer-usage.md).|
|2|QnA Maker prétraite la requête de l’utilisateur avec détection de la langue, vérificateurs d’orthographe et analyseurs lexicaux.|
|3|Ce prétraitement modifie la requête utilisateur afin d’offrir des résultats de recherche optimaux.|
|4|Cette requête modifiée est envoyée à l’Index de Recherche cognitive Azure qui reçoit `top` résultats. Si la réponse correcte n’est pas dans ces résultats, augmentez légèrement la valeur de `top`. En règle générale, une valeur de 10 pour `top` fonctionne dans 90 % des requêtes.|
|5|QnA Maker utilise des caractérisation syntaxiques et sémantiques pour déterminer la similarité entre la requête de l’utilisateur et les résultats QnA récupérés.|
|6|Le modèle d’outil de classement issu de l’apprentissage automatique utilise les différentes fonctionnalités, à partir de l’étape 5, pour déterminer les scores de confiance et le nouvel ordre de classement.|
|7|Les nouveaux résultats sont renvoyés vers l’application cliente selon l’ordre de classement.|
|||

Les fonctionnalités utilisées incluent notamment la sémantique au niveau des mots, l’importance des termes dans un corpus et les modèles sémantiques ayant fait l’objet d’un apprentissage profond pour déterminer la similarité et la pertinence entre deux chaînes de texte.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Le base de connaissances QnA Maker entraînée et [publiée](../quickstarts/create-publish-knowledge-base.md#publish-the-knowledge-base) reçoit une requête de l’utilisateur, à partir d’un robot ou d’une autre application cliente, au niveau de l’[API GenerateAnswer](../how-to/metadata-generateanswer-usage.md). Le diagramme suivant illustre le processus enclenché lors de la réception de la requête de l’utilisateur.

![Processus de modèle de classement par ordre de priorité pour une requête de l’utilisateur (préversion)](../media/qnamaker-concepts-knowledgebase/ranker-v2.png)

### <a name="ranker-process"></a>Processus de classement

Le processus est expliqué dans le tableau suivant.

|Étape|Objectif|
|--|--|
|1|L’application cliente envoie la requête de l’utilisateur vers l’[API GenerateAnswer](../how-to/metadata-generateanswer-usage.md).|
|2|QnA Maker prétraite la requête de l’utilisateur avec détection de la langue, vérificateurs d’orthographe et analyseurs lexicaux.|
|3|Ce prétraitement modifie la requête utilisateur afin d’offrir des résultats de recherche optimaux.|
|4|Cette requête modifiée est envoyée à l’Index de Recherche cognitive Azure qui reçoit `top` résultats. Si la réponse correcte n’est pas dans ces résultats, augmentez légèrement la valeur de `top`. En règle générale, une valeur de 10 pour `top` fonctionne dans 90 % des requêtes.|
|5|QnA Maker utilise un modèle basé sur un transformateur de pointe pour déterminer la similarité entre la requête utilisateur et les résultats QnA candidats extraits de Recherche cognitive Azure. Le modèle basé sur un transformateur est un modèle multilingue de deep learning, qui fonctionne horizontalement pour toutes les langues afin de déterminer les scores de confiance et le nouvel ordre de classement.|
|6|Les nouveaux résultats sont renvoyés vers l’application cliente selon l’ordre de classement.|
|||

Le ranker travaille sur toutes les autres questions et réponses pour trouver les paires QnA les plus adaptées à la requête utilisateur. Les utilisateurs ont la possibilité de configurer le ranker pour qu’il ne travaille que sur les questions. 

---

## <a name="http-request-and-response-with-endpoint"></a>Requête et réponse HTTP avec point de terminaison
Lorsque vous publiez votre base de connaissance, le service crée un point de terminaison HTTP basé sur REST pouvant être intégré dans votre application, généralement un bot conversationnel.

### <a name="the-user-query-request-to-generate-an-answer"></a>Demande de requête de l’utilisateur pour générer une réponse

Une requête utilisateur est la question que l’utilisateur final pose à la base de connaissances, telle que `How do I add a collaborator to my app?`. Cette requête est souvent dans un format de langage naturel ou composée de quelques mots clés qui représentent la question, tels que `help with collaborators`. La requête est envoyée à votre base de connaissances à partir d’une requête HTTP dans votre application cliente.

```json
{
    "question": "How do I add a collaborator to my app?",
    "top": 6,
    "isTest": true,
    "scoreThreshold": 20,
    "strictFilters": [
    {
        "name": "QuestionType",
        "value": "Support"
    }],
    "userId": "sd53lsY="
}
```

Vous contrôlez la réponse en définissant des propriétés telles que[scoreThreshold](./confidence-score.md#choose-a-score-threshold), [top](../how-to/improve-knowledge-base.md#use-the-top-property-in-the-generateanswer-request-to-get-several-matching-answers), et [strictFilters](../how-to/metadata-generateanswer-usage.md#filter-results-with-strictfilters-for-metadata-tags).

Utilisez [le contenu de la conversation](../how-to/metadata-generateanswer-usage.md#use-question-and-answer-results-to-keep-conversation-context) avec [la fonctionnalité multitour](../how-to/multiturn-conversation.md) pour que la conversation se poursuive afin d’affiner les questions et les réponses pour trouver la réponse correcte et définitive.

### <a name="the-response-from-a-call-to-generate-an-answer"></a>Réponse d’un appel pour générer une réponse

La réponse HTTP est la réponse extraite de la base de données. Il s’agit de la meilleure correspondance à une requête utilisateur donnée. La réponse comprend la solution et le score de prédiction. Si vous avez demandé plusieurs réponses principales, avec la propriété `top`, vous obtenez plus d’une réponse principale, chacune avec un score.

```json
{
    "answers": [
        {
            "questions": [
                "How do I add a collaborator to my app?",
                "What access control is provided for the app?",
                "How do I find user management and security?"
            ],
            "answer": "Use the Azure portal to add a collaborator using Access Control (IAM)",
            "score": 100,
            "id": 1,
            "source": "Editorial",
            "metadata": [
                {
                    "name": "QuestionType",
                    "value": "Support"
                },
                {
                    "name": "ToolDependency",
                    "value": "Azure Portal"
                }
            ]
        }
    ]
}
```


## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Score de confiance](./confidence-score.md)