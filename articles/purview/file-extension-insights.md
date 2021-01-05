---
title: Génération de rapports d’extensions de fichier à l’aide de Purview Insights
description: Ce guide pratique explique comment afficher et utiliser les rapports d’extensions de fichier Purview Insights sur vos données.
author: batamig
ms.author: bagol
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/24/2020
ms.openlocfilehash: 222106f5838a1eb237987afaa93924f1efe4a1f4
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96746282"
---
# <a name="file-extension-insights-about-your-data-from-azure-purview"></a>Insights relatifs aux extensions de fichier de vos données à partir d’Azure Purview 

Ce guide pratique explique comment accéder à des insights relatifs aux extensions de fichier ou aux types de fichiers trouvés dans vos données, les visualiser et les filtrer.

Les sources de données prises en charge sont les suivantes : Stockage Blob Azure, Azure Data Lake Storage (ADLS) GEN 1, Azure Data Lake Storage (ADLS) GEN 2.

Dans ce guide pratique, vous allez apprendre à effectuer les opérations suivantes :
> [!div class="checklist"]
> * Lancer votre compte Purview à partir d’Azure. 
> - Afficher les insights relatifs aux extensions de fichier de vos données.
> - Explorer au niveau du détail pour plus d’informations sur les extensions de fichier de vos données.

## <a name="prerequisites"></a>Prérequis 

Avant de commencer à utiliser Purview Insights, assurez-vous d’avoir effectué les étapes suivantes :

- Configurez vos ressources Azure et alimentez les comptes appropriés avec des données de test

- Configurez et terminez une analyse sur les données de test dans chaque source de données

Pour plus d’informations, consultez [Gérer des sources de données dans Azure Purview (préversion)](manage-data-sources.md).

## <a name="use-purview-file-extension-insights"></a>Utiliser les insights Purview relatifs aux extensions de fichier

Lors de l’analyse de vos ressources, Azure Purview est en mesure de détecter les types de fichiers qui se trouvent dans votre patrimoine de données et de vous fournir plus de détails sur chaque type de fichier. Les détails incluent le nombre de fichiers de chaque type, l’emplacement de ces fichiers, et s’ils peuvent être analysés pour les données sensibles.

> [!NOTE]
> Une fois que vous avez analysé vos types de sources, donnez quelques heures aux insights **Extensions de fichier** pour qu’ils tiennent compte des nouvelles ressources.

**Pour afficher les insights relatifs aux extensions de fichier :**

1. Accédez à l’[écran de l’instance **Azure Purview** dans le portail Azure](https://aka.ms/purviewportal) et sélectionnez votre compte Purview.

1. Sur la page **Vue d’ensemble**, dans la section **Démarrer**, sélectionnez la vignette **Lancer le compte Purview**.

1. Dans Purview, sélectionnez l’élément de menu **Insights** :::image type="icon" source="media/insights/ico-insights.png" border="false"::: à gauche pour accéder à la zone **Insights**.
    
1. Dans **Insights**, sélectionnez l’onglet **Extensions de fichier**.

    Le rapport affiche un résumé du nombre d’extensions de fichiers uniques identifiées, dont les dix principales extensions sont représentées dans un graphique, sur la période sélectionnée (par défaut : 30 jours).

    :::image type="content" source="media/file-extension-insights/file-extension-overview-small.png" alt-text="Rapport d’extensions de fichier – Vue d’ensemble" lightbox="media/file-extension-insights/file-extension-overview.png":::

    Pour en savoir plus, effectuez l’une des actions suivantes :

    - Sélectionnez le sélecteur **Période** en haut du rapport pour modifier l’intervalle de temps pendant lequel les extensions de fichier ont été trouvées.
    
    - Sélectionnez **Afficher plus** sous le graphique pour afficher la liste complète des extensions de fichier trouvées. Pour plus d’informations, consultez [Exploration hiérarchique des insights relatifs aux extensions de fichier](#file-extension-insights-drilldown). 

### <a name="file-extension-insights-drilldown"></a>Exploration hiérarchique des insights relatifs aux extensions de fichier

Une fois que vous avez consulté les informations basiques sur les types de fichiers figurant dans votre patrimoine de données, descendez dans la hiérarchie pour connaître leur emplacement et savoir s’ils peuvent être analysés pour détecter des données sensibles.

Par exemple :

:::image type="content" source="media/file-extension-insights/file-extension-drilldown-small.png" alt-text="Rapport d’extensions de fichier – Exploration hiérarchique" lightbox="media/file-extension-insights/file-extension-drilldown.png":::

La grille montre les détails de chaque extension de fichier trouvée, notamment :

- **Nombre de fichiers**. Nombre de fichiers portant l’extension spécifiée.
- **Analyse du contenu**. Indique si l’extension de fichier est prise en charge pour l’analyse des données sensibles.
- **Abonnements**. Nombre d’abonnements dans lesquels l’extension spécifiée a été trouvée.
- **Sources**. Nombre de sources trouvées portant l’extension de fichier spécifiée.



Utilisez les filtres situés au-dessus de la grille pour filtrer les données affichées :

|Option  |Description  |
|---------|---------|
|**Filtrer par mot clé**     |    Entrez le texte dans la zone **Filtrer par mot clé** pour filtrer les types de fichiers par nom. Par exemple, pour afficher uniquement les fichiers PDF, entrez `PDF`.     |
|**Time**        | Sélectionnez cette option pour filtrer sur un intervalle de temps spécifique pendant lequel vos données ont été créées. <br>**Par défaut :** 30 jours  |
|**Extension de fichier**     |Sélectionnez cette option pour filtrer la grille selon un ou plusieurs types de fichiers.        |
|**Sources**    |Sélectionnez cette option pour filtrer la grille selon les sources de données spécifiées. |
|**Analyse du contenu**     |Sélectionnez **Pris en charge** ou **Non pris en charge** pour afficher uniquement les types de fichiers qui peuvent être analysés pour rechercher des données sensibles ou les données qui ne peuvent pas être analysées, telles que les fichiers **.cert** ou **.jpg**. |
| | |

Au-dessus des filtres, sélectionnez **Modifier les colonnes** :::image type="icon" source="media/insights/ico-columns.png" border="false"::: pour afficher plus ou moins de colonnes dans votre grille ou pour en modifier l’ordre. 

Pour trier la grille, cliquez sur l’en-tête de la colonne qui vous intéresse.
## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les rapports d’insights Azure Purview
> [!div class="nextstepaction"]
> [Insights relatifs aux glossaires](glossary-insights.md)

> [!div class="nextstepaction"]
> [Insights relatifs aux analyses](scan-insights.md)

> [!div class="nextstepaction"]
> [Insights relatifs aux classifications](./classification-insights.md)

> [!div class="nextstepaction"]
> [Insights relatifs aux étiquettes de confidentialité](sensitivity-insights.md)
