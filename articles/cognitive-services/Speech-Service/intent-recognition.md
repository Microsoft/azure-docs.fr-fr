---
title: Présentation de la reconnaissance de l’intention – Service Speech
titleSuffix: Azure Cognitive Services
description: La reconnaissance de l’intention vous permet de reconnaître les objectifs de l’utilisateur que vous avez prédéfinis. Cet article est une présentation des avantages et des capacités du service de reconnaissance de l’intention.
services: cognitive-services
author: v-demjoh
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 10/13/2020
ms.author: v-demjoh
keywords: reconnaissance de l’intention
ms.openlocfilehash: 215b25e440b8cad76e0656e47d32b184edf4ac66
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95018699"
---
# <a name="what-is-intent-recognition"></a>Qu’est-ce que la reconnaissance de l’intention ?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Dans cette vue d’ensemble, vous découvrez les avantages et les capacités du service de reconnaissance de l’intention. Le SDK Speech de Cognitive Services s’intègre au Language Understanding Intelligent Service (LUIS) pour fournir la reconnaissance des intentions. Une intention est quelque chose que l’utilisateur souhaite faire : réserver un vol, vérifier la météo ou effectuer un appel.
Grâce à la reconnaissance de l’intention, vos applications, outils et appareils peuvent déterminer ce que l’utilisateur souhaite lancer ou faire en fonction des options que vous définissez dans LUIS.

## <a name="luis-key-required"></a>Clé LUIS obligatoire

* LUIS s’intègre au service Speech pour reconnaître les intentions à partir de la reconnaissance vocale. Vous n’avez besoin que de LUIS, aucun abonnement au service Speech n’est nécessaire.
* La reconnaissance de l’intention de Speech est intégrée au Kit de développement logiciel (SDK). Vous pouvez utiliser une clé LUIS avec le service Speech.
* La reconnaissance de l’intention par le biais du Kit de développement logiciel (SDK) Speech est [proposée dans un sous-ensemble de régions pris en charge par LUIS](./regions.md#intent-recognition).

## <a name="get-started"></a>Bien démarrer

Consultez le [démarrage rapide](quickstarts/intent-recognition.md) pour prendre en main la reconnaissance de l’intention.

## <a name="sample-code"></a>Exemple de code

Exemple de code pour la reconnaissance de l’intention :

* [Démarrage rapide : Utiliser une application domotique prédéfinie](../luis/luis-get-started-create-app.md)
* [Effectuer une reconnaissance des intentions vocales à l’aide du kit SDK Speech pour C#](./how-to-recognize-intents-from-speech-csharp.md)
* [Reconnaissance de l’intention et autres services Speech utilisant Unity en C#](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/unity/speechrecognizer)
* [Reconnaître des intentions à l’aide du Kit de développement logiciel (SDK) Speech pour Python](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/python/console)
* [Reconnaissance de l’intention et autres services Speech utilisant le Kit de développement logiciel (SDK) Speech pour C++ sur Windows](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/cpp/windows/console)
* [Reconnaissance de l’intention et autres services Speech utilisant le Kit de développement logiciel (SDK) Speech pour Java sur Windows ou Linux](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/java/jre/console)
* [Reconnaissance de l’intention et autres services Speech utilisant le Kit de développement logiciel (SDK) Speech pour JavaScript sur un navigateur web](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/js/browser)

## <a name="reference-docs"></a>Documents de référence

* [Kit de développement logiciel (SDK) de reconnaissance vocale](./speech-sdk.md)

## <a name="next-steps"></a>Étapes suivantes

* Suivre le [démarrage rapide](quickstarts/intent-recognition.md) sur la reconnaissance de l’intention
* [Obtenir gratuitement une clé d’abonnement au service Speech](overview.md#try-the-speech-service-for-free)
* [Obtenir le kit SDK Speech](speech-sdk.md)