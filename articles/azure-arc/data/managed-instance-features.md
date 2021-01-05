---
title: Fonctionnalités et capacités de SQL Managed Instance activé pour Azure Arc
description: Fonctionnalités et capacités de SQL Managed Instance activé pour Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 9c42acb69e13cc1eb0fbba3fcafaec1451bc4d77
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97589217"
---
# <a name="features-and-capabilities-of-azure-arc-enabled-sql-managed-instance"></a>Fonctionnalités et capacités de SQL Managed Instance activé pour Azure Arc

Les instances Azure SQL Managed Instance pour Azure Arc partagent une base de code commune avec la dernière version stable de SQL Server. La plupart des fonctionnalités standard du langage SQL, du traitement des requêtes et de la gestion des bases de données sont identiques. Les fonctionnalités communes à SQL Server et SQL Database ou SQL Managed Instance sont les suivantes :

- Fonctionnalités de langage : [Mots clés du langage de contrôle de flux ](/sql/t-sql/language-elements/control-of-flow), [Curseurs](/sql/t-sql/language-elements/cursors-transact-sql), [Types de données](/sql/t-sql/data-types/data-types-transact-sql), [Instructions DML](/sql/t-sql/queries/queries), [Prédicats](/sql/t-sql/queries/predicates), [Numéros séquentiels](/sql/relational-databases/sequence-numbers/sequence-numbers), [Procédures stockées](/sql/relational-databases/stored-procedures/stored-procedures-database-engine) et [Variables](/sql/t-sql/language-elements/variables-transact-sql).
- Fonctionnalités de base de données : [Réglage automatique (plan forcé)](/sql/relational-databases/automatic-tuning/automatic-tuning), [Suivi des modifications](/sql/relational-databases/track-changes/about-change-tracking-sql-server), [Classement de base de données](/sql/relational-databases/collations/set-or-change-the-database-collation), [Bases de données autonomes](/sql/relational-databases/databases/contained-databases), [Utilisateurs de base de données autonome](/sql/relational-databases/security/contained-database-users-making-your-database-portable), [Compression de données](/sql/relational-databases/data-compression/data-compression), [Paramètres de configuration de base de données](/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql), [Opérations en ligne sur les index](/sql/relational-databases/indexes/perform-index-operations-online), [Partitionnement](/sql/relational-databases/partitions/partitioned-tables-and-indexes) et [Tables temporelles](/sql/relational-databases/tables/temporal-tables) ([voir le guide de démarrage](/sql/relational-databases/tables/getting-started-with-system-versioned-temporal-tables)).
- Fonctionnalités de sécurité : [Rôles d’application](/sql/relational-databases/security/authentication-access/application-roles), [Masquage dynamique des données](/sql/relational-databases/security/dynamic-data-masking) ([Prise en main du masquage des données dynamiques de base de données SQL dans le Portail Azure](../../azure-sql/database/dynamic-data-masking-configure-portal.md)), [Sécurité au niveau des lignes](/sql/relational-databases/security/row-level-security)
- Fonctionnalités multi-modèles : [Traitement des graphiques](/sql/relational-databases/graphs/sql-graph-overview), [Données JSON](/sql/relational-databases/json/json-data-sql-server), [OPENXML](/sql/t-sql/functions/openxml-transact-sql)), [Données spatiales](/sql/relational-databases/spatial/spatial-data-sql-server), [OPENJSON](/sql/t-sql/functions/openjson-transact-sql) et [Index XML](/sql/t-sql/statements/create-xml-index-transact-sql).



[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="features-of-azure-arc-enabled-sql-managed-instance"></a>Fonctionnalités de SQL Managed Instance avec Azure Arc

###  <a name="rdbms-high-availability"></a><a name="RDBMSHA"></a> RDBMS High Availability  
  
|Fonctionnalité|SQL Managed Instance avec Azure Arc|
|-------------|----------------|
|Copie des journaux de transaction|Oui| 
|Compression de sauvegarde|Oui|
|Instantané de base de données|Oui|
|Instance de cluster de basculement Always On<sup>1</sup>| Non applicable. Fonctionnalités similaires disponibles |
|Groupes de disponibilité Always On<sup>2</sup>|Les fonctionnalités de haute disponibilité sont planifiées.|
|Groupes de disponibilité de base <sup>2</sup>|Les fonctionnalités de haute disponibilité sont planifiées.|
|Groupe de disponibilité à validation de réplica minimale <sup>2</sup>|Les fonctionnalités de haute disponibilité sont planifiées.|
|Groupe de disponibilité sans cluster|Oui|
|Restauration en ligne de pages et de fichiers|Oui|
|Indexation en ligne|Oui|
|Reconstructions d’index en ligne pouvant être reprises|Oui|
|Modification de schéma en ligne|Oui|
|Récupération rapide|Oui|
|Sauvegardes en miroir|Oui|
|Ajout de mémoire et de processeur à chaud|Oui|
|Sauvegarde chiffrée|Oui|
|Sauvegarde hybride vers Azure (sauvegarde vers une URL)|Oui|

<sup>1</sup> dans le scénario avec un échec de pod, une nouvelle instance Azure SQL Managed Instance démarre et s’attache à nouveau au volume persistant contenant vos données. [En savoir plus sur les volumes persistants Kubernetes ici](https://kubernetes.io/docs/concepts/storage/persistent-volumes).

<sup>2</sup> Les versions à venir fourniront des fonctionnalités AG 

###  <a name="rdbms-scalability-and-performance"></a><a name="RDBMSSP"></a> RDBMS Scalability and Performance  

| Fonctionnalité | SQL Managed Instance avec Azure Arc |
|--|--|
| columnstore | Oui |
| Fichiers binaires LOB dans les index columnstore cluster | Oui |
| Reconstruction d’index columnstore non cluster en ligne | Oui |
| OLTP en mémoire | Oui |
| Mémoire principale persistante | Oui |
| Partitionnement des tables et des index | Oui |
| Compression des données | Oui |
| gouverneur de ressources | Oui |
| Parallélisme de tables partitionnées | Oui |
| Mémoire de pages de grande taille compatible NUMA et allocation de tableau de tampons | Oui |
| Gouvernance des ressources d'E/S | Oui |
| Durabilité différée | Oui |
| Paramétrage automatique | Oui |
| Jointures adaptatives en mode batch | Oui |
| Retour d’allocation de mémoire en mode batch | Oui |
| Exécution entrelacée pour les fonctions table à instructions multiples | Oui |
| Améliorations de l’insertion en bloc | Oui |

###  <a name="rdbms-security"></a><a name="RDBMSS"></a> RDBMS Security

| Fonctionnalité | SQL Managed Instance avec Azure Arc |
|--|--|
| Sécurité au niveau des lignes | Oui |
| Always Encrypted | Oui |
| Always Encrypted avec enclaves sécurisées | Non |
| Masquage dynamique des données | Oui |
| Audit de base | Oui |
| Audit de granularité fine | Oui |
| Chiffrement transparent de base de données | Oui |
| Rôles définis par l’utilisateur | Oui |
| Bases de données autonomes | Oui |
| Chiffrement des sauvegardes | Oui |

###  <a name="rdbms-manageability"></a><a name="RDBMSM"></a> RDBMS Manageability  

| Fonctionnalité | SQL Managed Instance avec Azure Arc |
|--|--|
| Connexion administrateur dédiée | Oui |
| Prise en charge de scripts PowerShell | Oui |
| Prise en charge des opérations des composants d’application du niveau Données : extraction, déploiement, mise à niveau, suppression | Oui |
| Automation de stratégie (vérification selon la planification et sur modification) | Oui |
| Collecteur de données de performances | Oui |
| Rapports de performances standard | Oui |
| Repères de plan et gel de plan relatif | Oui |
| Requête directe de vues d'index (à l'aide de l'indicateur NOEXPAND) | Oui |
| Maintenance automatique des vues indexées | Oui |
| Vues partitionnées distribuées | Oui |
| Opérations d'index parallèles | Oui |
| Utilisation automatique de vues indexées par l'optimiseur de requête | Oui |
| Vérifications de cohérence parallèles | Oui |

### <a name="programmability"></a><a name="Programmability"></a> Programmability  

| Fonctionnalité | SQL Managed Instance avec Azure Arc |
|--|--|
| JSON | Oui |
| Magasin des requêtes | Oui |  |
| Temporal | Oui |  |
| Prise en charge XML native | Oui |  |
| Indexation XML | Oui |  |
| Fonctions MERGE & UPSERT | Oui |  |
| Types de données de date et d'heure | Oui |  |
| Support d'internationalisation | Oui |  |
| Recherche sémantique et en texte intégral | Non |
| Spécification d'une langue dans une requête | Oui |  |
| Service Broker (messagerie) | Oui |  |
| Transact-SQL, points de terminaison | Oui |  |
| Graph | Oui |  |
| Machine Learning Services | Non |  |
| PolyBase | Non |


### <a name="tools"></a>Outils

Les instances Azure SQL Managed Instance pour Azure Arc prennent en charge différents outils de données qui peuvent vous aider à gérer vos données.

| **Outil** | SQL Managed Instance avec Azure Arc|
| --- | --- | --- |
| Portail Azure <sup>1</sup> | Non |
| Azure CLI | Non |
| [Azure Data Studio](/sql/azure-data-studio/what-is) | Oui |
| Azure PowerShell | Oui |
| [Fichier BACPAC (exporter)](/sql/relational-databases/data-tier-applications/export-a-data-tier-application) | Oui |
| [Fichier BACPAC (importer)](/sql/relational-databases/data-tier-applications/import-a-bacpac-file-to-create-a-new-user-database) | Oui |
| [Outils SQL Server Data Tools (SSDT)](/sql/ssdt/download-sql-server-data-tools-ssdt) | Oui |
| [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) | Oui |
| [SQL Server PowerShell](/sql/relational-databases/scripting/sql-server-powershell) | Oui |
| [SQL Server Profiler](/sql/tools/sql-server-profiler/sql-server-profiler) | Oui |

<sup>1</sup> Le Portail Azure est utilisé uniquement pour afficher les instances Azure SQL Managed Instance pour Azure Arc en lecture seule pendant la version préliminaire.


### <a name="unsupported-features--services"></a><a name="Unsupported"></a> Fonctionnalités et services non pris en charge

Les fonctionnalités et services suivants ne sont pas disponibles pour SQL Managed Instance compatible avec Azure Arc. Le support de ces fonctionnalités sera de plus en plus activé dans le temps.

| Domaine | Fonctionnalité ou service non pris en charge |
|-----|-----|
| **Moteur de base de données** | Réplication de fusion |
| &nbsp; | Base de données pour Stretch |
| &nbsp; | Requête distribuée avec connexions tierces |
| &nbsp; | Serveurs liés à des sources de données autres que SQL Server et les produits SQL Azure |
| &nbsp; | Procédures stockées étendues système (XP_CMDSHELL, etc.) |
| &nbsp; | FileTable, FILESTREAM |
| &nbsp; | Assemblys CLR avec l’ensemble d’autorisations EXTERNAL_ACCESS ou UNSAFE |
| &nbsp; | Buffer Pool Extension |
| **SQL Server Agent** |  Sous-systèmes : CmdExec, PowerShell, lecture de la file d’attente, SSIS, SSAS, SSRS |
| &nbsp; | Alertes |
| &nbsp; | Sauvegarde managée |
| **Haute disponibilité** | Mise en miroir de bases de données  |
| **Sécurité** | Gestion de clés extensible |
| &nbsp; | Authentification AD pour les serveurs liés | 
| &nbsp; | Authentification AD pour les groupes de disponibilité | 