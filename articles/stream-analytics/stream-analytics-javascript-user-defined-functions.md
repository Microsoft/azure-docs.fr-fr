---
title: Fonctions JavaScript définies par l’utilisateur Azure Stream Analytics
description: Cet article est une introduction aux fonctions JavaScript définies par l’utilisateur dans Stream Analytics.
author: rodrigoaatmicrosoft
ms.author: rodrigoa
ms.service: stream-analytics
ms.topic: tutorial
ms.reviewer: mamccrea
ms.custom: mvc, devx-track-js
ms.date: 12/15/2020
ms.openlocfilehash: 085ac8c2ca7cfafcf0e40152458acf68dd847937
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97605528"
---
# <a name="javascript-user-defined-functions-in-azure-stream-analytics"></a>Fonctions JavaScript définies par l’utilisateur dans Azure Stream Analytics
 
Azure Stream Analytics prend en charge les fonctions définies par l’utilisateur écrites en JavaScript. Avec le large éventail de méthodes **String**, **RegExp**, **Math**, **Array** et **Date** que fournit JavaScript, les transformations complexes de données des travaux Stream Analytics sont plus faciles à créer.

## <a name="overview"></a>Vue d’ensemble

Les fonctions JavaScript définies par l’utilisateur prennent en charge les fonctions scalaires de calcul pur sans état qui ne nécessitent pas de connectivité externe. La valeur renvoyée par une fonction ne peut être qu’une valeur (unique) scalaire. Une fois que vous avez ajouté une fonction JavaScript définie par l’utilisateur à un travail, vous pouvez utiliser la fonction n’importe où dans la requête, comme une fonction scalaire intégrée.

Voici quelques scénarios dans lesquels les fonctions JavaScript définies par l’utilisateur vous seront utiles :
* Analyse et manipulation de chaînes avec des fonctions d’expressions régulières, par exemple **Regexp_Replace()** et **Regexp_Extract()**
* Décodage de données encodées, par exemple la conversion de binaire en hexadécimal
* Calculs mathématiques avec les fonctions **Math** de JavaScript
* Opérations de tableaux comme le tri, la jointure, la recherche et le remplissage

Voici quelques actions que vous ne pouvez pas effectuer avec une fonction JavaScript définie par l’utilisateur dans Stream Analytics :
* Appel de points de terminaison REST externes, par exemple la recherche inversée d’une adresse IP ou l’extraction de données de référence à partir d’une source externe
* Sérialisation ou désérialisation à un format d’événement personnalisé sur les entrées/sorties
* Création d’agrégats personnalisés

Bien qu’elles ne soient pas bloquées dans la définition des fonctions, évitez d’utiliser des fonctions telles que **Date.GetDate()** ou **Math.random()** . Ces fonctions ne retournent **pas** le même résultat à chaque fois que vous les appelez et le service Azure Stream Analytics ne conserve pas de journal des appels de fonction et des résultats retournés. Si une fonction retourne un résultat différent sur les mêmes événements, la reproductibilité n’est pas garantie quand un travail est relancé par vos soins ou par le service Stream Analytics.

## <a name="add-a-javascript-user-defined-function-to-your-job"></a>Ajouter une fonction JavaScript définie par l’utilisateur à votre travail

> [!NOTE]
> Ces étapes fonctionnent sur les travaux Stream Analytics configurés pour s’exécuter dans le cloud. Si votre travail Stream Analytics est configuré pour s’exécuter sur Azure IoT Edge, utilisez plutôt Visual Studio et [écrivez la fonction définie par l’utilisateur en C#](stream-analytics-edge-csharp-udf.md).

Pour créer une fonction JavaScript définie par l’utilisateur dans votre travail Stream Analytics, sélectionnez **Fonctions** sous **Topologie de la tâche**. Ensuite, sélectionnez **UDF Javascript** dans le menu déroulant **+Ajouter**. 

![Ajouter une fonction JavaScript définie par l’utilisateur](./media/javascript/stream-analytics-jsudf-add.png)

Vous devez ensuite fournir les propriétés suivantes et sélectionner **Enregistrer**.

|Propriété|Description|
|--------|-----------|
|Alias de fonction|Entrez un nom pour appeler la fonction dans votre requête.|
|Type de sortie|Type qui est retourné par la fonction JavaScript définie par l’utilisateur à votre requête Stream Analytics.|
|Définition de fonction|Implémentation de votre fonction JavaScript qui est exécutée chaque fois que votre fonction définie par l’utilisateur est appelée à partir de votre requête.|

## <a name="test-and-troubleshoot-javascript-udfs"></a>Tester les fonctions JavaScript définies par l’utilisateur et résoudre les problèmes les concernant 

Vous pouvez tester et déboguer votre logique de fonctions UDF JavaScript dans n’importe quel navigateur. Le débogage et le test de la logique de ces fonctions définies par l’utilisateur ne sont actuellement pas pris en charge dans le portail Stream Analytics. Une fois que la fonction fonctionne comme prévu, vous pouvez l’ajouter au travail Stream Analytics comme indiqué ci-dessus, puis l’appeler directement à partir de votre requête. Vous pouvez tester la logique de votre requête avec la fonction JavaScript définie par l’utilisateur à l’aide des [outils Stream Analytics pour Visual Studio](./stream-analytics-tools-for-visual-studio-install.md).

Les erreurs d’exécution JavaScript sont considérées comme irrécupérables et remontées par le biais du journal d’activité. Pour récupérer le journal, dans le portail Azure, accédez à votre travail et sélectionnez **Journal d’activité**.

## <a name="call-a-javascript-user-defined-function-in-a-query"></a>Appeler une fonction JavaScript définie par l’utilisateur dans une requête

Vous pouvez facilement appeler votre fonction JavaScript dans votre requête à l’aide de l’alias de fonction préfixé avec **udf**. Voici l’exemple d’une fonction JavaScript définie par l’utilisateur, qui convertit des valeurs hexadécimales en entier, appelée dans une requête Stream Analytics.

```SQL
    SELECT
        time,
        UDF.hex2Int(offset) AS IntOffset
    INTO
        output
    FROM
        InputStream
```

## <a name="supported-javascript-objects"></a>Objets JavaScript pris en charge

Les fonctions JavaScript définies par l’utilisateur Azure Stream Analytics prennent en charge les objets prédéfinis standard du langage JavaScript. Pour obtenir la liste de ces objets, consultez [Objets globaux](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects).

### <a name="stream-analytics-and-javascript-type-conversion"></a>Conversion de type Stream Analytics et JavaScript

Il existe des différences entre les types pris en charge dans le langage de requête Stream Analytics et JavaScript. Le tableau suivant répertorie les mappages de conversion entre les deux :

Stream Analytics | JavaScript
--- | ---
bigint | Number (JavaScript ne peut représenter les entiers que jusqu’à 2^53 précisément)
DateTime | Date (JavaScript ne prend en charge que les millisecondes)
double | Number
nvarchar(MAX) | String
Enregistrement | Object
Array | Array
NULL | Null

Voici les conversions de JavaScript à Stream Analytics :

JavaScript | Stream Analytics
--- | ---
Number | Bigint (si le nombre est rond et entre long.MinValue et long.MaxValue, sinon, double)
Date | DateTime
String | nvarchar(MAX)
Object | Enregistrement
Array | Array
Null, Undefined | NULL
Un autre type (par exemple une fonction ou une erreur) | Non pris en charge (entraîne une erreur d’exécution)

Le langage JavaScript respecte la casse, et la casse des champs d’objet dans le code JavaScript doit correspondre à la casse des champs dans les données entrantes. Les tâches avec le niveau de compatibilité 1.0 convertissent les champs de l’instruction SQL SELECT en minuscules. À partir de la compatibilité 1.1 et dans les versions ultérieures, les champs de l’instruction SELECT auront la même casse que celle spécifiée dans la requête SQL.

## <a name="other-javascript-user-defined-function-patterns"></a>Autres modèles de fonctions JavaScript définies par l’utilisateur

### <a name="write-nested-json-to-output"></a>Écrire du code JSON imbriqué en sortie

Si vous avez une étape de traitement de suivi qui prend la sortie du travail Stream Analytics en entrée et qu’un format JSON est requis, vous pouvez écrire une chaîne JSON comme sortie. L’exemple suivant appelle la fonction **JSON.stringify()** pour compresser toutes les paires nom/valeur de l’entrée, puis les écrire sous la forme d’une valeur de chaîne unique en sortie.

**Définition de la fonction JavaScript définie par l’utilisateur :**

```javascript
function main(x) {
return JSON.stringify(x);
}
```

**Exemple de requête :**
```SQL
SELECT
    DataString,
    DataValue,
    HexValue,
    UDF.jsonstringify(input) As InputEvent
INTO
    output
FROM
    input PARTITION BY PARTITIONID
```

### <a name="cast-string-to-json-object-to-process"></a>Convertir une chaîne en objet JSON à traiter

Si vous disposez d’un champ de chaîne JSON que vous souhaitez convertir en objet JSON à des fins de traitement dans une fonction JavaScript définie par l’utilisateur, vous pouvez utiliser la fonction **JSON.parse()** pour créer un objet JSON à utiliser ensuite.

**Définition de la fonction JavaScript définie par l’utilisateur :**

```javascript
function main(x) {
var person = JSON.parse(x);  
return person.name;
}
```

**Exemple de requête :**
```SQL
SELECT
    UDF.getName(input) AS Name
INTO
    output
FROM
    input
```

### <a name="use-trycatch-for-error-handling"></a>Utiliser try/catch pour la gestion des erreurs

Les blocs try/catch vous aident à identifier les problèmes liés aux données d’entrée incorrectes transmises dans une fonction JavaScript définie par l’utilisateur.

**Définition de la fonction JavaScript définie par l’utilisateur :**

```javascript
function main(input, x) {
    var obj = null;

    try{
        obj = JSON.parse(x);
    }catch(error){
        throw input;
    }
    
    return obj.Value;
}
```

**Exemple de requête : Transmettez l’intégralité de l’enregistrement en tant que premier paramètre de manière à ce qu’il soit renvoyé en cas d’erreur.**
```SQL
SELECT
    A.context.company AS Company,
    udf.getValue(A, A.context.value) as Value
INTO
    output
FROM
    input A
```

### <a name="tolocalestring"></a>toLocaleString()
La méthode **toLocaleString** en JavaScript peut être utilisée pour retourner une chaîne sensible au langage qui représente les données de date et d’heure d’où cette méthode est appelée.
Bien qu’Azure Stream Analtyics accepte uniquement l’heure UTC comme horodatage système, cette méthode peut être utilisée pour convertir l’horodatage système en un autre paramètre régional et fuseau horaire.
Cette méthode suit le même comportement d’implémentation que celui disponible dans Internet Explorer.

**Définition de la fonction JavaScript définie par l’utilisateur :**

```javascript
function main(datetime){
    const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
    return event.toLocaleDateString('de-DE', options);
}
```

**Exemple de requête : Passer une valeur DateTime comme valeur d’entrée**
```SQL
SELECT
    udf.toLocaleString(input.datetime) as localeString
INTO
    output
FROM
    input
```

La sortie de cette requête sera la valeur DateTime d’entrée dans **de-DE** avec les options fournies.
```
Samstag, 28. Dezember 2019
```

## <a name="user-logging"></a>Journalisation utilisateur
Le mécanisme de journalisation vous permet de capturer des informations personnalisées pendant l’exécution d’un travail. Vous pouvez utiliser les données de journal pour déboguer ou évaluer l’exactitude du code personnalisé en temps réel. Ce mécanisme est disponible via la méthode Console.Log().

```javascript
console.log('my error message');
```

Vous pouvez accéder aux messages du journal par le biais des [journaux de diagnostic](data-errors.md).
## <a name="next-steps"></a>Étapes suivantes

* [Fonction définie par l’utilisateur pour le machine learning](./machine-learning-udf.md)
* [C# UDF](./stream-analytics-edge-csharp-udf-methods.md)
