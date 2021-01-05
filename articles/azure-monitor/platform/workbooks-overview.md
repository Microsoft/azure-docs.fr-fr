---
title: Vue d’ensemble des classeurs Azure Monitor
description: Découvrez comment les classeurs fournissent un canevas flexible pour l’analyse des données et la création de rapports visuels enrichis au sein du portail Azure.
manager: carmonm
services: azure-monitor
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 07/23/2020
ms.openlocfilehash: 5dd553f0a41f82991c467ab256a87beabbff25ee
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92143639"
---
# <a name="azure-monitor-workbooks"></a>Classeurs Azure Monitor

Les classeurs fournissent un canevas flexible pour l’analyse des données et la création de rapports visuels enrichis au sein du portail Azure. Ils vous permettent d’exploiter plusieurs sources de données à travers l’écosystème Azure et de les combiner dans des expériences interactives unifiées. 

Voici une procédure vidéo expliquant la création de classeurs.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4B4Ap]

## <a name="data-sources"></a>Sources de données

Les classeurs peuvent interroger des données de plusieurs sources dans Azure. Les auteurs de classeurs peuvent transformer ces données pour fournir des insights sur la disponibilité, les performances, l’utilisation et l’intégrité globale des composants sous-jacents. Par exemple, l’analyse des journaux de performances à partir de machines virtuelles pour identifier les instances à UC élevée ou mémoire faible et afficher les résultats sous la forme d’une grille dans un rapport interactif.
  
Mais la véritable puissance des classeurs est la possibilité de combiner des données provenant de sources disparates au sein d’un même rapport. Cela permet la création de vues composites de ressources ou de jointures entre les ressources, ce qui permet d’obtenir des données et des insights plus riches, ce qui serait impossible sans cela.

Les classeurs sont actuellement compatibles avec les sources de données suivantes :

* [Journaux d’activité](workbooks-data-sources.md#logs)
* [Métriques](workbooks-data-sources.md#metrics)
* [Azure Resource Graph](workbooks-data-sources.md#azure-resource-graph)
* [Alertes (préversion)](workbooks-data-sources.md#alerts-preview)
* [Intégrité de la charge de travail](workbooks-data-sources.md#workload-health)
* [Azure Resource Health](workbooks-data-sources.md#azure-resource-health)
* [Explorateur de données Azure](workbooks-data-sources.md#azure-data-explorer)

## <a name="visualizations"></a>Visualisations

Les classeurs fournissent un ensemble complet de fonctionnalités pour la visualisation de vos données. Pour obtenir des exemples détaillés de chaque type de visualisation, vous pouvez consulter les exemples de liens ci-dessous :

* [Text](workbooks-text-visualizations.md)
* [Graphiques](workbooks-chart-visualizations.md)
* [Grilles](workbooks-grid-visualizations.md)
* [Vignettes](workbooks-tile-visualizations.md)
* [Arborescences](workbooks-tree-visualizations.md)
* [Graphes](workbooks-graph-visualizations.md)
* [Barre composite](workbooks-composite-bar.md)

![Exemples de visualisations de classeur](./media/workbooks-overview/visualizations.png)

## <a name="getting-started"></a>Prise en main

Pour explorer l’expérience des classeurs, accédez d’abord au service Azure Monitor. Pour ce faire, vous pouvez saisir **Monitor** dans la zone de recherche du portail Azure.

Sélectionnez ensuite **Classeurs**.

![Capture d’écran du bouton Classeurs mis en surbrillance dans une zone rouge](./media/workbooks-overview/workbooks.png)

### <a name="gallery"></a>Galerie

Vous accédez à la galerie de classeurs :

![Capture d’écran de la vue Galerie des classeurs Azure Monitor](./media/workbooks-overview/gallery.png)

### <a name="workbooks-versus-workbook-templates"></a>Classeurs et modèles de classeur

Vous pouvez voir un _classeur_ en vert et un certain nombre de _modèles de classeur_ en violet. Les modèles servent de rapports organisés qui sont conçus pour une réutilisation flexible par plusieurs utilisateurs et équipes. L’ouverture d’un modèle crée un classeur temporaire rempli avec le contenu du modèle. 

Vous pouvez ajuster les paramètres du classeur basé sur un modèle et effectuer une analyse sans craindre de perturber les futures expériences de création de rapports pour vos collègues. Si vous ouvrez un modèle, effectuez des ajustements, puis sélectionnez l’icône Enregistrer, vous enregistrerez le modèle en tant que classeur, qui s’afficherait alors en vert, laissant le modèle d’origine intact. 

En coulisses, les modèles diffèrent également des classeurs enregistrés. L’enregistrement d’un classeur crée une ressource Azure Resource Manager associée, tandis que le classeur temporaire créé lors de l’ouverture d’un modèle n’est associé à aucune ressource unique. Pour en savoir plus sur la gestion du contrôle d’accès dans les classeurs, consultez [l’article consacré au contrôle d’accès des classeurs](workbooks-access-control.md).

### <a name="exploring-a-workbook-template"></a>Exploration d’un modèle de classeur

Sélectionnez **Analyse de l’échec de l’application** pour voir un des modèles de classeur d’application par défaut.

![Capture d’écran du modèle d’analyse d’échec de l’application](./media/workbooks-overview/failure-analysis.png)

Comme indiqué précédemment, l’ouverture du modèle crée un classeur temporaire pour que vous puissiez interagir avec. Par défaut, le classeur s’ouvre en mode lecture, qui affiche uniquement les informations relatives à l’expérience d’analyse prévue qui a été créée par l’auteur du modèle d’origine.

Dans le cas de ce classeur particulier, l’expérience est interactive. Vous pouvez ajuster l’abonnement, les applications ciblées et l’intervalle de temps des données que vous souhaitez afficher. Une fois que vous avez effectué ces sélections, la grille des requêtes HTTP est également interactive, ce qui permet de modifier les données qui sont rendues dans les deux graphiques au bas du rapport.

### <a name="editing-mode"></a>Mode édition

Pour comprendre comment ce modèle de classeur est assemblé, vous devez basculer en mode édition en sélectionnant **Modifier**.

![Capture d’écran du bouton Modifier dans les classeurs.](./media/workbooks-overview/edit.png)

Une fois que vous êtes passé en mode édition, vous remarquerez un certain nombre de zones **Modifier** à de chaque aspect individuel de votre classeur.

![Capture d’écran du bouton Modifier](./media/workbooks-overview/edit-mode.png)

Si vous sélectionnez le bouton modifier se trouvant immédiatement sous la grille de données de requête, nous pouvons voir que cette partie de notre classeur se compose d’une requête Kusto sur les données d’une ressource Application Insights.

![Capture d’écran de la requête Kusto sous-jacente](./media/workbooks-overview/kusto.png)

En cliquant sur les autres boutons **Modifier** sur la droite, vous pouvez afficher un certain nombre de composants principaux qui composent les classeurs, comme des [zones de texte](workbooks-text-visualizations.md) des éléments d’interface utilisateur pour la [sélection de paramètres](workbooks-parameters.md) et d’autres [types de graphiques ou de visualisations](#visualizations). 

Explorez les modèles prédéfinis en mode édition, puis modifiez-les en fonction de vos besoins et enregistrez votre propre classeur personnalisé pour commencer à découvrir ce qui est possible avec les classeurs Azure Monitor.

## <a name="pinning-visualizations"></a>Épingler des visualisations

Les étapes de sélection de texte, de requête et de métriques d’un classeur peuvent être épinglées à l’aide du bouton Épingler sur ces éléments lorsque le classeur est en mode d’épinglage, ou si l’auteur du classeur a activé les paramètres de cet élément pour rendre l’icône d’épingle visible. 

Pour accéder au mode épinglage, cliquez sur **Modifier** pour passer en mode édition, puis sélectionnez l’icône en forme d’épingle bleue dans la barre supérieure. Une icône d’épingle est ensuite affichée au-dessus de la zone *Modifier* de chaque composant du classeur sur le côté droit de l’écran.

![Épingler l’expérience](./media/workbooks-overview/pin-experience.png)

> [!NOTE]
> L’état du classeur est enregistré au moment de l’épinglage et les classeurs épinglés sur un tableau de bord ne sont pas mis à jour si le classeur sous-jacent est modifié. Pour mettre à jour une partie de classeur épinglée, vous devez supprimer et réépingler cette partie.

## <a name="dashboard-time-ranges"></a>Intervalles de temps du tableau de bord

Les parties de requête des classeurs épinglés respectent l’intervalle de temps du tableau de bord si l’élément épinglé est configuré pour utiliser un paramètre *Intervalle de temps*. La valeur de l’intervalle de temps du tableau de bord sera utilisée comme valeur du paramètre d’intervalle de temps, et toute modification de l’intervalle de temps du tableau de bord entraînera la mise à jour de l’élément épinglé. Si une partie épinglée utilise l’intervalle de temps du tableau de bord, vous verrez le sous-titre de la partie épinglée mis à jour pour afficher l’intervalle de temps du tableau de bord chaque fois que l’intervalle de temps change. 

En outre, les parties de classeur épinglées à l’aide d’un paramètre d’intervalle de temps sont actualisées automatiquement à une cadence déterminée par l’intervalle de temps du tableau de bord. L’heure de la dernière exécution de la requête s’affiche dans le sous-titre de la partie épinglée.

Si une étape épinglée a une plage de temps définie explicitement (n’utilise pas de paramètre d’intervalle de temps), cet intervalle de temps sera toujours utilisé pour le tableau de bord, quels que soient les paramètres du tableau de bord. Le sous-titre de la partie épinglée n’affiche pas l’intervalle de temps du tableau de bord, et la requête n’est pas actualisée automatiquement sur le tableau de bord. Le sous-titre indique l’heure de la dernière exécution de la requête.

> [!NOTE]
> Les requêtes utilisant la source de données *fusionner* ne sont actuellement pas prises en charge lors de l’épinglage aux tableaux de bord.

## <a name="sharing-workbook-templates"></a>Partage de modèles de classeurs

Une fois que vous avez commencé à créer vos propres modèles de classeur, vous souhaiterez peut-être les partager avec la communauté. Pour plus d’informations et pour découvrir d’autres modèles qui ne font pas partie de la galerie de vues par défaut d’Azure Monitor, visitez notre [dépôt GitHub](https://github.com/Microsoft/Application-Insights-Workbooks/blob/master/README.md). Pour parcourir les classeurs existants, accédez à la [bibliothèque de classeurs](https://github.com/microsoft/Application-Insights-Workbooks/tree/master/Workbooks) sur GitHub.

## <a name="next-step"></a>Étape suivante

* [Commencez](#visualizations) à en apprendre davantage sur les nombreuses options pour les visualisations enrichies des classeurs.
* [Contrôlez](workbooks-access-control.md) et partagez l’accès à vos ressources de classeur.