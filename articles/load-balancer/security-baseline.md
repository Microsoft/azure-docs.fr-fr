---
title: Base de sécurité Azure pour Azure Load Balancer
description: La base de référence de sécurité Azure Load Balancer fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le Benchmark de sécurité Azure.
author: msmbaldwin
ms.service: load-balancer
ms.topic: conceptual
ms.date: 09/28/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 07dd369ceeefee85f4d9180a0b9b33a0d4c8bbdb
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94698204"
---
# <a name="azure-security-baseline-for-azure-load-balancer"></a>Base de sécurité Azure pour Azure Load Balancer

La base de référence de sécurité Azure pour Microsoft Azure Load Balancer contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement. La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques. Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Conseils** : Utilisez des instances Azure Load Balancer internes pour autoriser uniquement le trafic vers les ressources principales à partir de certains réseaux virtuels ou réseaux virtuels avec peering sans exposition à Internet. Implémentez un équilibreur de charge externe avec traduction d’adresses réseau source (SNAT) pour maquiller les adresses IP des ressources principales pour la protection contre l’exposition directe à Internet.

Azure propose deux types d’offres d’équilibreur de charge, Standard et De base. Utilisez Standard Load Balancer pour toutes les charges de travail de production. Implémentez des groupes de sécurité réseau et autorisez uniquement l’accès aux plages d’adresses IP et ports approuvés de votre application. Dans les cas où aucun groupe de sécurité réseau n’est affecté au sous-réseau principal ou à la carte réseau des machines virtuelles principales, le trafic n’est pas autorisé à utiliser ces ressources à partir de l’équilibreur de charge. Avec les équilibreurs de charge standard, fournissez des règles de trafic sortant pour définir un NAT sortant avec un groupe de sécurité réseau. Passez en revue les règles de trafic sortant pour régler le comportement de vos connexions sortantes. 

L’utilisation de Standard Load Balancer est recommandée pour vos charges de travail de production. En général, Basic Load Balancer est utilisé uniquement pour les tests, car le type De base est ouvert aux connexions à partir d’Internet par défaut et ne nécessite pas de groupes de sécurité réseau pour fonctionner. 

- [Connexions sortantes dans Azure](load-balancer-outbound-connections.md)

- [Mettre à niveau l’équilibreur de charge public Azure](./upgrade-basic-standard.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Conseils** : L’équilibreur de charge est un service direct, car il s’appuie sur les règles des groupes de sécurité réseau appliquées aux ressources principales et les règles de trafic sortant configurées pour contrôler l’accès à Internet.

Passez en revue les règles de trafic sortant configurées pour votre Standard Load Balancer via le panneau Règles de trafic sortant de votre Load Balancer et le panneau Règles d’équilibrage de charge dans lequel vous pouvez activer des règles de trafic sortant implicites.

Surveillez le nombre de vos connexions sortantes pour suivre la fréquence à laquelle vos ressources s’exposent sur Internet. 

Utilisez Security Center et suivez les recommandations de protection du réseau pour sécuriser vos ressources réseau Azure.

Suivez les recommandations de sécurité pour vos ressources principales et activez les journaux de flux de groupe de sécurité réseau et envoyez ceux-ci à un compte stockage Azure à des fins d’audit.

Envoyez aussi les journaux de flux à un espace de travail Log Analytics, puis utilisez Traffic Analytics pour obtenir des insights sur les modèles de trafic au sein de votre cloud Azure. Les avantages de Traffic Analytics comprennent la possibilité de visualiser l’activité réseau, d’identifier les zones réactives et des menaces de sécurité, de comprendre les modèles de flux de trafic, ainsi que d’épingler de mauvaises configurations du réseau.

- [Tutoriel : journaliser le trafic réseau à destination et en provenance d’une machine virtuelle à l’aide du portail Azure](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Traffic Analytics](../network-watcher/traffic-analytics.md)

- [Protéger vos ressources réseau](../security-center/security-center-network-recommendations.md)

- [Comment faire pour vérifier les statistiques de ma connexion sortante](./load-balancer-standard-diagnostics.md#how-do-i-check-my-outbound-connection-statistics)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Définissez explicitement la connectivité Internet et les adresses IP sources valides via des règles de trafic sortant et des groupes de sécurité réseau avec votre équilibreur de charge pour utiliser les informations sur les menaces de Microsoft pour protéger vos applications web.

- [Intégrer le pare-feu Azure](../firewall/integrate-lb.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Conseils** : Activez la protection Azure Distributed Denial of Service (DDoS) Standard sur les réseaux virtuels Azure pour vous protéger des attaques DDoS. 

Déployez le Pare-feu Azure aux limites réseau de l’organisation en activant le filtrage basé sur le renseignement sur les menaces et en le configurant sur « Alerter et refuser » afin de vous protéger de tout trafic réseau malveillant.

 

Utilisez les fonctionnalités de protection contre les menaces de Security Center pour détecter les communications avec des adresses IP malveillantes connues. 

Le Standard Load Balancer est conçu pour être sécurisé par défaut et fait partie d’un réseau virtuel privé et isolé. Il est fermé aux flux entrants, sauf s’il est ouvert par des groupes de sécurité réseau pour autoriser explicitement le trafic autorisé, et pour interdire les adresses IP malveillantes connues. À moins qu’un groupe de sécurité réseau sur un sous-réseau ou une carte réseau de votre ressource de machine virtuelle n’existe derrière l’équilibreur de charge, le trafic n’est pas autorisé à atteindre cette ressource. 

Déployez le Pare-feu Azure aux limites réseau de l’organisation en activant le filtrage basé sur le renseignement sur les menaces et en le configurant sur « Alerter et refuser » afin de vous protéger de tout trafic réseau malveillant pour empêcher les attaques d’adresses IP malveillantes. 

Implémentez les conseils pour intégrer le pare-feu Azure à votre Load Balancer.

Utilisez les fonctionnalités de protection contre les menaces de Security Center pour détecter les communications avec des adresses IP malveillantes connues. 

Security Center (niveau standard) offre un accès juste-à-temps aux machines virtuelles, et configure les adresses IP sources autorisées afin d’autoriser l’accès uniquement à partir d’adresses IP/plages approuvées.
 

Utilisez la fonctionnalité de sécurisation renforcée du réseau adaptatif Security Center pour les configurations NSG afin de limiter les ports et les adresses IP sources en fonction du trafic réel et du renseignement sur les menaces.
 

- [Gérer la norme Azure DDoS Protection à l’aide du portail Azure](../ddos-protection/manage-ddos-protection.md)

- [Fonctionnalité de filtrage basé sur la Threat Intelligence du Pare-feu Azure](../firewall/threat-intel.md)

- [Protection contre les menaces dans Azure Security Center](../security-center/azure-defender.md)

- [Sécuriser vos ports de gestion avec un accès juste-à-temps](../security-center/security-center-just-in-time.md)

- [Renforcement du réseau adaptatif dans Azure Security Center](../security-center/security-center-adaptive-network-hardening.md)

- [Intégrer le pare-feu Azure à votre équilibreur de charge](../firewall/overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Conseils** : Activez la capture de paquets Network Watcher pour analyser les activités anormales.

- [Créer une instance d’Azure Network Watcher](../network-watcher/network-watcher-create.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions (IDS/IPS) basés sur le réseau

**Conseils** : Implémentez une offre de la Place de marché Azure qui prend en charge les fonctionnalités IDS/IPS avec des fonctionnalités d’inspection de la charge utile dans l’environnement de votre équilibreur de charge. 

Utilisez le pare-feu de renseignement sur les menaces Azure si l’inspection de la charge utile n’est pas obligatoire. Le filtrage basé sur le renseignement sur les menaces du Pare-feu Azure génère des alertes et bloque le trafic en provenance ou à destination d’adresses IP et de domaines malveillants connus. Ces adresses IP et domaines proviennent du flux Microsoft Threat Intelligence.

Déployez la solution de pare-feu de votre choix dans les limites réseau de votre organisation pour détecter et/ou bloquer le trafic malveillant.

- [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall)

- [Guide pratique pour déployer le Pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md)

- [Guide pratique pour configurer des alertes avec le Pare-feu Azure](../firewall/threat-intel.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Définissez explicitement la connectivité Internet et les adresses IP sources valides via des règles de trafic sortant et des groupes de sécurité réseau avec votre équilibreur de charge pour utiliser les informations sur les menaces de Microsoft pour protéger vos applications web.

- [Intégrer le pare-feu Azure](../firewall/integrate-lb.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : Utilisez des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. Spécifiez le nom de l’étiquette de service dans le champ source ou de destination d’une règle pour autoriser ou refuser le trafic pour le service correspondant. 

Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent. 

Par défaut, chaque groupe de sécurité réseau inclut l’étiquette du service AzureLoadBalancer pour autoriser le trafic de la sonde d’intégrité. 

Reportez-vous à la documentation Azure pour toutes les étiquettes de service disponibles pour une utilisation dans les règles de groupe de sécurité réseau.

- [Balises de service disponibles](../virtual-network/service-tags-overview.md#available-service-tags)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Aide** : Définissez et implémentez des configurations de sécurité standard pour les ressources réseau à l'aide d'Azure Policy.

Utilisez Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les principaux artefacts d’environnement (par exemple, les modèles Resource Manager, les contrôles Azure RBAC et les stratégies) dans une même définition de blueprint. 

Appliquez le blueprint aux nouveaux abonnements et affinez le contrôle et la gestion via le contrôle de version.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Exemples Azure Policy pour le réseau](../governance/policy/samples/built-in-policies.md#network)

- [Guide pratique pour créer un blueprint Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : Utilisez des étiquettes de ressource pour les groupes de sécurité réseau (NSG) et autres ressources liées à la sécurité réseau et au trafic. 

Pour les règles de groupe de sécurité réseau, utilisez le champ « Description » pour documenter les règles qui autorisent le trafic vers/à partir d’un réseau.

Implémentez l’une des définitions Azure Policy intégrées en lien avec l’étiquetage, comme « Exiger une étiquette et sa valeur », qui garantissent que toutes les ressources sont créées avec des étiquettes et qui vous informent de l’existence de ressources non étiquetées.

Utilisez Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

- [Comment créer un réseau virtuel Azure](../virtual-network/quick-create-portal.md)

- [Comment filtrer le trafic réseau avec les règles de groupes de sécurité réseau](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : Utilisez le journal d’activité Azure pour effectuer le monitoring des configurations de ressources et détecter les changements apportés à vos ressources Azure. 

Créez des alertes dans Azure Monitor pour vous avertir lorsque des ressources critiques sont modifiées.

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : Passez en revue les modifications apportées à vos règles de trafic sortant et vos groupes de sécurité réseau pour vos équilibreurs de charge en consultant le journal d’activité dans vos abonnements. 

Affichez les journaux des hôtes internes pour vous assurer que vos ressources principales sont sécurisées.

Exportez ces journaux vers Log Analytics ou une autre plateforme de stockage. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, et utilisez des comptes Stockage Azure pour le stockage à long terme et l’archivage.

Activez ces données et intégrez-les à Azure Sentinel ou à une solution SIEM tierce, en fonction des besoins de votre organisation.

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md)

- [Guide pratique pour collecter les journaux des hôtes internes des machines virtuelles Azure avec Azure Monitor](../azure-monitor/learn/quick-collect-azurevm.md)

- [Guide pratique pour bien démarrer avec Azure Monitor et l’intégration d’une solution SIEM tierce](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

- [Journaux d’activité de la plateforme](../azure-monitor/platform/activity-log.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Passez en revue les informations de journalisation et d’audit du plan de gestion et de contrôle capturées avec les journaux d’activité pour l’équilibreur de charge de base. Ces paramètres de capture sont activés par défaut. 

Utilisez les journaux d’activité pour surveiller les actions sur les ressources afin d’afficher l’ensemble de l’activité et des états. 

Déterminez les opérations qui ont été effectuées sur les ressources de votre abonnement avec les journaux d’activité : 

- Qui a démarré l'opération
- Le moment où a eu lieu l’opération
- L’état de l’opération

- Les valeurs d’autres propriétés qui peuvent vous aider à effectuer des recherches sur l’opération

Récupérez les informations du journal d'activité par le biais d'Azure PowerShell, de l'interface de ligne de commande (CLI) Azure, de l'API REST Azure ou du portail Azure. 

Implémentez un diagnostic multidimensionnel pour les fonctionnalités de Standard Load Balancer avec Azure Monitor.  Celles-ci incluent les métriques disponibles pour la sécurité, notamment les informations sur les connexions et ports de traduction d’adresse réseau source (SNAT). Des métriques supplémentaires sur les paquets SYN (synchroniser) et des compteurs de paquets sont également disponibles. 

Récupérez les métriques multidimensionnelles par programmation à l’aide d’API et écrivez-les dans un compte de stockage via l’option « Toutes les métriques ».

Utilisez le pack de contenu des journaux d’audit Azure avec Microsoft Power BI pour analyser vos données avec des tableaux de bord préconfigurés, ou pour personnaliser les affichages en fonction de vos besoins.

Diffusez en continu les journaux vers Event Hub ou un espace de travail Log Analytics. Ils peuvent également être extraits à partir du Stockage Blob Azure et affichés dans différents outils, comme Excel et Power BI. 

Activez les données et intégrez-les à Azure Sentinel ou à une solution SIEM tierce, en fonction de vos besoins.

- [Consultez cet article pour obtenir des instructions pas à pas pour chaque méthode décrite dans les opérations d’audit avec Resource Manager](../azure-resource-manager/management/view-activity-logs.md)

- [Journaux Azure Monitor pour une instance publique de Basic Load Balancer](./load-balancer-monitor-log.md)

- [Afficher les journaux d’activité pour surveiller les actions sur les ressources](../azure-resource-manager/management/view-activity-logs.md)

- [Récupérer les métriques multidimensionnelles par programme via des API](./load-balancer-standard-diagnostics.md#retrieve-multi-dimensional-metrics-programmatically-via-apis)

- [Guide pratique pour bien démarrer avec Azure Monitor et l’intégration d’une solution SIEM tierce](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Le journal d’activité est activé par défaut et est conservé pendant 90 jours dans le magasin des journaux des événements d’Azure. Définissez la période de rétention de votre espace de travail Log Analytics en fonction des réglementations de conformité de votre organisation dans Azure Monitor. Utilisez les comptes de stockage Azure pour le stockage à long terme et l’archivage.

- [Article Afficher les journaux d’activité pour surveiller les actions sur les ressources](../azure-resource-manager/management/view-activity-logs.md)

- [Modification de la période de conservation des données dans Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

- [Guide pratique pour configurer la stratégie de conservation des journaux de compte de Stockage Azure](../storage/common/storage-monitor-storage-account.md#configure-logging)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : Surveillez, gérez et dépannez des ressources Standard Load Balancer sur la page Load Balancer dans le Portail Azure et la page Resource Health sous Azure Monitor. Les métriques disponibles incluent celles pour la sécurité, notamment les informations sur les connexions et ports de traduction d’adresse réseau source (SNAT). Des métriques supplémentaires sur les paquets SYN (synchroniser) et des compteurs de paquets sont également disponibles. 

Utilisez Azure Monitor pour vérifier l’état de la sonde d’intégrité des points de terminaison avec des métriques multidimensionnelles pour les équilibrages de charge standard, externes et internes. Collectez ces métriques par programmation à l’aide d’API et écrites dans un compte de stockage via l’option « Toutes les métriques ».

Les journaux Azure Monitor ne sont pas disponibles pour les instances Load Balancer de base internes. 

Utilisez Azure Monitor pour voir l’état de la sonde d’intégrité résumé par pool principal pour les équilibreurs de charge externes de base, par exemple le nombre d’instances dans votre pool principal qui ne reçoivent pas de requêtes de la part de l’équilibreur de charge en raison d’échecs de la sonde d’intégrité. 

Implémentez la journalisation avec Azure Operational Insights pour fournir des statistiques sur l’état d’intégrité de l’équilibreur de charge. 

Utilisez les journaux d’activité pour afficher des alertes et surveiller les actions sur les ressources et leur état dans vos abonnements Azure. Les journaux d’activité sont activés par défaut et peuvent être affichés dans le portail Azure. 

Utilisez Microsoft Power BI avec le pack de contenu de journaux d’audit Azure et analysez vos données avec des tableaux de bord préconfigurés. Personnalisez les vues en fonction des besoins de votre entreprise. 

Diffusez en continu les journaux vers Event Hub ou un espace de travail Log Analytics. Ils peuvent également être extraits à partir du Stockage Blob Azure et affichés dans différents outils, comme Excel et Power BI. Vous pouvez activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.

- [Sondes d'intégrité Load Balancer](./load-balancer-custom-probe-overview.md)

- [API REST Azure Monitor](/rest/api/monitor)

- [Récupération des métriques via l’API REST](/rest/api/monitor/metrics/list)

- [Diagnostics Azure Standard Load Balancer avec les métriques, les alertes et l’intégrité des ressources](./load-balancer-standard-diagnostics.md)

- [Journaux Azure Monitor pour une instance publique de Basic Load Balancer](./load-balancer-monitor-log.md)

- [Afficher vos métriques d’équilibreur de charge dans le portail Azure](./load-balancer-standard-diagnostics.md#view-your-load-balancer-metrics-in-the-azure-portal)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils** : Utilisez Security Center avec un espace de travail Log Analytics pour superviser les activités anormales liées à l’équilibreur de charge dans les journaux de sécurité et les événements et générer des alertes s’y rapportant.

Activez et intégrez les données dans Azure Sentinel ou une solution SIEM tierce.

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Guide pratique pour gérer les alertes dans Azure Security Center](../security-center/security-center-managing-and-responding-alerts.md)

- [Guide pratique pour générer une alerte sur des données de journal Log Analytics](../azure-monitor/learn/tutorial-response.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Conseils** : Non applicable à Azure Load Balancer. non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Conseils** : Non applicable, car Azure Load Balancer est un service réseau de base qui ne fait pas de requêtes DNS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Conseils** : Non applicable à Azure Load Balancer, car cette recommandation s’applique aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) vous permet de gérer l’accès aux ressources Azure, comme votre équilibreur de charge, par le biais d’attributions de rôles. Attribuez ces rôles aux utilisateurs, groupes, principaux de service et identités managées. 

Stockez les rôles prédéfinis et intégrés pour certaines ressources avec des outils tels qu’Azure CLI, Azure PowerShell ou le Portail Azure.

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="46-use-azure-rbac-to-manage-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : Utilisez Azure RBAC pour contrôler l’accès à vos ressources d’équilibreur de charge.

- [Comment configurer Azure RBAC](../role-based-access-control/role-assignments-portal.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Conseils** : L’équilibreur de charge est un service direct qui ne stocke pas de données client. Il fait partie de la plateforme sous-jacente qui est gérée par Microsoft. 

Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. 

Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes. 

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : Utilisez Azure Monitor avec le journal des activités Azure pour créer des alertes lorsque des modifications sont apportées à des ressources Azure critiques, comme les équilibreurs de charge utilisés pour les charges de travail de production importantes.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : Azure Resource Graph permet d’interroger/de découvrir toutes les ressources (telles que le calcul, le stockage, le réseau, les ports, les protocoles et ainsi de suite) dans vos abonnements. Azure Resource Manager est recommandé pour créer et utiliser les ressources actuelles. 

Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez l’ensemble des abonnements et ressources Azure dans vos abonnements.

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en utilisant des métadonnées pour les organiser de façon logique en suivant une taxonomie.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Aide** : Le cas échéant, utilisez des étiquettes, des groupes d’administration et des abonnements séparés pour organiser et suivre les ressources. 

Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de vos abonnements en temps utile.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4 : Dresser et tenir un inventaire des ressources Azure approuvées

**Conseils** : Créez une liste de ressources Azure approuvées selon les besoins de votre organisation, que vous pouvez exploiter en tant que mécanisme de liste verte. Cela permettra à votre organisation d’intégrer tous les services Azure nouvellement disponibles une fois qu’ils ont été officiellement examinés et approuvés par les processus d’évaluation de sécurité classiques de votre organisation.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Aide** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements.

Interrogez et découvrez des ressources avec Azure Resource Graph au sein d’abonnements détenus. 

Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

- [Guide pratique pour configurer l’accès conditionnel de façon à bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : Les logiciels nécessaires à des opérations métier, mais qui peuvent poser un risque élevé pour l’organisation, doivent être isolés sur leur propre machine virtuelle et/ou réseau virtuel et être suffisamment sécurisés à l’aide d’un Pare-feu Azure ou d’un groupe de sécurité réseau.

- [Guide pratique pour créer un réseau virtuel](../virtual-network/quick-create-portal.md)

- [Comment créer un groupe de sécurité réseau avec une configuration de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Utilisez des alias Azure Policy pour créer des stratégies personnalisées afin d’auditer ou d’appliquer la configuration de vos ressources Azure. Utilisez les définitions Azure Policy intégrées.

Azure Resource Manager est capable d’exporter le modèle au format JSON (JavaScript Object Notation), qui doit être examiné pour vérifier que les configurations répondent aux exigences de sécurité de votre organisation.

Exportez des modèles Azure Resource Manager au format JavaScript Object Notation (JSON), puis examinez-les régulièrement pour vous assurer que les configurations répondent aux exigences de sécurité de votre organisation. 

Implémentez les recommandations de Security Center comme base de référence de configuration sécurisée pour vos ressources Azure. 

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

- [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

- [Exportation monoressource ou multiressource vers un modèle sur le portail Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Recommandations de sécurité - Guide de référence](../security-center/recommendations-reference.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.  En outre, vous pouvez utiliser des modèles Azure Resource Manager pour appliquer la configuration de sécurité des ressources Azure requise par votre organisation. 

- [Comprendre les effets d'Azure Policy](../governance/policy/concepts/effects.md)

- [Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

- [Présentation des modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Conseils** : Utilisez Azure DevOps pour stocker et gérer de manière sécurisée votre code, comme les définitions de stratégies Azure personnalisées, les modèles Azure Resource Manager et les scripts Desired State Configuration. 

Octroyez ou refusez des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps ou dans Active Directory s’ils sont intégrés à TFS.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [À propos des autorisations et des groupes dans Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour les ressources Azure à l’aide d’Azure Policy.  Utilisez des alias Azure Policy pour créer des stratégies personnalisées afin d’auditer ou d’appliquer la configuration réseau de vos ressources Azure. Implémentez des définitions de stratégie intégrées relatives à vos ressources Azure Load Balancer spécifiques.  En outre, utilisez Azure Automation pour déployer des modifications de configuration. sur vos ressources Azure Load Balancer.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Utiliser des alias](../governance/policy/concepts/definition-structure.md#aliases)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Conseils** : Utilisez Security Center pour effectuer des analyses de base pour vos ressources Azure et Azure Policy pour les alertes et l’audit des configurations de ressources.

- [Corriger les recommandations dans Azure Security Center](../security-center/security-center-remediate-recommendations.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. 

La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou à l’analytique utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

Marquez les abonnements à l’aide d’étiquettes et créez un système de nommage pour identifier et classer les ressources Azure, en particulier celles qui traitent des données sensibles.  

Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

- [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md)

- [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations Security Center en utilisant la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure. 

Utilisez la fonctionnalité d’exportation continue de Security Center, qui vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. 

Utilisez le connecteur de données Security Center pour diffuser en continu les alertes vers Azure Sentinel.

- [Comment configurer l’exportation continue](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation de workflow de Security Center pour déclencher automatiquement des réponses aux alertes et aux recommandations de sécurité afin de protéger vos ressources Azure.

- [Configurer l’automatisation du workflow dans Security Center](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de pénétration du cloud Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft. 

- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)