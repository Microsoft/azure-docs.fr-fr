---
title: 'Joindre des données : Informations de référence sur les modules'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Joindre des données dans le concepteur Azure Machine Learning pour fusionner deux jeux de données.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 11/19/2019
ms.openlocfilehash: c23dca40d50c5837bd9ff45bc3c3d7fb2581685b
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93420748"
---
# <a name="join-data"></a>Joindre des données

Cet article décrit comment utiliser le module **Joindre des données** dans le concepteur Azure Machine Learning pour fusionner deux jeux de données à l’aide d’une opération de jointure basée sur la base de données.  

## <a name="how-to-configure-join-data"></a>Configuration de Joindre des données

Pour joindre deux jeux de données, ces derniers doivent être liés par une colonne clé. Les clés composites utilisant plusieurs colonnes sont également prises en charge. 

1. Ajoutez les jeux de données que vous souhaitez combiner, puis faites glisser le module **Joindre des données** dans votre pipeline. 

    Vous pouvez rechercher le module dans la catégorie **Transformation des données**, sous **Manipulation**.

1. Connectez les jeux de données au module **Joindre des données**. 
 
1. Sélectionnez **Lancer le sélecteur de colonne** pour choisir au moins une colonne clé. Assurez-vous de choisir les colonnes pour les entrées de gauche et de droite.

    Pour une clé unique :

    Sélectionnez une colonne clé unique pour les deux entrées.
    
    Pour une clé composite :

    Sélectionnez toutes les colonnes clés dans le même ordre pour l’entrée de gauche et de droite. Le module **Joindre des données** joindra les tables lorsque toutes les colonnes clés correspondront. Activez l’option **Autoriser les doublons et conserver l’ordre des colonnes de la sélection** si l’ordre des colonnes n’est pas identique à celui de la table d’origine. 

    ![sélecteur de colonne](media/module/join-data-column-selector.png)


1. Sélectionnez l’option **Respecter la casse** si vous souhaitez conserver le respect de la casse sur une jointure de colonne de texte. 
   
1. Utilisez la liste déroulante **Type de jointure** pour spécifier la méthode de combinaison des jeux de données.  
  
    * **Jointure interne** : une *jointure interne* est l’opération de jointure la plus courante. Elle renvoie les lignes combinées uniquement lorsque les valeurs des colonnes clés correspondent.  
  
    * **Jointure externe gauche** : une *jointure externe gauche* renvoie les lignes jointes pour toutes les lignes issues de la table de gauche. Lorsqu’une ligne dans la table de gauche ne présente aucune ligne correspondante dans la table de droite, la ligne retournée contient des valeurs manquantes pour toutes les colonnes issues de la table de droite. Vous pouvez également spécifier une valeur de remplacement pour les valeurs manquantes.  
  
    * **Jointure externe entière** : une *jointure externe complète* renvoie toutes les lignes de la table de gauche (**table1**) et de la table de droite (**table2**).  
  
         Pour chacune des lignes d’une des tables qui ne correspond à aucune ligne dans l’autre, le résultat inclut une ligne contenant les valeurs manquantes.  
  
    * **Semi-jointure gauche** : une *semi-jointure gauche* renvoie uniquement les valeurs issues de la table de gauche lorsque les valeurs des colonnes clés correspondent.  

1. Pour l’option **Conserver les colonnes clés de droite dans la table jointe** :

    * Sélectionnez cette option pour afficher les clés des deux tables d’entrée.
    * Désélectionnez-la pour retourner uniquement les colonnes clés issues de l’entrée de gauche.

1. Envoyez le pipeline.

1. Pour afficher les résultats, cliquez avec le bouton droit sur **Joindre des données**, puis sélectionnez **Visualiser**.

## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 