---
title: Utilisation de schémas définis par l’utilisateur
description: Conseils d’utilisation des schémas T-SQL définis par l’utilisateur pour développer des solutions dans des pools SQL dédiés dans Azure Synapse Analytics.
services: synapse-analytics
author: MSTehrani
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/17/2018
ms.author: emtehran
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 3204c77dd076d9aac6eb5a60b489280caefcbf4b
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460450"
---
# <a name="user-defined-schemas-for-dedicated-sql-pools-in-azure-synapse-analytics"></a>Schémas définis par l’utilisateur pour les pools SQL dédiés dans Azure Synapse Analytics
Cet article fournit plusieurs conseils d’utilisation des schémas T-SQL définis par l’utilisateur pour développer des solutions dans un pool SQL dédié.

## <a name="schemas-for-application-boundaries"></a>Schémas pour les limites d’application

Les entrepôts de données traditionnels utilisent souvent des bases de données distinctes pour créer des limites d’application basées sur la charge de travail, le domaine ou la sécurité. 

Par exemple, un entrepôt de données SQL Server classique peut inclure une base de données de la zone de transit, une base de données de l’entrepôt de données et quelques bases de données de mini-Data Warehouse. Dans cette topologie, chaque base de données joue le rôle de limite de sécurité et de charge de travail dans l’architecture.

À l’inverse, un pool SQL dédié exécute l’intégralité de la charge de travail des entrepôts de données au sein d’une seule et même base de données. Les jointures entre plusieurs bases de données ne sont pas autorisées. Un pool SQL dédié s’attend à ce que l’ensemble des tables utilisées par l’entrepôt soient stockées dans une seule et même base de données.

> [!NOTE]
> Le pool SQL ne prend pas en charge les requêtes de base de données croisées, de quelque nature que ce soit. Par conséquent, les implémentations d’entrepôt de données tirant parti de ce modèle devront être modifiées.
> 
> 

## <a name="recommendations"></a>Recommandations
Vous trouverez ci-dessous des recommandations pour la consolidation des charges de travail, de la sécurité, du domaine et des limites fonctionnelles à l’aide de schémas définis par l’utilisateur :

- Utilisez une base de données dans un pool SQL dédié pour exécuter l’intégralité de la charge de travail de l’entrepôt de données.
- Consolidez votre environnement d’entrepôt de données existant de manière à utiliser une seule base de données de pool SQL dédié.
- Tirez parti de **schémas définis par l’utilisateur** pour fournir la limite précédemment implémentée via des bases de données.

Si aucun schéma défini par l’utilisateur n’a été utilisé précédemment, vous partez de zéro. Utilisez l’ancien nom de base de données comme base pour vos schémas définis par l’utilisateur dans la base de données du pool SQL dédié.

Si des schémas ont déjà été utilisés, vous avez le choix entre plusieurs options :

- Vous pouvez supprimer les noms de schéma hérités et recommencer de zéro.
- Vous pouvez conserver le nom de schéma hérité et l’ajouter au nom de la table.
- Vous pouvez conserver le nom de schéma hérité, en implémentant les vues sur la table au sein d’un schéma supplémentaire pour recréer l’ancienne structure du schéma.

> [!NOTE]
> À première vue, la troisième option semble la plus tentante. Toutefois, si on l’étudie plus en détail, elle ne semble pas à la hauteur de ses promesses. Les vues sont en lecture seule dans un pool SQL dédié. Toute modification portant sur les données ou la table doit être effectuée sur la table de base. Par ailleurs, cette troisième option présente une couche de vues au sein de votre système. Peut-être devriez-vous étudier la question plus attentivement si vous utilisez déjà des vues dans votre architecture.
> 
> 

### <a name="examples"></a>Exemples :
Implémentez des schémas définis par l’utilisateur basés sur des noms de base de données :

```sql
CREATE SCHEMA [stg]; -- stg previously database name for staging database
GO
CREATE SCHEMA [edw]; -- edw previously database name for the data warehouse
GO
CREATE TABLE [stg].[customer] -- create staging tables in the stg schema
(       CustKey BIGINT NOT NULL
,       ...
);
GO
CREATE TABLE [edw].[customer] -- create data warehouse tables in the edw schema
(       CustKey BIGINT NOT NULL
,       ...
);
```

Conservez les noms de schéma hérités en les ajoutant au nom de la table. Utilisez des schémas pour la limite de charge de travail :

```sql
CREATE SCHEMA [stg]; -- stg defines the staging boundary
GO
CREATE SCHEMA [edw]; -- edw defines the data warehouse boundary
GO
CREATE TABLE [stg].[dim_customer] --pre-pend the old schema name to the table and create in the staging boundary
(       CustKey BIGINT NOT NULL
,       ...
);
GO
CREATE TABLE [edw].[dim_customer] --pre-pend the old schema name to the table and create in the data warehouse boundary
(       CustKey BIGINT NOT NULL
,       ...
);
```

Conservez les noms de schéma hérités à l’aide de vues :

```sql
CREATE SCHEMA [stg]; -- stg defines the staging boundary
GO
CREATE SCHEMA [edw]; -- stg defines the data warehouse boundary
GO
CREATE SCHEMA [dim]; -- edw defines the legacy schema name boundary
GO
CREATE TABLE [stg].[customer] -- create the base staging tables in the staging boundary
(       CustKey    BIGINT NOT NULL
,       ...
)
GO
CREATE TABLE [edw].[customer] -- create the base data warehouse tables in the data warehouse boundary
(       CustKey    BIGINT NOT NULL
,       ...
)
GO
CREATE VIEW [dim].[customer] -- create a view in the legacy schema name boundary for presentation consistency purposes only
AS
SELECT  CustKey
,       ...
FROM    [edw].customer
;
```

> [!NOTE]
> Toute modification portant sur la stratégie de schéma nécessite une révision du modèle de sécurité de la base de données. Dans de nombreux cas, il est possible de simplifier le modèle de sécurité en attribuant des autorisations au niveau du schéma. S’il vous faut des autorisations plus granulaires, vous pouvez utiliser des rôles de base de données.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir des conseils supplémentaires, consultez la [vue d’ensemble du développement](sql-data-warehouse-overview-develop.md).

