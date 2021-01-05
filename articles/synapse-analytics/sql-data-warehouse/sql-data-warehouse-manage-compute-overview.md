---
title: Gérer la ressource de calcul pour un pool SQL dédié (anciennement SQL DW)
description: Découvrez les capacités de scale-out des performances pour un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics. Effectuez un scale-out en ajustant la valeur DWU ou allégez les coûts en interrompant le pool SQL dédié.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/12/2019
ms.author: rortloff
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 300759b4ab6f806c02e748ff4c9a63a6a772bff4
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461081"
---
# <a name="manage-compute-for-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Gérer le calcul pour un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics

Découvrez comment gérer le calcul pour un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics. Réduisez les coûts en suspendant le pool SQL dédié, ou mettez à l’échelle le pool SQL dédié afin de répondre aux exigences en matière de performances.

## <a name="what-is-compute-management"></a>En quoi consiste la gestion des ressources de calcul ?

L’architecture du pool SQL dédié (anciennement SQL DW) sépare le stockage du calcul, ce qui permet de les mettre à l’échelle indépendamment l’un de l’autre. En conséquence, vous pouvez procéder à la mise à l’échelle du calcul afin de répondre aux exigences en matière de niveau de performance, sans modifier le stockage des données. Vous avez également la possibilité de suspendre ou reprendre des ressources de calcul. Dans le cadre de cette architecture, la [facturation](https://azure.microsoft.com/pricing/details/sql-data-warehouse/) du calcul est donc effectuée séparément de celle du stockage. Si vous n’avez pas besoin d’utiliser pool SQL dédié (anciennement SQL DW) pendant un certain temps, vous pouvez alléger les coûts de calcul en interrompant les ressources de calcul.

## <a name="scaling-compute"></a>Mise à l’échelle du calcul

Vous pouvez effectuer un scale-out ou un scale-back du calcul en ajustant le paramétrage relatif aux [unités DWU (Data Warehouse Units)](what-is-a-data-warehouse-unit-dwu-cdwu.md) de pool SQL dédié (anciennement SQL DW). Les performances de chargement et de requête peuvent s’accroître de manière linéaire à mesure que vous augmentez la valeur DWU.

Pour plus d’informations sur la procédure de montée en puissance, consultez les articles de démarrage rapide pour le [portail Azure](quickstart-scale-compute-portal.md), [PowerShell](quickstart-scale-compute-powershell.md) ou [T-SQL](quickstart-scale-compute-tsql.md). Vous pouvez également effectuer des opérations de montée en puissance à l’aide d’une [API REST](sql-data-warehouse-manage-compute-rest-api.md#scale-compute).

Pour procéder à une mise à l’échelle, le pool SQL dédié (anciennement SQL DW) commence par supprimer toutes les requêtes entrantes, puis il restaure les transactions afin de garantir un état cohérent. La mise à l’échelle ne se produit qu’une fois la restauration des transactions effectuée. Dans le cadre d’une opération de mise à l’échelle, le système détache la couche de stockage des nœuds de calcul, ajoute des nœuds de calcul, puis rattache la couche de stockage à la couche de calcul. Chaque pool SQL dédié (anciennement SQL DW) est stocké sous la forme de 60 distributions, qui sont réparties uniformément entre les nœuds de calcul. L’ajout de nœuds de calcul supplémentaires augmente la puissance de calcul. À mesure que le nombre de nœuds de calcul augmente, le nombre de distributions par nœud de calcul diminue, ce qui fournit davantage de puissance de calcul pour vos requêtes. De même, la diminution du nombre d’unités DWU réduit le nombre de nœuds de calcul, ce qui réduit les ressources de calcul pour les requêtes.

Le tableau ci-après illustre l’évolution du nombre de distributions par nœud de calcul à mesure que la valeur DWU change.  La valeur DW30000c fournit 60 nœuds de calcul et offre un niveau de performances de requête bien supérieur à celui de la valeur DW100c.

| Data Warehouse Units  | \# de nœuds de calcul | \# de distributions par nœud |
| -------- | ---------------- | -------------------------- |
| DW100c   | 1                | 60                         |
| DW200c   | 1                | 60                         |
| DW300c   | 1                | 60                         |
| DW400c   | 1                | 60                         |
| DW500c   | 1                | 60                         |
| DW1000c  | 2                | 30                         |
| DW1500c  | 3                | 20                         |
| DW2000c  | 4                | 15                         |
| DW2500c  | 5                | 12                         |
| DW3000c  | 6                | 10                         |
| DW5000c  | 10               | 6                          |
| DW6000c  | 12               | 5                          |
| DW7500c  | 15               | 4                          |
| DW10000c | 20               | 3                          |
| DW15000c | 30               | 2                          |
| DW30000c | 60               | 1                          |

## <a name="finding-the-right-size-of-data-warehouse-units"></a>Détermination du nombre d’unités DWU optimal

Pour découvrir les avantages d’une montée en puissance en termes de niveau de performance, en particulier dans le cas des valeurs DWU les plus élevées, vous devez utiliser un jeu de données d’au moins 1 To. Pour déterminer le nombre d’unités DWU idéal pour votre pool SQL dédié (anciennement SQL DW), essayez d’effectuer un scale-up et un scale-down. Exécutez quelques requêtes avec différentes valeurs DWU après avoir chargé vos données. La mise à l’échelle étant rapide, vous pouvez essayer plusieurs niveaux de performances en une heure ou moins.

Recommandations en matière de recherche de la valeur DWU optimale :

- Pour un pool SQL dédié (anciennement SQL DW) en développement, commencez par sélectionner un nombre plus réduit d’unités DWU.  DW400c ou DW200c est un bon point de départ.
- Analysez les performances de votre application, en observant notamment le nombre d’unités DWU sélectionné.
- Déterminez la mesure dans laquelle vous devez augmenter ou diminuer le nombre d’unités DWU à l’aide d’une mise à l’échelle linéaire.
- Continuez à effectuer des ajustements jusqu’à ce que vous atteigniez le niveau de performances requis par vos activités.

## <a name="when-to-scale-out"></a>Quand procéder à un scale-out

Une montée en puissance par l’augmentation de la valeur DWU a une incidence sur les aspects des performances ci-après :

- amélioration linéaire des performances du système pour les analyses, les agrégations et les instructions CTAS (Create Table As Select) ;
- augmentation du nombre de processus de lecture et d’écriture pour le chargement de données ;
- nombre maximal de requêtes simultanées et d’emplacements de concurrence.

Recommandations concernant les cas dans lesquels effectuer un scale-out de la valeur DWU :

- Avant d’exécuter une opération de chargement ou de transformation de données importante, effectuez un scale-out afin que vos données soient disponibles plus rapidement.
- Pendant les heures de pointe, effectuez un scale-out pour prendre en charge un nombre plus important de requêtes simultanées.

## <a name="what-if-scaling-out-does-not-improve-performance"></a>Que faire si la montée en puissance n’améliore pas les performances ?

L’ajout d’unités DWU augmente le parallélisme. Si le travail est réparti uniformément entre les nœuds de calcul, ce surcroît de parallélisme accroît le niveau de performance des requêtes. Si la montée en puissance n’améliore pas votre niveau de performance, certaines raisons peuvent l’expliquer. Il peut exister une asymétrie des données entre les distributions, ou il est possible que des requêtes introduisent un déplacement important des données. Pour examiner les problèmes liés aux performances des requêtes, consultez l’article relatif à la [résolution des problèmes de performances](sql-data-warehouse-troubleshoot.md#performance).

## <a name="pausing-and-resuming-compute"></a>Suspension et reprise du calcul

La suspension du calcul consiste à détacher la couche de stockage des nœuds de calcul. Les ressources de calcul sont libérées de votre compte. Pendant la suspension du calcul, vous n’êtes pas facturé pour ce dernier. La reprise du calcul rattache le stockage aux nœuds de calcul et réenclenche la facturation du calcul.
Lorsque vous interrompez un pool SQL dédié (anciennement SQL DW) :

- Les ressources de calcul et de mémoire sont renvoyées dans le pool des ressources disponibles du centre de données.
- Les coûts de DWU sont nuls pendant toute la durée de la suspension.
- Le stockage de données n’est pas affecté et vos données restent intactes.
- Toutes les opérations en cours d’exécution ou en file d’attente sont annulées.

Lorsque vous reprenez un pool SQL dédié (anciennement SQL DW) :

- Le pool SQL dédié (anciennement SQL DW) acquiert les ressources de calcul et de mémoire correspondant à votre paramètre DWU.
- Les frais de calcul concernant vos unités DWU vous sont de nouveau facturés.
- Vos données deviennent disponibles.
- Une fois le pool SQL dédié (anciennement SQL DW) en ligne, vous devez redémarrer vos requêtes de charge de travail.

Si vous souhaitez que votre pool SQL dédié (anciennement SQL DW) soit toujours accessible, envisagez d’effectuer un scale-down à sa taille minimale plutôt que de l’interrompre.

Pour plus d’informations sur les procédures de suspension et de reprise, consultez les articles de démarrage rapide pour le [portail Azure](pause-and-resume-compute-portal.md) ou pour [PowerShell](pause-and-resume-compute-powershell.md). Vous pouvez également utiliser [l’API REST de suspension](sql-data-warehouse-manage-compute-rest-api.md#pause-compute) ou [l’API REST de reprise](sql-data-warehouse-manage-compute-rest-api.md#resume-compute).

## <a name="drain-transactions-before-pausing-or-scaling"></a>Vider les transactions avant la suspension ou la mise à l’échelle

Nous vous recommandons d’autoriser l’achèvement des transactions existantes avant d’initialiser une opération de suspension ou de mise à l’échelle.

Lorsque vous suspendez ou mettez à l’échelle votre pool SQL dédié (anciennement SQL DW), vos requêtes sont annulées en arrière-plan au moment où vous lancez la demande de suspension ou de mise à l’échelle. L’annulation d’une simple requête SELECT est une opération rapide et n’a quasiment aucun impact sur le temps nécessaire à la suspension ou à la mise à l’échelle de votre instance.  Toutefois, les requêtes transactionnelles, qui modifient vos données ou la structure des données, ne pourront peut-être pas s’arrêter rapidement. **Par définition, les requêtes transactionnelles doivent être terminées dans leur intégralité ou annuler leurs modifications.** L’annulation du travail effectué par une requête transactionnelle peut être aussi longue, voire plus, que la modification originale appliquée par la requête. Par exemple, si vous annulez une requête qui supprimait des lignes et était en cours d’exécution depuis une heure, le système mettra peut-être une heure à insérer à nouveau les lignes supprimées. Si vous exécutez une suspension ou une mise à l’échelle pendant que les transactions sont en cours, votre suspension ou mise à l’échelle peut sembler très longue, car la suspension et la mise à l’échelle doivent attendre la fin de la restauration avant de se lancer.

Consultez aussi [Transactions](sql-data-warehouse-develop-transactions.md) et [Optimisation des transactions](sql-data-warehouse-develop-best-practices-transactions.md).

## <a name="automating-compute-management"></a>Automatisation de la gestion du calcul

Pour plus d’informations sur l’automatisation des opérations de gestion du calcul, consultez l’article décrivant comment [gérer le calcul avec Azure Functions](manage-compute-with-azure-functions.md).

L’exécution de chacune des opérations de montée en puissance, de suspension et de reprise peut nécessiter plusieurs minutes. Si vous procédez à une mise à l’échelle, une suspension ou une reprise automatiques, nous vous recommandons d’implémenter une logique pour vous assurer que certaines opérations ont été effectuées avant d’exécuter une autre action. La vérification de l’état du pool SQL dédié (anciennement SQL DW) au niveau de différents points de terminaison vous permet d’automatiser correctement de telles opérations.

Pour vérifier l’état du pool SQL dédié (anciennement SQL DW), consultez le guide de démarrage rapide [PowerShell](quickstart-scale-compute-powershell.md#check-data-warehouse-state) ou [T-SQL](quickstart-scale-compute-tsql.md#check-dedicated-sql-pool-formerly-sql-dw-state). Vous pouvez également vérifier l’état du pool SQL dédié (anciennement SQL DW) à l’aide d’une [API REST](sql-data-warehouse-manage-compute-rest-api.md#check-database-state).

## <a name="permissions"></a>Autorisations

La mise à l’échelle du pool SQL dédié (anciennement SQL DW) exige les autorisations décrites dans [ALTER DATABASE](/sql/t-sql/statements/alter-database-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  La suspension et la reprise requièrent l’autorisation [Collaborateur de base de données SQL](../../role-based-access-control/built-in-roles.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#sql-db-contributor), notamment Microsoft.Sql/servers/databases/action.

## <a name="next-steps"></a>Étapes suivantes

Consultez le guide pratique pour [gérer les ressources de calcul](manage-compute-with-azure-functions.md). Un autre aspect de la gestion des ressources de calcul concerne l’allocation de ressources de calcul différentes pour des requêtes spécifiques. Pour plus d’informations, consultez l’article [Classes de ressources pour la gestion des charges de travail](resource-classes-for-workload-management.md).
