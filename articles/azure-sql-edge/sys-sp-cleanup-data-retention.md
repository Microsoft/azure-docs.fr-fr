---
title: sys.sp_cleanup_data_retention (Transact-SQL) - Azure SQL Edge
description: Découvrez comment utiliser sys.sp_cleanup_data_retention (Transact-SQL) dans Azure SQL Edge
keywords: sys.sp_cleanup_data_retention (Transact-SQL), SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: reference
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 9c0a6700a476d4f7f875af5373e3c99bc4e25af2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930086"
---
# <a name="syssp_cleanup_data_retention-transact-sql"></a>sys.sp_cleanup_data_retention (Transact-SQL)

**S’applique à :** Azure SQL Edge

Effectue le nettoyage des enregistrements obsolètes dans les tables sur lesquelles des stratégies de conservation des données sont appliquées. Pour plus d’informations, consultez [Conservation des données](data-retention-overview.md). 

## <a name="syntax"></a>Syntaxe 

```sql
sys.sp_cleanup_data_retention   
    { [@schema_name = ] 'schema_name' },  
    { [@table_name = ] 'table_name' },   
    [ [@rowcount =] rowcount OUTPUT ]    

```

## <a name="arguments"></a>Arguments  
`[ @schema_name = ] schema_name`    
 Nom du schéma propriétaire de la table dans laquelle le nettoyage doit être effectué. *schema_name* est un paramètre obligatoire de type **sysname**.
  
`[ @table_name = ] 'table_name'`    
 Nom de la table dans laquelle l’opération de nettoyage doit être effectuée. *table_name* est un paramètre obligatoire de type **sysname**.

## <a name="output-parameter"></a>Paramètre de sortie  

`[ @rowcount = ] rowcount OUTPUT`   
 rowcount est un paramètre de sortie facultatif qui représente le nombre d’enregistrements nettoyés dans la table. *rowcount* a le type int.
  
## <a name="permissions"></a>Autorisations    
 Requiert les autorisations db_owner.

## <a name="next-steps"></a>Étapes suivantes
- [Conservation des données et purge automatique des données](data-retention-overview.md)
- [Gérer les données historiques avec la stratégie de conservation](data-retention-cleanup.md) 
- [Activer et désactiver la conservation des données](data-retention-enable-disable.md)
