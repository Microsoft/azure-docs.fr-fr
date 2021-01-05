---
title: Configurer le comportement d'apprentissage
description: Le mode Apprenti vous permet de vous appuyer sur le service Personalizer et ses fonctionnalités d'apprentissage automatique. Il fournit également des mesures indiquant que le service reçoit des informations dont il peut tirer parti dans le cadre de son apprentissage - sans mettre en péril le trafic en ligne.
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: how-to
ms.date: 05/01/2020
ms.openlocfilehash: 57a03b107678f83200b11f408784f6455cbceffd
ms.sourcegitcommit: 1d6ec4b6f60b7d9759269ce55b00c5ac5fb57d32
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94579289"
---
# <a name="configure-the-personalizer-learning-behavior"></a>Configurer le comportement d'apprentissage de Personalizer

Le [mode Apprenti](concept-apprentice-mode.md) vous permet de vous appuyer sur le service Personalizer et ses fonctionnalités d'apprentissage automatique. Il fournit également l'assurance que le service reçoit des informations dont il peut tirer parti dans le cadre de son apprentissage - sans mettre en péril le trafic en ligne.

[!INCLUDE [Important Blue Box - Apprentice mode pricing tier](./includes/important-apprentice-mode.md)]

## <a name="configure-apprentice-mode"></a>Configurer le mode Apprenti

1. Connectez-vous au [portail Azure](https://portal.azure.com) pour accéder à votre ressource Personalizer.

1. Sur la page **Configuration**, accédez à l'onglet **Comportement d'apprentissage**, sélectionnez **Retourner l'action de référence, apprendre en tant qu'apprenti**, puis sélectionnez **Enregistrer**.

> [!div class="mx-imgBorder"]
> ![Capture d'écran de la configuration du comportement d'apprentissage en mode Apprenti sur le portail Azure](media/settings/configure-learning-behavior-azure-portal.png)

## <a name="changes-to-the-existing-application"></a>Modifications apportées à l'application existante

L'application existante ne doit pas modifier la façon dont elle sélectionne les actions à afficher ni la façon dont elle détermine la valeur, la **récompense** de l'action. La seule modification apportée à l'application peut être l'ordre des actions envoyées à l'API Rank de Personalizer. L'action affichée par votre application est envoyée en tant que _première action_ dans la liste des actions. L'API [Rank](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/Rank) utilise cette première action pour effectuer l'apprentissage de votre modèle Personalizer.

### <a name="configure-your-application-to-call-the-rank-api"></a>Configurer votre application pour appeler l'API Rank

Pour ajouter Personalizer à votre application, vous devez appeler les API Rank et Reward.

1. Ajoutez l'appel de l'[API ​​Rank](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/Rank) après le point dans la logique d'application où vous déterminez la liste des actions et leurs caractéristiques. La première action de la liste des actions doit être l'action sélectionnée par la logique existante.

1. Configurez votre code pour afficher l'action associée à l'**ID de l'action récompensée** de la réponse de l'API Rank.

### <a name="configure-your-application-to-call-reward-api"></a>Configurer votre application pour appeler l'API Reward

1. Utilisez la logique métier existante pour calculer la **récompense** de l'action affichée. La valeur doit être comprise entre 0 et 1. Envoyez cette récompense à Personalizer à l'aide de l'[API Reward](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/Reward). La valeur de la récompense n'est pas attendue immédiatement et elle peut être retardée - en fonction de votre logique métier.

1. Si vous ne renvoyez pas la récompense avant la fin du **délai d'attente de récompense** configuré, la récompense par défaut est utilisée.

## <a name="evaluate-apprentice-mode"></a>Évaluer le mode d'apprentissage

Sur le portail Azure, accédez à la page **Évaluations** de votre ressource Personalizer et examinez les **performances du comportement d'apprentissage**.

> [!div class="mx-imgBorder"]
> ![Capture d'écran de l'examen de l'évaluation du comportement d'apprentissage en mode Apprenti sur le portail Azure](media/settings/evaluate-apprentice-mode.png)

Le mode Apprenti fournit les **mesures d'évaluation** suivantes :
* **Référence - récompense moyenne** :  Récompenses moyennes de l'action par défaut (référence) de l'application.
* **Personalizer - récompense moyenne** : Moyenne des récompenses totales que Personalizer aurait pu atteindre.
* **Taux de récompense sur les 1 000 événements les plus récents** : Rapport récompense de référence/récompense Personalizer - normalisé sur les 1 000 événements les plus récents.

## <a name="switch-behavior-to-online-mode"></a>Passer en mode En ligne

Lorsque vous déterminez que l'apprentissage de Personalizer a atteint une moyenne mobile de 75 à 85 %, le modèle est prêt à passer en mode En ligne.

Sur le portail Azure, accédez à votre ressource Personalizer. Sur la page **Configuration**, accédez à l'onglet **Comportement d'apprentissage**, puis sélectionnez **Retourner la meilleure action** et **Enregistrer**.

Il n'est pas nécessaire de modifier les appels des API Rank et Reward.

## <a name="next-steps"></a>Étapes suivantes

* [Gérer les paramètres de modèle et d'apprentissage](how-to-manage-model.md)
