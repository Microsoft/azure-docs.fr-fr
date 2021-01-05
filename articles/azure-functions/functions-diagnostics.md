---
title: Vue d’ensemble des diagnostics Azure Functions
description: Découvrez comment résoudre les problèmes de votre application de fonction avec les diagnostics Azure Functions.
author: yunjchoi
ms.topic: article
ms.date: 11/01/2019
ms.author: yunjchoi
ms.custom: na
ms.openlocfilehash: fb329273d442c023233fee2e7e01aabe5f5bff8c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83122361"
---
# <a name="azure-functions-diagnostics-overview"></a>Vue d’ensemble des diagnostics Azure Functions

Lorsque vous exécutez une application de fonction, vous voulez vous parer aux problèmes qui peuvent survenir, des erreurs 4xx aux échecs de déclenchement. Avec les diagnostics Azure Functions, vous pouvez résoudre les problèmes de votre application de fonction de manière intelligente et interactive sans frais supplémentaires. Lorsque vous rencontrez des problèmes avec votre application de fonction, les diagnostics Azure Functions indiquent ce qui est incorrect. Ils vous guident vers les informations appropriées pour dépanner et résoudre le problème plus facilement et plus rapidement. Cet article explique les principes fondamentaux de l’utilisation des diagnostics Azure Functions pour diagnostiquer et résoudre plus rapidement les problèmes courants liés aux applications de fonction.

## <a name="start-azure-functions-diagnostics"></a>Démarrer les diagnostics d’Azure Functions

Pour démarrer les diagnostics d’Azure Functions :

1. Accédez à votre application de fonction sur le [Portail Azure](https://portal.azure.com).
1. Sélectionnez **diagnostiquer et résoudre les problèmes** pour ouvrir les diagnostics d’Azure Functions.
1. Choisissez la catégorie qui décrit le mieux le problème de votre application de fonction en utilisant les mots clés de la vignette de la page d’accueil. Vous pouvez également saisir le mot clé qui décrit le mieux votre problème dans la barre de recherche. Par exemple, vous pouvez saisir `execution` pour afficher la liste des rapports de diagnostic liés à l’exécution de votre application de fonction et les ouvrir directement à partir de la page d’accueil.

   :::image type="content" source="./media/functions-diagnostics/functions-app-search-azure-functions-diagnostics.png" alt-text="Recherchez les diagnostics d’Azure Functions." border="true":::

## <a name="use-the-interactive-interface"></a>Utiliser l’interface interactive

Lorsque vous sélectionnez la catégorie de la page d’accueil correspondant le mieux au problème rencontré avec votre application de fonction, l’interface interactive des diagnostics Azure Functions, nommée Génie, peut vous aider à diagnostiquer et résoudre le problème. Vous pouvez utiliser les raccourcis de vignette fournis par Génie pour afficher le rapport de diagnostic complet de la catégorie de problème qui vous intéresse. Les raccourcis de vignette vous offrent un moyen d’accès direct aux métriques de diagnostic.

:::image type="content" source="./media/functions-diagnostics/genie.png" alt-text="Recherchez les diagnostics d’Azure Functions." border="false":::

Après avoir sélectionné une vignette, vous pouvez voir une liste des rubriques relatives au problème décrit dans la vignette. Ces rubriques fournissent des extraits du rapport complet, offrant des informations pertinentes. Sélectionnez les rubriques de votre choix pour examiner les problèmes de façon plus approfondie. En outre, vous pouvez sélectionner **Afficher le rapport complet** pour explorer toutes les rubriques sur une seule page.

:::image type="content" source="./media/functions-diagnostics/preview-of-diagnostic-report.png" alt-text="Recherchez les diagnostics d’Azure Functions." border="false":::

## <a name="view-a-diagnostic-report"></a>Afficher un rapport de diagnostic

Une fois que vous avez choisi une rubrique, vous pouvez afficher un rapport de diagnostic spécifique à votre application de fonction. Les rapports de diagnostic utilisent des icônes d’état pour indiquer la présence de problèmes spécifiques dans votre application. Vous y trouverez une description détaillée du problème, des actions recommandées, des mesures associées et des documents utiles. Les rapports de diagnostic personnalisés sont générés à partir d’une série de vérifications exécutées sur votre application de fonction. Les rapports de diagnostic peuvent être un outil utile pour identifier les problèmes dans votre application de fonction et vous guider vers la résolution du problème.

## <a name="find-the-problem-code"></a>Trouver le code problématique

Pour les fonctions basées sur des scripts, vous pouvez utiliser **Exécution de la fonction et Erreurs** sous **Function App cesse de fonctionner ou signale des erreurs** pour circonscrire la ligne de code provoquant des exceptions ou des erreurs. Cet outil permet de déterminer la cause racine et résoudre les problèmes à partir d’une ligne de code spécifique. Cette option n’est pas disponible pour les fonctions précompilées C# et Java.

:::image type="content" source="./media/functions-diagnostics/diagnostic-report-on-function-execution-errors.png" alt-text="Recherchez les diagnostics d’Azure Functions." border="false":::

:::image type="content" source="./media/functions-diagnostics/function-exception.png" alt-text="Recherchez les diagnostics d’Azure Functions." border="false":::

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez poser des questions ou envoyer vos retours sur les diagnostics Azure Functions sur [UserVoice](https://feedback.azure.com/forums/355860-azure-functions). Incluez `[Diag]` dans le titre de vos commentaires.

> [!div class="nextstepaction"]
> [Surveiller vos applications de fonction](functions-monitoring.md)
