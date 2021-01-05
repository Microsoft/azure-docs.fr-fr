---
title: Distribution globale de Azure Cosmos DB
description: Découvrez comment répliquer les données globalement avec Azure Cosmos DB dans le portail Azure
services: cosmos-db
author: SnehaGunda
ms.author: sngun
ms.service: cosmos-db
ms.topic: include
ms.date: 12/26/2018
ms.custom: include file
ms.openlocfilehash: 58788d6194454c8bd40730c9c350aa901924ba3d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96026024"
---
## <a name="add-global-database-regions-using-the-azure-portal"></a><a id="addregion"></a>Ajouter des régions de bases de donnée mondiales à l’aide du portail Azure
Azure Cosmos DB est disponible dans toutes les [régions Azure][azureregions] à travers le monde. Après avoir sélectionné le niveau de cohérence par défaut pour votre compte de base de données, vous pouvez associer une ou plusieurs régions (en fonction de votre choix de niveau de cohérence par défaut et de vos besoins de distribution mondiale).

1. Dans la barre à gauche du [portail Azure](https://portal.azure.com/), cliquez sur **Azure Cosmos DB**.
2. Dans la page **Azure Cosmos DB**, sélectionnez le compte de base de données à modifier.
3. Dans la page du compte, cliquez sur **Répliquer les données globalement** à partir du menu.
4. Dans la page **Répliquer les données globalement**, sélectionnez les régions à ajouter ou à supprimer en cliquant sur des régions sur la carte, puis cliquez sur **Enregistrer**. L’ajout de régions est payant. Pour plus d’informations, consultez la [page de tarification](https://azure.microsoft.com/pricing/details/cosmos-db/) ou l’article [Distribution mondiale des données avec Azure Cosmos DB](../articles/cosmos-db/distribute-data-globally.md).
   
    ![Cliquez sur les régions dans la carte pour les ajouter ou les supprimer.][1]
    
Une fois que vous ajoutez une deuxième région, l’option **Basculement manuel** est activée sur la page **Répliquer les données globalement** dans le portail. Vous pouvez utiliser cette option pour tester le processus de basculement ou modifier la région principale d’écriture. Une fois que vous ajoutez une troisième région, l’option **Priorités de basculement** est activée sur la même page afin que vous puissiez modifier l’ordre de basculement pour les lectures.  

### <a name="selecting-global-database-regions"></a>Sélection de régions de base de données mondiale
Il existe deux scénarios courants pour la configuration de deux ou plusieurs régions :

1. La fourniture aux utilisateurs finaux d’un accès à faible latence aux données, où qu’ils se trouvent dans le monde
2. L’ajout d’une résilience régionale pour la continuité des activités et la récupération d’urgence (BCDR)

Pour fournir une faible latence aux utilisateurs finaux, il est recommandé de déployer l’application et Azure Cosmos DB dans les régions correspondant à la localisation géographique des utilisateurs de l’application.

Pour BCDR, nous vous recommandons d’ajouter les régions en fonction des paires de régions décrites dans l’article [Continuité des activités et récupération d’urgence (BCDR) : régions jumelées Azure][bcdr].

<!--

## <a id="selectwriteregion"></a>Select the write region

While all regions associated with your Cosmos DB database account can serve reads (both, single item as well as multi-item paginated reads) and queries, only one region can actively receive the write (insert, upsert, replace, delete) requests. To set the active write region, do the following  


1. In the **Azure Cosmos DB** blade, select the database account to modify.
2. In the account blade, click **Replicate data globally** from the menu.
3. In the **Replicate data globally** blade, click **Manual Failover** from the top bar.
    ![Change the write region under Azure Cosmos DB Account > Replicate data globally > Manual Failover][2]
4. Select a read region to become the new write region, click the checkbox to confirm triggering a failover, and click OK
    ![Change the write region by selecting a new region in list under Azure Cosmos DB Account > Replicate data globally > Manual Failover][3]

--->

<!--Image references-->
[1]: ./media/cosmos-db-tutorial-global-distribution-portal/azure-cosmos-db-add-region.png
[2]: ./media/cosmos-db-tutorial-global-distribution-portal/azure-cosmos-db-manual-failover-1.png
[3]: ./media/cosmos-db-tutorial-global-distribution-portal/azure-cosmos-db-manual-failover-2.png

<!--Reference style links - using these makes the source content way more readable than using inline links-->
[bcdr]: ../articles/best-practices-availability-paired-regions.md
[consistency]: ../articles/cosmos-db/consistency-levels.md
[azureregions]: https://azure.microsoft.com/regions/#services
[offers]: https://azure.microsoft.com/pricing/details/cosmos-db/