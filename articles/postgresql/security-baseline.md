---
title: Base de référence de sécurité Azure pour Azure Database pour PostgreSQL – Serveur unique
description: La base de référence de sécurité Azure Database pour PostgreSQL – Serveur unique propose des instructions procédurales et des ressources pour l’implémentation des recommandations de sécurité spécifiées dans le Benchmark de sécurité Azure.
author: msmbaldwin
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/02/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 830f58ea11ba84dd2aee81c0352b3ede4aa4929c
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498249"
---
# <a name="azure-security-baseline-for-azure-database-for-postgresql---single-server"></a>Base de référence de sécurité Azure pour Azure Database pour PostgreSQL – Serveur unique

La base de référence de sécurité Azure pour Azure Database pour PostgreSQL – Serveur unique contient des recommandations qui vous aident à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Aide** : Configurer Private Link pour Azure Database pour PostgreSQL avec des points de terminaison privés Private Link vous permet de vous connecter à différents services PaaS dans Azure par le biais d’un point de terminaison privé. Azure Private Link intègre essentiellement les services Azure à votre Réseau virtuel privé. Le trafic entre votre réseau virtuel et l’instance PostgreSQL transite par le réseau principal de Microsoft.

Vous pouvez également utiliser des points de terminaison de service de réseau virtuel pour protéger et limiter l’accès réseau à vos implémentations Azure Database pour PostgreSQL. Les règles de réseau virtuel désignent une fonctionnalité de sécurité de pare-feu qui permet de contrôler si votre serveur Azure Database pour PostgreSQL doit accepter ou non les communications provenant de sous-réseaux spécifiques dans des réseaux virtuels.

Vous pouvez aussi sécuriser votre serveur Azure Database pour PostgreSQL avec des règles de pare-feu. Le pare-feu du serveur empêche tout accès à votre serveur de base de données jusqu’à ce que vous ayez spécifié les ordinateurs qui disposent d’autorisations. Pour configurer votre pare-feu, vous créez des règles de pare-feu qui spécifient les plages d’adresses IP acceptables. Vous pouvez créer des règles de pare-feu au niveau du serveur.

- [Comment configurer une Liaison privée pour Azure Database pour PostgreSQL](howto-configure-privatelink-portal.md)

- [Comment créer et gérer des points de terminaison de service de réseau virtuel et des règles de réseau virtuel dans Azure Database pour PostgreSQL](howto-manage-vnet-using-portal.md)

- [Comment configurer des règles de pare-feu d’Azure Database pour PostgreSQL](howto-manage-firewall-using-portal.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des interfaces réseau

**Aide** : Quand votre instance Azure Database pour PostgreSQL est sécurisée sur un point de terminaison privé, vous pouvez déployer des machines virtuelles dans le même réseau virtuel. Vous pouvez utiliser un groupe de sécurité réseau pour réduire le risque d’exfiltration de données. Activez les journaux de flux NSG et transférez-les vers un compte de stockage pour l'audit du trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

- [Comment configurer une Liaison privée pour Azure Database pour PostgreSQL](howto-configure-privatelink-portal.md)

- [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Guide pratique pour activer et utiliser Traffic Analytics](../network-watcher/traffic-analytics.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications avec des adresses IP connues comme étant malveillantes

**Aide** : Utiliser Advanced Threat Protection pour Azure Database pour PostgreSQL. Advanced Threat Protection détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses de vos bases de données.

Activez la protection DDoS standard sur les réseaux virtuels associés à vos instances Azure Database pour PostgreSQL pour vous protéger contre les attaques DDoS. Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

- [Comment configurer Advanced Threat Protection pour Azure Database pour PostgreSQL](howto-database-threat-protection-portal.md)

- [Guide pratique pour configurer la protection DDoS](../ddos-protection/manage-ddos-protection.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Aide** : Quand votre instance Azure Database pour PostgreSQL est sécurisée sur un point de terminaison privé, vous pouvez déployer des machines virtuelles dans le même réseau virtuel. Vous pouvez ensuite configurer un groupe de sécurité réseau pour réduire le risque d’exfiltration de données. Activez les journaux de flux NSG et transférez-les vers un compte de stockage pour l'audit du trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

- [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Guide pratique pour activer et utiliser Traffic Analytics](../network-watcher/traffic-analytics.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention d’intrusion (IDS/IPS) basés sur le réseau

**Aide** : Utiliser Advanced Threat Protection pour Azure Database pour PostgreSQL. Advanced Threat Protection détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses de vos bases de données.

- [Comment configurer Advanced Threat Protection pour Azure Database pour PostgreSQL](howto-database-threat-protection-portal.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Aide** : Pour les ressources qui doivent accéder à vos instances Azure Database pour PostgreSQL, utilisez des étiquettes de service de réseau virtuel afin de définir des contrôles d’accès réseau sur des groupes de sécurité réseau ou le pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de l’étiquette de service (par exemple SQL.WestUs) dans le champ de source ou de destination approprié d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Remarque : Azure Database pour PostgreSQL utilise l’étiquette de service « Microsoft.Sql ».

- [Pour plus d’informations sur l’utilisation des étiquettes de service](../virtual-network/service-tags-overview.md)

- [Comprendre l’utilisation des étiquettes de service pour Azure Database pour PostgreSQL](concepts-data-access-and-security-vnet.md#terminology-and-description)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour les paramètres réseau et les ressources réseau associées à vos instances Azure Database pour PostgreSQL avec Azure Policy. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.DBforPostgreSQL » et « Microsoft.Network » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de vos instances Azure Database pour PostgreSQL. Vous pouvez aussi utiliser des définitions de stratégie intégrées relatives à la mise en réseau ou à vos instances Azure Database pour PostgreSQL comme :

- DDoS Protection Standard doit être activé

- L’application de la connexion TLS doit être activée pour les serveurs de base de données PostgreSQL

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Exemples Azure Policy pour le réseau](../governance/policy/samples/index.md)

- [Guide pratique pour créer un blueprint Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Aide** : Utilisez des étiquettes pour les ressources liées à la sécurité réseau et au flux de trafic de vos instances Azure Database pour PostgreSQL afin de fournir des métadonnées et une organisation logique.

Utilisez l’une des définitions Azure Policy intégrées en lien avec l’étiquetage comme « Exiger une étiquette et sa valeur » pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Aide** : Utiliser le journal d’activité Azure pour superviser les configurations des ressources réseau et détecter les modifications des ressources réseau associées à vos instances Azure Database pour PostgreSQL. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Aide** : Microsoft conserve la source de temps utilisée pour les ressources Azure, par exemple Azure Database pour PostgreSQL pour les horodatages dans les journaux.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Activer les paramètres de diagnostic, les journaux du serveur et les journaux d’ingestion pour agréger les données de sécurité générées par vos instances Azure Database pour PostgreSQL. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, puis utilisez les comptes de stockage Azure pour le stockage à long terme/d’archivage. Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.

- [Comment configurer et accéder aux journaux du serveur pour Azure Database pour PostgreSQL](howto-configure-server-logs-in-portal.md)

- [Comment configurer et accéder aux journaux d’audit pour Azure Database pour PostgreSQL](concepts-audit.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Activez les paramètres de diagnostic sur vos instances Azure Database pour PostgreSQL afin d’accéder aux journaux d’audit, de sécurité et de ressources. Veillez à activer spécifiquement le journal d’audit de PostgreSQL. Les journaux d’activité, automatiquement disponibles, incluent la source de l’événement, la date, l’utilisateur, l’horodatage, les adresses sources, les adresses de destination et d’autres éléments utiles. Vous pouvez aussi activer les paramètres de diagnostic des journaux d’activité Azure et envoyer les journaux vers le même espace de travail Log Analytics ou le même compte de stockage.

- [Comment configurer et accéder aux journaux du serveur pour Azure Database pour PostgreSQL](howto-configure-server-logs-in-portal.md)

- [Comment configurer et accéder aux journaux d’audit pour Azure Database pour PostgreSQL](concepts-audit.md)

- [Guide pratique pour configurer les paramètres de diagnostic pour le journal d’activité Azure](../azure-monitor/platform/activity-log.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Aide** : Dans Azure Monitor, pour l’espace de travail Log Analytics utilisé pour stocker vos journaux Azure Database pour PostgreSQL, définissez la période de conservation dans le respect des réglementations de conformité de votre organisation. Utilisez les comptes de stockage Azure pour le stockage à long terme/d’archivage.

- [Définir les paramètres de conservation des journaux pour les espaces de travail Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

- [Stockage des journaux des ressources dans un compte de stockage Azure](../azure-monitor/platform/resource-logs.md#send-to-azure-storage)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Analyser et superviser les journaux de vos instances Azure Database pour PostgreSQL à la recherche d’un comportement anormal. Utilisez Log Analytics d’Azure Monitor pour examiner les journaux et effectuer des requêtes sur leurs données. Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [En savoir plus sur Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md)

- [Guide pratique pour effectuer des requêtes personnalisées dans Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils** : Activer Advanced Threat Protection pour Azure Database pour PostgreSQL. Advanced Threat Protection détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses de vos bases de données.

En outre, vous pouvez activer les journaux et les paramètres de diagnostic du serveur pour PostgreSQL, et envoyer les journaux à un espace de travail Log Analytics. Intégrez votre espace de travail Log Analytics à Azure Sentinel, car cela fournit une solution SOAR (Security Orchestration Automated Response). Cela permet de créer des playbooks (solutions automatisées) utilisables pour corriger des problèmes de sécurité.

- [Comment activer Advanced Threat Protection pour Azure Database pour PostgreSQL](howto-database-threat-protection-portal.md)

- [Comment configurer et accéder aux journaux du serveur pour Azure Database pour PostgreSQL](howto-configure-server-logs-in-portal.md)

- [Comment configurer et accéder aux journaux d’audit pour Azure Database pour PostgreSQL](concepts-audit.md)

- [Guide pratique pour configurer les paramètres de diagnostic pour le journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide** : Non applicable. Azure Database pour PostgreSQL ne traite pas et ne produit pas de journaux liés aux logiciels anti-programmes malveillants.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Aide** : Non applicable. Azure Database pour PostgreSQL ne traite pas et ne produit pas de journaux liés au DNS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Conserver un inventaire des comptes d’utilisateur qui ont un accès d’administration au plan de contrôle (par exemple le portail Azure) de vos instances Azure Database pour PostgreSQL. En outre, conservez un inventaire des comptes d’administration qui ont accès au plan de données (dans la base de données elle-même) de vos instances Azure Database pour PostgreSQL. (Quand vous créez un serveur PostgreSQL, vous fournissez les informations d’identification d’un administrateur. Cet administrateur peut être utilisé pour créer d’autres utilisateurs PostgreSQL.)

Azure Database pour PostgreSQL ne prend pas en charge le contrôle d’accès en fonction du rôle intégré, mais vous pouvez créer des rôles personnalisés basés sur des opérations de fournisseurs de ressources spécifiques.

- [Présentation des rôles personnalisés pour un abonnement Azure](../role-based-access-control/custom-roles.md) 

- [Présentation des opérations du fournisseur de ressources Azure Database pour PostgreSQL](../role-based-access-control/resource-provider-operations.md#microsoftdbforpostgresql) 

- [Présentation de la gestion des accès avec Azure Database pour PostgreSQL](concepts-security.md#access-management)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Aide** : Azure Active Directory et Azure Database pour PostgreSQL n’ont pas le concept de mots de passe par défaut.

Lors de la création de la ressource Azure Database pour PostgreSQL elle-même, Azure force la création d’un utilisateur administrateur avec un mot de passe fort. Cependant, une fois l’instance PostgreSQL créée, vous pouvez utiliser le premier compte d’administrateur de serveur que vous avez créé pour créer des utilisateurs supplémentaires et leur accorder un accès d’administration. Lors de la création de ces comptes, veillez à configurer un mot de passe fort et différent pour chaque compte.

- [Comment créer des comptes supplémentaires pour Azure Database pour PostgreSQL](howto-create-users.md)

- [Comment mettre à jour le mot de passe d’administrateur](howto-create-manage-server-portal.md#update-admin-password)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Aide** : Créer des procédures de fonctionnement standard autour de l’utilisation de comptes d’administration dédiés ayant accès à vos instances Azure Database pour PostgreSQL. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration. 

- [Présentation de l’identité et de l’accès Azure Security Center](../security-center/security-center-identity-access.md) 

- [Comprendre comment créer des utilisateurs administrateurs dans Azure Database pour PostgreSQL](howto-create-users.md#the-server-admin-account)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4 : Utiliser l’authentification unique (SSO) Azure Active Directory

**Conseils** : La connexion à Azure Database pour PostgreSQL est prise en charge à la fois avec le nom d’utilisateur/mot de passe configuré directement dans la base de données, et avec une identité Azure Active Directory (AD) et l’utilisation d’un jeton Azure AD pour se connecter. Quand vous utilisez un jeton de Azure AD, différentes méthodes sont prises en charge, comme un utilisateur Azure AD, un groupe Azure AD ou une application Azure AD qui se connecte à la base de données.

Séparément, l’accès au plan de contrôle pour PostgreSQL est disponible via l’API REST et prend en charge l’authentification unique. Pour vous authentifier, définissez l’en-tête d’autorisation pour vos demandes sur un jeton web JSON que vous avez obtenu auprès d’Azure Active Directory.

- [Authentification auprès d’Azure Database pour PostgreSQL avec Azure Active Directory](howto-configure-sign-in-aad-authentication.md)

- [Comprendre l’API REST d’Azure Database pour PostgreSQL](/rest/api/postgresql/)

- [Présentation de l’authentification SSO avec Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Aide** : Activez l’authentification multifacteur (MFA) Azure Active Directory et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center. L’utilisation de jetons Azure AD pour la connexion à votre base de données vous permet d’exiger l’authentification multifacteur pour les connexions à la base de données.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Authentification auprès d’Azure Database pour PostgreSQL avec Azure Active Directory](howto-configure-sign-in-aad-authentication.md)

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3.6 : Utiliser des stations de travail sécurisées et gérées par Azure pour les tâches administratives

**Aide** : Utilisez des stations de travail disposant d’un accès privilégié avec Multi-Factor Authentication (MFA) configuré pour se connecter aux ressources Azure et les configurer.

- [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Aide** : Activer Advanced Threat Protection pour Azure Database pour PostgreSQL de façon à générer des alertes en cas d’activité suspecte.

En outre, vous pouvez utiliser Azure Active Directory (AD) Privileged Identity Management pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.

Utilisez les détections de risque Azure AD pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs.

- [Comment configurer Advanced Threat Protection pour Azure Database pour PostgreSQL](howto-database-threat-protection-portal.md)

- [Déploiement de Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

- [Présentation des détections de risques Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Utilisez des emplacements nommés à accès conditionnel pour autoriser l’accès au portail et à Azure Resource Manager seulement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

- [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Conseils** : Utiliser Azure Active Directory (AD) comme système d’authentification et d’autorisation central. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

Pour la connexion à Azure Database pour PostgreSQL, il est recommandé d’utiliser Azure AD et un jeton Azure AD pour se connecter. Quand vous utilisez un jeton de Azure AD, différentes méthodes sont prises en charge, comme un utilisateur Azure AD, un groupe Azure AD ou une application Azure AD qui se connecte à la base de données.

Les informations d’identification Azure AD peuvent également être utilisées pour l’administration au niveau du plan de gestion (par exemple le portail Azure) pour contrôler les comptes d’administrateur PostgreSQL.

- [Authentification auprès d’Azure Database pour PostgreSQL avec Azure Active Directory](howto-configure-sign-in-aad-authentication.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Passer en revue les journaux Azure Active Directory pour découvrir les comptes obsolètes, qui peuvent inclure ceux ayant des rôles d’administration Azure Database pour PostgreSQL. Utilisez également des révisions d’accès d’identité Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise susceptibles d’être utilisées pour accéder à Azure Database pour PostgreSQL et les attributions de rôles. Il convient d’examiner régulièrement les accès des utilisateurs, par exemple tous les 90 jours, pour vérifier que seuls les utilisateurs appropriés sont autorisés à accéder.

- [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml)

- [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

- [Examen des utilisateurs PostgreSQL et des rôles attribués](https://www.postgresql.org/docs/current/database-roles.html)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Aide** : Activer les paramètres de diagnostic pour Azure Database pour PostgreSQL et Azure Active Directory de façon à envoyer tous les journaux à un espace de travail Log Analytics. Configurez les alertes souhaitées (comme les tentatives d’authentification en échec) dans Log Analytics.

- [Comment configurer et accéder aux journaux du serveur pour Azure Database pour PostgreSQL](howto-configure-server-logs-in-portal.md)

- [Comment configurer et accéder aux journaux d’audit pour Azure Database pour PostgreSQL](concepts-audit.md)

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12 : Alerter en cas d’écart de comportement de connexion à un compte

**Aide** : Activer Advanced Threat Protection pour Azure Database pour PostgreSQL de façon à générer des alertes en cas d’activité suspecte.

Utilisez les fonctionnalités de protection des identités et de détection des risques d’Azure Active Directory (AAD) pour configurer des réponses automatisées aux actions suspectes détectées. Vous pouvez activer des réponses automatisées par le biais d’Azure Sentinel pour implémenter les réponses de sécurité de votre organisation.

Vous pouvez aussi ingérer des journaux dans Azure Sentinel pour approfondir votre examen.

- [Comment configurer Advanced Threat Protection pour Azure Database pour PostgreSQL](howto-database-threat-protection-portal.md)

- [Vue d’ensemble d’Azure AD Identity Protection](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : Actuellement non disponible ; Customer Lockbox n’est pas encore pris en charge pour Azure Database pour PostgreSQL.

- [Liste des services pris en charge pour Customer Lockbox](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Aide** : Utilisez des étiquettes pour faciliter le suivi des instances Azure Database pour PostgreSQL ou des ressources associées qui stockent ou traitent des informations sensibles.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Utilisez une combinaison de Private Link, de points de terminaison de service et/ou de règles de pare-feu pour isoler et limiter l’accès réseau à vos instances Azure Database pour PostgreSQL.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Comment configurer une Liaison privée pour Azure Database pour PostgreSQL](howto-configure-privatelink-portal.md)

- [Comment créer et gérer des points de terminaison de service de réseau virtuel et des règles de réseau virtuel dans Azure Database pour PostgreSQL](howto-manage-vnet-using-portal.md)

- [Comment configurer des règles de pare-feu d’Azure Database pour PostgreSQL](concepts-firewall-rules.md)

**Supervision d’Azure Security Center** : Non disponible

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Aide** : Quand vous utilisez des machines virtuelles Azure pour accéder à des instances Azure Database pour PostgreSQL, utilisez Private Link, les configurations réseau PostgreSQL, les groupes de sécurité réseau et les étiquettes de service pour réduire le risque d’une exfiltration de données.

Microsoft gère l’infrastructure sous-jacente d’Azure Database pour PostgreSQL et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

- [Comment atténuer l’exfiltration de données pour Azure Database pour PostgreSQL](concepts-data-access-and-security-private-link.md)

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Azure Database pour PostgreSQL prend en charge la connexion de votre serveur PostgreSQL aux applications clientes via TLS (Transport Layer Security), anciennement SSL (Secure Sockets Layer). L’application de connexions TLS entre votre serveur de base de données et vos applications clientes vous protège contre les « attaques de l’intercepteur » en chiffrant le flux de données entre le serveur et votre application. Dans le portail Azure, vérifiez que l’option « Appliquer une connexion SSL » est activée par défaut pour toutes vos instances Azure Database pour PostgreSQL.

Les versions TLS actuellement prises en charge pour Azure Database pour PostgreSQL sont TLS 1.0, TLS 1.1, TLS 1.2.

- [Comment configurer le chiffrement en transit pour Azure Database pour PostgreSQL](concepts-ssl-connection-security.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Aide** : Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour Azure Database pour PostgreSQL. Implémentez une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Aide** : Utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour contrôler l’accès au plan de contrôle de base de données Azure Database pour PostgreSQL (par exemple, le portail Azure). Pour l’accès au plan de données (dans la base de données elle-même), utilisez des requêtes SQL pour créer des utilisateurs et configurer des autorisations utilisateur. Azure RBAC ne modifie pas les autorisations utilisateur dans la base de données.

- [Comment configurer Azure RBAC](../role-based-access-control/role-assignments-portal.md)

- [Guide pratique pour configurer l’accès utilisateur avec SQL pour Azure Database pour PostgreSQL](howto-create-users.md)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Aide** : Non applicable. Cette recommandation a trait aux ressources de calcul.

Microsoft gère l’infrastructure sous-jacente d’Azure Database pour PostgreSQL et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : Le service Azure Database pour PostgreSQL utilise le module de chiffrement conforme à la norme FIPS 140-2 pour chiffrer le stockage des données au repos. À l’exception des fichiers temporaires créés durant l’exécution des requêtes, toutes les données, notamment les sauvegardes, sont chiffrées sur le disque. Le service utilise le chiffrement AES 256 bits inclus dans le chiffrement de stockage Azure, et les clés sont gérées par le système. Le chiffrement de stockage est toujours activé et ne peut pas être désactivé.

Le chiffrement des données avec des clés gérées par le client pour Azure Database pour PostgreSQL Single Server permet le scénario BYOK (Bring Your Own Key) pour la protection des données au repos. À ce stade, vous devez demander l’accès pour utiliser cette fonctionnalité. Pour cela, contactez :

AskAzureDBforPostgreSQL@service.microsoft.com.

- [Comprendre le chiffrement au repos pour Azure Database pour PostgreSQL](concepts-security.md)

- [Comprendre le chiffrement au repos pour Azure Database pour PostgreSQL avec des clés gérées par le client](concepts-data-encryption-postgresql.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : Utiliser Azure Monitor avec le journal d’activité Azure pour créer des alertes en cas de modifications sur des instances de production Azure Database pour PostgreSQL et d’autres ressources critiques ou associées.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : suivez les recommandations d’Azure Security Center relatives à la sécurisation de votre Azure Database pour PostgreSQL et des ressources associées.

Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Azure Database pour PostgreSQL.

- [Comprendre les recommandations d’Azure Security Center](../security-center/recommendations-reference.md)

- [Couverture des fonctionnalités pour les services PaaS Azure dans Azure Security Center](../security-center/features-paas.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="53-deploy-automated-patch-management-solution-for-third-party-software-titles"></a>5.3 : Déployer une solution de gestion automatisée des correctifs des logiciels tiers

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Aide** : Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Azure Database pour PostgreSQL.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : utilisez Azure Resource Graph pour interroger et découvrir toutes les ressources (y compris les instances Azure Database pour PostgreSQL) dans vos abonnements. Vérifiez que vous disposez des autorisations (en lecture) appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent.

- [Guide pratique pour créer des requêtes avec Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription)

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Aide** : Appliquer des étiquettes à des instances Azure Database pour PostgreSQL et à d’autres ressources associées en ajoutant des métadonnées pour les organiser logiquement en une taxonomie.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Aide** : Utiliser des étiquettes, des groupes d’administration et des abonnements distincts pour organiser et suivre les instances Azure Database pour PostgreSQL et les ressources associées. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul et Azure dans son ensemble.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

Utilisez également Azure Resource Graph pour interroger/découvrir des ressources dans les abonnements.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

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

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ». Ceci peut empêcher la création et les modifications des ressources dans un environnement de haute sécurité, comme des instances Azure Database pour PostgreSQL contenant des informations sensibles.

- [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

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

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Définir et implémenter des configurations de sécurité standard pour vos instances Azure Database pour PostgreSQL avec Azure Policy. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.DBforPostgreSQL » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de vos instances Azure Database pour PostgreSQL. Vous pouvez aussi utiliser des définitions de stratégie intégrées relatives à vos instances Azure Database pour PostgreSQL comme :

- L’application de la connexion TLS doit être activée pour les serveurs de base de données PostgreSQL

- Les connexions de journal doivent être activées pour les serveurs de bases de données PostgreSQL

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Présentation des effets d’Azure Policy](../governance/policy/concepts/effects.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous utilisez des définitions Azure Policy personnalisées pour vos instances Azure Database pour PostgreSQL et des ressources associées, utilisez Azure Repos pour stocker et gérer votre code de façon sécurisée.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops&preserve-view=true)

- [Documentation Azure Repos](/azure/devops/repos/index?view=azure-devops&preserve-view=true)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Aide** : Utiliser des alias Azure Policy dans l’espace de noms « Microsoft.DBforPostgreSQL » pour créer des stratégies personnalisées afin d’alerter, d’auditer et d’appliquer des configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer des outils de gestion de la configuration pour les systèmes d'exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Aide** : Utiliser des alias Azure Policy dans l’espace de noms « Microsoft.DBforPostgreSQL » pour créer des stratégies personnalisées afin d’alerter, d’auditer et d’appliquer des configurations système. Utilisez les stratégies Azure Policy [auditer], [refuser] et [déployer si elle n’existe pas] pour appliquer automatiquement des configurations pour vos instances Azure Database pour PostgreSQL et les ressources associées.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Aide** : Pour les machines virtuelles Azure ou les applications web s’exécutant sur Azure App Service utilisées pour accéder à vos instances Azure Database pour PostgreSQL, utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets d’Azure Database pour PostgreSQL. Vérifiez que la suppression réversible est activée dans Key Vault.

- [Intégration aux identités managées Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

- [Créer un coffre de clés](../key-vault/general/quick-create-portal.md)

- [Fournir une authentification Key Vault avec une identité managée](../key-vault/general/assign-access-policy-portal.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Aide** : Le serveur Azure Database pour PostgreSQL prend en charge l’authentification Azure Active Directory pour accéder aux bases de données.  Quand vous créez un serveur Azure Database pour PostgreSQL, vous fournissez les informations d’identification pour un utilisateur administrateur. Cet administrateur peut être utilisé pour créer des utilisateurs de base de données supplémentaires.  

Pour les machines virtuelles Azure ou les applications web s’exécutant sur Azure App Service utilisées pour accéder à votre serveur Azure Database pour PostgreSQL, utilisez Managed Service Identity conjointement avec Azure Key Vault pour stocker et récupérer des informations d’identification pour Azure Database pour PostgreSQL. Vérifiez que la suppression réversible est activée dans Key Vault.

Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure Active Directory (AD). Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Key Vault, sans informations d’identification dans votre code.

- [Configurer des identités managées](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

- [Intégration aux identités managées Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure App Service), mais il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Aide** : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple Azure Database pour PostgreSQL), mais il ne s’exécute pas sur du contenu client.

Pré-analysez tout contenu chargé sur des ressources Azure non liées au calcul, comme App Service, Data Lake Storage, Stockage Blob, Azure Database pour PostgreSQL, etc. Microsoft ne peut pas accéder à vos données dans ces instances.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple Azure Database pour PostgreSQL), mais il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

## <a name="data-recovery"></a>Récupération de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : Garantir des sauvegardes automatiques régulières

**Aide** : Azure Database pour PostgreSQL effectue des sauvegardes des fichiers de données et du journal des transactions. Selon la taille de stockage maximale prise en charge, nous prenons en charge des sauvegardes complètes et différentielles (serveurs de stockage de 4 To maximum) ou des sauvegardes d’instantanés (serveurs de stockage jusqu’à 16 To maximum). Celles-ci vous permettent de restaurer un serveur à n’importe quel point dans le temps au sein de votre période de rétention de sauvegarde configurée. La période de rétention de sauvegarde par défaut est de sept jours. Vous pouvez éventuellement la configurer sur 35 jours maximum. Toutes les sauvegardes sont chiffrées à l’aide du chiffrement AES de 256 bits.

- [Comment sauvegarder un serveur dans Azure Database pour PostgreSQL](howto-restore-server-portal.md)

- [Comprendre la configuration initiale d’Azure Database pour PostgreSQL](tutorial-design-database-using-azure-portal.md)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Aide** : Azure Database pour PostgreSQL crée automatiquement des sauvegardes du serveur et les conserve dans un stockage géoredondant ou redondant localement, au choix de l’utilisateur. Les sauvegardes peuvent être utilisées pour restaurer votre serveur à un point dans le temps. La sauvegarde et la restauration sont une partie essentielle de toute stratégie de continuité d’activité, dans la mesure où elles protègent vos données des corruptions et des suppressions accidentelles.

Si vous utilisez Azure Key Vault pour stocker les informations d’identification de vos instances Azure Database pour PostgreSQL, veillez à faire des sauvegardes automatisées régulières de vos clés.

- [Comment sauvegarder un serveur dans Azure Database pour PostgreSQL](howto-restore-server-portal.md)

- [Comment sauvegarder des clés Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Aide** : Dans Azure Database pour PostgreSQL, l’exécution d’une restauration crée un serveur à partir de sauvegardes de serveur d’origine. Deux types de restauration sont disponibles : Restauration à un point dans le temps et géorestauration. La restauration à un point dans le temps est disponible avec l’option de redondance de la sauvegarde et elle crée un serveur dans la même région que votre serveur d’origine. La géorestauration est disponible seulement si vous avez configuré votre serveur pour le stockage géoredondant. Elle vous permet de restaurer votre serveur dans une autre région.

Le délai estimé de récupération dépend de plusieurs facteurs, notamment du nombre total de bases de données à récupérer dans la même région au même moment, de la taille des bases de données, de la taille du journal des transactions et de la bande passante réseau. Le délai de récupération est généralement inférieur à 12 heures.

Testez régulièrement la restauration de vos instances Azure Database pour PostgreSQL.

- [Comment sauvegarder un serveur dans Azure Database pour PostgreSQL](howto-restore-server-portal.md)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Azure Database pour PostgreSQL accepte les sauvegardes complètes, différentielles et du journal des transactions. Celles-ci vous permettent de restaurer un serveur à n’importe quel point dans le temps au sein de votre période de rétention de sauvegarde configurée. La période de rétention de sauvegarde par défaut est de sept jours. Vous pouvez éventuellement la configurer sur 35 jours maximum. Toutes les sauvegardes sont chiffrées à l’aide du chiffrement AES de 256 bits.

- [Comprendre la sauvegarde et la restauration dans Azure Database pour PostgreSQL](concepts-backup.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

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

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Aide** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

- [Reportez-vous à la publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Conseils** : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé.  Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser la sentinelle des alertes.

- [Comment configurer l’exportation continue](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

- [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de Microsoft pour garantir que vos tests d’intrusion sont conformes aux stratégies de Microsoft : https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

- [Vous trouverez ici plus d’informations sur la stratégie de Microsoft, sur l’exécution de Red Teaming et sur les tests d’intrusion de site actif sur l’infrastructure, les services et les applications cloud qui sont managés par Microsoft](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)