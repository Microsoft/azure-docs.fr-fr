---
title: Concepts, termes et entités
description: Découvrez les concepts, la terminologie et la hiérarchie des entités, notamment les travaux et les collections de travaux, d’Azure Scheduler
services: scheduler
ms.service: scheduler
ms.suite: infrastructure-services
author: derek1ee
ms.author: deli
ms.reviewer: klam, estfan
ms.topic: conceptual
ms.date: 08/18/2016
ms.openlocfilehash: 899c64e818896cde18e955d6abd82594734c4b57
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92368160"
---
# <a name="concepts-terminology-and-entities-in-azure-scheduler"></a>Concepts, terminologie et entités d’Azure Scheduler

> [!IMPORTANT]
> [Azure Logic Apps](../logic-apps/logic-apps-overview.md) remplace Azure Scheduler, qui est en phase de [mise hors service](../scheduler/migrate-from-scheduler-to-logic-apps.md#retire-date). Pour poursuivre les travaux que vous avez configurés dans Scheduler, veuillez [migrer vers Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md) dès que possible. 
>
> Scheduler n’est plus disponible dans le portail Azure, mais l’[API REST](/rest/api/scheduler) et les [applets de commande Azure Scheduler PowerShell](scheduler-powershell-reference.md) restent disponibles pour vous permettre de gérer vos travaux et collections de travaux.

## <a name="entity-hierarchy"></a>Hiérarchie des entités

L’API REST d’Azure Scheduler expose et utilise ces entités principales, ou ces ressources :

| Entité | Description |
|--------|-------------|
| **Travail** | Définit une action récurrente avec des stratégies d’exécution simples ou complexes. Les actions peuvent inclure des demandes HTTP, de file d’attente de stockage, de file d’attente Service Bus ou de rubrique Service Bus. | 
| **Collection de travaux** | Contient un groupe de travaux et conserve les paramètres, les quotas et les limitations qui sont partagés par les travaux de la collection. En tant que propriétaire d’un abonnement Azure, vous pouvez créer des collections de travaux et regrouper des travaux en fonction des limites de l’utilisation ou de l’application. Une collection de travaux a ces attributs : <p>- Limitée à une seule région. <br>- Vous permet d’appliquer des quotas de façon à limiter l’utilisation pour tous les travaux d’une collection. <br>- Les quotas incluent MaxJobs et MaxRecurrence. | 
| **Historique des travaux** | Décrit les détails d’exécution d’un travail, par exemple l’état et les détails de la réponse. |
||| 

## <a name="entity-management"></a>Gestion des identités

D’une façon générale, l’API REST de Scheduler expose ces opérations pour la gestion des entités.

### <a name="job-management"></a>Gestion des travaux

Prend en charge les opérations de création et de modification des travaux. Tous les travaux doivent appartenir à une collection de travaux existante ; il n’y a donc pas de création implicite. Pour plus d’informations, consultez [API REST de Scheduler - Travaux](/rest/api/scheduler/jobs). Voici l’adresse URI pour ces opérations :

```
https://management.azure.com/subscriptions/{subscriptionID}/resourceGroups/{resourceGroupName}/providers/Microsoft.Scheduler/jobCollections/{jobCollectionName}/jobs/{jobName}
```

### <a name="job-collection-management"></a>Gestion de la collection de travaux

Prend en charge les opérations de création et de modification des travaux et des collections de travaux, qui correspondent aux quotas et aux paramètres partagés. Par exemple, les quotas spécifient le nombre maximal de travaux et le plus petit intervalle de récurrence. Pour plus d’informations, consultez [API REST de Scheduler - Collections de travaux](/rest/api/scheduler/jobcollections). Voici l’adresse URI pour ces opérations :

```
https://management.azure.com/subscriptions/{subscriptionID}/resourceGroups/{resourceGroupName}/providers/Microsoft.Scheduler/jobCollections/{jobCollectionName}
```

### <a name="job-history-management"></a>Gestion de l’historique des travaux

Prend en charge l’opération GET pour l’extraction de 60 jours d’historique d’exécution des travaux, par exemple le temps écoulé pour le travail et les résultats d’exécution du travail. Inclut la prise en charge du paramètre de chaîne de requête pour le filtrage basé sur l’état et le statut. Pour plus d’informations, consultez [API REST de Scheduler - Lister l’historique des travaux](/rest/api/scheduler/jobs/listjobhistory). Voici l’adresse URI pour cette opération :

```
https://management.azure.com/subscriptions/{subscriptionID}/resourceGroups/{resourceGroupName}/providers/Microsoft.Scheduler/jobCollections/{jobCollectionName}/jobs/{jobName}/history
```

## <a name="job-types"></a>Types de travaux

Azure Scheduler prend en charge plusieurs types de travaux : 

* Travaux HTTP, notamment les travaux HTTPS qui prennent en charge TLS, quand vous avez le point de terminaison pour un service existant ou une charge de travail existante
* Travaux de file d’attente de stockage pour les charges de travail qui utilisent des files d’attente de stockage, comme l’envoi de messages à des files d’attente de stockage
* Travaux de file d’attente Service Bus pour les charges de travail qui utilisent des files d’attente Service Bus
* Travaux de rubrique Service Bus pour les charges de travail qui utilisent des rubriques Service Bus

## <a name="job-definition"></a>Définition du travail

D’une façon générale, un travail du planificateur comprend les éléments de base suivants :

* L’action qui s’exécute quand le minuteur du travail se déclenche
* Facultatif : l’heure à laquelle exécuter le travail
* Facultatif : quand et à quelle fréquence répéter le travail
* Facultatif : une action d’erreur qui s’exécute en cas d’échec de l’action principale

Le travail inclut également des données fournies par le système, comme l’heure d’exécution planifiée suivante du travail. La définition du code du travail est un objet au format JSON (JavaScript Objet Notation), qui comporte ces éléments :

| Élément | Obligatoire | Description | 
|---------|----------|-------------| 
| [**startTime**](#start-time) | Non | Heure de début du travail avec un décalage de fuseau horaire au [format ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) | 
| [**action**](#action) | Oui | Détails de l’action principale, qui peuvent inclure un objet **errorAction** | 
| [**errorAction**](#error-action) | Non | Détails de l’action secondaire qui s’exécute si l’action principale échoue |
| [**recurrence**](#recurrence) | Non | Détails tels que la fréquence et l’intervalle pour un travail périodique | 
| [**retryPolicy**](#retry-policy) | Non | Détails sur la fréquence à laquelle réessayer une action | 
| [**state**](#state) | Oui | Détails de l’état actuel du travail |
| [**status**](#status) | Oui | Détails de l’état actuel du travail, qui est contrôlé par le service |
||||

Voici un exemple qui montre une définition complète d’un travail pour une action HTTP avec des éléments détaillés plus complets, décrits dans les sections suivantes : 

```json
"properties": {
   "startTime": "2012-08-04T00:00Z",
   "action": {
      "type": "Http",
      "request": {
         "uri": "http://contoso.com/some-method", 
         "method": "PUT",          
         "body": "Posting from a timer",
         "headers": {
            "Content-Type": "application/json"
         },
         "retryPolicy": { 
             "retryType": "None" 
         },
      },
      "errorAction": {
         "type": "Http",
         "request": {
            "uri": "http://contoso.com/notifyError",
            "method": "POST"
         }
      }
   },
   "recurrence": {
      "frequency": "Week",
      "interval": 1,
      "schedule": {
         "weekDays": ["Monday", "Wednesday", "Friday"],
         "hours": [10, 22]
      },
      "count": 10,
      "endTime": "2012-11-04"
   },
   "state": "Disabled",
   "status": {
      "lastExecutionTime": "2007-03-01T13:00:00Z",
      "nextExecutionTime": "2007-03-01T14:00:00Z ",
      "executionCount": 3,
      "failureCount": 0,
      "faultedCount": 0
   }
}
```

<a name="start-time"></a>

## <a name="starttime"></a>startTime

Dans l’objet **startTime** , vous pouvez spécifier l’heure de début et un décalage de fuseau horaire au [format ISO 8601](https://en.wikipedia.org/wiki/ISO_8601).

<a name="action"></a>

## <a name="action"></a>action

Votre travail du planificateur exécute une **action** en fonction de la planification spécifiée. Scheduler prend en charge des actions HTTP, de file d’attente de stockage, de file d’attente Service Bus et de rubrique Service Bus. Si **l’action** principale échoue, Scheduler peut exécuter une action [**errorAction**](#erroraction) secondaire qui gère l’erreur. L’objet **action** décrit ces éléments :

* Type de service de l’action
* Détails de l’action
* Une action **errorAction** alternative

L’exemple précédent décrit une action HTTP. Voici un exemple d’action de file d’attente de stockage :

```json
"action": {
   "type": "storageQueue",
   "queueMessage": {
      "storageAccount": "myStorageAccount",  
      "queueName": "myqueue",                
      "sasToken": "TOKEN",                   
      "message": "My message body"
    }
}
```

Voici un exemple d’action de file d’attente Service Bus :

```json
"action": {
   "type": "serviceBusQueue",
   "serviceBusQueueMessage": {
      "queueName": "q1",  
      "namespace": "mySBNamespace",
      "transportType": "netMessaging", // Either netMessaging or AMQP
      "authentication": {  
         "sasKeyName": "QPolicy",
         "type": "sharedAccessKey"
      },
      "message": "Some message",  
      "brokeredMessageProperties": {},
      "customMessageProperties": {
         "appname": "FromScheduler"
      }
   }
},
```

Voici un exemple d’action de rubrique Service Bus :

```json
"action": {
   "type": "serviceBusTopic",
   "serviceBusTopicMessage": {
      "topicPath": "t1",  
      "namespace": "mySBNamespace",
      "transportType": "netMessaging", // Either netMessaging or AMQP
      "authentication": {
         "sasKeyName": "QPolicy",
         "type": "sharedAccessKey"
      },
      "message": "Some message",
      "brokeredMessageProperties": {},
      "customMessageProperties": {
         "appname": "FromScheduler"
      }
   }
},
```

Pour plus d’informations sur les signatures d’accès partagé, consultez [Autoriser avec des signatures d’accès partagé](../storage/common/storage-sas-overview.md).

<a name="error-action"></a>

## <a name="erroraction"></a>errorAction

Si **l’action** principale de votre travail échoue, Scheduler peut exécuter une action **errorAction** qui gère l’erreur. Dans **l’action** principale, vous pouvez spécifier un objet **errorAction** , ce qui permet à Scheduler d’appeler un point de terminaison de gestion des erreurs ou d’envoyer une notification à l’utilisateur. 

Par exemple, si un incident se produit au point de terminaison principal, vous pouvez utiliser **errorAction** pour appeler un point de terminaison secondaire ou pour envoyer une notification à un point de terminaison de gestion des erreurs. 

Comme pour **l’action** principale, l’action d’erreur peut utiliser une logique simple ou composite basée sur d’autres actions. 

<a name="recurrence"></a>

## <a name="recurrence"></a>recurrence

Un travail se répète si la définition JSON du travail inclut l’objet **recurrence** , par exemple :

```json
"recurrence": {
   "frequency": "Week",
   "interval": 1,
   "schedule": {
      "hours": [10, 22],
      "minutes": [0, 30],
      "weekDays": ["Monday", "Wednesday", "Friday"]
   },
   "count": 10,
   "endTime": "2012-11-04"
},
```

| Propriété | Obligatoire | Valeur | Description | 
|----------|----------|-------|-------------| 
| **frequency** | Oui, quand **recurrence** est utilisé | "Minute", "Hour", "Day", "Week", "Month", "Year" | Unité de temps entre les occurrences | 
| **interval** | Non | 1 à 1000 (inclusivement) | Entier positif qui détermine le nombre d’unités de temps entre chaque occurrence, en fonction de **frequency** | 
| **schedule** | Non | Variable | Détails pour les planifications avancées et plus complexes. Voir **hours** , **minutes** , **weekDays** , **months** et **monthDays** | 
| **hours** | Non | 1 à 24 | Tableau avec les marques d’heures pour le moment où exécuter le travail | 
| **minutes** | Non | 0 à 59 | Tableau avec les marques de minutes pour le moment où exécuter le travail | 
| **months** | Non | 1 à 12 | Tableau avec les mois pour le moment où exécuter le travail | 
| **monthDays** | Non | Variable | Tableau avec les jours du mois pour le moment où exécuter le travail | 
| **weekDays** | Non | "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday" | Tableau avec les jours de la semaine pour le moment où exécuter le travail | 
| **count** | Non | <*none*> | Nombre de récurrences. Par défaut, le nombre de récurrences est infini. Vous ne pouvez pas utiliser à la fois **count** et **endTime** , mais la règle qui se termine en premier est honorée. | 
| **endTime** | Non | <*none*> | La date et l’heure d’arrêt de la récurrence. Par défaut, le nombre de récurrences est infini. Vous ne pouvez pas utiliser à la fois **count** et **endTime** , mais la règle qui se termine en premier est honorée. | 
||||

Pour plus d’informations sur ces éléments, consultez [Créer des planifications complexes et des récurrences avancées](../scheduler/scheduler-advanced-complexity.md).

<a name="retry-policy"></a>

## <a name="retrypolicy"></a>retryPolicy

Dans le cas où un travail du planificateur échoue, vous pouvez configurer une stratégie de nouvelle tentative, qui détermine si et comment le planificateur retente l’action. Par défaut, Scheduler tente de l’exécuter encore quatre fois, à des intervalles de 30 secondes. Vous pouvez rendre cette stratégie plus ou moins agressive. Par exemple, cette stratégie réessaie une action deux fois par jour :

```json
"retryPolicy": { 
   "retryType": "Fixed",
   "retryInterval": "PT1D",
   "retryCount": 2
},
```

| Propriété | Obligatoire | Valeur | Description | 
|----------|----------|-------|-------------| 
| **retryType** | Oui | **Fixed** , **None** | Détermine si vous spécifiez une stratégie de nouvelle tentative ( **fixed** ) ou non ( **none** ). | 
| **retryInterval** | Non | PT30S | Spécifie l’intervalle et la fréquence entre les nouvelles tentatives au [format ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations). La valeur minimale est 15 secondes, alors que la valeur maximale est 18 mois. | 
| **retryCount** | Non | 4 | Spécifie le nombre de nouvelles tentatives. La valeur maximale est 20. | 
||||

Pour plus d’informations, consultez [Haute disponibilité et fiabilité](../scheduler/scheduler-high-availability-reliability.md).

<a name="status"></a>

## <a name="state"></a>state

L’état d’un travail est **Enabled** (Activé), **Disabled** (Désactivé), **Completed** (Terminé) ou **Faulted** (En erreur). Par exemple : 

`"state": "Disabled"`

Pour changer l’état des travaux en **Enabled** ou **Disabled** , vous pouvez utiliser l’opération PUT ou PATCH sur ces travaux.
Cependant, si un travail a l’état **Completed** ou **Faulted** , vous ne pouvez pas mettre à jour l’état ; vous pouvez cependant effectuer l’opération DELETE sur le travail. Scheduler supprime les travaux terminés et en erreur après 60 jours. 

<a name="status"></a>

## <a name="status"></a>status

Après le démarrage d’un travail, Scheduler retourne des informations sur l’état du travail via l’objet **status** , que seul Scheduler contrôle. Vous pouvez cependant trouver l’objet **status** à l’intérieur de l’objet **job** . Voici les informations incluses dans le statut d’un travail :

* Heure de l’exécution précédente, le cas échéant
* Heure de la prochaine exécution planifiée pour les travaux en cours
* Nombre d’exécutions du travail
* Nombre d’échecs, le cas échéant
* Nombre d’erreurs, le cas échéant

Par exemple :

```json
"status": {
   "lastExecutionTime": "2007-03-01T13:00:00Z",
   "nextExecutionTime": "2007-03-01T14:00:00Z ",
   "executionCount": 3,
   "failureCount": 0,
   "faultedCount": 0
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Créer des planifications complexes et des récurrences avancées](scheduler-advanced-complexity.md)
* [Informations de référence sur l’API REST d’Azure Scheluler](/rest/api/scheduler)
* [Informations de référence sur les applets de commande PowerShell d’Azure Scheluler](scheduler-powershell-reference.md)
* [Limites, quotas, valeurs par défaut et codes d’erreur](scheduler-limits-defaults-errors.md)