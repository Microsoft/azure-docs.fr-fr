---
title: 'Tutoriel : Migrer les services Bureau à distance PostgreSQL en ligne vers Azure Database pour PostgreSQL'
titleSuffix: Azure Database Migration Service
description: Découvrez comment effectuer une migration en ligne de RDS PostgreSQL vers Azure Database pour PostgreSQL à l’aide d’Azure Database Migration Service.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 04/11/2020
ms.openlocfilehash: 3bd79dc4935f5dfeb65d80ada544139dc88e129c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021433"
---
# <a name="tutorial-migrate-rds-postgresql-to-azure-db-for-postgresql-online-using-dms"></a>Tutoriel : Migrer Services Bureau à distance PostgreSQL vers Azure DB pour PostgreSQL en ligne à l’aide de DMS

Vous pouvez utiliser Azure Database Migration Service pour migrer des bases de données à partir d’une instance RDS PostgreSQL vers [Azure Database pour PostgreSQL](../postgresql/index.yml) en maintenant en ligne la base de données source pendant la migration. En d’autres termes, la migration peut être effectuée avec un temps d’arrêt minimal de l’application. Dans ce tutoriel, vous allez migrer l’exemple de base de données **DVD Rental** à partir d’une instance RDS PostgreSQL 9.6 vers Azure Database pour PostgreSQL à l’aide de l’activité de migration en ligne dans Azure Database Migration Service.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
>
> * Migrer l’exemple de schéma à l’aide de l’utilitaire pg_dump.
> * Créer une instance Azure Database Migration Service.
> * Créer un projet de migration en utilisant Azure Database Migration Service.
> * Exécuter la migration.
> * Surveiller la migration.
> * Effectuer le basculement de la migration.

> [!NOTE]
> Effectuer une migration en ligne à l’aide d’Azure Database Migration Service nécessite la création d’une instance basée sur le niveau tarifaire Premium. Pour plus d’informations, consultez la page de [tarification](https://azure.microsoft.com/pricing/details/database-migration/) du service Azure Database Migration Service. Nous chiffrons le disque pour empêcher le vol de données pendant le processus de migration.

> [!IMPORTANT]
> Pour une expérience de migration optimale, Microsoft vous recommande de créer une instance Azure Database Migration Service dans la même région Azure que la base de données cible. Le déplacement des données entre les régions ou les zones géographiques peut ralentir le processus de migration et introduire des erreurs.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

Cet article décrit comment effectuer une migration en ligne d’une instance locale de PostgreSQL vers Azure Database pour PostgreSQL.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez effectuer les opérations suivantes :

* Téléchargez et installez [PostgreSQL Community Edition](https://www.postgresql.org/download/) 9.5, 9.6 ou 10. La version du serveur PostgreSQL source doit être 9.5.11, 9.6.7, 10 ou une version ultérieure. Pour plus d’informations, consultez l’article [Versions de bases de données PostgreSQL prises en charge](../postgresql/concepts-supported-versions.md).

   Notez également que la version cible d'Azure Database pour PostgreSQL doit être égale ou ultérieure à la version PostgreSQL RDS. Par exemple, PostgreSQL RDS 9.6 peut migrer uniquement vers Azure Database pour PostgreSQL 9.6, 10 ou 11, et non vers Azure Database pour PostgreSQL 9.5.

* Créez une instance [Azure Database pour PostgreSQL](../postgresql/quickstart-create-server-database-portal.md) ou [Azure Database pour PostgreSQL – Hyperscale (Citus)](../postgresql/quickstart-create-hyperscale-portal.md). Reportez-vous à cette [section](../postgresql/quickstart-create-server-database-portal.md#connect-to-the-server-with-psql) du document pour obtenir des détails sur la façon de se connecter au serveur PostgreSQL avec pgAdmin.
* Créez un réseau virtuel Microsoft Azure pour Azure Database Migration Service à l’aide du modèle de déploiement Azure Resource Manager, qui fournit une connectivité site à site à vos serveurs sources locaux via [ExpressRoute](../expressroute/expressroute-introduction.md) ou un [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Pour plus d’informations sur la création d’un réseau virtuel, consultez la [documentation sur le réseau virtuel](../virtual-network/index.yml), en particulier les articles sur le démarrage rapide, qui fournissent des informations pas à pas.
* Vérifiez que les règles du groupe de sécurité réseau de votre réseau virtuel ne bloquent pas les ports suivants pour les communications entrantes vers Azure Database Migration Service : 443, 53, 9354, 445 et 12000. Pour plus d’informations sur le filtrage du trafic de groupe de sécurité réseau de réseau virtuel, consultez l’article [Filtrer le trafic avec les groupes de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Configurez votre [pare-feu Windows pour accéder au moteur de base de données](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Ouvrez votre pare-feu Windows pour permettre à Azure Database Migration Service d’accéder au serveur PostgreSQL source, par défaut le port TCP 5432.
* Lorsque vous utilisez une appliance de pare-feu devant vos bases de données sources, vous devrez peut-être ajouter des règles de pare-feu pour permettre à Azure Database Migration Service d’accéder aux bases de données sources pour la migration.
* Créez une [règle de pare-feu](../azure-sql/database/firewall-configure.md) de niveau serveur pour le serveur Azure Database pour PostgreSQL afin de permettre à Azure Database Migration Service d’accéder aux bases de données cibles. Fournissez la plage de sous-réseau du réseau virtuel utilisé pour Azure Database Migration Service.

### <a name="set-up-aws-rds-postgresql-for-replication"></a>Configurer AWS RDS PostgreSQL pour la réplication

1. Pour créer un groupe de paramètres, suivez les instructions fournies par AWS dans l’article [Utilisation de groupes de paramètres de base de données](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.html).
2. Utilisez le nom d’utilisateur principal pour vous connecter à la source à partir d’Azure Database Migration Service. Si vous utilisez un compte différent du compte d’utilisateur principal, ce compte doit avoir les rôles rds_superuser et rds_replication. Le rôle rds_replication accorde des autorisations pour gérer les emplacements logiques et pour diffuser des données à l’aide d’emplacements logiques.
3. Créez un groupe de paramètres avec la configuration suivante :

    a. Définissez le paramètre rds.logical_replication dans votre groupe de paramètres de base de données sur 1.

    b. max_wal_senders = [nombre de tâches simultanées] – Le paramètre max_wal_senders définit le nombre de tâches simultanées qui peuvent s’exécuter. Valeur recommandée : 10 tâches.

    c. max_replication_slots = [nombre d’emplacements]. Valeur recommandée : cinq emplacements.

4. Associez le groupe de paramètres que vous avez créé à l’instance RDS PostgreSQL.

## <a name="migrate-the-schema"></a>Migrer le schéma

1. Extrayez le schéma de la base de données source et appliquez-le à la base de données cible pour accomplir la migration de tous les objets de base de données, tels que les schémas de table, les index et les procédures stockées.

    La manière la plus simple de migrer uniquement le schéma consiste à utiliser pg_dump avec l’option -s. Pour plus d’informations, consultez les [exemples](https://www.postgresql.org/docs/9.6/app-pgdump.html#PG-DUMP-EXAMPLES) dans le tutoriel relatif à Postgres pg_dump.

    ```
    pg_dump -o -h hostname -U db_username -d db_name -s > your_schema.sql
    ```

    Par exemple, pour vider un fichier de schéma pour la base de données **dvdrental**, utilisez la commande suivante :

    ```
    pg_dump -o -h localhost -U postgres -d dvdrental -s  > dvdrentalSchema.sql
    ```

2. Créez une base de données vide dans le service cible, c’est-à-dire Azure Database pour PostgreSQL. Pour vous connecter et créer une base de données, consultez l’un des articles suivants :

    * [Créer un serveur Azure Database pour PostgreSQL à l’aide du portail Azure](../postgresql/quickstart-create-server-database-portal.md)
    * [Créer un serveur Azure Database pour PostgreSQL – Hyperscale (Citus) à l’aide du Portail Azure](../postgresql/quickstart-create-hyperscale-portal.md)

3. Importez le schéma dans le service cible, à savoir Azure Database pour PostgreSQL. Pour restaurer le fichier de vidage du schéma, exécutez la commande suivante :

    ```
    psql -h hostname -U db_username -d db_name < your_schema.sql
    ```

    Par exemple :

    ```
    psql -h mypgserver-20170401.postgres.database.azure.com  -U postgres -d dvdrental < dvdrentalSchema.sql
    ```

4. Si vous avez des clés étrangères dans votre schéma, la charge initiale et la synchronisation continue de la migration échouent. Pour extraire le script de suppression de clé étrangère et ajouter un script de clé étrangère à la destination (Azure Database pour PostgreSQL), exécutez le script suivant dans PgAdmin ou psql :
  
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

5. Exécutez la suppression de clé étrangère (deuxième colonne) dans le résultat de la requête pour supprimer la clé étrangère.

6. Si vous avez un déclencheur (d’insertion ou de mise à jour) dans les données, il appliquera l’intégrité des données dans la cible avant de répliquer les données de la source. Nous vous recommandons de désactiver les déclencheurs dans toutes les tables *côté cible* pendant la migration, puis d’activer les déclencheurs une fois la migration terminée.

    Pour désactiver les déclencheurs dans la base de données cible :

    ```
    SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
    ```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Inscrire le fournisseur de ressources Microsoft.DataMigration

1. Connectez-vous au portail Azure, sélectionnez **Tous les services**, puis **Abonnements**.

   ![Afficher les abonnements au portail](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-select-subscription1.png)

2. Sélectionnez l’abonnement dans lequel vous voulez créer l’instance Azure Database Migration Service, puis sélectionnez **Fournisseurs de ressources**.

    ![Afficher les fournisseurs de ressources](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-select-resource-provider.png)

3. Recherchez migration, puis à droite de **Microsoft.DataMigration**, sélectionnez **Inscrire**.

    ![S’inscrire auprès du fournisseur de ressources](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-register-resource-provider.png)

## <a name="create-an-instance-of-azure-database-migration-service"></a>Créer une instance Azure Database Migration Service

1. Dans le portail Azure, sélectionnez **+ Créer une ressource**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Service** dans la liste déroulante.

    ![Place de marché Azure](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/portal-marketplace.png)

2. Dans l’écran **Azure Database Migration Service**, sélectionnez **Créer**.

    ![Créer une instance Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create1.png)
  
3. Dans l’écran **Créer un service de migration**, spécifiez un nom pour le service, l’abonnement, et un réseau virtuel nouveau ou existant.

4. Sélectionnez l’emplacement dans lequel vous souhaitez créer l’instance Azure Database Migration Service.

5. Sélectionnez un réseau virtuel existant ou créez-en un.

    Le réseau virtuel fournit à Azure Database Migration Service un accès à l’instance PostgreSQL source et à l’instance Azure Database pour PostgreSQL cible.

    Pour plus d’informations sur la création d’un réseau virtuel dans le portail Azure, consultez l’article [Créer un réseau virtuel au moyen du portail Azure](../virtual-network/quick-create-portal.md).

6. Sélectionnez un niveau tarifaire. Pour cette migration en ligne, sélectionnez le niveau tarifaire Premium : 4vCores.

    ![Configurer des paramètres d’instance Azure Database Migration Service](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-settings5.png)

7. Sélectionnez **Créer** pour créer le service.

## <a name="create-a-migration-project"></a>Créer un projet de migration

Une fois le service créé, recherchez-le dans le portail Azure, ouvrez-le, puis créez un projet de migration.

1. Dans le portail Azure, sélectionnez **Tous les services**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Services**.

      ![Rechercher toutes les instances Azure Database Migration Service](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-search.png)

2. Dans l’écran **Azure Database Migration Services**, recherchez le nom de l’instance Azure Database Migration Service que vous avez créée et sélectionnez l’instance, puis **+ Nouveau projet de migration**.
3. Dans l’écran **Nouveau projet de migration**, spécifiez un nom pour le projet. Dans la zone de texte **Type de serveur source**, sélectionnez **AWS RDS pour PostgreSQL**, puis dans la zone de texte **Type de serveur cible**, sélectionnez **Azure Database pour PostgreSQL**.
4. Dans la section **Choisir un type d’activité**, sélectionnez **Migration de données en ligne**.

    > [!IMPORTANT]
    > Veillez à sélectionner **Migration des données en ligne** ; les migrations hors connexion ne sont pas prises en charge pour ce scénario.

    ![Créer un projet Azure Database Migration Service](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-create-project5.png)

    > [!NOTE]
    > Une autre possibilité consiste à choisir **Créer un projet uniquement** pour créer le projet de migration à ce stade et exécuter la migration ultérieurement.

5. Sélectionnez **Enregistrer**.

6. Sélectionnez **Créer et exécuter une activité** pour créer le projet et exécuter l’activité de migration.

    > [!NOTE]
    > Notez les conditions préalables requises pour configurer la migration en ligne dans le panneau de création de projet.

## <a name="specify-source-details"></a>Spécifier les détails de la source

* Dans l’écran **Ajouter les détails de la source**, spécifiez les détails de connexion de l’instance PostgreSQL source.

   ![Détails de la source](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-source-details5.png)

## <a name="specify-target-details"></a>Spécifier les détails de la cible

1. Sélectionnez **Enregistrer**, puis, dans l’écran **Détails de la cible**, spécifiez les détails de connexion pour le serveur Azure Database pour PostgreSQL cible qui est préprovisionné et dont le schéma **DVD Rentals** est déployé à l’aide de pg_dump.

    ![Détails de la cible](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-target-details.png)

2. Sélectionnez **Enregistrer**, puis dans l’écran **Mapper aux bases de données cibles**, mappez les bases de données source et cible pour la migration.

    Si la base de données cible porte le même nom que la base de données source, Azure Database Migration Service sélectionne la base de données cible par défaut.

    ![Mapper aux bases de données cibles](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-map-target-databases.png)

3. Sélectionnez **Enregistrer**, dans l’écran **Récapitulatif de la migration** pour la zone de texte **Nom de l’activité**, spécifiez un nom pour l’activité de migration, puis examinez le récapitulatif pour vous assurer que les détails de la source et de la cible correspondent à ceux que vous avez spécifiés précédemment.

    ![Résumé de la migration](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-migration-summary1.png)

## <a name="run-the-migration"></a>Exécuter la migration

* Sélectionnez **Exécuter la migration**.

    La fenêtre d’activité de migration s’affiche, et le champ **État** de l’activité présente la valeur **Initialisation en cours**.

## <a name="monitor-the-migration"></a>Surveiller la migration

1. Dans l’écran d’activité de migration, sélectionnez **Actualiser** pour mettre à jour l’affichage jusqu’à ce que le champ **État** de la migration prenne la valeur **En cours d’exécution**.

    ![État de l’activité – En cours d’exécution](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-activity-status3.png)

2. Sous **NOM DE LA BASE DE DONNÉES**, sélectionnez une base de données spécifique pour obtenir l’état de migration des opérations **Full data load** (Charge complète des données) et **Incremental data sync** (Synchronisation incrémentielle des données).

    L’opération **Full data load** (Charge complète des données) affiche l’état initial de migration de la charge, tandis que l’opération **Incremental data sync** (Synchronisation incrémentielle des données) indique l’état de capture des changements de données (CDC).

    ![Écran d’inventaire – charge complète des données](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-inventory-full-load.png)

    ![Écran d’inventaire – synchronisation incrémentielle des données](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-inventory-incremental.png)

## <a name="perform-migration-cutover"></a>Effectuer le basculement de la migration

Une fois le chargement complet initial effectué, les bases de données sont marquées de la mention **Ready to Cutover** (Prêt pour le basculement).

1. Lorsque vous êtes prêt à effectuer la migration de base de données, sélectionnez **Démarrer le basculement**.

2. Attendez que le compteur **Modifications en attente** affiche **0** pour vous assurer que toutes les transactions entrantes dans la base de données source sont arrêtées, activez la case à cocher **Confirmer**, puis sélectionnez **Appliquer**.

    ![Écran de basculement terminé](media/tutorial-rds-postgresql-server-azure-db-for-postgresql-online/dms-complete-cutover.png)

3. Lorsque l’état de migration de la base de données est **Terminé**, connectez vos applications à la nouvelle base de données Azure Database pour PostgreSQL cible.

Votre migration en ligne d’une instance locale de Services Bureau à distance PostgreSQL vers Azure Database pour PostgreSQL est maintenant terminée.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Azure Database Migration Service, consultez l’article [Qu’est-ce qu’Azure Database Migration Service ?](./dms-overview.md).
* Pour obtenir des informations sur Azure Database pour PostgreSQL, consultez l’article [Qu’est-ce qu’Azure Database pour PostgreSQL ?](../postgresql/overview.md).
* Si vous avez d’autres questions, envoyez un courrier pour [Poser des questions sur les migrations de base de données Azure](mailto:AskAzureDatabaseMigrations@service.microsoft.com).