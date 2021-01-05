---
title: 'Tutoriel : Effectuer la migration en ligne de PostgreSQL vers Azure Database pour PostgreSQL via Azure CLI'
titleSuffix: Azure Database Migration Service
description: Découvrez comment effectuer la migration en ligne d’une instance locale de PostgreSQL vers Azure Database pour PostgreSQL à l’aide d’Azure Database Migration Service via Azure CLI.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019, devx-track-azurecli
ms.topic: tutorial
ms.date: 04/11/2020
ms.openlocfilehash: fb303054a3026a305831309413c51c061a68c5d6
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97608049"
---
# <a name="tutorial-migrate-postgresql-to-azure-db-for-postgresql-online-using-dms-via-the-azure-cli"></a>Tutoriel : Effectuer la migration en ligne de PostgreSQL vers Azure DB pour PostgreSQL via Azure CLI à l’aide de DMS

Vous pouvez utiliser Azure Database Migration Service pour migrer les bases de données d’une instance PostgreSQL locale vers [Azure Database pour PostgreSQL](../postgresql/index.yml) avec un temps d’arrêt minimal. En d’autres termes, la migration peut être effectuée avec un temps d’arrêt minimal de l’application. Dans ce tutoriel, vous allez migrer l’exemple de base de données **DVD Rental** à partir d’une instance PostgreSQL 9.6 locale vers Azure Database pour PostgreSQL à l’aide de l’activité de migration en ligne dans Azure Database Migration Service.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
>
> * Migrez l’exemple de schéma à l’aide de l’utilitaire pg_dump.
> * Créer une instance Azure Database Migration Service.
> * Créer un projet de migration en utilisant Azure Database Migration Service.
> * Exécuter la migration.
> * Surveiller la migration.

> [!NOTE]
> L’utilisation d’Azure Database Migration Service pour effectuer une migration en ligne nécessite la création d’une instance basée sur le niveau tarifaire Premium. Nous chiffrons le disque pour empêcher le vol des données pendant le processus de migration.

> [!IMPORTANT]
> Pour une expérience de migration optimale, Microsoft recommande de créer une instance Azure Database Migration Service dans la même région Azure que la base de données cible. Le déplacement des données entre les régions ou les zones géographiques peut ralentir le processus de migration et introduire des erreurs.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez effectuer les opérations suivantes :

* Téléchargez et installez [PostgreSQL Community Edition](https://www.postgresql.org/download/) 9.5, 9.6 ou 10. La version du serveur PostgreSQL source doit être 9.5.11, 9.6.7, 10 ou une version ultérieure. Pour plus d’informations, consultez l’article [Versions de bases de données PostgreSQL prises en charge](../postgresql/concepts-supported-versions.md).

    Notez également que la version cible d'Azure Database pour PostgreSQL doit être égale ou ultérieure à la version PostgreSQL locale. Par exemple, PostgreSQL 9.6 peut migrer uniquement vers Azure Database pour PostgreSQL 9.6, 10 ou 11, et non vers Azure Database pour PostgreSQL 9.5.

* [Créez une instance dans Azure Database pour PostgreSQL](../postgresql/quickstart-create-server-database-portal.md) ou [créez un serveur Azure Database pour PostgreSQL - Hyperscale (Citus)](../postgresql/quickstart-create-hyperscale-portal.md).
* Créez un Réseau virtuel Microsoft Azure pour Azure Database Migration Service à l’aide du modèle de déploiement Azure Resource Manager, qui fournit une connectivité site à site à vos serveurs sources locaux via [ExpressRoute](../expressroute/expressroute-introduction.md) ou un [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Pour plus d’informations sur la création d’un réseau virtuel, consultez la [documentation sur le réseau virtuel](../virtual-network/index.yml), en particulier les articles sur le démarrage rapide, qui fournissent des informations pas à pas.

    > [!NOTE]
    > Pendant la configuration du réseau virtuel, si vous utilisez ExpressRoute avec le peering réseau à Microsoft, ajoutez ces [points de terminaison](../virtual-network/virtual-network-service-endpoints-overview.md) au sous-réseau où doit être provisionné le service :
    >
    > * Point de terminaison de base de données cible (un point de terminaison SQL ou Cosmos DB, par exemple)
    > * Point de terminaison de stockage
    > * Point de terminaison Service Bus
    >
    > Cette configuration est nécessaire car Azure Database Migration Service ne dispose pas d’une connectivité Internet.

* Vérifiez que les règles du groupe de sécurité réseau de votre réseau virtuel ne bloquent pas les ports suivants pour les communications entrantes vers Azure Database Migration Service : 443, 53, 9354, 445, 12000. Pour plus d’informations sur le filtrage du trafic de groupe de sécurité réseau de réseau virtuel, consultez l’article [Filtrer le trafic avec les groupes de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Configurez votre [pare-feu Windows pour accéder au moteur de base de données](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Ouvrez votre pare-feu Windows pour permettre à Azure Database Migration Service d’accéder au serveur PostgreSQL source, par défaut le port TCP 5432.
* Lorsque vous utilisez une appliance de pare-feu devant vos bases de données sources, vous devrez peut-être ajouter des règles de pare-feu pour permettre à Azure Database Migration Service d’accéder aux bases de données sources pour la migration.
* Créez une [règle de pare-feu](../azure-sql/database/firewall-configure.md) de niveau serveur pour Azure Database pour PostgreSQL afin de permettre à Azure Database Migration Service d’accéder aux bases de données cibles. Fournissez la plage de sous-réseau du réseau virtuel utilisé pour Azure Database Migration Service.
* Il existe deux méthodes pour appeler l’interface CLI :

  * Dans l’angle supérieur droit du portail Azure, sélectionnez le bouton Cloud Shell :

       ![Bouton Cloud Shell du portail Azure](media/tutorial-postgresql-to-azure-postgresql-online/cloud-shell-button.png)

  * Installez et exécutez l’interface CLI localement. CLI 2.0 est l’outil en ligne de commande pour la gestion des ressources Azure.

       Pour télécharger l’interface CLI, suivez les instructions fournies dans l’article [Installer Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest). Cet article répertorie également les plateformes qui prennent en charge CLI 2.0.

       Pour configurer Sous-système Windows pour Linux (WSL), suivez les instructions du [Guide d’installation de Windows 10](/windows/wsl/install-win10)

* Activez la réplication logique dans le fichier postgresql.config, puis définissez les paramètres suivants :

  * wal_level = **logical**
  * max_replication_slots = [nombre d’emplacements]. Valeur recommandée : **cinq emplacements**
  * max_wal_senders = [nombre de tâches simultanées] - le paramètre max_wal_senders définit le nombre de tâches simultanées qui peuvent s’exécuter, paramètre recommandé : **10 tâches**

## <a name="migrate-the-sample-schema"></a>Migrer l’exemple de schéma

Pour compléter tous les objets de base de données tels que les schémas de table, les index et les procédures stockées, nous devons extraire le schéma à partir de la base de données source et l’appliquer à la base de données.

1. Utilisez la commande pg_dump -s afin de créer un fichier de vidage de schéma pour une base de données. 

    ```
    pg_dump -o -h hostname -U db_username -d db_name -s > your_schema.sql
    ```

    Par exemple, pour vider un fichier de schéma d’une base de données dvdrental :
    ```
    pg_dump -o -h localhost -U postgres -d dvdrental -s  > dvdrentalSchema.sql
    ```

    Pour plus d’informations sur l’utilisation de l’utilitaire pg_dump, consultez les exemples du tutoriel [pg-dump](https://www.postgresql.org/docs/9.6/static/app-pgdump.html#PG-DUMP-EXAMPLES).

2. Créez une base de données vide dans votre environnement cible, c’est-à-dire Azure Database pour PostgreSQL.

    Pour plus d’informations sur la création et la connexion d’une base de données, consultez [Créer un serveur Azure Database pour PostgreSQL dans le portail Azure](../postgresql/quickstart-create-server-database-portal.md) ou [Créer un serveur Azure Database pour PostgreSQL - Hyperscale (Citus) dans le portail Azure](../postgresql/quickstart-create-hyperscale-portal.md).

3. Importez le schéma dans la base de données cible que vous avez créée en restaurant le fichier de vidage du schéma.

    ```
    psql -h hostname -U db_username -d db_name < your_schema.sql 
    ```

    Par exemple :

    ```
    psql -h mypgserver-20170401.postgres.database.azure.com  -U postgres -d dvdrental < dvdrentalSchema.sql
    ```

4. Si vous avez des clés étrangères dans votre schéma, la charge initiale et la synchronisation continue de la migration échouent. Exécutez le script suivant dans PgAdmin ou dans psql pour extraire le script de clé étrangère Drop et ajouter un script de clé étrangère à la destination (Azure Database pour PostgreSQL).
  
    ```
    SELECT Queries.tablename
           ,concat('alter table ', Queries.tablename, ' ', STRING_AGG(concat('DROP CONSTRAINT ', Queries.foreignkey), ',')) as DropQuery
                ,concat('alter table ', Queries.tablename, ' ',
                                                STRING_AGG(concat('ADD CONSTRAINT ', Queries.foreignkey, ' FOREIGN KEY (', column_name, ')', 'REFERENCES ', foreign_table_name, '(', foreign_column_name, ')' ), ',')) as AddQuery
        FROM
        (SELECT
        tc.table_schema,
        tc.constraint_name as foreignkey,
        tc.table_name as tableName,
        kcu.column_name,
        ccu.table_schema AS foreign_table_schema,
        ccu.table_name AS foreign_table_name,
        ccu.column_name AS foreign_column_name
    FROM
        information_schema.table_constraints AS tc
        JOIN information_schema.key_column_usage AS kcu
          ON tc.constraint_name = kcu.constraint_name
          AND tc.table_schema = kcu.table_schema
        JOIN information_schema.constraint_column_usage AS ccu
          ON ccu.constraint_name = tc.constraint_name
          AND ccu.table_schema = tc.table_schema
    WHERE constraint_type = 'FOREIGN KEY') Queries
      GROUP BY Queries.tablename;
    ```

    Exécutez la clé étrangère Drop (deuxième colonne) dans le résultat de la requête.

5. Des déclencheurs dans les données (déclencheur d’insertion ou de mise à jour) appliqueront l’intégrité des données dans la cible avant les données répliquées à partir de la source. Il est recommandé de désactiver les déclencheurs de toutes les tables **au niveau de la cible** pendant la migration, puis de réactiver les déclencheurs une fois celle-ci terminée.

    Pour désactiver les déclencheurs dans la base de données cible, utilisez la commande suivante :

    ```
    select concat ('alter table ', event_object_table, ' disable trigger ', trigger_name)
    from information_schema.triggers;
    ```

6. S’il existe un type de données ENUM dans des tables, il est recommandé de le mettre à jour temporairement en type de données « character varying » dans la table cible. À la fin de la réplication des données, rétablissez le type de données sur ENUM.

## <a name="provisioning-an-instance-of-dms-using-the-cli"></a>Approvisionnement d’une instance de DMS à l’aide de l’interface CLI

1. Installez l’extension de synchronisation dms :
   * Connectez-vous à Azure en exécutant la commande suivante :
       ```azurecli
       az login
       ```

   * Lorsque vous y êtes invité, ouvrez un navigateur web et entrez un code pour authentifier votre appareil. Suivez les instructions indiquées.
   * Ajoutez l’extension dms :
       * Pour répertorier les extensions disponibles, exécutez la commande suivante :

           ```azurecli
           az extension list-available –otable
           ```

       * Pour installer l’extension, exécutez la commande suivante :

           ```azurecli
           az extension add –n dms-preview
           ```

   * Pour vérifier que l’extension dms est correctement installée, exécutez la commande suivante :

       ```azurecli
       az extension list -otable
       ```
       Vous devez normalement voir la sortie suivante.

       ```output
       ExtensionType    Name
       ---------------  ------
       whl              dms
       ```

      > [!IMPORTANT]
      > Vérifiez que la version de votre extension est ultérieure à la version 0.11.0.

   * À tout moment, affichez toutes les commandes prises en charge dans DMS en exécutant :

       ```azurecli
       az dms -h
       ```

   * Si vous avez plusieurs abonnements Azure, exécutez la commande suivante pour sélectionner l’abonnement avec lequel vous souhaitez travailler.

        ```azurecli
       az account set -s 97181df2-909d-420b-ab93-1bff15acb6b7
        ```

2. Approvisionnez une instance DMS en exécutant la commande suivante :

   ```azurecli
   az dms create -l [location] -n <newServiceName> -g <yourResourceGroupName> --sku-name Premium_4vCores --subnet/subscriptions/{vnet subscription id}/resourceGroups/{vnet resource group}/providers/Microsoft.Network/virtualNetworks/{vnet name}/subnets/{subnet name} –tags tagName1=tagValue1 tagWithNoValue
   ```

   Par exemple, la commande suivante crée un service dans :

   * Localisation : USA Est 2
   * Abonnement : 97181df2-909d-420b-ab93-1bff15acb6b7
   * Nom du groupe de ressources : PostgresDemo
   * Nom du service DMS : PostgresCLI

   ```azurecli
   az dms create -l eastus2 -g PostgresDemo -n PostgresCLI --subnet /subscriptions/97181df2-909d-420b-ab93-1bff15acb6b7/resourceGroups/ERNetwork/providers/Microsoft.Network/virtualNetworks/AzureDMS-CORP-USC-VNET-5044/subnets/Subnet-1 --sku-name Premium_4vCores
   ```

   La création de l’instance du service DMS prend 10 à 12 minutes.

3. Pour identifier l’adresse IP de l’agent DMS afin de l’ajouter au fichier pg_hba.conf Postgres, exécutez la commande suivante :

    ```azurecli
    az network nic list -g <ResourceGroupName>--query '[].ipConfigurations | [].privateIpAddress'
    ```

    Par exemple :

    ```azurecli
    az network nic list -g PostgresDemo --query '[].ipConfigurations | [].privateIpAddress'
    ```

    Vous devriez voir un résultat similaire à celui de l’écran suivant : 

    ```output
    [
      "172.16.136.18"
    ]
    ```

4. Ajoutez l’adresse IP de l’agent DMS au fichier Postgres pg_hba.conf.

    * Notez l’adresse IP DMS après avoir terminé l’approvisionnement dans DMS.
    * Ajoutez l’adresse IP au fichier pg_hba.conf sur la source, comme dans l’entrée suivante :

        ```
        host     all            all        172.16.136.18/10    md5
        host     replication    postgres   172.16.136.18/10    md5
        ```

5. Ensuite, créez un projet de migration PostgreSQL en exécutant la commande suivante :
    
    ```azurecli
    az dms project create -l <location> -g <ResourceGroupName> --service-name <yourServiceName> --source-platform PostgreSQL --target-platform AzureDbforPostgreSQL -n <newProjectName>
    ```

    Par exemple, la commande suivante crée un projet à l’aide de ces paramètres :

   * Localisation : Centre-USA Ouest
   * Nom du groupe de ressources : PostgresDemo
   * Nom du service : PostgresCLI
   * Nom du projet : PGMigration
   * Plateforme source : PostgreSQL
   * Plateforme cible : AzureDbForPostgreSql

     ```azurecli
     az dms project create -l westcentralus -n PGMigration -g PostgresDemo --service-name PostgresCLI --source-platform PostgreSQL --target-platform AzureDbForPostgreSql
     ```

6. Créez une tâche de migration PostgreSQL en procédant comme suit.

    Cette étape inclut l’utilisation de l’adresse IP source, de l’ID utilisateur et du mot de passe, de l’adresse IP de destination, de l’ID utilisateur, du mot de passe et du type de tâche pour établir la connectivité.

   * Pour afficher une liste complète des options, exécutez la commande :

       ```azurecli
       az dms project task create -h
       ```

       Pour les connexions source et cible, le paramètre d’entrée fait référence à un fichier json qui contient la liste des objets.

       Le format de l’objet JSON de connexion pour les connexions PostgreSQL.
        
       ```json
       {
                   "userName": "user name",    // if this is missing or null, you will be prompted
                   "password": null,           // if this is missing or null (highly recommended) you will
               be prompted
                   "serverName": "server name",
                   "databaseName": "database name", // if this is missing, it will default to the 'postgres'
               server
                   "port": 5432                // if this is missing, it will default to 5432
               }
       ```

   * Il existe également un fichier json des options de base de données, qui répertorie les objets json. Pour PostgreSQL, le format de l’objet JSON des options de base de données est indiqué ci-dessous :

       ```json
       [
           {
               "name": "source database",
               "target_database_name": "target database",
           },
           ...n
       ]
       ```

   * Créez un fichier json à l’aide du Bloc-notes, copiez les commandes suivantes, collez-les dans le fichier, puis enregistrez le fichier sous C:\DMS\source.json.

        ```json
       {
                   "userName": "postgres",    
                   "password": null,           
               be prompted
                   "serverName": "13.51.14.222",
                   "databaseName": "dvdrental", 
                   "port": 5432                
               }
        ```

   * Créez un autre fichier nommé target.json et enregistrez-le sous C:\DMS\target.json. Incluez les commandes suivantes :

       ```json
       {
               "userName": " dms@builddemotarget",    
               "password": null,           
               "serverName": " builddemotarget.postgres.database.azure.com",
               "databaseName": "inventory", 
               "port": 5432                
           }
       ```

   * Créez un fichier json d’options de base de données qui répertorie un inventaire de la base de données à migrer :

       ```json
       [
           {
               "name": "dvdrental",
               "target_database_name": "dvdrental",
           }
       ]
       ```

   * Exécutez la commande suivante, qui accepte la source, la destination et les fichiers json d’options de base de données.

       ```azurecli
       az dms project task create -g PostgresDemo --project-name PGMigration --source-platform postgresql --target-platform azuredbforpostgresql --source-connection-json c:\DMS\source.json --database-options-json C:\DMS\option.json --service-name PostgresCLI --target-connection-json c:\DMS\target.json –task-type OnlineMigration -n runnowtask    
       ```

     À ce stade, vous avez correctement envoyé une tâche de migration.

7. Pour afficher la progression de la tâche, exécutez la commande suivante :

   ```azurecli
   az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
   ```

   OR

    ```azurecli
   az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask --expand output
    ```

8. Vous pouvez également interroger le paramètre migrationState à partir de la sortie développée :

    ```azurecli
    az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask --expand output --query 'properties.output[].migrationState | [0]' "READY_TO_COMPLETE"
    ```

Dans le fichier de sortie, plusieurs paramètres indiquent la progression de la migration. Par exemple, consultez le fichier de sortie ci-dessous :

  ```output
    "output": [                                 // Database Level
          {
            "appliedChanges": 0,         // Total incremental sync applied after full load
            "cdcDeleteCounter": 0        // Total delete operation  applied after full load
            "cdcInsertCounter": 0,       // Total insert operation applied after full load
            "cdcUpdateCounter": 0,       // Total update operation applied after full load
            "databaseName": "inventory",
            "endedOn": null,
            "fullLoadCompletedTables": 2,   //Number of tables completed full load
            "fullLoadErroredTables": 0,     //Number of tables that contain migration error
            "fullLoadLoadingTables": 0,     //Number of tables that are in loading status
            "fullLoadQueuedTables": 0,      //Number of tables that are in queued status
            "id": "db|inventory",
            "incomingChanges": 0,           //Number of changes after full load
            "initializationCompleted": true,
            "latency": 0,
            "migrationState": "READY_TO_COMPLETE",    //Status of migration task. READY_TO_COMPLETE means the database is ready for cutover
            "resultType": "DatabaseLevelOutput",
            "startedOn": "2018-07-05T23:36:02.27839+00:00"
          },
          {
            "databaseCount": 1,
            "endedOn": null,
            "id": "dd27aa3a-ed71-4bff-ab34-77db4261101c",
            "resultType": "MigrationLevelOutput",
            "sourceServer": "138.91.123.10",
            "sourceVersion": "PostgreSQL",
            "startedOn": "2018-07-05T23:36:02.27839+00:00",
            "state": "PENDING",
            "targetServer": "builddemotarget.postgres.database.azure.com",
            "targetVersion": "Azure Database for PostgreSQL"
          },
          {                                        // Table 1
            "cdcDeleteCounter": 0,
            "cdcInsertCounter": 0,
            "cdcUpdateCounter": 0,
            "dataErrorsCount": 0,
            "databaseName": "inventory",
            "fullLoadEndedOn": "2018-07-05T23:36:20.740701+00:00",    //Full load completed time
            "fullLoadEstFinishTime": "1970-01-01T00:00:00+00:00",
            "fullLoadStartedOn": "2018-07-05T23:36:15.864552+00:00",    //Full load started time
            "fullLoadTotalRows": 10,                     //Number of rows loaded in full load
            "fullLoadTotalVolumeBytes": 7056,            //Volume in Bytes in full load
            "id": "or|inventory|public|actor",
            "lastModifiedTime": "2018-07-05T23:36:16.880174+00:00",
            "resultType": "TableLevelOutput",
            "state": "COMPLETED",                       //State of migration for this table
            "tableName": "public.catalog",              //Table name
            "totalChangesApplied": 0                    //Total sync changes that applied after full load
          },
          {                                            //Table 2
            "cdcDeleteCounter": 0,
            "cdcInsertCounter": 50,
            "cdcUpdateCounter": 0,
            "dataErrorsCount": 0,
            "databaseName": "inventory",
            "fullLoadEndedOn": "2018-07-05T23:36:23.963138+00:00",
            "fullLoadEstFinishTime": "1970-01-01T00:00:00+00:00",
            "fullLoadStartedOn": "2018-07-05T23:36:19.302013+00:00",
            "fullLoadTotalRows": 112,
            "fullLoadTotalVolumeBytes": 46592,
            "id": "or|inventory|public|address",
            "lastModifiedTime": "2018-07-05T23:36:20.308646+00:00",
            "resultType": "TableLevelOutput",
            "state": "COMPLETED",
            "tableName": "public.orders",
            "totalChangesApplied": 0
          }
        ],                                      // DMS migration task state
        "state": "Running",    //Migration task state – Running means it is still listening to any changes that might come in
        "taskType": null
      },
      "resourceGroup": "PostgresDemo",
      "type": "Microsoft.DataMigration/services/projects/tasks"
  ```

## <a name="cutover-migration-task"></a>Tâche de migration de basculement

La base de données est prête pour le basculement lorsque le chargement est terminé. Selon la charge du serveur source qui traite les nouvelles transactions entrantes, la tâche DMS applique peut-être toujours les modifications après que le chargement complet.

Pour garantir la récupération de toutes les données, validez le nombre de lignes entre les bases de données source et cible. Par exemple, vous pouvez utiliser la commande suivante :

```
"migrationState": "READY_TO_COMPLETE", //Status of migration task. READY_TO_COMPLETE means database is ready for cutover
 "incomingChanges": 0, //continue to check for a period of 5-10 minutes to make sure no new incoming changes that need to be applied to the target server
   "fullLoadTotalRows": 10, //full load for table 1
    "cdcDeleteCounter": 0, //delete, insert and update counter on incremental sync after full load
    "cdcInsertCounter": 50,
    "cdcUpdateCounter": 0,
     "fullLoadTotalRows": 112, //full load for table 2
```

1. Effectuez la tâche de migration de base de données de basculement à l’aide de la commande suivante :

    ```azurecli
    az dms project task cutover -h
    ```

    Par exemple :

    ```azurecli
    az dms project task cutover --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask  --object-name Inventory
    ```

2. Pour surveiller la progression du basculement, exécutez la commande suivante :

    ```azurecli
    az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
    ```
3. Lorsque l’état de migration de la base de données indique **Terminé**, [recréez des séquences](https://wiki.postgresql.org/wiki/Fixing_Sequences) (le cas échéant) et connectez vos applications à la nouvelle instance cible d’Azure Database pour PostgreSQL.

## <a name="service-project-task-cleanup"></a>Nettoyage d’un service, d’un projet ou d’une tâche

Si vous avez besoin d’annuler ou de supprimer une tâche, un projet ou un service DMS, effectuez l’annulation dans selon la séquence suivante :

* Annuler toutes les tâches en cours d'exécution
* Supprimez la tâche
* Supprimez le projet
* Créez un service DMS

1. Pour annuler une tâche en cours d’exécution, utilisez la commande suivante :

    ```azurecli
    az dms project task cancel --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
     ```

2. Pour supprimer une tâche en cours d’exécution, utilisez la commande suivante :
    ```azurecli
    az dms project task delete --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
    ```

3. Pour annuler un projet en cours d’exécution, utilisez la commande suivante :
     ```azurecli
    az dms project task cancel -n runnowtask --project-name PGMigration -g PostgresDemo --service-name PostgresCLI
     ```

4. Pour supprimer un projet en cours d’exécution, utilisez la commande suivante :
    ```azurecli
    az dms project task delete -n runnowtask --project-name PGMigration -g PostgresDemo --service-name PostgresCLI
    ```

5. Pour supprimer un service DMS, utilisez la commande suivante :

     ```azurecli
    az dms delete -g ProgresDemo -n PostgresCLI
     ```

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les problèmes connus et les limitations lors de l’exécution de migrations en ligne vers Azure Database pour PostgreSQL, consultez l’article [Known issues and workarounds with Azure Database for PostgreSQL online migrations](known-issues-azure-postgresql-online.md) (Problèmes connus et solutions de contournement pour les migrations Azure Database pour PostgreSQL en ligne).
* Pour plus d’informations sur Azure Database Migration Service, consultez l’article [Qu’est-ce qu’Azure Database Migration Service ?](./dms-overview.md).
* Pour obtenir des informations sur Azure Database pour PostgreSQL, consultez l’article [Qu’est-ce qu’Azure Database pour PostgreSQL ?](../postgresql/overview.md).