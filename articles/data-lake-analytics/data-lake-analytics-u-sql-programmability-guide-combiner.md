---
title: Guide de programmabilité du combinateur défini par l’utilisateur U-SQL pour Azure Data Lake
description: En savoir plus sur le guide de programmabilité U-SQL UDO - combinateur défini par l’utilisateur.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: a6c560cf4ec11197183711656d69024591e7008c
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96512441"
---
# <a name="use-user-defined-combiner"></a>Utiliser un combinateur défini par l’utilisateur

## <a name="u-sql-udo-user-defined-combiner"></a>U-SQL UDO : combinateur défini par l’utilisateur
Un combinateur défini par l’utilisateur (ou UDC) vous permet de combiner des lignes des ensembles de lignes gauche et droit, selon une logique personnalisée. Un combinateur défini par l’utilisateur est utilisé avec l’expression COMBINE.

## <a name="how-to-define-and-use-user-defined-combiner"></a>Définition et utilisation d’un combinateur défini par l’utilisateur

Un combinateur est appelé avec l’expression COMBINE qui fournit les informations nécessaires sur les ensembles de lignes d’entrée, les colonnes de regroupement, le schéma de résultat attendu et des informations supplémentaires.

Pour appeler un combinateur dans un script U-SQL de base, nous utilisons la syntaxe suivante :

```usql
Combine_Expression :=
    'COMBINE' Combine_Input
    'WITH' Combine_Input
    Join_On_Clause
    Produce_Clause
    [Readonly_Clause]
    [Required_Clause]
    USING_Clause.
```

Pour plus d’informations, voir [Expression COMBINE (U-SQL)](/u-sql/statements-and-expressions/combine-expression).

Pour définir un combinateur défini par l’utilisateur, nous devons créer une interface `ICombiner` avec l’attribut [`SqlUserDefinedCombiner`], qui est facultatif pour une définition de combinateur défini par l’utilisateur.

Définition de classe `ICombiner` de base :

```csharp
public abstract class ICombiner : IUserDefinedOperator
{
protected ICombiner();
public virtual IEnumerable<IRow> Combine(List<IRowset> inputs,
       IUpdatableRow output);
public abstract IEnumerable<IRow> Combine(IRowset left, IRowset right,
       IUpdatableRow output);
}
```

L’implémentation personnalisée d’une interface `ICombiner` doit contenir la définition pour un remplacement de `IEnumerable<IRow>` Combine.

```csharp
[SqlUserDefinedCombiner]
public class MyCombiner : ICombiner
{

public override IEnumerable<IRow> Combine(IRowset left, IRowset right,
    IUpdatableRow output)
{
    …
}
}
```

L’attribut **SqlUserDefinedCombiner** indique que le type doit être inscrit en tant que combinateur défini par l’utilisateur. Cette classe ne peut pas être héritée.

**SqlUserDefinedCombiner** est utilisée pour définir la propriété du mode de combinateur. C’est un attribut facultatif pour une définition de combinateur défini par l’utilisateur.

CombinerMode     Mode

L’énumération CombinerMode peut prendre les valeurs suivantes :

* Complète (0) Chaque ligne de sortie dépend potentiellement de toutes les lignes d’entrée de gauche et de droite avec la même valeur de clé.

* Gauche (1) Chaque ligne de sortie dépend d’une seule ligne d’entrée de gauche (et potentiellement de toutes les lignes de droite avec la même valeur de clé).

* Droite (2) Chaque ligne de sortie dépend d’une seule ligne d’entrée de droite (et potentiellement de toutes les lignes de gauche avec la même valeur de clé).

* Interne (3) Chaque ligne de sortie dépend d’une seule ligne d’entrée de gauche et droite avec la même valeur.

Exemple :    [`SqlUserDefinedCombiner(Mode=CombinerMode.Left)`]


Les principaux objets de programmabilité sont :

```csharp
    public override IEnumerable<IRow> Combine(IRowset left, IRowset right,
        IUpdatableRow output
```

Les ensembles de lignes d’entrée sont transmis en tant que type d’interface **gauche** et **droit** `IRowset`. Les deux ensembles de lignes doivent être énumérés pour le traitement. Vous ne pouvez énumérer chaque interface qu’une seule fois. Il faut donc les énumérer et les mettre en cache si nécessaire.

Pour la mise en cache, nous pouvons créer un type List\<T\> de structure de mémoire résultant de l’exécution d’une requête LINQ, plus particulièrement List<`IRow`>. Le type de données anonymes peut également être utilisé pendant l’énumération.

Pour plus d’informations sur les requêtes LINQ, voir [Introduction aux requêtes LINQ (C#)](/dotnet/csharp/programming-guide/concepts/linq/introduction-to-linq-queries) et, pour plus d’informations sur l’interface IEnumerable\<T\>, voir [Interface IEnumerable\<T\>](/dotnet/api/system.collections.generic.ienumerable-1).

Pour obtenir les valeurs de données réelles du `IRowset` entrant, nous utilisons la méthode Get() de l’interface `IRow`.

```csharp
mycolumn = row.Get<int>("mycolumn")
```

Les noms des colonnes peuvent être déterminés en appelant la méthode de schéma `IRow`.

```csharp
ISchema schema = row.Schema;
var col = schema[i];
string val = row.Get<string>(col.Name)
```

Ou bien, en utilisant le nom de colonne de schéma :

```csharp
c# row.Get<int>(row.Schema[0].Name)
```

L’énumération générale avec LINQ prend la forme suivante :

```csharp
var myRowset =
            (from row in left.Rows
                          select new
                          {
                              Mycolumn = row.Get<int>("mycolumn"),
                          }).ToList();
```

Après l’énumération des deux ensembles de lignes, nous allons exécuter une boucle sur toutes les lignes. Pour chaque ligne de l’ensemble de lignes gauche, nous allons rechercher toutes les lignes remplissant la condition de notre combinateur.

Les valeurs de sortie doivent être définies avec la sortie `IUpdatableRow`.

```csharp
output.Set<int>("mycolumn", mycolumn)
```

La sortie réelle est déclenchée par l’appel de `yield return output.AsReadOnly();`.

Voici un exemple de combinateur :

```csharp
[SqlUserDefinedCombiner]
public class CombineSales : ICombiner
{

public override IEnumerable<IRow> Combine(IRowset left, IRowset right,
    IUpdatableRow output)
{
    var internetSales =
    (from row in left.Rows
          select new
          {
              ProductKey = row.Get<int>("ProductKey"),
              OrderDateKey = row.Get<int>("OrderDateKey"),
              SalesAmount = row.Get<decimal>("SalesAmount"),
              TaxAmt = row.Get<decimal>("TaxAmt")
          }).ToList();

    var resellerSales =
    (from row in right.Rows
     select new
     {
     ProductKey = row.Get<int>("ProductKey"),
     OrderDateKey = row.Get<int>("OrderDateKey"),
     SalesAmount = row.Get<decimal>("SalesAmount"),
     TaxAmt = row.Get<decimal>("TaxAmt")
     }).ToList();

    foreach (var row_i in internetSales)
    {
    foreach (var row_r in resellerSales)
    {

        if (
        row_i.OrderDateKey > 0
        && row_i.OrderDateKey < row_r.OrderDateKey
        && row_i.OrderDateKey == 20010701
        && (row_r.SalesAmount + row_r.TaxAmt) > 20000)
        {
        output.Set<int>("OrderDateKey", row_i.OrderDateKey);
        output.Set<int>("ProductKey", row_i.ProductKey);
        output.Set<decimal>("Internet_Sales_Amount", row_i.SalesAmount + row_i.TaxAmt);
        output.Set<decimal>("Reseller_Sales_Amount", row_r.SalesAmount + row_r.TaxAmt);
        }

    }
    }
    yield return output.AsReadOnly();
}
}
```

Dans ce scénario d’utilisation, nous créons un rapport analytique pour le détaillant. L’objectif est de rechercher tous les produits dont le prix est supérieur à 20 000 $, et dont la vente via le site web est plus rapide que via le détaillant standard au cours d’une période déterminée.

Voici le script U-SQL de base. Vous pouvez comparer la logique entre une commande JOIN standard et un combinateur :

```sql
DECLARE @LocalURI string = @"\usql-programmability\";

DECLARE @input_file_internet_sales string = @LocalURI+"FactInternetSales.txt";
DECLARE @input_file_reseller_sales string = @LocalURI+"FactResellerSales.txt";
DECLARE @output_file1 string = @LocalURI+"output_file1.tsv";
DECLARE @output_file2 string = @LocalURI+"output_file2.tsv";

@fact_internet_sales =
EXTRACT
    ProductKey int ,
    OrderDateKey int ,
    DueDateKey int ,
    ShipDateKey int ,
    CustomerKey int ,
    PromotionKey int ,
    CurrencyKey int ,
    SalesTerritoryKey int ,
    SalesOrderNumber String ,
    SalesOrderLineNumber  int ,
    RevisionNumber int ,
    OrderQuantity int ,
    UnitPrice decimal ,
    ExtendedAmount decimal,
    UnitPriceDiscountPct float ,
    DiscountAmount float ,
    ProductStandardCost decimal ,
    TotalProductCost decimal ,
    SalesAmount decimal ,
    TaxAmt decimal ,
    Freight decimal ,
    CarrierTrackingNumber String,
    CustomerPONumber String
FROM @input_file_internet_sales
USING Extractors.Text(delimiter:'|', encoding: Encoding.Unicode);

@fact_reseller_sales =
EXTRACT
    ProductKey int ,
    OrderDateKey int ,
    DueDateKey int ,
    ShipDateKey int ,
    ResellerKey int ,
    EmployeeKey int ,
    PromotionKey int ,
    CurrencyKey int ,
    SalesTerritoryKey int ,
    SalesOrderNumber String ,
    SalesOrderLineNumber  int ,
    RevisionNumber int ,
    OrderQuantity int ,
    UnitPrice decimal ,
    ExtendedAmount decimal,
    UnitPriceDiscountPct float ,
    DiscountAmount float ,
    ProductStandardCost decimal ,
    TotalProductCost decimal ,
    SalesAmount decimal ,
    TaxAmt decimal ,
    Freight decimal ,
    CarrierTrackingNumber String,
    CustomerPONumber String
FROM @input_file_reseller_sales
USING Extractors.Text(delimiter:'|', encoding: Encoding.Unicode);

@rs1 =
SELECT
    fis.OrderDateKey,
    fis.ProductKey,
    fis.SalesAmount+fis.TaxAmt AS Internet_Sales_Amount,
    frs.SalesAmount+frs.TaxAmt AS Reseller_Sales_Amount
FROM @fact_internet_sales AS fis
     INNER JOIN @fact_reseller_sales AS frs
     ON fis.ProductKey == frs.ProductKey
WHERE
    fis.OrderDateKey < frs.OrderDateKey
    AND fis.OrderDateKey == 20010701
    AND frs.SalesAmount+frs.TaxAmt > 20000;

@rs2 =
COMBINE @fact_internet_sales AS fis
WITH @fact_reseller_sales AS frs
ON fis.ProductKey == frs.ProductKey
PRODUCE OrderDateKey int,
        ProductKey int,
        Internet_Sales_Amount decimal,
        Reseller_Sales_Amount decimal
USING new USQL_Programmability.CombineSales();

OUTPUT @rs1 TO @output_file1 USING Outputters.Tsv();
OUTPUT @rs2 TO @output_file2 USING Outputters.Tsv();
```

Un combinateur défini par l’utilisateur peut être appelé en tant que nouvelle instance de l’objet applicateur :

```csharp
USING new MyNameSpace.MyCombiner();
```


Ou par l’appel d’une méthode de fabrique de wrapper :

```csharp
USING MyNameSpace.MyCombiner();
```

## <a name="next-steps"></a>Étapes suivantes
* [Guide de programmabilité U-SQL - vue d’ensemble](data-lake-analytics-u-sql-programmability-guide.md)
* [Guide de programmabilité U-SQL - UDT et UDAGG](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)