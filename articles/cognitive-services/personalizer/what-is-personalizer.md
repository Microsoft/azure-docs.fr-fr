---
title: Qu’est-ce que Personalizer ?
description: Personalizer est un service cloud qui vous permet de choisir la meilleure expérience à montrer à vos utilisateurs, en apprenant de leur comportement en temps réel.
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: overview
ms.date: 08/27/2020
ms.custom: cog-serv-seo-aug-2020
keywords: personalizer, Azure personalizer, machine learning
ms.openlocfilehash: 28aae130d062eaf57a66a9b90a6602c3874f1494
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97094146"
---
# <a name="what-is-personalizer"></a>Qu’est-ce que Personalizer ?

Azure Personalizer est un service cloud qui permet à vos applications de choisir l’élément de contenu le mieux adapté pour vos utilisateurs. Vous pouvez utiliser le service Personalizer pour déterminer le produit à suggérer aux acheteurs ou pour déterminer la position optimale pour une publicité. Une fois le contenu présenté à l’utilisateur, le système supervise le comportement de l’utilisateur en temps réel et signale un score de récompense au service Personalizer. Cela garantit l’amélioration continue du modèle Machine Learning, et la capacité de Personalizer à sélectionner le meilleur élément de contenu en fonction des informations contextuelles qu’il reçoit.

> [!TIP]
> Le contenu est toute unité d’information, comme du texte, des images, des URL, des e-mails, etc., ou tout autre élément que vous voulez sélectionner et présenter à vos utilisateurs.

Avant de commencer, n’hésitez pas à essayer [Personalizer avec cette démonstration interactive](https://personalizationdemo.azurewebsites.net/).

<!--
![What is personalizer animation](./media/what-is-personalizer.gif)
-->

## <a name="how-does-personalizer-select-the-best-content-item"></a>Comment Personalizer sélectionne l’élément de contenu le mieux adapté ?

Personalizer utilise l’**apprentissage par renforcement** pour sélectionner l’élément (_action_) le mieux adapté, en fonction du comportement collectif et des scores de récompense de tous les utilisateurs. Les actions sont des éléments de contenu, comme des articles de presse, des films spécifiques ou des produits.

L’appel de l’API **Rank** prend l’élément d’action, les caractéristiques de l’action et les caractéristiques de contexte pour sélectionner le premier élément d’action de la liste :

* **Actions avec caractéristiques** : éléments de contenu avec des caractéristiques précises pour chaque élément
* **Caractéristiques de contexte** : caractéristiques de vos utilisateurs, leur contexte ou leur environnement, lors de l’utilisation de votre application

L’appel de l’API Rank retourne l’ID de l’élément de contenu (ou __action__) à présenter à l’utilisateur dans le champ **Reward Action ID** (ID de l’action récompensée).

L’__action__ qui est présentée à l’utilisateur est choisie à l’aide de modèles Machine Learning, qui tentent d’optimiser la quantité totale de récompenses dans le temps.

### <a name="sample-scenarios"></a>Exemples de scénarios

Examinons quelques scénarios dans lesquels Personalizer peut être utilisé pour sélectionner le meilleur contenu à restituer à un utilisateur.

|Type de contenu|Actions (avec caractéristiques)|Caractéristiques contextuelles|ID de l’action récompensée retournée<br>(afficher ce contenu)|
|--|--|--|--|
|Liste des actualités|a. `The president...` (national, politique, [texte])<br>b. `Premier League ...` (mondial, sport, [texte, image, vidéo])<br> c. `Hurricane in the ...` (régional, météo, [texte,image]|Pays/Région de lecture des actualités<br>Mois ou saison<br>|a `The president...`|
|Liste de films|1. `Star Wars` (1977, [action, aventure, fantastique], George Lucas)<br>2. `Hoop Dreams` (1994, [documentaire, sport], Steve James<br>3. `Casablanca` (1942, [romantique, drame, guerre], Michael Curtiz)|Pays/Région du visionnage du film<br>Taille de l’écran<br>Type d’utilisateur<br>|3. `Casablanca`|
|Liste de produits|i. `Product A` (3 kg, $$$$, livraison en 24 heures)<br>ii. `Product B` (20 kg, $$, livraison en 2 semaines + frais de douanes)<br>iii. `Product C` (3 kg, $$$, livraison en 48 heures)|Pays/Région de consultation des produits<br>Niveau de dépense de l’utilisateur<br>Mois ou saison|ii. `Product B`|

Personalizer a utilisé l’apprentissage par renforcement pour sélectionner la meilleure action, appelée _ID d’action récompensée_. Le modèle Machine Learning utilise : 

* Un modèle entraîné : informations précédemment reçues du service de personnalisation utilisé pour améliorer le modèle Machine Learning
* Données actuelles : actions spécifiques avec des fonctionnalités et des caractéristiques de contexte

## <a name="when-to-use-personalizer"></a>Quand utiliser Personalizer

L’[API](https://go.microsoft.com/fwlink/?linkid=2092082) **Rank** de Personalizer est appelée chaque fois que votre application présente du contenu. Il s’agit d’un **événement**, qui est associé à un _ID d’événement_.

L’[API](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/Reward) **Reward** de Personalizer peut être appelée en temps réel ou retardée pour mieux répondre aux besoins de votre infrastructure. Vous déterminez le score de récompense en fonction des besoins de votre entreprise. Le score de récompense est compris entre 0 et 1. Il peut s’agir d’une valeur unique comme 1 pour Bon ou 0 pour Mauvais, ou d’un nombre produit par un algorithme que vous créez en tenant compte de vos objectifs et métriques métier.

## <a name="content-requirements"></a>Exigences de contenu

Utilisez Personalizer si votre contenu :

* Comprend un ensemble restreint d’éléments (environ 50 max.). Si vous disposez d’une liste plus longue, [utilisez un moteur de recommandations](where-can-you-use-personalizer.md#how-to-use-personalizer-with-a-recommendation-solution) pour réduire la liste à 50 éléments.
* Contient des informations décrivant le contenu que vous souhaitez classer : _actions avec caractéristiques_ et _caractéristiques de contexte_.
* A besoin d’un minimum de 1 000 événements de contenu par jour pour que Personalizer soit efficace. Si Personalizer ne reçoit pas le trafic minimal nécessaire, le service met plus de temps à déterminer l’élément de contenu le mieux adapté.

Étant donné que Personalizer utilise des informations collectives quasiment en temps réel pour retourner l’élément de contenu le mieux adapté, le service :
* Ne conserve pas et ne gère pas les informations de profil utilisateur
* Ne journalise pas les préférences ni l’historique de chaque utilisateur
* Ne nécessite pas le nettoyage ni l’étiquetage du contenu

## <a name="how-to-design-for-and-implement-personalizer"></a>Comment concevoir pour Personalizer et l’implémenter

1. [Concevez](concepts-features.md) et planifiez le contenu, les **_actions_** et le **_contexte_**. Déterminez l’algorithme de récompense pour le score de **_récompense_**.
1. Chaque [ressource Personalizer](how-to-settings.md) que vous créez est considérée comme une boucle d’apprentissage. La boucle recevra les appels des API Rank et Reward pour ce contenu ou cette expérience utilisateur.

    |Type de ressource| Objectif|
    |--|--|
    |[Mode apprenti](concept-apprentice-mode.md) `E0`|Effectuez l'apprentissage du modèle Personalizer sans affecter votre application existante, puis déployez le comportement d'apprentissage en ligne dans un environnement de production.|
    |Standard, `S0`|Comportement d’apprentissage en ligne dans un environnement de production|
    |Gratuit, `F0`| Essayer le comportement d’apprentissage en ligne dans un environnement hors production|

1. Ajoutez Personalizer à votre application, site web ou système :
    1. Ajoutez un appel **Rank** à Personalizer dans votre application, votre site web ou votre système afin de déterminer l’élément de _contenu_ le mieux adapté avant que celui-ci ne soit présenté à l’utilisateur.
    1. Présentez l’élément de _contenu_ le mieux adapté (qui correspond à l’_ID d’action récompensée_) à l’utilisateur.
    1. Appliquez une _logique métier_ aux informations collectées concernant le comportement de l’utilisateur, afin de déterminer le score de **récompense**, par exemple :

    |Comportement|Score de récompense calculé|
    |--|--|
    |L’utilisateur a sélectionné l’élément de _contenu_ le mieux adapté (ID d’action récompensée)|**1**|
    |L’utilisateur a sélectionné un autre contenu|**0**|
    |L’utilisateur s’est interrompu, a fait défiler la liste de manière indécise, puis a sélectionné l’élément de _contenu_ le mieux adapté (ID d’action récompensée)|**0,5**|

    1. Ajoutez un appel **Reward** en envoyant un score de récompense compris entre 0 et 1 :
        * Immédiatement après avoir affiché le contenu
        * Ou ultérieurement, sur un système hors connexion
    1. [Évaluez votre boucle](concepts-offline-evaluation.md) avec une évaluation hors connexion après une certaine période d’utilisation. L’évaluation hors connexion permet de tester et d’évaluer l’efficacité du service Personalizer sans changer votre code ni affecter l’expérience utilisateur.

## <a name="complete-a-quickstart"></a>Suivre un guide de démarrage rapide

Nous proposons des guides de démarrage rapide en C#, en JavaScript et en Python. Chaque guide de démarrage rapide est conçu pour vous montrer des modèles de conception de base et vous permettre d’exécuter du code en moins de 10 minutes. 

* [Démarrage rapide : Guide pratique pour la bibliothèque de client Personalizer](./quickstart-personalizer-sdk.md)

Une fois que vous avez pu vous familiariser avec le service Personalizer, essayez nos tutoriels et découvrez comment utiliser Personalizer dans des applications web, des chatbots ou un notebook Azure.

* [Tutoriel : Utiliser Personalizer dans une application web .NET](tutorial-use-personalizer-web-app.md)
* [Tutoriel : Utiliser Personalizer dans un chatbot .NET](tutorial-use-personalizer-chat-bot.md)
* [Tutoriel : Utiliser Personalizer dans un notebook Azure](tutorial-use-azure-notebook-generate-loop-data.md)

## <a name="reference"></a>Informations de référence 

* [SDK C#/.NET de Personalizer](/dotnet/api/overview/azure/cognitiveservices/client/personalizer?view=azure-dotnet)
* [SDK Go de Personalizer](https://github.com/Azure/azure-sdk-for-go/tree/master/services/preview)
* [SDK JavaScript de Personalizer](/javascript/api/@azure/cognitiveservices-personalizer/?view=azure-node-latest)
* [SDK Python de Personalizer](/python/api/overview/azure/cognitiveservices/personalizer?view=azure-python)
* [API REST](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api/operations/Rank)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Fonctionnement de Personalizer](how-personalizer-works.md)
> [Qu’est-ce que l’apprentissage par renforcement ?](concepts-reinforcement-learning.md)