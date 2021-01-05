---
title: Base de référence de sécurité Azure pour NAT de réseau virtuel
description: La base de référence de sécurité du service NAT de réseau virtuel fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 11/20/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 4ab71138f110df19ec84fa8707b480ad1bc6e72c
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95255064"
---
# <a name="azure-security-baseline-for-virtual-network-nat"></a>Base de référence de sécurité Azure pour NAT de réseau virtuel

Cette base de référence de sécurité applique les conseils du [Benchmark de sécurité Azure version 1.0](https://docs.microsoft.com/azure/security/benchmarks/overview-v1) au service NAT de réseau virtuel de Microsoft. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure.
Le contenu est regroupé selon les **contrôles de sécurité** définis par le benchmark de sécurité Azure et les conseils associés, applicables à NAT de réseau virtuel. Les **Contrôles** non applicables à NAT de réseau virtuel ont été exclus.

 
Pour voir comment le service NAT de réseau virtuel est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité NAT de réseau virtuel](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : sécurité réseau](https://docs.microsoft.com/azure/security/benchmarks/security-control-network-security).*

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Conseils** : NAT de réseau virtuel ne prend pas en charge le contrôle du trafic sortant avec les groupes de sécurité réseau (NSG).  Le trafic entrant n’est autorisé qu’en réponse à un flux sortant.

Toutefois, les journaux de flux de groupe de sécurité réseau (NSG) peuvent être utilisés avec les ressources de passerelle NAT pour superviser le trafic sortant.

Utilisez Azure Security Center et suivez les recommandations de protection du réseau pour sécuriser vos ressources réseau Azure. Activez les journaux de flux de groupe de sécurité réseau et envoyez ceux-ci à un compte Stockage Azure à des fins d’audit. Vous pouvez aussi envoyer les journaux de flux à un espace de travail Log Analytics, puis utiliser Traffic Analytics pour obtenir des insights sur les modèles de trafic au sein de votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau, d’identifier les zones réactives et des menaces de sécurité, de comprendre les modèles de flux de trafic, ainsi que d’épingler de mauvaises configurations du réseau. 

- [Vue d’ensemble de NAT de réseau virtuel](https://docs.microsoft.com/azure/virtual-network/nat-overview)

- [Ressources de passerelle NAT](https://docs.microsoft.com/azure/virtual-network/nat-gateway-resource)

- [Tutoriel : journaliser le trafic réseau à destination et en provenance d’une machine virtuelle à l’aide du portail Azure](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal) 

- [Traffic Analytics](https://docs.microsoft.com/azure/network-watcher/traffic-analytics)

- [Protéger vos ressources réseau](https://docs.microsoft.com/azure/security-center/security-center-network-recommendations)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Conseils** : Activez la capture de paquets Network Watcher pour analyser les activités anormales. 

- [Créer une instance d’Azure Network Watcher](https://docs.microsoft.com/azure/network-watcher/network-watcher-create)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour les sous-réseaux configurés avec des ressources de passerelle NAT, au moyen d’affectations et de définitions Azure Policy personnalisées et intégrées.

- [Guide pratique pour configurer et gérer Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

- [Exemples Azure Policy pour le réseau](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#network)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : Utilisez le journal d’activité Azure pour superviser les configurations de ressources, et détecter les modifications apportées à vos ressources de passerelle NAT et de réseau virtuel. Créez des alertes dans Azure Monitor pour vous avertir lorsque des ressources critiques sont modifiées.

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view)

- [Guide pratique pour créer des alertes dans Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : journalisation et supervision](https://docs.microsoft.com/azure/security/benchmarks/security-control-logging-monitoring).*

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : Ingérez les journaux liés à NAT de réseau virtuel par le biais d’Azure Monitor pour agréger les données de sécurité générées par les appareils de point de terminaison, les ressources réseau et d’autres systèmes de sécurité. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, et utilisez des comptes Stockage Azure pour le stockage à long terme et l’archivage.

Vous pouvez également activer et intégrer ces données dans Azure Sentinel ou une solution SIEM tierce.

- [Guide pratique pour intégrer Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard) 

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings) 

- [Guide pratique pour bien démarrer avec Azure Monitor et l’intégration d’une solution SIEM tierce](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Les journaux d’activité, disponibles automatiquement, contiennent toutes les opérations d’écriture (PUT, POST, DELETE) pour vos ressources de passerelle NAT, à l’exception des opérations de lecture (GET). Les journaux d’activité peuvent être utilisés pour rechercher une erreur lors de la résolution de problèmes ou pour surveiller la manière dont un utilisateur de votre organisation a modifié une ressource.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings) 

- [Présentation de la journalisation et des différents types de journaux dans Azure](https://docs.microsoft.com/azure/azure-monitor/platform/platform-logs-overview) 

Actuellement, NAT de réseau virtuel ne produit pas de journaux de diagnostic supplémentaires configurables par les clients.

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseil** : utilisez Azure Security Center avec un espace de travail Log Analytics pour superviser les activités anormales détectées dans les journaux de sécurité et les événements et générer des alertes s’y rapportant. Vous pouvez, autrement, activer Azure Sentinel et y intégrer les données.

- [Guide pratique pour intégrer Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard) 

- [Guide pratique pour gérer les alertes dans Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts) 

- [Guide pratique pour générer une alerte sur des données de journal Log Analytics](https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-response)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](https://docs.microsoft.com/azure/security/benchmarks/security-control-inventory-asset-management).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (telles que calcul, stockage, réseau, ports, protocoles, etc.) dans vos abonnements. 

Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez l’ensemble des abonnements et ressources Azure dans vos abonnements.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources s’appuyant sur Azure Resource Manager à l’avenir.

- [Requêtes Azure Resource Graph](https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des étiquettes à vos ressources Azure, groupes de ressources et abonnements pour les organiser de façon logique dans une taxonomie. Chaque balise se compose d’une paire nom-valeur. Par exemple, vous pouvez appliquer le nom « Environnement » et la valeur « Production » à toutes les ressources en production.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Appliquez des étiquettes pour la gestion des ressources Azure.

- [Guide pratique pour créer des abonnements Azure supplémentaires](https://docs.microsoft.com/azure/billing/billing-create-subscription) 

- [Guide pratique pour créer des groupes d’administration](https://docs.microsoft.com/azure/governance/management-groups/create) 

- [Guide pratique pour créer et utiliser des étiquettes](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4 : Dresser et tenir un inventaire des ressources Azure approuvées

**Conseils** : actuellement non disponible

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Aide** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements.
Utilisez Azure Resource Graph pour interroger/découvrir des ressources dans leurs abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

- [Guide pratique pour configurer et gérer Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage) 
- [Guide pratique pour créer des requêtes avec Azure Graph](https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Aide** : Utilisez Azure Policy pour limiter les services que vous pouvez approvisionner dans votre environnement.

- [Guide pratique pour configurer et gérer Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#general)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](https://docs.microsoft.com/azure/security/benchmarks/security-control-secure-configuration).*

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : Non applicable ; NAT de réseau virtuel ne dispose pas de configuration de sécurité.

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Utilisez Azure DevOps pour stocker et gérer de manière sécurisée votre code, comme les définitions de stratégies Azure personnalisées, les modèles Azure Resource Manager et les scripts Desired State Configuration. Pour accéder aux ressources que vous gérez dans Azure DevOps, vous pouvez accorder ou refuser des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps, ou à Active Directory s’il est intégré à TFS. 

- [Stocker du code dans Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops) 

- [À propos des autorisations et des groupes dans Azure DevOps](https://docs.microsoft.com/azure/devops/organizations/security/about-permissions)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : Non applicable ; NAT de réseau virtuel ne dispose pas de configuration de sécurité.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](https://docs.microsoft.com/azure/security/benchmarks/security-control-incident-response).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Une procédure de réponse aux incidents doit être créée pour que l’offre permette aux processus de réponse aux incidents appropriés de recevoir le niveau de priorité adéquat jusqu’à la résolution des incidents.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Une procédure de scoring et de hiérarchisation des incidents doit être créée pour que l’offre permette au scoring des menaces ou des risques approprié de recevoir le niveau de priorité adéquat jusqu’à la résolution des incidents.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Une procédure de réponse aux incidents de sécurité doit être créée et testée pour que l’offre assure une atténuation des risques appropriée, avec le niveau de priorité adéquat jusqu’à la résolution prévue des incidents.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Conseils** : Une procédure de réponse aux incidents de sécurité doit être créée et testée pour que l’offre assure une atténuation des risques appropriée, avec le niveau de priorité adéquat jusqu’à la résolution prévue des incidents.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Une procédure de réponse aux incidents de sécurité doit être créée et testée pour que l’offre assure une atténuation des risques appropriée, avec le niveau de priorité adéquat jusqu’à la résolution prévue des incidents.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Une procédure de réponse aux incidents de sécurité doit être créée et testée pour que l’offre assure une atténuation des risques appropriée, avec le niveau de priorité adéquat jusqu’à la résolution prévue des incidents.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : tests d’intrusion et exercices Red Team](https://docs.microsoft.com/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de pénétration du cloud Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft. 

- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Vue d’ensemble d’Azure Security Benchmark V2](/azure/security/benchmarks/overview)
- En savoir plus sur les [bases de référence de la sécurité Azure](/azure/security/benchmarks/security-baselines-overview)
