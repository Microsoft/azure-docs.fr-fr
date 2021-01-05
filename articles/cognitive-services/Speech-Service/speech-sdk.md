---
title: À propos du SDK Speech – Service Speech
titleSuffix: Azure Cognitive Services
description: Le kit de développement logiciel (SDK) Speech expose de nombreuses fonctionnalités du service Speech, ce qui facilite le développement d'applications à reconnaissance vocale.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/03/2020
ms.author: trbye
ms.openlocfilehash: 9356cbb6eb210d6745d90e8ad001917e73592cd3
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "96021982"
---
# <a name="about-the-speech-sdk"></a>À propos du kit SDK Speech

Le kit de développement logiciel (SDK) Speech expose de nombreuses fonctionnalités du service Speech afin de vous permettre de développer des applications à reconnaissance vocale. Le kit de développement logiciel (SDK) Speech est disponible dans de nombreux langages de programmation et sur toutes les plateformes.

[!INCLUDE [Speech SDK Platforms](../../../includes/cognitive-services-speech-service-speech-sdk-platforms.md)]

## <a name="scenario-capabilities"></a>Fonctionnalités associées aux scénarios

Le SDK Speech expose de nombreuses fonctionnalités du service Speech, mais pas toutes. Les fonctionnalités du Speech SDK sont souvent associées à des scénarios. Le SDK Speech est idéal pour les scénarios, en temps réel ou non, qui utilisent des appareils locaux, des fichiers, un espace de stockage d'objets blob Azure et même des flux d'entrée et de sortie. Lorsqu'un scénario n'est pas réalisable avec le SDK Speech, recherchez une alternative avec l'API REST.

### <a name="speech-to-text"></a>Reconnaissance vocale

La [reconnaissance vocale](speech-to-text.md) transcrit des flux audio en texte que vos applications, outils ou appareils peuvent utiliser ou afficher. Utilisez la reconnaissance vocale avec [LUIS (Language Understanding Intelligent Service)](../luis/index.yml) pour déduire les intentions de l’utilisateur à partir des transcriptions et agir sur des commandes vocales. Utilisez la [Traduction vocale](speech-translation.md) pour traduire une entrée vocale dans une autre langue en un seul appel. Pour plus d'informations, consultez [Les bases de la reconnaissance vocale](./get-started-speech-to-text.md).

**Reconnaissance vocale (SR), Liste d’expressions, Intention, Traduction et Conteneurs locaux** sont disponibles sur les plateformes suivantes :

  - C++/Windows & Linux & macOS
  - C#(Framework & .NET Core)/Windows & UWP & Unity & Xamarin & Linux & macOS
  - Java (JRE et Android)
  - JavaScript (navigateur et NodeJS)
  - Python
  - Swift
  - Objective-C  
  - Go (SR uniquement)

### <a name="text-to-speech"></a>Synthèse vocale

La [Synthèse vocale ](text-to-speech.md) convertit le texte en discours synthétisé proche de celui d'un être humain. Le texte d'entrée correspond à des littéraux de chaîne ou utilise [SSML (Speech Synthesis Markup Language)](speech-synthesis-markup.md). Pour plus d'informations sur les voix standard ou neurales, consultez [Langage de synthèse vocale et prise en charge vocale](language-support.md#text-to-speech).

La **Synthèse vocale (TTS)** est disponible sur les plateformes suivantes :

  - C++/Windows & Linux
  - C#/Windows & UWP & Unity
  - Java (JRE et Android)
  - Python
  - Swift
  - Objective-C
  - L’API REST TTS peut être utilisée dans tous les autres cas.

### <a name="voice-assistants"></a>Assistants vocaux

Les [assistants vocaux](voice-assistants.md) qui utilisent le SDK Speech permettent aux développeurs de créer des interfaces conversationnelles naturelles pour leurs applications et leurs expériences. Le service d'assistance vocale permet une interaction rapide et fiable entre un appareil et un assistant. L'implémentation utilise le canal Direct Line Speech de Bot Framework ou le service intégré Commandes personnalisées (préversion) pour la réalisation des tâches. En outre, des assistants vocaux peuvent utiliser des voix personnalisées dans le [Portail Custom Voice](https://aka.ms/customvoice) pour ajouter une expérience de sortie vocale unique.

Les **Assistants vocaux** sont disponibles sur les plateformes suivantes :

  - C++/Windows & Linux & macOS
  - C#/Windows
  - Java/Windows & Linux & macOS & Android (SDK Speech Devices)

#### <a name="keyword-spotting"></a>Identification de mot clé

Le SDK Speech prend en charge le concept d'[identification de mot clé](./custom-keyword-basics.md). Il s'agit d'identifier un mot clé dans le discours, puis de déclencher l'action correspondante. Par exemple, « Bonjour Cortana » active de l'Assistant Cortana.

L’**Identification de mot clé (KWS)** est disponible sur les plateformes suivantes :

  - C++/Windows & Linux
  - C#/Windows & Linux
  - Python/Windows & Linux
  - Java/Windows & Linux & Android (SDK Speech Devices)
  - La fonctionnalité d’identification de mot clé (KWS) peut fonctionner avec n’importe quel type de microphone. Toutefois, sa prise en charge officielle est actuellement limitée aux réseaux de microphones présents dans le matériel Azure Kinect DK ou dans le Kit de développement logiciel (SDK) Speech Devices

### <a name="meeting-scenarios"></a>Scénarios de réunion

Le SDK Speech est parfait pour transcrire des scénarios de réunion, que ce soit à partir d'un seul appareil ou d'une conversation multi-appareils.

#### <a name="conversation-transcription"></a>Transcription de conversation

La [Transcription de conversation](conversation-transcription.md) permet la reconnaissance vocale en temps réel (et asynchrone), l'identification de l'orateur et l'attribution de phrases à chaque orateur (également appelée *diarisation*). Il est parfait pour la transcription de rencontres en personne, avec possibilité de distinguer les orateurs.

La **Transcription de conversation** est disponible sur les plateformes suivantes :

  - C++/Windows & Linux
  - C# (Framework & .NET Core)/Windows & UWP & Linux
  - Java/Windows & Linux & Android (SDK Speech Devices)

#### <a name="multi-device-conversation"></a>Conversation multi-appareil

Avec la [Conversation multi-appareils](multi-device-conversation.md), connectez plusieurs appareils ou clients à une conversation pour envoyer des messages vocaux ou textuels, avec une prise en charge aisée de la transcription et de la traduction.

La **Conversation multi-appareils** est disponible sur les plateformes suivantes :

  - C++/Windows
  - C# (Framework & .NET Core)/Windows

### <a name="custom--agent-scenarios"></a>Scénarios personnalisés / d'agent

Le SDK Speech peut être utilisé pour transcrire des scénarios de centre d'appels, dans lesquels des données téléphoniques sont générées.

#### <a name="call-center-transcription"></a>Transcription de centre d’appel

La [Transcription de centre d'appels](call-center-transcription.md) est un scénario courant de reconnaissance vocale pour la transcription de gros volumes de données téléphoniques qui peuvent provenir de différents systèmes, tels que la réponse vocale interactive (RVI). Les derniers modèles de reconnaissance vocale du service Speech excellent à retranscrire ces données de téléphonie, même quand celles-ci sont difficiles à comprendre pour un humain.

La **Transcription de centre d'appels** est disponible par le biais du service Reconnaissance vocale par lots via son API REST et peut être utilisée dans n’importe quelle situation.

### <a name="codec-compressed-audio-input"></a>Entrée audio compressée par codec

Plusieurs des langages de programmation du SDK Speech prennent en charge les flux d'entrée audio compressés par codec. Pour plus d'informations, consultez <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-use-codec-compressed-audio-input-streams" target="_blank">Utiliser des formats d'entrée audio compressés<span class="docon docon-navigate-external x-hidden-focus"></span></a>.

L’**entrée audio compressée par codec** est disponible sur les plateformes suivantes :

  - C++/Linux
  - C#/Linux
  - Java/Linux, Android et iOS

## <a name="rest-api"></a>API REST

Bien que le SDK Speech englobe de nombreuses fonctionnalités du service Speech, pour certains scénarios, vous pouvez utiliser l'API REST.

### <a name="batch-transcription"></a>Transcription Batch

La [Transcription par lots](batch-transcription.md) permet de transcrire une reconnaissance vocale asynchrone de gros volumes de données. Elle n'est possible qu'à partir de l'API REST. En plus de convertir l'audio en texte, la reconnaissance vocale par lots permet également la diarisation et l'analyse des sentiments.

## <a name="customization"></a>Personnalisation

Le service Speech fournit d'excellentes fonctionnalités avec ses modèles par défaut dédiés à la reconnaissance vocale, à la synthèse vocale et à la traduction vocale. Vous pouvez également améliorer les performances de base du service pour l'adapter à votre cas d'usage unique. Le service Speech propose différents outils de personnalisation sans code qui vous facilitent la tâche et vous permettent de bénéficier d'un avantage concurrentiel grâce à des modèles personnalisés basés sur vos propres données. Votre organisation et vous êtes les seuls à avoir accès à ces modèles.

### <a name="custom-speech-to-text"></a>Reconnaissance vocale personnalisée

Si vous utilisez la reconnaissance vocale pour la reconnaissance et la transcription dans un environnement unique, vous pouvez créer et former des modèles acoustiques, linguistiques et de prononciation personnalisés pour traiter le bruit ambiant ou le vocabulaire spécifique au secteur. La création et la gestion de modèles vocaux personnalisés sans code sont disponibles via le [portail Custom Speech](https://aka.ms/customspeech). Une fois le modèle Custom Speech publié, il peut être utilisé par le SDK Speech.

### <a name="custom-text-to-speech"></a>Synthèse vocale personnalisée

La synthèse vocale personnalisée, également appelée Custom Voice, est un ensemble d'outils en ligne qui vous permettent de créer une voix unique reconnaissable entre toutes pour votre marque. La création et la gestion de modèles Custom Voice sans code sont disponibles via le [portail Custom Voice](https://aka.ms/customvoice). Une fois le modèle Custom Voice publié, il peut être utilisé par le SDK Speech.

## <a name="get-the-speech-sdk"></a>Obtenir le Kit de développement logiciel (SDK) de reconnaissance vocale

# <a name="windows"></a>[Windows](#tab/windows)

[!INCLUDE [Get the Speech SDK](includes/get-speech-sdk-windows.md)]

# <a name="linux"></a>[Linux](#tab/linux)

[!INCLUDE [Get the Speech SDK](includes/get-speech-sdk-linux.md)]

# <a name="ios"></a>[iOS](#tab/ios)

[!INCLUDE [Get the Speech SDK](includes/get-speech-sdk-ios.md)]

# <a name="macos"></a>[macOS](#tab/macos)

[!INCLUDE [Get the Speech SDK](includes/get-speech-sdk-macos.md)]

# <a name="android"></a>[Android](#tab/android)

[!INCLUDE [Get the Speech SDK](includes/get-speech-sdk-android.md)]

# <a name="nodejs"></a>[Node.JS](#tab/nodejs)

[!INCLUDE [Get the Node.js Speech SDK](includes/get-speech-sdk-nodejs.md)]

# <a name="browser"></a>[Browser](#tab/browser)

[!INCLUDE [Get the Browser Speech SDK](includes/get-speech-sdk-browser.md)]

---

[!INCLUDE [License notice](../../../includes/cognitive-services-speech-service-license-notice.md)]

[!INCLUDE [Sample source code](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]

## <a name="next-steps"></a>Étapes suivantes

* [Créez un compte Azure gratuit](https://azure.microsoft.com/free/cognitive-services/)
* [Découvrir comment utiliser la reconnaissance vocale en C#](./get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=dotnet)