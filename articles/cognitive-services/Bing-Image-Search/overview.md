---
title: Qu'est-ce que l'API Recherche d’images Bing ?
titleSuffix: Azure Cognitive Services
description: L’API Recherche d’images Bing vous permet d’utiliser les fonctionnalités Bing de recherche cognitive d’images dans votre application. En envoyant des requêtes de recherche utilisateur avec cette API, vous pouvez obtenir et afficher des images pertinentes et de haute qualité comparables aux Images Bing.
services: cognitive-services
author: aahill
manager: nitinme
ms.assetid: 1446AD8B-A685-4F5F-B4AA-74C8E9A40BE9
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: overview
ms.date: 12/18/2019
ms.author: aahi
ms.custom: seodec2018
ms.openlocfilehash: c52098d86efe60ee524735e6158add5260a0757f
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96338229"
---
# <a name="what-is-the-bing-image-search-api"></a>Qu'est-ce que l'API Recherche d’images Bing ?

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Recherche d’images Bing vous permet d’utiliser les fonctionnalités Bing de recherche d’images dans votre application. En envoyant des requêtes de recherche à l’API, vous pouvez obtenir des images de haute qualité comparables à celles de [bing.com/images](https://www.bing.com/images).

Alors que l’API Recherche d’images Bing fournit seulement des images dans les résultats de recherche, vous pouvez combiner ou utiliser les autres [API Recherche Bing](../bing-web-search/bing-api-comparison.md) disponibles pour rechercher de nombreux types de contenu sur le web.

## <a name="bing-image-search-features"></a>Fonctionnalités de recherche d’images Bing

| Fonctionnalité                                                                                                                                                                                 | Description                                                                                                                                                            |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Suggestion de termes de recherche en temps réel](./concepts/bing-image-search-sending-queries.md) | Améliorez votre expérience d’application en utilisant [l’API Suggestion automatique Bing](../bing-autosuggest/get-suggested-search-terms.md) pour afficher des suggestions de termes de recherche pendant la saisie de la requête. |
| [Filtrage et restriction des résultats d’image](./concepts/bing-image-search-get-images.md)                       | Filtrez les images renvoyées par Bing en modifiant les paramètres de requête.                                                                                                       |
| [Rognage, redimensionnement et affichage des miniatures](../bing-web-search/resize-and-crop-thumbnails.md)                                                | Modifiez et affichez les miniatures des images renvoyées par le service Recherche d’images Bing.                                                                                      |
| [Création d’un tableau croisé dynamique à partir des requêtes de recherche utilisateur et élargissement des requêtes](./concepts/bing-image-search-sending-queries.md)               | Élargissez vos capacités de recherche en incluant et en affichant les suggestions de termes de recherche Bing dans les requêtes.                                                                    |
| [Obtention des images tendance](trending-images.md)                                                                     | Personnalisez une recherche afin d’obtenir les images tendance du monde entier.                                                                                                          |

## <a name="workflow"></a>Workflow

L’API Recherche d’images Bing est un service web RESTful et peut donc être facilement appelé à partir de n’importe quel langage de programmation permettant d’exécuter des requêtes HTTP et d’analyser des réponses JSON. Vous pouvez utiliser ce service soit par le biais de [l’API REST](./quickstarts/csharp.md), soit à l’aide du [Kit de développement logiciel (SDK)](./quickstarts/client-libraries.md?pivots=programming-language-csharp%253fpivots%253dprogramming-language-csharp).

1. Créez un [compte d’API Cognitive Services](../cognitive-services-apis-create-account.md) disposant d’un accès aux API Recherche Bing. Si vous n’avez pas d’abonnement Azure, vous pouvez créer un [compte](https://azure.microsoft.com/free/cognitive-services/) gratuitement.
2. Envoyez une requête à l’API avec une [requête de recherche](./concepts/bing-image-search-sending-queries.md) valide.
3. Traitez la réponse de l’API en analysant le message JSON renvoyé.

## <a name="next-steps"></a>Étapes suivantes

Commencez par essayer la [démonstration interactive](https://azure.microsoft.com/services/cognitive-services/bing-image-search-api/) de l’API Recherche d’images Bing.
Cette démonstration vous explique comment personnaliser rapidement une requête de recherche et parcourir le web à la recherche d’images.

Pour exécuter rapidement votre première requête d’API, vous pouvez apprendre à effectuer les opérations suivantes :

* [envoyez des requêtes de recherche à Bing](./quickstarts/csharp.md) à l’aide de l’API REST ou
* [demandez et filtrez](./quickstarts/client-libraries.md?pivots=programming-language-csharp%253fpivots%253dprogramming-language-csharp) les images renvoyées par Bing à l’aide du Kit de développement logiciel (SDK).

## <a name="see-also"></a>Voir aussi

* [Détail des prix](https://azure.microsoft.com/pricing/details/cognitive-services/search-api/) des API Recherche Bing. 

* La section de référence de l’[API Recherche d’images Bing v7](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference) contient des informations sur les points de terminaison, les en-têtes, les réponses et les paramètres de requête de l’API.

* L’article [Conditions d’utilisation et d’affichage de l’API Recherche Bing](../bing-web-search/use-display-requirements.md) spécifie les utilisations acceptables du contenu et des informations obtenus par le biais des API Recherche Bing.

* L’article [Obtention d’images à partir du web avec l’API Recherche d’images Bing](./concepts/bing-image-search-get-images.md) explique comment rechercher et obtenir des images à partir du web.

* L’article [Envoi et utilisation des requêtes de recherche](./concepts/bing-image-search-sending-queries.md) décrit comment créer et personnaliser des requêtes de recherche et comment leur ajouter un tableau croisé dynamique.

* Visitez la [page hub de l’API Recherche Bing](../bing-web-search/overview.md) pour explorer les autres API disponibles.