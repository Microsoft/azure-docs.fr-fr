---
title: Présentation de la formation distribuée
titleSuffix: Azure Machine Learning
description: Découvrez quel type de formation distribuée Azure Machine Learning prend en charge et les intégrations d’infrastructure open source disponibles pour la formation distribuée.
services: machine-learning
ms.service: machine-learning
author: nibaccam
ms.author: nibaccam
ms.subservice: core
ms.topic: conceptual
ms.date: 03/27/2020
ms.openlocfilehash: 4e1972e0530656bf6cc0e6319191ab4ebbfbfddb
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93358573"
---
# <a name="distributed-training-with-azure-machine-learning"></a>Formation distribuée avec Azure Machine Learning

Dans cet article, vous allez découvrir la formation distribuée et la façon dont Azure Machine Learning la prend en charge pour les modèles de Deep Learning. 

Dans la formation distribuée, la charge de travail visant à effectuer l’apprentissage d’un modèle est fractionnée et partagée entre plusieurs mini-processeurs, appelés nœuds Worker. Ces nœuds Worker fonctionnent en parallèle pour accélérer la formation du modèle. La formation distribuée peut être utilisée pour les modèles de ML traditionnels, mais elle est mieux adaptée aux tâches gourmandes en ressources de calcul et en temps, par exemple le [Deep Learning](concept-deep-learning-vs-machine-learning.md) pour la formation de réseaux neuronaux profonds. 

## <a name="deep-learning-and-distributed-training"></a>Deep Learning et formation distribuée 

Il existe deux principaux types de formation distribuée : le [parallélisme des données](#data-parallelism) et le [parallélisme de modèle](#model-parallelism). Pour la formation distribuée sur des modèles de Deep Learning, le [Kit de développement logiciel (SDK) Azure Machine Learning dans Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) prend en charge les intégrations aux infrastructures populaires, PyTorch et TensorFlow. Les deux infrastructures utilisent le parallélisme des données pour la formation distribuée et peuvent exploiter [horovod](https://horovod.readthedocs.io/en/latest/summary_include.html) pour optimiser les vitesses de calcul. 

* [Entraînement distribué avec PyTorch](how-to-train-pytorch.md#distributed-training)

* [Entraînement distribué avec TensorFlow](how-to-train-tensorflow.md#distributed-training)

Pour les modèles de ML qui ne nécessitent pas de formation distribuée, consultez [Effectuer l’apprentissage de modèles avec Azure Machine Learning](concept-train-machine-learning-model.md#python-sdk) pour les différentes façons d’effectuer l’apprentissage des modèles à l’aide du Kit de développement logiciel (SDK) Python.

## <a name="data-parallelism"></a>Parallélisme des données

De ces deux approches de formation distribuée, le parallélisme des données est la plus facile à mettre en œuvre et suffit pour la plupart des cas d’usage.

Dans cette approche, les données sont divisées en partitions, où le nombre de partitions est égal au nombre total de nœuds disponibles dans le cluster de calcul. Le modèle est copié dans chacun de ces nœuds Worker, et chaque Worker opère sur son propre sous-ensemble de données. Gardez à l’esprit que chaque nœud doit avoir la capacité de prendre en charge le modèle en cours de formation, c’est-à-dire que le modèle doit s’adapter entièrement à chaque nœud. Le diagramme suivant fournit une démonstration visuelle de cette approche.

![Data-parallelism-concept-diagram](./media/concept-distributed-training/distributed-training.svg)

Chaque nœud calcule indépendamment les erreurs entre ses prédictions pour ses exemples de formation et les sorties étiquetées. À son tour, chaque nœud met à jour son modèle en fonction des erreurs et doit communiquer toutes ses modifications aux autres nœuds pour mettre à jour les modèles correspondants. Cela signifie que les nœuds Worker doivent synchroniser les paramètres de modèle, ou gradients, à la fin du calcul par lot pour s’assurer qu’ils effectuent l’apprentissage d’un modèle cohérent. 

## <a name="model-parallelism"></a>Parallélisme de modèle

Dans le parallélisme de modèle, également appelé parallélisme de réseau, le modèle est segmenté en différentes parties qui peuvent s’exécuter simultanément dans des nœuds différents, et chacune s’exécutera sur les mêmes données. L’extensibilité de cette méthode dépend du degré de parallélisation des tâches de l’algorithme et elle est plus complexe à implémenter que le parallélisme des données. 

Dans le parallélisme de modèle, les nœuds Worker doivent uniquement synchroniser les paramètres partagés, généralement une fois pour chaque étape de propagation ascendante ou descendante. En outre, les modèles plus volumineux ne sont pas un problème puisque chaque nœud opère sur une sous-section du modèle sur les mêmes données de formation.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [utiliser des cibles de calcul pour l’entraînement des modèles](how-to-set-up-training-targets.md) avec le SDK Python.
* Pour obtenir un exemple technique, consultez le [scénario d’architecture de référence](/azure/architecture/reference-architectures/ai/training-deep-learning).
* [Effectuer l’apprentissage de modèles de ML avec TensorFlow](how-to-train-tensorflow.md).
* [Effectuer l’apprentissage de modèles de ML avec PyTorch](how-to-train-pytorch.md).