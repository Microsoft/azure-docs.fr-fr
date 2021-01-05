---
title: Compétence cognitive Analyse d’image
titleSuffix: Azure Cognitive Search
description: Extrayez le texte sémantique par analyse d’image à l’aide de la compétence cognitive Analyse d’image dans un pipeline d’enrichissement de l’IA dans la Recherche cognitive Azure.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: 69b84a3edb606ed99b6aaca7db5ad0e57124f1b9
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91948933"
---
# <a name="image-analysis-cognitive-skill"></a>Compétence cognitive Analyse d’image

La compétence **Analyse d’image** extrait un ensemble complet de caractéristiques visuelles basées sur le contenu d’une image. Par exemple, vous pouvez générer une légende à partir d’une image, générer des balises ou identifier des célébrités et des paysages. Cette compétence utilise les modèles d’apprentissage automatique fournis par [Vision par ordinateur](../cognitive-services/computer-vision/overview.md) dans Cognitive Services. 

> [!NOTE]
> Les petits volumes (moins de 20 transactions) peuvent être exécutés gratuitement dans la Recherche cognitive Azure, mais l’exécution de charges de travail plus volumineuses nécessite l’[attachement d’une ressource facturable Cognitive Services](cognitive-search-attach-cognitive-services.md). Des frais s’appliquent durant l’appel des API dans Cognitive Services ainsi que pour l’extraction d’images dans le cadre de la phase de craquage de document de la Recherche cognitive Azure. L’extraction de texte à partir des documents est gratuite.
>
> L'exécution des compétences intégrées est facturée au prix actuel du [paiement à l'utilisation de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/). Les prix appliqués pour l’extraction d’images sont présentés sur la [page de tarification du service Recherche cognitive Azure](https://azure.microsoft.com/pricing/details/search/).


## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Vision.ImageAnalysisSkill 

## <a name="skill-parameters"></a>Paramètres de la compétence

Les paramètres respectent la casse.

| Nom du paramètre     | Description |
|--------------------|-------------|
| `defaultLanguageCode` |  Chaîne indiquant la langue à retourner. Le service retourne les résultats de la reconnaissance dans une langue donnée. Si ce paramètre n’est pas spécifié, la valeur par défaut est « en ». <br/><br/>Les langues prises en charge sont les suivantes : <br/>*en* : anglais (par défaut) <br/> *es* : espagnol <br/> *ja* : japonais <br/> *pt* : portugais <br/> *zh* : chinois simplifié|
| `visualFeatures` |    Tableau de chaînes qui indique les types de caractéristiques visuelles à retourner. Les types de caractéristiques visuelles valides sont les suivants :  <ul><li>*adult* : détecte si l’image est de nature pornographique (nudité ou acte sexuel) ou si elle est sordide (violence extrême ou sang). Le contenu sexuellement suggestif (ou contenu osé) est également détecté.</li><li>*brands* : détecte les différentes marques sur une image, y compris leur emplacement approximatif. La caractéristique visuelle *brands* n’est disponible qu’en anglais.</li><li> *categories* : catégorise le contenu de l’image en fonction d’une taxonomie définie dans la [documentation Vision par ordinateur](../cognitive-services/computer-vision/category-taxonomy.md) de Cognitive Services. </li><li>*description* : décrit le contenu de l’image avec une phrase complète dans les langues prises en charge.</li><li>*faces* : détecte si des visages sont présents. Si tel est le cas, génère des coordonnées, ainsi que des paramètres d’âge et de sexe.</li><li>  *objects* : détecte les différents objets sur une image, y compris leur emplacement approximatif. La caractéristique visuelle *objects* n’est disponible qu’en anglais.</li><li> *tags* : balise l’image avec une liste détaillée de mots liés au contenu de l’image.</li></ul> Les noms des caractéristiques visuelles respectent la casse. Notez que les caractéristiques visuelles *color* et *imageType* sont dépréciées, mais ces fonctionnalités sont toujours accessibles via une [compétence personnalisée](./cognitive-search-custom-skill-interface.md).|
| `details` | Tableau de chaînes indiquant les détails spécifiques à un domaine à retourner. Les types de caractéristiques visuelles valides sont les suivants : <ul><li>*celebrities* : identifie les célébrités éventuellement détectées dans l’image.</li><li>*landmarks* : identifie les paysages éventuellement détectés dans l’image. </li></ul> |

## <a name="skill-inputs"></a>Entrées de la compétence

| Nom d’entrée      | Description                                          |
|---------------|------------------------------------------------------|
| `image`         | Type complexe. Ne fonctionne actuellement qu'avec le champ « /documents/normalized_images », généré par l'indexeur d'objets Blob Azure lorsque ```imageAction``` est défini sur une valeur supérieure à ```none```. Pour plus d’informations, consultez [l’exemple](#sample-output).|



##  <a name="sample-skill-definition"></a>Exemple de définition de qualification

```json
        {
            "description": "Extract image analysis.",
            "@odata.type": "#Microsoft.Skills.Vision.ImageAnalysisSkill",
            "context": "/document/normalized_images/*",
            "defaultLanguageCode": "en",
            "visualFeatures": [
                "tags",
                "categories",
                "description",
                "faces",
                "brands"
            ],
            "inputs": [
                {
                    "name": "image",
                    "source": "/document/normalized_images/*"
                }
            ],
            "outputs": [
                {
                    "name": "categories"
                },
                {
                    "name": "tags"
                },
                {
                    "name": "description"
                },
                {
                    "name": "faces"
                },
                {
                    "name": "brands"
                }
            ]
        }
```
### <a name="sample-index-for-only-the-categories-description-faces-and-tags-fields"></a>Exemple d’index (uniquement pour les champs categories, description, faces et tags)
```json
{
    "fields": [
        {
            "name": "id",
            "type": "Edm.String",
            "key": true,
            "searchable": true,
            "filterable": false,
            "facetable": false,
            "sortable": true
        },
        {
            "name": "blob_uri",
            "type": "Edm.String",
            "searchable": true,
            "filterable": false,
            "facetable": false,
            "sortable": true
        },
        {
            "name": "content",
            "type": "Edm.String",
            "sortable": false,
            "searchable": true,
            "filterable": false,
            "facetable": false
        },
        {
            "name": "categories",
            "type": "Collection(Edm.ComplexType)",
            "fields": [
                {
                    "name": "name",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": false,
                    "facetable": false
                },
                {
                    "name": "score",
                    "type": "Edm.Double",
                    "searchable": false,
                    "filterable": false,
                    "facetable": false
                },
                {
                    "name": "detail",
                    "type": "Edm.ComplexType",
                    "fields": [
                        {
                            "name": "celebrities",
                            "type": "Collection(Edm.ComplexType)",
                            "fields": [
                                {
                                    "name": "name",
                                    "type": "Edm.String",
                                    "searchable": true,
                                    "filterable": false,
                                    "facetable": false
                                },
                                {
                                    "name": "faceBoundingBox",
                                    "type": "Collection(Edm.ComplexType)",
                                    "fields": [
                                        {
                                            "name": "x",
                                            "type": "Edm.Int32",
                                            "searchable": false,
                                            "filterable": false,
                                            "facetable": false
                                        },
                                        {
                                            "name": "y",
                                            "type": "Edm.Int32",
                                            "searchable": false,
                                            "filterable": false,
                                            "facetable": false
                                        }
                                    ]
                                },
                                {
                                    "name": "confidence",
                                    "type": "Edm.Double",
                                    "searchable": false,
                                    "filterable": false,
                                    "facetable": false
                                }
                            ]
                        },
                        {
                            "name": "landmarks",
                            "type": "Collection(Edm.ComplexType)",
                            "fields": [
                                {
                                    "name": "name",
                                    "type": "Edm.String",
                                    "searchable": true,
                                    "filterable": false,
                                    "facetable": false
                                },
                                {
                                    "name": "confidence",
                                    "type": "Edm.Double",
                                    "searchable": false,
                                    "filterable": false,
                                    "facetable": false
                                }
                            ]
                        }
                    ]
                }
            ]
        },
        {
            "name": "description",
            "type": "Collection(Edm.ComplexType)",
            "fields": [
                {
                    "name": "tags",
                    "type": "Collection(Edm.String)",
                    "searchable": true,
                    "filterable": false,
                    "facetable": false
                },
                {
                    "name": "captions",
                    "type": "Collection(Edm.ComplexType)",
                    "fields": [
                        {
                            "name": "text",
                            "type": "Edm.String",
                            "searchable": true,
                            "filterable": false,
                            "facetable": false
                        },
                        {
                            "name": "confidence",
                            "type": "Edm.Double",
                            "searchable": false,
                            "filterable": false,
                            "facetable": false
                        }
                    ]
                }
            ]
        },
        {
            "name": "faces",
            "type": "Collection(Edm.ComplexType)",
            "fields": [
                {
                    "name": "age",
                    "type": "Edm.Int32",
                    "searchable": false,
                    "filterable": false,
                    "facetable": false
                },
                {
                    "name": "gender",
                    "type": "Edm.String",
                    "searchable": false,
                    "filterable": false,
                    "facetable": false
                },
                {
                    "name": "faceBoundingBox",
                    "type": "Collection(Edm.ComplexType)",
                    "fields": [
                        {
                            "name": "x",
                            "type": "Edm.Int32",
                            "searchable": false,
                            "filterable": false,
                            "facetable": false
                        },
                        {
                            "name": "y",
                            "type": "Edm.Int32",
                            "searchable": false,
                            "filterable": false,
                            "facetable": false
                        }
                    ]
                }
            ]
        },
        {
            "name": "tags",
            "type": "Collection(Edm.ComplexType)",
            "fields": [
                {
                    "name": "name",
                    "type": "Edm.String",
                    "searchable": true,
                    "filterable": false,
                    "facetable": false
                },
                {
                    "name": "confidence",
                    "type": "Edm.Double",
                    "searchable": false,
                    "filterable": false,
                    "facetable": false
                }
            ]
        }
    ]
}

```
### <a name="sample-output-field-mapping-for-the-above-index"></a>Exemple de mappage de champs de sortie (pour l’index ci-dessus)
```json
    "outputFieldMappings": [
        {
            "sourceFieldName": "/document/normalized_images/*/categories/*",
            "targetFieldName": "categories"
        },
        {
            "sourceFieldName": "/document/normalized_images/*/tags/*",
            "targetFieldName": "tags"
        },
        {
            "sourceFieldName": "/document/normalized_images/*/description",
            "targetFieldName": "description"
        },
        {
            "sourceFieldName": "/document/normalized_images/*/faces/*",
            "targetFieldName": "faces"
        },
        {
            "sourceFieldName": "/document/normalized_images/*/brands/*/name",
            "targetFieldName": "brands"
        }
```
### <a name="variation-on-output-field-mappings-nested-properties"></a>Variation sur les mappages de champs de sortie (propriétés imbriquées)

Vous pouvez définir des mappages de champs de sortie sur des propriétés de niveau inférieur, comme simplement les points de repère ou les célébrités. Dans ce cas, assurez-vous que votre schéma d’index a un champ destiné à contenir spécifiquement les points de repère.

```json
    "outputFieldMappings": [
        {
            "sourceFieldName": "/document/normalized_images/*/categories/detail/celebrities/*",
            "targetFieldName": "celebrities"
        }
```
##  <a name="sample-input"></a>Exemple d’entrée

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "image": {
                    "data": "BASE64 ENCODED STRING OF A JPEG IMAGE",
                    "width": 500,
                    "height": 300,
                    "originalWidth": 5000,
                    "originalHeight": 3000,
                    "rotationFromOriginal": 90,
                    "contentOffset": 500,
                    "pageNumber": 2
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
        "categories": [
          {
            "name": "abstract_",
            "score": 0.00390625
          },
          {
            "name": "people_",
            "score": 0.83984375,
            "detail": {
              "celebrities": [
                {
                  "name": "Satya Nadella",
                  "faceBoundingBox": [
                        {
                            "x": 273,
                            "y": 309
                        },
                        {
                            "x": 395,
                            "y": 309
                        },
                        {
                            "x": 395,
                            "y": 431
                        },
                        {
                            "x": 273,
                            "y": 431
                        }
                    ],
                  "confidence": 0.999028444
                }
              ],
              "landmarks": [
                {
                  "name": "Forbidden City",
                  "confidence": 0.9978346
                }
              ]
            }
          }
        ],
        "adult": {
          "isAdultContent": false,
          "isRacyContent": false,
          "isGoryContent": false,
          "adultScore": 0.0934349000453949,
          "racyScore": 0.068613491952419281,
          "goreScore": 0.08928389008070282
        },
        "tags": [
          {
            "name": "person",
            "confidence": 0.98979085683822632
          },
          {
            "name": "man",
            "confidence": 0.94493889808654785
          },
          {
            "name": "outdoor",
            "confidence": 0.938492476940155
          },
          {
            "name": "window",
            "confidence": 0.89513939619064331
          }
        ],
        "description": {
          "tags": [
            "person",
            "man",
            "outdoor",
            "window",
            "glasses"
          ],
          "captions": [
            {
              "text": "Satya Nadella sitting on a bench",
              "confidence": 0.48293603002174407
            }
          ]
        },
        "requestId": "0dbec5ad-a3d3-4f7e-96b4-dfd57efe967d",
        "metadata": {
          "width": 1500,
          "height": 1000,
          "format": "Jpeg"
        },
        "faces": [
          {
            "age": 44,
            "gender": "Male",
            "faceBoundingBox": [
                {
                    "x": 1601,
                    "y": 395
                },
                {
                    "x": 1653,
                    "y": 395
                },
                {
                    "x": 1653,
                    "y": 447
                },
                {
                    "x": 1601,
                    "y": 447
                }
            ]
          }
        ],
        "objects": [
          {
            "rectangle": {
              "x": 25,
              "y": 43,
              "w": 172,
              "h": 140
            },
            "object": "person",
            "confidence": 0.931
          }
        ],
        "brands":[  
           {  
              "name":"Microsoft",
              "confidence": 0.903,
              "rectangle":{  
                 "x":20,
                 "y":97,
                 "w":62,
                 "h":52
              }
           }
        ]
      }
    }
  ]
}
```


## <a name="error-cases"></a>Cas d’erreur
Dans les cas d’erreur suivants, aucun élément n’est extrait.

| Code d'erreur | Description |
|------------|-------------|
| `NotSupportedLanguage` | La langue fournie n’est pas prise en charge. |
| `InvalidImageUrl` | L’URL de l’image est incorrecte ou inaccessible.|
| `InvalidImageFormat` | Les données d’entrée ne sont pas une image valide. |
| `InvalidImageSize` | L’image d’entrée est trop grande. |
| `NotSupportedVisualFeature`  | Le type de caractéristique spécifié n’est pas valide. |
| `NotSupportedImage` | Image non prise en charge, par exemple, pornographie enfantine. |
| `InvalidDetails` | Modèle spécifique à un domaine non pris en charge. |

Si vous obtenez une erreur similaire à `"One or more skills are invalid. Details: Error in skill #<num>: Outputs are not supported by skill: Landmarks"`, vérifiez le chemin. Les célébrités et les points de repère sont des propriétés sous `detail`.

```json
"categories":[  
      {  
         "name":"building_",
         "score":0.97265625,
         "detail":{  
            "landmarks":[  
               {  
                  "name":"Forbidden City",
                  "confidence":0.92013400793075562
               }
            ]
```

## <a name="see-also"></a>Voir aussi

+ [Compétences prédéfinies](cognitive-search-predefined-skills.md)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
+ [Créer un indexeur (REST)](/rest/api/searchservice/create-indexer)