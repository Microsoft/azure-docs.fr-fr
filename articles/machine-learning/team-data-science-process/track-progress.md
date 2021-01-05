---
title: Suivre la progression des projets TDSP
description: Méthode de suivi de la progression d’un projet de science des données pour les responsables de groupe, chefs d’équipe et coordinateurs de projet spécialisés dans la science des données.
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 78a543fabadcc0d4e1766af1bc5c65aac0dadebe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91358923"
---
# <a name="track-the-progress-of-data-science-projects"></a>Suivre la progression de projets de science des données

Les responsables de groupe, chefs d’équipe et coordinateurs de projet de science des données peuvent suivre la progression de leurs projets.  Les responsables veulent savoir quel travail a été effectué, qui l’a fait et le travail restant.   La gestion des attentes est un facteur de succès important.

## <a name="azure-devops-dashboards"></a>Tableaux de bord DevOps Azure

Si vous utilisez Azure DevOps, vous pouvez générer des tableaux de bord pour suivre les activités et éléments de travail associés à un projet Agile donné. Pour plus d’informations sur les tableaux de bord, consultez [Tableaux de bord, rapports et widgets](/azure/devops/report/dashboards/).

Pour obtenir des instructions sur la création et la personnalisation de tableaux de bord et de widgets dans Azure DevOps, consultez les guides de démarrage rapide suivants :

- [Ajouter et gérer des tableaux de bord](/azure/devops/report/dashboards/dashboards)
- [Ajouter des widgets à un tableau de bord](/azure/devops/report/dashboards/add-widget-to-dashboard)

## <a name="example-dashboard"></a>Exemple de tableau de bord

Voici un exemple simple de tableau de bord qui effectue le suivi des activités des sprints d’un projet de science des données Agile, notamment le nombre de validations dans des dépôts associés. 

- La vignette **countdown** montre le nombre de jours restants dans le sprint actuel. 

- Les deux **vignettes de code** montrent le nombre de validations dans les deux dépôts de projet au cours des sept derniers jours. 

- **Work items for TDSP Customer Project** montre les résultats d’une requête pour tous les éléments de travail et leur état. 

- Un **diagramme de flux cumulatif** montre le nombre d’éléments de travail fermés et actifs.

- Le **graphique Burndown** montre le travail qu’il reste à faire par rapport au temps restant dans le sprint.

- Le **graphique Burnup** montre le travail accompli par rapport à la quantité totale de travail dans le sprint.

![Capture d'écran représentant un exemple de tableau de bord Azure DevOps.](./media/track-progress/dashboard.png)

## <a name="next-steps"></a>Étapes suivantes

[Guides d’application du processus TDSP (Team Data Science Process)](walkthroughs.md) répertorie les procédures pas à pas qui décrivent toutes les étapes de processus. Les scénarios liés expliquent comment gérer les ressources cloud et locales dans des applications intelligentes. 
