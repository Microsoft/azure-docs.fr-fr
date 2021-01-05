---
title: Interroger des données Azure Blockchain Workbench à l’aide de SQL Server Management Studio
description: Découvrez comment vous connecter à la base de données SQL Azure Blockchain Workbench à partir de SQL Server Management Studio.
ms.date: 11/20/2019
ms.topic: how-to
ms.service: azure-blockchain
ms.reviewer: mmercuri
ms.openlocfilehash: 16e7f9a6c36ea42e1d0a4144e680baebee5a6c21
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009464"
---
# <a name="using-azure-blockchain-workbench-data-with-sql-server-management-studio"></a>Utilisation des données Azure Blockchain Workbench avec SQL Server Management Studio

Microsoft SQL Server Management Studio permet d’écrire et de tester rapidement des requêtes sur la base de données SQL Azure Blockchain Workbench. Cette section explique pas à pas comment vous connecter à la base de données SQL d’Azure Blockchain Workbench à partir de SQL Server Management Studio.

## <a name="prerequisites"></a>Conditions préalables requises

* Téléchargez [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017).

## <a name="connecting-sql-server-management-studio-to-data-in-azure-blockchain-workbench"></a>Connexion de SQL Server Management Studio aux données dans Azure Blockchain Workbench

1. Ouvrez SQL Server Management Studio et sélectionnez **Se connecter**.
2. Sélectionnez **Moteur de base de données**.

    ![Moteur de base de données](./media/data-sql-management-studio/database-engine.png)

3. Dans la boîte de dialogue **Se connecter au serveur**, saisissez le nom du serveur et les informations d’identification de votre base de données.

    Si vous utilisez les informations d’identification créées par le processus de déploiement d’Azure Blockchain Workbench, le nom d’utilisateur sera **dbadmin** et le mot de passe, celui que vous avez indiqué pendant le déploiement.

    ![Saisie des informations d’identification SQL](./media/data-sql-management-studio/sql-creds.png)

   1. SQL Server Management Studio affiche la liste des bases de données, des vues de base de données et des procédures stockées dans la base de données Azure Blockchain Workbench.

      ![Liste des bases de données](./media/data-sql-management-studio/db-list.png)

5. Pour afficher les données associées aux vues de la base de données, vous pouvez générer automatiquement une instruction select. Pour cela, procédez comme suit.
6. Cliquez avec le bouton droit sur l'une des vues de la base de données dans l'Explorateur d'objets.
7. Sélectionnez **Script View as** (Vue du script en tant que).
8. Choisissez **SELECT to** (SÉLECTIONNER vers).
9. Sélectionnez **Nouvelle fenêtre d’éditeur de requête**.
10. Vous pouvez créer une nouvelle requête en sélectionnant **Nouvelle requête**.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Vues de la base de données dans Azure Blockchain Workbench](database-views.md)
