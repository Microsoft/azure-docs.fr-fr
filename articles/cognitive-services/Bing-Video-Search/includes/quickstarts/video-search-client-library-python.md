---
title: 'Démarrage rapide : Bibliothèque de client Python Recherche de vidéos Bing'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/19/2020
ms.author: aahi
ms.openlocfilehash: 7a9fab8ba8bb9d21c9284cbf14bc67226d2ef9d3
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "80289858"
---
Utilisez ce guide de démarrage rapide pour démarrer une recherche de vidéos avec la bibliothèque de client Recherche de vidéos Bing pour Python. Si l’outil Recherche de vidéos Bing dispose d’une API REST compatible avec la plupart des langages de programmation, la bibliothèque de client offre quant à elle un moyen facile d’intégrer le service à vos applications. Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples/blob/master/samples/search/video_search_samples.py) avec des fonctionnalités et annotations supplémentaires.

[!INCLUDE [cognitive-services-bing-video-search-signup-requirements](~/includes/cognitive-services-bing-video-search-signup-requirements.md)]

## <a name="prerequisites"></a>Prérequis

- [Python](https://www.python.org/) 2.x ou 3.x
- Bibliothèque de client Recherche de vidéos Bing pour Python

Nous vous recommandons d’utiliser un [environnement virtuel](https://docs.python.org/3/tutorial/venv.html) python. Vous pouvez installer et lancer un environnement virtuel avec le [module venv](https://pypi.python.org/pypi/virtualenv). Installez virtualenv pour Python 2.7 avec :

```console
python -m venv mytestenv
```

Installez la bibliothèque de client Recherche de vidéos Bing avec :

```console
cd mytestenv
python -m pip install azure-cognitiveservices-search-videosearch
```

## <a name="create-and-initialize-the-application"></a>Créer et initialiser l’application

1. Créez un fichier Python dans votre éditeur ou IDE favori, puis ajoutez les instructions d’importation suivantes. 

    ```python
    from azure.cognitiveservices.search.videosearch import VideoSearchClient
    from azure.cognitiveservices.search.videosearch.models import VideoPricing, VideoLength, VideoResolution, VideoInsightModule
    from msrest.authentication import CognitiveServicesCredentials
    ```

2. Créez une variable pour votre clé d’abonnement. 

    ```python
    subscription_key = "YOUR-SUBSCRIPTION-KEY"
    endpoint = "YOUR-ENDPOINT"
    ```

## <a name="create-the-search-client"></a>Créer le client de recherche

Créez une instance de `CognitiveServicesCredentials`, et instanciez le client :

```python
client = VideoSearchAPI(endpoint, CognitiveServicesCredentials(subscription_key))
```

## <a name="send-a-search-request-and-get-a-response"></a>Envoyer une requête de recherche et obtenir une réponse

1. Utilisez `client.videos.search()` avec votre requête de recherche pour envoyer une demande à l’API Recherche de vidéos Bing et obtenir une réponse.

    ```python
    video_result = client.videos.search(query="SwiftKey")
    ```

2. Si la réponse contient des résultats de recherche, obtenez le premier, puis imprimez son ID, son nom et sont URL.

    ```python
    if video_result.value:
        first_video_result = video_result.value[0]
        print("Video result count: {}".format(len(video_result.value)))
        print("First video id: {}".format(first_video_result.video_id))
        print("First video name: {}".format(first_video_result.name))
        print("First video url: {}".format(first_video_result.content_url))
    else:
        print("Didn't see any video result data..")
    ```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application web monopage](../../tutorial-bing-video-search-single-page-app.md)

## <a name="see-also"></a>Voir aussi 

- [Qu’est-ce que l’API Recherche de vidéos Bing ?](../../overview.md)
- [Exemples du Kit de développement logiciel (SDK) .NET pour Cognitive Services](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7)
