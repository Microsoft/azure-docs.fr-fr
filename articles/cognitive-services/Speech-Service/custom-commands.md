---
title: Commandes personnalisées - Service Speech
titleSuffix: Azure Cognitive Services
description: Vue d’ensemble des fonctionnalités et des restrictions relatives à Commandes personnalisées, une solution permettant de créer des applications vocales.
services: cognitive-services
author: trrwilson
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: travisw
ms.openlocfilehash: 40736764cf9cec6f652e0147eb25d83f15c5bee2
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024940"
---
# <a name="what-is-custom-commands"></a>Présentation des commandes personnalisées

Les applications telles que les [assistants vocaux](voice-assistants.md) écoutent les utilisateurs et réagissent en prenant une mesure, souvent en parlant. Ils utilisent la [reconnaissance vocale](speech-to-text.md) pour transcrire la parole de l’utilisateur, puis agissent sur la compréhension du langage naturel du texte. Cette action comprend fréquemment une parole de l’assistant générée à l’aide de la [synthèse vocale](text-to-speech.md). Les appareils se connectent aux assistants avec l’objet `DialogServiceConnector` du kit de développement logiciel (SDK) Speech.

Les **commandes personnalisées** facilitent la création d’applications de commandes vocales complètes, optimisées pour les expériences d’interaction de type « voice-first ». Elles offrent une expérience de création unifiée, un modèle d’hébergement automatique et une complexité relativement inférieure, vous permettant de vous concentrer sur la conception de la meilleure solution pour vos scénarios de commandes vocales.

Les commandes personnalisées sont particulièrement adaptées à la réalisation de tâches ou aux scénarios de commande et de contrôle, en particulier pour les appareils Internet des objets (IoT) ambiants et sans écran. Les exemples incluent des solutions pour les secteurs de l’hôtellerie, de la vente au détail et de l’automobile, qui vous permettent de créer des expériences de commande vocale optimales pour recevoir vos clients, gérer l’inventaire de votre commerce, et contrôler les fonctionnalités d’un véhicule lors de vos déplacements.

> [!TIP]
> Consultez nos exemples de démonstration sur notre page d’accueil à l’adresse [https://speech.microsoft.com/customcommands](https://speech.microsoft.com/customcommands).

Si vous souhaitez créer des applications de conversation complexes, nous vous recommandons de tester Bot Framework en utilisant la [solution d’assistant virtuel](/azure/bot-service/bot-builder-enterprise-template-overview). Vous pouvez ajouter une voix à n’importe quel Bot Framework avec Direct Line Speech.

Les outils adaptés à la solution Commandes personnalisées disposent d’un vocabulaire fixe avec des ensembles de variables bien définis. Par exemple, les tâches d’automatisation domestiques, telles que le contrôle d’un thermostat, sont idéales.

   ![Exemples de scénarios de réalisation de tâches](media/voice-assistants/task-completion-examples.png "Exemple de réalisation de tâches")

## <a name="getting-started-with-custom-commands"></a>Prise en main de la solution Commandes personnalisées

Notre objectif avec les commandes personnalisées est de réduire votre charge cognitive pour découvrir toutes les différentes technologies et vous concentrer sur la création de votre application de commande vocale. Première étape de l’utilisation des commandes personnalisées pour <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices" target="_blank">créer une ressource Azure Speech <span class="docon docon-navigate-external x-hidden-focus"></span></a>. Vous pouvez créer une application Commandes personnalisées sur Speech Studio et la publier. Une fois cette opération effectuée, une application se trouvant sur un appareil peut communiquer avec elle à l’aide du kit de développement logiciel (SDK) Speech.

#### <a name="authoring-flow-for-custom-commands"></a>Création d’un flux pour Commandes personnalisées
   ![Création d’un flux pour Commandes personnalisées](media/voice-assistants/custom-commands-flow.png "Flux de création de la solution Commandes personnalisées")

Consultez notre guide de démarrage rapide qui vous aidera à exécuter en moins de 10 minutes un code pour votre première application Commandes personnalisées.

* [Créer un assistant vocal à l’aide de commandes personnalisées](quickstart-custom-commands-application.md)

Après avoir terminé le guide de démarrage rapide, consultez nos guides pratiques pour obtenir des instructions détaillées sur la conception, le développement, le débogage, le déploiement et l’intégration d’une application Commandes personnalisées.

## <a name="building-voice-assistants-with-custom-commands"></a>Création d’assistants vocaux avec des commandes personnalisées
> [!VIDEO https://www.youtube.com/embed/1zr0umHGFyc]

## <a name="next-steps"></a>Étapes suivantes

* [Obtenir gratuitement une clé d’abonnement au service Speech](overview.md#try-the-speech-service-for-free)
* [Consulter les exemples de notre référentiel d’assistants vocaux sur GitHub](https://aka.ms/speech/cc-samples)
* [Accéder à Speech Studio pour tester les commandes personnalisées](https://speech.microsoft.com/customcommands)
* [Obtenir le kit SDK Speech](speech-sdk.md)