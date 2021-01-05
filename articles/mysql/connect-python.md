---
title: 'Démarrage rapide : Se connecter à l’aide de Python – Azure Database pour MySQL'
description: Ce guide de démarrage rapide fournit plusieurs exemples de code Python, que vous pouvez utiliser pour vous connecter et interroger des données d’Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom:
- mvc
- seo-python-october2019
- devx-track-python
ms.devlang: python
ms.topic: quickstart
ms.date: 10/28/2020
ms.openlocfilehash: 8aa0ea4b1e01cc7363f49d5897695c7c237b339b
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94535586"
---
# <a name="quickstart-use-python-to-connect-and-query-data-in-azure-database-for-mysql"></a>Démarrage rapide : Utiliser Python pour se connecter et interroger des données dans Azure Database pour MySQL

Dans ce guide de démarrage rapide, vous vous connectez à Azure Database pour MySQL en utilisant Python. Vous utilisez ensuite des instructions SQL pour interroger, insérer, mettre à jour et supprimer des données dans la base de données depuis des plateformes Mac, Ubuntu Linux et Windows. 

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

## <a name="install-python-and-the-mysql-connector"></a>Installer Python et le connecteur MySQL

Effectuez les étapes suivantes pour installer Python et le connecteur MySQL pour Python sur votre ordinateur : 

> [!NOTE]
> Ce guide de démarrage rapide utilise le [Guide de développement Python/Connecteur MySQL](https://dev.mysql.com/doc/connector-python/en/).

1. Téléchargez et installez [Python 3.7 ou ultérieur](https://www.python.org/downloads/) pour votre système d’exploitation. Veillez à ajouter Python à votre `PATH`, car le connecteur MySQL en a besoin.
   
2. Ouvrez une invite de commandes ou un interpréteur de commandes `bash`, puis vérifiez votre version de Python en exécutant `python -V` avec le commutateur V majuscule.
   
3. Le programme d’installation de package `pip` est inclus dans les dernières versions de Python. Exécutez `pip install -U pip` pour mettre à jour `pip` avec la dernière version. 
   
   Si `pip` n’est pas installé, vous pouvez le télécharger et l’installer avec `get-pip.py`. Pour plus d’informations, consultez [Installation](https://pip.pypa.io/en/stable/installing/). 
   
4. Utilisez `pip` pour installer le connecteur MySQL pour Python et ses dépendances :
   
   ```bash
   pip install mysql-connector-python
   ```
   
[Vous rencontrez des problèmes ? Faites-le nous savoir](https://aka.ms/mysql-doc-feedback)

## <a name="get-connection-information"></a>Obtenir des informations de connexion

Obtenez les informations dont vous avez besoin pour vous connecter à Azure Database pour MySQL dans le portail Azure. Vous devez disposer du nom du serveur, du nom de la base de données et des informations d’identification de connexion.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
   
1. Dans la barre de recherche du portail, recherchez et sélectionnez le serveur Azure Database pour MySQL que vous avez créé, par exemple **mydemoserver**.
   
   :::image type="content" source="./media/connect-python/1_server-overview-name-login.png" alt-text="Nom du serveur de base de données Azure pour MySQL":::
   
1. Dans la page **Vue d’ensemble** du serveur, notez le **nom du serveur** et le **nom de connexion de l’administrateur du serveur**. Si vous oubliez votre mot de passe, vous pouvez également le réinitialiser dans cette page.
   
   :::image type="content" source="./media/connect-python/azure-database-for-mysql-server-overview-name-login.png" alt-text="Nom du serveur Azure Database pour MySQL 2":::

## <a name="step-1-create-a-table-and-insert-data"></a>Étape 1 : Créer une table et insérer des données

Utilisez le code suivant pour vous connecter au serveur et à la base de données, créer une table et charger les données à l’aide d’une instruction SQL **INSERT**. Le code importe la bibliothèque mysql.connector et utilise ces éléments :
- La fonction [connect()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysql-connector-connect.html) permettant de se connecter à Azure Database pour MySQL au moyen des [arguments](https://dev.mysql.com/doc/connector-python/en/connector-python-connectargs.html) de la collection de configurations. 
- La méthode [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) exécute la requête SQL sur la base de données MySQL. 
- [curseur.close()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-close.html) lorsque vous avez fini d’utiliser un curseur.
- [conn.close()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlconnection-close.html) pour fermer la connexion.

> [!IMPORTANT]
> - SSL est activé par défaut. Vous devrez peut-être télécharger le certificat [SSL DigiCertGlobalRootG2](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem) pour vous connecter à partir de votre environnement local.
> - Remplacez les espaces réservés `<mydemoserver>`, `<myadmin>`, `<mypassword>` et `<mydatabase>` par les valeurs de votre serveur et de votre base de données MySQL.

```python
import mysql.connector
from mysql.connector import errorcode

# Obtain connection string information from the portal
config = {
  'host':'<mydemoserver>.mysql.database.azure.com',
  'user':'<myadmin>@<mydemoserver>',
  'password':'<mypassword>',
  'database':'<mydatabase>',
  'client_flags': [ClientFlag.SSL],
  'ssl_cert': '/var/wwww/html/DigiCertGlobalRootG2.crt.pem'
}

# Construct connection string
try:
   conn = mysql.connector.connect(**config)
   print("Connection established")
except mysql.connector.Error as err:
  if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
    print("Something is wrong with the user name or password")
  elif err.errno == errorcode.ER_BAD_DB_ERROR:
    print("Database does not exist")
  else:
    print(err)
else:
  cursor = conn.cursor()

  # Drop previous table of same name if one exists
  cursor.execute("DROP TABLE IF EXISTS inventory;")
  print("Finished dropping table (if existed).")

  # Create table
  cursor.execute("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);")
  print("Finished creating table.")

  # Insert some data into table
  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("banana", 150))
  print("Inserted",cursor.rowcount,"row(s) of data.")
  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("orange", 154))
  print("Inserted",cursor.rowcount,"row(s) of data.")
  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("apple", 100))
  print("Inserted",cursor.rowcount,"row(s) of data.")

  # Cleanup
  conn.commit()
  cursor.close()
  conn.close()
  print("Done.")
```

[Vous rencontrez des problèmes ? Faites-le nous savoir](https://aka.ms/mysql-doc-feedback)

## <a name="step-2-read-data"></a>Étape 2 : Lire les données

Utilisez le code suivant pour vous connecter et lire des données à l’aide d’une instruction SQL **SELECT**. Le code importe la bibliothèque mysql.connector et utilise la méthode [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) pour exécuter la requête SQL sur la base de données MySQL. 

Le code lit les lignes de données à l’aide de la méthode [fetchall()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-fetchall.html), conserve le jeu de résultats dans une ligne de collection et utilise un itérateur `for` pour exécuter les lignes en boucle.

```python
  # Read data
  cursor.execute("SELECT * FROM inventory;")
  rows = cursor.fetchall()
  print("Read",cursor.rowcount,"row(s) of data.")

  # Print all rows
  for row in rows:
    print("Data row = (%s, %s, %s)" %(str(row[0]), str(row[1]), str(row[2])))

```

## <a name="step-3-update-data"></a>Étape 3 : Mettre à jour des données

Utilisez le code suivant pour vous connecter et mettre à jour les données à l’aide d’une instruction SQL **UPDATE**. Le code importe la bibliothèque mysql.connector et utilise la méthode [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) pour exécuter la requête SQL sur la base de données MySQL. 

```python
  # Update a data row in the table
  cursor.execute("UPDATE inventory SET quantity = %s WHERE name = %s;", (200, "banana"))
  print("Updated",cursor.rowcount,"row(s) of data.")
```

## <a name="step-4-delete-data"></a>Étape 4 : Suppression de données

Utilisez le code suivant pour vous connecter et supprimer des données à l’aide d’une instruction SQL **DELETE**. Le code importe la bibliothèque mysql.connector et utilise la méthode [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) pour exécuter la requête SQL sur la base de données MySQL. 

```python

  # Delete a data row in the table
  cursor.execute("DELETE FROM inventory WHERE name=%(param1)s;", {'param1':"orange"})
  print("Deleted",cursor.rowcount,"row(s) of data.")
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
