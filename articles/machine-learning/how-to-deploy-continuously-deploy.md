---
title: Déployer en continu des modèles Azure Machine Learning
titleSuffix: Azure Machine Learning
description: Découvrez comment déployer en continu des modèles avec l’extension DevOps d’Azure Machine Learning. Recherchez et déployez automatiquement les nouvelles versions de modèle.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 08/03/2020
ms.topic: conceptual
ms.reviewer: larryfr
ms.custom: how-to, tracking-python, deploy
ms.openlocfilehash: 6043ea4e1366890033571c2ba78ecdb2e59f64e1
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93325204"
---
# <a name="continuously-deploy-models"></a>Déployer des modèles en continu

Cet article explique comment utiliser un déploiement continu dans Azure DevOps pour rechercher automatiquement de nouvelles versions de modèles inscrits et transmettre ces nouveaux modèles en production.

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous avez déjà inscrit un modèle dans votre espace de travail Azure Machine Learning. Consultez [ce didacticiel](how-to-train-scikit-learn.md) pour obtenir un exemple d’apprentissage et d’inscription d’un modèle scikit-Learn.

## <a name="continuously-deploy-models"></a>Déployer des modèles en continu

Vous pouvez déployer des modèles en continu à l’aide de l’extension Machine Learning pour [Azure DevOps](https://azure.microsoft.com/services/devops/). Vous pouvez utiliser l’extension Machine Learning pour Azure DevOps pour déclencher un pipeline de déploiement quand un nouveau modèle Machine Learning est inscrit dans un espace de travail Azure Machine Learning.

1. Inscrivez-vous sur [Azure Pipelines](/azure/devops/pipelines/get-started/pipelines-sign-up?view=azure-devops), ce qui permet l’intégration et la livraison continues de votre application vers n’importe quelle plateforme ou n’importe quel cloud. (Azure Pipelines et les [pipelines Machine Learning](concept-ml-pipelines.md#compare) sont deux choses différentes.)

1. [Créez un projet Azure DevOps.](/azure/devops/organizations/projects/create-project?view=azure-devops)

1. Installez l’[extension Machine Learning pour Azure Pipelines](https://marketplace.visualstudio.com/items?itemName=ms-air-aiagility.vss-services-azureml&targetId=6756afbe-7032-4a36-9cb6-2771710cadc2&utm_source=vstsproduct&utm_medium=ExtHubManageList).

1. Utilisez les connexions au service pour configurer une connexion de principal de service à votre espace de travail Azure Machine Learning afin d’accéder à vos artefacts. Accédez aux paramètres du projet, puis sélectionnez **Connexions au service** et **Azure Resource Manager**  :

    [![Sélectionner Azure Resource Manager](media/how-to-deploy-and-where/view-service-connection.png)](media/how-to-deploy-and-where/view-service-connection-expanded.png)

1. Dans la liste **Niveau de portée** , sélectionnez **AzureMLWorkspace** , puis entrez le reste des valeurs :

    ![Sélectionner AzureMLWorkspace](media/how-to-deploy-and-where/resource-manager-connection.png)

1. Pour déployer en continu votre modèle Machine Learning à l’aide d’Azure Pipelines, sélectionnez **Mise en production** sous Pipelines. Ajoutez un nouvel artefact, puis sélectionnez l’artefact **Modèle AzureML** et la connexion au service que vous avez créée précédemment. Sélectionnez le modèle et la version pour déclencher un déploiement :

    [![Sélectionner le modèle AzureML](media/how-to-deploy-and-where/enable-modeltrigger-artifact.png)](media/how-to-deploy-and-where/enable-modeltrigger-artifact-expanded.png)

1. Activez le déclencheur de modèle sur votre artefact de modèle. Quand vous activez le déclencheur, chaque fois que la version spécifiée de ce modèle (autrement dit, la version la plus récente) est inscrite dans votre espace de travail, un pipeline de mise en production Azure DevOps est déclenché.

    [![Activer le déclencheur de modèle](media/how-to-deploy-and-where/set-modeltrigger.png)](media/how-to-deploy-and-where/set-modeltrigger-expanded.png)

## <a name="next-steps"></a>Étapes suivantes

Consultez les projets ci-dessous sur GitHub pour obtenir des exemples supplémentaires de déploiement continu pour des modèles ML.

* [Microsoft/MLOps](https://github.com/Microsoft/MLOps)
* [Microsoft/MLOpsPython](https://github.com/microsoft/MLOpsPython)