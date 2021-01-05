---
title: Se connecter à un pool SQL dédié (anciennement SQL DW) avec VSTS
description: Interrogez un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics avec Visual Studio.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 08/15/2019
ms.author: kevin
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: c55f8483ba54ecf9778693b364603d642ddb3deb
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452317"
---
# <a name="connect-to-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics-with-visual-studio-and-ssdt"></a>Se connecter à un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics avec Visual Studio et SSDT

> [!div class="op_single_selector"]
> * [Azure Data Studio](../sql/get-started-azure-data-studio.md)
> * [Power BI](sql-data-warehouse-get-started-visualize-with-power-bi.md)
> * [Visual Studio](sql-data-warehouse-query-visual-studio.md)
> * [sqlcmd](../sql/get-started-connect-sqlcmd.md) 
> * [SSMS](sql-data-warehouse-query-ssms.md)
> 
> 

Utilisez Visual Studio pour interroger un pool SQL dédié (anciennement SQL DW) dans Azure Synapse en seulement quelques minutes. Cette méthode utilise l’extension SQL Server Data Tools (SSDT) dans Visual Studio 2019. 

## <a name="prerequisites"></a>Prérequis
Pour utiliser ce didacticiel, vous avez besoin des éléments suivants :

* Un pool SQL dédié existant (anciennement SQL DW). Pour en créer un, consultez [Créer un pool SQL dédié (anciennement SQL DW)](create-data-warehouse-portal.md)
* SSDT pour Visual Studio. Si vous avez Visual Studio, vous disposez probablement déjà de SSDT pour Visual Studio. Pour obtenir des instructions et des options d’installation, consultez [Installation de Visual Studio et/ou SSDT](sql-data-warehouse-install-visual-studio.md).
* Le nom complet du serveur SQL. Pour obtenir ces informations, consultez [Se connecter à un pool SQL dédié (anciennement SQL DW)](sql-data-warehouse-connect-overview.md).

## <a name="1-connect-to-your-dedicated-sql-pool-formerly-sql-dw"></a>1. Se connecter au pool SQL dédié (anciennement SQL DW)
1. Ouvrez Visual Studio 2019.
2. Ouvrez l'Explorateur d’objets SQL Server en sélectionnant **Afficher** > **Explorateur d’objets SQL Server**.
   
    ![Explorateur d’objets SQL Server](./media/sql-data-warehouse-query-visual-studio/open-ssdt.png)
3. Cliquez sur l’icône **Ajouter SQL Server** .
   
    ![Ajouter SQL Server](./media/sql-data-warehouse-query-visual-studio/add-server.png)
4. Renseignez les champs dans la fenêtre Se connecter au serveur.
   
    ![Se connecter au serveur](./media/sql-data-warehouse-query-visual-studio/connection-dialog.png)
   
   * **Nom du serveur**. Saisissez le **nom du serveur** précédemment identifié.
   * **Authentification**. Sélectionnez **Authentification SQL Server** ou **Authentification intégrée Active Directory**.
   * **Nom d’utilisateur** et **Mot de passe**. Entrez un nom d’utilisateur et mot de passe si l’authentification SQL Server a été sélectionnée plus haut.
   * Cliquez sur **Connecter**.
5. Pour voir plus d’informations, développez votre serveur SQL Azure. Vous pouvez afficher les bases de données associées au serveur. Développez AdventureWorksDW pour voir les tables de votre exemple de base de données.
   
    ![Explorer AdventureWorksDW](./media/sql-data-warehouse-query-visual-studio/explore-sample.png)

## <a name="2-run-a-sample-query"></a>2. Exécuter un exemple de requête
Maintenant qu’une connexion à votre base de données a été établie, passons à l’écriture d’une requête.

1. Cliquez avec le bouton droit sur votre base de données dans l’Explorateur d’objets SQL Server.
2. Sélectionnez **Nouvelle requête**. Une nouvelle fenêtre de requête s’ouvre.
   
    ![Nouvelle requête](./media/sql-data-warehouse-query-visual-studio/new-query2.png)
3. Copiez la requête T-SQL suivante dans la fenêtre de requête :
   
    ```sql
    SELECT COUNT(*) FROM dbo.FactInternetSales;
    ```
4. Exécutez la requête en cliquant sur la flèche verte ou utilisez le raccourci : `CTRL`+`SHIFT`+`E`.
   
    ![Exécuter une requête](./media/sql-data-warehouse-query-visual-studio/run-query.png)
5. Passez en revue les résultats de la requête. Dans cet exemple, la table FactInternetSales a 60 398 lignes.
   
    ![Résultats de la requête](./media/sql-data-warehouse-query-visual-studio/query-results.png)

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous pouvez vous connecter et exécuter des requêtes, essayez de [visualiser les données avec Power BI](sql-data-warehouse-get-started-visualize-with-power-bi.md).

Pour configurer votre environnement pour l’authentification Azure Active Directory, consultez [S’authentifier auprès d’un pool SQL dédié (anciennement SQL DW)](sql-data-warehouse-authentication.md).
