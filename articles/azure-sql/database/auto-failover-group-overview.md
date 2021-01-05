---
title: Groupes de basculement automatique
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Les groupes de basculement automatique vous permettent de gérer la réplication et le basculement automatique/coordonné d’un groupe de bases de données sur un serveur, ou de toutes les bases de données dans une instance gérée.
services: sql-database
ms.service: sql-db-mi
ms.subservice: high-availability
ms.custom: sqldbrb=2, devx-track-azurecli
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: mathoma, sstein
ms.date: 11/16/2020
ms.openlocfilehash: 0d2248b9c0a289f5e4f9f2f8e987365ab58c49c0
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94988542"
---
# <a name="use-auto-failover-groups-to-enable-transparent-and-coordinated-failover-of-multiple-databases"></a>Utiliser les groupes de basculement automatique pour permettre le basculement transparent et coordonné de plusieurs bases de données
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Les groupes de basculement automatique vous permettent de gérer la réplication et le basculement d’un groupe de bases de données sur un serveur, ou de toutes les bases de données d’une instance gérée vers une autre région. Il s’agit d’une abstraction déclarative sur la fonctionnalité de [géoréplication active](active-geo-replication-overview.md) existante, conçue pour simplifier le déploiement et la gestion des bases de données géorépliquées à l’échelle. Vous pouvez déclencher le basculement manuellement, ou le déléguer au service Azure via une stratégie définie par l’utilisateur. Cette dernière option vous permet de récupérer automatiquement plusieurs bases de données associées dans une région secondaire, après une défaillance grave ou tout autre événement non planifié entraînant une perte totale ou partielle de la disponibilité de l’instance SQL Database ou SQL Managed Instance dans la région primaire. Un groupe de basculement peut inclure une ou plusieurs bases de données, généralement utilisées par la même application. En outre, vous pouvez utiliser les bases de données secondaires accessibles en lecture pour décharger les charges de travail de requêtes en lecture seule. Comme les groupes de basculement automatique impliquent de nombreuses bases de données, celles-ci doivent être configurées sur le serveur primaire. Les groupes de basculement automatique prennent en charge la réplication de toutes les bases de données du groupe vers une seule instance ou un seul serveur secondaire situé dans une autre région.

> [!NOTE]
> Si vous souhaitez disposer de plusieurs instances Azure SQL Database secondaires dans la même région ou dans des régions différentes, utilisez la [géo-réplication active](active-geo-replication-overview.md).

Lorsque vous utilisez des groupes de basculement automatique avec une stratégie de basculement automatique, toute panne qui affecte une ou plusieurs des bases de données du groupe donne lieu à un basculement automatique. En général, il s’agit d’incidents qui ne peuvent pas être corrigés automatiquement par les opérations de haute disponibilité automatiques intégrées. Les déclencheurs de basculement peuvent être, par exemple, un incident provoqué par un anneau de locataire SQL Database ou un anneau de contrôle ayant cessé de fonctionner en raison d’une fuite de mémoire du noyau du système d’exploitation sur plusieurs nœuds de calcul, ou encore un incident causé par un ou plusieurs anneaux de locataire en panne, car un câble réseau incorrect a été coupé au cours d’une désactivation de routine du matériel.  Pour plus d’informations, consultez [Haute disponibilité de SQL Database](high-availability-sla.md).

En outre, les groupes de basculement automatique fournissent des points de terminaison d’écouteur de lecture-écriture et de lecture seule qui restent inchangés pendant les basculements. Que vous utilisiez l’activation manuelle ou automatique du basculement, ce dernier bascule toutes les bases de données secondaires du groupe en bases de données primaires. Une fois le basculement des bases de données terminé, l’enregistrement DNS est automatiquement mis à jour pour rediriger les points de terminaison vers la nouvelle région. Pour en savoir plus sur les données d’objectif de point et de délai de récupération, voir [Vue d’ensemble de la continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md).

Lorsque vous utilisez des groupes de basculement automatique avec une stratégie de basculement automatique, toute panne qui affecte les bases de données sur un serveur ou une instance gérée donne lieu à un basculement automatique. Vous pouvez gérer le groupe de basculement automatique à l’aide des méthodes suivantes :

- [Azure portal](geo-distributed-application-configure-tutorial.md)
- [Azure CLI : Groupe de basculement](scripts/add-database-to-failover-group-cli.md)
- [PowerShell : Groupe de basculement](scripts/add-database-to-failover-group-powershell.md)
- [API REST : Groupe de basculement](/rest/api/sql/failovergroups).

Après le basculement, assurez-vous que les exigences d’authentification de votre serveur et de votre base de données ou de votre instance sont configurées sur la nouvelle base de données primaire. Pour plus d’informations, consultez [Gestion de la sécurité de la base de données SQL Azure après la récupération d’urgence](active-geo-replication-security-configure.md).

Pour assurer vraiment la continuité des activités, l’ajout d’une redondance de base de données entre les centres de données n’est qu’une partie de la solution. La récupération d’une application (service) de bout en bout après une défaillance catastrophique implique la récupération de tous les composants constituant le service et tous les services dépendants. En voici quelques exemples : logiciel client (il peut s’agir par exemple d’un navigateur avec un code JavaScript personnalisé), serveurs web frontaux, ressources de stockage et DNS. Il est essentiel que tous les composants résistent aux mêmes défaillances et redeviennent disponibles dans l’objectif de délai de récupération (RTO) de votre application. Par conséquent, vous devez identifier tous les services dépendants et comprendre les garanties et les fonctionnalités qu’ils fournissent. Ensuite, vous devez prendre les mesures appropriées pour vous assurer que votre service fonctionne pendant le basculement des services dont il dépend. Pour plus d’informations sur la conception de solutions pour la récupération d’urgence, consultez la page [Designing Cloud Solutions for Disaster Recovery Using active geo-replication](designing-cloud-solutions-for-disaster-recovery.md) (Conception de solutions cloud pour la récupération d’urgence à l’aide de la géo-réplication active).

## <a name="terminology-and-capabilities"></a>Terminologie et fonctionnalités

- **Groupe de basculement (FOG)**

  Un groupe de basculement est un groupe nommé de bases de données gérées par un même serveur ou dans une instance gérée, et qui peut basculer entièrement vers une autre région lorsqu’une partie ou la totalité des bases de données primaires devient indisponible en raison d’une panne dans la région primaire. Lorsqu’il est créé pour une instance SQL Managed Instance, un groupe de basculement contient toutes les bases de données utilisateur de l’instance. Ainsi, il n’est possible de configurer qu’un seul groupe de basculement par instance.
  
  > [!IMPORTANT]
  > Le nom du groupe de basculement doit être unique à l’échelle globale dans le domaine `.database.windows.net`.

- **Serveurs**

     Avec les serveurs, une partie ou la totalité des bases de données utilisateur d’un même serveur peut être placée dans un groupe de basculement. En outre, un serveur prend en charge plusieurs groupes de basculement sur un seul serveur.

- **Primaire**

  Le serveur ou l’instance gérée qui héberge les bases de données primaires dans le groupe de basculement.

- **Secondaire**

  Le serveur ou l’instance gérée qui héberge les bases de données secondaires dans le groupe de basculement. Le serveur logique secondaire ne peut pas être situé dans la même région que le serveur logique primaire.

- **Ajouter des bases de données uniques au groupe de basculement**

  Vous pouvez placer plusieurs bases de données uniques sur le même serveur, dans le même groupe de basculement. Si vous ajoutez une base de données unique au groupe de basculement, une base de données secondaire de la même édition et de la même taille de calcul est automatiquement créée sur le serveur secondaire  que vous avez spécifié à la création du groupe de basculement. Si vous ajoutez une base de données qui présente déjà une base de données secondaire dans le serveur secondaire, ce lien de géoréplication est hérité par le groupe. Lors de l’ajout d’une base de données qui présente déjà une base de données secondaire sur un serveur qui ne fait pas partie du groupe de basculement, une nouvelle base de données secondaire est créée sur le serveur secondaire.

  > [!IMPORTANT]
  > Assurez-vous que le serveur secondaire ne dispose pas d’une base de données portant le même nom, sauf s’il s’agit d’une base de données secondaire existante. Dans les groupes de basculement d’une instance SQL Managed Instance, toutes les bases de données utilisateur sont répliquées. Vous ne pouvez pas choisir un sous-ensemble de bases de données utilisateur pour la réplication dans le groupe de basculement.

- **Ajouter des bases de données d’un pool élastique au groupe de basculement**

  Vous pouvez placer une partie ou la totalité des bases de données d’un pool élastique dans le même groupe de basculement. Si la base de données primaire se trouve dans un pool élastique, la base de données secondaire est automatiquement créée dans le pool élastique du même nom (pool secondaire). Vérifiez que le serveur secondaire contient un pool élastique portant exactement le même nom et offrant une capacité disponible suffisante pour héberger les bases de données secondaires qui seront créées par le groupe de basculement. Si vous ajoutez une base de données dans le pool qui présente déjà une base de données secondaire dans le pool secondaire, ce lien de géoréplication est hérité par le groupe. Si vous ajoutez une base de données qui présente déjà une base de données secondaire sur un serveur ne faisant pas partie du groupe de basculement, une nouvelle base de données secondaire est créée dans le pool secondaire.
  
- **Amorçage initial**

  Lorsque vous ajoutez des bases de données, des pools élastiques ou des instances gérées à un groupe de basculement, il y a d’abord une phase d’amorçage initiale avant le démarrage de la réplication des données. La phase d’amorçage initiale est l’opération la plus longue et la plus coûteuse. Une fois l’amorçage initial terminé, les données sont synchronisées, puis seules les modifications de données ultérieures sont répliquées. Le temps nécessaire à la réalisation de l’amorçage initial dépend de la taille de vos données, du nombre de bases de données répliquées et de la vitesse de la liaison entre les entités dans le groupe de basculement. Dans des circonstances normales, la vitesse d’amorçage possible peut atteindre 500 Go par heure pour SQL Database, et 360 Go par heure pour SQL Managed Instance. L’amorçage est effectué pour toutes les bases de données en parallèle.

  Pour SQL Managed Instance, prenez en compte la vitesse de la liaison Express Route entre les deux instances lors de l’estimation de la durée de la phase d’amorçage initiale. Si la vitesse de la liaison entre les deux instances est plus lente que nécessaire, la durée de l’amorçage risque d’être sensiblement perturbée. Vous pouvez utiliser la vitesse d’amorçage indiquée, ainsi que le nombre de bases de données, la taille totale des données et la vitesse de liaison pour estimer la durée de la phase d’amorçage initiale avant le début de la réplication des données. Par exemple, pour une base de données individuelle de 100 Go, la phase d’amorçage initiale prendrait environ 1,2 heure si la liaison est en mesure d’envoyer (push) 84 Go par heure, et si aucune autre base de données n’est en cours d’amorçage. Si la liaison ne peut transférer que 10 Go par heure, l’amorçage d’une base de données de 100 Go prendra environ 10 heures. S’il faut répliquer plusieurs bases de données, l’amorçage est effectué en parallèle et, lorsqu’il est associé à une vitesse de liaison lente, la phase d’amorçage initiale peut prendre beaucoup plus de temps. Cela est particulièrement vrai si l’amorçage parallèle des données de toutes les bases de données est supérieur à la bande passante de liaison disponible. Si la bande passante réseau entre deux instances est limitée et que vous ajoutez plusieurs instances gérées à un groupe de basculement, ajoutez-les de manière successive, une par une. Étant donné une référence SKU de passerelle de taille appropriée entre les deux instances managées, si la bande passante du réseau d’entreprise le permet, il est possible d’atteindre des vitesses allant jusqu’à 360 Go par heure.  

- **Zone DNS**

  ID unique qui est généré automatiquement lorsqu’une instance SQL Managed Instance est créée. Un certificat multidomaine (SAN) est approvisionné pour cette instance afin d’authentifier les connexions clientes effectuées auprès de toutes les instances présentes dans la même zone DNS. Les deux instances gérées d’un même groupe de basculement doivent partager la zone DNS.
  
  > [!NOTE]
  > Il n’est pas nécessaire de disposer d’un ID de zone DNS pour les groupes de basculement créés pour SQL Database.

- **Écouteur en lecture-écriture du groupe de basculement**

  Un enregistrement DNS CNAME qui pointe vers l’URL du serveur primaire actuel. Il est généré automatiquement lorsque le groupe de basculement est créé, et permet à la charge de travail en lecture-écriture de se reconnecter en toute transparence à la base de données primaire lorsqu’elle est modifiée après le basculement. Lorsque le groupe de basculement est créé sur un serveur, l’enregistrement DNS CNAME pour l’URL de l’écouteur est formé comme suit : `<fog-name>.database.windows.net`. Lorsque le groupe de basculement est créé sur une instance SQL Managed Instance, l’enregistrement DNS CNAME pour l’URL de l’écouteur est formé comme suit : `<fog-name>.<zone_id>.database.windows.net`.

- **Écouteur en lecture seule du groupe de basculement**

  Enregistrement DNS CNAME formé pour l’écouteur en lecture seule pointant vers l’URL du serveur secondaire. Il est créé automatiquement lorsque le groupe de basculement est créé et permet à la charge de travail SQL en lecture seule de se connecter en toute transparence à la base de données secondaire à l’aide des règles d’équilibrage de charge spécifiées. Lorsque le groupe de basculement est créé sur un serveur, l’enregistrement DNS CNAME pour l’URL de l’écouteur est formé comme suit : `<fog-name>.secondary.database.windows.net`. Lorsque le groupe de basculement est créé sur une instance SQL Managed Instance, l’enregistrement DNS CNAME pour l’URL de l’écouteur est formé comme suit : `<fog-name>.secondary.<zone_id>.database.windows.net`.

- **Stratégie de basculement automatique**

  Par défaut, un groupe de basculement est configuré avec une stratégie de basculement automatique. Azure déclenche le basculement dès que la défaillance est détectée et que la période de grâce est arrivée à expiration. Le système doit vérifier que la panne ne peut pas être atténuée par [l’infrastructure de haute disponibilité](high-availability-sla.md) intégrée en raison de l’échelle de l’impact. Si vous souhaitez contrôler le workflow de basculement à partir de l’application ou manuellement, vous pouvez désactiver le basculement automatique.
  
  > [!NOTE]
  > Compte tenu du fait que la vérification de l’étendue de la panne et que la rapidité avec laquelle elle peut être atténuée impliquent des actions humaines de la part de l’équipe des opérations, la période de grâce ne peut pas être fixée en dessous d’une heure. Cette limitation s’applique à toutes les bases de données du groupe de basculement, quel que soit l’état de la synchronisation de leurs données.

- **Stratégie de basculement en lecture seule**

  Par défaut, le basculement de l’écouteur en lecture seule est désactivé. Il garantit que les performances du serveur principal ne sont pas affectées lorsque le serveur secondaire est hors connexion. Toutefois, cela signifie également que les sessions en lecture seule ne seront pas en mesure de se connecter tant que le serveur secondaire n’aura pas été récupérée. Si vous ne pouvez pas tolérer des temps d’arrêt pour les sessions en lecture seule et que vous pouvez utiliser le serveur principal pour le trafic en lecture seule et en lecture-écriture au prix d’une dégradation potentielle des performances du serveur principal, vous pouvez activer le basculement pour l’écouteur en lecture seule en configurant la propriété `AllowReadOnlyFailoverToPrimary`. Dans ce cas, le trafic en lecture seule est automatiquement redirigé vers le serveur principal si le serveur secondaire est indisponible.

  > [!NOTE]
  > La propriété `AllowReadOnlyFailoverToPrimary` n’a d’effet que si la stratégie de basculement automatique est activée et qu’un basculement automatique a été déclenché par Azure. Dans ce cas, si la propriété est définie sur True, le nouveau serveur principal servira les sessions en lecture-écriture et en lecture seule.

- **Basculement planifié**

   Le basculement planifié effectue une synchronisation complète entre les bases de données primaire et secondaire avant que la seconde ne bascule dans le rôle primaire. Cela empêche toute perte de données. Le basculement planifié des appareils est utilisé dans les scénarios suivants :

  - Simuler des récupérations d’urgence en production lorsque la perte de données n’est pas acceptable
  - Déplacer les bases de données vers une autre région
  - Renvoyer les bases de données vers la région primaire une fois la panne éliminée (restauration automatique).

- **Basculement non planifié**

   Un basculement non planifié ou forcé fait immédiatement basculer la base de données secondaire vers le rôle primaire sans synchronisation avec la base de données primaire. Cette opération entraîne une perte de données. Le basculement non planifié est utilisé comme méthode de récupération pendant les pannes, lorsque la base de données primaire n’est pas accessible. Lorsque la base de données primaire d’origine est de nouveau en ligne, elle se reconnecte sans synchronisation et devient une nouvelle base de données secondaire.

- **Basculement manuel**

  Vous pouvez lancer manuellement le basculement à tout moment, quelle que soit la configuration du basculement automatique. Si la stratégie de basculement automatique n’est pas configurée, un basculement manuel est nécessaire pour récupérer les bases de données dans le groupe de basculement vers le serveur secondaire. Vous pouvez lancer un basculement forcé ou convivial (avec synchronisation complète des données). Ce dernier peut être utilisé pour déplacer le serveur primaire dans la région secondaire. Une fois le basculement terminé, les enregistrements DNS sont automatiquement mis à jour pour garantir la connexion au nouveau serveur principal.

- **Période de grâce avec perte de données**

  Comme les bases de données primaires et secondaires sont synchronisées avec la réplication asynchrone, le basculement peut entraîner une perte de données. Vous pouvez personnaliser la stratégie de basculement automatique en fonction de la tolérance de votre application aux pertes de données. En configurant `GracePeriodWithDataLossHours`, vous pouvez contrôler le délai observé par le système avant d’initialiser le basculement qui est susceptible d’entraîner une perte de données.

- **Plusieurs groupes de basculement**

  Vous pouvez configurer plusieurs groupes de basculement pour la même paire de serveurs afin de contrôler l’étendue des basculements. Chaque groupe bascule indépendamment. Si votre application mutualisée fait appel à des pools élastiques, vous pouvez utiliser cette fonctionnalité pour combiner des bases de données primaires et secondaires dans chaque pool. De cette manière, vous pouvez réduire l’impact d’une panne à la moitié seulement des locataires.

  > [!NOTE]
  > SQL Managed Instance ne prend pas en charge plusieurs groupes de basculement.
  
## <a name="permissions"></a>Autorisations

Les autorisations pour un groupe de basculement sont gérées via un [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md). Le rôle [Contributeur SQL Server](../../role-based-access-control/built-in-roles.md#sql-server-contributor) dispose des autorisations nécessaires pour gérer des groupes de basculement.

### <a name="create-failover-group"></a>Créer un groupe de basculement

Pour créer un groupe de basculement, vous devez disposer d’un accès en écriture Azure RBAC aux serveurs principal et secondaire ainsi qu’à toutes les bases de données du groupe de basculement. Pour une instance SQL Managed Instance, vous devez disposer d’un accès en écriture Azure RBAC aux instances SQL Managed Instance principale et secondaire. Toutefois, les autorisations d’accès aux bases de données individuelles ne sont pas pertinentes dans la mesure où les bases de données SQL Managed Instance individuelles ne peuvent pas être ajoutées ou supprimées d’un groupe de basculement.

### <a name="update-a-failover-group"></a>Mettre à jour un groupe de basculement

Pour mettre à jour un groupe de basculement, vous devez disposer d’un accès en écriture Azure RBAC au groupe de basculement et à toutes les bases de données du serveur principal ou de l’instance gérée actuels.  

### <a name="fail-over-a-failover-group"></a>Faire basculer un groupe de basculement

Pour faire basculer un groupe de basculement, vous devez disposer d’un accès en écriture Azure RBAC au groupe de basculement sur le nouveau serveur principal ou la nouvelle instance gérée.

## <a name="best-practices-for-sql-database"></a>Meilleures pratiques pour SQL Database

Le groupe de basculement automatique doit être configuré sur le serveur primaire, qu’il connectera au serveur secondaire dans une autre région Azure. Les groupes peuvent inclure une partie ou la totalité des bases de données dans ces serveurs. Le diagramme suivant illustre la configuration standard d’une application cloud géoredondante avec plusieurs bases de données et un groupe de basculement automatique.

![Le diagramme montre une configuration standard d’une application cloud géoredondante utilisant plusieurs bases de données et un groupe de basculement automatique.](./media/auto-failover-group-overview/auto-failover-group.png)

> [!NOTE]
> Voir [Ajouter une base de données Azure SQL Database à un groupe de basculement](failover-group-add-single-database-tutorial.md) pour obtenir un didacticiel détaillé sur l’ajout d’une base de données SQL Database à un groupe de basculement.

Quand vous concevez un service en pensant à la continuité d’activité, suivez ces instructions générales :

### <a name="using-one-or-several-failover-groups-to-manage-failover-of-multiple-databases"></a>Utilisation d’un ou plusieurs groupes de basculement pour gérer le basculement de plusieurs bases de données

Un ou plusieurs groupes de basculement peuvent être créés entre deux serveurs situés dans des régions différentes (serveurs principal et serveur secondaire). Chaque groupe peut inclure une ou plusieurs bases de données qui sont récupérées ensemble dans le cas où une partie ou la totalité des bases de données primaires deviennent indisponibles en raison d’une panne dans la région principale. Le groupe de basculement crée une base de données géo-secondaire avec le même objectif de service que la base de données primaire. Si vous ajoutez une relation de géoréplication existante au groupe de basculement, vérifiez que la base de données géosecondaire est configurée avec le même niveau de service et la même taille de calcul que la base de données primaire.
  
> [!IMPORTANT]
> La création de groupes de basculement entre deux serveurs dans différents abonnements n’est actuellement pas prise en charge pour Azure SQL Database. Le fait de déplacer le serveur principal ou secondaire vers un autre abonnement après la création du groupe de basculement peut entraîner des défaillances au niveau des requêtes de basculement et d’autres opérations.

### <a name="using-read-write-listener-for-oltp-workload"></a>Utilisation d’un écouteur en lecture-écriture pour la charge de travail OLTP

Quand vous effectuez des opérations OLTP, utilisez `<fog-name>.database.windows.net`, car l’URL du serveur et les connexions sont automatiquement dirigées vers le serveur principal. L’URL ne change pas après le basculement. Notez que le basculement implique la mise à jour de l’enregistrement DNS de façon à ce que les connexions clients soient redirigées vers le nouveau serveur primaire seulement après l’actualisation du cache DNS.

### <a name="using-read-only-listener-for-read-only-workload"></a>Utilisation d’un écouteur en lecture seule pour une charge de travail en lecture seule

Si vous avez une charge de travail en lecture seule isolée logiquement et qui est tolérante à une certaine obsolescence des données, vous pouvez utiliser la base de données secondaire dans l’application. Pour les sessions en lecture seule, utilisez `<fog-name>.secondary.database.windows.net`, car l’URL du serveur et la connexion sont automatiquement dirigées vers le serveur principal. Il est également recommandé d’indiquer dans la tentative de lecture de la chaîne de connexion à l’aide de `ApplicationIntent=ReadOnly`.

### <a name="preparing-for-performance-degradation"></a>Préparation à une détérioration des performances

Une application Azure classique fait appel à plusieurs services Azure et inclut plusieurs composants. Le basculement automatique du groupe de basculement est déclenché en fonction de l’état des seuls composants Azure SQL. Il peut arriver que les autres services Azure de la région primaire ne soient pas affectés par la panne et que leurs composants soient toujours disponibles dans cette région. Une fois que les bases de données primaires ont basculé vers la région de récupération d’urgence, la latence entre les composants dépendants peut augmenter. Pour éviter les conséquences négatives d’une latence plus élevée sur les performances de l’application, vérifiez que tous les composants de l’application sont redondants dans la région de récupération d’urgence et suivez ces [consignes de sécurité réseau](#failover-groups-and-network-security).

### <a name="preparing-for-data-loss"></a>Préparation à une perte de données

Si une panne est détectée, Azure patiente pendant la période que vous avez spécifiée avec `GracePeriodWithDataLossHours`. La valeur par défaut est 1 heure. Si vous ne pouvez pas vous permettre de perdre des données, veillez à définir dans la commande `GracePeriodWithDataLossHours` un nombre suffisamment grand, par exemple, 24 heures. Utilisez le basculement de groupe manuel pour effectuer une restauration automatique du serveur secondaire au serveur primaire.

> [!IMPORTANT]
> Les pools élastiques disposant de 800 DTU au maximum et de plus de 250 bases de données utilisant la géoréplication peuvent rencontrer des problèmes, notamment des basculements planifiés plus longs et une dégradation des performances.  Ces problèmes sont davantage susceptibles de se produire pour les charges de travail intensives en écriture, quand les points de terminaison de géoréplication sont géographiquement très éloignés, ou quand plusieurs points de terminaison secondaires sont utilisés pour chaque base de données.  Les symptômes de ces problèmes sont signalés quand le décalage de la géoréplication augmente au fil du temps.  Ce décalage peut être surveillé avec [sys.dm_geo_replication_link_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-geo-replication-link-status-azure-sql-database).  Si ces problèmes se produisent, l’atténuation des risques inclut l’augmentation du nombre de DTU du pool ou la réduction du nombre de bases de données géorépliquées dans ce même pool.

### <a name="changing-secondary-region-of-the-failover-group"></a>Changement de la région secondaire du groupe de basculement

Pour illustrer la séquence de changement, nous partons du principe que le serveur A est le serveur principal, que le serveur B est le serveur secondaire existant et que le serveur C est le nouveau serveur secondaire dans la troisième région.  Pour faire la transition, suivez ces étapes :

1. Sur le serveur C, créez des bases de données secondaires supplémentaires de chaque base de données du serveur A en utilisant la [géoréplication active](active-geo-replication-overview.md). Chaque base de données présente sur le serveur A dispose alors de deux bases de données secondaires, une sur le serveur B et l’autre sur le serveur C. Les bases de données primaires restent ainsi protégées durant la transition.
2. Supprimez le groupe de basculement. À ce stade, les connexions échouent. La raison en est que les alias SQL des écouteurs du groupe de basculement ont été supprimés et que la passerelle ne reconnaît pas le nom du groupe de basculement.
3. Recréez le groupe de basculement en utilisant le même nom entre les serveurs A et C. À ce stade, les connexions n’échouent plus.
4. Ajoutez toutes les bases de données primaires du serveur A au nouveau groupe de basculement.
5. Supprimez le serveur B. Toutes les bases de données de B sont alors supprimées automatiquement.

### <a name="changing-primary-region-of-the-failover-group"></a>Changement de la région primaire du groupe de basculement

Pour illustrer la séquence de changement, nous partons du principe que le serveur A est le serveur principal, que le serveur B est le serveur secondaire existant et que le serveur C est le nouveau serveur principal dans la troisième région.  Pour faire la transition, suivez ces étapes :

1. Effectuez un basculement planifié pour faire du serveur B le serveur principal. Le serveur A devient alors le nouveau serveur secondaire. Le basculement peut occasionner un temps d’arrêt de plusieurs minutes. Sa durée effective dépend de la taille du groupe de basculement.
2. Sur le serveur C, créez des bases de données secondaires supplémentaires de chaque base de données du serveur B en utilisant la [géoréplication active](active-geo-replication-overview.md). Chaque base de données présente sur le serveur B dispose alors de deux bases de données secondaires, une sur le serveur A et l’autre sur le serveur C. Les bases de données primaires restent ainsi protégées durant la transition.
3. Supprimez le groupe de basculement. À ce stade, les connexions échouent. La raison en est que les alias SQL des écouteurs du groupe de basculement ont été supprimés et que la passerelle ne reconnaît pas le nom du groupe de basculement.
4. Recréez le groupe de basculement en utilisant le même nom entre les serveurs B et C. À ce stade, les connexions n’échouent plus.
5. Ajoutez toutes les bases de données primaires du serveur B au nouveau groupe de basculement.
6. Procédez à un basculement planifié du groupe de basculement pour interchanger B et C. Le serveur C devient alors le serveur principal et B le serveur secondaire. Toutes les bases de données secondaires du serveur A sont automatiquement liées aux bases de données primaires de C. Comme à l’étape 1, le basculement peut occasionner un temps d’arrêt de plusieurs minutes.
7. Supprimez le serveur A. Toutes les bases de données de A sont alors supprimées automatiquement.

> [!IMPORTANT]
> Une fois le groupe de basculement supprimé, les enregistrements DNS des points de terminaison de l’écouteur sont également supprimés. À ce stade, il n’est pas entièrement exclu que quelqu’un d’autre crée un groupe de basculement ou un alias de serveur sous le même nom, ce qui vous empêchera de l’utiliser à nouveau. Pour limiter ce risque, évitez d’utiliser des noms de groupe de basculement génériques.

## <a name="best-practices-for-sql-managed-instance"></a>Meilleures pratiques pour SQL Managed Instance

Le groupe de basculement automatique doit être configuré sur l’instance primaire, qu’il connectera à l’instance secondaire dans une autre région Azure.  Toutes les bases de données de l’instance seront répliquées sur l’instance secondaire.

Le diagramme suivant illustre la configuration standard d’une application cloud géoredondante avec une instance managée et un groupe de basculement automatique.

![Diagramme du basculement automatique](./media/auto-failover-group-overview/auto-failover-group-mi.png)

> [!NOTE]
> Voir [Ajouter une instance gérée à un groupe de basculement](../managed-instance/failover-group-add-instance-tutorial.md) pour obtenir un didacticiel détaillé sur l’ajout d’une instance SQL Managed Instance afin d’utiliser un groupe de basculement.

Si votre application utilise une instance SQL Managed Instance en tant que couche Données, suivez ces recommandations générales lors de la conception des éléments en rapport avec la continuité d’activité :

### <a name="creating-the-secondary-instance"></a>Création de l’instance secondaire

Pour garantir une connectivité ininterrompue à l’instance SQL Managed Instance principale après le basculement, les instances principale et secondaire doivent se trouver dans la même zone DNS. Cela garantit que le même certificat multidomaine (SAN) peut être utilisé pour authentifier les connexions clientes avec les deux instances du groupe de basculement. Lorsque votre application est prête pour le déploiement en production, créez une instance SQL Managed Instance secondaire dans une autre région et assurez-vous qu’elle partage la même zone DNS que l’instance SQL Managed Instance principale. Pour cela, vous pouvez spécifier un paramètre `DNS Zone Partner` facultatif à l’aide du Portail Azure, de PowerShell ou de l’API REST.

> [!IMPORTANT]
> La première instance gérée créée dans le sous-réseau détermine la zone DNS pour toutes les instances suivantes de ce même sous-réseau. Cela signifie que deux instances d'un même sous-réseau ne peuvent pas appartenir à des zones DNS différentes.

Pour plus d’informations sur la création de l’instance SQL Managed Instance secondaire dans la même zone DNS que l’instance principale, voir [Créer une instance managée secondaire](../managed-instance/failover-group-add-instance-tutorial.md#create-a-secondary-managed-instance).

### <a name="using-geo-paired-regions"></a>Utilisation de régions jumelées géographiquement

Déployez les deux instances managées dans des [régions jumelées](../../best-practices-availability-paired-regions.md) pour des raisons de performances. Les instances managées résidant dans des régions jumelées géographiquement offrent des performances nettement meilleures que celles résidant dans des régions non jumelées. 

### <a name="enabling-replication-traffic-between-two-instances"></a>Activation du trafic de réplication entre deux instances

Étant donné que chaque instance est isolée dans son propre réseau virtuel, le trafic bidirectionnel entre ces réseaux virtuels doit être autorisé. Voir [Passerelle VPN Azure](../../vpn-gateway/vpn-gateway-about-vpngateways.md)

### <a name="creating-a-failover-group-between-managed-instances-in-different-subscriptions"></a>Création d’un groupe de basculement entre des instances gérées d’abonnements différents

Vous pouvez créer un groupe de basculement entre des instances managées SQL dans deux abonnements différents, à condition que les abonnements soient associés au même [abonné Azure Active Directory](../../active-directory/fundamentals/active-directory-whatis.md#terminology). Lorsque vous utilisez l’API PowerShell, vous pouvez le faire en spécifiant le paramètre `PartnerSubscriptionId` pour l’instance SQL Managed Instance secondaire. Lors de l’utilisation de l’API REST, chaque ID d’instance inclus dans le paramètre `properties.managedInstancePairs` peut avoir son propre subscriptionID.
  
> [!IMPORTANT]
> Le Portail Azure ne prend pas en charge la création de groupes de basculement sur différents abonnements. De plus, pour les groupes de basculement existants sur différents abonnements et/ou groupes de ressources, le basculement ne peut pas être initié manuellement à l’aide du portail à partir de l’instance SQL Managed Instance principale. Initiez-le plutôt à partir de l’instance géosecondaire.

### <a name="managing-failover-to-secondary-instance"></a>Gestion du basculement vers une instance secondaire

Le groupe de basculement gère le basculement de toutes les bases de données dans l’instance SQL Managed Instance. Lorsqu’un groupe est créé, chaque base de données dans l’instance est automatiquement géorépliquée sur l’instance SQL Managed Instance secondaire. Vous ne pouvez pas utiliser les groupes de basculement pour initier le basculement partiel d’un sous-ensemble de bases de données.

> [!IMPORTANT]
> Si une base de données est supprimée de l’instance SQL Managed Instance principale, elle est également abandonnée automatiquement sur l’instance SQL Managed Instance géorépliquée secondaire.

### <a name="using-read-write-listener-for-oltp-workload"></a>Utilisation d’un écouteur en lecture-écriture pour la charge de travail OLTP

Quand vous effectuez des opérations OLTP, utilisez `<fog-name>.zone_id.database.windows.net`, car l’URL du serveur et les connexions sont automatiquement dirigées vers le serveur principal. L’URL ne change pas après le basculement. Le basculement implique la mise à jour de l’enregistrement DNS de façon à ce que les connexions clients soient redirigées vers le nouveau serveur primaire seulement après l’actualisation du cache DNS. Étant donné que l’instance secondaire partage la même zone DNS que l’instance principale, l’application cliente peut se reconnecter à l’aide du même certificat SAN.

### <a name="using-read-only-listener-to-connect-to-the-secondary-instance"></a>Utilisation d’un écouteur en lecture seule pour se connecter à l’instance secondaire

Si vous avez une charge de travail en lecture seule isolée logiquement et qui est tolérante à une certaine obsolescence des données, vous pouvez utiliser la base de données secondaire dans l’application. Pour vous connecter directement à l’instance géorépliquée secondaire, utilisez `<fog-name>.secondary.<zone_id>.database.windows.net`, car l’URL du serveur et la connexion sont automatiquement dirigées vers l’instance géorépliquée secondaire.

> [!NOTE]
> Dans les niveaux de service Premium, Critique pour l’entreprise et Hyperscale, Azure SQL Database prend en charge l’utilisation de [réplicas en lecture seule](read-scale-out.md) pour exécuter des charges de travail de requête en lecture seule, en utilisant la capacité d’un ou de plusieurs réplicas en lecture seule et le paramètre `ApplicationIntent=ReadOnly` dans la chaîne de connexion. Lorsque vous avez configuré une instance géorépliquée secondaire, vous pouvez utiliser cette fonction pour vous connecter à un réplica en lecture seule à l’emplacement primaire ou à l’emplacement géorépliqué.
>
> - Pour vous connecter à un réplica en lecture seule à l’emplacement primaire, utilisez `ApplicationIntent=ReadOnly` et `<fog-name>.<zone_id>.database.windows.net`.
> - Pour vous connecter à un réplica en lecture seule à l’emplacement secondaire, utilisez `ApplicationIntent=ReadOnly` et `<fog-name>.secondary.<zone_id>.database.windows.net`.

### <a name="preparing-for-performance-degradation"></a>Préparation à une détérioration des performances

Une application Azure classique fait appel à plusieurs services Azure et inclut plusieurs composants. Le basculement automatique du groupe de basculement est déclenché en fonction de l’état des seuls composants Azure SQL. Il peut arriver que les autres services Azure de la région primaire ne soient pas affectés par la panne et que leurs composants soient toujours disponibles dans cette région. Une fois que les bases de données primaires ont basculé vers la région secondaire, la latence entre les composants dépendants peut augmenter. Pour éviter les conséquences négatives d’une latence plus élevée sur les performances de l’application, vérifiez que tous les composants de l’application sont redondants dans la région secondaire et basculez les composants de l’application en même temps que la base de données. Au moment de la configuration, suivez les [instructions de sécurité réseau](#failover-groups-and-network-security) pour garantir la connectivité à la base de données dans la région secondaire.

### <a name="preparing-for-data-loss"></a>Préparation à une perte de données

Si une panne est détectée, un basculement en lecture-écriture est déclenché dans le cas où il n’y a à notre connaissance pas de perte de données. Dans le cas contraire, le basculement est différé pendant la période que vous spécifiez à l’aide du paramètre `GracePeriodWithDataLossHours`. Si vous avez spécifié `GracePeriodWithDataLossHours`, attendez-vous à une perte de données. En général, en cas de panne, Azure favorise la disponibilité. Si vous ne pouvez pas vous permettre de perdre des données, veillez à définir dans le paramètre GracePeriodWithDataLossHours un nombre suffisamment grand (par exemple, 24 heures), ou désactivez le basculement automatique.

La mise à jour DNS de l’écouteur en lecture-écriture se produit immédiatement après que le basculement est initié. Cette opération n’entraîne aucune perte de données. Toutefois, le processus de basculement des rôles des bases de données peut prendre jusqu’à 5 minutes dans des conditions normales. En attendant, certaines bases de données de la nouvelle instance principale resteront en lecture seule. Si un basculement est initié à l’aide de PowerShell, l’opération permettant de basculer le rôle de réplica principal est synchrone. S’il est initié à l’aide du portail Azure, l’interface utilisateur indiquera la progression. S’il est démarré à l’aide de l’API REST, utilisez le mécanisme d’interrogation standard d’Azure Resource Manager pour en surveiller la progression.

> [!IMPORTANT]
> Utilisez le basculement de groupe manuel pour redéplacer les bases de données primaires à leur emplacement d’origine. Lorsque la panne ayant provoqué le basculement est résolue, vous pouvez déplacer vos bases de données primaires vers leur emplacement d’origine. Pour ce faire, vous devez effectuer le basculement manuel du groupe.
  
### <a name="changing-secondary-region-of-the-failover-group"></a>Changement de la région secondaire du groupe de basculement

Supposons que l’instance A est l’instance principale, que l’instance B est l’instance secondaire existante et que l’instance C est la nouvelle instance secondaire dans la troisième région.  Pour faire la transition, suivez ces étapes :

1. Créez l’instance C avec la même taille que A et dans la même zone DNS.
2. Supprimez le groupe de basculement entre les instances A et B. À ce stade, les connexions échouent, car les alias SQL des écouteurs du groupe de basculement ont été supprimés et la passerelle ne reconnaît pas le nom du groupe de basculement. Les bases de données secondaires sont déconnectées des bases de données primaires et deviennent des bases de données en lecture-écriture.
3. Créez un groupe de basculement avec le même nom entre l’instance A et l’instance C. Suivez les instructions du [didacticiel relatif au groupe de basculement avec SQL Managed Instance](../managed-instance/failover-group-add-instance-tutorial.md). Il s’agit d’une opération de taille de données qui se termine dès lors que toutes les bases de données de l’instance A ont été amorcées et synchronisées.
4. Si vous n’en avez plus besoin, supprimez l’instance B pour éviter des frais inutiles.

> [!NOTE]
> Après l’étape 2 et tant que l’étape 3 n’est par terminée, les bases de données de l’instance A restent non protégées contre une défaillance irrémédiable de l’instance A.

### <a name="changing-primary-region-of-the-failover-group"></a>Changement de la région primaire du groupe de basculement

Supposons que l’instance A est l’instance principale, que l’instance B est l’instance secondaire existante et que l’instance C est la nouvelle instance principale dans la troisième région.  Pour faire la transition, suivez ces étapes :

1. Créez l’instance C avec la même taille que B et dans la même zone DNS.
2. Connectez-vous à l’instance B et basculez manuellement pour passer de l’instance principale à l’instance B. L’instance A devient automatiquement la nouvelle instance secondaire.
3. Supprimez le groupe de basculement entre les instances A et B. À ce stade, les connexions échouent, car les alias SQL des écouteurs du groupe de basculement ont été supprimés et la passerelle ne reconnaît pas le nom du groupe de basculement. Les bases de données secondaires sont déconnectées des bases de données primaires et deviennent des bases de données en lecture-écriture.
4. Créez un groupe de basculement avec le même nom entre l’instance A et l’instance C. Suivez les instructions du [tutoriel Groupe de basculement avec instance gérée](../managed-instance/failover-group-add-instance-tutorial.md). Il s’agit d’une opération de taille de données qui se termine dès lors que toutes les bases de données de l’instance A ont été amorcées et synchronisées.
5. Si vous n’en avez plus besoin, supprimez l’instance A pour éviter des frais inutiles.

> [!CAUTION]
> Après l’étape 3 et tant que l’étape 4 n’est par terminée, les bases de données de l’instance A restent non protégées contre une défaillance irrémédiable de l’instance A.

> [!IMPORTANT]
> Une fois le groupe de basculement supprimé, les enregistrements DNS des points de terminaison de l’écouteur sont également supprimés. À ce stade, il n’est pas entièrement exclu que quelqu’un d’autre crée un groupe de basculement ou un alias de serveur sous le même nom, ce qui vous empêchera de l’utiliser à nouveau. Pour limiter ce risque, évitez d’utiliser des noms de groupe de basculement génériques.

### <a name="enable-scenarios-dependent-on-objects-from-the-system-databases"></a>Activer les scénarios dépendant des objets des bases de données système
Les bases de données système ne sont pas répliquées vers l’instance secondaire dans un groupe de basculement. Pour activer les scénarios dépendant d’objets des bases de données système, assurez-vous de créer ces objets sur l’instance secondaire. Par exemple, si vous envisagez d’utiliser les mêmes connexions sur l’instance secondaire, veillez à les créer avec un ID de sécurité identique. 
```SQL
-- Code to create login on the secondary instance
CREATE LOGIN foo WITH PASSWORD = '<enterStrongPasswordHere>', SID = <login_sid>;
``` 


## <a name="failover-groups-and-network-security"></a>Groupes de basculement et sécurité réseau

Pour certaines applications, les règles de sécurité nécessitent que l’accès réseau à la couche Données soit limité à un ou plusieurs composants comme une machine virtuelle, un service web, etc. Cette exigence présente quelques défis pour la conception de la continuité d’activité et l’utilisation des groupes de basculement. Tenez compte des options suivantes lors de l’implémentation d’un accès restreint.

### <a name="using-failover-groups-and-virtual-network-rules"></a>Utilisation de groupes de basculement et de règles de réseau virtuel

Si vous utilisez des [règles et points de terminaison de service de réseau virtuel](vnet-service-endpoint-rule-overview.md) pour restreindre l’accès à votre base de données dans SQL Database ou SQL Managed Instance, n’oubliez pas que chaque point de terminaison de service de réseau virtuel s’applique à une seule région Azure. Le point de terminaison ne permet pas à d’autres régions d’accepter les communications provenant du sous-réseau. Par conséquent, seules les applications client déployées dans la même région peuvent se connecter à la base de données primaire. Comme le basculement entraîne la redirection des sessions du client SQL Database vers un serveur dans une autre région (secondaire), ces sessions échouent si elles proviennent d’un client situé en dehors de cette région. Pour cette raison, la stratégie de basculement automatique ne peut pas être activée si les instances ou serveurs participants sont inclus dans les règles de réseau virtuel. Pour prendre en charge le basculement manuel, effectuez les étapes suivantes :

1. Provisionnez les copies redondantes des composants front-end de votre application (service web, machines virtuelles, etc.) dans la région secondaire
2. Configurez les [règles de réseau virtuel](vnet-service-endpoint-rule-overview.md) individuellement pour les serveurs primaire et secondaire
3. Activez le [basculement frontend à l’aide d’une configuration Traffic Manager](designing-cloud-solutions-for-disaster-recovery.md#scenario-1-using-two-azure-regions-for-business-continuity-with-minimal-downtime)
4. Lancez un basculement manuel quand la panne est détectée. Cette option est optimisée pour les applications qui requièrent une latence constante entre le frontend et la couche Données, et prend en charge la récupération quand le frontend et/ou la couche Données sont affectés par la panne.

> [!NOTE]
> Si vous utilisez **l’écouteur en lecture seule** pour équilibrer une charge de travail en lecture seule, vérifiez que cette charge de travail est exécutée dans une machine virtuelle ou une autre ressource dans la région secondaire pour qu’elle puisse se connecter à la base de données secondaire.

### <a name="use-failover-groups-and-firewall-rules"></a>Utiliser des groupes de basculement et des règles de pare-feu

Si votre plan de continuité d’activité nécessite un basculement à l’aide de groupes avec basculement automatique, vous pouvez restreindre l’accès à votre base de données dans SQL Database, à l’aide des règles de pare-feu classiques. Pour prendre en charge le basculement automatique, effectuez les étapes suivantes :

1. [Créez une adresse IP publique](../../virtual-network/virtual-network-public-ip-address.md#create-a-public-ip-address).
2. [Créez un équilibreur de charge public](../../load-balancer/quickstart-load-balancer-standard-public-portal.md) et affectez-lui l’adresse IP publique.
3. [Créez un réseau virtuel et les machines virtuelles](../../load-balancer/quickstart-load-balancer-standard-public-portal.md) pour vos composants frontend.
4. [Créez un groupe de sécurité réseau](../../virtual-network/network-security-groups-overview.md) et configurez les connexions entrantes.
5. Vérifiez que les connexions sortantes sont ouvertes pour Azure SQL Database à l’aide de la [balise de service](../../virtual-network/network-security-groups-overview.md#service-tags) « Sql ».
6. Créez une [règle de pare-feu SQL Database](firewall-configure.md) pour autoriser le trafic entrant à partir de l’adresse IP publique que vous créez à l’étape 1.

Pour plus d’informations sur la configuration de l’accès sortant et l’adresse IP à utiliser dans les règles de pare-feu, voir [Connexions sortantes de l’équilibreur de charge](../../load-balancer/load-balancer-outbound-connections.md).

La configuration ci-dessus garantit que le basculement automatique ne bloque pas les connexions à partir des composants frontend et part du principe que l’application peut tolérer la plus longue latence entre le frontend et la couche Données.

> [!IMPORTANT]
> Pour garantir la continuité d’activité en cas de pannes régionales, vous devez vérifier la redondance géographique pour les composants frontend et les bases de données.

## <a name="enabling-geo-replication-between-managed-instances-and-their-vnets"></a>Activation de la géoréplication entre les instances gérées et leurs réseaux virtuels

Lorsque vous configurez un groupe de basculement entre les instances SQL Managed Instance primaire et secondaire dans deux régions différentes, chaque instance est isolée et utilise un réseau virtuel indépendant. Pour autoriser le trafic de réplication entre ces réseaux virtuels, vérifiez que les conditions préalables suivantes sont satisfaites :

- Les deux instances SQL Managed Instance doivent se trouver dans différentes régions Azure.
- Ces deux instances doivent avoir le même niveau de service et la même capacité de stockage.
- Votre instance SQL Managed Instance secondaire doit être vide (aucune base de données utilisateur).
- Les réseaux virtuels utilisés par les instances SQL Managed Instance doivent être connectés via une [passerelle VPN](../../vpn-gateway/vpn-gateway-about-vpngateways.md) ou [Express Route](../../expressroute/expressroute-howto-circuit-portal-resource-manager.md). Lorsque deux réseaux virtuels se connectent via un réseau local, assurez-vous qu’il n’existe pas de ports de blocage de règle de pare-feu 5022 et 11000-11999. L’appairage de réseaux virtuels mondiaux est pris en charge avec la limitation décrite dans la note ci-dessous.

   > [!IMPORTANT]
   > [Le 22/09/2020, nous avons annoncé l’appairage de réseaux virtuels mondiaux pour les clusters virtuels nouvellement créés](https://azure.microsoft.com/en-us/updates/global-virtual-network-peering-support-for-azure-sql-managed-instance-now-available/). Cela signifie que l’appairage de réseaux virtuels mondiaux est pris en charge pour les instances managées SQL créées dans des sous-réseaux vides après la date d’annonce, ainsi que pour toutes les instances managées ultérieures, créées dans ces sous-réseaux. Pour toutes les autres instances managées SQL, la prise en charge de l’appairage est limitée aux réseaux de la même région en raison des [contraintes de l’appairage de réseaux virtuels mondiaux](../../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints). Consultez également la section appropriée de l’article [Forum Aux Questions sur les réseaux virtuel Azure](../../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers) pour plus d’informations. 

- Les adresses IP des deux réseaux virtuels SQL Managed Instance ne peuvent pas se chevaucher.
- Vous devez configurer vos groupes de sécurité réseau (NSG) de telle sorte que les ports 5022 et la plage 11000 à 12000 soient ouverts en entrée et en sortie pour les connexions provenant du sous-réseau de l’autre instance gérée. Ceci est destiné à autoriser le trafic de réplication entre les instances.

   > [!IMPORTANT]
   > La configuration incorrecte des règles de groupes de sécurité réseau bloque les opérations de copie sur les bases de données.

- L’instance SQL Managed Instance secondaire est configurée avec l’ID de zone DNS approprié. La zone DNS est une propriété d’une instance SQL Managed Instance et d’un cluster virtuel sous-jacent, et son ID est inclus dans l’adresse du nom d’hôte. L’ID de zone est généré sous forme de chaîne aléatoire lors de la création de la première instance SQL Managed Instance de chaque réseau virtuel. De plus, le même ID est attribué à toutes les autres instances dans le même sous-réseau. Une fois attribué, la zone DNS ne peut pas être modifiée. Les instances SQL Managed Instance d’un même groupe de basculement doivent partager la zone DNS. Pour cela, vous devez transmettre l’ID de zone de l’instance principale en tant que valeur du paramètre DnsZonePartner lors de la création de l’instance secondaire.

   > [!NOTE]
   > Pour accéder à un didacticiel détaillé sur la configuration des groupes de basculement avec SQL Managed Instance, voir [Add a SQL Managed Instance to a failover group](../managed-instance/failover-group-add-instance-tutorial.md) (Ajouter une instance SQL Managed Instance à un groupe de basculement).

## <a name="upgrading-or-downgrading-a-primary-database"></a>Mise à niveau ou rétrogradation d’une base de données primaire

Vous pouvez augmenter ou diminuer la taille de calcul d’une base de données primaire (au sein du même niveau de service, mais pas entre les niveaux Usage général et Critique pour l’entreprise) sans déconnecter les bases de données secondaires. Lors d’une mise à niveau, nous vous recommandons de mettre à niveau toutes les bases de données secondaires dans un premier temps, avant de mettre à niveau la base de données primaire. Lors du passage à une version antérieure, inversez l’ordre : faites tout d’abord passer la base de données primaire à une version antérieure, puis toutes les bases de données secondaires dans un second temps. Lorsque vous passez la base de données à un niveau de service supérieur ou inférieur, cette recommandation est appliquée.

Cette séquence est recommandée dans le but spécifique d’éviter le problème de surcharge des bases de données secondaires avec une référence SKU inférieure. Celles-ci doivent alors être alimentées à nouveau lors de la mise à niveau ou du passage à une version antérieure. Vous pouvez également éviter le problème en attribuant un accès en lecture seule à la base de données primaire, ce qui peut cependant nuire à toutes les charges de travail en lecture-écriture qui y sont associées.

> [!NOTE]
> Si vous avez créé une base de données secondaire dans le cadre de la configuration des groupes de basculement, il n’est pas conseillé de faire passer cette base de données à une version antérieure. En effet, votre couche Données pourrait manquer de capacité pour traiter votre charge de travail normale après l’activation du basculement.

## <a name="preventing-the-loss-of-critical-data"></a>Prévention de la perte de données critiques

En raison de la latence élevée des réseaux étendus, la copie continue utilise un mécanisme de réplication asynchrone. La perte de certaines données reste donc inévitable en cas de défaillance. Or, pour certaines applications, une perte de données est inacceptable. Pour protéger ces mises à jour critiques, un développeur d’applications peut appeler la procédure système [sp_wait_for_database_copy_sync](/sql/relational-databases/system-stored-procedures/active-geo-replication-sp-wait-for-database-copy-sync) immédiatement après la validation de la transaction. L’appel de `sp_wait_for_database_copy_sync` bloque le thread appelant jusqu’à ce que la dernière transaction validée ait été transmise à la base de données secondaire. Toutefois, il n’attend pas que les transactions transmises soient relues et validées sur la base de données secondaire. `sp_wait_for_database_copy_sync` est limité à un lien de copie continue spécifique. Tout utilisateur disposant de droits de connexion à la base de données primaire peut appeler cette procédure.

> [!NOTE]
> `sp_wait_for_database_copy_sync` empêche la perte de données après un basculement, mais il ne garantit pas la synchronisation complète pour l’accès en lecture. Le délai causé par un appel de procédure `sp_wait_for_database_copy_sync` peut être significatif et dépend de la taille du journal des transactions au moment de l’appel.

## <a name="failover-groups-and-point-in-time-restore"></a>Groupes de basculement et limite de restauration dans le temps

Pour plus d’informations sur l’utilisation de la limite de restauration dans le temps avec les groupes de basculement, voir [Limite de restauration dans le temps](recovery-using-backups.md#point-in-time-restore).

## <a name="limitations-of-failover-groups"></a>Limitations des groupes de basculement

Notez les limitations suivantes :

- Il n’est pas possible de créer des groupes de basculement entre deux serveurs ou instances au sein des mêmes régions Azure.
- Les groupes de basculement ne peuvent pas être renommés. Vous devrez supprimer le groupe puis le recréer sous un autre nom.
- Le renommage d’une base de données n’est pas pris en charge pour les instances situées dans un groupe de basculement. Vous devez supprimer temporairement le groupe de basculement pour pouvoir renommer une base de données.
- Les bases de données système ne sont pas répliquées vers l’instance secondaire dans un groupe de basculement. Par conséquent, les scénarios qui dépendent des objets des bases de données système ne peuvent pas être appliqués sur l’instance secondaire, à moins que ces objets ne soient créés manuellement sur cette dernière.

## <a name="programmatically-managing-failover-groups"></a>Gestion par programmation des groupes de basculement

Comme indiqué plus haut, les groupes de basculement automatique et la géo-réplication active peuvent aussi être gérés par programme à l’aide d’Azure PowerShell et de l’API REST. Les tableaux ci-dessous décrivent l’ensemble des commandes disponibles. La géoréplication active comprend un ensemble d’API Azure Resource Manager pour la gestion, notamment [l’API REST Azure SQL Database](/rest/api/sql/) et les [applets de commande Azure PowerShell](/powershell/azure/). Ces API nécessitent l’utilisation de groupes de ressources et la prise en charge du contrôle d’accès en fonction du rôle (RBAC). Pour plus d’informations sur l’implémentation de rôles d’accès, consultez la page sur le [contrôle d’accès en fonction du rôle Azure (RBAC Azure)](../../role-based-access-control/overview.md).

### <a name="manage-sql-database-failover"></a>Gérer un basculement SQL Database

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

| Applet de commande | Description |
| --- | --- |
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) |Cette commande crée un groupe de basculement et l’enregistre dans les serveurs primaire et secondaire|
| [Remove-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/remove-azsqldatabasefailovergroup) | Supprime un groupe de basculement du serveur |
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Récupère la configuration d’un groupe de basculement |
| [Set-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/set-azsqldatabasefailovergroup) |Modifie la configuration d’un groupe de basculement |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup) | Déclenche le basculement d’un groupe de basculement vers le serveur secondaire |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup)|Ajoute une ou plusieurs bases de données à un groupe de basculement|

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

| Commande | Description |
| --- | --- |
| [az sql failover-group create](/cli/azure/sql/failover-group#az-sql-failover-group-create) |Cette commande crée un groupe de basculement et l’enregistre dans les serveurs primaire et secondaire|
| [az sql failover-group delete](/cli/azure/sql/failover-group#az-sql-failover-group-delete) | Supprime un groupe de basculement du serveur |
| [az sql failover-group show](/cli/azure/sql/failover-group#az-sql-failover-group-show) | Récupère la configuration d’un groupe de basculement |
| [az sql failover-group update](/cli/azure/sql/failover-group#az-sql-failover-group-update) |Modifie la configuration d’un groupe de basculement et/ou ajoute une ou plusieurs bases de données à un groupe de basculement|
| [az sql failover-group set-primary](/cli/azure/sql/failover-group#az-sql-failover-group-set-primary) | Déclenche le basculement d’un groupe de basculement vers le serveur secondaire |

# <a name="rest-api"></a>[API REST](#tab/rest-api)

| API | Description |
| --- | --- |
| [Créer ou mettre à jour un groupe de basculement](/rest/api/sql/failovergroups/createorupdate) | Crée ou met à jour un groupe de basculement |
| [Supprimer un groupe de basculement](/rest/api/sql/failovergroups/delete) | Supprime un groupe de basculement du serveur |
| [Basculement (planifié)](/rest/api/sql/failovergroups/failover) | Déclenche le basculement du serveur primaire actuel vers le serveur secondaire avec une synchronisation complète des données.|
| [Forcer le basculement et autoriser la perte de données](/rest/api/sql/failovergroups/forcefailoverallowdataloss) | Déclenche le basculement du serveur primaire actuel vers le serveur secondaire sans synchroniser les données. Cette opération peut occasionner une perte de données. |
| [Get Failover Group](/rest/api/sql/failovergroups/get) (Obtenir un groupe de basculement) | Récupère la configuration d’un groupe de basculement. |
| [Répertorier les groupes de basculement par serveur](/rest/api/sql/failovergroups/listbyserver) | Liste les groupes de basculement d’un serveur. |
| [Mettre à jour un groupe de basculement](/rest/api/sql/failovergroups/update) | Met à jour la configuration d’un groupe de basculement. |

---

### <a name="manage-sql-managed-instance-failover"></a>Gérer un basculement SQL Managed Instance


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

| Applet de commande | Description |
| --- | --- |
| [New-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/new-azsqldatabaseinstancefailovergroup) |Cette commande crée un groupe de basculement et l’enregistre dans les instances principale et secondaire|
| [Set-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/set-azsqldatabaseinstancefailovergroup) |Modifie la configuration d’un groupe de basculement|
| [Get-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/get-azsqldatabaseinstancefailovergroup) |Récupère la configuration d’un groupe de basculement|
| [Switch-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/switch-azsqldatabaseinstancefailovergroup) |Déclenche le basculement d’un groupe de basculement vers l’instance secondaire|
| [Remove-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/remove-azsqldatabaseinstancefailovergroup) | Supprime un groupe de basculement|


# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

| Commande | Description |
| --- | --- |
| [az sql failover-group create](/cli/azure/sql/failover-group#az-sql-failover-group-create) |Cette commande crée un groupe de basculement et l’enregistre dans les serveurs primaire et secondaire|
| [az sql failover-group delete](/cli/azure/sql/failover-group#az-sql-failover-group-delete) | Supprime un groupe de basculement du serveur |
| [az sql failover-group show](/cli/azure/sql/failover-group#az-sql-failover-group-show) | Récupère la configuration d’un groupe de basculement |
| [az sql failover-group update](/cli/azure/sql/failover-group#az-sql-failover-group-update) |Modifie la configuration d’un groupe de basculement et/ou ajoute une ou plusieurs bases de données à un groupe de basculement|
| [az sql failover-group set-primary](/cli/azure/sql/failover-group#az-sql-failover-group-set-primary) | Déclenche le basculement d’un groupe de basculement vers le serveur secondaire |

# <a name="rest-api"></a>[API REST](#tab/rest-api)

| API | Description |
| --- | --- |
| [Créer ou mettre à jour un groupe de basculement](/rest/api/sql/instancefailovergroups/createorupdate) | Crée ou met à jour la configuration d’un groupe de basculement |
| [Supprimer un groupe de basculement](/rest/api/sql/instancefailovergroups/delete) | Supprime un groupe de basculement de l’instance |
| [Basculement (planifié)](/rest/api/sql/instancefailovergroups/failover) | Déclenche le basculement de l’instance principale actuelle vers cette instance avec une synchronisation complète des données. |
| [Forcer le basculement et autoriser la perte de données](/rest/api/sql/instancefailovergroups/forcefailoverallowdataloss) | Déclenche le basculement de l’instance principale actuelle vers l’instance secondaire sans synchroniser les données. Cette opération peut occasionner une perte de données. |
| [Get Failover Group](/rest/api/sql/instancefailovergroups/get) (Obtenir un groupe de basculement) | Récupère la configuration d’un groupe de basculement. |
| [List Failover Groups - List By Location](/rest/api/sql/instancefailovergroups/listbylocation) (Répertorier les groupes de basculement - Liste par emplacement) | Répertorie les groupes de basculement d’un emplacement. |

---

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir des didacticiels détaillés, consultez
  - [Add SQL Database to a failover group](failover-group-add-single-database-tutorial.md) (Ajouter une base de données Azure SQL Database à un groupe de basculement)
  - [Ajouter un pool élastique à un groupe de basculement](failover-group-add-elastic-pool-tutorial.md)
  - [Add a SQL Managed Instance to a failover group](../managed-instance/failover-group-add-instance-tutorial.md) (Ajouter une instance SQL Managed Instance à un groupe de basculement)
- Pour obtenir des exemples de scripts, consultez :
  - [Use PowerShell to configure active geo-replication for Azure SQL Database](scripts/setup-geodr-and-failover-database-powershell.md) (Utiliser PowerShell pour configurer la géoréplication active pour Azure SQL Database)
  - [Utiliser PowerShell afin de configurer la géoréplication active pour avoir une base de données mise en pool dans Azure SQL Database](scripts/setup-geodr-and-failover-elastic-pool-powershell.md)
  - [Use PowerShell to add an Azure SQL Database to a failover group](scripts/add-database-to-failover-group-powershell.md) (Utiliser PowerShell afin d’ajouter une base de données Azure SQL Database à un groupe de basculement)
- Pour une vue d’ensemble de la continuité des activités et des scénarios, consultez [Vue d’ensemble de la continuité des activités](business-continuity-high-availability-disaster-recover-hadr-overview.md)
- Pour en savoir plus sur les sauvegardes automatisées d’une base de données Azure SQL, consultez [Sauvegardes automatisées d’une base de données SQL](automated-backups-overview.md).
- Pour en savoir plus sur l’utilisation des sauvegardes automatisées pour la récupération, consultez [Restaurer une base de données à partir des sauvegardes initiées par le service](recovery-using-backups.md).
- Pour en savoir plus sur les exigences d’authentification pour de nouveaux serveurs et bases de données primaires, consultez [Gestion de la sécurité de la base de données SQL Azure après la récupération d’urgence](active-geo-replication-security-configure.md).
