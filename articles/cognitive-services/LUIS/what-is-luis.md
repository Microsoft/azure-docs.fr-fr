---
title: Qu’est-ce que le service Language Understanding (LUIS) ?
description: 'Language Understanding (LUIS) : service d’API basé sur le cloud qui utilise le machine learning sur du langage naturel pour prédire une signification et extraire des informations.'
keywords: Azure, intelligence artificielle, ia, traitement en langage naturel, tln, compréhension du langage naturel, cln, LUIS, ia conversationnelle, chatbot ia, chatbot tln, ia tln, luis azure
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: overview
ms.date: 11/23/2020
ms.custom: cog-serv-seo-aug-2020
ms.openlocfilehash: fb36ab8a86a89d6383f93ad58c23956472841de2
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95539258"
---
# <a name="what-is-language-understanding-luis"></a>Qu’est-ce que le service Language Understanding (LUIS) ?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Language Understanding (LUIS) est un service d’IA conversationnelle qui applique une intelligence de machine learning personnalisée au texte en langage naturel des conversations d’un utilisateur afin d’en prédire le sens général et d’en extraire des informations détaillées pertinentes.

Une application cliente pour LUIS est une application de conversation qui communique avec un utilisateur en langage naturel pour accomplir une tâche. Des applications de réseaux sociaux, des chatbots d’IA et des applications vocales pour poste de travail sont des exemples d’applications clientes.

![Image conceptuelle de 3 applications clientes fonctionnant avec Cognitive Services Language Understanding (LUIS)](./media/luis-overview/luis-entry-point.png "Image conceptuelle de 3 applications clientes fonctionnant avec Cognitive Services Language Understanding (LUIS)")

## <a name="use-luis-in-a-chat-bot"></a>Utiliser LUIS dans un bot conversationnel

<a name="Accessing-LUIS"></a>

Une fois l’application Azure LUIS publiée, une application cliente envoie des énoncés (textes) à l’[API][endpoint-apis] du point de terminaison de traitement du langage naturel LUIS et reçoit les résultats sous forme de réponses JSON. Un bot conversationnel est un exemple d’application cliente courante pour LUIS.


![Image conceptuelle de LUIS fonctionnant avec un bot conversationnel pour prédire le texte de l’utilisateur avec une compréhension du langage naturel (NLP)](./media/luis-overview/LUIS-chat-bot-request-response.svg "Image conceptuelle de LUIS fonctionnant avec un bot conversationnel pour prédire le texte de l’utilisateur avec une compréhension du langage naturel (NLP)")

|Étape|Action|
|:--|:--|
|1|L’application cliente envoie _l’énoncé_ de l’utilisateur (texte dans ses propres mots), « I want to call my HR rep » (Je souhaite contacter mon représentant des ressources humaines), au point de terminaison LUIS sous la forme d’une requête HTTP.|
|2|LUIS vous permet de concevoir vos modèles de langage personnalisés pour ajouter de l’intelligence à votre application. Les modèles de langage formés par apprentissage automatique prennent le texte d’entrée non structuré de l’utilisateur et retournent une réponse au format JSON, avec une intention principale, `HRContact`. La réponse minimale du point de terminaison JSON contient l’énoncé de la requête, ainsi que l’intention présentant le score le plus élevé. Elle peut également extraire des données telles que l’entité _Type de contact_.|
|3|L’application cliente utilise la réponse JSON pour prendre des décisions concernant la manière dont les requêtes de l’utilisateur doivent être traitées. Ces décisions peuvent inclure un arbre de décision dans le code de l’infrastructure de bots, ainsi que des appels d’autres services. |

L’application LUIS offre l’intelligence nécessaire pour permettre à l’application cliente d’effectuer des choix pertinents. LUIS ne fournit pas ces choix.

<a name="Key-LUIS-concepts"></a>
<a name="what-is-a-luis-model"></a>

## <a name="natural-language-understanding-nlu"></a>Compréhension du langage naturel

[LUIS fournit une fonctionnalité d’intelligence artificielle (IA)](artificial-intelligence.md "LUIS fournit l’intelligence artificielle (IA)") appelée « compréhension du langage naturel », qui est un sous-ensemble de l’IA du traitement en langage naturel.

Votre application LUIS contient un modèle de langage naturel propre à un domaine. Vous pouvez démarrer l’application LUIS avec un modèle de domaine prédéfini, créer votre propre modèle ou fusionner les éléments d’un domaine prédéfini avec vos propres informations personnalisées.

* **Modèle prédéfini** LUIS dispose de nombreux modèles de domaine prédéfinis, incluant des intentions, des énoncés et des entités prédéfinies. Vous pouvez utiliser les entités prédéfinies sans avoir à utiliser les intentions ni les énoncés du modèle prédéfini. Les [modèles de domaines prédéfinis](./howto-add-prebuilt-models.md "Modèles de domaine prédéfinis") intègrent l’ensemble de la conception à votre intention et constituent un excellent moyen de commencer à utiliser LUIS rapidement.

* Le **modèle personnalisé** LUIS vous offre plusieurs façons d’identifier vos propres modèles personnalisés, notamment les intentions et les entités. Les entités incluent des entités issues du machine learning, des entités spécifiques ou littérales et une combinaison des deux.

Découvrez plus d’informations sur l’[l’IA de traitement en langage naturel](artificial-intelligence.md "NLP") et sur la partie LUIS de la compréhension du langage naturel.

## <a name="step-1-design-and-build-your-model"></a>Étape 1 : Concevoir et générer votre modèle

Concevez votre modèle avec des catégories d’intentions utilisateur appelées **[intentions](luis-concept-intent.md "intentions")** . Chaque intention nécessite des exemples **[d’énoncés](luis-concept-utterance.md "énoncés")** utilisateur. Chaque énoncé peut fournir diverses données qui doivent être extraites à l’aide d’[entités de machine learning](luis-concept-entity-types.md#effective-machine-learned-entities "Entité de machine-learning").

|Exemple d’énoncé d’un utilisateur|Intentionnel|Données extraites|
|-----------|-----------|-----------|
|`Book a flight to Seattle?`|RéserverVol|Seattle|
|`When does your store open?`|StoreHoursAndLocation|ouvert|
|`Schedule a meeting at 1pm with Bob in Distribution`|ScheduleMeeting|13h, Bob|

Créez le modèle à l’aide des API de [création](https://go.microsoft.com/fwlink/?linkid=2092087 "création"), du **[portail LUIS](https://www.luis.ai "Portail LUIS")**, ou des deux. Découvrez comment créer un modèle avec le [portail](get-started-portal-build-app.md "portail") et les [bibliothèques de client des SDK](azure-sdk-quickstart.md "Bibliothèques clientes du SDK").

## <a name="step-2-get-the-query-prediction"></a>Étape 2 : Obtenir une prédiction de requête

Une fois que vous avez entraîné et publié le modèle de votre application sur le point de terminaison, l’application cliente (par exemple, un chatbot) envoie les énoncés à l’API du [point de terminaison](https://go.microsoft.com/fwlink/?linkid=2092356 "endpoint") de prédiction. L’API applique le modèle à l’énoncé d’analyse et répond avec les résultats de prédiction au format JSON.

La réponse minimale du point de terminaison JSON contient l’énoncé de la requête, ainsi que l’intention présentant le score le plus élevé. Elle peut également extraire des données telles que l’entité **Contact Type** et ke sentiment général ci-après.

```JSON
{
    "query": "I want to call my HR rep",
    "prediction": {
        "topIntent": "HRContact",
        "intents": {
            "HRContact": {
                "score": 0.8582669
            }
        },
        "entities": {
            "Contact Type": [
                "call"
            ]
        },
        "sentiment": {
            "label": "neutral",
            "score": 0.5
        }
    }
}
```

## <a name="step-3-improve-model-prediction"></a>Étape 3 : Améliorer la prédiction du modèle

Une fois que votre application LUIS est publiée et reçoit des énoncés utilisateur réels, LUIS fournit un [apprentissage actif](luis-concept-review-endpoint-utterances.md "apprentissage actif") des énoncés de point de terminaison pour améliorer la précision de la prédiction. Passez en revue ces suggestions dans le cadre de vos tâches de maintenance et de votre cycle de développement.

<a name="using-luis"></a>

## <a name="development-lifecycle-and-tools"></a>Outils et cycle de vie de développement
LUIS offre différents outils, un contrôle de version et des fonctionnalités de collaboration avec d’autres auteurs LUIS à intégrer au [cycle de vie de développement complet](luis-concept-app-iteration.md "cycle de vie de développement").

En tant qu’API REST, Language Understanding (LUIS) est utilisable avec n’importe quel produit, service ou infrastructure avec une requête HTTP. LUIS fournit également des bibliothèques de client (SDK) pour plusieurs langages de programmation. En savoir plus sur les [ressources de développement](developer-reference-resource.md "ressources pour les développeurs") fournies.

Outils simplifiant et accélérant l’utilisation de LUIS avec un bot :
* [LUIS CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS "Interface de ligne de commande LUIS") Le package NPM offre des fonctions de création et de prédiction avec un outil en ligne de commande autonome ou sous forme d’importation.
* [LUISGen](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUISGen "LUISGen") LUISGen est un outil permettant de générer un code source C# et Typescript fortement typé à partir d’un modèle LUIS exporté.
* [Dispatch](https://aka.ms/dispatch-tool "Dispatch") permet d’utiliser plusieurs applications LUIS et QnA Maker à partir d’une application parente à l’aide du modèle de répartiteur.
* [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown "LUDown") LUDown est un outil en ligne de commande facilitant la gestion des modèles de langage pour votre bot.

## <a name="integrate-with-a-bot"></a>Intégration à un bot

Utilisez [Azure Bot Service](/azure/bot-service/ "Azure Bot Service") avec [Microsoft Bot Framework](https://dev.botframework.com/ "Microsoft Bot Framework") pour créer et déployer un chatbot. Concevez et développez des applications avec l’outil d’interface graphique [Composer](/composer/ "Composer") ou avec des [exemples de bots fonctionnels](https://github.com/microsoft/BotBuilder-Samples "exemples de bots fonctionnels") conçus pour les principaux scénarios de bot.

## <a name="integrate-with-other-cognitive-services"></a>Intégration à d’autres services Cognitive Services

Autres services Cognitive Services utilisés avec LUIS :
* [QnA Maker](../QnAMaker/overview/overview.md "QnA Maker") permet à différents types de textes de se combiner en une question et de répondre à la base de connaissances.
* [Speech](../Speech-Service/overview.md "Service Speech") convertit les requêtes énoncées à voix haute sous forme de texte.

LUIS fournit des fonctionnalités tirées de l’analyse de texte pour vos ressources LUIS existantes. Ces fonctionnalités incluent l’[analyse des sentiments](luis-how-to-publish-app.md#configuring-publish-settings "analyse des sentiments") et l’[extraction de phrases clés](luis-reference-prebuilt-keyphrase.md "à l’extraction de phrases clés") avec l’entité keyPhrase prédéfinie.

## <a name="learn-with-the-quickstarts"></a>Apprendre avec les guides de démarrage rapide

En savoir plus sur LUIS grâce aux guides de démarrage rapide via le [portail](get-started-portal-build-app.md "portail") et les [bibliothèques de client SDK](azure-sdk-quickstart.md "Bibliothèques clientes du SDK").


## <a name="deploy-on-premises-using-docker-containers"></a>Déployer localement en utilisant des conteneurs Docker

[Utilisez des conteneurs LUIS](luis-container-howto.md) pour déployer localement des fonctionnalités d’API. Ces conteneurs Docker vous donnent la possibilité de rapprocher le service plus près de vos données, ce qui peut être souhaitable pour des raisons de conformité, de sécurité ou opérationnelles.

## <a name="next-steps"></a>Étapes suivantes

* [Nouveautés](whats-new.md "Nouveautés") du service et documentation
* [Planifier votre application](luis-how-plan-your-app.md "Planifier votre application") avec des [intentions](luis-concept-intent.md "intentions") et des [entités](luis-concept-entity-types.md "entities").
* [Interroger le point de terminaison de prédiction](luis-get-started-get-intent-from-browser.md "Interroger le point de terminaison de prédiction")
* [Ressources pour les développeurs](developer-reference-resource.md "Ressources pour les développeurs") pour LUIS.

[bot-framework]: /bot-framework/
[flow]: /connectors/luis/
[authoring-apis]: https://go.microsoft.com/fwlink/?linkid=2092087
[endpoint-apis]: https://go.microsoft.com/fwlink/?linkid=2092356
[qnamaker]: https://qnamaker.ai/