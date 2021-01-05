---
title: Configurer des cibles de calcul pour l'apprentissage et l'inférence
titleSuffix: Azure Machine Learning
description: Ajoutez des ressources de calcul (cibles de calcul) à votre espace de travail pour l'apprentissage automatique et l'inférence
services: machine-learning
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.date: 10/02/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, contperf-fy21q1
ms.openlocfilehash: c25f3965775c6518629c92ccc371855d9178e648
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033712"
---
# <a name="set-up-compute-targets-for-model-training-and-deployment"></a>Configurer des cibles de calcul pour l'apprentissage et le déploiement de modèles

Apprenez à attacher des ressources de calcul Azure à votre espace de travail Azure Machine Learning.  Vous pourrez ensuite utiliser ces ressources comme [cibles de calcul](concept-compute-target.md) pour l'apprentissage et l'inférence dans vos tâches de Machine Learning.

Cet article explique comment configurer l'espace de travail pour utiliser ces ressources de calcul :

* Votre ordinateur local
* Machines virtuelles distantes
* Azure HDInsight
* Azure Batch
* Azure Databricks
* Service Analytique Azure Data Lake
* Azure Container Instance

Pour utiliser les cibles de calcul gérées par Azure Machine Learning service, consultez :


* [Instance de calcul Azure Machine Learning](how-to-create-manage-compute-instance.md)
* [Cluster de calcul Azure Machine Learning](how-to-create-attach-compute-cluster.md)
* [Cluster Azure Kubernetes Service](how-to-create-attach-kubernetes.md)

## <a name="prerequisites"></a>Prérequis

* Un espace de travail Azure Machine Learning. Pour plus d’informations, voir la page [Créer un espace de travail Azure Machine Learning](how-to-manage-workspace.md).

* L’[extension Azure CLI pour Machine Learning service](reference-azure-machine-learning-cli.md), le [SDK Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) ou l’[extension Azure Machine Learning pour Visual Studio Code](tutorial-setup-vscode-extension.md).

## <a name="limitations"></a>Limites

* **Évitez de créer plusieurs attachements en même temps dans le même calcul** depuis votre espace de travail. Par exemple, évitez d’attacher un cluster AKS à un espace de travail en utilisant deux noms différents. Chaque nouvel attachement va supprimer le ou les attachements précédents.

    Si vous souhaitez rattacher une cible de calcul, par exemple pour changer le protocole TLS ou un autre paramètre de configuration du cluster, vous devez d'abord supprimer l'attachement existant.

## <a name="whats-a-compute-target"></a>Qu’est-ce qu’une cible de calcul ?

Azure Machine Learning vous permet de faire l’apprentissage de votre modèle sur une variété de ressources ou d’environnements, appelés collectivement [__cibles de calcul__](concept-azure-machine-learning-architecture.md#compute-targets). Une cible de calcul peut être un ordinateur local ou une ressource cloud, comme une capacité de calcul Azure Machine Learning, Azure HDInsight ou une machine virtuelle distante.  Vous pouvez également utiliser des cibles de calcul pour le déploiement de modèles, comme décrit dans [« Déployer des modèles avec le service Azure Machine Learning »](how-to-deploy-and-where.md).


## <a name="local-computer"></a><a id="local"></a>Ordinateur local

Lorsque vous utilisez votre ordinateur local pour l’**entraînement**, il n’est pas nécessaire de créer une cible de calcul.  Il vous suffit de [soumettre l’exécution d’entraînement](how-to-set-up-training-targets.md) à partir de votre ordinateur local.

Lorsque vous utilisez votre ordinateur local pour l’**inférence**, Docker doit être installé. Pour effectuer le déploiement, utilisez [LocalWebservice.deploy_configuration()](/python/api/azureml-core/azureml.core.webservice.local.localwebservice?preserve-view=true&view=azure-ml-py#deploy-configuration-port-none-) pour définir le port que le service web devra utiliser. Utilisez ensuite la procédure de déploiement normale décrite dans [Déployer des modèles avec Azure Machine Learning](how-to-deploy-and-where.md).

## <a name="remote-virtual-machines"></a><a id="vm"></a>Machines virtuelles distantes

Azure Machine Learning prend également en charge l’utilisation de votre propre ressource de calcul et son attachement à votre espace de travail. Un tel type de ressource est une machine virtuelle distante arbitraire tant qu’elle est accessible depuis Azure Machine Learning. Il peut s’agir d’une machine virtuelle Azure, d’un serveur distant dans votre organisation, ou encore d’un serveur local. En particulier, avec l’adresse IP et les informations d’identification (nom d’utilisateur/mot de passe ou clé SSH), vous pouvez utiliser n’importe quelle machine virtuelle accessible pour les exécutions à distance.

Vous pouvez utiliser un environnement Conda intégré au système, un environnement Python déjà existant ou un conteneur Docker. Pour exécuter sur un conteneur Docker, vous devez disposer d’un moteur Docker en cours d’exécution sur la machine virtuelle. Cette fonctionnalité est particulièrement pratique quand vous voulez obtenir un environnement cloud de développement/expérience plus flexible que votre ordinateur local.

Pour ce scénario, utilisez Azure Data Science Virtual Machine (DSVM) en tant que machine virtuelle Azure. Cette machine virtuelle est un environnement de science des données et de développement d’intelligence artificielle préconfiguré dans Azure. La machine virtuelle offre un choix organisé d’outils et d’infrastructures pour le développement de l’apprentissage automatique en cycle de vie complet. Pour plus d’informations sur l’utilisation de la DSVM avec Azure Machine Learning, consultez [Configurer un environnement de développement](./how-to-configure-environment.md#dsvm).

1. **Créer** : Créez une Data Science Virtual Machine (DSVM) à utiliser pour former votre modèle. Pour savoir comment créer cette ressource, voir [Approvisionner une machine virtuelle pour la science des données pour Linux (Ubuntu)](./data-science-virtual-machine/dsvm-ubuntu-intro.md).

    > [!WARNING]
    > Azure Machine Learning prend uniquement en charge les machines virtuelles exécutant **Ubuntu**. Lorsque vous créez une machine virtuelle ou en choisissez une existante, celle-ci doit utiliser Ubuntu.
    > 
    > Azure Machine Learning nécessite également que la machine virtuelle dispose d’une __adresse IP publique__.

1. **Attacher** : Pour attacher une machine virtuelle existante en tant que cible de calcul, vous devez fournir l'ID de ressource, le nom d'utilisateur et le mot de passe de la machine virtuelle. L'ID de ressource de la machine virtuelle peut être construit à l'aide de l'ID d’abonnement, du nom du groupe de ressources et du nom de la machine virtuelle en utilisant le format de chaîne suivant : `/subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Compute/virtualMachines/<vm_name>`

 
   ```python
   from azureml.core.compute import RemoteCompute, ComputeTarget

   # Create the compute config 
   compute_target_name = "attach-dsvm"
   
   attach_config = RemoteCompute.attach_configuration(resource_id='<resource_id>',
                                                   ssh_port=22,
                                                   username='<username>',
                                                   password="<password>")

   # Attach the compute
   compute = ComputeTarget.attach(ws, compute_target_name, attach_config)

   compute.wait_for_completion(show_output=True)
   ```

   Vous pouvez également attacher la machine DSVM (Data Science Virtual Machine) à votre espace de travail [à l’aide d’Azure Machine Learning Studio](how-to-create-attach-compute-studio.md#attached-compute).

    > [!WARNING]
    > Ne créez pas plusieurs attachements en même temps dans la même machine virtuelle DSVM depuis votre espace de travail. Chaque nouvel attachement va supprimer le ou les attachements précédents.

1. **Configurer** : Créez une configuration de série de tests pour la cible de calcul Data Science Virtual Machine (DSVM). Docker et Conda sont utilisés pour créer et configurer l’environnement d’entraînement sur la DSVM.

   ```python
   from azureml.core import ScriptRunConfig
   from azureml.core.environment import Environment
   from azureml.core.conda_dependencies import CondaDependencies
   
   # Create environment
   myenv = Environment(name="myenv")
   
   # Specify the conda dependencies
   myenv.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'])
   
   # If no base image is explicitly specified the default CPU image "azureml.core.runconfig.DEFAULT_CPU_IMAGE" will be used
   # To use GPU in DSVM, you should specify the default GPU base Docker image or another GPU-enabled image:
   # myenv.docker.enabled = True
   # myenv.docker.base_image = azureml.core.runconfig.DEFAULT_GPU_IMAGE
   
   # Configure the run configuration with the Linux DSVM as the compute target and the environment defined above
   src = ScriptRunConfig(source_directory=".", script="train.py", compute_target=compute, environment=myenv) 
   ```

## <a name="azure-hdinsight"></a><a id="hdinsight"></a>Azure HDInsight 

Azure HDInsight est une plateforme populaire pour l’analytique de Big Data. Elle fournit Apache Spark, que vous pouvez utiliser pour entraîner votre modèle.

1. **Créer** :  Créez le cluster HDInsight à utiliser pour former votre modèle. Pour créer un cluster Spark sur HDInsight, consultez [Créer un cluster Spark dans HDInsight](../hdinsight/spark/apache-spark-jupyter-spark-sql.md). 

    > [!WARNING]
    > Azure Machine Learning nécessite également que le cluster HDInsight dispose d’une __adresse IP publique__.

    Lors de la création du cluster, vous devez spécifier un nom d’utilisateur SSH et un mot de passe. Notez ces valeurs, car vous en aurez besoin lors de l’utilisation de HDInsight comme cible de calcul.
    
    Une fois le cluster créé, connectez-le au nom d’hôte \<clustername>-ssh.azurehdinsight.net, où \<clustername>est le nom que vous avez donné au cluster. 

1. **Attacher** : Pour attacher un cluster HDInsight en tant que cible de calcul, vous devez fournir l'ID de ressource, le nom d'utilisateur et le mot de passe du cluster HDInsight. L'ID de ressource du cluster HDInsight peut être construit à l'aide de l'ID d’abonnement, du nom du groupe de ressources et du nom du cluster HDInsight en utilisant le format de chaîne suivant : `/subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.HDInsight/clusters/<cluster_name>`

    ```python
   from azureml.core.compute import ComputeTarget, HDInsightCompute
   from azureml.exceptions import ComputeTargetException

   try:
    # if you want to connect using SSH key instead of username/password you can provide parameters private_key_file and private_key_passphrase

    attach_config = HDInsightCompute.attach_configuration(resource_id='<resource_id>',
                                                          ssh_port=22, 
                                                          username='<ssh-username>', 
                                                          password='<ssh-pwd>')
    hdi_compute = ComputeTarget.attach(workspace=ws, 
                                       name='myhdi', 
                                       attach_configuration=attach_config)

   except ComputeTargetException as e:
    print("Caught = {}".format(e.message))

   hdi_compute.wait_for_completion(show_output=True)
   ```

   Vous pouvez également attacher le cluster HDInsight à votre espace de travail [à l’aide d’Azure Machine Learning Studio](how-to-create-attach-compute-studio.md#attached-compute).

    > [!WARNING]
    > Ne créez pas plusieurs attachements en même temps dans le même service HDInsight depuis votre espace de travail. Chaque nouvel attachement va supprimer le ou les attachements précédents.

1. **Configurer** : Créez une configuration de série de tests pour la cible de calcul HDI. 

   [!code-python[](~/aml-sdk-samples/ignore/doc-qa/how-to-set-up-training-targets/hdi.py?name=run_hdi)]


À présent que vous avez attaché la cible de calcul et configuré votre série de tests, l’étape suivante consiste à [soumettre la série de tests d’apprentissage](how-to-set-up-training-targets.md).

## <a name="azure-batch"></a><a id="azbatch"></a>Azure Batch 

Azure Batch sert à exécuter efficacement des applications de calcul haute performance (HPC) en parallèle et à grande échelle dans le cloud. AzureBatchStep peut être utilisé dans un pipeline Azure Machine Learning pour envoyer des travaux à un pool de machines Azure Batch.

Pour attacher Azure Batch comme cible de calcul, vous devez utiliser le Kit de développement logiciel Azure Machine Learning et fournir les informations suivantes :

-    **Nom de calcul Azure Batch** : Nom convivial à utiliser pour le calcul au sein de l’espace de travail
-    **Nom du compte Azure Batch** : Nom du compte Azure Batch
-    **Groupe de ressources** : Groupe de ressources qui contient le compte Azure Batch.

Le code suivant montre comment attacher Azure Batch comme cible de calcul :

```python
from azureml.core.compute import ComputeTarget, BatchCompute
from azureml.exceptions import ComputeTargetException

# Name to associate with new compute in workspace
batch_compute_name = 'mybatchcompute'

# Batch account details needed to attach as compute to workspace
batch_account_name = "<batch_account_name>"  # Name of the Batch account
# Name of the resource group which contains this account
batch_resource_group = "<batch_resource_group>"

try:
    # check if the compute is already attached
    batch_compute = BatchCompute(ws, batch_compute_name)
except ComputeTargetException:
    print('Attaching Batch compute...')
    provisioning_config = BatchCompute.attach_configuration(
        resource_group=batch_resource_group, account_name=batch_account_name)
    batch_compute = ComputeTarget.attach(
        ws, batch_compute_name, provisioning_config)
    batch_compute.wait_for_completion()
    print("Provisioning state:{}".format(batch_compute.provisioning_state))
    print("Provisioning errors:{}".format(batch_compute.provisioning_errors))

print("Using Batch compute:{}".format(batch_compute.cluster_resource_id))
```

> [!WARNING]
> Ne créez pas plusieurs attachements en même temps dans le même service Azure Batch depuis votre espace de travail. Chaque nouvel attachement va supprimer le ou les attachements précédents.

### <a name="azure-databricks"></a><a id="databricks"></a>Azure Databricks

Azure Databricks est un environnement basé sur Apache Spark dans le cloud Azure. Il peut être utilisé comme cible de calcul avec un pipeline Azure Machine Learning.

Créez un espace de travail Azure Databricks avant de l’utiliser. Pour créer une ressource d’espace de travail, consultez le document [Exécuter un travail Spark sur Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal).

Pour joindre Azure Databricks comme cible de calcul, fournissez les informations suivantes :

* __Nom de la capacité de calcul Databricks__ : nom que vous voulez affecter à cette ressource de calcul.
* __Nom de l’espace de travail Databricks__ : nom de l’espace de travail Azure Databricks.
* __Jeton d’accès Databricks__ : jeton d’accès utilisé pour s’authentifier auprès d’Azure Databricks. Pour générer un jeton d’accès, consultez le document [Authentification](https://docs.azuredatabricks.net/dev-tools/api/latest/authentication.html).

Le code suivant montre comment joindre Azure Databricks en tant que cible de calcul avec le kit de développement logiciel (SDK) Azure Machine Learning (__l’espace de travail Databricks doit être présent dans le même abonnement que votre espace de travail AML__) :

```python
import os
from azureml.core.compute import ComputeTarget, DatabricksCompute
from azureml.exceptions import ComputeTargetException

databricks_compute_name = os.environ.get(
    "AML_DATABRICKS_COMPUTE_NAME", "<databricks_compute_name>")
databricks_workspace_name = os.environ.get(
    "AML_DATABRICKS_WORKSPACE", "<databricks_workspace_name>")
databricks_resource_group = os.environ.get(
    "AML_DATABRICKS_RESOURCE_GROUP", "<databricks_resource_group>")
databricks_access_token = os.environ.get(
    "AML_DATABRICKS_ACCESS_TOKEN", "<databricks_access_token>")

try:
    databricks_compute = ComputeTarget(
        workspace=ws, name=databricks_compute_name)
    print('Compute target already exists')
except ComputeTargetException:
    print('compute not found')
    print('databricks_compute_name {}'.format(databricks_compute_name))
    print('databricks_workspace_name {}'.format(databricks_workspace_name))
    print('databricks_access_token {}'.format(databricks_access_token))

    # Create attach config
    attach_config = DatabricksCompute.attach_configuration(resource_group=databricks_resource_group,
                                                           workspace_name=databricks_workspace_name,
                                                           access_token=databricks_access_token)
    databricks_compute = ComputeTarget.attach(
        ws,
        databricks_compute_name,
        attach_config
    )

    databricks_compute.wait_for_completion(True)
```

Si vous souhaitez obtenir un exemple plus détaillé, veuillez consulter un [exemple de notebook](https://aka.ms/pl-databricks) sur GitHub.

> [!WARNING]
> Ne créez pas plusieurs attachements en même temps dans le même service Azure Databricks depuis votre espace de travail. Chaque nouvel attachement va supprimer le ou les attachements précédents.

### <a name="azure-data-lake-analytics"></a><a id="adla"></a>Azure Data Lake Analytics

Azure Data Lake Analytics est une plateforme analytique de Big Data dans le cloud Azure. Il peut être utilisé comme cible de calcul avec un pipeline Azure Machine Learning.

Créez un compte Azure Data Lake Analytics avant de l’utiliser. Pour créer cette ressource, consultez le document [Prise en main d’Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md).

Pour attacher Data Lake Analytics comme cible de calcul, vous devez utiliser le Kit de développement logiciel Azure Machine Learning et fournir les informations suivantes :

* __Nom de la capacité de calcul__ : nom que vous voulez affecter à cette ressource de calcul.
* __Groupe de ressources__ : groupe de ressources contenant le compte Data Lake Analytics.
* __Nom du compte__ : Le nom du compte Data Lake Analytics.

Le code suivant montre comment attacher Data Lake Analytics comme cible de calcul :

```python
import os
from azureml.core.compute import ComputeTarget, AdlaCompute
from azureml.exceptions import ComputeTargetException


adla_compute_name = os.environ.get(
    "AML_ADLA_COMPUTE_NAME", "<adla_compute_name>")
adla_resource_group = os.environ.get(
    "AML_ADLA_RESOURCE_GROUP", "<adla_resource_group>")
adla_account_name = os.environ.get(
    "AML_ADLA_ACCOUNT_NAME", "<adla_account_name>")

try:
    adla_compute = ComputeTarget(workspace=ws, name=adla_compute_name)
    print('Compute target already exists')
except ComputeTargetException:
    print('compute not found')
    print('adla_compute_name {}'.format(adla_compute_name))
    print('adla_resource_id {}'.format(adla_resource_group))
    print('adla_account_name {}'.format(adla_account_name))
    # create attach config
    attach_config = AdlaCompute.attach_configuration(resource_group=adla_resource_group,
                                                     account_name=adla_account_name)
    # Attach ADLA
    adla_compute = ComputeTarget.attach(
        ws,
        adla_compute_name,
        attach_config
    )

    adla_compute.wait_for_completion(True)
```

Si vous souhaitez obtenir un exemple plus détaillé, veuillez consulter un [exemple de notebook](https://aka.ms/pl-adla) sur GitHub.

> [!WARNING]
> Ne créez pas plusieurs attachements en même temps dans le même service Azure Data Lake Analytics depuis votre espace de travail. Chaque nouvel attachement va supprimer le ou les attachements précédents.

> [!TIP]
> Les pipelines Azure Machine Learning peuvent uniquement fonctionner avec les données stockées dans le magasin de données par défaut du compte Data Lake Analytics. Si les données dont vous avez besoin se trouvent dans un magasin non défini par défaut, vous pouvez utiliser un [`DataTransferStep`](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.data_transfer_step.datatransferstep?preserve-view=true&view=azure-ml-py) pour copier les données avant l’apprentissage.

## <a name="azure-container-instance"></a><a id="aci"></a>Azure Container Instance

Des instances Azure Container Instances (ACI) sont créées de façon dynamique lorsque vous déployez un modèle. Vous ne pouvez pas créer ou attacher des ACI à votre espace de travail de quelque autre manière que ce soit. Pour plus d’informations, consultez [Déployer un modèle dans Azure Container Instances](how-to-deploy-azure-container-instance.md).

## <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Azure Kubernetes Service (AKS) propose différentes options de configuration lorsqu’il est utilisé conjointement à Azure Machine Learning. Pour plus d’informations, consultez [Créer et attacher Azure Kubernetes Service](how-to-create-attach-kubernetes.md).


## <a name="notebook-examples"></a>Exemples de notebooks

Pour des exemples d’apprentissage avec différentes cibles de calcul, voir les blocs-notes suivants :
* [how-to-use-azureml/training](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training)
* [tutorials/img-classification-part1-training.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/image-classification-mnist-data/img-classification-part1-training.ipynb)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>Étapes suivantes

* Utilisez la ressource de calcul pour [configurer et soumettre une exécution d'entraînement](how-to-set-up-training-targets.md).
* [Tutoriel : Former un modèle](tutorial-train-models-with-aml.md) utilise une cible de calcul gérée pour former un modèle.
* Découvrez comment [optimiser efficacement les hyperparamètres](how-to-tune-hyperparameters.md) afin de générer des modèles plus efficaces.
* Une fois le modèle formé, découvrez [comment et où déployer les modèles](how-to-deploy-and-where.md).
* [Utiliser Azure Machine Learning avec des réseaux virtuels Azure](./how-to-network-security-overview.md)