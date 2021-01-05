---
title: SMOTE
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module SMOTE dans Azure Machine Learning pour augmenter le nombre d’exemples à faible incidence dans un jeu de données à l’aide d’un suréchantillonnage.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 10/16/2019
ms.openlocfilehash: 501f3e8946023d28d67a33fbbfca661afbc6306d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90898278"
---
# <a name="smote"></a>SMOTE

Cet article explique comment utiliser le module SMOTE dans le concepteur Azure Machine Learning pour augmenter le nombre de cas sous-représentés dans un jeu de données utilisé pour le machine learning. SMOTE représente un meilleur moyen d’augmenter le nombre de cas rares que la simple duplication de cas existants.  

Le module SMOTE doit être connecté à un jeu de données *déséquilibré*. Il existe de nombreuses raisons pour lesquelles un jeu de données peut être déséquilibré. Par exemple, la catégorie ciblée est rare dans la population ou les données sont difficiles à collecter. En général, SMOTE doit être utilisé lorsque la *classe* à analyser est sous-représentée. 
  
Le module retourne un jeu de données contenant les échantillons d’origine. Il retourne également un nombre d’échantillons minoritaires synthétiques, selon le pourcentage spécifié.  
  
## <a name="more-about-smote"></a>Informations complémentaires sur SMOTE

SMOTE (Synthetic Minority Oversampling Technique) est une technique statistique permettant d’augmenter le nombre de cas d’un jeu de données de façon équilibrée. Le module fonctionne par génération de nouvelles instances à partir de cas minoritaires fournis en entrée. Cette implémentation de SMOTE ne modifie *pas* le nombre de cas majoritaires.

Les nouvelles instances ne sont pas seulement des copies des cas minoritaires existants. En réalité, l’algorithme prend des échantillons de l’*espace de caractéristiques* pour chaque classe cible et ses plus proches voisins. L’algorithme génère ensuite de nouveaux échantillons qui combinent les caractéristiques du cas cible avec des caractéristiques de ses voisins. Cette approche augmente le nombre de caractéristiques disponibles pour chaque classe et rend les échantillons plus généraux.
  
SMOTE prend en entrée l’ensemble du jeu de données, mais augmente le pourcentage des cas minoritaires uniquement. Supposons, par exemple, un jeu de données déséquilibré dans lequel seulement 1 % des cas ont la valeur cible A (classe minoritaire) et 99 % des cas la valeur B. Pour multiplier par deux le pourcentage de cas minoritaires, entrez **200** comme **Pourcentage SMOTE** dans les propriétés du module.  
  
## <a name="examples"></a>Exemples  

Nous vous recommandons d’essayer d’utiliser SMOTE avec un petit jeu de données pour voir comment il fonctionne. L’exemple suivant utilise le jeu de données Blood Donation (Don de sang) disponible dans Azure Machine Learning Designer.
  
Si vous ajoutez le jeu de données à un pipeline et que vous sélectionnez **Visualiser** sur la sortie du jeu de données, vous voyez que, parmi les 748 lignes ou cas du jeu de données, il y en a 570 (76 %) de la classe 0 et 178 (24 %) de la classe 1. Le déséquilibre n’est pas catastrophique, mais la classe 1 représente les personnes qui ont fait un don de sang : ces lignes contiennent donc l’*espace de caractéristiques* que vous souhaitez modéliser.
 
Pour augmenter le nombre de cas, vous pouvez définir la valeur du **Pourcentage SMOTE** en utilisant des multiples de 100, comme suit :

||Classe 0|Classe 1|total|  
|-|-------------|-------------|-----------|  
|Jeu de données d’origine<br /><br /> (équivalent à **Pourcentage SMOTE** = **0**)|570<br /><br /> 76 %|178<br /><br /> 24 %|748|  
|**Pourcentage SMOTE** = **100**|570<br /><br /> 62 %|356<br /><br /> 38 %|926|  
|**Pourcentage SMOTE** = **200**|570<br /><br /> 52 %|534<br /><br /> 48 %|1 104|  
|**Pourcentage SMOTE** = **300**|570<br /><br /> 44 %|712<br /><br /> 56 %|1 282|  
  
> [!WARNING]
> Le fait d’augmenter le nombre de cas avec SMOTE ne garantit pas des modèles plus précis. Essayez de définir le pipeline avec différents pourcentages, différents jeux de caractéristiques et différents nombres de plus proches voisins pour voir en quoi l’ajout de cas influe sur votre modèle.  
  
## <a name="how-to-configure-smote"></a>Configurer SMOTE
  
1.  Ajoutez le module SMOTE à votre pipeline. Vous trouverez ce module sous **Modules de transformation de données**, dans la catégorie **Manipulation**.

2. Connectez le jeu de données que vous souhaitez enrichir. Si vous souhaitez spécifier l’espace de caractéristiques qui servira à générer les nouveaux cas, soit en utilisant seulement certaines colonnes, soit en en excluant d’autres, utilisez le module [Sélectionner des colonnes dans le jeu de données](select-columns-in-dataset.md). Vous pouvez ensuite isoler les colonnes à utiliser avant de vous servir de SMOTE.
  
    Dans le cas contraire, la création de nouveaux cas avec SMOTE se base sur *toutes* les colonnes que vous fournissez en entrée. Au moins une colonne de caractéristiques est numérique.
  
3.  Veillez à ce que la colonne contenant l’étiquette, ou classe cible, soit sélectionnée. SMOTE accepte uniquement les étiquettes binaires.
  
4.  Le module SMOTE identifie automatiquement la classe minoritaire dans la colonne d’étiquette, puis obtient tous les exemples de cette classe. Toutes les colonnes ne peuvent pas avoir de valeurs NaN.
  
5.  Dans l’option **Pourcentage SMOTE**, entrez un nombre entier indiquant le pourcentage cible de cas minoritaires dans le jeu de données de sortie. Par exemple :  
  
    - Vous entrez **0**. Le module SMOTE retourne exactement le même jeu de données que celui que vous avez fourni en entrée. Il n’ajoute aucun nouveau cas minoritaire. Dans ce jeu de données, la proportion des classes n’a pas changé.  
  
    - Vous entrez **100**. Le module SMOTE génère de nouveaux cas minoritaires. Il ajoute le même nombre de cas minoritaires que dans le jeu de données d’origine. Comme SMOTE n’augmente pas le nombre de cas majoritaires, la proportion des cas de chaque classe a changé.  
  
    - Vous entrez **200**. Le module double le pourcentage de cas minoritaires par rapport au jeu de données d’origine. Cela ne signifie *pas* obtenir deux fois plus de cas minoritaires qu’avant. Au lieu de cela, la taille du jeu de données est augmentée de telle sorte que le nombre de cas majoritaires reste le même. Le nombre de cas minoritaires est augmenté jusqu’à ce qu’il corresponde à la valeur de pourcentage souhaitée.  
  
    > [!NOTE]
    > Utilisez uniquement des multiples de 100 comme pourcentage SMOTE.

6.  Utilisez l’option **Number of nearest neighbors** (Nombre de plus proches voisins) pour déterminer la taille de l’espace de caractéristiques utilisé par l’algorithme SMOTE lors de la génération de nouveaux cas. Un plus proche voisin est une ligne de données (un cas) similaire à un cas cible. La distance entre deux cas se mesure en combinant les vecteurs pondérés de toutes les caractéristiques.  
  
    + En augmentant le nombre de plus proches voisins, on obtient des caractéristiques à partir de plus de cas.
    + En conservant un petit nombre de plus proches voisins, on utilise des caractéristiques plus proches de celles de l’échantillon d’origine.  
  
7. Entrez une valeur dans la zone **Random seed** (Valeur de départ aléatoire) si vous souhaitez obtenir les mêmes résultats à chaque exécution du même pipeline avec les mêmes données. Sinon, le module génèrera une valeur de départ aléatoire à partir des valeurs d’horloge du processeur au moment du déploiement du pipeline. La génération d’une valeur de départ aléatoire peut provoquer de légères différences entre les résultats au fil des exécutions.

8. Envoyez le pipeline.  
  
   La sortie du module est un jeu de données contenant les lignes d’origine ainsi que diverses lignes ajoutées avec des cas minoritaires.  

## <a name="technical-notes"></a>Notes techniques

+ Lorsque vous publiez un modèle qui utilise le module **SMOTE**, supprimez **SMOTE** du pipeline prédictif avant qu’il soit publié en tant que service web . En effet, SMOTE est destiné à l’amélioration d’un modèle pendant l’entraînement, et non au scoring. Une erreur pourrait se produire si le pipeline prédictif publié contenait le module SMOTE.

+ On obtient souvent de meilleurs résultats en nettoyant les valeurs manquantes ou en appliquant d’autres transformations pour corriger les données avant d’appliquer SMOTE. 

+ Certains chercheurs se sont demandé si SMOTE était efficace sur des données de grande dimension ou des données éparses, comme les données utilisées dans les jeux de données de classification de texte ou de génomique. Cet article offre un bon résumé des effets et de la validité théorique de l’application de SMOTE dans ce type de cas : [Blagus et Lusa : SMOTE for high-dimensional class-imbalanced data](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-14-106).

+ Si SMOTE n’est pas efficace dans votre jeu de données, vous pouvez envisager d’autres approches :
  + Méthodes pour suréchantillonner les cas minoritaires ou sous-échantillonner les cas majoritaires.
  + Des techniques d’ensemble qui aident directement l’apprenant, à l’aide du clustering, du bagging ou de l’adaptive boosting.


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 

