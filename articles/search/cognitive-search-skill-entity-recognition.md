---
title: Compétence cognitive Reconnaissance d’entités
titleSuffix: Azure Cognitive Search
description: Extrayez différents types d’entités d’un texte dans un pipeline d’enrichissement dans la Recherche cognitive Azure.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: ddfca965ea32ca726df39f894ba45f9580225d9d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91542459"
---
#    <a name="entity-recognition-cognitive-skill"></a>Compétence cognitive Reconnaissance d’entités

La compétence **Reconnaissance d’entités** extrait les entités de différents types du texte. Cette compétence utilise les modèles Machine Learning fournis par [Analyse de texte](../cognitive-services/text-analytics/overview.md) dans Cognitive Services.

> [!NOTE]
> Si vous élargissez le champ en augmentant la fréquence des traitements, en ajoutant des documents supplémentaires ou en ajoutant plusieurs algorithmes d’IA, vous devez [attacher une ressource Cognitive Services facturable](cognitive-search-attach-cognitive-services.md). Des frais s’appliquent durant l’appel des API dans Cognitive Services ainsi que pour l’extraction d’images dans le cadre de la phase de craquage de document de la Recherche cognitive Azure. L’extraction de texte à partir des documents est gratuite.
>
> L'exécution des compétences intégrées est facturée au prix actuel du [paiement à l'utilisation de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/). Les prix appliqués pour l’extraction d’images sont présentés sur la [page de tarification du service Recherche cognitive Azure](https://azure.microsoft.com/pricing/details/search/).


## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.EntityRecognitionSkill

## <a name="data-limits"></a>Limites de données
La taille maximale d’un enregistrement doit être de 50 000 caractères telle que mesurée par [`String.Length`](/dotnet/api/system.string.length). Si vous devez subdiviser vos données avant de les envoyer à l’extracteur de phrases clés, envisagez d’utiliser la [compétence Fractionnement de texte](cognitive-search-skill-textsplit.md).

## <a name="skill-parameters"></a>Paramètres de la compétence

Les paramètres respectent la casse et sont tous facultatifs.

| Nom du paramètre     | Description |
|--------------------|-------------|
| `categories`    | Tableau des catégories à extraire.  Types de catégorie possibles : `"Person"`, `"Location"`, `"Organization"`, `"Quantity"`, `"Datetime"`, `"URL"`, `"Email"`. Si aucune catégorie n’est précisée, tous les types sont retournés.|
| `defaultLanguageCode` |    Code de langue du texte d’entrée. Langues prises en charge : `ar, cs, da, de, en, es, fi, fr, hu, it, ja, ko, nl, no, pl, pt-BR, pt-PT, ru, sv, tr, zh-hans`. Les catégories d’entités ne sont pas toutes prises en charge pour toutes les langues. Consultez la remarque ci-dessous.|
| `minimumPrecision` | Valeur comprise entre 0 et 1 Si le score de confiance (dans la sortie `namedEntities`) est inférieur à cette valeur, l’entité n’est pas retournée. La valeur par défaut est 0. |
| `includeTypelessEntities` | Affectez la valeur `true` si vous souhaitez reconnaître les entités connues qui ne correspondent pas aux catégories actuelles. Les entités reconnues sont retournées dans le champ de sortie complexe `entities`. Par exemple, « Windows 10 » est une entité bien connue (un produit), mais étant donné que la catégorie « Produits » n’est pas prise en charge, cette entité est incluse dans le champ de sortie des entités. La valeur par défaut est `false` |


## <a name="skill-inputs"></a>Entrées de la compétence

| Nom d’entrée      | Description                   |
|---------------|-------------------------------|
| `languageCode`    | facultatif. La valeur par défaut est `"en"`.  |
| `text`          | Texte à analyser.          |

## <a name="skill-outputs"></a>Sorties de la compétence

> [!NOTE]
> toutes les catégories d’entités ne sont pas prises en charge pour toutes les langues. Les types de catégories d’entités `"Person"`, `"Location"` et `"Organization"` sont pris en charge pour la liste complète des langues ci-dessus. Seules les langues _de_, _en_, _es_, _fr_ et _zh-hans_ prennent en charge l’extraction des types `"Quantity"`, `"Datetime"`, `"URL"` et `"Email"`. Pour plus d’informations, consultez [Langues et régions prises en charge par l’API Analyse de texte](../cognitive-services/text-analytics/language-support.md).  

| Nom de sortie      | Description                   |
|---------------|-------------------------------|
| `persons`       | Tableau de chaînes représentant chacune le nom d’une personne. |
| `locations`  | Tableau de chaînes représentant chacune un lieu. |
| `organizations`  | Tableau de chaînes représentant chacune une organisation. |
| `quantities`  | Tableau de chaînes représentant chacune une quantité. |
| `dateTimes`  | Tableau de chaînes représentant chacune une valeur DateTime (telle qu’elle apparaît dans le texte). |
| `urls` | Tableau de chaînes représentant chacune une URL. |
| `emails` | Tableau de chaînes représentant chacune un e-mail. |
| `namedEntities` | Tableau de types complexes contenant les champs suivants : <ul><li>catégorie</li> <li>la valeur (le nom réel de l’entité) ;</li><li>le décalage (l’emplacement où elle a été trouvée dans le texte) ;</li><li>confiance (une valeur plus élevée signifie qu’il s’agit d’une entité réelle)</li></ul> |
| `entities` | Tableau de types complexes contenant des informations détaillées sur les entités extraites du texte, avec les champs suivants <ul><li> name (nom réel de l’entité ; il représente une forme « normalisée »)</li><li> wikipediaId</li><li>wikipediaLanguage</li><li>wikipediaUrl (lien vers la page Wikipedia relative à l’entité)</li><li>bingId</li><li>type (catégorie de l’entité reconnue)</li><li>subType (disponible uniquement pour certaines catégories ; il offre une vue plus précise du type d’entité)</li><li> matches (collection complexe contenant)<ul><li>text (texte brut pour l’entité)</li><li>offset (emplacement où cela a été trouvé)</li><li>length (longueur du texte brut d’entité)</li></ul></li></ul> |

##    <a name="sample-definition"></a>Exemple de définition

```json
  {
    "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
    "categories": [ "Person", "Email"],
    "defaultLanguageCode": "en",
    "includeTypelessEntities": true,
    "minimumPrecision": 0.5,
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      }
    ],
    "outputs": [
      {
        "name": "persons",
        "targetName": "people"
      },
      {
        "name": "emails",
        "targetName": "contact"
      },
      {
        "name": "entities"
      }
    ]
  }
```
##    <a name="sample-input"></a>Exemple d’entrée

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
           {
             "text": "Contoso corporation was founded by John Smith. They can be reached at contact@contoso.com",
             "languageCode": "en"
           }
      }
    ]
}
```

##    <a name="sample-output"></a>Exemple de sortie

```json
{
  "values": [
    {
      "recordId": "1",
      "data" : 
      {
        "persons": [ "John Smith"],
        "emails":["contact@contoso.com"],
        "namedEntities": 
        [
          {
            "category":"Person",
            "value": "John Smith",
            "offset": 35,
            "confidence": 0.98
          }
        ],
        "entities":  
        [
          {
            "name":"John Smith",
            "wikipediaId": null,
            "wikipediaLanguage": null,
            "wikipediaUrl": null,
            "bingId": null,
            "type": "Person",
            "subType": null,
            "matches": [{
                "text": "John Smith",
                "offset": 35,
                "length": 10
            }]
          },
          {
            "name": "contact@contoso.com",
            "wikipediaId": null,
            "wikipediaLanguage": null,
            "wikipediaUrl": null,
            "bingId": null,
            "type": "Email",
            "subType": null,
            "matches": [
            {
                "text": "contact@contoso.com",
                "offset": 70,
                "length": 19
            }]
          },
          {
            "name": "Contoso",
            "wikipediaId": "Contoso",
            "wikipediaLanguage": "en",
            "wikipediaUrl": "https://en.wikipedia.org/wiki/Contoso",
            "bingId": "349f014e-7a37-e619-0374-787ebb288113",
            "type": null,
            "subType": null,
            "matches": [
            {
                "text": "Contoso",
                "offset": 0,
                "length": 7
            }]
          }
        ]
      }
    }
  ]
}
```

Notez que les décalages renvoyés pour les entités dans la sortie de cette qualification le sont directement à partir de l'[API Analyse de texte](../cognitive-services/text-analytics/overview.md), ce qui signifie que si vous les utilisez pour indexer dans la chaîne d'origine, vous devez utiliser la classe [StringInfo](/dotnet/api/system.globalization.stringinfo) dans .NET afin d'extraire le bon contenu.  [Pour plus d'informations, cliquez ici](../cognitive-services/text-analytics/concepts/text-offsets.md).

## <a name="error-cases"></a>Cas d’erreur
Si le code de langue du document n’est pas pris en charge, une erreur est retournée et aucune entité n’est extraite.

## <a name="see-also"></a>Voir aussi

+ [Compétences prédéfinies](cognitive-search-predefined-skills.md)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)