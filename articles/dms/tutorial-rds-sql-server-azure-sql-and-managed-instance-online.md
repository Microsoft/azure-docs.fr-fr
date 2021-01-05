---
title: 'Tutoriel : Migrer RDS SQL Server en ligne vers SQL Database'
titleSuffix: Azure Database Migration Service
description: Apprenez à effectuer une migration en ligne de RDS SQL Server vers une base de données unique Azure SQL Database ou vers Azure SQL Database Managed Instance à l’aide d’Azure Database Migration Service.
services: dms
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 01/08/2020
ms.openlocfilehash: 249667dfa8c0491027f0244d4aa5e49d19399ab0
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94955036"
---
# <a name="tutorial-migrate-rds-sql-server-to-azure-sql-database-or-an-azure-sql-managed-instance-online-using-dms"></a>Tutoriel : Migrer en ligne RDS SQL Server vers Azure SQL Database ou une instance gérée SQL Azure à l’aide de DMS

Vous pouvez utiliser Azure Database Migration Service pour effectuer la migration des bases de données d’une instance RDS SQL Server vers [Azure SQL Database](/azure/sql-database/) ou une [instance gérée SQL Azure](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md) avec un temps d’arrêt minimal. Dans ce tutoriel, vous migrez la base de données **Adventureworks2012** restaurée sur une instance RDS SQL Server de SQL Server 2012 (ou version ultérieure) vers SQL Database ou une instance gérée SQL à l’aide d’Azure Database Migration Service.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Créer une base de données dans Azure SQL Database ou SQL Managed Instance 
> * Migrer l’exemple de schéma à l’aide de l’Assistant Migration des données.
> * Créer une instance Azure Database Migration Service.
> * Créer un projet de migration en utilisant Azure Database Migration Service.
> * Exécuter la migration.
> * Surveiller la migration.
> * Télécharger un rapport de migration.

> [!NOTE]
> Effectuer une migration en ligne à l’aide d’Azure Database Migration Service nécessite la création d’une instance basée sur le niveau tarifaire Premium. Pour plus d’informations, consultez la page de [tarification](https://azure.microsoft.com/pricing/details/database-migration/) du service Azure Database Migration Service.

> [!IMPORTANT]
> Pour une expérience de migration optimale, Microsoft vous recommande de créer une instance Azure Database Migration Service dans la même région Azure que la base de données cible. Le déplacement des données entre les régions ou les zones géographiques peut ralentir le processus de migration et introduire des erreurs.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

Cet article décrit le processus de migration en ligne de RDS SQL Server vers SQL Database ou SQL Managed Instance.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez effectuer les opérations suivantes :

* Créez une [base de données RDS SQL Server](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.SQLServer.html).
* [Créez une base de données dans Azure SQL Database sur le portail Azure](../azure-sql/database/single-database-create-quickstart.md) ou [créez une base de données dans SQL Managed Instance](../azure-sql/managed-instance/instance-create-quickstart.md), puis créez une base de données vide nommée **AdventureWorks2012**. 
* Téléchargez et installez [l’Assistant Migration de données](https://www.microsoft.com/download/details.aspx?id=53595) (DMA) v3.3 ou version ultérieure.
* Créer un réseau virtuel Microsoft Azure pour Azure Database Migration Service à l’aide du modèle de déploiement Azure Resource Manager. Si vous effectuez la migration vers une instance gérée SQL, veillez à créer l’instance DMS sur le même réseau virtuel que celui utilisé pour l’instance gérée SQL, mais dans un autre sous-réseau.  Si vous utilisez un autre réseau virtuel pour DMS, vous devrez créer un peering entre les deux réseaux virtuels. Pour plus d’informations sur la création d’un réseau virtuel, consultez la [documentation sur le réseau virtuel](../virtual-network/index.yml), en particulier les articles sur le démarrage rapide, qui fournissent des informations pas à pas.

    > [!NOTE]
    > Pendant la configuration du réseau virtuel, si vous utilisez ExpressRoute avec le peering réseau à Microsoft, ajoutez ces [points de terminaison](../virtual-network/virtual-network-service-endpoints-overview.md) au sous-réseau où doit être provisionné le service :
    >
    > * Point de terminaison de base de données cible (un point de terminaison SQL ou Cosmos DB, par exemple)
    > * Point de terminaison de stockage
    > * Point de terminaison Service Bus
    >
    > Cette configuration est nécessaire, car Azure Database Migration Service ne dispose pas d’une connectivité Internet. 

* Vérifiez que les règles du groupe de sécurité réseau de votre réseau virtuel ne bloquent pas les ports suivants pour les communications entrantes vers Azure Database Migration Service : 443, 53, 9354, 445, 12000. Pour plus d’informations sur le filtrage du trafic de groupe de sécurité réseau de réseau virtuel, consultez l’article [Filtrer le trafic avec les groupes de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Configurez votre [pare-feu Windows pour accéder au moteur de base de données](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Ouvrez votre Pare-feu Windows pour permettre à Azure Database Migration Service d’accéder au serveur SQL Server source, par défaut le port TCP 1433.
* Pour SQL Database, créez une [règle de pare-feu](../azure-sql/database/firewall-configure.md) au niveau du serveur pour qu’Azure Database Migration Service puisse accéder à la base de données cible. Fournissez la plage de sous-réseau du réseau virtuel utilisé pour Azure Database Migration Service.
* Vérifiez que les informations d’identification utilisées pour la connexion à l’instance RDS SQL Server source sont associées à un compte membre du rôle serveur « Processadmin » et membre des rôles de base de données « db_owner » pour toutes les bases de données à migrer.
* Vérifiez que les informations d’identification utilisées pour la connexion à la base de données cible comportent l’autorisation CONTROL DATABASE pour la base de données cible dans SQL Database, ainsi qu’un membre du rôle sysadmin en cas de migration vers une base de données située dans SQL Managed Instance.
* La version de l’instance RDS SQL Server source doit être SQL Server 2012 ou ultérieur. Pour déterminer la version que votre instance SQL Server exécute, consultez l’article [Comment faire pour déterminer la version, l’édition et le niveau de mise à jour de SQL Server et ses composants](https://support.microsoft.com/help/321185/how-to-determine-the-version-edition-and-update-level-of-sql-server-an).
* Activez la fonctionnalité CDC (capture des changements de données) sur la base de données RDS SQL Server ainsi que sur toutes les tables utilisateur sélectionnées pour la migration.
    > [!NOTE]
    > Vous pouvez utiliser le script ci-dessous pour activer CDC sur une base de données RDS SQL Server.
    ```
    exec msdb.dbo.rds_cdc_enable_db 'AdventureWorks2012'
    ```
    > Vous pouvez utiliser le script ci-dessous pour activer CDC sur toutes les tables.
    ```
    use <Database name>
    go
    exec sys.sp_cdc_enable_table 
    @source_schema = N'Schema name', 
    @source_name = N'table name', 
    @role_name = NULL, 
    @supports_net_changes = 1 --for PK table 1, non PK tables 0
    GO
    ```
* Désactivez les déclencheurs de base de données dans la base de données cible.
    > [!NOTE]
    > Pour rechercher les déclencheurs de base de données dans la base de données cible, exécutez la requête suivante :
    ```
    Use <Database name>
    go
    select * from sys.triggers
    DISABLE TRIGGER (Transact-SQL)
    ```
    Pour plus d’informations, consultez l’article [DISABLE TRIGGER (Transact-SQL)](/sql/t-sql/statements/disable-trigger-transact-sql?view=sql-server-2017).

## <a name="migrate-the-sample-schema"></a>Migrer l’exemple de schéma
Utilisez l’Assistant Migration de données pour effectuer la migration du schéma.

> [!NOTE]
> Avant de créer un projet de migration dans l’Assistant Migration de données, vérifiez que vous avez déjà provisionné une base de données dans SQL Database ou SQL Managed Instance, comme indiqué dans les prérequis. Pour les besoins de ce tutoriel, le nom de la base de données est **AdventureWorks2012**. Cependant, vous pouvez indiquer le nom de votre choix.

Pour effectuer la migration du schéma **AdventureWorks2012**, effectuez les étapes suivantes :

1. Dans Data Migration Assistant, sélectionnez l’icône Nouveau (+), puis sous **Type de projet**, sélectionnez **Migration**.
2. Spécifiez un nom de projet, dans la zone de texte **Type de serveur source**, sélectionnez **SQL Server** puis, dans la zone de texte **Type de serveur cible**, sélectionnez **Azure SQL Database**.

    > [!NOTE]
    > Pour le type de serveur cible, sélectionnez **Azure SQL Database** pour effectuer une migration à la fois vers Azure SQL Database et vers SQL Managed Instance.

3. Sous **Étendue de la migration**, sélectionnez **Schéma uniquement**.

    Une fois les étapes précédentes effectuées, l’interface DMA doit présenter l’aspect illustré ci-après :

    ![Créer un projet d’Assistant Migration des données](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-create-project.png)

4. Sélectionnez **Créer** pour créer le projet.
5. Dans DMA, spécifiez les détails de connexion source de votre serveur SQL Server, sélectionnez **Se connecter**, puis sélectionnez la base de données **AdventureWorks2012**.

    ![Détails de connexion source de l’Assistant Migration des données](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-source-connect.png)

6. Sélectionnez **Suivant**, sous **Se connecter au serveur cible**, indiquez les détails de la connexion cible pour la base de données SQL Database ou SQL Managed Instance, sélectionnez **Se connecter**, puis sélectionnez la base de données **AdventureWorksAzure** que vous avez préprovisionnée.

    ![Détails de connexion cible de l’Assistant Migration des données](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-target-connect.png)

7. Sélectionnez **Suivant** pour accéder à l’écran **Sélectionner des objets** dans lequel vous pouvez spécifier les objets de schéma dans la base de données **AdventureWorks2012** à déployer.

    Par défaut, tous les objets sont sélectionnés.

    ![Générer des scripts SQL](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-assessment-source.png)

8. Sélectionnez **Générer un script SQL** pour créer les scripts SQL, puis recherchez d’éventuelles erreurs dans les scripts.

    ![Script de schéma](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-schema-script.png)

9. Sélectionnez **Déployer le schéma** pour déployer le schéma, puis, une fois celui-ci déployé, recherchez d’éventuelles anomalies sur la cible.

    ![Déployer le schéma](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dma-schema-deploy.png)

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Inscrire le fournisseur de ressources Microsoft.DataMigration

1. Connectez-vous au portail Azure, sélectionnez **Tous les services**, puis **Abonnements**.

   ![Afficher les abonnements au portail](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/portal-select-subscription1.png)

2. Sélectionnez l’abonnement dans lequel vous souhaitez créer l’instance Azure Database Migration Service, puis sélectionnez **Fournisseurs de ressources**.

    ![Afficher les fournisseurs de ressources](media/tutorial-sql-server-to-azure-sql-online/portal-select-resource-provider.png)

3. Recherchez migration, puis à droite de **Microsoft.DataMigration**, sélectionnez **Inscrire**.

    ![S’inscrire auprès du fournisseur de ressources](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/portal-register-resource-provider.png)    

## <a name="create-an-instance"></a>Créer une instance

1. Dans le portail Azure, sélectionnez **+ Créer une ressource**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Service** dans la liste déroulante.

    ![Place de marché Azure](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/portal-marketplace.png)

2. Dans l’écran **Azure Database Migration Service**, sélectionnez **Créer**.

    ![Créer une instance Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create1.png)
  
3. Dans l’écran **Créer un service de migration**, spécifiez un nom pour le service, l’abonnement, et un réseau virtuel nouveau ou existant.

4. Sélectionnez l’emplacement dans lequel vous souhaitez créer l’instance Azure Database Migration Service. 

5. Sélectionnez un réseau virtuel existant ou créez-en un.

    Le réseau virtuel fournit à Azure Database Migration Service un accès au serveur SQL Server source, ainsi qu’à la base de données SQL Database ou SQL Managed Instance cible.

    Pour plus d’informations sur la création d’un réseau virtuel dans le portail Azure, consultez l’article [Créer un réseau virtuel au moyen du portail Azure](../virtual-network/quick-create-portal.md).

6. Sélectionnez un niveau tarifaire. Pour cette migration en ligne, veillez à sélectionner le niveau tarifaire Premium.

    Pour plus d’informations sur les coûts et les niveaux de tarification, consultez la [page de tarification](https://aka.ms/dms-pricing).

     ![Configurer des paramètres d’instance Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-settings3.png)

7. Sélectionnez **Créer** pour créer le service.

## <a name="create-a-migration-project"></a>Créer un projet de migration

Une fois le service créé, recherchez-le dans le portail Azure, ouvrez-le, puis créez un projet de migration.

1. Dans le portail Azure, sélectionnez **Tous les services**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Services**.

      ![Rechercher toutes les instances Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-search.png)

2. Dans l’écran **Azure Database Migration Services**, recherchez le nom de l’instance Azure Database Migration Service que vous avez créée, puis sélectionnez l’instance.

     ![Rechercher votre instance Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-instance-search.png)

3. Sélectionnez **+ Nouveau projet de migration**.
4. Dans l’écran **Nouveau projet de migration**, spécifiez un nom pour le projet. Dans la zone de texte **Type de serveur source**, sélectionnez **AWS RDS pour SQL Server**, puis dans la zone de texte **Type de serveur cible**, sélectionnez **Azure SQL Database**.

    > [!NOTE]
    > Pour le type de serveur cible, sélectionnez **Azure SQL Database** pour effectuer une migration à la fois vers SQL Database et vers SQL Managed Instance.

5. Dans la section **Choisir un type d’activité**, sélectionnez **Migration de données en ligne**.

    > [!IMPORTANT]
    > Veillez à sélectionner **Migration des données en ligne** ; les migrations hors connexion ne sont pas prises en charge pour ce scénario.

    ![Créer un projet Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create-project4.png)

    > [!NOTE]
    > Une autre possibilité consiste à choisir **Créer un projet uniquement** pour créer le projet de migration à ce stade et exécuter la migration ultérieurement.

6. Sélectionnez **Enregistrer**.

7. Sélectionnez **Créer et exécuter une activité** pour créer le projet et exécuter l’activité de migration.

    ![Création et exécution d’une activité Azure Database Migration Service](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-create-and-run-activity1.png)

## <a name="specify-source-details"></a>Spécifier les détails de la source

1. Dans l’écran **Détails de la source de migration**, spécifiez les détails de connexion de l’instance SQL Server source.

    Veillez à utiliser un nom de domaine complet pour le nom de l’instance SQL Server source.

2. Si vous n’avez pas installé de certificat approuvé sur votre serveur source, cochez la case **Faire confiance au certificat de serveur**.

    Quand aucun certificat approuvé n’est installé, SQL Server génère un certificat auto-signé au démarrage de l’instance. Ce certificat permet de chiffrer les informations d’identification des connexions clientes.

    > [!CAUTION]
    > Les connexions TLS chiffrées à l’aide d’un certificat auto-signé n’offrent pas de sécurité renforcée. Elles sont vulnérables aux attaques de l’intercepteur. Ne faites jamais confiance à une connexion TLS utilisant des certificats autosignés dans un environnement de production ou sur des serveurs connectés à Internet.

   ![Détails de la source](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-source-details3.png)

## <a name="specify-target-details"></a>Spécifier les détails de la cible

1. Sélectionnez **Enregistrer**, puis, dans l’écran **Détails de la cible de migration**, spécifiez les détails de connexion pour la base de données cible dans Azure.

    ![Sélectionner la cible](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-select-target3.png)

2. Sélectionnez **Enregistrer**, puis dans l’écran **Mapper aux bases de données cibles**, mappez les bases de données source et cible pour la migration.

    Si la base de données cible porte le même nom que la base de données source, Azure Database Migration Service sélectionne la base de données cible par défaut.

    ![Mapper aux bases de données cibles](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-map-targets-activity4.png)

3. Sélectionnez **Enregistrer**, dans l’écran **Sélectionner des tables**, développez la liste des tables et passez en revue la liste des champs concernés.

    Azure Database Migration Service sélectionne automatiquement toutes les tables sources vides qui se trouvent dans la base de données cible. Si vous souhaitez migrer à nouveau des tables contenant déjà des données, vous devez les sélectionner explicitement dans cet écran.

    ![Sélectionner des tables](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-configure-setting-activity4.png)

4. Sélectionnez **Enregistrer** après avoir défini les **Paramètres avancés de migration en ligne**.

    | Paramètre | Description |
    | ------------- | ------------- |
    | **Nombre maximal de tables à charger en parallèle** | Spécifie le nombre de tables que DMS exécute en parallèle durant la migration. La valeur par défaut est 5, mais vous pouvez définir une valeur optimale pour répondre à des besoins de migration spécifiques en fonction des migrations POC. |
    | **Quand la table source est tronquée** | Spécifie si DMS tronque la table cible durant la migration. Ce paramètre peut être utile si une ou plusieurs tables sont tronquées pendant le processus de migration. |
    | **Configurer les paramètres des données LOB** | Spécifie si DMS migre des données LOB illimitées, ou s’il limite les données LOB migrées à une taille spécifique.  Quand il existe une limite concernant les données LOB migrées, toutes les données LOB qui dépassent cette limite sont tronquées. Pour les migrations de production, il est recommandé de sélectionner **Autoriser une taille LOB illimitée** afin d’éviter la perte de données. Quand vous autorisez une taille LOB illimitée, cochez la case **Migrez les données LOB dans un seul bloc quand la taille LOB est inférieure à la taille (Ko) ci-dessous** pour améliorer les performances. |

    ![Définir les paramètres avancés de migration en ligne](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-advanced-online-migration-settings.png)

5. Sélectionnez **Enregistrer**, dans l’écran **Récapitulatif de la migration** pour la zone de texte **Nom de l’activité**, spécifiez un nom pour l’activité de migration, puis examinez le récapitulatif pour vous assurer que les détails de la source et de la cible correspondent à ceux que vous avez spécifiés précédemment.

    ![Résumé de la migration](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-migration-summary.png)

## <a name="run-the-migration"></a>Exécuter la migration

* Sélectionnez **Exécuter la migration**.

    La fenêtre d’activité de migration s’affiche, et le champ **État** de l’activité présente la valeur **Initialisation en cours**.

    ![État de l’activité - Initialisation en cours](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-activity-status2.png)

## <a name="monitor-the-migration"></a>Surveiller la migration

1. Dans l’écran d’activité de migration, sélectionnez **Actualiser** pour mettre à jour l’affichage jusqu’à ce que le champ **État** de la migration prenne la valeur **En cours d’exécution**.

2. Cliquez sur une base de données spécifique pour obtenir l’état de migration des opérations **Chargement complet des données en cours** et **Synchronisation des données incrémentielles**.

    ![État de l’activité - En cours](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-activity-in-progress.png)

## <a name="perform-migration-cutover"></a>Effectuer le basculement de la migration

Une fois le chargement complet initial effectué, les bases de données identifiées par le libellé **Prêt pour le basculement**.

1. Lorsque vous êtes prêt à effectuer la migration de base de données, sélectionnez **Démarrer le basculement**.

    ![Démarrer le basculement](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-start-cutover.png)

2. Veillez à arrêter toutes les transactions entrantes pour la base de données source ; attendez que le compteur **Changements en attente** indique la valeur **0**.
3. Sélectionnez **Confirmer**, puis **Appliquer**.
4. Lorsque la migration de base de données présente l’état **Terminé**, connectez vos applications à la nouvelle base de données cible.

    ![État de l’activité - Terminé](media/tutorial-rds-sql-to-azure-sql-and-managed-instance/dms-activity-completed.png)

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les limitations et les problèmes connus concernant les migrations en ligne vers Azure, consultez [Problèmes connus et solutions de contournement pour les migrations en ligne](known-issues-azure-sql-online.md).
* Pour plus d’informations sur Database Migration Service, consultez l’article [Qu’est-ce que Database Migration Service ?](./dms-overview.md)
* Pour plus d’informations sur SQL Database, consultez l’article [Qu’est-ce que le service SQL Database ?](../azure-sql/database/sql-database-paas-overview.md)
* Pour plus d’informations sur SQL Managed Instance, consultez [Présentation de SQL Managed Instance](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md).