---
title: Utilisation de points de terminaison Azure Machine Learning studio (classique) dans Azure Stream Analytics
description: Cet article décrit comment utiliser des fonctions Machine Learning définies par l’utilisateur dans Azure Stream Analytics.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 06/11/2019
ms.openlocfilehash: 236191710dac19a08db0e8ce94dc695d393009a7
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93127125"
---
# <a name="azure-machine-learning-studio-classic-integration-in-stream-analytics-preview"></a>Intégration d’Azure Machine Learning Studio (classique) dans Stream Analytics (préversion)
Stream Analytics prend en charge les fonctions définies par l’utilisateur qui appellent des points de terminaison Azure Machine Learning Studio (classique). Prise en charge de l’API REST de cette fonctionnalité est détaillée dans la [bibliothèque de l’API REST Stream Analytics](/rest/api/streamanalytics/). Cet article fournit les informations supplémentaires nécessaires à la mise en œuvre réussie de cette fonctionnalité dans Stream Analytics. Un didacticiel a également été validé et est disponible [ici](stream-analytics-machine-learning-integration-tutorial.md).

## <a name="overview-azure-machine-learning-studio-classic-terminology"></a>Présentation : Terminologie Azure Machine Learning Studio (classique)
Microsoft Azure Machine Learning Studio (classique) offre un outil collaboratif fonctionnant par glisser-déplacer qui vous permet de générer, tester et déployer des solutions d’analyse prédictive sur vos données. Cet outil s’appelle *Azure Machine Learning studio (classique)* . Studio (classique) permet d’interagir avec les ressources de Machine Learning et facilite la création, les tests et le remaniement d’un projet de conception. Ces ressources et leurs définitions se trouvent ci-dessous.

* **Espace de travail** : il s’agit d’un conteneur regroupant toutes les autres ressources Machine Learning à des fins de gestion et de contrôle.
* **Expérience** : Les *expériences* sont créées par des scientifiques des données qui utilisent des jeux de données et effectuent l’apprentissage d’un modèle Machine Learning.
* **Point de terminaison** : il s’agit de l’objet studio (classique) utilisé pour prendre des fonctionnalités en entrée, appliquer un modèle Machine Learning donné et renvoyer la sortie évaluée.
* **Service Web d’évaluation** : Un *service web d’évaluation* représente une collection de points de terminaison, comme indiqué ci-dessus.

Chaque point de terminaison dispose d’API servant à l’exécution de lots et l’exécution synchronisée. Stream Analytics utilise l’exécution synchronisée. Le service spécifique est nommé [Service requête/réponse](../machine-learning/classic/consume-web-services.md) dans Azure Machine Learning Studio (classique).

## <a name="studio-classic-resources-needed-for-stream-analytics-jobs"></a>Ressources studio (classique) nécessaires aux travaux Stream Analytics
Pour les besoins de l’analyse des travaux Stream Analytics, un point de terminaison demande/réponse, une clé [apikey](../machine-learning/classic/consume-web-services.md)et une définition swagger sont nécessaires pour une exécution réussie. Stream Analytics est doté d’un point de terminaison supplémentaire qui génère l’URL d’un point de terminaison swagger, fait des recherches dans l’interface et retourne une définition de fonction UDF par défaut à l’utilisateur.

## <a name="configure-a-stream-analytics-and-studio-classic-udf-via-rest-api"></a>Configuration d’une fonction définie par l’utilisateur Stream Analytics et studio (classique) au moyen d’une API REST
À l’aide des API REST, vous pouvez configurer votre travail de façon à appeler des fonctions studio (classique). La procédure comporte trois étapes :

1. Création d’un travail Stream Analytics
2. Définition d’une entrée
3. Définition d’une sortie
4. Création d’une fonction définie par l’utilisateur
5. Création d’une transformation Stream Analytics faisant appel à la fonction définie par l’utilisateur
6. Démarrage du travail

## <a name="creating-a-udf-with-basic-properties"></a>Création d’une fonction définie par l’utilisateur avec des propriétés de base
Par exemple, l’échantillon de code suivant crée un fichier UDF scalaire nommé *newudf* associé à un point de terminaison Azure Machine Learning Studio (classique). Notez que le *point de terminaison* (URI de service) se trouve sur la page d’aide API correspondant au service choisi et la clé *apiKey* se trouve sur la page principale des services.

```
    PUT : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.StreamAnalytics/streamingjobs/<streamingjobName>/functions/<udfName>?api-version=<apiVersion>
```

Exemple de corps de requête :

```json
    {
        "name": "newudf",
        "properties": {
            "type": "Scalar",
            "properties": {
                "binding": {
                    "type": "Microsoft.MachineLearning/WebService",
                    "properties": {
                        "endpoint": "https://ussouthcentral.services.azureml.net/workspaces/f80d5d7a77fb4b46bf2a30c63c078dca/services/b7be5e40fd194258796fb402c1958eaf/execute ",
                        "apiKey": "replacekeyhere"
                    }
                }
            }
        }
    }
```

## <a name="call-retrievedefaultdefinition-endpoint-for-default-udf"></a>Appeler le point de terminaison RetrieveDefaultDefinition pour la fonction définie par l’utilisateur par défaut
Une fois la structure de la fonction définie par l’utilisateur créée, une définition complète de la fonction définie par l’utilisateur est nécessaire. Le point de terminaison RetrieveDefaultDefinition vous permet d’obtenir une définition par défaut d’une fonction scalaire liée à un point de terminaison Azure Machine Learning Studio (classique). La charge utile ci-dessous impose d’obtenir la définition par défaut d’une fonction scalaire définie par l’utilisateur liée à un point de terminaison studio (classique). Il ne spécifie pas le point de terminaison réel tel qu’il a déjà été fourni pendant la requête PUT. Stream Analytics appelle le point de terminaison fourni dans la demande s’il est indiqué de façon explicite. Dans le cas contraire, il utilise celui qui a été référencé à l’origine. Ici, la fonction définie par l’utilisateur prend un paramètre à chaîne unique (une phrase) et retourne un seul résultat de type chaîne qui mentionne le libellé « sentiment » de cette phrase.

```
POST : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.StreamAnalytics/streamingjobs/<streamingjobName>/functions/<udfName>/RetrieveDefaultDefinition?api-version=<apiVersion>
```

Exemple de corps de requête :

```json
    {
        "bindingType": "Microsoft.MachineLearning/WebService",
        "bindingRetrievalProperties": {
            "executeEndpoint": null,
            "udfType": "Scalar"
        }
    }
```

Un exemple de sortie ressemble à ce qui suit.

```json
    {
        "name": "newudf",
        "properties": {
            "type": "Scalar",
            "properties": {
                "inputs": [{
                    "dataType": "nvarchar(max)",
                    "isConfigurationParameter": null
                }],
                "output": {
                    "dataType": "nvarchar(max)"
                },
                "binding": {
                    "type": "Microsoft.MachineLearning/WebService",
                    "properties": {
                        "endpoint": "https://ussouthcentral.services.azureml.net/workspaces/f80d5d7a77ga4a4bbf2a30c63c078dca/services/b7be5e40fd194258896fb602c1858eaf/execute",
                        "apiKey": null,
                        "inputs": {
                            "name": "input1",
                            "columnNames": [{
                                "name": "tweet",
                                "dataType": "string",
                                "mapTo": 0
                            }]
                        },
                        "outputs": [{
                            "name": "Sentiment",
                            "dataType": "string"
                        }],
                        "batchSize": 10
                    }
                }
            }
        }
    }
```

## <a name="patch-udf-with-the-response"></a>Fonction définie par l’utilisateur correctif avec la réponse
Maintenant la fonction définie par l’utilisateur peut être corrigée à l’aide de la réponse précédente, comme indiqué ci-dessous.

```
PATCH : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.StreamAnalytics/streamingjobs/<streamingjobName>/functions/<udfName>?api-version=<apiVersion>
```

Corps de la demande (sortie de RetrieveDefaultDefinition) :

```json
    {
        "name": "newudf",
        "properties": {
            "type": "Scalar",
            "properties": {
                "inputs": [{
                    "dataType": "nvarchar(max)",
                    "isConfigurationParameter": null
                }],
                "output": {
                    "dataType": "nvarchar(max)"
                },
                "binding": {
                    "type": "Microsoft.MachineLearning/WebService",
                    "properties": {
                        "endpoint": "https://ussouthcentral.services.azureml.net/workspaces/f80d5d7a77ga4a4bbf2a30c63c078dca/services/b7be5e40fd194258896fb602c1858eaf/execute",
                        "apiKey": null,
                        "inputs": {
                            "name": "input1",
                            "columnNames": [{
                                "name": "tweet",
                                "dataType": "string",
                                "mapTo": 0
                            }]
                        },
                        "outputs": [{
                            "name": "Sentiment",
                            "dataType": "string"
                        }],
                        "batchSize": 10
                    }
                }
            }
        }
    }
```

## <a name="implement-stream-analytics-transformation-to-call-the-udf"></a>Implémentation de la transformation Stream Analytics pour appeler la fonction définie par l’utilisateur
Maintenant, interrogez la fonction définie par l’utilisateur (nommée ici scoreTweet) pour chaque événement en entrée et rédiger une réponse pour cette sortie.

```json
    {
        "name": "transformation",
        "properties": {
            "streamingUnits": null,
            "query": "select *,scoreTweet(Tweet) TweetSentiment into blobOutput from blobInput"
        }
    }
```


## <a name="get-help"></a>Obtenir de l’aide
Pour obtenir de l’aide supplémentaire, essayez notre [page de questions Microsoft Q&R sur Azure Stream Analytics](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Étapes suivantes
* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)