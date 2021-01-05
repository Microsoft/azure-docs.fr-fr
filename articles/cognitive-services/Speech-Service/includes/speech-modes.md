---
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 01/22/2020
ms.author: trbye
ms.openlocfilehash: dc569050b78a5797808f2e2e000019ba516ba22e
ms.sourcegitcommit: 44844a49afe8ed824a6812346f5bad8bc5455030
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97739232"
---
**Interactive**
- Pour les scénarios de commande et de contrôle.
- A une valeur de délai d’expiration de segmentation de X.
- À la fin d’un énoncé reconnu, le service cesse de traiter l’audio de cet ID de demande et termine le tour. La connexion n'est pas fermée.
- La limite maximale de la reconnaissance est de 20 s.
- L’appel Carbon standard à invoquer est `RecognizeOnceAsync`.

**Conversation**
- Pour des reconnaissances plus longues.
- A une valeur de délai d’expiration de segmentation de Y. (Y != X)
- Traite plusieurs énoncés complets sans terminer le tour.
- Termine le tour en raison d’un silence trop long.
- Carbon continuera avec un nouvel ID de demande et une relecture de l’audio si nécessaire.
- Le service se déconnectera de force après 10 minutes de reconnaissance vocale.
- Carbon se reconnectera et relira l’audio non confirmé.
- Appelée dans Carbon avec `StartContinuousRecognition`.

**Dictée**
- Permet aux utilisateurs de spécifier la ponctuation oralement.
- Appelée dans Carbon en spécifiant `EnableDictation` sur l’objet `SpeechConfig` indépendamment de l’appel d’API qui démarre la reconnaissance.
- Le cluster interne <sup></sup>renvoie des messages `speech.fragment` pour les résultats intermédiaires, tandis que le cluster tiers <sup></sup>renvoie des messages `speech.hypothesis`.