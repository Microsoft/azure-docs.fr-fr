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
ms.openlocfilehash: a538eaf06013dcce6529c91816b419159a2600a1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89663183"
---
# <a name="graph-visualizations"></a>Visualisations de graphiques

Les classeurs prennent en charge la visualisation de graphes arbitraires basés sur les données des journaux pour afficher les relations entre les entités d’analyse.

Le graphique ci-dessous montre les données entrantes/sortantes d’un ordinateur via différents ports vers/depuis des ordinateurs externes. Il est coloré par type (ordinateur vs. port vs. adresse IP externe) et les tailles des bords correspondent à la quantité de données circulant entre. Les données sous-jacentes proviennent d’une requête KQL ciblant les connexions de machines virtuelles.

[![Capture d’écran de l’affichage en mosaïque du résumé](./media/workbooks-graph-visualizations/graph.png)](./media/workbooks-graph-visualizations/graph.png#lightbox)

## <a name="adding-a-graph"></a>Ajout d’un graphique
1. Basculez le classeur en mode d’édition en cliquant sur l’élément _Modifier_ de la barre d’outils.
2. Utilisez le lien **Ajouter une requête** pour ajouter un contrôle de requête de journal au classeur.
3. Sélectionnez le type de requête **Journal**, le type de ressource (par exemple, Application Insights) et les ressources à cibler.
4. Utilisez l’éditeur de requête pour saisir le KQL pour votre analyse.

    ```kusto
    let data = dependencies
    | summarize Calls = count() by App = appName, Request = operation_Name, Dependency = name
    | extend RequestId = strcat(App, '::', Request);
    let links = data
    | summarize Calls = sum(Calls) by App, RequestId
    | project SourceId = App, TargetId = RequestId, Calls, Kind = 'App -> Request'
    | union (data
        | project SourceId = RequestId, TargetId = Dependency, Calls, Kind = 'Request -> Dependency');
    let nodes = data
    | summarize Calls = sum(Calls) by App
    | project Id = App, Name = App, Calls, Kind = 'App'
    | union (data
        | summarize Calls = sum(Calls) by RequestId, Request
        | project Id = RequestId, Name = Request, Calls, Kind = 'Request')
    | union (data
        | summarize Calls = sum(Calls) by Dependency
        | project Id = Dependency, Name = Dependency, Calls, Kind = 'Dependency');
    nodes
    | union (links)
    ```

5. Définissez la visualisation sur **Graphe**
6. Sélectionnez le bouton **Paramètres de graphique** pour ouvrir le volet Paramètres.
7. Dans _Champs de disposition_ en bas, définissez :
    * ID de nœud : `Id`
    * ID source : `SourceId`
    * ID cible : `TargetId`
    * Étiquette de l’arête : `None`
    * Taille de l’arête : `Calls`
    * Taille du nœud : `None`
    * Type de coloration : `Categorical`
    * Champ Couleur du nœud : `Kind`
    * Palette de couleurs : `Pastel`
8. Dans _Paramètres de format de nœud_ en haut, définissez :
    * _Contenu en haut_- Utiliser la colonne : `Name`, Renderer de colonne : `Text`
    * _Contenu au centre_- Utiliser la colonne : `Calls`, Renderer de colonne : `Big Number`, Palette de couleurs : `None`
    * _Contenu en bas_- Utiliser la colonne : `Kind`, Renderer de colonne : `Text`
9. Sélectionnez le bouton _Enregistrer et fermer_ au bas du volet.

[![Screenshot of tile summary view with the above query and settings.](./media/workbooks-graph-visualizations/graph-settings.png)](./media/workbooks-graph-visualizations/graph-settings.png#lightbox)

## <a name="graph-settings"></a>Paramètres du graphique

| Paramètre         | Explication                                                                                                        |
|:----------------|:-------------------------------------------------------------------------------------------------------------------|
| `Node Id`       | Sélectionne une colonne qui fournit l’ID unique des nœuds sur le graphique. La valeur de la colonne peut être une chaîne ou un nombre. |
| `Source Id`     | Sélectionne une colonne qui fournit les ID des nœuds sources pour les arêtes sur le graphique. Les valeurs doivent correspondre à une valeur dans la colonne _ID de nœud_. |
| `Target Id`     | Sélectionne une colonne qui fournit les ID des nœuds cibles pour les arêtes sur le graphique. Les valeurs doivent correspondre à une valeur dans la colonne _ID de nœud_. |
| `Edge Label`    | Sélectionne une colonne qui fournit des étiquettes d’arête sur le graphique.                                                            |
| `Edge Size`     | Sélectionne une colonne qui fournit la mesure sur laquelle les largeurs d’arête seront basées.                                |
| `Node Size`     | Sélectionne une colonne qui fournit la mesure sur laquelle les zones de nœud seront basées.                                 |
| `Coloring Type` | Utilisé pour choisir le modèle de coloration des nœuds.                                                                            |

## <a name="node-coloring-types"></a>Types de coloration des nœuds

| Type de coloration | Explication |
|:------------- |:------------|
| `None`        | Tous les nœuds ont la même couleur. |
| `Categorical` | Les couleurs sont attribuées aux nœuds en fonction de la valeur ou de la catégorie d’une colonne dans le jeu de résultats. Dans l’exemple ci-dessus, la coloration est basée sur la colonne _Kind_ du jeu de résultats. Les palettes prises en charge sont `Default`, `Pastel` et `Cool tone`.  |
| `Field Based` | Dans ce type, une colonne fournit des valeurs RVB spécifiques à utiliser pour le nœud. Offre la plus grande flexibilité, mais nécessite généralement plus de travail pouvoir l’utiliser.  |

## <a name="node-format-settings"></a>Paramètres du format de nœud

Les auteurs de graphiques peuvent spécifier quel contenu va aux différentes parties d’un nœud : haut, gauche, centre, droit et bas. Les graphiques peuvent utiliser n’importe quel support des classeurs de renderers (texte, grand nombre, graphiques sparkline, icône, etc.).

## <a name="field-based-node-coloring"></a>Coloration des nœuds basés sur un champ

1. Basculez le classeur en mode d’édition en cliquant sur l’élément _Modifier_ de la barre d’outils.
2. Utilisez le lien **Ajouter une requête** pour ajouter un contrôle de requête de journal au classeur.
3. Sélectionnez le type de requête **Journal**, le type de ressource (par exemple, Application Insights) et les ressources à cibler.
4. Utilisez l’éditeur de requête pour saisir le KQL pour votre analyse.

     ```kusto
    let data = dependencies
    | summarize Calls = count() by App = appName, Request = operation_Name, Dependency = name
    | extend RequestId = strcat(App, '::', Request);
    let links = data
    | summarize Calls = sum(Calls) by App, RequestId
    | project SourceId = App, TargetId = RequestId, Calls, Kind = 'App -> Request'
    | union (data
        | project SourceId = RequestId, TargetId = Dependency, Calls, Kind = 'Request -> Dependency');
    let nodes = data
    | summarize Calls = sum(Calls) by App
    | project Id = App, Name = App, Calls, Color = 'FD7F23'
    | union (data
        | summarize Calls = sum(Calls) by RequestId, Request
        | project Id = RequestId, Name = Request, Calls, Color = 'B3DE8E')
    | union (data
        | summarize Calls = sum(Calls) by Dependency
        | project Id = Dependency, Name = Dependency, Calls, Color = 'C9B3D5');
    nodes
    | union (links)
    ```
5. Définissez la visualisation sur *Graphe*
6. Sélectionnez le bouton **Paramètres de graphe** pour ouvrir le volet Paramètres.
7. Dans *Champs de disposition* en bas, définissez :
    * ID de nœud : `Id`
    * ID source : `SourceId`
    * ID cible : `TargetId`
    * Étiquette de l’arête : `None`
    * Taille de l’arête : `Calls`
    * Taille du nœud : `Node`
    * Type de coloration : `Field Based`
    * Champ Couleur du nœud : `Color`
8. Dans *Paramètres de format de nœud* en haut, entrez ce qui suit.
    * Dans *Contenu en haut*, définissez :
        * Utiliser la colonne : `Name`
        * Renderer de colonne : `Text`
    * Dans *Contenu au centre*, définissez :
        * Utiliser la colonne : `Calls`
        * Renderer de colonne : `Big Number`
        * Palette de couleurs : `None`
    * Dans *Contenu en bas*, définissez :
        * Utiliser la colonne : `Kind`
        * Renderer de colonne : `Text`
9. Sélectionnez le bouton **Enregistrer et fermer** au bas du volet.

[![Screenshot showing the creation of a graph visualization with field base node coloring.](./media/workbooks-graph-visualizations/graph-field-based.png)](./media/workbooks-graph-visualizations/graph-field-based.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

* Les graphiques prennent également en charge le renderer de barre composite. Pour en savoir plus, consultez la [documentation sur la barre composite](workbooks-composite-bar.md).
* En savoir plus sur les [sources de données](workbooks-data-sources.md) que vous pouvez utiliser dans les classeurs.
