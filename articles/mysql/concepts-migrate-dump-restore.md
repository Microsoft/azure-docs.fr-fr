---
title: Migrer par sauvegarde et restauration - Azure Database pour MySQL
description: Cet article décrit deux méthodes courantes pour sauvegarder et restaurer des bases de données dans votre base de données Azure pour MySQL en utilisant des outils comme mysqldump, MySQL Workbench et PHPMyAdmin.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 10/30/2020
ms.openlocfilehash: f21587fe6a48d042ed98c126beb2a7dcaa39b7d8
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94537915"
---
# <a name="migrate-your-mysql-database-to-azure-database-for-mysql-using-dump-and-restore"></a>Migrer une base de données MySQL vers une base de données Azure pour MySQL à l’aide des images mémoire et de la restauration

[!INCLUDE[applies-to-single-flexible-server](includes/applies-to-single-flexible-server.md)]

Cet article décrit deux méthodes courantes pour sauvegarder et restaurer des bases de données dans votre base de données Azure pour MySQL.
- Sauvegarder et restaurer à partir de la ligne de commande (en utilisant mysqldump)
- Sauvegarder et restaurer à l’aide de PHPMyAdmin

Vous pouvez également consulter le [guide relatif à la migration de base de données](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide) pour obtenir des informations détaillées et des cas d’usage sur la migration de base de données vers Azure Database pour MySQL. Ce guide fournit des conseils pour réussir la planification et l’exécution d’une migration MySQL vers Azure.

## <a name="before-you-begin"></a>Avant de commencer
Pour parcourir ce guide pratique, vous aurez besoin des éléments suivants :
- [Créer un serveur de base de données Azure pour MySQL - Portail Azure](quickstart-create-mysql-server-database-using-azure-portal.md)
- Utilitaire de ligne de commande [mysqldump](https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html) installé sur une machine
- [MySQL Workbench](https://dev.mysql.com/downloads/workbench/) ou un autre outil MySQL tiers pour exécuter les commandes de sauvegarde et de restauration.

> [!TIP]
> Si vous souhaitez migrer des bases de données volumineuses avec des tailles de base de données supérieures à 1 To, vous pouvez utiliser des outils de la communauté comme **mydumper/myloader** prenant en charge l’exportation et l’importation parallèles. Découvrez [Comment migrer des bases de données MySQL volumineuses](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/best-practices-for-migrating-large-databases-to-azure-database/ba-p/1362699).


## <a name="common-use-cases-for-dump-and-restore"></a>Cas d’usage courants de la vidage et de restauration

Les cas d’utilisation courants sont les suivants :

- **Déplacement à partir d’un autre fournisseur de services gérés** : la plupart des fournisseurs de services gérés peuvent ne pas fournir l’accès au fichier de stockage physique pour des raisons de sécurité, de sorte que la sauvegarde et la restauration logiques constituent la seule option pour la migration.
- **Migration à partir d’un environnement local ou d’une machine virtuelle** : Azure Database pour MySQL ne prend pas en charge la restauration des sauvegardes physiques qui effectuent la sauvegarde et la restauration logiques comme approche unique.
- **Déplacement de votre stockage de sauvegarde depuis l’espace de stockage localement redondant vers le stockage géo-redondant** : la base de données Azure Database pour MySQL permet de configurer un stockage localement redondant ou géo-redondant, car la sauvegarde est uniquement autorisée lors de la création du serveur. Une fois que le serveur est approvisionné, vous ne pouvez pas modifier l’option de redondance du stockage de sauvegarde. Pour déplacer votre stockage de sauvegarde d’un stockage localement redondant vers un stockage géo-redondant, le vidage et la restauration sont la seule option. 
-  **Migration à partir de moteurs de stockage alternatifs vers InnoDB** : Azure Database pour MySQL prend en charge le moteur de stockage InnoDB uniquement. Par conséquent, elle ne prend pas en charge les autres moteurs de stockage. Si vos tables sont configurées avec d’autres moteurs de stockage, convertissez-les au format de moteur InnoDB avant la migration vers la base de données Azure pour MySQL.

    Par exemple, si votre site WordPress ou votre application web utilise les tables MyISAM, commencez par convertir ces tables en effectuant une migration vers le format InnoDB avant la restauration vers la base de données Azure pour MySQL. Utilisez la clause `ENGINE=InnoDB` pour définir le moteur utilisé lors de la création d’une table, puis transférez les données dans la table compatible avant la restauration.

   ```sql
   INSERT INTO innodb_table SELECT * FROM myisam_table ORDER BY primary_key_columns
   ```
> [!Important]
> - Pour éviter les problèmes de compatibilité, assurez-vous d’utiliser la même version de MySQL sur les systèmes source et de destination lors de la sauvegarde des bases de données. Par exemple, si votre serveur MySQL existant est en version 5.7, vous devriez effectuer une migration vers une base de données Azure pour MySQL configurée pour exécuter la version 5.7. La commande `mysql_upgrade` ne fonctionne pas avec un serveur de base de données Azure pour MySQL et n’est pas prise en charge. 
> - Si vous devez mettre à niveau des versions de MySQL, commencez par sauvegarder ou exporter votre base de données de version inférieure vers une version ultérieure de MySQL dans votre propre environnement. Ensuite, exécutez `mysql_upgrade` avant de tenter une migration vers une base de données Azure pour MySQL.

## <a name="performance-considerations"></a>Considérations relatives aux performances
Pour optimiser les performances, tenez compte des considérations suivantes lors de la sauvegarde de bases de données volumineuses :
-   Utilisez l’option `exclude-triggers` dans mysqldump lors de la sauvegarde de bases de données. Excluez les déclencheurs des fichiers de sauvegarde pour éviter tout déclenchement par les commandes correspondantes pendant la restauration des données.
-   Utilisez l’option `single-transaction` pour définir le mode d’isolation de transaction sur REPEATABLE READ et envoyer une instruction START TRANSACTION SQL au serveur avant le vidage des données. Le vidage de nombreuses tables en une seule transaction provoque la consommation de stockage supplémentaire pendant la restauration. Les options `single-transaction` et `lock-tables` s’excluent mutuellement, car LOCK TABLES provoque la validation implicite des transactions en attente. Pour vider des tables volumineuses, combinez l’option `single-transaction` avec l’option `quick`.
-   Utilisez la syntaxe pour lignes multiples `extended-insert` qui inclut plusieurs listes VALUE. Cela génère un fichier de vidage plus petit et accélère les insertions lors du rechargement du fichier.
-  Utilisez l’option `order-by-primary` dans mysqldump lors de la sauvegarde de bases de données, afin que les données soient scriptées selon l’ordre des clés primaires.
-   Utilisez l’option `disable-keys` dans mysqldump lors de la sauvegarde des données pour désactiver les contraintes de clé étrangère avant le chargement. La désactivation des vérifications de clé étrangère offre des gains de performances. Activez les contraintes et vérifiez les données après le chargement pour garantir l’intégrité référentielle.
-   Utilisez des tables partitionnées le cas échéant.
-   Chargez les données en parallèle. Évitez tout parallélisme excessif, qui vous ferait atteindre une limite de ressources, et surveillez les ressources à l’aide des mesures disponibles dans le Portail Azure.
-   Utilisez l’option `defer-table-indexes` dans mysqlpump lors de la sauvegarde des bases de données, afin que l’index soit créé après le chargement des données de tables.
-   Utilisez l’option `skip-definer` dans mysqlpump afin d’ignorer les clauses DEFINER et SQL SECURITY dans les instructions CREATE pour les vues et les procédures stockées.  Lorsque vous rechargez le fichier de vidage, il crée des objets qui utilisent les valeurs DEFINER et SQL SECURITY par défaut.
-   Copiez les fichiers de sauvegarde dans un magasin/objet blob Azure et effectuez la restauration à partir de celui-ci. Cette opération doit être beaucoup plus rapide que l’exécution de la restauration sur Internet.

## <a name="create-a-database-on-the-target-azure-database-for-mysql-server"></a>Créer une base de données sur le serveur cible de base de données Azure pour MySQL
Créez une base de données vide sur le serveur cible de base de données Azure pour MySQL vers lequel vous souhaitez effectuer la migration des données. Utilisez un outil tel que MySQL Workbench ou mysql.exe pour créer la base de données. La base de données peut avoir le même nom que celle qui contient les données capturées, mais vous pouvez également créer une base de données avec un autre nom.

Pour vous connecter, repérez les informations de connexion dans la **Vue d’ensemble** de votre Azure Database pour MySQL.

:::image type="content" source="./media/concepts-migrate-dump-restore/1_server-overview-name-login.png" alt-text="Trouver les informations de connexion dans le Portail Azure":::

Ajoutez les informations de connexion à MySQL Workbench.

:::image type="content" source="./media/concepts-migrate-dump-restore/2_setup-new-connection.png" alt-text="Chaîne de connexion MySQL Workbench":::

## <a name="preparing-the-target-azure-database-for-mysql-server-for-fast-data-loads"></a>Préparation du serveur Azure Database pour MySQL afin d’accélérer les chargements de données
Pour préparer le serveur Azure Database pour MySQL cible afin d’accélérer les chargements de données, les paramètres et la configuration de serveur suivants doivent être modifiés.
- max_allowed_packet : définissez cette valeur sur 1073741824 (par exemple, 1 Go) pour éviter tout problème de dépassement de capacité en raison de longues lignes.
- slow_query_log : définissez cette valeur sur OFF pour désactiver le journal des requêtes lentes. Cela élimine la surcharge causée par la journalisation de requêtes lentes pendant les chargements de données.
- query_store_capture_mode : définissez la valeur sur NONE pour désactiver le Magasin de données des requêtes. Cela permet d’éliminer la surcharge causée par les activités d’échantillonnage par le magasin de données des requêtes.
- innodb_buffer_pool_size : effectuez un scale-up du serveur avec la référence SKU 32 vCore Memory Optimized à partir du niveau tarifaire du portail pendant la migration afin d’augmenter la valeur innodb_buffer_pool_size. La valeur Innodb_buffer_pool_size peut être augmentée uniquement en effectuant un scale-up du calcul pour le serveur Azure Database pour MySQL.
- innodb_io_capacity & innodb_io_capacity_max : passez à 9000 à partir des paramètres de serveur dans le Portail Azure pour améliorer l’utilisation des E/S afin d’optimiser la vitesse de la migration.
- innodb_write_io_threads & innodb_write_io_threads : affectez la valeur 4 aux paramètres du serveur dans le Portail Azure pour accélérer la migration.
- Effectuer un scale-up du niveau de stockage : le nombre d’opérations d’E/S par seconde (IOPS) pour le serveur Azure Database pour MySQL augmente progressivement. Pour accélérer les chargements, vous pouvez augmenter le niveau de stockage et accroître ainsi les IOPS. N’oubliez pas que le stockage peut seulement faire l’objet d’un scale-up.

Une fois la migration terminée, vous pouvez rétablir les valeurs précédentes des paramètres du serveur et du niveau de calcul.

## <a name="dump-and-restore-using-mysqldump-utility"></a>Vider et restaurer à l’aide de l’utilitaire mysqldump

### <a name="create-a-backup-file-from-the-command-line-using-mysqldump"></a>Créer un fichier de sauvegarde en ligne de commande avec mysqldump
Pour sauvegarder une base de données MySQL existante sur le serveur local ou sur une machine virtuelle, exécutez la commande suivante :
```bash
$ mysqldump --opt -u [uname] -p[pass] [dbname] > [backupfile.sql]
```

Voici les paramètres à fournir :
- [uname] le nom d’utilisateur de votre base de données ;
- [pass] le mot de passe de votre base de données (remarque : il n’y a pas d’espace entre -p et le mot de passe) ;
- [dbname] le nom de votre base de données ;
- [backupfile.sql] le nom du fichier de sauvegarde de votre base de données ;
- [--opt] l’option mysqldump.

Par exemple, pour sauvegarder une base de données nommée « testdb » sur votre serveur MySQL avec le nom d’utilisateur « testuser » et sans mot de passe dans un fichier testdb_backup.sql, utilisez la commande suivante. La commande sauvegarde la base de données `testdb` dans un fichier appelé `testdb_backup.sql`, qui contient toutes les instructions SQL nécessaires pour recréer la base de données. Assurez-vous que le nom d’utilisateur « testuser » possède au moins le privilège SELECT pour les tableaux vidés, SHOW VIEW pour les vues de vidage, TRIGGER pour les déclencheurs vidés et LOCK TABLES si l’option --single-transaction n’est pas utilisée.

```bash
GRANT SELECT, LOCK TABLES, SHOW VIEW ON *.* TO 'testuser'@'hostname' IDENTIFIED BY 'password';
```
Exécutez maintenant mysqldump pour créer la sauvegarde de la base de données `testdb`

```bash
$ mysqldump -u root -p testdb > testdb_backup.sql
```
Pour sélectionner certaines tables en particulier à sauvegarder dans votre base de données, listez les noms de table en les séparant par des espaces. Par exemple, pour sauvegarder seulement les tables table1 et table2 de « testdb », suivez cet exemple :

```bash
$ mysqldump -u root -p testdb table1 table2 > testdb_tables_backup.sql
```
Pour sauvegarder plusieurs bases de données à la fois, utilisez le commutateur --database et listez les noms de bases de données en les séparant par des espaces.
```bash
$ mysqldump -u root -p --databases testdb1 testdb3 testdb5 > testdb135_backup.sql
```

### <a name="restore-your-mysql-database-using-command-line-or-mysql-workbench"></a>Restaurer votre base de données MySQL à l’aide d’une ligne de commande ou de MySQL Workbench
Une fois que vous avez créé la base de données cible, vous pouvez utiliser la commande mysql ou MySQL Workbench pour restaurer les données dans la base de données créée à cet effet à partir du fichier d’image mémoire.
```bash
mysql -h [hostname] -u [uname] -p[pass] [db_to_restore] < [backupfile.sql]
```
Dans cet exemple, vous restaurez les données dans la base de données nouvellement créée sur le serveur cible de base de données Azure pour MySQL.

Voici un exemple d’utilisation de **mysql** pour **Serveur unique** :

```bash
$ mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p testdb < testdb_backup.sql
```
Voici un exemple d’utilisation de **mysql** pour **Serveur flexible** :

```bash
$ mysql -h mydemoserver.mysql.database.azure.com -u myadmin -p testdb < testdb_backup.sql
```
---

## <a name="dump-and-restore-using-phpmyadmin"></a>Sauvegarder et restaurer à l’aide de PHPMyAdmin
Suivez les étapes ci-dessous pour vider et restaurer un base de données avec PHPMyadmin.

> [!NOTE]
> Pour un serveur unique, le nom d’utilisateur doit être au format « username@servername », mais pour un serveur flexible, vous pouvez simplement utiliser « username » si vous utilisez « username@servername » pour le serveur flexible, la connexion échoue.

### <a name="export-with-phpmyadmin"></a>Exporter avec PHPMyadmin
Pour exporter, vous pouvez utiliser l’outil courant phpMyAdmin, que vous avez peut-être déjà installé localement dans votre environnement. Pour exporter votre base de données MySQL avec PHPMyAdmin :
1. Ouvrez phpMyAdmin.
2. Sélectionnez votre base de données. Cliquez sur le nom de la base de données dans la liste de gauche.
3. Cliquer sur le lien **Exporter**. Une nouvelle page s’affiche, indiquant la sauvegarde de la base de données.
4. Dans la zone Exporter, cliquez sur le lien **Sélectionner tout** pour choisir les tables de votre base de données.
5. Dans la zone des options SQL, cliquez sur les options appropriées.
6. Cliquez sur l’option **Enregistrer en tant que fichier** et sur l’option de compression correspondante, puis sur le bouton **OK**. Une boîte de dialogue vous invite à enregistrer le fichier en local.

### <a name="import-using-phpmyadmin"></a>Importer avec PHPMyAdmin
L’importation d’une base de données est similaire à l’exportation. Effectuez les actions suivantes :
1. Ouvrez phpMyAdmin.
2. Dans la page de configuration phpMyAdmin, cliquez sur **Ajouter** pour ajouter votre serveur de base de données Azure pour MySQL. Indiquez les détails et informations de connexion.
3. Créez une base de données en lui donnant un nom approprié et sélectionnez-la à gauche de l’écran. Pour réécrire la base de données existante, cliquez sur le nom de la base de données, activez toutes les cases à cocher en regard des noms de table et sélectionnez **Supprimer** pour supprimer les tables existantes.
4. Cliquez sur le lien **SQL** pour afficher la page dans laquelle vous pouvez taper des commandes SQL ou charger votre fichier SQL.
5. Utilisez le bouton **Parcourir** pour trouver le fichier de base de données.
6. Cliquez sur le bouton **OK** pour exporter la sauvegarde, exécuter les commandes SQL et recréer votre base de données.

## <a name="known-issues"></a>Problèmes connus
Pour les problèmes connus, les trucs et astuces, nous vous recommandons de consulter notre blog [techcommunity](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/tips-and-tricks-in-using-mysqldump-and-mysql-restore-to-azure/ba-p/916912).

## <a name="next-steps"></a>Étapes suivantes
- [Connecter des applications à Azure Database pour MySQL](./howto-connection-string.md).
- Pour plus d’informations sur la migration de bases de données vers Azure Database pour MySQL, consultez le [Guide de migration des bases de données](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide).
- Si vous souhaitez migrer des bases de données volumineuses avec des tailles de base de données supérieures à 1 To, vous pouvez utiliser des outils de la communauté comme **mydumper/myloader** prenant en charge l’exportation et l’importation parallèles. Découvrez [Comment migrer des bases de données MySQL volumineuses](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/best-practices-for-migrating-large-databases-to-azure-database/ba-p/1362699).
