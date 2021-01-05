---
title: Déboguer et détecter un problème ParallelRunStep
titleSuffix: Azure Machine Learning
description: Déboguez et résolvez les problèmes ParallelRunStep dans les pipelines machine learning dans le SDK Azure Machine Learning pour Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: troubleshooting
ms.reviewer: jmartens, larryfr, vaidyas, laobri, tracych
ms.author: trmccorm
author: tmccrmck
ms.date: 09/23/2020
ms.openlocfilehash: 09f75e9e8f972ec44098e119dc5b30bd44638918
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94630462"
---
# <a name="debug-and-troubleshoot-parallelrunstep"></a>Déboguer et détecter un problème ParallelRunStep


Dans cet article, vous allez découvrir comment déboguer et détecter un problème de la classe [ParallelRunStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.parallel_run_step.parallelrunstep?preserve-view=true&view=azure-ml-py) à partir du [Kit de développement logiciel (SDK) Azure Machine Learning](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py).

## <a name="testing-scripts-locally"></a>Tester les scripts localement

Consultez la [section Tester les scripts localement](how-to-debug-visual-studio-code.md#debug-and-troubleshoot-machine-learning-pipelines) pour les pipelines de Machine Learning (ML). Votre ParallelRunStep s’exécute comme une étape dans les pipelines ML, de sorte que la même réponse s’applique aux deux.

## <a name="debugging-scripts-from-remote-context"></a>Déboguer les scripts à partir du contexte distant

La transition du débogage d’un script de scoring localement au débogage d’un script de scoring dans un pipeline réel peut être difficile. Pour plus d’informations sur la recherche de vos journaux dans le portail, consultez la [section Pipelines de Machine Learning sur le débogage de scripts à partir d’un contexte distant](how-to-debug-pipelines.md). Les informations contenues dans cette section s’appliquent également à un ParallelRunStep.

Par exemple, le fichier journal `70_driver_log.txt` contient des informations provenant du contrôleur qui lance le code ParallelRunStep.

En raison de la nature distribuée des travaux ParallelRunStep, les journaux peuvent provenir de plusieurs sources différentes. Toutefois, deux fichiers consolidés sont créés et fournissent des informations de haut niveau :

- `~/logs/job_progress_overview.txt`: Ce fichier fournit des informations de haut niveau sur le nombre de mini-lots (également appelés tâches) créés jusqu’à présent et le nombre de mini-lots traités jusqu’à présent. À cette fin, il affiche le résultat du travail. Si le travail a échoué, le message d’erreur s’affiche et indique où démarrer la résolution des problèmes.

- `~/logs/sys/master_role.txt`: Ce fichier fournit la vue du nœud principal (également connu sous le nom d’orchestrateur) du travail en cours d’exécution. Comprend la création de tâches, la supervision de la progression et le résultat de l’exécution.

Les journaux générés à partir du script d’entrée avec l’assistance EntryScript et les instructions print se trouvent dans les fichiers suivants :

- `~/logs/user/entry_script_log/<ip_address>/<process_name>.log.txt`: Ces fichiers sont les journaux écrits à partir d’entry_script avec le helper EntryScript.

- `~/logs/user/stdout/<ip_address>/<process_name>.stdout.txt` : ces fichiers sont les journaux de stdout (par exemple, l’instruction print) d’entry_script.

- `~/logs/user/stderr/<ip_address>/<process_name>.stderr.txt` : ces fichiers sont les journaux de stderr d’entry_script.

Pour une compréhension concise des erreurs contenues dans votre script :

- `~/logs/user/error.txt`: Ce fichier tentera de résumer les erreurs contenues dans votre script.

Pour plus d’informations sur les erreurs dans votre script :

- `~/logs/user/error/` : contient des traces de la pile complète d’exceptions levées pendant le chargement et l’exécution du script d’entrée.

Lorsque vous avez besoin de comprendre en détail la façon dont chaque nœud a exécuté le script de score, examinez les journaux de processus individuels pour chaque nœud. Les journaux de processus se trouvent dans le dossier `sys/node`, regroupés par nœuds Worker :

- `~/logs/sys/node/<ip_address>/<process_name>.txt`: Ce fichier fournit des informations détaillées sur chaque mini-lot au fur et à mesure qu’il est sélectionné ou traité par un Worker. Pour chaque mini-lot, ce fichier comprend les éléments suivants :

    - L’adresse IP et le PID du processus Worker. 
    - Le nombre total d’éléments, le nombre d’éléments traités avec succès et le nombre d’éléments ayant échoué.
    - L’heure de début, la durée, le temps de traitement et la durée de la méthode d’exécution.

Vous pouvez également trouver des informations sur l’utilisation des ressources des processus pour chaque Worker. Ces informations sont au format CSV et se trouvent dans `~/logs/sys/perf/<ip_address>/node_resource_usage.csv`. Les informations sur chaque processus sont disponibles sous `~logs/sys/perf/<ip_address>/processes_resource_usage.csv`.

### <a name="how-do-i-log-from-my-user-script-from-a-remote-context"></a>Comment se connecter à son script utilisateur depuis un contexte distant ?
ParallelRunStep peut exécuter plusieurs processus sur un nœud en fonction de process_count_per_node. Pour organiser les journaux de chaque processus sur un nœud et combiner les instructions print et log, nous vous recommandons d’utiliser l’enregistreur d’événements ParallelRunStep comme indiqué ci-dessous. Vous recevez d’EntryScript un enregistreur d’événements et affichez les journaux dans le dossier **logs/user** dans le portail.

**exemple de script d’entrée utilisant l’enregistreur d’événements :**
```python
from azureml_user.parallel_run import EntryScript

def init():
    """ Initialize the node."""
    entry_script = EntryScript()
    logger = entry_script.logger
    logger.debug("This will show up in files under logs/user on the Azure portal.")


def run(mini_batch):
    """ Accept and return the list back."""
    # This class is in singleton pattern and will return same instance as the one in init()
    entry_script = EntryScript()
    logger = entry_script.logger
    logger.debug(f"{__file__}: {mini_batch}.")
    ...

    return mini_batch
```

### <a name="how-could-i-pass-a-side-input-such-as-a-file-or-files-containing-a-lookup-table-to-all-my-workers"></a>Comment puis-je passer une entrée supplémentaire, par exemple un ou des fichiers contenant une table de recherche, à tous mes collaborateurs ?

L’utilisateur peut passer des données de référence au script à l’aide du paramètre side_inputs de ParalleRunStep. Tous les jeux de données fournis en tant que side_inputs seront montés sur chaque nœud Worker. L’utilisateur peut récupérer l’emplacement du montage en passant l’argument.

Construisez un [Jeu de données](/python/api/azureml-core/azureml.core.dataset.dataset?preserve-view=true&view=azure-ml-py) contenant les données de référence, et inscrivez-le auprès de votre espace de travail. Transmettez-le au paramètre `side_inputs` de votre `ParallelRunStep`. En outre, vous pouvez ajouter son chemin dans la section `arguments` pour accéder facilement à son chemin monté :

```python
label_config = label_ds.as_named_input("labels_input")
batch_score_step = ParallelRunStep(
    name=parallel_step_name,
    inputs=[input_images.as_named_input("input_images")],
    output=output_dir,
    arguments=["--labels_dir", label_config],
    side_inputs=[label_config],
    parallel_run_config=parallel_run_config,
)
```

Après cela, vous pouvez y accéder dans votre script d’inférence, par exemple dans votre méthode init(), comme suit :

```python
parser = argparse.ArgumentParser()
parser.add_argument('--labels_dir', dest="labels_dir", required=True)
args, _ = parser.parse_known_args()

labels_path = args.labels_dir
```

### <a name="how-to-use-input-datasets-with-service-principal-authentication"></a>Comment utiliser des jeux de données d’entrée avec l’authentification du principal du service ?

L’utilisateur peut passer des jeux de données d’entrée avec l’authentification du principal du service utilisée dans l’espace de travail. L’utilisation d’un tel jeu de données dans ParallelRunStep nécessite que le jeu de données soit inscrit pour pouvoir construire une configuration ParallelRunStep.

```python
service_principal = ServicePrincipalAuthentication(
    tenant_id="**_",
    service_principal_id="_*_",
    service_principal_password="_*_")
 
ws = Workspace(
    subscription_id="_*_",
    resource_group="_*_",
    workspace_name="_*_",
    auth=service_principal
    )
 
default_blob_store = ws.get_default_datastore() # or Datastore(ws, '_*_datastore-name_*_') 
ds = Dataset.File.from_files(default_blob_store, '_*path**_')
registered_ds = ds.register(ws, '_*_dataset-name_*_', create_new_version=True)
```

## <a name="next-steps"></a>Étapes suivantes

_ Consultez ces [notebooks Jupyter illustrant des pipelines Azure Machine Learning](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/machine-learning-pipelines).

* Consultez les informations de référence sur le SDK pour obtenir de l’aide sur le package [azureml-pipeline-Steps](/python/api/azureml-pipeline-steps/azureml.pipeline.steps?preserve-view=true&view=azure-ml-py). Consultez la [documentation](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.parallelrunstep?preserve-view=true&view=azure-ml-py) de référence pour la classe ParallelRunStep.

* Suivez le [tutoriel avancé](tutorial-pipeline-batch-scoring-classification.md) sur l’utilisation des pipelines avec ParallelRunStep. Le tutoriel montre comment passer un autre fichier en tant qu’entrée de côté.