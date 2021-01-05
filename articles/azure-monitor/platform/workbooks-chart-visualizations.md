---
title: Visualisations de graphiques de classeur Azure Monitor
description: Découvrez toutes les visualisations de graphiques de classeur Azure Monitor.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 09/04/2020
ms.author: lagayhar
ms.openlocfilehash: 139c8cdb1a227595d728d2acc0b09cf1eb210715
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90006378"
---
# <a name="chart-visualizations"></a>Visualisations de graphiques

Les classeurs permettent de présenter les données d’analyse sous forme de graphiques. Les types de graphiques pris en charge incluent les courbes, les barres, les catégories, les surfaces, les nuages de points, les secteurs et le temps. Les auteurs peuvent choisir de personnaliser la hauteur, la largeur, la palette de couleurs, la légende, les titres, les messages sans données, etc. du graphique et de personnaliser les types d’axe et les couleurs des séries à l’aide des paramètres du graphique.

Les classeurs prennent en charge les graphiques pour les journaux et les sources de données métriques.

## <a name="log-charts"></a>Graphiques de journal

La fonctionnalité Journaux d’activité d’Azure Monitor fournit aux propriétaires de ressources des informations détaillées sur le fonctionnement de leurs applications et de leur infrastructure. Contrairement aux métriques, les informations de journal ne sont pas collectées par défaut et nécessitent une sorte d’intégration de la collecte. Toutefois, lorsqu’ils sont présents, les journaux fournissent un grand nombre d’informations sur l’état de la ressource et des données utiles pour le diagnostic. Les classeurs permettent de présenter les données du journal sous forme de graphiques visuels pour l’analyse de l’utilisateur.

### <a name="adding-a-log-chart"></a>Ajout d’un graphique de journal

L’exemple ci-dessous montre la tendance des requêtes adressées à une application au cours des jours précédents.

1. Basculez le classeur en mode d’édition en sélectionnant l’élément **Modifier** de la barre d’outils.
2. Utilisez le lien **Ajouter une requête** pour ajouter un contrôle de requête de journal au classeur.
3. Sélectionnez le type de requête **Journal**, le type de ressource (par exemple, Application Insights) et les ressources à cibler.
4. Utiliser l’éditeur de requête pour saisir le [KQL](/azure/kusto/query/) pour votre analyse (par exemple, la tendance des requêtes).
5. Définissez la visualisation sur un des éléments suivants : **Zones**, **Barres**, **Bar (catégorie)** , **Courbes**, **Secteurs**, **Nuage de points** ou **Temps**.
6. Définissez d’autres paramètres si nécessaire, comme l’intervalle de temps, la visualisation, la taille, la palette de couleurs et la légende.

[![Capture d’écran du graphique de journal en mode édition](./media/workbooks-chart-visualizations/log-chart.png)](./media/workbooks-chart-visualizations/log-chart.png#lightbox)

### <a name="log-chart-parameters"></a>Paramètres du graphique de journal

| Paramètre | Explication | Exemple |
| ------------- |:-------------|:-------------|
| `Query Type` | Type de requête à utiliser. | Journal, Azure Resource Graph, etc. |
| `Resource Type` | Type de ressource à cibler. | Application Insights, Log Analytics ou Azure d’abord |
| `Resources` | Ensemble de ressources à partir duquel obtenir les valeurs de métriques. | MyApp1 |
| `Time Range` | Fenêtre de temps pour afficher le graphique du journal. | Dernière heure, dernières 24 heures, etc. |
| `Visualization` | Visualisation à utiliser. | Zones, Barres, Courbes, Secteurs, Nuages de points, Temps, Catégories |
| `Size` | Taille verticale du contrôle. | Petite, moyenne, grande ou maximale |
| `Color palette` | Palette de couleurs à utiliser dans le graphique. Ignoré en mode multi-métrique ou segmenté. | Bleu, vert, rouge, etc. |
| `Legend` | Fonction d’agrégation à utiliser pour la légende. | Somme ou moyenne des valeurs, ou valeur max, min, première, dernière |
| `Query` | Toute requête KQL qui retourne des données au format attendu par la visualisation du graphique. | _requests \| make-series Requests = count() default = 0 on timestamp from ago(1d) to now() step 1h_ |

### <a name="time-series-charts"></a>Graphiques de séries chronologiques

Vous pouvez facilement créer des graphiques de séries chronologiques, par exemple en zones, barres, courbes, nuages de points et heures, en utilisant le contrôle de requête dans Classeurs. L’essentiel est de disposer d’informations temporelles et métriques dans le jeu de résultats.

#### <a name="simple-time-series"></a>Série chronologique simple

La requête ci-dessous retourne une table avec deux colonnes : *timestamp* et *Requêtes*. Le contrôle de requête utilise *timestamp* pour l’axe X et *Requêtes* pour l’axe Y.

```kusto
requests
| summarize Requests = count() by bin(timestamp, 1h)
```

[![Screenshot of a simple time-series log line chart.](./media/workbooks-chart-visualizations/log-chart-line-simple.png)](./media/workbooks-chart-visualizations/log-chart-line-simple.png#lightbox)

#### <a name="time-series-with-multiple-metrics"></a>Série chronologique avec plusieurs métriques

La requête ci-dessous retourne une table avec trois colonnes : *timestamp*, *Requêtes* et *Utilisateurs*. Le contrôle de requête utilise *timestamp* pour l’axe X et *Requêtes* et *Utilisateurs* sous forme de séries distinctes pour l’axe Y.

```kusto
requests
| summarize Requests = count(), Users = dcount(user_Id) by bin(timestamp, 1h)
```

[![Screenshot of a time-series with multiple metrics log line chart.](./media/workbooks-chart-visualizations/log-chart-line-multi-metric.png)](./media/workbooks-chart-visualizations/log-chart-line-multi-metric.png#lightbox)

#### <a name="segmented-time-series"></a>Série chronologique segmentée

La requête ci-dessous retourne une table avec trois colonnes : *timestamp*, *Requêtes* et *RequestName*, où *RequestName* est une colonne catégorique avec les noms des requêtes. Le contrôle de requête utilise ici *timestamp* pour l’axe X et ajoute une série par valeur de *RequestName*.

```
requests
| summarize Request = count() by bin(timestamp, 1h), RequestName = name
```

[![Screenshot of a segmented time-series log line chart.](./media/workbooks-chart-visualizations/log-chart-line-segmented.png)](./media/workbooks-chart-visualizations/log-chart-line-segmented.png#lightbox)

### <a name="summarize-vs-make-series"></a>Summarize par rapport à make-series

Les exemples de la section précédente utilisent l’opérateur `summarize`, car il est plus facile à comprendre. Toutefois, celui-ci présente une limitation majeure, car il omet la ligne des résultats s’il n’y a aucun élément dans le compartiment. Cela peut avoir pour effet de décaler la fenêtre temporelle du graphique selon que les compartiments vides se trouvent à l’avant ou à l’arrière de l’intervalle de temps.

Il est généralement préférable d’utiliser l’opérateur `make-series` pour créer des données de série chronologique, qui a la possibilité de fournir des valeurs par défaut pour les compartiments vides.

La requête ci-dessous utilise l’opérateur `make-series`.

```kusto
requests
| make-series Requests = count() default = 0 on timestamp from ago(1d) to now() step 1h by RequestName = name
```

La requête ci-dessous montre un graphique similaire avec l’opérateur `summarize`.

```kusto
requests
| summarize Request = count() by bin(timestamp, 1h), RequestName = name
```

Même si le jeu de résultats sous-jacent est différent, il suffit à l’utilisateur de régler la visualisation sur une zone, une ligne, une barre ou une heure, et Classeurs s’occupe du reste.

[![Screenshot of a log line chart made from a make-series query](./media/workbooks-chart-visualizations/log-chart-line-make-series.png)](./media/workbooks-chart-visualizations/log-chart-line-make-series.png#lightbox)

### <a name="categorical-bar-chart-or-histogram"></a>Histogramme ou graphique catégorique à barres

Les graphiques catégoriques permettent aux utilisateurs de représenter une dimension ou une colonne sur l’axe X d’un graphique, ce qui est particulièrement utile dans les histogrammes. L’exemple ci-dessous montre la répartition des requêtes par code résultat.

```kusto
requests
| summarize Requests = count() by Result = strcat('Http ', resultCode)
| order by Requests desc
```

La requête retourne deux colonnes : la métrique *Requêtes* et la catégorie *Résultat*. Chaque valeur de la colonne *Résultat* obtiendra sa propre barre dans le graphique avec une hauteur proportionnelle à la métrique *Requêtes*.

[![Screenshot of a categorical bar chart for requests by result code](./media/workbooks-chart-visualizations/log-chart-categorical-bar.png)](./media/workbooks-chart-visualizations/log-chart-categorical-bar.png#lightbox)

### <a name="pie-charts"></a>Graphiques en secteurs

Les graphiques en secteurs permettent de visualiser les proportions numériques. L’exemple ci-dessous montre la proportion des requêtes par code résultat.

```kusto
requests
| summarize Requests = count() by Result = strcat('Http ', resultCode)
| order by Requests desc
```

La requête retourne deux colonnes : la métrique *Requêtes* et la catégorie *Résultat*. Chaque valeur de la colonne *Résultat* obtiendra sa propre section dans le graphique avec une taille proportionnelle à la métrique *Requêtes*.

[![Screenshot of a pie chart with slices representing result code](./media/workbooks-chart-visualizations/log-chart-pie-chart.png)](./media/workbooks-chart-visualizations/log-chart-pie-chart.png#lightbox)

## <a name="metric-charts"></a>Graphiques de métrique

La plupart des ressources Azure émettent des données métriques sur l’état et l’intégrité (par exemple, l’utilisation du processeur, la disponibilité du stockage, le nombre de transactions de base de données, les requêtes d’application défaillantes, etc.). Les classeurs permettent de visualiser ces données sous forme de graphiques de série chronologique.

### <a name="adding-a-metric-chart"></a>Ajout d’un graphique de métrique

L’exemple suivant indique le nombre de transactions dans un compte de stockage au cours de l’heure précédente. Cela permet au propriétaire du stockage de voir la tendance des transactions et de rechercher des anomalies dans le comportement.

1. Basculez le classeur en mode d’édition en sélectionnant l’élément **Modifier** de la barre d’outils.
2. Utilisez le lien **Ajouter une métrique** pour ajouter un contrôle de métrique au classeur.
3. Sélectionnez un type de ressource (par exemple, compte de stockage), les ressources à cibler, l’espace de noms et le nom de la métrique, et l’agrégation à utiliser.
4. Définissez d’autres paramètres si nécessaire, comme la plage de temps, le fractionnement, la visualisation, la taille et la palette de couleurs.

[![Capture d’écran du graphique de métrique en mode édition](./media/workbooks-chart-visualizations/metric-chart.png)](./media/workbooks-chart-visualizations/metric-chart.png#lightbox)

### <a name="metric-chart-parameters"></a>Paramètres du graphique de métriques

| Paramètre | Explication | Exemple |
| ------------- |:-------------|:-------------|
| `Resource Type` | Type de ressource à cibler. | Stockage ou machine virtuelle. |
| `Resources` | Ensemble de ressources à partir duquel obtenir les valeurs de métriques. | MyStorage1 |
| `Namespace` | Espace de noms avec la métrique. | Stockage > Blob |
| `Metric` | Métrique à visualiser. | Stockage > Blob > Transactions |
| `Aggregation` | Fonction d’agrégation à appliquer à la métrique. | Somme, Total, Moyenne, etc. |
| `Time Range` | Fenêtre temporelle dans laquelle afficher la métrique. | Dernière heure, dernières 24 heures, etc. |
| `Visualization` | Visualisation à utiliser. | Zones, Barres, Courbes, Nuages de points et Grille |
| `Split By` | Fractionner éventuellement la métrique sur une dimension. | Transactions par type de zone géographique |
| `Size` | Taille verticale du contrôle. | Petit, Moyen ou Grand |
| `Color palette` | Palette de couleurs à utiliser dans le graphique. Ignoré si le paramètre `Split by` est utilisé. | Bleu, vert, rouge, etc. |

### <a name="examples"></a>Exemples

Transactions fractionnées par nom d’API sous forme de graphique en courbes :

[![Screenshot of a metric line chart for Storage transactions split by API name](./media/workbooks-chart-visualizations/metric-chart-storage-split-line.png)](./media/workbooks-chart-visualizations/metric-chart-storage-split-line.png#lightbox)

Transactions fractionnées par type de réponse sous forme de grand graphique à barres :

[![Screenshot of a large metric bar chart for Storage transactions split by response type](./media/workbooks-chart-visualizations/metric-chart-storage-bar-large.png)](./media/workbooks-chart-visualizations/metric-chart-storage-bar-large.png#lightbox)

Latence moyenne sous forme de graphique à nuages de points :

[![Screenshot of a metric scatter chart for Storage latency](./media/workbooks-chart-visualizations/metric-chart-storage-scatter.png)](./media/workbooks-chart-visualizations/metric-chart-storage-scatter.png#lightbox)

## <a name="chart-settings"></a>Paramètres du graphique

Les auteurs peuvent utiliser des paramètres de graphique pour personnaliser les champs utilisés dans les axes des graphiques, les unités des axes, la mise en forme personnalisée, les plages, les comportements de regroupement, les légendes et les couleurs des séries.

### <a name="the-settings-tab"></a>Onglet de paramètres

L’onglet des paramètres contrôle les éléments suivants :

- Les paramètres des axes, notamment les champs, la mise en forme personnalisée qui permet aux utilisateurs de définir la mise en forme des nombres en fonction des valeurs des axes et les plages personnalisées.
- Les paramètres de regroupement, notamment le champ, les limites avant la création d’un groupe « Autres ».
- Les paramètres de légende, notamment l’indication des métriques (nom de la série, couleurs et nombres) en bas et/ou une légende (noms et couleurs des séries).

![Capture d’écran des paramètres du graphique.](./media/workbooks-chart-visualizations/chart-settings.png)

#### <a name="custom-formatting"></a>Mise en forme personnalisée

Les options de mise en forme des nombres sont les suivantes :

| Option de mise en forme             | Explication                                                                                           |
|:---------------------------- |:-------------------------------------------------------------------------------------------------------|
| `Units`                      | Unités de la colonne : différentes options pour le pourcentage, les nombres, l’heure, l’octet, le nombre/l’heure, les octets/l’heure, etc. Par exemple, l’unité d’une valeur de 1234 peut être définie sur millisecondes et être rendue sous la forme de 1 234 s.                                  |
| `Style`                      | Format à afficher en décimal, en devise, en pourcentage.                                               |
| `Show group separator`       | Case à cocher pour afficher les séparateurs de groupes. Affiche 1234 sous la forme de 1 234 en France.                                    |
| `Minimum integer digits`     | Nombre minimal de chiffres entiers à utiliser (valeur par défaut 1).                                                   |
| `Minimum fractional digits`  | Nombre minimal de chiffres fractionnaires à utiliser (valeur par défaut 0).                                                |
| `Maximum fractional digits`  | Nombre maximal de chiffres fractionnaires à utiliser.                                                            |
| `Minimum significant digits` | Nombre minimal de chiffres significatifs à utiliser (valeur par défaut 1).                                               |
| `Maximum significant digits` | Nombre maximal de chiffres significatifs à utiliser.                                                           |

![Capture d’écran des paramètres de l’axe X.](./media/workbooks-chart-visualizations/number-format-settings.png)

### <a name="the-series-tab"></a>Onglet des séries

L’onglet des paramètres de la série vous permet d’ajuster les étiquettes et les couleurs affichées pour les séries dans le graphique.

- Le champ `Series name` est utilisé pour faire correspondre une série à des données et, s’il y a correspondance, l’étiquette et la couleur d’affichage sont affichées.
- Le champ `Comment` est utile pour les créateurs de modèles, car ce commentaire peut être utilisé par les traducteurs pour localiser les étiquettes d’affichage.

![Capture d’écran des paramètres de la série.](./media/workbooks-chart-visualizations/series-settings.png)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment créer une [vignette dans les classeurs](workbooks-tile-visualizations.md).
- Découvrez comment créer des [classeurs interactifs](workbooks-interactive.md).