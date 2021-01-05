---
title: Configurer la réplication des données entrantes - Azure Database pour MySQL
description: Cet article décrit comment configurer la réplication des données entrantes pour Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 9/29/2020
ms.openlocfilehash: b7f1f16b5182658f42ad6594aace22fb5a1a80fc
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541400"
---
# <a name="how-to-configure-azure-database-for-mysql-data-in-replication"></a>Comment configurer Azure Database pour MySQL pour la réplication de données entrantes MySQL

Cet article décrit comment configurer [Réplication des données entrantes](concepts-data-in-replication.md) dans Azure Database pour MySQL en configurant les serveurs source et réplica. Cet article suppose que vous avez déjà utilisé des serveurs et des bases de données MySQL.

> [!NOTE]
> Communication sans stéréotype
>
> La diversité et l’inclusion sont au cœur des valeurs de Microsoft. Cet article contient des références au mot _esclave_. Le [guide de style de Microsoft sur la communication sans stéréotype](https://github.com/MicrosoftDocs/microsoft-style-guide/blob/master/styleguide/bias-free-communication.md) le reconnaît comme un mot à exclure. Le mot est utilisé dans cet article pour des raisons de cohérence, car il s’agit du mot qui figure dans le logiciel. Une fois que le mot aura été supprimé du logiciel, cet article sera mis à jour en conséquence.
>

Pour créer un réplica dans le service Azure Database pour MySQL, [Réplication des données entrantes](concepts-data-in-replication.md) synchronise les données provenant d’un serveur MySQL source qui s’exécute en local dans des machines virtuelles ou des services de base de données cloud. La réplication des données est basée sur une réplication selon la position du fichier journal binaire (binlog) native à MySQL. Pour en savoir plus sur la réplication binlog, consultez la [vue d’ensemble de la réplication binlog MySQL](https://dev.mysql.com/doc/refman/5.7/en/binlog-replication-configuration-overview.html).

Passez en revue les [limitations et conditions requises](concepts-data-in-replication.md#limitations-and-considerations) de la Réplication des données entrantes avant de suivre les étapes décrites dans cet article.

## <a name="create-a-mysql-server-to-be-used-as-replica"></a>Créer un serveur MySQL à utiliser en tant que réplica

1. Créer un nouveau serveur Azure Database pour MySQL

   Créez un nouveau serveur MySQL (par exemple, « replica.mysql.database.azure.com »). Reportez-vous à [Création d’un serveur Azure Database pour MySQL à l’aide du portail Azure](quickstart-create-mysql-server-database-using-azure-portal.md) pour la création de serveurs. Ce serveur est le serveur « réplica » dans la réplication de données entrantes.

   > [!IMPORTANT]
   > Ce serveur Azure Database pour MySQL doit être créé dans les niveaux de tarification à usage général ou à mémoire optimisée.
   > 

1. Créer les mêmes comptes d’utilisateur et les privilèges correspondants

   Les comptes d’utilisateur ne sont pas répliqués à partir du serveur source vers le serveur réplica. Si vous prévoyez de fournir aux utilisateurs un accès au serveur réplica, vous devez créer manuellement tous les comptes et privilèges correspondants sur ce nouveau serveur Azure Database pour MySQL.

1. Ajoutez l’adresse IP du serveur source aux règles de pare-feu du réplica. 

   Mettez à jour les règles de pare-feu à l’aide du [portail Azure](howto-manage-firewall-using-portal.md) ou d’[Azure CLI](howto-manage-firewall-using-cli.md).

## <a name="configure-the-source-server"></a>Configurer le serveur source
Les étapes suivantes servent à préparer et à configurer le serveur MySQL hébergé localement, dans une machine virtuelle ou par un service de base de données hébergé par d’autres fournisseurs de cloud pour la réplication de données entrantes. Ce serveur est le « maître » dans la réplication de données entrantes.


1. Avant de continuer, vérifiez la [configuration requise pour le serveur maître](concepts-data-in-replication.md#requirements). 

2. Assurez-vous que le serveur source autorise le trafic entrant et sortant sur le port 3306, que le serveur source dispose d’une **IP publique** et que le DNS est accessible publiquement ou possède un nom de domaine complet (FQDN). 
   
   Testez la connectivité au serveur source en tentant de vous connecter à partir d’un outil tel que la ligne de commande MySQL hébergée sur un autre ordinateur ou à partir d’[Azure Cloud Shell](../cloud-shell/overview.md) accessible sur le portail Azure.

   Si votre organisation a des stratégies de sécurité strictes et qu’elle n’autorise pas toutes les adresses IP sur le serveur source à permettre la communication d’Azure vers votre serveur source, vous pouvez éventuellement utiliser la commande ci-dessous pour déterminer l’adresse IP de votre serveur MySQL.

   1. Connectez-vous à Azure Database pour MySQL à l’aide d’un outil tel que la ligne de commande MySQL.
   2. Exécutez la requête ci-dessous.
      ```bash
      mysql> SELECT @@global.redirect_server_host;
      ```
      Voici un exemple de sortie :
      ```bash 
      +-----------------------------------------------------------+
      | @@global.redirect_server_host                             |
      +-----------------------------------------------------------+
      | e299ae56f000.tr1830.westus1-a.worker.database.windows.net |
       +-----------------------------------------------------------+
      ```
   3. Quittez la ligne de commande MySQL.
   4. Exécutez le code ci-dessous dans l’utilitaire Ping pour récupérer l’adresse IP.
      ```bash
      ping <output of step 2b>
      ``` 
      Par exemple : 
      ```bash      
      C:\Users\testuser> ping e299ae56f000.tr1830.westus1-a.worker.database.windows.net
      Pinging tr1830.westus1-a.worker.database.windows.net (**11.11.111.111**) 56(84) bytes of data.
      ```

   5. Configurez les règles de pare-feu de votre serveur source pour inclure l’adresse IP de sortie de l’étape précédente sur le port 3306.

   > [!NOTE]
   > Cette adresse IP peut changer en raison d’opérations de maintenance ou de déploiement. Cette méthode de connectivité concerne uniquement les clients qui ne peuvent pas se permettre d’autoriser toutes les adresses IP sur le port 3306.
   
1. Activer la journalisation binaire

   Vérifiez si la journalisation binaire a été activée sur le serveur source en exécutant la commande suivante : 

   ```sql
   SHOW VARIABLES LIKE 'log_bin';
   ```

   Si la variable [`log_bin`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_log_bin) est renvoyée avec la valeur « ON », la journalisation binaire est activée sur votre serveur. 

   Si `log_bin` est renvoyé avec la valeur « OFF », activez la journalisation binaire en modifiant votre fichier my.cnf ainsi que `log_bin=ON` et redémarrez le serveur pour que la modification prenne effet.

1. Paramètres du serveur source

   Réplication des données entrantes nécessite que le paramètre `lower_case_table_names` soit cohérent entre les serveurs source et réplica. Par défaut, ce paramètre est 1 dans Azure Database pour MySQL. 

   ```sql
   SET GLOBAL lower_case_table_names = 1;
   ```

1. Créer un nouveau rôle de réplication et définir une autorisation

   Créez un compte d’utilisateur sur le serveur source configuré avec des privilèges de réplication. Cela est possible via des commandes SQL ou un outil tel que MySQL Workbench. Si vous prévoyez une réplication avec SSL, cela doit être spécifié lors de la création de l’utilisateur. Pour comprendre comment [ajouter des comptes d’utilisateur](https://dev.mysql.com/doc/refman/5.7/en/user-names.html) sur votre serveur source, reportez-vous à la documentation MySQL. 

   Dans les commandes ci-dessous, le nouveau rôle de réplication créé peut accéder au serveur source à partir de n’importe quelle machine, et pas seulement de celle qui héberge le serveur source. Cela s’effectue en spécifiant « syncuser@’%’ » dans la commande de création d’un utilisateur. Consultez la documentation MySQL pour en savoir plus sur la [définition des noms de compte](https://dev.mysql.com/doc/refman/5.7/en/account-names.html).

   **Commande SQL**

   *Réplication avec SSL*

   Pour exiger le protocole SSL pour toutes les connexions utilisateur, utilisez la commande suivante pour créer un utilisateur : 

   ```sql
   CREATE USER 'syncuser'@'%' IDENTIFIED BY 'yourpassword';
   GRANT REPLICATION SLAVE ON *.* TO ' syncuser'@'%' REQUIRE SSL;
   ```

   *Réplication sans SSL*

   Si le protocole SSL n’est pas requis pour toutes les connexions, utilisez la commande suivante pour créer un utilisateur :

   ```sql
   CREATE USER 'syncuser'@'%' IDENTIFIED BY 'yourpassword';
   GRANT REPLICATION SLAVE ON *.* TO ' syncuser'@'%';
   ```

   **MySQL Workbench**

   Pour créer le rôle de réplication dans MySQL Workbench, ouvrez le panneau **Utilisateurs et privilèges** à partir du panneau **Gestion**. Cliquez ensuite sur **Ajouter un compte**. 
 
   :::image type="content" source="./media/howto-data-in-replication/users_privileges.png" alt-text="Utilisateurs et privilèges":::

   Tapez le nom d’utilisateur dans le champ **Nom de connexion**. 

   :::image type="content" source="./media/howto-data-in-replication/syncuser.png" alt-text="Synchroniser l’utilisateur":::
 
   Cliquez sur le panneau **Rôles administratifs**, puis sélectionnez **Subordonné de réplication** dans la liste **Privilèges globaux**. Cliquez ensuite sur **Appliquer** pour créer le rôle de réplication.

   :::image type="content" source="./media/howto-data-in-replication/replicationslave.png" alt-text="Subordonné de réplication":::

1. Définissez le serveur source en mode lecture seule.

   Avant de commencer à vider la base de données, le serveur doit être placé en mode lecture seule. En mode lecture seule, le serveur source ne peut traiter aucune transaction d’écriture. Évaluer l’impact sur votre entreprise et planifiez la fenêtre de lecture seule lors d’une période creuse, si nécessaire.

   ```sql
   FLUSH TABLES WITH READ LOCK;
   SET GLOBAL read_only = ON;
   ```

1. Obtenir le nom du fichier du journal binaire et le décalage

   Exécutez la commande [`show master status`](https://dev.mysql.com/doc/refman/5.7/en/show-master-status.html) afin de déterminer le nom de fichier du journal binaire actuel et le décalage.
    
   ```sql
    show master status;
   ```
   Les résultats doivent être comme suit. Assurez-vous de noter le nom du fichier binaire, tel qu’il sera utilisé lors des étapes ultérieures.

   :::image type="content" source="./media/howto-data-in-replication/masterstatus.png" alt-text="Résultats de l’état récapitulatif":::
 
## <a name="dump-and-restore-source-server"></a>Vider et restaurer le serveur source

1. Déterminez les bases de données et les tables que vous souhaitez répliquer dans Azure Database pour MySQL et effectuez la copie de sauvegarde à partir du serveur source.
 
    Vous pouvez utiliser mysqldump pour vider les bases de données de votre serveur maître. Pour plus d’informations, reportez-vous à [Dump & Restore](concepts-migrate-dump-restore.md) (Vider et restaurer). Il n’est pas nécessaire de vider les bibliothèques MySQL et de test.

1. Définissez le serveur source en mode lecture/écriture.

   Une fois la base de données vidée, remettez le serveur MySQL source en mode de lecture/écriture.

   ```sql
   SET GLOBAL read_only = OFF;
   UNLOCK TABLES;
   ```

1. Restaurer le fichier de vidage sur le nouveau serveur

   Restaurez le fichier de vidage sur le serveur créé dans le service Azure Database pour MySQL. Reportez-vous à [Dump & Restore](concepts-migrate-dump-restore.md) (Vider et restaurer) pour savoir comment restaurer un fichier de vidage sur un serveur MySQL. Si le fichier de vidage est volumineux, transférez-le vers une machine virtuelle dans Azure au sein de la même région que votre serveur réplica. Restaurez-le sur le serveur Azure Database pour MySQL à partir de la machine virtuelle.

## <a name="link-source-and-replica-servers-to-start-data-in-replication"></a>Lier les serveurs source et réplica pour démarrer Réplication des données entrantes

1. Définir le serveur source

   Toutes les fonctions de réplication de données entrantes sont effectuées par des procédures stockées. Vous trouverez toutes les procédures dans [Data-in Replication Stored Procedures](./reference-stored-procedures.md) (Procédures stockées de réplication de données entrantes). Les procédures stockées peuvent être exécutées dans l’interpréteur de commandes MySQL ou MySQL Workbench. 

   Pour lier deux serveurs et démarrer une réplication, connectez-vous au serveur réplica cible dans le service Azure Database pour MySQL, et définissez l’instance externe en tant que serveur source. Pour ce faire, utilisez la procédure stockée `mysql.az_replication_change_master` sur le serveur Azure DB pour MySQL.

   ```sql
   CALL mysql.az_replication_change_master('<master_host>', '<master_user>', '<master_password>', 3306, '<master_log_file>', <master_log_pos>, '<master_ssl_ca>');
   ```

   - master_host : nom d’hôte du serveur source
   - master_user : nom d’utilisateur pour le serveur source
   - master_password : mot de passe pour le serveur source
   - master_log_file : nom de fichier du journal binaire à partir de l’exécution de `show master status`
   - master_log_pos : position du journal binaire à partir de l’exécution de `show master status`
   - master_ssl_ca : contexte du certificat d’autorité de certification. Si vous n’utilisez pas le protocole SSL, transmettez une chaîne vide.
       - Il est recommandé de transmettre ce paramètre en tant que variable. Pour plus d’informations, consultez les exemples suivants.

   > [!NOTE]
   > Si le serveur source est hébergé dans une machine virtuelle Azure, activez l’option « Autoriser l’accès aux services Azure » pour autoriser les serveurs sources et de réplica à communiquer entre eux. Ce paramètre peut être modifié dans les options de **sécurité de la connexion**. Pour plus d’informations, consultez la [gestion des règles de pare-feu à l’aide du portail](howto-manage-firewall-using-portal.md).
      
   **Exemples**
   
   *Réplication avec SSL*
   
   La variable `@cert` est créée en exécutant les commandes MySQL suivantes : 
   
      ```sql
      SET @cert = '-----BEGIN CERTIFICATE-----
      PLACE YOUR PUBLIC KEY CERTIFICATE'`S CONTEXT HERE
      -----END CERTIFICATE-----'
      ```
   
   La réplication avec SSL est définie entre un serveur source hébergé dans le domaine « companya.com » et un serveur réplica hébergé dans Azure Database pour MySQL. Cette procédure stockée est exécutée sur le réplica. 
   
      ```sql
      CALL mysql.az_replication_change_master('master.companya.com', 'syncuser', 'P@ssword!', 3306, 'mysql-bin.000002', 120, @cert);
      ```
   *Réplication sans SSL*
   
   La réplication sans SSL est définie entre un serveur source hébergé dans le domaine « companya.com » et un serveur réplica hébergé dans Azure Database pour MySQL. Cette procédure stockée est exécutée sur le réplica.
   
      ```sql
      CALL mysql.az_replication_change_master('master.companya.com', 'syncuser', 'P@ssword!', 3306, 'mysql-bin.000002', 120, '');
      ```

1. Filtrage 
 
   Si vous souhaitez ignorer la réplication de certaines tables à partir de votre serveur maître, mettez à jour le paramètre serveur `replicate_wild_ignore_table` sur votre serveur de réplication. Vous pouvez fournir plusieurs modèles de table à l'aide d'une liste séparée par des virgules.

   Consultez la [documentation MySQL](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-wild-ignore-table) pour en savoir plus sur ce paramètre. 
    
   Pour mettre à jour le paramètre, utilisez le [portail Azure](howto-server-parameters.md) ou [Azure CLI](howto-configure-server-parameters-using-cli.md).

1. Démarrer la réplication

   Appelez la procédure stockée `mysql.az_replication_start` pour amorcer la réplication.

   ```sql
   CALL mysql.az_replication_start;
   ```

1. Vérifier l’état de la réplication

   Appelez la commande [`show slave status`](https://dev.mysql.com/doc/refman/5.7/en/show-slave-status.html) sur le serveur réplica pour afficher l’état de réplication.
    
   ```sql
   show slave status;
   ```

   Si l’état de `Slave_IO_Running` et `Slave_SQL_Running` est « yes » et que la valeur de `Seconds_Behind_Master` est « 0 », cela signifie que la réplication fonctionne correctement. `Seconds_Behind_Master` indique que le retard du réplica. Si la valeur est différente de « 0 », cela signifie que le réplica traite les mises à jour. 

## <a name="other-stored-procedures"></a>Autres procédures stockées

### <a name="stop-replication"></a>Arrêter la réplication

Pour arrêter la réplication entre le serveur source et le serveur réplica, utilisez la procédure stockée suivante :

```sql
CALL mysql.az_replication_stop;
```

### <a name="remove-replication-relationship"></a>Supprimer la relation de réplication

Pour supprimer la relation entre le serveur source et le serveur réplica, utilisez la procédure stockée suivante :

```sql
CALL mysql.az_replication_remove_master;
```

### <a name="skip-replication-error"></a>Ignorer l’erreur de réplication

Pour ignorer une erreur de réplication et autoriser la poursuite de la réplication, utilisez la procédure stockée suivante :
    
```sql
CALL mysql.az_replication_skip_counter;
```

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur la [Réplication des données entrantes](concepts-data-in-replication.md) pour Azure Database pour MySQL.