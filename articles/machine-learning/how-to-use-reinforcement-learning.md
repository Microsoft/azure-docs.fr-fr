---
title: Entraînement et déploiement d’un modèle d’apprentissage par renforcement (préversion)
titleSuffix: Azure Machine Learning
description: Apprenez à utiliser l’apprentissage par renforcement Azure Machine Learning (préversion) pour entraîner un agent d’apprentissage par renforcement à jouer à Pong.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: peterlu
author: peterclu
ms.date: 05/05/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python, contperf-fy21q2
ms.openlocfilehash: 7144d576694b6694f426533451717cef58c2da87
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97562444"
---
# <a name="reinforcement-learning-preview-with-azure-machine-learning"></a>Apprentissage par renforcement (préversion) avec Azure Machine Learning



> [!NOTE]
> L’apprentissage par renforcement Azure Machine Learning est actuellement une fonctionnalité en préversion. Seules les infrastructures Ray et RLlib sont prises en charge pour l’instant.

Cet article explique comment former un agent d’apprentissage par renforcement pour jouer au jeu vidéo Pong. Vous allez utiliser la bibliothèque Python open source [Ray RLlib](https://ray.readthedocs.io/en/master/rllib.html) avec Azure Machine Learning pour gérer la complexité de l’apprentissage par renforcement distribué.

Dans cet article, vous apprendrez comment :
> [!div class="checklist"]
> * Configurer une expérience
> * Définir des nœuds principal et Worker
> * Créer un estimateur d’apprentissage par renforcement
> * Soumettre une expérience pour démarrer une exécution
> * Afficher les résultats

Cet article est basé sur l’[exemple RLlib Pong](https://aka.ms/azureml-rl-pong) qui se trouve dans le bloc-notes Azure Machine Learning [Dépôt GitHub](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/reinforcement-learning/README.md).

## <a name="prerequisites"></a>Prérequis

Exécutez ce code dans l’un de ces environnements : Nous vous recommandons d’essayer une instance de calcul Azure Machine Learning pour bénéficier de l’expérience de démarrage la plus rapide. Vous pouvez cloner et exécuter rapidement les exemples de notebooks de renforcement sur une instance de calcul Azure Machine Learning.

 - Instance de calcul Azure Machine Learning

     - Découvrez comment cloner des exemples de bloc-notes dans [Didacticiel : Configurer l'environnement et l'espace de travail](tutorial-1st-experiment-sdk-setup.md).
         - Cloner le dossier **how-to-use-azureml** au lieu des **didacticiels**
     - Exécutez le bloc-notes de configuration du réseau virtuel situé dans `/how-to-use-azureml/reinforcement-learning/setup/devenv_setup.ipynb` pour ouvrir les ports réseau utilisés pour l’apprentissage par renforcement distribué.
     - Exécuter l’exemple de notebook `/how-to-use-azureml/reinforcement-learning/atari-on-distributed-compute/pong_rllib.ipynb`
 
 - Votre propre serveur de notebooks Jupyter

    - Installez le [Kit de développement logiciel (SDK) Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py).
    - Installez le [Kit de développement logiciel (SDK) d’apprentissage par renforcement d’Azure Machine Learning](/python/api/azureml-contrib-reinforcementlearning/?preserve-view=true&view=azure-ml-py) : `pip install --upgrade azureml-contrib-reinforcementlearning`
    - Créer un [fichier de configuration d’espace de travail](how-to-configure-environment.md#workspace).
    - Exécutez le réseau virtuel pour ouvrir les ports réseau utilisés pour l’apprentissage par renforcement distribué.


## <a name="how-to-train-a-pong-playing-agent"></a>Comment former un agent jouant à Pong

L’apprentissage par renforcement est une approche de l’apprentissage automatique qui consiste à apprendre en faisant. Alors que d’autres techniques d’apprentissage automatique apprennent par l’ingestion passive de données d’entrée et la recherche de modèles dans celles-ci, l’apprentissage par renforcement utilise des **agents d’apprentissage** pour prendre des décisions activement et apprendre de leurs résultats.

Vos agents d’apprentissage apprennent à jouer à Pong dans un **environnement simulé**. Les agents d’apprentissage prennent une décision à chaque image du jeu pour déplacer la raquette vers le haut ou le haut, ou rester sur place. Ils examinent l’état du jeu (image RVB de l’écran) pour prendre une décision.

L’apprentissage par renforcement utilise des **récompenses** pour indiquer à l’agent si ses décisions sont fructueuses. Dans cet exemple, l’agent obtient une récompense positive quand il marque un point et une récompense négative quand un point est marqué contre lui. Sur de nombreuses itérations, l’agent apprend à choisir l’action, en fonction de son état actuel, qui optimise la somme des récompenses futures attendues. Il est d’usage courant d’utiliser des **réseaux neuronaux profonds** pour effectuer cette optimisation de l’apprentissage par renforcement. 

L’apprentissage se termine quand l’agent atteint un score de récompense moyen de 18 par période d’apprentissage. Cela signifie que l’agent a battu son adversaire en gagnant en moyenne au moins 18 points par match au meilleur des 21 points.

Le processus d’itération dans la simulation et de réformation d’un réseau neuronal profond est coûteux en termes de calcul et nécessite beaucoup de données. Une façon d’améliorer les performances d’apprentissage par renforcement consiste à **paralléliser le travail** afin que plusieurs agents d’apprentissage puissent agir et apprendre simultanément. Toutefois, la gestion d’un environnement d’apprentissage par renforcement distribué peut être une entreprise complexe.

Azure Machine Learning fournit l’infrastructure nécessaire pour gérer les complexités liées à l’augmentation de l’échelle des charges de travail d’apprentissage par renforcement.

## <a name="set-up-the-environment"></a>Configurer l’environnement

Configurez l’environnement local de l’apprentissage par renforcement en procédant comme suit :
1. Chargement des packages Python requis
1. Initialisation de votre espace de travail
1. Création d’une expérience
1. Spécification d’un réseau virtuel configuré

### <a name="import-libraries"></a>Importer les bibliothèques

Importez les packages Python nécessaires pour exécuter le reste de cet exemple.

```python
# Azure ML Core imports
import azureml.core
from azureml.core import Workspace
from azureml.core import Experiment
from azureml.core.compute import AmlCompute
from azureml.core.compute import ComputeTarget
from azureml.core.runconfig import EnvironmentDefinition
from azureml.widgets import RunDetails
from azureml.tensorboard import Tensorboard

# Azure ML Reinforcement Learning imports
from azureml.contrib.train.rl import ReinforcementLearningEstimator, Ray
from azureml.contrib.train.rl import WorkerConfiguration
```

### <a name="initialize-a-workspace"></a>Initialiser un espace de travail

Initialisez un objet d’[espace de travail](concept-workspace.md) à partir du fichier `config.json` créé dans la [section Prérequis](#prerequisites). Si vous exécutez ce code dans une instance de calcul Azure Machine Learning, le fichier de configuration a déjà été créé pour vous.

```Python
ws = Workspace.from_config()
```

### <a name="create-a-reinforcement-learning-experiment"></a>Créer une expérience d’apprentissage par renforcement

Créez une [expérience](/python/api/azureml-core/azureml.core.experiment.experiment?preserve-view=true&view=azure-ml-py) pour suivre votre apprentissage par renforcement. Dans Azure Machine Learning, les expériences sont des collections logiques d’essais associés pour organiser des journaux d’exécution, un historique, des résultats et bien plus encore.

```python
experiment_name='rllib-pong-multi-node'

exp = Experiment(workspace=ws, name=experiment_name)
```

### <a name="specify-a-virtual-network"></a>Spécifier un réseau virtuel

Pour les travaux apprentissage par renforcement qui utilisent plusieurs cibles de calcul, vous devez spécifier un réseau virtuel avec des ports ouverts qui permettent aux nœuds Worker et principal de communiquer entre eux.

Le réseau virtuel peut se trouver dans n’importe quel groupe de ressources, mais celui-ci doit être situé dans la même région que votre espace de travail. Pour plus d’informations sur la configuration de votre réseau virtuel, consultez le notebook de configuration de l’espace de travail qui se trouve dans la section Prérequis. Vous spécifiez ici le nom du réseau virtuel dans votre groupe de ressources.

```python
vnet = 'your_vnet'
```

## <a name="define-head-and-worker-compute-targets"></a>Définir les cibles de calcul principal et Worker

Cet exemple utilise des cibles de calcul distinctes pour les nœuds Worker et principal de l’infrastructure Ray. Ces paramètres vous permettent de mettre à l’échelle vos ressources de calcul en fonction de la charge de travail. Définissez le nombre de nœuds et la taille de chacun de ceux-ci en fonction de vos besoins.

### <a name="head-computing-target"></a>Cible de calcul principale

Vous pouvez utiliser un cluster principal équipé d’un GPU pour améliorer les performances de Deep Learning. Le nœud principal forme le réseau neuronal que l’agent utilise pour prendre des décisions. Le nœud principal collecte également des points de données dans les nœuds Worker pour effectuer l’apprentissage du réseau neuronal.

Le calcul principal utilise une seule [machine virtuelle `STANDARD_NC6`](../virtual-machines/nc-series.md). Il dispose de six processeurs virtuels pour répartir le travail.


```python
from azureml.core.compute import AmlCompute, ComputeTarget

# choose a name for the Ray head cluster
head_compute_name = 'head-gpu'
head_compute_min_nodes = 0
head_compute_max_nodes = 2

# This example uses GPU VM. For using CPU VM, set SKU to STANDARD_D2_V2
head_vm_size = 'STANDARD_NC6'

if head_compute_name in ws.compute_targets:
    head_compute_target = ws.compute_targets[head_compute_name]
    if head_compute_target and type(head_compute_target) is AmlCompute:
        print(f'found head compute target. just use it {head_compute_name}')
else:
    print('creating a new head compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size = head_vm_size,
                                                                min_nodes = head_compute_min_nodes, 
                                                                max_nodes = head_compute_max_nodes,
                                                                vnet_resourcegroup_name = ws.resource_group,
                                                                vnet_name = vnet_name,
                                                                subnet_name = 'default')

    # create the cluster
    head_compute_target = ComputeTarget.create(ws, head_compute_name, provisioning_config)
    
    # can poll for a minimum number of nodes and for a specific timeout. 
    # if no min node count is provided it will use the scale settings for the cluster
    head_compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
    
     # For a more detailed view of current AmlCompute status, use get_status()
    print(head_compute_target.get_status().serialize())
```

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

### <a name="worker-computing-cluster"></a>Cluster de calcul Worker

Cet exemple utilise quatre [machines virtuelles `STANDARD_D2_V2`](../virtual-machines/nc-series.md) pour la cible de calcul Worker. Chaque nœud Worker dispose de 2 UC pour un total de 8 UC disponibles.

Les GPU ne sont pas nécessaires pour les nœuds Worker, car ils n’effectuent pas d’apprentissage approfondi. Les Workers exécutent les simulations de jeu et collectent des données.

```python
# choose a name for your Ray worker cluster
worker_compute_name = 'worker-cpu'
worker_compute_min_nodes = 0 
worker_compute_max_nodes = 4

# This example uses CPU VM. For using GPU VM, set SKU to STANDARD_NC6
worker_vm_size = 'STANDARD_D2_V2'

# Create the compute target if it hasn't been created already
if worker_compute_name in ws.compute_targets:
    worker_compute_target = ws.compute_targets[worker_compute_name]
    if worker_compute_target and type(worker_compute_target) is AmlCompute:
        print(f'found worker compute target. just use it {worker_compute_name}')
else:
    print('creating a new worker compute target...')
    provisioning_config = AmlCompute.provisioning_configuration(vm_size = worker_vm_size,
                                                                min_nodes = worker_compute_min_nodes, 
                                                                max_nodes = worker_compute_max_nodes,
                                                                vnet_resourcegroup_name = ws.resource_group,
                                                                vnet_name = vnet_name,
                                                                subnet_name = 'default')

    # create the cluster
    worker_compute_target = ComputeTarget.create(ws, worker_compute_name, provisioning_config)
    
    # can poll for a minimum number of nodes and for a specific timeout. 
    # if no min node count is provided it will use the scale settings for the cluster
    worker_compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
    
     # For a more detailed view of current AmlCompute status, use get_status()
    print(worker_compute_target.get_status().serialize())
```

## <a name="create-a-reinforcement-learning-estimator"></a>Créer un estimateur d’apprentissage par renforcement
Utilisez le [ReinforcementLearningEstimator](/python/api/azureml-contrib-reinforcementlearning/azureml.contrib.train.rl.reinforcementlearningestimator?preserve-view=true&view=azure-ml-py) pour soumettre un travail de formation à Azure Machine Learning.

Azure Machine Learning utilise des classes d’estimateur pour encapsuler les informations de configuration de l’exécution. Cela vous permet de spécifier comment configurer l’exécution d’un script. 

### <a name="define-a-worker-configuration"></a>Définir une configuration de Worker

L’objet WorkerConfiguration indique à Azure Machine Learning comment initialiser le cluster Worker qui exécute le script d’entrée.

```python
# Pip packages we will use for both head and worker
pip_packages=["ray[rllib]==0.8.3"] # Latest version of Ray has fixes for isses related to object transfers

# Specify the Ray worker configuration
worker_conf = WorkerConfiguration(
    
    # Azure ML compute cluster to run Ray workers
    compute_target=worker_compute_target, 
    
    # Number of worker nodes
    node_count=4,
    
    # GPU
    use_gpu=False, 
    
    # PIP packages to use
    pip_packages=pip_packages
)
```

### <a name="define-script-parameters"></a>Définir des paramètres de script

Le script d’entrée `pong_rllib.py` accepte une liste de paramètres qui définit le mode d’exécution du travail d’apprentissage. Le passage de ces paramètres via l’estimateur en tant que couche d’encapsulation facilite la modification des paramètres de script et l’exécution de configurations indépendamment les unes des autres.

Si vous spécifiez la valeur `num_workers` appropriée, vous tirerez le meilleur parti de vos efforts de parallélisation. Définissez le nombre de Workers sur le nombre d’UC disponibles. Pour cet exemple, vous pouvez utiliser le calcul suivant :

Le nœud principal est un [Standard_NC6](../virtual-machines/nc-series.md) avec 6 processeurs virtuels. Le cluster Worker est composé de 4 [machines virtuelles Standard_D2_V2](../cloud-services/cloud-services-sizes-specs.md#dv2-series) avec 2 UC chacune, pour un total de 8 UC. Toutefois, vous devez soustraire du nombre de Workers 1 UC dédiée au rôle de nœud principal.

6 UC + 8 UC - 1 UC de nœud principal = 13 Workers simultanés. Azure Machine Learning utilise des clusters principal et Worker pour distinguer les ressources de calcul. Toutefois, l’infrastructure Ray ne fait pas de distinction entre les clusters principal et Worker, et toutes les UC sont disponibles comme threads de travail.


```python
training_algorithm = "IMPALA"
rl_environment = "PongNoFrameskip-v4"

# Training script parameters
script_params = {
    
    # Training algorithm, IMPALA in this case
    "--run": training_algorithm,
    
    # Environment, Pong in this case
    "--env": rl_environment,
    
    # Add additional single quotes at the both ends of string values as we have spaces in the 
    # string parameters, outermost quotes are not passed to scripts as they are not actually part of string
    # Number of GPUs
    # Number of ray workers
    "--config": '\'{"num_gpus": 1, "num_workers": 13}\'',
    
    # Target episode reward mean to stop the training
    # Total training time in seconds
    "--stop": '\'{"episode_reward_mean": 18, "time_total_s": 3600}\'',
}
```

### <a name="define-the-reinforcement-learning-estimator"></a>Définir l’estimateur d’apprentissage par renforcement

Utilisez la liste de paramètres et l’objet configuration de Worker pour construire l’estimateur.

```python
# RL estimator
rl_estimator = ReinforcementLearningEstimator(
    
    # Location of source files
    source_directory='files',
    
    # Python script file
    entry_script="pong_rllib.py",
    
    # Parameters to pass to the script file
    # Defined above.
    script_params=script_params,
    
    # The Azure ML compute target set up for Ray head nodes
    compute_target=head_compute_target,
    
    # Pip packages
    pip_packages=pip_packages,
    
    # GPU usage
    use_gpu=True,
    
    # RL framework. Currently must be Ray.
    rl_framework=Ray(),
    
    # Ray worker configuration defined above.
    worker_configuration=worker_conf,
    
    # How long to wait for whole cluster to start
    cluster_coordination_timeout_seconds=3600,
    
    # Maximum time for the whole Ray job to run
    # This will cut off the run after an hour
    max_run_duration_seconds=3600,
    
    # Allow the docker container Ray runs in to make full use
    # of the shared memory available from the host OS.
    shm_size=24*1024*1024*1024
)
```

### <a name="entry-script"></a>Script d’entrée

Le [script d’entrée](https://aka.ms/azure-rl-pong-script) `pong_rllib.py` effectuer l’apprentissage d’un réseau neuronal à l’aide de l’[environnement Gym OpenAI](https://github.com/openai/gym/).`PongNoFrameSkip-v4` Les gyms OpenAI sont des interfaces standardisées pour tester des algorithmes d’apprentissage par renforcement sur des jeux Atari classiques.

Cet exemple utilise un algorithme d’apprentissage connu sous le nom d’[IMPALA](https://arxiv.org/abs/1802.01561) (Importance Weighted Actor-Learner Architecture). IMPALA parallélise chaque acteur d’apprentissage pour mettre à l’échelle sur plusieurs nœuds de calcul sans sacrifier la vitesse ou la stabilité.

[Ray Tune](https://ray.readthedocs.io/en/latest/tune.html) orchestre les tâches Worker de l’algorithme IMPALA.

```python
import ray
import ray.tune as tune
from ray.rllib import train

import os
import sys

from azureml.core import Run
from utils import callbacks

DEFAULT_RAY_ADDRESS = 'localhost:6379'

if __name__ == "__main__":

    # Parse arguments
    train_parser = train.create_parser()

    args = train_parser.parse_args()
    print("Algorithm config:", args.config)

    if args.ray_address is None:
        args.ray_address = DEFAULT_RAY_ADDRESS

    ray.init(address=args.ray_address)

    tune.run(run_or_experiment=args.run,
             config={
                 "env": args.env,
                 "num_gpus": args.config["num_gpus"],
                 "num_workers": args.config["num_workers"],
                 "callbacks": {"on_train_result": callbacks.on_train_result},
                 "sample_batch_size": 50,
                 "train_batch_size": 1000,
                 "num_sgd_iter": 2,
                 "num_data_loader_buffers": 2,
                 "model": {
                    "dim": 42
                 },
             },
             stop=args.stop,
             local_dir='./logs')
```

### <a name="logging-callback-function"></a>Fonction de rappel de journalisation


Le script d’entrée utilise une fonction utilitaire pour définir une [fonction de rappel d’infrastructure RLlib personnalisée](https://docs.ray.io/en/latest/rllib-training.html#callbacks-and-custom-metrics) pour journaliser des métriques dans votre espace de travail Azure Machine Learning. Découvrez comment afficher ces métriques dans la section [Surveiller et afficher les résultats](#monitor-and-view-results).

```python
'''RLlib callbacks module:
    Common callback methods to be passed to RLlib trainer.
'''
from azureml.core import Run

def on_train_result(info):
    '''Callback on train result to record metrics returned by trainer.
    '''
    run = Run.get_context()
    run.log(
        name='episode_reward_mean',
        value=info["result"]["episode_reward_mean"])
    run.log(
        name='episodes_total',
        value=info["result"]["episodes_total"])
```

## <a name="submit-a-run"></a>Envoyer une exécution

[Run](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py) gère l’historique des exécutions des travaux en cours ou terminés. 

```python
run = exp.submit(config=rl_estimator)
```
> [!NOTE]
> L’exécution peut prendre de 30 à 45 minutes.

## <a name="monitor-and-view-results"></a>Surveiller et afficher les résultats

Utilisez le widget Jupyter Azure Machine Learning pour voir l’état de vos exécutions en temps réel. Le widget affiche deux exécutions enfants : l’une pour le nœud principal et l’autre pour les nœuds Worker. 

```python
from azureml.widgets import RunDetails

RunDetails(run).show()
run.wait_for_completion()
```

1. Attendez le chargement du widget.
1. Sélectionnez l’exécution principale dans la liste des exécutions.

Pour obtenir des informations supplémentaires sur l’exécution dans le studio, sélectionnez **Cliquez ici pour afficher l’exécution dans Azure Machine Learning Studio**. Vous pouvez accéder à ces informations lorsque l’exécution est en cours, ou une fois qu’elle est terminée.

![Graphique linéaire montrant comment exécuter le widget des détails](./media/how-to-use-reinforcement-learning/pong-run-details-widget.png)

Le tracé **episode_reward_mean** montre le nombre moyen de points évalués par période d’apprentissage. Vous pouvez voir que l’agent d’apprentissage a initialement affiché des performances médiocres, perdant ses matches sans gagner un seul point (comme l’indique la valeur reward_mean de -21). En 100 itérations, l’agent d’apprentissage a appris à battre l’ordinateur adversaire avec une moyenne de 18 points.

Si vous parcourez les journaux de l’exécution enfant, vous pouvez voir les résultats de l’évaluation enregistrés dans le fichier driver_log.txt. Il se peut que vous deviez attendre quelques minutes avant que ces mesures deviennent disponibles sur la page Exécution.

Un bref travail vous a permis d’apprendre à configurer plusieurs ressources de calcul pour former un agent d’apprentissage par renforcement à très bien jouer au Pong contre un adversaire informatique.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert comment former un agent d’apprentissage par renforcement à l’aide d’un agent d’apprentissage IMPALA. Pour des exemples supplémentaires, accédez au [dépôt GitHub sur l’apprentissage par renforcement Azure Machine Learning](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/reinforcement-learning/README.md).