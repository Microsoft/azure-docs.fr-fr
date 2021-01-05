---
title: Récapitulatif et accès pour la conversion du Concepteur de vues Azure Monitor en classeurs
description: ''
author: austonli
ms.author: aul
ms.subservice: ''
ms.topic: conceptual
ms.date: 02/07/2020
ms.openlocfilehash: 7614dd1d60bad5c124269ae2af02d30a5aacfe3e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90564030"
---
# <a name="view-designer-to-workbooks-conversion-summary-and-access"></a>Récapitulatif et accès pour la conversion du Concepteur de vues en classeurs
Le [concepteur de vues](view-designer.md) est une fonctionnalité d'Azure Monitor qui vous permet de créer des vues personnalisées pour vous aider à visualiser les données de votre espace de travail Log Analytics, avec des graphiques, des listes et des chronologies. Celles-ci sont progressivement supprimées et remplacées par des classeurs, qui offrent des fonctionnalités supplémentaires. Cet article explique en détail comment vous pouvez créer un récapitulatif de la vue d’ensemble et les autorisations nécessaires pour accéder aux classeurs.

## <a name="creating-your-workspace-summary-from-azure-dashboard"></a>Création du récapitulatif de votre espace de travail à partir du tableau de bord Azure
Les utilisateurs du Concepteur de vues peuvent avoir l’habitude d’une vignette de vue d’ensemble pour représenter un ensemble de vues. Pour conserver une vue d’ensemble comme le récapitulatif de l’espace de travail du Concepteur de vues, les classeurs offrent des étapes épinglées, qui peuvent être épinglées sur votre [tableau de bord du portail Azure](../../azure-portal/azure-portal-dashboards.md). Tout comme les vignettes de vue d’ensemble du récapitulatif de l’espace de travail, les éléments de classeur épinglés sont liés directement à la vue du classeur.

Vous pouvez tirer parti du haut niveau des fonctionnalités de personnalisation fournies avec les tableaux de bord Azure, qui permet l’actualisation automatique, le déplacement, le dimensionnement et un filtrage supplémentaire pour vos visualisations et vos éléments épinglés. 

![La capture d’écran présente un tableau de bord Azure personnalisé intitulé Résumé de l’espace de travail.](media/view-designer-conversion-access/dashboard.png)

Créez un tableau de bord Azure ou sélectionnez un tableau de bord existant pour commencer à épingler des éléments de classeur.

Pour épingler un élément individuel, vous devez activer l’icône d’épingle pour votre étape spécifique. Pour cela, sélectionnez le bouton **Modifier** correspondant pour votre étape, puis sélectionnez l’icône d’engrenage pour ouvrir **Paramètres avancés**. Sélectionnez l’option pour **Toujours afficher l’icône d’épingle à cette étape** : une icône d’épingle apparaît alors dans le coin supérieur droit de votre étape. Cette épingle vous permet d’épingler des visualisations spécifiques à votre tableau de bord, comme les vignettes de vue d’ensemble.

![Épingler une étape](media/view-designer-conversion-access/pin-step.png)


Vous pouvez également épingler plusieurs visualisations à partir du classeur ou l’intégralité du contenu du classeur à un tableau de bord. Pour épingler le classeur entier, sélectionnez **Modifier** dans la barre d’outils du haut pour passer en **Mode d’édition**. Une icône d’épingle apparaît, vous permettant d’épingler l’intégralité de l’élément Classeur, ou toutes les étapes et visualisations individuelles du classeur.

![Épingler tout](media/view-designer-conversion-access/pin-all.png)



## <a name="sharing-and-viewing-permissions"></a>Partage et visualisation des autorisations 
Les classeurs présentent l’avantage d’être des document privés ou partagés. Par défaut, les classeurs enregistrés sont enregistrés sous **Mes rapports**, ce qui signifie que seul le créateur peut visualiser ce classeur.

Vous pouvez partager vos classeurs en sélectionnant l’icône **Partager** dans la barre d’outils du haut quand vous êtes en **Mode d’édition**. Vous êtes invité à déplacer votre classeur vers **Rapports partagés**, ce qui génère un lien qui fournit un accès direct au classeur.

Pour qu’un utilisateur puisse visualiser un classeur partagé, il doit avoir accès à l’abonnement et au groupe de ressources sous lequel le classeur est enregistré.

![Accès en fonction de l’abonnement](media/view-designer-conversion-access/subscription-access.png)

## <a name="next-steps"></a>Étapes suivantes

- [Tâches courantes](view-designer-conversion-tasks.md)