---
title: Afficher les résultats des tests système et du déploiement - Custom Translator
titleSuffix: Azure Cognitive Services
description: Au terme de votre apprentissage, passez en revue les tests système pour analyser vos résultats. Si ces résultats vous conviennent, faites une demande de déploiement pour le modèle concerné.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 542f62f04adc7d4858e4b4621b0ae85e906434ad
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94369322"
---
# <a name="view-system-test-results"></a>Afficher les résultats de test système

Au terme de votre apprentissage, passez en revue les tests système pour analyser vos résultats. Si ces résultats vous conviennent, faites une demande de déploiement pour le modèle concerné.

## <a name="system-test-results-page"></a>Page relative aux résultats des tests système

Sélectionnez un projet, puis l’onglet Modèles de ce projet. Recherchez le modèle que vous souhaitez utiliser, puis sélectionnez l’onglet Test.

L’onglet Test affiche ce qui suit :

1.  **Résultats des tests système :** résultat du processus de test lors des entraînements. Le processus de test génère le score BLEU.

    **Sentence Count (Nombre de phases) :** nombre de phrases parallèles utilisées dans le jeu de test.

     **BLEU Score (Score BLEU) :** score BLEU généré pour un modèle au terme de l’apprentissage.

    **État :** indique si le processus de test est terminé ou en cours.

    ![Résultats des tests système](media/how-to/how-to-system-test-results.png)

2.  Cliquez sur les résultats des tests système pour accéder à la page des détails correspondants. Cette page affiche la traduction automatique de phrases qui faisaient partie du jeu de données de test.

3.  Le tableau de la page de détails des résultats des tests présente deux colonnes : une par langue. La colonne correspondant à la langue source affiche la phrase à traduire. La colonne correspondant à la langue cible contient deux phrases par ligne.

    **Ref :** cette phrase correspond à la traduction de référence de la phrase source comme indiqué dans le jeu de données de test.

    **MT :** cette phrase correspond à la traduction automatique de la phrase source effectuée par le modèle généré au terme de l’apprentissage.

    ![Comparaison des résultats des tests système](media/how-to/how-to-system-test-results-2.png)

## <a name="download-test"></a>Télécharger le test

Cliquez sur le lien Télécharger les traductions pour télécharger un fichier zip. Le fichier zip contient la traduction automatique de phrases source du jeu de données de test.

![Télécharger le test](media/how-to/how-to-system-test-download.png)

Cette archive zip téléchargée contient trois fichiers.

1.  **custom.mt.txt :** ce fichier contient la traduction automatique des phrases de la langue source vers la langue cible effectuée par le modèle formé avec les données de l’utilisateur.

2.  **ref.txt :** ce fichier contient les traductions fournies par l’utilisateur des phrases de la langue source vers la langue cible.

3.  **source.txt :** ce fichier contient des phrases dans la langue source.

    ![Résultats des tests système téléchargés](media/how-to/how-to-download-system-test.png)

## <a name="deploy-a-model"></a>Déployer un modèle

Pour demander un déploiement :

1.  Sélectionnez un projet, accédez à l’onglet Modèles.

2. Pour un modèle formé réussi, il affiche le bouton « Déployer », s'il n'est pas déployé.

    ![Capture d’écran mettant en évidence le bouton Déployer permettant de déployer un modèle.](media/how-to/how-to-deploy-model.png)

3.  Cliquez sur Déployer.
4.  Sélectionnez **Deployed (Déployé)** pour la ou les régions où vous souhaitez déployer votre modèle, puis cliquez sur Enregistrer. Vous pouvez sélectionner **Deployed (Déployé)** pour plusieurs régions.

    ![Capture d’écran montrant où vous pouvez déployer un modèle ou annuler son déploiement.](media/how-to/how-to-deploy-model-regions.png)

5.  Vous pouvez afficher l’état de votre modèle dans la colonne « État ».

>[!Note]
>Custom Translator prend en charge 10 modèles déployés au sein d’un espace de travail à tout moment.

## <a name="update-deployment-settings"></a>Mettre à jour les paramètres de déploiement

Pour mettre à jour les paramètres de déploiement :

1.  Sélectionnez un projet, accédez à l’onglet **Models (Modèles)** .

2. Pour un modèle déployé réussi, il affiche le bouton **Update (Mettre à jour)** .

    ![Capture d’écran mettant en évidence le bouton Mettre à jour pour la mise à jour des paramètres de déploiement.](media/how-to/how-to-update-undeploy-model.png)

3.  Sélectionnez **Update**.
4.  Sélectionnez **Deployed (Déployé)** ou **Undeployed (Non déployé)** pour la ou les régions où vous souhaitez déployer votre modèle (ou en annuler le déploiement), puis cliquez sur **Save (Enregistrer)** .

    ![Déployer un modèle](media/how-to/how-to-undeploy-model.png)

>[!Note]
>Si vous sélectionnez **Undeployed (Non déployé)** pour toutes les régions, le déploiement du modèle est annulé dans toutes les régions et le modèle est mis dans un état annulé. Il n’est plus disponible pour utilisation.

## <a name="next-steps"></a>Étapes suivantes

- Commencez à utiliser votre modèle Custom Translator déployé via [Microsoft Translator Text API V3](../reference/v3-0-translate.md?tabs=curl).
- Découvrez [comment gérer les paramètres](how-to-manage-settings.md) pour partager votre espace de travail, gérer la clé d’abonnement.
- Découvrez [comment migrer votre espace de travail et le projet](how-to-migrate.md) depuis [Microsoft Translator Hub](https://hub.microsofttranslator.com)