---
title: Évaluer les résultats de l’expérience AutoML
titleSuffix: Azure Machine Learning
description: Apprenez à afficher et à évaluer les graphiques et les métriques pour chacune de vos exécutions d’expérience de Machine Learning automatisé.
services: machine-learning
author: aniththa
ms.author: anumamah
ms.reviewer: nibaccam
ms.service: machine-learning
ms.subservice: core
ms.date: 10/09/2020
ms.topic: conceptual
ms.custom: how-to, contperfq2, automl
ms.openlocfilehash: fcbe0fc5049f6e892f80f048a885c75420bc636e
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93359083"
---
# <a name="evaluate-automated-machine-learning-experiment-results"></a>Évaluer les résultats de l’expérience de Machine Learning automatisé

Cet article explique comment afficher et évaluer les résultats de vos expériences de Machine Learning automatisé, ML automatisé. Ces expériences comprennent plusieurs exécutions, chacune d’entre elles créant un modèle. Pour vous aider à évaluer chaque modèle, le ML automatisé génère automatiquement des métriques et graphiques de performances spécifiques pour votre type d’expérience. 

Par exemple, le ML automatisé fournit différents graphiques pour les modèles de classification et de régression. 

|Classification|régression ;
|---|---|
|<li> [Matrice de confusion](#confusion-matrix) <li>[Graphique de rappel de précision](#precision-recall-chart) <li> [ROC (Receiver operating characteristic)](#roc) <li> [Courbe d’élévation](#lift-curve)<li> [Courbe de gains](#gains-curve)<li> [Tracé d’étalonnage](#calibration-plot) | <li> [Prédiction et True](#pvt) <li> [Histogramme des résidus](#histo)|

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree) dès aujourd’hui.

* Créez une expérience pour votre exécution de machine learning automatisé, que ce soit avec le SDK ou dans Azure Machine Learning Studio.

    * Utilisez le Kit de développement logiciel (SDK) pour développer un [modèle de classification](how-to-auto-train-remote.md) ou un [modèle de régression](tutorial-auto-train-models.md)
    * Utilisez [Azure Machine Learning Studio](how-to-use-automated-ml-for-ml-models.md) pour créer un modèle de classification ou de régression en chargeant les données appropriées.

## <a name="view-run-results"></a>Afficher les résultats de l’exécution

Lorsque votre expérience de Machine Learning automatisé est terminée, un historique des exécutions est disponible dans votre espace de travail de Machine Learning via le [Azure Machine Learning studio](overview-what-is-machine-learning-studio.md). 

Pour les expériences de kit de développement logiciel (SDK), vous pouvez également voir ces mêmes résultats lors d’une exécution avec le `RunDetails` [widget Jupyter](/python/api/azureml-widgets/azureml.widgets?preserve-view=true&view=azure-ml-py).

Les étapes et les animations suivantes montrent comment afficher l’historique des exécutions et les métriques et graphiques de performances d’un modèle spécifique dans le studio.

![Procédure d’affichage de l’historique des exécutions et des métriques et graphiques de performances d’un modèle](./media/how-to-understand-automated-ml/view-run-metrics-ui.gif)

Pour afficher l’historique des exécutions et les métriques et graphiques de performances du modèle et les graphiques dans le studio : 

1. [Connectez-vous au studio](https://ml.azure.com/) et accédez à votre espace de travail.
1. Dans le panneau gauche de l’espace de travail, sélectionnez **Exécutions**.
1. Dans la liste des expériences, sélectionnez celle que vous voulez explorer.
1. Dans la table du bas, sélectionnez l’**Exécution**.
1. Dans l’onglet **Modèles**, sélectionnez le **Nom de l’algorithme** pour le modèle que vous voulez explorer.
1. Dans l’onglet **Métriques**, sélectionnez les métriques et graphiques que vous voulez évaluer pour ce modèle. 


<a name="classification"></a> 

## <a name="classification-performance-metrics"></a>Métriques de performances de classification

Le tableau suivant récapitule les métriques de performances de modèle calculées par le ML automatisé pour chaque modèle de classification généré pour votre expérience. 

Métrique|Description|Calcul|Paramètres supplémentaires
--|--|--|--
AUC_macro| « AUC » est « Area under the Receiver Operating Characteristic Curve » (la zone sous la courbe caractéristique de fonctionnement du récepteur). « Macro » est la moyenne arithmétique de l’AUC pour chaque classe.  | [Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html) | average="macro"|
AUC_micro| « AUC » est « Area under the Receiver Operating Characteristic Curve » (la zone sous la courbe caractéristique de fonctionnement du récepteur). « Micro » est calculé globalement en combinant les vrais positifs et les faux positifs de chaque classe.| [Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html) | average="micro"|
AUC_weighted  | « AUC » est « Area under the Receiver Operating Characteristic Curve » (la zone sous la courbe caractéristique de fonctionnement du récepteur). « Weighted » est la moyenne arithmétique du score pour chaque classe, pondérée par le nombre d’instances « true » dans chaque classe.| [Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_auc_score.html)|average="weighted"
accuracy|La précision est le pourcentage d’étiquettes prédites qui correspondent exactement aux étiquettes réelles. |[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.accuracy_score.html) |None|
average_precision_score_macro|La précision moyenne résume la courbe précision-rappel comme moyenne pondérée des précisions atteintes à chaque seuil, avec l’augmentation du rappel du seuil précédent utilisé comme pondération. Macro est la moyenne arithmétique du score de précision moyen de chaque classe.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html)|average="macro"|
average_precision_score_micro|La précision moyenne résume la courbe précision-rappel comme moyenne pondérée des précisions atteintes à chaque seuil, avec l’augmentation du rappel du seuil précédent utilisé comme pondération. « Micro » est calculé globalement en combinant les vrais positifs et les faux positifs de chaque limite.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html)|average="micro"|
average_precision_score_weighted|La précision moyenne résume la courbe précision-rappel comme moyenne pondérée des précisions atteintes à chaque seuil, avec l’augmentation du rappel du seuil précédent utilisé comme pondération. « Weighted » est la moyenne arithmétique du score de précision moyen pour chaque classe, pondérée par le nombre d’instances « true » dans chaque classe.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html)|average="weighted"|
balanced_accuracy|La précision équilibrée est la moyenne arithmétique du rappel pour chaque classe.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="macro"|
f1_score_macro|Le score F1 est la moyenne harmonique de la précision et du rappel. « Macro » est la moyenne arithmétique du score F1 pour chaque classe.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)|average="macro"|
f1_score_micro|Le score F1 est la moyenne harmonique de la précision et du rappel. « Micro » est calculé globalement en comptant le total des vrais positifs, des faux négatifs et des faux positifs.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)|average="micro"|
f1_score_weighted|Le score F1 est la moyenne harmonique de la précision et du rappel. Moyenne pondérée par fréquence de classe du score F1 pour chaque classe|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.f1_score.html)|average="weighted"|
log_loss|Il s’agit de la fonction de perte utilisée dans la régression logistique (multinomiale) et les extensions de celle-ci, comme les réseaux neuronaux, définie comme la probabilité logarithmique négative des étiquettes réelles, étant données les prédictions d’un classifieur probabiliste. Pour un échantillon unique avec l’étiquette vraie yt dans {0,1} et la probabilité estimée yp que yt = 1, la perte logarithmique est - -log P(yt&#124;yp) = -(yt log(yp) + (1 - yt) log(1 - yp)).|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.log_loss.html)|None|
norm_macro_recall|Le rappel macro normalisé est tel que la performance aléatoire ait un score de 0 et la performance parfaite ait un score de 1. Ceci est réalisé par norm_macro_recall := (recall_score_macro - R)/(1 - R), où R est la valeur attendue de recall_score_macro pour des prédictions aléatoires (c’est-à-dire R=0,5 pour la classification binaire et R=(1/C) pour les problèmes de classification de classe C).|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average = "macro" |
precision_score_macro|La précision est le pourcentage d’éléments prédits positivement qui sont correctement étiquetés. « Macro » est la moyenne arithmétique de la précision pour chaque classe.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html)|average="macro"|
precision_score_micro|La précision est le pourcentage d’éléments prédits positivement qui sont correctement étiquetés. « Micro » est calculé globalement en comptant le total des vrais positifs et des faux positifs.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html)|average="micro"|
precision_score_weighted|La précision est le pourcentage d’éléments prédits positivement qui sont correctement étiquetés. « Weighted » est la moyenne arithmétique de la précision pour chaque classe, pondérée par le nombre d’instances « true » dans chaque classe.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_score.html)|average="weighted"|
recall_score_macro|Le rappel est le pourcentage d’éléments d’une certaine classe correctement étiquetés. « Macro » est la moyenne arithmétique du rappel pour chaque classe.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="macro"|
recall_score_micro|Le rappel est le pourcentage d’éléments d’une certaine classe correctement étiquetés. « Micro » est calculé globalement en comptant le total des vrais positifs, des faux négatifs et des faux positifs.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="micro"|
recall_score_weighted|Le rappel est le pourcentage d’éléments d’une certaine classe correctement étiquetés. « Weighted » est la moyenne arithmétique du rappel pour chaque classe, pondérée par le nombre d’instances « true » dans chaque classe.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.recall_score.html)|average="weighted"|
weighted_accuracy|La précision pondérée est la précision où le poids donné à chaque exemple est égal à la proportion d’instances « true » dans la classe « true » de cet exemple.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.accuracy_score.html)|sample_weight est un vecteur égal à la proportion de cette classe pour chaque élément dans la cible|

### <a name="binary-vs-multiclass-metrics"></a>Métriques binaires et multiclasses

Le ML automatisé ne fait pas la différence entre les métriques binaires et multiclasses. Les mêmes métriques de validation sont signalées si un jeu de données a deux classes ou plus de deux classes. Toutefois, certaines métriques sont destinées à la classification multiclasse. Lorsqu’elles sont appliquées à un jeu de données binaire, ces métriques ne traitent pas les classes comme la classe `true`, comme vous pourriez vous y attendre. Les métriques qui sont clairement destinées à la multiclasse sont suivies d’un suffixe `micro`, `macro` ou `weighted`. `average_precision_score`, `f1_score`, `precision_score`, `recall_score` et `AUC` en sont des exemples.

Par exemple, au lieu de calculer le rappel comme `tp / (tp + fn)`, le rappel multiclasse pondéré (`micro`, `macro` ou `weighted`) fait la moyenne des deux classes d’un jeu de données de classification binaire. Cela revient à calculer le rappel pour la classe `true` et la classe `false` séparément, puis à obtenir la moyenne des deux.

## <a name="confusion-matrix"></a>Matrice de confusion

Une matrice de confusion décrit les performances d’un modèle de classification. Chaque ligne affiche les instances de la classe true, ou de la classe actuelle dans votre jeu de données, et chaque colonne représente les instances de la classe qui a été prédite par le modèle. 

Pour chaque matrice de confusion, le ML automatisé affiche la fréquence de chaque étiquette prédite (colonne) par rapport à celle de chaque étiquette True (ligne). Plus la couleur est sombre, plus le nombre dans cette partie de la matrice est élevé. 

### <a name="what-does-a-good-model-look-like"></a>À quoi ressemble un bon modèle ?

Une matrice de confusion compare la valeur réelle du jeu de données aux valeurs prédites par le modèle. Pour cette raison, les modèles Machine Learning seront d’une plus grande justesse si la plupart de leurs valeurs se trouvent le long de la diagonale (ce qui signifie que le modèle a prédit la bonne valeur). Si un modèle a un déséquilibre de classe, la matrice de confusion permet de détecter un modèle biaisé.

#### <a name="example-1-a-classification-model-with-poor-accuracy"></a>Exemple 1 : Modèle de classification avec un niveau de justesse bas
![Modèle de classification avec un niveau de justesse bas](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-confusion-matrix1.png)

#### <a name="example-2-a-classification-model-with-high-accuracy"></a>Exemple 2 : Modèle de classification avec un niveau de justesse élevé 
![Modèle de classification avec un niveau de justesse élevé](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-confusion-matrix2.png)

##### <a name="example-3-a-classification-model-with-high-accuracy-and-high-bias-in-model-predictions"></a>Exemple 3 : Modèle de classification avec un niveau de justesse élevé et un biais élevé dans les prédictions de modèle
![Modèle de classification avec un niveau de justesse élevé et un biais élevé dans les prédictions de modèle](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-biased-model.png)

<a name="precision-recall-chart"></a>

## <a name="precision-recall-chart"></a>Graphique de rappel de précision

La courbe Précision et rappel montre la relation qui existe entre la précision et le rappel dans un modèle. Le terme « précision » désigne la capacité d’un modèle à étiqueter correctement toutes les instances. Le rappel représente la capacité d’un classifieur à rechercher toutes les instances d’une étiquette spécifique.

Avec ce graphique, vous pouvez comparer les courbes de rappel de précision pour chaque modèle afin de déterminer quel modèle présente une relation acceptable entre précision et rappel pour votre problème d’entreprise spécifique. Ce graphique montre le rappel de précision de macro-moyenne, le rappel de précision de micro-moyenne et le rappel de précision associé à toutes les classes d’un modèle. 

La **macro-moyenne** calcule la métrique indépendamment de chaque classe, puis elle prend la moyenne en traitant toutes les classes de manière égale. La **micro-moyenne**, quant à elle, agrège les contributions de toutes les classes pour calculer la moyenne. La micro-moyenne est préférable si le jeu de données contient un déséquilibre de classe.

### <a name="what-does-a-good-model-look-like"></a>À quoi ressemble un bon modèle ?
La courbe Précision et rappel idéale dépend de l’objectif du problème métier. 

##### <a name="example-1-a-classification-model-with-low-precision-and-low-recall"></a>Exemple 1 : Modèle de classification avec un niveau faible de précision et de rappel
![Modèle de classification avec un niveau faible de précision et de rappel](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-precision-recall1.png)

##### <a name="example-2-a-classification-model-with-100-precision-and-100-recall"></a>Exemple 2 : Modèle de classification avec une précision et un rappel d’environ 100 % 
![Modèle de classification avec une précision et un rappel élevés](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-precision-recall2.png)

<a name="roc"></a>

## <a name="roc-chart"></a>Graphique ROC

Le ROC (Receiver Operating Characteristic) est un tracé d’étiquettes bien classées et mal classées pour un modèle spécifique. La courbe ROC peut être moins intéressante lors de l’apprentissage de modèles sur des jeux de données avec un déséquilibre de classe élevé, car la classe majoritaire peut être submergée par les classes minoritaires.

Vous pouvez visualiser la zone sous le graphique ROC comme la proportion d’échantillons correctement classés. Un utilisateur avancé du graphique ROC peut observer au-delà de la zone sous la courbe et obtenir une intuition pour les taux de vrais et faux positifs en tant que fonction du seuil de classification ou de la limite de décision.

### <a name="what-does-a-good-model-look-like"></a>À quoi ressemble un bon modèle ?
Une courbe ROC qui approche le coin supérieur gauche avec un taux de vrais positifs de 100 % et un taux de faux positifs de 0 % est le meilleur modèle. Un modèle aléatoire s’affiche sous la forme d’une ligne plate allant du coin inférieur gauche au coin supérieur droit. Un modèle pire qu’aléatoire serait en dessous de la ligne y = x.

#### <a name="example-1-a-classification-model-with-low-true-labels-and-high-false-labels"></a>Exemple 1 : Modèle de classification avec un nombre faible d’étiquettes Vrais et un nombre élevé d’étiquettes Faux
![Modèle de classification avec un nombre faible d’étiquettes Vrais et un nombre élevé d’étiquettes Faux](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-roc-1.png)

#### <a name="example-2-a-classification-model-with-high-true-labels-and-low-false-labels"></a>Exemple 2 : Modèle de classification avec un nombre élevé d’étiquettes Vrais et un nombre faible d’étiquettes Faux

![Modèle de classification avec un nombre élevé d’étiquettes Vrais et un nombre faible d’étiquettes Faux](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-roc-2.png)


<a name="lift-curve"></a>

## <a name="lift-chart"></a>Graphique de courbes d’élévation

Les graphiques de courbes d’élévation évaluent les performances de modèles de classification. Un graphique de courbes d’élévation dans quelle mesure un modèle est plus performant qu’un modèle aléatoire. Cela vous donne une performance relative qui prend en compte le fait que la classification est plus difficile lorsque vous augmentez le nombre de classes. Un modèle aléatoire prédit de manière incorrecte une fraction plus élevée d’échantillons d’un jeu de données avec dix classes par rapport à un jeu de données avec deux classes.

Vous pouvez comparer l’élévation du modèle généré automatiquement avec Azure Machine Learning par rapport à la ligne de base (modèle aléatoire) afin d’afficher le gain de valeur de ce modèle spécifique.

### <a name="what-does-a-good-model-look-like"></a>À quoi ressemble un bon modèle ?

Un modèle plus performant aura une courbe d’élévation plus élevée sur le graphique et plus éloignée de la ligne de base. 

#### <a name="example-1-a-classification-model-that-performs-poorly-compared-to-a-random-selection-model"></a>Exemple 1 : Modèle de classification avec des performances inférieures à celles d’un modèle de sélection aléatoire
![Modèle de classification avec des performances moins élevées que celles d’un modèle de sélection aléatoire](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-lift-curve1.png)

#### <a name="example-2-a-classification-model-that-performs-better-than-a-random-selection-model"></a>Exemple 2 : Modèle de classification avec des performances plus élevées que celles d’un modèle de sélection aléatoire
![Modèle de classification plus efficace](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-lift-curve2.png)

<a name="gains-curve"></a>

## <a name="cumulative-gains-chart"></a>Graphique de gains cumulés

Un graphique de gains cumulés évalue les performances d’un modèle de classification pour chaque partie des données. Pour chaque centile du jeu de données, le graphique indique le nombre d’échantillons qui ont été correctement classés par rapport à un modèle qui est toujours incorrect. Ces informations offrent une autre façon d’observer les résultats dans le graphique de courbes d’élévation associé.

Le graphique de gains cumulés vous aide à choisir la limite de classification au moyen d’un pourcentage qui correspond à un gain souhaité à partir du modèle. Vous pouvez comparer le graphique de gains cumulés à la ligne de base (modèle incorrect) pour voir le pourcentage d’échantillons qui ont été correctement classés à chaque centile de confiance.

#### <a name="what-does-a-good-model-look-like"></a>À quoi ressemble un bon modèle ?

À l’instar d’un graphique de courbes d’élévation, plus votre courbe de gains cumulés est au-dessus de la ligne de base, plus votre modèle est performant. En outre, plus votre courbe de gains cumulés est proche de l’angle supérieur gauche du graphique, plus le gain de votre modèle est proche de la ligne de base. 

##### <a name="example-1-a-classification-model-with-minimal-gain"></a>Exemple 1 : Modèle de classification avec des gains minimaux
![Modèle de classification avec des gains minimaux](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-gains-curve2.png)

##### <a name="example-2-a-classification-model-with-significant-gain"></a>Exemple 2 : Modèle de classification avec des gains importants
![Modèle de classification avec des gains importants](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-gains-curve1.png)

<a name="calibration-plot"></a>

## <a name="calibration-chart"></a>Graphique d’étalonnage

Un tracé d’étalonnage affiche le niveau de confiance d’un modèle prédictif. Pour ce faire, il montre la relation entre la probabilité prévue et la probabilité réelle, où le terme « probabilité » représente la vraisemblance pour une instance spécifique d’appartenir à une étiquette.

Pour tous les problèmes de classification, vous pouvez consulter la ligne d’étalonnage concernant la micro-moyenne, la macro-moyenne et chaque classe dans un modèle prédictif donné.

La **macro-moyenne** calcule la métrique indépendamment de chaque classe, puis elle prend la moyenne en traitant toutes les classes de manière égale. La **micro-moyenne**, quant à elle, agrège les contributions de toutes les classes pour calculer la moyenne. 

### <a name="what-does-a-good-model-look-like"></a>À quoi ressemble un bon modèle ?
Un modèle bien étalonné s’aligne sur la ligne y = x, où il prédit correctement la probabilité que les échantillons appartiennent à chaque classe. Un modèle trop confiant prédira excessivement des probabilités proches de zéro et un, étant rarement incertain quant à la classe de chaque exemple.

#### <a name="example-1-a-well-calibrated-model"></a>Exemple 1 : Modèle bien étalonné
![ Autre modèle bien étalonné](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-calib-curve1.png)

#### <a name="example-2-an-over-confident-model"></a>Exemple 2 : Modèle trop confiant
![Modèle trop confiant](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-calib-curve2.png)


<a name="regression"></a> 

## <a name="regression-performance-metrics"></a>Métriques de performances de régression

Le tableau suivant récapitule les métriques de performances de modèle calculées par la ML automatisé pour chaque modèle de régression ou modèle de prévision généré pour votre expérience. 

|Métrique|Description|Calcul|Paramètres supplémentaires
--|--|--|--|
explained_variance|La variance expliquée est la proportion selon laquelle un modèle mathématique compte pour la variation d’un jeu de données particulier. C’est le pourcentage de diminution de la variance des données d’origine par rapport à la variance des erreurs. Lorsque la moyenne des erreurs est 0, elle est égale au coefficient de détermination (voir r2_score ci-dessous).|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.explained_variance_score.html)|None|
r2_score|R2 est le coefficient de détermination ou le pourcentage de réduction dans les erreurs quadratiques comparé à un modèle de référence qui génère la moyenne. |[Calcul](https://scikit-learn.org/0.16/modules/generated/sklearn.metrics.r2_score.html)|None|
spearman_correlation|La corrélation de Spearman est une mesure non paramétrique de la monotonie de la relation entre deux jeux de données. Contrairement à la corrélation de Pearson, la corrélation de Spearman ne suppose pas que les deux jeux de données sont normalement distribués. Comme d’autres coefficients de corrélation, celle-ci varie entre -1 et +1, 0 impliquant l’absence de corrélation. Les corrélations de -1 ou +1 impliquent une relation monotone exacte. Les corrélations positives impliquent que quand x augmente, y augmente également. Les corrélations négatives impliquent que quand x augmente, y diminue.|[Calcul](https://docs.scipy.org/doc/scipy-0.16.1/reference/generated/scipy.stats.spearmanr.html)|None|
mean_absolute_error|L’erreur d’absolue moyenne est la valeur attendue de la valeur absolue de la différence entre la cible et la prédiction|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_absolute_error.html)|None|
normalized_mean_absolute_error|L’erreur d’absolue moyenne normalisée est l’erreur absolue moyenne divisée par la plage des données|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_absolute_error.html)|Diviser par la plage de données|
median_absolute_error|L’erreur absolue médiane est la médiane de toutes les différences absolues entre la cible et la prédiction. Cette perte est robuste pour les valeurs hors norme.|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.median_absolute_error.html)|None|
normalized_median_absolute_error|L’erreur d’absolue médiane normalisée est l’erreur absolue médiane divisée par la plage des données|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.median_absolute_error.html)|Diviser par la plage de données|
root_mean_squared_error|L’erreur quadratique moyenne racine est la racine carrée de la différence quadratique attendue entre la cible et la prédiction|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_error.html)|None|
normalized_root_mean_squared_error|L’erreur quadratique moyenne racine normalisée est l’erreur quadratique moyenne racine divisée par la plage des données|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_error.html)|Diviser par la plage de données|
root_mean_squared_log_error|L’erreur logarithmique quadratique moyenne racine est la racine carrée de l’erreur logarithmique quadratique attendue|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_log_error.html)|None|
normalized_root_mean_squared_log_error|L’erreur logarithmique quadratique moyenne racine normalisée est l’erreur logarithmique quadratique moyenne racine divisée par la plage des données|[Calcul](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.mean_squared_log_error.html)|Diviser par la plage de données|

<a name="pvt"></a>

## <a name="predicted-vs-true-chart"></a> Prédiction et True

Prédiction et True indique la relation entre une valeur prévue et sa valeur true en corrélation pour un problème de régression. 

Après chaque exécution, vous pouvez afficher un graphique de type Prédiction et True pour chaque modèle de régression. Pour protéger la confidentialité des données, les valeurs sont réunies dans un conteneur et la taille de chaque emplacement est affichée sous la forme d’un graphique à barres au bas de la zone de graphique. Vous pouvez comparer le modèle prédictif, dont la zone la plus claire indique les marges d’erreur, par rapport à la valeur idéale du modèle.

### <a name="what-does-a-good-model-look-like"></a>À quoi ressemble un bon modèle ?
Ce graphique peut servir à mesurer les performances d’un modèle, car plus les valeurs prévues sont proches de la ligne y=x, plus le modèle prédictif est performant.

#### <a name="example-1-a-regression-model-with-low-performance"></a>Exemple 1 : Modèle de régression faiblement performant
![Modèle de régression avec une justesse faible des prédictions](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-regression1.png)

#### <a name="example-2-a-regression-model-with-high-performance"></a>Exemple 2 : Modèle de régression hautement performant
![Modèle de régression avec un niveau de justesse élevé des prédictions](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-regression2.png)

<a name="histo"></a> 

## <a name="histogram-of-residuals-chart"></a> Histogramme des résidus

Le ML automatisé fournit automatiquement un graphique des résidus pour montrer comment les erreurs d’un modèle de régression sont réparties. Un résiduel est la différence entre la valeur prédite et la valeur réelle (`y_pred - y_true`). 

### <a name="what-does-a-good-model-look-like"></a>À quoi ressemble un bon modèle ?
Pour afficher une marge d’erreur avec un biais faible, l’histogramme des résidus doit avoir la forme d’une cloche centrée sur zéro.

#### <a name="example-1-a-regression-model-with-bias-in-its-errors"></a>Exemple 1 : Modèle de régression avec des erreurs comprenant un biais
![Modèle de régression avec des erreurs comprenant un biais](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-regression3.png)

#### <a name="example-2-a-regression-model-with-a-more-even-distribution-of-errors"></a>Exemple 2 : Modèle de régression avec des erreurs mieux réparties
![Modèle de régression avec des erreurs mieux réparties](./media/how-to-understand-automated-ml/azure-machine-learning-auto-ml-regression4.png)

<a name="explain-model"></a>

## <a name="model-interpretability-and-feature-importance"></a> Interprétabilité du modèle et importance des fonctionnalités
Le Machine Learning automatisé fournit un tableau de bord d’interprétation Machine Learning pour vos exécutions.

Pour plus d’informations sur l’activation des fonctionnalités d’interprétabilité, consultez [Interprétabilité : explications des modèles en Machine Learning automatisé](how-to-machine-learning-interpretability-automl.md).

> [!NOTE]
> Le modèle ForecastTCN n’est actuellement pas pris en charge par le client d’explication. Ce modèle ne renvoie pas de tableau de bord s’il est retourné comme meilleur modèle et ne prend pas en charge les exécutions d’explications à la demande.

## <a name="next-steps"></a>Étapes suivantes

+ Apprenez-en davantage sur le [ML automatisé](concept-automated-ml.md) dans Azure Machine Learning.
+ Testez les exemples de notebooks disponibles dans l’[explication du modèle de Machine Learning automatisé](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/explain-model).