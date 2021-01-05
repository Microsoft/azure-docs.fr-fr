---
title: Exécuter un script Python dans le concepteur
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le modèle Exécuter un script Python dans le concepteur Azure Machine Learning pour exécuter des opérations personnalisées écrites en Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: likebupt
ms.author: keli19
ms.date: 09/09/2020
ms.topic: conceptual
ms.custom: how-to, designer, devx-track-python
ms.openlocfilehash: dcc28d98efbc82079586de8cfbecd35effc93d6e
ms.sourcegitcommit: dc342bef86e822358efe2d363958f6075bcfc22a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94556231"
---
# <a name="run-python-code-in-azure-machine-learning-designer"></a>Exécuter du code Python dans le concepteur Azure Machine Learning

Dans cet article, vous allez apprendre à utiliser le module [Exécuter le script Python](algorithm-module-reference/execute-python-script.md) pour ajouter une logique personnalisée au concepteur Azure Machine Learning. Dans la procédure suivante, vous utilisez la bibliothèque Pandas pour effectuer une ingénierie de fonctionnalité simple.

Vous pouvez utiliser l’éditeur de code intégré pour ajouter rapidement une logique Python simple. Si vous souhaitez ajouter du code plus complexe ou charger des bibliothèques Python supplémentaires, vous devez utiliser la méthode du fichier zip.

L’environnement d’exécution par défaut utilise la distribution Anacondas de Python. Pour obtenir la liste complète des packages préinstallés, consultez la [page de référence du module Exécuter le script Python](algorithm-module-reference/execute-python-script.md).

![Mappage des entrées de l’exécution d’un script Python](media/how-to-designer-python/execute-python-map.png)

[!INCLUDE [machine-learning-missing-ui](../../includes/machine-learning-missing-ui.md)]

## <a name="execute-python-written-in-the-designer"></a>Exécuter Python écrit dans le concepteur

### <a name="add-the-execute-python-script-module"></a>Ajouter le module Exécuter le script Python

1. Recherchez le module **Exécuter le script Python** dans la palette du concepteur. Il se trouve dans la section **Langage Python**.

1. Glissez-déposez le module sur le canevas du pipeline.

### <a name="connect-input-datasets"></a>Connecter des jeux de données d’entrée

Cet article utilise l’exemple de jeu de données **Automobile price data (Raw)** . 

1. Glissez-déposez votre jeu de données sur le canevas du pipeline.

1. Connectez le port de sortie du jeu de données au port d’entrée situé en haut à gauche du module **Exécuter le script Python**. Le concepteur expose l’entrée en tant que paramètre au script de point d’entrée.
    
    Le port d’entrée de droite est réservé aux bibliothèques Python compressées.

    ![Connecter des jeux de données](media/how-to-designer-python/connect-dataset.png)
        

1. Notez le port d’entrée que vous utilisez. Le concepteur attribue le port d’entrée de gauche à la variable `dataset1` et le port d’entrée intermédiaire à la variable `dataset2`. 

Les modules d’entrée sont facultatifs, car vous pouvez générer ou importer des données directement dans le module **Exécuter le script Python**.

### <a name="write-your-python-code"></a>Écrire votre code Python

Le concepteur fournit un script initial de point d’entrée que vous pouvez modifier pour y entrer votre propre code Python. 

Dans cet exemple, vous utilisez Pandas pour combiner deux colonnes trouvées dans le jeu de données automobile, **Price** et **Horsepower**, pour créer une nouvelle colonne, **Dollars per horsepower**. Cette colonne représente le montant que vous payez pour chaque cheval-vapeur, ce qui peut être une fonctionnalité utile pour décider si une voiture est une bonne affaire d’un point de vue monétaire. 

1. Sélectionnez le module **Exécuter le script Python**.

1. Dans le volet qui s’affiche à droite du canevas, sélectionnez la zone de texte **Script Python**.

1. Copiez et collez le code suivant dans la zone de texte.

    ```python
    import pandas as pd
    
    def azureml_main(dataframe1 = None, dataframe2 = None):
        dataframe1['Dollar/HP'] = dataframe1.price / dataframe1.horsepower
        return dataframe1
    ```
    Votre pipeline doit ressembler à l’image suivante :
    
    ![Pipeline Exécuter Python](media/how-to-designer-python/execute-python-pipeline.png)

    Le script de point d’entrée doit contenir la fonction `azureml_main`. Il existe deux paramètres de fonction qui mappent aux deux ports d’entrée pour le module **Exécuter le script Python**.

    La valeur renvoyée doit être une trame de données Pandas. Vous pouvez retourner jusqu’à deux trames de données comme sorties de module.
    
1. Envoyez le pipeline.

À présent, vous disposez d’un jeu de données doté de la nouvelle fonctionnalité **Dollars/HP**, ce qui peut être utile pour la formation d’un outil de recommandation de voitures. Voici un exemple d’extraction de fonctionnalités et de réduction de la dimensionnalité. 

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [importer vos propres données](how-to-designer-import-data.md) dans le concepteur Azure Machine Learning.
