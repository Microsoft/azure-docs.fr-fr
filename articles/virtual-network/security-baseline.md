---
title: Ligne de base de sécurité Azure pour les réseaux virtuels
description: La base de référence de sécurité des réseaux virtuels Azure fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans Azure Security Benchmark.
author: msmbaldwin
ms.service: virtual-network
ms.topic: conceptual
ms.date: 10/01/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 36be61fd65db7ea02a3baec4b519a13231c420ec
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92514439"
---
# <a name="azure-security-baseline-for-virtual-network"></a>Ligne de base de sécurité Azure pour les réseaux virtuels

Cette base de référence de sécurité applique les conseils d’[ version 1.0](../security/benchmarks/overview-v1.md) aux réseaux virtuels Azure. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Le contenu est regroupé par les **contrôles de sécurité** définis par Azure Security Benchmark et les conseils associés applicables aux réseaux virtuels Azure. Les **Contrôles** non applicables au réseau virtuel Azure ont été exclus.

Pour voir comment le réseau virtuel Azure est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité Réseau virtuel Microsoft Azure](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des interfaces réseau

**Conseils**  : Utilisez Security Center et suivez les recommandations de protection du réseau pour sécuriser vos ressources réseau dans Azure. 

Envoyez ces journaux dans un espace de travail Log Analytics et utiliser Traffic Analytics pour obtenir des insights sur le flux de trafic dans votre cloud Azure. Traffic Analytics offre la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau. 

Utilisez les journaux Azure Monitor pour fournir des informations sur votre environnement. Un espace de travail doit être utilisé pour assembler et analyser les données, et peut s’intégrer à d’autres services Azure tels qu’Application Insights et Security Center. 

Choisissez les journaux de ressources à envoyer à un compte de stockage Azure ou à Event Hub. Une autre plateforme peut également être utilisée pour analyser les journaux. 

- [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Guide pratique pour activer et utiliser Traffic Analytics](../network-watcher/traffic-analytics.md)

- [Présentation de la sécurité réseau assurée par Security Center](../security-center/security-center-network-recommendations.md)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications avec des adresses IP connues comme étant malveillantes

**Conseils**  : Activez la protection DDoS (Distributed Denial of Service) Standard sur les réseaux virtuels Azure pour vous protéger des attaques DDoS.

Déployez le Pare-feu Azure aux limites réseau de l’organisation en activant le filtrage basé sur le renseignement sur les menaces et en le configurant sur « Alerter et refuser » afin de vous protéger de tout trafic réseau malveillant.

Utilisez les fonctionnalités de protection contre les menaces de Security Center pour détecter les communications avec des adresses IP malveillantes connues.

Appliquez les recommandations de sécurisation renforcée du réseau adaptatif Security Center pour les configurations NSG afin de limiter les ports et les adresses IP sources en fonction du trafic réel et du renseignement sur les menaces. 

- [Gérer la norme Azure DDoS Protection à l’aide du portail Azure](manage-ddos-protection.md)

- [Fonctionnalité de filtrage basé sur la Threat Intelligence du Pare-feu Azure](../firewall/threat-intel.md)

- [Protection contre les menaces dans Security Center](/azure/security-center/threat-protection)

- [Renforcement du réseau adaptatif dans Azure Security Center](../security-center/security-center-adaptive-network-hardening.md)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Conseils**  : Utilisez la capture de paquets de la passerelle VPN en plus des outils de capture de paquets couramment utilisés pour enregistrer les paquets réseau. 

Vous pouvez examiner des solutions NVA ou basées sur agent qui fournissent des fonctionnalités de visibilité du point d’accès terminal (TAP) ou réseau via nos solutions de partenaire Packet Broker disponibles dans les offres de la Place de marché Azure.

- [Configurer les captures de paquets pour les passerelles VPN](../vpn-gateway/packet-capture.md) 

- [Partenaire d’appliance virtuelle réseau](https://azure.microsoft.com/solutions/network-appliances)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions (IDS/IPS) basés sur le réseau

**Conseils**  : Utilisez un pare-feu Azure déployé sur votre réseau virtuel avec le renseignement sur les menaces activé. Utilisez le filtrage basé sur le renseignement sur les menaces du Pare-feu Azure pour générer des alertes ou refuser le trafic en provenance ou à destination d’adresses IP et de domaines malveillants connus. Ces adresses IP et domaines proviennent du flux Microsoft Threat Intelligence. 

Vous pouvez également sélectionner une offre de la place de marché Azure qui prend en charge les fonctionnalités IDS/IPS avec des fonctionnalités d’inspection de charge utile.

Déployez la solution de pare-feu de votre choix dans les limites réseau de votre organisation pour détecter et/ou refuser le trafic malveillant.

- [Guide pratique pour déployer le Pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md)

- [Guide pratique pour configurer des alertes avec le Pare-feu Azure](../firewall/threat-intel.md)

- [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils**  : Utilisez des étiquettes de service de réseau virtuel pour définir des contrôles d’accès réseau sur des groupes de sécurité réseau ou sur le Pare-feu Azure. Les étiquettes de service peuvent être utilisées à la place d’adresses IP spécifiques pendant la création de règles de sécurité. Autorisez ou refusez le trafic pour le service correspondant en spécifiant le nom de la balise de service (par exemple, ApiManagement) dans le champ Source ou Destination approprié d'une règle. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Utilisez des groupes de sécurité d’application pour simplifier les configurations de sécurité complexes. Les groupes de sécurité d’application vous permettent de configurer la sécurité réseau comme le prolongement naturel d’une structure d’application. Cela vous permet de regrouper des machines virtuelles et de définir des stratégies de sécurité réseau basées sur ces groupes.

- [Présentation et utilisation des balises de service](service-tags-overview.md)

- [Présentation et utilisation des groupes de sécurité d’application](/azure/virtual-network/security-overview#application-security-groups)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils**  : Définissez et implémentez des configurations de sécurité standard pour les ressources réseau avec Azure Policy et passez en revue les définitions de stratégie réseau intégrées pour l’implémentation.

Reportez-vous à la stratégie par défaut de Security Center qui contient les recommandations de sécurité disponibles relatives à vos réseaux virtuels.

Utilisez Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les artefacts d’environnement clés, tels que les modèles Azure Resource Manager, les affectations de contrôle d’accès en fonction du rôle Azure (Azure RBAC) et les stratégies au sein d’une seule définition de blueprint. Azure Blueprint peut être appliqué aux nouveaux abonnements pour une gestion et un contrôle précis grâce au contrôle de version. 

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Exemples Azure Policy pour le réseau](../governance/policy/samples/built-in-policies.md#network) 

- [Guide pratique pour créer un blueprint Azure](../governance/blueprints/create-blueprint-portal.md)

- [Activer la surveillance dans Azure Security Center](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Security%20Center/AzureSecurityCenter.json)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils**  : Utilisez des balises pour les groupes de sécurité réseau et autres ressources liées à la sécurité réseau et au trafic. Concernant les règles de groupe de sécurité réseau individuelles, utilisez le champ « Description » afin de spécifier le besoin métier, la durée et autres informations pour toutes les règles qui autorisent le trafic vers/depuis un réseau.
Utilisez l’une des définitions Azure Policy intégrées en lien avec l’étiquetage, comme « Exiger une étiquette et sa valeur », pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Choisissez Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

- [Guide pratique pour créer et utiliser des étiquettes](/azure/azure-resource-manager/resource-group-using-tags)

- [Guide pratique pour créer un réseau virtuel](quick-create-portal.md)

- [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils**  : Utilisez le journal d’activité Azure pour effectuer le monitoring des configurations de ressources et détecter les modifications apportées à votre réseau virtuel. Créez des alertes dans Azure Monitor, qui se déclenchent en cas de modification des ressources critiques.

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](/azure/azure-monitor/platform/activity-log-view)

- [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide**  : Autorisez Azure Monitor à accéder à vos journaux d’audit et d’activité, qui comprennent la source de l’événement, la date, l’utilisateur, l’horodatage, les adresses sources, les adresses de destination et d’autres éléments utiles. 

Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, puis utilisez les comptes de stockage Azure pour le stockage à long terme/d’archivage.
Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce. 

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md) 

- [Affichage et récupération des événements du journal d’activité Azure](/azure/azure-monitor/platform/activity-log-view)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Aide**  : Autorisez Azure Monitor à accéder à vos journaux d’audit et d’activité, qui comprennent la source de l’événement, la date, l’utilisateur, l’horodatage, les adresses sources, les adresses de destination et d’autres éléments utiles.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md) 

- [Affichage et récupération des événements du journal d’activité Azure](/azure/azure-monitor/platform/activity-log-view)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Aide**  : Dans Azure Monitor, définissez la période de conservation de votre espace de travail Log Analytics en fonction des obligations réglementaires de votre organisation. Utilisez des comptes de stockage Azure pour le stockage à long terme et la rétention du stockage des journaux de sécurité.

- [Modification de la période de conservation des données dans Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

- [Guide pratique pour configurer la stratégie de conservation des journaux de compte de Stockage Azure](../storage/common/storage-monitor-storage-account.md#configure-logging)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide**  : Analysez et supervisez les journaux pour détecter les comportements anormaux et examinez régulièrement les résultats. Utilisez un espace de travail Azure Log Analytics d’Azure Monitor pour effectuer des requêtes et des analyses, et servez-vous de comptes de stockage Azure pour le stockage à long terme et l’archivage. 

Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce. 

- [Présentation de l’espace de travail Log Analytics](../azure-monitor/log-query/get-started-portal.md)

- [Guide pratique pour effectuer des requêtes personnalisées dans Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Guide pratique pour bien démarrer avec Azure Monitor et l’intégration d’une solution SIEM tierce](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils**  : Utilisez Security Center avec un espace de travail Log Analytics pour superviser les activités anormales détectées dans les journaux de sécurité et les événements et générer des alertes s’y rapportant.

Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce pour les alertes.

- [Guide pratique pour gérer les alertes dans Security Center](../security-center/security-center-managing-and-responding-alerts.md)

- [Guide pratique pour générer une alerte sur des données de journal Log Analytics](../azure-monitor/learn/tutorial-response.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Conseils**  : Implémentez une solution tierce de journalisation DNS à partir de la Place de marché Azure en fonction des besoins de votre organisation.

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils**  : Utilisez les rôles Administrateur intégrés Azure Active Directory (Azure AD), qui peuvent être attribués et interrogés explicitement. 

Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration.

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils**  : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Security Center pour superviser le nombre de comptes d’administration.

Activez JIT (Juste-à-temps)/JEA (Just-Enough-Access) à l’aide de rôles Azure AD Privileged Identity Management pour les services Microsoft et d’Azure Resource Manager. 

- [En savoir plus sur Privileged Identity Management](/azure/active-directory/privileged-identity-management/)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4 : Utiliser l’authentification unique (SSO) Azure Active Directory

**Conseils**  : Utilisez l’authentification SSO avec Azure Active Directory (Azure AD) plutôt que de configurer des informations d’identification autonomes individuelles par service. Suivez les recommandations liées à la gestion des identités et des accès dans Security Center.

- [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](../active-directory/manage-apps/what-is-single-sign-on.md) 

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils**  : Activez l’authentification multifacteur (MFA) Azure Active Directory (Azure AD) et suivez les recommandations relatives à la gestion des identités et des accès dans le Centre de sécurité.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md) 

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3.6 : Utiliser des stations de travail sécurisées et gérées par Azure pour les tâches administratives

**Conseils**  : Utilisez des stations de travail disposant d’un accès privilégié avec Multi-Factor Authentication (MFA) configuré pour vous connecter et accéder aux ressources réseau Azure.

- [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations) 

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils**  : Utilisez la détection des risques d’Azure Active Directory (Azure AD) pour visualiser les alertes et les rapports sur les comportements à risque des utilisateurs. 

Ingérez les alertes de détection des risques de Security Center dans Azure Monitor et configurez des alertes/notifications personnalisées à l’aide de groupes d’actions.

- [Présentation des détections de risques Security Center (activité suspecte)](/azure/active-directory/reports-monitoring/concept-risk-events) 

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md) 

- [Configuration des groupes d’actions pour générer des alertes et des notifications personnalisées](../azure-monitor/platform/action-groups.md)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils**  : Utilisez des emplacements nommés à accès conditionnel pour autoriser l’accès uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

- [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Conseils**  : Le cas échéant, utilisez Azure Active Directory (Azure AD) comme système pour vos services. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit, mais aussi en salant, en hachant et en stockant de manière sécurisée les informations d’identification des utilisateurs.  

- [Création et configuration d’une instance Azure AD](../active-directory-domain-services/tutorial-create-instance.md)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Conseils**  : Utilisez Azure Active Directory (Azure AD) pour fournir des journaux pour vous aider à découvrir les comptes obsolètes. 

Des révisions d’accès des identités Azure peuvent être effectuées pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs doit être passé en revue régulièrement pour vérifier que seuls les utilisateurs actifs bénéficient d’un accès permanent.

- [Présentation des rapports Azure AD](/azure/active-directory/reports-monitoring/)

- [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Conseils**  : Implémentez l’intégration avec un outil SIEM ou de supervision quelconque basé sur votre accès aux activités de connexion Azure Active Directory (Azure AD) et aux sources des journaux d’événements à risque et d’audit. 

Simplifiez ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure Active Directory et en envoyant les journaux d’audit et les journaux de connexion dans un espace de travail Log Analytics. Toutes les alertes souhaitées peuvent être configurées dans l’espace de travail Log Analytics.

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12 : Alerter en cas d’écart de comportement de connexion à un compte

**Conseils**  : Utilisez les fonctionnalités de protection des identités et contre les risques d’Azure Active Directory (Azure AD) pour configurer des réponses automatisées aux actions suspectes détectées liées aux identités des utilisateurs de votre réseau virtuel. Ingérez des données dans Azure Sentinel pour approfondir vos examens.

- [Guide pratique pour afficher les connexions risquées Azure AD](/azure/active-directory/reports-monitoring/concept-risky-sign-ins)

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Instructions**  : Chiffrer toutes les informations sensibles en transit. Veillez à ce que les clients qui se connectent à vos ressources Azure sur vos réseaux virtuels puissent négocier le protocole TLS 1.2 ou ultérieur. Suivez les recommandations de Security Center relatives au chiffrement au repos et au chiffrement en transit. 

Microsoft propose plusieurs options qui peuvent être utilisées par les clients pour sécuriser les données en transit en interne au sein du réseau Azure, et en externe sur Internet, pour l'utilisateur final. Il s'agit notamment des communications par le biais de réseaux privés virtuels (utilisant le chiffrement IPsec/IKE), de Transport Layer Security (TLS) 1.2 ou version ultérieure (via des composants Azure tels qu'Application Gateway ou Azure Front Door), de protocoles utilisés directement sur les machines virtuelles Azure (comme Windows IPsec ou SMB), etc.

En outre, le « chiffrement par défaut » à l'aide de MACsec (norme IEEE au niveau de la couche de liaison de données) est activé pour tout le trafic Azure qui transite entre les centres de données Azure afin de garantir la confidentialité et l'intégrité des données client.

- [Présentation du chiffrement en transit avec Azure](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Partagé

### <a name="46-use-azure-rbac-to-manage-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources 

**Conseils**  : Utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour gérer l’accès aux données et ressources. Sinon, utilisez des méthodes de contrôle d’accès spécifiques au service. 

Choisissez des rôles intégrés tels que Propriétaire, Contributeur ou Contributeur de réseau et attribuez le rôle à l’étendue appropriée. Par exemple, vous pouvez affecter un sous-ensemble de fonctionnalités de réseau virtuel avec les autorisations spécifiques requises pour les réseaux virtuels à l’un de ces rôles. 

- [Comment configurer Azure RBAC](../role-based-access-control/role-assignments-portal.md)

- [Planifier des réseaux virtuels](virtual-network-vnet-plan-design-arm.md#permissions)

- [Réviser les rôles intégrés Azure](../role-based-access-control/built-in-roles.md#networking)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils**  : Utilisez Azure Monitor avec les journaux des activités Azure pour créer des alertes lorsque des modifications sont apportées à des ressources Azure critiques, comme les réseaux virtuels et les groupes de sécurité réseau.

- [Journalisation des diagnostics pour un groupe de sécurité réseau](virtual-network-nsg-manage-log.md)

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils**  : Utilisez Azure Resource Graph pour interroger et découvrir toutes les ressources réseau de vos abonnements, comme les réseaux privés et les sous-réseaux. Vérifiez que vous disposez des autorisations (en lecture) appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent.

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md) 

- [Guide pratique pour afficher ses abonnements Azure](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0) 

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils**  : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

- [Guide pratique pour créer et utiliser des étiquettes](/azure/azure-resource-manager/resource-group-using-tags)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils**  : Utilisez des balises, des groupes d’administration, voire des abonnements distincts, pour organiser et suivre le réseau virtuel et les ressources associées. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

- [Guide pratique pour créer des abonnements Azure supplémentaires](/azure/billing/billing-create-subscription) 

- [Guide pratique pour créer des groupes d’administration](/azure/governance/management-groups/create) 

- [Guide pratique pour créer et utiliser des étiquettes](/azure/azure-resource-manager/resource-group-using-tags)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Aide**  : Vous devez créer un inventaire des ressources Azure approuvées et des logiciels approuvés pour les ressources de calcul en fonction des besoins de votre organisation.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils**  : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients selon les définitions de stratégies intégrées suivantes :
- Types de ressources non autorisés 

- Types de ressources autorisés 

Interrogez ou découvrez des ressources dans les abonnements avec Azure Resource Graph dans des environnements haute sécurité, comme ceux avec des comptes de stockage Azure. 

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

- [Exemples Azure Policy intégrés pour un réseau virtuel](/azure/virtual-network/policy-samples)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils**  : Empêchez la création ou l’utilisation de ressources avec Azure Policy selon les exigences des stratégies de l’organisation. Implémentez des processus pour supprimer des ressources non autorisées.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils**  : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients selon les définitions de stratégies intégrées suivantes :
- Types de ressources non autorisés 

- Types de ressources autorisés 

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](/azure/governance/policy/samples/not-allowed-resource-types)

- [Exemples Azure Policy intégrés pour un réseau virtuel](/azure/virtual-network/policy-samples)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Conseils**  : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application Gestion Microsoft Azure.

- [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Conseils**  : Utilisez des alias Azure Policy pour créer des stratégies personnalisées afin d’auditer ou d’appliquer la configuration de vos ressources réseau Azure et aussi utiliser les définitions Azure Policy intégrées.

Exportez vos modèles de build avec Azure Resource Manager au format JavaScript Object Notation (JSON) et examinez-les pour vous assurer que les configurations sont conformes ou dépassent les exigences de sécurité de votre organisation.

Implémentez les recommandations de Security Center comme base de référence de configuration sécurisée pour vos ressources Azure.

- [Affichage des alias Azure Policy disponibles](https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

- [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

- [Exemples Azure Policy intégrés pour un réseau virtuel](/azure/virtual-network/policy-samples)

- [Exportation monoressource ou multiressource vers un modèle sur le portail Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Recommandations de sécurité - Guide de référence](../security-center/recommendations-reference.md)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils**  : Utilisez des modèles Resource Manager et des stratégies Azure pour configurer de manière sécurisée les ressources Azure associées au réseau virtuel et aux ressources liées.  Les modèles Azure Resource Manager sont des fichiers JSON (JavaScript Object Notation) utilisés pour déployer des machines virtuelles avec des ressources Azure. Microsoft effectue la maintenance sur les modèles de base.  

Utilisez les effets Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

- [Informations sur la création de modèles Azure Resource Manager](../virtual-machines/windows/ps-template.md) 

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Présentation des effets d’Azure Policy](../governance/policy/concepts/effects.md)

- [Exemples de modèles Azure Resource Manager pour un réseau virtuel](template-samples.md)

- [Exemples Azure Policy intégrés pour un réseau virtuel](/azure/virtual-network/policy-samples)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Conseils**  : Utilisez Azure DevOps pour stocker et gérer de manière sécurisée votre code, comme les stratégies Azure personnalisées, les modèles Resource Manager et les scripts Desired State Configuration. Et ainsi de suite.

Vous devez disposer d’autorisations pour accéder aux ressources que vous souhaitez gérer dans Azure DevOps, comme votre code, vos builds et le suivi du travail. La plupart des autorisations sont accordées par le biais de groupes de sécurité intégrés. Vous pouvez accorder ou refuser des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps ou dans Active Directory s’ils sont intégrés à Team Foundation Server.

- [Stocker du code dans Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops) 

- [À propos des autorisations et des groupes dans Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils**  : Définissez et implémentez des configurations de sécurité standard pour les ressources Azure à l’aide d’Azure Policy. Utilisez les alias Azure Policy pour créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de vos ressources Azure et toutes les définitions de stratégie intégrées associées à des ressources spécifiques. 

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Utiliser des alias](../governance/policy/concepts/definition-structure.md#aliases)

- [Exemples Azure Policy intégrés pour un réseau virtuel](/azure/virtual-network/policy-samples)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Conseils**  : Utilisez Security Center pour effectuer des analyses de base pour votre réseau virtuel Azure et les ressources associées. Utilisez Azure Policy pour alerter et auditer les configurations des ressources Azure.

- [Guide pratique pour Corriger les recommandations dans Security Center](../security-center/security-center-remediate-recommendations.md)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Exemples Azure Policy intégrés pour un réseau virtuel](/azure/virtual-network/policy-samples)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils**  : utilisez l’identité du service géré conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets pour vos ressources Azure hébergées sur un réseau virtuel Azure.

- [Intégration aux identités managées Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md) 

- [Créer un coffre de clés](/azure/key-vault/quick-create-portal) 

- [Fournir une authentification Key Vault avec une identité managée](/azure/key-vault/managed-identity)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils**  : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

## <a name="data-recovery"></a>Récupération de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : Garantir des sauvegardes automatiques régulières

**Conseils**  : Utilisez Azure Resource Manager pour déployer un réseau virtuel et les ressources associées. Azure Resource Manager permet d’exporter des modèles pouvant servir de sauvegardes pour restaurer le réseau virtuel et les ressources associées.  Utilisez Azure Automation pour appeler régulièrement l’API d’exportation de modèles Azure Resource Manager.

- [Vue d’ensemble d’Azure Resource Manager](../azure-resource-manager/management/overview.md)

- [Exemples de modèles Azure Resource Manager pour un réseau virtuel](template-samples.md)

- [Exportation monoressource ou multiressource vers un modèle sur le portail Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Groupes de ressources – Exportation du modèle](/rest/api/resources/resourcegroups/exporttemplate)

- [Présentation d’Azure Automation](../automation/automation-intro.md)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Conseils**  : Utilisez Azure Resource Manager pour déployer un réseau virtuel et les ressources associées. Azure Resource Manager permet d’exporter des modèles pouvant servir de sauvegardes pour restaurer le réseau virtuel et les ressources associées. Utilisez Azure Automation pour appeler régulièrement l’API d’exportation de modèles Azure Resource Manager. Sauvegardez les clés gérées par le client dans Azure Key Vault.

- [Vue d’ensemble d’Azure Resource Manager](../azure-resource-manager/management/overview.md)

- [Exemples de modèles Azure Resource Manager pour un réseau virtuel](template-samples.md)

- [Exportation monoressource ou multiressource vers un modèle sur le portail Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Groupes de ressources – Exportation du modèle](/rest/api/resources/resourcegroups/exporttemplate)

- [Présentation d’Azure Automation](../automation/automation-intro.md)

- [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils**  : Effectuez régulièrement le déploiement de modèles Azure Resource Manager sur un abonnement isolé et testez la restauration des clés gérées par le client sauvegardées.

- [Déploiement de ressources avec des modèles ARM et le Portail Azure](../azure-resource-manager/templates/deploy-portal.md)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés managées par le client

**Aide**  : Utilisez Azure DevOps pour stocker et gérer de manière sécurisée votre code, comme les définitions de stratégies Azure personnalisées et les modèles Azure Resource Manager. 

Accordez ou refusez des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps ou dans Active Directory s’ils sont intégrés à Team Foundation Server.  

Utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour protéger les clés gérées par le client.   

Activez la suppression réversible et la protection contre la purge dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.  

- [Stocker du code dans Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [À propos des autorisations et des groupes dans Azure DevOps](/azure/devops/organizations/security/about-permissions)

- [Guide pratique pour activer la suppression réversible et la protection contre la purge dans Key Vault](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal) 

- [Suppression réversible pour les objets blob de Stockage Azure](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils**  : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.  

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

- [Tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de votre propre plan de réponse aux incidents](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils**  : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

Marquez clairement les abonnements (par exemple : production ou non-production) à l’aide d’étiquettes et créez un système de nommage pour identifier et classer clairement les ressources Azure, en particulier celles qui traitent des données sensibles.  Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

- [Alertes de sécurité dans Security Center](../security-center/security-center-alerts-overview.md)

- [Organisation des ressources Azure à l’aide de catégories](/azure/azure-resource-manager/resource-group-using-tags)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils**  : Effectuez des exercices pour tester les capacités de réponse aux incidents de vos systèmes à intervalles réguliers, afin de protéger vos ressources Azure. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

- [Publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Instructions**  : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

- [Guide pratique pour définir le contact de sécurité de Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils**  : Exportez vos alertes et recommandations Security Center en utilisant la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. 

Vous pouvez également utiliser le connecteur de données Security Center pour diffuser en continu les alertes vers Azure Sentinel.

- [Comment configurer l’exportation continue](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils**  : Utilisez la fonctionnalité d’automatisation de workflow de Security Center pour déclencher automatiquement avec Logic Apps des réponses aux alertes et aux recommandations de sécurité afin de protéger vos ressources Azure.

- [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide**  : Suivez les règles d’engagement de Microsoft pour que vos tests d’intrusion soient conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft.

- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](/azure/security/benchmarks/overview)
- En savoir plus sur les [bases de référence de la sécurité Azure](/azure/security/benchmarks/security-baselines-overview)
