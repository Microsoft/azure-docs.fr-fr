---
title: Présentation des API Recherche Bing
titleSuffix: Azure Cognitive Services
description: Cet article présente les API Recherche Bing et explique comment activer les recherches Internet cognitives dans vos applications et vos services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 03/12/2019
ms.author: aahi
ms.openlocfilehash: 72b4755c6f01a10851e79cf274842f1599bc2c55
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349535"
---
# <a name="what-are-the-bing-search-apis"></a>Présentation des API Recherche Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Les API Recherche Bing servent à créer des applications et des services connectés au web, permettant de rechercher des pages web, des images, des actualités, des lieux, etc. sans publicités. En envoyant des requêtes de recherche à l’aide de l’API REST Recherche Bing ou des SDK, vous pouvez obtenir des informations et du contenu pertinents. Utilisez cet article pour découvrir les différentes API Recherche Bing et savoir comment intégrer les recherches cognitives à vos applications et vos services. Les limites concernant les tarifs et les taux d’utilisation peuvent varier selon les API.

## <a name="the-bing-web-search-api"></a>L’API Recherche Web Bing

L’[API Recherche Web Bing](../Bing-Web-Search/overview.md) retourne des pages web, des images, des vidéos, des actualités, et bien plus encore. Les requêtes de recherche envoyées à cette API peuvent être filtrées de manière à inclure ou exclure certains types de contenu.

Utilisez l’API Recherche Web Bing dans les applications qui devront rechercher tous les types de contenu web pertinents. Si votre application recherche un type spécifique de contenu en ligne, vous pouvez utiliser l’une des API de recherche suivantes :

## <a name="content-specific-bing-search-apis"></a>API Bing de recherche de contenu spécifique

L’API de recherche Bing suivante retourne du contenu spécifique tiré du Web, tel que des images, des actualités, des entreprises locales et des vidéos.

| API Bing | Description |
| -- | -- |
| [Recherche d’entités](../Bing-Entities-Search/overview.md) | L’API Recherche d’entités Bing retourne les résultats de recherche contenant des entités (personnes, lieux ou objets). En fonction de la requête, l’API retourne une ou plusieurs entités qui satisfont la requête de recherche. La requête de recherche peut inclure des personnes dignes d’intérêt, des entreprises locales, des points de repère, des destinations et bien plus encore. |
| [Recherche d’images](../Bing-Image-Search/overview.md) | L’API Recherche d’images Bing vous permet de trouver des images statiques et animées de haute qualité, similaires à celles de [Bing.com/images](https://www.Bing.com/images). Vous pouvez affiner la recherche de manière à inclure ou exclure des images en fonction d’un de leurs attributs (taille, couleur, licence ou actualité). Vous pouvez également rechercher des images populaires, charger des images pour obtenir des insights à leur sujet ou afficher des miniatures d’aperçu. |
| [Recherche d’actualités](../Bing-News-Search/search-the-web.md) | L’API Recherche d’actualités Bing vous permet de trouver des articles de presse similaires à ceux de [Bing.com/news](https://www.Bing.com/news). L’API retourne des articles issus de plusieurs sources ou appartenant à des domaines spécifiques. Vous pouvez rechercher des articles populaires ou des grands titres appartenant à plusieurs catégories. |
| [Recherche de vidéos](../Bing-Video-Search/overview.md) | L’API Recherche de vidéos Bing vous permet de rechercher des vidéos sur Internet. Trouvez des vidéos populaires, du contenu connexe et des miniatures d’aperçu. |
| [Recherche visuelle](../Bing-visual-search/overview.md) | Chargez une image ou utilisez une URL pour obtenir des informations pertinentes à son sujet, telles que des produits visuellement similaires, des images et des recherches associées. |
 [Recherche d’entreprise locale](../bing-local-business-search/overview.md) | L’API Recherche d’entreprises locales Bing permet à vos applications de rechercher les coordonnées d’entreprises à proximité à l’aide de requêtes. |

## <a name="the-bing-custom-search-api"></a>L’API Recherche personnalisée Bing

Le fait de créer une instance de recherche personnalisée avec l’API [Recherche personnalisée Bing](../Bing-Custom-Search/overview.md) vous permet de bénéficier d’une expérience de recherche axée uniquement sur le contenu et les sujets qui vous intéressent. Par exemple, une fois que vous aurez spécifié les domaines, les sites web et les pages web que Bing doit explorer, les résultats seront adaptés au contenu choisi par Recherche personnalisée Bing. Vous pouvez incorporer les API Suggestion automatique personnalisée, Image et Recherche de vidéos de Bing pour personnaliser encore davantage votre recherche.

## <a name="additional-bing-search-apis"></a>Autres API de recherche Bing

Les API de recherche Bing suivantes vous permettent d’améliorer votre expérience de recherche en les associant à d’autres API de recherche Bing.

| API | Description |
| -- | -- |
| [Suggestion automatique Bing](../Bing-Autosuggest/get-suggested-search-terms.md) | Améliorez l’expérience de recherche de votre application avec l’API Suggestion automatique Bing, qui retourne des suggestions de recherche en temps réel.  |
| [Statistiques Bing](bing-web-stats.md) | L’API Statistiques Bing fournit une analytique aux API de recherche Bing qu’utilise votre application. Dans cette analytique, figurent le volume d’appels, les principales chaînes de requête et la répartition géographique. |

## <a name="next-steps"></a>Étapes suivantes

* [Détail des tarifs](https://azure.microsoft.com/pricing/details/cognitive-services/search-api/) de l’API Recherche Bing
* L’article [Conditions d’utilisation et d’affichage de l’API Recherche Bing](./use-display-requirements.md) spécifie les utilisations acceptables du contenu et des informations obtenus par le biais des API Recherche Bing.