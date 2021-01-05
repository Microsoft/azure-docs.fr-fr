---
title: 'Tutoriel : Migrer PostgreSQL vers Azure DB pour PostgreSQL en ligne via le Portail Azure'
titleSuffix: Azure Database Migration Service
description: Découvrez comment effectuer la migration en ligne d’une instance PostgreSQL locale vers Azure Database pour PostgreSQL à l’aide d’Azure Database Migration Service via le Portail Azure.
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
ms.openlocfilehash: 6b9822e16a9c5604371bd4c8c1e136ce78f29820
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000774"
---
# <a name="tutorial-migrate-postgresql-to-azure-db-for-postgresql-online-using-dms-via-the-azure-portal"></a>Tutoriel : Migrer PostgreSQL vers Azure DB pour PostgreSQL en ligne via le Portail Azure à l’aide de DMS

Vous pouvez utiliser Azure Database Migration Service pour migrer les bases de données d’une instance PostgreSQL locale vers [Azure Database pour PostgreSQL](../postgresql/index.yml) avec un temps d’arrêt minimal pour l’application. Dans ce tutoriel, vous allez migrer l’exemple de base de données **DVD Rental** à partir d’une instance PostgreSQL 9.6 locale vers Azure Database pour PostgreSQL à l’aide de l’activité de migration en ligne dans Azure Database Migration Service.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
>
> * Migrer l’exemple de schéma à l’aide de l’utilitaire pg_dump.
> * Créer une instance Azure Database Migration Service.
> * Créer un projet de migration dans Azure Database Migration Service.
> * Exécuter la migration.
> * Surveiller la migration.
> * Effectuer le basculement de la migration.

> [!NOTE]
> L’utilisation d’Azure Database Migration Service pour effectuer une migration en ligne nécessite la création d’une instance basée sur le niveau tarifaire Premium. Nous chiffrons le disque pour empêcher le vol de données pendant le processus de migration.

> [!IMPORTANT]
> Pour une expérience de migration optimale, Microsoft recommande de créer une instance Azure Database Migration Service dans la même région Azure que la base de données cible. Le déplacement des données entre les régions ou les zones géographiques peut ralentir le processus de migration et introduire des erreurs.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez effectuer les opérations suivantes :

* Téléchargez et installez [PostgreSQL Community Edition](https://www.postgresql.org/download/) 9.4, 9.5, 9.6 ou 10. La version du serveur PostgreSQL source doit être 9.4, 9.5, 9.6, 10 ou 11. Pour plus d’informations, consultez l’article [Versions de bases de données PostgreSQL prises en charge](../postgresql/concepts-supported-versions.md).

    Notez également que la version cible d'Azure Database pour PostgreSQL doit être égale ou ultérieure à la version PostgreSQL locale. Par exemple, PostgreSQL 9.6 peut migrer vers Azure Database pour PostgreSQL 9.6, 10 ou 11, et non vers Azure Database pour PostgreSQL 9.5.

* [Créez un serveur dans Azure Database pour PostgreSQL](../postgresql/quickstart-create-server-database-portal.md) ou [créez un serveur Azure Database pour PostgreSQL – Hyperscale (Citus)](../postgresql/quickstart-create-hyperscale-portal.md).
* Créez un Réseau virtuel Microsoft Azure pour Azure Database Migration Service à l’aide du modèle de déploiement Azure Resource Manager, qui fournit une connectivité site à site à vos serveurs sources locaux via [ExpressRoute](../expressroute/expressroute-introduction.md) ou un [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Pour plus d’informations sur la création d’un réseau virtuel, consultez la [documentation sur le réseau virtuel](../virtual-network/index.yml), en particulier les articles sur le démarrage rapide, qui fournissent des informations pas à pas.

    > [!NOTE]
    > Pendant la configuration du réseau virtuel, si vous utilisez ExpressRoute avec le peering réseau à Microsoft, ajoutez ces [points de terminaison](../virtual-network/virtual-network-service-endpoints-overview.md) au sous-réseau où doit être provisionné le service :
    >
    > * Point de terminaison de base de données cible (un point de terminaison SQL ou Cosmos DB, par exemple)
    > * Point de terminaison de stockage
    > * Point de terminaison Service Bus
    >
    > Cette configuration est nécessaire car Azure Database Migration Service ne dispose pas d’une connectivité Internet.

* Vérifiez que les règles du groupe de sécurité réseau (NSG) de votre réseau virtuel ne bloquent pas les ports de communication entrante suivants pour Azure Database Migration Service : 443, 53, 9354, 445, 12000. Pour plus d’informations sur le filtrage du trafic de groupe de sécurité réseau de réseau virtuel, consultez l’article [Filtrer le trafic avec les groupes de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Configurez votre [pare-feu Windows pour accéder au moteur de base de données](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Ouvrez votre pare-feu Windows pour permettre à Azure Database Migration Service d’accéder au serveur PostgreSQL source, par défaut le port TCP 5432.
* Lorsque vous utilisez une appliance de pare-feu devant vos bases de données sources, vous devrez peut-être ajouter des règles de pare-feu pour permettre à Azure Database Migration Service d’accéder aux bases de données sources pour la migration.
* Créez une [règle de pare-feu](../azure-sql/database/firewall-configure.md) de niveau serveur pour Azure Database pour PostgreSQL afin de permettre à Azure Database Migration Service d’accéder aux bases de données cibles. Fournissez la plage de sous-réseau du réseau virtuel utilisé pour Azure Database Migration Service.
* Activez la réplication logique dans le fichier postgresql.config, puis définissez les paramètres suivants :

  * wal_level = **logical**
  * max_replication_slots = [nombre d’emplacements]. Valeur recommandée : **cinq emplacements**
  * max_wal_senders = [nombre de tâches simultanées] - le paramètre max_wal_senders définit le nombre de tâches simultanées qui peuvent s’exécuter, paramètre recommandé : **10 tâches**

> [!IMPORTANT]
> Toutes les tables de votre base de données existante nécessitent une clé primaire pour garantir que les modifications peuvent être synchronisées avec la base de données cible.

## <a name="migrate-the-sample-schema"></a>Migrer l’exemple de schéma

Pour compléter tous les objets de base de données tels que les schémas de table, les index et les procédures stockées, nous devons extraire le schéma à partir de la base de données source et l’appliquer à la base de données.

1. Utilisez la commande pg_dump -s afin de créer un fichier de vidage de schéma pour une base de données.

    ```
    pg_dump -o -h hostname -U db_username -d db_name -s > your_schema.sql
    ```

    Par exemple, pour créer un fichier de vidage de schéma pour la base de données **dvdrental** :

    ```
    pg_dump -o -h localhost -U postgres -d dvdrental -s -O -x > dvdrentalSchema.sql
    ```

    Pour plus d’informations sur l’utilisation de l’utilitaire pg_dump, consultez les exemples du tutoriel [pg-dump](https://www.postgresql.org/docs/9.6/static/app-pgdump.html#PG-DUMP-EXAMPLES).

2. Créez une base de données vide dans votre environnement cible, c’est-à-dire Azure Database pour PostgreSQL.

    Pour plus d’informations sur la création et la connexion d’une base de données, consultez l’article [Créer un serveur Azure Database pour PostgreSQL dans le Portail Azure](../postgresql/quickstart-create-server-database-portal.md) ou [Créer un serveur Azure Database pour PostgreSQL – Hyperscale (Citus) dans le Portail Azure](../postgresql/quickstart-create-hyperscale-portal.md).

    > [!NOTE]
    > Une instance Azure Database pour PostgreSQL – Hyperscale (Citus) dispose uniquement d’une base de données unique : **citus**.

3. Importez le schéma dans la base de données cible que vous avez créée en restaurant le fichier de vidage du schéma.

    ```
    psql -h hostname -U db_username -d db_name < your_schema.sql
    ```

    Par exemple :

    ```
    psql -h mypgserver-20170401.postgres.database.azure.com  -U postgres -d dvdrental citus < dvdrentalSchema.sql
    ```

4. Pour extraire le script de clé étrangère Drop et l’ajouter à la destination (Azure Database pour PostgreSQL), exécutez le script suivant dans PgAdmin ou psql.

   > [!IMPORTANT]
   > Les clés étrangères de votre schéma entraînent l’échec de la charge initiale et de la synchronisation continue de la migration.

    ```
    SELECT Q.table_name
        ,CONCAT('ALTER TABLE ', table_schema, '.', table_name, STRING_AGG(DISTINCT CONCAT(' DROP CONSTRAINT ', foreignkey), ','), ';') as DropQuery
            ,CONCAT('ALTER TABLE ', table_schema, '.', table_name, STRING_AGG(DISTINCT CONCAT(' ADD CONSTRAINT ', foreignkey, ' FOREIGN KEY (', column_name, ')', ' REFERENCES ', foreign_table_schema, '.', foreign_table_name, '(', foreign_column_name, ')' ), ','), ';') as AddQuery
    FROM
        (SELECT
        S.table_schema,
        S.foreignkey,
        S.table_name,
        STRING_AGG(DISTINCT S.column_name, ',') AS column_name,
        S.foreign_table_schema,
        S.foreign_table_name,
        STRING_AGG(DISTINCT S.foreign_column_name, ',') AS foreign_column_name
    FROM
        (SELECT DISTINCT
        tc.table_schema,
        tc.constraint_name AS foreignkey,
        tc.table_name,
        kcu.column_name,
        ccu.table_schema AS foreign_table_schema,
        ccu.table_name AS foreign_table_name,
        ccu.column_name AS foreign_column_name
        FROM information_schema.table_constraints AS tc
        JOIN information_schema.key_column_usage AS kcu ON tc.constraint_name = kcu.constraint_name AND tc.table_schema = kcu.table_schema
        JOIN information_schema.constraint_column_usage AS ccu ON ccu.constraint_name = tc.constraint_name AND ccu.table_schema = tc.table_schema
    WHERE constraint_type = 'FOREIGN KEY'
        ) S
        GROUP BY S.table_schema, S.foreignkey, S.table_name, S.foreign_table_schema, S.foreign_table_name
        ) Q
        GROUP BY Q.table_schema, Q.table_name;
    ```

5. Exécutez la clé étrangère Drop (deuxième colonne) dans le résultat de la requête.

6. Pour désactiver les déclencheurs dans la base de données cible, exécutez le script ci-dessous.

   > [!IMPORTANT]
   > Des déclencheurs (d’insertion ou de mise à jour) dans les données appliqueront l’intégrité des données dans la cible avant les données répliquées à partir de la source. Par conséquent, il est recommandé de désactiver les déclencheurs de toutes les tables **au niveau de la cible** pendant la migration, puis de réactiver les déclencheurs une fois celle-ci terminée.

    ```
    SELECT DISTINCT CONCAT('ALTER TABLE ', event_object_schema, '.', event_object_table, ' DISABLE TRIGGER ', trigger_name, ';')
    FROM information_schema.triggers
    ```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Inscrire le fournisseur de ressources Microsoft.DataMigration

1. Connectez-vous au portail Azure, sélectionnez **Tous les services**, puis **Abonnements**.

   ![Afficher les abonnements au portail](media/tutorial-postgresql-to-azure-postgresql-online-portal/portal-select-subscriptions.png)

2. Sélectionnez l’abonnement dans lequel vous voulez créer l’instance Azure Database Migration Service, puis sélectionnez **Fournisseurs de ressources**.

    ![Afficher les fournisseurs de ressources](media/tutorial-postgresql-to-azure-postgresql-online-portal/portal-select-resource-provider.png)

3. Recherchez migration, puis à droite de **Microsoft.DataMigration**, sélectionnez **Inscrire**.

    ![S’inscrire auprès du fournisseur de ressources](media/tutorial-postgresql-to-azure-postgresql-online-portal/portal-register-resource-provider.png)

## <a name="create-a-dms-instance"></a>Créer une instance de service DMS

1. Dans le portail Azure, sélectionnez **+ Créer une ressource**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Service** dans la liste déroulante.

    ![Place de marché Azure](media/tutorial-postgresql-to-azure-postgresql-online-portal/portal-marketplace.png)

2. Dans l’écran **Azure Database Migration Service**, sélectionnez **Créer**.

    ![Créer une instance Azure Database Migration Service](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-create1.png)
  
3. Dans l’écran **Créer un service de migration**, spécifiez un nom, l’abonnement, un groupe de ressources nouveau ou existant et l’emplacement pour le service.

4. Sélectionnez un réseau virtuel existant ou créez-en un.

    Le réseau virtuel fournit à Azure Database Migration Service un accès au serveur PostgreSQL source et à l’instance Azure Database pour PostgreSQL cible.

    Pour plus d’informations sur la création d’un réseau virtuel dans le portail Azure, consultez l’article [Créer un réseau virtuel au moyen du portail Azure](../virtual-network/quick-create-portal.md).

5. Sélectionnez un niveau tarifaire.

    Pour plus d’informations sur les coûts et les niveaux de tarification, consultez la [page de tarification](https://aka.ms/dms-pricing).

    ![Configurer des paramètres d’instance Azure Database Migration Service](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-settings4.png)

6. Sélectionnez **Vérifier + créer** pour créer le service.

   La création du service se terminera en 10 à 15 minutes environ.

## <a name="create-a-migration-project"></a>Créer un projet de migration

Une fois le service créé, recherchez-le dans le portail Azure, ouvrez-le, puis créez un projet de migration.

1. Dans le portail Azure, sélectionnez **Tous les services**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Services**.

      ![Localiser toutes les instances Azure Database Migration Service](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-search.png)

2. Dans l’écran **Azure Database Migration Services**, recherchez le nom de l’instance Azure Database Migration Service que vous avez créée et sélectionnez l’instance, puis **+ Nouveau projet de migration**.

3. Dans l’écran **Nouveau projet de migration**, spécifiez un nom pour le projet, dans la zone de texte **Type de serveur source**, sélectionnez **PostgreSQL** et, dans la zone de texte **Type de serveur cible**, sélectionnez **Azure Database pour PostgreSQL**.

4. Dans la section **Choisir un type d’activité**, sélectionnez **Migration de données en ligne**.

    ![Créer un projet Azure Database Migration Service](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-create-project.png)

    > [!NOTE]
    > Une autre possibilité consiste à choisir **Créer un projet uniquement** pour créer le projet de migration à ce stade et exécuter la migration ultérieurement.

5. Sélectionnez **Enregistrer**, notez les conditions requises pour pouvoir utiliser Azure Database Migration Service pour migrer des données, puis sélectionnez **Créer et exécuter une activité**.

## <a name="specify-source-details"></a>Spécifier les détails de la source

1. Dans l’écran **Ajouter les détails de la source**, spécifiez les détails de connexion de l’instance PostgreSQL source.

    ![Écran Ajouter les détails de la source](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-add-source-details.png)

2. Sélectionnez **Enregistrer**.

## <a name="specify-target-details"></a>Spécifier les détails de la cible

1. Dans l’écran **Détails de la cible**, spécifiez les détails de connexion pour le serveur Hyperscale (Citus) cible, qui est l’instance préapprovisionnée d’Hyperscale (Citus) sur laquelle le schéma **DVD Rentals** a été déployé à l’aide de pg_dump.

    ![Écran Détails de la cible](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-add-target-details.png)

2. Sélectionnez **Enregistrer**, puis dans l’écran **Mapper aux bases de données cibles**, mappez les bases de données source et cible pour la migration.

    Si la base de données cible porte le même nom que la base de données source, Azure Database Migration Service sélectionne la base de données cible par défaut.

    ![Écran Mapper aux bases de données cibles](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-map-target-databases.png)

3. Sélectionnez **Enregistrer** puis, dans l’écran **Paramètres de migration**, acceptez les valeurs par défaut.

    ![Écran Paramètres de migration](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-migration-settings.png)

4. Sélectionnez **Enregistrer**, dans l’écran **Récapitulatif de la migration** pour la zone de texte **Nom de l’activité**, spécifiez un nom pour l’activité de migration, puis examinez le récapitulatif pour vous assurer que les détails de la source et de la cible correspondent à ceux que vous avez spécifiés précédemment.

    ![Écran Récapitulatif de la migration](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-migration-summary.png)

## <a name="run-the-migration"></a>Exécuter la migration

* Sélectionnez **Exécuter la migration**.

    La fenêtre d’activité de migration s’affiche, et l’**état** de l’activité doit se mettre à jour pour indiquer **Sauvegarde en cours**.

## <a name="monitor-the-migration"></a>Surveiller la migration

1. Dans l’écran d’activité de migration, sélectionnez **Actualiser** pour mettre à jour l’affichage jusqu’à voir **l’état** de la migration s’afficher comme **Terminé**.

     ![Processus Surveiller la migration](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-monitor-migration.png)

2. Lorsque la migration est terminée, sous **Nom de la base de données**, sélectionnez une base de données spécifique pour obtenir l’état de migration des opérations **Full data load** (Charge complète des données) et **Incremental data sync** (Synchronisation incrémentielle des données).

   > [!NOTE]
   > L’opération **Full data load** (Charge complète des données) affiche l’état initial de migration de la charge, tandis que l’opération **Incremental data sync** (Synchronisation incrémentielle des données) indique l’état de capture des changements de données (CDC).

     ![Détails de la charge complète des données](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-full-data-load-details.png)

     ![Détails de la synchronisation incrémentielle des données](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-incremental-data-sync-details.png)

## <a name="perform-migration-cutover"></a>Effectuer le basculement de la migration

Une fois le chargement complet initial effectué, les bases de données identifiées par le libellé **Prêt pour le basculement**.

1. Lorsque vous êtes prêt à effectuer la migration de base de données, sélectionnez **Démarrer le basculement**.

2. Attendez que le compteur **Modifications en attente** affiche **0** pour vous assurer que toutes les transactions entrantes dans la base de données source sont arrêtées, activez la case à cocher **Confirmer**, puis sélectionnez **Appliquer**.

    ![Écran de basculement terminé](media/tutorial-postgresql-to-azure-postgresql-online-portal/dms-complete-cutover.png)

3. Lorsque l’état de migration de la base de données indique **Terminé**, [recréez des séquences](https://wiki.postgresql.org/wiki/Fixing_Sequences) (le cas échéant) et connectez vos applications à la nouvelle instance cible d’Azure Database pour PostgreSQL.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les problèmes connus et les limitations lors de l’exécution de migrations en ligne vers Azure Database pour PostgreSQL, consultez l’article [Known issues and workarounds with Azure Database for PostgreSQL online migrations](known-issues-azure-postgresql-online.md) (Problèmes connus et solutions de contournement pour les migrations Azure Database pour PostgreSQL en ligne).
* Pour plus d’informations sur Azure Database Migration Service, consultez l’article [Qu’est-ce qu’Azure Database Migration Service ?](./dms-overview.md).
* Pour obtenir des informations sur Azure Database pour PostgreSQL, consultez l’article [Qu’est-ce qu’Azure Database pour PostgreSQL ?](../postgresql/overview.md).