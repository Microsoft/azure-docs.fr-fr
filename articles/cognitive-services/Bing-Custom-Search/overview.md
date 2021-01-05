---
title: Qu’est-ce que l’API Recherche personnalisée Bing ?
titleSuffix: Azure Cognitive Services
description: L’API Recherche personnalisée Bing vous permet de créer des expériences de recherche adaptées aux sujets qui vous intéressent.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-custom-search
ms.topic: overview
ms.date: 12/18/2019
ms.author: aahi
ms.openlocfilehash: e70d4d83fcd9641fa86f013688b5a3a6e3652919
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96338399"
---
# <a name="what-is-the-bing-custom-search-api"></a>Qu’est-ce que l’API Recherche personnalisée Bing ?

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Recherche personnalisée Bing vous permet de créer des expériences de recherche exemptes de publicité, adaptées aux sujets qui vous intéressent. Vous pouvez spécifier les domaines et pages web dans lesquels Bing doit effectuer ses recherches, également épingler, élever ou rétrograder un contenu précis pour créer une vue personnalisée du web qui permet à vos utilisateurs d’obtenir rapidement des résultats pertinents par rapport à leurs recherches. 

## <a name="features"></a>Fonctionnalités

|Fonctionnalité  |Description  |
|---------|---------|
|[Suggestions personnalisées de recherche en temps réel](define-custom-suggestions.md)     | Fournir des suggestions de recherche qui peuvent être affichées sous forme de liste déroulante, au fur et à mesure que l’utilisateur tape un texte au clavier.       | 
|[Expériences personnalisées de recherche d’image](get-images-from-instance.md)     | Autoriser les utilisateurs à rechercher des images dans les domaines et sites web spécifiés dans votre instance de recherche personnalisée.        |        
|[Expériences personnalisées de recherche de vidéo](get-videos-from-instance.md)     | Autoriser les utilisateurs à rechercher des vidéos dans les domaines et sites web spécifiés dans votre instance de recherche personnalisée.        |    
|[Partager votre instance de recherche personnalisée](share-your-custom-search.md)     | Modifier et tester en collaboration votre instance de recherche en la partageant avec des membres de votre équipe.        | 
|[Configurer une interface utilisateur pour vos applications et sites web](hosted-ui.md)     | Fournit une interface utilisateur hébergée que vous pouvez facilement intégrer à vos applications et pages web en tant qu’extrait de code JavaScript.        | 
## <a name="workflow"></a>Workflow

Vous pouvez créer une instance de recherche personnalisée par l’intermédiaire du [portail Recherche personnalisée Bing](https://customsearch.ai). Ce portail vous permet de créer une instance de recherche personnalisée qui spécifie les domaines, sites web et pages web que Bing doit utiliser pour ses recherches, et ceux qu’il ne doit pas utiliser. Vous pouvez également utiliser le portail pour afficher un aperçu de l’expérience de recherche, ajuster les classements de recherche que fournit l’API et, au besoin, configurer une interface utilisateur de recherche affichable dans vos applications et vos sites web.

Après avoir créé votre instance de recherche, vous pouvez l’intégrer (avec une interface utilisateur si nécessaire) à votre application ou votre site web en appelant l’API Recherche personnalisée Bing :

![Image illustrant la possibilité de se connecter à la Recherche personnalisée Bing via l’API](media/BCS-Overview.png "Fonctionnement de la Recherche personnalisée Bing.")


## <a name="next-steps"></a>Étapes suivantes

Pour bien démarrer rapidement, consultez [Créer votre première instance Recherche personnalisée Bing](quick-start.md).

Pour plus d’informations sur la personnalisation de votre instance de recherche, consultez [Définir une instance de recherche personnalisée](define-your-custom-view.md).

Prenez soin de lire les [Conditions d’utilisation et d’affichage Bing](../bing-web-search/use-display-requirements.md) pour utiliser des résultats de recherche dans vos services et applications.

Visitez la [page hub de l’API Recherche Bing](../bing-web-search/overview.md) pour explorer les autres API disponibles.

Familiarisez-vous avec le contenu de référence pour chacun des points de terminaison de la recherche personnalisée. Vous y trouverez les points de terminaison, les en-têtes et les paramètres de requête à utiliser pour demander les résultats de la recherche. Vous y trouverez également les définitions des objets de réponse.

[!INCLUDE [cognitive-services-bing-url-note](../../../includes/cognitive-services-bing-url-note.md)]

- [API Recherche personnalisée](/rest/api/cognitiveservices-bingsearch/bing-custom-search-api-v7-reference)
- [API Image personnalisée](/rest/api/cognitiveservices-bingsearch/bing-custom-images-api-v7-reference)
- [API Vidéo personnalisée](/rest/api/cognitiveservices-bingsearch/bing-custom-videos-api-v7-reference)
- [API Suggestion automatique personnalisée](/rest/api/cognitiveservices-bingsearch/bing-custom-autosuggest-api-v7-reference)