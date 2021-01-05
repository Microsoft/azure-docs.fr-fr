---
title: 'Create Python Model : sur le module Modifier les métadonnées'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Create Python Model dans Azure Machine Learning pour créer un module personnalisé de modélisation ou de traitement de données.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 06/18/2020
ms.openlocfilehash: 03f80ebeb7b30d8c0fe14060335541d77b7523b0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90898615"
---
# <a name="create-python-model-module"></a>Module Create Python Model

Cet article décrit un module dans le concepteur Azure Machine Learning.

Découvrez comment utiliser le module Create Python Model pour créer un modèle non formé à partir d’un script Python. Vous pouvez baser le modèle sur tout apprenant inclus dans un package Python dans l’environnement du concepteur Azure Machine Learning. 

Après avoir créé le modèle, vous pouvez utiliser le module [Train Model](train-model.md) pour former le modèle sur un jeu de données, comme tout autre apprenant dans Azure Machine Learning. Le modèle formé peut être transmis au module [Score Model](score-model.md) pour utiliser le modèle afin d’effectuer des prédictions. Vous pouvez ensuite enregistrer le modèle formé et publier le flux de travail de notation en tant que service web.

> [!WARNING]
> Pour le moment, il est impossible de connecter ce module au module **Tune Model Hyperparameters** (Optimiser les hyperparamètres du modèle), ou de passer les résultats scorés d’un modèle Python au module [Evaluate Model](evaluate-model.md) (Évaluer le modèle). Si vous devez optimiser les hyperparamètres ou évaluer un modèle, vous pouvez écrire un script Python personnalisé à l’aide du module [Execute Python Script](execute-python-script.md) (Exécuter un script Python).


## <a name="configure-the-module"></a>Configurer le module

L’utilisation de ce module nécessite une connaissance intermédiaire ou approfondie de Python. Le module prend en charge l’utilisation de tout apprenant qui est inclus dans les packages Python déjà installés dans Azure Machine Learning. Consultez la liste des packages Python préinstallés dans le module [Execute Python Script](execute-python-script.md).

> [!NOTE]
> Soyez très prudent lorsque vous écrivez votre script et assurez-vous qu'il n'existe pas d’erreur de syntaxe, telle que l’utilisation d’un objet non déclaré ou d’un module non importé.

> [!NOTE]
> Portez également une attention particulière à la liste des modules préinstallés dans [Exécuter le script Python](execute-python-script.md). Importez uniquement les modules installés. N’installez pas les packages supplémentaires tels que « pip install xgboost » dans ce script, au risque de générer des erreurs lors de la lecture de modèles dans des modules en aval.
  
Cet article explique comment utiliser le module **Créer un modèle Python** avec un pipeline simple. Voici un diagramme du pipeline :

![Diagramme de Create Python Model](./media/module/create-python-model.png)

1. Sélectionnez **Créer un modèle Python** et modifiez le script pour implémenter votre processus de modélisation ou de gestion des données. Vous pouvez baser le modèle sur tout apprenant inclus dans un package Python dans l’environnement Azure Machine Learning.

> [!NOTE]
> Portez une attention particulière aux commentaires de l’exemple de code du script et vérifiez que votre script respecte strictement la condition, y compris le nom de la classe, les méthodes ainsi que la signature de la méthode. Toute violation entraîne des exceptions. 

   L’exemple suivant de code de classifieur Naive Bayes à deux classes utilise le package populaire *sklearn* :

   ```Python

   # The script MUST define a class named AzureMLModel.
   # This class MUST at least define the following three methods:
       # __init__: in which self.model must be assigned,
       # train: which trains self.model, the two input arguments must be pandas DataFrame,
       # predict: which generates prediction result, the input argument and the prediction result MUST be pandas DataFrame.
   # The signatures (method names and argument names) of all these methods MUST be exactly the same as the following example.

   # Please do not install extra packages such as "pip install xgboost" in this script,
   # otherwise errors will be raised when reading models in down-stream modules.
   
   import pandas as pd
   from sklearn.naive_bayes import GaussianNB


   class AzureMLModel:
       def __init__(self):
           self.model = GaussianNB()
           self.feature_column_names = list()

       def train(self, df_train, df_label):
           # self.feature_column_names records the column names used for training.
           # It is recommended to set this attribute before training so that the
           # feature columns used in predict and train methods have the same names.
           self.feature_column_names = df_train.columns.tolist()
           self.model.fit(df_train, df_label)

       def predict(self, df):
           # The feature columns used for prediction MUST have the same names as the ones for training.
           # The name of score column ("Scored Labels" in this case) MUST be different from any other columns in input data.
           return pd.DataFrame(
               {'Scored Labels': self.model.predict(df[self.feature_column_names]), 
                'probabilities': self.model.predict_proba(df[self.feature_column_names])[:, 1]}
           )


   ```

2. Connecter le module **Create Python Model** que vous venez de créer à **Train Model** et à **Score Model**.

3. Si vous devez évaluer le modèle, ajoutez un module [Execute Python Script](execute-python-script.md) et modifiez le script Python.

   Le script suivant est un exemple de code d’évaluation :

   ```Python


   # The script MUST contain a function named azureml_main
   # which is the entry point for this module.

   # imports up here can be used to 
   import pandas as pd

   # The entry point function MUST have two input arguments:
   #   Param<dataframe1>: a pandas.DataFrame
   #   Param<dataframe2>: a pandas.DataFrame
   def azureml_main(dataframe1 = None, dataframe2 = None):
    
       from sklearn.metrics import accuracy_score, precision_score, recall_score, roc_auc_score, roc_curve
       import pandas as pd
       import numpy as np
    
       scores = dataframe1.ix[:, ("income", "Scored Labels", "probabilities")]
       ytrue = np.array([0 if val == '<=50K' else 1 for val in scores["income"]])
       ypred = np.array([0 if val == '<=50K' else 1 for val in scores["Scored Labels"]])    
       probabilities = scores["probabilities"]
    
       accuracy, precision, recall, auc = \
       accuracy_score(ytrue, ypred),\
       precision_score(ytrue, ypred),\
       recall_score(ytrue, ypred),\
       roc_auc_score(ytrue, probabilities)
    
       metrics = pd.DataFrame();
       metrics["Metric"] = ["Accuracy", "Precision", "Recall", "AUC"];
       metrics["Value"] = [accuracy, precision, recall, auc]
    
       return metrics,

   ```

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 