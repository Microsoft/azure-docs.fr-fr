---
title: 'Démarrage rapide : Effectuer une recherche avec Python - API Recherche Web Bing'
titleSuffix: Azure Cognitive Services
description: Utilisez ce démarrage rapide pour envoyer des requêtes à l'API REST Recherche Web Bing à l'aide de Python et recevoir une réponse JSON
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: quickstart
ms.date: 05/22/2020
ms.author: aahi
ms.custom: seodec2018, devx-track-python
ms.openlocfilehash: 5a9eff1ef012e66f661ee99852a8d82f117a45c4
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350515"
---
# <a name="quickstart-use-python-to-call-the-bing-web-search-api"></a>Démarrage rapide : Utiliser Python pour appeler l'API Recherche Web Bing  

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Utilisez ce guide de démarrage rapide pour effectuer votre premier appel à l’API Recherche Web Bing. Cette application Python envoie une requête de recherche à l’API et affiche la réponse JSON. Bien que cette application soit écrite en Python, l’API est un service web RESTful compatible avec la plupart des langages de programmation.

Cet exemple est exécuté en tant que bloc-notes Jupyter sur [MyBinder](https://mybinder.org). Pour l’exécuter, sélectionnez le badge de lancement de Binder :

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/Microsoft/cognitive-services-notebooks/master?filepath=BingWebSearchAPI.ipynb)

## <a name="prerequisites"></a>Prérequis

* [Python 2.x ou 3.x](https://www.python.org/)

[!INCLUDE [bing-web-search-quickstart-signup](../../../../includes/bing-web-search-quickstart-signup.md)]

## <a name="define-variables"></a>Définir des variables

1. Remplacez la valeur `subscription_key` par une clé d’abonnement valide à partir de votre compte Azure.

   ```python
   subscription_key = "YOUR_ACCESS_KEY"
   assert subscription_key
   ```

2. Déclarez le point de terminaison de l’API Recherche Web Bing. Vous pouvez utiliser le point de terminaison global dans le code suivant, ou le point de terminaison de [sous-domaine personnalisé](../../../cognitive-services/cognitive-services-custom-subdomains.md) affiché dans le portail Azure pour votre ressource.

   ```python
   search_url = "https://api.cognitive.microsoft.com/bing/v7.0/search"
   ```

3. Si vous le souhaitez, vous pouvez personnaliser la requête de recherche en remplaçant la valeur de `search_term`.

   ```python
   search_term = "Azure Cognitive Services"
   ```

## <a name="make-a-request"></a>Exécuter une requête

Ce code utilise la bibliothèque `requests` pour appeler l’API Recherche Web Bing et retourner les résultats en tant qu’objet JSON. La clé API est transmise au dictionnaire `headers`, tandis que les termes de recherche et les paramètres de requête sont transmis au dictionnaire `params`. 

Pour obtenir la liste complète des options et paramètres, consultez [API Recherche Web Bing v7](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference).

```python
import requests

headers = {"Ocp-Apim-Subscription-Key": subscription_key}
params = {"q": search_term, "textDecorations": True, "textFormat": "HTML"}
response = requests.get(search_url, headers=headers, params=params)
response.raise_for_status()
search_results = response.json()
```

## <a name="format-and-display-the-response"></a>Mettre en forme et afficher la réponse

L’objet `search_results` inclut les résultats de la recherche ainsi que des métadonnées telles que les pages et les requêtes associées. Ce code utilise la bibliothèque `IPython.display` pour mettre en forme et afficher la réponse dans votre navigateur.

```python
from IPython.display import HTML

rows = "\n".join(["""<tr>
                       <td><a href=\"{0}\">{1}</a></td>
                       <td>{2}</td>
                     </tr>""".format(v["url"], v["name"], v["snippet"])
                  for v in search_results["webPages"]["value"]])
HTML("<table>{0}</table>".format(rows))
```

## <a name="sample-code-on-github"></a>Exemple de code sur GitHub

Pour exécuter ce code localement, consultez l’[exemple complet disponible sur GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/python/Search/BingWebSearchv7.py).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel sur l’application d’API Recherche Web Bing monopage](../tutorial-bing-web-search-single-page-app.md)

[!INCLUDE [bing-web-search-quickstart-see-also](../../../../includes/bing-web-search-quickstart-see-also.md)]