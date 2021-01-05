---
title: Qu’est-ce qu’un espace de travail ?
titleSuffix: Azure Machine Learning
description: L’espace de travail est la ressource de niveau supérieur du service Azure Machine Learning. Il conserve un historique de toutes les exécutions d’entraînement, avec les journaux, les métriques, les sorties et un instantané de vos scripts.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sgilley
author: sdgilley
ms.date: 09/22/2020
ms.openlocfilehash: 7acb555ddde20d6512119af85a8db055bcd9eeed
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94701651"
---
# <a name="what-is-an-azure-machine-learning-workspace"></a>Qu’est-ce qu’un espace de travail Azure Machine Learning ?

L’espace de travail est la ressource de niveau supérieur pour Azure Machine Learning. Il fournit un emplacement centralisé dans lequel exploiter tous les artefacts que vous créez lorsque vous utilisez Azure Machine Learning.  L’espace de travail conserve un historique de toutes les exécutions d’entraînement, y compris les journaux d’activité, les métriques, les sorties et un instantané de vos scripts. Vous utilisez ces informations pour déterminer quelle exécution d’entraînement produit le meilleur modèle.  

Une fois que vous disposez d’un modèle qui vous convient, inscrivez-le avec l’espace de travail. Ainsi, grâce au modèle inscrit et aux scripts de scoring, vous pouvez déployer sur Azure Container Instances, Azure Kubernetes Service ou sur un tableau FPGA (field programmable gate array) comme point de terminaison HTTP basé sur REST. Vous pouvez également déployer le modèle en tant que module sur un appareil Azure IoT Edge.

## <a name="taxonomy"></a>Taxonomie 

Le diagramme suivant représente une taxonomie de l’espace de travail :

[![Taxonomie de l’espace de travail](./media/concept-workspace/azure-machine-learning-taxonomy.png)](./media/concept-workspace/azure-machine-learning-taxonomy.png#lightbox)

Le diagramme montre les composants d’un espace de travail suivants :

+ Un espace de travail peut contenir des [instances de calcul Azure Machine Learning](concept-compute-instance.md), des ressources cloud configurées avec l’environnement Python nécessaire pour exécuter Azure Machine Learning.

+ Les [rôles utilisateur](how-to-assign-roles.md) vous permettent de partager votre espace de travail avec d’autres utilisateurs, équipes ou projets.
+ Les [cibles de calcul](concept-azure-machine-learning-architecture.md#compute-targets) vous permettent d’exécuter vos expériences.
+ Lorsque vous créez l’espace de travail, les [ressources associées](#resources) sont également créées à votre place.
+ Les [expériences](concept-azure-machine-learning-architecture.md#experiments) sont des exécutions d’entraînement qui vous permettent de créer vos modèles.  
+ Les [pipelines](concept-azure-machine-learning-architecture.md#ml-pipelines) sont des flux de travail réutilisables destinés à l’entraînement et la reformation de votre modèle.
+ Les [jeux de données](concept-azure-machine-learning-architecture.md#datasets-and-datastores) facilitent la gestion des données que vous utilisez pour l’entraînement des modèles et la création de pipelines.
+ Une fois que vous avez un modèle que vous souhaitez déployer, vous créez un modèle inscrit.
+ Utilisez le modèle inscrit et un script de scoring pour créer un [point de terminaison de déploiement](concept-azure-machine-learning-architecture.md#endpoints).

## <a name="tools-for-workspace-interaction"></a>Outils pour l’interaction avec l’espace de travail

Vous pouvez interagir avec votre espace de travail comme suit :

> [!IMPORTANT]
> Les outils marqués (préversion) ci-dessous sont actuellement en préversion publique.
> La préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail en production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

+ Sur le web :
    + [Azure Machine Learning studio ](https://ml.azure.com) 
    + [Concepteur Azure Machine Learning](concept-designer.md) 
+ Dans un environnement Python avec le [kit de développement logiciel (SDK) Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py).
+ Dans un environnement R avec le [kit de développement logiciel (SDK) Azure Machine Learning pour R (préversion)](https://azure.github.io/azureml-sdk-for-r/reference/index.html).
+ Sur la ligne de commande avec l’[extension CLI](./reference-azure-machine-learning-cli.md) Azure Machine Learning :
+ [Extension VS Code Azure Machine Learning](how-to-manage-resources-vscode.md#workspaces)


## <a name="machine-learning-with-a-workspace"></a>Machine Learning avec un espace de travail

Les tâches de Machine Learning lisent et/ou écrivent des artefacts sur votre espace de travail.

+ Exécuter une expérience pour entraîner un modèle : écrit les résultats de l’exécution d’expérience sur l’espace de travail.
+ Utiliser le Machine Learning automatisé pour entraîner un modèle : écrit les résultats de l’entraînement sur l’espace de travail.
+ Inscrivez un modèle dans l’espace de travail.
+ Déployer un modèle : utilise le modèle inscrit pour créer un déploiement.
+ Créez et exécutez des flux de travail réutilisables.
+ Affichez les artefacts de Machine Learning (expériences, pipelines, modèles, déploiements, etc.).
+ Suivez et supervisez les modèles.

## <a name="workspace-management"></a>Gestion de l’espace de travail

Vous pouvez également effectuer les tâches de gestion de l’espace de travail suivantes :

| Tâche de gestion de l’espace de travail   | Portail              | Studio | Kit SDK Python / Kit SDK R       | Interface de ligne de commande        | VS Code
|---------------------------|---------|---------|------------|------------|------------|
| Créer un espace de travail        | **&check;**     | | **&check;** | **&check;** | **&check;** |
| Gérer les accès à l’espace de travail    | **&check;**   || |  **&check;**    ||
| Créer et gérer des ressources de calcul    | **&check;**   | **&check;** | **&check;** |  **&check;**   ||
| Créer une machine virtuelle de notebooks |   | **&check;** | |     ||

> [!WARNING]
> Le déplacement de votre espace de travail Azure Machine Learning vers un autre abonnement, ou le déplacement de l’abonnement propriétaire vers un nouveau locataire, n’est pas pris en charge. En effet, cela peut provoquer des erreurs.

## <a name="create-a-workspace"></a><a name='create-workspace'></a> Créer un espace de travail

Il existe plusieurs moyens de créer un espace de travail :  

* Utilisez le [portail Azure](how-to-manage-workspace.md?tabs=azure-portal#create-a-workspace) comme interface de type pointer-cliquer pour vous guider dans chaque étape.
* Utilisez le [SDK Azure Machine Learning pour Python](how-to-manage-workspace.md?tabs=python#create-a-workspace) pour créer un espace de travail à la volée à partir de scripts Python ou de notebooks Jupyter.
* Utilisez un [modèle Azure Resource Manager](how-to-create-workspace-template.md) ou l’[interface CLI Azure Machine Learning](reference-azure-machine-learning-cli.md) lorsque vous devez automatiser ou personnaliser la création avec des normes de sécurité d’entreprise.
* Si vous travaillez dans Visual Studio Code, utilisez l'[extension VS Code](how-to-manage-resources-vscode.md#create-a-workspace).

> [!NOTE]
> Le nom de l’espace de travail n’est pas sensible à la casse.

## <a name="associated-resources"></a><a name="resources"></a> Ressources associées

Lorsque vous créez un nouvel espace de travail, celui-ci crée automatiquement plusieurs ressources Azure qui sont utilisées par l’espace de travail :

+ [Compte Stockage Azure](https://azure.microsoft.com/services/storage/) : Utilisé comme magasin de données par défaut pour l’espace de travail.  Les notebooks Jupyter utilisés avec vos instances de calcul Azure Machine Learning sont également stockés ici. 
  
  > [!IMPORTANT]
  > Par défaut, le compte de stockage est un compte v1 à usage général. Vous pouvez [mettre ce compte à niveau vers un compte v2 à usage général](../storage/common/storage-account-upgrade.md) après la création de l’espace de travail. N’activez pas l’espace de noms hiérarchique sur le compte de stockage après une mise à niveau vers un compte v2 universel.

  Si vous souhaitez utiliser un compte de stockage Azure existant, il ne doit pas s’agir d’un compte Premium (Premium_LRS ou Premium_GRS). Il ne peut pas non plus comporter d’espace de noms hiérarchique (utilisé avec Azure Data Lake Storage Gen2). Ni le stockage Premium ni les espaces de noms hiérarchiques ne sont pris en charge avec le compte de stockage _par défaut_ de l’espace de travail. Ils peuvent en revanche être utilisés avec des comptes de stockage _autres que les comptes par défaut_.
  
+ [Azure Container Registry](https://azure.microsoft.com/services/container-registry/) : Enregistre les conteneurs docker que vous utilisez pendant la formation et lorsque vous déployez un modèle. Pour réduire les coûts, ACR est **chargé en différé** jusqu’à la création des images de déploiement.

+ [Azure Application Insights](https://azure.microsoft.com/services/application-insights/) : Stocke les informations de supervision concernant les modèles.

+ [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) : Stocke les secrets qui sont utilisés par les cibles de calcul, ainsi que d’autres informations sensibles dont a besoin l’espace de travail.

> [!NOTE]
> Vous pouvez utiliser à la place des instances de ressources Azure existantes lorsque vous créez l’espace de travail avec le [SDK Python](how-to-manage-workspace.md?tabs=python#create-a-workspace), le [SDK R](https://azure.github.io/azureml-sdk-for-r/reference/create_workspace.html) ou la Azure Machine Learning CLI [à l’aide d’un modèle ARM](how-to-create-workspace-template.md).

<a name="wheres-enterprise"></a>

## <a name="what-happened-to-enterprise-edition"></a>Évolution de l’édition Enterprise

À compter de septembre 2020, toutes les fonctionnalités disponibles dans les espaces de travail de l’édition Entreprise sont désormais également disponibles dans les espaces de travail de l’édition De base. Les nouveaux espaces de travail d’entreprise ne peuvent plus être créés.  Les appels SDK, CLI ou Azure Resource Manager qui utilisent le paramètre `sku` continuent de fonctionner, mais un espace de travail De base est approvisionné.

À compter du 21 décembre, tous les espaces de travail de l’édition Entreprise seront automatiquement définis sur l’édition De base, qui offre les mêmes fonctionnalités. Ce processus ne génère aucun temps d’arrêt. Le 1er janvier 2021, l’édition Entreprise sera officiellement supprimée. 

Les clients des deux éditions sont responsables des coûts des ressources Azure consommés et n’ont pas besoin de payer des frais supplémentaires pour Azure Machine Learning. Pour plus d’informations, consultez la page [Tarification Azure Machine Learning](https://azure.microsoft.com/pricing/details/machine-learning/).

## <a name="next-steps"></a>Étapes suivantes

Pour bien démarrer avec Azure Machine Learning, voir :

+ [Vue d’ensemble d’Azure Machine Learning](overview-what-is-azure-ml.md)
+ [Créer et gérer un espace de travail](how-to-manage-workspace.md)
+ [Tutoriel : Bien démarrer avec Azure Machine Learning dans votre environnement de développement](tutorial-1st-experiment-sdk-setup-local.md)
+ [Tutoriel : Bien démarrer avec la création de votre première expérience ML sur une instance de calcul](tutorial-1st-experiment-sdk-setup.md)
+ [Tutoriel : Bien démarrer avec Azure Machine Learning et le SDK R](tutorial-1st-r-experiment.md)
+ [Tutoriel : Créer votre premier modèle de classification avec le machine learning automatisé](tutorial-first-experiment-automated-ml.md) 
+ [Tutoriel : Prédire le prix de voitures avec le concepteur](tutorial-designer-automobile-price-train-score.md)
