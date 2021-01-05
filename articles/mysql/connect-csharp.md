---
title: 'Démarrage rapide : Se connecter à l’aide de C# – Azure Database pour MySQL'
description: Ce guide de démarrage rapide fournit un exemple de code C# (.NET) que vous pouvez utiliser pour vous connecter et interroger des données de la base de données Azure pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom: mvc, devx-track-csharp
ms.devlang: csharp
ms.topic: quickstart
ms.date: 10/18/2020
ms.openlocfilehash: 3b90d8819b5d327c3ccd143257198c7ec8538f03
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94535824"
---
# <a name="quickstart-use-net-c-to-connect-and-query-data-in-azure-database-for-mysql"></a>Démarrage rapide : Utilisez .NET (C#) pour vous connecter et interroger des données dans Azure Database pour MySQL

Ce guide de démarrage rapide explique comment vous connecter à Azure Database pour MySQL en utilisant une application C#. Il détaille l’utilisation d’instructions SQL pour interroger la base de données, la mettre à jour, y insérer des données ou en supprimer. 

## <a name="prerequisites"></a>Prérequis
Voici les prérequis pour ce guide de démarrage rapide :

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free).
- Créer un serveur unique Azure Database pour MySQL à l’aide du [portail Azure](./quickstart-create-mysql-server-database-using-azure-portal.md) <br/> ou d’[Azure CLI](./quickstart-create-mysql-server-database-using-azure-cli.md), si vous n’en avez pas.
- Selon que vous utilisez un accès public ou privé, effectuez **UNE** des actions ci-dessous pour activer la connectivité.

|Action| Méthode de connexion|Guide pratique|
|:--------- |:--------- |:--------- |
| **Configurer les règles de pare-feu** | Public | [Portail](./howto-manage-firewall-using-portal.md) <br/> [INTERFACE DE LIGNE DE COMMANDE](./howto-manage-firewall-using-cli.md)|
| **Configurer le point de terminaison de service** | Public | [Portail](./howto-manage-vnet-using-portal.md) <br/> [INTERFACE DE LIGNE DE COMMANDE](./howto-manage-vnet-using-cli.md)| 
| **Configurer une liaison privée** | Privées | [Portail](./howto-configure-privatelink-portal.md) <br/> [INTERFACE DE LIGNE DE COMMANDE](./howto-configure-privatelink-cli.md) | 

- [Créer une base de données et un utilisateur non-administrateur](./howto-create-users.md)

[Vous rencontrez des problèmes ? Faites-le nous savoir](https://aka.ms/mysql-doc-feedback)

## <a name="create-a-c-project"></a>Créer un projet C#
À l’invite de commandes, exécutez :

```
mkdir AzureMySqlExample
cd AzureMySqlExample
dotnet new console
dotnet add package MySqlConnector
```

## <a name="get-connection-information"></a>Obtenir des informations de connexion
Obtenez les informations requises pour vous connecter à la base de données Azure pour MySQL. Vous devez disposer du nom de serveur complet et des informations d’identification.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans le menu de gauche du portail Azure, cliquez sur **Toutes les ressources**, puis recherchez le serveur que vous venez de créer, par exemple **mydemoserver**.
3. Cliquez sur le nom du serveur.
4. Dans le panneau **Vue d’ensemble** du serveur, notez le **nom du serveur** et le **nom de connexion de l’administrateur du serveur**. Si vous oubliez votre mot de passe, vous pouvez également le réinitialiser dans ce panneau.
 :::image type="content" source="./media/connect-csharp/1_server-overview-name-login.png" alt-text="Nom du serveur de base de données Azure pour MySQL":::

## <a name="step-1-connect-and-insert-data"></a>Étape 1 : Vous connecter et insérer des données
Utilisez le code suivant pour vous connecter et charger les données à l’aide des instructions SQL `CREATE TABLE` et `INSERT INTO`. Le code utilise les méthodes de la classe `MySqlConnection` :
- [OpenAsync()](/dotnet/api/system.data.common.dbconnection.openasync#System_Data_Common_DbConnection_OpenAsync) pour établir une connexion à MySQL.
- [CreateCommand()](/dotnet/api/system.data.common.dbconnection.createcommand) pour définir la propriété CommandText.
- [ExecuteNonQueryAsync()](/dotnet/api/system.data.common.dbcommand.executenonqueryasync) pour exécuter les commandes de base de données. 

Remplacez les paramètres `Server`, `Database`, `UserID` et `Password` par les valeurs que vous avez spécifiées lorsque vous avez créé le serveur et la base de données. 

```csharp
using System;
using System.Threading.Tasks;
using MySqlConnector;

namespace AzureMySqlExample
{
    class MySqlCreate
    {
        static async Task Main(string[] args)
        {
            var builder = new MySqlConnectionStringBuilder
            {
                Server = "YOUR-SERVER.mysql.database.azure.com",
                Database = "YOUR-DATABASE",
                UserID = "USER@YOUR-SERVER",
                Password = "PASSWORD",
                SslMode = MySqlSslMode.Required,
            };

            using (var conn = new MySqlConnection(builder.ConnectionString))
            {
                Console.WriteLine("Opening connection");
                await conn.OpenAsync();

                using (var command = conn.CreateCommand())
                {
                    command.CommandText = "DROP TABLE IF EXISTS inventory;";
                    await command.ExecuteNonQueryAsync();
                    Console.WriteLine("Finished dropping table (if existed)");

                    command.CommandText = "CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);";
                    await command.ExecuteNonQueryAsync();
                    Console.WriteLine("Finished creating table");

                    command.CommandText = @"INSERT INTO inventory (name, quantity) VALUES (@name1, @quantity1),
                        (@name2, @quantity2), (@name3, @quantity3);";
                    command.Parameters.AddWithValue("@name1", "banana");
                    command.Parameters.AddWithValue("@quantity1", 150);
                    command.Parameters.AddWithValue("@name2", "orange");
                    command.Parameters.AddWithValue("@quantity2", 154);
                    command.Parameters.AddWithValue("@name3", "apple");
                    command.Parameters.AddWithValue("@quantity3", 100);

                    int rowCount = await command.ExecuteNonQueryAsync();
                    Console.WriteLine(String.Format("Number of rows inserted={0}", rowCount));
                }

                // connection will be closed by the 'using' block
                Console.WriteLine("Closing connection");
            }

            Console.WriteLine("Press RETURN to exit");
            Console.ReadLine();
        }
    }
}
```

[Vous rencontrez des problèmes ? Faites-le nous savoir](https://aka.ms/mysql-doc-feedback)


## <a name="step-2-read-data"></a>Étape 2 : Lire les données

Utilisez le code suivant pour vous connecter et lire les données à l’aide d’une instruction SQL `SELECT`. Le code utilise la classe `MySqlConnection` avec les méthodes :
- [OpenAsync()](/dotnet/api/system.data.common.dbconnection.openasync#System_Data_Common_DbConnection_OpenAsync) pour établir une connexion à MySQL.
- [CreateCommand()](/dotnet/api/system.data.common.dbconnection.createcommand) pour définir la propriété CommandText.
- [ExecuteReaderAsync()](/dotnet/api/system.data.common.dbcommand.executereaderasync) pour exécuter les commandes de base de données. 
- [ReadAsync()](/dotnet/api/system.data.common.dbdatareader.readasync#System_Data_Common_DbDataReader_ReadAsync) pour accéder aux enregistrements contenus dans les résultats. Ensuite, le code utilise GetInt32 et GetString pour analyser les valeurs de l’enregistrement.


Remplacez les paramètres `Server`, `Database`, `UserID` et `Password` par les valeurs que vous avez spécifiées lorsque vous avez créé le serveur et la base de données. 

```csharp
using System;
using System.Threading.Tasks;
using MySqlConnector;

namespace AzureMySqlExample
{
    class MySqlRead
    {
        static async Task Main(string[] args)
        {
            var builder = new MySqlConnectionStringBuilder
            {
                Server = "YOUR-SERVER.mysql.database.azure.com",
                Database = "YOUR-DATABASE",
                UserID = "USER@YOUR-SERVER",
                Password = "PASSWORD",
                SslMode = MySqlSslMode.Required,
            };

            using (var conn = new MySqlConnection(builder.ConnectionString))
            {
                Console.WriteLine("Opening connection");
                await conn.OpenAsync();

                using (var command = conn.CreateCommand())
                {
                    command.CommandText = "SELECT * FROM inventory;";

                    using (var reader = await command.ExecuteReaderAsync())
                    {
                        while (await reader.ReadAsync())
                        {
                            Console.WriteLine(string.Format(
                                "Reading from table=({0}, {1}, {2})",
                                reader.GetInt32(0),
                                reader.GetString(1),
                                reader.GetInt32(2)));
                        }
                    }
                }

                Console.WriteLine("Closing connection");
            }

            Console.WriteLine("Press RETURN to exit");
            Console.ReadLine();
        }
    }
}
```

[Vous rencontrez des problèmes ? Faites-le nous savoir](https://aka.ms/mysql-doc-feedback)

## <a name="step-3-update-data"></a>Étape 3 : Mettre à jour des données
Utilisez le code suivant pour vous connecter et lire les données à l’aide d’une instruction SQL `UPDATE`. Le code utilise la classe `MySqlConnection` avec la méthode :
- [OpenAsync()](/dotnet/api/system.data.common.dbconnection.openasync#System_Data_Common_DbConnection_OpenAsync) pour établir une connexion à MySQL. 
- [CreateCommand()](/dotnet/api/system.data.common.dbconnection.createcommand) pour définir la propriété CommandText.
- [ExecuteNonQueryAsync()](/dotnet/api/system.data.common.dbcommand.executenonqueryasync) pour exécuter les commandes de base de données. 


Remplacez les paramètres `Server`, `Database`, `UserID` et `Password` par les valeurs que vous avez spécifiées lorsque vous avez créé le serveur et la base de données. 

```csharp
using System;
using System.Threading.Tasks;
using MySqlConnector;

namespace AzureMySqlExample
{
    class MySqlUpdate
    {
        static async Task Main(string[] args)
        {
            var builder = new MySqlConnectionStringBuilder
            {
                Server = "YOUR-SERVER.mysql.database.azure.com",
                Database = "YOUR-DATABASE",
                UserID = "USER@YOUR-SERVER",
                Password = "PASSWORD",
                SslMode = MySqlSslMode.Required,
            };

            using (var conn = new MySqlConnection(builder.ConnectionString))
            {
                Console.WriteLine("Opening connection");
                await conn.OpenAsync();

                using (var command = conn.CreateCommand())
                {
                    command.CommandText = "UPDATE inventory SET quantity = @quantity WHERE name = @name;";
                    command.Parameters.AddWithValue("@quantity", 200);
                    command.Parameters.AddWithValue("@name", "banana");

                    int rowCount = await command.ExecuteNonQueryAsync();
                    Console.WriteLine(String.Format("Number of rows updated={0}", rowCount));
                }

                Console.WriteLine("Closing connection");
            }

            Console.WriteLine("Press RETURN to exit");
            Console.ReadLine();
        }
    }
}
```
[Vous rencontrez des problèmes ? Faites-le nous savoir](https://aka.ms/mysql-doc-feedback)

## <a name="step-4-delete-data"></a>Étape 4 : Suppression de données
Utilisez le code suivant pour vous connecter et supprimer les données à l’aide d’une instruction SQL `DELETE`. 

Le code utilise la classe `MySqlConnection` avec la méthode :
- [OpenAsync()](/dotnet/api/system.data.common.dbconnection.openasync#System_Data_Common_DbConnection_OpenAsync) pour établir une connexion à MySQL.
- [CreateCommand()](/dotnet/api/system.data.common.dbconnection.createcommand) pour définir la propriété CommandText.
- [ExecuteNonQueryAsync()](/dotnet/api/system.data.common.dbcommand.executenonqueryasync) pour exécuter les commandes de base de données. 


Remplacez les paramètres `Server`, `Database`, `UserID` et `Password` par les valeurs que vous avez spécifiées lorsque vous avez créé le serveur et la base de données. 

```csharp
using System;
using System.Threading.Tasks;
using MySqlConnector;

namespace AzureMySqlExample
{
    class MySqlDelete
    {
        static async Task Main(string[] args)
        {
            var builder = new MySqlConnectionStringBuilder
            {
                Server = "YOUR-SERVER.mysql.database.azure.com",
                Database = "YOUR-DATABASE",
                UserID = "USER@YOUR-SERVER",
                Password = "PASSWORD",
                SslMode = MySqlSslMode.Required,
            };

            using (var conn = new MySqlConnection(builder.ConnectionString))
            {
                Console.WriteLine("Opening connection");
                await conn.OpenAsync();

                using (var command = conn.CreateCommand())
                {
                    command.CommandText = "DELETE FROM inventory WHERE name = @name;";
                    command.Parameters.AddWithValue("@name", "orange");

                    int rowCount = await command.ExecuteNonQueryAsync();
                    Console.WriteLine(String.Format("Number of rows deleted={0}", rowCount));
                }

                Console.WriteLine("Closing connection");
            }

            Console.WriteLine("Press RETURN to exit");
            Console.ReadLine();
        }
    }
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
> [Gérer un serveur Azure Database pour MySQL à l’aide du portail Azure](./howto-create-manage-server-portal.md)<br/>

> [!div class="nextstepaction"]
> [Gérer un serveur Azure Database pour MySQL à l’aide d’Azure CLI](./how-to-manage-single-server-cli.md)

[Vous ne trouvez pas ce que vous cherchez ? Faites-le nous savoir.](https://aka.ms/mysql-doc-feedback)
