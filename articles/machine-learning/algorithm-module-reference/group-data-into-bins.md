---
title: 'Grouper des données dans des compartiments : sur le module Modifier les métadonnées'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Grouper des données dans des compartiments pour regrouper des nombres ou modifier la distribution de données continues.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 10/13/2020
ms.openlocfilehash: 392cb9b4c2ded1b98b79ce8dcd780ac59e96b78a
ms.sourcegitcommit: 090ea6e8811663941827d1104b4593e29774fa19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91998507"
---
# <a name="group-data-into-bins-module"></a>Module Grouper des données dans des compartiments

Cet article explique comment utiliser le module Grouper des données dans des compartiments dans le concepteur Azure Machine Learning Designer pour regrouper des nombres ou modifier la distribution de données continues.

Le module Grouper des données dans des compartiments prend en charge plusieurs options de compartimentage des données. Vous pouvez modifier la manière dont les bords des compartiments sont définis et celle dont les valeurs sont réparties dans les compartiments. Vous pouvez par exemple :  

+ Tapez manuellement une série de valeurs pour définir les limites des compartiments.  
+ Attribuez des valeurs aux compartiments en utilisant des *quantiles*, ou classements centiles.  
+ Forcez une répartition uniforme des valeurs dans les compartiments.  

## <a name="more-about-binning-and-grouping"></a>Informations supplémentaires sur le compartimentage et le regroupement

Le *compartimentage*, ou regroupement, de données (parfois appelé *quantification*) est un outil important pour la préparation de données numériques à des fins d’apprentissage automatique. Il est utile dans des scénarios tels que les suivants :

+ Une colonne de nombres continus comprend trop de valeurs uniques pour être modélisée efficacement. Vous devez donc attribuer automatiquement ou manuellement les valeurs à des groupes pour créer un ensemble plus petit de plages discrètes.

+ Vous souhaitez remplacer une colonne de nombres par des valeurs catégorielles représentant des plages spécifiques.

    Par exemple, vous pouvez regrouper des valeurs dans une colonne Âge en spécifiant des plages personnalisées, telles que 1-15, 16-22, 23-30, etc. pour les données démographiques des utilisateurs.

+ Un jeu de données comprend quelques valeurs extrêmes, bien distinctes de la plage attendue, et ces valeurs ont une influence démesurée sur le modèle formé. Pour atténuer le biais dans le modèle, vous pouvez transformer les données en une distribution uniforme à l’aide de la méthode des quantiles.

    Avec cette méthode, le module Grouper des données dans des compartiments détermine les emplacements et largeurs de compartiments optimaux pour garantir que chaque compartiment contienne approximativement le même nombre d’échantillons. Ensuite, en fonction de la méthode de normalisation choisie, les valeurs dans les compartiments sont transformées en centiles ou mappées à un numéro de compartiment.

### <a name="examples-of-binning"></a>Exemples de compartimentage

Le diagramme suivant illustre la distribution de valeurs numériques avant et après compartimentage avec la méthode des *quantiles*. Notez qu’en comparaison des données brutes à gauche, les données ont été compartimentées et transformées en une échelle d’unité normale.  

> [!div class="mx-imgBorder"]
> ![Visualisation des résultats](media/module/group-data-into-bins-result-example.png)

Compte il existe un grand nombre de façons, toutes personnalisables, de regrouper des données, nous vous recommandons d’expérimenter avec différentes méthodes et valeurs. 

## <a name="how-to-configure-group-data-into-bins"></a>Comment configurer le module Grouper des données dans des compartiments

1. Ajoutez le module **Grouper des données dans des compartiments** à votre pipeline dans le concepteur. Ce module se trouve dans la catégorie **Transformation de données**.

2. Connectez le jeu de données contenant des données numériques à compartimenter. La quantification ne peut être appliquée qu’à des colonnes contenant des données numériques. 

    Si le jeu de données contient des colonnes non numériques, utilisez le module [Sélectionner des colonnes dans le jeu de données](select-columns-in-dataset.md) pour sélectionner un sous-ensemble de colonnes à utiliser.

3. Spécifiez le mode de compartimentage. Le mode de compartimentage déterminant d’autres paramètres, veillez à commencer par sélectionner le **Mode de compartimentage**. Les types de compartimentages suivants sont pris en charge :

    - **Quantiles** : la méthode quantile attribue des valeurs aux compartiments en fonction de classements centiles. Cette méthode est également appelée « compartimentages de hauteur égale ».

    - **Largeur égale** : Avec cette option, vous devez spécifier le nombre total de compartiments. Les valeurs de la colonne de données sont placées dans les compartiments de telle sorte que chaque compartiment présente le même intervalle entre les valeurs de début et de fin. Par conséquent, certains compartiments peuvent comporter plus de valeurs si des données sont agglutinées autour d’un certain point.

    - **Bords personnalisés** : vous pouvez spécifier les valeurs qui marquent le début de chaque compartiment. La valeur de bord est toujours la limite inférieure du compartiment. 
    
      Par exemple, supposons que vous souhaitez regrouper des valeurs dans deux compartiments : l’un avec les valeurs supérieures à 0, et l’autre avec les valeurs inférieures ou égales à 0. Dans ce cas, pour les bords d’emplacement, vous devez taper **0** dans la **liste séparée par des virgules de bords d’emplacement**. La sortie du module sera 1 et 2, indiquant l’index du compartiment pour chaque valeur de ligne. Notez que la liste de valeurs séparées par des virgules doit être classée dans un ordre croissant, par exemple 1, 3, 5, 7.
    
    > [!Note]
    > Le mode *MDL d’entropie* est défini dans Studio (classique) et il n’existe aucun package open source correspondant pouvant être utilisé pour la prise en charge de dans le concepteur.        

4. Si vous utilisez les modes de compartimentage **Quantiles** et **Largeur égale**, utilisez l’option **Nombre de compartiments** pour spécifier le nombre de compartiments, ou *quantiles*, que vous souhaitez créer.

5. Pour **Colonnes à compartimenter**, utilisez le sélecteur de colonne pour choisir les colonnes contenant les valeurs que vous voulez compartimenter. Les colonnes doivent contenir des données de type numérique.

    La même règle de compartimentage est appliquée à toutes les colonnes que vous choisissez. Si vous avez besoin de compartimenter certaines colonnes à l’aide d’une autre méthode, utilisez une instance distincte du module Grouper des données dans des compartiments pour chaque ensemble de colonnes.

    > [!WARNING]
    > Si vous choisissez une colonne qui n’est pas d’un type autorisé, une erreur d’exécution est générée. Le module retourne une erreur dès qu’il trouve une colonne d’un type non autorisé. Si vous recevez une erreur, passez en revue toutes les colonnes sélectionnées. L’erreur ne répertorie pas toutes les colonnes non valides.

6. Pour **Mode de sortie**, indiquez comment vous souhaitez sortir les valeurs quantifiées :

    + **Append**: crée une colonne avec les valeurs compartimentées et l’ajoute à la table d’entrée.

    + **Inplace** : remplace les valeurs d’origine par les nouvelles valeurs du jeu de données.

    + **ResultOnly** : retourne uniquement les colonnes de résultats.

7. Si vous sélectionnez le mode de compartimentage **Quantiles**, utilisez l’option **Normalisation par quantile** pour déterminer la manière dont les valeurs sont normalisées avant le tri en quantiles. Notez que la normalisation des valeurs transforme celles-ci, mais n’affecte pas le nombre final de compartiments.

    Les types de normalisation suivants sont pris en charge :

    + **Percent** : les valeurs sont normalisées dans la plage [0,100].

    + **PQuantile** : les valeurs sont normalisées dans la plage [0,1].

    + **QuantileIndex** :  les valeurs sont normalisées dans la plage [1,nombre de compartiments].

8. Si vous choisissez l’option **Bords personnalisés**, tapez une liste de nombres séparés par des virgules à utiliser comme *bords d’emplacement* dans la zone de texte **Liste séparée par des virgules des bords d’emplacement**. 

    Les valeurs indiquent le point qui divise les compartiments. Par exemple, si vous entrez une valeur de bord d’emplacement, deux compartiments sont générés. Si vous entrez deux valeurs de bord d’emplacement, trois compartiments sont générés.

    Les valeurs doivent être triées dans l’ordre de création des compartiments, de la plus petite à la plus grande.

10. Sélectionnez l’option **Marquer les colonnes comme catégorielles** pour indiquer que les colonnes quantifiées doivent être gérées en tant que variables catégorielles.

11. Envoyez le pipeline.

## <a name="results"></a>Résultats

Le module Grouper des données dans des compartiments retourne un jeu de données dans lequel chaque élément a été compartimenté conformément au mode spécifié. 

Il retourne également une *transformation par compartimentage*. Cette fonction peut être passée au module [Appliquer une transformation](apply-transformation.md) pour compartimenter de nouveaux échantillons de données en utilisant les mêmes modes et paramètres de compartimentage.  

> [!TIP]
> Si vous utilisez un compartimentage sur vos données d’apprentissage, vous devez utiliser la même méthode de compartimentage sur les données que vous utilisez à des fins de test et de prédiction. Vous devez également utiliser les mêmes emplacements de compartiment et les mêmes largeurs de compartiment. 
> 
> Pour vous assurer que les données sont toujours transformées à l’aide de la même méthode de compartimentage, nous vous recommandons d’enregistrer les transformations de données utiles. Vous pouvez ensuite les appliquer à d’autres jeux de données à l’aide du module [Appliquer une transformation](apply-transformation.md).

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 
