---
title: Conversion LINQ en SQL dans Azure Cosmos DB
description: Découvrez les opérateurs LINQ pris en charge et la façon dont les requêtes LINQ sont mappées aux requêtes SQL dans Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 11/11/2020
ms.author: tisande
ms.openlocfilehash: 122c95fe9ac017ad7a6957dcdb8323837be34f21
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96545381"
---
# <a name="linq-to-sql-translation"></a>Conversion LINQ en SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Le fournisseur de requêtes d’Azure Cosmos DB effectue le meilleur mappage possible entre une requête LINQ et une requête SQL Cosmos DB. Si vous souhaitez obtenir la requête SQL traduite à partir de LINQ, utilisez la méthode `ToString()` sur l’objet `IQueryable` généré. La description suivante suppose une connaissance de base de [LINQ](/dotnet/csharp/programming-guide/concepts/linq/introduction-to-linq-queries). En plus de LINQ, Azure Cosmos DB prend également en charge [Entity Framework Core](/ef/core/providers/cosmos/?tabs=dotnet-core-cli) qui fonctionne avec l’API SQL.

Le système de type du fournisseur de requêtes prend en charge uniquement les types primitifs JSON : numérique, booléen, chaîne et null.

Le fournisseur de requêtes prend en charge les expressions scalaires suivantes :

- Valeurs constantes, y compris les valeurs constantes des types de données primitifs au moment de l’évaluation de la requête.
  
- Expressions d’index de propriété/tableau qui font référence à la propriété d’un objet ou d’un élément de tableau. Par exemple :
  
  ```
    family.Id;
    family.children[0].familyName;
    family.children[0].grade;
    family.children[n].grade; //n is an int variable
  ```
  
- Expressions arithmétiques, y compris les expressions arithmétiques communes sur les valeurs numériques et booléennes. Pour obtenir la liste complète, consultez la [spécification SQL Azure Cosmos DB](sql-query-aggregate-functions.md).
  
  ```
    2 * family.children[0].grade;
    x + y;
  ```
  
- Expressions de comparaison de chaîne, qui incluent les comparaisons de valeurs de chaînes à certaines valeurs de chaînes constantes.  
  
  ```
    mother.familyName == "Wakefield";
    child.givenName == s; //s is a string variable
  ```
  
- Expressions de création d’objet/tableau, qui retournent un objet de type de valeur composée ou de type anonyme ou un tableau de tels objets. Vous pouvez imbriquer ces valeurs.
  
  ```
    new Parent { familyName = "Wakefield", givenName = "Robin" };
    new { first = 1, second = 2 }; //an anonymous type with two fields  
    new int[] { 3, child.grade, 5 };
  ```

## <a name="using-linq"></a>Utilisation de LINQ

Vous pouvez créer une requête LINQ à l’aide de `GetItemLinqQueryable`. Cet exemple illustre la génération et l’exécution asynchrone de requêtes LINQ avec un `FeedIterator` :

```csharp
using (FeedIterator<Book> setIterator = container.GetItemLinqQueryable<Book>()
                      .Where(b => b.Title == "War and Peace")
                      .ToFeedIterator<Book>())
 {
     //Asynchronous query execution
     while (setIterator.HasMoreResults)
     {
         foreach(var item in await setIterator.ReadNextAsync()){
         {
             Console.WriteLine(item.cost);
         }
       }
     }
 }
```

## <a name="supported-linq-operators"></a><a id="SupportedLinqOperators"></a>Opérateurs LINQ pris en charge

Le fournisseur LINQ inclus avec le kit SDK .NET SQL prend en charge les opérateurs suivants :

- **Select** : les projections sont traduites en [SELECT](sql-query-select.md), y compris la construction d’objets.
- **Where** : les filtres sont traduits en [WHERE](sql-query-where.md) et prennent en charge la traduction de `&&`, `||` et `!` en opérateurs SQL
- **SelectMany** : autorise le déroulement de tableaux vers la clause [JOIN](sql-query-join.md). Permet d’associer/imbriquer des expressions afin de filtrer les éléments de tableau.
- **OrderBy** et **OrderByDescending** : sont traduits en [ORDER BY](sql-query-order-by.md) avec ASC ou DESC.
- Les opérateurs **Count**, **Sum**, **Min**, **Max** et **Average** pour l’[agrégation](sql-query-aggregate-functions.md), et leurs équivalents asynchrones **CountAsync**, **SumAsync**, **MinAsync**, **MaxAsync** et **AverageAsync**.
- **CompareTo** : se traduit par des comparaisons de plages. Généralement utilisé pour les chaînes, car elles ne sont pas comparables en .NET.
- **Skip** et **Take** : traduit en [OFFSET et LIMIT](sql-query-offset-limit.md) afin de limiter les résultats d’une requête et d’effectuer la pagination.
- **Fonctions mathématiques** : prennent en charge la traduction des fonctions .NET `Abs`, `Acos`, `Asin`, `Atan`, `Ceiling`, `Cos`, `Exp`, `Floor`, `Log`, `Log10`, `Pow`, `Round`, `Sign`, `Sin`, `Sqrt`, `Tan` et `Truncate` vers les [fonctions mathématiques intégrées](sql-query-mathematical-functions.md) équivalentes.
- **Fonctions de chaîne** : prennent en charge la traduction des fonctions .NET `Concat`, `Contains`, `Count`, `EndsWith`, `IndexOf`, `Replace`, `Reverse`, `StartsWith`, `SubString`, `ToLower`, `ToUpper`, `TrimEnd` et `TrimStart` vers les [fonctions de chaîne intégrées](sql-query-string-functions.md) équivalentes.
- **Fonctions de tableau** : prennent en charge la traduction des fonctions .NET `Concat`, `Contains` et `Count` vers les [fonctions de tableau intégrées](sql-query-array-functions.md) équivalentes.
- **Fonctions d’extension géospatiale** : prennent en charge la traduction des méthodes stub `Distance`, `IsValid`, `IsValidDetailed` et `Within` vers les [fonctions géospatiales intégrées](sql-query-geospatial-query.md) équivalentes.
- **Fonction d’extension de fonction définie par l’utilisateur** : prend en charge la traduction de la méthode stub `UserDefinedFunctionProvider.Invoke` vers la [fonction définie par l’utilisateur](sql-query-udfs.md) correspondante.
- **Miscellaneous** : prend en charge la traduction de `Coalesce` et des [opérateurs](sql-query-operators.md) conditionnels. Peut traduire `Contains` en chaîne CONTAINS, ARRAY_CONTAINS ou IN, selon le contexte.

## <a name="examples"></a>Exemples

Les exemples suivants illustrent la traduction de certains opérateurs standard de requête LINQ en requêtes dans Azure Cosmos DB.

### <a name="select-operator"></a>Opérateur Select

La syntaxe est `input.Select(x => f(x))`, où `f` est une expression scalaire. Dans ce cas, `input` est un objet `IQueryable`.

**Opérateur Select, exemple 1 :**

- **Expression Lambda LINQ**
  
  ```csharp
      input.Select(family => family.parents[0].familyName);
  ```
  
- **SQL**
  
  ```sql
      SELECT VALUE f.parents[0].familyName
      FROM Families f
    ```
  
**Opérateur Select, exemple 2 :**

- **Expression Lambda LINQ**
  
  ```csharp
      input.Select(family => family.children[0].grade + c); // c is an int variable
  ```
  
- **SQL**
  
  ```sql
      SELECT VALUE f.children[0].grade + c
      FROM Families f
  ```
  
**Opérateur Select, exemple 3 :**

- **Expression Lambda LINQ**
  
  ```csharp
    input.Select(family => new
    {
        name = family.children[0].familyName,
        grade = family.children[0].grade + 3
    });
  ```
  
- **SQL**
  
  ```sql
      SELECT VALUE {"name":f.children[0].familyName,
                    "grade": f.children[0].grade + 3 }
      FROM Families f
  ```

### <a name="selectmany-operator"></a>Opérateur SelectMany

La syntaxe est `input.SelectMany(x => f(x))`, où `f` est une expression scalaire qui retourne un type de conteneur.

- **Expression Lambda LINQ**
  
  ```csharp
      input.SelectMany(family => family.children);
  ```
  
- **SQL**

  ```sql
      SELECT VALUE child
      FROM child IN Families.children
  ```

### <a name="where-operator"></a>Opérateur Where

La syntaxe est `input.Where(x => f(x))`, où `f` est une expression scalaire qui retourne une valeur booléenne.

**Opérateur Where, exemple 1 :**

- **Expression Lambda LINQ**
  
  ```csharp
      input.Where(family=> family.parents[0].familyName == "Wakefield");
  ```
  
- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      WHERE f.parents[0].familyName = "Wakefield"
  ```
  
**Opérateur Where, exemple 2 :**

- **Expression Lambda LINQ**
  
  ```csharp
      input.Where(
          family => family.parents[0].familyName == "Wakefield" &&
          family.children[0].grade < 3);
  ```
  
- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      WHERE f.parents[0].familyName = "Wakefield"
      AND f.children[0].grade < 3
  ```

## <a name="composite-sql-queries"></a>Requêtes SQL composites

Vous pouvez composer les opérateurs précédents pour former des requêtes plus puissantes. Étant donné que Cosmos DB prend en charge les conteneurs imbriqués, vous pouvez concaténer ou imbriquer la composition.

### <a name="concatenation"></a>Concaténation

La syntaxe est `input(.|.SelectMany())(.Select()|.Where())*`. Une requête concaténée peut commencer par une requête `SelectMany` facultative, suivie de plusieurs opérateurs `Select` ou `Where`.

**Concaténation, exemple 1 :**

- **Expression Lambda LINQ**
  
  ```csharp
      input.Select(family => family.parents[0])
          .Where(parent => parent.familyName == "Wakefield");
  ```

- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      WHERE f.parents[0].familyName = "Wakefield"
  ```

**Concaténation, exemple 2 :**

- **Expression Lambda LINQ**
  
  ```csharp
      input.Where(family => family.children[0].grade > 3)
          .Select(family => family.parents[0].familyName);
  ```

- **SQL**
  
  ```sql
      SELECT VALUE f.parents[0].familyName
      FROM Families f
      WHERE f.children[0].grade > 3
  ```

**Concaténation, exemple 3 :**

- **Expression Lambda LINQ**
  
  ```csharp
      input.Select(family => new { grade=family.children[0].grade}).
          Where(anon=> anon.grade < 3);
  ```
  
- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      WHERE ({grade: f.children[0].grade}.grade > 3)
  ```

**Concaténation, exemple 4 :**

- **Expression Lambda LINQ**
  
  ```csharp
      input.SelectMany(family => family.parents)
          .Where(parent => parents.familyName == "Wakefield");
  ```
  
- **SQL**
  
  ```sql
      SELECT *
      FROM p IN Families.parents
      WHERE p.familyName = "Wakefield"
  ```

### <a name="nesting"></a>Imbrication

La syntaxe est `input.SelectMany(x=>x.Q())`, où `Q` est un opérateur `Select`, `SelectMany` ou `Where`.

Une requête imbriquée applique la requête interne à chaque élément du conteneur externe. Une caractéristique importante est la capacité de la requête interne à faire référence aux champs des éléments du conteneur externe, à l’instar d’une jointure réflexive.

**Imbrication, exemple 1 :**

- **Expression Lambda LINQ**
  
  ```csharp
      input.SelectMany(family=>
          family.parents.Select(p => p.familyName));
  ```

- **SQL**
  
  ```sql
      SELECT VALUE p.familyName
      FROM Families f
      JOIN p IN f.parents
  ```

**Imbrication, exemple 2 :**

- **Expression Lambda LINQ**
  
  ```csharp
      input.SelectMany(family =>
          family.children.Where(child => child.familyName == "Jeff"));
  ```

- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      JOIN c IN f.children
      WHERE c.familyName = "Jeff"
  ```

**Imbrication, exemple 3 :**

- **Expression Lambda LINQ**
  
  ```csharp
      input.SelectMany(family => family.children.Where(
          child => child.familyName == family.parents[0].familyName));
  ```

- **SQL**
  
  ```sql
      SELECT *
      FROM Families f
      JOIN c IN f.children
      WHERE c.familyName = f.parents[0].familyName
  ```

## <a name="next-steps"></a>Étapes suivantes

- [Exemples .NET Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Modéliser des données de document](modeling-data.md)