---
title: Assistants vocaux Windows - Présentation
titleSuffix: Azure Cognitive Services
description: Présentation des assistants vocaux Windows, ainsi que des fonctionnalités et ressources de développement disponibles.
services: cognitive-services
author: cfogg6
manager: trrwilson
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: travisw
ms.openlocfilehash: 9e8b009ecc2181edfaad5da3d8d05ad0c1909051
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024753"
---
# <a name="voice-assistants-on-windows"></a>Assistants vocaux Windows

Sous Windows 10 version 2004 et versions ultérieures, les applications d'assistant vocal peuvent tirer parti des API Windows ConversationalAgent pour bénéficier d'une expérience d'assistant vocal complète.

## <a name="voice-assistant-features"></a>Fonctionnalités d'assistant vocal

Les applications d'agent vocal peuvent être activées en prononçant un mot clé afin d'offrir une expérience mains libres pilotée par la voix. L'activation vocale fonctionne également lorsque l'application est fermée et que l'écran est verrouillé.

En outre, Windows fournit un ensemble de paramètres de confidentialité qui permettent aux utilisateurs de contrôler l'activation vocale et l'activation par-dessus le verrouillage pour chaque application.

Après l'activation vocale, Windows peut gérer plusieurs agents actifs, et il avertit chaque assistant vocal lorsque ceux-ci sont interrompus ou désactivés. Cela permet aux applications de gérer correctement les interruptions et autres événements inter-agents.

## <a name="how-does-voice-activation-work"></a>Comment fonctionne l'activation vocale ?

Le runtime AAR (Agent Activation Runtime) est un processus Windows qui gère l'activation des applications via un mot clé prononcé oralement ou une pression sur un bouton. Il démarre avec Windows à condition qu'au moins une application du système soit inscrite. Les applications interagissent avec le runtime AAR par le biais des API ConversationalAgent du SDK Windows.

Lorsque l'utilisateur prononce un mot clé, le détecteur de mots clés logiciel ou matériel du système notifie le runtime AAR qu'un mot clé a été détecté, en fournissant un ID de mot clé. Le runtime AAR envoie à son tour une requête au service d'arrière-plan (BackgroundService) pour lancer l'application avec l'ID correspondant.

### <a name="registration"></a>Inscription

La première fois qu'une application activée par la voix est exécutée, elle inscrit son ID d'application et les informations relatives au mot clé via les API ConversationalAgent. Le runtime AAR inscrit toutes les configurations du mappage global auprès du détecteur de mots clés matériel ou logiciel du système, ce qui permet de détecter le mot clé de l'application. L'application [s'inscrit également auprès du service d'arrière-plan](/windows/uwp/launch-resume/register-a-background-task).

Cela signifie qu'une application ne peut pas être activée par la voix tant qu'elle n'a pas été exécutée une première fois et que l'inscription n'a pas été autorisée.

### <a name="receiving-an-activation"></a>Recevoir une activation

Dès réception de la requête du runtime AAR, le service d'arrière-plan lance l'application. L'application reçoit un signal via la méthode de cycle de vie OnBackgroundActivated de `App.xaml.cs` avec un argument d'événement unique. Cet argument indique à l'application qu'elle a été activée par le runtime AAR et qu'elle doit lancer la vérification du mot clé.

Si la vérification du mot clé est un succès, l'application peut effectuer une requête pour être affichée au premier plan. Lorsque cette requête aboutit, l'application affiche l'interface utilisateur et poursuit son interaction avec l'utilisateur.

Le runtime AAR signale toujours les applications actives lorsque leur mot clé est prononcé. Mais plutôt que d'effectuer ce signalement via la méthode de cycle de vie de `App.xaml.cs`, il l'effectue via un événement des API ConversationalAgent.

### <a name="keyword-verification"></a>Vérification du mot clé

Le détecteur de mots clés qui déclenche le lancement de l'application consomme désormais peu d'énergie en simplifiant le modèle de mot clé. Cela permet au détecteur de mots clés d'être « toujours activé » (Always-on) sans impact important sur la puissance, mais cela peut aussi se traduire par un nombre élevé de « fausses acceptations », avec des détections de mots clés qui n'ont pas été prononcés. C'est la raison pour laquelle le système d'activation vocale lance l'application en arrière-plan : pour permettre à l'application de vérifier que le mot clé a été prononcé avant d'interrompre la session en cours de l'utilisateur. Le runtime AAR enregistre l'audio depuis quelques secondes lorsque le mot clé est détecté. Celui-est est alors transmis à l'application. L'application peut l'utiliser pour exécuter un détecteur de mots clés plus fiable sur le même audio.

## <a name="next-steps"></a>Étapes suivantes

- **Passez en revue le guide de conception :** Notre [guide de conception](windows-voice-assistants-best-practices.md) présente les principales tâches requises afin de fournir les meilleures expériences possibles pour l'activation vocale sous Windows 10.
- **Consultez la page Prise en main :** Démarrez [ici](how-to-windows-voice-assistants-get-started.md) afin de connaître les étapes à suivre pour implémenter des assistants vocaux sous Windows, de la configuration de votre environnement de développement à l'introduction du guide d'implémentation.
- **Essayez l'exemple d'application** : Pour vous familiariser avec ces fonctionnalités, consultez la page [Exemple d'assistant vocal UWP](windows-voice-assistants-faq.md#the-uwp-voice-assistant-sample) et suivez les étapes décrites pour exécuter l'exemple de client.