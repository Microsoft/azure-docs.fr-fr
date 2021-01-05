---
title: Développer avec autoML et Azure Databricks
titleSuffix: Azure Machine Learning
description: Apprenez à configurer un environnement de développement dans Azure Machine Learning et Azure Databricks. Utilisez les kits de développement logiciel (SDK) Azure ML pour Databricks et Databricks avec autoML.
services: machine-learning
author: rastala
ms.author: roastala
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.date: 10/21/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: ef8ee7718aabb443fda6cd7b276ee53472261913
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93423700"
---
# <a name="set-up-a-development-environment-with-azure-databricks-and-automl-in-azure-machine-learning"></a>Configurer un environnement de développement avec Azure Databricks et autoML dans Azure Machine Learning 

Découvrez comment configurer un environnement de développement dans Azure Machine Learning qui utilise des Azure Databricks et ML automatisé.

Azure Databricks est parfait pour l'exécution de flux de travail de Machine Learning intensifs à grande échelle sur la plateforme Apache Spark évolutive dans le cloud Azure. Il fournit un environnement basé sur Notebook collaboratif avec un cluster de calcul basé sur l’UC ou le GPU.

Pour plus d’informations sur les autres environnements de développement Machine Learning, consultez [Configurer l’environnement de développement Python](how-to-configure-environment.md).


## <a name="prerequisite"></a>Configuration requise

Espace de travail Azure Machine Learning. SI vous n’en avez pas, vous pouvez créer un espace de travail Azure Machine Learning via le [Portail Azure](how-to-manage-workspace.md), [Azure CLI](how-to-manage-workspace-cli.md#create-a-workspace) et les [modèles Azure Resource Manager](how-to-create-workspace-template.md).


## <a name="azure-databricks-with-azure-machine-learning-and-automl"></a>Azure Databricks avec Azure Machine Learning et autoML

Azure Databricks s’intègre à Azure Machine Learning et à ses fonctionnalités autoML. 

Vous pouvez utiliser Azure Databricks :

+ Pour l’apprentissage d’un modèle à l’aide de Spark MLlib et déployer le modèle sur ACI/AKS.
+ Avec les fonctionnalités de [Machine Learning automatisé](concept-automated-ml.md) utilisant un kit de développement logiciel (SDK) Azure ML.
+ Comme cible de calcul à partir d’un [pipeline Azure Machine Learning](concept-ml-pipelines.md).

## <a name="set-up-a-databricks-cluster"></a>Configurer un cluster Databricks

Créez un [cluster Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal). Certains paramètres s’appliquent uniquement si vous installez le Kit de développement logiciel (SDK) pour l’apprentissage automatique automatisé sur Databricks.

**La création du cluster ne prend que quelques minutes.**

Utilisez les paramètres suivants :

| Paramètre |S’applique à| Valeur |
|----|---|---|
| Nom du cluster |toujours| Nom de votre cluster |
| Version de Databricks Runtime |toujours| Runtime 7.1 (Scala 2.21, Spark 3.0.0) - Non ML|
| Version Python |toujours| 3 |
| Type de Worker <br>(détermine le nombre maximal d’itérations concurrentes) |ML automatisé<br>uniquement| Machine virtuelle à mémoire optimisé, de préférence |
| Workers |toujours| 2 ou plus |
| Activer la mise à l’échelle automatique |ML automatisé<br>uniquement| Décochez la case |

Attendez que le cluster s’exécute avant de continuer.

## <a name="add-the-azure-ml-sdk-to-databricks"></a>Ajouter le kit de développement logiciel (SDK) Azure ML à Databricks

Une fois que le cluster est en cours d’exécution, [créez une bibliothèque](https://docs.databricks.com/user-guide/libraries.html#create-a-library) pour associer le package du Kit de développement logiciel (SDK) Azure Machine Learning approprié à votre cluster. 

Pour utiliser le ML automatisé, passez à [Ajouter le kit de développement logiciel SDK Azure ML avec AutoML](#add-the-azure-ml-sdk-with-automl-to-databricks).


1. Cliquez avec le bouton de droite sur le dossier de l’espace de travail actuel dans lequel vous souhaitez stocker la bibliothèque. Sélectionnez **Créer** > **Bibliothèque**.
    
    > [!TIP]
    > Si vous avez une ancienne version du Kit de développement logiciel (SDK), désélectionnez-la des bibliothèques installées du cluster et déplacez-la vers la corbeille. Installez la nouvelle version du SDK et redémarrez le cluster. En cas de problème après le redémarrage, détachez et rattachez votre cluster.

1. Choisissez l’option suivante (aucune autre installation de Kit de développement logiciel (SDK) n’est prise en charge)

   |Suppléments de &nbsp;packages&nbsp; de Kit de développement logiciel (SDK)|Source|Nom&nbsp;PyPi&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|
   |----|---|---|
   |Pour Databricks| Charger Python Egg ou PyPI | azureml-sdk[databricks]|

   > [!WARNING]
   > Aucun autre supplément de Kit de développement logiciel (SDK) ne peut être installé. Choisissez uniquement l’option [`databricks`].

   * Ne sélectionnez pas **Attacher automatiquement à tous les clusters**.
   * Sélectionnez **Attacher** en regard du nom de votre cluster.

1. Surveillez les erreurs jusqu’à ce que l’état soit défini sur **Attaché**, ce qui peut prendre plusieurs minutes.  Si cette étape échoue :

   Essayez de redémarrer votre cluster en procédant comme suit :
   1. Dans le volet gauche, sélectionnez **Clusters**.
   1. Sélectionnez le nom de votre cluster dans le tableau.
   1. Sous l’onglet **Bibliothèques**, sélectionnez **Redémarrer**.

   Une installation réalisée correctement ressemble à ceci : 

  ![Kit de développement logiciel (SDK) Azure Machine Learning pour Databricks](./media/how-to-configure-environment/amlsdk-withoutautoml.jpg) 

## <a name="add-the-azure-ml-sdk-with-automl-to-databricks"></a>Ajouter le kit de développement logiciel (SDK) Azure ML avec AutoML à Databricks
Si le cluster a été créé avec Databricks Runtime 7.1 ou une version ultérieure (*non* ML), exécutez la commande suivante dans la première cellule de votre notebook pour installer le Kit SDK AML.

```
%pip install --upgrade --force-reinstall -r https://aka.ms/automl_linux_requirements.txt
```
Pour le runtime 7.0 de Databricks et les versions inférieures, installez le Kit de développement logiciel (SDK) Azure Machine Learning à l’aide du [script init](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/azure-databricks/automl/README.md).

### <a name="automl-config-settings"></a>Paramètres de configuration AutoML

Dans la configuration d'AutoML, lorsque vous utilisez Azure Databricks, ajoutez les paramètres suivants :

- ```max_concurrent_iterations``` est basé sur le nombre de nœuds Worker dans votre cluster.
- ```spark_context=sc``` est basé sur le contexte spark par défaut.

## <a name="ml-notebooks-that-work-with-azure-databricks"></a>Notebooks ML compatible avec Azure Databricks

Lancez-vous :
+ Bien que de nombreux exemples de notebooks soient disponibles, **seuls [ ces exemples](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/azure-databricks) fonctionnent avec Azure Databricks.**

+ Importez ces exemples directement à partir de votre espace de travail. Voir ci-dessous : ![Sélectionner Importer](./media/how-to-configure-environment/azure-db-screenshot.png)
![Panneau Importer](./media/how-to-configure-environment/azure-db-import.png)

+ Découvrez comment [créer un pipeline avec Databricks en tant que cible de calcul de formation](how-to-create-your-first-pipeline.md).

## <a name="next-steps"></a>Étapes suivantes

- [Entraînez un modèle](tutorial-train-models-with-aml.md) sur Azure Machine Learning avec le jeu de données MNIST.
- Voir les [informations de référence sur le Kit de développement logiciel (SDK) Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py).
