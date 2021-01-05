---
title: Visualisations de textes de classeur Azure Monitor
description: Découvrez toutes les visualisations de textes de classeur Azure Monitor.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 09/04/2020
ms.author: lagayhar
ms.openlocfilehash: e8f2d9495484b781b26962c2946b5bada6c38b4c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89663161"
---
# <a name="text-visualizations"></a>Visualisations de textes

Les classeurs permettent aux auteurs d’inclure des blocs de texte dans leurs classeurs. Le texte peut être une analyse humaine de télémétrie, des informations pour aider les utilisateurs à interpréter vos données, des en-têtes de section, etc.

[![Capture d’écran de la table de texte Apdex](./media/workbooks-text-visualizations/apdex.png)](./media/workbooks-text-visualizations/apdex.png#lightbox)

Le texte est ajouté à l’aide d’un contrôle Markdown qui fournit un contrôle de mise en forme complet. Il s’agit notamment de styles d’en-tête et de police, de liens hypertexte, de tableaux, etc.

Mode d’édition :

![Capture d’écran d’une étape de texte en mode d’édition.](./media/workbooks-text-visualizations/text-edit-mode.png)

Mode Aperçu :

![Capture d’écran d’une étape de texte en mode d’édition dans l’onglet Aperçu.](./media/workbooks-text-visualizations/text-edit-mode-preview.png)

## <a name="add-a-text-control"></a>Ajouter un contrôle de texte

1. Basculez le classeur en mode d’édition en cliquant sur l’élément **Modifier** de la barre d’outils.
2. Utilisez le lien **Ajouter du texte** pour ajouter un contrôle de texte au classeur.
3. Ajoutez Markdown dans le champ éditeur.
4. Utilisez l’option *Style de texte* pour passer du Markdown simple au Markdown enveloppé avec le style standard information/avertissement/réussite/erreur du portail Azure.
5. Utilisez l’onglet **Aperçu** pour voir à quoi ressemblera votre contenu. Lors de la modification, l’aperçu affiche le contenu à l’intérieur d’une zone avec barre de défilement pour limiter sa taille. Toutefois, au moment de l’exécution, le contenu Markdown se développe pour remplir l’espace dont il a besoin, sans barre de défilement.
6. Sélectionnez le bouton **Modification terminée** pour terminer la modification de l’étape.

> [!TIP]
> Utilisez cet [aide-mémoire sur Markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) pour en savoir plus sur les différentes options de mise en forme.

## <a name="text-styles"></a>Styles de texte

Les styles de texte suivants sont disponibles pour l’étape de texte :

| Style     | Explication                                                                               |
|-----------|-------------------------------------------------------------------------------------------|
| `plain`   | Aucune mise en forme supplémentaire n’est appliquée.                                                      |
| `info`    | Style « information » du portail, avec une icône `ℹ` ou similaire et un arrière-plan généralement bleu.      |
| `error`   | Style « erreur » du portail, avec une icône `❌` ou similaire et un arrière-plan généralement rouge.     |
| `success` | Style « réussite » du portail, avec une icône `✔` ou similaire et un arrière-plan généralement vert.  |
| `upsell`  | Style « vente incitative » du portail, avec une icône `🚀` ou similaire et un arrière-plan généralement violet. |
| `warning` | Style « avertissement » du portail, avec une icône `⚠` ou similaire et un arrière-plan généralement bleu.   |

Au lieu de sélectionner un style spécifique, vous pouvez également choisir un paramètre de texte comme source du style. La valeur du paramètre doit être l’une des valeurs de texte ci-dessus. L’absence d’une valeur ou toute valeur non reconnue sera traitée comme style `plain`.

Exemple du style Information :

![Capture d’écran de ce à quoi ressemble le style Information.](./media/workbooks-text-visualizations/text-preview-info-style.png)

Exemple du style Avertissement :

![Capture d’écran de ce à quoi ressemble le style Avertissement.](./media/workbooks-text-visualizations/text-warning-style.png)

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment créer un [graphique dans les classeurs](workbooks-chart-visualizations.md).
* Découvrez comment créer une [grille dans les classeurs](workbooks-grid-visualizations.md).
