---
title: 'ML Studio (classique) : Prédire les réponses avec des modèles de régression – Azure'
description: Comment créer un modèle de régression simple pour prédire un prix dans la quatrième vidéo de la série Science des données pour les débutants 4. Comprend une régression linéaire avec des données cibles.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: sdgilley
ms.author: sgilley
ms.custom: seodec18
ms.date: 03/22/2019
ms.openlocfilehash: c02abf095beefdd131a5835d015175d56a63de47
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322819"
---
# <a name="predict-an-answer-with-a-simple-model"></a>Prédire une réponse à l’aide d’un modèle simple


## <a name="video-4-data-science-for-beginners-series"></a>Vidéo 4 : série Science des données pour les débutants
Découvrez comment créer un modèle de régression simple pour prédire le prix d’un diamant dans la quatrième vidéo de la série Science des données pour les débutants 4. Nous allons dessiner un modèle de régression avec des données cibles.

Pour tirer le meilleur parti de la série, regardez l’ensemble des vidéos. [Accéder à la liste des vidéos](#other-videos-in-this-series)
<br>

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/data-science-for-beginners-series-predict-an-answer-with-a-simple-model/player]
>
>

## <a name="other-videos-in-this-series"></a>Autres vidéos de cette série
*Science des données pour les débutants* offre une introduction rapide à la science des données en cinq petites vidéos.

* Vidéo 1 : [Les 5 questions auxquelles la science des données répond](data-science-for-beginners-the-5-questions-data-science-answers.md) *(5 min 14 s)*
* Vidéo 2 : [Vos données sont-elles prêtes pour la science des données ?](data-science-for-beginners-is-your-data-ready-for-data-science.md) *(4 min 56 sec)*
* Vidéo 3 : [Poser une question à laquelle les données permettent de répondre](data-science-for-beginners-ask-a-question-you-can-answer-with-data.md) *(4 min 17 s)*
* Vidéo 4 : Prédire une réponse à l’aide d’un modèle simple
* Vidéo 5 : [Copier le travail d’autres personnes pour des projets de science des données](data-science-for-beginners-copy-other-peoples-work-to-do-data-science.md) *(3 min 18 s)*

## <a name="transcript-predict-an-answer-with-a-simple-model"></a>Transcription : Prédire une réponse à l’aide d’un modèle simple
Bienvenue dans la quatrième vidéo de la série « Science des données pour les débutants ». Dans cette vidéo, nous allons générer un modèle simple et établir une prévision.

Un *modèle* est un scénario simplifié de nos données. Je vais vous montrer ce que cela signifie.

## <a name="collect-relevant-accurate-connected-enough-data"></a>Collecter assez de données pertinentes, précises et connectées
Supposons que je souhaite acheter un diamant. J’ai une bague qui appartenait à ma grand-mère avec un diamant de 1,35 carat, et je souhaite me faire une idée de sa valeur. J’emmène un bloc-notes et un stylo chez un bijoutier et note les prix de tous les diamants exposés et leur poids en carats. En commençant par le premier diamant : il pèse 1,01 carat et coûte 7 366 $.

Je procède de la même manière pour tous les diamants dans le magasin.

![Colonnes de données des diamants](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/diamond-data.png)

Notez que notre liste comprend deux colonnes. Chaque colonne a un attribut différent (poids en carats et prix) et chaque ligne est un point de données qui représente un diamant unique.

Nous venons de créer un petit jeu de données : une table. Notez qu’elle répond à nos critères de qualité :

* Les données sont **pertinentes** (le poids est sans aucun doute lié au prix)
* Elles sont **précises** (nous avons vérifié plusieurs fois les prix notés)
* Elles sont **connectées** (toutes les entrées du tableau sont renseignées)
* Et, comme nous allons le voir, nous disposons de **suffisamment** de données pour répondre à notre question

## <a name="ask-a-sharp-question"></a>Poser une question précise
Maintenant, nous allons poser notre question de façon directe : « Combien coûte un diamant de 1,35 carat ? »

Notre liste ne comprend pas de diamant de 1,35 carat, et nous devrons donc à utiliser le reste de nos données pour obtenir une réponse à la question.

## <a name="plot-the-existing-data"></a>Tracer les données existantes
La première chose que nous allons faire est de dessiner une ligne horizontale chiffrée, appelée un axe, pour représenter les poids. La plage de poids va de 0 à 2. Nous commençons donc par dessiner une ligne qui couvre cette plage et placer des graduations pour chaque demi-carat.

Nous dessinons ensuite un axe vertical pour représenter le prix et le connecter à l’axe horizontal des poids. Il sera exprimé en dollars. Nous disposons maintenant d’un ensemble d’axes de coordonnées.

![Axes des poids et des prix](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/weight-and-price-axes.png)

Maintenant , nous allons utiliser ces données et les transformer en un *nuage de points*. Il s’agit d’un excellent moyen de visualiser des jeux de données numériques.

Pour le premier point de données, nous visualisons une ligne verticale à 1,01 carat. Puis nous visualisons une ligne horizontale à 7 366 $. Nous dessinons un point à l’endroit où elles se rencontrent. Il représente le premier diamant.

Ensuite, nous procédons de la même manière avec chaque diamant de la liste. Lorsque nous en avons terminé, voici ce que nous obtenons : un ensemble de points représentant chacun un diamant.

![nuage de points](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/scatter-plot.png)

## <a name="draw-the-model-through-the-data-points"></a>Dessiner le modèle sur la base des points de données
Maintenant, si vous regardez les points et plissez les yeux, la collection ressemble à une large ligne floue. Nous pouvons prendre notre marqueur et tracer une ligne droite.

En dessinant une ligne, nous avons créé un *modèle*. Cela revient à prendre le monde réel et à en créer une version illustrée très simplifiée. Mais l’illustration est erronée : la ligne ne passe pas à travers tous les points de données. Mais il s’agit d’une simplification utile.

![Ligne de régression linéaire](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/linear-regression-line.png)

Le fait que la ligne ne passe pas exactement par tous les points n’est pas un problème. Les chercheurs de données expliquent cela en disant qu’il existe le modèle (la ligne) et que du *bruit* ou une *variance* peuvent être ensuite associés à chaque point. Nous avons d’un côté la relation sous-jacente parfaite, et de l’autre le monde réel moins clair qui ajoute du bruit et de l’incertitude.

Étant donné que nous essayons de répondre à la question *Combien ?* , il s’agit une *régression*. Et comme nous utilisons une ligne droite, c’est une *régression linéaire*.

## <a name="use-the-model-to-find-the-answer"></a>Utiliser le modèle pour trouver la réponse
Nous avons maintenant un modèle et nous lui posons notre question : Combien coûte un diamant de 1,35 carat ?

Pour répondre à notre question, nous visualisons 1,35 carat et dessinons une ligne verticale. À l’endroit où elle coupe la ligne du modèle, nous visualisons une ligne horizontale vers l’axe des dollars. Elle atteint 10 000. Et voilà ! Voici la réponse : Un diamant de 1,35 carat coûte environ 10 000 $.

![Rechercher la réponse en utilisant le modèle](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/find-the-answer.png)

## <a name="create-a-confidence-interval"></a>Créer un intervalle de confiance
Il est normal de vous demander le degré de précision de cette prédiction. Il est utile de savoir si le prix du diamant de 1,35 carat est très proche de 10 000 $, ou largement supérieur ou inférieur. Pour le déterminer, nous allons dessiner une enveloppe autour de la ligne de régression qui inclut la plupart des points. Il s’agit de notre *intervalle de confiance*  : nous sommes pratiquement certains que les prix se situent dans cet intervalle, car c’était le cas dans la plupart des cas dans le passé. Nous pouvons dessiner deux lignes horizontales supplémentaires où la ligne de 1,35 carat coupe le haut et le bas de l’intervalle.

![intervalle de confiance](./media/data-science-for-beginners-predict-an-answer-with-a-simple-model/confidence-interval.png)

Maintenant nous pouvons affirmer quelque chose sur notre intervalle de confiance :  nous pouvons dire en toute confiance que le prix d’un diamant de 1,35 carat est d’environ 10 000 $, et qu’il pourrait se trouver dans une plage allant de 8 000 $ à 12 000 $.

## <a name="were-done-with-no-math-or-computers"></a>Nous avons terminé, sans mathématiques ni ordinateur
Nous avons fait ce pour quoi les scientifiques sont payés, et ce simplement en dessinant :

* Nous avons posé une question à laquelle il était possible de répondre avec des données
* Nous avons créé un *modèle* à l’aide de la *régression linéaire*
* Nous avons fait une *prédiction* et l’avons affinée avec un *intervalle de confiance*

Et nous n’avons utilisé ni mathématiques ni ordinateur.

Maintenant, si nous disposions de plus d’informations telles que...

* la coupe du diamant,
* les variations de couleur (le niveau de blancheur du diamant),
* et le nombre d’inclusions dans le diamant,

...nous aurions plus de colonnes. Dans ce cas, les mathématiques se révèlent utiles. Si vous avez plus de deux colonnes, il est difficile de dessiner les points sur du papier. Les mathématiques vous permettent de tracer correctement cette ligne avec vos données.

Et, si nous avions deux mille ou deux millions de diamants au lieu de quelques uns, ce travail peut être réalisé beaucoup plus rapidement avec un ordinateur.

Aujourd’hui, nous avons expliqué comment effectuer une régression linéaire, et nous avons réalisé une prédiction à l’aide de données.

Nous vous invitons à regarder les autres vidéos de la série « Science des données pour les débutants » de Microsoft Azure Machine Learning Studio (classique).

## <a name="next-steps"></a>Étapes suivantes
* [Menez une première expérience de science des données avec Machine Learning Studio (classique)](create-experiment.md)
* [Consultez la présentation de Machine Learning sur Microsoft Azure](../overview-what-is-azure-ml.md)