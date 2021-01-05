---
title: sys.external_streams (Transact-SQL) - Azure SQL Edge
description: En savoir plus sur l’utilisation de sys.external_streams dans Azure SQL Edge
keywords: sys.external_streams, SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: reference
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2019
ms.openlocfilehash: 04950f01c06bc3c8ed3bb11a790310c2319a0579
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90900309"
---
# <a name="sysexternal_streams-transact-sql"></a>sys.external_streams (Transact-SQL)

Retourne une ligne pour chaque objet de flux externe créé dans l’étendue de la base de données.

|Nom de la colonne|Type de données|Description|  
|-----------------|---------------|-----------------|
|**name**|**sysname**|Nom du flux. Unique dans la base de données.|
|**object_id**|**int**|Numéro d'identification de l'objet de flux. Unique dans la base de données.|
|**principal_id**|**int**|Identificateur du principal qui est propriétaire de cet assembly.|
|**schema_id**|**int**| ID du schéma contenant l'objet.|
|**parent_object_id**|**id**| Numéro d'identification de l'objet parent de ce flux. Dans l'implémentation actuelle, cette valeur est toujours nulle.|
|**type**|**char(2)**|Type d'objet. Le type des objets de flux est toujours « ES »|
|**type_desc**|**nvarchar(60)**| Description du type d'objet. Le type des objets de flux est toujours « EXTERNAL_STREAM »|
|**create_date**|**datetime**| Date de création de l'objet.|
|**modify_date**|**datetime**| Date de la dernière modification de l'objet avec l'instruction ALTER.|
|**is_ms_shipped**|**bit**| Objet créé par un composant interne.|  
|**is_published**|**bit**|L'objet est publié.|  
|**is_schema_published**|**bit**|Seul le schéma de l'objet est publié.|
|**max_column_id_used**|**bit**| Cette colonne est utilisée à des fins internes et sera supprimée ultérieurement|  
|**uses_ansi_nulls**|**bit**| L'objet de flux a été créé lorsque l'option de base de données SET ANSI_NULLS était activée (ON)|
|**data_source_id**|**int**| ID d’objet de la source de données externe représentée par l’objet de flux |  
|**file_format_id**|**int**| ID d’objet du format de fichier externe utilisé par l’objet de flux. Le format de fichier externe est nécessaire pour spécifier la disposition des données référencées par un flux externe.| 
|**location**|**varchar(max)**| Cible de l’objet de flux externe. Pour plus d’informations, consultez [Créer un flux externe](overview.md). |
|**input_option**|**varchar(max)**| Options d’entrée utilisées lors de la création du flux externe. Pour plus d’informations, consultez [Créer un flux externe](overview.md). |
|**output_option**|**varchar(max)**| Options de sortie utilisées lors de la création du flux externe. Pour plus d’informations, consultez [Créer un flux externe](overview.md). | 

## <a name="permissions"></a>Autorisations

La visibilité des métadonnées dans les affichages catalogue est limitée aux éléments sécurisables qu'un utilisateur détient ou pour lesquels des autorisations lui ont été accordées. Pour plus d'informations, consultez [Metadata Visibility Configuration](/sql/relational-databases/security/metadata-visibility-configuration/).

## <a name="see-also"></a>Voir aussi

- [Affichages catalogue (Transact-SQL)](/sql/relational-databases/system-catalog-views/catalog-views-transact-sql/)
- [Vues système (Transact-SQL)](/sql/t-sql/language-reference/)
