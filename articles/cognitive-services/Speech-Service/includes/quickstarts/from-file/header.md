---
title: 'Démarrage rapide : Reconnaître la voix à partir d’un fichier audio – Service Speech'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 01/14/2020
ms.author: trbye
ms.openlocfilehash: a7c91775411f100a92dfcb0a7199dd4b25f6eca4
ms.sourcegitcommit: 34a6fa5fc66b1cfdfbf8178ef5cdb151c97c721c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/28/2020
ms.locfileid: "81400523"
---
Dans ce guide de démarrage rapide, vous allez utiliser le [kit de développement logiciel (SDK) Speech](~/articles/cognitive-services/speech-service/speech-sdk.md) pour reconnaître la voix à partir d’un fichier audio. Une fois les prérequis remplis, la reconnaissance vocale à partir d’un fichier se fait en quelques étapes seulement :
> [!div class="checklist"]
> * Créez un objet `SpeechConfig` à partir de votre clé d’abonnement et de votre région.
> * Créez un objet `AudioConfig` qui spécifie le nom du fichier WAV.
> * Créez un objet `SpeechRecognizer` à l’aide des objets `SpeechConfig` et `AudioConfig` ci-dessus.
> * À l’aide de l’objet `SpeechRecognizer`, démarrez le processus de reconnaissance pour un seul énoncé.
> * Inspectez le `SpeechRecognitionResult` retourné.
