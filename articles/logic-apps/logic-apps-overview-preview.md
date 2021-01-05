---
title: Vue d’ensemble de la préversion Azure Logic Apps
description: Azure Logic Apps (préversion) est une solution cloud permettant de créer des workflows avec état et sans état automatisés qui intègrent des applications, des données, des services et des systèmes avec un minimum de code pour les scénarios de niveau entreprise.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm, az-logic-apps-dev
ms.topic: conceptual
ms.date: 12/07/2020
ms.openlocfilehash: 0c4d0416c107c9ceb4cd0dbdf80761ab9b52d56c
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96923993"
---
# <a name="overview-azure-logic-apps-preview"></a>Présentation : Préversion Azure Logic Apps

> [!IMPORTANT]
> Cette fonctionnalité en préversion publique est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Grâce à Azure Logic Apps (préversion), vous pouvez créer des solutions d’automatisation et d’intégration entre les applications, les données, les services cloud et les systèmes en créant et en exécutant des applications logiques qui incluent des [workflows *avec état* et *sans état*](#stateful-stateless) à l’aide du nouveau type de ressource **Application logique (préversion)** . Grâce à ce nouveau type d’application logique, vous pouvez créer plusieurs workflows alimentés par le runtime d’Azure Logic Apps (préversion) remanié, qui offre de la portabilité, de meilleures performances et de la flexibilité pour le déploiement et l’exécution dans divers environnements d’hébergement, non seulement dans Azure, mais également dans les conteneurs Docker.

Comment est-ce possible ? Le runtime remanié utilise le [modèle d’extensibilité d’Azure Functions](../azure-functions/functions-bindings-register.md) et est hébergé en tant qu’extension sur le runtime d’Azure Functions. Cette architecture signifie que vous pouvez exécuter le nouveau type d’application logique partout où Azure Functions s’exécute. Vous pouvez héberger le runtime remanié sur pratiquement toutes les topologies de réseau et choisir n’importe quelle taille de calcul disponible pour gérer la charge de travail nécessaire requise par vos workflows. Pour plus d’informations, consultez [Introduction à Azure Functions](../azure-functions/functions-overview.md) et [Déclencheurs et liaisons Azure Functions](../azure-functions/functions-triggers-bindings.md).

Vous pouvez créer la ressource **Application logique (préversion)** en [commençant dans le portail Azure](create-stateful-stateless-workflows-azure-portal.md) ou en [créant un projet dans Visual Studio Code à l’aide de l’extension Azure Logic Apps (préversion)](create-stateful-stateless-workflows-visual-studio-code.md). En outre, dans Visual Studio Code, vous pouvez générer *et exécuter localement* vos workflows dans votre environnement de développement. Que vous utilisiez le portail ou Visual Studio Code, vous pouvez déployer et exécuter le nouveau type d’application logique dans les mêmes types d’environnements d’hébergement.

Cette présentation couvre les sujets suivants :

* [Différences entre Azure Logic Apps (préversion), l’environnement mutualisé d’Azure Logic Apps et l’environnement du service d’intégration](#preview-differences).

* [Différences entre les workflows avec état et sans état](#stateful-stateless), notamment les différences de comportement entre [les workflows avec état et sans état imbriqués](#nested-behavior).

* [Capacités de cette préversion publique](#public-preview-contents).

* [Mode de fonctionnement du modèle de tarification](#pricing-model).

* [Capacités modifiées, limitées, non disponibles ou non prises en charge](#limited-unavailable-unsupported).

* [Limites dans Azure Logic Apps (préversion)](#limits).

Pour plus d’informations, consultez les autres articles suivants :

* [Azure Logic Apps s’exécutant partout – Présentation approfondie du runtime](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-runtime-deep-dive/ba-p/1835564)

* [Problèmes connus de la préversion publique de Logic Apps (GitHub)](https://github.com/Azure/logicapps/blob/master/articles/logic-apps-public-preview-known-issues.md)

<a name="preview-differences"></a>

## <a name="how-does-azure-logic-apps-preview-differ"></a>En quoi Azure Logic Apps (préversion) est-il différent ?

Le runtime d’Azure Logic Apps (préversion) utilise l’extensibilité d’[Azure Functions](../azure-functions/functions-overview.md) et est hébergé en tant qu’extension sur le runtime d’Azure Functions. Cette architecture signifie que vous pouvez exécuter le nouveau type d’application logique partout où Azure Functions s’exécute. Vous pouvez héberger le runtime d’Azure Logic Apps (préversion) sur pratiquement toutes les topologies de réseau possibles et choisir n’importe quelle taille de calcul disponible pour gérer la charge de travail dont votre workflow a besoin. Pour plus d’informations sur l’extensibilité d’Azure Functions, consultez [Kit de développement logiciel (SDK) WebJobs : Création de liaisons d’entrée et de sortie personnalisées](https://github.com/Azure/azure-webjobs-sdk/wiki/Creating-custom-input-and-output-bindings).

Avec cette nouvelle approche, le runtime d’Azure Logic Apps (préversion) et vos workflows font tous deux partie de votre application que vous pouvez empaqueter ensemble. Cette capacité vous permet de déployer et d’exécuter vos workflows en copiant simplement les artefacts dans l’environnement d’hébergement et en démarrant votre application. Cette approche offre également une expérience plus standardisée pour la création de pipelines DevOps autour des projets de workflow pour l’exécution des tests et validations requis avant le déploiement des modifications dans les environnements de production. Pour plus d’informations, consultez [Azure Logic Apps s’exécutant partout – Présentation approfondie du runtime](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-runtime-deep-dive/ba-p/1835564).

Le tableau suivant résume les différences dans la manière dont les workflows partagent les ressources en fonction de l’environnement dans lequel ils s’exécutent. Pour connaître les différences de limites, consultez [Limites dans Azure Logic Apps (préversion)](#limits).

| Environnement | Partage et consommation des ressources |
|-------------|----------------------------------|
| Azure Logic Apps (mutualisé) | Les workflows *de clients se trouvant dans plusieurs locataires* partagent le même traitement (calcul), le même stockage, le même réseau, et ainsi de suite. |
| Azure Logic Apps (préversion) | Les workflows qui se trouvent *dans la même application logique* partagent le même traitement (calcul), le même stockage, le même réseau, et ainsi de suite. |
| Environnement de service d’intégration (non disponible dans le préversion) | Les workflows qui se trouvent *dans le même environnement* partagent le même traitement (calcul), le même stockage, le même réseau, et ainsi de suite. |
||||

En attendant, vous pouvez toujours créer le type d’application logique d’origine dans le portail Azure et dans Visual Studio Code à l’aide de l’extension Azure Logic Apps d’origine. Bien que les expériences de développement diffèrent entre le type d’application logique d’origine et le nouveau type d’application logique, votre abonnement Azure peut inclure les deux types. Vous pouvez afficher toutes les applications logiques déployées dans votre abonnement Azure et y accéder, mais elles sont organisées dans leurs propres catégories et sections.

<a name="stateful-stateless"></a>

## <a name="stateful-and-stateless-workflows"></a>Workflows avec état et sans état

* *Avec état*

  Créez des workflows avec état lorsque vous devez conserver, examiner ou référencer des données d’événements précédents. Ces workflows enregistrent les entrées et les sorties de chaque action, ainsi que leurs états, dans un stockage externe qui permet d’examiner les détails et l’historique d’exécution à l’issue de chaque exécution. Les workflows avec état offrent une haute résilience en cas d’interruption. Une fois les services et systèmes restaurés, vous pouvez reconstituer les exécutions interrompues à partir de l’état enregistré et réexécuter les workflows jusqu’à leur terme. Les flux de travail avec état peuvent continuer de s’exécuter pendant jusqu’à un an.

* *Sans état*

  Créez des workflows sans état lorsque vous n’avez pas besoin d’enregistrer, d’examiner ou de référencer des données d’événements précédents dans un stockage externe pour les consulter ultérieurement. Ces workflows enregistrent les entrées et les sorties pour chaque action et leurs états *uniquement en mémoire*, au lieu de transférer ces données vers un stockage externe. Par conséquent, les workflows sans état offrent des temps d’exécution plus courts généralement inférieurs à cinq minutes, des performances plus rapides avec des temps de réponse plus courts, un débit plus élevé, et des coûts d’exécution réduits, car les détails et l’historique d’exécution ne sont pas conservés dans un stockage externe. Toutefois, en cas de panne, les exécutions interrompues ne sont pas automatiquement restaurées, de sorte que l’appelant doit relancer manuellement les exécutions interrompues. Ces workflows peuvent uniquement s’exécuter de façon synchrone.

  Pour faciliter le débogage, vous pouvez activer l’historique des exécutions pour un workflow sans état (ce qui a un certain impact sur les performances), puis le désactiver lorsque vous avez terminé. Pour plus d’informations, consultez [Créer des workflows avec état et sans état dans Visual Studio Code](create-stateful-stateless-workflows-visual-studio-code.md#enable-run-history-stateless) ou [Créer des workflows avec état et sans état dans le portail Azure](create-stateful-stateless-workflows-visual-studio-code.md#enable-run-history-stateless).

  > [!NOTE]
  > Actuellement, les flux de travail sans état prennent en charge uniquement les *actions* pour les [connecteurs gérés](../connectors/apis-list.md#managed-api-connectors) déployés dans Azure, pas pour des déclencheurs. Pour démarrer votre workflow, sélectionnez le [déclencheur intégré Requête, Event Hubs ou Service Bus](../connectors/apis-list.md#built-ins). Ces déclencheurs s’exécutent en mode natif dans le runtime d’Azure Logic Apps (préversion). Pour plus d’informations sur les déclencheurs, actions et connecteurs limités, non disponibles ou non pris en charge, consultez [Capacités modifiées, limitées, non disponibles ou non prises en charge](#limited-unavailable-unsupported).

<a name="nested-behavior"></a>

### <a name="nested-behavior-differences-between-stateful-and-stateless-workflows"></a>Différences de comportement imbriqué entre les workflows avec état et sans état

Vous pouvez [faire en sorte qu’un workflow puisse être appelé](../logic-apps/logic-apps-http-endpoint.md) à partir d’autres workflows qui existent dans la même ressource **Application logique (préversion)** à l’aide du [déclencheur Requête](../connectors/connectors-native-reqres.md), du [déclencheur Webhook HTTP](../connectors/connectors-native-webhook.md) ou de déclencheurs de connecteur géré de [type ApiConnectionWebhook](../logic-apps/logic-apps-workflow-actions-triggers.md#apiconnectionwebhook-trigger) pouvant recevoir des requêtes HTTPS.

Voici les modèles de comportement que les workflows imbriqués peuvent suivre après qu’un workflow parent a appelé un workflow enfant :

* Modèle d’interrogation asynchrone

  Le flux de travail parent n’attend pas de réponse à son appel initial, mais vérifie continuellement l’historique des exécutions du flux de travail enfant jusqu’à la fin de l’exécution de celui-ci. Par défaut, les flux de travail avec état suivent ce modèle, qui est idéal pour les flux de travail enfants de longue durée susceptibles de dépasser les [limites de délai d’expiration de demande](../logic-apps/logic-apps-limits-and-config.md).

* Modèle synchrone (« fire and forget »)

  Le flux de travail enfant accuse réception de l’appel en retournant immédiatement une réponse `202 ACCEPTED`, et le flux de travail parent continue jusqu’à l’action suivante sans attendre les résultats du flux de travail enfant. Au lieu de cela, le flux de travail parent reçoit les résultats lorsque le flux de travail enfant termine son exécution. Les flux de travail enfants avec état qui n’incluent pas d’action Réponse suivent toujours le modèle synchrone. Pour les flux de travail enfants avec état, vous pouvez consulter l’historique des exécutions.

  Pour activer ce comportement, dans la définition JSON du flux de travail, définissez la propriété `operationOptions` sur la valeur `DisableAsyncPattern`. Pour plus d’informations, consultez [Types d’action et de déclencheur – Options d’opération](../logic-apps/logic-apps-workflow-actions-triggers.md#operation-options).

* Déclencher et attendre

  Pour un flux de travail enfant sans état, le flux de travail parent attend une réponse retournant les résultats du flux de travail enfant. Ce modèle fonctionne de manière similaire à l’utilisation [du déclencheur ou de l’action HTTP](../connectors/connectors-native-http.md) intégrés pour appeler un flux de travail enfant. Les flux de travail enfants sans état qui n’incluent pas d’action Réponse retournent immédiatement une réponse `202 ACCEPTED`, mais le flux de travail parent attend que le flux de travail enfant se termine avant de passer à l’action suivante. Ces comportements s’appliquent uniquement aux flux de travail enfants sans état.

Ce tableau spécifie le comportement du flux de travail enfant selon que les flux de travail parent et enfant sont avec état, sans état ou mixtes :

| Flux de travail parent | Flux de travail enfant | Comportement du flux de travail enfant |
|-----------------|----------------|----------------|
| Avec état | Avec état | Asynchrone ou synchrone avec le paramètre `"operationOptions": "DisableAsyncPattern"` |
| Avec état | Sans état | Déclencher et attendre |
| Sans état | Avec état | Synchrone |
| Sans état | Sans état | Déclencher et attendre |
||||

<a name="public-preview-contents"></a>

## <a name="capabilities"></a>Fonctionnalités

Azure Logic Apps (préversion) comprend de nombreuses capacités actuelles et supplémentaires, par exemple :

* Créez des applications logiques et leurs workflows à partir de [plus de 390 connecteurs](/connectors/connector-reference/connector-reference-logicapps-connectors) pour les applications et services SaaS (Software-as-a-service) et PaaS (Platform-as-a-service), ainsi que des connecteurs pour les systèmes locaux.

  * Certains connecteurs gérés, tels qu’Azure Service Bus, Azure Event Hubs et SQL Server, s’exécutent de la même façon que les déclencheurs et les actions intégrés qui sont natifs au runtime d’Azure Logic Apps (préversion), par exemple, le déclencheur Requête et l’action HTTP. Pour plus d’informations, consultez [Azure Logic Apps s’exécutant partout – Extensibilité de connecteur intégré](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-built-in-connector/ba-p/1921272).

  * Vous pouvez utiliser les actions B2B pour Liquid Operations et XML Operations sans compte d’intégration. Pour utiliser ces actions, vous devez disposer de cartes Liquid, de cartes XML ou de schémas XML que vous pouvez charger par le biais des actions respectives dans le portail Azure ou ajouter au dossier **Artefacts** de votre projet Visual Studio Code à l’aide des dossiers **Cartes** et **Schémas**.

  * Créez et déployez des applications logiques pouvant s’exécuter en tout lieu, car le service Azure Logic Apps génère des chaînes de connexion avec signature d’accès partagé (SAP) que ces applications logiques peuvent utiliser pour envoyer des demandes au point de terminaison du runtime de connexion cloud. Le service Logic Apps enregistre ces chaînes de connexion avec d’autres paramètres de l’application pour vous permettre de stocker facilement ces valeurs dans Azure Key Vault quand vous opérez un déploiement sur Azure.

    > [!NOTE]
    > Par défaut, l’[identité managée assignée par le système](../logic-apps/create-managed-service-identity.md) d’une ressource **Application logique (préversion)** est automatiquement activée pour authentifier les connexions au moment de l’exécution. Cette identité diffère des informations d’identification d’authentification ou de la chaîne de connexion que vous utilisez lors de la création d’une connexion. Si vous désactivez cette identité, les connexions ne fonctionneront pas au moment de l’exécution. Pour afficher ce paramètre, dans le menu de votre application logique, sous **Paramètres**, sélectionnez **Identité**.

* Créez des applications logiques avec des workflows sans état qui s’exécutent uniquement en mémoire, de sorte qu’elles se terminent plus rapidement, répondent plus promptement, présentent un débit plus élevé et réduisent le coût d’exécution, car les historiques et données d’exécution entre les actions ne sont pas conservés dans un stockage externe. Si vous le souhaitez, vous pouvez activer l’historique d’exécution pour faciliter le débogage. Pour plus d’informations, consultez [Applications logiques avec et sans état](#stateful-stateless).

* Exécutez, testez et déboguez localement vos applications logiques et leurs workflows dans l’environnement de développement Visual Studio Code.

  Avant d’exécuter et de tester votre application logique, vous pouvez faciliter le débogage en ajoutant et en utilisant des points d’arrêt dans le fichier **workflow.json** d’un workflow. Toutefois, les points d’arrêt sont pris en charge uniquement pour les actions à l’heure actuelle, et non pour les déclencheurs. Pour plus d’informations, consultez [Créer des workflows avec état et sans état dans Visual Studio Code](create-stateful-stateless-workflows-visual-studio-code.md#manage-breakpoints).

* Publiez ou déployez directement des applications logiques et leurs workflows de Visual Studio Code vers divers environnements d’hébergement tels qu’Azure et des [conteneurs Docker](/dotnet/core/docker/introduction).

* Activez les capacités de journalisation et de suivi des diagnostics pour votre application logique en utilisant [Application Insights](../azure-monitor/app/app-insights-overview.md) lorsque la fonctionnalité est prise en charge par votre abonnement Azure et les paramètres de l’application logique.

> [!NOTE]
> Pour plus d’informations sur les problèmes connus actuels, consultez la [page relative aux problèmes connus de la préversion publique de Logic Apps dans GitHub](https://github.com/Azure/logicapps/blob/master/articles/logic-apps-public-preview-known-issues.md).

<a name="pricing-model"></a>

## <a name="pricing-model"></a>Modèle de tarification

Quand vous créez le nouveau type d’application logique dans le portail Azure ou que vous le déployez à partir de Visual Studio Code, vous devez choisir un plan d’hébergement, [App Service ou Premium](../azure-functions/functions-scale.md), pour votre application logique. Ce plan détermine le modèle de tarification qui s’applique à l’exécution de votre application logique. Si vous sélectionnez le plan App Service, vous devez également choisir un [niveau tarifaire](../app-service/overview-hosting-plans.md).

Les workflows *avec état* utilisant un [stockage externe](../azure-functions/functions-scale.md#storage-account-requirements), la [tarification de Stockage Azure](https://azure.microsoft.com/pricing/details/storage/) s’applique aux transactions de stockage que le runtime d’Azure Logic Apps (préversion) exécute. Par exemple, des files d’attente sont utilisées pour la planification, tandis que des tables et blobs sont utilisés pour stocker les états de flux de travail.

> [!NOTE]
> Pendant la préversion publique, l’exécution d’applications logiques sur App Service n’occasionne pas de frais *supplémentaires* en plus du coût du plan sélectionné.

Pour plus d’informations sur les modèles de tarification qui s’appliquent à ce nouveau type de ressource, consultez les rubriques suivantes :

* [Échelle et hébergement dans Azure Functions](../azure-functions/functions-scale.md)
* [Montez une application en puissance dans Azure App Service](../app-service/manage-scale-up.md)
* [Détails de tarification d’Azure Functions](https://azure.microsoft.com/pricing/details/functions/)
* [Détails de tarification d’App Service](https://azure.microsoft.com/pricing/details/app-service/)
* [Détails de tarification de Stockage Azure](https://azure.microsoft.com/pricing/details/storage/)

<a name="limited-unavailable-unsupported"></a>

## <a name="changed-limited-unavailable-or-unsupported-capabilities"></a>Capacités modifiées, limitées, non disponibles ou non prises en charge

Dans Azure Logic Apps (préversion), ces capacités ont changé, ou elles sont actuellement limitées, non disponibles ou non prises en charge :

* **Déclencheurs et actions** : Certains déclencheurs intégrés ne sont pas disponibles, tels que Fenêtre glissante et Traitement par lots. Pour démarrer votre workflow, utilisez le [déclencheur intégré Périodicité, Requête, HTTP, Webhook HTTP, Event Hubs ou Service Bus](../connectors/apis-list.md). Les déclencheurs et actions intégrés s’exécutent en mode natif dans le runtime d’Azure Logic Apps (préversion), tandis que les connecteurs gérés sont déployés dans Azure. Dans le concepteur, les déclencheurs et les actions intégrés s’affichent sous l’onglet **Intégré**, tandis que les déclencheurs et les actions des connecteurs gérés s’affichent sous l’onglet **Azure**.

  > [!NOTE]
  > Pour s’exécuter localement dans Visual Studio Code, les déclencheurs et les actions basés sur un webhook nécessitent une configuration supplémentaire. Pour plus d’informations, consultez [Créer des workflows avec état et sans état dans Visual Studio Code](create-stateful-stateless-workflows-visual-studio-code.md#webhook-setup).

  * Pour les *workflows sans état*, l’onglet **Azure** ne s’affiche pas lorsque vous sélectionnez un déclencheur, car vous pouvez sélectionner uniquement des [*actions* de connecteur géré, et non des déclencheurs](../connectors/apis-list.md#managed-api-connectors). Bien que vous puissiez activer les connecteurs gérés déployés par Azure pour les workflows sans état, le concepteur n’affiche aucun déclencheur de connecteur géré que vous pouvez ajouter.

  * Pour les *workflows avec état*, à l’exception des déclencheurs et des actions répertoriés comme non disponibles ci-dessous, vous pouvez utiliser à la fois [les déclencheurs et les actions des connecteurs gérés](../connectors/apis-list.md#managed-api-connectors).

  * Ces déclencheurs et actions ont été modifiés ou sont actuellement limités, non pris en charge ou non disponibles :

    * Les [*déclencheurs* de la passerelle de données locale](../connectors/apis-list.md#on-premises-connectors) ne sont pas disponibles, mais les actions de la passerelle *le sont*.

    * Les [connecteurs personnalisés](../connectors/apis-list.md#custom-apis-and-connectors) ne sont pas disponibles.

    * L’action intégrée, [Azure Functions – Choisir une fonction Azure](logic-apps-azure-functions.md) est désormais **Azure Function Operations – Appeler une fonction Azure**. Cette action ne fonctionne actuellement que pour les fonctions créées à partir du modèle **Déclencheur HTTP**.

      Dans le portail Azure, vous pouvez sélectionner une fonction de déclencheur HTTP à laquelle vous avez accès en créant une connexion via l’expérience utilisateur. Si vous inspectez la définition JSON de l’action de fonction en mode code ou le fichier **workflow.json**, l’action fait référence à la fonction à l’aide d’une référence `connectionName`. Cette version extrait les informations de la fonction en tant que connexion, que vous pouvez trouver dans le fichier **connections.json** de votre projet, qui est disponible après la création d’une connexion.

      > [!NOTE]
      > Dans la préversion, l’action de la fonction ne prend en charge que l’authentification par chaîne de requête. Azure Logic Apps (préversion) obtient la clé par défaut de la fonction lors de l’établissement de la connexion, stocke cette clé dans les paramètres de votre application et utilise la clé pour l’authentification lors de l’appel de la fonction.
      >
      > Comme avec la version d’origine, si vous renouvelez cette clé, par exemple par le biais de l’expérience Azure Functions dans le portail, l’action de la fonction ne fonctionnera plus en raison de la clé devenue non valide. Pour résoudre ce problème, vous devez recréer la connexion à la fonction que vous souhaitez appeler ou mettre à jour les paramètres de votre application avec la nouvelle clé.

    * L’action intégrée, [Code inlined – Exécuter du code JavaScript](logic-apps-add-run-inline-code.md) est désormais **Opérations de code inlined – Exécuter du code JavaScript inlined**.

      * Les actions Opérations de code inlined n’ont plus besoin d’un compte d’intégration.

      * Si vous utilisez macOS ou Linux, l’option **Opérations de code inlined** est actuellement indisponible lorsque vous utilisez l’extension Azure Logic Apps (préversion) dans Visual Studio Code.

      * Si vous apportez des modifications à une action Opérations de code inlined, vous devez redémarrer votre application logique.

      * Les actions Opérations de code inlined ont des [limites mises à jour](logic-apps-overview-preview.md#inline-code-limits).

    * Certains [déclencheurs et actions B2B intégrés pour les comptes d’intégration](../connectors/apis-list.md#integration-account-connectors) ne sont pas disponibles, par exemple, les actions de codage et de décodage **Fichier plat**.

    * L’action intégrée, [Azure Logic Apps – Choisir un workflow d’application logique](logic-apps-http-endpoint.md) est désormais **Workflow Operations – Appeler un workflow dans cette application de workflow**.

* **Disponibilité du plan d’hébergement** : Si vous créez un nouveau type de ressource **Application logique (préversion)** dans le portail Azure ou que vous le déployez à partir de Visual Studio Code, vous pouvez uniquement utiliser le plan d’hébergement Premium ou App Service dans Azure. Les plans d’hébergement dits de consommation ne sont pas disponibles ni pris en charge pour le déploiement de ce type de ressource. Vous pouvez effectuer le déploiement de Visual Studio Code vers un conteneur Docker, mais pas vers un [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md).

* **Branches parallèles** : Actuellement, vous ne pouvez pas ajouter de branches parallèles à l’aide de la nouvelle expérience de concepteur. Toutefois, vous pouvez toujours ajouter ces branches par le biais de l’expérience de concepteur d’origine et les faire apparaître dans le nouveau concepteur.

  1. En bas du concepteur, désactivez la nouvelle expérience en sélectionnant le contrôle **Nouveau canevas**.

  1. Ajoutez les branches parallèles à votre workflow.

  1. Activez la nouvelle expérience en sélectionnant à nouveau le contrôle **Nouveau canevas**.

* **Contrôle de zoom** : Le contrôle de zoom n’est pas disponible actuellement sur le concepteur.

* **Débogage des points d’arrêt dans Visual Studio Code** : Bien que vous puissiez ajouter et utiliser des points d’arrêt à l’intérieur du fichier **workflow.json** pour un workflow, les points d’arrêt sont pris en charge uniquement pour les actions pour le moment, et non pour les déclencheurs. Pour plus d’informations, consultez [Créer des workflows avec état et sans état dans Visual Studio Code](create-stateful-stateless-workflows-visual-studio-code.md#manage-breakpoints).

<a name="limits"></a>

## <a name="updated-limits"></a>Limites mises à jour

Bien que de nombreuses limites d’Azure Logic Apps (préversion) restent les mêmes que les [limites d’Azure Logic Apps mutualisé](logic-apps-limits-and-config.md), ces limites ont été modifiées pour Azure Logic Apps (préversion).

<a name="http-timeout-limits"></a>

### <a name="http-timeout-limits"></a>Limites de délai d’expiration HTTP

Pour un appel entrant ou sortant unique, le délai d’expiration est de 230 secondes (soit 3,9 minutes) pour ces déclencheurs et actions :

* Requête sortante : déclencheur HTTP, action HTTP
* Requête entrante : déclencheur Requête, déclencheur Webhook HTTP, action Webhook HTTP

En comparaison, voici les limites de délai d’attente pour ces déclencheurs et actions dans d’autres environnements où s’exécutent les applications logiques et leurs workflows :

* Azure Logic Apps mutualisé : 120 secondes (soit 2 minutes)
* Environnement de service d’intégration : 240 secondes (soit 4 minutes)

Pour plus d’informations, consultez [Limites HTTP](logic-apps-limits-and-config.md#http-limits).

<a name="managed-connector-limits"></a>

### <a name="managed-connectors"></a>Connecteurs gérés

Les connecteurs gérés sont limités à 50 requêtes par minute et par connexion. Pour travailler sur les problèmes de limitation du connecteur, consultez [Gérer les problèmes de limitation (erreur 429 – « Trop de requêtes ») dans Azure Logic Apps](handle-throttling-problems-429-errors.md#connector-throttling).

<a name="inline-code-limits"></a>

### <a name="inline-code-operations-execute-javascript-code"></a>Opérations de code inlined (Exécuter du code JavaScript)

Pour une définition d’application logique unique, l’action Opérations de code inlined [**Exécuter du code JavaScript**](logic-apps-add-run-inline-code.md) se voit appliquer ces nouvelles limites :

* Le nombre maximal de caractères de code augmente de 1 024 caractères à 100 000 caractères.

* La durée maximale d’exécution du code augmente de 5 à 15 secondes.

Pour plus d’informations, consultez [Limites de définition d’application logique](logic-apps-limits-and-config.md#definition-limits).

## <a name="next-steps"></a>Étapes suivantes

* [Créer des workflows avec état et sans état dans le portail Azure](create-stateful-stateless-workflows-azure-portal.md)
* [Créer des workflows avec état et sans état dans Visual Studio Code](create-stateful-stateless-workflows-visual-studio-code.md)
* [Page relative aux problèmes connus de la préversion publique de Logic Apps dans GitHub](https://github.com/Azure/logicapps/blob/master/articles/logic-apps-public-preview-known-issues.md)

Nous aimerions également que vous nous fassiez part de votre expérience avec Azure Logic Apps (préversion).

* Pour tout bogue ou problème que vous rencontreriez, [créez un problème dans GitHub](https://github.com/Azure/logicapps/issues).
* Pour toute question, demande, commentaire ou retour d’expérience, utilisez de [formulaire de commentaires](https://aka.ms/lafeedback).