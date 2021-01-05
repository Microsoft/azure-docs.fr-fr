---
title: Base de référence de sécurité Azure
description: Base de référence de sécurité Azure pour Azure SQL Database et Azure SQL Managed Instance
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 09/21/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: f9930f42e7d76820674405a98ef53489b91f31f5
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94985686"
---
# <a name="azure-security-baseline-for-azure-sql-database--sql-managed-instance"></a>Base de référence de sécurité Azure pour Azure SQL Database et SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

La ligne de base de sécurité Azure pour Azure SQL Database contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : sécurité réseau](../../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1 : Protéger les ressources à l'aide de groupes de sécurité réseau ou du Pare-feu Azure sur votre réseau virtuel

**Conseils** : Vous pouvez activer Azure Private Link pour autoriser l'accès aux services PaaS Azure, comme SQL Database, ainsi qu’aux services de partenaires ou de clients hébergés par Azure sur un point de terminaison privé de votre réseau virtuel. Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public.

Pour permettre au trafic d'atteindre Azure SQL Database, utilisez les balises de service SQL et autorisez le trafic sortant via les groupes de sécurité réseau.

Les règles de réseau virtuel permettent à Azure SQL Database de n’accepter que les communications provenant de sous-réseaux sélectionnés à l’intérieur d’un réseau virtuel.

Comment configurer Azure Private Link pour Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-private-endpoint-overview#how-to-set-up-private-link-for-azure-sql-database

Comment utiliser des points de terminaison de service de réseau virtuel et des règles pour les serveurs :

https://docs.microsoft.com/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2 : Surveiller et consigner la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Aide** : Utilisez Azure Security Center et corrigez les recommandations de protection réseau pour le sous-réseau sur lequel Azure SQL Database est déployé.

Pour les machines virtuelles Azure qui se connecteront à votre instance Azure SQL Database, activez les journaux de flux du groupe de sécurité réseau (NSG) pour les NSG protégeant ces machines virtuelles, et envoyez les journaux dans un compte Stockage Azure à des fins d'audit du trafic.

Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

Activer les journaux de flux NSG :

https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Comprendre la sécurité réseau fournie par Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-network-recommendations

Activer et utiliser Traffic Analytics :

https://docs.microsoft.com/azure/network-watcher/traffic-analytics

Comprendre la sécurité réseau fournie par Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-network-recommendations

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Non applicable. Cette recommandation a trait à Azure App Service ou aux ressources de calcul hébergeant des applications web.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Conseils** : Activez le Service Protection DDoS Standard sur les réseaux virtuels associés à votre instance Azure SQL Database à des fins de protection contre les attaques par déni de service distribué. Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

Configurer la protection DDoS :

https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection

Comprendre la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-alerts-data-services

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5 : Consigner les paquets réseau et les journaux de flux

**Aide** : Pour les machines virtuelles Azure qui se connecteront à votre instance Azure SQL Database, activez les journaux de flux du groupe de sécurité réseau (NSG) pour les NSG protégeant ces machines virtuelles, et envoyez les journaux dans un compte Stockage Azure à des fins d'audit du trafic. Si cela s'avère nécessaire pour analyser une activité anormale, activez la capture de paquets Network Watcher.

Activer les journaux de flux NSG :

https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Activer Network Watcher :

https://docs.microsoft.com/azure/network-watcher/network-watcher-create

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Aide** : Activez Advanced Threat Protection (ATP) pour Azure SQL Database.  Les utilisateurs reçoivent une alerte en cas d’activités de base de données suspectes, de vulnérabilités potentielles, d’attaques par injection de code SQL et de modèles d’accès et de requêtes anormaux à la base de données. Advanced Threat Protection intègre également des alertes dans Azure Security Center.

Comprendre et utiliser Advanced Threat Protection pour Azure SQL Database : https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Non applicable. Cette recommandation a trait à Azure App Service ou aux ressources de calcul hébergeant des applications web.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : Utilisez des balises de service de réseau virtuel pour définir des contrôles d’accès réseau sur les groupes de sécurité réseau ou le Pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de la balise de service (par exemple, ApiManagement) dans le champ Source ou Destination approprié d'une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Lorsque vous utilisez des points de terminaison de service pour Azure SQL Database, une sortie à destination d'adresses IP publiques Azure SQL Database est requise : Des groupes de sécurité réseau (NSG) doivent être ouverts aux adresses IP Azure SQL Database pour autoriser la connectivité. Pour ce faire, vous pouvez utiliser des balises de service NSG pour Azure SQL Database.

Comprendre les balises de service avec points de terminaison de service pour Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview#limitations

Comprendre et utiliser les balises de service :

https://docs.microsoft.com/azure/virtual-network/service-tags-overview

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Aide** : Définissez et implémentez des configurations de sécurité réseau pour votre instance Azure SQL Database avec Azure Policy. Vous pouvez utiliser l’espace de noms « Microsoft. Sql » pour définir des définitions de stratégie personnalisées, ou utiliser l’une des définitions de stratégie intégrées conçues pour la protection réseau de serveur. Voici un exemple de stratégie de sécurité réseau intégrée applicable pour serveur : « SQL Database doit utiliser un point de terminaison de service de réseau virtuel ».

Utilisez Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les artefacts d’environnement clés, tels que les modèles Resource Manager, le contrôle d’accès en fonction du rôle Azure (Azure RBAC) et les stratégies au sein d’une seule définition de blueprint. Appliquez facilement le blueprint aux nouveaux abonnements et environnements, et ajustez le contrôle et la gestion par le biais du versioning.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Créer une instance Azure Blueprint : https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : Utilisez des balises pour les groupes de sécurité réseau (NSG) et autres ressources liées à la sécurité réseau et au trafic. Concernant les règles NSG individuelles, utilisez le champ « Description » afin de spécifier le besoin métier et/ou la durée (etc.) pour toutes les règles qui autorisent le trafic vers/depuis un réseau.

Utilisez l’une des définitions Azure Policy intégrées en lien avec l’étiquetage, comme « Exiger une étiquette et sa valeur », pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

Créer et utiliser des étiquettes :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Aide** : Utilisez le journal d’activité Azure pour superviser les configurations des ressources réseau et détecter les changements des ressources réseau associées à votre serveur. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.

Comment consulter et récupérer les événements du journal d’activité Azure :

https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view

Comment créer des alertes dans Azure Monitor :

https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Contrôle de sécurité : journalisation et supervision](../../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Conseils** : Microsoft gère les sources de temps pour les ressources Azure. Vous pouvez cependant mettre à jour la synchronisation date/heure pour vos déploiements de calcul.

Comment configurer la synchronisation date/heure pour des ressources de calcul Azure :

https://docs.microsoft.com/azure/virtual-machines/windows/time-sync

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : Activez l’audit pour Azure SQL Database afin de suivre les événements de base de données et les consigner dans un journal d’audit de votre compte Stockage Azure, de l’espace de travail Log Analytics ou d’Event Hubs (préversion).

En outre, vous pouvez diffuser en continu des données de télémétrie de diagnostic Azure SQL dans Azure SQL Analytics, une solution cloud qui supervise à grande échelle les performances d'Azure SQL Database et d'Azure SQL Managed Instance et entre plusieurs abonnements. Elle peut vous aider à collecter et visualiser des métriques de performances Azure SQL Database et intègre des fonctions d’intelligence destinées à résoudre les problèmes.

Comment configurer l’audit pour votre base de données Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-auditing

Comment collecter des journaux et des métriques de plateforme avec Azure Monitor :

https://docs.microsoft.com/azure/sql-database/sql-database-metrics-diag-logging

Comment diffuser en continu des diagnostics dans Azure SQL Analytics :

https://docs.microsoft.com/azure/sql-database/sql-database-metrics-diag-logging#stream-into-azure-sql-analytics

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Aide** : Activez l’audit sur votre serveur et choisissez un emplacement de stockage pour les journaux d’audit (Stockage Azure, Log Analytics ou Event Hub).

Comment activer l’audit pour Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-auditing

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : Non applicable. Ce benchmark a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Lorsque vous stockez vos journaux Azure SQL Database dans un espace de travail Log Analytics, définissez la durée de rétention en fonction des obligations réglementaires de votre organisation.

Définir des paramètres de rétention des journaux :

https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : Analysez et supervisez les journaux pour détecter les comportements anormaux et examinez régulièrement les résultats. Utilisez la fonctionnalité Advanced Threat Protection d'Azure Security Center pour alerter sur les activités inhabituelles liées à votre instance Azure SQL Database. Vous pouvez également configurer des alertes en fonction des valeurs de métrique ou des entrées du journal d’activité Azure en lien avec vos instances Azure SQL Database.

Comprendre Advanced Threat Protection et les alertes pour Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview

Comment configurer des alertes personnalisées pour Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-insights-alerts-portal?view=azps-1.4.0

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer des alertes en cas d’activité anormale

**Conseils** : Utilisez Azure Security Center protection avancée contre les menaces pour Azure SQL Database pour la surveillance et les alertes en cas d’activité anormale. Activez Azure Defender pour SQL pour vos bases de données SQL. Azure Defender pour SQL inclut des fonctionnalités permettant de découvrir et d’atténuer les vulnérabilités de votre base de données, et de détecter les activités anormales susceptibles d’indiquer une menace ciblant votre base de données.

Comprendre Advanced Threat Protection et les alertes pour Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview

Activation d’Azure Defender pour SQL pour vos bases de données SQL :

https://docs.microsoft.com/azure/azure-sql/database/azure-defender-for-sql

Comment gérer les alertes dans Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide** : Non applicable. Pour Azure SQL Database, la solution anti-programme malveillant est gérée par Microsoft sur la plateforme sous-jacente.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Aide** : Non applicable. La journalisation DNS ne s’applique pas à Azure SQL Database.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Aide** : Non applicable. L’audit de ligne de commande ne s’applique pas à Azure SQL Database.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Azure Active Directory (Azure AD) comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent être interrogés. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration.

Comment obtenir un rôle d’annuaire dans Azure AD avec PowerShell :

https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0

Comment obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell :

https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : Azure Active Directory n’intègre pas le concept des mots de passe par défaut. Lors de l’approvisionnement d’une instance Azure SQL Database, il vous est recommandé d’intégrer l’authentification à Azure Active Directory.

Comment configurer et gérer l’authentification Azure Active Directory avec Azure SQL :

https://docs.microsoft.com/azure/azure-sql/database/authentication-aad-configure

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des stratégies et des procédures concernant l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

Comprendre l’identité et l’accès dans Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Aide** : Non applicable. Bien qu'il vous soit possible de configurer l’authentification Azure Active Directory à des fins d'intégration à Azure SQL Database, l’authentification unique n’est pas prise en charge.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Aide** : Activez l’authentification multifacteur (MFA) Azure Active Directory (Azure AD) et suivez les recommandations de gestion des identités et des accès dans Azure Security Center.

Comment activer l’authentification multifacteur dans Azure :

https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Comment surveiller l’identité et l’accès dans Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils** : Utilisez une station de travail disposant d’un accès privilégié avec le service Multi-Factor Authentication (MFA) configuré pour se connecter aux ressources Azure et les configurer.

En savoir plus sur les stations de travail avec accès privilégié :

https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations

Comment activer l’authentification multifacteur dans Azure :

https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration

**Conseils** : Utilisez les rapports de sécurité Azure Active Directory pour générer des journaux et des alertes lorsque des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.

Utilisez Advanced Threat Protection pour Azure SQL Database pour détecter les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles ou potentiellement dangereuses de bases de données.

Comment identifier les utilisateurs Azure AD signalés pour une activité à risque :

https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-user-at-risk

Comment superviser les activités liées aux identités et aux accès des utilisateurs dans Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-identity-access

Examinez Advanced Threat Protection et les alertes potentielles :

https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview#advanced-threat-protection-alerts

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Utilisez des emplacements nommés à accès conditionnel pour autoriser l’accès au portail et à la gestion des ressources Azure uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

Comment configurer des emplacements nommés dans Azure : https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Créez un administrateur Azure Active Directory (Azure AD) pour votre serveur.

Comment configurer et gérer l’authentification Azure Active Directory avec Azure SQL :

https://docs.microsoft.com/azure/azure-sql/database/authentication-aad-configure

Création et configuration d’une instance Azure AD :

https://docs.microsoft.com/azure/active-directory-domain-services/tutorial-create-instance

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory (Azure AD) fournit des journaux pour vous aider à découvrir les comptes obsolètes. De plus, utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. Les accès des utilisateurs peuvent être revus régulièrement pour vérifier que seuls les utilisateurs appropriés continuent de bénéficier d’un accès.

Comment utiliser les révisions d’accès des identités Azure :

https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés

**Aide** : Configurez l’authentification Azure Active Directory (Azure AD) avec Azure SQL et créez des paramètres de diagnostic pour les comptes d’utilisateur Azure Active Directory, en envoyant les journaux d’audit et les journaux de connexion vers un espace de travail Log Analytics. Configurez les alertes souhaitées dans un espace de travail Log Analytics.

Comment configurer et gérer l’authentification Azure Active Directory avec Azure SQL :

https://docs.microsoft.com/azure/azure-sql/database/authentication-aad-configure

Comment intégrer des journaux d’activité Azure dans Azure Monitor :

https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Conseils** : Utilisez Azure Active Directory (Azure AD) Identity Protection et les détections de risques pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. De plus, vous pouvez ingérer des données dans Azure Sentinel pour approfondir votre examen.

Comment afficher les connexions à risque à Azure AD :

https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins

Comment configurer et activer des stratégies de protection des identités :

https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : Dans le cadre des scénarios de support où Microsoft a besoin d’accéder aux données client, Azure Customer Lockbox fournit une interface qui permet aux clients de consulter et d’approuver ou refuser les demandes d’accès aux données client.

Comprendre Customer Lockbox :

https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Contrôle de sécurité : protection des données](../../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

Créer et utiliser des étiquettes :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Les ressources doivent être séparées par un réseau virtuel/sous-réseau, marquées correctement et sécurisées dans un groupe de sécurité réseau ou Pare-feu Azure. Les ressources de stockage ou de traitement des données sensibles doivent être isolées. Utilisez Azure Private Link. Déployez Azure SQL Database dans votre réseau virtuel et connectez-vous en privé à l’aide de points de terminaison privés.

Créer des abonnements Azure supplémentaires :

https://docs.microsoft.com/azure/billing/billing-create-subscription

Créer des groupes d’administration :

https://docs.microsoft.com/azure/governance/management-groups/create

Créer et utiliser des balises :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

Comment configurer Azure Private Link pour Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-private-endpoint-overview#how-to-set-up-private-link-for-azure-sql-database

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Aide** : Pour les bases de données Azure SQL Database qui stockent ou traitent des informations sensibles, marquez la base de données et les ressources associées comme sensibles à l’aide de balises. Configurez Azure Private Link en conjonction avec les balises de service de groupe de sécurité réseau sur vos instances Azure SQL Database pour empêcher l’exfiltration d’informations sensibles.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

Comment configurer Azure Private Link et les groupes de sécurité réseau pour empêcher l’exfiltration de données sur vos instances Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-private-endpoint-overview

Comprendre la protection des données client dans Azure :

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Azure SQL Database sécurise vos données en utilisant le protocole Transport Layer Security pour chiffrer les données en mouvement. SQL Database applique en permanence le chiffrement (SSL/TLS) aux connexions. Ainsi, toutes les données sont chiffrées « en transit » entre le client et le serveur, quel que soit le paramètre Encrypt ou TrustServerCertificate de la chaîne de connexion.

Comprendre le chiffrement Azure SQL en transit :

https://docs.microsoft.com/azure/sql-database/sql-database-security-overview#information-protection-and-encryption

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Conseils** : Utilisez la fonctionnalité de découverte et classification des données Azure SQL Database. Découverte et classification des données offre des fonctionnalités avancées intégrées à Azure SQL Database pour la découverte, la classification, l’étiquetage &amp; protégeant les données sensibles dans vos bases de données.

Comment utiliser la découverte et la classification pour Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-data-discovery-and-classification

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Aide** : Utilisez Azure Active Directory (Azure AD) à des fins d’authentification et de contrôle d’accès aux instances Azure SQL Database.

Comment intégrer Azure SQL Database à Azure Active Directory à des fins d'authentification :

https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication

Comment contrôler l’accès dans Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-control-access

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Conseils** : Microsoft gère l’infrastructure sous-jacente d’Azure SQL Database et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

Comprendre la protection des données client dans Azure :

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide** : Transparent Data Encryption (TDE) est une technologie de chiffrement transparent des données qui contribue à protéger Azure SQL Database, Azure SQL Managed Instance et Azure Data Warehouse contre les menaces d’activités hors connexion malveillantes en chiffrant les données au repos. Il assure le chiffrement et le déchiffrement en temps réel de la base de données, des sauvegardes associées et des fichiers journaux des transactions au repos, sans que cela nécessite de modifier l’application. Par défaut, TDE est activé pour toutes les bases de données nouvellement déployées dans SQL Database et SQL Managed Instance. La clé de chiffrement TDE peut être gérée par Microsoft ou par le client.

Comment gérer le chiffrement transparent des données et utiliser vos propres clés de chiffrement :

https://docs.microsoft.com/azure/sql-database/transparent-data-encryption-azure-sql?tabs=azure-portal#manage-transparent-data-encryption

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes en cas de modifications sur des instances de production Azure SQL Database et autres ressources critiques ou associées.

Comment créer des alertes pour les événements du journal des activités Azure :

https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Aide** : Activez Azure Defender pour SQL pour Azure SQL Database et suivez les recommandations d’Azure Security Center relatives à l’évaluation des vulnérabilités sur vos serveurs.

Procédure d’exécution de l'évaluation des vulnérabilités sur Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-vulnerability-assessment

Activation d’Azure Defender pour SQL :

https://docs.microsoft.com/azure/azure-sql/database/azure-defender-for-sql

Comment implémenter les recommandations d’évaluation de la vulnérabilité d’Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-vulnerability-assessment-recommendations

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3 : Déployer une solution de gestion de correctif logiciel tiers automatisée

**Conseils** : Non applicable. Ce benchmark a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils** : Activez les analyses périodiques pour vos instances Azure SQL Database. Cela permet de configurer une évaluation des vulnérabilités afin d'exécuter automatiquement une analyse de votre base de données une fois par semaine. Un résumé de résultats d’analyse est envoyé aux adresses e-mail que vous fournissez. Comparez les résultats pour vérifier que les vulnérabilités ont été corrigées.

Comment exporter un rapport d’évaluation des vulnérabilités dans Azure Security Center :

https://docs.microsoft.com/azure/sql-database/sql-vulnerability-assessment#implementing-vulnerability-assessment

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : Utilisez les évaluations de risques par défaut (degré de sécurisation) fournies par Azure Security Center.

Comprendre le degré de sécurisation Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-secure-score

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1 : Utiliser la découverte de ressources Azure

**Aide** : Utilisez Azure Resource Graph pour interroger et découvrir toutes les ressources (y compris Azure SQL Database) au sein de vos abonnements.  Vérifiez que vous disposez des autorisations (en lecture) appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

Créer des requêtes avec Azure Resource Graph : https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Afficher vos abonnements Azure : https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0

Comprendre Azure RBAC : https://docs.microsoft.com/azure/role-based-access-control/overview

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

Créer et utiliser des balises :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des étiquettes, des groupes d’administration et diviser des abonnements, le cas échéant, pour organiser et suivre les ressources. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

Créer des abonnements Azure supplémentaires :

https://docs.microsoft.com/azure/billing/billing-create-subscription

Créer des groupes d’administration :

https://docs.microsoft.com/azure/governance/management-groups/create

Créer et utiliser des balises :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4 : Tenir un inventaire des ressources Azure approuvées et titres des logiciels

**Conseils** : Établissez la liste des ressources Azure et des logiciels approuvés pour vos ressources de calcul

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

Utilisez Azure Resource Graph pour interroger/découvrir les ressources dans vos abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Créer des requêtes avec Azure Graph : https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

Utilisez Azure Resource Graph pour interroger/découvrir les ressources dans vos abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Comment refuser un type de ressource spécifique avec Azure Policy : https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-implement-approved-application-list"></a>6.10 : Implémenter une liste d’applications approuvées

**Conseils** : Non applicable. Cette recommandation a trait aux applications s’exécutant sur des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="611-limit-users-ability-to-interact-with-azure-resources-manager-via-scripts"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager par le biais de scripts

**Conseils** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

Configurer l’accès conditionnel pour bloquer l’accès à Azure Resource Manager : https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : Non applicable. Cette recommandation a trait à App Service ou aux ressources de calcul hébergeant des applications web ou de bureau.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Utilisez les recommandations Azure Policy ou Azure Security Center pour Azure SQL Database afin de gérer les configurations de sécurité pour toutes les ressources Azure.

Configurer et gérer Azure Policy :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

Configurer et gérer Azure Policy :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Comprendre les effets d’Azure Policy :

https://docs.microsoft.com/azure/governance/policy/concepts/effects

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous utilisez des définitions Azure Policy personnalisées, utilisez Azure DevOps ou Azure Repos pour stocker et gérer votre code en toute sécurité.

Comment stocker du code dans Azure DevOps :

https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops

Documentation Azure Repos

https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7 : Déployer les outils de gestion de configuration système

**Conseils** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.SQL » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

Configurer et gérer Azure Policy :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la surveillance de la configuration automatique pour les services Azure

**Conseils** : Utilisez Azure Security Center pour effectuer des analyses de ligne de base pour Azure SQL Database.

Comment corriger les recommandations dans Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-sql-service-recommendations

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : Utilisez Azure Key Vault pour stocker les clés de chiffrement pour Azure SQL Database Transparent Data Encryption ( TDE).

Comment protéger les données sensibles stockées dans Azure SQL Database et stocker les clés de chiffrement dans Azure Key Vault :

https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils** : Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure Active Directory (Azure AD). Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Azure Key Vault, sans informations d’identification dans votre code.

Tutoriel : Utiliser une identité managée de machine virtuelle Windows attribuée par le système pour accéder à Azure SQL :

https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-sql

Configuration d’identités managées :

https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Implémentez Credential Scanner pour identifier les informations d’identification dans votre code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

Configuration de Credential Scanner : https://secdevtools.azurewebsites.net/helpcredscan.html

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul. Microsoft gère les logiciels anti-programme malveillant pour la plateforme sous-jacente.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Conseils** : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure App Service), mais il ne s’exécute pas sur du contenu client.

Pré-analysez tout contenu chargé sur des ressources Azure non liées au calcul, comme App Service, Data Lake Storage, Stockage Blob, Azure SQL Database, etc. Microsoft ne peut pas accéder à vos données dans ces instances.

Comprendre Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles : https://docs.microsoft.com/azure/security/fundamentals/antimalware

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul. Microsoft gère les logiciels anti-programme malveillant pour la plateforme sous-jacente.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-recovery"></a>Récupération des données

*Pour plus d’informations, consultez [Contrôle de sécurité : récupération de données](../../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Conseils** : Pour protéger votre entreprise contre la perte de données, Azure SQL Database crée automatiquement des sauvegardes de bases de données complètes (toutes les semaines), des sauvegardes de bases de données différentielles (toutes les 12 heures) et des sauvegardes de fichiers journaux (toutes les 5 à 10 minutes). Les sauvegardes sont stockées dans le stockage RA-GRS pendant au moins 7 jours pour tous les niveaux de service. Tous les niveaux de service, à l’exception du support De base, prennent en charge une période de rétention des sauvegardes configurable pour la restauration dans le temps jusqu’à 35 jours.

Pour répondre aux différentes exigences de conformité, vous pouvez sélectionner plusieurs périodes de rétention pour les sauvegardes hebdomadaires, mensuelles et/ou annuelles. La consommation du stockage dépend de la fréquence sélectionnée des sauvegardes et des périodes de conservation.

Comprendre les sauvegardes et la continuité d’activité avec Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-business-continuity

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Conseils** : Azure SQL Database crée automatiquement des sauvegardes de base de données, qui sont conservées entre 7 et 35 jours, en tirant parti du stockage géoredondant avec accès en lecture seule (RA-GRS) d’Azure pour garantir leur conservation, même en cas d’indisponibilité du centre de données. Ces sauvegardes sont créées automatiquement. Si nécessaire, activez les sauvegardes géoredondantes à long terme pour vos bases de données Azure SQL.

Si vous utilisez des clés gérées par le client pour Transparent Data Encryption, assurez-vous que ces clés sont sauvegardées.

Comprendre les sauvegardes dans Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-automated-backups?tabs=single-database

Comment sauvegarder des clés de coffre de clés dans Azure :

https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : Assurez-vous que la restauration des données du contenu s’effectue régulièrement dans Sauvegarde Azure. Si nécessaire, testez le contenu de restauration vers un réseau local virtuel isolé. Testez la restauration des clés gérées par le client sauvegardées.

Comment restaurer des clés de coffre de clés dans Azure :

https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0

Comment restaurer des sauvegardes Azure SQL Database à l’aide d’une restauration dans le temps :

https://docs.microsoft.com/azure/sql-database/sql-database-recovery-using-backups#point-in-time-restore

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : garantir la protection des sauvegardes et des clés gérées par le client

**Conseils** : Activez la suppression réversible dans Azure Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.

Comment activer la suppression réversible dans Key Vault :

https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Vérifiez qu’il existe des plans de réponse aux incidents écrits qui définissent les rôles du personnel, ainsi que les phases de gestion des incidents.

Comment configurer des automations de flux de travail dans Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center affecte un degré de gravité aux alertes pour vous aider à hiérarchiser l’ordre dans lequel vous remédiez à chaque alerte. Ainsi, quand une ressource est compromise, vous pouvez vous en occuper immédiatement. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

Alertes de sécurité dans Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-alerts-overview

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

Vous pouvez vous reporter à la publication du NIST : Guide de test, de formation et programmes d’exercice pour les plans informatiques et les fonctionnalités :

https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Conseils** : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité afin de vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé.

Comment définir le contact de sécurité Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour transmettre en continu les alertes à Sentinel.

Comment configurer l’exportation continue :

https://docs.microsoft.com/azure/security-center/continuous-export

Comment diffuser des alertes dans Azure Sentinel :

https://docs.microsoft.com/azure/sentinel/connect-azure-security-center

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

Comment configurer l’automatisation du flux de travail et Logic Apps :

https://docs.microsoft.com/azure/security-center/workflow-automation

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : tests d’intrusion et exercices Red Team](../../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1 : Procédez régulièrement à des tests d’intrusion de vos ressources Azure et veillez à résoudre tous les problèmes de sécurité critiques détectés dans un délai de 60 jours

**Conseils** : Respectez les règles d’engagement de Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft :

https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1.

Vous trouverez plus d’informations sur la stratégie de Microsoft et l’exécution de Red Teaming et des tests de pénétration de site actif sur l’infrastructure, les services et les applications cloud managés par Microsoft ici : https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consultez le [benchmark de sécurité Azure](../../security/benchmarks/overview.md)
- Apprenez-en davantage sur les [Bases de référence de la sécurité Azure](../../security/benchmarks/security-baselines-overview.md)