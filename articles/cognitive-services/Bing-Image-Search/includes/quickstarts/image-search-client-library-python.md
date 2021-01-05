---
title: 'Démarrage rapide : Bibliothèque de client Python Recherche d’images Bing'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/04/2020
ms.author: aahi
ms.openlocfilehash: d5d47f097fa216d69b8ed59fdb057378724c2228
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94625266"
---
Utilisez ce guide de démarrage rapide pour effectuer votre première recherche d’images à l’aide de la bibliothèque de client Recherche d’images Bing, qui est un wrapper de l’API et contient les mêmes fonctionnalités. Cette application Python simple envoie une requête de recherche d’image, analyse la réponse JSON et affiche l’URL de la première image retournée.

Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples/blob/master/samples/search/image-search-quickstart.py) avec une gestion des erreurs supplémentaire et des annotations.

## <a name="prerequisites"></a>Prérequis

* [Python 2.7 ou 3.4](https://www.python.org/) et les versions ultérieures.

* Le [bibliothèque de client Recherche d’images Azure](https://pypi.org/project/azure-cognitiveservices-search-imagesearch/) pour Python
    * Installer à l’aide de `pip install azure-cognitiveservices-search-imagesearch`

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](~/includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Créer et initialiser l’application

1. Créez un script Python dans votre éditeur ou IDE favori, puis les importations suivantes :

    ```python
    from azure.cognitiveservices.search.imagesearch import ImageSearchClient
    from msrest.authentication import CognitiveServicesCredentials
    ```

2. Créez des variables pour votre clé d’abonnement et votre terme de recherche.

    ```python
    subscription_key = "Enter your key here"
    subscription_endpoint = "Enter your endpoint here"
    search_term = "canadian rockies"
    ```

## <a name="create-the-image-search-client"></a>Créer le client Recherche d’images

1. Créez une instance de `CognitiveServicesCredentials`, puis utilisez-la pour instancier le client :

    ```python
    client = ImageSearchClient(endpoint=subscription_endpoint, credentials=CognitiveServicesCredentials(subscription_key))
    ```
1. Envoyez une requête de recherche à l’API Recherche d’images Bing :
    ```python
    image_results = client.images.search(query=search_term)
    ```
   ## <a name="process-and-view-the-results"></a>Traiter et afficher les résultats

Analysez les résultats d’image retournés par la réponse.


Si la réponse contient des résultats de recherche, stockez le premier résultat et imprimez ses détails, tels qu’une URL de miniature, l’URL d’origine, ainsi que le nombre total d’images retournées.  

```python
if image_results.value:
    first_image_result = image_results.value[0]
    print("Total number of images returned: {}".format(len(image_results.value)))
    print("First image thumbnail url: {}".format(
        first_image_result.thumbnail_url))
    print("First image content url: {}".format(first_image_result.content_url))
else:
    print("No image results returned!")
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel d’une application monopage Recherche d’images Bing](../../tutorial-bing-image-search-single-page-app.md)

## <a name="see-also"></a>Voir aussi

* [Qu’est-ce que la Recherche d’images Bing ?](../../overview.md)  
* [Essayez une démonstration interactive en ligne](https://azure.microsoft.com/services/cognitive-services/bing-image-search-api/)  
* [Exemples Python pour le SDK Azure Cognitive Services](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples)  
* [Documentation Azure Cognitive Services](../../../index.yml)
* [Informations de référence sur l’API Recherche d’images Bing](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference)