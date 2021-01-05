---
title: Présentation de SQL Data Sync pour Azure
description: Cette vue d’ensemble présente SQL Data Sync pour Azure, qui vous permet de synchroniser des données entre plusieurs bases de données cloud et locales.
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: data sync, sqldbrb=1, fasttrack-edit
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 08/20/2019
ms.openlocfilehash: 55fa106f0515405dcad969f05d28e0bc7b975b40
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96922278"
---
# <a name="what-is-sql-data-sync-for-azure"></a>Présentation de SQL Data Sync pour Azure

SQL Data Sync est un service basé sur Azure SQL Database qui vous permet de synchroniser les données choisies de manière bidirectionnelle sur plusieurs bases de données, à la fois locales et dans le cloud. 

> [!IMPORTANT]
> Azure SQL Data Sync ne prend pas en charge Azure SQL Managed Instance pour l’instant.


## <a name="overview"></a>Vue d’ensemble 

SQL Data Sync est basé sur le concept d’un groupe de synchronisation. Un groupe de synchronisation est un groupe de bases de données que vous souhaitez synchroniser.

Data Sync utilise une topologie hub and spoke pour synchroniser les données. Vous définissez l’une des bases de données du groupe de synchronisation en tant que base de données Hub. Le reste des bases de données sont des bases de données membres. La synchronisation se produit uniquement entre le hub et des membres individuels.

- Le **base de données Hub** doit être une base de données Azure SQL Database.
- Les **bases de données membres** peuvent être des bases de données dans Azure SQL Database ou dans des instances de SQL Server.
- La **base de données de métadonnées de synchronisation** contient les métadonnées et le journal de Data Sync. La base de données de métadonnées de synchronisation doit être une base de données Azure SQL Database située dans la même région que la base de données Hub. La base de données de métadonnées de synchronisation est créée par le client et lui appartient. Vous ne pouvez avoir qu’une seule base de données de métadonnées de synchronisation par région et par abonnement. La base de données de métadonnées de synchronisation ne peut pas être supprimée ou de renommée tant que des groupes de synchronisation ou des agents de synchronisation existent. Microsoft recommande de créer une nouvelle base de données vide, à utiliser comme base de données de métadonnées de synchronisation. SQL Data Sync crée les tables dans cette base de données et exécute une charge de travail fréquente.

> [!NOTE]
> Si vous utilisez une base de données locale comme base de données membre, vous devez [installer et configurer un agent de synchronisation local](sql-data-sync-sql-server-configure.md#add-on-prem).

![Synchroniser des données entre des bases de données](./media/sql-data-sync-data-sql-server-sql-database/sync-data-overview.png)

Un groupe de synchronisation dispose des propriétés suivantes :

- Le **schéma de synchronisation** décrit quelles sont les données en cours de synchronisation.
- Le **sens de synchronisation** peut être bidirectionnel ou peut circuler dans une seule direction. Autrement dit, le sens de synchronisation peut être *Hub vers membre*, *Membre vers hub*, ou les deux.
- L’**intervalle de synchronisation** correspond à la fréquence à laquelle la synchronisation se produit.
- La **stratégie de résolution de conflit** est une stratégie au niveau groupe, qui peut être *Priorité au hub* ou *Priorité au membre*.

## <a name="when-to-use"></a>Quand l’utiliser

Data Sync est utile dans les cas où les données doivent être tenues à jour entre plusieurs bases de données dans Azure SQL Database ou SQL Server. Voici les principaux cas d’usage pour Data Sync :

- **Synchronisation de données hybride :** Avec Data Sync, vous pouvez assurer la synchronisation de données entre vos bases de données dans SQL Server et Azure SQL Database pour activer des applications hybrides. Cette fonctionnalité peut intéresser les clients qui envisagent de passer au cloud et souhaiteraient placer une partie de leur application dans Azure.
- **Applications distribuées :** dans de nombreux cas, il est recommandé de séparer les différentes charges de travail entre plusieurs bases de données. Par exemple, si vous possédez une base de données de production de grande taille, mais que vous devez également exécuter un rapport ou une charge de travail analytique de ces données, il est utile de disposer d’une seconde base de données pour cette charge de travail supplémentaire. Cette approche réduit l’impact sur les performances de votre charge de travail de production. Vous pouvez utiliser Data Sync afin de maintenir la synchronisation de ces deux bases de données.
- **Applications distribuées globalement :** de nombreuses entreprises sont présentes dans plusieurs régions et même dans plusieurs pays/régions. Afin de réduire la latence du réseau, il est préférable de conserver vos données dans une région proche. Avec Data Sync, vous pouvez facilement synchroniser des bases de données dans différentes régions partout dans le monde.

Data Sync n’est pas la solution préconisée pour les scénarios suivants :

| Scénario | Certaines solutions recommandées |
|----------|----------------------------|
| Récupération d’urgence | [Sauvegardes géoredondantes Azure](automated-backups-overview.md) |
| Mise à l’échelle en lecture | [Utiliser des réplicas en lecture seule pour équilibrer des charges de travail de requêtes en lecture seule (préversion)](read-scale-out.md) |
| ETL (OLTP vers OLAP) | [Azure Data Factory](https://azure.microsoft.com/services/data-factory/) ou [SQL Server Integration Services](/sql/integration-services/sql-server-integration-services) |
| Migration de SQL Server vers Azure SQL Database. Toutefois, SQL Data Sync peut être utilisé une fois la migration terminée afin de garantir la synchronisation de la source et de la cible.  | [Azure Database Migration Service](https://azure.microsoft.com/services/database-migration/) |
|||

## <a name="how-it-works"></a>Fonctionnement

- **Suivi des modifications de données :** Data Sync effectue le suivi des modifications en utilisant des déclencheurs d’insertion, de mise à jour et de suppression. Les modifications sont enregistrées dans une table latérale dans la base de données utilisateur. BULK INSERT n’active aucun déclencheur par défaut. Si FIRE_TRIGGERS n’est pas spécifié, aucun déclencheur d’insertion ne s’exécute. Ajoutez l’option FIRE_TRIGGERS afin que Data Sync puisse suivre ces insertions. 
- **Synchronisation des données :** SQL Data Sync est conçu dans un modèle de Hub and Spoke. Le hub se synchronise avec chaque membre individuellement. Les modifications depuis le hub sont téléchargées vers le membre, puis les modifications à partir du membre sont chargées vers le hub.
- **Résolution des conflits :** Data Sync fournit deux options pour la résolution de conflit, *Priorité au hub* ou *Priorité au membre*.
  - Si vous sélectionnez *Priorité au hub*, les modifications dans le hub remplacent toujours les modifications dans le membre.
  - Si vous sélectionnez *Priorité au membre*, les modifications dans le membre remplacent toujours les modifications dans le hub. S’il existe plusieurs membres, la valeur finale dépend du membre qui se synchronise en premier.

## <a name="compare-with-transactional-replication"></a>Comparaison avec la réplication transactionnelle

| | Synchronisation des données | Réplication transactionnelle |
|---|---|---|
| **Avantages** | - Support actif/actif<br/>- Synchronisation bidirectionnelle entre la base de données Azure SQL et locale | - Latence réduite<br/>- Cohérence transactionnelle<br/>- Réutilisation de la topologie existante après la migration <br/>\- Prise en charge d’Azure SQL Managed Instance |
| **Inconvénients** | - Pas de cohérence transactionnelle<br/>- Impact plus important sur les performances | - Impossible de publier à partir d’Azure SQL Database <br/>- Coût de maintenance élevé |

## <a name="private-link-for-data-sync-preview"></a>Liaison privée pour Azure SQL Data Sync (préversion)
La nouvelle fonctionnalité de liaison privée (préversion) vous permet de choisir un point de terminaison privé géré par le service pour établir une connexion sécurisée entre le service de synchronisation et vos bases de données membres/Hub pendant le processus de synchronisation des données. Un point de terminaison privé géré par le service est une adresse IP privée au sein d’un réseau virtuel et d’un sous-réseau spécifiques. Dans Azure SQL Data Sync, le point de terminaison privé géré par le service est créé par Microsoft et est utilisé exclusivement par le service Synchronisation des données pour une opération de synchronisation donnée. Avant de configurer la liaison privée, lisez les [conditions générales](sql-data-sync-data-sql-server-sql-database.md#general-requirements) de la fonctionnalité. 

![Liaison privée pour Azure SQL Data Sync](./media/sql-data-sync-data-sql-server-sql-database/sync-private-link-overview.png)

> [!NOTE]
> Vous devez approuver manuellement le point de terminaison privé géré par le service dans la page **Connexions de point de terminaison privé** du portail Azure pendant le déploiement du groupe de synchronisation ou à l’aide de PowerShell.

## <a name="get-started"></a>Bien démarrer 

### <a name="set-up-data-sync-in-the-azure-portal"></a>Configurer Data Sync dans le portail Azure

- [Configurer Azure SQL Data Sync](sql-data-sync-sql-server-configure.md)
- Agent de synchronisation des données - [Agent de synchronisation des données pour Azure SQL Data Sync](sql-data-sync-agent-overview.md)

### <a name="set-up-data-sync-with-powershell"></a>Configurer Data Sync avec PowerShell

- [Utiliser PowerShell pour synchroniser des données entre plusieurs bases de données dans Azure SQL Database](scripts/sql-data-sync-sync-data-between-sql-databases.md)
- [Utiliser PowerShell pour synchroniser des données entre une base de données d’Azure SQL Database et une base de données d’une instance SQL Server](scripts/sql-data-sync-sync-data-between-azure-onprem.md)

### <a name="review-the-best-practices-for-data-sync"></a>Consultez les bonnes pratiques pour Data Sync

- [Bonnes pratiques pour Azure SQL Data Sync](sql-data-sync-best-practices.md)

### <a name="did-something-go-wrong"></a>Un problème est survenu ?

- [Résoudre les problèmes liés à Azure SQL Data Sync](./sql-data-sync-troubleshoot.md)

## <a name="consistency-and-performance"></a>Cohérence et performances

### <a name="eventual-consistency"></a>Cohérence éventuelle

Étant donné que Data Sync est basé sur le déclencheur, la cohérence transactionnelle n’est pas garantie. Microsoft garantit que toutes les modifications sont effectuées par la suite et que Data Sync n’entraîne pas de perte de données.

### <a name="performance-impact"></a>Impact sur les performances

Data Sync utilise des déclencheurs d’insertion, de mise à jour et de suppression pour effectuer le suivi des modifications. Cela crée des tables latérales dans la base de données utilisateur pour le suivi des modifications. Ces activités de suivi des modifications ont un impact sur votre charge de travail de base de données. Évaluez votre niveau de service et effectuez une mise à niveau si nécessaire.

Le provisionnement et le déprovisionnement lors de la création, la mise à jour et la suppression du groupe de synchronisation peuvent également avoir un impact sur les performances de la base de données.

## <a name="requirements-and-limitations"></a><a name="sync-req-lim"></a> Spécifications et limitations

### <a name="general-requirements"></a>Conditions générales

- Chaque table doit avoir une clé primaire. Ne modifiez pas la valeur de la clé primaire dans une ligne. Si vous avez à le faire, supprimez la ligne et recréez-la avec la nouvelle valeur de clé primaire.

> [!IMPORTANT]
> Le changement de la valeur d’une clé primaire existante entraîne le comportement incorrect suivant :
> - Les données entre le hub et le membre risquent d’être perdues même si la synchronisation ne signale aucun problème.
> - La synchronisation peut échouer, car la table de suivi contient une ligne qui n’existe pas dans la source en raison du changement de la clé primaire.

- Le niveau d’isolement d’instantané doit être activé pour le hub et les membres de synchronisation. Pour plus d’informations, consultez [Isolement de capture instantanée dans SQL Server](/dotnet/framework/data/adonet/sql/snapshot-isolation-in-sql-server).

- Pour utiliser la liaison privée avec Azure SQL Data Sync, les bases de données Hub et celles de membre doivent être hébergées dans Azure (même région ou régions différentes), dans le même type de cloud (par exemple, les deux dans le cloud public ou dans le cloud gouvernemental). En outre, pour utiliser une liaison privée, les fournisseurs de ressources Microsoft.Network doivent être inscrits pour les abonnements qui hébergent les serveurs hub et membres. Enfin, vous devez approuver manuellement la liaison privée pour Azure SQL Data Sync pendant la configuration de la synchronisation, dans la section « Connexions des point de terminaison privés » du portail Azure ou via PowerShell. Pour plus d’informations sur l’approbation de la liaison privée, consultez [Configurer SQL Data Sync](./sql-data-sync-sql-server-configure.md). Une fois que vous avez approuvé le point de terminaison privé géré par le service, toutes les communications entre le service de synchronisation et les bases de données membres/Hub se font par le biais de la liaison privée. Les groupes de synchronisation existants peuvent être mis à jour pour que cette fonctionnalité soit activée.

### <a name="general-limitations"></a>Limitations générales

- Une table ne peut pas avoir une colonne d’identité qui n’est pas la clé primaire.
- Une table doit avoir un index cluster pour utiliser la synchronisation des données.
- Une clé primaire ne peut pas avoir les types de données suivants : sql_variant, binary, varbinary, image et xml.
- Si vous utilisez les types de données suivants comme clé primaire, n’oubliez pas que la précision n’est prise en charge qu’à la seconde près : time, datetime, datetime2 et datetimeoffset.
- Les noms des objets (bases de données, tables et colonnes) ne peuvent pas contenir les caractères imprimables suivants : point (.), crochet gauche ou crochet droit (]).
- Un nom de table ne peut pas contenir les caractères imprimables : ! " # $ % ' ( ) * + - espace
- L’authentification Azure Active Directory n’est pas prise en charge.
- S’il existe des tables avec le même nom mais un schéma différent (par exemple, dbo.customers et sales.customers), une seule des tables peut être ajoutée à la synchronisation.
- Les colonnes avec des type de données définis par l’utilisateur ne sont pas prises en charge
- Le déplacement de serveurs entre différents abonnements n’est pas pris en charge. 

#### <a name="unsupported-data-types"></a>Types de données non pris en charge

- FileStream
- SQL/CLR UDT
- XMLSchemaCollection (prise en charge de XML)
- Cursor, RowVersion, Timestamp, Hierarchyid

#### <a name="unsupported-column-types"></a>Types de colonne non pris en charge

Data Sync ne peut pas synchroniser des colonnes en lecture seule ou générées par le système. Par exemple :

- Colonnes calculées
- Colonnes générées par le système pour les tables temporelles.

#### <a name="limitations-on-service-and-database-dimensions"></a>Limitations des dimensions de la base de données et du service

| **Dimensions**                                                  | **Limite**              | **Solution de contournement**              |
|-----------------------------------------------------------------|------------------------|-----------------------------|
| Nombre maximal de groupes de synchronisation auquel peut appartenir une base de données.       | 5                      |                             |
| Nombre maximal de points de terminaison dans un seul groupe de synchronisation              | 30                     |                             |
| Nombre maximal de points de terminaison locaux dans un seul groupe de synchronisation. | 5                      | Créer plusieurs groupes de synchronisation |
| Noms de la base de données, de la table, du schéma et des colonnes                       | 50 caractères par nom |                             |
| Tables dans un groupe de synchronisation                                          | 500                    | Créer plusieurs groupes de synchronisation |
| Colonnes d’une table dans un groupe de synchronisation                              | 1 000                   |                             |
| Taille de ligne de données sur une table                                        | 24 Mo                  |                             |

> [!NOTE]
> Il peut y avoir jusqu’à 30 points de terminaison dans un même groupe de synchronisation s’il n’existe qu’un seul groupe de synchronisation. S’il existe plus d’un groupe de synchronisation, le nombre total de points de terminaison dans tous les groupes de synchronisation ne peut pas dépasser 30. Si une base de données appartient à plusieurs groupes de synchronisation, elle est comptée comme plusieurs points de terminaison, et non pas un seul.

### <a name="network-requirements"></a>Configuration requise pour le réseau

> [!NOTE]
> Si vous utilisez une liaison privée, cette configuration requise du réseau ne s’applique pas. 

Quand le groupe de synchronisation est établi, le service Data Sync doit se connecter à la base de données Hub. Au moment où vous établissez le groupe de synchronisation, le serveur SQL Azure doit disposer de la configuration suivante dans ses paramètres `Firewalls and virtual networks` :

 * Le paramètre *Refuser l’accès au réseau public* doit être *désactivé*.
 * Le paramètre *Autoriser les services et les ressources Azure à accéder à ce serveur* doit avoir la valeur *Oui*, ou vous devez créer des règles IP pour les [adresses IP utilisées par le service Data Sync](network-access-controls-overview.md#data-sync).

Une fois le groupe de synchronisation créé et provisionné, vous pouvez désactiver ces paramètres. L’agent de synchronisation se connecte directement à la base de données Hub et vous pouvez utiliser les [règles IP de pare-feu](firewall-configure.md) du serveur ou des [points de terminaison privés](private-endpoint-overview.md) pour permettre à l’agent d’accéder au serveur hub.

> [!NOTE]
> Si vous modifiez les paramètres de schéma du groupe de synchronisation, vous devez autoriser le service Data Sync à accéder à nouveau au serveur afin que la base de données Hub puisse être reprovisionnée.

## <a name="faq-about-sql-data-sync"></a>FAQ sur SQL Data Sync

### <a name="how-much-does-the-sql-data-sync-service-cost"></a>Combien coûte le service de synchronisation des données SQL Data Sync ?

Aucun frais n’est facturé pour le service SQL Data Sync en lui-même. Toutefois, les frais de transfert de données pour le déplacement des données vers et depuis votre instance SQL Database vous seront facturés. Pour en savoir plus, voir [Tarification de SQL Database](https://azure.microsoft.com/pricing/details/sql-database/).

### <a name="what-regions-support-data-sync"></a>Quelles régions prennent en charge Data Sync ?

SQL Data Sync est disponible dans toutes les régions.

### <a name="is-a-sql-database-account-required"></a>Un compte SQL Database est-il requis ?

Oui. Vous devez avoir un compte SQL Database pour héberger la base de données Hub.

### <a name="can-i-use-data-sync-to-sync-between-sql-server-databases-only"></a>Puis-je utiliser Data Sync pour effectuer une synchronisation entre les bases de données SQL Server uniquement ?

Pas directement. Vous pouvez toutefois effectuer une synchronisation entre les bases de données SQL Server de façon indirecte, en créant une base de données Hub dans Azure, puis en ajoutant les bases de données locales au groupe de synchronisation.

### <a name="can-i-use-data-sync-to-sync-between-databases-in-sql-database-that-belong-to-different-subscriptions"></a>Puis-je utiliser Data Sync pour synchroniser entre des bases de données dans SQL Database qui appartiennent à différents abonnements

Oui. Vous pouvez synchroniser entre des bases de données qui appartiennent à des groupes de ressources appartenant à différents abonnements.

- Si les abonnements appartiennent au même locataire et que disposez d’autorisations sur tous les abonnements, vous pouvez configurer le groupe de synchronisation dans le portail Azure.
- Sinon, vous devez utiliser PowerShell pour ajouter les membres de synchronisation appartenant à différents abonnements.

### <a name="can-i-use-data-sync-to-sync-between-databases-in-sql-database-that-belong-to-different-clouds-like-azure-public-cloud-and-azure-china-21vianet"></a>Puis-je utiliser Data Sync pour synchroniser entre des bases de données SQL Database qui appartiennent à différents clouds (comme le cloud public Azure et Azure Chine 21Vianet)

Oui. Vous pouvez synchroniser entre des bases de données qui appartiennent à des clouds différents. Vous devez utiliser PowerShell pour ajouter les membres de synchronisation appartenant à différents abonnements.

### <a name="can-i-use-data-sync-to-seed-data-from-my-production-database-to-an-empty-database-and-then-sync-them"></a>Puis-je utiliser Data Sync pour envoyer des données de ma base de données de production vers une base de données vide, et ensuite les synchroniser ?

Oui. Créez manuellement le schéma dans la nouvelle base de données en créant le script à partir de la base de données d’origine. Après avoir créé le schéma, ajoutez les tables à un groupe de synchronisation pour copier les données et les garder synchronisées.

### <a name="should-i-use-sql-data-sync-to-back-up-and-restore-my-databases"></a>Dois-je utiliser SQL Data Sync pour sauvegarder et restaurer mes bases de données ?

Il est déconseillé d’utiliser SQL Data Sync pour créer une sauvegarde de vos données. Vous ne pouvez pas sauvegarder et restaurer à un point précis dans le temps car les synchronisations de SQL Data Sync ne sont pas affectées à des versions. Par ailleurs, SQL Data Sync ne sauvegarde pas d’autres objets SQL, notamment les procédures stockées, et n’effectue pas rapidement l’équivalent d’une opération de restauration.

Consultez [Copie d’une base de données dans Azure SQL Database](database-copy.md) pour prendre connaissance d’une technique de sauvegarde recommandée.

### <a name="can-data-sync-sync-encrypted-tables-and-columns"></a>Data Sync peut-il synchroniser des tables et colonnes chiffrées ?

- Si une base de données utilise Always Encrypted, vous ne pouvez synchroniser que les tables et colonnes qui sont *pas* chiffrées. Vous ne pouvez pas synchroniser les colonnes chiffrées, car Data Sync ne peut pas déchiffrer les données.
- Si une colonne utilise le chiffrement Column-Level Encryption (CLE), vous pouvez synchroniser la colonne tant que la taille de ligne est inférieure à la taille maximale de 24 Mo. Data Sync traite la colonne chiffrée par clé (CLE) comme des données binaires normales. Pour déchiffrer les données d’autres membres de synchronisation, vous devez avoir le même certificat.

### <a name="is-collation-supported-in-sql-data-sync"></a>Le classement est-il pris en charge dans SQL Data Sync ?

Oui. SQL Data Sync prend en charge le classement dans les scénarios suivants :

- Si les tables du schéma de synchronisation sélectionné ne sont pas encore dans votre Hub ou dans les bases de données membres, lorsque vous déployez le groupe de synchronisation, le service crée automatiquement les tables et colonnes correspondantes avec les paramètres de classement sélectionnés dans les bases de données de destination vides.
- Si les tables à synchroniser existent déjà dans votre Hub ou vos bases de données membres, SQL Data Sync requiert que les colonnes clés primaires aient le même classement dans le Hub que dans les bases de données membres pour déployer avec succès le groupe de synchronisation. Il n’existe aucune restriction de classement sur les colonnes autres que les colonnes clés primaires.

### <a name="is-federation-supported-in-sql-data-sync"></a>La fédération est-elle prise en charge dans SQL Data Sync ?

La base de données racine de fédération peut être utilisée sans limitation dans SQL Data Sync. Vous ne pouvez pas ajouter de point de terminaison de base de données fédérée à la version actuelle de SQL Data Sync.

### <a name="can-i-use-data-sync-to-sync-data-exported-from-dynamics-365-using-bring-your-own-database-byod-feature"></a>Puis-je utiliser SQL Data Sync pour synchroniser des données exportées à partir de Dynamics 365 à l’aide de la fonctionnalité BYOD (apportez votre propre base de données) ?

La fonctionnalité Dynamics 365 apportez votre propre base de données permet aux administrateurs d’exporter des entités de données depuis l’application dans leur propre base de données Microsoft Azure SQL. SQL Data Sync peut être utilisé pour synchroniser ces données dans d’autres bases de données si les données sont exportées à l’aide de **l’envoi (push) incrémentiel** (l’envoi intégral n’est pas pris en charge) et **si l’activation des déclencheurs dans la base de données cible** est définie sur **Oui**.

## <a name="next-steps"></a>Étapes suivantes

### <a name="update-the-schema-of-a-synced-database"></a>Mettre à jour le schéma d’une base de données synchronisée

Vous devez mettre à jour le schéma d’une base de données dans un groupe de synchronisation ? Les modifications de schéma ne sont pas répliquées automatiquement. Pour des solutions à ce problème, consultez les articles suivants :

- [Automatiser la réplication des modifications de schéma avec SQL Data Sync dans Azure](./sql-data-sync-update-sync-schema.md)
- [Utiliser PowerShell pour mettre à jour le schéma de synchronisation dans un groupe de synchronisation existant](scripts/update-sync-schema-in-sync-group.md)

### <a name="monitor-and-troubleshoot"></a>Superviser et dépanner

SQL Data Sync s’exécute-t-il comme prévu ? Pour surveiller l’activité et résoudre les problèmes, consultez les articles suivants :

- [Superviser SQL Data Sync avec des journaux Azure Monitor](./monitor-tune-overview.md)
- [Résoudre les problèmes liés à Azure SQL Data Sync](./sql-data-sync-troubleshoot.md)

### <a name="learn-more-about-azure-sql-database"></a>En savoir plus sur Azure SQL Database

Pour plus d’informations sur Azure SQL Database, consultez les articles suivants :

- [Vue d’ensemble des bases de données SQL](sql-database-paas-overview.md)
- [Gestion du cycle de vie des bases de données](/previous-versions/sql/sql-server-guides/jj907294(v=sql.110))
