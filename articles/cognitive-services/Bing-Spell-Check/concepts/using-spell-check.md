---
title: Utilisation de l’API Vérification orthographique Bing
titleSuffix: Azure Cognitive Services
description: Découvrez les modes, paramètres et autres informations Vérification orthographique Bing relatives à l’API.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-spell-check
ms.topic: conceptual
ms.date: 02/20/2019
ms.author: aahi
ms.openlocfilehash: 2031c31c6ea083452bbdbb95be74adf29be1f858
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349314"
---
# <a name="using-the-bing-spell-check-api"></a>Utilisation de l’API Vérification orthographique Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Consultez cet article pour savoir comment utiliser l’API Vérification orthographique Bing pour effectuer une vérification grammaticale et orthographique contextuelle. Bien que la plupart des vérificateurs d’orthographe s’appuient sur des ensembles de règles basées sur un dictionnaire, le vérificateur d’orthographe Bing exploite l’apprentissage automatique et la traduction automatique statistique pour fournir des corrections précises et contextuelles. 

## <a name="spell-check-modes"></a>Modes de vérification orthographique

L’API prend en charge deux modes de vérification, `Proof` et `Spell`.  Consultez les exemples [ici](https://azure.microsoft.com/services/cognitive-services/spell-check/).

### <a name="proof---for-documents"></a>Vérifier - pour les documents 

Le mode par défaut est `Proof`. Le mode de vérification `Proof` fournit des contrôles plus complets et ajoute des fonctionnalités de mise en majuscules, de ponctuation, etc., pour faciliter la création d’un document. Malheureusement, il est uniquement disponible pour les segments en-US (Anglais-États-Unis), es-ES (Espagnol), pt-BR (Portugais) (Remarque : uniquement en version bêta pour l’espagnol et le portugais). Pour tous les autres marchés, définissez le paramètre de requête « mode » sur Corriger l’orthographe. 

> [!NOTE]
> Si le texte de la requête contient plus de 4 096 caractères, il est tronqué à 4 096 caractères avant d’être traité. 

### <a name="spell----for-web-searchesqueries"></a>Corriger l’orthographe - pour les requêtes/recherches web

La fonction `Spell` est plus agressive et renvoie de meilleurs résultats de recherche. Le mode `Spell` détecte la plupart des fautes d’orthographe, mais passe à côté de certaines erreurs grammaticales repérées par le mode `Proof`, par exemple, la mise en majuscules et les répétitions de mots.

> [!NOTE]
> * La longueur maximale des requêtes est indiquée ci-dessous. Si la requête dépasse la longueur maximale, elle et ses résultats ne sont pas modifiés.
>    * 130 caractères pour les codes de langue suivants : en, de, es, fr, pl, pt, sv, ru, nl, nb, tr-tr, it, zh, ko. 
>    * 65 caractères pour tous les autres.
> * Le mode Spell ne prend pas en charge les crochets (`[` et `]`) dans les requêtes, ceux-ci pouvant entraîner des résultats incohérents. Nous vous recommandons de les supprimer de vos requêtes quand vous utilisez le mode Spell.

## <a name="market-setting"></a>Paramètre de segment

Un [code de marché](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v7-reference#market-codes) doit être spécifié avec le paramètre de requête `mkt` dans votre demande. L’API utilise sinon un marché par défaut en fonction de l’adresse IP de la demande.


## <a name="http-post-and-get-support"></a>Prise en charge HTTP POST et GET

L’API prend en charge les requêtes HTTP POST et HTTP GET. Le type de requête utilisé dépend de la longueur du texte que vous souhaitez vérifier. Si les chaînes sont toujours inférieures à 1 500 caractères, vous utiliserez des requêtes GET. Par contre, pour prendre en charge des chaînes contenant jusqu’à 10 000 caractères, vous utilisez des requêtes POST. La chaîne de texte peut inclure n’importe quel caractère UTF-8 valide.

L’exemple suivant montre une requête POST pour vérifier l’orthographe et la grammaire d’une chaîne de texte. L’exemple inclut le paramètre de requête [mode](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v7-reference#mode) par souci d’exhaustivité (il est possible qu’il ne soit pas défini car `mode` est défini par défaut sur Vérifier). Le paramètre de requête [texte](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v7-reference#text) contient la chaîne à vérifier.
  
```  
POST https://api.cognitive.microsoft.com/bing/v7.0/spellcheck?mode=proof&mkt=en-us HTTP/1.1  
Content-Type: application/x-www-form-urlencoded  
Content-Length: 47  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
 
text=when+its+your+turn+turn,+john,+come+runing  
``` 

Si vous utilisez HTTP GET, vous devez inclure le paramètre de requête `text` dans la chaîne de requête de l’URL.
  
Voici la réponse à la requête précédente. La réponse contient un objet [SpellCheck](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v7-reference#spellcheck). 
  
```json
{  
    "_type" : "SpellCheck",  
    "flaggedTokens" : [{  
        "offset" : 5,  
        "token" : "its",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "it's",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 25,  
        "token" : "john",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "John",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 19,  
        "token" : "turn",  
        "type" : "RepeatedToken",  
        "suggestions" : [{  
            "suggestion" : "",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 35,  
        "token" : "runing",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "running",  
            "score" : 1  
        }]  
    }]  
}  
```  
  
Le champ [flaggedTokens](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v7-reference#flaggedtokens) répertorie les erreurs d’orthographe et de grammaire détectées par l’API dans la chaîne [texte](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v7-reference#text). Le champ `token` contient le mot à remplacer. Vous devez utiliser le décalage de base zéro dans le champ `offset` pour trouver le jeton dans la chaîne `text`. Vous remplacez ensuite le mot à cet emplacement avec le mot indiqué dans le champ `suggestion`. 

Si le champ `type` indique RepeatedToken, vous remplacerez toujours le jeton par `suggestion`, mais vous devrez probablement aussi supprimer l’espace de fin.

## <a name="throttling-requests"></a>Demandes de limitation

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../../includes/cognitive-services-bing-throttling-requests.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Qu’est-ce que l’API Vérification orthographique Bing ?](../overview.md)
- [Informations de référence sur l’API Vérification orthographique Bing v7](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v7-reference)