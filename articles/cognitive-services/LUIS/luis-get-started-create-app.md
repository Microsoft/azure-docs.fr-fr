---
title: 'Démarrage rapide : Créer une application - LUIS'
description: Ce guide de démarrage rapide montre comment créer une application LUIS qui utilise le domaine prédéfini `HomeAutomation` pour allumer et éteindre des lumières et des appareils électriques. Ce domaine prédéfini vous fournit les intentions, les entités et des exemples d’énoncés. À la fin du processus, vous disposerez d’un point de terminaison LUIS exécuté dans le cloud.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: quickstart
ms.date: 10/13/2020
ms.openlocfilehash: 2048079a9b1eb5234d03c6fe57073228bd892d53
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95019158"
---
# <a name="quickstart-use-prebuilt-home-automation-app"></a>Démarrage rapide : Utiliser une application domotique prédéfinie

Dans ce guide de démarrage rapide, vous allez créer une application LUIS qui utilise le domaine prédéfini `HomeAutomation` pour allumer et éteindre des lumières et des appareils électriques. Ce domaine prédéfini vous fournit les intentions, les entités et des exemples d’énoncés. À la fin du processus, vous disposerez d’un point de terminaison LUIS exécuté dans le cloud.

[!INCLUDE [Sign in to LUIS](./includes/sign-in-process.md)]

[!INCLUDE [Select authoring resource](./includes/select-authoring-resource.md)]

## <a name="create-a-new-app"></a>Créer une application
Vous pouvez créer et gérer vos applications sur la page **Mes applications**.

### <a name="create-an-application"></a>Créer une application

Pour créer une application, cliquez sur **+ Nouvelle application**. 

Dans la fenêtre qui s’affiche, entrez les informations suivantes :

|Name  |Description  |
|---------|---------|
|Nom     | Nom de votre application. Par exemple, « domotique ».        |
|culture     | Langage que votre application comprend et parle.   |
|Description | Description de votre application.
|Ressource de prédiction | Ressource de prédiction qui recevra les requêtes. |

Sélectionnez **Terminé**.

>[!NOTE]
>La culture ne peut pas être modifiée une fois que l’application est créée.

## <a name="add-prebuilt-domain"></a>Ajouter un domaine prédéfini

1. Dans le volet de navigation de gauche, sélectionnez **Domaines prédéfinis**.
1. Recherchez **HomeAutomation**.
1. Sélectionnez **Ajouter un domaine** sur la carte HomeAutomation.

    > [!div class="mx-imgBorder"]
    > ![Sélectionnez « Domaines prédéfinis », puis recherchez « HomeAutomation ». Sélectionnez « Ajouter un domaine » sur la carte « HomeAutomation ».](media/luis-quickstart-new-app/home-automation.png)

    Une fois le domaine ajouté, la zone de domaine prédéfini affiche un bouton **Supprimer le domaine**.

## <a name="intents-and-entities"></a>Intentions et entités

1. Sélectionnez **Intentions** dans le volet de navigation de gauche pour consulter les intentions du domaine HomeAutomation. Il contient des exemples d’énoncés, tels que `HomeAutomation.QueryState` et     `HomeAutomation.SetDevice`.

    > [!NOTE]
    > **Aucun** est une intention fournie par toutes les applications LUIS. Elle vous permet de gérer les énoncés qui ne correspondent pas aux fonctionnalités fournies par votre application.

1. Sélectionnez l’intention **HomeAutomation.TurnOff**. L’intention contient une liste d’exemples d’énoncés qui sont associés à des entités.

    > [!div class="mx-imgBorder"]
    > [![Capture d’écran de l’intention HomeAutomation.TurnOff](media/luis-quickstart-new-app/home-automation-turnoff.png "Capture d’écran de l’intention HomeAutomation.TurnOff")](media/luis-quickstart-new-app/home-automation-turnoff.png)

1. Si vous souhaitez afficher les entités pour l’application, sélectionnez **Entités**. Si vous cliquez sur l’une des entités, par exemple **HomeAutomation.DeviceName**, vous verrez la liste des valeurs qui lui sont associées. 
 
    :::image type="content" source="media/luis-quickstart-new-app/entities-page.png" alt-text="Texte de l’image associée" lightbox="media/luis-quickstart-new-app/entities-page.png":::

## <a name="train-the-luis-app"></a>Entraîner l’application LUIS

[!INCLUDE [LUIS How to Train steps](includes/howto-train.md)]

## <a name="test-your-app"></a>Test de l'application
Une fois que vous avez formé votre application, vous pouvez la tester.

1. Sélectionnez **Test** dans le menu en haut à droite.

1. Saisissez un énoncé de test tel que `Turn off the lights` dans le volet de test interactif, puis appuyez sur Entrée. Par exemple, *Turn off the lights* (Éteindre les lumières).

    Dans cet exemple, `Turn off the lights` est identifié correctement comme l’intention avec le score le plus élevé pour **HomeAutomation.TurnOff**.

    ![Capture d’écran du panneau Test avec l’énoncé mis en surbrillance](media/luis-quickstart-new-app/review-test-inspection-pane-in-portal.png)

1. Sélectionnez **Inspecter** pour voir plus d’informations sur la prédiction.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran du panneau Test avec les informations d’inspection](media/luis-quickstart-new-app/test.png)

1. Fermez le volet de test.

<a name="publish-your-app"></a>

## <a name="publish-the-app-to-get-the-endpoint-url"></a>Publier l’application pour obtenir l’URL de point de terminaison

Pour recevoir une prédiction LUIS dans un chatbot ou une autre application cliente, vous devez publier l’application sur le point de terminaison de prédiction.

1. En haut à droite de la fenêtre, sélectionnez **Publier**.

1. Sélectionnez l’emplacement **Production**, puis sélectionnez **Terminé**.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran LUIS - Publier sur le point de terminaison](media/howto-publish/publish-app-popup.png)

1. Sélectionnez le lien **Accéder à vos URL de point de terminaison** dans la notification pour accéder à la page **Ressources Azure**. Les URL de point de terminaison sont répertoriées en tant qu’**Exemple de requête**.

<!-- [!INCLUDE [LUIS How to Publish steps](./includes/howto-publish.md)] -->

<a name="query-the-v2-api-prediction-endpoint"></a>

## <a name="query-the-v3-api-prediction-endpoint"></a>Interroger le point de terminaison de prédiction d’API V3

[!INCLUDE [LUIS How to get endpoint first step](./includes/v3-prediction-endpoint.md)]

2. Dans la barre d’adresse du navigateur, pour la chaîne de requête, assurez-vous que les barres de noms et de valeurs suivantes figurent dans l’URL. Si elles ne figurent pas dans la chaîne de requête, ajoutez-les :

    |Paire nom/valeur|
    |--|
    |`verbose=true`|
    |`show-all-intents=true`|

3. Dans la barre d’adresse du navigateur, allez à la fin de l’URL et entrez `turn off the living room light` pour la valeur _requête_, puis appuyez sur Entrée.

    ```json
    {
        "query": "turn off the living room light",
        "prediction": {
            "topIntent": "HomeAutomation.TurnOff",
            "intents": {
                "HomeAutomation.TurnOff": {
                    "score": 0.969448864
                },
                "HomeAutomation.QueryState": {
                    "score": 0.0122336326
                },
                "HomeAutomation.TurnUp": {
                    "score": 0.006547436
                },
                "HomeAutomation.TurnDown": {
                    "score": 0.0050634006
                },
                "HomeAutomation.SetDevice": {
                    "score": 0.004951761
                },
                "HomeAutomation.TurnOn": {
                    "score": 0.00312553928
                },
                "None": {
                    "score": 0.000552945654
                }
            },
            "entities": {
                "HomeAutomation.Location": [
                    "living room"
                ],
                "HomeAutomation.DeviceName": [
                    [
                        "living room light"
                    ]
                ],
                "HomeAutomation.DeviceType": [
                    [
                        "light"
                    ]
                ],
                "$instance": {
                    "HomeAutomation.Location": [
                        {
                            "type": "HomeAutomation.Location",
                            "text": "living room",
                            "startIndex": 13,
                            "length": 11,
                            "score": 0.902181149,
                            "modelTypeId": 1,
                            "modelType": "Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ],
                    "HomeAutomation.DeviceName": [
                        {
                            "type": "HomeAutomation.DeviceName",
                            "text": "living room light",
                            "startIndex": 13,
                            "length": 17,
                            "modelTypeId": 5,
                            "modelType": "List Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ],
                    "HomeAutomation.DeviceType": [
                        {
                            "type": "HomeAutomation.DeviceType",
                            "text": "light",
                            "startIndex": 25,
                            "length": 5,
                            "modelTypeId": 5,
                            "modelType": "List Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ]
                }
            }
        }
    }
    ```

    Découvrez-en plus sur le [point de terminaison de prédiction V3](luis-migration-api-v3.md).


## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez appeler le point de terminaison à partir du code :

> [!div class="nextstepaction"]
> [Appeler un point de terminaison LUIS à l’aide d’un code](./luis-get-started-get-intent-from-rest.md)