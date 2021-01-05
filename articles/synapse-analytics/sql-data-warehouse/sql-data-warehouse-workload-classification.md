---
title: Classification des charges de travail pour le pool SQL dédié
description: Conseils d'utilisation de la classification pour gérer la concurrence des requêtes, leur importance et les ressources de calcul en lien avec le pool SQL dédié dans Azure Synapse Analytics.
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
ms.openlocfilehash: bf19e2d1674d0a0c2102280b28b5549505c1dfab
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96447762"
---
# <a name="workload-classification-for-dedicated-sql-pool-in-azure-synapse-analytics"></a>Classification des charges de travail pour le pool SQL dédié dans Azure Synapse Analytics

Cet article explique le processus de classification des charges de travail pour l’attribution d’un groupe de charges de travail et d’une importance aux demandes entrantes avec les pools SQL dédiés dans Azure Synapse.

## <a name="classification"></a>classification ;

> [!Video https://www.youtube.com/embed/QcCRBAhoXpM]

La classification de la gestion d’une charge de travail permet aux stratégies de charge de travail d’être appliquées aux demandes par le biais de [classes de ressources](resource-classes-for-workload-management.md#what-are-resource-classes) et de l’[importance](sql-data-warehouse-workload-importance.md).

Bien qu’il existe de nombreuses façons de classifier des charges de travail d’entreposage de données, la classification la plus simple et la plus courante est la charge et l’interrogation. Vous chargez des données avec des instructions d’insertion, de mise à jour et de suppression.  Vous interrogez les données à l’aide des instructions SELECT. Une solution d’entrepôt de données possède souvent une stratégie de charge de travail pour l’activité de charge, tels que l’attribution d’une classe de ressources supérieure avec davantage de ressources. Une stratégie de charge de travail différente pourrait s’appliquer aux requêtes, comme une importance inférieure par rapport aux activités de charge.

Vous pouvez également effectuer une sous-classification de vos charges de travail de charge et d’interrogation. La sous-classification vous donne davantage de contrôle sur vos charges de travail. Par exemple, les charges de travail de requête peuvent se composer d’actualisations de cube, de requêtes de tableau de bord ou de requêtes ad-hoc. Vous pouvez classer chacune de ces charges de travail de requête avec différentes classes de ressources ou différents paramètres d’importance. La charge peut également faire l’objet d’une sous-classification. Les transformations volumineuses peuvent être affectées à des classes de ressources plus grandes. Une importance plus élevée peut servir à garantir le fait que les données clés des ventes soient chargées avant les données météorologiques ou un flux de données sociales.

Toutes les instructions ne sont pas classées car elles n’exigent pas de ressources ou n’influencent pas l’exécution.  Les commandes DBCC, les instructions BEGIN, COMMIT et ROLLBACK TRANSACTION ne sont pas classées.

## <a name="classification-process"></a>Processus de classification

La classification relative au pool SQL dédié s'effectue aujourd'hui en attribuant aux utilisateurs un rôle auquel une classe de ressource correspondante est allouée à l'aide de [sp_addrolemember](/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest). La possibilité de caractériser des demandes au-delà d’une connexion à une classe de ressource est limitée avec cette fonctionnalité. Une méthode plus riche pour la classification est désormais disponible avec la syntaxe [CRÉER UN CLASSIFIEUR DE CHARGE DE TRAVAIL](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Avec cette syntaxe, les utilisateurs du pool SQL dédié peuvent affecter une importance ainsi que la quantité de ressources système à une demande via le paramètre `workload_group`.

> [!NOTE]
> La classification est évaluée à chaque requête. Plusieurs demandes dans une seule session peuvent être classées différemment.

## <a name="classification-weighting"></a>Pondération de la classification

Dans le cadre du processus de classification, une pondération est en place pour déterminer le groupe de charge de travail affecté.  La pondération se présente comme suit :

|Paramètre du classifieur |Poids   |
|---------------------|---------|
|MEMBERNAME:USER      |64       |
|MEMBERNAME:ROLE      |32       |
|WLM_LABEL            |16       |
|WLM_CONTEXT          |8        |
|START_TIME/END_TIME  |4        |

Le paramètre `membername` est obligatoire.  Toutefois, si le membername spécifié est un utilisateur de base de données au lieu d’un rôle de base de données, la pondération de l’utilisateur est plus élevée, de sorte que le classifieur est choisi.

Si un utilisateur est membre de plusieurs rôles avec différentes classes de ressources affectées ou mises en correspondance dans plusieurs classifieurs, l’utilisateur bénéficie de l’attribution de la classe de ressource la plus élevée.  Ce comportement est cohérent avec le comportement d’attribution de la classe ressource existante.

## <a name="system-classifiers"></a>Classifieurs système

La classification de charge de travail possède des classifieurs de charge de travail système. Les classifieurs système effectuent le mappage des appartenances existantes de rôle de classe de ressource aux allocations de ressources de classe de ressource avec une importance normale. Les classifieurs système ne peuvent pas être annulés. Pour afficher les classifieurs système, vous pouvez exécuter la requête ci-dessous :

```sql
SELECT * FROM sys.workload_management_workload_classifiers where classifier_id <= 12
```

## <a name="mixing-resource-class-assignments-with-classifiers"></a>Mélange d’affectations de classe de ressources et des classifieurs

Les classifieurs système créés en votre nom offrent un accès facile pour migrer vers une classification de charge de travail. L’utilisation de mappages de rôle de classe de ressources avec une priorité de classification peut entraîner une mauvaise classification lorsque vous commencez à créer de nouveaux classifieurs avec une importance.

Examinez le cas suivant :

- Un entrepôt de données existant possède un utilisateur de base de données DBAUser attribué au rôle de classe de ressources largerc. L’attribution de classe de ressources a été effectuée avec sp_addrolemember.
- L’entrepôt de données est maintenant mis à jour avec la gestion des charges de travail.
- Pour tester la nouvelle syntaxe de classification, le rôle de base de données DBARole (dont DBAUser est membre), possède un classifieur créé pour lui, le mappant à mediumrc et possédant une importance élevée.
- Lorsque DBAUser se connecte et exécute une requête, celle-ci est assignée à largerc. La raison est que l’utilisateur est prioritaire sur une appartenance au rôle.

Pour simplifier la résolution des problèmes de classification, nous vous recommandons de supprimer les mappages de rôle de classe de ressources car vous créez des classifieurs de charge de travail.  Le code ci-dessous retourne des appartenances existantes aux rôles de classe de ressources.  Exécutez [sp_droprolemember](/sql/relational-databases/system-stored-procedures/sp-droprolemember-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour chaque nom de membre retourné par la classe de ressource correspondante.

```sql
SELECT  r.name AS [Resource Class]
,       m.name AS membername
FROM    sys.database_role_members rm
JOIN    sys.database_principals AS r ON rm.role_principal_id = r.principal_id
JOIN    sys.database_principals AS m ON rm.member_principal_id = m.principal_id
WHERE   r.name IN ('mediumrc','largerc','xlargerc','staticrc10','staticrc20','staticrc30','staticrc40','staticrc50','staticrc60','staticrc70','staticrc80');

--for each row returned run
sp_droprolemember '[Resource Class]', membername
```

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur la création d’un classifieur, consultez [CRÉER UN CLASSIFIEUR DE CHARGE DE TRAVAIL (Transact-SQL)](/sql/t-sql/statements/create-workload-classifier-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  
- Consultez le guide de démarrage rapide sur la création d’un classifieur de charge de travail [Créer un classifieur de charge de travail](quickstart-create-a-workload-classifier-tsql.md).
- Consultez les articles qui expliquent comment [configurer l’importance de la charge de travail](sql-data-warehouse-how-to-configure-workload-importance.md) et comment [gérer et surveiller la charge de travail](sql-data-warehouse-how-to-manage-and-monitor-workload-importance.md).
- Consultez [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour voir les requêtes et l’importance attribuée.
