---
title: 'Tutoriel : Révision des énoncés de point de terminaison - LUIS'
description: Dans ce tutoriel, améliorez les prédictions d’application en vérifiant ou corrigeant les énoncés reçus via le point de terminaison HTTP de LUIS dont ce dernier n’est pas sûr. Certains énoncés peuvent devoir faire l’objet d’une vérification d’intention, d’autres d’une vérification d’entité.
services: cognitive-services
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: tutorial
ms.date: 07/02/2020
ms.openlocfilehash: b8f8fa2cd3c9c22187bb95c55d9de2abb2e8caec
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91324635"
---
# <a name="tutorial-fix-unsure-predictions-by-reviewing-endpoint-utterances"></a>Tutoriel : Corriger les prédictions incertaines en révisant les énoncés de point de terminaison
Dans ce tutoriel, améliorez les prédictions de l’application en vérifiant ou corrigeant les énoncés reçus via le point de terminaison HTTPS de LUIS dont ce dernier n’est pas sûr. Vous devez examiner les énoncés de point de terminaison régulièrement dans le cadre de la maintenance LUIS planifiée.

Ce processus de révision permet à LUIS de découvrir le domaine de votre application. LUIS sélectionne les énoncés qui apparaissent dans la liste de révision. Cette liste est :

* Spécifique à l’application.
* Est destinée à améliorer la précision de prédiction de l’application.
* Doit être révisée régulièrement.

En passant en revue les énoncés de point de terminaison, vous vérifiez ou corrigez l’intention prédite de l’énoncé.

**Dans ce tutoriel, vous allez découvrir comment :**

<!-- green checkmark -->
> [!div class="checklist"]
> * Importer l’exemple d’application
> * Réviser les énoncés de point de terminaison
> * Entraîner et publier une application.
> * Interroger un point de terminaison de l’application pour voir la réponse JSON de LUIS

[!INCLUDE [LUIS Free account](../../../includes/cognitive-services-luis-free-key-short.md)]

## <a name="download-json-file-for-app"></a>Télécharger le fichier JSON pour l’application

Téléchargez et enregistrez le [fichier JSON de l’application](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/luis/apps/tutorial-fix-unsure-predictions.json?raw=true).

## <a name="import-json-file-for-app"></a>Importer le fichier JSON pour l’application


1. Sur le [portail LUIS](https://www.luis.ai), dans la page **Mes applications**, sélectionnez **+Nouvelle application de conversation**, puis **Importer en tant que JSON**. Recherchez le fichier JSON enregistré à l’étape précédente. Vous n’avez pas besoin de modifier le nom de l’application. Sélectionnez **Terminé**

1. Sélectionnez **Générer** puis **Intentions** pour voir les intentions, qui sont les principaux composants d’une application LUIS.

    :::image type="content" source="media/luis-tutorial-review-endpoint-utterances/initial-intents-in-app.png" alt-text="Passez de la page Versions à la page Intentions.":::

## <a name="train-the-app-to-apply-the-entity-changes-to-the-app"></a>Effectuer l’apprentissage de l’application pour appliquer les modifications d’entité à l’application

[!INCLUDE [LUIS How to Train steps](includes/howto-train.md)]

## <a name="publish-the-app-to-access-it-from-the-http-endpoint"></a>Publier l’application pour y accéder à partir du point de terminaison HTTP

[!INCLUDE [LUIS How to Publish steps](includes/howto-publish.md)]

## <a name="add-utterances-at-the-endpoint"></a>Ajouter des énoncés au point de terminaison

Dans cette application, vous avez des intentions et des entités, mais vous n’avez pas d’utilisation du point de terminaison. Cette utilisation du point de terminaison est nécessaire pour améliorer l’application avec le passage en revue des énoncés du point de terminaison.

1. [!INCLUDE [LUIS How to get endpoint first step](includes/howto-get-endpoint.md)]

1. Allez à la fin de l’URL dans la barre d’adresses, puis remplacez _YOUR_QUERY_HERE_ par les énoncés du tableau suivant. Pour chaque énoncé, soumettez l’énoncé et obtenez le résultat. Remplacez ensuite l’énoncé à la fin par l’énoncé suivant.

    |Énoncé du point de terminaison|Intention alignée|
    |--|--|
    |`I'm looking for a job with Natural Language Processing`|`GetJobInformation`|
    |`I want to cancel on March 3`|`Utilities.Cancel`|
    |`When were HRF-123456 and hrf-234567 published in the last year?`|`FindForm`|
    |`shift 123-45-6789 from Z-1242 to T-54672`|`MoveEmployee`|
    |`Please relocation jill-jones@mycompany.com from x-2345 to g-23456`|`MoveEmployee`|
    |`Here is my c.v. for the programmer job`|`ApplyForJob`|
    |`This is the lead welder paperwork.`|`ApplyForJob`|
    |`does form hrf-123456 cover the new dental benefits and medical plan`|`FindForm`|
    |`Jill Jones work with the media team on the public portal was amazing`|`EmployeeFeedback`|

    Il existe un seul pool d’énoncés à réviser, quelle que soit la version que vous modifiez ou la version de l’application publiée au point de terminaison.

## <a name="review-endpoint-utterances"></a>Réviser les énoncés de point de terminaison

Passez en revue les énoncés du point de terminaison pour une intention correctement alignée. Bien qu’il n’existe qu’un seul pool d’énoncés à examiner dans toutes les versions, le processus d’alignement correct de l’intention ajoute l’exemple d’énoncé seulement au _modèle actif_ actuel.

1. Dans la section **Build** du portail, sélectionnez **Réviser les énoncés de point de terminaison** dans le volet de navigation gauche. La liste est filtrée pour intention **ApplyForJob**.

    :::image type="content" source="./media/luis-tutorial-review-endpoint-utterances/review-endpoint-utterances-with-entity-view.png" alt-text="Passez de la page Versions à la page Intentions.":::

    Cet énoncé, `I'm looking for a job with Natural Language Processing`, n’a pas l’intention correcte, _GetJobInformation_. Elle a été mal prédite en tant que _ApplyForJob_ en raison de la similarité des noms de postes et des verbes utilisés dans les deux intentions.

1.  Pour aligner cet énoncé, sélectionnez l’**Intention alignée** correcte de `GetJobInformation`. Ajoutez l’énoncé modifié à l’application en sélectionnant la coche.

    Passez en revue les énoncés restants dans cette intention but, en corrigeant l’intention alignée de façon appropriée. Utilisez le tableau des énoncés initiaux dans ce tutoriel pour voir l’intention alignée.

    La liste **Réviser les énoncés de point de terminaison** ne doit plus contenir les énoncés corrigés. Si d’autres énoncés apparaissent, continuez d’examiner la liste, en corrigeant les intentions alignées jusqu’à ce que la liste soit vide.

    Les corrections d’étiquetage de l’entité sont effectuées une fois l’intention alignée, dans la page Détails de l’intention.

1. Former et publier à nouveau l’application.

## <a name="get-intent-prediction-from-endpoint"></a>Obtenir une prédiction d’intention à partir du point de terminaison

Pour vérifier que les exemples d’énoncés correctement alignés ont amélioré la prédiction de l’application, essayez un énoncé proche de l’énoncé corrigé.

1. [!INCLUDE [LUIS How to get endpoint first step](includes/howto-get-endpoint.md)]

1. Allez à la fin de l’URL dans la barre d’adresses, puis remplacez _YOUR_QUERY_HERE_ par `Are there any natural language processing jobs in my department right now?`.

   ```json
    {
        "query": "Are there any natural language processing jobs in my department right now?",
        "prediction": {
            "topIntent": "GetJobInformation",
            "intents": {
                "GetJobInformation": {
                    "score": 0.901367366
                },
                "ApplyForJob": {
                    "score": 0.0307973567
                },
                "EmployeeFeedback": {
                    "score": 0.0296942145
                },
                "MoveEmployee": {
                    "score": 0.00739785144
                },
                "FindForm": {
                    "score": 0.00449316856
                },
                "Utilities.Stop": {
                    "score": 0.00417657848
                },
                "Utilities.StartOver": {
                    "score": 0.00407167152
                },
                "Utilities.Help": {
                    "score": 0.003662492
                },
                "None": {
                    "score": 0.00335733569
                },
                "Utilities.Cancel": {
                    "score": 0.002225436
                },
                "Utilities.Confirm": {
                    "score": 0.00107437756
                }
            },
            "entities": {
                "keyPhrase": [
                    "natural language processing jobs",
                    "department"
                ],
                "datetimeV2": [
                    {
                        "type": "datetime",
                        "values": [
                            {
                                "timex": "PRESENT_REF",
                                "resolution": [
                                    {
                                        "value": "2020-07-02 21:45:50"
                                    }
                                ]
                            }
                        ]
                    }
                ],
                "$instance": {
                    "keyPhrase": [
                        {
                            "type": "builtin.keyPhrase",
                            "text": "natural language processing jobs",
                            "startIndex": 14,
                            "length": 32,
                            "modelTypeId": 2,
                            "modelType": "Prebuilt Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        },
                        {
                            "type": "builtin.keyPhrase",
                            "text": "department",
                            "startIndex": 53,
                            "length": 10,
                            "modelTypeId": 2,
                            "modelType": "Prebuilt Entity Extractor",
                            "recognitionSources": [
                                "model"
                            ]
                        }
                    ],
                    "datetimeV2": [
                        {
                            "type": "builtin.datetimeV2.datetime",
                            "text": "right now",
                            "startIndex": 64,
                            "length": 9,
                            "modelTypeId": 2,
                            "modelType": "Prebuilt Entity Extractor",
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

   Maintenant que les énoncés incertains sont correctement alignés, l’intention correcte a été prédite avec un **score élevé**.

## <a name="can-reviewing-be-replaced-by-adding-more-utterances"></a>La révision peut-elle être remplacée en ajoutant plus d’énoncés ?
Vous vous demandez sûrement pourquoi ne pas ajouter plus d’énoncés d’exemple. Quel est le but de la révision des énoncés de point de terminaison ? Dans une application LUIS réaliste, les énoncés de point de terminaison émanent d’utilisateurs avec des choix de mots et une disposition que vous n’avez pas encore utilisés. Si vous aviez utilisé le même choix de mots et la disposition, la prédiction d’origine aurait un pourcentage plus élevé.

## <a name="why-is-the-top-intent-on-the-utterance-list"></a>Pourquoi la meilleure intention est-elle sur la liste des énoncés ?
Certains des énoncés de point de terminaison auront un score de prédiction élevé dans la liste de révision. Vous devez quand même réviser et vérifier ces énoncés. Ils figurent dans la liste car l’intention suivante la plus haute avait un score trop proche de celui de la meilleure intention. Vous souhaitez une différence d’environ 15 % entre les deux meilleures intentions.

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [LUIS How to clean up resources](../../../includes/cognitive-services-luis-tutorial-how-to-clean-up-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez révisé les énoncés envoyés au point de terminaison et dont LUIS n’était pas sûr. Une fois ces énoncés vérifiés et déplacés vers les intentions correctes en tant qu’exemples d’énoncés, LUIS améliore la précision des prédictions.

> [!div class="nextstepaction"]
> [En savoir plus sur l’utilisation de ces modèles](luis-tutorial-pattern.md)
