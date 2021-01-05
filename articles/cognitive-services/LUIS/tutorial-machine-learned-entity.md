---
title: 'Tutoriel : Extraire des données structurées avec une entité issue du machine learning – LUIS'
description: Extrayez des données structurées d’un énoncé à l’aide de l’entité issue du machine learning. Pour augmenter la précision de l’extraction, ajoutez des sous-entités avec des composants.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: tutorial
ms.date: 05/08/2020
ms.openlocfilehash: aa2783abe86b90a907510dec8d7eb40162820ad5
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025257"
---
# <a name="tutorial-extract-structured-data-from-user-utterance-with-machine-learning-entities-in-language-understanding-luis"></a>Tutoriel : Extraire des données structurées à partir d’un énoncé utilisateur avec des entités issues du machine learning dans LUIS

Dans ce tutoriel, vous allez extraire des données structurées d’un énoncé à l’aide de l’entité issue du machine learning.

L’entité issue du machine learning prend en charge le [concept de décomposition du modèle](luis-concept-model.md#v3-authoring-model-decomposition) en fournissant des entités de sous-entités avec leurs [composants](luis-concept-feature.md).

**Dans ce tutoriel, vous allez découvrir comment :**

> [!div class="checklist"]
> * Importer l’exemple d’application
> * Ajouter une entité issue du machine learning
> * Ajouter une sous-entité et un composant
> * Former, tester et publier l’application
> * Obtenir une prédiction d’entité d’un point de terminaison

[!INCLUDE [LUIS Free account](includes/quickstart-tutorial-use-free-starter-key.md)]


## <a name="why-use-a-machine-learning-entity"></a>Pourquoi utiliser une entité issue du machine learning ?

Ce tutoriel ajoute une entité issue du machine learning pour extraire des données d’un énoncé de l’utilisateur.

L’entité définit les données à extraire de l’énoncé. Cela inclut l’attribution aux données d’un nom, d’un type (si possible), d’une résolution en cas d’ambiguïté, et du texte exact qui les compose.

Pour définir les données, vous devez :
* Créer l’entité
* Étiqueter le texte dans des exemples d’énoncés représentant l’entité. Ces exemples étiquetés enseignent à LUIS la nature de l’entité et l’emplacement où elle se trouve dans un énoncé.

## <a name="entity-decomposability-is-important"></a>La possibilité de décomposer l’entité est importante

La possibilité de décomposer l’entité est importante tant pour la prédiction d’intention que pour l’extraction de données avec l’entité.

Commencez avec une entité issue du machine learning, qui est l’entité de début et de niveau supérieur pour l’extraction de données. Décomposez ensuite l’entité en sous-entités.

Si vous ignorez à quel point votre entité doit être détaillée, quand vous démarrez votre application, la bonne pratique consiste à commencer avec une entité issue du machine learning, puis à décomposer celle-ci en sous-entités à mesure que votre application mûrit.

Dans ce tutoriel, vous allez créer une entité issue du machine learning afin de représenter une commande pour une application de pizza. L’entité va extraire le texte relatif à la commande, notamment la taille et la quantité.

Un énoncé de `Please deliver one large cheese pizza to me` doit extraire `one large cheese pizza` comme ordre, puis extraire également `1` pour la quantité et `large` pour la taille.

## <a name="download-json-file-for-app"></a>Télécharger le fichier JSON pour l’application

Téléchargez et enregistrez le [fichier JSON de l’application](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-language-understanding/master/documentation-samples/tutorials/machine-learned-entity/pizza-intents-only.json).

## <a name="import-json-file-for-app"></a>Importer le fichier JSON pour l’application

[!INCLUDE [Import app steps](includes/import-app-steps.md)]

## <a name="create-machine-learned-entity"></a>Créez une entité issue du machine learning

Pour extraire des détails d’une commande de pizza, créez une entité `Order` issue du machine learning de niveau supérieur.

1. Dans la page **intentions**, sélectionnez l’intention **OrderPizza**.

1. Dans la liste exemples d’énoncés, sélectionnez l’énoncé suivant.

    |Exemple d’énoncé de commande|
    |--|
    |`pickup a cheddar cheese pizza large with extra anchovies`|

    Cliquez et faites glisser votre curseur sur le texte pour le premier exemple d’intention. Dans le menu qui s’affiche, entrez le nom de l’entité en tant que `Order`. Sélectionnez ensuite `Order Create new entity` dans la liste.

    ![Étiqueter le début et de fin du texte pour une commande complète](media/tutorial-machine-learned-entity/mark-complete-order.png)

    > [!NOTE]
    > Une entité n’est pas toujours l’intégralité d’un énoncé. Dans ce cas précis, `pickup` indique comment la commande doit être reçue. D’un point de vue conceptuel, `pickup` doit faire partie de l’entité étiquetée pour la commande.

1. Dans la zone **Choisir un type d’entité**, sélectionnez **Ajouter une structure** puis **Suivant**. Une structure est nécessaire pour permettre l’ajout de sous-entités comme la taille et la quantité.

    ![La capture d’écran présente la fenêtre Choisir un type d’entité avec l’option Ajouter une structure cochée.](media/tutorial-machine-learned-entity/add-structure-to-entity.png)

1. Dans la zone **Add subentities (optional)** (Ajouter des sous-entités (facultatif)), sélectionnez **+** sur la ligne `Order`, puis ajoutez `Size` et `Quantity` comme sous-entités, puis sélectionnez **Create**.

    > [!div class="mx-imgBorder"]
    > ![La capture d’écran présente la fenêtre Ajouter des sous-entités (facultative) avec des sous-entités mises en surbrillance.](media/tutorial-machine-learned-entity/add-subentities-when-creating-machine-learned-entity.png)

## <a name="edit-subentities-to-improve-extraction"></a>Modifier les sous-entités pour améliorer l’extraction

Les étapes précédentes créent l’entité et la sous-entité. Pour améliorer l’extraction, ajoutez des caractéristiques aux sous-entités.

### <a name="improve-size-extraction-with-phrase-list"></a>Améliorer l’extraction de la taille avec une liste d’expressions

1. Sélectionnez **Entities** dans le menu de gauche, puis sélectionnez **Order** (Commande).

1. Sous l’onglet **Schema and features** (Schéma et caractéristiques), sélectionnez la sous-entité **Size** (Taille), puis sélectionnez **+ Add feature** (Ajouter une caractéristique).

1. Sélectionnez **Create new phrase list** (Créer une liste d’expressions) dans le menu déroulant.

1. Dans la zone **Créer une liste d’expressions**, entrez le nom `SizePhraselist`, puis entrez les valeurs des options `small`, `medium` et `large`. Quand la zone **Suggestions** se remplit, sélectionnez `extra large` et `xl`. Sélectionnez **Créer** pour créer la liste d’expressions.

    Ce composant de liste d’expressions aide la sous-entité `Size` à trouver des mots associés à la taille en lui fournissant des exemples de mots. Cette liste d’expressions n’a pas besoin d’inclure tous les mots associés à la taille, mais doit inclure des mots censés indiquer la taille.

### <a name="add-sizelist-entity"></a>Ajouter l’entité SizeList

L’ajout d’une liste de tailles courantes reconnues par l’application cliente facilite également l’extraction.

1. Sélectionnez **Entities** dans le menu de gauche, puis sélectionnez **+ Create**.

1. Attribuez à l’entité le nom `SizeListentity` et définissez le type sur **Liste** pour qu’il soit facile de la différencier de la `SizePhraselist` créée à la section précédente.

1. Ajoutez les tailles attendues par l’application cliente (`Small`, `Medium`, `Large` et `XLarge`), puis ajoutez des synonymes pour chacune d’elles. Les synonymes doivent correspondre aux termes qu’un utilisateur entre dans le chatbot. L’entité est extraite avec une entité de liste quand elle correspond exactement à la valeur normalisée ou aux synonymes.

    |Valeur normalisée|Synonymes|
    |--|--|
    |Petite|sm, sml, tiny, smallest|
    |Moyenne|md, mdm, regular, average, middle|
    |grand|lg, lrg, big|
    |XLarge|xl, biggest, giant|


    > [!div class="mx-imgBorder"]
    > ![La capture d’écran présente la fenêtre SizeList et des éléments de liste avec XLarge sélectionné.](media/tutorial-machine-learned-entity/size-list-entity-with-synonyms.png)

### <a name="add-feature-of-sizelist-entity"></a>Ajouter une caractéristique de l’entité SizeList

1. Sélectionnez **Entities** dans le menu de gauche pour revenir à la liste des entités.

1. Sélectionnez **Order** (Commande) dans la liste des entités.

1. Sous l’onglet **Schema and features** (Schéma et caractéristiques), sélectionnez l’entité **Size** (Taille), puis sélectionnez **+ Add feature** (Ajouter une caractéristique).

1. Sélectionnez **@ SizeListentity** dans la liste déroulante.

<a name="add-prebuilt-number-entity-to-app"></a>

### <a name="add-prebuilt-number-entity"></a>Ajouter une entité de nombre prédéfinie

L’ajout d’une entité de nombre prédéfinie facilite également l’extraction.

1. Sélectionnez **Entities** dans le menu de gauche, puis sélectionnez **Add prebuilt entity** (Ajouter une entité prédéfinie).

1. Sélectionnez **Number** dans la liste, puis sélectionnez **Done** (Terminé).

1. Sélectionnez **Entities** dans le menu de gauche pour revenir à la liste des entités.

### <a name="add-feature-of-prebuilt-number-entity"></a>Ajouter une caractéristique de l’entité de nombre prédéfinie

1. Sélectionnez **Order** (Commande) dans la liste des entités.

1. Sous l’onglet **Schema and features** (Schéma et caractéristiques), sélectionnez l’entité **Quantity**, puis sélectionnez **+ Add feature** (Ajouter une caractéristique).

1. Sélectionnez **@ number** dans la liste déroulante.

<a name="create-subcomponent-entity-with-constraint-to-help-extract-data"></a>

## <a name="configure-required-features"></a>Configurer les caractéristiques requises

Dans la page de détails de l’entité **Order**, sélectionnez l’astérisque, `*`, pour les caractéristiques **@ SizeList** et **@ number**. L’astérisque apparaît dans la même étiquette que le nom de la caractéristique.

> [!div class="mx-imgBorder"]
> ![La capture d’écran montre la caractéristique @SizeList avec l’astérisque et l’avertissement Requis.](media/tutorial-machine-learned-entity/set-required-feature-on-subentity.png)

<a name="label-text-as-entities-in-example-utterances"></a>
<a name="label-example-utterance-to-teach-luis-about-the-entity"></a>

## <a name="label-example-utterances"></a>Étiqueter les exemples d’énoncés

L’entité issue du machine learning est créée et les sous-entités ont des caractéristiques. Pour terminer l’amélioration de l’extraction, vous devez étiqueter les exemples d’énoncés avec les sous-entités.

1. Sélectionnez **Intents** (Intentions) dans le volet de navigation gauche, puis choisissez l’intention **OrderPizza**.

1. Pour ouvrir la **palette d’entités**, sélectionnez le symbole **@** dans la barre d’outils contextuelle.

1. Sélectionnez chaque ligne d’entité dans la palette, puis utilisez le curseur de la palette pour sélectionner l’entité dans chaque exemple d’énoncé. Quand vous avez terminé, la liste d’entités doit ressembler à l’image suivante.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran partielle de la configuration d’une caractéristique requise](media/tutorial-machine-learned-entity/labeled-example-utterances-for-machine-learned-entity.png)

## <a name="train-the-app"></a>Effectuer l’apprentissage de l’application

Pour effectuer l’apprentissage de l’application, sélectionnez **Former**. La formation applique les modifications, telles que les nouvelles entités et les énoncés étiquetés, au modèle actif.

## <a name="add-a-new-example-utterance"></a>Ajouter un nouvel exemple d’énoncé

1. Une fois l’entraînement terminé, ajoutez un nouvel exemple d’énoncé à l’intention `OrderPizza` pour voir comment LUIS comprend l’entité issue du machine learning.

    |Exemple d’énoncé de commande|
    |--|
    |`I need a large pepperoni pizza`|

    L’entité supérieure globale, `Order`, est étiquetée. La sous-entité `Size` l’est également avec des lignes en pointillés.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran partielle du nouvel exemple d’énoncé prédit avec une entité](media/tutorial-machine-learned-entity/new-example-utterance-predicted-with-entity.png)

    La ligne en pointillés indique la prédiction basée sur l’application entraînée actuelle.

1. Pour changer la prédiction en entité étiquetée, activez la case à cocher sur la même ligne.

    > [!div class="mx-imgBorder"]
    > ![La capture d’écran présente un exemple d’énoncé avec la coche mise en surbrillance.](media/tutorial-machine-learned-entity/confirm-entity-prediction-for-new-example-utterance-added.png)

    À ce stade, l’entité issue du machine learning fonctionne, car elle peut trouver l’entité dans un nouvel exemple d’énoncé. Lorsque vous ajoutez des exemples d’énoncés, si l’entité n’est pas correctement prédite, étiquetez-la, ainsi que les sous-entités. Si l’entité est correctement prédite, veillez à confirmer les prédictions.


## <a name="train-the-app-to-apply-the-entity-changes-to-the-app"></a>Effectuer l’apprentissage de l’application pour appliquer les modifications d’entité à l’application

Sélectionnez **Train** pour entraîner l’application avec ce nouvel énoncé.

À ce stade, la commande contient des détails qui peuvent être extraits (taille, quantité et texte de la commande totale). Il y a un affinement supplémentaire de l’entité `Order`, comme les garnitures de pizza, le type de croûte et les à côtés. Chacun de ces éléments doit être créé en tant que sous-entités de l’entité `Order`.

## <a name="test-the-app-to-validate-the-changes"></a>Tester l’application pour valider les modifications

Testez l’application à l’aide du volet **Test** interactif. Ce processus vous permet d’entrer un nouvel énoncé, puis d’afficher les résultats de la prédiction pour voir le fonctionnement de l’application active entraînée. La prédiction d’intention doit être assez fiable (au-dessus de 60 %) et l’extraction d’entité doit récupérer au moins l’entité `Order`. Les détails de l’entité Order peuvent être manquants, car ces quelques énoncés ne suffisent pas à gérer tous les cas.

1. Sélectionnez **Tester** dans la barre de navigation supérieure.
1. Entrez l’énoncé `2 small cheese pizzas for pickup`, puis sélectionnez Entrée. Le modèle actif a prédit l’intention correcte avec une fiabilité supérieure à 60 %.


1. Sélectionnez **Inspecter** pour voir les prédictions d’entité.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran partielle de l’affichage des prédictions d’entité dans le panneau de test interactif](media/tutorial-machine-learned-entity/interactive-test-panel-with-first-utterance-and-entity-predictions.png)

<a name="publish-with-sentiment-analysis"></a>

## <a name="publish-the-app-to-access-it-from-the-http-endpoint"></a>Publier l’application pour y accéder à partir du point de terminaison HTTP

Pour recevoir une prédiction LUIS dans un chatbot ou une autre application cliente, vous devez publier l’application sur le point de terminaison.

1. Dans le volet de navigation supérieur droit, sélectionnez **Publier**.

    ![Capture d’écran du bouton LUIS Publier sur le point de terminaison dans le menu en haut à droite](./media/howto-publish/publish-button.png)

1. Sélectionnez l’emplacement **Production**, puis sélectionnez successivement **Change settings** (Changer les paramètres), **Sentiment Analysis** (Analyse des sentiments) et **Done** (Terminé).

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran LUIS - Publier sur le point de terminaison](./media/tutorial-machine-learned-entity/publish-with-sentiment-analysis.png)

1. Sélectionnez le lien **Accéder à vos URL de point de terminaison** dans la notification pour accéder à la page **Ressources Azure**. Les URL de point de terminaison sont répertoriées en tant qu’**Exemple de requête**.

## <a name="get-intent-and-entity-prediction-from-http-endpoint"></a>Obtenir une prédiction d’intention et d’entité à partir d’un point de terminaison HTTP

1. [!INCLUDE [LUIS How to get endpoint first step](includes/howto-get-endpoint.md)]

1. Allez à la fin de l’URL dans la barre d’adresse, puis remplacez _YOUR_QUERY_HERE_ par la requête que vous avez entrée dans le panneau de test interactif.

    `2 small cheese pizzas for pickup`

    Le dernier paramètre de la chaîne de requête est `query`, l’énoncé est **query**.

    ```json
    {
        "query": "2 small cheese pizzas for pickup",
        "prediction": {
            "topIntent": "OrderPizza",
            "intents": {
                "OrderPizza": {
                    "score": 0.7812769
                },
                "None": {
                    "score": 0.0314020254
                },
                "Confirm": {
                    "score": 0.009299271
                },
                "Greeting": {
                    "score": 0.007551549
                }
            },
            "entities": {
                "Order": [
                    {
                        "Size": [
                            "small"
                        ],
                        "Quantity": [
                            2
                        ]
                    }
                ]
            }
        },
        "sentimentAnalysis":{
            "label":"neutral",
            "score":0.98
       }
    }
    ```


[!INCLUDE [LUIS How to clean up resources](includes/quickstart-tutorial-cleanup-resources.md)]

## <a name="related-information"></a>Informations connexes

* [Didacticiel – intentions](./tutorial-intents-only.md)
* [Concept – entités](luis-concept-entity-types.md) informations conceptuelles
* [Concept – fonctionnalités](luis-concept-feature.md) informations conceptuelles
* [Guide pratique pour entraîner](luis-how-to-train.md)
* [Comment publier](luis-how-to-publish-app.md)
* [Guide pratique pour tester dans le portail LUIS](luis-interactive-test.md)

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, l’application utilise une entité issue du machine learning pour rechercher l’intention d’un énoncé d’utilisateur et extraire des détails de cet énoncé. L’utilisation de l’entité issue du machine learning vous permet d’en décomposer les détails.

> [!div class="nextstepaction"]
> [Ajouter une entité keyphrase prédéfinie](./luis-reference-prebuilt-keyphrase.md)