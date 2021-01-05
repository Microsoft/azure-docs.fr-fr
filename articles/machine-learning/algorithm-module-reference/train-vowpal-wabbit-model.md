---
title: Entraîner un modèle Vowpal Wabbit
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Train Vowpal Wabbit Model (Entraîner un modèle Vowpal Wabbit) pour créer un modèle Machine Learning à l’aide d’une instance de Vowpal Wabbit.
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 07/02/2020
ms.openlocfilehash: 6bc9f69440be772910ea8200b5ccf7d5a5122ae6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90907801"
---
# <a name="train-vowpal-wabbit-model"></a>Entraîner un modèle Vowpal Wabbit
Cet article explique comment utiliser le module **Train Vowpal Wabbit Model** (Entraîner un modèle Vowpal Wabbit) dans le concepteur Azure Machine Learning pour créer un modèle Machine Learning à l’aide de Vowpal Wabbit.  

Si vous souhaitez utiliser Vowpal Wabbit pour le machine learning, formatez votre entrée conformément aux exigences de Vowpal Wabbit, et préparez les données au format nécessaire. Utilisez ce module pour spécifier les arguments de ligne de commande de Vowpal Wabbit. 

Au moment où le pipeline est exécuté, une instance de Vowpal Wabbit est chargée dans l’exécution de l’expérience en même temps que les données spécifiées. Une fois l’entraînement effectué, le modèle est resérialisé dans l’espace de travail. Vous pouvez utiliser le modèle immédiatement pour scorer les données. 

Pour entraîner de manière incrémentielle un modèle existant sur de nouvelles données, connectez un modèle enregistré au port d’entrée **Pre-trained Vowpal Wabbit model** (Modèle Vowpal Wabbit préentraîné) du module **Train Vowpal Wabbit Model**, puis ajoutez les nouvelles données à l’autre port d’entrée.  

## <a name="what-is-vowpal-wabbit"></a>Qu’est-ce que Vowpal Wabbit ?  

VW (Vowpal Wabbit) est un framework de machine learning parallèle rapide qui a été développé pour l’informatique distribuée par Yahoo! Research. Il a été porté plus tard sur Windows, puis adapté par John Langford (Microsoft Research) pour l’informatique scientifique dans les architectures parallèles.  

Les fonctionnalités de Vowpal Wabbit qui sont importantes pour le machine learning sont le continuous learning (online learning), la réduction de dimensionnalité et l’interactive learning. Vowpal Wabbit représente également une solution aux problèmes quand vous ne pouvez pas placer les données du modèle en mémoire.  

Les principaux utilisateurs de Vowpal Wabbit sont les scientifiques des données qui ont utilisé le framework pour des tâches de machine learning telles que la classification, la régression, la modélisation de thèmes ou la factorisation de matrices. Le wrapper Azure pour Vowpal Wabbit a des caractéristiques de performances très similaires à la version locale. Ainsi, vous pouvez non seulement utiliser les puissantes fonctionnalités de Vowpal Wabbit et ses performances natives, mais vous pouvez également publier facilement le modèle entraîné en tant que service opérationnel.  

Le module [Hachage des caractéristiques](feature-hashing.md) comprend également des fonctionnalités fournies par Vowpal Wabbit, qui vous permettent de transformer des jeux de données de texte en caractéristiques binaires à l’aide d’un algorithme de hachage.  

## <a name="how-to-configure-vowpal-wabbit-model"></a>Comment configurer un modèle Vowpal Wabbit  

Cette section explique comment entraîner un nouveau modèle et ajouter de nouvelles données à un modèle existant.

Contrairement aux autres modules du concepteur, ce module spécifie les paramètres du module et entraîne également le modèle. Si vous disposez d’un modèle existant, vous pouvez l’ajouter en tant qu’entrée facultative pour entraîner le modèle de manière incrémentielle.

+ [Préparer les données d’entrée dans l’un des formats nécessaires](#prepare-the-input-data)
+ [Entraîner un nouveau modèle](#create-and-train-a-vowpal-wabbit-model)
+ [Entraîner un modèle existant de manière incrémentielle](#retrain-an-existing-vowpal-wabbit-model)

### <a name="prepare-the-input-data"></a>Préparer les données d’entrée

Pour permettre l’entraînement d’un modèle à l’aide de ce module, le jeu de données d’entrée doit être composé d’une seule colonne de texte dans l’un des deux formats pris en charge : **SVMLight** ou **VW**. Cela ne signifie pas que Vowpal Wabbit analyse uniquement les données de texte, mais seulement que les caractéristiques et les valeurs doivent être préparées dans le format de fichier texte nécessaire.  

Les données peuvent être lues à partir de deux genres de jeu de données, un jeu de données de fichier ou un jeu de données tabulaire. Ces deux jeux de données doivent être au format SVMLight ou VW. Le format de données Vowpal Wabbit présente l’avantage de ne pas nécessiter un format en colonnes, ce qui permet de gagner de l’espace pour le traitement de données éparses. Pour plus d’informations sur ce format, consultez la [page wiki Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki/Input-format).  

### <a name="create-and-train-a-vowpal-wabbit-model"></a>Créer et entraîner un modèle Vowpal Wabbit

1. Ajoutez le module **Train Vowpal Wabbit Model** à votre expérience. 
  
2. Ajoutez le jeu de données d’entraînement et connectez-le aux **données d’entraînement**. Si le jeu de données d’entraînement est un répertoire qui contient le fichier de données d’entraînement, spécifiez le nom du fichier de données d’entraînement dans **Name of the training data file** (Nom du fichier de données d’entraînement). Si le jeu de données d’entraînement correspond à un seul fichier, n’indiquez rien dans **Name of the training data file**.

3. Dans la zone de texte **VW arguments** (Arguments VW), tapez les arguments de ligne de commande pour l’exécutable Vowpal Wabbit.

     Par exemple, vous pouvez ajouter *`–l`* pour spécifier le taux d’apprentissage, ou *`-b`* pour indiquer le nombre de bits de hachage.  

     Pour plus d’informations, consultez la section [Paramètres Vowpal Wabbit](#supported-and-unsupported-parameters).  

4. **Name of the training data file** (Nom du fichier de données d’entraînement) : tapez le nom du fichier qui contient les données d’entrée. Cet argument est utilisé uniquement quand le jeu de données d’entraînement est un répertoire.

5. **Specify file type** (Spécifier le type de fichier) : indiquez le format utilisé par vos données d’entraînement. Vowpal Wabbit prend en charge les deux formats de fichiers d’entrée suivants :  

    - **VW** représente le format interne utilisé par Vowpal Wabbit. Pour plus d’informations, consultez la [page wiki de Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki/Input-format). 
    - **SVMLight** est un format utilisé par d’autres outils de machine learning. 

6. **Output readable model file** (Fichier de modèle lisible en sortie) : sélectionnez l’option si vous souhaitez que le module enregistre le modèle lisible dans les enregistrements d’exécution. Cet argument correspond au paramètre `--readable_model` dans la ligne de commande VW.  

7. **Output inverted hash file** (Fichier de hachage inversé en sortie) : sélectionnez l’option si vous souhaitez que le module enregistre la fonction de hachage inversé dans les enregistrements d’exécution. Cet argument correspond au paramètre `--invert_hash` dans la ligne de commande VW.  

8. Envoyez le pipeline.

### <a name="retrain-an-existing-vowpal-wabbit-model"></a>Réentraîner un modèle Vowpal Wabbit existant

Vowpal Wabbit prend en charge l’entraînement incrémentiel via l’ajout de nouvelles données à un modèle existant. Il existe deux façons d’obtenir un modèle existant pour un réentraînement :

+ Utilisez la sortie d’un autre module **Train Vowpal Wabbit Model** dans le même pipeline.  
  
+ Recherchez un modèle enregistré dans la catégorie **Jeux de données** du volet de navigation gauche du concepteur, puis faites-le glisser vers votre pipeline.  

1. Ajoutez le module **Train Vowpal Wabbit Model** à votre pipeline.  
2. Connectez le modèle entraîné au port d’entrée **Pre-trained Vowpal Wabbit Model** (Modèle Vowpal Wabbit préentraîné) du module.
3. Connectez les nouvelles données d’entraînement au port d’entrée **Training data** (Données d’entraînement) du module.
4. Dans le volet de paramètres de **Train Vowpal Wabbit Model**, spécifiez le format des nouvelles données d’entraînement ainsi que le nom du fichier de données d’entraînement si le jeu de données d’entrée est un répertoire.
5. Sélectionnez les options **Output readable model file ** et **Output inverted hash file** si les fichiers correspondants doivent être enregistrés dans les enregistrements d’exécution.

6. Envoyez le pipeline.  
7. Sélectionnez le module, puis **Register dataset** (Inscrire le jeu de données) sous l’onglet **Outputs+logs** (Sorties+journaux) dans le volet droit, pour conserver le modèle mis à jour dans votre espace de travail Azure Machine Learning.  Si vous ne spécifiez pas de nouveau nom, le modèle mis à jour remplace le modèle enregistré existant.

## <a name="technical-notes"></a>Notes techniques

Cette section contient des détails, des conseils et des réponses aux questions fréquentes concernant l’implémentation.

### <a name="advantages-of-vowpal-wabbit"></a>Avantages de Vowpal Wabbit

Vowpal Wabbit fournit un machine learning extrêmement rapide pour les caractéristiques non linéaires telles que les n-grammes.  

Vowpal Wabbit utilise des techniques d’*online learning* telles que le SGD (descente du gradient stochastique) pour insérer, un par un, des enregistrements dans un modèle. Ainsi, il itère très vite sur les données brutes et peut développer un bon prédicteur plus rapidement que la plupart des autres modèles. Cette approche évite également d’avoir à lire toutes les données d’entraînement en mémoire.  

Vowpal Wabbit convertit toutes les données en codes de hachage, notamment les données de texte mais également les autres variables catégorielles. L’utilisation de codes de hachage rend la recherche de pondérations de régression plus efficace, qui est essentiel pour une descente de gradient stochastique efficace.  

###  <a name="supported-and-unsupported-parameters"></a>Paramètres pris en charge et non pris en charge 

Cette section décrit la prise en charge des paramètres de ligne de commande de Vowpal Wabbit dans le concepteur Azure Machine Learning. 

En règle générale, tous les arguments, à l’exception d’un ensemble limité, sont pris en charge. Pour obtenir la liste complète des arguments, consultez la [page wiki Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments).    

Les paramètres suivants ne sont pas pris en charge :

-   Les options d’entrée/de sortie spécifiées dans [https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments](https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments)  
  
     Ces propriétés sont déjà configurées automatiquement par le module.  
  
-   En outre, toute option qui génère plusieurs sorties ou accepte plusieurs entrées est interdite. Il s’agit notamment de *`--cbt`* , *`--lda`* et *`--wap`* .  
  
-   Seuls les algorithmes de machine learning supervisé sont pris en charge. Ces options ne sont donc pas prises en charge : *`–active`* , `--rank`, *`--search`* , etc. 

### <a name="restrictions"></a>Restrictions

L’objectif du service étant de prendre en charge les utilisateurs expérimentés de Vowpal Wabbit, les données d’entrée doivent être préparées à l’avance à l’aide du format de texte natif Vowpal Wabbit, au lieu du format de jeu de données utilisé par d’autres modules.

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 
