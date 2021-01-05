---
title: 'Démarrage rapide : Utiliser la bibliothèque cliente Recherche personnalisée Bing pour Python'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 02/27/2020
ms.author: aahi
ms.openlocfilehash: 92df45e38a926b4a791315f731e1d1cb078d5852
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94371679"
---
Commencez à utiliser la bibliothèque cliente Recherche personnalisée Bing pour Python. Suivez les étapes suivantes pour installer le package et essayer l’exemple de code pour les tâches de base. L’API Recherche personnalisée Bing vous permet de créer des expériences de recherche exemptes de publicité, adaptées aux sujets qui vous intéressent. Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples/blob/master/samples/search/custom_search_samples.py).

Utilisez la bibliothèque cliente Recherche personnalisée Bing pour Python pour :
* Rechercher des résultats de recherche sur le web à partir de votre instance Recherche personnalisée Bing.

[Documentation de référence](/python/api/azure-cognitiveservices-search-customsearch/?view=azure-python) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-search-customsearch) | [Package (PyPi)](https://pypi.org/project/azure-cognitiveservices-search-customsearch/) | [Exemples](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples/)


## <a name="prerequisites"></a>Prérequis

- Une instance Recherche personnalisée Bing. Consultez [Démarrage rapide : Créer votre première instance Recherche personnalisée Bing](../../quick-start.md) pour plus amples informations.
- Python[ 2.x ou 3.x](https://www.python.org/) 
- [SDK Recherche de vidéos Bing pour Python](https://pypi.org/project/azure-cognitiveservices-search-customsearch/) 

[!INCLUDE [cognitive-services-bing-custom-search-prerequisites](~/includes/cognitive-services-bing-custom-search-signup-requirements.md)]

## <a name="install-the-python-client-library"></a>Installer la bibliothèque cliente Python

Installez la bibliothèque cliente Recherche personnalisée Bing à l’aide de la commande suivante.

```Console
python -m pip install azure-cognitiveservices-search-customsearch
```


## <a name="create-a-new-application"></a>Créer une application

Créez un fichier Python dans votre éditeur ou IDE favori, puis ajoutez les importations suivantes.

```python
from azure.cognitiveservices.search.customsearch import CustomSearchClient
from msrest.authentication import CognitiveServicesCredentials
```

## <a name="create-a-search-client-and-send-a-request"></a>Créer un client de recherche et envoyer une requête

1. Créez une variable pour votre clé d’abonnement et votre point de terminaison.

    ```python
    subscription_key = 'your-subscription-key'
    endpoint = 'your-endpoint'
    ```

2. Créez une instance de `CustomSearchClient` utilisant un objet `CognitiveServicesCredentials` à l’aide de la clé d’abonnement. 

    ```python
    client = CustomSearchClient(endpoint=endpoint, credentials=CognitiveServicesCredentials(subscription_key))
    ```

3. Envoyez une requête de recherche avec `client.custom_instance.search()`. Ajoutez votre terme de recherche au paramètre `query`, et définissez `custom_config` sur votre ID de configuration personnalisée pour utiliser votre instance de recherche. Vous pouvez obtenir votre identifiant sur le portail [Recherche personnalisée Bing](https://www.customsearch.ai/), en cliquant sur l'onglet **Production**.

    ```python
    web_data = client.custom_instance.search(query="xbox", custom_config="your-configuration-id")
    ```

## <a name="view-the-search-results"></a>Afficher les résultats de la recherche

Si la requête renvoie des résultats de recherche sur une page web, obtenez le premier et imprimez son nom, son URL et le nombre total de pages web trouvées.

```python
if web_data.web_pages.value:
    first_web_result = web_data.web_pages.value[0]
    print("Web Pages result count: {}".format(len(web_data.web_pages.value)))
    print("First Web Page name: {}".format(first_web_result.name))
    print("First Web Page url: {}".format(first_web_result.url))
else:
    print("Didn't see any web data..")
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application web Recherche personnalisée](../../tutorials/custom-search-web-page.md)