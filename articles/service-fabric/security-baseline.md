---
title: Base de référence de sécurité Azure pour Service Fabric
description: La base de référence de sécurité Service Fabric fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: service-fabric
ms.topic: conceptual
ms.date: 07/22/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 9eab908d2767bcdc9305daa625c70be9b63b9756
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96485544"
---
# <a name="azure-security-baseline-for-service-fabric"></a>Base de référence de sécurité Azure pour Service Fabric

La base de référence de sécurité Service Fabric contient des recommandations qui vous aideront à améliorer l’état de la sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).



## <a name="network-security"></a>Sécurité du réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Conseils** : Assurez-vous qu'un groupe de sécurité réseau a été appliqué à tous les déploiements de sous-réseaux du réseau virtuel avec des contrôles d'accès réseau spécifiques aux ports et sources approuvés de votre application.

* [Déployer Pare-feu Azure à l’aide d’un modèle](../firewall/deploy-template.md)

* [Créez des réseaux périmétriques en utilisant des groupes de sécurité réseau Azure](../security/fundamentals/service-fabric-best-practices.md#use-network-isolation-and-security-with-azure-service-fabric)

* [Comment intégrer votre cluster Azure Service Fabric à un réseau virtuel existant](./service-fabric-patterns-networking.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Aide** : Utilisez Azure Security Center et appliquez les recommandations de protection réseau pour le réseau virtuel, le sous-réseau et le groupe de sécurité réseau utilisés pour sécuriser votre cluster Azure Service Fabric. Activez les journaux de flux de groupe de sécurité réseau (NSG) et envoyez les journaux dans un compte de Stockage Azure à l’audit du trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Azure Log Analytics et utiliser Azure Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Azure Traffic Analytics offre certains avantages, parmi lesquels la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, de détecter les menaces de sécurité, de bien comprendre les modèles de flux de trafic et enfin de repérer les configurations réseau incorrectes.

* [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Comment activer et utiliser Azure Traffic Analytics](../network-watcher/traffic-analytics.md)

* [Présentation de la sécurité réseau assurée par Azure Security Center](../security-center/security-center-network-recommendations.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Aide** : Proposez une passerelle frontale afin de fournir un point d’entrée unique pour les utilisateurs, les appareils ou d’autres applications. La gestion des API Azure s’intègre directement à Service Fabric, ce qui vous permet de sécuriser l’accès aux services principaux, d’empêcher les attaques par déni de service à l’aide de la limitation et de vérifier les clés API, les jetons JWT, les certificats et d’autres informations d’identification.

Déployez un pare-feu d'applications web Azure pour protéger les applications web critiques afin de bénéficier d'un contrôle supplémentaire du trafic entrant. Activez le paramètre de diagnostic du WAF et ingérez les journaux dans un compte de stockage, un hub d'événements ou un espace de travail Log Analytics.

* [Vue d’ensemble d’Azure Service Fabric avec Gestion des API](./service-fabric-api-management-overview.md)

* [Intégrer le service Gestion des API dans un réseau virtuel interne avec Application Gateway](../api-management/api-management-howto-integrate-internal-vnet-appgateway.md)

* [Guide pratique pour déployer Azure WAF](../web-application-firewall/ag/create-waf-policy-ag.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Aide** : Pour obtenir des protections contre les attaques DDoS, activez la protection Azure DDoS standard sur le réseau virtuel sur lequel votre cluster Azure Service Fabric est déployé. Utilisez la fonctionnalité Threat Intelligence intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

* [Guide pratique pour configurer la protection DDoS](../ddos-protection/manage-ddos-protection.md)

* [Présentation de la fonctionnalité Threat Intelligence intégrée à Azure Security Center](../security-center/azure-defender.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Conseils** : Activez les journaux de flux de groupe de sécurité réseau (NSG) pour le groupe de sécurité réseau attaché au sous-réseau utilisé pour protéger votre cluster Azure Service Fabric. Enregistrez les journaux de flux NSG dans un compte de stockage Azure pour générer des enregistrements de flux. Si cela s’avère nécessaire pour analyser une activité anormale, activez la capture de paquets Azure Network Watcher.

* [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Guide pratique pour activer Network Watcher](../network-watcher/network-watcher-create.md)

* [Utiliser l’analyse du trafic pour visualiser les journaux de flux NSG](../network-watcher/traffic-analytics.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Aide** : Sélectionnez une offre de la place de marché Azure qui prend en charge les fonctionnalités IDS/IPS avec des fonctionnalités d’inspection de charge utile. Si la détection et/ou la prévention des intrusions basées sur l’inspection de la charge utile ne font pas partie des exigences, vous pouvez utiliser le Pare-feu Azure avec Threat Intelligence. Le filtrage basé sur le renseignement sur les menaces du Pare-feu Azure peut générer des alertes et refuser le trafic depuis ou vers des adresses IP et des domaines malveillants connus. Ces adresses IP et domaines proviennent du flux Microsoft Threat Intelligence.

Déployez la solution de pare-feu de votre choix dans les limites réseau de votre organisation pour détecter et/ou refuser le trafic malveillant.

* [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall)

* [Guide pratique pour déployer le Pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md)

* [Guide pratique pour configurer des alertes avec le Pare-feu Azure](../firewall/threat-intel.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Aide** : Déployez Azure Application Gateway pour les applications web en activant le protocole HTTPS/SSL pour les certificats approuvés.

* [Guide pratique pour déployer Application Gateway](../application-gateway/quick-create-portal.md)

* [Guide pratique pour configurer Application Gateway de façon à utiliser le protocole HTTPS](../application-gateway/create-ssl-portal.md)

* [Présentation de l’équilibrage de charge de niveau 7 avec les passerelles d’applications web Azure](../application-gateway/overview.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : Utilisez les étiquettes de service du réseau virtuel pour définir les contrôles d’accès réseau sur les groupes de sécurité réseau (NSG) attachés au sous-réseau dans lequel votre cluster Azure Service Fabric est déployé. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de la balise de service (par exemple, ApiManagement) dans le champ Source ou Destination approprié d'une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

* [Balises de service du réseau virtuel](../virtual-network/service-tags-overview.md)

* [Meilleures pratiques de mise en réseau Service Fabric](./service-fabric-best-practices-networking.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour les ressources réseau liées à votre cluster Azure Service Fabric. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.ServiceFabric » et « Microsoft.Network » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de votre cluster Azure Service Fabric.

Vous pouvez également utiliser Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les principaux artefacts d’environnement (par exemple, les modèles Resource Manager, les contrôles Azure RBAC et les stratégies) dans une même définition de blueprint. Appliquez facilement le blueprint aux nouveaux abonnements et environnements, et ajustez le contrôle et la gestion par le biais du versioning.

* [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Guide pratique pour créer un blueprint Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Aide** : Utilisez des étiquettes pour les groupes de sécurité réseau et autres ressources liées à la sécurité réseau et au flux de trafic qui sont associés à votre cluster Azure Service Fabric. Concernant les règles NSG individuelles, utilisez le champ « Description » afin de spécifier le besoin métier et/ou la durée (etc.) pour toutes les règles qui autorisent le trafic vers/depuis un réseau.

Utilisez l’une des définitions de stratégie Azure intégrée en lien avec l’étiquetage comme « Exiger une étiquette et sa valeur » pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Vous pouvez utiliser Azure PowerShell ou l’interface de ligne de commande (CLI) Azure pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

* [Guide pratique pour créer un réseau virtuel](../virtual-network/quick-create-portal.md)

* [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Aide** : Utilisez le journal d’activité Azure pour superviser les configurations des ressources réseau et détecter les changements des ressources réseau associées à vos déploiements Azure Service Fabric. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.

* [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

* [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Contrôle de sécurité : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Conseils** : Microsoft gère les sources de temps pour les composants de cluster Azure Service Fabric. Vous pouvez mettre à jour la synchronisation de l’heure pour vos déploiements de calcul.

* [Comment configurer la synchronisation de l’heure pour les ressources de calcul Azure](../virtual-machines/windows/time-sync.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Vous pouvez intégrer votre cluster Azure Service Fabric à Azure Monitor pour agréger les données de sécurité générées par le cluster. Pour consulter des exemples de problèmes de diagnostic et de solutions Service Fabric.

* [Configurer l’intégration des journaux Azure Monitor avec Service Fabric](./service-fabric-diagnostics-oms-setup.md)

* [Paramétrer des journaux Azure Monitor pour superviser les conteneurs dans Azure Service Fabric](./service-fabric-tutorial-monitoring-wincontainers.md)

* [Diagnostic des scénarios Service Fabric courants](./service-fabric-diagnostics-common-scenarios.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Aide** : Activez Azure Monitor pour le cluster Service Fabric, puis dirigez-le vers un espace de travail Log Analytics. Cela permet de journaliser des informations de cluster pertinentes et des métriques de système d’exploitation pour tous les nœuds de cluster Azure Service Fabric.

* [Configurer l’intégration des journaux Azure Monitor avec Service Fabric](./service-fabric-diagnostics-oms-setup.md)

* [Paramétrer des journaux Azure Monitor pour superviser les conteneurs dans Azure Service Fabric](./service-fabric-tutorial-monitoring-wincontainers.md)

* [Comment déployer l’agent Log Analytics sur vos nœuds](./service-fabric-diagnostics-oms-agent.md)

* [Recherches de journal Log Analytics](../azure-monitor/log-query/log-query-overview.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Aide** : Intégration du cluster Azure Service Fabric à Azure Monitor. Vérifiez que la période de conservation des journaux définie dans l’espace de travail Log Analytics utilisé est conforme aux obligations réglementaires de votre organisation.

* [Configurer l’intégration des journaux Azure Monitor avec Service Fabric](./service-fabric-diagnostics-oms-setup.md)

* [Paramétrer des journaux Azure Monitor pour superviser les conteneurs dans Azure Service Fabric](./service-fabric-tutorial-monitoring-wincontainers.md)

* [Comment déployer l’agent Log Analytics sur vos nœuds](./service-fabric-diagnostics-oms-agent.md)

* [Comment configurer la période de conservation d’un espace de travail Log Analytics](../azure-monitor/platform/manage-cost-storage.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Aide** : Intégration du cluster Azure Service Fabric à Azure Monitor. Vérifiez que la période de conservation des journaux définie dans l’espace de travail Log Analytics utilisé est conforme aux obligations réglementaires de votre organisation.

* [Configurer l’intégration des journaux Azure Monitor avec Service Fabric](./service-fabric-diagnostics-oms-setup.md)

* [Paramétrer des journaux Azure Monitor pour superviser les conteneurs dans Azure Service Fabric](./service-fabric-tutorial-monitoring-wincontainers.md)

* [Comment déployer l’agent Log Analytics sur vos nœuds](./service-fabric-diagnostics-oms-agent.md)

* [Comment configurer la période de conservation d’un espace de travail Log Analytics](../azure-monitor/platform/manage-cost-storage.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Utilisez des requêtes d’espace de travail Log Analytics pour interroger des journaux Azure Service Fabric.

* [Recherches de journal Log Analytics](../azure-monitor/log-query/log-query-overview.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils** : Utilisez l’espace de travail Azure Log Analytics pour superviser les activités anormales dans les événements et les journaux de sécurité liés à votre cluster Azure Service Fabric, et générer des alertes s’y rapportant.

* [Guide pratique pour gérer les alertes dans Azure Security Center](../security-center/security-center-managing-and-responding-alerts.md)

* [Guide pratique pour générer une alerte sur des données de journal Log Analytics](../azure-monitor/learn/tutorial-response.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide** : Par défaut, Windows Defender est installé sur Windows Server 2016. Si vous n’utilisez pas Windows Defender, reportez-vous à la documentation de votre logiciel anti-programme malveillant pour les règles de configuration. Windows Defender n’est pas pris en charge sur Linux.

* [Pour plus d’informations, consultez Antivirus Windows Defender sur Windows Server 2016](/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-on-windows-server-2016)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Aide** : Implémentez une solution tierce pour la journalisation DNS.

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Instructions** : Configurez manuellement la journalisation de console au niveau de chaque nœud.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Tenez à jour l’enregistrement du compte d’administration local créé lors du provisionnement du cluster Azure Service Fabric ainsi que des autres comptes que vous créez. De plus, si l’intégration Azure AD est utilisée, Azure AD comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent donc être interrogés. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes qui sont membres de groupes d’administration.

Vous pouvez également suivre les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

* [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

* [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

* [Comment superviser les identités et les accès avec Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Aide** : Lors du provisionnement d’un cluster, Azure vous oblige à créer des mots de passe pour l’accès au portail web. Il n’y a pas de mot de passe par défaut à changer. Toutefois, vous pouvez spécifier des mots de passe différents pour l’accès au portail web.

* [Créer dans le portail Azure](./service-fabric-cluster-creation-via-portal.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Aide** : Intégrez l’authentification pour Azure Service Fabric avec Azure Active Directory. Créez des stratégies et des procédures concernant l’utilisation de comptes d’administration dédiés.

Vous pouvez également suivre les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

* [Configurer l’authentification client Azure Active Directory](./service-fabric-tutorial-create-vnet-and-windows-cluster.md#set-up-azure-active-directory-client-authentication)

* [Comment superviser les identités et les accès avec Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : Dans la mesure du possible, utilisez l’authentification unique (SSO) d’Azure Active Directory au lieu de configurer des informations d’identification autonomes individuelles par service. Suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

* [Présentation de l’authentification SSO avec Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Instructions** : Activez Azure AD MFA et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

* [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

* [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Aide** : Utilisez des stations de travail à accès privilégié avec l’authentification multifacteur (MFA) configurée pour la connexion à vos clusters Azure Service Fabric et aux ressources qui leur sont associées, ainsi que pour leur configuration.

* [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Aide** : Utilisez Azure Active Directory (AD) Privileged Identity Management pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement. Utilisez également les détections de risque Azure AD pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs.

* [Déploiement de Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

* [Présentation des détections de risques Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Aide** : Utilisez des emplacements nommés à accès conditionnel pour autoriser l’accès uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

* [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Utilisez Azure Active Directory (AAD) comme système central d’authentification et d’autorisation pour sécuriser l’accès aux points de terminaison de gestion des clusters Azure Service Fabric. AAD protège les données à l’aide d’un chiffrement renforcé pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

* [Guide pratique pour créer et configurer une instance AAD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

* [Configurer Azure Active Directory pour l'authentification cliente dans Service Fabric](./service-fabric-cluster-creation-setup-aad.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Utilisez l’authentification Azure Active Directory (AAD) avec votre cluster Azure Service Fabric. AAD fournit des journaux pour vous permettre de découvrir les comptes obsolètes. De plus, utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès de l’utilisateur peut être passé en revue régulièrement pour vérifier que seuls les utilisateurs appropriés bénéficient d’un accès permanent.

* [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="311-alert-on-account-login-behavior-deviation"></a>3.11 : Alerte en cas d’écart de comportement de connexion à un compte

**Instructions** : Utilisez la connexion Azure Active Directory (AAD) et les journaux d’audit pour superviser les tentatives d’accès à des comptes désactivés. Ces journaux peuvent être intégrés à n’importe quel outil SIEM/de supervision tiers.

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur AAD, et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Azure Log Analytics. Configurez les alertes souhaitées dans un espace de travail Azure Log Analytics.

* [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12 : Alerter en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez les fonctionnalités d’Azure AD Identity Risk and Identity Protection pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. Vous pouvez également ingérer des données dans Azure Sentinel pour approfondir votre examen.

* [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

* [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

* [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Aide** : Non disponible. Customer Lockbox n’est actuellement pas pris en charge pour Azure Service Fabric.

* [Liste des services pris en charge pour Customer Lockbox](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-protection"></a>Protection de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Aide** : Utilisez des étiquettes sur des ressources associées à vos déploiements de cluster Azure Service Fabric pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Les ressources doivent être séparées par un réseau virtuel/sous-réseau, marquées correctement et sécurisées par un groupe de sécurité réseau ou un pare-feu Azure. Les ressources de stockage ou de traitement des données sensibles doivent être suffisamment isolées. Pour les machines virtuelles qui stockent ou traitent des données sensibles, implémentez la stratégie et les procédures pour les désactiver lorsqu’elles ne sont pas utilisées.

* [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

* [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

* [Guide pratique pour créer un réseau virtuel](../virtual-network/quick-create-portal.md)

* [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

* [Guide pratique pour déployer le Pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md)

* [Configuration des options « Alerter » et « Alerter et refuser » du pare-feu Azure](../firewall/threat-intel.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Aide** : Déployez un outil automatisé sur les périmètres du réseau qui surveille le transfert non autorisé d’informations sensibles et bloque ces transferts tout en alertant les professionnels de la sécurité des informations.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données

robustes.

* [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Instructions** : Chiffrer toutes les informations sensibles en transit. Assurez-vous que les clients qui se connectent à vos ressources Azure peuvent négocier TLS 1.2 ou une version ultérieure.

Suivez les recommandations d’Azure Security Center relatives au chiffrement au repos et au chiffrement en transit, le cas échéant.

* [Présentation du chiffrement en transit avec Azure](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit)

* [Scénarios de sécurité d’un cluster Service Fabric](./service-fabric-cluster-security.md)

* [Guide de résolution des problèmes liés à Service Fabric pour la configuration TLS](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides/blob/master/Security/TLS%20Configuration.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Instructions** : Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour le Stockage Azure ou les ressources de calcul. Implémentez une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

* [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Aide** : Non applicable. Cette recommandation concerne les ressources qui stockent des données mais qui ne sont pas des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Aide** : Pour les clusters Azure Service Fabric qui stockent ou traitent des informations sensibles, marquez le cluster et les ressources associées comme sensibles à l’utilisation d’étiquettes. Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour le Stockage Azure ou les ressources de calcul. Implémentez une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

* [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide** : Utilisez le chiffrement au repos sur toutes les ressources Azure. Microsoft recommande d’autoriser Azure à gérer vos clés de chiffrement, mais il existe une option vous permettant de gérer vos propres clés dans certains cas.

* [Présentation du chiffrement au repos dans Azure](../security/fundamentals/encryption-atrest.md)

* [Guide pratique pour configurer des clés de chiffrement gérées par le client](../storage/common/customer-managed-keys-configure-key-vault.md)

* [Activer le chiffrement de disque pour les nœuds de cluster Azure Service Fabric dans Windows](./service-fabric-enable-azure-disk-encryption-windows.md)

* [Activer le chiffrement de disque pour les nœuds de cluster Azure Service Fabric dans Linux](./service-fabric-enable-azure-disk-encryption-linux.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor avec le journal des activités Azure pour créer des alertes lorsque des modifications sont apportées à des ressources Azure critiques.

* [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Aide** : Exécutez régulièrement les services d’analyse des fautes et de chaos Service Fabric pour simuler les erreurs sur l’ensemble du cluster afin d’évaluer la robustesse et la fiabilité de vos services.

Suivez les recommandations d’Azure Security Center relatives à l’évaluation des vulnérabilités sur vos machines virtuelles et images de conteneur Azure.

Utilisez une solution tierce pour effectuer des évaluations des vulnérabilités sur les périphériques réseau et les applications Web. Lors de l’exécution d’analyses à distance, n’utilisez pas un compte d’administration unique et perpétuel. Envisagez d’implémenter une méthodologie d’approvisionnement JIT pour le compte d’analyse. Les informations d’identification du compte d’analyse doivent être protégées, surveillées et utilisées uniquement pour l’analyse des vulnérabilités.

* [Introduction au service d’analyse des erreurs Service Fabric](./service-fabric-testability-overview.md)

* [Induire un chaos contrôlé dans les clusters Service Fabric](./service-fabric-controlled-chaos.md)

* [Implémenter les recommandations d'évaluation des vulnérabilités d'Azure Security Center](../security-center/deploy-vulnerability-assessment-vm.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : Activez les mises à niveau automatiques de l’image du système d’exploitation sur les groupes de machines virtuelles identiques de votre cluster Azure Service Fabric.

Pour tester les correctifs du système d’exploitation avant de passer en production, vous pouvez également utiliser le déclencheur manuel pour les mises à niveau d’images du système d’exploitation de votre groupe identique. Notez que l’option de déclencheur manuel ne fournit pas de restauration intégrée. Surveillez les correctifs du système d’exploitation à l’aide de la gestion des mises à jour proposée par Azure Automation.

* [Gestion des correctifs pour les nœuds de cluster Service Fabric](./service-fabric-best-practices-infrastructure-as-code.md#azure-virtual-machine-operating-system-automatic-upgrade-configuration)

* [Mises à niveau automatiques d'images de système d'exploitation avec des groupes de machines virtuelles identiques Azure](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md)

* [Comment mettre à jour les machines virtuelles avec le dernier modèle du groupe identique](../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model)

* [Présentation d’Update Management Azure Automation](../automation/update-management/overview.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="53-deploy-automated-patch-management-solution-for-third-party-software-titles"></a>5.3 : Déployer une solution de gestion automatisée des correctifs des logiciels tiers

**Aide** : Activez les mises à niveau automatiques de l’image du système d’exploitation sur les groupes de machines virtuelles identiques de votre cluster Azure Service Fabric. L’application d’orchestration des correctifs est une solution alternative conçue pour les clusters hébergés Service Fabric en dehors d’Azure. L’application d’orchestration des correctifs peut être utilisé avec les clusters Azure, avec une surcharge d’hébergement supplémentaire.

* [Gestion des correctifs pour les nœuds de cluster Service Fabric](./service-fabric-best-practices-infrastructure-as-code.md#azure-virtual-machine-operating-system-automatic-upgrade-configuration)

* [Mises à niveau automatiques d'images de système d'exploitation avec des groupes de machines virtuelles identiques Azure](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md)

* [Comment configurer la planification des correctifs du système d’exploitation pour les clusters Service Fabric](./service-fabric-patch-orchestration-application.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils** : Exportez les résultats de l’analyse à intervalles réguliers et comparez les résultats pour vérifier que les vulnérabilités ont été corrigées. Lorsque vous utilisez les recommandations de gestion des vulnérabilités proposées par Azure Security Center, vous pouvez faire pivoter le portail de la solution sélectionnée pour afficher les données d’analyse historiques.

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Instructions** : Utilisez un programme de scoring des risques courants (par exemple, Common Vulnerability Scoring System) ou bien les évaluations des risques par défaut fournies par votre outil d’analyse tiers.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Aide** : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (telles que calcul, stockage, réseau, ports et protocoles) dans vos abonnements. Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

* [Guide pratique pour créer des requêtes avec Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

* [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

* [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des étiquettes, des groupes d’administration et diviser des abonnements, le cas échéant, pour organiser et suivre les ressources. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

* [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

* [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Aide** : Définissez des ressources Azure approuvées et des logiciels approuvés pour les ressources de calcul.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :

Types de ressources non autorisés

Types de ressources autorisés

Utilisez Azure Resource Graph pour interroger/découvrir les ressources dans vos abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Instructions** : Implémentez une solution tierce afin de superviser les nœuds de cluster pour les applications logicielles non approuvées.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Aide** : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (comme le calcul, le stockage, le réseau, les ports, les protocoles, etc.), dont les clusters Azure Service Fabric, dans vos abonnements. Supprimez toutes les ressources Azure non approuvées que vous découvrez. Concernant les nœuds de cluster Azure Service Fabric, implémentez une solution tierce pour générer des alertes sur les logiciels non approuvés ou les supprimer.

* [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Aide** : Pour les nœuds de cluster Azure Service Fabric, implémentez une solution tierce qui empêche l’exécution de logiciels non autorisés.

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Aide** : Utilisez Azure Policy pour limiter les services que vous pouvez approvisionner dans votre environnement.

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés

**Aide** : Pour les nœuds de cluster Azure Service Fabric, implémentez une solution tierce qui empêche l’exécution de types de fichier non autorisés.

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

* [Guide pratique pour configurer l’accès conditionnel de façon à bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : Utilisez des configurations de système d’exploitation spécifiques ou des ressources tierces pour limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul Azure.

* [Par exemple, contrôler l’exécution d’un script PowerShell dans des environnements Windows](/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-6)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Aide** : Les logiciels nécessaires à des opérations métier, mais qui peuvent poser un risque élevé pour l’organisation, doivent être isolés sur leur propre machine virtuelle et/ou réseau virtuel et être suffisamment sécurisés à l’aide d’un Pare-feu Azure ou d’un groupe de sécurité réseau.

* [Guide pratique pour créer un réseau virtuel](../virtual-network/quick-create-portal.md)

* [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.ServiceFabric » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de votre cluster Service Fabric.

* [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Aide** : Les images du système d’exploitation Azure Service Fabric sont gérées et entretenues par Microsoft. Client chargé d’implémenter des configurations sécurisées pour le système d’exploitation de vos nœuds de cluster.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés pour vos clusters Azure Service Fabric et les ressources associées.

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Présentation des effets d’Azure Policy](../governance/policy/concepts/effects.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Aide** : Les images du système d’exploitation du cluster Azure Service Fabric sont gérées et entretenues par Microsoft. Client chargé de l’implémentation de la configuration de l’état au niveau du système d’exploitation.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous utilisez des définitions Azure Policy personnalisées, utilisez Azure DevOps ou Azure Repos pour stocker et gérer votre code en toute sécurité.

* [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

* [Documentation Azure Repos](/azure/devops/repos/index?view=azure-devops)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Aide** : Si vous avez recours à des images personnalisées, utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour garantir que seuls les utilisateurs autorisés ont accès à ces images. Pour les images conteneur, stockez-les dans Azure Container Registry et utilisez Azure RBAC pour vous assurer que seuls les utilisateurs autorisés peuvent accéder aux images.

* [Présentation d’Azure RBAC](../role-based-access-control/rbac-and-directory-admin-roles.md)

* [Comprendre Azure RBAC pour Container Registry](../container-registry/container-registry-roles.md)

* [Comment configurer Azure RBAC](../role-based-access-control/quickstart-assign-role-user-portal.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Aide** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.ServiceFabric » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer des outils de gestion de la configuration pour les systèmes d'exploitation

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Aide** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.ServiceFabric » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration de votre cluster Service Fabric.

* [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Aide** : Utilisez Azure Security Center pour effectuer des analyses de base pour les paramètres de système d’exploitation et d’ancrage des conteneurs.

* [Comprendre les recommandations concernant les conteneurs dans Azure Security Center](../security-center/container-security.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : Utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets pour vos applications Cloud.

* [Utilisation des identités managées pour Azure avec Service Fabric](./concepts-managed-identity.md)

* [Configurer la prise en charge des identités managées pour un nouveau cluster Service Fabric](./configure-new-azure-service-fabric-enable-managed-identity.md)

* [Utiliser l’identité managée avec une application Service Fabric](./how-to-managed-identity-service-fabric-app-code.md)

* [Prise en charge de KeyVaultReference pour les applications Service Fabric](./service-fabric-keyvault-references.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils** : Les identités gérées peuvent être utilisées dans les clusters Service Fabric déployés par Azure et pour les applications déployées en tant que ressources Azure. Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Key Vault, sans informations d’identification dans votre code.

* [Utilisation des identités managées pour Azure avec Service Fabric](./concepts-managed-identity.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Aide** : Si vous utilisez du code lié à votre déploiement Azure Service Fabric, vous pouvez implémenter Credential Scanner pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

Utilisez Azure Key Vault pour faire pivoter automatiquement les certificats de cluster Service Fabric.

* [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

* [Gestion des certificats dans des clusters Service Fabric](./cluster-security-certificate-management.md#certificate-rotation)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Aide** : Par défaut, l’antivirus Windows Defender est installé sur Windows Server 2016. L’interface utilisateur est installée par défaut sur certaines références SKU, mais elle n’est pas obligatoire.

Si vous n’utilisez pas Windows Defender, reportez-vous à la documentation de votre logiciel anti-programme malveillant pour les règles de configuration. Windows Defender n’est pas pris en charge sur Linux.

* [Comprendre l’antivirus Windows Defender sur Windows Server 2016](/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-on-windows-server-2016)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Aide** : Non applicable. Cette recommandation concerne les ressources qui stockent des données mais qui ne sont pas des ressources de calcul. Le logiciel anti-programme malveillant de Microsoft est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Service Fabric), mais ne s’exécute pas sur du contenu client.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Aide** : Non applicable. Cette recommandation concerne les ressources qui stockent des données mais qui ne sont pas des ressources de calcul. Le logiciel anti-programme malveillant de Microsoft est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Service Fabric), mais ne s’exécute pas sur du contenu client.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-recovery"></a>Récupération de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Aide** : Le service de sauvegarde et de restauration de Service Fabric permet de sauvegarder facilement et automatiquement les informations stockées dans les services avec état. Une sauvegarde périodique des données d’application est essentielle pour éviter la perte de données et l’indisponibilité du service. Service Fabric fournit un service de sauvegarde et restauration facultatif, ce qui vous permet de configurer une sauvegarde périodique des services fiables (Reliable Services) avec état (dont les services d’acteur) sans avoir à écrire du code supplémentaire. Il facilite également la restauration des sauvegardes précédemment effectuées.

* [Sauvegarde et restauration périodiques dans un cluster Azure Service Fabric](./service-fabric-backuprestoreservice-quickstart-azurecluster.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Aide** : Activez le service de restauration de sauvegarde dans votre cluster Service Fabric et créez des stratégies de sauvegarde pour sauvegarder régulièrement et à la demande des services avec état. Sauvegardez les clés gérées par le client dans Azure Key Vault.

* [Sauvegarde et restauration périodiques dans un cluster Azure Service Fabric](./service-fabric-backuprestoreservice-quickstart-azurecluster.md)

* [Compréhension de la configuration de la sauvegarde périodique dans Azure Service Fabric](./service-fabric-backuprestoreservice-configure-periodic-backup.md)

* [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Aide** : Vérifiez que vous pouvez effectuer la restauration à partir du service de restauration de sauvegarde en examinant périodiquement les informations de configuration de la sauvegarde et les sauvegardes disponibles. Testez la restauration des clés gérées par le client sauvegardées.

* [Compréhension de la configuration de la sauvegarde périodique dans Azure Service Fabric](./service-fabric-backuprestoreservice-configure-periodic-backup.md)

* [Restauration de la sauvegarde dans Azure Service Fabric](./service-fabric-backup-restore-service-trigger-restore.md)

* [Guide pratique pour restaurer des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Les sauvegardes effectuées à partir du service de restauration de sauvegarde Service Fabric utilisent un compte de stockage Azure dans votre abonnement. Le stockage Azure chiffre toutes les données dans un compte de stockage au repos. Par défaut, les données sont chiffrées avec des clés managées par Microsoft Afin d’avoir un contrôle supplémentaire sur les clés de chiffrement, vous pouvez fournir des clés gérées par le client à utiliser pour le chiffrement des données de stockage.

Si vous utilisez des clés gérées par le client, vérifiez que la suppression réversible est activée dans Key Vault pour protéger les clés contre toute suppression accidentelle ou malveillante.

* [Chiffrement du Stockage Azure au repos](../storage/common/storage-service-encryption.md)

* [Guide pratique pour activer la suppression réversible dans Key Vault](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Aide** : Développez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

* [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

* [Utiliser le guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de votre propre plan de réponse aux incidents](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Aide** : Azure Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez les abonnements à l’aide d’étiquettes et créez un système de nommage pour identifier et classer les ressources Azure, en particulier celles qui traitent des données sensibles. Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

* [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md)

* [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

* [NIST Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test NIST, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Instructions** : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

* [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser la sentinelle des alertes.

* [Comment configurer l’exportation continue](../security-center/continuous-export.md)

* [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

* [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de pénétration du cloud Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft.

* [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

* [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)