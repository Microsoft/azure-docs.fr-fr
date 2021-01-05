---
title: Préparer les données pour ML Studio (classique) – Team Data Science Process
description: Prétraitez et nettoyez les données afin de les préparer pour les utiliser efficacement dans le cadre d’un apprentissage automatique.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: caedcf313ab809e9607907545f26ca1b62bbeca7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012405"
---
# <a name="tasks-to-prepare-data-for-enhanced-machine-learning"></a>Tâches de préparation des données pour l'apprentissage automatique amélioré
Le prétraitement et le nettoyage des données sont des tâches importantes qui doivent intervenir avant d’utiliser un jeu de données pour la formation de modèles. Les données brutes sont souvent bruyantes, peu fiables et incomplètes. Leur utilisation pour la modélisation peut générer des résultats trompeurs. Ces tâches font partie du processus TDSP (Team Data Science Process) et suivent généralement l'exploration initiale d'un jeu de données utilisé pour découvrir et planifier le traitement préliminaire requis. Pour plus d'instructions sur le processus TDSP, consultez les étapes décrites dans le [processus TDSP (Team Data Science Process)](overview.md).

Les tâches de traitement préalable et de nettoyage, comme la tâche d'exploration de données, peuvent être exécutées dans une grande variété d'environnements, tels que SQL ou Hive ou Azure Machine Learning Studio (classique) et avec différents outils et langages, tels que R ou Python, en fonction de l'emplacement de stockage de vos données et leur mise en forme. Étant donné que le processus TDSP est itératif par nature, ces tâches peuvent avoir lieu à diverses étapes du flux de travail du processus.

Cet article présente différents concepts et tâches de prétraitement des données, à effectuer avant ou après l'intégration de ces données dans Azure Machine Learning (classique).

Pour obtenir un exemple d’exploration de données et de prétraitement dans Azure Machine Learning Studio (classique), consultez la vidéo [Prétraitement des données](https://azure.microsoft.com/documentation/videos/preprocessing-data-in-azure-ml-studio/).

## <a name="why-pre-process-and-clean-data"></a>Pourquoi prétraiter et nettoyer les données ?
Dans le monde réel, les données proviennent de plusieurs sources et processus. Elles peuvent contenir des anomalies ou des valeurs incorrectes qui compromettent la qualité du jeu de données. Les problèmes de qualité les plus fréquents sont les suivants :

* **Caractère incomplet** : des valeurs ou des attributs sont manquants.
* **Bruit** : les données contiennent des enregistrements erronés ou des aberrations.
* **Incohérence** : les données contiennent des enregistrements en conflit ou des contradictions.

La qualité des données est essentielle pour obtenir des modèles prédictifs performants. Pour éviter de traiter des données erronées et améliorer la performance du modèle, il faut impérativement analyser les données, détecter les anomalies le plus tôt possible et déterminer les étapes de prétraitement et de nettoyage appropriées.

## <a name="what-are-some-typical-data-health-screens-that-are-employed"></a>Quels sont les critères d’intégrité des données généralement employés ?
Il est possible d’évaluer la qualité globale des données en vérifiant les éléments suivants :

* Le nombre d' **enregistrements**.
* Le nombre d’**attributs** (ou de **fonctionnalités**).
* Le **type de données** des attributs (nominal, ordinal ou continu).
* Le nombre de **valeurs manquantes**.
* Données **bien formées**.
  * Si les données sont au format TSV ou CSV, vérifiez que les séparateurs de colonne et de ligne séparent correctement et systématiquement les colonnes et les lignes.
  * Si les données sont au format HTML ou XML, vérifiez qu'elles sont conformes à leur standard respectif.
  * Une analyse peut également se révéler nécessaire pour obtenir des informations structurées à partir de données non ou semi-structurées.
* **Enregistrements de données incohérents**. Vérifiez la plage des valeurs autorisées. Par exemple, si les données sont des notes moyennes d’étudiant, vérifiez qu’elles sont comprises dans la plage désignée.

Lorsque vous détectez des problèmes dans les données, des **étapes de traitement** s’imposent : nettoyage des valeurs manquantes, normalisation des données, discrétisation, traitement de texte pour supprimer et/ou remplacer des caractères incorporés susceptibles de perturber l’alignement des données, types de données mixtes dans les champs communs, etc.

**Azure Machine Learning n'exploite que les données tabulaires bien formées**.  Si les données sont déjà sous forme tabulaire, le prétraitement des données peut être effectué directement avec Azure Machine Learning Studio (classique) dans le Machine Learning.  Si elles ne sont pas au format tabulaire, comme le format XML, une analyse peut être nécessaire pour les convertir.  

## <a name="what-are-some-of-the-major-tasks-in-data-pre-processing"></a>Quelles sont les principales opérations effectuées lors du prétraitement des données ?
* **Nettoyage des données** :  compléter les valeurs manquantes, détecter et supprimer les données bruyantes et les aberrations.
* **Transformation des données** :  normaliser les données pour réduire le volume et le bruit.
* **Réduction des données** :  échantillonner les enregistrements de données ou les attributs pour faciliter la manipulation des données.
* **Discrétisation des données** :  convertir des attributs continus en attributs catégoriels pour simplifier l’exploitation des données dans certains outils d’apprentissage automatique.
* **Nettoyage du texte** : notamment supprimer les caractères incorporés pouvant perturber l’alignement des données (par exemple, les tabulations incorporées dans un fichier TSV) ou les nouvelles lignes qui peuvent interrompre des enregistrements.

Les sections suivantes décrivent certaines de ces étapes de traitement des données.

## <a name="how-to-deal-with-missing-values"></a>Comment gérer les valeurs manquantes ?
Si vous avez des valeurs manquantes, la première chose à faire est d’en identifier l’origine. Les méthodes les plus courantes de traitement des valeurs manquantes sont les suivantes :

* **Suppression** : supprimer les enregistrements ayant des valeurs manquantes.
* **Remplacement par une valeur factice** : remplacer des valeurs manquantes par une valeur factice. Par exemple, *inconnu* pour les valeurs catégorielles ou 0 pour les valeurs numériques.
* **Remplacement par la moyenne** : si les données manquantes sont numériques, les remplacer par la valeur moyenne.
* **Remplacement par l’élément le plus fréquent** : si les données manquantes sont catégorielles, remplacer les valeurs manquantes par l’élément le plus fréquent.
* **Remplacement par la valeur de régression** : remplacer les valeurs manquantes par des valeurs de régression.  

## <a name="how-to-normalize-data"></a>Comment normaliser les données ?
La normalisation des données restreint les valeurs numériques à une plage spécifiée. Les méthodes de normalisation les plus courantes sont les suivantes :

* **Normalisation min-max** : adapter linéairement les données à une plage comprise, par exemple, entre 0 et 1. La valeur minimale est 0 et la valeur maximale est 1.
* **Normalisation par le test Z** : mettre les données à l’échelle en fonction de la moyenne et de l’écart standard : diviser la différence entre les données et la moyenne par l’écart standard.
* **Mise à l’échelle décimale** : mettre les données à l’échelle en déplaçant le séparateur décimal de la valeur de l’attribut.  

## <a name="how-to-discretize-data"></a>Comment discrétiser les données ?
Pour discrétiser les données, il faut convertir les valeurs continues en attributs ou intervalles nominaux. Plusieurs méthodes permettent d’effectuer cette opération :

* **Compartimentage à largeur identique** : diviser la plage de toutes les valeurs possibles d’un attribut en N groupes de même taille et attribuer aux valeurs le numéro de compartiment qui leur correspond.
* **Compartimentage à hauteur identique** : diviser la plage de toutes les valeurs possibles d’un attribut en N groupes contenant le même nombre d’instances, puis attribuer à aux valeurs le numéro de compartiment qui leur correspond.  

## <a name="how-to-reduce-data"></a>Comment réduire les données ?
Plusieurs méthodes permettent de réduire la taille des données pour en faciliter la manipulation. Selon la taille et le domaine, les méthodes applicables sont les suivantes :

* **Échantillonnage des enregistrements** : échantillonner les enregistrements de données et ne choisir que le sous-ensemble représentatif.
* **Échantillonnage des attributs** : ne sélectionner que les attributs importants dans les données.  
* **Agrégation** : diviser les données en groupes et stocker les nombres de chaque groupe. Par exemple, le chiffre d’affaires quotidien d’une chaîne de restaurants sur les 20 dernières années peut être agrégé en un chiffre d’affaires mensuel pour réduire la taille des données.  

## <a name="how-to-clean-text-data"></a>Comment nettoyer les données textuelles ?
Les **champs de texte au format tabulaire** peuvent contenir des caractères qui perturbent l’alignement des données et/ou les limites des enregistrements. Par exemple, des tabulations incorporées dans un fichier TSV peuvent décaler les colonnes, et des caractères incorporés de nouvelle ligne peuvent déstructurer des enregistrements. Une manipulation inappropriée de l’encodage pendant l’écriture ou la lecture de texte provoque une perte d’informations ou l’ajout intempestif de caractères illisibles (comme les valeurs Null) et peut empêcher une analyse correcte du texte. Soyez prudent lorsque vous analysez et modifiez des champs de texte pour corriger l’alignement et/ou extraire des données structurées de données non ou semi-structurées.

**L’exploration des données** permet d’obtenir une vue sur l’état des données. Plusieurs problèmes peuvent être détectés pendant cette étape et les méthodes de correction correspondantes, appliquées.  Il est important de s’interroger sur l’origine du problème et la manière dont il a été introduit dans les données. Ce processus permet de déterminer les étapes de traitement des données à effectuer pour résoudre ces problèmes. L’identification des cas d’usage final et des personnages peut également être utilisée pour classer par ordre de priorité l’effort de traitement des données.

## <a name="references"></a>References
> *Data Mining: Concepts et Techniques*, 3e édition, Morgan Kaufmann, 2011, Jiawei Han, Micheline Kamber et Jian Pei
> 
> 

