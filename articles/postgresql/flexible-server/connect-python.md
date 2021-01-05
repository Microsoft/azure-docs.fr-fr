---
title: "Démarrage rapide : Se connecter à l'aide de Python - Azure Database pour PostgreSQL - Serveur flexible"
description: Ce guide de démarrage rapide fournit plusieurs exemples de code Python, que vous pouvez utiliser pour vous connecter et interroger des données à partir d'Azure Database pour PostgreSQL - Serveur flexible.
author: ambhatna
ms.author: ambhatna
ms.service: postgresql
ms.custom: mvc
ms.devlang: python
ms.topic: quickstart
ms.date: 09/22/2020
ms.openlocfilehash: 89dc36a9b1b1fee9ad10d55945c7fc17bf72f476
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "90945027"
---
# <a name="quickstart-use-python-to-connect-and-query-data-in-azure-database-for-postgresql---flexible-server"></a>Démarrage rapide : Utiliser Python afin de se connecter à Azure Database pour PostgreSQL - Serveur unique et d'interroger les données

> [!IMPORTANT]
> La fonctionnalité Azure Database pour PostgreSQL - Serveur flexible est disponible en préversion.

Dans ce guide de démarrage rapide, vous allez utiliser Python afin de vous connecter à Azure Database pour PostgreSQL - Serveur flexible. Vous utilisez ensuite des instructions SQL pour interroger, insérer, mettre à jour et supprimer des données dans la base de données depuis des plateformes Mac, Ubuntu Linux et Windows. 

Cet article part du principe que vous connaissez les bases du développement avec Python, mais que vous ne savez pas utiliser Azure Database pour PostgreSQL - Serveur flexible.

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
* Une instance d'Azure Database pour PostgreSQL - Serveur flexible. Pour créer un serveur flexible, consultez [Créer une instance d'Azure Database pour PostgreSQL - Serveur flexible à l'aide du portail Azure](./quickstart-create-server-portal.md).
* [Python](https://www.python.org/downloads/) 2.7.9+ ou 3.4+.
* Dernier programme d’installation de package [pip](https://pip.pypa.io/en/stable/installing/).

## <a name="preparing-your-client-workstation"></a>Préparation de votre station de travail cliente
- Si vous avez créé votre serveur flexible avec l'option *Accès privé (intégration au réseau virtuel)* , vous devez vous connecter à votre serveur à partir d'une ressource qui se trouve au sein du même réseau virtuel que votre serveur. Vous pouvez créer une machine virtuelle et l'ajouter au réseau virtuel créé avec votre serveur flexible. Reportez-vous à [Créer et gérer un réseau virtuel Azure Database pour PostgreSQL - Serveur flexible à l'aide du portail Azure](./how-to-manage-virtual-network-cli.md).
- Si vous avez créé votre serveur flexible avec l'option *Accès public (adresses IP autorisées)* , vous pouvez ajouter votre adresse IP locale à la liste des règles de pare-feu sur votre serveur. Reportez-vous à [Créer et gérer des règles de pare-feu Azure Database pour PostgreSQL - Serveur flexible à l'aide d'Azure CLI](./how-to-manage-firewall-cli.md).

## <a name="install-the-python-libraries-for-postgresql"></a>Installer des bibliothèques Python pour PostgreSQL
Le module [psycopg2](https://pypi.python.org/pypi/psycopg2/) permet de se connecter à une base de données PostgreSQL et de l’interroger, et il est disponible comme package [wheel](https://pythonwheels.com/) Linux, macOS ou Windows. Installez la version binaire du module qui inclut toutes les dépendances. Pour plus d’informations sur l’installation et les exigences `psycopg2`, reportez-vous à [Installation](http://initd.org/psycopg/docs/install.html). 

Pour installer `psycopg2`, ouvrez une invite de commandes ou de terminal et exécutez la commande `pip install psycopg2`.

## <a name="get-database-connection-information"></a>Obtenir des informations de connexion à la base de données
La connexion à une instance d'Azure Database pour PostgreSQL - Serveur flexible nécessite un nom de serveur complet et des informations d'identification de connexion. Vous pouvez vous procurer ces informations à partir du portail Azure.

1. Sur le [portail Azure](https://portal.azure.com/), recherchez et sélectionnez le nom de votre serveur flexible. 
2. Sur la page **Vue d’ensemble** du serveur, copiez le **nom du serveur** complet et le **nom d’utilisateur administrateur**. Le **nom de serveur** complet se présente toujours sous la forme *\<my-server-name>.postgres.database.azure.com*.

   Vous avez aussi besoin de votre mot de passe d’administrateur. Si vous l'avez oublié, vous pouvez le réinitialiser à partir de la page de présentation. 

   <!--![Azure Database for PostgreSQL server name](./media/connect-python/1-connection-string.png)-->

## <a name="how-to-run-the-python-examples"></a>Comment exécuter les exemples Python

Pour chaque exemple de code dans cet article :

1. Créez un fichier dans un éditeur de texte. 

1. Ajoutez l’exemple de code au fichier. Dans le code, remplacez :
   - `<server-name>` et `<admin-username>` par les valeurs copiées à partir du portail Azure.
   - `<admin-password>` avec votre mot de passe du serveur.
   - `<database-name>` par le nom de votre base de données Azure Database pour PostgreSQL - Serveur flexible. Une base de données par défaut nommée *postgres* a été automatiquement créée lorsque vous avez créé votre serveur. Vous pouvez renommer cette base de données ou en créer une en utilisant les commandes SQL. 

1. Enregistrez le fichier dans votre dossier de projets avec l’extension *.py*, comme *postgres-insert.py*. Pour Windows, assurez-vous que l’encodage UTF-8 est sélectionné lorsque vous enregistrez le fichier. 

1. Pour exécuter le fichier, passez à votre dossier de projets dans une interface de ligne de commande, et tapez `python` suivi du nom de fichier, par exemple `python postgres-insert.py`.

## <a name="create-a-table-and-insert-data"></a>Créer une table et insérer des données
L'exemple de code suivant se connecte à votre base de données Azure Database pour PostgreSQL - Serveur flexible à l'aide de la fonction [psycopg2.connect](http://initd.org/psycopg/docs/connection.html), et charge les données avec une instruction SQL **INSERT**. La fonction [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) exécute la requête SQL par rapport à la base de données. 

```Python
import psycopg2
# Update connection string information 
host = "<server-name>"
dbname = "<database-name>"
user = "<admin-username>"
password = "<admin-password>"
sslmode = "require"
# Construct connection string
conn_string = "host={0} user={1} dbname={2} password={3} sslmode={4}".format(host, user, dbname, password, sslmode)
conn = psycopg2.connect(conn_string) 
print("Connection established")
cursor = conn.cursor()
# Drop previous table of same name if one exists
cursor.execute("DROP TABLE IF EXISTS inventory;")
print("Finished dropping table (if existed)")
# Create a table
cursor.execute("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);")
print("Finished creating table")
# Insert some data into the table
cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("banana", 150))
cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("orange", 154))
cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("apple", 100))
print("Inserted 3 rows of data")
# Clean up
conn.commit()
cursor.close()
conn.close()
```

Lorsque le code est exécuté correctement, il produit le résultat suivant :

![Sortie de la ligne de commande](media/connect-python/2-example-python-output.png)

## <a name="read-data"></a>Lire les données
L'exemple de code suivant se connecte à votre base de données Azure Database pour PostgreSQL - Serveur flexible et utilise [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) avec l'instruction SQL **SELECT** pour lire les données. Cette fonction accepte une requête et renvoie un jeu de résultats pour itérer en utilisant [cursor.fetchall()](http://initd.org/psycopg/docs/cursor.html#cursor.fetchall). 

```Python
import psycopg2
# Update connection string information
host = "<server-name>"
dbname = "<database-name>"
user = "<admin-username>"
password = "<admin-password>"
sslmode = "require"
# Construct connection string
conn_string = "host={0} user={1} dbname={2} password={3} sslmode={4}".format(host, user, dbname, password, sslmode)
conn = psycopg2.connect(conn_string) 
print("Connection established")
cursor = conn.cursor()
# Fetch all rows from table
cursor.execute("SELECT * FROM inventory;")
rows = cursor.fetchall()
# Print all rows
for row in rows:
    print("Data row = (%s, %s, %s)" %(str(row[0]), str(row[1]), str(row[2])))
# Cleanup
conn.commit()
cursor.close()
conn.close()
```

## <a name="update-data"></a>Mettre à jour des données
L'exemple de code suivant se connecte à votre base de données Azure Database pour PostgreSQL - Serveur flexible et utilise [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) avec l'instruction SQL **UPDATE** pour mettre à jour les données. 

```Python
import psycopg2
# Update connection string information
host = "<server-name>"
dbname = "<database-name>"
user = "<admin-username>"
password = "<admin-password>"
sslmode = "require"
# Construct connection string
conn_string = "host={0} user={1} dbname={2} password={3} sslmode={4}".format(host, user, dbname, password, sslmode)
conn = psycopg2.connect(conn_string) 
print("Connection established")
cursor = conn.cursor()
# Update a data row in the table
cursor.execute("UPDATE inventory SET quantity = %s WHERE name = %s;", (200, "banana"))
print("Updated 1 row of data")
# Cleanup
conn.commit()
cursor.close()
conn.close()
```

## <a name="delete-data"></a>Suppression de données
L'exemple de code suivant se connecte à votre base de données Azure Database pour PostgreSQL - Serveur flexible et utilise [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) avec l'instruction SQL **DELETE** pour supprimer un élément d'inventaire que vous aviez précédemment inséré. 

```Python
import psycopg2
# Update connection string information
host = "<server-name>"
dbname = "<database-name>"
user = "<admin-username>"
password = "<admin-password>"
sslmode = "require"
# Construct connection string
conn_string = "host={0} user={1} dbname={2} password={3} sslmode={4}".format(host, user, dbname, password, sslmode)
conn = psycopg2.connect(conn_string) 
print("Connection established")
cursor = conn.cursor()
# Delete data row from table
cursor.execute("DELETE FROM inventory WHERE name = %s;", ("orange",))
print("Deleted 1 row of data")
# Cleanup
conn.commit()
cursor.close()
conn.close()
```

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Migrer votre base de données à l'aide de l'image mémoire et de la restauration](../howto-migrate-using-dump-and-restore.md)