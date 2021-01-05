---
title: Isolation des charges de travail
description: Conseils pour la définition de l’isolation des charges de travail avec des groupes de charges de travail dans Azure Synapse Analytics.
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
ms.openlocfilehash: a9ebee68c7abd90f5fb3345eec1ee929fc30ca20
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85212307"
---
# <a name="azure-synapse-analytics-workload-group-isolation"></a>Isolation des groupes de charges de travail Azure Synapse Analytics

Cet article explique comment les groupes de charge de travail peuvent être utilisés pour configurer l’isolation de la charge de travail, contenir des ressources et appliquer des règles runtime pour l’exécution des requêtes.

## <a name="workload-groups"></a>Groupes de charges de travail

Les groupes de charges de travail sont des conteneurs pour un ensemble de requêtes et constituent la base de la configuration de la gestion des charges de travail, y compris l’isolation de la charge de travail, sur un système.  Les groupes de charge de travail sont créés à l’aide de la syntaxe [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Une configuration de gestion de charge de travail simple peut gérer les charges de données et les requêtes utilisateur.  Par exemple, un groupe de charge de travail nommé `wgDataLoads` définit des aspects de charge de travail pour les données chargées dans le système. En outre, un groupe de charge de travail nommé `wgUserQueries` définit des aspects de charge de travail pour les utilisateurs qui exécutent des requêtes afin de lire des données à partir du système.

Les sections suivantes décrivent comment les groupes de charges de travail offrent la possibilité de définir l’isolation, l’autonomie, la définition des ressources de la requête et d’adhérer aux règles d’exécution.

## <a name="workload-isolation"></a>Isolation des charges de travail

L’isolation de la charge de travail signifie que les ressources sont réservées, exclusivement, pour un groupe de charge de travail.  L’isolation des charges de travail s’effectue en configurant le paramètre MIN_PERCENTAGE_RESOURCE sur une valeur supérieure à zéro dans la syntaxe [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Pour les charges de travail d’exécution continues qui doivent adhérer à des contrats de niveau de service étroits, l’isolation garantit que les ressources sont toujours disponibles pour le groupe de charge de travail.

La configuration de l’isolation de la charge de travail définit implicitement un niveau garanti de concurrence. Par exemple, un groupe de charge de travail avec un `MIN_PERCENTAGE_RESOURCE` défini sur 30 % et un `REQUEST_MIN_RESOURCE_GRANT_PERCENT` défini sur 2 % a 15 concurrences de garantie.  Le niveau de concurrence est garanti car 2 % des emplacements de ressources avec 15 concurrences sont réservés à tout moment dans le groupe de charge de travail (quelle que soit la façon dont `REQUEST_*MAX*_RESOURCE_GRANT_PERCENT` est configuré).  Si `REQUEST_MAX_RESOURCE_GRANT_PERCENT` est supérieur à `REQUEST_MIN_RESOURCE_GRANT_PERCENT` et `CAP_PERCENTAGE_RESOURCE` est supérieur à `MIN_PERCENTAGE_RESOURCE` des ressources supplémentaires sont ajoutées par requête.  Si `REQUEST_MAX_RESOURCE_GRANT_PERCENT` et `REQUEST_MIN_RESOURCE_GRANT_PERCENT` sont égaux et que `CAP_PERCENTAGE_RESOURCE` est supérieur à `MIN_PERCENTAGE_RESOURCE`, une concurrence supplémentaire est possible.  Considérez la méthode ci-dessous pour déterminer la concurrence garantie :

[Accès concurrentiel garanti] = [`MIN_PERCENTAGE_RESOURCE`] / [`REQUEST_MIN_RESOURCE_GRANT_PERCENT`]

> [!NOTE]
> Il existe des valeurs de niveau de service spécifiques au minimum pour min_percentage_resource.  Pour plus d’informations, consultez [Valeurs effectives](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest#effective-values) pour plus d’informations.

En l’absence d’isolation de la charge de travail, les requêtes opèrent dans le [pool partagé](#shared-pool-resources) de ressources.  L’accès aux ressources du pool partagé n’est pas garanti et est attribué selon l’[importance](sql-data-warehouse-workload-importance.md).

La configuration de l’isolation de la charge de travail doit être effectuée avec précaution, car les ressources sont allouées au groupe de charges de travail, même s’il n’y a pas de requêtes actives dans le groupe de charge de travail. L’isolation trop configurable peut entraîner une diminution de l’utilisation globale du système.

Les utilisateurs doivent éviter une solution de gestion de la charge de travail qui configure l’isolation de la charge de travail de 100 % : l’isolation de 100 % est obtenue lorsque la somme des min_percentage_resource configurés pour tous les groupes de charges de travail est égale à 100 %.  Ce type de configuration est trop restrictif et rigide, laissant peu de place pour les requêtes de ressources qui sont mal classées par erreur. Il existe une provision pour permettre l’exécution d’une requête à partir de groupes de charge de travail non configurés pour l’isolation. Les ressources allouées à cette requête s’affichent sous la forme d’un zéro dans les vues DMV des systèmes et empruntent un niveau smallrc d’allocation de ressources à partir des ressources réservées du système.

> [!NOTE]
> Pour garantir une utilisation optimale des ressources, envisagez une solution de gestion de la charge de travail qui exploite une certaine isolation pour garantir la satisfaction des contrats de niveau de service et les combiner avec les ressources partagées accessibles en fonction de l’[importance de la charge de travail](sql-data-warehouse-workload-importance.md).

## <a name="workload-containment"></a>Autonomie de la charge de travail

L’autonomie de la charge de travail fait référence à la limitation de la quantité de ressources qu’un groupe de charge de travail peut consommer.  L’isolation de la charge de travail s’effectue en configurant le paramètre MIN_PERCENTAGE_RESOURCE sur une valeur inférieure à 100 dans la syntaxe [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Considérez le scénario dans lequel les utilisateurs ont besoin d’un accès en lecture au système pour pouvoir exécuter une analyse de scénarios via des requêtes ad hoc.  Ces types de requêtes peuvent avoir un impact négatif sur les autres charges de travail qui s’exécutent sur le système.  La configuration de l’autonomie permet de limiter la quantité de ressources.

La configuration de l’autonomie de la charge de travail définit implicitement un niveau garanti de concurrence.  Avec un CAP_PERCENTAGE_RESOURCE défini sur 60 % et un REQUEST_MIN_RESOURCE_GRANT_PERCENT défini sur 1 %, un niveau jusqu’à 60 concurrences est autorisé pour le groupe de charge de travail.  Considérez la méthode incluse ci-dessous pour déterminer la concurrence maximale :

[Concurrence Max] = [`CAP_PERCENTAGE_RESOURCE`] / [`REQUEST_MIN_RESOURCE_GRANT_PERCENT`]

> [!NOTE]
> Le CAP_PERCENTAGE_RESOURCE effectif d’un groupe de charge de travail n’atteint pas 100 % lorsque les groupes de charges de travail avec MIN_PERCENTAGE_RESOURCE à un niveau supérieur à zéro sont créés.  Consultez [sys.dm_workload_management_workload_groups_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-workload-management-workload-group-stats-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour des valeurs de runtime effectives.

## <a name="resources-per-request-definition"></a>Ressources par définition de requête

Les groupes de charges de travail fournissent un mécanisme permettant de définir le nombre minimal et maximal de ressources qui sont allouées par requête avec les paramètres REQUEST_MIN_RESOURCE_GRANT_PERCENT et REQUEST_MAX_RESOURCE_GRANT_PERCENT dans la syntaxe [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).  Dans ce cas, les ressources sont l’UC et la mémoire.  La configuration de ces valeurs détermine la quantité de ressources et le niveau de concurrence pouvant être atteint sur le système.

> [!NOTE]
> REQUEST_MAX_RESOURCE_GRANT_PERCENT est un paramètre facultatif qui a comme valeur par défaut la même valeur que celle spécifiée pour REQUEST_MIN_RESOURCE_GRANT_PERCENT.

Comme le choix d’une classe de ressource, la configuration de REQUEST_MIN_RESOURCE_GRANT_PERCENT définit la valeur des ressources utilisées par une requête.  La quantité de ressources indiquée par la valeur définie est garantie pour l’allocation à la requête avant le début de l’exécution.  Pour les clients qui migrent des classes de ressources vers des groupes de charges de travail, envisagez de suivre l’article [Comment](sql-data-warehouse-how-to-convert-resource-classes-workload-groups.md) pour mapper des classes de ressources à des groupes de charge de travail comme point de départ.

La configuration de REQUEST_MAX_RESOURCE_GRANT_PERCENT sur une valeur supérieure à REQUEST_MIN_RESOURCE_GRANT_PERCENT permet au système d’allouer plus de ressources par requête.  Lors de la planification d’une requête, le système détermine l’allocation réelle des ressources à la requête, qui est comprise entre REQUEST_MIN_RESOURCE_GRANT_PERCENT et REQUEST_MAX_RESOURCE_GRANT_PERCENT, en fonction de la disponibilité des ressources dans le pool partagé et de la charge actuelle sur le requise.  Les ressources doivent exister dans le [pool partagé](#shared-pool-resources) des ressources lorsque la requête est planifiée.  

> [!NOTE]
> REQUEST_MIN_RESOURCE_GRANT_PERCENT et REQUEST_MAX_RESOURCE_GRANT_PERCENT ont des valeurs effectives qui dépendent des valeurs MIN_PERCENTAGE_RESOURCE et CAP_PERCENTAGE_RESOURCE effectives.  Consultez [sys.dm_workload_management_workload_groups_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-workload-management-workload-group-stats-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) pour des valeurs de runtime effectives.

## <a name="execution-rules"></a>Délai d’exécution

Sur les systèmes de création de rapports ad hoc, les clients peuvent exécuter accidentellement des pertes de contrôle de requêtes qui ont un impact sérieux sur la productivité des autres.  Les administrateurs système sont obligés de perdre du temps en éliminant des pertes de contrôle de requêtes pour libérer des ressources système.  Les groupes de charge de travail permettent de configurer une règle de délai d’expiration d’exécution de requête pour annuler les requêtes qui ont dépassé la valeur spécifiée.  La règle est configurée en définissant le paramètre `QUERY_EXECUTION_TIMEOUT_SEC` dans la syntaxe [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="shared-pool-resources"></a>Ressources de pool partagé

Les ressources de pool partagé sont les ressources qui ne sont pas configurées pour l’isolation.  Les groupes de charge de travail avec un MIN_PERCENTAGE_RESOURCE défini sur zéro tirent parti des ressources du pool partagé pour exécuter des requêtes.  Les groupes de charges de travail avec un CAP_PERCENTAGE_RESOURCE supérieur à MIN_PERCENTAGE_RESOURCE également utilisé des ressources partagées.  La quantité de ressources disponibles dans le pool partagé est calculée comme suit.

[Pool partagé] = 100-[somme de `MIN_PERCENTAGE_RESOURCE` pour tous les groupes de charge de travail]

L’accès aux ressources du pool partagé est alloué selon l’[importance](sql-data-warehouse-workload-importance.md).  Les requêtes ayant le même niveau d’importance accèdent aux ressources de pool partagé sur la base premier entré/premier sortie.

## <a name="next-steps"></a>Étapes suivantes

- [Démarrage rapîde : configurer l’isolation de la charge de travail](quickstart-configure-workload-isolation-tsql.md)
- [CREATE WORKLOAD GROUP](/sql/t-sql/statements/create-workload-group-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [Convertir les classes de ressources en groupes de charges de travail](sql-data-warehouse-how-to-convert-resource-classes-workload-groups.md).
- [Supervision du portail de gestion des charges de travail](sql-data-warehouse-workload-management-portal-monitor.md).  
