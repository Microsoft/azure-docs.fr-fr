---
title: Ligne de base de sécurité Azure pour Key Vault
description: Ligne de base de sécurité Azure pour Key Vault
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/10/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 7d116010d5e6fea5e1ba8a66431b874194506283
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94843449"
---
# <a name="azure-security-baseline-for-key-vault"></a>Ligne de base de sécurité Azure pour Key Vault

La ligne de base de sécurité Azure pour Key Vault contient des recommandations qui vous aideront à améliorer la situation de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : sécurité réseau](../../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1 : Protéger les ressources à l'aide de groupes de sécurité réseau ou du Pare-feu Azure sur votre réseau virtuel

**Conseils** : Intégrez Azure Key Vault avec Azure Private Link. 

Le service Azure Private Link vous permet d’accéder aux services Azure (par exemple, Azure Key Vault) ainsi que les services de partenaires ou de clients hébergés par Azure sur un point de terminaison privé de votre réseau virtuel.

Un point de terminaison privé Azure est une interface réseau qui vous connecte de façon privée et sécurisée à un service basé sur la technologie Azure Private Link. Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel. Sachant que l’ensemble du trafic à destination du service peut être routé via le point de terminaison privé, il n’y a aucun besoin de passerelles, d’appareils NAT, de connexions ExpressRoute ou VPN ou d’adresses IP publiques. Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public. Vous pouvez vous connecter à une instance d’une ressource Azure, ce qui vous donne le plus haut niveau de granularité en matière de contrôle d’accès.

Comment intégrer Key Vault avec Azure Private Link :

https://docs.microsoft.com/azure/key-vault/private-link-service


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2 : Surveiller et consigner la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Conseils** : Utilisez Azure Security Center et suivez les recommandations de protection du réseau pour sécuriser vos ressources configurées par Key Vault dans Azure. 

Pour plus d’informations sur la sécurité réseau fournie par Azure Security Center : 

https://docs.microsoft.com/azure/security-center/security-center-network-recommendations

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Conseils** : Activez le Service Protection DDoS Standard Azure sur les réseaux virtuels Azure associés à vos instances Key Vault pour la protection contre des attaques par déni de service distribué. Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

 
Gérer le Service Azure DDoS Protection à l’aide du portail Azure : https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection

Détection des menaces pour la couche des services Azure dans Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-alerts-service-layer


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5 : Consigner les paquets réseau et les journaux de flux

**Conseils** : Azure Key Vault n’utilise pas les groupes de sécurité réseau (NSG), et les journaux de flux pour Azure Key Vault ne sont pas capturés. Au lieu de cela, utilisez Azure Private Link pour sécuriser vos instances Azure Key Vault, et activez les paramètres de diagnostic pour enregistrer les mesures et les événements d’audit.

Intégrer Key Vault avec Azure Private Link :

https://docs.microsoft.com/azure/key-vault/private-link-service

Journalisation d’Azure Key Vault : https://docs.microsoft.com/azure/key-vault/key-vault-logging



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Conseils** : Cette exigence peut être satisfaite en configurant la protection avancée contre les menaces (ATP, Advanced Threat Protection) pour Azure Key Vault. ATP fournit une couche supplémentaire de veille de sécurité. Cet outil détecte les tentatives potentiellement dangereuses d’accès à des comptes Azure Key Vault ou d’exploitation de ceux-ci.

Quand Azure Security Center détecte une activité anormale, il affiche des alertes. Il envoie également un e-mail à l’administrateur des abonnements avec les détails de l’activité suspecte et des recommandations sur la façon d’examiner et de corriger les menaces identifiées.

Configurer la protection avancée contre les menaces pour Azure Key Vault :

https://docs.microsoft.com/azure/security-center/advanced-threat-protection-key-vault



**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : Pour les ressources qui doivent accéder à vos instances Azure Key Vault, utilisez des étiquettes de service Azure pour Azure Key Vault afin de définir des contrôles d’accès réseau sur des groupes de sécurité réseau ou le Pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de la balise de service (par exemple, ApiManagement) dans le champ Source ou Destination approprié d'une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Vue d’ensemble des étiquettes de service Azure : https://docs.microsoft.com/azure/virtual-network/service-tags-overview


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour les ressources réseau associées à vos instances Azure Key Vault à l’aide d’Azure Policy. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.keyvault » et « Microsoft. Network » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de vos instances Azure Key Vault. Vous pouvez également utiliser des définitions de stratégie intégrée en lien avec Azure Key Vault, par exemple :

Key Vault doit utiliser un point de terminaison de service de réseau virtuel

Tutoriel : Créer et gérer des stratégies pour assurer la conformité :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Exemples Azure Policy :

https://docs.microsoft.com/azure/governance/policy/samples/#networ

Démarrage rapide : Définir et affecter un blueprint dans le portail :

https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : Utilisez des étiquettes pour les ressources liées à la sécurité réseau et au flux de trafic de vos instances Azure Key Vault afin de fournir des métadonnées et une organisation logique.

Utilisez l’une des définitions Azure Policy intégrées en lien avec l’étiquetage, comme « Exiger une étiquette et sa valeur », pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

Utilisez des étiquettes pour organiser vos ressources Azure :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : Utilisez le journal d’activité Azure pour surveiller les configurations des ressources réseau et détecter les modifications de celles-ci associées à vos instances Azure Key Vault. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.

Affichez et récupérez les événements du journal d’activité Azure :

https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view

Créez, affichez et gérez des alertes de journal d’activité à l’aide d’Azure Monitor :

https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Contrôle de sécurité : journalisation et supervision](../../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Conseils** : Non applicable. Microsoft conserve la source de temps utilisée pour les ressources Azure, par exemple, Azure Key Vault, pour les timestamps dans les journaux.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : Ingérez les journaux via Azure Monitor pour agréger les données de sécurité générées par Azure Key Vault. Dans Azure Monitor, utilisez un espace de travail Azure Log Analytics pour effectuer des requêtes et des analyses, et servez-vous de comptes de stockage Azure pour le stockage à long terme et l’archivage. Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce. 

Journalisation d’Azure Key Vault :

https://docs.microsoft.com/azure/key-vault/key-vault-logging

Démarrage rapide : Comment intégrer Azure Sentinel :

https://docs.microsoft.com/azure/sentinel/quickstart-onboard


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Activez les paramètres de diagnostic sur vos instances Azure Key Vault pour accéder aux journaux d’audit, de sécurité et de diagnostic. Les journaux d’activité, automatiquement disponibles, incluent la source de l’événement, la date, l’utilisateur, l’horodatage, les adresses sources, les adresses de destination et d’autres éléments utiles.

Journalisation d’Azure Key Vault :

https://docs.microsoft.com/azure/key-vault/key-vault-logging


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Dans Azure Monitor, pour l’espace de travail Log Analytics utilisé pour stocker vos journaux Azure Key Vault, définissez la période de rétention dans le respect des réglementations de conformité de votre organisation. Utilisez les comptes de stockage Azure pour le stockage à long terme/d’archivage.

Modifiez la période de rétention des données : https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : Analysez et supervisez les journaux afin de détecter les comportements anormaux, et examinez régulièrement les résultats pour vos ressources protégées par Azure Key Vault. Utilisez l’espace de travail Log Analytics d’Azure Monitor pour examiner les journaux et effectuer des requêtes sur leurs données. Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce. 

Démarrage rapide : Intégrer Azure Sentinel :

https://docs.microsoft.com/azure/sentinel/quickstart-onboard

Bien démarrer avec Log Analytics dans Azure Monitor :

https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal

Bien démarrer avec les requêtes de journal dans Azure Monitor :

https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer des alertes en cas d’activité anormale

**Conseils** : Dans Azure Security Center, activez la protection avancée contre les menaces (ATP, Advanced Threat Protection) pour Key Vault. Activez les paramètres de diagnostic dans Azure Key Vault et envoyez les journaux à un espace de travail Log Analytics. Intégrez votre espace de travail Log Analytics à Azure Sentinel, car cela fournit une solution de réponse automatisée d’orchestration de sécurité. Cela permet de créer des playbooks (solutions automatisées) utilisables pour corriger des problèmes de sécurité.

Démarrage rapide : Intégrer Azure Sentinel :

https://docs.microsoft.com/azure/sentinel/quickstart-onboard

Gérer et résoudre les alertes de sécurité dans Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts

Répondre à des événements avec des alertes Azure Monitor :

https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-response


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Conseils** : Non applicable. Azure Key Vault ne traite pas et ne produit pas de journaux liés aux logiciels anti-programmes malveillants.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Conseils** : Non applicable. Azure Key Vault ne traite pas et ne produit pas de journaux liés au DNS.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : Conservez un inventaire de vos applications inscrites auprès d’Azure Active Directory, ainsi que tous les comptes d’utilisateur qui ont accès à vos clés, secrets et certificats Azure Key Vault. Vous pouvez utiliser le portail Azure ou PowerShell pour interroger et rapprocher les accès au Key Vault. Pour afficher les accès dans PowerShell, utilisez la commande suivante :

(Get-AzResource -ResourceId [KeyVaultResourceID]).Properties.AccessPolicies

Inscription d’une application auprès d’Azure Active Directory :

https://docs.microsoft.com/azure/key-vault/key-vault-manage-with-cli2#registering-an-application-with-azure-active-directory

Sécuriser l’accès à un coffre de clés :

https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : Non applicable. Azure Key Vault n’intègre pas le concept de mots de passe par défaut, car l’authentification est assurée par Active Directory et sécurisée avec le contrôle d’accès en fonction du rôle Azure (Azure RBAC).


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés ayant accès à vos instances Azure Key Vault. Utilisez la fonctionnalité de gestion des identités et des accès d’Azure Security Center (actuellement en préversion) pour superviser le nombre de comptes d’administration actifs.

Superviser les identités et les accès (préversion) :

https://docs.microsoft.com/azure/security-center/security-center-identity-access


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : Utilisez un principal de service Azure conjointement avec l’AppId, le TenantID et le ClientSecret pour authentifier votre application en toute transparence et récupérer le jeton à utiliser pour accéder à vos secrets Azure Key Vault.

Authentification de service à service auprès d’Azure Key Vault à l’aide de .NET :

https://docs.microsoft.com/azure/key-vault/service-to-service-authentication



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : Activez Azure Active Directory Multi-Factor Authentication et suivez Azure Security Center les recommandations relatives à la gestion des identités et des accès (actuellement en préversion) pour protéger vos ressources Event Hub.

Planifier un déploiement informatique d'Azure AD Multi-Factor Authentication :

https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Superviser les identités et les accès (préversion) :

https://docs.microsoft.com/azure/security-center/security-center-identity-access


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils** : Utilisez une station de travail à accès privilégié sur laquelle le service Azure AD Multi-Factor Authentication (MFA) est configuré pour vous connecter aux ressources Key Vault activées et les configurer. 

Stations de travail disposant d’un accès privilégié : https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations 

Planifier un déploiement informatique d'Azure AD Multi-Factor Authentication : https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration

**Aide** : Utilisez Azure Active Directory (Azure AD) Privileged Identity Management (PIM) pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement. Utilisez les détections de risque Azure AD pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs. Pour une journalisation complémentaire, envoyez les alertes de détection des risques d’Azure Security Center à Azure Monitor et configurez des alertes/notifications personnalisées à l’aide de groupes d’actions.

Activez la protection avancée contre les menaces pour Azure Key Vault de façon à générer des alertes en cas d’activité suspecte.

Déployez Azure AD Privileged Identity Management (PIM) : https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan

Configurez la protection avancée contre les menaces pour Azure Key Vault (préversion) : https://docs.microsoft.com/azure/security-center/advanced-threat-protection-key-vault

Alertes pour Azure Key Vault (préversion) : https://docs.microsoft.com/azure/security-center/alerts-reference#alerts-azurekv

Détections des risques dans Azure Active Directory : https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events

Créez et gérez des groupes d’actions dans le portail Azure : https://docs.microsoft.com/azure/azure-monitor/platform/action-groups



**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Configurez la condition d’emplacement d’une stratégie d’accès conditionnel et gérez vos emplacements nommés. Avec les emplacements nommés, vous pouvez créer des regroupements logiques de plages d’adresses IP ou de pays et régions. Vous pouvez restreindre l’accès à des ressources sensibles, telles que vos secrets Key Vault, à vos emplacements nommés configurés.

Présentation de la condition d’emplacement dans l’accès conditionnel Azure Active Directory : https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations



**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Utilisez Azure Active Directory (Azure AD) comme système central d’authentification et d’autorisation pour des ressources Azure telles que Key Vault. Ainsi, le mécanisme de contrôle d’accès en fonction du rôle Azure (Azure RBAC) est applicable aux ressources administratives sensibles.

 

Démarrage rapide : Créer un locataire dans Azure Active Directory :

https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Examinez les journaux Azure Active Directory (Azure AD) pour découvrir les comptes obsolètes disposant de rôles d’administrateur Azure Key Vault. Utilisez également des révisions d’accès Azure AD pour gérer efficacement les appartenances aux groupes, les accès à des applications métier susceptibles d’être utilisées pour accéder à Azure Key Vault et les attributions de rôles. Il convient d’examiner régulièrement les accès des utilisateurs, par exemple, tous les 90 jours, pour vérifier que seuls les utilisateurs appropriés sont autorisés à accéder.

Documentation sur les rapports et la supervision d’Azure Active Directory :

https://docs.microsoft.com/azure/active-directory/reports-monitoring/

Présentation des révisions d’accès Azure AD :

https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés

**Conseils** : Activez les paramètres de diagnostic pour Azure Key Vault et Azure Active Directory de façon à envoyer tous les journaux à un espace de travail Log Analytics. Configurez les alertes souhaitées (par exemple, les tentatives d’accès aux secrets désactivés) dans Log Analytics.

Intégrez des journaux d’activité Azure AD aux journaux d’activité Azure Monitor : https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

Migration à partir d’une ancienne solution Key Vault : https://docs.microsoft.com/azure/azure-monitor/insights/azure-key-vault#migrating-from-the-old-key-vault-solution



**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Conseils** : Utilisez les fonctionnalités de protection des identités et de détection des risques d’Azure Active Directory pour configurer des réponses automatiques aux actions suspectes détectées en lien avec vos ressources protégées Azure Key Vault. Vous devez activer des réponses automatisées via Azure Sentinel pour implémenter les réponses de sécurité de votre organisation. 

Rapports sur les connexions risquées dans le portail Azure Active Directory : https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins 

Procédure : Configurer et activer des stratégies de gestion des risques : https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

Comment intégrer Azure Sentinel : https://docs.microsoft.com/azure/sentinel/quickstart-onboard


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : Non applicable. Customer Lockbox n’est pas pris en charge pour Azure Key Vault.

Services et scénarios pris en charge généralement disponibles : https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Contrôle de sécurité : protection des données](../../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles sur des ressources Azure Key Vault. 

Utilisez des étiquettes pour organiser vos ressources Azure : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Vous pouvez sécuriser l’accès à Azure Key Vault en utilisant des points de terminaison de service de réseau virtuel configurés pour restreindre l’accès à des sous-réseaux spécifiques.

Une fois les règles de pare-feu appliquées, vous ne pouvez effectuer des opérations de plan de données Azure Key Vault que si votre demande provient de sous-réseaux ou de plages d’adresses IP autorisés. Cela s’applique également à l’accès à Azure Key Vault à partir du portail Azure. Bien que vous puissiez pouvez accéder à un coffre de clés à partir du portail Azure, il se peut que vous ne puissiez pas répertorier les clés, secrets ou certificats qu’il contient si votre ordinateur client ne figure pas dans la liste des ordinateurs autorisés. Cela affecte également le sélecteur Azure Key Vault et d’autres services Azure. Il se peut que vous puissiez afficher des listes de coffres de clés, mais pas de clés, si des règles de pare-feu empêchent votre ordinateur client de le faire.

Configurez des pare-feux et réseaux virtuels Azure Key Vault : https://docs.microsoft.com/azure/key-vault/key-vault-network-security

Points de terminaison de service de réseau virtuel pour Azure Key Vault : https://docs.microsoft.com/azure/key-vault/key-vault-overview-vnet-service-endpoints



**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : Toutes les données stockées dans Azure Key Vault sont considérées comme sensibles. Utilisez contrôles d’accès au plan de données Azure Key Vault pour contrôler l’accès aux secrets Azure Key Vault. Vous pouvez également utiliser le pare-feu intégré d’Azure Key Vault pour contrôler l’accès au niveau de la couche réseau. Pour surveiller l’accès à Azure Key Vault, activez les paramètres de diagnostic de Key Vault et envoyez les journaux à un compte de stockage Azure ou à un espace de travail Log Analytics.

Sécurisez l’accès à un coffre de clés : https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault

Configurez des pare-feux et réseaux virtuels Azure Key Vault : https://docs.microsoft.com/azure/key-vault/key-vault-network-security

Journalisation d’Azure Key Vault : https://docs.microsoft.com/azure/key-vault/key-vault-logging



**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Tout le trafic vers Azure Key Vault pour l’authentification, la gestion et l’accès au plan de données passe par le port HTTPS: 443. Il peut cependant arriver que du trafic transite occasionnellement via le port HTTP 80 pour la liste de révocation de certificats. Azure Key Vault continue à autoriser la réception des données TLS 1.1 et TLS 1.0. Vous pouvez limiter les données au protocole TLS 1.2 en configurant côté client.



Accédez à Azure Key Vault derrière un pare-feu : https://docs.microsoft.com/azure/key-vault/key-vault-access-behind-firewall



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Conseils** : Non applicable. Toutes les données au sein d’Azure Key Vault (secrets, clés et certificats) sont considérées comme sensibles.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Aide** : Sécurisez l’accès à la gestion et au plan de données de vos instances Azure Key Vault.

Sécurisez l’accès à un coffre de clés :

https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Conseils** : Microsoft gère l’infrastructure sous-jacente d’Azure Key Vault et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

Qu'est-ce qu'Azure Key Vault ?

https://docs.microsoft.com/azure/key-vault/key-vault-overview

Protection des données client Azure :

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : Tous les objets managés (clés, certificats et secrets) sont chiffrés au repos dans Azure Key Vault.

Documents associés :

- [Modèle de chiffrement et table de gestion des clés](../../security/fundamentals/encryption-atrest.md)


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : Utilisez la solution Azure Key Vault Analytics dans Azure Monitor pour consulter les journaux des événements d’audit d’Azure Key Vault.

Solution Azure Key Vault Analytics dans Azure Monitor :

https://docs.microsoft.com/azure/azure-monitor/insights/azure-key-vault



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Azure Key Vault.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : Non applicable. Microsoft assure la gestion des correctifs sur les systèmes sous-jacents prenant en charge Key Vault.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3 : Déployer une solution de gestion de correctif logiciel tiers automatisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils** : Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Azure Key Vault.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : Utilisez les évaluations de risques par défaut (Degré de sécurisation) fournies par Azure Security Center.

Améliorez votre degré de sécurisation dans Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-secure-score


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1 : Utiliser la découverte de ressources Azure

**Conseils** : Utilisez Azure Resource Graph pour interroger et découvrir toutes les ressources (y compris les instances Azure Key Vault) au sein de votre abonnement. Vérifiez que vous disposez des autorisations (en lecture) appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent.

Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide de l’Explorateur Azure Resource Graph :

https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Obtenir les abonnements auxquels le compte actuel peut accéder :

https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0

Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (RBAC Azure) ?

https://docs.microsoft.com/azure/role-based-access-control/overview

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des étiquettes aux ressources Azure Key Vault en fournissant des métadonnées pour les organiser de façon logique par catégories.

Créer et utiliser des balises :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des étiquettes, des groupes d’administration, voire des abonnements séparés, pour organiser et suivre les instances Azure Key Vault et les ressources associées. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

Créez un abonnement Azure supplémentaire :

https://docs.microsoft.com/azure/billing/billing-create-subscription

Créez des groupes d’administration pour l’organisation et la gestion des ressources :

https://docs.microsoft.com/azure/governance/management-groups/create

Utilisez des étiquettes pour organiser vos ressources Azure : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4 : Tenir un inventaire des ressources Azure approuvées et titres des logiciels

**Conseils** : Établissez la liste des ressources Azure et des logiciels approuvés pour vos ressources de calcul

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Appliquez des restrictions aux types de ressources pouvant être créées dans des abonnements clients en utilisant les définitions de stratégies Azure intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

Utilisez également Azure Resource Graph pour interroger/découvrir des ressources dans les abonnements.

Tutoriel : Créer et gérer des stratégies pour assurer la conformité : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide de l’Explorateur Azure Resource Graph : https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : Non applicable. Cette recommandation a trait à Azure dans son ensemble, ainsi qu’aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Appliquez des restrictions aux types de ressources pouvant être créées dans des abonnements clients en utilisant les définitions de stratégies Azure intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

Tutoriel : Créer et gérer des stratégies pour assurer la conformité : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Exemples Azure Policy : https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-implement-approved-application-list"></a>6.10 : Implémenter une liste d’applications approuvées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="611-limit-users-ability-to-interact-with-azureresources-manager-via-scripts"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager via des scripts

**Aide** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager (ARM) en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ». Cela peut empêcher la création et la modification de ressources dans un environnement de haute sécurité, comme en cas de configuration d’Azure Key Vault.

Gérer l’accès à la gestion Azure avec l’accès conditionnel :

https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Conseils** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.KeyVault » pour créer des stratégies personnalisées d’audit ou d’application de la configuration de vos instances Azure Key Vault. Vous pouvez également utiliser des définitions Azure Policy intégrées pour Azure Key Vault telles que :

Les objets Key Vault doivent être récupérables

Déployer les paramètres de diagnostic de Key Vault sur l’espace de travail Log Analytics

Les journaux de diagnostic dans Key Vault doivent être activés.

Key Vault doit utiliser un point de terminaison de service de réseau virtuel

Déployer les paramètres de diagnostic de Key Vault sur Event Hub

Utilisez les recommandations d’Azure Security Center comme ligne de base de configuration sécurisée pour vos instances Azure Key Vault.

Affichage des alias Azure Policy disponibles :

https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0

Tutoriel : Créer et gérer des stratégies pour assurer la conformité :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure Key Vault. 

Tutoriel : Créer et gérer des stratégies pour assurer la conformité :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage 

  
Comprendre les effets d’Azure Policy : 

https://docs.microsoft.com/azure/governance/policy/concepts/effects


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous utilisez des définitions Azure Policy personnalisées pour vos ressources Azure Key Vault, utilisez Azure Repos pour stocker et gérer votre code en toute sécurité.

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

**Conseils** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.KeyVault » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

Configurer et gérer Azure Policy :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la surveillance de la configuration automatique pour les services Azure

**Conseils** : Utilisez Azure Security Center pour effectuer des analyses de base de vos ressources protégées par Azure Key Vault. 

  

Comment corriger les recommandations dans Azure Security Center : 

https://docs.microsoft.com/azure/security-center/security-center-remediate-recommendations

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : Non applicable. Ce benchmark a trait aux ressources de calcul.


**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : Utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets pour vos applications Cloud. Assurez-vous que la suppression réversible est activée dans Azure Key Vault.

Intégration aux identités managées Azure :

https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

Pour créer un coffre de clés :

https://docs.microsoft.com/azure/key-vault/general/quick-create-portal

Comment s’authentifier auprès de Key Vault :

https://docs.microsoft.com/azure/key-vault/general/authentication

Comment attribuer une stratégie d’accès Key Vault :

https://docs.microsoft.com/azure/key-vault/general/assign-access-policy-portal

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils** : Utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets pour vos applications Cloud. 

  

* [Intégration aux identités managées Azure](../../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

* [Créer un coffre de clés](quick-create-portal.md)

* [Comment s’authentifier auprès de Key Vault](authentication.md)

* [Comment attribuer une stratégie d’accès Key Vault](assign-access-policy-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.  
  
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

**Conseils** : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure Key Vault), mais il ne s’exécute pas sur du contenu client.

Pré-analysez tout contenu chargé ou envoyé à des ressources Azure autres que de calcul, telles qu’Azure Key Vault. Microsoft ne peut pas accéder à vos données dans ces instances.

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

**Conseils** : Veillez à effectuer des sauvegardes automatisées régulières de vos certificats, clés, comptes de stockage managés et secrets Key Vault avec les commandes PowerShell suivantes :

- Backup-AzKeyVaultCertificate

- Backup-AzKeyVaultKey

- Backup-AzKeyVaultManagedStorageAccount

- Backup-AzKeyVaultSecret

Si vous le souhaitez, vous pouvez stocker vos sauvegardes Key Vault dans Sauvegarde Azure.

Comment sauvegarder des certificats Key Vault : https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultcertificate

Comment sauvegarder des clés Key Vault : https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey

Comment sauvegarder des comptes de stockage managé Key Vault : https://docs.microsoft.com/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount

Comment sauvegarder des secrets Key Vault : https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultsecret

Comment activer Sauvegarde Azure : https://docs.microsoft.com/azure/backup



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Conseils** : Effectuez des sauvegardes automatisées régulières de vos certificats, clés, comptes de stockage managés et secrets Key Vault avec les commandes PowerShell suivantes :

- Backup-AzKeyVaultCertificate

- Backup-AzKeyVaultKey

- Backup-AzKeyVaultManagedStorageAccount

- Backup-AzKeyVaultSecret

Si vous le souhaitez, vous pouvez stocker vos sauvegardes Key Vault dans Sauvegarde Azure.

Comment sauvegarder des certificats Key Vault : https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultcertificate

Comment sauvegarder des clés Key Vault : https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey

Comment sauvegarder des comptes de stockage managé Key Vault : https://docs.microsoft.com/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount

Comment sauvegarder des secrets Key Vault : https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultsecret

Comment activer Sauvegarde Azure : https://docs.microsoft.com/azure/backup



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : Effectuez régulièrement une restauration de données de vos certificats, clés, comptes de stockage managés et secrets Key Vault avec les commandes PowerShell suivantes :

- Restore-AzKeyVaultCertificate

- Restore-AzKeyVaultKey

- Restore-AzKeyVaultManagedStorageAccount

- Restore-AzKeyVaultSecret

Comment restaurer des certificats Key Vault : https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultcertificate?view=azurermps-6.13.0

Comment restaurer des clés Key Vault : https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0 

Comment restaurer des comptes de stockage managé Key Vault : https://docs.microsoft.com/powershell/module/az.keyvault/backup-azkeyvaultmanagedstorageaccount

Comment restaurer des secrets Key Vault : https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultsecret?view=azurermps-6.13.0


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Assurez-vous que la suppression réversible est activée pour Azure Key Vault. La suppression réversible permet de récupérer des coffres de clés et des objets de coffre supprimés, tels que des clés, secrets et autres certificats. 

Comment utiliser la suppression réversible dans Azure Key Vault : 

https://docs.microsoft.com/azure/key-vault/key-vault-soft-delete-powershell

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé. Ces processus doivent se concentrer sur la protection des systèmes sensibles, tels que ceux qui utilisent des secrets Key Vault.

Comment configurer des automations de flux de travail dans Azure Security Center : 

https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide   

Aide sur la création de votre propre processus de réponse aux incidents de sécurité :  

https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/

Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft :   

https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process   

Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents : 

https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte. En outre, marquez clairement les abonnements (par ex. production, non-production) et créez un système d’attribution de noms pour identifier et classer clairement les ressources Azure, en particulier celles qui traitent des données sensibles telles que des secrets Azure Key Vault.


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Effectuez des exercices pour tester les capacités de réponse aux incidents de vos systèmes à intervalles réguliers afin de protéger vos instances Azure Key Vault et les ressources associées. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

Reportez-vous à la publication du NIST : Guide de test, de formation et programmes d’exercice pour les plans informatiques et les fonctionnalités : 

https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Instructions** : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé.  Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

Comment définir le contact de sécurité Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure Key Vault. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue.  Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel. 

 

Comment configurer l’exportation continue : 

https://docs.microsoft.com/azure/security-center/continuous-export 

  

Comment diffuser des alertes dans Azure Sentinel : 

https://docs.microsoft.com/azure/sentinel/connect-azure-security-center

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation de flux de travail disponible dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » aux alertes de sécurité et recommandations afin de protéger vos ressources Azure Key Vault. 

 

Comment configurer l’automatisation du flux de travail et Logic Apps : 

https://docs.microsoft.com/azure/security-center/workflow-automation

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : tests d’intrusion et exercices Red Team](../../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1 : Procédez régulièrement à des tests d’intrusion de vos ressources Azure et veillez à résoudre tous les problèmes de sécurité critiques détectés dans un délai de 60 jours

**Conseils** : Vous ne devez pas effectuer de test d’intrusion directement sur le service Azure Key Vault, mais il est recommandé de tester vos ressources Azure utilisant Key Vault pour garantir la sécurité des secrets.

Vous devez suivre les règles d’engagement de Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft :

https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

Vous trouverez plus d’informations sur la stratégie de Microsoft et l’exécution des tests Red Teaming et de pénétration de sites en direct sur l’infrastructure, les services et les applications cloud gérés par Microsoft ici : 

https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consultez le [benchmark de sécurité Azure](../../security/benchmarks/overview.md)
- Apprenez-en davantage sur les [Bases de référence de la sécurité Azure](../../security/benchmarks/security-baselines-overview.md)