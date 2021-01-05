---
title: 'Tutoriel ML Studio (classique) 2 : Entraîner des modèles de risque de crédit - Azure'
description: Tutoriel détaillé indiquant comment créer une solution d’analyse prédictive pour l’évaluation des risques de crédit dans Azure Machine Learning Studio (classique). Ce tutoriel est la deuxième partie d’une série de tutoriels qui en compte trois. Il montre comment entraîner et évaluer des modèles.
keywords: risque de crédit, solution d’analyse prédictive, évaluation des risques
author: sdgilley
ms.author: sgilley
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: tutorial
ms.date: 02/11/2019
ms.openlocfilehash: 557c624b6ed683f701b6a8d38854cd8604e05b31
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96325262"
---
# <a name="tutorial-2-train-credit-risk-models---azure-machine-learning-studio-classic"></a>Tutoriel 2 : Former des modèles de risque de crédit - Azure Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![Cette coche signifie que cet article s’applique à Machine Learning Studio (classique).](../../../includes/media/aml-applies-to-skus/yes.png) Machine Learning Studio (classique)   ![Cette coche signifie que cet article s’applique à Azure Machine Learning.](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)

Dans ce tutoriel, vous étudiez de manière approfondie le processus de développement d’une solution d’analyse prédictive. Vous développez un modèle simple dans Machine Learning Studio (classique).  Vous déployez ensuite le modèle en tant que service web Azure Machine Learning.  Ce modèle déployé peut effectuer des prédictions à l’aide de nouvelles données. Ce tutoriel est la **deuxième partie d’une série de tutoriels qui en compte trois**.

Supposons que vous deviez prédire le risque lié à l'octroi d'un crédit à un individu sur la base des informations fournies lors d'une demande de crédit.  

L’évaluation du risque de crédit est un problème complexe, mais ce tutoriel va le simplifier un peu. Vous allez l’utiliser comme exemple de création d’une solution d’analyse prédictive à l’aide de Microsoft Azure Machine Learning Studio (classique). Vous allez utiliser Machine Learning Studio (classique) et un service web Machine Learning pour cette solution.  

Dans ce tutoriel en trois parties, vous commencez avec des données de risque crédit disponibles publiquement.  Ensuite, vous développez et entraînez un modèle prédictif.  Enfin, vous déployez le modèle en tant que service web.

Dans la [première partie du tutoriel](tutorial-part1-credit-risk.md), vous avez créé un espace de travail Machine Learning Studio (classique), chargé des données et créé une expérience.

Dans cette partie du tutoriel, vous allez effectuer les opérations suivantes :
 
> [!div class="checklist"]
> * Entraîner plusieurs modèles
> * Notation et évaluation des modèles


Dans la [troisième partie du tutoriel](tutorial-part3-credit-risk-deploy.md), vous allez déployer le modèle en tant que service web.

## <a name="prerequisites"></a>Prérequis

Effectuez la [première partie du tutoriel](tutorial-part1-credit-risk.md).

## <a name="train-multiple-models"></a><a name="train"></a>Entraîner plusieurs modèles

Un des avantages de l’utilisation d’Azure Machine Learning Studio (classique) pour créer des modèles Machine Learning est la possibilité d’essayer simultanément plusieurs types de modèle dans une expérience et de comparer les résultats. Ce type d’expérimentation vous aide à trouver la meilleure solution à votre problème.

Dans l’expérience développée au fil de ce tutoriel, vous allez créer deux types de modèles différents, puis comparer les résultats de la notation afin de choisir l’algorithme que vous voulez utiliser dans votre expérience finale.  

Vous avez le choix entre différents modèles. Pour connaître les modèles disponibles, développez le nœud **Machine Learning** dans la palette des modules, puis développez **Initialiser le modèle** et les nœuds inférieurs. Pour cette expérience, vous allez sélectionner les modules [Machines à vecteurs de support à deux classes][two-class-support-vector-machine] et [Arbre de décision optimisé à deux classes][two-class-boosted-decision-tree].

> [!TIP]
> Pour déterminer facilement l’algorithme Machine Learning le mieux adapté au problème à résoudre, consultez [Guide pratique pour choisir des algorithmes pour Microsoft Azure Machine Learning Studio (classique)](../how-to-select-algorithms.md).
> 
> 

Vous allez ajouter le module [Arbre de décision optimisé à deux classes][two-class-boosted-decision-tree] et le module [Machines à vecteurs de support à deux classes][two-class-support-vector-machine] dans cette expérience.

### <a name="two-class-boosted-decision-tree"></a>Arbre de décision optimisé à deux classes

Configurez d’abord le modèle d’arbre de décision optimisé.

1. Recherchez le module [Arbre de décision optimisé à deux classes][two-class-boosted-decision-tree] dans la palette des modules et faites-le glisser sur le canevas.

1. Recherchez le module [Former le modèle][train-model], faites-le glisser sur la zone de dessin et connectez la sortie du module [Arbre de décision optimisé à deux classes][two-class-boosted-decision-tree] au port d’entrée de gauche du module [Former le modèle][train-model].
   
   Le module [Arbre de décision optimisé à deux classes][two-class-boosted-decision-tree] initialise le modèle générique, tandis que le module [Former le modèle][train-model] utilise les données d’apprentissage pour former le modèle. 

1. Connectez la sortie de gauche du module [Exécuter un script R][execute-r-script] de gauche au port d’entrée de droite du module [Entraîner le modèle][train-model] (dans ce tutoriel, vous avez [utilisé les données provenant du côté gauche](#train) du module Fractionner les données pour l’entraînement).
   
   > [!TIP]
   > Pour cette expérience, vous n’avez pas besoin de deux des entrées et de l’une des sorties du module [Exécuter un script R][execute-r-script]. Vous pouvez donc les laisser détachées. 
   > 
   > 

Cette partie de l'expérience ressemble alors à ce qui suit :  

![Training a model](./media/tutorial-part2-credit-risk-train/experiment-with-train-model.png)

Maintenant, vous devez indiquer au module [Former le modèle][train-model] que vous voulez utiliser le modèle pour prédire la valeur Risque de crédit.

1. Sélectionnez le module [Entraîner le modèle][train-model]. Dans le volet **Propriétés**, cliquez sur **Lancer le sélecteur de colonne**.

1. Dans la boîte de dialogue **Sélectionner une seule colonne**, tapez « risque de crédit » dans le champ de recherche sous **Colonnes disponibles**, sélectionnez « Risque de crédit » ci-dessous et cliquez sur la flèche droite ( **>** ) pour déplacer « Risque de crédit » vers **Colonnes sélectionnées**. 

    ![Sélectionnez la colonne Risque de crédit pour le module Former le modèle](./media/tutorial-part2-credit-risk-train/train-model-select-column.png)

1. Cliquez sur la coche **OK**.

### <a name="two-class-support-vector-machine"></a>Machine à vecteurs de support à deux classes

Vous configurez ensuite le modèle SVM.  

Tout d’abord, une petite explication sur SVM. Les arbres de décision optimisés fonctionnent bien avec tout type de caractéristique. Toutefois, le module SVM générant un classifieur linéaire, le modèle qu'il génère obtient la meilleure erreur de test quand toutes les caractéristiques numériques sont à la même échelle. Pour convertir toutes les fonctions numériques à la même échelle, vous utilisez une transformation « Tanh » (avec le module [Normaliser les données][normalize-data]). Cette opération transforme les nombres en plage [0,1]. Le module SVM convertit les fonctionnalités de chaîne en fonctionnalités catégorielles, puis en fonctionnalités 0/1 binaires. Il est donc inutile de les transformer manuellement. De même, vous ne voulez pas transformer la colonne Risque du crédit (colonne 21). Elle est numérique, mais elle contient la valeur de prédiction pour laquelle vous entraînez le modèle ; vous devez donc la laisser seule.  

Pour configurer le modèle SVM, procédez comme suit :

1. Recherchez le module [Machine à vecteurs de support à deux classes][two-class-support-vector-machine] dans la palette des modules et faites-le glisser sur le canevas.

1. Cliquez avec le bouton droit sur le module [Former le modèle][train-model], sélectionnez **Copier**, puis cliquez avec le bouton droit sur le canevas et sélectionnez **Coller**. La copie du module [Former le modèle][train-model] contient les mêmes colonnes que l’original.

1. Connectez la sortie du module [Machines à vecteurs de support à deux classes][two-class-support-vector-machine] au port d’entrée de gauche du deuxième module [Former le modèle][train-model].

1. Trouvez le module [Normaliser les données][normalize-data] et faites-le glisser vers le canevas.

1. Connectez la sortie de gauche du module [Exécuter le script R][execute-r-script] de gauche à l’entrée de ce module (notez que le port de sortie d’un module peut être connecté à plusieurs autres modules).

1. Connectez le port de sortie de gauche du module [Normaliser les données][normalize-data] au port d’entrée de droite du deuxième module [Former le modèle][train-model].

Cette partie de l'expérience ressemble alors à ceci :  

![Training the second model](./media/tutorial-part2-credit-risk-train/svm-model-added.png)

Configurez maintenant le module [Normaliser les données][normalize-data] :

1. Cliquez pour sélectionner le module [Normaliser les données][normalize-data]. Dans le volet **Propriétés**, sélectionnez **Tanh** comme paramètre de la **Méthode de transformation**.

1. Cliquez sur **Lancer le sélecteur de colonne**, sélectionnez « Aucune colonne » pour **Commencer par**, sélectionnez **Inclure** dans la première liste déroulante, **Type de colonne** dans la deuxième, puis **Numérique** dans la troisième. Cette action spécifie que toutes les colonnes numériques (et elles seules) sont transformées.

1. Cliquez sur le signe plus (+) à droite de cette ligne. Cette opération crée une ligne de listes déroulantes. Sélectionnez **Exclure** dans la première liste déroulante, sélectionnez des **noms de colonne** dans la deuxième liste déroulante et entrez « Risque de crédit » dans le champ textuel. Cette opération indique que la colonne Risque de crédit doit être ignorée (en effet, celle-ci étant numérique, elle serait transformée).

1. Cliquez sur la coche **OK**.  

    ![Sélectionner des colonnes pour le module Normaliser les données](./media/tutorial-part2-credit-risk-train/normalize-data-select-column.png)


Le module [Normaliser les données][normalize-data] est maintenant configuré pour effectuer une transformation Tanh sur toutes les colonnes numériques, à l’exception de la colonne Risque de crédit.  

## <a name="score-and-evaluate-the-models"></a>Notation et évaluation des modèles

Vous utilisez les données de test exclues par le module [Fractionner les données][split] pour noter nos modèles formés. Vous pouvez ensuite comparer les résultats des deux modèles pour savoir lequel donne les meilleurs résultats.  

### <a name="add-the-score-model-modules"></a>Ajouter les modules Noter le modèle

1. Recherchez le module [Noter le modèle][score-model] et faites-le glisser sur le canevas.

1. Connectez le module [Former le modèle][train-model] relié au module [Arbre de décision optimisé à deux classes][two-class-boosted-decision-tree], au port d’entrée de gauche du module [Noter le modèle][score-model].

1. Connectez le module [Exécuter le script R][execute-r-script] (nos données de test) au port d’entrée de droite du module [Noter le modèle][score-model].

    ![Module Noter le modèle connecté](./media/tutorial-part2-credit-risk-train/score-model-connected.png)

   
   Le module [Noter le modèle][score-model] peut maintenant récupérer les informations sur le crédit dans les données de test, les traiter à l’aide du modèle et comparer les prévisions générées par le modèle à la colonne Risque de crédit dans les données de test.

1. Copiez et collez le module [Noter le modèle][score-model] pour créer une deuxième copie.

1. Connectez la sortie du modèle SVM (c’est-à-dire le port de sortie du module [Former le modèle][train-model] relié au module [Arbre de décision optimisé à deux classes][two-class-support-vector-machine]) au port d’entrée du deuxième module [Noter le modèle][score-model].

1. Pour le modèle SVM, vous devez effectuer la même transformation pour tester les données comme vous l’avez fait pour les données d’entraînement. Donc, copiez et collez le module [Normaliser les données][normalize-data] pour créer une autre copie et connectez-la au module[Exécuter le script R][execute-r-script] de droite.

1. Connectez la sortie de gauche du deuxième module [Normaliser les données][normalize-data] au port d’entrée de droite du deuxième module [Noter le modèle][score-model].

    ![Deux modules Noter le modèle connectés](./media/tutorial-part2-credit-risk-train/both-score-models-added.png)


### <a name="add-the-evaluate-model-module"></a>Ajouter le module Évaluer le modèle

Pour évaluer les deux résultats de notation et les comparer, vous utilisez un module [Évaluer le modèle][evaluate-model].  

1. Recherchez le module [Évaluer le modèle][evaluate-model] et faites-le glisser vers le canevas.

1. Connectez le port de sortie du module [Noter le modèle][score-model] associé au modèle Arbre de décision optimisé, au port d’entrée gauche du module [Évaluer le modèle][evaluate-model].

1. Connectez l’autre module [Noter le modèle][score-model] au port d’entrée de droite.  

    ![Module Évaluer le modèle connecté](./media/tutorial-part2-credit-risk-train/evaluate-model-added.png)


### <a name="run-the-experiment-and-check-the-results"></a>Exécuter l’expérience et vérifier les résultats

Pour exécuter l’expérience, cliquez sur le bouton **EXÉCUTER** sous le canevas. Cette opération peut prendre quelques minutes. Un indicateur rotatif sur chaque module indique que l’exécution est en cours. Puis une coche verte s’affiche pour signaler que l’exécution du module est terminée. Lorsque tous les modules comportent une coche, l'exécution de l'expérience est terminée.

L'expérience doit ressembler à ceci :  

![Evaluating both models](./media/tutorial-part2-credit-risk-train/final-experiment.png)


Pour vérifier les résultats, cliquez sur le port de sortie du module [Évaluer le modèle][evaluate-model] et sélectionnez **Visualiser**.  

Le module [Évaluer le modèle][evaluate-model] produit une paire de courbes et de mesures qui vous permettent de comparer les résultats des deux modèles notés. Vous pouvez afficher les résultats sous forme de courbes Receiver Operator Characteristic (ROC), Precision/Recall ou Lift. Les données supplémentaires affichées comprennent une matrice de confusion, les valeurs cumulées pour l’aire sous la courbe (ASC) et d’autres mesures. Vous pouvez modifier la valeur du seuil en déplaçant le curseur vers la gauche ou la droite et voir son influence sur l'ensemble des mesures.  

À droite du graphique, cliquez sur **Jeu de données noté** ou **Jeu de données noté à comparer** pour mettre en surbrillance la courbe associée et afficher les mesures associées en dessous. Dans la légende des courbes, « Jeu de données noté » correspond au port d’entrée de gauche du module [Évaluer le modèle][evaluate-model] (en l’occurrence, le modèle Arbre de décision optimisé). « Jeu de données noté à comparer » correspond au port d’entrée de droite, le modèle SVM dans notre cas. Lorsque vous cliquez sur une de ces étiquettes, la courbe de ce modèle apparaît en surbrillance ainsi que les mesures correspondantes, comme dans le graphique suivant.  

![ROC curves for models](./media/tutorial-part2-credit-risk-train/roc-curves.png)

En examinant ces valeurs, vous pouvez déterminer quel modèle est le plus susceptible de fournir les résultats que vous recherchez. Vous pouvez revenir en arrière et relancer votre expérience en modifiant les valeurs des différents modèles. 

Ce tutoriel n’explique pas comment interpréter ces résultats et optimiser les performances du modèle. Pour en savoir plus, consultez les articles suivants :
- [Évaluation des performances d’un modèle dans Azure Machine Learning Studio (classique)](evaluate-model-performance.md)
- [Choisir les paramètres permettant d’optimiser des algorithmes dans Azure Machine Learning Studio (classique)](algorithm-parameters-optimize.md)
- [Interpréter les résultats de modèle dans Azure Machine Learning Studio (classique)](interpret-model-results.md)

> [!TIP]
> À chaque exécution de l’expérience, un enregistrement de cet essai est conservé dans l’historique d’exécution. Vous pouvez afficher tous les essais de votre expérience (et y revenir) en cliquant sur **AFFICHER L'HISTORIQUE D'EXÉCUTION** , sous le canevas. Vous pouvez également cliquer sur **Exécution précédente** dans le volet **Propriétés** pour revenir à l’exécution précédant immédiatement celle que vous avez ouverte.
> 
> Vous pouvez faire une copie d’un essai de votre expérience en cliquant sur **ENREGISTRER SOUS** sous le canevas. 
> Utilisez les propriétés **Résumé** et **Description** de l’expérience pour conserver un enregistrement de vos essais dans les itérations de l’expérience.
> 
> Pour plus d’informations, consultez [Gérer des itérations d’expériences dans Azure Machine Learning Studio (classique)](manage-experiment-iterations.md).  
> 
> 

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [machine-learning-studio-clean-up](../../../includes/machine-learning-studio-clean-up.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez effectué les étapes suivantes : 
 
> [!div class="checklist"]
> * Créer une expérience
> * Entraîner plusieurs modèles
> * Notation et évaluation des modèles

Vous êtes maintenant prêt à déployer des modèles pour ces données.

> [!div class="nextstepaction"]
> [Didacticiel 3 : Classifier Iris : déployer un modèle](tutorial-part3-credit-risk-deploy.md)


<!-- Module References -->
[execute-r-script]: /azure/machine-learning/studio-module-reference/execute-r-script
[edit-metadata]: /azure/machine-learning/studio-module-reference/edit-metadata
[split]: /azure/machine-learning/studio-module-reference/split-data
[evaluate-model]: /azure/machine-learning/studio-module-reference/evaluate-model
[execute-r-script]: /azure/machine-learning/studio-module-reference/execute-r-script
[normalize-data]: /azure/machine-learning/studio-module-reference/normalize-data
[score-model]: /azure/machine-learning/studio-module-reference/score-model
[train-model]: /azure/machine-learning/studio-module-reference/train-model
[two-class-boosted-decision-tree]: /azure/machine-learning/studio-module-reference/two-class-boosted-decision-tree
[two-class-support-vector-machine]: /azure/machine-learning/studio-module-reference/two-class-support-vector-machine
[split]: https://msdn.microsoft.com/library/azure/70530644-c97a-4ab6-85f7-88bf30a8be5f/