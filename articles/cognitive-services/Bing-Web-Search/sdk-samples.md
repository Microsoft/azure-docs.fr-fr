---
title: Exemples du kit SDK Recherche Web Bing
titleSuffix: Azure Cognitive Services
description: Utilisez le kit SDK Recherche Web Bing pour ajouter des fonctionnalités de recherche à votre application Python, Node.js, C# ou Java.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: sample
ms.date: 05/15/2019
ms.author: aahi
ms.openlocfilehash: f13250bb2d6829cf7f1bf3dd0417974b810a36fe
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350430"
---
# <a name="bing-web-search-sdk-samples"></a>Exemples du kit SDK Recherche Web Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Le kit de développement logiciel (SDK) Recherche Web Bing est disponible en Python, Node.js, C# et Java. Vous trouverez les exemples de code, les prérequis et les instructions de génération sur GitHub. Les scénarios abordés sont les suivants :

* Exécuter une requête sur un mot unique et imprimer le nom et l’URL du premier résultat pour les pages web, les images, les articles d’infos et les vidéos.
* Exécuter une requête sur une expression, vérifier le nombre de résultats, puis imprimer le nom et l’URL du premier résultat.
* Exécuter une requête sur un terme de recherche avec les filtres de réponse définis sur `news`, puis imprimer les détails des résultats d’articles d’infos.
* Exécuter une requête sur un terme de recherche avec les paramètres `answerCount` et `promote`, puis imprimer les détails des résultats.

## <a name="sdks-and-libraries"></a>Kits SDK et bibliothèques

Utilisez ces liens pour accéder au kit SDK correspondant au langage de votre choix.

* Bien commencer avec les [exemples Python](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples)
  * Voir aussi les [bibliothèques Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-search-websearch) pour les définitions et les dépendances.
* Bien commencer avec les [exemples Node.js](https://github.com/Azure-Samples/cognitive-services-node-sdk-samples)
  * Consultez aussi [Recherche web Node.js](https://github.com/Azure/azure-sdk-for-node/tree/master/lib/services/cognitiveServicesWebSearch).
* Bien commencer avec les [exemples .NET](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7)
  * [Package NuGet](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.WebSearch/1.2.0)
  * Voir aussi les [bibliothèques .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Search.BingWebSearch) pour les définitions et les dépendances.
* Bien commencer avec les [exemples Java](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples)
  * Voir aussi [bibliothèques Java](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master/Search/BingWebSearch) pour les définitions et les dépendances.