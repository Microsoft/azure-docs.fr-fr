---
title: 'Tutoriel : Créer et déployer une compétence personnalisée avec Azure Machine Learning'
titleSuffix: Azure Cognitive Search
description: Ce tutoriel montre comment utiliser Azure Machine Learning pour créer et déployer une compétence personnalisée destinée au pipeline d’enrichissement par IA de la Recherche cognitive Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 09/25/2020
ms.openlocfilehash: fa961a5a6d3a3b827a082fbac2acc3431ac40949
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92057601"
---
# <a name="tutorial-build-and-deploy-a-custom-skill-with-azure-machine-learning"></a>Tutoriel : Créer et déployer une compétence personnalisée avec Azure Machine Learning 

Dans ce tutoriel, vous utilisez le [jeu de données des avis sur les hôtels](https://www.kaggle.com/datafiniti/hotel-reviews) (distribué dans le cadre de la licence Creative Commons [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.txt)) et vous créez une [compétence personnalisée](./cognitive-search-aml-skill.md) à l’aide d’Azure Machine Learning pour extraire, à partir des avis, des sentiments basés sur les aspects. L’affectation de sentiments positifs et négatifs au sein d’un même avis est ainsi possible, et une imputation correcte est alors effectuée aux entités identifiées, telles que le personnel, la chambre, la réception ou la piscine.

Pour entraîner le modèle de sentiment basé sur l’aspect dans Azure Machine Learning, vous allez utiliser le [dépôt de recettes nlp](https://github.com/microsoft/nlp-recipes/tree/master/examples/sentiment_analysis/absa). Le modèle sera ensuite déployé en tant que point de terminaison sur un cluster Azure Kubernetes. Aussitôt déployé, le point de terminaison est ajouté au pipeline d’enrichissement en tant que compétence Azure Machine Learning à utiliser par le service Recherche cognitive.

Deux jeux de données sont fournis. Si vous souhaitez effectuer l’apprentissage du modèle vous-même, le fichier hotel_reviews_1000.csv est demandé. Vous préférez ignorer l’étape d’entraînement ? Téléchargez le fichier hotel_reviews_100.csv.

> [!div class="checklist"]
> * Créer une instance Recherche cognitive Azure
> * Créer un espace de travail Azure Machine Learning (le service de recherche et l’espace de travail doivent se trouver dans le même abonnement)
> * Entraîner et déployer un modèle sur un cluster Azure Kubernetes
> * Lier un pipeline d’enrichissement par IA au modèle déployé
> * Ingérer la sortie du modèle déployé en tant que compétence personnalisée

> [!IMPORTANT] 
> Cette compétence est actuellement en préversion publique. Les fonctionnalités en préversion sont fournies sans contrat de niveau de service et ne sont pas recommandées pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Le SDK .NET n’est actuellement pas pris en charge.

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure : procurez-vous un [abonnement gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Service Recherche cognitive](./search-get-started-arm.md)
* [Ressource Cognitive Services](../cognitive-services/cognitive-services-apis-create-account.md?tabs=multiservice%2cwindows)
* [compte Stockage Azure](../storage/common/storage-account-create.md?tabs=azure-portal&toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
* [Espace de travail Azure Machine Learning](../machine-learning/how-to-manage-workspace.md)

## <a name="setup"></a>Programme d’installation

* Clonez ou téléchargez le contenu de l’[exemple de dépôt](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/AzureML-Custom-Skill).
* Procédez à l’extraction du contenu si le téléchargement est un fichier zip. Assurez-vous que les fichiers sont en lecture-écriture.
* Lors de la configuration des comptes et des services Azure, copiez les noms et les clés dans un fichier texte facilement accessible. Les noms et les clés sont ajoutés à la première cellule du notebook dans lequel les variables d’accès aux services Azure sont définies.
* Si vous n’êtes pas familiarisé avec Azure Machine Learning et ses spécifications, vous pouvez consulter ces documents avant de commencer :
 * [Configurer un environnement de développement pour Azure Machine Learning](../machine-learning/how-to-configure-environment.md)
 * [Créer et gérer des espaces de travail Azure Machine Learning dans le portail Azure](../machine-learning/how-to-manage-workspace.md)
 * Lors de la configuration de l’environnement de développement pour Azure Machine Learning, envisagez l’utilisation de l’[instance de calcul informatique](../machine-learning/how-to-configure-environment.md#compute-instance) pour accélérer et faciliter la prise en main.
* Chargez le fichier du jeu de données sur un conteneur dans le compte de stockage. Le plus gros fichier est nécessaire si vous souhaitez effectuer l’étape d’entraînement dans le notebook. Si vous préférez ignorer l’étape d’entraînement, le plus petit fichier est recommandé.

## <a name="open-notebook-and-connect-to-azure-services"></a>Ouvrir le notebook et se connecter aux services Azure

1. Placez dans la première cellule toutes les informations nécessaires aux variables qui permettront l’accès aux services Azure, puis exécutez la cellule.
1. L’exécution de la deuxième cellule confirmera que vous êtes connecté au service de recherche pour votre abonnement.
1. Les sections 1.1 à 1.5 vont créer le magasin de données, l’ensemble de compétences, l’index et l’indexeur du service de recherche.

À ce stade, vous pouvez choisir d’ignorer l’étape de création du jeu de données d’entraînement et celle d’expérimentation dans Azure Machine Learning pour passer directement à l’enregistrement des deux modèles fournis dans le dossier des modèles du dépôt GitHub. Si vous ignorez ces étapes, dans le notebook, passez à la section 3.5, Écrire un script de scoring. Vous gagnerez du temps, car les étapes de téléchargement et de chargement des données peuvent prendre jusqu’à 30 minutes.

## <a name="creating-and-training-the-models"></a>Création et entraînement des modèles

La section 2 comprend six cellules qui téléchargent le fichier des incorporations de gants à partir du dépôt de recettes nlp. Après le téléchargement, le fichier est ensuite chargé dans le magasin de données Azure Machine Learning. La taille du fichier .zip est de 2 Go environ, et l’exécution de ces tâches peut demander un certain temps. Une fois chargées, les données d’entraînement sont ensuite extraites ; vous êtes prêt désormais pour la section 3.

## <a name="train-the-aspect-based-sentiment-model-and-deploy-your-endpoint"></a>Entraîner le modèle de sentiment basé sur l’aspect et déployer votre point de terminaison

La section 3 du notebook entraîne les modèles qui ont été créés à la section 2 ; enregistrez ces modèles et déployez-les comme point de terminaison dans un cluster Azure Kubernetes. Si vous n’êtes pas familiarisé avec Azure Kubernetes, nous vous recommandons vivement de consulter les articles suivants avant d’essayer de créer un cluster d’inférence :

* [Présentation d’Azure Kubernetes Service](../aks/intro-kubernetes.md)
* [Concepts de base de Kubernetes pour Azure Kubernetes Service (AKS)](../aks/concepts-clusters-workloads.md)
* [Quotas, restrictions de taille de machine virtuelle et disponibilité des régions dans Azure Kubernetes Service (AKS)](../aks/quotas-skus-regions.md)

Les opérations de création et de déploiement du cluster d’inférence peuvent prendre jusqu’à 30 minutes. Pour tester le service web avant de passer aux dernières étapes, il est recommandé de mettre à jour votre ensemble de compétences et d’exécuter l’indexeur.

## <a name="update-the-skillset"></a>Mettre à jour l’ensemble de compétences

La section 4 du notebook comporte quatre cellules qui mettent à jour l’ensemble de compétences et l’indexeur. Vous pouvez également utiliser le portail pour sélectionner et appliquer la nouvelle compétence à l’ensemble de compétences, puis exécuter l’indexeur pour mettre à jour le service de recherche.

> [!VIDEO https://channel9.msdn.com/Shows/AI-Show/Active-Learning-with-Azure-Cognitive-Search/player#time=19m35s:paused/03/player]

Dans le portail, accédez à Ensemble de compétences, puis sélectionnez le lien Définition de l’ensemble de compétences (JSON). Le portail affiche le fichier JSON de l’ensemble de compétences qui a été créé dans les premières cellules du notebook. Dans la partie droite de l’écran, un menu déroulant vous permet de sélectionner le modèle de définition de l’ensemble de compétences. Sélectionnez le modèle Azure Machine Learning (AML). Indiquez le nom de l’espace de travail Azure Machine Learning ainsi que le point de terminaison du modèle déployé sur le cluster d’inférence. Le modèle sera mis à jour avec l’URI et la clé du point de terminaison.

> :::image type="content" source="media/cognitive-search-aml-skill/portal-aml-skillset-definition.png" alt-text="Modèle de définition de l’ensemble de compétences":::

À partir de la fenêtre, copiez le modèle de l’ensemble de compétences, puis collez-le dans la définition de l’ensemble de compétences, à gauche. Modifiez le modèle afin de préciser les valeurs manquantes pour :

* Nom
* Description
* Context
* Nom et source des « entrées »
* Nom et targetName des « sorties »

Enregistrez l’ensemble de compétences.

Après avoir enregistré l’ensemble de compétences, accédez à l’indexeur et sélectionnez le lien Définition de l’indexeur (JSON). Le portail affiche le code JSON de l’indexeur créé dans les premières cellules du notebook. Les mappages des champs de sortie devront être mis à jour avec les mappages de champs supplémentaires pour garantir la gestion et le passage correct par l’indexeur. Enregistrez vos modifications, puis sélectionnez Exécuter. 

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous travaillez dans votre propre abonnement, il est recommandé, à la fin de chaque projet, de déterminer si vous avez toujours besoin des ressources que vous avez créées. Les ressources laissées en cours d’exécution peuvent vous coûter de l’argent. Vous pouvez supprimer les ressources une par une, ou choisir de supprimer le groupe de ressources afin de supprimer l’ensemble des ressources.

Vous pouvez rechercher et gérer les ressources dans le portail à l’aide des liens **Toutes les ressources** ou **Groupes de ressources** situés dans le volet de navigation de gauche.

Si vous utilisez un service gratuit, n’oubliez pas que vous êtes limité à trois index, indexeurs et sources de données. Vous pouvez supprimer des éléments un par un dans le portail pour ne pas dépasser la limite.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Consulter l’API web de compétence personnalisée](./cognitive-search-custom-skill-web-api.md)
> [En savoir plus sur l’ajout de compétences personnalisées au pipeline d’enrichissement](./cognitive-search-custom-skill-interface.md)