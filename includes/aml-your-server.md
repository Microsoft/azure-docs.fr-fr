---
title: Fichier Include
description: Fichier Include
services: machine-learning
author: sdgilley
ms.service: machine-learning
ms.author: sgilley
manager: cgronlund
ms.custom: include file
ms.topic: include
ms.date: 03/05/2020
ms.openlocfilehash: 6e7580abfd113a279e2223b9cc9665f6a0b86bc8
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95557680"
---
1. Installez le kit SDK Azure Machine Learning pour Python en suivant les instructions fournies dans l’article sur le [kit SDK Azure Machine Learning](/python/api/overview/azure/ml/install?view=azure-ml-py).

1. Créez un [espace de travail Azure Machine Learning](../articles/machine-learning/how-to-manage-workspace.md).

1. Écrire un [ fichier de configuration](../articles/machine-learning/how-to-configure-environment.md#workspace) (**aml_config/config.json**).

1. Clonez le [référentiel GitHub](https://aka.ms/aml-notebooks).

    ```bash
    git clone https://github.com/Azure/MachineLearningNotebooks.git
    ```

1. Démarrez le serveur de blocs-notes de votre répertoire cloné.

    ```bash
    jupyter notebook
    ```