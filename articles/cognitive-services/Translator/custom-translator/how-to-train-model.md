---
title: Effectuer l'apprentissage d'un modèle - Custom Translator
titleSuffix: Azure Cognitive Services
description: L'apprentissage est une étape importante lors de la création d'un modèle de traduction. L'apprentissage repose sur les documents que vous sélectionnez.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 824516a327d45feb5b6a084a113633bd3a486abe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88508347"
---
# <a name="train-a-model"></a>Effectuer l’apprentissage d’un modèle

L’apprentissage d’un modèle est la première étape, et celle la plus importante, à suivre pour créer un modèle de traduction ; sinon, le modèle ne peut pas être généré. L'apprentissage repose sur les documents que vous sélectionnez. Lorsque vous sélectionnez des documents de type « Formation », tenez compte de l’exigence minimale de 10 000 phrases parallèles. Au fur et à mesure que vous sélectionnez des documents, nous affichons le nombre total de phrases de formation pour vous guider. Cette exigence ne s’applique pas lorsque vous sélectionnez uniquement des documents de type dictionnaire pour effectuer l’apprentissage d’un modèle.

Pour effectuer l'apprentissage d'un modèle :

1. Sélectionnez le projet dans lequel vous souhaitez générer un modèle.

2. L'onglet Données du projet affiche tous les documents pertinents pour la paire de langues du projet. Sélectionnez manuellement les documents que vous souhaitez utiliser pour effectuer l'apprentissage de votre modèle. Vous pouvez sélectionner les documents d'apprentissage, d'optimisation et de test à partir de cet écran. Et il vous suffit de sélectionner l'ensemble d'apprentissage et de laisser Custom Translator créer les ensembles d'optimisation et de test pour vous.

    - Nom du document : nom du document.

    - Association : déterminez si le document est un document bilingue ou unilingue. Les documents unilingues ne sont actuellement pas pris en charge pour l'apprentissage.

    - Type de document : apprentissage, optimisation, test ou dictionnaire.

    - Paire de langues : affiche la langue source et la langue cible du projet.

    - Phrases sources : affiche le nombre de phrases extraites du fichier source.

    - Phrases cibles : affiche le nombre de phrases extraites du fichier cible.

    ![Effectuer l'apprentissage du modèle](media/how-to/how-to-train-model.png)

3. Cliquez sur le bouton « Créer un modèle ».

4. Dans la boîte de dialogue, spécifiez le nom de votre modèle. Par défaut, l’option « Effectuer l’apprentissage immédiatement » est sélectionnée pour démarrer le pipeline de formation lorsque vous cliquez sur le bouton « Créer un modèle ». Vous pouvez sélectionner « Enregistrer en tant que brouillon » pour créer les métadonnées du modèle et mettre le modèle à l’état de brouillon, mais la formation du modèle ne démarrera pas. Par la suite, vous devez sélectionner manuellement les modèles à l’état brouillon dont vous souhaitez effectuer l’apprentissage.

5. Cliquez sur le bouton « Créer un modèle ».

    ![Boîte de dialogue Effectuer l'apprentissage du modèle](media/how-to/how-to-train-model-2.png)

6. Custom Translator soumet la formation et affiche son état dans l’onglet Modèles.

    ![Page Effectuer l'apprentissage d'un modèle](media/how-to/how-to-train-model-3.png)

>[!Note]
>Custom Translator prend en charge 10 formations simultanées au sein d’un espace de travail à tout moment.

## <a name="modify-a-model-name"></a>Modifier un nom de modèle

Vous pouvez modifier un nom de modèle à partir de la page Détails du modèle.

1. Dans la page des projets, cliquez sur le nom du projet où se trouve le modèle.
2. Cliquez sur l’onglet Modèle.
3. Cliquez sur le nom du modèle pour afficher les détails du modèle.
4. Cliquez sur l'icône en forme de crayon.

    ![Modifier le modèle](media/how-to/how-to-edit-model.png)

5. Dans la boîte de dialogue, modifiez le nom du modèle et donnez à votre modèle un nom explicite.

    ![Boîte de dialogue Modifier plus](media/how-to/how-to-edit-model-dialog.png)

6. Cliquez sur Enregistrer.

## <a name="next-steps"></a>Étapes suivantes

- Apprenez à [afficher les détails du modèle](how-to-view-model-details.md).
