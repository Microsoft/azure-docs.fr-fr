---
title: 'Démarrage rapide : Créer un assistant vocal personnalisé - Service Speech'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: trrwilson
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 03/20/2020
ms.author: travisw
ms.openlocfilehash: bb88ef7f048b2454f05ef33286e59277070c3ea0
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96187572"
---
Dans ce guide de démarrage rapide, vous allez utiliser le [SDK Speech](~/articles/cognitive-services/speech-service/speech-sdk.md) pour créer une application d’assistant vocal personnalisé qui se connecte à un bot que vous avez déjà créé et configuré. Si vous avez besoin de créer un bot, consultez [le tutoriel associé](~/articles/cognitive-services/speech-service/tutorial-voice-enable-your-bot-speech-sdk.md) pour obtenir un guide plus complet.

Une fois les quelques prérequis satisfaits, la connexion de votre assistant vocal personnalisé ne nécessite que quelques étapes :
> [!div class="checklist"]
> * Créez un objet `BotFrameworkConfig` à partir de votre clé d’abonnement et de votre région.
> * Créez un objet `DialogServiceConnector` à l’aide de l’objet `BotFrameworkConfig` ci-dessus.
> * À l’aide de l’objet `DialogServiceConnector`, démarrez le processus d’écoute pour un énoncé unique.
> * Inspectez le `ActivityReceivedEventArgs` retourné.
