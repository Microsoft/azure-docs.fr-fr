---
title: Interprétabilité des modèles dans le service Azure Machine Learning (version préliminaire)
titleSuffix: Azure Machine Learning
description: Apprenez à comprendre et à expliquer la façon dont votre modèle Machine Learning effectue des prédictions lors de la formation et de l’inférence à l’aide du Kit de développement logiciel (SDK) Python Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: mithigpe
author: minthigpen
ms.reviewer: Luis.Quintanilla
ms.date: 11/16/2020
ms.openlocfilehash: dff0aeaf84ce87ed728d333cb68aee3a349bc111
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94699387"
---
# <a name="model-interpretability-in-azure-machine-learning-preview"></a>Interprétabilité des modèles dans le service Azure Machine Learning (version préliminaire)


## <a name="overview-of-model-interpretability"></a>Vue d’ensemble de l’interprétabilité des modèles

L’interprétabilité est essentielle autant pour les scientifiques des données et les auditeurs que pour les décisionnaires en entreprise pour garantir la conformité avec les stratégies d’entreprise, les normes du secteur et les réglementations gouvernementales :

+ Les scientifiques des données doivent être capables d’expliquer leurs modèles aux dirigeants et aux parties prenantes, afin que ces derniers puissent comprendre la valeur et la précision de leurs découvertes. Ils ont également besoin d’interprétabilité pour déboguer leurs modèles et prendre des décisions éclairées sur la manière de les améliorer. 

+ Les auditeurs juridiques ont besoin d’outils pour valider les modèles en ce qui concerne la conformité réglementaire et surveiller l’impact des décisions des modèles sur les êtres humains. 

+ Les décisionnaires en entreprise veulent être sûrs de pouvoir jouer la transparence pour les utilisateurs finaux. Cela leur permet de gagner et de conserver leur confiance.


La capacité à expliquer un modèle Machine Learning est importante lors de deux phases principales du développement de modèle :
+ Pendant la phase de formation, les concepteurs et évaluateurs de modèles peuvent utiliser la sortie d’interprétabilité d’un modèle pour vérifier des hypothèses et gagner la confiance des parties prenantes. Ils utilisent également les insights qui ressortent du modèle pour le débogage, la confirmation que le comportement du modèle correspond à leurs objectifs et la recherche de l’iniquité du modèle ou de ses caractéristiques insignifiantes.

+ Au cours de la phase d’inférence, bénéficier d’une transparence sur les modèles déployés permet aux dirigeants de comprendre comment, « une fois déployé », le modèle fonctionne et comment ses décisions traitent et influencent les gens dans la vie réelle. 

## <a name="interpretability-with-azure-machine-learning"></a>Interprétabilité avec Azure Machine Learning

Les classes d’interprétabilité sont accessibles via le package de Kit de développement logiciel (SDK) suivant : (En savoir plus sur l’[installation de packages de Kit de développement logiciel (SDK) pour Azure Machine Learning](/python/api/overview/azure/ml/install?preserve-view=true&view=azure-ml-py))

* `azureml.interpret`, contient des fonctionnalités prises en charge par Microsoft.

Utilisez `pip install azureml-interpret` pour une utilisation générale.

## <a name="how-to-interpret-your-model"></a>Comment interpréter votre modèle

Grâce aux classes et aux méthodes dans le Kit de développement logiciel (SDK), vous pouvez :
+ Expliquer la prédiction du modèle en générant des valeurs d’importance des caractéristiques pour l’ensemble du modèle ou des points de donnée individuels. 
+ Réaliser l’interprétabilité des modèles sur des jeux de données du monde réel à l’échelle, pendant la formation et l’inférence.
+ Utiliser un tableau de bord de visualisation interactive pour découvrir des modèles de données et des explications au moment de la formation.


Dans Machine Learning, les **fonctionnalités** sont les champs de données utilisés pour prédire un point de données cible. Par exemple, pour prédire le risque de crédit, les champs de données d’âge, de taille du compte et d’âge de compte peuvent servir. Dans ce cas, l’âge, la taille du compte et l’âge du compte sont des **fonctionnalités**. L’importance d’une fonctionnalité vous indique dans quelle mesure chaque champ de données a contribué aux prédictions du modèle. Par exemple, l’âge peut être fortement utilisé dans la prédiction tandis que la taille et l’âge du compte n’affectent pas les valeurs de prédiction de manière significative. Ce processus permet aux scientifiques des données d’expliquer les prédictions obtenues, de sorte que les parties prenantes ont une visibilité sur les caractéristiques les plus importantes dans le modèle.

Découvrez les techniques d’interprétabilité, les modèles Machine Learning et les environnements d’exécution qui sont pris en charge ici.


## <a name="supported-interpretability-techniques"></a>Techniques d’interprétabilité prises en charge

 `azureml-interpret` utilise les techniques d’interprétabilité développées dans la [Interpret-Community](https://github.com/interpretml/interpret-community/), package Python open source permettant l’apprentissage de modèles interprétables et d’expliquer des systèmes IA de boîte noire. [Interpret-Community](https://github.com/interpretml/interpret-community/) fait office d’hôte pour les explicatifs pris en charge de ce SDK et prend actuellement en charge les techniques d’interprétabilité suivantes :

|Technique d’interprétabilité|Description|Type|
|--|--|--------------------|
|Explicatif d’arborescence SHAP| L’explicatif d’arborescence de [SHAP](https://github.com/slundberg/shap) cible l’algorithme d’estimation de valeur SHAP rapide de temps polynomial spécifique des **arborescences et ensembles d’arborescences**.|Spécifique au modèle|
|Explicatif approfondi SHAP| Selon l’explication proposée par SHAP, l’explicatif approfondi « est un algorithme d’approximation à vitesse élevée de valeurs SHAP dans des modèles de deep learning qui s’appuie sur une connexion avec DeepLIFT décrite dans le [document NIPS de SHAP](https://papers.nips.cc/paper/7062-a-unified-approach-to-interpreting-model-predictions). Les modèles **TensorFlow** et **Keras** utilisant le serveur principal TensorFlow sont pris en charge (une prise en charge préliminaire de PyTorch est également possible) ».|Spécifique au modèle|
|Explicatif linéaire SHAP| L’explicatif linéaire de SHAP calcule les valeurs SHAP pour un **modèle linéaire**, en prenant éventuellement en compte les corrélations entre les caractéristiques.|Spécifique au modèle|
|Explicatif de noyau SHAP| L’explicatif de noyau de SHAP utilise une régression linéaire locale spécialement pondérée pour estimer les valeurs SHAP pour **n’importe quel modèle**.|Indépendant du modèle|
|Explicatif d’imitation (substitution globale)| L’explicatif d’imitation repose sur l’idée d’entraînement de [modèles de substitution globaux](https://christophm.github.io/interpretable-ml-book/global.html) sur des modèles de boîte noire d’imitation. Un modèle de substitution global est un modèle intrinsèquement interprétable qui est formé pour estimer les prédictions de **n’importe quel modèle de boîte noire** aussi précisément que possible. Les scientifiques des données peuvent interpréter le modèle de substitution pour déduire des conclusions sur le modèle de boîte noire. Vous pouvez utiliser un des modèles interprétables suivants comme modèle de substitution : LightGBM (LGBMExplainableModel), régression linéaire (LinearExplainableModel), modèle explicable Stochastic Gradient Descent (SGDExplainableModel) et arbre de décision (DecisionTreeExplainableModel).|Indépendant du modèle|
|Explicatif d’importance de fonctionnalité de permutation (PFI)| L’importance de fonctionnalité de permutation est une technique utilisée pour expliquer les modèles de classification et de régression qui s’inspirent du [document relatif aux forêts aléatoires de Breiman](https://www.stat.berkeley.edu/~breiman/randomforest2001.pdf) (consultez la section 10). De façon générale, il fonctionne en permutant aléatoirement les données d’une caractéristique à la fois pour l’ensemble du jeu de données et en calculant dans quelle mesure la métrique de performances d’intérêt change. Plus la modification est importante, et plus la fonctionnalité l’est également. La PFI peut expliquer le comportement global de **n’importe quel modèle sous-jacent**, mais n’explique pas les prédictions individuelles. |Indépendant du modèle|




Outre les techniques d’interprétabilité décrites ci-dessus, nous prenons en charge un autre explicatif SHAP, appelé `TabularExplainer`. En fonction du modèle, `TabularExplainer` utilise l’un des explicatifs SHAP pris en charge :

* TreeExplainer pour tous les modèles basés sur une arborescence
* DeepExplainer pour les modèles DNN
* LinearExplainer pour les modèles linéaires
* KernelExplainer pour tous les autres modèles

`TabularExplainer` a également apporté des améliorations significatives des caractéristiques et des performances par rapport aux explicatifs SHAP directs :

* **Récapitulatif du jeu de données d’initialisation**. Dans les cas où la vitesse d’explication est plus importante, nous résumons le jeu de données d’initialisation et générons un petit ensemble d’exemples représentatifs, ce qui accélère la génération de valeurs globales et individuelles d’importance d’une caractéristique.
* **Échantillonnage du jeu de données d’évaluation**. Si l’utilisateur transmet un grand ensemble d’exemples d’évaluation, mais que tous ne doivent pas nécessairement être évalués, le paramètre d’échantillonnage peut être défini sur true pour accélérer le calcul de l’ensemble des explications du modèle.

Le diagramme suivant illustre la structure actuelle des explicatifs pris en charge.

[![Architecture d’interprétabilité de Machine Learning](./media/how-to-machine-learning-interpretability/interpretability-architecture.png)](./media/how-to-machine-learning-interpretability/interpretability-architecture.png#lightbox)


## <a name="supported-machine-learning-models"></a>Modèles Machine Learning pris en charge

Le package `azureml.interpret` du Kit de développement logiciel (SDK) prend en charge les modèles formés avec les formats de jeu de données suivants :
- `numpy.array`
- `pandas.DataFrame`
- `iml.datatypes.DenseData`
- `scipy.sparse.csr_matrix`

Les fonctions d’explication acceptent les modèles et les pipelines en entrée. Si un modèle est fourni, il doit implémenter la fonction de prédiction `predict` ou `predict_proba` conforme à la convention Scikit. Si votre modèle ne la prend pas en charge, vous pouvez inclure votre modèle dans une fonction wrapper qui génère le même résultat que `predict` ou `predict_proba` dans Scikit et utiliser cette fonction wrapper avec l’explicatif sélectionné. Si un pipeline est fourni, la fonction d’explication suppose que le script de pipeline en cours d’exécution retourne une prédiction. À l’aide de cette technique d’inclusion dans un wrapper, `azureml.interpret` peut prendre en charge des modèles formés via des infrastructures de Deep Learning PyTorch, TensorFlow et Keras, ainsi que des modèles Machine Learning classiques.

## <a name="local-and-remote-compute-target"></a>Cible de calcul locale et distante

Le package `azureml.interpret` est conçu pour fonctionner avec les cibles de calcul locales et distantes. Si vous les exécutez localement, les fonctions du Kit de développement logiciel (SDK) ne contactent aucun service Azure. 

Vous pouvez exécuter l’explication à distance sur Capacité de calcul Azure Machine Learning et consigner les informations d’explication dans le service d’historique des exécutions d’Azure Machine Learning. Lorsque ces informations sont consignées, les rapports et visualisations de l’explication peuvent être analysés par les utilisateurs sur Azure Machine Learning Studio.


## <a name="next-steps"></a>Étapes suivantes

- Consultez le [guide pratique](how-to-machine-learning-interpretability-aml.md) pour bénéficier de l’interprétabilité des modèles entraînés à la fois localement et sur des ressources de calcul distantes Azure Machine Learning. 
- Pour obtenir des scénarios supplémentaires, consultez les [exemples de notebooks](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/explain-model). 
- Si vous êtes intéressé par l’interprétabilité des scénarios de texte, consultez [Interpret-text](https://github.com/interpretml/interpret-text), un référentiel open source associé à [Interpret-Community](https://github.com/interpretml/interpret-community/), pour les techniques d’interprétabilité pour NLP. Le package `azureml.interpret` ne prend pas en charge ces techniques pour l’instant, mais vous pouvez commencer avec un [exemple de bloc-notes sur la classification du texte](https://github.com/interpretml/interpret-text/blob/master/notebooks/text_classification/text_classification_classical_text_explainer.ipynb).