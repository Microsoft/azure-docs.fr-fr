---
title: Définir des projections dans une base de connaissances
titleSuffix: Azure Cognitive Search
description: Exemples de modèles courants montrant comment projeter des documents enrichis dans la base de connaissances pour les utiliser avec Power BI ou Azure ML.
manager: nitinme
author: vkurpad
ms.author: vikurpad
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 3985564d49ce8a5c62b15f9537364418c0a8f5da
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509918"
---
# <a name="how-to-shape-and-export-enrichments"></a>guide pratique pour mettre en forme et exporter des enrichissements

Les projections sont l’expression physique de documents enrichis dans une base de connaissances. L’utilisation efficace de documents enrichis nécessite une structure. Dans cet article, vous allez explorer la structure et les relations, apprendre à créer des propriétés de projection ainsi qu’à lier des données aux différents types de projection créés. 

Pour créer une projection, les données sont mises en forme à l’aide d’une [compétence Modélisateur](cognitive-search-skill-shaper.md) afin de créer un objet personnalisé ou à l’aide de la syntaxe de mise en forme inline dans une définition de projection. 

Une forme de données contient toutes les données devant être projetées, mises en forme en tant que hiérarchie de nœuds. Cet article présente plusieurs techniques pour mettre en forme les données afin de pouvoir ensuite les projeter dans des structures physiques qui permettent la création de rapports, l’analyse ou le traitement en aval. 

Les exemples étudiés dans cet article sont tirés de cet [exemple d’API REST](https://github.com/Azure-Samples/azure-search-postman-samples/blob/master/projections/Projections%20Docs.postman_collection.json), que vous pouvez télécharger et exécuter dans un client HTTP.

## <a name="introduction-to-projection-examples"></a>Présentation des exemples de projection

Il existe trois types de [projections](knowledge-store-projection-overview.md) :

+ Tables
+ Objets
+ Fichiers

Les projections de table sont stockées dans le stockage Table Azure. Les projections d’objet et de fichier sont écrites dans le Stockage Blob, où les projections d’objet sont enregistrées en tant que fichiers JSON, et peuvent contenir le contenu du document source ainsi que les sorties ou enrichissements de compétence. Le pipeline d’enrichissement peut également extraire des binaires tels que des images ; ces binaires sont projetés en tant que projections de fichier. Quand un objet binaire est projeté en tant que projection d’objet, seules les métadonnées qui lui sont associées sont enregistrées en tant qu’objet blob JSON. 

Pour comprendre l’intersection entre la mise en forme des données et les projections, nous allons utiliser l’ensemble de compétences suivant comme base pour l’exploration de différentes configurations. Cet ensemble de compétences traite le contenu d’images et de texte brut. Pour les scénarios souhaités, les projections sont définies à partir du contenu du document et des sorties des compétences.

> [!IMPORTANT] 
> Quand vous expérimentez des projections, il est utile de [définir la propriété de cache de l’indexeur](search-howto-incremental-index.md) pour garantir le contrôle des coûts. La modification des projections entraîne l’enrichissement de la totalité du document si le cache de l’indexeur n’est pas défini. Quand le cache est défini et que seules les projections sont mises à jour, les exécutions de l’ensemble de compétences pour les documents précédemment enrichis n’entraînent pas de frais supplémentaires liés à Cognitive Services.

```json
{
    "name": "azureblob-skillset",
    "description": "Skillset created from the portal. skillsetName: azureblob-skillset; contentField: merged_content; enrichmentGranularity: document; knowledgeStoreStorageAccount: confdemo;",
    "skills": [
        {
            "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
            "name": "#1",
            "description": null,
            "context": "/document/merged_content",
            "categories": [
                "Person",
                "Quantity",
                "Organization",
                "URL",
                "Email",
                "Location",
                "DateTime"
            ],
            "defaultLanguageCode": "en",
            "minimumPrecision": null,
            "includeTypelessEntities": null,
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/merged_content"
                },
                {
                    "name": "languageCode",
                    "source": "/document/language"
                }
            ],
            "outputs": [
                {
                    "name": "persons",
                    "targetName": "people"
                },
                {
                    "name": "organizations",
                    "targetName": "organizations"
                },
                {
                    "name": "locations",
                    "targetName": "locations"
                },
                {
                    "name": "entities",
                    "targetName": "entities"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
            "name": "#2",
            "description": null,
            "context": "/document/merged_content",
            "defaultLanguageCode": "en",
            "maxKeyPhraseCount": null,
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/merged_content"
                },
                {
                    "name": "languageCode",
                    "source": "/document/language"
                }
            ],
            "outputs": [
                {
                    "name": "keyPhrases",
                    "targetName": "keyphrases"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",
            "name": "#3",
            "description": null,
            "context": "/document",
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/merged_content"
                }
            ],
            "outputs": [
                {
                    "name": "languageCode",
                    "targetName": "language"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.MergeSkill",
            "name": "#4",
            "description": null,
            "context": "/document",
            "insertPreTag": " ",
            "insertPostTag": " ",
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/content"
                },
                {
                    "name": "itemsToInsert",
                    "source": "/document/normalized_images/*/text"
                },
                {
                    "name": "offsets",
                    "source": "/document/normalized_images/*/contentOffset"
                }
            ],
            "outputs": [
                {
                    "name": "mergedText",
                    "targetName": "merged_content"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
            "name": "#5",
            "description": null,
            "context": "/document/normalized_images/*",
            "textExtractionAlgorithm": "printed",
            "lineEnding": "Space",
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
                    "name": "text",
                    "targetName": "text"
                },
                {
                    "name": "layoutText",
                    "targetName": "layoutText"
                }
            ]
        }
    ],
    "cognitiveServices": {
        "@odata.type": "#Microsoft.Azure.Search.CognitiveServicesByKey",
        "description": "DemosCS",
        "key": "<COGNITIVE SERVICES KEY>"
    },
    "knowledgeStore": null
}
```

À partir de cet ensemble de compétences, avec la valeur `knowledgeStore` Null comme base, notre premier exemple remplit l’objet `knowledgeStore`, configuré avec des projections qui créent des structures de données tabulaires utilisables dans d’autres scénarios. 

## <a name="projecting-to-tables"></a>Projection dans des tables

La projection dans des tables dans le Stockage Azure est utile pour la création de rapports et l’analyse des données à l’aide d’outils comme Power BI. Power BI peut lire le contenu des tables et découvrir les relations d’après les clés qui sont générées pendant la projection. Si vous voulez créer un tableau de bord, cette tâche sera plus simple si les données sont liées entre elles. 

Créons un tableau de bord dans lequel il est possible de visualiser les expressions clés extraites de documents en tant que nuage de mots clés. Pour créer la structure de données appropriée, ajoutez une compétence Modélisateur à l’ensemble de compétences en vue de créer une forme personnalisée avec les détails et les expressions clés spécifiques du document. La forme personnalisée est appelée `pbiShape` sur le nœud racine `document`.

> [!NOTE] 
> Les projections de table sont des tables du Stockage Azure auxquelles s’appliquent les limites de stockage imposées par le Stockage Azure. Pour plus d’informations, consultez les [limites du stockage de table](/rest/api/storageservices/understanding-the-table-service-data-model). Il est utile de savoir que la taille maximale respective d’une entité et d’une propriété est de 1 Mo et 64 Ko. Ces contraintes font des tables une bonne solution pour stocker un grand nombre de petites entités.

### <a name="using-a-shaper-skill-to-create-a-custom-shape"></a>Utilisation d’une compétence Modélisateur pour créer une forme personnalisée

Créez une forme personnalisée que vous pouvez projeter dans le stockage de table. Sans forme personnalisée, une projection ne peut référencer qu’un seul nœud (une projection par sortie). La création d’une forme personnalisée permet de regrouper différents éléments en un nouvel ensemble logique qui peut être projeté sous la forme d’une table unique, ou découpé et distribué dans une collection de tables. 

Dans cet exemple, la forme personnalisée combine les métadonnées et les entités et expressions clés identifiées. L’objet est appelé `pbiShape` et est apparenté à `/document`. 

> [!IMPORTANT] 
> L’une des finalités de la mise en forme est de s’assurer que tous les nœuds d’enrichissement sont exprimés dans un format JSON correct, ce qui est obligatoire pour la projection dans une base de connaissances. Cela est particulièrement vrai quand une arborescence d’enrichissement contient des nœuds qui n’ont pas le bon format JSON (par exemple, si un enrichissement est apparenté à une primitive comme une chaîne).
>
> Notez que les deux derniers nœuds, `KeyPhrases` et `Entities`, sont wrappés dans un objet JSON valide avec le `sourceContext`. Cela est nécessaire, car `keyphrases` et `entities` sont des enrichissements sur les primitives et ils doivent être convertis en JSON valide avant de pouvoir être projetés.
>


```json
{
    "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
    "name": "ShaperForTables",
    "description": null,
    "context": "/document",
    "inputs": [
        {
            "name": "metadata_storage_content_type",
            "source": "/document/metadata_storage_content_type",
            "sourceContext": null,
            "inputs": []
        },
        {
            "name": "metadata_storage_name",
            "source": "/document/metadata_storage_name",
            "sourceContext": null,
            "inputs": []
        },
        {
            "name": "metadata_storage_path",
            "source": "/document/metadata_storage_path",
            "sourceContext": null,
            "inputs": []
        },
        {
            "name": "metadata_content_type",
            "source": "/document/metadata_content_type",
            "sourceContext": null,
            "inputs": []
        },
        {
            "name": "keyPhrases",
            "source": null,
            "sourceContext": "/document/merged_content/keyphrases/*",
            "inputs": [
                {
                    "name": "KeyPhrases",
                    "source": "/document/merged_content/keyphrases/*"
                }

            ]
        },
        {
            "name": "Entities",
            "source": null,
            "sourceContext": "/document/merged_content/entities/*",
            "inputs": [
                {
                    "name": "Entities",
                    "source": "/document/merged_content/entities/*/name"
                }

            ]
        }
    ],
    "outputs": [
        {
            "name": "output",
            "targetName": "pbiShape"
        }
    ]
}
```

Ajoutez la compétence Modélisateur à l’ensemble de compétences. 

```json
    "name": "azureblob-skillset",
    "description": "A friendly description of the skillset goes here.",
    "skills": [
        {
            Shaper skill goes here
            }
        ],
    "cognitiveServices":  "A key goes here",
    "knowledgeStore": []
}  
```

Toutes les données nécessaires à la projection dans des tables étant désormais disponibles, mettez à jour l’objet knowledgeStore avec les définitions des tables. Dans cet exemple, nous avons trois tables, définies par les propriétés `tableName`, `source` et `generatedKeyName`.

```json
"knowledgeStore" : {
    "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
    "projections": [
        {
            "tables": [
                {
                    "tableName": "pbiDocument",
                    "generatedKeyName": "Documentid",
                    "source": "/document/pbiShape"
                },
                {
                    "tableName": "pbiKeyPhrases",
                    "generatedKeyName": "KeyPhraseid",
                    "source": "/document/pbiShape/keyPhrases/*"
                },
                {
                    "tableName": "pbiEntities",
                    "generatedKeyName": "Entityid",
                    "source": "/document/pbiShape/Entities/*"
                }
            ],
            "objects": [],
            "files": []
        }
    ]
}
```

Vous pouvez effectuer votre travail en procédant comme suit :

1. Définissez la propriété ```storageConnectionString``` sur une chaîne de connexion de compte de stockage v2 universel valide.  

1. Mettez à jour l’ensemble de compétences en envoyant la requête PUT.

1. Après la mise à jour de l’ensemble de compétences, exécutez l’indexeur. 

Vous disposez maintenant d’une projection de travail avec trois tables. Quand vous importez ces tables dans Power BI, celui-ci doit être en mesure de détecter les relations automatiquement.

Avant de passer à l’exemple suivant, revoyons les aspects de la projection de table pour comprendre les mécanismes de découpage et des données associées.

### <a name="slicing"></a>Découpage 

Le découpage est une technique qui divise une forme regroupée entière en plusieurs parties constitutives. Vous obtenez des tables distinctes mais liées entre elles, que vous pouvez utiliser individuellement.

Dans l’exemple, `pbiShape` est la forme regroupée (ou nœud d’enrichissement). Dans la définition de la projection, `pbiShape` est découpé en tables supplémentaires, ce qui vous permet d’extraire des parties de la forme, ```keyPhrases``` et ```Entities```. Dans Power BI, cette opération est utile lorsque plusieurs entités et expressions clés sont associées à chaque document : vous obtenez des insights plus complets si vous pouvez visualiser les entités et les expressions clés sous forme de données classifiées par catégorie.

Le découpage génère implicitement une relation entre les tables parent et enfant, en utilisant le ```generatedKeyName``` dans la table parent pour créer une colonne du même nom dans la table enfant. 

### <a name="naming-relationships"></a>Nommage des relations

Les propriétés ```generatedKeyName``` et ```referenceKeyName``` sont utilisées pour lier des données entre des tables ou même entre des types de projection. Chaque ligne de la table enfant/projection a une propriété qui pointe vers le parent. Le nom de la colonne ou de la propriété dans l’enfant est le ```referenceKeyName``` du parent. Quand le ```referenceKeyName``` n’est pas fourni, le service le définit par défaut sur le ```generatedKeyName``` du parent. 

Power BI utilise ces clés générées pour découvrir les relations dans les tables. S’il est nécessaire que la colonne de la table enfant soit nommée différemment, définissez la propriété ```referenceKeyName``` sur la table . Par exemple, vous pouvez définir le ```generatedKeyName``` en tant qu’ID sur la table pbiDocument et le ```referenceKeyName``` en tant que DocumentID. Ainsi, la colonne dans les tables pbiEntities et pbiKeyPhrases contenant l’ID de document seraient nommées DocumentID.

## <a name="projecting-to-objects"></a>Projection dans des objets

Les projections d’objet n’ont pas les mêmes limitations que les projections de table et elles sont mieux adaptées pour la projection de documents volumineux. Dans cet exemple, la totalité du document est envoyée en tant que projection d’objet. Les projections d’objet sont limitées à une seule projection dans un conteneur et elles ne peuvent pas être découpées.

Pour définir une projection d’objet, utilisez le tableau ```objects``` dans les projections. Vous pouvez générer une nouvelle forme à l’aide de la compétence Modélisateur ou utiliser la mise en forme inline de la projection d’objet. Tandis que l’exemple de tables illustre l’approche de la création d’une forme et du découpage, cet exemple illustre l’utilisation de la mise en forme inline. 

La mise en forme inline permet de créer une forme dans la définition des entrées d’une projection. La mise en forme inline crée un objet anonyme identique à ce qu’une compétence Modélisateur produirait (ici, `pbiShape`). La mise en forme inline est utile si vous définissez une forme que vous n’envisagez pas de réutiliser.

La propriété projections est un tableau. Cet exemple comprend une nouvelle instance de projection au tableau, où la définition knowledgeStore contient des projections inline. Quand vous utilisez des projections inline, vous pouvez omettre la compétence Modélisateur.

```json
"knowledgeStore" : {
        "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
        "projections": [
             {
                "tables": [ ],
                "objects": [
                    {
                        "storageContainer": "sampleobject",
                        "source": null,
                        "generatedKeyName": "myobject",
                        "sourceContext": "/document",
                        "inputs": [
                            {
                                "name": "metadata_storage_name",
                                "source": "/document/metadata_storage_name"
                            },
                            {
                                "name": "metadata_storage_path",
                                "source": "/document/metadata_storage_path"
                            },
                            {
                                "name": "content",
                                "source": "/document/content"
                            },
                            {
                                "name": "keyPhrases",
                                "source": "/document/merged_content/keyphrases/*"
                            },
                            {
                                "name": "entities",
                                "source": "/document/merged_content/entities/*/name"
                            },
                            {
                                "name": "ocrText",
                                "source": "/document/normalized_images/*/text"
                            },
                            {
                                "name": "ocrLayoutText",
                                "source": "/document/normalized_images/*/layoutText"
                            }
                        ]

                    }
                ],
                "files": []
            }
        ]
    }
```

## <a name="projecting-to-file"></a>Projection dans un fichier

Les projections de fichier sont des images extraites du document source ou des sorties d’enrichissement qui peuvent être projetées à partir du processus d’enrichissement. Les projections de fichier, comme les projections d’objet, sont implémentées sous forme d’objets blob dans le Stockage Azure et contiennent l’image. 

Pour générer une projection de fichier, utilisez le tableau `files` dans l’objet de projection. Cet exemple projette toutes les images extraites du document dans un conteneur appelé `samplefile`.

```json
"knowledgeStore" : {
        "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
        "projections": [
            {
                "tables": [ ],
                "objects": [ ],
                "files": [
                    {
                        "storageContainer": "samplefile",
                        "source": "/document/normalized_images/*"
                    }
                ]
            }
        ]
    }
```

## <a name="projecting-to-multiple-types"></a>Projection dans plusieurs types

Un scénario plus complexe peut vous obliger à projeter du contenu dans différents types de projection. Par exemple, si vous devez projeter des données telles que des expressions clés et des entités dans des tables, enregistrez les résultats OCR du texte et du texte de disposition en page sous forme d’objets, puis projetez les images en tant que fichiers. 

Cet exemple comprend les mises à jour apportées à l’ensemble de compétences avec les modifications suivantes :

1. Créer une table avec une ligne pour chaque document.
1. Créer une table liée à la table de documents avec chaque expression clé identifiée en tant que ligne dans cette table.
1. Créer une table liée à la table de documents avec chaque entité identifiée en tant que ligne dans cette table.
1. Créer une projection d’objet avec le texte de disposition pour chaque image.
1. Créez une projection de fichier, en projetant chaque image extraite.
1. Créer une table de référence croisée qui contient des références à la table de document, à la projection d’objet avec le texte de disposition et à la projection de fichier.

Ces modifications sont répercutées plus loin dans la définition knowledgeStore. 

### <a name="shape-data-for-cross-projection"></a>Mettre en forme les données pour la projection croisée

Pour obtenir les formes nécessaires pour ces projections, commencez par ajouter une nouvelle compétence Modélisateur qui crée un objet mis en forme appelé `crossProjection`. 

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
    "name": "ShaperForCross",
    "description": null,
    "context": "/document",
    "inputs": [
        {
            "name": "metadata_storage_name",
            "source": "/document/metadata_storage_name",
            "sourceContext": null,
            "inputs": []
        },
        {
            "name": "keyPhrases",
            "source": null,
            "sourceContext": "/document/merged_content/keyphrases/*",
            "inputs": [
                {
                    "name": "KeyPhrases",
                    "source": "/document/merged_content/keyphrases/*"
                }

            ]
        },
        {
            "name": "entities",
            "source": null,
            "sourceContext": "/document/merged_content/entities/*",
            "inputs": [
                {
                    "name": "Entities",
                    "source": "/document/merged_content/entities/*/name"
                }

            ]
        },
        {
            "name": "images",
            "source": null,
            "sourceContext": "/document/normalized_images/*",
            "inputs": [
                {
                    "name": "image",
                    "source": "/document/normalized_images/*"
                },
                {
                    "name": "layoutText",
                    "source": "/document/normalized_images/*/layoutText"
                },
                {
                    "name": "ocrText",
                    "source": "/document/normalized_images/*/text"
                }
                ]
        }
 
    ],
    "outputs": [
        {
            "name": "output",
            "targetName": "crossProjection"
        }
    ]
}
```

### <a name="define-table-object-and-file-projections"></a>Définir des projections de table, d’objet et de fichier

À partir de l’objet crossProjection regroupé, découpez l’objet en plusieurs tables, capturez la sortie OCR sous forme d’objets blob, puis enregistrez l’image en tant que fichiers (également dans le Stockage Blob).

```json
"knowledgeStore" : {
        "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
        "projections": [
             {
                "tables": [
                    {
                        "tableName": "crossDocument",
                        "generatedKeyName": "Id",
                        "source": "/document/crossProjection"
                    },
                    {
                        "tableName": "crossEntities",
                        "generatedKeyName": "EntityId",
                        "source": "/document/crossProjection/entities/*"
                    },
                    {
                        "tableName": "crossKeyPhrases",
                        "generatedKeyName": "KeyPhraseId",
                        "source": "/document/crossProjection/keyPhrases/*"
                    },
                    {
                        "tableName": "crossReference",
                        "generatedKeyName": "CrossId",
                        "source": "/document/crossProjection/images/*"
                    }
                     
                ],
                "objects": [
                    {
                        "storageContainer": "crossobject",
                        "generatedKeyName": "crosslayout",
                        "source": null,
                        "sourceContext": "/document/crossProjection/images/*/layoutText",
                        "inputs": [
                            {
                                "name": "OcrLayoutText",
                                "source": "/document/crossProjection/images/*/layoutText"
                            }
                        ]
                    }
                ],
                "files": [
                    {
                        "storageContainer": "crossimages",
                        "generatedKeyName": "crossimages",
                        "source": "/document/crossProjection/images/*/image"
                    }
                ]
            }
        ]
    }
```

Les projections d’objet nécessitent un nom de conteneur pour chaque projection ; les projections d’objet ou les projections de fichier ne peuvent pas partager de conteneur. 

### <a name="relationships-among-table-object-and-file-projections"></a>Relations entre les projections de table, d’objet et de fichier

Cet exemple met également en lumière une autre fonctionnalité des projections. En définissant plusieurs types de projections dans le même objet de projection, il existe une relation exprimée dans et entre les différents types (tables, objets, fichiers). Cela vous permet de commencer avec une ligne de tableau pour un document et de rechercher tout le texte OCR pour les images contenues dans ce document dans la projection de l’objet. 

Si vous ne souhaitez pas que les données soient liées, définissez les projections dans différents objets de projection. Par exemple, l’extrait de code suivant entraîne la liaison des tables, sans qu’aucune relation ne soit toutefois établie entre les tables et les projections d’objet (texte OCR). 

Les groupes de projection sont utiles quand vous souhaitez projeter les mêmes données dans différentes formes en fonction de besoins différents. Par exemple, un groupe de projections pour le tableau de bord Power BI et un autre groupe de projections pour la capture des données utilisées pour entraîner un modèle Machine Learning wrappé dans une compétence personnalisée.

Lors de la création de projections de types différents, les projections de fichier et d’objet sont générées en premier, et les chemins sont ajoutés aux tables.

```json
"knowledgeStore" : {
        "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
        "projections": [
            {
                "tables": [
                    {
                        "tableName": "unrelatedDocument",
                        "generatedKeyName": "Documentid",
                        "source": "/document/pbiShape"
                    },
                    {
                        "tableName": "unrelatedKeyPhrases",
                        "generatedKeyName": "KeyPhraseid",
                        "source": "/document/pbiShape/keyPhrases"
                    }
                ],
                "objects": [
                    
                ],
                "files": []
            }, 
            {
                "tables": [],
                "objects": [
                    {
                        "storageContainer": "unrelatedocrtext",
                        "source": null,
                        "sourceContext": "/document/normalized_images/*/text",
                        "inputs": [
                            {
                                "name": "ocrText",
                                "source": "/document/normalized_images/*/text"
                            }
                        ]
                    },
                    {
                        "storageContainer": "unrelatedocrlayout",
                        "source": null,
                        "sourceContext": "/document/normalized_images/*/layoutText",
                        "inputs": [
                            {
                                "name": "ocrLayoutText",
                                "source": "/document/normalized_images/*/layoutText"
                            }
                        ]
                    }
                ],
                "files": []
            }
        ]
    }
```

## <a name="common-issues"></a>Problèmes courants

Lors de la définition d’une projection, il existe quelques problèmes courants qui peuvent entraîner des résultats imprévus. Vérifiez les points ci-dessous si la sortie dans la base de connaissances ne correspond pas à la sortie prévue.

+ Absence de la mise en forme des enrichissements de chaînes dans un JSON valide. Quand des chaînes sont enrichies, par exemple la chaîne `merged_content` enrichie d’expressions clés, la propriété enrichie est représentée comme enfant de `merged_content` dans l’arborescence des enrichissements. La représentation par défaut n’est pas un JSON correctement mis en forme. Au moment de la projection, vous devez donc transformer l’enrichissement en un objet JSON valide avec un nom et une valeur.

+ Omission de ```/*``` à la fin d’un chemin source. Si la source d’une projection est `/document/pbiShape/keyPhrases`, le tableau d’expressions clés est projeté sous la forme d’un objet ou d’une ligne unique. À la place, définissez le chemin source sur `/document/pbiShape/keyPhrases/*` afin d’obtenir une seule ligne ou un seul objet pour chacune des expressions clés.

+ Erreurs dans la syntaxe du chemin. Les sélecteurs de chemin respectent la casse et peuvent entraîner des avertissements d’entrée manquante si vous n’utilisez pas la casse exacte du sélecteur.

## <a name="next-steps"></a>Étapes suivantes

Les exemples de cet article montrent des modèles courants de création de projections. Maintenant que vous avez bien compris les concepts, vous êtes mieux armé pour créer des projections adaptées à votre scénario particulier.

À mesure que vous explorez de nouvelles fonctionnalités, envisagez l’enrichissement incrémentiel en guise d’étape suivante. L’enrichissement incrémentiel est basé sur la mise en cache, ce qui vous permet de réutiliser tous les enrichissements qui ne sont pas impactés par une modification de l’ensemble de compétences. Cela s’avère particulièrement utile pour les pipelines qui incluent l’OCR et l’analyse des images.

> [!div class="nextstepaction"]
> [Présentation de l’enrichissement incrémentiel et de la mise en cache](cognitive-search-incremental-indexing-conceptual.md)

Pour obtenir une vue d’ensemble des projections, découvrez les fonctionnalités telles que les groupes et le découpage, et comment vous [les définissez dans un ensemble de compétences](knowledge-store-projection-overview.md).

> [!div class="nextstepaction"]
> [Projections dans une base de connaissances](knowledge-store-projection-overview.md)