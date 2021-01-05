---
title: Utiliser les données Azure Blockchain Workbench dans Microsoft Excel
description: Découvrez comment charger et afficher les données de la base de données SQL Azure Blockchain Workbench Preview dans Microsoft Excel.
ms.date: 09/05/2019
ms.topic: how-to
ms.reviewer: mmercuri
ms.openlocfilehash: 4dd941abdded6a93510f4a71119769cd73855bcc
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000162"
---
# <a name="view-azure-blockchain-workbench-data-with-microsoft-excel"></a>Afficher des données Azure Blockchain Workbench dans Microsoft Excel

Vous pouvez utiliser Microsoft Excel pour afficher des données dans la base de données SQL Azure Blockchain Workbench. Cet article fournit les étapes à suivre pour :

* Vous connecter à la base de données Blockchain Workbench à partir de Microsoft Excel
* Consulter les tables et vues de la base de données Blockchain Workbench
* Charger les données des vues Blockchain Workbench dans Excel

## <a name="connect-to-the-blockchain-workbench-database"></a>Se connecter à la base de données Blockchain Workbench

Pour vous connecter à une base de données Blockchain Workbench :

1. Ouvrez Microsoft Excel.
2. Dans l’onglet **Données**, sélectionnez **Obtenir des données**.
3. Sélectionnez **À partir d’Azure**, puis **À partir d’Azure SQL Database**.

   ![Connexion à Azure SQL Database](./media/data-excel/connect-sql-db.png)

4. Dans la boîte de dialogue **Base de données SQL Server** :

    * Pour **Serveur**, saisissez le nom du serveur Blockchain Workbench.
    * Pour **Base de données (facultatif)** , saisissez le nom de la base de données.

   ![Serveur de base de données et base de données](./media/data-excel/provide-server-db.png)

5. Dans la barre de navigation de la boîte de dialogue **Base de données SQL Server**, sélectionnez **Base de données**. Entrez vos **nom d’utilisateur** et **mot de passe**, puis cliquez sur **Se connecter**.

    > [!NOTE]
    > Si vous utilisez les informations d’identification créées pendant le processus de déploiement d’Azure Blockchain Workbench, le **nom d’utilisateur** est `dbadmin`. Le **mot de passe** est celui que vous avez créé au moment du déploiement de Blockchain Workbench.
    
   ![Informations d'identification permettant d’accéder à la base de données](./media/data-excel/provide-credentials.png)

## <a name="look-at-database-tables-and-views"></a>Examiner les tables et vues de la base de données

La boîte de dialogue du navigateur d’Excel s’ouvre après votre connexion à la base de données. Vous pouvez utiliser ce navigateur pour examiner les tables et les vues dans la base de données. Les vues sont destinées aux rapports et leurs noms sont précédés de **vw**.

   ![Aperçu du navigateur Excel d’une vue](./media/data-excel/excel-navigator.png)

## <a name="load-view-data-into-an-excel-workbook"></a>Charger les données des vues dans un classeur Excel

L’exemple suivant montre comment charger des données à partir d’une vue dans un classeur Excel.

1. Dans la barre de défilement du **navigateur**, sélectionnez la vue **vwContractAction**. L’aperçu **vwContractAction** montre toutes les actions associées à un contrat dans la base de données Blockchain Workbench.
2. Sélectionnez **Charger** pour récupérer toutes les données de la vue et les placer dans votre classeur Excel.

   ![Données chargées à partir d’une vue](./media/data-excel/view-data.png)

Maintenant que vous avez chargé les données, vous pouvez utiliser les fonctionnalités d’Excel pour créer vos propres rapports en utilisant les données de transaction et les métadonnées à partir de la base de données Azure Blockchain Workbench.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Vues de la base de données dans Azure Blockchain Workbench](database-views.md)