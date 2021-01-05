---
title: Compatibilité des pilotes et outils - Azure Database pour MySQL
description: Cet article décrit les pilotes MySQL et les outils de gestion compatibles avec Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: f2ea7af0ec9c740c07cd3a5e4ebd88a9ad1a031d
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94542641"
---
# <a name="mysql-drivers-and-management-tools-compatible-with-azure-database-for-mysql"></a>Pilotes MySQL et outils de gestion compatibles avec Azure Database pour MySQL
Cet article décrit les pilotes et outils de gestion compatibles avec Azure Database pour MySQL.

## <a name="mysql-drivers"></a>Pilotes MySQL
En termes de base de données MySQL, Azure Database pour MySQL utilise la version community edition la plus populaire au monde. Par conséquent, elle est compatible avec un large éventail de langages de programmation et de pilotes. L’objectif est de prendre en charge les trois versions les plus récentes de pilotes MySQL, et de poursuivre les efforts avec les auteurs de la Communauté open source, afin d’améliorer en permanence les fonctionnalités et la facilité d’utilisation des pilotes MySQL. Une liste de pilotes testés et détectés compatibles avec Azure Database pour MySQL 5.6 et 5.7 est fournie dans le tableau suivant :

| **Langage de programmation** | **Driver** | **Liens** | **Versions compatibles** | **Versions incompatibles** | **Remarques** |
| :----------------------- | :--------- | :-------- | :---------------------- | :------------------------ | :-------- |
| PHP | mysqli, pdo_mysql, mysqlnd | https://secure.php.net/downloads.php | 5.5, 5.6, 7.x | 5.3 | Pour la connexion PHP 7.0 à SSL MySQLi, ajoutez MYSQLI_CLIENT_SSL_DONT_VERIFY_SERVER_CERT dans la chaîne de connexion. <br> ```mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306, NULL, MYSQLI_CLIENT_SSL_DONT_VERIFY_SERVER_CERT);```<br> Ensemble PDO : option ```PDO::MYSQL_ATTR_SSL_VERIFY_SERVER_CERT``` définie sur false.|
| .NET | Connecteur MySQL Async pour .NET | https://github.com/mysql-net/MySqlConnector <br> [Package d’installation à partir de Nuget](https://www.nuget.org/packages/MySqlConnector/) | 0.27 et versions ultérieures | 0.26.5 et versions antérieures | |
| .NET | Connecteur MySQL/NET | https://github.com/mysql/mysql-connector-net | 6.6.3, 7.0, 8.0 |  | Un bogue d’encodage peut provoquer l’échec des connexions sur certains systèmes Windows non -UTF8. |
| Node.js | mysqljs | https://github.com/mysqljs/mysql/ <br> Package d’installation à partir de NPM :<br> Exécuter `npm install mysql` à partir de NPM | 2.15 | 2.14.1 et versions antérieures | |
| Node.js | node-mysql2 | https://github.com/sidorares/node-mysql2 | 1.3.4+ | | |
| Go | Pilote Go MySQL | https://github.com/go-sql-driver/mysql/releases | 1.3, 1.4 | 1.2 et versions antérieures | Utilisez `allowNativePasswords=true` dans la chaîne de connexion pour la version 1.3. La version 1.4 contient un correctif et `allowNativePasswords=true` n’est plus nécessaire. |
| Python | Connecteur MySQL/Python | https://pypi.python.org/pypi/mysql-connector-python | 1.2.3, 2.0, 2.1, 2.2, utiliser 8.0.16+ avec MySQL 8.0  | 1.2.2 et versions antérieures | |
| Python | PyMySQL | https://pypi.org/project/PyMySQL/ | 0.7.11, 0.8.0, 0.8.1, 0.9.3+ | 0.9.0 - 0.9.2 (régression dans web2py) | |
| Java | Connecteur MariaDB/J | https://downloads.mariadb.org/connector-java/ | 2.1, 2.0, 1.6 | 1.5.5 et versions antérieures | | 
| Java | Connecteur MySQL/J | https://github.com/mysql/mysql-connector-j | 5.1.21+, utiliser 8.0.17+ avec MySQL 8.0 | 5.1.20 et versions antérieures | |
| C | Connecteur MySQL/C (libmysqlclient) | https://dev.mysql.com/doc/refman/5.7/en/c-api-implementations.html | 6.0.2+ | | |
| C | Connecteur MySQL/ODBC (myodbc) | https://github.com/mysql/mysql-connector-odbc | 3.51.29+ | | |
| C++ | Connecteur MySQL/C++ | https://github.com/mysql/mysql-connector-cpp | 1.1.9+ | 1.1.3 et versions antérieures | | 
| C++ | MySQL++| https://github.com/tangentsoft/mysqlpp | 3.2.3+ | | |
| Ruby | mysql2 | https://github.com/brianmario/mysql2 | 0.4.10+ | | |
| R | RMySQL | https://github.com/rstats-db/RMySQL | 0.10.16+ | | |
| Swift | mysql-swift | https://github.com/novi/mysql-swift | 0.7.2+ | | |
| Swift | vapor/mysql | https://github.com/vapor/mysql-kit | 2.0.1+ | | |

## <a name="management-tools"></a>Outils de gestion
L’avantage de compatibilité s’étend également aux outils de gestion de base de données. Vos outils existants doivent continuer de fonctionner avec Azure Database pour MySQL, tant que la manipulation de base de données s’effectue dans les limites des autorisations utilisateur. Trois outils de gestion de base de données courants, qui ont été testés et détectés compatibles avec Azure Database pour MySQL 5.6 et 5.7, sont répertoriés dans le tableau suivant :

|                                     | **MySQL Workbench 6.x et supérieur** | **Navicat 12** | **PHPMyAdmin 4.x et supérieur** |
| :---------------------------------- | :----------------------------- | :------------- | :-------------------------|
| **Création, mise à jour, lecture, écriture, suppression** | X | X | X |
| **Connexion SSL** | X | X | X |
| **Saisie semi-automatique de la requête SQL** | X | X |  |
| **Importation et exportation de données** | X | X | X |
| **Exportation dans plusieurs formats** | X | X | X |
| **Sauvegarde et restauration** |  | X |  |
| **Affichage des paramètres du serveur** | X | X | X |
| **Affichage des connexions client** | X | X | X |

## <a name="next-steps"></a>Étapes suivantes

- [Résoudre les problèmes de connexion à la base de données Azure Database pour MySQL](howto-troubleshoot-common-connection-issues.md)