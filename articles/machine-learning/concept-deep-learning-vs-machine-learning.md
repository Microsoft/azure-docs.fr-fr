---
title: Apprentissage profond et apprentissage automatique
titleSuffix: Azure Machine Learning
description: Découvrez la relation de l’apprentissage profond (Deep Learning) avec l’apprentissage automatique (Machine Learning) et l’intelligence artificielle. Dans Azure Machine Learning, utilisez des modèles d’apprentissage profond pour la détection des fraudes, la détection d’objets, et bien plus.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: lazzeri
author: FrancescaLazzeri
ms.date: 12/15/2020
ms.custom: contperf-fy21q1,contperfq1
ms.openlocfilehash: f7c4529c68f79c6ad19f22054fd0b7d2ba5116db
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97562835"
---
# <a name="deep-learning-vs-machine-learning-in-azure-machine-learning"></a>Apprentissage profond et apprentissage automatique dans Azure Machine Learning

Cet article explique l’apprentissage profond et l’apprentissage automatique, ainsi que la façon dont ils s’intègrent dans la catégorie plus large de l’intelligence artificielle. Découvrez les solutions d’apprentissage profond que vous pouvez créer sur Azure Machine Learning, par exemple, pour la détection des fraudes, la reconnaissance vocale et faciale, l’analyse des sentiments et la prévision de séries chronologiques.

Pour obtenir des conseils sur le choix des algorithmes pour vos solutions, consultez l’[Aide-mémoire de l’algorithme Machine Learning](./algorithm-cheat-sheet.md?WT.mc_id=docs-article-lazzeri).

## <a name="deep-learning-machine-learning-and-ai"></a>Deep Learning, Machine Learning et intelligence artificielle

![Diagramme des relations : Intelligence artificielle, Machine Learning et Deep Learning](./media/concept-deep-learning-vs-machine-learning/ai-vs-machine-learning-vs-deep-learning.png)

Consultez les définitions suivantes pour comprendre la différence entre le deep learning, le machine learning et l’IA :

- Le **Deep Learning**, ou apprentissage profond, est un sous-ensemble du Machine Learning, ou apprentissage automatique, basé sur des réseaux neuronaux artificiels. Le _processus d’apprentissage_ est qualifié de _profond_ parce que la structure des réseaux neuronaux artificiels se compose de plusieurs couches d’entrée, de sortie et masquées. Chaque couche contient des unités qui transforment les données d’entrée en informations que la couche suivante peut utiliser une tâche prédictive spécifique. Grâce à cette structure, une machine est capable d’apprendre au travers de son propre traitement de données.

- Le **Machine Learning** est un sous-ensemble de l’intelligence artificielle utilisant des techniques (telles que le Deep Learning), qui permettent aux machines de tirer des enseignements de leur expérience pour améliorer la manière dont elles exécutent leur tâches. Le _processus d’apprentissage_ repose sur les étapes suivantes :

   1. Alimenter un algorithme en données (au cours de cette étape, vous pouvez fournir des informations supplémentaires au modèle, par exemple, en effectuant l’extraction de caractéristiques).
   1. Utilisez ces données pour entraîner un modèle.
   1. Tester et déployer le modèle.
   1. Utiliser le modèle déployé pour effectuer une tâche prédictive automatisée (en d’autres termes, appeler et utiliser le modèle déployé pour recevoir les prédictions retournées par le modèle).

- L’**intelligence artificielle** est une technique qui permet aux ordinateurs d’imiter l’intelligence humaine. Cette technique inclut le Machine Learning. 
 
Les techniques de Machine Learning et de Deep Learning vous permettent de créer des systèmes informatiques et des applications qui effectuent des tâches généralement associées à l’intelligence humaine. Ces tâches incluent la reconnaissance d’images, la reconnaissance vocale et la traduction linguistique.

## <a name="techniques-of-deep-learning-vs-machine-learning"></a>Techniques de Deep Learning et de Machine Learning 

À présent que vous comprenez les concepts de Machine Learning et de Deep Learning, comparons les deux techniques. Dans le Machine Learning, l’algorithme doit être informé de la façon d’effectuer une prédiction précise en utilisant plus d’informations (par exemple, en effectuant une extraction de caractéristiques). Dans le Deep Learning, l’algorithme peut apprendre à effectuer une prédiction précise par le biais de son propre traitement de données, grâce à la structure du réseau neuronal artificiel.

Le tableau suivant compare les deux techniques de manière plus détaillée :

| |Machine Learning dans sa globalité |Uniquement le Deep Learning|
|---|---|---|
|  **Nombre de points de données** | Peut utiliser de petites quantités de données pour faire des prédictions. | A besoin de grandes quantités de données d’entraînement pour effectuer des prédictions. |
|  **Dépendances matérielles** | Peut fonctionner sur des machines bas de gamme. Ne nécessite pas beaucoup de puissance de calcul. | Nécessite des machines haut de gamme. Effectue fondamentalement un grand nombre d’opérations de multiplication de matrices. Un GPU peut optimiser efficacement ces opérations. |
|  **Processus de personnalisation** | Nécessite que les caractéristiques soient identifiées et créées avec précision par les utilisateurs. | Apprend des caractéristiques de haut niveau à partir de données, et crée de nouvelles caractéristiques de façon autonome. |
|  **Approche d’apprentissage** | Scinde le processus d’apprentissage en étapes plus petites. Combine ensuite les résultats de chaque étape dans une seule sortie. | Parcourt le processus d’apprentissage en résolvant le problème de bout en bout. |
|  **Temps d’exécution** | Nécessite relativement peu de temps pour apprendre, de quelques secondes à quelques heures. | Nécessite généralement un temps d’entraînement assez long, car un algorithme de deep learning implique de nombreuses couches. |
|  **Sortie** | La sortie est généralement une valeur numérique, telle qu’une note ou une classification. | La sortie peut avoir plusieurs formats, comme un texte, un score ou un son. |

## <a name="transfer-learning"></a>Apprentissage de transfert

La formation de modèles Deep Learning nécessite souvent de grandes quantités de données de formation, des ressources de calcul haut de gamme (GPU, TPU) et un temps de formation plus long. Dans les cas où vous ne disposez pas de ces ressources, vous pouvez raccourcir le processus de formation à l’aide d’une technique appelée apprentissage de transfert.

L’apprentissage de transfert est une technique qui applique les connaissances acquises lors de la résolution d’un problème à un problème différent, mais connexe.

En raison de la structure des réseaux neuronaux, le premier ensemble de couches contient généralement des caractéristiques de niveau inférieur, tandis que le dernier ensemble de couches contient des caractéristiques de niveau supérieur qui sont plus proches du domaine en question. En réutilisant les couches finales dans un nouveau domaine ou problème, vous pouvez réduire considérablement le temps, les données et les ressources de calcul nécessaires à la formation du nouveau modèle. Par exemple, si vous avez déjà un modèle qui reconnaît les voitures, vous pouvez réutiliser ce modèle à l’aide de l’apprentissage de transfert pour qu’il reconnaisse également les camions, les motos et d’autres types de véhicules.

Découvrez comment appliquer l’apprentissage de transfert pour la classification d’images à l’aide d’une infrastructure open source dans Azure Machine Learning : [Classifier des images à l’aide d’un modèle Pytorch](./how-to-train-pytorch.md?WT.mc_id=docs-article-lazzeri).

## <a name="deep-learning-use-cases"></a>Cas d’utilisation du Deep Learning

En raison de sa structure de réseau neuronal artificiel, le Deep Learning excelle dans l’identification de modèles dans des données non structurées telles que des images, du son, de la vidéo et du texte. Pour cette raison, le Deep Learning transforme rapidement de nombreux secteurs, donc ceux de la santé, de l’énergie, des finances et des transports. Ces secteurs repensent actuellement leurs processus métier traditionnels. 

Certaines applications courantes du Deep Learning sont décrites dans les paragraphes suivants. Dans Azure Machine Learning, vous pouvez utiliser un modèle créé à partir d’une infrastructure open source ou créer le modèle à l’aide des outils fournis.

### <a name="named-entity-recognition"></a>Reconnaissance d’entité nommée

La reconnaissance d’entité nommée est une méthode de deep learning qui prend du texte en entrée et le transforme en une classe prédéfinie. Les informations obtenues peuvent être un code postal, une date, un ID de produit, par exemple. Elles peuvent ensuite être stockées dans un schéma structuré afin d’établir une liste d’adresses ou de servir de référence pour un moteur de validation d’identité.

### <a name="object-detection"></a>Détection d’objets

le Deep Learning est souvent utilisé pour la détection d’objets. La détection d’objet s’opère en deux temps : classification d’image, puis localisation d’image. La _classification_ d’image identifie les objets figurant sur l’image, tels que des voitures ou des personnes. La _localisation_ d’image fournit l’emplacement spécifique de ces objets. 

La détection d’objets est déjà utilisée dans les secteurs du jeu, de la distribution, du tourisme et des véhicules autonomes.

### <a name="image-caption-generation"></a>Génération de légende d’image

À l’instar de la reconnaissance d’image, la génération de légende d’image consiste à générer une légende décrivant le contenu d’une image. Lorsque vous pouvez détecter et étiqueter des objets dans des photos, l’étape suivante consiste à convertir ces étiquettes en phrases descriptives. 

Les applications de légendage d’images utilisent habituellement des réseaux neuronaux convolutifs pour identifier les objets dans les images, puis un réseau neuronal récurrent pour convertir les étiquettes en phrases cohérentes.

### <a name="machine-translation"></a>Traduction automatique

La traduction automatique consiste à traduire des mots ou des phrases d’une langue dans une autre langue. La traduction automatique existe depuis longtemps, mais le deep learning produit des résultats impressionnants dans deux domaines particuliers : la traduction automatique de texte (et la reconnaissance vocale) et la traduction automatique d’images.

Avec une transformation appropriée des données, un réseau neuronal est capable de comprendre du texte, du son et des images. La traduction automatique permet d’identifier des extraits de son dans des fichiers audio volumineux, ainsi que de transcrire de la parole ou des images en texte.

### <a name="text-analytics"></a>Analytique de texte

L’analyse de texte basée sur des méthodes de deep learning implique l’analyse de grandes quantités de données texte (par exemple, des documents médicaux ou des justificatifs de frais), la reconnaissance de modèles, et la création d’informations structurées et concises à partir des résultats.

Les entreprises utilisent le deep learning pour analyser le texte et détecter le délit d’initié et la conformité aux réglementations gouvernementales. Un autre exemple courant concerne la fraude aux assurances : l’analyse de texte a souvent été utilisée pour analyser de grandes quantités de documents dans le but de reconnaître d’éventuelles fraudes dans les déclarations de sinistres. 

## <a name="artificial-neural-networks"></a>Réseaux neuronaux artificiels

Les réseaux neuronaux artificiels sont formés par couches de nœuds connectés. Les modèles de Deep Learning utilisent des réseaux neuronaux comportant grand nombre de couches. 

Les sections suivantes explorent la plupart des typologies de réseau neuronal artificiel les plus populaires.

### <a name="feedforward-neural-network"></a>Réseau neuronal feedforward

Le réseau neuronal feedforward est le type de réseau neuronal artificiel le plus simple. Dans un réseau feedforward, les informations circulent dans une seule direction, de la couche d’entrée vers la couche de sortie. Les réseaux neuronaux feedforward transforment une entrée en la faisant passer par une série de couches masquées. Chaque couche est constituée d’un ensemble de neurones, et est entièrement connectée à tous les neurones de la couche précédente. La dernière couche entièrement connectée (couche de sortie) représente les prédictions générées.

### <a name="recurrent-neural-network"></a>Réseau de neurones récurrents

Les réseaux neuronaux récurrents sont un type de réseau neuronal artificiel largement utilisé. Ces réseaux enregistrent la sortie d’une couche et la renvoient à la couche d’entrée pour aider à prédire le résultat de la couche. Les réseaux neuronaux récurrents ont des capacités d’apprentissage exceptionnelles. Ils sont largement utilisés pour accomplir des tâches complexes comme la prévision de séries chronologiques, l’apprentissage d’écriture manuscrite et la reconnaissance linguistique.

### <a name="convolutional-neural-network"></a>Réseau neuronal convolutif

Un réseau neuronal convolutif est un réseau neuronal artificiel particulièrement efficace qui présente une architecture unique. Les couches sont organisées en trois dimensions : largeur, hauteur et profondeur. Les neurones d’une couche ne sont pas connectés à tous les neurones de la couche suivante, mais uniquement à une petite région de neurones de celle-ci. Le résultat final est réduit à un seul vecteur de notes de probabilité, organisées dans la dimension de la profondeur. 

Des réseaux neuronaux convolutifs sont utilisés dans des domaines tels que la reconnaissance vidéo, la reconnaissance d’images et les systèmes de recommandation.

## <a name="next-steps"></a>Étapes suivantes

Les articles suivants présentent des options supplémentaires pour l’utilisation de modèles d’apprentissage profond open source dans [Azure Machine Learning](./index.yml?WT.mc_id=docs-article-lazzeri) :


- [Classifier des chiffres manuscrits à l’aide d’un modèle TensorFlow](./how-to-train-tensorflow.md?WT.mc_id=docs-article-lazzeri) 

- [Classifier des chiffres manuscrits à l’aide d’un estimateur TensorFlow et de Keras](./how-to-train-keras.md?WT.mc_id=docs-article-lazzeri)

- [Classifier des chiffres manuscrits à l’aide d’un modèle Chainer](./how-to-set-up-training-targets.md)
