---
author: PatrickFarley
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 06/12/2019
ms.author: pafarley
ms.openlocfilehash: f0761847c3677b324ef16c5987eb9a1561dbcbe0
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "75379287"
---
Examinez les valeurs `"confidence"` pour chaque résultat de clé/valeur sous le nœud `"pageResults"`. Vous devez également examiner les scores de confiance dans le nœud `"readResults"`, qui correspondent à l’opération de lecture de texte. La confiance des résultats de lecture n’affecte pas la confiance des résultats de l’extraction de clé/valeur. Vous devez donc vérifier les deux.
* Si les scores de confiance pour l’opération de lecture sont faibles, essayez d’améliorer la qualité de vos documents d’entrée (voir [Critères des entrées](../overview.md#input-requirements)).
* Si les scores de confiance pour l’opération d’extraction de clé/valeur sont faibles, assurez-vous que les documents en cours d’analyse sont du même type que les documents utilisés dans le jeu d’entraînement. Si les documents du jeu d’entraînement présentent des variantes d’apparence, envisagez de les diviser en différents dossiers et d’entraîner un modèle pour chaque variante.

Les scores de confiance que vous ciblez dépendent de votre cas d’usage, mais il est généralement recommandé de cibler un score de 80 % ou plus. Pour les cas plus sensibles, tels que la lecture de dossiers médicaux ou de relevés de facturation, un score de 100 % est recommandé.