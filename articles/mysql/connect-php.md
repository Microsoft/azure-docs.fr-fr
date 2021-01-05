---
title: 'Démarrage rapide : Se connecter à l’aide de PHP – Azure Database pour MySQL'
description: Ce guide de démarrage rapide fournit plusieurs exemples de code PHP, que vous pouvez utiliser pour vous connecter et interroger des données de la base de données Azure pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 10/28/2020
ms.openlocfilehash: cb3b711c532ccf44bebf08d42b5284db458cf5b7
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492656"
---
# <a name="quickstart-use-php-to-connect-and-query-data-in-azure-database-for-mysql"></a>Démarrage rapide : Utilisation de PHP pour se connecter et interroger des données dans Azure Database pour MySQL
Ce guide de démarrage rapide vous explique comment vous connecter à une base de données Azure pour MySQL en utilisant une application [PHP](https://secure.php.net/manual/intro-whatis.php). Il détaille l’utilisation d’instructions SQL pour interroger la base de données, la mettre à jour, y insérer des données ou en supprimer.

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

- [Créer une base de données et un utilisateur non-administrateur](./howto-create-users.md?tabs=single-server)
- Installer la toute dernière version PHP pour votre système d’exploitation
    - [Exécuter sur macOS](https://secure.php.net/manual/install.macosx.php)
    - [PHP sur Linux](https://secure.php.net/manual/install.unix.php)
    - [PHP sur Windows](https://secure.php.net/manual/install.windows.php)

> [!NOTE]
> Nous utilisons la bibliothèque [MySQLi](https://www.php.net/manual/en/book.mysqli.php) pour gérer, connecter et interroger le serveur dans ce guide de démarrage rapide.

## <a name="get-connection-information"></a>Obtenir des informations de connexion
Vous pouvez récupérer les informations de connexion du serveur de base de données depuis le portail Azure en suivant ces étapes :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Accédez à la page Azure Database pour MySQL. Vous pouvez rechercher et sélectionner **Azure Database pour MySQL**.
:::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/find-azure-mysql-in-portal.png" alt-text="Trouver Azure Database pour MySQL":::

2. Sélectionnez votre serveur MySQL (par exemple **mydemoserver**).
3. Dans la page **Vue d’ensemble**, copiez le nom complet du serveur en regard de **Nom du serveur** et le nom d’utilisateur administrateur en regard de **Nom de connexion de l’administrateur du serveur**. Pour copier le nom du serveur ou de l’hôte, pointez dessus et sélectionnez l’icône **Copier**.

> [!IMPORTANT]
> - Si vous avez oublié votre mot de passe, vous pouvez [réinitialiser le mot de passe](./howto-create-manage-server-portal.md#update-admin-password).
> - Remplacez les valeurs des paramètres **host, username, password** et **db_name** par vos propres valeurs**.

## <a name="step-1-connect-to-the-server"></a>Étape 1 : Connexion au serveur
SSL est activé par défaut. Vous devrez peut-être télécharger le [certificat SSL DigiCertGlobalRootG2](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem) pour vous connecter depuis votre environnement local. Ce code appelle :
- [mysqli_init](https://secure.php.net/manual/mysqli.init.php) pour initialiser MySQLi.
- [mysqli_ssl_set](https://www.php.net/manual/en/mysqli.ssl-set.php) pour pointer sur le chemin du certificat SSL. Cette action est nécessaire pour votre environnement local, mais elle est inutile pour App Service Web App et les machines virtuelles Azure.
- [mysqli_real_connect](https://secure.php.net/manual/mysqli.real-connect.php) pour se connecter à MySQL.
- [mysqli_close](https://secure.php.net/manual/mysqli.close.php) pour fermer la connexion.


```php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin@mydemoserver';
$password = 'your_password';
$db_name = 'your_database';

//Initializes MySQLi
$conn = mysqli_init();

// If using  Azure Virtual machines or Azure Web App, 'mysqli-ssl_set()' is not required as the certificate is already installed on the machines.
mysqli_ssl_set($conn,NULL,NULL, "/var/www/html/DigiCertGlobalRootG2.crt.pem", NULL, NULL);

// Establish the connection
mysqli_real_connect($conn, 'mydemoserver.mysql.database.azure.com', 'myadmin@mydemoserver', 'yourpassword', 'quickstartdb', 3306, MYSQLI_CLIENT_SSL);

//If connection failed, show the error
if (mysqli_connect_errno($conn))
{
    die('Failed to connect to MySQL: '.mysqli_connect_error());
}
```
[Vous rencontrez des problèmes ? Faites-le nous savoir](https://aka.ms/mysql-doc-feedback)

## <a name="step-2-create-a-table"></a>Étape 2 : Créer une table
Utilisez le code suivant pour la connexion. Ce code appelle :
- [mysqli_query](https://secure.php.net/manual/mysqli.query.php) pour exécuter la requête.
```php
// Run the create table query
if (mysqli_query($conn, '
CREATE TABLE Products (
`Id` INT NOT NULL AUTO_INCREMENT ,
`ProductName` VARCHAR(200) NOT NULL ,
`Color` VARCHAR(50) NOT NULL ,
`Price` DOUBLE NOT NULL ,
PRIMARY KEY (`Id`)
);
')) {
printf("Table created\n");
}
```

## <a name="step-3-insert-data"></a>Étape 3 : Insertion des données
Utilisez le code suivant pour insérer des données à l’aide d’une instruction SQL **INSERT**. Ce code utilise les méthodes :
- [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) pour créer une instruction INSERT préparée.
- [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php) pour lier les paramètres de chaque valeur de colonne insérée.
- [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php).
- [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php) pour fermer l’instruction à l’aide de la méthode.


```php
//Create an Insert prepared statement and run it
$product_name = 'BrandNewProduct';
$product_color = 'Blue';
$product_price = 15.5;
if ($stmt = mysqli_prepare($conn, "INSERT INTO Products (ProductName, Color, Price) VALUES (?, ?, ?)"))
{
    mysqli_stmt_bind_param($stmt, 'ssd', $product_name, $product_color, $product_price);
    mysqli_stmt_execute($stmt);
    printf("Insert: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));
    mysqli_stmt_close($stmt);
}

```

## <a name="step-4-read-data"></a>Étape 4 : Lire les données
Utilisez le code suivant pour lire les données à l’aide d’une instruction SQL **SELECT**.  Le code utilise la méthode :
- [mysqli_query](https://secure.php.net/manual/mysqli.query.php) pour exécuter la requête **SELECT**.
- [mysqli_fetch_assoc](https://secure.php.net/manual/mysqli-result.fetch-assoc.php) pour extraire les lignes résultantes.

```php
//Run the Select query
printf("Reading data from table: \n");
$res = mysqli_query($conn, 'SELECT * FROM Products');
while ($row = mysqli_fetch_assoc($res))
 {
    var_dump($row);
 }

```


## <a name="step-5-delete-data"></a>Étape 5 : Suppression de données
Utilisez le code suivant pour supprimer des lignes au moyen d’une instruction SQL **DELETE**. Le code utilise les méthodes :
- [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) pour créer une instruction DELETE préparée.
- [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php) lie les paramètres.
- [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php) exécute l’instruction DELETE préparée.
- [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php) ferme l’instruction.

```php
//Run the Delete statement
$product_name = 'BrandNewProduct';
if ($stmt = mysqli_prepare($conn, "DELETE FROM Products WHERE ProductName = ?")) {
mysqli_stmt_bind_param($stmt, 's', $product_name);
mysqli_stmt_execute($stmt);
printf("Delete: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));
mysqli_stmt_close($stmt);
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