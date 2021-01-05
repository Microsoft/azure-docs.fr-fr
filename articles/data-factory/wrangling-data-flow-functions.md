---
title: Fonctions de transformation pour flux de données de wrangling dans Azure Data Factory
description: Vue d’ensemble des fonctions de flux de données de wrangling disponibles dans Azure Data Factory
author: dcstwh
ms.author: weetok
ms.reviewer: gamal
ms.service: data-factory
ms.topic: conceptual
ms.date: 11/01/2019
ms.openlocfilehash: 28c6228d4401db0fd1301137167a86cb6f18cbeb
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96494917"
---
# <a name="transformation-functions-in-wrangling-data-flow"></a>Fonctions de transformation dans le flux de données de wrangling

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Le flux de données de wrangling dans Azure Data Factory vous permet d’effectuer un rassemblement de données brutes à l’analyse et une préparation agile de données sans code à l’échelle du cloud. Le flux de données de wrangling s’intègre à [Power Query Online](/powerquery-m/power-query-m-reference) et rend les fonctions Power Query M disponibles pour le rassemblement de données brutes à l’analyse via l’exécution Spark. 

> [!NOTE]
> Le flux de wrangling data est actuellement disponible en préversion publique.

Actuellement, toutes les fonctions Power Query M ne sont pas prises en charge pour le rassemblement de données brutes à l’analyse, bien qu’elles soient disponibles pendant la création. Lors de la génération de vos flux de données de wrangling, le message d’erreur suivant s’affiche si une fonction n’est pas prise en charge :

`The Wrangling Data Flow is invalid. Expression.Error: The transformation logic is not supported. Please try a simpler expression.`

Voici une liste des fonctions Power Query M prises en charge.

## <a name="column-management"></a>Gestion des colonnes

* Sélection : [Table.SelectColumns](/powerquery-m/table-selectcolumns)
* Suppression : [Table.RemoveColumns](/powerquery-m/table-removecolumns)
* Renommage : [Table.RenameColumns](/powerquery-m/table-renamecolumns), [Table.PrefixColumns](/powerquery-m/table-prefixcolumns), [Table.TransformColumnNames](/powerquery-m/table-transformcolumnnames)
* Réorganisation : [Table.ReorderColumns](/powerquery-m/table-reordercolumns)

## <a name="row-filtering"></a>Filtrage de lignes

Utilisez la fonction M [Table.SelectRows](/powerquery-m/table-selectrows) pour filtrer avec les conditions suivantes :

* Égalité et inégalité
* Comparaisons numériques, de texte et de date (mais pas date/heure)
* Informations numériques, telles que [Number.IsEven](/powerquery-m/number-iseven)/[Odd](/powerquery-m/number-iseven)
* Autonomie du texte avec [Text.Contains](/powerquery-m/text-contains), [Text.StartsWith](/powerquery-m/text-startswith) ou [Text.EndsWith](/powerquery-m/text-endswith)
* Plages de dates, y compris toutes les [fonctions Date](/powerquery-m/date-functions) « IsIn » 
* Combinaisons de celles-ci au moyen des conditions and, or ou not

## <a name="adding-and-transforming-columns"></a>Ajout et transformation de colonnes

Les fonctions M suivantes ajoutent ou transforment les colonnes : [Table.AddColumn](/powerquery-m/table-addcolumn), [Table.TransformColumns](/powerquery-m/table-transformcolumns), [Table.ReplaceValue](/powerquery-m/table-replacevalue), [Table.DuplicateColumn](/powerquery-m/table-duplicatecolumn). Voici ci-après les fonctions de transformation prises en charge.

* Arithmétique numérique
* Concaténation de texte
* Arithmétique Date et Heure (Opérateurs arithmétiques, [Date.AddDays](/powerquery-m/date-adddays), [Date.AddMonths](/powerquery-m/date-addmonths), [Date.AddQuarters](/powerquery-m/date-addquarters), [Date.AddWeeks](/powerquery-m/date-addweeks), [Date.AddYears](/powerquery-m/date-addyears))
* Les durées peuvent être utilisées pour les opérations arithmétiques de date et d’heure, mais doivent être transformées en un autre type avant d’être écrites dans un récepteur (opérateurs arithmétiques, [#duration](/powerquery-m/sharpduration), [Duration.Days](/powerquery-m/duration-days), [Duration.Hours](/powerquery-m/duration-hours), [Duration.Minutes](/powerquery-m/duration-minutes), [Duration.Seconds](/powerquery-m/duration-seconds), [Duration.TotalDays](/powerquery-m/duration-totaldays), [Duration.TotalHours](/powerquery-m/duration-totalhours), [Duration.TotalMinutes](/powerquery-m/duration-totalminutes), [Duration.TotalSeconds](/powerquery-m/duration-totalseconds))    
* La plupart des fonctions numériques standard, scientifiques et trigonométriques (toutes les fonctions sous [Opérations](/powerquery-m/number-functions#operations), [Arrondi](/powerquery-m/number-functions#rounding) et [Trigonométrie](/powerquery-m/number-functions#trigonometry), *sauf* Number.Factorial, Number.Permutations et Number.Combinations)
* Remplacement ([Replacer.ReplaceText](/powerquery-m/replacer-replacetext), [Replacer.ReplaceValue](/powerquery-m/replacer-replacevalue), [Text.Replace](/powerquery-m/text-replace), [Text.Remove](/powerquery-m/text-remove))
* Extraction de texte positionnel ([Text.PositionOf](/powerquery-m/text-positionof), [Text.Length](/powerquery-m/text-length), [Text.Start](/powerquery-m/text-start), [Text.End](/powerquery-m/text-end), [Text.Middle](/powerquery-m/text-middle), [Text.ReplaceRange](/powerquery-m/text-replacerange), [Text.RemoveRange](/powerquery-m/text-removerange))
* Mise en forme du texte de base ([Text.Lower](/powerquery-m/text-lower), [Text.Upper](/powerquery-m/text-upper), [Text.Trim](/powerquery-m/text-trim)/[Start](/powerquery-m/text-trimstart)/[End](/powerquery-m/text-trimend), [Text.PadStart](/powerquery-m/text-padstart)/[End](/powerquery-m/text-padend), [Text.Reverse](/powerquery-m/text-reverse))
* Fonctions de date et heure ([Date.Day](/powerquery-m/date-day), [Date.Month](/powerquery-m/date-month), [Date.Year](/powerquery-m/date-year), [Time.Hour](/powerquery-m/time-hour), [Time.Minute](/powerquery-m/time-minute), [Time.Second](/powerquery-m/time-second), [Date.DayOfWeek](/powerquery-m/date-dayofweek), [Date.DayOfYear](/powerquery-m/date-dayofyear), [Date.DaysInMonth](/powerquery-m/date-daysinmonth))
* Expressions if (mais les branches doivent avoir des types correspondants)
* Filtres de lignes en tant que colonne logique
* Constantes de nombre, de texte, de logique, de date et de date/heure

<a name="mergingjoining-tables"></a>Fusion/jointure de tables
----------------------
* Power Query génère une jointure imbriquée (table.NestedJoin ; les utilisateurs peuvent également écrire manuellement [Table.AddJoinColumn](/powerquery-m/table-addjoincolumn)).
    Les utilisateurs doivent alors développer la colonne de jointure imbriquée dans une jointure non imbriquée (aucune prise en charge de Table.ExpandTableColumn dans un autre contexte).
* La fonction M [Table.Join](/powerquery-m/table-join) peut être écrite directement afin d’éviter la nécessité d’une étape d’expansion supplémentaire, mais l’utilisateur doit s’assurer qu’il n’existe aucun nom de colonne en double au sein des tables jointes
* Types de jointures pris en charge :   [Inner](/powerquery-m/joinkind-inner), [LeftOuter](/powerquery-m/joinkind-leftouter), [RightOuter](/powerquery-m/joinkind-rightouter), [FullOuter](/powerquery-m/joinkind-fullouter)
* Prise en charge de [Value.Equals](/powerquery-m/value-equals) et de [Value.NullableEquals](/powerquery-m/value-nullableequals) en tant que comparateurs d’égalité clés

## <a name="group-by"></a>Regrouper par

Utilisez [Table.Group](/powerquery-m/table-group) pour agréger des valeurs.
* Doit être utilisé avec une fonction d’agrégation
* Fonctions d’agrégation prises en charge :   [List.Sum](/powerquery-m/list-sum), [List.Count](/powerquery-m/list-count), [List.Average](/powerquery-m/list-average), [List.Min](/powerquery-m/list-min), [List.Max](/powerquery-m/list-max), [List.StandardDeviation](/powerquery-m/list-standarddeviation), [List.First](/powerquery-m/list-first) et [List.Last](/powerquery-m/list-last)

## <a name="sorting"></a>Tri

Utilisez [Table.Sort](/powerquery-m/table-sort) pour trier les valeurs.

## <a name="reducing-rows"></a>Réduction de lignes

Conserver et supprimer les premiers éléments, Conserver la plage (fonctions M correspondantes, qui prennent uniquement en charge les nombres, pas les conditions : [Table.FirstN](/powerquery-m/table-firstn), [Table.Skip](/powerquery-m/table-skip), [Table.RemoveFirstN](/powerquery-m/table-removefirstn), [Table.Range](/powerquery-m/table-range), [Table.MinN](/powerquery-m/table-minn), [Table.MaxN](/powerquery-m/table-maxn))

## <a name="known-unsupported-functions"></a>Fonctions connues non prises en charge

| Fonction | Statut |
| -- | -- |
| Table.PromoteHeaders | Non pris en charge. Le même résultat peut être obtenu en définissant « Première ligne comme en-tête » dans le jeu de données. |
| Table.CombineColumns | Il s’agit d’un scénario courant qui n’est pas directement pris en charge mais qui peut être obtenu en ajoutant une nouvelle colonne qui concatène deux colonnes données.  Par exemple, Table.AddColumn(RemoveEmailColumn, "Name", each [FirstName] & " " & [LastName]) |
| Table.TransformColumnTypes | Ce scénario est généralement pris en charge. Les scénarios suivants ne sont pas pris en charge : transformation de chaîne en type Devise, transformation de chaîne en type Heure, transformation de chaîne en type Pourcentage. |
| Table.NestedJoin | Effectuer une simple jointure entraînera une erreur de validation. Les colonnes doivent être développées pour que l’opération fonctionne. |
| Table.Distinct | La suppression des lignes en double n’est pas prise en charge. |
| Table.RemoveLastN | La suppression des lignes du bas n’est pas prise en charge. |
| Table.RowCount | Non pris en charge, mais peut être obtenu en ajoutant une colonne personnalisée contenant la valeur 1, puis en agrégeant cette colonne avec List.Sum. Table.Group est pris en charge. | 
| Gestion des erreurs au niveau des lignes | La gestion des erreurs au niveau des lignes n’est pas prise en charge actuellement. Par exemple, pour exclure les valeurs non numériques d’une colonne, une méthode consiste à transformer la colonne de texte en nombre. Les cellules qui ne peuvent pas être transformées sont dans un état d’erreur et doivent être filtrées. Ce scénario n’est pas possible dans les flux de wrangling data. |
| Table.Transpose | Non pris en charge |
| Table.Pivot | Non pris en charge |

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [créer un flux de données de wrangling](wrangling-data-flow-tutorial.md).
