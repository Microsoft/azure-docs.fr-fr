---
title: Compétence cognitive Détection PII (préversion)
titleSuffix: Azure Cognitive Search
description: Extrayez et masquez les informations personnelles d'un texte dans un pipeline d'enrichissement au sein de Recherche cognitive Azure. Cette compétence est actuellement en préversion publique.
manager: nitinme
author: careyjmac
ms.author: chalton
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: acacf617d3f1d9ab891d08b32fc2dfb14deb64a4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91540521"
---
# <a name="pii-detection-cognitive-skill"></a>Compétence cognitive Détection PII

> [!IMPORTANT] 
> Cette compétence est actuellement en préversion publique. Les fonctionnalités en préversion sont fournies sans contrat de niveau de service et ne sont pas recommandées pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Il n’y a actuellement pas de prise en charge du portail ou du SDK .NET.

La compétence **Détection des informations d'identification personnelle** extrait les informations personnelles d'un texte d'entrée et vous donne la possibilité de les masquer. Cette compétence utilise les modèles Machine Learning fournis par [Analyse de texte](../cognitive-services/text-analytics/overview.md) dans Cognitive Services.

> [!NOTE]
> Si vous élargissez le champ en augmentant la fréquence des traitements, en ajoutant des documents supplémentaires ou en ajoutant plusieurs algorithmes d’IA, vous devez [attacher une ressource Cognitive Services facturable](cognitive-search-attach-cognitive-services.md). Des frais s’appliquent durant l’appel des API dans Cognitive Services ainsi que pour l’extraction d’images dans le cadre de la phase de craquage de document de la Recherche cognitive Azure. L’extraction de texte à partir des documents est gratuite.
>
> L'exécution des compétences intégrées est facturée au prix actuel du [paiement à l'utilisation de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/). Les prix appliqués pour l’extraction d’images sont présentés sur la [page de tarification du service Recherche cognitive Azure](https://azure.microsoft.com/pricing/details/search/).

## <a name="odatatype"></a>@odata.type

Microsoft.Skills.Text.PIIDetectionSkill

## <a name="data-limits"></a>Limites de données

La taille maximale d’un enregistrement doit être de 50 000 caractères telle que mesurée par [`String.Length`](/dotnet/api/system.string.length). Si vous devez segmenter vos données avant de les envoyer à la compétence, utilisez la [compétence Fractionnement de texte](cognitive-search-skill-textsplit.md).

## <a name="skill-parameters"></a>Paramètres de la compétence

Les paramètres respectent la casse et sont tous facultatifs.

| Nom du paramètre     | Description |
|--------------------|-------------|
| `defaultLanguageCode` |    Code de langue du texte d’entrée. Pour le moment, seul `en` est pris en charge. |
| `minimumPrecision` | Valeur comprise entre 0.0 et 1.0. Si le score de confiance (dans la sortie `piiEntities`) est inférieur à la valeur `minimumPrecision` définie, l’entité n’est pas retournée ou masquée. L’option par défaut est 0.0. |
| `maskingMode` | Paramètre qui fournit différentes façons de masquer les informations personnelles détectées dans le texte d'entrée. Les options suivantes sont prises en charge : <ul><li>`none` (par défaut) : Aucun masquage n'intervient et la sortie `maskedText` n'est pas renvoyée. </li><li> `redact` : Supprime les entités détectées du texte d'entrée et ne remplace pas les valeurs supprimées. Dans ce cas, le décalage dans la sortie `piiEntities` est lié au texte d'origine, et non au texte masqué. </li><li> `replace` : Remplace les entités détectées par le caractère spécifié dans le paramètre `maskingCharacter`. Le caractère est répété sur la longueur de l’entité détectée afin que les décalages correspondent correctement à la fois au texte d’entrée et au `maskedText` de sortie.</li></ul> |
| `maskingCharacter` | Caractère utilisé pour masquer le texte si le paramètre `maskingMode` est défini sur `replace`. Les options suivantes sont prises en charge : `*` (par défaut), `#`, `X`. Ce paramètre peut uniquement être `null` si `maskingMode` n’est pas défini sur `replace`. |

## <a name="skill-inputs"></a>Entrées de la compétence

| Nom d’entrée      | Description                   |
|---------------|-------------------------------|
| `languageCode`    | facultatif. La valeur par défaut est `en`.  |
| `text`          | Texte à analyser.          |

## <a name="skill-outputs"></a>Sorties de la compétence

| Nom de sortie      | Description                   |
|---------------|-------------------------------|
| `piiEntities` | Tableau de types complexes contenant les champs suivants : <ul><li>text (informations d’identification personnelle réelles telles qu’elles ont été extraites)</li> <li>type</li><li>subtype</li><li>score (une valeur plus élevée signifie qu’il s’agit probablement d’une entité réelle)</li><li>offset (décalage dans le texte d’entrée)</li><li>length</li></ul> </br> [Vous trouverez des types et sous-types possibles ici.](../cognitive-services/text-analytics/named-entity-types.md?tabs=personal) |
| `maskedText` | Si `maskingMode` est défini sur une valeur autre que `none`, cette sortie correspond à la chaîne résultante du masquage effectué sur le texte d’entrée tel que décrit par le `maskingMode` sélectionné.  Si `maskingMode` est défini sur `none`, cette sortie n’est pas présente. |

## <a name="sample-definition"></a>Exemple de définition

```json
  {
    "@odata.type": "#Microsoft.Skills.Text.PIIDetectionSkill",
    "defaultLanguageCode": "en",
    "minimumPrecision": 0.5,
    "maskingMode": "replace",
    "maskingCharacter": "*",
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      }
    ],
    "outputs": [
      {
        "name": "piiEntities"
      },
      {
        "name": "maskedText"
      }
    ]
  }
```

## <a name="sample-input"></a>Exemple d’entrée

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
           {
             "text": "Microsoft employee with ssn 859-98-0987 is using our awesome API's."
           }
      }
    ]
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
{
  "values": [
    {
      "recordId": "1",
      "data" : 
      {
        "piiEntities":[ 
           { 
              "text":"859-98-0987",
              "type":"U.S. Social Security Number (SSN)",
              "subtype":"",
              "offset":28,
              "length":11,
              "score":0.65
           }
        ],
        "maskedText": "Microsoft employee with ssn *********** is using our awesome API's."
      }
    }
  ]
}
```

Les décalages renvoyés pour les entités dans la sortie de cette compétence le sont directement à partir de l'[API Analyse de texte](../cognitive-services/text-analytics/overview.md), ce qui signifie que si vous les utilisez pour indexer dans la chaîne d'origine, vous devez utiliser la classe [StringInfo](/dotnet/api/system.globalization.stringinfo) dans .NET afin d'extraire le bon contenu.  [Pour plus d’informations, cliquez ici.](../cognitive-services/text-analytics/concepts/text-offsets.md)

## <a name="errors-and-warnings"></a>Erreurs et avertissements

Si le code de langue du document n’est pas pris en charge, un avertissement est retourné et aucune entité n’est extraite.
Si votre texte est vide, un avertissement est renvoyé.
Si votre texte comprend plus de 50 000 caractères, seuls les 50 000 premiers caractères sont analysés et un avertissement est émis.

Si la compétence renvoie un avertissement, la sortie `maskedText` peut être vide, ce qui peut avoir un impact sur toutes les compétences qui attendent la sortie en aval. Vous devez donc rechercher tous les avertissements liés à une sortie manquante lors de l'écriture de la définition de votre ensemble de compétences.

## <a name="see-also"></a>Voir aussi

+ [Compétences prédéfinies](cognitive-search-predefined-skills.md)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)