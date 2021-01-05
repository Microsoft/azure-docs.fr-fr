---
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 11/09/2018
ms.author: jingwang
ms.openlocfilehash: 1ab404b838af65dcb75395dfeee1ca0553e497a1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177733"
---
## <a name="specifying-structure-definition-for-rectangular-datasets"></a>Spécification de la définition de la structure des jeux de données rectangulaires
La section structure dans les jeux de données JSON est une section **facultative** pour les tables rectangulaires (avec des lignes et colonnes) et contient une collection de colonnes de la table. Vous allez utiliser la section structure pour indiquer des informations de type pour les conversions de type ou pour effectuer des mappages de colonnes. Les sections suivantes décrivent ces fonctionnalités en détail. 

Chaque colonne contient les propriétés suivantes :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| name |Nom de la colonne. |Oui |
| type |Type de données de la colonne. Pour plus d’informations sur le moment où vous devez spécifier les informations de type, consultez la section des conversions de type ci-dessous. |Non |
| culture |Culture .NET à utiliser lorsque le type est spécifié et qu’il est de type .NET Datetime ou Datetimeoffset. La valeur par défaut est « fr-fr ». |Non |
| format |Chaîne de format à utiliser lorsque le type est spécifié et qu’il est de type .NET Datetime ou Datetimeoffset. |Non |

L’exemple suivant présente la section JSON structure d’une table qui comporte trois colonnes : userid, name et lastlogindate.

```json
"structure": 
[
    { "name": "userid"},
    { "name": "name"},
    { "name": "lastlogindate"}
],
```

Suivez les recommandations suivantes pour savoir quand inclure les informations de « structure » et pour connaître les éléments à inclure dans la section **structure** .

* **Pour les sources de données structurées** qui stockent le schéma de données et les informations de type, ainsi que les données proprement dites (sources telles qu’une table Azure, SQL Server, Oracle, etc.), vous devez spécifier la section « structure » uniquement si vous voulez effectuer un mappage des colonnes sources spécifiques aux colonnes spécifiques du récepteur et si leurs noms ne sont pas identiques (voir les détails de la section de mappage des colonnes ci-dessous). 
  
    Comme mentionné ci-dessus, les informations de type sont facultatives dans la section « structure ». Pour les sources structurées, les informations de type sont déjà disponibles dans le cadre de la définition du jeu de données dans le magasin de données. Vous ne devez donc pas les inclure lorsque vous incluez la section « structure ».
* **Pour un schéma des sources de données de lecture (en particulier les objets blob Azure)** , vous pouvez choisir de stocker des données sans les informations de type ou de schéma. Pour ces types de source de données, vous devez inclure « structure » dans les 2 cas suivants :
  * Vous souhaitez effectuer un mappage de colonnes.
  * Si le jeu de données est une source dans une activité de copie, vous pouvez fournir des informations de type dans la « structure », et la fabrique de données utilisera ces informations de type pour la conversion en types natifs pour le récepteur. Pour plus d’informations, consultez l’article [Déplacer des données vers et depuis un stockage d’objets blob Azure](../articles/data-factory/v1/data-factory-azure-blob-connector.md) .

### <a name="supported-net-based-types"></a>Types .NET pris en charge
La fabrique de données prend en charge les valeurs de type .NET conformes CLS suivantes pour fournir des informations de type dans la section « structure » du schéma dans les sources de données de lecture telles qu’un objet blob Azure.

* Int16
* Int32 
* Int64
* Unique
* Double
* Decimal
* Byte[]
* Bool
* String 
* Guid
* Datetime
* Datetimeoffset
* Timespan 

Pour Datetime et Datetimeoffset, vous pouvez également spécifier une chaîne « culture » et « format » pour faciliter l’analyse de votre chaîne Datetime personnalisée. Consultez l’exemple de conversion de type ci-dessous.

