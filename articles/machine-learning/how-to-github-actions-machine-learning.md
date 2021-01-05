---
title: GitHub Actions pour CI/CD
titleSuffix: Azure Machine Learning
description: En savoir plus sur la création d’un workflow GitHub Actions pour entraîner un modèle sur Azure Machine Learning
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: juliakm
ms.author: jukullam
ms.date: 10/19/2020
ms.topic: conceptual
ms.custom: github-actions-azure
ms.openlocfilehash: e7f6066cb7ed5c166d3e2bdc3f895073b05b92b9
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97605023"
---
# <a name="use-github-actions-with-azure-machine-learning"></a>Utiliser GitHub Actions avec Azure Machine Learning

Prise en main de [GitHub Actions](https://docs.github.com/en/free-pro-team@latest/actions) pour entraîner un modèle sur Azure Machine Learning. 

> [!NOTE]
> La fonctionnalité GitHub Actions pour Azure Machine Learning est fournie telle quelle et n’est pas entièrement prise en charge par Microsoft. Si vous rencontrez des problèmes avec une action spécifique, ouvrez un problème dans le référentiel pour l’action. Par exemple, si vous rencontrez un problème avec l’action aml-deploy, signalez le problème dans le référentiel [https://github.com/Azure/aml-deploy]( https://github.com/Azure/aml-deploy).

## <a name="prerequisites"></a>Prérequis 

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un compte GitHub. Si vous n’en avez pas, inscrivez-vous [gratuitement](https://github.com/join).  

## <a name="workflow-file-overview"></a>Vue d’ensemble du fichier de workflow

Un workflow est défini par un fichier YAML (.yml) situé dans le chemin `/.github/workflows/` de votre dépôt. Cette définition contient les étapes et les paramètres qui composent le workflow.

Le fichier comporte quatre sections :

|Section  |Tâches  |
|---------|---------|
|**Authentification** | 1. Définissez un principal de service. <br /> 2. Créez un secret GitHub. |
|**Connexion** | 1. Connectez-vous à un espace de travail de machine learning. <br /> 2. Connectez-vous à une cible de calcul. |
|**Exécuter** | 1. Soumettez une exécution d’entraînement. |
|**Déployer** | 1. Inscrivez le modèle auprès d’un registre Azure Machine Learning. 1. Déployer le modèle. |

## <a name="create-repository"></a>Créer un dépôt

Créez un nouveau référentiel à partir du [ML OPS avec GitHub Actions et un modèle Azure Machine Learning](https://github.com/machine-learning-apps/ml-template-azure). 

1. Ouvrez le [modèle](https://github.com/machine-learning-apps/ml-template-azure) sur GitHub. 
2. Sélectionnez **Utiliser ce modèle**. 

    Sélectionnez :::image type="content" source="media/how-to-github-actions-machine-learning/gh-actions-use-template.png" alt-text="Utiliser ce modèle":::
3. Créez un référentiel à partir du modèle. Définissez le nom du référentiel sur `ml-learning` ou sur le nom de votre choix. 


## <a name="generate-deployment-credentials"></a>Générer les informations d’identification du déploiement

Vous pouvez créer un [principal de service](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) à l’aide de la commande [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac&preserve-view=true) dans [Azure CLI](/cli/azure/). Exécutez cette commande en utilisant [Azure Cloud Shell](https://shell.azure.com/) dans le portail Azure ou en sélectionnant le bouton **Essayer**.

```azurecli-interactive
az ad sp create-for-rbac --name "myML" --role contributor \
                            --scopes /subscriptions/<subscription-id>/resourceGroups/<group-name> \
                            --sdk-auth
```

Dans l’exemple ci-dessus, remplacez les espaces réservés par votre ID d’abonnement, le nom de votre groupe de ressources et le nom de votre application. La sortie correspond à un objet JSON avec les informations d’identification de l’attribution de rôle qui fournit l’accès à votre application App Service, similaire à ce qui suit. Copiez cet objet JSON pour une version ultérieure.

```output 
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

## <a name="configure-the-github-secret"></a>Configurer le secret GitHub

1. Dans [GitHub](https://github.com/), parcourez votre référentiel, sélectionnez **Paramètres > Secrets > Ajouter un nouveau secret**.

2. Collez l’intégralité de la sortie JSON de la commande Azure CLI dans le champ de valeur du secret. Nommez le secret `AZURE_CREDENTIALS`.

## <a name="connect-to-the-workspace"></a>Se connecter à l’espace de travail

Utilisez l’[action Espace de travail Azure Machine Learning](https://github.com/marketplace/actions/azure-machine-learning-workspace) pour vous connecter à votre espace de travail Azure Machine Learning. 

```yaml
    - name: Connect/Create Azure Machine Learning Workspace
      id: aml_workspace
      uses: Azure/aml-workspace@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
```

Par défaut, l’action attend un fichier `workspace.json`. Si votre fichier JSON porte un nom différent, vous pouvez le spécifier avec le paramètre d’entrée `parameters_file`. S’il n’y a pas de fichier, un nouveau sera créé avec le nom du référentiel.


```yaml
    - name: Connect/Create Azure Machine Learning Workspace
      id: aml_workspace
      uses: Azure/aml-workspace@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          parameters_file: "alternate_workspace.json"
```
L’action écrit les propriétés de l’espace de travail Azure Resource Manager (ARM) dans un fichier de configuration, qui sera choisi par toutes les futures fonctionnalités GitHub Actions Azure Machine Learning. Le fichier est enregistré dans `GITHUB_WORKSPACE/aml_arm_config.json`. 

## <a name="connect-to-a-compute-target-in-azure-machine-learning"></a>Se connecter à une cible de calcul dans Azure Machine Learning

Utilisez l’[action Capacité de calcul Azure Machine Learning](https://github.com/Azure/aml-compute) pour vous connecter à une cible de calcul dans Azure Machine Learning.  Si la cible de calcul existe, l’action s’y connectera. Dans le cas contraire, l’action créera une nouvelle cible de calcul. L’action [Capacité de calcul AML](https://github.com/Azure/aml-compute) prend uniquement en charge le cluster de calcul Azure ML et le service Azure Kubernetes Service (AKS). 

```yaml
    - name: Connect/Create Azure Machine Learning Compute Target
      id: aml_compute_training
      uses: Azure/aml-compute@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
```
## <a name="submit-training-run"></a>Soumettre une exécution d’entraînement

Utilisez l'[action Entraînement Azure Machine Learning](https://github.com/Azure/aml-run) pour soumettre un ScriptRun, un estimateur ou un pipeline à Azure Machine Learning. 

```yaml
    - name: Submit training run
      id: aml_run
      uses: Azure/aml-run@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
```

## <a name="register-model-in-registry"></a>Inscrire un modèle dans le registre

Utilisez l'[action Inscrire le modèle Azure Machine Learning](https://github.com/Azure/aml-registermodel) pour inscrire un modèle dans Azure Machine Learning.

```yaml
    - name: Register model
      id: aml_registermodel
      uses: Azure/aml-registermodel@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          run_id:  ${{ steps.aml_run.outputs.run_id }}
          experiment_name: ${{ steps.aml_run.outputs.experiment_name }}
```

## <a name="deploy-model-to-azure-machine-learning-to-aci"></a>Déployer un modèle sur Azure Machine Learning vers ACI

Utilisez l'[action Déployer Azure Machine Learning](https://github.com/Azure/aml-deploy) pour déployer un modèle et créer un point de terminaison pour le modèle. Vous pouvez également utiliser Déployer Azure Machine Learning pour déployer vers Azure Kubernetes Service. Consultez [cet exemple de workflow](https://github.com/Azure-Samples/mlops-enterprise-template) pour un modèle qui se déploie sur Azure Kubernetes Service.

```yaml
    - name: Deploy model
      id: aml_deploy
      uses: Azure/aml-deploy@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          model_name:  ${{ steps.aml_registermodel.outputs.model_name }}
          model_version: ${{ steps.aml_registermodel.outputs.model_version }}

```

## <a name="complete-example"></a>Exemple complet

Entraîner et déployer votre modèle sur Azure Machine Learning. 

```yaml
# Actions train a model on Azure Machine Learning
name: Azure Machine Learning training and deployment
on:
  push:
    branches:
      - master
    # paths:
    #   - 'code/*'
jobs:
  train:
    runs-on: ubuntu-latest
    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - name: Check Out Repository
      id: checkout_repository
      uses: actions/checkout@v2
        
    # Connect or Create the Azure Machine Learning Workspace
    - name: Connect/Create Azure Machine Learning Workspace
      id: aml_workspace
      uses: Azure/aml-workspace@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
    
    # Connect or Create a Compute Target in Azure Machine Learning
    - name: Connect/Create Azure Machine Learning Compute Target
      id: aml_compute_training
      uses: Azure/aml-compute@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
    
    # Submit a training run to the Azure Machine Learning
    - name: Submit training run
      id: aml_run
      uses: Azure/aml-run@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}

    # Register model in Azure Machine Learning model registry
    - name: Register model
      id: aml_registermodel
      uses: Azure/aml-registermodel@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          run_id:  ${{ steps.aml_run.outputs.run_id }}
          experiment_name: ${{ steps.aml_run.outputs.experiment_name }}

    # Deploy model in Azure Machine Learning to ACI
    - name: Deploy model
      id: aml_deploy
      uses: Azure/aml-deploy@v1
      with:
          azure_credentials: ${{ secrets.AZURE_CREDENTIALS }}
          model_name:  ${{ steps.aml_registermodel.outputs.model_name }}
          model_version: ${{ steps.aml_registermodel.outputs.model_version }}

```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque votre groupe de ressource et référentiel ne sont plus nécessaires, nettoyez les ressources que vous avez déployées en supprimant le groupe de ressources et votre référentiel GitHub. 

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer et exécuter des pipelines de machine learning avec le kit SDK Azure Machine Learning](how-to-create-your-first-pipeline.md)
