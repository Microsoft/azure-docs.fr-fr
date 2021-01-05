---
title: Base de référence de sécurité Azure pour Event Hubs
description: Base de référence de sécurité Azure pour Event Hubs
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 03/25/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 9386b154148cf5b5b6b7cd0b43f5a4b4aa741059
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96339691"
---
# <a name="azure-security-baseline-for-event-hubs"></a>Base de référence de sécurité Azure pour Event Hubs

La base de référence de sécurité Azure pour Event Hubs contient des recommandations qui vous aideront à améliorer la situation de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1 : Protéger les ressources à l'aide de groupes de sécurité réseau ou du Pare-feu Azure sur votre réseau virtuel

**Aide** : L’intégration de hubs d’événements à des points de terminaison de service de réseau virtuel permet de sécuriser l’accès aux fonctionnalités de messagerie à partir de charges de travail, notamment celles de machines virtuelles liées à des réseaux virtuels. Le chemin du trafic réseau est sécurisé aux deux extrémités.

Une fois lié à au moins un point de terminaison de service de sous-réseau de réseau virtuel, l’espace de noms Event Hubs respectif n’accepte que le trafic provenant de sous-réseaux autorisés dans les réseaux virtuels. Du point de vue du réseau virtuel, la liaison de votre espace de noms Event Hubs à un point de terminaison de service configure un tunnel réseau isolé allant du sous-réseau de réseau virtuel au service de messagerie. 

Vous pouvez également créer un point de terminaison privé, interface réseau qui vous connecte de manière privée et sécurisée au service Azure Event Hubs à l’aide du service Azure Private Link. Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel. Sachant que l’ensemble du trafic à destination du service peut être routé via le point de terminaison privé, il n’y a aucun besoin de passerelles, d’appareils NAT, de connexions ExpressRoute ou VPN ou d’adresses IP publiques. 

Vous pouvez également sécuriser votre espace de noms Azure Event Hubs à l’aide de pare-feu. Azure Event Hubs prend en charge les contrôles d’accès basés sur IP pour le pare-feu entrant. Vous pouvez définir des règles de pare-feu à l’aide du portail Azure, de modèles Azure Resource Manager ou par le biais d’Azure CLI ou d’Azure PowerShell.

Comment utiliser des points de terminaison de service de réseau virtuel avec Azure Event Hubs : https://docs.microsoft.com/azure/event-hubs/event-hubs-service-endpoints

Pour plus d’informations, consultez Intégrer Azure Event Hubs à Azure Private Link : https://docs.microsoft.com/azure/event-hubs/private-link-service.

Activer l’intégration des réseaux virtuels et les pare-feu sur l’espace de noms Event Hubs : https://docs.microsoft.com/azure/event-hubs/event-hubs-tutorial-virtual-networks-firewalls

Comment configurer des règles de pare-feu IP pour des espaces de noms Azure Event Hubs : https://docs.microsoft.com/azure/event-hubs/event-hubs-ip-filtering

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2 : Surveiller et consigner la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Aide** : Utilisez Azure Security Center et suivez les recommandations de protection du réseau pour sécuriser vos ressources Event Hubs dans Azure. Si vous utilisez des machines virtuelles Azure pour accéder à vos hubs d’événements, activez les journaux de flux de groupe de sécurité réseau (NSG) et envoyez les journaux vers un compte de stockage pour auditer le trafic.

Activer les journaux de flux NSG : https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Comprendre la sécurité réseau fournie par Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-network-recommendations

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Conseils** : Activez le service Protection DDoS Standard sur les réseaux virtuels associés à vos hubs d’événements à des fins de protection contre les attaques par déni de service distribué (DDoS). Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

Comment configurer la protection DDoS : [https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection](../ddos-protection/manage-ddos-protection.md)

Pour plus d’informations sur la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-alerts-service-layer

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5 : Consigner les paquets réseau et les journaux de flux

**Aide** : Si vous utilisez des machines virtuelles Azure pour accéder à vos hubs d’événements, activez les journaux de flux de groupe de sécurité réseau (NSG) et envoyez les journaux vers un compte de stockage pour auditer le trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

Si cela s'avère nécessaire pour analyser une activité anormale, activez la capture de paquets Network Watcher.

Activer les journaux de flux NSG : https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Comment activer et utiliser Traffic Analytics : https://docs.microsoft.com/azure/network-watcher/traffic-analytics

Activer Network Watcher : https://docs.microsoft.com/azure/network-watcher/network-watcher-create

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Aide** : Si vous utilisez des machines virtuelles Azure pour accéder à vos hubs d’événements, sélectionnez une offre de la Place de marché Azure qui prend en charge les fonctionnalités IDS/IPS avec des fonctionnalités d’inspection de charge utile. Si la détection et/ou la prévention des intrusions basées sur l’inspection de la charge utile ne sont pas nécessaires pour votre organisation, vous pouvez utiliser la fonctionnalité de pare-feu Azure intégrée d’Azure Event Hubs. Vous pouvez limiter l’accès à votre espace de noms Event Hubs pour une plage limitée d’adresses IP ou pour une adresse IP spécifique à l’aide de règles de pare-feu.

Place de marché Azure :

https://azuremarketplace.microsoft.com/marketplace/?term=Firewall

Comment ajouter une règle de pare-feu dans Event Hubs pour une adresse IP spécifiée :

 https://docs.microsoft.com/azure/event-hubs/event-hubs-ip-filtering

**Supervision d’Azure Security Center** : Pas encore disponible

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Aide** : Non applicable, cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Aide** : Définissez et implémentez des configurations de sécurité standard pour les ressources réseau associées à vos espaces de noms Azure Event Hubs à l’aide d’Azure Policy. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.EventHub » et « Microsoft.Network » pour créer des stratégies personnalisées d’audit ou d’application de la configuration réseau de vos espaces de noms Event Hubs. Vous pouvez également utiliser des définitions de stratégie intégrée en lien avec Azure Event Hubs, par exemple :

- Event Hub doit utiliser un point de terminaison de service de réseau virtuel.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Stratégie intégrée Azure pour l’espace de noms Event Hubs : https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#event-hub



Exemples Azure Policy pour le réseau : https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#network



Créer une instance Azure Blueprint : https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Aide** : Utilisez des étiquettes pour les réseaux virtuels et autres ressources liées à la sécurité réseau et au flux de trafic qui sont associés à vos hubs d’événements.

Comment créer et utiliser des étiquettes : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Aide** : Utilisez le journal d’activité Azure pour superviser les configurations des ressources réseau et détecter les changements des ressources réseau associées à Azure Event Hubs. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.

Consulter et récupérer les événements du journal des activités Azure : https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view

Créer des alertes dans Azure Monitor : https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Contrôle de sécurité : journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Conseils** : Non applicable. Microsoft conserve la source de temps utilisée pour les ressources Azure, par exemple, Azure Event Hubs, pour les timestamps dans les journaux.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Dans Azure Monitor, configurez les journaux liés aux hubs d’événements dans les paramètres de diagnostic Journal d’activité et Event Hub pour envoyer les journaux vers un espace de travail Log Analytics à des fins d’interrogation ou vers un compte de stockage pour un stockage d’archivage à long terme.

Comment configurer les paramètres de diagnostic pour Azure Event Hubs : https://docs.microsoft.com/azure/event-hubs/event-hubs-diagnostic-logs

Présentation du journal d’activité Azure : https://docs.microsoft.com/azure/azure-monitor/platform/platform-logs-overview

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Activez les paramètres de diagnostic pour votre espace de noms Azure Event Hubs. Il existe trois catégories de paramètres de diagnostic pour Azure Event Hubs : journaux d’archivage, Journaux des opérations et journaux de mise à l’échelle automatique. Activez les journaux des opérations pour capturer les informations relatives à ce qui se passe pendant les opérations Event Hubs, en particulier le type d’opération tel que la création d’un hub d’événements, les ressources utilisées et l’état de l’opération.

En outre, vous pouvez activer les paramètres de diagnostic des journaux d’activité Azure et les envoyer vers un compte de stockage Azure, un hub d’événements ou un espace de travail Log Analytics. Les journaux d’activité fournissent des insights sur les opérations qui ont été effectuées sur vos ressources Azure Event Hubs et autres. À l’aide des journaux d’activité, vous pouvez déterminer « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) sur vos espaces de noms Azure Event Hubs.

Comment activer les paramètres de diagnostic pour Azure Event Hubs : https://docs.microsoft.com/azure/event-hubs/event-hubs-diagnostic-logs

Comment activer les paramètres de diagnostic pour le journal d’activité : https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Aide** : Dans Azure Monitor, définissez la période de conservation de votre espace de travail Log Analytics en fonction des obligations réglementaires de votre organisation pour capturer et examiner les incidents liés aux hubs d’événement.

Comment définir les paramètres de conservation des journaux pour les espaces de travail Log Analytics : https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Analysez et supervisez les journaux pour détecter les comportements anormaux et examinez régulièrement les résultats liés à vos hubs d’événements. Utilisez Log Analytics d’Azure Monitor pour examiner les journaux et effectuer des requêtes sur leurs données. Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.
 

Pour plus d’informations sur l’espace de travail Log Analytics, consultez https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal

Comment exécuter des requêtes personnalisées dans Azure Monitor : https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries

Comment intégrer Azure Sentinel : https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer des alertes en cas d’activité anormale

**Aide** : Dans Azure Monitor, configurez les journaux liés à Azure Event Hubs dans les paramètres de diagnostic Journal d’activité et Event Hub pour envoyer les journaux vers un espace de travail Log Analytics à des fins d’interrogation ou vers un compte de stockage pour un stockage d’archivage à long terme. Utilisez l’espace de travail Log Analytics afin de créer des alertes pour les activités anormales détectées dans les événements et journaux de sécurité.

Vous pouvez également activer et intégrer les données dans Azure Sentinel. 

Comprendre le journal d’activité Azure : https://docs.microsoft.com/azure/azure-monitor/platform/platform-logs-overview

Comment configurer les paramètres de diagnostic pour Azure Event Hubs : https://docs.microsoft.com/azure/event-hubs/event-hubs-diagnostic-logs

Comment générer une alerte sur des données de journal de l’espace de travail Log Analytics : https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-response

Comment intégrer Azure Sentinel : https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervision d’Azure Security Center** : Pas encore disponible

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide** : Non applicable. Event Hub ne traite pas la journalisation anti-programme malveillant.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Aide** : Non applicable. Event Hubs ne traite pas et ne produit pas de journaux liés au DNS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Aide** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Azure Active Directory (AD) comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent être interrogés. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration. 

Comment obtenir un rôle d’annuaire dans Azure AD avec PowerShell : https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0

Comment obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell : https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : L’accès au plan de contrôle pour Event Hubs est contrôlé par le biais d’Azure Active Directory (AD). Azure AD n’intègre pas le concept des mots de passe par défaut.

L’accès au plan de données pour Event Hubs est contrôlé par le biais d’Azure AD avec des identités managées ou des inscriptions d’applications ainsi que des signatures d’accès partagé. Les signatures d’accès partagé sont utilisées par les clients qui se connectent à vos hubs d’événements et peuvent être regénérées à tout moment.

Comprendre les signatures d’accès partagé pour Event Hubs : https://docs.microsoft.com/azure/event-hubs/authenticate-shared-access-signature

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

En outre, pour vous aider à suivre les comptes d’administration dédiés, vous pouvez utiliser des recommandations d’Azure Security Center ou des stratégies Azure intégrées, telles que les suivantes :

- Plusieurs propriétaires doivent être affectés à votre abonnement

- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

Comment utiliser Azure Security Center pour superviser l’identité et l’accès (préversion) : https://docs.microsoft.com/azure/security-center/security-center-identity-access

Comment utiliser Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : Microsoft Azure offre la gestion du contrôle d’accès intégré pour les ressources et les applications basées sur Azure Active Directory (AD). L’un des principaux avantages de l’utilisation d’Azure AD avec Azure Event Hubs est que vous n’avez plus besoin de stocker vos informations d’identification dans le code. Au lieu de cela, vous pouvez demander un jeton d’accès OAuth 2.0 à partir de la Plateforme d’identité Microsoft. Le nom de ressource à utiliser pour demander un jeton est https:\//eventhubs.azure.net/. Azure AD authentifie le principal de sécurité (un utilisateur, un groupe ou un principal de service) qui exécute l’application. Si l’authentification réussit, Azure AD retourne un jeton d’accès à l’application et l’application peut ensuite l’utiliser pour autoriser les demandes vers les ressources Azure Event Hubs.

Comment authentifier une application avec Azure AD pour accéder aux ressources Event Hubs : https://docs.microsoft.com/azure/event-hubs/authenticate-application

Comprendre l’authentification unique (SSO) avec Azure AD : https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : Activez Azure Active Directory Multi-Factor Authentication (MFA) et suivez les recommandations Azure Security Center sur la gestion des identités et des accès pour protéger vos ressources Event Hub.

Comment activer l’authentification multifacteur dans Azure : https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Comment surveiller l’identité et l’accès dans Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Aide** : Utilisez des stations de travail disposant d’un accès privilégié avec Multi-Factor Authentication (MFA) configuré pour se connecter aux ressources Event Hub et les configurer.

En savoir plus sur les stations de travail avec accès privilégié : https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations

Comment activer l’authentification multifacteur dans Azure : https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration

**Aide** : Utilisez Azure Active Directory (AD) Privileged Identity Management pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement. Utilisez les détections de risque Azure AD pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs. Pour une journalisation complémentaire, envoyez les alertes de détection des risques d’Azure Security Center à Azure Monitor et configurez des alertes/notifications personnalisées à l’aide de groupes d’actions.

Comment déployer Privileged Identity Management (PIM) : https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan

Comprendre les détections des risques Azure AD : https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events

Comment configurer des groupes d’actions pour générer des alertes et des notifications personnalisées : https://docs.microsoft.com/azure/azure-monitor/platform/action-groups

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Aide** : Utilisez des emplacements nommés à accès conditionnel pour autoriser l’accès uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.



Comment configurer des emplacements nommés dans Azure : https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Utilisez Azure Active Directory (AD) comme système central d’authentification et d’autorisation pour des ressources Azure telles qu’Event Hubs. Ainsi, le mécanisme de contrôle d’accès en fonction du rôle Azure (Azure RBAC) est applicable aux ressources administratives sensibles.

 Comment créer et configurer une instance Azure AD : https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant

Pour en savoir plus sur l’intégration d’Azure Event Hubs à Azure Active Directory (AAD), consultez Autoriser l’accès aux ressources Event Hubs à l’aide d’Azure Active Directory : https://docs.microsoft.com/azure/event-hubs/authorize-access-azure-active-directory

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory (AD) fournit des journaux pour vous aider à découvrir les comptes obsolètes. De plus, utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seules les utilisateurs appropriés continuent de bénéficier d’un accès.

En outre, effectuez une rotation régulière des signatures d’accès partagé de vos hubs d’événements.

Comprendre la génération de rapports Azure AD : https://docs.microsoft.com/azure/active-directory/reports-monitoring/

Comment utiliser les révisions d’accès des identités Azure : https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview

Comprendre les signatures d’accès partagé pour Event Hubs : https://docs.microsoft.com/azure/event-hubs/authenticate-shared-access-signature

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés

**Aide** : Vous avez accès aux activités de connexion Azure Active Directory (AD), aux sources des journaux d’événements à risque et d’audit, ce qui vous permet de les intégrer à un outil SIEM/de supervision.

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure AD et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes de journal souhaitées dans un espace de travail Log Analytics.

Comment intégrer des journaux d’activité Azure à Azure Monitor : https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

Autoriser l’accès aux ressources Event Hubs à l’aide d’Azure Active Directory : https://docs.microsoft.com/azure/event-hubs/authorize-access-azure-active-directory

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez les fonctionnalités de protection des identités et de détection des risques d’Azure Active Directory pour configurer des réponses automatiques aux actions suspectes détectées en lien avec vos ressources Event Hubs. Vous devez activer des réponses automatisées via Azure Sentinel pour implémenter les réponses de sécurité de votre organisation.

Comment afficher les connexions risquées à Azure AD : https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins

Comment configurer et activer des stratégies de protection des identités : https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

Comment intégrer Azure Sentinel : https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : Non disponible ; Customer Lockbox n’est pas encore pris en charge pour Event Hubs.

Liste des services pris en charge par Customer Lockbox : https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : actuellement non disponible

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Contrôle de sécurité : protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Aide** : Utilisez des étiquettes sur des ressources associées à vos hubs d’événements pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

Créer et utiliser des balises : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Les espaces de noms Event Hubs doivent être séparés par un réseau virtuel avec des points de terminaison de service activés et correctement étiquetés.

Vous pouvez également sécuriser votre espace de noms Azure Event Hubs à l’aide de pare-feu. Azure Event Hubs prend en charge les contrôles d’accès basés sur IP pour le pare-feu entrant. Vous pouvez définir des règles de pare-feu à l’aide du portail Azure, de modèles Azure Resource Manager ou par le biais d’Azure CLI ou d’Azure PowerShell.

Créer des abonnements Azure supplémentaires : https://docs.microsoft.com/azure/billing/billing-create-subscription

Créer des groupes d’administration : https://docs.microsoft.com/azure/governance/management-groups/create

Configurer des règles de pare-feu IP pour des espaces de noms Azure Event Hubs : https://docs.microsoft.com/azure/event-hubs/event-hubs-ip-filtering

Comment créer et utiliser des étiquettes : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

Comment créer un réseau virtuel : https://docs.microsoft.com/azure/virtual-network/quick-create-portal

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Aide** : Quand vous utilisez des machines virtuelles pour accéder à vos hubs d’événements, utilisez des réseaux virtuels, des points de terminaison de service, un pare-feu Event Hubs, des groupes de sécurité réseau et des étiquettes de service pour limiter les risques d’exfiltration de données.

Microsoft gère l’infrastructure sous-jacente d’Azure Event Hubs et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

Configurer des règles de pare-feu IP pour des espaces de noms Azure Event Hubs : https://docs.microsoft.com/azure/event-hubs/event-hubs-ip-filtering

Comprendre les points de terminaison de service de réseau virtuel avec Azure Event Hubs : https://docs.microsoft.com/azure/event-hubs/event-hubs-service-endpoints

Intégrer Azure Event Hubs à Azure Private Link : https://docs.microsoft.com/azure/event-hubs/private-link-service

Comprendre les groupes de sécurité réseau et les étiquettes de service : https://docs.microsoft.com/azure/virtual-network/security-overview

Comprendre la protection des données client dans Azure : https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Aide** : Par défaut, Azure Event Hubs impose des communications chiffrées par le protocole TLS. Les versions 1.0, 1.1 et 1.2 de TLS sont actuellement prises en charge. Toutefois, TLS 1.0 et 1.1 étant en passe de dépréciation dans l’ensemble du secteur, utilisez TLS 1.2 dans la mesure du possible.

Pour comprendre les fonctionnalités de sécurité d’Event Hubs, consultez Sécurité réseau : https://docs.microsoft.com/azure/event-hubs/network-security

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Conseils** : Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour Azure Event Hubs. Implémentez une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

Comprendre la protection des données client dans Azure : https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Aide** : Azure Event Hubs prend en charge l’utilisation d’Azure Active Directory (AD) pour autoriser les requêtes de ressources Event Hubs. Azure AD vous permet d’utiliser le contrôle d’accès en fonction du rôle (Azure RBAC) pour accorder des autorisations à un principal de sécurité, qui peut être un utilisateur ou un principal de service d’application.

Comprendre les rôles Azure RBAC et disponibles pour Azure Event Hubs : https://docs.microsoft.com/azure/event-hubs/authorize-access-azure-active-directory

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Aide** : Non applicable. Ces conseils concernent les ressources de calcul.

Microsoft gère l’infrastructure sous-jacente d’Event Hubs et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

Comprendre la protection des données client dans Azure : https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide** : Azure Event Hubs prend en charge le chiffrement des données au repos à l’aide de clés gérées par Microsoft ou de clés gérées par le client. Cette fonctionnalité vous permet de créer, de faire pivoter, de désactiver et de révoquer l'accès aux clés gérées par le client qui sont utilisées pour chiffrer les données Azure Event Hubs au repos.

Comment configurer les clés gérées par le client pour le chiffrement des données Azure Event Hubs : https://docs.microsoft.com/azure/event-hubs/configure-customer-managed-key

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes en cas de modifications sur des instances de production Azure Event Hubs et autres ressources critiques ou associées.

Comment créer des alertes pour les événements du journal d’activité Azure : https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Aide** : Non applicable. Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Event Hubs.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Aide** : Non applicable. Microsoft assure la gestion des correctifs sur les systèmes sous-jacents prenant en charge Event Hubs.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3 : Déployer une solution de gestion de correctif logiciel tiers automatisée

**Aide** : Non applicable. Le benchmark a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Aide** : Non applicable. Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Event Hubs.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Aide** : Non applicable. Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Event Hubs.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1 : Utiliser la découverte de ressources Azure

**Conseils** : Utilisez Azure Resource Graph pour interroger et découvrir toutes les ressources (y compris les espaces de noms Azure Event Hubs) au sein de vos abonnements. Vérifiez que vous disposez des autorisations (en lecture) appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent.

Créer des requêtes avec Azure Resource Graph : https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Afficher vos abonnements Azure : https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0

Comprendre Azure RBAC : https://docs.microsoft.com/azure/role-based-access-control/overview

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

Comment créer et utiliser des étiquettes : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des étiquettes, des groupes d’administration, voire des abonnements séparés, pour organiser et suivre les espaces de noms Azure Event Hubs et les ressources associées. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

Créer des abonnements Azure supplémentaires : https://docs.microsoft.com/azure/billing/billing-create-subscription

Créer des groupes d’administration : https://docs.microsoft.com/azure/governance/management-groups/create

Créer et utiliser des balises : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4 : Tenir un inventaire des ressources Azure approuvées et titres des logiciels

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul et Azure dans son ensemble.

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

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul et Azure dans son ensemble.

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

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Comment refuser un type de ressource spécifique avec Azure Policy : https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-implement-approved-application-list"></a>6.10 : Implémenter une liste d’applications approuvées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="611-divlimit-users-ability-to-interact-with-azure-resource-manager-via-scriptsdiv"></a>6.11 : <div>Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager via des scripts</div>

**Aide** : Configurez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

Configurer l’accès conditionnel pour bloquer l’accès à Azure Resource Manager : https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management

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

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Définissez et implémentez des configurations de sécurité standard pour vos déploiements Azure Event Hubs. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.EventHub » pour créer des stratégies personnalisées d’audit ou d’application de configurations. Vous pouvez également utiliser des définitions de stratégie intégrée pour Azure Event Hubs, par exemple :

- Les journaux de diagnostic dans Event Hub doivent être activés.

- Event Hub doit utiliser un point de terminaison de service de réseau virtuel

Stratégie intégrée Azure pour l’espace de noms Event Hubs : https://docs.microsoft.com/azure/governance/policy/samples/built-in-policies#event-hub

Affichage des alias Azure Policy disponibles : https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Event Hubs. 

Comment configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

 
Pour plus d’informations sur les effets d’Azure Policy : https://docs.microsoft.com/azure/governance/policy/concepts/effects

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Conseils** : Si vous utilisez des définitions Azure Policy personnalisées pour vos ressources Event Hubs ou associées, utilisez Azure Repos pour stocker et gérer votre code de manière sécurisée.

Stocker du code dans Azure DevOps : https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops

Documentation Azure Repos : https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7 : Déployer les outils de gestion de configuration système

**Aide** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.EventHub » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la surveillance de la configuration automatique pour les services Azure

**Aide** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.EventHub » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. Utilisez les stratégies Azure Policy [auditer], [refuser] et [déployer s’il n’existe pas] pour appliquer automatiquement des configurations pour vos déploiements Azure Event Hubs et les ressources associées.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Aide** : Pour les machines virtuelles Azure ou les applications web s’exécutant sur Azure App Service utilisées pour accéder à vos hubs d’événements, utilisez Managed Service Identity conjointement avec Azure Key Vault afin de simplifier et sécuriser la gestion des signatures d’accès partagé pour vos déploiements Azure Event Hubs. Vérifiez que la suppression réversible est activée dans Key Vault.

Authentifier une identité managée avec Azure Active Directory pour accéder aux ressources Event Hubs : https://docs.microsoft.com/azure/event-hubs/authenticate-managed-identity?tabs=latest

Configurer des clés gérées par le client pour Event Hubs : https://docs.microsoft.com/azure/event-hubs/configure-customer-managed-key 

Intégration aux identités managées Azure : https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

Pour créer un coffre de clés : https://docs.microsoft.com/azure/key-vault/general/quick-create-portal

Comment s’authentifier auprès de Key Vault : https://docs.microsoft.com/azure/key-vault/general/authentication

Comment attribuer une stratégie d’accès Key Vault : https://docs.microsoft.com/azure/key-vault/general/assign-access-policy-portal

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils** : Pour les machines virtuelles Azure ou les applications web s’exécutant sur Azure App Service utilisées pour accéder à vos hubs d’événements, utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser Azure Event Hubs. Vérifiez que la suppression réversible est activée dans Key Vault.

Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure Active Directory (AD). Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Azure Key Vault, sans informations d’identification dans votre code.

Authentifier une identité managée avec Azure Active Directory pour accéder aux ressources Event Hubs : https://docs.microsoft.com/azure/event-hubs/authenticate-managed-identity?tabs=latest 

Configurer des clés gérées par le client pour Event Hubs : https://docs.microsoft.com/azure/event-hubs/configure-customer-managed-key 

Configurer des identités managées : https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm

Intégration aux identités managées Azure : https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

Configuration de Credential Scanner : https://secdevtools.azurewebsites.net/helpcredscan.html

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure App Service), mais il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Aide** : Pré-analysez tout contenu chargé sur des ressources Azure non liées au calcul, comme Azure Event Hubs, App Service, Data Lake Storage, Stockage Blob, Azure Database pour PostgreSQL, etc. Microsoft ne peut pas accéder à vos données dans ces instances.

Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure Cache pour Redis), mais il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-recovery"></a>Récupération des données

*Pour plus d’informations, consultez [Contrôle de sécurité : récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Aide** : Configurez la géo-reprise d’activité après sinistre pour Azure Event Hubs. Si tout un centre de données ou une région Azure complète (si aucune zone de disponibilité n’est utilisée) connaît un temps d’arrêt, il est essentiel que le traitement des données puisse continuer dans les autres régions ou centres de données. Pour cette raison, la géo-reprise d’activité après sinistre et la géoréplication sont des fonctionnalités importantes pour les entreprises. Azure Event Hubs prend en charge la géorécupération d’urgence et la géoréplication au niveau de l’espace de noms. 

Comprendre la géo-reprise d’activité après sinistre pour Azure Event Hubs : https://docs.microsoft.com/azure/event-hubs/event-hubs-geo-dr#availability-zones

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Aide** : Azure Event Hubs fournit une fonctionnalité de chiffrement des données au repos avec Azure Storage Service Encryption (Azure SSE). Event Hubs utilise le service Stockage Azure pour stocker les données. Par défaut, toutes les données stockées avec ce service sont chiffrées à l'aide de clés gérées par Microsoft. Si vous utilisez Azure Key Vault pour stocker des clés gérées par le client, veillez à effectuer des sauvegardes automatisées régulières de vos clés.

Veillez à effectuer des sauvegardes automatisées régulières de vos secrets Key Vault avec la commande PowerShell suivante : Backup-AzKeyVaultSecret

Comment configurer les clés gérées par le client pour le chiffrement des données Azure Event Hubs au repos : https://docs.microsoft.com/azure/event-hubs/configure-customer-managed-key

Comment sauvegarder des secrets Key Vault : https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultsecret

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : valider toutes les sauvegardes, y compris les clés gérées par le client

**Aide** : Testez la restauration des clés gérées par le client sauvegardées.

 

Comment restaurer des clés de coffre de clés dans Azure : https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Activez la suppression réversible dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante. Azure Event Hubs nécessite que les fonctionnalités Suppression réversible et Ne pas vider soient configurées pour les clés gérées par le client.

Configurez la suppression réversible pour le compte de stockage Azure servant à capturer les données Event Hubs. Notez que cette fonctionnalité n’est pas encore prise en charge pour Azure Data Lake Storage Gen 2.

Comment activer la suppression réversible dans Key Vault : https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal

Configurer un coffre de clés avec des clés : https://docs.microsoft.com/azure/event-hubs/configure-customer-managed-key

Suppression réversible pour les objets blob de Stockage Azure : https://docs.microsoft.com//azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Vérifiez qu’il existe des plans de réponse aux incidents écrits qui définissent les rôles du personnel, ainsi que les phases de gestion des incidents.

Comment configurer des automatisations de workflow dans Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center affecte un degré de gravité aux alertes pour vous aider à hiérarchiser l’ordre dans lequel vous remédiez à chaque alerte. Ainsi, quand une ressource est compromise, vous pouvez vous en occuper immédiatement. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

**Supervision Azure Security Center** : Oui

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

**Aide** : Respectez les règles d’engagement de Microsoft pour garantir que vos tests d’intrusion sont conformes aux stratégies Microsoft : https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1.
Vous trouverez plus d’informations sur la stratégie de Microsoft et l’exécution de Red Teaming et des tests de pénétration de site actif sur l’infrastructure, les services et les applications cloud managés par Microsoft ici : https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="next-steps"></a>Étapes suivantes

- Consultez le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- Apprenez-en davantage sur les [Bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)