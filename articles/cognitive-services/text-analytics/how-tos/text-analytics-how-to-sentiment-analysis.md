---
title: Effectuer une analyse des sentiments et une exploration des opinions avec l’API REST Analyse de texte
titleSuffix: Azure Cognitive Services
description: Cet article vous montre comment détecter les sentiments et explorer les opinions dans du texte avec l’API Analyse de texte d’Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: sample
ms.date: 12/04/2020
ms.author: aahi
ms.openlocfilehash: 4f5c47000355a004f8d76d8b740b705618748380
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505220"
---
# <a name="how-to-sentiment-analysis-and-opinion-mining"></a>Procédure : Analyse des sentiments et Exploration des opinions

La fonctionnalité Analyse des sentiments de l’API Analyse de texte offre deux façons de détecter les sentiments positifs et négatifs. Si vous envoyez une demande d’Analyse de sentiments, l’API retourne des étiquettes de sentiment, comme « negative » (négatif), « neutral » (neutre) et « positive » (positif), et des scores de confiance au niveau de la phrase et du document. Vous pouvez également envoyer des demandes d’Exploration des opinions en utilisant le point de terminaison Analyse des sentiments, qui fournit des informations précises sur les opinions relatives à des aspects (comme les attributs de produits ou de services) du texte. 

Les modèles IA utilisés par l’API sont fournis par le service ; vous n’avez qu’à envoyer le contenu à analyser.

## <a name="sentiment-analysis-versions-and-features"></a>Versions et fonctionnalités d’Analyse des sentiments

[!INCLUDE [v3 region availability](../includes/v3-region-availability.md)]

| Fonctionnalité                                   | Analyse des sentiments v3 | Analyse des sentiments v3.1 (préversion) |
|-------------------------------------------|-----------------------|-----------------------------------|
| Méthodes pour les requêtes uniques et de lots    | X                     | X                                 |
| Scores et étiquetage d’Analyse des sentiments             | X                     | X                                 |
| [Conteneur Docker](text-analytics-how-to-install-containers.md) basé sur Linux | X  |  |
| Exploration des opinions                            |                       | X                                 |

## <a name="sentiment-analysis"></a>Analyse des sentiments

Dans sa version 3.x, Analyse des sentiments applique des étiquettes de sentiment au texte, qui sont retournées au niveau de la phrase et du document, avec un score de confiance pour chacune d’elles. 

Les étiquettes sont *positive* (positif), *negative* (négatif) et *neutral* (neutre). Au niveau du document, l’étiquette de sentiment *mixed* (mixte) peut aussi être retournée. Le sentiment du document est déterminé comme suit :

| Sentiment des phrases                                                                            | Étiquette de document retournée |
|-----------------------------------------------------------------------------------------------|-------------------------|
| Le document contient au moins une phrase `positive`. Le reste des phrases est de type `neutral`. | `positive`              |
| Le document contient au moins une phrase `negative`. Le reste des phrases est de type `neutral`. | `negative`              |
| Le document contient au moins une phrase `negative` et au moins une phrase `positive`.    | `mixed`                 |
| Toutes les phrases du document sont de type `neutral`.                                                  | `neutral`               |

Les scores de confiance sont compris entre 1 et 0. Plus les scores sont proches de 1, plus le niveau de confiance dans la classification de l’étiquette est élevé ; inversement, plus les scores sont faibles, plus le niveau de confiance est bas. Pour chaque document ou chaque phrase, les scores prédits associés aux étiquettes (positif, négatif et neutre) donnent la somme de 1.

## <a name="opinion-mining"></a>Exploration des opinions

L’Exploration des opinions est une fonctionnalité d’Analyse des sentiments, disponible dans la préversion de la version 3.1. Également connu sous le nom d’Analyse des sentiments basée sur l’aspect dans le registre du traitement en langage naturel, cette fonctionnalité fournit des informations plus granulaires sur les opinions liées aux aspects (tels que les attributs de produits ou de services) dans le texte.

Par exemple, si un client laisse un commentaire sur un hôtel, comme « la chambre était géniale, mais le personnel peu sympathique », Exploration des opinions va repérer des aspects dans le texte ainsi que les opinions et les sentiments associés. Analyse des sentiments peut signaler seulement un sentiment négatif.

:::image type="content" source="../media/how-tos/opinion-mining.png" alt-text="Diagramme de l’exemple d’Exploration des opinions" lightbox="../media/how-tos/opinion-mining.png":::

Pour avoir l’Exploration des opinions dans vos résultats, vous devez inclure l’indicateur `opinionMining=true` dans une demande d’Analyse de sentiments. Les résultats de l’Exploration des opinions sont inclus dans la réponse de l’Analyse des sentiments. L’exploration des opinions est une extension de l’Analyse des sentiments qui est incluse dans votre [niveau tarifaire](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/) actuel.


## <a name="sending-a-rest-api-request"></a>Envoie d’une requête d’API REST 

### <a name="preparation"></a>Préparation

La qualité des résultats de l’analyse des sentiments est d’autant meilleure que vous lui donnez de petites quantités de texte à analyser. Au contraire, l’extraction d’expressions clés fonctionne mieux sur de plus grands blocs de texte. Pour obtenir des résultats optimaux pour ces deux opérations, envisagez de restructurer les entrées en conséquence.

Vous devez disposer des documents JSON dans ce format : ID, texte et langue. Analyse des sentiments prend en charge une large palette de langues, et plus de langues encore en préversion. Pour en savoir plus, consultez [Langages pris en charge](../language-support.md).

La taille du document doit être inférieure à 5 120 caractères par document. Pour connaître le nombre maximal de documents autorisés dans une collection, consultez l’article [Limites de données](../concepts/data-limits.md?tabs=version-3) sous Concepts. La collection est soumise dans le corps de la demande.

## <a name="structure-the-request"></a>Structurer la requête

Créez une requête POST. Vous pouvez [utiliser Postman](text-analytics-how-to-call-api.md) ou la **console de test d’API** via les liens de référence suivants pour en structurer une rapidement et l’envoyer. 

#### <a name="version-31-preview3"></a>[Version 3.1-preview.3](#tab/version-3-1)

[Informations de référence sur Analyse des sentiments v3.1](https://westcentralus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-1-preview-3/operations/Sentiment)

#### <a name="version-30"></a>[Version 3.0](#tab/version-3)

[Informations de référence sur Analyse des sentiments v3](https://westus2.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0/operations/Sentiment)

---

### <a name="request-endpoints"></a>Points de terminaison de requête

Définissez le point de terminaison HTTPS pour l’analyse des sentiments à l’aide d’une ressource Analyse de texte sur Azure ou d’un [conteneur Analyse de texte](text-analytics-how-to-install-containers.md) instancié. Vous devez inclure l’URL correspondant à la version que vous souhaitez utiliser. Par exemple :

> [!NOTE]
> Vous pouvez trouver la clé et le point de terminaison pour votre ressource Analyse de texte dans le portail Azure. Ces informations se trouvent dans la page **Démarrage rapide** de la ressource, sous **gestion des ressources**. 

#### <a name="version-31-preview3"></a>[Version 3.1-preview.3](#tab/version-3-1)

**Analyse des sentiments**

`https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.3/sentiment`

**Exploration des opinions**

Pour obtenir les résultats de l’Exploration des opinions, vous devez inclure le paramètre `opinionMining=true`. Par exemple :

`https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.3/sentiment?opinionMining=true`

La valeur par défaut de ce paramètre est `false`. 

#### <a name="version-30"></a>[Version 3.0](#tab/version-3)

**Analyse des sentiments**

Dans la version 3.0, le seul point de terminaison disponible est pour l’Analyse des sentiments.
 
`https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.0/sentiment`

---

Définissez un en-tête de requête pour inclure votre clé d’API Analyse de texte. Dans le corps de la demande, fournissez la collection de documents JSON que vous avez préparée pour cette analyse.

### <a name="example-request-for-sentiment-analysis-and-opinion-mining"></a>Exemple de demande pour l’Analyse des sentiments et l’Exploration des opinions  

Voici un exemple de contenu que vous pouvez soumettre pour analyse des sentiments. Le format de la demande est le même pour les deux versions, `v3.0` et `v3.1-preview`.
    
```json
{
  "documents": [
    {
      "language": "en",
      "id": "1",
      "text": "The restaurant had great food and our waiter was friendly."
    }
  ]
}
```

### <a name="post-the-request"></a>Publier la requête

L’analyse est effectuée à la réception de la demande. Pour connaître la taille et le nombre de requêtes que vous pouvez envoyer par minute et seconde, consultez la section [Limites de données](../overview.md#data-limits) dans la vue d’ensemble.

L’API Analyse de texte est sans état. Aucune donnée n’est stockée dans votre compte, et les résultats sont retournés immédiatement dans la réponse.


### <a name="view-the-results"></a>View the results

La sortie est retournée immédiatement. Vous pouvez diffuser les résultats vers une application qui accepte JSON ou enregistrer la sortie dans un fichier sur le système local. Ensuite, importez la sortie dans une application que vous pouvez utiliser pour trier, rechercher et manipuler les données. En raison de la prise en charge multilingue et des émojis, la réponse peut contenir des décalages de texte. Pour plus d’informations, consultez le [guide pratique pour traiter les décalages](../concepts/text-offsets.md).

#### <a name="version-31-preview3"></a>[Version 3.1-preview.3](#tab/version-3-1)

### <a name="sentiment-analysis-and-opinion-mining-example-response"></a>Exemple de réponse de l’Analyse des sentiments et de l’Exploration des opinions

> [!IMPORTANT]
> Voici un exemple JSON pour l’utilisation de l’Exploration des opinions avec l’Analyse des sentiments, dans la version 3.1 de l’API. Si vous ne demandez pas l’Exploration des opinions, la réponse de l’API sera la même que sous l’onglet **Version 3.0**.  

Analyse des sentiments version 3.1 peut retourner des objets de réponse à la fois pour l’Analyse des sentiments et l’Exploration des opinions.
  
L’analyse des sentiments retourne une étiquette de sentiment et un score de confiance pour l’ensemble du document et chaque phrase qu’il contient. Plus les scores sont proches de 1, plus le niveau de confiance dans la classification de l’étiquette est élevé ; inversement, plus les scores sont faibles, plus le niveau de confiance est bas. Un document peut contenir plusieurs phrases, et les scores de confiance dans chaque document ou phrase s’additionnent pour arriver à 1.

L’Exploration des opinions va localiser des aspects dans le texte ainsi que les opinions et les sentiments associés. Dans la réponse ci-dessous, la phrase *The restaurant had great food and our waiter was friendly* (Les mets au restaurant étaient excellents et le serveur sympathique) présente deux aspects : *food* (mets) et *waiter* (serveur). La propriété `relations` de chaque aspect contient une valeur `ref` avec la référence d’URI aux objets `documents`, `sentences` et `opinions` associés.

```json
{
    "documents": [
        {
            "id": "1",
            "sentiment": "positive",
            "confidenceScores": {
                "positive": 1.0,
                "neutral": 0.0,
                "negative": 0.0
            },
            "sentences": [
                {
                    "sentiment": "positive",
                    "confidenceScores": {
                        "positive": 1.0,
                        "neutral": 0.0,
                        "negative": 0.0
                    },
                    "offset": 0,
                    "length": 58,
                    "text": "The restaurant had great food and our waiter was friendly.",
                    "aspects": [
                        {
                            "sentiment": "positive",
                            "confidenceScores": {
                                "positive": 1.0,
                                "negative": 0.0
                            },
                            "offset": 25,
                            "length": 4,
                            "text": "food",
                            "relations": [
                                {
                                    "relationType": "opinion",
                                    "ref": "#/documents/0/sentences/0/opinions/0"
                                }
                            ]
                        },
                        {
                            "sentiment": "positive",
                            "confidenceScores": {
                                "positive": 1.0,
                                "negative": 0.0
                            },
                            "offset": 38,
                            "length": 6,
                            "text": "waiter",
                            "relations": [
                                {
                                    "relationType": "opinion",
                                    "ref": "#/documents/0/sentences/0/opinions/1"
                                }
                            ]
                        }
                    ],
                    "opinions": [
                        {
                            "sentiment": "positive",
                            "confidenceScores": {
                                "positive": 1.0,
                                "negative": 0.0
                            },
                            "offset": 19,
                            "length": 5,
                            "text": "great",
                            "isNegated": false
                        },
                        {
                            "sentiment": "positive",
                            "confidenceScores": {
                                "positive": 1.0,
                                "negative": 0.0
                            },
                            "offset": 49,
                            "length": 8,
                            "text": "friendly",
                            "isNegated": false
                        }
                    ]
                }
            ],
            "warnings": []
        }
    ],
    "errors": [],
    "modelVersion": "2020-04-01"
}
```

#### <a name="version-30"></a>[Version 3.0](#tab/version-3)

### <a name="sentiment-analysis-example-response"></a>Exemple de réponse de l’Analyse des sentiments

L’analyse des sentiments retourne une étiquette de sentiment et un score de confiance pour l’ensemble du document et chaque phrase qu’il contient. Plus les scores sont proches de 1, plus le niveau de confiance dans la classification de l’étiquette est élevé ; inversement, plus les scores sont faibles, plus le niveau de confiance est bas. Un document peut contenir plusieurs phrases, et les scores de confiance dans chaque document ou phrase s’additionnent pour arriver à 1.

Les réponses d’Analyse des sentiments v3 contiennent des étiquettes de sentiment pour chaque phrase et document analysés.

```json
{
    "documents": [
        {
            "id": "1",
            "sentiment": "positive",
            "confidenceScores": {
                "positive": 1.0,
                "neutral": 0.0,
                "negative": 0.0
            },
            "sentences": [
                {
                    "sentiment": "positive",
                    "confidenceScores": {
                        "positive": 1.0,
                        "neutral": 0.0,
                        "negative": 0.0
                    },
                    "offset": 0,
                    "length": 58,
                    "text": "The restaurant had great food and our waiter was friendly."
                }
            ],
            "warnings": []
        }
    ],
    "errors": [],
    "modelVersion": "2020-04-01"
}
```

---

## <a name="summary"></a>Résumé

Dans cet article, vous avez découvert les concepts et le workflow de l’analyse des sentiments avec l’API Analyse de texte. En résumé :

+ L’Analyse des sentiments et l’Exploration des opinions sont disponibles pour certaines langues.
+ Les documents JSON figurant dans le corps de la demande incluent un ID, un texte et un code de langue.
+ La requête POST s’effectue sur un point de terminaison `/sentiment`, à l’aide [d’une clé d’accès et d’un point de terminaison](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) personnalisés valides pour votre abonnement.
+ Utilisez `opinionMining=true` dans les demandes d’Analyse des sentiments pour obtenir des résultats de l’Exploration des opinions.
+ La sortie de réponse, qui se compose d’un score de sentiment pour chaque ID de document, peut être diffusée vers n’importe quelle application qui accepte JSON, Par exemple, Excel et Power BI.

## <a name="see-also"></a>Voir aussi

* [Vue d’ensemble d’Analyse de texte](../overview.md)
* [Utilisation de la bibliothèque cliente Analyse de texte](../quickstarts/client-libraries-rest-api.md)
* [Nouveautés](../whats-new.md)
