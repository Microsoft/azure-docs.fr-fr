---
title: Insights sur les ressources en lien avec vos données dans Azure Purview (préversion)
description: Ce guide pratique explique comment afficher et utiliser les rapports Purview sur les insights relatifs aux ressources en lien avec vos données.
author: SunetraVirdi
ms.author: suvirdi
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/20/2020
ms.openlocfilehash: ac93d835b8dbdd5a12d031825dcb879160df5e95
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575344"
---
# <a name="asset-insights-on-your-data-in-azure-purview"></a>Insights relatifs aux ressources en lien avec vos données dans Azure Purview

Ce guide pratique explique comment accéder aux rapports sur les insights relatifs aux ressources Azure Purview en lien avec vos données, ainsi que comment les afficher et les filtrer.

Dans ce guide pratique, vous allez apprendre à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Afficher des insights à partir de votre compte Purview.
> * Obtenir une vue globale de vos données.
> * Obtenir plus de détails sur le nombre de ressources.

## <a name="prerequisites"></a>Prérequis

Avant de commencer à utiliser les insights Purview, assurez-vous d’avoir effectué les étapes suivantes :

* Configurez vos ressources Azure et renseignez le compte avec des données.

* Configurez et effectuez une analyse sur le type de source.

Pour plus d’informations, consultez [Gérer des sources de données dans Azure Purview (préversion)](manage-data-sources.md).

## <a name="use-purview-asset-insights"></a>Utiliser les insights relatifs aux ressources dans Purview

Dans Azure Purview, vous pouvez inscrire et analyser les types de sources. Une fois l’analyse terminée, vous pouvez afficher la distribution des ressources dans la zone Insights relatifs aux ressources, qui vous indique l’état de votre patrimoine de données par classification et par jeux de ressources. Cette zone vous indique également s’il y a une modification de la taille des données.

> [!NOTE]
> Une fois que vous avez analysé vos types de sources, donnez une heure aux insights relatifs aux ressources pour qu’ils tiennent compte des nouvelles ressources.

1. Accédez à votre ressource Azure Purview dans le portail Azure.

1. Sur la page **Overview (Vue d’ensemble)** , dans la section **Get Started (Démarrer)** , sélectionnez la vignette **Open Purview Studio (Ouvrir Purview Studio)** .

   :::image type="content" source="./media/asset-insights/portal-access.png" alt-text="Lancer Purview à partir du portail Azure":::

1. Dans la page d’**accueil** de Purview, sélectionnez la vignette **View insights (Afficher les insights)** pour accéder à la zone **Insights** :::image type="icon" source="media/asset-insights/ico-insights.png" border="false":::.

   :::image type="content" source="./media/asset-insights/view-insights.png" alt-text="Afficher vos insights dans le portail Azure":::

1. Dans la zone **Insights** :::image type="icon" source="media/asset-insights/ico-insights.png" border="false":::, sélectionnez **Assets (Ressources)** pour afficher le rapport Purview **Asset insights (Insights relatifs aux ressources)** .

### <a name="view-asset-insights"></a>Afficher des insights relatifs aux ressources

1. La page principale **Insights relatifs aux ressources** affiche les zones suivantes :

2. Indicateurs de performance clés de haut niveau pour afficher les types de sources, les ressources classifiées et les ressources découvertes
 
3. Le premier graphe présente les **ressources par type de source**.

4. Affichez la distribution de vos ressources par type de source. Choisissez une classification ou une catégorie de classification entière pour afficher la distribution des ressources par type de source. 
 
5. Pour plus d’informations, sélectionnez l’option **Afficher plus**, qui affiche une forme tabulaire des types de sources et des décomptes de ressources. Les filtres de classification sont transmis à cette page.

   :::image type="content" source="./media/asset-insights/highlight-kpis.png" alt-text="Afficher les indicateurs de performance clés et le graphe dans les insights relatifs aux ressources":::
 
6. Sélectionnez une source spécifique pour laquelle vous souhaitez afficher les dossiers principaux avec les décomptes de ressources. 

   :::image type="content" source="./media/asset-insights/select-data-source.png" alt-text="Sélectionner le type de source":::
 
7. Sélectionnez le nombre total de ressources par rapport au dossier supérieur dans le type de source que vous avez sélectionné ci-dessus.

   :::image type="content" source="./media/asset-insights/file-path.png" alt-text="Afficher les chemins d’accès des fichiers":::

8. Affichez la liste des fichiers dans le dossier. Revenez aux insights à l’aide de la barre de navigation.

   :::image type="content" source="./media/asset-insights/list-page.png" alt-text="Afficher la liste des ressources":::  

### <a name="file-based-source-types"></a>Types de source basés sur des fichiers
Les deux graphes suivants dans les insights relatifs aux ressources montrent une distribution de types de source basés sur des fichiers. Le premier graphe appelé **Tendance de taille (Go) de type de fichier dans les types de sources** affiche les principales tendances de tailles de type de fichier au cours des 30 derniers jours. 
 
1. Choisissez votre type de source pour afficher le type de fichier dans la source. 
 
1. Sélectionnez **Afficher plus** pour afficher la taille actuelle des données, le changement de taille, le nombre de ressources actuelles et la modification du nombre de ressources.
 
   > [!NOTE]
   > Si l’analyse a été exécutée une seule fois au cours des 30 derniers jours ou si une modification du catalogue telle que l’ajout ou la suppression d’une classification s’est produite une seule fois en 30 jours, les informations sur les modifications ci-dessus sont vides.

1. Si vous cliquez sur le type de source, consultez les dossiers du haut avec les principaux changements de nombres de ressources.

1. Sélectionnez le chemin d’accès pour afficher la liste des ressources.

Le deuxième graphe des types de source basés sur des fichiers est intitulé **_Fichiers non associés à un jeu de ressources_**. Si vous vous attendez à ce que tous les fichiers soient regroupés dans un jeu de ressources, ce graphe peut vous aider à découvrir les ressources qui n’ont pas été regroupées. Les ressources manquantes peuvent indiquer la présence du mauvais modèle de fichier dans le dossier. Suivez les mêmes étapes que dans d’autres graphes pour afficher plus de détails sur les fichiers.

   :::image type="content" source="./media/asset-insights/file-based-assets.png" alt-text="Afficher les ressources basées sur des fichiers":::  

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les rapports Azure Purview avec les [insights relatifs aux analyses](./scan-insights.md)
