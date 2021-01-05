---
title: Prise en charge linguistique - API Recherche d’actualités Bing
titleSuffix: Azure Cognitive Services
description: Liste des langages naturels, des pays et des régions pris en charge par l’API Recherche Web Bing.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-news-search
ms.topic: conceptual
ms.date: 1/10/2019
ms.author: aahi
ms.openlocfilehash: 9c7fd03c2239cea05dc79ad4dd1965fe253a2ce9
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96341595"
---
# <a name="language-and-region-support-for-the-bing-news-search-api"></a>Langages et régions pris en charge par l’API Recherche d’actualités Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Recherche d’actualités Bing prend en charge de nombreux pays/régions, dont beaucoup possèdent plusieurs langues. Spécifier un pays/une région avec une requête sert principalement à affiner les résultats de la recherche en fonction des centres d’intérêt dans ce pays/cette région. En outre, les résultats peuvent contenir des liens vers Bing, servant à localiser l’expérience utilisateur de Bing en fonction du pays/de la région ou de la langue spécifiée.

Vous pouvez spécifier un pays/une région à l’aide du paramètre de requête `cc`. Si vous spécifiez un pays/une région, vous devez également spécifier un ou plusieurs codes de langue à l’aide de l’en-tête HTTP `Accept-Language`. Les langues prises en charge varient selon le pays/la région ; elles sont indiquées pour chaque pays/région dans le tableau Marchés.

Vous pouvez également indiquer le marché à l’aide du paramètre de requête `mkt` et d’un code issu du tableau **Marchés**. Le fait d’indiquer un marché spécifie simultanément un pays/une région et une langue par défaut. Dans ce cas, le paramètre de requête `setLang` peut être défini sur un code de langue. Généralement, il s’agit de la même langue que celle spécifiée par `mkt`, sauf si l’utilisateur préfère voir Bing dans une autre langue.

## <a name="supported-markets-for-news-search-endpoint"></a>Marchés pris en charge pour le point de terminaison de recherche d’actualités

Pour le point de terminaison `/news/search`, le tableau suivant répertorie les valeurs de code de marché que vous pouvez utiliser pour spécifier le paramètre de requête `mkt`. Bing retourne uniquement le contenu pour ces marchés. La liste est susceptible d’être modifiée.  

Pour obtenir la liste des codes pays/régions que vous pouvez spécifier dans le paramètre de requête `cc`, consultez [Codes pays](#countrycodes).  

|Pays/région|Langage|Code du marché|  
|---------------------|--------------|-----------------|
|Danemark|Danois|da-DK|
|Autriche|Allemand|de-AT|
|Suisse|Allemand|de-CH|
|Allemagne|Allemand|de-DE|
|Australie|Anglais|en-AU|
|Canada|Anglais|en-CA|
|Royaume-Uni|Anglais|en-GB|
|Indonésie|Anglais|en-ID|
|Irlande|Anglais|en-IE|
|Inde|Anglais|en-IN|
|Malaisie|Anglais|en-MY|
|Nouvelle-Zélande|Anglais|en-NZ|
|République des Philippines|Anglais|en-PH|
|Singapour|Anglais|en-SG|
|États-Unis|Anglais|fr-FR|
|Anglais|général|en-WW|
|Anglais|général|en-XA|
|Afrique du Sud|Anglais|en-ZA|
|Argentine|Espagnol|es-AR|
|Chili|Espagnol|es-CL|
|Espagne|Espagnol|es-ES|
|Mexique|Espagnol|es-MX|
|États-Unis|Espagnol|es-US|
|Espagnol|général|es-XL|
|Finlande|Finnois|fi-FI|  
|France|Français|fr-BE|
|Canada|Français|fr-CA|
|Belgique|Néerlandais|nl-BE|
|Suisse|Français|fr-CH|
|France|Français|fr-FR|  
|Italie|Italien|it-IT|
|Hong Kong (R.A.S.)|Chinois traditionnel|zh-HK|  
|Taïwan|Chinois traditionnel|zh-TW|
|Japon|Japonais|ja-JP|  
|Corée du Sud|Coréen|ko-KR|  
|Pays-Bas|Néerlandais|nl-NL|  
|République populaire de Chine|Chinois|zh-CN|  
|Brésil|Portugais|pt-br|
|Russie|Russe|ru-RU|  
|Suède|Suédois|sv-SE|  
|Turquie|Turc|tr-TR|  

## <a name="supported-markets-for-news-endpoint"></a>Marchés pris en charge pour le point de terminaison des actualités
Pour le point de terminaison `/news`, le tableau suivant répertorie les valeurs de code de marché que vous pouvez utiliser pour spécifier le paramètre de requête `mkt`. Bing retourne uniquement le contenu pour ces marchés. La liste est susceptible d’être modifiée.  

Pour obtenir la liste des codes pays/régions que vous pouvez spécifier dans le paramètre de requête `cc`, consultez [Codes pays](#countrycodes).  

|Pays/région|Langage|Code du marché|  
|---------------------|--------------|-----------------|
|Danemark|Danois|da-DK|
|Allemagne|Allemand|de-DE|
|Australie|Anglais|en-AU|
|Royaume-Uni|Anglais|en-GB|
|États-Unis|Anglais|fr-FR|
|Anglais|général|en-WW|
|Chili|Espagnol|es-CL|
|Mexique|Espagnol|es-MX|
|États-Unis|Espagnol|es-US|
|Finlande|Finnois|fi-FI|  
|Canada|Français|fr-CA|
|France|Français|fr-FR|  
|Italie|Italien|it-IT|
|Brésil|Portugais|pt-br|
|République populaire de Chine|Chinois|zh-CN|

## <a name="supported-markets-for-news-trending-endpoint"></a>Marchés pris en charge pour le point de terminaison des tendances d’actualités
Pour le point de terminaison `/news/trendingtopics`, le tableau suivant répertorie les valeurs de code de marché que vous pouvez utiliser pour spécifier le paramètre de requête `mkt`. Bing retourne uniquement le contenu pour ces marchés. La liste est susceptible d’être modifiée.  

Pour obtenir la liste des codes pays/régions que vous pouvez spécifier dans le paramètre de requête `cc`, consultez [Codes pays](#countrycodes).  

|Pays/région|Langage|Code du marché|  
|---------------------|--------------|-----------------|
|Allemagne|Allemand|de-DE|
|Australie|Anglais|en-AU|
|Royaume-Uni|Anglais|en-GB|
|États-Unis|Anglais|fr-FR|
|Canada|Anglais|en-CA|
|Inde|Anglais|en-IN|
|France|Français|fr-FR|
|Canada|Français|fr-CA|
|Brésil|Portugais|pt-br|
|République populaire de Chine|Chinois|zh-CN|


<a name="countrycodes"></a>   
### <a name="country-codes"></a>Codes de pays  

Vous trouverez ci-dessous les codes pays/régions que vous pouvez spécifier dans le paramètre de requête `cc`. La liste est susceptible d’être modifiée.  

|Pays/région|Code pays|  
|---------------------|------------------|  
|Argentine|AR|  
|Australie|AU|  
|Autriche|AT|  
|Belgique|BE|  
|Brésil|BR|  
|Canada|CA|  
|Chili|CL|  
|Danemark|DK|  
|Finlande|FI|  
|France|FR|  
|Allemagne|DE|  
|Hong Kong (R.A.S.)|HK|  
|Inde|IN|  
|Indonésie|id|  
|Italie|IT|  
|Japon|JP|  
|Corée du Sud|KR|  
|Malaisie|MY|  
|Mexique|MX|  
|Pays-Bas|NL|  
|Nouvelle-Zélande|NZ|  
|Norvège|Non|  
|République populaire de Chine|CN|  
|Pologne|PL|  
|Portugal|PT|  
|République des Philippines|PH|  
|Russie|RU|  
|Arabie Saoudite|SA|  
|Afrique du Sud|ZA|  
|Espagne|ES|  
|Suède|SE|  
|Suisse|CH|  
|Taïwan|TW|  
|Turquie|TR|  
|Royaume-Uni|Go|  
|États-Unis|US|

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les points de terminaison de recherche d’actualités Bing, consultez [Référence de l’API de recherche d’actualité v7](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference).