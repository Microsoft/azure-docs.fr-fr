---
title: 'Tutoriel : Prédire les intentions - LUIS'
description: Dans ce tutoriel, vous allez créer une application personnalisée qui prédit l’intention d’un utilisateur sur la base de l’énoncé (texte).
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: tutorial
ms.date: 05/05/2020
ms.openlocfilehash: 398324099f923a5141b172d61f9fd41a69dbf069
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025291"
---
# <a name="tutorial-build-a-luis-app-to-determine-user-intentions"></a>Tutoriel : Générer une application LUIS pour déterminer les intentions d’un utilisateur

Dans ce didacticiel, vous allez créer une application personnalisée qui prédit l’intention d’un utilisateur sur la base de l’énoncé (texte).

**Dans ce tutoriel, vous allez découvrir comment :**

> [!div class="checklist"]
> * Créer une application
> * Créer des intentions
> * Ajouter des exemples d’énoncés
> * Entraîner une application
> * Publier une application
> * Obtenir une prédiction d’intention à partir du point de terminaison

## <a name="user-intentions-as-intents"></a>Intentions utilisateur en tant qu’intentions

L’objet de l’application est de déterminer l’intention véhiculée par un texte de conversation en langage naturel :

`I'd like to order a veggie pizza with a salad on the side.`

Ces intentions sont classées dans la catégorie **intentions**.

|Intentionnel|Objectif|
|--|--|
|`ModifyOrder`|Déterminer la commande de pizza de l’utilisateur.|
|`Greeting`|Commencez la conversation avec le bot.|
|`ConfirmOrder`|Confirmez la commande de pizza.|
|`None`|Déterminez si l’utilisateur demande une chose à laquelle l’application LUIS n’est pas conçue pour répondre. Cette intention est fournie dans le cadre de la création de l’application et elle ne peut pas être supprimée. |

## <a name="create-a-new-app"></a>Créer une application

[!INCLUDE [Follow these steps to create a new LUIS app](includes/create-pizza-app.md)]

## <a name="create-a-new-intent"></a>Créer une intention

Une intention sert à classifier les énoncés utilisateur en fonction de l’intention de l’utilisateur, déterminée à partir du texte en langage naturel.

Pour classifier un énoncé, l’intention a besoin d’exemples d’énoncés utilisateur qui doivent être classifiés avec cette intention.

1. Dans la section **Générer**, dans la page **Intentions**, sélectionnez **+Créer** pour créer une intention. Entrez le nom de l’intention (`OrderPizza`), puis sélectionnez **Terminé**.

    L’intention `OrderPizza` est prédite quand un utilisateur veut commander une pizza.

1. Ajoutez plusieurs exemples d’énoncés à cette intention qu’un utilisateur est selon vous susceptible de demander :

    |Exemples d’énoncés `OrderPizza`|
    |--|
    |`can i get a pepperoni pizza and a can of coke please`|
    |`can i get a small pizza with onions peppers and olives`|
    |`delivery for a small pepperoni pizza`|
    |`pickup a cheddar cheese pizza large with extra anchovies`|
    |`i need 2 large cheese pizzas 6 large pepperoni pizzas and 1 large supreme pizza`|
    |`Order a pizza for me`|

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de l’ajout d’exemples d’énoncés dans le portail LUIS sur la page des intentions](media/tutorial-intents-only/add-example-utterances-for-pizza-order.png)

    En fournissant des _exemples d’énoncés_, vous formez LUIS à déterminer les types d’énoncés à prédire pour cette intention. Il s’agit d’exemples positifs. Les énoncés de toutes les autres intentions sont traités comme des exemples négatifs pour cette intention.

    [!INCLUDE [Do not use too few utterances](includes/do-not-use-too-few-utterances.md)]

## <a name="create-remaining-intents"></a>Créer les intentions restantes

1. Créez l’intention `Greeting` et ajoutez les exemples d’énoncés suivants. Ceci est l’intention pour déterminer si un utilisateur entame une nouvelle conversation pour commander une pizza.

    |Exemples d’énoncés `Greeting`|
    |--|
    |`Hi`|
    |`Hello`|
    |`Hey`|
    |`Start`|
    |`Begin`|

1. Créez l’intention `Confirm` et ajoutez les exemples d’énoncés suivants. Ceci est l’intention pour déterminer si un utilisateur a fini de commander et accepte les détails de la commande.

    |Exemples d’énoncés `Confirm`|
    |--|
    |`Go ahead`|
    |`ok`|
    |`Yes`|
    |`Sure`|


## <a name="none-intent-example-utterances"></a>Exemples d’énoncés de l’intention None

[!INCLUDE [Follow these steps to add the None intent to the app](includes/add-example-utterances-none-intent.md)]

## <a name="train-the-app"></a>Effectuer l’apprentissage de l’application

[!INCLUDE [LUIS How to Train steps](includes/howto-train.md)]

## <a name="publish-the-app"></a>Publier l’application

[!INCLUDE [LUIS How to Publish steps](includes/howto-publish.md)]

## <a name="get-intent-prediction"></a>Obtenir une prédiction d’intention

1. [!INCLUDE [LUIS How to get endpoint first step](includes/howto-get-endpoint.md)]

1. Allez à la fin de l’URL dans la barre d’adresses, puis entrez :

    `get a medium vegetarian pizza for delivery`

  
    Le paramètre de la chaîne de requête est `query`. L’énoncé **requête** est passé à l’URI. Cet énoncé diffère de tous les exemples d’énoncés. Il doit constituer un bon test pour vérifier si LUIS apprend et prédit l’intention `OrderPizza` en tant qu’intention avec le score le plus élevé.

    ```JSON
    {
        "query": "get a medium vegetarian pizza for delivery",
        "prediction": {
            "topIntent": "OrderPizza",
            "intents": {
                "OrderPizza": {
                    "score": 0.6488959
                },
                "None": {
                    "score": 0.139966831
                },
                "Confirm": {
                    "score": 0.00736504374
                },
                "Greeting": {
                    "score": 0.003970454
                }
            },
            "entities": {}
        }
    }
    ```

    Le tableau d’entités est vide, car cette application n’a pas encore d’entités (unités de données d’un énoncé à extraire).

    Le résultat JSON identifie l’intention avec le score le plus élevé en tant que propriété **`prediction.topIntent`** . Tous les scores sont compris entre 1 et 0, le meilleur score étant celui qui est le plus proche de 1.

1. Modifiez paramètre **requête** de l’URL pour cibler l’intention **Salutations** :

    `Howdy`

    Ceci n’est pas exactement la même chose qu’un exemple d’énoncé. C’est donc un bon test pour voir si LUIS peut savoir ce qui faut prédire avec cette intention.

    ```json
    {
        "query": "howdy",
        "prediction": {
            "topIntent": "Greeting",
            "intents": {
                "Greeting": {
                    "score": 0.446016937
                },
                "Confirm": {
                    "score": 0.2390079
                },
                "None": {
                    "score": 0.09119555
                },
                "OrderPizza": {
                    "score": 0.00109590159
                }
            },
            "entities": {}
        }
    }
    ```

    Cette prédiction a un score de confiance de 44 %. Pour augmenter le score de confiance, ajoutez entre 15 et 30 exemples d’énoncés.

## <a name="client-application-next-steps"></a>Étapes suivantes de l’application cliente

Dans ce didacticiel, vous avez créé une application LUIS et des intentions, ajouté des exemples d’énoncés pour chaque intention, ajouté des exemples d’énoncés à l’intention None, puis effectué un apprentissage, publié et testé au niveau du point de terminaison. Ce sont les étapes de base de la génération d’un modèle LUIS.

Une fois que LUIS a retourné la réponse JSON, il en a fini avec cette demande. LUIS ne fournit pas de réponses aux énoncés de l’utilisateur ; il identifie uniquement le type d’informations demandé dans un langage naturel. Le suivi de la conversation est fourni par l’application cliente telle qu’un bot Azure.


[!INCLUDE [LUIS How to clean up resources](includes/quickstart-tutorial-cleanup-resources.md)]

## <a name="related-information"></a>Informations connexes

* [Types d’entités](luis-concept-entity-types.md)
* [Guide pratique pour entraîner](luis-how-to-train.md)
* [Comment publier](luis-how-to-publish-app.md)
* [Guide pratique pour tester dans le portail LUIS](luis-interactive-test.md)
* [Azure Bot](/azure/bot-service/?view=azure-bot-service-4.0)


## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Ajouter une entité décomposable à cette application](tutorial-machine-learned-entity.md)