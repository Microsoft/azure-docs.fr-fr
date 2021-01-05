---
title: Alertes en cas de problèmes dans Azure Cloud Services à l’aide de l’intégration Diagnostics Azure avec Azure Application Insights | Microsoft Docs
description: Surveillez les problèmes tels que les échecs de démarrage, les incidents et les boucles de recyclage des rôles dans Azure Cloud Services avec Azure Application Insights
ms.topic: conceptual
ms.date: 06/07/2018
ms.reviewer: harelbr
ms.openlocfilehash: 1cdfc6dc3ac74997743512ee07f9293699e3ad10
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87309288"
---
# <a name="alert-on-issues-in-azure-cloud-services-using-the-azure-diagnostics-integration-with-azure-application-insights"></a>Alertes en cas de problèmes dans Azure Cloud Services à l’aide de l’intégration Diagnostics Azure avec Azure Application Insights

Dans cet article, nous allons voir comment configurer les règles d’alerte qui surveillent les problèmes tels que les échecs de démarrage, les incidents et les boucles de recyclage des rôles dans Azure Cloud Services (rôles web et de travail).

La méthode décrite dans cet article est basée sur l’[intégration Diagnostics Azure avec Application Insights](https://azure.microsoft.com/blog/azure-diagnostics-integration-with-application-insights/)et la fonctionnalité [Alertes de journal pour Application Insights](https://azure.microsoft.com/blog/log-alerts-for-application-insights-preview/) publiée récemment.

## <a name="define-a-base-query"></a>Définir une requête de base

Pour commencer, nous allons définir une requête de base qui extrait les événements du journal des événements Windows du canal Windows Azure, qui sont capturés dans Application Insights en tant qu’enregistrements de suivi.
Ces enregistrements peuvent être utilisés pour détecter divers problèmes dans Azure Cloud Services, tels que les échecs de démarrage, les problèmes de runtime et les boucles de recyclage.

> [!NOTE]
> La requête de base ci-dessous recherche les problèmes dans une fenêtre de temps de 30 minutes et suppose une latence de 10 minutes pour la réception des enregistrements de télémétrie. Ces valeurs par défaut peuvent être configurées selon vos besoins.

```
let window = 30m;
let endTime = ago(10m);
let EventLogs = traces
| where timestamp > endTime - window and timestamp < endTime
| extend channel = tostring(customDimensions.Channel), eventId = tostring(customDimensions.EventId)
| where channel == 'Windows Azure' and isnotempty(eventId)
| where tostring(customDimensions.DeploymentName) !contains 'deployment' // discard records captured from local machines
| project timestamp, channel, eventId, message, cloud_RoleInstance, cloud_RoleName, itemCount;
```

## <a name="check-for-specific-event-ids"></a>Rechercher des ID d’événement spécifiques

Après avoir récupéré les événements du journal des événements Windows, des problèmes spécifiques peuvent être détectés en recherchant leurs ID d’événement et propriétés de message respectives (voir les exemples ci-dessous).
Combinez simplement la requête de base ci-dessus avec l’une des requêtes ci-dessous, et utilisez cette requête combinée lors de la définition de la règle d’alerte de journal.

> [!NOTE]
> Dans les exemples ci-dessous, un problème sera détecté si plus de trois événements sont trouvés pendant la fenêtre de temps analysée. Cette valeur par défaut peut être configurée pour modifier la sensibilité de la règle d’alerte.

```
// Detect failures in the OnStart method
EventLogs
| where eventId == '2001'
| where message contains '.OnStart()'
| summarize Failures = sum(itemCount) by cloud_RoleInstance, cloud_RoleName
| where Failures > 3
```

```
// Detect failures during runtime
EventLogs
| where eventId == '2001'
| where message contains '.Run()'
| summarize Failures = sum(itemCount) by cloud_RoleInstance, cloud_RoleName
| where Failures > 3
```

```
// Detect failures when running a startup task
EventLogs
| where eventId == '1000'
| summarize Failures = sum(itemCount) by cloud_RoleInstance, cloud_RoleName
| where Failures > 3
```

```
// Detect recycle loops
EventLogs
| where eventId == '1006'
| summarize Failures = sum(itemCount) by cloud_RoleInstance, cloud_RoleName
| where Failures > 3
```

## <a name="create-an-alert"></a>Créer une alerte

Dans le menu de navigation de votre ressource Application Insights, accédez à **Alertes**, puis sélectionnez **Nouvelle règle d’alerte**.

![Capture d’écran de Créer une règle](./media/proactive-cloud-services/001.png)

Dans la fenêtre **Créer une règle**, dans la section **Définir la condition de l’alerte**, cliquez sur **Ajouter des critères**, puis sélectionnez **Recherche de journal personnalisée**.

![Capture d’écran de Définir les critères de condition de l’alerte](./media/proactive-cloud-services/002.png)

Dans la zone **Rechercher la requête**, collez la requête combinée que vous avez préparée à l’étape précédente.

Ensuite, passez à la zone **Seuil** et définissez sa valeur sur 0. Vous pouvez éventuellement régler les **champs** **Période** et Fréquence.
Cliquez sur **Done**.

![Capture d’écran de Configurer la requête de la logique du signal](./media/proactive-cloud-services/003.png)

Dans la section **Définir les détails de l’alerte**, donnez un **Nom** et une **Description** à la règle d’alerte, et définissez sa **Gravité**.
En outre, assurez-vous que le bouton **Activer la règle lors de sa création** est défini sur **Oui**.

![Capture d’écran de Détails de l’alerte](./media/proactive-cloud-services/004.png)

Dans la section **Définir un groupe d’actions**, vous pouvez sélectionner un **groupe d’actions** existant ou en créez un.
Vous pouvez faire en sorte que le groupe d’actions contienne plusieurs actions de différents types.

![Capture d’écran de Groupe d’actions](./media/proactive-cloud-services/005.png)

Une fois que vous avez défini le groupe d’actions, confirmez vos modifications, puis cliquez sur **Créer une règle d’alerte**.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la détection automatique :

[Défaillances](./proactive-failure-diagnostics.md)
[Fuites de mémoire](./proactive-potential-memory-leak.md)
[Anomalies de performance](./proactive-performance-diagnostics.md)

