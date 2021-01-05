---
title: Comment utiliser l’Analyse de texte pour la santé
titleSuffix: Azure Cognitive Services
description: Découvrez comment extraire et étiqueter des informations médicales de texte clinique non structuré avec l’Analyse de texte pour la santé.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 12/02/2020
ms.author: aahi
ms.custom: references_regions
ms.openlocfilehash: 27af0ffdeb254dbb671a0618d0e9973336a7f529
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96559012"
---
# <a name="how-to-use-text-analytics-for-health-preview"></a>Procédure : Utiliser l’Analyse de texte pour la santé (préversion)

> [!IMPORTANT] 
> L’Analyse de texte pour l’intégrité est une fonctionnalité en version préliminaire fournie « en l’état » et « avec toutes les erreurs ». Par conséquent, **l’Analyse de texte pour l’intégrité (préversion) ne doit pas être implémentée ou déployée dans le cadre d’une utilisation en production.** L’Analyse de texte pour l’intégrité n’est pas destinée à être utilisée en tant que dispositif médical, support clinique, outil de diagnostic ou autre technologie destinée à être utilisée dans le diagnostic, la guérison, l’atténuation, le traitement ou la prévention de maladies ou d’autres conditions, et aucune licence ou droit n’est accordé par Microsoft pour utiliser cette fonctionnalité à ces fins. Cette fonctionnalité n’est pas conçue ou destinée à être mise en œuvre ou déployée en remplacement de conseils médicaux professionnels ou d’avis de santé, de diagnostic, de traitement ou de jugement clinique d’un professionnel de la santé, et ne doit pas être utilisé en tant que tel. Le client est seul responsable de l’utilisation de l’Analyse de texte pour l’intégrité. Microsoft ne garantit pas que l’Analyse de texte pour l’intégrité ou les ressources fournies dans le cadre de la fonctionnalité sont suffisantes pour des raisons médicales ou répondent aux exigences médicales de toute personne. 


Analyse de texte pour la santé est une fonctionnalité du service de l’API Analyse de texte qui extrait et étiquette des informations médicales pertinentes à partir de textes non structurés, tels que les notes du médecin, les bilans de sortie d’hospitalisation, les documents cliniques et les dossiers médicaux électroniques.  Il existe deux façons d’utiliser ce service : 

* API web (asynchrone) 
* Conteneur Docker (synchrone)   

> [!VIDEO https://channel9.msdn.com/Shows/AI-Show/Introducing-Text-Analytics-for-Health/player]

## <a name="features"></a>Fonctionnalités

Analyse de texte pour la santé effectue une reconnaissance d’entité nommée (NER), une extraction de relations, une négation d’entité et une liaison d’entité sur du texte en anglais pour découvrir des insights dans du texte clinique et biomédical non structuré.

### <a name="named-entity-recognition"></a>[Reconnaissance d’entité nommée](#tab/ner)

La Reconnaissance d’entité nommée détecte les mots et les expressions mentionnés dans du texte non structuré qui peuvent être associés à un ou plusieurs types sémantiques, tels que le diagnostic, le nom de médicaments, le symptôme/le signe ou l’âge.

> [!div class="mx-imgBorder"]
> ![NER d’intégrité](../media/ta-for-health/health-named-entity-recognition.png)

### <a name="relation-extraction"></a>[Extraction de relations](#tab/relation-extraction)

L’extraction de relations identifie les connexions pertinentes entre les concepts mentionnés dans le texte. Par exemple, une relation « heure de maladie » est trouvée en associant un nom de pathologie à une indication temporelle. 

> [!div class="mx-imgBorder"]
> ![RE d’intégrité](../media/ta-for-health/health-relation-extraction.png)


### <a name="entity-linking"></a>[Entity Linking](#tab/entity-linking)

La liaison d’entités distingue des entités distinctes en associant des entités nommées mentionnées dans du texte à des concepts trouvés dans une base de données de concepts prédéfinie. Par exemple, le système de langage médical unifié (UMLS).

> [!div class="mx-imgBorder"]
> ![EL d’intégrité](../media/ta-for-health/health-entity-linking.png)

L’Analyse de texte pour la santé prend en charge la liaison avec le vocabulaire médical et biomédical figurant dans la source de connaissances Metathesaurus de l’[UMLS](https://www.nlm.nih.gov/research/umls/sourcereleasedocs/index.html) (Unified Medical Language System).

### <a name="negation-detection"></a>[Détection de négation](#tab/negation-detection) 

La signification du contenu médical est fortement affectée par les modificateurs, tels que la négation, qui peuvent avoir une incidence critique en cas de mauvais diagnostic. L’Analyse de texte pour la santé prend en charge la détection de négation pour les différentes entités mentionnées dans le texte. 

> [!div class="mx-imgBorder"]
> ![NEG d’intégrité](../media/ta-for-health/health-negation.png)

---

Consultez les [catégories d’entité](../named-entity-types.md?tabs=health) retournées par l’Analyse de texte pour la santé pour obtenir un liste complète des entités prises en charge.

### <a name="supported-languages-and-regions"></a>Langues et régions prises en charge

L’Analyse de texte pour la santé prend uniquement en charge les documents en langue anglaise. 

L’API web hébergée Analyse de texte pour la santé est actuellement disponible uniquement dans les régions suivantes : USA Ouest 2, USA Est 2, USA Centre, Europe Nord et Europe Ouest.

## <a name="request-access-to-the-public-preview"></a>Demander accès à la préversion publique

Renseignez et envoyez le [formulaire de demande Cognitive Services](https://aka.ms/csgate) pour demander l’accès à la préversion publique d’Analyse de texte pour la santé. Vous ne serez pas facturé pour l’utilisation de l’Analyse de texte pour la santé. 

Le formulaire demande des informations sur vous, votre entreprise et le scénario d’utilisateur pour lequel vous allez utiliser le conteneur. Une fois le formulaire envoyé, l’équipe Azure Cognitive Services Azure l’examinera et vous informera de sa décision par courrier électronique.

> [!IMPORTANT]
> * Dans le formulaire, vous devez utiliser une adresse e-mail associée à un ID d’abonnement Azure.
> * La ressource Azure que vous utilisez doit avoir été créée avec l’ID d’abonnement Azure approuvé. 
> * Vérifiez votre adresse e-mail (boîtes de réception et dossiers de courrier indésirable) pour obtenir des mises à jour sur l’état de votre application auprès de Microsoft.

## <a name="using-the-docker-container"></a>Utilisation du conteneur Docker 

Pour exécuter le conteneur Analyse de texte pour la santé dans votre propre environnement, suivez ces [instructions pour télécharger et installer le conteneur](../how-tos/text-analytics-how-to-install-containers.md?tabs=healthcare).

## <a name="using-the-client-library"></a>Utilisation de la bibliothèque de client

La dernière préversion de la bibliothèque de client Analyse de texte vous permet d’appeler Analyse de texte pour la santé en utilisant un objet client. Reportez-vous à la documentation de référence et consultez les exemples sur GitHub :
* [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics)
* [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/)
* [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics)



## <a name="sending-a-rest-api-request"></a>Envoie d’une requête d’API REST 

### <a name="preparation"></a>Préparation

La qualité des résultats de la fonctionnalité Analyse de texte pour la santé est d’autant meilleure que vous lui donnez de petites quantités de texte à analyser. C’est l’inverse de certaines autres fonctionnalités Analyse de texte comme l’extraction d’expressions clés qui donne de meilleurs résultats avec des blocs de texte plus volumineux. Pour obtenir des résultats optimaux pour ces opérations, envisagez de restructurer les entrées en conséquence.

Vous devez disposer des documents JSON dans ce format : ID, texte et langue. 

La taille du document doit être inférieure à 5 120 caractères par document. Pour connaître le nombre maximal de documents autorisés dans une collection, consultez l’article [Limites de données](../concepts/data-limits.md?tabs=version-3) sous Concepts. La collection est soumise dans le corps de la demande.

### <a name="structure-the-api-request-for-the-hosted-asynchronous-web-api"></a>Structurer la requête d’API pour l’API web asynchrone hébergée

Pour le conteneur et l’API web hébergée, vous devez créer une requête POST. Vous pouvez [utiliser Postman](text-analytics-how-to-call-api.md), une commande cURL ou la **console de test d’API** indiquée dans les [informations de référence sur l’API hébergée Analyse de texte pour la santé](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/Health) pour créer et envoyer rapidement une requête POST à l’API web hébergée dans la région de votre choix. 

Voici un exemple de fichier JSON joint au corps POST de la requête d’API Analyse de texte pour la santé :

```json
example.json

{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Subject was administered 100mg remdesivir intravenously over a period of 120 min"
    }
  ]
}
```

### <a name="hosted-asynchronous-web-api-response"></a>Réponse de l’API web asynchrone hébergée 

Étant donné que cette requête POST est utilisée pour envoyer un travail pour l’opération asynchrone, il n’y a pas de texte dans l’objet de la réponse.  Toutefois, vous avez besoin de la valeur operation-location KEY dans les en-têtes de réponse pour effectuer une requête GET afin de vérifier l’état du travail et de la sortie.  Voici un exemple de valeur operation-location KEY dans l’en-tête de réponse de la requête POST :

`https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.3/entities/health/jobs/<jobID>`

Pour vérifier l’état du travail, faites une requête GET à l’URL dans la valeur de l’en-tête operation-location KEY de la réponse POST.  Les états suivants sont utilisés pour refléter l’état d’un travail : `NotStarted`, `running`, `succeeded`, `failed`, `rejected`, `cancelling` et `cancelled`.  

Vous pouvez annuler un travail dont l’état est `NotStarted` ou `running` avec un appel DELETE HTTP à la même URL que la requête GET.  Pour plus d’informations sur l’appel DELETE, consultez les [informations de référence sur l’API hébergée Analyse de texte pour la santé](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/CancelHealthJob).

Voici un exemple de réponse d’une requête GET.  Notez que la sortie est disponible et récupérable jusqu’à ce que le délai `expirationDateTime` (24 heures à partir de l’heure de création du travail) soit écoulé, après quoi la sortie est purgée.

```json
{
    "jobId": "b672c6f5-7c0d-4783-ba8c-4d0c47213454",
    "lastUpdateDateTime": "2020-11-18T01:45:00Z",
    "createdDateTime": "2020-11-18T01:44:55Z",
    "expirationDateTime": "2020-11-19T01:44:55Z",
    "status": "succeeded",
    "errors": [],
    "results": {
        "documents": [
            {
                "id": "1",
                "entities": [
                    {
                        "offset": 25,
                        "length": 5,
                        "text": "100mg",
                        "category": "Dosage",
                        "confidenceScore": 1.0,
                        "isNegated": false
                    },
                    {
                        "offset": 31,
                        "length": 10,
                        "text": "remdesivir",
                        "category": "MedicationName",
                        "confidenceScore": 1.0,
                        "isNegated": false,
                        "links": [
                            {
                                "dataSource": "UMLS",
                                "id": "C4726677"
                            },
                            {
                                "dataSource": "MSH",
                                "id": "C000606551"
                            },
                            {
                                "dataSource": "NCI",
                                "id": "C152185"
                            },
                            {
                                "dataSource": "NCI_FDA",
                                "id": "3QKI37EEHE"
                            }
                        ]
                    },
                    {
                        "offset": 42,
                        "length": 13,
                        "text": "intravenously",
                        "category": "MedicationRoute",
                        "confidenceScore": 1.0,
                        "isNegated": false
                    },
                    {
                        "offset": 56,
                        "length": 4,
                        "text": "over",
                        "category": "Time",
                        "confidenceScore": 0.87,
                        "isNegated": false
                    },
                    {
                        "offset": 73,
                        "length": 7,
                        "text": "120 min",
                        "category": "Time",
                        "confidenceScore": 0.99,
                        "isNegated": false
                    }
                ],
                "relations": [
                    {
                        "relationType": "DosageOfMedication",
                        "bidirectional": false,
                        "source": "#/results/documents/0/entities/0",
                        "target": "#/results/documents/0/entities/1"
                    },
                    {
                        "relationType": "RouteOfMedication",
                        "bidirectional": false,
                        "source": "#/results/documents/0/entities/2",
                        "target": "#/results/documents/0/entities/1"
                    },
                    {
                        "relationType": "TimeOfMedication",
                        "bidirectional": false,
                        "source": "#/results/documents/0/entities/3",
                        "target": "#/results/documents/0/entities/1"
                    },
                    {
                        "relationType": "TimeOfMedication",
                        "bidirectional": false,
                        "source": "#/results/documents/0/entities/4",
                        "target": "#/results/documents/0/entities/1"
                    }
                ],
                "warnings": []
            }
        ],
        "errors": [],
        "modelVersion": "2020-09-03"
    }
}
```


### <a name="structure-the-api-request-for-the-container"></a>Structurer la requête d’API pour le conteneur

Vous pouvez [utiliser Postman](text-analytics-how-to-call-api.md) ou l’exemple de requête cURL ci-dessous pour envoyer une requête au conteneur que vous avez déployé, en remplaçant la variable `serverURL` par la valeur appropriée.  Notez que la version de l’API dans l’URL du conteneur est différente de celle de l’API hébergée.

```bash
curl -X POST 'http://<serverURL>:5000/text/analytics/v3.2-preview.1/entities/health' --header 'Content-Type: application/json' --header 'accept: application/json' --data-binary @example.json

```

Le code JSON suivant est un exemple de fichier JSON joint au corps POST de la requête API d’Analyse de texte pour la santé :

```json
example.json

{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "Patient reported itchy sores after swimming in the lake."
    },
    {
      "language": "en",
      "id": "2",
      "text": "Prescribed 50mg benadryl, taken twice daily."
    }
  ]
}
```

### <a name="container-response-body"></a>Corps de la réponse du conteneur

Le code JSON suivant est un exemple de corps de la réponse de l’API Analyse de texte pour la santé à l’issue de l’appel synchrone conteneurisé :

```json
{
    "documents": [
        {
            "id": "1",
            "entities": [
                {
                    "id": "0",
                    "offset": 25,
                    "length": 5,
                    "text": "100mg",
                    "category": "Dosage",
                    "confidenceScore": 1.0,
                    "isNegated": false
                },
                {
                    "id": "1",
                    "offset": 31,
                    "length": 10,
                    "text": "remdesivir",
                    "category": "MedicationName",
                    "confidenceScore": 1.0,
                    "isNegated": false,
                    "links": [
                        {
                            "dataSource": "UMLS",
                            "id": "C4726677"
                        },
                        {
                            "dataSource": "MSH",
                            "id": "C000606551"
                        },
                        {
                            "dataSource": "NCI",
                            "id": "C152185"
                        },
                        {
                            "dataSource": "NCI_FDA",
                            "id": "3QKI37EEHE"
                        }
                    ]
                },
                {
                    "id": "2",
                    "offset": 42,
                    "length": 13,
                    "text": "intravenously",
                    "category": "MedicationRoute",
                    "confidenceScore": 1.0,
                    "isNegated": false
                },
                {
                    "id": "3",
                    "offset": 56,
                    "length": 4,
                    "text": "over",
                    "category": "Time",
                    "confidenceScore": 0.87,
                    "isNegated": false
                },
                {
                    "id": "4",
                    "offset": 73,
                    "length": 7,
                    "text": "120 min",
                    "category": "Time",
                    "confidenceScore": 0.99,
                    "isNegated": false
                }
            ],
            "relations": [
                {
                    "relationType": "DosageOfMedication",
                    "bidirectional": false,
                    "source": "#/documents/0/entities/0",
                    "target": "#/documents/0/entities/1"
                },
                {
                    "relationType": "RouteOfMedication",
                    "bidirectional": false,
                    "source": "#/documents/0/entities/2",
                    "target": "#/documents/0/entities/1"
                },
                {
                    "relationType": "TimeOfMedication",
                    "bidirectional": false,
                    "source": "#/documents/0/entities/3",
                    "target": "#/documents/0/entities/1"
                },
                {
                    "relationType": "TimeOfMedication",
                    "bidirectional": false,
                    "source": "#/documents/0/entities/4",
                    "target": "#/documents/0/entities/1"
                }
            ]
        }
    ],
    "errors": [],
    "modelVersion": "2020-09-03"
}
```

### <a name="negation-detection-output"></a>Sortie de détection de négation

Fans certains cas, en utilisant la détection de négation, un terme de négation unique peut traiter plusieurs termes à la fois. La négation d’une entité reconnue est représentée dans la sortie JSON par la valeur booléenne de l’indicateur `isNegated`, par exemple :

```json
{
  "id": "2",
  "offset": 90,
  "length": 10,
  "text": "chest pain",
  "category": "SymptomOrSign",
  "score": 0.9972,
  "isNegated": true,
  "links": [
    {
      "dataSource": "UMLS",
      "id": "C0008031"
    },
    {
      "dataSource": "CHV",
      "id": "0000023593"
    },
    ...
```

### <a name="relation-extraction-output"></a>Sortie d’extraction de relations

La sortie d’extraction de relations contient des références URI à la *source* de la relation, et sa *cible*. Les entités ayant le rôle de relation de `ENTITY` sont affectées au champ `target`. Les entités ayant le rôle de relation de `ATTRIBUTE` sont affectées au champ `source`. Les relations d’abréviation contiennent des champs `source` et `target` bidirectionnels, et `bidirectional` sera défini sur `true`. 

```json
"relations": [
                {
                    "relationType": "DosageOfMedication",
                    "bidirectional": false,
                    "source": "#/documents/1/entities/0",
                    "target": "#/documents/1/entities/1"
                },
                {
                    "relationType": "FrequencyOfMedication",
                    "bidirectional": false,
                    "source": "#/documents/1/entities/2",
                    "target": "#/documents/1/entities/1"
                }
            ]
  },
...
]
```

## <a name="see-also"></a>Voir aussi

* [Vue d’ensemble d’Analyse de texte](../overview.md)
* [Catégories d’entité nommée](../named-entity-types.md)
* [Nouveautés](../whats-new.md)
