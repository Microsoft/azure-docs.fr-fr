---
title: Questions fréquentes (FAQ) sur le niveau Hyperscale dans Azure SQL Database
description: Réponses aux questions fréquemment posées par les clients sur une base de données dans SQL Database dans le niveau de service Hyperscale, communément appelée « base de données Hyperscale ».
services: sql-database
ms.service: sql-database
ms.subservice: ''
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: dimitri-furman
ms.author: dfurman
ms.reviewer: ''
ms.date: 03/03/2020
ms.openlocfilehash: 4ea1982e7545f4ac39a5ecd15dc9e19a582ae31c
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96459632"
---
# <a name="azure-sql-database-hyperscale-faq"></a>Questions fréquentes (FAQ) sur le niveau Hyperscale dans Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article fournit des réponses aux questions fréquemment posées pour les clients envisageant d’utiliser une base de données pour le niveau de service Hyperscale Azure SQL Database, appelée simplement Hyperscale dans le reste de ce FAQ. Cet article décrit les scénarios pris en charge par Hyperscale et les caractéristiques qui sont compatibles avec Hyperscale.

- Ce FAQ est destiné aux personnes qui ont des connaissances générales sur le niveau de service Hyperscale et qui cherchent des réponses à leurs questions et préoccupations spécifiques.
- Il n’est pas un guide de mise en œuvre et n’est pas destiné à répondre à des questions sur la façon d’utiliser une base de données Hyperscale. Pour une introduction à Hyperscale, nous vous recommandons de vous reporter à la documentation [Azure SQL Database Hyperscale](service-tier-hyperscale.md).

## <a name="general-questions"></a>Questions générales

### <a name="what-is-a-hyperscale-database"></a>Qu’est-ce qu’une base de données Hyperscale ?

Une base de données Hyperscale est une base de données dans SQL Database dans le niveau de service Hyperscale qui repose sur la technologie de stockage Scale-out d’Hyperscale. Une base de données Hyperscale prend en charge jusqu’à 100 To de données, offre des performances et un débit élevés, ainsi qu’une mise à l’échelle rapide pour répondre aux exigences des charges de travail. La mise à l’échelle est transparente pour l’application : la connectivité, le traitement des requêtes, etc., fonctionnent comme pour toute autre base de données dans Azure SQL Database.

### <a name="what-resource-types-and-purchasing-models-support-hyperscale"></a>Quels sont les types de ressources et les modèles d’achat qui prennent en charge Hyperscale ?

Le niveau de service Hyperscale est disponible seulement pour les bases de données uniques avec le modèle d’achat vCore dans Azure SQL Database.  

### <a name="how-does-the-hyperscale-service-tier-differ-from-the-general-purpose-and-business-critical-service-tiers"></a>En quoi le niveau de service Hyperscale diffère-t-il des niveaux de service Usage général et Critique pour l’entreprise ?

Les niveaux de service basés sur vCore sont différenciés en fonction de la disponibilité de la base de données et du type de stockage, des performances et de la taille maximale, comme décrit dans le tableau suivant.

| | Type de ressource | Usage général |  Hyperscale | Critique pour l’entreprise |
|:---:|:---:|:---:|:---:|:---:|
| **Idéal pour** |Tous|Offre des options de calcul et de stockage équilibrées et économiques.|La plupart des charges de travail d’entreprise. Mise à l’échelle automatique de la taille de stockage jusqu’à 100 To, mise à l’échelle verticale et horizontale rapide du calcul, restauration rapide de la base de données.|Applications OLTP avec des débits de transactions élevés et une faible latence des E/S. Offre une meilleure résilience aux défaillances et des basculements rapides à l’aide de plusieurs réplicas mis à jour de façon synchrone.|
|  **Type de ressource** ||SQL Database / SQL Managed Instance | Base de données unique | SQL Database / SQL Managed Instance |
| **Taille de calcul**|SQL Database* | 1 à 80 cœurs virtuels | 1 à 80 cœurs virtuels* | 1 à 80 cœurs virtuels |
| **Taille de calcul**|Instance managée SQL | 8, 16, 24, 32, 40, 64, 80 cœurs virtuels | N/A | 8, 16, 24, 32, 40, 64, 80 cœurs virtuels |
| **Type de stockage** | Tous |Stockage distant Premium (par instance) | Stockage découplé avec cache disque SSD local (par instance) | Stockage SSD local ultra-rapide (par instance) |
| **Taille de stockage** | SQL Database *| 5 Go - 4 To | Jusqu’à 100 To | 5 Go - 4 To |
| **Taille de stockage** | Instance managée SQL  | 32 Go - 8 To | N/A | 32 Go - 4 To |
| **D’OPÉRATIONS D’E/S PAR SECONDE** | Base de données unique | 500 IOPS par vCore avec 7000 IOPS au maximum | L’architecture hyperscale est une architecture à plusieurs niveaux avec une mise en cache sur plusieurs niveaux. L’efficacité des IOPS dépend de la charge de travail. | 5 000 IOPS avec un maximum de 200 000 IOPS|
| **D’OPÉRATIONS D’E/S PAR SECONDE** | Instance managée SQL | Dépend de la taille du fichier | N/A | 1375 IOPS/vCore |
|**Disponibilité**|Tous|1 réplica, pas d’échelle horizontale en lecture, pas de cache local | Plusieurs réplicas, jusqu’à 4 échelles horizontales en lecture, cache local partiel | 3 réplicas, 1 échelle horizontale en lecture, haute disponibilité redondante interzone, stockage local complet |
|**Sauvegardes**|Tous|RA-GRS, 7 à 35 jours de rétention (7 jours par défaut)| RA-GRS, 7 jours de rétention, récupération jusqu’à une date et heure (PITR) à durée constante | RA-GRS, 7 à 35 jours de rétention (7 jours par défaut) |

\* Pools élastiques ne sont pas pris en charge dans le niveau de service Hyperscale

### <a name="who-should-use-the-hyperscale-service-tier"></a>À qui est destiné le niveau de service Hyperscale ?

Le niveau de service Hyperscale est destiné aux clients qui ont de grandes bases de données SQL Server locales et qui veulent moderniser leurs applications en les déplaçant dans le cloud, ou aux clients qui utilisent déjà Azure SQL Database et qui veulent étendre considérablement le potentiel de croissance de leurs bases de données. Hyperscale est également destiné aux clients qui recherchent à la fois des hautes performances et une scalabilité élevée. Avec Hyperscale, vous bénéficiez des avantages suivants :

- Taille de la base de données jusqu’à 100 To
- Sauvegardes de base de données rapides, quelle que soit la taille des bases de données (les sauvegardes sont basées sur des captures instantanées de stockage)
- Restaurations de base de données rapides, quelle que soit la taille des bases de données (les restaurations sont effectuées à partir des captures instantanées de stockage)
- Débit de journalisation plus élevé, quelle que soit la taille de la base de données et le nombre de vCores
- Échelle horizontale en lecture à l’aide d’un ou de plusieurs réplicas en lecture seule, utilisés pour le déchargement de lecture et en tant que serveurs de secours.
- Scale-up rapide des calculs, en durée constante, de façon à gagner en puissance pour faire face à des charges de travail importantes, puis à réduire cette puissance, toujours en durée constante. C’est similaire au scale-up et au scale-down entre par exemple une machine P6 et une machine P11, mais beaucoup plus rapide, car il ne s’agit pas d’une opération relative à la taille des données.

### <a name="what-regions-currently-support-hyperscale"></a>Quelles régions prennent actuellement en charge Hyperscale ?

Le niveau de service Hyperscale est actuellement disponible dans les régions répertoriées dans [Vue d’ensemble de l’option Hyperscale d’Azure SQL Database](service-tier-hyperscale.md#regions).

### <a name="can-i-create-multiple-hyperscale-databases-per-server"></a>Puis-je créer plusieurs bases de données Hyperscale par serveur ?

Oui. Pour plus d’informations et pour connaître les limites quant au nombre de bases de données Hyperscale par serveur, consultez [Limites de ressources de SQL Database pour les bases de données uniques et mises en pool en pool sur un serveur](resource-limits-logical-server.md).

### <a name="what-are-the-performance-characteristics-of-a-hyperscale-database"></a>Quelles sont les caractéristiques en matière de performances d’une base de données Hyperscale ?

L’architecture Hyperscale fournit des performances et des débits élevés avec la prise en charge de bases de données de grande taille.

### <a name="what-is-the-scalability-of-a-hyperscale-database"></a>Quelle est la scalabilité d’une base de données Hyperscale ?

Hyperscale offre une scalabilité rapide en fonction de la demande de votre charge de travail.

- **Scale-up/down**

  Avec Hyperscale, vous pouvez augmenter la taille de calcul principale en termes de ressources, comme le processeur et la mémoire, puis mettre à l’échelle vers le bas, en durée constante. Comme le stockage est partagé, le scale-up et le scale-down ne sont pas une opération relative à la taille des données.  
- **Scale-up/down**

  Avec Hyperscale, vous avec également la possibilité de provisionner un ou plusieurs réplicas de calcul supplémentaires, que vous pouvez utiliser pour répondre à vos demandes de lecture. Cela signifie que vous pouvez utiliser ces réplicas de calcul supplémentaires comme réplicas en lecture seule pour décharger votre charge de travail de lecture en dehors de la capacité de calcul principale. En plus des opérations en lecture seule, ces réplicas font également office de serveurs de secours en cas de basculement à partir du serveur principal.

  Le provisionnement de chacun de ces réplicas de calcul supplémentaires peut être effectué en durée constante et est une opération en ligne. Vous pouvez vous connecter à ces réplicas de calcul supplémentaires en lecture seule en définissant l’argument `ApplicationIntent` de votre chaîne de connexion sur `ReadOnly`. Les connexions avec l’intention d’application `ReadOnly` sont automatiquement routées vers un des nœuds de calcul supplémentaires en lecture seule.

## <a name="deep-dive-questions"></a>Questions approfondies

### <a name="can-i-mix-hyperscale-and-single-databases-in-a-single-server"></a>Puis-je associer une base de données Hyperscale et des bases de données uniques dans un même serveur ?

Oui, vous pouvez.

### <a name="does-hyperscale-require-my-application-programming-model-to-change"></a>Hyperscale nécessite-t-il la modification de mon modèle de programmation d’application ?

Non, votre modèle de programmation d’application reste tel quel. Vous utilisez votre chaîne de connexion comme d’habitude et les autres méthodes normales pour interagir avec votre base de données Hyperscale.

### <a name="what-transaction-isolation-level-is-the-default-in-a-hyperscale-database"></a>Quel est le niveau d’isolation par défaut des transactions dans une base de données Hyperscale

Sur le réplica principal, le niveau d’isolement par défaut des transactions est RCSI (Read Committed Snapshot Isolation). Sur les réplicas secondaires avec échelle horizontale en lecture, le niveau d’isolation par défaut est Instantané.

### <a name="can-i-bring-my-on-premises-or-iaas-sql-server-license-to-hyperscale"></a>Puis-je utiliser ma licence SQL Server locale ou IaaS pour Hyperscale ?

Oui, [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) est disponible pour Hyperscale. Chaque cœur SQL Server Standard peut être mappé à 1 cœur virtuel Hyperscale. Chaque cœur SQL Server Entreprise peut être mappé à 4 cœurs virtuels Hyperscale. Vous n’avez pas besoin d’une licence SQL pour les réplicas secondaires. Le prix Azure Hybrid Benefit est appliqué automatiquement aux réplicas (secondaires) en échelle horizontale en lecture.

### <a name="what-kind-of-workloads-is-hyperscale-designed-for"></a>Pour quelle type de charges de travail Hyperscale est-il conçu ?

Hyperscale prend en charge toutes les charges de travail SQL Server, mais il est principalement optimisé pour OLTP. Vous pouvez également exécuter des charges de travail hybrides (HTAP) et analytiques (mini-Data Warehouse).

### <a name="how-can-i-choose-between-azure-synapse-analytics-and-azure-sql-database-hyperscale"></a>Comment choisir entre Azure Synapse Analytics et Azure SQL Database Hyperscale ?

Si vous exécutez actuellement des requêtes analytiques interactives avec SQL Server comme entrepôt de données, Hyperscale est une option intéressante, car vous pouvez héberger des entrepôts de données de taille petite et moyenne (par exemple de quelques To jusqu’à 100 To) à un coût inférieur, et vous pouvez migrer les charges de travail de votre entrepôt de données SQL Server vers Hyperscale avec des modifications minimales du code T-SQL.

Si vous exécutez de l’analytique des données à grande échelle avec des requêtes complexes et des taux d’ingestion supérieurs à 100 Mo/s, ou en utilisant Parallel Data Warehouse, Teradata ou d’autres entrepôts de données MPP (Massively Parallel Processing), Azure Synapse Analytics peut être le meilleur choix.
  
## <a name="hyperscale-compute-questions"></a>Questions sur la capacité de calcul d’Hyperscale

### <a name="can-i-pause-my-compute-at-any-time"></a>Puis-je interrompre à tout moment ma capacité de calcul ?

Pas pour l’instant, mais vous pouvez faire descendre en puissance votre calcul et le nombre de réplicas pour réduire les coûts pendant les heures creuses.

### <a name="can-i-provision-a-compute-replica-with-extra-ram-for-my-memory-intensive-workload"></a>Puis-je approvisionner un réplica de calcul avec de la RAM supplémentaire pour ma charge de travail utilisant beaucoup de mémoire ?

Non. Pour obtenir davantage de RAM, vous devez effectuer une mise à niveau vers une taille de calcul plus grande. Pour plus d’informations, consultez [Tailles de stockage et taille de calcul Hyperscale](resource-limits-vcore-single-databases.md#hyperscale---provisioned-compute---gen5).

### <a name="can-i-provision-multiple-compute-replicas-of-different-sizes"></a>Puis-je approvisionner plusieurs réplicas de calcul de tailles différentes ?

Non.

### <a name="how-many-read-scale-out-replicas-are-supported"></a>Combien de réplicas avec échelle horizontale en lecture sont pris en charge ?

Les bases de données Hyperscale sont créées par défaut avec un réplica avec échelle horizontale en lecture (deux réplicas y compris le principal). Vous pouvez mettre à l’échelle le nombre de réplicas en lecture seule entre 0 et 4 à l’aide du [Portail Azure](https://portal.azure.com) ou de l’[API REST](/rest/api/sql/databases/createorupdate).

### <a name="for-high-availability-do-i-need-to-provision-additional-compute-replicas"></a>Pour la haute disponibilité, dois-je approvisionner des nœuds de calcul supplémentaires ?

Dans les bases de données Hyperscale, la résilience des données est fournie au niveau du stockage. Un seul réplica est suffisant pour fournir la résilience. Quand le réplica de capacité de calcul est défaillant, un nouveau réplica est créé automatiquement sans perte de données.

Cependant, s’il n’existe qu’un seul réplica, un certain temps peut être nécessaire pour générer le cache local dans le nouveau réplica après le basculement. Pendant la phase de reconstruction du cache, la base de données extrait les données directement à partir des serveurs de pages, ce qui entraîne une latence de stockage supérieure et une dégradation des performances des requêtes.

Pour les applications critiques qui nécessitent une haute disponibilité avec un impact de basculement minimal, vous devez approvisionner au moins 2 réplicas de calcul, y compris le réplica de calcul principal. Il s’agit de la configuration par défaut. De cette façon, il y a un réplica de secours à chaud qui fait office de cible de basculement.

## <a name="data-size-and-storage-questions"></a>Questions sur la taille et le stockage des données

### <a name="what-is-the-maximum-database-size-supported-with-hyperscale"></a>Quelle est la taille maximale de base de données prise en charge avec Hyperscale ?

100 To.

### <a name="what-is-the-size-of-the-transaction-log-with-hyperscale"></a>Quelle est la taille du journal des transactions avec Hyperscale ?

Le journal des transactions avec Hyperscale est pratiquement infini. Vous n’avez pas à vous soucier d’un espace insuffisant pour le journal sur un système qui a un débit de journalisation élevé. Cependant, le débit de génération du journal peut être limité pour les charges de travail en écriture agressives continues. La vitesse de génération de journal maximale soutenue est de 100 Mo/s.

### <a name="does-my-tempdb-scale-as-my-database-grows"></a>Ma `tempdb` évolue-t-elle au fil de la croissance de ma base de données ?

Votre base de données `tempdb` se trouve sur un stockage SSD local et elle est dimensionnée proportionnellement à la taille de calcul que vous provisionnez. Votre base de données `tempdb` est optimisée de façon à fournir des performances optimales. La taille de `tempdb` n’est pas configurable et elle est gérée pour vous.

### <a name="does-my-database-size-automatically-grow-or-do-i-have-to-manage-the-size-of-data-files"></a>La taille de ma base de données augmente-t-elle automatiquement ou dois-je gérer la taille des fichiers de données ?

La taille de votre base de données croît automatiquement au fil de l’insertion/ingestion de données.

### <a name="what-is-the-smallest-database-size-that-hyperscale-supports-or-starts-with"></a>Quelle est la plus petite taille de base de données prise en charge par Hyperscale ou avec laquelle il peut démarrer ?

40 Go. Une base de données Hyperscale est créée avec une taille de départ de 10 Go. Ensuite, elle commence à croître de 10 Go toutes les 10 minutes, jusqu’à atteindre la taille de 40 Go. Chacun de ces blocs de 10 Go est alloué sur un serveur de pages différent afin de fournir un nombre d’IOPS plus élevé et un parallélisme d’E/S supérieur. En raison de cette optimisation, même si vous choisissez une taille de base de données initiale inférieure à 40 Go, la base de données passera automatiquement à au moins 40 Go.

### <a name="in-what-increments-does-my-database-size-grow"></a>De quel incrément la taille de ma base de données augmente-t-elle ?

Chaque fichier de données croît de 10 Go. Plusieurs fichiers de données peuvent croître en même temps.

### <a name="is-the-storage-in-hyperscale-local-or-remote"></a>Le stockage dans Hyperscale est-il local ou distant ?

Dans Hyperscale, les fichiers de données sont stockés dans le stockage Azure standard. Les données sont entièrement mises en cache sur le stockage SSD local, sur les serveurs de pages qui sont proches des réplicas de calcul. En outre, les réplicas de calcul ont des caches de données sur un SSD local et en mémoire, afin de réduire la fréquence des extractions de données sur les serveurs de pages distants.

### <a name="can-i-manage-or-define-files-or-filegroups-with-hyperscale"></a>Puis-je gérer ou définir des fichiers ou des groupes de fichiers avec Hyperscale ?

Non. Les fichiers de données sont ajoutés automatiquement. Les raisons courantes de la création de groupes de fichiers supplémentaires ne s’appliquent pas à l’architecture de stockage Hyperscale.

### <a name="can-i-provision-a-hard-cap-on-the-data-growth-for-my-database"></a>Puis-je provisionner une limite stricte sur la croissance des données pour ma base de données ?

Non.

### <a name="how-are-data-files-laid-out-with-hyperscale"></a>Comment les fichiers de données se présentent-ils avec Hyperscale ?

Les fichiers de données sont contrôlés par les serveurs de pages, avec un serveur de pages par fichier de données. Au fil de l’augmentation de la taille des données, des fichiers de données et des serveurs de pages associés sont ajoutés.

### <a name="is-database-shrink-supported"></a>La réduction de base de données est-elle prise en charge ?

Non.

### <a name="is-data-compression-supported"></a>La compression des données est-elle prise en charge ?

Oui, y compris la compression de ligne, de page et columnstore.

### <a name="if-i-have-a-huge-table-does-my-table-data-get-spread-out-across-multiple-data-files"></a>Si j’ai une table très grande, les données de ma table sont-elles réparties dans plusieurs fichiers de données ?

Oui. Les pages de données associées à une table donnée peuvent être stockées dans plusieurs fichiers de données, qui font tous partie du même groupe de fichiers. SQL Server utilise une [stratégie de remplissage proportionnel](/sql/relational-databases/databases/database-files-and-filegroups#file-and-filegroup-fill-strategy) pour distribuer les données entre des fichiers de données.

## <a name="data-migration-questions"></a>Questions relatives à la migration des données

### <a name="can-i-move-my-existing-databases-in-azure-sql-database-to-the-hyperscale-service-tier"></a>Puis-je déplacer mes bases de données d’Azure SQL Database vers le niveau de service Hyperscale ?

Oui. Vous pouvez déplacer vos bases de données existantes d’Azure SQL Database vers Hyperscale. Il s’agit d’une migration unidirectionnelle. Vous ne pouvez pas déplacer des bases de données depuis Hyperscale vers un autre niveau de service. Pour les preuves de concept, nous vous recommandons d’effectuer une copie de votre base de données et de migrer la copie vers Hyperscale. 

Le temps nécessaire pour déplacer une base de données existante vers Hyperscale comprend le temps de copie des données et celui de relecture des modifications apportées dans la base de données source lors de la copie des données. Le temps de copie des données est proportionnel à la taille des données. Le temps de relecture des modifications sera plus court si le déplacement est effectué pendant une période de faible activité d’écriture.
  
### <a name="can-i-move-my-hyperscale-databases-to-other-service-tiers"></a>Puis-je déplacer mes bases de données Hyperscale vers d’autres niveaux de service ?

Non. Actuellement, vous ne pouvez pas déplacer une base de données Hyperscale vers un autre niveau de service.

### <a name="do-i-lose-any-functionality-or-capabilities-after-migration-to-the-hyperscale-service-tier"></a>Est-ce que je perds des fonctionnalités ou des capacités après la migration vers le niveau de service Hyperscale ?

Oui. Certaines fonctionnalités Azure SQL Database ne sont pas encore prises en charge dans Hyperscale, notamment la conservation à long terme des sauvegardes. Après la migration de vos bases de données vers Hyperscale, ces fonctionnalités cessent de fonctionner.  Nous espérons que ces limites ne seront pas définitives.

### <a name="can-i-move-my-on-premises-sql-server-database-or-my-sql-server-database-in-a-cloud-virtual-machine-to-hyperscale"></a>Puis-je déplacer ma base de données SQL Server locale ou ma base de données SQL Server dans une machine virtuelle cloud vers Hyperscale ?

Oui. Vous pouvez utiliser toutes les technologies de migration existantes pour migrer vers Hyperscale, notamment la réplication transactionnelle, et toute autre technologie de déplacement de données (copie en bloc, Azure Data Factory, Azure Databricks, SSIS). Voir aussi l’[Azure Database Migration Service](../../dms/dms-overview.md), qui prend en charge de nombreux scénarios de migration.

### <a name="what-is-my-downtime-during-migration-from-an-on-premises-or-virtual-machine-environment-to-hyperscale-and-how-can-i-minimize-it"></a>Quelle est la durée du temps d’arrêt pendant la migration depuis un environnement local ou de machines virtuelles vers Hyperscale, et comment puis-je la minimiser ?

Le temps d’arrêt pour la migration vers Hyperscale est le même que quand vous migrez vos bases de données vers d’autres niveaux de service dans Azure SQL Database. Vous pouvez utiliser la [réplication transactionnelle](replication-to-sql-database.md#data-migration-scenario
) afin de réduire le temps d’arrêt dû à la migration pour les bases de données d’une taille allant jusqu’à plusieurs To. Pour de très grandes bases de données (plus de 10 To), vous pouvez envisager de migrer les données avec ADF, Spark ou d’autres technologies de déplacement des données.

### <a name="how-much-time-would-it-take-to-bring-in-x-amount-of-data-to-hyperscale"></a>Combien de temps cela prend-t-il de charger une quantité X de données dans Hyperscale ?

Hyperscale peut consommer 100 Mo/s de données nouvelles ou modifiées, mais le temps nécessaire pour déplacer des données dans des bases de données dans Azure SQL Database est également affecté par le débit du réseau disponible, la vitesse de lecture de la source et l’objectif de niveau de service de la base de données cible.

### <a name="can-i-read-data-from-blob-storage-and-do-fast-load-like-polybase-in-azure-synapse-analytics"></a>Est-ce que je peux lire les données d’un stockage d’objets blob (comme PolyBase dans Azure Synapse Analytics) et faire un chargement rapide ?

Vous pouvez faire en sorte qu’une application cliente lise des données depuis Stockage Azure et charger des données dans une base de données Hyperscale (tout comme vous pouvez le faire avec n’importe quelle autre base de données dans Azure SQL Database). Actuellement, PolyBase n’est pas pris en charge dans Azure SQL Database. Comme alternative pour fournir une charge rapide, vous pouvez utiliser [Azure Data Factory](../../data-factory/index.yml) ou utiliser un travail Spark dans [Azure Databricks](/azure/azure-databricks/) avec le [connecteur Spark pour SQL](spark-connector.md). Le connecteur Spark pour SQL prend en charge l’insertion en bloc.

Il est également possible de lire en bloc des données à partir du magasin d’objets blob Azure à l’aide de BULK INSERT ou OPENROWSET : [Exemples d’accès en bloc à des données dans Stockage Blob Azure](/sql/relational-databases/import-export/examples-of-bulk-access-to-data-in-azure-blob-storage?view=sql-server-2017#accessing-data-in-a-csv-file-referencing-an-azure-blob-storage-location).

Le modèle de récupération simple ou de journalisation en bloc n’est pas pris en charge dans Hyperscale. Le modèle de récupération complète est nécessaire pour fournir une haute disponibilité et la récupération jusqu`à une date et heure. Cependant, l’architecture de journalisation Hyperscale fournit un meilleur débit d’ingestion en comparaison avec d’autres niveaux de services Azure SQL Database.

### <a name="does-hyperscale-allow-provisioning-multiple-nodes-for-parallel-ingesting-of-large-amounts-of-data"></a>Hyperscale permet-il l’approvisionnement de plusieurs nœuds pour l’ingestion en parallèle de grandes quantités de données ?

Non. Hyperscale est une architecture à multitraitement symétrique (SMP) et n’est pas une architecture MMP (Massively Parallel Processing) ou une architecture multimaître. Vous pouvez créer plusieurs réplicas seulement pour effectuer un scale-out des charges de travail en lecture seule.

### <a name="what-is-the-oldest-sql-server-version-supported-for-migration-to-hyperscale"></a>Quelle est la version de SQL Server la plus ancienne prise en charge pour la migration vers Hyperscale ?

SQL Server 2005. Pour plus d’informations, consultez [Migrer vers une base de données unique ou une base de données mise en pool](migrate-to-database-from-sql-server.md#migrate-to-a-single-database-or-a-pooled-database). Pour les problèmes de compatibilité, consultez [Résolution des problèmes de compatibilité de la migration de base de données](migrate-to-database-from-sql-server.md#resolving-database-migration-compatibility-issues).

### <a name="does-hyperscale-support-migration-from-other-data-sources-such-as-amazon-aurora-mysql-postgresql-oracle-db2-and-other-database-platforms"></a>Hyperscale prend-t-il en charge la migration depuis d’autres sources de données, comme Amazon Aurora, MySQL, PostgreSQL, Oracle, DB2 et d’autres plateformes de base de données ?

Oui. [Azure Database Migration Service](../../dms/dms-overview.md) prend en charge de nombreux scénarios de migration.

## <a name="business-continuity-and-disaster-recovery-questions"></a>Questions sur la continuité des activités et la reprise d’activité

### <a name="what-slas-are-provided-for-a-hyperscale-database"></a>Quels sont les contrats SLA fournis pour une base de données Hyperscale ?

Voir [SLA pour Azure SQL Database](https://azure.microsoft.com/support/legal/sla/sql-database/v1_4/). Les réplicas de calcul secondaires supplémentaires augmentent la disponibilité, jusqu’à 99,99 % pour une base de données avec deux réplicas de calcul secondaires ou plus.

### <a name="are-the-database-backups-managed-for-me-by-azure-sql-database"></a>Les sauvegardes de base de données sont-elles gérées pour moi par Azure SQL Database ?

Oui.

### <a name="how-often-are-the-database-backups-taken"></a>Quelle est la fréquence des sauvegardes de base de données ?

Il n’existe pas de sauvegardes traditionnelles complètes, différentielles et de fichier journal pour les bases de données Hyperscale. Au lieu de cela, il existe des captures instantanées de stockage de fichiers de données. Le journal généré est simplement conservé en l’état pour la période de rétention configurée, ce qui permet une restauration à n’importe quel point dans le temps au cours de la période de rétention.

### <a name="does-hyperscale-support-point-in-time-restore"></a>Hyperscale prend-t-il en charge la limite de restauration dans le temps ?

Oui.

### <a name="what-is-the-recovery-point-objective-rporecovery-time-objective-rto-for-database-restore-in-hyperscale"></a>Quel est l’objectif de point de récupération (RPO)/objectif de délai de récupération (RTO) pour la restauration de base de données dans Hyperscale ?

Le RPO est de 0 min. La plupart des opérations de restauration se terminent en 60 minutes, quelle que soit la taille de la base de données. La durée de restauration peut être plus longue pour des bases de données plus volumineuses et si la base de données a fait l’objet d’une activité d’écriture importante avant et jusqu’au point de restauration dans le temps.

### <a name="does-database-backup-affect-compute-performance-on-my-primary-or-secondary-replicas"></a>La sauvegarde de base de données affecte-t-elle les performances de calcul sur mes réplicas principaux ou secondaires ?

Non. Les sauvegardes sont gérées par le sous-système de stockage et tirent parti des captures instantanées de stockage. Elles n’affectent pas les charges de travail utilisateur.

### <a name="can-i-perform-geo-restore-with-a-hyperscale-database"></a>Puis-je effectuer une géorestauration avec une base de données Hyperscale ?

Oui. La géo-restauration est entièrement prise en charge. Contrairement à la limite de restauration dans le temps, la géorestauration nécessite une opération à l’échelle des données. Les fichiers de données étant copiés en parallèle, la durée de cette opération dépend principalement de la taille du fichier le plus volumineux dans la base de données, plutôt que de la taille totale de celle-ci. La durée de la géorestauration est beaucoup plus courte si la base de données est restaurée dans la région Azure [associée](../../best-practices-availability-paired-regions.md) à la région de la base de données source.

### <a name="can-i-set-up-geo-replication-with-hyperscale-database"></a>Puis-je configurer la géoréplication avec une base de données Hyperscale ?

Pas pour l'instant.

### <a name="can-i-take-a-hyperscale-database-backup-and-restore-it-to-my-on-premises-server-or-on-sql-server-in-a-vm"></a>Puis-je sauvegarder une base de données Hyperscale et la restaurer sur mon serveur local ou sur SQL Server dans une machine virtuelle ?

Non. Le format de stockage pour les bases de données Hyperscale est différent des versions publiées de SQL Server, et vous ne contrôlez pas les sauvegardes et vous n’y avez pas accès. Pour extraire vos données d’une base de données Hyperscale, vous pouvez extraire les données à l’aide des technologies de déplacement de données, par exemple Azure Data Factory, Azure Databricks, SSIS, etc.

## <a name="cross-feature-questions"></a>Questions sur les fonctionnalités croisées

### <a name="do-i-lose-any-functionality-or-capabilities-after-migration-to-the-hyperscale-service-tier"></a>Est-ce que je perds des fonctionnalités ou des capacités après la migration vers le niveau de service Hyperscale ?

Oui. Certaines fonctionnalités Azure SQL Database ne sont pas prises en charge dans Hyperscale, notamment la conservation à long terme des sauvegardes. Après la migration de vos bases de données vers Hyperscale, ces fonctionnalités cessent de fonctionner.

### <a name="will-polybase-work-with-hyperscale"></a>PolyBase fonctionne-t-il avec Hyperscale ?

Non. PolyBase n’est pas pris en charge dans Azure SQL Database.

### <a name="does-hyperscale-have-support-for-r-and-python"></a>Hyperscale peut-il prendre en charge R et Python ?

Pas pour l'instant.

### <a name="are-compute-nodes-containerized"></a>Les nœuds de calcul sont-ils placés dans des conteneurs ?

Non. Les processus Hyperscale s’exécutent sur des nœuds [Service Fabric](https://azure.microsoft.com/services/service-fabric/) (machines virtuelles), et non dans des conteneurs.

## <a name="performance-questions"></a>Questions sur les performances

### <a name="how-much-write-throughput-can-i-push-in-a-hyperscale-database"></a>Quel débit d’écriture puis-je envoyer (push) dans une base de données Hyperscale ?

Le débit limite du journal des transactions est défini sur 100 Mo/s pour toute taille de calcul Hyperscale. La capacité à atteindre ce taux dépend de plusieurs facteurs, notamment le type de charge de travail, la configuration du client et la capacité de calcul suffisante sur le réplica de calcul principal pour produire le journal à ce rythme.

### <a name="how-many-iops-do-i-get-on-the-largest-compute"></a>Combien d’IOPS puis-je obteni sur le plus grand calcul ?

Les IOPS et la latence d’E/S varient en fonction des modèles de charge de travail. Si les données faisant l’objet d’un accès sont mises en cache sur le réplica de calcul, vous verrez les mêmes performances d’E/S qu’avec le disque SSD local.

### <a name="does-my-throughput-get-affected-by-backups"></a>Mon débit est-il affecté par les sauvegardes ?

Non. La capacité de calcul est découplée de la couche de stockage. Cela élimine l’impact sur les performances de la sauvegarde.

### <a name="does-my-throughput-get-affected-as-i-provision-additional-compute-replicas"></a>Mon débit est-il affecté quand je provisionne des réplicas de calcul supplémentaires ?

Comme le stockage est partagé et qu’aucune réplication physique directe n’est effectuée entre les réplicas de calcul principaux et secondaires, le débit sur le réplica principal n’est pas affecté directement par l’ajout de réplicas secondaires. Cependant, nous pouvons limiter la charge de travail en écriture agressive continue sur le réplica principal pour que le journal s’applique sur les réplicas secondaires et les serveurs de pages à rattraper, et éviter ainsi des performances de lecture médiocres sur les réplicas secondaires.

### <a name="how-do-i-diagnose-and-troubleshoot-performance-problems-in-a-hyperscale-database"></a>Comment faire pour diagnostiquer et résoudre les problèmes de performances dans une base de données Hyperscale ?

Pour la plupart des problèmes de performances, en particulier ceux qui ne proviennent pas des performances de stockage, les étapes courantes de diagnostic et de résolution des problèmes SQL s’appliquent. Pour obtenir des diagnostics de stockage spécifiques à Hyperscale, consultez [Diagnostics de résolution des problèmes de performances Hyperscale SQL](hyperscale-performance-diagnostics.md).

## <a name="scalability-questions"></a>Questions sur la scalabilité

### <a name="how-long-would-it-take-to-scale-up-and-down-a-compute-replica"></a>Quel est le temps nécessaire pour un scale-up et un scale-down d’un réplica de calcul ?

Monter ou descendre le calcul en puissance doit prendre 2 minutes, quelle que soit la taille des données.

### <a name="is-my-database-offline-while-the-scaling-updown-operation-is-in-progress"></a>Ma base de données est-elle hors connexion pendant l’exécution de l’opération de scale-up/down ?

Non. Le scale-up/down se fait en ligne.

### <a name="should-i-expect-connection-drop-when-the-scaling-operations-are-in-progress"></a>Dois-je m’attendre à la suppression des connexions quand les opérations de mise à l’échelle sont en cours ?

Le scale-up ou le scale-down entraînent la suppression des connexions existantes quand un basculement se produit à la fin de l’opération de mise à l'échelle. L’ajout de réplicas secondaires n’entraîne pas la suppression des connexions.

### <a name="is-the-scaling-up-and-down-of-compute-replicas-automatic-or-end-user-triggered-operation"></a>Le scale-up et le scale-down de réplicas de calcul sont-ils des opérations automatiques ou déclenchées par l’utilisateur final ?

Utilisateur final. Pas automatique.  

### <a name="does-the-size-of-my-tempdb-database-and-rbpex-cache-also-grow-as-the-compute-is-scaled-up"></a>La taille de ma base de données `tempdb` et du cache RBPEX augmente-t-elle également au fur et à mesure que le calcul est mis à l’échelle ?

Oui. La taille de la base de données `tempdb` et du [cache RBPEX](service-tier-hyperscale.md#distributed-functions-architecture) sur les nœuds de calcul augmente automatiquement à mesure que le nombre de cœurs augmente.

### <a name="can-i-provision-multiple-primary-compute-replicas-such-as-a-multi-master-system-where-multiple-primary-compute-heads-can-drive-a-higher-level-of-concurrency"></a>Puis-je approvisionner plusieurs réplicas de calcul principaux, comme un système multimaître où plusieurs têtes de calcul principales peuvent gérer un niveau de concurrence plus élevé ?

Non. Seul le réplica de calcul principal accepte les demandes de lecture/écriture. Les réplicas de calcul secondaires acceptent seulement les demandes en lecture seule.

## <a name="read-scale-out-questions"></a>Questions sur l’échelle horizontale en lecture

### <a name="how-many-secondary-compute-replicas-can-i-provision"></a>Combien de réplicas de calcul secondaires puis-je approvisionner ?

Nous créons par défaut un réplica secondaire pour les bases de données Hyperscale. Si vous souhaitez ajuster le nombre de réplicas, vous pouvez le faire au moyen du [Portail Microsoft Azure](https://portal.azure.com) ou de l’[API REST](/rest/api/sql/databases/createorupdate).

### <a name="how-do-i-connect-to-these-secondary-compute-replicas"></a>Comment se connecter à ces réplicas de calcul secondaires ?

Vous pouvez vous connecter à ces réplicas de calcul supplémentaires en lecture seule en définissant l’argument `ApplicationIntent` de votre chaîne de connexion sur `ReadOnly`. Les connexions marquées avec `ReadOnly` sont automatiquement routées vers un des réplicas de calcul supplémentaires en lecture seule. Pour plus d’informations, consultez [Utiliser des réplicas en lecture seule pour décharger des charges de travail de requêtes en lecture seule](read-scale-out.md).

### <a name="how-do-i-validate-if-i-have-successfully-connected-to-secondary-compute-replica-using-ssms-or-other-client-tools"></a>Comment faire pour vérifier si la connexion au réplica de calcul secondaire a bien été établie à l’aide de SSMS ou d’autres outils clients ?

Vous pouvez exécuter la requête T-SQL suivante : `SELECT DATABASEPROPERTYEX ('<database_name>', 'Updateability')`.
Le résultat est `READ_ONLY` si vous êtes connecté à un réplica secondaire en lecture seule, et `READ_WRITE` si vous êtes connecté au réplica principal. Notez que le contexte de base de données doit être défini sur le nom de la base de données Hyperscale, et non sur la base de données `master`.

### <a name="can-i-create-a-dedicated-endpoint-for-a-read-scale-out-replica"></a>Puis-je créer un point de terminaison dédié pour le réplica avec une échelle horizontale en lecture ?

Non. Vous pouvez vous connecter au réplica avec une échelle horizontale en lecture uniquement en spécifiant `ApplicationIntent=ReadOnly`.

### <a name="does-the-system-do-intelligent-load-balancing-of-the-read-workload"></a>Est-ce que le système effectue un équilibrage de charge intelligent de la charge de travail de lecture ?

Non. Une nouvelle connexion avec intention de lecture seule est redirigée vers un réplica de scale-out de lecture arbitraire.

### <a name="can-i-scale-updown-the-secondary-compute-replicas-independently-of-the-primary-replica"></a>Puis-je effectuer un scale-up/down des réplicas de calcul secondaires indépendamment du réplica principal ?

Non. Les réplicas de calcul secondaires sont également utilisés comme cibles de basculement à haute disponibilité. Ils doivent donc avoir la même configuration que le principal pour fournir les performances attendues après le basculement.

### <a name="do-i-get-different-tempdb-sizing-for-my-primary-compute-and-my-additional-secondary-compute-replicas"></a>Le dimensionnement de `tempdb` est-il différent pour ma capacité de calcul principale et pour mes réplicas de calcul secondaires supplémentaires ?

Non. Votre base de données `tempdb` est configurée en fonction de l’approvisionnement de la taille de calcul : vos réplicas de calcul secondaires ont la même taille que la capacité de calcul principale.

### <a name="can-i-add-indexes-and-views-on-my-secondary-compute-replicas"></a>Puis-je ajouter des index et des vues sur mes réplicas de calcul secondaires ?

Non. Les bases de données Hyperscale ont un stockage partagé, ce qui signifie que tous les réplicas de calcul voient les mêmes tables, les mêmes index et les mêmes vues. Si vous voulez des index supplémentaires optimisés pour les lectures sur les réplicas secondaires, vous devez d’abord les ajouter sur le réplica principal.

### <a name="how-much-delay-is-there-going-to-be-between-the-primary-and-secondary-compute-replicas"></a>Quel est le décalage entre le réplica de calcul principal et le réplica de calcul secondaire ?

La latence des données entre le moment où une transaction est validée sur le réplica principal et le moment où elle est lisible sur un réplica secondaire dépend de la vitesse de génération de journal actuelle, de la taille de la transaction, de la charge sur le réplica et d’autres facteurs. La latence des données standard pour les petites transactions est de l’ordre de dizaines de millisecondes, mais il n’y a pas de limite supérieure à la latence des données. Les données situées sur un réplica secondaire donné sont toujours cohérentes au niveau transactionnel. Toutefois, à un moment donné, la latence des données peut être différente pour différents réplicas secondaires. Les charges de travail qui doivent lire les données validées immédiatement doivent s’exécuter sur le réplica principal.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le niveau de service Hyperscale, consultez [Niveau de service Hyperscale](service-tier-hyperscale.md).