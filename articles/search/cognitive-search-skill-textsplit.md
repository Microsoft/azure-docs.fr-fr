---
title: Compétence cognitive Fractionnement de texte
titleSuffix: Azure Cognitive Search
description: Découpe le texte en blocs ou des pages de texte en fonction de leur longueur dans un pipeline d’enrichissement par IA dans la Recherche cognitive Azure.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: 52aaeb01fef551eee350c6db662c2690ef7b3e78
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84981946"
---
# <a name="text-split-cognitive-skill"></a>Compétence cognitive Fractionnement de texte

La compétence **Fractionnement de texte** découpe le texte en segments. Vous pouvez choisir de décomposer le texte en phrases ou en pages d’une longueur donnée. Cette opération est particulièrement utile si d’autres compétences en aval imposent des critères de longueur maximale du texte. 

> [!NOTE]
> Cette compétence n’est pas liée à une API Cognitive Services et son utilisation ne vous est pas facturée. Toutefois, vous devez toujours [attacher une ressource Cognitive Services](cognitive-search-attach-cognitive-services.md) pour remplacer l’option de ressource **Gratuit** qui vous limite à un petit nombre d’enrichissements quotidiens par jour.

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.SplitSkill 

## <a name="skill-parameters"></a>Paramètres de la compétence

Les paramètres respectent la casse.

| Nom du paramètre     | Description |
|--------------------|-------------|
| `textSplitMode`    | « pages » ou « sentences » (phrases) | 
| `maximumPageLength` | Si textSplitMode est défini sur « pages », il s’agit de la longueur maximale de la page selon `String.Length`. La valeur minimale est 300.  Si textSplitMode est réglé sur « pages », l’algorithme essaie de fractionner le texte en blocs d’une taille maximale de « maximumPageLength ». Dans ce cas, l’algorithme fera de son mieux pour arrêter la phrase sur une limite de phrase, de sorte que la taille du bloc peut être légèrement inférieure à « maximumPageLength ». | 
| `defaultLanguageCode` | (Facultatif) L’un des codes de langue suivants : `da, de, en, es, fi, fr, it, ko, pt`. La langue par défaut est l’anglais (en). Quelques points à prendre en compte :<ul><li>Si vous utilisez un format codelangue-codepays, seule la partie codelangue du format est utilisée.</li><li>Si la langue ne figure pas dans la liste précédente, la compétence Fractionnement découpe le texte suivant les limites de caractères.</li><li>Il est utile d’indiquer un code de langue pour éviter de couper un mot en deux dans les langues sans espaces blancs, par exemple, le chinois, le japonais et le coréen.</li><li>Si vous ne connaissez pas la langue (par exemple, vous devez fractionner le texte pour l’entrée dans [LanguageDetectionSkill](cognitive-search-skill-language-detection.md)), la valeur par défaut de l’anglais (en) doit être suffisante. </li></ul>  |


## <a name="skill-inputs"></a>Entrées de la compétence

| Nom du paramètre       | Description      |
|----------------------|------------------|
| `text`    | Texte à fractionner en sous-chaînes. |
| `languageCode`    | (Facultatif) Code de langue du document. Si vous ne connaissez pas la langue (par exemple, vous devez fractionner le texte pour l’entrée dans [LanguageDetectionSkill](cognitive-search-skill-language-detection.md)), vous pouvez retirer cette entrée.  |

## <a name="skill-outputs"></a>Sorties de la compétence 

| Nom du paramètre     | Description |
|--------------------|-------------|
| `textItems`   | Tableau des sous-chaînes extraites. |


##  <a name="sample-definition"></a>Exemple de définition

```json
{
    "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
    "textSplitMode" : "pages", 
    "maximumPageLength": 1000,
    "defaultLanguageCode": "en",
    "inputs": [
        {
            "name": "text",
            "source": "/document/content"
        },
        {
            "name": "languageCode",
            "source": "/document/language"
        }
    ],
    "outputs": [
        {
            "name": "textItems",
            "targetName": "mypages"
        }
    ]
}
```

##  <a name="sample-input"></a>Exemple d’entrée

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "text": "This is a the loan application for Joe Romero, a Microsoft employee who was born in Chile and who then moved to Australia…",
                "languageCode": "en"
            }
        },
        {
            "recordId": "2",
            "data": {
                "text": "This is the second document, which will be broken into several pages...",
                "languageCode": "en"
            }
        }
    ]
}
```

##  <a name="sample-output"></a>Exemple de sortie

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "textItems": [
                    "This is the loan…",
                    "On the second page we…"
                ]
            }
        },
        {
            "recordId": "2",
            "data": {
                "textItems": [
                    "This is the second document...",
                    "On the second page of the second doc…"
                ]
            }
        }
    ]
}
```

## <a name="error-cases"></a>Cas d’erreur
Si la langue n’est pas prise en charge, un avertissement est généré et le texte est fractionné suivant les limites de caractères.

## <a name="see-also"></a>Voir aussi

+ [Compétences prédéfinies](cognitive-search-predefined-skills.md)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
