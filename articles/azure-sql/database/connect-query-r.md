---
title: Utiliser R avec Azure SQL Database Machine Learning Services (préversion) pour interroger une base de données
titleSuffix: Azure SQL Database Machine Learning Services (preview)
description: Cet article explique comment utiliser un script R avec Azure SQL Database Machine Learning Services pour vous connecter à une base de données dans Azure SQL Database et l’interroger à l’aide d’instructions Transact-SQL.
services: sql-database
ms.service: sql-database
ms.subservice: machine-learning
ms.custom: sqldbrb=2
ms.devlang: python
ms.topic: quickstart
author: garyericson
ms.author: garye
ms.reviewer: davidph, sstein
manager: cgronlun
ms.date: 05/29/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 2e32a4abeae78aa7105f21ecffbb18c2eae841a4
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185621"
---
# <a name="quickstart-use-r-with-azure-sql-database-machine-learning-services-preview-to-query-a-database"></a>Démarrage rapide : Utiliser R avec Azure SQL Database Machine Learning Services (préversion) pour interroger une base de données 

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Dans ce guide de démarrage rapide, vous utilisez R avec Azure SQL Database Machine Learning Services pour vous connecter à une base de données dans Azure SQL Database et l’interroger à l’aide d’instructions T-SQL.

[!INCLUDE[ml-preview-note](../../../includes/sql-database-ml-preview-note.md)]

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [Azure SQL Database](single-database-create-quickstart.md)
- [Machine Learning Services](../managed-instance/machine-learning-services-overview.md) avec R activé.
- [SSMS](/sql/ssms/sql-server-management-studio-ssms) (SQL Server Management Studio)

> [!IMPORTANT]
> Les scripts fournis dans cet article utilisent la base de données **Adventure Works**.

Machine Learning Services avec R est une fonctionnalité d’Azure SQL Database utilisée pour exécuter des scripts R dans les bases de données. Pour plus d’informations, consultez le [projet R](https://www.r-project.org/).

## <a name="get-the-sql-server-connection-information"></a>Obtenir les informations de connexion SQL Server

Obtenez les informations de connexion dont vous avez besoin pour vous connecter à la base de données dans Azure SQL Database. Vous aurez besoin du nom complet du serveur ou de l’hôte, du nom de la base de données et des informations de connexion pour les procédures suivantes.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Accédez à la page **Bases de données SQL** ou **Instances managées SQL**.

3. Dans la page **Vue d’ensemble**, vérifiez le nom de serveur complet à côté de **Nom de serveur** pour une base de données dans Azure SQL Database ou le nom de serveur complet à côté de **Hôte** pour une instance managée dans Azure SQL Managed Instance. Pour copier le nom du serveur ou de l’hôte, pointez dessus et sélectionnez l’icône **Copier**.

## <a name="create-code-to-query-your-database"></a>Créer du code pour interroger votre base de données

1. Ouvrez **SQL Server Management Studio** et connectez-vous à votre base de données.

   Si vous avez besoin d’aide pour la connexion, consultez [Démarrage rapide : Utiliser SQL Server Management Studio pour se connecter à une base de données dans Azure SQL Database et l’interroger](connect-query-ssms.md).

1. Transmettez la totalité du script R à la procédure stockée [sp_execute_external_script](/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql).

   Le script est transmis via l’argument `@script`. Tout ce qui se trouve dans l’argument `@script` doit être du code R valide.
   
   >[!IMPORTANT]
   >Dans cet exemple, le code utilise les exemples de données AdventureWorksLT que vous pouvez choisir comme source lors de la création de votre base de données. Si votre base de données comporte des données différentes, utilisez des tables de votre propre base de données dans la requête SELECT. 

    ```sql
    EXECUTE sp_execute_external_script
    @language = N'R'
    , @script = N'OutputDataSet <- InputDataSet;'
    , @input_data_1 = N'SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName FROM [SalesLT].[ProductCategory] pc JOIN [SalesLT].[Product] p ON pc.productcategoryid = p.productcategoryid'
    ```

   > [!NOTE]
   > Si vous rencontrez des erreurs, cela peut être dû au fait que la préversion publique de Machine Learning Services (avec R) n’est pas activée pour votre base de données. Consultez [Composants requis](#prerequisites) ci-dessus.

## <a name="run-the-code"></a>Exécuter le code

1. Exécutez la procédure stockée [sp_execute_external_script](/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql).

1. Vérifiez que les 20 premières lignes catégorie/produit sont retournées dans la fenêtre **Messages**.

## <a name="next-steps"></a>Étapes suivantes

- [Concevoir votre première base de données dans Azure SQL Database](design-first-database-tutorial.md)
- [Machine Learning Services (avec R) dans Azure SQL Database](../managed-instance/machine-learning-services-overview.md)
- [Créer et exécuter des scripts R simples dans Azure SQL Database Machine Learning Services (préversion)](/sql/machine-learning/tutorials/quickstart-r-create-script?context=%2fazure%2fazure-sql%2fmanaged-instance%2fcontext%2fml-context)