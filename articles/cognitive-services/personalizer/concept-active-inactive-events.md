---
title: Événements actifs et inactifs - Personalizer
description: Cet article traite de l’utilisation des événements actifs et inactifs au sein du service Personalizer.
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 02/20/2020
ms.openlocfilehash: 7d1044d02ceba1f3d0996b1fe1c8a9a44b31049b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91253647"
---
# <a name="active-and-inactive-events"></a>Événements actifs et inactifs

Un événement **actif** est un appel à un classement dans lequel vous savez que vous allez montrer le résultat au client et déterminer le score de récompense. Il s'agit du comportement par défaut.

Un événement **inactif** est un appel à un classement dans lequel vous ne savez pas si l’utilisateur verra jamais l’action recommandée, en raison de la logique métier. Cela vous permet d’ignorer l’événement afin que Personalizer ne soit pas entraîné avec la récompense par défaut. Les événements inactifs ne doivent pas appeler l’API de récompense.

Il est important que la boucle d’apprentissage connaisse le type réel d’événement. Un événement inactif n’aura pas d’appel de récompense. Un événement actif doit avoir un appel de récompense mais, si l’appel d’API n’est jamais effectué, le score de récompense par défaut est appliqué. Remplacez l’état inactif d’un événement par actif dès que vous savez qu’il aura une incidence sur l’expérience de l’utilisateur.

## <a name="typical-active-events-scenario"></a>Scénario d’événements actifs standard

Quand votre application appelle l’API de classement, vous recevez l’action que l’application doit afficher dans le champ **rewardActionId**.  À partir de ce moment, Personalizer attend un appel de récompense avec un score de récompense portant le même ID d’événement. Le score de récompense est utilisé afin d’entraîner le modèle pour les appels de classement ultérieurs. Si aucun appel de récompense n’est reçu pour l’ID d’événement, une récompense par défaut est appliquée. Les [récompenses par défaut](how-to-settings.md#configure-rewards-for-the-feedback-loop) sont définies sur votre ressource Personalizer dans le portail Azure.

## <a name="other-event-type-scenarios"></a>Autres scénarios de type d’événement

Dans certains scénarios, l’application peut avoir besoin de l’appel de classement avant de savoir si le résultat sera utilisé ou affiché pour l’utilisateur. Cela peut se produire dans les situations où, par exemple, le rendu de la page du contenu promu est remplacé par une campagne marketing. Si le résultat de l’appel de classement n’a jamais été utilisé et que l’utilisateur ne l’a jamais vu, n’envoyez pas d’appel de récompense correspondant.

En règle générale, ces scénarios se présentent dans les cas suivants :

* Vous prérendez l’interface utilisateur que l’utilisateur peut voir ou pas.
* Votre application effectue une personnalisation prédictive dans laquelle les appels d’API de classement sont effectués avec moins de contexte en temps réel, et leur application peut ou non utiliser la sortie.

Dans ce cas, utilisez Personalizer pour effectuer l’appel de classement, en demandant à ce que l’événement soit _inactif_. Personalizer n’attend pas de récompense pour cet événement, et n’applique pas non plus de récompense par défaut.

Plus tard dans votre logique métier, si l’application utilise les informations de l’appel de classement, _activez_ tout simplement l’événement. Une fois l’événement actif, Personalizer attend une récompense d’événement. Si aucun appel explicite n’est effectué à destination de l'API de récompense, Personalizer applique une récompense par défaut.

## <a name="inactive-events"></a>Événements inactifs

Pour désactiver la formation pour un événement, procédez à un appel de classement à l'aide de `learningEnabled = False`.

Pour un événement inactif, la formation est implicitement activée si vous envoyez une récompense pour l’eventId ou appelez l’API `activate` pour cet eventId.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez [comment déterminer le score de récompense et les données à prendre en compte](concept-rewards.md).
