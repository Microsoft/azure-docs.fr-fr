---
title: Où et comment utiliser Personalizer
description: Personalizer peut être appliqué dans tous les cas où votre application peut sélectionner l’élément, l’action ou le produit corrects à montrer, de façon à améliorer l’expérience, obtenir de meilleurs résultats métier ou améliorer la productivité.
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 02/18/2020
ms.openlocfilehash: 87179950c33a6facb59ce499aa0ae393e53fb37f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91777222"
---
# <a name="where-and-how-to-use-personalizer"></a>Où et comment utiliser Personalizer

Utilisez Personalizer dans toutes les situations où votre application doit sélectionner l’action (contenu) appropriée à montrer, de façon à améliorer l’expérience, obtenir de meilleurs résultats métier ou améliorer la productivité.

Personalizer utilise l’apprentissage par renforcement pour sélectionner l’action (contenu) à montrer à l’utilisateur. La sélection peut varier considérablement selon la quantité, la qualité et la distribution des données envoyées au service.

## <a name="example-use-cases-for-personalizer"></a>Exemple de cas d’utilisation pour Personalizer

* **Clarification et désambiguïsation de l’intention** : aidez vos utilisateurs à vivre une meilleure expérience lorsque leur intention n’est pas claire en fournissant une option personnalisée.
* **Suggestions par défaut** pour les menus et les options : demandez au bot de suggérer l’élément le plus probable de manière personnalisée, avant de présenter un menu impersonnel ou une liste d’alternatives.
* **Caractéristiques et ton du bot** : pour les bots pouvant varier le ton, la verbosité et le style d’écriture, envisagez de diversifier ces caractéristiques.
* **Contenu des notifications et alertes** : choisissez le texte à utiliser pour les alertes afin d’impliquer davantage les utilisateurs.
* **Minutage des notifications et alertes** : personnalisez l’apprentissage du moment opportun pour envoyer des notifications aux utilisateurs afin de les impliquer davantage.


## <a name="expectations-required-to-use-personalizer"></a>Attentes requises pour utiliser Personalizer

Vous pouvez appliquer Personalizer dans les situations où vous respectez ou pouvez implémenter les instructions suivantes.

|Instruction|Explication|
|--|--|
|Objectif métier|Vous avez un objectif métier ou d’utilisabilité pour votre application.|
|Contenu|Il y a un endroit dans votre application où une décision contextuelle de ce qu’il faut montrer aux utilisateurs va améliorer cet objectif.|
|Quantité du contenu|Vous avez moins de 50 actions à classer par appel.|
|Agréger les données|Le meilleur choix peut et doit être appris du comportement collectif des utilisateurs et du score de récompense total.|
|Utilisation éthique|L’utilisation du machine learning pour la personnalisation suit des [guides pour une utilisation responsable](ethics-responsible-use.md) et les choix que vous faites.
|Meilleure option unique|La décision contextuelle peut être exprimée sous la forme d’un classement de la meilleure option (action) à partir d’un ensemble limité de choix.|
|Score de résultat|Le degré de pertinence du choix classé pour votre application peut être déterminé en mesurant certains aspects du comportement de l’utilisateur et en l’exprimant dans un _[score de récompense](concept-rewards.md)_ .|
|Moment pertinent|Le score de récompense n’introduit pas trop de facteurs perturbants ou externes. La durée de l’expérience est suffisamment courte pour que le score de récompense puisse être calculé pendant la période de temps où il reste pertinent.|
|Caractéristiques contextuelles suffisantes|Vous pouvez exprimer le contexte pour le classement sous la forme d’une liste d’au moins cinq [caractéristiques](concepts-features.md) dont vous pensez qu’elles peuvent aider à faire le bon choix ; ceci n’inclut pas d’informations d’identification spécifiques à l’utilisateur.|
|Caractéristiques d’action suffisantes|Vous disposez d’informations sur chaque choix de contenu, _action_, sous la forme d’une liste d’au moins 5 [caractéristiques](concepts-features.md) dont vous pensez qu’elles peuvent aider Personalizer à faire le bon choix.|
|Données quotidiennes|Il existe suffisamment d’événements pour continuer à obtenir une personnalisation optimale si le problème se déplace au fil du temps (comme des préférences quant à des actualités ou à la mode). Personalizer s’adaptera aux changements continus du monde réel, mais les résultats ne seront pas optimaux s’il n’y pas suffisamment de données et d’événements servant de base à l’apprentissage pour découvrir et décider de nouveaux modèles. Vous devez choisir un cas d’utilisation qui se produit suffisamment souvent. Recherchez des cas d’utilisation qui se produisent au moins 500 fois par jour.|
|Données historiques|Votre application peut conserver des données sur une période suffisamment longue pour accumuler un historique d’au moins 100 000 interactions. Cela permet à Personalizer de collecter suffisamment de données pour effectuer des évaluations hors connexion et optimiser la stratégie.|

**N’utilisez pas Personalizer** lorsque le comportement personnalisé ne peut pas être découvert pour tous les utilisateurs. Par exemple, utiliser Personalizer pour suggérer une commande de pizza à partir d’une liste de 20 options de menu possibles est utile, mais le contact à appeler à partir de la liste de contacts de l’utilisateur quand celui-ci a besoin d’aide pour garder les enfants (comme « Grand-mère ») n’est pas un élément personnalisable dans votre base d’utilisateurs.

## <a name="how-to-use-personalizer-in-a-web-application"></a>Comment utiliser Personalizer dans une application web

L’ajout d’une boucle d’apprentissage à une application web inclut les étapes suivantes :

* Déterminer l’expérience à personnaliser, les actions et fonctionnalités disponibles, les fonctionnalités à utiliser selon le contexte et la récompense à donner.
* Ajouter une référence au Kit de développement logiciel (SDK) Personnalisation dans votre application.
* Appeler l’API de classement lorsque vous êtes prêt à personnaliser.
* Stocker l’eventId. Vous enverrez une récompense avec l’API de récompense ultérieurement.
1. Appelez Activer pour l’événement une fois que vous êtes certain que l’utilisateur a vu votre page personnalisée.
1. Attendez que l’utilisateur sélectionne un contenu classé.
1. Appelez l’API de récompense pour spécifier la qualité de la sortie de l’API de classement.

## <a name="how-to-use-personalizer-with-a-chat-bot"></a>Comment utiliser Personalizer avec un bot conversationnel

Cet exemple montre comment utiliser la fonctionnalité Personnalisation pour faire une suggestion par défaut au lieu d’envoyer à l’utilisateur une série de menus ou de choix à chaque fois.

* Téléchargez le [code](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/tree/master/samples/ChatbotExample) pour cet exemple.
* Configurez votre solution de bot. Veillez à publier votre application LUIS.
* Gérez les appels des API de classement et de récompense pour le bot.
    * Ajoutez du code pour gérer le traitement des intentions LUIS. Si la valeur **Aucune** est renvoyée en tant que première intention ou si le score de la première intention est inférieur au seuil de votre logique métier, envoyez la liste des intentions à Personalizer pour classer les intentions.
    * Affichez la liste des intentions à l’utilisateur sous la forme de liens sélectionnables, avec la première intention en haut du classement dans la réponse de l’API de classement.
    * Capturez la sélection de l’utilisateur et envoyez-la dans l’appel de l’API de récompense.

### <a name="recommended-bot-patterns"></a>Modèles bot recommandés

* Faites des appels de l’API de classement de Personalizer chaque fois qu’il est nécessaire de lever une ambiguïté, par opposition à la mise en cache des résultats pour chaque utilisateur. Le résultat de la désambiguïsation de l’intention peut changer au fil du temps pour une personne, et le fait de permettre à l’API de classement d’explorer les changements accélérera l’apprentissage global.
* Choisissez une interaction commune à de nombreux utilisateurs afin de disposer de suffisamment de données à personnaliser. Par exemple, des questions préliminaires peuvent être plus adaptées que des clarifications demandées à un stade de la conversation que de rares utilisateurs atteignent.
* Utiliser le classement par ordre de priorité des appels d’API pour activer des conversations de la « première suggestion est bonne », où l’utilisateur se fait demander : « vous aimeriez X ? » ou « vouliez vous dire X ? » et l’utilisateur peut simplement confirmer ; par opposition aux options de l’utilisateur à partir d’un menu. Par exemple, si l’utilisateur dit « Je souhaiterais un café », le bot peut lui demander « Voulez-vous un double espresso ? ». De cette façon, le signal de récompense est également fort parce qu’il se rapporte directement à la suggestion.

## <a name="how-to-use-personalizer-with-a-recommendation-solution"></a>Comment utiliser Personalizer avec une solution de recommandation

De nombreuses entreprises utilisent des moteurs de recommandation, des outils de marketing et de gestion des campagnes, la segmentation et le clustering de l’audience, le filtrage collaboratif et d’autres moyens pour recommander aux clients les produits d’un catalogue étendu.

Le [dépôt GitHub des générateurs de recommandations de Microsoft](https://github.com/Microsoft/Recommenders) fournit des exemples et des bonnes pratiques pour générer des systèmes de recommandation, fournis sous la forme de notebooks Jupyter. Il fournit des exemples fonctionnels pour la préparation des données, la création de modèles, et l’évaluation, l’optimisation et l’exploitation des moteurs de recommandation, pour de nombreuses approches courantes, notamment xDeepFM, SAR, ALS, RBM et DKN.

Personalizer peut fonctionner avec un moteur de recommandation quand il est présent.

* Les moteurs de recommandation prennent de grandes quantités d’éléments (par exemple 500 000) et recommandent un sous-ensemble (par exemple les 20 premiers) à partir de centaines ou de milliers d’options.
* Personalizer prend un petit nombre d’actions avec un grand nombre d’informations sur celles-ci et les classe en temps réel pour un contexte donné avec de nombreuses caractéristiques, tandis que la plupart des moteurs de recommandation utilisent seulement quelques attributs sur les utilisateurs, les produits et leurs interactions.
* Personalizer est conçu pour explorer de façon autonome et en permanence les préférences des utilisateurs, ce qui produit de meilleurs résultats là où le contenu change rapidement, comme les actualités, les événements en direct, le contenu dynamique d’une communauté, le contenu avec des mises à jour quotidiennes ou le contenu sujet à des variations saisonnières.

Une utilisation courante consiste à prendre le résultat d’un moteur de recommandation (par exemple les 20 premiers produits pour un client spécifique) et à l’utiliser comme actions en entrée pour Personalizer.

## <a name="adding-content-safeguards-to-your-application"></a>Ajout de protections de contenu à votre application

Si votre application autorise des variations importantes du contenu affiché aux utilisateurs, et qu’une partie de ce contenu peut être préjudiciable ou inappropriée pour certains utilisateurs, vous devez vous assurer à l’avance que les protections adéquates sont en place pour empêcher les utilisateurs de voir du contenu inapproprié. la suivante :
    * Obtenir la liste des actions à classer.
    * Filtrer les celles qui ne sont pas viables pour le public.
    * Classer uniquement les actions viables.
    * Afficher à l’utilisateur l’action en tête du classement.

Dans certaines architectures, la séquence ci-dessus peut être difficile à implémenter. Dans ce cas, il existe une autre approche pour l’implémentation des protections après le classement, mais il convient de prendre des dispositions pour que les actions qui ne relèvent pas de la protection ne soient pas utilisées pour effectuer l’apprentissage du modèle Personalizer.

* Répertorier les actions à classer avec l’apprentissage désactivé.
* Classer les actions.
* Vérifier si l’action en tête du classement est viable.
    * Si l’action en tête du classement est viable, activer l’apprentissage pour ce classement, puis l’afficher à l’utilisateur.
    * Si l’action en tête du classement n’est pas viable, ne pas activer l’apprentissage pour ce classement, et décider selon votre propre logique ou sur la base d’une autre approche ce qu’il convient de montrer à l’utilisateur. Même si vous utilisez la deuxième option la mieux classée, n’activez pas l’apprentissage pour ce classement.


## <a name="next-steps"></a>Étapes suivantes

[Déontologie et utilisation responsable](ethics-responsible-use.md).
