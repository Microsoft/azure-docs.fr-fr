---
title: 'Tutoriel : Créer le modèle prédictif avec le Machine Learning automatisé (partie 1 sur 2)'
titleSuffix: Azure Machine Learning
description: Découvrez comment créer et déployer des modèles Machine Learning automatisé, afin de pouvoir utiliser le meilleur modèle pour prédire les résultats dans Microsoft Power BI.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.author: samkemp
author: samuel100
ms.reviewer: sdgilley
ms.date: 12/11/2020
ms.openlocfilehash: 897f493edf6ccdebb25c201e8e4f9babfb0754c5
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97370756"
---
# <a name="tutorial-power-bi-integration---create-the-predictive-model-using-automated-machine-learning-part-1-of-2"></a>Tutoriel : Intégration de Power BI – Créer le modèle prédictif avec le Machine Learning automatisé (partie 1 sur 2)

Dans la première partie de ce tutoriel, vous entraînez et déployez un modèle Machine Learning prédictif à l’aide du Machine Learning automatisé dans le studio Azure Machine Learning.  Dans la partie 2, vous utilisez le modèle le plus performant pour prédire des résultats dans Microsoft Power BI.

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Créer un cluster de calcul Azure Machine Learning
> * Créer un jeu de données
> * Créer une exécution de ML automatisé
> * Déployer le meilleur modèle sur un point de terminaison de scoring en temps réel


Il existe trois façons différentes de créer et de déployer le modèle que vous utiliserez dans Power BI.  Cet article couvre l’option C : Entraîner et déployer des modèles à l’aide du Machine Learning automatisé dans le studio.  Cette option montre une expérience de création sans code qui automatise entièrement la préparation des données et l’entraînement du modèle. 

Vous pouvez utiliser à la place :

* [Option A : Entraîner et déployer des modèles à l’aide de notebooks](tutorial-power-bi-custom-model.md) – expérience de création Code First à l’aide de notebooks Jupyter hébergés dans le studio Azure Machine Learning.
* [Option B : Entraîner et déployer des modèles à l’aide du concepteur](tutorial-power-bi-designer-model.md) – expérience de création à faible code à l’aide du concepteur (interface utilisateur de glisser-déplacer).

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure ([un essai gratuit est disponible](https://aka.ms/AMLFree)). 
- Un espace de travail Azure Machine Learning. Si vous ne disposez pas déjà d’un espace de travail, suivez le [guide pratique pour créer un espace de travail Azure Machine Learning](./how-to-manage-workspace.md#create-a-workspace).

## <a name="create-compute-cluster"></a>Créer un cluster de calcul

Le Machine Learning automatisé entraîne automatiquement un grand nombre de modèles Machine Learning différents pour trouver l’algorithme et les paramètres « les plus adaptés ». Azure Machine Learning parallélise l’exécution de l’entraînement du modèle sur un cluster de calcul.

Dans le [studio Azure Machine Learning](https://ml.azure.com), sélectionnez **Calculer** dans le menu de gauche, puis l’onglet **Clusters de calcul**. Sélectionnez **Nouveau** :

:::image type="content" source="media/tutorial-power-bi/create-compute-cluster.png" alt-text="Capture d’écran montrant comment créer un cluster de calcul":::

Dans l’écran **Créer un cluster de calcul** :

1. Sélectionnez une taille de machine virtuelle (pour les besoins de ce tutoriel, une machine `Standard_D11_v2` est correcte).
1. Sélectionnez **Suivant**.
1. Spécifiez un nom de calcul valide.
1. Conservez un **nombre minimal de nœuds** de 0.
1. Modifiez le **nombre maximal de nœuds** en spécifiant 4.
1. Sélectionnez **Créer**

Vous pouvez voir que l’état de votre cluster est passé à **Création**.

>[!NOTE]
> Une fois le cluster créé, il aura 0 nœud, ce qui signifie qu’aucun coût de calcul n’est facturé. Vous générez des coûts seulement quand le travail de Machine Learning automatisé est exécuté. Le cluster revient automatiquement à 0 après un temps d’inactivité de 120 secondes.


## <a name="create-dataset"></a>Créer un jeu de données

Dans ce tutoriel, vous utilisez le [dataset Diabetes](https://www4.stat.ncsu.edu/~boos/var.select/diabetes.html), qui est mis à votre disposition dans [Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/).

Pour créer le dataset, sélectionnez **Datasets** dans le menu de gauche, puis **Créer un dataset** – les options suivantes s’affichent :

:::image type="content" source="media/tutorial-power-bi/create-dataset.png" alt-text="Capture d’écran montrant comment créer un dataset":::

Sélectionnez **Depuis Open Datasets**, puis dans l’écran **Créer un dataset depuis Open Datasets** :

1. Recherchez *diabetes* à l’aide de la barre de recherche.
1. Sélectionnez **Exemple : Diabetes**.
1. Sélectionnez **Suivant**.
1. Spécifiez un nom pour votre dataset – *diabetes*.
1. Sélectionnez **Créer**

Vous pouvez explorer les données en sélectionnant le dataset puis **Explorer** :

:::image type="content" source="media/tutorial-power-bi/explore-dataset.png" alt-text="Capture d’écran montrant comment explorer le dataset":::

Les données comportent 10 variables d’entrée de mesures de référence (telles que l’âge, le sexe, l’indice de masse corporelle, la pression artérielle moyenne et six mesures d’analyse sanguine) et une variable cible nommée **Y** (une mesure quantitative de la progression du diabète un an après les mesures de référence).

## <a name="create-automated-ml-run"></a>Créer l’exécution de ML automatisé

Dans le [studio Azure Machine Learning](https://ml.azure.com), sélectionnez **ML automatisé** dans le menu de gauche, puis **Nouvelle exécution de ML automatisé** :

:::image type="content" source="media/tutorial-power-bi/create-new-run.png" alt-text="Capture d’écran montrant comment créer une nouvelle exécution de ML automatisé":::

Ensuite, sélectionnez le dataset **diabetes** que vous avez créé précédemment et sélectionnez **Suivant** :

:::image type="content" source="media/tutorial-power-bi/select-dataset.png" alt-text="Capture d’écran montrant comment sélectionner un dataset":::
 
Dans l’écran **Configurer l’exécution** :

1. Sous **Nom de l’expérience**, sélectionnez **Créer nouveau**.
1. Spécifiez un nom pour l’expérience.
1. Dans le champ Colonne cible, sélectionnez **Y**.
1. Dans le champ **Sélectionner un cluster de calcul**, sélectionnez le cluster de calcul que vous avez créé précédemment. 

Votre formulaire rempli doit ressembler au suivant :

:::image type="content" source="media/tutorial-power-bi/configure-automated.png" alt-text="Capture d’écran montrant comment configurer le ML automatisé":::

Enfin, vous devez sélectionner la tâche de Machine Learning à effectuer, à savoir **Régression** :

:::image type="content" source="media/tutorial-power-bi/configure-task.png" alt-text="Capture d’écran montrant comment configurer la tâche":::

Sélectionnez **Terminer**.

> [!IMPORTANT]
> Il faut environ 30 minutes pour que le ML automatisé termine l’entraînement des 100 modèles différents.

## <a name="deploy-the-best-model"></a>Déployer le meilleur modèle

Une fois l’exécution du ML automatisé terminée, vous pouvez voir la liste de tous les différents modèles Machine Learning qui ont été essayés en sélectionnant l’onglet **Modèles**. Les modèles sont classés dans l’ordre de leurs performances : le modèle le plus performant figure en tête de liste. Lorsque vous sélectionnez le meilleur modèle, le bouton **Déployer** est activé :

:::image type="content" source="media/tutorial-power-bi/list-models.png" alt-text="Capture d’écran montrant la liste des modèles":::

Sélectionnez **Déployer** pour afficher l’écran **Déployer un modèle** :

1. Spécifiez un nom pour votre service de modèle : utilisez **diabetes-model**.
1. Sélectionnez **Azure Container Service**.
1. Sélectionnez **Déployer**

Un message indiquant que le modèle a été déployé avec succès doit s’afficher.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez vu comment entraîner et déployer un modèle Machine Learning à l’aide du ML automatisé. Le tutoriel suivant présente la façon de consommer (noter) ce modèle à partir de Power BI.

> [!div class="nextstepaction"]
> [Tutoriel : Consommer un modèle dans Power BI](/power-bi/connect-data/service-aml-integrate?context=azure/machine-learning/context/ml-context)
