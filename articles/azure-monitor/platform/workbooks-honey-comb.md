---
title: Visualisations en alvéoles des classeurs Azure Monitor
description: Découvrez les visualisations en alvéoles des classeurs Azure Monitor.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 09/18/2020
ms.author: lagayhar
ms.openlocfilehash: 859240623c5dd3009890cd2633ffb9cccf230080
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91339048"
---
# <a name="honey-comb-visualizations"></a>Visualisations en alvéoles

Les alvéoles offrent des vues à haute densité des métriques ou des catégories, qui peuvent éventuellement être regroupées en clusters. Elles permettent une identification visuelle et un approfondissement des zones réactives.

L'illustration ci-dessous représente l'utilisation du processeur des machines virtuelles sur deux abonnements. Chaque cellule représente une machine virtuelle, et la couleur/l'étiquette représente l'utilisation moyenne du processeur (les cellules rouges représentent les machines à chaud). Les machines virtuelles sont regroupées par abonnement.

[![Capture d'écran représentant l'utilisation du processeur des machines virtuelles sur deux abonnements](.\media\workbooks-honey-comb\cpu-example.png)](.\media\workbooks-honey-comb\cpu-example.png#lightbox)

## <a name="adding-a-honey-comb"></a>Ajouter d'une alvéole

1. Basculez le classeur en mode d’édition en cliquant sur l’élément Modifier de la barre d’outils.
2. Sélectionnez **Ajouter**, puis *Ajouter une requête* pour ajouter un contrôle de requête de journal au classeur.
3. Sélectionnez « Journaux » comme *Source de données*, « Log Analytics » comme *Type de ressource* et, pour *Ressource*, pointez vers un espace de travail contenant le journal des performances de la machine virtuelle.
4. Utilisez l'éditeur de requête afin d'entrer le KQL pour votre analyse.

```kusto
    Perf
| where CounterName == 'Available MBytes'
| summarize CounterValue = avg(CounterValue) by Computer, _ResourceId
| extend ResourceGroup = extract(@'/subscriptions/.+/resourcegroups/(.+)/providers/microsoft.compute/virtualmachines/.+', 1, _ResourceId)
| extend ResourceGroup = iff(ResourceGroup == '', 'On-premise computers', ResourceGroup), Id = strcat(_ResourceId, '::', Computer)
```

5. Exécutez la requête.
6. Définissez la *visualisation* sur « Graphique ».
7. Sélectionnez **Paramètres du graphe**.
    1. Dans *Champs de disposition* en bas, définissez :
        1. Type de graphique : **Clusters Hive**.
        2. ID nœud : `Id`.
        3. Regrouper par : `None`.
        4. Taille du nœud : 100.
        5. Marge entre les hexagones : `5`.
        6. Type de coloration : **Carte thermique**.
        7. Champ Couleur du nœud : `CouterValue`.
        8. Palette de couleurs : `Red to Green`.
        9. Valeur minimale : `100`.
        10. Valeur maximale : `2000`.
    2. Dans *Paramètres de format de nœud* en haut, définissez :
        1. Contenu en haut :
            1. Utiliser la colonne : `Computer`
            2. Renderer de colonne : `Text`.
        9. Contenu au centre :
            1. Utiliser la colonne : `CounterValue`
            2. Renderer de colonne : `Big Number`.
            3. Palette de couleurs : `None`.
            4. Cochez la case *Mise en forme personnalisée des nombres*.
            5. Unités : `Megabytes`.
            6. Nombre maximal de chiffres décimaux : `1`.
8. Sélectionnez le bouton **Enregistrer et fermer** au bas du volet.

[![Capture d'écran du contrôle de requête, des paramètres de graphique et des alvéoles avec la requête et les paramètres ci-dessus](.\media\workbooks-honey-comb\available-memory.png)](.\media\workbooks-honey-comb\available-memory.png#lightbox)

## <a name="honey-comb-layout-settings"></a>Paramètres de disposition en alvéoles

| Paramètre | Explication |
|:------------- |:-------------|
| `Node Id` | Sélectionne une colonne qui fournit l'ID unique des nœuds. La valeur de la colonne peut être une chaîne ou un nombre. |
| `Group By Field` | Sélectionne la colonne sur laquelle les nœuds doivent être regroupés. |
| `Node Size` | Définit la taille des cellules hexagonales. À utiliser avec la propriété `Margin between hexagons` pour personnaliser l'apparence du graphique en alvéoles. |
| `Margin between hexagons` | Définit l'écart entre les cellules hexagonales. À utiliser avec la propriété `Node size` pour personnaliser l'apparence du graphique en alvéoles. |
| `Coloring type` | Sélectionne le schéma à utiliser pour colorer le nœud. |
| `Node Color Field` | Sélectionne une colonne qui fournit la mesure sur laquelle les zones de nœud seront basées. |

## <a name="node-coloring-types"></a>Types de coloration des nœuds

| Type de coloration | Explication |
|:------------- |:-------------|
| `None` | Tous les nœuds ont la même couleur. |
| `Categorical` | Les couleurs sont attribuées aux nœuds en fonction de la valeur ou de la catégorie d’une colonne dans le jeu de résultats. Dans l’exemple ci-dessus, la coloration est basée sur la colonne _Kind_ du jeu de résultats. Les palettes prises en charge sont `Default`, `Pastel` et `Cool tone`.  |
| `Heatmap` | Dans ce type, les cellules sont colorées en fonction d'une colonne de métriques et d'une palette de couleurs. Il s'agit d'un moyen simple de mettre en évidence les métriques réparties entre les cellules. |
| `Thresholds` | Dans ce type, les couleurs des cellules sont définies par des règles de seuil (par exemple, _Processeur > 90 % => Rouge, 60 % > Processeur > 90 % => Jaune, Processeur < 60 % => Vert_) |
| `Field Based` | Dans ce type, une colonne fournit des valeurs RVB spécifiques à utiliser pour le nœud. Offre la plus grande flexibilité, mais nécessite généralement plus de travail pouvoir l’utiliser.  |
      
## <a name="node-format-settings"></a>Paramètres du format de nœud

Les auteurs des alvéoles peuvent spécifier quel contenu va aux différentes parties d'un nœud : haut, gauche, centre, droit et bas. Les auteurs sont libres d'utiliser n'importe quel support de classeurs de renderers (texte, grand nombre, graphiques sparkline, icône, etc.).

## <a name="next-steps"></a>Étapes suivantes

- Apprenez à créer un [Renderer de barre composite dans les classeurs](workbooks-composite-bar.md).
- Apprenez à [importer des données de journal Azure Monitor dans Power BI](powerbi.md).
