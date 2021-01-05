---
title: Utiliser le module Score Wide et Deep Recommender
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser le module du générateur de recommandations Score Wide and Deep dans Azure Machine Learning pour noter les prédictions de recommandation pour un jeu de données.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 08/12/2020
ms.openlocfilehash: 9a1a3892e6a47aabd9b5129ca551900494616bc8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90905165"
---
# <a name="score-wide-and-deep-recommender"></a>Générateur de recommandations Score Wide and Deep

Cet article explique comment utiliser le module de **Recommandation Score Wide and Deep** dans le concepteur Azure Machine Learning pour créer des prédictions basées sur un modèle de recommandation formé, sur la base de la formation Wide and Deep Google.

Le générateur de recommandations Wide and Deep peut générer deux différents types de prédictions :

- [Prédire des notations pour un utilisateur et un élément donné](#predict-ratings)

- [Recommander des éléments à un utilisateur donné](#recommend-items)


Lorsque vous créez le dernier type de prédictions, vous pouvez travailler en *mode de production* ou en *mode d’évaluation*.

- Le **mode de production** prend en compte tous les utilisateurs ou éléments, et est généralement utilisé dans un service web. Vous pouvez créer des scores pour les nouveaux utilisateurs, pas seulement pour les utilisateurs vus au cours de l’apprentissage. 

- Le **mode d’évaluation** fonctionne sur un ensemble réduit d’utilisateurs ou d’éléments qui peut être évalué, et est généralement utilisé pendant l’expérimentation.

Pour plus d'informations sur le générateur de recommandations Wide and Deep et sa théorie sous-jacente, consultez l'étude qui lui est consacrée :  [Wide & Deep Learning pour les systèmes de recommandation](https://arxiv.org/pdf/1606.07792.pdf).  

## <a name="how-to-configure-score-wide-and-deep-recommender"></a>Configuration du générateur de recommandations Score Wide and Deep

Ce module prend en charge différents types de recommandations, chacune avec des exigences différentes. Cliquez sur le lien correspondant au type de données que vous avez et au type de recommandation que vous souhaitez créer.

+ [Prédire des notations](#predict-ratings)
+ [Recommander des éléments](#recommend-items)

### <a name="predict-ratings"></a>Prédire des notations

Lorsque vous prédisez des notations, le modèle calcule la manière dont un utilisateur donné réagira à un élément particulier, en fonction des données d’apprentissage. Par conséquent, les données d’entrée pour le calcul de score doivent fournir à la fois l’utilisateur et l’élément à évaluer.

1. Ajoutez un modèle de recommandation Wide and Deep formé à votre expérience et connectez-le au **modèle de recommandation Wide and Deep formé**.  Vous devez créer le modèle à l’aide du module [Entraîner le générateur de recommandations Wide and Deep](train-wide-and-deep-recommender.md).

2. **Type de prédiction de recommandation** : Sélectionnez **Prédiction de notations**. Aucun paramètre supplémentaire n’est nécessaire.

3. Ajoutez les données pour lesquelles vous souhaitez effectuer des prédictions et connectez-les au **Jeu de données à noter**.

    Pour prédire les notations, le jeu de données d’entrée doit contenir des paires utilisateur-élément.

    Le jeu de données peut contenir une troisième colonne de notation facultative pour la paire utilisateur-élément dans la première et la deuxième colonne, mais la troisième colonne sera ignorée pendant la prédiction.

4.  (Facultatif). Si vous disposez d’un jeu de données de caractéristiques d’utilisateurs, connectez-le aux **Caractéristiques d’utilisateurs**.

    Le jeu de données de caractéristiques d’utilisateurs doit contenir l’identificateur de l’utilisateur dans la première colonne. Les colonnes restantes doivent contenir des valeurs qui caractérisent les utilisateurs, comme leur sexe, leurs préférences, leur emplacement, etc.
  
    Les caractéristiques des utilisateurs qui ont des éléments notés dans le jeu de données d’apprentissage sont ignorées par la **Recommandation Score Wide and Deep**, car elles ont déjà été apprises lors de la formation. Par conséquent, filtrez votre jeu de données à l’avance pour inclure uniquement les utilisateurs *qui démarrent à froid* ou les utilisateurs qui n’ont évalué aucun élément.

    > [!WARNING]
    > Si le modèle a été formé sans utiliser les caractéristiques d’utilisateurs, vous ne pouvez pas introduire de caractéristiques d’utilisateurs lors du scoring.

5. Si vous avez un jeu de données de caractéristiques d’éléments, vous pouvez le connecter aux **Caractéristiques d'éléments**.

    Le jeu de données de caractéristiques d’éléments doit contenir un identificateur d’élément dans la première colonne. Les autres colonnes doivent contenir des valeurs qui caractérisent les éléments.

    Les caractéristiques des éléments notés dans le jeu de données d’apprentissage sont ignorées par la **Recommandation Score Wide and Deep**, car elles ont déjà été apprises lors de la formation. Par conséquent, limitez votre jeu de données de scoring aux *éléments qui démarrent à froid* ou aux éléments qui n’ont pas été évalués par des utilisateurs.

    > [!WARNING]
    > Si le modèle a été formé sans utiliser les caractéristiques d’éléments, vous ne pouvez pas introduire de caractéristiques d’éléments lors du scoring.

7. Exécutez l’expérience.

### <a name="results-for-rating-predictions"></a>Résultats des prédictions de notation 

Le jeu de données de sortie contient trois colonnes, contenant l’utilisateur, l’élément, et la notation prédite pour chaque utilisateur et élément d’entrée.

En outre, les modifications suivantes sont appliquées au moment du scoring :

-  Pour une colonne numérique de caractéristique d’utilisateur ou d’élément, les valeurs manquantes sont automatiquement remplacées par la **moyenne** des valeurs non manquantes du jeu de formation. Pour une caractéristique catégorique, les valeurs manquantes sont remplacées par la même valeur catégorique en dehors des valeurs possibles de cette caractéristique.
-  Aucune traduction n'est appliquée aux valeurs de caractéristiques, afin de maintenir leur caractère épars.

### <a name="recommend-items"></a>Recommander des éléments

Pour recommander des éléments pour les utilisateurs, vous fournissez une liste d’utilisateurs et d’éléments comme entrée. À partir de ces données, le modèle utilise ses connaissances sur les éléments et les utilisateurs existants afin de générer une liste d’éléments avec un attrait probable pour chaque utilisateur. Vous pouvez personnaliser le nombre de recommandations renvoyées et définir un seuil pour le nombre de recommandations précédentes requises pour générer une recommandation.

1. Ajoutez un modèle de recommandation Wide and Deep formé à votre expérience et connectez-le au **modèle de recommandation Wide and Deep formé**.  Vous devez créer le modèle à l’aide du module [Entraîner le générateur de recommandations Wide and Deep](train-wide-and-deep-recommender.md).

2. Pour recommander des éléments pour une liste donnée d’utilisateurs, définissez **Type de prédiction de recommandation** sur **Recommandation d’éléments**.

3. **Sélection d’éléments recommandés** : Indiquez si vous utilisez le module de notation en production ou pour l’évaluation de modèle, en choisissant l’une des valeurs suivantes :

    - **À partir d’éléments notés (pour l’évaluation du modèle)** : Sélectionnez cette option si vous développez ou testez un modèle. Cette option active le **mode d’évaluation**, le module génère alors des recommandations uniquement à partir des éléments du jeu de données d’entrée qui ont été évalués.
    - **À partir de tous les éléments** : Sélectionnez cette option si vous configurez un essai pour une utilisation dans un service web ou en production.  Cette option active le **mode de production**, le module émet alors des recommandations à partir de tous les éléments vus pendant l’entraînement.
    - **À partir d’éléments non notés (pour suggérer de nouveaux éléments aux utilisateurs)** : Sélectionnez cette option si vous voulez que le module émette des recommandations uniquement à partir des éléments du jeu de données d’entrée qui n’ont pas été évalués. 
4. Ajoutez le jeu de données pour lequel vous souhaitez effectuer des prédictions et connectez-le au **Jeu de données à noter**.

    - Si vous choisissez l’option **À partir de tous les éléments**, le jeu de données d’entrée doit se composer d’une seule colonne, contenant les identificateurs des utilisateurs pour lesquels formuler des recommandations.

        Le jeu de données peut inclure deux colonnes supplémentaires d’identificateurs d’éléments et d’évaluations, mais ces deux colonnes sont ignorées. 

    - Si vous choisissez l’option **À partir d’éléments notés (pour l’évaluation du modèle)** , le jeu de données d’entrée doit être composé de **paires utilisateur-élément**. La première colonne doit contenir l’identificateur d’**utilisateur**. La deuxième colonne doit contenir les identificateurs d’**élément** correspondants.

        Le jeu de données peut inclure une troisième colonne de notations utilisateur-élément, mais cette colonne est ignorée.
        
    - Pour **À partir d’éléments non notés (pour suggérer de nouveaux éléments aux utilisateurs)** , le jeu de données d’entrée doit être composé de paires utilisateur-élément. La première colonne doit contenir l’identificateur d’utilisateur. La deuxième colonne doit contenir les identificateurs d’éléments correspondants.

        Le jeu de données peut inclure une troisième colonne de notations utilisateur-élément, mais cette colonne est ignorée.

5. (Facultatif). Si vous disposez d’un jeu de données de **caractéristiques d’utilisateurs**, connectez-le aux **Caractéristiques d’utilisateurs**.

    La première colonne du jeu de données des caractéristiques d’utilisateurs doit contenir l’identificateur de l’utilisateur. Les colonnes restantes doivent contenir des valeurs qui caractérisent l’utilisateur, comme le sexe, les préférences, l’emplacement, etc.

    Les caractéristiques des utilisateurs qui ont des éléments notés sont ignorées par la **Recommandation Score Wide and Deep**, car elles ont déjà été apprises lors de la formation. Par conséquent, vous pouvez filtrer votre jeu de données à l’avance pour inclure uniquement les utilisateurs *qui démarrent à froid* ou les utilisateurs qui n’ont évalué aucun élément.

    > [!WARNING]
    >  Si le modèle a été formé sans utiliser les caractéristiques d’utilisateurs, vous ne pouvez pas appliquer des caractéristiques d’utilisateurs lors du scoring.

6. (Facultatif) Si vous avez un jeu de données de **caractéristiques d’éléments**, vous pouvez le connecter aux **Caractéristiques d'éléments**.

    La première colonne du jeu de données de caractéristiques d’éléments doit contenir l’identificateur d’élément. Les autres colonnes doivent contenir des valeurs qui caractérisent les éléments.

    Les caractéristiques des éléments notés sont ignorées par la **Recommandation Score Wide and Deep**, car elles ont déjà été apprises lors de la formation. Par conséquent, vous pouvez limiter votre jeu de données de scoring aux *éléments qui démarrent à froid* ou aux éléments qui n’ont pas été évalués par des utilisateurs.

    > [!WARNING]
    >  Si le modèle a été formé sans utiliser de caractéristiques d'éléments, n'utilisez pas celles-ci lors du calcul de score.  

7. **Nombre maximal d’éléments à recommander à un utilisateur** : Saisissez le nombre d’éléments à retourner pour chaque utilisateur. Par défaut, 5 éléments sont recommandés.

8. **Taille minimale du pool de recommandations par utilisateur** : Saisissez une valeur qui indique le nombre de recommandations antérieures requises.  Par défaut, ce paramètre est défini sur 2, ce qui signifie que l’élément doit avoir été recommandé par au moins deux autres utilisateurs.

    Cette option ne doit être utilisée que si vous effectuez une notation en mode évaluation. L’option n’est pas disponible si vous sélectionnez **À partir de tous les éléments** ou **À partir d’éléments non notés (pour suggérer de nouveaux éléments aux utilisateurs)** .

9. Pour **À partir d’éléments non notés (pour suggérer de nouveaux éléments aux utilisateurs)** , utilisez le troisième port d’entrée nommé **Données d’entraînement** pour supprimer les éléments qui ont déjà été notés dans les résultats de prédiction.

    Pour appliquer ce filtre, connectez le jeu de données d’entraînement d’origine au port d’entrée.

10. Exécutez l’expérience.
### <a name="results-of-item-recommendation"></a>Résultats de la recommandation d’élément

Le jeu de données noté retourné par le **module Générateur de recommandations Score Wide and Deep** liste les éléments recommandés pour chaque utilisateur.

- La première colonne contient les identificateurs d’utilisateur.
- Plusieurs colonnes supplémentaires sont générées, en fonction de la valeur que vous avez définie pour **Nombre maximal d’éléments à recommander à un utilisateur**. Chaque colonne contient un élément recommandé (par identificateur). Les recommandations sont classées par affinité utilisateur-élément, l’élément disposant de l’affinité la plus élevée est placé dans la colonne **Élément 1**.

##  <a name="technical-notes"></a>Notes techniques

Cette section contient des réponses à certaines questions courantes sur l’utilisation du générateur de recommandations Wide and Deep pour créer des prédictions.  

###  <a name="cold-start-users-and-recommendations"></a>Utilisateurs démarrant à froid et recommandations

En règle générale, pour créer des recommandations, le module **Recommandation Score Wide and Deep** nécessite les mêmes entrées que celles que vous avez utilisées lors de la formation du modèle, y compris un ID d’utilisateur. Cela est dû au fait que l’algorithme a besoin de savoir s’il a appris des choses sur cet utilisateur au cours de la formation. 

Toutefois, s'il s'agit d'un nouvel utilisateur, vous ne disposez peut-être pas d'un identifiant, mais uniquement de certaines caractéristiques telles que l'âge, le sexe et autres.

Vous pouvez toujours générer des recommandations pour des utilisateurs nouveaux dans votre système, en les traitant comme des *utilisateurs qui démarrent à froid*. Dans ce cas, l'algorithme de recommandation n'utilise pas l'historique ou des évaluations précédentes, mais uniquement les caractéristiques de l'utilisateur.

À des fins de prédiction, un utilisateur qui démarre à froid est défini comme un utilisateur dont l'identifiant n'a pas été utilisé pour la formation. Pour vous assurer que les ID ne correspondent pas aux ID utilisés lors de la formation, vous pouvez créer des identificateurs. Par exemple, vous pouvez générer des identifiants aléatoires dans une plage spécifiée, ou allouer à l'avance une série d'identifiants aux futurs nouveaux utilisateurs.

Toutefois, si vous n'avez pas de données de filtrage collaboratives, telles qu'un vecteur de caractéristiques d'utilisateur, il est préférable d'utiliser un dispositif d'apprentissage par classification ou régression.

###  <a name="production-use-of-the-wide-and-deep-recommender"></a>Utilisation du générateur de recommandations Wide and Deep en production

Si, après avoir expérimenté le module Score Wide and Deep Recommender, vous transposez le modèle vers la production, vous devez être conscient des différences clés existant entre le générateur de recommandations en mode évaluation et en mode production :

- L’évaluation, par définition, requiert des prédictions qui peuvent être vérifiées par rapport à la *vérité de terrain* dans un jeu de test. Par conséquent, lorsque vous évaluez le générateur de recommandations, il doit prédire uniquement les éléments qui ont été évalués dans le jeu de test. Cela restreint nécessairement les valeurs possibles qui sont prédites.

    Toutefois, lorsque vous procédez au fonctionnement du modèle, vous modifiez généralement le mode de prédiction pour effectuer des recommandations en fonction de tous les éléments possibles, afin d’obtenir les meilleures prédictions. Pour la plupart de ces prédictions, il n’y a pas de vérité de terrain correspondante, ainsi la précision de la recommandation ne peut pas être vérifiée de la même façon que lors de l’expérimentation.

- Si vous ne fournissez pas d'identifiant d'utilisateur en production, mais uniquement un vecteur de caractéristiques, vous pouvez obtenir en réponse la liste de toutes les recommandations pour tous les utilisateurs possibles. Veillez à fournir un identifiant utilisateur.

    Pour limiter le nombre de recommandations renvoyées, vous pouvez également spécifier le nombre maximal d'éléments renvoyés par utilisateur. 



## <a name="next-steps"></a>Étapes suivantes

Regardez les [modules disponibles](module-reference.md) pour Azure Machine Learning. 
