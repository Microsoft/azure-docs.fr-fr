---
author: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 11/09/2018
ms.author: nitinme
ms.openlocfilehash: 1d8340054ace25a0cdc36eef9c3d5b4238a6f99b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177347"
---
Le service et votre type d’abonnement déterminent le nombre de requêtes par seconde (RPS) que vous pouvez formuler. Assurez-vous que votre application inclut la logique permettant de respecter votre quota. Si la limite RPS est atteinte ou dépassée, la requête échoue et un code d’état HTTP 429 est renvoyé. La réponse inclut l’en-tête `Retry-After`, qui indique le délai d’attente que vous devez respecter avant d’envoyer une autre requête.

## <a name="denial-of-service-versus-throttling"></a>Déni de service (DOS) et limitation de requêtes

Le service établit une distinction entre une attaque par déni de service et une violation du nombre RPS. Si le service suspecte une attaque par déni de service, la requête réussit (le code d’état HTTP est 200 OK). Toutefois, le corps de la réponse est vide.