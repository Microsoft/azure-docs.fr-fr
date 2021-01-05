---
title: 'Convertir au format CSV : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Convertir au format CSV dans le concepteur Azure Machine Learning Designer pour convertir un jeu de données en un fichier CSV réutilisable.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 10/22/2019
ms.openlocfilehash: cc58689e30e9b03e490c0871f3decd50372371fa
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93421921"
---
# <a name="convert-to-csv-module"></a>Module Convertir au format CSV

Cet article décrit un module dans le concepteur Azure Machine Learning.

Utilisez ce module pour convertir un jeu de données dans un fichier au format CSV qui peut être téléchargé, exporté ou partagé avec des modules de script R ou Python.

### <a name="more-about-the-csv-format"></a>Plus d’informations sur le format CSV 

Le format CSV, qui signifie « valeurs séparées par des virgules », est un format de fichier utilisé par de nombreux outils de Machine Learning externes. CSV est un format d’échange courant lorsque vous utilisez des langages open source tels que R ou Python.

Même si vous effectuez la majeure partie de votre travail dans Azure Machine Learning, il peut parfois s’avérer pratique de convertir votre jeu de données au format CSV pour l’utiliser dans des outils externes. Par exemple :

+ Téléchargez le fichier CSV pour l’ouvrir avec Excel, ou importez-le dans une base de données relationnelle.  
+ Enregistrez le fichier CSV dans le stockage cloud et connectez-vous à celui-ci depuis Power BI pour créer des visualisations.  
+ Utilisez le format CSV pour préparer des données pour une utilisation dans R et Python. 

Lorsque vous convertissez un jeu de données au format CSV, le fichier csv est enregistré dans votre espace de travail Azure ML. Vous pouvez utiliser un utilitaire de stockage Azure pour ouvrir et utiliser le fichier directement. Vous pouvez également accéder au fichier CSV dans le concepteur en sélectionnant le module **Convertir au format CSV**, puis l’icône d’histogramme sous l’onglet **Sorties** dans le panneau droit pour afficher la sortie. Vous pouvez télécharger le fichier CSV à partir du dossier de résultats dans un répertoire local.  

## <a name="how-to-configure-convert-to-csv"></a>Comment configurer Convertir au format CSV


1.  Ajoutez le module Convertir au format CSV à votre pipeline. Ce module se trouve dans le groupe **Transformation des données** dans le concepteur. 

2. Connectez-le à n’importe quel module qui génère un jeu de données.   
  
3.  Envoyez le pipeline.

### <a name="results"></a>Résultats
  

Sélectionnez l’onglet **Sorties** dans le panneau droit de **Convertir au format CSV**, puis l’une de ces icônes sous **Port outputs** (Sorties de port).  

+ **Inscrire le jeu de données** : Sélectionnez l’icône et réenregistrez le fichier CSV dans l’espace de travail Azure ML comme un autre jeu de données. Vous pouvez trouver le jeu de données sous la forme d’un module dans l’arborescence du module sous la section **Mes jeux de données**.

 + **Voir la sortie** : Sélectionnez l’icône représentant un œil et suivez les instructions pour parcourir le dossier **Results_dataset**, puis téléchargez le fichier data.csv.

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 