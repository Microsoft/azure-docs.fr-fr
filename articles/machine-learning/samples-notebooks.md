---
title: Exemples de notebooks Jupyter
titleSuffix: Azure Machine Learning
description: Découvrez comment trouver et utiliser les notebooks Jupyter qui sont conçus pour vous aider à explorer le SDK, et qui servent de modèles pour vos propres projets Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: sample
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.date: 03/05/2020
ms.custom: seodec18
ms.openlocfilehash: 38d0c350718480506e1267c161a85505ece4b69d
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452255"
---
# <a name="explore-azure-machine-learning-with-jupyter-notebooks"></a>Explorer Azure Machine Learning avec des notebooks Jupyter

> [!NOTE] 
> Vous trouverez un dépôt d’exemples créés par la communauté à l’adresse https://github.com/Azure/azureml-examples.

L’[exemple de dépôt Azure Machine Learning Notebooks](https://github.com/azure/machinelearningnotebooks) contient les derniers exemples du Kit de développement logiciel (SDK) Python Azure Machine Learning. Ces notebooks Jupyter sont conçus pour vous aider à explorer le SDK et servent de modèles pour vos propres projets Machine Learning.

Cet article vous montre comment accéder au dépôt dans les environnements suivants :

- [Instance de calcul Azure Machine Learning](#notebookvm)
- [Utiliser son propre serveur de notebooks](#byo)
- [Data Science Virtual Machine](#dsvm) (Machine virtuelle Science des données)

> [!NOTE]
> Une fois que vous avez cloné le dépôt, vous voyez que des notebooks de tutoriel se trouvent dans le dossier **tutorials**, et que des notebooks relatifs à certaines fonctionnalités se trouvent dans le dossier **how-to-use-azureml**.

<a name="notebookvm"></a>
## <a name="get-samples-on-azure-machine-learning-compute-instance"></a>Obtenir des exemples d’instance de calcul Azure Machine Learning

Pour bien démarrer avec les exemples, le plus simple est de suivre le [Tutoriel : Configurer l'environnement et l'espace de travail](tutorial-1st-experiment-sdk-setup.md). Une fois que vous l’aurez suivi, vous disposerez d’un serveur de notebook dédié dans lequel auront été préchargés le SDK et l’exemple de dépôt. Aucune installation ni aucun téléchargement ne sont nécessaires.

<a name="byo"></a>

## <a name="get-samples-on-your-notebook-server"></a>Recevoir des exemples sur votre serveur de notebook

Si vous souhaitez utiliser votre propre serveur de notebooks pour le développement local, effectuez les étapes suivantes :

[!INCLUDE [aml-your-server](../../includes/aml-your-server.md)]

Ces instructions permettent d’installer les packages de base du SDK qui sont nécessaires pour le guide de démarrage rapide et les notebooks du tutoriel. Les autres exemples de notebooks peuvent nécessiter l’installation de composants supplémentaires. Pour plus d’informations, consultez [Installer le kit SDK Azure Machine Learning pour Python](/python/api/overview/azure/ml/install).

<a name="dsvm"></a>
## <a name="get-samples-on-dsvm"></a>Recevoir des exemples sur DSVM

La machine virtuelle DSVM (Data Science Virtual Machine) est une image de machine virtuelle personnalisée spécialement conçue pour la science des données. Lorsque vous [créez une machine virtuelle DSVM](how-to-configure-environment.md#dsvm), le SDK et le serveur de notebooks sont installés et configurés automatiquement. Toutefois, vous devrez créer un espace de travail et cloner l’exemple de dépôt.

[!INCLUDE [aml-dsvm-server](../../includes/aml-dsvm-server.md)]

## <a name="next-steps"></a>Étapes suivantes

Explorez les [exemples de notebooks](https://github.com/Azure/MachineLearningNotebooks) pour découvrir ce que permet de faire Azure Machine Learning.

Pour obtenir d’autres exemples de projets GitHub, consultez les dépôts suivantes :
+ [Azure/azureml-examples](https://github.com/Azure/azureml-examples)
+ [Microsoft/MLOps](https://github.com/Microsoft/MLOps)
+ [Microsoft/MLOpsPython](https://github.com/microsoft/MLOpsPython)

Essayez ces tutoriels :

- [Entraîner un modèle de classification d’images avec MNIST](tutorial-train-models-with-aml.md)

- [Préparer des données et utiliser le machine learning automatisé pour entraîner un modèle de régression avec le jeu de données NYC Taxi](tutorial-auto-train-models.md)
