---
title: Gérer le graphique des flux de données de mappage
description: Gérer et modifier efficacement le graphique des flux de données de mappage
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 09/02/2020
ms.openlocfilehash: f55364fe0eabae4bb1698bd655cde8d3e01b127a
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97590458"
---
# <a name="managing-the-mapping-data-flow-graph"></a>Gérer le graphique des flux de données de mappage

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Les flux de données de mappage sont créés à l'aide d'une aire de conception appelée graphique des flux de données. Dans le graphique, la logique de transformation est construite de gauche à droite, et des flux de données supplémentaires sont ajoutés de haut en bas. Pour ajouter une nouvelle transformation, sélectionnez le signe plus (+) situé dans la partie inférieure droite d’une transformation existante.

![Canevas](media/data-flow/canvas-2.png)

À mesure que vos flux de données gagnent en complexité, utilisez les mécanismes suivants pour gérer efficacement le graphique des flux de données et y naviguer. 

## <a name="moving-transformations"></a>Déplacer les transformations

Dans les flux de données de mappage, un ensemble de logiques de transformation connectées s'appelle un **flux**. Le champ **Flux entrant** détermine quel flux de données alimente la transformation en cours. Chaque transformation est associée à un ou deux flux entrants, selon sa fonction, et représente un flux de sortie. Le schéma de sortie des flux entrants détermine quelles métadonnées de colonne peuvent être référencées par la transformation en cours.

![Déplacer le nœud](media/data-flow/move-nodes.png "Déplacer le nœud")

Contrairement aux canevas de pipeline, la modification des transformations de flux de données ne s'effectue pas à l'aide d'un modèle glisser-déposer. Pour modifier le flux entrant ou « déplacer » une transformation, choisissez une autre valeur dans la liste déroulante **Flux entrant**. Toutes les transformations situées en aval se déplacent alors avec la transformation modifiée. Le graphique est automatiquement mis à jour pour afficher le nouveau flux logique. Si vous remplacez le flux entrant par une transformation qui dispose déjà d'une transformation en aval, une nouvelle branche ou un nouveau flux de données parallèle est créé. Découvrez-en plus sur les [nouvelles branches d'un flux de données de mappage](data-flow-new-branch.md).

## <a name="hide-graph-and-show-graph"></a>Masquer le graphique et afficher le graphique

Lorsque vous modifiez votre transformation, vous pouvez développer le panneau de configuration pour qu'il occupe la totalité du canevas, ce qui masque le graphique. Cliquez sur le chevron orienté vers le haut sur le côté droit du canevas.

![Masquer le graphique](media/data-flow/hide-graph.png "Masquer le graphique")

Lorsque le graphique est masqué, vous pouvez vous déplacer entre les transformations d'un flux en cliquant sur **Suivant** ou **Précédent**. Cliquez sur le chevron orienté vers le bas pour afficher le graphique.

![Afficher le graphique](media/data-flow/show-graph.png "Afficher le graphique")

## <a name="searching-for-transformations"></a>Rechercher des transformations

Pour trouver rapidement une transformation dans votre graphique, cliquez sur l'icône **Rechercher** au-dessus du paramètre de zoom.

![Capture d’écran montrant le bouton Rechercher.](media/data-flow/search-1.png "Graphique de recherche")

Pour accéder à une transformation, vous pouvez effectuer une recherche par nom de transformation ou par description.

![Capture d’écran montrant la zone de texte de recherche.](media/data-flow/search-2.png "Graphique de recherche")

## <a name="hide-reference-nodes"></a>Masquer les nœuds de référence

Si votre flux de données comporte des transformations de jointure, de recherche, d'existence ou d'union, le flux de données affiche les nœuds de référence de tous les flux entrants. Si vous souhaitez réduire l'espace vertical occupé, vous pouvez réduire vos nœuds de référence. Pour ce faire, cliquez avec le bouton droit sur le canevas, puis sélectionnez **Masquer les nœuds de référence**.

![Masquer les nœuds de référence](media/data-flow/hide-reference-nodes.png "Masquer les nœuds de référence")

## <a name="next-steps"></a>Étapes suivantes

Une fois votre logique de flux de données terminée, activez le [mode de débogage](concepts-data-flow-debug-mode.md) et testez-la dans un aperçu des données.
