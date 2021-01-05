---
title: Base de référence de sécurité Azure pour HDInsight
description: Base de référence de sécurité Azure pour HDInsight
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/09/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: ff156ab401bc940a0f8d2f8e28c77c2b30b2bea3
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92539341"
---
# <a name="azure-security-baseline-for-hdinsight"></a>Base de référence de sécurité Azure pour HDInsight

La ligne de base de sécurité Azure pour HDInsight contient des recommandations qui permettent d’améliorer l’état de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1 : Protéger les ressources à l'aide de groupes de sécurité réseau ou du Pare-feu Azure sur votre réseau virtuel

**Instructions**  : La sécurité du périmètre dans Azure HDInsight est obtenue par le biais de réseaux virtuels. Un administrateur d’entreprise peut créer un cluster dans un réseau virtuel, et utiliser un groupe de sécurité réseau (NSG) pour restreindre l’accès au réseau virtuel. Seules les adresses IP autorisées dans les règles de groupe de sécurité réseau de trafic entrant sont en mesure de communiquer avec le cluster Azure HDInsight. Cette configuration assure la sécurité du périmètre. Tous les clusters déployés sur un réseau virtuel auront aussi un point de terminaison privé qui se résout en une adresse IP privée dans le réseau virtuel, pour un accès HTTP privé aux passerelles de cluster.

Pour réduire le risque de perte de données via l’exfiltration, limitez le trafic réseau sortant pour les clusters Azure HDInsight à l’aide du pare-feu Azure.

Comment déployer Azure HDInsight dans un réseau virtuel et utiliser un groupe de sécurité réseau pour garantir la sécurité : https://docs.microsoft.com/azure/hdinsight/hdinsight-create-virtual-network

Comment limiter le trafic sortant pour les clusters Azure HDInsight avec le pare-feu Azure : https://docs.microsoft.com/azure/hdinsight/hdinsight-restrict-outbound-traffic

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2 : Surveiller et consigner la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Instructions**  : Utilisez Azure Security Center et appliquez les recommandations de protection réseau pour le réseau virtuel, le sous-réseau et le groupe de sécurité réseau utilisés pour sécuriser votre cluster Azure HDInsight. Activez les journaux de flux de groupe de sécurité réseau (NSG) et envoyez les journaux dans un compte de Stockage Azure à l’audit du trafic. Vous pouvez également envoyer des journaux de flux de groupe de sécurité réseau (NSG) à un espace de travail Azure Log Analytics et utiliser Azure Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages d’Azure Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

Activer les journaux de flux NSG :

https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Comment activer et utiliser Azure Traffic Analytics :

https://docs.microsoft.com/azure/network-watcher/traffic-analytics

Comprendre la sécurité réseau fournie par Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-network-recommendations

**Supervision Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Aide**  : Non applicable. Le test d’évaluation est destiné à Azure Apps Service ou aux ressources de calcul hébergeant des applications web.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Instructions**  : Pour obtenir des protections contre les attaques DDoS, activez la protection Azure DDoS standard sur le réseau virtuel sur lequel votre instance Azure HDInsight est déployée. Utilisez la fonctionnalité Threat Intelligence intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

Configurer la protection DDoS :

https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection

Comprendre la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-alerts-service-layer

**Supervision Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5 : Consigner les paquets réseau et les journaux de flux

**Instructions**  : Activez les journaux de flux de groupe de sécurité réseau (NSG) pour le groupe de sécurité réseau attaché au sous-réseau utilisé pour protéger votre cluster Azure HDInsight. Enregistrez les journaux de flux NSG dans un compte de Stockage Azure pour générer des enregistrements de flux. Si cela s’avère nécessaire pour analyser une activité anormale, activez la capture de paquets Azure Network Watcher.

Activer les journaux de flux NSG :

https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Activer Network Watcher :

https://docs.microsoft.com/azure/network-watcher/network-watcher-create

**Supervision Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Instructions**  : Cette condition peut être respectée avec l’ID de contrôle de sécurité Azure 1.1. Déployez un cluster Azure HDInsight dans un réseau virtuel et sécurisez-le avec un groupe de sécurité réseau (NSG).

Plusieurs dépendances pour Azure HDInsight exigent un trafic entrant. Le trafic de gestion entrant ne peut pas être envoyé via un dispositif de pare-feu. Les adresses sources pour le trafic de gestion nécessaire sont connues et publiées. Créez des règles de groupe de sécurité réseau avec ces informations pour autoriser le trafic à partir d’emplacements approuvés uniquement, en sécurisant le trafic entrant vers les clusters.

Pour réduire le risque de perte de données via l’exfiltration, limitez le trafic réseau sortant pour les clusters Azure HDInsight à l’aide du pare-feu Azure.

Comment déployer HDInsight dans un réseau virtuel et utiliser un groupe de sécurité réseau pour garantir la sécurité : https://docs.microsoft.com/azure/hdinsight/hdinsight-create-virtual-network

Comprendre les dépendances HDInsight et l’utilisation du pare-feu : https://docs.microsoft.com/azure/hdinsight/hdinsight-restrict-outbound-traffic

Adresses IP de gestion HDInsight : https://docs.microsoft.com/azure/hdinsight/hdinsight-management-ip-addresses

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Aide**  : Non applicable. Le test d’évaluation est destiné à Azure Apps Service ou aux ressources de calcul hébergeant des applications web.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Instructions**  : Utilisez les étiquettes de service du réseau virtuel pour définir les contrôles d’accès réseau sur les groupes de sécurité réseau (NSG) attachés au sous-réseau dans lequel votre cluster Azure HDInsight est déployé. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de la balise de service (par exemple, ApiManagement) dans le champ Source ou Destination approprié d'une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Comprendre et utiliser des étiquettes de service pour Azure HDInsight :

https://docs.microsoft.com/azure/virtual-network/security-overview#service-tags

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Instructions**  : Définissez et implémentez des configurations de sécurité standard pour les ressources réseau liées à votre cluster Azure HDInsight. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.HDInsight » et « Microsoft.Network » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de votre cluster Azure HDInsight.

Vous pouvez également utiliser Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les principaux artefacts d’environnement (par exemple, les modèles Resource Manager, les contrôles Azure RBAC et les stratégies) dans une même définition de blueprint. Appliquez facilement le blueprint aux nouveaux abonnements et environnements, et ajustez le contrôle et la gestion par le biais du versioning.

Affichage des alias Azure Policy disponibles :

https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias Guide pratique pour configurer et gérer Azure Policy :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Créer une instance d'Azure Blueprint :

https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Instructions**  : Utilisez des étiquettes pour les groupes de sécurité réseau et autres ressources liées à la sécurité réseau et au flux de trafic qui sont associés à votre cluster Azure HDInsight. Concernant les règles NSG individuelles, utilisez le champ « Description » afin de spécifier le besoin métier et/ou la durée (etc.) pour toutes les règles qui autorisent le trafic vers/depuis un réseau.

Utilisez l’une des définitions de stratégie Azure intégrée en lien avec l’étiquetage comme « Exiger une étiquette et sa valeur » pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Vous pouvez utiliser Azure PowerShell ou l’interface de ligne de commande (CLI) Azure pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

Créer et utiliser des balises :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

Créer un réseau virtuel :

https://docs.microsoft.com/azure/virtual-network/quick-create-portal

Créer un groupe de sécurité réseau (NSG) avec une configuration de sécurité :

https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Instructions**  : Utilisez le journal d’activité Azure pour superviser les configurations des ressources réseau et détecter les changements des ressources réseau associées à vos déploiements Azure HDInsight. Dans Azure Monitor, créez des alertes qui se déclenchent quand des changements sont apportés à des ressources réseau critiques.

Comment consulter et récupérer les événements du journal d’activité Azure :

https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view

Comment créer des alertes dans Azure Monitor : https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Contrôle de sécurité : journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Instructions**  : Microsoft gère les sources de temps pour les composants de cluster Azure HDInsight. Vous pouvez mettre à jour la synchronisation de l’heure pour vos déploiements de calcul.

Comment configurer la synchronisation date/heure pour des ressources de calcul Azure :

https://docs.microsoft.com/azure/virtual-machines/windows/time-sync

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Instructions**  : Vous pouvez intégrer votre cluster Azure HDInsight à Azure Monitor pour agréger les données de sécurité générées par le cluster. Utilisez des requêtes personnalisées pour détecter les menaces dans l’environnement et y répondre. 

Comment intégrer un cluster Azure HDInsight à Azure Monitor :

https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-oms-log-analytics-tutorial

Comment créer des requêtes personnalisées pour un cluster Azure HDInsight :

https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-oms-log-analytics-use-queries

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Instructions**  : Activez Azure Monitor pour le cluster HDInsight, puis dirigez-le vers un espace de travail Log Analytics. Cela permet de journaliser des informations de cluster pertinentes et des métriques de système d’exploitation pour tous les nœuds de cluster Azure HDInsight.

Comment activer la journalisation pour un cluster HDInsight :

 https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-oms-log-analytics-tutorial

Comment interroger des journaux HDInsight :

https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-oms-log-analytics-use-queries

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Instructions**  : Intégrez un cluster Azure HDInsight à Azure Monitor. Vérifiez que la période de conservation des journaux définie dans l’espace de travail Log Analytics utilisé est conforme aux obligations réglementaires de votre organisation.

Comment intégrer un cluster Azure HDInsight à Azure Monitor :

https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-oms-log-analytics-tutorial

Comment configurer la période de conservation d’un espace de travail Log Analytics :

https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Instructions**  : Intégrez un cluster Azure HDInsight à Azure Monitor. Vérifiez que la période de conservation des journaux définie dans l’espace de travail Azure Log Analytics utilisé est conforme aux obligations réglementaires de votre organisation.

Comment intégrer un cluster Azure HDInsight à Azure Monitor :

https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-oms-log-analytics-tutorial

Comment configurer la période de conservation d’un espace de travail Log Analytics :

https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Instructions**  : Utilisez des requêtes d’espace de travail Log Analytics pour interroger des journaux Azure HDInsight :

Comment créer des requêtes personnalisées pour les clusters Azure HDInsight :

https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-oms-log-analytics-use-queries

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer des alertes en cas d’activité anormale

**Instructions**  : Utilisez l’espace de travail Azure Log Analytics pour superviser les activités anormales dans les événements et les journaux de sécurité liés à votre cluster Azure HDInsight, et générer des alertes s’y rapportant.

Comment gérer les alertes dans Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts

Comment générer une alerte sur des données de journal Log Analytics :

https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-response

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Instructions**  : Azure HDInsight est fourni avec Clamscan préinstallé et activé pour les images de nœud de cluster. Toutefois, vous devez gérer le logiciel et agréger/superviser manuellement tous les journaux générés par Clamscan.

Comprendre Clamscan :

https://docs.microsoft.com/azure/hdinsight/hdinsight-faq#security-and-certificates

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Instructions**  : Implémentez une solution tierce pour la journalisation DNS.

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Instructions**  : Configurez manuellement la journalisation de console au niveau de chaque nœud.

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Instructions**  : Tenez à jour l’enregistrement du compte d’administration local créé lors du provisionnement du cluster Azure HDInsight ainsi que des autres comptes que vous créez. De plus, si l’intégration Azure AD est utilisée, Azure AD comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent donc être interrogés. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes qui sont membres de groupes d’administration.

Vous pouvez également suivre les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

Comment obtenir un rôle d’annuaire dans Azure AD avec PowerShell :

https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole

Comment obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell :

https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember

Comment superviser les identités et les accès avec Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervision Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Instructions**  : Lors du provisionnement d’un cluster, Azure vous oblige à créer des mots de passe pour l’accès au portail web et l’accès SSH (Secure Shell). Il n’y a pas de mot de passe par défaut à changer. Toutefois, vous pouvez spécifier des mots de passe différents pour l’accès SSH et l’accès au le portail web.

Comment définir des mots de passe lors du provisionnement d’un cluster Azure HDInsight :

https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-linux-use-ssh-unix

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Instructions**  : Intégrez l’authentification pour un cluster HDInsight avec Azure Active Directory. Créez des stratégies et des procédures concernant l’utilisation de comptes d’administration dédiés.

Vous pouvez également suivre les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

Comment intégrer l’authentification Azure HDInsight avec Azure Active Directory :

https://docs.microsoft.com/azure/hdinsight/domain-joined/apache-domain-joined-configure-using-azure-adds

Comment superviser les identités et les accès avec Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervision Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Instructions**  : Utilisez le broker d’ID Azure HDInsight pour vous connecter à des clusters du Pack Sécurité Entreprise (ESP) à l’aide de Multi-Factor Authentication, sans fournir de mot de passe. Si vous êtes déjà connecté à d’autres services Azure, comme le portail Azure, vous pouvez vous connecter à votre cluster Azure HDInsight avec une expérience d’authentification unique (SSO).

Comment activer le broker d’ID Azure HDInsight  :

https://docs.microsoft.com/azure/hdinsight/domain-joined/identity-broker#enable-hdinsight-id-broker

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Instructions**  : Activez Azure AD MFA et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center. Comme les clusters Azure HDInsight pour lesquels le Pack Sécurité Entreprise est configuré peuvent être connectés à un domaine, les utilisateurs du domaine peuvent utiliser leurs informations d’identification de domaine pour s’authentifier auprès des clusters et exécuter des tâches de Big Data. Quand vous vous authentifiez avec l’authentification multifacteur (MFA) activée, les utilisateurs sont tenus de fournir un deuxième facteur d’authentification.

Comment activer l’authentification multifacteur dans Azure :

https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Comment surveiller l’identité et l’accès dans Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervision Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Instructions**  : Utilisez des stations de travail à accès privilégié avec l’authentification multifacteur (MFA) configurée pour la connexion à vos clusters Azure HDInsight et aux ressources qui leur sont associées, ainsi que pour leur configuration.

En savoir plus sur les stations de travail avec accès privilégié :

https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations

Comment activer l’authentification multifacteur dans Azure :

https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

**Supervision Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration

**Instructions**  : Comme les clusters Azure HDInsight pour lesquels le Pack Sécurité Entreprise est configuré peuvent être connectés à un domaine, les utilisateurs du domaine peuvent utiliser leurs informations d’identification de domaine pour s’authentifier. Vous pouvez utiliser des rapports de sécurité Azure Active Directory (AAD) pour générer des journaux et des alertes quand des activités suspectes ou non sécurisées se produisent dans l’environnement AAD. Utiliser Azure Security Center pour superviser les activités liées aux identités et aux accès.

Comment identifier les utilisateurs AAD marqués d’un indicateur d’activité à risque :

https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-user-at-risk

Comment superviser les activités liées aux identités et aux accès des utilisateurs dans Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Instructions**  : Comme les clusters Azure HDInsight pour lesquels le Pack Sécurité Entreprise est configuré peuvent être connectés à un domaine, les utilisateurs du domaine peuvent utiliser leurs informations d’identification de domaine pour s’authentifier. Utilisez des emplacements nommés à accès conditionnel pour autoriser l’accès uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

Comment configurer des emplacements nommés dans Azure :

https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Instructions**  : Utilisez Azure Active Directory (AAD) comme système d’authentification et d’autorisation central. AAD protège les données à l’aide d’un chiffrement renforcé pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

Comme les clusters Azure HDInsight pour lesquels le Pack Sécurité Entreprise (ESP) est configuré peuvent être connectés à un domaine, les utilisateurs du domaine peuvent utiliser leurs informations d’identification de domaine pour s’authentifier auprès des clusters.

Comment créer et configurer une instance AAD :

https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant

Comment configurer un Pack Sécurité Entreprise avec Azure Active Directory Domain Services dans Azure HDInsight :

https://docs.microsoft.com/azure/hdinsight/domain-joined/apache-domain-joined-configure-using-azure-adds

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Instructions**  : Utilisez l’authentification Azure Active Directory (AAD) avec votre cluster Azure HDInsight. AAD fournit des journaux pour vous permettre de découvrir les comptes obsolètes. De plus, utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès de l’utilisateur peut être passé en revue régulièrement pour vérifier que seuls les utilisateurs appropriés bénéficient d’un accès permanent. 

Comment utiliser les révisions d’accès des identités Azure :

https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés

**Instructions**  : Utilisez la connexion Azure Active Directory (AAD) et les journaux d’audit pour superviser les tentatives d’accès à des comptes désactivés. Ces journaux peuvent être intégrés à n’importe quel outil SIEM/de supervision tiers.

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur AAD, et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Azure Log Analytics. Configurez les alertes souhaitées dans un espace de travail Azure Log Analytics.

Comment intégrer des journaux d’activité Azure dans Azure Monitor :

https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Instructions**  : Comme les clusters Azure HDInsight pour lesquels le Pack Sécurité Entreprise (ESP) est configuré peuvent être connectés à un domaine, les utilisateurs du domaine peuvent utiliser leurs informations d’identification de domaine pour s’authentifier auprès des clusters.  Utilisez les détections de risques et la fonctionnalité Identity Risk d’Azure Active Directory (AAD) pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. De plus, vous pouvez ingérer des données dans Azure Sentinel pour approfondir votre examen.

Comment examiner les connexions risquées à AAD :

https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins

Comment configurer et activer des stratégies de protection des identités :

https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Instructions**  : Non disponible. Customer Lockbox n’est actuellement pas pris en charge pour Azure HDInsight.

Liste des services pris en charge pour Customer Lockbox : https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability



**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Contrôle de sécurité : protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Instructions**  : Utilisez des étiquettes sur des ressources associées à vos déploiements Azure HDInsight pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

Comment créer et utiliser des étiquettes :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils**  : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Les clusters Azure HDInsight et tous les comptes de stockage associés doivent être séparés par un réseau/sous-réseau virtuel, étiquetés de manière appropriée et sécurisés dans un groupe de sécurité réseau (NSG) ou un pare-feu Azure. Les données de cluster doivent être contenues dans un compte de Stockage Azure sécurisé ou Azure Data Lake Storage (Gen1 ou Gen2).

Choisir les options de stockage pour votre cluster Azure HDInsight :

https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-compare-storage-options

Comment sécuriser Azure Data Lake Storage :

https://docs.microsoft.com/azure/data-lake-store/data-lake-store-security-overview

Comment sécuriser des comptes de Stockage Azure :

https://docs.microsoft.com/azure/storage/common/storage-security-guide

**Supervision Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Instructions**  : Pour les clusters Azure HDInsight qui stockent ou traitent des informations sensibles, marquez le cluster et les ressources associées comme sensibles à l’utilisation d’étiquettes. Pour réduire le risque de perte de données via l’exfiltration, limitez le trafic réseau sortant pour les clusters Azure HDInsight à l’aide du pare-feu Azure.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et fait de grands efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

Comment limiter le trafic sortant pour les clusters Azure HDInsight avec le pare-feu Azure :

https://docs.microsoft.com/azure/hdinsight/hdinsight-restrict-outbound-traffic

Comprendre la protection des données client dans Azure :

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Instructions**  : Chiffrer toutes les informations sensibles en transit. Vérifiez que les clients qui se connectent à votre cluster Azure HDInsight ou à vos magasins de données de cluster (comptes de Stockage Azure ou Azure Data Lake Storage Gen1/Gen2) peuvent négocier TLS 1.2 ou une version ultérieure. Par défaut, les ressources Microsoft Azure négocient TLS 1.2. 

Comprendre le chiffrement Azure Data Lake Storage en transit :

https://docs.microsoft.com/azure/data-lake-store/data-lake-store-security-overview

Comprendre le chiffrement de compte de Stockage Azure en transit :

https://docs.microsoft.com/azure/storage/common/storage-security-guide#encryption-in-transit

**Supervision Azure Security Center**  : Oui

**Responsabilité**  : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Instructions**  : Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour le Stockage Azure ou les ressources de calcul. Implémentez une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour un ensemble robuste de contrôles et de fonctionnalités de protection des données.

Comprendre la protection des données client dans Azure :

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Partagé

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Instructions**  : Avec le Pack Sécurité Entreprise (ESP) Azure HDInsight, vous pouvez utiliser Apache Ranger pour créer et gérer des stratégies de contrôle d’accès et d’obfuscation des données affinées pour vos données stockées dans des fichiers, des dossiers, des bases de données, des tables et des lignes/colonnes. L’administrateur Hadoop peut configurer le contrôle d’accès en fonction du rôle (RBAC) pour sécuriser Apache Hive, HBase, Kafka et Spark à l’aide de ces plug-ins dans Apache Ranger.

La configuration de stratégies RBAC avec Apache Ranger vous permet d’associer des autorisations à un rôle dans l’organisation. Cette couche d’abstraction permet de s’assurer plus facilement que les utilisateurs disposent uniquement des autorisations nécessaires pour effectuer leur travail.

Configurations du Pack Sécurité Entreprise avec Azure Active Directory Domain Services dans HDInsight :

https://docs.microsoft.com/azure/hdinsight/domain-joined/apache-domain-joined-configure-using-azure-adds

Vue d’ensemble de la sécurité d’entreprise dans Azure HDInsight :

https://docs.microsoft.com/azure/hdinsight/domain-joined/hdinsight-security-overview



**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Instructions**  : Pour les clusters Azure HDInsight qui stockent ou traitent des informations sensibles, marquez le cluster et les ressources associées comme sensibles à l’utilisation d’étiquettes. Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour le Stockage Azure ou les ressources de calcul. Implémentez une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour un ensemble robuste de contrôles et de fonctionnalités de protection des données.

Comprendre la protection des données client dans Azure :

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Partagé

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Instructions**  : Si vous utilisez Azure SQL Database pour stocker des métadonnées Apache Hive et Apache Oozie, vérifiez que les données SQL restent chiffrées en permanence. Pour les comptes de Stockage Azure et Data Lake Storage (Gen1 ou Gen2), nous vous recommandons d’autoriser Microsoft à gérer vos clés de chiffrement. Vous avez toutefois la possibilité de gérer vos propres clés.

Comment gérer les clés de chiffrement des comptes de Stockage Azure :

https://docs.microsoft.com/azure/storage/common/storage-encryption-keys-portal

Comment créer un compte Azure Data Lake Storage à l’aide de clés de chiffrement gérées par le client :

https://docs.microsoft.com/azure/data-lake-store/data-lake-store-get-started-portal

Comprendre le chiffrement pour Azure SQL Database :

https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview#data-encryption

Comment configurer le chiffrement transparent des données (TDE, Transparent Data Encryption) pour SQL Database à l’aide de clés gérées par le client :

https://docs.microsoft.com/azure/sql-database/transparent-data-encryption-azure-sql?tabs=azure-portal

**Supervision Azure Security Center**  : Oui

**Responsabilité**  : Partagé

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Instructions**  : Configurez les paramètres de diagnostic de comptes de stockage Azure associés à des clusters Azure HDInsight pour superviser et journaliser toutes les opérations CRUD sur les données de cluster. Activez l’audit pour tous les comptes de Stockage ou Data Lake Stores associés au cluster Azure HDInsight.

Comment activer une journalisation/un audit supplémentaires pour un compte de Stockage Azure :

https://docs.microsoft.com/azure/storage/common/storage-monitor-storage-account

Comment activer une journalisation/un audit supplémentaires pour Azure Data Lake Storage :

https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-diagnostic-logs

**Supervision Azure Security Center**  : Oui

**Responsabilité**  : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Instructions**  : Implémentez une solution tierce de gestion des vulnérabilités.

Éventuellement, si vous disposez d’un abonnement à Rapid7, à Qualys ou à toute autre plateforme de gestion des vulnérabilités, vous pouvez utiliser des actions de script pour installer des agents d’évaluation des vulnérabilités sur vos nœuds de cluster Azure HDInsight et gérer les nœuds par le biais du portail correspondant.

Comment installer manuellement l’agent Rapid7 :

https://insightvm.help.rapid7.com/docs/install

Comment installer manuellement l’agent Qualys :

https://www.qualys.com/docs/qualys-cloud-agent-linux-install-guide.pdf

Comment utiliser des actions de script :

https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-customize-cluster-linux

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Instructions**  : Les mises à jour automatiques du système ont été activées pour les images de nœud de cluster. Toutefois, vous devez redémarrer régulièrement les nœuds de cluster pour garantir que les mises à jour sont appliquées.

Microsoft pour tenir à jour et mettre à jour les images de nœud Azure HDInsight de base.

Comment configurer la planification de mise à jour corrective du système d’exploitation pour les clusters HDInsight :

https://docs.microsoft.com/azure/hdinsight/hdinsight-os-patching

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Partagé

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3 : Déployer une solution de gestion de correctif logiciel tiers automatisée

**Instructions**  : Utilisez des actions de script ou d’autres mécanismes pour corriger vos clusters Azure HDInsight. Les clusters nouvellement créés disposeront toujours des mises à jour et des correctifs de sécurité les plus récents.

Comment configurer la planification de la mise à jour corrective du système d’exploitation pour les clusters Azure HDInsight Linux :

https://docs.microsoft.com/azure/hdinsight/hdinsight-os-patching

Comment utiliser des actions de script :

https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-customize-cluster-linux

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Instructions**  : Implémentez une solution tierce de gestion des vulnérabilités qui permet de comparer les analyses de vulnérabilités au fil du temps. Si vous disposez d’un abonnement Rapid7 ou Qualys, vous pouvez utiliser le portail de ce fournisseur pour consulter et comparer les analyses de vulnérabilités dos à dos.

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Instructions**  : Utilisez un programme de scoring des risques courants (par exemple, Common Vulnerability Scoring System) ou bien les évaluations des risques par défaut fournies par votre outil d’analyse tiers.

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1 : Utiliser la découverte de ressources Azure

**Instructions**  : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (comme le calcul, le stockage, le réseau, les ports, les protocoles, etc.), dont les clusters Azure HDInsight, dans vos abonnements.  Vérifiez que vous disposez des autorisations (en lecture) appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

Créer des requêtes avec Azure Resource Graph :

https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Afficher vos abonnements Azure :

https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription

Comprendre Azure RBAC :

https://docs.microsoft.com/azure/role-based-access-control/overview

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils**  : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

Comment créer et utiliser des étiquettes :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils**  : Utilisez des étiquettes, des groupes d’administration et diviser des abonnements, le cas échéant, pour organiser et suivre les ressources. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

Créer des abonnements Azure supplémentaires :

https://docs.microsoft.com/azure/billing/billing-create-subscription

Créer des groupes d’administration :

https://docs.microsoft.com/azure/governance/management-groups/create

Comment créer et utiliser des étiquettes :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4 : Tenir un inventaire des ressources Azure approuvées et titres des logiciels

**Conseils**  : Établissez la liste des ressources Azure et des logiciels approuvés pour vos ressources de calcul

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils**  : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

Utilisez Azure Resource Graph pour interroger/découvrir les ressources dans vos abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Créer des requêtes avec Azure Graph : https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal



**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Instructions**  : Implémentez une solution tierce afin de superviser les nœuds de cluster pour les applications logicielles non approuvées.

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Instructions**  : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (comme le calcul, le stockage, le réseau, les ports, les protocoles, etc.), dont les clusters Azure HDInsight, dans vos abonnements.  Supprimez toutes les ressources Azure non approuvées que vous découvrez. Pour les nœuds de cluster Azure HDInsight, implémentez une solution tierce afin de supprimer les logiciels non approuvés ou de générer des alertes s’y rapportant.

Créer des requêtes avec Azure Graph :

https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Instructions**  : Pour les nœuds de cluster Azure HDInsight, implémentez une solution tierce qui empêche l’exécution de logiciels non autorisés.


**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Aide**  : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Comment refuser un type de ressource spécifique avec Azure Policy : https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types


**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="610-implement-approved-application-list"></a>6.10 : Implémenter une liste d’applications approuvées

**Instructions**  : Pour les nœuds de cluster Azure HDInsight, implémentez une solution tierce qui empêche l’exécution de types de fichiers non autorisés.


**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resources-manager-via-scripts"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager par le biais de scripts

**Conseils**  : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

Configurer l’accès conditionnel pour bloquer l’accès à Azure Resource Manager : https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management


**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Aide**  : Non applicable. Cela ne concerne pas Azure HDInsight, car les utilisateurs (non-administrateurs) du cluster n’ont pas besoin d’accéder aux nœuds individuels pour exécuter des travaux. L’administrateur de cluster dispose d’un accès racine à tous les nœuds de cluster.

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Non applicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Aide**  : Non applicable. Le test d’évaluation est destiné à Azure Apps Service ou aux ressources de calcul hébergeant des applications web.

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Non applicable

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Instructions**  : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.HDInsight » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de votre cluster HDInsight.

Affichage des alias Azure Policy disponibles :

https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias

Configurer et gérer Azure Policy :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Instructions**  : Images du système d’exploitation Azure HDInsight gérées et tenues à jour par Microsoft. Client chargé d’implémenter des configurations sécurisées pour le système d’exploitation de vos nœuds de cluster. 


**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide**  : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés pour vos clusters Azure HDInsight et les ressources associées.

Configurer et gérer Azure Policy :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Comprendre les effets d’Azure Policy :

https://docs.microsoft.com/azure/governance/policy/concepts/effects

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Instructions**  : Images du système d’exploitation Azure HDInsight gérées et tenues à jour par Microsoft. Client chargé de l’implémentation de la configuration de l’état au niveau du système d’exploitation.


**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Partagé

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide**  : Si vous utilisez des définitions Azure Policy personnalisées, utilisez Azure DevOps ou Azure Repos pour stocker et gérer votre code en toute sécurité.

[Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops&preserve-view=true)

[Documentation Azure Repos](/azure/devops/repos/index?view=azure-devops&preserve-view=true)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Aide**  : Non applicable. Les images personnalisées ne sont pas applicables à Azure HDInsight.

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Non applicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7 : Déployer les outils de gestion de configuration système

**Instructions**  : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.HDInsight » afin de créer des stratégies personnalisées afin d’alerter, d’auditer ou d’appliquer des configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

Configurer et gérer Azure Policy :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation

**Instructions**  : Implémentez une solution tierce afin de maintenir l’état souhaité pour les systèmes d’exploitation de vos nœuds de cluster.

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la surveillance de la configuration automatique pour les services Azure

**Instructions**  : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.HDInsight » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration de votre cluster HDInsight.

[Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias)

[Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Instructions**  : Implémentez une solution tierce pour superviser l’état des systèmes d’exploitation de vos nœuds de cluster.

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Instructions**  : Azure HDInsight assure la prise en charge du modèle BYOK (Bring Your Own Key) pour Apache Kafka. Cette fonctionnalité vous permet de posséder et de gérer les clés servant à chiffrer les données au repos.

Tous les disques managés dans Azure HDInsight sont protégés par Azure Storage Service Encryption (SSE). Par défaut, les données stockées sur ces disques sont chiffrées à l’aide de clés gérées par Microsoft. Si vous activez BYOK, vous devez fournir la clé de chiffrement pour permettre à Azure HDInsight de l’utiliser et de la gérer à l’aide d’Azure Key Vault.

Key Vault peut également être utilisé avec des déploiements Azure HDInsight pour gérer les clés du stockage de cluster (comptes de Stockage Azure et Azure Data Lake Storage)

Comment apporter sa propre clé pour Apache Kafka sur Azure HDInsight :

https://docs.microsoft.com/azure/hdinsight/kafka/apache-kafka-byok

Comment gérer les clés de chiffrement des comptes de Stockage Azure :

https://docs.microsoft.com/azure/storage/common/storage-encryption-keys-portal

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Instructions**  : L’utilisation d’identités managées dans Azure HDInsight permet à vos clusters d’accéder aux services de domaine Azure Active Directory, à Azure Key Vault ou aux fichiers stockés dans Azure Data Lake Storage Gen2.

Comprendre les identités managées avec Azure HDInsight :

https://docs.microsoft.com/azure/hdinsight/hdinsight-managed-identities

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Instructions**  : Si vous utilisez du code lié à votre déploiement Azure HDInsight, vous pouvez implémenter Credential Scanner pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault. 

Configuration de Credential Scanner :

https://secdevtools.azurewebsites.net/helpcredscan.html

**Supervision Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Instructions**  : Azure HDInsight est fourni avec Clamscan préinstallé et activé pour les images de nœud de cluster. Toutefois, vous devez gérer le logiciel et agréger/superviser manuellement tous les journaux générés par Clamscan.

Comprendre Clamscan pour Azure HDInsight :

https://docs.microsoft.com/azure/hdinsight/hdinsight-faq#security-and-certificates

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Instructions**  : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure, mais il ne s’exécute pas sur du contenu client.

Pré-analysez tous les fichiers téléchargés vers toute ressource Azure associée à votre déploiement de cluster Azure HDInsight, comme un stockage Data Lake Storage, un stockage Blob, etc. Microsoft ne peut pas accéder aux données client dans ces instances.

Comprendre Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles :

 https://docs.microsoft.com/azure/security/fundamentals/antimalware

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Partagé

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Instructions**  : Azure HDInsight est fourni avec Clamscan préinstallé et activé pour les images de nœud de cluster. Clamscan effectue automatiquement des mises à jour du moteur et des définitions. Toutefois, l’agrégation et la gestion des journaux devront être effectuées manuellement.

Comprendre Clamscan pour Azure HDInsight :

https://docs.microsoft.com/azure/hdinsight/hdinsight-faq#security-and-certificates

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

## <a name="data-recovery"></a>Récupération des données

*Pour plus d’informations, consultez [Contrôle de sécurité : récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Instructions**  : Quand vous utilisez un compte de Stockage Azure pour le magasin de données de cluster HDInsight, choisissez l’option de redondance appropriée (LRS, ZRS, GRS, RA-GRS).  Quand vous utilisez une base de données Azure SQL pour le magasin de données de cluster Azure HDInsight, configurez la géoréplication active.

Comment configurer la redondance du stockage pour les comptes de Stockage Azure :

https://docs.microsoft.com/azure/storage/common/storage-redundancy

Comment configurer la redondance pour une base de données SQL Azure :

https://docs.microsoft.com/azure/sql-database/sql-database-active-geo-replication

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Instructions**  : Quand vous utilisez un compte de Stockage Azure pour le magasin de données de cluster Azure HDInsight, choisissez l’option de redondance appropriée (LRS, ZRS, GRS, RA-GRS). Si vous utilisez Azure Key Vault pour n’importe quelle partie de votre déploiement Azure HDInsight, vérifiez que vos clés sont sauvegardées.

Choisir les options de stockage pour votre cluster Azure HDInsight :

https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-compare-storage-options

Comment configurer la redondance du stockage pour les comptes de Stockage Azure :

https://docs.microsoft.com/azure/storage/common/storage-redundancy

Comment sauvegarder des clés Key Vault dans Azure :

https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : valider toutes les sauvegardes, y compris les clés gérées par le client

**Instructions**  : Si Azure Key Vault est utilisé avec votre déploiement Azure HDInsight, testez la restauration des clés gérées par le client sauvegardées.

Comment apporter sa propre clé pour Apache Kafka sur Azure HDInsight :

https://docs.microsoft.com/azure/hdinsight/kafka/apache-kafka-byok

Comment restaurer des clés de coffre de clés dans Azure :

https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : garantir la protection des sauvegardes et des clés gérées par le client

**Instructions**  : Si Azure Key Vault est utilisé avec votre déploiement Azure HDInsight, activez la suppression réversible dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.

Comment activer la suppression réversible dans Azure Key Vault :

https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils**  : Vérifiez qu’il existe des plans de réponse aux incidents écrits qui définissent les rôles du personnel, ainsi que les phases de gestion des incidents.

Comment configurer des automations de flux de travail dans Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide

**Supervision Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils**  : Security Center affecte un degré de gravité aux alertes pour vous aider à hiérarchiser l’ordre dans lequel vous remédiez à chaque alerte. Ainsi, quand une ressource est compromise, vous pouvez vous en occuper immédiatement. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Aide**  : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins. Reportez-vous à la publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques) : https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf

**Supervision Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Instructions**  : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le centre de réponse aux incidents de sécurité Microsoft (MSRC, Microsoft Security Response Center) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé.

Comment définir le contact de sécurité Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Supervision Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils**  : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser la sentinelle des alertes.

Comment configurer l’exportation continue :

https://docs.microsoft.com/azure/security-center/continuous-export

Comment diffuser des alertes dans Azure Sentinel :

https://docs.microsoft.com/azure/sentinel/connect-azure-security-center

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils**  : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

Comment configurer l’automatisation du flux de travail et Logic Apps :

https://docs.microsoft.com/azure/security-center/workflow-automation

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1 : Procédez régulièrement à des tests d’intrusion de vos ressources Azure et veillez à résoudre tous les problèmes de sécurité critiques détectés dans un délai de 60 jours

**Conseils**  : Respectez les règles d’engagement de Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft :

https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1.

Vous trouverez plus d’informations sur la stratégie de Microsoft et l’exécution de Red Teaming et des tests de pénétration de site actif sur l’infrastructure, les services et les applications cloud managés par Microsoft ici : https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consultez le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- Apprenez-en davantage sur les [Bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)