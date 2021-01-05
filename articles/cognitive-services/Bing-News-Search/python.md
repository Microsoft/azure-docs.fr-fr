---
title: 'Démarrage rapide : Effectuer une recherche d’actualités à l’aide de Python et l’API REST Recherche d’actualités Bing'
titleSuffix: Azure Cognitive Services
description: Utilisez ce démarrage rapide pour envoyer une requête à l’API REST Recherche d’actualités Bing à l’aide de Python et recevez une réponse JSON.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-news-search
ms.topic: quickstart
ms.date: 06/16/2020
ms.author: aahi
ms.custom: seodec2018, devx-track-python
ms.openlocfilehash: f47c0c5ad271b89348fe0baa8ac1fd5ebd9cf2eb
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96351263"
---
# <a name="quickstart-perform-a-news-search-using-python-and-the-bing-news-search-rest-api"></a>Démarrage rapide : Effectuer une recherche d’actualités à l’aide de Python et l’API REST Recherche d’actualités Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Utilisez ce guide de démarrage rapide pour effectuer votre premier appel à l’API Recherche d’actualités Bing. Cette application Python simple envoie une requête de recherche à l’API et traite le résultat JSON. 

Bien que cette application soit écrite en Python, l’API est un service web RESTful compatible avec la plupart des langages de programmation.

Pour exécuter cet exemple de code en tant que notebook Jupyter sur [MyBinder](https://mybinder.org), sélectionnez le badge de **lancement de Binder** : 

[![Lancer Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/Microsoft/cognitive-services-notebooks/master?filepath=BingNewsSearchAPI.ipynb)

Le code source de cet exemple est également disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/python/Search/BingNewsSearchv7.py).

[!INCLUDE [cognitive-services-bing-news-search-signup-requirements](../../../includes/cognitive-services-bing-news-search-signup-requirements.md)]

## <a name="create-and-initialize-the-application"></a>Créer et initialiser l’application

Créez un fichier Python dans votre éditeur ou IDE favori, puis importez le module de requête. Créez des variables pour votre clé d’abonnement, le point de terminaison et le terme de recherche. Vous pouvez utiliser le point de terminaison global dans le code suivant, ou le point de terminaison de [sous-domaine personnalisé](../../cognitive-services/cognitive-services-custom-subdomains.md) affiché dans le portail Azure pour votre ressource.

```python
import requests

subscription_key = "your subscription key"
search_term = "Microsoft"
search_url = "https://api.cognitive.microsoft.com/bing/v7.0/news/search"
```

## <a name="create-parameters-for-the-request"></a>Créer les paramètres de la requête

Ajoutez votre clé d’abonnement à un nouveau dictionnaire en utilisant `Ocp-Apim-Subscription-Key` comme clé. Faites de même pour vos paramètres de recherche.

```python
headers = {"Ocp-Apim-Subscription-Key" : subscription_key}
params  = {"q": search_term, "textDecorations": True, "textFormat": "HTML"}
```

## <a name="send-a-request-and-get-a-response"></a>Envoyer une requête et obtenir une réponse

1. Utilisez la bibliothèque de requêtes pour appeler l’API Recherche visuelle Bing avec votre clé d’abonnement et les objets de dictionnaire que vous avez créés à l’étape précédente.

    ```python
    response = requests.get(search_url, headers=headers, params=params)
    response.raise_for_status()
    search_results = json.dumps(response.json())
    ```

2. Accédez aux descriptions des articles contenus dans la réponse de l’API, qui est stockée dans `search_results` sous la forme d’un objet JSON. 
    
    ```python
    descriptions = [article["description"] for article in search_results["value"]]
    ```

## <a name="display-the-results"></a>Afficher les résultats

Ces descriptions peuvent ensuite être restituées sous la forme d’un tableau, avec le mot clé de recherche surligné en gras.

```python
from IPython.display import HTML
rows = "\n".join(["<tr><td>{0}</td></tr>".format(desc)
                  for desc in descriptions])
HTML("<table>"+rows+"</table>")
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application web monopage](tutorial-bing-news-search-single-page-app.md)