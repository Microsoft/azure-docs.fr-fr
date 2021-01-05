---
title: Fichier include
description: Fichier include
services: machine-learning
author: sdgilley
ms.service: machine-learning
ms.author: sgilley
manager: cgronlund
ms.custom: include file
ms.topic: include
ms.date: 09/17/2020
ms.openlocfilehash: 87b1f4ab7b7091970d7bb76ae1e00b06549fb0b4
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96152573"
---
**Les cibles de calcul peuvent être réutilisées d’un travail de formation à l’autre**. Par exemple, une fois que vous avez joint une machine virtuelle distante à votre espace de travail, vous pouvez la réutiliser pour plusieurs travaux. Pour les pipelines de Machine Learning, utilisez l’[étape de pipeline](/python/api/azureml-pipeline-steps/azureml.pipeline.steps?preserve-view=true&view=azure-ml-py) appropriée pour chaque cible de calcul.

Vous pouvez utiliser n’importe laquelle des ressources suivantes pour une cible de calcul de formation pour la plupart des travaux. Certaines ressources ne peuvent pas être utilisées pour l’apprentissage automatique automatisé, des pipelines d’apprentissage automatique ou un concepteur.

|&nbsp;Cibles de formation|[Machine learning automatisé](../articles/machine-learning/concept-automated-ml.md) | [Pipelines d’apprentissage automatique](../articles/machine-learning/concept-ml-pipelines.md) | [Concepteur Azure Machine Learning](../articles/machine-learning/concept-designer.md)
|----|:----:|:----:|:----:|
|[Ordinateur local](../articles/machine-learning/how-to-attach-compute-targets.md#local)| Oui | &nbsp; | &nbsp; |
|[Cluster de calcul Azure Machine Learning](../articles/machine-learning/how-to-create-attach-compute-cluster.md)| Oui | Oui | Oui |
|[Instance de calcul Azure Machine Learning](../articles/machine-learning/how-to-create-manage-compute-instance.md) | Oui (via un Kit de développement logiciel (SDK))  | Oui |  |
|[Machine virtuelle distante](../articles/machine-learning/how-to-attach-compute-targets.md#vm) | Oui  | Oui | &nbsp; |
|[Azure&nbsp;Databricks](../articles/machine-learning/how-to-attach-compute-targets.md#databricks)| Oui (Kit de développement logiciel (SDK) en mode local uniquement) | Oui | &nbsp; |
|[Service Analytique Azure Data Lake](../articles/machine-learning/how-to-attach-compute-targets.md#adla) | &nbsp; | Oui | &nbsp; |
|[Azure HDInsight](../articles/machine-learning/how-to-attach-compute-targets.md#hdinsight) | &nbsp; | Oui | &nbsp; |
|[Azure Batch](../articles/machine-learning/how-to-attach-compute-targets.md#azbatch) | &nbsp; | Oui | &nbsp; |
