---
title: Benchmark de sécurité Azure v2 – Journalisation et détection des menaces
description: 'Benchmark de sécurité Azure v2 : Journalisation et détection des menaces'
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 09/20/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 99e7a8fcb34c963cfab366f1913508a7ff3f4a51
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94408700"
---
# <a name="security-control-v2-logging-and-threat-detection"></a>Contrôle de sécurité V2 : Journalisation et détection des menaces

La fonctionnalité Journalisation et détection des menaces recouvre les contrôles destinés à détecter les menaces dans Azure et à activer, collecter et stocker les journaux d’audit pour les services Azure. Cela comprend l’activation des processus de détection, d’investigation et de correction avec des contrôles visant à générer des alertes de haute qualité avec la détection native des menaces dans les services Azure. Cela inclut également la collecte des journaux avec Azure Monitor, la centralisation de l’analyse de la sécurité avec Azure Sentinel, la synchronisation de l’heure et la rétention des journaux. 

## <a name="lt-1-enable-threat-detection-for-azure-resources"></a>LT-1 : Activer la détection des menaces pour les ressources Azure

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| LT-1 | 6.7 | AU-3, AU-6, AU-12, SI-4 |

Veillez à surveiller les différents types de ressources Azure pour identifier les menaces et anomalies potentielles. Concentrez-vous sur l’obtention d’alertes de haute qualité afin de réduire les faux positifs que doivent trier les analystes. Les alertes peuvent provenir de données de journal, d’agents ou d’autres données.

Utilisez la capacité de détection des menaces intégrée à Azure Security Center, qui repose sur la surveillance de la télémétrie du service Azure et sur l’analyse des journaux de service. Les données sont collectées à l’aide de l’agent Log Analytics, qui lit divers journaux d’événements et configurations liées à la sécurité du système et copie les données dans votre espace de travail à des fins d’analyse. 

En outre, utilisez Azure Sentinel pour créer des règles d’analyse, qui repèrent des menaces correspondant à des critères spécifiques dans votre environnement. Les règles génèrent des incidents lorsque les critères sont respectés, de sorte que vous pouvez enquêter sur chaque incident. Azure Sentinel peut également importer le renseignement sur les menaces provenant de tiers pour améliorer la capacité de détection des menaces. 

- [Protection contre les menaces dans Azure Security Center](../../security-center/azure-defender.md)

- [Guide de référence des alertes de sécurité Azure Security Center](../../security-center/alerts-reference.md)

- [Créer des règles d’analytique personnalisées pour détecter des menaces](../../sentinel/tutorial-detect-threats-custom.md)

- [Renseignement sur les menaces informatiques dans Azure Sentinel](/azure/architecture/example-scenario/data/sentinel-threat-intelligence)

**Responsabilité**  : Customer

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Opérations de sécurité](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)   

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Renseignement sur les menaces](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

## <a name="lt-2-enable-threat-detection-for-azure-identity-and-access-management"></a>LT-2 : Activer la détection des menaces pour la gestion des identités et des accès Azure

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| LT-2 | 6.8 | AU-3, AU-6, AU-12, SI-4 |

Azure AD fournit les journaux d’utilisateur suivants qui peuvent être consultés dans les rapports Azure AD ou intégrés à Azure Monitor, Azure Sentinel ou d’autres outils de surveillance/SIEM pour des cas d’usage plus sophistiqués de surveillance et d’analytique : 
-   Connexions – le rapport de connexions fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.

-   Journaux d’audit : traçabilité proposée via des journaux d’activité pour toutes les modifications effectuées par diverses fonctionnalités au sein d’Azure AD. Par exemple, les journaux d’audit peuvent inclure les modifications apportées à des ressources dans Azure AD comme l’ajout ou la suppression d’utilisateurs, d’applications, de groupes, de rôles, de stratégies, etc.

-   Connexions risquées : une connexion risquée est une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.

-   Utilisateurs avec indicateur de risque : un utilisateur à risque correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Azure Security Center pouvez également alerter en cas de certaines activités suspectes, comme un nombre excessif de tentatives d’authentification ayant échoué et la présence de comptes dépréciés dans l’abonnement. En plus de la surveillance de base de l’hygiène de sécurité, le module Protection contre les menaces d’Azure Security Center peut également collecter des alertes de sécurité plus approfondies à partir de ressources de calcul Azure individuelles (telles que des machines virtuelles, des conteneurs ou un service d’application), de ressources de données (telles que la base de données SQL et le stockage) et de couches de service Azure. Cette capacité vous permet de voir les anomalies de compte à l’intérieur des ressources individuelles.

- [Rapports d’activité d’audit dans Azure AD](../../active-directory/reports-monitoring/concept-audit-logs.md)

- [Activer Azure AD Identity Protection](../../active-directory/identity-protection/overview-identity-protection.md)

- [Protection contre les menaces dans Azure Security Center](../../security-center/azure-defender.md)

**Responsabilité**  : Customer

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Opérations de sécurité](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)   

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Renseignement sur les menaces](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

## <a name="lt-3-enable-logging-for-azure-network-activities"></a>LT-3 : Activer la journalisation pour les activités réseau Azure

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| LT-3 | 9.3, 12.2, 12.5, 12.8 | AU-3, AU-6, AU-12, SI-4 |

Activez et collectez les journaux de ressources des groupes de sécurité réseau (NSG), les journaux de flux du NSG, les journaux de Pare-feu Azure et ceux de Web Application Firewall (WAF) afin d’analyser la sécurité et de prendre en charge les enquêtes sur les incidents, la chasse aux menaces et la génération d’alertes de sécurité. Vous pouvez envoyer les journaux de flux à un espace de travail Log Analytics sur Azure Monitor, puis utiliser Traffic Analytics pour obtenir des insights. Veillez à collecter les journaux de requêtes DNS pour faciliter la corrélation d’autres données réseau.

- [Tutoriel : journaliser le trafic réseau à destination et en provenance d’une machine virtuelle à l’aide du portail Azure](../../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Journaux et métriques du pare-feu Azure](../../firewall/logs-and-metrics.md)

- [Traffic Analytics](../../network-watcher/traffic-analytics.md)

- [Surveillance avec Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md)

- [Solutions de supervision réseau Azure dans Azure Monitor](../../azure-monitor/insights/azure-networking-analytics.md)

- [Rassemblement d’informations sur votre infrastructure DNS avec la solution DNS Analytics](../../azure-monitor/insights/dns-analytics.md)

**Responsabilité**  : Customer

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Opérations de sécurité](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)   

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Renseignement sur les menaces](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

## <a name="lt-4-enable-logging-for-azure-resources"></a>LT-4 : Activer la journalisation pour les ressources Azure

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| LT-4 | 6.2, 6.3, 8.8 | AU-3, AU-12 |

Activez la journalisation des ressources Azure pour répondre aux exigences de conformité, de détection des menaces, de chasse aux menaces et d’investigation des incidents. 

Vous pouvez utiliser Azure Security Center et Azure Policy pour activer les journaux de ressources et la collecte des données de journal sur les ressources Azure pour accéder aux journaux d’audit, de sécurité et de ressources. Les journaux d’activité, automatiquement disponibles, incluent la source de l’événement, la date, l’utilisateur, l’horodatage, les adresses sources, les adresses de destination et d’autres éléments utiles. 

- [Présentation de la journalisation et des différents types de journaux dans Azure](../../azure-monitor/platform/platform-logs-overview.md)

- [Présentation de la collecte de données Azure Security Center](../../security-center/security-center-enable-data-collection.md)

**Responsabilité**  : Partagé

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Opérations de sécurité](/azure/cloud-adoption-framework/organize/cloud-security-operations-center)

Sécurité d’infrastructure et de point de terminaison 

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Renseignement sur les menaces](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

## <a name="lt-5-centralize-security-log-management-and-analysis"></a>LT-5 : Centraliser la gestion et l’analyse des journaux de sécurité

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| LT-5 | 6.5, 6.6 | AU-3, SI-4 |

Centralisez le stockage et l’analyse de la journalisation pour activer la corrélation. Pour chaque source de journal, assurez-vous d’avoir attribué un propriétaire de données, des conseils d’accès, un emplacement de stockage, les outils utilisés pour traiter les données et y accéder, ainsi que les exigences de conservation des données.

Veillez à intégrer les journaux d’activité Azure dans votre journalisation centralisée. Ingérez des journaux par le biais d’Azure Monitor pour agréger les données de sécurité générées par les appareils de point de terminaison, les ressources réseau et d’autres systèmes de sécurité. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, et utilisez des comptes Stockage Azure pour le stockage à long terme et l’archivage.

En outre, activez et intégrez les données dans Azure Sentinel ou une solution SIEM tierce.

De nombreuses organisations choisissent d’utiliser Azure Sentinel pour les données « chaudes » qui sont utilisées fréquemment et Stockage Azure pour les données « froides » qui sont utilisées moins fréquemment. 

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../../azure-monitor/platform/diagnostic-settings.md)

- [Guide pratique pour intégrer Azure Sentinel](../../sentinel/quickstart-onboard.md)

**Responsabilité**  : Customer

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

## <a name="lt-6-configure-log-storage-retention"></a>LT-6 : Configurer la rétention du stockage des journaux

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| LT-6 | 6.4 | AU-3, AU-11 |

Configurez la rétention des journaux en fonction de vos besoins en matière de conformité, de réglementation et d’exigences métier. 

Dans Azure Monitor, vous pouvez définir la période de rétention de votre espace de travail Log Analytics en fonction des règles de conformité de votre organisation. Utilisez des comptes Stockage Azure, Data Lake ou d’espace de travail Log Analytics pour le stockage à long terme et l’archivage.

- [Modification de la période de conservation des données dans Log Analytics](../../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

- [Guide pratique pour configurer la stratégie de conservation des journaux de compte de Stockage Azure](../../storage/common/storage-monitor-storage-account.md#configure-logging)

- [Exportation des alertes et recommandations Azure Security Center](../../security-center/continuous-export.md)

**Responsabilité**  : Customer

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Opérations de sécurité](/azure/cloud-adoption-framework/organize/cloud-security-operations-center) 

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="lt-7-use-approved-time-synchronization-sources"></a>LT-7 : Utiliser des sources de synchronisation date/heure approuvées

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| LT-7 | 6.1 | AU-8 |

Microsoft gère les sources temporelles pour la plupart des services PaaS et SaaS d’Azure. Pour vos machines virtuelles, utilisez le serveur NTP par défaut de Microsoft pour la synchronisation de l’heure, sauf si vous avez une exigence spécifique.  Si vous devez mettre en place votre propre serveur NTP (Network Time Protocol), veillez à sécuriser le port 123 du service UDP.

Tous les journaux générés par des ressources dans Azure fournissent des horodatages avec le fuseau horaire spécifié par défaut.

- [Guide pratique pour configurer la synchronisation date/heure pour les ressources de calcul Windows Azure](../../virtual-machines/windows/time-sync.md)

- [Guide pratique pour configurer la synchronisation date/heure pour les ressources de calcul Linux Azure](../../virtual-machines/linux/time-sync.md)

- [Comment désactiver le protocole UDP entrant pour les services Azure](https://support.microsoft.com/help/4558520/how-to-disable-inbound-udp-for-azure-services)

**Responsabilité**  : Partagé

**Parties prenantes de la sécurité des clients** ( [En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Stratégie et normes](/azure/cloud-adoption-framework/organize/cloud-security-policy-standards)

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)