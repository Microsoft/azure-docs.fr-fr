---
title: Compétence cognitive Traduction de texte
titleSuffix: Azure Cognitive Search
description: Évalue le texte et, pour chaque enregistrement, retourne le texte traduit dans la langue cible spécifiée dans un pipeline d’enrichissement de l’IA dans la Recherche cognitive Azure.
manager: nitinme
author: careyjmac
ms.author: chalton
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 0953d750ee8b59e9889512bb64cfd276a0bbeb53
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97654862"
---
#   <a name="text-translation-cognitive-skill"></a>Compétence cognitive Traduction de texte

La compétence **Traduction de texte** évalue le texte et, pour chaque enregistrement, retourne le texte traduit dans la langue cible spécifiée. Cette compétence utilise l’[API de traduction de texte Translator Text v3.0](../cognitive-services/translator/reference/v3-0-translate.md) disponible dans Cognitive Services.

Cette fonctionnalité est utile si vous vous attendez à ce que vos documents ne soient pas tous dans la même langue, auquel cas vous pouvez normaliser le texte dans une seule langue avant l’indexation de la recherche en le traduisant.  Elle est également utile dans les cas d’usage de localisation, où vous pouvez avoir besoin de copies du même texte dans plusieurs langues.

L’[API Traduction de texte Translator Text v3.0](../cognitive-services/translator/reference/v3-0-reference.md) est un service cognitif non régional, ce qui signifie que vos données ne sont pas conservées dans la même région que votre ressource Recherche cognitive Azure ou Cognitive Services associée.

> [!NOTE]
> Si vous élargissez le champ en augmentant la fréquence des traitements, en ajoutant des documents supplémentaires ou en ajoutant plusieurs algorithmes d’IA, vous devez [attacher une ressource Cognitive Services facturable](cognitive-search-attach-cognitive-services.md). Des frais s’appliquent durant l’appel des API dans Cognitive Services ainsi que pour l’extraction d’images dans le cadre de la phase de craquage de document de la Recherche cognitive Azure. L’extraction de texte à partir des documents est gratuite.
>
> L'exécution des compétences intégrées est facturée au prix actuel du [paiement à l'utilisation de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/). Les prix appliqués pour l’extraction d’images sont présentés sur la [page de tarification du service Recherche cognitive Azure](https://azure.microsoft.com/pricing/details/search/).

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.TranslationSkill

## <a name="data-limits"></a>Limites de données
La taille maximale d’un enregistrement doit être de 50 000 caractères telle que mesurée par [`String.Length`](/dotnet/api/system.string.length). Si vous devez subdiviser vos données avant de les envoyer à la compétence de traduction de texte, utilisez la [compétence Fractionnement de texte](cognitive-search-skill-textsplit.md).

## <a name="skill-parameters"></a>Paramètres de la compétence

Les paramètres respectent la casse.

| Entrées | Description |
|---------------------|-------------|
| defaultToLanguageCode | (Obligatoire) Code de la langue dans laquelle traduire les documents, pour ceux qui ne spécifient pas explicitement la langue cible. <br/> Voir la [Liste complète des langues prises en charge](../cognitive-services/translator/language-support.md). |
| defaultFromLanguageCode | (Facultatif) Code de la langue à partir de laquelle traduire les documents, pour ceux qui ne spécifient pas explicitement la langue source.  Si defaultFromLanguageCode n’est pas spécifié, la détection automatique de la langue fournie par l’API de traduction de texte Translator Text est utilisée pour déterminer la langue source. <br/> Voir la [Liste complète des langues prises en charge](../cognitive-services/translator/language-support.md). |
| suggestedFrom | (Facultatif) Code de la langue à partir de laquelle traduire les documents quand ni l’entrée fromLanguageCode ni le paramètre defaultFromLanguageCode ne sont fournis, et que la détection automatique de la langue échoue.  Si la langue suggestedFrom n’est pas spécifiée, l’anglais (en) est utilisé comme langue suggestedFrom. <br/> Voir la [Liste complète des langues prises en charge](../cognitive-services/translator/language-support.md). |

## <a name="skill-inputs"></a>Entrées de la compétence

| Nom d’entrée     | Description |
|--------------------|-------------|
| text | Texte à traduire.|
| toLanguageCode    | Chaîne indiquant la langue dans laquelle le texte doit être traduit. Si cette entrée n’est pas spécifiée, defaultToLanguageCode est utilisé pour traduire le texte. <br/>Voir la [Liste complète des langues prises en charge](../cognitive-services/translator/language-support.md).|
| fromLanguageCode  | Chaîne indiquant la langue actuelle du texte. Si ce paramètre n’est pas spécifié, defaultFromLanguageCode (ou la détection automatique de la langue si defaultFromLanguageCode n’est pas fourni) est utilisé pour traduire le texte. <br/>Voir la [Liste complète des langues prises en charge](../cognitive-services/translator/language-support.md).|

## <a name="skill-outputs"></a>Sorties de la compétence

| Nom de sortie    | Description |
|--------------------|-------------|
| translatedText | Chaîne de résultat de la traduction du texte de translatedFromLanguageCode vers translatedToLanguageCode.|
| translatedToLanguageCode  | Chaîne indiquant le code de langue dans lequel le texte a été traduit. Utile si vous traduisez dans plusieurs langues et que vous souhaitez pouvoir effectuer le suivi de la langue de chaque texte.|
| translatedFromLanguageCode    | Chaîne indiquant le code de langue à partir duquel le texte a été traduit. Utile si vous avez opté pour l’option de détection automatique de la langue, car cette sortie vous donnera le résultat de cette détection.|

##  <a name="sample-definition"></a>Exemple de définition

```json
 {
    "@odata.type": "#Microsoft.Skills.Text.TranslationSkill",
    "defaultToLanguageCode": "fr",
    "suggestedFrom": "en",
    "context": "/document",
    "inputs": [
      {
        "name": "text",
        "source": "/document/text"
      }
    ],
    "outputs": [
      {
        "name": "translatedText",
        "targetName": "translatedText"
      },
      {
        "name": "translatedFromLanguageCode",
        "targetName": "translatedFromLanguageCode"
      },
      {
        "name": "translatedToLanguageCode",
        "targetName": "translatedToLanguageCode"
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
      "data":
        {
          "text": "We hold these truths to be self-evident, that all men are created equal."
        }
    },
    {
      "recordId": "2",
      "data":
        {
          "text": "Estamos muy felices de estar con ustedes."
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
      "data":
        {
          "translatedText": "Nous tenons ces vérités pour évidentes, que tous les hommes sont créés égaux.",
          "translatedFromLanguageCode": "en",
          "translatedToLanguageCode": "fr"
        }
    },
    {
      "recordId": "2",
      "data":
        {
          "translatedText": "Nous sommes très heureux d'être avec vous.",
          "translatedFromLanguageCode": "es",
          "translatedToLanguageCode": "fr"
        }
    }
  ]
}
```


## <a name="errors-and-warnings"></a>Erreurs et avertissements
Si vous fournissez un code de langue non pris en charge pour la langue source ou cible, une erreur est générée et le texte n’est pas traduit.
Si votre texte est vide, un avertissement est généré.
Si votre texte comprend plus de 50 000 caractères, seuls les 50 000 premiers caractères sont traduits et un avertissement est émis.

## <a name="see-also"></a>Voir aussi

+ [Compétences prédéfinies](cognitive-search-predefined-skills.md)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)