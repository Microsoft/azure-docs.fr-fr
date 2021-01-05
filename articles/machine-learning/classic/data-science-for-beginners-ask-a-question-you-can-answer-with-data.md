---
title: Poser une question à laquelle les données peuvent répondre – ML Studio (classique) – Azure
description: Découvrez comment formuler une question de science des données précise dans la vidéo 3 Science des données pour les débutants. Inclut une comparaison des questions de classification et de régression.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: sdgilley
ms.author: sgilley
ms.date: 03/22/2019
ms.openlocfilehash: dbac7c8818300ffb09959969e6b79f949177f457
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93307678"
---
# <a name="ask-a-question-you-can-answer-with-data"></a>Poser une question à laquelle les données permettent de répondre
## <a name="video-3-data-science-for-beginners-series"></a>Vidéo 3 : série Science des données pour les débutants
Découvrez comment formuler un problème de science des données en une question dans la vidéo 3 Science des données pour les débutants. Cette vidéo inclut une comparaison des questions faisant appel aux algorithmes de classification et de régression.

Pour tirer le meilleur parti de la série, regardez l’ensemble des vidéos. [Accéder à la liste des vidéos](#other-videos-in-this-series)
<br>

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Data-science-for-beginners-Ask-a-question-you-can-answer-with-data/player]
>
>

## <a name="other-videos-in-this-series"></a>Autres vidéos de cette série
*Science des données pour les débutants* offre une introduction rapide à la science des données en cinq petites vidéos.

* Vidéo 1 : [Les 5 questions auxquelles la science des données répond](data-science-for-beginners-the-5-questions-data-science-answers.md) *(5 min 14 s)*
* Vidéo 2 : [Vos données sont-elles prêtes pour la science des données ?](data-science-for-beginners-is-your-data-ready-for-data-science.md) *(4 min 56 sec)*
* Vidéo 3 : Poser une question à laquelle les données permettent de répondre
* Vidéo 4 : [Prédire une réponse à l’aide d’un modèle simple](data-science-for-beginners-predict-an-answer-with-a-simple-model.md) *(7 min 42 sec)*
* Vidéo 5 : [Copier le travail d’autres personnes pour des projets de science des données](data-science-for-beginners-copy-other-peoples-work-to-do-data-science.md) *(3 min 18 s)*

## <a name="transcript-ask-a-question-you-can-answer-with-data"></a>Transcription : Poser une question à laquelle les données permettent de répondre
Bienvenue dans la troisième vidéo de la série « Science des données pour les débutants ».  

Dans celle-ci, nous allons vous donner des conseils pour formuler une question à laquelle les données permettent de répondre.

Pour tirer le meilleur parti de cette vidéo, nous vous recommandons de regarder les deux vidéos précédentes de la série  : « Les 5 questions auxquelles la science des données répond » et « Vos données sont-elles prêtes pour la science des données ? »

## <a name="ask-a-sharp-question"></a>Poser une question précise
Comme nous l’avons vu, la science des données est le processus consistant à utiliser des noms (également appelés catégories ou étiquettes) et des nombres pour prédire la réponse à une question. Toutefois, la question ne peut pas avoir n’importe quelle forme. Il doit s’agir d’une *question précise.*

Une question vague n’appelle pas une réponse par un nom ou un nombre. Une question précise si.

Imaginons que vous avez trouvé une lampe magique renfermant un génie qui répond correctement à toutes les questions que vous lui posez. Mais ce génie est espiègle, et il fait en sorte de rendre sa réponse la plus vague et confuse possible. Vous devez le coincer avec une question si bien formulée qu’il sera obligé de vous dire ce que vous voulez savoir.

Si vous lui posez une question vague comme « Que va-t-il arriver à mon action ? », le génie pourra très bien répondre « Son cours va changer ». C’est une réponse correcte, mais pas vraiment utile.

Mais si vous lui posez une question précise comme « Quel sera le cours de mon action la semaine prochaine ? », le génie n’aura pas d’autre choix que de vous donner une réponse spécifique et de prédire un cours.

## <a name="examples-of-your-answer-target-data"></a>Exemples de réponse : données cibles
Une fois que vous avez formulé votre question, vérifiez si vous avez des exemples de la réponse dans vos données.

Si votre question est « Quel sera le cours de mon action la semaine prochaine ? », vous devez vous assurer que vos données incluent l’historique du cours de l’action.

Si votre question est « Quelle voiture de mon parc va tomber en panne en premier ? », vous devez vous assurer que vos données incluent des informations concernant les pannes précédentes.

![Données cibles : exemples de votre réponse. Formulez une question de science des données.](./media/data-science-for-beginners-ask-a-question-you-can-answer-with-data/target-data.png)

Ces exemples de réponses sont appelés des cibles. Une cible correspond à ce que nous essayons de prédire sur les points de données futurs, qu’il s’agisse d’une catégorie ou d’un nombre.

Si vous n’avez pas de données cibles, vous devez en collecter. Sans cela, vous ne pourrez pas obtenir de réponse à votre question.

## <a name="reformulate-your-question"></a>Reformulez votre question
Il est parfois possible de reformuler la question pour obtenir une réponse plus utile.

La question « Ce point de données est-il le point A ou B ? » prédit la catégorie (ou nom ou étiquette) d’un élément. Pour y répondre, nous utilisons un *algorithme de classification*.

La question « Combien ? » ou « Combien ? » prévoit une quantité. Pour y répondre, nous utilisons un *algorithme de régression*.

Pour voir comment nous pouvons transformer ces questions, examinons la question suivante : « Quel article est le plus intéressant pour ce lecteur ? ». Cette question appelle une prédiction d’un seul choix parmi de nombreuses possibilités, en d’autres termes, « S’agit-il de A, B, C ou D ? ». - avec un algorithme de classification.

Mais il serait peut-être plus facile de répondre à cette question si elle était reformulée ainsi : « Quel intérêt présente chacun des articles de cette liste pour ce lecteur ? ». Vous pouvez maintenant attribuer un score à chaque article, et il sera ensuite facile d’identifier l’article dont le score est le plus élevé. Il s’agit là d’une reformulation d’une question de classification en question de régression (ou « Combien ? »).

![Reformulez votre question. Question de classification et question de régression.](./media/data-science-for-beginners-ask-a-question-you-can-answer-with-data/classification-question-vs-regression-question.png)

La manière dont vous posez une question est une indication de l’algorithme qui peut vous donner une réponse.

Vous constaterez que certaines familles d’algorithmes, telles que celles de notre exemple sur les articles, sont étroitement liées. Vous pouvez reformuler votre question de manière à utiliser l’algorithme qui vous donnera la réponse la plus utile.

Mais surtout, vous devez poser une question précise, à laquelle les données permettent de répondre. Sans oublier de vous assurer que vous disposez des données appropriées pour obtenir une réponse.

Vous connaissez maintenant certains principes de base à observer pour poser une question à laquelle les données permettent de répondre.

Nous vous invitons à regarder les autres vidéos de la série « Science des données pour les débutants » de Microsoft Azure Machine Learning Studio (classique).

## <a name="next-steps"></a>Étapes suivantes
* [Menez une première expérience de science des données avec Machine Learning Studio (classique)](create-experiment.md)
* [Consultez la présentation de Machine Learning sur Microsoft Azure](../overview-what-is-azure-ml.md)