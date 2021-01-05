---
title: 'Tutoriel : Migrer SQL Server en ligne vers une base de données unique SQL'
titleSuffix: Azure Database Migration Service
description: Apprenez à effectuer une migration en ligne entre SQL Server et Azure SQL Database à l'aide d'Azure Database Migration Service.
services: dms
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 01/21/2020
ms.openlocfilehash: 29a154e33d8025f53b929bf0bb7602b2d0a64b13
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94955019"
---
# <a name="tutorial-migrate-sql-server-to-a-single-database-or-pooled-database-in-azure-sql-database-online-using-dms"></a>Tutoriel : Migrer SQL Server vers une base de données unique ou mise en pool dans Azure SQL Database en ligne à l’aide de DMS

Vous pouvez utiliser Azure Database Migration Service pour migrer les bases de données d'une instance de SQL Server vers [Azure SQL Database](/azure/sql-database/) avec un temps d'arrêt minimal. Dans ce tutoriel, vous migrez la base de données **Adventureworks2012** restaurée sur une instance locale de SQL Server 2016 (ou une version ultérieure) vers une base de données unique ou mise en pool dans Azure SQL Database à l’aide du service Azure Database Migration Service.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
>
> - Évaluer votre base de données locale à l’aide de l’Assistant Migration des données.
> - Migrer l’exemple de schéma à l’aide de l’Assistant Migration des données.
> - Créer une instance Azure Database Migration Service.
> - Créer un projet de migration en utilisant Azure Database Migration Service.
> - Exécuter la migration.
> - Surveiller la migration.
> - Télécharger un rapport de migration.

> [!NOTE]
> Effectuer une migration en ligne à l’aide d’Azure Database Migration Service nécessite la création d’une instance basée sur le niveau tarifaire Premium. Pour plus d’informations, consultez la page de [tarification](https://azure.microsoft.com/pricing/details/database-migration/) du service Azure Database Migration Service.

> [!IMPORTANT]
> Pour une expérience de migration optimale, Microsoft vous recommande de créer une instance Azure Database Migration Service dans la même région Azure que la base de données cible. Le déplacement des données entre les régions ou les zones géographiques peut ralentir le processus de migration et introduire des erreurs.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

Cet article décrit une migration en ligne de SQL Server vers une base de données unique ou une base de données mise en pool dans Azure SQL Database. Pour une migration hors connexion, consultez [Migrer SQL Server vers Azure SQL Database hors connexion à l’aide de DMS](tutorial-sql-server-to-azure-sql.md).

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez effectuer les opérations suivantes :

- Téléchargez et installez [SQL Server 2012 ou ultérieur](https://www.microsoft.com/sql-server/sql-server-downloads).
- Activez le protocole TCP/IP, qui est désactivé par défaut pendant l’installation de SQL Server Express, en suivant les instructions de l’article [Activer ou désactiver un protocole réseau de serveur](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure).
- Créez une base de données unique (ou mise en pool) dans Azure SQL Database, en suivant les indications de l’article [Créer une base de données unique dans Azure SQL Database à l’aide du Portail Azure](../azure-sql/database/single-database-create-quickstart.md).

    > [!NOTE]
    > Si vous utilisez SQL Server Integration Services (SSIS) pour migrer la base de données de catalogue de vos projets ou de vos packages SSIS (SSISDB) entre SQL Server et Azure SQL Database, la base de données SSISDB de destination sera créée et gérée automatiquement lorsque vous provisionnerez SSIS dans Azure Data Factory (ADF). Pour plus d’informations sur la migration des packages SSIS, consultez l’article [Migrer des packages SQL Server Integration Services vers Azure](./how-to-migrate-ssis-packages.md).

- Téléchargez et installez [l’Assistant Migration de données](https://www.microsoft.com/download/details.aspx?id=53595) (DMA) v3.3 ou version ultérieure.
- Créer un réseau virtuel Microsoft Azure pour Azure Database Migration Service à l’aide du modèle de déploiement Azure Resource Manager, qui fournit une connectivité de site à site à vos serveurs sources locaux, utilisez [ExpressRoute](../expressroute/expressroute-introduction.md) ou un [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Pour plus d’informations sur la création d’un réseau virtuel, consultez la [documentation sur le réseau virtuel](../virtual-network/index.yml), en particulier les articles sur le démarrage rapide, qui fournissent des informations pas à pas.

    > [!NOTE]
    > Pendant la configuration du réseau virtuel, si vous utilisez ExpressRoute avec le peering réseau à Microsoft, ajoutez ces [points de terminaison](../virtual-network/virtual-network-service-endpoints-overview.md) au sous-réseau où doit être provisionné le service :
    > - Point de terminaison de base de données cible (un point de terminaison SQL ou Cosmos DB, par exemple)
    > - Point de terminaison de stockage
    > - Point de terminaison Service Bus
    >
    > Cette configuration est nécessaire, car Azure Database Migration Service ne dispose pas d’une connectivité Internet.

- Vérifiez que les règles du groupe de sécurité réseau de votre réseau virtuel ne bloquent pas les ports suivants pour les communications entrantes vers Azure Database Migration Service : 443, 53, 9354, 445, 12000. Pour plus d’informations sur le filtrage du trafic de groupe de sécurité réseau de réseau virtuel Azure, consultez l’article [Filtrer le trafic réseau avec les groupes de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md).
- Configurez votre [pare-feu Windows pour accéder au moteur de base de données](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
- Ouvrez votre Pare-feu Windows pour permettre à Azure Database Migration Service d’accéder au serveur SQL Server source, par défaut le port TCP 1433. Si votre instance par défaut écoute sur un autre port, ajoutez ce port au pare-feu.
- Si vous exécutez plusieurs instances nommées de SQL Server avec des ports dynamiques, vous pouvez activer le service SQL Browser et autoriser l’accès au port UDP 1434 à travers vos pare-feu, de sorte qu’Azure Database Migration Service puisse se connecter à une instance nommée sur votre serveur source.
- Lorsque vous utilisez une appliance de pare-feu devant vos bases de données sources, vous devrez peut-être ajouter des règles de pare-feu pour permettre à Azure Database Migration Service d’accéder aux bases de données sources pour la migration.
- Créez une [règle de pare-feu](../azure-sql/database/firewall-configure.md) au niveau du serveur pour qu'Azure SQL Database permette à Azure Database Migration Service d'accéder aux bases de données cibles. Fournissez la plage de sous-réseau du réseau virtuel utilisé pour Azure Database Migration Service.
- Assurez-vous que les informations d’identification utilisées pour se connecter à une instance SQL Server source disposent des autorisations [CONTROL SERVER](/sql/t-sql/statements/grant-server-permissions-transact-sql).
- Assurez-vous que les informations d’identification utilisées pour se connecter à l’instance Azure SQL Database cible disposent des autorisations CONTROL DATABASE pour les instances Azure SQL Database cibles.
- Le serveur SQL Server source doit exécuter SQL Server 2005 ou ultérieur. Pour déterminer la version que votre instance SQL Server exécute, consultez l’article [Comment faire pour déterminer la version, l’édition et le niveau de mise à jour de SQL Server et ses composants](https://support.microsoft.com/help/321185/how-to-determine-the-version-edition-and-update-level-of-sql-server-an).
- Les bases de données doivent être en mode de récupération complète ou en mode de récupération utilisant les journaux de transactions. Pour déterminer le mode de récupération configuré pour votre instance SQL Server, consultez l’article [Afficher ou modifier le mode de récupération d’une base de données (SQL Server)](/sql/relational-databases/backup-restore/view-or-change-the-recovery-model-of-a-database-sql-server?view=sql-server-2017).
- Veillez à effectuer des sauvegardes complètes des bases de données. Pour créer une sauvegarde de base de données complète, consultez l’article [Guide pratique pour créer une sauvegarde de base de données complète (Transact-SQL)](/previous-versions/sql/sql-server-2008-r2/ms191304(v=sql.105)).
- Si aucune des tables ne comporte de clé primaire, activez la capture des changements de données (CDC) sur la base de données et sur une ou plusieurs tables spécifiques.
    > [!NOTE]
    > Vous pouvez utiliser le script ci-après pour rechercher toutes les tables dépourvues de clé primaire.
    
    ```sql
    USE <DBName>;
    go
    SELECT is_tracked_by_cdc, name AS TableName
    FROM sys.tables WHERE type = 'U' and is_ms_shipped = 0 AND
    OBJECTPROPERTY(OBJECT_ID, 'TableHasPrimaryKey') = 0;
    ```

    Si les résultats indiquent une ou plusieurs tables pour lesquelles « is_tracked_by_cdc » présente la valeur 0, activez la capture des changements de données pour la base de données et pour les tables spécifiques en exécutant la procédure décrite dans l’article [Activer et désactiver la capture de données modifiées (SQL Server)](/sql/relational-databases/track-changes/enable-and-disable-change-data-capture-sql-server?view=sql-server-2017).

- Configurez le rôle de base de données du serveur de distribution pour le serveur SQL Server source.

    >[!NOTE]
    > Pour déterminer si les composants de réplication sont installés, utilisez la requête ci-après.

    ```sql
    USE master;
    DECLARE @installed int;
    EXEC @installed = sys.sp_MS_replication_installed;
    SELECT @installed as installed;
    ```

    Si le résultat renvoie un message d’erreur vous suggérant d’installer des composants de réplication, installez les composants de réplication SQL Server en suivant la procédure décrite dans l’article [Installer la réplication SQL Server](/sql/database-engine/install-windows/install-sql-server-replication?view=sql-server-2017).

    Si la réplication est déjà installée, exécutez la commande T-SQL ci-après pour vérifier si le rôle de distribution est configuré sur le serveur SQL Server source.

    ```sql
    EXEC sp_get_distributor;
    ```

    Si le serveur de distribution indique la valeur NULL dans la sortie de la commande ci-dessus, ce qui signifie que la distribution n’est pas configurée, configurez la distribution en utilisant les instructions de l’article [Configurer la publication et la distribution](/sql/relational-databases/replication/configure-publishing-and-distribution?view=sql-server-2017).

- Désactivez les déclencheurs de base de données sur la base de données Azure SQL Database cible.
    >[!NOTE]
    > Pour rechercher les déclencheurs de base de données sur la base de données Azure SQL Database cible, exécutez la requête suivante :

    ```sql
    Use <Database name>
    select * from sys.triggers
    DISABLE TRIGGER (Transact-SQL)
    ```

    Pour plus d’informations, consultez l’article [DISABLE TRIGGER (Transact-SQL)](/sql/t-sql/statements/disable-trigger-transact-sql?view=sql-server-2017). 

## <a name="assess-your-on-premises-database"></a>Évaluer votre base de données locale

Avant de pouvoir migrer des données d'une instance de SQL Server vers Azure SQL Database, vous devez évaluer la base de données SQL Server afin de détecter tout problème de blocage qui pourrait empêcher la migration. Une fois que vous avez téléchargé et installé l’Assistant Migration de données v3.3 ou version ultérieure, suivez les étapes décrites dans l’article [Évaluation de la migration de SQL Server](/sql/dma/dma-assesssqlonprem) pour évaluer la base de données locale.

Pour évaluer une base de données locale, effectuez les étapes suivantes :

1. Dans DMA, sélectionnez l’icône Nouveau (+), puis sélectionnez le type de projet **Évaluation**.
2. Spécifiez un nom de projet, dans la zone de texte **Type de serveur source**, sélectionnez **SQL Server** puis, dans la zone de texte **Type de serveur cible**, sélectionnez **Azure SQL Database**, puis sélectionnez **Créer** pour créer le projet.

   Quand vous évaluez la base de données SQL Server source qui doit faire l’objet d’une migration vers une base de données unique ou mise en pool dans Azure SQL Database, vous pouvez choisir l’un des types de rapport d’évaluation suivants, ou les deux :

   - Vérifier la compatibilité de la base de données
   - Vérifier la parité de fonctionnalité

   Les deux types de rapports sont sélectionnés par défaut.

3. Dans DMA, sur l’écran **Options**, sélectionnez **Suivant**.
4. Dans l’écran **Sélectionner les sources**, dans la boîte de dialogue **Se connecter à un serveur**, fournissez les détails de connexion à votre SQL Server, puis sélectionnez **Se connecter**.
5. Dans la boîte de dialogue **Add sources (Ajouter des sources)** , sélectionnez **AdventureWorks2012** et **Ajouter**, puis **Start Assessment (Démarrer l’évaluation)** .

    > [!NOTE]
    > Si vous utilisez SSIS, sachez que l’Assistant Migration de données ne prend pas en charge l’évaluation de la SSISDB source. Toutefois, les projets ou packages SSIS seront évalués/validés lors de leur redéploiement vers la SSISDB de destination qui est hébergée par Azure SQL Database. Pour plus d’informations sur la migration des packages SSIS, consultez l’article [Migrer des packages SQL Server Integration Services vers Azure](./how-to-migrate-ssis-packages.md).

    Une fois l’évaluation terminée, les résultats s’affichent comme illustré dans le graphique suivant :

    ![Évaluer la migration des données](media/tutorial-sql-server-to-azure-sql-online/dma-assessments.png)

    En ce qui concerne les bases de données uniques ou les bases de données mises en pool dans Azure SQL Database, les évaluations vous permettent d’identifier les problèmes de parité des fonctionnalités ainsi que les problèmes de blocage de migration pour le déploiement sur une base de données unique ou mise en pool.

    - La catégorie de **parité de fonctionnalité SQL Server** fournit un ensemble complet de recommandations, d’approches alternatives disponibles dans Azure et de procédures d’atténuation pour vous aider à planifier les efforts de vos projets de migration.
    - La catégorie de **problèmes de compatibilité** indique les fonctionnalités partiellement prises en charge ou non prises en charge qui reflètent les problèmes de compatibilité susceptibles de bloquer la migration des bases de données SQL Server vers Azure SQL Database. Des recommandations sont également fournies pour vous aider à résoudre ces problèmes.

6. Passez en revue les résultats d’évaluation en ce qui concerne les problèmes de blocage de migration et les problèmes de parité de fonctionnalité en sélectionnant les options spécifiques.

## <a name="migrate-the-sample-schema"></a>Migrer l’exemple de schéma

Une fois que vous êtes à l’aise avec l’évaluation et que vous pensez que la base de données sélectionnée est un candidat viable pour la migration vers une base de données unique ou mise en pool dans Azure SQL Database, utilisez DMA pour migrer le schéma vers Azure SQL Database.

> [!NOTE]
> Avant de créer un projet de migration dans DMA, vérifiez que vous avez déjà provisionné une base de données SQL dans Azure, comme indiqué dans les prérequis. Dans ce tutoriel, le nom de la base de données Azure SQL devrait être **AdventureWorksAzure**, mais vous pouvez lui attribuer un autre nom si vous le souhaitez.

> [!IMPORTANT]
> Si vous utilisez SSIS, sachez que l’Assistant Migration de données ne prend pas en charge la migration de la SSISDB source. Toutefois, vous pouvez redéployer vos projets ou packages SSIS vers la SSISDB de destination qui est hébergée par Azure SQL Database. Pour plus d’informations sur la migration des packages SSIS, consultez l’article [Migrer des packages SQL Server Integration Services vers Azure](./how-to-migrate-ssis-packages.md).

Pour migrer le schéma **AdventureWorks2012** vers une base de données unique ou une base de données mise en pool Azure SQL Database, effectuez ce qui suit :

1. Dans Data Migration Assistant, sélectionnez l’icône Nouveau (+), puis sous **Type de projet**, sélectionnez **Migration**.
2. Spécifiez un nom de projet, dans la zone de texte **Type de serveur source**, sélectionnez **SQL Server** puis, dans la zone de texte **Type de serveur cible**, sélectionnez **Azure SQL Database**.
3. Sous **Étendue de la migration**, sélectionnez **Schéma uniquement**.

    Une fois les étapes précédentes effectuées, l’interface DMA doit présenter l’aspect illustré ci-après :

    ![Créer un projet d’Assistant Migration des données](media/tutorial-sql-server-to-azure-sql-online/dma-create-project.png)

4. Sélectionnez **Créer** pour créer le projet.
5. Dans DMA, spécifiez les détails de connexion source de votre serveur SQL Server, sélectionnez **Se connecter**, puis sélectionnez la base de données **AdventureWorks2012**.

    ![Détails de connexion source de l’Assistant Migration des données](media/tutorial-sql-server-to-azure-sql-online/dma-source-connect.png)

6. Sélectionnez **Suivant**, sous **Se connecter au serveur cible**, spécifiez les détails de connexion cible de la base de données Azure SQL, sélectionnez **Se connecter**, puis sélectionnez la base de données **AdventureWorksAzure** que vous avez pré-approvisionnée dans Azure SQL Database.

    ![Détails de connexion cible de l’Assistant Migration des données](media/tutorial-sql-server-to-azure-sql-online/dma-target-connect.png)

7. Sélectionnez **Suivant** pour accéder à l’écran **Sélectionner des objets** dans lequel vous pouvez spécifier les objets de schéma dans la base de données **AdventureWorks2012** à déployer sur Azure SQL Database.

    Par défaut, tous les objets sont sélectionnés.

    ![Générer des scripts SQL](media/tutorial-sql-server-to-azure-sql-online/dma-assessment-source.png)

8. Sélectionnez **Générer un script SQL** pour créer les scripts SQL, puis recherchez d’éventuelles erreurs dans les scripts.

    ![Script de schéma](media/tutorial-sql-server-to-azure-sql-online/dma-schema-script.png)

9. Sélectionnez **Déployer le schéma** pour déployer sur Azure SQL Database puis, une fois le schéma déployé, recherchez d’éventuelles anomalies sur le serveur cible.

    ![Déployer le schéma](media/tutorial-sql-server-to-azure-sql-online/dma-schema-deploy.png)

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Inscrire le fournisseur de ressources Microsoft.DataMigration

1. Connectez-vous au portail Azure, sélectionnez **Tous les services**, puis **Abonnements**.

   ![Afficher les abonnements au portail](media/tutorial-sql-server-to-azure-sql-online/portal-select-subscription1.png)

2. Sélectionnez l’abonnement dans lequel vous souhaitez créer l’instance Azure Database Migration Service, puis sélectionnez **Fournisseurs de ressources**.

    ![Afficher les fournisseurs de ressources](media/tutorial-sql-server-to-azure-sql-online/portal-select-resource-provider.png)

3. Recherchez migration, puis à droite de **Microsoft.DataMigration**, sélectionnez **Inscrire**.

    ![S’inscrire auprès du fournisseur de ressources](media/tutorial-sql-server-to-azure-sql-online/portal-register-resource-provider.png)

## <a name="create-an-instance"></a>Créer une instance

1. Dans le portail Azure, sélectionnez **+ Créer une ressource**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Service** dans la liste déroulante.

    ![Place de marché Azure](media/tutorial-sql-server-to-azure-sql-online/portal-marketplace.png)

2. Dans l’écran **Azure Database Migration Service**, sélectionnez **Créer**.

    ![Créer une instance Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql-online/dms-create1.png)
  
3. Dans l’écran **Créer un service de migration**, spécifiez un nom pour le service, l’abonnement, et un réseau virtuel nouveau ou existant.

4. Sélectionnez l’emplacement dans lequel vous souhaitez créer l’instance Azure Database Migration Service. 

5. Sélectionnez un réseau virtuel existant ou créez-en un.

    Le réseau virtuel fournit à Azure Database Migration Service un accès au serveur SQL Server source et à l’instance Azure SQL Database cible.

    Pour plus d’informations sur la création d’un réseau virtuel dans le portail Azure, consultez l’article [Créer un réseau virtuel au moyen du portail Azure](../virtual-network/quick-create-portal.md).

6. Sélectionnez un niveau tarifaire.

    Pour plus d’informations sur les coûts et les niveaux de tarification, consultez la [page de tarification](https://aka.ms/dms-pricing).

    ![Configurer des paramètres d’instance Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql-online/dms-settings2.png)

7. Sélectionnez **Créer** pour créer le service.

## <a name="create-a-migration-project"></a>Créer un projet de migration

Une fois le service créé, recherchez-le dans le portail Azure, ouvrez-le, puis créez un projet de migration.

1. Dans le portail Azure, sélectionnez **Tous les services**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Services**.

    ![Rechercher toutes les instances Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql-online/dms-search.png)

2. Dans l’écran **Azure Database Migration Services**, recherchez le nom de l’instance Azure Database Migration Service que vous avez créée, puis sélectionnez l’instance.

    ![Rechercher votre instance Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql-online/dms-instance-search.png)

3. Sélectionnez **+ Nouveau projet de migration**.
4. Dans l’écran **Nouveau projet de migration**, spécifiez un nom pour le projet, dans la zone de texte **Type de serveur source**, sélectionnez **SQL Server** et, dans la zone de texte **Type de serveur cible**, sélectionnez **Azure SQL Database**.
5. Dans la section **Choisir un type d’activité**, sélectionnez **Migration de données en ligne**.

    ![Créer un projet Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql-online/dms-create-project3.png)

    > [!NOTE]
    > Une autre possibilité consiste à choisir **Créer un projet uniquement** pour créer le projet de migration à ce stade et exécuter la migration ultérieurement.

6. Sélectionnez **Enregistrer**.

7. Sélectionnez **Créer et exécuter une activité** pour créer le projet et exécuter l’activité de migration.

    ![Création et exécution d’une activité Azure Database Migration Service](media/tutorial-sql-server-to-azure-sql-online/dms-create-and-run-activity.png)

## <a name="specify-source-details"></a>Spécifier les détails de la source

1. Dans l’écran **Détails de la source de migration**, spécifiez les détails de connexion de l’instance SQL Server source.

    Veillez à utiliser un nom de domaine complet pour le nom de l’instance SQL Server source. Vous pouvez également utiliser l’adresse IP quand la résolution de noms DNS est impossible.

2. Si vous n’avez pas installé de certificat approuvé sur votre serveur source, cochez la case **Faire confiance au certificat de serveur**.

    Quand aucun certificat approuvé n’est installé, SQL Server génère un certificat auto-signé au démarrage de l’instance. Ce certificat permet de chiffrer les informations d’identification des connexions clientes.

    > [!CAUTION]
    > Les connexions TLS chiffrées à l’aide d’un certificat auto-signé n’offrent pas de sécurité renforcée. Elles sont vulnérables aux attaques de l’intercepteur. Ne faites jamais confiance à une connexion TLS utilisant des certificats autosignés dans un environnement de production ou sur des serveurs connectés à Internet.

   ![Détails de la source](media/tutorial-sql-server-to-azure-sql-online/dms-source-details3.png)

    > [!IMPORTANT]
    > Si vous utilisez SSIS, sachez que Database Migration Service ne prend pas en charge la migration de la SSISDB source. Toutefois, vous pouvez redéployer vos projets ou packages SSIS vers la SSISDB de destination qui est hébergée par Azure SQL Database. Pour plus d’informations sur la migration des packages SSIS, consultez l’article [Migrer des packages SQL Server Integration Services vers Azure](./how-to-migrate-ssis-packages.md).

## <a name="specify-target-details"></a>Spécifier les détails de la cible

1. Sélectionnez **Enregistrer** puis, sur l’écran **Détails de la cible de migration**, spécifiez les détails de connexion de l’instance cible d’Azure SQL Database, à savoir l’instance d’Azure SQL Database pré-approvisionnée sur laquelle le schéma **AdventureWorks2012** a été déployé à l’aide de DMA.

    ![Sélectionner la cible](media/tutorial-sql-server-to-azure-sql-online/dms-select-target3.png)

2. Sélectionnez **Enregistrer**, puis dans l’écran **Mapper aux bases de données cibles**, mappez les bases de données source et cible pour la migration.

    Si la base de données cible porte le même nom que la base de données source, Azure Database Migration Service sélectionne la base de données cible par défaut.

    ![Mapper aux bases de données cibles](media/tutorial-sql-server-to-azure-sql-online/dms-map-targets-activity3.png)

3. Sélectionnez **Enregistrer**, dans l’écran **Sélectionner des tables**, développez la liste des tables et passez en revue la liste des champs concernés.

    Azure Database Migration Service sélectionne automatiquement toutes les tables sources vides qui existent sur l’instance Azure SQL Database cible. Si vous souhaitez migrer à nouveau les tables qui contiennent déjà des données, vous devez sélectionner explicitement les tables sur ce panneau.

    ![Sélectionner des tables](media/tutorial-sql-server-to-azure-sql-online/dms-configure-setting-activity3.png)

4. Sélectionnez **Enregistrer**, dans l’écran **Récapitulatif de la migration** pour la zone de texte **Nom de l’activité**, spécifiez un nom pour l’activité de migration, puis examinez le récapitulatif pour vous assurer que les détails de la source et de la cible correspondent à ceux que vous avez spécifiés précédemment.

    ![Résumé de la migration](media/tutorial-sql-server-to-azure-sql-online/dms-migration-summary.png)

## <a name="run-the-migration"></a>Exécuter la migration

- Sélectionnez **Exécuter la migration**.

    La fenêtre d’activité de migration s’affiche, et le champ **État** de l’activité présente la valeur **Initialisation en cours**.

    ![État de l’activité - Initialisation en cours](media/tutorial-sql-server-to-azure-sql-online/dms-activity-status2.png)

## <a name="monitor-the-migration"></a>Surveiller la migration

1. Dans l’écran d’activité de migration, sélectionnez **Actualiser** pour mettre à jour l’affichage jusqu’à ce que le champ **État** de la migration prenne la valeur **En cours d’exécution**.

2. Cliquez sur une base de données spécifique pour obtenir l’état de migration des opérations **Chargement complet des données en cours** et **Synchronisation des données incrémentielles**.

    ![État de l’activité - En cours](media/tutorial-sql-server-to-azure-sql-online/dms-activity-in-progress.png)

## <a name="perform-migration-cutover"></a>Effectuer le basculement de la migration

Une fois le chargement complet initial effectué, les bases de données identifiées par le libellé **Prêt pour le basculement**.

1. Lorsque vous êtes prêt à effectuer la migration de base de données, sélectionnez **Démarrer le basculement**.

    ![Démarrer le basculement](media/tutorial-sql-server-to-azure-sql-online/dms-start-cutover.png)

2. Veillez à arrêter toutes les transactions entrantes pour la base de données source ; attendez que le compteur **Changements en attente** indique la valeur **0**.
3. Sélectionnez **Confirmer**, puis **Appliquer**.
4. Lorsque la migration de base de données présente l’état **Terminé**, connectez vos applications à la nouvelle base de données Azure SQL cible.

    ![État de l’activité - Terminé](media/tutorial-sql-server-to-azure-sql-online/dms-activity-completed.png)

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les limitations et problèmes connus concernant les migrations en ligne vers Azure SQL Database, consultez l’article sur les [problèmes connus et solutions de contournement des migrations en ligne Azure SQL Database](known-issues-azure-sql-online.md).
- Pour plus d’informations sur Azure Database Migration Service, consultez l’article [Qu’est-ce qu’Azure Database Migration Service ?](./dms-overview.md).
- Pour plus d’informations sur Azure SQL Database, consultez l’article [Qu’est-ce que le service Azure SQL Database ?](../azure-sql/database/sql-database-paas-overview.md).