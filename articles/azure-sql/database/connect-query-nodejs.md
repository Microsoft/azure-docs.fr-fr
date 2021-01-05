---
title: Utiliser Node.js pour interroger une base de données
description: Comment utiliser Node.js pour créer un programme qui se connecte à une base de données dans Azure SQL Database ou Azure SQL Managed Instance, et pour l’interroger à l’aide d’instructions T-SQL.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.devlang: nodejs
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: v-masebo
ms.date: 05/29/2020
ms.custom: seo-javascript-september2019, seo-javascript-october2019, sqldbrb=2, devx-track-js
ms.openlocfilehash: 44530577972839aacb803d1722fa97716088fa0d
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91325434"
---
# <a name="quickstart-use-nodejs-to-query-a-database-in-azure-sql-database-or-azure-sql-managed-instance"></a>Démarrage rapide : Utiliser Node.js pour interroger une base de données dans Azure SQL Database ou Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Dans ce guide de démarrage rapide, vous utilisez Node.js pour vous connecter à une base de données et interroger les données.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce démarrage rapide, les éléments suivants sont requis :

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

  | Action | SQL Database | Instance managée SQL | SQL Server sur une machine virtuelle Azure |
  |:--- |:--- |:---|:---|
  | Créer| [Portail](single-database-create-quickstart.md) | [Portail](../managed-instance/instance-create-quickstart.md) | [Portail](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  || [INTERFACE DE LIGNE DE COMMANDE](scripts/create-and-configure-database-cli.md) | [INTERFACE DE LIGNE DE COMMANDE](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44) |
  || [PowerShell](scripts/create-and-configure-database-powershell.md) | [PowerShell](../managed-instance/scripts/create-configure-managed-instance-powershell.md) | [PowerShell](../virtual-machines/windows/sql-vm-create-powershell-quickstart.md)
  | Configurer | [Règle de pare-feu IP au niveau du serveur](firewall-create-server-level-portal-quickstart.md)| [Connectivité à partir d’une machine virtuelle](../managed-instance/connect-vm-instance-configure.md)|
  |||[Connectivité locale](../managed-instance/point-to-site-p2s-configure.md) | [Se connecter à une instance SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
  |Charger les données|Adventure Works chargé dans le cadre du guide de démarrage rapide|[Restaurer Wide World Importers](../managed-instance/restore-sample-database-quickstart.md) | [Restaurer Wide World Importers](../managed-instance/restore-sample-database-quickstart.md) |
  |||Restaurer ou importer Adventure Works à partir d’un fichier [BACPAC](database-import.md) sur [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)| Restaurer ou importer Adventure Works à partir d’un fichier [BACPAC](database-import.md) sur [GitHub](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works)|
  |||


- Logiciels associés à [Node.js](https://nodejs.org)

  # <a name="macos"></a>[macOS](#tab/macos)

  Installez Homebrew et Node.js, puis installez le pilote ODBC et SQLCMD en suivant les étapes **1.2** et **1.3** dans [Créer des applications Node.js à l’aide de SQL Server sur macOS](https://www.microsoft.com/sql-server/developer-get-started/node/mac/).

  # <a name="ubuntu"></a>[Ubuntu](#tab/ubuntu)

  Installez Node.js, puis le pilote ODBC et SQLCMD en suivant les étapes **1.2** et **1.3** dans [Créer des applications Node.js à l’aide de SQL Server sur Ubuntu](https://www.microsoft.com/sql-server/developer-get-started/node/ubuntu/).

  # <a name="windows"></a>[Windows](#tab/windows)

  Installez Chocolatey et Node.js, puis installez le pilote ODBC et SQLCMD en suivant les étapes **1.2** et **1.3** dans [Créer des applications Node.js à l’aide de SQL Server sur Windows](https://www.microsoft.com/sql-server/developer-get-started/node/windows/).

  ---

> [!IMPORTANT]
> Les scripts fournis dans cet article utilisent la base de données **Adventure Works**.

> [!NOTE]
> Vous pouvez éventuellement choisir d’utiliser une instance managée Azure SQL.
>
> Pour la création et la configuration, utilisez le [portail Azure](../managed-instance/instance-create-quickstart.md), [PowerShell](../managed-instance/scripts/create-configure-managed-instance-powershell.md) ou l’[interface CLI](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44), puis configurez une connectivité [locale](../managed-instance/point-to-site-p2s-configure.md) ou de [machine virtuelle](../managed-instance/connect-vm-instance-configure.md).
>
> Pour charger des données, consultez [Restaurer avec BACPAC](database-import.md) avec le fichier [Adventure Works](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/adventure-works), ou consultez [Restaurer la base de données Wide World Importers](../managed-instance/restore-sample-database-quickstart.md).

## <a name="get-server-connection-information"></a>Obtenir les informations de connexion du serveur

Obtenez les informations de connexion dont vous avez besoin pour vous connecter à la base de données dans Azure SQL Database. Vous aurez besoin du nom complet du serveur ou de l’hôte, du nom de la base de données et des informations de connexion pour les procédures suivantes.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Accédez à la page **Bases de données SQL** ou **Instances managées SQL**.

3. Dans la page **Vue d’ensemble**, vérifiez le nom de serveur complet à côté de **Nom de serveur** pour une base de données dans Azure SQL Database, ou le nom de serveur complet (ou l’adresse IP) à côté de **Hôte** pour une instance managée Azure SQL ou un serveur SQL sur une machine virtuelle Azure. Pour copier le nom du serveur ou de l’hôte, pointez dessus et sélectionnez l’icône **Copier**.

> [!NOTE]
> Pour obtenir les informations de connexion de SQL Server sur une machine virtuelle Azure, consultez [Se connecter à SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md#connect-to-sql-server).

## <a name="create-the-project"></a>Créer le projet

Ouvrez une invite de commandes et créez un dossier nommé *sqltest*. Ouvrez le dossier que vous avez créé et exécutez la commande suivante :

  ```bash
  npm init -y
  npm install tedious
  ```

## <a name="add-code-to-query-the-database"></a>Ajouter du code pour interroger la base de données

1. Dans votre éditeur de texte favori, créez un nouveau fichier, *sqltest.js*.

1. Remplacez son contenu par le code ci-dessous. Ensuite, ajoutez les valeurs appropriées pour vos serveur, base de données, utilisateur et mot de passe.

    ```js
    const { Connection, Request } = require("tedious");

    // Create connection to database
    const config = {
      authentication: {
        options: {
          userName: "username", // update me
          password: "password" // update me
        },
        type: "default"
      },
      server: "your_server.database.windows.net", // update me
      options: {
        database: "your_database", //update me
        encrypt: true
      }
    };

    const connection = new Connection(config);

    // Attempt to connect and execute queries if connection goes through
    connection.on("connect", err => {
      if (err) {
        console.error(err.message);
      } else {
        queryDatabase();
      }
    });

    function queryDatabase() {
      console.log("Reading rows from the Table...");

      // Read all rows from table
      const request = new Request(
        `SELECT TOP 20 pc.Name as CategoryName,
                       p.name as ProductName
         FROM [SalesLT].[ProductCategory] pc
         JOIN [SalesLT].[Product] p ON pc.productcategoryid = p.productcategoryid`,
        (err, rowCount) => {
          if (err) {
            console.error(err.message);
          } else {
            console.log(`${rowCount} row(s) returned`);
          }
        }
      );

      request.on("row", columns => {
        columns.forEach(column => {
          console.log("%s\t%s", column.metadata.colName, column.value);
        });
      });

      connection.execSql(request);
    }
    ```

> [!NOTE]
> Cet exemple de code utilise l’exemple de base de données **AdventureWorksLT** dans Azure SQL Database.

## <a name="run-the-code"></a>Exécuter le code

1. À l’invite de commande, exécutez ce programme.

    ```bash
    node sqltest.js
    ```

1. Vérifiez que les 20 premières lignes ont été renvoyées et fermez la fenêtre d’application.

## <a name="next-steps"></a>Étapes suivantes

- [Node.js Driver for SQL Server](/sql/connect/node-js/node-js-driver-for-sql-server) (Pilote Node.js pour SQL Server)

- Se connecter et exécuter des requêtes sur Windows/Linux/macOS avec [.NET core](connect-query-dotnet-core.md), [Visual Studio Code](connect-query-vscode.md) ou [SSMS](connect-query-ssms.md) (Windows uniquement)

- [Prise en main de .NET Core sur Windows/Linux/macOS à l’aide de la ligne de commande](/dotnet/core/tutorials/using-with-xplat-cli)

- Concevoir votre première base de données dans Azure SQL Database avec [.NET](design-first-database-csharp-tutorial.md) ou [SSMS](design-first-database-tutorial.md)
