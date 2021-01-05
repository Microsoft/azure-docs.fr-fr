---
title: Transformation de clé de substitution dans le flux de données de mappage
description: Comment utiliser une transformation de clé de substitution du mappage de flux de données Azure Data Factory pour générer des valeurs de clés séquentielles
author: kromerm
ms.author: makromer
ms.reviewer: daperlov
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 10/30/2020
ms.openlocfilehash: d1f8993b1adc297b1bfadba114df76a66e59afa2
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93147172"
---
# <a name="surrogate-key-transformation-in-mapping-data-flow"></a>Transformation de clé de substitution dans le flux de données de mappage 

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Utilisez la transformation de clé de substitution pour ajouter une valeur de clé incrémentielle à chaque ligne de données. Cette fonctionnalité est utile pour concevoir des tables de dimension dans un modèle de données analytiques suivant un schéma en étoile. Dans ce type de schéma, chacun des membres des tables de dimension a besoin d’une clé unique non commerciale.

## <a name="configuration"></a>Configuration

![Transformation de clé de substitution](media/data-flow/surrogate.png "Transformation de clé de substitution")

**Colonne clé :** nom de la colonne clé de substitution générée.

**Valeur de début :** la valeur de clé la plus faible qui sera générée.

## <a name="increment-keys-from-existing-sources"></a>Incrémenter les clés de sources existantes

Pour démarrer votre séquence à partir d’une valeur qui existe dans une source, nous vous recommandons d’utiliser un récepteur de cache pour enregistrer cette valeur et d’utiliser une transformation de colonne dérivée pour ajouter les deux valeurs ensemble. Utilisez une recherche mise en cache pour obtenir la sortie et l’ajouter à la clé générée. Pour plus d’informations, apprenez-en plus sur les [récepteurs de cache](data-flow-sink.md#cache-sink) et les [recherches mises en cache](concepts-data-flow-expression-builder.md#cached-lookup).

![Recherche de clé de substitution](media/data-flow/cached-lookup-example.png "Recherche de clé de substitution")

### <a name="increment-from-existing-maximum-value"></a>Incrémentation à partir d’une valeur maximale existante

La technique à appliquer pour amorcer la valeur de clé avec la valeur maximale précédente dépend de l’emplacement des données sources.

#### <a name="database-sources"></a>Sources de base de données

Utilisez une option de requête SQL pour sélectionner MAX() à partir de votre source. Par exemple, `Select MAX(<surrogateKeyName>) as maxval from <sourceTable>`.

![Requête de clé de substitution](media/data-flow/surrogate-key-max-database.png "Requête de transformation de clé de substitution")

#### <a name="file-sources"></a>Sources de fichiers

Si votre valeur maximale précédente se trouve dans un fichier, utilisez la fonction `max()` dans la transformation d’agrégation pour la récupérer :

![Fichier de clé de substitution](media/data-flow/surrogate-key-max-file.png "Fichier de clé de substitution")

Dans les deux cas, vous devrez écrire dans un récepteur de cache et rechercher la valeur. 


## <a name="data-flow-script"></a>Script de flux de données

### <a name="syntax"></a>Syntaxe

```
<incomingStream> 
    keyGenerate(
        output(<surrogateColumnName> as long),
        startAt: <number>L
    ) ~> <surrogateKeyTransformationName>
```

### <a name="example"></a>Exemple

![Transformation de clé de substitution](media/data-flow/surrogate.png "Transformation de clé de substitution")

Le flux de données correspondant à la configuration de clé de substitution ci-dessus est présenté dans l’extrait de code suivant.

```
AggregateDayStats
    keyGenerate(
        output(key as long),
        startAt: 1L
    ) ~> SurrogateKey1
```

## <a name="next-steps"></a>Étapes suivantes

Ces exemples utilisent les transformations [Joindre](data-flow-join.md) et [Colonne dérivée](data-flow-derived-column.md).
