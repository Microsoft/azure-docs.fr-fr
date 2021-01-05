---
title: Rapports de classifications sur vos données à l’aide de Purview Insights (préversion)
description: Ce guide pratique explique comment afficher et utiliser les rapports de classifications Purview Insights sur vos données.
author: batamig
ms.author: bagol
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/24/2020
ms.openlocfilehash: bb3c7cc3f51eae90c5b712d224407e639b232fbc
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96938883"
---
# <a name="classification-insights-about-your-data-from-azure-purview"></a>Insights relatifs aux classifications de vos données à partir d’Azure Purview

Ce guide pratique explique comment accéder aux rapports Purview « Insights relatifs aux classifications » qui concernent vos données.

Les sources de données prises en charge sont les suivantes : Stockage Blob Azure, Azure Data Lake Storage (ADLS) GEN 1, Azure Data Lake Storage (ADLS) GEN 2, Azure Cosmos DB (API SQL), Azure Synapse Analytics (anciennement SQL DW), Azure SQL Database, Azure SQL Managed Instance, SQL Server.

Dans ce guide pratique, vous allez apprendre à effectuer les opérations suivantes :

> [!div class="checklist"]
> - Lancer votre compte Purview à partir d’Azure.
> - Afficher les insights relatifs aux classifications de vos données.
> - Explorer au niveau du détail pour plus d’informations sur les classifications de vos données.

## <a name="prerequisites"></a>Prérequis

Avant de commencer à utiliser les insights Purview, assurez-vous d’avoir effectué les étapes suivantes :

- Configurez vos ressources Azure et alimentez les comptes appropriés avec des données de test

- Configurez et terminez une analyse sur les données de test dans chaque source de données 

Pour plus d’informations, consultez [Gérer des sources de données dans Azure Purview (préversion)](manage-data-sources.md).

## <a name="use-purview-classification-insights"></a>Utiliser les insights Purview relatifs aux classifications

Dans Azure Purview, les classifications sont similaires aux étiquettes d’objet et permettent de marquer et d’identifier les données d’un type spécifique qui se trouvent dans votre patrimoine de données pendant l’analyse.

Purview utilise les mêmes types d’informations sensibles que Microsoft 365, ce qui vous permet d’étendre vos stratégies de sécurité existantes et la protection à l’ensemble de votre patrimoine de données.

> [!NOTE]
> Une fois que vous avez analysé vos types de sources, donnez quelques heures aux insights **Classification** pour qu’ils tiennent compte des nouvelles ressources.

**Pour afficher les insights relatifs aux classifications :**

1. Accédez à l’[écran de l’instance **Azure Purview** dans le portail Azure](https://aka.ms/purviewportal) et sélectionnez votre compte Purview.

1. Sur la page **Vue d’ensemble**, dans la section **Démarrer**, sélectionnez la vignette **Lancer le compte Purview**.

1. Dans Purview, sélectionnez l’élément de menu **Insights** :::image type="icon" source="media/insights/ico-insights.png" border="false"::: à gauche pour accéder à la zone **Insights**.

1. Dans la zone **Insights** :::image type="icon" source="media/insights/ico-insights.png" border="false":::, sélectionnez **Classification** pour afficher le rapport Purview **Insights relatifs aux classifications**.

   :::image type="content" source="./media/insights/select-classification-labeling-small.png" alt-text="Rapport des insights relatifs aux classifications" lightbox="media/insights/select-classification-labeling.png":::

   La page principale d’**Insights relatifs aux classifications** affiche les zones suivantes :

   |Domaine  |Description  |
   |---------|---------|
   |**Vue d’ensemble des sources avec classifications**     |Affiche les vignettes qui indiquent : <br>– le nombre d’abonnements trouvés dans vos données ; <br>– le nombre de classifications uniques trouvées dans vos données ; <br>– le nombre de sources classifiées trouvées ; <br>– le nombre de fichiers classifiés trouvés ; <br>– le nombre de tables classifiées trouvées.         |
   |**Principales sources avec données classifiées (30 derniers jours)**     |Affiche la tendance, au cours des 30 derniers jours, du nombre de sources trouvées avec des données classifiées.            |
   |**Principales catégories de classification par sources**     |Affiche le nombre de sources trouvées par catégorie de classification, par exemple **Finances** ou **Gouvernement**.      |
   |**Principales classifications des fichiers**     |Affiche les principales classifications appliquées aux fichiers de vos données, telles que les numéros de carte de crédit ou les numéros d’identification nationaux.         |
   |**Principales classifications des tables**     | Affiche les principales classifications appliquées aux tables de vos données, telles que les informations d’identification personnelles. |   
   |  **Activité Classification** <br>(fichiers et tables) |  Affiche des graphiques distincts pour les fichiers et les tables, chacun indiquant le nombre de fichiers ou de tables classifiés au cours de la période sélectionnée. <br>**Par défaut** : 30 jours<br>Sélectionnez le filtre **Période** au-dessus des graphiques pour sélectionner un autre intervalle de temps à afficher.    |
   |    |    |

## <a name="classification-insights-drilldown"></a>Exploration hiérarchique des insights relatifs aux classifications

Dans l’un des graphiques **Insights relatifs aux classifications** suivants, sélectionnez le lien **Afficher plus** pour l’explorer au niveau du détail :

- **Principales catégories de classification par sources**
- **Principales classifications des fichiers**
- **Principales classifications des tables**
- **Activité Classification > Données de classification**

Par exemple :

:::image type="content" source="media/insights/view-classifications-small.png" alt-text="Afficher toutes les classifications" lightbox="media/insights/view-classifications.png":::

Pour en savoir plus, effectuez l’une des actions suivantes :

|Option  |Description  |
|---------|---------|
|**Filtrer vos données**     |  Utilisez les filtres situés au-dessus de la grille pour filtrer les données affichées, notamment le nom de la classification, le nom de l’abonnement ou le type de source. <br><br>Si vous n’êtes pas sûr du nom exact de la classification, vous pouvez entrer tout ou partie du nom dans la zone **Filtrer par mot clé**.       |
|**Trier la grille** |Pour trier la grille, sélectionner l’en-tête de la colonne qui vous intéresse. | 
|**Modifier les colonnes**     |  Pour afficher plus ou moins de colonnes dans votre grille, sélectionnez **Modifier les colonnes** :::image type="icon" source="media/insights/ico-columns.png" border="false":::, puis sélectionnez les colonnes que vous souhaitez afficher ou dont vous voulez modifier l’ordre.   |
|**Explorer plus en détail**     | Pour explorer une classification spécifique au niveau du détail, sélectionnez un nom dans la colonne **Classification** pour afficher le rapport **Classification par source**. <br><br>Ce rapport affiche les données de la classification sélectionnée, notamment le nom de la source, le type de source, l’ID d’abonnement et le nombre de fichiers et de tables classifiés.      |
|**Parcourir les ressources**     |  Pour parcourir les ressources trouvées avec une classification ou une source spécifique, sélectionnez une classification ou une source (selon le rapport que vous consultez), puis sélectionnez **Parcourir les ressources** :::image type="icon" source="media/insights/ico-browse-assets.png" border="false"::: au-dessus des filtres. <br><br>Les résultats de la recherche affichent toutes les ressources classifiées trouvées pour le filtre sélectionné.  Pour plus d’informations, consultez [Rechercher dans le catalogue de données Azure Purview](how-to-search-catalog.md).       |
| | |

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les rapports d’insights Azure Purview
> [!div class="nextstepaction"]
> [Insights relatifs aux glossaires](glossary-insights.md)

> [!div class="nextstepaction"]
> [Insights relatifs aux analyses](scan-insights.md)

> [!div class="nextstepaction"]
> [Insights relatifs à l’étiquetage de confidentialité](./sensitivity-insights.md)

> [!div class="nextstepaction"]
> [Insights relatifs aux extensions de fichier](file-extension-insights.md)