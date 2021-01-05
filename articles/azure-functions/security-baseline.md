---
title: Base de référence de sécurité Azure pour Azure Functions
description: Base de référence de sécurité Azure pour Azure Functions
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 05/04/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 86e13c0f57b30d2c53d9194e89ec89e7abdf1574
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96187049"
---
# <a name="azure-security-baseline-for-azure-functions"></a>Base de référence de sécurité Azure pour Azure Functions

La base de référence de sécurité Azure pour Azure Functions contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez la [Vue d’ensemble des bases de référence de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1 : Protéger les ressources à l'aide de groupes de sécurité réseau ou du Pare-feu Azure sur votre réseau virtuel

**Aide** : Intégrez vos applications Azure Functions à un réseau virtuel Azure. Les applications de fonction qui s’exécutent dans le plan Premium ont les mêmes capacités d’hébergement que les applications web dans Azure App Service, ce qui inclut la fonctionnalité d’intégration au réseau virtuel.  Les réseaux virtuels Azure vous permettent de placer un grand nombre de vos ressources Azure, par exemple Azure Functions, dans un réseau routable non Internet.

- [Guide pratique pour intégrer Azure Functions à un réseau virtuel Azure](./functions-create-vnet.md)

- [Comprendre l’intégration au réseau virtuel pour Azure Functions et Azure App Service](../app-service/web-sites-integrate-with-vnet.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2 : Surveiller et consigner la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Aide** : Utilisez Azure Security Center et suivez les recommandations de protection de réseau pour sécuriser les ressources réseau et les configurations réseau liées à vos applications Azure Functions.

Si vous utilisez des NSG (groupes de sécurité réseau) dans votre implémentation d’Azure Functions, activez les journaux de flux NSG et envoyez-les vers un compte de stockage Azure pour les audits de trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

- [Présentation de la sécurité réseau assurée par Azure Security Center](../security-center/security-center-network-recommendations.md)

- [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Guide pratique pour activer et utiliser Traffic Analytics](../network-watcher/traffic-analytics.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Aide** : Pour sécuriser complètement vos points de terminaison Azure Functions en production, implémentez l’une des options de sécurité suivantes au niveau de l’application de fonction :
- Activer l’authentification/autorisation App Service pour votre application de fonction
- Utiliser le service Azure APIM (Gestion des API) pour authentifier les requêtes
- Déployer l’application de fonction sur Azure App Service Environment.

De plus, vérifiez que le débogage à distance a été désactivé pour vos ressources Azure Functions de production. De plus, le mécanisme CORS (Cross-Origin Resource Sharing) ne doit pas autoriser tous les domaines à accéder à votre application Azure Functions. Autorisez uniquement les domaines nécessaires à interagir avec votre application Azure Functions.

Déployez le pare-feu WAF (pare-feu d’applications web) dans le cadre de la configuration réseau pour bénéficier d’une inspection supplémentaire du trafic entrant. Activez le paramètre de diagnostic du WAF et ingérez les journaux dans un compte de stockage, un hub d'événements ou un espace de travail Log Analytics. 

- [Guide pratique pour sécuriser les points de terminaison Azure Functions en production](./functions-bindings-http-webhook-trigger.md?tabs=csharp#secure-an-http-endpoint-in-production)

- [Guide pratique pour déployer Azure WAF](../web-application-firewall/ag/create-waf-policy-ag.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Aide** : Activez le service Protection DDoS Standard sur les réseaux virtuels associés à vos applications Azure Functions pour vous protéger contre les attaques DDoS. Utilisez Azure Security Center Integrated Threat Intelligence pour refuser les communications avec les adresses IP publiques connues comme étant malveillantes ou inutilisées.
De plus, configurez une passerelle front-end, par exemple le pare-feu d’applications web Azure, pour authentifier toutes les requêtes entrantes et filtrer le trafic malveillant. Le pare-feu d’applications web permet de sécuriser vos applications Azure Functions en inspectant le trafic web entrant pour bloquer les attaques par injection de code SQL, les attaques XSS (scripts intersites), les attaques par chargement de programmes malveillants ainsi que les attaques DDoS. L’introduction d’un WAF nécessite la fonctionnalité App Service Environment ou l’utilisation de points de terminaison privés (préversion). Vérifiez que les points de terminaison privés ne sont plus en (préversion) avant de les utiliser avec des charges de travail de production.

- [Options de mise en réseau d’Azure Functions](./functions-networking-options.md)

- [Plan Premium Azure Functions](./functions-scale.md#premium-plan)

- [Introduction aux environnements App Service](../app-service/environment/intro.md)

- [Considérations relatives à la mise en réseau pour un environnement App Service](../app-service/environment/network-info.md)

- [Guide pratique pour configurer la protection DDoS](../ddos-protection/manage-ddos-protection.md)

- [Guide pratique pour déployer le Pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md)

- [Présentation de la fonctionnalité Threat Intelligence intégrée à Azure Security Center](../security-center/azure-defender.md)

- [Présentation de la fonctionnalité de renforcement du réseau adaptatif d’Azure Security Center](../security-center/security-center-adaptive-network-hardening.md)

- [Présentation de la fonctionnalité de contrôle d’accès réseau juste-à-temps d’Azure Security Center](../security-center/security-center-just-in-time.md)

- [Utilisation de points de terminaison privés pour Azure Functions](../app-service/networking/private-endpoint.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5 : Consigner les paquets réseau et les journaux de flux

**Aide** : Si vous utilisez des NSG (groupes de sécurité réseau) dans votre implémentation d’Azure Functions, activez les journaux de flux de groupe de sécurité réseau, puis envoyez-les vers un compte de stockage pour un audit du trafic. Vous pouvez également envoyer les journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour obtenir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

- [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Guide pratique pour activer et utiliser Traffic Analytics](../network-watcher/traffic-analytics.md)

- [Guide pratique pour activer Network Watcher](../network-watcher/network-watcher-create.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Aide** : Configurez une passerelle front-end, par exemple le pare-feu d’applications web Azure, pour authentifier toutes les requêtes entrantes et filtrer le trafic malveillant. Le pare-feu d’applications web Azure permet de sécuriser vos applications Functions en inspectant le trafic web entrant pour bloquer les attaques par injection de code SQL, les attaques XSS (scripts intersites), les attaques par chargement de programmes malveillants ainsi que les attaques DDoS. L’introduction d’un WAF nécessite la fonctionnalité App Service Environment ou l’utilisation de points de terminaison privés (préversion). Vérifiez que les points de terminaison privés ne sont plus en (préversion) avant de les utiliser avec des charges de travail de production.

Il existe également plusieurs options de place de marché, par exemple le WAF Barracuda pour Azure, disponibles sur la Place de Marché Azure, qui incluent des fonctionnalités IDS/IPS.

- [Options de mise en réseau d’Azure Functions](./functions-networking-options.md)

- [Plan Premium Azure Functions](./functions-scale.md#premium-plan)

- [Introduction aux environnements App Service](../app-service/environment/intro.md)

- [Considérations relatives à la mise en réseau pour un environnement App Service](../app-service/environment/network-info.md)

- [Présentation du pare-feu d’applications web Azure](../web-application-firewall/index.yml)

- [Utilisation de points de terminaison privés pour Azure Functions](../app-service/networking/private-endpoint.md)

- [Présentation du service cloud WAF Barracuda](../app-service/environment/app-service-app-service-environment-web-application-firewall.md#configuring-your-barracuda-waf-cloud-service)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Aide** : Configurez une passerelle front-end pour votre réseau, par exemple le pare-feu d’applications web Azure avec un chiffrement TLS de bout en bout. L’introduction d’un WAF nécessite la fonctionnalité App Service Environment ou l’utilisation de points de terminaison privés (préversion). Vérifiez que les points de terminaison privés ne sont plus en (préversion) avant de les utiliser avec des charges de travail de production.

- [Options de mise en réseau d’Azure Functions](./functions-networking-options.md)

- [Plan Premium Azure Functions](./functions-scale.md#premium-plan)

- [Introduction aux environnements App Service](../app-service/environment/intro.md)

- [Considérations relatives à la mise en réseau pour un environnement App Service](../app-service/environment/network-info.md)

- [Présentation du pare-feu d’applications web Azure](../web-application-firewall/index.yml)

- [Guide pratique pour la configuration du chiffrement TLS de bout en bout avec Application Gateway et le portail](../application-gateway/end-to-end-ssl-portal.md)

- [Utilisation de points de terminaison privés pour Azure Functions](../app-service/networking/private-endpoint.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Aide** : Utilisez les étiquettes du service Réseau virtuel pour définir des contrôles d’accès réseau sur les groupes de sécurité réseau ou le Pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de l’étiquette de service (par exemple AzureAppService) dans le champ Source ou Destination approprié d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

- [Pour plus d’informations sur l’utilisation des étiquettes de service](../virtual-network/service-tags-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Aide** : Définissez et implémentez des configurations de sécurité standard pour les paramètres réseau liés à votre instance d’Azure Functions. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.Web » et « Microsoft.Network » pour créer des stratégies personnalisées d’audit ou d’application de la configuration réseau de vos applications Azure Functions. Vous pouvez également utiliser des définitions de stratégie intégrées pour Azure Functions, par exemple :
- CORS ne doit pas autoriser toutes les ressources à accéder à vos applications de fonction
- Function App ne doit pas être accessible via HTTPS
- La dernière version de TLS doit être utilisée dans votre application de fonction

Vous pouvez également utiliser Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les artefacts d’environnement clés, tels que les modèles Resource Manager, le contrôle d’accès en fonction du rôle Azure (Azure RBAC) et les stratégies au sein d’une seule définition de blueprint. Vous pouvez facilement appliquer le blueprint aux nouveaux abonnements et environnements, et ajuster le contrôle et la gestion par le biais du contrôle de version.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer un blueprint Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Aide** : Si vous utilisez des NSG (groupes de sécurité réseau) dans votre implémentation d’Azure Functions, utilisez des étiquettes pour les NSG et autres ressources liées à la sécurité réseau et au flux de trafic. Concernant les règles NSG individuelles, utilisez le champ « Description » afin de spécifier le besoin métier et/ou la durée (etc.) pour toutes les règles qui autorisent le trafic vers/depuis un réseau.

Utilisez l’une des définitions de stratégie Azure intégrée en lien avec l’étiquetage, telles que « Exiger une étiquette et sa valeur » pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Aide** : Utilisez le journal d’activité Azure pour superviser les configurations des ressources réseau et détecter les changements apportés aux paramètres et ressources réseau liés à vos déploiements d’Azure Functions. Créez des alertes dans Azure Monitor, qui se déclenchent quand des changements sont apportés à des paramètres ou ressources réseau critiques. 

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Contrôle de sécurité : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Aide** : Microsoft conserve la source de temps utilisée pour les ressources Azure, par exemple Azure Functions, dans le cadre des horodatages des journaux.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Pour la journalisation d’audit du plan de contrôle, activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux vers un espace de travail Log Analytics, Event Hub ou un compte de stockage Azure pour archivage. À l’aide des données des journaux d’activité Azure, vous pouvez déterminer les éléments « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de contrôle pour vos ressources Azure.

Azure Functions est également intégré à Azure Application Insights pour la supervision des fonctions. Application Insights collecte des données de journal, de performances et d’erreur. Il détecte automatiquement les anomalies de performances et intègre de puissants outils d’analyse conçus pour aider à diagnostiquer les problèmes et à comprendre la manière dont vos fonctions sont utilisées.

Si vous disposez d’une journalisation de sécurité/d’audit personnalisée intégrée dans votre application de fonction Azure, activez le paramètre de diagnostic « FunctionAppLogs », et envoyez les journaux à un espace de travail Log Analytics, un hub d’événements Azure ou un compte de stockage Azure à des fins d’archivage. 

Vous pouvez éventuellement activer et intégrer les données dans Azure Sentinel ou une solution SIEM (Informations de sécurité et gestion d’événements) tierce. 

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Guide pratique pour configurer Azure Functions avec Azure Application Insights](./functions-monitoring.md)

- [Guide pratique pour activer les paramètres de diagnostic (journaux générés par l’utilisateur) pour Azure Functions](./functions-monitor-log-analytics.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Aide** : Pour la journalisation d’audit du plan de contrôle, activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux vers un espace de travail Log Analytics, Event Hub ou un compte de stockage Azure pour archivage. À l’aide des données des journaux d’activité Azure, vous pouvez déterminer les éléments « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de contrôle pour vos ressources Azure.

Si vous disposez d’une journalisation de sécurité/d’audit personnalisée intégrée dans votre application de fonction Azure, activez le paramètre de diagnostic « FunctionAppLogs », et envoyez les journaux à un espace de travail Log Analytics, un hub d’événements Azure ou un compte de stockage Azure à des fins d’archivage. 

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Guide pratique pour activer les paramètres de diagnostic (journaux générés par l’utilisateur) pour Azure Functions](./functions-monitor-log-analytics.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Aide** : Dans Azure Monitor, définissez la période de conservation des journaux pour les espaces de travail Log Analytics associés à vos applications Azure Functions conformément aux réglementations de conformité de votre organisation.

- [Guide pratique pour définir les paramètres de conservation des journaux](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Activez les paramètres de diagnostic du journal d’activité Azure ainsi que les paramètres de diagnostic de votre application Azure Functions, puis envoyez les journaux à un espace de travail Log Analytics. Effectuez des requêtes dans Log Analytics pour rechercher des termes, identifier des tendances, analyser des modèles et fournir de nombreux autres insights basés sur les données collectées.

Activez Application Insights pour vos applications Azure Functions afin de collecter les données de journal, de performances et d’erreur. Vous pouvez voir les données de télémétrie collectées par Application Insights dans le portail Azure.

Si vous disposez d’une journalisation de sécurité/d’audit personnalisée intégrée dans votre application de fonction Azure, activez le paramètre de diagnostic « FunctionAppLogs », et envoyez les journaux à un espace de travail Log Analytics, un hub d’événements Azure ou un compte de stockage Azure à des fins d’archivage. 

Vous pouvez éventuellement activer et intégrer les données dans Azure Sentinel ou une solution SIEM (Informations de sécurité et gestion d’événements) tierce. 

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Guide pratique pour activer les paramètres de diagnostic pour Azure Functions](./functions-monitor-log-analytics.md)

- [Guide pratique pour configurer Azure Functions avec Azure Application Insights et voir les données de télémétrie](./functions-monitoring.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer des alertes en cas d’activité anormale

**Aide** : Activez les paramètres de diagnostic du journal d’activité Azure ainsi que les paramètres de diagnostic de votre application Azure Functions, puis envoyez les journaux à un espace de travail Log Analytics. Effectuez des requêtes dans Log Analytics pour rechercher des termes, identifier des tendances, analyser des modèles et fournir de nombreux autres insights basés sur les données collectées. Vous pouvez créer des alertes en fonction de vos requêtes d’espace de travail Log Analytics.

Activez Application Insights pour vos applications Azure Functions afin de collecter les données de journal, de performances et d’erreur. Vous pouvez voir les données de télémétrie collectées par Application Insights, et créer des alertes dans le portail Azure.

Vous pouvez éventuellement activer et intégrer les données dans Azure Sentinel ou une solution SIEM (Informations de sécurité et gestion d’événements) tierce. 

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Guide pratique pour activer les paramètres de diagnostic pour Azure Functions](./functions-monitor-log-analytics.md)

- [Guide pratique pour activer Application Insights pour Azure Functions](./configure-monitoring.md#enable-application-insights-integration)

- [Guide pratique pour créer des alertes dans Azure](../azure-monitor/learn/tutorial-response.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide** : Non applicable. Les applications Azure Functions ne traitent pas ou ne produisent pas de journaux liés aux logiciels antiprogramme malveillant.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Aide** : Non applicable. Les applications Azure Functions ne traitent pas ou ne produisent pas de journaux DNS accessibles à l’utilisateur.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Azure Active Directory (AD) comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent être interrogés. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration. 

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : L’accès au plan de contrôle par Azure Functions est contrôlé via Azure Active Directory (AD). Azure AD n’intègre pas le concept des mots de passe par défaut.

L’accès au plan de données peut être contrôlé par plusieurs moyens, notamment les clés d’autorisation, les restrictions de réseau et la validation d’une identité Azure AD. Les clés d’autorisation sont utilisées par les clients qui se connectent à vos points de terminaison HTTP Azure Functions, et peuvent être regénérées à tout moment. Ces clés sont générées pour les nouveaux points de terminaison HTTP par défaut.

Plusieurs méthodes de déploiement sont disponibles pour les applications de fonction. Certaines d’entre elles peuvent tirer profit d’un ensemble d’informations d’identification générées. Passez en revue les méthodes de déploiement qui vont être utilisées pour votre application.

- [Sécuriser un point de terminaison HTTP](./functions-bindings-http-webhook-trigger.md?tabs=csharp#secure-an-http-endpoint-in-production)

- [Guide pratique pour obtenir et regénérer les clés d’autorisation](./functions-bindings-http-webhook-trigger.md?tabs=csharp#obtaining-keys)

- [Technologies de déploiement dans Azure Functions](./functions-deployment-technologies.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

En outre, pour vous aider à suivre les comptes d’administration dédiés, vous pouvez utiliser des recommandations d’Azure Security Center ou des stratégies Azure intégrées, telles que les suivantes : Plus d’un propriétaire doit être assigné à votre abonnement. Les comptes déconseillés possédant des autorisations de propriétaire doivent être supprimés de votre abonnement. Les comptes externes possédant des autorisations de propriétaire doivent être supprimés de votre abonnement.

- [Utilisation d’Azure Security Center pour superviser l’identité et l’accès (préversion)](../security-center/security-center-identity-access.md)

- [Utilisation d’Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Aide** : Dans la mesure du possible, utilisez l’authentification unique SSO d’Azure Active Directory au lieu de configurer des informations d’identification autonomes individuelles pour l’accès aux données de votre application de fonction. Suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center. Implémentez l’authentification unique pour vos applications Azure Functions à l’aide de la fonctionnalité d’authentification/autorisation App Service.

- [Présentation de l’authentification et de l’autorisation dans Azure Functions](../app-service/overview-authentication-authorization.md#identity-providers)

- [Présentation de l’authentification SSO avec Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Aide** : Activez l’authentification multifacteur (MFA) Azure Active Directory (AAD) et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils** : Utilisez des stations de travail disposant d’un accès privilégié avec Multi-Factor Authentication (MFA) configuré pour vous connecter aux ressources Azure et les configurer.

- [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration

**Aide** : Utilisez Azure Active Directory (AD) Privileged Identity Management pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.

Utilisez également les détections de risque Azure AD pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs.

- [Déploiement de Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

- [Présentation des détections de risques Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Utilisez des emplacements nommés avec accès conditionnel pour autoriser l’accès au Portail Azure uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

- [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Utilisez Azure Active Directory (AD) en tant que système central d’authentification et d’autorisation pour vos applications Azure Functions. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

- [Guide pratique pour configurer votre application Azure Functions afin d’utiliser la connexion Azure AD](../app-service/configure-authentication-provider-aad.md)

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory (AD) fournit des journaux pour vous aider à découvrir les comptes obsolètes. De plus, utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seules les utilisateurs appropriés continuent de bénéficier d’un accès. 

- [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml)

- [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés

**Aide** : Utilisez Azure Active Directory (AD) en tant que système central d’authentification et d’autorisation pour vos applications Azure Functions. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

Vous avez accès aux activités de connexion Azure AD et aux sources des journaux des événements à risque et des audits, ce qui vous permet de les intégrer à Azure Sentinel ou à un outil SIEM tiers.

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure AD et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes de journal souhaitées dans un espace de travail Log Analytics.

- [Guide pratique pour configurer votre application Azure Functions afin d’utiliser la connexion Azure AD](../app-service/configure-authentication-provider-aad.md)

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Procédure d’intégration d’Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez Azure Active Directory (AD) en tant que système central d’authentification et d’autorisation pour vos applications Azure Functions. Pour les écarts de comportement de connexion au compte sur le plan de contrôle (le portail Azure), utilisez Azure Active Directory (AD) Identity Protection et les fonctionnalités de détection de risques afin de configurer des réponses automatiques aux actions suspectes détectées liées aux identités des utilisateurs. Vous pouvez également ingérer des données dans Azure Sentinel pour approfondir votre examen.

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Aide** : Non disponible pour le moment. Customer Lockbox n’est pas pris en charge pour Azure Functions.

- [Liste des services pris en charge pour Customer Lockbox](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-protection"></a>Protection de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Les applications Azure Functions doivent être séparées par un réseau/sous-réseau virtuel et étiquetées de manière appropriée.

Vous pouvez également utiliser des points de terminaison privés pour effectuer l’isolement réseau. Un point de terminaison privé Azure est une interface réseau qui vous connecte de façon privée et sécurisée à un service (exemple : point de terminaison HTTPS d’application Azure Functions) avec la technologie Azure Private Link. Private Endpoint utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel. Les points de terminaison privés sont en (préversion) pour les applications de fonction qui s’exécutent dans le plan Premium. Vérifiez que les points de terminaison privés ne sont plus en (préversion) avant de les utiliser avec des charges de travail de production.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

- [Options de mise en réseau d’Azure Functions](./functions-networking-options.md)

- [Plan Premium Azure Functions](./functions-scale.md#premium-plan)

- [Présentation du point de terminaison privé](../private-link/private-endpoint-overview.md)

- [Utilisation de points de terminaison privés pour Azure Functions](../app-service/networking/private-endpoint.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Aide** : Déployez un outil automatisé sur les périmètres du réseau qui surveille le transfert non autorisé d’informations sensibles et bloque ces transferts tout en alertant les professionnels de la sécurité des informations. 

Microsoft gère l’infrastructure sous-jacente d’Azure Functions, et a implémenté des contrôles stricts pour empêcher la perte ou la divulgation des données client.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Non applicable

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Dans vos applications Azure Functions, dans le portail Azure, sous « Fonctionnalités de la plateforme :  Réseaux : SSL », activez le paramètre « HTTPS uniquement », puis affectez à la version minimale de TLS la valeur 1.2.

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Aide** : Non disponible pour le moment. Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas disponibles pour Azure Functions. Étiquetez des applications de fonction qui peuvent traiter des informations sensibles en tant que telles et implémenter une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Aide** : utilisez le contrôle d’accès en fonction du rôle Azure (RBAC Azure) pour contrôler l’accès au plan de contrôle d’Azure Functions (portail Azure). 

- [Comment configurer Azure RBAC](../role-based-access-control/role-assignments-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

Microsoft gère l’infrastructure sous-jacente d’Azure Functions, et a implémenté des contrôles stricts pour empêcher la perte ou la divulgation des données client.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide** : Quand vous créez une application de fonction, vous devez créer un compte de stockage Azure à usage général qui prend en charge le stockage Blob, File d’attente et Table, ou établir un lien vers un compte de ce type. Cela est dû au fait que Functions s’appuie sur Stockage Azure pour les opérations telles que la gestion des déclencheurs et la journalisation des exécutions de fonctions. Le stockage Azure chiffre toutes les données dans un compte de stockage au repos. Par défaut, les données sont chiffrées avec des clés managées par Microsoft Afin d’avoir un contrôle supplémentaire sur les clés de chiffrement, vous pouvez fournir des clés gérées par le client à utiliser pour le chiffrement des données de fichiers et d’objets blob. Ces clés doivent être présentes dans Azure Key Vault pour que l’application de fonction puisse accéder au compte de stockage.

- [Présentation des considérations relatives au stockage pour Azure Functions](./storage-considerations.md)

- [Présentation du chiffrement du Stockage Azure pour les données au repos](../storage/common/storage-service-encryption.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes quand des changements sont apportés aux applications Azure Functions de production ainsi qu’à d’autres ressources critiques ou connexes.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : Adoptez une pratique DevSecOps pour avoir la certitude que vos applications Azure Functions sont sécurisées et qu’elles le restent autant que possible tout au long de leur cycle de vie. DevSecOps intègre l’équipe de sécurité de votre organisation et ses capacités dans vos pratiques DevOps, faisant ainsi de la sécurité une responsabilité partagée par tous les membres de l’équipe.

De plus, suivez les recommandations d’Azure Security Center sur la sécurisation de vos applications Azure Functions.

- [Guide pratique pour ajouter une validation de sécurité continue à votre pipeline CI/CD](/azure/devops/migrate/security-validation-cicd-pipeline?view=azure-devops)

- [Implémenter les recommandations d'évaluation des vulnérabilités d'Azure Security Center](../security-center/deploy-vulnerability-assessment-vm.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="52-deploy-an-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution automatisée de gestion des correctifs du système d’exploitation

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3 : Déployer une solution de gestion de correctif logiciel tiers automatisée

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Aide** : Microsoft effectue une gestion des vulnérabilités sur les systèmes sous-jacents qui prennent en charge Azure Functions. Toutefois, vous pouvez utiliser la gravité des recommandations dans Azure Security Center ainsi que le niveau de sécurité pour mesurer les risques dans votre environnement. Votre niveau de sécurité est basé sur le nombre de recommandations Security Center que vous avez appliquées pour atténuer les risques. Pour classer par ordre de priorité les recommandations à résoudre en premier, réfléchissez à la gravité de chacune d’entre elles.

- [Guide de référence des recommandations de sécurité](../security-center/recommendations-reference.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Partagé

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1 : Utiliser la découverte de ressources Azure

**Conseils** : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (telles que calcul, stockage, réseau, ports et protocoles) dans vos abonnements.  Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

- [Guide pratique pour créer des requêtes avec Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Aide** : Utilisez des étiquettes, des groupes d’administration et des abonnements distincts, le cas échéant, pour organiser et suivre les ressources Azure. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

Appliquez également des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant la stratégie Azure avec les définitions intégrées suivantes : Types de ressource non autorisés et types de ressource autorisés

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4 : Tenir un inventaire des ressources Azure approuvées et titres des logiciels

**Aide** : Définissez des ressources Azure approuvées et des logiciels approuvés pour les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements. 

Utilisez Azure Resource Graph pour interroger/découvrir des ressources dans leurs abonnements.  Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées. 

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Aide** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes : Types de ressources non autorisés et Types de ressources autorisés.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-implement-approved-application-list"></a>6.10 : Implémenter une liste d’applications approuvées

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="611-limit-users-ability-to-interact-with-azure-resources-manager-via-scripts"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager par le biais de scripts

**Aide** : Configurez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

- [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Aide** : Pour les applications Azure Functions sensibles ou à haut risque, implémentez des abonnements et/ou des groupes d’administration distincts afin de garantir l’isolation.

Déployez des applications Azure Functions à haut risque dans leur propre réseau virtuel. La sécurité de périmètre dans Azure Functions est obtenue via les réseaux virtuels. Les fonctions qui s’exécutent dans le plan Premium ou l’ASE (App Service Environment) peuvent être intégrées aux réseaux virtuels. Choisissez la meilleure architecture pour votre cas d’usage.

- [Options de mise en réseau d’Azure Functions](./functions-networking-options.md)

- [Plan Premium Azure Functions](./functions-scale.md#premium-plan)

- [Considérations relatives à la mise en réseau pour un environnement App Service](../app-service/environment/network-info.md)

- [Guide pratique pour créer un ASE externe](../app-service/environment/create-external-ase.md)

Guide pratique pour créer un ASE interne :

- [https://docs.microsoft.com/azure/app-service/environment/create-ilb-as](../virtual-network/quick-create-portal.md)

- [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Définissez et implémentez des configurations de sécurité standard pour votre application Azure Functions avec Azure Policy. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.Web » pour créer des stratégies personnalisées d’audit ou d’application de la configuration de vos applications Azure Functions. Vous pouvez également utiliser des définitions de stratégie intégrée, par exemple :
- Une identité managée doit être utilisée dans votre application de fonction
- Le débogage à distance doit être désactivé pour les applications de fonction
- Function App ne doit pas être accessible via HTTPS

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : Utilisez Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Présentation des effets d’Azure Policy](../governance/policy/concepts/effects.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Non applicable. Bien qu’il soit possible de déployer des fonctions locales, cette recommandation concerne les ressources de calcul IaaS. Quand vous déployez des fonctions locales, vous êtes responsable de la configuration sécurisée de votre environnement.

- [Présentation des fonctions locales](./functions-runtime-install.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Stockez et gérez les modèles ARM et les définitions de stratégie Azure personnalisées de manière sécurisée dans le contrôle de code source.

- [Qu’est-ce que l’infrastructure en tant que code](/azure/devops/learn/what-is-infrastructure-as-code)

- [Concevoir des workflows de type Policy as Code](../governance/policy/concepts/policy-as-code.md)

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Documentation Azure Repos](/azure/devops/repos/index?view=azure-devops)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7 : Déployer les outils de gestion de configuration système

**Aide** : Utilisez les définitions Azure Policy intégrées ainsi que les alias Azure Policy dans l’espace de noms « Microsoft.Web » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la surveillance de la configuration automatique pour les services Azure

**Conseils** : Utilisez les définitions Azure Policy intégrées ainsi que les alias Azure Policy dans l’espace de noms « Microsoft.Web » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. Utilisez une stratégie Azure [auditer], [refuser] et [déployer si elle n’existe pas] pour appliquer automatiquement des configurations pour vos ressources Azure.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Aide** : Utilisez les identités managées conjointement avec Azure Key Vault afin de simplifier et sécuriser la gestion des secrets pour vos applications cloud. Les identités managées permettent à votre application de fonction de s’authentifier auprès d’un service qui prend en charge l’authentification Azure AD, notamment Key Vault, sans informations d’identification dans votre code.

- [Créer un coffre de clés](../key-vault/secrets/quick-create-portal.md)

- [Guide pratique pour utiliser des identités managées avec App Service et Azure Functions](../app-service/overview-managed-identity.md)

* [Comment s’authentifier auprès de Key Vault](../key-vault/general/authentication.md)

* [Comment attribuer une stratégie d’accès Key Vault](../key-vault/general/assign-access-policy-portal.md)

- [Utiliser des références Key Vault pour App Service et Azure Functions](../app-service/app-service-key-vault-references.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Aide** : Utilisez les identités managées pour fournir à votre application Azure Functions une identité managée automatiquement dans Azure AD. Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Key Vault, sans informations d’identification dans votre code.

- [Guide pratique pour utiliser des identités managées avec App Service et Azure Functions](../app-service/overview-managed-identity.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault. 

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple Azure Functions), mais il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Aide** : Non applicable. Cette recommandation concerne les ressources qui stockent des données mais qui ne sont pas des ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Aide** : Non applicable. Cette recommandation concerne les ressources qui stockent des données mais qui ne sont pas des ressources de calcul.

Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple Azure Functions), mais il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-recovery"></a>Récupération de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Aide** : Utilisez la fonctionnalité de sauvegarde et de restauration pour planifier des sauvegardes régulières de votre application. Les applications de fonction qui s’exécutent dans le plan Premium ont les mêmes capacités d’hébergement que les applications web dans Azure App Service, ce qui inclut la fonctionnalité Sauvegarde et restauration.

Utilisez également une solution de contrôle de code source telle qu’Azure Repos et Azure DevOps pour stocker et gérer votre code de manière sécurisée. Azure DevOps Services tire profit de nombreuses fonctionnalités du stockage Azure pour garantir la disponibilité des données en cas de défaillance matérielle, d’interruption de service ou de sinistre régional. De plus, l’équipe Azure DevOps suit des procédures pour protéger les données contre toute suppression accidentelle ou malveillante.

- [Sauvegarde de votre application dans Azure](../app-service/manage-backup.md)

- [Présentation de la disponibilité des données dans Azure DevOps](/azure/devops/organizations/security/data-protection?view=azure-devops#data-availability)

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Documentation Azure Repos](/azure/devops/repos/index?view=azure-devops)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Conseils** : Utilisez la fonctionnalité de sauvegarde et de restauration pour planifier des sauvegardes régulières de votre application. Les applications de fonction qui s’exécutent dans le plan Premium ont les mêmes capacités d’hébergement que les applications web dans Azure App Service, ce qui inclut la fonctionnalité Sauvegarde et restauration. Clés de sauvegarde gérées par le client dans Azure Key Vault.

Utilisez également une solution de contrôle de code source telle qu’Azure Repos et Azure DevOps pour stocker et gérer votre code de manière sécurisée. Azure DevOps Services tire profit de nombreuses fonctionnalités du stockage Azure pour garantir la disponibilité des données en cas de défaillance matérielle, d’interruption de service ou de sinistre régional. De plus, l’équipe Azure DevOps suit des procédures pour protéger les données contre toute suppression accidentelle ou malveillante.

- [Sauvegarde de votre application dans Azure](../app-service/manage-backup.md)

- [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

- [Présentation de la disponibilité des données dans Azure DevOps](/azure/devops/organizations/security/data-protection?view=azure-devops#data-availability)

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Documentation Azure Repos](/azure/devops/repos/index?view=azure-devops)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : valider toutes les sauvegardes, y compris les clés gérées par le client

**Aide** : Vérifiez périodiquement que vous pouvez effectuer une restauration à partir de la fonctionnalité de sauvegarde et de restauration. Si vous utilisez un autre emplacement hors connexion pour sauvegarder votre code, vérifiez périodiquement que vous pouvez effectuer des restaurations complètes. Testez la restauration des clés gérées par le client sauvegardées.

- [Restaurer une application dans Azure à partir d’une sauvegarde](../app-service/web-sites-restore.md)

- [Restaurer une application dans Azure à partir d’une capture instantanée](../app-service/app-service-web-restore-snapshots.md)

- [Guide pratique pour restaurer des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Les sauvegardes effectuées à partir de la fonctionnalité de sauvegarde et de restauration utilisent un compte de stockage Azure dans votre abonnement. Le stockage Azure chiffre toutes les données dans un compte de stockage au repos. Par défaut, les données sont chiffrées avec des clés managées par Microsoft Afin d’avoir un contrôle supplémentaire sur les clés de chiffrement, vous pouvez fournir des clés gérées par le client à utiliser pour le chiffrement des données de stockage.

Si vous utilisez des clés gérées par le client, vérifiez que la suppression réversible est activée dans Key Vault pour protéger les clés contre toute suppression accidentelle ou malveillante.

- [Chiffrement du Stockage Azure au repos](../storage/common/storage-service-encryption.md)

- [Guide pratique pour activer la suppression réversible dans Key Vault](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

- [Comment configurer des automatisations de workflow dans Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez clairement les abonnements (par ex. production, non production) et créez un système d’attribution de noms pour identifier et classer les ressources Azure de façon claire.

**Supervision Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

- [Reportez-vous à la publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Conseils** : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé.  Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel.

- [Comment configurer l’exportation continue](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Aide** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » aux alertes et recommandations de sécurité.

- [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de Microsoft pour que vos tests d’intrusion soient conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft.

- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)