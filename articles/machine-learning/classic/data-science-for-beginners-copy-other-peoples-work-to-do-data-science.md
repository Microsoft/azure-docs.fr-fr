---
title: Copier des exemples de science des données – ML Studio (classique) – Azure
description: 'Une pierre d’angle de la science des données : amener d’autres personnes à faire votre travail. Obtenez des exemples d’apprentissage automatique à partir d’Azure AI Gallery.'
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: sdgilley
ms.author: sgilley
ms.custom: seodec18
ms.date: 03/22/2019
ms.openlocfilehash: 84caf1ea684e46eec00f8479bd0e7c4123e731b1
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322802"
---
# <a name="copy-other-peoples-work-to-do-data-science"></a>Copier le travail d’autres personnes pour des projets de science des données
## <a name="video-5-data-science-for-beginners-series"></a>Vidéo 5 : série Science des données pour les débutants
L’une des pierres d’angle de la science des données est d’amener d’autres personnes à faire votre travail. Par exemple, trouver un exemple d’algorithme de clustering dans Azure AI Gallery à utiliser pour votre propre expérience d’apprentissage automatique.

> [!IMPORTANT]
> **Cortana Intelligence Gallery** a été renommée **Azure AI Gallery**. Par conséquent, le texte et les images de cette transcription diffèrent légèrement de la vidéo, qui utilise l’ancien nom.
>

Pour tirer le meilleur parti de la série, regardez l’ensemble des vidéos. [Accéder à la liste des vidéos](#other-videos-in-this-series)
<br>

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/data-science-for-beginners-series-copy-other-peoples-work-to-do-data-science/player]
>
>

## <a name="other-videos-in-this-series"></a>Autres vidéos de cette série
*Science des données pour les débutants* offre une introduction rapide à la science des données en cinq petites vidéos.

* Vidéo 1 : [Les 5 questions auxquelles la science des données répond](data-science-for-beginners-the-5-questions-data-science-answers.md) *(5 min 14 s)*
* Vidéo 2 : [Vos données sont-elles prêtes pour la science des données ?](data-science-for-beginners-is-your-data-ready-for-data-science.md) *(4 min 56 sec)*
* Vidéo 3 : [Poser une question à laquelle les données permettent de répondre](data-science-for-beginners-ask-a-question-you-can-answer-with-data.md) *(4 min 17 s)*
* Vidéo 4 : [Prédire une réponse à l’aide d’un modèle simple](data-science-for-beginners-predict-an-answer-with-a-simple-model.md) *(7 min 42 sec)*
* Vidéo 5 : Copier le travail d’autres personnes pour des projets de science des données

## <a name="transcript-copy-other-peoples-work-to-do-data-science"></a>Transcription : Copier le travail d’autres personnes pour des projets de science des données
Bienvenue dans la cinquième vidéo de la série « Science des données pour les débutants ».

Dans celle-ci, vous découvrirez des exemples que vous pourrez emprunter comme point de départ pour votre propre travail. Pour tirer le meilleur parti de cette vidéo, nous vous recommandons de regarder les vidéos précédentes.

L’une des pierres d’angle de la science des données est d’amener d’autres personnes à faire votre travail.

## <a name="find-examples-in-the-azure-ai-gallery"></a>Rechercher des exemples dans Azure AI Gallery

Microsoft propose un service cloud appelé [Azure Machine Learning Studio (classique)](https://azure.microsoft.com/services/machine-learning-studio/). Il fournit un espace de travail dans lequel essayer différents algorithmes d’apprentissage automatique et, lorsque votre solution est en place, vous permet de la lancer en tant que service web.

Une partie de ce service s’appelle **[Azure AI Gallery](https://gallery.azure.ai/)** . Elle contient des ressources, notamment une collection d’expériences Azure Machine Learning Studio (classique) ou encore des modèles créés et fournis par des utilisateurs. Ces expériences sont un excellent moyen d’exploiter la réflexion et les efforts d’autres personnes pour vous aider dans vos propres solutions. N’hésitez pas à aller la découvrir.

![Galerie Azure AI](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/azure-ai-gallery.png)

Si vous cliquez sur **Expériences** en haut de la page, vous découvrirez les expériences les plus récentes et les plus populaires de la galerie. Vous pouvez parcourir le reste des expériences en cliquant sur **Parcourir tout** en haut de l’écran, puis entrer des termes de recherche et sélectionner des filtres de recherche.

## <a name="find-and-use-a-clustering-algorithm-example"></a>Rechercher et utiliser un exemple d’algorithme de clustering
Par exemple, si vous souhaitez voir un exemple de fonctionnement du clustering, recherchez des expériences de **« clustering sweep »** .

![Rechercher des expériences de clustering](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/search-for-clustering-experiments.png)

Voici une expérience intéressante qu’un utilisateur a fourni à la galerie.

![Expérience de clustering](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/clustering-experiment.png)

Cliquez sur cette expérience pour accéder à une page web décrivant le travail effectué par ce collaborateur, ainsi que certains de ses résultats.

![Page de description d’expérience de clustering](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/clustering-experiment-description-page.png)

Notez le lien **Ouvrir dans Studio (classique)** .

![Bouton Ouvrir dans Studio (classique)](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/open-in-studio.png)

Je peux cliquer dessus pour accéder directement à **Azure Machine Learning Studio (classique)** . Cette opération crée une copie de l’expérience et la place dans mon espace de travail. Cela inclut un jeu de données du contributeur, tout le traitement effectué, tous les algorithmes utilisés et la méthode d’enregistrement des résultats.

![Ouvrir une expérience de galerie dans Machine Learning Studio (classique) – Exemple d’algorithme de clustering](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/cluster-experiment-open-in-studio.png)

J’ai maintenant un point de départ. Je peux remplacer leurs données par les miennes et peaufiner le modèle. Cela me donne une longueur d’avance et me permet de tirer parti du travail déjà effectué par d’autres utilisateurs expérimentés.

## <a name="find-experiments-that-demonstrate-machine-learning-techniques"></a>Découvrir des expériences qui illustrent les techniques d’apprentissage automatique
Il existe d’autres expériences dans [Azure AI Gallery](https://gallery.azure.ai) fournies spécifiquement pour donner des exemples de procédures aux personnes novices en matière de science des données. Par exemple, il existe une expérience dans la galerie qui montre comment gérer les valeurs manquantes ([Méthodes de gestion des valeurs manquantes](https://gallery.azure.ai/Experiment/Methods-for-handling-missing-values-1)). Elle vous guide dans les 15 façons de remplacer les valeurs vides et présente les avantages de chaque méthode et quand les utiliser.

![Ouvrir des expériences de galerie dans Machine Learning Studio (classique) – Méthodes pour les valeurs manquantes](./media/data-science-for-beginners-copy-other-peoples-work-to-do-data-science/experiment-methods-for-handling-missing-values.png)

[Azure AI Gallery](https://gallery.azure.ai) est l’endroit où trouver des expériences de travail que vous pouvez utiliser comme point de départ pour vos propres solutions.

Nous vous invitons à regarder les autres vidéos de la série « Science des données pour les débutants » de Microsoft Azure Machine Learning Studio (classique).

## <a name="next-steps"></a>Étapes suivantes
* [Menez votre première expérience de science des données avec Azure Machine Learning Studio (classique)](create-experiment.md)
* [Consultez la présentation de Machine Learning sur Microsoft Azure](../overview-what-is-azure-ml.md)