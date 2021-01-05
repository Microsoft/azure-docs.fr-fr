---
title: Exemples d’insights Bing - Recherche visuelle Bing
titleSuffix: Azure Cognitive Services
description: Cet article contient des exemples de la manière dont la recherche visuelle utilise et présente des insights d’images sur Bing.com.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: conceptual
ms.date: 04/03/2019
ms.author: scottwhi
ms.openlocfilehash: e0a3bdc7e68301c067b7e1ca80a3e708522e0df1
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96493353"
---
# <a name="examples-of-bing-insights-usage"></a>Exemples d’utilisation d’insights de Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Cet article contient des exemples d’utilisation et d’affichage d’insights relatifs aux images sur Bing.com.

## <a name="pagesincluding-insight-example"></a>Exemple d’insight PagesIncluding

L’exemple suivant affiche un lien vers la première page web et permet à l’utilisateur de développer et de réduire la liste des autres pages web qui contiennent l’image :

![Pages développés incluant](./media/pages-including.PNG)

## <a name="shoppingsources-insight-example"></a>Exemple d’insight ShoppingSources

L’exemple suivant montre comment Bing peut afficher des sources d’achat des produits illustrés dans l’image :

![Sources d’achat](./media/shopping-sources.PNG)

## <a name="visualsearch-insight-example"></a>Exemple d’insight VisualSearch

L’exemple suivant montre comment Bing peut afficher des images visuellement similaires (consultez **Images liées** dans l’exemple) :

![Images visuellement similaires](./media/similar-images.PNG)

## <a name="recipes-insight-example"></a>Exemple d’insight Recipes

L’exemple suivant montre comment Bing peut afficher des recettes pour les produits alimentaires illustrés dans l’image. L’exemple informe l’utilisateur que des recettes sont disponibles :

![Recettes et pages incluant](./media/recipes-pages-including.PNG)

 Et fournit le lien vers les recettes lorsque l’utilisateur développe la liste :

![Pages de recettes développées incluant](./media/expanded-recipes-pages-including.PNG)

## <a name="relatedsearches-insight-example"></a>Exemple d’insight RelatedSearches

L’exemple suivant montre comment Bing peut afficher les recherches connexes d’images effectuées par d’autres utilisateurs. Si l’utilisateur clique sur l’image, il est dirigé vers la page des résultats de la recherche Bing.com/images pour cette requête liée.

![Recherches connexes d’images](./media/bordered-related-searches.PNG)

## <a name="entity-insight-example"></a>Exemple d’insight Entity

L’exemple suivant montre comment Bing peut afficher des informations sur l’entité (personne, emplacement ou chose) illustrée dans l’image. Si l’utilisateur clique sur l’entité, il est dirigé vers la page des résultats de la recherche Bing.com pour l’entité :

![Entité illustrée dans l’image](./media/entity.PNG)

## <a name="displaying-other-insights-that-the-user-might-explore"></a>Afficher d’autres insights que l’utilisateur peut explorer

L’exemple suivant montre comment Bing peut afficher d’autres informations sur l’image que l’utilisateur peut explorer.

![Explorer d’autres insights sur l’image](./media/apple-pie-more-tags.PNG)

## <a name="bounding-boxes-and-hot-spots"></a>Rectangles englobants et zones réactives

Les balises autres que celles par défaut incluent le rectangle englobant qui identifie la zone d’intérêt dans l’image à laquelle la balise s’applique. Si le rectangle englobant n’identifie pas l’image entière, utilisez le rectangle englobant pour créer une zone réactive sur l’image. L’utilisateur peut cliquer sur la zone réactive pour obtenir des informations relatives au contenu figurant sous la zone réactive (ou rectangle). Par exemple, si l’image est une image relative à la mode, les résultats peuvent contenir des balises (et des rectangles englobants) pour les accessoires illustrés dans l’image, par exemple un sac, des bijoux, des foulards, et ainsi de suite. L’exemple suivant montre un rectangle de zone réactive pour les lunettes de soleil illustrées dans l’image :

![Rectangle englobant et zone réactive](./media/click-to-search.PNG)

## <a name="next-steps"></a>Étapes suivantes

Pour configurer votre première demande, consultez ces guides de démarrage rapide :

* [C#](quickstarts/csharp.md)

* [Java](quickstarts/java.md)

* [node.js](quickstarts/nodejs.md)

* [Python](quickstarts/python.md)