---
title: Importance des charges de travail
description: Conseils pour la définition de l’importance des requêtes de pool SQL dédié dans Azure Synapse Analytics.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 07c781672874bff306c9d25a464ec66414ebc9f1
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322125"
---
# <a name="azure-synapse-analytics-workload-importance"></a>Importance des charges de travail Azure Synapse Analytics

Cet article explique comment l’importance des charges de travail peut influer sur l’ordre d’exécution des requêtes de pool SQL dédié dans Azure Synapse.

## <a name="importance"></a>importance

> [!Video https://www.youtube.com/embed/_2rLMljOjw8]

Certaines charges de travail d'entreposage de données peuvent être plus importantes que d'autres pour une entreprise.  Imaginez un scénario dans lequel les données de vente critiques sont chargées avant la clôture de la période fiscale.  Les chargements de données pour d'autres sources, telles que les données météo, n'ont pas de contrats de niveau de service stricts. Définir une haute importance pour une requête de chargement de données de vente et une faible importance pour une demande de chargement de données météo confère au chargement des données de vente une priorité sur les ressources et permet son exécution plus rapide.

## <a name="importance-levels"></a>Niveaux d'importance

Il existe cinq niveaux d’importance : low (faible), below_normal (inférieure à la normale), normal (normale), above_normal (supérieure à la normale) et high (haute).  Par défaut, les requêtes se voient attribuer une importance normale. Les requêtes dotées du même niveau d’importance répondent au même comportement de planification qu'actuellement.

## <a name="importance-scenarios"></a>Scénarios avec importance

Au-delà du scénario avec importance de base décrit ci-dessus et portant sur des données de vente et données météo, il existe d'autres scénarios dans lesquels l'importance des charges de travail contribue à répondre aux besoins de traitement et d'interrogation des données.

### <a name="locking"></a>Verrouillage

L'accès aux verrous des activités de lecture et d’écriture est un point de contention naturelle. Des activités telles que le [basculement des partitions](sql-data-warehouse-tables-partition.md) ou [RENAME OBJECT](/sql/t-sql/statements/rename-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) requièrent des verrous avec élévation de privilèges.  Sans l’importance des charges de travail, un pool SQL dédié dans Azure Synapse optimise le débit. Il y a optimisation du débit lorsque les requêtes en cours d'exécution et en file d'attente présentent les mêmes besoins de verrouillage en présence de ressources disponibles, et que les requêtes en file d'attente peuvent contourner les requêtes présentant des besoins de verrouillage plus élevés que celles arrivées dans la file d'attente plus tôt. L'importance des charges de travail est appliquée aux requêtes présentant des besoins de verrouillage plus élevés. Les requêtes présentant une importance plus élevée sont exécutées avant les requêtes présentant une plus faible importance.

Prenons l’exemple suivant :

- Q1 est en cours d’exécution et sélectionne des données dans SalesFact.
- Q2 est en file d’attente en attendant que Q1 se termine.  Cette requête a été soumise à 9h00 et tente de procéder au basculement des partitions de nouvelles données dans SalesFact.
- Q3 est soumise à 9h01 et souhaite sélectionner des données dans SalesFact.

Si Q2 et Q3 présentent la même importance, Q1 continue de s'exécuter et Q3 va commencer à s'exécuter. Q2 continue d'attendre un verrou exclusif sur SalesFact.  Si Q2 présente une importance plus élevée que Q3, Q3 attend que Q2 ait terminé avant de commencer à s'exécuter.

### <a name="non-uniform-requests"></a>Requêtes non uniformes

Les requêtes soumises avec différentes classes de ressources sont un autre exemple de scénario dans lequel l'importance contribue à répondre aux besoins d'interrogation.  Comme indiqué précédemment, si l’importance est la même, un pool SQL dédié dans Azure Synapse optimise le débit. Quand des demandes de tailles différentes (par exemple, smallrc ou mediumrc) sont mises en file d’attente, un pool SQL dédié choisit la première demande arrivée en fonction des ressources disponibles. Si l’importance des charges de travail est appliquée, la requête présentant la plus haute importance est planifiée ensuite.
  
Considérez l'exemple suivant sur DW500c :

- Q1, Q2, Q3 et Q4 exécutent des requêtes smallrc.
- Q5 est soumise avec la classe de ressources mediumrc à 9h00.
- Q6 est soumise avec la classe de ressources smallrc à 9h01.

La classe de ressources de Q5 étant mediumrc, elle nécessite deux emplacements de concurrence. Q5 doit attendre la fin de deux requêtes.  Cela étant, lorsqu’une des requêtes en cours d’exécution (Q1 à Q4) se termine, Q6 est immédiatement planifiée, car les ressources permettent son exécution.  Si Q5 présente une importance plus élevée que Q6, Q6 attend la fin de Q5 pour commencer à s'exécuter.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur la création d’un classifieur, consultez [CRÉER UN CLASSIFIEUR DE CHARGE DE TRAVAIL (Transact-SQL)](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  
- Pour plus d’informations sur la classification des charges de travail, consultez [Classification des charges de travail](sql-data-warehouse-workload-classification.md).  
- Pour savoir comment créer un classifieur de charge de travail, consultez le démarrage rapide [Créer un classifieur de charge de travail](quickstart-create-a-workload-classifier-tsql.md).
- Consultez les articles qui expliquent comment [Configurer l’importance de la charge de travail](sql-data-warehouse-how-to-configure-workload-importance.md) et comment [Gérer et surveiller la charge de travail](sql-data-warehouse-how-to-manage-and-monitor-workload-importance.md).
- Consultez [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour voir les requêtes et l’importance attribuée.
