---
title: Tester votre application LUIS
titleSuffix: Azure Cognitive Services
description: Le test est le processus consistant à fournir des exemples d’énoncés à LUIS et obtenir une réponse des intentions et des entités reconnues par LUIS.
manager: nitinme
services: cognitive-services
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 10/10/2019
ms.openlocfilehash: ef63bf68fec592b768986a1329663508443c514b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91535846"
---
# <a name="testing-example-utterances-in-luis"></a>Test d’exemples d’énoncés dans LUIS

Le test est le processus consistant à fournir des exemples d’énoncés à LUIS et obtenir une réponse des intentions et des entités reconnues par LUIS. 

Vous pouvez tester LUIS de manière interactive, un énoncé à la fois, ou fournir un ensemble d'énoncés. Pendant le test, vous pouvez comparer la réponse de prédiction du modèle actif actuel à la réponse de prédiction du modèle publié. 

<a name="A-test-score"></a>
<a name="Score-all-intents"></a>
<a name="E-(exponent)-notation"></a>

## <a name="what-is-a-score-in-testing"></a>Qu’est-ce qu’un score dans le test ?
Consultez les concepts du [Score de prédiction](luis-concept-prediction-score.md) pour en savoir plus sur les scores de prédiction.

## <a name="interactive-testing"></a>Test interactif
Le test interactif s’effectue à partir du panneau **Test** du portail LUIS. Vous pouvez entrer un énoncé pour voir comment les entités et les intentions sont identifiées et évaluées. Si LUIS ne prévoit pas les intentions et les entités comme prévu sur un énoncé dans le panneau de test, copiez l’énoncé dans la page **Intention** en tant que nouvel énoncé. Puis, étiquetez les parties de cet énoncé pour les entités et effectuez l’apprentissage de LUIS. 

## <a name="batch-testing"></a>Test par lot
Consultez [Test par lot](luis-concept-batch-test.md) si vous testez plusieurs énoncés à la fois.

## <a name="endpoint-testing"></a>Test des points de terminaison
Vous pouvez tester en utilisant le [point de terminaison](luis-glossary.md#endpoint) avec deux versions maximum de votre application. Avec la version principale ou live de votre application définie en tant que point de terminaison de **production**, ajoutez une deuxième version au point de terminaison **intermédiaire**. Cette approche vous offre trois versions d’un énoncé : le modèle actuel dans le volet Test du site web [LUIS](luis-reference-regions.md) et les deux versions sur les deux points de terminaison différents. 

Tous les tests sur votre point de terminaison sont inclus dans votre quota d’utilisation. 

## <a name="do-not-log-tests"></a>Ne pas enregistrer les tests
Si vous testez sur un point de terminaison et que vous ne souhaitez pas que l’énoncé soit enregistré, pensez à utiliser la configuration de la chaîne de requête `logging=false`.

## <a name="where-to-find-utterances"></a>Où trouver les énoncés
LUIS stocke tous les énoncés enregistrés dans le journal des requêtes, disponible au téléchargement sur le portail LUIS à la page **Applications**, ainsi que les [API de création](https://go.microsoft.com/fwlink/?linkid=2092087) LUIS. 

Les énoncés dont LUIS n’est pas sûr sont répertoriés dans la page **[Réviser les énoncés de point de terminaison](luis-how-to-review-endpoint-utterances.md)** du site web [LUIS](luis-reference-regions.md). 

## <a name="remember-to-train"></a>Penser à former
Pensez à [former](luis-how-to-train.md) LUIS après avoir apporté des modifications au modèle. Les modifications apportées à l’application LUIS ne sont pas visibles dans le test avant que l’application ait été formée. 

## <a name="best-practices"></a>Meilleures pratiques
Découvrir les [meilleures pratiques](luis-concept-best-practices.md).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur le [test](luis-interactive-test.md) de vos énoncés.
