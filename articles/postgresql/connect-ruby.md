---
title: 'Démarrage rapide : Se connecter avec Ruby – Azure Database pour PostgreSQL – Serveur unique'
description: Ce démarrage rapide fournit un exemple de code Ruby que vous pouvez utiliser pour vous connecter et interroger des données à partir d’Azure Database pour PostgreSQL (serveur unique).
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.custom: mvc
ms.devlang: ruby
ms.topic: quickstart
ms.date: 5/6/2019
ms.openlocfilehash: 20e45454284af230da74896d5b3f5e9da676dbb4
ms.sourcegitcommit: beacda0b2b4b3a415b16ac2f58ddfb03dd1a04cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97831701"
---
# <a name="quickstart-use-ruby-to-connect-and-query-data-in-azure-database-for-postgresql---single-server"></a>Démarrage rapide : Utiliser Ruby afin de se connecter à Azure Database pour PostgreSQL et d'interroger les données - Serveur unique

Ce guide de démarrage rapide vous explique comment vous connecter à une base de données Azure pour PostgreSQL en utilisant une application [Ruby](https://www.ruby-lang.org). Il détaille l’utilisation d’instructions SQL pour interroger la base de données, la mettre à jour, y insérer des données ou en supprimer. Cet article suppose que vous connaissez les bases du développement via Ruby, et que vous ne savez pas utiliser Azure Database pour PostgreSQL.

## <a name="prerequisites"></a>Prérequis
Ce guide de démarrage rapide s’appuie sur les ressources créées dans l’un de ces guides :
- [Créer une base de données - Portail](quickstart-create-server-database-portal.md)
- [Créer une base de données - Interface de ligne de commande Azure](quickstart-create-server-database-azure-cli.md)

Vous devez également avoir installé :
- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [Ruby pg](https://rubygems.org/gems/pg/), le module PostgreSQL pour Ruby

## <a name="get-connection-information"></a>Obtenir des informations de connexion
Obtenez les informations de connexion requises pour vous connecter à la base de données Azure pour PostgreSQL. Vous devez disposer du nom de serveur complet et des informations d’identification.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans le menu de gauche du portail Azure, cliquez sur **Toutes les ressources**, puis recherchez le serveur que vous venez de créer, par exemple **mydemoserver**.
3. Cliquez sur le nom du serveur.
4. Dans le panneau **Vue d’ensemble** du serveur, notez le **nom du serveur** et le **nom de connexion de l’administrateur du serveur**. Si vous oubliez votre mot de passe, vous pouvez également le réinitialiser dans ce panneau.
 :::image type="content" source="./media/connect-ruby/1-connection-string.png" alt-text="Nom du serveur Azure Database pour PostgreSQL":::

> [!NOTE]
> Le symbole `@` dans le nom d’utilisateur Azure Postgres a été codé URL en tant que `%40` dans toutes les chaînes de connexion.

## <a name="connect-and-create-a-table"></a>Se connecter et créer une table
Utilisez le code suivant pour vous connecter et créer une table à l’aide de l’instruction **CREATE TABLE**, suivie des instructions SQL **INSERT INTO** pour ajouter des lignes à la table.

Le code utilise un objet ```PG::Connection``` avec le constructeur ```new``` pour se connecter à Azure Database pour PostgreSQL. Ensuite, il appelle la méthode ```exec()``` pour exécuter les commandes DROP, CREATE TABLE et INSERT INTO. Le code vérifie les erreurs à l’aide de la classe ```PG::Error```. Ensuite, il appelle la méthode ```close()``` pour fermer la connexion, avant de s’arrêter. Pour plus d'informations sur ces classes et méthodes, consultez la documentation de référence de Ruby PG.

Remplacez les chaînes `host`, `database`, `user` et `password` par vos propres valeurs.


```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin%40mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :dbname => database, :port => '5432', :password => password)
    puts 'Successfully created connection to database'

    # Drop previous table of same name if one exists
    connection.exec('DROP TABLE IF EXISTS inventory;')
    puts 'Finished dropping table (if existed).'

    # Drop previous table of same name if one exists.
    connection.exec('CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);')
    puts 'Finished creating table.'

    # Insert some data into table.
    connection.exec("INSERT INTO inventory VALUES(1, 'banana', 150)")
    connection.exec("INSERT INTO inventory VALUES(2, 'orange', 154)")
    connection.exec("INSERT INTO inventory VALUES(3, 'apple', 100)")
    puts 'Inserted 3 rows of data.'

rescue PG::Error => e
    puts e.message

ensure
    connection.close if connection
end
```

## <a name="read-data"></a>Lire les données
Utilisez le code suivant pour vous connecter et lire des données à l’aide d’une instruction SQL **SELECT**.

Le code utilise un objet ```PG::Connection``` objet avec le constructeur ```new``` pour se connecter à la base de données Azure pour PostgreSQL. Ensuite, il appelle la méthode ```exec()``` pour exécuter la commande SELECT, en conservant les résultats dans un jeu de résultats. La collection de jeu de résultats est itérée au sein de la boucle `resultSet.each do`, les valeurs de ligne actuelles étant conservées dans la variable `row`. Le code vérifie les erreurs à l’aide de la classe ```PG::Error```. Ensuite, il appelle la méthode ```close()``` pour fermer la connexion, avant de s’arrêter. Pour plus d'informations sur ces classes et méthodes, consultez la documentation de référence de Ruby PG.

Remplacez les chaînes `host`, `database`, `user` et `password` par vos propres valeurs.

```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin%40mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :database => dbname, :port => '5432', :password => password)
    puts 'Successfully created connection to database.'

    resultSet = connection.exec('SELECT * from inventory;')
    resultSet.each do |row|
        puts 'Data row = (%s, %s, %s)' % [row['id'], row['name'], row['quantity']]
    end

rescue PG::Error => e
    puts e.message

ensure
    connection.close if connection
end
```

## <a name="update-data"></a>Mettre à jour des données
Utilisez le code suivant pour vous connecter et mettre à jour les données à l’aide d’une instruction SQL **UPDATE**.

Le code utilise un objet ```PG::Connection``` objet avec le constructeur ```new``` pour se connecter à la base de données Azure pour PostgreSQL. Ensuite, il appelle la méthode ```exec()``` pour exécuter la commande UPDATE. Le code vérifie les erreurs à l’aide de la classe ```PG::Error```. Ensuite, il appelle la méthode ```close()``` pour fermer la connexion, avant de s’arrêter. Pour plus d'informations sur ces classes et méthodes, consultez la [documentation de référence de Ruby PG](https://rubygems.org/gems/pg).

Remplacez les chaînes `host`, `database`, `user` et `password` par vos propres valeurs.

```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin%40mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :dbname => database, :port => '5432', :password => password)
    puts 'Successfully created connection to database.'

    # Modify some data in table.
    connection.exec('UPDATE inventory SET quantity = %d WHERE name = %s;' % [200, '\'banana\''])
    puts 'Updated 1 row of data.'

rescue PG::Error => e
    puts e.message

ensure
    connection.close if connection
end
```


## <a name="delete-data"></a>Suppression de données
Utilisez le code suivant pour vous connecter et lire des données à l’aide d’une instruction SQL **DELETE**.

Le code utilise un objet ```PG::Connection``` objet avec le constructeur ```new``` pour se connecter à la base de données Azure pour PostgreSQL. Ensuite, il appelle la méthode ```exec()``` pour exécuter la commande UPDATE. Le code vérifie les erreurs à l’aide de la classe ```PG::Error```. Ensuite, il appelle la méthode ```close()``` pour fermer la connexion, avant de s’arrêter.

Remplacez les chaînes `host`, `database`, `user` et `password` par vos propres valeurs.

```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin%40mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :dbname => database, :port => '5432', :password => password)
    puts 'Successfully created connection to database.'

    # Modify some data in table.
    connection.exec('DELETE FROM inventory WHERE name = %s;' % ['\'orange\''])
    puts 'Deleted 1 row of data.'

rescue PG::Error => e
    puts e.message

ensure
    connection.close if connection
end
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
> [Migration de votre base de données PostgreSQL par exportation et importation](./howto-migrate-using-export-and-import.md) <br/>
> [!div class="nextstepaction"]
> [Documentation de référence de Ruby Pg](https://rubygems.org/gems/pg)
