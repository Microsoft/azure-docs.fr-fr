---
title: 'Démarrage rapide : Se connecter avec Python – Azure Database pour PostgreSQL – Serveur unique'
description: Ce démarrage rapide fournit des exemples de code Python que vous pouvez utiliser pour vous connecter et interroger des données à partir de Azure Database pour PostgreSQL (serveur unique).
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.custom: mvc, devcenter, devx-track-python
ms.devlang: python
ms.topic: quickstart
ms.date: 10/28/2020
ms.openlocfilehash: db94a82112f2670facd4d89178f11653c5316c36
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95998938"
---
# <a name="quickstart-use-python-to-connect-and-query-data-in-azure-database-for-postgresql---single-server"></a>Démarrage rapide : Utiliser Python afin de se connecter à Azure Database pour PostgreSQL et d’interroger les données – Serveur unique

Ce guide de démarrage rapide vous explique comment vous connecter à la base de données sur un serveur unique Azure Database pour PostgreSQL et comment exécuter des instructions SQL pour interroger des données à l’aide de Python sur macOS, Ubuntu Linux ou Windows.

> [!TIP]
> Si vous envisagez de créer une application Django avec PostgreSQL, consultez le tutoriel [Déployer une application web Django avec PostgreSQL](../app-service/tutorial-python-postgresql-app.md).


## <a name="prerequisites"></a>Prérequis
Voici les prérequis pour ce guide de démarrage rapide :

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free).
- Créez un serveur unique Azure Database pour PostgreSQL à l’aide du [portail Azure](./quickstart-create-server-database-portal.md) <br/> ou d’[Azure CLI](./quickstart-create-server-database-azure-cli.md), si vous n’en avez pas.
- Selon que vous utilisez un accès public ou privé, effectuez **UNE** des actions ci-dessous pour activer la connectivité.

  |Action| Méthode de connexion|Guide pratique|
  |:--------- |:--------- |:--------- |
  | **Configurer les règles de pare-feu** | Public | [Portail](./howto-manage-firewall-using-portal.md) <br/> [INTERFACE DE LIGNE DE COMMANDE](./howto-manage-firewall-using-cli.md)|
  | **Configurer le point de terminaison de service** | Public | [Portail](./howto-manage-vnet-using-portal.md) <br/> [INTERFACE DE LIGNE DE COMMANDE](./howto-manage-vnet-using-cli.md)|
  | **Configurer une liaison privée** | Privées | [Portail](./howto-configure-privatelink-portal.md) <br/> [INTERFACE DE LIGNE DE COMMANDE](./howto-configure-privatelink-cli.md) |

- [Python](https://www.python.org/downloads/) 2.7.9+ ou 3.4+.

- Dernier programme d’installation de package [pip](https://pip.pypa.io/en/stable/installing/).
- Installez [psycopg2](https://pypi.python.org/pypi/psycopg2/) en exécutant la commande `pip install psycopg2` dans un terminal ou une fenêtre d’invite de commandes. Pour plus d’informations, consultez [Comment installer `psycopg2`](http://initd.org/psycopg/docs/install.html).

## <a name="get-database-connection-information"></a>Obtenir des informations de connexion à la base de données
La connexion d’une base de données Azure Database pour PostgreSQL nécessite un nom de serveur complet et des informations d’identification de connexion. Vous pouvez vous procurer ces informations à partir du portail Azure.

1. Dans le [portail Azure](https://portal.azure.com/), cherchez et sélectionnez le nom de votre serveur Azure Database pour PostgreSQL.
1. Sur la page **Vue d’ensemble** du serveur, copiez le **nom du serveur** complet et le **nom d’utilisateur administrateur**. Le **nom de serveur complet** a toujours la forme *\<my-server-name>.postgres.database.azure.com*, et le **nom d’utilisateur administrateur** a toujours la forme *\<my-admin-username>@\<my-server-name>* .

   Vous avez aussi besoin de votre mot de passe d’administrateur. Si vous l’avez oublié, vous pouvez le réinitialiser à partir de cette page.

   :::image type="content" source="./media/connect-python/1-connection-string.png" alt-text="Nom du serveur Azure Database pour PostgreSQL":::

> [!IMPORTANT]
>  Remplacez les valeurs suivantes :
>   - `<server-name>` et `<admin-username>` par les valeurs copiées à partir du portail Azure.
>   - `<admin-password>` avec votre mot de passe du serveur.
>   - `<database-name>` Une base de données par défaut nommée *postgres* a été automatiquement créée lorsque vous avez créé votre serveur. Vous pouvez renommer cette base de données ou [créer une autre base de données](https://www.postgresql.org/docs/9.0/sql-createdatabase.html) en utilisant des commandes SQL.

## <a name="step-1-connect-and-insert-data"></a>Étape 1 : Vous connecter et insérer des données
L’exemple de code suivant se connecte à votre base de données Azure Database pour PostgreSQL à l’aide de
-  la fonction [psycopg2.connect](http://initd.org/psycopg/docs/connection.html), puis il charge les données en utilisant une instruction SQL **INSERT**.
- La fonction [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) exécute la requête SQL sur la base de données.

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

:::image type="content" source="media/connect-python/2-example-python-output.png" alt-text="Sortie de la ligne de commande":::


[Vous rencontrez des problèmes ? Faites-le nous savoir](https://aka.ms/postgres-doc-feedback)

## <a name="step-2-read-data"></a>Étape 2 : Lire les données
L’exemple de code suivant se connecte à votre base de données Azure Database pour PostgreSQL et utilise
- [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) avec l’instruction SQL **SELECT** pour lire les données.
- [cursor.fetchall()](http://initd.org/psycopg/docs/cursor.html#cursor.fetchall) accepte une requête et retourne un jeu de résultats pour itérer dessus

```Python

# Fetch all rows from table
cursor.execute("SELECT * FROM inventory;")
rows = cursor.fetchall()

# Print all rows
for row in rows:
    print("Data row = (%s, %s, %s)" %(str(row[0]), str(row[1]), str(row[2])))


```
[Vous rencontrez des problèmes ? Faites-le nous savoir](https://aka.ms/postgres-doc-feedback)

## <a name="step-3-update-data"></a>Étape 3 : Mettre à jour des données
L’exemple de code suivant utilise [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) avec l’instruction SQL **UPDATE** pour mettre à jour les données.

```Python

# Update a data row in the table
cursor.execute("UPDATE inventory SET quantity = %s WHERE name = %s;", (200, "banana"))
print("Updated 1 row of data")

```
[Vous rencontrez des problèmes ? Faites-le nous savoir](https://aka.ms/postgres-doc-feedback)

## <a name="step-5-delete-data"></a>Étape 5 : Suppression de données
L’exemple de code suivant exécute [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) avec l’instruction SQL **DELETE** pour supprimer un élément d’inventaire que vous aviez précédemment inséré.

```Python

# Delete data row from table
cursor.execute("DELETE FROM inventory WHERE name = %s;", ("orange",))
print("Deleted 1 row of data")

```

[Vous rencontrez des problèmes ? Faites-le nous savoir](https://aka.ms/postgres-doc-feedback)

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
> [Gérer un serveur Azure Database pour MySQL à l’aide d’Azure CLI](./how-to-manage-server-cli.md)<br/>

[Vous ne trouvez pas ce que vous cherchez ? Faites-le nous savoir.](https://aka.ms/postgres-doc-feedback)
