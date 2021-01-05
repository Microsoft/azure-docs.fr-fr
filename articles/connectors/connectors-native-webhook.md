---
title: Attendre des événements et y répondre
description: Automatiser les flux de travail qui déclenchent, suspendent et reprennent en fonction des événements au niveau d’un point de terminaison de service à l’aide d’Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: jonfan, logicappspm
ms.topic: conceptual
ms.date: 08/27/2020
tags: connectors
ms.openlocfilehash: 7c6f3c4e3e4a2a29fe6a02c03043e3dfb81a2010
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89227897"
---
# <a name="create-and-run-automated-event-based-workflows-by-using-http-webhooks-in-azure-logic-apps"></a>Créer et exécuter des flux de travail automatisés basés sur des événements à l’aide de Webhooks HTTP dans Azure Logic Apps

Avec [Azure Logic Apps](../logic-apps/logic-apps-overview.md) et le connecteur HTTP Webhook intégré, vous pouvez créer des tâches et des workflows automatisées qui s’abonnent à un point de terminaison de service, attendre des événements spécifiques et exécuter des flux en fonction de ces événements plutôt que de vérifier ou d’*interroger* régulièrement ce point de terminaison.

Voici des exemples de flux de travail basés sur webhook :

* Attendre qu’un élément provienne d’un [Azure Event Hub](https://github.com/logicappsio/EventHubAPI) avant de déclencher l’exécution d’une application logique.
* Attendre une approbation avant de poursuivre un flux de travail.

Cet article explique comment utiliser le déclencheur Webhook et l’action Webhook pour que votre application logique puisse recevoir des événements et y répondre sur un point de terminaison de service.

## <a name="how-do-webhooks-work"></a>Comment fonctionnent les webhooks ?

Un déclencheur de Webhook est basé sur un événement et ne dépend pas de la vérification ou de l’interrogation périodiques de nouveaux éléments. Lorsque vous enregistrez une application logique avec un déclencheur de Webhook, ou lorsque vous modifiez votre application logique en l’activant, le déclencheur de Webhook *s’abonne* au point de terminaison de service spécifié en inscrivant une *URL de rappel* avec ce point de terminaison. Le déclencheur attend ensuite que ce service de point de terminaison appelle l’URL, ce qui a pour effet de démarrer l’exécution de l’application logique. Comme le [déclencheur de demande](connectors-native-reqres.md), l’application logique se déclenche dès que l’événement spécifié se produit. Le déclencheur webhook *se désabonne* du service de point de terminaison si vous supprimez le déclencheur et enregistrez votre application logique, ou lorsque vous modifiez votre application logique en la désactivant.

Une action webhook est également basée sur un événement et *s’abonne* à un service de point de terminaison spécifiques en inscrivant une *URL de rappel* avec ce point de terminaison. L’action du Webhook suspend le flux de travail de l’application logique et attend que le point de terminaison de service appelle l’URL avant de reprendre l’exécution de l’application logique. L’action webhook se *désabonne* du point de terminaison de service dans les cas suivants :

* Lorsque l’action de webhook s’est terminée avec succès.
* Si l’exécution de l’application logique est annulée en attendant une réponse.
* Avant que la logique d’application arrive à expiration.

Par exemple, l’action [**Envoyer un e-mail d’approbation**](connectors-create-api-office365-outlook.md) du connecteur Office 365 Outlook est un exemple d’action de Webhook qui suit ce modèle. Vous pouvez étendre ce modèle à tout service à l’aide de l’action de Webhook.

Pour plus d’informations, consultez les rubriques suivantes :

* [Webhooks et abonnements](../logic-apps/logic-apps-workflow-actions-triggers.md#webhooks-and-subscriptions)
* [Créer des API personnalisées qui prennent en charge un Webhook](../logic-apps/logic-apps-create-api-app.md)

Pour plus d’informations sur le chiffrement, la sécurité et l’autorisation des appels entrants à votre application logique, comme [Transport Layer Security (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security), précédemment appelé Secure Sockets Layer (SSL), ou [Azure Active Directory Open Authentication (Azure AD OAuth)](../active-directory/develop/index.yml), consultez [Sécuriser l’accès et les données – Accès pour les appels entrants aux déclencheurs basés sur des requêtes](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests).

## <a name="prerequisites"></a>Prérequis

* Un compte et un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* URL d’un point de terminaison ou d’une API déjà déployés prenant en charge le modèle d’abonnement et de désabonnement de Webhook pour des [déclencheurs de Webhook](../logic-apps/logic-apps-create-api-app.md#webhook-triggers) ou des [actions de Webhook](../logic-apps/logic-apps-create-api-app.md#webhook-actions) dans des applications logiques, selon le cas.

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md). Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md)

* Application logique dans laquelle vous souhaitez attendre des événements spécifiques au point de terminaison cible. Pour commencer à utiliser le déclencheur de Webhook HTTP, [créez une application logique vide](../logic-apps/quickstart-create-first-logic-app-workflow.md). Pour utiliser l’action de Webhook HTTP, démarrez votre application logique avec un déclencheur de votre choix. Cet exemple utilise le déclencheur HTTP en tant que première étape.

## <a name="add-an-http-webhook-trigger"></a>Ajouter un déclencheur de Webhook HTTP

Ce déclencheur intégré appelle le point de terminaison d’abonnement sur le service cible et inscrit une URL de rappel auprès de celui-ci. Votre application logique attend ensuite que le service cible envoie une demande `HTTP POST` à l’URL de rappel. Lorsque cet événement se produit, le déclencheur s’active et transmet toutes les données de la demande au workflow.

1. Connectez-vous au [portail Azure](https://portal.azure.com). Ouvrez votre application logique vide dans le Concepteur d’application logique.

1. Dans la zone de recherche du concepteur, entrez `http webhook` en tant que filtre. Dans la liste **Déclencheurs**, sélectionnez le déclencheur **Webhook HTTP**.

   ![Sélectionner le déclencheur Webhook HTTP](./media/connectors-native-webhook/select-http-webhook-trigger.png)

   Cet exemple renomme le déclencheur en `HTTP Webhook trigger` afin que l’étape ait un nom plus descriptif. Par ailleurs, l’exemple ajoute ensuite une action de Webhook HTTP, et les deux noms doivent être uniques.

1. Spécifiez les valeurs des [paramètres du déclencheur de Webhook HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-webhook-trigger) que vous souhaitez utiliser pour les appels d’abonnement et de désabonnement.

   Dans cet exemple, le déclencheur comprend les méthodes, URI et corps de message à utiliser lors de l’exécution des opérations d’abonnement et de désabonnement.

   ![Entrer les paramètres du déclencheur de Webhook HTTP](./media/connectors-native-webhook/http-webhook-trigger-parameters.png)

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Abonnement – Méthode** | Oui | Méthode à utiliser lors de l’abonnement au point de terminaison cible |
   | **Abonnement – URI** | Oui | URL à utiliser pour l’abonnement au point de terminaison cible |
   | **Abonnement – Corps** | Non | Corps de message à inclure dans la demande d’abonnement. Cet exemple inclut l’URL de rappel qui identifie de façon unique l’abonné, qui est votre application logique, en utilisant l’expression `@listCallbackUrl()` pour récupérer l’URL de rappel de votre application logique. |
   | **Désabonnement – Méthode** | Non | Méthode à utiliser lors du désabonnement du point de terminaison cible |
   | **Désabonnement – URI** | Non | URL à utiliser pour le désabonnement du point de terminaison cible |
   | **Désabonnement – Corps** | Non | Corps de message facultatif à inclure dans la demande de désabonnement <p><p>**Remarque** : Cette propriété ne prend pas en charge l’utilisation de la fonction `listCallbackUrl()`. Toutefois, le déclencheur inclut et envoie automatiquement les en-têtes `x-ms-client-tracking-id` et `x-ms-workflow-operation-name` que le service cible peut utiliser pour identifier l’abonné de façon unique. |
   ||||

1. Pour ajouter d’autres propriétés de déclencheur, ouvrez la liste **Ajouter un nouveau paramètre**.

   ![Ajouter d’autres propriétés de déclencheur](./media/connectors-native-webhook/http-webhook-trigger-add-properties.png)

   Par exemple, si vous avez besoin d’utiliser l’authentification, vous pouvez ajouter les propriétés **Abonnement – Authentification** et **Désabonnement – Authentification**. Pour en savoir plus sur les types d’authentification disponibles pour Webhook HTTP, consultez [Ajouter l’authentification aux appels sortants](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound).

1. Continuez à générer le flux de travail de votre application logique avec des actions qui s’exécutent quand le déclencheur se déclenche.

1. Lorsque vous avez terminé, pensez à enregistrer votre application logique. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

   L’enregistrement de votre application logique appelle le point de terminaison d’abonnement sur le service cible et inscrit l’URL de rappel. Votre application logique attend ensuite que le service cible envoie une demande `HTTP POST` à l’URL de rappel. Lorsque cet événement se produit, le déclencheur s’active et transmet toutes les données de la demande au workflow. Si cette opération aboutit, le déclencheur se désabonne du point de terminaison, et votre application logique continue d’exécuter le workflow restant.

## <a name="add-an-http-webhook-action"></a>Ajouter une action de Webhook HTTP

Cette action intégrée appelle le point de terminaison d’abonnement sur le service cible et inscrit une URL de rappel auprès de celui-ci. Ensuite, votre application logique s’interrompt et attend que le service cible envoie une demande `HTTP POST` à l’URL de rappel. Lorsque cet événement se produit, l’action transmet toutes les données de la demande au workflow. Si l’opération aboutit, l’action se désabonne du point de terminaison, et votre application logique continue d’exécuter le workflow restant.

1. Connectez-vous au [portail Azure](https://portal.azure.com). Ouvrez votre application logique dans le Concepteur d’applications logiques.

   Cet exemple utilise le déclencheur de Webhook HTTP en tant que première étape.

1. Sous l’étape où vous souhaitez ajouter l’action de Webhook HTTP, sélectionnez **Étape suivante**.

   Pour ajouter une action entre des étapes, placez votre pointeur au-dessus de la flèche qui les sépare. Cliquez sur le signe ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Dans la zone de recherche du concepteur, entrez `http webhook` en tant que filtre. Dans la liste **Actions**, sélectionnez l’action **Webhook HTTP**.

   ![Sélectionner l’action de Webhook HTTP](./media/connectors-native-webhook/select-http-webhook-action.png)

   Cet exemple renomme l’action « Action de Webhook HTTP » afin qu’elle ait un nom plus descriptif.

1. Spécifiez les valeurs des paramètres d’action de Webhook HTTP, qui sont similaires aux [paramètres du déclencheur de Webhook HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-webhook-trigger) que vous souhaitez utiliser pour les appels d’abonnement et de désabonnement.

   Dans cet exemple, l’action comprend les méthodes, URI et corps de message à utiliser lors de l’exécution des opérations d’abonnement et de désabonnement.

   ![Entrer des paramètres d’action de Webhook HTTP](./media/connectors-native-webhook/http-webhook-action-parameters.png)

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Abonnement – Méthode** | Oui | Méthode à utiliser lors de l’abonnement au point de terminaison cible |
   | **Abonnement – URI** | Oui | URL à utiliser pour l’abonnement au point de terminaison cible |
   | **Abonnement – Corps** | Non | Corps de message à inclure dans la demande d’abonnement. Cet exemple inclut l’URL de rappel qui identifie de façon unique l’abonné, qui est votre application logique, en utilisant l’expression `@listCallbackUrl()` pour récupérer l’URL de rappel de votre application logique. |
   | **Désabonnement – Méthode** | Non | Méthode à utiliser lors du désabonnement du point de terminaison cible |
   | **Désabonnement – URI** | Non | URL à utiliser pour le désabonnement du point de terminaison cible |
   | **Désabonnement – Corps** | Non | Corps de message facultatif à inclure dans la demande de désabonnement <p><p>**Remarque** : Cette propriété ne prend pas en charge l’utilisation de la fonction `listCallbackUrl()`. Toutefois, l’action inclut et envoie automatiquement les en-têtes `x-ms-client-tracking-id` et `x-ms-workflow-operation-name` que le service cible peut utiliser pour identifier l’abonné de façon unique. |
   ||||

1. Pour ajouter d’autres propriétés d’action, ouvrez la liste **Ajouter un nouveau paramètre**.

   ![Ajouter d’autres propriétés d’action](./media/connectors-native-webhook/http-webhook-action-add-properties.png)

   Par exemple, si vous avez besoin d’utiliser l’authentification, vous pouvez ajouter les propriétés **Abonnement – Authentification** et **Désabonnement – Authentification**. Pour en savoir plus sur les types d’authentification disponibles pour Webhook HTTP, consultez [Ajouter l’authentification aux appels sortants](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound).

1. Lorsque vous avez terminé, pensez à enregistrer votre application logique. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

   Votre application logique appelle le point de terminaison d’abonnement sur le service cible et inscrit l’URL de rappel. L’application logique suspend ensuite le worflow et attend que le service cible envoie une demande `HTTP POST` à l’URL de rappel. Lorsque cet événement se produit, l’action transmet toutes les données de la demande au workflow. Si l’opération aboutit, l’action se désabonne du point de terminaison, et votre application logique continue d’exécuter le workflow restant.

## <a name="trigger-and-action-outputs"></a>Sorties des déclencheurs et des actions

Voici d’autres informations sur les sorties d’un déclencheur ou d’une action de Webhook HTTP qui renvoient les informations suivantes :

| Nom de la propriété | Type | Description |
|---------------|------|-------------|
| headers | object | En-têtes de la requête |
| body | object | Objet JSON | Objet avec le contenu du corps de la requête |
| Code d’état | int | Code d’état de la requête |
|||

| Code d’état | Description |
|-------------|-------------|
| 200 | OK |
| 202 | Acceptée |
| 400 | Demande incorrecte |
| 401 | Non autorisé |
| 403 | Interdit |
| 404 | Introuvable |
| 500 | Erreur interne du serveur. Une erreur inconnue s’est produite. |
|||

## <a name="connector-reference"></a>Référence de connecteur

Pour plus d’informations sur les paramètres de déclencheur et d’action, qui sont similaires entre eux, voir [Paramètres de Webhook HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-webhook-trigger).

## <a name="next-steps"></a>Étapes suivantes

* [Sécuriser l’accès et les données – Accès pour les appels entrants aux déclencheurs basés sur des requêtes](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests)
* [Connecteurs pour Logic Apps](../connectors/apis-list.md)
