---
title: 'Réseau neuronal à deux classes : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Réseau neuronal à deux classes dans Azure Machine Learning pour créer un classifieur binaire.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 04/22/2020
ms.openlocfilehash: 9131a2439facef00cae818bffef38e536a40a2fd
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421156"
---
# <a name="two-class-neural-network-module"></a>Module Two-Class Neural Network (Réseau neuronal à deux classes)

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez ce module pour créer un modèle de réseau neuronal qui peut être utilisé pour prédire une cible ayant uniquement deux valeurs.

La classification à l’aide de réseaux neuronaux est une méthode d’apprentissage supervisée. Ainsi, elle nécessite un *jeu de données avec balises*, qui inclut une colonne d’étiquette. Par exemple, vous pouvez utiliser ce modèle de réseau neuronal pour prédire des résultats binaires tels que l’affection ou non d’un patient par une maladie ou la possibilité ou non qu’un ordinateur fasse l’objet d’une défaillance dans une fenêtre de temps donnée.  

Après avoir défini le modèle, effectuez son apprentissage en fournissant un jeu de données avec balises et le modèle en tant qu’entrée pour [Train Model](./train-model.md) (Entraîner le modèle). Le modèle formé peut ensuite être utilisé pour prédire des valeurs pour les nouvelles entrées.

### <a name="more-about-neural-networks"></a>Plus d’informations sur les réseaux neuronaux

Un réseau neuronal est un ensemble de couches connectées entre elles. Les entrées représentent la première couche et sont connectées à une couche de sortie par un graphique acyclique constitué de nœuds et de bords pondérés.

Entre les couches d’entrée et de sortie, vous pouvez insérer plusieurs couches masquées. La plupart des tâches prédictives peuvent être accomplies facilement avec une ou quelques couches masquées. Toutefois, de récentes recherches ont montré que les réseaux neuronaux profonds avec nombreuses couches peuvent être efficaces pour les tâches complexes telles que la reconnaissance vocale ou des images. Les couches successives sont utilisées pour modéliser des niveaux de profondeur sémantique toujours plus importants.

La relation entre les entrées et les sorties est tirée de la formation du réseau neuronal sur les données d’entrée. Le graphique commence par les entrées, se poursuit avec la couche masquée et s’achève avec la couche de sortie. Tous les nœuds d’une couche sont connectés par les bords pondérés aux nœuds de la couche suivante.

Pour calculer la sortie du réseau pour une entrée donnée, une valeur est calculée sur chaque nœud dans les couches masquées et dans la couche de sortie. La valeur est définie selon le calcul de la somme pondérée des valeurs des nœuds de la couche précédente. Une fonction d’activation est ensuite appliquée à cette somme pondérée.
  
## <a name="how-to-configure"></a>Comment configurer

1.  Ajoutez le module **Réseau neuronal à deux classes** à votre pipeline. Vous pouvez trouver ce module sous **Machine Learning**, **Initialiser**, dans la catégorie **Classification**.  
  
2.  Spécifiez le mode d’apprentissage du modèle en définissant l’option **Créer un mode d’apprentissage**.  
  
    -   **Single Parameter** (Paramètre unique) : choisissez cette option si vous savez déjà comment vous souhaitez configurer le modèle.

    -   **Plage de paramètres** : en cas de doute sur les paramètres à utiliser, utilisez le module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md). Vous fournissez une plage de valeurs, et le mode d'apprentissage effectue une itération sur plusieurs combinaisons de paramètres pour déterminer la combinaison de valeurs qui produit le meilleur résultat.  

3.  Pour **Hidden layer specification** (Spécification de couche masquée), sélectionnez le type d’architecture réseau à créer.  
  
    -   **Fully connected case** (Cas entièrement connecté) : utilise l’architecture de réseau neuronal par défaut définie pour les réseaux neuronaux à deux classes comme suit :
  
        -   Possède une couche cachée.
  
        -   La couche de sortie est entièrement connectée à la couche masquée, et la couche masquée est entièrement connectée à la couche d’entrée.
  
        -   Le nombre de nœuds dans la couche d’entrée correspond au nombre de caractéristiques dans les données de formation.
  
        -   Le nombre de nœuds dans la couche masquée est défini par l’utilisateur. La valeur par défaut est 100.
  
        -   Le nombre de nœuds correspond au nombre de classes. Pour un réseau neuronal à deux classes, cela signifie que toutes les entrées doivent correspondre à l’un des deux nœuds dans la couche de sortie.

5.  Pour **Learning rate** (Taux d’apprentissage), définissez la taille de l’étape effectuée à chaque itération, avant correction. Avec une valeur de taux d’apprentissage supérieure, le modèle convergera peut-être plus rapidement, mais cette valeur peut dépasser les minima locaux.

6.  Pour **Number of learning iterations** (Nombre d’itérations d’apprentissage), spécifiez le nombre maximal de fois où l’algorithme doit traiter les cas d’apprentissage.

7.  Pour **The initial learning weights diameter** (Le diamètre initial des pondérations d’apprentissage), spécifiez les pondérations de nœud au début du processus d’apprentissage.

8.  Pour **The momentum** (La dynamique), spécifiez une pondération à appliquer pendant l’apprentissage aux nœuds des itérations précédentes.  

10. Sélectionnez l’option **Shuffle examples** (Réorganiser les exemples de façon aléatoire) pour réorganiser les cas entre les itérations de façon aléatoire. Si vous désélectionnez cette option, les cas sont traités exactement dans le même ordre chaque fois que vous exécutez le pipeline.
  
11. Pour **Random number seed** (Valeur initiale aléatoire), tapez une valeur à utiliser comme valeur initiale.
  
     La spécification d’une valeur de départ est utile lorsque vous souhaitez garantir la répétabilité entre les exécutions du même pipeline.  Sinon, une valeur d’horloge système est utilisée comme valeur de départ, ce qui peut entraîner des résultats légèrement différents chaque fois que vous exécutez le pipeline.
  
13. Ajoutez un jeu de données étiqueté au pipeline, puis entraînez le modèle :

    + Si vous définissez **Create trainer mode** (Créer un mode d’apprentissage) sur **Single Parameter** (Paramètre unique), connectez un jeu de données balisé au module [Entraîner le modèle](train-model.md).  
  
    + Si vous définissez **Créer un mode d’entraînement** sur **Plage de paramètres**, connectez un jeu de données avec balises et entraînez le modèle en utilisant [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md).  
  
    > [!NOTE]
    > 
    > Si vous transmettez une plage de paramètres à [Entraîner le modèle](train-model.md), elle utilise uniquement la valeur par défaut dans la liste de paramètres unique.  
    > 
    > Si vous transmettez un ensemble unique de valeurs de paramètre au module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md), quand il attend une plage de paramètres pour chaque paramètre, il ignore les valeurs et utilise les valeurs par défaut pour l’apprenant.  
    > 
    > Si vous sélectionnez l’option **Plage de paramètres** et que vous entrez une valeur unique pour un paramètre, cette valeur unique que vous avez spécifiée est utilisée tout au long du balayage, même si d’autres paramètres changent dans une plage de valeurs.  
  
14. Envoyez le pipeline.

## <a name="results"></a>Résultats

Une fois l’apprentissage terminé :

+ Pour enregistrer un instantané du modèle entraîné, sélectionnez l’onglet **Sorties** dans le panneau droit du module **Entraîner le modèle**. Sélectionnez l’icône **Inscrire le jeu de données** pour enregistrer le modèle en tant que module réutilisable.

+ Pour utiliser le modèle à des fins de scoring, ajoutez le module **Modèle de scoring** à un pipeline.


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 
