---
title: Utiliser .NET Core pour se connecter à une base de données et l’interroger
description: Cette rubrique explique comment utiliser .NET Core pour créer un programme qui se connecte à une base de données dans Azure SQL Database ou Azure SQL Managed Instance et l’interroge à l’aide d’instructions Transact-SQL.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=2, devx-track-csharp
ms.devlang: dotnet
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 05/29/2020
ms.openlocfilehash: 32ea1dd2141a8df1fb495af64848f87e9f152328
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92669735"
---
# <a name="quickstart-use-net-core-c-to-query-a-database-in-azure-sql-database-or-azure-sql-managed-instance"></a>Démarrage rapide : Utiliser .NET Core (C#) pour interroger une base de données dans Azure SQL Database ou Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Dans ce guide de démarrage rapide, vous allez utiliser [.NET Core](https://www.microsoft.com/net/) et du code C# pour vous connecter à une base de données. Vous allez ensuite exécuter une instruction Transact-SQL pour interroger des données.

> [!TIP]
> Le module Microsoft Learn suivant vous aide à apprendre gratuitement comment [Développer et configurer une application ASP.Net qui interroge une base de données dans Azure SQL Database](/learn/modules/develop-app-that-queries-azure-sql/).

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce démarrage rapide, les éléments suivants sont requis :

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- Une base de données. Utilisez l’un de ces guides de démarrage rapide pour créer et configurer une base de données :

  | Action | SQL Database | Instance managée SQL | SQL Server sur une machine virtuelle Azure |
  |:--- |:--- |:---|:---|
  | Créer| [Portail](single-database-create-quickstart.md) | [Portail](../managed-instance/instance-create-quickstart.md) | [Portail](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  || [INTERFACE DE LIGNE DE COMMANDE](scripts/create-and-configure-database-cli.md) | [INTERFACE DE LIGNE DE COMMANDE](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44) |
  || [PowerShell](scripts/create-and-configure-database-powershell.md) | [PowerShell](../managed-instance/scripts/create-configure-managed-instance-powershell.md) | [PowerShell](../virtual-machines/windows/sql-vm-create-powershell-quickstart.md)
  | Configurer | [Règle de pare-feu IP au niveau du serveur](firewall-create-server-level-portal-quickstart.md)| [Connectivité à partir d’une machine virtuelle](../managed-instance/connect-vm-instance-configure.md)|
  |||[Connectivité locale](../managed-instance/point-to-site-p2s-configure.md) | [Se connecter à une instance SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  |Charger les données|Adventure Works chargé dans le cadre du guide de démarrage rapide|[Restaurer Wide World Importers](../managed-instance/restore-sample-database-quickstart.md) | [Restaurer Wide World Importers](../managed-instance/restore-sample-database-quickstart.md) |
  |||Restaurer ou importer Adventure Works à partir d’un fichier [BACPAC](database-import.md) disponible sur [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)| Restaurer ou importer Adventure Works à partir d’un fichier [BACPAC](database-import.md) disponible sur [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)|
  |||

  > [!IMPORTANT]
  > Les scripts fournis dans cet article utilisent la base de données Adventure Works. Avec une instance managée SQL, vous devez importer la base de données Adventure Works dans une base de données d’instance, ou modifier les scripts fournis dans cet article pour utiliser la base de données Wide World Importers.

- Une version installée de [.NET Core pour votre système d’exploitation](https://www.microsoft.com/net/core).

> [!NOTE]
> Ce guide de démarrage rapide utilise la base de données *mySampleDatabase* . Si vous souhaitez utiliser une autre base de données, vous devrez modifier les références de base de données et modifier la requête `SELECT` dans le code C#.

## <a name="get-server-connection-information"></a>Obtenir les informations de connexion du serveur

Obtenez les informations de connexion dont vous avez besoin pour vous connecter à la base de données dans Azure SQL Database. Vous aurez besoin du nom complet du serveur ou de l’hôte, du nom de la base de données et des informations de connexion pour les procédures suivantes.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Accédez à la page **Bases de données SQL** ou **Instances managées SQL** .

3. Dans la page **Vue d’ensemble** , vérifiez le nom de serveur complet à côté de **Nom de serveur** pour la base de données dans Azure SQL Database, ou le nom de serveur complet (ou l’adresse IP) à côté de **Hôte** pour une instance managée Azure SQL ou un serveur SQL sur une machine virtuelle Azure. Pour copier le nom du serveur ou de l’hôte, pointez dessus et sélectionnez l’icône **Copier** .

> [!NOTE]
> Pour obtenir les informations de connexion de SQL Server sur une machine virtuelle Azure, consultez [Se connecter à une instance SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md#connect-to-sql-server).

## <a name="get-adonet-connection-information-optional---sql-database-only"></a>Obtenir les informations de connexion ADO.NET (facultatif - pour SQL Database uniquement)

1. Accédez à la page **mySampleDatabase** , puis, sous **Paramètres** , sélectionnez **Chaînes de connexion** .

2. Examinez la chaîne de connexion **ADO.NET** complète.

    ![Chaîne de connexion ADO.NET](./media/connect-query-dotnet-core/adonet-connection-string2.png)

3. Copiez la chaîne de connexion **ADO.NET** si vous voulez l’utiliser.
  
## <a name="create-a-new-net-core-project"></a>Créer un nouveau projet .NET Core

1. Ouvrez une invite de commandes et créez un dossier nommé **sqltest** . Accédez à ce dossier et exécutez cette commande.

    ```cmd
    dotnet new console
    ```

    Cette commande crée de nouveaux fichiers projet d’application, y compris un fichier de code C# initial ( **Program.cs** ), un fichier de configuration XML ( **sqltest.csproj** ) et les fichiers binaires nécessaires.

2. Dans un éditeur de texte, ouvrez **sqltest.csproj** et collez le code XML suivant entre les balises `<Project>`. Ce code XML ajoute `System.Data.SqlClient` en tant que dépendance.

    ```xml
    <ItemGroup>
        <PackageReference Include="System.Data.SqlClient" Version="4.6.0" />
    </ItemGroup>
    ```

## <a name="insert-code-to-query-the-database-in-azure-sql-database"></a>Ajouter du code pour interroger la base de données dans Azure SQL Database

1. Ouvrez le fichier **Program.cs** dans un éditeur de texte.

2. Remplacez le contenu par le code suivant et ajoutez les valeurs appropriées pour votre serveur, base de données, nom d’utilisateur et mot de passe.

> [!NOTE]
> Pour utiliser une chaîne de connexion ADO.NET, remplacez les 4 lignes dans le code définissant le serveur, la base de données, le nom d’utilisateur et le mot de passe par la ligne ci-dessous. Dans la chaîne, définissez votre nom d’utilisateur et votre mot de passe.
>
>    `builder.ConnectionString="<your_ado_net_connection_string>";`

```csharp
using System;
using System.Data.SqlClient;
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

                builder.DataSource = "<your_server.database.windows.net>"; 
                builder.UserID = "<your_username>";            
                builder.Password = "<your_password>";     
                builder.InitialCatalog = "<your_database>";
         
                using (SqlConnection connection = new SqlConnection(builder.ConnectionString))
                {
                    Console.WriteLine("\nQuery data example:");
                    Console.WriteLine("=========================================\n");
                    
                    connection.Open();       
                    StringBuilder sb = new StringBuilder();
                    sb.Append("SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName ");
                    sb.Append("FROM [SalesLT].[ProductCategory] pc ");
                    sb.Append("JOIN [SalesLT].[Product] p ");
                    sb.Append("ON pc.productcategoryid = p.productcategoryid;");
                    String sql = sb.ToString();

                    using (SqlCommand command = new SqlCommand(sql, connection))
                    {
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
            Console.WriteLine("\nDone. Press enter.");
            Console.ReadLine(); 
        }
    }
}
```

## <a name="run-the-code"></a>Exécuter le code

1. À l’invite, exécutez les commandes suivantes.

   ```cmd
   dotnet restore
   dotnet run
   ```

2. Vérifiez que les 20 premières lignes sont retournées.

   ```text
   Query data example:
   =========================================

   Road Frames HL Road Frame - Black, 58
   Road Frames HL Road Frame - Red, 58
   Helmets Sport-100 Helmet, Red
   Helmets Sport-100 Helmet, Black
   Socks Mountain Bike Socks, M
   Socks Mountain Bike Socks, L
   Helmets Sport-100 Helmet, Blue
   Caps AWC Logo Cap
   Jerseys Long-Sleeve Logo Jersey, S
   Jerseys Long-Sleeve Logo Jersey, M
   Jerseys Long-Sleeve Logo Jersey, L
   Jerseys Long-Sleeve Logo Jersey, XL
   Road Frames HL Road Frame - Red, 62
   Road Frames HL Road Frame - Red, 44
   Road Frames HL Road Frame - Red, 48
   Road Frames HL Road Frame - Red, 52
   Road Frames HL Road Frame - Red, 56
   Road Frames LL Road Frame - Black, 58
   Road Frames LL Road Frame - Black, 60
   Road Frames LL Road Frame - Black, 62

   Done. Press enter.
   ```

3. Appuyez sur **Entrée** pour fermer la fenêtre d’application.

## <a name="next-steps"></a>Étapes suivantes

- [Prise en main de .NET Core sur Windows/Linux/macOS à l’aide de la ligne de commande](/dotnet/core/tutorials/using-with-xplat-cli).
- Découvrez comment [vous connecter à Azure SQL Database ou Azure SQL Managed Instance et exécuter des requêtes à l’aide de .NET Framework et Visual Studio](connect-query-dotnet-visual-studio.md).  
- Découvrez comment [concevoir votre première base de données à l’aide de SSMS](design-first-database-tutorial.md) ou comment [concevoir une base de données et vous y connecter avec C# et ADO.NET](design-first-database-csharp-tutorial.md).
- Pour plus d’informations sur .NET, consultez la [documentation .NET](/dotnet/).