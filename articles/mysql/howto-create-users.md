---
title: Créer des bases de données et des utilisateurs Azure Database pour MySQL
description: Cet article explique comment créer des comptes d’utilisateur pour interagir avec un serveur Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 10/1/2020
ms.openlocfilehash: 2e934ede193d6efb9cc795c6b63cb485b88f792e
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541417"
---
# <a name="create-databases-and-users-in-azure-database-for-mysql"></a>Créer des bases de données et des utilisateurs dans Azure Database pour MySQL

[!INCLUDE[applies-to-single-flexible-server](includes/applies-to-single-flexible-server.md)]

Cet article explique comment créer des utilisateurs dans Azure Database pour MySQL.

> [!NOTE]
> **Communication sans biais**
>
> La diversité et l’inclusion sont au cœur des valeurs de Microsoft. Cet article contient des références au mot *esclave*. Le [guide de style de Microsoft sur la communication sans stéréotype](https://github.com/MicrosoftDocs/microsoft-style-guide/blob/master/styleguide/bias-free-communication.md) le reconnaît comme un mot à exclure. Le mot est utilisé dans cet article par souci de cohérence, car il s’agit du mot qui figure actuellement dans le logiciel. Une fois que le mot aura été supprimé du logiciel, cet article sera mis à jour en conséquence.
>

Quand vous avez créé votre serveur Azure Database pour MySQL, vous avez fourni un nom d’utilisateur et un mot de passe d’administrateur de serveur. Pour plus d’informations, consultez ce [démarrage rapide](quickstart-create-mysql-server-database-using-azure-portal.md). Vous pouvez déterminer le nom d’utilisateur de l’administrateur de serveur dans le portail Azure.

L’utilisateur administrateur du serveur dispose des privilèges suivants : 

   SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER


Après avoir créé un serveur Azure Database pour MySQL, vous pouvez utiliser le premier compte administrateur du serveur pour créer des utilisateurs supplémentaires et leur accorder un accès administrateur. Vous pouvez également utiliser le compte administrateur du serveur pour créer des utilisateurs possédant moins de privilèges et ayant accès à des schémas de base de données individuels.

> [!NOTE]
> Le privilège SUPER et le rôle DBA ne sont pas pris en charge. Pour comprendre ce qui n’est pas pris en charge dans le service, consultez les [privilèges](concepts-limits.md#privileges--data-manipulation-support) dans l’article sur les limitations.
>
> Les plug-ins de mot de passe comme `validate_password` et `caching_sha2_password` ne sont pas pris en charge par le service.


## <a name="to-create-a-database-with-a-non-admin-user-in-azure-database-for-mysql"></a>Pour créer une base de données avec un utilisateur non administrateur dans Azure Database pour MySQL

1. Obtenez les informations de connexion et le nom d’utilisateur administrateur.
   Pour vous connecter à votre serveur de base de données, il vous faut le nom de serveur complet et les informations d’identification de connexion d’administrateur. Vous pouvez facilement localiser le nom du serveur et les informations de connexion sur la page **Vue d’ensemble** ou sur la page **Propriétés** du serveur sur le portail Azure.

2. Utilisez le compte et le mot de passe d’administrateur pour vous connecter à votre serveur de base de données. Utilisez l’outil client de votre choix, tel que MySQL Workbench, mysql.exe ou HeidiSQL.
   
   Si vous ne savez pas comment vous connecter, consultez [Se connecter et interroger des données pour Serveur unique](./connect-workbench.md) ou [Se connecter et interroger des données pour Serveur flexible](./flexible-server/connect-workbench.md).

3. Modifiez et exécutez le code SQL suivant. Remplacez la valeur d’espace réservé `db_user` par le nouveau nom d’utilisateur que vous avez prévu d’utiliser. Remplacez la valeur d’espace réservé `testdb` par le nom de votre base de données.

   Ce code SQL crée une nouvelle base de données nommée testdb. Il crée ensuite un nouvel utilisateur dans le service MySQL et lui accorde tous les privilèges pour le nouveau schéma de la base de données (testdb.\*).

   ```sql
   CREATE DATABASE testdb;

   CREATE USER 'db_user'@'%' IDENTIFIED BY 'StrongPassword!';

   GRANT ALL PRIVILEGES ON testdb . * TO 'db_user'@'%';

   FLUSH PRIVILEGES;
   ```

4. Vérifiez les privilèges accordés dans la base de données :

   ```sql
   USE testdb;

   SHOW GRANTS FOR 'db_user'@'%';
   ```

5. Connectez-vous au serveur en spécifiant la base de données désignée et en utilisant les nouveaux nom d’utilisateur et mot de passe. Cet exemple montre la ligne de commande mysql. Lorsque vous utilisez cette commande, vous êtes invité à entrer le mot de passe de l’utilisateur. Utilisez le nom de votre serveur, le nom de votre base de données et votre nom d’utilisateur.

   # <a name="single-server"></a>[Serveur unique](#tab/single-server)

   ```azurecli-interactive
   mysql --host mydemoserver.mysql.database.azure.com --database testdb --user db_user@mydemoserver -p
   ```
   # <a name="flexible-server"></a>[Serveur flexible](#tab/flexible-server)

   ```azurecli-interactive
   mysql --host mydemoserver.mysql.database.azure.com --database testdb --user db_user -p
   ```
 ---

## <a name="to-create-additional-admin-users-in-azure-database-for-mysql"></a>Pour créer des utilisateurs administrateurs supplémentaires dans Azure Database pour MySQL

1. Obtenez les informations de connexion et le nom d’utilisateur administrateur.
   Pour vous connecter à votre serveur de base de données, il vous faut le nom de serveur complet et les informations d’identification de connexion d’administrateur. Vous pouvez facilement localiser le nom du serveur et les informations de connexion sur la page **Vue d’ensemble** ou sur la page **Propriétés** du serveur sur le portail Azure.

2. Utilisez le compte et le mot de passe d’administrateur pour vous connecter à votre serveur de base de données. Utilisez l’outil client de votre choix, tel que MySQL Workbench, mysql.exe ou HeidiSQL.
   
   Si vous ne savez pas comment vous connecter, consultez [Utiliser MySQL Workbench pour se connecter et interroger des données](./connect-workbench.md).

3. Modifiez et exécutez le code SQL suivant. Remplacez la valeur d’espace réservé `new_master_user` par votre nouveau nom d’utilisateur. Cette syntaxe accorde les privilèges répertoriés sur tous les schémas de base de données ( *.* ) à l’utilisateur (`new_master_user` dans cet exemple).

   ```sql
   CREATE USER 'new_master_user'@'%' IDENTIFIED BY 'StrongPassword!';

   GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER ON *.* TO 'new_master_user'@'%' WITH GRANT OPTION;

   FLUSH PRIVILEGES;
   ```

4. Vérifiez les privilèges accordés :

   ```sql
   USE sys;

   SHOW GRANTS FOR 'new_master_user'@'%';
   ```

## <a name="azure_superuser"></a>azure_superuser

Tous les serveurs Azure Database pour MySQL sont créés avec un utilisateur appelé « azure_superuser ». Il s’agit d’un compte système créé par Microsoft pour gérer le serveur afin d’effectuer la surveillance, les sauvegardes et d’autres opérations de maintenance régulière. Les ingénieurs du support technique peuvent également utiliser ce compte pour accéder au serveur lors d’un incident d’authentification par certificat et il doivent demander l’accès à l’aide de processus juste-à-temps (JIT).

## <a name="next-steps"></a>Étapes suivantes

Ouvrez le pare-feu pour les adresses IP des machines des nouveaux utilisateurs afin de leur permettre de se connecter :
- [Créer et gérer des règles de pare-feu sur un serveur unique](howto-manage-firewall-using-portal.md) 
- [Créer et gérer des règles de pare-feu sur un serveur flexible](flexible-server/how-to-connect-tls-ssl.md)

Pour plus d’informations sur la gestion des comptes d’utilisateurs, consultez la documentation du produit MySQL relative à la [gestion des comptes d’utilisateurs](https://dev.mysql.com/doc/refman/5.7/en/access-control.html), à la [ syntaxe GRANT](https://dev.mysql.com/doc/refman/5.7/en/grant.html) et aux [privilèges](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html).
