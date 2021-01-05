---
title: Qu'est-ce qu'Azure SQL Managed Instance ?
description: Azure SQL Managed Instance est presque 100 % compatible avec le dernier moteur de base de données SQL Server (Édition Entreprise)
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: overview
author: bonova
ms.author: bonova
ms.reviewer: sstein, vanto
ms.date: 08/14/2020
ms.openlocfilehash: 83f38797e406ff7e62503f59ef979b9ce4f07f97
ms.sourcegitcommit: f6236e0fa28343cf0e478ab630d43e3fd78b9596
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94917934"
---
# <a name="what-is-azure-sql-managed-instance"></a>Qu'est-ce qu'Azure SQL Managed Instance ?
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Azure SQL Managed Instance est le service de base de données cloud intelligent et évolutif qui combine la plus grande compatibilité de moteur de base de données SQL Server avec tous les avantages d’une plateforme en tant que service entièrement gérée et persistante. SQL Managed Instance est presque 100 % compatible avec le moteur de base de données SQL Server (Édition Entreprise) le plus récent et fournit une implémentation de [réseau virtuel (VNet)](../../virtual-network/virtual-networks-overview.md) native traitant les problèmes de sécurité courants ainsi qu’un [modèle d’entreprise](https://azure.microsoft.com/pricing/details/sql-database/) favorable aux clients SQL Server existants. SQL Managed Instance permet aux clients SQL Server existants d’effectuer une migration « lift-and-shift » de leurs applications locales vers le cloud en apportant des modifications minimales aux applications et bases de données. En même temps, SQL Managed Instance conserve toutes les fonctionnalités PaaS (correctifs et mises à jour de versions automatiques, [sauvegardes automatisées](../database/automated-backups-overview.md), [haute disponibilité](../database/high-availability-sla.md)), ce qui réduit considérablement le temps de gestion et le coût total de possession.

Si vous débutez avec Azure SQL Managed Instance, regardez la vidéo *Azure SQL Managed Instance*, qui fait partie de notre [série de vidéos Azure SQL](https://channel9.msdn.com/Series/Azure-SQL-for-Beginners?WT.mc_id=azuresql4beg_azuresql-ch9-niner) approfondies :
> [!VIDEO https://channel9.msdn.com/Series/Azure-SQL-for-Beginners/Azure-SQL-Managed-Instance-Overview-6-of-61/player]

> [!IMPORTANT]
> Pour obtenir la liste des régions où SQL Managed Instance est actuellement disponible, consultez [régions prises en charge](resource-limits.md#supported-regions).

Le diagramme suivant présente les principales fonctionnalités de SQL Managed Instance :

![Fonctionnalités clés](./media/sql-managed-instance-paas-overview/key-features.png)

Azure SQL Managed Instance est conçu pour les clients cherchant à effectuer une migration d’un grand nombre d’applications locales ou provenant d’environnements IaaS, générés automatiquement ou fournis par un éditeur de logiciels indépendant, afin de gérer complètement un environnement cloud PaaS avec le moins d’efforts de migration possible. Avec le service [Azure Data Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md#create-an-azure-database-migration-service-instance) entièrement automatisé, les clients peuvent effectuer une migration « lift-and-shift » de leurs instances SQL Server existantes vers une instance SQL Managed Instance compatible avec SQL Server et qui offre une isolation totale des instances des clients avec une prise en charge native des réseaux virtuels. Pour plus d’informations sur les options et les outils de migration, consultez [Vue d’ensemble de la migration : SQL Server vers Azure SQL Managed Instance](../migration-guides/managed-instance/sql-server-to-managed-instance-overview.md).</br> Avec Software Assurance, vous pouvez échanger vos licences existantes pour bénéficier de tarifs réduits sur SQL Managed Instance à l’aide d’[Azure Hybrid Benefit pour SQL Server](https://azure.microsoft.com/pricing/hybrid-benefit/). SQL Managed Instance est la meilleure destination de migration dans le cloud pour les instances SQL Server qui nécessitent une haute sécurité et une surface de programmabilité riche.

## <a name="key-features-and-capabilities"></a>Fonctionnalités principales

SQL Managed Instance combine les meilleures fonctionnalités d’Azure SQL Database et du moteur de base de données SQL Server.

> [!IMPORTANT]
> SQL Managed Instance s’exécute avec toutes les fonctionnalités de la version la plus récente de SQL Server, notamment les opérations en ligne, les corrections de plan automatiques et d’autres améliorations des performances d’entreprise. Une comparaison des fonctionnalités disponibles est expliquée dans [Comparaison des fonctionnalités : Azure SQL Managed Instance et SQL Server](../database/features-comparison.md).

| **Avantages PaaS** | **Continuité de l’activité** |
| --- | --- |
|Aucun achat ni gestion de matériel <br>Aucun temps de gestion à dédier à l’infrastructure sous-jacente <br>Provisionnement et mise à l’échelle du service rapides <br>Application automatisée de correctifs et de mises à niveau de version <br>Intégration à d’autres services de données PaaS |Contrat SLA à 99,99 % de durée de fonctionnement  <br>Une [haute disponibilité](../database/high-availability-sla.md) intégrée <br>Données protégées par des [sauvegardes automatisées](../database/automated-backups-overview.md) <br>Période de rétention de sauvegarde configurable par le client <br>[Sauvegardes](/sql/t-sql/statements/backup-transact-sql?preserve-view=true&view=azuresqldb-mi-current) lancées par l’utilisateur <br>Fonctionnalité de [limite de restauration dans le temps d’une base de données](../database/recovery-using-backups.md#point-in-time-restore) |
|**Sécurité et conformité** | **Gestion**|
|Environnement isolé ([intégration de réseau virtuel](connectivity-architecture-overview.md), service de locataire unique, calcul et stockage dédiés) <br>[Chiffrement transparent des données (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)<br>[Authentification Azure Active Directory (Azure AD)](../database/authentication-aad-overview.md), prise en charge de l’authentification unique <br> <a href="/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current">Principaux de serveur (connexions) Azure AD</a>  <br>Conformité aux mêmes normes qu’une base de données Azure SQL <br>[Audit SQL](auditing-configure.md) <br>[Protection avancée contre les menaces](threat-detection-configure.md) |API Azure Resource Manager pour automatiser le provisionnement et la mise à l’échelle des services <br>Fonctionnalités du portail Azure pour le provisionnement et la mise à l’échelle manuels des services <br>Service de migration des données

> [!IMPORTANT]
> Azure SQL Managed Instance a été certifié par rapport à plusieurs normes de conformité. Pour plus d’informations, consultez le document [Microsoft Azure Compliance Offerings](https://servicetrust.microsoft.com/ViewPage/MSComplianceGuideV3?command=Download&downloadType=Document&downloadId=44bbae63-bf4d-4e3b-9d3d-c96fb25ec363&tab=7027ead0-3d6b-11e9-b9e1-290b1eb4cdeb&docTab=7027ead0-3d6b-11e9-b9e1-290b1eb4cdeb_FAQ_and_White_Papers), qui fournit la liste la plus récente de certifications de conformité SQL Managed Instance, sous la rubrique **SQL Database**.

Les fonctionnalités clés de SQL Managed Instance figurent dans le tableau suivant :

|Fonctionnalité | Description|
|---|---|
| Version/Build de SQL Server | Moteur de base de données SQL Server (dernière version stable) |
| Sauvegardes automatisées gérées | Oui |
| Analyse et métriques des instances et bases de données intégrées | Oui |
| Mise à jour corrective automatique des logiciels | Oui |
| Les dernières fonctionnalités du moteur de base de données | Oui |
| Nombre de fichiers de données (ROWS) par base de données | Multiple |
| Nombre de fichiers journaux (LOG) par base de données | 1 |
| Réseau virtuel - Déploiement Azure Resource Manager | Oui |
| Réseau virtuel - Modèle de déploiement classique | Non |
| Prise en charge du portail | Oui|
| Integration Services (SSIS) intégré | Non : SSIS fait partie de la [plateforme PaaS Azure Data Factory](../../data-factory/tutorial-deploy-ssis-packages-azure.md) |
| Analysis Services (SSAS) intégré | Non : SSAS est une [plateforme PaaS](../../analysis-services/analysis-services-overview.md) distincte |
| Reporting Services (SSRS) intégré | Non : utilisez les [Rapports paginés Power BI](/power-bi/paginated-reports/paginated-reports-report-builder-power-bi) à la place ou hébergez SSRS sur une machine virtuelle Azure. Même si SQL Managed Instance ne peut pas exécuter SSRS en tant que service, il peut héberger des [bases de données de catalogue SSRS](/sql/reporting-services/install-windows/ssrs-report-server-create-a-report-server-database#database-server-version-requirements) pour un serveur de rapports installé sur une machine virtuelle Azure à l’aide de l’authentification SQL Server. |
|||

## <a name="vcore-based-purchasing-model"></a>Modèle d’achat vCore

Le [modèle d’achat vCore](../database/service-tiers-vcore.md) pour SQL Managed Instance vous assure flexibilité, contrôle et transparence. Il permet de traduire de manière simple les exigences des charges de travail locales dans le cloud. Ce modèle vous permet de changer la capacité de calcul, la mémoire et le stockage en fonction des besoins de vos charges de travail. Le modèle vCore permet également de réaliser jusqu'à 55 % d'économies avec [Azure Hybrid Benefit pour SQL Server](https://azure.microsoft.com/pricing/hybrid-benefit/).

Dans le modèle vCore, vous pouvez choisir entre différentes générations de matériel.

- Les processeurs logiques **Gen4** sont basés sur des processeurs Intel® E5-2673 v3 (Haswell) de 2,4 GHz, un disque SSD attaché, des cœurs physiques, 7 Go de RAM par cœur et des tailles de calcul comprises entre 8 et 24 vCores.
- Les processeurs logiques **Gen5** sont basés sur des processeurs Intel® E5-2673 v4 (Broadwell) de 2,3 GHz, Intel® SP-8160 (Skylake) et Intel® 8272CL (Cascade Lake) de 2,5 GHz, un disque SSD NVMe rapide, un cœur logique multithread, et des tailles de calcul comprises entre 4 et 80 cœurs.

Vous trouverez des informations sur les différences entre les générations de matériel dans les [limites de ressources de SQL Managed Instance](resource-limits.md#hardware-generation-characteristics).

## <a name="service-tiers"></a>Niveaux de service

SQL Managed Instance est disponible en deux niveaux de service :

- **Usage général** : conçu pour des applications avec des exigences de performances et de latence d’E/S standard.
- **Critique pour l’entreprise** : conçu pour les applications avec des exigences de latence d’E/S faible et un impact minimal des opérations de maintenance sous-jacentes sur la charge de travail.

Les deux niveaux de service garantissent une disponibilité de 99,99 % et vous permettent de sélectionner la taille de stockage et la capacité de calcul indépendamment. Pour plus d’informations sur l’architecture à haute disponibilité d’Azure SQL Managed Instance, consultez [Haute disponibilité et Azure SQL Managed Instance](../database/high-availability-sla.md).

### <a name="general-purpose-service-tier"></a>Niveau de service Usage général

La liste suivante décrit les principales caractéristiques du niveau de service Usage général :

- Concevoir pour la majorité des applications métier avec des exigences de performances standard
- Stockage Blob Azure à hautes performances (8 To)
- [Haute disponibilité](../database/high-availability-sla.md#basic-standard-and-general-purpose-service-tier-locally-redundant-availability) intégrée basée sur un Stockage Blob Azure fiable et [Azure Service Fabric](../../service-fabric/service-fabric-overview.md)

Pour plus d’informations, consultez les articles [storage layer in general purpose tier](https://medium.com/azure-sqldb-managed-instance/file-layout-in-general-purpose-azure-sql-managed-instance-cf21fff9c76c) et [storage performance best practices and considerations for SQL Managed Instance (general purpose)](/archive/blogs/sqlcat/storage-performance-best-practices-and-considerations-for-azure-sql-db-managed-instance-general-purpose).

Vous trouverez plus d’informations sur les différences entre les niveaux de service dans les [limites de ressources de SQL Managed Instance](resource-limits.md#service-tier-characteristics).

### <a name="business-critical-service-tier"></a>Niveau de service Critique pour l’entreprise

Le niveau de service Critique pour l’entreprise est conçu pour les applications dont les exigences en termes d’E/S sont élevées. Il offre la meilleure résilience aux défaillances grâce à l’utilisation de plusieurs réplicas isolés.

La liste suivante décrit les principales caractéristiques du niveau de service Critique pour l’entreprise :

- Conçu pour les applications d’entreprise avec les exigences les plus hautes en matière de performances et de disponibilité
- Fourni avec un stockage SSD local extrêmement rapide (jusqu’à 1 To sur Gen4, jusqu’à 4 To sur Gen5)
- [Haute disponibilité](../database/high-availability-sla.md#premium-and-business-critical-service-tier-locally-redundant-availability) intégrée basée sur les [groupes de disponibilité AlwaysOn](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server) et [Azure Service Fabric](../../service-fabric/service-fabric-overview.md)
- Un [réplica de base de données en lecture seule](../database/read-scale-out.md) intégré supplémentaire qui peut être utilisé pour les rapports et d’autres charges de travail en lecture seule
- [OLTP en mémoire](../in-memory-oltp-overview.md), qui peut être utilisé pour les charges de travail avec des exigences de hautes performances  

Vous trouverez plus d’informations sur les différences entre les niveaux de service dans les [limites de ressources de SQL Managed Instance](resource-limits.md#service-tier-characteristics).

## <a name="management-operations"></a>Opérations de gestion

Azure SQL Managed Instance fournit des opérations de gestion que vous pouvez utiliser pour déployer automatiquement de nouvelles instances managées, mettre à jour les propriétés des instances et supprimer des instances quand vous n’en avez plus besoin. Vous trouverez une explication détaillée des opérations de gestion sur la page [Opérations de gestion des instances managées](management-operations-overview.md).

## <a name="advanced-security-and-compliance"></a>Sécurité et conformité avancées

SQL Managed Instance intègre les fonctionnalités de sécurité avancées fournies par la plateforme Azure et le moteur de base de données SQL Server.

### <a name="security-isolation"></a>Isolation à des fins de sécurité

SQL Managed Instance offre une meilleure isolation de la sécurité vis-à-vis des autres locataires dans la plateforme Azure. L’isolation de la sécurité inclut :

- [Implémentation d’un réseau virtuel natif](connectivity-architecture-overview.md) et sa connexion à votre environnement local à l’aide d’Azure ExpressRoute ou d’une passerelle VPN.
- Dans un déploiement par défaut, le point de terminaison SQL est exposé uniquement par le biais d’une adresse IP privée, ce qui permet de sécuriser la connexion à partir de réseaux Azure privés ou hybrides.
- Locataire unique avec infrastructure sous-jacente dédiée (calcul, stockage).

Le diagramme suivant présente différentes options de connectivité pour vos applications :

![Haute disponibilité](./media/sql-managed-instance-paas-overview/application-deployment-topologies.png)  

Pour plus d’informations sur l’intégration de réseau virtuel et l’application de stratégie de mise en réseau au niveau du sous-réseau, consultez [Architecture de réseau virtuel pour les instances managées](connectivity-architecture-overview.md) et [Connecter votre application à une instance managée](connect-application-instance.md).

> [!IMPORTANT]
> Placez plusieurs instances gérées dans le même sous-réseau, partout où vos exigences de sécurité l’autorisent, car cela vous apportera des avantages supplémentaires. La colocation d’instances dans le même sous-réseau simplifie considérablement la maintenance de l’infrastructure réseau et réduit le temps de provisionnement des instances, car une durée de provisionnement longue est associée au coût de déploiement de la première instance managée dans un sous-réseau.

### <a name="security-features"></a>Fonctionnalités de sécurité

Azure SQL Managed Instance fournit un ensemble de fonctionnalités de sécurité avancées qui peuvent être utilisées pour protéger vos données.

- [L’audit SQL Managed Instance](auditing-configure.md) suit les événements de base de données et les écrit dans un fichier journal d’audit placé dans votre compte de stockage Azure. L’audit peut vous aider à respecter une conformité réglementaire, à comprendre l’activité de la base de données et à découvrir des discordances et anomalies susceptibles d’indiquer des problèmes pour l’entreprise ou des violations de la sécurité.
- Chiffrement des données en mouvement : SQL Managed Instance sécurise vos données par le biais d’un chiffrement des données en mouvement à l’aide du protocole TLS. En plus du protocole TLS, SQL Managed Instance offre une protection des données sensibles en vol, au repos et pendant le traitement des requêtes avec [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine). Always offre la protection des données contre les failles de sécurité impliquant le vol de données critiques. Par exemple, avec Always Encrypted, les numéros de carte de crédit sont toujours chiffrés dans la base de données, même pendant le traitement des requêtes, ce qui permet le déchiffrement au point d’utilisation par les applications ou le personnel autorisés qui doivent traiter ces données.
- [Advanced Threat Protection](threat-detection-configure.md) complète l’[audit](auditing-configure.md) en fournissant une couche supplémentaire d’informations de sécurité intégrée au service qui détecte les tentatives d’accès ou d’exploitation de base de données inhabituelles et potentiellement dangereuses. Vous êtes alerté en cas d’activités suspectes, de vulnérabilités potentielles, d’attaques par injection de code SQL et de modèles d’accès anormaux à la base de données. Les alertes de la protection avancée contre les menaces sont consultables dans [Azure Security Center](https://azure.microsoft.com/services/security-center/). Elles fournissent des détails sur les activités suspectes et recommandent l’action à entreprendre pour analyser et atténuer la menace.  
- Le [masquage dynamique des données](/sql/relational-databases/security/dynamic-data-masking) limite l’exposition des données sensibles en les masquant aux utilisateurs sans privilèges. Le masquage des données dynamique contribue à empêcher tout accès non autorisé aux données sensibles en vous permettant d’indiquer la quantité de données sensibles à révéler avec un impact minimal sur la couche Application. Il s’agit d’une fonctionnalité de sécurité basée sur des stratégies qui masque les données sensibles dans le jeu de résultats d’une requête, sur des champs de base de données désignés (les données dans la base de données ne sont pas modifiées).
- La [sécurité au niveau des lignes](/sql/relational-databases/security/row-level-security) (RLS) vous permet de contrôler l’accès aux lignes d’une table de base de données en fonction des caractéristiques de l’utilisateur qui exécute une requête (par exemple, appartenance à un groupe ou contexte d’exécution). La sécurité au niveau des colonnes simplifie la conception et le codage de la sécurité dans votre application. Elle vous permet d’implémenter des restrictions sur l’accès aux lignes de données. Par exemple, en s’assurant que les employés ne peuvent accéder qu’aux lignes de données utiles à leur service, ou en limitant l’accès aux données aux seules données pertinentes.
- [Transparent Data Encryption (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) : également appelé chiffrement des données au repos, chiffre les fichiers de données SQL Managed Instance. TDE effectue le chiffrement et le déchiffrement d’E/S en temps réel des données et des fichiers journaux. Le chiffrement utilise une clé de chiffrement de base de données stockée dans l’enregistrement de démarrage de base de données à des fins de disponibilité lors de la récupération. Vous pouvez protéger toutes vos bases de données dans une instance managée avec Transparent Data Encryption. Il s’agit de la technologie de chiffrement au repos éprouvée de SQL Server, qui est requise par de nombreuses normes de conformité comme protection contre le vol d’un support de stockage.

La migration d’une base de données chiffrée vers SQL Managed Instance est prise en charge via Azure Database Migration Service ou une restauration native. Si vous envisagez de migrer une base de données chiffrée en utilisant une restauration native, la migration du certificat TDE existant de l’instance SQL Server vers SQL Managed Instance est une étape obligatoire. Pour plus d’informations sur les options de migration, consultez [Migration de SQL Server vers SQL Managed Instance](migrate-to-instance-from-sql-server.md).

## <a name="azure-active-directory-integration"></a>Intégration d’Azure Active Directory

SQL Managed Instance prend en charge les connexions à des moteurs de base de données SQL Server traditionnelles et les connexions intégrées à Azure AD. Les principaux (connexions) de serveur Azure AD (**préversion publique**) correspondent à une version cloud Azure des connexions aux bases de données locales que vous utilisez dans votre environnement local. Les principaux (connexions) de serveur Azure AD vous permettent de spécifier des utilisateurs et des groupes de votre locataire Azure AD sous forme de principaux limités à une instance et capables d’effectuer toutes les opérations au niveau de l’instance, y compris les requêtes entre plusieurs bases de données au sein de la même instance managée.

Une nouvelle syntaxe a été ajoutée pour créer des principaux (connexions) de serveur Azure AD (**À PARTIR D’UN FOURNISSEUR EXTERNE**). Pour plus d’informations sur la syntaxe, consultez <a href="/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current">CREATE LOGIN</a> et lisez l’article [Provisionner un administrateur Azure Active Directory pour SQL Managed Instance](../database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).

### <a name="azure-active-directory-integration-and-multi-factor-authentication"></a>Intégration d’Azure Active Directory et authentification multifacteur

SQL Managed Instance vous permet de gérer de manière centralisée les identités d’utilisateur de base de données et d’autres services Microsoft avec l’[Intégration d’Azure Active Directory](../database/authentication-aad-overview.md). Cette fonctionnalité simplifie la gestion des autorisations et améliore la sécurité. Azure Active Directory prend en charge l’[authentification multifacteur](../database/authentication-mfa-ssms-configure.md) pour augmenter la sécurité des données et des applications, ainsi qu’un processus d’authentification unique.

### <a name="authentication"></a>Authentification

L’authentification SQL Managed Instance fait référence à la façon dont les utilisateurs prouvent leur identité quand ils se connectent à la base de données. SQL Managed Instance prend en charge deux types d’authentification :  

- **Authentification SQL** :

  Cette méthode d’authentification utilise un nom d’utilisateur et un mot de passe.
- **Authentification Azure Active Directory** :

  Cette méthode d’authentification utilise des identités gérées par Azure Active Directory, et est prise en charge pour les domaines managés et intégrés. Utilisez l’authentification Active Directory (sécurité intégrée) [dans la mesure du possible](/sql/relational-databases/security/choose-an-authentication-mode).

### <a name="authorization"></a>Autorisation

Le terme autorisation fait référence aux actions qu’un utilisateur peut exécuter au sein d’une base de données dans Azure SQL Managed Instance. Celles-ci sont contrôlées par les appartenances aux rôles et les autorisations au niveau objet de la base de données de votre compte d’utilisateur. SQL Managed Instance a les mêmes fonctionnalités d’autorisation que SQL Server 2017.

## <a name="database-migration"></a>Migration de base de données

SQL Managed Instance cible des scénarios d’utilisateur impliquant une migration de base de données en masse depuis des implémentations locales ou IaaS. SQL Managed Instance prend en charge plusieurs options de migration de base de données qui sont décrites dans les guides de migration. Consultez [Vue d’ensemble de la migration : SQL Server vers Azure SQL Managed Instance](../migration-guides/managed-instance/sql-server-to-managed-instance-overview.md) pour plus d’informations.

### <a name="backup-and-restore"></a>Sauvegarde et restauration  

L’approche de la migration s’appuie sur les sauvegardes SQL dans Stockage Blob Azure. Les sauvegardes stockées dans un objet blob de stockage Azure peuvent être directement restaurées dans une instance managée à l’aide de la [commande T-SQL RESTORE](/sql/t-sql/statements/restore-statements-transact-sql?preserve-view=true&view=azuresqldb-mi-current).

- Pour obtenir un guide de démarrage rapide montrant comment restaurer le fichier de sauvegarde de base de données Wide World Importers - Standard, consultez [Restaurer un fichier de sauvegarde dans une instance managée](restore-sample-database-quickstart.md). Ce guide de démarrage rapide vous montre que vous devez charger un fichier de sauvegarde dans le stockage d’objets blob Azure et le sécuriser à l’aide d’une clé de signature d’accès partagé (SAS).
- Pour plus d’informations sur la restauration à partir d’une URL, consultez [Restauration native à partir d’une URL](../migration-guides/managed-instance/sql-server-to-managed-instance-guide.md#backup-and-restore).

> [!IMPORTANT]
> Les sauvegardes d’une instance managée peuvent uniquement être restaurées sur une autre instance managée. Elles ne peuvent pas être restaurées sur une instance SQL Server ou une base de données Azure SQL.

### <a name="database-migration-service"></a>Database Migration Service

Azure Database Migration Service est un service complètement managé conçu pour permettre la migration transparente de plusieurs sources de base de données vers des plateformes de données Azure, pour un temps d’arrêt minime. Ce service simplifie les tâches nécessaires pour déplacer les bases de données externes et les bases de données SQL Server vers Azure SQL Database, Azure SQL Managed Instance et SQL Server dans la machine virtuelle Azure. Consultez [Comment migrer votre base de données locale vers SQL Managed Instance à l’aide de Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).

## <a name="sql-features-supported"></a>Fonctionnalités SQL prises en charge

SQL Managed Instance a pour but d’offrir une compatibilité de la surface d’exposition proche de 100 % avec la dernière version de SQL Server par le biais d’un plan de mise en production intermédiaire. Pour obtenir la liste des fonctionnalités et des différences, consultez [Comparaison des fonctionnalités SQL Managed Instance](../database/features-comparison.md). Pour obtenir la liste des différences de T-SQL entre SQL Managed Instance et SQL Server, consultez [Différences de T-SQL entre SQL Managed Instance et SQL Server](transact-sql-tsql-differences-sql-server.md).

SQL Managed Instance prend en charge la compatibilité descendante avec les bases de données SQL Server 2008. La migration directe à partir de serveurs de base de données SQL Server 2005 est prise en charge, le niveau de compatibilité des bases de données SQL Server 2005 qui ont migré est mis à jour vers SQL Server 2008.
  
Le diagramme suivant présente la compatibilité de la surface d’exposition dans SQL Managed Instance :  

![compatibilité de la surface d’exposition](./media/sql-managed-instance-paas-overview/migration.png)

### <a name="key-differences-between-sql-server-on-premises-and-sql-managed-instance"></a>Principales différences entre SQL Server local et SQL Managed Instance

SQL Managed Instance tire parti du fait d’être toujours à jour dans le cloud, ce qui signifie que certaines fonctionnalités de SQL Server peuvent quant à elles être obsolètes, supprimées ou remplacées par d’autres. Dans certains cas, les outils doivent reconnaître qu'une fonctionnalité particulière agit de façon légèrement différente ou que le service est exécuté dans un environnement que vous ne contrôlez pas entièrement.

Quelques différences clés :

- La haute disponibilité est intégrée et préconfigurée avec une technologie similaire aux [groupes de disponibilité AlwaysOn](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server).
- Le client ne dispose que de sauvegardes automatisées et de la restauration dans le temps limitée. Les clients peuvent lancer des sauvegardes `copy-only` qui n’interfèrent pas avec la chaîne de sauvegarde automatique.
- La spécification de chemins physiques complets n’étant pas prise en charge, tous les scénarios correspondants doivent être pris en charge d’une autre façon : RESTORE DB ne prend pas en charge WITH MOVE, CREATE DB n’autorise pas les chemins d’accès physiques, BULK INSERT fonctionne uniquement avec les objets BLOB Azure, etc.
- SQL Managed Instance prend en charge l’[authentification Azure AD](../database/authentication-aad-overview.md) comme une alternative cloud à l’authentification Windows.
- SQL Managed Instance gère automatiquement le groupe de fichiers et les fichiers XTP des bases de données contenant des objets OLTP en mémoire.
- SQL Managed Instance prend en charge SSIS (SQL Server Integration Services) et peut héberger le catalogue SSIS (SSISDB) qui stocke les packages SSIS, mais ceux-ci sont exécutés sur une instance Azure-SSIS IR (Integration Runtime) managée dans Azure Data Factory. Consultez [Créer un runtime d’intégration Azure-SSIS dans Data Factory](../../data-factory/create-azure-ssis-integration-runtime.md). Pour comparer les fonctionnalités SSIS, consultez [Comparer SQL Database et SQL Managed Instance](../../data-factory/create-azure-ssis-integration-runtime.md#comparison-of-sql-database-and-sql-managed-instance).

### <a name="administration-features"></a>Fonctionnalités d’administration

SQL Managed Instance permet à l’administrateur système de passer moins de temps sur les tâches d’administration, car le service les effectue pour vous ou les simplifie considérablement. Par exemple, l’[installation et la mise à jour corrective d’un système d’exploitation ou système de gestion de base de données relationnelle](../database/high-availability-sla.md), le [redimensionnement et la configuration d’une instance dynamique](../database/single-database-scale.md), les [sauvegardes](../database/automated-backups-overview.md), la [réplication de base de données](replication-between-two-instances-configure-tutorial.md) (notamment des bases de données système), la [configuration de la haute disponibilité](../database/high-availability-sla.md) et la configuration des flux de données de [supervision des performances](../../azure-monitor/insights/azure-sql.md) et de l’intégrité.

Pour plus d’informations, consultez [une liste des fonctionnalités SQL Managed Instance prises en charge et non prises en charge](../database/features-comparison.md) et les [différences T-SQL entre SQL Managed Instance et SQL Server](transact-sql-tsql-differences-sql-server.md).

### <a name="programmatically-identify-a-managed-instance"></a>Identifier par programmation une instance managée

Le tableau suivant montre plusieurs propriétés, accessibles par le biais de Transact-SQL, que vous pouvez utiliser pour détecter que votre application fonctionne avec SQL Managed Instance et récupérer des propriétés importantes.

|Propriété|Valeur|Commentaire|
|---|---|---|
|`@@VERSION`|Microsoft SQL Azure (RTM) - 12.0.2000.8 2018-03-07 Copyright (C) 2018 Microsoft Corporation.|Cette valeur est identique à celle indiquée dans SQL Database. **N'indique pas** le moteur SQL version 12 (SQL Server 2014). SQL Managed Instance exécute toujours la dernière version stable du moteur SQL, qui est égale ou supérieure à la dernière version RTM disponible de SQL Server.  |
|`SERVERPROPERTY ('Edition')`|SQL Azure|Cette valeur est identique à celle indiquée dans SQL Database.|
|`SERVERPROPERTY('EngineEdition')`|8|Cette valeur identifie de façon unique une instance managée.|
|`@@SERVERNAME`, `SERVERPROPERTY ('ServerName')`|Nom DNS d’instance complet au format suivant :`<instanceName>`.`<dnsPrefix>`.database.windows.net, où `<instanceName>` est le nom fourni par le client, tandis que `<dnsPrefix>` est une partie générée automatiquement du nom garantissant l’unicité des noms DNS globaux (par exemple, « wcus17662feb9ce98 »)|Exemple : my-managed-instance.wcus17662feb9ce98.database.windows.net|

## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment créer votre première instance managée, consultez le [Guide de démarrage rapide](instance-create-quickstart.md).
- Pour consulter la liste des fonctionnalités et les comparer, consultez [Fonctionnalités SQL communes](../database/features-comparison.md).
- Pour plus d’informations sur la configuration du réseau virtuel, consultez [Configuration de réseau virtuel SQL Managed Instance](connectivity-architecture-overview.md).
- Pour obtenir un guide de démarrage rapide qui crée une instance managée et restaure une base de données à partir d’un fichier de sauvegarde, consultez [Créer une instance managée](instance-create-quickstart.md).
- Pour accéder à un tutoriel expliquant comment utiliser Azure Database Migration Service pour la migration, consultez [Migration SQL Managed Instance à l’aide d’Azure Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).
- Pour une supervision avancée des performances de base de données SQL Managed Instance avec des informations de dépannage intégrées, consultez [Superviser Azure SQL Managed Instance avec Azure SQL Analytics](../../azure-monitor/insights/azure-sql.md).
- Pour plus d’informations sur la tarification, voir [Tarification SQL Database](https://azure.microsoft.com/pricing/details/sql-database/managed/).