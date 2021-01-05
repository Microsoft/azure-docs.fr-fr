---
title: Assistants vocaux sur Windows - Prise en main
titleSuffix: Azure Cognitive Services
description: Étapes à suivre pour commencer à développer un agent vocal sur Windows, y compris une référence au guide de démarrage rapide de l’exemple de code.
services: cognitive-services
author: cfogg6
manager: trrwilson
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: travisw
ms.openlocfilehash: 77d756bc0c7bbdfa38bb1262638ef85b051fc234
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026486"
---
# <a name="getting-started-with-voice-assistants-on-windows"></a>Prise en main des assistants vocaux sur Windows

Ce guide détaille les étapes à suivre pour commencer à développer un assistant vocal sur Windows.

## <a name="set-up-your-development-environment"></a>Configurer l''environnement de développement

Pour commencer à développer un assistant vocal pour Windows, vous devez vous assurer que vous disposez de l’environnement de développement qui convient.

- **Visual Studio :** Vous devrez installer [Microsoft Visual Studio 2017](https://visualstudio.microsoft.com/), Community Edition ou une version ultérieure
- **Version de Windows** : Un PC avec une build anneau Rapide de Windows Insider et la version Windows Insider du SDK Windows.  Le fonctionnement de cet exemple de code a été vérifié sur la build Windows Insider 19025.vb_release_analog.191112-1600 avec le SDK Windows 19018.  Les builds ou kits de développement logiciel (SDK) ultérieurs aux versions vérifiées doivent être compatibles.
- **Outils de développement UWP** : Charge de travail de développement pour la plateforme Windows universelle dans Visual Studio.  Consultez la page UWP [Configurer](/windows/uwp/get-started/get-set-up) pour préparer votre machine au développement d’applications UWP.
- **Micro opérationnel et sortie audio**

## <a name="obtain-resources-from-microsoft"></a>Obtenir des ressources Microsoft

Certaines ressources requises pour un agent vocal entièrement personnalisé sur Windows nécessitent des ressources Microsoft. L’[exemple d'assistant vocal UWP](windows-voice-assistants-faq.md#the-uwp-voice-assistant-sample) fournit des exemples de versions de ces ressources à des fins de développement et de test initiaux et dès lors, cette section n'est pas nécessaire à des fins de développement initial.

- **Modèle de mot clé :** L’activation vocale requiert un modèle de mot clé Microsoft sous forme de fichier .bin. Le fichier .bin fourni dans l’exemple d'assistant vocal UWP est formé sur le mot clé « contoso ».
- **Jeton de fonctionnalité à accès limité :** Les API ConversationalAgent fournissant un accès à l’audio du micro, elles sont protégées dans le cadre des restrictions relatives aux fonctionnalités à accès limité.  Pour utiliser une fonctionnalité à accès limité, vous devez obtenir auprès de Microsoft un jeton de fonctionnalité à accès limité connecté à l’identité de package de votre application.

## <a name="establish-a-dialog-service"></a>Établir un service de dialogue

Pour une expérience d'assistant vocal complète, l’application doit disposer d'un service de dialogue permettant ce qui suit :

- Détecter un mot clé dans un fichier audio donné
- Écouter l’entrée de l’utilisateur et la convertir en texte
- Fournir le texte à un bot
- Traduire la réponse textuelle du bot en sortie audio

Voici la configuration requise pour créer un service de dialogue de base à l’aide de Direct Line Speech.

- **Abonnement Speech Services :** Abonnement à Cognitive Speech Services pour les conversions de synthèse vocale. Essayez gratuitement Speech Services [ici](./overview.md#try-the-speech-service-for-free).
- **Bot Framework :**  Un bot créé à l’aide de Bot Framework version 4.2 ou ultérieure abonné à [Direct Line Speech](./direct-line-speech.md) pour activer l’entrée et la sortie vocales. [Ce guide](./tutorial-voice-enable-your-bot-speech-sdk.md) contient des instructions pas à pas permettant de créer un « bot d’écho » et de s’abonner à Direct Line Speech. Vous pouvez également consulter les étapes permettant de créer un bot personnalisé [ici](https://blog.botframework.com/2018/05/07/build-a-microsoft-bot-framework-bot-with-the-bot-builder-sdk-v4/), puis suivre les étapes disponibles [ici](./tutorial-voice-enable-your-bot-speech-sdk.md) pour l'abonner à Direct Line Speech, mais avec votre nouveau bot plutôt que le « bot d'écho ».

## <a name="try-out-the-sample-app"></a>Essayer un exemple d'application

Avec votre clé d’abonnement Speech Services et l’ID du bot d'écho, vous êtes prêt à essayer l’exemple d'[assistant vocal UWP](windows-voice-assistants-faq.md#the-uwp-voice-assistant-sample). Suivez les instructions du fichier Lisez-moi pour exécuter l’application et entrez vos informations d’identification.

## <a name="create-your-own-voice-assistant-for-windows"></a>Créer votre propre assistant vocal pour Windows

Après avoir reçu de Microsoft le jeton de fonctionnalité à accès limité et le fichier .bin, vous pouvez commencer votre propre assistant vocal sur Windows.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Lire le guide d’implémentation de l’assistant vocal](windows-voice-assistants-implementation-guide.md)