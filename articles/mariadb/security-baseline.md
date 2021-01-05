---
title: Base de référence de sécurité Azure pour Azure Database for MariaDB
description: Base de référence de sécurité Azure pour Azure Database for MariaDB
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 03/23/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 043a2481647076abb5a6e242ffa2ea8b3f282c4d
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94637164"
---
# <a name="azure-security-baseline-for-azure-database-for-mariadb"></a>Base de référence de sécurité Azure pour Azure Database for MariaDB

La base de référence de sécurité Azure pour Azure Database for MariaDB contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1 : Protéger les ressources à l'aide de groupes de sécurité réseau ou du Pare-feu Azure sur votre réseau virtuel

**Aide** : Configurer Private Link pour Azure Database for MariaDB avec des points de terminaison privés Private Link vous permet de vous connecter à différents services PaaS dans Azure par le biais d’un point de terminaison privé. Azure Private Link intègre essentiellement les services Azure à votre Réseau virtuel privé. Le trafic entre votre réseau virtuel et l’instance MariaDB transite par le réseau principal de Microsoft.

Vous pouvez également utiliser des points de terminaison de service de réseau virtuel pour protéger et limiter l’accès réseau à vos implémentations Azure Database for MariaDB. Les règles de réseau virtuel désignent une fonctionnalité de sécurité de pare-feu qui permet de contrôler si votre instance Azure Database for MariaDB doit accepter ou non les communications provenant de sous-réseaux spécifiques dans des réseaux virtuels.

Vous pouvez aussi sécuriser votre instance Azure Database for MariaDB avec des règles de pare-feu. Le pare-feu du serveur empêche tout accès à votre serveur de base de données jusqu’à ce que vous ayez spécifié les ordinateurs qui disposent d’autorisations. Pour configurer votre pare-feu, vous créez des règles de pare-feu qui spécifient les plages d’adresses IP acceptables. Vous pouvez créer des règles de pare-feu au niveau du serveur.

Comment configurer Private Link pour Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/howto-configure-privatelink-portal

Comment créer et gérer des points de terminaison de service de réseau virtuel et des règles de réseau virtuel dans le serveur Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/howto-manage-vnet-portal

Comment configurer des règles de pare-feu d’Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/concepts-firewall-rules

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2 : Surveiller et consigner la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Conseils** : Quand votre serveur Azure Database for MariaDB est sécurisée sur un point de terminaison privé, vous pouvez déployer des machines virtuelles dans le même réseau virtuel. Vous pouvez utiliser un groupe de sécurité réseau pour réduire le risque d’exfiltration de données. Activez les journaux de flux NSG et transférez-les vers un compte de stockage pour l'audit du trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

Comment configurer Private Link pour Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/howto-configure-privatelink-portal

Activer les journaux de flux NSG : https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal Comment activer et utiliser Traffic Analytics : https://docs.microsoft.com/azure/network-watcher/traffic-analytics



**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Aide** : Utiliser Advanced Threat Protection pour Azure Database for MariaDB. Advanced Threat Protection détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses de vos bases de données.

Activez la protection DDoS standard sur les réseaux virtuels associés à vos instances Azure Database for MariaDB pour vous protéger contre les attaques DDoS. Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

Comment configurer Advanced Threat Protection pour Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/howto-database-threat-protection-portal

Comment configurer la protection DDoS : https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection



**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5 : Consigner les paquets réseau et les journaux de flux

**Conseils** : Quand votre serveur Azure Database for MariaDB est sécurisée sur un point de terminaison privé, vous pouvez déployer des machines virtuelles dans le même réseau virtuel. Vous pouvez ensuite configurer un groupe de sécurité réseau pour réduire le risque d’exfiltration de données. Activez les journaux de flux NSG et transférez-les vers un compte de stockage pour l'audit du trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

Activer les journaux de flux NSG : https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal Comment activer et utiliser Traffic Analytics : https://docs.microsoft.com/azure/network-watcher/traffic-analytics



**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Aide** : Utiliser Advanced Threat Protection pour Azure Database for MariaDB. Advanced Threat Protection détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses de vos bases de données.
Comment configurer Advanced Threat Protection pour Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/howto-database-threat-protection-portal


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Aide** : Pour les ressources qui doivent accéder à vos instances Azure Database for MariaDB, utilisez des étiquettes de service de réseau virtuel afin de définir des contrôles d’accès réseau sur des groupes de sécurité réseau ou le pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de l’étiquette de service (par exemple SQL.WestUs) dans le champ de source ou de destination approprié d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.
Remarque : Azure Database for MariaDB utilise l’étiquette de service « Microsoft.Sql ».

Pour plus d’informations sur l’utilisation d’étiquettes de service : https://docs.microsoft.com/azure/virtual-network/service-tags-overview Comprendre l’utilisation des étiquettes de service pour Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/concepts-data-access-security-vnet#terminology-and-description



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : Définir et implémenter des configurations de sécurité standard pour les paramètres réseau et les ressources réseau associées à vos instances Azure Database for MariaDB avec Azure Policy. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.DBforMariaDB » et « Microsoft.Network » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de vos instances Azure Database for MariaDB. Vous pouvez aussi utiliser des définitions de stratégie intégrées relatives à la mise en réseau ou à vos instances Azure Database for MariaDB comme :

- DDoS Protection Standard doit être activé

- Le point de terminaison privé doit être activé pour les serveurs MariaDB

- Le serveur MariaDB doit utiliser un point de terminaison de service de réseau virtuel

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Exemples Azure Policy pour le réseau : https://docs.microsoft.com/azure/governance/policy/samples/

Créer une instance Azure Blueprint : https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Aide** : Utilisez des étiquettes pour les ressources liées à la sécurité réseau et au flux de trafic pour vos instances MariaDB afin de fournir des métadonnées et une organisation logique.

Utilisez l’une des définitions Azure Policy intégrée en lien avec l’étiquetage comme, « Exiger une étiquette et sa valeur » pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

Créer et utiliser des balises : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Aide** : Utiliser le journal d’activité Azure pour superviser les configurations des ressources réseau et détecter les modifications des ressources réseau associées à vos instances Azure Database for MariaDB. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.
Comment consulter et récupérer les événements du journal d’activité Azure : https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view Comment créer des alertes dans Azure Monitor : https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Contrôle de sécurité : journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Aide** : Microsoft conserve la source de temps utilisée pour les ressources Azure, par exemple Azure Database for MariaDB pour les horodatages dans les journaux.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Activer les paramètres de diagnostic, les journaux du serveur et les journaux d’ingestion pour agréger les données de sécurité générées par vos instances Azure Database for MariaDB. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, puis utilisez les comptes de stockage Azure pour le stockage à long terme/d’archivage. Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.
Comment configurer et accéder aux journaux du serveur pour Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/concepts-server-logs

Comment configurer et accéder aux journaux d’audit pour Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/howto-configure-audit-logs-portal Comment intégrer Azure Sentinel : https://docs.microsoft.com/azure/sentinel/quickstart-onboard



**Supervision d’Azure Security Center** : Non disponible

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Aide** : Activer les paramètres de diagnostic sur vos instances Azure Database for MariaDB afin d’accéder aux journaux d’audit, de sécurité et de diagnostic. Veillez à activer spécifiquement le journal d’audit de MariaDB. Les journaux d’activité, automatiquement disponibles, incluent la source de l’événement, la date, l’utilisateur, l’horodatage, les adresses sources, les adresses de destination et d’autres éléments utiles. Vous pouvez aussi activer les paramètres de diagnostic des journaux d’activité Azure et envoyer les journaux vers le même espace de travail Log Analytics ou le même compte de stockage.

Comment configurer et accéder aux journaux du serveur pour Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/concepts-server-logs Comment configurer et accéder aux journaux d’audit pour Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/howto-configure-audit-logs-portal Comment configurer les paramètres de diagnostic pour le journal d’activité Azure : https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy



**Supervision d’Azure Security Center** : Non disponible

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Aide** : Dans Azure Monitor, pour l’espace de travail Log Analytics utilisé pour stocker vos journaux Azure Database for MariaDB, définissez la période de conservation dans le respect des réglementations de conformité de votre organisation. Utilisez les comptes de stockage Azure pour le stockage à long terme/d’archivage.
Comment définir les paramètres de conservation des journaux pour les espaces de travail Log Analytics : https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period Stockage des journaux des ressources dans un compte de stockage Azure : https://docs.microsoft.com/azure/azure-monitor/platform/resource-logs-collect-storage



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Analyser et superviser les journaux de vos instances MariaDB à la recherche d’un comportement anormal. Utilisez l’espace de travail Log Analytics d’Azure Monitor pour examiner les journaux et effectuer des requêtes sur leurs données. Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.

Comment intégrer Azure Sentinel : https://docs.microsoft.com/azure/sentinel/quickstart-onboard

Pour plus d’informations sur l’espace de travail Log Analytics, consultez https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal

Comment exécuter des requêtes personnalisées dans Azure Monitor : https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer des alertes en cas d’activité anormale

**Conseils** : Activer Advanced Threat Protection pour MariaDB. La protection Advanced Threat Protection pour Azure Database for MariaDB détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses de vos bases de données.

En outre, vous pouvez activer les journaux et les paramètres de diagnostic du serveur pour MariaDB, et envoyer les journaux à un espace de travail Log Analytics. Intégrez votre espace de travail Log Analytics à Azure Sentinel, car cela fournit une solution SOAR (Security Orchestration Automated Response). Cela permet de créer des playbooks (solutions automatisées) utilisables pour corriger des problèmes de sécurité.

Comment activer Advanced Threat Protection pour MariaDB : https://docs.microsoft.com/azure/mariadb/howto-database-threat-protection-portal

Comment configurer et accéder aux journaux du serveur pour MariaDB : https://docs.microsoft.com/azure/mariadb/concepts-server-logs

Comment configurer et accéder aux journaux d’audit pour MariaDB : https://docs.microsoft.com/azure/mariadb/howto-configure-audit-logs-portal

Comment intégrer Azure Sentinel : https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide** : Non applicable. MariaDB ne traite pas et ne produit pas de journaux liés aux logiciels anti-programmes malveillants.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Aide** : Non applicable. MariaDB ne traite pas ou ne produit pas de journaux liés à DNS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Aide** : Non applicable. Ce benchmark a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Conserver un inventaire des comptes d’utilisateur qui ont un accès d’administration au plan de gestion (Portail Azure/Azure Resource Manager) de vos instances MariaDB. En outre, conservez un inventaire des comptes d’administration qui ont accès au plan de données de vos instances MariaDB. (Quand vous créez le serveur MariaDB, vous fournissez les informations d’identification d’un administrateur. Cet administrateur peut être utilisé pour créer d’autres utilisateurs MariaDB.)

Comprendre la gestion des accès pour MariaDB : https://docs.microsoft.com/azure/mariadb/concepts-security#access-management

Comprendre les rôles Azure intégrés pour les abonnements Azure : https://docs.microsoft.com/azure/role-based-access-control/built-in-roles


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : Azure Active Directory n’intègre pas le concept des mots de passe par défaut.

Lors de la création de la ressource MariaDB elle-même, Azure force la création d’un utilisateur administrateur avec un mot de passe fort. Cependant, une fois l’instance MariaDB créée, vous pouvez utiliser le premier compte d’administrateur de serveur que vous avez créé pour créer des utilisateurs supplémentaires et leur accorder un accès d’administration. Lors de la création de ces comptes, veillez à configurer un mot de passe fort et différent pour chaque compte.

Comment créer des comptes supplémentaires pour MariaDB : https://docs.microsoft.com/azure/mariadb/howto-create-users


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Aide** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés ayant accès à vos instances MariaDB. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

Comprendre l’identité et l’accès dans Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : L’accès du plan de données à MariaDB est contrôlé par les identités stockées dans la base de données et ne prend pas en charge l’authentification unique. L’accès au plan de contrôle pour MariaDB est disponible via l’API REST et prend en charge l’authentification unique. Pour vous authentifier, définissez l’en-tête d’autorisation pour vos demandes sur un jeton web JSON que vous avez obtenu auprès d’Azure Active Directory.

Comprendre l’API REST d’Azure Database for MariaDB : https://docs.microsoft.com/rest/api/mariadb/

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Instructions** : Activez Azure AD MFA et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

Comment activer l’authentification multifacteur dans Azure : https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Comment surveiller l’identité et l’accès dans Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Aide** : Utilisez des stations de travail avec accès privilégié avec l’authentification multifacteur (MFA) configurée pour la connexion aux ressources Azure et leur configuration.

En savoir plus sur les stations de travail avec accès privilégié : https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations

Comment activer l’authentification multifacteur dans Azure : https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration

**Aide** : Activer Advanced Threat Protection pour MariaDB de façon à générer des alertes en cas d’activité suspecte.

En outre, vous pouvez utiliser Azure AD Privileged Identity Management pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement. Utilisez les détections de risque Azure AD pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs.

Comment configurer Advanced Threat Protection pour MariaDB : https://docs.microsoft.com/azure/mariadb/howto-database-threat-protection-portal

Comment déployer Privileged Identity Management (PIM) : https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan

Comprendre les détections de risques Azure AD : https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Utiliser des emplacements nommés à accès conditionnel pour autoriser l’accès seulement depuis des regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions pour limiter l’accès à des ressources Azure comme MariaDB.

Comment configurer des emplacements nommés dans Azure : https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Instructions** : Utilisez Azure Active Directory (AAD) comme système d’authentification et d’autorisation central. AAD protège les données à l’aide d’un chiffrement renforcé pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

L’authentification Azure AD ne peut pas être utilisée pour un accès direct au plan de données MariaDB. Les informations d’identification Azure AD peuvent cependant être utilisées pour l’administration au niveau du plan de gestion (par exemple le portail Azure) pour contrôler les comptes d’administrateur MariaDB.

Comment mettre à jour le mot de passe de l’administrateur pour MariaDB : https://docs.microsoft.com/azure/mariadb/howto-create-manage-server-portal#update-admin-password

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Passer en revue les journaux Azure Active Directory pour découvrir les comptes obsolètes, qui peuvent inclure ceux ayant des rôles d’administration Azure Database for MariaDB. Utilisez également des révisions d’accès d’identité Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise susceptibles d’être utilisées pour accéder à Azure Database for MariaDB et les attributions de rôles. Il convient d’examiner régulièrement les accès des utilisateurs, par exemple tous les 90 jours, pour vérifier que seuls les utilisateurs appropriés sont autorisés à accéder.

Comprendre la génération de rapports Azure AD : https://docs.microsoft.com/azure/active-directory/reports-monitoring/

Comment utiliser les révisions d’accès des identités Azure : https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés

**Aide** : Activer les paramètres de diagnostic pour MariaDB et Azure Active Directory de façon à envoyer tous les journaux à un espace de travail Log Analytics. Configurez les alertes souhaitées (comme les tentatives d’authentification en échec) dans l’espace de travail Log Analytics.

Comment configurer et accéder aux journaux du serveur pour MariaDB : https://docs.microsoft.com/azure/mariadb/concepts-server-logs

Comment configurer et accéder aux journaux d’audit pour MariaDB : https://docs.microsoft.com/azure/mariadb/howto-configure-audit-logs-portal

Comment intégrer les journaux d’activité Azure dans Azure Monitor : https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

**Supervision d’Azure Security Center** : Non disponible

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Conseils** : Activer Advanced Threat Protection pour MariaDB. La protection Advanced Threat Protection pour Azure Database for MariaDB détecte les activités anormales indiquant des tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses de vos bases de données.

Utilisez les fonctionnalités de protection des identités et de détection des risques d’Azure Active Directory (AAD) pour configurer des réponses automatisées aux actions suspectes détectées. Vous pouvez activer des réponses automatisées via Azure Sentinel pour implémenter les réponses de sécurité de votre organisation.

Comment activer Advanced Threat Protection pour MariaDB : https://docs.microsoft.com/azure/mariadb/howto-database-threat-protection-portal

Comment configurer et activer des stratégies de risque Identity Protection : https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

Comment visualiser les connexions à risque à Azure AD : https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins

Comment intégrer Azure Sentinel : https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervision d’Azure Security Center** : Non disponible

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Aide** : Non applicable. Customer Lockbox n’est pas pris en charge pour Azure Database for MariaDB.

Liste des services pris en charge pour Customer Lockbox : https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Contrôle de sécurité : protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Aide** : Utilisez des étiquettes pour faciliter le suivi des instances Azure Database for MariaDB ou des ressources associées qui stockent ou traitent des informations sensibles.

Créer et utiliser des balises : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Utilisez une combinaison de Private Link, de points de terminaison de service et/ou de règles de pare-feu MariaDB pour isoler et limiter l’accès réseau à vos instances MariaDB.

Créer des abonnements Azure supplémentaires : https://docs.microsoft.com/azure/billing/billing-create-subscription

Créer des groupes d’administration : https://docs.microsoft.com/azure/governance/management-groups/create

Comment configurer Private Link pour Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/concepts-data-access-security-private-link

Comment configurer des points de terminaison de service dans Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/howto-manage-vnet-portal

Comment configurer des règles de pare-feu d’Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/concepts-firewall-rules

**Supervision d’Azure Security Center** : Non disponible

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Aide** : Quand vous utilisez des machines virtuelles Azure pour accéder à des instances MariaDB, utilisez Private Link, les configurations réseau MariaDB, les groupes de sécurité réseau et les étiquettes de service pour réduire le risque d’une exfiltration de données.

Microsoft gère l’infrastructure sous-jacente de MariaDB, et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

Comment atténuer l’exfiltration de données pour Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/concepts-data-access-security-private-link

Comprendre la protection des données client dans Azure : https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Aide** : Azure Database for MariaDB prend en charge la connexion de votre serveur Azure Database for MariaDB aux applications clientes via TLS (Transport Layer Security), anciennement SSL (Secure Sockets Layer). L’application de connexions TLS entre votre serveur de base de données et vos applications clientes vous protège contre les « attaques de l’intercepteur » en chiffrant le flux de données entre le serveur et votre application. Dans le portail Azure, vérifiez que l’option « Appliquer une connexion SSL » est activée par défaut pour toutes vos instances MariaDB.

Comment configurer le chiffrement en transit pour MariaDB : https://docs.microsoft.com/azure/mariadb/howto-configure-ssl

**Supervision d’Azure Security Center** : Non disponible

**Responsabilité** : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Aide** : Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour Azure Database for MariaDB. Implémentez une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

Comprendre la protection des données client dans Azure : https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision d’Azure Security Center** : Non disponible

**Responsabilité** : Partagé

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Aide** : Utilisez le contrôle d’accès en fonction du rôle Azure (RBAC Azure) pour contrôler l’accès à la base de données Azure pour le plan de gestion MariaDB (Portail Azure/Azure Resource Manager). Pour l’accès au plan de données (dans la base de données elle-même), utilisez des requêtes SQL pour créer des utilisateurs et configurer des autorisations utilisateur.

Comment configurer Azure RBAC : https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal

Comment configurer l’accès utilisateur avec SQL pour MariaDB : https://docs.microsoft.com/azure/mariadb/howto-create-users

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Conseils** : Non applicable. Ces instructions sont destinées aux ressources de calcul.

Microsoft gère l’infrastructure sous-jacente de MariaDB, et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

Comprendre la protection des données client dans Azure : https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : Le service Azure Database for MariaDB utilise le module de chiffrement conforme à la norme FIPS 140-2 pour chiffrer le stockage des données au repos. À l’exception des fichiers temporaires créés durant l’exécution des requêtes, toutes les données, notamment les sauvegardes, sont chiffrées sur le disque. Le service utilise le chiffrement AES 256 bits inclus dans le chiffrement de stockage Azure, et les clés sont gérées par le système. Le chiffrement de stockage est toujours activé et ne peut pas être désactivé.

Comprendre le chiffrement au repos pour MariaDB : https://docs.microsoft.com/azure/mariadb/concepts-security

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes en cas de modifications sur des instances de production Azure Database for MariaDB et d’autres ressources critiques ou associées.

Comment créer des alertes pour les événements du journal d’activité Azure : https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Aide** : Actuellement non disponible ; Azure Security Center ne prend pas encore en charge l’évaluation des vulnérabilités pour le serveur Azure Database for MariaDB.


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3 : Déployer une solution de gestion de correctif logiciel tiers automatisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge le serveur Azure Database for MariaDB.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1 : Utiliser la découverte de ressources Azure

**Aide** : Utilisez Azure Resource Graph pour interroger et découvrir toutes les ressources (y compris le serveur Azure Database for MariaDB) dans vos abonnements. Vérifiez que vous disposez des autorisations (en lecture) appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent.

Créer des requêtes avec Azure Resource Graph : https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Afficher vos abonnements Azure : https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription

Comprendre Azure RBAC : https://docs.microsoft.com/azure/role-based-access-control/overview

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Aide** : Appliquer des étiquettes au serveur Azure Database for MariaDB et à d’autres ressources associées en ajoutant des métadonnées pour les organiser logiquement en une taxonomie.

Comment créer et utiliser des étiquettes : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Aide** : Utiliser des étiquettes, des groupes d’administration et des abonnements distincts pour organiser et suivre le serveur Azure Database for MariaDB et les ressources associées. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

Créer des abonnements Azure supplémentaires : https://docs.microsoft.com/azure/billing/billing-create-subscription

Créer des groupes d’administration : https://docs.microsoft.com/azure/governance/management-groups/create

Comment créer et utiliser des étiquettes : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4 : Tenir un inventaire des ressources Azure approuvées et titres des logiciels

**Aide** : Non applicable. Cette recommandation concerne les ressources de calcul et Azure dans son ensemble.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

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

**Aide** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :

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

### <a name="611-divlimit-users-ability-to-interact-with-azure-resources-manager-via-scriptsdiv"></a>6.11 : <div>Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager par le biais de scripts</div>

**Conseils** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ». Ceci peut empêcher la création et les modifications des ressources dans un environnement de haute sécurité, comme le serveur Azure Database for MariaDB contenant des informations sensibles.

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

**Conseils** : Définir et implémenter des configurations de sécurité standard pour vos instances Azure Database for MariaDB avec Azure Policy. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.DBforMariaDB » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de vos instances Azure Database for MariaDB. Vous pouvez aussi utiliser des définitions de stratégie intégrées relatives à vos instances Azure Database for MariaDB comme :

- La sauvegarde géoredondante doit être activée pour Azure Database for MariaDB

Affichage des alias Azure Policy disponibles : https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Comprendre les effets d’Azure Policy : https://docs.microsoft.com/azure/governance/policy/concepts/effects



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous utilisez des définitions Azure Policy personnalisées pour vos serveurs Azure Database for MariaDB et les ressources associées, utilisez Azure Repos pour stocker et gérer votre code de façon sécurisée.

[Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops&preserve-view=true)

[Documentation Azure Repos](/azure/devops/repos/index?view=azure-devops&preserve-view=true)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7 : Déployer les outils de gestion de configuration système

**Aide** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.DBforMariaDB » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la surveillance de la configuration automatique pour les services Azure

**Aide** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.DBforMariaDB » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. Utilisez une stratégie Azure Policy [auditer], [refuser] et [déployer s’il n’existe pas] afin d’appliquer automatiquement des configurations pour vos instances Azure Database for MariaDB et les ressources associées.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Aide** : Pour les machines virtuelles Azure ou les applications web s’exécutant sur Azure App Service utilisées pour accéder à vos serveurs Azure Database for MariaDB, utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets d’Azure Database for MariaDB. Vérifiez que la suppression réversible est activée dans Key Vault.

Intégration aux identités managées Azure : https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

Pour créer un coffre de clés : https://docs.microsoft.com/azure/key-vault/general/quick-create-portal

Comment s’authentifier auprès de Key Vault : https://docs.microsoft.com/azure/key-vault/general/authentication

Comment attribuer une stratégie d’accès Key Vault : https://docs.microsoft.com/azure/key-vault/general/assign-access-policy-portal

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Aide** : Azure Database for MariaDB Server ne prend actuellement pas en charge l’authentification Azure Active Directory pour accéder aux bases de données.  Quand vous créez un serveur Azure Database for MariaDB, vous fournissez les informations d’identification d’un utilisateur administrateur. Cet administrateur peut ensuite créer d’autres utilisateurs MariaDB.  

Pour les machines virtuelles Azure ou les applications web s’exécutant sur Azure App Service utilisées pour accéder à votre serveur Azure Database for MariaDB, utilisez Managed Service Identity conjointement avec Azure Key Vault pour stocker et récupérer des informations d’identification pour Azure Database for MariaDB.  Vérifiez que la suppression réversible est activée dans Key Vault.

Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure Active Directory (AD). Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Key Vault, sans informations d’identification dans votre code. Configuration d’identités managées : https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm. Intégration aux identités managées Azure : https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity.



**Supervision d’Azure Security Center** : Non applicable

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

**Aide** : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple le serveur Azure Database for MariaDB), mais il ne s’exécute pas sur du contenu client.

Pré-analysez tout contenu chargé sur des ressources Azure non liées au calcul, comme App Service, Data Lake Storage, Stockage Blob, le serveur Azure Database for MariaDB, etc. Microsoft ne peut pas accéder à vos données dans ces instances.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple le serveur Azure Database for MariaDB), mais il ne s’exécute pas sur du contenu client.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-recovery"></a>Récupération des données

*Pour plus d’informations, consultez [Contrôle de sécurité : récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Aide** : Azure Database for MariaDB accepte les sauvegardes complètes, différentielles et de journal des transactions.  Azure Database for MariaDB crée automatiquement des sauvegardes de serveur et les conserve dans un stockage géoredondant ou redondant localement configuré par l’utilisateur. Les sauvegardes peuvent être utilisées pour restaurer votre serveur à un point dans le temps. La sauvegarde et la restauration sont une partie essentielle de toute stratégie de continuité d’activité, dans la mesure où elles protègent vos données des corruptions et des suppressions accidentelles.  La période de rétention de sauvegarde par défaut est de sept jours. Vous pouvez éventuellement la configurer sur 35 jours maximum. Toutes les sauvegardes sont chiffrées à l’aide du chiffrement AES de 256 bits.

Comprendre les sauvegardes pour MariaDB :  https://docs.microsoft.com/azure/mariadb/concepts-backup

Comprendre la configuration initiale de MariaDB : https://docs.microsoft.com/azure/mariadb/tutorial-design-database-using-portal



**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Aide** : Azure Database for MariaDB crée automatiquement des sauvegardes de serveur et les conserve dans un stockage géoredondant ou redondant localement configuré par l’utilisateur. Les sauvegardes peuvent être utilisées pour restaurer votre serveur à un point dans le temps.  La sauvegarde et la restauration sont une partie essentielle de toute stratégie de continuité d’activité, dans la mesure où elles protègent vos données des corruptions et des suppressions accidentelles.

Si vous utilisez Key Vault pour le chiffrement des données côté client pour les données stockées sur votre serveur MariaDB, veillez à effectuer des sauvegardes automatisées régulières de vos clés.

Comprendre les sauvegardes pour MariaDB :  https://docs.microsoft.com/azure/mariadb/concepts-backup

Comment sauvegarder des clés Key Vault : https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : valider toutes les sauvegardes, y compris les clés gérées par le client

**Aide** : Dans Azure Database for MariaDB, effectuez une restauration à partir de sauvegardes du serveur d’origine pour tester périodiquement les sauvegardes. Deux types de restauration sont disponibles : Restauration à un point dans le temps et géorestauration. La restauration à un point dans le temps est disponible avec l’option de redondance de la sauvegarde et elle crée un serveur dans la même région que votre serveur d’origine. La géorestauration est disponible seulement si vous avez configuré votre serveur pour le stockage géoredondant. Elle vous permet de restaurer votre serveur dans une autre région.

Le délai estimé de récupération dépend de plusieurs facteurs, notamment du nombre total de bases de données à récupérer dans la même région au même moment, de la taille des bases de données, de la taille du journal des transactions et de la bande passante réseau. Le délai de récupération est généralement inférieur à 12 heures.

Comprendre la sauvegarde et la restauration dans Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/concepts-backup#restore


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Azure Database for MariaDB accepte les sauvegardes complètes, différentielles et de journal des transactions. Celles-ci vous permettent de restaurer un serveur à n’importe quel point dans le temps au sein de votre période de rétention de sauvegarde configurée. La période de rétention de sauvegarde par défaut est de sept jours. Vous pouvez éventuellement la configurer sur 35 jours maximum. Toutes les sauvegardes sont chiffrées à l’aide du chiffrement AES de 256 bits.

Comprendre la sauvegarde et la restauration dans Azure Database for MariaDB : https://docs.microsoft.com/azure/mariadb/concepts-backup


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

- Aide sur la création de votre propre processus de réponse aux incidents de sécurité : https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/
- Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft : https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/
- Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents : https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte. 

En outre, marquez clairement les abonnements (par ex. production, non-production) à l’aide d’étiquettes et créez un système de nommage pour identifier et classer clairement les ressources Azure, en particulier celles qui traitent des données sensibles.  Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

- Alertes de sécurité dans Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-alerts-overview

- Utilisez des étiquettes pour organiser vos ressources Azure : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Effectuez des exercices pour tester les capacités de réponse aux incidents de vos systèmes à intervalles réguliers, afin de protéger vos ressources Azure. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

- Reportez-vous à la publication du NIST : « Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities » : https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Instructions** : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

- Comment définir le contact de sécurité d’Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Aide** : Exportez vos alertes et recommandations Azure Security Center en utilisant la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel.

- Comment configurer l’exportation continue : https://docs.microsoft.com/azure/security-center/continuous-export
- Comment diffuser en continu des alertes dans Azure Sentinel : https://docs.microsoft.com/azure/sentinel/connect-azure-security-center

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Aide** : Utilisez la fonctionnalité d’automatisation de flux de travail disponible dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » aux alertes et aux recommandations de sécurité afin de protéger vos ressources Azure.
    

Comment configurer l’automatisation des workflows et Logic Apps : https://docs.microsoft.com/azure/security-center/workflow-automation

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1 : Procédez régulièrement à des tests d’intrusion de vos ressources Azure et veillez à résoudre tous les problèmes de sécurité critiques détectés dans un délai de 60 jours

**Aide** : Suivez les règles d’engagement de Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft :

https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

Vous trouverez plus d’informations sur la stratégie de Microsoft, sur l’exécution de Red Teaming, et sur les tests d’intrusion de site actif sur l’infrastructure, les services et les applications cloud managés par Microsoft ici : https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consultez le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- Apprenez-en davantage sur les [Bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)