---
title: 'Tutoriel : Migrer les services Bureau à distance MySQL en ligne vers Azure Database pour MySQL'
titleSuffix: Azure Database Migration Service
description: Découvrez comment effectuer une migration en ligne de RDS MySQL vers Azure Database pour MySQL à l’aide d’Azure Database Migration Service.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 06/09/2020
ms.openlocfilehash: b8d5c763b68a9f69add14ab8430c117e5705a515
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94955087"
---
# <a name="tutorial-migrate-rds-mysql-to-azure-database-for-mysql-online-using-dms"></a>Tutoriel : Migrer RDS MySQL vers Azure Database pour MySQL en ligne à l’aide de DMS

Vous pouvez utiliser Azure Database Migration Service pour migrer des bases de données à partir d’une instance RDS MySQL vers [Azure Database pour MySQL](../mysql/index.yml) tandis que la base de données source reste en ligne pendant la migration. En d’autres termes, la migration peut être effectuée avec un temps d’arrêt minimal de l’application. Dans ce didacticiel, vous allez migrer l’exemple de base de données **Employés** à partir d’une instance RDS MySQL vers Azure Database pour MySQL à l’aide de l’activité de migration en ligne dans Azure Database Migration Service.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
>
> * Migrer l’exemple de schéma à l’aide des utilitaires mysqldump et mysql.
> * Créer une instance Azure Database Migration Service.
> * Créer un projet de migration en utilisant Azure Database Migration Service.
> * Exécuter la migration.
> * Surveiller la migration.

> [!NOTE]
> L’utilisation d’Azure Database Migration Service pour effectuer une migration en ligne nécessite la création d’une instance basée sur le niveau tarifaire Premium. Pour plus d’informations, consultez la page de [tarification](https://azure.microsoft.com/pricing/details/database-migration/) du service Azure Database Migration Service.

> [!IMPORTANT]
> Pour une expérience de migration optimale, Microsoft vous recommande de créer une instance Azure Database Migration Service dans la même région Azure que la base de données cible. Le déplacement des données entre les régions ou les zones géographiques peut ralentir le processus de migration et introduire des erreurs.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

Cet article décrit comment effectuer une migration en ligne d’une instance RDS MySQL vers Azure Database pour MySQL.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez effectuer les opérations suivantes :

* Assurez-vous que le serveur MySQL source exécute une édition Communauté de MySQL prise en charge. Pour déterminer la version de votre instance MySQL, dans l’utilitaire mysql ou MySQL Workbench, exécutez la commande suivante :

    ```
    SELECT @@version;
    ```

    Pour plus d’informations, voir l’article [Versions prises en charge d’Azure Database pour MySQL pour MySQL](../mysql/concepts-supported-versions.md).

* Téléchargez et installez l’exemple de base de données [MySQL **Employés**](https://dev.mysql.com/doc/employee/en/employees-installation.html).
* Créez une instance d’[Azure Database pour MySQL](../mysql/quickstart-create-mysql-server-database-using-azure-portal.md).
* Créez un réseau virtuel Microsoft Azure pour Azure Database Migration Service à l’aide du modèle de déploiement Azure Resource Manager, qui fournit une connectivité site à site à vos serveurs sources locaux via [ExpressRoute](../expressroute/expressroute-introduction.md) ou un [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Pour plus d’informations sur la création d’un réseau virtuel, consultez la [documentation sur le réseau virtuel](../virtual-network/index.yml), en particulier les articles sur le démarrage rapide, qui fournissent des informations pas à pas.
* Vérifiez que les règles du groupe de sécurité réseau de votre réseau virtuel ne bloquent pas les ports suivants pour les communications entrantes vers Azure Database Migration Service : 443, 53, 9354, 445 et 12000. Pour plus d’informations sur le filtrage du trafic de groupe de sécurité réseau de réseau virtuel, consultez l’article [Filtrer le trafic avec les groupes de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Configurez votre [pare-feu Windows](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access) (ou votre pare-feu Linux) pour permettre l’accès au moteur de base de données. Pour MySQL Server, autorisez le port 3306 pour la connectivité.

> [!NOTE]
> Azure Database pour MySQL ne prend en charge que les tables InnoDB. Pour savoir comment convertir des tables MyISAM en InnoDB, voir l’article [Convertir des tables MyISAM en InnoDB](https://dev.mysql.com/doc/refman/5.7/en/converting-tables-to-innodb.html).

### <a name="set-up-aws-rds-mysql-for-replication"></a>Configurer AWS RDS MySQL pour la réplication

1. Pour créer un groupe de paramètres, suivez les instructions fournies par AWS dans la section **Format de journalisation binaire** de l’article [Fichiers journaux de base de données MySQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.Concepts.MySQL.html).
2. Créez un groupe de paramètres avec la configuration suivante :
    * log_bin = ON
    * binlog_format = row
    * binlog_checksum = NONE
3. Enregistrez le nouveau groupe de paramètres.
4. Associez le nouveau groupe de paramètres à l’instance MySQL des Services Bureau à distance. Un redémarrage peut être nécessaire.
5. Une fois le groupe de paramètres en place, connectez-vous à l’instance MySQL et [définissez la rétention de binlog](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/mysql_rds_set_configuration.html#mysql_rds_set_configuration-usage-notes.binlog-retention-hours) sur au moins 5 jours.
```
call mysql.rds_set_configuration('binlog retention hours', 120);
```

## <a name="migrate-the-schema"></a>Migrer le schéma

1. Extrayez le schéma de la base de données source et appliquez-le à la base de données cible pour accomplir la migration de tous les objets de base de données, tels que les schémas de table, les index et les procédures stockées.

    La manière la plus simple de migrer uniquement le schéma consiste à utiliser mysqldump avec le paramètre --no-data. La commande pour migrer le schéma est la suivante :

    ```
    mysqldump -h [servername] -u [username] -p[password] --databases [db name] --no-data > [schema file path]
    ```
    
    Par exemple, pour vider un fichier de schéma pour la base de données **Employés**, utilisez la commande suivante :
    
    ```
    mysqldump -h 10.10.123.123 -u root -p --databases employees --no-data > d:\employees.sql
    ```

2. Importez le schéma dans le service cible, à savoir Azure Database pour MySQL. Pour restaurer le fichier de vidage du schéma, exécutez la commande suivante :

    ```
    mysql.exe -h [servername] -u [username] -p[password] [database]< [schema file path]
    ```

    Par exemple, pour importer le schéma pour la base de données **Employés** :

    ```
    mysql.exe -h shausample.mysql.database.azure.com -u dms@shausample -p employees < d:\employees.sql
    ```

3. Si vous avez des clés étrangères dans votre schéma, la charge initiale et la synchronisation continue de la migration échouent. Pour extraire le script drop foreign key et ajouter un script de clé étrangère à la destination (Azure Database pour MySQL), exécutez le script suivant dans MySQL Workbench :

    ```
    SET group_concat_max_len = 8192;
        SELECT SchemaName, GROUP_CONCAT(DropQuery SEPARATOR ';\n') as DropQuery, GROUP_CONCAT(AddQuery SEPARATOR ';\n') as AddQuery
        FROM
        (SELECT
        KCU.REFERENCED_TABLE_SCHEMA as SchemaName,
                    KCU.TABLE_NAME,
                    KCU.COLUMN_NAME,
                    CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' DROP FOREIGN KEY ', KCU.CONSTRAINT_NAME) AS DropQuery,
        CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' ADD CONSTRAINT ', KCU.CONSTRAINT_NAME, ' FOREIGN KEY (`', KCU.COLUMN_NAME, '`) REFERENCES `', KCU.REFERENCED_TABLE_NAME, '` (`', KCU.REFERENCED_COLUMN_NAME, '`) ON UPDATE ',RC.UPDATE_RULE, ' ON DELETE ',RC.DELETE_RULE) AS AddQuery
                    FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE KCU, information_schema.REFERENTIAL_CONSTRAINTS RC
                    WHERE
                      KCU.CONSTRAINT_NAME = RC.CONSTRAINT_NAME
                      AND KCU.REFERENCED_TABLE_SCHEMA = RC.UNIQUE_CONSTRAINT_SCHEMA
      AND KCU.REFERENCED_TABLE_SCHEMA = 'SchemaName') Queries
      GROUP BY SchemaName;
    ```

4. Exécutez la suppression de clé étrangère (deuxième colonne) dans le résultat de la requête pour supprimer la clé étrangère.

> [!NOTE]
> Azure DMS ne prend pas en charge l’action référentielle CASCADE, qui permet de supprimer ou de mettre à jour automatiquement une ligne correspondante dans la table enfant lorsqu’une ligne est supprimée ou mise à jour dans la table parente. Pour plus d’informations, dans la documentation MySQL, consultez la section Actions référentielles de l’article [Contraintes FOREIGN KEY](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html).
> Azure DMS requiert la suppression des contraintes de clé étrangère sur le serveur de base de données cible pendant le chargement initial des données, et vous ne pouvez pas utiliser les actions référentielles. Si votre charge de travail dépend de la mise à jour d’une table enfant associée par le biais de cette action référentielle, nous vous recommandons d’effectuer [un vidage et une restauration](../mysql/concepts-migrate-dump-restore.md) à la place. 

5. Si vous avez un déclencheur (d’insertion ou de mise à jour) dans les données, il appliquera l’intégrité des données dans la cible avant de répliquer les données de la source. Nous vous recommandons de désactiver les déclencheurs dans toutes les tables *côté cible* pendant la migration, puis d’activer les déclencheurs une fois la migration terminée.

    Pour désactiver les déclencheurs dans la base de données cible :

    ```
    select concat ('alter table ', event_object_table, ' disable trigger ', trigger_name)
    from information_schema.triggers;
    ```

6. S’il existe des occurrences du type de données ENUM dans des tables, nous vous recommandons de mettre à jour temporairement le type de données « character varying » dans la table cible. Une fois la réplication des données terminée, rétablissez le type de données ENUM.

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Inscrire le fournisseur de ressources Microsoft.DataMigration

1. Connectez-vous au portail Azure, sélectionnez **Tous les services**, puis **Abonnements**.

   ![Afficher les abonnements au portail](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/portal-select-subscription1.png)

2. Sélectionnez l’abonnement dans lequel vous voulez créer l’instance Azure Database Migration Service, puis sélectionnez **Fournisseurs de ressources**.

    ![Afficher les fournisseurs de ressources](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/portal-select-resource-provider.png)

3. Recherchez migration, puis à droite de **Microsoft.DataMigration**, sélectionnez **Inscrire**.

    ![S’inscrire auprès du fournisseur de ressources](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/portal-register-resource-provider.png)

## <a name="create-an-instance-of-azure-database-migration-service"></a>Créer une instance Azure Database Migration Service

1. Dans le portail Azure, sélectionnez **+ Créer une ressource**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Service** dans la liste déroulante.

    ![Place de marché Azure](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/portal-marketplace.png)

2. Dans l’écran **Azure Database Migration Service**, sélectionnez **Créer**.

    ![Créer une instance Azure Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-create1.png)
  
3. Dans l’écran **Créer un service de migration**, spécifiez un nom pour le service, l’abonnement, et un réseau virtuel nouveau ou existant.

4. Sélectionnez l’emplacement dans lequel vous souhaitez créer l’instance Azure Database Migration Service.

5. Sélectionnez un réseau virtuel existant ou créez-en un.

    Le réseau virtuel fournit à Azure Database Migration Service un accès à l’instance MySQL source et à l’instance Azure Database pour MySQL cible.

    Pour plus d’informations sur la création d’un réseau virtuel dans le portail Azure, consultez l’article [Créer un réseau virtuel au moyen du portail Azure](../virtual-network/quick-create-portal.md).

6. Sélectionnez un niveau tarifaire. Pour cette migration en ligne, sélectionnez le niveau tarifaire Premium : 4vCores.

    ![Configurer des paramètres d’instance Azure Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-settings3.png)

7. Sélectionnez **Créer** pour créer le service.

## <a name="create-a-migration-project"></a>Créer un projet de migration

Une fois le service créé, recherchez-le dans le portail Azure, ouvrez-le, puis créez un projet de migration.

1. Dans le portail Azure, sélectionnez **Tous les services**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Services**.

      ![Rechercher toutes les instances Azure Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-search.png)

2. Dans l’écran **Azure Database Migration Services**, recherchez le nom de l’instance Azure Database Migration Service que vous avez créée, puis sélectionnez l’instance.

     ![Rechercher votre instance Azure Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-instance-search.png)

3. Sélectionnez **+ Nouveau projet de migration**.
4. Dans l’écran **Nouveau projet de migration**, spécifiez un nom pour le projet dans la zone de texte **Type de serveur source**, sélectionnez **MySQL**, puis, dans la zone de texte **Type de serveur cible**, sélectionnez **AzureDbForMySQL**.
5. Dans la section **Choisir un type d’activité**, sélectionnez **Migration de données en ligne**.

    > [!IMPORTANT]
    > Veillez à sélectionner **Migration des données en ligne** ; les migrations hors connexion ne sont pas prises en charge pour ce scénario.

    ![Créer un projet Azure Database Migration Service](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-create-project6.png)

    > [!NOTE]
    > Une autre possibilité consiste à choisir **Créer un projet uniquement** pour créer le projet de migration à ce stade et exécuter la migration ultérieurement.

6. Sélectionnez **Enregistrer**.

7. Sélectionnez **Créer et exécuter une activité** pour créer le projet et exécuter l’activité de migration.

    > [!NOTE]
    > Notez les conditions préalables requises pour configurer la migration en ligne dans le panneau de création de projet.

## <a name="specify-source-details"></a>Spécifier les détails de la source

* Dans l’écran **Détails de la source de migration**, spécifiez les détails de connexion pour l’instance MySQL source.

   ![Détails de la source](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-source-details5.png)

## <a name="specify-target-details"></a>Spécifier les détails de la cible

1. Sélectionnez **Enregistrer**, puis, dans l’écran **Détails de la cible**, spécifiez les détails de connexion pour le serveur Azure Database pour MySQL cible qui est préprovisionné et a le schéma **Employés** déployé à l’aide de MySQLDump.

    ![Sélectionner la cible](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-select-target5.png)

2. Sélectionnez **Enregistrer**, puis dans l’écran **Mapper aux bases de données cibles**, mappez les bases de données source et cible pour la migration.

    Si la base de données cible porte le même nom que la base de données source, Azure Database Migration Service sélectionne la base de données cible par défaut.

    ![Mapper aux bases de données cibles](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-map-targets-activity5.png)

3. Sélectionnez **Enregistrer**, dans l’écran **Récapitulatif de la migration** pour la zone de texte **Nom de l’activité**, spécifiez un nom pour l’activité de migration, puis examinez le récapitulatif pour vous assurer que les détails de la source et de la cible correspondent à ceux que vous avez spécifiés précédemment.

    ![Résumé de la migration](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-migration-summary2.png)

## <a name="run-the-migration"></a>Exécuter la migration

* Sélectionnez **Exécuter la migration**.

    La fenêtre d’activité de migration s’affiche, et le champ **État** de l’activité présente la valeur **Initialisation en cours**.

## <a name="monitor-the-migration"></a>Surveiller la migration

1. Dans l’écran d’activité de migration, sélectionnez **Actualiser** pour mettre à jour l’affichage jusqu’à ce que le champ **État** de la migration prenne la valeur **En cours d’exécution**.

    ![État de l’activité – En cours d’exécution](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-activity-status4.png)

2. Sous **NOM DE LA BASE DE DONNÉES**, sélectionnez une base de données spécifique pour obtenir l’état de migration des opérations **Full data load** (Charge complète des données) et **Incremental data sync** (Synchronisation incrémentielle des données).

    L’opération **Full data load** (Charge complète des données) affiche l’état initial de migration de la charge, tandis que l’opération **Incremental data sync** (Synchronisation incrémentielle des données) indique l’état de capture des changements de données (CDC).

    ![Écran d’inventaire – charge complète des données](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-inventory-full-load.png)

    ![Écran d’inventaire – synchronisation incrémentielle des données](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-inventory-incremental.png)

## <a name="perform-migration-cutover"></a>Effectuer le basculement de la migration

Une fois le chargement complet initial effectué, les bases de données sont marquées de la mention **Ready to Cutover** (Prêt pour le basculement).

1. Lorsque vous êtes prêt à effectuer la migration de base de données, sélectionnez **Démarrer le basculement**.

    ![Démarrer le basculement](media/tutorial-rds-mysql-server-azure-db-for-mysql-online/dms-inventory-start-cutover.png)

2. Veillez à arrêter toutes les transactions entrantes pour la base de données source ; attendez que le compteur **Changements en attente** indique la valeur **0**.
3. Sélectionnez **Confirmer**, puis **Appliquer**.
4. Lorsque l’état de migration de la base de données est **Terminé**, connectez vos applications à la nouvelle base de données Azure Database pour MySQL cible.

Votre migration en ligne d’une instance locale de MySQL vers Azure Database pour MySQL est maintenant terminée.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Azure Database Migration Service, consultez l’article [Qu’est-ce qu’Azure Database Migration Service ?](./dms-overview.md).
* Pour plus d’informations sur Azure Database pour MySQL, voir l’article [Qu’est-ce qu’Azure Database pour MySQL ?](../mysql/overview.md).
* Si vous avez d’autres questions, envoyez un courrier pour [Poser des questions sur les migrations de base de données Azure](mailto:AskAzureDatabaseMigrations@service.microsoft.com).