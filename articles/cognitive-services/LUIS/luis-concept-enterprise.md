---
title: Concepts d’entreprise – LUIS
titleSuffix: Azure Cognitive Services
description: Découvrez les principes de conception de grandes applications LUIS ou de plusieurs applications incluant à la fois LUIS et QnA Maker.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 07/29/2019
ms.openlocfilehash: 2e2165b81c7cd634fe79ec4438a550ad365f5a30
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95019175"
---
# <a name="enterprise-strategies-for-a-luis-app"></a>Stratégies d’entreprise pour une application LUIS
Révisez ces stratégies de conception pour votre application d’entreprise.

## <a name="when-you-expect-luis-requests-beyond-the-quota"></a>Si vous attendez des requêtes LUIS au-delà du quota

LUIS a un quota mensuel et un quota par seconde, en fonction du niveau tarifaire de la ressource Azure. 

Si le taux de demandes de votre application LUIS dépasse le [quota](https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/) autorisé, vous pouvez :

* Répartir la charge vers d’autres applications LUIS avec la [même définition d’application](#use-multiple-apps-with-same-app-definition). Cela inclut éventuellement l’exécution de l’application LUIS à partir d’un [conteneur](luis-container-howto.md). 
* Créez et [attribuez plusieurs clés](#assign-multiple-luis-keys-to-same-app) à l’application. 

### <a name="use-multiple-apps-with-same-app-definition"></a>Utiliser plusieurs applications avec la même définition d’application
Exportez l’application LUIS originale, puis réimportez-la dans des applications distinctes. Chaque application a son propre ID d’application. Lorsque vous publiez, au lieu d’utiliser la même clé pour toutes les applications, créez une clé distincte pour chaque application. Équilibrez la charge sur toutes les applications afin qu’aucune application unique ne soit surchargée. Ajoutez [Application Insights](./luis-csharp-tutorial-bf-v4.md) pour surveiller l’utilisation. 

Pour obtenir la même intention principale dans toutes les applications, vérifiez que la prédiction d’intention entre la première et la deuxième intention est assez variée pour ne pas provoquer de confusion chez LUIS en donnant des résultats différents d’une application à l’autre en raison de variations mineures des énoncés. 

Lors de la formation de ces applications sœurs, veillez à [effectuer l’apprentissage avec toutes les données](luis-how-to-train.md#train-with-all-data).

Désigner une seule application en tant que le maître. Tous les énoncés dont la révision est suggérée doivent être ajoutés à l’application maître, puis déplacés vers toutes les autres applications. Il s’agit d’une exportation complète de l’application, ou du chargement des énoncés étiquetés du maître vers les enfants. Le chargement peut être effectué à partir du site web [LUIS](luis-reference-regions.md) ou de l’API de création pour un [seul énoncé](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c08) ou pour un [lot](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09). 

Pour un apprentissage actif, planifiez une [révision d’énoncés de point de terminaison](luis-how-to-review-endpoint-utterances.md) à intervalles réguliers, par exemple toutes les deux semaines, puis recommencez l’apprentissage et republiez. 

### <a name="assign-multiple-luis-keys-to-same-app"></a>Affecter plusieurs clés LUIS à la même application
Si votre application LUIS reçoit plus d’accès au point de terminaison que les quotas de votre clé unique ne le permettent, créez et affectez plusieurs clés à l’application LUIS. Créez un gestionnaire de trafic ou un équilibreur de charge afin de gérer les requêtes de point de terminaison pour toutes les clés de point de terminaison. 

## <a name="when-your-monolithic-app-returns-wrong-intent"></a>Lorsque votre application monolithique retourne une intention incorrecte
Si votre application est destinée à prédire une grande variété d’énoncés de l’utilisateur, envisagez d’implémenter le [modèle de répartition](#dispatch-tool-and-model). La rupture d’une application monolithique permet à LUIS de concentrer avec succès la détection sur les intentions au lieu d’être perturbée par les intentions des applications parent et enfants. 

Pour un apprentissage actif, planifiez une [révision d’énoncés de point de terminaison](luis-how-to-review-endpoint-utterances.md) à intervalles réguliers, par exemple toutes les deux semaines, puis entraînez et publiez à nouveau. 

## <a name="when-you-need-to-have-more-than-500-intents"></a>Lorsque vous avez besoin de plus de 500 intentions
Par exemple, supposons que vous développez un compagnon Office qui a plus de 500 intentions. Si 200 intentions sont liées à la planification des réunions, 200 sont des rappels, 200 concernent l’obtention d’informations sur les collègues et 200 sont destinées à l’envoi de courrier électronique, groupez les intentions afin que chaque groupe soit dans une même application, puis créez une application de niveau supérieur qui contient chaque intention. Utilisez le [modèle de répartition](#dispatch-tool-and-model) pour créer l’application de niveau supérieur. Modifiez ensuite votre bot pour utiliser l’appel en cascade comme indiqué dans le [didacticiel du modèle de répartition](/azure/bot-service/bot-builder-tutorial-dispatch?branch=master&tabs=cs&view=azure-bot-service-4.0). 

## <a name="when-you-need-to-combine-several-luis-and-qna-maker-apps"></a>Lorsque vous souhaitez combiner plusieurs applications LUIS et QnA Maker
Si vous avez plusieurs applications LUIS et QnA Maker qui doivent répondre à un bot, utilisez le [modèle de répartition](#dispatch-tool-and-model) pour générer l’application de niveau supérieur.  Modifiez ensuite votre bot pour utiliser l’appel en cascade comme indiqué dans le [didacticiel du modèle de répartition](/azure/bot-service/bot-builder-tutorial-dispatch?branch=master&tabs=cs&view=azure-bot-service-4.0). 

## <a name="dispatch-tool-and-model"></a>Outil et modèle de répartition
Utilisez l’outil en ligne de commande [Répartir][dispatch-tool] disponible dans [BotBuilder-tools](https://github.com/Microsoft/botbuilder-tools) pour combiner plusieurs applications LUIS et/ou QnA Maker dans une application LUIS parente. Cette approche vous permet d’avoir un domaine parent incluant tous les sujets et différents domaines de sujets enfants dans des applications distinctes. 

![Image conceptuelle de l’architecture de répartition](./media/luis-concept-enterprise/dispatch-architecture.png)

Le domaine parent est noté dans LUIS avec une version nommée `Dispatch` dans la liste des applications. 

Le chatbot reçoit l’énoncé, puis l’envoie à l’application LUIS parente pour la prédiction. L’intention prédite principale provenant de l’application parente détermine quelle application LUIS enfant est appelée ensuite. Le chatbot envoie l’énoncé à l’application enfant pour une prédiction plus précise.

Comprendre comment cette hiérarchie d’appels est établie à partir de Bot Builder v4 [dispatcher-application-tutorial](/azure/bot-service/bot-builder-tutorial-dispatch?branch=master&tabs=cs&view=azure-bot-service-4.0).  

### <a name="intent-limits-in-dispatch-model"></a>Limites des intentions dans le modèle de répartition
Une application de répartition a 500 sources de répartition maximum, ce qui équivaut à 500 intentions. 

## <a name="more-information"></a>Informations complémentaires

* [Kit de développement logiciel (SDK) Bot Framework](https://github.com/Microsoft/botframework)
* [Didacticiel du modèle de répartition](/azure/bot-service/bot-builder-tutorial-dispatch?branch=master&tabs=cs&view=azure-bot-service-4.0)
* [CLI Dispatch](https://github.com/Microsoft/botbuilder-tools)
* Exemple de bot du modèle de répartition – [.NET](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/14.nlp-with-dispatch), [Node.js](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/14.nlp-with-dispatch)

## <a name="next-steps"></a>Étapes suivantes

* Découvrir comment [tester un lot](luis-how-to-batch-test.md)

[dispatcher-application-tutorial]: /azure/bot-service/bot-builder-tutorial-dispatch?branch=master
[dispatch-tool]: https://aka.ms/dispatch-tool