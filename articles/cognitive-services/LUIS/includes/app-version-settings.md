---
title: Fichier include
ms.topic: include
ms.custom: include file
ms.date: 5/17/2020
ms.openlocfilehash: 79a45825be66b44d869ef741917436de87a1f5d0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83653207"
---
Découvrez les [concepts](../luis-concept-utterance.md#utterance-normalization-for-diacritics-and-punctuation) de la normalisation et comment utiliser les API [version](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/versions-update-application-version-settings) pour mettre à jour ces paramètres ou, dans le portail LUIS, utilisez la section **Manage** (Gérer) de la page **Settings** (Paramètres).


|Paramètre de l’interface utilisateur|Paramètre d’API|Information|
|--|--|--|
|Utiliser un apprentissage non déterministe|`UseAllTrainingData`|L’entraînement utilise un petit pourcentage d’échantillonnage négatif. Si vous souhaitez utiliser toutes les données au lieu du petit échantillonnage négatif, définissez la valeur `true`. |
|Normaliser les signes diacritiques|`NormalizeDiacritics`|La normalisation des signes diacritiques remplace les caractères possédant des signes diacritiques dans les énoncés par des caractères normaux. Ce paramètre est uniquement disponible dans les [langues](../luis-reference-application-settings.md#diacritics-normalization) qui prennent en charge les signes diacritiques.|
|Normaliser la ponctuation|`NormalizePunctuation`|La normalisation de la ponctuation signifie qu’avant l’apprentissage de vos modèles et avant la prédiction de vos requêtes de point de terminaison, la ponctuation est retirée des énoncés.|
|Normaliser les formulaires Word|`NormalizeWordForm`|Ignorez les formulaires Word au-delà de la racine.|
