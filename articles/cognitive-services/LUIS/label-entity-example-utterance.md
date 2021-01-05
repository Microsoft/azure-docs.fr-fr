---
title: Étiqueter une entité dans un exemple d’énoncé
description: Découvrez comment étiqueter une entité issue du Machine Learning avec des sous-entités dans un exemple d’énoncé, dans une page de détails d’intention du portail LUIS.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 05/20/2020
ms.openlocfilehash: ffbaa2e40d5924ba61e548398e63295cf7dba2b0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019733"
---
# <a name="label-machine-learning-entity-in-an-example-utterance"></a>Étiqueter une entité issue du Machine Learning dans un exemple d’énoncé

L’étiquetage d’une entité dans un exemple d’énoncé donne à LUIS un exemple de cette entité et indique où celle-ci peut apparaître dans l’énoncé.

Vous pouvez étiqueter les entités et sous-entités apprises issues du Machine Learning.

Comme vous ne pouvez pas étiqueter d’expression régulière, de liste ou d’entités prédéfinies, créez une entité ou une sous-entité, puis ajoutez celle-ci comme fonctionnalité, le cas échéant, à l’entité ou à la sous-entité.

## <a name="label-example-utterances-from-the-intent-detail-page"></a>Étiqueter des exemples d’énoncés à partir de la page de détails de l’intention

Pour étiqueter des exemples d’entités au sein de l’énoncé, sélectionnez l’intention de l’énoncé.

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Sélectionnez l’intention qui contient les exemples d’énoncés que vous souhaitez étiqueter pour l’extraction avec une entité.
1. Sélectionnez le texte à étiqueter, puis sélectionnez l’entité.

## <a name="two-techniques-to-label-entities"></a>Deux techniques pour étiqueter des entités

La page de détail de l’intention prend en charge deux techniques d’étiquetage.
* Sélectionnez une entité ou une sous-entité dans la [palette des entités](#label-with-the-entity-palette-visible) puis faites votre sélection dans l’exemple de texte d’énoncé. Il s’agit de la technique recommandée, car vous pouvez vérifier visuellement que vous travaillez avec l’entité ou la sous-entité appropriée, en fonction de votre schéma.
* Commencez par faire votre sélection dans l’exemple de texte d’énoncé. À ce moment-là, un menu contextuel des [options d’étiquetage](#how-to-label-entity-from-in-place-menu) apparaît.

## <a name="label-with-the-entity-palette-visible"></a>Étiqueter à l’aide de la palette des entités visible

Une fois que vous avez [planifié votre schéma avec des entités](luis-how-plan-your-app.md), gardez la **palette des entités** visible lors de l’étiquetage. La **palette des entités** est un rappel des entités dont vous avez planifié l’extraction.

Pour accéder à la **palette des entités**, sélectionnez le symbole **@** dans la barre d’outils contextuelle au-dessus de l’exemple de liste d’énoncés.

> [!div class="mx-imgBorder"]
> ![Capture d’écran de la page Détails de l’intention de la palette des entités.](media/label-utterances/entity-palette-from-tool-bar.png)

## <a name="how-to-label-entity-from-entity-palette"></a>Comment étiqueter une entité depuis la palette des entités

La palette des entités offre une alternative à l’expérience d’étiquetage précédente. Elle vous permet de surligner du texte afin de l’étiqueter instantanément avec une entité.

1. Ouvrez la palette des entités en sélectionnant le symbole **@** en haut à droite de la table des énoncés.

2. Sélectionnez l’entité que vous souhaitez étiqueter depuis la palette. Cette action est indiquée visuellement par un nouveau curseur. Le curseur suit les déplacements de la souris dans le portail LUIS.

3. Dans l’exemple d’énoncé, _peignez_ l’entité avec le curseur.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran montrant l’entité peinte avec le curseur.](media/label-utterances/example-1-label-machine-learned-entity-palette-label-action.png)

## <a name="adding-entity-as-a-feature-from-the-entity-palette"></a>Ajout d’une entité en tant que fonctionnalité à partir de la palette des entités

La section inférieure de la palette des entités vous permet d’ajouter des fonctionnalités à l’entité actuellement sélectionnée. Vous pouvez faire votre sélection parmi toutes les entités et les listes d’expressions existantes, ou créer une nouvelle liste d’expressions.

> [!div class="mx-imgBorder"]
> ![Capture d’écran de la palette des entités avec l’entité en tant que fonctionnalité](media/label-utterances/entity-palette-entity-as-a-feature.png)

## <a name="labeling-entity-roles"></a>Étiquetage des rôles d’entité

Les rôles d’entité sont étiquetés à l’aide de la **palette des entités**.

1. Dans la page des détails de l’intention, sélectionnez la **palette d’entités** dans la barre d’outils contextuelle.
1. Une fois la palette d’entités ouverte, sélectionnez l’entité dans la liste des entités.
1. Sous la liste des entités, sélectionnez un rôle existant.
1. Dans l’exemple de texte d’énoncé, étiquetez le texte avec le rôle d’entité.

## <a name="how-to-label-entity-from-in-place-menu"></a>Comment étiqueter une entité depuis le menu sur place

L’étiquetage sur place vous permet de sélectionner rapidement le texte dans l’énoncé et de l’étiqueter. Vous pouvez également créer une entité de Machine Learning ou une entité de liste à partir du texte étiqueté.

Prenez l’exemple d’énoncé, `hi, please I want a cheese pizza in 20 minutes`.

Sélectionnez le texte le plus à gauche, puis sélectionnez le texte le plus à droite de l’entité. Ensuite, choisissez l’entité que vous souhaitez étiqueter depuis le menu sur place.

> [!div class="mx-imgBorder"]
> ![Étiquetage de l’entité complète issue du Machine Learning](media/label-utterances/label-steps-in-place-menu.png)

## <a name="review-labeled-text"></a>Vérifier le texte étiqueté

Après l’étiquetage, vérifiez l’exemple d’énoncé et vérifiez que l’étendue de texte sélectionnée a été soulignée avec l’entité choisie. La ligne pleine indique que le texte a été étiqueté.

> [!div class="mx-imgBorder"]
> ![Entité complète issue du Machine Learning étiquetée](media/label-utterances/example-1-label-machine-learned-entity-complete-order-labeled.png)

## <a name="confirm-predicted-entity"></a>Vérifier l’entité prédite

Si l’étendue de texte est encadrée par des pointillés, cela indique que le texte est prédit mais qu’il n’est _pas encore étiqueté_. Pour transformer la prédiction en étiquette, sélectionnez la ligne de l’énoncé, puis sélectionnez **Confirmer les entités** dans la barre d’outils contextuelle.

## <a name="relabeling-over-existing-entities"></a>Réétiquetage d’entités existantes

Si vous réétiquetez du texte déjà étiqueté, LUIS peut fractionner ou fusionner les étiquettes existantes.

## <a name="labeling-for-punctuation"></a>Étiquetage de la ponctuation

Vous n’avez pas besoin d’étiqueter la ponctuation. Utilisez les [paramètres d’application](luis-reference-application-settings.md) pour contrôler l’impact de la ponctuation sur les prédictions d’énoncé.

## <a name="unlabel-entities"></a>Supprimer les étiquettes des entités

> [!NOTE]
> Seules les entités issues du Machine Learning peuvent être désétiquetées. Vous ne pouvez pas étiqueter ou désétiqueter des entités d’expression régulière, des entités de liste ou des entités prédéfinies.

Pour supprimer l’étiquette d’une entité, choisissez le nom de l’entité et sélectionnez **Unlabel** (Supprimer l’étiquette) depuis le menu sur place.

> [!div class="mx-imgBorder"]
> ![Capture d’écran montrant une entité désétiquetée](media/label-utterances/unlabel-entity-using-in-place-menu.png)

## <a name="automatic-labeling-for-parent-and-child-entities"></a>Étiquetage automatique pour les entités parent et enfant

Lorsque vous étiquetez une entité parente, toute sous-entité pouvant être prédite en se basant sur la version actuellement en formation est étiquetée.

Lorsque vous étiquetez une sous-entité, le parent est automatiquement étiqueté.

## <a name="automatic-labeling-for-non-machine-learned-entities"></a>Étiquetage automatique pour les entités non issues du Machine Learning

Les entités non issues du Machine Learning incluent les entités prédéfinies, les entités d’expression régulière, les entités de liste et toutes les entités de modèle. Celles-ci sont automatiquement étiquetées par LUIS. Vous n’avez donc pas à le faire manuellement.

## <a name="intent-prediction-errors"></a>Erreurs de prédiction de l’intention

Une erreur de prédiction d’intention indique que l’exemple d’énoncé, compte tenu de l’application actuellement en formation, ne serait pas prédit pour l’intention.

Découvrez comment [afficher ces erreurs](luis-how-to-add-intents.md#intent-prediction-errors) sur la page de détails de l’intention.

## <a name="entity-prediction-errors"></a>Erreurs de prédiction des entités

Les erreurs de prédiction d’entité indiquent que l’entité prédite ne correspond pas à l’entité étiquetée. Ces erreurs sont signalées par la présence d’un indicateur d’avertissement en regard de l’énoncé.

> [!div class="mx-imgBorder"]
> ![Palette des entités pour une entité issue du Machine Learning](media/label-utterances/example-utterance-indicates-prediction-error.png)

## <a name="next-steps"></a>Étapes suivantes

Dans le [tableau de bord](luis-how-to-use-dashboard.md), [passez en revue les énoncés de point de terminaison](luis-how-to-review-endpoint-utterances.md) pour améliorer la qualité de prédiction de votre application.
