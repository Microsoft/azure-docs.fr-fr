---
title: Compétence cognitive OCR
titleSuffix: Azure Cognitive Search
description: Extrayez le texte de fichiers image à l’aide de la reconnaissance optique de caractères (OCR) dans un pipeline d’enrichissement dans la Recherche cognitive Azure.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: 8b6a7c3e05b26cbda80ebf1a3fc0d4fed8255e6b
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91950803"
---
# <a name="ocr-cognitive-skill"></a>Compétence cognitive OCR

La compétence de **reconnaissance optique de caractères (OCR)** reconnaît le texte imprimé et manuscrit dans des fichiers image. Cette compétence utilise les modèles Machine Learning fournis par l’API [Vision par ordinateur](../cognitive-services/computer-vision/overview.md) [v3.0](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-ga/operations/5d986960601faab4bf452005) dans Cognitive Services. La compétence de **reconnaissance optique de caractères (OCR)** est mappée à la fonctionnalité suivante :

+ Pour l’anglais, l’espagnol, l’allemand, le français, l’italien, le portugais et le néerlandais, la nouvelle API [« Lecture »](../cognitive-services/computer-vision/concept-recognizing-text.md#read-api) est utilisée.
+ Pour toutes les autres langues, l’API [« OCR »](../cognitive-services/computer-vision/concept-recognizing-text.md#ocr-api) est utilisée.

La compétence **OCR** extrait le texte de fichiers d’image. Les formats de fichiers pris en charge sont les suivants :

+ .JPEG
+ .JPG
+ .PNG
+ .BMP
+ .GIF
+ .TIFF

> [!NOTE]
> Si vous élargissez le champ en augmentant la fréquence des traitements, en ajoutant des documents supplémentaires ou en ajoutant plusieurs algorithmes d’IA, vous devez [attacher une ressource Cognitive Services facturable](cognitive-search-attach-cognitive-services.md). Des frais s’appliquent durant l’appel des API dans Cognitive Services ainsi que pour l’extraction d’images dans le cadre de la phase de craquage de document de la Recherche cognitive Azure. L’extraction de texte à partir des documents est gratuite.
>
> L'exécution des compétences intégrées est facturée au prix actuel du [paiement à l'utilisation de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/). Les prix appliqués pour l’extraction d’images sont présentés sur la [page de tarification du service Recherche cognitive Azure](https://azure.microsoft.com/pricing/details/search/).


## <a name="skill-parameters"></a>Paramètres de la compétence

Les paramètres respectent la casse.

| Nom du paramètre     | Description |
|--------------------|-------------|
| `detectOrientation`   | Active la détection automatique de l’orientation de l’image. <br/> Valeurs valides : true, false.|
| `defaultLanguageCode` | <p>   Code de langue du texte d’entrée. Les langages pris en charge incluent : <br/> zh-Hans (chinois simplifié) <br/> zh-Hant (chinois traditionnel) <br/>cs (tchèque) <br/>da (danois) <br/>nl (néerlandais) <br/>en (anglais) <br/>fi (finnois)  <br/>fr (français) <br/>  de (allemand) <br/>el (grec) <br/> hu (hongrois) <br/> it (italien) <br/>  ja (japonais) <br/> ko (coréen) <br/> nb (norvégien) <br/>   pl (polonais) <br/> pt (portugais) <br/>  ru (russe) <br/>  es (espagnol) <br/>  sv (suédois) <br/>  tr (turc) <br/> ar (arabe) <br/> ro (roumain) <br/> sr-Cyrl (serbe cyrillique) <br/> sr-Latn (serbe latin) <br/>  sk (slovaque) <br/>  unk (inconnu) <br/><br/> Si le code langue n’est pas spécifié ou est Null, la langue est définie automatiquement sur l’anglais. Si la langue est explicitement définie sur « unk », la langue sera détectée automatiquement. </p> |
| `lineEnding` | La valeur à utiliser entre chaque ligne détectée. Valeurs possibles : "Space", "CarriageReturn", "LineFeed".  La valeur par défaut est "Space" |

Auparavant, un paramètre appelé "textExtractionAlgorithm" servait à indiquer si la compétence devait extraire du texte "printed" ou "handwritten".  Ce paramètre est obsolète et inutile car le dernier algorithme de l'API Read est capable d'extraire les deux types de texte à la fois.  Si votre définition de compétence inclut déjà ce paramètre, inutile de le supprimer, mais il ne sera plus utilisé, et les deux types de texte seront dorénavant extraits, quel que soit leur paramétrage.

## <a name="skill-inputs"></a>Entrées de la compétence

| Nom d’entrée      | Description                                          |
|---------------|------------------------------------------------------|
| `image`         | Type complexe. Ne fonctionne actuellement qu'avec le champ « /documents/normalized_images », généré par l'indexeur d'objets Blob Azure lorsque ```imageAction``` est défini sur une valeur supérieure à ```none```. Pour plus d’informations, consultez [l’exemple](#sample-output).|


## <a name="skill-outputs"></a>Sorties de la compétence
| Nom de sortie     | Description                   |
|---------------|-------------------------------|
| `text`            | Texte brut extrait de l’image.   |
| `layoutText`    | Type complexe qui décrit le texte extrait ainsi que l’emplacement où le texte a été trouvé.|


## <a name="sample-definition"></a>Exemple de définition

```json
{
  "skills": [
    {
      "description": "Extracts text (plain and structured) from image.",
      "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
      "context": "/document/normalized_images/*",
      "defaultLanguageCode": null,
      "detectOrientation": true,
      "inputs": [
        {
          "name": "image",
          "source": "/document/normalized_images/*"
        }
      ],
      "outputs": [
        {
          "name": "text",
          "targetName": "myText"
        },
        {
          "name": "layoutText",
          "targetName": "myLayoutText"
        }
      ]
    }
  ]
}
```
<a name="sample-output"></a>

## <a name="sample-text-and-layouttext-output"></a>Exemple de sortie text et layoutText

```json
{
  "text": "Hello World. -John",
  "layoutText":
  {
    "language" : "en",
    "text" : "Hello World. -John",
    "lines" : [
      {
        "boundingBox":
        [ {"x":10, "y":10}, {"x":50, "y":10}, {"x":50, "y":30},{"x":10, "y":30}],
        "text":"Hello World."
      },
      {
        "boundingBox": [ {"x":110, "y":10}, {"x":150, "y":10}, {"x":150, "y":30},{"x":110, "y":30}],
        "text":"-John"
      }
    ],
    "words": [
      {
        "boundingBox": [ {"x":110, "y":10}, {"x":150, "y":10}, {"x":150, "y":30},{"x":110, "y":30}],
        "text":"Hello"
      },
      {
        "boundingBox": [ {"x":110, "y":10}, {"x":150, "y":10}, {"x":150, "y":30},{"x":110, "y":30}],
        "text":"World."
      },
      {
        "boundingBox": [ {"x":110, "y":10}, {"x":150, "y":10}, {"x":150, "y":30},{"x":110, "y":30}],
        "text":"-John"
      }
    ]
  }
}
```

## <a name="sample-merging-text-extracted-from-embedded-images-with-the-content-of-the-document"></a>Exemple : Fusion du texte extrait d’images incorporées avec le contenu du document.

La fusion de texte permet notamment de fusionner la représentation textuelle d’images (texte issu d’une compétence OCR ou légende d’une image) dans le champ de contenu d’un document.

L’exemple d’ensemble de compétences suivant crée un champ *merged_text*. Ce champ comprend le contenu textuel de votre document et le texte obtenu par reconnaissance optique de caractères de chacune des images incorporées dans ce document.

#### <a name="request-body-syntax"></a>Syntaxe du corps de la demande
```json
{
  "description": "Extract text from images and merge with content text to produce merged_text",
  "skills":
  [
    {
      "description": "Extract text (plain and structured) from image.",
      "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
      "context": "/document/normalized_images/*",
      "defaultLanguageCode": "en",
      "detectOrientation": true,
      "inputs": [
        {
          "name": "image",
          "source": "/document/normalized_images/*"
        }
      ],
      "outputs": [
        {
          "name": "text"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.MergeSkill",
      "description": "Create merged_text, which includes all the textual representation of each image inserted at the right location in the content field.",
      "context": "/document",
      "insertPreTag": " ",
      "insertPostTag": " ",
      "inputs": [
        {
          "name":"text",
          "source": "/document/content"
        },
        {
          "name": "itemsToInsert", 
          "source": "/document/normalized_images/*/text"
        },
        {
          "name":"offsets", 
          "source": "/document/normalized_images/*/contentOffset"
        }
      ],
      "outputs": [
        {
          "name": "mergedText", 
          "targetName" : "merged_text"
        }
      ]
    }
  ]
}
```
L’exemple d’ensemble de compétences ci-dessus suppose l’existence d’un champ d’images normalisées. Pour générer ce champ, définissez la configuration *imageAction* dans la définition de l’indexeur sur *generateNormalizedImages* comme indiqué ci-dessous :

```json
{
  //...rest of your indexer definition goes here ...
  "parameters": {
    "configuration": {
      "dataToExtract":"contentAndMetadata",
      "imageAction":"generateNormalizedImages"
    }
  }
}
```

## <a name="see-also"></a>Voir aussi
+ [Compétences prédéfinies](cognitive-search-predefined-skills.md)
+ [Compétence Fusion de texte](cognitive-search-skill-textmerger.md)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
+ [Créer un indexeur (REST)](/rest/api/searchservice/create-indexer)