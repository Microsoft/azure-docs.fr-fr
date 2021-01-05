---
title: SQL Server vers SQL Managed Instance - Guide de migration
description: Suivez ce guide pour migrer vos bases de données SQL Server vers Azure SQL Managed Instance.
ms.service: sql-managed-instance
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: mokabiru
ms.author: mokabiru
ms.reviewer: MashaMSFT
ms.date: 11/06/2020
ms.openlocfilehash: 5d5404537ad107a54bd32110727e5a7d0f74ebea
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96326894"
---
# <a name="migration-guide-sql-server-to-sql-managed-instance"></a>Guide de migration : de SQL Server vers SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../../includes/appliesto-sqlmi.md)]

Ce guide vous aide à migrer votre instance de SQL Server vers Azure SQL Managed Instance. 

Vous pouvez migrer SQL Server s’exécutant en local ou sur : 

- SQL Server sur les machines virtuelles  
- Amazon Web Services (AWS) EC2 
- Amazon Relational Database Service (AWS RDS) 
- Compute Engine (Google Cloud Platform - GCP)  
- Cloud SQL pour SQL Server (Google Cloud Platform - GCP) 

Pour plus d’informations sur la migration, consultez la [vue d’ensemble de la migration](sql-server-to-managed-instance-overview.md). Pour d’autres scénarios, consultez le [guide de migration de base de données](https://datamigration.microsoft.com/).

:::image type="content" source="media/sql-server-to-managed-instance-overview/migration-process-flow-small.png" alt-text="Flux du processus de migration":::

## <a name="prerequisites"></a>Prérequis 

Pour pouvoir migrer votre serveur SQL Server vers Azure SQL Managed Instance, respectez les prérequis suivants : 

- Choisir une [méthode de migration](sql-server-to-managed-instance-overview.md#compare-migration-options) et les outils correspondants nécessaires
- Installer l’[outil DMA (Assistant Migration de données)](https://www.microsoft.com/download/details.aspx?id=53595) sur une machine qui peut se connecter à votre serveur SQL Server source


## <a name="pre-migration"></a>Prémigration

Une fois que vous avez vérifié que votre environnement source est pris en charge, commencez par la phase de prémigration. Découvrez toutes les sources de données existantes, évaluez la faisabilité de la migration et identifiez les problèmes bloquants susceptibles d’empêcher la migration de s’effectuer correctement.  

### <a name="discover"></a>Découvrez

Durant la phase de découverte, analysez le réseau pour identifier toutes les instances et fonctionnalités de SQL Server utilisées par votre organisation. 

Utilisez [Azure Migrate](../../../migrate/migrate-services-overview.md) pour évaluer la pertinence de la migration des serveurs locaux, effectuer un dimensionnement en fonction des performances et fournir des estimations de coût pour les exécuter dans Azure. 

Vous pouvez également utiliser  [Microsoft Assessment and Planning Toolkit (« MAP Toolkit »)](https://www.microsoft.com/download/details.aspx?id=7826) pour évaluer votre infrastructure informatique actuelle. La boîte à outils fournit un puissant outil d’inventaire, d’évaluation et de création de rapports, qui permet de simplifier le processus de planification de la migration. 

Pour plus d’informations sur les outils utilisables au cours de la phase de découverte, consultez [Services et outils disponibles pour les scénarios de migration de données](../../../dms/dms-tools-matrix.md). 

### <a name="assess"></a>Évaluer 

Une fois les sources de données découvertes, évaluez les instances de SQL Server locales que vous pouvez migrer vers Azure SQL Managed Instance pour identifier les obstacles ou les problèmes de compatibilité liés à la migration. 

Vous pouvez utiliser l’Assistant Migration de données (version 4.1 et ultérieure) pour évaluer les bases de données et obtenir les informations suivantes : 

- [Recommandations relatives aux cibles Azure](/sql/dma/dma-assess-sql-data-estate-to-sqldb)
- [Recommandations relatives aux références SKU Azure](/sql/dma/dma-sku-recommend-sql-db)

Pour évaluer votre environnement à l’aide de l’Assistant Migration de données, suivez les étapes ci-dessous : 

1. Ouvrez l’outil [DMA (Assistant Migration de données)](https://www.microsoft.com/download/details.aspx?id=53595). 
1. Sélectionnez **Fichier**, puis choisissez **Nouvelle évaluation**. 
1. Spécifiez un nom de projet, sélectionnez le type de serveur source SQL Server, et le type de serveur cible Azure SQL Managed Instance. 
1. Sélectionnez les types de rapport d’évaluation à générer. Par exemple, la compatibilité de base de données et la parité des fonctionnalités. Selon le type d’évaluation, les autorisations nécessaires sur le serveur SQL Server source peuvent être différentes.  L’outil DMA met en évidence les autorisations nécessaires d’après l’outil Advisor choisi avant d’exécuter l’évaluation.
    - La catégorie **parité des fonctionnalités** fournit un ensemble complet de recommandations, d’alternatives disponibles dans Azure et d’étapes d’atténuation, qui vous aident à planifier votre projet de migration. (autorisations d’administrateur système nécessaires)
    - La catégorie **problèmes de compatibilité** identifie les problèmes de compatibilité des fonctionnalités partiellement ou non prises en charge, qui peuvent bloquer la migration. Elle fournit également les recommandations à suivre pour les résoudre (autorisations `CONNECT SQL`, `VIEW SERVER STATE` et `VIEW ANY DEFINITION` nécessaires).
1. Spécifiez les détails de la connexion source de votre serveur SQL Server, puis connectez-vous à la base de données source.
1. Sélectionnez **Démarrer l’évaluation**. 
1. À la fin du processus, sélectionnez et passez en revue les rapports d’évaluation pour identifier les problèmes bloquants et les problèmes de parité des fonctionnalités relatifs à la migration. Vous pouvez également exporter le rapport d’évaluation vers un fichier partagé avec d’autres équipes ou membres du personnel de votre organisation. 
1. Déterminez le niveau de compatibilité de la base de données pour réduire les efforts postmigration.  
1. Identifiez la meilleure référence SKU d’Azure SQL Managed Instance pour votre charge de travail locale. 

Pour en savoir plus, consultez [Effectuer une évaluation de migration SQL Server avec l’Assistant Migration de données](/sql/dma/dma-assesssqlonprem).

Si SQL Managed Instance n’est pas une cible appropriée pour votre charge de travail, SQL Server sur les machines virtuelles Azure peut représenter une cible alternative viable pour votre entreprise. 

#### <a name="scaled-assessments-and-analysis"></a>Évaluations mises à l’échelle et analyse

L’Assistant Migration de données prend en charge l’exécution d’évaluations mises à l’échelle et la centralisation des rapports d’évaluation à des fins d’analyse. Si vous disposez de plusieurs serveurs et bases de données qui doivent être évalués et analysés à grande échelle afin de fournir une vue plus large du parc de données, cliquez sur les liens suivants pour en savoir plus.

- [Effectuer des évaluations mises à l’échelle à l’aide de PowerShell](/sql/dma/dma-consolidatereports)
- [Analyse des rapports d’évaluation à l’aide de Power BI](/sql/dma/dma-consolidatereports#dma-reports)

> [!IMPORTANT]
>L’exécution d’évaluations à grande échelle sur plusieurs bases de données peut également être automatisée à l’aide de l’[utilitaire en ligne de commande de DMA](/sql/dma/dma-commandline). Vous pouvez ensuite charger les résultats vers [Azure Migrate](/sql/dma/dma-assess-sql-data-estate-to-sqldb#view-target-readiness-assessment-results) pour approfondir l’analyse et préparer la cible.

### <a name="create-a-performance-baseline"></a>Créer un référentiel de performance

Si vous avez besoin de comparer les performances de votre charge de travail s’exécutant sur SQL Managed Instance avec votre charge de travail d’origine s’exécutant sur SQL Server, créez une base de référence des performances à utiliser pour la comparaison. Pour en savoir plus, consultez les informations relatives à la [base de référence des performances](sql-server-to-managed-instance-performance-baseline.md). 

### <a name="create-sql-managed-instance"></a>Créer une instance SQL Database Managed Instance 

En vous basant sur les informations de la phase de découverte et d’évaluation, créez une cible SQL Managed Instance de taille appropriée. Pour ce faire, vous pouvez utiliser le [portail Azure](../../managed-instance/instance-create-quickstart.md), [PowerShell](../../managed-instance/scripts/create-configure-managed-instance-powershell.md) ou un modèle [ARM (Azure Resource Manager)](../../managed-instance/create-template-quickstart.md). 


## <a name="migrate"></a>Migrer

Une fois que vous avez effectué les tâches associées à la phase de prémigration, vous êtes prêt à effectuer la migration du schéma et des données. 

Migrez les données à l’aide de la [méthode de migration](sql-server-to-managed-instance-overview.md#compare-migration-options) de votre choix. 

Ce guide décrit les deux options les plus répandues : Azure Database Migration Service (DMS) et la fonctionnalité native de sauvegarde et restauration. 

### <a name="database-migration-service"></a>Database Migration Service

Pour effectuer des migrations à l’aide de DMS, suivez les étapes ci-dessous :

1. Inscrivez le fournisseur de ressources **Microsoft.DataMigration** dans votre abonnement, si vous effectuez cette opération pour la première fois.
1. Créez une instance Azure Database Migration Service là où vous voulez (de préférence dans la même région que votre instance cible d’Azure SQL Managed Instance), puis sélectionnez un réseau virtuel existant, ou créez-en un autre pour héberger votre instance de DMS.
1. Une fois votre instance de DMS créée, créez un projet de migration, puis spécifiez le type de serveur source **SQL Server** et le type de serveur cible **Azure SQL Database Managed Instance**. Choisissez le type d’activité dans le panneau de création de projet : migration de données en ligne ou hors connexion. 
1.  Spécifiez les détails du serveur SQL Server source dans la page **Détails de la source de migration** ainsi que les détails de la cible Azure SQL Managed Instance dans la page **Détails de la cible de migration**. Sélectionnez **Suivant**.
1. Choisissez la base de données à migrer. 
1. Fournissez les paramètres de configuration nécessaires pour spécifier le **partage réseau SMB** qui contient vos fichiers de sauvegarde de base de données. Utilisez les informations d’identification d’utilisateur Windows auprès de DMS, qui permettent d’accéder au partage réseau. Fournissez les **détails de votre compte de stockage Azure**. 
1. Passez en revue le récapitulatif de la migration, puis choisissez **Exécuter la migration**. Vous pouvez ensuite superviser l’activité de migration et vérifier la progression de la migration de votre base de données.
1. Une fois la base de données restaurée, choisissez **Démarrer le basculement**. Le processus de migration copie la sauvegarde de la fin du journal, une fois qu’elle est disponible dans le partage réseau SMB, puis la restaure sur la cible. 
1. Arrêtez tout trafic entrant à destination de votre base de données source, puis mettez à jour la chaîne de connexion vers la nouvelle base de données Azure SQL Managed Instance. 

Pour accéder à un tutoriel étape par étape basé sur cette option de migration, consultez [Procéder à la migration en ligne de SQL Server vers Azure SQL Managed Instance à l’aide de DMS](../../../dms/tutorial-sql-server-managed-instance-online.md). 
   


### <a name="backup-and-restore"></a>Sauvegarde et restauration 

L’une des fonctionnalités clés d’Azure SQL Managed Instance pour accélérer et faciliter la migration des bases de données est la restauration native des fichiers de sauvegarde de base de données (`.bak`) présents sur le service [Stockage Azure](https://azure.microsoft.com/services/storage/). La sauvegarde et la restauration sont des opérations asynchrones qui dépendent de la taille de votre base de données. 

Le diagramme suivant fournit une vue d’ensemble du processus :

![Le diagramme affiche SQL Server avec une flèche intitulée SAUVEGARDE / Télécharger vers l’URL qui passe à Azure Storage, et une deuxième flèche intitulée RESTAURER à partir de l’URL qui circule du stockage Azure vers une Managed Instance de SQL.](./media/sql-server-to-managed-instance-overview/migration-restore.png)

> [!NOTE]
> Le temps nécessaire pour effectuer une sauvegarde, la charger vers le stockage Azure et exécuter une opération de restauration native sur Azure SQL Managed Instance dépend de la taille de la base de données. Prenez en compte un temps d’arrêt suffisant pour les grandes bases de données. 


Pour effectuer une migration à l’aide de la fonctionnalité de sauvegarde et restauration, suivez les étapes ci-dessous : 

1. Sauvegardez votre base de données sur le Stockage Blob Azure. Par exemple, utilisez la fonctionnalité de [sauvegarde vers une URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url) dans [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms). Utilisez [Microsoft Azure Tools](https://go.microsoft.com/fwlink/?LinkID=324399) pour prendre en charge les bases de données antérieures à SQL Server 2012 SP1 CU2. 
1. Connectez-vous à votre instance Azure SQL Managed Instance via SQL Server Management Studio. 
1. Créez des informations d’identification à l’aide d’une signature d’accès partagé pour accéder à votre compte Stockage Blob Azure avec vos sauvegardes de base de données. Par exemple :

   ```sql
   CREATE CREDENTIAL [https://mitutorials.blob.core.windows.net/databases]
   WITH IDENTITY = 'SHARED ACCESS SIGNATURE'
   , SECRET = 'sv=2017-11-09&ss=bfqt&srt=sco&sp=rwdlacup&se=2028-09-06T02:52:55Z&st=2018-09-04T18:52:55Z&spr=https&sig=WOTiM%2FS4GVF%2FEEs9DGQR9Im0W%2BwndxW2CQ7%2B5fHd7Is%3D'
   ```
1. Restaurez la sauvegarde à partir du conteneur Azure Storage Blob. Par exemple : 

    ```sql
   RESTORE DATABASE [TargetDatabaseName] FROM URL =
     'https://mitutorials.blob.core.windows.net/databases/WideWorldImporters-Standard.bak'
   ```

1. Une fois la restauration effectuée, visualisez la base de données dans l’**Explorateur d’objets** de SQL Server Management Studio. 

Pour en savoir plus sur cette option de migration, consultez [Restaurer une base de données dans Azure SQL Managed Instance avec SSMS](../../managed-instance/restore-sample-database-quickstart.md).

> [!NOTE]
> Une opération de restauration de base de données est asynchrone et peut être retentée. SQL Server Management Studio peut générer une erreur en cas d’interruption de la connexion ou d’expiration d’un délai d’attente. Azure SQL Database continue d’essayer de restaurer la base de données en arrière-plan et vous pouvez suivre l’avancement de la restauration dans les vues [sys.dm_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) et [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database).



## <a name="data-sync-and-cutover"></a>Synchronisation des données et basculement

Quand vous utilisez des options de migration qui répliquent/synchronisent en permanence les changements apportés aux données de la source vers la cible, les données et le schéma sources peuvent varier et dériver par rapport à la cible. Pendant la synchronisation des données, vérifiez que tous les changements apportés à la source sont capturés et appliqués à la cible au cours du processus de migration. 

Une fois que vous avez vérifié que les données sont identiques sur la source et la cible, vous pouvez effectuer le basculement de l’environnement source vers l’environnement cible. Il est important de planifier le processus de basculement avec les équipes métier/applicatives. Ainsi, vous avez la garantie que l’interruption minimale qui a lieu pendant le basculement n’affecte pas la continuité de l’activité. 

> [!IMPORTANT]
> Pour plus d’informations sur les étapes spécifiques associées à l’exécution d’un basculement dans le cadre de migrations à l’aide du service DMS, consultez [Exécution du basculement de migration](../../../dms/tutorial-sql-server-managed-instance-online.md#performing-migration-cutover).


## <a name="post-migration"></a>Postmigration

Une fois que vous avez réussi la phase de migration, vous devez effectuer toute une série de tâches postmigration pour vérifier que tout fonctionne de manière fluide et efficace. 

La phase postmigration est cruciale pour résoudre les problèmes de justesse et d’exhaustivité des données ainsi que pour gérer les problèmes de performances liés à la charge de travail. 

### <a name="remediate-applications"></a>Corriger les applications 

Une fois les données migrées vers l’environnement cible, toutes les applications qui consommaient la source doivent commencer à consommer la cible. Dans certains cas, cela vous oblige à apporter des changements aux applications.

### <a name="perform-tests"></a>Effectuer des tests

L’approche de test pour la migration de base de données comprend les activités suivantes :

1. **Développer des tests de validation** : pour tester la migration d’une base de données, vous devez utiliser des requêtes SQL. Vous devez créer les requêtes de validation à exécuter sur les bases de données source et cible. Vos requêtes de validation doivent couvrir l’étendue que vous avez définie.
1. **Configurer un environnement de test**: l’environnement de test doit contenir une copie de la base de données source et de la base de données cible. Veillez à isoler l’environnement de test.
1. **Exécuter des tests de validation** : exécutez les tests de validation sur la source et sur la cible, puis analysez les résultats.
1. **Exécuter des tests de performances**: exécutez un test de performances sur la source et sur la cible, puis analysez et comparez les résultats.

   > [!NOTE]
   > Pour obtenir de l’aide sur le développement et l’exécution de tests de validation post-migration, envisagez d’utiliser la Solution de qualité des données disponible dans le partenaire [QuerySurge](https://www.querysurge.com/company/partners/microsoft). 



## <a name="leverage-advanced-features"></a>Tirer profit des fonctionnalités avancées 

Veillez à tirer parti des fonctionnalités cloud avancées offertes par SQL Managed Instance, notamment la [haute disponibilité intégrée](../../database/high-availability-sla.md), la [détection des menaces](../../database/azure-defender-for-sql.md) ainsi que la [supervision et le paramétrage de votre charge de travail](../../database/monitor-tune-overview.md). 

[Azure SQL Analytics](../../../azure-monitor/insights/azure-sql.md) vous permet de superviser un grand nombre d’instances managées de manière centralisée.

Certaines fonctionnalités SQL Server sont disponibles uniquement une fois que vous avez fait passer le [niveau de compatibilité de la base de données](/sql/relational-databases/databases/view-or-change-the-compatibility-level-of-a-database) au dernier niveau (150). 


## <a name="next-steps"></a>Étapes suivantes

- Si vous souhaitez obtenir une matrice des services et outils Microsoft et tiers qui peuvent vous aider dans différents scénarios de migration de données et de base de données ainsi que pour des tâches spécialisées, consultez [Services et outils disponibles pour les scénarios de migration de données](../../../dms/dms-tools-matrix.md).

- Pour en savoir plus sur Azure SQL Managed Instance, consultez :
   - [Niveaux de service dans Azure SQL Managed Instance](../../managed-instance/sql-managed-instance-paas-overview.md#service-tiers)
   - [Différences entre SQL Server et Azure SQL Managed Instance](../../managed-instance/transact-sql-tsql-differences-sql-server.md)
   - [Outil de calcul du coût total de possession Azure](https://azure.microsoft.com/pricing/tco/calculator/) 


- Pour en savoir plus sur le framework et le cycle d’adoption des migrations cloud, consultez
   -  [Cloud Adoption Framework pour Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Bonnes pratiques pour l’évaluation des coûts et du dimensionnement des charges de travail migrées vers Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 

- Pour évaluer la couche d’accès aux applications, consultez [Data Access Migration Toolkit (préversion)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)
- Pour plus d’informations sur l’exécution des tests A/B de la couche d’accès aux données, consultez [Assistant Expérimentation de base de données](/sql/dea/database-experimentation-assistant-overview).