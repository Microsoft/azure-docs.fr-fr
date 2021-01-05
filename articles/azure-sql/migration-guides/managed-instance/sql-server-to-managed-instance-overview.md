---
title: Vue d’ensemble de la migration SQL Server vers SQL Managed Instance
description: Découvrez les différents outils et options à disposition pour migrer vos bases de données SQL Server vers Azure SQL Managed Instance.
ms.service: sql-managed-instance
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: mokabiru
ms.author: mokabiru
ms.reviewer: MashaMSFT
ms.date: 11/06/2020
ms.openlocfilehash: 878d59445dde839ca9e702ac0c49af676e48a42f
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96531188"
---
# <a name="migration-overview-sql-server-to-sql-managed-instance"></a>Vue d’ensemble de la migration : de SQL Server vers SQL Managed Instance
[!INCLUDE[appliesto--sqlmi](../../includes/appliesto-sqlmi.md)]

Découvrez les différentes options de migration et les éléments à prendre en considération pour migrer SQL Server vers Azure SQL Managed Instance. 

Vous pouvez migrer SQL Server s’exécutant en local ou sur : 

- SQL Server sur les machines virtuelles  
- Amazon Web Services (AWS) EC2 
- Amazon Relational Database Service (AWS RDS) 
- Compute Engine (Google Cloud Platform – GCP)  
- Cloud SQL pour SQL Server (Google Cloud Platform – GCP) 

Pour d’autres scénarios, consultez le [guide de migration de bases de données](https://datamigration.microsoft.com/). 

## <a name="overview"></a>Vue d’ensemble

[Azure SQL Managed Instance](../../managed-instance/sql-managed-instance-paas-overview.md) est une option cible recommandée pour les charges de travail SQL Server qui requièrent un service complètement managé sans avoir à gérer des machines virtuelles ou leurs systèmes d’exploitation. SQL Managed Instance vous permet d’effectuer un lift-and-shift de vos applications locales vers Azure avec des modifications minimales des applications ou des bases de données tout en isolant complètement vos instances grâce à la prise en charge native du réseau virtuel. 

## <a name="considerations"></a>Considérations 

Les facteurs clés à prendre en considération au moment d’évaluer les différentes options de migration dépendent des éléments suivants : 
- Nombre de serveurs et de bases de données
- Taille des bases de données
- Temps d’arrêt acceptable pour l’entreprise pendant le processus de migration 

L’un des principaux avantages de la migration de SQL Server vers SQL Managed Instance est que vous pouvez choisir de migrer la totalité de l’instance ou simplement un sous-ensemble de bases de données individuelles. Prévoyez d’inclure les éléments suivants dans votre processus de migration : 
- Toutes les bases de données qui doivent être colocalisées dans la même instance 
- Les objets au niveau de l’instance dont votre application dépend, notamment les connexions, les informations d’identification, les travaux et opérateurs SQL Agent, ainsi que les déclencheurs au niveau du serveur. 

> [!NOTE]
> Azure SQL Managed Instance garantit une disponibilité de 99,99 % même dans les scénarios critiques, de sorte que les frais supplémentaires engendrés par certaines fonctionnalités de SQL MI ne peut pas être désactivés. Pour plus d’informations, consultez le blog [Causes racines susceptibles de provoquer des différences de performances sur SQL Server et Azure SQL Managed Instance](https://azure.microsoft.com/blog/key-causes-of-performance-differences-between-sql-managed-instance-and-sql-server/). 


## <a name="choose-appropriate-target"></a>Choisir la cible appropriée

Voici quelques lignes directrices générales pour vous aider à choisir le bon niveau de service et les caractéristiques de SQL Managed Instance afin d’atteindre votre [ligne de base en matière de performances](sql-server-to-managed-instance-performance-baseline.md) : 

- Utilisez la ligne de base d’utilisation de l’UC pour approvisionner une instance gérée qui correspond au nombre de cœurs utilisés par votre instance SQL Server. Il peut être nécessaire de mettre à l’échelle les ressources pour qu’elles correspondent aux [caractéristiques de la génération du matériel](../../managed-instance/resource-limits.md#hardware-generation-characteristics). 
- Utilisez la ligne de base d’utilisation de la mémoire pour choisir une [option vCore](../../managed-instance/resource-limits.md#service-tier-characteristics) qui correspond de manière appropriée à votre allocation de mémoire. 
- Utilisez la latence d’E/S de ligne de base du sous-système de fichiers pour choisir entre les niveaux de service Usage général (latence supérieure à 5 ms) et Critique pour l’entreprise (latence inférieure à 3 ms). 
- Utilisez le débit de ligne de base pour préallouer la taille des fichiers de données et des fichiers journaux pour atteindre les performances d’E/S attendues. 

Vous pouvez choisir des ressources de calcul et de stockage pendant le déploiement et en changer par la suite via le [portail Azure](../../database/scale-resources.md) sans temps d’arrêt pour votre application. 

> [!IMPORTANT]
> Toute différence dans la [configuration requise du réseau virtuel d’instances gérées](../../managed-instance/connectivity-architecture-overview.md#network-requirements) peut vous empêcher de créer de nouvelles instances ou d’utiliser celles qui existent déjà. En savoir plus sur la [création de nouveaux réseaux](../../managed-instance/virtual-network-subnet-create-arm-template.md) et sur la [configuration des réseaux existants](../../managed-instance/vnet-existing-add-subnet.md?branch=release-ignite-arc-data). 

### <a name="sql-server-vm-alternative"></a>Alternative à la machine virtuelle SQL Server

Votre entreprise a peut-être des exigences qui font de SQL Server sur machines virtuelles Azure une cible plus indiquée qu’Azure SQL Managed Instance. 

Si les conditions suivantes s’appliquent à votre entreprise, envisagez d’opter pour une machine virtuelle SQL Server à la place : 

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
|[Azure Database Migration Service (DMS)](../../../dms/tutorial-sql-server-to-managed-instance.md)  | Service Azure interne qui prend en charge la migration en mode hors connexion pour les applications qui peuvent se permettre un temps d’arrêt pendant le processus de migration. Contrairement à la migration continue en mode en ligne, la migration en mode hors connexion exécute une restauration unique d’une sauvegarde complète de la base de données de la source vers la cible. | 
|[Sauvegarde et restauration natives](../../managed-instance/restore-sample-database-quickstart.md) | SQL Managed Instance prend en charge la restauration des sauvegardes de base de données SQL Server natives (fichiers .bak), ce qui en fait l’option de migration la plus simple pour les clients qui peuvent fournir des sauvegardes complètes de base de données sur le stockage Azure. Les sauvegardes complètes et différentielles sont également prises en charge et documentées dans la [section sur les ressources de migration](#migration-assets) plus loin dans cet article.| 
| | |

### <a name="alternative-tools"></a>Autres outils

Le tableau suivant liste d’autres outils de migration : 

|Technology |Description  |
|---------|---------|
|[Réplication transactionnelle](../../managed-instance/replication-transactional-overview.md) | Réplique les données des tables de la base de données SQL Server source dans Azure SQL Managed Instance en proposant une option de migration de type éditeur-abonné tout en préservant la cohérence transactionnelle. |  |
|[Copie en bloc](/sql/relational-databases/import-export/import-and-export-bulk-data-by-using-the-bcp-utility-sql-server)| L’[utilitaire bcp (bulk copy program)](/sql/tools/bcp-utility) permet de copier les données d’une instance SQL Server dans un fichier de données. Utilisez l’utilitaire BCP pour exporter les données de votre source et importer le fichier de données dans l’instance Azure SQL Managed Instance cible.</br></br> Pour profiter d’opérations de copie en bloc à haut débit lors du déplacement de données vers Azure SQL Managed Instance, l’[outil Smart Bulk Copy](/samples/azure-samples/smartbulkcopy/smart-bulk-copy/) peut être utilisé pour optimiser la vitesse de transfert en tirant parti des tâches de copie en parallèle. | 
|[Assistant Importation et exportation/BACPAC](../../database/database-import.md?tabs=azure-powershell)| [BACPAC](/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac) est un fichier Windows avec une extension `.bacpac` qui encapsule le schéma et les données d’une base de données. BACPAC permet d’exporter des données à partir d’une source SQL Server et d’importer le fichier dans Azure SQL Managed Instance.  |  
|[Azure Data Factory (ADF)](../../../data-factory/connector-azure-sql-managed-instance.md)| L’[activité de copie](../../../data-factory/copy-activity-overview.md) d’Azure Data Factory migre les données des bases de données SQL Server sources vers Azure SQL Managed Instance à l’aide de connecteurs intégrés et d’un [runtime d’intégration](../../../data-factory/concepts-integration-runtime.md).</br> </br> ADF prend en charge un large éventail de [connecteurs](../../../data-factory/connector-overview.md) pour déplacer les données de sources SQL Server vers Azure SQL Managed Instance. |
| | |

## <a name="compare-migration-options"></a>Comparer les options de migration

Comparez les options de migration de façon à emprunter la voie qui vous permettra de répondre à vos besoins métier. 

### <a name="recommended-options"></a>Options recommandées

Le tableau suivant compare les options de migration recommandées : 

|Option de migration  |Quand l’utiliser  |Considérations  |
|---------|---------|---------|
|[Azure Database Migration Service (DMS)](../../../dms/tutorial-sql-server-to-managed-instance.md) | – Migration de bases de données uniques ou de plusieurs bases de données à grande échelle. </br> – Acceptation de temps d’arrêt pendant le processus de migration. </br> </br> Sources prises en charge : </br> - Serveur SQL (2005 à 2019) en local ou machine virtuelle Azure </br> - AWS EC2 </br> - AWS RDS </br> - Machine virtuelle SQL Server GCP Compute |  – Les migrations à grande échelle peuvent être automatisées via [PowerShell](../../../dms/howto-sql-server-to-azure-sql-mi-powershell.md). </br> – La durée de la migration dépend de la taille de la base de données et est influencée par le temps de sauvegarde et de restauration. </br> – Un temps d’arrêt suffisant peut être nécessaire. |
|[Sauvegarde et restauration natives](../../managed-instance/restore-sample-database-quickstart.md) | – Migration de bases de données d’applications métier individuelles.  </br> – Migration rapide et facile sans un service ou un outil de migration distinct.  </br> </br> Sources prises en charge : </br> - Serveur SQL (2005 à 2019) en local ou machine virtuelle Azure </br> - AWS EC2 </br> - AWS RDS </br> – Machine virtuelle SQL Server GCP Compute | – La sauvegarde de base de données utilise plusieurs threads pour optimiser le transfert de données vers Stockage Blob Azure, mais la bande passante de l’ISV et la taille de la base de données peuvent avoir un impact sur le débit de transfert. </br> – Les temps d’arrêt doivent tenir compte du temps nécessaire pour effectuer une sauvegarde et une restauration complètes (ce qui correspond à une taille d’une opération de données).| 
| | | |

### <a name="alternative-options"></a>Autres options

Le tableau suivant compare les autres options de migration : 

|Méthode/technologique |Quand l’utiliser |Considérations  |
|---------|---------|---------|
|[Réplication transactionnelle](../../managed-instance/replication-transactional-overview.md) | – Migration avec une publication continue des modifications des tables de base de données sources vers les tables de base de données Azure SQL Managed Instance cibles. </br> – Migration complète ou partielle des tables sélectionnées (sous-ensemble d’une base de données).  </br> </br> Sources prises en charge : </br> – SQL Server (2012 à 2019) avec certaines limitations </br> – AWS EC2  </br> - Machine virtuelle SQL Server GCP Compute | </br> - La configuration est relativement complexe par rapport à d’autres options de migration.   </br> - Offre une option de réplication continue pour migrer les données (sans mettre les bases de données en mode hors connexion).</br> - La réplication transactionnelle présente plusieurs limitations dont il faut tenir compte au moment de configurer le serveur de publication sur l’instance SQL Server source. Pour plus d’informations, consultez [Limitations de la publication d’objets](/sql/relational-databases/replication/publish/publish-data-and-database-objects#limitations-on-publishing-objects).  </br> – Il est possible de [surveiller l’activité de réplication](/sql/relational-databases/replication/monitor/monitoring-replication).    |
|[Copie en bloc](/sql/relational-databases/import-export/import-and-export-bulk-data-by-using-the-bcp-utility-sql-server)| – Migration complète ou partielle de données. </br> – Acceptation de temps d’arrêt. </br> </br> Sources prises en charge : </br> - Serveur SQL (2005 à 2019) en local ou machine virtuelle Azure </br> - AWS EC2 </br> - AWS RDS </br> - Machine virtuelle SQL Server GCP Compute   | - Impose un temps d’arrêt pour exporter les données de la source et les importer dans la cible. </br> – Les formats de fichiers et les types de données utilisés dans l’exportation/importation doivent être en cohérence avec les schémas de table. |
|[Assistant Importation et exportation/BACPAC](../../database/database-import.md)| – Migration de bases de données d’applications métier individuelles. </br>– Adapté aux bases de données de plus petite taille.  </br>  – Ne nécessite pas de service ou d’outil de migration distinct. </br> </br> Sources prises en charge : </br> - Serveur SQL (2005 à 2019) en local ou machine virtuelle Azure </br> - AWS EC2 </br> - AWS RDS </br> - Machine virtuelle SQL Server GCP Compute  |   </br> - Impose un temps d’arrêt dans la mesure où les données doivent être exportées à la source et importées à destination.   </br> - Les formats de fichiers et les types de données utilisés dans l’exportation/importation doivent être en cohérence avec les schémas de table pour éviter des erreurs de troncation et/ou de non-correspondance de types de données. </br> – L’exportation d’une base de données comportant un grand nombre d’objets peut prendre beaucoup plus de temps. |
|[Azure Data Factory (ADF)](../../../data-factory/connector-azure-sql-managed-instance.md)| – Migration et/ou transformation des données à partir des bases de données SQL Server sources.</br> – Fusion de données issues de plusieurs sources de données vers Azure SQL Managed Instance, généralement pour des charges de travail de type décisionnel.   </br> – Impose la création de pipelines de déplacement des données dans Azure Data Factory pour déplacer les données de la source vers la destination.   </br> - Le [coût](https://azure.microsoft.com/pricing/details/data-factory/data-pipeline/) est une considération importante qui varie en fonction des déclencheurs de pipeline, des exécutions d’activité, de la durée de déplacement des données, etc. |
| | | |

## <a name="feature-interoperability"></a>Interopérabilité des fonctionnalités 

D’autres éléments sont à prendre en considération quand il s’agit de migrer des charges de travail qui reposent sur d’autres fonctionnalités SQL Server. 

#### <a name="sql-server-integration-services"></a>SQL Server Integration Services

Migrez des packages et des projets SQL Server Integration Services (SSIS) dans SSISDB vers Azure SQL Managed Instance à l’aide d’[Azure Database Migration Service (DMS)](../../../dms/how-to-migrate-ssis-packages-managed-instance.md). 

Seuls les packages SSIS dans SSISDB à partir de SQL Server 2012 sont pris en charge pour la migration. Convertissez les packages SSIS hérités avant la migration. Pour en savoir plus, consultez le [tutoriel de conversion de projet](/sql/integration-services/lesson-6-2-converting-the-project-to-the-project-deployment-model). 


#### <a name="sql-server-reporting-services"></a>SQL Server Reporting Services

Les rapports SQL Server Reporting Services (SSRS) peuvent être migrés vers des rapports paginés dans Power BI. Utilisez l’ [outil de migration RDL](https://github.com/microsoft/RdlMigration) pour préparer et migrer vos rapports. Cet outil a été développé par Microsoft pour aider les utilisateurs à effectuer la migration des rapports RDL entre leurs serveurs SSRS et Power BI. Il est disponible sur GitHub avec une documentation qui décrit de bout en bout chaque étape du scénario de migration. 

#### <a name="sql-server-analysis-services"></a>SQL Server Analysis Services

Les modèles tabulaires SQL Server Analysis Services de SQL Server 2012 et versions ultérieures peuvent être migrés vers Azure Analysis Services, qui est un modèle de déploiement PaaS pour le modèle tabulaire Analysis Services dans Azure. Vous pouvez en savoir plus sur la migration de modèles locaux vers Azure Analysis Services dans ce [tutoriel vidéo](https://azure.microsoft.com/resources/videos/azure-analysis-services-moving-models/).

Vous pouvez également envisager de migrer vos modèles tabulaires Analysis Services locaux vers [Power BI Premium à l’aide des nouveaux points de terminaison en lecture/écriture XMLA](/power-bi/admin/service-premium-connect-tools). 
> [!NOTE]
> La fonctionnalité de points de terminaison en lecture/écriture XMLA de Power BI est actuellement en Préversion publique et ne doit pas être prise en compte pour les charges de travail de production tant que la fonctionnalité n’est pas disponible de manière générale.

#### <a name="high-availability"></a>Haute disponibilité

Les fonctionnalités de haute disponibilité SQL Server, les instances de cluster de basculement Always On et les groupes de disponibilité Always On deviennent obsolètes sur l’instance Azure SQL Managed Instance cible, car l’architecture de haute disponibilité est déjà intégrée dans les niveaux de service [Usage général (modèle de disponibilité Standard)](../../database/high-availability-sla.md#basic-standard-and-general-purpose-service-tier-locally-redundant-availability) et [Critique pour l’entreprise (modèle de disponibilité Premium)](../../database/high-availability-sla.md#premium-and-business-critical-service-tier-locally-redundant-availability) de SQL Managed Instance. Le modèle de disponibilité Premium propose également une échelle horizontale en lecture qui permet de se connecter à l’un des nœuds secondaires à des fins de lecture seule.     

Au-delà de l’architecture de haute disponibilité incluse dans Azure SQL Managed Instance, il y a aussi la fonctionnalité des [groupes de basculement automatique ](../../database/auto-failover-group-overview.md) qui vous permet de gérer la réplication et le basculement des bases de données d’une instance gérée vers une autre région. 

#### <a name="sql-agent-jobs"></a>Travaux SQL Agent

Utilisez l’option Azure Database Migration Service (DMS) en mode hors connexion pour migrer des [travaux SQL Agent](../../../dms/howto-sql-server-to-azure-sql-mi-powershell.md#offline-migrations). Dans le cas contraire, créez un script des travaux en Transact-SQL (T-SQL) à l’aide de SQL Server Management Studio, puis recréez-les manuellement sur l’instance Azure SQL Managed Instance cible. 

> [!IMPORTANT]
> À l’heure actuelle, Azure DMS prend uniquement en charge les travaux comportant des étapes du sous-système T-SQL. Les travaux comportant des étapes du package SSIS devront être migrés manuellement. 

#### <a name="logins-and-groups"></a>Connexions et groupes

Les connexions SQL à partir du serveur SQL source peuvent être déplacées vers Azure SQL Managed Instance à l’aide de Database Migration Service (DMS) en mode hors connexion.  Utilisez le panneau **[Connexions sélectionnées](../../../dms/tutorial-sql-server-to-managed-instance.md#select-logins)** dans l’**Assistant Migration** pour migrer les connexions vers votre instance Azure SQL Managed Instance cible. 

Par défaut, Azure Database Migration Service prend uniquement en charge la migration des connexions SQL. Toutefois, vous pouvez rendre possible la migration des connexions Windows en procédant comme suit :

Assurez-vous que l’instance Azure SQL Managed Instance cible dispose de l’accès en lecture Azure AD, qui peut être configuré dans le portail Azure par un utilisateur doté du rôle **Administrateur d’entreprise** ou d’un rôle **Administrateur général**.
Configurez votre instance Azure Database Migration Service pour activer la migration des connexions d’utilisateur ou de groupe Windows, qui est configurée via le Portail Azure, dans la page Configuration. Après avoir activé ce paramètre, redémarrez le service pour que les modifications prennent effet.

Après le redémarrage du service, les connexions d’utilisateur ou de groupe Windows apparaissent dans la liste des connexions disponibles pour la migration. Pour toutes les connexions d’utilisateur ou de groupe Windows que vous migrez, vous êtes invité à fournir le nom de domaine associé. Les comptes d’utilisateur de service (comptes avec le nom de domaine NT AUTHORITY) et les comptes d’utilisateur virtuels (nom de compte avec le nom de domaine NT SERVICE) ne sont pas pris en charge.

Pour en savoir plus, consultez [comment migrer des utilisateurs et des groupes Windows dans une instance SQL vers Azure SQL Managed Instance en utilisant le langage T-SQL](../../managed-instance/migrate-sql-server-users-to-instance-transact-sql-tsql-tutorial.md).

Une autre solution consiste à utiliser l’[outil utilitaire PowerShell](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/MoveLogins) spécialement conçu par les architectes de migration de données Microsoft. Cet utilitaire s’appuie sur PowerShell pour créer un script T-SQL en vue de recréer les connexions et de sélectionner les utilisateurs de base de données de la source vers la cible. L’outil mappe automatiquement les comptes Windows AD aux comptes Azure AD et peut effectuer une recherche de nom d’utilisateur principal (UPN) pour chaque connexion auprès de l’instance Active Directory source. L’outil scripte les rôles de serveur et de base de données personnalisés ainsi que l’appartenance à un rôle, le rôle de base de données et les autorisations utilisateur. Les bases de données autonomes ne sont actuellement pas prises en charge, et seul un sous-ensemble des autorisations SQL Server possibles est scripté. 

#### <a name="encryption"></a>Chiffrement

Lors de la migration de bases de données protégées par  [Transparent Data Encryption](../../database/transparent-data-encryption-tde-overview.md) vers une instance gérée à l’aide de l’option de restauration native, [migrez le certificat correspondant](../../managed-instance/tde-certificate-migrate.md) du serveur SQL source vers l’instance SQL Managed Instance cible *avant* la restauration de la base de données. 

#### <a name="system-databases"></a>Bases de données système

La restauration de bases de données système n’est pas prise en charge. Pour migrer des objets au niveau de l’instance (stockés dans des bases de données MASTER ou msdb), scriptez-les en utilisant le langage Transact-SQL (T-SQL), puis recréez-les sur l’instance gérée cible. 

## <a name="leverage-advanced-features"></a>Tirer parti des fonctionnalités avancées 

Veillez à tirer parti des fonctionnalités informatiques avancées offertes par Azure SQL Managed Instance. Par exemple, vous n’avez plus à vous soucier de la gestion des sauvegardes, car le service le fait à votre place. Vous pouvez effectuer une restauration à n’importe quel [point dans le temps inclus dans la période de rétention](../../database/recovery-using-backups.md#point-in-time-restore). De plus, vous n’avez pas à vous soucier de la configuration de la  [haute disponibilité](../../database/high-availability-sla.md), car cette dernière est intégrée. 

Pour renforcer la sécurité, envisagez d’utiliser l’ [authentification Azure Active Directory](../../database/authentication-aad-overview.md), l’[audit](../../managed-instance/auditing-configure.md), la  [détection des menaces](../../database/azure-defender-for-sql.md), la  [sécurité au niveau des lignes](/sql/relational-databases/security/row-level-security) et le  [masquage dynamique des données](/sql/relational-databases/security/dynamic-data-masking).

En plus des fonctionnalités avancées en matière de sécurité et de gestion, SQL Managed Instance fournit un ensemble d’outils perfectionnés pouvant vous aider dans la [supervision et l’optimisation de votre charge de travail](../../database/monitor-tune-overview.md). [Azure SQL Analytics](../../../azure-monitor/insights/azure-sql.md) vous permet de surveiller un large ensemble d’instances gérées de manière centralisée. Le  [réglage automatique](/sql/relational-databases/automatic-tuning/automatic-tuning#automatic-plan-correction) des instances gérées surveille en permanence les performances de vos statistiques d’exécution du plan SQL et corrige automatiquement les problèmes de performances identifiés. 

Certaines fonctionnalités sont disponibles uniquement une fois que vous avez fait passer le [niveau de compatibilité de la base de données](/sql/relational-databases/databases/view-or-change-the-compatibility-level-of-a-database) au dernier niveau (150). 

## <a name="migration-assets"></a>Ressources de migration 

Pour obtenir une aide supplémentaire, consultez les ressources suivantes qui ont été développées pour des projets de migration concrets.

|Asset  |Description  |
|---------|---------|
|[Outil et modèle d’évaluation d’une charge de travail de données](https://github.com/Microsoft/DataMigrationTeam/tree/master/Data%20Workload%20Assessment%20Model%20and%20Tool)| Cet outil fournit les plateformes cibles, la préparation du cloud et le niveau de correction des applications/bases de données « les mieux adaptés » pour une charge de travail donnée. Il propose une génération de rapports et des calculs simples en un clic qui permettent d’accélérer les évaluations d’un vaste domaine en fournissant un processus de décision de plateforme cible automatisé et uniforme.|
|[Utilitaire DBLoader](https://github.com/microsoft/DataMigrationTeam/tree/master/DBLoader%20Utility)|DBLoader permet de charger des données dans SQL Server à partir de fichiers texte délimités. Cet utilitaire de console Windows utilise l’interface de chargement en bloc du client natif SQL Server, qui fonctionne sur toutes les versions de SQL Server, dont Azure SQL MI.|
|[Utilitaire permettant de déplacer des connexions SQL Server locales vers Azure SQL Managed Instance](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/MoveLogins)|Script PowerShell qui crée un script de commande T-SQL visant à recréer les connexions et à sélectionner les utilisateurs de base de données d’un serveur SQL local vers Azure SQL Managed Instance. Cet outil permet de mapper automatiquement des comptes Windows AD à des comptes Azure AD et éventuellement de migrer des connexions natives SQL Server.|
|[Automatisation de la collecte des données perfmon à l’aide de Logman](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/Perfmon%20Data%20Collection%20Automation%20Using%20Logman)|Outil qui collecte les données perfmon pour comprendre le niveau de performance de la ligne de base et qui contribue à la recommandation de la cible de migration. Cet outil se sert de logman.exe pour créer la commande qui va créer, démarrer, arrêter et supprimer les compteurs de performances définis sur un serveur SQL distant.|
|[Livre blanc : Migration de base de données vers Azure SQL Managed Instance en restaurant des sauvegardes complètes et différentielles](https://github.com/microsoft/DataMigrationTeam/blob/master/Whitepapers/Database%20migrations%20to%20Azure%20SQL%20DB%20Managed%20Instance%20-%20%20Restore%20with%20Full%20and%20Differential%20backups.pdf)|Ce livre blanc fournit des instructions et des étapes pour accélérer les migrations de SQL Server vers Azure SQL Managed Instance si vous avez uniquement des sauvegardes différentielles et complètes (et aucune capacité de sauvegarde de fichier journal).|

Ces ressources ont été développées dans le cadre du programme Data SQL Ninja, qui est sponsorisé par l’équipe d’ingénierie Groupe de données Azure. La charte fondamentale du programme Data SQL Ninja a pour objet d’initier et d’accélérer une modernisation complexe et de faire face aux opportunités de migration de plateforme de données vers la plateforme de données Azure de Microsoft. Si vous pensez que votre organisation aimerait participer au programme Data SQL Ninja, contactez votre équipe en charge des compte et demandez-lui de soumettre une candidature.


## <a name="next-steps"></a>Étapes suivantes

Pour commencer votre migration de SQL Server vers Azure SQL Managed Instance, consultez le [guide de migration de SQL Server vers Azure SQL Managed Instance](sql-server-to-managed-instance-guide.md).

- Pour obtenir une matrice des services et outils Microsoft et tiers qui peuvent vous aider dans les différents scénarios de migration de données et de base de données ainsi que leurs tâches spécialisées, consultez [Services et outils de migration de données](../../../dms/dms-tools-matrix.md).

- Pour en savoir plus sur Azure SQL Managed Instance, consultez :
   - [Niveaux de service dans Azure SQL Managed Instance](../../managed-instance/sql-managed-instance-paas-overview.md#service-tiers)
   - [Différences entre SQL Server et Azure SQL Managed Instance](../../managed-instance/transact-sql-tsql-differences-sql-server.md)
   - [Calculatrice du coût total de possession Azure](https://azure.microsoft.com/pricing/tco/calculator/) 


- Pour plus d’informations sur l’infrastructure et le cycle d’adoption pour les migrations cloud, consultez :
   -  [Cloud Adoption Framework pour Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Meilleures pratiques pour l’évaluation des coûts et le dimensionnement des charges de travail migrées vers Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 


- Pour évaluer la couche d’accès aux applications, consultez [Data Access Migration Toolkit (préversion)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit).
- Pour savoir comment exécuter un test A/B sur la couche d’accès aux données, consultez [Assistant Expérimentation de base de données](/sql/dea/database-experimentation-assistant-overview).