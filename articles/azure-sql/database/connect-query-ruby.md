---
title: Utiliser Ruby pour les requêtes
description: Cette rubrique vous explique comment utiliser Ruby pour créer un programme qui se connecte à une base de données et l’interroge à l’aide d’instructions Transact-SQL.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: ruby
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 05/29/2020
ms.openlocfilehash: 0434a99d28b4f71594e0ca9ce312087dee5b0ee2
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92672535"
---
# <a name="quickstart-use-ruby-to-query-a-database-in-azure-sql-database-or-azure-sql-managed-instance"></a>Démarrage rapide : Utiliser Ruby pour interroger une base de données dans Azure SQL Database ou Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Ce guide de démarrage rapide explique comment utiliser [Ruby](https://www.ruby-lang.org) pour se connecter à une base de données et l’interroger à l’aide d’instructions Transact-SQL.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce démarrage rapide, vous avez besoin de ce qui suit :

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
  
- Ruby et les logiciels connexes pour votre système d’exploitation :
  
  - **macOS**  : Installez Homebrew, rbenv et ruby-build, Ruby, FreeTDS et TinyTDS. Consultez les étapes 1.2, 1.3, 1.4, 1.5 et 2.1 dans [Créer des applications Ruby à l’aide de SQL Server sur macOS](https://www.microsoft.com/sql-server/developer-get-started/ruby/mac/).
  
  - **Ubuntu**  : Installez les prérequis pour Ruby, rbenv et ruby-build, Ruby, FreeTDS et TinyTDS. Consultez les étapes 1.2, 1.3, 1.4, 1.5 et 2.1 dans [Créer des applications Ruby à l’aide de SQL Server sur Ubuntu](https://www.microsoft.com/sql-server/developer-get-started/ruby/ubuntu/).
  
  - **Windows**  : Installez Ruby, Ruby Devkit et TinyTDS. Voir [Configurer l’environnement de développement pour le développement Ruby](/sql/connect/ruby/step-1-configure-development-environment-for-ruby-development).

## <a name="get-server-connection-information"></a>Obtenir les informations de connexion du serveur

Obtenez les informations de connexion dont vous avez besoin pour vous connecter à une base de données dans Azure SQL Database. Vous aurez besoin du nom complet du serveur ou de l’hôte, du nom de la base de données et des informations de connexion pour les procédures suivantes.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Accédez à la page **Bases de données SQL** ou **Instances managées SQL** .

3. Dans la page **Vue d’ensemble** , vérifiez le nom de serveur complet à côté de **Nom de serveur** pour une base de données dans Azure SQL Database, ou le nom de serveur complet (ou l’adresse IP) à côté de **Hôte** pour une instance managée Azure SQL ou un serveur SQL sur une machine virtuelle Azure. Pour copier le nom du serveur ou de l’hôte, pointez dessus et sélectionnez l’icône **Copier** .

> [!NOTE]
> Pour obtenir les informations de connexion de SQL Server sur une machine virtuelle Azure, consultez [Se connecter à une instance SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md#connect-to-sql-server).

## <a name="create-code-to-query-your-database-in-azure-sql-database"></a>Créer du code pour interroger la base de données dans Azure SQL Database

1. Créez un nouveau fichier nommé *sqltest.py* dans un éditeur de texte ou de code.

1. Ajoutez le code ci-dessous. Remplacez les valeurs `<server>`, `<database>`, `<username>` et `<password>` de votre base de données dans Azure SQL Database.

   >[!IMPORTANT]
   >Dans cet exemple, le code utilise les exemples de données AdventureWorksLT que vous pouvez choisir comme source lors de la création de votre base de données. Si votre base de données comporte des données différentes, utilisez des tables de votre propre base de données dans la requête SELECT. 

   ```ruby
   require 'tiny_tds'
   server = '<server>.database.windows.net'
   database = '<database>'
   username = '<username>'
   password = '<password>'
   client = TinyTds::Client.new username: username, password: password, 
       host: server, port: 1433, database: database, azure: true
   
   puts "Reading data from table"
   tsql = "SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
           FROM [SalesLT].[ProductCategory] pc
           JOIN [SalesLT].[Product] p
           ON pc.productcategoryid = p.productcategoryid"
   result = client.execute(tsql)
   result.each do |row|
       puts row
   end
   ```

## <a name="run-the-code"></a>Exécuter le code

1. Depuis une invite de commandes, exécutez la commande suivante :

   ```bash
   ruby sqltest.rb
   ```
   
1. Vérifiez que les 20 premières lignes catégorie/produit de votre base de données sont retournées. 

## <a name="next-steps"></a>Étapes suivantes
- [Concevoir votre première base de données dans Azure SQL Database](design-first-database-tutorial.md)
- [GitHub repository for TinyTDS](https://github.com/rails-sqlserver/tiny_tds) (Référentiel GitHub pour TinyTDS)
- [Signaler des problèmes ou poser des questions sur TinyTDS](https://github.com/rails-sqlserver/tiny_tds/issues)
- [Pilote Ruby pour SQL Server](/sql/connect/ruby/ruby-driver-for-sql-server/)