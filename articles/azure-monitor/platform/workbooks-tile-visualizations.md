---
title: Visualisations de vignette de classeur Azure Monitor
description: Découvrez toutes les visualisations de vignette de classeur Azure Monitor.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 09/04/2020
ms.author: lagayhar
ms.openlocfilehash: 07de9fcd01559e59a1bcd5d8928075aebcf79cc7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89663165"
---
# <a name="tile-visualizations"></a>Visualisations de vignettes

Les vignettes sont un moyen utile de présenter des données de synthèse dans des classeurs. L’image ci-dessous montre un cas d’usage courant de vignettes présentées avec une synthèse de l’application au-dessus d’une grille détaillée.

[![Capture d’écran de l’affichage en mosaïque du résumé](./media/workbooks-tile-visualizations/tiles-summary.png)](./media/workbooks-tile-visualizations/tiles-summary.png#lightbox)

Les vignettes de classeur prennent en charge un titre, un sous-titre, du texte long, des icônes, des dégradés basés sur les métriques, des lignes/barres spark, des pieds de page, etc.

## <a name="adding-a-tile"></a>Ajout d'une vignette

1. Basculez le classeur en mode d’édition en cliquant sur l’élément _Modifier_ de la barre d’outils.
2. Sélectionnez **Ajouter**, puis *Ajouter une requête* pour ajouter un contrôle de requête de journal au classeur.
3. Sélectionnez le type de requête **Journal**, le type de ressource (par exemple, Application Insights) et les ressources à cibler.
4. Utilisez l’éditeur de requête pour saisir le KQL pour votre analyse.

    ```kusto
    requests
    | summarize Requests = count() by appName, name
    | top 7 by Requests desc
    ```

5. Définissez la taille sur **Maximale**.
6. Définissez la visualisation sur **Vignettes**.
7. Sélectionnez le bouton **Paramètres de vignette** pour ouvrir le volet des paramètres.
    1. Dans *Titre*, définissez :
        * Utiliser la colonne : `name`.
    2. Dans *Gauche*, définissez :
        * Utiliser la colonne : `Requests`.
        * Convertisseur de colonne : `Big Number`.
        * Palette de couleurs : `Green to Red`
        * Valeur minimale : `0`.
    3. Dans *Bas*, définissez :
        * Utiliser la colonne : `appName`.
8. Sélectionnez le bouton **Enregistrer et fermer** au bas du volet.

[![Capture d’écran de l’affichage de la synthèse des vignettes, avec au-dessus la requête et les paramètres de vignette.](./media/workbooks-tile-visualizations/tile-settings.png)](./media/workbooks-tile-visualizations/tile-settings.png#lightbox)

Les vignettes en mode lecture :

[![Capture d’écran de l’affichage de la synthèse des vignettes en mode lecture.](./media/workbooks-tile-visualizations/tiles-read-mode.png)](./media/workbooks-tile-visualizations/tiles-read-mode.png#lightbox)

## <a name="spark-lines-in-tiles"></a>Graphiques sparkline dans les vignettes

1. Basculez le classeur en mode d’édition en cliquant sur l’élément _Modifier_ de la barre d’outils.
2. Ajoutez un paramètre d’intervalle de temps nommé `TimeRange`.
    1. Sélectionnez **Ajouter**, puis *Ajouter des paramètres*.
    2. Dans le contrôle des paramètres, sélectionnez **Ajouter un paramètre**.
    3. Entrez `TimeRange` dans le champ *Nom du paramètre* et choisissez `Time range picker` pour le *Type de paramètre*.
    4. Sélectionnez **Enregistrer** en haut du volet, puis sélectionnez **Modification effectuée** dans le contrôle des paramètres.
3. Sélectionnez **Ajouter**, puis *Ajouter une requête* pour ajouter un contrôle de requête de journal sous le contrôle des paramètres.
4. Sélectionnez le type de requête **Journal**, le type de ressource (par exemple, Application Insights) et les ressources à cibler.
5. Utilisez l’éditeur de requête pour saisir le KQL pour votre analyse.

    ```kusto
    let topRequests = requests
    | summarize Requests = count() by appName, name
    | top 7 by Requests desc;
    let topRequestNames = topRequests | project name;
    requests
    | where name in (topRequestNames)
    | make-series Trend = count() default = 0 on timestamp from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by name
    | join (topRequests) on name
    | project-away name1, timestamp
    ```

6. Sélectionnez **Run Query** (Exécuter la requête). (Veillez à définir `TimeRange` sur une valeur de votre choix avant d’exécuter la requête.)
7. Définissez la *Visualisation* sur « Vignettes ».
8. Définissez la *Taille* sur « Maximale ».
9. Sélectionnez **Paramètres de vignette**.
    1. Dans *Vignette*, définissez :
        * Utiliser la colonne : `name`.
    2. Dans *Vignette secondaire*, définissez :
        *  Utiliser la colonne : `appNAme`.
    3. Dans *Gauche*, définissez :
        *  Utiliser la colonne : `Requests`.
        * Convertisseur de colonne : `Big Number`.
        * Palette de couleurs : `Green to Red`.
        * Valeur minimale : `0`.
    4. Dans *Bas*, définissez :
        * Utiliser la colonne : `Trend`.
        * Convertisseur de colonne : `Spark line`.
        * Palette de couleurs : `Green to Red`.
        * Valeur minimale : `0`.
10. Sélectionnez **Enregistrer et fermer** au bas du volet.

![Capture d’écran de la visualisation des vignettes avec un graphique sparkline](./media/workbooks-tile-visualizations/spark-line.png)

## <a name="tile-sizes"></a>Tailles des vignettes

L’auteur a la possibilité de définir la largeur des vignettes dans les paramètres de vignette.

* `fixed` (valeur par défaut)

    Le comportement par défaut des vignettes est d’adopter la même largeur fixe, environ 160 pixels de large, à laquelle s’ajoute l’espace autour des vignettes.

    ![Capture d’écran affichant les vignettes à largeur fixe](./media/workbooks-tile-visualizations/tiles-fixed.png)
* `auto`

    Chaque titre est réduit ou augmenté pour s’ajuster à son contenu, en revanche les vignettes sont limitées à leur largeur d’affichage (pas de défilement horizontal).

    ![Capture d’écran montrant des vignettes à largeur ajustable automatiquement](./media/workbooks-tile-visualizations/tiles-auto.png)
* `full size`

    Chaque titre occupe toujours la largeur maximale de l’affichage des vignettes, à raison d’un titre par ligne.

     ![Capture d’écran montrant les vignettes à largeur maximale](./media/workbooks-tile-visualizations/tiles-full.png)

## <a name="next-steps"></a>Étapes suivantes

* Les vignettes prennent également en charge le convertisseur de barre composite. Pour en savoir plus, consultez la [Documentation sur la barre composite](workbooks-composite-bar.md).
* Pour en apprendre davantage sur les paramètres de temps tels que `TimeRange`, consultez la [Documentation sur les paramètres de temps des classeurs](workbooks-time.md).