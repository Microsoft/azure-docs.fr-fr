---
title: Pipeline d’apprentissage automatique YAML
titleSuffix: Azure Machine Learning
description: Découvrez comment définir un pipeline d’apprentissage automatique à l’aide d’un fichier YAML. Les définitions de pipeline YAML sont utilisées avec l’extension Machine Learning pour Azure CLI.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.reviewer: larryfr
ms.author: nilsp
author: NilsPohlmann
ms.date: 07/31/2020
ms.custom: devx-track-python
ms.openlocfilehash: bfeab990c841f6b65e665b4a8aabdfd8b251da60
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93323909"
---
# <a name="define-machine-learning-pipelines-in-yaml"></a>Définir des pipelines d’apprentissage automatique en YAML

Découvrez comment définir vos pipelines d’apprentissage automatique en [YAML](https://yaml.org/). Lors de l’utilisation de l’extension Machine Learning pour Azure CLI, un grand nombre des commandes associées au pipeline attendent un fichier YAML qui définit ce dernier.

Le tableau suivant répertorie les éléments qui sont ou non actuellement pris en charge lors de la définition d’un pipeline en YAML :

| Type d’étape | Pris en charge ? |
| ----- | :-----: |
| PythonScriptStep | Oui |
| ParallelRunStep | Oui |
| AdlaStep | Oui |
| AzureBatchStep | Oui |
| DatabricksStep | Oui |
| DataTransferStep | Oui |
| AutoMLStep | Non |
| HyperDriveStep | Non |
| ModuleStep | Oui |
| MPIStep | Non |
| EstimatorStep | Non |

## <a name="pipeline-definition"></a>Définition de pipeline

Une définition de pipeline utilise les clés suivantes, qui correspondent à la classe [Pipelines](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipeline.pipeline?preserve-view=true&view=azure-ml-py) :

| Clé YAML | Description |
| ----- | ----- |
| `name` | Description du pipeline. |
| `parameters` | Paramètre(s) du pipeline. |
| `data_reference` | Définit comment et où les données doivent être mises à disposition dans une exécution. |
| `default_compute` | Cible de calcul par défaut dans laquelle toutes les étapes du pipeline s’exécutent. |
| `steps` | Étapes utilisées dans le pipeline. |

## <a name="parameters"></a>Paramètres

La section `parameters` utilise les clés suivantes, qui correspondent à la classe [PipelineParameter](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelineparameter?preserve-view=true&view=azure-ml-py) :

| Clé YAML | Description |
| ---- | ---- |
| `type` | Type valeur du paramètre. Les types valides sont `string`, `int`, `float`, `bool` ou `datapath`. |
| `default` | Valeur par défaut. |

Chaque paramètre est nommé. Par exemple, l’extrait de code YAML suivant définit trois paramètres nommés `NumIterationsParameter`, `DataPathParameter` et `NodeCountParameter` :

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        NumIterationsParameter:
            type: int
            default: 40
        DataPathParameter:
            type: datapath
            default:
                datastore: workspaceblobstore
                path_on_datastore: sample2.txt
        NodeCountParameter:
            type: int
            default: 4
```

## <a name="data-reference"></a>Référence de données

La section `data_references` utilise les clés suivantes, qui correspondent à la classe [DataReference](/python/api/azureml-core/azureml.data.data_reference.datareference?preserve-view=true&view=azure-ml-py) :

| Clé YAML | Description |
| ----- | ----- |
| `datastore` | Magasin de banques à référencer. |
| `path_on_datastore` | Chemin d’accès relatif dans le stockage de sauvegarde pour la référence de données. |

Chaque référence de données est contenue dans une clé. Par exemple, l’extrait de code YAML suivant définit une référence de données stockée dans la clé nommée `employee_data` :

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        employee_data:
            datastore: adftestadla
            path_on_datastore: "adla_sample/sample_input.csv"
```

## <a name="steps"></a>Étapes

Les étapes définissent un environnement de calcul, ainsi que les fichiers à exécuter sur l’environnement. Pour définir le type d’une étape, utilisez la clé `type` :

| Type d’étape | Description |
| ----- | ----- |
| `AdlaStep` | Exécute un script U-SQL à l’aide d’Azure Data Lake Analytics. Correspond à la classe [AdlaStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.adlastep?preserve-view=true&view=azure-ml-py). |
| `AzureBatchStep` | Exécute des travaux à l’aide d’Azure Batch. Correspond à la classe [AzureBatchStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.azurebatchstep?preserve-view=true&view=azure-ml-py). |
| `DatabricsStep` | Ajoute un notebook Databricks, un script Python ou un fichier JAR. Correspond à la classe [DatabricksStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.databricksstep?preserve-view=true&view=azure-ml-py). |
| `DataTransferStep` | Transfère des données entre les options de stockage. Correspond à la classe [DataTransferStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.datatransferstep?preserve-view=true&view=azure-ml-py). |
| `PythonScriptStep` | Exécute un script Python. Correspond à la classe [PythonScriptStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.python_script_step.pythonscriptstep?preserve-view=true&view=azure-ml-py). |
| `ParallelRunStep` | Exécute un script Python pour traiter de grandes quantités de données de façon asynchrone et en parallèle. Correspond à la classe [ParallelRunStep](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.parallel_run_step.parallelrunstep?preserve-view=true&view=azure-ml-py). |

### <a name="adla-step"></a>Étape ADLA

| Clé YAML | Description |
| ----- | ----- |
| `script_name` | Nom du script U-SQL (relatif à `source_directory`). |
| `compute_target` | Cible de calcul Azure Data Lake à utiliser pour cette étape. |
| `parameters` | [Paramètres](#parameters) vers le pipeline. |
| `inputs` | Les entrées peuvent être [InputPortBinding](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding?preserve-view=true&view=azure-ml-py), [DataReference](#data-reference), [PortDataReference](/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference?preserve-view=true&view=azure-ml-py), [PipelineData](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?preserve-view=true&view=azure-ml-py), [DataSet](/python/api/azureml-core/azureml.core.dataset%28class%29?preserve-view=true&view=azure-ml-py), [DatasetDefinition](/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition?preserve-view=true&view=azure-ml-py) ou [PipelineDataset](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset?preserve-view=true&view=azure-ml-py). |
| `outputs` | Les sorties peuvent être [PipelineData](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?preserve-view=true&view=azure-ml-py) ou [OutputPortBinding](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding?preserve-view=true&view=azure-ml-py). |
| `source_directory` | Répertoire qui contient le script, les assemblys, etc. |
| `priority` | Valeur de priorité à utiliser pour le travail en cours. |
| `params` | Dictionnaire de paires nom-valeur. |
| `degree_of_parallelism` | Degré de parallélisme à utiliser pour ce travail. |
| `runtime_version` | Version du runtime du moteur Data Lake Analytics. |
| `allow_reuse` | Détermine si l’étape doit réutiliser les résultats précédents lorsqu’elle est exécutée avec les mêmes paramètres. |

L’exemple suivant contient une définition de l’étape ADLA :

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        employee_data:
            datastore: adftestadla
            path_on_datastore: "adla_sample/sample_input.csv"
    default_compute: adlacomp
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "AdlaStep"
            name: "MyAdlaStep"
            script_name: "sample_script.usql"
            source_directory: "D:\\scripts\\Adla"
            inputs:
                employee_data:
                    source: employee_data
            outputs:
                OutputData:
                    destination: Output4
                    datastore: adftestadla
                    bind_mode: mount
```

### <a name="azure-batch-step"></a>Étape Azure Batch

| Clé YAML | Description |
| ----- | ----- |
| `compute_target` | Cible de calcul Azure Batch à utiliser pour cette étape. |
| `inputs` | Les entrées peuvent être [InputPortBinding](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding?preserve-view=true&view=azure-ml-py), [DataReference](#data-reference), [PortDataReference](/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference?preserve-view=true&view=azure-ml-py), [PipelineData](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?preserve-view=true&view=azure-ml-py), [DataSet](/python/api/azureml-core/azureml.core.dataset%28class%29?preserve-view=true&view=azure-ml-py), [DatasetDefinition](/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition?preserve-view=true&view=azure-ml-py) ou [PipelineDataset](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset?preserve-view=true&view=azure-ml-py). |
| `outputs` | Les sorties peuvent être [PipelineData](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?preserve-view=true&view=azure-ml-py) ou [OutputPortBinding](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding?preserve-view=true&view=azure-ml-py). |
| `source_directory` | Répertoire qui contient les fichiers binaires du module, l’exécutable, les assemblys, etc. |
| `executable` | Nom de la commande ou de l’exécutable qui sera exécuté dans le cadre de ce travail. |
| `create_pool` | Indicateur booléen pour indiquer s’il faut créer le pool avant d’exécuter le travail. |
| `delete_batch_job_after_finish` | Indicateur booléen pour indiquer s’il faut supprimer le travail du compte Batch une fois terminé. |
| `delete_batch_pool_after_finish` | Indicateur booléen pour indiquer s’il faut supprimer le pool une fois le travail terminé. |
| `is_positive_exit_code_failure` | Indicateur booléen pour indiquer si le travail échoue lorsque la tâche se termine avec un code positif. |
| `vm_image_urn` | Si `create_pool` est `True` et que la machine virtuelle utilise `VirtualMachineConfiguration`. |
| `pool_id` | ID du pool dans lequel le travail sera exécuté. |
| `allow_reuse` | Détermine si l’étape doit réutiliser les résultats précédents lorsqu’elle est exécutée avec les mêmes paramètres. |

L’exemple suivant contient une définition de l’étape Azure Batch :

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        input:
            datastore: workspaceblobstore
            path_on_datastore: "input.txt"
    default_compute: testbatch
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "AzureBatchStep"
            name: "MyAzureBatchStep"
            pool_id: "MyPoolName"
            create_pool: true
            executable: "azurebatch.cmd"
            source_directory: "D:\\scripts\\AureBatch"
            allow_reuse: false
            inputs:
                input:
                    source: input
            outputs:
                output:
                    destination: output
                    datastore: workspaceblobstore
```

### <a name="databricks-step"></a>Étape Databricks

| Clé YAML | Description |
| ----- | ----- |
| `compute_target` | Cible de calcul Azure Databricks à utiliser pour cette étape. |
| `inputs` | Les entrées peuvent être [InputPortBinding](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding?preserve-view=true&view=azure-ml-py), [DataReference](#data-reference), [PortDataReference](/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference?preserve-view=true&view=azure-ml-py), [PipelineData](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?preserve-view=true&view=azure-ml-py), [DataSet](/python/api/azureml-core/azureml.core.dataset%28class%29?preserve-view=true&view=azure-ml-py), [DatasetDefinition](/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition?preserve-view=true&view=azure-ml-py) ou [PipelineDataset](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset?preserve-view=true&view=azure-ml-py). |
| `outputs` | Les sorties peuvent être [PipelineData](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?preserve-view=true&view=azure-ml-py) ou [OutputPortBinding](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding?preserve-view=true&view=azure-ml-py). |
| `run_name` | Nom dans Databricks pour cette exécution. |
| `source_directory` | Répertoire qui contient le script et d’autres fichiers. |
| `num_workers` | Nombre statique de Workers pour le cluster d’exécution Databricks. |
| `runconfig` | Chemin d’accès du fichier `.runconfig`. Ce fichier est une représentation YAML de la classe [RunConfiguration](/python/api/azureml-core/azureml.core.runconfiguration?preserve-view=true&view=azure-ml-py). Pour plus d’informations sur la structure de ce fichier, consultez [runconfigschema.json](https://github.com/microsoft/MLOps/blob/b4bdcf8c369d188e83f40be8b748b49821f71cf2/infra-as-code/runconfigschema.json). |
| `allow_reuse` | Détermine si l’étape doit réutiliser les résultats précédents lorsqu’elle est exécutée avec les mêmes paramètres. |

L’exemple suivant contient une étape Databricks :

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        adls_test_data:
            datastore: adftestadla
            path_on_datastore: "testdata"
        blob_test_data:
            datastore: workspaceblobstore
            path_on_datastore: "dbtest"
    default_compute: mydatabricks
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "DatabricksStep"
            name: "MyDatabrickStep"
            run_name: "DatabricksRun"
            python_script_name: "train-db-local.py"
            source_directory: "D:\\scripts\\Databricks"
            num_workers: 1
            allow_reuse: true
            inputs:
                blob_test_data:
                    source: blob_test_data
            outputs:
                OutputData:
                    destination: Output4
                    datastore: workspaceblobstore
                    bind_mode: mount
```

### <a name="data-transfer-step"></a>Étape de transfert de données

| Clé YAML | Description |
| ----- | ----- |
| `compute_target` | Cible de calcul Azure Data Factory à utiliser pour cette étape. |
| `source_data_reference` | Connexion d’entrée qui sert de source pour les opérations de transfert de données. Les valeurs prises en charge sont [InputPortBinding](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding?preserve-view=true&view=azure-ml-py), [DataReference](#data-reference), [PortDataReference](/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference?preserve-view=true&view=azure-ml-py), [PipelineData](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?preserve-view=true&view=azure-ml-py), [DataSet](/python/api/azureml-core/azureml.core.dataset%28class%29?preserve-view=true&view=azure-ml-py), [DatasetDefinition](/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition?preserve-view=true&view=azure-ml-py) ou [PipelineDataset](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset?preserve-view=true&view=azure-ml-py). |
| `destination_data_reference` | Connexion d’entrée qui sert de destination pour les opérations de transfert de données. Les valeurs prises en charge sont [PipelineData](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?preserve-view=true&view=azure-ml-py) et [OutputPortBinding](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding?preserve-view=true&view=azure-ml-py). |
| `allow_reuse` | Détermine si l’étape doit réutiliser les résultats précédents lorsqu’elle est exécutée avec les mêmes paramètres. |

L’exemple suivant contient une étape de transfert de données :

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        adls_test_data:
            datastore: adftestadla
            path_on_datastore: "testdata"
        blob_test_data:
            datastore: workspaceblobstore
            path_on_datastore: "testdata"
    default_compute: adftest
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "DataTransferStep"
            name: "MyDataTransferStep"
            adla_compute_name: adftest
            source_data_reference:
                adls_test_data:
                    source: adls_test_data
            destination_data_reference:
                blob_test_data:
                    source: blob_test_data
```

### <a name="python-script-step"></a>Étape de script Python

| Clé YAML | Description |
| ----- | ----- |
| `inputs` | Les entrées peuvent être [InputPortBinding](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.inputportbinding?preserve-view=true&view=azure-ml-py), [DataReference](#data-reference), [PortDataReference](/python/api/azureml-pipeline-core/azureml.pipeline.core.portdatareference?preserve-view=true&view=azure-ml-py), [PipelineData](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?preserve-view=true&view=azure-ml-py), [DataSet](/python/api/azureml-core/azureml.core.dataset%28class%29?preserve-view=true&view=azure-ml-py), [DatasetDefinition](/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition?preserve-view=true&view=azure-ml-py) ou [PipelineDataset](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset?preserve-view=true&view=azure-ml-py). |
| `outputs` | Les sorties peuvent être [PipelineData](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?preserve-view=true&view=azure-ml-py) ou [OutputPortBinding](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding?preserve-view=true&view=azure-ml-py). |
| `script_name` | Nom du script Python (relatif à `source_directory`). |
| `source_directory` | Répertoire contenant le script, l’environnement Conda, etc. |
| `runconfig` | Chemin d’accès du fichier `.runconfig`. Ce fichier est une représentation YAML de la classe [RunConfiguration](/python/api/azureml-core/azureml.core.runconfiguration?preserve-view=true&view=azure-ml-py). Pour plus d’informations sur la structure de ce fichier, consultez [runconfig.json](https://github.com/microsoft/MLOps/blob/b4bdcf8c369d188e83f40be8b748b49821f71cf2/infra-as-code/runconfigschema.json). |
| `allow_reuse` | Détermine si l’étape doit réutiliser les résultats précédents lorsqu’elle est exécutée avec les mêmes paramètres. |

L’exemple suivant contient une étape de script Python :

```yaml
pipeline:
    name: SamplePipelineFromYaml
    parameters:
        PipelineParam1:
            type: int
            default: 3
    data_references:
        DataReference1:
            datastore: workspaceblobstore
            path_on_datastore: testfolder/sample.txt
    default_compute: cpu-cluster
    steps:
        Step1:
            runconfig: "D:\\Yaml\\default_runconfig.yml"
            parameters:
                NUM_ITERATIONS_2:
                    source: PipelineParam1
                NUM_ITERATIONS_1: 7
            type: "PythonScriptStep"
            name: "MyPythonScriptStep"
            script_name: "train.py"
            allow_reuse: True
            source_directory: "D:\\scripts\\PythonScript"
            inputs:
                InputData:
                    source: DataReference1
            outputs:
                OutputData:
                    destination: Output4
                    datastore: workspaceblobstore
                    bind_mode: mount
```

### <a name="parallel-run-step"></a>Étape d’exécution parallèle

| Clé YAML | Description |
| ----- | ----- |
| `inputs` | Les entrées peuvent être [Dataset](/python/api/azureml-core/azureml.core.dataset%28class%29?preserve-view=true&view=azure-ml-py), [DatasetDefinition](/python/api/azureml-core/azureml.data.dataset_definition.datasetdefinition?preserve-view=true&view=azure-ml-py) ou [PipelineDataset](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedataset?preserve-view=true&view=azure-ml-py). |
| `outputs` | Les sorties peuvent être [PipelineData](/python/api/azureml-pipeline-core/azureml.pipeline.core.pipelinedata?preserve-view=true&view=azure-ml-py) ou [OutputPortBinding](/python/api/azureml-pipeline-core/azureml.pipeline.core.graph.outputportbinding?preserve-view=true&view=azure-ml-py). |
| `script_name` | Nom du script Python (relatif à `source_directory`). |
| `source_directory` | Répertoire contenant le script, l’environnement Conda, etc. |
| `parallel_run_config` | Chemin d’accès du fichier `parallel_run_config.yml`. Ce fichier est une représentation YAML de la classe [ParallelRunConfig](/python/api/azureml-pipeline-steps/azureml.pipeline.steps.parallelrunconfig?preserve-view=true&view=azure-ml-py). |
| `allow_reuse` | Détermine si l’étape doit réutiliser les résultats précédents lorsqu’elle est exécutée avec les mêmes paramètres. |

L’exemple suivant contient une étape d’exécution parallèle :

```yaml
pipeline:
    description: SamplePipelineFromYaml
    default_compute: cpu-cluster
    data_references:
        MyMinistInput:
            dataset_name: mnist_sample_data
    parameters:
        PipelineParamTimeout:
            type: int
            default: 600
    steps:        
        Step1:
            parallel_run_config: "yaml/parallel_run_config.yml"
            type: "ParallelRunStep"
            name: "parallel-run-step-1"
            allow_reuse: True
            arguments:
            - "--progress_update_timeout"
            - parameter:timeout_parameter
            - "--side_input"
            - side_input:SideInputData
            parameters:
                timeout_parameter:
                    source: PipelineParamTimeout
            inputs:
                InputData:
                    source: MyMinistInput
            side_inputs:
                SideInputData:
                    source: Output4
                    bind_mode: mount
            outputs:
                OutputDataStep2:
                    destination: Output5
                    datastore: workspaceblobstore
                    bind_mode: mount
```

### <a name="pipeline-with-multiple-steps"></a>Pipeline avec plusieurs étapes 

| Clé YAML | Description |
| ----- | ----- |
| `steps` | Séquence d’une ou plusieurs définitions PipelineStep. Notez que les clés `destination` du paramètre `outputs` d’une étape deviennent les clés `source` pour le paramètre `inputs` de l’étape suivante.| 

```yaml
pipeline:
    name: SamplePipelineFromYAML
    description: Sample multistep YAML pipeline
    data_references:
        TitanicDS:
            dataset_name: 'titanic_ds'
            bind_mode: download
    default_compute: cpu-cluster
    steps:
        Dataprep:
            type: "PythonScriptStep"
            name: "DataPrep Step"
            compute: cpu-cluster
            runconfig: ".\\default_runconfig.yml"
            script_name: "prep.py"
            arguments:
            - '--train_path'
            - output:train_path
            - '--test_path'
            - output:test_path
            allow_reuse: True
            inputs:
                titanic_ds:
                    source: TitanicDS
                    bind_mode: download
            outputs:
                train_path:
                    destination: train_csv
                    datastore: workspaceblobstore
                test_path:
                    destination: test_csv
        Training:
            type: "PythonScriptStep"
            name: "Training Step"
            compute: cpu-cluster
            runconfig: ".\\default_runconfig.yml"
            script_name: "train.py"
            arguments:
            - "--train_path"
            - input:train_path
            - "--test_path"
            - input:test_path
            inputs:
                train_path:
                    source: train_csv
                    bind_mode: download
                test_path:
                    source: test_csv
                    bind_mode: download

```

## <a name="schedules"></a>Planifications

Lors de la définition de la planification d’un pipeline, ce dernier peut être déclenché par le magasin de données ou récurrent selon un intervalle de temps. Voici les clés utilisées pour définir une planification :

| Clé YAML | Description |
| ----- | ----- |
| `description` | Description de la planification. |
| `recurrence` | Contient les paramètres de périodicité, si la planification est récurrente. |
| `pipeline_parameters` | Tous les paramètres requis par le pipeline. |
| `wait_for_provisioning` | Indique s’il faut attendre la fin de l’approvisionnement de la planification. |
| `wait_timeout` | Nombre de secondes à attendre avant l’expiration du délai. |
| `datastore_name` | Magasin de données à surveiller pour les blobs modifiés/ajoutés. |
| `polling_interval` | Durée (en minutes) entre l’interrogation des blobs modifiés/ajoutés. Valeur par défaut : 5 minutes. Pris en charge uniquement pour les planifications de magasin de données. |
| `data_path_parameter_name` | Nom du paramètre de pipeline relatif au chemin d’accès des données à définir avec le chemin d’accès des blobs modifiés. Pris en charge uniquement pour les planifications de magasin de données. |
| `continue_on_step_failure` | Indique s’il faut poursuivre l’exécution des autres étapes du PipelineRun envoyé si une étape échoue. S’il est fourni, remplace le paramètre `continue_on_step_failure` du pipeline.
| `path_on_datastore` | facultatif. Chemin d’accès sur le magasin de données à surveiller pour les blobs modifiés/ajoutés. Le chemin d’accès se trouve sous le conteneur du magasin de données, de sorte que le chemin d’accès réel surveillé par la planification est container/`path_on_datastore`. S’il n’y en a aucun, le conteneur du magasin de données est surveillé. Les ajouts/modifications effectués dans un sous-dossier du `path_on_datastore` ne sont pas surveillés. Pris en charge uniquement pour les planifications de magasin de données. |

L’exemple suivant contient la définition d’une planification déclenchée par un magasin de données :

```yaml
Schedule: 
      description: "Test create with datastore" 
      recurrence: ~ 
      pipeline_parameters: {} 
      wait_for_provisioning: True 
      wait_timeout: 3600 
      datastore_name: "workspaceblobstore" 
      polling_interval: 5 
      data_path_parameter_name: "input_data" 
      continue_on_step_failure: None 
      path_on_datastore: "file/path" 
```

Lors de la définition d’une **planification récurrente** , utilisez les clés suivantes sous `recurrence` :

| Clé YAML | Description |
| ----- | ----- |
| `frequency` | Fréquence de récurrence de la planification. Les valeurs valides sont `"Minute"`, `"Hour"`, `"Day"`, `"Week"` ou `"Month"`. |
| `interval` | Fréquence de déclenchement de la planification. La valeur entière est le nombre d’unités de temps à attendre avant que la planification se déclenche à nouveau. |
| `start_time` | Heure de début de la planification. Le format de chaîne de la valeur est `YYYY-MM-DDThh:mm:ss`. Si aucune heure de début n’est indiquée, la première charge de travail est exécutée instantanément et les charges de travail futures sont exécutées en fonction de la planification. Si l’heure de début est dans le passé, la première charge de travail est exécutée au moment d’exécution calculé suivant. |
| `time_zone` | Fuseau horaire de l’heure de début. Si aucun fuseau horaire n’est indiqué, le temps universel coordonné (UTC) est utilisé. |
| `hours` | Si la valeur de `frequency` est `"Day"` ou `"Week"`, vous pouvez spécifier un ou plusieurs entiers compris entre 0 et 23, séparés par des virgules, pour les heures de la journée où le pipeline doit s’exécuter. Seuls `time_of_day` ou `hours` et `minutes` peuvent être utilisés. |
| `minutes` | Si la valeur de `frequency` est `"Day"` ou `"Week"`, vous pouvez spécifier un ou plusieurs entiers compris entre 0 et 59, séparés par des virgules, pour les minutes de l’heure où le pipeline doit s’exécuter. Seuls `time_of_day` ou `hours` et `minutes` peuvent être utilisés. |
| `time_of_day` | Si la valeur de `frequency` est `"Day"` ou `"Week"`, vous pouvez spécifier un moment de la journée où la planification doit s’exécuter. Le format de chaîne de la valeur est `hh:mm`. Seuls `time_of_day` ou `hours` et `minutes` peuvent être utilisés. |
| `week_days` | Si la valeur de `frequency` est `"Week"`, vous pouvez spécifier un ou plusieurs jours, séparés par des virgules, où la planification doit s’exécuter. Les valeurs valides sont `"Monday"`, `"Tuesday"`, `"Wednesday"`, `"Thursday"`, `"Friday"`, `"Saturday"` et `"Sunday"`. |

L’exemple suivant contient la définition d’une planification récurrente :

```yaml
Schedule: 
    description: "Test create with recurrence" 
    recurrence: 
        frequency: Week # Can be "Minute", "Hour", "Day", "Week", or "Month". 
        interval: 1 # how often fires 
        start_time: 2019-06-07T10:50:00 
        time_zone: UTC 
        hours: 
        - 1 
        minutes: 
        - 0 
        time_of_day: null 
        week_days: 
        - Friday 
    pipeline_parameters: 
        'a': 1 
    wait_for_provisioning: True 
    wait_timeout: 3600 
    datastore_name: ~ 
    polling_interval: ~ 
    data_path_parameter_name: ~ 
    continue_on_step_failure: None 
    path_on_datastore: ~ 
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [utiliser l’extension CLI pour Azure Machine Learning](reference-azure-machine-learning-cli.md).