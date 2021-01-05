---
title: Types de données de table dans SQL Synapse
description: Recommandations pour la définition des types de données de table dans SQL synapse.
services: synapse-analytics
author: filippopovic
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.custom: ''
ms.openlocfilehash: cf627444407e7b0c43d15485fde3c342c6c24c7f
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93314882"
---
# <a name="table-data-types-in-synapse-sql"></a>Types de données de table dans SQL Synapse

Dans cet article, vous trouverez des recommandations pour définir les types de données de table dans Synapse SQL. 

## <a name="data-types"></a>Types de données

SQL Synapse prend en charge les types de données les plus couramment utilisés. Pour obtenir la liste des types de données pris en charge, consultez les [types de données](/sql/t-sql/statements/create-table-azure-sql-data-warehouse#DataTypes&preserve-view=true) dans l’instruction CREATE TABLE. 

## <a name="minimize-row-length"></a>Réduction de la longueur de ligne

En réduisant la taille des types de données, vous réduisez la longueur de ligne, ce qui entraîne une amélioration des performances de requête. Utilisez le plus petit type de données qui fonctionne pour vos données.

- Évitez de définir des colonnes de caractères ayant une grande longueur par défaut. Par exemple, si la valeur la plus longue est de 25 caractères, définissez la colonne en tant que VARCHAR(25).
- Évitez d’utiliser [NVARCHAR][NVARCHAR] lorsque vous avez uniquement besoin de VARCHAR.
- Lorsque c’est possible, utilisez NVARCHAR(4000) ou VARCHAR(8000) au lieu de NVARCHAR(MAX) ou VARCHAR(MAX).

> [!NOTE]
> Si vous utilisez des tables externes PolyBase pour charger vos tables Synapse SQL, la longueur définie pour la ligne de table ne doit pas dépasser 1 Mo. Lorsqu’une ligne avec des données de longueur variable dépasse 1 Mo, vous pouvez télécharger la ligne avec BCP, mais pas avec PolyBase.

## <a name="identify-unsupported-data-types"></a>Identification des types de données non pris en charge

Si vous migrez votre base de données à partir d’une autre base de données SQL, vous pouvez rencontrer des types de données qui ne sont pas pris en charge dans SQL Synapse. Utilisez cette requête pour identifier les types de données non pris en charge dans votre schéma SQL existant.

```sql
SELECT  t.[name], c.[name], c.[system_type_id], c.[user_type_id], y.[is_user_defined], y.[name]
FROM sys.tables  t
JOIN sys.columns c on t.[object_id]    = c.[object_id]
JOIN sys.types   y on c.[user_type_id] = y.[user_type_id]
WHERE y.[name] IN ('geography','geometry','hierarchyid','image','text','ntext','sql_variant','xml')
 AND  y.[is_user_defined] = 1;
```

## <a name="workarounds-for-unsupported-data-types"></a><a name="unsupported-data-types"></a>Solutions de contournement pour les types de données non pris en charge

La liste suivante présente les types de données que SQL Synapse ne prend pas en charge, ainsi que des alternatives à ceux-ci.

| Type de données non pris en charge | Solution de contournement |
| --- | --- |
| [geometry](/sql/t-sql/spatial-geometry/spatial-types-geometry-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true&preserve-view=true) |[varbinary](/sql/t-sql/data-types/binary-and-varbinary-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |
| [Geography](/sql/t-sql/spatial-geography/spatial-types-geography) |[varbinary](/sql/t-sql/data-types/binary-and-varbinary-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |
| [hierarchyid](/sql/t-sql/data-types/hierarchyid-data-type-method-reference) |[nvarchar](/sql/t-sql/data-types/nchar-and-nvarchar-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)(4000) |
| [image](/sql/t-sql/data-types/ntext-text-and-image-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |[varbinary](/sql/t-sql/data-types/binary-and-varbinary-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |
| [text](/sql/t-sql/data-types/ntext-text-and-image-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |[varchar](/sql/t-sql/data-types/char-and-varchar-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |
| [ntext](/sql/t-sql/data-types/ntext-text-and-image-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |[nvarchar](/sql/t-sql/data-types/nchar-and-nvarchar-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |
| [sql_variant](/sql/t-sql/data-types/sql-variant-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Fractionnez la colonne en plusieurs colonnes fortement typées. |
| [table](/sql/t-sql/data-types/table-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Convertissez-les en tables temporaires ou envisagez de stocker les données dans le stockage à l’aide de [CETAS](../sql/develop-tables-cetas.md). |
| [timestamp](/sql/t-sql/data-types/date-and-time-types) |Corrigez le code à utiliser [datetime2](/sql/t-sql/data-types/datetime2-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) et la fonction [CURRENT_TIMESTAMP](/sql/t-sql/functions/current-timestamp-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true). Seules les constantes sont prises en charge en tant que valeurs par défaut, par conséquent le paramètre current_timestamp ne peut pas être défini comme une contrainte par défaut. Si vous devez migrer les valeurs de version de ligne à partir d’une colonne de type horodatage, utilisez le paramètre [BINARY](/sql/t-sql/data-types/binary-and-varbinary-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)(8) ou [VARBINARY](/sql/t-sql/data-types/binary-and-varbinary-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)(8) pour les valeurs de version de ligne NOT NULL ou NULL. |
| [xml](/sql/t-sql/xml/xml-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |[varchar](/sql/t-sql/data-types/char-and-varchar-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |
| [type défini par l’utilisateur](/sql/relational-databases/native-client/features/using-user-defined-types) |Revenez au type de données natif lorsque c’est possible. |
| valeurs par défaut | Les valeurs par défaut prennent uniquement en charge des littéraux et des constantes. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le développement des tables, consultez [Vue d’ensemble des tables](develop-overview.md).
