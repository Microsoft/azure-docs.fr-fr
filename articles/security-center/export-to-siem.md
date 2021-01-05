---
title: Diffusez vos alertes d’Azure Security Center à des systèmes Security Information and Event Management (SIEM) et d’autres solutions de surveillance
description: Découvrez comment diffuser en continu vos alertes de sécurité dans Azure Sentinel, des systèmes SIEM tiers, SOAR ou des solutions ITSM
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 10/06/2020
ms.author: memildin
ms.openlocfilehash: 6ffb6ced6fc828733dd627943a3d4b54e8293ad2
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92791900"
---
# <a name="stream-alerts-to-a-siem-soar-or-it-service-management-solution"></a>Diffuser des alertes vers un système SIEM, SOAR ou une solution de gestion des services informatiques

Azure Security Center peut diffuser vos alertes de sécurité vers des systèmes SIEM tiers, SOAR ou des solutions de gestion des services informatiques populaires.

Il existe des outils Azure natifs pour vous permettre d’afficher les données d’alerte dans toutes les solutions les plus populaires actuellement utilisées, y compris :

- **Azure Sentinel**
- **Splunk Enterprise and Splunk Cloud**
- **QRadar d’IBM**
- **ServiceNow**
- **ArcSight**
- **Power BI**
- **Palo Alto Networks**

## <a name="stream-alerts-to-azure-sentinel"></a>Diffuser en continu des alertes vers Azure Sentinel 

Security Center s’intègre en mode natif avec Azure Sentinel, la solution SIEM et SOAR cloud native d’Azure. 

[En savoir plus sur Azure Sentinel](../sentinel/overview.md).

### <a name="azure-sentinels-connectors-for-security-center"></a>Connecteurs Azure Sentinel pour Security Center

Azure Sentinel comprend des connecteurs intégrés pour Azure Security Center au niveau de l’abonnement et du locataire :

- [Diffuser en continu des alertes vers Azure Sentinel au niveau de l’abonnement](../sentinel/connect-azure-security-center.md)
- [Connecter tous les abonnements de votre locataire à Azure Sentinel](https://techcommunity.microsoft.com/t5/azure-sentinel/azure-security-center-auto-connect-to-sentinel/ba-p/1387539) 

### <a name="configure-ingestion-of-all-audit-logs-into-azure-sentinel"></a>Configurer l’ingestion de tous les journaux d’audit dans Azure Sentinel 

Une autre solution pour examiner les alertes Security Center dans Azure Sentinel consiste à diffuser vos journaux d’audit dans Azure Sentinel :
    - [Connecter les événements de sécurité Windows](../sentinel/connect-windows-security-events.md)
    - [Collecter des données de sources Linux à l’aide de Syslog](../sentinel/connect-syslog.md)
    - [Connecter des données à partir du journal d’activité Azure](../sentinel/connect-azure-activity.md)

> [!TIP]
> Azure Sentinel est facturé en fonction du volume de données ingérées pour l’analyse dans Azure Sentinel et stocké dans l’espace de travail Azure Monitor Log Analytics. Azure Sentinel offre un modèle de tarification flexible et prévisible. [Pour plus d’informations, consultez la page de tarification d’Azure Sentinel](https://azure.microsoft.com/pricing/details/azure-sentinel/).


## <a name="stream-alerts-with-microsoft-graph-security-api"></a>Diffuser en continu des alertes avec l’API de sécurité Microsoft Graph

Security Center dispose d’une intégration prête à l’emploi avec l’API de sécurité Microsoft Graph. Aucune configuration n’est requise et aucun coût supplémentaire n’est nécessaire. 

Vous pouvez utiliser cette API pour diffuser en continu des alertes à partir de votre **locataire entier** (et des données de nombreux autres produits de sécurité Microsoft) dans des solutions SIEM et autres plateformes tierces populaires :

- **Splunk Enterprise et Splunk Cloud** - [Utiliser le module complémentaire d’API de sécurité Microsoft Graph pour Splunk](https://splunkbase.splunk.com/app/4564/) 
- **Power BI** - [Se connecter à l’API de sécurité Microsoft Graph dans Power BI Desktop](/power-bi/connect-data/desktop-connect-graph-security)
- **ServiceNow** - [Suivre les instructions d’installation et de configuration de l’application d’API de sécurité Microsoft Graph à partir du magasin ServiceNow](https://docs.servicenow.com/bundle/orlando-security-management/page/product/secops-integration-sir/secops-integration-ms-graph/task/ms-graph-install.html)
- **QRadar** - [Module de prise en charge des appareils IBM pour Azure Security Center via l’API Microsoft Graph](https://www.ibm.com/support/knowledgecenter/SS42VS_DSM/com.ibm.dsm.doc/c_dsm_guide_ms_azure_security_center_overview.html) 
- **Palo Alto Networks** , **Anomali** , **Lookout** , **InSpark** , et plus - [API de sécurité Microsoft Graph](https://www.microsoft.com/security/business/graph-security-api#office-MultiFeatureCarousel-09jr2ji)

[En savoir plus sur l’API de sécurité Microsoft Graph](https://www.microsoft.com/security/business/graph-security-api).


## <a name="stream-alerts-with-azure-monitor"></a>Diffuser en continu des alertes avec Azure Monitor 

Pour diffuser des alertes dans **ArcSight** , **Splunk** , **SumoLogic** , les serveurs syslog, **LogRhythm** , **la plateforme d’observabilité cloud Logz.io** et d’autres solutions de surveillance. Connectez Security Center à Azure Monitor via Azure Event Hubs :

1. Activez l’[exportation continue](continuous-export.md) pour diffuser des alertes Security Center vers une instance Azure Event Hub dédiée au niveau de l’abonnement. 
    > [!TIP]
    > Pour effectuer cette opération au niveau du groupe d’administration à l’aide d’Azure Policy, consultez [Créer des configurations d’automatisation d’exportation continue à l’échelle](continuous-export.md?tabs=azure-policy#configure-continuous-export-at-scale-using-the-supplied-policies)

1. [Connectez Azure Event Hub à votre solution préférée à l’aide des connecteurs intégrés d’Azure Monitor](../azure-monitor/platform/stream-monitoring-data-event-hubs.md#partner-tools-with-azure-monitor-integration).

1. Si vous le souhaitez, diffusez les journaux bruts dans Azure Event Hub et connectez-vous à votre solution préférée. En savoir plus sur les [données de surveillance disponibles](../azure-monitor/platform/stream-monitoring-data-event-hubs.md#monitoring-data-available).

> [!TIP]
> Pour afficher les schémas d’événements des types de données exportés, visitez les [schémas d’événements Event Hub](https://aka.ms/ASCAutomationSchemas).


## <a name="next-steps"></a>Étapes suivantes

Cette page explique comment garantir que vos données d’alerte Azure Security Center sont disponibles dans l’outil SIEM, SOAR ou ITSM de votre choix. Consultez les documents connexes suivants :

- [Qu’est-ce qu’Azure Sentinel ?](../sentinel/overview.md)
- [Validation d’alerte dans Azure Security Center](security-center-alert-validation.md) : Vérifiez que vos alertes sont correctement configurées
- [Exporter en continu des données Security Center](continuous-export.md)