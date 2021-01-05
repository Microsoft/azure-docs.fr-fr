---
title: sys.external_streaming_jobs (Transact-SQL) - Azure SQL Edge
description: En savoir plus sur l'utilisation de sys.external_streaming_jobs dans Azure SQL Edge
keywords: sys.external_streaming_jobs, SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: reference
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2019
ms.openlocfilehash: 9643c58f5c9fa1db3e3eb7ec75ce6d3b41620aa3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90900336"
---
# <a name="sysexternal_streaming_jobs-transact-sql"></a>sys.external_streaming_jobs (Transact-SQL)

Retourne une ligne pour chaque tâche de streaming externe créée dans l’étendue de la base de données.

|Nom de la colonne|Type de données|Description|  
|-----------------|---------------|-----------------|
|**name**|**sysname**|Nom du flux. Unique dans la base de données.|
|**object_id**|**int**|Numéro d'identification de l'objet de flux. Unique dans la base de données.|
|**principal_id**|**int**|Identificateur du principal qui est propriétaire de cet assembly.|
|**schema_id**|**int**| ID du schéma contenant l'objet.|
|**parent_object_id**|**id**| Numéro d'identification de l'objet parent de ce flux. Dans l'implémentation actuelle, cette valeur est toujours nulle.|
|**type**|**char(2)**|Type d'objet. Le type des objets de flux est toujours « EJ »|
|**type_desc**|**nvarchar(60)**| Description du type d'objet. Le type des objets de flux est toujours « EXTERNAL_STREAMING_JOB »|
|**create_date**|**datetime**| Date de création de l'objet.|
|**modify_date**|**datetime**| Dans l’implémentation actuelle, cette valeur est identique à la valeur create_date de l’objet de flux. |
|**is_ms_shipped**|**bit**| Objet créé par un composant interne.|  
|**is_published**|**bit**| L'objet est publié.|  
|**is_schema_published**|**bit**|Seul le schéma de l'objet est publié.|
|**uses_ansi_nulls**|**bit**| L'objet de flux a été créé lorsque l'option de base de données SET ANSI_NULLS était activée (ON)|
|**instruction**|**varchar(max)**| Texte de la requête Stream Analytics pour la tâche de streaming. Pour plus d’informations, consultez [sp_create_streaming_job](overview.md). |
|**statut**|**int**| État actuel de la tâche de streaming. Les valeurs possibles sont les suivantes : <br /><br /> **Created** = 0. La tâche de streaming a été créée, mais n’a pas encore été démarrée. <br /><br /> **Starting** = 1. La tâche de streaming est cours de démarrage. <br /><br /> **Failed** = 6. La tâche de streaming a échoué. Cela indique généralement une erreur irrécupérable lors du traitement. <br /><br /> **Stopped** = 4. La tâche de streaming a été arrêtée. <br /><br /> **Idle** = 7. La tâche de streaming est en cours d’exécution, mais il n’y a aucune entrée à traiter. <br /><br /> **Processing** = 8. La tâche de streaming est en cours d’exécution et traite les entrées. Cet état indique un état sain de la tâche de streaming. <br /><br /> **Degraded** = 9. La tâche de streaming est en cours d’exécution, mais des erreurs de sérialisation/désérialisation d'entrée ou de sortie récupérables se sont produites lors du traitement des entrées. La tâche d’entrée continuera de s’exécuter, mais supprimera les entrées rencontrant des erreurs.|

## <a name="permissions"></a>Autorisations

La visibilité des métadonnées dans les affichages catalogue est limitée aux éléments sécurisables qu'un utilisateur détient ou pour lesquels des autorisations lui ont été accordées. Pour plus d'informations, consultez [Metadata Visibility Configuration](/sql/relational-databases/security/metadata-visibility-configuration/).

## <a name="see-also"></a>Voir aussi

- [Affichages catalogue de streaming T-SQL](overview.md)
- [Affichages catalogue (Transact-SQL)](/sql/relational-databases/system-catalog-views/catalog-views-transact-sql/)
- [Vues système (Transact-SQL)](/sql/t-sql/language-reference/)

