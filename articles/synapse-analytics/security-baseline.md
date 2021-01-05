---
title: Base de référence de la sécurité Azure pour Azure Synapse Analytics
description: La base de référence de sécurité Synapse Analytics fournit des instructions et des ressources pour l’implémentation des recommandations de sécurité spécifiées dans Azure Security Benchmark.
author: msmbaldwin
ms.service: synapse-analytics
ms.subservice: security
ms.topic: conceptual
ms.date: 07/22/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 972788e04e12553e7c05f30cc246c6fb562031a4
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461372"
---
# <a name="azure-security-baseline-for-azure-synapse-analytics"></a>Base de référence de la sécurité Azure pour Azure Synapse Analytics

La base de référence de sécurité Azure pour Azure Synapse Analytics contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](https://docs.microsoft.com/azure/security/benchmarks/overview), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](https://docs.microsoft.com/azure/security/benchmarks/security-baselines-overview).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : Sécurité réseau](/azure/security/benchmarks/security-control-network-security).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Aide** : Sécurisez votre serveur Azure SQL sur un réseau virtuel via Azure Private Link. Azure Private Link vous permet d’accéder aux services PaaS Azure sur un point de terminaison privé de votre réseau virtuel. Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft.

Si vous vous connectez à votre pool Synapse SQL, vous pouvez également limiter l’étendue de la connexion sortante à la base de données SQL à l’aide d’un groupe de sécurité réseau. Désactivez tout le trafic des services Azure vers la base de données SQL via le point de terminaison public en paramétrant l’option Autoriser les services Azure sur OFF. Assurez-vous qu’aucune IP publique n’est autorisée dans les règles de pare-feu.

* [Présentation d’Azure Private Link](https://docs.microsoft.com/azure/private-link/private-link-overview)

* [Présentation d’Azure Private Link pour Azure Synapse SQL](https://docs.microsoft.com/azure/sql-database/sql-database-private-endpoint-overview)

* [Guide pratique pour créer un réseau virtuel](https://docs.microsoft.com/azure/virtual-network/quick-create-portal)

* [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des interfaces réseau

**Conseils** : Lorsque vous vous connectez à votre pool Azure Synapse SQL et que vous avez activé les journaux de flux du groupe de sécurité réseau (NSG), les journaux sont envoyés vers un compte de stockage Azure pour l’audit du trafic.

Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

* [Guide pratique pour activer les journaux de flux NSG](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal)

* [Présentation de la sécurité réseau assurée par Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-network-recommendations)

* [Guide pratique pour activer et utiliser Traffic Analytics](https://docs.microsoft.com/azure/network-watcher/traffic-analytics)

* [Présentation de la sécurité réseau assurée par Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-network-recommendations)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Non applicable. Cette recommandation a trait à Azure App Service ou aux ressources de calcul hébergeant des applications web.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications avec des adresses IP connues comme étant malveillantes

**Aide** : Utilisez ATP (Advanced Threat Protection) pour Azure Synapse SQL. ATP détecte les activités anormales indiquant des tentatives inhabituelles et potentiellement dangereuses d’accès aux bases de données ou d’exploitation de ces dernières et peut déclencher différences alertes, telles que « Injection potentielle de code SQL » et « Accès à partir d’un emplacement inhabituel ». ATP fait partie de l’offre Advanced Data Security (ADS) ; elle est accessible et peut être gérée par le biais du portail SQL ADS central.

Activez le service Protection DDoS Standard sur les réseaux virtuels associés à Azure Synapse SQL à des fins de protection contre les attaques par déni de service distribuées. Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

* [Présentation d’ATP pour Azure Synapse SQL](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview)

* [Guide pratique pour activer Advanced Data Security pour Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security)

* [Vue d’ensemble d’ADS](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security)

* [Guide pratique pour configurer la protection DDoS](https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection)

* [Présentation de la fonctionnalité Threat Intelligence intégrée à Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-alerts-data-services)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Conseils** : Lorsque vous vous connectez à votre pool SQL dédié et que vous avez activé les journaux de flux du groupe de sécurité réseau (NSG), envoyez les journaux vers un compte de stockage Azure pour l’audit du trafic. Vous pouvez également envoyer des journaux de flux à un espace de travail Log Analytics ou les diffuser en continu vers Event Hubs. Si cela s'avère nécessaire pour analyser une activité anormale, activez la capture de paquets Network Watcher.

* [Guide pratique pour activer les journaux de flux NSG](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal)

* [Guide pratique pour activer Network Watcher](https://docs.microsoft.com/azure/network-watcher/network-watcher-create)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions (IDS/IPS) basés sur le réseau

**Aide** : Utilisez ATP (Advanced Threat Protection) pour Azure Synapse SQL. ATP détecte les activités anormales indiquant des tentatives inhabituelles et potentiellement dangereuses d’accès aux bases de données ou d’exploitation de ces dernières et peut déclencher différences alertes, telles que « Injection potentielle de code SQL » et « Accès à partir d’un emplacement inhabituel ». ATP fait partie de l’offre Advanced Data Security (ADS) ; elle est accessible et peut être gérée par le biais du portail SQL ADS central. ATP intègre également des alertes dans Azure Security Center.

* [Présentation d’ATP pour Azure Synapse SQL](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Non applicable. Cette recommandation a trait à Azure App Service ou aux ressources de calcul hébergeant des applications web.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Aide** : Utilisez les balises de service du réseau virtuel pour définir des contrôles d’accès réseau sur les groupes de sécurité réseau ou Pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de la balise de service (par exemple, ApiManagement) dans le champ Source ou Destination approprié d'une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Lorsque vous utilisez un point de terminaison de service pour votre pool SQL dédié, une sortie à destination d’adresses IP publiques Azure SQL Database est requise : Des groupes de sécurité réseau (NSG) doivent être ouverts aux adresses IP Azure SQL Database pour autoriser la connectivité. Pour ce faire, vous pouvez utiliser des balises de service NSG pour Azure SQL Database.

* [Comprendre les balises de service avec points de terminaison de service pour Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview#limitations)

* [Comprendre et utiliser les balises de service](https://docs.microsoft.com/azure/virtual-network/service-tags-overview)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : Définissez et implémentez des configurations de sécurité réseau pour les ressources associées à votre pool SQL dédié à l’aide d’Azure Policy. Vous pouvez utiliser l’espace de noms « Microsoft.Sql » pour définir des définitions de stratégie personnalisées ou utiliser l’une des définitions de stratégie intégrées conçues pour la protection réseau de serveur ou la base de données Azure SQL. Voici un exemple de stratégie de sécurité réseau intégrée applicable pour serveur Azure SQL Database : « SQL Server doit utiliser un point de terminaison de service de réseau virtuel ».

Utilisez Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les artefacts d’environnement clés, tels que les modèles Resource Manager, le contrôle d’accès en fonction du rôle Azure (Azure RBAC) et les stratégies au sein d’une seule définition de blueprint. Appliquez facilement le blueprint aux nouveaux abonnements et environnements, et ajustez le contrôle et la gestion par le biais du versioning.

* [Guide pratique pour configurer et gérer Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Guide pratique pour créer un blueprint Azure](https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : Utilisez des balises pour les groupes de sécurité réseau (NSG) et autres ressources liées à la sécurité réseau et au trafic. Concernant les règles NSG individuelles, utilisez le champ « Description » afin de spécifier le besoin métier et/ou la durée (etc.) pour toutes les règles qui autorisent le trafic vers/depuis un réseau.

Utilisez l’une des définitions Azure Policy intégrées en lien avec l’étiquetage, comme « Exiger une étiquette et sa valeur », pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

* [Guide pratique pour créer et utiliser des étiquettes](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : Utilisez le journal d’activité Azure pour surveiller les configurations des ressources réseau et détecter les modifications des ressources réseau associées à votre pool SQL dédié. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.

* [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view)

* [Guide pratique pour créer des alertes dans Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Contrôle de sécurité : Journalisation et supervision](/azure/security/benchmarks/security-control-logging-monitoring).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Conseils** : Microsoft gère les sources de temps pour les ressources Azure. Vous pouvez cependant mettre à jour la synchronisation date/heure pour vos déploiements de calcul.

* [Comment configurer la synchronisation de l’heure pour les ressources de calcul Azure](https://docs.microsoft.com/azure/virtual-machines/windows/time-sync)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Une stratégie d’audit peut être définie pour une base de données spécifique ou en tant que stratégie de serveur par défaut dans Azure (qui héberge Azure Synapse). Une stratégie de serveur s’applique aux bases de données existantes et à celles qui sont nouvellement créées sur le serveur.

Si l’audit du serveur est activé, il s’applique toujours à la base de données. La base de données est auditée, quels que soient les paramètres d’audit de la base de données.

Lorsque vous activez les audits, vous pouvez les écrire dans un journal d’audit dans votre compte de stockage Azure, votre espace de travail Log Analytics ou Event Hubs.

Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.

* [Guide pratique pour configurer l’audit pour vos ressources Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/auditing-overview#server-vs-database-level)

* [Guide pratique pour intégrer Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Activez l’audit au niveau du serveur Azure SQL pour votre pool SQL dédié et choisissez un emplacement de stockage pour les journaux d’audit (stockage Azure, Log Analytics ou Event Hubs).

L’audit peut être activé au niveau de la base de données ou du serveur, mais il est recommandé de l’activer uniquement au niveau du serveur, sauf si vous avez besoin de configurer un récepteur de données distinct ou une conservation des données pour une base de données spécifique.

* [Guide pratique pour activer l’audit pour Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-auditing)

* [Guide pratique pour activer l’audit pour votre serveur](https://docs.microsoft.com/azure/azure-sql/database/auditing-overview#setup-auditing)

* [Différences entre les stratégies d’audit au niveau du serveur et de la base de données](https://docs.microsoft.com/azure/sql-database/sql-database-auditing#server-vs-database-level)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : Non applicable. Ce benchmark a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Lors du stockage des journaux relatifs à votre pool SQL dédié dans un compte de stockage, un espace de travail Log Analytics ou un Event Hub, définissez la période de rétention des journaux conformément aux réglementations de conformité de votre organisation.

* [Gérer le cycle de vie du stockage Blob Azure](https://docs.microsoft.com/azure/storage/blobs/storage-lifecycle-management-concepts?tabs=azure-portal)

* [Définir les paramètres de rétention des journaux dans un espace de travail Log Analytics](https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period)

* [Capturer des événements de streaming dans Event Hubs](https://docs.microsoft.com/azure/event-hubs/event-hubs-capture-overview)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : Analysez et supervisez les journaux pour détecter les comportements anormaux et examinez régulièrement les résultats. Utilisez Advanced Threat Protection pour Azure SQL Database conjointement avec Azure Security Center pour signaler les activités inhabituelles liées à votre base de données SQL. Vous pouvez également configurer des alertes en fonction des valeurs de métrique ou des entrées du journal d’activité Azure en lien avec votre base de données SQL.

Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.

* [Présentation d’Advanced Threat Protection et des alertes pour Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview)

* [Guide pratique pour activer Advanced Data Security pour Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security)

* [Guide pratique pour configurer des alertes personnalisées pour Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-insights-alerts-portal?view=azps-1.4.0&preserve-view=true)

* [Guide pratique pour intégrer Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Aide** : Utilisez Advanced Threat Protection (ATP) pour Azure SQL Database conjointement avec Azure Security Center pour surveiller et signaler les activités anormales. ATP fait partie de l’offre Advanced Data Security (ADS) ; elle est accessible et peut être gérée par le biais du portail SQL ADS central. ADS inclut des fonctionnalités permettant de découvrir et de classifier les données sensibles, de présenter et de corriger les vulnérabilités de votre base de données et de détecter les activités anormales susceptibles d’indiquer une menace ciblant votre base de données.

Vous pouvez également activer et intégrer les données dans Azure Sentinel.

* [Présentation d’Advanced Threat Protection et des alertes pour Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview)

* [Guide pratique pour activer Advanced Data Security pour Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security)

* [Guide pratique pour gérer les alertes dans Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts)

* [Guide pratique pour intégrer Azure Sentinel](https://docs.microsoft.com/azure/sentinel/quickstart-onboard)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Conseils** : Non applicable. Pour les ressources associées à votre pool SQL dédié, la solution anti-programme malveillant est gérée par Microsoft sur la plateforme sous-jacente.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Conseils** : Non applicable. Aucun journal DNS n’est généré par les ressources associées à votre pool SQL dédié.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Aide** : Non applicable. L’audit de ligne de commande ne s’applique pas à Azure Synapse SQL.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](/azure/security/benchmarks/security-control-identity-access-control).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Les utilisateurs sont authentifiés à l’aide d’Azure Active Directory ou de l’authentification SQL.

Lorsque vous déployez Azure SQL pour la première fois, vous spécifiez une connexion d'administrateur et un mot de passe associé à cette connexion. Ce compte d’administration est appelé Administration de serveur. Vous pouvez identifier les comptes d’administrateur d’une base de données en ouvrant le portail Azure et en accédant à l’onglet Propriétés de votre serveur ou instance gérée. Vous pouvez également configurer un compte d’administrateur Azure AD avec des autorisations d’administration complètes, ce qui est nécessaire si vous souhaitez activer l’authentification Azure Active Directory.

Pour les opérations de gestion, utilisez les rôles Azure intégrés qui doivent être attribués explicitement. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration.

* [Authentification pour SQL Database](https://docs.microsoft.com/azure/azure-sql/database/security-overview#authentication)

* [Créer des comptes pour les utilisateurs non-administrateurs](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#create-accounts-for-non-administrator-users)

* [Utiliser un compte Azure Active Directory pour l’authentification](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#create-additional-logins-and-users-having-administrative-permissions)

* [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0&preserve-view=true)

* [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0&preserve-view=true)

* [Guide pratique pour gérer des connexions et comptes administrateur existants dans Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#existing-logins-and-user-accounts-after-creating-a-new-database)

* [Rôles intégrés Azure](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : Azure Active Directory n’intègre pas le concept des mots de passe par défaut. Lors de l’approvisionnement d’un pool SQL dédié, il vous est recommandé d’intégrer l’authentification à Azure Active Directory. Avec cette méthode d'authentification, l'utilisateur soumet un nom de compte d'utilisateur et demande que le service utilise les informations d'identification stockées dans Azure Active Directory (Azure AD).

* [Guide pratique pour configurer et gérer l’authentification Azure Active Directory avec Azure SQL](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure?tabs=azure-powershell#active-directory-password-authentication)

* [Présentation de l’authentification dans Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#existing-logins-and-user-accounts-after-creating-a-new-database)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des stratégies et des procédures concernant l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès d’Azure Security Center pour surveiller le nombre de comptes d’administration qui se connectent via Azure Active Directory.

Pour identifier les comptes administrateur d’une base de données, ouvrez le portail Azure, puis accédez à l’onglet Propriétés de votre serveur ou instance gérée.

* [Présentation de l’identité et de l’accès Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-identity-access)

* [Guide pratique pour gérer des connexions et comptes administrateur existants dans Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage#existing-logins-and-user-accounts-after-creating-a-new-database)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4 : Utiliser l’authentification unique (SSO) Azure Active Directory

**Aide** : Utilisez une inscription d’application Azure (principal du service) afin de récupérer un jeton qui peut être utilisé pour interagir avec votre entrepôt de données au niveau du plan de contrôle (Portail Azure) via des appels d’API.

* [Appels d’API REST Azure](https://docs.microsoft.com/rest/api/azure/#how-to-call-azure-rest-apis-with-postman)

* [Inscription de votre application cliente (principal du service) à l’aide d’Azure AD](https://docs.microsoft.com/rest/api/azure/#register-your-client-application-with-azure-ad)

* [Informations relatives à l’API REST Azure Synapse SQL](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-manage-compute-rest-api)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Aide** : Activez l’authentification multifacteur (MFA) Azure Active Directory (AAD) et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

* [Guide pratique pour activer l’authentification MFA dans Azure](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted)

* [Guide pratique pour superviser les identités et les accès dans Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-identity-access)

* [Présentation de l’authentification multifacteur dans Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/authentication-mfa-ssms-overview)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3.6 : Utiliser des stations de travail sécurisées et gérées par Azure pour les tâches administratives

**Aide** : Utilisez une station de travail disposant d’un accès privilégié avec l’authentification multifacteur (MFA) configurée pour vous connecter aux ressources Azure et les configurer.

* [En savoir plus sur les stations de travail à accès privilégié](https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Guide pratique pour activer l’authentification MFA dans Azure](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils** : Utilisez les rapports de sécurité Azure Active Directory pour générer des journaux et des alertes lorsque des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.

Utilisez Advanced Threat Protection pour Azure SQL Database conjointement avec Azure Security Center pour détecter et signaler les activités anormales indiquant des tentatives inhabituelles et potentiellement dangereuses d’accès aux bases de données ou d’exploitation de ces dernières.

L’audit SQL Server vous permet de créer des audits de serveur, qui peuvent contenir des spécifications d’audit de serveur pour les événements au niveau du serveur et des spécifications d’audit de base de données pour les événements au niveau de la base de données. Les événements audités peuvent être écrits dans les journaux d’événements ou les fichiers d’audit.

* [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-user-at-risk)

* [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-identity-access)

* [Examen d’Advanced Threat Protection et des alertes potentielles](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-overview#advanced-threat-protection-alerts)

* [Présentation des connexions et des comptes d’utilisateur dans Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage)

* [Présentation de l’audit SQL Server](https://docs.microsoft.com/sql/relational-databases/security/auditing/sql-server-audit-database-engine?view=sql-server-ver15&preserve-view=true)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Utilisez des emplacements nommés à accès conditionnel pour autoriser l’accès au portail et à la gestion des ressources Azure uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

* [Guide pratique pour configurer des emplacements nommés dans Azure](https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Conseils** : Créez un administrateur Azure Active Directory (AD) pour le serveur Azure SQL Database dans votre pool SQL dédié.

* [Guide pratique pour configurer et gérer l’authentification Azure AD avec Azure SQL](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure)

* [Création et configuration d’une instance Azure AD](https://docs.microsoft.com/azure/active-directory-domain-services/tutorial-create-instance)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Conseils** : Azure Active Directory fournit des journaux pour vous aider à découvrir les comptes obsolètes. Par ailleurs, utilisez les révisions d’accès des identités Azure Active Directory pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. Les accès des utilisateurs peuvent être revus régulièrement pour vérifier que seuls les utilisateurs appropriés continuent de bénéficier d’un accès.

Lorsque vous utilisez l’authentification SQL, créez des utilisateurs de base de données autonome dans la base de données. Veillez à placer un ou plusieurs utilisateurs de base de données dans un rôle de base de données personnalisé avec les autorisations spécifiques appropriées pour ce groupe d’utilisateurs.

* [Guide pratique pour utiliser les révisions d’accès](https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview)

* [Présentation des connexions et des comptes d’utilisateur dans Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Aide** : Configurez l’authentification Azure Active Directory (AD) avec Azure SQL et activez les paramètres de diagnostic pour les comptes d’utilisateur Azure Active Directory, ce qui envoie les journaux d’audit et les journaux de connexion vers un espace de travail Log Analytics. Configurez les alertes souhaitées dans Log Analytics.

Lorsque vous utilisez l’authentification SQL, créez des utilisateurs de base de données autonome dans la base de données. Veillez à placer un ou plusieurs utilisateurs de base de données dans un rôle de base de données personnalisé avec les autorisations spécifiques appropriées pour ce groupe d’utilisateurs.

* [Guide pratique pour utiliser les révisions d’accès](https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview)

* [Guide pratique pour configurer et gérer l’authentification Azure AD avec Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure)

* [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics)

* [Présentation des connexions et des comptes d’utilisateur dans Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12 : Alerter en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez les fonctionnalités de détection de risque et de protection des identités d’Azure Active Directory (Azure AD) pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. De plus, vous pouvez intégrer et ingérer des données dans Azure Sentinel pour une enquête plus approfondie.

Lorsque vous utilisez l’authentification SQL, créez des utilisateurs de base de données autonome dans la base de données. Veillez à placer un ou plusieurs utilisateurs de base de données dans un rôle de base de données personnalisé avec les autorisations spécifiques appropriées pour ce groupe d’utilisateurs.

* [Guide pratique pour afficher les connexions à risque à Azure AD](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins)

* [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies)

* [Procédure d’intégration d’Azure Sentinel](https://docs.microsoft.com/azure/sentinel/connect-data-sources)

* [Présentation des connexions et des comptes d’utilisateur dans Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : Dans le cadre des scénarios de support où Microsoft a besoin d’accéder aux données relatives à la base de données Azure SQL dans votre pool SQL dédié, Azure Customer Lockbox fournit une interface qui vous permet de consulter et d’approuver ou refuser les demandes d’accès aux données.

* [Présentation de Customer Lockbox](https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Protection des données](/azure/security/benchmarks/security-control-data-protection).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

Découverte et classification de données est intégrée à Azure Synapse SQL. Elle offre des fonctionnalités avancées pour la découverte, la classification, l’étiquetage et le signalement des données sensibles dans vos bases de données.

* [Guide pratique pour créer et utiliser des étiquettes](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

* [Présentation de Découverte et classification des données](https://docs.microsoft.com/azure/azure-sql/database/data-discovery-and-classification-overview)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Les ressources doivent être séparées par un réseau virtuel ou un sous-réseau, étiquetés de manière appropriée et sécurisés au sein d’un groupe de sécurité réseau ou de Pare-feu Azure. Les ressources de stockage ou de traitement des données sensibles doivent être isolées. Utilisez Azure Private Link : déployez votre serveur Azure SQL au sein d’un réseau virtuel et connectez-vous en toute sécurité à l’aide d’Azure Private Link.

* [Guide pratique pour créer des abonnements Azure supplémentaires](https://docs.microsoft.com/azure/billing/billing-create-subscription)

* [Guide pratique pour créer des groupes d’administration](https://docs.microsoft.com/azure/governance/management-groups/create)

* [Guide pratique pour créer et utiliser des étiquettes](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

* [Guide pratique pour configurer Azure Private Link pour Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-private-endpoint-overview#how-to-set-up-private-link-for-azure-sql-database)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : Pour toute base de données Azure SQL de votre pool SQL dédié qui stocke ou traite des informations sensibles, marquez la base de données et les ressources associées comme sensibles à l’aide de balises. Configurez Azure Private Link en conjonction avec les balises de service du groupe de sécurité réseau (NSG) sur vos instances Azure SQL Database pour empêcher l’exfiltration d’informations sensibles.

En outre, Advanced Threat Protection pour Azure SQL Database, Azure SQL Managed Instance et Azure Synapse détecte les activités anormales indiquant des tentatives inhabituelles et potentiellement dangereuses d’accès aux bases de données ou d’exploitation de ces dernières.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

* [Guide pratique pour configurer Azure Private Link et les groupes de sécurité réseau afin d’empêcher l’exfiltration de données sur vos instances Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-private-endpoint-overview)

* [Présentation d’Advanced Threat Protection pour Azure SQL Database](https://docs.microsoft.com/azure/azure-sql/database/threat-detection-overview)

* [Présentation de la protection des données client dans Azure](https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Azure SQL Database sécurise vos données en utilisant le protocole Transport Layer Security pour chiffrer les données en mouvement. SQL Server applique en permanence le chiffrement (SSL/TLS) aux connexions. Ainsi, toutes les données sont chiffrées « en transit » entre le client et le serveur, quel que soit le paramètre Encrypt ou TrustServerCertificate de la chaîne de connexion.

* [Présentation du chiffrement Azure SQL en transit](https://docs.microsoft.com/azure/sql-database/sql-database-security-overview#information-protection-and-encryption)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Aide** : Utilisez la fonctionnalité Découverte et classification des données d’Azure Synapse SQL. Découverte et classification des données offre des capacités avancées intégrées à Azure SQL Database pour la découverte, la classification, l’étiquetage et la protection des données sensibles dans vos bases de données.

Découverte et classification des données fait partie de l’offre Advanced Data Security, qui est un package unifié de capacités de sécurité SQL avancées. L’accès au composant Découverte et classification des données ainsi que sa gestion se font via le portail SQL ADS central.

En outre, vous pouvez configurer une stratégie de masquage dynamique des données (DDM) dans le portail Azure. Le moteur de recommandations DDM signale certains champs de votre base de données comme étant des champs potentiellement sensibles qui peuvent être de bons candidats au masquage.

* [Utilisation de la découverte et de la classification des données pour Azure SQL Server](https://docs.microsoft.com/azure/sql-database/sql-database-data-discovery-and-classification)

* [Présentation du masquage dynamique des données pour Azure Synapse SQL](https://docs.microsoft.com/azure/sql-database/sql-database-dynamic-data-masking-get-started)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : Utilisez le contrôle d’accès en fonction du rôle (RBAC) Azure pour gérer l’accès aux bases de données Azure SQL dans votre pool SQL dédié.

L’autorisation est contrôlée par les appartenances aux rôles de base de données et les autorisations au niveau objet de votre compte d’utilisateur. Nous vous recommandons, à titre de meilleure pratique, d’accorder aux utilisateurs des privilèges aussi réduits que possible.

* [Guide pratique pour intégrer Azure SQL Server à Azure Active Directory à des fins d’authentification](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication)

* [Guide pratique pour contrôler l’accès dans Azure SQL Server](https://docs.microsoft.com/azure/sql-database/sql-database-control-access)

* [Présentation de l’authentification et de l’autorisation dans Azure SQL](https://docs.microsoft.com/azure/azure-sql/database/logins-create-manage)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Aide** : Non applicable. Microsoft gère l’infrastructure sous-jacente d’Azure Synapse SQL et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition des données client.

* [Présentation de la protection des données client dans Azure](https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide** : La technologie Transparent Data Encryption (TDE) vous aide à protéger Azure Synapse SQL contre les activités malveillantes hors connexion en chiffrant les données au repos. Il assure le chiffrement et le déchiffrement en temps réel de la base de données, des sauvegardes associées et des fichiers journaux des transactions au repos, sans que cela nécessite de modifier l’application. Dans Azure, la configuration par défaut de TDE spécifie que la clé de chiffrement de la base de données (DEK) est protégée par un certificat de serveur intégré. Vous pouvez également utiliser le chiffrement TDE géré par le client, aussi appelé prise en charge BYOK (Bring Your Own Key) pour TDE. Dans ce scénario, le protecteur TDE qui chiffre la clé DEK est une clé asymétrique managée par le client, stockée dans une solution Azure Key Vault du client, qui la gère (système d'administration de clés externe d’Azure) et il ne quitte jamais le coffre de clés.

* [Présentation du chiffrement transparent des données géré par le service](https://docs.microsoft.com/azure/azure-sql/database/transparent-data-encryption-tde-overview?tabs=azure-portal)

* [Présentation du chiffrement transparent des données géré par le client](https://docs.microsoft.com/azure/azure-sql/database/transparent-data-encryption-tde-overview?tabs=azure-portal#customer-managed-transparent-data-encryption---bring-your-own-key)

* [Guide pratique pour activer TDE à l’aide de votre propre clé](https://docs.microsoft.com/azure/azure-sql/database/transparent-data-encryption-byok-configure)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes en cas de modifications sur des instances de production de pools Synapse SQL et autres ressources critiques ou connexes.

En outre, vous pouvez configurer des alertes pour les bases de données de votre pool Synapse SQL à l’aide du portail Azure. Les alertes peuvent vous envoyer un e-mail ou appeler un webhook lorsqu'une métrique (taille de la base de données ou utilisation du processeur, par exemple) atteint le seuil.

* [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log)

* [Guide pratique pour créer des alertes pour Azure Synapse SQL](https://docs.microsoft.com/azure/azure-sql/database/alerts-insights-configure-portal)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](/azure/security/benchmarks/security-control-vulnerability-management).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Aide** : Activez Advanced Data Security et suivez les recommandations d’Azure Security Center concernant l’évaluation des vulnérabilités sur vos bases de données Azure SQL.

* [Procédure d’exécution de l’évaluation des vulnérabilités sur vos bases de données Azure SQL](https://docs.microsoft.com/azure/sql-database/sql-vulnerability-assessment)

* [Procédure d’activation d’Advanced Data Security](https://docs.microsoft.com/azure/sql-database/sql-database-advanced-data-security)

* [Implémenter les recommandations d'évaluation des vulnérabilités d'Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-vulnerability-assessment-recommendations)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="53-deploy-automated-patch-management-solution-for-third-party-software-titles"></a>5.3 : Déployer une solution de gestion automatisée des correctifs des logiciels tiers

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Aide** : Évaluation des vulnérabilités est un service d’analyse intégré à Azure Synapse SQL. Le service utilise une base de connaissances de règles qui signalent les failles de sécurité. Il met en évidence les écarts par rapport aux bonnes pratiques, tels que les configurations incorrectes, les autorisations excessives et les données sensibles non protégées. L’évaluation des vulnérabilités est accessible et peut être gérée par le biais du portail central SQL Advanced Data Security (ADS).

* [Procédure de gestion et d’exportation des analyses d’évaluation des vulnérabilités dans le portail SQL ADS](https://docs.microsoft.com/azure/sql-database/sql-vulnerability-assessment)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : Utilisez les évaluations de risques par défaut (degré de sécurisation) fournies par Azure Security Center.

Découverte et classification de données est intégrée à Azure Synapse SQL. Elle offre des fonctionnalités avancées pour la découverte, la classification, l’étiquetage et le signalement des données sensibles dans vos bases de données.

* [Présentation du degré de sécurisation Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-secure-score)

* [Présentation de Découverte et classification des données](https://docs.microsoft.com/azure/azure-sql/database/data-discovery-and-classification-overview)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](/azure/security/benchmarks/security-control-inventory-asset-management).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : Utilisez Azure Resource Graph pour interroger et découvrir toutes les ressources associées à votre pool SQL dédié au sein de vos abonnements. Vérifiez que vous disposez des autorisations (en lecture) appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent.

Bien que les ressources Azure classiques puissent être découvertes via Azure Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

* [Guide pratique pour créer des requêtes avec Azure Resource Graph](https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal)

* [Guide pratique pour afficher ses abonnements Azure](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0&preserve-view=true)

* [Présentation d’Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/overview)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

* [Guide pratique pour créer et utiliser des étiquettes](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des étiquettes, des groupes d’administration et diviser des abonnements, le cas échéant, pour organiser et suivre les ressources. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

* [Guide pratique pour créer des abonnements Azure supplémentaires](https://docs.microsoft.com/azure/billing/billing-create-subscription)

* [Guide pratique pour créer des groupes d’administration](https://docs.microsoft.com/azure/governance/management-groups/create)

* [Guide pratique pour créer et utiliser des étiquettes](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Conseils** : Définissez une liste de ressources Azure approuvées associées à votre pool SQL dédié.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients selon les définitions de stratégies intégrées suivantes :
- Types de ressources non autorisés
- Types de ressources autorisés

Utilisez Azure Resource Graph pour interroger/découvrir les ressources dans vos abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

* [Guide pratique pour configurer et gérer Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Guide pratique pour créer des requêtes avec Azure Resource Graph](https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal)

**Supervision Azure Security Center** : actuellement non disponible

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

* [Guide pratique pour configurer et gérer Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés

**Aide** : Non applicable. Cette recommandation a trait aux applications s’exécutant sur des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

* [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : Toute ressource associée à votre pool SQL dédié nécessaire à des opérations métier, mais qui peut présenter un risque plus élevé pour l’organisation, doit être isolée sur sa propre machine virtuelle ou son propre réseau virtuel et être suffisamment sécurisée à l’aide d’un pare-feu Azure ou d’un groupe de sécurité réseau.

* [Guide pratique pour créer un réseau virtuel](https://docs.microsoft.com/azure/virtual-network/quick-create-portal)

* [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](/azure/security/benchmarks/security-control-secure-configuration).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Conseils** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.Sql » pour créer des stratégies personnalisées visant à auditer des ressources associées à votre pool SQL dédié ou à appliquer la configuration de ces ressources. Vous pouvez également utiliser des définitions de stratégie intégrées pour des bases de données/serveurs Azure, par exemple :
- Déployer la détection de menaces sur les serveurs SQL
- SQL Server doit utiliser un point de terminaison de service de réseau virtuel

* [Affichage des alias Azure Policy disponibles](https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0&preserve-view=true)

* [Guide pratique pour configurer et gérer Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

* [Guide pratique pour configurer et gérer Azure Policy](https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage)

* [Présentation des effets d’Azure Policy](https://docs.microsoft.com/azure/governance/policy/concepts/effects)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous utilisez des définitions Azure Policy personnalisées, utilisez Azure DevOps ou Azure Repos pour stocker et gérer votre code en toute sécurité.

* [Stocker du code dans Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops&preserve-view=true)

* [Documentation Azure Repos](https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops&preserve-view=true)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Aide** : Non applicable. Azure Synapse SQL n’a pas de paramètres de sécurité configurables.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer des outils de gestion de la configuration pour les systèmes d'exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Conseils** : Tirez parti d’Azure Security Center pour effectuer des analyses de ligne de base pour toutes les ressources associées à votre pool SQL dédié.

* [Corriger les recommandations dans Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-sql-service-recommendations)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Aide** : Transparent Data Encryption (TDE) avec clés gérées par le client dans Azure Key Vault vous permet de chiffrer la clé de chiffrement de base de données (DEK) générée automatiquement à l’aide d’une clé asymétrique gérée par le client et appelée protecteur TDE. Cela s’appelle aussi généralement la prise en charge Bring Your Own Key (BYOK) pour Transparent Data Encryption. Dans le scénario BYOK, le protecteur TDE est stocké dans un coffre de clés Azure détenu et géré par le client. En outre, assurez-vous que la suppression réversible est activée dans Azure Key Vault.

* [Guide pratique pour activer TDE à l’aide d’une clé gérée par le client à partir d’Azure Key Vault](https://docs.microsoft.com/azure/azure-sql/database/transparent-data-encryption-byok-configure?tabs=azure-powershell)

* [Guide pratique pour activer la suppression réversible dans Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-soft-delete-powershell)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Aide** : Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure Active Directory (AD). Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Azure Key Vault, sans informations d’identification dans votre code.

* [Tutoriel : Utiliser une identité managée de machine virtuelle Windows affectée par le système pour accéder à Azure SQL](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-sql)

* [Configurer des identités managées](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Implémentez Credential Scanner pour identifier les informations d’identification dans votre code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

* [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](/azure/security/benchmarks/security-control-malware-defense).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul. Microsoft gère les logiciels anti-programme malveillant pour la plateforme sous-jacente.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Aide** : Le logiciel anti-programme malveillant de Microsoft est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure Synapse SQL), mais ne s’exécute pas sur du contenu client.

Pré-analysez tout contenu chargé sur des ressources Azure non liées au calcul, comme App Service, Data Lake Storage, Stockage Blob, Azure SQL Server, etc. Microsoft ne peut pas accéder à vos données dans ces instances.

* [Présentation de Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles](https://docs.microsoft.com/azure/security/fundamentals/antimalware)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul. Microsoft gère les logiciels anti-programme malveillant pour la plateforme sous-jacente.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-recovery"></a>Récupération de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Récupération de données](/azure/security/benchmarks/security-control-data-recovery).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : Garantir des sauvegardes automatiques régulières

**Conseils** : Des instantanés de votre pool SQL dédié sont automatiquement pris pendant la journée, créant des points de restauration qui sont disponibles pendant sept jours. Cette période de conservation ne peut pas être modifiée. Le pool SQL dédié prend en charge un objectif de point de récupération (RPO) de huit heures. Vous pouvez restaurer votre entrepôt de données dans la région primaire à partir de n’importe quelle capture instantanée prise au cours des sept derniers jours. Notez que vous pouvez également déclencher manuellement des instantanés si nécessaire.

* [Sauvegarde et restauration dans un pool SQL dédié](/azure/synapse-analytics/sql-data-warehouse/backup-and-restore)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Aide** : Des captures instantanées de votre entrepôt de données sont automatiquement prises pendant la journée, créant des points de restauration qui sont disponibles pendant sept jours. Cette période de conservation ne peut pas être modifiée. Le pool SQL dédié prend en charge un objectif de point de récupération (RPO) de huit heures. Vous pouvez restaurer votre entrepôt de données dans la région primaire à partir de n’importe quelle capture instantanée prise au cours des sept derniers jours. Notez que vous pouvez également déclencher manuellement des instantanés si nécessaire.

Si vous utilisez une clé gérée par le client pour chiffrer votre clé de chiffrement de base de données, assurez-vous que votre clé est sauvegardée.

* [Sauvegarde et restauration dans un pool SQL dédié](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/backup-and-restore)

* [Guide pratique pour sauvegarder des clés Azure Key Vault](https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0&preserve-view=true)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : Testez régulièrement vos points de restauration pour vous assurer que vos instantanés sont valides. Pour restaurer un pool SQL dédié existant à partir d’un point de restauration, vous pouvez utiliser le portail Azure ou PowerShell. Testez la restauration des clés gérées par le client sauvegardées.

* [Guide pratique pour restaurer des clés Azure Key Vault](https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0&preserve-view=true)

* [Sauvegarde et restauration dans un pool SQL dédié](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/backup-and-restore)

* [Guide pratique pour restaurer un pool SQL dédié existant](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-restore-active-paused-dw)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Dans Azure SQL Database, vous pouvez configurer une base de données unique ou mise en pool avec une stratégie de conservation des sauvegardes à long terme (LTR) afin de conserver automatiquement les sauvegardes de base de données dans des conteneurs de stockage Blob Azure distincts pendant une durée maximale de 10 ans. Les données dans Stockage Azure sont chiffrées et déchiffrées en toute transparence à l’aide du chiffrement AES 256 bits, un des chiffrements par blocs les plus puissants actuellement disponibles, et sont conformes à la norme FIPS 140-2.

Par défaut, les données d’un compte de stockage sont chiffrées à l’aide de clés managées par Microsoft. Vous pouvez vous reposer sur les clés managées par Microsoft pour le chiffrement des vos données, ou vous pouvez gérer le chiffrement avec vos propres clés. Si vous gérez vos propres clés à l’aide de Key Vault, assurez-vous que la suppression réversible est activée.

* [Guide pratique pour gérer la conservation à long terme des sauvegardes Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-long-term-backup-retention-configure)

* [Chiffrement du stockage Azure pour les données au repos](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)

* [Guide pratique pour activer la suppression réversible dans Key Vault](https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal)

**Supervision Azure Security Center** : N/A

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](/azure/security/benchmarks/security-control-incident-response).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Vérifiez qu’il existe des plans de réponse aux incidents écrits qui définissent les rôles du personnel, ainsi que les phases de gestion des incidents.

* [Comment configurer des automatisations de workflow dans Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center affecte un degré de gravité aux alertes pour vous aider à hiérarchiser l’ordre dans lequel vous remédiez à chaque alerte. Ainsi, quand une ressource est compromise, vous pouvez vous en occuper immédiatement. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

* [Alertes de sécurité dans le Centre de sécurité Azure](https://docs.microsoft.com/azure/security-center/security-center-alerts-overview)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

* [Vous pouvez vous reporter à la publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Instructions** : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le centre de réponse aux incidents de sécurité Microsoft (MSRC, Microsoft Security Response Center) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé.

* [Comment définir le contact de sécurité d’Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour transmettre en continu les alertes à Sentinel.

* [Comment configurer l’exportation continue](https://docs.microsoft.com/azure/security-center/continuous-export)

* [Comment envoyer des alertes à Azure Sentinel](https://docs.microsoft.com/azure/sentinel/connect-azure-security-center)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

* [Comment configurer l’automatisation des workflows et Logic Apps](https://docs.microsoft.com/azure/security-center/workflow-automation)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : Tests d’intrusion et exercices Red Team](/azure/security/benchmarks/security-control-penetration-tests-red-team-exercises).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Conseil** : * [Respectez les règles d’engagement de Microsoft pour garantir la conformité de vos tests d’intrusion aux stratégies Microsoft](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1.)

* [Vous trouverez ici plus d’informations sur la stratégie de Microsoft, sur l’exécution de Red Teaming et sur les tests d’intrusion de site actif sur l’infrastructure, les services et les applications cloud qui sont managés par Microsoft.](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](/azure/security/benchmarks/overview)
- En savoir plus sur les [bases de référence de la sécurité Azure](/azure/security/benchmarks/security-baselines-overview)
