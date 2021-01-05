---
title: Insights sur les analyses de vos données dans Azure Purview
description: Ce guide pratique explique comment afficher et utiliser les rapports Purview d’insights sur les analyses en lien avec vos données.
author: SunetraVirdi
ms.author: suvirdi
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/20/2020
ms.openlocfilehash: 00f72e1de230cdc68f86010b7b25d86debaa5eb5
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575786"
---
# <a name="scan-insights-on-your-data-in-azure-purview"></a>Insights sur les analyses de vos données dans Azure Purview

Ce guide pratique explique comment accéder aux rapports d’insights sur les analyses Azure Purview en lien avec vos données, ainsi que comment les afficher et les filtrer.

Dans ce guide pratique, vous allez apprendre à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Afficher des insights à partir de votre compte Purview.
> * Obtenir une vue globale de vos analyses.

## <a name="prerequisites"></a>Prérequis

Avant de commencer à utiliser les insights Purview, assurez-vous d’avoir effectué les étapes suivantes :

* Configurez vos ressources Azure et renseignez le compte avec des données.
* Configurez et effectuez une analyse sur la source de données.

Pour plus d’informations, consultez [Gérer des sources de données dans Azure Purview (préversion)](manage-data-sources.md).

## <a name="use-purview-scan-insights"></a>Utiliser les insights sur les analyses dans Purview

Dans Azure Purview, vous pouvez inscrire et analyser les types de sources. Vous pouvez afficher l’état de l’analyse dans le temps dans les insights sur les analyses. Les insights vous indiquent le nombre d’analyses qui ont échoué, qui ont réussi ou qui ont été annulées pendant une période donnée.

### <a name="view-scan-insights"></a>Afficher des insights sur les analyses

1. Accédez à l’écran de l’instance **Azure Purview** dans le portail Azure et sélectionnez votre compte Purview.

1. Sur la page **Overview (Vue d’ensemble)** , dans la section **Get Started (Démarrer)** , sélectionnez la vignette **Open Purview Studio (Ouvrir Purview Studio)** .

   :::image type="content" source="./media/scan-insights/portal-access.png" alt-text="Lancer Purview à partir du portail Azure":::

1. Dans la page d’**accueil** de Purview, sélectionnez la vignette **View insights (Afficher les insights)** pour accéder à la zone **Insights** :::image type="icon" source="media/scan-insights/ico-insights.png" border="false":::.

   :::image type="content" source="./media/scan-insights/view-insights.png" alt-text="Afficher vos insights dans le portail Azure":::

1. Dans la zone **Insights** :::image type="icon" source="media/scan-insights/ico-insights.png" border="false"::: , sélectionnez **Scans (Analyses)** pour afficher le rapport Purview **Scan insights (Insights sur les analyses)** .

### <a name="view-high-level-kpis-to-show-count-of-scans-by-status"></a>Afficher les indicateurs de performance clés de haut niveau pour afficher le nombre d’analyses par état
 
Les indicateurs de performance clés affichent le nombre total d’analyses exécutées sur une période donnée. La période est définie par défaut sur les 30 derniers jours. Toutefois, vous pouvez également sélectionner les sept derniers jours. En fonction du filtre de temps, les valeurs d’indicateur de performance clé reflètent le nombre d’analyses de façon appropriée.


En fonction de la valeur de filtre de temps sélectionnée, vous pouvez voir la distribution des analyses qui ont réussi, qui ont échoué et qui ont été annulées par semaine ou par jour dans le graphe.

   :::image type="content" source="./media/scan-insights/scan-insights.png" alt-text="Afficher des insights sur les analyses":::

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les rapports Azure Purview avec les [insights relatifs aux ressources](./asset-insights.md)
