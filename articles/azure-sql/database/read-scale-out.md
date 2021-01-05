---
title: Lire des requêtes sur des réplicas
description: Azure SQL permet d'utiliser des réplicas en lecture seule pour les charges de travail en lecture. C'est ce qu'on appelle l'échelle horizontale en lecture.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: sstein
ms.date: 09/03/2020
ms.openlocfilehash: 9c09a54daa482d738ded9f7aca1c95c2b640617e
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790268"
---
# <a name="use-read-only-replicas-to-offload-read-only-query-workloads"></a>Utiliser des réplicas en lecture seule pour décharger des charges de travail de requêtes en lecture seule
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Dans le cadre d’une [architecture à haute disponibilité](high-availability-sla.md#premium-and-business-critical-service-tier-locally-redundant-availability), chaque base de données, chaque base de données du pool élastique et chaque instance managée des niveaux de service Premium et Critique pour l’entreprise sont automatiquement provisionnées avec un réplica principal en lecture-écriture et plusieurs réplicas secondaires en lecture seule. Les réplicas secondaires sont approvisionnés avec la même taille de calcul que le réplica principal. La fonctionnalité *Échelle horizontale en lecture* vous permet de décharger les charges de travail en lecture seule à l'aide de la capacité de calcul de l'un des réplicas en lecture seule au lieu de les exécuter sur le réplica en lecture-écriture. De cette façon, certaines charges de travail en lecture seule peuvent être isolées des charges de travail en lecture-écriture et n'affecteront pas leurs performances. Cette fonctionnalité est destinée aux applications qui incluent des charges de travail en lecture seule séparées logiquement, telles que des analyses. Aux niveaux de service Premium et Critique pour l’entreprise, les applications peuvent bénéficier d’avantages en matière de performances en exploitant cette capacité supplémentaire sans coût supplémentaire.

La fonctionnalité *Échelle horizontale en lecture* est également disponible au niveau de service Hyperscale lorsqu'au moins un réplica secondaire est créé. Plusieurs réplicas secondaires peuvent être utilisés pour équilibrer les charges de travail en lecture seule qui nécessitent plus de ressources qu'il n'en existe sur un réplica secondaire.

L’architecture à haute disponibilité des niveaux de service De base, Standard et Usage général n’inclut pas de réplica. La fonctionnalité *Échelle horizontale en lecture* n'est pas disponible à ces niveaux de service.

Le diagramme suivant illustre la fonctionnalité.

![Réplicas en lecture seule](./media/read-scale-out/business-critical-service-tier-read-scale-out.png)

La fonctionnalité *Échelle horizontale en lecture* est activée par défaut sur les nouvelles bases de données Premium, Critiques pour l'entreprise et Hyperscale. Pour Hyperscale, un réplica secondaire est créé par défaut pour les nouvelles bases de données. 

> [!NOTE]
> L'échelle horizontale en lecture est toujours activée au niveau de service Critique pour l'entreprise de Managed Instance.

Si votre chaîne de connexion SQL est configurée avec `ApplicationIntent=ReadOnly`, l'application est redirigée vers un réplica en lecture seule de cette base de données ou instance gérée. Pour plus d’informations sur la manière d’utiliser la propriété `ApplicationIntent`, voir [Spécification de l’intention de l’application](/sql/relational-databases/native-client/features/sql-server-native-client-support-for-high-availability-disaster-recovery#specifying-application-intent).

Si vous souhaitez vous assurer que l’application se connecte au réplica principal quel que soit le paramètre `ApplicationIntent` de la chaîne de connexion SQL, vous devez désactiver explicitement l’échelle horizontale en lecture lors de la création de la base de données ou de la modification de sa configuration. Par exemple, si vous mettez à niveau votre base de données du niveau Standard ou General Purpose vers le niveau Premium, Critique pour l'entreprise ou Hyperscale et que vous souhaitez que toutes vos connexions continuent d'être dirigées vers le réplica principal, désactivez la fonctionnalité Échelle horizontale en lecture. Pour plus d'informations sur la désactivation de cette fonctionnalité, consultez [Activer et désactiver l'échelle horizontale en lecture](#enable-and-disable-read-scale-out).

> [!NOTE]
> Les fonctionnalités Magasin des requêtes et Générateur de profils SQL ne sont pas prises en charge sur les réplicas en lecture seule. 

## <a name="data-consistency"></a>Cohérence des données

Un des avantages des réplicas est qu’ils sont toujours dans un état cohérent au niveau transactionnel, mais il peut y avoir une petite latence à différents moments entre les différents réplicas. La fonctionnalité Échelle horizontale en lecture prend en charge la cohérence au niveau de la session. Ainsi, si la session en lecture seule se reconnecte après une erreur de connexion engendrée par l’indisponibilité du réplica, elle peut être redirigée vers un autre réplica qui n’est pas complètement synchronisé avec le réplica en lecture-écriture. De même, si une application écrit des données à l’aide d’une session en lecture-écriture et les lit immédiatement à l’aide d’une session en lecture seule, il est possible que les dernières mises à jour ne soient pas visibles immédiatement sur le réplica. La latence est due à une opération de rétablissement de journal des transactions asynchrone.

> [!NOTE]
> Les latences de réplication dans la région sont réduites, et cette situation est rare. Pour surveiller la latence de réplication, consultez [Surveillance et dépannage de réplica en lecture seule](#monitoring-and-troubleshooting-read-only-replicas).

## <a name="connect-to-a-read-only-replica"></a>Se connecter à un réplica en lecture seule

Quand vous activez Échelle horizontale en lecture pour une base de données, l’option `ApplicationIntent` dans la chaîne de connexion fournie par le client détermine si la connexion est routée vers le réplica en écriture ou un réplica en lecture seule. Plus précisément, si la valeur `ApplicationIntent` est `ReadWrite` (valeur par défaut), la connexion est dirigée vers le réplica en lecture-écriture. Ce comportement est identique à celui qui se produit lorsque `ApplicationIntent` n'est pas inclus dans la chaîne de connexion. Si la valeur `ApplicationIntent` est `ReadOnly`, la connexion est acheminée vers un réplica en lecture seule.

Par exemple, la chaîne de connexion suivante connecte le client à un réplica en lecture seule (en remplaçant les éléments entre crochets pointus par les valeurs correctes pour votre environnement et en supprimant ces crochets) :

```sql
Server=tcp:<server>.database.windows.net;Database=<mydatabase>;ApplicationIntent=ReadOnly;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;
```

L’une des chaînes de connexion suivantes connecte le client à un réplica en lecture-écriture (en remplaçant les éléments entre crochets pointus par les valeurs correctes pour votre environnement et en supprimant ces crochets) :

```sql
Server=tcp:<server>.database.windows.net;Database=<mydatabase>;ApplicationIntent=ReadWrite;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;

Server=tcp:<server>.database.windows.net;Database=<mydatabase>;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;
```

## <a name="verify-that-a-connection-is-to-a-read-only-replica"></a>Vérifier que la connexion à un réplica en lecture seule est établie

Vous pouvez vérifier si vous êtes connecté à un réplica en lecture seule en exécutant la requête suivante dans le contexte de votre base de données. Celle-ci renvoie READ_ONLY lorsque vous êtes connecté à un réplica en lecture seule.

```sql
SELECT DATABASEPROPERTYEX(DB_NAME(), 'Updateability');
```

> [!NOTE]
> Aux niveaux de service Premium et Critique pour l'entreprise, vous ne pouvez accéder qu'à un seul réplica en lecture seule à la fois. Hyperscale prend en charge plusieurs réplicas en lecture seule.

## <a name="monitoring-and-troubleshooting-read-only-replicas"></a>Surveillance et dépannage de réplicas en lecture seule

Lorsqu'elles sont connectées à un réplica en lecture seule, les vues de gestion dynamique (DMV) reflètent l'état du réplica et peuvent être interrogées à des fins de surveillance et de dépannage. Le moteur de base de données fournit plusieurs affichages pour exposer un large éventail de données de surveillance. 

Les affichages couramment utilisés sont les suivants :

| Nom | Objectif |
|:---|:---|
|[sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database)| Fournit des métriques sur l'utilisation des ressources au cours de la dernière heure, y compris sur le processeur, les E/S de données et l'utilisation des écritures de journal par rapport aux limites d'objectif de service.|
|[sys.dm_os_wait_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql)| Fournit des statistiques d'attente agrégées pour l'instance du moteur de base de données. |
|[sys.dm_database_replica_states](/sql/relational-databases/system-dynamic-management-views/sys-dm-database-replica-states-azure-sql-database)| Fournit des statistiques sur l'état d'intégrité et la synchronisation des réplicas. La taille de la file d'attente de restauration par progression et la vitesse de restauration par progression constituent des indicateurs de la latence des données sur le réplica en lecture seule. |
|[sys.dm_os_performance_counters](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-performance-counters-transact-sql)| Fournit les compteurs de performances du moteur de base de données.|
|[sys.dm_exec_query_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-stats-transact-sql)| Fournit des statistiques d'exécution par requête, telles que le nombre d'exécutions, le temps processeur utilisé, etc.|
|[sys.dm_exec_query_plan()](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-plan-transact-sql)| Fournit les plans de requête mis en cache. |
|[sys.dm_exec_sql_text()](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sql-text-transact-sql)| Fournit un texte de requête pour un plan de requête mis en cache.|
|[sys.dm_exec_query_profiles](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-plan-stats-transact-sql)| Fournit la progression en temps réel pendant l'exécution des requêtes.|
|[sys.dm_exec_query_plan_stats()](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-plan-stats-transact-sql)| Fournit le dernier plan d'exécution réel connu, y compris les statistiques d'exécution relatives à une requête.|
|[sys.dm_io_virtual_file_stats()](/sql/relational-databases/system-dynamic-management-views/sys-dm-io-virtual-file-stats-transact-sql)| Fournit des statistiques de stockage IOPS, de débit et de latence pour tous les fichiers de base de données. |

> [!NOTE]
> Les vues de gestion dynamique `sys.resource_stats` et `sys.elastic_pool_resource_stats` de la base de données MASTER logique renvoient les données d'utilisation des ressources du réplica principal.

### <a name="monitoring-read-only-replicas-with-extended-events"></a>Surveillance des réplicas en lecture seule avec événements étendus

Il est impossible de créer une session d'événements étendus en cas de connexion à un réplica en lecture seule. Toutefois, dans Azure SQL Database, les définitions des sessions d'[Événements étendus](xevent-db-diff-from-svr.md) figurant dans l'étendue de la base de données et qui ont été créées et modifiées sur le réplica principal se répliquent sur les réplicas en lecture seule, y compris les géo-réplicas, et capturent les événements sur les réplicas en lecture seule.

Une session d'événements étendus sur un réplica en lecture seule basé sur une définition de session du réplica principal peut être démarrée et arrêtée indépendamment du réplica principal. Lorsqu'une session d'événements étendus est supprimée du réplica principal, elle est également supprimée de tous les réplicas en lecture seule.

### <a name="transaction-isolation-level-on-read-only-replicas"></a>Niveau d'isolement des transactions sur les réplicas en lecture seule

Les requêtes exécutées sur les réplicas en lecture seule sont toujours mappées avec le niveau d'isolement [de capture instantanée](/dotnet/framework/data/adonet/sql/snapshot-isolation-in-sql-server). L'isolement de capture instantanée utilise le contrôle de version de ligne pour éviter les scénarios où les lecteurs bloquent les enregistreurs.

Dans de rares cas, si une transaction d'isolement de capture instantanée accède à des métadonnées d'objet qui ont été modifiées dans une autre transaction simultanée, elle peut recevoir l'erreur [3961](/sql/relational-databases/errors-events/mssqlserver-3961-database-engine-error), « La transaction d'isolement de capture instantanée a échoué dans la base de données '%.*ls' car l'objet auquel l'instruction a eu accès a été modifié par une instruction DDL dans une autre transaction simultanée depuis le début de cette transaction. Elle est rejetée, car les métadonnées ne font pas l’objet d’une gestion des versions. Une mise à jour simultanée des métadonnées peut provoquer des incohérences si elle est combinée avec un isolement de capture instantanée. »

### <a name="long-running-queries-on-read-only-replicas"></a>Requêtes longues sur les réplicas en lecture seule

Les requêtes exécutées sur les réplicas en lecture seule doivent accéder aux métadonnées des objets référencés dans la requête (tables, index, statistiques, etc.) Dans de rares cas, si un objet de métadonnées est modifié sur le réplica principal alors qu'une requête contient un verrou correspondant au même objet sur le réplica en lecture seule, la requête peut [bloquer](/sql/database-engine/availability-groups/windows/troubleshoot-primary-changes-not-reflected-on-secondary#BKMK_REDOBLOCK) le processus qui applique les modifications du réplica principal au réplica en lecture seule. Si une telle requête devait s'exécuter pendant une longue période, elle entraînerait une désynchronisation importante entre le réplica en lecture seule et le réplica principal. 

Si une requête longue exécutée sur un réplica en lecture seule provoque ce type de blocage, elle est automatiquement interrompue et la session reçoit l'erreur 1219, « Votre session a été déconnectée en raison d'une opération DDL de priorité supérieure ».

> [!NOTE]
> Si vous recevez l'erreur 3961 ou 1219 lors de l'exécution de requêtes sur un réplica en lecture seule, relancez la requête.

> [!TIP]
> Aux niveaux de service Premium et Critique pour l'entreprise, en cas de connexion à un réplica en lecture seule, les colonnes `redo_queue_size` et `redo_rate` de la vue de gestion dynamique [sys.dm_database_replica_states](/sql/relational-databases/system-dynamic-management-views/sys-dm-database-replica-states-azure-sql-database) peuvent être utilisées pour surveiller le processus de synchronisation des données et servir d'indicateurs de latence des données sur le réplica en lecture seule.
> 

## <a name="enable-and-disable-read-scale-out"></a>Activer et désactiver l’échelle horizontale en lecture

L’échelle horizontale en lecture est activée par défaut sur les niveaux de service Premium, Critique pour l’entreprise et Hyperscale. L’échelle horizontale en lecture ne peut pas être activée au niveau de service De base, Standard ou Usage général. L’échelle horizontale en lecture est automatiquement désactivée sur les bases de données Hyperscale configurées avec zéro réplica.

Vous pouvez désactiver et réactiver l'échelle horizontale en lecture sur des bases de données uniques et des bases de données de pool élastique aux niveaux de service Premium ou Critique pour l’entreprise en utilisant les méthodes suivantes.

> [!NOTE]
> Pour les bases de données uniques et les bases de données de pool élastique, la possibilité de désactiver l'échelle horizontale en lecture est fournie à des fins de compatibilité descendante. L'échelle horizontale en lecture ne peut pas être désactivée sur les instances gérées de niveau Critique pour l'entreprise.

### <a name="azure-portal"></a>Portail Azure

Vous pouvez gérer le paramètre d’échelle horizontale en lecture sur le panneau base de données **Configurer** .

### <a name="powershell"></a>PowerShell

> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge, mais tous les développements à venir sont destinés au module Az.Sql. Le module Azure Resource Manager continuera à recevoir des résolutions de bogues jusqu’à au moins décembre 2020.  Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules Azure Resource Manager. Pour plus d’informations sur leur compatibilité, consultez [Présentation du nouveau module Az Azure PowerShell](/powershell/azure/new-azureps-module-az).

La gestion de l’échelle horizontale en lecture dans Azure PowerShell nécessite la version d’Azure PowerShell de décembre 2016 ou plus récente. Pour obtenir la version de PowerShell la plus récente, consultez [Azure PowerShell](/powershell/azure/install-az-ps).

Vous pouvez activer ou désactiver l’échelle horizontale en lecture dans Azure PowerShell en appelant l’applet de commande [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase) et en transmettant la valeur souhaitée (`Enabled` ou `Disabled`) pour le paramètre `-ReadScale`.

Pour désactiver la fonctionnalité Échelle horizontale en lecture sur une base de données existante (en remplaçant les éléments entre crochets angulaires par les valeurs correctes pour votre environnement et en supprimant ces crochets) :

```powershell
Set-AzSqlDatabase -ResourceGroupName <resourceGroupName> -ServerName <serverName> -DatabaseName <databaseName> -ReadScale Disabled
```

Pour désactiver la fonctionnalité Échelle horizontale en lecture sur une base de données existante (en remplaçant les éléments entre crochets angulaires par les valeurs correctes pour votre environnement et en supprimant ces crochets) :

```powershell
New-AzSqlDatabase -ResourceGroupName <resourceGroupName> -ServerName <serverName> -DatabaseName <databaseName> -ReadScale Disabled -Edition Premium
```

Pour réactiver la fonctionnalité Échelle horizontale en lecture sur une base de données existante (en remplaçant les éléments entre crochets angulaires par les valeurs correctes pour votre environnement et en supprimant ces crochets) :

```powershell
Set-AzSqlDatabase -ResourceGroupName <resourceGroupName> -ServerName <serverName> -DatabaseName <databaseName> -ReadScale Enabled
```

### <a name="rest-api"></a>API REST

Pour créer une base de données avec échelle horizontale en lecture désactivée, ou pour modifier le paramétrage d’une base de données existante, utilisez la méthode suivante avec la propriété `readScale` définie sur `Enabled` ou `Disabled`, comme dans l’exemple de requête suivant.

```rest
Method: PUT
URL: https://management.azure.com/subscriptions/{SubscriptionId}/resourceGroups/{GroupName}/providers/Microsoft.Sql/servers/{ServerName}/databases/{DatabaseName}?api-version= 2014-04-01-preview
Body: {
   "properties": {
      "readScale":"Disabled"
   }
}
```

Pour plus d’informations, consultez [Bases de données - Créer ou mettre à jour](/rest/api/sql/databases/createorupdate).

## <a name="using-the-tempdb-database-on-a-read-only-replica"></a>Utilisation de la base de données `tempdb` sur un réplica en lecture seule

La base de données `tempdb` du réplica principal n'est pas répliquée sur les réplicas en lecture seule. Chaque réplica possède sa propre base de données `tempdb` générée lors de la création du réplica. Il vérifie que la base de données `tempdb` peut être mise à jour et modifiée pendant l'exécution de votre requête. Si votre charge de travail en lecture seule dépend de l'utilisation d'objets `tempdb`, vous devez intégrer ceux-ci dans votre script de requête.

## <a name="using-read-scale-out-with-geo-replicated-databases"></a>Utilisation de l’échelle horizontale en lecture avec des bases de données géorépliquées

Les bases de données secondaires géorépliquées disposent de la même architecture de haute disponibilité que les bases de données primaires. Si vous vous connectez à la base de données secondaire géorépliquée et que l'échelle horizontale en lecture est activée, vos sessions avec `ApplicationIntent=ReadOnly` sont acheminées vers l'un des réplicas haute disponibilité de la même façon que sur la base de données primaire accessible en écriture. Les sessions sans `ApplicationIntent=ReadOnly` sont routées vers le réplica principal de la base de données secondaire géorépliquée, qui est également en lecture seule. 

Ainsi, la création d'un géo-réplica fournit deux autres réplicas en lecture seule pour une base de données primaire en lecture-écriture, soit un total de trois réplicas en lecture seule. Chaque géo-réplica supplémentaire fournit une autre paire de réplicas en lecture seule. Des géo-réplicas peuvent être créés dans n'importe quelle région Azure, y compris dans la région de la base de données primaire.

> [!NOTE]
> Il n'y a pas de tourniquet (round robin) automatique ou de routage à charge équilibrée entre les réplicas d'une base de données secondaire géo-répliquée.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur l’offre SQL Database Hyperscale, voir [Niveau de service Hyperscale](service-tier-hyperscale.md).