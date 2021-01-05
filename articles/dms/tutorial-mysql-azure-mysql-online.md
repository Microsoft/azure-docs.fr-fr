---
title: 'Tutoriel : Migrer MySQL en ligne vers Azure Database pour MySQL'
titleSuffix: Azure Database Migration Service
description: Découvrez comment effectuer la migration en ligne d’une instance locale de MySQL vers Azure Database pour MySQL, à l’aide d’Azure Database Migration Service.
services: dms
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 01/08/2020
ms.openlocfilehash: 41c0bd23bbd2d69506a979c5a36ac40f73258f2c
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97605511"
---
# <a name="tutorial-migrate-mysql-to-azure-database-for-mysql-online-using-dms"></a>Tutoriel : Migration de MySQL vers Azure Database pour MySQL en ligne à l’aide de DMS

Vous pouvez utiliser Azure Database Migration Service pour effectuer la migration de bases de données entre une instance locale de MySQL et [Azure Database pour MySQL](../mysql/index.yml), pour un temps d’arrêt minimal. En d’autres termes, la migration peut être effectuée avec un temps d’arrêt minimal de l’application. Dans ce tutoriel, vous allez effectuer la migration de l’exemple de base de données **Employees** à partir d’une instance MySQL 5.7 locale vers Azure Database pour MySQL, à l’aide d’une activité de migration en ligne dans Azure Database Migration Service.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
>
> * Migrez l’exemple de schéma à l’aide de l’utilitaire mysqldump.
> * Créer une instance Azure Database Migration Service.
> * Créer un projet de migration en utilisant Azure Database Migration Service.
> * Exécuter la migration.
> * Surveiller la migration.

> [!NOTE]
> L’utilisation d’Azure Database Migration Service pour effectuer une migration en ligne nécessite la création d’une instance basée sur le niveau tarifaire Premium.

> [!IMPORTANT]
> Pour une expérience de migration optimale, Microsoft recommande de créer une instance Azure Database Migration Service dans la même région Azure que la base de données cible. Le déplacement des données entre les régions ou les zones géographiques peut ralentir le processus de migration et introduire des erreurs.

> [!NOTE]
> Communication sans stéréotype
>
> La diversité et l’inclusion sont au cœur des valeurs de Microsoft. Cet article contient des références au mot _esclave_. Le [guide de style de Microsoft sur la communication sans stéréotype](https://github.com/MicrosoftDocs/microsoft-style-guide/blob/master/styleguide/bias-free-communication.md) le reconnaît comme un mot à exclure. Le mot est utilisé dans cet article pour des raisons de cohérence, car il s’agit du mot qui figure dans le logiciel. Une fois que le mot aura été supprimé du logiciel, cet article sera mis à jour en conséquence.
>


## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez effectuer les opérations suivantes :

* Téléchargez et installez [MySQL Community Edition](https://dev.mysql.com/downloads/mysql/) 5.6 ou 5.7. La version de MySQL locale doit correspondre à la version d’Azure Database pour MySQL. Par exemple, MySQL 5.6 ne peut migrer que vers Azure Database pour MySQL 5.6, et ne peut pas être migré vers 5.7. Les migrations vers ou depuis MySQL 8.0 ne sont pas prises en charge. Les migrations vers ou depuis MySQL 8.0 ne sont pas prises en charge.
* [Créez une instance dans Azure Database pour MySQL](../mysql/quickstart-create-mysql-server-database-using-azure-portal.md). Consultez l’article [Utilisation de MySQL Workbench pour vous connecter et interroger des données](../mysql/connect-workbench.md) pour plus d’informations sur la connexion et la création d’une base de données à l’aide du portail Azure.  
* Créez un réseau virtuel Microsoft Azure pour Azure Database Migration Service à l’aide du modèle de déploiement Azure Resource Manager, qui fournit une connectivité site à site à vos serveurs sources locaux via [ExpressRoute](../expressroute/expressroute-introduction.md) ou un [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Pour plus d’informations sur la création d’un réseau virtuel, consultez la [documentation sur le réseau virtuel](../virtual-network/index.yml), en particulier les articles sur le démarrage rapide, qui fournissent des informations pas à pas.

    > [!NOTE]
    > Pendant la configuration du réseau virtuel, si vous utilisez ExpressRoute avec le peering réseau à Microsoft, ajoutez ces [points de terminaison](../virtual-network/virtual-network-service-endpoints-overview.md) au sous-réseau où doit être provisionné le service :
    >
    > * Point de terminaison de base de données cible (un point de terminaison SQL ou Cosmos DB, par exemple)
    > * Point de terminaison de stockage
    > * Point de terminaison Service Bus
    >
    > Cette configuration est nécessaire car Azure Database Migration Service ne dispose pas d’une connectivité Internet.

* Vérifiez que les règles du groupe de sécurité réseau de votre réseau virtuel ne bloquent pas les ports suivants pour les communications entrantes vers Azure Database Migration Service : 443, 53, 9354, 445, 12000. Pour plus d’informations sur le filtrage du trafic de groupe de sécurité réseau de réseau virtuel, consultez l’article [Filtrer le trafic avec les groupes de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Configurez votre [pare-feu Windows pour accéder au moteur de base de données](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Ouvrez votre pare-feu Windows pour permettre à Azure Database Migration Service d’accéder au serveur MySQL source via le port TCP 3306 (par défaut).
* Lorsque vous utilisez une appliance de pare-feu devant vos bases de données sources, vous devrez peut-être ajouter des règles de pare-feu pour permettre à Azure Database Migration Service d’accéder aux bases de données sources pour la migration.
* Créez une [règle de pare-feu](../azure-sql/database/firewall-configure.md) de niveau serveur pour Azure Database pour MySQL afin de permettre à Azure Database Migration Service d’accéder aux bases de données cibles. Fournissez la plage de sous-réseau du réseau virtuel utilisé pour Azure Database Migration Service.
* Le serveur MySQL source doit être sur une édition MySQL Community prise en charge. Pour déterminer la version de l’instance MySQL, exécutez la commande suivante dans l’utilitaire MySQL ou MySQL Workbench :

    ```
    SELECT @@version;
    ```

* Azure Database pour MySQL ne prend en charge que les tables InnoDB. Pour convertir les tables MyISAM en InnoDB, consultez l’article [Convertir des tables MyISAM en InnoDB](https://dev.mysql.com/doc/refman/5.7/en/converting-tables-to-innodb.html)

* Activez la journalisation binaire dans le fichier my.ini (Windows) ou my.cnf (Unix) dans la base de données source à l’aide de la configuration suivante :

  * **server_id** = 1 ou supérieur (uniquement pour MySQL 5.6)
  * **log-bin** =\<path>> (valable uniquement pour MySQL 5.6)    Par exemple : log-bin = E:\MySQL_logs\BinLog
  * **binlog_format** = row
  * **Expire_logs_days** = 5 (il est recommandé de ne pas utiliser la valeur zéro ; uniquement pour MySQL 5.6)
  * **binlog_row_image** = full (uniquement pour MySQL 5.6)
  * **log_slave_updates** = 1

* L'utilisateur doit avoir le rôle ReplicationAdmin avec les privilèges suivants :

  * **CLIENT DE RÉPLICATION** : obligatoire pour les tâches Traitement des changements uniquement. En d’autres termes, les tâches Chargement complet uniquement n’ont pas besoin de ce privilège.
  * **RÉPLICA DE RÉPLICATION** : obligatoire pour les tâches Traitement des changements uniquement. En d’autres termes, les tâches Chargement complet uniquement n’ont pas besoin de ce privilège.
  * **SUPER** : obligatoire uniquement dans les versions antérieures à MySQL 5.6.6.

## <a name="migrate-the-sample-schema"></a>Migrer l’exemple de schéma

Pour compléter tous les objets de base de données tels que les schémas de table, les index et les procédures stockées, nous devons extraire le schéma à partir de la base de données source et l’appliquer à la base de données. Pour extraire le schéma, vous pouvez utiliser mysqldump avec le paramètre `--no-data`.

En supposant que vous avez l’exemple de base de données des **employés** MySQL dans le système local, la commande pour effectuer la migration du schéma en utilisant mysqldump est :

```
mysqldump -h [servername] -u [username] -p[password] --databases [db name] --no-data > [schema file path]
```

Par exemple :

```
mysqldump -h 10.10.123.123 -u root -p --databases employees --no-data > d:\employees.sql
```

Pour importer le schéma vers la cible Azure Database pour MySQL, exécutez la commande suivante :

```
mysql.exe -h [servername] -u [username] -p[password] [database]< [schema file path]
 ```

Par exemple :

```
mysql.exe -h shausample.mysql.database.azure.com -u dms@shausample -p employees < d:\employees.sql
 ```

Si vous avez des clés étrangères dans votre schéma, la charge initiale et la synchronisation continue de la migration échouent.  Exécutez le script suivant dans MySQL Workbench pour extraire le script de clé étrangère Drop et ajouter le script de clé étrangère.

```sql
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

Exécutez la clé étrangère Drop (qui est la deuxième colonne) dans le résultat de la requête pour supprimer la clé étrangère.

> [!NOTE]
> Azure DMS ne prend pas en charge l’action référentielle CASCADE, qui permet de supprimer ou de mettre à jour automatiquement une ligne correspondante dans la table enfant lorsqu’une ligne est supprimée ou mise à jour dans la table parente. Pour plus d’informations, dans la documentation MySQL, consultez la section Actions référentielles de l’article [Contraintes FOREIGN KEY](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html).
> Azure DMS requiert la suppression des contraintes de clé étrangère sur le serveur de base de données cible pendant le chargement initial des données, et vous ne pouvez pas utiliser les actions référentielles. Si votre charge de travail dépend de la mise à jour d’une table enfant associée par le biais de cette action référentielle, nous vous recommandons d’effectuer [un fichier de sauvegarde et une restauration](../mysql/concepts-migrate-dump-restore.md) à la place. 


> [!IMPORTANT]
> Si vous importez des données à l’aide d’une sauvegarde, supprimez les commandes CREATE DEFINER manuellement ou à l’aide de la commande --skip-definer lors de l’exécution d’une tâche mysqldump. DEFINER requiert des privilèges de superutilisateur pour créer et est limité dans Azure Database pour MySQL.

Si vous avez un déclencheur dans les données (déclencheur d’insertion ou de mise à jour), il appliquera l’intégrité des données dans la cible avant les données répliquées à partir de la source. Nous vous recommandons de désactiver les déclencheurs de toutes les tables au niveau de la cible pendant la migration, puis d’activer les déclencheurs une fois la migration terminée.

Pour désactiver les déclencheurs dans la base de données cible, utilisez la commande suivante :

```
SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
```

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Inscrire le fournisseur de ressources Microsoft.DataMigration

1. Connectez-vous au portail Azure, sélectionnez **Tous les services**, puis **Abonnements**.

   ![Afficher les abonnements au portail](media/tutorial-mysql-to-azure-mysql-online/portal-select-subscriptions.png)

2. Sélectionnez l’abonnement dans lequel vous voulez créer l’instance Azure Database Migration Service, puis sélectionnez **Fournisseurs de ressources**.

    ![Afficher les fournisseurs de ressources](media/tutorial-mysql-to-azure-mysql-online/portal-select-resource-provider.png)

3. Recherchez migration, puis à droite de **Microsoft.DataMigration**, sélectionnez **Inscrire**.

    ![S’inscrire auprès du fournisseur de ressources](media/tutorial-mysql-to-azure-mysql-online/portal-register-resource-provider.png)

## <a name="create-a-dms-instance"></a>Créer une instance de service DMS

1. Dans le portail Azure, sélectionnez **+ Créer une ressource**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Service** dans la liste déroulante.

    ![Place de marché Azure](media/tutorial-mysql-to-azure-mysql-online/portal-marketplace.png)

2. Dans l’écran **Azure Database Migration Service**, sélectionnez **Créer**.

    ![Créer une instance Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-create1.png)
  
3. Dans l’écran **Créer un service de migration**, spécifiez un nom pour le service, l’abonnement, et un réseau virtuel nouveau ou existant.

4. Sélectionnez un réseau virtuel existant ou créez-en un.

    Le réseau virtuel fournit à Azure Database Migration Service un accès au serveur SQL Server source et à l’instance Azure SQL Database cible.

    Pour plus d’informations sur la création d’un réseau virtuel dans le portail Azure, consultez l’article [Créer un réseau virtuel au moyen du portail Azure](../virtual-network/quick-create-portal.md).

5. Sélectionnez un niveau tarifaire.

    Pour plus d’informations sur les coûts et les niveaux de tarification, consultez la [page de tarification](https://aka.ms/dms-pricing).

    ![Configurer des paramètres d’instance Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-settings3.png)

6. Sélectionnez **Créer** pour créer le service.

## <a name="create-a-migration-project"></a>Créer un projet de migration

Une fois le service créé, recherchez-le dans le portail Azure, ouvrez-le, puis créez un projet de migration.

1. Dans le portail Azure, sélectionnez **Tous les services**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Services**.

      ![Localiser toutes les instances Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-search.png)

2. Dans l’écran **Azure Database Migration Services**, recherchez le nom de l’instance Azure Database Migration Service que vous avez créée, puis sélectionnez l’instance.

     ![Localisez votre instance Azure Database Migration Service.](media/tutorial-mysql-to-azure-mysql-online/dms-instance-search.png)

3. Sélectionnez **+ Nouveau projet de migration**.
4. Dans l’écran **Nouveau projet de migration**, spécifiez un nom pour le projet, dans la zone de texte **Type de serveur source**, sélectionnez **MySQL** et, dans la zone de texte **Type de serveur cible**, sélectionnez **AzureDbForMySQL**.
5. Dans la section **Choisir un type d’activité**, sélectionnez **Migration de données en ligne**

    ![Créer un projet Azure Database Migration Service](media/tutorial-mysql-to-azure-mysql-online/dms-create-project4.png)

    > [!NOTE]
    > Une autre possibilité consiste à choisir **Créer un projet uniquement** pour créer le projet de migration à ce stade et exécuter la migration ultérieurement.

6. Sélectionnez **Enregistrer**, notez les conditions requises pour utiliser correctement DMS pour migrer des données, puis sélectionnez **Créer et exécuter une activité**.

## <a name="specify-source-details"></a>Spécifier les détails de la source

1. Dans l’écran **Ajouter les détails de la source**, spécifiez les détails de connexion de l’instance MySQL source.

    ![Écran Ajouter les détails de la source](media/tutorial-mysql-to-azure-mysql-online/dms-add-source-details.png)

## <a name="specify-target-details"></a>Spécifier les détails de la cible

1. Sélectionnez **Enregistrer** puis, dans l’écran **Détails de la cible**, spécifiez les détails de connexion du serveur Azure Database pour MySQL cible, à savoir l’instance Azure Database pour MySQL pré-provisionnée sur laquelle le schéma **Employés** a été déployé à l’aide de mysqldump.

    ![Écran Détails de la cible](media/tutorial-mysql-to-azure-mysql-online/dms-add-target-details.png)

2. Sélectionnez **Enregistrer**, puis dans l’écran **Mapper aux bases de données cibles**, mappez les bases de données source et cible pour la migration.

    Si la base de données cible porte le même nom que la base de données source, Azure Database Migration Service sélectionne la base de données cible par défaut.

    ![Mapper aux bases de données cibles](media/tutorial-mysql-to-azure-mysql-online/dms-map-target-details.png)
   > [!NOTE] 
   > Bien que vous puissiez sélectionner plusieurs bases de données au cours de cette étape, chaque instance d’Azure Database Migration Service prend en charge jusqu’à quatre bases de données pour la migration simultanée. De plus, il existe une limite de deux instances d’Azure Database Migration Service par région dans un abonnement. Par exemple, si vous avez 40 bases de données à migrer, vous pouvez seulement migrer huit d’entre elles simultanément, et uniquement si vous avez créé deux instances d’Azure Database Migration Service.

3. Sélectionnez **Enregistrer**, dans l’écran **Récapitulatif de la migration** pour la zone de texte **Nom de l’activité**, spécifiez un nom pour l’activité de migration, puis examinez le récapitulatif pour vous assurer que les détails de la source et de la cible correspondent à ceux que vous avez spécifiés précédemment.

    ![Résumé de la migration](media/tutorial-mysql-to-azure-mysql-online/dms-migration-summary.png)

## <a name="run-the-migration"></a>Exécuter la migration

* Sélectionnez **Exécuter la migration**.

    La fenêtre d’activité de migration s’affiche, et le champ **État** de l’activité affiche **Initialisation en cours**.

## <a name="monitor-the-migration"></a>Surveiller la migration

1. Dans l’écran d’activité de migration, sélectionnez **Actualiser** pour mettre à jour l’affichage jusqu’à voir **l’état** de la migration s’afficher comme **Terminé**.

     ![État d’activité : Terminé](media/tutorial-mysql-to-azure-mysql-online/dms-activity-completed.png)

2. Sous **Nom de la base de données**, sélectionnez une base de données spécifique pour obtenir l’état de migration des opérations **Chargement complet des données en cours** et **Synchronisation des données incrémentielles**.

    L’opération Charge complète des données affichera l’état de migration de la charge initiale, tandis que l’opération Synchronisation des données incrémentielles indiquera l’état de capture des changements de données (CDC).

     ![État de l’activité : Charge complète terminée](media/tutorial-mysql-to-azure-mysql-online/dms-activity-full-load-completed.png)

     ![État de l’activité : Synchronisation des données incrémentielles](media/tutorial-mysql-to-azure-mysql-online/dms-activity-incremental-data-sync.png)

## <a name="perform-migration-cutover"></a>Effectuer le basculement de la migration

Une fois le chargement complet initial effectué, les bases de données identifiées par le libellé **Prêt pour le basculement**.

1. Lorsque vous êtes prêt à effectuer la migration de base de données, sélectionnez **Démarrer le basculement**.

    ![Démarrer le basculement](media/tutorial-mysql-to-azure-mysql-online/dms-start-cutover.png)

2. Veillez à arrêter toutes les transactions entrantes pour la base de données source ; attendez que le compteur **Changements en attente** indique la valeur **0**.
3. Sélectionnez **Confirmer**, puis **Appliquer**.
4. Lorsque la migration de base de données présente l’état **Terminé**, connectez vos applications à la nouvelle base de données Azure SQL cible.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les limitations et problèmes connus lors de l’exécution de migrations en ligne vers Azure Database pour MySQL, consultez l’article [Known issues and workarounds with Azure Database for MySQL online migrations](known-issues-azure-mysql-online.md) (Problèmes connus et solutions de contournement pour les migrations Azure Database pour MySQL en ligne).
* Pour plus d’informations sur Azure Database Migration Service, consultez l’article [Qu’est-ce qu’Azure Database Migration Service ?](./dms-overview.md).
* Pour plus d’informations sur Azure Database pour MySQL, voir l’article [Qu’est-ce qu’Azure Database pour MySQL ?](../mysql/overview.md).