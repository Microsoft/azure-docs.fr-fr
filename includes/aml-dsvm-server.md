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
ms.openlocfilehash: 2504245a14eee5308e046148f8073cf34ffa8143
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "79485942"
---
1. [Créez un espace de travail Azure Machine Learning](../articles/machine-learning/how-to-manage-workspace.md).

1. Clonez le [référentiel GitHub](https://aka.ms/aml-notebooks).

    ```bash
    git clone https://github.com/Azure/MachineLearningNotebooks.git
    ```

1. Ajoutez au répertoire cloné un fichier de configuration d’espace de travail à l’aide de l’une des méthodes suivantes :

    * Dans le [portail Azure](https://ms.portal.azure.com), sélectionnez **Télécharger config.json** à partir de la section **Vue d’ensemble** de votre espace de travail. 

    ![Télécharger config.json](./media/aml-dsvm-server/download-config.png)

    * Créez un espace de travail à l’aide du code du bloc-notes [configuration.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/configuration.ipynb) dans votre répertoire cloné.

1. Démarrez le serveur de blocs-notes de votre répertoire cloné.

    ```bash
    jupyter notebook
    ```