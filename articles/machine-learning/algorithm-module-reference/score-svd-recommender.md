---
title: 'Noter le générateur de recommandations SVD : sur le module Modifier les métadonnées'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module Noter le générateur de recommandations SVD dans Azure Machine Learning pour noter les prédictions de recommandation pour un jeu de données.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 08/10/2020
ms.openlocfilehash: bf62fa995724b8e1fff757e89945cc39db3d9842
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90893715"
---
# <a name="score-svd-recommender"></a>Noter le générateur de recommandations SVD

Cet article décrit comment utiliser le module Noter le générateur de recommandations SVD dans le concepteur Azure Machine Learning. Utilisez ce module pour créer des prédictions à l’aide d’un modèle de recommandation entraîné en fonction de l’algorithme SVD (décomposition à valeur unique).

Le générateur de recommandations SVD peut générer deux différents types de prédictions :

- [Prédire des notations pour un utilisateur et un élément donné](#prediction-of-ratings)
- [Recommander des éléments à un utilisateur](#recommendations-for-users)

Lorsque vous créez le second type de prédictions, vous pouvez utiliser l’un des modes suivants :

- Le **mode de production** prend en compte tous les utilisateurs ou éléments. Il est généralement utilisé dans un service web.

  Vous pouvez créer des scores pour les nouveaux utilisateurs, pas seulement pour les utilisateurs vus au cours de l’apprentissage. Pour plus d’informations, consultez les [notes techniques](#technical-notes). 

- Le **mode d’évaluation** fonctionne sur un ensemble réduit d’utilisateurs ou d’éléments pouvant être évalués. Il est généralement utilisé pendant les opérations de pipeline.

Pour plus d’informations sur l’algorithme de générateur de recommandations SVD, consultez l’article scientifique [Matrix factorization techniques for recommender systems](https://datajobs.com/data-science-repo/Recommender-Systems-[Netflix].pdf).

## <a name="how-to-configure-score-svd-recommender"></a>Comment configurer le module Noter le générateur de recommandations SVD

Ce module prend en charge deux types de prédictions, ayant chacun des exigences différentes. 

###  <a name="prediction-of-ratings"></a>Prédiction des notations

Lorsque vous prédisez des notations, le modèle calcule la manière dont un utilisateur réagira à un élément particulier, en fonction des données d’entraînement. Les données d’entrée pour le scoring doivent fournir à la fois un utilisateur et l’élément à noter.

1. Ajoutez un modèle de recommandation formé à votre pipeline et connectez-le au **Générateur de recommandations SVD entraîné**. Vous devez créer le modèle à l’aide du module [Entraîner le générateur de recommandations SVD](train-SVD-recommender.md).

2. Pour **Recommender prediction kind** (Type de prédiction de recommandation), sélectionnez **Rating Prediction** (Prédiction de notations). Aucun autre paramètre n’est obligatoire.

3. Ajoutez les données pour lesquelles vous souhaitez effectuer des prédictions et connectez-les au **Jeu de données à noter**.

   Pour que le modèle puisse prédire les notations, le jeu de données d’entrée doit contenir des paires utilisateur-élément.

   Le jeu de données peut contenir une troisième colonne de notations facultative pour la paire utilisateur-élément figurant dans les première et deuxième colonnes. Mais la troisième colonne sera ignorée pendant la prédiction.

4. Envoyez le pipeline.

### <a name="results-for-rating-predictions"></a>Résultats des prédictions de notation 

Le jeu de données de sortie contient trois colonnes : utilisateurs, éléments et la notation prédite pour chaque utilisateur et élément d’entrée.

###  <a name="recommendations-for-users"></a>Recommandations pour les utilisateurs 

Pour recommander des éléments pour les utilisateurs, vous fournissez une liste d’utilisateurs et d’éléments comme entrée. À partir de ces données, le modèle utilise ses connaissances sur les éléments et les utilisateurs existants afin de générer une liste d’éléments avec un attrait probable pour chaque utilisateur. Vous pouvez personnaliser le nombre de recommandations retournées. Vous pouvez également définir un seuil pour le nombre de recommandations précédentes requises pour générer une recommandation.

1. Ajoutez un modèle de recommandation formé à votre pipeline et connectez-le au **Générateur de recommandations SVD entraîné**.  Vous devez créer le modèle à l’aide du module [Entraîner le générateur de recommandations SVD](train-svd-recommender.md).

2. Pour recommander des éléments pour une liste d’utilisateurs, définissez **Recommender prediction kind** (Type de prédiction de recommandation) sur **Item Recommendation** (Recommandation d’éléments).

3. Pour **Recommended item selection** (Sélection d’éléments recommandés), indiquez si vous utilisez le module de scoring en production ou pour l’évaluation de modèle. Choisissez l’une des valeurs suivantes :

    - **À partir de tous les éléments** : Sélectionnez cette option si vous configurez un pipeline pour l’utiliser dans un service web ou en production.  Cette option active le *mode de production*. Le module émet des recommandations à partir de tous les éléments vus pendant l’entraînement.

    - **À partir d’éléments notés (pour l’évaluation du modèle)** : Sélectionnez cette option si vous développez ou testez un modèle. Cette option active le *mode d’évaluation*. Le module émet des recommandations uniquement à partir des éléments du jeu de données d’entrée qui ont été évalués.
    
    - **À partir d’éléments non notés (pour suggérer de nouveaux éléments aux utilisateurs)** : Sélectionnez cette option si vous voulez que le module émette des recommandations uniquement à partir des éléments du jeu de données d’entrée qui n’ont pas été évalués. 

4. Ajoutez le jeu de données pour lequel vous souhaitez effectuer des prédictions et connectez-le au **Jeu de données à noter**.

    - Pour **À partir de tous les éléments**, le jeu de données d’entrée doit être composé d’une seule colonne. Il contient les identificateurs des utilisateurs pour lesquels il convient d’émettre des recommandations.

      Le jeu de données peut inclure deux colonnes supplémentaires d’identificateurs d’éléments et d’évaluations, mais ces deux colonnes sont ignorées. 

    - Pour **À partir d’éléments notés (pour l’évaluation du modèle)** , le jeu de données d’entrée doit être composé de paires utilisateur-élément. La première colonne doit contenir l’identificateur d’utilisateur. La deuxième colonne doit contenir les identificateurs d’éléments correspondants.

      Le jeu de données peut inclure une troisième colonne de notations utilisateur-élément, mais cette colonne est ignorée.

    - Pour **À partir d’éléments non notés (pour suggérer de nouveaux éléments aux utilisateurs)** , le jeu de données d’entrée doit être composé de paires utilisateur-élément. La première colonne doit contenir l’identificateur d’utilisateur. La deuxième colonne doit contenir les identificateurs d’éléments correspondants.

     Le jeu de données peut inclure une troisième colonne de notations utilisateur-élément, mais cette colonne est ignorée.

5. **Nombre maximal d’éléments à recommander à un utilisateur** : Entrez le nombre d’éléments à retourner pour chaque utilisateur. Par défaut, le module recommande cinq éléments.

6. **Taille minimale du pool de recommandations par utilisateur** : Entrez une valeur qui indique le nombre de recommandations antérieures requises. Par défaut, ce paramètre est défini sur 2, ce qui signifie qu’au moins deux autres utilisateurs doivent avoir recommandé l’élément.

   Utilisez cette option seulement si vous effectuez un scoring en mode d’évaluation. L’option n’est pas disponible si vous sélectionnez **À partir de tous les éléments** ou **À partir d’éléments non notés (pour suggérer de nouveaux éléments aux utilisateurs)** .

7.  Pour **À partir d’éléments non notés (pour suggérer de nouveaux éléments aux utilisateurs)** , utilisez le troisième port d’entrée nommé **Données d’entraînement** pour supprimer les éléments qui ont déjà été notés dans les résultats de prédiction.

    Pour appliquer ce filtre, connectez le jeu de données d’entraînement d’origine au port d’entrée.

8. Envoyez le pipeline.

### <a name="results-of-item-recommendation"></a>Résultats de la recommandation d’élément

Le jeu de données noté retourné par le module Noter le générateur de recommandations SVD liste les éléments recommandés pour chaque utilisateur :

- La première colonne contient les identificateurs d’utilisateur.
- Plusieurs colonnes supplémentaires sont générées, en fonction de la valeur que vous avez définie pour **Maximum number of items to recommend to a user** (Nombre maximal d’éléments à recommander à un utilisateur). Chaque colonne contient un élément recommandé (par identificateur). Les recommandations sont classées par ordre d’affinité utilisateur-élément. L’élément dont l’affinité est la plus élevée est placé dans la colonne **Élément 1**.


##  <a name="technical-notes"></a>Notes techniques

Si vous avez un pipeline avec le générateur de recommandations SVD et que vous placez le modèle en production, soyez conscient qu’il existe des différences clés entre l’utilisation du générateur de recommandations en mode d’évaluation et en mode de production.

L’évaluation, par définition, requiert des prédictions qui peuvent être vérifiées par rapport à la *vérité de terrain* dans un jeu de test. Lorsque vous évaluez le générateur de recommandations, il doit prédire uniquement les éléments qui ont été notés dans le jeu de test. Cela restreint les valeurs possibles qui sont prédites.

Lorsque vous faites fonctionner le modèle, vous modifiez généralement le mode de prédiction pour effectuer des recommandations en fonction de tous les éléments possibles, afin d’obtenir les meilleures prédictions. Pour un grand nombre de ces prédictions, il n’y a pas de réalité du terrain correspondante. Par conséquent, la justesse de la recommandation ne peut pas être vérifiée de la même façon que lors des opérations de pipeline.


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 
