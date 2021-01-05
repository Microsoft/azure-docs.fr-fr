---
title: Utiliser l'API de requête avec langage intégré JavaScript dans les procédures stockées et déclencheurs Azure Cosmos DB
description: Cet article présente les concepts de l’API de requête avec langage intégré JavaScript pour créer des procédures stockées et des déclencheurs dans Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/07/2020
ms.author: tisande
ms.reviewer: sngun
ms.custom: devx-track-js
ms.openlocfilehash: b2563a9af0e0ca6943059698e29d139143780d93
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340972"
---
# <a name="javascript-query-api-in-azure-cosmos-db"></a>API de requête JavaScript dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Outre l'émission de requêtes à l'aide de l'API SQL dans Azure Cosmos DB, le [kit SDK côté serveur Cosmos DB](https://github.com/Azure/azure-cosmosdb-js-server/) fournit une interface JavaScript pour exécuter des requêtes optimisées dans les procédures stockées et déclencheurs Cosmos DB. Vous n’êtes pas obligé de connaître le langage SQL pour utiliser cette interface JavaScript. L’API de requête JavaScript permet de créer des requêtes par programmation, en transmettant des fonctions de prédicat dans une séquence d’appels de fonction, avec une syntaxe usuelle des types prédéfinis de tableau ECMAScript5 et des bibliothèques JavaScript courantes, telles que Lodash. Les requêtes sont analysées par le runtime JavaScript et exécutées efficacement à l’aide d’index Azure Cosmos DB.

## <a name="supported-javascript-functions"></a>Fonctions JavaScript prises en charge

| **Fonction** | **Description** |
|---------|---------|
|`chain() ... .value([callback] [, options])`|Commence un appel chaîné qui doit se terminer par value().|
|`filter(predicateFunction [, options] [, callback])`|Filtre l’entrée à l’aide d’une fonction de prédicat qui renvoie true/false afin de filtrer les documents d’entrée dans le jeu résultant. Cette fonction se comporte comme une clause WHERE dans SQL.|
|`flatten([isShallow] [, options] [, callback])`|Combine et aplatit les tableaux à partir de chaque élément d’entrée en un seul tableau. Cette fonction se comporte comme SelectMany dans SQL.|
|`map(transformationFunction [, options] [, callback])`|Applique une projection à partir d’une fonction de transformation qui mappe chaque élément d’entrée à une valeur ou un objet JavaScript. Cette fonction se comporte comme une clause SELECT dans SQL.|
|`pluck([propertyName] [, options] [, callback])`|Cette fonction est un raccourci pour un mappage qui extrait la valeur d’une propriété unique de chaque élément d’entrée.|
|`sortBy([predicate] [, options] [, callback])`|Produit un nouvel ensemble de documents, en triant les documents dans le flux de documents d’entrée, dans l’ordre croissant avec le prédicat donné. Cette fonction se comporte comme une clause ORDER BY dans SQL.|
|`sortByDescending([predicate] [, options] [, callback])`|Produit un nouvel ensemble de documents, en triant les documents dans le flux de documents d’entrée, dans l’ordre décroissant avec le prédicat donné. Cette fonction se comporte comme une clause ORDER BY x DESC dans SQL.|
|`unwind(collectionSelector, [resultSelector], [options], [callback])`|Effectue une jointure réflexive avec tableau interne et ajoute les résultats des deux côtés, sous forme de tuples, à la projection de résultats. Par exemple, joindre un document person avec person.pets produirait des tuples [person, pet]. Ce comportement est semblable à SelectMany dans .NET LINK.|

Quand elles sont incluses dans les fonctions de prédicat et/ou de sélecteur, les constructions JavaScript suivantes sont automatiquement optimisées pour s’exécuter directement sur les index Azure Cosmos DB :

- Opérateurs simples : `=` `+` `-` `*` `/` `%` `|` `^` `&` `==` `!=` `===` `!===` `<` `>` `<=` `>=` `||` `&&` `<<` `>>` `>>>!` `~`
- Littéraux, notamment le littéral d’objet : {}
- var, return

Les constructions JavaScript suivantes ne sont pas optimisées pour les index Azure Cosmos DB :

- Flux de contrôle (par exemple, if, for, while)
- Appels de fonction

Pour plus d’informations, consultez la [documentation JavaScript Azure Cosmos DB côté serveur](https://github.com/Azure/azure-cosmosdb-js-server/).

## <a name="sql-to-javascript-cheat-sheet"></a>Aide-mémoire SQL vers JavaScript

Le tableau suivant présente différentes requêtes SQL et les requêtes JavaScript correspondantes. Comme pour les requêtes SQL, les propriétés (par exemple, item.id) respectent la casse.

> [!NOTE]
> `__` (trait de soulignement double) est un alias pour `getContext().getCollection()` lors de l’utilisation de l’API de requête JavaScript.

|**SQL**|**API de requête JavaScript**|**Description**|
|---|---|---|
|SELECT *<br>FROM docs| __.map(function(doc) { <br>&nbsp;&nbsp;&nbsp;&nbsp;return doc;<br>});|Renvoie tous les documents (paginés avec jeton de continuation) tels quels.|
|SELECT <br>&nbsp;&nbsp;&nbsp;docs.id,<br>&nbsp;&nbsp;&nbsp;docs.message AS msg,<br>&nbsp;&nbsp;&nbsp;docs.actions <br>FROM docs|__.map(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;return {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;id: doc.id,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;msg: doc.message,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;actions:doc.actions<br>&nbsp;&nbsp;&nbsp;&nbsp;};<br>});|Projette l’ID, le message (alias msg) et l’action de tous les documents.|
|SELECT *<br>FROM docs<br>WHERE<br>&nbsp;&nbsp;&nbsp;docs.id="X998_Y998"|__.filter(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;return doc.id ==="X998_Y998";<br>});|Requêtes pour les documents avec le prédicat : id = "X998_Y998".|
|SELECT *<br>FROM docs<br>WHERE<br>&nbsp;&nbsp;&nbsp;ARRAY_CONTAINS(docs.Tags, 123)|__.filter(function(x) {<br>&nbsp;&nbsp;&nbsp;&nbsp;return x.Tags &amp;& x.Tags.indexOf(123) &amp;gt; -1;<br>});|Requêtes pour les documents comportant une propriété Tags, et Tags est un tableau contenant la valeur 123.|
|SELECT<br>&nbsp;&nbsp;&nbsp;docs.id,<br>&nbsp;&nbsp;&nbsp;docs.message AS msg<br>FROM docs<br>WHERE<br>&nbsp;&nbsp;&nbsp;docs.id="X998_Y998"|__.chain()<br>&nbsp;&nbsp;&nbsp;&nbsp;.filter(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return doc.id ==="X998_Y998";<br>&nbsp;&nbsp;&nbsp;&nbsp;})<br>&nbsp;&nbsp;&nbsp;&nbsp;.map(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;id: doc.id,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;msg: doc.message<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;};<br>&nbsp;&nbsp;&nbsp;&nbsp;})<br>.value();|Interroge les documents avec un prédicat, id = "X998_Y998", puis projette l’ID et le message (alias msg).|
|SELECT VALUE tag<br>FROM docs<br>JOIN tag IN docs.Tags<br>ORDER BY docs._ts|__.chain()<br>&nbsp;&nbsp;&nbsp;&nbsp;.filter(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;return doc.Tags &amp;& Array.isArray(doc.Tags);<br>&nbsp;&nbsp;&nbsp;&nbsp;})<br>&nbsp;&nbsp;&nbsp;&nbsp;.sortBy(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return doc._ts;<br>&nbsp;&nbsp;&nbsp;&nbsp;})<br>&nbsp;&nbsp;&nbsp;&nbsp;.pluck("Tags")<br>&nbsp;&nbsp;&nbsp;&nbsp;.flatten()<br>&nbsp;&nbsp;&nbsp;&nbsp;.value()|Filtre les documents comportant une propriété de tableau, Tags, trie les documents résultants par la propriété système _ts timestamp, puis projette et aplatit le tableau Tags.|

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus de concepts, et comment écrire et utiliser procédures stockées, déclencheurs et fonctions définies par l’utilisateur dans Azure Cosmos DB :

- [Comment écrire des procédures stockées et des déclencheurs à l’aide de l’API de requête JavaScript](how-to-write-javascript-query-api.md)
- [Utilisation des procédures stockées, déclencheurs et fonctions définies par l’utilisateur dans Azure Cosmos DB](stored-procedures-triggers-udfs.md)
- [Guide pratique pour utiliser des procédures stockées, des déclencheurs et des fonctions définies par l’utilisateur dans Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md)
- [Référence d’API serveur Javascript Azure Cosmos DB](https://azure.github.io/azure-cosmosdb-js-server)
- [ES6 de JavaScript (ECMA 2015)](https://www.ecma-international.org/ecma-262/6.0/)
