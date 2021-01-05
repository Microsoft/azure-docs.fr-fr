---
title: La géoréplication active
description: Utilisez la géoréplication active pour créer des bases de données secondaires de bases de données individuelles accessibles en lecture dans Azure SQL Database dans des régions de centres de données identiques ou différentes.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, sstein
ms.date: 08/27/2020
ms.openlocfilehash: c7a24dbe93bf0096e327804be07acc3f67d2f03b
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94985754"
---
# <a name="creating-and-using-active-geo-replication---azure-sql-database"></a>Création et utilisation de la géoréplication active - Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

La géoréplication active est une fonctionnalité Azure SQL Database qui vous permet de créer des bases de données secondaires lisibles à partir de bases de données individuelles sur un serveur dans un centre de données identique ou différent (région).

> [!NOTE]
> La géoréplication active n’est pas prise en charge par Azure SQL Managed Instance. Pour le basculement géographique d’instances de SQL Managed Instance, utilisez des [groupes de basculement automatique](auto-failover-group-overview.md).

La géoréplication active est conçue comme une solution de continuité d’activité qui permet à l’application d’effectuer une reprise d’activité rapide au niveau des bases de données individuelles en cas de panne régionale ou à plus grande échelle. Si la géoréplication est activée, l’application peut lancer le basculement vers une base de données secondaire dans une autre région Azure. Jusqu’à quatre bases de données secondaires sont prises en charge dans des régions identiques ou différentes, et les bases de données secondaires peuvent également servir pour les requêtes d’accès en lecture seule. Le basculement doit être lancé manuellement par l’application ou l’utilisateur. Après le basculement, la nouvelle base de données primaire présente un point de terminaison de connexion différent.

> [!NOTE]
> La géoréplication active réplique les modifications en diffusant en continu le journal des transactions de la base de données. Elle n’est pas liée à la [réplication transactionnelle](/sql/relational-databases/replication/transactional/transactional-replication), qui réplique les modifications en exécutant des commandes DML (INSERT, UPDATE, DELETE).

Le diagramme suivant illustre la configuration standard d’une application cloud géoredondante avec la géoréplication active.

![Géoréplication active](./media/active-geo-replication-overview/geo-replication.png )

> [!IMPORTANT]
> SQL Database prend également en charge les groupes de basculement automatique. Pour plus d’informations, voir la section sur l’utilisation des [Groupes de basculement automatique](auto-failover-group-overview.md).

Si, pour une raison quelconque, votre base de données primaire échoue ou doit simplement être mise hors connexion, vous pouvez initier un basculement vers l’une de vos bases de données secondaires. Lorsque le basculement est activé pour l’une des bases de données secondaires, toutes les autres bases de données secondaires sont automatiquement liées à la nouvelle base de données primaire.

Vous pouvez gérer la réplication et le basculement d’une base de données individuelle ou d’un ensemble de bases de données sur un serveur ou dans un pool élastique à l’aide de la géo-réplication active. Pour ce faire, vous pouvez utiliser les éléments suivants :

- [Portail Azure](active-geo-replication-configure-portal.md)
- [PowerShell : base de données unique](scripts/setup-geodr-and-failover-database-powershell.md)
- [PowerShell : pool élastique](scripts/setup-geodr-and-failover-elastic-pool-powershell.md)
- [Transact-SQL : base de données unique ou pool élastique](/sql/t-sql/statements/alter-database-azure-sql-database)
- [API REST : base de données unique](/rest/api/sql/replicationlinks)

La géoréplication active tire parti de la technologie de [groupe de disponibilité Always On](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) du moteur de base de données pour répliquer de manière asynchrone les transactions validées sur la base de données primaire vers une base de données secondaire à l’aide de l’isolement de capture instantanée. Les groupes de basculement automatique fournissent la sémantique de groupe en plus de la géo-réplication active. Cependant, le même mécanisme de réplication asynchrone est utilisé. À un moment donné, la base de données secondaire peut être légèrement en retard sur la base de données primaire, mais les données secondaire ne peuvent jamais contenir de transactions partielles. Grâce à la redondance entre régions, les applications peuvent récupérer rapidement d’une perte permanente de tout ou partie d’un centre de données résultant de catastrophes naturelles, de graves erreurs humaines ou d’actes de malveillance. Vous trouverez les données d’objectif de point de récupération dans [Vue d’ensemble de la continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md).

> [!NOTE]
> En cas de défaillance du réseau entre deux régions, nous tentons de rétablir les connexions toutes les 10 secondes.

> [!IMPORTANT]
> Pour garantir qu’une modification critique de la base de données primaire est répliquée sur la base de données secondaire avant le basculement, vous pouvez forcer la synchronisation pour vous assurer que les modifications critiques (par exemple, les mises à jour de mot de passe) sont répliquées. La synchronisation forcée a un impact sur les performances, car elle bloque le thread appelant jusqu’à ce que toutes les transactions validées soient répliquées. Pour plus d’informations, consultez [sp_wait_for_database_copy_sync](/sql/relational-databases/system-stored-procedures/active-geo-replication-sp-wait-for-database-copy-sync). Pour superviser le décalage de réplication entre la base de données primaire et la base de données géosecondaire, voir [sys.dm_geo_replication_link_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-geo-replication-link-status-azure-sql-database).

La figure suivante présente un exemple de géo-réplication active configurée avec une base de données primaire située dans la région USA Centre Nord, ainsi qu’une base de données secondaire située dans la région USA Centre Sud.

![Relation de géo-réplication](./media/active-geo-replication-overview/geo-replication-relationship.png)

Comme les bases de données secondaires sont accessibles en lecture, elles peuvent être utilisées pour décharger des charges de travail en lecture seule, telles que des travaux de génération de rapports. Si vous utilisez la géo-réplication active, il est possible de créer la base de données secondaire dans la même région que la base de données primaire, mais cela n’augmente pas la résistance de l’application aux défaillances graves. Si vous utilisez des groupes de basculement automatique, votre base de données secondaire est toujours créée dans une autre région.

En plus de la récupération d’urgence, la géo-réplication active peut être utilisée dans les scénarios suivants :

- **Migration de base de données** : vous pouvez vous servir de la géoréplication active pour migrer une base de données d’un serveur vers un autre serveur en ligne avec un temps d’arrêt minimal.
- **Mises à niveau de l’application** : vous pouvez créer une base de données secondaire supplémentaire faisant office de copie de restauration automatique lors des mises à niveau de l’application.

Pour assurer vraiment la continuité des activités, l’ajout d’une redondance de base de données entre les centres de données n’est qu’une partie de la solution. La récupération d’une application (service) de bout en bout après une défaillance catastrophique implique la récupération de tous les composants constituant le service et tous les services dépendants. En voici quelques exemples : logiciel client (il peut s’agir par exemple d’un navigateur avec un code JavaScript personnalisé), serveurs web frontaux, ressources de stockage et DNS. Il est essentiel que tous les composants résistent aux mêmes défaillances et redeviennent disponibles dans l’objectif de délai de récupération (RTO) de votre application. Par conséquent, vous devez identifier tous les services dépendants et comprendre les garanties et les fonctionnalités qu’ils fournissent. Ensuite, vous devez prendre les mesures appropriées pour vous assurer que votre service fonctionne pendant le basculement des services dont il dépend. Pour plus d’informations sur la conception de solutions pour la récupération d’urgence, consultez la page [Designing Cloud Solutions for Disaster Recovery Using active geo-replication](designing-cloud-solutions-for-disaster-recovery.md) (Conception de solutions cloud pour la récupération d’urgence à l’aide de la géo-réplication active).

## <a name="active-geo-replication-terminology-and-capabilities"></a>Terminologie et fonctionnalités associées à la géoréplication active

- **Réplication asynchrone automatique**

  Vous pouvez créer une base de données secondaire seulement par ajout à une base de données existante. La base de données secondaire peut être créée sur n’importe quel serveur. Une fois créée, la base de données secondaire est remplie avec les données copiées à partir de la base de données primaire. Ce processus est appelé amorçage. Une fois la base de données secondaire créée et amorcée, les mises à jour de la base de données primaire sont automatiquement répliquées de manière asynchrone sur la base de données secondaire. La réplication asynchrone signifie que les transactions sont validées sur la base de données primaire avant leur réplication sur la base de données secondaire.

- **Réplicas secondaires accessibles en lecture**

  Une application peut accéder à une base de données secondaire pour des opérations en lecture seule avec des principaux de sécurité identiques ou différents de ceux utilisés pour accéder à la base de données primaire. Les bases de données secondaires fonctionnent en mode d’isolement d’instantané pour garantir que la réplication des mises à jour de la base de données primaire n’est pas retardée par des requêtes exécutées sur la base de données secondaire.

> [!NOTE]
> La relecture du journal est différée sur la base de données secondaire s’il existe des mises à jour de schéma sur le Principal. Cette dernière nécessite un verrouillage de schéma sur la base de données secondaire.

> [!IMPORTANT]
> Vous pouvez utiliser la géoréplication pour créer une base de données secondaire dans la même région que la base de données primaire. Vous pouvez utiliser cette base de données secondaire pour équilibrer les charges de travail en lecture seule dans la même région. Toutefois, une base de données secondaire dans la même région n'offre pas davantage de résilience aux défaillances et ne constitue donc pas une cible de basculement adapté à des fins de récupération d’urgence. De même, elle ne garantit en rien l’isolation de la zone de disponibilité. Utilisez le niveau de service Critique pour l’entreprise ou Premium avec une [configuration de zone redondante](high-availability-sla.md#premium-and-business-critical-service-tier-zone-redundant-availability) ou la [configuration de zone redondante](high-availability-sla.md#general-purpose-service-tier-zone-redundant-availability-preview) de niveau de service Usage général pour isoler la zone de disponibilité.
>

- **Basculement planifié**

  Le basculement planifié bascule les rôles des bases de données primaires et secondaires une fois la synchronisation complète terminée. Il s’agit d’une opération en ligne qui n’entraîne pas de perte de données. L’heure de l’opération dépend de la taille du journal des transactions sur le serveur principal qui doit être synchronisé. Le basculement planifié est conçu pour les scénarios suivants : (a) pour simuler la récupération d’urgence en production lorsque la perte de données n’est pas acceptable ; (b) pour déplacer la base de données vers une autre région ; et (c) pour renvoyer la base de données vers la région primaire après la résolution de la panne (restauration automatique).

- **Basculement non planifié**

  Un basculement non planifié ou forcé fait immédiatement basculer la base de données secondaire vers le rôle primaire sans synchronisation avec la base de données primaire. Toutes les transactions validées sur le serveur principal qui ne sont pas répliquées sur la base de données secondaire seront perdues. Cette opération est conçue comme une méthode de récupération pendant les pannes lorsque le serveur principal n’est pas accessible, mais que la disponibilité de la base de données doit être restaurée rapidement. Lorsque la base de données primaire d’origine est de nouveau en ligne, elle se reconnecte et devient une nouvelle base de données secondaire. Toutes les transactions non synchronisées avant le basculement sont conservées dans le fichier de sauvegarde, mais ne sont pas synchronisées avec le nouveau réplica principal, pour éviter les conflits. Ces transactions devront être fusionnées manuellement avec la version la plus récente de la base de données primaire.

- **Bases de données secondaires accessibles en lecture**

  Jusqu’à 4 bases de données secondaires peuvent être créées pour chaque base primaire. S’il n’existe qu’une seule base de données secondaire, en cas d’échec de celle-ci, l’application est exposée à un risque plus élevé jusqu’à la création d’une nouvelle base de données secondaire. S’il existe plusieurs bases de données secondaires, l’application reste protégée même en cas d’échec de l’une des bases de données secondaires. Les bases de données secondaires supplémentaires peuvent également être utilisées pour effectuer un scale-out des charges de travail en lecture seule

  > [!NOTE]
  > Si vous utilisez la géoréplication active pour créer une application mondialement distribuée et que vous avez besoin de fournir un accès en lecture seule aux données dans plus de quatre régions, vous pouvez créer la base de données secondaire d’une base de données secondaire (processus connu sous le nom de chaînage). De cette manière, vous pouvez assurer une mise à l’échelle quasiment illimitée pour la réplication de base de données. En outre, le chaînage réduit la charge de réplication qui pèse sur la base de données primaire. Le compromis est l’augmentation du décalage de réplication sur les bases de données secondaires situées aux extrémités.

- **Géoréplication des bases de données dans un pool élastique**

  Chaque base de données secondaire peut participer séparément à un pool élastique ou ne faire partie d’aucun pool élastique. Le choix du pool pour chaque base de données secondaire est distinct et ne dépend pas de la configuration des autres bases de données secondaires (primaires comme secondaires). Chaque pool élastique est contenu dans une seule région. Par conséquent plusieurs bases de données secondaires de la même topologie ne peuvent jamais partager un pool élastique.

- **Basculement et restauration automatique contrôlés par l’utilisateur**

  Une base de données secondaire peut être basculée explicitement à tout moment vers le rôle primaire par l’application ou l’utilisateur. Pendant une panne réelle, l’option « non planifiée » doit être utilisée pour promouvoir immédiatement une base de données secondaire en base de données primaire. Lorsque la base de données primaire en échec récupère et redevient disponible, le système marque automatiquement la base de données primaire récupérée comme base de données secondaire, et la met à jour par rapport à la nouvelle base de données primaire. En raison de la nature asynchrone de la réplication, une petite quantité de données peut être perdue lors de basculements non planifiés si une base de données primaire échoue avant la réplication des modifications les plus récentes sur la base de données secondaire. Quand une base de données primaire avec plusieurs bases de données secondaires bascule, le système reconfigure automatiquement les relations de réplication et lie les bases de données secondaires restantes à la base de données nouvellement promue comme primaire sans aucune intervention de l’utilisateur. Une fois la panne à l’origine du basculement résolue, il peut être judicieux de rétablir l’application dans la région primaire. Pour ce faire, la commande de basculement doit être appelée avec l’option « planifiée ».

## <a name="preparing-secondary-database-for-failover"></a>Préparation de la base de données secondaire pour le basculement

Pour vous assurer que votre application peut accéder immédiatement au nouveau réplica principal après le basculement, assurez-vous que les exigences d’authentification de votre serveur et de la base de données secondaires sont correctement configurées. Pour plus d’informations, consultez [Gestion de la sécurité de la base de données SQL Azure après la récupération d’urgence](active-geo-replication-security-configure.md). Pour garantir la conformité après le basculement, vérifiez que la stratégie de rétention de sauvegarde de la base de données secondaire correspond à celle du serveur principal. Ces paramètres ne font pas partie de la base de données et ne sont pas répliqués. Par défaut, le réplica secondaire est configuré avec une période de rétention de récupération jusqu`à une date et heure par défaut de sept jours. Pour plus d’informations, consultez [Sauvegardes automatisées d’une base de données SQL](automated-backups-overview.md).

> [!IMPORTANT]
> Si votre base de données est membre d'un groupe de basculement, vous ne pouvez pas lancer son basculement à l'aide de la commande de basculement de la géoréplication. Utilisez la commande de basculement du groupe. Pour basculer une base de données individuelle, vous devez d'abord la supprimer du groupe de basculement. Pour plus d'informations, consultez [Groupes de basculement](auto-failover-group-overview.md).

## <a name="configuring-secondary-database"></a>Configuration d'une base de données secondaire

Les bases de données primaire et secondaire doivent offrir le même niveau de service. Il est également vivement recommandé de créer la base de données secondaire avec les mêmes redondance de stockage de sauvegarde taille de calcul (DTU ou vCores) que la base de données primaire. En cas de charge de travail d’écriture importante de la base de données primaire, une base de données secondaire dotée d’une taille de calcul inférieure peut ne pas suivre. Cela entraîne le décalage de la phase de restauration par progression sur la base de données secondaire et l’indisponibilité potentielle de la base de données secondaire. Pour atténuer ces risques, la géoréplication active limite si nécessaire le taux de journalisation des transactions de la base de données primaire pour permettre à ses bases de données secondaires de rattraper le retard.

Après un basculement, une configuration de base de données secondaire déséquilibrée peut aussi altérer le niveau de performance de l’application en raison de la capacité de calcul insuffisante de la nouvelle base de données primaire. Dans ce cas, il est nécessaire d’effectuer un scale-up de l’objectif du service de base de données au niveau nécessaire, ce qui peut nécessiter beaucoup de temps et de ressources de calcul. Un basculement à [haute disponibilité](high-availability-sla.md) est également nécessaire à la fin du processus de scale-up.

Si vous décidez de créer une base de données secondaire avec une taille de calcul inférieure, vous pouvez utiliser le graphique de pourcentage d’E/S du journal dans le portail Azure pour estimer la taille de calcul minimale de la base de données secondaire qui est nécessaire pour supporter la charge de réplication. Par exemple, si votre base de données primaire est P6 (1000 DTU) et que son pourcentage d’écriture dans le journal est de 50 %, la base de données secondaire doit être au moins P4 (500 DTU). Pour récupérer les données d’E/S historiques du journal, utilisez la vue [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database). Pour récupérer les données d’écriture récentes dans le journal avec une granularité plus élevée qui reflète mieux les pics à court terme du taux de journalisation, utilisez la vue [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database).

La limitation du taux de journalisation des transactions sur la base de données primaire en raison d’une réduction de la taille de calcul sur une base de données secondaire est signalée à l’aide du type d’attente HADR_THROTTLE_LOG_RATE_MISMATCHED_SLO, visible dans les vues de base de données [sys.dm_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) et [sys.dm_os_wait_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql).

Par défaut, la redondance de stockage de sauvegarde de la base de données secondaire est identique à celle de la base de données primaire. Vous pouvez configurer la base de données secondaire avec une redondance de stockage de sauvegarde différente. Les sauvegardes sont toujours effectuées sur la base de données primaire. Si la base de données secondaire est configurée avec une redondance de stockage de sauvegarde différente, après le basculement effectué lors de la promotion de la base de données secondaire en base de données primaire, les sauvegardes sont facturées en fonction de la redondance de stockage sélectionnée sur la nouvelle base de données primaires (ancienne secondaire). 

> [!NOTE]
> Le taux de journalisation des transactions sur la base de données primaire peut être limité pour des raisons non liées à une taille de calcul inférieure sur une base de données secondaire. Ce genre de limitation peut se produire même si la base de données secondaire a une taille de calcul identique ou supérieure à celle de la base de données primaire. Pour plus d’informations, notamment sur les types d’attente pour les différents genres de limitation du taux de journalisation, consultez [Gouvernance relative au taux de journalisation des transactions](resource-limits-logical-server.md#transaction-log-rate-governance).

> [!NOTE]
> La redondance configurable du stockage de sauvegarde Azure SQL Database est actuellement disponible uniquement en préversion publique dans la région Brésil Sud et mise à la disposition générale dans la région Azure Asie Sud-Est. Lorsque la base de données source est créée avec une redondance de stockage de sauvegarde redondante localement ou redondante interzone, la création d’une base de données secondaire dans une autre région Azure n’est pas prise en charge. 

Pour plus d’informations sur les tailles de calcul SQL Database, consultez [Présentation des niveaux de service SQL Database](purchasing-models.md).

## <a name="cross-subscription-geo-replication"></a>Géoréplication entre abonnements

Pour configurer la géoréplication active entre deux bases de données appartenant à des abonnements différents (que ce soit sous le même locataire ou non), vous devez suivre la procédure spéciale décrite dans cette section.  La procédure repose sur des commandes SQL et requiert ce qui suit :

- Création d'un ID de connexion privilégié sur les deux serveurs
- Ajout de l'adresse IP à la liste verte du client qui procède à la modification sur les deux serveurs (comme l'adresse IP de l'hôte qui exécute SQL Server Management Studio).

Le client qui procède aux modifications doit disposer d'un accès réseau au serveur principal. Bien que la même adresse IP client doive être ajoutée à la liste verte sur le serveur secondaire, la connectivité réseau au serveur secondaire n'est pas strictement nécessaire.

### <a name="on-the-master-of-the-primary-server"></a>Sur le maître du serveur principal

1. Ajoutez l'adresse IP à la liste verte du client qui procède aux modifications (pour plus d'informations, consultez [Configurer le pare-feu](firewall-configure.md)).
1. Créez un ID de connexion dédié à la configuration de la géoréplication active (et, si nécessaire, modifiez les informations d'identification) :

   ```sql
   create login geodrsetup with password = 'ComplexPassword01'
   ```

1. Créez un utilisateur et attribuez-lui le rôle dbmanager :

   ```sql
   create user geodrsetup for login geodrsetup
   alter role dbmanager add member geodrsetup
   ```

1. Prenez note du SID associé au nouvel ID de connexion en utilisant la requête suivante :

   ```sql
   select sid from sys.sql_logins where name = 'geodrsetup'
   ```

### <a name="on-the-source-database-on-the-primary-server"></a>Sur la base de données source du serveur principal

1. Créez un utilisateur pour le même ID de connexion :

   ```sql
   create user geodrsetup for login geodrsetup
   ```

1. Attribuez le rôle db_owner à l'utilisateur :

   ```sql
   alter role db_owner add member geodrsetup
   ```

### <a name="on-the-master-of-the-secondary-server"></a>Sur le maître du serveur secondaire

1. Ajoutez l’adresse IP du client à la liste autorisée sous les règles de pare-feu pour le serveur secondaire. Vérifiez que l’adresse IP du client ajoutée sur le serveur principal est exactement la même que celle qui a été ajoutée au serveur secondaire. Il s’agit d’une étape obligatoire à effectuer avant d’exécuter la commande ALTER DATABASE ADD SECONDARY pour lancer la géo-réplication.

1. Créez le même ID de connexion que sur le serveur principal, en utilisant les mêmes nom d'utilisateur/mot de passe et le même SID :

   ```sql
   create login geodrsetup with password = 'ComplexPassword01', sid=0x010600000000006400000000000000001C98F52B95D9C84BBBA8578FACE37C3E
   ```

1. Créez un utilisateur et attribuez-lui le rôle dbmanager :

   ```sql
   create user geodrsetup for login geodrsetup;
   alter role dbmanager add member geodrsetup
   ```

### <a name="on-the-master-of-the-primary-server"></a>Sur le maître du serveur principal

1. Connectez-vous au maître du serveur principal à l'aide du nouvel ID de connexion.
1. Créez un réplica secondaire de la base de données source sur le serveur secondaire (si nécessaire, modifiez le nom de la base de données et le nom du serveur) :

   ```sql
   alter database dbrep add secondary on server <servername>
   ```

Au terme de la configuration initiale, les utilisateurs, les ID de connexion et les règles de pare-feu créés peuvent être supprimés.

## <a name="keeping-credentials-and-firewall-rules-in-sync"></a>Synchronisation des informations d’identification et des règles de pare-feu

Nous recommandons d’utiliser des [règles de pare-feu IP au niveau de la base de données](firewall-configure.md) pour les bases de données géorépliquées, de façon à ce que ces règles puissent être répliquées avec la base de données, garantissant ainsi que toutes les bases de données secondaires ont les mêmes règles de pare-feu IP que la base de données primaire. Cette approche évite aux clients de devoir configurer et tenir à jour manuellement les règles de pare-feu sur les serveurs hébergeant les bases de données primaire et secondaires. De même, le recours à des [utilisateurs de base de données contenus](logins-create-manage.md) pour l’accès aux données garantit que les bases de données primaires et secondaires ont toujours les mêmes informations d’identification d’utilisateur afin qu’un basculement n’entraîne aucune interruption due à une discordance d’ID de connexion et de mots de passe. Avec l’ajout [d’Azure Active Directory](../../active-directory/fundamentals/active-directory-whatis.md), les clients peuvent gérer l’accès utilisateur aux bases de données primaires et secondaires. Cela élimine également la nécessité de gérer les informations d’identification dans l’ensemble des bases de données.

## <a name="upgrading-or-downgrading-primary-database"></a>Mise à niveau ou rétrogradation de la base de données primaire

Vous pouvez augmenter ou diminuer la taille de calcul d’une base de données primaire (au sein du même niveau de service, mais pas entre les niveaux Usage général et Critique pour l’entreprise) sans déconnecter les bases de données secondaires. Lors d’une mise à niveau, nous vous recommandons de mettre à niveau la base de données secondaire dans un premier temps, avant de mettre à niveau la base de données primaire. Lors d’une rétrogradation, inversez l’ordre : rétrogradez tout d’abord la base de données primaire, puis dans un second temps la base de données secondaire. Lorsque vous passez la base de données à un niveau de service supérieur ou inférieur, cette recommandation est appliquée.

> [!NOTE]
> Si vous avez créé une base de données secondaire dans le cadre de la configuration des groupes de basculement, il n’est pas conseillé de passer la base de données secondaire à un niveau de service inférieur. En effet, votre couche Données pourrait manquer de capacité pour traiter votre charge de travail normale après l’activation du basculement.

> [!IMPORTANT]
> Pour permettre la mise à l'échelle vers le niveau supérieur d'une base de données primaire dans un groupe de basculement, la base de données secondaire doit avoir été préalablement mise à l'échelle vers le niveau supérieur. Si vous tentez de mettre à l’échelle la base de données primaire préalablement à la base de données secondaire, l'erreur suivante peut s'afficher :
>
> `Error message: The source database 'Primaryserver.DBName' cannot have higher edition than the target database 'Secondaryserver.DBName'. Upgrade the edition on the target before upgrading the source.`
>

## <a name="preventing-the-loss-of-critical-data"></a>Prévention de la perte de données critiques

En raison de la latence élevée des réseaux étendus, la copie continue utilise un mécanisme de réplication asynchrone. La perte de certaines données reste donc inévitable en cas de défaillance. Or, pour certaines applications, une perte de données est inacceptable. Pour protéger ces mises à jour critiques, un développeur d’applications peut appeler la procédure système [sp_wait_for_database_copy_sync](/sql/relational-databases/system-stored-procedures/active-geo-replication-sp-wait-for-database-copy-sync) immédiatement après la validation de la transaction. L’appel de **sp_wait_for_database_copy_sync** bloque le thread appelant jusqu’à ce que la dernière transaction validée ait été transmise à la base de données secondaire. Toutefois, il n’attend pas que les transactions transmises soient relues et validées sur la base de données secondaire. **sp_wait_for_database_copy_sync** est limité à une relation de copie continue spécifique. Tout utilisateur disposant de droits de connexion à la base de données primaire peut appeler cette procédure.

> [!NOTE]
> **sp_wait_for_database_copy_sync** empêche la perte de données après un basculement, mais il ne garantit pas la synchronisation complète pour l’accès en lecture. Le délai causé par un appel de procédure **sp_wait_for_database_copy_sync** peut être significatif et dépend de la taille du journal des transactions au moment de l’appel.

## <a name="monitoring-geo-replication-lag"></a>Surveiller le décalage de la géoréplication

Pour surveiller le décalage par rapport au RPO, utilisez la colonne *replication_lag_sec* de [sys.dm_geo_replication_link_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-geo-replication-link-status-azure-sql-database) sur la base de données primaire. Elle affiche le décalage en secondes entre les transactions validées sur la base de données primaire et les transactions persistantes sur la base de données secondaire. Par exemple, une valeur de décalage d'une seconde indique qu'en cas d'interruption de la base de données primaire, à ce moment précis, et de basculement, les transactions les plus récentes ne seront pas sauvegardées à hauteur d'une seconde.

Pour mesurer le décalage ayant trait aux modifications de la base de données primaire appliquées à la base de données secondaire, c'est-à-dire pour lire à partir de la base de données secondaire, comparez la valeur *last_commit* sur la base de données secondaire avec la même valeur sur la base de données primaire.

> [!NOTE]
> Sur la base de données primaire, *replication_lag_sec* peut présenter une valeur NULL, ce qui signifie que la base de données primaire ne connaît pas précisément l'état de la base de données secondaire.   Cela se produit généralement après le redémarrage du processus et relève d'une situation temporaire. Envisagez d’alerter l’application si *replication_lag_sec* présente une valeur NULL pendant une période prolongée. En effet, la base de données secondaire peut ne pas être en mesure de communiquer avec la base de données primaire en raison d'un échec de connectivité permanent. D'autres cas de figure peuvent aussi être à l'origine d'une importante différence de valeur *last_commit* entre les bases de données primaire et secondaire. Par exemple, si une validation intervient sur la base de données primaire après une longue période sans modifications, la différence augmente considérablement avant de redevenir rapidement nulle. Envisagez un état d'erreur lorsque la différence entre ces deux valeurs demeure importante pendant une période prolongée.

## <a name="programmatically-managing-active-geo-replication"></a>Gestion de la géo-réplication active par programmation

Comme indiqué plus haut, la géoréplication active peut aussi être gérée par programme à l’aide d’Azure PowerShell et de l’API REST. Les tableaux ci-dessous décrivent l’ensemble des commandes disponibles. La géoréplication active comprend un ensemble d’API Azure Resource Manager pour la gestion, notamment [l’API REST Azure SQL Database](/rest/api/sql/) et les [applets de commande Azure PowerShell](/powershell/azure/). Ces API nécessitent l’utilisation de groupes de ressources et la prise en charge du contrôle d’accès en fonction du rôle (RBAC). Pour plus d’informations sur l’implémentation de rôles d’accès, consultez la page sur le [contrôle d’accès en fonction du rôle Azure (RBAC Azure)](../../role-based-access-control/overview.md).

### <a name="t-sql-manage-failover-of-single-and-pooled-databases"></a>T-SQL : Gérer le basculement des bases de données uniques et mises en pool

> [!IMPORTANT]
> Ces commandes Transact-SQL s’appliquent uniquement à la géoréplication active et ne s’appliquent pas aux groupes de basculement. Par conséquent, elles ne s’appliquent pas non plus aux instances de SQL Managed Instance, car elles prennent uniquement en charge les groupes de basculement.

| Commande | Description |
| --- | --- |
| [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?preserve-view=true&view=azuresqldb-current) |Utilise l’argument ADD SECONDARY ON SERVER afin de créer une base de données secondaire pour une base de données existante puis lance la réplication des données |
| [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?preserve-view=true&view=azuresqldb-current) |Utilise l’argument FAILOVER ou FORCE_FAILOVER_ALLOW_DATA_LOSS pour basculer d’une base de données secondaire à une base de données principale afin de lancer le basculement |
| [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql?preserve-view=true&view=azuresqldb-current) |Utilise l’argument REMOVE SECONDARY ON SERVER pour mettre fin à une réplication de données entre une base de données SQL et la base de données secondaire spécifiée. |
| [sys.geo_replication_links](/sql/relational-databases/system-dynamic-management-views/sys-geo-replication-links-azure-sql-database) |Retourne des informations concernant tous les liens de réplication existants pour chaque base de données sur un serveur. |
| [sys.dm_geo_replication_link_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-geo-replication-link-status-azure-sql-database) |Obtient l’heure de la dernière réplication, le dernier décalage de la réplication et d’autres informations sur le lien de réplication pour une base de données spécifique. |
| [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database) |Affiche l’état de toutes les opérations de base de données, y compris l’état des liens de réplication. |
| [sp_wait_for_database_copy_sync](/sql/relational-databases/system-stored-procedures/active-geo-replication-sp-wait-for-database-copy-sync) |oblige l’application à attendre que toutes les transactions validées sont répliquées et acceptées par la base de données secondaire active. |
|  | |

### <a name="powershell-manage-failover-of-single-and-pooled-databases"></a>PowerShell : Gérer le basculement des bases de données uniques et mises en pool

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par Azure SQL Database, mais tous les développements futurs sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm.

| Applet de commande | Description |
| --- | --- |
| [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) |Obtient une ou plusieurs bases de données. |
| [New-AzSqlDatabaseSecondary](/powershell/module/az.sql/new-azsqldatabasesecondary) |Crée une base de données secondaire pour une base de données existante puis lance la réplication des données. |
| [Set-AzSqlDatabaseSecondary](/powershell/module/az.sql/set-azsqldatabasesecondary) |Bascule d’une base de données secondaire à une base de données principale afin de lancer le basculement. |
| [Remove-AzSqlDatabaseSecondary](/powershell/module/az.sql/remove-azsqldatabasesecondary) |Met fin à une réplication de données entre une base de données SQL et la base de données secondaire spécifiée. |
| [Get-AzSqlDatabaseReplicationLink](/powershell/module/az.sql/get-azsqldatabasereplicationlink) |Obtient les liens de géo-réplication entre une instance Azure SQL Database et un groupe de ressources ou un serveur SQL logique. |
|  | |

> [!IMPORTANT]
> Pour plus d’exemples de scripts, consultez [Configurer et basculer une base de données unique à l’aide de la géoréplication active](scripts/setup-geodr-and-failover-database-powershell.md) et [Configurer et basculer une base de données mise en pool à l’aide de la géoréplication active](scripts/setup-geodr-and-failover-elastic-pool-powershell.md).

### <a name="rest-api-manage-failover-of-single-and-pooled-databases"></a>API REST : Gérer le basculement des bases de données uniques et mises en pool

| API | Description |
| --- | --- |
| [.Créer ou mettre à jour la base de données (createMode=Restore)](/rest/api/sql/databases/createorupdate) |Crée, met à jour ou restaure une base de données principale ou secondaire. |
| [Créer ou mettre à jour l’état de la base de données](/rest/api/sql/databases/createorupdate) |Retourne l’état durant une opération de création. |
| [Définir la base de données secondaire comme principale (basculement planifié)](/rest/api/sql/replicationlinks/failover) |Définit la base de données secondaire principale via basculement à partir de la base de données primaire actuelle. **Cette option n’est pas prise en charge pour SQL Managed Instance.**|
| [Définir la base de données secondaire comme principale (basculement non planifié)](/rest/api/sql/replicationlinks/failoverallowdataloss) |Définit la base de données secondaire principale via basculement à partir de la base de données primaire actuelle. Cette opération peut entraîner une perte de données. **Cette option n’est pas prise en charge pour SQL Managed Instance.**|
| [Obtenir un lien de réplication](/rest/api/sql/replicationlinks/get) |Obtient un liens de réplication spécifique pour une base de données particulière dans un partenariat de géo-réplication. Récupère les informations visibles dans la vue de catalogue sys.geo_replication_links. **Cette option n’est pas prise en charge pour SQL Managed Instance.**|
| [Liens de réplication - Liste par base de données](/rest/api/sql/replicationlinks/listbydatabase) | Obtient tous les liens de réplication pour une base de données donnée dans un partenariat de géo-réplication. Récupère les informations visibles dans la vue de catalogue sys.geo_replication_links. |
| [Supprimer un lien de réplication](/rest/api/sql/replicationlinks/delete) | Supprime un lien de réplication de base de données. Opération impossible pendant le basculement. |
|  | |

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir des exemples de scripts, consultez :
  - [Configurer et basculer une base de données unique à l’aide de la géoréplication active](scripts/setup-geodr-and-failover-database-powershell.md)
  - [Configurer et basculer une base de données mise en pool à l’aide de la géoréplication active](scripts/setup-geodr-and-failover-elastic-pool-powershell.md)
- SQL Database prend également en charge les groupes de basculement automatique. Pour plus d’informations, voir la section sur l’utilisation des [Groupes de basculement automatique](auto-failover-group-overview.md).
- Pour une vue d’ensemble de la continuité des activités et des scénarios, consultez [Vue d’ensemble de la continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md)
- Pour en savoir plus sur les sauvegardes automatisées d’une base de données Azure SQL, consultez [Sauvegardes automatisées d’une base de données SQL](automated-backups-overview.md).
- Pour en savoir plus sur l’utilisation des sauvegardes automatisées pour la récupération, consultez [Restaurer une base de données à partir des sauvegardes initiées par le service](recovery-using-backups.md).
- Pour en savoir plus sur les exigences d’authentification pour de nouveaux serveurs et bases de données primaires, consultez [Gestion de la sécurité de la base de données SQL Azure après la récupération d’urgence](active-geo-replication-security-configure.md).
