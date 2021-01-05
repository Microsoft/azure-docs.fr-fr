---
title: Ajouter des intentions - LUIS
titleSuffix: Azure Cognitive Services
description: Ajoutez des intentions à votre application LUIS pour identifier des groupes de questions ou de commandes qui ont les mêmes intentions.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/17/2020
ms.service: cognitive-services
ms.openlocfilehash: f2401c032f0fc90024e0049fad5f696b8a184544
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95018937"
---
# <a name="add-intents-to-determine-user-intention-of-utterances"></a>Ajouter des intentions pour déterminer l’intention de l’utilisateur des énoncés

Ajoutez des [intentions](luis-concept-intent.md) à votre application LUIS pour identifier des groupes de questions ou de commandes qui ont la même intention.

Dans le portail LUIS, les intentions sont gérées à partir du volet gauche **Intents**, accessible en cliquant sur la section **Build** de la barre de navigation supérieure.

## <a name="add-an-intent-to-your-app"></a>Ajouter une intention à votre application

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Dans la page **Intentions**, sélectionnez **Créer une intention**.
1. Dans la boîte de dialogue **Créer une intention**, entrez le nom de l’intention, par exemple `ModifyOrder`, puis sélectionnez **Terminé**.

    > [!div class="mx-imgBorder"]
    > ![Ajouter une intention](./media/luis-how-to-add-intents/Addintent-dialogbox.png)

    L’intention a besoin d’[exemples d’énoncés](luis-concept-utterance.md) pour prédire les énoncés au niveau du point de terminaison de prédiction publié.

## <a name="add-an-example-utterance"></a>Ajouter un exemple d’énoncé

Les exemples d’énoncés sont des exemples de texte de questions ou de commandes des utilisateurs. Pour apprendre à Language Understanding (LUIS) à quel moment prédire cette intention, vous devez ajouter des exemples d’énoncés à une intention. LUIS a besoin d’une plage comprise entre 15 et 30 exemples d’énoncés pour commencer à comprendre l’intention. N’ajoutez pas d’exemples d’énoncés en bloc. Chaque énoncé doit être soigneusement choisi en fonction de sa différence par rapport aux exemples qui existent déjà dans l’intention.

1. Dans la page des détails de l’intention, entrez un énoncé pertinent que vous attendez de vos utilisateurs, tel que `Deliver a large cheese pizza`, dans la zone de texte sous le nom d’intention, puis appuyez sur Entrée.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de la page de détails Intentions, avec l’énoncé en surbrillance](./media/luis-how-to-add-intents/add-new-utterance-to-intent.png)

    LUIS convertit tous les énoncés en minuscules et ajoute des espaces autour des [occurrences](luis-language-support.md#tokenization) comme les traits d’union.

<a name="#intent-prediction-discrepancy-errors"></a>

## <a name="intent-prediction-errors"></a>Erreurs de prédiction de l’intention

Une erreur de prédiction d’intention est déterminée quand l’énoncé n’est pas prédit avec l’application entraînée pour l’intention.

1. Pour trouver et corriger les erreurs de prédiction d’énoncés, utilisez les options **Filtre** des prédictions incorrectes et vagues.

    > [!div class="mx-imgBorder"]
    > ![Pour trouver et corriger les erreurs de prédiction d’énoncés, utilisez l’option Filtre.](./media/luis-how-to-add-intents/find-intent-prediction-errors.png)

1. Pour afficher la valeur du score dans la page de détails de l’intention, sélectionnez **Afficher les scores d’intention détaillés** dans le menu d’options **Affichage**.

    Une fois les filtres et la vue appliqués, les exemples d’énoncés comportant des erreurs apparaissent dans la liste avec les problèmes associés.

Chaque ligne indique le score de prédiction de l’apprentissage en cours de l’exemple d’énoncé, le score du rival le plus proche et l’évaluation, à savoir la différence entre ces deux scores.

### <a name="fixing-intents"></a>Corriger les intentions

Pour savoir comment corriger les erreurs d’intention de prédiction, utilisez le [Tableau de bord Résumé](luis-how-to-use-dashboard.md). Il comporte une analyse du dernier apprentissage de la version active et propose les meilleures suggestions de correction du modèle.

## <a name="using-the-contextual-toolbar"></a>Utilisation de la barre d’outils contextuelle

La barre d’outils de contextuelle propose d’autres actions :

* Modifier ou supprimer un exemple d’énoncé
* Réaffecter l’exemple d’énoncé à une autre intention
* Filtres et vues : afficher uniquement les énoncés contenant les entités filtrées ou afficher les détails facultatifs
* Rechercher dans les exemples d’énoncés

## <a name="train-your-app-after-changing-model-with-intents"></a>Former votre application après modification du modèle avec des intentions

Après avoir ajouté, modifié ou supprimé des intentions, [entraînez](luis-how-to-train.md) et [publiez](luis-how-to-publish-app.md) votre application pour que vos modifications soient appliquées aux requêtes de point de terminaison. N’effectuez pas d’entraînement après chaque petite modification. Effectuez l’entraînement après avoir apporté un groupe de modifications.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment ajouter des [exemples d’énoncés](./luis-how-to-add-entities.md) avec des entités.