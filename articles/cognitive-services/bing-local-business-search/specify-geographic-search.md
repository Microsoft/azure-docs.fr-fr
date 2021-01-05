---
title: Utiliser les limites géographiques pour filtrer les résultats de l’API Recherche d’entreprises locales Bing
titleSuffix: Azure Cognitive Services
description: Lisez cet article pour apprendre à filtrer les résultats de recherche de l’API Recherche d’entreprises locales Bing.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-local-business
ms.topic: conceptual
ms.date: 11/01/2018
ms.author: rosh
ms.openlocfilehash: 2e72abd777ac4834385fd916b730d438b8540203
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96490259"
---
# <a name="use-geographic-boundaries-to-filter-results-from-the-bing-local-business-search-api"></a>Utiliser les limites géographiques pour filtrer les résultats de l’API Recherche d’entreprises locales Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Recherche d’entreprises locales Bing vous permet de définir des limites sur la zone géographique dans laquelle vous effectuez une recherche à l’aide du paramètre de requête `localCircularView` ou `localMapView`. N’utilisez qu’un seul paramètre dans vos requêtes. 

Si l’un des critères de recherche contient un emplacement géographique explicite, l’API Recherche d’entreprises locales Bing l’utilise automatiquement pour définir les limites des résultats de recherche. Par exemple, si le terme recherché est `sailing in San Diego`, `San Diego` sera utilisé comme emplacement ; tous les autres emplacements spécifiés dans les paramètres de requête et les en-têtes de l’utilisateur seront ignorés. 

Si aucun emplacement géographique n’est détecté dans le critère de recherche ou spécifié à l’aide des paramètres de requête, l’API Recherche d’entreprises locales Bing tente de déterminer l’emplacement à partir des en-têtes `X-Search-ClientIP` et `X-Search-Location` de la demande. Si aucun de ces en-têtes n’est spécifié, l’API déterminera l’emplacement à partir de l’IP client de la demande, ou des coordonnées GPS pour les appareils mobiles.

## <a name="localcircularview"></a>localCircularView

Le paramètre `localCircularView` crée une zone géographique circulaire autour d’un ensemble de coordonnées latitude/longitude, définie par un rayon. Lorsque ce paramètre est utilisé, les réponses de l’API Recherche d’entreprises locales Bing ne comportent que des emplacements se trouvant dans ce cercle, à la différence du paramètre `localMapView` qui peut inclure des lieux proches de la zone de recherche.

Pour spécifier une zone de recherche géographique circulaire, choisissez une latitude et une longitude qui représenteront le centre du cercle, ainsi qu’un rayon en mètres. Ce paramètre peut ensuite être ajouté à la chaîne de requête, par exemple : `q=Restaurants&localCircularView=47.6421,-122.13715,5000`.

Requête complète :

```
https://api.cognitive.microsoft.com/bing/v7.0/localbusinesses/search?q=restaurant&localCircularView=47.6421,-122.13715,5000&appid=0123456789ABCDEF&mkt=en-us&form=monitr
```

## <a name="localmapview"></a>localMapView

Le paramètre `localMapView` spécifie une zone de recherche géographique rectangulaire à l’aide de deux jeux de coordonnées qui représentent son coin sud-est et son coin nord-ouest. Lorsque ce paramètre est utilisé, les réponses de l’API Recherche d’entreprises locales Bing peuvent comporter des emplacements se trouvant dans la zone spécifiée et légèrement en dehors, à la différence du paramètre `localCircularView` qui n’inclut que des lieux situés dans la zone de recherche.

Pour spécifier une zone de recherche rectangulaire, choisissez deux jeux de coordonnées latitude/longitude qui représenteront le coin sud-est et le coin nord-ouest de la limite. Commencez par définir les coordonnées sud-est, comme dans l’exemple suivant : `localMapView=47.619987,-122.181671,47.6421,-122.13715`.

Requête complète :

```
https://api.cognitive.microsoft.com/bing/v7.0/localbusinesses/search?q=restaurant&localMapView=47.619987,-122.181671,47.6421,-122.13715&appid=0123456789ABCDEF&mkt=en-us&form=monitr
```

## <a name="next-steps"></a>Étapes suivantes
- [Démarrage rapide Java de la Recherche d’entreprises locales](quickstarts/local-search-java-quickstart.md)
- [Démarrage rapide C# de la Recherche d’entreprises locales](quickstarts/local-quickstart.md)
- [Démarrage rapide Node de la Recherche d’entreprises locales](quickstarts/local-search-node-quickstart.md)
- [Démarrage rapide Python de la Recherche d’entreprises locales](quickstarts/local-search-python-quickstart.md)