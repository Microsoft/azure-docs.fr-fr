---
title: Génération de rapports sur vos données à l’aide de Purview Insights
description: Ce guide pratique explique comment afficher et utiliser les rapports Purview d’insights sur les glossaires en lien avec vos données.
author: SunetraVirdi
ms.author: suvirdi
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/20/2020
ms.openlocfilehash: f61d99a61cb50886d70489b586d948bfa751e196
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576772"
---
# <a name="glossary-insights-on-your-data-in-azure-purview"></a>Insights sur les glossaires de vos données dans Azure Purview

Ce guide pratique explique comment accéder aux rapports d’insights relatifs aux glossaires Azure Purview en lien avec vos données, ainsi que comment les afficher et les filtrer.

Dans ce guide pratique, vous allez apprendre à effectuer les opérations suivantes :

> [!div class="checklist"]
> - Accéder à des insights à partir de votre compte Purview
> - Obtenir une vue globale de vos données

## <a name="prerequisites"></a>Prérequis

Avant de commencer à utiliser les insights Purview, assurez-vous d’avoir effectué les étapes suivantes :

- Configurer vos ressources Azure et renseigner le compte avec des données

- Configurer et effectuer une analyse sur le type de source

- Configurer un glossaire et attacher des ressources aux termes de glossaire

Pour plus d’informations, consultez [Gérer des sources de données dans Azure Purview (préversion)](manage-data-sources.md).

## <a name="use-purview-glossary-insights"></a>Utiliser des insights relatifs aux glossaires Purview

Dans Azure Purview, vous pouvez créer des termes de glossaire et les attacher à des ressources. Plus tard, vous pouvez afficher la répartition du glossaire dans les insights relatifs au glossaire. Cela vous indique l’état de votre glossaire par des termes attachés aux ressources. Cela vous indique également les termes par état et par distribution des rôles en fonction du nombre d’utilisateurs.

**Pour afficher des insights relatifs aux glossaires** :

1. Accédez à l’écran de l’instance **Azure Purview** [dans le Portail Azure](https://aka.ms/purviewportal) et sélectionnez votre compte Purview.

1. Sur la page **Vue d’ensemble**, dans la section **Démarrer**, sélectionnez la vignette **Lancer le compte Purview**.

   :::image type="content" source="./media/glossary-insights/portal-access.png" alt-text="Lancer Purview à partir du Portail Azure":::

1. Dans la page d’**accueil** de Purview, sélectionnez la vignette **Afficher les insights** pour accéder à la zone **Insights** :::image type="icon" source="media/glossary-insights/ico-insights.png" border="false":::.

   :::image type="content" source="./media/glossary-insights/view-insights.png" alt-text="Afficher vos insights dans le Portail Azure":::

1. Dans la zone **Insights**:::image type="icon" source="media/glossary-insights/ico-insights.png" border="false":::, sélectionnez **Glossaire** pour afficher le rapport Purview **Insights relatifs aux glossaires**.

La page **Insights relatifs aux glossaires** affiche les zones suivantes :
1. **KPI de haut niveau** pour afficher les termes du glossaire et les utilisateurs du catalogue

2. **Principaux termes du glossaire et nombre de ressources** affiche les 5 premiers termes du glossaire avec les ressources qui y sont attachées. Toutes les autres ressources sont représentées dans la catégorie « Autre » du graphique.

3. **Termes de glossaire par état** affiche la répartition des termes de glossaire par état, par exemple « Brouillon », « Approuvé », « Alerte » et « Expiré ». 

1. Pointez ou cliquez sur la tranche du graphique avec un état et notez le nombre de termes avec cet état.

1. **Distribution des rôles par nombre d’utilisateurs** illustre la distribution des rôles en fonction du nombre d’utilisateurs par rôle dans Purview.

   :::image type="content" source="./media/glossary-insights/glossary-insights1.png" alt-text="Afficher des insights sur les glossaires":::

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les rapports d’insight Azure Purview via les [Insights relatifs aux ressources](./asset-insights.md)