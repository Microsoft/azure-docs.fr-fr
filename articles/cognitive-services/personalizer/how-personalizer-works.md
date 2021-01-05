---
title: Fonctionnement de Personalizer - Personalizer
description: La _boucle_ Personalizer utilise le Machine Learning pour générer le modèle qui prédit la première action pour votre contenu. L’apprentissage du modèle est effectué exclusivement sur les données que vous lui avez envoyées avec les appels de classement et de récompense.
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 02/18/2020
ms.openlocfilehash: cfbe5cf8c19bfafb38f6149391e09350785ebf9c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91303605"
---
# <a name="how-personalizer-works"></a>Fonctionnement de Personalizer

La ressource Personalizer, votre _boucle d’apprentissage_, utilise le Machine Learning pour générer le modèle qui prédit la première action pour votre contenu. L’apprentissage du modèle est effectué exclusivement sur les données que vous lui avez envoyées avec les appels de **classement** et de **récompense**. Chaque boucle est complètement indépendante des autres.

## <a name="rank-and-reward-apis-impact-the-model"></a>Les API de classement et de récompense ont un impact sur le modèle

Vous envoyez des _actions avec caractéristiques_ et des _caractéristiques de contexte_ à l’API de classement. L’API de **classement** décide d’utiliser :

* _Exploiter_ : Le modèle actuel pour déterminer la meilleure action en fonction des données antérieures.
* _Explorer_ : Sélectionne une action différente de l’action classée en premier. Vous [configurez ce pourcentage](how-to-settings.md#configure-exploration-to-allow-the-learning-loop-to-adapt) pour votre ressource Personalizer dans le Portail Azure.

Vous déterminez le score de récompense et envoyez ce score à l’API de récompense. L’API de **récompense** :

* Collecte des données pour entraîner le modèle en enregistrant les caractéristiques et les scores de récompense de chaque appel du classement.
* Utilise ces données pour mettre à jour le modèle en fonction de la configuration spécifiée dans la _stratégie d’apprentissage_.

## <a name="your-system-calling-personalizer"></a>Appel de Personalizer par votre système

L’illustration suivante montre l’architecture du flux des appels des API de classement et de récompense :

![texte de remplacement](./media/how-personalizer-works/personalization-how-it-works.png "Fonctionnement de la personnalisation")

1. Vous envoyez des _actions avec caractéristiques_ et des _caractéristiques de contexte_ à l’API de classement.

    * Personalizer décide d’exploiter le modèle actuel ou d’explorer de nouveaux choix pour le modèle.
    * Le résultat du classement est envoyé à EventHub.
1. Le meilleur résultat est retourné à votre système comme _ID d’action récompensée_.
    Votre système présente ce contenu et détermine un score de récompense en fonction de vos propres règles d’entreprise.
1. Votre système renvoie le score de récompense à la boucle d’apprentissage.
    * Quand Personalizer reçoit la récompense, elle est envoyée à EventHub.
    * Le classement et la récompense sont mis en corrélation.
    * Le modèle d’intelligence artificielle est mis à jour en fonction des résultats de la corrélation.
    * Le moteur d’inférence est mis à jour avec le nouveau modèle.

## <a name="personalizer-retrains-your-model"></a>Personalizer effectue à nouveau l’apprentissage de votre modèle

Personalizer effectue à nouveau l’apprentissage de votre modèle en fonction du paramètre **Fréquence de mise à jour du modèle** sur votre ressource Personalizer dans le Portail Azure.

Personalizer utilise toutes les données actuellement conservées, en fonction du paramètre **Conservation des données** en nombre de jours sur votre ressource Personalizer dans le Portail Azure.

## <a name="research-behind-personalizer"></a>Recherches derrière Personalizer

Personalizer est basé sur la science et la recherche de pointe dans le domaine de l’[apprentissage par renforcement](concepts-reinforcement-learning.md), notamment sur des articles, des activités de recherche et des domaines en cours d’exploration de Microsoft Research.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [principaux scénarios](where-can-you-use-personalizer.md) pour Personalizer