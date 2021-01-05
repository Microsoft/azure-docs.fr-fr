---
title: Qu’est l’API Recherche d’entreprises locales Bing ?
titleSuffix: Azure Cognitive Services
description: L’API Recherche d’entreprises locales Bing est un service RESTful permettant à vos applications de rechercher des informations sur des lieux et des entreprises à proximité à l’aide de requêtes.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-local-business
ms.topic: overview
ms.date: 03/24/2020
ms.author: aahi
ms.openlocfilehash: 7a5829623707797b98593f837d6cadf009410f31
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96487012"
---
# <a name="what-is-bing-local-business-search"></a>Présentation de Recherche d’entreprises locales Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
L’API Recherche d’entreprises locales Bing est un service RESTful permettant à vos applications de rechercher des informations sur des entreprises locales à l’aide de requêtes. Par exemple, `q=<business-name> in Redmond, Washington` ou `q=Italian restaurants near me`. 

## <a name="features"></a>Fonctionnalités
| Fonctionnalité | Description |  
| -- | -- | 
| [Rechercher des entreprises et des emplacements locaux](quickstarts/local-quickstart.md) | L’API Recherche d’entreprises locales Bing obtient les résultats localisés d’une requête. Les résultats incluent une URL pour le site web de l’entreprise et présentent un texte, un numéro de téléphone et un emplacement géographique, notamment : les coordonnées GPS, la ville et l’adresse postale. |  
| [Filtrer les résultats locaux avec des limites géographiques](specify-geographic-search.md) | Ajoutez des coordonnées comme paramètres de recherche pour limiter les résultats à une zone géographique spécifique, selon une zone circulaire ou un carré. | 
| [Filtrer les résultats des entreprises locales par catégorie](local-categories.md) | Recherchez les résultats des entreprises locales par catégorie. Cette option utilise l’emplacement IP inverse ou les coordonnées GPS de l’appelant pour retourner des résultats localisés dans différentes catégories d’entreprise.|

## <a name="workflow"></a>Workflow
Appelez l’API Recherche d’entreprises locales Bing à partir de n’importe quel langage de programmation permettant d’exécuter des requêtes HTTP et d’analyser des réponses JSON. Ce service est accessible à l’aide de l’API REST.
 
1. Créez un [compte d’API Cognitive Services](../cognitive-services-apis-create-account.md) disposant d’un accès aux API Recherche Bing. Si vous n’avez pas d’abonnement Azure, vous pouvez [créer un compte gratuit](https://azure.microsoft.com/free/cognitive-services/).   
2. Encodez par URL vos termes de recherche pour le paramètre de requête `q=""`. Par exemple, `q=nearby+restaurant` ou `q=nearby%20restaurant`. Définissez également la pagination, si nécessaire. 
3. Envoyer une [API Recherche d’entreprises locales Bing](quickstarts/local-quickstart.md) 
4. Analyser la réponse JSON 

> [!NOTE]
> Actuellement, la recherche d’entreprise locale : 
> * Prend en charge uniquement le marché `en-US`. 
> * Ne prend pas en charge Suggestion automatique Bing. 

## <a name="next-steps"></a>Étapes suivantes
- [Requête et réponse](local-search-query-response.md)
- [Démarrage rapide de la Recherche d’entreprises locales](quickstarts/local-quickstart.md)
- [Informations de référence sur l’API Recherche d’entreprises locales](local-search-reference.md)
- [Conditions d’utilisation et d’affichage](../bing-web-search/use-display-requirements.md)