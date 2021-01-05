---
title: MLflow Tracking pour les expériences de Machine Learning Azure Databricks
titleSuffix: Azure Machine Learning
description: Configurez MLflow avec Azure Machine Learning pour consigner les métriques et les artefacts des expériences de Machine Learning Azure Databricks.
services: machine-learning
author: nibaccam
ms.author: nibaccam
ms.service: machine-learning
ms.subservice: core
ms.reviewer: nibaccam
ms.date: 09/22/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 72079cc399eea249bce4d285e2c3c4fbf9304708
ms.sourcegitcommit: 6cca6698e98e61c1eea2afea681442bd306487a4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/24/2020
ms.locfileid: "97760603"
---
# <a name="track-azure-databricks-ml-experiments-with-mlflow-and-azure-machine-learning-preview"></a>Assurer le suivi des expériences de Machine Learning Azure Databricks avec MLflow et Azure Machine Learning (préversion)

Cet article explique comment activer l'URI de suivi et l'API de journalisation de MLflow, collectivement appelées [MLflow Tracking](https://mlflow.org/docs/latest/quickstart.html#using-the-tracking-api), pour lier vos expériences Azure Databricks (ADB), MLflow et Azure Machine Learning.

[MLflow](https://www.mlflow.org) est une bibliothèque open source permettant de gérer le cycle de vie de vos expériences de Machine Learning. MLFlow Tracking est un composant de MLflow qui consigne et assure le suivi de vos métriques d'exécution d'apprentissage et de vos artefacts de modèle. Découvrez-en plus sur [Azure Databricks et MLflow](/azure/databricks/applications/mlflow/). 

Consultez [Assurer le suivi des exécutions d'expériences et créer des points de terminaison avec MLflow et Azure Machine Learning](how-to-use-mlflow.md) pour bénéficier d'intégrations supplémentaires de fonctionnalités MLflow et Azure Machine Learning.

>[!NOTE]
> En tant que bibliothèque open source, MLflow change fréquemment. Ainsi, les fonctionnalités mises à disposition via l’intégration d’Azure Machine Learning et de MLflow doivent être considérées comme une préversion et non entièrement prises en charge par Microsoft.

> [!TIP]
> Les informations contenues dans ce document sont principalement destinées aux scientifiques des données et aux développeurs qui veulent superviser le processus d’entraînement du modèle. Si vous êtes un administrateur intéressé par la supervision de l’utilisation de ressources et d’événements d’Azure Machine Learning, comme des quotas, des cycles de formation accomplis ou des déploiements de modèles effectués, voir [Supervision d’Azure Machine Learning](monitor-azure-machine-learning.md).

## <a name="prerequisites"></a>Prérequis

* Installez le package `azureml-mlflow`. 
    * Ce package intègre automatiquement `azureml-core` du [Kit de développement logiciel (SDK) Python Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py), qui fournit la connectivité nécessaire pour que MLflow accède à votre espace de travail.
* Un [cluster et un espace de travail Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal).
* [Créez un espace de travail Azure Machine Learning](how-to-manage-workspace.md).

## <a name="track-azure-databricks-runs"></a>Suivre les exécutions Azure Databricks

MLflow Tracking avec Azure Machine Learning vous permet de stocker les métriques et artefacts consignés à partir de vos exécutions Azure Databricks dans vos espaces de travail suivants : 

* Espace de travail Azure Databricks.
* Espace de travail Azure Machine Learning

Après avoir créé votre cluster et votre espace de travail Azure Databricks : 

1. Installez la bibliothèque *azureml-mlflow* de PyPi pour vous assurer que votre cluster peut accéder aux fonctions et classes nécessaires.

1. Configurez votre notebook d'expérience.

1. Reliez vos espaces de travail Azure Databricks et Azure Machine Learning.

Des informations supplémentaires sur ces étapes sont fournies dans les sections suivantes pour vous permettre de mener à bien vos expériences MLflow avec Azure Databricks. 

## <a name="install-libraries"></a>Installation des bibliothèques

Pour installer des bibliothèques sur votre cluster, accédez à l'onglet **Bibliothèques** et sélectionnez **Installer**.

 ![mlflow avec azure databricks](./media/how-to-use-mlflow-azure-databricks/azure-databricks-cluster-libraries.png)

Dans le champ **Package**, entrez azureml-mlflow, puis sélectionnez Installer. Répétez cette étape si nécessaire afin d’installer d'autres packages sur votre cluster pour votre expérience.

 ![Installation de bibliothèque mlflow Azure DB](./media/how-to-use-mlflow-azure-databricks/install-libraries.png)

## <a name="set-up-your-notebook"></a>Configuration de votre notebook 

Une fois votre cluster ADB configuré : 
1. Sélectionnez **Espaces de travail** dans le volet de navigation de gauche. 
1. Développez le menu déroulant des espaces de travail et sélectionnez **Importer**.
1. Effectuez un glisser-déplacer, ou recherchez votre notebook d'expérience, pour importer votre espace de travail ADB.
1. Sélectionnez **Importer**. Votre notebook d'expérience s'ouvre automatiquement.
1. Sous le titre du notebook en haut à gauche, sélectionnez le cluster auquel que vous souhaitez joindre votre notebook d'expérience. 

## <a name="connect-your-azure-databricks-and-azure-machine-learning-workspaces"></a>Lier vos espaces de travail Azure Databricks et Azure Machine Learning

Le fait de lier votre espace de travail ADB à votre espace de travail Azure Machine Learning vous permet de suivre vos données d'expérience dans l'espace de travail Azure Machine Learning.

Pour lier votre espace de travail ADB à un espace de travail Azure Machine Learning (nouveau ou existant) : 
1. Connectez-vous au [portail Azure](https://ms.portal.azure.com).
1. Accédez à la **page de présentation** de votre espace de travail ADB.
1. Sélectionnez le bouton **Lier l'espace de travail Azure Machine Learning** en bas à droite. 

 ![Lier les espaces de travail Azure DB et Azure Machine Learning](./media/how-to-use-mlflow-azure-databricks/link-workspaces.png)

## <a name="mlflow-tracking-in-your-workspaces"></a>MLflow Tracking dans vos espaces de travail

Une fois que vous avez instancié votre espace de travail, MLflow Tracking est automatiquement configuré pour être suivi aux emplacements suivants :

* L’espace de travail Azure Machine Learning lié.
* Votre espace de travail ADB d’origine. 

Toutes vos expériences sont consignées dans le service de suivi Azure Machine Learning managé.

Le code suivant doit figurer dans votre notebook d'expérience pour l'accès à votre espace de travail Azure Machine Learning lié. 

Ce code : 

*  obtient les détails de votre abonnement Azure pour instancier votre espace de travail Azure Machine Learning ; 

* part du principe que vous disposez d'un groupe de ressources et d'un espace de travail Azure Machine Learning, sinon vous pouvez [les créer](how-to-manage-workspace.md) ; 

* définit le nom de l'expérience. Le `user_name` ici est cohérent avec le `user_name` associé à l'espace de travail Azure Databricks.

```python
import mlflow
import mlflow.azureml
import azureml.mlflow
import azureml.core

from azureml.core import Workspace

subscription_id = 'subscription_id'

# Azure Machine Learning resource group NOT the managed resource group
resource_group = 'resource_group_name' 

#Azure Machine Learning workspace name, NOT Azure Databricks workspace
workspace_name = 'workspace_name'  

# Instantiate Azure Machine Learning workspace
ws = Workspace.get(name=workspace_name,
                   subscription_id=subscription_id,
                   resource_group=resource_group)

#Set MLflow experiment. 
experimentName = "/Users/{user_name}/{experiment_folder}/{experiment_name}" 
mlflow.set_experiment(experimentName) 

```

### <a name="set-mlflow-tracking-to-only-track-in-your-azure-machine-learning-workspace"></a>Configurer MLflow Tracking pour qu’il effectue le suivi uniquement dans votre espace de travail Azure Machine Learning

Si vous préférez gérer vos expériences suivies dans un emplacement centralisé, vous pouvez définir MLflow Tracking pour effectuer le suivi **uniquement** dans votre espace de travail Azure Machine Learning. 

Ajoutez le code suivant dans votre script :

```python
uri = ws.get_mlflow_tracking_uri()
mlflow.set_tracking_uri(uri)
```

Dans votre script d’apprentissage, importez `mlflow` pour utiliser les API de journalisation MLflow et commencez à journaliser vos métriques d’exécution. L'exemple suivant journalise la métrique epoch loss. 

```python
import mlflow 
mlflow.log_metric('epoch_loss', loss.item()) 
```

## <a name="register-models-with-mlflow"></a>Inscrire des modèles avec MLflow

Au terme de l'apprentissage de votre modèle, vous pouvez le consigner et l'inscrire sur le serveur de suivi principal avec la méthode `mlflow.<model_flavor>.log_model()`. `<model_flavor>` fait référence au framework associé au modèle. [Découvrez les versions du modèle prises en charge](https://mlflow.org/docs/latest/models.html#model-api). 

Par défaut, le serveur de suivi principal est l'espace de travail Azure Databricks ; sauf si vous avez choisi de [configurer MLflow Tracking pour limiter le suivi à votre espace de travail Azure Machine Learning](#set-mlflow-tracking-to-only-track-in-your-azure-machine-learning-workspace), auquel cas le serveur de suivi principal est l'espace de travail Azure Machine Learning.   

* **S'il n'existe pas de modèle inscrit sous ce nom**, la méthode en inscrit un nouveau, crée une Version 1, puis renvoie un objet MLflow ModelVersion. 

* **S'il existe déjà un modèle inscrit sous ce nom**, la méthode crée une nouvelle version du modèle et renvoie l'objet version. 

```python
mlflow.spark.log_model(model, artifact_path = "model", 
                       registered_model_name = 'model_name')  

mlflow.sklearn.log_model(model, artifact_path = "model", 
                         registered_model_name = 'model_name') 
```

## <a name="create-endpoints-for-mlflow-models"></a>Créer des points de terminaison pour les modèles MLflow

Lorsque vous êtes prêt à créer un point de terminaison pour vos modèles ML : vous pouvez effectuer un déploiement en tant que : 

* Service web de requête-réponse Azure Machine Learning pour le scoring interactif. Ce déploiement vous permet de tirer parti des capacités de gestion de modèles et de détection des dérives de données d'Azure Machine Learning et de les appliquer à vos modèles de production. 

* Objets du modèle MLFlow, qui peuvent être utilisés dans des pipelines de diffusion en continu ou de traitement par lot en tant que fonctions Python ou fonctions Pandas définies par l'utilisateur dans l'espace de travail Azure Databricks.

### <a name="deploy-models-to-azure-machine-learning-endpoints"></a>Déployer des modèles sur des points de terminaison Azure Machine Learning 
Vous pouvez utiliser l'API [mlflow.azureml.deploy](https://www.mlflow.org/docs/latest/python_api/mlflow.azureml.html#mlflow.azureml.deploy) pour déployer un modèle dans votre espace de travail Azure Machine Learning. Si vous avez uniquement inscrit le modèle dans l'espace de travail Azure Databricks, comme décrit dans la section [Inscrire des modèles avec MLflow](#register-models-with-mlflow), spécifiez le paramètre `model_name` afin d'inscrire le modèle dans l'espace de travail Azure Machine Learning. 

Les exécutions Azure Databricks peuvent être déployées sur les points de terminaison suivants : 
* [Azure Container Instance](how-to-deploy-models-with-mlflow.md#deploy-to-aci)
* [Azure Kubernetes Service](how-to-deploy-models-with-mlflow.md#deploy-to-aks)

### <a name="deploy-models-to-adb-endpoints-for-batch-scoring"></a>Déployer des modèles sur des points de terminaison ADB pour le scoring par lot 

Vous pouvez choisir des clusters Azure Databricks pour le scoring par lot. Le modèle MLFlow est chargé et utilisé en tant que fonction Spark Pandas définie par l'utilisateur pour calculer le score des nouvelles données. 

```python
from pyspark.sql.types import ArrayType, FloatType 

model_uri = "runs:/"+last_run_id+ {model_path} 

#Create a Spark UDF for the MLFlow model 

pyfunc_udf = mlflow.pyfunc.spark_udf(spark, model_uri) 

#Load Scoring Data into Spark Dataframe 

scoreDf = spark.table({table_name}).where({required_conditions}) 


#Make Prediction 

preds = (scoreDf 

           .withColumn('target_column_name', pyfunc_udf('Input_column1', 'Input_column2', ' Input_column3', …)) 

        ) 

display(preds) 
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne prévoyez pas d’utiliser les métriques et artefacts enregistrés dans votre espace de travail, la possibilité de les supprimer individuellement est actuellement indisponible. Au lieu de cela, supprimez le groupe de ressources contenant le compte de stockage et l’espace de travail afin d’éviter des frais supplémentaires :

1. Dans le portail Azure, sélectionnez **Groupes de ressources** tout à gauche.

   ![Supprimer dans le portail Azure](./media/how-to-use-mlflow-azure-databricks/delete-resources.png)

1. À partir de la liste, sélectionnez le groupe de ressources créé.

1. Sélectionnez **Supprimer le groupe de ressources**.

1. Entrez le nom du groupe de ressources. Puis sélectionnez **Supprimer**.

## <a name="example-notebooks"></a>Exemples de notebooks

Les [notebooks MLflow avec Azure Machine Learning](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/track-and-monitor-experiments/using-mlflow) illustrent et développent les concepts abordés dans cet article.

## <a name="next-steps"></a>Étapes suivantes

* [Gérez vos modèles](concept-model-management-and-deployment.md).
* [Assurez le suivi des exécutions d'expériences et créez des points de terminaison avec MLflow et Azure Machine Learning](how-to-use-mlflow.md). 
* Découvrez-en plus sur [Azure Databricks et MLflow](/azure/databricks/applications/mlflow/).