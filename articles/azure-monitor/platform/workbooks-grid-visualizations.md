---
title: Visualisations de grille de classeur Azure Monitor
description: Découvrez toutes les visualisations de grille de classeur Azure Monitor.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 09/04/2020
ms.author: lagayhar
ms.openlocfilehash: 114245030ba2356861b83a580f103d3b84c0eae3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89663135"
---
# <a name="grid-visualizations"></a>Visualisations de grille

Les grilles ou les tables sont un moyen courant de présenter des données aux utilisateurs. Les classeurs permettent aux utilisateurs de définir un style individuel pour les colonnes de la grille pour fournir une interface utilisateur riche à leurs rapports.

L’exemple ci-dessous montre une grille qui combine des icônes, des cartes thermiques et des sparklines pour présenter des informations complexes. Le classeur fournit également des opérations de tri, une zone de recherche et un bouton Aller à l’analyse.

[![Capture d’écran de la grille basée sur un journal](./media/workbooks-grid-visualizations/grid.png)](./media/workbooks-grid-visualizations/grid.png#lightbox)

## <a name="adding-a-log-based-grid"></a>Ajout d’une grille basée sur un journal

1. Basculez le classeur en mode d’édition en cliquant sur l’élément **Modifier** de la barre d’outils.
2. Utilisez le lien **Ajouter une requête** pour ajouter un contrôle de requête de journal au classeur.
3. Sélectionnez le type de requête **Journal**, le type de ressource (par exemple, Application Insights) et les ressources à cibler.
4. Utilisez l’éditeur de requête pour saisir le KQL pour votre analyse (par exemple, les machines virtuelles dont la mémoire est inférieure à un seuil)
5. Définir la visualisation sur **Grille**
6. Définissez d’autres paramètres si nécessaire, comme la plage de temps, la taille, la palette de couleurs et la légende.

[![Capture d’écran de la requête de grille basée sur un journal](./media/workbooks-grid-visualizations/grid-query.png)](./media/workbooks-grid-visualizations/grid-query.png#lightbox)

## <a name="log-chart-parameters"></a>Paramètres du graphique de journal

| Paramètre | Explication | Exemple |
| ------------- |:-------------|:-------------|
| `Query Type` | Type de requête à utiliser. | Journal, Azure Resource Graph, etc. |
| `Resource Type` | Type de ressource à cibler. | Application Insights, Log Analytics ou Azure d’abord |
| `Resources` | Ensemble de ressources à partir duquel obtenir les valeurs de métriques. | MyApp1 |
| `Time Range` | Fenêtre de temps pour afficher le graphique du journal. | Dernière heure, dernières 24 heures, etc. |
| `Visualization` | Visualisation à utiliser. | Grille |
| `Size` | Taille verticale du contrôle. | Petite, moyenne, grande ou maximale |
| `Query` | Toute requête KQL qui retourne des données au format attendu par la visualisation du graphique. | _requests \| summarize Requests = count() by name_ |

## <a name="simple-grid"></a>Grille simple

Les classeurs peuvent restituer les résultats KQL sous la forme d’un tableau simple. La grille ci-dessous affiche le nombre de requêtes et d’utilisateurs uniques par type de requête dans une application.

```kusto
requests
| where name !endswith('.eot')
| summarize Requests = count(), Users = dcount(user_Id) by name
| order by Requests desc
```

[![Capture d’écran d’une grille de journal en mode édition](./media/workbooks-grid-visualizations/log-chart-simple-grid.png)](./media/workbooks-grid-visualizations/log-chart-simple-grid.png#lightbox)

## <a name="grid-styling"></a>Application de styles à une grille

Bien qu’un tableau simple affiche des données, sa lecture est difficile et les Insights n’y apparaissent pas toujours de façon évidente. Le fait d’appliquer un style à la grille peut faciliter la lecture et l’interprétation des données.

Voici la même grille que celle de la section précédente, avec l’application du style à cartes thermiques.

[![Capture d’écran d’une grille de journal avec le style à cartes thermiques appliqué aux colonnes](./media/workbooks-grid-visualizations/log-chart-grid-heatmap.png)](./media/workbooks-grid-visualizations/log-chart-grid-heatmap.png#lightbox)

Voici la même grille, avec le style à barres : [![Capture d’écran d’une grille de journal avec l’application du style à barres dans les colonnes](./media/workbooks-grid-visualizations/log-chart-grid-bar.png)](./media/workbooks-grid-visualizations/log-chart-grid-bar.png#lightbox)

### <a name="styling-a-grid-column"></a>Définition d’un style de colonne de grille

1. Sélectionnez le bouton **Paramètres de colonne** dans la barre d’outils du contrôle de requête.
2. Dans le panneau *Modifier les paramètres de colonne*, sélectionnez la colonne dans laquelle appliquer un style.
3. Choisissez un convertisseur de colonne (par exemple, carte thermique, barre, barre en dessous, etc.) et les paramètres associés pour définir le style de votre colonne.

Voici un exemple qui applique le style à barres dans la colonne des *Requêtes* :

[![Capture d’écran d’une grille de journal avec le style à barres appliqué dans la colonne des requêtes.](./media/workbooks-grid-visualizations/log-chart-grid-column-settings-start.png)](./media/workbooks-grid-visualizations/log-chart-grid-column-settings-start.png#lightbox)

### <a name="column-renderers"></a>Convertisseurs de colonne

| Convertisseur de colonne | Explication | Options supplémentaires |
|:------------- |:-------------|:-------------|
| `Automatic` | Valeur par défaut - utilise le convertisseur qui convient le mieux en fonction du type de colonne.  |  |
| `Text` | Génère le rendu des valeurs de la colonne sous forme de texte. | |
| `Right Aligned` | Semblable à du texte, mais il est aligné à droite. | |
| `Date/Time` | Génère le rendu d’une chaîne de date et d’heure lisible. | |
| `Heatmap` | Colore les cellules de la grille en fonction de la valeur de la cellule. | Palette de couleurs et valeur minimale/maximale utilisée pour la mise à l’échelle. |
| `Bar` | Restitue une barre en regard de la cellule, selon la valeur de la cellule. | Palette de couleurs et valeur minimale/maximale utilisée pour la mise à l’échelle. |
| `Bar underneath` | Restitue une barre dans le bas de la cellule, en fonction de la valeur de la cellule. | Palette de couleurs et valeur minimale/maximale utilisée pour la mise à l’échelle. |
| `Composite bar` | Effectue le rendu d’une barre composite au moyen des colonnes spécifiées dans la ligne. Pour plus d’informations, consultez [Barre composite](workbooks-composite-bar.md). | Colonnes avec les couleurs correspondantes pour restituer la barre, et une étiquette à afficher en haut de la barre. |
| `Spark bars` | Génère le rendu d’une barre de graphique sparkline dans la cellule, en fonction des valeurs d’un tableau dynamique dans la cellule. Par exemple, la colonne Tendance de la capture d’écran plus haut. | Palette de couleurs et valeur minimale/maximale utilisée pour la mise à l’échelle. |
| `Spark lines` | Génère le rendu d’une ligne de graphique sparkline dans la cellule, en fonction des valeurs d’un tableau dynamique dans la cellule. | Palette de couleurs et valeur minimale/maximale utilisée pour la mise à l’échelle. |
| `Icon` | Restitue des icônes en fonction des valeurs de texte dans la cellule. Les valeurs prises en charge incluent : `cancelled`, `critical`, `disabled`, `error`, `failed`, `info`, `none`, `pending`, `stopped`, `question`, `success`, `unknown`, `warning` `uninitialized`, `resource`, `up`, `down`, `left`, `right`, `trendup`, `trenddown`, `4`, `3`, `2`, `1`, `Sev0`, `Sev1`, `Sev2`, `Sev3`, `Sev4`, `Fired`, `Resolved`, `Available`, `Unavailable`, `Degraded`, `Unknown` et `Blank`.|  |
| `Link` | Effectue le rendu d’un lien qui, lorsque l’utilisateur clique dessus, exécute une action configurable. Utilisez cette option si vous souhaitez que l’élément soit *uniquement* un lien.  Tous les autres types peuvent *également* être un lien en utilisant le paramètre `Make this item a link`. Pour plus d’informations, consultez [Actions du lien](#link-actions) ci-dessous. |  |
| `Location` | Génère le rendu d’un nom de région Azure convivial, basé sur des ID de région. |  |
| `Resource type` | Génère le rendu d’une chaîne type de ressource convivial en fonction d’un ID de type de ressource  |  |
| `Resource` | Restitue un nom de ressource convivial et un lien basé sur un ID de ressource  | Option permettant d’afficher l’icône du type de ressource  |
| `Resource group` | Restitue un nom de groupe de ressources convivial et un lien basé sur un ID de groupe de ressources. Si la valeur de la cellule n’est pas un groupe de ressources, elle sera convertie comme tel.  | Option permettant d’afficher l’icône du groupe de ressources  |
| `Subscription` | Génère le rendu d’un nom d’abonnement convivial et d’un lien basé sur un ID d’abonnement. Si la valeur de la cellule n’est pas un abonnement, elle sera convertie comme tel.  | Option permettant d’afficher l’icône d’abonnement.  |
| `Hidden` | Masque la colonne dans la grille. Utile lorsque la requête par défaut retourne plus de colonnes que nécessaire alors qu’un projet-away n’est pas souhaitable. |  |

### <a name="link-actions"></a>Actions du lien

Si le convertisseur `Link` est sélectionné ou que la case à cocher *Convertir cet élément en lien* est activée, l’auteur peut configurer une action de lien qui se produira à la sélection de la cellule. En général, l’utilisateur accède à un autre affichage en rapport avec le contexte de la cellule, ou il peut ouvrir une URL.

### <a name="custom-formatting"></a>Mise en forme personnalisée

Les classeurs permettent également aux utilisateurs de définir la mise en forme des nombres de leurs valeurs de cellule. Pour ce faire, ils peuvent cliquer sur la case à cocher *Mise en forme personnalisée*, lorsqu’elle est disponible.

| Option de mise en forme | Explication |
|:------------- |:-------------|
| `Units` | Unités de la colonne : différentes options pour le pourcentage, les nombres, l’heure, l’octet, le nombre/l’heure, les octets/l’heure, etc. Par exemple, l’unité d’une valeur de 1234 peut être définie sur millisecondes, et être rendue sous la forme de 1 234 s. |
| `Style` | Format à afficher en décimal, en devise, en pourcentage. |
| `Show group separator` | Case à cocher pour afficher les séparateurs de groupes. Affiche 1234 sous la forme de 1 234 en France. |
| `Minimum integer digits` | Nombre minimal de chiffres entiers à utiliser (valeur par défaut 1). |
| `Minimum fractional digits` | Nombre minimal de chiffres fractionnaires à utiliser (valeur par défaut 0).  |
| `Maximum fractional digits` | Nombre maximal de chiffres fractionnaires à utiliser. |
| `Minimum significant digits` | Nombre minimal de chiffres significatifs à utiliser (valeur par défaut 1). |
| `Maximum significant digits` | Nombre maximal de chiffres significatifs à utiliser. |
| `Custom text for missing values` | Quand un point de données n’a pas de valeur, ce texte personnalisé est affiché à la place d’un espace vide. |

### <a name="custom-date-formatting"></a>Mise en forme personnalisée de la date

Lorsque l’auteur spécifie qu’une colonne est définie sur le convertisseur Date/Heure, il peut indiquer des options de mise en forme de date personnalisée en activant la case à cocher *Mise en forme personnalisée de la date*.

| Option de mise en forme | Explication |
|:------------- |:-------------|
| `Style` | Format de rendu d’une date selon un format court, long, complet, ou d’une heure dans un format d’heure courte ou longue. |
| `Show time as` | Permet à l’auteur de choisir entre l’affichage de l’heure locale (par défaut) et l’heure UTC (temps universel coordonné). Selon le style du format de date sélectionné, les informations relatives au fuseau horaire UTC/heure peuvent ne pas s’afficher. |

## <a name="custom-column-width-setting"></a>Paramètre de largeur de colonne personnalisée

L’auteur peut personnaliser la largeur de n’importe quelle colonne de la grille à l’aide du champ *Largeur de colonne personnalisée* dans *Paramètres de colonne*.

![Capture d’écran des paramètres de colonne avec le champ Largeur de colonne personnalisée mis en évidence dans un encadré rouge](./media/workbooks-grid-visualizations/custom-column-width-setting.png)

Si le champ est laissé vide, la largeur est automatiquement déterminée en fonction du nombre de caractères dans la colonne, et du nombre de colonnes visibles. L’unité par défaut est « ch » (caractères).

Si vous sélectionnez le bouton **(Largeur actuelle)** en bleu dans l’étiquette, le champ de texte est renseigné avec la largeur existante de la colonne sélectionnée. Si une valeur est présente dans le champ de la largeur personnalisée sans indication d’unité de mesure, la valeur par défaut de l’unité de mesure est utilisée.

Les unités de mesure disponibles sont les suivantes :

| Unité de mesure | Définition           |
|:--------------------|:---------------------|
| ch                  | caractères (par défaut) |
| px                  | pixels               |
| fr                  | unités fractionnaires     |
| %                   | percentage           |

Validation de l’entrée : si la validation échoue, un message d’aide s’affiche en rouge en dessous du champ, mais l’utilisateur peut toujours appliquer la largeur. Si une valeur est présente dans l’entrée, elle est analysée. Si aucune unité de mesure valide n’est trouvée, la valeur par défaut est utilisée.

Il n’y a pas de largeur minimale/maximale, car elles sont laissées à l’appréciation de l’auteur. Le champ de la largeur de colonne personnalisée est désactivé pour les colonnes masquées.

## <a name="examples"></a>Exemples

### <a name="spark-lines-and-bar-underneath"></a>Graphique sparkline et barre en dessous

L’exemple ci-dessous montre le nombre de requêtes et leur tendance, par nom de requête.

```kusto
requests
| make-series Trend = count() default = 0 on timestamp from ago(1d) to now() step 1h by name
| project-away timestamp
| join kind = inner (requests
    | summarize Requests = count() by name
    ) on name
| project name, Requests, Trend
| order by Requests desc
```

[![Capture d’écran d’une grille de journal, avec une barre en dessous et un graphique sparkline](./media/workbooks-grid-visualizations/log-chart-grid-spark-line.png)](./media/workbooks-grid-visualizations/log-chart-grid-spark-line.png#lightbox)

### <a name="heatmap-with-shared-scales-and-custom-formatting"></a>Carte thermique avec échelles partagées et mise en forme personnalisée

Cet exemple montre diverses métriques de durée de requête et leurs nombres. Le convertisseur de carte thermique utilise les valeurs minimales définies dans les paramètres, ou il calcule une valeur minimale et une valeur maximale pour la colonne, puis affecte une couleur d’arrière-plan de la palette sélectionnée pour la cellule, en fonction de la valeur de la cellule définie par rapport aux valeurs minimale et maximale de la colonne.

```
requests
| summarize Mean = avg(duration), (Median, p80, p95, p99) = percentiles(duration, 50, 80, 95, 99), Requests = count() by name
| order by Requests desc
```

[![Capture d’écran d’une grille de journal avec une carte thermique affichant une mise à l’échelle partagée sur plusieurs colonnes](./media/workbooks-grid-visualizations/log-chart-grid-shared-scale.png)](./media/workbooks-grid-visualizations/log-chart-grid-shared-scale.png#lightbox)

Dans l’exemple ci-dessus, une palette (vert ou rouge) et une échelle partagées sont utilisées pour colorer les colonnes (moyen, médian, P80, P95 et P99). Une palette distincte (bleu) est utilisée pour la colonne de requête.

#### <a name="shared-scale"></a>Échelle partagée

Pour obtenir une échelle partagée :

1. Utilisez des expressions régulières pour sélectionner plusieurs colonnes auxquelles appliquer un paramètre. Par exemple, définissez le nom de la colonne sur `Mean|Median|p80|p95|p99` pour les sélectionner toutes.
2. Supprimez les paramètres par défaut des colonnes individuelles.

De cette façon, le nouveau paramètre commun à plusieurs colonnes devra appliquer son paramétrage pour inclure une échelle partagée.

[![Capture d’écran d’un paramètre de grille de journal permettant d’obtenir une échelle partagée sur plusieurs colonnes](./media/workbooks-grid-visualizations/log-chart-grid-shared-scale-settings.png)](./media/workbooks-grid-visualizations/log-chart-grid-shared-scale-settings.png#lightbox)

### <a name="icons-to-represent-status"></a>Icônes pour représenter l’état

L’exemple ci-dessous montre l’état personnalisé des requêtes en fonction de la durée de p95.

```
requests
| summarize p95 = percentile(duration, 95) by name
| order by p95 desc
| project Status = case(p95 > 5000, 'critical', p95 > 1000, 'error', 'success'), name, p95
```

[![Capture d’écran d’une grille de journal avec une carte thermique présentant une échelle partagée sur plusieurs colonnes à l’aide de la requête ci-dessus.](./media/workbooks-grid-visualizations/log-chart-grid-icons.png)](./media/workbooks-grid-visualizations/log-chart-grid-icons.png#lightbox)

Les noms d’icône pris en charge sont les suivants : `cancelled`, `critical`, `disabled`, `error`, `failed`, `info`, `none`, `pending`, `stopped`, `question`, `success`, `unknown`, `warning` `uninitialized`, `resource`, `up`, `down`, `left`, `right`, `trendup`, `trenddown`, `4`, `3`, `2`, `1`, `Sev0`, `Sev1`, `Sev2`, `Sev3`, `Sev4`, `Fired`, `Resolved`, `Available`, `Unavailable`, `Degraded`, `Unknown` et `Blank`.

### <a name="using-thresholds-with-links"></a>Utilisation de seuils avec les liens

Les instructions ci-dessous expliquent comment utiliser les seuils avec les liens pour affecter des icônes et ouvrir différents classeurs. Chaque lien de la grille ouvre un modèle de classeur différent pour cette ressource Application Insights.

1. Basculez le classeur en mode d’édition par la sélection de l’élément *Modifier* de la barre d’outils.
2. Sélectionnez **Ajouter**, puis *Ajouter une requête*.
3. Changez la *Source de données* pour « JSON », et *Visualisation* pour « Grille ».
4. Entrez la requête suivante.

```json
[ 
    { "name": "warning", "link": "Community-Workbooks/Performance/Performance Counter Analysis" },
    { "name": "info", "link": "Community-Workbooks/Performance/Performance Insights" },
    { "name": "error", "link": "Community-Workbooks/Performance/Apdex" }
]
```

5. Exécutez la requête.
6. Sélectionnez **Paramètres de colonne** pour ouvrir les paramètres.
7. Sélectionnez « nom » dans *Colonnes*.
8. Sous *Convertisseur de colonne*, choisissez « Seuils ».
9.  Entrez et choisissez les *Paramètres de seuil* suivants.
   
    | Opérateur | Valeur   | Icônes   |
    |----------|---------|---------|
    | ==       | warning | Avertissement |
    | ==       | error   | Failed  |

    ![Capture d’écran de l’onglet Modifier les paramètres de colonne, avec les paramètres ci-dessus.](./media/workbooks-grid-visualizations/column-settings.png)

    Conservez la ligne par défaut telle quelle. Vous pouvez indiquer le texte que vous voulez. La colonne Texte prend un format de chaîne en tant qu’entrée ; elle est renseignée avec la valeur et l’unité des colonnes Valeur et Unité si elles sont spécifiées. Par exemple, si « avertissement » est la valeur de la colonne, le texte peut être « Lien {0} {1} ! » ; il sera affiché sous la forme « lien avertissement ! ».
10. Sélectionnez la case à cocher *Convertir cet élément en lien*.
    1. Sous *Vue à ouvrir*, choisissez « Classeur (Modèle) ».
    2. Sous *La valeur du lien vient de*, choisissez « lien ».
    3. Sélectionnez l’option *Ouvrir le lien dans le panneau de contexte*.
    4. Choisissez les paramètres suivants dans *Paramètres de lien du classeur*.
        1. Sous *L’ID de modèle provient de*, choisissez « Colonne ».
        2. Sous *Colonne*, choisissez « lien ».

    ![Capture d’écran des paramètres de lien avec les paramètres ci-dessus.](./media/workbooks-grid-visualizations/make-this-item-a-link.png)

11. Sélectionnez « lien » dans *Colonnes*. Sous Paramètres, à côté de *Convertisseur de colonne*, sélectionnez **(Masquer la colonne)** .
1. Pour modifier le nom complet de la colonne « nom », sélectionnez l’onglet **Étiquettes**. Sur la ligne avec le « nom » en tant que son *ID de colonne*, sous *Étiquette de colonne, entrez le nom que vous souhaitez afficher.
2. Sélectionnez **Appliquer**

![Capture d’écran des seuils dans la grille avec les paramètres ci-dessus](./media/workbooks-grid-visualizations/thresholds-workbooks-links.png)

## <a name="fractional-units-percentages"></a>Pourcentages d’unités fractionnaires

L’unité fractionnaire (fr) est une unité de mesure dynamique couramment utilisée dans divers types de grilles. En cas de modification de la taille/résolution de la fenêtre, la largeur fr change également.

La capture d’écran ci-dessous montre un tableau à huit colonnes, d’égale largeur et faisant chacune 1fr de large. Dès lors que la taille de la fenêtre change, la largeur de chaque colonne se modifie proportionnellement.

[![Capture d’écran des colonnes d’une grille affichant chacune une valeur de largeur de colonne de 1fr](./media/workbooks-grid-visualizations/custom-column-width-fr.png)](./media/workbooks-grid-visualizations/custom-column-width-fr.png#lightbox)

L’image ci-dessous montre le même tableau, à l’exception de la première colonne dont la largeur est définie sur 50 %. Ce choix définit de manière dynamique la largeur de la colonne à la moitié de la largeur totale de la grille. Le redimensionnement de la fenêtre conservera la largeur de 50 %, sauf si la taille de la fenêtre devient trop petite. Ces colonnes dynamiques présentent une largeur minimale en fonction de leur contenu. Les 50% restants de la grille sont répartis entre les huit unités fractionnaires totales. La colonne « Type » ci-dessous est définie sur 2fr, si bien qu’elle prend un quart de l’espace restant. Comme la largeur des autres colonne est de 1fr pour chacune, elles prennent chacune un huitième de la moitié droite de la grille.

[![Capture d’écran de colonnes dans une grille, avec une valeur de largeur de colonne à 50 %, et le reste des colonnes à 1fr chacune](./media/workbooks-grid-visualizations/custom-column-width-fr2.png)](./media/workbooks-grid-visualizations/custom-column-width-fr2.png#lightbox)

La combinaison des largeurs fr, %, px et ch est possible et fonctionne de la même façon que les exemples précédents. Les largeurs définies par les unités statiques (ch et px) sont des constantes fixes qui ne changent pas, même si la fenêtre/la résolution est modifiée. Les colonnes définies par % prennent leur pourcentage en fonction de la largeur totale de la grille (qui peut ne pas être exacte en raison des largeurs minimales précédentes). Les colonnes définies avec la valeur fr fractionnent simplement l’espace restant de la grille, en fonction du nombre d’unités fractionnaires qui leur est allouée.

[![Capture d’écran de colonnes dans une grille, avec un assortiment de différentes unités de largeur utilisées](./media/workbooks-grid-visualizations/custom-column-width-fr3.png)](./media/workbooks-grid-visualizations/custom-column-width-fr3.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment créer une [arborescence dans les classeurs](workbooks-tree-visualizations.md).
* Découvrez comment créer des [paramètres de texte de classeur](workbooks-text.md).