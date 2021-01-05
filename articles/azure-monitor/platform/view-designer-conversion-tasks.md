---
title: Concepteur de vues Azure Monitor - Tâches courantes liées à la conversion de vues en classeurs
description: ''
author: austonli
ms.author: aul
ms.subservice: ''
ms.topic: conceptual
ms.date: 02/07/2020
ms.openlocfilehash: 5559dac916262998d621b40757398088ec613609
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77658742"
---
# <a name="view-designer-to-workbooks-conversion-common-tasks"></a>Concepteur de vues - Tâches courantes liées à la conversion de vues en classeurs
Le [concepteur de vues](view-designer.md) est une fonctionnalité d'Azure Monitor qui vous permet de créer des vues personnalisées pour vous aider à visualiser les données de votre espace de travail Log Analytics, avec des graphiques, des listes et des chronologies. Celles-ci sont progressivement supprimées et remplacées par des classeurs qui offrent des fonctionnalités supplémentaires. Cet article détaille les tâches courantes liées à la conversion de vues en classeurs.


## <a name="quickstart-with-preset-view-designer-templates"></a>Démarrage rapide avec les modèles prédéfinis du concepteur de vues

Les classeurs des espaces de travail Log Analytics disposent déjà de modèles compatibles avec certaines des vues du concepteur de vues. Dans la catégorie **Guides du concepteur de vues**, sélectionnez **Guide de transition du concepteur de vues** pour en savoir plus sur les options disponibles, ou sélectionnez l'un des modèles prédéfinis.

![Exemples de modèles](media/view-designer-conversion-tasks/templates.png)

## <a name="enabling-time-range-filter"></a>Activer un filtre d'intervalle de temps
Un filtre d'intervalle de temps par défaut est intégré au concepteur de vues, mais dans les classeurs, ce paramètre n'est pas activé par défaut. Les classeurs permettent aux utilisateurs de créer leurs propres filtres d'intervalle de temps pour les appliquer à leurs journaux de données. Les étapes à suivre pour générer le filtre sont répertoriées ci-dessous :

Sélectionnez l'option **Ajouter des paramètres**. Le **Style** par défaut est défini sur *Pilules*.

![Ajouter un paramètre](media/view-designer-conversion-tasks/add-param.png)

 Sélectionnez le bouton **Ajouter un paramètre**.

![Ajouter un paramètre](media/view-designer-conversion-tasks/add-parameter.png)

Dans le menu de la barre latérale, accédez à la zone de texte **Nom du paramètre** et entrez *TimeRange*. Définissez **Type de paramètre** sur *Sélecteur d'intervalle de temps*. Cochez la case **Obligatoire ?** .

![Menu Paramètres](media/view-designer-conversion-tasks/parameter-menu.png)

Enregistrez le paramètre dans le coin supérieur gauche du menu de la barre latérale. Dans la liste déroulante, vous pouvez conserver le paramètre *Non défini* par défaut ou sélectionner une valeur **TimeRange** par défaut, telle que *24 heures*. Sélectionnez **Modification terminée**.

Les paramètres peuvent être utilisés dans les requêtes en ajoutant des accolades {} autour du nom de votre paramètre. Vous trouverez plus de détails sur les paramètres dans la [section Paramètres de la documentation consacrée aux classeurs](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Documentation/Parameters/Parameters.md).

## <a name="updating-queries-with-the-timerange-parameter"></a>Mettre à jour des requêtes avec le paramètre TimeRange

### <a name="option-1-select-timerange-from-the-time-range-dropdown"></a>Option 1 : sélectionner TimeRange dans la liste déroulante Intervalle de temps.

![Paramètre de temps](media/view-designer-conversion-tasks/time-parameter.png)

### <a name="option-2-update-your-log-queries"></a>Option n°2 : mettre à jour les requêtes de journal

Dans votre requête, ajoutez la ligne : `| where TimeGenerated {TimeRange}` comme dans l'exemple suivant :

Requête d’origine
```KQL
search * 
| summarize count() by Type
```

Requête mise à jour
```KQL
search * 
| where TimeGenerated {TimeRange} 
| summarize count() by Type
```

## <a name="including-a-list"></a>Inclure une liste
La plupart des vues du concepteur de vues incluent une liste, et vous pouvez reproduire cette liste standard dans un classeur.

![Liste de vignettes](media/view-designer-conversion-tasks/tile-list.png)

Ajoutez une visualisation en cliquant sur **Ajouter une requête** dans les options de la cellule.

![Ajouter un paramètre](media/view-designer-conversion-tasks/add-param.png)

Le concepteur de vues utilise une requête par défaut qui correspond à la syntaxe de l'exemple d'origine. Celle-ci peut être mise à jour en remplaçant la requête par le formulaire mis à jour comme dans l'exemple suivant :

Requête d’origine
```KQL
search * 
| summarize AggregatedValue = count() by Type
```

Requête mise à jour
```KQL
search * 
| summarize Count = count() by Type
```

Une liste semblable à la suivante est générée :

![Exemple de liste](media/view-designer-conversion-tasks/list-example.png)

## <a name="enabling-sparklines"></a>Activer des graphiques sparkline
Une caractéristique commune aux grilles consiste à ajouter des graphiques sparkline pour résumer divers modèles de données au fil du temps. Le concepteur de vues propose la fonctionnalité **Activer des graphiques sparkline** pour toutes les listes, à l'instar des classeurs. Pour inclure dans vos données des graphiques sparkline qui correspondent au concepteur de vues, joignez les données à votre requête d'origine comme dans l'exemple suivant :

Requête d’origine
```KQL
search *
| summarize AggregatedValue = count() by Type) on Type
```

Requête mise à jour
```KQL
search * 
| summarize AggregatedValue = count() by Type
| join kind = inner (search * 
    | make-series Trend = count() default=0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by Type) on Type
| project Type, AggregatedValue, Trend
```

Sélectionnez **Paramètres de colonne**.
![Paramètres de colonne](media/view-designer-conversion-tasks/column-settings.png)

Mettez à jour la liste déroulante **Convertisseur de colonne** pour la transformer en *Zone Spark*.
![Sparklines](media/view-designer-conversion-tasks/sparkline.png)

Enregistrez les paramètres et réexécutez la requête afin de mettre à jour votre tableau pour y inclure un graphique sparkline.

La grille obtenue est semblable à la suivante : ![Exemple de graphique sparkline](media/view-designer-conversion-tasks/sparkline-example.png)

## <a name="advanced-cell-settings"></a>Paramètres de cellule avancés
Pour mettre en miroir le concepteur de vues, vous pouvez effectuer des tâches telles que la modification de la taille des cellules du classeur ou l'ajout d'épingles et de liens externes aux journaux.

Pour accéder à **Paramètres avancés**, sélectionnez l'icône en forme d'engrenage en bas de chaque cellule.

![Paramètres avancés](media/view-designer-conversion-tasks/advanced-settings.png)

Un menu répertoriant diverses options s'affiche :

![Paramètres avancés](media/view-designer-conversion-tasks/advanced-settings-settings.png)

Pour ajouter une épingle et un lien à une requête externe, cochez les cases correspondantes. Pour ajouter un titre à votre cellule, entrez le titre souhaité dans la section **Titre du graphique**.

Par défaut, chaque cellule du classeur est définie pour occuper toute la largeur de la page, mais vous pouvez réduire la cellule sous l'onglet **Style** du menu **Paramètres avancés**

![Style des paramètres avancés](media/view-designer-conversion-tasks/advanced-settings-style.png)

 
## <a name="additional-parameters"></a>Paramètres supplémentaires
Sélectionnez **Ajouter un paramètre** pour créer un nouveau paramètre dans votre classeur. 

Pour sélectionner un abonnement, entrez *Abonnement* dans le champ **Nom du paramètre** du menu latéral et sélectionnez *Sélecteur d'abonnement* dans la liste déroulante **Type de paramètre**.

![Menu Abonnement](media/view-designer-conversion-tasks/subscription-filter.png)

Pour sélectionner une ressource, entrez *Ressource* dans le champ **Nom du paramètre** du menu latéral et sélectionnez *Sélecteur de ressources* dans la liste déroulante **Type de paramètre**.

![Menu Ressources](media/view-designer-conversion-tasks/resource-filter.png)

Des listes déroulantes sont alors insérées pour vous permettre d'accéder à vos différents abonnements et ressources.

![Liste déroulante des abonnements et ressources](media/view-designer-conversion-tasks/subscription-resource.png)


## <a name="next-steps"></a>Étapes suivantes
- [Conversions des vignettes](view-designer-conversion-tiles.md)
