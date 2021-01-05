---
title: Scores de prédiction - LUIS
description: Un score de prédiction indique le degré de confiance que le service d’API LUIS a dans les résultats de prédiction, en fonction d’un énoncé utilisateur.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 04/14/2020
ms.openlocfilehash: d836273e61752ff208133466016ce7c6ff9c28fa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91316458"
---
# <a name="prediction-scores-indicate-prediction-accuracy-for-intent-and-entities"></a>Les scores de prédiction indiquent la précision des prédictions pour les entités et l’intention

Un score de prédiction indique le degré de confiance que LUIS a dans les résultats de prédiction d’un énoncé d’utilisateur.

Un score de prédiction est compris entre zéro (0) et un (1). Un exemple de score LUIS de grande confiance est égal à 0,99. Un exemple de score faible confiance est égal à 0,01.

|Valeur du score|Confiance|
|--|--|
|1|correspondance définie|
|0,99|grande confiance|
|0.01|faible confiance|
|0|échec définitif de correspondance|

## <a name="top-scoring-intent"></a>Intention de score principale

Chaque prédiction d’énoncé retourne une intention de score principale. Cette prédiction est une comparaison numérique des scores de prédiction.

## <a name="proximity-of-scores-to-each-other"></a>Proximité des scores entre eux

La différence entre les deux principaux scores peut être très faible. LUIS n’indique pas cette proximité autrement qu’en retournant le meilleur score.

## <a name="return-prediction-score-for-all-intents"></a>Retourner le score de prédiction pour toutes les intentions

Un résultat de test ou de point de terminaison peut inclure toutes les intentions. Cette configuration est définie sur le point de terminaison en utilisant la paire nom/valeur de chaîne de requête correcte.

|API de prédiction|Nom de chaîne de requête|
|--|--|
|V3|`show-all-intents=true`|
|V2|`verbose=true`|

## <a name="review-intents-with-similar-scores"></a>Passer en revue les intentions aux scores similaires

La révision du score pour toutes les intentions est une bonne manière de vérifier non seulement que l’intention correcte est identifiée, mais aussi que le score de l’intention identifiée suivante est sensiblement et systématiquement plus faible pour les énoncés.

Si plusieurs intentions ont des scores de prédiction proches, en se basant sur le contexte d’un énoncé, LUIS peut basculer d’une intention à l’autre. Pour corriger cette situation, continuez à ajouter des énoncés à chaque intention avec une plus grande variété de différences contextuelles ou vous pouvez faire en sorte que l’application cliente, par exemple un bot de conversation, effectue des choix programmatiques sur la façon de gérer les deux meilleures intentions.

Les deux intentions, qui ont des scores trop proches, peuvent s’inverser en raison d’une **formation non déterministe**. Le score le plus élevé peut devenir le second score le plus élevé, et le second score le plus élevé peut devenir le score le plus élevé. Pour éviter ce problème, ajoutez des énoncés de l’exemple à chacune des deux principales intentions de cet énoncé, avec un choix de mots et de contexte permettant de différencier les deux intentions. Les deux intentions doivent avoir le même nombre d’énoncés d’exemple. Une règle empirique de séparation visant à empêcher l’inversion en raison de la formation, constitue une différence de score de 15 %.

Vous pouvez désactiver la **formation non déterministe** en [effectuant l’apprentissage avec toutes les données](luis-how-to-train.md#train-with-all-data).

## <a name="differences-with-predictions-between-different-training-sessions"></a>Différences des prédictions entre des sessions de formation différentes

Lorsque vous effectuez l’apprentissage du même modèle dans une autre application et que les scores ne sont pas les mêmes, la différence est due à la **formation non déterministe** (élément de caractère aléatoire). En second lieu, un chevauchement d’énoncé sur plus d’une intention signifie que l’intention principale pour le même énoncé peut changer en fonction de l’apprentissage.

Si votre chatbot requiert un score LUIS spécifique pour indiquer la confiance dans un énoncé, vous devez utiliser la différence de score entre les deux intentions principales. Cette situation offre une flexibilité pour varier l’apprentissage.

Vous pouvez désactiver la **formation non déterministe** en [effectuant l’apprentissage avec toutes les données](luis-how-to-train.md#train-with-all-data).

## <a name="e-exponent-notation"></a>Notation E (exposant)

Les scores de prédiction peuvent utiliser la notation d’exposant, _apparaissant_ au-dessus de la plage 0-1, par exemple `9.910309E-07`. Ce score indique un très **petit** nombre.

|Score de notation E |Score réel|
|--|--|
|9.910309E-07|.0000009910309|

<a name="punctuation"></a>

## <a name="application-settings"></a>Paramètres de l’application

Utilisez des [paramètres d’application ](luis-reference-application-settings.md) pour contrôler la façon dont les signes diacritiques et la ponctuation impactent les scores de prédiction.

## <a name="next-steps"></a>Étapes suivantes

Consulter [Ajouter des entités](luis-how-to-add-entities.md) pour découvrir comment ajouter des entités à votre application LUIS.
