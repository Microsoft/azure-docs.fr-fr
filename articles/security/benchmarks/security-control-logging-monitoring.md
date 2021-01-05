---
title: Contrôle de sécurité Azure - Journalisation et supervision
description: Contrôle de sécurité Azure – Journalisation et monitoring
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 8492e986b378ea19f8c3c4cb7f94fcb0f155b427
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96487845"
---
# <a name="security-control-logging-and-monitoring"></a>Contrôle de sécurité : Journalisation et supervision

La journalisation et la supervision de la sécurité se concentrent sur les activités liées à l’activation, l’acquisition et le stockage des journaux d’audit pour les services Azure.

## <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 2.1 | 6.1 | Microsoft |

Microsoft gère les sources de temps pour les ressources Azure. Toutefois, vous avez la possibilité de gérer les paramètres de synchronisation date/heure pour vos ressources de calcul.

- [Guide pratique pour configurer la synchronisation date/heure pour les ressources de calcul Windows Azure](../../virtual-machines/windows/time-sync.md)

- [Guide pratique pour configurer la synchronisation date/heure pour les ressources de calcul Linux Azure](../../virtual-machines/linux/time-sync.md)

## <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 2.2 | 6.5, 6.6 | Customer |

Ingérez des journaux par le biais d’Azure Monitor pour agréger les données de sécurité générées par les appareils de point de terminaison, les ressources réseau et d’autres systèmes de sécurité. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, puis utilisez les comptes de stockage Azure pour le stockage à long terme/d’archivage.

Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce. 

- [Guide pratique pour intégrer Azure Sentinel](../../sentinel/quickstart-onboard.md)

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../../azure-monitor/platform/diagnostic-settings.md)

- [Guide pratique pour collecter les journaux des hôtes internes des machines virtuelles Azure avec Azure Monitor](../../azure-monitor/learn/quick-collect-azurevm.md)

- [Guide pratique pour bien démarrer avec Azure Monitor et l’intégration d’une solution SIEM tierce](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

## <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 2.3 | 6.2, 6.3 | Customer |

Activez les paramètres de diagnostic sur les ressources Azure pour accéder aux journaux d’audit, de sécurité et de diagnostic. Les journaux d’activité, automatiquement disponibles, incluent la source de l’événement, la date, l’utilisateur, l’horodatage, les adresses sources, les adresses de destination et d’autres éléments utiles.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../../azure-monitor/platform/diagnostic-settings.md)

- [Présentation de la journalisation et des différents types de journaux dans Azure](../../azure-monitor/platform/platform-logs-overview.md)

## <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 2.4 | 6.2, 6.3 | Customer |

Si la ressource de calcul appartient à Microsoft, alors Microsoft est responsable de sa supervision. Si la ressource de calcul appartient à votre organisation, il vous incombe de la superviser. Vous pouvez utiliser Azure Security Center pour superviser le système d’exploitation. Les données collectées par Security Center auprès du système d’exploitation incluent le type et la version du système d’exploitation, le système d’exploitation (journaux des événements Windows), les processus en cours d’exécution, le nom de l’ordinateur, les adresses IP et l’utilisateur connecté. L’agent Log Analytics collecte également les fichiers de vidage sur incident.

- [Guide pratique pour collecter les journaux des hôtes internes des machines virtuelles Azure avec Azure Monitor](../../azure-monitor/learn/quick-collect-azurevm.md)

- [Présentation de la collecte de données Azure Security Center](../../security-center/security-center-enable-data-collection.md)

## <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 2.5 | 6.4 | Customer |

Dans Azure Monitor, définissez la période de conservation de votre espace de travail Log Analytics en fonction des obligations réglementaires de votre organisation. Utilisez les comptes de stockage Azure pour le stockage à long terme/d’archivage.

- [Modification de la période de conservation des données dans Log Analytics](../../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

- [Guide pratique pour configurer la stratégie de conservation des journaux de compte de Stockage Azure](../../storage/common/storage-monitor-storage-account.md#configure-logging)

## <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 2.6 | 6.7 | Customer |

Analysez et supervisez les journaux pour détecter les comportements anormaux et examinez régulièrement les résultats. Utilisez l’espace de travail Log Analytics d’Azure Monitor pour examiner les journaux et effectuer des requêtes sur leurs données.

Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce. 

- [Guide pratique pour intégrer Azure Sentinel](../../sentinel/quickstart-onboard.md)

- [Présentation de l’espace de travail Log Analytics](../../azure-monitor/log-query/log-analytics-tutorial.md)

- [Guide pratique pour effectuer des requêtes personnalisées dans Azure Monitor](../../azure-monitor/log-query/get-started-queries.md)

## <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 2.7 | 6.8 | Customer |

Utilisez Azure Security Center avec un espace de travail Log Analytics pour superviser les activités anormales détectées dans les journaux de sécurité et les événements et générer des alertes s’y rapportant.

Vous pouvez également activer et intégrer les données dans Azure Sentinel.

- [Guide pratique pour intégrer Azure Sentinel](../../sentinel/quickstart-onboard.md)

- [Guide pratique pour gérer les alertes dans Azure Security Center](../../security-center/security-center-managing-and-responding-alerts.md)

- [Guide pratique pour générer une alerte sur des données de journal Log Analytics](../../azure-monitor/learn/tutorial-response.md)

## <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 2.8 | 8.6 | Customer |

Activez la collecte des événements anti-programme malveillant pour les machines virtuelles et services cloud Azure.

- [Guide pratique pour configurer Microsoft Antimalware pour les machines virtuelles](/powershell/module/servicemanagement/azure.service/set-azurevmmicrosoftantimalwareextension?view=azuresmps-4.0.0)

- [Guide pratique pour configurer Microsoft Antimalware pour les services cloud](/powershell/module/servicemanagement/azure.service/set-azureserviceantimalwareextension?view=azuresmps-4.0.0)

- [Présentation de Microsoft Antimalware](../fundamentals/antimalware.md)

## <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 2,9 | 8,7 | Customer |

Implémentez une solution tierce de journalisation DNS à partir de la Place de marché Azure en fonction des besoins de votre organisation.  

## <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 2.10 | 8.8 | Customer |

Utilisez Microsoft Monitoring Agent sur toutes les machines virtuelles Windows Azure prises en charge pour journaliser l’événement de création de processus et le champ de ligne de commande.   En ce qui concerne les machines virtuelles Linux Azure prises en charge, vous pouvez configurer manuellement la journalisation de la console au niveau de chaque nœud et utiliser Syslog pour stocker les données.  Exploitez également l’espace de travail Log Analytics d’Azure Monitor pour examiner les journaux et effectuer des requêtes sur les données journalisées à partir des machines virtuelles Azure. 

- [Collecte de données dans Azure Security Center](../../security-center/security-center-enable-data-collection.md#data-collection-tier)

- [Guide pratique pour effectuer des requêtes personnalisées dans Azure Monitor](../../azure-monitor/log-query/get-started-queries.md)

- [Sources de données Syslog dans Azure Monitor](../../azure-monitor/platform/data-sources-syslog.md)


## <a name="next-steps"></a>Étapes suivantes

- Consultez le contrôle de sécurité suivant : [Identité et contrôle d’accès](security-control-identity-access-control.md)