---
title: 'Démarrage rapide : Se connecter à l’aide de C++ – Azure Database pour MySQL'
description: Ce guide de démarrage rapide fournit un exemple de code C++ que vous pouvez utiliser pour vous connecter à la base de données Azure pour MySQL et pour interroger les données de cette dernière.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom: mvc
ms.devlang: cpp
ms.topic: quickstart
ms.date: 5/26/2020
ms.openlocfilehash: e8b2842300e43a9de88cc0b9b3ae9ce6cd754612
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94535841"
---
# <a name="quickstart-use-connectorc-to-connect-and-query-data-in-azure-database-for-mysql"></a>Démarrage rapide : Utiliser Connector/C++ pour vous connecter et interroger des données dans Azure Database pour MySQL

Ce guide de démarrage rapide explique comment se connecter à une base de données Azure Database pour MySQL à l’aide d’une application C#. Il détaille l’utilisation d’instructions SQL pour interroger la base de données, la mettre à jour, y insérer des données ou en supprimer. Cette rubrique part du principe que vous connaissez les bases du développement C++ et que vous ne savez pas utiliser Azure Database pour MySQL.

## <a name="prerequisites"></a>Prérequis

Ce guide de démarrage rapide s’appuie sur les ressources créées dans l’un des guides suivants :
- [Créer un serveur de base de données Azure pour MySQL à l’aide du Portail Azure](./quickstart-create-mysql-server-database-using-azure-portal.md)
- [Création d’un serveur de base de données Azure pour MySQL à l’aide d’Azure CLI](./quickstart-create-mysql-server-database-using-azure-cli.md)

Il vous faudra également :
- Installer [.NET Framework](https://www.microsoft.com/net/download)
- Installez [Visual Studio](https://www.visualstudio.com/downloads/)
- Installer [MySQL Connector/C++](https://dev.mysql.com/downloads/connector/cpp/) 
- Installer [Boost](https://www.boost.org/)

> [!IMPORTANT] 
> Vérifiez que l’adresse IP à partir de laquelle vous vous connectez a été ajoutée aux règles de pare-feu du serveur à l’aide du [portail Azure](./howto-manage-firewall-using-portal.md) ou [d’Azure CLI](./howto-manage-firewall-using-cli.md)

## <a name="install-visual-studio-and-net"></a>Installer Visual Studio et .NET
Les étapes de cette section supposent que vous êtes familiarisé avec l’utilisation de .NET.

### <a name="windows"></a>**Windows**
- Installez Visual Studio 2019 Community. Visual Studio 2019 Community est un environnement de développement intégré (IDE) complet, extensible et gratuit. Cet IDE vous permet de créer des applications modernes pour Android, iOS et Windows, des applications web et de base de données, et des services cloud. Vous pouvez installer la totalité de .NET Framework ou simplement .NET Core : les extraits de code du guide de démarrage rapide fonctionnent avec les deux. Si votre ordinateur est déjà équipé de Visual Studio, ignorez les deux étapes suivantes.
   1. Téléchargez le [programme d’installation de Visual Studio 2019](https://www.visualstudio.com/thank-you-downloading-visual-studio/?sku=Community&rel=15). 
   2. Exécutez le programme d’installation et suivez que les invites.

### <a name="configure-visual-studio"></a>**Configurer Visual Studio**
1. Dans Visual Studio, Projet -> Propriétés -> Éditeur de liens -> Général > Répertoires de bibliothèques supplémentaires, ajoutez le répertoire « \lib\opt » (par exemple : C:\Program Files (x86)\MySQL\MySQL Connector C++ 1.1.9\lib\opt) du connecteur C++.
2. Dans Visual Studio, Projet -> Propriétés -> C/C++ -> Général > Autres répertoires include :
   - Ajoutez le répertoire « \include » du connecteur c++ (par exemple : C:\Program Files (x86)\MySQL\MySQL Connector C++ 1.1.9\include\).
   - Ajoutez le répertoire racine de la bibliothèque Boost (par exemple : C:\boost_1_64_0\).
3. Dans Visual Studio, Projet -> Propriétés -> Éditeur de liens -> Entrée -> Dépendances supplémentaires, ajoutez **mysqlcppconn.lib** dans le champ de texte.
4. Copiez le fichier **mysqlcppconn.dll** du dossier de la bibliothèque de connecteurs C++ de l’étape 3 vers le même répertoire que l’exécutable de l’application, ou ajoutez-le à la variable d’environnement pour permettre à votre application de le retrouver.

## <a name="get-connection-information"></a>Obtenir des informations de connexion
Obtenez les informations requises pour vous connecter à la base de données Azure pour MySQL. Vous devez disposer du nom de serveur complet et des informations d’identification.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans le menu de gauche du portail Azure, cliquez sur **Toutes les ressources**, puis recherchez le serveur que vous venez de créer, par exemple **mydemoserver**.
3. Cliquez sur le nom du serveur.
4. Dans le panneau **Vue d’ensemble** du serveur, notez le **nom du serveur** et le **nom de connexion de l’administrateur du serveur**. Si vous oubliez votre mot de passe, vous pouvez également le réinitialiser dans ce panneau.
 :::image type="content" source="./media/connect-cpp/1_server-overview-name-login.png" alt-text="Nom du serveur de base de données Azure pour MySQL":::

## <a name="connect-create-table-and-insert-data"></a>Se connecter, créer des tables et insérer des données
Utilisez le code suivant pour vous connecter et charger les données à l’aide d’instructions SQL **CREATE TABLE** et **INSERT INTO**. Le code utilise la classe sql::Driver avec la méthode connect() pour établir une connexion à MySQL. Le code utilise ensuite les méthodes createStatement() et execute() pour exécuter les commandes de base de données. 

Remplacez les valeurs des paramètres Hôte, DBName, Utilisateur et Mot de passe. Vous pouvez remplacer les paramètres par les valeurs que vous avez spécifiées lorsque vous avez créé le serveur et la base de données. 

```c++
#include <stdlib.h>
#include <iostream>
#include "stdafx.h"

#include "mysql_connection.h"
#include <cppconn/driver.h>
#include <cppconn/exception.h>
#include <cppconn/prepared_statement.h>
using namespace std;

//for demonstration only. never save your password in the code!
const string server = "tcp://yourservername.mysql.database.azure.com:3306";
const string username = "username@servername";
const string password = "yourpassword";

int main()
{
    sql::Driver *driver;
    sql::Connection *con;
    sql::Statement *stmt;
    sql::PreparedStatement *pstmt;

    try
    {
        driver = get_driver_instance();
        con = driver->connect(server, username, password);
    }
    catch (sql::SQLException e)
    {
        cout << "Could not connect to server. Error message: " << e.what() << endl;
        system("pause");
        exit(1);
    }

    //please create database "quickstartdb" ahead of time
    con->setSchema("quickstartdb");

    stmt = con->createStatement();
    stmt->execute("DROP TABLE IF EXISTS inventory");
    cout << "Finished dropping table (if existed)" << endl;
    stmt->execute("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);");
    cout << "Finished creating table" << endl;
    delete stmt;

    pstmt = con->prepareStatement("INSERT INTO inventory(name, quantity) VALUES(?,?)");
    pstmt->setString(1, "banana");
    pstmt->setInt(2, 150);
    pstmt->execute();
    cout << "One row inserted." << endl;

    pstmt->setString(1, "orange");
    pstmt->setInt(2, 154);
    pstmt->execute();
    cout << "One row inserted." << endl;

    pstmt->setString(1, "apple");
    pstmt->setInt(2, 100);
    pstmt->execute();
    cout << "One row inserted." << endl;

    delete pstmt;
    delete con;
    system("pause");
    return 0;
}
```

## <a name="read-data"></a>Lire les données

Utilisez le code suivant pour vous connecter et lire des données à l’aide d’une instruction SQL **SELECT**. Le code utilise la classe sql::Driver avec la méthode connect() pour établir une connexion à MySQL. Ensuite, le code utilise les méthodes prepareStatement() et executeQuery() pour exécuter les commandes de sélection. Il utilise alors next() pour accéder aux enregistrements suivants dans les résultats et, enfin, getInt() et getString() pour analyser les valeurs de l’enregistrement.

Remplacez les valeurs des paramètres Hôte, DBName, Utilisateur et Mot de passe. Vous pouvez remplacer les paramètres par les valeurs que vous avez spécifiées lorsque vous avez créé le serveur et la base de données. 

```c++
#include <stdlib.h>
#include <iostream>
#include "stdafx.h"

#include "mysql_connection.h"
#include <cppconn/driver.h>
#include <cppconn/exception.h>
#include <cppconn/resultset.h>
#include <cppconn/prepared_statement.h>
using namespace std;

//for demonstration only. never save your password in the code!
const string server = "tcp://yourservername.mysql.database.azure.com:3306";
const string username = "username@servername";
const string password = "yourpassword";

int main()
{
    sql::Driver *driver;
    sql::Connection *con;
    sql::PreparedStatement *pstmt;
    sql::ResultSet *result;

    try
    {
        driver = get_driver_instance();
        //for demonstration only. never save password in the code!
        con = driver->connect(server, username, password);
    }
    catch (sql::SQLException e)
    {
        cout << "Could not connect to server. Error message: " << e.what() << endl;
        system("pause");
        exit(1);
    }

    con->setSchema("quickstartdb");

    //select  
    pstmt = con->prepareStatement("SELECT * FROM inventory;");
    result = pstmt->executeQuery();

    while (result->next())
        printf("Reading from table=(%d, %s, %d)\n", result->getInt(1), result->getString(2).c_str(), result->getInt(3));

    delete result;
    delete pstmt;
    delete con;
    system("pause");
    return 0;
}
```

## <a name="update-data"></a>Mettre à jour des données
Utilisez le code suivant pour vous connecter et lire les données à l’aide d’une instruction SQL **UPDATE**. Le code utilise la classe sql::Driver avec la méthode connect() pour établir une connexion à MySQL. Ensuite, le code utilise les méthodes prepareStatement() et executeQuery() pour exécuter les commandes de mise à jour. 

Remplacez les valeurs des paramètres Hôte, DBName, Utilisateur et Mot de passe. Vous pouvez remplacer les paramètres par les valeurs que vous avez spécifiées lorsque vous avez créé le serveur et la base de données. 

```c++
#include <stdlib.h>
#include <iostream>
#include "stdafx.h"

#include "mysql_connection.h"
#include <cppconn/driver.h>
#include <cppconn/exception.h>
#include <cppconn/resultset.h>
#include <cppconn/prepared_statement.h>
using namespace std;

//for demonstration only. never save your password in the code!
const string server = "tcp://yourservername.mysql.database.azure.com:3306";
const string username = "username@servername";
const string password = "yourpassword";

int main()
{
    sql::Driver *driver;
    sql::Connection *con;
    sql::PreparedStatement *pstmt;

    try
    {
        driver = get_driver_instance();
        //for demonstration only. never save password in the code!
        con = driver->connect(server, username, password);
    }
    catch (sql::SQLException e)
    {
        cout << "Could not connect to server. Error message: " << e.what() << endl;
        system("pause");
        exit(1);
    }
    
    con->setSchema("quickstartdb");

    //update
    pstmt = con->prepareStatement("UPDATE inventory SET quantity = ? WHERE name = ?");
    pstmt->setInt(1, 200);
    pstmt->setString(2, "banana");
    pstmt->executeQuery();
    printf("Row updated\n");

    delete con;
    delete pstmt;
    system("pause");
    return 0;
}
```


## <a name="delete-data"></a>Suppression de données
Utilisez le code suivant pour vous connecter et lire les données à l’aide d’une instruction SQL **DELETE**. Le code utilise la classe sql::Driver avec la méthode connect() pour établir une connexion à MySQL. Ensuite, le code utilise les méthodes prepareStatement() et executeQuery() pour exécuter les commandes de suppression.

Remplacez les valeurs des paramètres Hôte, DBName, Utilisateur et Mot de passe. Vous pouvez remplacer les paramètres par les valeurs que vous avez spécifiées lorsque vous avez créé le serveur et la base de données. 

```c++
#include <stdlib.h>
#include <iostream>
#include "stdafx.h"

#include "mysql_connection.h"
#include <cppconn/driver.h>
#include <cppconn/exception.h>
#include <cppconn/resultset.h>
#include <cppconn/prepared_statement.h>
using namespace std;

//for demonstration only. never save your password in the code!
const string server = "tcp://yourservername.mysql.database.azure.com:3306";
const string username = "username@servername";
const string password = "yourpassword";

int main()
{
    sql::Driver *driver;
    sql::Connection *con;
    sql::PreparedStatement *pstmt;
    sql::ResultSet *result;

    try
    {
        driver = get_driver_instance();
        //for demonstration only. never save password in the code!
        con = driver->connect(server, username, password);
    }
    catch (sql::SQLException e)
    {
        cout << "Could not connect to server. Error message: " << e.what() << endl;
        system("pause");
        exit(1);
    }
    
    con->setSchema("quickstartdb");
        
    //delete
    pstmt = con->prepareStatement("DELETE FROM inventory WHERE name = ?");
    pstmt->setString(1, "orange");
    result = pstmt->executeQuery();
    printf("Row deleted\n");    
    
    delete pstmt;
    delete con;
    delete result;
    system("pause");
    return 0;
}
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour nettoyer toutes les ressources utilisées dans le cadre de ce guide de démarrage rapide, supprimez le groupe de ressources à l’aide de la commande suivante :

```azurecli
az group delete \
    --name $AZ_RESOURCE_GROUP \
    --yes
```

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Migrer une base de données MySQL vers une base de données Azure pour MySQL à l’aide des images mémoire et de la restauration](concepts-migrate-dump-restore.md)
