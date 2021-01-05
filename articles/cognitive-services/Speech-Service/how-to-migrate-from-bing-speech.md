---
title: Migrer de la Reconnaissance vocale Bing vers le service Speech
titleSuffix: Azure Cognitive Services
description: Découvrez comment migrer un abonnement Reconnaissance vocale Bing existant vers le service Speech d’Azure Cognitive Services.
services: cognitive-services
author: wsturman
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/03/2020
ms.author: nitinme
ms.openlocfilehash: d0ffd786d3fb6bb5f0d70095d947c81caa070518
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499133"
---
# <a name="migrate-from-bing-speech-to-the-speech-service"></a>Migrer de la Reconnaissance vocale Bing vers le service Speech

Référez-vous à cet article pour migrer vos applications de l’API Reconnaissance vocale Bing vers le service Speech.

Cet article décrit les différences qui existent entre les API Reconnaissance vocale Bing et le service Speech, et suggère des stratégies pour la migration de vos applications. Votre clé d’abonnement API Reconnaissance vocale Bing ne fonctionnera pas avec le service Speech. Vous avez besoin d’un nouvel abonnement au service Speech.

Une clé unique d’abonnement au service Speech permet d’accéder aux fonctionnalités suivantes. Elles sont mesurées séparément pour que seules les fonctions que vous utilisez vous soient facturées.

* [Reconnaissance vocale](speech-to-text.md)
* [Reconnaissance vocale personnalisée](https://cris.ai)
* [Synthèse vocale](text-to-speech.md)
* [Voix de synthèse vocale personnalisées](./how-to-custom-voice-create-voice.md)
* [Traduction vocale](speech-translation.md) (n’inclut pas la [traduction de texte](../translator/translator-info-overview.md))

Le [Kit de développement logiciel (SDK) Speech](speech-sdk.md) constitue un remplacement fonctionnel des bibliothèques clientes de Reconnaissance vocale Bing, mais utilise une API différente.

## <a name="comparison-of-features"></a>Comparaison des fonctionnalités

Le service Speech est très similaire à la Reconnaissance vocale Bing, à l’exception des différences suivantes.

| Fonctionnalité | Reconnaissance vocale Bing | Service Speech | Détails |
|--|--|--|--|
| Kit de développement logiciel (SDK) C# | :heavy_check_mark: | :heavy_check_mark: | Le service Speech prend en charge Windows 10, la plateforme Windows universelle (UWP) et .NET Standard 2.0. |
| KIT DE DÉVELOPPEMENT LOGICIEL C++ | :heavy_minus_sign: | :heavy_check_mark: | Le service Speech prend en charge Windows et Linux. |
| Kit de développement logiciel (SDK) Java | :heavy_check_mark: | :heavy_check_mark: | Le service Speech prend en charge Android et Speech Devices. |
| Reconnaissance vocale continue | 10 minutes | Illimité | Le kit de développement logiciel (SDK) Speech prend en charge une reconnaissance continue illimitée et se reconnecte automatiquement après un délai d’expiration ou une déconnexion. |
| Résultats partiels ou intermédiaires | :heavy_check_mark: | :heavy_check_mark: | Pris en charge avec le kit de développement logiciel (SDK) Speech. |
| Modèles vocaux personnalisés | :heavy_check_mark: | :heavy_check_mark: | La Reconnaissance vocale Bing nécessite un abonnement Custom Speech séparé. |
| Polices de la voix personnalisées | :heavy_check_mark: | :heavy_check_mark: | La Reconnaissance vocale Bing nécessite un abonnement Custom Voice séparé. |
| Voix 24 kHz | :heavy_minus_sign: | :heavy_check_mark: |
| Reconnaissance de l’intention vocale | Nécessite un appel d’API LUIS séparé | Intégrée (avec Kit de développement logiciel (SDK)) | Vous pouvez utiliser une clé LUIS avec le service Speech. |
| Reconnaissance de l’intention simple | :heavy_minus_sign: | :heavy_check_mark: |
| Transcription par lots de fichiers audio longs | :heavy_minus_sign: | :heavy_check_mark: |
| Mode de reconnaissance | Manuel via URI de point de terminaison | Automatique | Le mode de reconnaissance n’est pas disponible dans le service Speech. |
| Lieu du point de terminaison | Global | Zones géographiques | Les points de terminaison régionaux améliorent la latence. |
| API REST | :heavy_check_mark: | :heavy_check_mark: | Les API REST du service Speech sont compatibles avec la Reconnaissance vocale Bing (point de terminaison différent). Les API REST prennent en charge les fonctionnalités de synthèse vocale et de reconnaissance vocale limitée. |
| Protocoles WebSocket | :heavy_check_mark: | :heavy_minus_sign: | Le kit de développement logiciel (SDK) Speech récupère les connexions de socket Web pour les fonctionnalités qui requièrent une connexion constante au service, de sorte qu’il n’y a plus de prise en charge pour s’y abonner manuellement. |
| Appels d’API de service à service | :heavy_check_mark: | :heavy_minus_sign: | Fournis dans la Reconnaissance vocale Bing via la bibliothèque de services C#. |
| Kit de développement logiciel (SDK) open source | :heavy_check_mark: | :heavy_minus_sign: |

Le service Speech utilise un modèle tarifaire basé sur un temps donné (plutôt que sur les transactions). Pour plus de détails, consultez les [tarifs du service Speech](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

## <a name="migration-strategies"></a>Stratégies de migration

Si votre organisation ou vous-même disposez d’applications en développement ou en production qui utilisent l’API Reconnaissance vocale Bing, vous devez les mettre à jour pour qu’elles utilisent le service Speech dès que possible. Pour accéder aux kits SDK, aux exemples de code et aux tutoriels disponibles, consultez la [documentation du service Speech](index.yml).

Les [API REST](./overview.md#reference-docs) du service Speech sont compatibles avec les API Reconnaissance vocale Bing. Si vous utilisez les API REST Reconnaissance vocale Bing, vous devez uniquement changer le point de terminaison REST et basculer vers une clé d’abonnement Speech.

Si vous utilisez une bibliothèque de client Reconnaissance vocale Bing pour un langage de programmation spécifique, la migration vers le [Kit de développement logiciel (SDK) Speech](speech-sdk.md) nécessite l’apport de modifications à votre application, car l’API est différente. Le kit de développement logiciel (SDK) Speech peut simplifier votre code tout en vous donnant accès à de nouvelles fonctionnalités. Le kit SDK Speech est disponible dans un large éventail de langages de programmation. Les API sont similaires sur toutes les plateformes, et facilitent le développement multi-plateforme.

Le service Speech n’offre pas de point de terminaison global. Vous devez déterminer si votre application fonctionne efficacement lorsqu’elle utilise un point de terminaison régional unique pour tout son trafic. Autrement, utilisez la géolocalisation pour déterminer le point de terminaison plus efficace. Vous devez disposer d’un abonnement Speech distinct pour chaque région.

Pour prendre en main du kit de développement logiciel (SDK) Speech :

1. Téléchargez le [Kit de développement logiciel (SDK) Speech](speech-sdk.md).
1. Parcourez les [guides de démarrage rapide](./get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=dotnet) et les [tutoriels](how-to-recognize-intents-from-speech-csharp.md) relatifs au service Speech. Consultez également les [exemples de code](./speech-sdk.md#sample-source-code) pour vous familiariser avec les nouvelles API.
1. Mettez à jour votre application pour utiliser le service Speech.

## <a name="support"></a>Support

Les clients de Reconnaissance vocale Bing doivent contacter le service clientèle en ouvrant un [ticket de support](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). Vous pouvez également nous contacter si vous avez besoin d’un [plan de support technique](https://azure.microsoft.com/support/plans/).

Pour la prise en charge du service Speech, du SDK et de l’API, consultez la [page de support](../cognitive-services-support-options.md?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext%253fcontext%253d%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext) du service Speech.

## <a name="next-steps"></a>Étapes suivantes

* [Essayer gratuitement le service Speech](overview.md#try-the-speech-service-for-free)
* [Commencer avec la reconnaissance vocale](get-started-speech-to-text.md)
* [Bien démarrer avec la conversion de texte par synthèse vocale](get-started-text-to-speech.md)

## <a name="see-also"></a>Voir aussi

* [Notes de publication du service Speech](releasenotes.md)
* [Qu’est-ce que le Service Speech](overview.md)
* [Documentation sur le SDK Speech et sur le service Speech](speech-sdk.md#get-the-speech-sdk)
