---
title: Utiliser Visual Studio avec .NET et C# pour interroger
description: Utilisez Visual Studio pour créer une application C# qui se connecte à une base de données dans Azure SQL Database ou Azure SQL Managed instance et qui exécute des requêtes.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: devx-track-csharp, sqldbrb=2
ms.devlang: dotnet
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 08/10/2020
ms.openlocfilehash: a864b2b3e0379a8b0a1d67c97a63b3d5c52f9e58
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92669715"
---
# <a name="quickstart-use-net-and-c-in-visual-studio-to-connect-to-and-query-a-database-in-azure-sql-database-or-azure-sql-managed-instance"></a>Démarrage rapide : Utiliser .NET et C# dans Visual Studio pour se connecter à une base de données dans Azure SQL Database ou Azure SQL Managed Instance et interroger la base de données
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Ce guide de démarrage rapide montre comment utiliser [.NET Framework](https://www.microsoft.com/net/) et le code C# dans Visual Studio pour interroger une base de données dans Azure SQL Database à l’aide d’instructions Transact-SQL.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce démarrage rapide, les éléments suivants sont requis :

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- Une base de données dans Azure SQL Database. Utilisez l’un de ces guides de démarrage rapide pour créer et configurer une base de données dans Azure SQL Database :

  | Action | SQL Database | Instance managée SQL | SQL Server sur une machine virtuelle Azure |
  |:--- |:--- |:---|:---|
  | Créer| [Portail](single-database-create-quickstart.md) | [Portail](../managed-instance/instance-create-quickstart.md) | [Portail](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  || [INTERFACE DE LIGNE DE COMMANDE](scripts/create-and-configure-database-cli.md) | [INTERFACE DE LIGNE DE COMMANDE](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44) |
  || [PowerShell](scripts/create-and-configure-database-powershell.md) | [PowerShell](../managed-instance/scripts/create-configure-managed-instance-powershell.md) | [PowerShell](../virtual-machines/windows/sql-vm-create-powershell-quickstart.md)
  | Configurer | [Règle de pare-feu IP au niveau du serveur](firewall-create-server-level-portal-quickstart.md)| [Connectivité à partir d’une machine virtuelle](../managed-instance/connect-vm-instance-configure.md)|
  |||[Connectivité locale](../managed-instance/point-to-site-p2s-configure.md) | [Se connecter à SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  |Charger les données|Adventure Works chargé dans le cadre du guide de démarrage rapide|[Restaurer Wide World Importers](../managed-instance/restore-sample-database-quickstart.md) | [Restaurer Wide World Importers](../managed-instance/restore-sample-database-quickstart.md) |
  |||Restaurer ou importer Adventure Works à partir d’un fichier [BACPAC](database-import.md) disponible sur [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)| Restaurer ou importer Adventure Works à partir d’un fichier [BACPAC](database-import.md) disponible sur [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)|
  |||

  > [!IMPORTANT]
  > Les scripts fournis dans cet article utilisent la base de données Adventure Works. Avec une instance managée SQL, vous devez importer la base de données Adventure Works dans une base de données d’instance, ou modifier les scripts fournis dans cet article pour utiliser la base de données Wide World Importers.

- [Visual Studio 2019](https://www.visualstudio.com/downloads/) édition Community, Professional ou Enterprise.

## <a name="get-server-connection-information"></a>Obtenir les informations de connexion du serveur

Obtenez les informations de connexion dont vous avez besoin pour vous connecter à la base de données. Vous aurez besoin du nom complet du serveur ou de l’hôte, du nom de la base de données et des informations de connexion pour les procédures suivantes.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Accédez à la page **Bases de données SQL** ou **Instances managées SQL** .

3. Dans la page **Vue d’ensemble** , vérifiez le nom de serveur complet à côté de **Nom de serveur** pour une base de données dans Azure SQL Database, ou le nom de serveur complet (ou l’adresse IP) à côté de **Hôte** pour une instance managée Azure SQL ou un serveur SQL sur une machine virtuelle Azure. Pour copier le nom du serveur ou de l’hôte, pointez dessus et sélectionnez l’icône **Copier** .

> [!NOTE]
> Pour obtenir les informations de connexion de SQL Server sur une machine virtuelle Azure, consultez [Se connecter à une instance SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md#connect-to-sql-server).

## <a name="create-code-to-query-the-database-in-azure-sql-database"></a>Créer du code pour interroger la base de données dans Azure SQL Database

1. Dans Visual Studio, créez un projet. 
   
1. Dans la boîte de dialogue **Nouveau projet** , sélectionnez **Visual C#** , **Application console (.NET Framework)** .
   
1. Entrez *sqltest* pour le nom du projet puis sélectionnez **OK** . Le nouveau projet est créé. 
   
1. Sélectionnez **Projet** > **Gérer les packages NuGet** . 
   
1. Dans **Gestionnaire de package NuGet** , sélectionnez l'onglet **Parcourir** , puis recherchez et sélectionnez **Microsoft.Data.SqlClient** .
   
1. Sur la page **Microsoft.Data.SqlClient** , sélectionnez **Installer** . 
   - Si vous y êtes invité, sélectionnez **OK** pour poursuivre l’installation. 
   - Si une fenêtre **Acceptation de la licence** apparaît, sélectionnez **J’accepte** .
   
1. Une fois l’installation terminée, vous pouvez fermer le **Gestionnaire de package NuGet** . 
   
1. Dans l’éditeur de code, remplacez le contenu de **Program.cs** par le code suivant. Remplacez `<your_server>`, `<your_username>`, `<your_password>` et `<your_database>` par vos valeurs.
   
   >[!IMPORTANT]
   >Dans cet exemple, le code utilise les exemples de données AdventureWorksLT que vous pouvez choisir comme source lors de la création de votre base de données. Si votre base de données comporte des données différentes, utilisez des tables de votre propre base de données dans la requête SELECT. 
   
   ```csharp
   using System;
   using Microsoft.Data.SqlClient;
   using System.Text;
   
   namespace sqltest
   {
       class Program
       {
           static void Main(string[] args)
           {
               try 
               { 
                   SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder();
                   builder.DataSource = "<your_server>.database.windows.net"; 
                   builder.UserID = "<your_username>";            
                   builder.Password = "<your_password>";     
                   builder.InitialCatalog = "<your_database>";
   
                   using (SqlConnection connection = new SqlConnection(builder.ConnectionString))
                   {
                       Console.WriteLine("\nQuery data example:");
                       Console.WriteLine("=========================================\n");
                       
                       StringBuilder sb = new StringBuilder();
                       sb.Append("SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName ");
                       sb.Append("FROM [SalesLT].[ProductCategory] pc ");
                       sb.Append("JOIN [SalesLT].[Product] p ");
                       sb.Append("ON pc.productcategoryid = p.productcategoryid;");
                       String sql = sb.ToString();
   
                       using (SqlCommand command = new SqlCommand(sql, connection))
                       {
                           connection.Open();
                           using (SqlDataReader reader = command.ExecuteReader())
                           {
                               while (reader.Read())
                               {
                                   Console.WriteLine("{0} {1}", reader.GetString(0), reader.GetString(1));
                               }
                           }
                       }                    
                   }
               }
               catch (SqlException e)
               {
                   Console.WriteLine(e.ToString());
               }
               Console.ReadLine();
           }
       }
   }
   ```

## <a name="run-the-code"></a>Exécuter le code

1. Pour exécuter l’application, sélectionnez **Déboguer** > **Démarrer le débogage** , ou sélectionnez **Démarrer** dans la barre d’outils, ou appuyez sur **F5** .
1. Vérifiez que les 20 premières lignes catégorie/produit de votre base de données sont retournées, puis fermez la fenêtre de l’application.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [vous connecter à une base de données dans Azure SQL Database et l’interroger en utilisant .NET Core](connect-query-dotnet-core.md) sur Windows/Linux/macOS.  
- En savoir plus sur la [prise en main de .NET Core sur Windows/Linux/macOS à l’aide de la ligne de commande](/dotnet/core/tutorials/using-with-xplat-cli).
- Découvrez comment [concevoir votre première base de données dans Azure SQL Database en utilisant SSMS](design-first-database-tutorial.md) ou comment [concevoir votre première base de données dans Azure SQL Database en utilisant .NET](design-first-database-csharp-tutorial.md).
- Pour plus d’informations sur .NET, consultez la [documentation .NET](/dotnet/).
- Exemple de logique de nouvelle tentative : [Connexion résiliente à Azure SQL avec ADO.NET][step-4-connect-resiliently-to-sql-with-ado-net-a78n].


<!-- Link references. -->

[step-4-connect-resiliently-to-sql-with-ado-net-a78n]: /sql/connect/ado-net/step-4-connect-resiliently-sql-ado-net