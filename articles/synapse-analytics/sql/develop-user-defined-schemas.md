---
title: Schémas définis par l’utilisateur dans SQL Synapse
description: Dans les sections ci-dessous, vous trouverez différents conseils pour l’utilisation de schémas T-SQL définis par l’utilisateur afin de développer des solutions avec la fonctionnalité SQL Synapse d’Azure Synapse Analytics.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: 6920d6cd190927747a69fc4dc0f0791278665abe
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96451696"
---
# <a name="user-defined-schemas-within-synapse-sql"></a>Schémas définis par l’utilisateur dans SQL Synapse

Dans les sections ci-dessous, vous trouverez différents conseils pour l’utilisation de schémas T-SQL définis par l’utilisateur afin de développer des solutions avec SQL Synapse.

## <a name="schemas-for-application-boundaries"></a>Schémas pour les limites d’application

L’architecture analytique traditionnelle utilise souvent des bases de données distinctes pour créer des limites d’application basées sur la charge de travail, le domaine ou la sécurité. Par exemple, une infrastructure analytique SQL Server classique peut inclure une base de données de la zone de transit, une base de données analytique et des bases de données de mini-Data Warehouse. Dans cette topologie, chaque base de données joue le rôle de limite de sécurité et de charge de travail dans l’architecture.

Au lieu de cela, SQL Synapse exécute l’intégralité de la charge de travail analytique dans une base de données. Les jointures entre plusieurs bases de données ne sont pas autorisées. SQL Synapse s’attend à ce que l’ensemble des tables utilisées par l’entrepôt soient stockées au sein d’une seule et même base de données.

> [!NOTE]
> Les pools SQL dédiés ne prennent pas en charge les requêtes sur plusieurs bases de données, de quelque nature que ce soit. Par conséquent, les implémentations d’analytique tirant parti de ce modèle devront être modifiées. Le pool SQL serverless prend en charge les requêtes sur plusieurs bases de données.

## <a name="user-defined-schema-recommendations"></a>Recommandations relatives au schéma défini par l’utilisateur

Vous trouverez des recommandations pour la consolidation des limites de charge de travail, fonctionnelles, de domaine et de sécurité à l’aide de schémas définis par l’utilisateur :

- Utilisez une base de données pour exécuter l’intégralité de votre charge de travail analytique.
- Consolidez votre environnement analytique existant pour qu’il utilise une seule base de données.
- Tirez parti de **schémas définis par l’utilisateur** pour fournir la limite précédemment implémentée via des bases de données.

Si aucun schéma défini par l’utilisateur n’a été utilisé précédemment, vous partez de zéro. Utilisez l’ancien nom de base de données comme base pour vos schémas définis par l’utilisateur dans la base de données SQL Synapse.

Si des schémas ont déjà été utilisés, vous avez le choix entre plusieurs options :

- Vous pouvez supprimer les noms de schéma hérités et recommencer de zéro.
- Vous pouvez conserver le nom de schéma hérité et l’ajouter au nom de la table.
- Vous pouvez conserver le nom de schéma hérité, en implémentant les vues sur la table au sein d’un schéma supplémentaire, qui recrée l’ancienne structure du schéma.

> [!NOTE]
> À première vue, la troisième option semble la plus tentante. Les affichages sont en lecture seule dans SQL Synapse. Toute modification portant sur les données ou la table doit être effectuée sur la table de base. Par ailleurs, cette troisième option présente une couche de vues au sein de votre système. Peut-être devriez-vous étudier la question plus attentivement si vous utilisez déjà des vues dans votre architecture.
> 
> 

### <a name="examples"></a>Exemples

Implémentez des schémas définis par l’utilisateur basés sur des noms de base de données.

```sql
CREATE SCHEMA [stg]; -- stg previously database name for staging database
GO
CREATE SCHEMA [edw]; -- edw previously database name for the analytics
GO
CREATE TABLE [stg].[customer] -- create staging tables in the stg schema
(       CustKey BIGINT NOT NULL
,       ...
);
GO
CREATE TABLE [edw].[customer] -- create analytics tables in the edw schema
(       CustKey BIGINT NOT NULL
,       ...
);
```

Conservez les noms de schéma hérités en les ajoutant au nom de la table. utilisation de schémas pour la limite de charge de travail

```sql
CREATE SCHEMA [stg]; -- stg defines the staging boundary
GO
CREATE SCHEMA [edw]; -- edw defines the analytics boundary
GO
CREATE TABLE [stg].[dim_customer] --pre-pend the old schema name to the table and create in the staging boundary
(       CustKey BIGINT NOT NULL
,       ...
);
GO
CREATE TABLE [edw].[dim_customer] --pre-pend the old schema name to the table and create in the analytics boundary
(       CustKey BIGINT NOT NULL
,       ...
);
```

Conservez les noms de schémas hérités à l’aide de vues.

```sql
CREATE SCHEMA [stg]; -- stg defines the staging boundary
GO
CREATE SCHEMA [edw]; -- stg defines the analytics boundary
GO
CREATE SCHEMA [dim]; -- edw defines the legacy schema name boundary
GO
CREATE TABLE [stg].[customer] -- create the base staging tables in the staging boundary
(       CustKey    BIGINT NOT NULL
,       ...
)
GO
CREATE TABLE [edw].[customer] -- create the base analytics tables in the analytics boundary
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
> Toute modification portant sur la stratégie de schéma nécessite une révision du modèle de sécurité de la base de données. Dans de nombreux cas, il est possible de simplifier le modèle de sécurité en attribuant des autorisations au niveau du schéma.

S’il vous faut des autorisations plus granulaires, vous pouvez utiliser des rôles de base de données. Pour en savoir plus sur les rôles de base de données, consultez l’article [Gérer les rôles et les utilisateurs de base de données](../../analysis-services/analysis-services-database-users.md).

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des conseils supplémentaires en matière de développement, consultez [Présentation du développement SQL Synapse](develop-overview.md).
