---
title: Configurer l’importance des charges de travail pour le pool SQL dédié
description: Découvrez comment définir l’importance du niveau de la demande dans Azure Synapse Analytics.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.subservice: sql-dw
ms.topic: conceptual
ms.date: 05/15/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 067551d198f717dd40995cb8bc3e1345e82f078f
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461904"
---
# <a name="configure-workload-importance-in-dedicated-sql-pool-for-azure-synapse-analytics"></a>Configurer l’importance des charges de travail dans un pool SQL dédié pour Azure Synapse Analytics

Le fait de définir l’importance dans un pool SQL dédié pour Azure Synapse permet d’influencer la planification des requêtes. Les requêtes plus importantes seront programmées de manière à s’exécuter avant les requêtes moins importantes. Pour affecter une importance aux requêtes, vous devez créer un classifieur de charges de travail.

## <a name="create-a-workload-classifier-with-importance"></a>Créer un classifieur de charges de travail avec une importance

Souvent, dans un scénario d’entrepôt de données, certains utilisateurs, sur un système occupé, doivent exécuter leurs requêtes rapidement.  Les utilisateurs peuvent être des responsables d’une société ayant besoin d’exécuter des rapports ou l’utilisateur peut être un analyste exécutant une requête ad hoc. Pour attribuer une importance, vous créez un classificateur de charge de travail et l’importance est assignée à une requête.  Les exemples ci-dessous utilisent la syntaxe de [création d’un classifieur de charge de travail](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour créer deux classifieurs. `Membername` peut être un utilisateur unique ou un groupe.  Pour rechercher des utilisateurs du pool SQL dédié existants, exécutez :

```sql
Select name from sys.sysusers
```

Pour créer un classifieur de charges de travail pour un utilisateur avec une importance haute, exécutez :

```sql
CREATE WORKLOAD CLASSIFIER ExecReportsClassifier
    WITH (WORKLOAD_GROUP = 'xlargerc'
         ,MEMBERNAME     = 'name' 
         ,IMPORTANCE     = above_normal);
```

Pour créer un classifieur de charges de travail pour un utilisateur exécutant des requêtes adhoc avec une importance inférieure, exécutez :  

```sql
CREATE WORKLOAD CLASSIFIER AdhocClassifier
    WITH (WORKLOAD_GROUP = 'xlargerc'
         ,MEMBERNAME     = 'name' 
         ,IMPORTANCE     = below_normal);
```

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur la gestion des charges de travail, consultez [Classification des charges de travail](sql-data-warehouse-workload-classification.md)
- Pour plus d’informations sur l’importance, consultez [Importance de la charge de travail](sql-data-warehouse-workload-importance.md)

> [!div class="nextstepaction"]
> [Accéder à Gérer et superviser l’importance de la charge de travail](sql-data-warehouse-how-to-manage-and-monitor-workload-importance.md)
