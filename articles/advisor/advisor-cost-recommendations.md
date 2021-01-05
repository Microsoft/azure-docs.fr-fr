---
title: Réduire les coûts de service grâce à Azure Advisor
description: Le conseiller Azure permet d’optimiser le coût de vos déploiements Azure.
ms.topic: article
ms.date: 09/27/2020
ms.openlocfilehash: 767c82c818535304552c1c98ff50a252acc0d08a
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96458511"
---
# <a name="reduce-service-costs-by-using-azure-advisor"></a>Réduire les coûts de service à l’aide d’Azure Advisor

Azure Advisor vous aide à optimiser et à réduire votre dépense Azure globale en identifiant les ressources inactives et sous-utilisées.  Vous pouvez obtenir des recommandations en matière de coûts dans l’onglet **Coût** du tableau de bord Advisor.

## <a name="optimize-virtual-machine-spend-by-resizing-or-shutting-down-underutilized-instances"></a>Optimiser le coût de la machine virtuelle en redimensionnant ou en arrêtant les instances sous-utilisées 

Alors que certains scénarios d’application peuvent par définition entraîner une faible utilisation, vous pouvez souvent faire des économies grâce à la gestion de la taille de vos machines virtuelles et de leur nombre. 

Les actions recommandées sont l’arrêt ou le redimensionnement, en fonction de la ressource évaluée.

Dans Advisor, le modèle d’évaluation avancée envisage l’arrêt des machines virtuelles lorsque toutes ces affirmations sont vraies : 
- Le 95e centile de la valeur maximale de l’utilisation du processeur est inférieur à 3 %. 
- L’utilisation du réseau est inférieure à 2 % pendant une période de 7 jours.
- La sollicitation de la mémoire est inférieure aux valeurs de seuil.

Advisor envisage le redimensionnement des machines virtuelles lorsqu’il est possible de faire tenir la charge actuelle dans une référence SKU plus petite (au sein de la même famille de références SKU) ou dans un plus petit nombre d’instances de sorte que :
- La charge actuelle ne dépasse pas 80 % d’utilisation pour les charges de travail auxquelles l’utilisateur n’est pas confronté. 
- La charge ne dépasse pas 40 % pour les charges de travail accessibles aux utilisateurs. 

Ici, Advisor détermine le type de charge de travail en analysant les caractéristiques d’utilisation du processeur de la charge de travail.

Advisor indique une estimation des économies réalisables avec l’une ou l’autre des actions recommandées : redimensionnement ou arrêt. Pour le redimensionnement, Advisor fournit des informations sur les références SKU actuelles et cibles.

Si vous souhaitez être plus « agressif » dans l’identification des machines virtuelles sous-utilisées, vous pouvez ajuster la règle d’utilisation du processeur par abonnement.

## <a name="optimize-spend-for-mariadb-mysql-and-postgresql-servers-by-right-sizing"></a>Optimiser les dépenses pour les serveurs MariaDB, MySQL et PostgreSQL en les redimensionnant 
Advisor analyse votre utilisation et évalue si les ressources de votre serveur de base de données MariaDB, MySQL ou PostgreSQL ont été sous-utilisées pendant une période prolongée au cours des sept derniers jours. Une faible utilisation des ressources entraîne des dépenses indésirables que vous pouvez corriger sans impact significatif sur les performances. Pour réduire vos coûts et gérer efficacement vos ressources, nous vous recommandons de réduire de moitié la taille de calcul (vCores).

## <a name="reduce-costs-by-eliminating-unprovisioned-expressroute-circuits"></a>Réduire les coûts en éliminant les circuits ExpressRoute non approvisionnés

Advisor identifie les circuits Azure ExpressRoute dont l’état de fournisseur est **Non approvisionné** depuis plus d’un mois. Il recommande de supprimer le circuit si vous ne prévoyez pas de l’approvisionner auprès de votre fournisseur de connectivité.

## <a name="reduce-costs-by-deleting-or-reconfiguring-idle-virtual-network-gateways"></a>Réduire les coûts en supprimant ou en reconfigurant les passerelles de réseau virtuel inactives

Advisor identifie les passerelles de réseau virtuel qui sont restées inactives pendant plus de 90 jours. Dans la mesure où ces passerelles sont facturées à l’heure, vous devez envisager de les reconfigurer ou de les supprimer si vous ne souhaitez plus les utiliser. 

## <a name="buy-reserved-virtual-machine-instances-to-save-money-over-pay-as-you-go-costs"></a>Acheter des instances de machine virtuelle réservées pour économiser sur les coûts du paiement à l’utilisation

Advisor examine votre utilisation des machines virtuelles au cours des 30 derniers jours pour déterminer si vous pouvez faire des économies en achetant une réservation Azure. Advisor vous indique les régions et les tailles où le potentiel d’économie est le plus élevé et une estimation des économies réalisées grâce à l’achat de réservations. Avec les réservations Azure, vous pouvez acheter au préalable les coûts de base de vos machines virtuelles. Les remises s’appliquent automatiquement aux machines virtuelles nouvelles ou existantes ayant la même taille et la même région que vos réservations. [En savoir plus sur Azure Reserved VM Instances.](https://azure.microsoft.com/pricing/reserved-vm-instances/)

Advisor vous informe également de vos instances réservées qui expireront dans les 30 prochains jours. Il vous recommande d’acheter de nouvelles instances réservées pour éviter les tarifs du paiement à l’utilisation.

## <a name="buy-reserved-instances-for-several-resource-types-to-save-over-your-pay-as-you-go-costs"></a>Acheter des instances réservées pour plusieurs types de ressources afin d’économiser sur les coûts de paiement à l’utilisation

Advisor analyse les habitudes d’utilisation des 30 derniers jours pour les ressources suivantes et recommande des achats de capacité de réserve qui optimisent les coûts.

### <a name="azure-cosmos-db-reserved-capacity"></a>Capacité de réserve Azure Cosmos DB
Advisor analyse vos habitudes d’utilisation d’Azure Cosmos DB au cours des 30 derniers jours et recommande des achats de capacité de réserve afin d’optimiser les coûts. En utilisant la capacité de réserve, vous pouvez acheter à l’avance l’utilisation horaire d’Azure Cosmos DB et économiser sur vos coûts du paiement à l’utilisation. La capacité de réserve constitue un avantage de facturation et s’applique automatiquement aux déploiements nouveaux et existants. Advisor calcule les estimations d’économies pour les abonnements individuels en utilisant la tarification de réservation sur 3 ans et en extrapolant les habitudes d’utilisation observées au cours des 30 derniers jours. Les recommandations relatives aux étendues partagées sont disponibles pour les achats de capacité de réserve et peuvent accroître les économies.

### <a name="sql-database-and-sql-managed-instance-reserved-capacity"></a>Capacité de réserve SQL Database et SQL Managed Instance
Advisor analyse les habitudes d’utilisation de SQL Database et SQL Managed Instance au cours des 30 derniers jours. Il recommande ensuite des achats de capacité de réserve qui optimisent les coûts. En utilisant la capacité de réserve, vous pouvez acheter à l’avance l’utilisation horaire de la base de données SQL et économiser sur vos coûts de calcul SQL. Votre licence SQL est facturée séparément et ne bénéficie pas d’une remise avec la réservation. La capacité de réserve constitue un avantage de facturation et s’applique automatiquement aux déploiements nouveaux et existants. Advisor calcule les estimations d’économies pour les abonnements individuels en utilisant la tarification de réservation sur 3 ans et en extrapolant les habitudes d’utilisation observées au cours des 30 derniers jours. Les recommandations relatives aux étendues partagées sont disponibles pour les achats de capacité de réserve et peuvent accroître les économies. Pour plus d’informations, consultez [Capacité de réserve Azure SQL Database et SQL Managed Instance](../azure-sql/database/reserved-capacity-overview.md).

### <a name="app-service-stamp-fee-reserved-capacity"></a>Capacité de réserve pour les frais de timbre App Service
Advisor analyse le schéma d’utilisation des frais de timbre pour votre environnement isolé Azure App Service au cours des 30 derniers jours et recommande des achats de capacité de réserve qui optimisent les coûts. En utilisant la capacité de réserve, vous pouvez acheter à l’avance l’utilisation horaire pour les frais de timbre de l’environnement isolé et économiser sur les coûts du paiement à l’utilisation. Notez que la capacité de réserve s’applique uniquement aux frais de timbre et non aux instances App Service. La capacité de réserve constitue un avantage de facturation et s’applique automatiquement aux déploiements nouveaux et existants. Advisor calcule les estimations d’économies pour les abonnements individuels en utilisant la tarification de réservation sur 3 ans selon les habitudes d’utilisation au cours des 30 derniers jours.

### <a name="blob-storage-reserved-capacity"></a>Capacité de réserve du stockage blob
Advisor analyse votre utilisation du stockage Blob Azure et du stockage Azure Data Lake au cours des 30 derniers jours. Il calcule ensuite les achats de capacité de réserve qui optimisent les coûts. En utilisant la capacité de réserve, vous pouvez acheter à l’avance l’utilisation horaire et économiser sur vos coûts à la demande actuels. La capacité de réserve du stockage blob s’applique uniquement aux données stockées dans des comptes v2 universel Blob Azure et des comptes Azure Data Lake Storage Gen2. La capacité de réserve constitue un avantage de facturation et s’applique automatiquement aux déploiements nouveaux et existants. Advisor calcule les estimations d’économies pour les abonnements individuels en utilisant la tarification de réservation sur 3 ans et les habitudes d’utilisation observées au cours des 30 derniers jours. Les recommandations relatives aux étendues partagées sont disponibles pour les achats de capacité de réserve et peuvent accroître les économies.

### <a name="mariadb-mysql-and-postgresql-reserved-capacity"></a>Capacité de réserve MariaDB, MySQL et PostgreSQL
Advisor analyse vos modèles d’utilisation pour Azure Database for MariaDB, Azure Database pour MySQL et Azure Database pour PostgreSQL au cours des 30 derniers jours. Il recommande ensuite des achats de capacité de réserve qui optimisent les coûts. En utilisant la capacité de réserve, vous pouvez acheter à l’avance l’utilisation horaire de MariaDB, MySQL et PostgreSQL et économiser sur vos coûts actuels. La capacité de réserve constitue un avantage de facturation et s’applique automatiquement aux déploiements nouveaux et existants. Advisor calcule les estimations d’économies pour les abonnements individuels en utilisant la tarification de réservation sur 3 ans et les habitudes d’utilisation observées au cours des 30 derniers jours. Les recommandations relatives aux étendues partagées sont disponibles pour les achats de capacité de réserve et peuvent accroître les économies.

### <a name="azure-synapse-analytics-reserved-capacity"></a>Capacité de réserve Azure Synapse Analytics
Advisor analyse vos habitudes d’utilisation d’Azure Synapse Analytics au cours des 30 derniers jours et recommande des achats de capacité de réserve qui optimisent les coûts. En utilisant la capacité de réserve, vous pouvez acheter à l’avance l’utilisation horaire de Synapse Analytics et économiser sur vos coûts à la demande. La capacité de réserve constitue un avantage de facturation et s’applique automatiquement aux déploiements nouveaux et existants. Advisor calcule les estimations d’économies pour les abonnements individuels en utilisant la tarification de réservation sur 3 ans et les habitudes d’utilisation observées au cours des 30 derniers jours. Les recommandations relatives aux étendues partagées sont disponibles pour les achats de capacité de réserve et peuvent accroître les économies.

## <a name="delete-unassociated-public-ip-addresses-to-save-money"></a>Supprimer les adresses IP publiques non associées pour réaliser des économies

Advisor identifie les IP publiques qui ne sont pas associées à des ressources Azure telles que les équilibreurs de charge et les machines virtuelles. Des frais nominaux sont associés à ces IP publiques. Si vous n’avez pas l’intention de les utiliser, vous pouvez faire des économies en les supprimant.

## <a name="delete-azure-data-factory-pipelines-that-are-failing"></a>Supprimer les pipelines Azure Data Factory en échec

Advisor détecte les pipelines Azure Data Factory qui échouent de façon répétée. Il vous recommande de résoudre les problèmes ou de supprimer les pipelines si vous n’en avez pas besoin. Ces pipelines vous sont facturés , même s’ils sont en échec et ne vous sont dès lors d’aucune utilité.

## <a name="use-standard-snapshots-for-managed-disks"></a>Utiliser des instantanés standard pour les disques managés
Pour économiser jusqu’à 60 %, nous vous recommandons de stocker vos instantanés dans un stockage standard, quel que soit le type de stockage du disque parent. Il s’agit de l’option par défaut pour les instantanés de disques managés. Advisor identifie les instantanés stockés dans un stockage premium et vous recommande de les migrer vers un stockage standard. [En savoir plus sur la tarification des disques managés.](https://aka.ms/aa_manageddisksnapshot_learnmore)

## <a name="use-lifecycle-management"></a>Utiliser la gestion du cycle de vie
En utilisant les renseignements quant à la taille totale, aux transactions et au nombre d’objets du stockage Blob Azure, Advisor détecte si vous devez activer la gestion du cycle de vie afin de hiérarchiser les données sur un ou plusieurs de vos comptes de stockage. Il vous invite à créer des règles de gestion du cycle de vie pour hiérarchiser automatiquement vos données dans les stockages Froid ou Archive afin d’optimiser vos coûts de stockage tout en conservant vos données dans le stockage Blob Azure pour la compatibilité des applications.

## <a name="create-an-ephemeral-os-disk-recommendation"></a>Créer une recommandation de disque de système d’exploitation éphémère
Le [disque de système d’exploitation éphémère](../virtual-machines/ephemeral-os-disks.md) vous permet d’effectuer les opérations suivantes : 
- Réduire les coûts de stockage pour les disques de système d’exploitation. 
- Diminuer la latence de lecture/écriture sur les disques de système d’exploitation. 
- Obtenir des opérations de réinitialisation de machine virtuelle plus rapides en rétablissant le système d’exploitation (et le disque temporaire) à son état d’origine.

Il est préférable d’utiliser le disque de système d’exploitation éphémère pour les machines virtuelles IaaS à courte durée de vie ou les machines virtuelles supportant des charges de travail sans état. Advisor fournit des recommandations pour les ressources qui peuvent tirer parti du disque de système d’exploitation éphémère.

## <a name="reduce-azure-data-explorer-table-cache-period-policy-for-cluster-cost-optimization-preview"></a>Réduire la période de mise en cache (stratégie) des tables Azure Data Explorer pour l’optimisation du coût des clusters (préversion)
Advisor identifie les ressources pour lesquelles la réduction de la période de mise en cache des tables permet de libérer les nœuds de cluster Azure Data Explorer qui utilisent peu le processeur et qui disposent de peu de mémoire, mais dont la taille de cache est élevée.

## <a name="how-to-access-cost-recommendations-in-azure-advisor"></a>Accès aux recommandations de coût dans le conseiller Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Recherchez et sélectionnez [**Advisor**](https://aka.ms/azureadvisordashboard) à partir de n’importe quelle page.

1. Dans le tableau de bord **Advisor**, sélectionnez l’onglet **Coût**.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les recommandations d’Advisor, consultez les ressources suivantes :
* [Présentation du conseiller](advisor-overview.md)
* [Score Advisor](azure-advisor-score.md)
* [Prise en main d’Advisor](advisor-get-started.md)
* [Recommandations d’Advisor en matière de performances](advisor-performance-recommendations.md)
* [Recommandations d’Advisor en matière de haute disponibilité](advisor-high-availability-recommendations.md)
* [Recommandations d’Advisor en matière de sécurité](advisor-security-recommendations.md)
* [Recommandations d’Advisor en matière d’excellence opérationnelle](advisor-operational-excellence-recommendations.md)