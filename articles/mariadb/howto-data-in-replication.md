---
title: Configurer la réplication des données entrantes - Azure Database for MariaDB
description: Cet article décrit comment configurer la réplication des données entrantes dans Azure Database for MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 9/29/2020
ms.openlocfilehash: fe7e02cc34dc9c97e540d7b8d96c48ee8d5cfe09
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94535365"
---
# <a name="configure-data-in-replication-in-azure-database-for-mariadb"></a>Configurer la réplication des données entrantes dans Azure Database for MariaDB

Cet article décrit comment configurer [Réplication des données entrantes](concepts-data-in-replication.md) dans Azure Database for MariaDB en configurant les serveurs source et réplica. Cet article suppose que vous ayez déjà utilisé des serveurs et des bases de données MariaDB.

Pour créer un réplica dans Azure Database for MariaDB, [Réplication des données entrantes](concepts-data-in-replication.md) synchronise les données provenant d’un serveur MariaDB source qui s’exécute en local dans des machines virtuelles ou des services de base de données cloud. La réplication des données entrantes est basée sur la réplication selon la position du fichier journal binaire (binlog) native à MariaDB. Pour découvrir plus en détail la réplication binlog, consultez la [vue d’ensemble de la réplication binlog](https://mariadb.com/kb/en/library/replication-overview/).

Passez en revue les [limitations et conditions requises](concepts-data-in-replication.md#limitations-and-considerations) de la Réplication des données entrantes avant de suivre les étapes décrites dans cet article.

> [!NOTE]
> Si votre serveur source correspond à la version 10.2 ou ultérieure, nous vous recommandons de définir Réplication des données entrantes à l’aide de l’[ID de transaction global](https://mariadb.com/kb/en/library/gtid/).


## <a name="create-a-mariadb-server-to-use-as-a-replica"></a>Créer un serveur MariaDB à utiliser en tant que réplica

1. Créez un nouveau serveur Azure Database for MariaDB (par exemple, replica.mariadb.database.azure.com). Le serveur correspond au serveur réplica dans la réplication de données entrantes.

    Pour plus d'informations sur la création d'un serveur, consultez [Créer un serveur Azure Database for MariaDB à l’aide du portail Azure](quickstart-create-mariadb-server-database-using-azure-portal.md).

   > [!IMPORTANT]
   > Vous devez créer le serveur Azure Database for MariaDB dans les niveaux tarifaires Usage général ou Mémoire optimisée.

2. Créez des comptes d’utilisateurs identiques et les privilèges correspondants.
    
    Les comptes d’utilisateur ne sont pas répliqués à partir du serveur source vers le serveur réplica. Pour fournir aux utilisateurs un accès au serveur réplica, vous devez créer manuellement tous les comptes et privilèges correspondants sur le nouveau serveur Azure Database for MariaDB.

3. Ajoutez l’adresse IP du serveur source aux règles de pare-feu du réplica. 

   Mettez à jour les règles de pare-feu à l’aide du [portail Azure](howto-manage-firewall-portal.md) ou d’[Azure CLI](howto-manage-firewall-cli.md).

> [!NOTE]
> Communication sans biais
>
> La diversité et l’inclusion sont au cœur des valeurs de Microsoft. Cet article contient des références au mot _esclave_. Le [guide de style de Microsoft sur la communication sans stéréotype](https://github.com/MicrosoftDocs/microsoft-style-guide/blob/master/styleguide/bias-free-communication.md) le reconnaît comme un mot à exclure. Le mot est utilisé dans cet article pour des raisons de cohérence, car il s’agit du mot qui figure dans le logiciel. Une fois que le mot aura été supprimé du logiciel, cet article sera mis à jour en conséquence.
>

## <a name="configure-the-source-server"></a>Configurer le serveur source

Les étapes suivantes préparent et configurent le serveur MariaDB hébergé localement, dans une machine virtuelle ou dans un service de base de données cloud pour la réplication des données entrantes. Le serveur MariaDB correspond à la source dans Réplication des données entrantes.

1. Avant de continuer, vérifiez la [configuration requise pour le serveur maître](concepts-data-in-replication.md#requirements). 

2. Assurez-vous que le serveur source autorise le trafic entrant et sortant sur le port 3306, que le serveur source dispose d’une **IP publique** et que le DNS est accessible publiquement ou possède un nom de domaine complet (FQDN). 
   
   Testez la connectivité au serveur source en tentant de vous connecter à partir d’un outil tel que la ligne de commande MySQL hébergée sur un autre ordinateur ou à partir d’[Azure Cloud Shell](../cloud-shell/overview.md) accessible sur le portail Azure.

   Si votre organisation a des stratégies de sécurité strictes et qu’elle n’autorise pas toutes les adresses IP sur le serveur source à permettre la communication d’Azure vers votre serveur source, vous pouvez éventuellement utiliser la commande ci-dessous pour déterminer l’adresse IP de votre serveur Azure Database for MariaDB.
    
   1. Connectez-vous à Azure Database for MariaDB à l’aide d’un outil tel que la ligne de commande MySQL.
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

2. Activez la journalisation binaire.
    
    Pour vérifier que la journalisation binaire est activée sur le serveur maître, entrez la commande suivante :

   ```sql
   SHOW VARIABLES LIKE 'log_bin';
   ```

   Si la variable [`log_bin`](https://mariadb.com/kb/en/library/replication-and-binary-log-server-system-variables/#log_bin) renvoie la valeur `ON`, la journalisation binaire est activée sur votre serveur.

   Si `log_bin` renvoie la valeur `OFF`, modifiez le fichier **my.cnf** pour permettre à `log_bin=ON` d'activer la journalisation binaire. Redémarrez le serveur pour que la modification prenne effet.

3. Configurez les paramètres du serveur source.

    Réplication des données entrantes nécessite que le paramètre `lower_case_table_names` soit cohérent entre les serveurs source et réplica. Par défaut, le paramètre `lower_case_table_names` est défini sur `1` dans Azure Database for MariaDB.

   ```sql
   SET GLOBAL lower_case_table_names = 1;
   ```

4. Créez un nouveau rôle de réplication et définir des autorisations.

   Créez un compte d’utilisateur sur le serveur source configuré avec des privilèges de réplication. Vous pouvez créer un compte à l’aide des commandes SQL ou MySQL Workbench. Si vous envisagez une réplication avec SSL, vous devez le spécifier lors de la création du compte d’utilisateur.
   
   Pour savoir comment ajouter des comptes d’utilisateurs sur votre serveur source, consultez la [documentation MariaDB](https://mariadb.com/kb/en/library/create-user/).

   Les commandes suivantes permettent au nouveau rôle de réplication d’accéder au serveur source à partir de n’importe quelle machine, et pas seulement de celle qui héberge le serveur source. Pour cet accès, spécifiez **syncuser\@'%'** dans la commande pour créer un utilisateur.
   
   Pour en savoir plus sur la documentation MariaDB, consultez la [définition des noms de compte](https://mariadb.com/kb/en/library/create-user/#account-names).

   **Commande SQL**

   - Réplication avec SSL

       Pour exiger le protocole SSL pour toutes les connexions utilisateur, entrez la commande suivante pour créer un utilisateur :

       ```sql
       CREATE USER 'syncuser'@'%' IDENTIFIED BY 'yourpassword';
       GRANT REPLICATION SLAVE ON *.* TO ' syncuser'@'%' REQUIRE SSL;
       ```

   - Réplication sans SSL

       Si le protocole SSL n’est pas requis pour toutes les connexions, entrez la commande suivante pour créer un utilisateur :
    
       ```sql
       CREATE USER 'syncuser'@'%' IDENTIFIED BY 'yourpassword';
       GRANT REPLICATION SLAVE ON *.* TO ' syncuser'@'%';
       ```

   **MySQL Workbench**

   Pour créer le rôle de réplication dans MySQL Workbench, dans le volet **Gestion**, sélectionnez **Utilisateurs et privilèges**. Sélectionnez ensuite **Ajouter un compte**.
 
   ![Utilisateurs et privilèges](./media/howto-data-in-replication/users_privileges.png)

   Entrez un nom d’utilisateur dans le champ **Nom de connexion**.

   ![Synchroniser l’utilisateur](./media/howto-data-in-replication/syncuser.png)
 
   Sélectionnez le panneau **Rôles administratifs**, puis dans la liste **Privilèges globaux**, sélectionnez **Subordonné de réplication**. Sélectionnez **Appliquer** pour créer le rôle de réplication.

   ![Subordonné de réplication](./media/howto-data-in-replication/replicationslave.png)


5. Définissez le serveur source en mode lecture seule.

   Avant de vider une base de données, le serveur doit être placé en mode lecture seule. En mode lecture seule, le serveur source ne peut traiter aucune transaction d’écriture. Pour éviter tout impact sur votre entreprise, planifiez la fenêtre en lecture seule lors d'une période creuse.

   ```sql
   FLUSH TABLES WITH READ LOCK;
   SET GLOBAL read_only = ON;
   ```

6. Obtenez le nom actuel du fichier du journal binaire et le décalage.

   Pour déterminer le nom de fichier du journal binaire actuel et le décalage, exécutez la commande [`show master status`](https://mariadb.com/kb/en/library/show-master-status/).
    
   ```sql
   show master status;
   ```
   Les résultats doivent être similaires au tableau suivant :
   
   ![Résultats de l’état récapitulatif](./media/howto-data-in-replication/masterstatus.png)

   Notez le nom du fichier binaire car il sera utilisé lors des étapes ultérieures.
   
7. Obtenez la position de l'identificateur GTID (facultatif, requis pour la réplication avec GTID).

   Exécutez la fonction [`BINLOG_GTID_POS`](https://mariadb.com/kb/en/library/binlog_gtid_pos/) pour obtenir la position de l'identificateur GTID pour le nom de fichier binlog correspondant et le décalage.
  
    ```sql
    select BINLOG_GTID_POS('<binlog file name>', <binlog offset>);
    ```
 

## <a name="dump-and-restore-the-source-server"></a>Vider et restaurer le serveur source

1. Videz toutes les bases de données du serveur source.

   Utilisez mysqldump pour vider toutes les bases de données du serveur source. Il n’est pas nécessaire de vider les bibliothèques MySQL et de test.

    Pour plus d’informations, consultez [Vider et restaurer](howto-migrate-dump-restore.md).

2. Définissez le serveur source en mode lecture/écriture.

   Une fois la base de données sauvegardée, remettez le serveur MariaDB source en mode de lecture/écriture.

   ```sql
   SET GLOBAL read_only = OFF;
   UNLOCK TABLES;
   ```

3. Restaurez le fichier de vidage sur le nouveau serveur.

   Restaurez le fichier de vidage sur le serveur créé dans le service Azure Database for MariaDB. Consultez [Sauvegarder et restaurer](howto-migrate-dump-restore.md) pour savoir comment restaurer un fichier de vidage sur un serveur MariaDB.

   Si le fichier de vidage est volumineux, transférez-le vers une machine virtuelle dans Azure au sein de la même région que votre serveur réplica. Restaurez-le sur le serveur Azure Database for MariaDB à partir de la machine virtuelle.

## <a name="link-the-source-and-replica-servers-to-start-data-in-replication"></a>Lier les serveurs source et réplica pour démarrer Réplication des données entrantes

1. Définissez le serveur source.

   Toutes les fonctions de réplication de données entrantes sont effectuées par des procédures stockées. Vous trouverez toutes les procédures dans [Data-in Replication Stored Procedures](reference-stored-procedures.md) (Procédures stockées de réplication de données entrantes). Les procédures stockées peuvent être exécutées dans l’interpréteur de commandes MySQL ou MySQL Workbench.

   Pour lier deux serveurs et démarrer une réplication, connectez-vous au serveur réplica cible dans le service Azure Database for MariaDB. Définissez ensuite l’instance externe en tant que serveur source à l’aide de la procédure stockée `mysql.az_replication_change_master` ou `mysql.az_replication_change_master_with_gtid` sur le serveur Azure Database for MariaDB.

   ```sql
   CALL mysql.az_replication_change_master('<master_host>', '<master_user>', '<master_password>', 3306, '<master_log_file>', <master_log_pos>, '<master_ssl_ca>');
   ```
   
   ou
   
   ```sql
   CALL mysql.az_replication_change_master_with_gtid('<master_host>', '<master_user>', '<master_password>', 3306, '<master_gtid_pos>', '<master_ssl_ca>');
   ```

   - master_host : nom d’hôte du serveur source
   - master_user : nom d’utilisateur pour le serveur source
   - master_password : mot de passe pour le serveur source
   - master_log_file : nom de fichier du journal binaire à partir de l’exécution de `show master status`
   - master_log_pos : position du journal binaire à partir de l’exécution de `show master status`
   - master_gtid_pos : position de l'identificateur GTID à partir de l'exécution de `select BINLOG_GTID_POS('<binlog file name>', <binlog offset>);`
   - master_ssl_ca : contexte du certificat d’autorité de certification. Si vous n’utilisez pas le protocole SSL, transmettez une chaîne vide.*
    
    
    *Nous vous recommandons de transmettre le paramètre master_ssl_ca en tant que variable. Pour plus d'informations, consultez les exemples suivants.

   **Exemples**

   - Réplication avec SSL

       Créez la variable `@cert` en exécutant les commandes suivantes :

       ```sql
       SET @cert = '-----BEGIN CERTIFICATE-----
       PLACE YOUR PUBLIC KEY CERTIFICATE\'S CONTEXT HERE
       -----END CERTIFICATE-----'
       ```

       La réplication avec SSL est définie entre un serveur source hébergé dans le domaine « companya.com » et un serveur réplica hébergé dans Azure Database for MariaDB. Cette procédure stockée est exécutée sur le réplica.
    
       ```sql
       CALL mysql.az_replication_change_master('master.companya.com', 'syncuser', 'P@ssword!', 3306, 'mariadb-bin.000016', 475, @cert);
       ```
   - Réplication sans SSL

       La réplication sans SSL est définie entre un serveur source hébergé dans le domaine « companya.com » et un serveur réplica hébergé dans Azure Database for MariaDB. Cette procédure stockée est exécutée sur le réplica.

       ```sql
       CALL mysql.az_replication_change_master('master.companya.com', 'syncuser', 'P@ssword!', 3306, 'mariadb-bin.000016', 475, '');
       ```

2. Démarrez la réplication.

   Appelez la procédure stockée `mysql.az_replication_start` pour commencer la réplication.

   ```sql
   CALL mysql.az_replication_start;
   ```

3. Vérifiez l’état de la réplication.

   Appelez la commande [`show slave status`](https://mariadb.com/kb/en/library/show-slave-status/) sur le serveur réplica pour afficher l’état de réplication.
    
   ```sql
   show slave status;
   ```

   Si l’état de `Slave_IO_Running` et `Slave_SQL_Running` est `yes` et la valeur de `Seconds_Behind_Master` est `0`, cela signifie que la réplication fonctionne. `Seconds_Behind_Master` indique que le retard du réplica. Si la valeur est différente de `0`, cela signifie que le réplica traite les mises à jour.

4. Mettez à jour les variables du serveur correspondant pour renforcer la sécurité de la réplication des données entrantes (uniquement requis pour la réplication sans GTID).
    
    En raison d’une limitation de la réplication native dans MariaDB, vous devez définir les variables [`sync_master_info`](https://mariadb.com/kb/en/library/replication-and-binary-log-system-variables/#sync_master_info) et [`sync_relay_log_info`](https://mariadb.com/kb/en/library/replication-and-binary-log-system-variables/#sync_relay_log_info) sur la réplication sans scénario GTID.

    Vérifiez les variables `sync_master_info` et `sync_relay_log_info` de votre serveur esclave pour vous assurer que la réplication des données entrantes est stable, et définissez les variables sur `1`.
    
## <a name="other-stored-procedures"></a>Autres procédures stockées

### <a name="stop-replication"></a>Arrêter la réplication

Pour arrêter la réplication entre le serveur source et le serveur réplica, utilisez la procédure stockée suivante :

```sql
CALL mysql.az_replication_stop;
```

### <a name="remove-the-replication-relationship"></a>Supprimer la relation de réplication

Pour supprimer la relation entre le serveur source et le serveur réplica, utilisez la procédure stockée suivante :

```sql
CALL mysql.az_replication_remove_master;
```

### <a name="skip-the-replication-error"></a>Ignorer l'erreur de réplication

Pour ignorer une erreur de réplication et autoriser la réplication, utilisez la procédure stockée suivante :
    
```sql
CALL mysql.az_replication_skip_counter;
```

## <a name="next-steps"></a>Étapes suivantes
Découvrez plus en détail la [réplication des données entrantes](concepts-data-in-replication.md) pour Azure Database for MariaDB.
