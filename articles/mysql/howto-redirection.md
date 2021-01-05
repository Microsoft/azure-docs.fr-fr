---
title: Se connecter à l’aide de la redirection – Azure Database pour MySQL
description: Cet article décrit comment vous pouvez configurer votre application pour qu’elle se connecte à Azure Database pour MySQL avec la redirection.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 6/8/2020
ms.openlocfilehash: bea618b34809544308caba46a0ffbdecee91d69f
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541808"
---
# <a name="connect-to-azure-database-for-mysql-with-redirection"></a>Se connecter à Azure Database pour MySQL avec la redirection

Cette rubrique explique comment connecter une application à votre serveur Azure Database pour MySQL avec le mode de redirection. La redirection vise à réduire la latence réseau entre les applications clientes et les serveurs MySQL en permettant aux applications de se connecter directement aux nœuds du serveur back-end.

## <a name="before-you-begin"></a>Avant de commencer
Connectez-vous au [portail Azure](https://portal.azure.com). Créez un serveur Azure Database pour MySQL avec un moteur version 5.6, 5.7 ou 8.0. 

Pour plus d’informations, consultez la rubrique relative à la création d’un serveur Azure Database pour MySQL à l’aide du [portail Azure](quickstart-create-mysql-server-database-using-azure-portal.md) ou d’[Azure CLI](quickstart-create-mysql-server-database-using-azure-cli.md).

## <a name="enable-redirection"></a>Activer la redirection

Sur votre serveur Azure Database pour MySQL, configurez le paramètre `redirect_enabled` sur `ON` pour autoriser les connexions avec le mode de redirection. Pour mettre à jour ce paramètre de serveur, utilisez le [portail Azure](howto-server-parameters.md) ou [Azure CLI](howto-configure-server-parameters-using-cli.md).

## <a name="php"></a>PHP

La prise en charge de la redirection dans les applications PHP est disponible dans l’extension [mysqlnd_azure](https://github.com/microsoft/mysqlnd_azure), développée par Microsoft. 

L’extension mysqlnd_azure peut être ajoutée aux applications PHP via PECL, et il est vivement recommandé d’installer et de configurer l’extension par le biais du [package PECL](https://pecl.php.net/package/mysqlnd_azure) officiel.

> [!IMPORTANT]
> La prise en charge de la redirection dans l’extension [mysqlnd_azure](https://github.com/microsoft/mysqlnd_azure) PHP est actuellement disponible en préversion.

### <a name="redirection-logic"></a>Logique de redirection

>[!IMPORTANT]
> À partir de la version 1.1.0, la logique ou le comportement de redirection a été mis à jour. **Il est donc recommandé d’utiliser la version 1.1.0+.**

Le comportement de redirection est déterminé par la valeur de `mysqlnd_azure.enableRedirect`. Le tableau ci-dessous décrit le comportement de la redirection en fonction de la valeur de ce paramètre à partir de la **version 1.1.0+** .

Si vous utilisez une version antérieure de l’extension mysqlnd_azure (version 1.0.0-1.0.3), le comportement de la redirection est déterminé par la valeur de `mysqlnd_azure.enabled`. Les valeurs valides sont `off` (agit de la même façon que le comportement décrit dans le tableau ci-dessous) et `on` (agit comme `preferred` dans le tableau ci-dessous).  

|**Valeur mysqlnd_azure.enableRedirect**| **Comportement**|
|----------------------------------------|-------------|
|`off` ou `0`|La redirection ne sera pas utilisée. |
|`on` ou `1`|- Si la connexion n’utilise pas le protocole SSL côté pilote, aucune connexion n’est établie. L’erreur suivante est retournée : *« mysqlnd_azure.enableRedirect est activé, mais l’option SSL n’est pas définie dans la chaîne de connexion. La redirection n’est possible qu’avec SSL. »*<br>- Si le protocole SSL est utilisé côté pilote, mais que la redirection n’est pas prise en charge sur le serveur, la première connexion est abandonnée et l’erreur suivante est retournée : *« La connexion a été abandonnée, car la redirection n’est pas activée sur le serveur MySQL ou le package réseau ne respecte pas le protocole de redirection. »*<br>- Si le serveur MySQL prend en charge la redirection, mais que la connexion redirigée a échoué pour une raison quelconque, la première connexion au proxy est également abandonnée. Elle retourne l’erreur de la connexion redirigée.|
|`preferred` ou `2`<br> (valeur par défaut)|- mysqlnd_azure utilise la redirection, si possible.<br>- Si la connexion n’utilise pas le protocole SSL côte pilote, si le serveur ne prend pas en charge la redirection, ou si la connexion redirigée échoue pour une raison récupérable alors que la connexion par proxy est encore valide, elle revient à la première connexion par proxy.|

Les sections suivantes du document décrivent l’installation de l’extension `mysqlnd_azure` à l’aide de PECL et la définition de la valeur de ce paramètre.

### <a name="ubuntu-linux"></a>Ubuntu Linux

#### <a name="prerequisites"></a>Prérequis 
- Versions PHP 7.2.15+ et 7.3.2+
- PHP PEAR 
- php-mysql
- Serveur Azure Database pour MySQL

1. Installez [mysqlnd_azure](https://github.com/microsoft/mysqlnd_azure) avec [PECL](https://pecl.php.net/package/mysqlnd_azure). Il est recommandé d’utiliser la version 1.1.0+.

    ```bash
    sudo pecl install mysqlnd_azure
    ```

2. Localisez le répertoire d’extension (`extension_dir`) en exécutant ce qui suit :

    ```bash
    php -i | grep "extension_dir"
    ```

3. Remplacez les répertoires par le dossier retourné et vérifiez que `mysqlnd_azure.so` se trouve dans ce dossier. 

4. Localisez le dossier des fichiers .ini en exécutant ce qui suit : 

    ```bash
    php -i | grep "dir for additional .ini files"
    ```

5. Remplacez les répertoires par ce dossier retourné. 

6. Créez un nouveau fichier .ini pour `mysqlnd_azure`. Assurez-vous que l’ordre alphabétique du nom se trouve après celui de mysqnld, car les modules sont chargés en fonction de l’ordre des noms des fichiers ini. Par exemple, si `mysqlnd`.ini est nommé `10-mysqlnd.ini`, nommez le fichier ini mysqlnd `20-mysqlnd-azure.ini`.

7. Dans le nouveau fichier .ini, ajoutez les lignes suivantes pour activer la redirection.

    ```bash
    extension=mysqlnd_azure
    mysqlnd_azure.enableRedirect = on/off/preferred
    ```

### <a name="windows"></a>Windows

#### <a name="prerequisites"></a>Prérequis 
- Versions PHP 7.2.15+ et 7.3.2+
- php-mysql
- Serveur Azure Database pour MySQL

1. Déterminez si vous exécutez une version x64 ou x86 de PHP en exécutant la commande suivante :

    ```cmd
    php -i | findstr "Thread"
    ```

2. Téléchargez la version x64 ou x86 correspondante de la DLL [mysqlnd_azure](https://github.com/microsoft/mysqlnd_azure) à partir de [PECL](https://pecl.php.net/package/mysqlnd_azure) correspondant à votre version de PHP. Il est recommandé d’utiliser la version 1.1.0+.

3. Extrayez le fichier zip et recherchez la DLL nommée `php_mysqlnd_azure.dll`.

4. Localisez le répertoire d’extension (`extension_dir`) en exécutant la commande suivante :

    ```cmd
    php -i | find "extension_dir"
    ```

5. Copiez le fichier `php_mysqlnd_azure.dll` dans le répertoire retourné à l’étape 4. 

6. Localisez le dossier PHP contenant le fichier `php.ini` à l’aide de la commande suivante :

    ```cmd
    php -i | find "Loaded Configuration File"
    ```

7. Modifiez le fichier `php.ini` et ajoutez les lignes supplémentaires suivantes pour activer la redirection. 

    Sous la section Extensions dynamiques : 
    ```cmd
    extension=mysqlnd_azure
    ```
    
    Sous la section Paramètres du module :     
    ```cmd 
    [mysqlnd_azure]
    mysqlnd_azure.enableRedirect = on/off/preferred
    ```

### <a name="confirm-redirection"></a>Confirmer la redirection

Vous pouvez également confirmer que la redirection est configurée avec l’exemple de code PHP ci-dessous. Créez un fichier PHP nommé `mysqlConnect.php` et collez-y le code ci-dessous. Mettez à jour le nom du serveur, le nom d’utilisateur et le mot de passe en les remplaçant par vos propres valeurs. 
 
 ```php
<?php
$host = '<yourservername>.mysql.database.azure.com';
$username = '<yourusername>@<yourservername>';
$password = '<yourpassword>';
$db_name = 'testdb';
  echo "mysqlnd_azure.enableRedirect: ", ini_get("mysqlnd_azure.enableRedirect"), "\n";
  $db = mysqli_init();
  //The connection must be configured with SSL for redirection test
  $link = mysqli_real_connect ($db, $host, $username, $password, $db_name, 3306, NULL, MYSQLI_CLIENT_SSL);
  if (!$link) {
     die ('Connect error (' . mysqli_connect_errno() . '): ' . mysqli_connect_error() . "\n");
  }
  else {
    echo $db->host_info, "\n"; //if redirection succeeds, the host_info will differ from the hostname you used used to connect
    $res = $db->query('SHOW TABLES;'); //test query with the connection
    print_r ($res);
    $db->close();
  }
?>
 ```

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les chaînes de connexion, consultez [Chaînes de connexions](howto-connection-string.md).
