---
title: Informations de référence sur l’accélération des requêtes en langage SQL
titleSuffix: Azure Storage
description: Découvrez comment utiliser la syntaxe SQL d’accélération des requêtes.
services: storage
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 09/09/2020
ms.author: normesta
ms.subservice: data-lake-storage-gen2
ms.reviewer: ereilebr
ms.openlocfilehash: 2eda67e377a3b61e696e732b916d788c00a18eae
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95908776"
---
# <a name="query-acceleration-sql-language-reference"></a>Informations de référence sur l’accélération des requêtes en langage SQL

L’accélération des requêtes prend en charge un langage de type SQL ANSI pour exprimer des requêtes sur le contenu des blobs.  Le dialecte SQL d’accélération des requêtes est un sous-ensemble de SQL ANSI, avec un ensemble limité de types de données, d’opérateurs, etc. pris en charge, mais il s’étend également au SQL ANSI pour prendre en charge les requêtes sur des formats de données hiérarchiques semi-structurés tels que JSON. 

## <a name="select-syntax"></a>Syntaxe SELECT

La seule instruction SQL prise en charge par l’accélération des requêtes est l’instruction SELECT. Cet exemple retourne chaque ligne pour laquelle l’expression retourne la valeur true.

```sql
SELECT * FROM table [WHERE expression] [LIMIT limit]
```

Pour les données au format CSV, *table* doit être `BlobStorage`.  Cela signifie que la requête sera exécutée sur le blob spécifié dans l’appel REST.
Pour les données au format JSON, *table* est un « descripteur de table ».   Voir la section [Descripteurs de table](#table-descriptors) de cet article.

Dans l’exemple suivant, pour chaque ligne pour laquelle l’*expression* WHERE retourne la valeur true, cette instruction renverra une nouvelle ligne créée à partir de l’évaluation de chacune des expressions de projection.


```sql
SELECT expression [, expression …] FROM table [WHERE expression] [LIMIT limit]
```

L’exemple suivant retourne un calcul d’agrégation (par exemple, la valeur moyenne d’une colonne particulière) sur chacune des lignes pour lesquelles l’*expression* retourne la valeur true. 

```sql
SELECT aggregate_expression FROM table [WHERE expression] [LIMIT limit]
```

L’exemple suivant retourne des décalages appropriés pour le fractionnement d’un blob au format CSV.  Consultez la section [Sys.Split](#sys-split) de cet article.

```sql
SELECT sys.split(split_size)FROM BlobStorage
```

<a id="data-types"></a>

## <a name="data-types"></a>Types de données

|Type de données|Description|
|---------|-------------------------------------------|
|INT      |Entier signé 64 bits.                     |
|FLOAT    |Virgule flottante (« double précision ») 64 bits.|
|STRING   |Chaîne Unicode de longueur variable.            |
|timestamp|Limite dans le temps.                           |
|BOOLEAN  |True ou false.                             |

Lors de la lecture de valeurs à partir de données au format CSV, toutes les valeurs sont lues en tant que chaînes.  Les valeurs de chaîne peuvent être converties en d’autres types à l’aide d’expressions CAST.  Les valeurs peuvent être castées implicitement en d’autres types en fonction du contexte. Pour plus d’informations, consultez [Priorité des types de données (Transact-SQL)](/sql/t-sql/data-types/data-type-precedence-transact-sql).

## <a name="expressions"></a>Expressions

### <a name="referencing-fields"></a>Référencement des champs

Pour les données au format JSON ou les données au format CSV avec une ligne d’en-tête, les champs peuvent être référencés par nom.  Les noms de champs peuvent être entre guillemets ou sans guillemets. Les noms de champs entre guillemets sont placés entre guillemets doubles ("), peuvent contenir des espaces et respectent la casse.  Les noms de champs sans guillemets ne respectent pas la casse ni ne doivent contenir de caractères spéciaux.

Dans les données au format CSV, les champs peuvent également être référencés par un ordinal, précédé d’un trait de soulignement (_).  Par exemple, le premier champ peut être référencé par _1, ou le onzième champ par _11.  Le référencement des champs par ordinal est utile pour les données au format CSV qui ne contiennent pas de ligne d’en-tête, auquel cas le seul moyen de référencer un champ particulier est par un nombre ordinal.

### <a name="operators"></a>Opérateurs

Les opérateurs SQL standard suivants sont pris en charge :

|Opérateur|Description|
|--|--|
|[=](/sql/t-sql/language-elements/equals-transact-sql)    |Compare l'égalité de deux expressions (opérateur de comparaison).|
|[!=](/sql/t-sql/language-elements/not-equal-to-transact-sql-exclamation)    |Teste si une expression est différente d'une autre expression (opérateur de comparaison).|
|[<>](/sql/t-sql/language-elements/not-equal-to-transact-sql-traditional)    |Compare deux expressions avec l’opérateur de comparaison différent de.|
|[<](/sql/t-sql/language-elements/less-than-transact-sql)    |Compare deux expressions avec l’opérateur de comparaison inférieur à.|
|[<=](/sql/t-sql/language-elements/less-than-or-equal-to-transact-sql)    |Compare deux expressions avec l’opérateur de comparaison inférieur ou égal à.|
|[>](/sql/t-sql/language-elements/greater-than-transact-sql)    |Compare deux expressions avec l’opérateur de comparaison supérieur à. |
|[>=](/sql/t-sql/language-elements/greater-than-or-equal-to-transact-sql)    |Compare deux expressions avec l'opérateur de comparaison supérieur ou égal à.|
|[+](/sql/t-sql/language-elements/add-transact-sql)    |Additionne deux nombres. Cet opérateur arithmétique d'addition peut aussi ajouter un nombre de jours à une date.|
|[-](/sql/t-sql/language-elements/subtract-transact-sql)    |Effectue une soustraction entre deux nombres (opérateur de soustraction arithmétique). |
|[/](/sql/t-sql/language-elements/divide-transact-sql)    |Divise un nombre par un autre (opérateur arithmétique de division).|
|[*](/sql/t-sql/language-elements/multiply-transact-sql)    |Multiplie deux expressions (opérateur arithmétique de multiplication).|
|[%](/sql/t-sql/language-elements/modulo-transact-sql)    |Renvoie le reste d'un nombre divisé par un autre.|
|[AND](/sql/t-sql/language-elements/bitwise-and-transact-sql)    |Effectue une opération AND logique au niveau du bit avec deux valeurs entières.|
|[OR](/sql/t-sql/language-elements/bitwise-or-transact-sql)    |Exécute une opération logique OR au niveau du bit entre deux valeurs entières spécifiées, traduites en expressions binaires dans les instructions Transact-SQL.|
|[NOT](/sql/t-sql/language-elements/not-transact-sql)    |Inverse une entrée booléenne.|
|[CAST](/sql/t-sql/functions/cast-and-convert-transact-sql)    |Convertit une expression d'un type de données à un autre.|
|[BETWEEN](/sql/t-sql/language-elements/between-transact-sql)    |Définit un intervalle sur lequel la recherche doit porter.|
|[IN](/sql/t-sql/language-elements/in-transact-sql)    |Détermine si une valeur donnée correspond à la valeur d'une liste ou d'une sous-requête.|
|[NULLIF](/sql/t-sql/language-elements/nullif-transact-sql)    |Retourne une valeur NULL si les deux expressions spécifiées sont égales.|
|[COALESCE](/sql/t-sql/language-elements/coalesce-transact-sql)    |Évalue les arguments dans l’ordre et retourne la valeur actuelle de la première expression qui ne prend pas initialement la valeur NULL.|

Si les types de données à gauche et à droite d’un opérateur sont différents, la conversion automatique sera effectuée en fonction des règles spécifiées ici : [Priorité des types de données (Transact-SQL)](/sql/t-sql/data-types/data-type-precedence-transact-sql).

Le langage SQL d’accélération des requêtes prend en charge uniquement un très petit sous-ensemble des types de données décrits dans cet article.  Consultez la section [Types de données](#data-types) de cet article.

### <a name="casts"></a>Casts

Le langage SQL d’accélération des requêtes prend en charge l’opérateur CAST, conformément aux règles ci-dessous : [Conversion de types de données (Moteur de base de données)](/sql/t-sql/data-types/data-type-conversion-database-engine).  

Le langage SQL d’accélération des requêtes prend en charge uniquement un minuscule sous-ensemble des types de données décrits dans cet article.  Consultez la section [Types de données](#data-types) de cet article.

### <a name="string-functions"></a>Fonctions de chaînes

Le langage SQL d’accélération des requêtes prend en charge les fonctions de chaîne SQL standard suivantes :

|Fonction|Description|
|--|--|
|CHAR_LENGTH    | Retourne la longueur en caractères de l’expression de chaîne, si l’expression de chaîne est de type de données character ; sinon, retourne la longueur en octets de l’expression de chaîne (le plus petit entier qui n’est pas inférieur au nombre de bits divisé par 8). (Cette fonction est la même que la fonction CHARACTER_LENGTH.)|
|CHARACTER_LENGTH    |Retourne la longueur en caractères de l’expression de chaîne, si l’expression de chaîne est de type de données character ; sinon, retourne la longueur en octets de l’expression de chaîne (le plus petit entier qui n’est pas inférieur au nombre de bits divisé par 8). (Cette fonction est la même que la fonction CHAR_LENGTH.)|
|[LOWER](/sql/t-sql/functions/lower-transact-sql)    |Retourne une chaîne de caractères après avoir transformé les caractères majuscules en caractères minuscules.|
|[UPPER](/sql/t-sql/functions/upper-transact-sql)    |Transforme les caractères d'une expression en minuscules en caractères majuscules.|
|[SUBSTRING](/sql/t-sql/functions/substring-transact-sql)    |Retourne une partie d’une expression de type caractère, binaire, texte ou image dans SQL Server.|
|[TRIM](/sql/t-sql/functions/trim-transact-sql)    |Supprime le caractère espace char(32) ou d’autres caractères spécifiés au début et à la fin d’une chaîne.|
|LEADING    |Description|
|TRAILING    |Description|

Voici quelques exemples :

|Fonction| Exemple|Résultats|
|---|---|---|
|CHARACTER_LENGTH|``SELECT CHARACTER_LENGTH('abcdefg') from BlobStorage`` |``7``|
|CHAR_LENGTH|``SELECT CHAR_LENGTH(_1) from BlobStorage``|``1``|
|LOWER|``SELECT LOWER('AbCdEfG') from BlobStorage``|``abcdefg``|
|UPPER|``SELECT UPPER('AbCdEfG') from BlobStorage``|``ABCDEFG``|
|SUBSTRING|``SUBSTRING('123456789', 1, 5)``|``23456``|
|TRIM|``TRIM(BOTH '123' FROM '1112211Microsoft22211122')``|``Microsoft``|

### <a name="date-functions"></a>Fonctions de date

Les fonctions de date SQL standard suivantes sont prises en charge :

``DATE_ADD``, ``DATE_DIFF``, ``EXTRACT``, ``TO_STRING``, ``TO_TIMESTAMP``.

Nous convertissons actuellement tous les [formats de date de la norme ISO 8601](https://www.w3.org/TR/NOTE-datetime). 

#### <a name="date_add-function"></a>Fonction DATE_ADD

Le langage SQL d’accélération des requêtes prend en charge l’année, le mois, le jour, l’heure, la minute et la seconde pour la fonction ``DATE_ADD``.

Exemples :

``sql DATE_ADD(datepart, quantity, timestamp) DATE_ADD('minute', 1, CAST('2017-01-02T03:04:05.006Z' AS TIMESTAMP)
```

#### DATE_DIFF function

The query acceleration SQL language supports year, month, day, hour, minute, second for the ``DATE_DIFF`` function.

```sql
DATE_DIFF(datepart, timestamp, timestamp)
DATE_DIFF('hour','2018-11-09T00:00+05:30','2018-11-09T01:00:23-08:00') 
```

#### <a name="extract-function"></a>Fonction EXTRACT

Pour l’extraction autre que l’élément de date pris en charge pour la fonction ``DATE_ADD``, le langage SQL d’accélération des requêtes prend en charge timezone_hour et timezone_minute en tant qu’élément de date.

Exemples :

```sql
EXTRACT(datepart FROM timestampstring)
EXTRACT(YEAR FROM '2010-01-01T')
```

#### <a name="to_string-function"></a>Fonction TO_STRING

Exemples :

```sql
TO_STRING(TimeStamp , format)
TO_STRING(CAST('1969-07-20T20:18Z' AS TIMESTAMP),  'MMMM d, y')
```

Ce tableau décrit les chaînes que vous pouvez utiliser pour spécifier le format de sortie de la fonction ``TO_STRING``.

|Chaîne de format    |Output                               |
|-----------------|-------------------------------------|
|yy               |Année au format à 2 chiffres : « 99 » pour 1999|
|y                |Année au format à 4 chiffres               |
|aaaa             |Année au format à 4 chiffres               |
|M                |Mois de l’année : 1                    |
|MM               |Mois complété par un zéro : 01               |
|MMM              |Abbr. Mois de l’année : JAN            |
|MMMM             |Nom de mois complet : mai                      |
|d                |Jour du mois (1 à 31)                  |
|jj               |Jour du mois complété par un zéro (01 à 31)     |
|a                |AM ou PM                             |
|h                |Heure de la journée (1 à 12)                   |
|hh               |Heure de la journée complétée par un zéro (01 à 12)     |
|H                |Heure de la journée (0 à 23)                   |
|HH               |Heure de la journée complétée par un zéro (00 à 23)      |
|m                |Minute de l’heure (0 à 59)                |
|MM               |Minute complétée par un zéro (00 à 59)           |
|s                |Secondes des minutes (0 à 59)             |
|ss               |Secondes complétées par un zéro (00 à 59)          |
|S                |Fraction de secondes (0,1 à 0,9)        |
|SS               |Fraction de secondes (0,01 à 0,99)      |
|SSS              |Fraction de secondes (0,001 à 0,999)    |
|X                |Décalage en heures                      |
|XX ou XXXX       |Décalage en heures et minutes (+0430)  |
|XXX ou XXXXX     |Décalage en heures et minutes (-07:00) |
|x                |Décalage en heures (7)                  |
|xx ou xxxx       |Décalage en heures et minutes (+0530)    |
|Xxx ou xxxxx     |Décalage en heures et minutes (+05:30)   |

#### <a name="to_timestamp-function"></a>Fonction TO_TIMESTAMP

Seuls les formats ISO 8601 sont pris en charge.

Exemples :

```sql
TO_TIMESTAMP(string)
TO_TIMESTAMP('2007T')
```

> [!NOTE]
> Vous pouvez également utiliser la fonction ``UTCNOW`` pour récupérer l’heure système.


## <a name="aggregate-expressions"></a>Expressions d’agrégation

Une instruction SELECT peut contenir une ou plusieurs expressions de projection ou une expression d’agrégation unique.  Les expressions d’agrégation suivantes sont prises en charge :

|Expression|Description|
|--|--|
|[COUNT(\*)](https://docs.microsoft.com/sql/t-sql/functions/count-transact-sql)    |Retourne le nombre d’enregistrements qui correspondent à l’expression de prédicat.|
|[COUNT(expression)](https://docs.microsoft.com/sql/t-sql/functions/count-transact-sql)    |Retourne le nombre d’enregistrements pour lesquels l’expression n’a pas une valeur nulle.|
|[AVERAGE(expression)](https://docs.microsoft.com/sql/t-sql/functions/avg-transact-sql)    |Retourne la moyenne de toutes les valeurs non nulles de l’expression.|
|[MIN(expression)](https://docs.microsoft.com/sql/t-sql/functions/min-transact-sql)    |Retourne la valeur non nulle minimale de l’expression.|
|[MAX(expression](https://docs.microsoft.com/sql/t-sql/functions/max-transact-sql)    |Retourne la valeur non nulle maximale de l’expression.|
|[SUM(expression)](https://docs.microsoft.com/sql/t-sql/functions/sum-transact-sql)    |Retourne la somme de toutes les valeurs non nulles d’une expression.|

### <a name="missing"></a>MISSING

L’opérateur ``IS MISSING`` est le seul opérateur non standard pris en charge par le langage SQL d’accélération des requêtes.  Pour les données JSON, si un champ est manquant dans un enregistrement d’entrée particulier, le champ d’expression ``IS MISSING`` prend la valeur booléenne true.

<a id="table-descriptors"></a>

## <a name="table-descriptors"></a>Descripteurs de table

Pour les données CSV, le nom de la table est toujours `BlobStorage`.  Par exemple :

```sql
SELECT * FROM BlobStorage
```

Pour les données JSON, des options supplémentaires sont disponibles :

```sql
SELECT * FROM BlobStorage[*].path
```

Cela permet d’interroger des sous-ensembles de données JSON.

Pour les requêtes JSON, vous pouvez indiquer le chemin d’accès dans le cadre de la clause FROM. Ces chemins d’accès permettent d’analyser le sous-ensemble de données JSON. Ces chemins d’accès peuvent faire référence à des valeurs Objet et Tableau JSON.

Prenons un exemple pour comprendre cela plus en détail.

Voici notre exemple de données :

```json
{
  "id": 1,
  "name": "mouse",
  "price": 12.5,
  "tags": [
    "wireless",
    "accessory"
  ],
  "dimensions": {
    "length": 3,
    "width": 2,
    "height": 2
  },
  "weight": 0.2,
  "warehouses": [
    {
      "latitude": 41.8,
      "longitude": -87.6
    }
  ]
}
```

Parmi les données ci-dessus, vous pouvez être intéressé uniquement par l’objet JSON `warehouses`. L’objet `warehouses` est un type de tableau JSON. Vous pouvez donc l’indiquer dans la clause FROM. Votre exemple de requête peut se présenter comme suit.

```sql
SELECT latitude FROM BlobStorage[*].warehouses[*]
```

La requête récupère tous les champs, mais sélectionne uniquement la latitude.

Si vous souhaitez accéder uniquement à la valeur de l’objet JSON `dimensions`, vous pouvez utiliser la référence à cet objet dans votre requête. Par exemple :

```sql
SELECT length FROM BlobStorage[*].dimensions
```

Cela limite également votre accès aux membres de l’objet `dimensions`. Si vous souhaitez accéder à d’autres membres de champs JSON et aux valeurs internes d’objets JSON, vous pouvez utiliser une requête telle que celle illustrée dans l’exemple suivant :

```sql
SELECT weight,warehouses[0].longitude,id,tags[1] FROM BlobStorage[*]
```

> [!NOTE]
> BlobStorage et BlobStorage[\*] font tous deux référence à l’objet entier. Toutefois, si vous avez un chemin d’accès dans la clause FROM, vous devez utiliser BlobStorage[\*].path.

<a id="sys-split"></a>

## <a name="syssplit"></a>Sys.Split

Il s’agit d’une forme spéciale de l’instruction SELECT, qui est uniquement disponible pour les données au format CSV.

```sql
SELECT sys.split(split_size)FROM BlobStorage
```

Utilisez cette instruction dans les cas où vous souhaitez télécharger et traiter les enregistrements de données CSV par lots. De cette façon, vous pouvez traiter les enregistrements en parallèle au lieu de devoir télécharger tous les enregistrements en même temps. Cette instruction ne retourne pas d’enregistrements du fichier CSV. Au lieu de cela, elle retourne une collection de tailles de lot. Vous pouvez ensuite utiliser chaque taille de lot pour récupérer un lot d’enregistrements de données. 

Utilisez le paramètre *split_size* pour spécifier le nombre d’octets que doit contenir chaque lot. Par exemple, si vous souhaitez traiter uniquement 10 Mo de données à la fois, l’instruction ressemble à ceci : `SELECT sys.split(10485760)FROM BlobStorage`, car 10 Mo est égal à 10 485 760 octets. Chaque lot contiendra autant d’enregistrements que possible dans ces 10 Mo. 

Dans la plupart des cas, la taille de chaque lot sera légèrement supérieure au nombre que vous spécifiez. Cela est dû au fait qu’un lot ne peut pas contenir un enregistrement partiel. Si le dernier enregistrement d’un lot commence avant que votre seuil soit atteint, le lot est plus volumineux afin de pouvoir contenir l’enregistrement complet. La taille du dernier lot sera probablement inférieure à la taille que vous spécifiez.

>[!NOTE]
> Le valeur de split_size doit être au minimum de 10 Mo (10485760).

## <a name="see-also"></a>Voir aussi

- [Accélération des requêtes Azure Data Lake Storage](data-lake-storage-query-acceleration.md)
- [Filtrer des données à l’aide de l’accélération des requêtes d’Azure Data Lake Storage](data-lake-storage-query-acceleration-how-to.md)