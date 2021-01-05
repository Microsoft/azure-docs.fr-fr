---
title: Score de récompense - Personalizer
description: Le score de récompense indique à quel point le choix de personnalisation, RewardActionID, était adapté à l’utilisateur. La valeur du score de récompense est déterminée par votre logique métier, en fonction des observations sur le comportement utilisateur. Personalizer effectue l’apprentissage de ses modèles Machine Learning en évaluant les récompenses.
ms.service: cognitive-services
ms.subservice: personalizer
ms.date: 02/20/2020
ms.topic: conceptual
ms.openlocfilehash: f3249ba2089c3d9650aa46f665353ad392d0e773
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94365565"
---
# <a name="reward-scores-indicate-success-of-personalization"></a>Les scores de récompense indiquent la réussite de la personnalisation

Le score de récompense indique à quel point le choix de personnalisation, [RewardActionID](/rest/api/cognitiveservices/personalizer/rank/rank#response), était adapté à l’utilisateur. La valeur du score de récompense est déterminée par votre logique métier, en fonction des observations sur le comportement utilisateur.

Personalizer effectue l’apprentissage de ses modèles Machine Learning en évaluant les récompenses.

Découvrez [comment](how-to-settings.md#configure-rewards-for-the-feedback-loop) configurer le score de récompense par défaut dans le portail Azure pour votre ressource Personalizer.

## <a name="use-reward-api-to-send-reward-score-to-personalizer"></a>Utiliser l’API Reward (Récompense) pour envoyer un score de récompense à Personalizer

Les récompenses sont envoyées à Personalizer par l’[API Reward](/rest/api/cognitiveservices/personalizer/events/reward). En général, une récompense est un nombre compris entre 0 et 1. Une récompense négative (avec la valeur -1) est possible dans certains scénarios, et ne doit être utilisée que si vous êtes familiarisé avec l’apprentissage par renforcement. Personalizer effectue l’apprentissage du modèle afin d’obtenir la somme la plus élevée possible de récompenses au fil du temps.

Les récompenses sont envoyées une fois que le comportement de l’utilisateur a eu lieu, parfois plusieurs jours plus tard. La configuration du délai maximal après lequel Personalizer considère qu’un événement ne reçoit aucune récompense ou reçoit une récompense par défaut s’effectue avec le [Temps d’attente des récompenses](#reward-wait-time) dans le portail Azure.

Si le score de récompense pour un événement n’a pas été reçu dans le **Temps d’attente des récompenses** , la **récompense par défaut** est appliquée. En règle générale, la **[récompense par défaut](how-to-settings.md#configure-reward-settings-for-the-feedback-loop-based-on-use-case)** est configurée sur zéro.


## <a name="behaviors-and-data-to-consider-for-rewards"></a>Comportements et données à prendre en compte pour les récompenses

Prenez en compte ces signaux et comportements pour le contexte du score de récompense :

* L’entrée de l’utilisateur elle-même, avec des suggestions lorsque des options sont impliquées (« Voulez-vous dire X ? »).
* La longueur de la session.
* Le temps entre les sessions.
* L’analyse des sentiments des interactions de l’utilisateur.
* Des questions directes et mini-enquêtes pour lesquelles le bot demande à l’utilisateur d’envoyer des commentaires sur l’utilité, la précision.
* La réponse aux alertes, ou le retard de réponse aux alertes.

## <a name="composing-reward-scores"></a>Composition de scores de récompense

Un score de récompense doit être calculé dans votre logique métier. Le score peut être représenté comme suit :

* Un seul nombre envoyé une fois
* Un score envoyé immédiatement (par exemple, 0,8) et un score supplémentaire envoyé ultérieurement (généralement 0,2).

## <a name="default-rewards"></a>Récompenses par défaut

Si aucune récompense n’est reçue dans le [Temps d’attente des récompenses](#reward-wait-time), soit la durée qui s’est écoulée depuis l’appel Rank, Personalizer applique implicitement la **récompense par défaut** à cet événement Rank.

## <a name="building-up-rewards-with-multiple-factors"></a>Construction de récompenses avec plusieurs facteurs

Pour une personnalisation efficace, vous pouvez générer le score de récompense selon plusieurs facteurs.

Par exemple, vous pouvez appliquer les règles suivantes pour personnaliser une liste de contenu vidéo :

|Comportement de l’utilisateur|Valeur de score partielle|
|--|--|
|L’utilisateur a cliqué sur le premier élément.|Récompense +0,5|
|L’utilisateur a ouvert le contenu réel associé à cet élément.|Récompense +0,3|
|L’utilisateur a regardé 5 minutes du contenu ou 30 %, la durée la plus longue étant retenue.|Récompense +0,2|
|||

Vous pouvez ensuite envoyer la récompense totale à l’API.

## <a name="calling-the-reward-api-multiple-times"></a>Appeler l’API Reward plusieurs fois

Vous pouvez également appeler l’API Reward en utilisant le même ID d’événement, en envoyant différents scores de récompense. Lorsque Personalizer reçoit ces récompenses, il détermine la récompense finale pour cet événement en les agrégeant tel que spécifié dans la configuration Personalizer.

Valeurs d’agrégation :

*  **First**  : prend le premier score de récompense reçu pour l’événement et ignore le reste.
* **Sum**  : prend tous les scores de récompense collectés pour l’ID d’événement et les additionne.

Toutes les récompenses pour un événement, qui sont reçues une fois que le **Temps d’attente des récompenses** s’est écoulé, sont ignorées et n’affectent pas l’apprentissage des modèles.

En additionnant les scores de récompense, votre récompense finale peut se trouver en dehors de la plage prévue. Cela n’entraîne pas l’échec du service.

## <a name="best-practices-for-calculating-reward-score"></a>Meilleures pratiques pour le calcul du score de récompense

* **Prendre en compte les réels indicateurs d’une personnalisation réussie** : il est facile de raisonner en termes de clics, mais une bonne récompense repose sur ce que vous souhaitez que vos utilisateurs *réalisent* et non sur ce que vous souhaitez que les personnes *fassent*.  Par exemple, offrir des récompenses en cas de clics peut entraîner la sélection d’un contenu « piège à clics ».

* **Utiliser un score de récompense pour la qualité du fonctionnement de la personnalisation** : avec un peu de chance, la personnalisation d’une suggestion de film a pour effet que l’utilisateur regarde le film et lui accorde une note élevée. Étant donné que la classification des films dépend de nombreuses choses (la qualité de l’action, l’humeur de l’utilisateur), cela n’est pas un bon indicateur du bon fonctionnement de la *personnalisation*. Toutefois, si un utilisateur regarde les premières minutes du film, cela peut constituer un meilleur signal de l’efficacité de la personnalisation. Et l’envoi d’une récompense de 1 au bout de 5 minutes représentera un signal encore meilleur.

* **Les récompenses s’appliquent uniquement aux RewardActionID** : Personalizer applique les récompenses pour comprendre l’efficacité de l’action spécifiée dans RewardActionID. Si vous choisissez d’afficher les autres actions et que l’utilisateur clique dessus, la récompense doit être de zéro.

* **Envisager les conséquences inattendues** : créez des fonctions de récompense qui aboutissent à des résultats responsables avec une [déontologie et une utilisation responsable](ethics-responsible-use.md).

* **Utiliser les récompenses incrémentielles** : l’ajout de récompenses partielles pour les comportements utilisateur moins significatifs permet à Personalizer d’obtenir de meilleure récompenses. Cette récompense incrémentielle permet à l’algorithme de savoir qu’il est sur le point d’inciter l’utilisateur à avoir le comportement souhaité.
    * Si vous affichez une liste de films, si l’utilisateur pointe sur le premier film pendant un certain temps pour voir plus d’informations, vous pouvez déterminer que l’utilisateur a manifesté un certain engagement. Le comportement peut compter avec un score de récompense de 0,1.
    * Si l’utilisateur a ouvert la page puis l’a fermée, le score de récompense peut être 0,2.

## <a name="reward-wait-time"></a>Temps d’attente des récompenses

Personalizer met en corrélation les informations d’un appel Rank avec les récompenses envoyées dans des appels de récompense pour effectuer l’apprentissage du modèle. Ceux-ci peuvent se produire à des moments différents. Personalizer attend pendant une durée limitée, en commençant lors de l’appel Rank, même si ce dernier a été effectué en tant qu’événement inactif et a été activé ultérieurement.

Si le **Temps d’attente des récompenses** arrive à expiration et qu’aucune information de récompense n’a été générée, une récompense par défaut est appliquée à cet événement pour apprentissage. La durée d’attente maximale est de 6 jours.

## <a name="best-practices-for-reward-wait-time"></a>Meilleures pratiques pour le temps d’attente des récompenses

Suivez ces recommandations pour améliorer les résultats.

* Définissez un temps d’attente des récompenses aussi court que possible, tout en laissant suffisamment de temps pour obtenir des retours des utilisateurs.

* Ne choisissez pas une durée qui est plus courte que le temps nécessaire pour obtenir des commentaires. Par exemple, si certaines de vos récompenses ont lieu une fois qu’un utilisateur a visionné 1 minute d’une vidéo, la longueur de l’expérience doit être au moins le double.

## <a name="next-steps"></a>Étapes suivantes

* [Apprentissage par renforcement](concepts-reinforcement-learning.md)
* [Essayer l’API Rank](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/Rank/console)
* [Essayer l’API Reward](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/Reward)