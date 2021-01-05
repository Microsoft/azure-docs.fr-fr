---
title: Tableau de bord - Language Understanding - LUIS
titleSuffix: Azure Cognitive Services
description: Corriger les intentions et les entités avec le tableau de bord de votre application entraînée. Le tableau de bord affiche des informations sur l’application globale, en soulignant les intentions qui doivent être corrigées.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 11/08/2019
ms.openlocfilehash: 4867a065a85fab1e4abc7f19401239e5b76e1da4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91541405"
---
# <a name="how-to-use-the-dashboard-to-improve-your-app"></a>Guide pratique pour utiliser le tableau de bord afin d’améliorer votre application

Recherchez et corrigez les problèmes liés aux intentions de votre application entraînée quand vous utilisez des exemples d’énoncés. Le tableau de bord affiche des informations sur l’application globale, en soulignant les intentions qui doivent être corrigées.

L’examen de l’analyse du tableau de bord est un processus itératif, répété à mesure que vous changez et améliorez votre modèle.

Cette page ne contiendra pas une analyse pertinente pour les applications dont les intentions n’ont pas d’exemples d’énoncés (elles sont connues sous le nom d’applications _modèle-uniquement_).

## <a name="what-issues-can-be-fixed-from-dashboard"></a>Quels sont les problèmes qui peuvent être résolus à partir du tableau de bord ?

Les trois problèmes abordés dans le tableau de bord sont :

|Problème|Couleur de graphique|Explication|
|--|--|--|
|Déséquilibre des données|-|Cela se produit quand la quantité d’exemples d’énoncés varie considérablement. Toutes les intentions doivent avoir _à peu près_ le même nombre d’exemples d’énoncés, sauf l’intention Aucune. Elle doit avoir uniquement 10 à 15 % de la quantité totale d’énoncés dans l’application.<br><br> Si les données sont déséquilibrées, mais que la précision d’intention se situe au-dessus d’un certain seuil, ce déséquilibre n’est pas signalé comme un problème.<br><br>**Commencez par ce problème ; il peut être la cause racine des autres problèmes.**|
|Prédictions vagues|Orange|Cela se produit quand l’intention supérieure et les scores de l’intention suivante sont suffisamment proches pour pouvoir basculer vers l’entraînement suivant, en raison d’un [échantillonnage négatif](luis-how-to-train.md#train-with-all-data) ou de l’ajout d’exemples d’énoncés supplémentaires à l’intention. |
|Prédictions incorrectes|Rouge|Cela se produit quand un exemple d’énoncé n’est pas prédit pour l’intention étiquetée (l’intention dans laquelle il figure).|

Les prédictions correctes sont représentées par la couleur bleue.

Le tableau de bord affiche ces problèmes, indique les intentions affectées et suggère la procédure à suivre pour améliorer l’application.

## <a name="before-app-is-trained"></a>Avant l’entraînement de l’application

Avant l’entraînement de l’application, le tableau de bord ne contient aucune suggestion concernant les corrections. Vous devez entraîner votre application pour voir ces suggestions.

## <a name="check-your-publishing-status"></a>Vérifier l’état de la publication

La carte **État de publication** contient des informations sur la dernière publication de la version active.

Vérifiez que la version active est celle que vous souhaitez corriger.

![Le tableau de bord affiche les services externes de l’application, les régions publiées et les accès aux points de terminaison agrégés.](./media/luis-how-to-use-dashboard/analytics-card-1-shows-app-summary-and-endpoint-hits.png)

Il affiche également les services externes, les régions publiées et les accès aux points de terminaison agrégés.

## <a name="review-training-evaluation"></a>Passer en revue l’évaluation de l’entraînement

La carte **Évaluation de l’apprentissage** contient la synthèse agrégée de la précision globale de votre application par zone. Le score indique la qualité des intentions.

![La carte Évaluation de l’apprentissage contient la première zone d’informations concernant la précision globale de votre application.](./media/luis-how-to-use-dashboard/analytics-card-2-shows-app-overall-accuracy.png)

Le graphique indique les intentions prédites correctement et les zones à problème avec différentes couleurs. À mesure que vous améliorez l’application avec les suggestions, ce score augmente.

Les suggestions de correction sont réparties par type de problème et sont les plus importantes pour votre application. Si vous préférez examiner et résoudre les problèmes par intention, utilisez la carte **[Intention présentant des erreurs de prédiction](#intents-with-errors)** en bas de la page.

Chaque zone à problème comporte des intentions qui doivent être corrigées. Quand vous sélectionnez le nom de l’intention, la page **Intention** s’ouvre avec un filtre appliqué aux énoncés. Ce filtre vous permet de vous concentrer sur les énoncés qui posent problème.

### <a name="compare-changes-across-versions"></a>Comparer les modifications entre les versions

Créez une nouvelle version avant d’apporter des modifications à l’application. Dans la nouvelle version, apportez les modifications suggérées aux exemples d’énoncés de l’intention, puis effectuez un nouvel entraînement. Dans la carte **Évaluation de l’apprentissage** de la page du tableau de bord, utilisez l’option **Afficher le changement par rapport à la version à jour dont l’apprentissage a été effectué** pour comparer les modifications.

![Comparer les modifications entre les versions](./media/luis-how-to-use-dashboard/compare-improvement-across-versions.png)

### <a name="fix-version-by-adding-or-editing-example-utterances-and-retraining"></a>Corriger la version en ajoutant ou en modifiant des exemples d’énoncés et en réentraînant l’application

La principale méthode de correction de votre application consiste à ajouter ou à modifier des exemples d’énoncés et à la réentraîner. Les énoncés nouveaux ou modifiés doivent être conformes aux instructions relatives aux [énoncés variés](luis-concept-utterance.md).

L’ajout d’exemples d’énoncés doit être effectué par une personne qui :

* a un degré élevé de compréhension de ce que sont les énoncés dans les différentes intentions.
* sait comment les énoncés dans une intention peuvent être confondus avec une autre intention.
* est capable de décider si deux intentions qui sont souvent confondues doivent être rassemblées en une intention unique. Dans ce cas, les données différentes doivent être extraites avec les entités.

### <a name="patterns-and-phrase-lists"></a>Modèles et listes d’expressions

La page d’analytique n’indique pas quand utiliser des [modèles](luis-concept-patterns.md) ou des [listes d’expressions](luis-concept-feature.md). Si vous les ajoutez, cela pourra vous aider à corriger les prédictions incorrectes ou vagues, mais n’aura aucun effet sur le déséquilibre des données.

### <a name="review-data-imbalance"></a>Examiner le déséquilibre des données

Commencez par ce problème ; il peut être la cause racine des autres problèmes.

La liste d’intentions à **déséquilibre des données** montre les intentions qui ont besoin de plus d’énoncés pour corriger le déséquilibre des données.

**Pour résoudre ce problème**

* Ajoutez des énoncés à l’intention, puis effectuez un nouvel entraînement.

N’ajoutez pas d’énoncé à l’intention Aucune, sauf si ceci vous est suggéré dans le tableau de bord.

> [!Tip]
> Utilisez la troisième section de la page, **Énoncés par intention**, avec le paramètre **Énoncés (nombre)** , comme guide visuel rapide indiquant les intentions qui ont besoin de davantage d’énoncés.
    ![Utilisez « Énoncés (nombre) » pour rechercher les intentions présentant un déséquilibre des données.](./media/luis-how-to-use-dashboard/predictions-per-intent-number-of-utterances.png)

### <a name="review-incorrect-predictions"></a>Passer en revue les prédictions incorrectes

La liste d’intentions à **prédiction incorrecte** montre les intentions ayant des énoncés qui sont utilisés comme exemples pour une intention spécifique, mais qui sont prédits pour différentes intentions.

**Pour résoudre ce problème**

* Modifiez les énoncés afin qu’ils soient plus propres à l’intention, puis effectuez un nouvel entraînement.
* Combinez les intentions si des énoncés sont trop étroitement alignés, puis effectuez un nouvel entraînement.

### <a name="review-unclear-predictions"></a>Passer en revue les prédictions vagues

La liste d’intentions à **prédiction vague** montre les intentions ayant des énoncés avec des scores de prédiction qui ne sont pas assez loin de leur rival le plus proche, et dont l’intention supérieure de l’énoncé est susceptible de changer lors de l’entraînement suivant en raison d’un [échantillonnage négatif](luis-how-to-train.md#train-with-all-data).

**Pour résoudre ce problème**

* Modifiez les énoncés afin qu’ils soient plus propres à l’intention, puis effectuez un nouvel entraînement.
* Combinez les intentions si des énoncés sont trop étroitement alignés, puis effectuez un nouvel entraînement.

## <a name="utterances-per-intent"></a>Énoncés par intention

Cette carte affiche l’intégrité globale de l’application parmi les intentions. À mesure que vous corrigez les intentions et effectuez de nouveaux entraînements, continuez à garder un œil sur cette carte pour voir si elle mentionne des problèmes.

Le graphique suivant montre une application bien équilibrée avec presque aucun problème à résoudre.

![Le graphique suivant montre une application bien équilibrée avec presque aucun problème à résoudre.](./media/luis-how-to-use-dashboard/utterance-per-intent-shows-data-balance.png)

Le graphique suivant montre une application mal équilibrée avec de nombreux problèmes à résoudre.

![La capture d’écran montre les prédictions par intention avec plusieurs résultats pas clairs ou prédits de manière incorrecte.](./media/luis-how-to-use-dashboard/utterance-per-intent-shows-data-imbalance.png)

Placez le curseur sur la barre de chaque intention pour obtenir des informations sur l’intention.

![La capture d’écran montre les prédictions par intention avec les détails des résultats pas clairs ou prédits de manière incorrecte.](./media/luis-how-to-use-dashboard/utterances-per-intent-with-details-of-errors.png)

Utilisez la fonctionnalité **Trier par** pour organiser les intentions par type de problème et vous concentrer sur les intentions les plus problématiques.

## <a name="intents-with-errors"></a>Intention présentant des erreurs de prédiction

Cette carte vous permet de passer en revue les problèmes pour une intention spécifique. La vue par défaut de cette carte montre les intentions les plus problématiques, afin que vous puissiez concentrer vos efforts là où il le faut.

![La carte Intention présentant des erreurs de prédiction vous permet de passer en revue les problèmes pour une intention spécifique. Cette carte est filtrée par défaut afin d’afficher les intentions les plus problématiques, pour que vous puissiez concentrer vos efforts là où il le faut.](./media/luis-how-to-use-dashboard/most-problematic-intents-with-errors.png)

Le graphique en anneau supérieur affiche les trois types de problème au niveau de l’intention. S’il existe des problèmes des trois types, chaque type a son propre graphique en dessous, avec les éventuelles intentions rivales.

### <a name="filter-intents-by-issue-and-percentage"></a>Filtrer les intentions par problème et par pourcentage

Cette section de la carte vous permet de trouver des exemples d’énoncés situés en dehors de votre seuil d’erreur. Dans l’idéal, vous souhaitez que les prédictions correctes soient significatives. Ce pourcentage est basé sur les clients et l’entreprise.

Déterminez les pourcentages de seuil qui conviennent à votre entreprise.

Le filtre vous permet de trouver les intentions présentant un problème spécifique :

|Filtrer|Pourcentage conseillé|Objectif|
|--|--|--|
|Intentions les plus problématiques|-|**Commencez ici** : la correction des énoncés dans cette intention améliorera l’application davantage que d’autres corrections.|
|Prédictions correctes ci-dessous|60%|Il s’agit du pourcentage d’énoncés dans l’intention sélectionnée qui sont corrects mais qui ont un score de confiance inférieur au seuil. |
|Prédictions vagues ci-dessus|15 %|Il s’agit du pourcentage d’énoncés dans l’intention sélectionnée qui sont confondus avec l’intention rivale la plus proche.|
|Prédictions incorrectes ci-dessus|15 %|Il s’agit du pourcentage d’énoncés dans l’intention sélectionnée qui sont prédits de façon incorrecte. |

### <a name="correct-prediction-threshold"></a>Seuil de prédiction correct

Quel est le score de confiance d’une prédiction correcte, pour vous ? Au début du développement d’application, votre cible peut être de 60 %. Utilisez **Prédictions correctes ci-dessous** avec le pourcentage de 60 % pour trouver les énoncés dans l’intention sélectionnée qui doivent être corrigés.

### <a name="unclear-or-incorrect-prediction-threshold"></a>Seuil de prédiction vague ou incorrect

Ces deux filtres vous permettent de trouver les énoncés dans l’intention sélectionnée qui sont au-delà de votre seuil. Vous pouvez considérer ces deux pourcentages comme des pourcentages d’erreur. Si un taux d’erreur de 10 à 15 % pour les prédictions vous convient, définissez le seuil de filtre à 15 % pour rechercher tous les énoncés au-dessus de cette valeur.

## <a name="next-steps"></a>Étapes suivantes

* [Gérer vos ressources Azure](luis-how-to-azure-subscription.md)
