---
title: Tester une application dans le portail LUIS
description: Utilisez LUIS (Language Understanding) pour travailler en continu sur votre application afin d’affiner et d’améliorer sa compréhension de la langue.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 06/02/2020
ms.openlocfilehash: b6fcc294c2b8c131d9a4a058c653ae1f64652ccf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91324856"
---
# <a name="test-your-luis-app-in-the-luis-portal"></a>Tester votre application LUIS dans le portail LUIS

[Le test](luis-concept-test.md) d’une application est un processus itératif. Après avoir formé votre application LUIS, testez-la avec des exemples d’énoncés afin de voir si les intentions et les entités sont reconnues correctement. Si ce n’est pas le cas, apportez des mises à jour à l’application LUIS, puis formez-la et testez-la à nouveau.

<!-- anchors for H2 name changes -->
<a name="train-your-app"></a>
<a name="test-your-app"></a>
<a name="access-the-test-page"></a>
<a name="luis-interactive-testing"></a>

## <a name="train-before-testing"></a>Effectuer l’apprentissage avant le test

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Pour tester la version la plus récente de l’application active, sélectionnez **Effectuer l’apprentissage** dans le menu supérieur avant de réaliser le test.

## <a name="test-an-utterance"></a>Tester un énoncé

L’énoncé de test ne doit pas être exactement le même que les exemples d’énoncés donnés dans l’application. Il doit inclure le choix de mots, la longueur de l’expression et l’utilisation de l’entité que vous attendez d’un utilisateur.

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).

1. Pour accéder au panneau déroulant **Test**, sélectionnez **Test** dans le panneau supérieur de votre application.

    > [!div class="mx-imgBorder"]
    > ![Page Effectuer l’apprentissage de l’application et la tester](./media/luis-how-to-interactive-test/test.png)

1. Entrez un énoncé dans la zone de texte, puis sélectionnez Entrée. Vous pouvez entrer autant d’énoncés de test que vous le souhaitez dans **Test**, mais uniquement un énoncé à la fois.

1. L’énoncé, son intention principale et son score sont ajoutés à la liste des énoncés sous la zone de texte.

    > [!div class="mx-imgBorder"]
    > ![Le test interactif identifie l’intention incorrecte](./media/luis-how-to-interactive-test/test-weather-1.png)

## <a name="inspect-the-prediction"></a>Inspecter la prédiction

Vous inspectez les détails du résultat du test dans le panneau **Inspecter**.

1. Avec le panneau déroulant **Test** ouvert, sélectionnez **Inspecter** pour consulter un énoncé à comparer.

    > [!div class="mx-imgBorder"]
    > ![Sélectionnez le bouton Inspecter pour afficher plus de détails sur les résultats de test](./media/luis-how-to-interactive-test/inspect.png)

1. Le panneau **Inspection** s’affiche. Le panneau inclut l’intention de score la plus élevée, ainsi que toutes les entités identifiées. Le panneau affiche la prédiction de l’énoncé sélectionné.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran partielle du panneau Inspection de test](./media/luis-how-to-interactive-test/inspect-panel.png)

## <a name="add-to-example-utterances"></a>Ajouter des exemples d’énoncés

À partir du panneau d’inspection, vous pouvez ajouter l’énoncé de test à une intention en sélectionnant **Ajouter aux exemples d’énoncés**.

## <a name="disable-required-features"></a>Désactiver les fonctionnalités requises

Ce bouton permet de déterminer si l’application formée prédit correctement les entités en fonction des fonctionnalités requises. Le réglage par défaut consiste à appliquer la caractéristique comme requis lors de la prédiction. Sélectionnez ce bouton pour voir quelle serait la prédiction si la caractéristique de la sous-entité n’était pas requise.

### <a name="when-to-disable-required-features"></a>Quand désactiver les caractéristiques requises

L’application formée peut prédire de manière erronée une entité issue du Machine Learning en fonction de l’un des éléments suivants :
* Étiquetage incorrect des énoncés d’exemples.
* Non-correspondance entre le texte et la caractéristique obligatoire.

Par exemple : une entité issue du Machine Learning avec une sous-entité du nom d’une personne.

:::image type="content" source="media/luis-how-to-interactive-test/disable-required-feature.png" alt-text="Capture d’écran du portail LUIS : schéma d’entités issues du Machine Learning avec la caractéristique requise":::

Voici un exemple d’énoncé pour cette entité issue du Machine Learning : `Assign Bob Jones to work on the new security feature`.

L’extraction doit être `security feature` comme description de ticket et `Bob Jones` comme ingénieur, deux sous-entités de l’entité `Assign ticket`.

Pour aider la sous-entité à prédire correctement, ajoutez l’entité prédéfinie [PersonName](luis-reference-prebuilt-person.md) comme caractéristique à la sous-entité `engineer`. Si vous rendez la caractéristique obligatoire, cela signifie que la sous-entité sera extraite uniquement si l’entité prédéfinie PersonName est prédite pour le texte. Cela signifie que tout nom figurant dans le texte qui n’est pas prévisible avec la sous-entité PersonName ne sera pas retourné comme sous-entité étiquetée `engineer`.

Lorsque vous utilisez le volet de test interactif et que vous voyez qu’une sous-entité, avec une caractéristique obligatoire, n’est pas prédite, désactivez ce paramètre pour voir si la sous-entité serait prédite sans que la caractéristique ne soit obligatoire. Il est possible que la sous-entité puisse être correctement prédite sans la caractéristique obligatoire en raison de l’étiquetage correct des énoncés d’exemple.

## <a name="view-sentiment-results"></a>Afficher les résultats des sentiments

Si **Analyse des sentiments** est configurée sur la page **[Publier](luis-how-to-publish-app.md#enable-sentiment-analysis)** , les résultats du test incluent le sentiment trouvé dans l’énoncé.

## <a name="correct-matched-patterns-intent"></a>Corriger l’intention du modèle mis en correspondance

Si vous utilisez [Modèles](luis-concept-patterns.md) et si l’énoncé a été mis en correspondance avec un modèle, mais que l’intention prédite est incorrecte, sélectionnez le lien **Modifier** en regard du modèle, puis sélectionnez l’intention correcte.

## <a name="compare-with-published-version"></a>Comparer avec la version publiée

Vous pouvez tester la version active de votre application avec la version de [point de terminaison](luis-glossary.md#endpoint) publiée. Dans le panneau **Inspecter**, sélectionnez **Comparer avec la version publiée**. Tout test exécuté sur le modèle publié est déduit du solde de votre quota d’abonnement Azure.

> [!div class="mx-imgBorder"]
> ![Comparer avec la version publiée](./media/luis-how-to-interactive-test/inspect-panel-compare.png)

## <a name="view-endpoint-json-in-test-panel"></a>Afficher le JSON de point de terminaison dans le panneau de test
Vous pouvez afficher le JSON de point de terminaison retourné pour la comparaison en sélectionnant la **vue Afficher JSON**.

> [!div class="mx-imgBorder"]
> ![Réponse JSON publiée](./media/luis-how-to-interactive-test/inspect-panel-compare-json.png)

## <a name="additional-settings-in-test-panel"></a>Paramètres supplémentaires dans le panneau de test

### <a name="luis-endpoint"></a>Point de terminaison LUIS

Si vous avez plusieurs points de terminaison LUIS, utilisez le lien **Paramètres supplémentaires** dans le panneau Test publié pour modifier le point de terminaison utilisé pour le test. Si vous ne savez pas quel point de terminaison utiliser, sélectionnez la valeur par défaut **Starter_Key**.

> [!div class="mx-imgBorder"]
> ![Panneau de test avec le lien Paramètres supplémentaires mis en surbrillance](media/luis-how-to-interactive-test/additional-settings-v3-settings.png)


## <a name="batch-testing"></a>Test par lot
Consultez les [concepts](luis-concept-batch-test.md) du test par lot et apprenez [comment](luis-how-to-batch-test.md) tester un lot d’énoncés.

## <a name="next-steps"></a>Étapes suivantes

Si le test indique que votre application LUIS ne reconnaît pas les entités et les intentions correctes, vous pouvez travailler à l’amélioration de la précision de votre application LUIS en étiquetant plus d’énoncés ou en ajoutant des fonctionnalités.

* [Étiqueter des énoncés suggérés avec LUIS](luis-how-to-review-endpoint-utterances.md)
* [Utiliser les fonctionnalités pour améliorer les performances de votre application LUIS](luis-how-to-add-features.md)
