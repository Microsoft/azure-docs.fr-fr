---
title: Assistants vocaux Windows - FAQ
titleSuffix: Azure Cognitive Services
description: Questions fréquemment posées lors du développement d'un agent vocal Windows.
services: cognitive-services
author: cfogg6
manager: trrwilson
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: travisw
ms.openlocfilehash: 3cb4dd16592f3af962b32fe3005d2ad4b241d831
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84457005"
---
# <a name="samples-and-faqs"></a>Exemples et FAQ

## <a name="the-uwp-voice-assistant-sample"></a>L'exemple d'assistant vocal UWP

L'exemple d'assistant vocal UWP est un assistant vocal Windows qui sert à :

- illustrer l'utilisation des API Windows ConversationalAgent ;
- présenter les meilleures pratiques en matière d'agent vocal Windows ;
- fournir une application adaptable et des composants réutilisables pour votre application d'agent MVP ;
- montrer comment le service Direct Line Speech, associé à Bot Framework ou Commandes personnalisées, peut être utilisé avec les API Windows ConversationalAgent pour une expérience d'agent vocal de bout en bout.

La documentation fournie avec l'exemple d'application présente le chemin du code d'activation vocale et de gestion des agents, des conditions préalables au démarrage jusqu'au nettoyage approprié.

> [!div class="nextstepaction"]
> [Reportez-vous au dépôt GitHub pour accéder à l’exemple UWP](https://aka.ms/MVA/sample)

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="how-do-i-contact-microsoft-for-resources-like-limited-access-feature-tokens-and-keyword-model-files"></a>Comment puis-je contacter Microsoft pour obtenir des ressources telles que des jetons de fonctionnalités à accès limité et des fichiers de modèle de mot clé ?

Pour accéder à ces ressources, contactez winvoiceassistants@microsoft.com.

### <a name="my-app-is-showing-in-a-small-window-when-i-activate-it-by-voice-how-can-i-transition-from-the-compact-view-to-a-full-application-window"></a>Lorsque j'active mon application par la voix, celle-ci s'affiche dans une petite fenêtre. Comment puis-je passer de l'affichage compact à une fenêtre d'application complète ?

La première fois que vous activez votre application par la voix, celle-ci se lance dans un affichage compact. Pour obtenir de l'aide sur les différents affichages et sur le passage de l'un à l'autre pour les assistants vocaux Windows, consultez les [Conseils de conception relatifs l'activation vocale (préversion)](windows-voice-assistants-best-practices.md#design-guidance-for-voice-activation-preview).

Pour passer de l'affichage compact à l'affichage complet de l'application, utilisez l'API appView `TryEnterViewModeAsync` :

`var appView = ApplicationView.GetForCurrentView();
 await appView.TryEnterViewModeAsync(ApplicationViewMode.Default);`

### <a name="why-are-voice-assistant-features-on-windows-only-enabled-for-uwp-applications"></a>Pourquoi les fonctionnalités d'assistant vocal Windows ne sont-elles activées que pour les applications UWP ?

Compte tenu des risques que font courir les fonctionnalités telles que l'activation vocale en matière de confidentialité, les fonctionnalités de la plateforme UWP sont nécessaires pour permettre aux fonctionnalités de l'assistant vocal Windows d'être suffisamment sécurisées.

### <a name="the-uwp-voice-assistant-sample-uses-direct-line-speech-do-i-have-to-use-direct-line-speech-for-my-voice-assistant-on-windows"></a>L'exemple d'assistant vocal UWP utilise Direct Line Speech. Dois-je utiliser Direct Line Speech pour mon assistant vocal Windows ?

L'exemple d'application UWP a été développé à l'aide de Direct Line Speech et du kit de développement logiciel Speech Services SDK pour illustrer l'utilisation d'un service de dialogue avec la capacité Agent conversationnel Windows. Cependant, vous pouvez utiliser n'importe quel service pour la vérification des mots clés locaux et cloud, la conversion de parole en texte, le dialogue avec les bots et la conversion de texte en parole.