---
title: Langage de requête
titleSuffix: Azure Digital Twins
description: Découvrez les principes de base du langage des requêtes Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 11/19/2020
ms.topic: conceptual
ms.service: digital-twins
ms.custom: contperf-fy21q2
ms.openlocfilehash: 742cff544886a1499bccfa575684edef708da7bd
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97028357"
---
# <a name="about-the-query-language-for-azure-digital-twins"></a>À propos du langage de requête pour Azure Digital Twins

N’oubliez pas que le centre d’Azure Digital Twins est le [graphe de jumeaux](concepts-twins-graph.md) construit à partir des jumeaux numériques et des relations. 

Ce graphique peut être interrogé pour obtenir des informations sur les jumeaux numériques et les relations qu’il contient. Ces requêtes sont écrites dans un langage de requête de type SQL personnalisé, appelé **langage des requêtes Azure Digital Twins**. Ce langage est similaire au [langage de requête IoT Hub](../iot-hub/iot-hub-devguide-query-language.md) par ses nombreuses fonctionnalités comparables.

Cet article décrit les principes fondamentaux du langage de requête et de ses fonctionnalités. Pour obtenir des exemples plus détaillés de syntaxe de requête et savoir comment exécuter des requêtes, consultez [*Guide pratique : Interroger le graphique de jumeaux*](how-to-query-graph.md).

## <a name="about-the-queries"></a>À propos des requêtes

Vous pouvez utiliser le langage de requête Azure Digital Twins pour récupérer des jumeaux numériques en fonction de leurs...
* leurs propriétés (y compris les [propriétés de balise](how-to-use-tags.md))
* modèles
* relationships
  - leurs propriétés des relations

Pour soumettre une requête au service à partir d’une application cliente, vous allez utiliser l’[**API de requête**](/rest/api/digital-twins/dataplane/query) Azure Digital Twins. L’une des façons d’utiliser l’API consiste à utiliser l’un des [SDK](how-to-use-apis-sdks.md#overview-data-plane-apis) pour Azure Digital Twins.

## <a name="reference-expressions-and-conditions"></a>Référence : Expressions et conditions

Cette section décrit les opérateurs et les fonctions disponibles pour écrire des requêtes Azure Digital Twins. Pour obtenir des exemples de requête illustrant l’utilisation de ces fonctionnalités, consultez [*Guide pratique : Interroger le graphique de jumeaux*](how-to-query-graph.md).

> [!NOTE]
> Toutes les opérations de requête Azure Digital Twins sont sensibles à la casse. Veillez donc à utiliser les noms exacts définis dans les modèles. Si les noms des propriétés sont mal orthographiés ou ne respectent pas la casse, le jeu de résultats sera vide et aucune erreur ne sera retournée.

### <a name="operators"></a>Opérateurs

Les opérateurs suivants sont pris en charge :

| Famille | Opérateurs |
| --- | --- |
| Logical |`AND`, `OR`, `NOT` |
| Comparaison | `=`, `!=`, `<`, `>`, `<=`, `>=` |
| Contient | `IN`, `NIN` |

### <a name="functions"></a>Fonctions

Les fonctions de vérification et de conversion de type suivantes sont prises en charge :

| Fonction | Description |
| -------- | ----------- |
| `IS_DEFINED` | Retourne une valeur booléenne indiquant si une valeur a été attribuée à la propriété. Uniquement pris en charge lorsque la valeur est de type primitif. Exemples de types primitifs : chaîne, booléen, numérique ou `null`. `DateTime`, les types d’objets et les tableaux ne sont pas pris en charge. |
| `IS_OF_MODEL` | Retourne une valeur booléenne indiquant si le jumeau spécifié correspond au type de modèle spécifié. |
| `IS_BOOL` | Retourne une valeur booléenne indiquant si l’expression spécifiée est du type booléen. |
| `IS_NUMBER` | Retourne une valeur booléenne indiquant si l’expression spécifiée est du type nombre. |
| `IS_STRING` | Retourne une valeur booléenne indiquant si l’expression spécifiée est du type chaîne. |
| `IS_NULL` | Retourne une valeur booléenne indiquant si l’expression spécifiée est de type null. |
| `IS_PRIMITIVE` | Retourne une valeur booléenne indiquant si l’expression spécifiée est de type primitif (chaîne, booléen, numérique ou `null`). |
| `IS_OBJECT` | Retourne une valeur booléenne indiquant si l’expression spécifiée est du type objet JSON. |

Les fonctions de chaîne suivantes sont prises en charge :

| Fonction | Description |
| -------- | ----------- |
| `STARTSWITH(x, y)` | Retourne une valeur booléenne indiquant si la première expression de chaîne commence par la seconde. |
| `ENDSWITH(x, y)` | Retourne une valeur booléenne indiquant si la première expression de chaîne se termine par la seconde. |

## <a name="query-limitations"></a>Limitations des requêtes

Cette section décrit les limitations du langage de requête.

* Synchronisation : Il peut se passer jusqu’à 10 secondes avant que les modifications de votre instance ne soient reflétées dans les requêtes. Par exemple, si vous effectuez une opération telle que la création ou la suppression de jumeaux avec l’API DigitalTwins, le résultat peut ne pas apparaître immédiatement dans les requêtes de l’API de requête. Il suffit d’attendre quelques instants pour que le résultat apparaisse.
* Aucune sous-requête n’est prise en charge dans l’instruction `FROM`.
* La sémantique `OUTER JOIN` n’est pas prise en charge, ce qui signifie que si la relation a un rang égal à zéro, la « ligne » entière sera éliminée du jeu de résultats de sortie.
* La profondeur de la traversée du graphe est restreinte à cinq niveaux `JOIN` autorisés par requête.
* La source des opérations `JOIN` est restreinte : la requête doit déclarer les jumeaux au début.

## <a name="next-steps"></a>Étapes suivantes

Apprenez à écrire des requêtes et consultez des exemples de codes clients dans [*Guide pratique : Interroger le graphique de jumeaux*](how-to-query-graph.md).