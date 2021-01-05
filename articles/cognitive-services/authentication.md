---
title: Authentification
titleSuffix: Azure Cognitive Services
description: 'Pour authentifier une requête auprès d’une ressource Azure Cognitive Services, trois options s’offrent à vous : une clé d’abonnement, un jeton du porteur ou un abonnement multiservice. Cet article décrit chaque méthode et explique comment adresser une requête.'
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 11/22/2019
ms.author: erhopf
ms.openlocfilehash: c7aeb9e9f4de7b4de62f9b5a8da6d997e32a2399
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94363321"
---
# <a name="authenticate-requests-to-azure-cognitive-services"></a>Authentifier des requêtes auprès d’Azure Cognitive Services

Chaque requête adressée à un service Azure Cognitive Services doit inclure un en-tête d’authentification. Cet en-tête passe une clé d’abonnement ou un jeton d’accès qui sert à valider votre abonnement à un service ou à un groupe de services. Cet article présente trois façons d’authentifier une requête et les conditions de chaque méthode.

* Authentifier avec une clé d’abonnement [monoservice](#authenticate-with-a-single-service-subscription-key) ou [multiservice](#authenticate-with-a-multi-service-subscription-key)
* Authentifier avec un [jeton](#authenticate-with-an-authentication-token)
* Authentifier avec [Azure Active Directory (AAD)](#authenticate-with-azure-active-directory)

## <a name="prerequisites"></a>Prérequis

Pour adresser une requête, vous devez disposer d’un compte Azure et d’un abonnement Azure Cognitive Services. Si vous avez déjà un compte, passez à la section suivante. Si vous n’avez pas de compte, ce guide va vous aider à en créer un en quelques minutes : [Créer un compte Cognitive Services pour Azure](cognitive-services-apis-create-account.md).

Vous pouvez obtenir votre clé d’abonnement sur le [portail Azure](cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) après la [création de votre compte](https://azure.microsoft.com/free/cognitive-services/).

## <a name="authentication-headers"></a>En-têtes d’authentification

Passons rapidement en revue les en-têtes d’authentification disponibles en vue d’une utilisation avec Azure Cognitive Services.

| En-tête | Description |
|--------|-------------|
| Ocp-Apim-Subscription-Key | Utilisez cet en-tête pour authentifier une requête avec une clé d’abonnement à un service spécifique ou une clé d’abonnement multiservice. |
| Ocp-Apim-Subscription-Region | Cet en-tête n’est nécessaire que si vous utilisez une clé d’abonnement multiservice avec le [service Translator](./Translator/reference/v3-0-reference.md). Utilisez cet en-tête pour spécifier la région de l’abonnement. |
| Autorisation | Utilisez cet en-tête si vous utilisez un jeton d’authentification. Les étapes à suivre pour effectuer un échange de jeton sont détaillées dans les sections suivantes. La valeur fournie est au format suivant : `Bearer <TOKEN>`. |

## <a name="authenticate-with-a-single-service-subscription-key"></a>Authentification avec une clé d’abonnement monoservice

La première option consiste à authentifier une requête avec une clé d’abonnement pour un service spécifique, tel que Translator. Les clés sont disponibles dans le portail Azure pour chaque ressource que vous avez créée. Pour utiliser une clé d’abonnement pour authentifier une requête, vous devez la passer sous la forme de l’en-tête `Ocp-Apim-Subscription-Key`.

Ces exemples de requêtes montrent comment utiliser l’en-tête `Ocp-Apim-Subscription-Key`. Si vous utilisez ces exemples, n’oubliez pas d’inclure une clé d’abonnement valide.

Voici un exemple d’appel à l’API Recherche Web Bing :
```cURL
curl -X GET 'https://api.cognitive.microsoft.com/bing/v7.0/search?q=Welsch%20Pembroke%20Corgis' \
-H 'Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY' | json_pp
```

Voici un exemple d’appel au service Translator :
```cURL
curl -X POST 'https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de' \
-H 'Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY' \
-H 'Content-Type: application/json' \
--data-raw '[{ "text": "How much for the cup of coffee?" }]' | json_pp
```

La vidéo suivante montre l’utilisation d’une clé Cognitive Services.

## <a name="authenticate-with-a-multi-service-subscription-key"></a>Authentification avec une clé d’abonnement multiservice

>[!WARNING]
> À ce stade, les services suivants **ne prennent pas en charge** les clés multiservices : QnA Maker, services Speech, Custom Vision et Détecteur d’anomalies.

Cette option utilise également une clé d’abonnement pour authentifier les requêtes. La principale différence tient au fait qu’une clé d’abonnement n’est pas liée à un service spécifique. Vous pouvez ainsi utiliser une seule clé pour authentifier des requêtes auprès de plusieurs services Cognitive Services. Pour plus d’informations sur la disponibilité régionale, les fonctionnalités prises en charge et les tarifs, consultez les [tarifs de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/).

La clé d’abonnement est fournie dans chaque requête sous la forme de l’en-tête `Ocp-Apim-Subscription-Key`.

[![Démonstration de clé d’abonnement multiservice à Azure Cognitive Services](./media/index/single-key-demonstration-video.png)](https://www.youtube.com/watch?v=psHtA1p7Cas&feature=youtu.be)

### <a name="supported-regions"></a>Régions prises en charge

Quand vous utilisez la clé d’abonnement multiservice pour adresser une requête à `api.cognitive.microsoft.com`, vous devez inclure la région dans l’URL. Par exemple : `westus.api.cognitive.microsoft.com`.

Quand vous utilisez la clé d’abonnement multiservice avec le service Translator, vous devez spécifier la région de l’abonnement avec l’en-tête `Ocp-Apim-Subscription-Region`.

L’authentification multiservice est prise en charge dans ces régions :

- `australiaeast`
- `brazilsouth`
- `canadacentral`
- `centralindia`
- `eastasia`
- `eastus`
- `japaneast`
- `northeurope`
- `southcentralus`
- `southeastasia`
- `uksouth`
- `westcentralus`
- `westeurope`
- `westus`
- `westus2`

### <a name="sample-requests"></a>Exemples de demandes

Voici un exemple d’appel à l’API Recherche Web Bing :

```cURL
curl -X GET 'https://YOUR-REGION.api.cognitive.microsoft.com/bing/v7.0/search?q=Welsch%20Pembroke%20Corgis' \
-H 'Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY' | json_pp
```

Voici un exemple d’appel au service Translator :

```cURL
curl -X POST 'https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de' \
-H 'Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY' \
-H 'Ocp-Apim-Subscription-Region: YOUR_SUBSCRIPTION_REGION' \
-H 'Content-Type: application/json' \
--data-raw '[{ "text": "How much for the cup of coffee?" }]' | json_pp
```

## <a name="authenticate-with-an-authentication-token"></a>Authentification avec un jeton d’authentification

Certains services Azure Cognitive Services acceptent et, dans certains cas, nécessitent un jeton d’authentification. Les services suivants prennent actuellement en charge les jetons d’authentification :

* API de traduction de texte
* Services Speech : API REST de reconnaissance vocale
* Services Speech : API REST de synthèse vocale

>[!NOTE]
> QnA Maker utilise également l’en-tête Authorization, mais nécessite une clé de point de terminaison. Pour plus d’informations, consultez [QnA Maker : Obtenir des réponses à partir de la base de connaissances](./qnamaker/quickstarts/get-answer-from-knowledge-base-using-url-tool.md).

>[!WARNING]
> Les services qui prennent en charge les jetons d’authentification peuvent changer au fil du temps. Pensez donc à consulter la référence sur l’API d’un service avant d’utiliser cette méthode d’authentification.

Vous pouvez échanger les clés d’abonnement monoservice et multiservice contre des jetons d’authentification. Les jetons d’authentification sont valides pour une durée de 10 minutes.

Les jetons d’authentification sont incluses dans une requête sous la forme de l’en-tête `Authorization`. La valeur du jeton fournie doit être précédée de `Bearer`. Par exemple : `Bearer YOUR_AUTH_TOKEN`.

### <a name="sample-requests"></a>Exemples de demandes

Utilisez cette URL pour échanger une clé d'abonnement contre un jeton d'authentification : `https://YOUR-REGION.api.cognitive.microsoft.com/sts/v1.0/issueToken`.

```cURL
curl -v -X POST \
"https://YOUR-REGION.api.cognitive.microsoft.com/sts/v1.0/issueToken" \
-H "Content-type: application/x-www-form-urlencoded" \
-H "Content-length: 0" \
-H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY"
```

Ces régions multiservices prennent en charge l’échange de jeton :

- `australiaeast`
- `brazilsouth`
- `canadacentral`
- `centralindia`
- `eastasia`
- `eastus`
- `japaneast`
- `northeurope`
- `southcentralus`
- `southeastasia`
- `uksouth`
- `westcentralus`
- `westeurope`
- `westus`
- `westus2`

Après avoir obtenu un jeton d’authentification, vous devez le passer dans chaque requête sous la forme de l’en-tête `Authorization`. Voici un exemple d’appel au service Translator :

```cURL
curl -X POST 'https://api.cognitive.microsofttranslator.com/translate?api-version=3.0&from=en&to=de' \
-H 'Authorization: Bearer YOUR_AUTH_TOKEN' \
-H 'Content-Type: application/json' \
--data-raw '[{ "text": "How much for the cup of coffee?" }]' | json_pp
```

[!INCLUDE [](../../includes/cognitive-services-azure-active-directory-authentication.md)]

## <a name="see-also"></a>Voir aussi

* [Qu’est-ce que Cognitive Services ?](./what-are-cognitive-services.md)
* [Tarifs de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/)
* [Sous-domaines personnalisés](cognitive-services-custom-subdomains.md)