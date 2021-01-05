---
title: Planifier votre application - QnA Maker
description: Découvrez comment planifier votre application QnA Maker. Comprenez comment QnA Maker fonctionne et interagit avec d’autres services Azure et certains concepts de la base de connaissances.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: 048b53186aa0be388d9d801cd6590d4295a4faa7
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353168"
---
# <a name="plan-your-qna-maker-app"></a>Planifier votre application QnA Maker

Pour planifier votre application QnA Maker, vous devez comprendre comment QnA Maker fonctionne et interagit avec d’autres services Azure. Vous devez également avoir une bonne compréhension des concepts de la base de connaissances.

## <a name="azure-resources"></a>Ressources Azure

Chaque [ressource Azure](azure-resources.md#resource-purposes) créée avec QnA Maker a une utilité précise. Chaque ressource a sa propre finalité, ses propres limites et son propre [niveau tarifaire](azure-resources.md#pricing-tier-considerations). Il est important de comprendre la fonction de ces ressources afin de pouvoir utiliser cette connaissance dans votre processus de planification.

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

| Ressource | Objectif |
|--|--|
| Ressource [QnA Maker](azure-resources.md#qna-maker-resource) | Création et prédiction de requêtes |
| Ressource [Recherche cognitive](azure-resources.md#cognitive-search-resource) | Stockage et recherche de données |
| [Ressource App Service et plan App Service](azure-resources.md#app-service-and-app-service-plan) | Interroger le point de terminaison de prédiction |
| Ressource [Application Insights](azure-resources.md#application-insights) | Données de télémétrie concernant la prédiction des requêtes |

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

| Ressource | Objectif |
|--|--|
| Ressource [QnA Maker](azure-resources.md#qna-maker-resource) | Création, point de terminaison de prédiction de requête et télémétrie|
| Ressource [Recherche cognitive](azure-resources.md#cognitive-search-resource) | Stockage et recherche de données |

---
### <a name="resource-planning"></a>Planification des ressources

Le niveau gratuit, `F0`, de chaque ressource fonctionne et peut fournir une expérience de création et de prédiction de requêtes. Vous pouvez utiliser ce niveau pour apprendre à créer et à prédire des requêtes. Lorsque vous passez à un scénario de production ou à un scénario dynamique, vous devez réévaluer votre choix de ressources.

#### <a name="qna-maker-resource"></a>Ressource QnA Maker

Une même ressource QnA Maker peut héberger plusieurs bases de connaissances. Le nombre de bases de connaissances est déterminé par la quantité d’index pris en charge du niveau tarifaire Recherche cognitive. En savoir plus sur la [relation entre les index et les bases de connaissances](azure-resources.md#index-usage).

#### <a name="knowledge-base-size-and-throughput"></a>Taille et débit des bases de connaissances

Lorsque vous créez une application réelle, vous devez prévoir des ressources suffisantes pour la taille de votre base de connaissances et pour la quantité de demandes de prédiction des requêtes auxquelles vous vous attendez.

La taille d’une base de connaissances est déterminée par les éléments suivants :
* Les limites du niveau tarifaire des [ressources Recherche cognitive](../../../search/search-limits-quotas-capacity.md)
* Les [limites QnA Maker](../limits.md)

La demande de prédiction des requêtes de la base de connaissances est déterminée par le plan d’application web et l’application web. Reportez-vous aux [paramètres recommandés](azure-resources.md#recommended-settings) pour planifier votre niveau tarifaire.

### <a name="resource-sharing"></a>Partage de ressources

Si certaines de ces ressources sont déjà utilisées, vous pouvez décider de partager ces ressources. Découvrez les ressources qui [peuvent être partagées](azure-resources.md#share-services-with-qna-maker) en sachant que le partage des ressources est un scénario avancé.

Toutes les bases de connaissances qui sont créées dans une même ressource QnA Maker partagent le même point de terminaison de prédiction des requêtes destiné aux **tests**.

### <a name="understand-the-impact-of-resource-selection"></a>Comprendre l’impact du choix des ressources

En choisissant des ressources adaptées, vous permettez à la base de connaissances de répondre correctement aux prédictions de requête.

Si votre base de connaissances ne fonctionne pas correctement, le problème est généralement lié à une mauvaise gestion des ressources.

Un tel problème nécessite une investigation afin de déterminer quelle [ressource doit être changée](azure-resources.md#when-to-change-a-pricing-tier).

## <a name="knowledge-bases"></a>Bases de connaissances

Une base de connaissances est directement liée à sa ressource QnA Maker. Elle contient les paires de questions et réponses (QnA) utilisées pour répondre aux demandes de prédiction des requêtes.

### <a name="language-considerations"></a>Observations relatives au langage

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

C’est la première base de connaissances créée dans votre ressource QnA Maker qui détermine la langue de la ressource. Vous ne pouvez définir qu’une seule langue pour une ressource QnA Maker.

Vous pouvez structurer vos ressources QnA Maker en fonction de la langue ou utiliser [Traducteur](../../translator/translator-info-overview.md) pour traduire une requête dans la langue de la base de connaissances avant de l’envoyer au point de terminaison de prédiction des requêtes.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Vous pouvez à présent avoir des bases de connaissances dans différentes langues au sein d’une même ressource QnA Maker. Quand vous créez la première base de connaissances, vous pouvez choisir d’utiliser la ressource pour des bases de connaissances dans une seule langue ou dans plusieurs langues.

![Sélection de la base de connaissances multilingue QnA Maker managé (préversion)](../media/concept-plan-your-knowledge-base/qnamaker-v2-select-multilanguage-knowledge-base.png)

> [!NOTE]
> Si vous activez des paramètres de langue par base de connaissances, vous ne pouvez pas créer autant de bases de connaissances dans votre ressource QnA Maker. Voici [plus d’informations sur les limitations relatives aux paramètres de langue](./azure-resources.md).

---

### <a name="ingest-data-sources"></a>Ingestion de sources de données

Pour créer une base de connaissances, vous pouvez utiliser l’une des [sources de données](../index.yml) ingérées suivantes :

* Une URL publique
* Une URL SharePoint privée
* Fichier

Le processus d’ingestion convertit les [types de contenu pris en charge](../index.yml) en Markdown. Toutes les modifications apportées à la *réponse* sont effectuées avec Markdown. Après avoir créé votre base de connaissances, vous pouvez modifier les [paires question/réponse](question-answer-set.md) dans le portail QnA Maker à l’aide de la [création de texte enrichi](../how-to/edit-knowledge-base.md#rich-text-editing-for-answer).

### <a name="data-format-considerations"></a>Considérations relatives aux formats des données

Étant donné que le format final d’une paire question/réponse est le format Markdown, il est important de connaître la [prise en charge du format Markdown](../reference-markdown-format.md).

Les images liées doivent être disponibles à partir d’une URL publique à afficher dans le volet de test du portail QnA Maker ou dans une application cliente. QnA Maker ne fournit pas d’authentification du contenu, notamment les images.

### <a name="bot-personality"></a>Personnalité de bot

Ajoutez une personnalité de bot à votre base de connaissances avec [chit-chat](../how-to/chit-chat-knowledge-base.md). Cette personnalité comprend des réponses fournies pour un certain type de conversation, par exemple *professionnelle* ou *amicale*. Chit-chat est fourni sous la forme d’un ensemble de conversations, que vous pouvez ajouter, modifier ou supprimer.

Une personnalité de bot est recommandée si votre bot se connecte à votre base de connaissances. Vous pouvez choisir d’utiliser chit-chat dans votre base de connaissances même si vous vous connectez aussi à d’autres services. Toutefois, vous devrez examiner la façon dont le service bot interagit pour savoir si la conception architecturale est adaptée à votre utilisation.

### <a name="conversation-flow-with-a-knowledge-base"></a>Flux de conversation avec une base de connaissances

Le flux de conversation commence généralement par la salutation d’un utilisateur, comme `Hi` ou `Hello`. Votre base de connaissances peut répondre avec une réponse générique, comme `Hi, how can I help you`, puis elle peut afficher des invites de suivi pour poursuivre la conversation.

Vous devez concevoir votre flux de conversation en utilisant une boucle pour qu’un utilisateur sache comment utiliser votre bot et ne soit pas abandonné par le bot au cours de la conversation. Les [invites de suivi](../how-to/multiturn-conversation.md) fournissent une liaison entre les paires de questions-réponses, et c’est ce qui crée le flux de conversation.

### <a name="authoring-with-collaborators"></a>Création d’une base de connaissances avec des collaborateurs

Les collaborateurs peuvent être des développeurs qui partagent la totalité de la pile de développement de l’application de base de connaissances, ou ils peuvent uniquement être affectés à la création de la base de connaissances.

Lorsque vous créez une base de connaissances, vous pouvez appliquer différentes [autorisations d’accès en fonction du rôle](../index.yml) dans le portail Azure afin de limiter les possibilités d’un collaborateur.

## <a name="integration-with-client-applications"></a>Intégration aux applications clientes

L’intégration aux [applications clientes](../index.yml) s’effectue en envoyant une requête au point de terminaison du runtime de prédiction. Une requête est envoyée à votre base de connaissances, en même temps qu’une requête SDK ou REST envoyée au point de terminaison de l’application web QnA Maker.

Afin d’authentifier correctement une requête du client, l’application cliente doit envoyer les bonnes informations d’identification et le bon ID de la base de connaissances. Si vous utilisez Azure Bot Service, configurez ces paramètres dans le cadre de la configuration du bot dans le portail Azure.

### <a name="conversation-flow-in-a-client-application"></a>Flux de conversation dans une application cliente

Dans une [application cliente](../index.yml) telle qu’un bot Azure, le flux de conversation peut nécessiter des fonctionnalités avant et après l’interaction avec la base de connaissances.

Votre application cliente prend-elle en charge le flux conversationnel, soit en fournissant d’autres moyens pour traiter les invites de suivi, soit en incluant chit-chat ? Si c’est le cas, concevez-les tôt et assurez-vous que la requête de l’application cliente est correctement traitée par un autre service ou lorsqu’elle est envoyée à votre base de connaissances.

### <a name="dispatch-between-qna-maker-and-language-understanding-luis"></a>Distribution entre QnA Maker et LUIS

Une application cliente peut fournir plusieurs fonctionnalités, dont une seule bénéficie des réponses de la base de connaissances. Les autres fonctionnalités doivent encore comprendre le texte conversationnel et en extraire la signification.

Souvent, l’architecture d’une application cliente consiste à utiliser QnA Maker et [LUIS](../../LUIS/what-is-luis.md) ensemble. LUIS permet la classification et l’extraction de texte pour toutes les requêtes, y compris celles envoyées aux autres services. QnA Maker fournit quant à lui les réponses de votre base de connaissances.

Dans un tel scénario d’[architecture partagée](../choose-natural-language-processing-service.md), la distribution entre ces deux services s’effectue à l’aide de l’outil [Dispatch](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch) fourni par Bot Framework.

### <a name="active-learning-from-a-client-application"></a>Apprentissage actif à partir d’une application cliente

QnA Maker utilise l’_apprentissage actif_ pour améliorer la qualité de votre base de connaissances en suggérant des questions alternatives pour votre réponse. L’application cliente est responsable en partie de cet [apprentissage actif](active-learning-suggestions.md). Par le biais d’invites conversationnels, l’application cliente peut détecter que la base de connaissances a renvoyé une réponse qui n’est pas utile à l’utilisateur et peut déterminer une meilleure réponse. L’application cliente doit [renvoyer ces informations à la base de connaissances](active-learning-suggestions.md#how-you-give-explicit-feedback-with-the-train-api) pour améliorer la qualité de la prédiction.

### <a name="providing-a-default-answer"></a>Fournir une réponse par défaut

Si votre base de connaissances ne trouve pas de réponse, elle retourne une _réponse par défaut_. Cette réponse est configurable sur la page **Paramètres** du portail QnA Maker ou dans les [API](/rest/api/cognitiveservices/qnamaker/knowledgebase/update#request-body).

Cette réponse par défaut est différente de la réponse par défaut du bot Azure. Vous configurez la réponse par défaut de votre bot Azure dans le portail Azure dans le cadre des paramètres de configuration. Elle est retournée lorsque le seuil de score n’est pas atteint.

## <a name="prediction"></a>Prédiction

La prédiction correspond à la réponse de votre base de connaissances et contient plus d’informations que la simple réponse. Pour obtenir une réponse de prédiction de requête, utilisez l’[API GenerateAnswer](query-knowledge-base.md).

### <a name="prediction-score-fluctuations"></a>Fluctuations du score de prédiction

Un score peut changer en fonction de plusieurs facteurs :

* Nombre de réponses que vous avez demandées en réponse à [GenerateAnswer](query-knowledge-base.md) avec la propriété `top`
* Diversité des questions alternatives disponibles
* Filtrage des métadonnées
* Requête envoyée à la base de connaissances `test` ou `production`

Il existe un [classement des réponses en deux phases](query-knowledge-base.md#how-qna-maker-processes-a-user-query-to-select-the-best-answer) :
- Recherche cognitive : premier rang. Fixez le nombre de _réponses autorisées_ suffisamment haut pour que les meilleures réponses soient retournées par Recherche cognitive, puis passées dans le ranker de QnA Maker.
- QnA Maker : second rang. Utilisez la caractérisation et l’apprentissage automatique pour déterminer la meilleure réponse.

### <a name="service-updates"></a>Mises à jour de service

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Appliquez les [dernières mises à jour du runtime](../how-to/set-up-qnamaker-service-azure.md#get-the-latest-runtime-updates) pour gérer automatiquement les mises à jour de service.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Dans QnA Maker managé (préversion), le runtime est managé par le service QnA Maker lui-même. Les mises à jour de service ne sont donc pas applicables.

---

### <a name="scaling-throughput-and-resiliency"></a>Mise à l’échelle, débit et résilience

La mise à l’échelle, le débit et la résilience sont déterminés par les [ressources Azure](../how-to/set-up-qnamaker-service-azure.md), leurs niveaux tarifaires et toute architecture environnante telle que [Traffic Manager](../how-to/set-up-qnamaker-service-azure.md#business-continuity-with-traffic-manager).

### <a name="analytics-with-application-insights"></a>Analytique avec Application Insights

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Toutes les requêtes de votre base de connaissances sont stockées dans Application Insights. Utilisez nos [requêtes les plus fréquentes](../how-to/get-analytics-knowledge-base.md) pour comprendre vos métriques.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Dans le déploiement manage, la télémétrie est proposée par le biais du [service Azure Monitor](../../../azure-monitor/index.yml). Utilisez nos [requêtes les plus fréquentes](../how-to/get-analytics-knowledge-base.md) pour comprendre vos métriques.


---

## <a name="development-lifecycle"></a>Cycle de vie de développement

Le [cycle de vie de développement](development-lifecycle-knowledge-base.md) d’une base de connaissances comprend sa modification, son test et sa publication.

### <a name="knowledge-base-development-of-qna-maker-pairs"></a>Développement d’une base de connaissances de paires QnA Maker

Vos [paires de questions-réponses](question-answer-set.md) doivent être conçues et développées en fonction de l’utilisation de votre application cliente.

Chaque paire peut contenir les éléments suivants :
* Métadonnées : filtrables lors de l’interrogation pour vous permettre d’étiqueter vos paires question/réponse avec des informations supplémentaires sur la source, le contenu, le format et l’objectif de vos données.
* Invites de suivi : permettent de déterminer un chemin au sein de votre base de connaissances afin que l’utilisateur obtienne la bonne réponse.
* Questions alternatives : importantes pour trouver une réponse à partir de plusieurs formulations d’une même question. Les [suggestions d’apprentissage actif](active-learning-suggestions.md) deviennent des questions alternatives.

### <a name="devops-development"></a>Développement DevOps

Le développement d’une base de connaissances à insérer dans un pipeline DevOps nécessite que la base de connaissances soit isolée pendant les [tests par lots](../index.yml).

Une base de connaissances partage l’index Recherche cognitive avec toutes les autres bases de connaissances de la ressource QnA Maker. Si la base de connaissances est isolée par une partition, le partage de l’index peut entraîner une différence de score par rapport à celui de la base de connaissances publiée.

Pour obtenir le _même score_ pour les bases de connaissances `test` et `production`, isolez une ressource QnA Maker dans une base de connaissances unique. Dans cette architecture, la ressource ne doit vivre que pendant la durée du test par lots isolé.

## <a name="next-steps"></a>Étapes suivantes

* [Ressources Azure](../how-to/set-up-qnamaker-service-azure.md)
* [Paires de questions-réponses](question-answer-set.md)