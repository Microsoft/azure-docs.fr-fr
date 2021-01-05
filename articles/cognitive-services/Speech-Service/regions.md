---
title: Régions – Service Speech
titleSuffix: Azure Cognitive Services
description: Liste des régions et points de terminaison disponibles pour le service Speech, incluant la reconnaissance vocale, la synthèse vocale et la traduction vocale.
services: cognitive-services
author: mahilleb-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: panosper
ms.custom: seodec18
ms.openlocfilehash: 7b872340c50c800ebe4d4d6c7ca56b3b327bf162
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95972643"
---
# <a name="speech-service-supported-regions"></a>Régions prises en charge pour le service Speech

Le service Speech permet à votre application de convertir de l’audio en texte, d’effectuer une traduction vocale et de convertir du texte par synthèse vocale. Le service est disponible dans plusieurs régions avec des points de terminaison uniques pour le SDK et les API REST Speech.

Le portail Speech permettant d’effectuer des configurations personnalisées de votre expérience vocale pour toutes les régions est disponible ici : https://speech.microsoft.com

Gardez à l’esprit les points suivants lorsque vous envisagez les régions :

* Si votre application utilise un [SDK Speech](speech-sdk.md), vous fournissez l’identificateur de région, tel que `westus`, quand vous créez une configuration de reconnaissance vocale.
* Si votre application utilise l’une des [API REST](./overview.md#reference-docs) du service Speech, la région fait partie de l’URI de point de terminaison que vous utilisez pour effectuer des requêtes.
* Les clés créées pour une région sont valides uniquement dans cette région. Si vous essayez de les utiliser avec d’autres régions, des erreurs d’authentification se produisent.

## <a name="speech-sdk"></a>Kit de développement logiciel (SDK) de reconnaissance vocale

Dans le [SDK Speech](speech-sdk.md), les régions sont spécifiées sous forme de chaînes (par exemple, en tant que paramètres de `SpeechConfig.FromSubscription`dans le SDK Speech pour C#).

### <a name="speech-to-text-text-to-speech-and-translation"></a>Reconnaissance vocale, synthèse vocale et traduction vocale

Le portail de personnalisation vocale est disponible ici : https://speech.microsoft.com

Le service Speech est disponible dans les régions suivantes pour la **reconnaissance vocale**, la **synthèse vocale** et la **traduction** :

[!INCLUDE [](../../../includes/cognitive-services-speech-service-region-identifier.md)]

Si vous utilisez le [SDK Speech](speech-sdk.md), les régions sont spécifiées par l’**identificateur de région** (par exemple comme paramètre de `SpeechConfig.FromSubscription`). Assurez-vous que la région correspond à la région de votre abonnement.

### <a name="intent-recognition"></a>Reconnaissance de l’intention

Les régions disponibles pour la **reconnaissance de l'intention** via le kit de développement logiciel (SDK) Speech sont les suivantes :

| Région globale | Région           | Identificateur de la région |
| ------------- | ---------------- | -------------------- |
| Asia          | Asie Est        | `eastasia`           |
| Asia          | Asie Sud-Est   | `southeastasia`      |
| Australie     | Australie Est   | `australiaeast`      |
| Europe        | Europe Nord     | `northeurope`        |
| Europe        | Europe Ouest      | `westeurope`         |
| Amérique du Nord | USA Est          | `eastus`             |
| Amérique du Nord | USA Est 2        | `eastus2`            |
| Amérique du Nord | États-Unis - partie centrale méridionale | `southcentralus`     |
| Amérique du Nord | Centre-USA Ouest  | `westcentralus`      |
| Amérique du Nord | USA Ouest          | `westus`             |
| Amérique du Nord | USA Ouest 2        | `westus2`            |
| Amérique du Sud | Brésil Sud     | `brazilsouth`        |

Il s'agit d'un sous-ensemble des régions de publication prises en charge par le [service Language Understanding (LUIS)](../luis/luis-reference-regions.md).

### <a name="voice-assistants"></a>Assistants vocaux

Le [Kit de développement logiciel (SDK) Speech](speech-sdk.md) prend en charge les fonctionnalités d’**assistant vocal** dans les régions suivantes :

| Région         | Identificateur de la région |
| -------------- | -------------------- |
| USA Ouest        | `westus`             |
| USA Ouest 2      | `westus2`            |
| USA Est        | `eastus`             |
| USA Est 2      | `eastus2`            |
| Europe Ouest    | `westeurope`         |
| Europe Nord   | `northeurope`        |
| Asie Sud-Est | `southeastasia`      |

### <a name="speaker-recognition"></a>Reconnaissance de l’orateur

Reconnaissance de l’orateur est actuellement disponible uniquement dans la région `westus`.

## <a name="rest-apis"></a>API REST

Le service Speech expose également des points de terminaison REST pour les requêtes de reconnaissance vocale et de synthèse vocale.

### <a name="speech-to-text"></a>Reconnaissance vocale

Pour obtenir la documentation de référence relative à la reconnaissance vocale, consultez [API REST de reconnaissance vocale](rest-speech-to-text.md).

Le point de terminaison de l’API REST a le format suivant :

```
https://<REGION_IDENTIFIER>.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1
```

Remplacez `<REGION_IDENTIFIER>` par l'identificateur correspondant à la région de votre abonnement dans le tableau suivant :

[!INCLUDE [](../../../includes/cognitive-services-speech-service-region-identifier.md)]

> [!NOTE]
> Le paramètre de langue doit être ajouté à l'URL pour éviter de recevoir une erreur HTTP 4xx. Par exemple, la langue définie sur la valeur Anglais (États-Unis) à l’aide du point de terminaison USA Ouest est : `https://westus.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US`.

### <a name="text-to-speech"></a>Synthèse vocale

Pour obtenir la documentation de référence relative à la synthèse vocale, consultez [API REST de synthèse vocale](rest-text-to-speech.md).

[!INCLUDE [](../../../includes/cognitive-services-speech-service-endpoints-text-to-speech.md)]