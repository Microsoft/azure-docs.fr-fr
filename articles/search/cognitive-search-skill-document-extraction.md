---
title: Compétence cognitive Extraction de document (préversion)
titleSuffix: Azure Cognitive Search
description: Extrait le contenu d’un fichier dans le pipeline d’enrichissement. Cette compétence est actuellement en préversion publique.
manager: nitinme
author: careyjmac
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/17/2020
ms.author: chalton
ms.openlocfilehash: 5098d897d84ff6af31d430c0ddd2e26d8d6b08a7
ms.sourcegitcommit: 1140ff2b0424633e6e10797f6654359947038b8d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97814293"
---
# <a name="document-extraction-cognitive-skill"></a>Compétence cognitive Extraction de document

> [!IMPORTANT] 
> Cette compétence est actuellement en préversion publique. Les fonctionnalités en préversion sont fournies sans contrat de niveau de service et ne sont pas recommandées pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Il n’y a actuellement pas de prise en charge du portail ou du SDK .NET.

La compétence **Extraction de document** extrait le contenu d’un fichier dans le pipeline d’enrichissement. Cela vous permet de tirer parti de l’étape d’extraction de document qui se produit normalement avant l’exécution des compétences avec des fichiers qui peuvent être générés par d’autres compétences.

> [!NOTE]
> Si vous élargissez le champ en augmentant la fréquence des traitements, en ajoutant des documents supplémentaires ou en ajoutant plusieurs algorithmes d’IA, vous devez [attacher une ressource Cognitive Services facturable](cognitive-search-attach-cognitive-services.md). Des frais sont applicables durant l’appel des API dans Cognitive Services ainsi que pour l’extraction d’images durant la phase d’extraction du contenu des documents dans l’indexation. L’extraction de texte à partir des documents est gratuite.
>
> L'exécution des compétences intégrées est facturée au prix actuel du [paiement à l'utilisation de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/). Les prix appliqués pour l'extraction d'images sont présentés sur la [page de tarification](https://azure.microsoft.com/pricing/details/search/).

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Util.DocumentExtractionSkill

## <a name="skill-parameters"></a>Paramètres de la compétence

Les paramètres respectent la casse.

| Entrées | Valeurs autorisées | Description |
|-----------------|----------------|-------------|
| `parsingMode`   | `default` <br/> `text` <br/> `json`  | Définissez ce paramètre sur `default` pour l’extraction de documents à partir de fichiers qui ne sont pas en mode texte pur ou JSON. Définissez ce paramètre sur `text` pour améliorer les performances des fichiers en texte brut. Définissez ce paramètre sur `json` pour extraire le contenu structuré des fichiers JSON. Si `parsingMode` n’est pas défini explicitement, il sera défini sur `default`. |
| `dataToExtract` | `contentAndMetadata` <br/> `allMetadata` | Définissez ce paramètre sur `contentAndMetadata` pour extraire toutes les métadonnées et le contenu textuel de chaque fichier. Définissez ce paramètre sur `allMetadata` pour extraire uniquement les [métadonnées spécifiques au type de contenu](search-howto-indexing-azure-blob-storage.md#ContentSpecificMetadata) (par exemple, seules les métadonnées propres aux fichiers .png). Si `dataToExtract` n’est pas défini explicitement, il sera défini sur `contentAndMetadata`. |
| `configuration` | Voir ci-dessous. | Dictionnaire de paramètres facultatifs qui ajustent le mode d’exécution de l’extraction de document. Reportez-vous au tableau ci-dessous pour consulter une description des propriétés de configuration prises en charge. |

| Paramètre de configuration   | Valeurs autorisées | Description |
|-------------------------|----------------|-------------|
| `imageAction`           | `none`<br/> `generateNormalizedImages`<br/> `generateNormalizedImagePerPage` | Définissez ce paramètre sur `none` pour ignorer les images incorporées ou les fichiers image du jeu de données. Il s’agit de la valeur par défaut. <br/>Pour l’[analyse d’image à l’aide de compétences cognitives](cognitive-search-concept-image-scenarios.md), définissez ce paramètre sur `generateNormalizedImages` pour que la compétence crée un tableau d’images normalisées dans le cadre du craquage de documents. Cette action nécessite de définir le paramètre `parsingMode` sur `default` et le paramètre `dataToExtract` sur `contentAndMetadata`. Une image normalisée fait référence à un traitement supplémentaire qui entraîne une sortie d’image uniforme, dimensionnée et pivotée pour promouvoir un rendu cohérent lorsque vous incluez des images dans les résultats d’une recherche visuelle (par exemple, des photographies de même taille dans un contrôle de graphique comme illustré dans la [démonstration JFK](https://github.com/Microsoft/AzureSearch_JFK_Files)). Ces informations sont générées pour chaque image lorsque vous utilisez cette option.  <br/>Si vous définissez le paramètre sur `generateNormalizedImagePerPage`, les fichiers PDF sont traités différemment. Au lieu d’extraire les images incorporées, chaque page est rendue sous la forme d’une image et normalisée en conséquence.  Les autres types de fichiers sont traités de la même façon que si `generateNormalizedImages` était défini.
| `normalizedImageMaxWidth` | Entier compris entre 50 et 10000 | Largeur maximale (en pixels) des images normalisées générées. La valeur par défaut est 2000. | 
| `normalizedImageMaxHeight` | Entier compris entre 50 et 10000 | Hauteur maximale (en pixels) des images normalisées générées. La valeur par défaut est 2000. |

> [!NOTE]
> La valeur par défaut de 2000 pixels pour la hauteur et la largeur maximales des images normalisées est basée sur les tailles maximales prises en charge par la [compétence de reconnaissance optique de caractères](cognitive-search-skill-ocr.md) et la [compétence d’analyse d’image](cognitive-search-skill-image-analysis.md). La [compétence de reconnaissance optique de caractères](cognitive-search-skill-ocr.md) (OCR) prend en charge une largeur et une hauteur maximales de 4200 pour les langues autres que l'anglais et de 10000 pour l'anglais.  Si vous augmentez les limites maximales, le traitement des images plus volumineuses peut échouer en fonction de la définition de vos compétences et de la langue des documents. 
## <a name="skill-inputs"></a>Entrées de la compétence

| Nom d’entrée     | Description |
|--------------------|-------------|
| `file_data` | Fichier à partir duquel le contenu doit être extrait. |

L’entrée « file_data » doit être un objet défini comme suit :

```json
{
  "$type": "file",
  "data": "BASE64 encoded string of the file"
}
```

Cet objet de référence de fichier peut être généré de 3 manières :

 - En définissant le paramètre `allowSkillsetToReadFileData` de votre définition d’indexeur sur la valeur « true ».  Ceci permet de créer un chemin `/document/file_data` qui est un objet représentant les données du fichier d’origine téléchargées à partir de votre source de données d’objets blob. Ce paramètre s’applique uniquement aux données dans le stockage d’objets BLOB.

 - En définissant le paramètre `imageAction` de votre définition d’indexeur sur autre valeur que `none`.  Ceci crée un tableau d’images qui suivent la convention nécessaire pour les entrées de cette compétence, si elles sont passées individuellement (c’est-à-dire `/document/normalized_images/*`).

 - Avoir une compétence personnalisée renvoie un objet JSON défini exactement comme indiqué ci-dessus.  Le paramètre `$type` doit être défini sur `file` et le paramètre `data` doit être données du tableau d’octets encodé en base 64 du contenu du fichier.

## <a name="skill-outputs"></a>Sorties de la compétence

| Nom de sortie    | Description |
|--------------|-------------|
| `content` | Contenu textuel du document. |
| `normalized_images`   | Si la propriété `imageAction` est définie sur une autre valeur que `none`, le nouveau champ *normalized_images* contiendra un tableau d’images. Si vous souhaitez en savoir plus sur le format de sortie de chaque image, veuillez consulter la [documentation relative à l’extraction d’images](cognitive-search-concept-image-scenarios.md). |

##  <a name="sample-definition"></a>Exemple de définition

```json
 {
    "@odata.type": "#Microsoft.Skills.Util.DocumentExtractionSkill",
    "parsingMode": "default",
    "dataToExtract": "contentAndMetadata",
    "configuration": {
        "imageAction": "generateNormalizedImages",
        "normalizedImageMaxWidth": 2000,
        "normalizedImageMaxHeight": 2000
    },
    "context": "/document",
    "inputs": [
      {
        "name": "file_data",
        "source": "/document/file_data"
      }
    ],
    "outputs": [
      {
        "name": "content",
        "targetName": "extracted_content"
      },
      {
        "name": "normalized_images",
        "targetName": "extracted_normalized_images"
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
        "file_data": {
          "$type": "file",
          "data": "aGVsbG8="
        }
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
        "content": "hello",
        "normalized_images": []
      }
    }
  ]
}
```

## <a name="see-also"></a>Voir aussi

+ [Compétences prédéfinies](cognitive-search-predefined-skills.md)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
+ [Comment traiter et extraire des informations d’images dans des scénarios de recherche cognitive](cognitive-search-concept-image-scenarios.md)