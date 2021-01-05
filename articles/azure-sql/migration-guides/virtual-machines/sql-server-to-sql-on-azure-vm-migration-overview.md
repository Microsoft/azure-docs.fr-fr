---
title: SQL Server vers SQL Server sur machine virtuelle Azure (Vue d’ensemble de la migration)
description: Découvrez les différentes stratégies de migration qui existent lorsque vous souhaitez migrer de SQL Server vers SQL Server sur des machines virtuelles Azure.
ms.custom: ''
ms.service: virtual-machines-sql
ms.subservice: ''
ms.devlang: ''
ms.topic: how-to
author: markjones-msft
ms.author: markjon
ms.reviewer: mathoma
ms.date: 11/06/2020
ms.openlocfilehash: 4979902853602073e6230ef7387d6c6596fe77da
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96325914"
---
# <a name="migration-overview-sql-server-to-sql-server-on-azure-vms"></a>Vue d’ensemble de la migration : SQL Server vers SQL Server sur les machines virtuelles Azure
[!INCLUDE[appliesto--sqlmi](../../includes/appliesto-sqlvm.md)]

Découvrez les différentes stratégies de migration qui existent pour migrer de SQL Server vers SQL Server sur des machines virtuelles Azure. 

Vous pouvez migrer SQL Server s’exécutant en local ou sur :

- SQL Server sur les machines virtuelles  
- Amazon Web Services (AWS) EC2 
- Amazon Relational Database Service (AWS RDS) 
- Compute Engine (Google Cloud Platform – GCP)

Pour d’autres scénarios, consultez le [guide de migration de base de données](https://datamigration.microsoft.com/). 

## <a name="overview"></a>Vue d’ensemble

Migrez vers [SQL Server sur les machines virtuelles Azure](../../virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md) lorsque vous souhaitez utiliser l’environnement familier de SQL Server avec le contrôle du système d’exploitation et que vous souhaitez tirer parti des fonctionnalités fournies par le cloud, telles que la haute disponibilité intégrée des machines virtuelles, les [sauvegardes automatisées](../../virtual-machines/windows/automated-backup.md) et la [mise à jour corrective automatisée](../../virtual-machines/windows/automated-patching.md). 

Réalisez des économies en dotant votre propre licence du [modèle de licence Azure Hybrid Benefit](../../virtual-machines/windows/licensing-model-azure-hybrid-benefit-ahb-change.md) ou étendez la prise en charge de SQL Server 2008 et SQL Server 2008 R2 en obtenant des [mises à jour de sécurité gratuites](../../virtual-machines/windows/sql-server-2008-extend-end-of-support.md). 


## <a name="choosing-appropriate-target"></a>Choix de la cible appropriée

Les machines virtuelles Azure s’exécutent dans de nombreuses régions d’Azure et offrent également une variété de [tailles de machine](../../../virtual-machines/sizes.md) et d’[options de stockage](../../../virtual-machines/disks-types.md). Pour déterminer la taille correcte de la machine virtuelle et du stockage pour votre charge de travail SQL Server, consultez [Recommandations de performances pour SQL Server sur les machines virtuelles Azure](../../virtual-machines/windows/performance-guidelines-best-practices.md#vm-size-guidance). Pour déterminer les besoins en matière de stockage et de taille de machine virtuelle pour votre charge de travail, il est recommandé de les dimensionner par le biais d’une [évaluation Azure Migrate](../../../migrate/concepts-assessment-calculation.md#types-of-assessments) basée sur les performances. Si cette option n’est pas disponible, consultez l’article suivant sur la création de votre propre [ligne de base pour les performances](https://azure.microsoft.com/services/virtual-machines/sql-server/).

Il convient également de réfléchir à l’installation et à la configuration correctes de SQL Server sur une machine virtuelle. Il est recommandé d’utiliser la [galerie d’images de machine virtuelle Azure SQL](../../virtual-machines/windows/create-sql-vm-portal.md), car cela vous permet de créer une machine virtuelle SQL Server avec la version, l’édition et le système d’exploitation appropriés. Cela permettra également d’inscrire automatiquement la machine virtuelle Azure auprès du [fournisseur de ressources](../../virtual-machines/windows/create-sql-vm-portal.md) SQL Server, activant ainsi des fonctionnalités telles que les sauvegardes automatisées et la mise à jour corrective automatisée.

## <a name="migration-strategies"></a>Stratégies de migration

Il existe deux stratégies de migration pour migrer vos bases de données utilisateur vers une instance SQL sur des machines virtuelles Azure : **migrer** et **lift-and-shift**. 

L’approche appropriée pour votre entreprise dépend généralement des facteurs suivants : 

- Taille et échelle de la migration
- Vitesse de migration
- Prise en charge des applications pour la modification du code
- Nécessité de changer de version de SQL Server, de système d’exploitation ou les deux
- Cycle de vie de la prise en charge de vos produits existants
- Fenêtre de temps d’arrêt de l’application pendant la migration

:::image type="content" source="media/sql-server-to-sql-on-azure-vm-individual-databases-guide/virtual-machine-migration-downtime.png" alt-text="Temps d’arrêt de la migration des machines virtuelles":::

Le tableau suivant décrit les différences entre les deux stratégies de migration :
<br />

| **Stratégie de migration** | **Description** | **Quand utiliser** |
| --- | --- | --- |
| **Lift-and-shift** | Utilisez la stratégie de migration Lift-and-shift pour déplacer l’intégralité des serveurs SQL physiques ou virtuels de leur emplacement actuel vers une instance SQL sur une machine virtuelle Azure sans aucune modification du système d’exploitation ou de la version de SQL Server. Pour effectuer une migration Lift-and-shift, consultez [Azure Migrate](../../../migrate/migrate-services-overview.md). <br /><br /> Le serveur source reste en ligne et traite les requêtes, tandis que le serveur source et le serveur de destination synchronisent les données, ce qui permet une migration presque transparente. | À utiliser pour les migrations simples aux migrations à très grande échelle, applicable aussi aux scénarios comme la sortie d’un centre de données. <br /><br /> Modifications de code minimales ou nulles requises pour les bases de données ou les applications SQL des utilisateurs, ce qui permet des migrations globales plus rapides. <br /><br />Aucune étape supplémentaire n’est requise pour la migration des services décisionnels tels que [SSIS](/sql/integration-services/sql-server-integration-services), [SSRS](/sql/reporting-services/create-deploy-and-manage-mobile-and-paginated-reports) et [SSAS](/analysis-services/analysis-services-overview). |
|**Migrer** | Utilisez une stratégie de migration lorsque vous souhaitez mettre à niveau la version cible du serveur SQL et/ou du système d’exploitation. <br /> <br /> Sélectionnez une machine virtuelle Azure sur Place de marché Azure ou une image SQL Server préparée qui correspond à la version source de SQL Server. | À utiliser lorsqu’il est nécessaire ou souhaitable d’utiliser des fonctionnalités disponibles dans des versions plus récentes de SQL Server, ou lorsqu’il est nécessaire de mettre à niveau les versions héritées de SQL Server et/ou du système d’exploitation qui ne sont plus prises en charge.  <br /> <br /> Peut nécessiter des modifications de l’application ou de la base de données utilisateur pour prendre en charge la mise à niveau du serveur SQL. <br /><br />Il peut y avoir des considérations supplémentaires relatives à la migration des [services décisionnels](#business-intelligence) s’ils s’inscrivent dans le cadre de la migration. |


## <a name="lift-and-shift"></a>Migration lift-and-shift  

Le tableau suivant détaille la méthode disponible pour la stratégie de migration **Lift-and-shift** pour migrer votre base de données SQL Server vers SQL Server sur les machines virtuelles Azure :
<br />

|**Méthode** | **Version source minimale** | **Version cible minimale** | **Contrainte de taille de la sauvegarde source** |  **Remarques** |
| --- | --- | --- | --- | --- |
| [Azure Migrate](../../../migrate/index.yml) | SQL Server 2008 SP4| SQL Server 2008 SP4| [Limite de stockage de machine virtuelle Azure](../../../index.yml) |  Le serveur SQL existant doit être déplacé tel quel vers une instance SQL sur une machine virtuelle Azure. Peut mettre à l’échelle des charges de travail de migration allant jusqu’à 35 000 machines virtuelles. <br /><br /> Les serveurs sources restent en ligne et traitent les requêtes pendant la synchronisation des données du serveur, réduisant ainsi au minimum les temps d’arrêt. <br /><br /> **Automatisation et scripts** : [scripts Azure Site Recovery](../../../migrate/how-to-migrate-at-scale.md) et [exemple de migration et de planification mises à l’échelle pour Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale).|

## <a name="migrate"></a>Migrer  

En raison de sa facilité de configuration, l’approche de migration recommandée consiste à effectuer localement une [sauvegarde](/sql/t-sql/statements/backup-transact-sql) native de SQL Server puis à copier le fichier sur Azure. Cette méthode prend en charge des bases de données plus volumineuses (> 1 To) pour toutes les versions de SQL Server à partir de 2008 et des sauvegardes de base de données plus volumineuses (> 1 To). Toutefois, pour les bases de données à partir de SQL Server 2014, dont la taille est inférieure à 1 To et qui ont une bonne connectivité à Azure, [la sauvegarde SQL Server vers une URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url) est la meilleure approche. 

Lors de la migration de bases de données SQL Server vers une instance SQL sur des machines virtuelles Azure, il est important de choisir une approche adaptée lorsque vous devez basculer vers le serveur cible, car cela influe sur la fenêtre de temps d’arrêt de l’application.

Le tableau suivant détaille toutes les méthodes disponibles pour migrer votre base de données SQL Server vers SQL Server sur les machines virtuelles Azure :
<br />

|**Méthode** | **Version source minimale** | **Version cible minimale** | **Contrainte de taille de la sauvegarde source** | **Remarques** |
| --- | --- | --- | --- | --- |
| **[Sauvegarde dans un fichier](sql-server-to-sql-on-azure-vm-individual-databases-guide.md#migrate)** | SQL Server 2008 SP4 | SQL Server 2008 SP4| [Limite de stockage de machine virtuelle Azure](../../../index.yml) |  Il s’agit d’une technique simple et éprouvée pour déplacer des bases de données d’une machine à l’autre. Utilisez la compression afin de réduire la taille de la sauvegarde pour le transfert. <br /><br /> **Automatisation et scripts** : [Transact-SQL (T-SQL)](/sql/t-sql/statements/backup-transact-sql) et [AzCopy dans Stockage Blob](../../../storage/common/storage-use-azcopy-v10.md)  |
| **[Sauvegarde vers une URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url)** | SQL Server 2012 SP1 CU2 | SQL Server 2012 SP1 CU2| 12,8 To pour SQL Server 2016, sinon 1 To | Autre façon de déplacer le fichier de sauvegarde sur la machine virtuelle à l’aide du stockage Azure. Utilisez la compression afin de réduire la taille de la sauvegarde pour le transfert. <br /><br /> **Automatisation et scripts** :  [T-SQL ou plan de maintenance](/sql/relational-databases/backup-restore/sql-server-backup-to-url) |
| **[Assistant Migration de données Microsoft (DMA)](/sql/dma/dma-overview)** | SQL Server 2005| SQL Server 2008 SP4| [Limite de stockage de machine virtuelle Azure](../../../index.yml) |  L’[Assistant Migration de données](/sql/dma/dma-overview) évalue SQL Server localement, puis effectue une mise à niveau en toute transparence vers les versions ultérieures de SQL Server ou migre vers SQL Server sur les machines virtuelles Azure, Azure SQL Database ou Azure SQL Managed Instance. <br /><br /> Ne doit pas être utilisée sur des bases de données utilisateur compatibles le flux de fichier.<br /><br /> L’Assistant Migration de données offre également la possibilité de migrer des [identifiants SQL et Windows](/sql/dma/dma-migrateserverlogins) et d’évaluer des [packages SSIS](/sql/dma/dma-assess-ssis). <br /><br /> **Automatisation et scripts** : [Interface de ligne de commande](/sql/dma/dma-commandline) |
| **[Détachement et attachement](../../virtual-machines/windows/migrate-to-vm-from-sql-server.md#detach-and-attach-from-a-url)** | SQL Server 2008 SP4 | SQL Server 2014 | [Limite de stockage de machine virtuelle Azure](../../../index.yml) | Utilisez cette méthode quand vous envisagez de [stocker ces fichiers à l’aide du service Stockage Blob Azure](/sql/relational-databases/databases/sql-server-data-files-in-microsoft-azure) et de les attacher à une instance SQL sur une machine virtuelle Azure, particulièrement utile avec les bases de données très volumineuses ou lorsque le temps de sauvegarde et de restauration est trop long. <br /><br /> **Automatisation et scripts** :  [T-SQL](/sql/relational-databases/databases/detach-a-database#TsqlProcedure) et [AzCopy dans Stockage Blob](../../../storage/common/storage-use-azcopy-v10.md)|
|**[Copie des journaux de transaction](sql-server-to-sql-on-azure-vm-individual-databases-guide.md#migrate)** | SQL Server 2008 SP4 (Windows uniquement) | SQL Server 2008 SP4 (Windows uniquement) | [Limite de stockage de machine virtuelle Azure](../../../index.yml) | La copie des journaux de transaction réplique les fichiers journaux de transactions locaux dans une instance SQL sur une machine virtuelle Azure. <br /><br /> Cela permet de réduire au minimum les temps d’arrêt pendant le basculement et de réduire les frais de configuration par rapport à la configuration d’un groupe de disponibilité Always On. <br /><br /> **Automatisation et scripts** : [T-SQL](/sql/database-engine/log-shipping/log-shipping-tables-and-stored-procedures)  |
| **[Groupe de disponibilité distribué](../../virtual-machines/windows/business-continuity-high-availability-disaster-recovery-hadr-overview.md#hybrid-it-disaster-recovery-solutions)** | SQL Server 2016| SQL Server 2016 | [Limite de stockage de machine virtuelle Azure](../../../index.yml) |  Un [groupe de disponibilité distribué](/sql/database-engine/availability-groups/windows/distributed-availability-groups) est un type spécial de groupe de disponibilité qui englobe deux groupes de disponibilité distincts. Les groupes de disponibilité qui participent à un groupe de disponibilité distribué n’ont pas besoin de se trouver au même emplacement et incluent la prise en charge interdomaine. <br /><br /> Cette méthode réduit le temps d’arrêt, à utiliser quand un groupe de disponibilité est configuré localement. <br /><br /> **Automatisation et scripts** : [T-SQL](/sql/t-sql/statements/alter-availability-group-transact-sql)  |
| | | | | |

&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp;
 
> [!TIP]
> Pour les transferts de données volumineux avec des options de réseau limitées ou inexistantes, consultez [Transferts de données volumineux avec une connectivité limitée](../../../storage/common/storage-solution-large-dataset-low-network.md).
> 

### <a name="considerations"></a>Considérations

La liste suivante répertorie les points clés à prendre en compte lors de l’examen des méthodes de migration :

- Pour obtenir des performances optimales de transfert de données, migrez les bases de données et les fichiers vers une instance SQL sur une machine virtuelle Azure à l’aide d’un fichier de sauvegarde compressé. Pour les bases de données plus volumineuses, en plus de la compression, [fractionnez le fichier de sauvegarde en fichiers plus petits](/sql/relational-databases/backup-restore/back-up-files-and-filegroups-sql-server) pour accroître les performances lors de la sauvegarde et du transfert. 
- Si vous effectuez une migration à partir de SQL Server 2014 ou une version ultérieure, envisagez de [chiffrer les sauvegardes](/sql/relational-databases/backup-restore/backup-encryption) pour protéger les données lors du transfert sur le réseau.
- Pour réduire le temps d’arrêt lors de la migration de base de données, utilisez l’option de groupe de disponibilité AlwaysOn. 
- Pour réduire le temps d’arrêt sans encourir de frais supplémentaires liés à la configuration d’un groupe de disponibilité, utilisez l’option de copie des journaux de transaction. 
- Pour les options de réseau limitées ou inexistantes, utilisez des méthodes de migration hors connexion, telles que la sauvegarde et la restauration, ou [les services de transfert de disque](../../../storage/common/storage-solution-large-dataset-low-network.md) disponibles dans Azure.
- Pour modifier également la version de SQL Server sur SQL Server sur une machine virtuelle Azure, consultez [Modifier l’édition de SQL Server](../../virtual-machines/windows/change-sql-server-edition.md).

## <a name="business-intelligence"></a>Business Intelligence 

Il peut y avoir des considérations supplémentaires lors de la migration de services décisionnels SQL Server en dehors du cadre des migrations de base de données utilisateur. 

Ces services comprennent :

- [**SQL Server Integration Services (SSIS)**](/sql/integration-services/install-windows/upgrade-integration-services)
- [**SQL Server Reporting Services (SSRS)**](/sql/reporting-services/install-windows/upgrade-and-migrate-reporting-services)
- [**SQL Server Analysis Services (SSAS)**](/sql/database-engine/install-windows/upgrade-analysis-services)

## <a name="supported-versions"></a>Versions prises en charge

Lorsque vous vous préparez à migrer des bases de données SQL Server vers SQL Server sur les machines virtuelles Azure, veillez à prendre en compte les versions de SQL Server qui sont prises en charge. Pour obtenir la liste des versions de SQL Server actuellement prises en charge sur les machines virtuelles Azure, consultez [SQL Server sur les machines virtuelles Azure](../../virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md#get-started-with-sql-server-vms).


## <a name="next-steps"></a>Étapes suivantes

Pour commencer la migration de vos bases de données SQL Server vers SQL Server sur les machines virtuelles Azure, consultez le [guide de migration de base de données individuelle](sql-server-to-sql-on-azure-vm-individual-databases-guide.md). 

- Si vous souhaitez obtenir une matrice des services et outils Microsoft et tiers qui peuvent vous aider dans différents scénarios de migration de données et de base de données ainsi que pour des tâches spécialisées, consultez l’article [Services et outils disponibles pour la migration de données](../../../dms/dms-tools-matrix.md).

- Pour en savoir plus sur Azure SQL, consultez :
   - [Options de déploiement](../../azure-sql-iaas-vs-paas-what-is-overview.md)
   - [SQL Server sur machines virtuelles Azure](../../virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md)
   - [Calculatrice du coût total de possession Azure](https://azure.microsoft.com/pricing/tco/calculator/) 


- Pour plus d’informations sur l’infrastructure et le cycle d’adoption pour les migrations cloud, consultez :
   -  [Cloud Adoption Framework pour Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Meilleures pratiques pour l’évaluation des coûts et le dimensionnement des charges de travail migrées vers Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 

- Pour plus d’informations sur la gestion des licences, consultez :
   - [BYOL (apportez votre propre licence) avec Azure Hybrid Benefit](../../virtual-machines/windows/licensing-model-azure-hybrid-benefit-ahb-change.md)
   - [Bénéficier d’un support étendu gratuit pour SQL Server 2008 et SQL Server 2008 R2](../../virtual-machines/windows/sql-server-2008-extend-end-of-support.md)


- Pour évaluer la couche d’accès aux applications, consultez [Data Access Migration Toolkit (préversion)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit).
- Pour savoir comment exécuter un test A/B sur la couche d’accès aux données, consultez [Assistant Expérimentation de base de données](/sql/dea/database-experimentation-assistant-overview).