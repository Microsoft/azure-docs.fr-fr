---
title: 'Tutoriel : Créer le modèle prédictif par glisser-déplacer (partie 1 sur 2)'
titleSuffix: Azure Machine Learning
description: Découvrez comment créer et déployer un modèle prédictif Machine Learning avec le concepteur, afin de pouvoir l’utiliser pour prédire des résultats dans Microsoft Power BI.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.author: samkemp
author: samuel100
ms.reviewer: sdgilley
ms.date: 12/11/2020
ms.openlocfilehash: f0e1ffe60069a2379f8eddab1aae74db2b4eac50
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97370660"
---
# <a name="tutorial--power-bi-integration---drag-and-drop-to-create-the-predictive-model-part-1-of-2"></a>Tutoriel :  Intégration Power BI – Créer le modèle prédictif par glisser-déplacer (partie 1 sur 2)

Dans la première partie de ce tutoriel, vous entraînez et déployez un modèle Machine Learning prédictif à l’aide du concepteur Azure Machine Learning, une interface utilisateur de glisser-déplacer à faible code. Dans la partie 2, vous utiliserez le modèle pour prédire des résultats dans Microsoft Power BI.

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Créer une instance de calcul Azure Machine Learning
> * Créer un cluster d’inférence Azure Machine Learning
> * Créer un jeu de données
> * Entraîner un modèle de régression
> * Déployer le modèle sur un point de terminaison de scoring en temps réel


Il existe trois façons différentes de créer et de déployer le modèle que vous utiliserez dans Power BI.  Cet article couvre l’option B : Entraîner et déployer des modèles à l’aide du concepteur.  Cette option illustre une expérience de création à faible code à l’aide du concepteur (une interface utilisateur de glisser-déplacer).  

Vous pouvez utiliser à la place :

* [Option A : Entraîner et déployer des modèles à l’aide de notebooks](tutorial-power-bi-custom-model.md) – expérience de création Code First à l’aide de notebooks Jupyter hébergés dans le studio Azure Machine Learning.
* [Option C : Entraîner et déployer des modèles à l’aide du Machine Learning automatisé](tutorial-power-bi-automated-model.md) – expérience de création sans code qui automatise entièrement la préparation des données et l’entraînement du modèle.

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure ([un essai gratuit est disponible](https://aka.ms/AMLFree)). 
- Un espace de travail Azure Machine Learning. Si vous ne disposez pas déjà d’un espace de travail, suivez le [guide pratique pour créer un espace de travail Azure Machine Learning](./how-to-manage-workspace.md#create-a-workspace).
- Connaissances préliminaires des workflows de Machine Learning.


## <a name="create-compute-for-training-and-scoring"></a>Créer un calcul pour l’entraînement et le scoring

Dans cette section, vous créez une *instance de calcul*, qui est utilisée pour l’entraînement de modèles Machine Learning. En outre, vous créez un *cluster d’inférence* qui sera utilisé pour héberger le modèle déployé pour le scoring en temps réel.

Connectez-vous au [studio Azure Machine Learning](https://ml.azure.com) et sélectionnez **Calculer** dans le menu de gauche, puis **Nouveau** :

:::image type="content" source="media/tutorial-power-bi/create-new-compute.png" alt-text="Capture d’écran montrant comment créer une instance de calcul":::

Dans l’écran **Créer une instance de calcul** obtenu, sélectionnez une taille de machine virtuelle (pour ce tutoriel, sélectionnez `Standard_D11_v2`), puis **Suivant**. Dans la page des paramètres, fournissez un nom valide pour votre instance de calcul, puis sélectionnez **Créer**. 

>[!TIP]
> L’instance de calcul peut également être utilisée pour créer et exécuter des notebooks.

Vous pouvez maintenant voir que l’**état** de votre instance de calcul est **Création en cours** – le provisionnement de la machine prendra environ 4 minutes. Pendant que vous patientez, sélectionnez l’onglet **Cluster d’inférence** dans la page de calcul, puis **Nouveau** :

:::image type="content" source="media/tutorial-power-bi/create-cluster.png" alt-text="Capture d’écran montrant comment créer un cluster d’inférence":::

Dans la page **Créer un cluster d’inférence** obtenue, sélectionnez une région suivie d’une taille de machine virtuelle (pour ce tutoriel, sélectionnez `Standard_D11_v2`), puis sélectionnez **Suivant**. Dans la page **Configurer les paramètres** :

1. Spécifiez un nom de calcul valide.
1. Sélectionnez **Dev-test** comme objectif de cluster (crée un nœud unique pour héberger le modèle déployé).
1. Sélectionnez **Créer**

Vous pouvez maintenant voir que l’**état** de votre cluster d’inférence est **Création en cours** – le déploiement de votre cluster à nœud unique prendra environ 4 minutes.

## <a name="create-a-dataset"></a>Créer un jeu de données

Dans ce tutoriel, vous utilisez le [dataset Diabetes](https://www4.stat.ncsu.edu/~boos/var.select/diabetes.html), qui est mis à votre disposition dans [Azure Open Datasets](https://azure.microsoft.com/services/open-datasets/).

Pour créer le dataset, dans le menu de gauche, sélectionnez **Datasets**, puis **Créer un dataset** – les options suivantes s’affichent :

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

## <a name="create-a-machine-learning-model-using-designer"></a>Créer un modèle Machine Learning à l’aide du concepteur

Une fois que vous avez créé le calcul et les datasets, vous pouvez passer à la création du modèle Machine Learning à l’aide du concepteur. Dans le studio Azure Machine Learning, sélectionnez **Concepteur**, puis **Nouveau pipeline** :

:::image type="content" source="media/tutorial-power-bi/create-designer.png" alt-text="Capture d’écran montrant comment créer un pipeline":::

Vous voyez un *canevas* vide où vous pouvez également voir un **menu Paramètres** :

:::image type="content" source="media/tutorial-power-bi/select-compute.png" alt-text="Capture d’écran montrant comment sélectionner une cible de calcul":::

Dans le **menu Paramètres**, choisissez **Sélectionner une cible de calcul**, sélectionnez l’instance de calcul que vous avez créée précédemment, puis sélectionnez **Enregistrer**. Renommez votre **nom de brouillon** en un nom plus facile à mémoriser (par exemple, *diabetes-model*) et entrez une description.

Ensuite, dans la liste des ressources, développez **Datasets** et recherchez le dataset **diabetes** – effectuez un glisser-déplacer de ce module sur le canevas :

:::image type="content" source="media/tutorial-power-bi/drag-component.png" alt-text="Capture d’écran montrant comment faire glisser un composant":::

Ensuite, effectuez un glisser-déplacer des composants suivants sur le canevas :

1. Régression linéaire (située dans **Algorithmes de Machine Learning**)
1. Entraîner le modèle (situé dans **Entraînement de modèle**)

Votre canevas devrait ressembler à ceci (notez que le haut et le bas des composants présentent de petits cercles appelés ports, encadrés en rouge ci-dessous) :

:::image type="content" source="media/tutorial-power-bi/connections.png" alt-text="Capture d’écran montrant des composants non connectés":::
 
Ensuite, vous devez *relier* ces composants les uns aux autres. Sélectionnez le port au bas du dataset **diabetes** et faites-le glisser jusqu’au port droit en haut du composant **Entraîner le modèle**. Sélectionnez le port au bas du composant **Régression linéaire** et faites-le glisser jusqu’au port gauche en haut du port **Entraîner le modèle**.

Choisissez la colonne dans le dataset à utiliser comme variable d’étiquette (cible) à prédire. Sélectionnez le composant **Entraîner le modèle**, puis **Modifier la colonne**. Dans la boîte de dialogue, sélectionnez **Entrer le nom de la colonne** puis **Y** dans la liste déroulante :

:::image type="content" source="media/tutorial-power-bi/label-columns.png" alt-text="Capture d’écran montrant la sélection de la colonne d’étiquette":::

Sélectionnez **Enregistrer**. Votre *workflow* de Machine Learning doit se présenter comme suit :

:::image type="content" source="media/tutorial-power-bi/connected-diagram.png" alt-text="Capture d’écran montrant des composants connectés":::

Sélectionnez **Soumettre** puis **Créer nouveau** sous l’expérience. Fournissez un nom pour l’expérience, puis sélectionnez **Soumettre**.

>[!NOTE]
> La première exécution de votre expérience doit prendre environ 5 minutes. Les exécutions suivantes sont beaucoup plus rapides – les caches du concepteur exécutent déjà les composants pour réduire la latence.

Une fois l’expérience terminée, vous voyez :

:::image type="content" source="media/tutorial-power-bi/completed-run.png" alt-text="Capture d’écran montrant l’exécution terminée":::

Vous pouvez inspecter les journaux de l’expérience en sélectionnant **Entraîner le modèle** puis **Sorties + journaux**.

## <a name="deploy-the-model"></a>Déployer le modèle

Pour déployer le modèle, sélectionnez **Créer un pipeline d’inférence** (commande située en haut du canevas), puis **Pipeline d’inférence en temps réel** :

:::image type="content" source="media/tutorial-power-bi/pipeline.png" alt-text="Capture d’écran montrant la commande Pipeline d’inférence en temps réel":::
 
Le pipeline se réduit uniquement aux composants nécessaires pour effectuer le scoring du modèle. Lorsque vous effectuez le scoring des données, vous ne connaissez pas les valeurs des variables cibles. Par conséquent, nous pouvons supprimer **Y** du dataset. Pour supprimer cela, ajoutez au canevas un composant **Sélectionner des colonnes dans le dataset**. Reliez le composant pour que le dataset diabetes soit l’entrée, et que les résultats soient la sortie dans le composant **Noter le modèle** :

:::image type="content" source="media/tutorial-power-bi/remove-column.png" alt-text="Capture d’écran montrant la suppression d’une colonne":::

Sélectionnez le composant **Sélectionner des colonnes dans le dataset** sur le canevas, puis **Modifier les colonnes**. Dans la boîte de dialogue Sélectionner les colonnes, sélectionnez **Par nom**, puis vérifiez que toutes les variables d’entrée sont sélectionnées, mais **pas** la cible :

:::image type="content" source="media/tutorial-power-bi/removal-settings.png" alt-text="Capture d’écran montrant la suppression des paramètres d’une colonne":::

Sélectionnez **Enregistrer**. Enfin, sélectionnez le composant **Noter le modèle** et veillez à ce que la case à cocher **Ajouter des colonnes de score à la sortie** soit désactivée (seules les prédictions sont renvoyées, plutôt que les entrées *et* les prédictions, ce qui réduit la latence) :

:::image type="content" source="media/tutorial-power-bi/score-settings.png" alt-text="Capture d’écran montrant les paramètres du composant Noter le modèle":::

En haut du canevas, sélectionnez **Envoyer**.

Une fois que vous avez correctement exécuté le pipeline d’inférence, vous pouvez déployer le modèle dans votre cluster d’inférence. Sélectionnez **Déployer**, ce qui affichera la boîte de dialogue **Configurer le point de terminaison en temps réel**. Sélectionnez **Déployer un nouveau point de terminaison en temps réel**, nommez le point de terminaison **my-diabetes-model**, sélectionnez l’inférence que vous avez créée précédemment, puis sélectionnez **Déployer** :

:::image type="content" source="media/tutorial-power-bi/endpoint-settings.png" alt-text="Capture d’écran montrant les paramètres de point de terminaison en temps réel":::
## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez vu comment entraîner et déployer un modèle de concepteur. Dans la partie suivante, vous allez apprendre à consommer (noter) ce modèle à partir de Power BI.

> [!div class="nextstepaction"]
> [Tutoriel : Consommer un modèle dans Power BI](/power-bi/connect-data/service-aml-integrate?context=azure/machine-learning/context/ml-context)
