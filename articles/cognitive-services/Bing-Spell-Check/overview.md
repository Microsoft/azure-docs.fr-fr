---
title: Qu’est-ce que l’API Vérification orthographique Bing ?
titleSuffix: Azure Cognitive Services
description: Apprenez-en davantage sur l’API Vérification orthographique Bing qui utilise l’apprentissage automatique et la traduction automatique statistique pour vérifier l’orthographe en contexte.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-spell-check
ms.topic: overview
ms.date: 12/19/2019
ms.author: aahi
ms.openlocfilehash: be864e04143a381a08dedb5693026f05c42c5421
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349297"
---
# <a name="what-is-the-bing-spell-check-api"></a>Qu’est-ce que l’API Vérification orthographique Bing ?

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Vérification orthographique Bing vous permet d’effectuer une vérification orthographique et grammaticale contextuelle sur du texte. Bien que la plupart des vérificateurs d’orthographe s’appuient sur des ensembles de règles basées sur un dictionnaire, le vérificateur d’orthographe Bing exploite l’apprentissage automatique et la traduction automatique statistique pour fournir des corrections précises et contextuelles. 

## <a name="features"></a>Fonctionnalités

| Fonctionnalité | Description |
|---------|---------|
|Plusieurs modes de vérification orthographique     | Plusieurs modes de vérification orthographique vous permettent d’obtenir des corrections axées sur la grammaire et/ou l’orthographe. |
|Reconnaissance de l’argot et du langage informel     | Reconnaissez des expressions courantes et des termes informels utilisés dans le texte.         |
|Distinction entre des mots similaires     | Apprenez à bien utiliser des mots qui semblent similaires, mais ont une signification différente (par exemple, « mère » et « mer »)        |
|Prise en charge de marques, de titres et de l’usage courant     | Reconnaissez les nouvelles marques, les titres et les dernières expressions populaires |

## <a name="workflow"></a>Workflow

L’API Vérification orthographique Bing peut être facilement appelée à partir de n’importe quel langage de programmation permettant d’exécuter des demandes HTTP et d’analyser des réponses JSON. Ce service est accessible à l’aide de l’API REST ou des kits SDK Vérification orthographique Bing. 

1. Créez un [compte d’API Cognitive Services](../cognitive-services-apis-create-account.md) disposant d’un accès aux API Recherche Bing. Si vous n’avez pas d’abonnement Azure, vous pouvez créer un compte gratuit. 
2. Envoyez une demande à l’API Recherche Web Bing.
3. Analyser la réponse JSON

## <a name="next-steps"></a>Étapes suivantes

Essayez tout d’abord la [démonstration interactive](https://azure.microsoft.com/services/cognitive-services/spell-check/) de l’API Vérification orthographique Bing pour voir comment vous pouvez vérifier rapidement différents textes.

Lorsque vous êtes prêt à appeler l’API, créez un [compte d’API Cognitive Services](../../cognitive-services/cognitive-services-apis-create-account.md). Si vous n’avez pas d’abonnement Azure, vous pouvez créer un [compte](https://azure.microsoft.com/free/cognitive-services/) gratuitement.

Vous pouvez également consulter la [page hub de l’API Recherche Bing](../bing-web-search/overview.md) pour explorer les autres API disponibles.