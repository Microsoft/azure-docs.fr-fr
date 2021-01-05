---
title: 'Tutoriel : Migrer Azure Database pour PostgreSQL vers Azure Database pour PostgreSQL en ligne via le portail Azure'
titleSuffix: Azure Database Migration Service
description: Découvrez comment effectuer la migration en ligne d’une instance Azure Database pour PostgreSQL vers une autre à l’aide d’Azure Database Migration Service via le portail Azure.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 07/21/2020
ms.openlocfilehash: 772fc9f21f36a1487e2e3a1acbe644f9fd12e0f4
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97563192"
---
# <a name="tutorial-migrateupgrade-azure-db-for-postgresql---single-server-to-azure-db-for-postgresql---single-server--online-using-dms-via-the-azure-portal"></a>Tutoriel : Migrer/Mettre à niveau Azure Database pour PostgreSQL – Serveur unique vers Azure Database pour PostgreSQL – Serveur unique en ligne utilisant DMS via le portail Azure

Vous pouvez utiliser Azure Database Migration Service pour migrer les bases de données d’une instance [Azure Database pour PostgreSQL – Serveur unique](../postgresql/overview.md#azure-database-for-postgresql---single-server) vers une version identique ou différente d’une instance Azure Database pour PostgreSQL – Serveur unique ou vers Azure Database pour PostgreSQL – Serveur flexible moyennant un temps d’arrêt minimal. Dans ce tutoriel, vous allez migrer l’exemple de base de données **DVD Rental** à partir d’une instance Azure Database pour PostgreSQL v10 vers Azure Database pour PostgreSQL – Serveur unique à l’aide de l’activité de migration en ligne dans Azure Database Migration Service.

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

> [!IMPORTANT]
> La migration à partir d’Azure Database pour PostgreSQL est prise en charge pour PostgreSQL version 10 et versions ultérieures. Vous pouvez également utiliser ce didacticiel pour migrer d’une instance Azure Database pour PostgreSQL à une autre ou vers une instance Hyperscale (Citus).

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez effectuer les opérations suivantes :

* Vérifier [État des scénarios de migration pris en charge par Azure Database Migration Service](./resource-scenario-status.md) pour connaître les combinaisons de migration et de versions prises en charge. 
* Une instance [Azure Database pour PostgreSQL](../postgresql/index.yml) version 10 et ultérieure existante avec la base de données **DVD Rental**. 

    Notez également que la version cible d'Azure Database pour PostgreSQL doit être égale ou ultérieure à la version PostgreSQL locale. Par exemple, PostgreSQL 10 peut migrer vers Azure Database pour PostgreSQL 10 ou 11, et non vers Azure Database pour PostgreSQL 9.6.

* [Créer un serveur dans Azure Database pour PostgreSQL](../postgresql/quickstart-create-server-database-portal.md) ou [créer un serveur Azure Database pour PostgreSQL – Hyperscale (Citus)](../postgresql/quickstart-create-hyperscale-portal.md) comme serveur de base de données cible pour y migrer les données.
* Créer un réseau virtuel Microsoft Azure pour Azure Database Migration Service à l’aide du modèle de déploiement Azure Resource Manager. Pour plus d’informations sur la création d’un réseau virtuel, consultez la [documentation sur le réseau virtuel](../virtual-network/index.yml), en particulier les articles sur le démarrage rapide, qui fournissent des informations pas à pas.

* Vérifiez que les règles du groupe de sécurité réseau (NSG) de votre réseau virtuel ne bloquent pas les ports de communication entrante suivants pour Azure Database Migration Service : 443, 53, 9354, 445, 12000. Pour plus d’informations sur le filtrage du trafic de groupe de sécurité réseau de réseau virtuel, consultez l’article [Filtrer le trafic avec les groupes de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Créez une [règle de pare-feu](../azure-sql/database/firewall-configure.md) de niveau serveur pour la source Azure Database pour PostgreSQL afin de permettre à Azure Database Migration Service d’accéder aux bases de données sources. Fournissez la plage de sous-réseau du réseau virtuel utilisé pour Azure Database Migration Service.
* Créez une [règle de pare-feu](../azure-sql/database/firewall-configure.md) de niveau serveur pour la cible Azure Database pour PostgreSQL afin de permettre à Azure Database Migration Service d’accéder aux bases de données cibles. Fournissez la plage de sous-réseau du réseau virtuel utilisé pour Azure Database Migration Service.
* [Activez la réplication logique](../postgresql/concepts-logical.md) dans la source Azure Database pour PostgreSQL. 
* Définissez les paramètres de serveur suivants dans l’instance Azure Database pour PostgreSQL utilisée comme source :

  * max_replication_slots = [nombre d’emplacements], valeur recommandée **dix emplacements**
  * max_wal_senders = [nombre de tâches simultanées] - le paramètre max_wal_senders définit le nombre de tâches simultanées qui peuvent s’exécuter, paramètre recommandé : **10 tâches**

> [!NOTE]
> Les paramètres de serveur ci-dessus sont statiques et nécessitent un redémarrage de votre instance Azure Database pour PostgreSQL pour qu’ils prennent effet. Pour plus d’informations sur le basculement des paramètres du serveur, consultez [Configurer les paramètres Azure Database pour PostgreSQL](../postgresql/howto-configure-server-parameters-using-portal.md).

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
    pg_dump -o -h mypgserver-source.postgres.database.azure.com -U pguser@mypgserver-source -d dvdrental -s -O -x > dvdrentalSchema.sql
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
    psql -h mypgserver-source.postgres.database.azure.com  -U pguser@mypgserver-source -d dvdrental citus < dvdrentalSchema.sql
    ```

4. Pour extraire le script de clé étrangère Drop et l’ajouter à la destination (Azure Database pour PostgreSQL), exécutez le script suivant dans PgAdmin ou psql.

   > [!IMPORTANT]
   > Les clés étrangères de votre schéma entraînent l’échec de la charge initiale et de la synchronisation continue de la migration.

    ```
    SELECT Q.table_name
        ,CONCAT('ALTER TABLE ','"', table_schema,'"', '.','"', table_name ,'"', STRING_AGG(DISTINCT CONCAT(' DROP CONSTRAINT ','"', foreignkey,'"'), ','), ';') as DropQuery
            ,CONCAT('ALTER TABLE ','"', table_schema,'"', '.','"', table_name,'"', STRING_AGG(DISTINCT CONCAT(' ADD CONSTRAINT ','"', foreignkey,'"', ' FOREIGN KEY (','"', column_name,'"', ')', ' REFERENCES ','"', foreign_table_schema,'"', '.','"', foreign_table_name,'"', '(','"', foreign_column_name,'"', ')',' ON UPDATE ',update_rule,' ON DELETE ',delete_rule), ','), ';') as AddQuery
    FROM
        (SELECT
        S.table_schema,
        S.foreignkey,
        S.table_name,
        STRING_AGG(DISTINCT S.column_name, ',') AS column_name,
        S.foreign_table_schema,
        S.foreign_table_name,
        STRING_AGG(DISTINCT S.foreign_column_name, ',') AS foreign_column_name,
        S.update_rule,
        S.delete_rule
    FROM
        (SELECT DISTINCT
        tc.table_schema,
        tc.constraint_name AS foreignkey,
        tc.table_name,
        kcu.column_name,
        ccu.table_schema AS foreign_table_schema,
        ccu.table_name AS foreign_table_name,
        ccu.column_name AS foreign_column_name,
        rc.update_rule AS update_rule,
        rc.delete_rule AS delete_rule
        FROM information_schema.table_constraints AS tc
        JOIN information_schema.key_column_usage AS kcu ON tc.constraint_name = kcu.constraint_name AND tc.table_schema = kcu.table_schema
        JOIN information_schema.constraint_column_usage AS ccu ON ccu.constraint_name = tc.constraint_name AND ccu.table_schema = tc.table_schema
        JOIN information_schema.referential_constraints as rc ON rc.constraint_name = tc.constraint_name AND rc.constraint_schema = tc.table_schema
    WHERE constraint_type = 'FOREIGN KEY'
        ) S
        GROUP BY S.table_schema, S.foreignkey, S.table_name, S.foreign_table_schema, S.foreign_table_name,S.update_rule,S.delete_rule
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

   ![Afficher les abonnements au portail](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/portal-select-subscriptions.png)

2. Sélectionnez l’abonnement dans lequel vous voulez créer l’instance Azure Database Migration Service, puis sélectionnez **Fournisseurs de ressources**.

    ![Afficher les fournisseurs de ressources](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/portal-select-resource-provider.png)

3. Recherchez migration, puis à droite de **Microsoft.DataMigration**, sélectionnez **Inscrire**.

    ![S’inscrire auprès du fournisseur de ressources](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/portal-register-resource-provider.png)

## <a name="create-a-dms-instance"></a>Créer une instance de service DMS

1. Dans le portail Azure, sélectionnez **+ Créer une ressource**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Service** dans la liste déroulante.

    ![Place de marché Azure](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/portal-marketplace.png)

2. Dans l’écran **Azure Database Migration Service**, sélectionnez **Créer**.

    ![Créer une instance Azure Database Migration Service](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/dms-create1.png)
  
3. Dans l’écran **Créer un service de migration**, spécifiez un nom, l’abonnement, un groupe de ressources nouveau ou existant et l’emplacement pour le service.

4. Sélectionnez un réseau virtuel existant ou créez-en un.

    Le réseau virtuel fournit à Azure Database Migration Service un accès au serveur PostgreSQL source et à l’instance Azure Database pour PostgreSQL cible.

    Pour plus d’informations sur la création d’un réseau virtuel dans le portail Azure, consultez l’article [Créer un réseau virtuel au moyen du portail Azure](../virtual-network/quick-create-portal.md).

5. Sélectionnez un niveau tarifaire.

    Pour plus d’informations sur les coûts et les niveaux de tarification, consultez la [page de tarification](https://aka.ms/dms-pricing).

    ![Configurer des paramètres d’instance Azure Database Migration Service](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/dms-settings4.png)

6. Sélectionnez **Vérifier + créer** pour créer le service.

   La création du service se terminera en 10 à 15 minutes environ.

## <a name="create-a-migration-project"></a>Créer un projet de migration

Une fois le service créé, recherchez-le dans le portail Azure, ouvrez-le, puis créez un projet de migration.

1. Dans le portail Azure, sélectionnez **Tous les services**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Services**.

      ![Localiser toutes les instances Azure Database Migration Service](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/dms-search.png)

2. Dans l’écran **Azure Database Migration Services**, recherchez le nom de l’instance Azure Database Migration Service que vous avez créée et sélectionnez l’instance, puis **+ Nouveau projet de migration**.

3. Dans l’écran **Nouveau projet de migration**, spécifiez un nom pour le projet, dans la zone de texte **Type de serveur source**, sélectionnez **PostgreSQL** et, dans la zone de texte **Type de serveur cible**, sélectionnez **Azure Database pour PostgreSQL**.
    > [!NOTE]
    > Choisissez **PostgreSQL** dans **Type de serveur source** même si le serveur source est une instance **Azure Database pour PostgreSQL**.  

4. Dans la section **Choisir un type d’activité**, sélectionnez **Migration de données en ligne**.

    ![Créer un projet Azure Database Migration Service](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/dms-create-project.png)

    > [!NOTE]
    > Une autre possibilité consiste à choisir **Créer un projet uniquement** pour créer le projet de migration à ce stade et exécuter la migration ultérieurement.

5. Sélectionnez **Enregistrer**, notez les conditions requises pour pouvoir utiliser Azure Database Migration Service pour migrer des données, puis sélectionnez **Créer et exécuter une activité**.

## <a name="specify-source-details"></a>Spécifier les détails de la source

1. Dans l’écran **Ajouter les détails de la source**, spécifiez les détails de connexion de l’instance PostgreSQL source.

    ![Écran Ajouter les détails de la source](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/dms-add-source-details.png)

    > [!NOTE]
    > Vous pouvez trouver des détails tels que « Nom du serveur », « Port du serveur », « Nom de la base de données », etc., dans le portail **Azure Database pour PostgreSQL**.

2. Sélectionnez **Enregistrer**.

## <a name="specify-target-details"></a>Spécifier les détails de la cible

1. Dans l’écran **Détails de la cible**, spécifiez les détails de connexion pour le serveur Hyperscale (Citus) cible, qui est l’instance préapprovisionnée d’Hyperscale (Citus) sur laquelle le schéma **DVD Rentals** a été déployé à l’aide de pg_dump.

    ![Écran Détails de la cible](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/dms-add-target-details.png)

    > [!NOTE]
    > Vous pouvez migrer d’une instance Azure Database pour PostgreSQL vers une autre instance de serveur unique Azure Database pour PostgreSQL ou vers un serveur Hyperscale (Citus).

2. Sélectionnez **Enregistrer**, puis dans l’écran **Mapper aux bases de données cibles**, mappez les bases de données source et cible pour la migration.

    Si la base de données cible porte le même nom que la base de données source, Azure Database Migration Service sélectionne la base de données cible par défaut.

    ![Écran Mapper aux bases de données cibles](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/dms-map-target-databases.png)

3. Sélectionnez **Enregistrer** puis, dans l’écran **Paramètres de migration**, acceptez les valeurs par défaut.

    ![Écran Paramètres de migration](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/dms-migration-settings.png)

4. Sélectionnez **Enregistrer**, dans l’écran **Récapitulatif de la migration** pour la zone de texte **Nom de l’activité**, spécifiez un nom pour l’activité de migration, puis examinez le récapitulatif pour vous assurer que les détails de la source et de la cible correspondent à ceux que vous avez spécifiés précédemment.

    ![Écran Récapitulatif de la migration](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/dms-migration-summary.png)

## <a name="run-the-migration"></a>Exécuter la migration

* Sélectionnez **Exécuter la migration**.

La fenêtre d’activité de migration s’affiche, et l’**état** de l’activité doit se mettre à jour pour indiquer **Sauvegarde en cours**. Vous pouvez rencontrer l’erreur suivante lors de la mise à niveau à partir d’Azure DB pour PostgreSQL 9.5 ou 9.6 :

**Un scénario a signalé une erreur inconnue. 28000: no pg_hba.conf entry for replication connection from host "40.121.141.121", user "sr"**

Cela est dû au fait que PostgreSQL ne dispose pas des privilèges exigés pour créer les artefacts de réplication logique nécessaires. Pour activer les privilèges exigés, vous pouvez effectuer les opérations suivantes :

1. Ouvrez les paramètres « Sécurité de la connexion » du serveur source Azure DB pour PostgreSQL que vous souhaitez déplacer ou mettre à niveau.
2. Ajoutez une nouvelle règle de pare-feu avec un nom se terminant par « _replrule », puis ajoutez l’adresse IP du message d’erreur aux champs Adresse IP de début et Adresse IP de fin. Pour l’exemple d’erreur ci-dessus -
> Nom de la règle de pare-feu = sr_replrule; Adresse IP de début = 40.121.141.121; Adresse IP de fin = 40.121.141.121

3. Cliquez sur Enregistrer, puis attendez que la modification soit terminée. 
4. Retentez l’activité DMS. 

## <a name="monitor-the-migration"></a>Surveiller la migration

1. Dans l’écran d’activité de migration, sélectionnez **Actualiser** pour mettre à jour l’affichage jusqu’à voir **l’état** de la migration s’afficher comme **Terminé**.

     ![Processus Surveiller la migration](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/dms-monitor-migration.png)

2. Lorsque la migration est terminée, sous **Nom de la base de données**, sélectionnez une base de données spécifique pour obtenir l’état de migration des opérations **Full data load** (Charge complète des données) et **Incremental data sync** (Synchronisation incrémentielle des données).

   > [!NOTE]
   > L’opération **Full data load** (Charge complète des données) affiche l’état initial de migration de la charge, tandis que l’opération **Incremental data sync** (Synchronisation incrémentielle des données) indique l’état de capture des changements de données (CDC).

     ![Détails de la charge complète des données](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/dms-full-data-load-details.png)

     ![Détails de la synchronisation incrémentielle des données](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/dms-incremental-data-sync-details.png)

## <a name="perform-migration-cutover"></a>Effectuer le basculement de la migration

Une fois le chargement complet initial effectué, les bases de données identifiées par le libellé **Prêt pour le basculement**.

1. Lorsque vous êtes prêt à effectuer la migration de base de données, sélectionnez **Démarrer le basculement**.

2. Attendez que le compteur **Modifications en attente** affiche **0** pour vous assurer que toutes les transactions entrantes dans la base de données source sont arrêtées, activez la case à cocher **Confirmer**, puis sélectionnez **Appliquer**.

    ![Écran de basculement terminé](media/tutorial-azure-postgresql-to-azure-postgresql-online-portal/dms-complete-cutover.png)

3. Lorsque l’état de migration de la base de données indique **Terminé**, [recréez des séquences](https://wiki.postgresql.org/wiki/Fixing_Sequences) (le cas échéant) et connectez vos applications à la nouvelle instance cible d’Azure Database pour PostgreSQL.
 
> [!NOTE]
> Azure Database Migration Service permet d’effectuer des mises à niveau de version majeures avec un temps d’arrêt réduit dans Azure Database pour PostgreSQL – Serveur unique. Vous devez tout d’abord configurer une base de données cible avec la version de PostgreSQL supérieure, les paramètres réseau et les paramètres souhaités. Ensuite, vous pouvez lancer la migration vers les bases de données cibles à l’aide de la procédure décrite ci-dessus. Une fois le basculement vers le serveur de base de données cible effectué, vous pouvez mettre à jour votre chaîne de connexion d’application pour qu’elle pointe vers le serveur de base de données cible. 

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les problèmes connus et les limitations lors de l’exécution de migrations en ligne vers Azure Database pour PostgreSQL, consultez l’article [Known issues and workarounds with Azure Database for PostgreSQL online migrations](known-issues-azure-postgresql-online.md) (Problèmes connus et solutions de contournement pour les migrations Azure Database pour PostgreSQL en ligne).
* Pour plus d’informations sur Azure Database Migration Service, consultez l’article [Qu’est-ce qu’Azure Database Migration Service ?](./dms-overview.md).
* Pour obtenir des informations sur Azure Database pour PostgreSQL, consultez l’article [Qu’est-ce qu’Azure Database pour PostgreSQL ?](../postgresql/overview.md).
