---
title: "Démarrage rapide : Se connecter à l'aide de PHP - Azure Database pour MySQL - Serveur flexible"
description: Ce guide de démarrage rapide fournit plusieurs exemples de code PHP, que vous pouvez utiliser pour vous connecter et interroger des données à partir d'Azure Database pour MySQL - Serveur flexible.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 9/21/2020
ms.openlocfilehash: dc6b069e3c7686ec6964dab890e503aa193cf6fe
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545104"
---
# <a name="quickstart-use-php-to-connect-and-query-data-in-azure-database-for-mysql---flexible-server"></a>Démarrage rapide : Utiliser PHP pour se connecter et interroger des données dans Azure Database pour MySQL - Serveur flexible

> [!IMPORTANT]
> Azure Database pour MySQL - Serveur flexible est actuellement disponible en préversion publique.

Ce guide de démarrage rapide vous explique comment vous connecter à Azure Database pour MySQL - Serveur flexible en utilisant une application [PHP](https://secure.php.net/manual/intro-whatis.php). Il détaille l’utilisation d’instructions SQL pour interroger la base de données, la mettre à jour, y insérer des données ou en supprimer. Cet article part du principe que vous connaissez les bases du développement PHP et que vous ne savez pas utiliser Azure Database pour MySQL - Serveur flexible.

## <a name="prerequisites"></a>Prérequis
Ce guide de démarrage rapide s’appuie sur les ressources créées dans l’un de ces guides :

- [Créer un serveur flexible Azure Database pour MySQL à l'aide du portail Azure](./quickstart-create-server-portal.md)
- [Créer un serveur flexible Azure Database pour MySQL à l'aide d'Azure CLI](./quickstart-create-server-cli.md)

## <a name="preparing-your-client-workstation"></a>Préparation de votre station de travail cliente
1. Si vous avez créé votre serveur flexible avec l'option *Accès privé (intégration au réseau virtuel)* , vous devez vous connecter à votre serveur à partir d'une ressource qui se trouve au sein du même réseau virtuel que votre serveur. Vous pouvez créer une machine virtuelle et l'ajouter au réseau virtuel créé avec votre serveur flexible. Reportez-vous à [Créer et gérer un réseau virtuel Azure Database pour MySQL - Serveur flexible à l'aide d'Azure CLI](./how-to-manage-virtual-network-cli.md).

2. Si vous avez créé votre serveur flexible avec l'option *Accès public (adresses IP autorisées)* , vous pouvez ajouter votre adresse IP locale à la liste des règles de pare-feu sur votre serveur. Reportez-vous à [Créer et gérer des règles de pare-feu Azure Database pour MySQL - Serveur flexible à l'aide d'Azure CLI](./how-to-manage-firewall-cli.md).

### <a name="install-php"></a>Installer PHP

Installez PHP sur votre serveur, ou créez une [application web](../../app-service/overview.md) Azure incluant PHP.  Reportez-vous à [Créer et gérer des règles de pare-feu](./how-to-manage-firewall-portal.md) pour apprendre à créer des règles de pare-feu.

#### <a name="macos"></a>macOS

1. Téléchargez [PHP version 7.1.4](https://secure.php.net/downloads.php).
2. Installez PHP et consultez le [manuel PHP](https://secure.php.net/manual/install.macosx.php) pour poursuivre la configuration.

#### <a name="linux-ubuntu"></a>Linux (Ubuntu)

1. Téléchargez [la version non thread-safe de PHP 7.1.4 (x64)](https://secure.php.net/downloads.php).
2. Installez PHP et consultez le [manuel PHP](https://secure.php.net/manual/install.unix.php) pour poursuivre la configuration.

#### <a name="windows"></a>Windows

1. Téléchargez [la version non thread-safe de PHP 7.1.4 (x64)](https://windows.php.net/download#php-7.1).
2. Installez PHP et consultez le [manuel PHP](https://secure.php.net/manual/install.windows.php) pour poursuivre la configuration.

## <a name="get-connection-information"></a>Obtenir des informations de connexion

Obtenez les informations requises pour vous connecter à Azure Database pour MySQL - Serveur flexible. Vous devez disposer du nom du serveur complet et des informations d'identification.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans le menu de gauche du portail Azure, sélectionnez **Toutes les ressources** , puis recherchez le serveur que vous venez de créer, par exemple **mydemoserver** .
3. Sélectionnez le nom du serveur.
4. Dans le panneau **Vue d’ensemble** du serveur, notez le **nom du serveur** et le **nom de connexion de l’administrateur du serveur** . Si vous oubliez votre mot de passe, vous pouvez également le réinitialiser dans ce panneau.
 <!---:::image type="content" source="./media/connect-php/1_server-overview-name-login.png" alt-text="Azure Database for MySQL Flexible Server name":::--->

## <a name="connecting-to-flexible-server-using-tlsssl-in-php"></a>Connexion à un serveur flexible à l'aide du protocole TLS/SSL dans PHP

Pour établir une connexion chiffrée à votre serveur flexible par le biais du protocole TLS/SSL à partir de votre application, reportez-vous aux exemples de code suivants. Vous pouvez télécharger le certificat nécessaire pour communiquer via TLS/SSL à partir de [https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem)

```php using SSL
$conn = mysqli_init();
mysqli_ssl_set($conn,NULL,NULL, "/var/www/html/DigiCertGlobalRootCA.crt.pem", NULL, NULL);
mysqli_real_connect($conn, 'mydemoserver.mysql.database.azure.com', 'myadmin', 'yourpassword', 'quickstartdb', 3306, MYSQLI_CLIENT_SSL);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}
```

## <a name="connect-and-create-a-table"></a>Se connecter et créer une table

Utilisez le code suivant pour vous connecter et créer une table à l’aide de l’instruction SQL **CREATE TABLE** .

Ce code utilise la classe **d’extension MySQL améliorée** (mysqli) incluse dans PHP. Il appelle les méthodes [mysqli_init](https://secure.php.net/manual/mysqli.init.php) et [mysqli_real_connect](https://secure.php.net/manual/mysqli.real-connect.php) pour se connecter à MySQL. Ensuite, il appelle la méthode [mysqli_query](https://secure.php.net/manual/mysqli.query.php) pour exécuter la requête. Enfin, il appelle la méthode [mysqli_close](https://secure.php.net/manual/mysqli.close.php) pour fermer la connexion.

Remplacez les paramètres db_name, de l’hôte, du nom d’utilisateur et du mot de passe par vos propres valeurs.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

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

//Close the connection
mysqli_close($conn);
?>
```

## <a name="insert-data"></a>Insertion des données

Utilisez le code suivant pour vous connecter et insérer des données à l’aide d’une instruction SQL **INSERT** .

Ce code utilise la classe **d’extension MySQL améliorée** (mysqli) incluse dans PHP. Le code utilise la méthode [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) pour créer une instruction Insert préparée, puis lie les paramètres de chaque valeur de colonne insérée à l’aide de la méthode [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php). Il exécute l’instruction à l’aide de la méthode [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php), puis ferme l’instruction à l’aide de la méthode [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php).

Remplacez les paramètres db_name, de l’hôte, du nom d’utilisateur et du mot de passe par vos propres valeurs.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

//Create an Insert prepared statement and run it
$product_name = 'BrandNewProduct';
$product_color = 'Blue';
$product_price = 15.5;
if ($stmt = mysqli_prepare($conn, "INSERT INTO Products (ProductName, Color, Price) VALUES (?, ?, ?)")) {
mysqli_stmt_bind_param($stmt, 'ssd', $product_name, $product_color, $product_price);
mysqli_stmt_execute($stmt);
printf("Insert: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));
mysqli_stmt_close($stmt);
}

// Close the connection
mysqli_close($conn);
?>
```

## <a name="read-data"></a>Lire les données

Utilisez le code suivant pour vous connecter et lire des données à l’aide d’une instruction SQL **SELECT** .  Ce code utilise la classe **d’extension MySQL améliorée** (mysqli) incluse dans PHP. Le code utilise la méthode [mysqli_query](https://secure.php.net/manual/mysqli.query.php) pour exécuter la requête sql, et la méthode [mysqli_fetch_assoc](https://secure.php.net/manual/mysqli-result.fetch-assoc.php) pour extraire les lignes ainsi créées.

Remplacez les paramètres db_name, de l’hôte, du nom d’utilisateur et du mot de passe par vos propres valeurs.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

//Run the Select query
printf("Reading data from table: \n");
$res = mysqli_query($conn, 'SELECT * FROM Products');
while ($row = mysqli_fetch_assoc($res)) {
var_dump($row);
}

//Close the connection
mysqli_close($conn);
?>
```

## <a name="update-data"></a>Mettre à jour des données

Utilisez le code suivant pour vous connecter et mettre à jour les données à l’aide d’une instruction SQL **UPDATE** .

Ce code utilise la classe **d’extension MySQL améliorée** (mysqli) incluse dans PHP. Le code utilise la méthode [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) pour créer une instruction Update préparée, puis lie les paramètres de chaque valeur de colonne mise à jour à l’aide de la méthode [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php). Il exécute l’instruction à l’aide de la méthode [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php), puis ferme l’instruction à l’aide de la méthode [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php).

Remplacez les paramètres db_name, de l’hôte, du nom d’utilisateur et du mot de passe par vos propres valeurs.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

//Run the Update statement
$product_name = 'BrandNewProduct';
$new_product_price = 15.1;
if ($stmt = mysqli_prepare($conn, "UPDATE Products SET Price = ? WHERE ProductName = ?")) {
mysqli_stmt_bind_param($stmt, 'ds', $new_product_price, $product_name);
mysqli_stmt_execute($stmt);
printf("Update: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));

//Close the connection
mysqli_stmt_close($stmt);
}

mysqli_close($conn);
?>
```


## <a name="delete-data"></a>Suppression de données
Utilisez le code suivant pour vous connecter et lire les données à l’aide d’une instruction SQL **DELETE** .

Ce code utilise la classe **d’extension MySQL améliorée** (mysqli) incluse dans PHP. Le code utilise la méthode [mysqli_prepare](https://secure.php.net/manual/mysqli.prepare.php) pour créer une instruction Delete préparée, puis lie les paramètres de la clause Where à l’aide de la méthode [mysqli_stmt_bind_param](https://secure.php.net/manual/mysqli-stmt.bind-param.php). Il exécute l’instruction à l’aide de la méthode [mysqli_stmt_execute](https://secure.php.net/manual/mysqli-stmt.execute.php), puis ferme l’instruction à l’aide de la méthode [mysqli_stmt_close](https://secure.php.net/manual/mysqli-stmt.close.php).

Remplacez les paramètres db_name, de l’hôte, du nom d’utilisateur et du mot de passe par vos propres valeurs.

```php
<?php
$host = 'mydemoserver.mysql.database.azure.com';
$username = 'myadmin';
$password = 'your_password';
$db_name = 'your_database';

//Establishes the connection
$conn = mysqli_init();
mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}

//Run the Delete statement
$product_name = 'BrandNewProduct';
if ($stmt = mysqli_prepare($conn, "DELETE FROM Products WHERE ProductName = ?")) {
mysqli_stmt_bind_param($stmt, 's', $product_name);
mysqli_stmt_execute($stmt);
printf("Delete: Affected %d rows\n", mysqli_stmt_affected_rows($stmt));
mysqli_stmt_close($stmt);
}

//Close the connection
mysqli_close($conn);
?>
```
## <a name="next-steps"></a>Étapes suivantes
- [Connectivité chiffrée à l'aide du protocole TLS (Transport Layer Security) 1.2 dans Azure Database pour MySQL - Serveur flexible](./how-to-connect-tls-ssl.md)
- En savoir plus sur la [Mise en réseau dans Azure Database pour MySQL - Serveur flexible](./concepts-networking.md)
- [Créer et gérer des règles de pare-feu Azure Database pour MySQL - Serveur flexible à l'aide du portail Azure](./how-to-manage-firewall-portal.md)
- [Créer et gérer un réseau virtuel Azure Database pour MySQL - Serveur flexible à l'aide du portail Azure](./how-to-manage-virtual-network-portal.md)