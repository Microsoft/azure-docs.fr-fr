---
title: Schémas des alertes Azure Security Center
description: Cet article décrit les différents schémas utilisés par Azure Security Center pour les alertes de sécurité.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/19/2020
ms.author: memildin
ms.openlocfilehash: f9b3be69ab57c0abf7523169303def899f325229
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92789214"
---
# <a name="security-alerts-schemas"></a>Schémas des alertes de sécurité

Si Azure Defender est activé pour votre abonnement, vous recevez des alertes de sécurité lorsque Security Center détecte des menaces pour leurs ressources.

Vous pouvez consulter ces alertes de sécurité sur les pages **Protection contre les menaces** d’Azure Security Center, ou via des outils externes tels que :

- [Azure Sentinel](../sentinel/index.yml) : SIEM Microsoft natif dans le cloud. Le connecteur Sentinel reçoit les alertes d’Azure Security Center et les envoie à l’[espace de travail Log Analytics](../azure-monitor/learn/quick-create-workspace.md) pour Azure Sentinel.
- SIEM tiers : Envoyer des données à [Azure Event Hubs](../event-hubs/index.yml). Intégrez ensuite vos données Event Hub à un SIEM tiers. En savoir plus dans [Diffuser des alertes vers un système SIEM, SOAR ou une solution de gestion des services informatiques](export-to-siem.md).
- [API REST](/rest/api/securitycenter/) : si vous utilisez l’API REST pour accéder aux alertes, consultez la [documentation en ligne sur l’API Alertes](/rest/api/securitycenter/alerts).

Si vous utilisez des méthodes de programmation pour consommer les alertes, vous aurez besoin du schéma approprié pour rechercher les champs qui vous intéressent. En outre, si vous exportez vers un hub d’événements ou tentez de déclencher l’automatisation du workflow avec des connecteurs HTTP génériques, utilisez les schémas pour analyser correctement les objets JSON.

>[!IMPORTANT]
> Le schéma est légèrement différent pour chacun de ces scénarios. Veillez donc à sélectionner l’onglet approprié ci-dessous.


## <a name="the-schemas"></a>Les schémas 


### <a name="workflow-automation-and-continuous-export-to-event-hub"></a>[Automatisation du workflow et exportation continue vers Event Hub](#tab/schema-continuousexport)

### <a name="sample-json-for-alerts-sent-to-logic-apps-event-hub-and-third-party-siems"></a>Exemple d’objet JSON pour les alertes envoyées à Logic Apps, Event Hub et les SIEM tiers

Vous trouverez ci-dessous le schéma des événements d’alerte transmis à :

- Des instances d’application logique Azure configurées dans l’automatisation de workflow de Security Center
- Azure Event Hub utilisant la fonctionnalité d’exportation continue de Security Center

Pour plus d’informations sur la fonctionnalité d’automatisation de workflow, consultez [Automatiser les réponses aux déclencheurs de Security Center](workflow-automation.md).

Pour plus d’informations sur l’exportation continue, consultez [Exporter en continu les données Security Center](continuous-export.md).

[!INCLUDE [Workflow schema](../../includes/security-center-alerts-schema-workflow-automation.md)]




### <a name="azure-sentinel-and-log-analytics-workspaces"></a>[Azure Sentinel et espaces de travail Log Analytics](#tab/schema-sentinel)

Le connecteur Sentinel reçoit les alertes d’Azure Security Center et les envoie à l’espace de travail Log Analytics pour Azure Sentinel. 

Pour créer un cas Sentinel ou un incident à l’aide d’alertes Security Center, vous aurez besoin du schéma pour les alertes indiquées ci-dessous. 

Pour plus d’informations sur Azure Sentinel, consultez [la documentation](../sentinel/index.yml).

[!INCLUDE [Sentinel and workspace schema](../../includes/security-center-alerts-schema-log-analytics-workspace.md)]




### <a name="azure-activity-log"></a>[Journal d'activité Azure](#tab/schema-activitylog)

Azure Security Center audite les alertes de sécurité générées en tant qu’événements dans le journal d'activité Azure.

Vous pouvez afficher les événements d’alertes de sécurité dans le journal d’activité en recherchant l’événement Activer l’alerte, comme indiqué ci-dessous :

[![Recherche de l’événement Activer l’alerte dans le journal d’activité](media/alerts-schemas/sample-activity-log-alert.png)](media/alerts-schemas/sample-activity-log-alert.png#lightbox)


### <a name="sample-json-for-alerts-sent-to-azure-activity-log"></a>Exemple d’objet JSON pour les alertes envoyées au journal d’activité Azure

```json
{
    "channels": "Operation",
    "correlationId": "2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff",
    "description": "PREVIEW - Role binding to the cluster-admin role detected. Kubernetes audit log analysis detected a new binding to the cluster-admin role which gives administrator privileges.\r\nUnnecessary administrator privileges might cause privilege escalation in the cluster.",
    "eventDataId": "2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff",
    "eventName": {
        "value": "PREVIEW - Role binding to the cluster-admin role detected",
        "localizedValue": "PREVIEW - Role binding to the cluster-admin role detected"
    },
    "category": {
        "value": "Security",
        "localizedValue": "Security"
    },
    "eventTimestamp": "2019-12-25T18:52:36.801035Z",
    "id": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RESOURCE_GROUP_NAME/providers/Microsoft.Security/locations/centralus/alerts/2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff/events/2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff/ticks/637128967568010350",
    "level": "Informational",
    "operationId": "2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff",
    "operationName": {
        "value": "Microsoft.Security/locations/alerts/activate/action",
        "localizedValue": "Activate Alert"
    },
    "resourceGroupName": "RESOURCE_GROUP_NAME",
    "resourceProviderName": {
        "value": "Microsoft.Security",
        "localizedValue": "Microsoft.Security"
    },
    "resourceType": {
        "value": "Microsoft.Security/locations/alerts",
        "localizedValue": "Microsoft.Security/locations/alerts"
    },
    "resourceId": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RESOURCE_GROUP_NAME/providers/Microsoft.Security/locations/centralus/alerts/2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff",
    "status": {
        "value": "Active",
        "localizedValue": "Active"
    },
    "subStatus": {
        "value": "",
        "localizedValue": ""
    },
    "submissionTimestamp": "2019-12-25T19:14:03.5507487Z",
    "subscriptionId": "SUBSCRIPTION_ID",
    "properties": {
        "clusterRoleBindingName": "cluster-admin-binding",
        "subjectName": "for-binding-test",
        "subjectKind": "ServiceAccount",
        "username": "masterclient",
        "actionTaken": "Detected",
        "resourceType": "Kubernetes Service",
        "severity": "Low",
        "intent": "[\"Persistence\"]",
        "compromisedEntity": "ASC-IGNITE-DEMO",
        "remediationSteps": "[\"Review the user in the alert details. If cluster-admin is unnecessary for this user, consider granting lower privileges to the user.\"]",
        "attackedResourceType": "Kubernetes Service"
    },
    "relatedEvents": []
}
```

### <a name="the-data-model-of-the-schema"></a>Modèle de données du schéma

|Champ|Description|
|----|----|
|**canaux**|Constante, « Operation »|
|**correlationId**|L’ID d’alerte Azure Security Center|
|**description**|Description de l’alerte|
|**eventDataId**|Voir correlationId|
|**eventName**|Les sous-champs value et localizedValue contiennent le nom complet de l’alerte|
|**category**|Les sous-champs value et localizedValue sont constants - « Security »|
|**eventTimestamp**|Horodateur UTC de l’heure à laquelle l'alerte a été générée|
|**id**|ID complet de l’alerte|
|**level**|Constante, « Informational »|
|**operationId**|Voir correlationId|
|**operationName**|Le champ de valeur est constant : « Microsoft.Security/locations/alerts/activate/action »,et la valeur localisée sera « Activer l’alerte » (peut être localisée par les paramètres régionaux de l’utilisateur)|
|**resourceGroupName**|Inclura le nom du groupe de ressources|
|**resourceProviderName**|Les sous-champs value et localizedValue sont constants - « Microsoft.Security »|
|**resourceType**|Les sous-champs value et localizedValue sont constants - « Microsoft.Security/locations/alerts »|
|**resourceId**|L’ID complet de la ressource Azure|
|**statut**|Les sous-champs value et localizedValue sont constants - « Active »|
|**subStatus**|Les sous-champs value et localizedValue sont vides|
|**submissionTimestamp**|Horodateur UTC de l’envoi d’un événement au Journal d’activité|
|**subscriptionId**|ID d’abonnement de la ressource compromise|
|**properties**|Conteneur JSON de propriétés supplémentaires relatives à l’alerte. Celles-ci peuvent passer d’une alerte à l’autre. Toutefois, les champs suivants s’affichent dans toutes les alertes :<br>- severity : Gravité de l’attaque<br>- compromisedEntity : Nom de la ressource compromise<br>- remediationSteps : Tableau des étapes de correction à effectuer<br>- intent : Intention de la chaîne de destruction de l’alerte. Les intentions possibles sont documentées dans le [tableau des intentions](alerts-reference.md#intentions)|
|**relatedEvents**|Constante - tableau vide|
|||





### <a name="ms-graph-api"></a>[API MS Graph](#tab/schema-graphapi)

Microsoft Graph est la passerelle vers les données et les informations dans Microsoft 365. Il fournit un modèle de programmabilité unifié que vous pouvez utiliser pour accéder à la gigantesque quantité de données disponibles dans Microsoft 365, Windows 10 et Enterprise Mobility + Security. Utilisez la richesse des données de Microsoft Graph afin de créer des applications pour les organisations et les consommateurs qui interagissent avec des millions d’utilisateurs.

Le schéma et une représentation JSON pour les alertes de sécurité envoyées à MS Graph sont disponibles dans [la documentation de Microsoft Graph](/graph/api/resources/alert?preserve-view=true&view=graph-rest-1.0).

---


## <a name="next-steps"></a>Étapes suivantes

Cet article décrit les schémas utilisés par les outils de protection contre les menaces Azure Security Center pour envoyer des informations sur les alertes de sécurité.

Pour plus d’informations sur les méthodes d’accès aux alertes de sécurité en dehors de Security Center, consultez les pages suivantes :

- [Azure Sentinel](../sentinel/index.yml) : SIEM Microsoft natif dans le cloud
- [Azure Event Hubs](../event-hubs/index.yml) : service Microsoft d’ingestion de données en temps réel complètement managé
- [Exporter en continu les données Security Center](continuous-export.md)
- [Espaces de travail Log Analytics](../azure-monitor/learn/quick-create-workspace.md) - Azure Monitor stocke les données de journal dans un espace de travail Log Analytics, c’est-à-dire un conteneur de données et d’informations de configuration