---
title: Créer des utilisateurs - Azure Database for MariaDB
description: Cet article explique comment vous pouvez créer des comptes d’utilisateurs pour interagir avec un serveur Azure Database for MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 10/1/2020
ms.openlocfilehash: 882c8365bda87e97bfbc3bee9bdd320b312b4114
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94542709"
---
# <a name="create-users-in-azure-database-for-mariadb"></a>Créer des utilisateurs dans Azure Database for MariaDB 
Cet article explique comment vous pouvez créer des utilisateurs dans Azure Database for MariaDB.

> [!NOTE]
> Communication sans biais
>
> La diversité et l’inclusion sont au cœur des valeurs de Microsoft. Cet article contient des références au mot _esclave_. Le [guide de style de Microsoft sur la communication sans stéréotype](https://github.com/MicrosoftDocs/microsoft-style-guide/blob/master/styleguide/bias-free-communication.md) le reconnaît comme un mot à exclure. Le mot est utilisé dans cet article pour des raisons de cohérence, car il s’agit du mot qui figure dans le logiciel. Une fois que le mot aura été supprimé du logiciel, cet article sera mis à jour en conséquence.
>

Quand vous avez créé votre serveur Azure Database for MariaDB, vous avez fourni un nom d’utilisateur et un mot de passe pour la connexion d’administrateur du serveur. Pour plus d’informations, vous pouvez suivre le [Guide de démarrage rapide](quickstart-create-mariadb-server-database-using-azure-portal.md). Vous pouvez localiser le nom d’utilisateur pour la connexion en tant qu’administrateur du serveur dans le portail Azure.

L’utilisateur administrateur de serveur possède les privilèges suivants pour votre serveur : SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER

Une fois le serveur Azure Database for MariaDB créé, vous pouvez utiliser le premier compte d’utilisateur Admin du serveur pour créer des utilisateurs supplémentaires et leur accorder un accès administrateur. Le compte administrateur du serveur peut être utilisé pour créer des utilisateurs possédant moins de privilèges et ayant accès à des schémas de base de données individuels.

> [!NOTE]
> Le privilège SUPER et le rôle DBA ne sont pas pris en charge. Pour comprendre ce qui n’est pas pris en charge dans le service, consultez les [privilèges](concepts-limits.md#privileges--data-manipulation-support) dans l’article sur les limitations.<br><br>
> Les plug-ins de mot de passe, tels que « validate_password » et « caching_sha2_password », ne sont pas pris en charge par le service.

## <a name="create-additional-admin-users"></a>Créer des utilisateurs administrateurs supplémentaires
1. Obtenez les informations de connexion et le nom d’utilisateur administrateur.
   Pour vous connecter à votre serveur de base de données, il vous faut le nom de serveur complet et les informations d’identification de connexion d’administrateur. Vous pouvez facilement localiser le nom du serveur et les informations de connexion sur la page **Vue d’ensemble** ou sur la page **Propriétés** du serveur dans le portail Azure. 

2. Utilisez le compte et le mot de passe d’administrateur pour vous connecter à votre serveur de base de données. Utilisez votre outil client préféré, tel que MySQL Workbench, mysql.exe, HeidiSQL ou d’autres. 
   Si vous ne savez pas comment vous connecter, consultez [Utilisation de MySQL Workbench pour vous connecter et interroger des données](./connect-workbench.md).

3. Modifiez et exécutez le code SQL suivant. Indiquez votre nouveau nom d’utilisateur sous la valeur de l’espace réservé `new_master_user`. Cette syntaxe accorde les privilèges répertoriés sur tous les schémas de base de données ( *.* ) au nom d’utilisateur (new_master_user dans cet exemple). 

   ```sql
   CREATE USER 'new_master_user'@'%' IDENTIFIED BY 'StrongPassword!';
   
   GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER ON *.* TO 'new_master_user'@'%' WITH GRANT OPTION; 
   
   FLUSH PRIVILEGES;
   ```

4. Vérifiez les privilèges accordés. 
   ```sql
   USE sys;
   
   SHOW GRANTS FOR 'new_master_user'@'%';
   ```

## <a name="create-database-users"></a>Créer des utilisateurs de base de données

1. Obtenez les informations de connexion et le nom d’utilisateur administrateur.
   Pour vous connecter à votre serveur de base de données, il vous faut le nom de serveur complet et les informations d’identification de connexion d’administrateur. Vous pouvez facilement localiser le nom du serveur et les informations de connexion sur la page **Vue d’ensemble** ou sur la page **Propriétés** du serveur dans le portail Azure. 

2. Utilisez le compte et le mot de passe d’administrateur pour vous connecter à votre serveur de base de données. Utilisez votre outil client préféré, tel que MySQL Workbench, mysql.exe, HeidiSQL ou d’autres. 
   Si vous ne savez pas comment vous connecter, consultez [Utilisation de MySQL Workbench pour vous connecter et interroger des données](./connect-workbench.md).

3. Modifiez et exécutez le code SQL suivant. Remplacez la valeur de l’espace réservé `db_user` par votre nouveau nom d’utilisateur prévu, et la valeur de l’espace réservé `testdb` par le nom à attribuer à la base de données.

   Cette syntaxe de code SQL crée une nouvelle base de données nommée testdb à titre d’exemple. Elle crée ensuite un utilisateur dans le service Azure Database for MariaDB et accorde tous les privilèges au nouveau schéma de base de données (testdb.\*) pour cet utilisateur. 

   ```sql
   CREATE DATABASE testdb;
   
   CREATE USER 'db_user'@'%' IDENTIFIED BY 'StrongPassword!';
   
   GRANT ALL PRIVILEGES ON testdb . * TO 'db_user'@'%';
   
   FLUSH PRIVILEGES;
   ```

4. Vérifiez les privilèges accordés dans la base de données.
   ```sql
   USE testdb;
   
   SHOW GRANTS FOR 'db_user'@'%';
   ```

5. Connectez-vous au serveur, en spécifiant la base de données désignée, à l’aide du nouveau nom d’utilisateur et mot de passe. Cet exemple montre la ligne de commande mysql. Cette commande vous invite à entrer le mot de passe pour le nom d’utilisateur. Indiquez votre propre nom de serveur, nom de base de données et nom d’utilisateur.

   ```bash
   mysql --host mydemoserver.mariadb.database.azure.com --database testdb --user db_user@mydemoserver -p
   ```
   Pour plus d’informations sur la gestion des comptes d’utilisateurs, consultez la documentation MariaDB relative à la [gestion des comptes d’utilisateurs](https://mariadb.com/kb/en/library/user-account-management/), à la [ syntaxe GRANT](https://mariadb.com/kb/en/library/grant/) et aux [privilèges](https://mariadb.com/kb/en/library/grant/#privilege-levels).

## <a name="azure_superuser"></a>azure_superuser

Tous les serveurs Azure Database pour MySQL sont créés avec un utilisateur appelé « azure_superuser ». Il s’agit d’un compte système créé par Microsoft pour gérer le serveur afin d’effectuer la surveillance, les sauvegardes et d’autres opérations de maintenance régulière. Les ingénieurs du support technique peuvent également utiliser ce compte pour accéder au serveur lors d’un incident d’authentification par certificat et il doivent demander l’accès à l’aide de processus juste-à-temps (JIT).

## <a name="next-steps"></a>Étapes suivantes
Ouvrez le pare-feu pour les adresses IP des machines des nouveaux utilisateurs afin de leur permettre de se connecter : [Créer et gérer des règles de pare-feu Azure Database for MariaDB à l’aide du Portail Azure](howto-manage-firewall-portal.md)  

<!--or [Azure CLI](howto-manage-firewall-using-cli.md).-->
