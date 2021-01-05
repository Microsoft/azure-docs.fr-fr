---
title: Fichier include
description: Fichier include
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.custom: include file
ms.date: 04/16/2020
ms.openlocfilehash: 78b93c05621754ae499e4ae8ca6b66c5bbfaad1b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96028116"
---
Cognitive Services fournit deux services de traitement en langage naturel, [Language Understanding](../luis/what-is-luis.md) et [QnA Maker](../qnamaker/overview/overview.md), chacun avec un objectif différent. Découvrez quand utiliser chaque service et comment ils se complètent.

Le traitement en langage naturel (NLP) permet à votre application cliente, telle qu’un robot de conversation, de travailler avec vos utilisateurs, en utilisant un langage naturel. Un utilisateur entre une phrase ou une expression. Le texte de l’utilisateur peut avoir une grammaire, une orthographe et une ponctuation médiocres. Cognitive Service peut tout de même traiter la phrase de l’utilisateur, en retournant les informations dont le robot a besoin pour aider l’utilisateur.

## <a name="cognitive-services-with-nlp"></a>Cognitive Services avec NLP

Language Understanding (LUIS) et QnA Maker fournissent des NLP. L’application cliente envoie un texte en langage naturel. Le service prend le texte, le traite et retourne un résultat.

## <a name="when-to-use-each-service"></a>Quand utiliser chaque service

Language Understanding (LUIS) et QnA Maker s’occupent de problèmes différents. LUIS détermine l’intention du texte d’un utilisateur (appelé énoncé), tandis que QnA Maker détermine la réponse au texte d’un utilisateur (appelé requête).

Pour sélectionner le service approprié, vous devez comprendre le texte de l’utilisateur provenant de l’application cliente, ainsi que les informations que l’application cliente a besoin de recevoir de Cognitive Service.

Si votre robot de conversation reçoit le texte `How do I get to the Human Resources building on the Seattle North campus?`, utilisez le graphique ci-dessous pour comprendre le fonctionnement de chaque service avec le texte.

|Service|L’application cliente détermine|
|--|--|
|LUIS|**Détermine l’intention de l’utilisateur** du texte – le service ne retourne pas la réponse à la question. Par exemple, ce texte est classé comme correspondant à `FindLocation` l’intention.<br>|
|QnA Maker|**Retourne la réponse à la question** à partir d’une base de connaissances personnalisée. Par exemple, ce texte est déterminé comme une question avec la réponse texte statique de `Get on the #9 bus and get off at Franklin street`.|
|||

> [!div class="mx-imgBorder"]
> ![Infographie pour déterminer quand utiliser LUIS et quand utiliser QnA Maker](./luis-qna-maker-together-decision.png)

## <a name="when-do-you-use-luis"></a>Quand utiliser LUIS ?

Utilisez LUIS lorsque vous avez besoin de connaître l’intention de l’énoncé dans le cadre d’un processus dans le robot de conversation. Si vous continuez avec l’exemple de texte, `How do I get to the Human Resources building on the Seattle North campus?`, une fois que vous savez que l’utilisateur a l’intention de trouver un emplacement, vous pouvez transmettre des détails sur l’énoncé (extrait avec les entités) à un autre service, tel qu’un serveur de transport, pour obtenir la réponse.

Vous n’avez pas besoin de combiner LUIS et QnA Maker pour déterminer l’intention.

Vous pouvez combiner les deux services pour cet énoncé, si le robot de conversation doit traiter le texte en fonction des intentions et des entités (à l’aide de LUIS), ainsi que pour rechercher la réponse du texte statique spécifique (à l’aide de QnA Maker).

## <a name="when-do-you-use-qna-maker"></a>Quand utiliser QnA Maker ?

Utilisez QnA Maker lorsque vous avez une base de connaissances statique de réponses. Cette base de connaissances est personnalisée en fonction de vos besoins, que vous avez créés avec des documents tels que des PDF et des URL.

Poursuivez avec l’exemple d'énoncé, `How do I get to the Human Resources building on the Seattle North campus?`, envoyez le texte, en tant que requête, à votre service QnA Maker publié et recevez la meilleure réponse.

Vous n’avez pas besoin de combiner LUIS et QnA Maker pour déterminer la réponse à la question.

Vous pouvez combiner les deux services pour ce énoncé, si le robot de conversation doit traiter le texte en fonction des intentions et des entités (à l’aide de LUIS), ainsi que pour trouver la réponse (à l’aide de QnA Maker).

## <a name="use-both-services-when-your-knowledge-base-is-incomplete"></a>Utiliser les deux services lorsque votre base de connaissances est incomplète

Si vous créez votre base de connaissances QnA Maker, mais que vous savez que le domaine du sujet change (par exemple, des informations actualisées), vous pouvez combiner les services LUIS et QnA Maker. Cela vous permet d’utiliser les informations de votre base de connaissances, mais également d’utiliser LUIS pour déterminer l’intention d’un utilisateur. Une fois que l’application client a l’intention, elle peut demander des informations pertinentes à partir d’une autre source.

Votre application cliente doit surveiller à la fois les réponses LUIS et QnA Maker pour les scores. Si le score de QnA Maker est inférieur à un seuil arbitraire, utilisez l’intention et les informations d’entité retournées par LUIS pour transmettre les informations à un service tiers.

En poursuivant avec le texte d’exemple, `How do I get to the Human Resources building on the Seattle North campus?`, supposez que QnA Maker retourne un score de confiance faible. Utilisez l’intention retournée par LUIS, `FindLocation` et toutes les entités extraites, telles que `Human Resources building` et `Seattle North campus`, pour envoyer ces informations à un service de mappage ou de recherche pour une autre réponse.

Vous pouvez présenter cette réponse tierce à l’utilisateur pour la validation. Une fois que vous avez approuvé l’utilisateur, vous pouvez revenir à QnA Maker pour ajouter les informations qui vous permettent de développer vos connaissances.

## <a name="use-both-services-when-your-chat-bot-needs-more-information"></a>Utilisez les deux services lorsque votre robot de conversation a besoin d’informations supplémentaires

Si votre robot de conversation a besoin d’informations supplémentaires par rapport à l’un des services, pour poursuivre l’utilisation d’un arbre de décision, utilisez les deux services et traitez les deux réponses dans l’application cliente.

Utilisez l'outil **[CLI Dispatch](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch)** de l’infrastructure robot pour vous aider à créer un processus permettant de travailler avec les deux services. Cet outil crée une application d'intention LUIS de premier plan qui diffuse entre LUIS et QnA Maker en tant qu'applications enfants. [En savoir plus](/azure/bot-service/bot-builder-tutorial-dispatch?tabs=cs&view=azure-bot-service-4.0) sur l’intégration avec LUIS, QnA Maker et Bot Framework.

Utilisez l’exemple de générateur de bot, **NLP avec Dispatch**, dans [C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/14.nlp-with-dispatch) ou [Node.js](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/14.nlp-with-dispatch), pour implémenter ce type de robot de conversation.

## <a name="best-practices"></a>Meilleures pratiques

Implémentez les meilleures pratiques pour chaque service :

* Bonnes pratiques [LUIS](../luis/luis-concept-best-practices.md)
* Bonnes pratiques [QnA Maker](../qnamaker/concepts/best-practices.md)

## <a name="see-also"></a>Voir aussi

* [Language Understanding (LUIS)](../luis/what-is-luis.md)
* [QnA Maker](../qnamaker/overview/overview.md)
* [CLI Dispatch](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch)
* [Exemples d’infrastructure robot](https://github.com/Microsoft/BotBuilder-Samples)
* [Service Azure Bot](/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0)
* [Émulateur de bot Azure](https://github.com/Microsoft/BotFramework-Emulator)
* [Conversation Web d’infrastructure de robot](https://github.com/microsoft/BotFramework-WebChat)