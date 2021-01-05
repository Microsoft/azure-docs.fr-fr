---
title: Assistants vocaux - Service Speech
titleSuffix: Azure Cognitive Services
description: Vue d’ensemble des fonctionnalités et des restrictions des assistants vocaux utilisant le kit de développement logiciel (SDK) Speech.
services: cognitive-services
author: trrwilson
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: travisw
ms.openlocfilehash: 09859be61029242e987603e01b5c3fba8dffb2ca
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483884"
---
# <a name="what-is-a-voice-assistant"></a>Qu’est-ce qu’un Assistant vocal ?

Les assistants vocaux qui utilisent le service Speech permettent aux développeurs de créer des interfaces conversationnelles naturelles pour leurs applications et leurs expériences.

Le service d’assistant vocal permet une interaction rapide et fiable entre un appareil et une implémentation d’assistant qui utilise soit (1) [Direct Line Speech](direct-line-speech.md) (via Azure Bot Service) pour l’ajout de fonctionnalités vocales à vos bots, soit (2) Commandes personnalisées pour les scénarios de commande vocale.

## <a name="choosing-an-assistant-solution"></a>Choix d’une solution d’Assistant

La première étape de création d’un Assistant vocal est de décider de ce qu’il doit faire. Le service Speech fournit plusieurs solutions complémentaires pour la création des interactions de votre Assistant. Vous pouvez ajouter des fonctionnalités vocales entrantes et sortantes à votre bot flexible et polyvalent créé à l’aide d’Azure Bot Service à l’aide du canal [Direct Line Speech](direct-line-speech.md), ou tirer parti de la simplicité de la création d’une application [Commandes personnalisées](custom-commands.md) pour des scénarios de commande vocale simples.

| Si vous voulez... | Envisagez... | Par exemple... |
|-------------------|------------------|----------------|
|Conversation libre avec une intégration de compétences robustes et un contrôle complet du déploiement | Bot Azure Bot Service avec canal [Direct Line Speech](direct-line-speech.md) | <ul><li>« Je dois aller à Seattle »</li><li>« Quel genre de pizza puis-je commander ? »</li></ul>
|Conversations de commande vocale ou commandes simples axées sur des tâches avec création et hébergement simplifiés | [Commandes personnalisées](custom-commands.md) | <ul><li>« Allumer le plafonnier »</li><li>« Augmenter la température de 5 degrés »</li><li>D’autres exemples sont [disponibles ici](https://speech.microsoft.com/customcommands)</li></ul>

Nous recommandons [Direct Line Speech](direct-line-speech.md) comme meilleur choix par défaut si vous n’êtes pas encore sûr de ce que votre Assistant doit traiter. Il offre une intégration avec un ensemble complet d’outils et d’aides à la création, comme le [modèle de solution d’Assistant virtuel et d’entreprise](/azure/bot-service/bot-builder-enterprise-template-overview) et le [service QnA Maker](../qnamaker/overview/overview.md) pour créer des modèles courants et utiliser vos sources de connaissances existantes.

Les [commandes personnalisées](custom-commands.md) facilitent la création d’applications de commandes vocales complètes, optimisées pour les expériences d’interaction de type « voice-first ». Elles offrent une expérience de création unifiée, un modèle d’hébergement automatique et une complexité relativement inférieure, vous permettant de vous concentrer sur la conception de la meilleure solution pour vos scénarios de commandes vocales.

   ![Comparaison des solutions Assistant](media/voice-assistants/assistant-solution-comparison.png "Comparaison des solutions Assistant")


## <a name="reference-architecture-for-building-a-voice-assistant-using-the-speech-sdk"></a>Architecture de référence pour la création d’un assistant vocal à l’aide du kit de développement logiciel (SDK) Speech

   ![Diagramme conceptuel du flux du service d’orchestration de l’assistant vocal](media/voice-assistants/overview.png "Flux de l’assistant vocal")

## <a name="core-features"></a>Fonctionnalités de base

Que vous choisissiez [Direct Line Speech](direct-line-speech.md) ou [Commandes personnalisées](custom-commands.md) pour créer les interactions de votre Assistant, vous pouvez utiliser un ensemble complet de fonctionnalités de personnalisation pour personnaliser votre Assistant avec votre marque, votre produit et votre personnalité.

| Category | Fonctionnalités |
|----------|----------|
|[Mot clé personnalisé](./custom-keyword-basics.md) | Les utilisateurs peuvent engager des conversations avec les assistants au moyen de mots clés personnalisés, tels que « Bonjour Contoso ». Une application effectue cette action avec un moteur de mot clé personnalisé dans le SDK Speech, qui peut être configuré avec un mot déclencheur personnalisé [que vous pouvez générer ici](./custom-keyword-basics.md). Les assistants vocaux peuvent utiliser la vérification des mots clés côté service pour améliorer la précision de l’activation des mots clés (par rapport à l’appareil seul).
|[Reconnaissance vocale](speech-to-text.md) | Les assistants vocaux convertissent en temps réel le contenu audio en texte écrit à l’aide de la [Reconnaissance vocale](speech-to-text.md) du service Speech. Lors de sa transcription, ce texte est disponible pour l’implémentation de votre assistant et votre application cliente.
|[Synthèse vocale](text-to-speech.md) | Les réponses textuelles de votre assistant seront synthétisées à l’aide de la [Synthèse vocale](text-to-speech.md) du service Speech. Cette synthèse est ensuite disponible pour votre application cliente sous forme de flux audio. Microsoft offre la possibilité de créer votre propre voix TTS neuronale personnalisée et de haute qualité qui donne une voix à votre marque. Pour en savoir plus, [contactez-nous](mailto:mstts@microsoft.com).

## <a name="getting-started-with-voice-assistants"></a>Prise en main des assistants vocaux

Nous proposons des guides de démarrage rapide conçus pour vous permettre d’exécuter du code en moins de 10 minutes. Ce tableau comprend une liste de guides de démarrage rapide pour les assistants organisés par langage.

* [Démarrage rapide : Créer un assistant vocal personnalisé avec Direct Line Speech](quickstarts/voice-assistants.md)
* [Démarrage rapide : Créer une application de commande vocale à l’aide de Commandes personnalisées](quickstart-custom-commands-application.md)

## <a name="sample-code-and-tutorials"></a>Exemple de code et tutoriels

Un exemple de code pour la création d’un assistant vocal est disponible sur GitHub. Ces exemples couvrent l’application cliente pour la connexion à votre assistant dans plusieurs langages de programmation populaires.

* [Exemples d’assistant vocal sur GitHub](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant)
* [Tutoriel : Activation vocale de votre assistant créé à l’aide d’Azure Bot Service avec le kit de développement logiciel (SDK) Speech C#](tutorial-voice-enable-your-bot-speech-sdk.md)
* [Tutoriel : Création d’une application Commandes personnalisées avec des commandes vocales simples](how-to-custom-commands-create-application-with-simple-commands.md)

## <a name="customization"></a>Personnalisation

Les assistants vocaux créés à l’aide des services Azure Speech peuvent utiliser toute la gamme des options de personnalisation.

* [Discours personnalisé](./custom-speech-overview.md)
* [Custom Voice](how-to-custom-voice.md)
* [Mot clé personnalisé](custom-keyword-overview.md)

> [!NOTE]
> Les options de personnalisation varient selon la langue/paramètres régionaux (voir [Langues prises en charge](language-support.md)).

## <a name="next-steps"></a>Étapes suivantes

* [Obtenir gratuitement une clé d’abonnement au service Speech](overview.md#try-the-speech-service-for-free)
* [En savoir plus sur Commandes personnalisées](custom-commands.md)
* [En savoir plus sur Direct Line Speech](direct-line-speech.md)
* [Obtenir le kit SDK Speech](speech-sdk.md)