---
title: Base de référence sur la sécurité Azure pour la Gestion des API
description: Base de référence sur la sécurité Azure pour la Gestion des API
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 05/04/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 62b4629d258f8779dc53b4b8e00684d3b793291d
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183751"
---
# <a name="azure-security-baseline-for-api-management"></a>Base de référence sur la sécurité Azure pour la Gestion des API

La base de référence sur la sécurité Azure pour la Gestion des API contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez la [Vue d’ensemble des bases de référence de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1 : Protéger les ressources à l'aide de groupes de sécurité réseau ou du Pare-feu Azure sur votre réseau virtuel

**Aide** : Vous pouvez déployer la Gestion des API Azure sur le réseau virtuel Azure afin de lui permettre d’accéder aux services back-end au sein même du réseau. Le portail des développeurs et la passerelle Gestion des API peuvent être configurés pour être accessibles à partir d’Internet (externe) ou uniquement au sein du réseau virtuel (interne).
- Externe : la passerelle Gestion des API et le portail des développeurs sont accessibles à partir de l’Internet public via un équilibreur de charge externe. La passerelle peut accéder aux ressources au sein du réseau virtuel.
- Interne : la passerelle Gestion des API et le portail des développeurs sont accessibles uniquement au sein du réseau virtuel via un équilibreur de charge interne. La passerelle peut accéder aux ressources au sein du réseau virtuel.

Le trafic entrant et sortant dans le sous-réseau où la Gestion des API est déployée peut être contrôlé à l’aide de groupes de sécurité réseau.

* [Utilisation de la gestion des API Azure avec des réseaux virtuels](./api-management-using-with-vnet.md)

* [Utilisation du service Gestion des API Azure avec un réseau virtuel interne](./api-management-using-with-internal-vnet.md)

* [Intégrer le service Gestion des API dans un réseau virtuel interne avec Application Gateway](./api-management-howto-integrate-internal-vnet-appgateway.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2 : Surveiller et consigner la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Conseils** : Le trafic entrant et sortant dans le sous-réseau où la Gestion des API est déployée peut être contrôlé à l’aide de groupes de sécurité réseau. Déployez un groupe de sécurité réseau sur votre sous-réseau de gestion des API, puis activez les journaux de flux NSG et envoyez-les vers un compte de stockage Azure pour l’audit du trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

Attention : Lorsque vous configurez un groupe de sécurité réseau sur le sous-réseau de gestion des API, certains ports doivent rester ouverts. Si ces ports ne sont pas disponibles, la gestion des API risque de ne pas fonctionner correctement et d’être inaccessible.

* [Configurations NSG pour la Gestion des API Azure](./api-management-using-with-vnet.md#-common-network-configuration-issues)

* [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Guide pratique pour activer et utiliser Traffic Analytics](../network-watcher/traffic-analytics.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Aide** : Pour protéger les API web/HTTP critiques, configurez la Gestion des API au sein d’un réseau virtuel en mode Interne, puis configurez une instance d’Azure Application Gateway. Application Gateway est un service PaaS. Il agit comme un proxy inverse et fournit un équilibrage de charge L7, un routage, un pare-feu d’applications web et d’autres services.

Le fait d’associer la Gestion des API provisionnée dans un réseau virtuel interne à un front-end Application Gateway permet les scénarios suivants :
- Utilisez une même ressource de gestion des API pour exposer toutes les API à l’ensemble des utilisateurs internes et externes.
- Utilisez une même ressource de gestion des API pour exposer un sous-réseau d’API aux utilisateurs externes.
- Fournissez un moyen d’activer et de désactiver l’accès à la Gestion des API à partir de l’Internet public.

Remarque : Cette fonctionnalité est disponible dans les niveaux Premium et Développeur de Gestion des API.

* [Intégrer le service Gestion des API dans un réseau virtuel interne avec Application Gateway](./api-management-howto-integrate-internal-vnet-appgateway.md)

* [Présentation d’Azure Application Gateway](../application-gateway/index.yml)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Aide** : Configurez la Gestion des API au sein d’un réseau virtuel en mode Interne, puis configurez une instance d’Azure Application Gateway. Application Gateway est un service PaaS. Il agit comme un proxy inverse et fournit un équilibrage de charge L7, un routage, un pare-feu d’applications web et d’autres services.

Le fait d’associer la Gestion des API provisionnée dans un réseau virtuel interne à un front-end Application Gateway permet les scénarios suivants :
- Utilisez une même ressource de gestion des API pour exposer toutes les API à l’ensemble des utilisateurs internes et externes.
- Utilisez une même ressource de gestion des API pour exposer un sous-réseau d’API aux utilisateurs externes.
- Fournissez un moyen d’activer et de désactiver l’accès à la Gestion des API à partir de l’Internet public.

Remarque : Cette fonctionnalité est disponible dans les niveaux Premium et Développeur de Gestion des API.

Activez Azure DDoS Protection Standard dans le réseau virtuel associé au déploiement de la Gestion des API afin de vous protéger contre les attaques par déni de service distribué (DDoS).

Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

* [Intégrer le service Gestion des API dans un réseau virtuel interne avec Application Gateway](./api-management-howto-integrate-internal-vnet-appgateway.md)

* [Présentation d’Azure Application Gateway](../application-gateway/index.yml)

* [Configurer Azure DDoS Protection Standard](../ddos-protection/manage-ddos-protection.md)

* [Présentation de la fonctionnalité Threat Intelligence intégrée à Azure Security Center](../security-center/azure-defender.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5 : Consigner les paquets réseau et les journaux de flux

**Aide** : Le trafic entrant et sortant dans le sous-réseau où la Gestion des API est déployée peut être contrôlé à l’aide de groupes de sécurité réseau. Déployez un groupe de sécurité réseau sur votre sous-réseau de gestion des API, puis activez les journaux de flux NSG et envoyez-les vers un compte de stockage Azure pour l’audit du trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

Attention : Lorsque vous configurez un groupe de sécurité réseau sur le sous-réseau de gestion des API, certains ports doivent rester ouverts. Si ces ports ne sont pas disponibles, la gestion des API risque de ne pas fonctionner correctement et d’être inaccessible.

* [Configurations NSG pour la Gestion des API Azure](./api-management-using-with-vnet.md#-common-network-configuration-issues)

* [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Guide pratique pour activer et utiliser Traffic Analytics](../network-watcher/traffic-analytics.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Conseils** : Configurez la Gestion des API au sein d’un réseau virtuel en mode Interne, puis configurez une instance d’Azure Application Gateway. Application Gateway est un service PaaS. Il agit comme un proxy inverse et fournit un équilibrage de charge L7, un routage, un pare-feu d’applications web et d’autres services. Le pare-feu d’applications web Application Gateway fournit une protection contre les vulnérabilités et les failles de sécurité courantes, et peut s’exécuter en deux modes :
- Mode de détection : Surveille et journalise toutes les alertes de menace. Vous pouvez activer la journalisation des diagnostics pour Application Gateway dans la section Diagnostics. Vous devez également vérifier que le journal du pare-feu d’applications web est sélectionné et activé. Le pare-feu d’applications web ne bloque pas les demandes entrantes quand il opère en mode de détection.
- Mode de prévention : Bloque les intrusions et les attaques détectées par les règles. L’attaquant reçoit une exception « 403 Accès non autorisé » et la connexion est fermée. Le mode de prévention enregistre de telles attaques dans les journaux WAF.

Le fait d’associer la Gestion des API provisionnée dans un réseau virtuel interne à un front-end Application Gateway permet les scénarios suivants :
- Utilisez une même ressource de gestion des API pour exposer toutes les API à l’ensemble des utilisateurs internes et externes.
- Utilisez une même ressource de gestion des API pour exposer un sous-réseau d’API aux utilisateurs externes.
- Fournissez un moyen d’activer et de désactiver l’accès à la Gestion des API à partir de l’Internet public.

Remarque : Cette fonctionnalité est disponible dans les niveaux Premium et Développeur de Gestion des API.

* [Intégrer le service Gestion des API dans un réseau virtuel interne avec Application Gateway](./api-management-howto-integrate-internal-vnet-appgateway.md)

* [Présentation du pare-feu d’applications web Azure Application Gateway](../web-application-firewall/ag/ag-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Aide** : Pour gérer le trafic qui arrive aux API web/HTTP, déployez la Gestion des API sur un réseau virtuel avec App Service Environment, en mode externe ou interne.

En mode interne, configurez une instance d’Azure Application Gateway devant la Gestion des API. Application Gateway est un service PaaS. Il agit comme un proxy inverse et fournit un équilibrage de charge L7, un routage, un pare-feu d’applications web et d’autres services. Le pare-feu d’applications web Application Gateway fournit une protection contre les vulnérabilités et les failles de sécurité courantes.

Le fait d’associer la Gestion des API provisionnée dans un réseau virtuel interne à un front-end Application Gateway permet les scénarios suivants :
- Utilisez une même ressource de gestion des API pour exposer toutes les API à l’ensemble des utilisateurs internes et externes.
- Utilisez une même ressource de gestion des API pour exposer un sous-réseau d’API aux utilisateurs externes.
- Fournissez un moyen d’activer et de désactiver l’accès à la Gestion des API à partir de l’Internet public.

Remarque : Cette fonctionnalité est disponible dans les niveaux Premium et Développeur de Gestion des API.

* [Exposer des API privées aux utilisateurs externes](/azure/architecture/example-scenario/apps/publish-internal-apis-externally)

* [Utiliser la Gestion des API dans un réseau virtuel](./api-management-using-with-vnet.md)

* [Pare-feu d’applications web Azure sur Azure Application Gateway](../web-application-firewall/ag/ag-overview.md)

* [Présentation d’Azure Application Gateway](../application-gateway/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Aide** : Utilisez des étiquettes de service de réseau virtuel pour définir des contrôles d’accès réseau dans les groupes de sécurité réseau qui sont utilisés sur vos sous-réseaux de gestion des API. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de la balise de service (par exemple, ApiManagement) dans le champ Source ou Destination approprié d'une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Attention : Lorsque vous configurez un groupe de sécurité réseau sur le sous-réseau de gestion des API, certains ports doivent rester ouverts. Si ces ports ne sont pas disponibles, la gestion des API risque de ne pas fonctionner correctement et d’être inaccessible.

* [Comprendre et utiliser les étiquettes de service](../virtual-network/service-tags-overview.md)

* [Ports nécessaires pour la Gestion des API](./api-management-using-with-vnet.md#-common-network-configuration-issues)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Aide** : Définissez et implémentez des configurations de sécurité standard pour les paramètres réseau relatifs à vos déploiements de gestion des API Azure. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.ApiManagement » et « Microsoft.Network » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de vos déploiements de gestion des API Azure et des ressources associées. Vous pouvez également utiliser des définitions de stratégies intégrées pour Azure Virtual Networks, par exemple :
- DDoS Protection Standard doit être activé

Vous pouvez également utiliser Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les artefacts d’environnement clés, tels que les modèles Resource Manager, le contrôle d’accès en fonction du rôle Azure (Azure RBAC) et les stratégies au sein d’une seule définition de blueprint. Vous pouvez facilement appliquer le blueprint aux nouveaux abonnements et environnements, et ajuster le contrôle et la gestion par le biais du contrôle de version.

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Guide pratique pour créer un blueprint Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : Utilisez des étiquettes pour les groupes de sécurité réseau (NSG) et autres ressources liées à la sécurité réseau et au trafic. Concernant les règles NSG, vous pouvez utiliser le champ « Description » afin de spécifier un besoin métier ou une durée pour l’ensemble des règles qui autorisent le trafic vers un réseau ou à partir de celui-ci.

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

* [Guide pratique pour créer un réseau virtuel](../virtual-network/quick-create-portal.md)

* [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : Utilisez le journal d’activité Azure pour superviser les configurations des ressources réseau et détecter les modifications apportées aux ressources réseau qui sont associées à vos déploiements de gestion des API Azure. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.

* [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

* [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Contrôle de sécurité : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Aide** : Microsoft gère les sources de temps pour la Gestion des API.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Dans Azure Monitor, utilisez un ou plusieurs espaces de travail Log Analytics pour interroger et analyser les données, pour envoyer des journaux vers le Stockage Azure en vue d’un stockage à long terme, d’un archivage ou d’une analyse hors connexion, ou pour exporter des journaux vers d’autres solutions d’analytique situées dans Azure ou ailleurs à l’aide d’Azure Event Hubs. Par défaut, la Gestion des API Azure affiche des journaux et des métriques dans Azure Monitor. Le niveau de détail de la journalisation peut être configuré au niveau du service ou au niveau de l’API.

Outre Azure Monitor, la Gestion des API Azure peut être intégrée à un ou plusieurs services Azure Application Insights. Les paramètres de journalisation d’Application Insights peuvent être configurés au niveau du service ou au niveau de l’API.

Vous pouvez également activer et intégrer des données dans Azure Sentinel ou une solution SIEM tierce.

* [Comment configurer les paramètres de diagnostic](../azure-monitor/platform/diagnostic-settings.md#create-in-azure-portal)

* [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

* [Guide pratique pour bien démarrer avec Azure Monitor et l’intégration d’une solution SIEM tierce](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

* [Créer un pipeline de journalisation et d’analytique personnalisé](./api-management-howto-log-event-hubs.md)

* [Intégration à Azure Application Insights](./api-management-howto-app-insights.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Pour la journalisation d’audit du plan de contrôle, activez les paramètres de diagnostic du journal d’activité Azure, puis envoyez les journaux d’activité vers un espace de travail Log Analytics en vue d’une création de rapports et d’une analyse, vers le Stockage Azure pour une conservation à long terme, ou vers Azure Event Hubs pour une exportation vers d’autres solutions d’analytique situées dans Azure ou ailleurs. À l’aide des données des journaux d’activité Azure, vous pouvez déterminer les éléments « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de contrôle pour le service Gestion des API Azure.

Pour la journalisation d’audit du plan de données, les journaux de diagnostic fournissent des informations détaillées sur les opérations et les erreurs qui sont importantes pour l’audit. Ces informations servent également à résoudre les problèmes. Les journaux de diagnostic diffèrent des journaux d’activité. Le journal d’activité fournit des informations sur les opérations qui ont été effectuées sur vos ressources Azure. Les journaux de diagnostic fournissent des informations sur les opérations effectuées par votre ressource.

* [Activation des paramètres de diagnostic pour le journal d’activité Azure](../azure-monitor/platform/activity-log.md)

* [Activation des paramètres de diagnostic pour la Gestion des API Azure](./api-management-howto-use-azure-monitor.md#activity-logs)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Aide** : Dans Azure Monitor, définissez la période de conservation de votre espace de travail Log Analytics en fonction des obligations réglementaires de votre organisation. Utilisez les comptes de stockage Azure pour le stockage à long terme et l’archivage.

* [Définir les paramètres de conservation des journaux pour les espaces de travail Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

* [Archiver les journaux d’activité dans un compte de stockage Azure](../azure-monitor/platform/resource-logs.md#send-to-azure-storage)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : La Gestion des API Azure émet des journaux et des métriques en continu vers Azure Monitor, ce qui vous donne une visibilité en quasi temps réel de l’état et du niveau d’intégrité de vos API. Avec Azure Monitor et des espaces de travail Log Analytics, vous pouvez consulter, interroger, visualiser, router et archiver des données, configurer des alertes et effectuer des actions sur les métriques et les journaux provenant de la Gestion des API et des ressources associées. Analysez et supervisez les journaux pour détecter les comportements anormaux et examinez régulièrement les résultats.

Si vous le souhaitez, vous pouvez intégrer la Gestion des API à Azure Application Insights afin de l’utiliser comme outil principal ou secondaire de supervision, de suivi, de création de rapports et de création d’alertes.

* [Superviser et consulter les journaux de la Gestion des API Azure](./api-management-howto-use-azure-monitor.md)

* [Guide pratique pour effectuer des requêtes personnalisées dans Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

* [Présentation de l’espace de travail Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md)

* [Intégration à Azure Application Insights](./api-management-howto-app-insights.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer des alertes en cas d’activité anormale

**Aide** : Activez les paramètres de diagnostic des journaux d’activité Azure, ainsi que les paramètres de diagnostic de vos instances Gestion des API Azure, puis envoyez les journaux vers un espace de travail Log Analytics. Effectuez des requêtes dans Log Analytics pour rechercher des termes, identifier des tendances, analyser des modèles et fournir de nombreux autres insights basés sur les données collectées. Vous pouvez créer des alertes basées sur vos requêtes d’espace de travail Log Analytics.

Créez des alertes de métriques pour être prévenu lorsqu’un événement inattendu se produit. Par exemple, recevez des notifications lorsque votre instance de la Gestion des API Azure a dépassé sa capacité maximale attendue sur une période donnée, ou si plusieurs requêtes ou erreurs de passerelle non autorisées ont lieu au cours d’une période prédéfinie.

Si vous le souhaitez, vous pouvez intégrer la Gestion des API à Azure Application Insights afin de l’utiliser comme outil principal ou secondaire de supervision, de suivi, de création de rapports et de création d’alertes.

Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.

* [Activation des paramètres de diagnostic pour le journal d’activité Azure](../azure-monitor/platform/activity-log.md)

* [Activation des paramètres de diagnostic pour la Gestion des API Azure](./api-management-howto-use-azure-monitor.md#activity-logs)

* [Configurer une règle d’alerte pour Gestion des API Azure](./api-management-howto-use-azure-monitor.md#set-up-an-alert-rule)

* [Afficher les métriques de capacité d’une instance Gestion des API Azure](./api-management-capacity.md)

* [Intégration à Azure Application Insights](./api-management-howto-app-insights.md)

* [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide** : Non applicable. La Gestion des API Azure ne traite pas et ne produit pas de journaux liés aux logiciels anti-programmes malveillants.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Conseils** : Non applicable. La Gestion des API Azure ne traite pas et ne produit pas de journaux DNS accessibles par les utilisateurs.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Tenez à jour un inventaire des comptes qui disposent d’un accès administratif au plan de contrôle de la Gestion des API Azure (portail Azure).

Azure Active Directory (AD) comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent être interrogés. La Gestion des API utilise ces rôles ainsi que le contrôle d’accès en fonction du rôle (RBAC) pour gérer de façon précise l’accès aux services et aux entités de la Gestion des API.

En outre, la Gestion des API contient un groupe d’administrateurs intégré dans son système utilisateur. Dans la Gestion des API, les groupes permettent de contrôler la visibilité des API dans le portail des développeurs. En outre, les membres du groupe Administrateurs peuvent voir toutes les API.

Suivez les recommandations d’Azure Security Center concernant la gestion et la maintenance des comptes d’administration.

* [Guide pratique pour utiliser le contrôle d’accès en fonction du rôle dans Gestion des API Azure](./api-management-role-based-access-control.md)

* [Obtenir la liste des utilisateurs d’une instance Gestion des API Azure](/powershell/module/az.apimanagement/get-azapimanagementuser)

* [Obtenir la liste des utilisateurs affectés à un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/az.resources/get-azroleassignment)

* [Obtenir une définition de rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/az.resources/get-azroledefinition)

* [Recommandations Azure Security Center concernant l’accès et les identités](../security-center/recommendations-reference.md#recs-identity)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Aide** : La Gestion des API Azure n’intègre pas le concept des mots de passe ou clés par défaut.

Les abonnements Gestion des API Azure, qui sont un moyen de sécuriser l’accès aux API, sont toutefois fournis avec une paire de clés d’abonnement générées. Les clients peuvent regénérer ces clés d’abonnement à tout moment.

* [Abonnements à la Gestion des API Azure](./api-management-subscriptions.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

En outre, pour vous aider à suivre les comptes d’administration dédiés, vous pouvez utiliser des recommandations d’Azure Security Center ou des stratégies Azure intégrées, telles que les suivantes :
- Plusieurs propriétaires doivent être affectés à votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

* [Utilisation d’Azure Security Center pour superviser l’identité et l’accès (préversion)](../security-center/security-center-identity-access.md)

* [Utilisation d’Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Aide** : La Gestion des API Azure peut être configurée de manière à utiliser Azure Active Directory en tant que fournisseur d’identité pour authentifier les utilisateurs sur le portail des développeurs et tirer parti des fonctionnalités d’authentification unique proposées par Azure AD. Une fois ceci configuré, les nouveaux utilisateurs du portail des développeurs peuvent choisir de suivre le processus d’inscription prêt à l’emploi en s’authentifiant d’abord par le biais d’Azure AD, puis en terminant le processus d’inscription sur le portail une fois qu’ils sont authentifiés.

* [Autoriser des comptes de développeurs à l’aide d’Azure Active Directory dans Gestion des API Azure](./api-management-howto-aad.md)

Le processus de connexion ou d’inscription peut également être personnalisé par le biais de la délégation. La délégation vous permet d’utiliser votre site web existant pour gérer les connexions/inscriptions des développeurs et l’abonnement aux produits au lieu de faire appel aux fonctionnalités intégrées du portail des développeurs. Elle permet à votre site web de conserver les données utilisateur et de valider ces étapes de façon personnalisée.

* [Délégation de l'inscription des utilisateurs et des abonnements aux produits](./api-management-howto-setup-delegation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Aide** : Activez l’authentification multifacteur (MFA) Azure Active Directory (AAD) et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

* [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

* [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils** : Utilisez des stations de travail disposant d’un accès privilégié avec Multi-Factor Authentication (MFA) configuré pour vous connecter aux ressources Azure et les configurer.

* [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration

**Aide** : Utilisez Azure Active Directory (AD) Privileged Identity Management pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.

Utilisez également les détections de risque Azure AD pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs.

* [Déploiement de Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

* [Présentation des détections de risques Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Utilisez des emplacements nommés avec accès conditionnel pour autoriser l’accès au Portail Azure uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

* [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Dans la mesure du possible, utilisez Azure AD comme système central d’authentification et d’autorisation. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

Configurez le portail des développeurs où se trouve la Gestion des API Azure de manière à authentifier les comptes de développeurs à l’aide d’Azure Active Directory.

Configurez votre instance Gestion des API pour protéger vos API à l’aide du protocole OAuth 2.0 et d’Azure Active Directory (AD).

* [Autoriser des comptes de développeurs à l’aide d’Azure Active Directory dans la Gestion des API Azure](./api-management-howto-aad.md)

* [Protéger une API à l’aide d’OAuth 2.0 avec Azure Active Directory et la Gestion des API](./api-management-howto-protect-backend-with-aad.md)

* [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory fournit des journaux pour vous aider à découvrir les comptes obsolètes. Les clients peuvent utiliser les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès utilisateur peut être vérifié régulièrement afin de s’assurer que seuls les utilisateurs prévus continuent d’avoir un accès.

Les clients peuvent gérer l’inventaire des comptes d’utilisateur de la Gestion des API afin de vérifier l’accès de chacun. Dans la Gestion des API, les développeurs sont les utilisateurs des API que vous exposez via la Gestion des API. Par défaut, les comptes de développeurs nouvellement créés sont actifs, et sont associés au groupe Développeurs. Les comptes de développeurs dont l’état est Actif peuvent être utilisés pour accéder à toutes les API auxquelles ils sont abonnés.

Les administrateurs peuvent créer des groupes personnalisés ou utiliser des groupes externes dans des locataires Azure Active Directory qui leur sont associés. Des groupes externes et personnalisés peuvent être utilisés avec des groupes système offrant une certaine visibilité aux développeurs et un accès aux produits d’API.

* [Gestion des comptes d’utilisateur dans Gestion des API Azure](./api-management-howto-create-or-invite-developers.md)

* [Obtenir la liste des utilisateurs de la Gestion des API](/powershell/module/az.apimanagement/get-azapimanagementuser)

* [Création et utilisation de groupes pour gérer les comptes de développeurs dans Gestion des API Azure](./api-management-howto-create-groups.md)

* [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés

**Aide** : Configurez votre instance Gestion des API Azure pour authentifier les comptes de développeur en utilisant Azure Active Directory comme un fournisseur d’identité dans la Gestion des API Azure.

Configurez votre instance Gestion des API pour protéger vos API à l’aide du protocole OAuth 2.0 et d’Azure Active Directory (AD).

Configurez la stratégie de validation JWT pour les requêtes d’API entrantes afin de confirmer l’existence et la validité d’un jeton valide.

Créez des paramètres de diagnostic pour les comptes d’utilisateur Azure AD. Ensuite, envoyez les journaux d’audit et les journaux de connexion vers un espace de travail Log Analytics. Configurez les alertes souhaitées dans Log Analytics. En outre, vous pouvez intégrer l’espace de travail Log Analytics à Azure Sentinel ou à une solution SIEM tierce.

Configurez une supervision avancée avec la Gestion des API à l’aide de la stratégie `log-to-eventhub`, capturez toutes les informations de contexte supplémentaires qui sont nécessaires à l’analyse de la sécurité, puis envoyez-les à Azure Sentinel ou à une solution SIEM tierce.

* [Autoriser des comptes de développeurs à l’aide d’Azure Active Directory dans la Gestion des API Azure](./api-management-howto-aad.md)

* [Protéger une API à l’aide d’OAuth 2.0 avec Azure Active Directory et la Gestion des API](./api-management-howto-protect-backend-with-aad.md)

* [Stratégies de restriction des accès de la Gestion des API](./api-management-access-restriction-policies.md)

* [Intégrer des journaux Azure AD à Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

* [Procédure d’intégration d’Azure Sentinel](../sentinel/quickstart-onboard.md)

* [Supervision avancée des API](./api-management-log-to-eventhub-sample.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Aide** : Pour les écarts de comportement des connexions au compte dans le plan de contrôle (portail Azure), utilisez Azure Active Directory (AD) Identity Protection et les fonctionnalités de détections de risques pour configurer des réponses automatiques aux actions suspectes détectées , concernant les identités d’utilisateur. Vous pouvez également ingérer des données dans Azure Sentinel pour approfondir votre examen.

* [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

* [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

* [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Aide** : Non disponible actuellement. Customer Lockbox n’est pas pris en charge pour la Gestion des API Azure.

* [Liste des services pris en charge pour Customer Lockbox](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Les instances Gestion des API Azure doivent être séparées par un réseau ou un sous-réseau virtuel, et être étiquetées correctement.

* [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

* [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

* [Utilisation de la gestion des API Azure avec des réseaux virtuels](./api-management-using-with-vnet.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Aide** : Non disponible. Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données, ne sont pas disponibles pour la Gestion des API Azure.

Microsoft gère l’infrastructure sous-jacente de la Gestion des API Azure, et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition des données client.

* [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Aide** : Les appels de plan de gestion se font par le biais d’Azure Resource Manager sur TLS. Un jeton web JSON valide (JWT) est nécessaire. Les appels de plan de données peuvent être sécurisés via TLS et un des mécanismes d’authentification pris en charge (certificat client ou JWT, par exemple).

* [Présentation de la protection des données dans la Gestion des API Azure](#data-protection)

* [Gérer les paramètres TLS dans la Gestion des API Azure](./api-management-howto-manage-protocols-ciphers.md)

* [Protéger les API dans la Gestion des API Azure avec Azure Active Directory](./api-management-howto-protect-backend-with-aad.md)

* [Protéger les API dans la Gestion des API Azure avec Azure Active Directory B2C](./howto-protect-backend-frontend-azure-ad-b2c.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Aide** : Non disponible. Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données, ne sont pas disponibles pour la Gestion des API Azure. Étiquetez les services Gestion des API Azure qui peuvent traiter des informations sensibles en tant que telles, et implémentez une solution tierce à des fins de conformité, si nécessaire.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

* [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Aide** : Utilisez le contrôle d’accès en fonction du rôle pour contrôler l’accès à la Gestion des API Azure. Gestion des API Azure utilise le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour gérer de façon précise l’accès aux services et aux entités Gestion des API (par exemple, les API et les stratégies).

* [Guide pratique pour utiliser le contrôle d’accès en fonction du rôle dans Gestion des API Azure](./api-management-role-based-access-control.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

Microsoft gère l’infrastructure sous-jacente de la Gestion des API Azure, et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition des données client.

* [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide** : Les données sensibles telles que les certificats, les clés et les valeurs secrètes sont chiffrées à l’aide de clés d’instance de service gérées par le service. Toutes les clés de chiffrement sont propres à l'instance de service et gérées par ce dernier.

* [Protection et chiffrement des données au repos avec la Gestion des API Azure](#data-protection)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes en cas de modifications apportées aux applications Azure Functions de production et aux autres ressources critiques ou associées.

* [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

* [Utiliser Azure Monitor et le journal d’activité Azure dans la Gestion des API Azure](./api-management-howto-use-azure-monitor.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Aide** : Non disponible. L’évaluation des vulnérabilités dans Azure Security Center n’est pas disponible pour la Gestion des API Azure.

La plateforme sous-jacente est analysée et corrigée par Microsoft. Passez en revue les contrôles de sécurité disponibles afin de réduire les vulnérabilités liées à la configuration du service.

* [Contrôles de sécurité disponibles pour la Gestion des API Azure]()

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3 : Déployer une solution de gestion de correctif logiciel tiers automatisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : Non disponible. L’évaluation des vulnérabilités dans Azure Security Center n’est pas disponible pour la Gestion des API Azure.

La plateforme sous-jacente est analysée et corrigée par Microsoft. Les clients passent en revue les contrôles de sécurité disponibles afin de réduire les vulnérabilités liées à la configuration du service.

* [Contrôles de sécurité disponibles pour la Gestion des API Azure]()

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1 : Utiliser la découverte de ressources Azure

**Conseils** : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (telles que calcul, stockage, réseau, ports et protocoles) dans vos abonnements. Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

* [Guide pratique pour créer des requêtes avec Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

* [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription)

* [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

* [Créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Aide** : Utilisez des étiquettes, des groupes d’administration et des abonnements distincts, le cas échéant, pour organiser et suivre les ressources Azure. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

Appliquez également des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant la stratégie Azure avec les définitions intégrées suivantes :
- Types de ressources non autorisés
- Types de ressources autorisés

* [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

* [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4 : Tenir un inventaire des ressources Azure approuvées et titres des logiciels

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements, en utilisant Azure Policy avec les définitions intégrées suivantes :
- Types de ressources non autorisés
- Types de ressources autorisés

Utilisez Azure Resource Graph pour interroger/découvrir des ressources dans leurs abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :
- Types de ressources non autorisés
- Types de ressources autorisés

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-implement-approved-application-list"></a>6.10 : Implémenter une liste d’applications approuvées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resources-manager-via-scripts"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager par le biais de scripts

**Aide** : Configurez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

* [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

* [Contrôle d’accès en fonction du rôle dans la Gestion des API Azure](./api-management-role-based-access-control.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Définissez et implémentez des configurations de sécurité standard pour votre service Gestion des API Azure avec Azure Policy. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.ApiManagement » pour créer des stratégies personnalisées en vue d’auditer ou d’appliquer la configuration de vos services Gestion des API Azure.

* [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias)

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Définissez et implémentez des configurations de sécurité standard pour vos services Gestion des API Azure avec Azure Policy. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.ApiManagement » pour créer des stratégies personnalisées en vue d’auditer ou d’appliquer la configuration des instances Gestion des API Azure. Utilisez Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Présentation des effets d’Azure Policy](../governance/policy/concepts/effects.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous utilisez des définitions de stratégie Azure personnalisées, utilisez Azure DevOps ou Azure Repos pour stocker et gérer la configuration de votre service Gestion des API Azure de façon sécurisée.

* [Stocker des fichiers dans Azure DevOps](/azure/devops/repos/git/gitworkflow)

* [Documentation Azure Repos](/azure/devops/repos/index)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="77-deploy-system-configuration-management-tools"></a>7.7 : Déployer les outils de gestion de configuration système

**Aide** : Définissez et implémentez des configurations de sécurité standard pour vos services Gestion des API Azure avec Azure Policy. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.ApiManagement » pour créer des stratégies personnalisées en vue d’auditer ou d’appliquer la configuration des instances Gestion des API Azure. Utilisez Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Présentation des effets d’Azure Policy](../governance/policy/concepts/effects.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la surveillance de la configuration automatique pour les services Azure

**Conseils** : Utilisez le kit de ressources Azure API Management DevOps pour gérer la configuration de la Gestion des API Azure.

En outre, définissez et implémentez des configurations de sécurité standard pour vos services Gestion des API Azure avec Azure Policy. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.ApiManagement » pour créer des stratégies personnalisées en vue d’auditer ou d’appliquer la configuration des instances Gestion des API Azure. Utilisez Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Présentation des effets d’Azure Policy](../governance/policy/concepts/effects.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Aide** : Utilisez Key Vault pour gérer les certificats et configurer la rotation automatique. Si vous utilisez Azure Key Vault pour gérer le certificat SSL du domaine personnalisé, vérifiez que le certificat est inséré dans Key Vault comme un certificat, et non comme un secret.

* [Définir des noms de domaine personnalisés avec les recommandations relatives à la rotation des clés Key Vault](./configure-custom-domain.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Microsoft

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Aide** : Utilisez une identité de service managée générée par Azure Active Directory (AD) pour permettre à votre instance Gestion des API d’accéder facilement et de façon sécurisée aux autres ressources protégées par Azure AD telles qu’Azure Key Vault.

* [Créer une identité managée pour une instance Gestion des API](./api-management-howto-use-managed-service-identity.md)

* [Stratégie pour s’authentifier avec une identité managée](./api-management-authentication-policies.md#ManagedIdentity)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

* [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (comme la Gestion des API Azure). Toutefois, il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Aide** : Non applicable. Cette recommandation concerne les ressources conçues pour stocker des données, et non à des fins de calcul.

Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (comme la Gestion des API Azure). Toutefois, il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Aide** : Non applicable. Cette recommandation concerne les ressources conçues pour stocker des données, et non à des fins de calcul.

Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (comme la Gestion des API Azure). Toutefois, il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-recovery"></a>Récupération de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Aide** : En publiant et en gérant vos API via Gestion des API Azure, vous bénéficiez de fonctionnalités de tolérance de panne et d’infrastructure que vous n’avez plus à concevoir, implémenter ou gérer manuellement. La Gestion des API prend en charge le déploiement multirégion, ce qui rend le plan de données imperméable aux pannes régionales, sans ajouter de surcharge opérationnelle.

La fonctionnalité de sauvegarde et de restauration de service de la Gestion des API fournit les bases nécessaires pour implémenter votre stratégie de reprise d’activité. Les opérations de sauvegarde et de restauration peuvent être effectuées manuellement ou automatiquement.

* [Déployer un plan de données Gestion des API dans plusieurs régions](./api-management-howto-deploy-multi-region.md)

* [Comment implémenter une récupération d’urgence à l’aide d’une sauvegarde de service et la récupérer dans Gestion des API Azure](./api-management-howto-disaster-recovery-backup-restore.md#calling-the-backup-and-restore-operations)

* [Appeler l’opération de sauvegarde de la Gestion des API](/rest/api/apimanagement/2019-12-01/apimanagementservice/backup)

* [Appeler l’opération de restauration de la Gestion des API](/rest/api/apimanagement/2019-12-01/apimanagementservice/restore)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Aide** : Les opérations de sauvegarde et de restauration fournies par la Gestion des API Azure permettent une sauvegarde et une restauration complètes du système.

Vous pouvez utiliser des identités managées afin d’obtenir des certificats pour les noms de domaine personnalisés du service Gestion des API, à partir d’Azure Key Vault. Sauvegardez les certificats stockés dans Azure Key Vault.

* [Comment implémenter une récupération d’urgence à l’aide d’une sauvegarde de service et la récupérer dans Gestion des API Azure](./api-management-howto-disaster-recovery-backup-restore.md#calling-the-backup-and-restore-operations)

* [Sauvegarder des certificats Azure Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultcertificate)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : valider toutes les sauvegardes, y compris les clés gérées par le client

**Aide** : Validez les sauvegardes en effectuant un test de restauration du service et des certificats à partir des sauvegardes.

* [Appeler l’opération de restauration de la Gestion des API](/rest/api/apimanagement/2019-12-01/apimanagementservice/restore)

* [Restaurer des certificats Azure Key Vault](/powershell/module/azurerm.keyvault/restore-azurekeyvaultcertificate)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : La Gestion des API Azure écrit les sauvegardes dans les comptes de stockage Azure du client. Suivez les recommandations de sécurité relatives au Stockage Azure pour protéger votre sauvegarde.

* [Comment implémenter une récupération d’urgence à l’aide d’une sauvegarde de service et la récupérer dans Gestion des API Azure](./api-management-howto-disaster-recovery-backup-restore.md#calling-the-backup-and-restore-operations)

* [Recommandation de sécurité pour Stockage Blob](../storage/blobs/security-recommendations.md)

Activez la suppression réversible dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.

* [Activer la suppression réversible dans Key Vault](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

* [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

* [Tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de votre propre plan de réponse aux incidents](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez clairement les abonnements (par ex. production, non-production) à l’aide d’étiquettes et créez un système de nommage pour identifier et classer clairement les ressources Azure, en particulier celles qui traitent des données sensibles. Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

* [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md)

* [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Effectuez des exercices pour tester les capacités de réponse aux incidents de vos systèmes à intervalles réguliers, afin de protéger vos ressources Azure. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

* [Publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Instructions** : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

* [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Aide** : Exportez vos alertes et recommandations Azure Security Center en utilisant la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel.

* [Comment configurer l’exportation continue](../security-center/continuous-export.md)

* [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Aide** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » en cas d’alertes et de recommandations de sécurité.

* [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-to-remediate-all-critical-security-findings-within-60-days"></a>11.1 : Effectuer régulièrement des tests d’intrusion de vos ressources Azure et résoudre tous les problèmes de sécurité critiques détectés dans un délai de 60 jours

**Conseil** : * [Respectez les règles d’engagement de Microsoft pour garantir la conformité de vos tests d’intrusion aux stratégies Microsoft](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1.)

* [Vous trouverez ici plus d’informations sur la stratégie de Microsoft, sur l’exécution de Red Teaming et sur les tests d’intrusion de site actif sur l’infrastructure, les services et les applications cloud qui sont managés par Microsoft.](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)