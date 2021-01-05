---
title: 'Tutoriel : Pipelines ML pour le scoring par lots'
titleSuffix: Azure Machine Learning
description: Dans ce tutoriel, vous créez un pipeline Machine Learning pour effectuer un scoring par lots. Concentrez-vous sur le machine learning plutôt que sur l’infrastructure et l’automatisation.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: lobrien
ms.author: laobri
ms.reviewer: laobri
ms.date: 10/13/2020
ms.custom: contperf-fy20q4, devx-track-python
ms.openlocfilehash: b0b415cce37e464abcba9fab5ad4c1196b1b2e1b
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033474"
---
# <a name="tutorial-build-an-azure-machine-learning-pipeline-for-batch-scoring"></a>Tutoriel : Créer un pipeline Azure Machine Learning pour le scoring par lots



Dans ce tutoriel avancé, vous allez découvrir comment créer un [pipeline Azure Machine Learning](concept-ml-pipelines.md) pour exécuter un travail de scoring par lots. Les pipelines Machine Learning optimisent votre workflow à divers niveaux : vitesse, portabilité et réutilisation. Ainsi, vous pouvez vous concentrer sur le Machine Learning, plutôt que sur l’infrastructure et l’automatisation. Après avoir créé et publié un pipeline, vous configurez un point de terminaison REST qui vous permet de déclencher ce pipeline à partir de n’importe quelle bibliothèque HTTP sur n’importe quelle plateforme. 

L’exemple utilise un modèle de réseau neuronal convolutif [Inception-V3](https://arxiv.org/abs/1512.00567) préentraîné implémenté dans Tensorflow pour classifier les images sans étiquette. 

Dans ce tutoriel, vous allez effectuer les tâches suivantes :

> [!div class="checklist"]
> * Configurer l’espace de travail 
> * Télécharger et stocker des exemples de données
> * Créer des objets de jeu de données pour extraire et sortir des données
> * Télécharger, préparer et inscrire le modèle dans votre espace de travail
> * Provisionner des cibles de calcul et créer un script de scoring
> * Utiliser la classe `ParallelRunStep` pour le calcul pour le scoring par mot asynchrone
> * Créer, exécuter et publier un pipeline
> * Activer un point de terminaison REST pour le pipeline

Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree) dès aujourd’hui.

## <a name="prerequisites"></a>Prérequis

* Si vous n’avez pas encore d’espace de travail Azure Machine Learning ou de machine virtuelle de notebook, effectuez la [Partie 1 du tutoriel d’installation](tutorial-1st-experiment-sdk-setup.md).
* Une fois le tutoriel d’installation effectué, utilisez le même serveur de notebooks pour ouvrir le notebook *tutorials/machine-learning-pipelines-advanced/tutorial-pipeline-batch-scoring-classification.ipynb*.

Si vous souhaitez suivre le tutoriel d’installation dans votre propre [environnement local](how-to-configure-environment.md#local), vous pouvez y accéder sur [GitHub](https://github.com/Azure/MachineLearningNotebooks/tree/master/tutorials). Exécutez `pip install azureml-sdk[notebooks] azureml-pipeline-core azureml-pipeline-steps pandas requests` pour vous procurer les packages requis.

## <a name="configure-workspace-and-create-a-datastore"></a>Configurer l’espace de travail et créer un magasin de données

Créez un objet d’espace de travail à partir de l’espace de travail Azure Machine Learning existant.

```python
from azureml.core import Workspace
ws = Workspace.from_config()
```

> [!IMPORTANT]
> Cet extrait de code s’attend à ce que la configuration de l’espace de travail soit enregistrée dans le répertoire actif ou son parent. Pour plus d’informations sur la création d’un espace de travail, consultez [Créer et gérer les espaces de travail Azure Machine Learning](how-to-manage-workspace.md). Pour plus d’informations sur l’enregistrement de la configuration dans un fichier, consultez [Créer un fichier de configuration d’espace de travail](how-to-configure-environment.md#workspace).

## <a name="create-a-datastore-for-sample-images"></a>Créer une banque de données pour des exemples d’images

Pour le compte `pipelinedata`, récupérez l’exemple de données publiques d’évaluation ImageNet à partir du conteneur d’objets blob public `sampledata`. Appelez `register_azure_blob_container()` pour rendre les données disponibles dans l’espace de travail sous le nom `images_datastore`. Définissez ensuite le magasin de données par défaut de l’espace de travail en tant que magasin de données de sortie. Utilisez le magasin de données de sortie pour effectuer un scoring de la sortie dans le pipeline.

Pour plus d’informations sur l’accès aux données, consultez [Guide pratique pour accéder aux données](./how-to-access-data.md).

```python
from azureml.core.datastore import Datastore

batchscore_blob = Datastore.register_azure_blob_container(ws, 
                      datastore_name="images_datastore", 
                      container_name="sampledata", 
                      account_name="pipelinedata", 
                      overwrite=True)

def_data_store = ws.get_default_datastore()
```

## <a name="create-dataset-objects"></a>Créer des objets de jeu de données

Durant la création de pipelines, les objets `Dataset` sont utilisés pour lire les données des banques de données d’espace de travail, et les objets `PipelineData` sont utilisés pour transférer les données intermédiaires entre les étapes de pipeline.

> [!Important]
> L’exemple de scoring par lots de ce tutoriel utilise une seule étape de pipeline. Dans les cas d’usage qui comportent plusieurs étapes, le flux classique inclut les étapes suivantes :
>
> 1. Utilisez des objets `Dataset` en tant qu’*entrées* pour extraire les données brutes, exécutez certaines transformations, puis effectuez la *sortie* d’un objet `PipelineData`.
>
> 2. Utilisez l’*objet de sortie* `PipelineData` de l’étape précédente en tant qu’*objet d’entrée*. Répétez cette opération pour les étapes suivantes.

Dans ce scénario, vous créez des objets `Dataset` qui correspondent aux répertoires de magasin de données pour les images d’entrée et les étiquettes de classification (valeurs de test y). Vous créez également un objet `PipelineData` pour les données de sortie du scoring par lots.

```python
from azureml.core.dataset import Dataset
from azureml.pipeline.core import PipelineData

input_images = Dataset.File.from_files((batchscore_blob, "batchscoring/images/"))
label_ds = Dataset.File.from_files((batchscore_blob, "batchscoring/labels/"))
output_dir = PipelineData(name="scores", 
                          datastore=def_data_store, 
                          output_path_on_compute="batchscoring/results")
```

Inscrivez les jeux de données dans l’espace de travail si vous voulez les réutiliser ultérieurement. Cette étape est facultative.

```python

input_images = input_images.register(workspace = ws, name = "input_images")
label_ds = label_ds.register(workspace = ws, name = "label_ds")
```

## <a name="download-and-register-the-model"></a>Télécharger et inscrire le modèle

Téléchargez le modèle Tensorflow préentraîné afin de l’utiliser pour le scoring par lots dans un pipeline. Commencez par créer un répertoire local où stocker le modèle. Téléchargez et extrayez ensuite le modèle.

```python
import os
import tarfile
import urllib.request

if not os.path.isdir("models"):
    os.mkdir("models")
    
response = urllib.request.urlretrieve("http://download.tensorflow.org/models/inception_v3_2016_08_28.tar.gz", "model.tar.gz")
tar = tarfile.open("model.tar.gz", "r:gz")
tar.extractall("models")
```

Inscrivez ensuite le modèle auprès de votre espace de travail pour pouvoir le récupérer facilement dans le processus de pipeline. Dans la fonction statique `register()`, le paramètre `model_name` représente la clé à utiliser pour localiser votre modèle dans le kit SDK.

```python
from azureml.core.model import Model
 
model = Model.register(model_path="models/inception_v3.ckpt",
                       model_name="inception",
                       tags={"pretrained": "inception"},
                       description="Imagenet trained tensorflow inception",
                       workspace=ws)
```

## <a name="create-and-attach-the-remote-compute-target"></a>Créer et attacher la cible de calcul distante

Vous ne pouvez pas exécuter les pipelines Machine Learning localement, donc exécutez-les sur des ressources cloud ou des *cibles de calcul distantes*. Une cible de calcul distante est un environnement Compute virtuel réutilisable où vous exécutez des expériences et des workflows Machine Learning. 

Exécutez le code suivant pour créer une cible [`AmlCompute`](/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute?preserve-view=true&view=azure-ml-py) ayant un GPU, puis attachez-la à votre espace de travail. Pour plus d’informations sur les cibles de calcul, consultez l’[article conceptuel](./concept-compute-target.md).


```python
from azureml.core.compute import AmlCompute, ComputeTarget
from azureml.exceptions import ComputeTargetException
compute_name = "gpu-cluster"

# checks to see if compute target already exists in workspace, else create it
try:
    compute_target = ComputeTarget(workspace=ws, name=compute_name)
except ComputeTargetException:
    config = AmlCompute.provisioning_configuration(vm_size="STANDARD_NC6",
                                                   vm_priority="lowpriority", 
                                                   min_nodes=0, 
                                                   max_nodes=1)

    compute_target = ComputeTarget.create(workspace=ws, name=compute_name, provisioning_configuration=config)
    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
```

## <a name="write-a-scoring-script"></a>Écrire un script de scoring

Pour effectuer le scoring, créez un script de scoring par lots nommé `batch_scoring.py` dans le répertoire actif. Le script accepte les images d’entrée, applique le modèle de classification, puis génère les prédictions dans un fichier de résultats.

Le script `batch_scoring.py` prend les paramètres suivants, qui sont passés à partir de `ParallelRunStep` que vous allez créer plus loin :

- `--model_name`: Nom du modèle utilisé.
- `--labels_dir`: Emplacement du fichier `labels.txt`.

L’infrastructure de pipeline utilise la classe `ArgumentParser` pour passer les paramètres aux étapes de pipeline. Par exemple, dans le code suivant, le premier argument `--model_name` reçoit l’identificateur de propriété `model_name`. Dans la fonction `init()`, `Model.get_model_path(args.model_name)` permet d’accéder à cette propriété.


```python
%%writefile batch_scoring.py

import os
import argparse
import datetime
import time
import tensorflow as tf
from math import ceil
import numpy as np
import shutil
from tensorflow.contrib.slim.python.slim.nets import inception_v3

from azureml.core import Run
from azureml.core.model import Model
from azureml.core.dataset import Dataset

slim = tf.contrib.slim

image_size = 299
num_channel = 3


def get_class_label_dict(labels_dir):
    label = []
    labels_path = os.path.join(labels_dir, 'labels.txt')
    proto_as_ascii_lines = tf.gfile.GFile(labels_path).readlines()
    for l in proto_as_ascii_lines:
        label.append(l.rstrip())
    return label


def init():
    global g_tf_sess, probabilities, label_dict, input_images

    parser = argparse.ArgumentParser(description="Start a tensorflow model serving")
    parser.add_argument('--model_name', dest="model_name", required=True)
    parser.add_argument('--labels_dir', dest="labels_dir", required=True)
    args, _ = parser.parse_known_args()

    label_dict = get_class_label_dict(args.labels_dir)
    classes_num = len(label_dict)

    with slim.arg_scope(inception_v3.inception_v3_arg_scope()):
        input_images = tf.placeholder(tf.float32, [1, image_size, image_size, num_channel])
        logits, _ = inception_v3.inception_v3(input_images,
                                              num_classes=classes_num,
                                              is_training=False)
        probabilities = tf.argmax(logits, 1)

    config = tf.ConfigProto()
    config.gpu_options.allow_growth = True
    g_tf_sess = tf.Session(config=config)
    g_tf_sess.run(tf.global_variables_initializer())
    g_tf_sess.run(tf.local_variables_initializer())

    model_path = Model.get_model_path(args.model_name)
    saver = tf.train.Saver()
    saver.restore(g_tf_sess, model_path)


def file_to_tensor(file_path):
    image_string = tf.read_file(file_path)
    image = tf.image.decode_image(image_string, channels=3)

    image.set_shape([None, None, None])
    image = tf.image.resize_images(image, [image_size, image_size])
    image = tf.divide(tf.subtract(image, [0]), [255])
    image.set_shape([image_size, image_size, num_channel])
    return image


def run(mini_batch):
    result_list = []
    for file_path in mini_batch:
        test_image = file_to_tensor(file_path)
        out = g_tf_sess.run(test_image)
        result = g_tf_sess.run(probabilities, feed_dict={input_images: [out]})
        result_list.append(os.path.basename(file_path) + ": " + label_dict[result[0]])
    return result_list
```

> [!TIP]
> Le pipeline de ce tutoriel comporte une seule étape et écrit la sortie dans un fichier. Pour les pipelines multiétapes, `ArgumentParser` vous permet également de définir un répertoire où écrire les données de sortie à entrer dans les étapes suivantes. Pour obtenir un exemple de passage de données entre plusieurs étapes de pipeline à l’aide du modèle de conception `ArgumentParser`, consultez le [notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/nyc-taxi-data-regression-model-building/nyc-taxi-data-regression-model-building.ipynb).

## <a name="build-the-pipeline"></a>Générer le pipeline

Avant d’exécuter le pipeline, créez un objet qui définit l’environnement Python et crée les dépendances nécessaires à votre script `batch_scoring.py`. La dépendance principale nécessaire est Tensorflow, mais vous installez également `azureml-core` et `azureml-dataprep[fuse]` exigés par ParallelRunStep. Spécifiez également la prise en charge de Docker et Docker-GPU.

```python
from azureml.core import Environment
from azureml.core.conda_dependencies import CondaDependencies
from azureml.core.runconfig import DEFAULT_GPU_IMAGE

cd = CondaDependencies.create(pip_packages=["tensorflow-gpu==1.15.2",
                                            "azureml-core", "azureml-dataprep[fuse]"])
env = Environment(name="parallelenv")
env.python.conda_dependencies = cd
env.docker.base_image = DEFAULT_GPU_IMAGE
```

### <a name="create-the-configuration-to-wrap-the-script"></a>Créer la configuration pour encapsuler le script

Créez l’étape du pipeline à l’aide du script, de la configuration de l’environnement et des paramètres. Spécifiez la cible de calcul que vous avez déjà attachée à votre espace de travail.

```python
from azureml.pipeline.steps import ParallelRunConfig

parallel_run_config = ParallelRunConfig(
    environment=env,
    entry_script="batch_scoring.py",
    source_directory=".",
    output_action="append_row",
    mini_batch_size="20",
    error_threshold=1,
    compute_target=compute_target,
    process_count_per_node=2,
    node_count=1
)
```

### <a name="create-the-pipeline-step"></a>Créer l’étape du pipeline

Une étape de pipeline est un objet qui encapsule tout ce dont vous avez besoin pour exécuter un pipeline, notamment :

* Paramètres d’environnement et de dépendance
* Ressource de calcul où exécuter le pipeline
* Données d’entrée et de sortie ainsi que tous les paramètres personnalisés
* Référence à un script ou une logique de kit SDK à exécuter au cours de l’étape

Plusieurs classes héritent de la classe parente [`PipelineStep`](/python/api/azureml-pipeline-core/azureml.pipeline.core.builder.pipelinestep?preserve-view=true&view=azure-ml-py). Vous pouvez choisir des classes pour utiliser des frameworks ou des piles spécifiques afin de créer une étape. Dans cet exemple, vous utilisez la classe `ParallelRunStep` pour définir la logique de votre étape en utilisant un script Python personnalisé. Si un argument de votre script est une entrée ou une sortie de l’étape, il doit être défini *à la fois* dans le tableau `arguments`*ainsi que* dans le paramètre `input` ou `output`, respectivement. 

Dans les scénarios qui comportent plusieurs étapes, une référence d’objet dans le tableau `outputs` devient disponible en tant qu’*entrée* pour une étape de pipeline suivante.

```python
from azureml.pipeline.steps import ParallelRunStep
from datetime import datetime

parallel_step_name = "batchscoring-" + datetime.now().strftime("%Y%m%d%H%M")

label_config = label_ds.as_named_input("labels_input")

batch_score_step = ParallelRunStep(
    name=parallel_step_name,
    inputs=[input_images.as_named_input("input_images")],
    output=output_dir,
    arguments=["--model_name", "inception",
               "--labels_dir", label_config],
    side_inputs=[label_config],
    parallel_run_config=parallel_run_config,
    allow_reuse=False
)
```

Pour obtenir la liste de toutes les classes utilisables dans les différents types d’étape, consultez les informations relatives au [package d’étapes](/python/api/azureml-pipeline-steps/azureml.pipeline.steps?preserve-view=true&view=azure-ml-py).

## <a name="submit-the-pipeline"></a>Envoyer le pipeline

À présent, exécutez le pipeline. Commencez par créer un objet `Pipeline` en utilisant votre référence d’espace de travail et l’étape de pipeline que vous avez créée. Le paramètre `steps` est un tableau d’étapes. Dans le cas présent, le scoring par lots ne comporte qu’une seule étape. Pour créer des pipelines qui comportent plusieurs étapes, placez ces dernières dans l’ordre dans ce tableau.

Utilisez ensuite la fonction `Experiment.submit()` pour soumettre l’exécution du pipeline. La fonction `wait_for_completion` génère des journaux durant le processus de création du pipeline. Vous pouvez utiliser les journaux pour voir la progression actuelle.

> [!IMPORTANT]
> La première exécution du pipeline prend environ *15 minutes*. Toutes les dépendances doivent être téléchargées, une image Docker est créée, et l’environnement Python est provisionné et créé. La réexécution du pipeline prend beaucoup moins de temps, car ces ressources sont réutilisées au lieu d’être créées. Toutefois, le temps total d’exécution du pipeline dépend de la charge de travail de vos scripts et des processus qui s’exécutent à chaque étape du pipeline.

```python
from azureml.core import Experiment
from azureml.pipeline.core import Pipeline

pipeline = Pipeline(workspace=ws, steps=[batch_score_step])
pipeline_run = Experiment(ws, 'batch_scoring').submit(pipeline)
pipeline_run.wait_for_completion(show_output=True)
```

### <a name="download-and-review-output"></a>Télécharger et passer en revue la sortie

Exécutez le code suivant pour télécharger le fichier de sortie créé à partir du script `batch_scoring.py`. Examinez ensuite les résultats du scoring.

```python
import pandas as pd

batch_run = next(pipeline_run.get_children())
batch_output = batch_run.get_output_data("scores")
batch_output.download(local_path="inception_results")

for root, dirs, files in os.walk("inception_results"):
    for file in files:
        if file.endswith("parallel_run_step.txt"):
            result_file = os.path.join(root, file)

df = pd.read_csv(result_file, delimiter=":", header=None)
df.columns = ["Filename", "Prediction"]
print("Prediction has ", df.shape[0], " rows")
df.head(10)
```

## <a name="publish-and-run-from-a-rest-endpoint"></a>Publier et exécuter à partir d’un point de terminaison REST

Exécutez le code suivant pour publier le pipeline sur votre espace de travail. Dans votre espace de travail dans Azure Machine Learning Studio, vous pouvez voir les métadonnées du pipeline, notamment l’historique des exécutions et leurs durées. Vous pouvez également exécuter le pipeline manuellement à partir du studio.

La publication du pipeline active un point de terminaison REST qui vous permet de réexécuter le pipeline à partir de n’importe quelle bibliothèque HTTP sur n’importe quelle plateforme.

```python
published_pipeline = pipeline_run.publish_pipeline(
    name="Inception_v3_scoring", description="Batch scoring using Inception v3 model", version="1.0")

published_pipeline
```

Pour exécuter le pipeline à partir du point de terminaison REST, vous avez besoin d’un en-tête d’authentification de type porteur OAuth2. L’exemple suivant utilise l’authentification interactive (à des fins d’illustration). Toutefois, dans la plupart des scénarios de production qui nécessitent une authentification automatisée ou sans assistance, utilisez l’authentification du principal de service, comme [décrit dans cet article](how-to-setup-authentication.md).

Pour permettre l’authentification du principal de service, créez une *inscription d’application* dans *Azure Active Directory*. Commencez par générer un secret client, puis accordez au principal de service un *accès en fonction du rôle* à votre espace de travail Machine Learning. Utilisez la classe [`ServicePrincipalAuthentication`](/python/api/azureml-core/azureml.core.authentication.serviceprincipalauthentication?preserve-view=true&view=azure-ml-py) pour gérer votre flux d’authentification. 

[`InteractiveLoginAuthentication`](/python/api/azureml-core/azureml.core.authentication.interactiveloginauthentication?preserve-view=true&view=azure-ml-py) et `ServicePrincipalAuthentication` héritent de `AbstractAuthentication`. Dans les deux cas, utilisez la fonction [`get_authentication_header()`](/python/api/azureml-core/azureml.core.authentication.abstractauthentication?preserve-view=true&view=azure-ml-py#&preserve-view=trueget-authentication-header--) de la même manière pour extraire l’en-tête :

```python
from azureml.core.authentication import InteractiveLoginAuthentication

interactive_auth = InteractiveLoginAuthentication()
auth_header = interactive_auth.get_authentication_header()
```

Obtenez l’URL REST de la propriété `endpoint` de l’objet de pipeline publié. Vous pouvez également trouver l’URL REST dans votre espace de travail dans Azure Machine Learning Studio. 

Créez une requête HTTP POST au point de terminaison. Spécifiez votre en-tête d’authentification dans la requête. Ajoutez un objet de charge utile JSON qui porte le nom de l’expérience.

Exécutez la requête pour déclencher l’exécution. Incluez le code permettant d’accéder à la clé `Id` à partir du dictionnaire de réponses pour obtenir la valeur de l’ID d’exécution.

```python
import requests

rest_endpoint = published_pipeline.endpoint
response = requests.post(rest_endpoint, 
                         headers=auth_header, 
                         json={"ExperimentName": "batch_scoring",
                               "ParameterAssignments": {"process_count_per_node": 6}})
run_id = response.json()["Id"]
```

Utilisez l’ID d’exécution pour superviser l’état de la nouvelle exécution. La nouvelle exécution prend à nouveau 10 à 15 minutes pour s’achever. 

La nouvelle exécution va ressembler au pipeline que vous avez exécuté dans le tutoriel. Vous pouvez choisir de ne pas voir la sortie complète.

```python
from azureml.pipeline.core.run import PipelineRun
from azureml.widgets import RunDetails

published_pipeline_run = PipelineRun(ws.experiments["batch_scoring"], run_id)
RunDetails(published_pipeline_run).show()
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Sautez cette section si vous prévoyez de suivre d’autres tutoriels Azure Machine Learning.

### <a name="stop-the-compute-instance"></a>Arrêter l’instance de calcul

[!INCLUDE [aml-stop-server](../../includes/aml-stop-server.md)]

### <a name="delete-everything"></a>Tout supprimer

Si vous n’avez pas l’intention d’utiliser les ressources que vous avez créées, supprimez-les pour éviter des frais :

1. Dans le portail Azure, dans le menu de gauche, sélectionnez **Groupes de ressources**.
1. Dans la liste des groupes de ressources, sélectionnez le groupe de ressources que vous avez créé.
1. Sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources. Ensuite, sélectionnez **Supprimer**.

Vous pouvez également conserver le groupe de ressources mais supprimer un espace de travail unique. Affichez les propriétés de l’espace de travail, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel sur les pipelines Machine Learning, vous avez effectué les tâches suivantes :

> [!div class="checklist"]
> * Création d’un pipeline avec des dépendances d’environnement à exécuter sur une ressource de calcul GPU distante
> * Création d’un script de scoring pour exécuter des prédictions par lots à l’aide d’un modèle Tensorflow préentraîné
> * Publication d’un pipeline à exécuter à partir d’un point de terminaison REST

Pour obtenir des exemples supplémentaires de création de pipelines à l’aide du kit SDK Machine Learning, consultez le [dépôt de notebooks](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/machine-learning-pipelines).
