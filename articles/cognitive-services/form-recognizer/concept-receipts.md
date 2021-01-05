---
title: Tickets de caisse - Form Recognizer
titleSuffix: Azure Cognitive Services
description: Découvrez les concepts liés à l’analyse des tickets de caisse avec l’API Form Recognizer - utilisation et limites.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 08/17/2019
ms.author: pafarley
ms.openlocfilehash: 769dea079339af2c6307d9230e047a654dc3d5dd
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95492208"
---
# <a name="receipt-concepts"></a>Concepts relatifs aux reçus

Form Recognizer peut analyser les accusés de réception à l’aide de l’un de ses modèles prédéfinis. L’API Receipt extrait les informations clés des recettes de vente en anglais, telles que le nom du partenaire, la date de transaction, le total de la transaction, les postes, etc. 

## <a name="understanding-receipts"></a>Comprendre les tickets de caisse 

De nombreuses entreprises et personnes continuent de s’appuyer sur l’extraction manuelle des données à partir de leurs recettes de vente, qu’il s’agisse de rapports sur les dépenses commerciales, de remboursement, d’audit, d’imposition, de budgétisation, de marketing ou autres. Souvent, dans ces scénarios, les images du ticket de caisse physique sont requises à des fins de validation.  

L’extraction automatique de données à partir de ces tickets de caisse peut être compliquée. Les tickets de caisse peuvent être froissés et difficiles à lire, les parties imprimées ou manuscrites et les images des tickets de caisse sur les smartphones peuvent être de mauvaise qualité. En outre, les champs et les modèles de ticket de caisse peuvent varier considérablement selon le marché, la région et le commerçant. Ces défis, tant au niveau de l’extraction des données que de la détection des champs, font du traitement des tickets de caisse un problème unique.  

Grâce à la reconnaissance optique de caractères (OCR) et à notre modèle de ticket de caisse prédéfini, l’API Receipt permet de traiter ces scénarios et d’extraire des données des tickets de caisse, par exemple le nom du commerçant, le pourboire, le total, les postes, etc. Avec cette API, il n’est pas nécessaire de former un modèle ; il suffit d’envoyer le ticket de caisse à l’API Analyze Receipt, et les données sont extraites.

![exemple de ticket de caisse](./media/contoso-receipt-small.png)

## <a name="what-does-the-receipt-api-do"></a>Que fait l’API Receipt ? 

L’API Receipt prédéfinie extrait le contenu des tickets de caisse de vente &mdash; c’est-à-dire le type de ticket de caisse que vous obtiendriez généralement dans un restaurant, un détaillant ou une épicerie.

### <a name="fields-extracted"></a>Champs extraits

* Nom du commerçant 
* Adresse du commerçant 
* Numéro de téléphone du commerçant 
* Date de la transaction 
* Temps de transaction 
* Sous-total 
* Taxe 
* Total 
* Conseil 
* Extraction de poste (par exemple, quantité d’articles, prix de l’article, nom de l’article)

### <a name="additional-features"></a>Fonctionnalités supplémentaires

L’API Receipt renvoie également les informations suivantes :

* Type d’accusé de réception (par exemple, une carte de crédit, etc.)
* Niveau de confiance de champ (chaque champ retourne une valeur de confiance associée)
* Texte brut OCR (sortie texte extraite par OCR pour l’intégralité du ticket de caisse)
* Cadre englobant pour chaque valeur, ligne et mot

## <a name="try-it-out"></a>Faire un essai

Pour tester le service de ticket de caisse de Form Recognizer, accédez à l’outil d’exemple d’interface utilisateur en ligne :

> [!div class="nextstepaction"]
> [Essayer des modèles prédéfinis](https://fott-preview.azurewebsites.net/)

## <a name="input-requirements"></a>Critères des entrées

[!INCLUDE [input reqs](./includes/input-requirements-receipts.md)]

## <a name="supported-locales"></a>Paramètres régionaux pris en charge 

* **Receipt v2.0 intégré** (GA) prend en charge les tickets de caisse de ventes dans les paramètres régionaux EN-US
* **Receipt v2.1-preview.2 intégré** (préversion publique) ajoute une prise en charge supplémentaire pour les paramètres régionaux de ticket de caisse suivants : 
  * EN-AU 
  * EN-CA 
  * EN-GB 
  * EN-IN 

  > [!NOTE]
  > Entrée de langue 
  >
  > Receipt v2.1-preview.2 intégré compte un paramètre de demande facultatif permettant de spécifier les paramètres régionaux de tickets de caisse provenant de marchés anglophones supplémentaires. Pour les tickets de caisse en anglais d’Australie (EN-AU), du Canada (EN-CA), de Grande-Bretagne (EN-GB) et d’Inde (EN-IN), vous pouvez spécifier les paramètres régionaux pour obtenir des résultats améliorés. Si aucun paramètre régional n’est spécifié dans Receipt v2.1-preview.2, le modèle utilise par défaut est le modèle EN-US.


## <a name="the-analyze-receipt-operation"></a>Opération Analyser le ticket de caisse

L’[analyse de ticket de caisse](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/AnalyzeReceiptAsync) prend une image ou un fichier PDF d’un ticket de caisse comme entrée et en extrait les valeurs d’intérêt et le texte. L’appel retourne un champ d’en-tête de réponse appelé `Operation-Location`. La valeur `Operation-Location` est une URL qui contient l’ID de résultat à utiliser à l’étape suivante.

|En-tête de réponse| URL de résultat |
|:-----|:----|
|Operation-Location | `https://cognitiveservice/formrecognizer/v2.0/prebuilt/receipt/analyzeResults/56a36454-fc4d-4354-aa07-880cfbf0064f` |

## <a name="the-get-analyze-receipt-result-operation"></a>Opération obtenir le résultat de l’analyse du ticket de caisse

La seconde étape consiste à appeler l’opération d’[obtention du résultat de l’analyse du ticket de caisse](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/GetAnalyzeReceiptResult). Cette opération prend en entrée l’ID de résultat créé par l’opération d’analyse du ticket de caisse. Elle retourne une réponse JSON qui contient un champ **État** avec les possibles valeurs suivantes. Vous appelez cette opération de façon itérative jusqu’à ce qu’elle retourne avec la valeur **succeeded**. Utilisez un intervalle de 3 à 5 secondes pour éviter de dépasser le taux de demandes par seconde (RPS).

|Champ| Type | Valeurs possibles |
|:-----|:----:|:----|
|status | string | notStarted : L’opération d’analyse n’a pas commencé. |
| |  | running : L’opération d’analyse est en cours. |
| |  | failed : L’opération d’analyse a échoué. |
| |  | succeeded : L’opération d’analyse a réussi. |

Quand le champ d’**état** a la valeur de **réussite**, la réponse JSON inclut les résultats de compréhension de ticket de caisse et de reconnaissance de texte. Le résultat de compréhension de ticket de caisse est organisé sous la forme d’un dictionnaire de valeurs de champ nommé, où chaque valeur contient le texte extrait, la valeur normalisée, le cadre englobant, la confiance et les éléments de mot correspondants. Le résultat de reconnaissance de texte est organisé sous la forme d’une hiérarchie de lignes et de mots, avec du texte, un cadre englobant et des informations de confiance.

![exemples de résultats de ticket de caisse](./media/contoso-receipt-2-information.png)

### <a name="sample-json-output"></a>Exemple de sortir JSON

Voici un exemple de réponse JSON correcte : Le nœud readResults contient tout le texte reconnu. Le texte est organisé par page, puis par ligne, puis par mots individuels. Le nœud documentResults contient les valeurs spécifiques à la carte de visite découvertes par le modèle. C’est là que vous trouverez des paires clé/valeur utiles comme le prénom, le nom, le nom de la société, etc.

```json
{ 
  "status":"succeeded",
  "createdDateTime":"2019-12-17T04:11:24Z",
  "lastUpdatedDateTime":"2019-12-17T04:11:32Z",
  "analyzeResult":{ 
    "version":"2.0.0",
    "readResults":[ 
      { 
        "page":1,
        "angle":0.6893,
        "width":1688,
        "height":3000,
        "unit":"pixel",
        "language":"en",
        "lines":[ 
          { 
            "text":"Contoso",
            "boundingBox":[ 
              635,
              510,
              1086,
              461,
              1098,
              558,
              643,
              604
            ],
            "words":[ 
              { 
                "text":"Contoso",
                "boundingBox":[ 
                  639,
                  510,
                  1087,
                  461,
                  1098,
                  551,
                  646,
                  604
                ],
                "confidence":0.955
              }
            ]
          },
          ...
        ]
      }
    ],
    "documentResults":[ 
      { 
        "docType":"prebuilt:receipt",
        "pageRange":[ 
          1,
          1
        ],
        "fields":{ 
          "ReceiptType":{ 
            "type":"string",
            "valueString":"Itemized",
            "confidence":0.692
          },
          "MerchantName":{ 
            "type":"string",
            "valueString":"Contoso Contoso",
            "text":"Contoso Contoso",
            "boundingBox":[ 
              378.2,
              292.4,
              1117.7,
              468.3,
              1035.7,
              812.7,
              296.3,
              636.8
            ],
            "page":1,
            "confidence":0.613,
            "elements":[ 
              "#/readResults/0/lines/0/words/0",
              "#/readResults/0/lines/1/words/0"
            ]
          },
          "MerchantAddress":{ 
            "type":"string",
            "valueString":"123 Main Street Redmond, WA 98052",
            "text":"123 Main Street Redmond, WA 98052",
            "boundingBox":[ 
              302,
              675.8,
              848.1,
              793.7,
              809.9,
              970.4,
              263.9,
              852.5
            ],
            "page":1,
            "confidence":0.99,
            "elements":[ 
              "#/readResults/0/lines/2/words/0",
              "#/readResults/0/lines/2/words/1",
              "#/readResults/0/lines/2/words/2",
              "#/readResults/0/lines/3/words/0",
              "#/readResults/0/lines/3/words/1",
              "#/readResults/0/lines/3/words/2"
            ]
          },
          "MerchantPhoneNumber":{ 
            "type":"phoneNumber",
            "valuePhoneNumber":"+19876543210",
            "text":"987-654-3210",
            "boundingBox":[ 
              278,
              1004,
              656.3,
              1054.7,
              646.8,
              1125.3,
              268.5,
              1074.7
            ],
            "page":1,
            "confidence":0.99,
            "elements":[ 
              "#/readResults/0/lines/4/words/0"
            ]
          },
          "TransactionDate":{ 
            "type":"date",
            "valueDate":"2019-06-10",
            "text":"6/10/2019",
            "boundingBox":[ 
              265.1,
              1228.4,
              525,
              1247,
              518.9,
              1332.1,
              259,
              1313.5
            ],
            "page":1,
            "confidence":0.99,
            "elements":[ 
              "#/readResults/0/lines/5/words/0"
            ]
          },
          "TransactionTime":{ 
            "type":"time",
            "valueTime":"13:59:00",
            "text":"13:59",
            "boundingBox":[ 
              541,
              1248,
              677.3,
              1261.5,
              668.9,
              1346.5,
              532.6,
              1333
            ],
            "page":1,
            "confidence":0.977,
            "elements":[ 
              "#/readResults/0/lines/5/words/1"
            ]
          },
          "Items":{ 
            "type":"array",
            "valueArray":[ 
              { 
                "type":"object",
                "valueObject":{ 
                  "Quantity":{ 
                    "type":"number",
                    "text":"1",
                    "boundingBox":[ 
                      245.1,
                      1581.5,
                      300.9,
                      1585.1,
                      295,
                      1676,
                      239.2,
                      1672.4
                    ],
                    "page":1,
                    "confidence":0.92,
                    "elements":[ 
                      "#/readResults/0/lines/7/words/0"
                    ]
                  },
                  "Name":{ 
                    "type":"string",
                    "valueString":"Cappuccino",
                    "text":"Cappuccino",
                    "boundingBox":[ 
                      322,
                      1586,
                      654.2,
                      1601.1,
                      650,
                      1693,
                      317.8,
                      1678
                    ],
                    "page":1,
                    "confidence":0.923,
                    "elements":[ 
                      "#/readResults/0/lines/7/words/1"
                    ]
                  },
                  "TotalPrice":{ 
                    "type":"number",
                    "valueNumber":2.2,
                    "text":"$2.20",
                    "boundingBox":[ 
                      1107.7,
                      1584,
                      1263,
                      1574,
                      1268.3,
                      1656,
                      1113,
                      1666
                    ],
                    "page":1,
                    "confidence":0.918,
                    "elements":[ 
                      "#/readResults/0/lines/8/words/0"
                    ]
                  }
                }
              },
              ...
            ]
          },
          "Subtotal":{ 
            "type":"number",
            "valueNumber":11.7,
            "text":"11.70",
            "boundingBox":[ 
              1146,
              2221,
              1297.3,
              2223,
              1296,
              2319,
              1144.7,
              2317
            ],
            "page":1,
            "confidence":0.955,
            "elements":[ 
              "#/readResults/0/lines/13/words/1"
            ]
          },
          "Tax":{ 
            "type":"number",
            "valueNumber":1.17,
            "text":"1.17",
            "boundingBox":[ 
              1190,
              2359,
              1304,
              2359,
              1304,
              2456,
              1190,
              2456
            ],
            "page":1,
            "confidence":0.979,
            "elements":[ 
              "#/readResults/0/lines/15/words/1"
            ]
          },
          "Tip":{ 
            "type":"number",
            "valueNumber":1.63,
            "text":"1.63",
            "boundingBox":[ 
              1094,
              2479,
              1267.7,
              2485,
              1264,
              2591,
              1090.3,
              2585
            ],
            "page":1,
            "confidence":0.941,
            "elements":[ 
              "#/readResults/0/lines/17/words/1"
            ]
          },
          "Total":{ 
            "type":"number",
            "valueNumber":14.5,
            "text":"$14.50",
            "boundingBox":[ 
              1034.2,
              2617,
              1387.5,
              2638.2,
              1380,
              2763,
              1026.7,
              2741.8
            ],
            "page":1,
            "confidence":0.985,
            "elements":[ 
              "#/readResults/0/lines/19/words/0"
            ]
          }
        }
      }
    ]
  }
}
```


## <a name="customer-scenarios"></a>Scénarios clients  

Les données extraites avec l’API Receipt peuvent être utilisées pour effectuer diverses tâches. Voici quelques exemples de ce que nos clients ont accompli avec l’API Receipt. 

### <a name="business-expense-reporting"></a>Notes de frais professionnels  

Souvent, le classement des dépenses commerciales implique de consacrer du temps à la saisie manuelle de données à partir d’images de ticket de caisse. Avec l’API Receipt, vous pouvez utiliser les champs extraits pour automatiser partiellement ce processus et analyser rapidement vos tickets de caisse.  

Étant donné que l’API Receipt a une sortie JSON simple, vous pouvez utiliser les valeurs de champ extraites de plusieurs façons. Intégrez les applications de dépenses internes pour préremplir les notes de frais. Pour en savoir plus sur ce scénario, lisez comment Acumatica utilise l’API Receipt pour [faire de la déclaration de dépenses un processus moins fastidieux](https://customers.microsoft.com/story/762684-acumatica-partner-professional-services-azure).  

### <a name="auditing-and-accounting"></a>Audit et comptabilité 

La sortie de l’API Receipt peut également être utilisée pour effectuer une analyse sur un grand nombre de dépenses à différents stades de la note de frais et du processus de remboursement. Vous pouvez traiter les tickets de caisse pour les trier en vue d’un audit manuel ou d’approbations rapides.  

La sortie de Receipt est également utile pour la gestion générale des livres comptables pour une utilisation professionnelle ou personnelle. Utilisez l’API Receipt pour transformer toute image de réception brute/données PDF en une sortie numérique exploitable.

### <a name="consumer-behavior"></a>Comportement du consommateur 

Les tickets de caisse contiennent des données utiles vous permettant d’analyser le comportement des consommateurs et les tendances d’achat.

L’API Receipt alimente également la [fonctionnalité de traitement de tickets de caisse AIBuilder](/ai-builder/prebuilt-receipt-processing).

## <a name="next-steps"></a>Étapes suivantes

- Suivez un guide de [démarrage rapide : bibliothèque de client Form Recognizer](quickstarts/client-library.md) pour commencer à écrire une application de traitement de tickets de caisse avec Form Recognizer dans le langage de votre choix.
- Ou, suivez le guide de [Démarrage rapide Python de l’API Receipt](./quickstarts/python-receipts.md) pour reconnaître les tickets de caisse à l’aide de l’API REST.

## <a name="see-also"></a>Voir aussi

* [Qu’est-ce que Form Recognizer ?](./overview.md)
* [Documentation de référence sur l’API REST](./index.yml)