---
title: Méthode Dictionary Lookup de Translator
titleSuffix: Azure Cognitive Services
description: La méthode de recherche dans le dictionnaire indique les autres traductions d’un mot et quelques expressions idiomatiques.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 01/21/2020
ms.author: swmachan
ms.openlocfilehash: 6ada9c06f0befbea4b9aa85f95651e96dec595b7
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021284"
---
# <a name="translator-30-dictionary-lookup"></a>Translator 3.0 : Recherche dans le dictionnaire

Indique les autres traductions d’un mot et quelques expressions idiomatiques. Pour chaque terme, vous trouverez des informations sur la nature grammaticale ainsi qu’une liste de traductions inverses. Les traductions inverses sont utiles pour comprendre la traduction dans son contexte. L’opération [Exemple de dictionnaire](./v3-0-dictionary-examples.md) permet d’avoir un aperçu plus détaillé en affichant des exemples d’utilisation de chaque paire de traduction.

## <a name="request-url"></a>URL de la demande

Envoyez une demande `POST` à :

```HTTP
https://api.cognitive.microsofttranslator.com/dictionary/lookup?api-version=3.0
```

## <a name="request-parameters"></a>Paramètres de la demande

Les paramètres de demande transmis à la chaîne de requête sont les suivants :

| Paramètre de requête  | Description |
| ------ | ----------- |
| api-version <img width=200/>   | **Paramètre obligatoire**.<br/>Version de l’API demandée par le client. La valeur doit être `3.0` |
| de | **Paramètre obligatoire**.<br/>Spécifie la langue du texte d’entrée. La langue source doit être l’une des [langues prises en charge](./v3-0-languages.md) incluses dans l’étendue `dictionary`. |
| to   | **Paramètre obligatoire**.<br/>Spécifie la langue du texte de sortie. La langue cible doit être l’une des [langues prises en charge](v3-0-languages.md) incluses dans l’étendue `dictionary`. |


Les en-têtes de demande sont les suivants :

| headers  | Description |
| ------ | ----------- |
| En-tête(s) d’authentification <img width=200/>  | **En-tête de demande obligatoire**.<br/>Voir les <a href="/azure/cognitive-services/translator/reference/v3-0-reference#authentication">options disponibles pour l’authentification</a>. |
| Content-Type | **En-tête de demande obligatoire**.<br/>Spécifie le type de contenu de la charge utile. Les valeurs possibles sont les suivantes : `application/json`. |
| Content-Length   | **En-tête de demande obligatoire**.<br/>Longueur du corps de la demande. |
| X-ClientTraceId   | **Facultatif**.<br/>GUID généré par le client pour identifier de façon unique la demande. Vous pouvez omettre cet en-tête si vous incluez l’ID de trace dans la chaîne de requête à l’aide d’un paramètre de requête appelé `ClientTraceId`. |

## <a name="request-body"></a>Corps de la demande

Le corps de la demande est un tableau JSON. Chaque élément du tableau est un objet JSON avec une propriété de chaîne nommée `Text`, qui représente le terme à rechercher.

```json
[
    {"Text":"fly"}
]
```

Les limites suivantes s'appliquent :

* Le tableau ne peut pas compter plus de 10 éléments.
* La valeur texte d’un élément de tableau ne peut pas dépasser 100 caractères, espaces compris.

## <a name="response-body"></a>Response body

Une réponse correcte est un tableau JSON avec un résultat pour chaque chaîne dans le tableau d’entrée. Un objet de résultat inclut les propriétés suivantes :

  * `normalizedSource`: chaîne indiquant la forme normalisée du terme source. Par exemple, si la requête est « JOHN », la forme normalisée sera « john ». Le contenu de ce champ devient l’entrée des [exemples de recherche](./v3-0-dictionary-examples.md).
    
  * `displaySource`: chaîne affichant le terme source sous une forme mieux adaptée à l’affichage à l’utilisateur final. Par exemple, si l’entrée est « JOHN », la forme d’affichage reflète l’orthographe habituel du nom : « John ». 

  * `translations`: liste des traductions du terme source. Chaque élément de la liste est un objet dont les propriétés sont les suivantes :

    * `normalizedTarget`: chaîne indiquant la forme normalisée de ce terme dans la langue cible. Cette valeur doit être utilisée comme entrée pour les [exemples de recherche](./v3-0-dictionary-examples.md).

    * `displayTarget`: chaîne affichant le terme dans la langue cible et sous une forme mieux adaptée à l’affichage à l’utilisateur final. En règle générale, la seule différence avec le `normalizedTarget` réside au niveau de la mise en majuscules. Par exemple, un nom propre tel que « Juan » aura `normalizedTarget = "juan"` et `displayTarget = "Juan"`.

    * `posTag`: chaîne associant ce terme à une balise morphosyntaxique.

        | Nom de la balise | Description  |
        |----------|--------------|
        | ADJ      | Adjectifs   |
        | ADV      | Adverbes      |
        | CONJ     | Conjonctions |
        | DET      | Déterminants  |
        | MODAL    | Verbes et adverbes        |
        | NOUN     | Noms        |
        | PREP     | Prépositions |
        | PRON     | Pronoms     |
        | VERB     | Verbes et adverbes        |
        | OTHER    | Autres        |

        En tant que note d’implémentation, ces balises ont été déterminées par le balisage morphosyntaxique de l’anglais, puis en prenant la balise la plus fréquente pour chaque paire source/cible. Par conséquent, si les gens traduisent fréquemment un mot espagnol par une autre balise morphosyntaxique en anglais, les balises risquent d’être erronées (par rapport au mot espagnol).

    * `confidence`: valeur comprise entre 0,0 et 1,0 représentant la « confiance » (ou pour être plus précis la « probabilité dans les données d’entraînement ») de cette paire source-traduction. La somme des scores de confiance d’un mot source peut ou non totaliser 1,0. 

    * `prefixWord`: chaîne indiquant le mot à afficher en préfixe de la traduction. Actuellement, il s’agit du déterminant genré des noms, dans les langues utilisant des déterminants genrés. Par exemple, le préfixe du mot espagnol « mosca » est « la », car « mosca » est un nom féminin en espagnol. Cela dépend uniquement de la traduction, et non pas de la source. En l’absence de préfixe, la chaîne sera vide.
    
    * `backTranslations`: liste des « traductions inverses » de la cible. Par exemple, les mots source pouvant être traduits dans la langue cible. La liste est assurée de contenir le mot source qui a été demandé (par exemple, si le mot source recherché est « fly », vous êtes sûr que « fly » figurera dans la liste `backTranslations`). Toutefois, il n’est pas garanti que le terme arrive en première position. Chaque élément de la liste `backTranslations` est un objet décrit par les propriétés suivantes :

        * `normalizedText`: chaîne indiquant la forme normalisée du terme source qui est une traduction inverse de la cible. Cette valeur doit être utilisée comme entrée pour les [exemples de recherche](./v3-0-dictionary-examples.md).        

        * `displayText`: chaîne indiquant le terme source qui est une traduction inverse de la cible sous une forme mieux adaptée à l’affichage à l’utilisateur final.

        * `numExamples`: nombre entier représentant le nombre d’exemples disponibles pour cette paire source-traduction. Les exemples réels doivent être récupérés avec un appel séparé aux [exemples de recherche](./v3-0-dictionary-examples.md). Le nombre est principalement destiné à faciliter l’affichage dans une expérience utilisateur. Par exemple, une interface utilisateur peut ajouter un lien hypertexte vers la traduction inverse si le nombre d’exemples est supérieur à zéro, et afficher la traduction inverse en tant que texte brut s’il n’y a aucun exemple. Notez que le nombre réel d’exemples retournés par un appel aux [exemples de recherche](./v3-0-dictionary-examples.md) peut être inférieur à `numExamples`, car un filtrage supplémentaire peut être appliqué à la volée pour supprimer les exemples non pertinents.
        
        * `frequencyCount`: nombre entier représentant la fréquence de cette paire source-traduction dans les données. L’objectif principal de ce champ est de fournir une interface utilisateur dans laquelle il est possible de trier les traductions inverses afin que les termes les plus fréquents apparaissent en premier.

    > [!NOTE]
    > Si le terme recherché ne figure pas dans le dictionnaire, la réponse est 200 (OK), mais la liste `translations` est vide.

## <a name="examples"></a>Exemples

Cet exemple montre comment rechercher d’autres traductions en espagnol du terme anglais `fly`.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/dictionary/lookup?api-version=3.0&from=en&to=es" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'fly'}]"
```

Le corps de la réponse (abrégé pour plus de clarté) est :

```
[
    {
        "normalizedSource":"fly",
        "displaySource":"fly",
        "translations":[
            {
                "normalizedTarget":"volar",
                "displayTarget":"volar",
                "posTag":"VERB",
                "confidence":0.4081,
                "prefixWord":"",
                "backTranslations":[
                    {"normalizedText":"fly","displayText":"fly","numExamples":15,"frequencyCount":4637},
                    {"normalizedText":"flying","displayText":"flying","numExamples":15,"frequencyCount":1365},
                    {"normalizedText":"blow","displayText":"blow","numExamples":15,"frequencyCount":503},
                    {"normalizedText":"flight","displayText":"flight","numExamples":15,"frequencyCount":135}
                ]
            },
            {
                "normalizedTarget":"mosca",
                "displayTarget":"mosca",
                "posTag":"NOUN",
                "confidence":0.2668,
                "prefixWord":"",
                "backTranslations":[
                    {"normalizedText":"fly","displayText":"fly","numExamples":15,"frequencyCount":1697},
                    {"normalizedText":"flyweight","displayText":"flyweight","numExamples":0,"frequencyCount":48},
                    {"normalizedText":"flies","displayText":"flies","numExamples":9,"frequencyCount":34}
                ]
            },
            //
            // ...list abbreviated for documentation clarity
            //
        ]
    }
]
```

Cet exemple montre ce qui se passe lorsque le terme recherché n’existe pas pour la paire de dictionnaire valide.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/dictionary/lookup?api-version=3.0&from=en&to=es" -H "X-ClientTraceId: 875030C7-5380-40B8-8A03-63DACCF69C11" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'fly123456'}]"
```

Étant donné que le terme ne figure pas dans le dictionnaire, le corps de réponse inclut une liste `translations` vide.

```
[
    {
        "normalizedSource":"fly123456",
        "displaySource":"fly123456",
        "translations":[]
    }
]
```