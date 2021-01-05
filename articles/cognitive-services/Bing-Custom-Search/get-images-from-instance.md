---
title: Obtenir des images à partir de votre vue personnalisée – Recherche personnalisée Bing
titleSuffix: Azure Cognitive Services
description: Vue d’ensemble de l’utilisation de la Recherche personnalisée Bing pour obtenir des images à partir de votre vue personnalisée du web.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-custom-search
ms.topic: conceptual
ms.date: 09/10/2018
ms.author: scottwhi
ms.openlocfilehash: 5025a68030f5dc3aec07d33af3f98370c8d64b87
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96338467"
---
# <a name="get-images-from-your-custom-view"></a>Obtenir des images à partir de votre vue personnalisée

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

La Recherche personnalisée d’images Bing vous permet d’enrichir votre expérience de recherche personnalisée avec des images. Comme les résultats web, la recherche personnalisée prend en charge la recherche d’images dans la liste de sites web de votre instance. Vous pouvez obtenir les images en utilisant l’API Recherche personnalisée d’images de Bing ou la fonctionnalité d’interface utilisateur hébergée. La fonctionnalité d’interface utilisateur hébergée est simple à utiliser et recommandée pour rendre votre expérience de recherche rapidement opérationnelle.  Pour plus d’informations sur la configuration de votre interface utilisateur hébergée visant à inclure des images, consultez [Configurer votre expérience d’interface utilisateur hébergée](hosted-ui.md).

Si vous souhaitez contrôler davantage l’affichage des résultats de recherche, vous pouvez utiliser l’API Recherche personnalisée d’images de Bing. Étant donné que l’appel de l’API ressemble à l’appel de l’API Recherche d’images Bing, consultez [Recherche d’images Bing](../Bing-Image-Search/overview.md) pour des exemples d’appel de l’API. Au préalable, familiarisez-vous avec le contenu des [Informations de référence sur l’API Recherche personnalisée d’images](/rest/api/cognitiveservices-bingsearch/bing-custom-images-api-v7-reference). Les principales différences sont les paramètres de requête pris en charge (vous devez inclure le paramètre de requête customConfig) et le point de terminaison auquel vous envoyez vos requêtes.

<!--
## Next steps

[Call your custom view](search-your-custom-view.md)
-->