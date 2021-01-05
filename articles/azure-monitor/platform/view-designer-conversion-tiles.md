---
title: Concepteur de vues Azure Monitor pour les conversions de vignettes de classeurs
description: ''
author: austonli
ms.author: aul
ms.subservice: ''
ms.topic: conceptual
ms.date: 02/07/2020
ms.openlocfilehash: f07d15521c787dfd588c285bff57616059caa2f3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77658624"
---
# <a name="azure-monitor-view-designer-tile-conversions"></a>Conversions de vignettes du Concepteur de vues Azure Monitor
Le [Concepteur de vues](view-designer.md) est une fonctionnalité d’Azure Monitor qui vous permet de créer des vues personnalisées pour vous aider à visualiser les données de votre espace de travail Log Analytics, avec des graphiques, des listes et des chronologies. Ces vues sont progressivement supprimées et remplacées par des classeurs qui offrent des fonctionnalités supplémentaires. Cet article fournit des détails sur la conversion de différentes vignettes en classeurs.

## <a name="donut--list-tile"></a>Vignette Anneau et liste

![Anneau et liste](media/view-designer-conversion-tiles/donut-list.png)

La recréation de la vignette Anneau et liste dans des classeurs implique deux visualisations distinctes. Pour la partie en anneau, il existe deux options.
Pour les deux, commencez par sélectionner **Ajouter une requête**, puis collez la requête d’origine à partir du Concepteur de vues dans la cellule.

**Option 1 :** Sélectionnez **Graphique en secteurs** dans la liste déroulante **Visualisation** : ![Menu de visualisations de graphique en secteurs](media/view-designer-conversion-tiles/pie-chart.png)

**Option 2 :** Sélectionnez **Définir par requête** dans la liste déroulante **Visualisation**, puis ajoutez `| render piechart` à la requête :

 ![Menu Visualisation](media/view-designer-conversion-tiles/set-by-query.png)

**Exemple**

Requête d’origine
```KQL
search * 
| summarize AggregatedValue = count() by Type 
| order by AggregatedValue desc
```

Requête mise à jour
```KQL
search * 
| summarize AggregatedValue = count() by Type 
| order by AggregatedValue desc 
| render piechart
```

Pour créer une liste et activer les graphiques sparkline, voir l’article sur les [tâches courantes](view-designer-conversion-tasks.md).

Voici un exemple illustrant la manière dont la vignette Anneau et liste peut être réinterprétée dans des classeurs :

![Classeurs Anneau et liste](media/view-designer-conversion-tiles/donut-workbooks.png)

## <a name="line-chart--list-tile"></a>Vignette Graphique en courbes et liste
![Graphique en courbes et liste](media/view-designer-conversion-tiles/line-list.png) 

Pour recréer la partie graphique en courbes, mettez à jour la requête comme suit :

Requête d’origine
```KQL
search * 
| summarize AggregatedValue = count() by Type
```

Requête mise à jour
```KQL
search * 
| make-series Count = count() default=0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by Type
```

Il existe deux options pour visualiser le graphique en courbes

**Option 1 :** Sélectionnez **Graphique en courbes** dans la liste déroulante **Visualisation** :
 
 ![Menu Graphique en courbes](media/view-designer-conversion-tiles/line-visualization.png)

**Option 2 :** Sélectionnez **Définir par requête** dans la liste déroulante **Visualisation**, puis ajoutez `| render linechart` à la requête :

 ![Menu Visualisation](media/view-designer-conversion-tiles/set-by-query.png)

**Exemple**

```KQL
search * 
| make-series Count = count() default=0 on TimeGenerated from {TimeRange:start} to {TimeRange:end} step {TimeRange:grain} by Type 
| render linechart_
```

Pour créer une liste et activer les graphiques sparkline, voir l’article sur les [tâches courantes](view-designer-conversion-tasks.md).

Voici un exemple illustrant la manière dont la vignette Graphique en courbes et liste peut être réinterprétée dans des classeurs :

![Classeurs Graphique en courbes et liste](media/view-designer-conversion-tiles/line-workbooks.png)

## <a name="number--list-tile"></a>Vignette Nombre et liste

 ![Liste de vignettes](media/view-designer-conversion-tiles/tile-list-example.png)

Pour la vignette Nombre, mettez à jour la requête comme suit :

Requête d’origine
```KQL
search * 
| summarize AggregatedValue = count() by Computer | count
```

Requête mise à jour
```KQL
search *
| summarize AggregatedValue = count() by Computer 
| summarize Count = count()
```

Passez de la liste déroulante Visualisation à la liste **Vignettes**, puis sélectionnez **Paramètres de la vignette**.
 ![Visualisation des vignettes](media/view-designer-conversion-tiles/tile-visualization.png)

Laissez la section **Titre** vide, puis sélectionnez **Gauche**. Modifiez la valeur de **Utiliser la colonne :** en **Nombre**, et celle de **Convertisseur de colonne** en **Grand nombre** :

![Paramètres de la vignette](media/view-designer-conversion-tiles/tile-settings.png)

 
Pour créer une liste et activer les graphiques sparkline, voir l’article sur les [tâches courantes](view-designer-conversion-tasks.md).

Voici un exemple illustrant la manière dont la vignette Nombre et liste peut être réinterprétée dans des classeurs :

![Classeurs Nombre et liste](media/view-designer-conversion-tiles/number-workbooks.png)

## <a name="timeline--list"></a>Chronologie et liste

 ![Chronologie et liste](media/view-designer-conversion-tiles/time-list.png)

Pour Chronologie, mettez à jour votre requête comme suit :

Requête d’origine
```KQL
search * 
| sort by TimeGenerated desc
```

Requête mise à jour
```KQL
search * 
| summarize Count = count() by Computer, bin(TimeGenerated,{TimeRange:grain})
```

Il existe deux options pour visualiser la requête sous la forme d’un histogramme :

**Option 1 :** Sélectionnez **Histogramme** dans la liste déroulante **Visualisation** : ![Visualisation Histogramme](media/view-designer-conversion-tiles/bar-visualization.png)
 
**Option 2 :** Sélectionnez **Définir par requête** dans la liste déroulante **Visualisation**, puis ajoutez `| render barchart` à la requête :

 ![Menu Visualisation](media/view-designer-conversion-tiles/set-by-query.png)

 
Pour créer une liste et activer les graphiques sparkline, voir l’article sur les [tâches courantes](view-designer-conversion-tasks.md).

Voici un exemple illustrant la manière dont la vignette Chronologie et liste peut être réinterprétée dans des classeurs :

![Classeurs Chronologie et liste](media/view-designer-conversion-tiles/time-workbooks.png)

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble de la transition du Concepteur de vues en classeurs](view-designer-conversion-overview.md)
