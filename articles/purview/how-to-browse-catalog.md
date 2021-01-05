---
title: Guide pratique pour parcourir le catalogue de données
description: Cet article explique dans les grandes lignes comment parcourir le catalogue de données Azure Purview en fonction du type de ressource.
author: chanuengg
ms.author: csugunan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 11/30/2020
ms.openlocfilehash: dd2fd4fbc9d8aecd9821d668dd02bd4b68659cf6
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97695064"
---
# <a name="browse-the-azure-purview-data-catalog"></a>Navigation dans le catalogue de données Azure Purview

Cet article explique comment découvrir des données dans un catalogue de données Azure Purview à l’aide de l’espace de noms hiérarchique de la source de données.

## <a name="browse-experience"></a>Expérience de navigation

Un consommateur de données peut rechercher des données avec l’espace de noms hiérarchique familier de chacune des sources de données dans un mode Explorateur. Une fois la source de données inscrite et analysée, le mappage de données extrait des informations sur la structure (espace de noms hiérarchique) de la source de données. Ces informations sont utilisées pour créer l’expérience de navigation de la recherche de données.

Il est par exemple facile de trouver un jeu de données nommé *DateDimension* dans un dossier nommé *Dimensions* dans Azure Data Lake Storage Gen2. Vous pouvez utiliser l’expérience « Parcourir par type de ressource » pour accéder au compte de stockage ADLS Gen2. Naviguez ensuite jusqu’au service > conteneur > dossier(s) pour atteindre le dossier *Dimensions* en question. La table *DateDimension* apparaît alors.

Une expérience de navigation native avec l’espace de noms hiérarchique est proposée pour chacune des sources de données correspondantes.

## <a name="browse-the-data-catalog-by-asset-type"></a>Navigation dans le catalogue de données par type de ressource

1. Pour parcourir les ressources de données, vous pouvez sélectionner **Parcourir par type de ressource** sur la page d’accueil.

    :::image type="content" source="media/how-to-browse-catalog/studio-home-page.png" alt-text="Page d’accueil Purview" border="true":::

1. Sur la page **Parcourir les types de ressources**, les vignettes sont classées en fonction des sources de données. Pour approfondir l’exploration des ressources de chaque source de données, sélectionnez la vignette correspondante.

    :::image type="content" source="media/how-to-browse-catalog/browse-asset-types.jpg" alt-text="Page Parcourir les types de ressources" border="true":::

1. Sur la page suivante figurent les ressources de niveau supérieur sous le type de données choisi. Choisissez l’une d’elles pour explorer son contenu.

    :::image type="content" source="media/how-to-browse-catalog/asset-type-specific-browse.jpg" alt-text="Page de navigation propre au type de ressource, en l’occurrence le compte de stockage Azure" border="true":::

1. Le mode Explorateur s’ouvre. Pour commencer, sélectionnez la ressource dans le volet gauche. Les ressources enfants s’affichent dans le volet droit de la page.

    :::image type="content" source="media/how-to-browse-catalog/explorer-view.jpg" alt-text="Mode Explorateur" border="true":::

1. Pour afficher les détails d’une ressource, sélectionnez le nom ou le bouton ellipse tout à droite.

    :::image type="content" source="media/how-to-browse-catalog/view-asset-detail-click-ellipses.jpg" alt-text="Sélection du bouton ellipse pour afficher la page Détails de la ressource" border="true":::

## <a name="view-related-data-assets"></a>Afficher les ressources de données associées

Une fois sur la page Détails de la ressource, vous pouvez également afficher des ressources de données associées. Par exemple, vous pouvez accéder au dossier parent de *DateDimension* (*Dimensions*) ou même naviguer jusqu’au conteneur pour afficher les ressources qui figurent sous la hiérarchie en question.

1. Sur la page Détails de la ressource, sélectionnez l’onglet **Ressources associées** pour afficher des ressources associées.

    :::image type="content" source="media/how-to-browse-catalog/launch-related-tab.jpg" alt-text="Onglet Ressources associées" border="true":::

1. La hiérarchie complète de la ressource active s’affiche sur la gauche.

    :::image type="content" source="media/how-to-browse-catalog/hierarchical-structure.jpg" alt-text="Structure hiérarchique" border="true":::

1. Sélectionnez un niveau dans la hiérarchie pour voir la liste des ressources qui se trouvent juste au-dessous.

    :::image type="content" source="media/how-to-browse-catalog/select-different-hierarchy.jpg" alt-text="Sélection d’une autre hiérarchie" border="true":::

## <a name="next-steps"></a>Étapes suivantes

- [Guide pratique pour créer, importer et exporter des termes de glossaire](how-to-create-import-export-glossary.md)
- [Guide pratique pour gérer les modèles de termes dans un glossaire métier](how-to-manage-term-templates.md)
