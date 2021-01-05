---
title: Précision de l’ajout de modèles - LUIS
titleSuffix: Azure Cognitive Services
description: Ajouter des gabarits de modèle pour améliorer la précision de la prédiction dans les applications Language Understanding (LUIS).
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/17/2020
ms.openlocfilehash: c5e1cc5cdda921e63315c2904d560b7ad3939776
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95018852"
---
# <a name="how-to-add-patterns-to-improve-prediction-accuracy"></a>Comment ajouter des modèles pour améliorer la précision de la prédiction
Lorsqu’une application LUIS reçoit des énoncés de point de terminaison, utilisez un [modèle](luis-concept-patterns.md) pour améliorer la précision de la prédiction des énoncés qui s’appuient sur un modèle d’ordre des mots et de choix de mot. Les modèles utilisent une [syntaxe](luis-concept-patterns.md#pattern-syntax) spécifique pour indiquer l’emplacement des éléments suivants : [entités](luis-concept-entity-types.md), [rôles](./luis-concept-entity-types.md) d’entités et texte facultatif.

> [!CAUTION]
> Les modèles incluent uniquement les entités de machine-learning parent, pas les sous-entités.

## <a name="add-template-utterance-using-correct-syntax"></a>Ajouter un modèle d’énoncé de modèle utilisant une syntaxe correcte

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Dans le volet gauche, sous **Improve app performance** (Améliorer les performances de l’application), sélectionnez **Patterns** (Modèles).

1. Sélectionnez l’intention appropriée pour le modèle.

1. Dans la zone de texte du gabarit, saisissez l’énoncé du gabarit, puis sélectionnez Entrée. Lorsque vous souhaitez entrer le nom de l’entité, utilisez la syntaxe d’entité de modèle appropriée. Commencez la syntaxe de l’entité avec `{`. La liste des entités s’affiche. Sélectionnez la bonne entité.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de l’entité pour le modèle](./media/luis-how-to-model-intent-pattern/patterns-3.png)

    Si votre entité inclut un [rôle](./luis-concept-entity-types.md), indiquez le rôle avec un signe deux-points, `:`, après le nom de l’entité, tel que `{Location:Origin}`. La liste des rôles pour les entités s’affiche dans une liste. Sélectionnez le rôle, puis Entrée.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de l’entité avec le rôle](./media/luis-how-to-model-intent-pattern/patterns-4.png)

    Lorsque vous avez sélectionné l’entité appropriée, terminez d’entrer le modèle, puis sélectionnez Entrée. Lorsque vous avez terminé d’entrer des modèles, [formez](luis-how-to-train.md) votre application.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran du modèle d’entrée avec les deux types d’entités](./media/luis-how-to-model-intent-pattern/patterns-5.png)

## <a name="create-a-patternany-entity"></a>Créer une entité pattern.any

Les entités [pattern.any](luis-concept-entity-types.md) sont uniquement valides dans les [modèles](luis-how-to-model-intent-pattern.md), et non dans les exemples d’énoncés des intentions. Ce type d’entité aide LUIS à trouver la fin des entités présentant une longueur variable et différents choix de mots. Cette entité étant utilisée dans un modèle, LUIS sait où se trouve la fin de l’entité dans le modèle d’énoncé.

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Dans la section **Build**, sélectionnez **Entités** dans le panneau gauche, puis **+ Créer**.

1. Dans la boîte de dialogue **Choisir un type d’entité**, entrez le nom de l’entité dans la zone **Nom**, puis sélectionnez **pattern.any** comme **Type** et sélectionnez **Créer**.

    Une fois que vous [avez créé un énoncé de modèle](luis-how-to-model-intent-pattern.md) à l’aide de cette entité, l’entité est extraite avec un algorithme de machine-learning et de la correspondance à du texte combinés.

## <a name="adding-example-utterances-as-pattern"></a>Ajout d’exemples d’énoncés en tant que modèles

Si vous souhaitez ajouter un modèle pour une entité, la méthode _la plus facile_ consiste à créer le modèle à partir de la page Détails de l’intention. Cela garantit que votre syntaxe correspond à l’exemple d’énoncé.

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Dans la page de la liste **Intentions**, sélectionnez le nom d’intention de l’exemple de énoncé à partir duquel vous souhaitez créer un modèle d’énoncé.
1. Dans la page Détails de l’intention, sélectionnez la ligne de l’exemple d’énoncé que vous souhaitez utiliser comme modèle d’énoncé, puis sélectionnez **+ Ajouter en tant que modèle** dans la barre d’outils contextuelle.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de la sélection d’un exemple d’énoncé comme modèle dans la page Détails de l’intention.](./media/luis-how-to-model-intent-pattern/add-example-utterances-as-pattern-template-utterance-from-intent-detail-page.png)

    L’énoncé doit inclure une entité afin de créer un modèle à partir de l’énoncé.

1. Dans la zone contextuelle, sélectionnez **Terminé** dans la page **Confirmer les modèles**. Vous n’avez pas besoin de définir les sous-entités ou les fonctionnalités des entités. Il vous suffit de répertorier l’entité de machine-learning.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de la confirmation de l’exemple d’énoncé en tant que modèle dans la page Détails de l’intention.](./media/luis-how-to-model-intent-pattern/confirm-patterns-from-example-utterance-intent-detail-page.png)

1. Si vous avez besoin de modifier le modèle, par exemple en sélectionnant un texte facultatif avec les crochets `[]`, vous devez effectuer cette modification à partir de la page **Modèles**.

1. Dans la barre de navigation, sélectionnez **Effectuer l’apprentissage** pour effectuer l’apprentissage de l’application avec le nouveau modèle.

## <a name="train-your-app-after-changing-model-with-patterns"></a>Former votre application après modification du modèle avec des modèles
Après avoir ajouté, modifié, supprimé ou réaffecté un énoncé, [formez](luis-how-to-train.md) et [publiez](luis-how-to-publish-app.md) votre application pour que vos modifications affectent les requêtes de point de terminaison.

<a name="search-patterns"></a>
<a name="edit-a-pattern"></a>
<a name="reassign-individual-pattern-to-different-intent"></a>
<a name="reassign-several-patterns-to-different-intent"></a>
<a name="delete-a-single-pattern"></a>
<a name="delete-several-patterns"></a>
<a name="filter-pattern-list-by-entity"></a>
<a name="filter-pattern-list-by-intent"></a>
<a name="remove-entity-or-intent-filter"></a>
<a name="add-pattern-from-existing-utterance-on-intent-or-entity-page"></a>

## <a name="use-contextual-toolbar"></a>Utiliser la barre d’outils contextuelle

La barre d’outils contextuelle au-dessus de la liste des modèles vous permet d’effectuer les opérations suivantes :

* Rechercher des modèles
* Modifier un modèle
* Réaffecter un modèle à une autre intention
* Réaffecter plusieurs modèles à une autre intention
* Delete-a-single-pattern
* Supprimer plusieurs modèles
* Filtrer la liste des modèles par entité
* Filter-pattern-list-by-intent
* Supprimer un filtre d’entité ou d’intention
* Ajouter un modèle à partir d’un énoncé existant sur une page d’intention ou d’entité

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [créer un modèle](luis-tutorial-pattern.md) avec pattern.any et des rôles à l’aide d’un tutoriel.
* Découvrez comment [former](luis-how-to-train.md) votre application.
