---
title: Utiliser le module Générateur de recommandations Train Wide and Deep
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Générateur de recommandations Train Wide and Deep dans le concepteur Azure Machine Learning afin d’effectuer l’apprentissage d’un modèle de recommandation.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 06/12/2020
ms.openlocfilehash: ce713167272c9e97754fdf6f6d065519aaea3d15
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421173"
---
# <a name="train-wide--deep-recommender"></a>Générateur de recommandations Train Wide and Deep
Cet article explique comment utiliser le module **Générateur de recommandations Train Wide and Deep** dans le concepteur Azure Machine Learning afin d’effectuer la formation d’un modèle de recommandation. Ce module est basé sur l’apprentissage Wide and Deep, proposé par Google.

Le module de **recommandation Train Wide and Deep** lit un jeu de données composé de triplets utilisateur-élément-évaluation et, éventuellement, de certaines caractéristiques d'utilisateur et d'élément. Il retourne un générateur de recommandations Wide and Deep.  Vous pouvez ensuite utiliser le modèle formé pour générer des prédictions de note ou des recommandations à l’aide du module [Générateur de recommandations Score Wide and Deep](score-wide-and-deep-recommender.md).  

## <a name="more-about-recommendation-models-and-the-wide--deep-recommender"></a>En savoir plus sur les modèles de recommandation et le générateur de recommandations Wide and Deep  

L’objectif principal d’un système de recommandation est de recommander un ou plusieurs *éléments* aux *utilisateurs* du système. Les exemples d’un élément peuvent être un film, un restaurant, un livre ou une chanson. Un utilisateur peut être une personne, un groupe de personnes ou une autre entité avec des préférences d’élément.  

Il existe deux approches principales pour les systèmes de recommandation. 

+ La première est l’approche **basée sur le contenu**, qui utilise des fonctionnalités à la fois pour les utilisateurs et les éléments. Les utilisateurs peuvent être décrits par des propriétés telles que l’âge et le sexe, et les éléments peuvent être décrits par des propriétés telles que l’auteur et le fabricant. Vous trouverez des exemples typiques de systèmes de recommandations basés sur le contenu sur les réseaux sociaux. 
+ La deuxième approche est le **filtrage collaboratif**, qui utilise uniquement les identificateurs des utilisateurs et des éléments, et obtient des informations implicites sur ces entités à partir d’une matrice (éparse) d’évaluations fournies par les utilisateurs aux éléments. Nous pouvons en savoir plus sur les éléments qu’ils ont notés et sur les autres utilisateurs ayant évalué les mêmes éléments.  

Le générateur de recommandations Wide and Deep combine ces approches avec un filtrage collaboratif et une approche basée sur le contenu. Il est donc considéré comme un **système de recommandation hybride**. 

Comment cela fonctionne : Quand un utilisateur est relativement nouveau pour le système, les prédictions sont améliorées en exploitant les informations spécifiques relatives à l'utilisateur, et donc en abordant le problème bien connu du « démarrage à froid ». Toutefois, après avoir recueilli un nombre suffisant d'évaluations d'un utilisateur particulier, il est possible d'élaborer des prédictions entièrement personnalisées en fonction de ses propres évaluations plutôt que de ses seules caractéristiques. Il en résulte donc une transition en douceur de recommandations basées sur le contenu vers des recommandations basées sur un filtrage collaboratif. Même si les caractéristiques d'un utilisateur ou d'un élément ne sont pas disponibles, le module de recommandantion Wide and Deep continue de fonctionner en mode filtrage collaboratif.  

Pour plus d'informations sur le générateur de recommandations Wide and Deep et son algorithme probabiliste sous-jacent, consultez l'étude qui lui est consacrée : [Wide & Deep Learning pour les systèmes de recommandation](https://arxiv.org/pdf/1606.07792.pdf).  

## <a name="how-to-configure-train-wide--deep-recommender"></a>Comment configurer le module Entraîner le générateur de recommandations Wide and Deep  

+ [Préparer les données d’apprentissage](#prepare-data)
+ [Formation du modèle](#train-the-model)

### <a name="prepare-data"></a>Préparer les données

Avant d’essayer d’utiliser le module, il est essentiel que vos données soient au format attendu par le modèle de recommandation. Un jeu de données d’apprentissage **triples utilisateur-élément-notation** est requis, mais vous pouvez également inclure des caractéristiques d’utilisateurs et des caractéristiques d’éléments (si disponibles) dans des jeux de données distincts.

#### <a name="required-dataset-of-user-item-ratings"></a>Jeu de données requis de triplets utilisateur-élément-évaluation

Les données d’entrée utilisées pour l’apprentissage doivent contenir le bon type de données au bon format : 

+ La première colonne doit contenir des identificateurs d’utilisateur.
+ La deuxième colonne doit contenir des identificateurs d’élément.
+ La troisième colonne contient l’évaluation de la paire utilisateur-élément. Les valeurs d’évaluation doivent être de type numérique. 

Par exemple, un jeu utilisateur-élément-évaluation typique peut se présenter comme suit :

|UserId|MovieId|Rating|
|------------|-------------|------------|
|1|68646|10|
|223|31381|10|

#### <a name="user-features-dataset-optional"></a>Jeu de données de caractéristiques utilisateur (facultatif)

Le jeu de données des **caractéristiques d’utilisateurs** doit contenir des identificateurs pour les utilisateurs et utiliser les mêmes identificateurs que ceux fournis dans la première colonne du jeu de données utilisateur-élément-évaluation. Les colonnes restantes peuvent contenir n’importe quel nombre de caractéristiques qui décrivent les utilisateurs.  

Par exemple, un ensemble typique de caractéristiques d’utilisateurs peut se présenter comme suit : 

|UserId|Age|Sexe|Intérêt|Emplacement|
|------------|--------------|-----------------------|---------------|------------|
|1|25|male| Tragédie    |Europe|
|223|40|female|Romance|Asia|

#### <a name="item-features-dataset-optional"></a>Jeu de données de caractéristiques d’éléments (facultatif)

La première colonne de ce jeu de données doit contenir des identificateurs d'éléments. Les colonnes restantes peuvent contenir n’importe quel nombre de caractéristiques descriptives pour les éléments.  

Par exemple, un jeu de caractéristiques d’éléments typique peut se présenter comme suit :  

|MovieId|Intitulé|Langue d’origine|Genres|Year|
|-------------|-------------|-------------------|-----------|---------------|
|68646|Le Parrain|Anglais|Tragédie|1972|
|31381|Autant en emporte le vent|Anglais|Historique|1939|

### <a name="train-the-model"></a>Effectuer l’apprentissage du modèle

1.  Ajoutez le module **Générateur de recommandations Train Wide and Deep** à votre expérience dans le concepteur et connectez-le aux données d’entraînement.  
  
2. Si vous disposez d’un jeu de données distinct de caractéristiques d’utilisateurs et/ou de caractéristiques d’éléments, connectez-le au module **Générateur de recommandations Train Wide and Deep**.  
  
    - **Jeu de données de caractéristiques d’utilisateurs** : Connectez le jeu de données qui décrit les utilisateurs à la deuxième entrée.
    - **Jeu de données de caractéristiques d’éléments** : Connectez le jeu de données qui décrit les éléments à la troisième entrée.  
    
3.  **Époques** : indiquez le nombre de fois où l’algorithme doit traiter les données de formation entières. 

    Plus ce nombre est élevé, plus la formation est appropriée ; toutefois, la formation coûtera plus de temps et pourra entraîner un surajustement.

4. **Taille de lot** : indiquez le nombre d’exemples de formation utilisés dans une étape de formation. 

     Cet hyperparamètre peut influencer la vitesse de formation. Une taille de lot plus élevée entraîne une époque de coût plus faible, mais peut augmenter le temps de convergence. Et si le traitement par lots est trop important pour le GPU/processeur, une erreur de mémoire peut se produire.

5.  **Générateur d’optimisations Wide** : sélectionnez un générateur d’optimisations pour appliquer des gradients à la partie Wide du modèle.

6.  **Taux d’apprentissage du générateur d’optimisations Wide** : entrez un nombre compris entre 0,0 et 2,0, qui définit le taux d’apprentissage du générateur d’optimisations Wide.

    Cet hyperparamètre détermine la taille de pas à chaque étape de la formation tout en évoluant vers une perte de fonction minimale. Un taux d’apprentissage trop important peut entraîner un dépassement des minima, tandis qu’un taux d’apprentissage trop faible peut entraîner un problème de convergence.

7.  **Dimension de caractéristique croisée** : saisissez la dimension en entrant les caractéristiques des ID d’utilisateur et des ID d’élément souhaitées. 

    Le générateur de recommandations Wide & Deep effectue une transformation entre produits sur les caractéristiques d’ID d’utilisateur et d’ID d’élément par défaut. Le résultat croisé sera haché en fonction de ce nombre pour garantir la dimension.

8.  **Générateur d’optimisations Deep** : sélectionnez un générateur d’optimisations pour appliquer des gradients à la partie Deep du modèle.

9.  **Taux d’apprentissage du générateur d’optimisations Deep** : entrez un nombre compris entre 0,0 et 2,0, qui définit le taux d’apprentissage du générateur d’optimisations Deep.

10.  **Dimension d’incorporation d’utilisateur** : tapez un entier pour spécifier la dimension de l’incorporation de l’ID d’utilisateur.

     Le générateur de recommandations Wide and Deep crée les incorporations d’ID d’utilisateur et les incorporations d’ID d’élément partagées à la fois pour les parties Wide et Deep.

11.  **Dimension d’incorporation d’élément** : tapez un entier pour spécifier la dimension de l’incorporation de l’ID d’élément.

12.  **Dimension d’incorporation de caractéristiques catégoriques** : entrez un entier pour spécifier les dimensions des incorporations des caractéristiques catégoriques.

     Dans la partie Deep du générateur de recommandations Wide and Deep, un vecteur d’incorporation est appris pour chaque caractéristique catégorique. Et ces vecteurs d’incorporation partagent la même dimension.

13.  **Unités masquées** : tapez le nombre de nœuds masqués du composant Deep. Le nombre de nœuds dans chaque couche est séparé par des virgules. Par exemple, en tapant « 1000,500,100 », vous spécifiez que le composant Deep a trois couches, respectivement de 1000 nœuds, 500 nœuds et 100 nœuds.

14.  **Fonction d’activation** : sélectionnez une fonction d’activation appliquée à chaque couche ; la valeur par défaut est ReLU.

15.  **Abandon** : entrez un nombre compris entre 0,0 et 1,0 pour déterminer la probabilité que les sorties soient abandonnées dans chaque couche au cours de la formation.

     L’abandon est une méthode de régularisation pour empêcher le surajustement des réseaux neuronaux. Une décision courante pour cette valeur est de commencer par 0,5, qui semble être proche de la valeur optimale pour un large éventail de réseaux et de tâches.

16.  **Normalisation par lot** : sélectionnez cette option pour utiliser la normalisation par lot après chaque couche masquée dans le composant Deep.

     La normalisation par lot est une technique permettant de combattre le problème d’écart de covariable interne pendant la formation des réseaux. En général, elle peut contribuer à améliorer la vitesse, les performances et la stabilité des réseaux. 

17.  Exécuter le pipeline.

##  <a name="technical-notes"></a>Notes techniques

Le module Wide and Deep forme conjointement des modèles linéaires Wide et des réseaux neuronaux Deep pour combiner les atouts de la mémorisation et de la généralisation. Le composant Wide accepte un ensemble de caractéristiques brutes et de transformations de caractéristiques pour mémoriser les interactions entre les caractéristiques. Et avec moins d’ingénierie de caractéristiques, le composant Deep est généralisé aux combinaisons de caractéristiques invisibles par le biais d’incorporations de caractéristiques denses de faible dimension. 

Dans l’implémentation du générateur de recommandations Wide and Deep, le module utilise une structure de modèle par défaut. Le composant Wide prend des incorporations d’utilisateurs, des incorporations d’éléments et la transformation entre produits des ID d’utilisateur et des ID d’élément en entrée. Pour la partie Deep du modèle, un vecteur d’incorporation est appris pour chaque caractéristique catégorique. Avec d’autres vecteurs de caractéristiques numériques, ces vecteurs sont ensuite alimentés dans le réseau neuronal prédictif Deep. La partie Wide et la partie Deep sont combinées en additionnant leurs chances de journalisation finale comme prédiction, ce qui se rejoint finalement dans une seule fonction de perte courante pour la formation conjointe.


## <a name="next-steps"></a>Étapes suivantes

Regardez les [modules disponibles](module-reference.md) pour Azure Machine Learning. 
