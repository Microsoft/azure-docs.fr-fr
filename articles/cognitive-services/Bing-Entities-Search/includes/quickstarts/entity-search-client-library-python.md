---
title: 'Démarrage rapide : Bibliothèque de client Python Recherche d’entités Bing'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/06/2020
ms.author: aahi
ms.openlocfilehash: 0c0a124773eab8166806312dc47ded24e1cd841f
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "79136598"
---
Utilisez ce guide de démarrage rapide pour commencer à rechercher des entités avec la bibliothèque de client Recherche d’entités Bing pour Python. Si l’outil Recherche d’entités Bing a une API REST compatible avec la plupart des langages de programmation, la bibliothèque de client offre quant à elle un moyen facile d’intégrer le service à vos applications. Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples/blob/master/samples/search/entity_search_samples.py).

## <a name="prerequisites"></a>Prérequis

* Python[ 2.x ou 3.x](https://www.python.org/)

* [SDK Recherche d’entités Bing pour Python](https://pypi.org/project/azure-cognitiveservices-search-entitysearch/)

Nous vous recommandons d’utiliser un environnement virtuel python. Vous pouvez installer et lancer un environnement virtuel avec le module venv. Vous pouvez installer virtualenv avec :

```Console
python -m venv mytestenv
```

Installez la bibliothèque de client Recherche d’entités Bing avec :

```Console
cd mytestenv
python -m pip install azure-cognitiveservices-search-entitysearch
```

[!INCLUDE [cognitive-services-bing-news-search-signup-requirements](~/includes/cognitive-services-bing-entity-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Créer et initialiser l’application

1. Créez un fichier Python dans votre éditeur ou IDE favori, puis ajoutez les instructions d’importation suivantes. 

    ```python
    from azure.cognitiveservices.search.entitysearch import EntitySearchClient
    from azure.cognitiveservices.search.entitysearch.models import Place, ErrorResponseException
    from msrest.authentication import CognitiveServicesCredentials
    ```

2. Créez une variable pour votre clé d’abonnement et votre point de terminaison. Instanciez le client en créant un objet `CognitiveServicesCredentials` avec votre clé.
    
    ```python
    subscription_key = "YOUR-SUBSCRIPTION-KEY"
    endpoint = "YOUR-ENDPOINT"
    client = EntitySearchclient(endpoint=endpoint, credentials=CognitiveServicesCredentials(subscription_key))
    ```

## <a name="send-a-search-request-and-receive-a-response"></a>Envoyer une requête de recherche et recevoir une réponse

1. Envoyez une requête de recherche à Recherche d’entités Bing avec `client.entities.search()` et ladite requête. 
    
    ```python
    entity_data = client.entities.search(query="Gibralter")
    ```

2. Si des entités sont retournées, convertissez `entity_data.entities.value` en liste et imprimez le premier résultat.
    ```python
    if entity_data.entities.value:
    
        main_entities = [entity for entity in entity_data.entities.value
                         if entity.entity_presentation_info.entity_scenario == "DominantEntity"]
    
        if main_entities:
            print(main_entities[0].description)
    ```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application web monopage](../../tutorial-bing-entities-search-single-page-app.md)

* [Qu’est-ce que l’API Recherche d’entités Bing ?](../../overview.md )