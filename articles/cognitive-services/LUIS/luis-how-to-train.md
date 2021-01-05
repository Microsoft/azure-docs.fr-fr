---
title: Effectuer l’apprentissage de l’application - LUIS
titleSuffix: Azure Cognitive Services
description: L’entraînement est le processus au cours duquel votre version d’application Language Understanding (LUIS) apprend afin d’améliorer sa compréhension du langage naturel. Formez votre application LUIS après des mises à jour du modèle, telles qu’un ajout, une édition, un étiquetage ou une suppression d’entités, d’intentions ou d’énoncés.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 11/15/2019
ms.openlocfilehash: a53857061110d5a77ac3d166277e7076f4f7f9c6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91541371"
---
# <a name="train-your-active-version-of-the-luis-app"></a>Entraîner votre version active de l’application LUIS

La formation est le processus au cours duquel votre application Language Understanding (LUIS) apprend afin d’améliorer sa compréhension du langage naturel. Formez votre application LUIS après des mises à jour du modèle, telles qu’un ajout, une édition, un étiquetage ou une suppression d’entités, d’intentions ou d’énoncés.

La formation et le [test](luis-concept-test.md) d’une application est un processus itératif. Après avoir formé votre application LUIS, vous la testez avec des exemples d’énoncés afin de voir si les intentions et les entités sont reconnues correctement. Si ce n’est pas le cas, apportez des mises à jour à l’application LUIS, puis formez-la et testez-la à nouveau.

La formation est appliquée à la version active dans le portail LUIS.

## <a name="how-to-train-interactively"></a>Comment effectuer l’apprentissage de manière interactive

Pour démarrer le processus itératif dans le [portail LUIS](https://www.luis.ai), vous devez commencer par effectuer l’apprentissage de votre application LUIS au moins une fois. Assurez-vous que chaque intention a au moins un énoncé avant l’apprentissage.

1. Accédez à votre application en sélectionnant son nom dans la page **My Apps** (Mes applications).

1. Dans le volet supérieur votre application, sélectionnez **Train** (Former).

1. Une fois la formation terminée, une notification s’affiche en haut du navigateur.

## <a name="training-date-and-time"></a>Date et heure de l’apprentissage

La date et l’heure de l’apprentissage sont GMT+2.

## <a name="train-with-all-data"></a>Entraîner avec toutes les données

L’entraînement utilise un petit pourcentage d’échantillonnage négatif. Si vous souhaitez utiliser toutes les données au lieu du petit échantillonnage négatif, utilisez l’[API](#version-settings-api-use-of-usealltrainingdata).

### <a name="version-settings-api-use-of-usealltrainingdata"></a>Utilisation des paramètres de version d’API UseAllTrainingData

Utilisez l’[API des paramètres de version](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/versions-update-application-version-settings) avec le paramètre `UseAllTrainingData` défini sur true pour désactiver cette fonctionnalité.

## <a name="unnecessary-training"></a>Formation inutile

Vous n’avez pas besoin d’effectuer l’apprentissage après chaque modification. La formation doit être effectuée après l’application d’un groupe de modifications au modèle et l’étape suivante consiste à tester ou à publier. Si vous n’avez pas besoin de tester ou de publier, la formation n’est pas nécessaire.

## <a name="training-with-the-rest-apis"></a>Formation avec les API REST

Pour effectuer l’apprentissage dans le portail LUIS, il suffit de cliquer sur le bouton **Former**. La formation avec les API REST est un processus en deux étapes. La première consiste à [demander une apprentissage](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c45) avec HTTP POST. Ensuite, demandez l’[état de la formation](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c46) avec HTTP Get.

Pour savoir quand la formation est terminée, vous devez interroger l’état jusqu’à ce que tous les modèles sont correctement formés.

## <a name="next-steps"></a>Étapes suivantes

* [Test interactif](luis-interactive-test.md)
* [Test par lot](luis-how-to-batch-test.md)
