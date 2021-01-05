---
title: Frameworks Deep Learning et d’intelligence artificielle
titleSuffix: Azure Data Science Virtual Machine
description: Infrastructures et outils de Deep Learning disponibles sur Azure Data Science Virtual Machine.
keywords: outils de science des données, machine virtuelle science des données, outils pour la science des données, science des données linux
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
ms.custom: devx-track-python
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 07/17/2020
ms.openlocfilehash: c8c6ee3d5eecc02ac921d7b10f2394380af684c2
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93309029"
---
# <a name="deep-learning-and-ai-frameworks-for-the-azure-data-science-vm"></a>Infrastructures de Deep learning et d’intelligence artificielle pour Azure Data Science Virtual Machine
Les Infrastructures de Deep learning sur Data Science Virtual Machine (DSVM) sont répertoriées ci-dessous.

## <a name="caffe"></a>[Caffe](https://github.com/BVLC/caffe)

| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge | |
| Éditions DSVM prises en charge      | Ubuntu 16.04    |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | Caffe est installé dans `/opt/caffe`.   Des exemples sont disponibles dans `/opt/caffe/examples`.|
| Comment l’exécuter      | Utilisez X2Go pour vous connecter à votre machine virtuelle, puis démarrez un nouveau terminal et entrez ce qui suit :<br/>`cd /opt/caffe/examples`<br/>`source activate root`<br/>`jupyter notebook`<br/><br/>Une nouvelle fenêtre de navigateur s’ouvre avec les exemples de bloc-notes. Les fichiers binaires sont installés dans /opt/caffe/build/install/bin.<br/><br/>La version installée de Caffe nécessite Python 2.7 et ne fonctionne pas avec Python 3.5, activé par défaut. Pour basculer vers Python 2.7, exécutez `source activate root` afin de passer à l’environnement Anaconda.|    

## <a name="caffe2"></a>[Caffe2](https://github.com/caffe2/caffe2)

| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge | |
| Éditions DSVM prises en charge      | Ubuntu 16.04     |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | Caffe2 est installé dans l’environnement conda Python 2.7 (root). |
| Comment l’exécuter      | Terminal : Démarrez Python, puis importez Caffe2. <br/> * JupyterHub : [Connectez-vous à JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine), puis accédez au répertoire Caffe2 pour rechercher des exemples de notebooks. Certains bloc-notes exigent que la racine Caffe2 soit définie dans le code Python. Entrez /opt/caffe2. |

## <a name="chainer"></a>[Chainer](https://chainer.org/)

| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge | 5.2 |
| Éditions DSVM prises en charge      | Ubuntu 16.04    |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | Chainer est installé dans l’environnement Python 3.5. |
| Comment l’exécuter      | Terminal : Activez l’environnement Python 3.5, exécutez `python`, puis `import chainer`. <br/> * JupyterHub : [Connectez-vous à JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine), puis accédez au répertoire Chainer pour rechercher des exemples de notebooks.| 

## <a name="cuda-cudnn-nvidia-driver"></a>[Pilote CUDA, cuDNN, NVIDIA](https://developer.nvidia.com/cuda-toolkit)

| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge | 10.0.130|
| Éditions DSVM prises en charge      | Windows Server 2019<br>Ubuntu 18.04<br> Windows 2016<br> Ubuntu 16.04  |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  |_nvidia-smi_ est disponible sur le chemin système.  |
| Comment l’exécuter      | Ouvrez une invite de commandes (sur Windows) ou un terminal (sur Linux), puis exécutez _nvidia-smi_. |


## <a name="horovod"></a>[Horovod](https://github.com/uber/horovod)

| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge | 0.16.1|
| Éditions DSVM prises en charge      | Ubuntu 18.04<br> Ubuntu 16.04   |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | Horovod est installé dans l’environnement Python 3.5 |
| Comment l’exécuter      | Activez l’environnement approprié sur le terminal, puis exécutez Python. |

## <a name="keras"></a>[Keras](https://keras.io/)

| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge | 2.2.4 |
| Éditions DSVM prises en charge      | Windows Server 2019<br>Ubuntu 18.04<br> Windows 2016 <br> Ubuntu 16.04   |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | Keras est installé dans l’environnement Python 3.6 sous Windows et dans l’environnement Python 3.5 sous Linux |
| Comment l’exécuter      | Activez l’environnement approprié sur le terminal, puis exécutez Python. |

## <a name="microsoft-cognitive-toolkit-cntk"></a>[Microsoft Cognitive Toolkit (CNTK)](/cognitive-toolkit/)

| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge | 2.5.1 |
| Éditions DSVM prises en charge      | Windows 2016 <br> Ubuntu 16.04   |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | CNTK est installé dans Python 3.6 sous [Windows 2016](dsvm-tools-languages.md#python-windows-server-2016-edition) et dans l’environnement Python 3.5 sous [Linux](./dsvm-tools-languages.md#python-linux-edition). |
| Comment l’exécuter      | Terminal : Activez l’environnement approprié, puis exécutez Python. <br/>Jupyter : Connectez-vous à [Jupyter](provision-vm.md) ou [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine), puis ouvrez le répertoire CNTK pour obtenir des exemples. |

## <a name="mxnet"></a>[MXNet](https://mxnet.apache.org/)
| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge | 1.3.0 |
| Éditions DSVM prises en charge      | Windows 2016 <br> Ubuntu 16.04    |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | MXNet est installé dans `C:\dsvm\tools\mxnet` sur Windows et `/dsvm/tools/mxnet` sur Ubuntu. Les liaisons Python sont installées en Python 3.6 sur [Windows 2016](dsvm-tools-languages.md#python-windows-server-2016-edition) et en Python 3.5 sur [Linux](./dsvm-tools-languages.md#python-linux-edition)). Les liaisons R sont également installées sur la machine DSVM Ubuntu. |
| Comment l’exécuter      | Terminal : Activez l’environnement conda approprié, puis exécutez `import mxnet`. <br/>Jupyter : Connectez-vous à [Jupyter](provision-vm.md#access-the-dsvm) ou à [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine), puis ouvrez le répertoire `mxnet` pour obtenir des exemples. |

## <a name="mxnet-model-server"></a>[MXNet Model Server](https://github.com/awslabs/mxnet-model-server#quick-start)

| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge | 1.0.1 |
| Éditions DSVM prises en charge      | Windows 2016 <br> Ubuntu 16.04    |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | MXNet Model Server est installé en Python 3.6 sur [Windows 2016](dsvm-tools-languages.md#python-windows-server-2016-edition) et en Python 3.5 sur [Linux](./dsvm-tools-languages.md#python-linux-edition). |
| Comment l’exécuter      | Terminal : Exécutez `sudo systemctl stop jupyterhub` pour arrêter le service JupyterHub en premier, car les deux écoutent le même port. Ensuite, activez l’environnement conda approprié et exécutez `mxnet-model-server --start --models squeezenet=https://s3.amazonaws.com/model-server/model_archive_1.0/squeezenet_v1.1.mar`. |

## <a name="nvidia-system-management-interface-nvidia-smi"></a>[NVidia System Management Interface (nvidia-smi)](https://developer.nvidia.com/nvidia-system-management-interface)

| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge |  |
| Éditions DSVM prises en charge      | Windows Server 2019<br>Ubuntu 18.04<br> Windows 2016<br> Ubuntu 16.04   |
| Pour quoi faire ? | Outil NVIDIA pour interroger l’activité du GPU |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | `nvidia-smi` se trouve sur le chemin système. |
| Comment l’exécuter      | Sur une machine virtuelle **comprenant des GPU** , ouvrez une invite de commandes (sur Windows) ou un terminal (sur Linux), puis exécutez `nvidia-smi`. |

## <a name="pytorch"></a>[PyTorch](https://pytorch.org/)

| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge | 1.2.0 (Ubuntu 16.04), 1.4.0 (Ubuntu 18.04, Windows 2019) |
| Éditions DSVM prises en charge      | Windows Server 2019<br>Ubuntu 18.04<br> Ubuntu 16.04 |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | Installé dans [Python 3.5](dsvm-tools-languages.md#python-linux-edition). Des exemples de notebooks Jupyter sont inclus, et des exemples sont disponibles dans /dsvm/samples/pytorch. |
| Comment l’exécuter      | Terminal : Activez l’environnement approprié, puis exécutez Python.<br/>* [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine) : Connectez-vous, puis ouvrez le répertoire PyTorch pour obtenir des exemples.  |

## <a name="tensorflow"></a>[TensorFlow](https://www.tensorflow.org/)

| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge | 1.13 |
| Éditions DSVM prises en charge      | Windows Server 2019<br>Ubuntu 18.04<br> Windows 2016 <br> Ubuntu 16.04 |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | Installé dans l’environnement Python 3.5 sous [Linux](dsvm-tools-languages.md#python-linux-edition) et dans l’environnement Python 3.6 sous [Windows 2016](dsvm-tools-languages.md#python-windows-server-2016-edition) |
| Comment l’exécuter      | Terminal : Activez l’environnement approprié, puis exécutez Python. <br/> * Jupyter : Connectez-vous à [Jupyter](provision-vm.md) ou [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine), puis ouvrez le répertoire TensorFlow pour obtenir des exemples.   |

## <a name="tensorflow-serving"></a>[TensorFlow Serving](https://www.tensorflow.org/serving/)

| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge | 1.12 |
| Éditions DSVM prises en charge      | Ubuntu 16.04 |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | tensorflow_model_server est disponible sur le terminal. |
| Comment l’exécuter      |  D’autres exemples sont disponibles [en ligne](https://www.tensorflow.org/serving/).   |


## <a name="theano"></a>[Theano](https://github.com/Theano/Theano)

| Category | Valeur |
| ------------- | ------------- |
| Version(s) prise(s) en charge | 1.0.3 |
| Éditions DSVM prises en charge      | Ubuntu 16.04 |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  |Theano est installé dans les environnements Python 2.7 ( _root_ ) et Python 3.5 ( _py35_ ). |
| Comment l’exécuter      |  Terminal : Activez la version de Python souhaitée (root ou py35), exécutez Python, puis importez Theano.<br/>* Jupyter : Sélectionnez le noyau Python 2.7 ou 3.5, puis importez Theano.  <br/>Pour contourner un bogue récent MKL (Math Kernel Library), vous devez d’abord définir la couche de threads MKL comme suit :<br/><br/>`export MKL_THREADING_LAYER=GNU`  |