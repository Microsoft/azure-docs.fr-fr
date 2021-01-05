---
title: Visualisations d’arborescences de classeur Azure Monitor
description: Découvrez toutes les visualisations d’arborescences de classeur Azure Monitor.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 09/04/2020
ms.author: lagayhar
ms.openlocfilehash: a815f4dcf7c76d4c11b4f2e227802b15f9c92a54
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89663128"
---
# <a name="tree-visualizations"></a>Visualisations d’arborescences

Les classeurs prennent en charge les vues hiérarchiques via des grilles d’arborescence. Les arborescences permettent de développer certaines lignes au niveau suivant pour une expérience d’exploration.

L’exemple ci-dessous montre les mesures d’intégrité du conteneur (taille de la plage de travail) visualisées comme une grille arborescente. Les nœuds de niveau supérieur sont ici des nœuds Azure Kubernetes Service (AKS), le niveau suivant étant les pods et le niveau final étant les conteneurs. Notez que vous pouvez toujours mettre en forme vos colonnes comme dans une grille (carte thermique, icônes, lien). Dans ce cas, la source de données sous-jacente est un espace de travail Log Analytics avec des journaux AKS.

[![Capture d’écran de l’affichage en mosaïque du résumé](./media/workbooks-tree-visualizations/trees.png)](./media/workbooks-tree-visualizations/trees.png#lightbox)

## <a name="adding-a-tree-grid"></a>Ajout d’une grille arborescente
1. Basculez le classeur en mode d’édition en cliquant sur l’élément _Modifier_ de la barre d’outils.
2. Sélectionnez **Ajouter**, puis *Ajouter une requête* pour ajouter un contrôle de requête de journal au classeur.
3. Sélectionnez le type de requête **Journal**, le type de ressource (par exemple, Application Insights) et les ressources à cibler.
4. Utiliser l’éditeur de requête pour saisir le KQL pour votre analyse
    ```kusto
    requests
    | summarize Requests = count() by ParentId = appName, Id = name
    | extend Kind = 'Request', Name = strcat('🌐 ', Id)
    | union (requests
    | summarize Requests = count() by Id = appName
    | extend Kind = 'Request', ParentId = '', Name = strcat('📱 ', Id))
    | project Name, Kind, Requests, Id, ParentId
    | order by Requests desc
    ```
5. Définir la visualisation sur **Grille**
6. Sélectionnez le bouton **Paramètres de colonne** pour ouvrir le volet Paramètres.
7. Dans la section **Arborescence/Regrouper par paramètres** en bas, définissez :
    * Type d'arborescence : `Parent/Child`
    * Champ d'ID : `Id`
    * Champ d'ID parent : `ParentId`
    * Afficher le contrôle pour développer sur : `Name`
    * Sélectionnez la case à cocher *Développer le niveau supérieur de l’arborescence*.
8. Dans la section _Colonnes_ en haut, définissez :
    * _Id_ - Convertisseur de colonne : `Hidden`
    * _ID parent_ - Renderer de colonne : `Hidden`
    * _Requêtes_ - Renderer de colonne : `Bar`, Couleur : `Blue`, Valeur minimale : `0`
9. Sélectionnez le bouton **Enregistrer et fermer** au bas du volet.

[![Screenshot of tile summary view with the above query and settings.](./media/workbooks-tree-visualizations/tree-settings.png)](./media/workbooks-tree-visualizations/tree-settings.png#lightbox)

## <a name="tree-settings"></a>Paramètres d’arborescence

| Paramètre | Explication |
|:------------- |:-------------|
| `Id Field` | ID unique de chaque ligne dans la grille. |
| `Parent Id Field` | ID du parent de la ligne actuelle. |
| `Show the expander on` | Colonne sur laquelle afficher l’icône de développement d’arborescence. Il est courant que les grilles d’arborescence masquent leur ID et le champ d’ID parent, car ils ne sont pas très lisibles. Au lieu de cela, l’icône de développement apparaît sur un champ avec une valeur plus lisible, comme le nom de l’entité. |
| `Expand the top level of the tree` | Si cette option est activée, la grille de l’arborescence est développée au niveau supérieur. Utile si vous souhaitez afficher plus d’informations par défaut. |

## <a name="grouping-in-a-grid"></a>Regroupement dans une grille

Le regroupement vous permet de générer des vues hiérarchiques similaires à celles ci-dessus avec des requêtes beaucoup plus simples. Vous perdez l’agrégation sur les nœuds internes de l’arborescence, mais cela est acceptable dans certains scénarios. Utilisez *Regrouper par* pour générer des arborescences lorsque le jeu de résultats sous-jacent ne peut pas être transformé en forme libre appropriée, par exemple : données d’alerte, d’intégrité et de métriques.

## <a name="adding-a-tree-using-grouping"></a>Ajout d’une arborescence à l’aide d’un regroupement

1. Basculez le classeur en mode d’édition en cliquant sur l’élément _Modifier_ de la barre d’outils.
2. Sélectionnez **Ajouter**, puis *Ajouter une requête* pour ajouter un contrôle de requête de journal au classeur.
3. Sélectionnez le type de requête **Journal**, le type de ressource (par exemple, Application Insights) et les ressources à cibler.
4. Utiliser l’éditeur de requête pour saisir le KQL pour votre analyse
    ```kusto
    requests
    | summarize Requests = count() by App = appName, RequestName = name
    | order by Requests desc
    ```
1. Définissez la visualisation sur *Grille*.
2. Sélectionnez le bouton **Paramètres de colonne** pour ouvrir le volet Paramètres.
3. Dans la section *Arborescence/Regrouper par paramètres* en bas, définissez :
    * Type d'arborescence : `Group By`
    * Regrouper par : `App`
    * Puis par : `None`
    * Sélectionnez la case à cocher *Développer le niveau supérieur de l’arborescence*.
4. Dans la section *Colonnes* en haut, définissez :
    * *Application* - Renderer de colonne : `Hidden`
    * *Requêtes* - Renderer de colonne : `Bar`, Couleur : `Blue`, Valeur minimale : `0`
5. Sélectionnez le bouton **Enregistrer et fermer** au bas du volet.

[![Screenshot showing the creation of a tree visualization in workbooks](./media/workbooks-tree-visualizations/tree-group-create.png)](./media/workbooks-tree-visualizations/tree-group-create.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment créer un [graphique dans les classeurs](workbooks-graph-visualizations.md).
* Découvrez comment créer une [vignette dans les classeurs](workbooks-tile-visualizations.md).
