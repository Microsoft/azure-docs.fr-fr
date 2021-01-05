---
title: Date_Bucket (Transact-SQL) - Azure SQL Edge
description: Découvrez comment utiliser Date_Bucket dans Azure SQL Edge
keywords: Date_Bucket, SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: reference
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/03/2020
ms.openlocfilehash: 9d81419721e94a2e181f094c0e0e64b1b23544a8
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93073517"
---
# <a name="date_bucket-transact-sql"></a>Date_Bucket (Transact-SQL)

Cette fonction retourne la valeur datetime correspondant au début de chaque compartiment datetime, à partir de l’horodatage défini par le paramètre `origin` ou la valeur d’origine par défaut de `1900-01-01 00:00:00.000` si le paramètre Origin n’est pas spécifié. 

Consultez [Types de données et fonctions de date et d'heure &#40;Transact-SQL&#41;](/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql/) pour obtenir une vue d'ensemble de tous les types de données et fonctions de date et d'heure.

[Conventions syntaxiques de Transact-SQL](/sql/t-sql/language-elements/transact-sql-syntax-conventions-transact-sql/)

## <a name="syntax"></a>Syntaxe

```sql
DATE_BUCKET (datePart, number, date, origin)
```

## <a name="arguments"></a>Arguments

*datePart*

Partie de la *date* utilisée avec le paramètre « number ». Ex. Année, mois, minute, seconde, etc.

> [!NOTE]
> `DATE_BUCKET` n’accepte pas d’équivalents de variables définis par l’utilisateur pour les arguments *datePart*.
  
|*datePart*|Abréviations|  
|---|---|
|**day**|**dd** , **d**|  
|**week**|**wk** , **ww**| 
|**month**|**mm** , **m**|
|**quarter**|**qq** , **q**|  
|**year**|**yy** , **yyyy**|  
|**hour**|**hh**|  
|**minute**|**mi** , **n**|  
|**second**|**ss** , **s**|  
|**millisecond**|**ms**|  

*number*

Nombre entier qui détermine la largeur du compartiment combiné avec l'argument *datePart*. Cela correspond à la largeur des compartiments dataPart à partir de l’heure d’origine. **`This argument cannot be a negative integer value`**. 

*date*

Expression qui peut être résolue en valeur, parmi les suivantes :

+ **date**
+ **datetime**
+ **datetimeoffset**
+ **datetime2**
+ **smalldatetime**
+ **time**

Pour *date* , `DATE_BUCKET` accepte une expression de colonne, une expression ou une variable définie par l’utilisateur si elle est résolue en l’un des types de données mentionnés ci-dessus.

**Origine** 

Expression facultative qui peut être résolue en l’une des valeurs suivantes :

+ **date**
+ **datetime**
+ **datetimeoffset**
+ **datetime2**
+ **smalldatetime**
+ **time**

Le type de données pour `Origin` doit correspondre au type de données du paramètre `Date`. 

`DATE_BUCKET` utilise une valeur de date d’origine par défaut de `1900-01-01 00:00:00.000` (soit minuit, lundi 1er janvier 1900), si aucune valeur Origin n’est spécifiée pour la fonction.

## <a name="return-type"></a>Type de retour

Le type de données de la valeur de retour pour cette méthode est dynamique. Le type de retour dépend de l’argument fourni pour `date`. Si un type de données d’entrée valide est fourni pour `date`, `DATE_BUCKET` retourne le même type de données. `DATE_BUCKET` génère une erreur si un littéral de chaîne est spécifié pour le paramètre `date`.

## <a name="return-values"></a>Valeurs de retour

### <a name="understanding-the-output-from-date_bucket"></a>Présentation de la sortie depuis `DATE_BUCKET`

`Date_Bucket` retourne la dernière valeur de date ou d’heure correspondant au paramètre datePart et number. Par exemple, dans les expressions ci-dessous, `Date_Bucket` retourne la valeur de sortie de `2020-04-13 00:00:00.0000000`, car la sortie est calculée en fonction des compartiments d’une semaine à partir de l’heure d’origine par défaut de `1900-01-01 00:00:00.000`. La valeur `2020-04-13 00:00:00.0000000` correspond à 6 276 semaines à partir de la valeur d’origine de `1900-01-01 00:00:00.000`. 

```sql
declare @date datetime2 = '2020-04-15 21:22:11'
Select DATE_BUCKET(wk, 1, @date)
```

Pour toutes les expressions ci-dessous, la même valeur de sortie de `2020-04-13 00:00:00.0000000` est retournée. Cela est dû au fait que `2020-04-13 00:00:00.0000000` correspond à 6 276 semaines par rapport à la date d’origine et que 6276 est divisible par 2, 3, 4 et 6.

```sql
declare @date datetime2 = '2020-04-15 21:22:11'
Select DATE_BUCKET(wk, 2, @date)
Select DATE_BUCKET(wk, 3, @date)
Select DATE_BUCKET(wk, 4, @date)
Select DATE_BUCKET(wk, 6, @date)
```

La sortie de l’expression ci-dessous est `2020-04-06 00:00:00.0000000`, correspondant à 6 275 semaines à partir de la date d’origine par défaut `1900-01-01 00:00:00.000`.

```sql
declare @date datetime2 = '2020-04-15 21:22:11'
Select DATE_BUCKET(wk, 5, @date)
```

La sortie de l’expression ci-dessous est `2020-06-09 00:00:00.0000000`, correspondant à 75 semaines à partir de la date d’origine spécifiée `2019-01-01 00:00:00`.

```sql
declare @date datetime2 = '2020-06-15 21:22:11'
declare @origin datetime2 = '2019-01-01 00:00:00'
Select DATE_BUCKET(wk, 5, @date, @origin)
```

## <a name="datepart-argument"></a>Argument datepart

**dayofyear** , **day** et **weekday** renvoient la même valeur. Chaque *datepart* et ses abréviations retournent la même valeur.
  
## <a name="number-argument"></a>Argument number

L’argument *number* ne peut pas dépasser la plage des valeurs **int** positives. Dans les instructions suivantes, l’argument pour *number* dépasse la plage des **int** de 1. L'instruction suivante retourne le message d'erreur suivant : `Msg 8115, Level 16, State 2, Line 2. Arithmetic overflow error converting expression to data type int."`
  
```sql
declare @date datetime2 = '2020-04-30 00:00:00'
Select DATE_BUCKET(dd, 2147483648, @date)
```  

Si une valeur number négative est transmise à la fonction `Date_Bucket`, l’erreur suivante est retournée. 

```sql
Msg 9834, Level 16, State 1, Line 1
Invalid bucket width value passed to date_bucket function. Only positive values are allowed.
````

## <a name="date-argument"></a>Argument date  

`DATE_BUCKET` retourne la valeur de base correspondant au type de données de l’argument `date`. Dans l’exemple suivant, une valeur de sortie accompagnée du type de données datetime2 est retournée. 

```sql
Select DATE_BUCKET(dd, 10, SYSUTCDATETIME())
```

## <a name="origin-argument"></a>Argument origin  

Le type de données des arguments `origin` et `date` doit être le même. Si des types de données différents sont utilisés, une erreur est générée.

## <a name="remarks"></a>Notes

Utilisez `DATE_BUCKET` dans les clauses suivantes :

+ GROUP BY
+ HAVING
+ ORDER BY
+ SELECT \<list>
+ WHERE

## <a name="examples"></a>Exemples

### <a name="a-calculating-date_bucket-with-a-bucket-width-of-1-from-the-origin-time"></a>R. Calcul de Date_Bucket avec une largeur de compartiment de 1 à partir de l’heure d’origine

Chacune de ces instructions incrémente *date_bucket* avec une largeur de compartiment de 1 à partir de l’heure d’origine :

```sql
declare @date datetime2 = '2020-04-30 21:21:21'
Select 'Week',  DATE_BUCKET(wk, 1, @date)
Union All
Select 'Day',  DATE_BUCKET(dd, 1, @date)
Union All
Select 'Hour',  DATE_BUCKET(hh, 1, @date)
Union All
Select 'Minutes',  DATE_BUCKET(mi, 1, @date)
Union All
Select 'Seconds',  DATE_BUCKET(ss, 1, @date)
```

Voici le jeu de résultats obtenu.

```sql
Week    2020-04-27 00:00:00.0000000
Day     2020-04-30 00:00:00.0000000
Hour    2020-04-30 21:00:00.0000000
Minutes 2020-04-30 21:21:00.0000000
Seconds 2020-04-30 21:21:21.0000000
```

### <a name="b-using-expressions-as-arguments-for-the-number-and-date-parameters"></a>B. Utilisation d'expressions comme arguments pour les paramètres de date et numériques

Ces exemples utilisent différents types d’expressions comme arguments pour les paramètres *number* et *date*. Ces exemples sont générés à l’aide de la base de données « AdventureWorksDW2017 ».
  
#### <a name="specifying-user-defined-variables-as-number-and-date"></a>Spécification de variables définies par l'utilisateur comme nombre et date  

Cet exemple spécifie des variables définies par l’utilisateur comme arguments pour *number* et *date* :
  
```sql
DECLARE @days int = 365,
        @datetime datetime2 = '2000-01-01 01:01:01.1110000'; /* 2000 was a leap year */;  
SELECT Date_Bucket(day, @days, @datetime);
```

Voici le jeu de résultats obtenu.

```sql
---------------------------
1999-12-08 00:00:00.0000000

(1 row affected)
```  

#### <a name="specifying-a-column-as-date"></a>Spécification d'une colonne comme date

Dans l’exemple ci-dessous, nous calculons les sommes OrderQuantity et UnitPrice regroupées sur les compartiments de dates hebdomadaires.
  
```sql
SELECT
    Date_Bucket(week, 1 ,cast(Shipdate as datetime2)) AS ShippedDateBucket
    ,Sum(OrderQuantity)  As SumOrderQuantity
    ,Sum(UnitPrice) As SumUnitPrice
FROM dbo.FactInternetSales FIS
where Shipdate between '2011-01-03 00:00:00.000' and '2011-02-28 00:00:00.000'
Group by Date_Bucket(week, 1 ,cast(Shipdate as datetime2))
order by 1
```  

Voici le jeu de résultats obtenu.
  
```sql
ShippedDateBucket           SumOrderQuantity SumUnitPrice
--------------------------- ---------------- ---------------------
2011-01-03 00:00:00.0000000 21               65589.7546
2011-01-10 00:00:00.0000000 27               89938.5464
2011-01-17 00:00:00.0000000 31               104404.9064
2011-01-24 00:00:00.0000000 36               118525.6846
2011-01-31 00:00:00.0000000 39               123555.431
2011-02-07 00:00:00.0000000 35               109342.351
2011-02-14 00:00:00.0000000 32               107804.8964
2011-02-21 00:00:00.0000000 37               119456.3428
2011-02-28 00:00:00.0000000 9                28968.6982
```  

#### <a name="specifying-scalar-system-function-as-date"></a>Spécification de la fonction système scalaire comme date

Cet exemple spécifie `SYSDATETIME` pour *date*. La valeur exacte retournée varie selon le jour et l’heure de l’exécution de l’instruction :
  
```sql
SELECT Date_Bucket(wk, 10, SYSDATETIME());  
```  

Voici le jeu de résultats obtenu.

```sql
---------------------------
2020-03-02 00:00:00.0000000

(1 row affected)
```  

#### <a name="specifying-scalar-subqueries-and-scalar-functions-as-number-and-date"></a>Spécification de sous-requêtes scalaires et de fonctions scalaires comme nombre et date

Cet exemple utilise des sous-requêtes scalaires, `MAX(OrderDate)`, comme arguments pour *number* et *date*. `(SELECT top 1 CustomerKey FROM dbo.DimCustomer where GeographyKey > 100)` sert d’argument artificiel pour le paramètre number pour illustrer la sélection d’un argument *number* dans une liste de valeurs.
  
```sql
SELECT DATE_BUCKET(week,(SELECT top 1 CustomerKey FROM dbo.DimCustomer where GeographyKey > 100),  
    (SELECT MAX(OrderDate) FROM dbo.FactInternetSales));  
```  
  
#### <a name="specifying-numeric-expressions-and-scalar-system-functions-as-number-and-date"></a>Spécification d'expressions numériques et de fonctions système scalaires comme nombre et date

Cet exemple utilise une expression numérique [(10/2)] et des fonctions système scalaires (SYSDATETIME) comme arguments pour number et date.
  
```sql
SELECT Date_Bucket(week,(10/2), SYSDATETIME());
```

#### <a name="specifying-an-aggregate-window-function-as-number"></a>Spécification d'une fonction d'agrégation comme nombre

Cet exemple utilise une fonction d’agrégation comme argument pour *number*.
  
```sql
Select 
    DISTINCT DATE_BUCKET(day, 30, Cast([shipdate] as datetime2)) as DateBucket,
    First_Value([SalesOrderNumber]) OVER (Order by DATE_BUCKET(day, 30, Cast([shipdate] as datetime2))) as First_Value_In_Bucket,
    Last_Value([SalesOrderNumber]) OVER (Order by DATE_BUCKET(day, 30, Cast([shipdate] as datetime2))) as Last_Value_In_Bucket
    from [dbo].[FactInternetSales]
Where ShipDate between '2011-01-03 00:00:00.000' and '2011-02-28 00:00:00.000'
order by DateBucket
GO  
``` 
### <a name="c-using-a-non-default-origin-value"></a>C. Utilisation d’une valeur d’origine différente de la valeur par défaut

Cet exemple utilise une valeur d’orgine différente de la valeur par défaut pour générer les compartiments de date. 

```sql
declare @date datetime2 = '2020-06-15 21:22:11'
declare @origin datetime2 = '2019-01-01 00:00:00'
Select DATE_BUCKET(hh, 2, @date, @origin)
```

## <a name="see-also"></a>Voir aussi

[CAST et CONVERT &#40;Transact-SQL&#41;](/sql/t-sql/functions/cast-and-convert-transact-sql/)
