---
title: Référence du schéma des types d’actions et de déclencheurs
description: Guide de référence du schéma des types d’actions et de déclencheurs pour le langage de définition du workflow dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: jonfan, logicappspm
ms.topic: conceptual
ms.date: 09/22/2020
ms.custom: devx-track-js
ms.openlocfilehash: 3dbfcacb6ea5922a01d52dfe39189f09f48d4b4a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006079"
---
# <a name="schema-reference-guide-for-trigger-and-action-types-in-azure-logic-apps"></a>Guide de référence du schéma des types d’actions et de déclencheurs dans Azure Logic Apps

Ce document de référence décrit les types généraux utilisés pour identifier les déclencheurs et les actions dans la définition de flux de travail sous-jacente de votre application logique, qui est décrite et validée par le [langage de définition du flux de travail](../logic-apps/logic-apps-workflow-definition-language.md). Pour rechercher des déclencheurs et des actions de connecteur spécifiques que vous pouvez utiliser dans vos applications logiques, consultez la liste figurant sous la [présentation des connecteurs](/connectors/).

<a name="triggers-overview"></a>

## <a name="triggers-overview"></a>Vue d’ensemble des déclencheurs

Tous les flux de travail incluent un déclencheur, qui définit les appels qui instancient et démarrent le workflow. Voici les catégories générales de déclencheurs :

* Déclencheur d’*interrogation*, qui vérifie le point de terminaison d’un service à intervalles réguliers

* Déclencheur *push*, qui crée un abonnement à un point de terminaison et fournit une *URL de rappel* permettant au point de terminaison d’informer le déclencheur quand l’événement spécifié se produit ou quand des données sont disponibles. Le déclencheur attend ensuite la réponse du point de terminaison avant de s’activer

Les déclencheurs ont les éléments principaux suivants, bien que certains soient facultatifs :  
  
```json
"<trigger-name>": {
   "type": "<trigger-type>",
   "inputs": { "<trigger-inputs>" },
   "recurrence": { 
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>
   },
   "conditions": [ "<array-with-conditions>" ],
   "runtimeConfiguration": { "<runtime-config-options>" },
   "splitOn": "<splitOn-expression>",
   "operationOptions": "<operation-option>"
},
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*trigger-name*> | String | Nom du déclencheur | 
| <*trigger-type*> | String | Type de déclencheur, tel que « Http » ou « ApiConnection » | 
| <*trigger-inputs*> | Objet JSON | Entrées qui définissent le comportement du déclencheur | 
| <*time-unit*> | String | Unité de temps qui décrit la fréquence d’activation du déclencheur : "Second", "Minute", "Hour", "Day", "Week", "Month" | 
| <*number-of-time-units*> | Integer | Valeur qui spécifie la fréquence d’activation du déclencheur, qui correspond au nombre d’unités de temps à attendre avant que le déclencheur soit activé à nouveau <p>Les intervalles minimaux et maximaux sont les suivants : <p>- Mois : 1-16 mois </br>Jour : 1-500 jours </br>- Heure : 1-12 000 heures </br>- Minute : 1-72 000 minutes </br>- Seconde : 1-9 999 999 secondes<p>Par exemple, si l’intervalle est défini sur 6 et la fréquence sur « Month », la périodicité est tous les six mois. | 
|||| 

*Facultatif*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*array-with-conditions*> | Array | Tableau qui contient une ou plusieurs [conditions](#trigger-conditions) qui déterminent s’il faut exécuter le workflow. Uniquement disponible pour les déclencheurs. | 
| <*runtime-config-options*> | Objet JSON | Vous pouvez modifier le comportement d’exécution du déclencheur en définissant des propriétés `runtimeConfiguration`. Pour plus d’informations, consultez [Paramètres de configuration d’exécution](#runtime-config-options). | 
| <*splitOn-expression*> | String | Pour les déclencheurs qui retournent un tableau, vous pouvez spécifier une expression qui [fractionne ou *dégroupe*](#split-on-debatch) des éléments de tableau dans plusieurs instances de workflows à des fins de traitement. | 
| <*operation-option*> | String | Vous pouvez modifier le comportement par défaut en définissant la propriété `operationOptions`. Pour plus d’informations, consultez [Options d’opérations](#operation-options). | 
|||| 

## <a name="trigger-types-list"></a>Liste des types de déclencheurs

Chaque type de déclencheur a une interface et des entrées différentes qui définissent son comportement. 

### <a name="built-in-triggers"></a>Déclencheurs intégrés

| Type de déclencheur | Description | 
|--------------|-------------| 
| [**HTTP**](#http-trigger) | Vérifie ou *interroge* n’importe quel point de terminaison. Ce point de terminaison doit être conforme à un contrat de déclencheur spécifique, soit en utilisant un modèle asynchrone `202`, soit en retournant un tableau. | 
| [**HTTPWebhook**](#http-webhook-trigger) | Crée un point de terminaison pouvant être appelé pour votre application logique, mais appelle l’URL spécifiée pour inscrire ou désinscrire. |
| [**Recurrence**](#recurrence-trigger) | Se déclenche selon une planification définie. Vous pouvez définir une date et une heure ultérieures pour déclencher ce déclencheur. En fonction de la fréquence, vous pouvez également spécifier des heures et des jours d’exécution de votre workflow. | 
| [**Request**](#request-trigger)  | Crée un point de terminaison pouvant être appelé pour votre application logique (également appelé déclencheur « manuel »). Par exemple, consultez [Appeler, déclencher ou imbriquer des workflows via des points de terminaison HTTP](../logic-apps/logic-apps-http-endpoint.md). | 
||| 

### <a name="managed-api-triggers"></a>Déclencheurs d’API managées

| Type de déclencheur | Description | 
|--------------|-------------| 
| [**ApiConnection**](#apiconnection-trigger) | Vérifie ou *interroge* un point de terminaison à l’aide d’[API managées par Microsoft](../connectors/apis-list.md). | 
| [**ApiConnectionWebhook**](#apiconnectionwebhook-trigger) | Crée un point de terminaison pouvant être appelé pour votre application logique en appelant des [API managées par Microsoft](../connectors/apis-list.md) pour s’abonner et annuler l’abonnement. | 
||| 

## <a name="triggers---detailed-reference"></a>Déclencheurs - Référence détaillée

<a name="apiconnection-trigger"></a>

### <a name="apiconnection-trigger"></a>Déclencheur APIConnection  

Ce déclencheur vérifie ou *interroge* un point de terminaison à l’aide d’[API managées par Microsoft](../connectors/apis-list.md) afin que les paramètres de ce déclencheur puissent différer en fonction du point de terminaison. De nombreuses sections de cette définition de déclencheur sont facultatives. Le comportement du déclencheur dépend de l’inclusion des sections.

```json
"<APIConnection_trigger_name>": {
   "type": "ApiConnection",
   "inputs": {
      "host": {
         "connection": {
            "name": "@parameters('$connections')['<connection-name>']['connectionId']"
         }
      },
      "method": "<method-type>",
      "path": "/<api-operation>",
      "retryPolicy": { "<retry-behavior>" },
      "queries": { "<query-parameters>" }
   },
   "recurrence": { 
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-runs-queue>
      }
   },
   "splitOn": "<splitOn-expression>",
   "operationOptions": "<operation-option>"
}
```

*Obligatoire*

| Valeur | Type | Description |
|-------|------|-------------|
| <*APIConnection_trigger_name*> | String | Nom du déclencheur |
| <*connection-name*> | String | Nom de la connexion à l’API managée utilisée par le workflow |
| <*method-type*> | String | Méthode HTTP pour communiquer avec l’API managée : « GET », « PUT », « POST », « PATCH », « DELETE » |
| <*api-operation*> | String | Opération d’API à appeler |
| <*time-unit*> | String | Unité de temps qui décrit la fréquence d’activation du déclencheur : "Second", "Minute", "Hour", "Day", "Week", "Month" |
| <*number-of-time-units*> | Integer | Valeur qui spécifie la fréquence d’activation du déclencheur, qui correspond au nombre d’unités de temps à attendre avant que le déclencheur soit activé à nouveau <p>Les intervalles minimaux et maximaux sont les suivants : <p>- Mois : 1-16 mois </br>Jour : 1-500 jours </br>- Heure : 1-12 000 heures </br>- Minute : 1-72 000 minutes </br>- Seconde : 1-9 999 999 secondes<p>Par exemple, si l’intervalle est défini sur 6 et la fréquence sur « Month », la périodicité est tous les six mois. |
||||

*Facultatif*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*retry-behavior*> | Objet JSON | Personnalise le comportement de nouvelle tentative pour les défaillances intermittentes, qui présentent le code d’état 408, 429 et 5XX, ainsi que les éventuelles exceptions de connectivité. Pour plus d’informations, consultez [Stratégies de relance](../logic-apps/logic-apps-exception-handling.md#retry-policies). | 
| <*query-parameters*> | Objet JSON | Paramètres de requête à inclure avec l’appel d’API. Par exemple, l’objet `"queries": { "api-version": "2018-01-01" }` ajoute `?api-version=2018-01-01` à l’appel. | 
| <*max-runs*> | Integer | Par défaut, les instances de workflow s’exécutent en même temps (simultanément ou en parallèle) jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer cette limite en définissant une nouvelle valeur <*count*>, consultez [Changer la concurrence du déclencheur](#change-trigger-concurrency). | 
| <*max-runs-queue*> | Integer | Lorsque votre application logique exécute déjà le nombre maximal d’instances, que vous pouvez modifier en fonction de la propriété `runtimeConfiguration.concurrency.runs`, les nouvelles exécutions sont placées dans cette file d’attente jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer la limite par défaut, consultez [Changer la limite d’exécutions en attente](#change-waiting-runs). | 
| <*splitOn-expression*> | String | Pour les déclencheurs qui retournent des tableaux, cette expression référence le tableau à utiliser pour que vous puissiez créer et exécuter une instance de workflow pour chaque élément du tableau, au lieu d’utiliser une boucle « for each ». <p>Par exemple, cette expression représente un élément dans le tableau retourné dans le contenu du corps du déclencheur : `@triggerbody()?['value']` |
| <*operation-option*> | String | Vous pouvez modifier le comportement par défaut en définissant la propriété `operationOptions`. Pour plus d’informations, consultez [Options d’opérations](#operation-options). |
||||

*Sorties*
 
| Élément | Type | Description |
|---------|------|-------------|
| headers | Objet JSON | En-têtes de la réponse |
| body | Objet JSON | Corps de la réponse |
| Code d’état | Integer | Code d’état de la réponse |
|||| 

*Exemple*

Cette définition de déclencheur vérifie tous les jours les e-mails dans la boîte de réception d’un compte professionnel ou scolaire :

```json
"When_a_new_email_arrives": {
   "type": "ApiConnection",
   "inputs": {
      "host": {
         "connection": {
            "name": "@parameters('$connections')['office365']['connectionId']"
         }
      },
      "method": "get",
      "path": "/Mail/OnNewEmail",
      "queries": {
          "fetchOnlyWithAttachment": false,
          "folderPath": "Inbox",
          "importance": "Any",
          "includeAttachments": false
      }
   },
   "recurrence": {
      "frequency": "Day",
      "interval": 1
   }
}
```

<a name="apiconnectionwebhook-trigger"></a>

### <a name="apiconnectionwebhook-trigger"></a>Déclencheur ApiConnectionWebhook

Ce déclencheur envoie une demande d’abonnement à un point de terminaison à l’aide d’une [API managée par Microsoft](../connectors/apis-list.md), fournit une *URL de rappel* où le point de terminaison peut envoyer une réponse, et attend que le point de terminaison réponde. Pour plus d’informations, consultez [Abonnements de point de terminaison](#subscribe-unsubscribe).

```json
"<ApiConnectionWebhook_trigger_name>": {
   "type": "ApiConnectionWebhook",
   "inputs": {
      "body": {
          "NotificationUrl": "@{listCallbackUrl()}"
      },
      "host": {
         "connection": {
            "name": "@parameters('$connections')['<connection-name>']['connectionId']"
         }
      },
      "retryPolicy": { "<retry-behavior>" },
      "queries": "<query-parameters>"
   },
   "runTimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-run-queue>
      }
   },
   "splitOn": "<splitOn-expression>",
   "operationOptions": "<operation-option>"
}
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*connection-name*> | String | Nom de la connexion à l’API managée utilisée par le workflow | 
| <*body-content*> | Objet JSON | Tout contenu de message à envoyer en tant que charge utile à l’API managée | 
||||

*Facultatif*

| Valeur | Type | Description |
|-------|------|-------------|
| <*retry-behavior*> | Objet JSON | Personnalise le comportement de nouvelle tentative pour les défaillances intermittentes, qui présentent le code d’état 408, 429 et 5XX, ainsi que les éventuelles exceptions de connectivité. Pour plus d’informations, consultez [Stratégies de relance](../logic-apps/logic-apps-exception-handling.md#retry-policies). |
| <*query-parameters*> | Objet JSON | Paramètres de requête à inclure avec l’appel d’API <p>Par exemple, l’objet `"queries": { "api-version": "2018-01-01" }` ajoute `?api-version=2018-01-01` à l’appel. |
| <*max-runs*> | Integer | Par défaut, les instances de workflow s’exécutent en même temps (simultanément ou en parallèle) jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer cette limite en définissant une nouvelle valeur <*count*>, consultez [Changer la concurrence du déclencheur](#change-trigger-concurrency). |
| <*max-runs-queue*> | Integer | Lorsque votre application logique exécute déjà le nombre maximal d’instances, que vous pouvez modifier en fonction de la propriété `runtimeConfiguration.concurrency.runs`, les nouvelles exécutions sont placées dans cette file d’attente jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer la limite par défaut, consultez [Changer la limite d’exécutions en attente](#change-waiting-runs). | 
| <*splitOn-expression*> | String | Pour les déclencheurs qui retournent des tableaux, cette expression référence le tableau à utiliser pour que vous puissiez créer et exécuter une instance de workflow pour chaque élément du tableau, au lieu d’utiliser une boucle « for each ». <p>Par exemple, cette expression représente un élément dans le tableau retourné dans le contenu du corps du déclencheur : `@triggerbody()?['value']` |
| <*operation-option*> | String | Vous pouvez modifier le comportement par défaut en définissant la propriété `operationOptions`. Pour plus d’informations, consultez [Options d’opérations](#operation-options). | 
|||| 

*Exemple*

Cette définition de déclencheur s’abonne à l’API Office 365 Outlook, fournit une URL de rappel au point de terminaison d’API, et attend que le point de terminaison réponde quand un nouvel e-mail arrive.

```json
"When_a_new_email_arrives_(webhook)": {
   "type": "ApiConnectionWebhook",
   "inputs": {
      "body": {
         "NotificationUrl": "@{listCallbackUrl()}" 
      },
      "host": {
         "connection": {
            "name": "@parameters('$connections')['office365']['connectionId']"
         }
      },
      "path": "/MailSubscription/$subscriptions",
      "queries": {
          "folderPath": "Inbox",
          "hasAttachment": "Any",
          "importance": "Any"
      }
   },
   "splitOn": "@triggerBody()?['value']"
}
```

<a name="http-trigger"></a>

### <a name="http-trigger"></a>Déclencheur HTTP

Ce déclencheur envoie une requête au point de terminaison HTTP ou HTTPS spécifié d’après la planification de périodicité définie. Le déclencheur vérifie ensuite la réponse pour déterminer si le workflow s’exécute. Pour plus d’informations, consultez [Appeler des points de terminaison de service via HTTP ou HTTPS à partir d’Azure Logic Apps](../connectors/connectors-native-http.md).

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "<method-type>",
      "uri": "<HTTP-or-HTTPS-endpoint-URL>",
      "headers": { "<header-content>" },
      "queries": "<query-parameters>",
      "body": "<body-content>",
      "authentication": { "<authentication-type-and-property-values>" },
      "retryPolicy": {
         "type": "<retry-behavior>"
      }
   },
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-runs-queue>
      }
   },
   "operationOptions": "<operation-option>"
}
```

*Obligatoire*

| Propriété | Valeur | Type | Description |
|----------|-------|------|-------------|
| `method` | <*method-type*> | String | Méthode à utiliser pour envoyer la requête sortante : « GET », « PUT », « POST », « PATCH » ou « DELETE » |
| `uri` | <*HTTP-or-HTTPS-endpoint-URL*> | String | URL du point de terminaison HTTP ou HTTPS où vous voulez envoyer la requête sortante. Taille de chaîne maximale : 2 Ko <p>Pour un service ou une ressource Azure, cette syntaxe URI inclut l'ID de la ressource et le chemin vers la ressource à laquelle vous voulez accéder. |
| `frequency` | <*time-unit*> | String | Unité de temps qui décrit la fréquence d’activation du déclencheur : "Second", "Minute", "Hour", "Day", "Week", "Month" |
| `interval` | <*number-of-time-units*> | Integer | Valeur qui spécifie la fréquence d’activation du déclencheur, qui correspond au nombre d’unités de temps à attendre avant que le déclencheur soit activé à nouveau <p>Les intervalles minimaux et maximaux sont les suivants : <p>- Mois : 1-16 mois </br>Jour : 1-500 jours </br>- Heure : 1-12 000 heures </br>- Minute : 1-72 000 minutes </br>- Seconde : 1-9 999 999 secondes<p>Par exemple, si l’intervalle est défini sur 6 et la fréquence sur « Month », la périodicité est tous les six mois. |
|||||

*Facultatif*

| Propriété | Valeur | Type | Description |
|----------|-------|------|-------------|
| `headers` | <*header-content*> | Objet JSON | Tous les en-têtes que vous devez inclure avec la demande <p>Par exemple, pour définir la langue et le type : <p>`"headers": { "Accept-Language": "en-us", "Content-Type": "application/json" }` |
| `queries` | <*query-parameters*> | Objet JSON | Tous les paramètres de requête que vous devez utiliser dans la requête <p>Par exemple, l’objet `"queries": { "api-version": "2018-01-01" }` ajoute `?api-version=2018-01-01` à la requête. |
| `body` | <*body-content*> | Objet JSON | Contenu du message à envoyer en tant que charge utile avec la requête |
| `authentication` | <*authentication-type-and-property-values*> | Objet JSON | Le modèle d'authentification que la requête utilise pour authentifier les requêtes sortantes. Pour plus d’informations, voir [Ajouter l’authentification aux appels sortants](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound). Au-delà de Scheduler, la propriété `authority` est prise en charge. Si vous ne spécifiez aucune valeur, la valeur par défaut est `https://management.azure.com/`, mais vous pouvez utiliser une autre valeur. |
| `retryPolicy` > `type` | <*retry-behavior*> | Objet JSON | Personnalise le comportement de nouvelle tentative pour les défaillances intermittentes, qui présentent le code d’état 408, 429 et 5XX, ainsi que les éventuelles exceptions de connectivité. Pour plus d’informations, consultez [Stratégies de relance](../logic-apps/logic-apps-exception-handling.md#retry-policies). |
| `runs` | <*max-runs*> | Integer | Par défaut, les instances de workflow s’exécutent en même temps (simultanément ou en parallèle) jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer cette limite en définissant une nouvelle valeur <*count*>, consultez [Changer la concurrence du déclencheur](#change-trigger-concurrency). |
| `maximumWaitingRuns` | <*max-runs-queue*> | Integer | Lorsque votre application logique exécute déjà le nombre maximal d’instances, que vous pouvez modifier en fonction de la propriété `runtimeConfiguration.concurrency.runs`, les nouvelles exécutions sont placées dans cette file d’attente jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer la limite par défaut, consultez [Changer la limite d’exécutions en attente](#change-waiting-runs). |
| `operationOptions` | <*operation-option*> | String | Vous pouvez modifier le comportement par défaut en définissant la propriété `operationOptions`. Pour plus d’informations, consultez [Options d’opérations](#operation-options). |
|||||

*Sorties*

| Élément | Type | Description |
|---------|------|-------------|
| `headers` | Objet JSON | En-têtes de la réponse |
| `body` | Objet JSON | Corps de la réponse |
| `status code` | Integer | Code d’état de la réponse |
||||

*Conditions requises pour les requêtes entrantes*

Pour fonctionner correctement avec votre application logique, le point de terminaison doit être conforme à un modèle ou contrat de déclencheur spécifique, et reconnaître les propriétés de réponse suivantes :

| Propriété | Obligatoire | Description |
|----------|----------|-------------|
| Code d’état | Oui | Le code d’état « 200 OK » démarre une exécution. Les autres codes d’état ne démarrent pas d’exécution. |
| En-tête Retry-after | Non | Nombre de secondes au bout duquel l’application logique interroge à nouveau le point de terminaison |
| En-tête Location | Non | URL à appeler lors du prochain intervalle d’interrogation. Si aucune valeur n’est spécifiée, l’URL d’origine est utilisée. |
|||| 

*Exemples de comportements pour différentes requêtes*

| Code d’état | Retry after (Réessayer après) | Comportement | 
|-------------|-------------|----------|
| 200 | {aucune} | Exécutez le workflow et vérifiez de nouveau pour obtenir plus de données après la périodicité définie. | 
| 200 | 10 secondes | Exécutez le workflow et vérifiez de nouveau pour obtenir plus de données après 10 secondes. |  
| 202 | 60 secondes | Ne pas déclencher le workflow. La prochaine tentative se produit dans une minute conformément à la périodicité définie. Si la périodicité définie est inférieure à une minute, l’en-tête retry-after est prioritaire. Dans le cas contraire, la périodicité définie est utilisée. | 
| 400 | {aucune} | Requête incorrecte, ne pas exécuter le workflow. Si aucune valeur `retryPolicy` n’est définie, la stratégie par défaut est utilisée. Une fois que le nombre de nouvelles tentatives a été atteint, le déclencheur vérifie de nouveau pour obtenir plus de données après la périodicité définie. | 
| 500 | {aucune}| Erreur de serveur, ne pas exécuter le workflow. Si aucune valeur `retryPolicy` n’est définie, la stratégie par défaut est utilisée. Une fois que le nombre de nouvelles tentatives a été atteint, le déclencheur vérifie de nouveau pour obtenir plus de données après la périodicité définie. | 
|||| 

<a name="http-webhook-trigger"></a>

### <a name="httpwebhook-trigger"></a>Déclencheur HTTPWebhook  

Ce déclencheur permet à votre application logique de pouvoir être appelée en créant un point de terminaison capable d’inscrire un abonnement en appelant l’URL de point de terminaison spécifiée. Quand vous créez ce déclencheur dans votre workflow, une requête sortante effectue l’appel pour inscrire l’abonnement. De cette façon, le déclencheur peut commencer à écouter les événements. Quand une opération rend ce déclencheur non valide, une requête sortante effectue automatiquement l’appel afin d’annuler l’abonnement. Pour plus d’informations, consultez [Abonnements de point de terminaison](#subscribe-unsubscribe).

Vous pouvez également spécifier des [limites asynchrones](#asynchronous-limits) sur un déclencheur **HTTPWebhook**. Le comportement du déclencheur varie en fonction des sections que vous incluez ou omettez.

```json
"HTTP_Webhook": {
   "type": "HttpWebhook",
   "inputs": {
      "subscribe": {
         "method": "<method-type>",
         "uri": "<endpoint-subscribe-URL>",
         "headers": { "<header-content>" },
         "body": "<body-content>",
         "authentication": { "<authentication-type>" },
         "retryPolicy": { "<retry-behavior>" }
         },
      },
      "unsubscribe": {
         "method": "<method-type>",
         "url": "<endpoint-unsubscribe-URL>",
         "headers": { "<header-content>" },
         "body": "<body-content>",
         "authentication": { "<authentication-type>" }
      }
   },
   "runTimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-runs-queue>
      }
   },
   "operationOptions": "<operation-option>"
}
```

Certaines valeurs, telles que < *-method-type*>, sont disponibles pour les objets `"subscribe"` et `"unsubscribe"`.

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*method-type*> | String | Méthode HTTP utilisée pour la requête d’abonnement : « GET », « PUT », « POST », « PATCH » ou « DELETE » | 
| <*endpoint-subscribe-URL*> | String | URL du point de terminaison où envoyer la requête d’abonnement | 
|||| 

*Facultatif*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*method-type*> | String | Méthode HTTP utilisée pour la requête d’annulation : « GET », « PUT », « POST », « PATCH » ou « DELETE » | 
| <*endpoint-unsubscribe-URL*> | String | URL du point de terminaison où envoyer la requête d’annulation | 
| <*body-content*> | String | Tout contenu de message à envoyer dans la requête d’abonnement ou d’annulation | 
| <*authentication-type*> | Objet JSON | Le modèle d'authentification que la requête utilise pour authentifier les requêtes sortantes. Pour plus d’informations, voir [Ajouter l’authentification aux appels sortants](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound). |
| <*retry-behavior*> | Objet JSON | Personnalise le comportement de nouvelle tentative pour les défaillances intermittentes, qui présentent le code d’état 408, 429 et 5XX, ainsi que les éventuelles exceptions de connectivité. Pour plus d’informations, consultez [Stratégies de relance](../logic-apps/logic-apps-exception-handling.md#retry-policies). | 
| <*max-runs*> | Integer | Par défaut, les instances de workflow s’exécutent toutes en même temps (simultanément ou en parallèle) jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer cette limite en définissant une nouvelle valeur <*count*>, consultez [Changer la concurrence du déclencheur](#change-trigger-concurrency). | 
| <*max-runs-queue*> | Integer | Lorsque votre application logique exécute déjà le nombre maximal d’instances, que vous pouvez modifier en fonction de la propriété `runtimeConfiguration.concurrency.runs`, les nouvelles exécutions sont placées dans cette file d’attente jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer la limite par défaut, consultez [Changer la limite d’exécutions en attente](#change-waiting-runs). | 
| <*operation-option*> | String | Vous pouvez modifier le comportement par défaut en définissant la propriété `operationOptions`. Pour plus d’informations, consultez [Options d’opérations](#operation-options). | 
|||| 

*Sorties* 

| Élément | Type | Description |
|---------|------|-------------| 
| headers | Objet JSON | En-têtes de la réponse | 
| body | Objet JSON | Corps de la réponse | 
| Code d’état | Integer | Code d’état de la réponse | 
|||| 

*Exemple*

Ce déclencheur crée un abonnement au point de terminaison spécifié, fournit une URL de rappel unique, et attend des articles sur la technologie venant d’être publiés.

```json
"HTTP_Webhook": {
   "type": "HttpWebhook",
   "inputs": {
      "subscribe": {
         "method": "POST",
         "uri": "https://pubsubhubbub.appspot.com/subscribe",
         "body": {
            "hub.callback": "@{listCallbackUrl()}",
            "hub.mode": "subscribe",
            "hub.topic": "https://pubsubhubbub.appspot.com/articleCategories/technology"
         },
      },
      "unsubscribe": {
         "method": "POST",
         "url": "https://pubsubhubbub.appspot.com/subscribe",
         "body": {
            "hub.callback": "@{workflow().endpoint}@{listCallbackUrl()}",
            "hub.mode": "unsubscribe",
            "hub.topic": "https://pubsubhubbub.appspot.com/articleCategories/technology"
         }
      }
   }
}
```

<a name="recurrence-trigger"></a>

### <a name="recurrence-trigger"></a>Déclencheur recurrence  

Ce déclencheur s’active en fonction de la planification de périodicité que vous spécifiez. Il offre un moyen simple de créer un workflow à l’exécution régulière.

```json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
      "startTime": "<start-date-time-with-format-YYYY-MM-DDThh:mm:ss>",
      "timeZone": "<time-zone>",
      "schedule": {
         // Applies only when frequency is Day or Week. Separate values with commas.
         "hours": [ <one-or-more-hour-marks> ], 
         // Applies only when frequency is Day or Week. Separate values with commas.
         "minutes": [ <one-or-more-minute-marks> ], 
         // Applies only when frequency is Week. Separate values with commas.
         "weekDays": [ "Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday" ] 
      }
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-runs-queue>
      }
   },
   "operationOptions": "<operation-option>"
}
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*time-unit*> | String | Unité de temps qui décrit la fréquence d’activation du déclencheur : "Second", "Minute", "Hour", "Day", "Week", "Month" | 
| <*number-of-time-units*> | Integer | Valeur qui spécifie la fréquence d’activation du déclencheur, qui correspond au nombre d’unités de temps à attendre avant que le déclencheur soit activé à nouveau <p>Les intervalles minimaux et maximaux sont les suivants : <p>- Mois : 1-16 mois </br>Jour : 1-500 jours </br>- Heure : 1-12 000 heures </br>- Minute : 1-72 000 minutes </br>- Seconde : 1-9 999 999 secondes<p>Par exemple, si l’intervalle est défini sur 6 et la fréquence sur « Month », la périodicité est tous les six mois. | 
|||| 

*Facultatif*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*start-date-time-with-format-YYYY-MM-DDThh:mm:ss*> | String | Date et heure de début au format suivant : <p>AAAA-MM-JJThh:mm:ss si vous spécifiez un fuseau horaire <p>-ou- <p>AAAA-MM-JJThh:mm:ssZ si vous ne spécifiez pas de fuseau horaire <p>Par exemple, si vous choisissez le 18 septembre 2017 à 14h, spécifiez « 2017-09-18T14:00:00 » et spécifiez un fuseau horaire tel que « Pacific Standard Time » (Heure standard du Pacifique), ou spécifiez « 2017-09-18T14:00:00Z » sans fuseau horaire. <p>**Remarque :** Cette heure de début ne peut pas dépasser 49 ans dans le futur, et doit être conforme à la [spécification de date/heure ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) au [format de date/heure UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time), mais sans [décalage UTC](https://en.wikipedia.org/wiki/UTC_offset). Si vous ne spécifiez pas de fuseau horaire, vous devez ajouter la lettre « Z » à la fin, sans espace. Ce « Z » fait référence au [temps nautique](https://en.wikipedia.org/wiki/Nautical_time) équivalent. <p>Pour les planifications simples, l’heure de début est la première occurrence, tandis que pour les planifications complexes, le déclencheur ne s’active pas avant l’heure de début. Pour plus d’informations sur les dates et heures de début, consultez [Create and schedule regularly running tasks](../connectors/connectors-native-recurrence.md) (Créer et planifier des tâches à exécution régulière). | 
| <*time-zone*> | String | S’applique uniquement quand vous spécifiez une heure de début, car ce déclencheur n’accepte pas le [décalage UTC](https://en.wikipedia.org/wiki/UTC_offset). Spécifiez le fuseau horaire à appliquer. | 
| <*one-or-more-hour-marks*> | Entier ou tableau d’entiers | Si vous spécifiez « Jour » ou « Semaine » pour `frequency`, vous pouvez spécifier un ou plusieurs entiers compris entre 0 et 23, séparés par des virgules, pour les heures de la journée durant lesquelles exécuter le workflow. <p>Par exemple, si vous spécifiez « 10 », « 12 » et « 14 », vous obtenez 10h00, 12h00 et 14h00 comme marques horaires. | 
| <*one-or-more-minute-marks*> | Entier ou tableau d’entiers | Si vous spécifiez « Jour » ou « Semaine » pour `frequency`, vous pouvez spécifier un ou plusieurs entiers compris entre 0 et 59, séparés par des virgules, pour les minutes de l’heure durant lesquelles exécuter le workflow. <p>Par exemple, vous pouvez spécifier « 30 » pour les minutes et à l’aide de l’exemple précédent des heures de la journée, vous obtenez 10h30, 12h30 et 14h30. | 
| weekDays | Chaîne ou tableau de chaînes | Si vous spécifiez « Semaine » pour `frequency`, vous pouvez spécifier un ou plusieurs jours, séparés par des virgules, pour exécuter le workflow : « Monday », « Tuesday », « Wednesday », « Thursday », « Friday », « Saturday » et « Sunday » | 
| <*max-runs*> | Integer | Par défaut, les instances de workflow s’exécutent toutes en même temps (simultanément ou en parallèle) jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer cette limite en définissant une nouvelle valeur <*count*>, consultez [Changer la concurrence du déclencheur](#change-trigger-concurrency). | 
| <*max-runs-queue*> | Integer | Lorsque votre application logique exécute déjà le nombre maximal d’instances, que vous pouvez modifier en fonction de la propriété `runtimeConfiguration.concurrency.runs`, les nouvelles exécutions sont placées dans cette file d’attente jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer la limite par défaut, consultez [Changer la limite d’exécutions en attente](#change-waiting-runs). | 
| <*operation-option*> | String | Vous pouvez modifier le comportement par défaut en définissant la propriété `operationOptions`. Pour plus d’informations, consultez [Options d’opérations](#operation-options). | 
|||| 

*Exemple 1*

Ce déclencheur de périodicité de base s’active tous les jours :

```json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "Day",
      "interval": 1
   }
}
```

*Exemple 2*

Vous pouvez définir une date et une heure de début pour activer le déclencheur. Ce déclencheur de périodicité démarre à la date spécifiée, puis s’active tous les jours :

```json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "Day",
      "interval": 1,
      "startTime": "2017-09-18T00:00:00Z"
   }
}
```

*Exemple 3*

Ce déclencheur de périodicité commence le 9 septembre 2017 à 14h00, et il s’active tous les lundis à 10h30, 12h30 et 14h30 Heure standard du Pacifique :

``` json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "Week",
      "interval": 1,
      "schedule": {
         "hours": [ 10, 12, 14 ],
         "minutes": [ 30 ],
         "weekDays": [ "Monday" ]
      },
      "startTime": "2017-09-07T14:00:00",
      "timeZone": "Pacific Standard Time"
   }
}
```

Pour plus d’informations et pour obtenir des exemples pour ce déclencheur, consultez [Créer et planifier l’exécution régulière de tâches](../connectors/connectors-native-recurrence.md).

<a name="request-trigger"></a>

### <a name="request-trigger"></a>Déclencheur de requête

Ce déclencheur fait en sorte que votre application logique puisse être appelée en créant un point de terminaison qui peut accepter des requêtes entrantes. Pour ce déclencheur, fournit un schéma JSON qui décrit et valide la charge utile ou les entrées que le déclencheur reçoit de la requête entrante. Le schéma facilite également le référencement des propriétés de déclencheur à partir d’actions ultérieures dans le workflow.

Pour appeler ce déclencheur, vous devez utiliser l’API `listCallbackUrl`, qui est décrite dans l’[API REST de service de workflow](/rest/api/logic/workflows). Pour découvrir comment utiliser ce déclencheur en tant que point de terminaison HTTP, consultez [Appeler, déclencher ou imbriquer des workflows via des points de terminaison HTTP](../logic-apps/logic-apps-http-endpoint.md).

```json
"manual": {
   "type": "Request",
   "kind": "Http",
   "inputs": {
      "method": "<method-type>",
      "relativePath": "<relative-path-for-accepted-parameter>",
      "schema": {
         "type": "object",
         "properties": { 
            "<property-name>": {
               "type": "<property-type>"
            }
         },
         "required": [ "<required-properties>" ]
      }
   },
   "runTimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-run-queue>
      },
   },
   "operationOptions": "<operation-option>"
}
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*property-name*> | String | Nom d’une propriété dans le schéma JSON qui décrit la charge utile | 
| <*property-type*> | String | Type de la propriété | 
|||| 

*Facultatif*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*method-type*> | String | Méthode que les requêtes entrantes doivent utiliser pour appeler votre application logique : « GET », « PUT », « POST », « PATCH », « DELETE » |
| <*relative-path-for-accepted-parameter*> | String | Chemin relatif pour le paramètre que l’URL de votre point de terminaison peut accepter | 
| <*required-properties*> | Array | Une ou plusieurs propriétés qui nécessitent des valeurs. | 
| <*max-runs*> | Integer | Par défaut, les instances de workflow s’exécutent toutes en même temps (simultanément ou en parallèle) jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer cette limite en définissant une nouvelle valeur <*count*>, consultez [Changer la concurrence du déclencheur](#change-trigger-concurrency). | 
| <*max-runs-queue*> | Integer | Lorsque votre application logique exécute déjà le nombre maximal d’instances, que vous pouvez modifier en fonction de la propriété `runtimeConfiguration.concurrency.runs`, les nouvelles exécutions sont placées dans cette file d’attente jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer la limite par défaut, consultez [Changer la limite d’exécutions en attente](#change-waiting-runs). | 
| <*operation-option*> | String | Vous pouvez modifier le comportement par défaut en définissant la propriété `operationOptions`. Pour plus d’informations, consultez [Options d’opérations](#operation-options). | 
|||| 

*Exemple*

Ce déclencheur spécifie qu’une requête entrante doit utiliser la méthode HTTP POST pour appeler le déclencheur, et inclut un schéma qui valide l’entrée de la requête entrante :

```json
"manual": {
   "type": "Request",
   "kind": "Http",
   "inputs": {
      "method": "POST",
      "schema": {
         "type": "object",
         "properties": {
            "customerName": {
               "type": "String"
            },
            "customerAddress": { 
               "type": "Object",
               "properties": {
                  "streetAddress": {
                     "type": "string"
                  },
                  "city": {
                     "type": "string"
                  }
               }
            }
         }
      }
   }
}
```

<a name="trigger-conditions"></a>

## <a name="trigger-conditions"></a>Conditions du déclencheur

Pour les déclencheurs uniquement, vous pouvez inclure un tableau qui contient une ou plusieurs expressions pour des conditions qui déterminent si le workflow doit s’exécuter. Pour ajouter la propriété `conditions` à un déclencheur dans votre flux de travail, ouvrez l’application logique dans l’éditeur en mode code.

Par exemple, vous pouvez spécifier qu’un déclencheur est activé uniquement quand un site web retourne une erreur interne du serveur référençant le code d’état du déclencheur dans la propriété `conditions` :

```json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "Hour",
      "interval": 1
   },
   "conditions": [ {
      "expression": "@equals(triggers().code, 'InternalServerError')"
   } ]
}
```

Par défaut, un déclencheur s’active uniquement après avoir reçu une réponse « 200 OK ». Quand une expression référence le code d’état d’un déclencheur, le comportement par défaut du déclencheur est remplacé. Par conséquent, si vous souhaitez que le déclencheur s’active pour plusieurs codes d’état, par exemple les codes d’état « 200 » et « 201 », vous devez inclure cette expression comme condition :

`@or(equals(triggers().code, 200),equals(triggers().code, 201))` 

<a name="split-on-debatch"></a>

## <a name="trigger-multiple-runs"></a>Déclencher plusieurs exécutions

Si votre déclencheur renvoie un tableau à traiter pour votre application logique, il arrive qu’une boucle « for each » prenne trop de temps pour traiter chaque élément du tableau. Au lieu de cela, vous pouvez utiliser la propriété **SplitOn** dans votre déclencheur pour *décomposer* le tableau. La décomposition sépare les éléments de tableau et démarre une nouvelle instance de flux de travail qui s’exécute pour chaque élément du tableau. Par exemple, cette approche est utile lorsque vous souhaitez interroger un point de terminaison qui peut renvoyer plusieurs nouveaux éléments entre les intervalles d’interrogation. Pour connaître le nombre maximal d’éléments de tableau que **SplitOn** peut traiter en une seule exécution d’application logique, consultez [Limites et configurations](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). 

> [!NOTE]
> Vous ne pouvez pas utiliser **SplitOn** avec un modèle de réponse synchrone. Les workflows qui utilisent **SplitOn** et incluent une réponse action s’exécutent de façon asynchrone et envoient immédiatement une réponse `202 ACCEPTED`.
>
> Lorsque la concurrence du déclencheur est activée, la [limite SplitOn](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits) est considérablement réduite. Si le nombre d'éléments dépasse cette limite, la fonction SplitOn est désactivée.
 
Si le fichier Swagger de votre déclencheur décrit une charge utile sous forme de tableau, la propriété **SplitOn** est automatiquement ajoutée à votre déclencheur. Sinon, ajoutez cette propriété à l’intérieur de la charge utile de la réponse qui contient le tableau à décomposer.

*Exemple*

Supposez que vous avez une API qui retourne la réponse suivante : 
  
```json
{
   "Status": "Succeeded",
   "Rows": [ 
      { 
         "id": 938109380,
         "name": "customer-name-one"
      },
      {
         "id": 938109381,
         "name": "customer-name-two"
      }
   ]
}
```

Comme votre application logique a uniquement besoin du contenu du tableau dans `Rows`, vous pouvez créer votre déclencheur comme dans cet exemple :

``` json
"HTTP_Debatch": {
   "type": "Http",
    "inputs": {
        "uri": "https://mydomain.com/myAPI",
        "method": "GET"
    },
   "recurrence": {
      "frequency": "Second",
      "interval": 1
    },
    "splitOn": "@triggerBody()?.Rows"
}
```

> [!NOTE]
> Si vous utilisez la commande `SplitOn`, vous ne pouvez pas obtenir les propriétés qui ne se trouvent pas dans le tableau. Pour cet exemple, vous ne pouvez donc pas obtenir la propriété `status` dans la réponse renvoyée par l’API.
> 
> Dans cet exemple, nous utilisons l’opérateur `?` afin d’éviter un échec si la propriété `Rows` n’existe pas.

Votre définition de workflow peut désormais utiliser `@triggerBody().name` pour obtenir les valeurs `name`, qui sont `"customer-name-one"` à partir de la première exécution et `"customer-name-two"` à partir de la deuxième exécution. Par conséquent, les sorties du déclencheur se présentent comme dans les exemples suivants :

```json
{
   "body": {
      "id": 938109380,
      "name": "customer-name-one"
   }
}
```

```json
{
   "body": {
      "id": 938109381,
      "name": "customer-name-two"
   }
}
```

<a name="actions-overview"></a>

## <a name="actions-overview"></a>Vue d’ensemble des actions

Azure Logic Apps fournit différents types d’actions, chacun avec des entrées différentes qui définissent le comportement unique d’une action. Les actions ont les principaux éléments suivants, bien que certains soient facultatifs :

```json
"<action-name>": {
   "type": "<action-type>",
   "inputs": { 
      "<input-name>": { "<input-value>" },
      "retryPolicy": "<retry-behavior>" 
   },
   "runAfter": { "<previous-trigger-or-action-status>" },
   "runtimeConfiguration": { "<runtime-config-options>" },
   "operationOptions": "<operation-option>"
},
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------|
| <*action-name*> | String | Nom de l’action | 
| <*action-type*> | String | Type d’action, par exemple « Http » ou « ApiConnection »| 
| <*input-name*> | String | Nom d’une entrée qui définit le comportement de l’action | 
| <*input-value*> | Divers | Valeur d’entrée, qui peut être une chaîne, un entier, nu objet JSON, et ainsi de suite | 
| <*previous-trigger-or-action-status*> | Objet JSON | Nom et état résultant du déclencheur ou de l’action qui doit s’exécuter juste avant que cette action puisse s’exécuter | 
|||| 

*Facultatif*

| Valeur | Type | Description | 
|-------|------|-------------|
| <*retry-behavior*> | Objet JSON | Personnalise le comportement de nouvelle tentative pour les défaillances intermittentes, qui présentent le code d’état 408, 429 et 5XX, ainsi que les éventuelles exceptions de connectivité. Pour plus d’informations, consultez Stratégies de relance. | 
| <*runtime-config-options*> | Objet JSON | Pour certaines actions, vous pouvez changer le comportement de l’action au moment de l’exécution en définissant des propriétés `runtimeConfiguration`. Pour plus d’informations, consultez [Paramètres de configuration d’exécution](#runtime-config-options). | 
| <*operation-option*> | String | Pour certaines actions, vous pouvez changer le comportement par défaut en définissant la propriété `operationOptions`. Pour plus d’informations, consultez [Options d’opérations](#operation-options). | 
|||| 

## <a name="action-types-list"></a>Liste des types d’actions

Voici quelques types d’actions couramment utilisés : 

* [Types d’actions intégrés](#built-in-actions), tels que ces exemples et bien plus encore : 

  * [**HTTP**](#http-action) pour appeler des points de terminaison par le biais du protocole HTTP ou HTTPS

  * [**Response**](#response-action) pour répondre aux requêtes

  * [**Exécuter du code JavaScript**](#run-javascript-code) pour l’exécution d’extraits de code JavaScript

  * [**Function**](#function-action) pour appeler Azure Functions

  * Actions d’opérations de données telles que [**Join**](#join-action), [**Compose**](#compose-action), [**Table**](#table-action), [**Select**](#select-action) et autres, qui créent ou transforment des données à partir de diverses entrées

  * [**Workflow**](#workflow-action) pour appeler un autre workflow d’application logique

* [Types d’actions d’API managées](#managed-api-actions) comme [**ApiConnection**](#apiconnection-action) et [**ApiConnectionWebHook**](#apiconnectionwebhook-action), qui appellent différents connecteurs et API managés par Microsoft, par exemple Azure Service Bus, Office 365 Outlook, Power BI, Stockage Blob Azure, OneDrive, GitHub et bien plus encore

* [Types d’actions de workflows de contrôle](#control-workflow-actions) comme [**If**](#if-action), [**Foreach**](#foreach-action), [**Switch**](#switch-action), [**Scope**](#scope-action) et [**Until**](#until-action), qui contiennent d’autres actions et vous aident à organiser l’exécution du workflow

<a name="built-in-actions"></a>

### <a name="built-in-actions"></a>Actions intégrées

| Type d'action | Description | 
|-------------|-------------| 
| [**Composer**](#compose-action) | Crée une sortie unique à partir des entrées, qui peuvent avoir différents types. | 
| [**Exécuter du code JavaScript**](#run-javascript-code) | Exécutez des extraits de code JavaScript qui répondent à des critères spécifiques. Pour connaître les exigences relatives au code et d’autres informations, voir [Ajouter et exécuter des extraits de code avec du code inclus dans Azure Logic Apps](../logic-apps/logic-apps-add-run-inline-code.md). |
| [**Function**](#function-action) | Appelle une fonction Azure. | 
| [**HTTP**](#http-action) | Appelle un point de terminaison HTTP. | 
| [**Joindre**](#join-action) | Crée une chaîne à partir de tous les éléments d’un tableau, et sépare ces éléments avec un caractère délimiteur spécifié. | 
| [**Analyser JSON**](#parse-json-action) | Crée des jetons conviviaux à partir de propriétés dans du contenu JSON. Vous pouvez ensuite référencer ces propriétés en incluant les jetons dans votre application logique. | 
| [**Query**](#query-action) | Crée un tableau à partir des éléments d’un autre tableau en fonction d’une condition ou d’un filtre. | 
| [**Response**](#response-action) | Crée une réponse à une requête ou un appel entrant. | 
| [**Sélectionner**](#select-action) | Crée un tableau avec des objets JSON en transformant les éléments d’un autre tableau en fonction de la carte spécifiée. | 
| [**Table**](#table-action) | Crée une table CSV ou HTML à partir d’un tableau. | 
| [**Terminate**](#terminate-action) | Arrête un workflow en cours d’exécution. | 
| [**Wait**](#wait-action) | Interrompt votre workflow pour une durée spécifiée ou jusqu’à la date et l’heure spécifiées. | 
| [**Workflow**](#workflow-action) | Imbrique un workflow à l’intérieur d’un autre workflow. | 
||| 

<a name="managed-api-actions"></a>

### <a name="managed-api-actions"></a>Actions d’API managées

| Type d'action | Description | 
|-------------|-------------|  
| [**ApiConnection**](#apiconnection-action) | Appelle un point de terminaison HTTP à l’aide d’une [API managée par Microsoft](../connectors/apis-list.md). | 
| [**ApiConnectionWebhook**](#apiconnectionwebhook-action) | Fonctionne comme HTTPWebhook, mais utilise une [API managée par Microsoft](../connectors/apis-list.md). | 
||| 

<a name="control-workflow-actions"></a>

### <a name="control-workflow-actions"></a>Actions de workflows de contrôle

Ces actions vous permettent de contrôler l’exécution du workflow et d’inclure d’autres actions. À partir de l’extérieur d’une action de workflow de contrôle, vous pouvez directement référencer des actions à l’intérieur de cette action de workflow de contrôle. Par exemple, si vous avez une action `Http` à l’intérieur d’une étendue, vous pouvez référencer l’expression `@body('Http')` à partir de n’importe où dans le workflow. Toutefois, les actions qui existent à l’intérieur d’une action de workflow de contrôle ne peuvent « s’exécuter qu’après d’autres actions » qui se trouvent dans la même structure de workflow de contrôle.

| Type d'action | Description | 
|-------------|-------------| 
| [**ForEach**](#foreach-action) | Exécuter les mêmes actions dans une boucle pour chaque élément d’un tableau. | 
| [**If**](#if-action) | Exécuter des actions selon que la condition spécifiée est true ou false. | 
| [**Scope**](#scope-action) | Exécuter des actions en fonction de l’état de groupe d’un ensemble d’actions. | 
| [**Switch**](#switch-action) | Exécuter des actions organisées en cas quand les valeurs d’expressions, d’objets ou de jetons correspondent aux valeurs spécifiées par chaque cas. | 
| [**Until**](#until-action) | Exécuter des actions dans une boucle jusqu’à ce que la condition spécifiée ait la valeur true. | 
|||  

## <a name="actions---detailed-reference"></a>Actions - Référence détaillée

<a name="apiconnection-action"></a>

### <a name="apiconnection-action"></a>Action APIConnection

Cette action envoie une requête HTTP à une [API managée par Microsoft](../connectors/apis-list.md) et exige des informations sur l’API et les paramètres, ainsi qu’une référence à une connexion valide. 

``` json
"<action-name>": {
   "type": "ApiConnection",
   "inputs": {
      "host": {
         "connection": {
            "name": "@parameters('$connections')['<api-name>']['connectionId']"
         },
         "<other-action-specific-input-properties>"        
      },
      "method": "<method-type>",
      "path": "/<api-operation>",
      "retryPolicy": "<retry-behavior>",
      "queries": { "<query-parameters>" },
      "<other-action-specific-properties>"
    },
    "runAfter": {}
}
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*action-name*> | String | Nom de l’action fournie par le connecteur | 
| <*api-name*> | String | Nom de l’API managée par Microsoft qui est utilisée pour la connexion | 
| <*method-type*> | String | Méthode HTTP d’appel de l’API : « GET », « PUT », « POST », « PATCH » ou « DELETE » | 
| <*api-operation*> | String | Opération d’API à appeler | 
|||| 

*Facultatif*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*other-action-specific-input-properties*> | Objet JSON | Toutes autres propriétés d’entrée qui s’appliquent à cette action spécifique | 
| <*retry-behavior*> | Objet JSON | Personnalise le comportement de nouvelle tentative pour les défaillances intermittentes, qui présentent le code d’état 408, 429 et 5XX, ainsi que les éventuelles exceptions de connectivité. Pour plus d’informations, consultez [Stratégies de relance](../logic-apps/logic-apps-exception-handling.md#retry-policies). | 
| <*query-parameters*> | Objet JSON | Paramètres de requête à inclure avec l’appel d’API. <p>Par exemple, l’objet `"queries": { "api-version": "2018-01-01" }` ajoute `?api-version=2018-01-01` à l’appel. | 
| <*other-action-specific-properties*> | Objet JSON | Toutes autres propriétés qui s’appliquent à cette action spécifique | 
|||| 

*Exemple*

Cette définition décrit l’action **Envoyer un e-mail** pour le connecteur Office 365 Outlook, qui est une API managée par Microsoft : 

```json
"Send_an_email": {
   "type": "ApiConnection",
   "inputs": {
      "body": {
         "Body": "Thank you for your membership!",
         "Subject": "Hello and welcome!",
         "To": "Sophie.Owen@contoso.com"
      },
      "host": {
         "connection": {
            "name": "@parameters('$connections')['office365']['connectionId']"
         }
      },
      "method": "POST",
      "path": "/Mail"
    },
    "runAfter": {}
}
```

<a name="apiconnection-webhook-action"></a>

### <a name="apiconnectionwebhook-action"></a>Action APIConnectionWebhook

Cette action envoie une demande d’abonnement à un point de terminaison par le biais du protocole HTTP à l’aide d’une [API managée par Microsoft](../connectors/apis-list.md), fournit une *URL de rappel* où le point de terminaison peut envoyer une réponse, et attend que le point de terminaison réponde. Pour plus d’informations, consultez [Abonnements de point de terminaison](#subscribe-unsubscribe).

```json
"<action-name>": {
   "type": "ApiConnectionWebhook",
   "inputs": {
      "subscribe": {
         "method": "<method-type>",
         "uri": "<api-subscribe-URL>",
         "headers": { "<header-content>" },
         "body": "<body-content>",
         "authentication": { "<authentication-type>" },
         "retryPolicy": "<retry-behavior>",
         "queries": { "<query-parameters>" },
         "<other-action-specific-input-properties>"
      },
      "unsubscribe": {
         "method": "<method-type>",
         "uri": "<api-unsubscribe-URL>",
         "headers": { "<header-content>" },
         "body": "<body-content>",
         "authentication": { "<authentication-type>" },
         "<other-action-specific-properties>"
      },
   },
   "runAfter": {}
}
```

Certaines valeurs, telles que < *-method-type*>, sont disponibles pour les objets `"subscribe"` et `"unsubscribe"`.

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*action-name*> | String | Nom de l’action fournie par le connecteur | 
| <*method-type*> | String | Méthode HTTP à utiliser pour s’abonner ou annuler l’abonnement à un point de terminaison : « GET », « PUT », « POST », « PATCH » ou « DELETE » | 
| <*api-subscribe-URL*> | String | URI à utiliser pour s’abonner à l’API | 
|||| 

*Facultatif*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*api-unsubscribe-URL*> | String | URI à utiliser pour annuler l’abonnement à l’API | 
| <*header-content*> | Objet JSON | En-têtes à envoyer dans la requête <p>Par exemple, pour définir la langue et le type d’une requête : <p>`"headers": { "Accept-Language": "en-us", "Content-Type": "application/json" }` |
| <*body-content*> | Objet JSON | Tout contenu de message à envoyer dans la requête | 
| <*authentication-type*> | Objet JSON | Le modèle d'authentification que la requête utilise pour authentifier les requêtes sortantes. Pour plus d’informations, voir [Ajouter l’authentification aux appels sortants](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound). |
| <*retry-behavior*> | Objet JSON | Personnalise le comportement de nouvelle tentative pour les défaillances intermittentes, qui présentent le code d’état 408, 429 et 5XX, ainsi que les éventuelles exceptions de connectivité. Pour plus d’informations, consultez [Stratégies de relance](../logic-apps/logic-apps-exception-handling.md#retry-policies). | 
| <*query-parameters*> | Objet JSON | Paramètres de requête à inclure avec l’appel d’API <p>Par exemple, l’objet `"queries": { "api-version": "2018-01-01" }` ajoute `?api-version=2018-01-01` à l’appel. | 
| <*other-action-specific-input-properties*> | Objet JSON | Toutes autres propriétés d’entrée qui s’appliquent à cette action spécifique | 
| <*other-action-specific-properties*> | Objet JSON | Toutes autres propriétés qui s’appliquent à cette action spécifique | 
|||| 

Vous pouvez aussi spécifier les limites sur une action **ApiConnectionWebhook** de la même manière que les [limites asynchrones HTTP](#asynchronous-limits).

<a name="compose-action"></a>

### <a name="compose-action"></a>Action compose

Cette action crée une sortie unique à partir de plusieurs entrées, y compris des expressions. La sortie et les entrées peuvent avoir n’importe quel type pris en charge nativement par Azure Logic Apps, tels que des tableaux, objets JSON, XML et binaires. Vous pouvez ensuite utiliser la sortie de l’action dans d’autres actions. 

```json
"Compose": {
   "type": "Compose",
   "inputs": "<inputs-to-compose>",
   "runAfter": {}
},
```

*Obligatoire* 

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*inputs-to-compose*> | Quelconque | Entrées pour la création d’une sortie unique | 
|||| 

*Exemple 1*

<!-- markdownlint-disable MD038 -->
Cette définition d’action fusionne `abcdefg ` avec un espace de fin et la valeur `1234` :
<!-- markdownlint-enable MD038 -->

```json
"Compose": {
   "type": "Compose",
   "inputs": "abcdefg 1234",
   "runAfter": {}
},
```

Voici la sortie créée par cette action :

`abcdefg 1234`

*Exemple 2*

Cette définition d’action fusionne une variable de chaîne qui contient `abcdefg` et une variable de type entier qui contient `1234` :

```json
"Compose": {
   "type": "Compose",
   "inputs": "@{variables('myString')}@{variables('myInteger')}",
   "runAfter": {}
},
```

Voici la sortie créée par cette action :

`"abcdefg1234"`

<a name="run-javascript-code"></a>

### <a name="execute-javascript-code-action"></a>Action Exécuter du code JavaScript

Cette action exécute un extrait de code JavaScript et renvoie les résultats via un jeton `Result` auquel les actions ultérieures peuvent faire référence.

```json
"Execute_JavaScript_Code": {
   "type": "JavaScriptCode",
   "inputs": {
      "code": "<JavaScript-code-snippet>",
      "explicitDependencies": {
         "actions": [ <previous-actions> ],
         "includeTrigger": true
      }
   },
   "runAfter": {}
}
```

*Obligatoire*

| Valeur | Type | Description |
|-------|------|-------------|
| <*JavaScript-code-snippet*> | Variable | Code JavaScript que vous souhaitez exécuter. Pour connaître les exigences relatives au code et d’autres informations, voir [Ajouter et exécuter des extraits de code avec du code inclus dans Azure Logic Apps](../logic-apps/logic-apps-add-run-inline-code.md). <p>Dans l’attribut`code`, votre extrait de code peut utiliser l’objet `workflowContext`en lecture seule en tant qu’entrée. Cet objet possède des sous-propriétés qui permettent à votre code d’accéder aux résultats provenant du déclencheur, ainsi que les actions précédentes, dans votre flux de travail. Pour en savoir plus sur l’objet `workflowContext`, voir [Faire référence aux résultats des actions et du déclencheur dans votre code](../logic-apps/logic-apps-add-run-inline-code.md#workflowcontext). |
||||

*Obligatoire dans certains cas*

L’attribut `explicitDependencies` spécifie que vous souhaitez inclure explicitement les résultats à partir du déclencheur et/ou des actions précédentes, en tant que dépendances de votre extrait de code. Pour en savoir plus sur l’ajout de ces dépendances, voir [Add parameters for inline code](../logic-apps/logic-apps-add-run-inline-code.md#add-parameters) (Ajout de paramètres pour le code inclus). 

Pour l’attribut `includeTrigger`, vous pouvez spécifier les valeurs `true` ou `false`.

| Valeur | Type | Description |
|-------|------|-------------|
| <*previous-actions*> | Tableau de chaînes | Tableau incluant les noms d’action que vous avez spécifiés. Utilisez les noms d’action qui s’affichent dans votre définition de flux de travail lorsqu’ils utilisent des traits de soulignement (_), et non des espaces (« »). |
||||

*Exemple 1*

Cette action exécute du code qui obtient le nom de votre application logique et retourne le texte « Hello world from \<logic-app-name> » en tant que résultat. Dans cet exemple, le code référence le nom du flux de travail en accédant à la propriété `workflowContext.workflow.name` via l’objet `workflowContext` en lecture seule. Pour en savoir plus sur l’utilisation de l’objet `workflowContext`, voir [Faire référence aux résultats des actions et du déclencheur dans votre code](../logic-apps/logic-apps-add-run-inline-code.md#workflowcontext).

```json
"Execute_JavaScript_Code": {
   "type": "JavaScriptCode",
   "inputs": {
      "code": "var text = \"Hello world from \" + workflowContext.workflow.name;\r\n\r\nreturn text;"
   },
   "runAfter": {}
}
```

*Exemple 2*

Cette action exécute le code dans une application logique qui se déclenche lorsqu’un nouvel e-mail arrive sur un compte professionnel ou scolaire. L’application logique utilise également une action Envoyer un e-mail d’approbation, qui transfère le contenu de l’e-mail reçu ainsi qu’une requête d’approbation.

Le code extrait les adresses e-mail à partir de la propriété `Body` du déclencheur, et renvoie les adresses avec la valeur de la propriété `SelectedOption` à partir de l’action d’approbation. L’action inclut de manière explicite l’action Envoyer un e-mail d’approbation en tant que dépendance dans l’attribut `explicitDependencies` > `actions`.

```json
"Execute_JavaScript_Code": {
   "type": "JavaScriptCode",
   "inputs": {
      "code": "var re = /(([^<>()\\[\\]\\\\.,;:\\s@\"]+(\\.[^<>()\\[\\]\\\\.,;:\\s@\"]+)*)|(\".+\"))@((\\[[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}])|(([a-zA-Z\\-0-9]+\\.)+[a-zA-Z]{2,}))/g;\r\n\r\nvar email = workflowContext.trigger.outputs.body.Body;\r\n\r\nvar reply = workflowContext.actions.Send_approval_email_.outputs.body.SelectedOption;\r\n\r\nreturn email.match(re) + \" - \" + reply;\r\n;",
      "explicitDependencies": {
         "actions": [
            "Send_approval_email_"
         ]
      }
   },
   "runAfter": {}
}
```



<a name="function-action"></a>

### <a name="function-action"></a>Action function

Cette action appelle une [fonction Azure](../azure-functions/functions-create-first-azure-function.md) créée précédemment.

```json
"<Azure-function-name>": {
   "type": "Function",
   "inputs": {
     "function": {
        "id": "<Azure-function-ID>"
      },
      "method": "<method-type>",
      "headers": { "<header-content>" },
      "body": { "<body-content>" },
      "queries": { "<query-parameters>" } 
   },
   "runAfter": {}
}
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------|  
| <*Azure-function-ID*> | String | ID de ressource de la fonction Azure que vous souhaitez appeler. Voici le format de cette valeur :<p>"/subscriptions/<*ID_abonnement_Azure*>/resourceGroups/<*groupe_ressources_Azure*>/providers/Microsoft.Web/sites/<*nom_application_fonction_Azure*>/functions/<*nom_fonction_Azure*>" | 
| <*method-type*> | String | Méthode HTTP à utiliser pour appeler la fonction : « GET », « PUT », « POST », « PATCH » ou « DELETE » <p>Si elle n’est pas spécifiée, la méthode par défaut est « POST ». | 
||||

*Facultatif*

| Valeur | Type | Description | 
|-------|------|-------------|  
| <*header-content*> | Objet JSON | En-têtes à envoyer avec l’appel <p>Par exemple, pour définir la langue et le type d’une requête : <p>`"headers": { "Accept-Language": "en-us", "Content-Type": "application/json" }` |
| <*body-content*> | Objet JSON | Tout contenu de message à envoyer dans la requête | 
| <*query-parameters*> | Objet JSON | Paramètres de requête à inclure avec l’appel d’API <p>Par exemple, l’objet `"queries": { "api-version": "2018-01-01" }` ajoute `?api-version=2018-01-01` à l’appel. | 
| <*other-action-specific-input-properties*> | Objet JSON | Toutes autres propriétés d’entrée qui s’appliquent à cette action spécifique | 
| <*other-action-specific-properties*> | Objet JSON | Toutes autres propriétés qui s’appliquent à cette action spécifique | 
||||

Quand vous enregistrez votre application logique, le moteur Logic Apps effectue ces contrôles sur la fonction référencée :

* Votre workflow doit avoir accès à la fonction.

* Votre workflow ne peut utiliser qu’un déclencheur HTTP standard ou un déclencheur webhook JSON générique. 

  Le moteur Logic Apps obtient et met en cache l’URL du déclencheur, qui est utilisée au moment de l’exécution. Toutefois, si une opération invalide l’URL en cache, l’action **Function** échoue au moment de l’exécution. Pour résoudre ce problème, réenregistrez l’application logique afin qu’elle obtienne et mette en cache l’URL du déclencheur.

* La fonction ne doit pas avoir d’itinéraire défini.

* Seuls les niveaux d’autorisation « fonction » et « anonyme » sont autorisés. 

*Exemple*

Cette définition d’action appelle la fonction « GetProductID » créée précédemment :

```json
"GetProductID": {
   "type": "Function",
   "inputs": {
     "function": {
        "id": "/subscriptions/<XXXXXXXXXXXXXXXXXXXX>/resourceGroups/myLogicAppResourceGroup/providers/Microsoft.Web/sites/InventoryChecker/functions/GetProductID"
      },
      "method": "POST",
      "headers": { 
          "x-ms-date": "@utcnow()"
       },
      "body": { 
          "Product_ID": "@variables('ProductID')"
      }
   },
   "runAfter": {}
}
```

<a name="http-action"></a>

### <a name="http-action"></a>Action HTTP

Cette action envoie une requête au point de terminaison HTTP ou HTTPS spécifié et vérifie la réponse pour déterminer si le workflow s’exécute. Pour plus d’informations, consultez [Appeler des points de terminaison de service via HTTP ou HTTPS à partir d’Azure Logic Apps](../connectors/connectors-native-http.md).

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "<method-type>",
      "uri": "<HTTP-or-HTTPS-endpoint-URL>",
      "headers": { "<header-content>" },
      "queries": { "<query-parameters>" },
      "body": "<body-content>",
      "authentication": { "<authentication-type-and-property-values>" },
      "retryPolicy": {
         "type": "<retry-behavior>"
      },
   },
   "runAfter": {}
}
```

*Obligatoire*

| Propriété | Valeur | Type | Description |
|----------|-------|------|-------------|
| `method` | <*method-type*> | String | Méthode à utiliser pour envoyer la requête sortante : « GET », « PUT », « POST », « PATCH » ou « DELETE » |
| `uri` | <*HTTP-or-HTTPS-endpoint-URL*> | String | URL du point de terminaison HTTP ou HTTPS où vous voulez envoyer la requête sortante. Taille de chaîne maximale : 2 Ko <p>Pour un service ou une ressource Azure, cette syntaxe URI inclut l'ID de la ressource et le chemin vers la ressource à laquelle vous voulez accéder. |
|||||

*Facultatif*

| Propriété | Valeur | Type | Description |
|----------|-------|------|-------------|
| `headers` | <*header-content*> | Objet JSON | Tous les en-têtes que vous devez inclure avec la demande <p>Par exemple, pour définir la langue et le type : <p>`"headers": { "Accept-Language": "en-us", "Content-Type": "application/json" }` |
| `queries` | <*query-parameters*> | Objet JSON | Tous les paramètres de requête que vous devez utiliser dans la requête <p>Par exemple, l’objet `"queries": { "api-version": "2018-01-01" }` ajoute `?api-version=2018-01-01` à l’appel. |
| `body` | <*body-content*> | Objet JSON | Contenu du message à envoyer en tant que charge utile avec la requête |
| `authentication` | <*authentication-type-and-property-values*> | Objet JSON | Le modèle d'authentification que la requête utilise pour authentifier les requêtes sortantes. Pour plus d’informations, voir [Ajouter l’authentification aux appels sortants](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound). Au-delà de Scheduler, la propriété `authority` est prise en charge. Si vous ne spécifiez aucune valeur, la valeur par défaut est `https://management.azure.com/`, mais vous pouvez utiliser une autre valeur. |
| `retryPolicy` > `type` | <*retry-behavior*> | Objet JSON | Personnalise le comportement de nouvelle tentative pour les défaillances intermittentes, qui présentent le code d’état 408, 429 et 5XX, ainsi que les éventuelles exceptions de connectivité. Pour plus d’informations, consultez [Stratégies de relance](../logic-apps/logic-apps-exception-handling.md#retry-policies). |
| <*other-action-specific-input-properties*> | <*input-property*> | Objet JSON | Toutes autres propriétés d’entrée qui s’appliquent à cette action spécifique |
| <*other-action-specific-properties*> | <*property-value*> | Objet JSON | Toutes autres propriétés qui s’appliquent à cette action spécifique |
|||||

*Exemple*

Cette définition d’action obtient les informations les plus récentes en envoyant une requête au point de terminaison spécifié :

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "https://mynews.example.com/latest"
   }
}
```

<a name="join-action"></a>

### <a name="join-action"></a>Action Joindre

Cette action crée une chaîne à partir de tous les éléments d’un tableau, et sépare ces éléments avec le caractère délimiteur spécifié. 

```json
"Join": {
   "type": "Join",
   "inputs": {
      "from": <array>,
      "joinWith": "<delimiter>"
   },
   "runAfter": {}
}
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*array*> | Array | Tableau ou expression qui fournit les éléments sources. Si vous spécifiez une expression, placez l’expression entre guillemets doubles. | 
| <*delimiter*> | Chaîne d’un seul caractère | Caractère qui sépare chaque élément dans la chaîne | 
|||| 

*Exemple*

Supposez que vous avez créé une variable « myIntegerArray » qui contient ce tableau d’entiers : 

`[1,2,3,4]` 

Cette définition d’action obtient les valeurs de la variable en utilisant la fonction `variables()` dans une expression, et elle crée cette chaîne avec ces valeurs, séparées par une virgule : `"1,2,3,4"`

```json
"Join": {
   "type": "Join",
   "inputs": {
      "from": "@variables('myIntegerArray')",
      "joinWith": ","
   },
   "runAfter": {}
}
```

<a name="parse-json-action"></a>

### <a name="parse-json-action"></a>Action d’analyse de JSON

Cette action crée des *jetons* ou champs conviviaux à partir des propriétés dans le contenu JSON. Vous pouvez ensuite accéder à ces propriétés dans votre application logique en utilisant les jetons à la place. Par exemple, quand vous souhaitez utiliser la sortie JSON de services tels qu’Azure Service Bus et Azure Cosmos DB, vous pouvez inclure cette action dans votre application logique afin de pouvoir référencer plus facilement les données dans cette sortie.

```json
"Parse_JSON": {
   "type": "ParseJson",
   "inputs": {
      "content": "<JSON-source>",
         "schema": { "<JSON-schema>" }
      },
      "runAfter": {}
},
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*JSON-source*> | Objet JSON | Contenu JSON que vous souhaitez analyser | 
| <*JSON-schema*> | Objet JSON | Schéma JSON qui décrit le contenu JSON sous-jacent, utilisé par l’action pour analyser le contenu JSON source. <p>**Conseil** : Dans le Concepteur d’applications logiques, vous pouvez fournir le schéma ou fournir un exemple de charge utile afin que l’action puisse générer le schéma. | 
|||| 

*Exemple*

Cette définition d’action crée les jetons que vous pouvez utiliser dans votre flux de travail, mais uniquement dans les actions qui s’exécutent après l’action **Analyser JSON** :

`FirstName`, `LastName` et `Email`

```json
"Parse_JSON": {
   "type": "ParseJson",
   "inputs": {
      "content": {
         "Member": {
            "Email": "Sophie.Owen@contoso.com",
            "FirstName": "Sophie",
            "LastName": "Owen"
         }
      },
      "schema": {
         "type": "object",
         "properties": {
            "Member": {
               "type": "object",
               "properties": {
                  "Email": {
                     "type": "string"
                  },
                  "FirstName": {
                     "type": "string"
                  },
                  "LastName": {
                     "type": "string"
                  }
               }
            }
         }
      }
   },
   "runAfter": { }
},
```

Dans cet exemple, la propriété « content » spécifie le contenu JSON devant être analysé par l’action. Vous pouvez également fournir ce contenu JSON comme exemple de charge utile pour générer le schéma.

```json
"content": {
   "Member": { 
      "FirstName": "Sophie",
      "LastName": "Owen",
      "Email": "Sophie.Owen@contoso.com"
   }
},
```

La propriété « schema » spécifie le schéma JSON utilisé pour décrire le contenu JSON :

```json
"schema": {
   "type": "object",
   "properties": {
      "Member": {
         "type": "object",
         "properties": {
            "FirstName": {
               "type": "string"
            },
            "LastName": {
               "type": "string"
            },
            "Email": {
               "type": "string"
            }
         }
      }
   }
}
```

<a name="query-action"></a>

### <a name="query-action"></a>Action de requête

Cette action crée un tableau à partir des éléments d’un autre tableau en fonction d’une condition ou d’un filtre spécifié.

```json
"Filter_array": {
   "type": "Query",
   "inputs": {
      "from": <array>,
      "where": "<condition-or-filter>"
   },
   "runAfter": {}
}
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*array*> | Array | Tableau ou expression qui fournit les éléments sources. Si vous spécifiez une expression, placez l’expression entre guillemets doubles. |
| <*condition-or-filter*> | String | Condition utilisée pour le filtrage des éléments dans le tableau source <p>**Remarque** : Si aucune valeur ne remplit la condition, l’action crée un tableau vide. |
|||| 

*Exemple*

Cette définition d’action crée un tableau qui contient des valeurs supérieures à la valeur spécifiée (deux) :

```json
"Filter_array": {
   "type": "Query",
   "inputs": {
      "from": [ 1, 3, 0, 5, 4, 2 ],
      "where": "@greater(item(), 2)"
   }
}
```

<a name="response-action"></a>

### <a name="response-action"></a>Action de réponse  

Cette action crée la charge utile pour la réponse à une requête HTTP. 

```json
"Response" {
    "type": "Response",
    "kind": "http",
    "inputs": {
        "statusCode": 200,
        "headers": { <response-headers> },
        "body": { <response-body> }
    },
    "runAfter": {}
},
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*response-status-code*> | Integer | Code d’état HTTP envoyé à la requête entrante. Le code par défaut est « 200 OK », mais il peut s’agir de tout code d’état valide commençant par 2xx, 4xx ou 5xx, mais pas avec 3xxx. | 
|||| 

*Facultatif*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*response-headers*> | Objet JSON | Un ou plusieurs en-têtes à inclure avec la réponse | 
| <*response-body*> | Divers | Corps de réponse, qui peut être une chaîne, un objet JSON ou même du contenu binaire d’une action précédente | 
|||| 

*Exemple*

Cette définition d’action crée une réponse à une requête HTTP avec le code d’état, le corps du message et les en-têtes de message spécifiés :

```json
"Response": {
   "type": "Response",
   "inputs": {
      "statusCode": 200,
      "body": {
         "ProductID": 0,
         "Description": "Organic Apples"
      },
      "headers": {
         "x-ms-date": "@utcnow()",
         "content-type": "application/json"
      }
   },
   "runAfter": {}
}
```

*Restrictions*

Contrairement à d’autres actions, l’action **Response** présente des restrictions spéciales : 

* Votre workflow peut utiliser l’action **Response** uniquement quand le workflow démarre avec un déclencheur de requête HTTP, ce qui signifie que votre workflow doit être déclenché par une requête HTTP.

* Votre workflow peut utiliser l’action **Response** n’importe où *sauf* à l’intérieur de boucles **Foreach**, de boucles **Until**, y compris les boucles séquentielles, et de branches parallèles. 

* La requête HTTP d’origine obtient la réponse de votre workflow uniquement quand toutes les actions requises par  l’action **Response** sont terminées dans la [limite de délai d’attente de requête HTTP](../logic-apps/logic-apps-limits-and-config.md#request-limits).

  Toutefois, si votre workflow appelle une autre application logique en tant que workflow imbriqué, le workflow parent attend que le workflow imbriqué se termine, quelle que soit la durée nécessaire.

* Quand votre workflow utilise l’action **Response** et un modèle de réponse synchrone, le workflow ne peut pas non plus utiliser la commande **splitOn** dans la définition du déclencheur, car cette commande crée plusieurs exécutions. Vérifiez ce cas quand la méthode PUT est utilisée et, dans l’affirmative, retournez une réponse « requête incorrecte ».

  Sinon, si votre workflow utilise la commande **splitOn** et une action **Response**, le workflow s’exécute de façon asynchrone et retourne immédiatement une réponse « 202 ACCEPTED ».

* Quand l’exécution du workflow atteint l’action **Response**, mais que la requête entrante a déjà reçu une réponse, l’action **Response**, est marquée comme « Échec » à cause du conflit. En conséquence, l’exécution de votre application logique est également marquée avec l’état « Échec ».

<a name="select-action"></a>

### <a name="select-action"></a>Action select

Cette action crée un tableau avec des objets JSON en transformant les éléments d’un autre tableau en fonction de la carte spécifiée. Le tableau de sortie et le tableau source ont toujours le même nombre d’éléments. Bien que vous ne puissiez pas changer le nombre d’objets dans le tableau de sortie, vous pouvez ajouter ou supprimer des propriétés et leurs valeurs à ces objets. La propriété `select` spécifie au moins une paire clé-valeur qui définit le mappage pour transformer les éléments du tableau source. Une paire clé-valeur représente une propriété et sa valeur parmi tous les objets dans le tableau de sortie.

```json
"Select": {
   "type": "Select",
   "inputs": {
      "from": <array>,
      "select": { 
          "<key-name>": "<expression>",
          "<key-name>": "<expression>"        
      }
   },
   "runAfter": {}
},
```

*Obligatoire* 

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*array*> | Array | Tableau ou expression qui fournit les éléments sources. N’oubliez pas de placer une expression entre guillemets doubles. <p>**Remarque** : Si le tableau source est vide, l’action crée un tableau vide. | 
| <*key-name*> | String | Nom de la propriété attribué au résultat à partir de <*expression*> <p>Pour ajouter une nouvelle propriété à tous les objets dans le tableau de sortie, fournissez un <*key-name*> pour cette propriété et une <*expression*> pour la valeur de propriété. <p>Pour supprimer une propriété de tous les objets dans le tableau, omettez le <*key-name*> pour cette propriété. | 
| <*expression*> | String | Expression qui transforme l’élément du tableau source et assigne le résultat à <*key-name*> | 
|||| 

L’action **Select** crée un tableau en tant que sortie. Par conséquent, toute action qui souhaite utiliser cette sortie doit accepter un tableau, ou vous devez convertir le tableau vers le type acceptée par l’action consommatrice. Par exemple, pour convertir le tableau de sortie en une chaîne, vous pouvez passer ce tableau à l’action **Compose**, puis référencer la sortie de l’action **Compose** dans vos autres actions.

*Exemple*

Cette définition d’action crée un tableau d’objets JSON à partir d’un tableau d’entiers. L’action itère au sein du tableau source, obtient chaque valeur entière à l’aide de l’expression `@item()`, et assigne chaque valeur à la propriété « `number` » dans chaque objet JSON :

```json
"Select": {
   "type": "Select",
   "inputs": {
      "from": [ 1, 2, 3 ],
      "select": { 
         "number": "@item()" 
      }
   },
   "runAfter": {}
},
```

Voici le tableau créé par cette action :

`[ { "number": 1 }, { "number": 2 }, { "number": 3 } ]`

Pour utiliser ce tableau dans d’autres actions, passez cette sortie dans une action **Compose** :

```json
"Compose": {
   "type": "Compose",
   "inputs": "@body('Select')",
   "runAfter": {
      "Select": [ "Succeeded" ]
   }
},
```

Vous pouvez ensuite utiliser la sortie de l’action **Compose** dans vos autres actions, par exemple l’action **Office 365 Outlook - Envoyer un e-mail** :

```json
"Send_an_email": {
   "type": "ApiConnection",
   "inputs": {
      "body": {
         "Body": "@{outputs('Compose')}",
         "Subject": "Output array from Select and Compose actions",
         "To": "<your-email@domain>"
      },
      "host": {
         "connection": {
            "name": "@parameters('$connections')['office365']['connectionId']"
         }
      },
      "method": "post",
      "path": "/Mail"
   },
   "runAfter": {
      "Compose": [ "Succeeded" ]
   }
},
```

<a name="table-action"></a>

### <a name="table-action"></a>Action table

Cette action crée une table CSV ou HTML à partir d’un tableau. Pour les tableaux contenant des objets JSON, cette action crée automatiquement les en-têtes de colonnes à partir des noms de propriétés des objets. Pour les tableaux contenant d’autres types de données, vous devez spécifier les en-têtes de colonnes et les valeurs. Par exemple, ce tableau inclut les propriétés « ID » et « Product_Name » que cette action peut utiliser pour les noms d’en-têtes de colonnes :

`[ {"ID": 0, "Product_Name": "Apples"}, {"ID": 1, "Product_Name": "Oranges"} ]` 

```json
"Create_<CSV | HTML>_table": {
   "type": "Table",
   "inputs": {
      "format": "<CSV | HTML>",
      "from": <array>,
      "columns": [ 
         {
            "header": "<column-name>",
            "value": "<column-value>"
         },
         {
            "header": "<column-name>",
            "value": "<column-value>"
         } 
      ]
   },
   "runAfter": {}
}
```

*Obligatoire* 

| Valeur | Type | Description | 
|-------|------|-------------| 
| \<CSV *or* HTML>| String | Format de la table que vous souhaitez créer | 
| <*array*> | Array | Tableau ou expression qui fournit les éléments sources pour la table <p>**Remarque** : Si le tableau source est vide, l’action crée une table vide. | 
|||| 

*Facultatif*

Pour spécifier ou personnaliser des en-têtes de colonne et des valeurs, utilisez le tableau `columns`. Quand des paires `header-value` ont le même nom d’en-tête, leurs valeurs s’affichent dans la même colonne sous ce nom d’en-tête. Dans le cas contraire, chaque en-tête unique définit une colonne unique.

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*column-name*> | String | Nom d’en-tête pour une colonne | 
| <*column-value*> | Quelconque | Valeur de la colonne | 
|||| 

*Exemple 1*

Supposez que vous avez créé une variable « myItemArray » qui contenant actuellement ce tableau :

`[ {"ID": 0, "Product_Name": "Apples"}, {"ID": 1, "Product_Name": "Oranges"} ]`

Cette définition d’action crée une table CSV à partir de la variable « myItemArray ». L’expression utilisée par la propriété `from` obtient le tableau à partir de « myItemArray » à l’aide de la fonction `variables()` :

```json
"Create_CSV_table": {
   "type": "Table",
   "inputs": {
      "format": "CSV",
      "from": "@variables('myItemArray')"
   },
   "runAfter": {}
}
```

Voici la table CSV créée par cette action : 

```
ID,Product_Name 
0,Apples 
1,Oranges 
```

*Exemple 2*

Cette définition d’action crée une table HTML à partir de la variable « myItemArray ». L’expression utilisée par la propriété `from` obtient le tableau à partir de « myItemArray » à l’aide de la fonction `variables()` :

```json
"Create_HTML_table": {
   "type": "Table",
   "inputs": {
      "format": "HTML",
      "from": "@variables('myItemArray')"
   },
   "runAfter": {}
}
```

Voici la table HTML créée par cette action : 

<table><thead><tr><th>id</th><th>Product_Name</th></tr></thead><tbody><tr><td>0</td><td>Pommes</td></tr><tr><td>1</td><td>Oranges</td></tr></tbody></table>

*Exemple 3*

Cette définition d’action crée une table HTML à partir de la variable « myItemArray ». Toutefois, cet exemple remplace les noms d’en-têtes de colonnes par défaut par « Stock_ID » et « Description », et ajoute le mot « Organic » aux valeurs dans la colonne « Description ».

```json
"Create_HTML_table": {
   "type": "Table",
   "inputs": {
      "format": "HTML",
      "from": "@variables('myItemArray')",
      "columns": [ 
         {
            "header": "Stock_ID",
            "value": "@item().ID"
         },
         {
            "header": "Description",
            "value": "@concat('Organic ', item().Product_Name)"
         }
      ]
    },
   "runAfter": {}
},
```

Voici la table HTML créée par cette action : 

<table><thead><tr><th>Stock_ID</th><th>Description</th></tr></thead><tbody><tr><td>0</td><td>Organic Apples</td></tr><tr><td>1</td><td>Organic Oranges</td></tr></tbody></table>

<a name="terminate-action"></a>

### <a name="terminate-action"></a>Action terminate

Cette action arrête l’exécution de l’instance de flux de travail, annule toutes les actions en cours, ignore les actions restantes et renvoie l’état spécifié. Vous pouvez par exemple utiliser l’action **Terminate** quand votre application logique doit se fermer complètement suite à un état d’erreur. Cette action n’affecte pas les actions déjà terminées, et ne peut pas apparaître à l’intérieur de boucles **Foreach** et **Until**, y compris les boucles séquentielles.

```json
"Terminate": {
   "type": "Terminate",
   "inputs": {
       "runStatus": "<status>",
       "runError": {
            "code": "<error-code-or-name>",
            "message": "<error-message>"
       }
   },
   "runAfter": {}
}
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*status*> | String | État à retourner pour l’exécution : « Démarré », « Échec » ou « Réussi » |
|||| 

*Facultatif*

Les propriétés de l’objet « runStatus » s’appliquent uniquement quand la propriété « runStatus » a l’état « Failed ».

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*error-code-or-name*> | String | Code ou nom de l’erreur |
| <*error-message*> | String | Message ou texte qui décrit l’erreur et les actions que peut effectuer l’utilisateur de l’application | 
|||| 

*Exemple*

Cette définition d’action arrête l’exécution d’un workflow, affecte la valeur « Failed » à l’état d’exécution, et retourne l’état, un code d’erreur et un message d’erreur :

```json
"Terminate": {
    "type": "Terminate",
    "inputs": {
        "runStatus": "Failed",
        "runError": {
            "code": "Unexpected response",
            "message": "The service received an unexpected response. Please try again."
        }
   },
   "runAfter": {}
}
```

<a name="wait-action"></a>

### <a name="wait-action"></a>Action wait

Cette action interrompt l’exécution du workflow pendant l’intervalle spécifié ou jusqu’à l’heure spécifiée, mais pas les deux.

*Intervalle spécifié*

```json
"Delay": {
   "type": "Wait",
   "inputs": {
      "interval": {
         "count": <number-of-units>,
         "unit": "<interval>"
      }
   },
   "runAfter": {}
},
```

*Heure spécifiée*

```json
"Delay_until": {
   "type": "Wait",
   "inputs": {
      "until": {
         "timestamp": "<date-time-stamp>"
      }
   },
   "runAfter": {}
},
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*number-of-units*> | Integer | Pour l’action **Delay**, il s’agit du nombre d’unités pendant lesquelles attendre | 
| <*interval*> | String | Pour l’action **Delay**, il s’agit de l’intervalle d’attente : "Second", "Minute", "Hour", "Day", "Week", "Month" | 
| <*date-time-stamp*> | String | Pour l’action **Delay**, il s’agit de la date et de l’heure de reprise de l’exécution. Cette valeur doit utiliser le [format date/heure UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time). | 
|||| 

*Exemple 1*

Cette définition d’action suspend le workflow pendant 15 minutes :

```json
"Delay": {
   "type": "Wait",
   "inputs": {
      "interval": {
         "count": 15,
         "unit": "Minute"
      }
   },
   "runAfter": {}
},
```

*Exemple 2*

Cette définition d’action suspend le workflow jusqu’à l’heure spécifiée :

```json
"Delay_until": {
   "type": "Wait",
   "inputs": {
      "until": {
         "timestamp": "2017-10-01T00:00:00Z"
      }
   },
   "runAfter": {}
},
```

<a name="workflow-action"></a>

### <a name="workflow-action"></a>Action workflow

Cette action appelle une autre application logique créée précédemment, ce qui signifie que vous pouvez inclure et réutiliser d’autres workflows d’application logique. Vous pouvez également utiliser les sorties de l’application logique enfant ou *imbriquée* dans des actions qui suivent l’application logique imbriquée, à condition que l’application logique enfant retourne une réponse.

Le moteur Logic Apps vérifie l’accès au déclencheur que vous souhaitez appeler. Par conséquent, vérifiez que vous pouvez accéder à ce déclencheur. En outre, l’application logique imbriquée doit répondre aux critères suivants :

* Un déclencheur fait en sorte que l’application logique imbriquée puisse être appelée (par exemple un déclencheur [Request](#request-trigger) ou [HTTP](#http-trigger))

* Il s’agit du même abonnement Azure que votre application logique parente

* Pour utiliser les sorties de l’application logique imbriquée dans votre application logique parente, l’application logique imbriquée doit avoir une action [Response](#response-action)

```json
"<nested-logic-app-name>": {
   "type": "Workflow",
   "inputs": {
      "body": { "<body-content" },
      "headers": { "<header-content>" },
      "host": {
         "triggerName": "<trigger-name>",
         "workflow": {
            "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group>/providers/Microsoft.Logic/<nested-logic-app-name>"
         }
      }
   },
   "runAfter": {}
}
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*nested-logic-app-name*> | String | Nom de l’application logique que vous souhaitez appeler | 
| <*trigger-name*> | String | Nom du déclencheur dans l’application logique imbriquée que vous souhaitez appeler | 
| <*Azure-subscription-ID*> | String | ID d’abonnement Azure pour l’application logique imbriquée |
| <*Azure-resource-group*> | String | Nom du groupe de ressources Azure pour l’application logique imbriquée |
| <*nested-logic-app-name*> | String | Nom de l’application logique que vous souhaitez appeler |
||||

*Facultatif*

| Valeur | Type | Description | 
|-------|------|-------------|  
| <*header-content*> | Objet JSON | En-têtes à envoyer avec l’appel | 
| <*body-content*> | Objet JSON | Tout contenu de message à envoyer avec l’appel | 
||||

*Sorties*

Les sorties de cette action dépendent de l’action Response de l’application logique imbriquée. Si celle-ci n’inclut pas d’action Response, les sorties sont vides.

*Exemple*

Une fois que l’action « Start_search » se termine correctement, cette définition d’action de workflow appelle une autre application logique nommée « Get_product_information », qui transmet les entrées spécifiées :

```json
"actions": {
   "Start_search": { <action-definition> },
   "Get_product_information": {
      "type": "Workflow",
      "inputs": {
         "body": {
            "ProductID": "24601",
         },
         "host": {
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/InventoryManager-RG/providers/Microsoft.Logic/Get_product_information",
            "triggerName": "Find_product"
         },
         "headers": {
            "content-type": "application/json"
         }
      },
      "runAfter": { 
         "Start_search": [ "Succeeded" ]
      }
   }
},
```

## <a name="control-workflow-action-details"></a>Détails des actions de workflow de contrôle

<a name="foreach-action"></a>

### <a name="foreach-action"></a>Action Foreach

Cette action de bouclage effectue une itération au sein d’un tableau et exécute des actions sur chaque élément. Par défaut, la boucle « for each » s’exécute en parallèle jusqu’à un nombre maximal de boucles. Pour plus d’informations sur ce maximum, consultez [Limites et configuration](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Dévouvrez [comment créer des boucles « for each »](../logic-apps/logic-apps-control-flow-loops.md#foreach-loop).

```json
"For_each": {
   "type": "Foreach",
   "actions": { 
      "<action-1>": { "<action-definition-1>" },
      "<action-2>": { "<action-definition-2>" }
   },
   "foreach": "<for-each-expression>",
   "runAfter": {},
   "runtimeConfiguration": {
      "concurrency": {
         "repetitions": <count>
      }
    },
    "operationOptions": "<operation-option>"
}
```

*Obligatoire* 

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*action-1...n*> | String | Noms des actions qui s’exécutent sur chaque élément du tableau | 
| <*action-definition-1...n*> | Objet JSON | Définitions des actions qui s’exécutent | 
| <*for-each-expression*> | String | Expression référence chaque élément du tableau spécifié | 
|||| 

*Facultatif*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*count*> | Integer | Par défaut, les itérations de boucle « for each » s’exécutent en même temps (simultanément ou en parallèle) jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer cette limite en définissant une nouvelle valeur <*count*>, consultez [Changer la concurrence de boucle « for each »](#change-for-each-concurrency). | 
| <*operation-option*> | String | Pour exécuter une boucle « for each » séquentiellement plutôt qu’en parallèle, affectez la valeur `Sequential` à <*operation-option*> ou la valeur `1` à <*count*>, mais pas les deux. Pour plus d’informations, consultez [Exécuter des boucles « for each » séquentiellement](#sequential-for-each). | 
|||| 

*Exemple*

Cette boucle « for each » envoie un e-mail pour chaque élément du tableau qui contient des pièces jointes à un e-mail entrant. La boucle envoie un e-mail, pièce jointe comprise, à une personne qui passe en revue la pièce jointe.

```json
"For_each": {
   "type": "Foreach",
   "actions": {
      "Send_an_email": {
         "type": "ApiConnection",
         "inputs": {
            "body": {
               "Body": "@base64ToString(items('For_each')?['Content'])",
               "Subject": "Review attachment",
               "To": "Sophie.Owen@contoso.com"
                },
            "host": {
               "connection": {
                  "id": "@parameters('$connections')['office365']['connectionId']"
               }
            },
            "method": "post",
            "path": "/Mail"
         },
         "runAfter": {}
      }
   },
   "foreach": "@triggerBody()?['Attachments']",
   "runAfter": {}
}
```

Pour spécifier uniquement un tableau qui est passé en tant que sortie du déclencheur, cette expression obtient le tableau <*array-name*> à partir du corps du déclencheur. Pour éviter un échec si le tableau n’existe pas, l’expression utilise l’opérateur `?` :

`@triggerBody()?['<array-name>']` 

<a name="if-action"></a>

### <a name="if-action"></a>Action If

Cette action, qui est une *instruction conditionnelle*, évalue une expression qui représente une condition et exécute une branche différente selon que la condition est true ou false. Si la condition est true, elle est marquée avec l’état « Succeeded ». Découvrez [comment créer des instructions conditionnelles](../logic-apps/logic-apps-control-flow-conditional-statement.md).

``` json
"Condition": {
   "type": "If",
   "expression": { "<condition>" },
   "actions": {
      "<action-1>": { "<action-definition>" }
   },
   "else": {
      "actions": {
        "<action-2>": { "<action-definition" }
      }
   },
   "runAfter": {}
}
```

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*condition*> | Objet JSON | Condition, qui peut être une expression, à évaluer | 
| <*action-1*> | Objet JSON | Action à exécuter quand <*condition*> a la valeur true | 
| <*action-definition*> | Objet JSON | Définition de l’action | 
| <*action-2*> | Objet JSON | Action à exécuter quand <*condition*> a la valeur false | 
|||| 

Les actions dans les objets `actions` ou `else` reçoivent ces états :

* Réussite, lorsqu’elles sont exécutées et qu’elles réussissent
* Échec, lorsqu’elles sont exécutées et qu’elles échouent
* Ignoré, lorsque la branche respective ne s’exécute pas

*Exemple*

Cette condition spécifie que quand la variable entière a une valeur supérieure à zéro, le workflow vérifie un site web. Si la variable est égale à zéro ou moins, le workflow vérifie un autre site web.

```json
"Condition": {
   "type": "If",
   "expression": {
      "and": [ {
         "greater": [ "@variables('myIntegerVariable')", 0 ] 
      } ]
   },
   "actions": { 
      "HTTP - Check this website": {
         "type": "Http",
         "inputs": {
         "method": "GET",
            "uri": "http://this-url"
         },
         "runAfter": {}
      }
   },
   "else": {
      "actions": {
         "HTTP - Check this other website": {
            "type": "Http",
            "inputs": {
               "method": "GET",
               "uri": "http://this-other-url"
            },
            "runAfter": {}
         }
      }
   },
   "runAfter": {}
}
```  

#### <a name="how-conditions-use-expressions"></a>Utilisation des expressions dans les conditions

Voici des exemples illustrant comment utiliser des expressions dans des conditions :
  
| JSON | Résultats | 
|------|--------| 
| "expression": "@parameters(’<*a_condition_spéciale*>’)" | Pour les expressions booléennes, la condition est satisfaite pour toute valeur évaluée comme True. <p>Pour convertir d’autres types en valeurs booléennes, utilisez la fonction `empty()` ou `equals()`. | 
| "expression": "@greater(actions(’<*action*>’).output.value, parameters(’<*seuil*>’))" | Pour les fonctions de comparaison, l’action s’exécute uniquement quand la sortie de <*action*> est supérieure à la valeur de <*seuil*>. | 
| "expression": "@or(greater(actions(’<*action*>’).output.value, parameters(’<*seuil*>’)), less(actions(’<*même_action*>’).output.value, 100))" | Pour les fonctions logiques et la création d’expressions booléennes imbriquées, l’action s’exécute quand la sortie de <*action*> est supérieure à la valeur de <*seuil*> ou inférieure à 100. | 
| "expression": "@equals(length(actions(’<*action*>’).outputs.errors), 0))" | Vous pouvez utiliser des fonctions de tableau pour vérifier si le tableau contient des éléments. L’action s’exécute quand le tableau `errors` est vide. | 
||| 

<a name="scope-action"></a>

### <a name="scope-action"></a>Action scope

Cette action regroupe logiquement des actions en *étendues*, qui reçoivent leur propre état à la fin de l’exécution des actions dans cette étendue. Vous pouvez ensuite utiliser l’état de l’étendue pour déterminer si d’autres actions s’exécutent. Découvrez comment [créer des étendues](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md).

```json
"Scope": {
   "type": "Scope",
   "actions": {
      "<inner-action-1>": {
         "type": "<action-type>",
         "inputs": { "<action-inputs>" },
         "runAfter": {}
      },
      "<inner-action-2>": {
         "type": "<action-type>",
         "inputs": { "<action-inputs>" },
         "runAfter": {}
      }
   }
}
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------|  
| <*inner-action-1...n*> | Objet JSON | Une ou plusieurs actions qui s’exécutent au sein de l’étendue |
| <*action-inputs*> | Objet JSON | Entrées pour chaque action |
|||| 

<a name="switch-action"></a>

### <a name="switch-action"></a>Action Switch

Cette action, également appelée *instruction switch*, organise d’autres actions en *cas*, et affecte une valeur à chaque cas, sauf au cas par défaut s’il en existe un. Quand votre workflow s’exécute, l’action **Switch** compare la valeur d’une expression, d’un objet ou d’un jeton aux valeurs spécifiées pour chaque cas. Si l’action **Switch** détecte un cas correspondant, votre workflow exécute uniquement les actions pour ce cas. Chaque fois que l’action **Switch** s’exécute, soit il n’existe qu’un seul cas correspondant, soit il n’existe aucune correspondance. Si aucune correspondance n’existe, l’action **Switch** exécute les actions par défaut. Découvrez [comment créer des instructions switch](../logic-apps/logic-apps-control-flow-switch-statement.md).

``` json
"Switch": {
   "type": "Switch",
   "expression": "<expression-object-or-token>",
   "cases": {
      "Case": {
         "actions": {
           "<action-name>": { "<action-definition>" }
         },
         "case": "<matching-value>"
      },
      "Case_2": {
         "actions": {
           "<action-name>": { "<action-definition>" }
         },
         "case": "<matching-value>"
      }
   },
   "default": {
      "actions": {
         "<default-action-name>": { "<default-action-definition>" }
      }
   },
   "runAfter": {}
}
```

*Obligatoire*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*expression-object-or-token*> | Variable | Expression, objet JSON ou jeton à évaluer | 
| <*action-name*> | String | Nom de l’action à exécuter pour le cas correspondant | 
| <*action-definition*> | Objet JSON | Définition de l’action à exécuter pour le cas correspondant | 
| <*matching-value*> | Variable | Valeur à comparer au résultat évalué | 
|||| 

*Facultatif*

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*default-action-name*> | String | Nom de l’action par défaut à exécuter quand il n’existe aucun cas correspondant | 
| <*default-action-definition*> | Objet JSON | Définition de l’action à exécuter quand il n’existe aucun cas correspondant | 
|||| 

*Exemple*

Cette définition d’action évalue si la personne qui répond à l’e-mail de demande d’approbation a sélectionné l’option « Approuver » ou « Rejeter ». En fonction de ce choix, l’action **Switch** exécute les actions pour le cas correspondant, qui consistent à envoyer un autre e-mail au répondant, mais avec un libellé différent dans chaque cas. 

``` json
"Switch": {
   "type": "Switch",
   "expression": "@body('Send_approval_email')?['SelectedOption']",
   "cases": {
      "Case": {
         "actions": {
            "Send_an_email": { 
               "type": "ApiConnection",
               "inputs": {
                  "Body": "Thank you for your approval.",
                  "Subject": "Response received", 
                  "To": "Sophie.Owen@contoso.com"
               },
               "host": {
                  "connection": {
                     "name": "@parameters('$connections')['office365']['connectionId']"
                  }
               },
               "method": "post",
               "path": "/Mail"
            },
            "runAfter": {}
         },
         "case": "Approve"
      },
      "Case_2": {
         "actions": {
            "Send_an_email_2": { 
               "type": "ApiConnection",
               "inputs": {
                  "Body": "Thank you for your response.",
                  "Subject": "Response received", 
                  "To": "Sophie.Owen@contoso.com"
               },
               "host": {
                  "connection": {
                     "name": "@parameters('$connections')['office365']['connectionId']"
                  }
               },
               "method": "post",
               "path": "/Mail"
            },
            "runAfter": {}     
         },
         "case": "Reject"
      }
   },
   "default": {
      "actions": { 
         "Send_an_email_3": { 
            "type": "ApiConnection",
            "inputs": {
               "Body": "Please respond with either 'Approve' or 'Reject'.",
               "Subject": "Please respond", 
               "To": "Sophie.Owen@contoso.com"
            },
            "host": {
               "connection": {
                  "name": "@parameters('$connections')['office365']['connectionId']"
               }
            },
            "method": "post",
            "path": "/Mail"
         },
         "runAfter": {} 
      }
   },
   "runAfter": {
      "Send_approval_email": [ 
         "Succeeded"
      ]
   }
}
```

<a name="until-action"></a>

### <a name="until-action"></a>Action until

Cette action de boucle contient des actions qui s’exécutent jusqu’à ce que la condition spécifiée ait la valeur true. La boucle vérifie la condition en guise de dernière étape une fois toutes les autres actions exécutées. Vous pouvez inclure plusieurs actions dans l’objet `"actions"`, et l’action doit définir au moins une limite. Découvrez [comment créer des boucles « until »](../logic-apps/logic-apps-control-flow-loops.md#until-loop). 

```json
 "Until": {
   "type": "Until",
   "actions": {
      "<action-name>": {
         "type": "<action-type>",
         "inputs": { "<action-inputs>" },
         "runAfter": {}
      },
      "<action-name>": {
         "type": "<action-type>",
         "inputs": { "<action-inputs>" },
         "runAfter": {}
      }
   },
   "expression": "<condition>",
   "limit": {
      "count": <loop-count>,
      "timeout": "<loop-timeout>"
   },
   "runAfter": {}
}
```

| Valeur | Type | Description | 
|-------|------|-------------| 
| <*action-name*> | String | Nom de l’action que vous souhaitez exécuter à l’intérieur de la boucle | 
| <*action-type*> | String | Type d’action à exécuter | 
| <*action-inputs*> | Divers | Entrées pour l’action à exécuter | 
| <*condition*> | String | Condition ou expression à évaluer une fois que toutes les actions de la boucle ont été exécutées | 
| <*loop-count*> | Integer | Quantité limite de boucles que l’action peut exécuter. Pour plus d’informations sur la limite par défaut et la limite maximale, consultez [Limites and configuration pour Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). | 
| <*loop-timeout*> | String | Durée d’exécution maximale de la boucle. La valeur par défaut de `timeout` est `PT1H`, qui est le [format ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) requis. |
|||| 

> [!NOTE]
> Si l’expression dépend de la sortie d’une action dans la boucle Until, veillez à prendre en compte les échecs résultant de cette action.

*Exemple*

Cette définition d’action de boucle envoie une requête HTTP à l’URL spécifiée jusqu’à ce que l’une des conditions suivantes soit remplie :

* La requête obtient une réponse avec le code d’état « 200 OK ».
* La boucle s’est exécutée 60 fois.
* La boucle s’est exécutée pendant une heure.

```json
 "Run_until_loop_succeeds_or_expires": {
    "type": "Until",
    "actions": {
        "HTTP": {
            "type": "Http",
            "inputs": {
                "method": "GET",
                "uri": "http://myurl"
            },
            "runAfter": {}
        }
    },
    "expression": "@equals(outputs('HTTP')['statusCode'], 200)",
    "limit": {
        "count": 60,
        "timeout": "PT1H"
    },
    "runAfter": {}
}
```

<a name="subscribe-unsubscribe"></a>

## <a name="webhooks-and-subscriptions"></a>Webhooks et abonnements

Les actions et déclencheurs basés sur des webhooks ne vérifient pas régulièrement les points de terminaison, mais attendent des événements ou des données spécifiques à ces points de terminaison. Ces actions et déclencheurs *s’abonnent* aux points de terminaison en fournissant une *URL de rappel* où le point de terminaison peut envoyer des réponses.

L’appel `subscribe` est effectué quand le workflow change de quelque façon, par exemple quand les informations d’identification sont renouvelées ou quand les paramètres d’entrée d’un déclencheur ou d’une action changent. Cet appel utilise les mêmes paramètres que les actions HTTP standard. 

L’appel `unsubscribe` se produit automatiquement quand une opération rend le déclencheur ou l’action non valide, par exemple :

* Suppression ou désactivation du déclencheur. 
* Suppression ou désactivation du workflow. 
* Suppression ou désactivation de l’inscription. 

Pour prendre en charge ces appels, l’expression `@listCallbackUrl()` retourne une « URL de rappel » unique pour le déclencheur ou l’action. Cette URL représente un identificateur unique des points de terminaison qui utilisent l’API REST du service. Les paramètres de cette fonction sont les mêmes que ceux de l’action ou du déclencheur webhook.

<a name="asynchronous-limits"></a>

## <a name="change-asynchronous-duration"></a>Modifier la durée asynchrone

Pour les déclencheurs et les actions, vous pouvez limiter la durée pour le modèle asynchrone à un intervalle de temps spécifique en ajoutant la propriété `limit.timeout`. De cette façon, si l’action n’est pas terminée à la fin de l’intervalle, l’état de l’action est marqué comme `Cancelled` avec le code `ActionTimedOut`. La propriété `timeout` utilise le [format ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations).

``` json
"<trigger-or-action-name>": {
   "type": "Workflow | Webhook | Http | ApiConnectionWebhook | ApiConnection",
   "inputs": {},
   "limit": {
      "timeout": "PT10S"
   },
   "runAfter": {}
}
```

<a name="runtime-config-options"></a>

## <a name="runtime-configuration-settings"></a>Paramètres de configuration d’exécution

Vous pouvez changer le comportement d’exécution par défaut pour les déclencheurs et les actions en ajoutant ces propriétés `runtimeConfiguration` à la définition de déclencheur ou d’action.

| Propriété | Type | Description | Déclencheur ou action | 
|----------|------|-------------|-------------------| 
| `runtimeConfiguration.concurrency.runs` | Integer | Modifiez la [*limite par défaut*](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits) du nombre d’instances de workflow qui peuvent s’exécuter en même temps (simultanément ou en parallèle). L’ajustement de cette valeur peut aider à limiter le nombre de requêtes reçues par les systèmes back-end. <p>L’affectation de la valeur `1` à la propriété `runs` fonctionne de la même façon que l’affectation de la valeur `SingleInstance` à la propriété `operationOptions`. Vous pouvez définir l’une ou l’autre propriété, mais pas les deux. <p>Pour modifier la limite par défaut, consultez [Changer la concurrence du déclencheur](#change-trigger-concurrency) ou [Déclencher des instances séquentiellement](#sequential-trigger). | Tous les déclencheurs | 
| `runtimeConfiguration.concurrency.maximumWaitingRuns` | Integer | Modifiez la [*limite par défaut*](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits) du nombre d’instances de workflow qui doivent attendre de s’exécuter quand votre application logique exécute déjà le nombre maximal d’instances simultanées. <p>Pour changer la limite par défaut, consultez [Changer la limite d’exécutions en attente](#change-waiting-runs). | Tous les déclencheurs | 
| `runtimeConfiguration.concurrency.repetitions` | Integer | Modifiez la [*limite par défaut*](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits) du nombre d’itérations de boucle « for each » qui peuvent s’exécuter en même temps (simultanément ou en parallèle). <p>L’affectation de la valeur `1` à la propriété `repetitions` fonctionne de la même façon que l’affectation de la valeur `SingleInstance` à la propriété `operationOptions`. Vous pouvez définir l’une ou l’autre propriété, mais pas les deux. <p>Pour changer la limite par défaut, consultez [Changer la concurrence « for each »](#change-for-each-concurrency) ou [Exécuter des boucles « for each » séquentiellement](#sequential-for-each). | Action : <p>[Foreach](#foreach-action) | 
| `runtimeConfiguration.paginationPolicy.minimumItemCount` | Integer | Pour des actions spécifiques qui prennent en charge la pagination activée et pour laquelle cette dernière est activée, cette valeur spécifie le nombre *minimal* de résultats à récupérer. <p>Pour savoir comment activer la pagination, voir [Obtenir d’autres données, des articles ou des enregistrements à l’aide de la pagination dans Azure Logic Apps](../logic-apps/logic-apps-exceed-default-page-size-with-pagination.md) | Action : Différentes possibilités |
| `runtimeConfiguration.secureData.properties` | Array | Sur de nombreux déclencheurs et actions, ces paramètres permettent de masquer les entrées et/ou les sorties dans l’historique d’exécution d’une application logique. <p>Pour en savoir plus sur la protection de ces données, consultez [Masquer les entrées et sorties de l’historique d’exécution](../logic-apps/logic-apps-securing-a-logic-app.md#secure-data-code-view). | La plupart des déclencheurs et des actions |
| `runtimeConfiguration.staticResult` | Objet JSON | Pour les actions qui prennent en charge le paramètre [Résultat statique](../logic-apps/test-logic-apps-mock-data-static-results.md) et pour lesquelles ce dernier est activé, l’objet `staticResult` possède ces attributs : <p>- `name`, qui fait référence au nom de définition du résultat statique de l’action en cours, affiché dans l’attribut `staticResults` au sein de l’attribut `definition` du flux de travail. Pour en savoir plus, consultez le schéma de référence du langage de définition du flux de travail de la section [Résultats statiques](../logic-apps/logic-apps-workflow-definition-language.md#static-results). <p> - `staticResultOptions`, qui spécifie si les résultats statiques ont la valeur `Enabled` ou non pour l’action en cours. <p>Pour en savoir plus, voir [Tester des applications logiques avec des données fictives en configurant des résultats statiques](../logic-apps/test-logic-apps-mock-data-static-results.md). | Action : Différentes possibilités |
||||| 

<a name="operation-options"></a>

## <a name="operation-options"></a>Options d’opérations

Vous pouvez changer le comportement par défaut pour les déclencheurs et les actions avec la propriété `operationOptions` dans la définition de déclencheur ou d’action.

| Option d’opération | Type | Description | Déclencheur ou action | 
|------------------|------|-------------|-------------------| 
| `DisableAsyncPattern` | String | Exécuter des actions basées sur HTTP de manière synchrone plutôt qu’asynchrone. <p><p>Pour définir cette option, consultez [Exécuter des actions de manière synchrone](#disable-asynchronous-pattern). | Actions : <p>[ApiConnection](#apiconnection-action), <br>[HTTP](#http-action), <br>[Réponse](#response-action) | 
| `IncludeAuthorizationHeadersInOutputs` | String | Pour les applications logiques qui [activent Azure Active Directory Open Authentication (Azure AD OAuth)](../logic-apps/logic-apps-securing-a-logic-app.md#enable-oauth) pour autoriser l’accès aux appels entrants au point de terminaison d’un déclencheur de requête, incluez l’en-tête `Authorization` du jeton d’accès OAuth dans les sorties du déclencheur. Pour plus d’informations, consultez [Inclure l’en-tête « Authorization » dans les sorties du déclencheur de requête](../logic-apps/logic-apps-securing-a-logic-app.md#include-auth-header). | Déclencheurs : <p>[Requête](#request-trigger), <br>[Déclencheur HTTPWebhook](#http-webhook-trigger) | 
| `Sequential` | String | Exécutez les itérations de boucle « for each » une à la fois, plutôt que toutes en même temps en parallèle. <p>Cette option fonctionne de la même façon que l’affectation de la valeur `1` à la propriété `runtimeConfiguration.concurrency.repetitions`. Vous pouvez définir l’une ou l’autre propriété, mais pas les deux. <p><p>Pour définir cette option, consultez [Exécuter des boucles « for each » séquentiellement](#sequential-for-each).| Action : <p>[Foreach](#foreach-action) | 
| `SingleInstance` | String | Exécuter le déclencheur pour chaque instance d’application logique de manière séquentielle, et attendre que l’exécution active précédente se termine avant de déclencher l’instance d’application logique suivante. <p><p>Cette option fonctionne de la même façon que l’affectation de la valeur `1` à la propriété `runtimeConfiguration.concurrency.runs`. Vous pouvez définir l’une ou l’autre propriété, mais pas les deux. <p>Pour définir cette option, consultez [Déclencher des instances séquentiellement](#sequential-trigger). | Tous les déclencheurs | 
||||

<a name="change-trigger-concurrency"></a>

### <a name="change-trigger-concurrency"></a>Changer la concurrence du déclencheur

Par défaut, les instances de workflow d’application logique s’exécutent toutes en même temps (simultanément ou en parallèle). Ce comportement signifie que chaque instance de déclencheur s’active avant la fin de l’exécution de l’instance de workflow active précédente. Toutefois, le nombre d’instances exécutées simultanément a une [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Lorsque le nombre d’instances de workflow exécutées simultanément atteint cette limite, toute autre nouvelle instance doit attendre de s’exécuter. Cette limite aide à contrôler le nombre de requêtes reçues par les systèmes backend.

Lorsque vous activez le contrôle d’accès concurrentiel du déclencheur, les instances de déclencheur s’exécutent en parallèle jusqu’à la [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Pour changer cette limite de concurrence, vous pouvez utiliser l’éditeur en mode code ou le Concepteur d’applications logiques, car la modification du paramètre de concurrence par le biais du concepteur ajoute ou met à jour la propriété `runtimeConfiguration.concurrency.runs` dans la définition de déclencheur sous-jacente et vice versa. Cette propriété contrôle le nombre maximal de nouvelles instances de workflow qui peuvent s’exécuter en parallèle.

Voici quelques considérations liées à l’activation de la concurrence sur un déclencheur :

* Lorsque la concurrence est activée, la [limite SplitOn](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits) est considérablement réduite pour la [décomposition des tableaux](#split-on-debatch). Si le nombre d'éléments dépasse cette limite, la fonction SplitOn est désactivée.

* Vous ne pouvez pas désactiver la concurrence après avoir activé le contrôle de la concurrence.

* Quand la concurrence est activée, une instance d’application logique de longue durée peut amener de nouvelles instances d’application logique à entrer dans un état d’attente. Cet état empêche Azure Logic Apps de créer des instances et se produit même quand le nombre d’exécutions simultanées est inférieur au nombre maximal spécifié d’exécutions simultanées.

  * Pour interrompre cet état, annulez les instances les plus anciennes qui sont *toujours en cours d’exécution*.

    1. Dans le menu de votre application logique, sélectionnez **Vue d’ensemble**.

    1. Dans la section **Historique des exécutions**, sélectionnez l’instance la plus ancienne qui est toujours en cours d’exécution, par exemple :

       ![Sélectionner l’instance en cours d’exécution la plus ancienne](./media/logic-apps-workflow-actions-triggers/waiting-runs.png)

       > [!TIP]
       > Pour voir uniquement les instances qui sont toujours en cours d’exécution, ouvrez la liste **Tous**, puis sélectionnez **En cours d’exécution**.

    1. Sous **Exécution d’application logique**, sélectionnez **Annuler l’exécution**.

       ![Rechercher l’instance en cours d’exécution la plus ancienne](./media/logic-apps-workflow-actions-triggers/cancel-run.png)

  * Pour contourner cette éventualité, ajoutez un délai d’expiration aux actions susceptibles de retenir ces exécutions. Si vous travaillez dans l’éditeur de code, consultez [Changer la durée asynchrone](#asynchronous-limits). Dans le cas contraire, si vous utilisez le concepteur, procédez comme suit :

    1. Dans votre application logique, sur l’action à laquelle vous souhaitez ajouter un délai d’attente, en haut à droite, sélectionnez le bouton des points de suspension ( **...** ), puis sélectionnez **Paramètres**.

       ![Ouvrir les paramètres de l’action](./media/logic-apps-workflow-actions-triggers/action-settings.png)

    1. Sous **Expiration**, spécifiez la durée du délai d’expiration au [format ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations).

       ![Spécifier la durée du délai d’expiration](./media/logic-apps-workflow-actions-triggers/timeout.png)

* Pour exécuter votre application logique séquentiellement, définissez la concurrence du déclencheur sur `1` à l’aide de l’éditeur en mode code ou du concepteur. Veillez à ne pas définir également la propriété `operationOptions` du déclencheur sur `SingleInstance` dans l’éditeur en mode code. car vous obtiendriez une erreur de validation. Pour plus d’informations, consultez [Déclencher des instances séquentiellement](#sequential-trigger).

#### <a name="edit-in-code-view"></a>Modifier en mode code 

Dans la définition de déclencheur sous-jacente, ajoutez la propriété `runtimeConfiguration.concurrency.runs`, qui peut avoir une valeur comprise entre `1` et `50`.

Voici un exemple qui limite les exécutions simultanées à 10 instances :

```json
"<trigger-name>": {
   "type": "<trigger-name>",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": 10
      }
   }
}
```

Pour plus d’informations, consultez [Paramètres de configuration d’exécution](#runtime-config-options).

#### <a name="edit-in-logic-apps-designer"></a>Modifier dans le Concepteur d’applications logiques

1. En haut à droite du déclencheur, sélectionnez le bouton représentant des points de suspension ( **...** ), puis **Paramètres**.

1. Sous **Contrôle d’accès concurrentiel**, définissez **Limite** sur **Activé**. 

1. Positionnez le curseur **Degré de parallélisme** sur la valeur souhaitée. Pour exécuter votre application logique séquentiellement, faites glisser le curseur vers la valeur **1**.

<a name="change-for-each-concurrency"></a>

### <a name="change-for-each-concurrency"></a>Changer la concurrence « for each »

Par défaut, les itérations de boucle « for each » s’exécutent toutes en même temps (simultanément ou en parallèle). Ce comportement signifie que chaque itération commence à s’exécuter avant la fin de l’exécution de l’itération précédente. Toutefois, le nombre d’itérations exécutées simultanément a une [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Lorsque le nombre d’itérations exécutées simultanément atteint cette limite, toute autre itération doit attendre de s’exécuter.

Pour changer la limite par défaut, vous pouvez utiliser l’éditeur en mode code ou le Concepteur d’applications logiques, car la modification du paramètre de concurrence par le biais du concepteur ajoute ou met à jour la propriété `runtimeConfiguration.concurrency.repetitions` dans la définition d’action « for each » sous-jacente et vice versa. Cette propriété contrôle le nombre maximal d’itérations qui peuvent s’exécuter en parallèle.

> [!NOTE] 
> Si vous configurez l’action « for each » pour une exécution séquentielle à l’aide du concepteur ou de l’éditeur en mode code, n’affectez pas la valeur `Sequential` à la propriété `operationOptions` de l’action dans l’éditeur en mode code, car vous obtiendriez une erreur de validation. Pour plus d’informations, consultez [Exécuter des boucles « for each » séquentiellement](#sequential-for-each).

#### <a name="edit-in-code-view"></a>Modifier en mode code 

Dans la définition « for each » sous-jacente, ajoutez ou mettez à jour la propriété `runtimeConfiguration.concurrency.repetitions`, qui peut avoir une valeur comprise entre `1` et `50`.

Voici un exemple qui limite les exécutions simultanées à 10 itérations :

```json
"For_each" {
   "type": "Foreach",
   "actions": { "<actions-to-run>" },
   "foreach": "<for-each-expression>",
   "runAfter": {},
   "runtimeConfiguration": {
      "concurrency": {
         "repetitions": 10
      }
   }
}
```

Pour plus d’informations, consultez [Paramètres de configuration d’exécution](#runtime-config-options).

#### <a name="edit-in-logic-apps-designer"></a>Modifier dans le Concepteur d’applications logiques

1. Dans l’angle supérieur droit de l’action **For each**, sélectionnez le bouton représentant des points de suspension ( **...** ), puis **Paramètres**.

1. Sous **Contrôle d’accès concurrentiel**, définissez **Contrôle d’accès concurrentiel** sur **Activé**.

1. Positionnez le curseur **Degré de parallélisme** sur la valeur souhaitée. Pour exécuter votre application logique séquentiellement, faites glisser le curseur vers la valeur **1**.

<a name="change-waiting-runs"></a>

### <a name="change-waiting-runs-limit"></a>Modifier la limite d’exécutions en attente

Par défaut, les instances de workflow d’application logique s’exécutent toutes en même temps (simultanément ou en parallèle). Ce comportement signifie que chaque instance de déclencheur s’active avant la fin de l’exécution de l’instance de workflow active précédente. Toutefois, le nombre d’instances exécutées simultanément a une [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Lorsque le nombre d’instances de workflow exécutées simultanément atteint cette limite, toute autre nouvelle instance doit attendre de s’exécuter.

Le nombre d’exécutions en attente a également une [limite par défaut](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Lorsque le nombre d’exécutions en attente atteint cette limite, le moteur Logic Apps n’accepte plus de nouvelles exécutions. Les requêtes et les déclencheurs webhook retournent des erreurs 429, et les déclencheurs récurrents commencent à ignorer les tentatives d’interrogation.

Non seulement vous pouvez [changer la limite par défaut de concurrence de déclencheur](#change-trigger-concurrency), mais vous pouvez aussi changer la limite par défaut des exécutions en attente. Dans la définition de déclencheur sous-jacente, ajoutez la propriété `runtimeConfiguration.concurrency.maximumWaitingRuns`, qui peut avoir une valeur comprise entre `1` et `100`.

```json
"<trigger-name>": {
   "type": "<trigger-name>",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
   },
   "runtimeConfiguration": {
      "concurrency": {
         "maximumWaitingRuns": 50
      }
   }
}
```

Pour plus d’informations, consultez [Paramètres de configuration d’exécution](#runtime-config-options).

<a name="sequential-trigger"></a>

### <a name="trigger-instances-sequentially"></a>Déclencher des instances séquentiellement

Pour exécuter chaque instance de flux de travail de l’application logique uniquement après la fin de l’exécution de l’instance précédente, configurez le déclencheur pour qu’il s’exécute de manière séquentielle. Vous pouvez utiliser l’éditeur en mode code ou le Concepteur d’applications logiques, car la modification du paramètre de concurrence par le biais du concepteur ajoute ou met également à jour la propriété `runtimeConfiguration.concurrency.runs` dans la définition de déclencheur sous-jacente et vice versa.

> [!NOTE] 
> Quand vous configurez un déclencheur pour une exécution séquentielle à l’aide du concepteur ou de l’éditeur en mode code, n’affectez pas la valeur `Sequential` à la propriété `operationOptions` du déclencheur dans l’éditeur en mode code, car vous obtiendriez une erreur de validation. 

#### <a name="edit-in-code-view"></a>Modifier en mode code

Dans la définition du déclencheur, définissez l’une ou l’autre de ces propriétés, mais pas les deux. 

Affectez la valeur `1` à la propriété `runtimeConfiguration.concurrency.runs` :

```json
"<trigger-name>": {
   "type": "<trigger-name>",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": 1
      }
   }
}
```

*-ou-*

Affectez la valeur `SingleInstance` à la propriété `operationOptions` :

```json
"<trigger-name>": {
   "type": "<trigger-name>",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
   },
   "operationOptions": "SingleInstance"
}
```

Pour plus d’informations, consultez [Paramètres de configuration d’exécution](#runtime-config-options) et [Options d’opérations](#operation-options).

#### <a name="edit-in-logic-apps-designer"></a>Modifier dans le Concepteur d’applications logiques

1. En haut à droite du déclencheur, sélectionnez le bouton représentant des points de suspension ( **...** ), puis **Paramètres**.

1. Sous **Contrôle d’accès concurrentiel**, définissez **Limite** sur **Activé**. 

1. Positionnez le curseur **Degré de parallélisme** sur le chiffre `1`. 

<a name="sequential-for-each"></a>

### <a name="run-for-each-loops-sequentially"></a>Exécuter des boucles « for each » séquentiellement

Pour exécuter une itération de boucle « for each » uniquement après la fin de l’exécution de l’itération précédente, configurez l’action « for each » pour qu’elle s’exécute de manière séquentielle. Vous pouvez utiliser l’éditeur en mode code ou le Concepteur d’applications logiques, car la modification de la concurrence de l’action par le biais du concepteur ajoute ou met également à jour la propriété `runtimeConfiguration.concurrency.repetitions` dans la définition d’action sous-jacente et vice versa.

> [!NOTE] 
> Quand vous configurez une action « for each » pour une exécution séquentielle à l’aide du concepteur ou de l’éditeur en mode code, n’affectez pas la valeur `Sequential` à la propriété `operationOptions` de l’action dans l’éditeur en mode code, car vous obtiendriez une erreur de validation. 

#### <a name="edit-in-code-view"></a>Modifier en mode code

Dans la définition de l’action, définissez l’une ou l’autre de ces propriétés, mais pas les deux. 

Affectez la valeur `1` à la propriété `runtimeConfiguration.concurrency.repetitions` :

```json
"For_each" {
   "type": "Foreach",
   "actions": { "<actions-to-run>" },
   "foreach": "<for-each-expression>",
   "runAfter": {},
   "runtimeConfiguration": {
      "concurrency": {
         "repetitions": 1
      }
   }
}
```

*-ou-*

Affectez la valeur `Sequential` à la propriété `operationOptions` :

```json
"For_each" {
   "type": "Foreach",
   "actions": { "<actions-to-run>" },
   "foreach": "<for-each-expression>",
   "runAfter": {},
   "operationOptions": "Sequential"
}
```

Pour plus d’informations, consultez [Paramètres de configuration d’exécution](#runtime-config-options) et [Options d’opérations](#operation-options).

#### <a name="edit-in-logic-apps-designer"></a>Modifier dans le Concepteur d’applications logiques

1. En haut à droite de l’action **For each**, sélectionnez le bouton représentant des points de suspension ( **...** ), puis **Paramètres**.

1. Sous **Contrôle d’accès concurrentiel**, définissez **Contrôle d’accès concurrentiel** sur **Activé**.

1. Positionnez le curseur **Degré de parallélisme** sur le chiffre `1`.

<a name="disable-asynchronous-pattern"></a>

### <a name="run-actions-in-a-synchronous-operation-pattern"></a>Exécuter des actions dans un modèle d’opération synchrone

Par défaut, l’action HTTP et les actions APIConnection dans Azure Logic Apps suivent le [*modèle d’opération asynchrone*](/azure/architecture/patterns/async-request-reply) standard, alors que l’action Response suit le *modèle d’opération synchrone*. Le modèle asynchrone spécifie qu’une fois qu’une action a appelé ou envoyé une requête au point de terminaison, au service, au système ou à l’API spécifiée, le récepteur retourne immédiatement une réponse [« 202 ACCEPTED »](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.2.3). Ce code confirme que le récepteur a accepté la requête, mais qu’il n’a pas fini le traitement. La réponse peut inclure un en-tête `location` qui spécifie l’URL et un ID d’actualisation que l’appelant peut utiliser pour interroger ou vérifier en permanence l’état de la requête asynchrone jusqu’à ce que le récepteur arrête le traitement et retourne la réponse de réussite [« 200 OK »](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.2.1) ou une réponse autre qu’une réponse 202. Pour plus d’informations, consultez [L’intégration asynchrone des microservices permet l’autonomie des microservices](/azure/architecture/microservices/design/interservice-communication#synchronous-versus-asynchronous-messaging).

* Dans le concepteur d’application logique, l’action HTTP, les actions APIConnection et l’action Response ont le paramètre **Modèle asynchrone**. Quand il est activé, ce paramètre spécifie que l’appelant n’a pas à attendre la fin du traitement et qu’il peut passer à l’action suivante, tout en continuant de vérifier l’état jusqu’à ce que le traitement s’arrête. S’il est désactivé, ce paramètre spécifie que l’appelant doit attendre la fin du traitement avant de passer à l’action suivante. Pour trouver ce paramètre, effectuez les étapes suivantes :

  1. Dans la barre de titre de l’action HTTP, sélectionnez le bouton de sélection ( **...** ) qui permet d’ouvrir les paramètres de l’action.

  1. Recherchez le paramètre **Modèle asynchrone**.

     ![Paramètre « Modèle asynchrone »](./media/logic-apps-workflow-actions-triggers/asynchronous-pattern-setting.png)

* Dans la définition JSON (JavaScript Object Notation) sous-jacente de l’action, l’action HTTP et les actions APIConnection suivent implicitement le modèle d’opération asynchrone.

Dans certains scénarios, vous pouvez souhaiter qu’une action suive plutôt le modèle synchrone. Par exemple, quand vous utilisez l’action HTTP, vous souhaitez peut-être :

* [Éviter les expirations de délai d’attente HTTP pour les tâches de longue durée](../connectors/connectors-native-http.md#avoid-http-timeouts)
* [Désactiver la vérification des en-têtes d’emplacement](../connectors/connectors-native-http.md#disable-location-header-check)

Dans ce cas, vous pouvez exécuter une action de manière synchrone en procédant comme indiqué ci-dessous :

* Remplacez la version d’interrogation de l’action par une version de Webhook, le cas échéant.

* Désactivez le comportement asynchrone de l’action en procédant de l’une des façons suivantes :

  * Dans concepteur d’application logique, [désactivez le paramètre **Modèle asynchrone**](#turn-off-asynchronous-pattern-setting).

  * Dans la définition JSON sous-jacente de l’action, [ajoutez l’option d’opération `"DisableAsyncPattern"`](#add-disable-async-pattern-option).

<a name="turn-off-asynchronous-pattern-setting"></a>

#### <a name="turn-off-asynchronous-pattern-setting"></a>Désactiver le paramètre **Modèle asynchrone**

1. Dans le concepteur d’application logique, dans la barre de titre de l’action, sélectionnez le bouton de sélection ( **...** ) qui permet d’ouvrir les paramètres de l’action.

1. Recherchez le paramètre **Modèle asynchrone**, configurez-le sur **Désactivé** s’il est activé, puis sélectionnez **Terminé**.

   ![Désactiver le paramètre « Modèle asynchrone »](./media/logic-apps-workflow-actions-triggers/disable-asynchronous-pattern-setting.png)

<a name="add-disable-async-pattern-option"></a>

#### <a name="disable-asynchronous-pattern-in-actions-json-definition"></a>Désactiver le modèle asynchrone dans la définition JSON de l’action

Dans la définition JSON sous-jacente de l’action, ajoutez la [propriété « operationOptions »](#operation-options) et affectez-lui la valeur `"DisableAsyncPattern"` sous la section `"inputs"` de l’action, par exemple :

```json
"<some-long-running-action>": {
   "type": "Http",
   "inputs": { "<action-inputs>" },
   "operationOptions": "DisableAsyncPattern",
   "runAfter": {}
}
```

<a name="authenticate-triggers-actions"></a>

## <a name="authenticate-triggers-and-actions"></a>Authentifier les déclencheurs et les actions

Les points de terminaison HTTP et HTTPS prennent en charge différents types d’authentification. En fonction du déclencheur ou de l’action que vous utilisez pour effectuer des appels sortants ou des demandes qui accèdent à ces points de terminaison, vous pouvez choisir parmi différentes plages de types d’authentification. Pour plus d’informations, voir [Ajouter l’authentification aux appels sortants](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur le [langage de définition de workflow](../logic-apps/logic-apps-workflow-definition-language.md)
