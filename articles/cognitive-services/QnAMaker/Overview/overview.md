---
title: Qu’est-ce que le service QnA Maker ?
description: QnA Maker est un service NLP cloud qui permet de créer facilement une couche conversationnelle naturelle sur vos données. Il peut être utilisé pour trouver la réponse la plus appropriée à une entrée donnée en langage naturel à partir de votre base de connaissances personnalisée (base d’informations).
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: overview
ms.date: 05/26/2020
ms.custom: cog-serv-seo-aug-2020
keywords: qna maker, chatbot avec peu de code, invites multitours
ms.openlocfilehash: 9655f64da455fb7a7124a97e6dc2d1d8437d05b7
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353134"
---
# <a name="what-is-qna-maker"></a>Qu’est-ce que QnA Maker ?

QnA Maker est un service cloud de traitement en langage naturel (NLP, Natural Language Processing) qui vous permet de créer une couche conversationnelle naturelle sur vos données. Il est utilisé pour trouver la réponse la plus appropriée à une entrée à partir de votre base de connaissances personnalisée (base d’informations).

QnA Maker est couramment utilisé pour créer des applications clientes conversationnelles, notamment des applications de réseaux sociaux, des chatbots et des applications de bureau à reconnaissance vocale.

## <a name="when-to-use-qna-maker"></a>Quand utiliser QnA Maker ?

* **Quand vous avez des informations statiques** : utilisez QnA Maker quand vous avez des informations statiques dans votre base de connaissances de réponses. Cette base de connaissances, créée avec des documents tels que [des PDF et des contenus accessibles par des URL](../index.yml), est personnalisée en fonction de vos besoins.
* **Quand vous souhaitez fournir la même réponse à une requête, une question ou une commande** : quand différents utilisateurs soumettent la même question, la même réponse est retournée.
* **Quand vous souhaitez filtrer des informations statiques en fonction de méta-informations** : ajoutez des balises de [métadonnées](../how-to/metadata-generateanswer-usage.md) pour fournir des options de filtrage supplémentaires relatives aux utilisateurs et informations de votre application cliente. Les [échanges](../how-to/chit-chat-knowledge-base.md), types ou formats de contenu, objets de contenu et actualisations de contenu représentent des informations de métadonnées courantes.
* **Quand vous souhaitez gérer une conversation de bot incluant des informations statiques** : votre base de connaissances répond à une commande ou au texte conversationnel d’un utilisateur. Si la réponse fait partie d’un flux de conversation prédéterminé, représenté dans votre base de connaissances avec un [contexte multitour](../how-to/multiturn-conversation.md), le bot peut facilement fournir ce flux.

## <a name="what-is-a-knowledge-base"></a>Qu’est-ce qu’une base de connaissances ?

QnA Maker [importe votre contenu](../index.yml) dans une base de connaissances comprenant des paires question/réponse. Le processus d’importation extrait des informations sur la relation entre les différentes parties de votre contenu structuré et semi-structuré pour définir des relations entre les paires question/réponse. Vous pouvez modifier ces paires question/réponse ou en ajouter de nouvelles.

Le contenu de la paire question/réponse comprend les éléments suivants :
* Toutes les autres formes de la question
* Les étiquettes de métadonnées utilisées pour filtrer les choix de réponse lors de la recherche
* Des invites de suivi pour poursuivre le perfectionnement de la recherche

![Exemple de question et de réponse avec des métadonnées](../media/qnamaker-overview-learnabout/example-question-and-answer-with-metadata.png)

Une fois que vous avez publié votre base de connaissances, une application cliente envoie la question d’un utilisateur au point de terminaison. Votre service QnA Maker traite la question et y répond avec la meilleure réponse.

## <a name="create-a-chat-bot-programmatically"></a>Créer un chatbot programmatiquement

Une fois qu’une base de connaissances QnA Maker est publiée, une application cliente envoie une question au point de terminaison de votre base de connaissances et reçoit les résultats sous forme de réponse JSON. Un chatbot est un exemple d’application cliente courante pour QnA Maker.

![Poser une question à un bot et obtenir une réponse à partir du contenu de la base de connaissances](../media/qnamaker-overview-learnabout/bot-chat-with-qnamaker.png)

|Étape|Action|
|:--|:--|
|1|L’application cliente envoie la _question_ de l’utilisateur (texte dans ses propres mots) « How do I programmatically update my Knowledge Base? » au point de terminaison de votre base de connaissances.|
|2|QnA Maker utilise la base de connaissances entraînée pour fournir la réponse correcte et les invites de suivi qui peuvent être utilisées pour affiner la recherche de la meilleure réponse. QnA Maker retourne une réponse au format JSON.|
|3|L’application cliente utilise la réponse JSON pour prendre des décisions concernant la manière de poursuivre la conversation. Ces décisions peuvent inclure l’affichage de la réponse principale et la présentation de choix supplémentaires pour affiner la recherche de la meilleure réponse. |
|||

## <a name="build-low-code-chat-bots"></a>Créer des chatbots avec peu de code

Le portail QnA Maker offre tout l’environnement nécessaire pour créer une base de connaissances. Vous pouvez importer des documents sous leur forme actuelle dans votre base de connaissances. Ces documents (p.ex., FAQ, manuel de produit, feuille de calcul ou page web) sont convertis en paires question/réponse. Chaque paire est analysée pour identifier des invites de suivi et est connectée à d’autres paires. Le format _Markdown_ final prend en charge les présentations riches, notamment les images et les liens.

Une fois votre base de connaissances modifiée, publiez-la sur un [bot Azure Web App](https://azure.microsoft.com/services/bot-service/) de travail sans écrire le moindre code. Testez votre bot dans le [portail Azure](https://portal.azure.com) ou téléchargez-le et poursuivez le développement.

## <a name="high-quality-responses-with-layered-ranking"></a>Réponses de haute qualité avec classement par couches

Le système de QnA Maker s’appuie sur une approche de classement par couches. Les données sont stockées dans la Recherche Azure, qui sert également de première couche de classement. Les meilleurs résultats de la Recherche Azure sont ensuite transmis par le biais du modèle de reclassement NLP de QnA Maker pour produire les résultats finaux et le score de confiance.

## <a name="multi-turn-conversations"></a>Conversations multitours

QnA Maker propose des invites multitours et un apprentissage actif pour vous aider à améliorer vos paires question/réponse de base.

Les **invites multitours** vous donnent la possibilité d’associer les paires de questions et réponses. Cette association permet à l’application cliente de fournir une réponse principale et fournit davantage de questions pour affiner la recherche d’une réponse finale.

Dès lors que la base de connaissances reçoit des questions des utilisateurs au point de terminaison publié, QnA Maker applique l’**apprentissage actif** à ces questions réelles pour suggérer des modifications à apporter à la base de connaissances afin d’en améliorer la qualité.

## <a name="development-lifecycle"></a>Cycle de vie de développement

QnA Maker offre des fonctionnalités de création, d’entraînement et de publication ainsi que des autorisations de collaboration, s’intégrant à l’ensemble du cycle de vie de développement.

> [!div class="mx-imgBorder"]
> ![Image conceptuelle du cycle de développement](../media/qnamaker-overview-learnabout/development-cycle.png)


## <a name="complete-a-quickstart"></a>Suivre un guide de démarrage rapide

Nous proposons des guides de démarrage rapide pour la plupart des langages de programmation. Chaque guide est conçu pour vous montrer des modèles de conception de base et vous permettre d’exécuter du code en moins de 10 minutes. Consultez la liste suivante pour connaître le guide de démarrage rapide de chaque fonctionnalité.

* [Démarrer avec la bibliothèque de client QnA Maker](../quickstarts/quickstart-sdk.md)
* [Démarrer avec le portail QnA Maker](../quickstarts/create-publish-knowledge-base.md)
* [Démarrer avec les API REST QnA Maker](../quickstarts/quickstart-rest-curl.md)


## <a name="next-steps"></a>Étapes suivantes
QnA Maker fournit tout ce dont vous avez besoin pour créer, gérer et déployer une base de connaissances personnalisée.

> [!div class="nextstepaction"]
> [Passer en revue les derniers changements](../whats-new.md)