---
title: Envoyer des notifications Azure Service Health par le biais de webhooks
description: Envoyez des notifications personnalisées sur les événements d’intégrité de service à votre système de gestion des problèmes existants.
ms.topic: conceptual
ms.service: service-health
ms.date: 3/27/2018
ms.openlocfilehash: 05b0572c89a29fddc881f9977ee437d1319e6254
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86518926"
---
# <a name="use-a-webhook-to-configure-health-notifications-for-problem-management-systems"></a>Utiliser un webhook afin de configurer les notifications d’intégrité pour les systèmes de gestion des problèmes

Cet article vous montre comment configurer les alertes Azure Service Health pour envoyer des données via des webhooks à votre système de notification existant.

Vous pouvez configurer des alertes Service Health pour recevoir une notification par SMS ou par e-mail quand un incident lié à un service Azure vous concerne.

Toutefois, vous avez peut-être déjà en place un système de notification externe que vous préférez utiliser. Cet article identifie les parties les plus importantes de la charge utile du webhook. Il explique également comment créer des alertes personnalisées pour recevoir une notification en cas de problèmes liés à des services qui vous concernent.

Si vous souhaitez utiliser une intégration préconfigurée, consultez :
* [Configurer des alertes avec ServiceNow](service-health-alert-webhook-servicenow.md)
* [Configurer des alertes avec PagerDuty](service-health-alert-webhook-pagerduty.md)
* [Configurer des alertes avec OpsGenie](service-health-alert-webhook-opsgenie.md)

**Regarder une vidéo d’introduction :**

>[!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2OtUV]

## <a name="configure-a-custom-notification-by-using-the-service-health-webhook-payload"></a>Configurer une notification personnalisée à l’aide de la charge utile du webhook Service Health
Pour configurer l’intégration de votre propre webhook personnalisé, vous devez analyser la charge utile JSON envoyée via la notification Service Health.

Consultez [un exemple](../azure-monitor/platform/activity-log-alerts-webhook.md) de charge utile de webhook `ServiceHealth`.

Vous pouvez vérifier qu’il s’agit d’une alerte Service Health en examinant `context.eventSource == "ServiceHealth"`. Les propriétés suivantes sont les plus pertinentes :
- **data.context.activityLog.status**
- **data.context.activityLog.level**
- **data.context.activityLog.subscriptionId**
- **data.context.activityLog.properties.title**
- **data.context.activityLog.properties.impactStartTime**
- **data.context.activityLog.properties.communication**
- **data.context.activityLog.properties.impactedServices**
- **data.context.activityLog.properties.trackingId**

## <a name="create-a-link-to-the-service-health-dashboard-for-an-incident"></a>Créer un lien vers le tableau de bord Service Health pour un incident
Vous pouvez créer un lien direct vers votre tableau de bord Service Health sur un poste de travail ou un appareil mobile en générant une URL spécialisée. Utilisez le *trackingId* ainsi que les trois premiers et les trois derniers chiffres de votre *subscriptionId* au format suivant :

https<i></i>://app.azure.com/h/ *&lt;trackingId&gt;* / *&lt;trois premiers et trois derniers chiffres de subscriptionId&gt;*

Par exemple, si votre *subscriptionId* est bba14129-e895-429b-8809-278e836ecdb3 et si votre *trackingId* est 0DET-URB, votre URL Service Health est :

https<i></i>://app.azure.com/h/0DET-URB/bbadb3

## <a name="use-the-level-to-detect-the-severity-of-the-issue"></a>Utiliser la propriété level pour détecter le niveau de gravité du problème
Du niveau de gravité le plus faible au plus élevé, la propriété **level** de la charge utile peut avoir la valeur *Informational*, *Warning*, *Error* ou *Critical*.

## <a name="parse-the-impacted-services-to-determine-the-incident-scope"></a>Analyser les services impactés pour déterminer l’étendue de l’incident
Les alertes Service Health peuvent vous informer sur les problèmes rencontrés dans plusieurs régions et services. Pour obtenir tous les détails, vous devez analyser la valeur de `impactedServices`.

Le contenu est une chaîne [JSON](https://json.org/) faisant l’objet d’une séquence d’échappement. En l’absence de séquence d’échappement, le contenu correspond à un autre objet JSON qui peut être analysé de manière régulière. Par exemple :

```json
{"data.context.activityLog.properties.impactedServices": "[{\"ImpactedRegions\":[{\"RegionName\":\"Australia East\"},{\"RegionName\":\"Australia Southeast\"}],\"ServiceName\":\"Alerts & Metrics\"},{\"ImpactedRegions\":[{\"RegionName\":\"Australia Southeast\"}],\"ServiceName\":\"App Service\"}]"}
```

devient :

```json
[
   {
      "ImpactedRegions":[
         {
            "RegionName":"Australia East"
         },
         {
            "RegionName":"Australia Southeast"
         }
      ],
      "ServiceName":"Alerts & Metrics"
   },
   {
      "ImpactedRegions":[
         {
            "RegionName":"Australia Southeast"
         }
      ],
      "ServiceName":"App Service"
   }
]
```

Cet exemple illustre des problèmes qui se produisent pour :
- « Alerts & Metrics » (Alertes et métriques) dans les régions Australie Est et Australie Sud-Est
- « App Service » dans la région Australie Sud-Est

## <a name="test-your-webhook-integration-via-an-http-post-request"></a>Tester l’intégration de votre Webhook via une requête HTTP POST

Procédez comme suit :

1. Créez la charge utile Service Health à envoyer. Vous trouverez un exemple de charge utile de Webhook Service Health dans [Webhook des alertes du journal d’activité Azure](../azure-monitor/platform/activity-log-alerts-webhook.md).

1. Créez une requête HTTP POST comme suit :

    ```
    POST        https://your.webhook.endpoint

    HEADERS     Content-Type: application/json

    BODY        <service health payload>
    ```
   Vous devez recevoir une réponse « 2XX - Successful » (2XX - Réussite).

1. Accédez à [PagerDuty](https://www.pagerduty.com/) pour confirmer que votre intégration a été définie avec succès.

## <a name="next-steps"></a>Étapes suivantes
- Consultez le [schéma webhook des alertes de journal d’activité](../azure-monitor/platform/activity-log-alerts-webhook.md). 
- En savoir plus sur les [notifications sur l’intégrité du service](./service-notifications.md).
- En savoir plus sur les [groupes d’actions](../azure-monitor/platform/action-groups.md).
