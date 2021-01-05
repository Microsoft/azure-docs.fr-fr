---
title: Visualiser les données dans l’Explorateur Time Series Insights – Azure Time Series Insights Gen2 | Microsoft Docs
description: Découvrez les fonctionnalités et options disponibles dans l’Explorateur Azure Time Series Insights.
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 10/01/2020
ms.custom: seodec18
ms.openlocfilehash: edfbbdd1b30cba74cf4f63d6b5e84c2e616e1a0f
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95020978"
---
# <a name="azure-time-series-insights-explorer"></a>Explorateur Azure Time Series Insights

Cet article décrit les diverses fonctionnalités et options disponibles dans l’[Environnement de démonstration](https://insights.timeseries.azure.com/preview/demo) d’Azure Time Series Insights Gen2.

## <a name="prerequisites"></a>Prérequis

Pour commencer à utiliser l’Explorateur Azure Time Series Insights, vous devez :

* Avoir un environnement Azure Time Series Insights Gen2 approvisionné. Pour en savoir plus sur l’approvisionnement d’une instance, lisez le tutoriel [Azure Time Series Insights Gen2](./tutorials-set-up-tsi-environment.md).
* [Fournissez l’accès aux données](./concepts-access-policies.md) à l’environnement Azure Time Series Insights Gen2 que vous avez créé pour le compte. Vous pouvez octroyer l’accès à d’autres personnes ainsi qu’à vous-même.
* Ajoutez une source d’événement à l’environnement Azure Time Series Insights Gen2 pour envoyer (push) des données à ce dernier :
  * Découvrez [comment vous connecter à un Event Hub](./how-to-ingest-data-event-hub.md).
  * Découvrez [comment vous connecter à un Hub IoT](./how-to-ingest-data-iot-hub.md).

## <a name="explore-the-azure-time-series-insights-explorer"></a>Découvrir l’Explorateur Azure Time Series Insights

L’Explorateur Azure Time Series Insights est composé des sept éléments suivants :

[![Vue d’ensemble de l’Explorateur Azure Time Series Insights](media/v2-update-explorer/preview-explorer-overview.png)](media/v2-update-explorer/preview-explorer-overview.png#lightbox)

1. [Panneau Environment](#1-environment-panel) : Affiche tous vos environnements Azure Time Series Insights Gen2.
1. [Barre de navigation](#2-navigation-bar) : Vous permet de basculer entre les pages **Analyser** et **Modèle**.
1. [Arborescence hiérarchique et volet de recherche](#3-hierarchy-tree-and-search-panel) : Vous permet de sélectionner et de rechercher des éléments de données spécifiques à convertir en graphique.
1. [Zone de séries chronologiques](#4-time-series-well) : Affiche tous vos éléments de données actuellement sélectionnés.
1. [Panneau Graphique](#5-chart-panel) : affiche votre graphique actif.
1. [Chronologie](#6-time-editor-panel) : vous permet de modifier votre intervalle de temps actif.
1. [Barre de l’application](#7-app-bar) : Contient vos options de gestion d’utilisateur, comme le locataire actuel, et vous permet de les modifier, ainsi que les paramètres de langage.

## <a name="1-environment-panel"></a>1. Volet Environment

Le volet Environnement affiche tous les environnements Azure Time Series Insights Gen2 auxquels vous avez accès. La liste comprend les environnements Gen2 et Gen1. Sélectionnez simplement l’environnement que vous voulez utiliser pour y être dirigé immédiatement.

1. Sélectionnez la flèche déroulante en regard de votre environnement affiché.

   [![Volet Environment](media/v2-update-explorer/environment-panel.png)](media/v2-update-explorer/environment-panel.png#lightbox)

1. Ensuite, sélectionnez l’environnement souhaité.

## <a name="2-navigation-bar"></a>2. Barre de navigation

  [![Barre de navigation](media/v2-update-explorer/tsi-preview-navigation-bar.png)](media/v2-update-explorer/tsi-preview-navigation-bar.png#lightbox)

Utilisez la barre de navigation pour naviguer entre deux vues :

* **Analyser** : permet de créer des graphiques et d’effectuer des analytiques enrichies de vos données de série chronologique basées ou non sur un modèle.
* **Modèle** : permet d’envoyer les nouveaux types, hiérarchies et instances d’Azure Time Series Insights Gen2 à votre modèle de série chronologique.

### <a name="model-authoring"></a>Création de modèle

La préversion d’Azure Time Series Insights Gen2 prend en charge la totalité des opérations de création, de lecture, de mise à jour et de suppression (CRUD) sur votre modèle de série chronologique.

[![Le panneau de recherche de modèle](media/v2-update-explorer/model-search-panel.png)](media/v2-update-explorer/model-search-panel.png#lightbox)

* **Type de modèle de série chronologique** : les types du modèle de série chronologique vous permettent de définir des variables ou des formules pour l’exécution de calculs. Ils sont associés à une instance spécifique du modèle de série chronologique. Un type peut avoir une ou plusieurs variables.
* **Hiérarchies de modèle de série temporelle** : les hiérarchies sont des organisations systématiques de vos données. Elles décrivent les relations entre les différentes instances dans votre modèle de série chronologique.
* **Instance de modèle de série chronologique** : les instances représentent la série chronologique elle-même. Dans la plupart des cas, ils s’agit de **DeviceID** ou d’**AssetID**, qui est l’identificateur unique de la ressource dans l’environnement.

Pour plus d’informations le modèle Time Series, consultez [Modèles Time Series](./concepts-model-overview.md).

## <a name="3-hierarchy-tree-and-search-panel"></a>3. Arborescence hiérarchique et volet de recherche

L’arborescence hiérarchique et le volet de recherche vous permettent de rechercher et de parcourir facilement votre hiérarchie de [modèles Time Series](./concepts-model-overview.md) pour rechercher les instances de séries chronologiques spécifiques que vous voulez afficher sur votre graphique. Lorsque vous sélectionnez vos instances, elles sont ajoutées au graphique actuel et aussi aux données.

[![Arborescence hiérarchique et volet de recherche](media/v2-update-explorer/tsi-preview-explorer-hierarchy-search.png)](media/v2-update-explorer/tsi-preview-explorer-hierarchy-search.png#lightbox)

Le volet des résultats de la recherche vous permet également d’afficher vos résultats dans une vue hiérarchique ou une vue liste, ce qui facilite la recherche des instances que vous souhaitez afficher.

## <a name="4-time-series-well"></a>4. Puits de série chronologique

Le puits affiche les champs d’instance et d’autres métadonnées associées aux instances du modèle de série chronologique. Les cases à cocher à droite vous permettent de masquer ou d’afficher des instances spécifiques du graphique actuel.

  [![Le puits Gen2](media/v2-update-explorer/tsi-preview-ui-explorer-well.png)](media/v2-update-explorer/tsi-preview-ui-explorer-well.png#lightbox)

Vous pouvez supprimer des éléments de données spécifiques à partir de vos données en cours en sélectionnant le contrôle **Supprimer** (corbeille) affiché en rouge à gauche de l’élément. La source vous permet également de contrôler la façon dont chaque élément est affiché dans le graphique. Vous pouvez choisir d’ajouter des ombres min./max., des points de données, de déplacer l’élément dans le temps et de visualiser l’instance de manière échelonnée.

En outre, le contrôle Explorations vous permet de créer facilement des décalages temporels et des nuages de points.  

  [![Options de disposition de la source](media/v2-update-explorer/well-layout-options.png)](media/v2-update-explorer/well-layout-options.png#lightbox)

> [!NOTE]
> Si le message suivant apparaît, c’est que l’instance n’a pas de données pour l’intervalle de temps sélectionné. Pour résoudre ce problème, augmentez l’intervalle de temps ou vérifiez que l’instance envoie des données.
>
> ![Aucune notification de données](media/v2-update-explorer/tsi-preview-no-data-warning.png)

## <a name="5-chart-panel"></a>5. Volet Graphique

Le graphique vous permet d’afficher les instances de séries chronologiques sous forme de lignes. Vous pouvez réduire le panneau d’environnement, le modèle de données et le panneau de contrôle des intervalles de temps en cliquant sur les contrôles web afin d’agrandir le graphique.

  [![Vue d’ensemble du graphique Gen2](media/v2-update-explorer/tsi-preview-chart-overview.png)](media/v2-update-explorer/tsi-preview-chart-overview.png#lightbox)

1. **Type de graphique** : Contrôle quels éléments de données sont disponibles pour la visualisation.

1. **Taille de l’intervalle** : L’outil curseur Taille de l’intervalle vous permet d’effectuer un zoom avant/arrière sur les intervalles pour la même plage de dates. Ceci fournit un contrôle plus précis du déplacement entre des grandes tranches de temps qui montrent des tendances lissées pour des tranches se réduisant jusqu’à la milliseconde, ce qui vous permet de passer en revue des segments précis de vos donnés avec une haute résolution. Le point de départ par défaut du curseur est défini comme la vue optimale des données à partir de votre sélection ; résolution de l’équilibrage, vitesse de requête et granularité.

1. **Zoom et panoramique** : Sélectionnez ce contrôle pour effectuer un zoom et un panoramique sur le graphique.

1. **Contrôle de l’axe Y** : parcourt les options d’affichage de l’axe Y disponibles :

    * `Stacked`: Chaque ligne comporte un axe y individuel.
    * `Overlap`: permet d’empiler plusieurs lignes sur le même axe Y, celui-ci étant modifié en fonction de la ligne sélectionnée.
    * `Shared`: Toutes les données de l’axe y sont affichées ensemble.

1. **Élément de marqueur** : élément de données actuellement sélectionné et détails associés.

Vous pouvez affiner une tranche de données spécifique en **cliquant** sur un point de données sur le graphique actuel tout en maintenant le bouton de la souris enfoncé, puis en faisant glisser la zone sélectionnée vers l’extrémité de votre choix. **Cliquez avec le bouton droit** sur la zone bleue sélectionnée, puis sélectionnez **Zoomer** comme indiqué ci-dessous. Vous pouvez également afficher et télécharger les événements de télémétrie pour la période sélectionnée.

  [![Zoom sur graphique Gen2](media/v2-update-explorer/preview-chart-zoom.png)](media/v2-update-explorer/preview-chart-zoom.png#lightbox)

Une fois que vous avez effectué l’action **Zoomer**, votre jeu de données sélectionné est affiché. Sélectionnez le contrôle de format pour parcourir les trois représentations de l’axe Y de vos données.

  [![Axe y de graphique Gen2](media/v2-update-explorer/tsi-preview-explorer-standard-chart.png)](media/v2-update-explorer/tsi-preview-explorer-standard-chart.png#lightbox)

Un exemple de **graphiques superposés** est donné ici :

  [![Option de graphiques superposés](media/v2-update-explorer/tsi-preview-explorer-overlapping-chart.png)](media/v2-update-explorer/tsi-preview-explorer-overlapping-chart.png#lightbox)

Le bouton **Plus d’actions** se développe pour montrer les options **Télécharger en tant que CSV**, **Se connecter à Power BI**, **Montrer les données de graphique sous forme de table**, et **Explorer les événements bruts**.

  [![Option Plus d’actions](media/v2-update-explorer/more-actions-icon.png)](media/v2-update-explorer/more-actions-icon.png#lightbox)

Apprenez-en davantage sur l’option **Se connecter à Power BI** dans [Connecteur Power BI](concepts-power-bi.md).

## <a name="6-time-editor-panel"></a>6. Panneau de l’éditeur d’heure

Lorsque vous utilisez Azure Time Series Insights Gen2, vous commencez par sélectionner un intervalle de temps. L’intervalle de temps sélectionné détermine le jeu de données disponible pour la manipulation avec les widgets de mise à jour d’Azure Time Series Insights Gen2.

  [![Volet de sélection d’heure](media/v2-update-explorer/tsi-preview-explorer-timeline-element.png)](media/v2-update-explorer/tsi-preview-explorer-timeline-element.png#lightbox)

> [!TIP]
> Une partie de la chronologie est mise en surbrillance avec la couleur orange ou ambre pour indiquer l’étendue des données disponibles dans le magasin à chaud.

Les contrôles web suivants sont disponibles dans Azure Time Series Insights Gen2 pour la sélection de votre intervalle de temps actif.

  [![Contrôle Source d’exploration](media/v2-update-explorer/exploration-well-control.png)](media/v2-update-explorer/exploration-well-control.png#lightbox)

1. **Contrôle de curseur de plage de dates interne** : utilisez les deux contrôles de point de terminaison en les faisant glisser sur l’intervalle de temps souhaité. Cette plage de dates interne est limitée par le contrôle de curseur de plage de dates externe.

1. **Boutons pour augmenter ou diminuer la plage de dates** : augmentez ou réduisez votre intervalle de temps en sélectionnant l’un des boutons pour l’intervalle souhaité.

1. **Contrôle de réduction de l’intervalle de temps** : ce contrôle web vous permet de masquer tous les contrôles à l’exception de l’outil de curseur de plage de dates interne.

1. **Contrôle de curseur de plage de dates externe** : utilisez les contrôles de point de terminaison pour sélectionner la plage de dates externe, qui sera disponible pour votre contrôle de plage de dates interne.

1. **Contrôle de curseur d’intervalle de temps** : permet de basculer rapidement entre des intervalles de temps prédéfinis, tels que les **30 dernières minutes**, les **12 dernières heures** ou une **plage personnalisée**. La modification de cette valeur a pour effet de changer également les plages d’intervalles disponibles présentées dans l’outil de curseur de taille d’intervalle.

   [![Volet de sélection de début et de fin](media/v2-update-explorer/to-and-from-element.png)](media/v2-update-explorer/to-and-from-element.png#lightbox)

## <a name="7-app-bar"></a>7. Barre de l’application

Le volet de navigation d’Azure Time Series Insights Gen2 apparaît en haut de votre application. Il offre les fonctionnalités suivantes :

### <a name="current-session-share-link-control"></a>Contrôle de lien de partage de session en cours

  [![Icône Partager](media/v2-update-explorer/tsi-preview-explorer-share-icon.png)](media/v2-update-explorer/tsi-preview-explorer-share-icon.png#lightbox)

Sélectionnez la nouvelle icône **Partager** pour partager une URL de lien avec votre équipe.

  [![Partager l’URL de votre instance](media/v2-update-explorer/tsi-preview-explorer-share-your-view.png)](media/v2-update-explorer/tsi-preview-explorer-share-your-view.png#lightbox)

### <a name="tenant-section"></a>Section de locataire

  [![Sélection de locataire](media/v2-update-explorer/tsi-preview-explorer-tenant-selection.png)](media/v2-update-explorer/tsi-preview-explorer-tenant-selection.png#lightbox)

* Affiche les informations actuelles de votre compte de connexion Azure Time Series Insights Gen2.
* Utilisez-les pour basculer entre les thèmes disponibles.
* Utilisez-les pour afficher l’[environnement de démonstration](https://insights.timeseries.azure.com/preview/demo) Gen2.

### <a name="theme-selection"></a>Sélection de thème

Pour choisir un nouveau thème, sélectionnez l’icône de votre profil située dans l’angle supérieur droit. Ensuite, sélectionnez **Modifier le thème**.

  [![Sélection de thème](media/v2-update-explorer/tsi-preview-theme-selection.png)](media/v2-update-explorer/tsi-preview-theme-selection.png#lightbox)

> [!TIP]
> Vous pouvez également sélectionner la langue en sélectionnant l’icône de votre profil.

L’Explorateur Azure Time Series Insights prend en charge deux thèmes :

* **Thème clair** : thème par défaut affiché dans ce document.
* **Thème foncé** : Affiche l’Explorateur comme illustré ici :

  [![Thème sombre sélectionné](media/v2-update-explorer/tsi-preview-dark-theme-selected.png)](media/v2-update-explorer/tsi-preview-dark-theme-selected.png#lightbox)

## <a name="gen1-environment-controls"></a>Contrôles d’environnement Gen1

### <a name="gen2-terms-panel"></a>Volet des conditions Gen2

Cette section s’applique uniquement aux environnements Gen1 existants qui tentent d’utiliser l’explorateur dans l’interface utilisateur mise à jour. Vous souhaiterez peut-être utiliser le produit Gen1 et le produit Gen2 combinés. Nous avons ajouté certaines fonctionnalités de l’interface utilisateur existante à l’explorateur mis à jour, mais vous pouvez bénéficier de l’expérience d’interface utilisateur complète pour un environnement Gen1 dans le nouvel Explorateur Azure Time Series Insights.

Au lieu de la hiérarchie, le volet des conditions d’Azure Time Series Insights Gen2 s’affiche. Le panneau des termes vous permet de définir des requêtes dans votre environnement. Utilisez-le pour filtrer vos données en fonction d’un prédicat.

  [![Volet de requête Where](media/v2-update-explorer/s1-s2-preview-query.png)](media/v2-update-explorer/s1-s2-preview-query.png#lightbox)

Le volet de l’éditeur des conditions d’Azure Time Series Insights Gen2 accepte les paramètres suivants :

**Where** : la clause Where vous permet de filtrer rapidement vos événements à l’aide de l’ensemble d’opérandes répertoriées dans le tableau ci-dessous. Si vous effectuez une recherche en sélectionnant un opérande, le prédicat est automatiquement mis à jour selon cette recherche. Les types d’opérandes pris en charge comprennent les suivants :

| Opération    | Types pris en charge    | Notes |
| --- | --- | --- |
| `<`, `>`, `<=`, `>=` | Double, DateTime, TimeSpan | |
| `=`, `!=`, `<>` | Chaîne, Bool, Double, DateTime, TimeSpan, NULL |
| `IN` | Chaîne, Bool, Double, DateTime, TimeSpan, NULL | Tous les opérandes doivent être du même type ou être la constante NULL. |
| `HAS` | String | Seuls les littéraux de chaîne constante sont autorisés à droite. Les chaînes vides et NULL ne sont pas autorisées. |

Pour plus d’informations sur les types d’opérations et de données de requête pris en charge, consultez [Time Series Expression (TSX)](/rest/api/time-series-insights/reference-time-series-expression-syntax).

### <a name="examples-of-where-clauses"></a>Exemples de clauses Where

  [![Exemples de clauses Where](media/v2-update-explorer/tsi-preview-example-queries.png)](media/v2-update-explorer/tsi-preview-example-queries.png#lightbox)

**Mesure** : liste déroulante affichant toutes les colonnes numériques (**Doubles**) que vous pouvez utiliser en tant qu’éléments pour votre graphique actuel.

**Diviser par** : cette liste déroulante affiche toutes les colonnes de catégories disponibles (chaînes) dans votre modèle pour le regroupement de vos données. Vous pouvez ajouter jusqu’à cinq conditions à afficher sur le même axe X. Entrez les paramètres souhaités, puis sélectionnez **Ajouter** pour ajouter un terme.

  [![Vue de requête et de filtre n°1](media/v2-update-explorer/s1-s2-preview-filtered-view.png)](media/v2-update-explorer/s1-s2-preview-filtered-view.png#lightbox)

Vous pouvez afficher et masquer des éléments dans le panneau de graphique en sélectionnant l’icône visible, comme illustré dans l’image suivante. Pour supprimer complètement les requêtes, sélectionnez la croix rouge **X**.

  [![Option Annuler une vue de requête et de filtre](media/v2-update-explorer/s1-s2-preview-filtered-view-cancel.png)](media/v2-update-explorer/s1-s2-preview-filtered-view-cancel.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur la [réception de données](./concepts-ingestion-overview.md) dans votre environnement.

* Consultez l’article sur le [stockage](concepts-storage.md).

* Découvrez la [modélisation des données](./concepts-model-overview.md) dans Azure Time Series Insights Gen2.

* Découvrez [comment diagnostiquer et dépanner](./how-to-diagnose-troubleshoot.md) votre environnement.