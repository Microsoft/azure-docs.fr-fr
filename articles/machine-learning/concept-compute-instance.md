---
title: Qu’est-ce qu’une instance de calcul Azure Machine Learning ?
titleSuffix: Azure Machine Learning
description: En savoir plus sur l’instance de calcul Azure Machine Learning, une station de travail informatique entièrement managée.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sgilley
author: sdgilley
ms.date: 10/02/2020
ms.openlocfilehash: 7d07dda7741aa075b8d853a8d06d3d3218ca901a
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94563815"
---
# <a name="what-is-an-azure-machine-learning-compute-instance"></a>Qu’est-ce qu’une instance de calcul Azure Machine Learning ?

Une instance de calcul Azure Machine Learning est une station de travail cloud managée pour les scientifiques des données.

Les instances de calcul facilitent le démarrage du développement avec Azure Machine Learning. Elles fournissent aux administrateurs informatiques des fonctionnalités de gestion et de préparation de l’entreprise.  

Utilisez une instance de calcul comme environnement de développement complètement configuré et managé dans le cloud pour l’apprentissage automatique. Elle peut également être utilisée comme cible de calcul pour l’apprentissage et l’inférence à des fins de développement et de test.  

Pour un entraînement de modèle de niveau production, utilisez un [cluster de calcul Azure Machine Learning](how-to-create-attach-compute-cluster.md) avec des fonctionnalités de mise à l’échelle multinœud. Pour le déploiement d’un modèle de niveau production, utilisez un [cluster Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md).

Pour que la fonctionnalité d’instance de calcul Jupyter fonctionne, vérifiez que la communication avec le socket web n’est pas désactivée. Vérifiez que votre réseau autorise les connexions WebSocket à *.instances.azureml.net et *.instances.azureml.ms.

## <a name="why-use-a-compute-instance"></a>Pourquoi créer une instance de calcul ?

Une instance de calcul est une station de travail cloud complètement managée qui est optimisée pour votre environnement de développement Machine Learning. Elle vous permet de bénéficier des avantages suivants :

|Principaux avantages|Description|
|----|----|
|Productivité|Vous pouvez créer et déployer des modèles à l’aide des blocs-notes intégrés et des outils suivants dans Azure Machine Learning Studio :<br/>-  Jupyter<br/>-  JupyterLab<br/>-  RStudio (préversion)<br/>L’instance de calcul est entièrement intégrée avec l’espace de travail et le studio Azure Machine Learning. Vous pouvez partager des blocs-notes et des données avec d’autres scientifiques des données dans l’espace de travail.<br/> Vous pouvez également utiliser [VS Code](https://techcommunity.microsoft.com/t5/azure-ai/power-your-vs-code-notebooks-with-azml-compute-instances/ba-p/1629630) avec des instances de calcul.
|Managée et sécurisée|Réduisez votre empreinte de sécurité et ajoutez la conformité aux exigences de sécurité de l’entreprise. Les instances de calcul fournissent des stratégies de gestion robustes et des configurations de mise en réseau sécurisées telles que :<br/><br/>- Provisionnement automatique par les modèles Resource Manager ou le SDK Azure Machine Learning<br/>- [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md)<br/>- [Prise en charge des réseaux virtuels](./how-to-secure-training-vnet.md#compute-instance)<br/>- Stratégie SSH pour activer/désactiver l’accès SSH<br/>TLS 1.2 activé |
|Préconfiguré&nbsp;pour&nbsp;ML|Gagnez du temps sur les tâches d’installation grâce à des packages ML préconfigurés et à jour, des infrastructures de Deep Learning et des pilotes GPU.|
|Entièrement personnalisable|Les scénarios avancés deviennent un jeu d’enfant grâce à la prise en charge étendue des types de machines virtuelles Azure, y compris les GPU et la personnalisation de bas niveau persistante, comme l’installation de packages et de pilotes. |

Vous pouvez [créer une instance de calcul](how-to-create-manage-compute-instance.md?tabs=python#create) vous-même ou un administrateur peut [créer une instance de calcul pour vous](how-to-create-manage-compute-instance.md?tabs=python#create-on-behalf-of-preview).

## <a name="tools-and-environments"></a><a name="contents"></a>Outils et environnements

> [!IMPORTANT]
> Les éléments marqués (préversion) dans cet article sont actuellement en préversion publique.
> La préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail en production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

L’instance de calcul Azure Machine Learning vous permet de créer, d’effectuer l’apprentissage et de déployer des modèles dans une expérience de notebook entièrement intégrée dans votre espace de travail.

Vous pouvez exécuter des blocs-notes Jupyter dans [VS Code](https://techcommunity.microsoft.com/t5/azure-ai/power-your-vs-code-notebooks-with-azml-compute-instances/ba-p/1629630) à l’aide de l’instance de calcul en tant que serveur distant sans avoir besoin de SSH. Vous pouvez également activer l’intégration de VS Code via l’[extension SSH à distance](https://devblogs.microsoft.com/python/enhance-your-azure-machine-learning-experience-with-the-vs-code-extension/).

Vous pouvez [installer des packages](how-to-create-manage-compute-instance.md#install-packages) et [ajouter des noyaux](how-to-create-manage-compute-instance.md#add-new-kernels) à votre instance de calcul.  

Les outils et environnements suivants sont déjà installés sur l’instance de calcul : 

|Outils et environnements généraux|Détails|
|----|:----:|
|Pilotes|`CUDA`</br>`cuDNN`</br>`NVIDIA`</br>`Blob FUSE` |
|Bibliothèque Intel MPI||
|Azure CLI ||
|Exemples Azure Machine Learning ||
|Docker||
|Nginx||
|NCCL 2.0 ||
|Protobuf|| 

|Outils et environnements **R**|Détails|
|----|:----:|
|Édition open source du serveur RStudio (préversion)||
|Noyau R||
|SDK Azure Machine Learning pour R|[azuremlsdk](https://azure.github.io/azureml-sdk-for-r/reference/index.html)</br>Exemples du Kit de développement logiciel (SDK)|

|Outils et environnements **PYTHON**|Détails|
|----|----|
|Anaconda Python||
|Jupyter et extensions||
|Jupyterlab et extensions||
[SDK Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py)</br>de PyPI|Comprend la plupart des packages supplémentaires azureml.  Pour afficher la liste complète, [ouvrez une fenêtre de terminal sur votre instance de calcul](how-to-run-jupyter-notebooks.md#terminal), puis exécutez <br/> `conda list -n azureml_py36 azureml*` |
|Autres packages PyPI|`jupytext`</br>`tensorboard`</br>`nbconvert`</br>`notebook`</br>`Pillow`|
|Packages Conda|`cython`</br>`numpy`</br>`ipykernel`</br>`scikit-learn`</br>`matplotlib`</br>`tqdm`</br>`joblib`</br>`nodejs`</br>`nb_conda_kernels`|
|Packages Deep learning|`PyTorch`</br>`TensorFlow`</br>`Keras`</br>`Horovod`</br>`MLFlow`</br>`pandas-ml`</br>`scrapbook`|
|Packages ONNX|`keras2onnx`</br>`onnx`</br>`onnxconverter-common`</br>`skl2onnx`</br>`onnxmltools`|
|Exemples de SDK Azure Machine Learning pour Python et R||

Les packages Python sont tous installés dans l’environnement **Python 3.6 – AzureML**.  

## <a name="accessing-files"></a>Accès aux fichiers

Les notebooks et les scripts R sont stockés dans le compte de stockage par défaut de votre espace de travail dans le partage de fichiers Azure.  Ces fichiers se trouvent dans le répertoire « Fichiers utilisateur ». Grâce à ce stockage, il est facile de partager des notebooks entre des instances de calcul. Le compte de stockage conserve également vos notebooks en toute sécurité lorsque vous arrêtez ou supprimez une instance de calcul.

Le compte de partage de fichiers Azure de votre espace de travail est monté en tant que lecteur sur l’instance de calcul. Ce lecteur est le répertoire de travail par défaut pour Jupyter, Jupyter Labs et RStudio. Cela signifie que les blocs-notes et les autres fichiers que vous créez dans Jupyter, JupyterLab ou RStudio sont automatiquement stockés dans le partage de fichiers, et disponibles pour être utilisés dans d’autres instances de calcul également.

Les fichiers qui se trouvent dans le partage de fichiers sont accessibles à partir de toutes les instances de calcul du même espace de travail. Toutes les modifications apportées à ces fichiers sur l’instance de calcul seront conservées de manière fiable dans le partage de fichiers.

Vous pouvez également cloner les exemples les plus récents d’Azure Machine Learning dans votre dossier dans le répertoire des fichiers utilisateur du partage de fichiers de l’espace de travail.

L’écriture de petits fichiers peut être plus lente sur des lecteurs réseau que sur le disque local de l’instance de calcul.  Si vous écrivez de nombreux petits fichiers, essayez d’utiliser un répertoire directement sur l’instance de calcul, par exemple un répertoire `/tmp`. Notez que ces fichiers ne seront pas accessibles à partir d’autres instances de calcul. 

Vous pouvez utiliser le répertoire `/tmp` sur l’instance de calcul pour vos données temporaires.  Toutefois, n’écrivez pas de fichiers de données volumineux sur le disque du système d’exploitation de l’instance de calcul.  Utilisez plutôt des [magasin de données](concept-azure-machine-learning-architecture.md#datasets-and-datastores). Si vous avez installé l’extension Git JupyterLab, cela peut également entraîner un ralentissement des performances de l’instance de calcul.

## <a name="managing-a-compute-instance"></a>Gestion d’une instance de calcul

Dans votre espace de travail dans Azure Machine Learning Studio, sélectionnez **Calcul**, puis sélectionnez **Instance de calcul** en haut.

![Gérer une instance de calcul](./media/concept-compute-instance/manage-compute-instance.png)

Vous pouvez effectuer les actions suivantes :

* [Créer une instance de calcul](#create). 
* Actualiser l’onglet des instances de calcul.
* Démarrer, arrêter et redémarrer une instance de calcul.  Vous payez l’instance chaque fois qu’elle s’exécute. Quand vous n’utilisez pas l’instance de calcul, arrêtez-la pour réduire les coûts. L’arrêt d’une instance de calcul a pour effet de la libérer. Ensuite, redémarrez-la quand vous en avez besoin. Veuillez noter que l’arrêt de l’instance de calcul met fin à la facturation des heures de calcul, mais que le disque, l’adresse IP publique et l’équilibreur de charge standard vous seront toujours facturés.
* Supprimer une instance de calcul.
* Filtrer la liste des instances de calcul pour afficher uniquement celles que vous avez créées.

Pour chaque instance de calcul de votre espace de travail que vous pouvez utiliser, vous pouvez :

* Accéder à Jupyter, JupyterLab et RStudio sur l’instance de calcul.
* Ajouter un accès SSH à une instance de calcul. L’accès SSH est désactivé par défaut, mais il peut être activé au moment de la création de l’instance de calcul. L’accès SSH s’effectue par le biais d’un mécanisme de clé publique/privée. L’onglet vous donne des informations sur la connexion SSH, telles que l’adresse IP, le nom d’utilisateur et le numéro de port.
* Obtenir des informations sur une instance de calcul spécifique, telles que l’adresse IP et la région.

Le [contrôle d'accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md) vous permet de contrôler les utilisateurs de l'espace de travail qui peuvent créer, supprimer, démarrer ou arrêter une instance de calcul. Tous les utilisateurs ayant les rôles Contributeur et Propriétaire dans l’espace de travail sont autorisés à créer, supprimer, démarrer, arrêter et redémarrer des instances de calcul dans tout l’espace de travail. Toutefois, seul le créateur d’une instance de calcul spécifique, ou l’utilisateur affecté si elle a été créée en son nom, est autorisé à accéder à Jupyter, JupyterLab et RStudio sur cette instance de calcul. Une instance de calcul est dédiée à un seul utilisateur disposant d’un accès racine et peut accéder au terminal par le biais de Jupyter/JupyterLab/RStudio. L'instance de calcul utilisera une connexion mono-utilisateur et toutes les actions utiliseront l'identité de cet utilisateur pour le contrôle d'accès en fonction du rôle Azure (Azure RBAC) et l'attribution des exécutions d'expériences. L’accès SSH est contrôlé par le biais d’un mécanisme de clé publique/privée.

Ces actions peuvent être contrôlées par Azure RBAC :
* *Microsoft.MachineLearningServices/workspaces/computes/read*
* *Microsoft.MachineLearningServices/workspaces/computes/write*
* *Microsoft.MachineLearningServices/workspaces/computes/delete*
* *Microsoft.MachineLearningServices/workspaces/computes/start/action*
* *Microsoft.MachineLearningServices/workspaces/computes/stop/action*
* *Microsoft.MachineLearningServices/workspaces/computes/restart/action*

### <a name="create-a-compute-instance"></a><a name="create"></a>Créer une instance de calcul

Dans votre espace de travail dans Azure Machine Learning Studio, [créez une instance de calcul](how-to-create-attach-compute-studio.md#compute-instance) à partir de la section **Calcul** ou de la section **Notebooks** lorsque vous êtes prêt à exécuter l’un de vos notebooks. 

Vous pouvez également créer une instance
* Directement à partir de l’[expérience de blocs-notes intégrés](tutorial-1st-experiment-sdk-setup.md#azure)
* Dans le portail Azure
* À partir d’un modèle Azure Resource Manager. Pour un exemple de modèle, consultez [Créer un modèle d’instance de calcul Azure Machine Learning](https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-compute-create-computeinstance).
* Avec le [Kit de développement logiciel (SDK) Azure Machine Learning](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/machine-learning/concept-compute-instance.md)
* À partir de l’[extension CLI pour Azure Machine Learning](reference-azure-machine-learning-cli.md#computeinstance)

Le quota de cœurs dédiés par région par famille de machine virtuelle et le quota régional total, qui s’appliquent à la création d’une instance de calcul, sont unifiés et partagés avec le quota de clusters de calcul d’entraînement Azure Machine Learning. L’arrêt de l’instance de calcul n’a pas pour effet de libérer le quota pour s’assurer que vous puissiez redémarrer l’instance de calcul.


### <a name="create-on-behalf-of-preview"></a>Créer au nom de (préversion)

En tant qu’administrateur, vous pouvez créer une instance de calcul au nom d’un scientifique des données et lui affecter l’instance avec :
* [Modèle Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-compute-create-computeinstance)  Pour plus d’informations sur la façon de trouver les valeurs TenantID et ObjectID nécessaires dans ce modèle, consultez [Rechercher des ID d’objet d’identité pour la configuration de l’authentification](../healthcare-apis/find-identity-object-ids.md).  Vous pouvez également trouver ces valeurs dans le portail Azure Active Directory.
* API REST

Le Scientifique Données pour lequel vous créez l'instance de calcul doit disposer des autorisations Azure RBAC suivantes : 
* *Microsoft.MachineLearningServices/workspaces/computes/start/action*
* *Microsoft.MachineLearningServices/workspaces/computes/stop/action*
* *Microsoft.MachineLearningServices/workspaces/computes/restart/action*
* *Microsoft.MachineLearningServices/workspaces/computes/applicationaccess/action*

Le scientifique des données peut démarrer, arrêter et redémarrer l’instance de calcul. Il peut utiliser l’instance de calcul pour :
* Jupyter
* JupyterLab
* RStudio
* Notebooks intégrés

## <a name="compute-target"></a>Cible de calcul

Les instances de calcul peuvent être utilisées comme [cible de calcul d’entraînement](concept-compute-target.md#train), comme les clusters d’entraînement pour le calcul Azure Machine Learning. 

Une instance de calcul :
* a une file d’attente de travaux ;
* exécute des travaux en toute sécurité dans un environnement de réseau virtuel, sans qu’il soit nécessaire d’ouvrir un port SSH. Le travail s’exécute dans un environnement conteneurisé et empaquette les dépendances de votre modèle dans un conteneur Docker.
* peut exécuter plusieurs petits travaux en parallèle (préversion).  Deux travaux par cœur peuvent s’exécuter en parallèle, tandis que les autres travaux sont mis en file d’attente.
* Prend en charge les travaux d’entraînement distribués sur un seul nœud équipé de plusieurs GPU

Vous pouvez utiliser une instance de calcul en tant que cible de déploiement d’inférence locale dans des scénarios de test ou de débogage.


## <a name="what-happened-to-notebook-vm"></a><a name="notebookvm"></a>Qu’est-il arrivé à la machine virtuelle Notebook ?

Les instances de calcul remplacent la machine virtuelle Notebook.  

Tous les fichiers notebook stockés dans le partage de fichiers de l’espace de travail et les données dans les banques de données de l’espace de travail sont accessibles à partir d’une instance de calcul. Toutefois, tous les packages personnalisés qui ont été précédemment installés sur une machine virtuelle de notebooks devront être réinstallés sur l’instance de calcul. Les limitations de quota qui s’appliquent à la création des clusters de calcul s’appliquent aussi à la création des instances de calcul.

Impossible de créer des machines virtuelles de notebooks. Toutefois, vous pouvez toujours accéder aux machines virtuelles Notebook que vous avez créées et les utiliser, avec toutes leurs fonctionnalités. Des instances de calcul peuvent être créées dans le même espace de travail que les machines virtuelles Notebook existantes.


## <a name="next-steps"></a>Étapes suivantes

* [Créer et gérer une instance de calcul](how-to-create-manage-compute-instance.md)
* [Tutoriel : Effectuer l’apprentissage de votre premier modèle ML](tutorial-1st-experiment-sdk-train.md) montre comment utiliser une instance de calcul avec un notebook intégré.
