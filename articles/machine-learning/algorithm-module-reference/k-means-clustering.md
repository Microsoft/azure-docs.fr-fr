---
title: 'Clustering k-moyennes : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Clustering k-moyennes dans Azure Machine Learning pour entraîner les modèles de clustering.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 08/04/2020
ms.openlocfilehash: 7e0b61c1ca6ae30044e4c9d4705bdce01eac1942
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93323693"
---
# <a name="module-k-means-clustering"></a>Module : Clustering k-moyennes

Cet article décrit comment utiliser le module *Clustering k-moyennes* dans le concepteur Azure Machine Learning Studio pour créer un modèle de clustering k-moyennes non formé. 
 
K-moyennes est l’un des algorithmes *non supervisé* les plus simples et les plus connus. Vous pouvez utiliser cet algorithme pour diverses tâches d’apprentissage automatique, telles que : 

* [Détection de données anormales](/archive/msdn-magazine/2013/february/data-clustering-detecting-abnormal-data-using-k-means-clustering).
* Clustering de documents de texte.
* Analyse des jeux de données avant d’utiliser d’autres méthodes de classification ou de régression. 

Pour créer un modèle de clustering, vous devez :

* Ajouter ce module à votre pipeline.
* Connecter un jeu de données.
* Définir des paramètres, tels que le nombre de clusters prévus, la mesure de la distance à utiliser pour créer les clusters et ainsi de suite. 
  
Une fois que vous avez configuré les hyperparamètres du module, vous connectez le modèle non formé au [modèle Train Clustering](train-clustering-model.md). Étant donné que l’algorithme k-moyennes est une méthode d’apprentissage non supervisée, une colonne d’étiquette est facultative. 

+ Si vos données incluent une étiquette, vous pouvez utiliser les valeurs d’étiquette pour guider la sélection des clusters et optimiser le modèle. 

+ Si vos données n’ont aucune étiquette, l’algorithme crée des clusters qui représentent les catégories possibles, uniquement en fonction des données.  

##  <a name="understand-k-means-clustering"></a>Comprendre le clustering k-moyennes
 
En règle générale, le clustering utilise des techniques itératives pour grouper les cas d’un jeu de données dans des clusters qui possèdent des caractéristiques similaires. Ces regroupements sont utiles pour explorer des données, identifier les anomalies dans les données et élaborer des prédictions. Les modèles de clustering peuvent aussi vous aider à identifier des relations dans un jeu de données que vous pourriez ne pas obtenir en naviguant ou simplement en observant. Pour ces raisons, le clustering est souvent utilisé dans les premières phases des tâches d’apprentissage automatique, pour explorer les données et découvrir des corrélations inattendues.  
  
 Lorsque vous configurez un modèle de clustering à l’aide de la méthode k-moyennes, vous devez spécifier un nombre cible *k* qui indique le nombre de *centroïdes* souhaités dans le modèle. Le centroïde est un point représentatif de chaque cluster. L’algorithme k-moyennes attribue chaque point de données entrantes à l’un des clusters en minimisant la somme des carrés au sein du cluster. 
 
Lorsqu’il traite les données d’apprentissage, l’algorithme k-moyennes commence par un ensemble initial de centroïdes choisis au hasard. Les centroïdes servent de point de départ pour les clusters, et ils s’appliquent à l’algorithme de Lloyd pour affiner de manière itérative leurs emplacements. L’algorithme k-moyennes cesse de créer et affiner des clusters lorsqu’il répond à une ou plusieurs de ces conditions :  
  
-   Les centroïdes sont stabilisés, ce qui signifie que les attributions de cluster pour différents points cessent de changer et que l’algorithme a convergé vers une solution.  
  
-   L’algorithme a terminé l’exécution du nombre spécifié d’itérations.  
  
 Une fois que vous avez terminé la phase de formation, vous utilisez le module [Attribuer des données à des clusters](assign-data-to-clusters.md) pour attribuer de nouveaux cas à un des clusters que vous avez trouvé à l’aide de l’algorithme k-moyennes. Vous réalisez une attribution de cluster en calculant la distance entre le nouveau cas et le centroïde de chaque cluster. Chaque nouveau cas est attribué au cluster au centroïde le plus proche.  

## <a name="configure-the-k-means-clustering-module"></a>Configurer le module de Clustering k-moyennes
  
1.  Ajoutez le module **Clustering k-moyennes** à votre pipeline.  
  
2.  Pour spécifier le mode d’apprentissage du modèle, sélectionnez l’option **Create trainer mode** (Créer un mode d’apprentissage).  
  
    -   **Single Parameter** (Paramètre unique) : Si vous connaissez les paramètres exacts que vous souhaitez utiliser dans le modèle de clustering, vous pouvez fournir un ensemble spécifique de valeurs comme arguments.  
  
3.  Pour le **nombre de centroïdes** , entrez le nombre de clusters par lesquels doit commencer l’algorithme.  
  
     Il n’est pas garanti que le modèle produise exactement ce nombre de clusters. L’algorithme commence par ce nombre de points de données et effectue une itération pour trouver la configuration optimale. Vous pouvez consulter le [code source de sklearn](https://github.com/scikit-learn/scikit-learn/blob/fd237278e/sklearn/cluster/_kmeans.py#L1069).
  
4.  La propriété **Initialisation** est utilisé pour spécifier l’algorithme utilisé pour définir la configuration initiale du cluster.  
  
    -   **Premier N** : Un certain nombre initial de points de données est choisi dans le jeu de données et utilisé comme moyennes initiales. 
    
         Cette méthode est également appelée *méthode Forgy*.  
  
    -   **Aléatoire** : L’algorithme place au hasard un point de données dans un cluster avant de calculer la moyenne initiale, qui sera le centroïde des points attribués aléatoirement du cluster. 

         Cette méthode est également appelée méthode de *partition aléatoire*.  
  
    -   **K-moyennes ++** : Il s’agit de la méthode par défaut pour l’initialisation de clusters.  
  
         L’algorithme **k-moyennes ++** a été proposé en 2007 par David Arthur et Sergei Vassilvitskii dans le but d’éviter un clustering pauvre par l’algorithme k-moyennes standard. La méthode **k-moyennes ++** améliore la méthode k-moyennes standard via une méthode différente dans le choix des centres de cluster initiaux.  
  
    
5.  Pour **Valeur de départ numérique aléatoire** , vous pouvez éventuellement saisir une valeur à utiliser comme valeur initiale pour l’initialisation de cluster. Cette valeur peut avoir un effet significatif sur la sélection du cluster.  
  
6.  Pour **Métrique** , choisissez la fonction à utiliser pour mesurer la distance entre des vecteurs de cluster, ou entre les nouveaux points de données et le centroïde choisi au hasard. Azure Machine Learning prend en charge les métriques de distance de cluster suivantes :  
  
    -   **Euclidienne** : La distance Euclidienne est couramment utilisée pour mesure l’écart de cluster pour le clustering k-moyennes. Cette métrique est recommandée car elle minimise la distance moyenne entre les points et les centroïdes.
  
7.  Pour **Itérations** , saisissez le nombre d’itération que l’algorithme doit effectuer sur les données formées avant de finaliser la sélection des centroïdes.  
  
     Vous pouvez ajuster ce paramètre pour équilibrer la précision en fonction du temps de formation.  
  
8.  Pour **Attribuer un mode d’étiquette** , choisissez une option qui spécifie comment doit être gérée une colonne d’étiquette, si présente dans le jeu de données.  
  
     Étant donné que le clustering k-moyennes est une méthode d’apprentissage automatique non supervisé, les étiquettes sont facultatives. Toutefois, si votre jeu de données dispose déjà d’une colonne d’étiquette, vous pouvez utiliser ces valeurs pour guider la sélection des clusters, ou vous pouvez spécifier que les valeurs doivent être ignorées.  
  
    -   **Ignorer la colonne d’étiquette** : Les valeurs dans la colonne d’étiquette sont ignorées et ne sont pas utilisées pour générer le modèle.
  
    -   **Indiquer les valeurs manquantes** : Les valeurs de la colonne d’étiquette sont utilisées en tant que fonctionnalités pour aider à créer les clusters. S’il manque une étiquette à une valeur, cette dernière est attribuée à l’aide d’autres fonctionnalités.  
  
    -   **Remplacer du plus proche au centre** : Les valeurs de la colonne d’étiquette sont remplacées par les valeurs d’étiquette prédites, à l’aide de l’étiquette du point le plus proche du centroïde actuel.  

8.  Sélectionnez l’option **Normaliser les fonctionnalités** si vous souhaitez normaliser les fonctionnalités avant la formation.
  
     Si vous appliquez la normalisation avant la formation, les points de données sont normalisés à `[0,1]` par MinMaxNormalizer.

10. Effectuez l’apprentissage du modèle.  
  
    -   Si vous définissez le **mode Create trainer** (Créer formateur) sur **Paramètre unique** , ajoutez un jeu de données balisé et formez le modèle à l’aide du module [Train Clustering Model](train-clustering-model.md).  
  
## <a name="results"></a>Résultats

Une fois la configuration et la formation du modèle terminées, vous disposez d’un modèle que vous pouvez utiliser pour générer des scores. Toutefois, il existe plusieurs façons de former le modèle, ainsi que plusieurs méthodes pour afficher et utiliser les résultats : 

### <a name="capture-a-snapshot-of-the-model-in-your-workspace"></a>Capturer un instantané du modèle dans votre espace de travail

Si vous avez utilisé le module [Train Clustering Model](train-clustering-model.md) :

1. Sélectionnez le module **Entraîner un modèle de clustering** et ouvrez le volet droit.

2. Sélectionnez l’onglet **Sorties**. Sélectionnez l’icône **Enregistrer le jeu de données** pour enregistrer une copie du modèle formé.

Le modèle enregistré représente les données d’apprentissage au moment où vous avez enregistré le modèle. Si vous mettez à jour ultérieurement les données de formation utilisées dans le pipeline, le modèle enregistré n’est pas mis à jour. 

### <a name="see-the-clustering-result-dataset"></a>Consulter le jeu de données de résultat du clustering 

Si vous avez utilisé le module [Train Clustering Model](train-clustering-model.md) :

1. Cliquez avec le bouton droit sur le module **Train Clustering Model**.

2. Sélectionnez **Visualiser**.

### <a name="tips-for-generating-the-best-clustering-model"></a>Conseils pour générer le meilleur modèle de clustering  

Nous savons que le processus de *seeding* utilisé pendant le clustering peut affecter de manière significative le modèle. Le seeding, ou amorçage, correspond au placement initial de points dans des centroïdes potentiels.
 
Par exemple, si le jeu de données contient de nombreuses valeurs hors norme, et que l’une d’elles est choisie pour amorcer les clusters, aucun autre point de données n’est compatible avec ce cluster, et le cluster peut être un singleton. Autrement dit, il pourrait n’avoir qu’un seul point.  
  
Vous pouvez éviter ce problème de deux façons :  
  
-   Modifiez le nombre de centroïdes et essayez plusieurs valeurs de départ.  
  
-   Créez plusieurs modèles, en variant la métrique ou en effectuant davantage d’itérations.  
  
En général, avec les modèles de clustering, il est possible qu’une configuration donnée crée un ensemble localement optimisé de clusters. En d’autres termes, l’ensemble des clusters retourné par le modèle n’est adapté qu’aux points de données actuels et ne peut pas s’étendre à d’autres données. Si vous utilisez une autre configuration initiale, la méthode k-moyennes peut trouver une configuration différente, supérieure. 

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning.