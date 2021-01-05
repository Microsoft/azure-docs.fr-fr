---
title: Compétence cognitive conditionnelle
titleSuffix: Azure Cognitive Search
description: La compétence conditionnelle dans la Recherche cognitive Azure permet de filtrer, de créer des valeurs par défaut et de fusionner des valeurs dans une définition d’ensemble de compétences.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: b5f1fc7f877854dd06fbbe09ff82e47208fa12d0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "72792043"
---
# <a name="conditional-cognitive-skill"></a>Compétence cognitive conditionnelle

La compétence **conditionnelle** permet des scénarios de Recherche cognitive Azure qui nécessitent une opération booléenne pour déterminer les données à affecter à une sortie. Ces scénarios incluent le filtrage, l’attribution d’une valeur par défaut et la fusion de données selon une condition.

Le pseudo-code suivant illustre ce que réalise la compétence conditionnelle :

```
if (condition) 
    { output = whenTrue } 
else 
    { output = whenFalse } 
```

> [!NOTE]
> Cette compétence n’est pas liée à une API Azure Cognitive Services et son utilisation ne vous est pas facturée. Toutefois, vous devez toujours [joindre une ressource Cognitive Services](cognitive-search-attach-cognitive-services.md) pour remplacer l’option de ressource « Gratuit » qui vous limite à un petit nombre d’enrichissements quotidiens.

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Util.ConditionalSkill


## <a name="evaluated-fields"></a>Champs évalués

Cette compétence est spéciale, car ses entrées sont des champs évalués.

Les éléments suivants sont des valeurs valides d’une expression :

-   Chemins d’accès d’annotation (les chemins d’accès dans les expressions doivent être délimités par "$(" et ")").
 <br/>
    Exemples :
    ```
        "= $(/document)"
        "= $(/document/content)"
    ```

-  Littéraux (chaînes, nombres, true, false, null) <br/>
    Exemples :
    ```
       "= 'this is a string'"   // string (note the single quotation marks)
       "= 34"                   // number
       "= true"                 // Boolean
       "= null"                 // null value
    ```

-  Expressions qui utilisent des opérateurs de comparaison (==, !=, >=, >, <=, <) <br/>
    Exemples :
    ```
        "= $(/document/language) == 'en'"
        "= $(/document/sentiment) >= 0.5"
    ```

-   Expressions qui utilisent des opérateurs booléens (&&, ||, !, ^) <br/>
    Exemples :
    ```
        "= $(/document/language) == 'en' && $(/document/sentiment) > 0.5"
        "= !true"
    ```

-   Expressions qui utilisent des opérateurs numériques (+, -, \*, /, %) <br/>
    Exemples : 
    ```
        "= $(/document/sentiment) + 0.5"         // addition
        "= $(/document/totalValue) * 1.10"       // multiplication
        "= $(/document/lengthInMeters) / 0.3049" // division
    ```

Étant donné que la compétence conditionnelle prend en charge l’évaluation, vous pouvez l’utiliser dans les scénarios de transformation mineure. Pour obtenir un exemple, consultez la section [Définition de compétence 4](#transformation-example).

## <a name="skill-inputs"></a>Entrées de la compétence
Les entrées respectent la casse.

| Entrée   | Description |
|-------------|-------------|
| condition   | Cette entrée est un [champ évalué](#evaluated-fields) qui représente la condition à évaluer. Cette condition doit être évaluée selon une valeur booléenne (*true* ou *false*).   <br/>  Exemples : <br/> "= true" <br/> "= $(/document/language) =='fr'" <br/> "= $(/document/pages/\*/language) == $(/document/expectedLanguage)" <br/> |
| whenTrue    | Cette entrée est un [champ évalué](#evaluated-fields) qui représente la valeur à renvoyer si la condition est évaluée sur *true*. Les chaînes constantes doivent être renvoyées entre guillemets simples (' et '). <br/>Exemples de valeurs : <br/> "= 'contract'"<br/>"= $(/document/contractType)" <br/> "= $(/document/entities/\*)" <br/> |
| whenFalse   | Cette entrée est un [champ évalué](#evaluated-fields) qui représente la valeur à renvoyer si la condition est évaluée sur *false*. <br/>Exemples de valeurs : <br/> "= 'contract'"<br/>"= $(/document/contractType)" <br/> "= $(/document/entities/\*)" <br/>

## <a name="skill-outputs"></a>Sorties de la compétence
Il existe une seule sortie appelée simplement "output." Elle renvoie la valeur *whenFalse* si la condition est false ou *whenTrue* si la condition est true.

## <a name="examples"></a>Exemples

### <a name="sample-skill-definition-1-filter-documents-to-return-only-french-documents"></a>Exemple de définition de compétence 1 : Filtrer des documents pour renvoyer uniquement les documents français

La sortie suivante renvoie un tableau de phrases ("/document/frenchSentences") si la langue du document est le français. Si la langue n’est pas le français, la valeur est définie sur *null*.

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document",
    "inputs": [
        { "name": "condition", "source": "= $(/document/language) == 'fr'" },
        { "name": "whenTrue", "source": "/document/sentences" },
        { "name": "whenFalse", "source": "= null" }
    ],
    "outputs": [ { "name": "output", "targetName": "frenchSentences" } ]
}
```
Si "/document/frenchSentences" est utilisé comme *contexte* d’une autre compétence, cette compétence s’exécute uniquement si "/document/frenchSentences" n’est pas défini sur *null*.


### <a name="sample-skill-definition-2-set-a-default-value-for-a-value-that-doesnt-exist"></a>Exemple de définition de compétence 2 : Définir une valeur par défaut pour une valeur qui n’existe pas

La sortie suivante crée une annotation ("/document/languageWithDefault") qui est définie sur la langue du document ou sur "es" si la langue n’est pas définie.

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document",
    "inputs": [
        { "name": "condition", "source": "= $(/document/language) == null" },
        { "name": "whenTrue", "source": "= 'es'" },
        { "name": "whenFalse", "source": "= $(/document/language)" }
    ],
    "outputs": [ { "name": "output", "targetName": "languageWithDefault" } ]
}
```

### <a name="sample-skill-definition-3-merge-values-from-two-fields-into-one"></a>Exemple de définition de compétence 3 : Fusionner les valeurs de deux champs dans un même champ

Dans cet exemple, certaines phrases ont une propriété *frenchSentiment*. Chaque fois que la propriété *frenchSentiment* est null, nous souhaitons utiliser la valeur *englishSentiment*. Nous attribuons la sortie à un membre qui est appelé *sentiment* ("/document/sentiment/*/sentiment").

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document/sentences/*",
    "inputs": [
        { "name": "condition", "source": "= $(/document/sentences/*/frenchSentiment) == null" },
        { "name": "whenTrue", "source": "/document/sentences/*/englishSentiment" },
        { "name": "whenFalse", "source": "/document/sentences/*/frenchSentiment" }
    ],
    "outputs": [ { "name": "output", "targetName": "sentiment" } ]
}
```

## <a name="transformation-example"></a>Exemples de transformation
### <a name="sample-skill-definition-4-data-transformation-on-a-single-field"></a>Exemple de définition de compétence 4 : Transformation des données sur un seul champ

Dans cet exemple, nous recevons un *sentiment* compris entre 0 et 1. Nous voulons le transformer pour qu’il soit compris entre -1 et 1. Nous pouvons utiliser la compétence conditionnelle pour effectuer cette transformation mineure.

Dans cet exemple, nous n’utilisons pas l’aspect conditionnel de la compétence, car la condition est toujours *true*.

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document/sentences/*",
    "inputs": [
        { "name": "condition", "source": "= true" },
        { "name": "whenTrue", "source": "= $(/document/sentences/*/sentiment) * 2 - 1" },
        { "name": "whenFalse", "source": "= 0" }
    ],
    "outputs": [ { "name": "output", "targetName": "normalizedSentiment" } ]
}
```

## <a name="special-considerations"></a>Considérations spéciales
Certains paramètres sont évalués. Vous devez donc suivre le modèle documenté avec une attention particulière. Les expressions doivent commencer par un signe égal. Un chemin d’accès doit être délimité par "$(" et ")". Veillez à placer les chaînes entre guillemets simples. Ceci aide l’évaluateur à faire la distinction entre les chaînes et les opérateurs et chemins d’accès réels. En outre, assurez-vous d’ajouter un espace blanc autour des opérateurs (par exemple, un "*" dans un chemin d’accès signifie autre chose qu’une multiplication).


## <a name="next-steps"></a>Étapes suivantes

+ [Compétences prédéfinies](cognitive-search-predefined-skills.md)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
