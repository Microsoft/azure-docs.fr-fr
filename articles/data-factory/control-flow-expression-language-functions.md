---
title: Expressions et fonctions dans Azure Data Factory
description: Cet article fournit des informations sur les expressions et fonctions que vous pouvez utiliser pour la création d’entités de fabrique de données.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 11/25/2019
ms.openlocfilehash: 3c966f0efc51a3b2fa8908e060b4031ae1ad1e50
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500017"
---
# <a name="expressions-and-functions-in-azure-data-factory"></a>Expressions et fonctions dans Azure Data Factory

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-functions-variables.md)
> * [Version actuelle](control-flow-expression-language-functions.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article fournit des détails sur les expressions et fonctions prises en charge par Azure Data Factory. 

## <a name="expressions"></a>Expressions

Les valeurs JSON indiquées dans la définition peuvent être littérales. Il peut également s’agir d’expressions évaluées lors de l’exécution du runtime. Par exemple :  
  
```json
"name": "value"
```

 or  
  
```json
"name": "@pipeline().parameters.password"
```

Les expressions peuvent apparaître n’importe où dans une valeur de chaîne JSON, et entraînent toujours une autre valeur JSON. Si une valeur JSON est une expression, le corps de celle-ci est extrait en supprimant l’arobase (\@). Si une chaîne littérale devant commencer par \@ est nécessaire, elle doit être placée dans une séquence d’échappement en utilisant \@\@. Les exemples suivants montrent comment les expressions sont évaluées.  
  
|Valeur JSON|Résultats|  
|----------------|------------|  
|« parameters »|Les caractères « parameters » sont retournés.|  
|« parameters[1] »|Les caractères « parameters[1] » sont retournés.|  
|"\@\@"|Une chaîne de 1 caractère contenant « \@ » est retournée.|  
|" \@"|Une chaîne de 2 caractères contenant « \@ » est retournée.|  
  
 Des expressions peuvent également apparaître dans des chaînes, qui utilisent une fonctionnalité appelée *interpolation de chaîne* où les expressions sont encapsulées dans `@{ ... }`. Par exemple : `"name" : "First Name: @{pipeline().parameters.firstName} Last Name: @{pipeline().parameters.lastName}"`  
  
 En utilisant une interpolation de chaîne, le résultat est toujours une chaîne. Supposons que j’ai défini `myNumber` comme `42` et `myString` comme `foo` :  
  
|Valeur JSON|Résultats|  
|----------------|------------|  
|"\@pipeline().parameters.myString"| Retourne `foo` en tant que chaîne.|  
|"\@{pipeline().parameters.myString}"| Retourne `foo` en tant que chaîne.|  
|"\@pipeline().parameters.myNumber"| Retourne `42` en tant que *nombre*.|  
|"\@{pipeline().parameters.myNumber}"| Retourne `42` en tant que *chaîne*.|  
|"Answer is: @{pipeline().parameters.myNumber}"| Retourne la chaîne `Answer is: 42`.|  
|"\@concat(’Answer is: ’, string(pipeline().parameters.myNumber))"| Retourne la chaîne `Answer is: 42`.|  
|"Answer is: \@\@{pipeline().parameters.myNumber}"| Retourne la chaîne `Answer is: @{pipeline().parameters.myNumber}`.|  
  
## <a name="examples"></a>Exemples

### <a name="complex-expression-example"></a>Exemple d’expression complexe
L’exemple ci-dessous montre un exemple complexe qui fait référence à un sous-champ profond d’une sortie d’activité. Pour faire référence à un paramètre de pipeline qui prend la valeur d’un sous-champ, utilisez la syntaxe [] au lieu de l’opérateur point(.) (comme dans le cas de subfield1 et subfield2)

@activity('*activityName*').output.*subfield1*.*subfield2*[pipeline().parameters.*subfield3*].*subfield4*

### <a name="a-dataset-with-a-parameter"></a>Un jeu de données avec un paramètre
Dans l’exemple suivant, BlobDataset prend un paramètre nommé **path**. Sa valeur est utilisée pour donner une valeur à la propriété **folderPath** à l’aide de l’expression : `dataset().path`. 

```json
{
    "name": "BlobDataset",
    "properties": {
        "type": "AzureBlob",
        "typeProperties": {
            "folderPath": "@dataset().path"
        },
        "linkedServiceName": {
            "referenceName": "AzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "parameters": {
            "path": {
                "type": "String"
            }
        }
    }
}
```

### <a name="a-pipeline-with-a-parameter"></a>Un pipeline avec un paramètre
Dans l’exemple suivant, le pipeline prend les paramètres **inputPath** et **outputPath**. Le **path** du jeu de données blob paramétrable est défini par les valeurs de ces paramètres. Voici la syntaxe utilisée : `pipeline().parameters.parametername`. 

```json
{
    "name": "Adfv2QuickStartPipeline",
    "properties": {
        "activities": [
            {
                "name": "CopyFromBlobToBlob",
                "type": "Copy",
                "inputs": [
                    {
                        "referenceName": "BlobDataset",
                        "parameters": {
                            "path": "@pipeline().parameters.inputPath"
                        },
                        "type": "DatasetReference"
                    }
                ],
                "outputs": [
                    {
                        "referenceName": "BlobDataset",
                        "parameters": {
                            "path": "@pipeline().parameters.outputPath"
                        },
                        "type": "DatasetReference"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                }
            }
        ],
        "parameters": {
            "inputPath": {
                "type": "String"
            },
            "outputPath": {
                "type": "String"
            }
        }
    }
}
```
### <a name="tutorial"></a>Didacticiel
Cela [tutoriel](https://azure.microsoft.com/mediahandler/files/resourcefiles/azure-data-factory-passing-parameters/Azure%20data%20Factory-Whitepaper-PassingParameters.pdf) vous montre comment transmettre des paramètres entre un pipeline et une activité et entre les activités.

  
## <a name="functions"></a>Fonctions

Vous pouvez appeler des fonctions dans des expressions. Les sections suivantes fournissent des informations sur les fonctions qui peut être utilisées dans une expression.  

## <a name="string-functions"></a>Fonctions de chaîne  

Pour travailler avec des chaînes, vous pouvez utiliser ces fonctions de chaîne, ainsi que certaines [fonctions de collection](#collection-functions).
Les fonctions de chaîne sont uniquement utilisables sur des chaînes.

| Fonction de chaîne | Tâche |
| --------------- | ---- |
| [concat](control-flow-expression-language-functions.md#concat) | Combine au moins deux chaînes et retourne la chaîne combinée. |
| [endsWith](control-flow-expression-language-functions.md#endswith) | Vérifier si une chaîne se termine par la sous-chaîne spécifiée. |
| [guid](control-flow-expression-language-functions.md#guid) | Générer un identificateur global unique (GUID) sous forme de chaîne. |
| [indexOf](control-flow-expression-language-functions.md#indexof) | Renvoyer la position de départ d’une sous-chaîne. |
| [lastIndexOf](control-flow-expression-language-functions.md#lastindexof) | Retourne la position de départ de la dernière occurrence d’une sous-chaîne. |
| [replace](control-flow-expression-language-functions.md#replace) | Remplacer une sous-chaîne par la chaîne spécifiée et renvoyer la chaîne mise à jour. |
| [split](control-flow-expression-language-functions.md#split) | Retourne un tableau qui contient des sous-chaînes, séparées par des virgules, extraites à partir d’une chaîne plus grande sur la base d’un caractère délimiteur spécifié dans la chaîne d’origine. |
| [startsWith](control-flow-expression-language-functions.md#startswith) | Vérifie si une chaîne commence par une sous-chaîne spécifique. |
| [substring](control-flow-expression-language-functions.md#substring) | Renvoyer les caractères d’une chaîne, en commençant à partir de la position spécifiée. |
| [toLower](control-flow-expression-language-functions.md#toLower) | Retourne une chaîne en minuscules. |
| [toUpper](control-flow-expression-language-functions.md#toUpper) | Retourne une chaîne en majuscules. |
| [trim](control-flow-expression-language-functions.md#trim) | Supprime les espaces blancs de début et de fin d’une chaîne et retourne la chaîne mise à jour. |

## <a name="collection-functions"></a>Fonctions de collection

Pour travailler avec des collections, généralement des tableaux, des chaînes et parfois, des dictionnaires, vous pouvez utiliser ces fonctions de collection.

| Fonction de collection | Tâche |
| ------------------- | ---- |
| [contains](control-flow-expression-language-functions.md#contains) | Vérifie si une collection contient un élément spécifique. |
| [empty](control-flow-expression-language-functions.md#empty) | Vérifie si une collection est vide. |
| [first](control-flow-expression-language-functions.md#first) | Renvoyer le premier élément d’une collection. |
| [intersection](control-flow-expression-language-functions.md#intersection) | Retourne une collection qui contient *uniquement* les éléments communs aux collections spécifiées. |
| [join](control-flow-expression-language-functions.md#join) | Renvoyer une chaîne qui contient *tous* les éléments d’un tableau, séparés par le caractère spécifié. |
| [last](control-flow-expression-language-functions.md#last) | Retourne le dernier élément d’une collection. |
| [length](control-flow-expression-language-functions.md#length) | Renvoyer le nombre d’éléments d’une chaîne ou d’un tableau. |
| [skip](control-flow-expression-language-functions.md#skip) | Supprime des éléments du début d’une collection et retourne *tous les autres* éléments. |
| [take](control-flow-expression-language-functions.md#take) | Retourne des éléments du début d’une collection. |
| [union](control-flow-expression-language-functions.md#union) | Retourne une collection qui contient *tous* les éléments des collections spécifiées. | 

## <a name="logical-functions"></a>Fonctions logiques  

Ces fonctions sont utiles à l’intérieur de conditions, et permettent d’évaluer tout type de logique.  
  
| Fonction de comparaison logique | Tâche |
| --------------------------- | ---- |
| [and](control-flow-expression-language-functions.md#and) | Vérifie si toutes les expressions sont vraies. |
| [equals](control-flow-expression-language-functions.md#equals) | Vérifier si les deux valeurs sont équivalentes. |
| [greater](control-flow-expression-language-functions.md#greater) | Vérifie si la première valeur est supérieure à la seconde. |
| [greaterOrEquals](control-flow-expression-language-functions.md#greaterOrEquals) | Vérifie si la première valeur est supérieure ou égale à la seconde. |
| [if](control-flow-expression-language-functions.md#if) | Vérifie si une expression est vraie ou fausse. En fonction du résultat, retourne une valeur spécifiée. |
| [less](control-flow-expression-language-functions.md#less) | Vérifie si la première valeur est inférieure à la seconde. |
| [lessOrEquals](control-flow-expression-language-functions.md#lessOrEquals) | Vérifie si la première valeur est inférieure ou égale à la seconde. |
| [not](control-flow-expression-language-functions.md#not) | Vérifie si une expression est fausse. |
| [or](control-flow-expression-language-functions.md#or) | Vérifie si au moins une expression est vraie. |
  
## <a name="conversion-functions"></a>Fonctions de conversion  

 Ces fonctions permettent de convertir chacun des types natifs du langage :  
-   string
-   entier
-   float
-   boolean
-   tableaux
-   dictionnaires

| Fonction de conversion | Tâche |
| ------------------- | ---- |
| [array](control-flow-expression-language-functions.md#array) | Retourne un tableau à partir d’une entrée spécifique unique. Pour des entrées multiples, consultez [createArray](control-flow-expression-language-functions.md#createArray). |
| [base64](control-flow-expression-language-functions.md#base64) | Retourne la version encodée en Base64 d’une chaîne. |
| [base64ToBinary](control-flow-expression-language-functions.md#base64ToBinary) | Retourne la version binaire d’une chaîne encodée en Base64. |
| [base64ToString](control-flow-expression-language-functions.md#base64ToString) | Renvoyer la version de type chaîne d’une chaîne encodée en base 64. |
| [binary](control-flow-expression-language-functions.md#binary) | Renvoyer la version binaire d’une valeur d’entrée. |
| [bool](control-flow-expression-language-functions.md#bool) | Renvoyer la version booléenne d’une valeur d’entrée. |
| [coalesce](control-flow-expression-language-functions.md#coalesce) | Retourne la première valeur autre que Null d’un ou plusieurs paramètres. |
| [createArray](control-flow-expression-language-functions.md#createArray) | Retourne un tableau à partir de plusieurs entrées. |
| [dataUri](control-flow-expression-language-functions.md#dataUri) | Renvoyer l’URI de données d’une valeur d’entrée. |
| [dataUriToBinary](control-flow-expression-language-functions.md#dataUriToBinary) | Renvoyer la version binaire d’un URI de données. |
| [dataUriToString](control-flow-expression-language-functions.md#dataUriToString) | Renvoyer la version de type chaîne d’un URI de données. |
| [decodeBase64](control-flow-expression-language-functions.md#decodeBase64) | Renvoyer la version de type chaîne d’une chaîne encodée en base 64. |
| [decodeDataUri](control-flow-expression-language-functions.md#decodeDataUri) | Renvoyer la version binaire d’un URI de données. |
| [decodeUriComponent](control-flow-expression-language-functions.md#decodeUriComponent) | Retourne une chaîne qui remplace les caractères d’échappement par des versions décodées. |
| [encodeUriComponent](control-flow-expression-language-functions.md#encodeUriComponent) | Renvoyer une chaîne qui remplace les caractères non sécurisés pour les URL par des caractères d'échappement. |
| [float](control-flow-expression-language-functions.md#float) | Renvoyer un nombre à virgule flottante pour une valeur d’entrée. |
| [int](control-flow-expression-language-functions.md#int) | Retourne la version de type entier d’une chaîne. |
| [json](control-flow-expression-language-functions.md#json) | Retourne la valeur ou l’objet de type JavaScript Object Notation (JSON) d’une chaîne ou d’un élément XML. |
| [string](control-flow-expression-language-functions.md#string) | Renvoyer la version de type chaîne d’une valeur d’entrée. |
| [uriComponent](control-flow-expression-language-functions.md#uriComponent) | Renvoyer la version encodée dans un URI d’une valeur d’entrée en remplaçant les caractères non sécurisés pour les URL par des caractères d’échappement. |
| [uriComponentToBinary](control-flow-expression-language-functions.md#uriComponentToBinary) | Renvoyer la version binaire d’une chaîne encodée dans un URI. |
| [uriComponentToString](control-flow-expression-language-functions.md#uriComponentToString) | Renvoyer la version de type chaîne d’une chaîne encodée dans un URI. |
| [xml](control-flow-expression-language-functions.md#xml) | Renvoyer la version de type entier d’une chaîne. |
| [xpath](control-flow-expression-language-functions.md#xpath) | Vérifie si le code XML contient des valeurs ou des nœuds qui correspondent à une expression de langage XPath et retourne les valeurs ou les nœuds correspondants. |

## <a name="math-functions"></a>Fonctions mathématiques  
 Ces fonctions peuvent être utilisées pour les deux types de nombre : **entiers** et **flottants**.  

| Fonction mathématique | Tâche |
| ------------- | ---- |
| [add](control-flow-expression-language-functions.md#add) | Retourne le résultat de l’addition de deux nombres. |
| [div](control-flow-expression-language-functions.md#div) | Renvoyer le résultat de la division de deux nombres. |
| [max](control-flow-expression-language-functions.md#max) | Renvoyer la valeur la plus élevée d’un ensemble de nombres ou d’un tableau. |
| [min](control-flow-expression-language-functions.md#min) | Retourne la plus petite valeur d’un ensemble de nombres ou d’un tableau. |
| [mod](control-flow-expression-language-functions.md#mod) | Retourne le reste de la division de deux nombres. |
| [mul](control-flow-expression-language-functions.md#mul) | Retourne le produit de la multiplication de deux nombres. |
| [rand](control-flow-expression-language-functions.md#rand) | Renvoyer un entier aléatoire à partir d’une plage spécifique. |
| [range](control-flow-expression-language-functions.md#range) | Retourne un tableau d’entiers qui commence par un entier spécifique. |
| [sub](control-flow-expression-language-functions.md#sub) | Retourne le résultat de la soustraction du second nombre du premier. |
  
## <a name="date-functions"></a>Fonctions de date  

| Fonction de date ou heure | Tâche |
| --------------------- | ---- |
| [addDays](control-flow-expression-language-functions.md#addDays) | Ajoute un nombre de jours à un horodatage. |
| [addHours](control-flow-expression-language-functions.md#addHours) | Ajoute un nombre d’heures à un horodatage. |
| [addMinutes](control-flow-expression-language-functions.md#addMinutes) | Ajoute un nombre de minutes à un horodatage. |
| [addSeconds](control-flow-expression-language-functions.md#addSeconds) | Ajoute un nombre de secondes à un horodatage. |
| [addToTime](control-flow-expression-language-functions.md#addToTime) | Ajoute un nombre d’unités de temps à un horodatage. Voir aussi [getFutureTime](control-flow-expression-language-functions.md#getFutureTime). |
| [convertFromUtc](control-flow-expression-language-functions.md#convertFromUtc) | Convertit un horodatage du temps universel coordonné (UTC) au fuseau horaire cible. |
| [convertTimeZone](control-flow-expression-language-functions.md#convertTimeZone) | Convertit un horodatage du fuseau horaire source au fuseau horaire cible. |
| [convertToUtc](control-flow-expression-language-functions.md#convertToUtc) | Convertit un horodatage du fuseau horaire source en temps universel coordonné (UTC). |
| [dayOfMonth](control-flow-expression-language-functions.md#dayOfMonth) | Renvoyer le jour du composant mois d’un timestamp. |
| [dayOfWeek](control-flow-expression-language-functions.md#dayOfWeek) | Renvoyer le jour du composant semaine d’un timestamp. |
| [dayOfYear](control-flow-expression-language-functions.md#dayOfYear) | Renvoyer le jour du composant année d’un timestamp. |
| [formatDateTime](control-flow-expression-language-functions.md#formatDateTime) | Retourner l’horodatage sous forme de chaîne dans un format facultatif. |
| [getFutureTime](control-flow-expression-language-functions.md#getFutureTime) | Retourne l’horodatage actuel plus les unités de temps spécifiées. Voir aussi [addToTime](control-flow-expression-language-functions.md#addToTime). |
| [getPastTime](control-flow-expression-language-functions.md#getPastTime) | Retourne l’horodatage actuel moins les unités de temps spécifiées. Voir aussi [subtractFromTime](control-flow-expression-language-functions.md#subtractFromTime). |
| [startOfDay](control-flow-expression-language-functions.md#startOfDay) | Retourne le début du jour d’un horodatage. |
| [startOfHour](control-flow-expression-language-functions.md#startOfHour) | Retourne le début de l’heure d’un horodatage. |
| [startOfMonth](control-flow-expression-language-functions.md#startOfMonth) | Retourne le début du mois pour un horodatage. |
| [subtractFromTime](control-flow-expression-language-functions.md#subtractFromTime) | Soustrait un nombre d’unités de temps d’un horodatage. Voir aussi [getPastTime](control-flow-expression-language-functions.md#getPastTime). |
| [ticks](control-flow-expression-language-functions.md#ticks) | Retourne la valeur de la propriété `ticks` pour un horodatage spécifique. |
| [utcNow](control-flow-expression-language-functions.md#utcNow) | Renvoyer le timestamp actuel sous forme de chaîne. |

## <a name="function-reference"></a>Référence des fonctions

Cette section répertorie toutes les fonctions disponibles par ordre alphabétique.

<a name="add"></a>

### <a name="add"></a>add

Retourne le résultat de l’addition de deux nombres.

```
add(<summand_1>, <summand_2>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*summand_1*>, <*summand_2*> | Oui | Entier, flottant ou mixte | Nombres à ajouter |
|||||

| Valeur retournée | Type | Description |
| ------------ | -----| ----------- |
| <*result-sum*> | Entier ou flottant | Résultat de l’addition des nombres spécifiés |
||||

*Exemple*

Cet exemple illustre l’addition des nombres spécifiés :

```
add(1, 1.5)
```

Et retourne ce résultat : `2.5`

<a name="addDays"></a>

### <a name="adddays"></a>addDays

Ajoute un nombre de jours à un horodatage.

```
addDays('<timestamp>', <days>, '<format>'?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
| <*days*> | Oui | Integer | Nombre de jours positif ou négatif à ajouter |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updated-timestamp*> | String | Horodatage plus le nombre de jours spécifié  |
||||

*Exemple 1*

Cet exemple illustre l’ajout de 10 jours à l’horodatage spécifié :

```
addDays('2018-03-15T13:00:00Z', 10)
```

Et retourne ce résultat : `"2018-03-25T00:00:0000000Z"`

*Exemple 2*

Cet exemple illustre le retrait de cinq jours de l’horodatage spécifié :

```
addDays('2018-03-15T00:00:00Z', -5)
```

Et retourne ce résultat : `"2018-03-10T00:00:0000000Z"`

<a name="addHours"></a>

### <a name="addhours"></a>addHours

Ajoute un nombre d’heures à un horodatage.

```
addHours('<timestamp>', <hours>, '<format>'?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
| <*hours*> | Oui | Integer | Nombre d’heures positif ou négatif à ajouter |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updated-timestamp*> | String | Horodatage plus le nombre d’heures spécifié  |
||||

*Exemple 1*

Cet exemple illustre l’ajout de 10 heures à l’horodatage spécifié :

```
addHours('2018-03-15T00:00:00Z', 10)
```

Et retourne ce résultat : `"2018-03-15T10:00:0000000Z"`

*Exemple 2*

Cet exemple illustre le retrait de cinq heures de l’horodatage spécifié :

```
addHours('2018-03-15T15:00:00Z', -5)
```

Et retourne ce résultat : `"2018-03-15T10:00:0000000Z"`

<a name="addMinutes"></a>

### <a name="addminutes"></a>addMinutes

Ajoute un nombre de minutes à un horodatage.

```
addMinutes('<timestamp>', <minutes>, '<format>'?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
| <*minutes*> | Oui | Integer | Nombre de minutes positif ou négatif à ajouter |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updated-timestamp*> | String | Horodatage plus le nombre de minutes spécifié |
||||

*Exemple 1*

Cet exemple illustre l’ajout de 10 minutes à l’horodatage spécifié :

```
addMinutes('2018-03-15T00:10:00Z', 10)
```

Et retourne ce résultat : `"2018-03-15T00:20:00.0000000Z"`

*Exemple 2*

Cet exemple illustre le retrait de cinq minutes de l’horodatage spécifié :

```
addMinutes('2018-03-15T00:20:00Z', -5)
```

Et retourne ce résultat : `"2018-03-15T00:15:00.0000000Z"`

<a name="addSeconds"></a>

### <a name="addseconds"></a>addSeconds

Ajoute un nombre de secondes à un horodatage.

```
addSeconds('<timestamp>', <seconds>, '<format>'?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
| <*seconds*> | Oui | Integer | Nombre de secondes positif ou négatif à ajouter |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updated-timestamp*> | String | Horodatage plus le nombre de secondes spécifié  |
||||

*Exemple 1*

Cet exemple illustre l’ajout de 10 secondes à l’horodatage spécifié :

```
addSeconds('2018-03-15T00:00:00Z', 10)
```

Et retourne ce résultat : `"2018-03-15T00:00:10.0000000Z"`

*Exemple 2*

Cet exemple illustre le retrait de cinq secondes de l’horodatage spécifié :

```
addSeconds('2018-03-15T00:00:30Z', -5)
```

Et retourne ce résultat : `"2018-03-15T00:00:25.0000000Z"`

<a name="addToTime"></a>

### <a name="addtotime"></a>addToTime

Ajoute un nombre d’unités de temps à un horodatage.
Voir aussi [getFutureTime()](#getFutureTime).

```
addToTime('<timestamp>', <interval>, '<timeUnit>', '<format>'?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
| <*interval*> | Oui | Integer | Nombre d’unités de temps spécifiées à ajouter |
| <*timeUnit*> | Oui | String | L’unité de temps à utiliser avec *interval* : "Second", "Minute", "Hour", "Day", "Week", "Month", "Year" |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updated-timestamp*> | String | Horodatage plus nombre d’unités de temps spécifié  |
||||

*Exemple 1*

Cet exemple illustre l’ajout d’un jour à l’horodatage spécifié :

```
addToTime('2018-01-01T00:00:00Z', 1, 'Day')
```

Et retourne ce résultat : `"2018-01-02T00:00:00.0000000Z"`

*Exemple 2*

Cet exemple illustre l’ajout d’un jour à l’horodatage spécifié :

```
addToTime('2018-01-01T00:00:00Z', 1, 'Day', 'D')
```

Et retourne le résultat en utilisant le format « D » facultatif : `"Tuesday, January 2, 2018"`

<a name="and"></a>

### <a name="and"></a>and

Vérifie si les deux expressions sont vraies.
Retourne la valeur true lorsque les deux expressions sont vraies ou la valeur false lorsque au moins une expression est fausse.

```
and(<expression1>, <expression2>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*expression1*>, <*expression2*> | Oui | Boolean | Expressions à vérifier |
|||||

| Valeur retournée | Type | Description |
| ------------ | -----| ----------- |
| True ou False | Boolean | Retourne la valeur true lorsque les deux expressions sont vraies. Retourne la valeur false si au moins une expression est fausse. |
||||

*Exemple 1*

Ces exemples vérifient si les valeurs booléennes spécifiées sont vraies :

```
and(true, true)
and(false, true)
and(false, false)
```

Et retournent les résultats suivants :

* Premier exemple : comme les deux expressions sont vraies, la valeur `true` est retournée.
* Deuxième exemple : comme une des expressions est fausse, la valeur `false` est retournée.
* Troisième exemple : comme les deux expressions sont fausses, la valeur `false` est retournée.

*Exemple 2*

Ces exemples vérifient si les expressions spécifiées sont vraies :

```
and(equals(1, 1), equals(2, 2))
and(equals(1, 1), equals(1, 2))
and(equals(1, 2), equals(1, 3))
```

Et retournent les résultats suivants :

* Premier exemple : comme les deux expressions sont vraies, la valeur `true` est retournée.
* Deuxième exemple : comme une des expressions est fausse, la valeur `false` est retournée.
* Troisième exemple : comme les deux expressions sont fausses, la valeur `false` est retournée.

<a name="array"></a>

### <a name="array"></a>tableau

Retourne un tableau à partir d’une entrée spécifique unique.
Pour des entrées multiples, consultez [createArray()](#createArray).

```
array('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne permettant de créer un tableau |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| [<*valeur*>] | Array | Tableau qui contient l’entrée spécifiée unique |
||||

*Exemple*

Cet exemple illustre la création d’un tableau à partir de la chaîne « hello » :

```
array('hello')
```

Et retourne ce résultat : `["hello"]`

<a name="base64"></a>

### <a name="base64"></a>base64

Retourne la version encodée en Base64 d’une chaîne.

```
base64('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne d’entrée |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*base64-string*> | String | Retourne la version encodée en Base64 de la chaîne d’entrée |
||||

*Exemple*

Cet exemple illustre la conversion d’une chaîne « hello » en chaîne encodée en Base64 :

```
base64('hello')
```

Et retourne ce résultat : `"aGVsbG8="`

<a name="base64ToBinary"></a>

### <a name="base64tobinary"></a>base64ToBinary

Retourne la version binaire d’une chaîne encodée en Base64.

```
base64ToBinary('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne encodée en Base64 à convertir |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*binary-for-base64-string*> | String | Version binaire de la chaîne encodée en Base64 |
||||

*Exemple*

Cet exemple illustre la conversion d’une chaîne « aGVsbG8= » encodée en Base64 en chaîne binaire :

```
base64ToBinary('aGVsbG8=')
```

Et retourne ce résultat :

`"0110000101000111010101100111001101100010010001110011100000111101"`

<a name="base64ToString"></a>

### <a name="base64tostring"></a>base64ToString

Retourne la version de type chaîne d’une chaîne encodée en Base64, en décodant efficacement la chaîne Base64.
Utilisez cette fonction plutôt que la fonction [decodeBase64()](#decodeBase64).
Bien que les deux fonctions agissent de manière identique, la fonction `base64ToString()` est préférée.

```
base64ToString('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne encodée en Base64 à décoder |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*decoded-base64-string*> | String | Version de type chaîne d’une chaîne encodée en Base64 |
||||

*Exemple*

Cet exemple illustre la conversion d’une chaîne « aGVsbG8= » encodée en Base64 en chaîne :

```
base64ToString('aGVsbG8=')
```

Et retourne ce résultat : `"hello"`

<a name="binary"></a>

### <a name="binary"></a>binary

Retourne la version binaire d’une chaîne.

```
binary('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne à convertir |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*binary-for-input-value*> | String | Version binaire de la chaîne spécifiée |
||||

*Exemple*

Cet exemple illustre la conversion d’une chaîne « hello » en chaîne binaire :

```
binary('hello')
```

Et retourne ce résultat :

`"0110100001100101011011000110110001101111"`

<a name="bool"></a>

### <a name="bool"></a>bool

Retourne la version booléenne d’une valeur.

```
bool(<value>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | Quelconque | Valeur à convertir |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| True ou False | Boolean | Version booléenne de la valeur spécifiée |
||||

*Exemple*

Ces exemples illustrent la conversion des valeurs spécifiées en valeurs booléennes :

```
bool(1)
bool(0)
```

Et retournent les résultats suivants :

* Premier exemple : `true`
* Second exemple : `false`

<a name="coalesce"></a>

### <a name="coalesce"></a>coalesce

Retourne la première valeur autre que Null d’un ou plusieurs paramètres.
Les chaînes vides, les tableaux vides et les objets vides ne sont pas null.

```
coalesce(<object_1>, <object_2>, ...)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*object_1*>, <*object_2*>... | Oui | N’importe lequel, possibilité de combiner plusieurs types | Recherche de la valeur Null dans un ou plusieurs éléments |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*first-non-null-item*> | Quelconque | Premier élément ou valeur autre que Null. Si tous les paramètres ont la valeur Null, cette fonction retourne la valeur Null. |
||||

*Exemple*

Ces exemples retournent la première valeur autre que Null à partir des valeurs spécifiées ou la valeur Null lorsque toutes les valeurs le sont :

```
coalesce(null, true, false)
coalesce(null, 'hello', 'world')
coalesce(null, null, null)
```

Et retournent les résultats suivants :

* Premier exemple : `true`
* Second exemple : `"hello"`
* Troisième exemple : `null`

<a name="concat"></a>

### <a name="concat"></a>concat

Combine au moins deux chaînes et retourne la chaîne combinée.

```
concat('<text1>', '<text2>', ...)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*text1*>, <*text2*>... | Oui | String | Au moins deux chaînes à combiner |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*text1text2...* > | String | Chaîne créée à partir des chaînes d’entrée combinées |
||||

*Exemple*

Cet exemple illustre la combinaison des chaînes « Hello » et « World » :

```
concat('Hello', 'World')
```

Et retourne ce résultat : `"HelloWorld"`

<a name="contains"></a>

### <a name="contains"></a>contains

Vérifie si une collection contient un élément spécifique.
Retourne la valeur true lorsque l’élément est trouvé ou la valeur false lorsqu’il est introuvable.
Cette fonction respecte la casse.

```
contains('<collection>', '<value>')
contains([<collection>], '<value>')
```

Plus précisément, cette fonction agit sur ces types de collection :

* Une *chaîne* pour y rechercher une *sous-chaîne*
* Un *tableau* pour y rechercher une *valeur*
* Un *ensemble de clés* pour y rechercher une *clé*

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*collection*> | Oui | Chaîne, tableau ou ensemble de clés | Collection à vérifier |
| <*value*> | Oui | Respectivement, chaîne, tableau ou ensemble de clés | Élément à rechercher |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| True ou False | Boolean | Retourne la valeur true lorsque l’élément est trouvé. Retourne la valeur false lorsqu’elle est introuvable. |
||||

*Exemple 1*

Cet exemple recherche dans la chaîne « hello world » la sous-chaîne « world » et retourne la valeur true :

```
contains('hello world', 'world')
```

*Exemple 2*

Cet exemple recherche dans la chaîne « hello world » la sous-chaîne « universe » et retourne la valeur false :

```
contains('hello world', 'universe')
```

<a name="convertFromUtc"></a>

### <a name="convertfromutc"></a>convertFromUtc

Convertit un horodatage du temps universel coordonné (UTC) au fuseau horaire cible.

```
convertFromUtc('<timestamp>', '<destinationTimeZone>', '<format>'?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
| <*destinationTimeZone*> | Oui | String | Nom du fuseau horaire cible. Pour les noms de fuseau horaire, consultez [Valeurs d’index de fuseau horaire Microsoft](https://support.microsoft.com/help/973627/microsoft-time-zone-index-values), mais vous devrez peut-être supprimer les signes de ponctuation du nom de fuseau horaire. |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*converted-timestamp*> | String | Horodatage converti au fuseau horaire cible |
||||

*Exemple 1*

Cet exemple illustre la conversion d’un horodatage au fuseau horaire spécifié :

```
convertFromUtc('2018-01-01T08:00:00.0000000Z', 'Pacific Standard Time')
```

Et retourne ce résultat : `"2018-01-01T00:00:00Z"`

*Exemple 2*

Cet exemple illustre la conversion d’un horodatage au fuseau horaire et au format spécifiés :

```
convertFromUtc('2018-01-01T08:00:00.0000000Z', 'Pacific Standard Time', 'D')
```

Et retourne ce résultat : `"Monday, January 1, 2018"`

<a name="convertTimeZone"></a>

### <a name="converttimezone"></a>convertTimeZone

Convertit un horodatage du fuseau horaire source au fuseau horaire cible.

```
convertTimeZone('<timestamp>', '<sourceTimeZone>', '<destinationTimeZone>', '<format>'?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
| <*sourceTimeZone*> | Oui | String | Nom du fuseau horaire source. Pour les noms de fuseau horaire, consultez [Valeurs d’index de fuseau horaire Microsoft](https://support.microsoft.com/help/973627/microsoft-time-zone-index-values), mais vous devrez peut-être supprimer les signes de ponctuation du nom de fuseau horaire. |
| <*destinationTimeZone*> | Oui | String | Nom du fuseau horaire cible. Pour les noms de fuseau horaire, consultez [Valeurs d’index de fuseau horaire Microsoft](https://support.microsoft.com/help/973627/microsoft-time-zone-index-values), mais vous devrez peut-être supprimer les signes de ponctuation du nom de fuseau horaire. |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*converted-timestamp*> | String | Horodatage converti au fuseau horaire cible |
||||

*Exemple 1*

Cet exemple illustre la conversion du fuseau horaire source au fuseau horaire cible :

```
convertTimeZone('2018-01-01T08:00:00.0000000Z', 'UTC', 'Pacific Standard Time')
```

Et retourne ce résultat : `"2018-01-01T00:00:00.0000000"`

*Exemple 2*

Cet exemple illustre la conversion d’un fuseau horaire au format et au fuseau horaire spécifiés :

```
convertTimeZone('2018-01-01T80:00:00.0000000Z', 'UTC', 'Pacific Standard Time', 'D')
```

Et retourne ce résultat : `"Monday, January 1, 2018"`

<a name="convertToUtc"></a>

### <a name="converttoutc"></a>convertToUtc

Convertit un horodatage du fuseau horaire source en temps universel coordonné (UTC).

```
convertToUtc('<timestamp>', '<sourceTimeZone>', '<format>'?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
| <*sourceTimeZone*> | Oui | String | Nom du fuseau horaire source. Pour les noms de fuseau horaire, consultez [Valeurs d’index de fuseau horaire Microsoft](https://support.microsoft.com/help/973627/microsoft-time-zone-index-values), mais vous devrez peut-être supprimer les signes de ponctuation du nom de fuseau horaire. |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*converted-timestamp*> | String | Horodatage converti en heure UTC |
||||

*Exemple 1*

Cet exemple illustre la conversion d’un horodatage en heure UTC :

```
convertToUtc('01/01/2018 00:00:00', 'Pacific Standard Time')
```

Et retourne ce résultat : `"2018-01-01T08:00:00.0000000Z"`

*Exemple 2*

Cet exemple illustre la conversion d’un horodatage en heure UTC :

```
convertToUtc('01/01/2018 00:00:00', 'Pacific Standard Time', 'D')
```

Et retourne ce résultat : `"Monday, January 1, 2018"`

<a name="createArray"></a>

### <a name="createarray"></a>createArray

Retourne un tableau à partir de plusieurs entrées.
Pour les tableaux à entrée unique, consultez [array()](#array).

```
createArray('<object1>', '<object2>', ...)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*object1*>, <*object2*>... | Oui | N’importe lequel, mais pas de combinaison | Au moins deux éléments pour créer le tableau |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| [<*object1*>, <*object2*>...] | Array | Tableau créé à partir de tous les éléments d’entrée |
||||

*Exemple*

Cet exemple illustre la création d’un tableau à partir de ces entrées :

```
createArray('h', 'e', 'l', 'l', 'o')
```

Et retourne ce résultat : `["h", "e", "l", "l", "o"]`

<a name="dataUri"></a>

### <a name="datauri"></a>dataUri

Retourne un URI de données pour une chaîne.

```
dataUri('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne à convertir |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*data-uri*> | String | URI de données de la chaîne d’entrée |
||||

*Exemple*

Cet exemple illustre la création d’un URI de données à partir de la chaîne « hello » :

```
dataUri('hello')
```

Et retourne ce résultat : `"data:text/plain;charset=utf-8;base64,aGVsbG8="`

<a name="dataUriToBinary"></a>

### <a name="datauritobinary"></a>dataUriToBinary

Retourne la version binaire d’un URI (Uniform Resource Identifier) de données.
Utilisez cette fonction plutôt que la fonction [decodeDataUri()](#decodeDataUri).
Bien que les deux fonctions agissent de manière identique, la fonction `dataUriBinary()` est préférée.

```
dataUriToBinary('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | URI de données à convertir |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*binary-for-data-uri*> | String | Version binaire de l’URI de données |
||||

*Exemple*

Cet exemple illustre la création d’une version binaire pour cet URI de données :

```
dataUriToBinary('data:text/plain;charset=utf-8;base64,aGVsbG8=')
```

Et retourne ce résultat :

`"01100100011000010111010001100001001110100111010001100101011110000111010000101111011100000
1101100011000010110100101101110001110110110001101101000011000010111001001110011011001010111
0100001111010111010101110100011001100010110100111000001110110110001001100001011100110110010
10011011000110100001011000110000101000111010101100111001101100010010001110011100000111101"`

<a name="dataUriToString"></a>

### <a name="datauritostring"></a>dataUriToString

Retourne la version de type chaîne d’un URI de données.

```
dataUriToString('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | URI de données à convertir |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*string-for-data-uri*> | String | Version de type chaîne de l’URI de données |
||||

*Exemple*

Cet exemple illustre la création d’une chaîne pour cet URI de données :

```
dataUriToString('data:text/plain;charset=utf-8;base64,aGVsbG8=')
```

Et retourne ce résultat : `"hello"`

<a name="dayOfMonth"></a>

### <a name="dayofmonth"></a>dayOfMonth

Retourne le jour du mois d’un horodatage.

```
dayOfMonth('<timestamp>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*day-of-month*> | Integer | Jour du mois de l’horodatage spécifié |
||||

*Exemple*

Cet exemple retourne le numéro du jour du mois à partir de cet horodatage :

```
dayOfMonth('2018-03-15T13:27:36Z')
```

Et retourne ce résultat : `15`

<a name="dayOfWeek"></a>

### <a name="dayofweek"></a>dayOfWeek

Retourne le jour de la semaine à partir d’un horodatage.

```
dayOfWeek('<timestamp>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*day-of-week*> | Integer | Jour de la semaine à partir de l’horodatage spécifié où dimanche représente 0, lundi représente 1, etc. |
||||

*Exemple*

Cet exemple retourne le numéro du jour de la semaine à partir de cet horodatage :

```
dayOfWeek('2018-03-15T13:27:36Z')
```

Et retourne ce résultat : `3`

<a name="dayOfYear"></a>

### <a name="dayofyear"></a>dayOfYear

Retourne le jour de l’année à partir d’un horodatage.

```
dayOfYear('<timestamp>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*day-of-year*> | Integer | Jour de l’année à partir de l’horodatage spécifié |
||||

*Exemple*

Cet exemple retourne le numéro du jour de l’année à partir de cet horodatage :

```
dayOfYear('2018-03-15T13:27:36Z')
```

Et retourne ce résultat : `74`

<a name="decodeBase64"></a>

### <a name="decodebase64"></a>decodeBase64

Retourne la version de type chaîne d’une chaîne encodée en Base64, en décodant efficacement la chaîne Base64.
Pensez à utiliser la fonction [base64ToString()](#base64ToString) plutôt que la fonction `decodeBase64()`.
Bien que les deux fonctions agissent de manière identique, la fonction `base64ToString()` est préférée.

```
decodeBase64('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne encodée en Base64 à décoder |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*decoded-base64-string*> | String | Version de type chaîne d’une chaîne encodée en Base64 |
||||

*Exemple*

Cet exemple illustre la création d’une chaîne pour une chaîne encodée en Base64 :

```
decodeBase64('aGVsbG8=')
```

Et retourne ce résultat : `"hello"`

<a name="decodeDataUri"></a>

### <a name="decodedatauri"></a>decodeDataUri

Retourne la version binaire d’un URI (Uniform Resource Identifier) de données.
Pensez à utiliser la fonction [dataUriToBinary()](#dataUriToBinary) plutôt que la fonction `decodeDataUri()`.
Bien que les deux fonctions agissent de manière identique, la fonction `dataUriToBinary()` est préférée.

```
decodeDataUri('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne d’URI de données à décoder |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*binary-for-data-uri*> | String | Version binaire d’une chaîne d’URI de données |
||||

*Exemple*

Cet exemple retourne la version binaire de cet URI de données :

```
decodeDataUri('data:text/plain;charset=utf-8;base64,aGVsbG8=')
```

Et retourne ce résultat :

`"01100100011000010111010001100001001110100111010001100101011110000111010000101111011100000
1101100011000010110100101101110001110110110001101101000011000010111001001110011011001010111
0100001111010111010101110100011001100010110100111000001110110110001001100001011100110110010
10011011000110100001011000110000101000111010101100111001101100010010001110011100000111101"`

<a name="decodeUriComponent"></a>

### <a name="decodeuricomponent"></a>decodeUriComponent

Retourne une chaîne qui remplace les caractères d’échappement par des versions décodées.

```
decodeUriComponent('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne contenant les caractères d’échappement à décoder |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*decoded-uri*> | String | Chaîne mise à jour contenant les caractères d’échappement décodés |
||||

*Exemple*

Cet exemple illustre le remplacement des caractères d’échappement de cette chaîne par des versions décodées :

```
decodeUriComponent('http%3A%2F%2Fcontoso.com')
```

Et retourne ce résultat : `"https://contoso.com"`

<a name="div"></a>

### <a name="div"></a>div

Retourne l’entier résultant de la division de deux nombres.
Pour obtenir le reste, consultez [mod()](#mod).

```
div(<dividend>, <divisor>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*dividend*> | Oui | Entier ou flottant | Nombre à diviser par le *diviseur* |
| <*divisor*> | Oui | Entier ou flottant | Nombre qui divise le *dividende*, mais qui ne peut pas être égal à 0 |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*quotient-result*> | Integer | Entier résultant de la division du premier nombre par le second nombre |
||||

*Exemple*

Les deux exemples illustrent la division du premier nombre par le second :

```
div(10, 5)
div(11, 5)
```

Et retourne ce résultat : `2`

<a name="encodeUriComponent"></a>

### <a name="encodeuricomponent"></a>encodeUriComponent

Retourne une version encodée sous forme d’URI d’une chaîne en remplaçant les caractères non sécurisés pour les URL par des caractères d’échappement.
Pensez à utiliser la fonction [uriComponent()](#uriComponent) plutôt que la fonction `encodeUriComponent()`.
Bien que les deux fonctions agissent de manière identique, la fonction `uriComponent()` est préférée.

```
encodeUriComponent('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne à convertir au format encodé sous forme d’URI |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*encoded-uri*> | String | Chaîne encodée sous forme d’URI avec des caractères d’échappement |
||||

*Exemple*

Cet exemple illustre la création d’une version encodée sous forme d’URI pour cette chaîne :

```
encodeUriComponent('https://contoso.com')
```

Et retourne ce résultat : `"http%3A%2F%2Fcontoso.com"`

<a name="empty"></a>

### <a name="empty"></a>empty

Vérifie si une collection est vide.
Retourne la valeur true lorsque la collection est vide ou la valeur false dans le cas contraire.

```
empty('<collection>')
empty([<collection>])
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*collection*> | Oui | Chaîne, tableau ou objet | Collection à vérifier |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| True ou False | Boolean | Retourne la valeur true lorsque la collection est vide. Retourne la valeur false lorsque la collection n’est pas vide. |
||||

*Exemple*

Ces exemples vérifient si les collections spécifiées sont vides :

```
empty('')
empty('abc')
```

Et retournent les résultats suivants :

* Premier exemple : comme une chaîne vide est passée, la fonction retourne `true`.
* Deuxième exemple : comme la chaîne « abc » est passée, la fonction retourne `false`.

<a name="endswith"></a>

### <a name="endswith"></a>endsWith

Vérifie si une chaîne se termine par une sous-chaîne spécifique.
Retourne la valeur true lorsque la sous-chaîne est trouvée ou la valeur false lorsqu’elle est introuvable.
Cette fonction ne respecte pas la casse.

```
endsWith('<text>', '<searchText>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*text*> | Oui | String | Chaîne à vérifier |
| <*searchText*> | Oui | String | Sous-chaîne de fin à rechercher |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| True ou False  | Boolean | Retourne la valeur true lorsque la sous-chaîne de fin est trouvée. Retourne la valeur false lorsqu’elle est introuvable. |
||||

*Exemple 1*

Cet exemple vérifie si la chaîne « hello world » se termine par la chaîne « world » :

```
endsWith('hello world', 'world')
```

Et retourne ce résultat : `true`

*Exemple 2*

Cet exemple vérifie si la chaîne « hello world » se termine par la chaîne « universe » :

```
endsWith('hello world', 'universe')
```

Et retourne ce résultat : `false`

<a name="equals"></a>

### <a name="equals"></a>equals

Vérifie si les deux valeurs, expressions ou objets sont équivalents.
Retourne la valeur true si les deux sont équivalents ou la valeur false s’ils ne le sont pas.

```
equals('<object1>', '<object2>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*object1*>, <*object2*> | Oui | Divers | Valeurs, expressions ou objets à comparer |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| True ou False | Boolean | Retourne la valeur true lorsque les deux sont équivalents. Retourne la valeur false lorsque les deux ne sont pas équivalents. |
||||

*Exemple*

Ces exemples vérifient si les entrées spécifiées sont équivalentes.

```
equals(true, 1)
equals('abc', 'abcd')
```

Et retournent les résultats suivants :

* Premier exemple : comme les deux valeurs sont équivalentes, la fonction retourne `true`.
* Deuxième exemple : comme les deux valeurs ne sont pas équivalentes, la fonction retourne `false`.

<a name="first"></a>

### <a name="first"></a>first

Retourne le premier élément d’une chaîne ou d’un tableau.

```
first('<collection>')
first([<collection>])
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*collection*> | Oui | Chaîne ou tableau | Collection dans laquelle rechercher le premier élément |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*first-collection-item*> | Quelconque | Premier élément de la collection |
||||

*Exemple*

Ces exemples illustrent la recherche du premier élément dans ces collections :

```
first('hello')
first(createArray(0, 1, 2))
```

Et retournent les résultats suivants :

* Premier exemple : `"h"`
* Second exemple : `0`

<a name="float"></a>

### <a name="float"></a>float

Convertit une version de type chaîne d’un nombre à virgule flottante en nombre réel à virgule flottante.

```
float('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne qui contient un nombre valide à virgule flottante à convertir |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*float-value*> | Float | Nombre à virgule flottante de la chaîne spécifiée |
||||

*Exemple*

Cet exemple illustre la création d’une version de type chaîne pour ce nombre à virgule flottante :

```
float('10.333')
```

Et retourne ce résultat : `10.333`

<a name="formatDateTime"></a>

### <a name="formatdatetime"></a>formatDateTime

Retourne un horodatage au format spécifié.

```
formatDateTime('<timestamp>', '<format>'?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*reformatted-timestamp*> | String | Horodatage mis à jour au format spécifié |
||||

*Exemple*

Cet exemple illustre la conversion d’un horodatage au format spécifié :

```
formatDateTime('03/15/2018 12:00:00', 'yyyy-MM-ddTHH:mm:ss')
```

Et retourne ce résultat : `"2018-03-15T12:00:00"`

<a name="getFutureTime"></a>

### <a name="getfuturetime"></a>getFutureTime

Retourne l’horodatage actuel plus les unités de temps spécifiées.

```
getFutureTime(<interval>, <timeUnit>, <format>?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*interval*> | Oui | Integer | Nombre d’unités de temps spécifiées à ajouter |
| <*timeUnit*> | Oui | String | L’unité de temps à utiliser avec *interval* : "Second", "Minute", "Hour", "Day", "Week", "Month", "Year" |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updated-timestamp*> | String | Horodatage actuel plus nombre d’unités de temps spécifié |
||||

*Exemple 1*

Supposez que l’horodatage actuel soit « 2018-03-01T00:00:00.0000000Z ».
Cet exemple illustre l’ajout de cinq jours à cet horodatage :

```
getFutureTime(5, 'Day')
```

Et retourne ce résultat : `"2018-03-06T00:00:00.0000000Z"`

*Exemple 2*

Supposez que l’horodatage actuel soit « 2018-03-01T00:00:00.0000000Z ».
Cet exemple illustre l’ajout de cinq jours et convertit le résultat au format « D » :

```
getFutureTime(5, 'Day', 'D')
```

Et retourne ce résultat : `"Tuesday, March 6, 2018"`

<a name="getPastTime"></a>

### <a name="getpasttime"></a>getPastTime

Retourne l’horodatage actuel moins les unités de temps spécifiées.

```
getPastTime(<interval>, <timeUnit>, <format>?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*interval*> | Oui | Integer | Nombre d’unités de temps spécifiées à soustraire |
| <*timeUnit*> | Oui | String | L’unité de temps à utiliser avec *interval* : "Second", "Minute", "Hour", "Day", "Week", "Month", "Year" |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updated-timestamp*> | String | Horodatage actuel moins nombre d’unités de temps spécifié |
||||

*Exemple 1*

Supposez que l’horodatage actuel soit « 2018-02-01T00:00:00.0000000Z ».
Cet exemple illustre le retrait de cinq jours de cet horodatage :

```
getPastTime(5, 'Day')
```

Et retourne ce résultat : `"2018-01-27T00:00:00.0000000Z"`

*Exemple 2*

Supposez que l’horodatage actuel soit « 2018-02-01T00:00:00.0000000Z ».
Cet exemple illustre le retrait de cinq jours et convertit le résultat au format « D » :

```
getPastTime(5, 'Day', 'D')
```

Et retourne ce résultat : `"Saturday, January 27, 2018"`

<a name="greater"></a>

### <a name="greater"></a>greater

Vérifie si la première valeur est supérieure à la seconde.
Retourne la valeur true si la première valeur est supérieure ou la valeur false si elle est inférieure.

```
greater(<value>, <compareTo>)
greater('<value>', '<compareTo>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | Entier, flottant ou chaîne | Première valeur à vérifier pour déterminer si elle est supérieure à la seconde |
| <*compareTo*> | Oui | Respectivement entier, flottant ou chaîne | Valeur de comparaison |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| True ou False | Boolean | Retourne la valeur true si la première valeur est supérieure à la seconde. Retourne la valeur false si la première valeur est égale ou inférieure à la seconde. |
||||

*Exemple*

Ces exemples vérifient si la première valeur est supérieure à la seconde :

```
greater(10, 5)
greater('apple', 'banana')
```

Et retournent les résultats suivants :

* Premier exemple : `true`
* Second exemple : `false`

<a name="greaterOrEquals"></a>

### <a name="greaterorequals"></a>greaterOrEquals

Vérifie si la première valeur est supérieure ou égale à la seconde.
Retourne la valeur true si la première valeur est supérieure ou égale à la seconde, ou la valeur false si la première valeur est inférieure à la seconde.

```
greaterOrEquals(<value>, <compareTo>)
greaterOrEquals('<value>', '<compareTo>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | Entier, flottant ou chaîne | Première valeur à vérifier pour déterminer si elle est supérieure ou égale à la seconde |
| <*compareTo*> | Oui | Respectivement entier, flottant ou chaîne | Valeur de comparaison |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| True ou False | Boolean | Retourne la valeur true si la première valeur est supérieure ou égale à la seconde. Retourne la valeur false si la première valeur est inférieure à la seconde. |
||||

*Exemple*

Ces exemples vérifient si la première valeur est supérieure ou égale à la seconde :

```
greaterOrEquals(5, 5)
greaterOrEquals('apple', 'banana')
```

Et retournent les résultats suivants :

* Premier exemple : `true`
* Second exemple : `false`

<a name="guid"></a>

### <a name="guid"></a>guid

Génère un identificateur global unique (GUID) sous la forme d’une chaîne, par exemple « c2ecc88d-88c8-4096-912c-d6f2e2b138ce » :

```
guid()
```

Vous pouvez également spécifier un format différent pour le GUID autre que le format par défaut, « D », qui correspond à 32 chiffres séparés par des traits d’union.

```
guid('<format>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*format*> | Non | String | Un seul [spécificateur de format](/dotnet/api/system.guid.tostring) pour le GUID retourné. Par défaut, il s’agit du format « D », mais vous pouvez utiliser « N », « D », « B », « P » ou « X ». |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*GUID-value*> | String | GUID généré de manière aléatoire |
||||

*Exemple*

Cet exemple illustre la génération du même GUID, mais sous forme de 32 chiffres séparés par des traits d’union et placés entre parenthèses :

```
guid('P')
```

Et retourne ce résultat : `"(c2ecc88d-88c8-4096-912c-d6f2e2b138ce)"`

<a name="if"></a>

### <a name="if"></a>if

Vérifie si une expression est vraie ou fausse.
En fonction du résultat, retourne une valeur spécifiée.

```
if(<expression>, <valueIfTrue>, <valueIfFalse>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*expression*> | Oui | Boolean | Expression à vérifier |
| <*valueIfTrue*> | Oui | Quelconque | Valeur à retourner lorsque l’expression est vraie |
| <*valueIfFalse*> | Oui | Quelconque | Valeur à retourner lorsque l’expression est fausse |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*specified-return-value*> | Quelconque | Valeur spécifiée retournée selon que l’expression est vraie ou fausse |
||||

*Exemple*

Cet exemple retourne `"yes"`, car l’expression spécifiée retourne la valeur true.
Dans le cas contraire, l’exemple retourne `"no"` :

```
if(equals(1, 1), 'yes', 'no')
```

<a name="indexof"></a>

### <a name="indexof"></a>indexOf

Retourne la position ou la valeur d’index de départ d’une sous-chaîne.
Cette fonction ne respecte pas la casse, et les index commencent par 0.

```
indexOf('<text>', '<searchText>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*text*> | Oui | String | Chaîne comportant la sous-chaîne à rechercher |
| <*searchText*> | Oui | String | Sous-chaîne à rechercher |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*index-value*>| Integer | Position ou valeur d’index de départ de la sous-chaîne spécifiée. <p>Si la chaîne est introuvable, retourne le chiffre -1. |
||||

*Exemple*

Cet exemple recherche la valeur d’index de départ de la sous-chaîne « world » dans la chaîne « hello world » :

```
indexOf('hello world', 'world')
```

Et retourne ce résultat : `6`

<a name="int"></a>

### <a name="int"></a>int

Retourne la version de type entier d’une chaîne.

```
int('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne à convertir |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*integer-result*> | Integer | Version de type entier de la chaîne spécifiée |
||||

*Exemple*

Cet exemple illustre la création d’une version de type entier pour la chaîne « 10 » :

```
int('10')
```

Et retourne ce résultat : `10`

<a name="json"></a>

### <a name="json"></a>json

Retourne la valeur ou l’objet de type JavaScript Object Notation (JSON) d’une chaîne ou d’un élément XML.

```
json('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | Chaîne ou élément XML | Chaîne ou élément XML à convertir |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*JSON-result*> | Objet ou type natif JSON | Objet ou valeur de type natif JSON correspondant à la chaîne ou à l’élément XML spécifiés. Si la chaîne est Null, la fonction retourne un objet vide. |
||||

*Exemple 1*

Cet exemple illustre la conversion de cette chaîne en valeur JSON :

```
json('[1, 2, 3]')
```

Et retourne ce résultat : `[1, 2, 3]`

*Exemple 2*

Cet exemple illustre la conversion de cette chaîne au format JSON :

```
json('{"fullName": "Sophia Owen"}')
```

Et retourne ce résultat :

```
{
  "fullName": "Sophia Owen"
}
```

*Exemple 3*

Cet exemple illustre la conversion de cet élément XML au format JSON :

```
json(xml('<?xml version="1.0"?> <root> <person id='1'> <name>Sophia Owen</name> <occupation>Engineer</occupation> </person> </root>'))
```

Et retourne ce résultat :

```json
{
   "?xml": { "@version": "1.0" },
   "root": {
      "person": [ {
         "@id": "1",
         "name": "Sophia Owen",
         "occupation": "Engineer"
      } ]
   }
}
```

<a name="intersection"></a>

### <a name="intersection"></a>intersection

Retourne une collection qui contient *uniquement* les éléments communs aux collections spécifiées.
Pour qu’il apparaisse dans le résultat, un élément doit apparaître dans toutes les collections transmises à cette fonction.
Si un ou plusieurs éléments portent le même nom, le dernier élément de ce nom apparaît dans le résultat.

```
intersection([<collection1>], [<collection2>], ...)
intersection('<collection1>', '<collection2>', ...)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*collection1*>, <*collection2*>... | Oui | Tableau ou objet, mais pas les deux | Collections à partir desquelles vous souhaitez *uniquement* les éléments communs |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*common-items*> | Tableau ou objet, respectivement | Collection qui contient uniquement les éléments communs aux collections spécifiées |
||||

*Exemple*

Cet exemple recherche les éléments communs dans ces tableaux :

```
intersection(createArray(1, 2, 3), createArray(101, 2, 1, 10), createArray(6, 8, 1, 2))
```

Et retourne un tableau comportant *uniquement* ces éléments : `[1, 2]`

<a name="join"></a>

### <a name="join"></a>join

Retourne une chaîne qui contient tous les éléments d’un tableau, et dont tous les caractères sont séparés par un *séparateur*.

```
join([<collection>], '<delimiter>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*collection*> | Oui | Array | Tableau qui comporte les éléments à joindre |
| <*delimiter*> | Oui | String | Séparateur affiché entre chaque caractère de la chaîne obtenue |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*char1*><*delimiter*><*char2*><*delimiter*>... | String | Chaîne obtenue créée à partir de tous les éléments du tableau spécifié |
||||

*Exemple*

Cet exemple illustre la création d’une chaîne à partir de tous les éléments de ce tableau avec le caractère spécifié comme séparateur :

```
join(createArray('a', 'b', 'c'), '.')
```

Et retourne ce résultat : `"a.b.c"`

<a name="last"></a>

### <a name="last"></a>last

Retourne le dernier élément d’une collection.

```
last('<collection>')
last([<collection>])
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*collection*> | Oui | Chaîne ou tableau | Collection dans laquelle rechercher le dernier élément |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*last-collection-item*> | Chaîne ou tableau, respectivement | Dernier élément de la collection |
||||

*Exemple*

Ces exemples illustrent la recherche du dernier élément dans ces collections :

```
last('abcd')
last(createArray(0, 1, 2, 3))
```

Et retournent les résultats suivants :

* Premier exemple : `"d"`
* Second exemple : `3`

<a name="lastindexof"></a>

### <a name="lastindexof"></a>lastIndexOf

Retourne la position de départ ou la valeur d’index de la dernière occurrence d’une sous-chaîne.
Cette fonction ne respecte pas la casse, et les index commencent par 0.

```
lastIndexOf('<text>', '<searchText>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*text*> | Oui | String | Chaîne comportant la sous-chaîne à rechercher |
| <*searchText*> | Oui | String | Sous-chaîne à rechercher |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*ending-index-value*> | Integer | Retourne la position de départ ou la valeur d’index de la dernière occurrence de la sous-chaîne spécifiée. <p>Si la chaîne est introuvable, retourne le chiffre -1. |
||||

*Exemple*

Cet exemple recherche la valeur d’index de départ de la dernière occurrence de la sous-chaîne « world » dans la chaîne « hello world » :

```
lastIndexOf('hello world', 'world')
```

Et retourne ce résultat : `6`

<a name="length"></a>

### <a name="length"></a>length

Retourne le nombre d’éléments d’une collection.

```
length('<collection>')
length([<collection>])
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*collection*> | Oui | Chaîne ou tableau | Collection contenant les éléments à comptabiliser |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*length-or-count*> | Integer | Nombre d’éléments de la collection |
||||

*Exemple*

Ces exemples illustrent le comptage du nombre d’éléments contenus dans ces collections :

```
length('abcd')
length(createArray(0, 1, 2, 3))
```

Et retourne ce résultat : `4`

<a name="less"></a>

### <a name="less"></a>less

Vérifie si la première valeur est inférieure à la seconde.
Retourne la valeur true si la première valeur est inférieure à la seconde, ou la valeur false si la première valeur est supérieure à la seconde.

```
less(<value>, <compareTo>)
less('<value>', '<compareTo>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | Entier, flottant ou chaîne | Première valeur à vérifier pour déterminer si elle est inférieure à la seconde |
| <*compareTo*> | Oui | Respectivement entier, flottant ou chaîne | Élément de comparaison |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| True ou False | Boolean | Retourne la valeur true si la première valeur est inférieure à la seconde. Retourne la valeur false si la première valeur est égale ou supérieure à la seconde. |
||||

*Exemple*

Ces exemples vérifient si la première valeur est inférieure à la seconde.

```
less(5, 10)
less('banana', 'apple')
```

Et retournent les résultats suivants :

* Premier exemple : `true`
* Second exemple : `false`

<a name="lessOrEquals"></a>

### <a name="lessorequals"></a>lessOrEquals

Vérifie si la première valeur est inférieure ou égale à la seconde.
Retourne la valeur true si la première valeur est inférieure ou égale à la seconde, ou la valeur false si la première valeur est supérieure à la seconde.

```
lessOrEquals(<value>, <compareTo>)
lessOrEquals('<value>', '<compareTo>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | Entier, flottant ou chaîne | Première valeur à vérifier pour déterminer si elle est inférieure ou égale à la seconde |
| <*compareTo*> | Oui | Respectivement entier, flottant ou chaîne | Élément de comparaison |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| True ou False  | Boolean | Retourne la valeur true si la première valeur est inférieure ou égale à la seconde. Retourne la valeur false si la première valeur est supérieure à la seconde. |
||||

*Exemple*

Ces exemples vérifient si la première valeur est inférieure ou égale à la seconde.

```
lessOrEquals(10, 10)
lessOrEquals('apply', 'apple')
```

Et retournent les résultats suivants :

* Premier exemple : `true`
* Second exemple : `false`

<a name="max"></a>

### <a name="max"></a>max

Retourne la valeur la plus élevée d’une liste ou d’un tableau de nombres incluse aux deux extrémités.

```
max(<number1>, <number2>, ...)
max([<number1>, <number2>, ...])
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*number1*>, <*number2*>... | Oui | Entier, flottant ou les deux | Ensemble de nombres dont vous voulez la valeur la plus élevée |
| [<*number1*>, <*number2*>...] | Oui | Tableau : entier, flottant ou les deux | Tableau de nombres dont vous voulez la valeur la plus élevée |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*max-value*> | Entier ou flottant | Valeur la plus élevée dans le tableau ou l’ensemble de nombres spécifié |
||||

*Exemple*

Ces exemples illustrent l’obtention de la valeur la plus élevée de l’ensemble des nombres et du tableau :

```
max(1, 2, 3)
max(createArray(1, 2, 3))
```

Et retourne ce résultat : `3`

<a name="min"></a>

### <a name="min"></a>min

Retourne la plus petite valeur d’un ensemble de nombres ou d’un tableau.

```
min(<number1>, <number2>, ...)
min([<number1>, <number2>, ...])
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*number1*>, <*number2*>... | Oui | Entier, flottant ou les deux | Ensemble de nombres dont vous voulez la valeur la plus petite |
| [<*number1*>, <*number2*>...] | Oui | Tableau : entier, flottant ou les deux | Tableau de nombres dont vous voulez la valeur la plus petite |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*min-value*> | Entier ou flottant | Valeur la plus petite dans le tableau ou l’ensemble de nombres spécifiés |
||||

*Exemple*

Ces exemples illustrent l’obtention de la valeur la plus petite de l’ensemble des nombres et du tableau :

```
min(1, 2, 3)
min(createArray(1, 2, 3))
```

Et retourne ce résultat : `1`

<a name="mod"></a>

### <a name="mod"></a>mod

Retourne le reste de la division de deux nombres.
Pour obtenir le résultat sous forme d’un entier, consultez [div()](#div).

```
mod(<dividend>, <divisor>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*dividend*> | Oui | Entier ou flottant | Nombre à diviser par le *diviseur* |
| <*divisor*> | Oui | Entier ou flottant | Nombre qui divise le *dividende*, mais qui ne peut pas être égal à 0. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*modulo-result*> | Entier ou flottant | Reste de la division du premier nombre par le second nombre |
||||

*Exemple*

Cet exemple illustre la division du premier nombre par le second nombre :

```
mod(3, 2)
```

Et retourne ce résultat : `1`

<a name="mul"></a>

### <a name="mul"></a>mul

Retourne le produit de la multiplication de deux nombres.

```
mul(<multiplicand1>, <multiplicand2>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*multiplicand1*> | Oui | Entier ou flottant | Nombre à multiplier par *multiplicand2* |
| <*multiplicand2*> | Oui | Entier ou flottant | Nombre qui multiplie *multiplicand1* |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*product-result*> | Entier ou flottant | Produit de la multiplication du premier nombre par le second nombre |
||||

*Exemple*

Ces exemples illustrent la multiplication du premier nombre par le second nombre :

```
mul(1, 2)
mul(1.5, 2)
```

Et retournent les résultats suivants :

* Premier exemple : `2`
* Second exemple : `3`

<a name="not"></a>

### <a name="not"></a>not

Vérifie si une expression est fausse.
Retourne la valeur true lorsque l’expression est fausse, ou la valeur false lorsque l’expression est vraie.

```json
not(<expression>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*expression*> | Oui | Boolean | Expression à vérifier |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| True ou False | Boolean | Retourne la valeur true lorsque l’expression est fausse. Retourne la valeur false lorsque l’expression est vraie. |
||||

*Exemple 1*

Ces exemples vérifient si les expressions spécifiées sont fausses :

```json
not(false)
not(true)
```

Et retournent les résultats suivants :

* Premier exemple : comme l’expression est fausse, la fonction retourne `true`.
* Deuxième exemple : comme l’expression est vraie, la fonction retourne `false`.

*Exemple 2*

Ces exemples vérifient si les expressions spécifiées sont fausses :

```json
not(equals(1, 2))
not(equals(1, 1))
```

Et retournent les résultats suivants :

* Premier exemple : comme l’expression est fausse, la fonction retourne `true`.
* Deuxième exemple : comme l’expression est vraie, la fonction retourne `false`.

<a name="or"></a>

### <a name="or"></a>or

Vérifie si au moins une expression est vraie.
Retourne la valeur true si au moins une expression est vraie ou la valeur false si les expressions sont fausses.

```
or(<expression1>, <expression2>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*expression1*>, <*expression2*> | Oui | Boolean | Expressions à vérifier |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| True ou False | Boolean | Retourne la valeur true si au moins une expression est vraie. Retourne la valeur false lorsque les expressions sont fausses. |
||||

*Exemple 1*

Ces exemples vérifient si au moins une expression est vraie :

```json
or(true, false)
or(false, false)
```

Et retournent les résultats suivants :

* Premier exemple : comme au moins une expression est vraie, la fonction retourne `true`.
* Deuxième exemple : comme les deux expressions sont fausses, la fonction retourne `false`.

*Exemple 2*

Ces exemples vérifient si au moins une expression est vraie :

```json
or(equals(1, 1), equals(1, 2))
or(equals(1, 2), equals(1, 3))
```

Et retournent les résultats suivants :

* Premier exemple : comme au moins une expression est vraie, la fonction retourne `true`.
* Deuxième exemple : comme les deux expressions sont fausses, la fonction retourne `false`.

<a name="rand"></a>

### <a name="rand"></a>rand

Retourne un entier aléatoire à partir d’une plage spécifiée, qui est inclus uniquement au point de départ.

```
rand(<minValue>, <maxValue>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*minValue*> | Oui | Integer | Entier le plus petit de la plage |
| <*maxValue*> | Oui | Integer | Entier qui suit l’entier le plus élevé dans la plage que la fonction peut retourner |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*random-result*> | Integer | Entier aléatoire retourné à partir de la plage spécifiée |
||||

*Exemple*

Cet exemple illustre l’obtention d’un entier aléatoire à partir de la plage spécifiée, en excluant la valeur maximale :

```
rand(1, 5)
```

Et retourne l’un de ces nombres comme résultat : `1`, `2`, `3` ou `4`

<a name="range"></a>

### <a name="range"></a>range

Retourne un tableau d’entiers qui commence par un entier spécifique.

```
range(<startIndex>, <count>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*startIndex*> | Oui | Integer | Une valeur entière qui constitue le premier élément du tableau |
| <*count*> | Oui | Integer | Nombre d’entiers du tableau |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| [<*range-result*>] | Array | Tableau dont les entiers commencent à partir de l’index spécifié |
||||

*Exemple*

Cet exemple illustre la création d’un tableau d’entiers qui commence à partir de l’index spécifié et qui possède le nombre spécifié d’entiers :

```
range(1, 4)
```

Et retourne ce résultat : `[1, 2, 3, 4]`

<a name="replace"></a>

### <a name="replace"></a>remplacer

Remplace une sous-chaîne par la chaîne spécifiée et retourne la chaîne de résultat. Cette fonction respecte la casse.

```
replace('<text>', '<oldText>', '<newText>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*text*> | Oui | String | Chaîne comportant la sous-chaîne à remplacer |
| <*oldText*> | Oui | String | Sous-chaîne à remplacer |
| <*newText*> | Oui | String | Chaîne de remplacement |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updated-text*> | String | Chaîne mise à jour après le remplacement de la sous-chaîne <p>Si la sous-chaîne est introuvable, retourne la chaîne d’origine. |
||||

*Exemple*

Cet exemple recherche l’ancienne sous-chaîne dans l’ancienne chaîne, et remplace « old » par « new » :

```
replace('the old string', 'old', 'new')
```

Et retourne ce résultat : `"the new string"`

<a name="skip"></a>

### <a name="skip"></a>skip

Supprime des éléments du début d’une collection et retourne *tous les autres* éléments.

```
skip([<collection>], <count>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*collection*> | Oui | Array | Collection dont vous voulez supprimer des éléments |
| <*count*> | Oui | Integer | Entier positif correspondant au nombre d’éléments à supprimer au début |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| [<*updated-collection*>] | Array | Collection mise à jour à l’issue de la suppression des éléments spécifiés |
||||

*Exemple*

Cet exemple illustre la suppression d’un élément, le nombre 0, au début du tableau spécifié :

```
skip(createArray(0, 1, 2, 3), 1)
```

Et retourne ce tableau avec les éléments restants : `[1,2,3]`

<a name="split"></a>

### <a name="split"></a>split

Retourne un tableau qui contient des sous-chaînes, séparées par des virgules, extraites sur la base du caractère délimiteur spécifié dans la chaîne d’origine.

```
split('<text>', '<delimiter>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*text*> | Oui | String | La chaîne pour séparer les sous-chaînes en fonction du délimiteur spécifié dans la chaîne d’origine |
| <*delimiter*> | Oui | String | Le caractère dans la chaîne d’origine à utiliser comme délimiteur |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| [<*substring1*>,<*substring2*>,...] | Array | Un tableau qui contient les sous-chaînes extraites de la chaîne d’origine, séparées par des virgules |
||||

*Exemple*

Cet exemple crée un tableau avec des sous-chaînes extraites de la chaîne spécifiée selon le caractère spécifié comme délimiteur :

```
split('a_b_c', '_')
```

Et retourne ce tableau en tant que résultat : `["a","b","c"]`

<a name="startOfDay"></a>

### <a name="startofday"></a>startOfDay

Retourne le début du jour d’un horodatage.

```
startOfDay('<timestamp>', '<format>'?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updated-timestamp*> | String | Horodatage spécifié, mais commençant à la marque zéro-heure du jour |
||||

*Exemple*

Cet exemple recherche le début du jour pour cet horodatage :

```
startOfDay('2018-03-15T13:30:30Z')
```

Et retourne ce résultat : `"2018-03-15T00:00:00.0000000Z"`

<a name="startOfHour"></a>

### <a name="startofhour"></a>startOfHour

Retourne le début de l’heure d’un horodatage.

```
startOfHour('<timestamp>', '<format>'?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updated-timestamp*> | String | Horodatage spécifié, mais commençant à la marque zéro-minute de l’heure |
||||

*Exemple*

Cet exemple recherche le début de l’heure pour cet horodatage :

```
startOfHour('2018-03-15T13:30:30Z')
```

Et retourne ce résultat : `"2018-03-15T13:00:00.0000000Z"`

<a name="startOfMonth"></a>

### <a name="startofmonth"></a>startOfMonth

Retourne le début du mois pour un horodatage.

```
startOfMonth('<timestamp>', '<format>'?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updated-timestamp*> | String | Horodatage spécifié, mais démarrant le premier jour du mois à la marque zéro-heure |
||||

*Exemple*

Cet exemple retourne le début du mois pour cet horodatage :

```
startOfMonth('2018-03-15T13:30:30Z')
```

Et retourne ce résultat : `"2018-03-01T00:00:00.0000000Z"`

<a name="startswith"></a>

### <a name="startswith"></a>startsWith

Vérifie si une chaîne commence par une sous-chaîne spécifique.
Retourne la valeur true lorsque la sous-chaîne est trouvée ou la valeur false lorsqu’elle est introuvable.
Cette fonction ne respecte pas la casse.

```
startsWith('<text>', '<searchText>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*text*> | Oui | String | Chaîne à vérifier |
| <*searchText*> | Oui | String | Chaîne de départ à rechercher |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| True ou False  | Boolean | Retourne la valeur true lorsque la sous-chaîne de début est trouvée. Retourne la valeur false lorsqu’elle est introuvable. |
||||

*Exemple 1*

Cet exemple vérifie si la chaîne « hello world » commence par la sous-chaîne « hello » :

```
startsWith('hello world', 'hello')
```

Et retourne ce résultat : `true`

*Exemple 2*

Cet exemple vérifie si la chaîne « hello world » commence par la sous-chaîne « greetings » :

```
startsWith('hello world', 'greetings')
```

Et retourne ce résultat : `false`

<a name="string"></a>

### <a name="string"></a>string

Retourne la version de type chaîne d’une valeur.

```
string(<value>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | Quelconque | Valeur à convertir |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*string-value*> | String | Version de type chaîne de la valeur spécifiée |
||||

*Exemple 1*

Cet exemple illustre la création de la version de type chaîne pour ce nombre :

```
string(10)
```

Et retourne ce résultat : `"10"`

*Exemple 2*

Cet exemple illustre la création d’une chaîne pour l’objet JSON spécifié et utilise le caractère de barre oblique inverse (\\) comme caractère d’échappement pour le guillemet double (").

```
string( { "name": "Sophie Owen" } )
```

Et retourne ce résultat : `"{ \\"name\\": \\"Sophie Owen\\" }"`

<a name="sub"></a>

### <a name="sub"></a>sub

Retourne le résultat de la soustraction du second nombre du premier.

```
sub(<minuend>, <subtrahend>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*minuend*> | Oui | Entier ou flottant | Nombre à partir duquel soustraire le *plus petit terme* |
| <*subtrahend*> | Oui | Entier ou flottant | Nombre à soustraire du *plus petit terme* |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*result*> | Entier ou flottant | Résultat de la soustraction du second nombre du premier |
||||

*Exemple*

Cet exemple illustre la soustraction du second nombre du premier :

```
sub(10.3, .3)
```

Et retourne ce résultat : `10`

<a name="substring"></a>

### <a name="substring"></a>substring

Retourne les caractères d’une chaîne, en commençant à partir de la position spécifiée ou de l’index.
Les valeurs d’index commencent par le chiffre 0.

```
substring('<text>', <startIndex>, <length>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*text*> | Oui | String | Chaîne dont vous souhaitez les caractères |
| <*startIndex*> | Oui | Integer | Une valeur positive supérieure ou égale à 0 que vous souhaitez utiliser comme la valeur de position ou l’index de départ |
| <*length*> | Oui | Integer | Nombre positif de caractères que vous souhaitez dans la sous-chaîne |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*substring-result*> | String | Sous-chaîne contenant le nombre spécifié de caractères, en commençant à la position d’index spécifiée dans la chaîne source |
||||

*Exemple*

Cet exemple illustre la création d’une sous-chaîne de cinq caractères à partir de la chaîne spécifiée, en commençant à partir de la valeur d’index 6 :

```
substring('hello world', 6, 5)
```

Et retourne ce résultat : `"world"`

<a name="subtractFromTime"></a>

### <a name="subtractfromtime"></a>subtractFromTime

Soustrait un nombre d’unités de temps d’un horodatage.
Voir aussi [getPastTime](#getPastTime).

```
subtractFromTime('<timestamp>', <interval>, '<timeUnit>', '<format>'?)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne qui contient l’horodatage |
| <*interval*> | Oui | Integer | Nombre d’unités de temps spécifiées à soustraire |
| <*timeUnit*> | Oui | String | L’unité de temps à utiliser avec *interval* : "Second", "Minute", "Hour", "Day", "Week", "Month", "Year" |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updated-timestamp*> | String | Horodatage moins le nombre d’unités de temps spécifié |
||||

*Exemple 1*

Cet exemple soustrait un jour de cet horodatage :

```
subtractFromTime('2018-01-02T00:00:00Z', 1, 'Day')
```

Et retourne ce résultat : `"2018-01-01T00:00:00:0000000Z"`

*Exemple 2*

Cet exemple soustrait un jour de cet horodatage :

```
subtractFromTime('2018-01-02T00:00:00Z', 1, 'Day', 'D')
```

Et retourne ce résultat en utilisant le format « D » facultatif : `"Monday, January, 1, 2018"`

<a name="take"></a>

### <a name="take"></a>take

Retourne des éléments du début d’une collection.

```
take('<collection>', <count>)
take([<collection>], <count>)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*collection*> | Oui | Chaîne ou tableau | Collection dont vous souhaitez des éléments |
| <*count*> | Oui | Integer | Entier positif correspondant au nombre d’éléments que vous souhaitez à partir du début |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*subset*> ou [<*subset*>] | Chaîne ou tableau, respectivement | Chaîne ou tableau qui comporte le nombre spécifié d’éléments obtenus à partir du début de la collection d’origine |
||||

*Exemple*

Ces exemples illustrent l’obtention du nombre spécifié d’éléments à partir du début de ces collections :

```
take('abcde', 3)
take(createArray(0, 1, 2, 3, 4), 3)
```

Et retournent les résultats suivants :

* Premier exemple : `"abc"`
* Second exemple : `[0, 1, 2]`

<a name="ticks"></a>

### <a name="ticks"></a>ticks

Retourne la valeur de la propriété `ticks` pour un horodatage spécifique.
Une *graduation* est un intervalle de 100 nanosecondes.

```
ticks('<timestamp>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*timestamp*> | Oui | String | Chaîne d’un horodatage |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*ticks-number*> | Integer | Nombre de graduations depuis l’horodatage spécifié |
||||

<a name="toLower"></a>

### <a name="tolower"></a>toLower

Retourne une chaîne en minuscules. Si un caractère de la chaîne n’a pas de version en minuscules, ce caractère reste tel quel dans la chaîne retournée.

```
toLower('<text>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*text*> | Oui | String | Chaîne à retourner en minuscules |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*lowercase-text*> | String | Chaîne d’origine en minuscules |
||||

*Exemple*

Cet exemple illustre la conversion de cette chaîne en minuscules :

```
toLower('Hello World')
```

Et retourne ce résultat : `"hello world"`

<a name="toUpper"></a>

### <a name="toupper"></a>toUpper

Retourne une chaîne en majuscules. Si un caractère de la chaîne n’a pas de version en majuscules, ce caractère reste tel quel dans la chaîne retournée.

```
toUpper('<text>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*text*> | Oui | String | Chaîne à retourner en majuscules |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*uppercase-text*> | String | Chaîne d’origine en majuscules |
||||

*Exemple*

Cet exemple illustre la conversion de cette chaîne en majuscules :

```
toUpper('Hello World')
```

Et retourne ce résultat : `"HELLO WORLD"`

<a name="trim"></a>

### <a name="trim"></a>trim

Supprime les espaces blancs de début et de fin d’une chaîne et retourne la chaîne mise à jour.

```
trim('<text>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*text*> | Oui | String | Chaîne contenant les espaces blancs de début et de fin à supprimer |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updatedText*> | String | Version mise à jour de la chaîne d’origine sans espace blanc de début ou de fin |
||||

*Exemple*

Cet exemple illustre la suppression des espaces blancs de début et de fin de la chaîne « Hello World » :

```
trim(' Hello World  ')
```

Et retourne ce résultat : `"Hello World"`

<a name="union"></a>

### <a name="union"></a>union

Retourne une collection qui contient *tous* les éléments des collections spécifiées.
Pour qu’il apparaisse dans le résultat, un élément peut apparaître dans n’importe quelle collection transmise à cette fonction. Si un ou plusieurs éléments portent le même nom, le dernier élément de ce nom apparaît dans le résultat.

```
union('<collection1>', '<collection2>', ...)
union([<collection1>], [<collection2>], ...)
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*collection1*>, <*collection2*>...  | Oui | Tableau ou objet, mais pas les deux | Collections dont vous souhaitez *tous* les éléments |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*updatedCollection*> | Tableau ou objet, respectivement | Collection qui contient tous les éléments des collections spécifiées (pas de doublons) |
||||

*Exemple*

Cet exemple illustre l’obtention de *tous* les éléments de ces collections :

```
union(createArray(1, 2, 3), createArray(1, 2, 10, 101))
```

Et retourne ce résultat : `[1, 2, 3, 10, 101]`

<a name="uriComponent"></a>

### <a name="uricomponent"></a>uriComponent

Retourne une version encodée sous forme d’URI d’une chaîne en remplaçant les caractères non sécurisés pour les URL par des caractères d’échappement.
Utilisez cette fonction plutôt que la fonction [encodeUriComponent()](#encodeUriComponent).
Bien que les deux fonctions agissent de manière identique, la fonction `uriComponent()` est préférée.

```
uriComponent('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne à convertir au format encodé sous forme d’URI |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*encoded-uri*> | String | Chaîne encodée sous forme d’URI avec des caractères d’échappement |
||||

*Exemple*

Cet exemple illustre la création d’une version encodée sous forme d’URI pour cette chaîne :

```
uriComponent('https://contoso.com')
```

Et retourne ce résultat : `"http%3A%2F%2Fcontoso.com"`

<a name="uriComponentToBinary"></a>

### <a name="uricomponenttobinary"></a>uriComponentToBinary

Retourne la version binaire d’un composant d’URI.

```
uriComponentToBinary('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne encodée sous forme d’URI à convertir |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*binary-for-encoded-uri*> | String | Version binaire de la chaîne encodée sous forme d’URI. Le contenu binaire est encodé en Base64 et représenté par `$content`. |
||||

*Exemple*

Cet exemple illustre la création de la version binaire de cette chaîne encodée sous forme d’URI :

```
uriComponentToBinary('http%3A%2F%2Fcontoso.com')
```

Et retourne ce résultat :

`"001000100110100001110100011101000111000000100101001100
11010000010010010100110010010001100010010100110010010001
10011000110110111101101110011101000110111101110011011011
110010111001100011011011110110110100100010"`

<a name="uriComponentToString"></a>

### <a name="uricomponenttostring"></a>uriComponentToString

Retourne la version de type chaîne d’une chaîne encodée sous forme d’URI, en décodant efficacement la chaîne encodée sous forme d’URI.

```
uriComponentToString('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne encodée sous forme d’URI à décoder |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*decoded-uri*> | String | Version décodée de la chaîne encodée sous forme d’URI |
||||

*Exemple*

Cet exemple illustre la création de la version de type chaîne décodée pour cette chaîne encodée sous forme d’URI :

```
uriComponentToString('http%3A%2F%2Fcontoso.com')
```

Et retourne ce résultat : `"https://contoso.com"`

<a name="utcNow"></a>

### <a name="utcnow"></a>utcNow

Retourne l’horodatage actuel.

```
utcNow('<format>')
```

Si vous le souhaitez, vous pouvez spécifier un autre format avec le paramètre <*format*>.

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*format*> | Non | String | [Spécificateur de format unique](/dotnet/standard/base-types/standard-date-and-time-format-strings) ou [modèle de format personnalisé](/dotnet/standard/base-types/custom-date-and-time-format-strings). Le format par défaut de l’horodatage est [« o »](/dotnet/standard/base-types/standard-date-and-time-format-strings) (aaaa-MM-jjT:mm:ss:fffffffK), qui est conforme à la norme [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) et conserve les informations de fuseau horaire. |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*current-timestamp*> | String | Date et heure actuelles |
||||

*Exemple 1*

Supposons que nous soyons le 15 avril 2018 à 13:00:00.
Cet exemple illustre l’obtention de l’horodatage actuel :

```
utcNow()
```

Et retourne ce résultat : `"2018-04-15T13:00:00.0000000Z"`

*Exemple 2*

Supposons que nous soyons le 15 avril 2018 à 13:00:00.
Cet exemple illustre l’obtention de l’horodatage actuel en utilisant le format « D » facultatif :

```
utcNow('D')
```

Et retourne ce résultat : `"Sunday, April 15, 2018"`

<a name="xml"></a>

### <a name="xml"></a>Xml

Retourne la version XML d’une chaîne qui contient un objet JSON.

```
xml('<value>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*value*> | Oui | String | Chaîne contenant l’objet JSON à convertir <p>L’objet JSON ne doit avoir qu’une seule propriété racine, qui ne peut pas être un tableau. <br>Utilisez le caractère de barre oblique inverse (\\) comme caractère d’échappement pour le guillemet double ("). |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*xml-version*> | Object | Élément XML encodé correspondant à la chaîne ou à l’objet JSON spécifié |
||||

*Exemple 1*

Cet exemple illustre la création de la version XML de cette chaîne, qui contient un objet JSON :

`xml(json('{ \"name\": \"Sophia Owen\" }'))`

Et retourne le résultat XML suivant :

```xml
<name>Sophia Owen</name>
```

*Exemple 2*

Supposons que vous disposiez de cet objet JSON :

```json
{
  "person": {
    "name": "Sophia Owen",
    "city": "Seattle"
  }
}
```

Cet exemple illustre la création d’un élément XML pour une chaîne qui contient l’objet JSON :

`xml(json('{\"person\": {\"name\": \"Sophia Owen\", \"city\": \"Seattle\"}}'))`

Et retourne le résultat XML suivant :

```xml
<person>
  <name>Sophia Owen</name>
  <city>Seattle</city>
<person>
```

<a name="xpath"></a>

### <a name="xpath"></a>xpath

Vérifie si le code XML contient des valeurs ou des nœuds qui correspondent à une expression de langage XPath et retourne les valeurs ou les nœuds correspondants. Une expression XPath, ou simplement « XPath », vous permet de parcourir une structure de document XML afin de pouvoir sélectionner des nœuds ou des valeurs de calcul dans le contenu XML.

```
xpath('<xml>', '<xpath>')
```

| Paramètre | Obligatoire | Type | Description |
| --------- | -------- | ---- | ----------- |
| <*xml*> | Oui | Quelconque | Chaîne XML pour rechercher des nœuds ou des valeurs qui correspondent à une valeur d’expression XPath |
| <*xpath*> | Oui | Quelconque | Expression XPath utilisée pour rechercher des nœuds ou des valeurs XML correspondants |
|||||

| Valeur retournée | Type | Description |
| ------------ | ---- | ----------- |
| <*xml-node*> | XML | Nœud XML lorsqu’un seul nœud correspond à l’expression XPath spécifiée |
| <*value*> | Quelconque | Valeur d’un nœud XML lorsqu’un seul nœud correspond à l’expression XPath spécifiée |
| [<*xml-node1*>, <*xml-node2*>...] </br>-ou- </br>[<*value1*>, <*value2*>...] | Array | Tableau contenant des nœuds ou valeurs XML qui correspondent à l’expression XPath spécifiée |
||||

*Exemple 1*

À la suite de l’exemple 1, cet exemple recherche les nœuds qui correspondent au nœud `<count></count>` et ajoute ces valeurs de nœud avec la fonction `sum()` :

`xpath(xml(parameters('items')), 'sum(/produce/item/count)')`

Et retourne ce résultat : `30`

*Exemple 2*

Pour cet exemple, les deux expressions recherchent des nœuds qui correspondent au nœud `<location></location>`, dans les arguments spécifiés, notamment XML avec un espace de noms. Les expressions utilisent le caractère de barre oblique inverse (\\) comme caractère d’échappement pour le guillemet double (").

* *Expression 1*

  `xpath(xml(body('Http')), '/*[name()=\"file\"]/*[name()=\"location\"]')`

* *Expression 2*

  `xpath(xml(body('Http')), '/*[local-name()=\"file\" and namespace-uri()=\"http://contoso.com\"]/*[local-name()=\"location\"]')`

Voici les arguments :

* Ce code XML, qui inclut l’espace de noms du document XML, `xmlns="http://contoso.com"` :

  ```xml
  <?xml version="1.0"?> <file xmlns="http://contoso.com"> <location>Paris</location> </file>
  ```

* L’une ou l’autre des expressions XPath ici :

  * `/*[name()=\"file\"]/*[name()=\"location\"]`

  * `/*[local-name()=\"file\" and namespace-uri()=\"http://contoso.com\"]/*[local-name()=\"location\"]`

Voici le nœud obtenu qui correspond au nœud `<location></location>` :

```xml
<location xmlns="https://contoso.com">Paris</location>
```

*Exemple 3*

À la suite de l’exemple 3, cet exemple recherche la valeur du nœud `<location></location>` :

`xpath(xml(body('Http')), 'string(/*[name()=\"file\"]/*[name()=\"location\"])')`

Et retourne ce résultat : `"Paris"`

## <a name="next-steps"></a>Étapes suivantes
Pour la liste des variables système que vous pouvez utiliser dans des expressions, voir [variables système](control-flow-system-variables.md).