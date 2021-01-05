---
title: Ligne de base de sécurité Azure pour Cosmos DB
description: Ligne de base de sécurité Azure pour Cosmos DB
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 03/16/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 961dae4cd35c7529a5661d1ddc7d0d1f6e4db634
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96029113"
---
# <a name="azure-security-baseline-for-cosmos-db"></a>Ligne de base de sécurité Azure pour Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

La ligne de base de sécurité Azure pour Cosmos DB contient des recommandations qui vous aideront à améliorer la situation de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1 : Protéger les ressources à l'aide de groupes de sécurité réseau ou du Pare-feu Azure sur votre réseau virtuel

**Conseils** : À l’aide d’Azure Private Link, vous pouvez vous connecter à un compte Azure Cosmos via un point de terminaison privé. Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public. Vous pouvez également réduire le risque d’exfiltration de données en configurant un ensemble strict de règles de trafic sortant sur un groupe de sécurité réseau (NSG) et en associant ce groupe de sécurité réseau à votre sous-réseau.

Vous pouvez également utiliser des points de terminaison de service pour sécuriser votre compte Azure Cosmos. En activant un point de terminaison de service, vous pouvez configurer des comptes Azure Cosmos pour autoriser l’accès uniquement à partir d’un sous-réseau spécifique de réseau virtuel Azure. Une fois le point de terminaison de service Azure Cosmos DB activé, vous pouvez limiter l’accès à un compte Azure Cosmos avec des connexions à partir d’un sous-réseau dans un réseau virtuel.

Vous pouvez également sécuriser les données stockées dans votre compte Azure Cosmos à l’aide de pare-feu IP. Azure Cosmos DB prend en charge les contrôles d’accès basés sur IP pour le pare-feu entrant. Vous pouvez définir un pare-feu IP sur le compte Azure Cosmos à l’aide du Portail Azure, de modèles Azure Resource Manager ou par le biais d’Azure CLI ou d’Azure PowerShell.

Présentation d’Azure Private Link : https://docs.microsoft.com/azure/private-link/private-link-overview

Comment configurer un point de terminaison privé pour Azure Cosmos DB :  https://docs.microsoft.com/azure/cosmos-db/how-to-configure-private-endpoints 

Comment créer un groupe de sécurité réseau avec une configuration de sécurité :  https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic

Comment configurer le pare-feu IP dans Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/how-to-configure-firewall

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2 : Surveiller et consigner la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Aide** : Utilisez Azure Security Center et suivez les recommandations de protection du réseau pour sécuriser les ressources réseau associées à votre compte Azure Cosmos.

Lorsque des machines virtuelles sont déployées dans le même réseau virtuel que votre compte Azure Cosmos, vous pouvez utiliser un groupe de sécurité réseau (NSG) pour réduire le risque d’exfiltration de données. Activez les journaux de flux NSG et transférez-les vers un compte de stockage Azure pour les audits du trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

Comprendre la sécurité réseau fournie par Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-network-recommendations

Activer les journaux de flux NSG : https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Comment activer et utiliser Traffic Analytics : https://docs.microsoft.com/azure/network-watcher/traffic-analytics

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Aide** : Utilisez la fonctionnalité CORS (Cross Origin Resource Sharing) pour permettre à une application web exécutée dans un domaine d’accéder aux ressources d’un autre domaine. Les navigateurs web implémentent une restriction de sécurité, appelée stratégie de même origine, qui empêche une page web d’appeler des API dans un domaine différent. Toutefois, CORS permet au domaine d’origine d’appeler des API dans un autre domaine de façon sécurisée. Après avoir activé la prise en charge CORS pour votre compte Azure Cosmos, seules les requêtes authentifiées sont évaluées pour déterminer si elles sont autorisées selon les règles que vous avez spécifiées.

Configuration du partage des ressources cross-origin : https://docs.microsoft.com/azure/cosmos-db/how-to-configure-cross-origin-resource-sharing

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Aide** : Utilisez ATP (Advanced Threat Protection) pour Azure Cosmos DB. ATP pour Azure Cosmos DB offre une couche supplémentaire de sécurité intelligente qui détecte les tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses des comptes Azure Cosmos. Cette couche de protection vous permet de traiter efficacement les menaces et de les intégrer aux systèmes centraux de supervision de la sécurité.

Activez la protection DDoS standard sur les réseaux virtuels associés à vos instances Azure Cosmos DB pour vous protéger contre les attaques DDoS. Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

Comment configurer Advanced Threat Protection pour Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/cosmos-db-advanced-threat-protection

Comment configurer la protection DDoS : https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection

Comprendre la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-alerts-service-layer

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5 : Consigner les paquets réseau et les journaux de flux

**Aide** : Activez les journaux de flux de groupe de sécurité réseau (NSG) et envoyez les journaux dans un compte de stockage pour auditer le trafic. Vous pouvez envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

Activer les journaux de flux NSG : https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Comment activer et utiliser Traffic Analytics : https://docs.microsoft.com/azure/network-watcher/traffic-analytics

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Aide** : Utilisez ATP (Advanced Threat Protection) pour Azure Cosmos DB. ATP pour Azure Cosmos DB offre une couche supplémentaire de sécurité intelligente qui détecte les tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses des comptes Azure Cosmos. Cette couche de protection vous permet de traiter efficacement les menaces et de les intégrer aux systèmes centraux de supervision de la sécurité. 

Comment configurer Advanced Threat Protection pour Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/cosmos-db-advanced-threat-protection

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Non applicable. La recommandation concerne les applications web s’exécutant sur Azure App Service ou les ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Aide** : Pour les ressources qui doivent accéder à votre compte Azure Cosmos, utilisez des étiquettes de service de réseau virtuel afin de définir des contrôles d’accès réseau sur des groupes de sécurité réseau ou le Pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de l’étiquette de service (par exemple, AzureCosmosDB) dans le champ Source ou Destination approprié d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Pour plus d’informations sur l’utilisation d’étiquettes de service : https://docs.microsoft.com/azure/virtual-network/service-tags-overview

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Aide** : Définissez et implémentez des configurations de sécurité standard pour les ressources réseau à l'aide d'Azure Policy. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.DocumentDB » et « Microsoft.Network » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de vos instances Azure Cosmos DB. Vous pouvez également utiliser des définitions de stratégie intégrée pour Azure Cosmos DB, par exemple :

- Déployer Advanced Threat Protection pour les comptes Cosmos DB

- Cosmos DB doit utiliser un point de terminaison de service de réseau virtuel

Vous pouvez également utiliser Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les artefacts d’environnement clés, tels que les modèles Resource Manager, le contrôle d’accès en fonction du rôle Azure (Azure RBAC) et les stratégies au sein d’une seule définition de blueprint. Vous pouvez facilement appliquer le blueprint aux nouveaux abonnements et environnements, et ajuster le contrôle et la gestion par le biais du contrôle de version.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Créer une instance Azure Blueprint : https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Aide** : Utilisez des balises pour les ressources réseau associées à votre déploiement Azure Cosmos DB afin de les organiser logiquement dans une taxonomie.

Comment créer et utiliser des étiquettes : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Aide** : Utilisez le journal d’activité Azure pour surveiller les configurations des ressources réseau et détecter les modifications de celles-ci associées à vos instances Azure Cosmos DB. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques. 

Consulter et récupérer les événements du journal des activités Azure : https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view

Créer des alertes dans Azure Monitor : https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Contrôle de sécurité : journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Aide** : Microsoft conserve la source de temps utilisée pour les ressources Azure, par exemple Azure Cosmos DB, pour les horodatages dans les journaux.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Ingérez les journaux via Azure Monitor pour agréger les données de sécurité générées par Azure Cosmos DB. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, puis utilisez les comptes de stockage pour le stockage à long terme/d’archivage. Vous pouvez également activer et intégrer des données dans Azure Sentinel ou une solution SIEM (Security Information and Event Management) tierce. 

Comment activer les journaux de diagnostic pour Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/logging

Comment intégrer Azure Sentinel : https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Aide** : Activez les paramètres de diagnostic pour Azure Cosmos DB et envoyez les journaux vers un espace de travail ou un compte de stockage Log Analytics. Les paramètres de diagnostic dans Azure Cosmos DB sont utilisés pour collecter les journaux des ressources. Ces journaux sont capturés par requête et sont également appelés « journaux des plans de données ». Les opérations Delete, Insert et Read sont des exemples d’opérations de plan de données. Vous pouvez également activer les paramètres de diagnostic des journaux d’activité Azure et les envoyer au même espace de travail Log Analytics.

Comment activer les paramètres de diagnostic pour Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/logging

Comment activer les paramètres de diagnostic pour le journal d'activité Azure : https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Dans Azure Monitor, définissez la période de rétention des journaux pour les espaces de travail Log Analytics associés à vos instances Azure Cosmos DB conformément aux réglementations de conformité de votre organisation.

Définir des paramètres de rétention des journaux : https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Vous pouvez exécuter des requêtes dans Log Analytics, un espace de travail permet de rechercher des termes, d’identifier des tendances, d’analyser des modèles et de fournir de nombreuses autres informations basées sur les journaux de Azure Cosmos DB que vous avez collectés.

Comment exécuter des requêtes pour Azure Cosmos DB dans les espaces de travail Log Analytics : https://docs.microsoft.com/azure/cosmos-db/monitor-cosmos-db

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer des alertes en cas d’activité anormale

**Aide** : Dans Azure Security Center, activez Advanced Threat Protection pour Azure Cosmos DB afin de surveiller les activités anormales dans les journaux de sécurité et les événements. Activez les paramètres de diagnostic dans Azure Cosmos DB et envoyez les journaux à un espace de travail Log Analytics.

 

Vous pouvez également intégrer votre espace de travail Log Analytics à Azure Sentinel, car cela fournit une solution SOAR (Security Orchestration Automated Response). Cela permet de créer des playbooks (solutions automatisées) utilisables pour corriger des problèmes de sécurité. En outre, vous pouvez créer des alertes de journal personnalisées dans votre espace de travail Log Analytics à l’aide d’Azure Monitor.

Liste des alertes de protection contre les menaces pour Azure Cosmos DB : https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-azurecosmos

Comment intégrer Azure Sentinel : https://docs.microsoft.com/azure/sentinel/quickstart-onboard

Créer, afficher et gérer des alertes de journal à l’aide d’Azure Monitor : https://docs.microsoft.com/azure/azure-monitor/platform/alerts-log

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide** : Non applicable. Azure Cosmos DB ne traite pas et ne produit pas de journaux liés aux logiciels anti-programmes malveillants.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Aide** : Non applicable. Azure Cosmos DB ne traite pas et ne produit pas de journaux liés au DNS.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : Vous pouvez utiliser le volet de gestion des identités et des accès (IAM) dans le Portail Azure pour configurer le contrôle d’accès en fonction du rôle Azure (Azure RBAC) et conserver l’inventaire sur les ressources d’Azure Cosmos DB. Les rôles sont appliqués aux utilisateurs, aux groupes, aux principaux de service et aux identités gérées dans Active Directory. Vous pouvez utiliser des rôles intégrés ou des rôles personnalisés pour les individus et les groupes.

Azure Cosmos DB intègre Azure RBAC pour les scénarios de gestion courants dans Azure Cosmos DB. Une personne disposant d’un profil dans Azure Active Directory (AD) peut assigner ces rôles Azure à des utilisateurs, des groupes, des principaux de services ou des identités gérées pour accorder ou refuser l’accès aux ressources et aux opérations sur les ressources d’Azure Cosmos DB.

Vous pouvez également utiliser le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes qui sont membres de groupes d’administration. 

De plus, certaines actions dans Azure Cosmos DB peuvent être contrôlées avec Azure Active Directory et de clés primaires spécifiques au compte.  Utilisez le paramètre de compte « disableKeyBasedMetadataWriteAccess » pour contrôler l’accès aux clés.

Comprendre le contrôle d’accès en fonction du rôle Azure (RBAC) dans Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/role-based-access-control

Créez vos propres rôles personnalisés à l’aide des actions Azure Cosmos DB (espace de noms Microsoft.DocumentDB) : https://docs.microsoft.com/azure/role-based-access-control/resource-provider-operations#microsoftdocumentdb

Créer un nouveau rôle dans Azure Active Directory : https://docs.microsoft.com/azure/role-based-access-control/custom-roles

Comment obtenir un rôle d’annuaire dans Azure Active Directory avec PowerShell : https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0&preserve-view=true

Comment obtenir les membres d’un rôle d’annuaire dans Azure Active Directory avec PowerShell : https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0&preserve-view=true

Restreindre l’accès des utilisateurs aux opérations de données uniquement : https://docs.microsoft.com/azure/cosmos-db/how-to-restrict-user-data

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Aide** : Le concept de mots de passe par défaut ou vides n’existe pas en ce qui concerne Azure AD ou Azure Cosmos DB. À la place, Azure Cosmos DB utilise deux types de clés pour authentifier les utilisateurs et permettre d’accéder à ses données et à ses ressources : les clés primaires et les jetons de ressources. Les clés peuvent être régénérées à tout moment.

Comprendre l’accès sécurisé aux données dans Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/secure-access-to-data

Régénération des clés d’Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/manage-with-powershell#regenerate-keys

Comment accéder par programme aux clés à l’aide d’Azure Active Directory : https://docs.microsoft.com/azure/cosmos-db/certificate-based-authentication

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Non applicable ; Azure Cosmos DB ne prend pas en charge les comptes d’administrateur.  Tous les accès sont intégrés à Azure Active Directory et au contrôle d’accès en fonction du rôle Azure (Azure RBAC).



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Aide** : Azure Cosmos DB utilise deux types de clés pour autoriser les utilisateurs et ne prend pas en charge l’authentification unique (SSO) au niveau du plan de données. L’accès au plan de contrôle pour Cosmos DB est disponible via l’API REST et prend en charge l’authentification unique. Pour vous authentifier, définissez l’en-tête d’autorisation pour vos demandes sur un jeton web JSON que vous avez obtenu auprès d’Azure Active Directory.

Comprendre l’API REST d’Azure Database for Cosmos DB : https://docs.microsoft.com/rest/api/cosmos-db/

Comprendre l’authentification unique avec Azure Active Directory : https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Aide** : Activer l’authentification multifacteur Azure Active Directory et suivre les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

Comment activer l’authentification multifacteur dans Azure : https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Comment surveiller l’identité et l’accès dans Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Aide** : Utilisez des stations de travail disposant d’un accès privilégié avec Multi-Factor Authentication configuré pour vous connecter aux ressources Azure et les configurer.

En savoir plus sur les stations de travail avec accès privilégié : https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations

Comment activer l’authentification multifacteur dans Azure : https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration

**Aide** : Utilisez ATP (Advanced Threat Protection) pour Azure Cosmos DB. ATP pour Azure Cosmos DB offre une couche supplémentaire de sécurité intelligente qui détecte les tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses des comptes Azure Cosmos. Cette couche de protection vous permet de traiter efficacement les menaces et de les intégrer aux systèmes centraux de supervision de la sécurité. 

En outre, vous pouvez utiliser Azure Active Directory (AD) Privileged Identity Management pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.

Utilisez les détections de risque Azure AD pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs.

Comment déployer Privileged Identity Management (PIM) : https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan

Comprendre les détections de risques Azure AD : https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Configurez la condition d’emplacement d’une stratégie d’accès conditionnel et gérez vos emplacements nommés. Avec les emplacements nommés, vous pouvez créer des regroupements logiques de plages d’adresses IP ou de pays et régions. Vous pouvez restreindre l’accès à des ressources sensibles, telles que vos instances Azure Cosmos DB, à vos emplacements nommés configurés.

Comment configurer des emplacements nommés dans Azure : https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Conseils** : Utiliser Azure Active Directory (AD) comme système d’authentification et d’autorisation central. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

Comment créer et configurer une instance d’Azure Active Directory : https://docs.microsoft.com/azure/active-directory-domain-services/tutorial-create-instance

Comment configurer et gérer l’authentification Azure Active Directory avec Azure SQL : https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication-configure

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory fournit des journaux pour vous aider à découvrir les comptes obsolètes. De plus, vous pouvez utiliser les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès de l’utilisateur peut être passé en revue régulièrement pour vérifier que seuls les utilisateurs appropriés bénéficient d’un accès permanent.

Comment utiliser les révisions d’accès des identités Azure : https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés

**Aide** : Vous pouvez créer des paramètres de diagnostic pour des comptes d’utilisateur Azure Active Directory, en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics dans lequel vous pouvez configurer les alertes souhaitées.

Comment intégrer des journaux d’activité Azure dans Azure Monitor : https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Conseils** : Utilisez ATP (Advanced Threat Protection) pour Azure Cosmos DB. ATP pour Azure Cosmos DB offre une couche supplémentaire de sécurité intelligente qui détecte les tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses des comptes Azure Cosmos. Cette couche de protection vous permet de traiter efficacement les menaces et de les intégrer aux systèmes centraux de supervision de la sécurité. 

Vous pouvez utiliser Azure Active Directory Identity Protection et la fonctionnalité de détections de risques pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. De plus, vous pouvez ingérer des journaux dans Azure Sentinel pour approfondir votre examen.

Comment afficher les connexions risquées Azure Active Directory : https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins

Comment configurer et activer des stratégies de protection des identités : https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

Comment intégrer Azure Sentinel : https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : Customer Lockbox ne s’applique pas à Cosmos DB.  Les employés de Microsoft ne peuvent pas accéder aux ressources de données même avec les autorisations des clients.

Liste des services pris en charge pour Customer Lockbox : https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Contrôle de sécurité : protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des instances Azure Cosmos DB qui stockent ou traitent des informations sensibles.

Comment créer et utiliser des étiquettes : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Les instances Azure Cosmos DB sont séparées par un réseau virtuel/sous-réseau, étiquetés de manière appropriée et sécurisés dans un groupe de sécurité réseau (NSG) ou un Pare-feu Azure. Les instances Azure Cosmos DB stockant des données sensibles doivent être isolées. À l’aide d’Azure Private Link, vous pouvez vous connecter à un compte d’instance Azure Cosmos DB via un point de terminaison privé. Le point de terminaison privé est un ensemble d’adresses IP privées dans un sous-réseau au sein de votre réseau virtuel. Vous pouvez ensuite limiter l’accès aux adresses IP privées sélectionnées. 

Créer des abonnements Azure supplémentaires : https://docs.microsoft.com/azure/billing/billing-create-subscription

Comment créer des groupes d’administration : https://docs.microsoft.com/azure/governance/management-groups/create

Comment créer et utiliser des étiquettes : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

Comment configurer un point de terminaison privé pour Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/how-to-configure-private-endpoints

Comment créer un groupe de sécurité réseau avec une configuration de sécurité : https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : Vous pouvez déployer Advanced Threat Protection pour Azure Cosmos DB, qui va détecter les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles ou potentiellement dangereuses de bases de données. Il peut actuellement déclencher les alertes suivantes :

- Accès à partir d’emplacements inhabituels

- Extraction de données inhabituelles

En outre, lorsque vous utilisez des machines virtuelles pour accéder à vos instances Azure Cosmos DB, utilisez un lien privé, un pare-feu, des groupes de sécurité réseau et des étiquettes de service pour limiter les risques d’exfiltration de données. Microsoft gère l’infrastructure sous-jacente d’Azure Cosmos DB et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

Comment configurer Advanced Threat Protection pour Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/cosmos-db-advanced-threat-protection

Comprendre la protection des données client dans Azure : https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Toutes les connexions à Azure Cosmos DB prennent en charge le protocole HTTPS. Azure Cosmos DB prend également en charge le protocole TLS1.2. Il est possible d’appliquer une version TLS minimale côté serveur. Pour ce faire, veuillez contacter [azurecosmosdbtls@service.microsoft.com](mailto:azurecosmosdbtls@service.microsoft.com).

Vue d’ensemble de la sécurité Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/database-security

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Aide** : Les fonctionnalités d’identification automatique des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour Azure Cosmos DB. Toutefois, vous pouvez utiliser l’intégration de la Recherche cognitive Azure pour la classification et l’analyse des données. Vous pouvez également implémenter une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

Indexer les données Azure Cosmos DB avec Recherche cognitive Azure : https://docs.microsoft.com/azure/search/search-howto-index-cosmosdb?toc=/azure/cosmos-db/toc.json&amp;bc=/azure/cosmos-db/breadcrumb/toc.json

Comprendre la protection des données client dans Azure : https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : Azure Cosmos DB fournit un contrôle d’accès en fonction du rôle Azure (Azure RBAC) intégré pour les scénarios de gestion courants dans Azure Cosmos DB. Une personne disposant d’un profil dans Azure Active Directory peut assigner ces rôles Azure à des utilisateurs, des groupes, des principaux de service ou des identités managées pour accorder ou refuser l’accès aux ressources et aux opérations sur les ressources Azure Cosmos DB. L’attribution des rôles est limitée au seul accès au plan de contrôle, ce qui inclut l’accès aux comptes, bases de données, conteneurs et offres (débit) d’Azure Cosmos.

Comment implémenter Azure RBAC dans Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/role-based-access-control

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Aide** : Non applicable. Ces conseils concernent les ressources de calcul.

Microsoft gère l’infrastructure sous-jacente de Cosmos DB, et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

Comprendre la protection des données client dans Azure : https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : Toutes les données utilisateur stockées dans Cosmos DB sont chiffrées au repos par défaut. Il n’y a aucun contrôle pour le désactiver. Azure Cosmos DB utilise le chiffrement AES-256 dans toutes les régions où le compte est exécuté.

Par défaut, Microsoft gère les clés utilisées pour chiffrer les données dans votre compte Azure Cosmos. Vous pouvez éventuellement choisir d’ajouter une deuxième couche de chiffrement avec vos propres clés.

Comprendre le chiffrement au repos avec Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/database-encryption-at-rest

Comprendre la gestion des clés pour le chiffrement au repos avec Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/cosmos-db-security-controls

Comment configurer des clés gérées par le client pour votre compte Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/how-to-setup-cmk

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : Utilisez Azure Monitor avec le journal des activités Azure pour créer des alertes lorsque des modifications sont apportées à des instances de production d’Azure Cosmos DB.

Comment créer des alertes pour les événements du journal d’activité Azure : https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

Comment créer des alertes pour les événements du journal d’activité Azure : https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Aide** : Suivez les recommandations d’Azure Security Center pour vos instances d’Azure Cosmos DB. 

Microsoft effectue des mises à jour correctives du système et la gestion des vulnérabilités sur les hôtes sous-jacents qui prennent en charge vos instances Azure Cosmos DB. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

Fonctionnalités prises en charge disponibles dans Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-services?tabs=features-windows

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Aide** : Non applicable. Ces conseils concernent les ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3 : Déployer une solution de gestion de correctif logiciel tiers automatisée

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Aide** : Non applicable. Ces conseils concernent les ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1 : Utiliser la découverte de ressources Azure

**Aide** : Utilisez le Portail Azure ou Azure Resource Graph pour découvrir toutes les ressources (sans se limiter à Azure Cosmos DB, mais également pour les ressources telles que le calcul, les autres stockages, le réseau, les ports et les protocoles, etc.) au sein de vos abonnements.  Vérifiez que vous disposez des autorisations appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

Créer des requêtes avec Azure Resource Graph : https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Afficher vos abonnements Azure : https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0&preserve-view=true

Comprendre le contrôle d’accès en fonction du rôle Azure : https://docs.microsoft.com/azure/role-based-access-control/overview

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Aide** : Appliquez des étiquettes à vos instances Azure Cosmos DB et aux ressources associées présentant des métadonnées pour les organiser logiquement dans une taxonomie.

Comment créer et utiliser des étiquettes : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

Quelles ressources Azure Cosmos DB prennent en charge les étiquettes : https://docs.microsoft.com/azure/azure-resource-manager/management/tag-support#microsoftdocumentdb

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des étiquettes, des groupes d’administration, voire des abonnements séparés, pour organiser et suivre les ressources associées, y compris mais sans s’y limiter les ressources Azure Cosmos DB. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

Créer des abonnements Azure supplémentaires : https://docs.microsoft.com/azure/billing/billing-create-subscription

Créer des groupes d’administration : https://docs.microsoft.com/azure/governance/management-groups/create

Créer et utiliser des balises : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4 : Tenir un inventaire des ressources Azure approuvées et titres des logiciels

**Aide** : Non applicable. Ces conseils concernent les ressources de calcul et Azure dans son ensemble.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

Utilisez également Azure Resource Graph pour interroger/découvrir des ressources dans les abonnements.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Créer des requêtes avec Azure Graph : https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : Non applicable. Cette ligne de base concerne les ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul et Azure dans son ensemble.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Aide** : Non applicable. Ces conseils concernent les ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :

- Types de ressources non autorisés 

- Types de ressources autorisés

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Comment refuser un type de ressource spécifique avec Azure Policy : https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-implement-approved-application-list"></a>6.10 : Implémenter une liste d’applications approuvées

**Aide** : Non applicable. Ces conseils concernent les ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="611-limit-users-ability-to-interact-with-azureresources-manager-via-scripts"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager via des scripts

**Conseils** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ». Cela peut empêcher la création et la modification de ressources dans un environnement de haute sécurité.

Configurer l’accès conditionnel pour bloquer l’accès à Azure Resource Manager : https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Aide** : Non applicable. Ces conseils concernent les ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Aide** : Non applicable. Ces conseils ont trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Définissez et implémentez des configurations de sécurité standard pour vos instances Cosmos DB avec Azure Policy. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.DocumentDB » pour créer des stratégies personnalisées d’audit ou d’application de la configuration de vos instances Cosmos DB. Vous pouvez également utiliser des définitions de stratégie intégrée pour Azure Cosmos DB, par exemple :

- Déployer Advanced Threat Protection pour les comptes Cosmos DB

- Cosmos DB doit utiliser un point de terminaison de service de réseau virtuel

Affichage des alias Azure Policy disponibles : https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0&preserve-view=true

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Comprendre les effets d’Azure Policy : https://docs.microsoft.com/azure/governance/policy/concepts/effects

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous utilisez des définitions Azure Policy personnalisées pour vos ressources Cosmos DB ou associées, utilisez Azure Repos pour stocker et gérer votre code de manière sécurisée.

Documentation Azure Repos : https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops&preserve-view=true https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops&preserve-view=true

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7 : Déployer les outils de gestion de configuration système

**Aide** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.DocumentDB » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la surveillance de la configuration automatique pour les services Azure

**Aide** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.DocumentDB » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. Utilisez les commandes Azure Policy [auditer], [refuser] et [déployer si elle n’existe pas] pour appliquer automatiquement des configurations pour vos instances Azure Cosmos DB et les ressources associées. 

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Aide** : Pour les machines virtuelles Azure ou les applications web s’exécutant sur Azure App Service utilisées pour accéder à vos instances Azure Cosmos DB, utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets Azure Cosmos DB. Vérifiez que la suppression réversible est activée dans Key Vault.

Intégration aux identités managées Azure : https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

Pour créer un coffre de clés : https://docs.microsoft.com/azure/key-vault/general/quick-create-portal

Comment s’authentifier auprès de Key Vault : https://docs.microsoft.com/azure/key-vault/general/authentication

Comment attribuer une stratégie d’accès Key Vault : https://docs.microsoft.com/azure/key-vault/general/assign-access-policy-portal

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Aide** : Pour les machines virtuelles Azure ou les applications web s’exécutant sur Azure App Service utilisées pour accéder à vos instances Azure Cosmos DB, utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets Azure Cosmos DB.

Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure Active Directory (AD). Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Key Vault, sans informations d’identification dans votre code.

Configurer des identités managées : https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm

Intégration aux identités managées Azure : https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

Comment configurer Credential Scanner : https://secdevtools.azurewebsites.net/helpcredscan.html

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Aide** : Non applicable. Ces conseils concernent les ressources de calcul. Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure App Service), mais il ne s’exécute pas sur du contenu client.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Aide** : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure App Service), mais il ne s’exécute pas sur du contenu client.

Il vous incombe de pré-analyser tous les fichiers téléchargés vers des ressources Azure sans calcul, y compris Azure Cosmos DB. Microsoft ne peut pas accéder aux données client et ne peut donc pas effectuer d’analyses anti-programme malveillant du contenu client en votre nom.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Aide** : Non applicable. Le benchmark a trait aux ressources de calcul. Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure, mais il ne s’exécute pas sur du contenu client.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-recovery"></a>Récupération des données

*Pour plus d’informations, consultez [Contrôle de sécurité : récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Aide** : Azure Cosmos DB prend des captures instantanées de vos données toutes les quatre heures. Toutes les sauvegardes sont stockées séparément dans un service de stockage, et ces sauvegardes sont répliquées globalement pour garantir la résilience contre les sinistres régionaux. À chaque instant, seuls les deux dernières captures instantanées sont conservées. Toutefois, si le conteneur ou la base de données est supprimé, Azure Cosmos DB conserve les captures instantanées existantes du conteneur ou de la base de données pendant 30 jours. Contactez le support Azure pour effectuer une restauration à partir d’une sauvegarde.

Comprendre les sauvegardes automatisées Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/online-backup-and-restore

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Conseils** : Azure Cosmos DB sauvegarde automatiquement vos données à intervalles réguliers. Si une base de données ou un conteneur est supprimé, vous pouvez émettre un ticket de support ou appeler le support Azure pour restaurer les données à partir de sauvegardes en ligne automatiques. Le support Azure est disponible seulement pour certains plans, comme Standard, Développeur et les plans plus élevés. Pour restaurer une capture instantanée spécifique de la sauvegarde, Azure Cosmos DB exige que les données soient accessibles pendant la durée du cycle de sauvegarde de cette capture instantanée. 

Si vous utilisez Key Vault pour stocker les informations d’identification de vos instances Cosmos DB, veillez à faire des sauvegardes automatisées régulières de vos clés.

Comprendre les sauvegardes automatisées Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/online-backup-and-restore

Comment restaurer des données dans Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/how-to-backup-and-restore

Comment sauvegarder des clés Key Vault : https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : valider toutes les sauvegardes, y compris les clés gérées par le client

**Aide** : Si une base de données ou un conteneur est supprimé, vous pouvez émettre un ticket de support ou appeler le support Azure pour restaurer les données à partir de sauvegardes en ligne automatiques. Le support Azure est disponible seulement pour certains plans, comme Standard, Développeur et les plans plus élevés. Pour restaurer une capture instantanée spécifique de la sauvegarde, Azure Cosmos DB exige que les données soient accessibles pendant la durée du cycle de sauvegarde de cette capture instantanée.

Testez la restauration de vos secrets stockés dans Azure Key Vault à l’aide de PowerShell. Le cmdlet Restore-AzureKeyVaultKey crée une clé dans le coffre de clés spécifié. Cette clé est un réplica de la clé sauvegardée dans le fichier d’entrée et porte le même nom que la clé d’origine.

Comprendre les sauvegardes automatisées Azure Cosmos DB :

https://docs.microsoft.com/azure/cosmos-db/online-backup-and-restore

Comment restaurer des données dans Azure Cosmos DB :

https://docs.microsoft.com/azure/cosmos-db/how-to-backup-and-restore

Comment restaurer des secrets Azure Key Vault :

https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0&preserve-view=true

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Comme toutes les données utilisateur stockées dans Cosmos DB sont chiffrées au repos et lors du transport, vous n’avez aucune action à effectuer. En d’autres termes, le chiffrement au repos est « activé » par défaut. Il n’existe aucun contrôle à activer ou à désactiver. Azure Cosmos DB utilise le chiffrement AES-256 dans toutes les régions où le compte est exécuté.

Activez la suppression réversible dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.

Comprendre le chiffrement des données dans Azure Cosmos DB : https://docs.microsoft.com/azure/cosmos-db/database-encryption-at-rest

Comment activer la suppression réversible dans Key Vault : https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal

**Supervision Azure Security Center** : Oui

**Responsabilité** : Partagé

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

Vous pouvez également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de votre propre plan de réponse aux incidents : https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf

Comment configurer des automatisations de workflow dans Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide

Aide sur la création de votre propre processus de réponse aux incidents de sécurité : https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/

Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft : https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que le Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez clairement les abonnements (par ex. production, non production) et créez un système d’attribution de noms pour identifier et classer les ressources Azure de façon claire.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Aide** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

Reportez-vous à la publication du NIST : « Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities » : https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Conseils** : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé.  Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

Comment définir le contact de sécurité d’Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser la sentinelle des alertes.

Comment configurer l’exportation continue : https://docs.microsoft.com/azure/security-center/continuous-export

Comment diffuser en continu des alertes dans Azure Sentinel : https://docs.microsoft.com/azure/sentinel/connect-azure-security-center

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

Comment configurer l’automatisation des workflows et Logic Apps : https://docs.microsoft.com/azure/security-center/workflow-automation

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1 : Procédez régulièrement à des tests d’intrusion de vos ressources Azure et veillez à résoudre tous les problèmes de sécurité critiques détectés dans un délai de 60 jours

**Aide** : Suivre les règles d’engagement de Microsoft pour garantir que vos tests d’intrusion sont conformes aux stratégies de Microsoft : https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

Vous trouverez plus d’informations sur la stratégie de Microsoft, sur l’exécution de Red Teaming, et sur les tests d’intrusion de site actif sur l’infrastructure, les services et les applications cloud managés par Microsoft ici : https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consultez le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- Apprenez-en davantage sur les [Bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)
