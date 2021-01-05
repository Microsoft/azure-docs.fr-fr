---
title: Afficher les détails d’un modèle – Custom Translator
titleSuffix: Azure Cognitive Services
description: 'L’onglet Models (Modèles), sous n’importe quel projet, donne des informations sur chaque modèle : nom, état, score BLEU, nombre de phrases d’apprentissage, d’optimisation et de test, etc.'
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 52e305f9d50ab0f383810778be681ac41be97c72
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88509782"
---
# <a name="view-model-details"></a>Afficher les détails du modèle

L’onglet Models (Modèles) qui se trouve sous le projet présente tous les modèles entraînés pour ce projet.

Pour chacun des modèles du projet, ces informations s’affichent.

1. Nom du modèle : affiche le nom d'un modèle donné.

2. État : affiche l'état d'un modèle donné. Un nouvel apprentissage conserve l’état Submitted (Soumis) jusqu'à ce qu’il soit accepté. Il devient ensuite Dataprocessing (Traitement de données) lorsque le service évalue le contenu des documents. Une fois cette évaluation terminée, l’état passe à Running (En cours d’exécution) : vous pouvez alors voir le nombre de phrases impliquées dans l’apprentissage, y compris dans les jeux d’optimisation et de test qui sont créés automatiquement. Voici la liste des états des modèles.

    - Submitted : spécifie que le serveur principal traite les documents correspondant à ce modèle.

    - TrainingQueued : spécifie que la formation se trouve dans la file d'attente du système de traduction automatique correspondant à ce modèle.

    - Running : spécifie que la formation est en cours d'exécution dans le système de traduction automatique correspondant à ce modèle.

    - Succeeded : spécifie que la formation a abouti dans le système de traduction automatique et qu'un modèle est disponible. Dans cet état, un score BLEU s’affiche pour ce modèle.

    - Deployed: spécifie que le modèle formé est soumis au système de traduction automatique en vue de son déploiement.

    - Undeploying : spécifie que le déploiement du modèle est en cours d'annulation.

    - Undeployed : spécifie que le processus d'annulation du déploiement d'un modèle a abouti.

    - Training Failed : spécifie que la formation a échoué. En cas d’échec de l’apprentissage, relancez la tâche d’apprentissage. Si l’erreur persiste, contactez-nous. Ne supprimez pas le modèle qui a échoué.

    - DataProcessingFailed : spécifie que le traitement de données a échoué pour un ou plusieurs documents appartenant au modèle.

    - DeploymentFailed : spécifie que le déploiement du modèle a échoué.

    - MigratedDraft : spécifie que le modèle est à l'état de brouillon après la migration de Hub vers Custom Translator.

3. BLEU Score (Score BLEU) : score BLEU (Bilingual Evaluation Understudy) du modèle. Il révèle la qualité du système de traduction, en indiquant dans quelle mesure les traductions effectuées par le système de traduction à l’issue de cet apprentissage correspondent aux phrases de référence du jeu de données de test. Le score BLEU apparaît si l’apprentissage s’est terminé avec succès. Si l’apprentissage n’est pas fini ou a échoué, il ne s’affiche pas.

4. Training Sentence count : nombre total de phrases utilisées dans le jeu d’apprentissage.

5. Tuning Sentence count : nombre total de phrases utilisées dans le jeu d'optimisation.

6.  Training Sentence count : nombre total de phrases utilisées dans le jeu de test.

7.  Mono Sentence count : nombre total de phrases utilisées dans le jeu mono.

8.  Bouton d’action Deploy (Déployer) : un modèle dont l’apprentissage a réussi, mais qui n’a pas été déployé présente le bouton Deploy (Déployer). Si le modèle est déployé, le bouton Undeploy (Annuler le déploiement) s’affiche.

9. Supprimer : bouton permettant de supprimer le modèle. Le fait de supprimer un modèle n’entraîne pas la suppression des documents utilisés pour le créer.

    ![Afficher les détails du modèle](media/how-to/how-to-view-model-details.png)

>[!Note]
>Pour comparer des apprentissages consécutifs sur les mêmes systèmes, il est important que le jeu d’optimisation et le jeu de test restent constants.

## <a name="view-model-training-details"></a>Afficher les informations d’apprentissage du modèle

Une fois l’apprentissage terminé, vous pouvez consulter des informations à ce sujet sur la page des détails. Sélectionnez un projet, recherchez et sélectionnez l’onglet Models (Modèles), puis choisissez un modèle.

La page de modèle comporte deux onglets : Détails de la formation et Test.

1.  **Détails de la formation :** cet onglet présente la liste des documents utilisés pour la formation :

    -  Nom du document : ce champ affiche le nom du document.

    -  Type du document : ce champ affiche le type du document (en parallèle/mono).

    -  Nombre de phrases dans la langue source : ce champ affiche le nombre de phrases présentes dans la langue source.

    -  Nombre de phrases dans la langue cible : ce champ affiche le nombre de phrases présentes dans la langue cible.

    -  Phrases alignées : ce champ affiche le nombre de phrases alignées par Custom Translator pendant le processus d'alignement.

    -  Phrases utilisées : ce champ affiche le nombre de phrases utilisées par Custom Translator pendant cette formation.

    ![Informations d’apprentissage du modèle](media/how-to/how-to-model-training-details.png)

2.  **Test :** cet onglet affiche les détails du test pour une formation réussie.

## <a name="next-steps"></a>Étapes suivantes

- Consultez les [résultats des tests](how-to-view-system-test-results.md) et analysez les résultats de l’apprentissage.
