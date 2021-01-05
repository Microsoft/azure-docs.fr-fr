---
title: Cartes de visite - Form Recognizer
titleSuffix: Azure Cognitive Services
description: Découvrez les concepts liés à l’analyse des cartes de visite avec l’API Form Recognizer - utilisation et limites.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 08/17/2019
ms.author: pafarley
ms.openlocfilehash: 76e19c36f0fa4096966c5dd23d2384b6b30f2271
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95498858"
---
# <a name="business-card-concepts"></a>Concepts relatifs aux cartes de visite

Azure Form Recognizer peut analyser et extraire des informations de contact à partir de cartes de visite à l’aide de l’un de ses modèles prédéfinis. L’API Carte de visite associe de puissantes fonctionnalités de reconnaissance optique de caractères (OCR) à notre modèle de compréhension de carte de visite pour extraire des informations clés à partir de cartes de visite en anglais. Elle extrait les informations de contact personnel, le nom de la société, la fonction et plus encore. L’API Carte de visite prédéfinie est en disponibilité publique dans la préversion de Form Recognizer v2.1. 

## <a name="what-does-the-business-card-api-do"></a>Quelle est la fonction de l’API Carte de visite ?

L’API Carte de visite extrait les champs clés des cartes de visite et les retourne dans une réponse JSON organisée.

![Image détaillée de Contoso à partir de la sortie JSON + FOTT](./media/business-card-english.jpg)

### <a name="fields-extracted"></a>Champs extraits :

* Noms des contacts 
  * Prénoms
  * Noms de famille
* Noms de société 
* Departments 
* Fonctions 
* E-mails 
* Sites web 
* Adresses 
* Numéros de téléphone 
  * Téléphones mobiles 
  * Télécopies 
  * Téléphones professionnels 
  * Autres téléphones 

L’API Carte de visite peut également retourner tout le texte reconnu de la carte de visite. Cette sortie OCR est incluse dans la réponse JSON.  

### <a name="input-requirements"></a>Spécifications relatives aux entrées 

[!INCLUDE [input reqs](./includes/input-requirements-receipts.md)]

## <a name="the-analyze-business-card-operation"></a>Opération d’analyse de carte de visite

L’opération d’[analyse de carte de visite](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/AnalyzeBusinessCardAsync) prend une image ou un fichier PDF de carte de visite comme entrée et extrait les valeurs présentant un intérêt. L’appel retourne un champ d’en-tête de réponse appelé `Operation-Location`. La valeur `Operation-Location` est une URL qui contient l’ID de résultat à utiliser à l’étape suivante.

|En-tête de réponse| URL de résultat |
|:-----|:----|
|Operation-Location | `https://cognitiveservice/formrecognizer/v2.1-preview.2/prebuilt/businessCard/analyzeResults/49a36324-fc4b-4387-aa06-090cfbf0064f` |

## <a name="the-get-analyze-business-card-result-operation"></a>Opération d’obtention du résultat de l’analyse de la carte de visite

La seconde étape consiste à appeler l’opération d’[obtention du résultat de l’analyse de la carte de visite](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/GetAnalyzeBusinessCardResult). Cette opération prend en entrée l’ID de résultat créé par l’opération d’analyse de carte de visite. Elle retourne une réponse JSON qui contient un champ **État** avec les possibles valeurs suivantes. Vous appelez cette opération de façon itérative jusqu’à ce qu’elle retourne avec la valeur **succeeded**. Utilisez un intervalle de 3 à 5 secondes pour éviter de dépasser le taux de demandes par seconde (RPS).

|Champ| Type | Valeurs possibles |
|:-----|:----:|:----|
|status | string | notStarted : L’opération d’analyse n’a pas commencé.<br /><br />running : L’opération d’analyse est en cours.<br /><br />failed : L’opération d’analyse a échoué.<br /><br />succeeded : L’opération d’analyse a réussi.|

Quand le champ **status** a la valeur de **succeeded**, la réponse JSON inclut les résultats de la compréhension de carte de visite et de la reconnaissance de texte facultative, si nécessaire. Le résultat de compréhension de carte de visite est organisé sous la forme d’un dictionnaire de valeurs de champ nommé, où chaque valeur contient le texte extrait, la valeur normalisée, le cadre englobant, la confiance et les éléments de mot correspondants. Le résultat de reconnaissance de texte est organisé sous la forme d’une hiérarchie de lignes et de mots, avec du texte, un cadre englobant et des informations de confiance.

![Exemple de sortie de carte de visite](./media/business-card-results.png)

### <a name="sample-json-output"></a>Exemple de sortir JSON

Voici un exemple de réponse JSON correcte : Le nœud readResults contient tout le texte reconnu. Le texte est organisé par page, puis par ligne, puis par mots individuels. Le nœud documentResults contient les valeurs spécifiques à la carte de visite découvertes par le modèle. C’est là que vous trouverez des informations de contact utiles comme le prénom, le nom, le nom de la société, etc.

```json
{
    "status": "succeeded",
    "createdDateTime": "2020-08-20T17:41:19Z",
    "lastUpdatedDateTime": "2020-08-20T17:41:24Z",
    "analyzeResult": {
        "version": "2.1.0",
        "readResults": [
            {
                "page": 1,
                "angle": -17.0956,
                "width": 4032,
                "height": 3024,
                "unit": "pixel",
                   "lines": 
                             {
                        "text": "Dr. Avery Smith",
                        "boundingBox": [
                            419.3,
                            1154.6,
                            1589.6,
                            877.9,
                            1618.9,
                            1001.7,
                            448.6,
                            1278.4
                        ],
                        "words": [
                            {
                                "text": "Dr.",
                                "boundingBox": [
                                    419,
                            ]
    
            }
        ],
        "documentResults": [
            {
                "docType": "prebuilt:businesscard",
                "pageRange": [
                    1,
                    1
                ],
                "fields": {
                    "ContactNames": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "object",
                                "valueObject": {
                                    "FirstName": {
                                        "type": "string",
                                        "valueString": "Avery",
                                        "text": "Avery",
                                        "boundingBox": [
                                            703,
                                            1096,
                                            1134,
                                            989,
                                            1165,
                                            1109,
                                            733,
                                            1206
                                        ],
                                        "page": 1
                                    },
                                    "LastName": {
                                        "type": "string",
                                        "valueString": "Smith",
                                        "text": "Smith",
                                        "boundingBox": [
                                            1186,
                                            976,
                                            1585,
                                            879,
                                            1618,
                                            998,
                                            1218,
                                            1096
                                        ],
                                        "page": 1
                                    }
                                },
                                "text": "Dr. Avery Smith",
                                "boundingBox": [
                                    419.3,
                                    1154.6,
                                    1589.6,
                                    877.9,
                                    1618.9,
                                    1001.7,
                                    448.6,
                                    1278.4
                                ],
                                "confidence": 0.97
                            }
                        ]
                    },
                    "JobTitles": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "Senior Researcher",
                                "text": "Senior Researcher",
                                "boundingBox": [
                                    451.8,
                                    1301.9,
                                    1313.5,
                                    1099.9,
                                    1333.8,
                                    1186.7,
                                    472.2,
                                    1388.7
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Departments": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "Cloud & Al Department",
                                "text": "Cloud & Al Department",
                                "boundingBox": [
                                    480.1,
                                    1403.3,
                                    1590.5,
                                    1129.6,
                                    1612.6,
                                    1219.6,
                                    502.3,
                                    1493.3
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Emails": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "avery.smith@contoso.com",
                                "text": "avery.smith@contoso.com",
                                "boundingBox": [
                                    2107,
                                    934,
                                    2917,
                                    696,
                                    2935,
                                    764,
                                    2126,
                                    995
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Websites": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "https://www.contoso.com/",
                                "text": "https://www.contoso.com/",
                                "boundingBox": [
                                    2121,
                                    1002,
                                    2992,
                                    755,
                                    3014,
                                    826,
                                    2143,
                                    1077
                                ],
                                "page": 1,
                                "confidence": 0.995
                            }
                        ]
                    },
                    "MobilePhones": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "phoneNumber",
                                "text": "+44 (0) 7911 123456",
                                "boundingBox": [
                                    2434.9,
                                    1033.3,
                                    3072,
                                    836,
                                    3096.2,
                                    914.3,
                                    2459.1,
                                    1111.6
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "OtherPhones": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "phoneNumber",
                                "text": "+44 (0) 20 9876 5432",
                                "boundingBox": [
                                    2473.2,
                                    1115.4,
                                    3139.2,
                                    907.7,
                                    3163.2,
                                    984.7,
                                    2497.2,
                                    1192.4
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Faxes": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "phoneNumber",
                                "text": "+44 (0) 20 6789 2345",
                                "boundingBox": [
                                    2525,
                                    1185.4,
                                    3192.4,
                                    977.9,
                                    3217.9,
                                    1060,
                                    2550.5,
                                    1267.5
                                ],
                                "page": 1,
                                "confidence": 0.99
                            }
                        ]
                    },
                    "Addresses": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "2 Kingdom Street Paddington, London, W2 6BD",
                                "text": "2 Kingdom Street Paddington, London, W2 6BD",
                                "boundingBox": [
                                    1230,
                                    2138,
                                    2535.2,
                                    1678.6,
                                    2614.2,
                                    1903.1,
                                    1309,
                                    2362.5
                                ],
                                "page": 1,
                                "confidence": 0.977
                            }
                        ]
                    },
                    "CompanyNames": {
                        "type": "array",
                        "valueArray": [
                            {
                                "type": "string",
                                "valueString": "Contoso",
                                "text": "Contoso",
                                "boundingBox": [
                                    1152,
                                    1916,
                                    2293,
                                    1552,
                                    2358,
                                    1733,
                                    1219,
                                    2105
                                ],
                                "page": 1,
                                "confidence": 0.97
                            }
                        ]
                    }
                }
            }
        ]
    }
}
```

Pour implémenter l’extraction de données de carte de visite à l’aide de Python et de l’API REST, suivez le guide de démarrage rapide [Extraire des données de cartes de visite](./QuickStarts/python-business-cards.md).

## <a name="customer-scenarios"></a>Scénarios de client  

Les données extraites avec l’API Carte de visite peuvent être utilisées pour effectuer diverses tâches. L’extraction de ces informations de contact permet de gagner automatiquement du temps à ceux ayant des rôles répondant aux clients. Voici quelques exemples de ce que nos clients ont accompli avec l’API Carte de visite :

* Extraction d’informations de contact des cartes de visite et création rapide de contacts téléphoniques. 
* Intégration à CRM pour créer automatiquement des contacts à l’aide d’images de carte de visite. 
* Suivi des prospects.  
* Extraction d’informations de contact en bloc à partir d’images de carte de visite existantes. 

L’API Carte de visite optimise également la [fonctionnalité de traitement des cartes de visite AI Builder](/ai-builder/prebuilt-business-card).

## <a name="next-steps"></a>Étapes suivantes

- Suivez le [Guide de démarrage rapide Python de l’API des cartes de visite](./quickstarts/python-business-cards.md) pour commencer à reconnaître les cartes de visite.

## <a name="see-also"></a>Voir aussi

* [Qu’est-ce que Form Recognizer ?](./overview.md)
* [Documentation de référence sur l’API REST](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/AnalyzeBusinessCardAsync)