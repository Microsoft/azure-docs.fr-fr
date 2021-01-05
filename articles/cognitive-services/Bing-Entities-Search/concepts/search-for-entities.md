---
title: Rechercher des entités avec l’API Recherche d’entités Bing
titleSuffix: Azure Cognitive Services
description: Utilisez l’API Recherche d’entités Bing pour extraire et rechercher des entités et des lieux à partir de requêtes de recherche.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-entity-search
ms.topic: conceptual
ms.date: 02/01/2019
ms.author: aahi
ms.openlocfilehash: 9dabceda17defb24f2a916cd641f625feb551c6a
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353287"
---
# <a name="searching-for-entities-with-the-bing-entity-api"></a>Recherche d’entités avec l’API Recherche d’entités Bing

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

## <a name="suggest-search-terms-with-the-bing-autosuggest-api"></a>Suggérer des termes de recherche avec l’API Suggestion automatique Bing

Si vous fournissez une zone de recherche dans laquelle l’utilisateur entre son terme de recherche, utilisez [l’API Suggestion automatique Bing](../../bing-autosuggest/get-suggested-search-terms.md) pour améliorer l’expérience. À mesure que l’utilisateur tape des termes de recherche, l’API suggère des chaînes de requête.

Quand l’utilisateur a terminé d’entrer son terme de recherche, encodez-le par URL avant de définir le paramètre de requête [q](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#query). Par exemple, si l’utilisateur saisit *Marcus Appel*, définissez `q` sur *Marcus+Appel* ou *Marcus%20Appel*.

Si le terme de recherche contient une faute d’orthographe, la réponse à la recherche inclut un objet [QueryContext](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#querycontext). L’objet affiche l’orthographe d’origine et l’orthographe corrigée utilisées par Bing pour la recherche.

```json
"queryContext": {
    "originalQuery": "hollo wrld",
    "alteredQuery": "hello world",
    "alterationOverrideQuery": "+hollo wrld",
    "adultIntent": false
}
```

## <a name="the-bing-entity-search-api-response"></a>Réponse de l’API Recherche d’entités Bing

La réponse de l’API contient un objet [SearchResponse](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#searchresponse). Si Bing trouve une entité ou un lieu pertinent(e), l’objet inclut le champ `entities`, le champ `places`, ou les deux. Sinon, l’objet de réponse n’inclut aucun de ces champs.
> [!NOTE]
> Les réponses d’entité prennent en charge plusieurs marchés, mais la réponse Lieux prend en charge uniquement les adresses professionnelles aux États-Unis. 

Le champ `entities` est un objet [EntityAnswer](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference) qui contient une liste d’objets [Entity](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#entity) (voir le champ `value`). La liste peut contenir une seule entité dominante, plusieurs entités de levée de l’ambiguïté ou les deux. 

Une entité dominante est retournée quand Bing pense être qu’il s’agit de la seule entité satisfaisant à la requête (il n’existe aucune ambiguïté quant à l’entité qui satisfait la requête). Si plusieurs entités sont susceptibles de satisfaire la requête, la liste contient plusieurs entités de levée d’ambiguïté. Par exemple, si la requête utilise un titre générique de franchise cinématographique, la liste contiendra des entités de levée d’ambiguïté. Toutefois, si la requête spécifie un titre spécifique de la franchise, la liste contiendra une seule entité dominante.

Les entités incluent des personnalités connues tels que des chanteurs, des acteurs, des athlètes, des mannequins, etc., des lieux et des repères comme le mont Rainier ou le Lincoln Memorial et des éléments comme une banane, un Goldendoodle, un livre ou un titre de film. Le champ [entityPresentationInfo](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#entitypresentationinfo) contient des conseils qui identifient le type d’entité. Par exemple, s’il s’agit d’une personne, d’un film, d’un animal ou d’un site. Pour obtenir la liste des types possibles, consultez [Types d’entités](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#entity-types).

```json
"entityPresentationInfo": {
    "entityScenario": "DominantEntity",
    "entityTypeHints": ["Attraction"],
    "entityTypeDisplayHint": "Mountain"
}, ...
```

Le code suivant illustre une réponse incluant une entité dominante et de levée d’ambiguïté.

```json
{
    "_type": "SearchResponse",
    "queryContext": {
        "originalQuery": "Mount Rainier"
    },
    "entities": {
        "value": [{
            "contractualRules": [{
                "_type": "ContractualRules/LicenseAttribution",
                "targetPropertyName": "description",
                "mustBeCloseToContent": true,
                "license": {
                    "name": "CC-BY-SA",
                    "url": "https://creativecommons.org/licenses/by-sa/3.0/"
                },
                "licenseNotice": "Text under CC-BY-SA license"
            },
            {
                "_type": "ContractualRules/LinkAttribution",
                "targetPropertyName": "description",
                "mustBeCloseToContent": true,
                "text": "contoso.com",
                "url": "http://contoso.com/mount_rainier"
            },
            {
                "_type": "ContractualRules/MediaAttribution",
                "targetPropertyName": "image",
                "mustBeCloseToContent": true,
                "url": "http://contoso.com/mount-rainier"
            }],
            "webSearchUrl": "https://www.bing.com/search?q=Mount%20Rainier...",
            "name": "Mount Rainier",
            "url": "http://www.northwindtraders.com/",
            "image": {
                "name": "Mount Rainier",
                "thumbnailUrl": "https://www.bing.com/th?id=A4ae343983daa4...",
                "provider": [{
                    "_type": "Organization",
                    "url": "http://contoso.com/mount_rainier"
                }],
                "hostPageUrl": "http://contoso.com/commons/7/72/mount_rain...",
                "width": 110,
                "height": 110
            },
            "description": "Mount Rainier is 14,411 ft tall and the highest mountain...",
            "entityPresentationInfo": {
                "entityScenario": "DominantEntity",
                "entityTypeHints": ["Attraction"]
            },
            "bingId": "38b9431e-cf91-93be-0584-c42a3ecbfdc7"
        },
        {
            "contractualRules": [{
                "_type": "ContractualRules/MediaAttribution",
                "targetPropertyName": "image",
                "mustBeCloseToContent": true,
                "url": "http://contoso.com/mount_rainier_national_park"
            }],
            "webSearchUrl": "https://www.bing.com/search?q=Mount%20Rainier%20National...",
            "name": "Mount Rainier National Park",
            "url": "http://worldwideimporters.com/",
            "image": {
                "name": "Mount Rainier National Park",
                "thumbnailUrl": "https://www.bing.com/th?id=A91bdc5a1b648a695a39...",
                "provider": [{
                    "_type": "Organization",
                    "url": "http://contoso.com/mount_rainier_national_park"
                }],
                "hostPageUrl": "http://contoso.com/en/7/7a...",
                "width": 50,
                "height": 50
            },
            "description": "Mount Rainier National Park is a United States National Park...",
            "entityPresentationInfo": {
                "entityScenario": "DisambiguationItem",
                "entityTypeHints": ["Organization"]
            },
            "bingId": "29d4b681-227a-3924-7bb1-8a54e8666b8c"
        }]
    }
}
```

L’entité inclut un champ `name`, `description` et `image`. Lorsque vous affichez ces champs dans votre expérience utilisateur, vous devez les attribuer. Le champ `contractualRules` contient une liste d’attributions que vous devez appliquer. La règle contractuelle identifie le champ auquel s’applique l’attribution. Pour plus d’informations sur l’application de l’attribution, consultez [Attribution](#data-attribution).

```json
"contractualRules": [{
    "_type": "ContractualRules/LicenseAttribution",
    "targetPropertyName": "description",
    "mustBeCloseToContent": true,
    "license": {
        "name": "CC-BY-SA",
        "url": "https://creativecommons.org/licenses/by-sa/3.0/"
    },
    "licenseNotice": "Text under CC-BY-SA license"
},
{
    "_type": "ContractualRules/LinkAttribution",
    "targetPropertyName": "description",
    "mustBeCloseToContent": true,
    "text": "contoso.com",
    "url": "http://contoso.com/wiki/Mount_Rainier"
},
{
    "_type": "ContractualRules/MediaAttribution",
    "targetPropertyName": "image",
    "mustBeCloseToContent": true,
    "url": "http://contoso.com/wiki/Mount_Rainier"
}], ...
```

Lorsque vous affichez les informations de l’entité (nom, description et image), vous devez également utiliser l’URL dans le champ `webSearchUrl` pour le lien vers la page de résultats de la recherche Bing qui contient l’entité.

## <a name="find-places"></a>Recherche de lieux

Le champ `places` est un objet [LocalEntityAnswer](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference) qui contient la liste des objets [Place](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#place) (consultez les [types d’entités](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#entity-types) pour plus d’informations). La liste contient une ou plusieurs entités locales qui satisfont à la requête.

Les lieux incluent des restaurants, des hôtels ou des entreprises locales. Le champ [entityPresentationInfo](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#entitypresentationinfo) contient des conseils qui identifient le type d’entité local. La liste contient une liste de conseils comme Place, LocalBusiness, Restaurant. Chaque conseil successif de ce tableau se rapproche du type d’entité. Pour obtenir la liste des types possibles, consultez [Types d’entités](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#entity-types).

```json
"entityPresentationInfo": {
    "entityScenario": "ListItem",
    "entityTypeHints": ["Place",
    "LocalBusiness",
    "Restaurant"]
}, ...
```
> [!NOTE]
> Les réponses d’entité prennent en charge plusieurs marchés, mais la réponse Lieux prend en charge uniquement les adresses professionnelles aux États-Unis. 

Les requêtes d’entités locales telles que *restaurant à proximité* ont besoin de l’emplacement de l’utilisateur pour fournir des résultats précis. Vos requêtes doivent toujours utiliser les en-têtes X-Search-Location et X-MSEdge-ClientIP pour spécifier l’emplacement de l’utilisateur. Si Bing pense que la requête a besoin de l’emplacement de l’utilisateur, il définit le champ `askUserForLocation` de [QueryContext](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#querycontext) sur **true**. 

```json
{
    "_type": "SearchResponse",
    "queryContext": {
        "originalQuery": "Sinful Bakery and Cafe",
        "askUserForLocation": true
    },
    ...
}
```

Un résultat de lieu inclut le nom, l’adresse, le numéro de téléphone et l’URL du lieu sur le site web de l’entité. Lorsque vous affichez les informations de l’entité, vous devez également utiliser l’URL dans le champ `webSearchUrl` pour le lien vers la page de résultats de la recherche Bing qui contient l’entité.

```json
"places": {
    "value": [{
        "_type": "Restaurant",
        "webSearchUrl": "https://www.bing.com/search?q=Sinful%20Bakery...",
        "name": "Liberty's Delightful Sinful Bakery & Cafe",
        "url": "http://libertysdelightfulsinfulbakeryandcafe.com/",
        "entityPresentationInfo": {
            "entityScenario": "ListItem",
            "entityTypeHints": ["Place",
            "LocalBusiness",
            "Restaurant"]
        },
        "address": {
            "addressLocality": "Seattle",
            "addressRegion": "WA",
            "postalCode": "98112",
            "addressCountry": "US",
            "neighborhood": "Madison Park"
        },
        "telephone": "(800) 555-1212"
    }]
}
```

> [!NOTE]
> Vous, ou un tiers agissant en votre nom, ne pouvez pas utiliser, conserver, stocker, mettre en cache, partager ou distribuer les données issues de l’API Entités à des fins de test, de développement, de formation, de distribution, ni rendre disponible un service ou une fonctionnalité non Microsoft.  

## <a name="data-attribution"></a>Attribution de données

Les réponses API Entité Bing contiennent des informations qui appartiennent à des tiers. Vous êtes chargé de vous assurer que votre utilisation est appropriée, par exemple en respectant toute licence Creative Commons sur laquelle repose votre expérience utilisateur.

Si une réponse ou un résultat inclut les champs `contractualRules`, `attributions` ou `provider`, vous devez attribuer les données. Si la réponse n’inclut aucun de ces champs, aucune attribution n’est requise. Si la réponse inclut le champ `contractualRules` et `attributions` et/ou `provider`, vous devez utiliser les règles contractuelles pour attribuer les données.

L’exemple suivant montre une entité qui inclut une règle contractuelle MediaAttribution et une image qui inclut un champ `provider`. La règle MediaAttribution identifie l’image en tant que cible de la règle, donc vous ignorez le champ `provider` de l’image et utilisez à la place la règle MediaAttribution permettant l’attribution.  

```json
"value": [{
    "contractualRules": [
        ...
        {
            "_type": "ContractualRules/MediaAttribution",
            "targetPropertyName": "image",
            "mustBeCloseToContent": true,
            "url": "http://contoso.com/mount_rainier"
        }
    ],
    ...
    "image": {
        "name": "Mount Rainier",
        "thumbnailUrl": "https://www.bing.com/th?id=A46378861201...",
        "provider": [{
            "_type": "Organization",
            "url": "http://contoso.com/mount_rainier"
        }],
        "hostPageUrl": "http://www.graphicdesigninstitute.com/Uploaded...",
        "width": 110,
        "height": 110
    },
    ...
}]
```

Si une règle contractuelle inclut le champ `targetPropertyName`, la règle s’applique uniquement au champ ciblé. Sinon, la règle s’applique à l’objet parent qui contient le champ `contractualRules`.

Dans l’exemple suivant, la règle `LinkAttribution` inclut le champ `targetPropertyName`, donc la règle s’applique au champ `description`. Pour les règles qui s’appliquent à des champs spécifiques, vous devez inclure une ligne immédiatement après les données cibles contenant un lien hypertexte vers le site web du fournisseur. Par exemple, pour attribuer la description, incluez une ligne immédiatement après le texte de description contenant un lien hypertexte vers les données sur le site web du fournisseur, dans ce cas créez un lien vers contoso.com.

```json
"entities": {
    "value": [{
            ...
            "description": "Marcus Appel is a former American....",
            ...
            "contractualRules": [{
                    "_type": "ContractualRules/LinkAttribution",
                    "targetPropertyName": "description",
                    "mustBeCloseToContent": true,
                    "text": "contoso.com",
                    "url": "http://contoso.com/cr?IG=B8AD73..."
                 },
            ...
  
```

### <a name="license-attribution"></a>Attribution de licence

Si la liste des règles contractuelles inclut une règle [LicenseAttribution](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#licenseattribution), vous devez afficher les avis sur la ligne qui suit immédiatement le contenu auquel la licence s’applique. La règle `LicenseAttribution` utilise le champ `targetPropertyName` pour identifier la propriété à laquelle la licence s’applique.

Vous trouverez ci-dessous un exemple qui inclut une règle `LicenseAttribution`.

![Attribution de licence](../media/cognitive-services-bing-entities-api/licenseattribution.png)

L’avis de licence que vous affichez doit inclure un lien hypertexte vers le site web contenant des informations sur la licence. En règle générale, vous utilisez le nom de la licence en tant que lien hypertexte. Par exemple, si l’avis indique **Texte sous licence CC-BY-SA** et que CC-BY-SA est le nom de la licence, vous en faites un lien hypertexte.

### <a name="link-and-text-attribution"></a>Attribution de texte et de lien

Les règles [LinkAttribution](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#linkattribution) et [TextAttribution](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#textattribution) sont généralement utilisées pour identifier le fournisseur des données. Le champ `targetPropertyName` identifie le champ auquel la règle s’applique.

Pour attribuer les fournisseurs, incluez une ligne immédiatement après le contenu auquel les attributions s’appliquent (par exemple, le champ ciblé). La ligne doit être clairement étiquetée pour indiquer que les fournisseurs constituent la source des données. Par exemple, « Données de : contoso.com ». Pour les règles `LinkAttribution`, vous devez créer un lien hypertexte vers le site web du fournisseur.

Vous trouverez ci-dessous un exemple qui inclut les règles `LinkAttribution` et `TextAttribution`.

![Attribution de texte de lien](../media/cognitive-services-bing-entities-api/linktextattribution.png)

### <a name="media-attribution"></a>Attribution média

Si l’entité contient une image et que vous l’affichez, vous devez fournir un lien cliquable vers le site web du fournisseur. Si l’entité inclut une règle [MediaAttribution](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#mediaattribution), utilisez l’URL de la règle pour créer le lien cliquable. Sinon, utilisez l’URL incluse dans le champ `provider` de l’image pour créer le lien cliquable.

Vous trouverez ci-dessous un exemple qui inclut le champ `provider` d’une image et les règles contractuelles. Étant donné que l’exemple inclut la règle contractuelle, vous ignorez le champ `provider` de l’image et appliquez la règle `MediaAttribution`.

![Attribution média](../media/cognitive-services-bing-entities-api/mediaattribution.png)

### <a name="search-or-search-like-experience"></a>Recherche ou expérience de type recherche

Comme avec l’API Recherche Web Bing, l’API Recherche d’entités Bing peut être utilisée uniquement comme résultat d’une requête d’utilisateur directe ou d’une recherche ou comme résultat d’une action dans une application ou une expérience pouvant logiquement être interprétée comme une requête de recherche d’utilisateur. À titre d’illustration, voici quelques exemples de recherches ou d’expériences de type recherche acceptables.

- L’utilisateur saisit une requête directement dans la zone de recherche d’une application
- L’utilisateur sélectionne un texte ou une image spécifique et demande « plus d’informations » ou « des informations supplémentaires ».
- L’utilisateur interroge un robot de recherche sur un sujet particulier.
- L’utilisateur s’arrête sur un objet ou une entité donné(e) dans un scénario de type recherche visuelle.

Si vous n’êtes pas certain que votre expérience peut être considérée comme une expérience de type recherche, nous vous recommandons de vérifier auprès de Microsoft.

## <a name="throttling-requests"></a>Demandes de limitation

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../../includes/cognitive-services-bing-throttling-requests.md)]

## <a name="next-steps"></a>Étapes suivantes

* Suivez un [guide de démarrage rapide](../quickstarts/csharp.md) pour rechercher des entités avec l’API Recherche d’entités Bing.