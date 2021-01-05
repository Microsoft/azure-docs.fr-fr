---
title: Guide pratique pour interroger des prédictions à l’aide d’un navigateur - LUIS
description: Dans cet article, utilisez une application LUIS publique disponible pour déterminer l’intention d’un utilisateur à partir du texte conversationnel dans un navigateur.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 11/30/2020
ms.openlocfilehash: 6aa681c8ea60cb57ac3e6d143c60cc46bc057719
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96436910"
---
# <a name="how-to-query-the-prediction-runtime-with-user-text"></a>Guide pratique pour interroger le runtime de prédiction avec du texte utilisateur

Pour comprendre ce que retourne un point de terminaison de prédiction LUIS, affichez un résultat de prédiction dans un navigateur web.

## <a name="prerequisites"></a>Prérequis

Pour pouvoir interroger une application publique, voici ce dont vous avez besoin :

* Vos informations de ressource LUIS (Language Understanding) :
    * **Clé de prédiction**, qui peut être obtenue à partir du [portail LUIS](https://www.luis.ai/). Si vous n’avez pas encore d’abonnement pour créer une clé, vous pouvez vous inscrire pour obtenir un [compte gratuit](https://azure.microsoft.com/free/cognitive-services).
    * **Sous-domaine du point de terminaison de prédiction** : le sous-domaine est également le **nom** de votre ressource LUIS.
* Un ID d’application LUIS : utilisez l’ID d’application IoT publique `df67dcdb-c37d-46af-88e1-8b97951ca1c2`. La requête utilisateur utilisée dans le code du guide de démarrage rapide est propre à cette application.

## <a name="use-the-browser-to-see-predictions"></a>Utiliser le navigateur pour voir des prédictions

1. Ouvrez un navigateur web.
1. Utilisez les URL complètes ci-dessous, en remplaçant `YOUR-KEY` par votre propre clé de prédiction LUIS. Les requêtes sont des requêtes GET qui incluent l’autorisation, avec votre clé de prédiction LUIS, en tant que paramètre de chaîne de requête.

    #### <a name="v3-prediction-request"></a>[Requête de prédiction V3](#tab/V3-1-1)


    Le format de l’URL V3 pour une demande de point de terminaison **GET** (par les emplacements) est :

    `
    https://YOUR-LUIS-ENDPOINT-SUBDOMAIN.api.cognitive.microsoft.com/luis/prediction/v3.0/apps/df67dcdb-c37d-46af-88e1-8b97951ca1c2/slots/production/predict?query=turn on all lights&subscription-key=YOUR-LUIS-PREDICTION-KEY
    `

    #### <a name="v2-prediction-request"></a>[Requête de prédiction V2](#tab/V2-1-2)

    Le format de l’URL V2 pour une demande de point de terminaison **GET** est :

    `
    https://YOUR-LUIS-ENDPOINT-SUBDOMAIN.api.cognitive.microsoft.com/luis/v2.0/apps/df67dcdb-c37d-46af-88e1-8b97951ca1c2?subscription-key=YOUR-LUIS-PREDICTION-KEY&q=turn on all lights
    `

1. Collez l’URL dans une fenêtre de navigateur, puis appuyez sur Entrée. Le navigateur affiche un résultat JSON qui indique que LUIS a détecté l’intention `HomeAutomation.TurnOn` comme première intention et l’entité `HomeAutomation.Operation` pourvue de la valeur `on`.

    #### <a name="v3-prediction-response"></a>[Réponse de prédiction V3](#tab/V3-2-1)

    ```JSON
    {
        "query": "turn on all lights",
        "prediction": {
            "topIntent": "HomeAutomation.TurnOn",
            "intents": {
                "HomeAutomation.TurnOn": {
                    "score": 0.5375382
                }
            },
            "entities": {
                "HomeAutomation.Operation": [
                    "on"
                ]
            }
        }
    }
    ```

    #### <a name="v2-prediction-response"></a>[Réponse de prédiction V2](#tab/V2-2-2)

    ```json
    {
        "query": "turn on all lights",
        "topScoringIntent": {
            "intent": "HomeAutomation.TurnOn",
            "score": 0.5375382
        },
        "entities": [
            {
                "entity": "on",
                "type": "HomeAutomation.Operation",
                "startIndex": 5,
                "endIndex": 6,
                "score": 0.724984169
            }
        ]
    }
    ```

    * * *

1. Pour voir toutes les intentions, ajoutez le paramètre de chaîne de requête approprié.

    #### <a name="v3-prediction-endpoint"></a>[Point de terminaison de prédiction V3](#tab/V3-3-1)

    Ajoutez `show-all-intents=true` à la fin de la chaîne de requête pour **afficher toutes les intentions** :

    `
    https://YOUR-LUIS-ENDPOINT-SUBDOMAIN.api.cognitive.microsoft.com/luis/predict/v3.0/apps/df67dcdb-c37d-46af-88e1-8b97951ca1c2/slots/production/predict?query=turn on all lights&subscription-key=YOUR-LUIS-PREDICTION-KEY&show-all-intents=true
    `

    ```JSON
    {
        "query": "turn off the living room light",
        "prediction": {
            "topIntent": "HomeAutomation.TurnOn",
            "intents": {
                "HomeAutomation.TurnOn": {
                    "score": 0.5375382
                },
                "None": {
                    "score": 0.08687421
                },
                "HomeAutomation.TurnOff": {
                    "score": 0.0207554
                }
            },
            "entities": {
                "HomeAutomation.Operation": [
                    "on"
                ]
            }
        }
    }
    ```

    #### <a name="v2-prediction-endpoint"></a>[Point de terminaison de prédiction V2](#tab/V2)

    Ajoutez `verbose=true` à la fin de la chaîne de requête pour **afficher toutes les intentions** :

    `
    https://YOUR-LUIS-ENDPOINT-SUBDOMAIN.api.cognitive.microsoft.com/luis/v2.0/apps/df67dcdb-c37d-46af-88e1-8b97951ca1c2?q=turn on all lights&subscription-key=YOUR-LUIS-PREDICTION-KEY&verbose=true
    `

    ```json
    {
        "query": "turn on all lights",
        "topScoringIntent": {
            "intent": "HomeAutomation.TurnOn",
            "score": 0.5375382
        },
        "intents": [
            {
                "intent": "HomeAutomation.TurnOn",
                "score": 0.5375382
            },
            {
                "intent": "None",
                "score": 0.08687421
            },
            {
                "intent": "HomeAutomation.TurnOff",
                "score": 0.0207554
            }
        ],
        "entities": [
            {
                "entity": "on",
                "type": "HomeAutomation.Operation",
                "startIndex": 5,
                "endIndex": 6,
                "score": 0.724984169
            }
        ]
    }
    ```

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus :
* [Point de terminaison de prédiction V3](luis-migration-api-v3.md)
* [Sous-domaines personnalisés](../cognitive-services-custom-subdomains.md)

> [!div class="nextstepaction"]
> [Créer une application dans le portail LUIS](get-started-portal-build-app.md)
