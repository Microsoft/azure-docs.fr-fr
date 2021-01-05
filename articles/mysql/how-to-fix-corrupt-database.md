---
title: Résoudre une corruption de base de données – Azure Database pour MySQL
description: Dans cet article, vous allez découvrir comment corriger des problèmes de corruption de base de données dans Azure Database pour MySQL.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 09/21/2020
ms.openlocfilehash: 21e4189e56f704129710da5b1d39613c4e1b1df5
ms.sourcegitcommit: d9ba60f15aa6eafc3c5ae8d592bacaf21d97a871
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/06/2020
ms.locfileid: "91766887"
---
# <a name="troubleshoot-database-corruption-in-azure-database-for-mysql"></a>Résoudre des problèmes de corruption de base de données dans Azure Database pour MySQL
[!INCLUDE[applies-to-single-flexible-server](includes/applies-to-single-flexible-server.md)]

La corruption d’une base de données peut entraîner un temps d’arrêt de votre application. Il est également essentiel de résoudre les problèmes de corruption à temps afin d’éviter toute perte de données. En cas de corruption d’une base de données, l’erreur suivante s’affiche dans les journaux de votre serveur : `InnoDB: Database page corruption on disk or a failed.`

Dans cet article, vous allez apprendre à résoudre les problèmes de corruption d’une base de données ou d’une table. Azure Database pour MySQL utilise le moteur InnoDB. Il intègre des opérations automatisées de vérification et de réparation de la corruption. InnoDB recherche les pages corrompues en exécutant des sommes de contrôle sur chaque page qu’il lit. S’il trouve une incohérence en termes de somme de contrôle, il arrête automatiquement le serveur MySQL.

Essayez les options suivantes pour atténuer rapidement les problèmes de corruption de votre base de données.

## <a name="restart-your-mysql-server"></a>Redémarrer votre serveur MySQL

En général, vous remarquez qu’une base de données ou une table est corrompue lorsque votre application y accède. InnoDB intègre un mécanisme de récupération après incident capable de résoudre la plupart des problèmes lors d’un redémarrage du serveur. Par conséquent, le redémarrage du serveur peut aider celui-ci à récupérer d’un incident à l’origine de l’état incorrect de la base de données.

## <a name="use-the-dump-and-restore-method"></a>Utiliser la méthode vidage sur incident et restauration

Nous vous recommandons de résoudre les problèmes de corruption à l’aide d’une méthode *vidage sur incident et restauration*. Cette méthode implique les opérations suivantes :
1. Accès à la table corrompue.
1. Utilisation de l’utilitaire mysqldump pour créer une sauvegarde logique de la table. La sauvegarde conserve la structure de la table et les données qu’elle contient.
1. Rechargement de la table dans la base de données.

### <a name="back-up-your-database-or-tables"></a>Sauvegarder vos bases de données ou tables

> [!Important]
> - Assurez-vous d’avoir configuré une règle de pare-feu pour accéder au serveur à partir de votre ordinateur client. Pour plus d’informations, consultez [configurer une règle de pare-feu sur un serveur unique](howto-manage-firewall-using-portal.md) et [configurer une règle de pare-feu sur un serveur flexible](flexible-server/how-to-connect-tls-ssl.md).
> - Utilisez l’option SSL `--ssl-cert` pour l’utilitaire mysqldump si SSL est activé.

Créez un fichier de sauvegarde en ligne de commande avec mysqldump. Utilisez la commande suivante :

```
$ mysqldump [--ssl-cert=/path/to/pem] -h [host] -u [uname] -p[pass] [dbname] > [backupfile.sql]
```

Descriptions des paramètres :
- `[ssl-cert=/path/to/pem]` : Chemin d’accès au certificat SSL. Téléchargez le certificat SSL sur votre ordinateur client et définissez le chemin d’accès dans la commande. N’utilisez pas ce paramètre si le protocole SSL est désactivé.
- `[host]` : Votre serveur Azure Database pour MySQL.
- `[uname]` : Nom de votre utilisateur administrateur de serveur.
- `[pass]` : Mot de passe de votre utilisateur administrateur.
- `[dbname]` : Nom de votre base de données.
- `[backupfile.sql]` : Nom du fichier de la sauvegarde de votre base de données.

> [!Important]
> - Pour un serveur unique, utilisez le format `admin-user@servername` pour remplacer `myserveradmin` dans les commandes suivantes.
> - Pour un serveur flexible, utilisez le format `admin-user` pour remplacer `myserveradmin` dans les commandes suivantes.

Si une table spécifique est corrompue, sélectionnez des tables spécifiques dans votre base de données à sauvegarder :
```
$ mysqldump --ssl-cert=</path/to/pem> -h mydemoserver.mysql.database.azure.com -u myserveradmin -p testdb table1 table2 > testdb_tables_backup.sql
```

Pour sauvegarder une ou plusieurs bases de données, utilisez le commutateur `--database` et répertoriez les noms de base de données séparés par des espaces :

```
$ mysqldump --ssl-cert=</path/to/pem>  -h mydemoserver.mysql.database.azure.com -u myserveradmin -p --databases testdb1 testdb3 testdb5 > testdb135_backup.sql
```

### <a name="restore-your-database-or-tables"></a>Restaurer votre base ou vos tables

Les étapes suivantes montrent comment restaurer votre base de données ou vos tables. Une fois le fichier de sauvegarde créé, vous pouvez restaurer les tables ou les bases de données à l’aide de l’utilitaire mysql. Exécutez cette commande :

```
mysql  --ssl-cert=</path/to/pem> -h [hostname] -u [uname] -p[pass] [db_to_restore] < [backupfile.sql]
```
Voici un exemple de restauration de `testdb` à partir d’un fichier de sauvegarde créé avec l’utilitaire mysqldump : 

> [!Important]
> - Pour un serveur unique, utilisez le format `admin-user@servername` pour remplacer `myserveradmin` dans la commande suivante.
> - Pour un serveur flexible, utilisez le format ```admin-user``` pour remplacer `myserveradmin` dans la commande suivante. 

```
$ mysql --ssl-cert=</path/to/pem> -h mydemoserver.mysql.database.azure.com -u myserveradmin -p testdb < testdb_backup.sql
```

## <a name="next-steps"></a>Étapes suivantes
Si les étapes ci-dessus ne permettent pas de résoudre le problème, vous pouvez toujours restaurer le serveur entier :
- [Restaurer un serveur dans Azure Database pour MySQL - Serveur unique](howto-restore-server-portal.md)
- [Restaurer un serveur dans Azure Database pour MySQL - Serveur flexible](flexible-server/how-to-restore-server-portal.md)



