---
title: Installer des conteneurs Speech
titleSuffix: Azure Cognitive Services
description: Détaille les options de configuration de graphique Helm en parapluie pour Speech.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 0257f3af44cc85d0a3656472db224ae5a7e19161
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002271"
---
### <a name="speech-umbrella-chart"></a>Speech (graphique en parapluie)

Les valeurs dans le graphique en parapluie de niveau supérieur remplacent les valeurs du sous-graphique. Par conséquent, toutes les valeurs locales personnalisées doivent être ajoutées ici.

|Paramètre|Description|Default|
| -- | -- | -- | -- |
| `speechToText.enabled` | Indique si la **reconnaissance vocale** est activée ou non. | `true` |
| `speechToText.verification.enabled` | Indique si la fonctionnalité `helm test` pour la **reconnaissance vocale** est activée ou non. | `true` |
| `speechToText.verification.image.registry` | Le référentiel d’images docker que `helm test` utilise pour tester le service de **reconnaissance vocale**. Helm crée un pod distinct au sein du cluster pour le test, et extrait l’image *test-use* à partir de ce registre. | `docker.io` |
| `speechToText.verification.image.repository` | Le référentiel d’images docker que `helm test` utilise pour tester le service de **reconnaissance vocale**. Le pod de test Helm utilise ce référentiel pour extraire l’image *test-use*. | `antsu/on-prem-client` |
| `speechToText.verification.image.tag` | La balise d’image docker utilisée avec `helm test` pour le service de **reconnaissance vocale**. Le pod de test Helm utilise cette balise pour extraire l’image *test-use*. | `latest` |
| `speechToText.verification.image.pullByHash` | Indique si l’image docker *test-use* est extraite par hachage. Si `true`, `speechToText.verification.image.hash` doit être ajouté, avec une valeur de hachage d’image valide. | `false` |
| `speechToText.verification.image.arguments` | Les arguments utilisés pour exécuter l’image docker *test-use*. Le pod de test Helm passe ces arguments au conteneur lors de l’exécution de `helm test`. | `"./speech-to-text-client"`<br/> `"./audio/whatstheweatherlike.wav"` <br/> `"--expect=What's the weather like"`<br/>`"--host=$(SPEECH_TO_TEXT_HOST)"`<br/>`"--port=$(SPEECH_TO_TEXT_PORT)"` |
| `textToSpeech.enabled` | Indique si le service de **synthèse vocale** est activé ou non. | `true` |
| `textToSpeech.verification.enabled` | Indique si la fonctionnalité `helm test` pour la **reconnaissance vocale** est activée ou non. | `true` |
| `textToSpeech.verification.image.registry` | Le référentiel d’images docker que `helm test` utilise pour tester le service de **reconnaissance vocale**. Helm crée un pod distinct au sein du cluster pour le test, et extrait l’image *test-use* à partir de ce registre. | `docker.io` |
| `textToSpeech.verification.image.repository` | Le référentiel d’images docker que `helm test` utilise pour tester le service de **reconnaissance vocale**. Le pod de test Helm utilise ce référentiel pour extraire l’image *test-use*. | `antsu/on-prem-client` |
| `textToSpeech.verification.image.tag` | La balise d’image docker utilisée avec `helm test` pour le service de **reconnaissance vocale**. Le pod de test Helm utilise cette balise pour extraire l’image *test-use*. | `latest` |
| `textToSpeech.verification.image.pullByHash` | Indique si l’image docker *test-use* est extraite par hachage. Si `true`, `textToSpeech.verification.image.hash` doit être ajouté, avec une valeur de hachage d’image valide. | `false` |
| `textToSpeech.verification.image.arguments` | Les arguments à exécuter avec l’image docker *test-use*. Le pod de test Helm passe ces arguments au conteneur lors de l’exécution de `helm test`. | `"./text-to-speech-client"`<br/> `"--input='What's the weather like'"` <br/> `"--host=$(TEXT_TO_SPEECH_HOST)"`<br/>`"--port=$(TEXT_TO_SPEECH_PORT)"` |