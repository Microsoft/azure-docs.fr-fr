---
title: Fichier include
description: Fichier include
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: include
ms.custom: include file
ms.date: 11/09/2020
ms.openlocfilehash: cbabfc1ae0c54c4ef20d3c689506e486bf4b6a66
ms.sourcegitcommit: 051908e18ce42b3b5d09822f8cfcac094e1f93c2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94386485"
---
Ce démarrage rapide basé sur cURL vous montre pas à pas comment obtenir une réponse de votre base de connaissances.

## <a name="prerequisites"></a>Prérequis

* Dernière version de [**cURL**](https://curl.haxx.se/).
* Vous devez avoir :
    * Un [service QnA Maker](../How-To/set-up-qnamaker-service-azure.md)
    * Une base de connaissances entraînée et publiée avec des questions et des réponses, issue du précédent [démarrage rapide](../Quickstarts/add-question-metadata-portal.md), configurée avec des métadonnées et un échange de conversation

> [!NOTE]
> Quand vous êtes prêt à générer une réponse à une question issue de votre base de connaissances, vous devez [entraîner](../Quickstarts/create-publish-knowledge-base.md#save-and-train) et [publier](../Quickstarts/create-publish-knowledge-base.md#publish-the-knowledge-base) votre base de connaissances. Une fois votre base de connaissances publiée, la page **Publier** affiche les paramètres de requête HTTP pour générer une réponse. L’onglet **cURL** présente les paramètres nécessaires à la génération d’une réponse à partir de l’outil de ligne de commande.

## <a name="use-metadata-to-filter-answer"></a>Utiliser des métadonnées pour filtrer la réponse

Utilisez la base de connaissances issue du démarrage rapide précédent pour demander une réponse basée sur les métadonnées.

1. À partir de la page **Paramètres** de la base de connaissances, sélectionnez l’onglet **CURL** pour voir un exemple de commande cURL permettant de générer une réponse issue de la base de connaissances.
1. Copiez la commande dans un environnement modifiable (comme un fichier texte) afin de pouvoir modifier la commande. Modifiez la valeur de la question comme suit pour faire en sorte que les métadonnées de `service:qna_maker` soient utilisées comme filtre pour les paires de Q/R.

   # <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

    ```bash
    curl -X POST https://replace-with-your-resource-name.azurewebsites.net/qnamaker/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer -H "Authorization: EndpointKey replace-with-your-endpoint-key" -H "Content-type: application/json" -d "{'top':30, 'question':'size','strictFilters': [{'name':'service','value':'qna_maker'}]}"
    ```
    # <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)
    
    ```bash
    curl -X POST https://replace-with-your-resource-name.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer -H   "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" -H "Content-type: application/json" -d "{'top':30, 'question':'size','strictFilters': [{'name':'service','value':'qna_maker'}]}"
    ```
    ---

    La question n’est qu’un seul mot, `size`, qui peut retourner une des deux paires de Q/R. Le paramètre `strictFilters` indique à la réponse de se réduire uniquement aux réponses de `qna_maker`.

1. La réponse inclut uniquement la réponse qui répond aux critères de filtre. La réponse cURL suivante a été mise en forme pour en améliorer la lisibilité :

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

```json
Debug: {Enable:true}
```

1. Modifiez la commande cURL pour inclure la propriété de débogage afin d’obtenir plus d’informations.

   # <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)
    ```bash
    curl -X POST https://replace-with-your-resource-name.azurewebsites.net/qnamaker/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer -H "Authorization: EndpointKey replace-with-your-endpoint-key" -H "Content-type: application/json" -d "{'question':'size', 'Debug':{'Enable':true}}"
    ```
    # <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)
    ```bash
    curl -X POST https://replace-with-your-resource-name.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer -H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" -H "Content-type: application/json" -d "{'question':'size', 'Debug':{'Enable':true}}"
    ```
    ---

1. La réponse comprend les informations pertinentes. Dans la sortie JSON suivante, certains détails de débogage ont été remplacés par des points de suspension par souci de concision.

    ```console
    {
        "answers": [
            {
                "questions": [
                    "How do I share a knowledge base with others?"
                ],
                "answer": "Sharing works at the level of a QnA Maker service, that is, all knowledge bases in the service will be shared. Read [here](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/collaborate-knowledge-base) to learn how to collaborate on a knowledge base.",
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

Cette propriété est une valeur booléenne.

```json
isTest:true
```

La commande cURL ressemble à ceci :
# <a name="qna-maker-ga"></a>[QnA Maker GA](#tab/v1)
```bash
curl -X POST https://replace-with-your-resource-name.azurewebsites.net/qnamaker/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer -H "Authorization: EndpointKey replace-with-your-endpoint-key" -H "Content-type: application/json" -d "{'question':'size', 'IsTest':true}"
```
# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)
```bash
curl -X POST https://replace-with-your-resource-name.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer -H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" -H "Content-type: application/json" -d "{'question':'size', 'IsTest':true}"
```

---
La réponse JSON utilise le même schéma que la requête de base de connaissances publiée.

> [!NOTE]
> Si les bases de connaissances de test et publiée sont exactement les mêmes, vous pouvez quand même observer de légères variations, car l’index de test est partagé entre toutes les bases de connaissances de la ressource.

## <a name="use-curl-to-query-for-a-chit-chat-answer"></a>Utiliser cURL pour demander une réponse d’échange de conversation (chit-chat)

1. Dans le terminal prenant en charge cURL, utilisez une instruction de fin de conversation de bot provenant de l’utilisateur, comme `Thank you` en tant que question. Il n’y a aucune autre propriété à définir.
    
   # <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

    ```bash
    curl -X POST https://replace-with-your-resource-name.azurewebsites.net/qnamaker/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer -H "Authorization: EndpointKey replace-with-your-endpoint-key" -H "Content-type: application/json" -d "{'question':'thank you'}"
    ```
    # <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)
    ```bash
    curl -X POST https://replace-with-your-resource-name.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer -H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" -H "Content-type: application/json" -d "{'question':'thank you'}"
    ```
    ---

1. Exécutez la commande cURL et recevez la réponse JSON comprenant le score et la réponse.

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

1. Ajoutez la propriété `threshold` pour demander une réponse à `size` avec un seuil de 80 % ou plus. La base de connaissances ne doit pas trouver cette réponse, car le score de la question s’élève à 71 %. Le résultat retourne la réponse par défaut que vous avez fournie lors de la création de la base de connaissances.

   # <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)
    ```bash
    curl -X POST https://replace-with-your-resource-name.azurewebsites.net/qnamaker/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer -H "Authorization: EndpointKey replace-with-your-endpoint-key" -H "Content-type: application/json" -d "{'question':'size', 'scoreThreshold':80.00}"
    ```
    # <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)
    ```bash
    curl -X POST https://replace-with-your-resource-name.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer -H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" -H "Content-type: application/json" -d "{'question':'size', 'scoreThreshold':80.00}"
    ```
    ---

1. Exécutez la commande cURL pour recevoir la réponse JSON.

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
    
   # <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)
    ```bash
    curl -X POST https://replace-with-your-resource-name.azurewebsites.net/qnamaker/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer -H "Authorization: EndpointKey replace-with-your-endpoint-key" -H "Content-type: application/json" -d "{'question':'size', 'scoreThreshold':60.00}"
    ```
    # <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)
    ```bash
    curl -X POST https://replace-with-your-resource-name.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/replace-with-your-knowledge-base-id/generateAnswer -H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" -H "Content-type: application/json" -d "{'question':'size', 'scoreThreshold':60.00}"
    ```
    ---

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