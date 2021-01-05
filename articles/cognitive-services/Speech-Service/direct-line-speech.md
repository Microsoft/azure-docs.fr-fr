---
title: Direct Line Speech - Service Speech
titleSuffix: Azure Cognitive Services
description: Vue d’ensemble des fonctionnalités et des restrictions des assistants vocaux utilisant Direct Line Speech avec le kit de développement logiciel (SDK) Speech.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: trbye
ms.openlocfilehash: e8c7e21b0784aacc85bd02e3c1702e1a710e76ab
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021879"
---
# <a name="what-is-direct-line-speech"></a>Qu’est-ce que Direct Line Speech ?

**Direct Line Speech** est une solution de bout en bout robuste permettant de créer un assistant vocal extensible et flexible. Elle est alimentée par Bot Framework et son canal Direct Line Speech, qui est optimisé pour l’interaction vocale avec les bots.

Les [assistants vocaux](voice-assistants.md) écoutent les utilisateurs et réagissent en prenant une mesure, souvent en parlant. Ils utilisent la [reconnaissance vocale](speech-to-text.md) pour transcrire la parole de l’utilisateur, puis agissent sur la compréhension du langage naturel du texte. Cette action comprend fréquemment une parole de l’assistant générée à l’aide de la [synthèse vocale](text-to-speech.md).

Direct Line Speech offre les plus hauts niveaux de personnalisation et de sophistication pour les assistants vocaux. Il est conçu pour les scénarios de conversation qui sont ouverts, naturels ou hybrides, avec l’achèvement des tâches ou l’utilisation des commandes et des contrôles. Ce degré élevé de flexibilité s'accompagne d'une plus grande complexité, et les scénarios qui sont adaptés à des tâches bien définies à l’aide du langage naturel peuvent envisager d’utiliser [Custom Commands (préversion)](custom-commands.md) pour une expérience de solution simplifiée.

## <a name="getting-started-with-direct-line-speech"></a>Prise en main de Direct Line Speech

Les premières étapes de création d’un assistant vocal à l’aide de Direct Line Speech consiste à [obtenir une clé d’abonnement de reconnaissance vocale](overview.md#try-the-speech-service-for-free), créer un bot associé à cet abonnement et à connecter le bot au canal Direct Line Speech.

   ![Diagramme conceptuel du flux de service d’orchestration Direct Line Speech](media/voice-assistants/overview-directlinespeech.png "Flux du canal Speech")

Pour un guide complet et détaillé sur la création d’un assistant vocal simple utilisant Direct Line Speech, consultez le [didacticiel sur l’activation de la reconnaissance vocale de votre bot avec le Kit de développement logiciel (SDK) Speech et le canal Direct Line Speech](tutorial-voice-enable-your-bot-speech-sdk.md).

Nous proposons également des guides de démarrage rapide conçus pour vous permettre d’exécuter du code et d’apprendre les API rapidement. Ce tableau comprend une liste de guides de démarrage rapide pour les assistants vocaux organisés par langage et plateforme.

| Démarrage rapide | Plateforme | Informations de référence sur l'API |
|------------|----------|---------------|
| C#, UWP | Windows | [Parcourir](/dotnet/api/microsoft.cognitiveservices.speech) |
| Java | Windows, macOS, Linux | [Parcourir](/java/api/com.microsoft.cognitiveservices.speech) |
| Java | Android | [Parcourir](/java/api/com.microsoft.cognitiveservices.speech) |

## <a name="sample-code"></a>Exemple de code

Un exemple de code pour la création d’un assistant vocal est disponible sur GitHub. Ces exemples couvrent l’application cliente pour la connexion à votre assistant dans plusieurs langages de programmation populaires.

* [Exemples d’assistant vocal (Kit de développement logiciel [SDK])](https://aka.ms/csspeech/samples)
* [Tutoriel : Activer les fonctions vocales sur votre assistant avec le SDK Speech, C#](tutorial-voice-enable-your-bot-speech-sdk.md)

## <a name="customization"></a>Personnalisation

Les assistants vocaux créés à l’aide du service Speech peuvent utiliser la gamme complète des options de personnalisation disponibles pour la [reconnaissance vocale](speech-to-text.md), la [synthèse vocale](text-to-speech.md) et la [sélection de mot clé personnalisé](./custom-keyword-basics.md).

> [!NOTE]
> Les options de personnalisation varient selon la langue/paramètres régionaux (voir [Langues prises en charge](./language-support.md)).

Direct Line Speech et ses fonctionnalités pour les assistants vocaux sont un supplément idéal pour le [modèle Virtual Assistant Solution and Enterprise](/azure/bot-service/bot-builder-enterprise-template-overview). Bien que Direct Line Speech puisse fonctionner avec n’importe quel bot compatible, ces ressources fournissent une ligne de base réutilisable pour des expériences conversationnelles de haute qualité, ainsi que des compétences et des modèles sous-jacents communs pour un démarrage rapide.

## <a name="reference-docs"></a>Documents de référence

* [Kit de développement logiciel (SDK) de reconnaissance vocale](./speech-sdk.md)
* [Service de robot Azure](/azure/bot-service/?view=azure-bot-service-4.0)

## <a name="next-steps"></a>Étapes suivantes

* [Obtenir gratuitement une clé d’abonnement au service Speech](overview.md#try-the-speech-service-for-free)
* [Obtenir le kit SDK Speech](speech-sdk.md)
* [Créer et déployer un bot de base](/azure/bot-service/bot-builder-tutorial-basic-deploy?view=azure-bot-service-4.0)
* [Obtenir le modèle Virtual Assistant Solution and Enterprise](https://github.com/Microsoft/AI)