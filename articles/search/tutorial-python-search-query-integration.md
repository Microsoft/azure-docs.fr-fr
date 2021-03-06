---
title: 'Tutoriel Python : Principales caractéristiques de l’intégration de la recherche'
titleSuffix: Azure Cognitive Search
description: Découvrez les requêtes d’intégration de la recherche du kit SDK Python utilisées dans le site web avec fonctionnalité de recherche grâce à cet aide-mémoire.
manager: nitinme
author: diberry
ms.author: diberry
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 05/25/2021
ms.custom: devx-track-python
ms.devlang: python
ms.openlocfilehash: a8f1078e714bd88d2b2e9911a9e1708c21aa91b2
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/27/2021
ms.locfileid: "110580583"
---
# <a name="4---python-search-integration-cheat-sheet"></a>4 - Aide-mémoire de l’intégration de la recherche Python

Dans les leçons précédentes, vous avez ajouté la recherche à une application web statique. Cette leçon met en évidence les étapes essentielles qui établissent l’intégration. Si vous recherchez un aide-mémoire sur l’intégration de la recherche dans votre application Python, cet article explique ce que vous devez savoir.

L’application est disponible : 
* [Exemple](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/search-website)
* [Site web de démonstration - aka.ms/azs-good-books](https://aka.ms/azs-good-books)

## <a name="azure-sdk-azure-search-documents"></a>Kit SDK Azure : azure-search-documents

L’application de fonction utilise le Kit de développement logiciel (SDK) Azure pour Recherche cognitive :

* [Package PYPI : azure-search-documents](https://pypi.org/project/azure-search-documents/)
* [Documentation de référence](/python/api/azure-search-documents)

L’application de fonction s’authentifie via le kit SDK auprès de l’API cloud Recherche cognitive à l’aide du nom de la ressource, de la clé API et du nom de l’index. Les secrets sont stockés dans les paramètres de l’application web statique et extraits dans Azure Function en tant que variables d’environnement. 

## <a name="configure-secrets-in-a-configuration-file"></a>Configurer des secrets dans un fichier de configuration

Les variables d’environnement des paramètres de l’application de fonction Azure sont extraites à partir d’un fichier, `__init__.py`, et sont partagées entre les trois fonctions API. 

:::code language="python" source="~/azure-search-python-samples/search-website/api/shared_code/__init__.py" highlight="6-9" :::

## <a name="azure-function-search-the-catalog"></a>Azure Function : Rechercher dans le catalogue

L’[API](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/api/Search/__init__.py) Recherche accepte un terme de recherche et effectue la recherche parmi les documents dans l’index de recherche, en retournant une liste de correspondances. 

Le routage de l’API Search est contenu dans les liaisons [function.json](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/api/Search/function.json).

Azure Functions extrait les informations de configuration de la recherche et exécute la requête.

:::code language="python" source="~/azure-search-python-samples/search-website/api/Search/__init__.py" highlight="8-18, 122" :::

## <a name="client-search-from-the-catalog"></a>Client : recherche dans le catalogue

Appelez Azure Function dans le client React à l’aide du code suivant. 

:::code language="javascript" source="~/azure-search-python-samples/search-website/src/pages/Search/Search.js" highlight="42-55" :::

## <a name="azure-function-suggestions-from-the-catalog"></a>Azure Function : suggestions à partir du catalogue

L’[API](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/api/Suggest/__init__.py) `Suggest` prend un terme de recherche lors de la saisie et suggère des termes de recherche tels que des titres et des auteurs de livres dans les documents de l’index de recherche, renvoyant une petite liste de correspondances. 

Le générateur de suggestions de recherche, `sg`, est défini dans le [fichier de schéma](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/bulk-upload/good-books-index.json) utilisé lors du chargement en bloc.

Le routage de l’API Suggest est contenu dans les liaisons [function.json](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/api/Suggest/function.json).

:::code language="python" source="~/azure-search-python-samples/search-website/api/Suggest/__init__.py" highlight="8-23, 35" :::

## <a name="client-suggestions-from-the-catalog"></a>Client : suggestions à partir du catalogue

L’API de fonction Suggest est appelée dans l’application React au niveau de `\src\components\SearchBar\SearchBar.js` dans le cadre de l’initialisation du composant :

:::code language="javascript" source="~/azure-search-python-samples/search-website/src/components/SearchBar/SearchBar.js" highlight="52-60" :::

## <a name="azure-function-get-specific-document"></a>Azure Function : accéder à un document spécifique 

L’[API](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/api/Lookup/__init__.py) `Lookup` prend un ID et retourne l’objet de document à partir de l’index de recherche. 

Le routage de l’API Lookup est contenu dans les liaisons [function.json](https://github.com/Azure-Samples/azure-search-python-samples/blob/master/search-website/api/Lookup/function.json).

:::code language="python" source="~/azure-search-python-samples/search-website/api/Lookup/__init__.py" highlight="8-18, 27" :::

## <a name="client-get-specific-document"></a>Client : accéder à un document spécifique 

Cette API de fonction est appelée dans l’application React à l’emplacement `\src\pages\Details\Detail.js` dans le cadre de l’initialisation du composant :

:::code language="javascript" source="~/azure-search-python-samples/search-website/src/pages/Details/Details.js" highlight="19-29" :::

## <a name="next-steps"></a>Étapes suivantes

* [Indexer des données Azure SQL](search-indexer-tutorial.md)
