---
title: Haute disponibilité
titleSuffix: Azure SQL Database and SQL Managed Instance
description: En savoir plus sur les fonctionnalités de haute disponibilité des services Azure SQL Database et SQL Managed Instance
services: sql-database
ms.service: sql-db-mi
ms.subservice: high-availability
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: sashan
ms.author: sashan
ms.reviewer: sstein, sashan
ms.date: 10/28/2020
ms.openlocfilehash: 15067a046d8adc0ba38101bbe24cdc48cd433d56
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97095438"
---
# <a name="high-availability-for-azure-sql-database-and-sql-managed-instance"></a>Haute disponibilité des services Azure SQL Database et SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

L'objectif de l'architecture Haute disponibilité d'Azure SQL Database et de SQL Managed Instance est de garantir que votre base de données est opérationnelle au minimum 99,99 % du temps (pour plus d'informations sur les contrats de niveau de service [SLA] spécifiques aux différents niveaux de service, consultez [SLA d'Azure SQL Database et SQL Managed Instance](https://azure.microsoft.com/support/legal/sla/sql-database/)), sans que vous vous préoccupiez de l'impact des opérations de maintenance et des pannes. Azure gère automatiquement les tâches de maintenance critiques, telles les mises à jour correctives, les sauvegardes, les mises à niveau de Windows et d'Azure SQL, ainsi que les événements non planifiés, comme les défaillances matérielles, logicielles ou réseau sous-jacentes.  Lorsque la base de données sous-jacente d'Azure SQL Database fait l'objet d'une mise à jour corrective ou d'un basculement, le temps d'arrêt n'est pas perceptible si vous [utilisez une logique de nouvelle tentative](develop-overview.md#resiliency) dans votre application. Pour assurer la disponibilité de vos données, SQL Database et SQL Managed Instance bénéficient de fonctionnalités de récupération rapide, même dans les situations les plus critiques.

La solution de haute disponibilité est conçue pour garantir que les données validées ne sont jamais perdues pour cause d’échecs, que les opérations de maintenance n’affectent pas votre charge de travail, et que la base de données n’est pas un point de défaillance unique dans votre architecture logicielle. Rien, pas même une fenêtre de maintenance ou un temps d’arrêt, ne doit vous obliger à arrêter la charge de travail pendant la mise à niveau ou la maintenance de la base de données.

Deux modèles d'architecture à haute disponibilité sont disponibles :

- Le **modèle de disponibilité Standard** est basé sur la séparation du calcul et du stockage.  Il s’appuie sur la haute disponibilité et la fiabilité du niveau stockage à distance. Cette architecture cible des applications métier économiques, capables de tolérer une baisse de performances pendant les activités de maintenance.
- Le **modèle de disponibilité Premium** est basé sur un cluster de processus de moteur de base de données. Il repose sur le fait qu’il existe toujours un quorum de nœuds de moteur de base de données disponible. Cette architecture s’adresse à des applications stratégiques ayant des performances d’e/s supérieures et un taux de transactions élevé ; elle garantit un impact minimal sur les performances de votre charge de travail pendant les activités de maintenance.

Comme SQL Database et SQL Managed Instance s'exécutent tous les deux sur la dernière version stable du moteur de base de données SQL Server et du système d'exploitation Windows, la plupart des utilisateurs ne se rendent pas compte des mises à niveau qui sont effectuées en continu.

## <a name="basic-standard-and-general-purpose-service-tier-locally-redundant-availability"></a>Disponibilité de redondance locale des niveaux de service De base, Standard et Usage général

Les niveaux de service de base, standard et d’usage général tirent parti de l’architecture de disponibilité standard pour le calcul provisionné et serverless. L’illustration suivante montre quatre nœuds distincts, avec les couches de calcul et de stockage séparées.

![Séparation du calcul et du stockage](./media/high-availability-sla/general-purpose-service-tier.png)

Le modèle de disponibilité Standard comprend deux couches :

- Une couche de calcul sans état, qui exécute le processus `sqlservr.exe` et contient uniquement des données transitoires et mises en cache comme TempDB, les bases de données model sur le SSD attaché, ainsi que le cache du plan, le pool de mémoires tampons et le pool columnstore en mémoire. Ce nœud sans état est géré par Azure Service Fabric qui initialise `sqlservr.exe`, contrôle l’intégrité du nœud et effectue le basculement vers un autre nœud si nécessaire.
- Une couche de données avec état, comprenant les fichiers de base de données (.mdf/.ldf) stockés dans le service Stockage Blob Azure. La fonctionnalité de disponibilité et redondance des données est intégrée au stockage d’objets blob Azure. Celle-ci garantit la conservation de chaque enregistrement du fichier journal ou de chaque page du fichier de données, même si le processus `sqlservr.exe` se bloque.

Dès que le moteur de base de données ou que le système d'exploitation est mis à niveau, ou qu'une défaillance est détectée, Azure Service Fabric déplace le processus `sqlservr.exe` sans état vers un autre nœud de calcul sans état disposant d'une capacité disponible suffisante. Les données conservées dans le stockage Blob Azure ne sont pas affectées par le déplacement, et les fichiers de données ou de journaux sont joints au processus `sqlservr.exe` nouvellement initialisé. Ce processus garantit une disponibilité de 99,99 %, mais une charge de travail lourde peut accuser une baisse de performances pendant la transition, car le nouveau processus `sqlservr.exe` démarre avec le cache à froid.

## <a name="general-purpose-service-tier-zone-redundant-availability-preview"></a>Disponibilité redondante interzone du niveau de service Usage général (préversion)

La configuration redondante interzone pour le niveau de service Usage général utilise [Zones de disponibilité Azure](../../availability-zones/az-overview.md)  pour répliquer les bases de données sur plusieurs emplacements physiques au sein d’une région Azure. En sélectionnant la redondance dans une zone, vous pouvez rendre vos bases de données uniques à usage général et vos pools élastiques nouveaux et existants résistants à un plus grand éventail d’échecs, notamment les pannes graves de centre de données, sans aucune modification de la logique d’application.

La configuration redondante interzone pour le niveau Usage général contient deux couches :  

- Une couche de données avec état, comprenant les fichiers de base de données (.mdf/.ldf) stockés dans le [partage de fichiers du stockage](../../storage/files/storage-how-to-create-premium-fileshare.md) redondant interzone (ZRS PFS). Avec le [stockage redondant interzone](../../storage/common/storage-redundancy.md), les fichiers de données et les fichiers journaux sont copiés de façon synchrone sur trois zones de disponibilité Azure physiquement isolées.
- Une couche de calcul sans état, qui exécute le processus sqlservr.exe et contient uniquement des données transitoires et mises en cache comme TempDB, les bases de données model sur le SSD attaché, ainsi que le cache du plan, le pool de mémoires tampons et le pool columnstore en mémoire. Ce nœud sans état est géré par Azure Service Fabric qui initialise sqlservr.exe, contrôle l’intégrité du nœud et effectue le basculement vers un autre nœud si nécessaire. Pour les bases de données redondantes interzones à usage général, les nœuds avec une capacité de rechange sont facilement disponibles dans d’autres zones de disponibilité pour le basculement.

La version redondante interzone de l’architecture de haute disponibilité pour le niveau de service à usage général est illustrée dans le diagramme suivant :

![Configuration redondante interzone pour le niveau Usage général](./media/high-availability-sla/zone-redundant-for-general-purpose.png)

> [!IMPORTANT]
> Pour obtenir des informations à jour sur les régions qui prennent en charge les bases de données redondantes dans une zone, consultez [Prise en charge des services par région](../../availability-zones/az-region.md). La configuration redondante interzone n’est disponible que lorsque le matériel de calcul Gen5 est sélectionné. Cette fonctionnalité n'est pas disponible dans SQL Managed Instance.

> [!NOTE]
> Les bases de données à usage général dont la taille est de 80 vcore peuvent subir une altération des performances avec une configuration redondante interzone. En outre, les opérations telles que la sauvegarde, la restauration, la copie de base de données et la configuration de relations de récupération d’urgence de zone géographique peuvent connaître un ralentissement des performances pour toutes les bases de données uniques de plus de 1 To. 

## <a name="premium-and-business-critical-service-tier-locally-redundant-availability"></a>Disponibilité redondante locale des niveaux de service Premium et Critique pour l’entreprise

Les niveaux de service Premium et Critique pour l'entreprise tirent parti du modèle de disponibilité Premium, qui intègre des ressources de calcul (processus `sqlservr.exe`) et du stockage (SSD attaché localement) sur un seul nœud. La haute disponibilité est obtenue en répliquant calcul et stockage sur des nœuds supplémentaires pour la création d’un cluster à trois ou quatre nœuds.

![Cluster de nœuds de moteur de base de données](./media/high-availability-sla/business-critical-service-tier.png)

Les fichiers de base de données sous-jacents (.mdf/.ldf) sont placés sur le stockage SSD attaché, afin de fournir une latence des E/S très faible à votre charge de travail. La haute disponibilité est implémentée au moyen d'une technologie semblable à celles des [groupes de disponibilité AlwaysOn](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) de SQL Server. Le cluster comprend un seul réplica principal qui est accessible pour les charges de travail en lecture-écriture des clients, et un maximum de trois réplicas secondaires (de calcul et de stockage) contenant les copies des données. Le nœud principal envoie (push) régulièrement et dans l’ordre des modifications sur les nœuds secondaires, et s’assure que les données sont synchronisées sur au moins un réplica secondaire avant de valider chaque transaction. Ce processus garantit qu’en cas de plantage du nœud principal pour une quelconque raison, il existe toujours un nœud entièrement synchronisé vers lequel basculer. Le basculement est initié par Azure Service Fabric. Lorsque le réplica secondaire devient le nouveau nœud principal, un autre réplica secondaire est créé pour garantir que le cluster dispose de suffisamment de nœuds (ensemble du quorum). Une fois le basculement terminé, les connexions Azure SQL sont automatiquement redirigées vers le nouveau nœud principal.

Autre avantage, le modèle de disponibilité Premium permet de rediriger les connexions Azure SQL en lecture seule vers un des réplicas secondaires. Cette fonctionnalité est appelée [Scale-out en lecture](read-scale-out.md). Elle fournit 100 % de capacité de calcul, sans frais supplémentaires, pour décharger depuis le réplica principal des opérations en lecture seule, telles que les charges de travail analytiques.

## <a name="premium-and-business-critical-service-tier-zone-redundant-availability"></a>Disponibilité redondante interzone des niveaux de service Premium et Critique pour l’entreprise 

Par défaut, le cluster de nœuds pour le modèle de disponibilité Premium est créé dans le même centre de données. Avec l’introduction des [Zones de disponibilité Azure](../../availability-zones/az-overview.md), SQL Database peut placer différents réplicas de la base de données Critique pour l’entreprise dans des zones de disponibilité distinctes au sein de la même région. Pour éliminer un point de défaillance unique, l’anneau de contrôle est également dupliqué sur plusieurs fuseaux horaires sous forme de trois anneaux de passerelle (GW). Le routage vers un anneau de passerelle spécifique est contrôlé par [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md) (ATM). Étant donné que la configuration redondante interzone dans les niveaux de service Premium ou Critique pour l’entreprise ne crée pas de redondance de base de données supplémentaire, vous pouvez l’activer sans frais supplémentaires. En sélectionnant une configuration redondante dans une zone, vous rendez vos bases de données Premium ou Critique pour l’entreprise résistantes à un plus grand éventail d’échecs, notamment les pannes graves de centre de données, sans aucune modification à la logique d’application. Vous pouvez également convertir vos bases de données ou pools Premium ou Critique pour l’entreprise en configuration avec redondance dans une zone.

Les bases de données de redondance de zone, ayant des réplicas dans différents centres de données avec une certaine distance entre eux, la latence accrue du réseau peut augmenter le temps de validation et ainsi avoir un impact sur les performances de certaines charges de travail OLTP. Vous pouvez toujours revenir à la configuration de zone unique en désactivant le paramètre de redondance de zone. Ce processus est une opération en ligne, semblable à la mise à niveau des niveaux de service ordinaires. À la fin du processus, la base de données ou le pool est migré à partir d’un anneau de redondance de zone vers un anneau de zone unique, ou vice versa.

> [!IMPORTANT]
> Lorsque vous utilisez le niveau Critique pour l’entreprise, la configuration de zone redondante est uniquement disponible lorsque le matériel de calcul Gen5 est sélectionné. Pour obtenir des informations à jour sur les régions qui prennent en charge les bases de données redondantes dans une zone, consultez [Prise en charge des services par région](../../availability-zones/az-region.md).

> [!NOTE]
> Cette fonctionnalité n'est pas disponible dans SQL Managed Instance.

La version avec redondance de zone de l’architecture de haute disponibilité est illustrée dans le diagramme suivant :

![architecture haute disponibilité avec redondance de zone](./media/high-availability-sla/zone-redundant-business-critical-service-tier.png)


## <a name="hyperscale-service-tier-availability"></a>Disponibilité du niveau de service Hyperscale

L'architecture du niveau de service Hyperscale est décrite dans [Architecture des fonctions distribuées](./service-tier-hyperscale.md#distributed-functions-architecture) et n'est actuellement disponible que pour SQL Database, et non pour SQL Managed Instance.

![Architecture fonctionnelle Hyperscale](./media/high-availability-sla/hyperscale-architecture.png)

Le modèle de disponibilité dans Hyperscale comprend quatre couches :

- Une couche de calcul sans état, qui exécute les processus `sqlservr.exe` et contient uniquement des données transitoires et mises en cache, comme le cache RBPEX non couvrant, TempDB, une base de données model, etc. sur le SSD attaché, ainsi que le cache du plan, le pool de mémoires tampons et le pool columnstore en mémoire. Cette couche sans état comprend le réplica de calcul principal et, éventuellement, un certain nombre de réplicas de calcul secondaires qui peuvent servir de cibles de basculement.
- Une couche de stockage sans état formée de serveurs de pages. Cette couche est le moteur de stockage distribué des processus `sqlservr.exe` s’exécutant sur les réplicas de calcul. Chaque serveur de pages contient uniquement des données temporaires et mises en cache, comme le cache RBPEX couvrant sur le disque SSD attaché ainsi que les pages de données mise en cache dans la mémoire. Chaque serveur de pages dispose d’un serveur de pages appairé dans une configuration active-active pour assurer l’équilibrage de charge, la redondance et la haute disponibilité.
- Une couche de stockage du journal des transactions avec état formée du nœud de calcul exécutant le processus du service de journal, la zone d’atterrissage du journal des transactions et le stockage durable du journal des transactions. La zone d’atterrissage et le stockage durable utilisent Stockage Azure, qui assure la disponibilité et la [redondance](../../storage/common/storage-redundancy.md) du journal des transactions, ce qui garantit la durabilité des données pour les transactions validées.
- Une couche de stockage de données avec état avec les fichiers de base de données (.mdf/.ndf) qui sont stockés dans Stockage Azure et mis à jour par les serveurs de pages. Cette couche utilise les fonctionnalités de disponibilité et de [redondance](../../storage/common/storage-redundancy.md) des données de Stockage Azure. Cela garantit que chaque page d’un fichier de données est conservée, même si les processus des autres couches de l’architecture Hyperscale se bloquent ou si des nœuds de calcul subissent une défaillance.

Dans toutes les couches Hyperscale, les nœuds de calcul s’exécutent sur Azure Service Fabric, qui contrôle la santé de chaque nœud et assure les basculements vers des nœuds sains disponibles quand cela est nécessaire.

Pour plus d’informations sur la haute disponibilité dans Hyperscale, consultez [Haute disponibilité de la base de données dans Hyperscale ](./service-tier-hyperscale.md#database-high-availability-in-hyperscale).


## <a name="accelerated-database-recovery-adr"></a>Récupération de base de données accélérée (ADR)

La [récupération de base de données accélérée](../accelerated-database-recovery.md) est une nouvelle fonctionnalité du moteur de base de données qui améliore considérablement la disponibilité des bases de données, particulièrement en présence de transactions durables. Actuellement, ADS est disponible pour Azure SQL Database, Azure SQL Managed Instance et Azure Synapse Analytics.

## <a name="testing-application-fault-resiliency"></a>Test de résilience aux erreurs de l’application

La haute disponibilité est un élément fondamental de la plateforme SQL Database et SQL Managed Instance ; elle fonctionne de manière transparente pour votre application de base de données. Toutefois, nous avons conscience que vous souhaitez peut-être tester, avant le déploiement en production, la manière dont les opérations de basculement automatique initiées pendant les événements, planifiés ou non, impacteraient une application. Vous pouvez déclencher manuellement un basculement en appelant une API spéciale pour redémarrer une base de données, un pool élastique ou une instance gérée. Dans le cas d’une base de données ou d’un pool élastique redondant dans une zone, l’appel d’API entraînerait la redirection des connexions clientes vers le nouveau réplica principal dans une zone de disponibilité différente de l’ancien. Ainsi, en plus de tester l’impact du basculement sur les sessions de base de données existantes, vous pouvez aussi vérifier s’il a un impact sur les performances de bout en bout en raison des changements de latence réseau. Sachant que l’opération de redémarrage est intrusive et qu’un grand nombre de redémarrages pourrait peser sur la plateforme, un seul appel de basculement est autorisé toutes les 15 minutes pour chaque base de données, pool élastique ou instance gérée.

Un basculement peut être initié à l’aide de PowerShell, de l’API REST ou d’Azure CLI :

|Type de déploiement|PowerShell|API REST| Azure CLI|
|:---|:---|:---|:---|
|Base de données|[Invoke-AzSqlDatabaseFailover](/powershell/module/az.sql/invoke-azsqldatabasefailover)|[Basculement de base de données](/rest/api/sql/databases/failover)|[az rest](/cli/azure/reference-index#az-rest) peut permettre d’invoquer un appel d’API REST à partir d’Azure CLI|
|Pool élastique|[Invoke-AzSqlElasticPoolFailover](/powershell/module/az.sql/invoke-azsqlelasticpoolfailover)|[Basculement de pool élastique](/rest/api/sql/elasticpools(failover)/failover/)|[az rest](/cli/azure/reference-index#az-rest) peut permettre d’invoquer un appel d’API REST à partir d’Azure CLI|
|Instance gérée|[Invoke-AzSqlInstanceFailover](/powershell/module/az.sql/Invoke-AzSqlInstanceFailover/)|[Instances gérées - Basculement](/rest/api/sql/managed%20instances%20-%20failover/failover)|[Basculement az sql mi](/cli/azure/sql/mi/#az-sql-mi-failover)|

> [!IMPORTANT]
> La commande de basculement n’est pas disponible pour les réplicas secondaires accessibles en lecture des bases de données Hyperscale.

## <a name="conclusion"></a>Conclusion

Azure SQL Database et Azure SQL Managed Instance disposent d'une solution de haute disponibilité intégrée, qui est incorporée en profondeur à la plateforme Azure. Cette solution dépend de Service Fabric pour la détection et la récupération des défaillances, mais aussi du stockage Blob Azure pour la protection des données et des zones de disponibilité pour une meilleure tolérance aux pannes (comme mentionné plus haut dans ce document, cela n’est pas encore applicable à Azure SQL Managed Instance). Par ailleurs, SQL Database et SQL Managed Instance tirent parti de la technologie de groupe de disponibilité Always On de l'instance SQL Server pour la réplication et le basculement. La combinaison de ces technologies permet aux applications de profiter pleinement des avantages d’un modèle de stockage mixte, et de prendre en charge les contrats de niveau de service les plus exigeants.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [Zones de disponibilité Azure](../../availability-zones/az-overview.md)
- En savoir plus sur [Service Fabric](../../service-fabric/service-fabric-overview.md)
- En savoir plus sur [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md).
- Découvrir [Comment lancer un basculement manuel sur SQL Managed Instance](../managed-instance/user-initiated-failover.md)
- Pour découvrir plus d’options concernant la haute disponibilité et la reprise d’activité après sinistre, consultez [Continuité d’activité](business-continuity-high-availability-disaster-recover-hadr-overview.md).