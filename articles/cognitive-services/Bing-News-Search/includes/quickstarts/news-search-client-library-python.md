---
title: 'Démarrage rapide : Bibliothèque de client Python Recherche d’actualités Bing'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/12/2020
ms.author: aahi
ms.openlocfilehash: c1bd0d86a3fd9d19d67d84b9b05955421373e01e
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "79503508"
---
Utilisez ce guide de démarrage rapide pour démarrer une recherche d’actualités avec la bibliothèque de client Recherche d’actualités Bing pour Python. Si l’outil Recherche d’actualités Bing dispose d’une API REST compatible avec la plupart des langages de programmation, la bibliothèque de client offre quant à elle un moyen facile d’intégrer le service à vos applications. Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples/blob/master/samples/search/news_search_samples.py).

## <a name="prerequisites"></a>Prérequis

* [Python](https://www.python.org/) 2.x ou 3.x

Il est recommandé d’utiliser un [environnement virtuel](https://docs.python.org/3/tutorial/venv.html) pour votre développement Python. Vous pouvez installer et initialiser l’environnement virtuel avec le [module venv](https://pypi.python.org/pypi/virtualenv). Vous devez installer un environnement virtuel pour Python 2.7. Vous pouvez créer un environnement virtuel avec :

```console
python -m venv mytestenv
```

Vous pouvez installer les dépendances de la bibliothèque de client Recherche d’actualités Bing à l’aide de cette commande :
    
```console
python -m pip install azure-cognitiveservices-search-newssearch
```

[!INCLUDE [cognitive-services-bing-news-search-signup-requirements](~/includes/cognitive-services-bing-news-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Créer et initialiser l’application

1. Créez un fichier Python dans votre éditeur ou IDE favori, puis importez les bibliothèques suivantes. Créez une variable pour votre clé d’abonnement ainsi qu’un terme de recherche.

    ```python
    from azure.cognitiveservices.search.newssearch import NewsSearchClient
    from msrest.authentication import CognitiveServicesCredentials
    subscription_key = "YOUR-SUBSCRIPTION-KEY"
    endpoint = "YOUR-ENDPOINT"
    search_term = "Quantum Computing"
    ```

## <a name="initialize-the-client-and-send-a-request"></a>Initialiser le client et envoyer une requête

1. Créez une instance de `CognitiveServicesCredentials`.
    
    ```python
    client = NewsSearchClient(endpoint=endpoint, credentials=CognitiveServicesCredentials(subscription_key))
    ```

2. Envoyez une requête de recherche à l’API Recherche d’actualités et stockez la réponse.

    ```python
    news_result = client.news.search(query=search_term, market="en-us", count=10)
    ```

## <a name="parse-the-response"></a>Analyser la réponse

Si des résultats sont retournés, affichez le premier résultat de la page web :

```python
if news_result.value:
    first_news_result = news_result.value[0]
    print("Total estimated matches value: {}".format(
        news_result.total_estimated_matches))
    print("News result count: {}".format(len(news_result.value)))
    print("First news name: {}".format(first_news_result.name))
    print("First news url: {}".format(first_news_result.url))
    print("First news description: {}".format(first_news_result.description))
    print("First published time: {}".format(first_news_result.date_published))
    print("First news provider: {}".format(first_news_result.provider[0].name))
else:
    print("Didn't see any news result data..")
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application web monopage](../../tutorial-bing-news-search-single-page-app.md)
