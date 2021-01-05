---
title: 'ML Studio (classique) : démarrer des expériences à partir d’exemples – Azure'
description: Découvrez comment utiliser les exemples d’expérience de machine learning pour créer des expériences avec Azure AI Gallery et Azure Machine Learning Studio (classique).
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: sample
author: likebupt
ms.author: keli19
ms.custom: seodec18, previous-author=heatherbshapiro, previous-ms.author=hshapiro
ms.date: 01/05/2018
ms.openlocfilehash: cfee8a32c54bff5aecb1e298db8b4814255418b0
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93325750"
---
# <a name="create-azure-machine-learning-studio-classic-experiments-from-working-examples-in-azure-ai-gallery"></a>Créer des expériences Azure Machine Learning Studio (classique) à partir d’exemples d’utilisation disponibles dans Azure AI Gallery

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)




Découvrez comment démarrer avec des exemples d’expérience de la [galerie Azure AI](https://gallery.azure.ai/) au lieu de créer des expériences d’apprentissage automatique de toutes pièces. Vous pouvez utiliser les exemples pour créer votre propre solution d’apprentissage automatique.

La galerie contient des exemples d’expérience fournis par l’équipe Microsoft Azure Machine Learning Studio (classique), ainsi que des exemples partagés par la communauté Machine Learning. Vous pouvez poser des questions ou publier des commentaires sur les expériences.

Pour savoir comment utiliser la galerie, regardez la vidéo de 3 minutes [Copier le travail d’autres personnes pour des projets de science des données](data-science-for-beginners-copy-other-peoples-work-to-do-data-science.md) issue de la série [Science des données pour les débutants](data-science-for-beginners-the-5-questions-data-science-answers.md).



## <a name="find-an-experiment-to-copy-in-azure-ai-gallery"></a>Rechercher une expérience à copier dans la galerie AI Azure
Pour afficher les expériences disponibles dans la galerie, rendez-vous dans la [Galerie](https://gallery.azure.ai/) , puis cliquez sur **Expériences** en haut de la page.

### <a name="find-the-newest-or-most-popular-experiments"></a>Rechercher les expériences les plus récentes ou les plus populaires
Dans cette page, vous pouvez afficher les expériences **Recently added** (Récemment ajoutées), consulter la section **What’s popular** (Les plus demandées) ou découvrir les dernières **expériences Microsoft les plus populaires**.

### <a name="look-for-an-experiment-that-meets-specific-requirements"></a>Rechercher une expérience qui remplit les conditions requises
Pour parcourir toutes les expériences :

1. Cliquez sur **Browse all** (Parcourir tout) en haut de la page.
2. À gauche, dans la section **Catégories** , sous **Affiner par** , sélectionnez **Tester** pour afficher toutes les expériences de la galerie.
3. Vous y trouverez des expériences qui répondent à vos besoins de différentes façons :
   * **Sélectionnez des filtres à gauche.** Par exemple, pour parcourir les expériences utilisant un algorithme de détection d’anomalie PCA : sous **Catégories** , cliquez sur **Tester**. Ensuite, sous **Algorithmes utilisés** , cliquez sur **Afficher tout** et choisissez **Détection d’anomalie PCA** dans la boîte de dialogue. Vous devrez peut-être faire défiler le voir.<br></br>
     ![Sélectionner des filtres](./media/sample-experiments/choose-an-algorithm.png)
   * **Utilisez la zone de recherche.** Par exemple, pour rechercher les expériences partagées par Microsoft sur la reconnaissance de chiffres et qui utilisent un algorithme de machine à vecteurs de support à deux classes, entrez « reconnaissance de chiffres » dans la zone de recherche. Ensuite, sélectionnez les filtres **Experiment** (Expérience), **Microsoft content only** (Contenu Microsoft uniquement) et **Two-Class Support Vector Machine** (Machine à vecteurs de support à deux classes) :<br></br>
     ![Utiliser la zone de recherche](./media/sample-experiments/search-for-experiments.png)
4. Cliquez sur une expérience pour en savoir plus à propos de celle-ci.
5. Pour exécuter et/ou modifier l’expérience, cliquez sur **Ouvrir dans Studio** sur la page de l’expérience. <br></br>

    ![Exemple d'expérience](./media/sample-experiments/example-experiment.png)

## <a name="create-a-new-experiment-using-an-example-as-a-template"></a>Créer une expérience à l’aide d’un exemple en tant que modèle
Vous pouvez également créer une expérience dans Machine Learning Studio (classique) en utilisant un exemple de la galerie comme modèle.

1. Connectez-vous à [Studio](https://studio.azureml.net) avec les informations d’identification de votre compte Microsoft, puis cliquez sur **Nouveau** pour créer une expérience.
2. Parcourez les exemples de contenu, puis cliquez sur l’un d’entre eux.

Une expérience est créée dans votre espace de travail Machine Learning Studio (classique) en utilisant l’exemple d’expérience comme modèle.

## <a name="next-steps"></a>Étapes suivantes
* [Importer des données à partir de diverses sources](import-data.md)
* [Déploiement d’un service web Machine Learning](deploy-a-machine-learning-web-service.md)