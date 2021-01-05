---
title: Utiliser les fonctionnalités pour améliorer la liste de mots LUIS
titleSuffix: Azure Cognitive Services
description: Utiliser Language Understanding (LUIS) pour ajouter des fonctionnalités d’application pouvant améliorer la détection ou la prédiction d’intentions et d’entités dans des catégories et des modèles
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/17/2020
ms.openlocfilehash: 33435d2367b92f312d41f0e32fdbb3a93d59a9fd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999924"
---
# <a name="use-features-to-boost-signal-of-word-list"></a>Utiliser des fonctionnalités pour améliorer le signal de liste de mots

Vous pouvez ajouter des fonctionnalités à votre application LUIS pour améliorer sa précision. Ces fonctionnalités aident LUIS en indiquant que certains mots et expressions font partie d’un vocabulaire de domaine d’application.

Passez en revue les [concepts](luis-concept-feature.md) pour comprendre quand et pourquoi utiliser une fonctionnalité.

## <a name="add-phrase-list-as-a-feature"></a>Ajouter une liste d’expressions en tant que fonctionnalité

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Sélectionnez **Créer**, puis **Fonctionnalités** dans le panneau gauche de votre application.

1. Dans la page **Fonctionnalités**, sélectionnez **+ Créer**.

1. Dans la boîte de dialogue **Créer une fonctionnalité de liste d’expressions**, entrez un nom comme `Cities`. Dans la zone **Valeur**, entrez des exemples de villes, comme `Seattle`. Vous pouvez taper une valeur à la fois, ou un ensemble de valeurs séparées par des virgules, puis appuyer sur **Entrée**.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de l’ajout de la fonctionnalité (liste d’expressions) Villes](./media/luis-add-features/add-phrase-list-cities.png)

    Une fois que vous avez entré suffisamment de valeurs pour LUIS, des suggestions apparaissent. Vous pouvez cliquer sur **+ Ajouter** pour toutes les valeurs proposées ou sélectionner des termes individuels.

1. Laissez la case **Ces valeurs sont interchangeables** cochée si les expressions peuvent être utilisées indifféremment.

1. La liste d’expressions peut s’appliquer à l’application entière avec le paramètre **Global** ou à un modèle spécifique (intention ou entité). Si vous créez la liste d’expressions en tant que _fonctionnalité_ à partir d’une intention ou d’une entité, le bouton bascule n’est pas défini sur global. Dans ce cas, la signification du bouton bascule est que la fonctionnalité est propre à ce modèle uniquement, par conséquent _non globale_ à l’application.

1. Sélectionnez **Terminé**. La nouvelle fonctionnalité est ajoutée à la page **Fonctionnalités ML**.

<a name="edit-phrase-list"></a>
<a name="delete-phrase-list"></a>
<a name="deactivate-phrase-list"></a>


> [!Note]
> Vous pouvez supprimer ou désactiver une liste d’expressions dans la barre d’outils contextuelle de la page **Fonctionnalités ML**.

## <a name="global-phrase-list-applies-to-entire-app"></a>La liste des expressions globales s’applique à l’ensemble de l’application

Une liste d’expressions doit être appliquée à l’intention ou à l’entité qu’elle est destinée à aider, mais il peut arriver qu’une liste d’expressions soit appliquée à l’application entière en tant que fonctionnalité **Globale**.

Dans la page Fonctionnalités ML, sélectionnez la liste d’expressions, puis sélectionnez **Rendre global** dans la barre d’outils contextuelle supérieure.

## <a name="model-as-a-feature"></a>Modèle en tant que fonctionnalité

Une entité peut être une [fonctionnalité ajoutée à une intention ou une entité](luis-concept-feature.md).

Pour ajouter une entité en tant que fonctionnalité à une intention, sélectionnez l’intention dans la page intentions, puis sélectionnez **+ Ajouter une fonctionnalité** au-dessus de la barre d’outils contextuelle. La liste inclut toutes les listes d’expressions et les entités qui peuvent être appliquées en tant que fonctionnalités.

Pour ajouter une entité en tant que fonctionnalité à une autre entité, vous pouvez ajouter la fonctionnalité sur la page de détails d’intention à l’aide de la [Palette d’entités](label-entity-example-utterance.md#adding-entity-as-a-feature-from-the-entity-palette) ou vous pouvez [Ajouter la fonctionnalité](luis-how-to-add-entities.md#add-a-feature-to-a-machine-learned-entity) sur la page de détails de l’entité.

## <a name="next-steps"></a>Étapes suivantes

Après l’ajout, la modification, la suppression ou la désactivation d’une fonctionnalité, [réentraînez et retestez l’application](luis-interactive-test.md) pour voir si les performances s’améliorent.
