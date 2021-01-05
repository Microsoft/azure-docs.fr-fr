---
title: Compatibilité des pilotes et outils - Azure Database for MariaDB
description: Cet article décrit les pilotes MariaDB et les outils de gestion compatibles avec Azure Database for MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 49fd2c8da3ba0a44f4901bc6c7a2241e9e20df4d
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541689"
---
# <a name="mariadb-drivers-and-management-tools-compatible-with-azure-database-for-mariadb"></a>Pilotes MariaDB et outils de gestion compatibles avec Azure Database for MariaDB

Cet article décrit les pilotes et outils de gestion compatibles avec Azure Database for MariaDB.

## <a name="mariadb-drivers"></a>Pilotes MariaDB

Azure Database for MariaDB utilise la version Community du serveur MariaDB. Par conséquent, elle est compatible avec un large éventail de langages de programmation et de pilotes. L’API et le protocole MariaDB sont compatibles avec ceux utilisés par MySQL. Autrement dit, les connecteurs qui fonctionnent avec MySQL doivent également fonctionner avec MariaDB.

L’objectif est de prendre en charge les trois versions les plus récentes des pilotes MariaDB, et de travailler avec les auteurs de la communauté open source pour améliorer en permanence les fonctionnalités et la compatibilité des pilotes MariaDB. Le tableau ci-dessous liste les pilotes qui ont été testés et jugés compatibles avec Azure Database for MariaDB 10.2 :

**Driver** | **Liens** | **Versions compatibles** | **Versions incompatibles** | **Remarques**
---|---|---|---|---
PHP | https://secure.php.net/downloads.php | 5.5, 5.6, 7.x | 5.3 | Pour la connexion PHP 7.0 à SSL MySQLi, ajoutez MYSQLI_CLIENT_SSL_DONT_VERIFY_SERVER_CERT dans la chaîne de connexion. <br> ```mysqli_real_connect($conn, $host, $username, $password, $db_name, 3306, NULL, MYSQLI_CLIENT_SSL_DONT_VERIFY_SERVER_CERT);```<br> Ensemble PDO : option ```PDO::MYSQL_ATTR_SSL_VERIFY_SERVER_CERT``` définie sur false.
.NET | [MySqlConnector sur GitHub](https://github.com/mysql-net/MySqlConnector) <br> [Package d’installation à partir de Nuget](https://www.nuget.org/packages/MySqlConnector/) | 0.27 et versions ultérieures | 0.26.5 et versions antérieures |
Connecteur MySQL/NET | [Connecteur MySQL/NET](https://github.com/mysql/mysql-connector-net) | 8.0, 7.0, 6.10 |  | Un bogue d’encodage peut provoquer l’échec des connexions sur certains systèmes Windows non -UTF8.
Node.js |  [MySQLjs sur GitHub](https://github.com/mysqljs/mysql/) <br> Package d’installation à partir de NPM :<br> Exécuter `npm install mysql` à partir de NPM | 2.15 | 2.14.1 et versions antérieures
GO | https://github.com/go-sql-driver/mysql/releases | 1.3, 1.4 | 1.2 et versions antérieures | Utilisez `allowNativePasswords=true` dans la chaîne de connexion pour la version 1.3. La version 1.4 contient un correctif et `allowNativePasswords=true` n’est plus nécessaire.
Python | https://pypi.python.org/pypi/mysql-connector-python | 1.2.3, 2.0, 2.1, 2.2 | 1.2.2 et versions antérieures |
Java | https://downloads.mariadb.org/connector-java/ | 2.1, 2.0, 1.6 | 1.5.5 et versions antérieures |

## <a name="management-tools"></a>Outils de gestion

L’avantage de compatibilité s’étend également aux outils de gestion de base de données. Vos outils existants doivent continuer de fonctionner avec Azure Database for MariaDB, tant que la manipulation de base de données s’effectue dans les limites des autorisations utilisateur. Le tableau ci-dessous liste trois outils de gestion de base de données courants qui ont été testés et jugés compatibles avec Azure Database for MariaDB 10.2 :

| Action | **MySQL Workbench 6.x et supérieur** | **Navicat 12** | **PHPMyAdmin 4.x et supérieur**
---|---|---|---
Création, mise à jour, lecture, écriture, suppression | X | X | X
Connexion SSL | X | X | X
Saisie semi-automatique de la requête SQL | X | X |
Importation et exportation de données | X | X | X
Exportation dans plusieurs formats | X | X | X
Sauvegarde et restauration |  | X |
Affichage des paramètres du serveur | X | X | X
Affichage des connexions client | X | X | X

## <a name="next-steps"></a>Étapes suivantes

- [Résoudre les problèmes de connexion à la base de données Azure Database for MariaDB](howto-troubleshoot-common-connection-issues.md)