---
title: Prise en charge linguistique - API Recherche personnalisée Bing
titleSuffix: Azure Cognitive Services
description: Liste des langues et régions prises en charge par l’API Recherche personnalisée Bing.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-custom-search
ms.topic: conceptual
ms.date: 09/25/2018
ms.author: aahi
ms.openlocfilehash: 73b25ce1093ce697aa0937161d2af49526c231b7
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352352"
---
# <a name="language-and-region-support-for-the-bing-custom-search-api"></a>Langues et régions prises en charge par l’API Recherche personnalisée Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Contrat Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Recherche personnalisée Bing prend en charge plus d’une trentaine de pays/régions, dont beaucoup possèdent plusieurs langues.

La requête peut indiquer le paramètre [mkt](/rest/api/cognitiveservices-bingsearch/bing-custom-search-api-v7-reference#mkt) (facultatif), qui vous permet de choisir le marché dont proviennent les résultats. Pour obtenir la liste complète des paramètres de requête facultatifs, consultez [Query Parameters](/rest/api/cognitiveservices-bingsearch/bing-custom-search-api-v7-reference#query-parameters) (Paramètres de requête).

Vous pouvez spécifier un pays/une région à l’aide du paramètre de requête `cc`. Si vous spécifiez un pays/une région, vous devez également spécifier un ou plusieurs codes de langue à l’aide de l’en-tête `Accept-Language`. Les langues prises en charge varient selon le pays/la région ; elles sont indiquées pour chaque pays/région dans le tableau **Marchés**.

Étant donné que l’en-tête `Accept-Language` et le paramètre de requête `setLang` s’excluent mutuellement, ne spécifiez pas les deux. Pour plus d’informations, consultez [Accept-Language](/rest/api/cognitiveservices-bingsearch/bing-custom-search-api-v7-reference#acceptlanguage).

## <a name="countriesregions"></a>Pays/régions

|Pays/région|Code|
|-------|----|
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
|Chine|CN|
|Pologne|PL|
|Portugal|PT|
|Philippines|PH|
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


## <a name="markets"></a>Marchés

|Pays/région|Langage|Code du marché|
|-------|--------|-----------|
|Argentine|Espagnol|es-AR|
|Australie|Anglais|en-AU|
|Autriche|Allemand|de-AT|
|Belgique|Néerlandais|nl-BE|
|Belgique|Français|fr-BE|
|Brésil|Portugais|pt-br|
|Canada|Anglais|en-CA|
|Canada|Français|fr-CA|
|Chili|Espagnol|es-CL|
|Danemark|Danois|da-DK|
|Finlande|Finnois|fi-FI|
|France|Français|fr-FR|
|Allemagne|Allemand|de-DE|
|Hong Kong (R.A.S.)|Chinois traditionnel|zh-HK|
|Inde|Anglais|en-IN|
|Indonésie|Anglais|en-ID|
|Italie|Italien|it-IT|
|Japon|Japonais|ja-JP|
|Corée du Sud|Coréen|ko-KR|
|Malaisie|Anglais|en-MY|
|Mexique|Espagnol|es-MX|
|Pays-Bas|Néerlandais|nl-NL|
|Nouvelle-Zélande|Anglais|en-NZ|
|Norvège|Norvégien|no-NO|
|Chine|Chinois|zh-CN|
|Pologne|Polonais|pl-PL|
|Portugal|Portugais|pt-PT|
|Philippines|Anglais|en-PH|
|Russie|Russe|ru-RU|
|Arabie Saoudite|Arabe|ar-SA|
|Afrique du Sud|Anglais|en-ZA|
|Espagne|Espagnol|es-ES|
|Suède|Suédois|sv-SE|
|Suisse|Français|fr-CH|
|Suisse|Allemand|de-CH|
|Taïwan|Chinois traditionnel|zh-TW|
|Turquie|Turc|tr-TR|
|Royaume-Uni|Anglais|en-GB|
|États-Unis|Anglais|fr-FR|
|États-Unis|Espagnol|es-US|