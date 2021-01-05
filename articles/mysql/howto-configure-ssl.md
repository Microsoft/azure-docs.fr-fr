---
title: Configurer SSL - Azure Database pour MySQL
description: Instructions pour configurer correctement la base de données Azure pour MySQL et les applications associées afin d’utiliser correctement les connexions SSL
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 07/08/2020
ms.custom: devx-track-python, devx-track-csharp
ms.openlocfilehash: 1c3e5a44e01f3fa43b82644103066f5a03684ad2
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94591578"
---
# <a name="configure-ssl-connectivity-in-your-application-to-securely-connect-to-azure-database-for-mysql"></a>Configuration de la connectivité SSL dans votre application pour se connecter en toute sécurité à la base de données Azure pour MySQL

La base de données Azure pour MySQL prend en charge la connexion de votre serveur Azure Database pour MySQL aux applications clientes à l’aide de Secure Sockets Layer (SSL). L’application de connexions SSL entre votre serveur de base de données et vos applications clientes vous protège contre les « attaques de l’intercepteur » en chiffrant le flux de données entre le serveur et votre application.

## <a name="step-1-obtain-ssl-certificate"></a>Étape 1 : Obtenir un certificat SSL

Téléchargez le certificat nécessaire pour communiquer via le protocole SSL avec votre serveur Azure Database pour MySQL à partir de [https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) et enregistrez le fichier de certificat sur votre disque local (ce didacticiel utilise c:\ssl à titre d’exemple).
**Pour Microsoft Internet Explorer et Microsoft Edge :** Une fois le téléchargement terminé, renommez le certificat en BaltimoreCyberTrustRoot.crt.pem.

>[!NOTE]
> Sur la base des commentaires des clients, nous avons étendu la dépréciation du certificat racine pour notre autorité de certification racine Baltimore existante jusqu’au 15 février 2021 (15/2/2021).

> [!IMPORTANT] 
> Le certificat racine SSL est configuré pour expirer à partir du 15 février 2021 (15/2/2021). Mettez à jour votre application pour utiliser le [nouveau certificat](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem). Pour plus d’informations, consultez [Mises à jour de certificat planifiées](concepts-certificate-rotation.md)

Consultez les liens suivants afin d’obtenir des certificats pour des serveurs de clouds souverains : [Azure Government](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem), [Azure Chine](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem) et [Azure Allemagne](https://www.d-trust.net/cgi-bin/D-TRUST_Root_Class_3_CA_2_2009.crt).

## <a name="step-2-bind-ssl"></a>Étape 2 : Créer une liaison SSL

Pour obtenir des chaînes de connexion de langage de programmation spécifiques, reportez-vous à l’[exemple de code](howto-configure-ssl.md#sample-code) ci-dessous.

### <a name="connecting-to-server-using-mysql-workbench-over-ssl"></a>Connexion au serveur à l’aide de MySQL Workbench via le protocole SSL

Configurez MySQL Workbench pour vous connecter en toute sécurité via le protocole SSL.

1. Dans la boîte de dialogue Configurer une nouvelle connexion, accédez à l’onglet **SSL**.

1. Mettez à jour le champ **Utiliser SSL** pour « Exiger ».

1. Dans le champ **SSL CA File:** , entrez l’emplacement du fichier **BaltimoreCyberTrustRoot.crt.pem**.

   :::image type="content" source="./media/howto-configure-ssl/mysql-workbench-ssl.png" alt-text="Enregistrer la configuration SSL":::

Pour les connexions existantes, vous pouvez lier SSL en cliquant avec le bouton droit sur l’icône de connexion et en choisissant Modifier. Accédez ensuite à l’onglet **SSL**, puis liez le fichier de certificat.

### <a name="connecting-to-server-using-the-mysql-cli-over-ssl"></a>Connexion au serveur à l’aide de l’interface de ligne de commande MySQL via le protocole SSL

Vous pouvez également lier le certificat SSL dans l’interface de ligne de commande MySQL en exécutant les commandes suivantes.

```bash
mysql.exe -h mydemoserver.mysql.database.azure.com -u Username@mydemoserver -p --ssl-mode=REQUIRED --ssl-ca=c:\ssl\BaltimoreCyberTrustRoot.crt.pem
```

> [!NOTE]
> Quand vous utilisez l’interface de ligne de commande MySQL sur Windows, vous pouvez recevoir une erreur `SSL connection error: Certificate signature check failed`. Dans ce cas, remplacez les paramètres `--ssl-mode=REQUIRED --ssl-ca={filepath}` par `--ssl`.

## <a name="step-3--enforcing-ssl-connections-in-azure"></a>Étape 3 :  Application de connexions SSL dans Azure

### <a name="using-the-azure-portal"></a>Utilisation du portail Azure

À partir du portail Azure, accédez à votre serveur de base de données Azure pour MySQL, puis cliquez sur **Sécurité de la connexion**. Utilisez le bouton bascule pour activer ou désactiver le paramètre **Appliquer une connexion SSL**, puis cliquez sur **Enregistrer**. Microsoft recommande de toujours activer le paramètre **Appliquer une connexion SSL** pour renforcer la sécurité.

:::image type="content" source="./media/howto-configure-ssl/enable-ssl.png" alt-text="Application de connexions SSL dans Azure Database pour MySQL : capture d'écran du portail Azure":::

### <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure

Vous pouvez activer ou désactiver le paramètre **ssl-enforcement** en utilisant respectivement les valeurs Enabled ou Disabled dans Azure CLI.

```azurecli-interactive
az mysql server update --resource-group myresource --name mydemoserver --ssl-enforcement Enabled
```

## <a name="step-4-verify-the-ssl-connection"></a>Étape 4 : Vérifier la connexion SSL

Exécuter la commande mysql **status** pour vérifier que vous êtes connecté à votre serveur MySQL à l’aide de SSL :

```dos
mysql> status
```

Confirmez le chiffrement de la connexion en vérifiant la sortie, qui doit indiquer :  **SSL: Cipher in use is AES256-SHA** (SSL  : Le chiffrement utilisé est AES256-SHA)

## <a name="sample-code"></a>Exemple de code

Pour établir une connexion sécurisée vers une base de données Azure Database pour MySQL sur SSL à partir de votre application, consultez les exemples de code suivants :

Reportez-vous à la liste des [pilotes compatibles](concepts-compatibility.md) pris en charge par le service Azure Database pour MySQL.

### <a name="php"></a>PHP

```php
$conn = mysqli_init();
mysqli_ssl_set($conn,NULL,NULL, "/var/www/html/BaltimoreCyberTrustRoot.crt.pem", NULL, NULL);
mysqli_real_connect($conn, 'mydemoserver.mysql.database.azure.com', 'myadmin@mydemoserver', 'yourpassword', 'quickstartdb', 3306, MYSQLI_CLIENT_SSL);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}
```

### <a name="php-using-pdo"></a>PHP (avec PDO)

```phppdo
$options = array(
    PDO::MYSQL_ATTR_SSL_CA => '/var/www/html/BaltimoreCyberTrustRoot.crt.pem'
);
$db = new PDO('mysql:host=mydemoserver.mysql.database.azure.com;port=3306;dbname=databasename', 'username@mydemoserver', 'yourpassword', $options);
```

### <a name="python-mysqlconnector-python"></a>Python (MySQLConnector Python)

```python
try:
    conn = mysql.connector.connect(user='myadmin@mydemoserver',
                                   password='yourpassword',
                                   database='quickstartdb',
                                   host='mydemoserver.mysql.database.azure.com',
                                   ssl_ca='/var/www/html/BaltimoreCyberTrustRoot.crt.pem')
except mysql.connector.Error as err:
    print(err)
```

### <a name="python-pymysql"></a>Python (PyMySQL)

```python
conn = pymysql.connect(user='myadmin@mydemoserver',
                       password='yourpassword',
                       database='quickstartdb',
                       host='mydemoserver.mysql.database.azure.com',
                       ssl={'ca': '/var/www/html/BaltimoreCyberTrustRoot.crt.pem'})
```

### <a name="django-pymysql"></a>Django (PyMySQL)

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'quickstartdb',
        'USER': 'myadmin@mydemoserver',
        'PASSWORD': 'yourpassword',
        'HOST': 'mydemoserver.mysql.database.azure.com',
        'PORT': '3306',
        'OPTIONS': {
            'ssl': {'ca': '/var/www/html/BaltimoreCyberTrustRoot.crt.pem'}
        }
    }
}
```

### <a name="ruby"></a>Ruby

```ruby
client = Mysql2::Client.new(
        :host     => 'mydemoserver.mysql.database.azure.com',
        :username => 'myadmin@mydemoserver',
        :password => 'yourpassword',
        :database => 'quickstartdb',
        :sslca => '/var/www/html/BaltimoreCyberTrustRoot.crt.pem'
    )
```

### <a name="golang"></a>Golang

```go
rootCertPool := x509.NewCertPool()
pem, _ := ioutil.ReadFile("/var/www/html/BaltimoreCyberTrustRoot.crt.pem")
if ok := rootCertPool.AppendCertsFromPEM(pem); !ok {
    log.Fatal("Failed to append PEM.")
}
mysql.RegisterTLSConfig("custom", &tls.Config{RootCAs: rootCertPool})
var connectionString string
connectionString = fmt.Sprintf("%s:%s@tcp(%s:3306)/%s?allowNativePasswords=true&tls=custom",'myadmin@mydemoserver' , 'yourpassword', 'mydemoserver.mysql.database.azure.com', 'quickstartdb')
db, _ := sql.Open("mysql", connectionString)
```

### <a name="java-mysql-connector-for-java"></a>Java (connecteur MySQL pour Java)

```java
# generate truststore and keystore in code
String importCert = " -import "+
    " -alias mysqlServerCACert "+
    " -file " + ssl_ca +
    " -keystore truststore "+
    " -trustcacerts " +
    " -storepass password -noprompt ";
String genKey = " -genkey -keyalg rsa " +
    " -alias mysqlClientCertificate -keystore keystore " +
    " -storepass password123 -keypass password " +
    " -dname CN=MS ";
sun.security.tools.keytool.Main.main(importCert.trim().split("\\s+"));
sun.security.tools.keytool.Main.main(genKey.trim().split("\\s+"));

# use the generated keystore and truststore
System.setProperty("javax.net.ssl.keyStore","path_to_keystore_file");
System.setProperty("javax.net.ssl.keyStorePassword","password");
System.setProperty("javax.net.ssl.trustStore","path_to_truststore_file");
System.setProperty("javax.net.ssl.trustStorePassword","password");

url = String.format("jdbc:mysql://%s/%s?serverTimezone=UTC&useSSL=true", 'mydemoserver.mysql.database.azure.com', 'quickstartdb');
properties.setProperty("user", 'myadmin@mydemoserver');
properties.setProperty("password", 'yourpassword');
conn = DriverManager.getConnection(url, properties);
```

### <a name="java-mariadb-connector-for-java"></a>Java (connecteur MariaDB pour Java)

```java
# generate truststore and keystore in code
String importCert = " -import "+
    " -alias mysqlServerCACert "+
    " -file " + ssl_ca +
    " -keystore truststore "+
    " -trustcacerts " +
    " -storepass password -noprompt ";
String genKey = " -genkey -keyalg rsa " +
    " -alias mysqlClientCertificate -keystore keystore " +
    " -storepass password123 -keypass password " +
    " -dname CN=MS ";
sun.security.tools.keytool.Main.main(importCert.trim().split("\\s+"));
sun.security.tools.keytool.Main.main(genKey.trim().split("\\s+"));

# use the generated keystore and truststore
System.setProperty("javax.net.ssl.keyStore","path_to_keystore_file");
System.setProperty("javax.net.ssl.keyStorePassword","password");
System.setProperty("javax.net.ssl.trustStore","path_to_truststore_file");
System.setProperty("javax.net.ssl.trustStorePassword","password");

url = String.format("jdbc:mariadb://%s/%s?useSSL=true&trustServerCertificate=true", 'mydemoserver.mysql.database.azure.com', 'quickstartdb');
properties.setProperty("user", 'myadmin@mydemoserver');
properties.setProperty("password", 'yourpassword');
conn = DriverManager.getConnection(url, properties);
```

### <a name="net-mysqlconnector"></a>.NET (MySqlConnector)

```csharp
var builder = new MySqlConnectionStringBuilder
{
    Server = "mydemoserver.mysql.database.azure.com",
    UserID = "myadmin@mydemoserver",
    Password = "yourpassword",
    Database = "quickstartdb",
    SslMode = MySqlSslMode.VerifyCA,
    SslCa = "BaltimoreCyberTrustRoot.crt.pem",
};
using (var connection = new MySqlConnection(builder.ConnectionString))
{
    connection.Open();
}
```

## <a name="next-steps"></a>Étapes suivantes

Passez en revue les différentes options de connectivité d’application de la rubrique [Bibliothèques de connexions pour Azure Database pour MySQL](concepts-connection-libraries.md)
