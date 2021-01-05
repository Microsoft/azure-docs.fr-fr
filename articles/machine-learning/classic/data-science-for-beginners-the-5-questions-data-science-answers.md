---
title: 'ML Studio (classique) : La science des données pour les débutants - Azure'
description: La série Science des données pour les débutants enseigne des concepts de base en 5 courtes vidéos, et commence par Les 5 questions de la science des données. À partir d’Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: sdgilley
ms.author: sgilley
ms.custom: seodec18
ms.date: 03/22/2019
ms.openlocfilehash: ae4eca82df498f1aa8f9ef84691cecdf719d0d76
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93307477"
---
# <a name="data-science-for-beginners-video-1-the-5-questions-data-science-answers"></a>La science des données pour les débutants, vidéo 1 : Les cinq questions auxquelles répond la science des données

Découvrez la science des données dans la série de cinq courtes vidéos de présentation *Science des données pour les débutants* d’un célèbre spécialiste des données. Ces vidéos sont simples mais utiles, que vous souhaitiez travailler sur la science des données ou collaborer avec d’autres scientifiques.

Cette première vidéo concerne les types de questions auxquelles la science des données est capable de répondre. Pour tirer le meilleur parti de la série, regardez l’ensemble des vidéos. [Accéder à la liste des vidéos](#other-videos-in-this-series)
<br>

> [!VIDEO https://channel9.msdn.com/Shows/SupervisionNotRequired/8/player]
>
>

## <a name="other-videos-in-this-series"></a>Autres vidéos de cette série
*Science des données pour les débutants* offre une introduction rapide à la science des données en 25 minutes au total. Regardez les cinq vidéos :

* Vidéo 1 : Les cinq questions auxquelles répond la science des données
* Vidéo 2 : [Vos données sont-elles prêtes pour la science des données ?](data-science-for-beginners-is-your-data-ready-for-data-science.md) *(4 min 56 sec)*
* Vidéo 3 : [Poser une question à laquelle les données permettent de répondre](data-science-for-beginners-ask-a-question-you-can-answer-with-data.md) *(4 min 17 s)*
* Vidéo 4 : [Prédire une réponse à l’aide d’un modèle simple](data-science-for-beginners-predict-an-answer-with-a-simple-model.md) *(7 min 42 sec)*
* Vidéo 5 : [Copier le travail d’autres personnes pour des projets de science des données](data-science-for-beginners-copy-other-peoples-work-to-do-data-science.md) *(3 min 18 s)*

## <a name="transcript-the-5-questions-data-science-answers"></a>Transcription : Les cinq questions auxquelles répond la science des données
Bonjour ! Bienvenue dans la série de vidéos *Science des données pour les débutants*.

La science des données peut être intimidante. Mais je vais vous présenter ici les principes de base sans équations ni jargon de programmation informatique.

Dans cette première vidéo, nous allons parler des « 5 questions auxquelles la science des données répond ».

La science des données utilise des nombres et des noms (également connus sous les noms de catégories ou d’étiquettes) pour prédire des réponses à des questions.

Cela peut vous surprendre, mais *il existe cinq questions auxquelles la science des données répond* :

* Est-ce A ou B ?
* Est-ce étrange ?
* Quelle quantité (ou combien) ?
* Comment les données sont-elles organisées ?
* Que dois-je faire ensuite ?

Un groupe spécifique de méthodes d’apprentissage automatique, appelées algorithmes, permet de répondre à ces questions.

Le mieux est de considérer un algorithme comme une recette et vos données comme les ingrédients. Un algorithme indique comment combiner et associer les données afin d’obtenir une réponse. Les ordinateurs sont comme un mixeur. Ils prennent en charge une grande partie des tâches difficiles de l’algorithme, et ce assez rapidement.

## <a name="question-1-is-this-a-or-b-uses-classification-algorithms"></a>La question 1, Est-ce A ou B ?, utilise des algorithmes de classification
Commençons par la question : Est-ce A ou B ?

![Algorithmes de classification : Est-ce A ou B ?](./media/data-science-for-beginners-the-5-questions-data-science-answers/classification-algorithms.png)

Cette famille d’algorithmes est appelée la classification double classe.

Elle est utile pour toute question qui n’a que deux réponses possibles.

Par exemple :

* Une défaillance de ce pneu se produira-t-elle dans les mille prochains kilomètres ? Oui ou non ?
* Qu’est-ce qui attire le plus les clients : un coupon de 5 $ ou une remise de 25 % ?

Cette question peut également être reformulée de façon à comporter plus de deux options : est-ce A ou B ou C ou D, etc. ?  On parle de classification multiclasse. Elle est utile lorsque vous avez plusieurs (ou plusieurs milliers) de réponses possibles. La classification multiclasse choisit celle qui est la plus probable.

## <a name="question-2-is-this-weird-uses-anomaly-detection-algorithms"></a>La question 2, Est-ce bizarre ?, utilise des algorithmes de détection d’anomalie.
La deuxième question à laquelle la science des données peut répondre est : Est-ce étrange ? Pour répondre à cette question, on utilise une famille d’algorithmes appelée détection des anomalies.

![Algorithmes de détection d’anomalie : Est-ce étrange ?](./media/data-science-for-beginners-the-5-questions-data-science-answers/anomaly-detection-algorithms.png)

Si vous disposez d’une carte de crédit, vous avez déjà bénéficié de la détection des anomalies. Votre société de carte de crédit analyse vos modèles d’achat, afin de vous alerter en cas de fraude possible. Les frais considérés comme « étranges » peuvent être liés à un achat dans un magasin dans lequel vous ne faites normalement aucun achat ou à un achat anormalement coûteux.

Cette question peut se révéler utile dans de nombreux cas. Exemple :

* Si votre voiture dispose d’indicateurs de pression, vous vous posez peut-être les questions suivantes : La pression indiquée par l’indicateur est-elle normale ?
* Si votre suivi porte sur Internet, vous pouvez vous demander : ce message d’Internet est-il classique ?

La détection d’anomalies signale des événements ou comportements inattendus ou inhabituels. Elle donne des indications sur l’endroit où rechercher des problèmes.

## <a name="question-3-how-much-or-how-many-uses-regression-algorithms"></a>La question 3, Quelle quantité ? ou Combien ?, utilise des algorithmes de régression
L’apprentissage automatique peut également prédire une quantité ou un volume. La famille d’algorithmes qui répond à cette question est appelée régression.

![Algorithmes de régression : Quelle quantité ? ou Combien ?](./media/data-science-for-beginners-the-5-questions-data-science-answers/regression-algorithms.png)

Les algorithmes de régression effectuent des prédictions numériques, notamment :

* Quelle sera la température mardi prochain ?  
* Quelles seront mes ventes au quatrième trimestre ?

Ils peuvent répondre à toute question dont la réponse est un chiffre.

## <a name="question-4-how-is-this-organized-uses-clustering-algorithms"></a>La question 4, Comment les données sont-elles organisées ?, utilise des algorithmes de clustering
Les deux dernières questions sont plus avancées.

Parfois, vous souhaitez comprendre la structure d’un jeu de données : comment est-il organisé ? Pour cette question, vous ne disposez pas d’exemples dont vous connaissez déjà les résultats.

Il existe de nombreuses manières d’analyser la structure de données. L’une des approches est le clustering. Elle sépare les données en groupes naturels pour en simplifier l’interprétation. Avec le clustering, il n’existe pas une seule réponse.

![Algorithmes de clustering : Comment les données sont-elles organisées ?](./media/data-science-for-beginners-the-5-questions-data-science-answers/clustering-algorithms.png)

Les exemples suivants sont des questions de clustering courantes :

* Quels spectateurs aiment quels types de films ?
* Quels modèles défaillent de la même manière ?

En comprenant comment les données sont organisées, vous pouvez mieux comprendre (et prédire) des événements et des comportements.  

## <a name="question-5-what-should-i-do-now-uses-reinforcement-learning-algorithms"></a>La question 5, Que dois-je faire maintenant ?, utilise des algorithmes d’apprentissage par renforcement
La dernière question : que dois-je faire maintenant ? \- utilise une famille d’algorithmes appelée apprentissage de renforcement.

L’apprentissage par renforcement est inspiré de la manière dont le cerveau de rats et d’humains répond aux punitions et aux récompenses. Ces algorithmes apprennent sur la base de résultats pour décider de l’action suivante.

En règle générale, l’apprentissage par renforcement est une solution parfaitement adaptée à des systèmes automatisés qui doivent prendre un grand nombre de petites décisions sans instructions humaines.

![Algorithmes d’apprentissage par renforcement : Que dois-je faire ensuite ?](./media/data-science-for-beginners-the-5-questions-data-science-answers/reinforcement-learning-algorithms.png)

Les questions auxquelles il répond concernent toujours l’action à exécuter (généralement par une machine ou un robot). Voici quelques exemples :

* Pour le système de régulation de la température d’une maison : ajuster la température ou la laisser telle quelle ?  
* Pour une voiture autonome : au feu orange, freiner ou accélérer ?  
* Pour un robot aspirateur : continuer à aspirer ou revenir à la borne de recharge ?

Les algorithmes d’apprentissage par renforcement collectent les données au fur et à mesure et apprend de ses essais et de ses erreurs.

Et voilà ! Ce sont les 5 questions auxquelles la science des données peut répondre.

## <a name="next-steps"></a>Étapes suivantes
* [Menez une première expérience de science des données avec Machine Learning Studio (classique)](create-experiment.md)
* [Consultez la présentation de Machine Learning sur Microsoft Azure](../overview-what-is-azure-ml.md)