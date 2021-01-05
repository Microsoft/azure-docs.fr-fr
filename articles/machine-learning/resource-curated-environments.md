---
title: Environnements organisés
titleSuffix: Azure Machine Learning
description: En savoir plus sur les environnements organisés Azure Machine Learning, un ensemble d’environnements préconfigurés qui permettent de réduire les temps de préparation du déploiement et des expérimentations.
services: machine-learning
author: luisquintanilla
ms.author: luquinta
ms.reviewer: luquinta
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.date: 11/16/2020
ms.openlocfilehash: 0e6817e03c5e5363ad925367b0632e26fc2da4a2
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94701396"
---
# <a name="azure-machine-learning-curated-environments"></a>Environnements organisés Azure Machine Learning

Cet article liste les environnements organisés dans Azure Machine Learning, ainsi que les packages et les canaux qui y sont préinstallés. Les environnements organisés sont fournis par Azure Machine Learning et sont disponibles dans votre espace de travail par défaut. Ils sont sauvegardés par des images Docker mises en cache, ce qui réduit le coût de préparation de l’exécution et permet un déploiement plus rapide. Utilisez ces environnements pour prendre rapidement en main les différents infrastructures d’apprentissage automatique.

> [!NOTE]
> Cette liste a été mise à jour en septembre 2020. Utilisez le SDK Python pour récupérer la liste la plus à jour. Pour plus d’informations, consultez l’[article sur les environnements](./how-to-use-environments.md#use-a-curated-environment).

## <a name="azureml-automl"></a>AzureML-AutoML

**Canaux de packages :**

* anaconda
* conda-forge
* pytorch

**Packages Conda :**

* python
* numpy
* scikit-learn
* pandas
* py-xgboost
* fbprophet
* jours fériés
* setuptools-git
* psutil

**Packages PIP :**

* azureml-core
* azureml-pipeline-core
* azureml-telemetry
* azureml-defaults
* azureml-interpret
* azureml-explain-model
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-train-automl-runtime
* inference-schema
* py-cpuinfo

## <a name="azureml-automl-dnn"></a>AzureML-AutoML-DNN

**Canaux de packages :**

* anaconda
* conda-forge
* pytorch

**Packages Conda :**

* python
* numpy
* scikit-learn
* pandas
* py-xgboost
* fbprophet
* jours fériés
* setuptools-git
* pytorch
* cudatoolkit
* psutil

**Packages PIP :**

* azureml-core
* azureml-pipeline-core
* azureml-telemetry
* azureml-defaults
* azureml-interpret
* azureml-explain-model
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-train-automl-runtime
* inference-schema
* pytorch-transformers
* spacy
* en_core_web_sm
* py-cpuinfo

## <a name="azureml-automl-dnn-gpu"></a>AzureML-AutoML-DNN-GPU

**Canaux de packages :**

* anaconda
* conda-forge
* pytorch

**Packages Conda :**

* python
* numpy
* scikit-learn
* pandas
* fbprophet
* jours fériés
* setuptools-git
* pytorch
* cudatoolkit
* psutil

**Packages PIP :**

* azureml-core
* azureml-pipeline-core
* azureml-telemetry
* azureml-defaults
* azureml-interpret
* azureml-explain-model
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-train-automl-runtime
* inference-schema
* horovod
* pytorch-transformers
* spacy
* en_core_web_sm
* py-cpuinfo

## <a name="azureml-automl-dnn-vision-gpu"></a>AzureML-AutoML-DNN-Vision-GPU

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-dataset-runtime
* azureml-contrib-dataset
* azureml-telemetry
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-defaults
* azureml-interpret
* azureml-explain-model
* azureml-train-automl-runtime
* azureml-train-automl
* azureml-contrib-automl-dnn-vision

## <a name="azureml-automl-gpu"></a>AzureML-AutoML-GPU

**Canaux de packages :**

* anaconda
* conda-forge
* pytorch

**Packages Conda :**

* python
* numpy
* scikit-learn
* pandas
* fbprophet
* jours fériés
* setuptools-git
* psutil

**Packages PIP :**

* azureml-core
* azureml-pipeline-core
* azureml-telemetry
* azureml-defaults
* azureml-interpret
* azureml-explain-model
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-train-automl-runtime
* inference-schema
* py-cpuinfo

## <a name="azureml-chainer-510-cpu"></a>AzureML-Chainer-5.1.0-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* chainer
* mpi4py

## <a name="azureml-chainer-510-gpu"></a>AzureML-chainer-5.1.0-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* chainer
* cupy-cuda90
* mpi4py

## <a name="azureml-dask-cpu"></a>AzureML-Dask-CPU

**Canaux de packages :**

* conda-forge
* pytorch
* defaults

**Packages Conda :**

* python

**Packages PIP :**

* adlfs
* azureml-core
* azureml-dataset-runtime
* dask[complete]
* dask-ml[complete]
* distributed
* fastparquet
* fsspec
* joblib
* jupyterlab
* lz4
* mpi4py
* notebook
* pyarrow

## <a name="azureml-dask-gpu"></a>AzureML-Dask-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python
* matplotlib

**Packages PIP :**

* azureml-defaults
* adlfs
* azureml-core
* dask[complete]
* dask-ml[complete]
* distributed
* fastparquet
* fsspec
* joblib
* jupyterlab
* lz4
* mpi4py
* notebook
* pyarrow

## <a name="azureml-hyperdrive-forecastdnn"></a>AzureML-Hyperdrive-ForecastDNN

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-pipeline-core
* azureml-telemetry
* azureml-defaults
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-train-automl-runtime
* azureml-contrib-automl-dnn-forecasting

## <a name="azureml-minimal"></a>AzureML-Minimal

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults

## <a name="azureml-pyspark-mmlspark-015"></a>AzureML-PySpark-MmlSpark-0.15

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core

## <a name="azureml-pytorch-10-cpu"></a>AzureML-PyTorch-1.0-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod

## <a name="azureml-pytorch-10-gpu"></a>AzureML-PyTorch-1.0-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod

## <a name="azureml-pytorch-11-cpu"></a>AzureML-PyTorch-1.1-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-11-gpu"></a>AzureML-PyTorch-1.1-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-12-cpu"></a>AzureML-PyTorch-1.2-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-12-gpu"></a>AzureML-PyTorch-1.2-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-13-cpu"></a>AzureML-PyTorch-1.3-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-13-gpu"></a>AzureML-PyTorch-1.3-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-14-cpu"></a>AzureML-PyTorch-1.4-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-14-gpu"></a>AzureML-PyTorch-1.4-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-15-cpu"></a>AzureML-PyTorch-1.5-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-15-gpu"></a>AzureML-PyTorch-1.5-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-16-cpu"></a>AzureML-PyTorch-1.6-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-pytorch-16-gpu"></a>AzureML-PyTorch-1.6-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* torch
* torchvision
* mkl
* horovod
* tensorboard
* future

## <a name="azureml-scikit-learn-0203"></a>AzureML-Scikit-learn-0.20.3

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* scikit-learn
* scipy
* joblib

## <a name="azureml-tensorflow-110-cpu"></a>AzureML-TensorFlow-1.10-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow
* horovod

## <a name="azureml-tensorflow-110-gpu"></a>AzureML-TensorFlow-1.10-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow-gpu
* horovod

## <a name="azureml-tensorflow-112-cpu"></a>AzureML-TensorFlow-1.12-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow
* horovod

## <a name="azureml-tensorflow-112-gpu"></a>AzureML-TensorFlow-1.12-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow-gpu
* horovod

## <a name="azureml-tensorflow-113-cpu"></a>AzureML-TensorFlow-1.13-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow
* horovod

## <a name="azureml-tensorflow-113-gpu"></a>AzureML-TensorFlow-1.13-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow-gpu
* horovod

## <a name="azureml-tensorflow-20-cpu"></a>AzureML-TensorFlow-2.0-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow
* horovod

## <a name="azureml-tensorflow-20-gpu"></a>AzureML-TensorFlow-2.0-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow-gpu
* horovod

## <a name="azureml-tensorflow-21-cpu"></a>AzureML-TensorFlow-2.1-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow
* horovod

## <a name="azureml-tensorflow-21-gpu"></a>AzureML-TensorFlow-2.1-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow-gpu
* horovod

## <a name="azureml-tensorflow-22-cpu"></a>AzureML-TensorFlow-2.2-CPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow
* horovod

## <a name="azureml-tensorflow-22-gpu"></a>AzureML-TensorFlow-2.2-GPU

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* tensorflow-gpu
* horovod

## <a name="azureml-tutorial"></a>AzureML-Tutorial

**Canaux de packages :**

* anaconda
* conda-forge

**Packages Conda :**

* python
* pandas
* numpy
* tqdm
* scikit-learn
* matplotlib

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-telemetry
* azureml-train-restclients-hyperdrive
* azureml-train-core
* azureml-widgets
* azureml-pipeline-core
* azureml-pipeline-steps
* azureml-opendatasets
* azureml-automl-core
* azureml-automl-runtime
* azureml-train-automl-client
* azureml-train-automl-runtime
* azureml-train-automl
* azureml-train
* azureml-sdk
* azureml-interpret
* azureml-tensorboard
* azureml-mlflow
* mlflow
* sklearn-pandas

## <a name="azureml-vowpalwabbit-880"></a>AzureML-VowpalWabbit-8.8.0

**Canaux de packages :**

* conda-forge

**Packages Conda :**

* python

**Packages PIP :**

* azureml-core
* azureml-defaults
* azureml-dataset-runtime[fuse,pandas]
