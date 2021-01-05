---
title: Qu’est-ce que le service de reconnaissance vocale ?
titleSuffix: Azure Cognitive Services
description: Le service Speech rassemble la reconnaissance vocale, la synthèse vocale et la traduction vocale dans un seul abonnement Azure. Ajoutez les services Speech à vos applications, outils et appareils avec le SDK Speech, le SDK Speech Devices ou les API REST.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: overview
ms.date: 11/23/2020
ms.author: trbye
ms.openlocfilehash: 3b20773de62f036b507f1612f2b38d613d54ba2f
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96485193"
---
# <a name="what-is-the-speech-service"></a>Qu’est-ce que le service de reconnaissance vocale ?

Le service Speech réunit la reconnaissance vocale, la synthèse vocale et la traduction vocale dans un même abonnement Azure. Vous pouvez aisément activer vos applications, outils et appareils pour les services [CLI Speech](spx-overview.md), [SDK Speech](./speech-sdk.md), [SDK Speech Devices](./speech-devices-sdk-quickstart.md?pivots=platform-android), [Speech Studio](https://speech.microsoft.com/) ou les [API REST](#reference-docs).

> [!IMPORTANT]
> Le service Speech remplace l’API Reconnaissance vocale Bing et Traduction de conversation Translator Speech. Pour obtenir des instructions de migration, consultez la section _Migration_.

Les fonctionnalités suivantes font partie du service Speech. Pour en savoir plus sur les cas d’utilisation courants de chaque fonctionnalité, utilisez les liens figurant dans ce tableau, ou parcourez la référence de l’API.

| Service | Fonctionnalité | Description | Kit SDK | REST |
|---------|---------|-------------|-----|------|
| [Reconnaissance vocale](speech-to-text.md) | Reconnaissance vocale en temps réel | La reconnaissance vocale transcrit ou traduit en temps réel des fichiers locaux ou des flux audio en texte que vos applications, outils ou appareils peuvent consommer ou afficher. Utilisez la reconnaissance vocale avec [LUIS (Language Understanding Intelligent Service)](../luis/index.yml) pour déduire les intentions de l’utilisateur à partir des transcriptions et agir sur des commandes vocales. | [Oui](./speech-sdk.md) | [Oui](#reference-docs) |
| | [Reconnaissance vocale par lots](batch-transcription.md) | Transcrit de manière asynchrone la reconnaissance vocale de grandes quantités de données audio stockées dans Stockage Blob Azure. En plus de convertir l’audio en texte, la reconnaissance vocale par lots permet également d’effectuer la diarisation et l’analyse des sentiments. | Non | [Oui](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0) |
| | [Conversation multi-appareil](multi-device-conversation.md) | Connecter plusieurs appareils ou clients dans une conversation pour envoyer des messages vocaux ou textuels, avec une prise en charge simple de la transcription et de la traduction| Oui | Non |
| | [Transcription de conversation](./conversation-transcription.md) | Permet la reconnaissance vocale en temps réel, l’identification de l’orateur et la structuration (diarisation). Il est parfait pour la transcription de rencontres en personne, avec possibilité de distinguer les orateurs. | Oui | Non |
| | [Créer des modèles vocaux personnalisés](#customize-your-speech-experience) | Si vous utilisez la reconnaissance vocale pour la reconnaissance et la transcription dans un environnement unique, vous pouvez créer et former des modèles de prononciation, de langue et acoustiques personnalisés pour prendre en compte un bruit ambiant ou le vocabulaire spécifique d’un secteur. | Non | [Oui](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0) |
| [Synthèse vocale](text-to-speech.md) | Synthèse vocale | La synthèse vocale convertit le texte d’entrée en parole synthétisée quasi humaine avec le [langage SSML (Speech Synthesis Markup Language)](speech-synthesis-markup.md). Faites votre choix parmi les voix standard et les voix neuronales (voir [Prise en charge linguistique](language-support.md)). | [Oui](./speech-sdk.md) | [Oui](#reference-docs) |
| | [Créer des voix personnalisées](#customize-your-speech-experience) | Créez des polices de voix personnalisées propres à vos marques ou produits. | Non | [Oui](#reference-docs) |
| [Traduction vocale](speech-translation.md) | Traduction vocale | La traduction vocale permet à vos applications, outils et appareils d’effectuer de la traduction multilingue en temps réel de la parole. Utilisez ce service pour la traduction de voix en voix et de voix en texte. | [Oui](./speech-sdk.md) | Non |
| [Assistants vocaux](voice-assistants.md) | Assistants vocaux | Les assistants vocaux qui utilisent le service Speech permettent aux développeurs de créer des interfaces conversationnelles naturelles pour leurs applications et leurs expériences. Le service d’assistant vocal permet une interaction rapide et fiable entre un appareil et une implémentation d’assistant qui utilise le canal Direct Line Speech de Bot Framework ou le service intégré Custom Commands (préversion) pour réaliser la tâche. | [Oui](voice-assistants.md) | Non |
| [Reconnaissance de l’orateur](speaker-recognition-overview.md) | Identification et vérification de l’orateur | Le service Reconnaissance de l’orateur fournit des algorithmes qui vérifient et identifient les orateurs d’après leurs caractéristiques vocales uniques. Le service Reconnaissance de l’orateur est utilisé pour répondre à la question « qui parle ? ». | Oui | [Oui](/rest/api/speakerrecognition/) |


[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

## <a name="try-the-speech-service-for-free"></a>Essayez le service Speech gratuitement

Pour les étapes suivantes, vous devez disposer d'un compte Microsoft et d’un compte Azure. Si vous n’avez pas de compte Microsoft, vous pouvez vous inscrire gratuitement sur le [portail de compte Microsoft](https://account.microsoft.com/account). Sélectionnez **Se connecter avec Microsoft** puis, à l’invite de connexion, sélectionnez **Créer un compte Microsoft**. Suivez les étapes pour créer et vérifier votre nouveau compte Microsoft.

Lorsque vous disposez d’un compte Microsoft, accédez à la [page d’inscription Azure](https://azure.microsoft.com/free/ai/), sélectionnez **Démarrer gratuitement** et créez un compte Azure à l’aide de votre compte Microsoft. Voici une vidéo montrant [comment s’inscrire pour obtenir un compte gratuit Azure](https://www.youtube.com/watch?v=GWT2R1C_uUU).

> [!NOTE]
> Quand vous vous inscrivez pour obtenir un compte Azure gratuit, celui-ci est fourni avec 200 $ de crédit de service que vous pouvez utiliser pour un abonnement au service Speech payant, valide pendant 30 jours. Vos services Azure sont désactivés dès que vous avez épuisé tous vos crédits ou que les crédits arrivent à expiration à la fin des 30 jours. Pour continuer à utiliser les services Azure, vous devez alors mettre à niveau votre compte. Pour plus d’informations, consultez [Guide pratique pour mettre à niveau votre compte gratuit Azure](../../cost-management-billing/manage/upgrade-azure-subscription.md). 
>
> Le service Speech propose deux niveaux de service, gratuit (f0) et abonnement (s0), chacun présentant des limitations et des avantages différents. Si vous utilisez le niveau de service Speech gratuit à faible volume, vous pouvez conserver cet abonnement gratuit, même après l’expiration de votre évaluation gratuite ou de votre crédit de service. Pour plus d’informations, consultez [Tarifs de Cognitive Services - Service Speech](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

### <a name="create-the-azure-resource"></a>Créer une ressource Azure

Pour ajouter une ressource de service Speech (niveau gratuit ou payant) à votre compte Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en utilisant votre compte Microsoft.

1. Sélectionnez **Créer une ressource** en haut à gauche du portail. Si vous ne voyez pas l’option **Créer une ressource**, vous pouvez y accéder en sélectionnant le menu réduit situé en haut à gauche de l’écran :

1. Dans la fenêtre **Nouveau**, saisissez « speech » dans la zone de recherche et appuyez sur ENTRÉE.

1. Dans les résultats de la recherche, sélectionnez **Speech**.

   ![résultats de la recherche speech](media/index/speech-search.png)

1. Sélectionnez **Créer**, puis :

   - Donnez un nom unique à votre nouvelle ressource. Ce nom vous permet de faire la distinction entre plusieurs abonnements liés au même service.
   - Choisissez l’abonnement Azure associé à la nouvelle ressource pour déterminer la façon dont les frais sont facturés. Voici la présentation de la [marche à suivre pour créer un abonnement Azure](../../cost-management-billing/manage/create-subscription.md#create-a-subscription-in-the-azure-portal) dans le portail Azure.
   - Choisissez la [région](regions.md) où la ressource sera utilisée. Azure est une plateforme cloud mondiale, disponible dans de nombreuses régions du monde. Pour obtenir des performances optimales, sélectionnez la région la plus proche de vous ou l’endroit où votre application s’exécute. Les disponibilités du service Speech varient selon les régions. Veillez à créer votre ressource dans une région prise en charge. Consultez la [prise en charge des régions pour les services Speech](./regions.md#speech-to-text-text-to-speech-and-translation).
   - Choisissez soit un niveau tarifaire payant (S0) ou gratuit (F0). Pour obtenir des informations complètes sur les tarifs et les quotas d’utilisation pour chaque niveau, sélectionnez **Afficher tous les détails de la tarification** ou consultez les [tarifs des services vocaux](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/). Pour connaître les limites des ressources, consultez [Limites d’Azure Cognitive Services](../../azure-resource-manager/management/azure-subscription-service-limits.md#azure-cognitive-services-limits).
   - Créez un groupe de ressources pour cet abonnement Speech ou affectez l’abonnement à un groupe de ressources existant. Les groupes de ressources vous permettent d’organiser vos différents abonnements Azure.
   - Sélectionnez **Create** (Créer). Cela vous permet d’atteindre la vue d’ensemble du déploiement et d’afficher les messages de progression de ce dernier.  
<!--
> [!NOTE]
> You can create an unlimited number of standard-tier subscriptions in one or multiple regions. However, you can create only one free-tier subscription. Model deployments on the free tier that remain unused for 7 days will be decommissioned automatically.
-->
Le déploiement de votre nouvelle ressource Speech prend quelques instants. 

### <a name="find-keys-and-region"></a>Rechercher les clés et la région

Pour rechercher les clés et la région d’un déploiement terminé, effectuez les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en utilisant votre compte Microsoft.

2. Sélectionnez **Toutes les ressources**, puis le nom de votre ressource Cognitive Services.

3. Dans le volet gauche, sous **GESTION DES RESSOURCES**, sélectionnez **Clés et point de terminaison**.

Chaque abonnement dispose de deux clés ; vous pouvez utiliser l’une ou l’autre dans votre application. Pour copier-coller une clé dans votre éditeur de code ou dans un autre emplacement, cliquez sur le bouton Copier à côté de chaque clé, puis changez de fenêtre pour coller le contenu du Presse-papiers à l’emplacement souhaité.

Copiez aussi la valeur `LOCATION`, qui correspond à votre ID de région (par exemple, `westus`, `westeurope`) pour les appels des SDK.

> [!IMPORTANT]
> Ces clés d’abonnement sont utilisées pour accéder à votre API Cognitive Service. Ne partagez pas vos clés. Stockez-les en toute sécurité, par exemple, à l’aide d’Azure Key Vault. Nous vous recommandons également de régénérer ces clés régulièrement. Une seule clé est nécessaire pour effectuer un appel d’API. Lors de la régénération de la première clé, vous pouvez utiliser la deuxième clé pour un accès continu au service.

## <a name="complete-a-quickstart"></a>Suivre un guide de démarrage rapide

Nous proposons des guides de démarrage rapide pour la plupart des langages de programmation. Chaque guide est conçu pour vous montrer des modèles de conception de base et vous permettre d’exécuter du code en moins de 10 minutes. Consultez la liste suivante pour connaître le guide de démarrage rapide de chaque fonctionnalité.

* [Guide de démarrage rapide sur la reconnaissance vocale](get-started-speech-to-text.md)
* [Démarrage rapide sur la conversion de texte par synthèse vocale](get-started-text-to-speech.md)
* [Guide de démarrage rapide sur la traduction vocale](./get-started-speech-translation.md)
* [Guide de démarrage rapide sur la reconnaissance de l’intention](quickstarts/intent-recognition.md)
* [Guide de démarrage rapide sur la reconnaissance de l’orateur](./get-started-speaker-recognition.md)

Une fois que vous avez suivi le guide de démarrage rapide du service Speech, essayez nos tutoriels qui vous feront découvrir différents scénarios.

- [Tutoriel : Effectuer une reconnaissance des intentions du discours à l’aide du Kit de développement logiciel (SDK) Speech et de LUIS, C#](how-to-recognize-intents-from-speech-csharp.md)
- [Tutoriel : Activer les fonctions vocales sur votre bot avec le SDK Speech, C#](tutorial-voice-enable-your-bot-speech-sdk.md)
- [Tutoriel : Créer une application Flask pour la traduction de texte, l’analyse de sentiments et la synthèse vocale de texte traduit, REST](../translator/tutorial-build-flask-app-translation-synthesis.md?bc=%2fazure%2fcognitive-services%2fspeech-service%2fbreadcrumb%2ftoc.json%252c%2fen-us%2fazure%2fbread%2ftoc.json&toc=%2fazure%2fcognitive-services%2fspeech-service%2ftoc.json%252c%2fen-us%2fazure%2fcognitive-services%2fspeech-service%2ftoc.json)

## <a name="get-sample-code"></a>Obtenir un exemple de code

Un exemple de code est disponible sur GitHub pour le service Speech. Ces exemples couvrent des scénarios courants tels que la lecture du signal audio d’un fichier ou d’un flux, la reconnaissance continue et ponctuelle, et l’utilisation de modèles personnalisés. Pour voir les exemples SDK et REST, suivez ces liens :

- [Exemples de reconnaissance vocale, de synthèse vocale et de traduction vocale (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)
- [Exemples de transcription par lot (REST)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/batch)
- [Exemples de synthèse vocale (REST)](https://github.com/Azure-Samples/Cognitive-Speech-TTS)
- [Exemples d’assistant vocal (Kit de développement logiciel [SDK])](https://aka.ms/csspeech/samples)

## <a name="customize-your-speech-experience"></a>Personnaliser votre expérience de reconnaissance vocale

Le service Speech fonctionne bien avec les modèles intégrés. Cependant, vous pouvez personnaliser et optimiser l’expérience pour votre produit ou environnement. Les options de personnalisation vont de l’ajustement du modèle acoustique aux polices de voix uniques pour votre marque.

D’autres produits proposent des modèles vocaux adaptés à des fins spécifiques, telles que la santé ou l’assurance, mais sont disponibles pour tous les publics. Dans Azure Speech, la personnalisation constitue *votre avantage concurrentiel*, qui n’est pas celui des autres utilisateurs ou clients. En d’autres termes, vos modèles sont privés et adaptés à votre cas d’utilisation.

| Speech Service | Plateforme | Description |
| -------------- | -------- | ----------- |
| Reconnaissance vocale | [Discours personnalisé](https://aka.ms/customspeech) | Personnalisez les modèles de reconnaissance vocale en fonction de vos besoins et des données disponibles. Éliminez les obstacles à la reconnaissance vocale, tels que le style d’élocution, le bruit de fond et le vocabulaire. |
| Synthèse vocale | [Custom Voice](https://aka.ms/customvoice) | Créez une voix unique et reconnaissable pour vos applications de synthèse vocale, avec vos données vocales disponibles. Vous pouvez ensuite affiner le réglage des sorties vocales en ajustant tout un ensemble de paramètres vocaux. |

## <a name="deploy-on-premises-using-docker-containers"></a>Déployer localement en utilisant des conteneurs Docker

[Utilisez des conteneurs du service Speech](speech-container-howto.md) pour déployer localement des fonctionnalités d’API. Ces conteneurs Docker vous donnent la possibilité de rapprocher le service plus près de vos données, ce qui peut être souhaitable pour des raisons de conformité, de sécurité ou opérationnelles. Le service Speech offre les conteneurs suivants :

* Reconnaissance vocale standard
* Reconnaissance vocale personnalisée
* Synthèse vocale standard
* Synthèse vocale neuronale
* Synthèse vocale personnalisée (préversion)
* Détection de langue vocale (préversion)

## <a name="reference-docs"></a>Documents de référence

- [Kit de développement logiciel (SDK) de reconnaissance vocale](./speech-sdk.md)
- [SDK Speech Devices](speech-devices-sdk.md)
- [API REST : Reconnaissance vocale](rest-speech-to-text.md)
- [API REST : Synthèse vocale](rest-text-to-speech.md)
- [API REST : Transcription et personnalisation par lot](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0)


## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Bien démarrer avec la reconnaissance vocale](./get-started-speech-to-text.md)
> [Bien démarrer avec la synthèse vocale](get-started-text-to-speech.md)