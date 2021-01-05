---
title: Filtrage d’événements pour Azure Event Grid
description: Explique comment filtrer des événements lors de la création d’un abonnement Azure Event Grid.
ms.topic: conceptual
ms.date: 12/03/2020
ms.openlocfilehash: bc3e84037693fcd909961ba409871d947ef1de7d
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574904"
---
# <a name="understand-event-filtering-for-event-grid-subscriptions"></a>Comprendre le filtrage d’événements pour les abonnements Event Grid

Cet article décrit les différentes façons de filtrer les événements qui sont envoyés à votre point de terminaison. Quand vous créez un abonnement à un événement, trois options de filtrage s’offrent à vous :

* Types d’événements
* Objet commençant ou se terminant par
* Opérateurs et champs avancés

## <a name="event-type-filtering"></a>Filtrage par type d’événement

Par défaut, tous les [types d’événements](event-schema.md) pour la source de l’événement sont envoyés au point de terminaison. Vous pouvez décider d’envoyer uniquement certains types d’événements à votre point de terminaison. Par exemple, vous pouvez être notifié des mises à jour de vos ressources, mais ne pas être notifié d’autres opérations comme les suppressions. Dans ce cas, effectuez le filtrage par type d’événement `Microsoft.Resources.ResourceWriteSuccess`. Fournissez un tableau avec les types d’événements, ou spécifiez `All` pour obtenir tous les types d’événements de la source de l’événement.

La syntaxe JSON pour le filtrage par type d’événement est :

```json
"filter": {
  "includedEventTypes": [
    "Microsoft.Resources.ResourceWriteFailure",
    "Microsoft.Resources.ResourceWriteSuccess"
  ]
}
```

## <a name="subject-filtering"></a>Filtrage par objet

Pour le filtrage simple par objet, spécifiez une valeur de départ ou de fin pour l’objet. Par exemple, vous pouvez spécifier que l’objet se termine par `.txt` pour obtenir uniquement les événements liés au chargement d’un fichier texte sur le compte de stockage. Ou vous pouvez filtrer sur l’objet commençant par `/blobServices/default/containers/testcontainer` pour obtenir tous les événements pour ce conteneur, mais pas pour d’autres conteneurs du compte de stockage.

Lorsque vous publiez des événements dans des rubriques personnalisées, créez des objets pour vos événements permettant aux abonnés de savoir facilement si l’événement les intéresse. Les abonnés utilisent la propriété d’objet pour filtrer et router des événements. Envisagez d’ajouter le chemin à l’origine de l’événement, de sorte que les abonnés puissent filtrer sur des segments de ce chemin. Le chemin d’accès permet aux abonnés de filtrer les événements avec précision ou à grande échelle. Si vous fournissez un chemin de trois segments comme `/A/B/C` dans l’objet, les abonnés peuvent filtrer sur le premier segment `/A` pour obtenir un vaste ensemble d’événements. Ces abonnés obtiennent des événements avec des objets tels que `/A/B/C` ou `/A/D/E`. Les autres abonnés peuvent filtrer par `/A/B` pour obtenir un ensemble plus restreint d’événements.

La syntaxe JSON pour le filtrage par objet est :

```json
"filter": {
  "subjectBeginsWith": "/blobServices/default/containers/mycontainer/log",
  "subjectEndsWith": ".jpg"
}

```

## <a name="advanced-filtering"></a>Filtrage avancé

Pour filtrer sur des valeurs dans les champs de données et spécifier l’opérateur de comparaison, utilisez l’option de filtrage avancé. Dans le filtrage avancé, vous spécifiez les éléments suivants :

* type d'opérateur : type de comparaison.
* clé : champ des données d’événement que vous utilisez pour le filtrage. Ce peut être un nombre, une valeur booléenne ou une chaîne.
* valeurs : valeur ou valeurs à comparer à la clé.

Si vous spécifiez un seul filtre avec plusieurs valeurs, une opération **OU** est effectuée : la valeur du champ clé doit donc être une de ces valeurs. Voici un exemple :

```json
"advancedFilters": [
    {
        "operatorType": "StringContains",
        "key": "Subject",
        "values": [
            "/providers/microsoft.devtestlab/",
            "/providers/Microsoft.Compute/virtualMachines/"
        ]
    }
]
```

Si vous spécifiez plusieurs filtres différents, une opération **ET** est effectuée : chaque condition du filtre doit donc être satisfaite. Voici un exemple : 

```json
"advancedFilters": [
    {
        "operatorType": "StringContains",
        "key": "Subject",
        "values": [
            "/providers/microsoft.devtestlab/"
        ]
    },
    {
        "operatorType": "StringContains",
        "key": "Subject",
        "values": [
            "/providers/Microsoft.Compute/virtualMachines/"
        ]
    }
]
```

### <a name="operators"></a>Opérateurs

Les opérateurs disponibles pour les **nombres** sont les suivants :

* NumberGreaterThan
* NumberGreaterThanOrEquals
* NumberLessThan
* NumberLessThanOrEquals
* NumberIn
* NumberNotIn

L’opérateur disponible pour les valeurs **booléennes** est : 
- BoolEquals

Les opérateurs disponibles pour les **chaînes** sont les suivants :

* StringContains
* StringBeginsWith
* StringEndsWith
* StringIn
* StringNotIn

Les comparaisons de chaînes ne sont **pas toutes** sensibles à la casse.

> [!NOTE]
> Si le JSON de l’événement ne contient pas de clé de filtre avancé, le filtre est évalué comme **non correspondant** pour les opérateurs suivants : 
> - NumberGreaterThan
> - NumberGreaterThanOrEquals
> - NumberLessThan
> - NumberLessThanOrEquals
> - NumberIn
> - BoolEquals
> - StringContains
> - StringBeginsWith
> - StringEndsWith
> - StringIn
> 
>Le filtre est évalué comme **correspondant** pour les opérateurs suivants :
> - NumberNotIn
> - StringNotIn

### <a name="key"></a>Clé

Pour les événements dans le schéma Event Grid, utilisez les valeurs suivantes pour la clé :

* id
* Rubrique
* Objet
* Type d’événement
* DataVersion
* Données d’événement (comme Data.key1)

Pour les événements dans le schéma des événements cloud, utilisez les valeurs suivantes pour la clé :

* EventId
* Source
* Type d’événement
* EventTypeVersion
* Données d’événement (comme Data.key1)

Pour le schéma d’entrée personnalisé, utilisez les champs de données d’événement (comme Data.key1).

### <a name="values"></a>Valeurs

Les valeurs peuvent être les suivantes :

* nombre
* string
* boolean
* tableau

### <a name="limitations"></a>Limites

Le filtrage avancé présente les limites suivantes :

* 5 filtres avancés et 25 valeurs de filtre pour tous les filtres par abonnement Event Grid
* 512 caractères par valeur de type chaîne
* Cinq valeurs pour les opérateurs **dans** et **pas dans**
* Clés contenant le signe **`.` (point)** . Par exemple, `http://schemas.microsoft.com/claims/authnclassreference` ou `john.doe@contoso.com`. Actuellement, les caractères d’échappement ne sont pas pris en charge dans les clés. 

La même clé peut être utilisée dans plusieurs filtres.

### <a name="examples"></a>Exemples

### <a name="stringcontains"></a>StringContains

```json
"advancedFilters": [{
    "operatorType": "StringContains",
    "key": "data.key1",
    "values": [
        "microsoft", 
        "azure"
    ]
}]
```

### <a name="stringbeginswith"></a>StringBeginsWith

```json
"advancedFilters": [{
    "operatorType": "StringBeginsWith",
    "key": "data.key1",
    "values": [
        "event", 
        "grid"
    ]
}]
```

### <a name="stringendswith"></a>StringEndsWith

```json
"advancedFilters": [{
    "operatorType": "StringEndsWith",
    "key": "data.key1",
    "values": [
        "jpg", 
        "jpeg", 
        "png"
    ]
}]
```

### <a name="stringin"></a>StringIn

```json
"advancedFilters": [{
    "operatorType": "StringIn",
    "key": "data.key1",
    "values": [
        "exact", 
        "string", 
        "matches"
    ]
}]
```

### <a name="stringnotin"></a>StringNotIn

```json
"advancedFilters": [{
    "operatorType": "StringNotIn",
    "key": "data.key1",
    "values": [
        "aws", 
        "bridge"
    ]
}]
```

### <a name="numberin"></a>NumberIn

```json

"advancedFilters": [{
    "operatorType": "NumberIn",
    "key": "data.counter",
    "values": [
        5,
        1
    ]
}]

```

### <a name="numbernotin"></a>NumberNotIn

```json
"advancedFilters": [{
    "operatorType": "NumberNotIn",
    "key": "data.counter",
    "values": [
        41,
        0,
        0
    ]
}]
```

### <a name="numberlessthan"></a>NumberLessThan

```json
"advancedFilters": [{
    "operatorType": "NumberLessThan",
    "key": "data.counter",
    "value": 100
}]
```

### <a name="numbergreaterthan"></a>NumberGreaterThan

```json
"advancedFilters": [{
    "operatorType": "NumberGreaterThan",
    "key": "data.counter",
    "value": 20
}]
```

### <a name="numberlessthanorequals"></a>NumberLessThanOrEquals

```json
"advancedFilters": [{
    "operatorType": "NumberLessThanOrEquals",
    "key": "data.counter",
    "value": 100
}]
```

### <a name="numbergreaterthanorequals"></a>NumberGreaterThanOrEquals

```json
"advancedFilters": [{
    "operatorType": "NumberGreaterThanOrEquals",
    "key": "data.counter",
    "value": 30
}]
```

### <a name="boolequals"></a>BoolEquals

```json
"advancedFilters": [{
    "operatorType": "BoolEquals",
    "key": "data.isEnabled",
    "value": true
}]
```


## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur le filtrage d’événements avec PowerShell et Azure CLI, consultez [Filtrer des événements pour Event Grid](how-to-filter-events.md).
* Pour une prise en main rapide d’Event Grid, consultez [Créer et acheminer des événements personnalisés avec Azure Event Grid](custom-event-quickstart.md).
