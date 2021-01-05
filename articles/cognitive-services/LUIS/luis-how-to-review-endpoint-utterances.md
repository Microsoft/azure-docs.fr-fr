---
title: Passer en revue les énoncés de l’utilisateur – LUIS
titleSuffix: Azure Cognitive Services
description: Passez en revue les énoncés capturés par l’apprentissage actif afin de sélectionner l’intention et marquer les entités pour les énoncés du monde réel ; acceptez les modifications, entraînez et publiez.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 12/08/2020
ms.openlocfilehash: ea2b44d05d25756a16b6b84f0734966b1f579848
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97007600"
---
# <a name="how-to-improve-the-luis-app-by-reviewing-endpoint-utterances"></a>Comment améliorer l’application LUIS en examinant les énoncés de point de terminaison

Le processus de vérification des énoncés de point de terminaison pour obtenir des prédictions correctes est appelé [apprentissage actif](luis-concept-review-endpoint-utterances.md). L’apprentissage actif capture les requêtes de point de terminaison et sélectionne les énoncés de point de terminaison de l’utilisateur dont il n’est pas sûr. Vous passez en revue ces énoncés pour sélectionner l’intention et marquez des entités pour ces énoncés réalistes. Acceptez ces modifications dans vos exemples d’énoncés, puis formez et publiez. Ensuite, LUIS identifie les énoncés de manière plus précise.

## <a name="log-user-queries-to-enable-active-learning"></a>Enregistrer les requêtes utilisateur pour activer l’apprentissage actif

Pour activer l’apprentissage actif, vous devez enregistrer des requêtes utilisateur. Pour cela, vous appelez la [requête de point de terminaison](luis-get-started-create-app.md#query-the-v3-api-prediction-endpoint) avec la valeur et le paramètre querystring `log=true`.

Utilisez le portail LUIS pour créer la requête de point de terminaison correcte.

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et sélectionnez vos **abonnement** et **ressource de création** pour voir les applications affectées à cette dernière.
1. Ouvrez votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).
1. Accédez à la section **Manage** (Gérer), puis sélectionnez **Azure resources** (Ressources Azure).
1. Pour la ressource de prédiction assignée, sélectionnez **Change query parameters** (Modifier les paramètres de requête).

    > [!div class="mx-imgBorder"]
    > ![Capture d'écran représentant le lien de modification des paramètres de requête.](./media/luis-tutorial-review-endpoint-utterances/azure-portal-change-query-url-settings.png)

1. Activez **Save logs** (Enregistrer les journaux), puis enregistrez en sélectionnant **Done**.

    > [!div class="mx-imgBorder"]
    > ![Utilisez le portail LUIS pour enregistrer les journaux, ce qui est nécessaire pour l’apprentissage actif.](./media/luis-tutorial-review-endpoint-utterances/luis-portal-manage-azure-resource-save-logs.png)

     Cette action modifie l’URL de l’exemple en ajoutant le paramètre querystring `log=true`. Copiez et utilisez l’exemple d’URL de requête modifié lors de l’exécution de requêtes de prédiction sur le point de terminaison du runtime.

## <a name="correct-intent-predictions-to-align-utterances"></a>Corriger les prédictions d’intention pour aligner les énoncés

Chaque énoncé contient une intention suggérée affichée dans la colonne **Aligned intent** (Intention alignée).

> [!div class="mx-imgBorder"]
> [![Passez en revue les énoncés de point de terminaison dont LUIS n’est pas sûr](./media/label-suggested-utterances/review-endpoint-utterances.png)](./media/label-suggested-utterances/review-endpoint-utterances.png#lightbox)

Si vous acceptez cette intention, cochez la case. Si n’acceptez pas la suggestion, sélectionnez l’intention appropriée dans la liste déroulante de l’intention alignée, puis sélectionnez la coche à droite de l’intention alignée. Une fois que vous avez coché la case, l’énoncé est déplacé vers l’intention et supprimé de la liste **Réviser les énoncés de point de terminaison**.

> [!TIP]
> Il est important d’accéder à la page des détails de l’intention pour passer en revue et corriger les prédictions d’entité de tous les exemples d’énoncés de la liste **Réviser les énoncés de point de terminaison**.

## <a name="delete-utterance"></a>Supprimer l’énoncé

Chaque énoncé peut être supprimé de la liste de révision. Une fois supprimé, il n’apparaîtra plus dans la liste. Cela est vrai même si l’utilisateur entre le même énoncé à partir du point de terminaison.

Si vous ne savez pas si vous devez supprimer l’énoncé, déplacez-le vers l’intention None (Aucune), ou créez une nouvelle intention comme `miscellaneous` et déplacez l’énoncé vers cette intention.

## <a name="disable-active-learning"></a>Désactiver l’apprentissage actif

Pour désactiver l’apprentissage actif, n’enregistrez pas des requêtes de l’utilisateur. Pour cela, définissez la [requête de point de terminaison](luis-get-started-create-app.md#query-the-v2-api-prediction-endpoint) avec le paramètre et la valeur querystring `log=false` ou n’utilisez pas la valeur querystring, car la valeur par défaut est false.

## <a name="next-steps"></a>Étapes suivantes

Pour tester l’amélioration de la performance après avoir étiqueté les énoncés suggérés, vous pouvez accéder à la console de test en sélectionnant **Test** dans le panneau supérieur. Pour obtenir des instructions sur la façon de tester votre application à l’aide de la console de test, consultez [Former et tester votre application](luis-interactive-test.md).
