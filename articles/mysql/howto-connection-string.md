---
title: Chaînes de connexion - Azure Database pour MySQL
description: Ce document répertorie les chaînes de connexion actuellement prises en charge pour les applications qui se connectent à la base de données Azure pour MySQL, notamment ADO.NET (C#), JDBC, Node.js, ODBC, PHP, Python et Ruby.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 3/18/2020
ms.custom: devx-track-python, devx-track-js
ms.openlocfilehash: 40af2660f0052a876ef9310bc2426295ba67558b
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541485"
---
# <a name="how-to-connect-applications-to-azure-database-for-mysql"></a>Connexion d’applications à Azure Database pour MySQL
Cette rubrique répertorie les types de chaînes de connexion pris en charge par Azure Database pour MySQL, ainsi que des modèles et des exemples. Vous pouvez avoir des paramètres et des réglages différents dans votre chaîne de connexion.

- Pour obtenir le certificat, consultez [Configuration de la connectivité SSL dans votre application pour se connecter en toute sécurité à la base de données Azure pour MySQL](./howto-configure-ssl.md).
- {votre_hôte} = \<servername>.mysql.database.azure.com
- {votre_utilisateur}@{nomserveur} = format ID utilisateur pour une authentification correcte.  Si vous utilisez uniquement l’ID utilisateur, l’authentification échoue.

## <a name="adonet"></a>ADO.NET
```ado.net
Server={your_host};Port={your_port};Database={your_database};Uid={username@servername};Pwd={your_password};[SslMode=Required;]
```

Dans cet exemple, le nom du serveur est `mydemoserver`, le nom de la base de données est `wpdb`, le nom d’utilisateur est `WPAdmin` et le mot de passe est `mypassword!2`. Donc, la chaîne de connexion devrait être :

```ado.net
Server= "mydemoserver.mysql.database.azure.com"; Port=3306; Database= "wpdb"; Uid= "WPAdmin@mydemoserver"; Pwd="mypassword!2"; SslMode=Required;
```

## <a name="jdbc"></a>JDBC
```jdbc
String url ="jdbc:mysql://%s:%s/%s[?verifyServerCertificate=true&useSSL=true&requireSSL=true]",{your_host},{your_port},{your_database}"; myDbConn = DriverManager.getConnection(url, {username@servername}, {your_password}";
```

## <a name="nodejs"></a>Node.js
```node.js
var conn = mysql.createConnection({host: {your_host}, user: {username@servername}, password: {your_password}, database: {your_database}, Port: {your_port}[, ssl:{ca:fs.readFileSync({ca-cert filename})}}]);
```

## <a name="odbc"></a>ODBC
```odbc
DRIVER={MySQL ODBC 5.3 UNICODE Driver};Server={your_host};Port={your_port};Database={your_database};Uid={username@servername};Pwd={your_password}; [sslca={ca-cert filename}; sslverify=1; Option=3;]
```

## <a name="php"></a>PHP
```php
$con=mysqli_init(); [mysqli_ssl_set($con, NULL, NULL, {ca-cert filename}, NULL, NULL);] mysqli_real_connect($con, {your_host}, {username@servername}, {your_password}, {your_database}, {your_port});
```

## <a name="python"></a>Python
```python
cnx = mysql.connector.connect(user={username@servername}, password={your_password}, host={your_host}, port={your_port}, database={your_database}[, ssl_ca={ca-cert filename}, ssl_verify_cert=true])
```

## <a name="ruby"></a>Ruby
```ruby
client = Mysql2::Client.new(username: {username@servername}, password: {your_password}, database: {your_database}, host: {your_host}, port: {your_port}[, sslca:{ca-cert filename}, sslverify:false, sslcipher:'AES256-SHA'])
```

## <a name="get-the-connection-string-details-from-the-azure-portal"></a>Obtenir les détails de la chaîne de connexion dans le portail Azure
Dans le [portail Azure](https://portal.azure.com), accédez à votre serveur Azure Database pour MySQL, puis cliquez sur **Chaînes de connexion** pour obtenir la liste des chaînes de votre instance : :::image type="content" source="./media/howto-connection-strings/connection-strings-on-portal.png" alt-text="Le panneau Chaînes de connexion dans le portail Azure":::

La chaîne fournit des détails tels que le pilote, le serveur et d’autres paramètres de connexion de base de données. Modifiez ces exemples pour utiliser vos propres paramètres, tels que le nom de la base de données, le mot de passe, etc. Vous pouvez ensuite utiliser cette chaîne pour vous connecter au serveur à partir de votre code et de vos applications.

## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur les bibliothèques de connexions, consultez [Concepts - Bibliothèques de connexions](./concepts-connection-libraries.md).
