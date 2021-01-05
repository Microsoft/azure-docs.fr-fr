---
title: Démarrage rapide sur la transcription de conversation en temps réel - Service Speech
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser la transcription de conversation en temps réel avec le SDK Speech. La fonctionnalité de transcription de conversation vous permet de transcrire des réunions et d’autres conversations avec la possibilité d’ajouter, de supprimer et d’identifier des participants en diffusant en streaming du contenu audio vers le service Speech.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 10/20/2020
ms.author: trbye
zone_pivot_groups: acs-js-csharp
ms.openlocfilehash: ae3d7b329c55ce0be101cee73e1fc7674a2d75aa
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026537"
---
# <a name="get-started-with-real-time-conversation-transcription"></a>Bien démarrer avec la transcription de conversation en temps réel

L’API **ConversationTranscriber** du SDK Speech vous permet de transcrire des réunions et d’autres conversations avec la possibilité d’ajouter, supprimer et identifier des participants en diffusant en streaming du contenu audio vers le service Speech à l’aide de `PullStream` ou `PushStream`. Vous créez d’abord des signatures vocales pour chaque participant à l’aide de l’API REST, puis vous utilisez les signatures vocales avec le SDK pour transcrire les conversations. Pour plus d’informations, consultez la [présentation de la fonctionnalité de transcription de conversation](conversation-transcription.md).

## <a name="limitations"></a>Limites

* Disponible uniquement dans les régions d’abonnement suivantes : `centralus`, `eastasia`, `eastus`, `westeurope`
* Nécessite un réseau circulaire de sept microphones avec un flux de référence de lecture. Le réseau de microphones doit respecter [nos spécifications](./speech-devices-sdk-microphone.md).
* Le [SDK Speech Devices](speech-devices-sdk.md) fournit des appareils appropriés et un exemple d’application illustrant la transcription de conversation.

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous disposez d’un compte Azure et d’un abonnement au service Speech. Si vous n’avez pas de compte et d’abonnement, [essayez le service Speech gratuitement](overview.md#try-the-speech-service-for-free).

::: zone pivot="programming-language-javascript"
[!INCLUDE [JavaScript Basics include](includes/how-to/conversation-transcription/real-time-javascript.md)]
::: zone-end

::: zone pivot="programming-language-csharp"
[!INCLUDE [C# Basics include](includes/how-to/conversation-transcription/real-time-csharp.md)]
::: zone-end

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Transcription de conversation asynchrone](how-to-async-conversation-transcription.md)
> [Exemple de code d’appareil ROOBO](https://github.com/Azure-Samples/Cognitive-Services-Speech-Devices-SDK/blob/master/Samples/Java/Android/Speech%20Devices%20SDK%20Starter%20App/example/app/src/main/java/com/microsoft/cognitiveservices/speech/samples/sdsdkstarterapp/ConversationTranscription.java)
> [Exemple de code du kit de développement Azure Kinect](https://github.com/Azure-Samples/Cognitive-Services-Speech-Devices-SDK/blob/master/Samples/Java/Windows_Linux/SampleDemo/src/com/microsoft/cognitiveservices/speech/samples/Cts.java)