---
title: Modèles prédéfinis pour Language Understanding
titleSuffix: Azure Cognitive Services
description: LUIS inclut un ensemble de modèles prédéfinis pour ajouter rapidement des scénarios de conversation utilisateur courants.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 05/17/2020
ms.openlocfilehash: 2080139732d71aa5df9a9d4ad13fcc4545128dff
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91541915"
---
# <a name="add-prebuilt-models-for-common-usage-scenarios"></a>Modèles prédéfinis pour les scénarios d’usage courants

LUIS inclut un ensemble de modèles prédéfinis pour ajouter rapidement des scénarios de conversation utilisateur courants. Il s’agit d’un moyen simple et rapide d’ajouter des fonctionnalités à votre application cliente conversationnelle sans avoir à concevoir les modèles pour ces fonctionnalités.

## <a name="add-a-prebuilt-domain"></a>Ajouter un domaine prédéfini

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).

1. Sélectionnez **Prebuilt Domains** (Domaines prédéfinis) dans la barre d’outils gauche.

1. Recherchez le domaine à ajouter à l’application, puis sélectionnez le bouton **Add domain** (Ajouter un domaine).

    > [!div class="mx-imgBorder"]
    > ![Ajouter le domaine prédéfini Calendrier](./media/luis-prebuilt-domains/add-prebuilt-domain.png)

## <a name="add-a-prebuilt-intent"></a>Ajouter une intention prédéfinie

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).

1. Dans la page **Intents** (Intentions), sélectionnez **Add prebuilt domain intent** (Ajouter une intention de domaine prédéfinie) à partir de la barre d’outils au-dessus de la liste des intentions.

1. Sélectionnez une intention dans la boîte de dialogue contextuelle.

    > [!div class="mx-imgBorder"]
    > ![Ajouter une intention prédéfinie](./media/luis-prebuilt-domains/add-prebuilt-domain-intents.png)

1. Cliquez sur le bouton **Terminé**.

## <a name="add-a-prebuilt-entity"></a>Ajouter une entité prédéfinie
1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Sur la gauche, sélectionnez **Entités**.

1. Dans la page **Entities** (Entités), sélectionnez **Add prebuilt entity** (Ajouter une entité prédéfinie).

1. Dans la boîte de dialogue **Add prebuilt entities** (Ajouter des entités prédéfinies), sélectionnez l’entité prédéfinie.

    > [!div class="mx-imgBorder"]
    > ![Boîte de dialogue Add prebuilt entities (Ajouter des entités prédéfinies)](./media/luis-prebuilt-domains/add-prebuilt-entity.png)

1. Sélectionnez **Terminé**. Une fois l’entité ajoutée, vous n’avez pas besoin d’effectuer l’apprentissage de l’application.

## <a name="add-a-prebuilt-domain-entity"></a>Ajouter une entité de domaine prédéfinie
1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Sur la gauche, sélectionnez **Entités**.

1. Dans la page **Entities** (Entités), sélectionnez **Add prebuilt domain entity** (Ajouter une entité de domaine prédéfinie).

1. Dans la boîte de dialogue **Add prebuilt domain models** (Ajouter des modèles de domaine prédéfinis), sélectionnez l’entité de domaine prédéfinie.

1. Sélectionnez **Terminé**. Une fois l’entité ajoutée, vous n’avez pas besoin d’effectuer l’apprentissage de l’application.

## <a name="publish-to-view-prebuilt-model-from-prediction-endpoint"></a>Publier pour voir le modèle prédéfini à partir du point de terminaison de prédiction

La manière la plus simple de visualiser la valeur d’un modèle prédéfini consiste à interroger cette valeur à partir du point de terminaison publié.

## <a name="entities-containing-a-prebuilt-entity-token"></a>Entités contenant un jeton d’entité prédéfinie

Si l’une de vos entités de Machine Learning a besoin d’une fonctionnalité requise d’une entité prédéfinie, ajoutez-y une sous-entité à l’entité de Machine Learning, puis ajoutez une fonctionnalité _requise_ d’une entité prédéfinie.

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Générer un modèle à partir d’un fichier .csv avec des API REST](./luis-tutorial-node-import-utterances-csv.md)
