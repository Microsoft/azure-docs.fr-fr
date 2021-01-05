---
title: Créer des vues pour analyser les données de journal dans Azure Monitor | Microsoft Docs
description: Grâce au Concepteur de vues d’Azure Monitor, vous pouvez créer des vues personnalisées affichées dans le portail Azure qui contiennent différentes visualisations sur les données dans l’espace de travail Log Analytics. Cet article contient une présentation du Concepteur de vues et présente des procédures de création et de modification des vues personnalisées.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/04/2020
ms.openlocfilehash: f3ae081c8657a3ee6a27b0f9208c3eb4e4e745cc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87759731"
---
# <a name="create-custom-views-by-using-view-designer-in-azure-monitor"></a>Créer des vues personnalisées à l’aide du Concepteur de vues dans Azure Monitor
Grâce au Concepteur de vues d’Azure Monitor, vous pouvez créer plusieurs vues personnalisées dans le portail Azure qui peuvent vous aider à visualiser les données dans votre espace de travail Log Analytics. Cet article fournit une présentation du Concepteur de vues et des procédures de création et de modification des vues personnalisées.

> [!IMPORTANT]
> Les vues d'Azure Monitor ont été remplacées par des [classeurs](workbooks-overview.md) qui offrent des fonctionnalités supplémentaires. Pour plus d'informations sur la conversion de vos vues existantes en classeurs, consultez le [Guide de transition : du concepteur de vues Azure Monitor aux classeurs](view-designer-conversion-overview.md).
 


Pour plus d’informations sur le Concepteur de vues, consultez :

* [Référence de vignette](view-designer-tiles.md) : guide de référence pour les paramètres de chacune des vignettes disponibles dans vos vues personnalisées.
* [Référence des composants de visualisation](view-designer-parts.md) : guide de référence pour les paramètres des composants de visualisation disponibles dans vos vues personnalisées.


## <a name="concepts"></a>Concepts
Les vues sont affichées dans la page **Vue d’ensemble** d’Azure Monitor dans le portail Azure. Ouvrez cette page à partir du menu **Azure Monitor** en cliquant sur **Plus** sous la section **Insights**. Les vignettes de chaque vue personnalisée sont affichées par ordre alphabétique, et les vignettes pour les solutions de supervision sont installées dans le même espace de travail.

![Page Vue d’ensemble](media/view-designer/overview-page.png)

Les vues que vous créez avec le Concepteur de vues contiennent les éléments décrits dans le tableau suivant :

| Élément | Description |
|:--- |:--- |
| Vignettes | Sont affichées dans la page **Vue d’ensemble** d’Azure Monitor. Chaque vignette affiche une synthèse visuelle de la vue personnalisée qu’elle représente. Chaque type de vignette fournit une visualisation différente de vos enregistrements. Vous sélectionnez une vignette pour afficher une vue personnalisée. |
| Vue personnalisée | Affichée quand vous sélectionnez une vignette. Chaque vue contient un ou plusieurs composants de visualisation. |
| Composants de visualisation | Présente une visualisation de données dans l’espace de travail Log Analytics en fonction d’une ou plusieurs [requêtes de journal](../log-query/log-query-overview.md). La plupart des composants incluent un en-tête, qui fournit une visualisation d’ensemble, et une liste, qui montre les premiers résultats. Chaque type de composant produit différentes visualisations des enregistrements dans l’espace de travail Log Analytics. Vous sélectionnez des éléments dans le composant pour effectuer une requête de journal qui fournit des enregistrements détaillés. |

## <a name="required-permissions"></a>Autorisations requises
Vous avez au minimum besoin d’[autorisations de niveau contributeur](manage-access.md#manage-access-using-azure-permissions) dans l’espace de travail Log Analytics pour créer ou modifier des vues. Si vous n’avez pas cette autorisation, l’option Concepteur de vues n’apparaît pas dans le menu.


## <a name="work-with-an-existing-view"></a>Utiliser une vue existante
Les vues créées avec le Concepteur de vues affichent les options suivantes :

![Menu Vue d’ensemble](media/view-designer/overview-menu.png)

Les options sont décrites dans le tableau suivant :

| Option | Description |
|:--|:--|
| Actualiser   | Actualise la vue avec les données les plus récentes. | 
| Journaux d’activité      | Ouvre [Log Analytics](../log-query/log-query-overview.md) pour analyser des données avec des requêtes de journal. |
| Modifier       | Ouvre la vue dans le Concepteur de vues pour modifier son contenu et sa configuration.  |
| Clone      | Crée une vue et l’ouvre dans le Concepteur de vues. Le nom de la nouvelle vue est identique à celui de la vue d’origine, avec le mot *Copy* ajouté à la fin. |
| Plage de dates | Définissez un filtre de date et d’heure pour les données incluses dans la vue. Cette plage de dates est appliquée avant les plages de dates définies dans les requêtes dans la vue.  |
| +          | Définissez un filtre personnalisé qui est défini pour la vue. |


## <a name="create-a-new-view"></a>Créer une vue
Vous pouvez créer une nouvelle vue dans le Concepteur de vues en sélectionnant **Concepteur de vues** dans le menu de votre espace de travail Log Analytics.

![Vignette Concepteur de vues](media/view-designer/view-designer-tile.png)


## <a name="work-with-view-designer"></a>Utiliser le Concepteur de vues
Vous utilisez le Concepteur de vues pour créer des vues ou les modifier. 

Le Concepteur de vues comporte trois volets : 
* **Conception** : contient la vue personnalisée que vous créez ou modifiez. 
* **Contrôles** : contient les vignettes et les composants que vous ajoutez au volet **Conception**. 
* **Propriétés** : affiche les propriétés des vignettes ou des composants sélectionnés.

![Concepteur de vue](media/view-designer/view-designer-screenshot.png)

### <a name="configure-the-view-tile"></a>Configurer la vignette de vue
Un vue personnalisée ne peut avoir qu’une seule vignette. Pour afficher la vignette active ou en sélectionner une autre, dans le volet **Contrôle** sélectionnez l’onglet **Vignette**. Le volet **Propriétés** affiche les propriétés de la vignette active. 

Vous pouvez configurer les propriétés de la vignette conformément aux informations fournies dans [Référence de la vignette](view-designer-tiles.md), puis cliquer sur **Appliquer** pour enregistrer les modifications.

### <a name="configure-the-visualization-parts"></a>Configurer les composants de visualisation
Une vue peut inclure une nombre quelconque de composants de visualisation. Pour ajouter des composants à une vue, sélectionnez l’onglet **Vue**, puis sélectionnez un composant de visualisation. Le volet **Propriétés** affiche les propriétés du composant sélectionné. 

Vous pouvez configurer les propriétés de la vue conformément aux informations fournies dans la [Référence du composant de visualisation](view-designer-parts.md), puis cliquer sur **Appliquer** pour enregistrer les modifications.

Les vues ne comprennent qu’une seule ligne de composants de visualisation. Vous pouvez réorganiser les composants existants en les faisant glisser vers un nouvel emplacement.

Vous pouvez supprimer un composant de visualisation de la vue en sélectionnant le bouton **X** dans l’angle supérieur droit du composant.


### <a name="menu-options"></a>Options de menu
Les options pour l’utilisation des vues en mode d’édition sont décrites dans le tableau suivant.

![Menu Edition](media/view-designer/edit-menu.png)

| Option | Description |
|:--|:--|
| Enregistrer        | Enregistre les modifications et ferme la vue. |
| Annuler      | Ignore les modifications et ferme la vue. |
| Supprimer la vue | Supprime la vue. |
| Exporter      | Exporte la vue vers un [modèle Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md) que vous pouvez importer dans un autre espace de travail. Le nom du fichier est identique à celui de la vue, avec une extension *omsview*. |
| Importer      | Importe le fichier *omsview* que vous avez exporté à partir d’un autre espace de travail. Cette opération remplace la configuration de la vue existante. |
| Clone       | Crée une vue et l’ouvre dans le Concepteur de vues. Le nom de la nouvelle vue est identique à celui de la vue d’origine, avec le mot *Copy* ajouté à la fin. |

## <a name="next-steps"></a>Étapes suivantes
* Ajouter des [vignettes](view-designer-tiles.md) à votre vue personnalisée.
* Ajouter des [composants de visualisation](view-designer-parts.md) à votre vue personnalisée.
