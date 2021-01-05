---
title: 'Réseau neuronal multiclasse : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Réseau neuronal multiclasse dans le concepteur Azure Machine Learning pour prédire une cible ayant des valeurs multiclasses.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 04/22/2020
ms.openlocfilehash: a4f7f8a7793f31ffbf2099cbfb314fc5097319f5
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421258"
---
# <a name="multiclass-neural-network-module"></a>Module Multiclass Neural Network (Réseau neuronal à multiclasse)

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez ce module pour créer un modèle de réseau neuronal qui peut être utilisé pour prédire une cible ayant plusieurs valeurs. 

Par exemple, les réseaux neuronaux de ce type peuvent être utilisés dans des tâches complexes de vision par ordinateur, telles que la reconnaissance de chiffres ou de lettres, la classification de documents et la reconnaissance de formes.

La classification à l’aide de réseaux neuronaux est une méthode d’apprentissage supervisée. Ainsi, elle nécessite un *jeu de données avec balises*, qui inclut une colonne d’étiquette.

Vous pouvez effectuer l’apprentissage du modèle en fournissant un jeu de données avec balises et le modèle en tant qu’entrée pour [Train Model](./train-model.md) (Entraîner le modèle). Le modèle formé peut ensuite être utilisé pour prédire des valeurs pour les nouveaux exemples d’entrées.  

## <a name="about-neural-networks"></a>À propos des réseaux neuronaux

Un réseau neuronal est un ensemble de couches connectées entre elles. Les entrées représentent la première couche et sont connectées à une couche de sortie par un graphique acyclique constitué de nœuds et de bords pondérés.

Entre les couches d’entrée et de sortie, vous pouvez insérer plusieurs couches masquées. La plupart des tâches prédictives peuvent être accomplies facilement avec une ou quelques couches masquées. Toutefois, de récentes recherches ont montré que les réseaux neuronaux profonds avec nombreuses couches peuvent être efficaces pour les tâches complexes telles que la reconnaissance vocale ou des images. Les couches successives sont utilisées pour modéliser des niveaux de profondeur sémantique toujours plus importants.

La relation entre les entrées et les sorties est tirée de la formation du réseau neuronal sur les données d’entrée. Le graphique commence par les entrées, se poursuit avec la couche masquée et s’achève avec la couche de sortie. Tous les nœuds d’une couche sont connectés par les bords pondérés aux nœuds de la couche suivante.

Pour calculer la sortie du réseau pour une entrée donnée, une valeur est calculée sur chaque nœud dans les couches masquées et dans la couche de sortie. La valeur est définie selon le calcul de la somme pondérée des valeurs des nœuds de la couche précédente. Une fonction d’activation est ensuite appliquée à cette somme pondérée.

## <a name="configure-multiclass-neural-network"></a>Configurer un réseau neuronal multiclasse

1. Ajoutez le module **Réseau neuronal multiclasse** à votre pipeline dans le concepteur. Vous pouvez trouver ce module sous **Machine Learning**, **Initialiser**, dans la catégorie **Classification**.

2. **Create trainer mode** (Créer un mode d’apprentissage) : Utilisez cette option pour spécifier comment vous voulez que le modèle soit entraîné :

    - **Single Parameter** (Paramètre unique) : choisissez cette option si vous savez déjà comment vous souhaitez configurer le modèle.

    - **Plage de paramètres** : Sélectionnez cette option si vous n’êtes pas sûr des paramètres à choisir et que vous souhaitez exécuter un balayage des paramètres. Sélectionnez la plage de valeurs sur laquelle vous souhaitez effectuer l’itération. Le module [Optimiser les hyperparamètres du modèle](tune-model-hyperparameters.md) effectue alors une itération sur toutes les combinaisons possibles des paramètres que vous avez fournis pour déterminer les hyperparamètres qui produisent les résultats optimaux.  

3. **Hidden layer specification** (Spécification de couche masquée) : Sélectionnez le type d’architecture de réseau à créer.

    - **Fully connected case** (Cas entièrement connecté) : Sélectionnez cette option pour créer un modèle à l’aide de l’architecture de réseau neuronal par défaut. Voici les valeurs par défaut pour les modèles de réseau neuronal multiclasse :

        - Une couche cachée
        - La couche de sortie est entièrement connectée à la couche masquée.
        - La couche masquée est entièrement connectée à la couche d’entrée.
        - Le nombre de nœuds dans la couche d’entrée est déterminé par le nombre de caractéristiques dans les données de formation.
        - Le nombre de nœuds dans la couche masquée peut être défini par l’utilisateur. La valeur par défaut est 100.
        - Le nombre de nœuds dans la couche de sortie dépend du nombre de classes.
  
   

5. **Number of hidden nodes** (Nombre de nœuds masqués) : cette option vous permet de personnaliser le nombre de nœuds masqués dans l’architecture par défaut. Entrez le nombre de nœuds masqués. La valeur par défaut est une couche masquée avec 100 nœuds.

6. **The learning rate** (Le taux d’apprentissage) : définissez la taille de l’étape franchie à chaque itération, avant correction. Avec une valeur de taux d’apprentissage supérieure, le modèle convergera peut-être plus rapidement, mais cette valeur peut dépasser les minima locaux.

7. **Number of learning iterations** (Nombre d’itérations d’apprentissage) : spécifiez le nombre maximal de fois où l’algorithme doit traiter les cas d’apprentissage.

8. **The initial learning weights diameter** (Le diamètre initial des pondérations d’apprentissage) : spécifiez les pondérations de nœud au début du processus d’apprentissage.

9. **The momentum** (La dynamique) : spécifiez une pondération à appliquer pendant l’apprentissage aux nœuds des itérations précédentes.
  
11. **Shuffle examples** (Réorganiser les exemples de façon aléatoire) : sélectionnez cette option pour réorganiser les cas entre les itérations de façon aléatoire.

    Si vous désélectionnez cette option, les cas sont traités exactement dans le même ordre chaque fois que vous exécutez le pipeline.

12. **Random number seed** (Valeur de départ aléatoire) : Entrez une valeur à utiliser comme valeur de départ, si vous souhaitez garantir la répétabilité entre les exécutions du même pipeline.

14. Effectuez l’apprentissage du modèle :

    + Si vous définissez **Create trainer mode** (Créer un mode d’apprentissage) sur **Single Parameter** (Paramètre unique), connectez un jeu de données balisé au module [Entraîner le modèle](train-model.md).  
  
    + Si vous définissez le mode **Créer un formateur** sur **Plage de paramètre**, connectez un jeu de données balisé et formez le modèle à l’aide d’[Optimiser des hyperparamètres de modèles](tune-model-hyperparameters.md).  
  
    > [!NOTE]
    > 
    > Si vous transmettez une plage de paramètres à [Effectuer l'apprentissage du modèle](train-model.md), elle utilise uniquement la valeur par défaut dans la liste de paramètres unique.  
    > 
    > Si vous transmettez un ensemble unique de valeurs de paramètre au module [Optimiser des hyperparamètres de modèles](tune-model-hyperparameters.md), quand il attend une plage de paramètres de configuration pour chaque paramètre, il ignore les valeurs et utilise les valeurs par défaut pour l’apprenant.  
    > 
    > Si vous sélectionnez l’option **Plage de paramètres** et que vous entrez une valeur unique pour un paramètre, cette valeur unique que vous avez spécifiée est utilisée tout au long du balayage, même si d’autres paramètres changent sur une plage de valeurs.  
  

## <a name="results"></a>Résultats

Une fois l’apprentissage terminé :

- Pour enregistrer un instantané du modèle entraîné, sélectionnez l’onglet **Sorties** dans le panneau droit du module **Entraîner le modèle**. Sélectionnez l’icône **Inscrire le jeu de données** pour enregistrer le modèle en tant que module réutilisable.

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 