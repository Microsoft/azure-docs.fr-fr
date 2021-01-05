---
title: Utiliser les données Azure Blockchain Workbench dans Microsoft Power BI
description: Découvrez comment charger et afficher les données de la base de données SQL Azure Blockchain Workbench dans Microsoft Power BI.
ms.date: 04/22/2020
ms.topic: how-to
ms.reviewer: sunri
ms.openlocfilehash: 7e0e585ce45616c2402972c725b502f4b704d1cd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000145"
---
# <a name="using-azure-blockchain-workbench-data-with-microsoft-power-bi"></a>Utiliser les données Azure Blockchain Workbench avec Microsoft Power BI

Microsoft Power BI permet de générer facilement de puissants rapports à partir de bases de données SQL à l’aide de Power BI Desktop, puis de les publier sur [https://www.powerbi.com](https://www.powerbi.com).

Cet article contient une procédure pas à pas montrant comment se connecter à la base de données SQL Azure Blockchain Workbench à partir de Power BI Desktop, mais aussi comment créer un rapport et le déployer sur powerbi.com.

## <a name="prerequisites"></a>Prérequis

* Téléchargez [Power BI Desktop](https://powerbi.microsoft.com/desktop/).

## <a name="connecting-power-bi-to-data-in-azure-blockchain-workbench"></a>Connecter Power BI aux données dans Azure Blockchain Workbench

1.  Ouvrez Power BI Desktop.
2.  Sélectionnez **Obtenir des données**.

    ![Obtenir des données](./media/data-powerbi/get-data.png)
3.  Sélectionnez **SQL Server** dans la liste des types de source de données.

4.  Indiquez le nom de serveur et celui de la base de données dans la boîte de dialogue. Précisez si vous souhaitez importer les données ou effectuer une **DirectQuery**. Sélectionnez **OK**.

    ![Sélectionner SQL Server](./media/data-powerbi/select-sql.png)

5.  Indiquez les informations d’identification de la base de données pour accéder à Azure Blockchain Workbench. Sélectionnez **Base de données** et saisissez vos informations d’identification.

    Si vous utilisez les informations d’identification créées par le processus de déploiement d’Azure Blockchain Workbench, le nom d’utilisateur sera **dbadmin** et le mot de passe, celui que vous avez indiqué pendant le déploiement.

    ![Paramètres de la base de données SQL](./media/data-powerbi/db-settings.png)

6.  Une fois que vous êtes connecté à la base de données, la boîte de dialogue **Navigateur** affiche les tables et vues disponibles dans la base de données. Les vues sont destinées aux rapports et leurs noms sont précédés de **vw**.

    ![Capture d’écran de Power BI Desktop montrant la boîte de dialogue Navigateur avec l’option vwContractAction sélectionnée.](./media/data-powerbi/navigator.png)

7.  Sélectionnez les vues à inclure. À des fins de démonstration, nous incluons **vwContractAction** qui fournit des détails sur les actions effectuées sur un contrat.

    ![Sélectionner les vues](./media/data-powerbi/select-views.png)

Vous pouvez désormais créer et publier des rapports comme vous le feriez normalement avec Power BI.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Vues de la base de données dans Azure Blockchain Workbench](database-views.md)