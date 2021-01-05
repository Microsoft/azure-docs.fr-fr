---
title: Améliorer les performances des applications Azure avec Advisor
description: Utilisez les recommandations d’Azure Advisor en matière de performances pour optimiser la vitesse et la réactivité de vos applications stratégiques.
ms.topic: article
ms.date: 07/29/2020
ms.openlocfilehash: 55f5ac6784bf613170fd10060799ab5ad1290a62
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183343"
---
# <a name="improve-the-performance-of-azure-applications-by-using-azure-advisor"></a>Améliorer les performances des applications Azure à l’aide d’Azure Advisor

Les recommandations d’Azure Advisor en matière de performances peuvent contribuer à améliorer la vitesse et la réactivité de vos applications stratégiques. Vous pouvez obtenir des recommandations en matière de performances à l’aide d’Advisor dans l’onglet **Performances** du tableau de bord d’Advisor.

## <a name="reduce-dns-time-to-live-on-your-traffic-manager-profile-to-fail-over-to-healthy-endpoints-faster"></a>Diminuer la durée de vie DNS dans votre profil Traffic Manager pour basculer plus rapidement vers des points de terminaison sains

Vous pouvez utiliser les [paramètres TTL (durée de vie)](../traffic-manager/traffic-manager-performance-considerations.md) sur votre profil Azure Traffic Manager pour spécifier l’intervalle de temps nécessaire avant de changer de point de terminaison lorsque celui-ci ne répond plus. Si vous réduisez les valeurs TTL, les clients sont redirigés plus rapidement vers des points de terminaison opérationnels.

Azure Advisor identifie les profils Traffic Manager dont la TTL configurée est plus longue. Il recommande de la régler à 20 secondes ou 60 secondes selon que le profil est configuré pour [Bascule rapide](https://azure.microsoft.com/roadmap/fast-failover-and-tcp-probing-in-azure-traffic-manager/).

## <a name="improve-database-performance-by-using-sql-database-advisor-temporarily-disabled"></a>Améliorer les performances de la base de données à l’aide de SQL Database Advisor (temporairement désactivé)

Azure Advisor offre une vue cohérente et consolidée des recommandations pour toutes vos ressources Azure. Il s’intègre à SQL Database Advisor pour vous proposer des recommandations en vue d’améliorer les performances de vos bases de données.  SQL Database Advisor évalue les performances de vos bases de données en analysant votre historique d'utilisation. Il propose alors les recommandations les plus adaptées pour exécuter la charge de travail standard de la base de données.

> [!NOTE]
> Avant de pouvoir obtenir des recommandations, votre base de données doit être utilisée pendant environ une semaine et, au cours de cette semaine, une certaine activité doit être assurée de manière cohérente. SQL Database Advisor peut plus facilement optimiser les modèles de requête cohérents que les pics d’activité aléatoires.

Pour plus d’informations, consultez [SQL Database Advisor](../azure-sql/database/database-advisor-implement-performance-recommendations.md).

## <a name="upgrade-your-storage-client-library-to-the-latest-version-for-better-reliability-and-performance"></a>Mettre à niveau votre bibliothèque de client de stockage vers la version la plus récente afin d’optimiser la fiabilité et les performances

La version la plus récente du Kit de développement logiciel (SDK) de la bibliothèque de client de stockage contient des correctifs pour les problèmes signalés par les clients et identifiés de manière proactive par notre processus d’assurance qualité. La dernière version offre également une optimisation de la fiabilité et des performances, ainsi que de nouvelles fonctionnalités qui peuvent améliorer votre expérience globale d’utilisation de Stockage Azure. Advisor fournit des recommandations ainsi que les étapes à suivre pour effectuer une mise à niveau vers la dernière version du Kit de développement logiciel (SDK) si vous utilisez une version obsolète. Les recommandations concernent les langages pris en charge : C++ et .NET.

## <a name="improve-app-service-performance-and-reliability"></a>Améliorer la fiabilité et les performances d’App Service

Azure Advisor intègre des recommandations visant à améliorer votre expérience App Services et à vous faire découvrir des fonctionnalités appropriées de la plateforme. Exemples de recommandations App Service :
* Détection des instances où les ressources de mémoire ou de processeur sont épuisées par des runtimes d’application, avec options d’atténuation.
* Détection des instances où la colocalisation de ressources telles que les applications web et les bases de données peut améliorer les performances et réduire les coûts.

Pour plus d’informations, consultez [Meilleures pratiques pour Azure App Service](../app-service/app-service-best-practices.md).

## <a name="use-managed-disks-to-prevent-disk-io-throttling"></a>Utiliser des disques managés pour éviter la limitation des E/S de disque

Advisor identifie les machines virtuelles appartenant à un compte de stockage qui atteint sa cible de scalabilité. Cette condition rend les machines virtuelles sensibles à la limitation des E/S. Advisor recommande l’utilisation de disques managés pour prévenir toute détérioration des performances.

## <a name="improve-the-performance-and-reliability-of-virtual-machine-disks-by-using-premium-storage"></a>Améliorer les performances et la fiabilité des disques de machine virtuelle en utilisant le stockage Premium

Advisor identifie les machines virtuelles dont les disques standard ont un volume de transactions élevé sur votre compte de stockage. Il recommande la mise à niveau vers des disques Premium. 

Azure Premium Storage offre une prise en charge très performante et à faible latence des disques pour les machines virtuelles exécutant des charges de travail qui utilisent beaucoup d'E/S. Les disques de machine virtuelle qui utilisent des comptes Stockage Premium stockent les données sur des disques SSD. Pour que votre application bénéficie de performances optimales, nous vous recommandons de migrer les disques de machine virtuelle nécessitant un nombre élevé d’IOPS vers Stockage Premium.

## <a name="remove-data-skew-on-your-azure-synapse-analytics-tables-to-increase-query-performance"></a>Supprimer l’asymétrie des données sur les tables Azure Synapse Analytics pour augmenter les performances de requête

L’asymétrie des données peut provoquer des déplacements de données ou des goulots d’étranglement de ressource inutiles lors de l’exécution de votre charge de travail. Advisor détecte l’asymétrie des données dont la distribution est supérieure à 15 %. Il vous recommande de redistribuer vos données et de revoir vos sélections des clés de distribution des tables. Pour en savoir plus sur l’identification et la suppression d’une asymétrie, consultez [résolution des problèmes d’asymétrie](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-distribute.md#how-to-tell-if-your-distribution-column-is-a-good-choice).

## <a name="create-or-update-outdated-table-statistics-in-your-azure-synapse-analytics-tables-to-increase-query-performance"></a>Créer ou mettre à jour les statistiques de table obsolètes sur votre table Azure Synapse Analytics pour augmenter les performances de requête

Advisor identifie les tables qui n’ont pas de [statistiques de table](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-statistics.md) à jour et recommande la création ou la mise à jour des statistiques. L’optimiseur de requête d’Azure Synapse Analytics utilise des statistiques à jour pour estimer la cardinalité ou le nombre de lignes dans les résultats de la requête. Ces estimations permettent à l’optimiseur de requête de créer un plan de requête pour fournir des performances plus rapides.

## <a name="improve-mysql-connection-management"></a>Améliorer la gestion des connexions MySQL

L’analyse d’Advisor peut indiquer que votre application qui se connecte à un serveur MySQL ne gère peut-être pas efficacement les connexions. Cette condition peut entraîner une consommation inutile des ressources et une latence globale élevée des applications. Pour améliorer la gestion des connexions, nous vous recommandons de réduire le nombre de connexions à courte durée de vie et d’éliminer les connexions inactives non nécessaires. Vous pouvez apporter ces améliorations en configurant un regroupement de connexions côté serveur, comme ProxySQL.


## <a name="scale-up-to-optimize-cache-utilization-on-your-azure-synapse-analytics-tables-to-increase-query-performance"></a>Effectuer un scale-up pour optimiser l’utilisation du cache sur vos tables Azure Synapse Analytics et ainsi améliorer les performances de requête

Azure Advisor détecte si vos tables Azure Synapse Analytics ont un pourcentage élevé d’utilisation du cache et un faible pourcentage d’accès. Cette condition indique une éviction élevée du cache, ce qui peut avoir une incidence sur les performances de votre instance Azure Synapse Analytics. Advisor vous recommande d’effectuer un scale-up de votre instance Azure Synapse Analytics pour garantir l’allocation d’une capacité de cache suffisante pour votre charge de travail.

## <a name="convert-azure-synapse-analytics-tables-to-replicated-tables-to-increase-query-performance"></a>Convertir les tables Azure Synapse Analytics en tables répliquées pour augmenter les performances de requête

Advisor identifie les tables qui ne sont pas des tables répliquées, mais qui tirent parti de la conversion. Il vous suggère de convertir ces tables. Les recommandations sont basées sur les éléments suivants :
- Taille de la table répliquée. 
- Nombre de colonnes. 
- Type de distribution de tables. 
- Nombre de partitions sur la table Azure Synapse Analytics. 

Une méthode heuristique supplémentaire peut être fournie dans la recommandation relative au contexte. Pour en savoir plus sur la détermination de cette recommandation, consultez [Recommandations Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-concept-recommendations.md#replicate-tables). 

## <a name="migrate-your-storage-account-to-azure-resource-manager-to-get-the-latest-azure-features"></a>Migrer votre compte de stockage vers Azure Resource Manager pour obtenir les fonctionnalités Azure les plus récentes

Migrez votre modèle de déploiement de compte de stockage vers Azure Resource Manager pour tirer parti des éléments suivants :
- Déploiements de modèles.
- Options de sécurité supplémentaires.
- Possibilité de mettre à niveau vers un compte GPv2 pour que vous puissiez utiliser les fonctionnalités Stockage Azure les plus récentes. 

Advisor identifie les comptes de stockage autonomes qui utilisent le modèle de déploiement classique et recommande une migration vers le modèle de déploiement Resource Manager.

> [!NOTE]
> Les alertes classiques d’Azure Monitor ont été mises hors service en août 2019. Nous vous recommandons de mettre à niveau votre compte de stockage classique pour utiliser Resource Manager afin de conserver la fonctionnalité d'alerte sur la nouvelle plateforme. Pour plus d’informations, consultez [Mise hors service des alertes classiques](../azure-monitor/platform/monitoring-classic-retirement.md#retirement-of-classic-monitoring-and-alerting-platform).

## <a name="design-your-storage-accounts-to-prevent-reaching-the-maximum-subscription-limit"></a>Créer vos comptes de stockage de manière à éviter d’atteindre le nombre maximum d’abonnements

Une région Azure prend en charge un maximum de 250 comptes de stockage par abonnement. Une fois cette limite atteinte, vous ne pourrez pas créer d’autres comptes de stockage dans cette combinaison abonnement/région. Advisor vérifie vos abonnements, puis fournit des recommandations pour vous aider à créer moins de comptes de stockage pour une combinaison abonnement/région qui est proche de la limite maximale.

## <a name="consider-increasing-the-size-of-your-vpn-gateway-sku-to-address-high-p2s-use"></a>Envisager d’agrandir la taille de la référence SKU de passerelle VPN pour faire face à une forte utilisation P2S

Chaque référence SKU de passerelle VPN Azure ne peut prendre en charge qu’un nombre spécifié de connexions P2S simultanées. Si le nombre de connexions est proche de la limite de votre passerelle, les tentatives de connexion suivantes peuvent échouer. Si vous augmentez la taille de votre passerelle, vous pouvez prendre en charge davantage d’utilisateurs P2S simultanés. Advisor fournit des recommandations et des instructions visant à augmenter la taille de votre passerelle.

## <a name="consider-increasing-the-size-of-your-vpn-gateway-sku-to-address-high-cpu"></a>Envisager d’agrandir la taille de la référence SKU de passerelle VPN pour faire face à une forte sollicitation du processeur

Dans une charge de trafic élevée, votre passerelle VPN peut supprimer des paquets en raison de la forte sollicitation du processeur. Envisagez de mettre à niveau votre référence SKU de passerelle VPN. Une taille de passerelle VPN accrue garantira que les connexions ne seront pas supprimées en raison d’une forte sollicitation du processeur. Advisor fournit des recommandations pour traiter ce problème de manière proactive. 

## <a name="increase-batch-size-when-loading-to-maximize-load-throughput-data-compression-and-query-performance"></a>Agrandissez la taille de lot pendant le chargement pour optimiser le débit de charge, la compression des données et le niveau de performance des requêtes

Advisor détecte si vous pouvez améliorer les performances et le débit de la charge en agrandissant la taille de lot pendant le chargement dans votre base de données. Vous pouvez envisager d’utiliser l’instruction COPY. Si vous ne pouvez pas utiliser l’instruction COPY, envisagez d’augmenter la taille de lot quand vous utilisez des utilitaires de chargement comme l’API SQLBulkCopy ou BCP. Une bonne règle générale consiste à utiliser une taille de lot comprise entre 100 000 et 1 million de lignes. L’augmentation de la taille de lot augmentera le débit de charge, la compression des données et les performances des requêtes.

## <a name="co-locate-the-storage-account-in-the-same-region-to-minimize-latency-when-loading"></a>Colocaliser le compte de stockage dans la même région pour réduire la latence pendant le chargement

Advisor détecte si vous effectuez le chargement à partir d’une région différente de celle de votre pool SQL dédié. Envisagez de charger à partir d’un compte de stockage qui se trouve dans la même région que votre pool SQL dédié pour réduire la latence lors du chargement des données. Cette modification permettra de réduire la latence et d’améliorer les performances de charge.

## <a name="use-a-supported-kubernetes-version"></a>Utiliser une version de Kubernetes prise en charge

Advisor détecte les versions non prises en charge de Kubernetes.

## <a name="optimize-the-performance-of-your-azure-database-for-mysql-azure-database-for-postgresql-and-azure-database-for-mariadb-servers"></a>Optimiser les performances des serveurs Azure Database for MariaDB, Azure Database pour MySQL et Azure Database pour PostgreSQL

### <a name="fix-the-cpu-pressure-of-your-azure-database-for-mysql-azure-database-for-postgresql-and-azure-database-for-mariadb-servers-with-cpu-bottlenecks"></a>Corriger la sollicitation du processeur par les serveurs Azure Database for MariaDB, Azure Database pour MySQL et Azure Database pour PostgreSQL à l’aide de goulots d’étranglement de processeur
Une utilisation intense du processeur pendant une période prolongée peut ralentir les performances des requêtes pour votre charge de travail. L’augmentation de la taille du processeur aide à optimiser l’exécution des requêtes de base de données, ainsi qu’à améliorer les performances globales. Advisor identifie les serveurs faisant une utilisation intense de processeur qui exécutent probablement des charges de travail limitées par celui-ci et recommande une mise à l’échelle de votre capacité de calcul.

### <a name="reduce-memory-constraints-on-your-azure-database-for-mysql-azure-database-for-postgresql-and-azure-database-for-mariadb-servers-or-move-to-a-memory-optimized-sku"></a>Réduire les contraintes de mémoire sur les serveurs Azure Database for MariaDB, Azure Database pour MySQL et Azure Database pour PostgreSQL ou passer à une référence SKU à mémoire optimisée
Un taux d’accès au cache faible peut ralentir les performances de requête et augmenter les IOPS. Cette condition peut être due à un mauvais plan de requête ou à une charge de travail gourmande en mémoire. Une correction du plan de requête ou une [augmentation de la mémoire](../postgresql/concepts-pricing-tiers.md) du serveur Azure Database for MariaDB, Azure Database pour MySQL ou Azure Database pour PostgreSQL permet d’optimiser l’exécution de la charge de travail de la base de données. Azure Advisor identifie les serveurs concernés par cette forte attrition du pool de mémoires tampons. Il vous recommande d’effectuer l’une des actions suivantes : 
- Corriger le plan de requête
- Passer à une référence SKU dotée de plus de mémoire 
- Augmentez la taille de stockage pour obtenir davantage d’IOPS.

### <a name="use-an-azure-database-for-mysql-or-azure-database-for-postgresql-read-replica-to-scale-out-reads-for-read-intensive-workloads"></a>Utiliser un réplica en lecture Azure Database pour MySQL ou Azure Database pour PostgreSQL afin d’effectuer un scale-out des charges de travail dont les accès en lecture sont intensifs
Advisor utilise des méthodes heuristiques basées sur la charge de travail, telles que le ratio lectures/écritures sur le serveur au cours des sept derniers jours, pour identifier les charges de travail dont les accès en lecture sont intensifs. Une ressource Azure Database pour PostgreSQL ou Azure Database pour MySQL avec un ratio lectures/écritures élevé peut entraîner des contentions de processeur ou de mémoire et un ralentissement des performances de requête. L’ajout d’un [réplica](../postgresql/howto-read-replicas-portal.md) vous aide à effectuer un scale-out des lectures vers le serveur réplica et à éviter les contraintes d’UC ou de mémoire sur le serveur principal. Advisor identifie les serveurs traitant des charges de travail dont les accès en lectures sont intensifs et vous recommande d’ajouter un [réplica en lecture](../postgresql/concepts-read-replicas.md) pour alléger certaines charges de travail en lecture.


### <a name="scale-your-azure-database-for-mysql-azure-database-for-postgresql-or-azure-database-for-mariadb-server-to-a-higher-sku-to-prevent-connection-constraints"></a>Mettre à l’échelle le serveur Azure Database pour MySQL, Azure Database pour PostgreSQL ou Azure Database for MariaDB vers une référence SKU de niveau supérieur afin d’empêcher les contraintes de connexion
Chaque nouvelle connexion à votre serveur de base de données monopolise de la mémoire. Les performances du serveur de base de données se dégradent si les connexions à votre serveur échouent en raison d’une [limite supérieure](../postgresql/concepts-limits.md) de la mémoire. Azure Advisor identifie les serveurs qui rencontrent un grand nombre d’échecs de connexion. Il recommande de mettre à niveau les limites de connexion de votre serveur pour lui fournir plus de mémoire en effectuant l’une des actions suivantes :
- Effectuer un scale-up de la capacité de calcul. 
- Utiliser des références SKU à mémoire optimisée, qui ont davantage de capacité de calcul par cœur.

## <a name="scale-your-cache-to-a-different-size-or-sku-to-improve-cache-and-application-performance"></a>Mettre à l’échelle le cache à une taille ou à une référence SKU différentes afin d’améliorer les performances du cache et de l’application

Les instances de cache fonctionnent mieux lorsqu’elles ne sont pas soumises à une forte sollicitation de la mémoire, à une charge importante du serveur ou à une bande passante réseau élevée. Ces conditions peuvent entraîner une non-réponse, une perte de données ou une indisponibilité. Advisor identifie les instances de cache dans ces conditions. Il vous recommande d’effectuer l’une des actions suivantes :
- Appliquez les meilleures pratiques pour réduire la sollicitation de la mémoire, la charge du serveur ou la bande passante réseau.
- Mettez à l’échelle à une autre taille ou référence SKU ayant plus de capacité.

## <a name="add-regions-with-traffic-to-your-azure-cosmos-db-account"></a>Ajouter des régions avec du trafic vers votre compte Azure Cosmos DB

Advisor détecte les comptes Azure Cosmos DB recevant du trafic en provenance d’une région non configurée. Il recommande l’ajout de cette région. Cela a pour effet d’améliorer la latence des requêtes en provenance de cette région et de garantir la disponibilité en cas de panne de la région. [En savoir plus sur la distribution de données mondiale avec Azure Cosmos DB.](../cosmos-db/distribute-data-globally.md)

## <a name="configure-your-azure-cosmos-db-indexing-policy-by-using-custom-included-or-excluded-paths"></a>Configurer la stratégie d’indexation Azure Cosmos DB avec des chemins inclus ou exclus personnalisés

Advisor identifie les conteneurs Azure Cosmos DB qui utilisent la stratégie d’indexation par défaut, mais qui pourraient bénéficier d’une stratégie d’indexation personnalisée. Cette détermination est basée sur le modèle de charge de travail. La stratégie d’indexation par défaut indexe toutes les propriétés. Une stratégie d’indexation personnalisée avec des chemins inclus ou exclus explicites utilisés dans les filtres de requêtes peut réduire les unités de requête et le stockage utilisés pour l’indexation. [En savoir plus sur la modification des stratégies d’indexation.](../cosmos-db/index-policy.md)

## <a name="set-your-azure-cosmos-db-query-page-size-maxitemcount-to--1"></a>Définir la taille de page de la requête Azure Cosmos DB (MaxItemCount) avec la valeur -1 

Azure Advisor identifie les conteneurs Azure Cosmos DB qui utilisent une taille de page de requête de 100. Il recommande d’utiliser une taille de page de -1 pour des analyses plus rapides. [En savoir plus sur MaxItemCount.](../cosmos-db/sql-api-query-metrics.md)

## <a name="consider-using-accelerated-writes-feature-in-your-hbase-cluster-to-improve-cluster-performance"></a>Envisagez d’utiliser la fonctionnalité Écritures accélérées dans votre cluster HBase pour améliorer les performances du cluster
Azure Advisor analyse les journaux système des 7 derniers jours et identifie si votre cluster a rencontré les scénarios suivants :
1. Latence élevée de la durée de synchronisation du journal WAL 
2. Nombre élevé de demandes d’écriture (au moins 3 fenêtres d’une heure de plus de 1000 demandes d’écriture moyennes par seconde et par nœud)

Ces conditions sont des indicateurs que votre cluster subit des latences d’écriture élevées. Cela peut être dû à une charge de travail importante effectuée sur votre cluster. Pour améliorer les performances de votre cluster, vous pouvez envisager d’utiliser la fonctionnalité Écritures accélérées fournie par Azure HDInsight HBase. La fonctionnalité Écritures accélérées pour les clusters HDInsight Apache HBase associe des disques SSD managés Premium à chaque serveur de région (nœud Worker) au lieu d’utiliser le stockage cloud. Ainsi, elle offre une faible latence en écriture et une meilleure résilience pour vos applications. Pour plus d’informations sur cette fonctionnalité, cliquez sur [En savoir plus](../hdinsight/hbase/apache-hbase-accelerated-writes.md#how-to-enable-accelerated-writes-for-hbase-in-hdinsight)

## <a name="review-azure-data-explorer-table-cache-period-policy-for-better-performance-preview"></a>Modifier la période de mise en cache des tables Azure Data Explorer (stratégie) pour de meilleures performances (préversion)
Cette recommandation expose les tables Azure Data Explorer ayant un grand nombre de requêtes qui dépassent la période de mise en cache configurée (stratégie) (vous verrez les 10 tables qui accèdent le plus aux données hors cache, en pourcentage de requêtes). L’action recommandée pour améliorer les performances du cluster est la suivante : Limitez au maximum les requêtes envoyées à cette table (tout en restant dans l’intervalle de temps défini par la stratégie). Si l’intégralité des données de l’intervalle de temps sont nécessaires, vous pouvez également augmenter la période de mise en cache vers la valeur recommandée.

## <a name="improve-performance-by-optimizing-mysql-temporary-table-sizing"></a>Améliorer le niveau de performance en optimisant le dimensionnement des tables temporaires MySQL
L’analyse du conseiller interne indique que votre serveur MySQL peut entraîner une surcharge d’E/S inutile en raison de paramètres insuffisants au niveau des tables temporaires. Cela peut générer des transactions sur disque inutiles et une réduction du niveau de performance. Nous vous recommandons d’augmenter les valeurs des paramètres « tmp_table_size » et « max_heap_table_size » pour réduire le nombre de transactions sur disque. [En savoir plus](https://aka.ms/azure_mysql_tmp_table)

## <a name="distribute-data-in-server-group-to-distribute-workload-among-nodes"></a>Distribuer des données dans un groupe de serveurs pour répartir la charge de travail entre les nœuds
Advisor identifie les groupes de serveurs sur lesquels les données n’ont pas été distribuées mais restent sur le coordinateur. Pour bénéficier de tous les avantages offerts par Hyperscale (Citus), Advisor vous recommande de distribuer les données sur les nœuds worker de vos groupes de serveurs. Cela améliorera les performances des requêtes en utilisant la ressource de chaque nœud dans le groupe de serveurs. [En savoir plus](https://go.microsoft.com/fwlink/?linkid=2135201) 

## <a name="improve-user-experience-and-connectivity-by-deploying-vms-closer-to-windows-virtual-desktop-deployment-location"></a>Améliorer l’expérience utilisateur et la connectivité en déployant des machines virtuelles plus près de l’emplacement de déploiement de Windows Virtual Desktop
Nous avons déterminé que vos machines virtuelles se trouvaient dans une région différente ou loin de celle à partir duquel vos utilisateurs se connectent à l’aide de Windows Virtual Desktop (WVD). Cela peut allonger les temps de réponse de connexion et affecter l’expérience utilisateur globale sur WVD. Lors de la création de machines virtuelles pour vos pools d’hôtes, vous devez essayer d’utiliser une région plus proche de l’utilisateur. Une proximité étroite garantit une satisfaction continue avec le service WVD et une meilleure qualité d’expérience globale. [En savoir plus sur la latence de connexion](../virtual-desktop/connection-latency.md).

## <a name="upgrade-to-the-latest-version-of-the-immersive-reader-sdk"></a>Effectuer une mise à niveau vers la dernière version du SDK du Lecteur immersif
Nous avons identifié des ressources dans cet abonnement qui utilisent des versions obsolètes du SDK du Lecteur immersif. L’utilisation de la dernière version du SDK du Lecteur immersif vous offre une sécurité et des performances mises à jour ainsi qu’un ensemble étendu de fonctionnalités permettant de personnaliser et d’améliorer votre expérience d’intégration.
En savoir plus sur le [kit SDK Lecteur immersif](../cognitive-services/immersive-reader/index.yml).


## <a name="how-to-access-performance-recommendations-in-advisor"></a>Comment accéder aux recommandations en matière de performances dans Advisor

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis ouvrez [Advisor](https://aka.ms/azureadvisordashboard).

2.  Dans le tableau de bord Advisor, sélectionnez l’onglet **Performances**.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les recommandations d’Advisor, consultez les ressources suivantes :

* [Présentation du conseiller](advisor-overview.md)
* [Prise en main d’Advisor](advisor-get-started.md)
* [Score Advisor](azure-advisor-score.md)
* [Recommandations d’Advisor en matière de coûts](advisor-cost-recommendations.md)
* [Recommandations d’Advisor en matière de fiabilité](advisor-high-availability-recommendations.md)
* [Recommandations d’Advisor en matière de sécurité](advisor-security-recommendations.md)
* [Recommandations d’Advisor en matière d’excellence opérationnelle](advisor-operational-excellence-recommendations.md)
* [API REST Advisor](/rest/api/advisor/)