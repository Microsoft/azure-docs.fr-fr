---
title: Diagnostic des incidents à l’aide de Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Découvrez comment diagnostiquer un incident à l’aide de Metrics Advisor et obtenir une vue détaillée des anomalies de vos données.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 08/19/2020
ms.author: mbullwin
ms.openlocfilehash: d1f792859aa5407cfaceda0e3ed1b5c21ee160f0
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92043372"
---
# <a name="how-to-diagnose-an-incident-using-metrics-advisor"></a>Procédure : Diagnostic d’un incident à l’aide de Metrics Advisor

Metrics Advisor propose plusieurs fonctionnalités de diagnostic, fournit une vue détaillée des incidents détectés et effectue une analyse de la cause racine. Lorsqu’un groupe d’anomalies est détecté sur une métrique, Metrics Advisor les regroupe dans une hiérarchie et les analyse.

> [!NOTE]
> Metrics Advisor prend en charge les diagnostics d’incidents des métriques d’au moins une dimension et mesure avec le type *numérique*. Votre métrique doit comporter pour chaque dimension une valeur de dimension agrégée comme SUM, utilisée pour créer la hiérarchie de diagnostics. Metrics Advisor propose des [**Paramètres de cumul automatique**](onboard-your-data.md#automatic-roll-up-settings) pour faciliter la génération de valeurs agrégées. 

Cliquez sur **Hub d’incidents** dans la fenêtre de navigation de gauche pour afficher tous les incidents d’une métrique donnée. En haut de la page, vous pouvez sélectionner différentes métriques pour voir leur configuration de détection et les résultats de la détection, et modifier l’intervalle de temps.

> [!TIP]
> Le **Hub d’incidents** est également accessible par les moyens suivants :
> * Cliquez sur un point de données dans la visualisation de votre métrique et utilisez les liens situés en bas de la fenêtre **Ajouter un commentaire** qui s’affiche.
> * Cliquez sur l’une des anomalies de l’onglet **Incidents** de votre métrique. 

La section **Vue d’ensemble** contient les résultats de la détection, notamment le nombre d’anomalies et d’alertes dans l’intervalle de temps sélectionné.

:::image type="content" source="../media/diagnostics/incident-hub-overview.png" alt-text="Hub d’incidents" lightbox="../media/diagnostics/incident-hub-overview.png":::

Les incidents détectés au sein de la métrique et de l’intervalle de temps sélectionnés apparaissent dans la **Liste des incidents**. Il existe des options permettant de filtrer et d’organiser les incidents, par exemple, par gravité. Cliquez sur l’un des incidents pour accéder à la page **Incident** et bénéficier de diagnostics plus poussés.

:::image type="content" source="../media/diagnostics/incident-list.png" alt-text="Hub d’incidents" lightbox="../media/diagnostics/incident-list.png":::

La section **Diagnostic** permet d’effectuer une analyse approfondie d’un incident et offre des outils servant à identifier les causes racines.

:::image type="content" source="../media/diagnostics/diagnose-incident.png" alt-text="Hub d’incidents" lightbox="../media/diagnostics/diagnose-incident.png" :::

## <a name="root-cause-advice"></a>Conseil cause racine

Lorsqu’un groupe d’anomalies est détecté dans une métrique et provoque un incident, Metrics Advisor tente d’analyser la cause racine de ce dernier. Le **Conseil cause racine** fournit des suggestions automatiques de causes probables de l’incident. Cette fonctionnalité n’est disponible que s’il existe une valeur agrégée dans la dimension. Si la métrique ne comporte pas de dimension, la cause racine sera elle-même. Les causes racines figurent dans le volet droit. Il peut arriver que plusieurs raisons apparaissent. Si le tableau ne contient pas de données, cela signifie que la dimension ne répond pas aux conditions requises pour effectuer l’analyse.

:::image type="content" source="../media/diagnostics/root-cause-advice.png" lightbox="../media/diagnostics/root-cause-advice.png" alt-text="Hub d’incidents":::


Lorsque la métrique de la cause racine est fournie avec des dimensions spécifiques, vous pouvez cliquer sur **Accéder à la métrique** pour afficher ses détails.

## <a name="incident-tree"></a>Arborescence de l’incident

En plus de l’analyse automatisée des causes racines potentielles, Metrics Advisor prend en charge l’analyse manuelle des causes racines à l’aide de **l’Arborescence de l’incident**. Il existe deux types d’arborescences des incidents sur la page Incident : l’arborescence de **diagnostic rapide** et **l’arborescence interactive**.

L’arborescence de diagnostic rapide permet de diagnostiquer un incident en cours, le nœud racine étant limité au nœud racine de l’incident actuel. Il est possible de développer et de réduire les nœuds de l’arborescence en cliquant dessus. Sa série s’affiche avec celle des incidents actuels dans le graphe au-dessus de l’arborescence.

L’arborescence interactive permet de diagnostiquer les incidents actuels, ainsi que d’autres plus anciens et ceux qui sont associés. Lorsque vous utilisez l’arborescence interactive, cliquez avec le bouton droit sur un nœud pour ouvrir un menu d’actions, dans lequel vous pouvez choisir une dimension pour monter dans la hiérarchie des nœuds racines et une autre pour explorer chaque nœud au niveau du détail. En cliquant sur le bouton Annuler de la liste de dimensions en haut, vous pouvez supprimer le zoom avant et arrière à partir de cette dimension. Cliquez sur un nœud pour le sélectionner et afficher sa série avec celle des incidents actuels dans le graphe.

:::image type="content" source="../media/diagnostics/incident-tree.png" alt-text="Hub d’incidents" lightbox="../media/diagnostics/incident-tree.png" :::

## <a name="anomaly-drill-down"></a>Exploration d’anomalies

Lorsque vous consultez des informations sur les incidents, il peut vous falloir plus de détails, par exemple, sur les différentes dimensions et les timestamps. Si vos données comportent une ou plusieurs dimensions, vous pouvez utiliser la fonction d’exploration au niveau du détail pour obtenir une vue plus détaillée. 

Pour utiliser la fonction d’exploration au niveau du détail, cliquez sur l’onglet **Exploration des métriques** dans le **Hub d’incidents**. 

:::image type="content" source="../media/diagnostics/metric-drilling.png" lightbox="../media/diagnostics/metric-drilling.png" alt-text="Hub d’incidents":::

Le paramètre **Dimensions** est une liste de dimensions pour un incident. Il est possible de sélectionner d’autres valeurs de dimension disponibles pour chacun d’eux. Après modification de la valeur de dimension Le paramètre **Timestamp** permet d’afficher l’incident actuel à différents moments dans le temps.

### <a name="select-drilling-options-and-choose-a-dimension"></a>Sélection des options d’exploration et choix d’une dimension

Il existe deux types d’options d’exploration : **Exploration au niveau du détail** et **Comparaison horizontale**.

> [!Note]
> 1. Pour descendre dans la hiérarchie, vous pouvez explorer les données de différentes valeurs de dimension, à l’exception des dimensions actuellement sélectionnées. 
> 2. La comparaison horizontale permet d’explorer les données de différentes valeurs de dimension, à l’exception des dimensions Tout.

:::image type="content" source="../media/diagnostics/drill-down-dimension.png"  lightbox="../media/diagnostics/drill-down-dimension.png" alt-text="Hub d’incidents":::

### <a name="value-comparison-for-different-dimension-values"></a>Comparaison de différentes valeurs de dimension

La deuxième section de l’onglet Exploration au niveau du détail est un tableau présentant des comparaisons pour différentes valeurs de dimension. Il comprend la valeur, la valeur de référence, la valeur de différence et la valeur delta, et indique s’il s’agit d’une anomalie.
 
:::image type="content" source="../media/diagnostics/drill-down-comparison.png" alt-text="Hub d’incidents" lightbox="../media/diagnostics/drill-down-comparison.png":::


### <a name="value-and-expected-value-comparisons-for-different-dimension-value"></a>Comparaisons entre les valeurs et les valeurs attendues pour différentes valeurs de dimension

La troisième section de l’onglet Exploration au niveau du détail est un histogramme comportant les valeurs et les valeurs attendues, pour différentes valeurs de dimension. L’histogramme est trié selon la différence entre la valeur et la valeur attendue. Il est facile de trouver la valeur inattendue présentant l’impact le plus important. Par exemple, dans l’illustration ci-dessus, nous pouvons constater que, à l’exception de la valeur Tout, c’est **US7** qui contribue le plus à l’anomalie.

:::image type="content" source="../media/diagnostics/drill-down-table.png" alt-text="Hub d’incidents" lightbox="../media/diagnostics/drill-down-table.png":::

### <a name="raw-value-visualization"></a>Visualisation de la valeur brute
La dernière partie de l’onglet Exploration au niveau du détail est un graphique en courbes des valeurs brutes. Ce graphique évite d’avoir à accéder à la page Métrique pour afficher les détails.

:::image type="content" source="../media/diagnostics/drill-down-line-chart.png" alt-text="Hub d’incidents" lightbox="../media/diagnostics/drill-down-line-chart.png":::

## <a name="view-similar-anomalies-using-time-series-clustering"></a>Affichage d’anomalies similaires à l’aide du clustering de séries chronologiques

Lorsque vous affichez un incident, vous pouvez utiliser l’onglet **Clustering de séries chronologiques similaires** pour voir les différentes séries associées. Les séries d’un même groupe sont résumées ensemble. L’image ci-dessus nous apprend qu’il existe au moins deux groupes de séries. Cette fonctionnalité n’est disponible que si les conditions suivantes sont remplies :

1. Les métriques doivent comporter une ou plusieurs dimensions ou valeurs de dimension.
2. Les séries d’une même métrique doivent présenter une tendance similaire.

Les dimensions disponibles figurent dans la partie supérieure de l’onglet. Vous pouvez effectuer une sélection pour spécifier la série.

:::image type="content" source="../media/diagnostics/series-group.png" lightbox="../media/diagnostics/series-group.png"alt-text="Hub d’incidents":::

## <a name="compare-time-series"></a>Comparaison de séries chronologiques

Lorsqu’une anomalie est détectée sur une série chronologique spécifique, il est parfois utile de la comparer à d’autres séries dans une seule visualisation. Cliquez sur l’onglet **Outils de comparaison**, puis cliquez sur le bouton bleu **+ Ajouter** . 

:::image type="content" source="../media/diagnostics/add-series.png" alt-text="Hub d’incidents" lightbox="../media/diagnostics/add-series.png":::

Sélectionnez une série à partir de votre flux de données. Vous pouvez choisir la même granularité ou non. Sélectionnez les dimensions cibles et chargez la tendance de la série, puis cliquez sur **OK** pour la comparer à une série précédente. Les séries seront regroupées en une seule visualisation. Vous pouvez ajouter d’autres séries pour comparaison afin d’obtenir des insights supplémentaires. Cliquez sur le menu déroulant en haut de l’onglet **Outils de comparaison** pour comparer les données de la série chronologique sur une période décalée.  

> [!Warning]
> Pour effectuer une comparaison, l’analyse des données de série chronologique peut nécessiter des déplacements de point de données. La granularité de vos données doit donc la prendre en charge. Par exemple, si vos données sont hebdomadaires et que vous utilisez la comparaison **Jour par jour**, vous ne recevrez aucun résultat. Dans cet exemple, vous devrez utiliser la comparaison **Mois par mois**.

Après avoir sélectionné une comparaison décalée, vous pouvez choisir si vous souhaitez comparer les valeurs de données, les valeurs delta ou le pourcentage delta.

> [!Note]
> * La **valeur des données** correspond à la valeur des données brutes.
> * La **valeur delta** est la différence entre la valeur brute et la valeur comparée.
> * Le **pourcentage de valeur delta** correspond à la différence entre la valeur brute et la valeur comparée divisée par la valeur comparée.

## <a name="related-incidents-across-metrics"></a>Incidents associés à plusieurs métriques

Il est parfois nécessaire de vérifier les incidents de différentes métriques en même temps ou des incidents associés dans d’autres métriques. Vous trouverez la liste des incidents associés dans la section **Analyse croisée des métriques**. 

:::image type="content" source="../media/graph/metrics-graph-cross-metrics-analysis.png" alt-text="Hub d’incidents":::

Pour pouvoir voir les incidents associés à la métrique actuelle, vous devez ajouter une relation entre les métriques. Cliquez sur **Paramètres du graphe de métriques** pour ajouter une relation. Seules les métriques possédant les mêmes noms de dimensions peuvent être associées. Utilisez les paramètres suivants.

- Flux de données et métrique actuels : flux de données et métrique de l’incident actuel
- Direction : sens de la relation entre deux métriques. (sans effet sur la liste des incidents associés maintenant)
- Autre flux de données et métrique : flux de données et métrique à associer à la métrique actuelle


## <a name="next-steps"></a>Étapes suivantes 

- [Ajustement de la détection d’anomalie selon les commentaires](anomaly-feedback.md)
- [Configuration des métriques et affinage de la configuration de la détection](configure-metrics.md)
