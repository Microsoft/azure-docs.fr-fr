---
title: 'Guide de transition : du concepteur de vues Azure Monitor aux classeurs'
description: ''
author: austonli
ms.author: aul
ms.subservice: ''
ms.topic: conceptual
ms.date: 08/04/2020
ms.openlocfilehash: 3179daec361aa287b79c5e9a468908d0ccdd2982
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92143955"
---
# <a name="azure-monitor-view-designer-to-workbooks-transition-guide"></a>Guide de transition : du concepteur de vues Azure Monitor aux classeurs
Le [concepteur de vues](view-designer.md) est une fonctionnalité d'Azure Monitor qui vous permet de créer des vues personnalisées pour vous aider à visualiser les données de votre espace de travail Log Analytics, avec des graphiques, des listes et des chronologies. Ils ont migré vers les Classeurs pour fournir un canevas flexible pour l’analyse des données et la création de rapports visuels enrichis au sein du portail Azure. Cet article vous aide à effectuer la transition du Concepteur de vues vers les Classeurs. 


## <a name="workbooks-overview"></a>Présentation des classeurs
Les [classeurs](../insights/vminsights-workbooks.md) regroupent du texte, des  [requêtes de journal](/azure/data-explorer/kusto/query/), des métriques et des paramètres sous la forme de rapports interactifs complets. Les membres d'une équipe qui disposent du même accès aux ressources Azure peuvent également modifier des classeurs.

Les classeurs sont utiles pour les scénarios tels que les suivants :

-   Explorer l’utilisation de votre machine virtuelle quand vous ne connaissez pas à l’avance les métriques intéressantes : utilisation du processeur, espace disque, mémoire, dépendances réseau, etc. Contrairement à d’autres outils d’analyse de l’utilisation, les classeurs vous permettent de combiner plusieurs types de visualisations et d’analyses, ce qui les rend très utiles pour ce type d’exploration sous forme libre.
-   Expliquer à votre équipe le fonctionnement d’une machine virtuelle provisionnée récemment, en affichant les métriques des compteurs clés et d’autres événements de journal.
-   Partager les résultats d’une expérimentation de redimensionnement de votre machine virtuelle avec d’autres membres de votre équipe. Vous pouvez décrire les objectifs de l'expérimentation avec du texte, puis présenter les différentes métriques d'utilisation et les requêtes analytiques utilisées pour évaluer l'expérimentation, en vous aidant de légendes claires indiquant si chaque métrique se situe au-dessus ou en dessous de la cible.
-   Créer des rapports relatifs à l’impact d’une panne sur l’utilisation de votre machine virtuelle, en combinant des données, une explication du texte et une présentation des étapes suivantes pour éviter à l’avenir d’éventuelles interruptions.


## <a name="why-convert-view-designer-dashboards-to-workbooks"></a>Pourquoi convertir les tableaux de bord du concepteur de vues en classeurs ?

Le concepteur de vues permet de générer différentes vues et visualisations basées sur des requêtes. Toutefois, de nombreuses personnalisations générales restent limitées, telles que la mise en forme des grilles et les dispositions de vignettes, ou la sélection d’autres graphiques pour représenter vos données. Le concepteur de vues est limité à un total de neuf vignettes distinctes pour représenter vos données.

Les classeurs sont une plateforme qui révèle tout le potentiel de vos données. Non seulement les classeurs conservent toutes les fonctionnalités, mais ils prennent également en charge des fonctionnalités supplémentaires par le biais de texte, de métriques, de paramètres, et bien plus encore. Par exemple, les classeurs permettent aux utilisateurs de regrouper des grilles denses et d'ajouter des barres de recherche pour filtrer et analyser facilement les données. 

### <a name="advantages-of-using-workbooks-over-view-designer"></a>Avantages de l'utilisation de classeurs par rapport au concepteur de vues

* Prennent à la fois en charge les journaux et les métriques
* Permettent de bénéficier à la fois de vues personnelles pour le contrôle d'accès individuel et de vues de classeurs partagés
* Proposent des options de disposition personnalisées avec des onglets ainsi que des commandes de dimensionnement et de mise à l'échelle
* Permettent d'interroger plusieurs espaces de travail Log Analytics, applications Application Insights et abonnements
* Permettent d'utiliser des paramètres personnalisés qui mettent dynamiquement à jour les graphiques et visualisations associés
* Prennent en charge la galerie de modèles de l'instance publique de GitHub

Bien que ce guide propose des étapes simples pour recréer directement différentes vues couramment utilisées du concepteur de vues, les classeurs permettent aux utilisateurs de créer et de concevoir leurs propres visualisations et métriques personnalisées. La capture d'écran suivante, extraite du [modèle d'utilisation de l'espace de travail](https://go.microsoft.com/fwlink/?linkid=874159&resourceId=Azure%20Monitor&featureName=Workbooks&itemId=community-Workbooks%2FAzure%20Monitor%20-%20Workspaces%2FWorkspace%20Usage&workbookTemplateName=Workspace%20Usage&func=NavigateToPortalFeature&type=workbook), illustre ce que les classeurs sont capables de créer :


![Exemple d'application de classeurs](media/view-designer-conversion-overview/workbook-template-example.jpg)


## <a name="how-to-start-using-workbooks"></a>Commencer à utiliser des classeurs
Les classeurs ouverts à partir de la fonctionnalité Classeurs sont activés dans les espaces de travail Log Analytics et apparaissent sous forme d'éléments sur la barre de navigation latérale, sous l'emplacement du concepteur de vues.

![Navigation dans les classeurs](media/view-designer-conversion-overview/workbooks-nav.png)

Une fois la fonctionnalité sélectionnée, une galerie répertorie tous les classeurs et modèles enregistrés pour votre espace de travail.

![Galerie de classeurs](media/view-designer-conversion-overview/workbooks-gallery.png)

Pour lancer un nouveau classeur, vous pouvez sélectionner le modèle **Vide** sous **Démarrage rapide**, ou l'icône **Nouveau** de la barre de navigation supérieure. Pour afficher les modèles ou revenir aux classeurs enregistrés, sélectionnez l'élément dans la galerie ou recherchez le nom sur la barre de recherche.

Pour enregistrer un classeur, vous devrez enregistrer le rapport avec un titre, un abonnement, un groupe de ressources et un emplacement spécifiques.
Le classeur sera automatiquement renseigné avec les mêmes paramètres que l'espace de travail LA, avec le même abonnement et le même groupe de ressources, mais les utilisateurs pourront modifier ces paramètres de rapport. Par défaut, les classeurs sont enregistrés dans *Mes rapports* et l'utilisateur individuel est le seul à y avoir accès. Ils peuvent également être directement enregistrés dans des rapports partagés, ou faire l'objet de partages ultérieurs.

![Enregistrement des classeurs](media/view-designer-conversion-overview/workbooks-save.png)

## <a name="next-steps"></a>Étapes suivantes

- [Options de conversion](view-designer-conversion-options.md)