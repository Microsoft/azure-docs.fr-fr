---
title: Appairage et alignement des phrases - Custom Translator
titleSuffix: Azure Cognitive Services
description: Pendant l’exécution de l’apprentissage, les phrases présentes dans les documents parallèles sont appariées ou alignées. Custom Translator apprend à traduire une phrase à la fois, en lisant une phrase, la traduction de cette phrase. Ensuite, il aligne les mots et les phrases de ces deux phrases les uns avec les autres.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: e5d360933cbeb611046aede6164ec2e2fa497664
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347793"
---
# <a name="sentence-pairing-and-alignment-in-parallel-documents"></a>Appairage et alignement des phrases dans des documents parallèles

Une fois les documents chargés, les phrases présentes dans les documents parallèles sont appairées ou alignées. Custom Translator indique le nombre de phrases qu’il a pu apparier comme Phrases alignées dans chacun des jeux de données.

## <a name="pairing-and-alignment-process"></a>Processus d’appairage et d’alignement

Custom Translator apprend la traduction de phrases à raison d’une phrase à la fois. Il lit une phrase de la source, puis la traduction de cette phrase de la cible. Ensuite, il aligne les mots et les phrases de ces deux phrases les uns avec les autres. Ce processus lui permet de créer un mappage des mots et des phrases en une phrase avec les mots et les phrases équivalents dans la traduction de cette phrase. L’alignement vise à garantir que le système effectue son apprentissage sur des phrases qui sont des traductions les unes des autres.

## <a name="pre-aligned-documents"></a>Documents préalignés

Si vous savez que vous avez des documents parallèles, vous pouvez remplacer l’alignement des phrases en fournissant des fichiers texte préalignés. Vous pouvez extraire toutes les phrases des deux documents dans un fichier texte, organiser une phrase par ligne et le charger avec une extension `.align`. L’extension `.align` signale à Custom Translator qu’il doit ignorer l’alignement des phrases.

Pour de meilleurs résultats, assurez-vous d’avoir une phrase par ligne dans vos fichiers.  Évitez d’avoir des caractères de saut de ligne au sein d’une phrase, car cela entraîne des alignements médiocres.

## <a name="suggested-minimum-number-of-sentences"></a>Nombre minimal de phrases suggéré

Pour qu’un entraînement réussisse, le tableau ci-dessous indique le nombre minimal de phrases nécessaires dans chaque type de document. Cette limitation est un filet de sécurité qui garantit que vos phrases parallèles contiendront suffisamment de vocabulaire unique pour entraîner correctement un modèle de traduction. La règle générale est qu’avoir plus de phrases parallèles dans le domaine d’une qualité égale à celle des traductions humaines doit produire des modèles de qualité supérieure.

| Type du document   | Nombre minimal de phrases suggéré | Nombre maximal de phrases |
|------------|--------------------------------------------|--------------------------------|
| Entrainement   | 10 000                                     | Pas de limite supérieure                 |
| Réglage     | 500                                      | 2 500       |
| Test    | 500                                      | 2 500  |
| Dictionnaire | 0                                          | Pas de limite supérieure                 |

> [!NOTE]
> - L’entraînement ne commencera pas et échouera si le nombre minimal de 10 000 phrases d’entraînement n’est pas atteint. 
> - Le réglage et le test sont facultatifs. Si vous ne les fournissez pas, le système supprime un pourcentage approprié de l’entraînement à utiliser pour la validation et le test. 
> - Il est possible d’entraîner un modèle en n’utilisant que des données de dictionnaire. Reportez-vous à [Qu’est-ce qu’un dictionnaire ?](./what-is-dictionary.md).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment utiliser un [dictionnaire](what-is-dictionary.md) dans Custom Translator.
