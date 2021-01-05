---
title: Ajouter des étiquettes à des jumeaux numériques
titleSuffix: Azure Digital Twins
description: Découvrez comment implémenter des étiquettes sur des jumeaux numériques
author: baanders
ms.author: baanders
ms.date: 7/22/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: c2620b52c426871b0ec85e3db237be2d373d42f1
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96458702"
---
# <a name="add-tags-to-digital-twins"></a>Ajouter des étiquettes à des jumeaux numériques 

Vous pouvez utiliser le concept d’étiquettes pour identifier et catégoriser davantage vos jumeaux numériques. En particulier, les utilisateurs peuvent être amenés à répliquer des étiquettes à partir de systèmes existants, tels que les [étiquettes Haystack](https://project-haystack.org/doc/TagModel), dans leurs instances Azure Digital Twins. 

Ce document décrit les modèles qui peuvent être utilisés pour implémenter des étiquettes sur des jumeaux numériques.

Les étiquettes sont d’abord ajoutées en tant que propriétés dans le [modèle](concepts-models.md) qui décrit un jumeau numérique. Cette propriété est ensuite définie sur le jumeau lorsqu’il est créé en fonction du modèle. Après cela, les étiquettes peuvent être utilisées dans les [requêtes](concepts-query-language.md) pour identifier et filtrer vos jumeaux.

## <a name="marker-tags"></a>Étiquettes de marqueur 

Une **étiquette de marqueur** est une chaîne simple utilisée pour marquer ou catégoriser un jumeau numérique, telle que « bleu » ou « rouge ». Cette chaîne est le nom de l’étiquette et les étiquettes de marqueur n’ont pas de valeur significative : l’étiquette est significative par sa seule présence (ou absence). 

### <a name="add-marker-tags-to-model"></a>Ajouter des étiquettes de marqueur à un modèle 

Les balises de marqueur sont modélisées sous la forme d’un mappage [DTDL](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) de `string` en `boolean`. La valeur booléenne `mapValue` est ignorée, car la présence de la balise est tout ce qui est importe. 

Voici un extrait d’un modèle de jumeau qui implémente une balise de marqueur en tant que propriété :

```json
{
  "@type": "Property",
  "name": "tags",
  "schema": {
    "@type": "Map",
    "mapKey": {
      "name": "tagName",
      "schema": "string"
    },
    "mapValue": {
      "name": "tagValue",
      "schema": "boolean"
    }
  }
}
```

### <a name="add-marker-tags-to-digital-twins"></a>Ajouter des balises de marqueur à des jumeaux numériques

Une fois que la propriété `tags` fait partie d’un modèle de jumeau numérique, vous pouvez définir la balise de marqueur dans le jumeau numérique en définissant la valeur de cette propriété. 

Voici un exemple qui remplit la propriété `tags` de marqueur pour trois jumeaux :

```csharp
entity-01: "tags": { "red": true, "round": true } 
entity-02: "tags": { "blue": true, "round": true } 
entity-03: "tags": { "red": true, "large": true } 
```

### <a name="query-with-marker-tags"></a>Requête avec des balises de marqueur

Une fois que des balises ont été ajoutées aux jumeaux numériques, les balises peuvent être utilisées pour filtrer les jumeaux dans des requêtes. 

Voici une requête permettant d’obtenir tous les jumeaux qui sont rouges (marqués de la balise « red ») : 

```sql
SELECT * FROM digitaltwins WHERE is_defined(tags.red) 
```

Vous pouvez également combiner des balises pour créer des requêtes plus complexes. Voici une requête permettant d’obtenir tous les jumeaux qui sont ronds et pas rouges : 

```sql
SELECT * FROM digitaltwins WHERE NOT is_defined(tags.red) AND is_defined(tags.round) 
```

## <a name="value-tags"></a>Balises de valeur 

Une **balise de valeur** est une paire clé-valeur utilisée pour attribuer une valeur à chaque balise, telle que `"color": "blue"` ou `"color": "red"`. Une fois qu’une balise de valeur a été créée, vous pouvez également l’utiliser comme balise de marqueur en ignorant sa valeur. 

### <a name="add-value-tags-to-model"></a>Ajouter des balises de valeur à un modèle 

Les balises de valeur sont modélisées sous la forme d’un mappage [DTDL](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) de `string` en `string`. Les éléments `mapKey` et `mapValue` sont tous les deux significatifs. 

Voici un extrait d’un modèle de jumeau qui implémente une balise de valeur en tant que propriété :

```json
{
  "@type": "Property",
  "name": "tags",
  "schema": {
    "@type": "Map",
    "mapKey": {
      "name": "tagName",
      "schema": "string"
    },
    "mapValue": {
      "name": "tagValue",
      "schema": "string"
    }
  }
} 
```

### <a name="add-value-tags-to-digital-twins"></a>Ajouter des balises de valeur à des jumeaux numériques

Comme avec les balises de marqueur, vous pouvez définir la balise de valeur dans un jumeau numérique en définissant la valeur de cette propriété `tags` à partir du modèle. Pour utiliser une balise de valeur comme balise de marqueur, vous pouvez définir le champ `tagValue` sur la valeur de chaîne vide (`""`). 

Voici un exemple qui remplit la propriété `tags` de valeur pour trois jumeaux :

```csharp
entity-01: "tags": { "red": "", "size": "large" } 
entity-02: "tags": { "purple": "", "size": "small" } 
entity-03: "tags": { "red": "", "size": "small" } 
```

Notez que `red` et `purple` sont utilisés comme balises de marqueur dans cet exemple.

### <a name="query-with-value-tags"></a>Requête avec des balises de valeur

Comme avec les balises de marqueur, vous pouvez utiliser des balises de valeur pour filtrer les jumeaux dans des requêtes. Vous pouvez également utiliser des balises de valeur et des balises de marqueur ensemble.

Dans l’exemple ci-dessus, `red` est utilisé en tant que balise de marqueur. Voici une requête permettant d’obtenir tous les jumeaux qui sont rouges (marqués de la balise « red ») : 

```sql
SELECT * FROM digitaltwins WHERE is_defined(tags.red) 
```

Voici une requête permettant d’obtenir toutes les entités qui sont de petite taille (balise de valeur) et pas rouges : 

```sql
SELECT * FROM digitaltwins WHERE NOT is_defined(tags.red) AND tags.size = 'small' 
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus d’informations sur la conception et la gestion de modèles de jumeaux numériques :
* [*Guide pratique : Gérer des modèles personnalisés*](how-to-manage-model.md)

Découvrez plus d’informations sur l’interrogation du graphe de jumeaux :
* [*Guide pratique : Interroger le graphe de jumeaux*](how-to-query-graph.md)