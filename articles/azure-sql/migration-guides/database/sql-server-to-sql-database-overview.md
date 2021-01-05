---
title: 'De SQL Server vers SQL Database : Vue d’ensemble de la migration'
description: Découvrez les différents outils et options à disposition pour migrer vos bases de données SQL Server vers Azure SQL Database.
ms.service: sql-database
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: mokabiru
ms.author: mokabiru
ms.reviewer: MashaMSFT
ms.date: 11/06/2020
ms.openlocfilehash: 2d668775e8d11faa0b2913cebc41e5217b49c278
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97357323"
---
# <a name="migration-overview-sql-server-to-sql-database"></a>Vue d’ensemble de la migration : de SQL Server vers SQL Database
[!INCLUDE[appliesto--sqldb](../../includes/appliesto-sqldb.md)]

Découvrez les différentes options de migration et les éléments à prendre en considération pour migrer SQL Server vers Azure SQL Database. 

Vous pouvez migrer les bases de données SQL Server s’exécutant en local ou sur : 

- SQL Server sur les machines virtuelles  
- Amazon Web Services (AWS) EC2 
- Amazon Relational Database Service (AWS RDS) 
- Compute Engine (Google Cloud Platform – GCP)  
- Cloud SQL pour SQL Server (Google Cloud Platform – GCP) 

Pour d’autres scénarios, consultez le [guide de migration de bases de données](https://datamigration.microsoft.com/). 

## <a name="overview"></a>Vue d’ensemble

[Azure SQL Database](../../database/sql-database-paas-overview.md) est une option cible recommandée pour les charges de travail SQL Server qui nécessitent une plateforme en tant que service (PaaS) entièrement gérée. SQL Database gère la plupart des fonctions de gestion de base de données ainsi que les fonctionnalités de haute disponibilité, de traitement intelligent des requêtes, de scalabilité et de performances intégrées afin de s’adapter à divers types d’applications différents. 

SQL Database offre une certaine flexibilité avec plusieurs [modèles de déploiement](../../database/sql-database-paas-overview.md#deployment-models) et [niveaux de service](../../database/service-tiers-vcore.md#service-tiers) qui prennent en charge différents types d’applications ou de charges de travail.

L’un des principaux avantages que vous pouvez tirer d’une migration vers SQL Database est la possibilité de moderniser votre application en tirant parti des capacités PaaS et d’éliminer toute dépendance vis-à-vis des composants techniques limités au niveau de l’instance comme les travaux SQL Agent.

Vous pouvez aussi réaliser des économies en migrant vos licences locales SQL Server vers Azure SQL Database en utilisant [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) pour SQL Server, à condition d’opter pour le [modèle d’achat vCore](../../database/service-tiers-vcore.md).

Ce guide vise à clarifier les options de migration et les éléments à prendre en considération pendant que vous préparez la migration de vos bases de données SQL Server vers Azure SQL Database.  

## <a name="considerations"></a>Considérations 

Les facteurs clés à prendre en considération au moment d’évaluer les différentes options de migration dépendent des éléments suivants : 

- Nombre de serveurs et de bases de données
- Taille des bases de données
- Temps d’arrêt acceptable pour l’entreprise pendant le processus de migration 

Les options de migration listées dans ce guide prennent en compte ces facteurs. Pour une migration de données logiques vers Azure SQL Database, le temps de migration peut dépendre du nombre d’objets contenus dans une base de données et de la taille de la base de données. 

Il existe des outils pour différentes charges de travail et différentes préférences utilisateur. En effet, si certains outils permettent de migrer rapidement une base de données unique via une interface utilisateur, d’autres permettent de migrer plusieurs bases de données et peuvent être automatisées pour gérer les migrations à grande échelle. 

## <a name="choose-appropriate-target"></a>Choisir la cible appropriée

Tenez compte des recommandations générales pour choisir le modèle de déploiement et le niveau de service d’Azure SQL Database qui conviennent le mieux. Vous pouvez choisir des ressources de calcul et de stockage pendant le déploiement et en changer par la suite via le [portail Azure](../../database/scale-resources.md) sans temps d’arrêt pour votre application.


**Modèles de déploiement** : il est important de bien comprendre la charge de travail de votre application et le modèle d’utilisation avant de vous décider entre une base de données unique ou un pool élastique. 

- Une [base de données unique](../../database/single-database-overview.md) est une base de données entièrement gérée qui convient pour la plupart des microservices et applications cloud modernes.
- Un [pool élastique](../../database/elastic-pool-overview.md) est un ensemble de bases de données uniques qui partagent un ensemble de ressources comme le processeur ou la mémoire ; il permet de combiner des bases de données dans un pool avec des modèles d’utilisation prévisibles qui peut partager efficacement un même ensemble de ressources.

**Modèles d’achat** : vous avez le choix entre les modèles d’achat vCore, DTU ou serverless. 

- Le [modèle vCore](../../database/service-tiers-vcore.md) vous permet de choisir le nombre de vCores pour votre instance Azure SQL Database, ce qui en fait le choix le plus simple quand il s’agit d’opérer une transition à partir de SQL Server en local. Cette option est la seule à permettre de réaliser des économies sur le coût de licence grâce à [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/). 
- Le [modèle DTU](../../database/service-tiers-dtu.md) extrait les ressources sous-jacentes de calcul, de mémoire et d’E/S afin d’offrir une unité de transaction de base de données (DTU) composite. 
- Le [modèle serverless](../../database/serverless-tier-overview.md) s’adresse aux charges de travail qui requièrent une mise à l’échelle automatique à la demande avec des ressources de calcul facturées par seconde d’utilisation. Le niveau de calcul serverless suspend automatiquement les bases de données pendant les périodes d’inactivité (seul le stockage est facturé) et relance automatiquement leur exécution dès que l’activité reprend. 

**Niveaux de service** : effectuez un choix parmi trois niveaux de service conçus pour différents types d'applications.

- Le [niveau de service Usage général/Standard](../../database/service-tier-general-purpose.md) est une option économique et équilibrée dont les capacités de calcul et de stockage sont adaptées aux applications de niveau intermédiaire/inférieur, avec des capacités de redondance intégrées au niveau de la couche de stockage pour récupérer après des défaillances. Conçu pour la plupart des charges de travail de base de données. 
- Le [niveau de service Critique pour l’entreprise/Premium](../../database/service-tier-business-critical.md) s’adresse aux applications de niveau élevé qui nécessitent des taux de transaction élevés, des E/S à faible latence et un haut niveau de résilience avec des réplicas secondaires disponibles à la fois pour le basculement et pour le déchargement des charges de travail en lecture.
- Le [niveau de service Hyperscale](../../database/service-tier-hyperscale.md) s’adresse aux bases de données qui ont des volumes de données croissants et qui doivent faire l’objet d’un scale-up automatique pouvant atteindre une taille de base de données de 100 To. Conçu pour les bases de données très volumineuses. 

> [!IMPORTANT]
> Le [taux de journalisation des transactions est régi](../../database/resource-limits-logical-server.md#transaction-log-rate-governance) par Azure SQL Database pour limiter les taux d’ingestion élevés. À cet effet, pendant la migration, il peut s’avérer nécessaire de mettre à l’échelle les ressources de la base de données cible (vCores/DTU) pour alléger la pression sur le processeur ou le débit. Choisissez une base de données cible de taille appropriée, mais prévoyez si nécessaire de procéder à un scale-up des ressources pour la migration. 


### <a name="sql-server-on-azure-vm-alternative"></a>SQL Server sur des machines virtuelles Azure en solution de remplacement

Votre entreprise a peut-être des exigences qui font de [SQL Server sur machines virtuelles Azure](../../virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md) une cible plus indiquée qu’Azure SQL Database. 

Si les conditions suivantes s’appliquent à votre entreprise, envisagez d’opter pour la solution SQL Server sur machines virtuelle Azure : 

- Si vous avez besoin d’un accès direct au système d’exploitation ou au système de fichiers, par exemple pour installer des agents tiers ou personnalisés sur une même machine virtuelle avec SQL Server. 
- Si vous dépendez obligatoirement de fonctionnalités qui ne sont pas encore prises en charge, comme FileStream/FileTable, PolyBase et les transactions entre plusieurs instances. 
- Si vous devez absolument conserver une version spécifique de SQL Server (2012, par exemple). 
- Si vos exigences de calcul sont beaucoup plus faibles que ce qu’offre l’instance gérée (un seul vCore, par exemple) et que l’option de consolidation de bases de données n’est pas une option acceptable. 


## <a name="migration-tools"></a>Outils de migration 

Les outils recommandés pour la migration sont l’Assistant Migration de données et Azure Database Migration Service. D’autres options de migration sont également disponibles. 

### <a name="recommended-tools"></a>Outils recommandés

Le tableau suivant liste les outils de migration recommandés : 

|Technology | Description|
|---------|---------|
|[Assistant Migration de données (DMA)](/sql/dma/dma-migrateonpremsqltosqldb)|L’Assistant Migration de données est un outil de bureau qui effectue des évaluations homogènes de SQL Server et des migrations vers Azure SQL Database (à la fois le schéma et les données). L’outil peut être installé sur un serveur local ou sur votre ordinateur local, sous réserve qu’il dispose d’une connectivité avec vos bases de données sources. Le processus de migration est un déplacement de données logiques entre les objets de la base de données source et ceux de la base de données cible. </br> - Migration de bases de données uniques (à la fois le schéma et les données)|
|[Azure Database Migration Service (DMS)](../../../dms/tutorial-sql-server-to-azure-sql.md)|Service Azure interne vous permettant de migrer vos bases de données SQL Server vers Azure SQL Database à partir du portail Azure ou de manière automatisée avec PowerShell. Azure DMS vous impose de sélectionner un réseau virtuel Azure par défaut pendant le provisionnement pour vérifier qu’il existe une connectivité avec vos bases de données SQL Server sources. </br> - Migration de bases de données uniques ou à grande échelle. |
| | |


### <a name="alternative-tools"></a>Autres outils

Le tableau suivant liste d’autres outils de migration : 

|Technology |Description  |
|---------|---------|
|[Réplication transactionnelle](../../database/replication-to-sql-database.md)|Réplique les données des tables de la base de données SQL Server source dans Azure SQL Database en proposant une option de migration de type publication-abonnement tout en préservant la cohérence transactionnelle. Les modifications de données incrémentielles sont propagées aux abonnés à mesure qu’elles se produisent sur les serveurs de publication.|
|[Service d’importation/exportation / BACPAC](../../database/database-import.md)|[BACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac) est un fichier Windows avec une extension `.bacpac` qui encapsule le schéma et les données d’une base de données. BACPAC permet d’exporter des données à partir d’une source SQL Server et d’importer des données dans Azure SQL Database. Le fichier BACPAC peut être importé dans une nouvelle instance Azure SQL Database à partir du portail Azure. </br></br> Pour des raisons d’échelle et de performances, si vous disposez de bases de données volumineuses ou en nombre, envisagez d’utiliser l’utilitaire de ligne de commande [SqlPackage](../../database/database-import.md#using-sqlpackage) pour exporter et importer des bases de données.|
|[Copie en bloc](/sql/relational-databases/import-export/import-and-export-bulk-data-by-using-the-bcp-utility-sql-server)|L’[utilitaire bcp (bulk copy program)](/sql/tools/bcp-utility) permet de copier les données d’une instance de SQL Server dans un fichier de données. Utilisez l’utilitaire BCP pour exporter les données de votre source et importer le fichier de données dans l’instance Azure SQL Database cible. </br></br> Pour profiter d’opérations de copie en bloc à haut débit lors du déplacement de données vers Azure SQL Database, l’[outil Smart Bulk Copy](/samples/azure-samples/smartbulkcopy/smart-bulk-copy/) peut être utilisé pour optimiser la vitesse de transfert en tirant parti des tâches de copie en parallèle.|
|[Azure Data Factory (ADF)](../../../data-factory/connector-azure-sql-database.md)|L’[activité de copie](../../../data-factory/copy-activity-overview.md) d’Azure Data Factory migre les données des bases de données SQL Server sources vers Azure SQL Database à l’aide de connecteurs intégrés et d’un [runtime d’intégration](../../../data-factory/concepts-integration-runtime.md).</br> </br> ADF prend en charge un large éventail de [connecteurs](../../../data-factory/connector-overview.md) pour déplacer les données de sources SQL Server vers Azure SQL Database.|
|[Synchronisation des données SQL](../../database/sql-data-sync-data-sql-server-sql-database.md)|SQL Data Sync est un service basé sur Azure SQL Database qui vous permet de synchroniser les données choisies de manière bidirectionnelle sur plusieurs bases de données, à la fois locales et dans le cloud.</br>Data Sync est utile dans les cas où les données doivent être tenues à jour entre plusieurs bases de données dans Azure SQL Database ou SQL Server.|
| | |

## <a name="compare-migration-options"></a>Comparer les options de migration

Comparez les options de migration de façon à emprunter la voie qui vous permettra de répondre à vos besoins métier. 

### <a name="recommended-options"></a>Options recommandées

Le tableau suivant compare les options de migration recommandées : 

|Option de migration  |Quand l’utiliser  |Considérations  |
|---------|---------|---------|
|[Assistant Migration de données (DMA)](/sql/dma/dma-migrateonpremsqltosqldb) | - Migration de bases de données uniques (à la fois le schéma et les données).  </br> - Acceptation de temps d’arrêt pendant le processus de migration des données. </br> </br> Sources prises en charge : </br> - Serveur SQL (2005 à 2019) en local ou machine virtuelle Azure </br> - AWS EC2 </br> - AWS RDS </br> - Machine virtuelle SQL Server GCP Compute | - L’activité de migration procède à des déplacements de données entre objets de base de données (de la source vers la cible), d’où la recommandation de l’exécuter durant les heures creuses. </br> - L’Assistant Migration de données (DMA) indique l’état de la migration par objet de base de données, notamment le nombre de lignes migrées.  </br> - Pour les migrations de grande ampleur pour ce qui est du nombre de bases de données ou de la taille de base de données, utilisez Azure Database Migration Service listé ci-dessous.|
|[Azure Database Migration Service (DMS)](../../../dms/tutorial-sql-server-to-azure-sql.md)| - Migration de bases de données uniques ou à grande échelle. </br> - Acceptation de temps d’arrêt pendant le processus de migration. </br> </br> Sources prises en charge : </br> - Serveur SQL (2005 à 2019) en local ou machine virtuelle Azure </br> - AWS EC2 </br> - AWS RDS </br> - Machine virtuelle SQL Server GCP Compute | - Les migrations à grande échelle peuvent être automatisées via [PowerShell](../../../dms/howto-sql-server-to-azure-sql-powershell.md). </br> - La durée de la migration dépend de la taille de la base de données et du nombre d’objets qu’elle contient. </br> - Impose une définition de la base de données source en mode Lecture seule. |
| | | |

### <a name="alternative-options"></a>Autres options

Le tableau suivant compare les autres options de migration : 

|Méthode/technologique |Quand l’utiliser    |Considérations  |
|---------|---------|---------|
|[Réplication transactionnelle](../../database/replication-to-sql-database.md)| - Migration avec une publication continue des modifications des tables de base de données sources vers les tables Azure SQL Database cibles. </br> - Migration complète ou partielle des tables sélectionnées (sous-ensemble d’une base de données).  </br> </br> Sources prises en charge : </br> - [SQL Server (2016 à 2019) avec certaines limitations](/sql/relational-databases/replication/replication-backward-compatibility) </br> - AWS EC2  </br> - Machine virtuelle SQL Server GCP Compute  | - La configuration est relativement complexe par rapport à d’autres options de migration.   </br> - Offre une option de réplication continue pour migrer les données (sans mettre les bases de données en mode hors connexion).  </br> - La réplication transactionnelle présente plusieurs limitations dont il faut tenir compte au moment de configurer le serveur de publication sur l’instance SQL Server source. Pour plus d’informations, consultez [Limitations de la publication d’objets](/sql/relational-databases/replication/publish/publish-data-and-database-objects#limitations-on-publishing-objects). </br>- Il est possible de [superviser l’activité de réplication](/sql/relational-databases/replication/monitor/monitoring-replication).    |
|[Service d’importation/exportation / BACPAC](../../database/database-import.md)| - Migration de bases de données d’applications métier individuelles. </br>- Adapté aux bases de données de plus petite taille.  </br>  - Ne nécessite pas de service ou d’outil de migration distinct. </br> </br> Sources prises en charge : </br> - Serveur SQL (2005 à 2019) en local ou machine virtuelle Azure </br> - AWS EC2 </br> - AWS RDS </br> - Machine virtuelle SQL Server GCP Compute  |  - Impose un temps d’arrêt dans la mesure où les données doivent être exportées à la source et importées à destination.   </br> - Les formats de fichiers et les types de données utilisés dans l’exportation/importation doivent être en cohérence avec les schémas de table pour éviter des erreurs de troncation et/ou de non-correspondance de types de données.  </br> - L’exportation d’une base de données comportant un grand nombre d’objets peut prendre beaucoup plus de temps.       |
|[Copie en bloc](/sql/relational-databases/import-export/import-and-export-bulk-data-by-using-the-bcp-utility-sql-server)| - Migration complète ou partielle de données. </br> - Acceptation de temps d’arrêt. </br> </br> Sources prises en charge : </br> - Serveur SQL (2005 à 2019) en local ou machine virtuelle Azure </br> - AWS EC2 </br> - AWS RDS </br> - Machine virtuelle SQL Server GCP Compute   | - Impose un temps d’arrêt pour exporter les données de la source et les importer dans la cible. </br> - Les formats de fichiers et les types de données utilisés dans l’exportation/importation doivent être en cohérence avec les schémas de table. |
|[Azure Data Factory (ADF)](../../../data-factory/connector-azure-sql-database.md)| - Migration et/ou transformation des données à partir des bases de données SQL Server sources. </br> - Fusion de données issues de plusieurs sources de données vers Azure SQL Database, généralement pour des charges de travail de type décisionnel.  |  - Impose la création de pipelines de déplacement de données dans Azure Data Factory pour déplacer les données de la source vers la destination.   </br> Le - [coût](https://azure.microsoft.com/pricing/details/data-factory/data-pipeline/) est une considération importante qui varie en fonction des déclencheurs de pipeline, des exécutions d’activité, de la durée de déplacement des données, etc. |
|[Synchronisation des données SQL](../../database/sql-data-sync-data-sql-server-sql-database.md)| - Synchronisation des données entre les bases de données source et cible.</br> - Convient pour l’exécution d’une synchronisation continue entre Azure SQL Database et une instance SQL Server locale dans un flux bidirectionnel. | - Azure SQL Database doit être la base de données Hub pour la synchronisation avec une base de données SQL Server local en tant que base de données membre.</br> - En comparaison de la réplication transactionnelle, SQL Data Sync prend en charge la synchronisation bidirectionnelle des données entre un environnement local et Azure SQL Database. </br> -Selon la charge de travail, son impact sur le niveau de performance peut être plus élevé.|
| | | |

## <a name="feature-interoperability"></a>Interopérabilité des fonctionnalités 

D’autres éléments sont à prendre en considération quand il s’agit de migrer des charges de travail qui reposent sur d’autres fonctionnalités SQL Server.

#### <a name="sql-server-integration-services"></a>SQL Server Integration Services
Vous pouvez migrer des packages SQL Server Integration Services (SSIS) vers Azure en les redéployant sur le runtime Azure-SSIS dans [Azure Data Factory](../../../data-factory/introduction.md). Azure Data Factory [prend en charge la migration des packages SSIS](../../../data-factory/scenario-ssis-migration-overview.md#azure-sql-database-as-database-workload-destination) en offrant un runtime conçu pour exécuter les packages SSIS dans Azure. Une autre solution consiste à réécrire la logique ETL de SSIS en mode natif dans ADF en utilisant des [flux de données](../../../data-factory/concepts-data-flow-overview.md).


#### <a name="sql-server-reporting-services"></a>SQL Server Reporting Services
Migrez les rapports SQL Server Reporting Services (SSRS) vers des rapports paginés dans Power BI. Utilisez l’ [outil de migration RDL](https://github.com/microsoft/RdlMigration) pour préparer et migrer vos rapports. Cet outil a été développé par Microsoft pour aider les utilisateurs à effectuer la migration des rapports RDL entre leurs serveurs SSRS et Power BI. Il est disponible sur GitHub avec une documentation qui décrit de bout en bout chaque étape du scénario de migration. 

#### <a name="high-availability"></a>Haute disponibilité
La configuration manuelle des fonctionnalités de haute disponibilité SQL Server comme les instances de cluster de basculement Always On et les groupes de disponibilité Always On devient obsolète sur l’instance Azure SQL Database cible, car l’architecture à haute disponibilité est déjà intégrée dans les niveaux de service [Usage général (modèle de disponibilité Standard)](../../database/high-availability-sla.md#basic-standard-and-general-purpose-service-tier-locally-redundant-availability) et [Critique pour l’entreprise (modèle de disponibilité Premium)](../../database/high-availability-sla.md#premium-and-business-critical-service-tier-locally-redundant-availability) de SQL Database. Le niveau de service Critique pour l’entreprise/Premium propose également un scale-out en lecture qui permet de se connecter à l’un des nœuds secondaires pour des besoins de lecture seule. 

Au-delà de l’architecture de haute disponibilité incluse dans Azure SQL Database, il y a aussi la fonctionnalité des [groupes de basculement automatique ](../../database/auto-failover-group-overview.md) qui vous permet de gérer la réplication et le basculement des bases de données d’une instance gérée vers une autre région. 

#### <a name="sql-agent-jobs"></a>Travaux SQL Agent
Les travaux SQL Agent ne sont pas directement pris en charge dans Azure SQL Database et doivent être déployés dans des [tâches de base de données élastique (préversion)](../../database/job-automation-overview.md#elastic-database-jobs-preview).

#### <a name="logins-and-groups"></a>Connexions et groupes
Déplacez les connexions SQL de l’instance SQL Server source vers Azure SQL Database à l’aide de Database Migration Service (DMS) en mode hors connexion.  Utilisez le panneau **Connexions sélectionnées** dans l’**Assistant Migration** pour migrer les connexions vers votre instance SQL Database cible. 

Les utilisateurs et groupes Windows peuvent aussi être migrés à l’aide de DMS en activant le bouton bascule correspondant dans la page de configuration de DMS. 

Une autre solution consiste à utiliser l’[outil utilitaire PowerShell](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/MoveLogins) spécialement conçu par les architectes de migration de données Microsoft. Cet utilitaire s’appuie sur PowerShell pour créer un script Transact-SQL (T-SQL) en vue de recréer les connexions et de sélectionner les utilisateurs de base de données de la source vers la cible. L’outil mappe automatiquement les comptes Windows AD aux comptes Azure AD et peut effectuer une recherche de nom d’utilisateur principal (UPN) pour chaque connexion auprès de l’instance Active Directory source. L’outil scripte les rôles de serveur et de base de données personnalisés ainsi que l’appartenance à un rôle, le rôle de base de données et les autorisations utilisateur. Les bases de données autonomes ne sont pas encore prises en charge et seul un sous-ensemble des autorisations SQL Server possibles est scripté. 


#### <a name="system-databases"></a>Bases de données système
Pour Azure SQL Database, les seules bases de données système applicables sont [MASTER](/sql/relational-databases/databases/master-database) et tempdb. Pour plus d’informations, consultez [Tempdb dans Azure SQL Database](/sql/relational-databases/databases/tempdb-database#tempdb-database-in-sql-database).

## <a name="leverage-advanced-features"></a>Exploiter les fonctionnalités avancées 

Veillez à tirer parti des fonctionnalités cloud avancées offertes par SQL Database. Par exemple, vous n’avez plus à vous soucier de la gestion des sauvegardes, car le service le fait à votre place. Vous pouvez effectuer une restauration à n’importe quel [point dans le temps inclus dans la période de rétention](../../database/recovery-using-backups.md#point-in-time-restore). 

Pour renforcer la sécurité, envisagez d’utiliser l’ [authentification Azure Active Directory](../../database/authentication-aad-overview.md), l’[audit](../../database/auditing-overview.md), la  [détection des menaces](../../database/azure-defender-for-sql.md), la  [sécurité au niveau des lignes](/sql/relational-databases/security/row-level-security) et le  [masquage dynamique des données](/sql/relational-databases/security/dynamic-data-masking).

En plus des fonctionnalités de gestion et de sécurité avancées, Azure SQL Database propose un ensemble d’outils avancés qui peuvent vous aider à [superviser et ajuster votre charge de travail](../../database/monitor-tune-overview.md). [Azure SQL Analytics (préversion)](../../../azure-monitor/insights/azure-sql.md) est une solution cloud avancée qui permet de superviser dans une même vue les performances de toutes vos bases de données dans Azure SQL Database à grande échelle et sur plusieurs abonnements. Azure SQL Analytics collecte et visualise des métriques de performances clés à l’aide d’une intelligence intégrée pour résoudre les problèmes de performances.

Le [réglage automatique](/sql/relational-databases/automatic-tuning/automatic-tuning#automatic-plan-correction)  supervise en permanence les statistiques de performances de votre plan d’exécution SQL et corrige automatiquement les problèmes de performances identifiés. 


## <a name="migration-assets"></a>Ressources de migration 

Pour obtenir une aide supplémentaire, consultez les ressources suivantes qui ont été développées pour des projets de migration concrets.

|Asset  |Description  |
|---------|---------|
|[Outil et modèle d’évaluation d’une charge de travail de données](https://github.com/Microsoft/DataMigrationTeam/tree/master/Data%20Workload%20Assessment%20Model%20and%20Tool)| Cet outil fournit les plateformes cibles, la préparation du cloud et le niveau de correction des applications/bases de données « les mieux adaptés » pour une charge de travail donnée. Il propose des fonctionnalités de génération de rapports et de calculs simples en un clic qui permettent d’accélérer les évaluations d’un vaste domaine en fournissant un processus de décision de plateforme cible automatisé et uniforme.|
|[Utilitaire DBLoader](https://github.com/microsoft/DataMigrationTeam/tree/master/DBLoader%20Utility)|DBLoader permet de charger des données dans SQL Server à partir de fichiers texte délimités. Cet utilitaire de console Windows utilise l’interface de chargement en bloc du client natif SQL Server, qui fonctionne sur toutes les versions de SQL Server, dont Azure SQL Database.|
|[Création de bases de données en bloc avec PowerShell](https://github.com/Microsoft/DataMigrationTeam/tree/master/Bulk%20Database%20Creation%20with%20PowerShell)|Ce processus est constitué d’un jeu de trois scripts PowerShell qui créent un groupe de ressources (create_rg. ps1), le [serveur logique dans Azure](../../database/logical-servers.md) (create_sqlserver. ps1) et Azure SQL Database (create_sqldb. ps1). Sachant que les scripts incluent des fonctionnalités d’exécution en boucle, vous pouvez itérer et créer autant de serveurs et de bases de données que nécessaire.|
|[Déploiement de schémas en bloc avec MSSQL-Scripter et PowerShell](https://github.com/Microsoft/DataMigrationTeam/tree/master/Bulk%20Schema%20Deployment%20with%20MSSQL-Scripter%20&%20PowerShell)|Cette ressource crée un groupe de ressources, un ou plusieurs [serveurs logiques dans Azure](../../database/logical-servers.md) pour héberger Azure SQL Database, exporte chaque schéma à partir d’une instance SQL Server locale (ou de plusieurs instances SQL Server (2005+)), puis les importe dans Azure SQL Database.|
|[Convertir des travaux SQL Server Agent en tâches de base de données élastique](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/Convert%20SQL%20Server%20Agent%20Jobs%20into%20Elastic%20Database%20Jobs)|Ce script migre vos travaux SQL Server Agent sources vers des tâches de base de données élastique.|
|[Envoyer des messages à partir d’Azure SQL Database](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/AF%20SendMail)|Il s’agit d’une solution de remplacement à la fonctionnalité SendMail disponible dans les instances SQL Server locales. Elle s’appuie sur Azure Functions et le service Azure SendGrid pour envoyer des messages à partir d’Azure SQL Database.|
|[Utilitaire permettant de déplacer des connexions SQL Server locales vers Azure SQL Database](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/MoveLogins)|Script PowerShell qui crée un script de commande T-SQL visant à recréer les connexions et à sélectionner les utilisateurs de base de données d’une instance SQL Server locale vers Azure SQL Database. Cet outil permet de mapper automatiquement des comptes Windows AD à des comptes Azure AD et éventuellement de migrer des connexions natives SQL Server.|
|[Automatisation de la collecte des données PerfMon à l’aide de Logman](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/Perfmon%20Data%20Collection%20Automation%20Using%20Logman)|Outil qui collecte les données PerMon pour comprendre le niveau de performance de référence et aider avec des recommandations pour la cible de migration. Cet outil se sert de logman.exe pour créer la commande qui va créer, démarrer, arrêter et supprimer les compteurs de performances définis sur une instance SQL Server distante.|
|[Livre blanc – Migration de base de données vers Azure SQL DB avec BACPAC](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Database%20migrations%20-%20Benchmarks%20and%20Steps%20to%20Import%20to%20Azure%20SQL%20DB%20Single%20Database%20from%20BACPAC.pdf)|Ce livre blanc fournit les instructions et les étapes à suivre pour accélérer les migrations de SQL Server vers Azure SQL Database avec des fichiers BACPAC.|

Ces ressources ont été développées dans le cadre du programme Data SQL Ninja, qui est sponsorisé par l’équipe d’ingénierie Groupe de données Azure. La charte fondamentale du programme Data SQL Ninja a pour objet d’initier et d’accélérer une modernisation complexe et de faire face aux opportunités de migration de plateforme de données vers la plateforme de données Azure de Microsoft. Si vous pensez que votre organisation aimerait participer au programme Data SQL Ninja, contactez votre équipe en charge des compte et demandez-lui de soumettre une candidature.


## <a name="next-steps"></a>Étapes suivantes

Pour commencer la migration de votre instance SQL Server vers Azure SQL Database, consultez le [Guide de migration de SQL Server vers Azure SQL Database](sql-server-to-sql-database-guide.md).

- Pour obtenir une matrice des services et outils Microsoft et tiers qui peuvent vous aider dans les différents scénarios de migration de données et de base de données ainsi que dans des tâches spécialisées, consultez [Services et outils de migration de données](../../../dms/dms-tools-matrix.md).

- Pour plus d’informations sur Azure SQL Database, consultez :
   - [Vue d’ensemble d’Azure SQL Database](../../database/sql-database-paas-overview.md)
   - [Calculatrice du coût total de possession Azure](https://azure.microsoft.com/pricing/tco/calculator/) 

- Pour plus d’informations sur l’infrastructure et le cycle d’adoption pour les migrations cloud, consultez :
   -  [Cloud Adoption Framework pour Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Meilleures pratiques pour l’évaluation des coûts et le dimensionnement des charges de travail migrées vers Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 


- Pour évaluer la couche d’accès aux applications, consultez [Data Access Migration Toolkit (préversion)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit).
- Pour savoir comment exécuter un test A/B sur la couche d’accès aux données, consultez [Assistant Expérimentation de base de données](/sql/dea/database-experimentation-assistant-overview).