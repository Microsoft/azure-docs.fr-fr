---
title: 'Démarrage rapide : Créer un classifieur de charge de travail - T-SQL'
description: Utilisez T-SQL pour créer un classifieur de charge de travail avec une importance haute.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: a63cd95fdcee7c9ed0c49ba41b4d7e7e6de8f4bf
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96458725"
---
# <a name="quickstart-create-a-workload-classifier-using-t-sql"></a>Démarrage rapide : Créer un classifieur de charge de travail à l’aide de T-SQL

Dans ce guide de démarrage rapide, vous créez rapidement un classifieur de charge de travail avec une importance haute pour le PDG (CEO) de votre organisation. Ce classifieur de charge de travail donne la priorités aux requêtes du PDG sur les autres requêtes avec une importance inférieure dans la file d’attente.

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

> [!NOTE]
> La création d’une instance de pool SQL dédié dans Azure Synapse Analytics peut donner lieu à un nouveau service facturable.  Pour plus d’informations, consultez [Tarification Azure Synapse Analytics](https://azure.microsoft.com/pricing/details/sql-data-warehouse/).
>
>

## <a name="prerequisites"></a>Prérequis

Ce guide de démarrage rapide suppose que vous avez déjà provisionné une instance de pool SQL dédié dans Azure Synapse Analytics, et que vous disposez d’autorisations CONTROL DATABASE. Si vous devez en créer une, utilisez [Créer et connecter – Portail](create-data-warehouse-portal.md) pour créer un pool SQL dédié nommé **mySampleDataWarehouse**.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/).

## <a name="create-login-for-theceo"></a>Créer une connexion pour TheCEO

Créez une connexion d’authentification SQL Server dans la base de données `master` à l’aide de [CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour « TheCEO ».

```sql
IF NOT EXISTS (SELECT * FROM sys.sql_logins WHERE name = 'TheCEO')
BEGIN
CREATE LOGIN [TheCEO] WITH PASSWORD='<strongpassword>'
END
;
```

## <a name="create-user"></a>Créer un utilisateur

[Créez l’utilisateur](/sql/t-sql/statements/create-user-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest), « TheCEO », dans mySampleDataWarehouse.

```sql
IF NOT EXISTS (SELECT * FROM sys.database_principals WHERE name = 'THECEO')
BEGIN
CREATE USER [TheCEO] FOR LOGIN [TheCEO]
END
;
```

## <a name="create-a-workload-classifier"></a>Créer un classifieur de charge de travail

Créez un [classifieur de charge de travail](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour « TheCEO » avec une importance haute.

```sql
DROP WORKLOAD CLASSIFIER [wgcTheCEO];
CREATE WORKLOAD CLASSIFIER [wgcTheCEO]
WITH (WORKLOAD_GROUP = 'xlargerc'
      ,MEMBERNAME = 'TheCEO'
      ,IMPORTANCE = HIGH);
```

## <a name="view-existing-classifiers"></a>Voir les classifieurs existants

```sql
SELECT * FROM sys.workload_management_workload_classifiers
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

```sql
DROP WORKLOAD CLASSIFIER [wgcTheCEO]
DROP USER [TheCEO]
;
```

Vous êtes facturé pour les unités d’entrepôt de données et les données stockées dans votre pool SQL dédié. Ces ressources de calcul et de stockage sont facturées séparément.

- Si vous voulez conserver les données dans le stockage, vous pouvez suspendre le calcul quand vous n’utilisez pas le pool SQL dédié. Quand vous suspendez le calcul, vous êtes facturé uniquement pour le stockage des données. Quand vous êtes prêt à utiliser les données, reprenez le calcul.
- Si vous voulez éviter des frais à venir, vous pouvez supprimer le pool SQL dédié.

Suivez ces étapes pour nettoyer les ressources.

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis sélectionnez votre pool SQL dédié.

    ![Nettoyer les ressources](./media/load-data-from-azure-blob-storage-using-polybase/clean-up-resources.png)

2. Pour suspendre le calcul, sélectionnez le bouton **Suspendre**. Quand le pool SQL dédié est suspendu, un bouton **Démarrer** est visible.  Pour reprendre le calcul, sélectionnez **Démarrer**.

3. Pour supprimer le pool SQL dédié afin de ne pas être facturé pour le calcul ou le stockage, sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

- Vous avez créé un classifieur de charge de travail. Exécutez quelques requêtes avec l’identité TheCEO pour voir comment elles fonctionnent. Consultez [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour voir les requêtes et l’importance attribuée.
- Pour plus d’informations sur la gestion de charge de travail du pool SQL dédié, consultez [Importance des charges de travail](sql-data-warehouse-workload-importance.md) et [Classification des charges de travail](sql-data-warehouse-workload-classification.md).
- Consultez les articles qui expliquent comment [Configurer l’importance de la charge de travail](sql-data-warehouse-how-to-configure-workload-importance.md) et comment [Gérer et surveiller la charge de travail](sql-data-warehouse-how-to-manage-and-monitor-workload-importance.md).
