---
title: Voir une base de connaissances avec l’Explorateur Stockage
titleSuffix: Azure Cognitive Search
description: Visualisez et analysez une base de connaissances Recherche cognitive Azure avec l’Explorateur Stockage du portail Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: ba0b02067c032f9038051c169866588ded44af73
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85566020"
---
# <a name="view-a-knowledge-store-with-storage-explorer"></a>Voir une base de connaissances avec l’Explorateur Stockage

Cet article explique, par exemple, comment se connecter à une base de connaissances et l’explorer à l’aide de l’Explorateur Stockage dans le portail Azure.

## <a name="prerequisites"></a>Prérequis

+ Suivez les étapes décrites dans [Créer une base de connaissances dans le portail Azure](knowledge-store-create-portal.md) pour créer l’exemple de base de connaissances utilisé dans cette procédure pas à pas.

+ Vous aurez également besoin du nom du compte de stockage Azure utilisé pour créer la base de connaissances ainsi que de sa clé d’accès dans le portail Azure.

## <a name="view-edit-and-query-a-knowledge-store-in-storage-explorer"></a>Visualiser, modifier et interroger une base de connaissances dans l’Explorateur Stockage

1. Dans le portail Azure, [ouvrez le compte de stockage](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Storage%2storageAccounts/) que vous avez utilisé pour créer la base de connaissances.

1. Dans le volet de navigation de gauche du compte de stockage, cliquez sur **Explorateur Stockage**.

1. Développez la liste **TABLES** pour afficher la liste des projections de tables Azure créées durant l’exécution de l’Assistant **Importation des données** sur les exemples de données d’avis des clients concernant les hôtels.

Sélectionnez une table pour voir les données enrichies, notamment les phrases clés et les scores de sentiment.

   ![Afficher des tables dans l’Explorateur Stockage](media/knowledge-store-view-storage-explorer/storage-explorer-tables.png "Afficher des tables dans l’Explorateur Stockage")

Pour changer le type de données d’une valeur de table ou pour changer des valeurs individuelles dans votre table, cliquez sur **Modifier**. Quand vous changez le type de données d’une colonne dans une ligne de table, il s’applique à toutes les lignes.

   ![Modifier une table dans l’Explorateur Stockage](media/knowledge-store-view-storage-explorer/storage-explorer-edit-table.png "Modifier une table dans l’Explorateur Stockage")

Pour exécuter des requêtes, cliquez sur **Requête** dans la barre de commandes, puis entrez vos conditions.  

   ![Interroger une table dans l’Explorateur Stockage](media/knowledge-store-view-storage-explorer/storage-explorer-query-table.png "Interroger une table dans l’Explorateur Stockage")

## <a name="clean-up"></a>Nettoyer

Lorsque vous travaillez dans votre propre abonnement, il est recommandé, à la fin de chaque projet, de déterminer si vous avez toujours besoin des ressources que vous avez créées. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens **Toutes les ressources** ou **Groupes de ressources** situés dans le volet de navigation de gauche.

Si vous utilisez un service gratuit, n’oubliez pas que vous êtes limité à trois index, indexeurs et sources de données. Vous pouvez supprimer des éléments un par un dans le portail pour ne pas dépasser la limite.

## <a name="next-steps"></a>Étapes suivantes

Connectez cette base de connaissances à Power BI pour effectuer une analyse plus approfondie, ou allez plus loin avec le code en utilisant l’API REST et Postman pour créer une autre base de connaissances.

> [!div class="nextstepaction"]
> [Se connecter avec Power BI](knowledge-store-connect-power-bi.md)
> [Créer une base de connaissances avec REST](knowledge-store-create-rest.md)
