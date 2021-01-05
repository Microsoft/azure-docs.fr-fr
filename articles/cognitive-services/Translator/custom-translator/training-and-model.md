---
title: Que sont un entraînement et un modèle ? - Custom Translator
titleSuffix: Azure Cognitive Services
description: 'Un modèle est le système qui fournit une traduction pour une paire de langues spécifique. Le résultat d’un entraînement réussi est un modèle. Lors de l’entraînement d’un modèle, trois jeux de données qui s’excluent mutuellement sont nécessaires : le jeu de données d’entraînement, le jeu de données de paramétrage et le jeu de données de test.'
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 08/17/2020
ms.author: swmachan
ms.openlocfilehash: 1765aaed3c0562da13c539845bfe19f6f85ed4ef
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94369016"
---
# <a name="what-are-trainings-and-models"></a>Que sont les entraînements et les modèles ?

Un modèle est le système qui fournit une traduction pour une paire de langues spécifique.
Le résultat d’un entraînement réussi est un modèle. Lors de l’entraînement d’un modèle, trois types de documents mutuellement exclusifs sont nécessaires : entraînement, optimisation et test. Un type de document de dictionnaire peut également être fourni. Consultez [Alignement de phrases](./sentence-alignment.md#suggested-minimum-number-of-sentences).

Si seules des données d’entraînement sont fournies lors de la mise en file d’attente d’un entraînement, Custom Translator assemble automatiquement les données d’optimisation et de test. Il utilise un sous-ensemble aléatoire de phrases de vos documents d’entraînement et exclut ces phrases des données d’entraînement elles-mêmes.

## <a name="training-document-type-for-custom-translator"></a>Type de document d’entraînement pour Custom Translator

Les documents inclus dans le jeu d’entraînement sont utilisés par Custom Translator comme base pour la création de votre modèle. Pendant l’exécution de l’entraînement, les phrases qui sont présentes dans ces documents sont alignées (ou appairées). Vous pouvez prendre la liberté de composer votre jeu de documents d’entraînement. Vous pouvez inclure des documents que vous considérez d’une pertinence indirecte dans un modèle. Là encore, vous devez les exclure d’un autre jeu pour voir l’impact sur le [score BLEU (Bilingual Evaluation Understudy)](what-is-bleu-score.md). Tant que les jeux de paramétrage et de test restent constants, n’hésitez pas à faire des essais avec la composition du jeu d’entraînement. Cette approche est un moyen efficace de modifier la qualité de votre système de traduction.

Vous pouvez exécuter plusieurs entraînements dans un projet et comparer les [scores BLEU](what-is-bleu-score.md) entre toutes les exécutions d’entraînements. Quand vous exécutez plusieurs entraînements pour une comparaison, vérifiez que les mêmes données de paramétrage/test sont spécifiées chaque fois. Veillez également à examiner les résultats manuellement sous l’onglet [Test](how-to-view-system-test-results.md).

## <a name="tuning-document-type-for-custom-translator"></a>Type de document d’optimisation pour Custom Translator

Les documents parallèles inclus dans ce jeu sont utilisés par Custom Translator afin de régler le système de traduction pour des résultats optimaux.

Les données d’optimisation sont utilisées lors de l’entraînement pour ajuster tous les paramètres et les pondérations du système de traduction sur les valeurs optimales. Choisissez vos données d’optimisation avec soin : elles doivent être représentatives du contenu des documents que vous prévoyez de traduire dans le futur. Les données d’optimisation ont une influence majeure sur la qualité des traductions produites. L’optimisation permet au système de traduction de produire des traductions qui sont les plus proches des exemples que vous fournissez dans les données d’optimisation. Vous n’avez pas besoin de plus de 2 500 phrases dans vos données d’optimisation. Pour une qualité optimale de traduction, il est recommandé de sélectionner le jeu de paramétrage manuellement en choisissant la sélection la plus représentative des phrases.

Quand vous créez votre jeu de paramétrage, choisissez des phrases d’une longueur significative et représentative des futures phrases que vous prévoyez de traduire. Vous devez également choisir des phrases avec des mots et expressions que vous allez traduire dans la distribution approximative attendue dans vos futures traductions. En pratique, une longueur de phrase de 7 à 10 mots produit les meilleurs résultats, car ces phrases contiennent suffisamment de contexte pour montrer de l’inflexion et fournir une longueur de phrase qui soit significative sans être trop complexe.

Une bonne description du type de phrases à utiliser dans le jeu de paramétrage est la prose : des phrases concrètes fluides. Pas de cellules de tableau, pas de poèmes, pas de listes de choses, pas seulement des signes de ponctuation ni des nombres dans une phrase : un langage normal.

Si vous sélectionnez manuellement vos données d’optimisation, n’y incluez aucune phrase identique à des phrases contenues dans vos données d’entraînement et de test. Comme les données d’optimisation ont un impact significatif sur la qualité des traductions, choisissez les phrases avec soin.

Si vous ne savez pas quoi choisir pour vos données d’optimisation, sélectionnez simplement les données d’entraînement et laissez Custom Translator sélectionner vos données d’optimisation pour vous. Quand vous laissez Custom Translator choisir les données d’optimisation automatiquement, il utilise un sous-ensemble aléatoire de phrases de vos documents d’entraînement bilingues et exclut ces phrases des données d’entraînement elles-mêmes.

## <a name="testing-dataset-for-custom-translator"></a>Jeu de données de test pour Custom Translator

Les documents parallèles inclus dans le jeu de test sont utilisés pour calculer le score BLEU (Bilingual Evaluation Understudy). Ce score indique la qualité de votre système de traduction. Ce score vous signale en réalité dans quelle mesure les traductions effectuées par le système de traduction à l’issue de cet entraînement correspondent aux phrases de référence du jeu de données de test.

Le score BLEU est une mesure de la différence entre la traduction automatique et la traduction de référence. Sa valeur est comprise entre 0 et 100. Un score de 0 indique que pas un seul mot de la référence n’apparaît dans la traduction. Un score de 100 indique que la traduction automatique correspond exactement à la référence : le même mot figure à la même position exacte. Le score que vous recevez est la moyenne du score BLEU pour toutes les phrases des données d’entraînement.

Les données de test doivent inclure des documents parallèles où les phrases de la langue cible sont les traductions les plus souhaitables des phrases de la langue source correspondante dans la paire source-cible. Vous pouvez utiliser les mêmes critères que ceux utilisés pour composer les données d’optimisation. Cependant, les données de test n’ont aucune influence sur la qualité du système de traduction. Elles sont utilisées exclusivement pour générer le score BLEU pour vous.

Vous n’avez pas besoin de plus de 2 500 phrases comme données de test. Quand vous laissez le système choisir le jeu de test automatiquement, il utilise un sous-ensemble aléatoire de phrases de vos documents d’entraînement bilingues et exclut ces phrases du support d’entraînement lui-même.

Vous pouvez afficher les traductions personnalisées du jeu de test et les comparer aux traductions fournies dans votre jeu de test, en accédant à l’onglet de test dans un modèle.