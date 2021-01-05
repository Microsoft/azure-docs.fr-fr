---
title: Fichier include
description: Fichier Include
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: include
ms.custom: include file
ms.date: 11/09/2020
ms.openlocfilehash: fa497b69b067d5556f11effdb52505895ecc3bdd
ms.sourcegitcommit: 051908e18ce42b3b5d09822f8cfcac094e1f93c2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94386482"
---
Ce guide de démarrage rapide basé sur Postman vous aide à obtenir une réponse de votre base de connaissances.

## <a name="prerequisites"></a>Prérequis

* Dernière version de [**Postman**](https://www.getpostman.com/).
* Vous devez avoir :
    * Un [service QnA Maker](../How-To/set-up-qnamaker-service-azure.md)
    * Une [base de connaissances entraînée et publiée avec des questions et des réponses](../Quickstarts/add-question-metadata-portal.md), issue du précédent démarrage rapide, configurée avec des métadonnées et un échange de conversation

> [!NOTE]
> Quand vous êtes prêt à générer une réponse à une question issue de votre base de connaissances, vous devez [entraîner](../Quickstarts/create-publish-knowledge-base.md#save-and-train) et [publier](../Quickstarts/create-publish-knowledge-base.md#publish-the-knowledge-base) votre base de connaissances. Une fois votre base de connaissances publiée, la page **Publier** affiche les paramètres de requête HTTP pour générer une réponse. L’onglet **Postman** présente les paramètres nécessaires pour générer une réponse.

## <a name="set-up-postman-for-requests"></a>Configurer Postman pour les requêtes

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Ce guide de démarrage rapide utilise les mêmes paramètres pour la requête **POST** Postman, puis configure le JSON du corps POST envoyé au service en fonction de ce que vous essayez d’interroger.

Utilisez cette procédure pour configurer Postman, puis lisez chaque section suivante pour configurer le JSON du corps POST.

1. À partir de la page **Paramètres** de la base de connaissances, sélectionnez l’onglet **Postman** pour voir la configuration utilisée pour générer une réponse de la base de connaissances. Copiez les informations suivantes à utiliser dans Postman.

    |Nom|Paramètre|Objectif et valeur|
    |--|--|--|
    |`POST`| `/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer`|Il s’agit de la méthode HTTP et de la route de l’URL.|
    |`Host`|`https://YOUR-RESOURCE_NAME.azurewebsites.net/qnamaker`|Il s’agit de l’hôte de l’URL. Concaténez les valeurs Host et Post pour obtenir l’URL generateAnswer complète.|
    |`Authorization`|`EndpointKey xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`|Valeur d’en-tête nécessaire pour autoriser votre requête auprès d’Azure. |
    |`Content-type`|`application/json`|Valeur d’en-tête de votre contenu.|
    ||`{"question":"<Your question>"}`|Corps de la requête POST sous forme d’objet JSON. Cette valeur est modifiée dans chaque section suivante en fonction de ce que la requête est censée faire.|

1. Ouvrez Postman et créez un requête **POST** de base avec les paramètres de votre base de connaissances publiée. Dans les sections suivantes, modifiez le JSON du corps POST pour changer la requête adressée à votre base de connaissances.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Ce guide de démarrage rapide utilise les mêmes paramètres pour la requête **POST** Postman, puis configure le JSON du corps POST envoyé au service en fonction de ce que vous essayez d’interroger.

Utilisez cette procédure pour configurer Postman, puis lisez chaque section suivante pour configurer le JSON du corps POST.

1. À partir de la page **Paramètres** de la base de connaissances, sélectionnez l’onglet **Postman** pour voir la configuration utilisée pour générer une réponse de la base de connaissances. Copiez les informations suivantes à utiliser dans Postman.

    |Nom|Paramètre|Objectif et valeur|
    |--|--|--|
    |`POST`| `/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer`|Il s’agit de la méthode HTTP et de la route de l’URL.|
    |`Host`|`https://YOUR-RESOURCE_NAME.cognitiveservices.azure.com/qnamaker`|Il s’agit de l’hôte de l’URL. Concaténez les valeurs Host et Post pour obtenir l’URL generateAnswer complète.|
    |`Ocp-Apim-Subscription-Key`|`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`|Valeur d’en-tête nécessaire pour autoriser votre requête. |
    |`Content-type`|`application/json`|Valeur d’en-tête de votre contenu.|
    ||`{"question":"<Your question>"}`|Corps de la requête POST sous forme d’objet JSON. Cette valeur est modifiée dans chaque section suivante en fonction de ce que la requête est censée faire.|

1. Ouvrez Postman et créez un requête **POST** de base avec les paramètres de votre base de connaissances publiée. Dans les sections suivantes, modifiez le JSON du corps POST pour changer la requête adressée à votre base de connaissances.
---

## <a name="use-metadata-to-filter-answer"></a>Utiliser des métadonnées pour filtrer la réponse

Dans un démarrage rapide précédent, des métadonnées ont été ajoutées à deux paires de Q/R pour faire la distinction entre deux questions différentes. Ajoutez les métadonnées à la requête pour limiter le filtre à la paire de Q/R appropriée.

1. Dans Postman, changez uniquement le JSON de la requête en ajoutant la propriété `strictFilters` avec la paire nom/valeur de `service:qna_maker`. Le JSON du corps doit être :

    ```json
    {
        'question':'size',
        'strictFilters': [
            {
                'name':'service','value':'qna_maker'
            }
        ]
    }
    ```

    La question est un seul mot, `size`, qui peut retourner l’une ou l’autre des deux paires question/réponse. Le paramètre `strictFilters` indique à la réponse de se réduire uniquement aux réponses de `qna_maker`.

1. La réponse inclut uniquement la réponse qui répond aux critères de filtre.

    La réponse suivante a été mise en forme pour en améliorer la lisibilité :

    ```JSON
    {
        "answers": [
            {
                "questions": [
                    "How large a knowledge base can I create?",
                    "What is the max size of a knowledge base?",
                    "How many GB of data can a knowledge base hold?"
                ],
                "answer": "The size of the knowledge base depends on the SKU of Azure search you choose when creating the QnA Maker service. Read [here](https://docs.microsoft.com/azure/cognitive-services/qnamaker/tutorials/choosing-capacity-qnamaker-deployment) for more details.",
                "score": 68.76,
                "id": 3,
                "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/troubleshooting",
                "metadata": [
                    {
                        "name": "link_in_answer",
                        "value": "true"
                    },
                    {
                        "name": "service",
                        "value": "qna_maker"
                    }
                ],
                "context": {
                    "isContextOnly": false,
                    "prompts": []
                }
            }
        ],
        "debugInfo": null
    }
    ```

    S’il y a une paire question/réponse qui ne correspond pas au terme recherché, mais qui est conforme au filtre, elle n’est pas retournée. À la place, la réponse générale `No good match found in KB.` est retournée.

## <a name="use-debug-query-property"></a>Utiliser la propriété de requête de débogage

> [!NOTE]
>Nous vous déconseillons d’utiliser la propriété Debug pour les dépendances. Cette propriété a été ajoutée pour aider l’équipe produit à résoudre les problèmes.

Les informations de débogage vous aident à comprendre comment la réponse retournée a été déterminée. Bien qu’elles s’avèrent utiles, elles ne sont pas indispensables. Pour générer une réponse avec les informations de débogage, ajoutez la propriété `debug` :

1. Dans Postman, changez uniquement le JSON du corps en ajoutant la propriété `debug`. Le JSON doit être :

    ```json
    {
        'question':'size',
        'Debug': {
            'Enable':true
        }

    }
    ```

1. La réponse comprend les informations pertinentes. Dans la sortie JSON suivante, certains détails de débogage ont été remplacés par des points de suspension.

    ```console
    {
        "answers": [
            {
                "questions": [
                    "How do I share a knowledge base with others?"
                ],
                "answer": "Sharing works at the level of a QnA Maker service, that is, all knowledge bases in the service will be shared. Read [here](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/collaborate-knowledge-base) how to collaborate on a knowledge base.",
                "score": 56.07,
                "id": 5,
                "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/troubleshooting",
                "metadata": [],
                "context": {
                    "isContextOnly": false,
                    "prompts": []
                }
            }
        ],
        "debugInfo": {
            "userQuery": {
                "question": "How do I programmatically update my Knowledge Base?",
                "top": 1,
                "userId": null,
                "strictFilters": [],
                "isTest": false,
                "debug": {
                    "enable": true,
                    "recordL1SearchLatency": false,
                    "mockQnaL1Content": null
                },
                "rankerType": 0,
                "context": null,
                "qnaId": 0,
                "scoreThreshold": 0.0
            },
            "rankerInfo": {
                "specialFuzzyQuery": "how do i programmatically~6 update my knowledge base",
                "synonyms": "what s...",
                "rankerLanguage": "English",
                "rankerFileName": "https://qnamakerstore.blob.core.windows.net/qnamakerdata/rankers/ranker-English.ini",
                "rankersDirectory": "D:\\home\\site\\wwwroot\\Data\\QnAMaker\\rd0003ffa60fc45.24.0\\RankerData\\Rankers",
                "allQnAsfeatureValues": {
                    "WordnetSimilarity": {
                        "5": 0.54706300120043716,...
                    },
                    ...
                },
                "rankerVersion": "V2",
                "rankerModelType": "TreeEnsemble",
                "rankerType": 0,
                "indexResultsCount": 25,
                "reRankerResultsCount": 1
            },
            "runtimeVersion": "5.24.0",
            "indexDebugInfo": {
                "indexDefinition": {
                    "name": "064a4112-bd65-42e8-b01d-141c4c9cd09e",
                    "fields": [...
                    ],
                    "scoringProfiles": [],
                    "defaultScoringProfile": null,
                    "corsOptions": null,
                    "suggesters": [],
                    "analyzers": [],
                    "tokenizers": [],
                    "tokenFilters": [],
                    "charFilters": [],
                    "@odata.etag": "\"0x8D7A920EA5EE6FE\""
                },
                "qnaCount": 117,
                "parameters": {},
                "azureSearchResult": {
                    "continuationToken": null,
                    "@odata.count": null,
                    "@search.coverage": null,
                    "@search.facets": null,
                    "@search.nextPageParameters": null,
                    "value": [...],
                    "@odata.nextLink": null
                }
            },
            "l1SearchLatencyInMs": 0,
            "qnaL1Results": {...}
        },
        "activeLearningEnabled": true
    }
    ```

## <a name="use-test-knowledge-base"></a>Utiliser la base de connaissances de test

Si vous voulez obtenir une réponse de la base de connaissances de test, utilisez la propriété du corps `isTest`.

Dans Postman, changez uniquement le JSON du corps en ajoutant la propriété `isTest`. Le JSON doit être :

```json
{
    'question':'size',
    'isTest': true
}
```

La réponse JSON utilise le même schéma que la requête de base de connaissances publiée.

> [!NOTE]
> Si les bases de connaissances de test et publiée sont exactement les mêmes, vous pouvez quand même observer de légères variations, car l’index de test est partagé entre toutes les bases de connaissances de la ressource.

## <a name="query-for-a-chit-chat-answer"></a>Demander une réponse issue d’un échange de conversation

1. Dans Postman, remplacez uniquement le JSON du corps par une instruction de fin de conversation de l’utilisateur. Le JSON doit être :

    ```json
    {
        'question':'thank you'
    }
    ```

1. La réponse inclut le score et la réponse.

    ```json
    {
      "answers": [
          {
              "questions": [
                  "I thank you",
                  "Oh, thank you",
                  "My sincere thanks",
                  "My humblest thanks to you",
                  "Marvelous, thanks",
                  "Marvelous, thank you kindly",
                  "Marvelous, thank you",
                  "Many thanks to you",
                  "Many thanks",
                  "Kthx",
                  "I'm grateful, thanks",
                  "Ahh, thanks",
                  "I'm grateful for that, thank you",
                  "Perfecto, thanks",
                  "I appreciate you",
                  "I appreciate that",
                  "I appreciate it",
                  "I am very thankful for that",
                  "How kind, thank you",
                  "Great, thanks",
                  "Great, thank you",
                  "Gracias",
                  "Gotcha, thanks",
                  "Gotcha, thank you",
                  "Awesome thanks!",
                  "I'm grateful for that, thank you kindly",
                  "thank you pal",
                  "Wonderful, thank you!",
                  "Wonderful, thank you very much",
                  "Why thank you",
                  "Thx",
                  "Thnx",
                  "That's very kind",
                  "That's great, thanks",
                  "That is lovely, thanks",
                  "That is awesome, thanks!",
                  "Thanks bot",
                  "Thanks a lot",
                  "Okay, thanks!",
                  "Thank you so much",
                  "Perfect, thanks",
                  "Thank you my friend",
                  "Thank you kindly",
                  "Thank you for that",
                  "Thank you bot",
                  "Thank you",
                  "Right on, thanks very much",
                  "Right on, thanks a lot",
                  "Radical, thanks",
                  "Rad, thanks",
                  "Rad thank you",
                  "Wonderful, thanks!",
                  "Thanks"
              ],
              "answer": "You're welcome.",
              "score": 100.0,
              "id": 75,
              "source": "qna_chitchat_Professional.tsv",
              "metadata": [
                  {
                      "name": "editorial",
                      "value": "chitchat"
                  }
              ],
              "context": {
                  "isContextOnly": false,
                  "prompts": []
              }
          }
      ],
      "debugInfo": null,
      "activeLearningEnabled": true
    }
    ```

    Étant donné que la question de `Thank you` correspond exactement à une question d’échanges de conversation, QnA Maker est totalement confiant avec un score de 100. QnA Maker a également retourné toutes les questions associées, ainsi que la propriété de métadonnées contenant les informations de la balise de métadonnées d’échange de conversation.

## <a name="use-threshold-and-default-answer"></a>Utiliser un seuil et une réponse par défaut

Vous pouvez demander un seuil minimal pour la réponse. Si le seuil n’est pas atteint, la réponse par défaut est retournée.

1. Dans Postman, remplacez uniquement le JSON du corps par une instruction de fin de conversation de l’utilisateur. Le JSON doit être :

    ```json
    {
        'question':'size',
        'scoreThreshold':80.00
    }
    ```

    La base de connaissances ne doit pas trouver cette réponse, car le score de la question s’élève à 71 %. Elle doit donc retourner la réponse par défaut que vous avez fournie lors de la création de la base de connaissances.

    La réponse JSON retournée, qui inclut le score et la réponse, est la suivante :

    ```json
    {
        "answers": [
            {
                "questions": [],
                "answer": "No good match found in KB.",
                "score": 0.0,
                "id": -1,
                "source": null,
                "metadata": []
            }
        ],
        "debugInfo": null,
        "activeLearningEnabled": true
    }
    ```

    QnA Maker a retourné un score de `0`, ce qui signifie « pas de certitude ». Il a également retourné la réponse par défaut.

1. Remplacez la valeur du seuil par 60 % et réexécutez la requête :

    ```json
    {
        'question':'size',
        'scoreThreshold':60.00
    }
    ```

    Le JSON retourné a trouvé la réponse.

    ```json
    {
        "answers": [
            {
                "questions": [
                    "How large a knowledge base can I create?",
                    "What is the max size of a knowledge base?",
                    "How many GB of data can a knowledge base hold?"
                ],
                "answer": "The size of the knowledge base depends on the SKU of Azure search you choose when creating the QnA Maker service. Read [here](https://docs.microsoft.com/azure/cognitive-services/qnamaker/tutorials/choosing-capacity-qnamaker-deployment) for more details.",
                "score": 71.1,
                "id": 3,
                "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/troubleshooting",
                "metadata": [
                    {
                        "name": "link_in_answer",
                        "value": "true"
                    },
                    {
                        "name": "server",
                        "value": "qna_maker"
                    }
                ],
                "context": {
                    "isContextOnly": false,
                    "prompts": []
                }
            }
        ],
        "debugInfo": null,
        "activeLearningEnabled": true
    }
    ```