---
title: sys.external_job_streams (Transact-SQL) – Azure SQL Edge
description: En savoir plus sur l’utilisation de sys.external_job_streams dans Azure SQL Edge
keywords: sys.external_job_streams, SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: reference
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2019
ms.openlocfilehash: 35010d3aba7f6d5ee3185291c917ff7726ba8bd7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90900356"
---
# <a name="sysexternal_job_streams-transact-sql"></a>sys.external_job_streams (Transact-SQL)

Retourne une ligne pour chaque objet de flux externe d'entrée ou de sortie mappé à une tâche de streaming externe.

|Nom de la colonne|Type de données|Description|  
|-----------------|---------------|-----------------|
|**job_id**|**int**| Numéro d'identification de l'objet de tâche de streaming. Cette colonne est mappée à la colonne object_id de sys.external_streaming_jobs.|
|**stream_id**|**int**| Numéro d'identification de l'objet de flux. Cette colonne est mappée à la colonne object_id de sys.external_streams. |
|**is_input**|**bit**| 1 si l’objet de flux est utilisé en tant qu'entrée pour la tâche de streaming, sinon 0.|
|**is_output**|**bit**| 1 si l’objet de flux est utilisé en tant que sortie pour la tâche de streaming, sinon 0.|

## <a name="example"></a>Exemple

Cet affichage catalogue est utilisé avec les affichages catalogue `sys.external_streams` et `sys.external_streaming_jobs`. Vous trouverez ci-dessous un exemple de requête.

```sql
Select
    sj.Name as Job_Name,
    sj.Create_date as Job_Create_date,
    sj.modify_date as Job_Modify_date,
    sj.statement as Stream_Job_Query,
    Input_Stream_Name =
        Case js.is_input
            when 1 then s.Name
            else null
        END,
    output_Stream_Name =
        case js.is_output
            when 1 then s.Name
            else null
        END,
    s.location as Stream_Location
from sys.external_job_streams js
inner join sys.external_streams s on s.object_id = js.stream_id
inner join sys.external_streaming_jobs sj on sj.object_id = js.job_id
```

## <a name="permissions"></a>Autorisations

La visibilité des métadonnées dans les affichages catalogue est limitée aux éléments sécurisables qu'un utilisateur détient ou pour lesquels des autorisations lui ont été accordées. Pour plus d'informations, consultez [Metadata Visibility Configuration](/sql/relational-databases/security/metadata-visibility-configuration/).

## <a name="see-also"></a>Voir aussi

- [Affichages catalogue (Transact-SQL)](/sql/relational-databases/system-catalog-views/catalog-views-transact-sql/)
- [Vues système (Transact-SQL)](/sql/t-sql/language-reference/)
