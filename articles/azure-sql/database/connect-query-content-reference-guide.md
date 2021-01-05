---
title: Se connecter et interroger
description: Liens vers les guides de démarrage rapide Azure SQL Database qui expliquent comment se connecter à une base de données dans Azure SQL Database ou Azure SQL Managed Instance, et comment l’interroger.
titleSuffix: Azure SQL Database & SQL Managed Instance
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: guide
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 05/29/2020
ms.openlocfilehash: e25469e23358ade32e050bed0818338dc290e71f
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96451389"
---
# <a name="azure-sql-database-and-azure-sql-managed-instance-connect-and-query-articles"></a>Articles sur la connexion et l’interrogation dans Azure SQL Database et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Le document suivant contient des liens vers des exemples Azure qui montrent comment se connecter à une base de données Azure SQL Database ou Azure SQL Managed Instance, puis comment l’interroger. Pour obtenir des recommandations connexes à la sécurité TLS, consultez [Considérations relatives au protocole TLS pour la connectivité des bases de données](#tls-considerations-for-database-connectivity).

## <a name="quickstarts"></a>Démarrages rapides

| Démarrage rapide | Description |
|---|---|
|[SQL Server Management Studio](connect-query-ssms.md)|Ce guide de démarrage rapide montre comment utiliser SSMS pour se connecter à une base de données, puis comment utiliser des instructions Transact-SQL pour interroger, ajouter, mettre à jour et supprimer des données dans la base de données.|
|[Azure Data Studio](/sql/azure-data-studio/quickstart-sql-database?toc=%2fazure%2fsql-database%2ftoc.json)|Ce guide de démarrage rapide montre comment utiliser Azure Data Studio pour se connecter à une base de données, puis comment utiliser des instructions T-SQL (Transact-SQL) pour créer le TutorialDB utilisé dans les tutoriels Azure Data Studio.|
|[Azure portal](connect-query-portal.md)|Ce guide de démarrage rapide montre comment utiliser l’éditeur de requête pour se connecter à une base de données (Azure SQL Database uniquement), puis comment utiliser des instructions Transact-SQL pour interroger, ajouter, mettre à jour et supprimer des données dans la base de données.|
|[Visual Studio Code](connect-query-vscode.md)|Ce guide de démarrage rapide montre comment utiliser Visual Studio Code pour se connecter à une base de données, puis comment utiliser des instructions Transact-SQL pour interroger, ajouter, mettre à jour et supprimer des données dans la base de données.|
|[.NET avec Visual Studio](connect-query-dotnet-visual-studio.md)|Ce guide de démarrage rapide montre comment utiliser .NET Framework pour créer un programme C# avec Visual Studio en vue de se connecter à une base de données et d’utiliser des instructions Transact-SQL pour interroger les données.|
|[.NET Core](connect-query-dotnet-core.md)|Ce guide de démarrage rapide montre comment utiliser .NET Core sur Windows/Linux/macOS pour créer un programme C# en vue de se connecter à une base de données et d’utiliser des instructions Transact-SQL pour interroger les données.|
|[Go](connect-query-go.md)|Ce guide de démarrage rapide montre comment utiliser Go pour se connecter à une base de données. Les instructions Transact-SQL permettant d’interroger et de modifier des données sont également présentées.|
|[Java](connect-query-java.md)|Ce guide de démarrage rapide explique comment utiliser Java pour se connecter à une base de données et comment l’interroger à l’aide d’instructions Transact-SQL.|
|[Node.JS](connect-query-nodejs.md)|Ce guide de démarrage rapide explique comment utiliser Node.js pour créer un programme en vue de se connecter à une base de données et l’interroger à l’aide d’instructions Transact-SQL.|
|[PHP](connect-query-php.md)|Ce guide de démarrage rapide explique comment utiliser PHP pour créer un programme en vue de se connecter à une base de données et l’interroger à l’aide d’instructions Transact-SQL.|
|[Python](connect-query-python.md)|Ce guide de démarrage rapide explique comment utiliser Python pour se connecter à une base de données et comment l’interroger à l’aide d’instructions Transact-SQL. |
|[Ruby](connect-query-ruby.md)|Ce guide de démarrage rapide explique comment utiliser Ruby pour créer un programme en vue de se connecter à une base de données et l’interroger à l’aide d’instructions Transact-SQL.|
|[R](connect-query-r.md)|Ce guide de démarrage rapide explique comment utiliser R avec Machine Learning Services pour créer un programme en vue de se connecter à une base de données dans Azure SQL Database et l’interroger à l’aide d’instructions Transact-SQL.|
|||

## <a name="tls-considerations-for-database-connectivity"></a>Considérations relatives au protocole TLS pour la connectivité des bases de données

Le protocole TLS (Transport Layer Security) est utilisé par tous les pilotes fournis et pris en charge par Microsoft pour la connexion aux bases de données dans Azure SQL Database ou Azure SQL Managed Instance. Aucune configuration spéciale n’est nécessaire. Pour l’ensemble des connexions à une instance SQL Server, une base de données dans Azure SQL Database ou une instance dans Azure SQL Managed Instance, nous recommandons que toutes les applications définissent les configurations suivantes ou leurs équivalents :

- **Encrypt = On**
- **TrustServerCertificate = Off**

Certains systèmes utilisent des mots clés différents, mais équivalents pour ces configurations. Ces configurations garantissent que le pilote du client vérifie l’identité du certificat TLS provenant du serveur.

Nous vous recommandons également de désactiver les protocoles TLS 1.1 et 1.0 sur le client si vous devez respecter la norme de sécurité des données de l’industrie des cartes de paiement (PCI - DSS).

Les pilotes autres que Microsoft peuvent ne pas utiliser le protocole TLS par défaut. Cela peut être un facteur à prendre en compte lors de la connexion à Azure SQL Database ou Azure SQL Managed Instance. Il se peut que les applications présentent des pilotes incorporés qui ne permettent pas de contrôler ces paramètres de connexion. Nous vous recommandons d’examiner la sécurité de ces pilotes et applications avant de les utiliser sur les systèmes qui interagissent avec des données sensibles.

## <a name="libraries"></a>Bibliothèques

Vous avez le choix entre plusieurs frameworks et bibliothèques pour la connexion à Azure SQL Database ou Azure SQL Managed Instance. Découvrez nos [didacticiels de démarrage](https://aka.ms/sqldev) pour commencer rapidement à utiliser des langages de programmation comme C#, Java, Node.js, PHP et Python. Ensuite, créez une application à l’aide de SQL Server sur Linux, Windows ou Docker sur macOS.

Le tableau suivant répertorie les bibliothèques de connectivité ou *pilotes* que les applications clientes peuvent utiliser dans une variété de langages pour se connecter à SQL Server et l’utiliser localement ou dans le cloud. Vous pouvez les utiliser sur Linux, Windows ou Docker pour vous connecter à Azure SQL Database, Azure SQL Managed Instance et Azure Synapse Analytics.

| Langage | Plateforme | Ressources supplémentaires | Téléchargement | Bien démarrer |
| :-- | :-- | :-- | :-- | :-- |
| C# | Windows, Linux, macOS | [Microsoft ADO.NET pour SQL Server](/sql/connect/ado-net/microsoft-ado-net-sql-server) | [Télécharger](https://www.microsoft.com/net/download/) | [Prise en main](https://www.microsoft.com/sql-server/developer-get-started/csharp/ubuntu)
| Java | Windows, Linux, macOS | [Pilote Microsoft JDBC pour SQL Server](/sql/connect/jdbc/microsoft-jdbc-driver-for-sql-server/) | [Télécharger](https://go.microsoft.com/fwlink/?linkid=852460) |  [Prise en main](https://www.microsoft.com/sql-server/developer-get-started/java/ubuntu)
| PHP | Windows, Linux, macOS| [Pilote PHP SQL pour SQL Server](/sql/connect/php/microsoft-php-driver-for-sql-server) | [Télécharger](/sql/connect/php/download-drivers-php-sql-server) | [Prise en main](https://www.microsoft.com/sql-server/developer-get-started/php/ubuntu/)
| Node.js | Windows, Linux, macOS | [Pilote Node.js pour SQL Server](/sql/connect/node-js/node-js-driver-for-sql-server/) | [Installer](/sql/connect/node-js/step-1-configure-development-environment-for-node-js-development/) |  [Prise en main](https://www.microsoft.com/sql-server/developer-get-started/node/ubuntu)
| Python | Windows, Linux, macOS | [Pilote Python SQL](/sql/connect/python/python-driver-for-sql-server/) | Choix d’installation : <br/> \* [pymssql](/sql/connect/python/pymssql/step-1-configure-development-environment-for-pymssql-python-development/) <br/> \* [pyodbc](/sql/connect/python/pyodbc/step-1-configure-development-environment-for-pyodbc-python-development/) |  [Prise en main](https://www.microsoft.com/sql-server/developer-get-started/python/ubuntu)
| Ruby | Windows, Linux, macOS | [Pilote Ruby pour SQL Server](/sql/connect/ruby/ruby-driver-for-sql-server/) | [Installer](/sql/connect/ruby/step-1-configure-development-environment-for-ruby-development/) | [Prise en main](https://www.microsoft.com/sql-server/developer-get-started/ruby/ubuntu)
| C++ | Windows, Linux, macOS | [Pilote Microsoft ODBC pour SQL Server](/sql/connect/odbc/microsoft-odbc-driver-for-sql-server/) | [Télécharger](/sql/connect/odbc/microsoft-odbc-driver-for-sql-server/) |  

Le tableau suivant répertorie des exemples de frameworks de mappage relationnel objet (ORM) et de frameworks web que les applications clientes peuvent utiliser avec SQL Server, Azure SQL Database, Azure SQL Managed Instance ou Azure Synapse Analytics. Vous pouvez utiliser les frameworks sur Linux, Windows ou Docker.

| Langage | Plateforme | ORM |
| :-- | :-- | :-- |
| C# | Windows, Linux, macOS | [Entity Framework](/ef)<br>[Entity Framework Core](/ef/core/index) |
| Java | Windows, Linux, macOS |[Mettre en veille prolongée](https://hibernate.org/orm)|
| PHP | Windows, Linux, macOS | [Laravel (Eloquent)](https://laravel.com/docs/eloquent)<br>[Doctrine](https://www.doctrine-project.org/projects/orm.html) |
| Node.js | Windows, Linux, macOS | [Sequelize ORM](https://sequelize.org/) |
| Python | Windows, Linux, macOS |[Django](https://www.djangoproject.com/) |
| Ruby | Windows, Linux, macOS | [Ruby on rails](https://rubyonrails.org/) |
||||

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur l’architecture de connectivité, consultez [Architecture de connectivité Azure SQL Database](connectivity-architecture.md).
- Trouvez les [pilotes SQL Server](/sql/connect/sql-connection-libraries/) utilisés pour établir une connexion à partir d’applications clientes.
- Établissez une connexion à Azure SQL Database ou Azure SQL Managed Instance :
  - [Connexion et interrogation en utilisant .NET (C#)](connect-query-dotnet-core.md)
  - [Connexion et interrogation en utilisant PHP](connect-query-php.md)
  - [Connexion et interrogation en utilisant Node.js](connect-query-nodejs.md)
  - [Connexion et interrogation en utilisant Java](connect-query-java.md)
  - [Connexion et interrogation en utilisant Python](connect-query-python.md)
  - [Connexion et interrogation en utilisant Ruby](connect-query-ruby.md)
- Exemples de code de logique de nouvelle tentative :
  - [Connexion résiliente avec ADO.NET][step-4-connect-resiliently-to-sql-with-ado-net-a78n]
  - [Connexion résiliente avec PHP][step-4-connect-resiliently-to-sql-with-php-p42h]

<!-- Link references. -->

[step-4-connect-resiliently-to-sql-with-ado-net-a78n]: /sql/connect/ado-net/step-4-connect-resiliently-sql-ado-net

[step-4-connect-resiliently-to-sql-with-php-p42h]: /sql/connect/php/step-4-connect-resiliently-to-sql-with-php