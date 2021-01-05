---
title: Guide de l’utilisateur sur la traçabilité Data Catalog (préversion)
description: Cet article présente la fonctionnalité de traçabilité de catalogue d’Azure Purview.
author: chanuengg
ms.author: csugunan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 11/29/2020
ms.openlocfilehash: a319dbce2502f35272cf9b70da2022f581d64275
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550285"
---
# <a name="azure-purview-data-catalog-lineage-user-guide"></a>Guide de l’utilisateur sur la traçabilité du Data Catalog dans Azure Purview

Cet article fournit une vue d’ensemble de la fonctionnalité de traçabilité des données dans Data Catalog dans Azure Purview.

## <a name="background"></a>Arrière-plan

L’une des fonctionnalités de plateforme d’Azure Purview permet d’afficher la traçabilité entre les jeux de données créés par les processus de données. Les systèmes comme Data Factory, Data Share et Power BI capturent la traçabilité des données au fur et à mesure qu’elles se déplacent. Les rapports de traçabilité personnalisés sont également pris en charge via les hooks Atlas et l’API REST.

## <a name="lineage-collection"></a>Collecte des données de traçabilité 
 Les métadonnées collectées dans Azure Purview à partir de systèmes de données d’entreprise sont regroupées pour afficher une traçabilité de données de bout en bout. Les systèmes de données qui collectent la traçabilité dans Purview sont classés de manière générale en trois types.

### <a name="data-processing-system"></a>Système de traitement des données
Les outils d’intégration de données et ETL peuvent envoyer les données de traçabilité dans Azure Purview au moment de l’exécution. Les outils tels que Data Factory, Data Share, Synapse, Azure Databricks, etc., appartiennent à cette catégorie de systèmes de données. Les systèmes de traitement des données référencent les jeux de données comme source à partir de différentes bases de données et solutions de stockage pour créer des jeux de données cibles. La liste des systèmes de traitement de données actuellement intégrés avec Purview pour la traçabilité est répertoriée dans le tableau ci-dessous.


| Système de traitement des données | Portée prise en charge |
| ---------------------- | ------------|
| Azure Data Factory | [Activité de copie](how-to-link-azure-data-factory.md#data-factory-copy-data-support) <br> [Activité de flux de données](how-to-link-azure-data-factory.md#data-factory-data-flow-support) <br> [Activité Exécuter le package SSIS](how-to-link-azure-data-factory.md#data-factory-execute-ssis-package-support) |
| Azure Data Share | [Partager l’instantané](how-to-link-azure-data-share.md) |
 
### <a name="data-storage-systems"></a>Systèmes de stockage de données
Les solutions de bases de données et de stockage, telles que SQL Server, Teradata et SAP, ont des moteurs de requête pour transformer des données à l’aide du langage de script. La traçabilité des données à partir de procédures stockées est collectée dans Purview et alignée avec la traçabilité d’autres systèmes.

| Système de stockage de données | Portée prise en charge |
| ---------------------- | ------------|
| Teradata | Procédures stockées

### <a name="data-analytics--reporting-systems"></a>Systèmes de création de rapports et d’analytique données
Les systèmes de données tels qu’Azure ML et Power BI signalent la traçabilité dans Azure Purview. Ces systèmes utilisent les jeux de données des systèmes de stockage et les traitent via leur méta-modèle pour créer des tableaux de bord BI, des expériences ML, etc.

| Système de création de rapports et d’analytique données | Portée prise en charge |
| ---------------------- | ------------|
| Power BI | [Jeux de données, flux de données, rapports et tableaux de bord](register-scan-power-bi-tenant.md)

## <a name="get-started-with-lineage"></a>Prise en main de la traçabilité

La traçabilité dans Purview inclut des jeux de données et des processus. Les jeux de données sont également appelés « nœuds », tandis que les processus peuvent également être appelés « périphéries » :

* **Jeu de données (nœud)**  : jeu de données (structuré ou non structuré) fourni comme entrée d’un processus. Par exemple, une table SQL, un objet blob Azure et des fichiers (tels que .csv et .xml) sont tous considérés comme des jeux de données. Dans la section Traçabilité de Purview, les jeux de données sont représentés par des zones rectangulaires.

* **Processus (Périphérie)**  : une activité ou une transformation effectuée sur un jeu de données est appelée processus. Par exemple, l’activité de copie ADF, l’instantané Data Share, etc. Dans la section Traçabilité de Purview, les processus sont représentés par des zones arrondies.

Pour accéder aux informations de traçabilité d’une ressource dans Purview, procédez comme suit :

1. Dans le portail Azure, accédez à la page [Azure Purview accounts (Comptes Azure Purview)](https://aka.ms/purviewportal).

1. Sélectionnez votre compte Azure Purview dans la liste, puis sélectionnez **Launch purview account (Lancer le compte Purview)** à partir de la page **Overview (Vue d’ensemble)** .

1. Dans la page d’**accueil** d’Azure Purview, recherchez le nom d’un jeu de données ou le nom d’un processus, par exemple Copie ADF ou Activité de flux de données. Appuyez ensuite sur Entrée.

1. Dans les résultats de la recherche, sélectionnez la ressource et sélectionnez son onglet **Traçabilité**.

   :::image type="content" source="./media/catalog-lineage-user-guide/select-lineage-from-asset.png" alt-text="Capture d’écran montrant comment sélectionner l’onglet Traçabilité." border="true":::

## <a name="asset-level-lineage"></a>Traçabilité au niveau de la ressource

Azure Purview prend en charge la traçabilité au niveau des ressources pour les jeux de données et les processus. Pour afficher la traçabilité au niveau de la ressource, accédez à l’onglet **Traçabilité** de la ressource actuelle dans le catalogue. Sélectionnez le nœud de ressource de jeu de données actif. Par défaut, la liste des colonnes appartenant aux données s’affiche dans le volet gauche.

   :::image type="content" source="./media/catalog-lineage-user-guide/view-columns-from-lineage.png" alt-text="Capture d’écran montrant comment sélectionner l’option d’affichage des colonnes dans la page Traçabilité" border="true":::

## <a name="column-level-lineage"></a>Traçabilité au niveau de la colonne

Azure Purview prend en charge la traçabilité au niveau dela colonne pour les jeux de données. Pour afficher la traçabilité au niveau de la colonne, accédez à l’onglet **Traçabilité** de la ressource actuelle dans le catalogue et procédez comme suit :

1. Une fois que vous êtes sous l’onglet Traçabilité, dans le volet gauche, activez la case à cocher en regard de chaque colonne que vous souhaitez afficher dans la traçabilité des données.

   :::image type="content" source="./media/catalog-lineage-user-guide/select-columns-to-show-in-lineage.png" alt-text="Capture d’écran montrant comment sélectionner des colonnes à afficher dans la page Traçabilité" lightbox="./media/catalog-lineage-user-guide/select-columns-to-show-in-lineage.png":::

1. Pointez sur une colonne sélectionnée dans le volet gauche ou dans le jeu de données du canevas de traçabilité pour voir le mappage de colonnes. Toutes les instances de colonne sont mises en surbrillance.

   :::image type="content" source="./media/catalog-lineage-user-guide/show-column-flow-in-lineage.png" alt-text="Capture d’écran montrant comment pointer sur un nom de colonne pour mettre en surbrillance le contenu d’une colonne dans une trajectoire de traçabilité des données." lightbox="./media/catalog-lineage-user-guide/show-column-flow-in-lineage.png":::

1. Si le nombre de colonnes est supérieur à ce qui peut être affiché dans le volet gauche, utilisez l’option de filtre pour sélectionner une colonne spécifique par nom. Vous pouvez également utiliser la souris pour faire défiler la liste.

   :::image type="content" source="./media/catalog-lineage-user-guide/filter-columns-by-name.png" alt-text="Capture d’écran montrant comment filtrer les colonnes par nom de colonne sur la page de traçabilité." lightbox="./media/catalog-lineage-user-guide/filter-columns-by-name.png":::

1. Si le canevas de traçabilité contient plus de nœuds et de périphéries, utilisez le filtre pour sélectionner la ressource de données ou les nœuds de processus par nom. Vous pouvez également utiliser votre souris pour naviguer dans la fenêtre de traçabilité.

   :::image type="content" source="./media/catalog-lineage-user-guide/filter-assets-by-name.png" alt-text="Capture d’écran montrant les nœuds de ressources de données par nom dans la page de traçabilité." lightbox="./media/catalog-lineage-user-guide/filter-assets-by-name.png":::

1. Utilisez le bouton bascule dans le volet gauche pour mettre en surbrillance la liste des jeux de données dans le canevas de traçabilité. Si vous désactivez le bouton bascule, toutes les ressources contenant au moins une des colonnes sélectionnées s’affichent. Si vous activez le bouton bascule, seuls les jeux de données qui contiennent toutes les colonnes s’affichent.

   :::image type="content" source="./media/catalog-lineage-user-guide/use-toggle-to-filter-nodes.png" alt-text="Capture d’écran montrant comment utiliser le bouton bascule pour filtrer la liste des nœuds sur la page de traçabilité." lightbox="./media/catalog-lineage-user-guide/use-toggle-to-filter-nodes.png":::

1. Sélectionnez **Switch to asset (Basculer vers la ressource)** sur n’importe quelle ressource pour afficher ses métadonnées à partir de la vue Traçabilité. Vous pouvez ainsi facilement accéder à une autre ressource dans le catalogue à partir de la vue Traçabilité.

   :::image type="content" source="./media/catalog-lineage-user-guide/select-switch-to-asset.png" alt-text="Capture d’écran montrant comment sélectionner l’option de basculement vers la ressource dans une ressource de données de traçabilité." lightbox="./media/catalog-lineage-user-guide/select-switch-to-asset.png":::

1. Le canevas de traçabilité peut devenir complexe pour les jeux de données populaires. Pour éviter tout encombrement, l’affichage par défaut n’affiche que cinq niveaux de traçabilité pour la ressource active. Le reste de la traçabilité peut être développé en cliquant sur les bulles dans le canevas de traçabilité. Les consommateurs de données peuvent également masquer les ressources du canevas qui ne présentent aucun intérêt. Pour réduire davantage l’encombrement, désactivez le bouton bascule **More Lineage (Plus de traçabilité)** en ligne en haut du canevas de traçabilité. Cette action masque toutes les bulles dans le canevas de traçabilité.

   :::image type="content" source="./media/catalog-lineage-user-guide/use-toggle-to-hide-bubbles.png" alt-text="Capture d’écran montrant comment sélectionner le bouton bascule permettant d’afficher davantage de traçabilité." lightbox="./media/catalog-lineage-user-guide/use-toggle-to-hide-bubbles.png":::

1. Utilisez les boutons intelligents dans le canevas de traçabilité pour obtenir une vue optimale de la traçabilité. Les fonctionnalités Disposition automatique, zoom pour ajuster, zoom avant/arrière, plein écran et carte de navigation sont disponibles pour une expérience de traçabilité immersive dans le catalogue.

   :::image type="content" source="./media/catalog-lineage-user-guide/use-lineage-smart-buttons.png" alt-text="Capture d’écran montrant comment sélectionner les boutons intelligents Traçabilité." lightbox="./media/catalog-lineage-user-guide/use-lineage-smart-buttons.png":::

## <a name="next-steps"></a>Étapes suivantes

* [Lien vers Azure Data Factory pour la traçabilité](how-to-link-azure-data-factory.md)
* [Lien vers Azure Data Share pour la traçabilité](how-to-link-azure-data-share.md)