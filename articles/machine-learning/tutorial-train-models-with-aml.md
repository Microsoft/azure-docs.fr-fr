---
title: 'Tutoriel sur la classification d’images : Entraîner des modèles'
titleSuffix: Azure Machine Learning
description: Utilisez Azure Machine Learning pour entraîner un modèle de classification d’images avec scikit-learn dans un notebook Jupyter Notebook en Python. Ce tutoriel est la première partie d’une série de deux.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: sdgilley
ms.author: sgilley
ms.date: 09/28/2020
ms.custom: seodec18, devx-track-python
ms.openlocfilehash: d1dbe51dd095290c296699bbb4bc6bd3a8caf7bf
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96862426"
---
# <a name="tutorial-train-image-classification-models-with-mnist-data-and-scikit-learn"></a>Tutoriel : Entraîner des modèles de classification d’images avec des données MNIST et scikit-learn 


Dans ce tutoriel, vous allez entraîner un modèle Machine Learning sur des ressources de calcul distantes. Vous allez utiliser le workflow d’entraînement et de déploiement pour Azure Machine Learning dans un notebook Jupyter Notebook en Python.  Vous pourrez ensuite utiliser le bloc-notes en tant que modèle pour entraîner votre propre modèle Machine Learning avec vos propres données. Ce tutoriel est le **premier d’une série de deux**.  

Ce tutoriel montre comment effectuer l’entraînement d’une régression logistique simple à l’aide du jeu de données [MNIST](http://yann.lecun.com/exdb/mnist/) et de [scikit-learn](https://scikit-learn.org) avec Azure Machine Learning. MNIST est un jeu de données populaire composé de 70 000 images en nuances de gris. Chaque image est un chiffre manuscrit de 28 x 28 pixels, représentant un nombre de zéro à neuf. L’objectif est de créer un classifieur multiclasse pour identifier le chiffre représenté par une image donnée.

Découvrez comment effectuer les actions suivantes :

> [!div class="checklist"]
> * Configurer l’environnement de développement
> * Accéder aux données et les examiner
> * Effectuez l’apprentissage d’un modèle de régression logistique simple sur un cluster distant.
> * Examiner les résultats de l’entraînement et inscrire le meilleur modèle

Vous découvrirez comment sélectionner un modèle et le déployer dans la [deuxième partie de ce tutoriel](tutorial-deploy-models-with-aml.md).

Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree) dès aujourd’hui.

>[!NOTE]
> Le code présenté dans cet article a été testé avec le kit [SDK Azure Machine Learning](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) version 1.13.0.

## <a name="prerequisites"></a>Prérequis

* Suivre le [Tutoriel : Commencez à créer votre première expérience Azure ML](tutorial-1st-experiment-sdk-setup.md) pour :
    * Créer un espace de travail
    * Clonez le notebook des tutoriels dans votre dossier au sein de l’espace de travail.
    * Créez une instance de calcul basée sur le cloud.

* Dans votre dossier *tutorials/image-classification-mnist-data* cloné, ouvrez le notebook *img-classification-part1-training.ipynb*. 


Vous trouverez également le tutoriel et le fichier **utils.py** correspondant sur [GitHub](https://github.com/Azure/MachineLearningNotebooks/tree/master/tutorials) si vous souhaitez les utiliser dans votre propre [environnement local](how-to-configure-environment.md#local). Exécutez `pip install azureml-sdk[notebooks] azureml-opendatasets matplotlib` pour installer les dépendances de ce didacticiel.

> [!Important]
> Le reste de cet article contient le même contenu que ce que vous voyez dans le notebook.  
>
> Basculez maintenant vers le notebook Jupyter Notebook si vous voulez lire le code à mesure que vous l’exécutez. 
> Pour exécuter une seule cellule de code dans un notebook, cliquez sur celle-ci et appuyez sur **Maj + Entrée**. Sinon, exécutez l’intégralité du notebook en choisissant **Run all** (Tout exécuter) dans la barre d’outils supérieure.

## <a name="set-up-your-development-environment"></a><a name="start"></a>Configurer votre environnement de développement

Toute la configuration pour votre travail de développement peut être effectuée dans un bloc-notes Python. La configuration comprend les actions suivantes :

* Importer des packages Python
* Se connecter à un espace de travail, afin que votre ordinateur local puisse communiquer avec des ressources distantes
* Créer une expérience pour effectuer le suivi de toutes vos exécutions
* Créer une cible de calcul distante à utiliser pour l’entraînement

### <a name="import-packages"></a>Importer des packages

Importez les packages Python dont vous avez besoin dans cette session. Affichez également la version du SDK Azure Machine Learning :

```python
%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt

import azureml.core
from azureml.core import Workspace

# check core SDK version number
print("Azure ML SDK Version: ", azureml.core.VERSION)
```

### <a name="connect-to-a-workspace"></a>Se connecter à un espace de travail

Créez un objet d’espace de travail à partir de l’espace de travail existant. `Workspace.from_config()` lit le fichier **config.json** et charge les détails dans un objet nommé `ws` :

```python
# load workspace configuration from the config.json file in the current folder.
ws = Workspace.from_config()
print(ws.name, ws.location, ws.resource_group, sep='\t')
```

### <a name="create-an-experiment"></a>Créer une expérience

Créez une expérience pour effectuer le suivi des exécutions dans votre espace de travail. Un espace de travail peut avoir plusieurs expériences :

```python
from azureml.core import Experiment
experiment_name = 'sklearn-mnist'

exp = Experiment(workspace=ws, name=experiment_name)
```

### <a name="create-or-attach-an-existing-compute-target"></a>Créer ou attacher une cible de calcul existante

En utilisant le service managé Capacité de calcul Azure Machine Learning, les scientifiques des données peuvent entraîner des modèles Machine Learning sur des clusters de machines virtuelles Azure. Il peut s’agir notamment de machines virtuelles prenant en charge les GPU. Dans ce tutoriel, vous allez créer la Capacité de calcul Azure Machine Learning qui vous servira d’environnement d’entraînement. Vous soumettrez du code Python à exécuter sur cette machine virtuelle plus loin dans ce tutoriel. 

Le code ci-dessous crée automatiquement les clusters de calcul s’ils n’existent pas encore dans votre espace de travail. Il configure un cluster qui effectue un scale-down à 0 lorsqu’il n’est pas utilisé et peut effectuer un scale-up jusqu’à un maximum de 4 nœuds. 

 **La création de la cible de calcul prend environ cinq minutes.** Si la ressource de calcul se trouve déjà dans l’espace de travail, le code l’utilise et ignore le processus de création.

```python
from azureml.core.compute import AmlCompute
from azureml.core.compute import ComputeTarget
import os

# choose a name for your cluster
compute_name = os.environ.get("AML_COMPUTE_CLUSTER_NAME", "cpu-cluster")
compute_min_nodes = os.environ.get("AML_COMPUTE_CLUSTER_MIN_NODES", 0)
compute_max_nodes = os.environ.get("AML_COMPUTE_CLUSTER_MAX_NODES", 4)

# This example uses CPU VM. For using GPU VM, set SKU to STANDARD_NC6
vm_size = os.environ.get("AML_COMPUTE_CLUSTER_SKU", "STANDARD_D2_V2")


if compute_name in ws.compute_targets:
    compute_target = ws.compute_targets[compute_name]
    if compute_target and type(compute_target) is AmlCompute:
        print('found compute target. just use it. ' + compute_name)
else:
    print('creating a new compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size=vm_size,
                                                                min_nodes=compute_min_nodes,
                                                                max_nodes=compute_max_nodes)

    # create the cluster
    compute_target = ComputeTarget.create(
        ws, compute_name, provisioning_config)

    # can poll for a minimum number of nodes and for a specific timeout.
    # if no min node count is provided it will use the scale settings for the cluster
    compute_target.wait_for_completion(
        show_output=True, min_node_count=None, timeout_in_minutes=20)

    # For a more detailed view of current AmlCompute status, use get_status()
    print(compute_target.get_status().serialize())
```

Vous disposez désormais des packages et des ressources de calcul nécessaires pour entraîner un modèle dans le cloud. 

## <a name="explore-data"></a>Explorer des données

Avant d’entraîner un modèle, vous devez comprendre les données que vous utilisez pour l’entraîner. Dans cette section, vous allez découvrir comment :

* Télécharger le jeu de données MNIST
* Afficher des exemples d’images

### <a name="download-the-mnist-dataset"></a>Télécharger le jeu de données MNIST

Utilisez Azure Open Datasets pour récupérer les fichiers de données MNIST bruts. Les jeux de données [Azure Open Datasets](../open-datasets/overview-what-are-open-datasets.md) sont des jeux de données publics organisés que vous pouvez utiliser pour ajouter des fonctionnalités spécifiques à des scénarios à des solutions de Machine Learning afin d'obtenir des modèles plus précis. Chaque jeu de données a une classe correspondante, `MNIST` dans ce cas, pour récupérer les données de différentes façons.

Ce code récupère les données sous la forme d’un objet `FileDataset`, qui est une sous-classe de `Dataset`. Un `FileDataset` fait référence à des fichiers uniques ou multiples de différents formats dans vos magasins de fichiers ou vos URL publiques. La classe vous permet de télécharger ou de monter les fichiers dans votre calcul en créant une référence à l’emplacement de la source de données. En outre, vous inscrivez le jeu de données dans votre espace de travail pour faciliter la récupération pendant la formation.

Pour en savoir plus sur les jeux de données et leur utilisation dans le kit de développement logiciel, suivez le [guide pratique](how-to-create-register-datasets.md).

```python
from azureml.core import Dataset
from azureml.opendatasets import MNIST

data_folder = os.path.join(os.getcwd(), 'data')
os.makedirs(data_folder, exist_ok=True)

mnist_file_dataset = MNIST.get_file_dataset()
mnist_file_dataset.download(data_folder, overwrite=True)

mnist_file_dataset = mnist_file_dataset.register(workspace=ws,
                                                 name='mnist_opendataset',
                                                 description='training and test dataset',
                                                 create_new_version=True)
```

### <a name="display-some-sample-images"></a>Afficher des exemples d’images

Chargez les fichiers compressés dans des tableaux `numpy`. Utilisez ensuite `matplotlib` pour tracer 30 images aléatoires du jeu de données avec leurs étiquettes au-dessus. Cette étape nécessite une fonction `load_data` qui est incluse dans un fichier `utils.py`. Ce fichier est inclus dans l’exemple de dossier. Vérifiez qu’il se trouve dans le même dossier que ce bloc-notes. La fonction `load_data` analyse les fichiers compressés dans des tableaux Numpy.

```python
# make sure utils.py is in the same directory as this code
from utils import load_data
import glob


# note we also shrink the intensity values (X) from 0-255 to 0-1. This helps the model converge faster.
X_train = load_data(glob.glob(os.path.join(data_folder,"**/train-images-idx3-ubyte.gz"), recursive=True)[0], False) / 255.0
X_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-images-idx3-ubyte.gz"), recursive=True)[0], False) / 255.0
y_train = load_data(glob.glob(os.path.join(data_folder,"**/train-labels-idx1-ubyte.gz"), recursive=True)[0], True).reshape(-1)
y_test = load_data(glob.glob(os.path.join(data_folder,"**/t10k-labels-idx1-ubyte.gz"), recursive=True)[0], True).reshape(-1)


# now let's show some randomly chosen images from the traininng set.
count = 0
sample_size = 30
plt.figure(figsize=(16, 6))
for i in np.random.permutation(X_train.shape[0])[:sample_size]:
    count = count + 1
    plt.subplot(1, sample_size, count)
    plt.axhline('')
    plt.axvline('')
    plt.text(x=10, y=-10, s=y_train[i], fontsize=18)
    plt.imshow(X_train[i].reshape(28, 28), cmap=plt.cm.Greys)
plt.show()
```

Un échantillon aléatoire d’images affiche :

![Échantillon aléatoire d’images](./media/tutorial-train-models-with-aml/digits.png)

Vous avez maintenant une idée de l’aspect de ces images et du résultat de prédiction attendu.

## <a name="train-on-a-remote-cluster"></a>Effectuer l’entraînement sur un cluster distant

Pour cette tâche, vous envoyez le travail pour l’exécuter sur le cluster d’entraînement distant défini plus tôt.  Pour envoyer un travail, vous devez :
* Créer un répertoire
* Créer un script d’entraînement
* Créer une configuration d’exécution de script
* Envoi du travail

### <a name="create-a-directory"></a>Créer un répertoire

Créez un répertoire pour fournir le code nécessaire à partir de votre ordinateur à la ressource distante.

```python
import os
script_folder = os.path.join(os.getcwd(), "sklearn-mnist")
os.makedirs(script_folder, exist_ok=True)
```

### <a name="create-a-training-script"></a>Créer un script d’entraînement

Pour envoyer le travail au cluster, vous devez d’abord créer un script d’entraînement. Exécutez le code suivant pour créer le script d’entraînement nommé `train.py` dans le répertoire que vous venez de créer.

```python
%%writefile $script_folder/train.py

import argparse
import os
import numpy as np
import glob

from sklearn.linear_model import LogisticRegression
import joblib

from azureml.core import Run
from utils import load_data

# let user feed in 2 parameters, the dataset to mount or download, and the regularization rate of the logistic regression model
parser = argparse.ArgumentParser()
parser.add_argument('--data-folder', type=str, dest='data_folder', help='data folder mounting point')
parser.add_argument('--regularization', type=float, dest='reg', default=0.01, help='regularization rate')
args = parser.parse_args()

data_folder = args.data_folder
print('Data folder:', data_folder)

# load train and test set into numpy arrays
# note we scale the pixel intensity values to 0-1 (by dividing it with 255.0) so the model can converge faster.
X_train = load_data(glob.glob(os.path.join(data_folder, '**/train-images-idx3-ubyte.gz'), recursive=True)[0], False) / 255.0
X_test = load_data(glob.glob(os.path.join(data_folder, '**/t10k-images-idx3-ubyte.gz'), recursive=True)[0], False) / 255.0
y_train = load_data(glob.glob(os.path.join(data_folder, '**/train-labels-idx1-ubyte.gz'), recursive=True)[0], True).reshape(-1)
y_test = load_data(glob.glob(os.path.join(data_folder, '**/t10k-labels-idx1-ubyte.gz'), recursive=True)[0], True).reshape(-1)

print(X_train.shape, y_train.shape, X_test.shape, y_test.shape, sep = '\n')

# get hold of the current run
run = Run.get_context()

print('Train a logistic regression model with regularization rate of', args.reg)
clf = LogisticRegression(C=1.0/args.reg, solver="liblinear", multi_class="auto", random_state=42)
clf.fit(X_train, y_train)

print('Predict the test set')
y_hat = clf.predict(X_test)

# calculate accuracy on the prediction
acc = np.average(y_hat == y_test)
print('Accuracy is', acc)

run.log('regularization rate', np.float(args.reg))
run.log('accuracy', np.float(acc))

os.makedirs('outputs', exist_ok=True)
# note file saved in the outputs folder is automatically uploaded into experiment record
joblib.dump(value=clf, filename='outputs/sklearn_mnist_model.pkl')
```

Notez comment le script obtient les données et enregistre les modèles :

+ Le script d’entraînement lit un argument pour rechercher le répertoire qui contient les données. Quand vous envoyez le travail ultérieurement, vous pointez vers la banque de données pour cet argument : ```parser.add_argument('--data-folder', type=str, dest='data_folder', help='data directory mounting point')```

+ Le script d’entraînement enregistre votre modèle dans un répertoire nommé **outputs**. Tout ce qui est écrit dans ce répertoire est chargé automatiquement dans votre espace de travail. Vous accédez à votre modèle à partir de ce répertoire plus loin dans le tutoriel. `joblib.dump(value=clf, filename='outputs/sklearn_mnist_model.pkl')`

+ Le script d’entraînement nécessite le fichier `utils.py` pour charger correctement le jeu de données. Le code suivant copie `utils.py` dans `script_folder`, afin que le fichier soit accessible en plus du script d’entraînement sur la ressource distante.

  ```python
  import shutil
  shutil.copy('utils.py', script_folder)
  ```

### <a name="configure-the-training-job"></a>Configurer le travail d’entraînement

Créez un objet [ScriptRunConfig](/python/api/azureml-core/azureml.core.scriptrunconfig?preserve-view=true&view=azure-ml-py) pour spécifier les détails de configuration de votre travail d’entraînement, y compris votre script d’entraînement, l’environnement à utiliser et la cible de calcul sur laquelle effectuer l’exécution. Configurez le ScriptRunConfig en spécifiant :

* Le répertoire qui contient vos scripts. Tous les fichiers dans ce répertoire sont chargés dans les nœuds de cluster pour l’exécution.
* La cible de calcul. Dans le cas présent, vous utilisez le cluster de calcul Azure Machine Learning que vous avez créé.
* Le nom du script d’entraînement, **train.py**.
* Un environnement qui contient les bibliothèques nécessaires à l’exécution du script.
* Arguments nécessaires pour le script d’entraînement.

Dans ce tutoriel, cette cible est AmlCompute. Tous les fichiers du dossier de scripts sont chargés dans les nœuds de cluster pour l’exécution. **--data_folder** est configuré pour utiliser le jeu de données.

Pour commencer, créez l’environnement qui contient la bibliothèque scikit-learn, le package azureml-dataset-runtime nécessaire pour accéder au jeu de données et azureml-defaults, qui contient les dépendances pour les métriques de journalisation. Par ailleurs, azureml-defaults contient les dépendances nécessaires au déploiement du modèle en tant que service web dans la deuxième partie de ce tutoriel.

Une fois l’environnement défini, inscrivez-le dans de l’espace de travail pour le réutiliser dans la deuxième partie du tutoriel.

```python
from azureml.core.environment import Environment
from azureml.core.conda_dependencies import CondaDependencies

# to install required packages
env = Environment('tutorial-env')
cd = CondaDependencies.create(pip_packages=['azureml-dataset-runtime[pandas,fuse]', 'azureml-defaults'], conda_packages=['scikit-learn==0.22.1'])

env.python.conda_dependencies = cd

# Register environment to re-use later
env.register(workspace=ws)
```

Ensuite, créez le ScriptRunConfig en spécifiant le script d’entraînement, la cible de calcul et l’environnement.

```python
from azureml.core import ScriptRunConfig

args = ['--data-folder', mnist_file_dataset.as_mount(), '--regularization', 0.5]

src = ScriptRunConfig(source_directory=script_folder,
                      script='train.py', 
                      arguments=args,
                      compute_target=compute_target,
                      environment=env)
```

### <a name="submit-the-job-to-the-cluster"></a>Envoyer le travail au cluster

Exécutez l’expérience en soumettant l’objet ScriptRunConfig :

```python
run = exp.submit(config=src)
run
```

L’appel étant asynchrone, il retourne un état **en cours de préparation** ou **en cours d’exécution** dès le démarrage du travail.

## <a name="monitor-a-remote-run"></a>Superviser une exécution à distance

Au total, la première exécution prend **environ 10 minutes**. Mais pour les exécutions suivantes, tant que les dépendances de script ne changent pas, la même image est réutilisée. La durée de démarrage du conteneur est donc beaucoup plus courte.

Ce qui se passe pendant que vous attendez :

- **Création d’images** : Une image Docker qui correspond à l’environnement Python spécifié par l’environnement Azure ML est créée. L’image est chargée dans l’espace de travail. La création et le chargement de l’image prennent **environ cinq minutes**.

  Cette étape se produit une fois pour chaque environnement Python, puisque le conteneur est mis en cache pour les exécutions suivantes. Lors de la création d’image, les journaux d’activité sont diffusés en continu vers l’historique d’exécutions. Vous pouvez superviser la progression de la création d’image à l’aide de ces journaux d’activité.

- **Mise à l’échelle** : si le cluster distant nécessite plus de nœuds pour l’exécution que la quantité disponible actuellement, des nœuds supplémentaires sont ajoutés automatiquement. En général, la mise à l’échelle prend **environ cinq minutes.**

- **En cours d’exécution** : Lors de cette étape, les fichiers et les scripts nécessaires sont envoyés à la cible de calcul. Ensuite, les banques de données sont montées ou copiées. Pour finir, **entry_script** est exécuté. Pendant que le travail s’exécute, **stdout** et le répertoire **./logs** sont diffusés en continu vers l’historique d’exécutions. Vous pouvez superviser la progression de l’exécution à l’aide de ces journaux d’activité.

- **Post-traitement** : le répertoire **./outputs** de l’exécution est copié dans l’historique des exécutions de votre espace de travail, afin que vous puissiez accéder à ces résultats.

Vous pouvez vérifier la progression d’un travail en cours d’exécution de plusieurs façons. Ce tutoriel utilise un widget Jupyter et une méthode `wait_for_completion`.

### <a name="jupyter-widget"></a>Widget Jupyter

Surveillez la progression de l’exécution avec un [widget Jupyter](/python/api/azureml-widgets/azureml.widgets?preserve-view=true&view=azure-ml-py). Comme l’envoi de l’exécution, le widget est asynchrone et fournit des mises à jour automatiques toutes les 10 à 15 secondes jusqu’à ce que la tâche soit terminée :

```python
from azureml.widgets import RunDetails
RunDetails(run).show()
```

À la fin de l’entraînement, le widget ressemblera à ceci :

![Widget de bloc-notes](./media/tutorial-train-models-with-aml/widget.png)

Si vous devez annuler une exécution, vous pouvez suivre [ces instructions](./how-to-manage-runs.md).

### <a name="get-log-results-upon-completion"></a>Obtenir des résultats du journal lors de l’achèvement

L’entraînement et la supervision du modèle se produisent en arrière-plan. Attendez que l’entraînement du modèle soit terminé avant d’exécuter davantage de code. Utilisez `wait_for_completion` pour indiquer quand l’entraînement du modèle est terminé :

```python
run.wait_for_completion(show_output=False)  # specify True for a verbose log
```

### <a name="display-run-results"></a>Afficher les résultats de l’exécution

Vous disposez maintenant d’un modèle entraîné sur un cluster distant. Récupérez la précision du modèle :

```python
print(run.get_metrics())
```

La sortie montre que le modèle distant a une précision de 0.9204 :

`{'regularization rate': 0.8, 'accuracy': 0.9204}`

Dans le tutoriel suivant, vous explorerez ce modèle plus en détail.

## <a name="register-model"></a>Inscrire le modèle

La dernière étape du script d’entraînement a écrit le fichier `outputs/sklearn_mnist_model.pkl` dans un répertoire nommé `outputs` sur la machine virtuelle du cluster où le travail est exécuté. `outputs` est un répertoire spécial, dans la mesure où tout son contenu est chargé automatiquement dans votre espace de travail. Ce contenu apparaît dans l’enregistrement d’exécution dans l’expérience sous votre espace de travail. Ainsi, le fichier de modèle est désormais disponible également dans votre espace de travail.

Vous pouvez voir les fichiers associés à cette exécution :

```python
print(run.get_file_names())
```

Inscrivez le modèle dans l’espace de travail afin de pouvoir (vous-même ou d’autres collaborateurs) interroger, examiner et déployer ultérieurement ce modèle :

```python
# register model
model = run.register_model(model_name='sklearn_mnist',
                           model_path='outputs/sklearn_mnist_model.pkl')
print(model.name, model.id, model.version, sep='\t')
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

Vous pouvez aussi supprimer uniquement le cluster de calcul Machine Learning. Toutefois, la mise à l’échelle automatique est activée et le minimal de cluster est égal à zéro. Par conséquent, cette ressource n’entraînera aucun frais de calcul supplémentaires quand elle ne sera pas en cours d’utilisation :

```python
# Optionally, delete the Azure Machine Learning Compute cluster
compute_target.delete()
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel sur Azure Machine Learning, vous avez utilisé Python pour les tâches suivantes :

> [!div class="checklist"]
> * Configurer l’environnement de développement
> * Accéder aux données et les examiner
> * Entraîner plusieurs modèles sur un cluster distant à l’aide de la bibliothèque de Machine Learning scikit-learn populaire
> * Examiner les détails de l’entraînement et inscrire le meilleur modèle

Vous êtes prêt à déployer ce modèle inscrit en suivant les instructions fournies dans la deuxième partie de cette série de tutoriels :

> [!div class="nextstepaction"]
> [Tutoriel 2 : Déployer des modèles](tutorial-deploy-models-with-aml.md)
