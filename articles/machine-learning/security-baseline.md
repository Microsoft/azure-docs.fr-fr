---
title: Ligne de base de la sécurité Azure pour Azure Machine Learning
description: La ligne de base de la sécurité pour Azure Machine Learning fournit des instructions et ressources pour l’implémentation des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 08/19/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: f3f66209b2d0a1bae18364e38790c237952e729f
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96458261"
---
# <a name="azure-security-baseline-for-azure-machine-learning"></a>Ligne de base de la sécurité Azure pour Azure Machine Learning

La ligne de base de la sécurité pour Microsoft Azure Machine Learning contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement. La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques. Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Conseils** : Azure Machine Learning s’appuie sur d’autres services Azure pour les ressources de calcul. Les ressources de calcul (cibles de calcul) sont utilisées pour entraîner et déployer des modèles. Vous pouvez créer ces cibles de calcul dans un réseau virtuel. Par exemple, vous pouvez utiliser une instance de calcul Azure Machine Learning pour entraîner un modèle et déployer celui-ci sur Azure Kubernetes Service (AKS). Cet article explique comment sécuriser vos cycles de vie d’apprentissage automatique en isolant les travaux d’apprentissage et d’inférence Azure Machine Learning au sein d’un réseau virtuel Azure.

Le pare-feu Azure peut être utilisé pour contrôler l’accès à votre espace de travail Azure Machine Learning et à l’Internet public.

- [Vue d’ensemble de l’isolement et de la confidentialité des réseaux virtuels](how-to-network-security-overview.md)

- [Utiliser l’espace de travail derrière le Pare-feu Azure pour Azure Machine Learning](how-to-access-azureml-behind-firewall.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Conseils** : Azure Machine Learning s’appuie sur d’autres services Azure pour les ressources de calcul. Affectez des groupes de sécurité réseau aux réseaux créés dans le cadre de votre déploiement d’Azure Machine Learning. 

Activez les journaux de flux de groupe de sécurité réseau et envoyez ceux-ci à un compte Stockage Azure à des fins d’audit. Vous pouvez aussi envoyer les journaux de flux à un espace de travail Log Analytics, puis utiliser Traffic Analytics pour obtenir des insights sur les modèles de trafic au sein de votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau, d’identifier les zones réactives et des menaces de sécurité, de comprendre les modèles de flux de trafic, ainsi que d’épingler de mauvaises configurations du réseau.

- [Tutoriel : journaliser le trafic réseau à destination et en provenance d’une machine virtuelle à l’aide du portail Azure](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Traffic Analytics](../network-watcher/traffic-analytics.md)

- [Protéger vos ressources réseau](../security-center/security-center-network-recommendations.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : vous pouvez activer le protocole HTTPS pour sécuriser la communication avec les services web déployés par Azure Machine Learning. Les services web sont déployés sur Azure Kubernetes Service (AKS) ou Azure Container Instances (ACI), et sécurisent les données soumises par les clients. Vous pouvez également utiliser une adresse IP privée avec AKS afin de restreindre le scoring de telle sorte que seuls les clients situés derrière un réseau virtuel puissent accéder au service web.

- [Utiliser TLS pour sécuriser un service web par le biais d’Azure Machine Learning](how-to-secure-web-service.md)

- [Vue d’ensemble de l’isolement et de la confidentialité des réseaux virtuels](how-to-network-security-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Conseils** : Activez la protection DDoS standard sur les réseaux virtuels associés à votre instance Azure Machine Learning afin de prévenir les attaques par déni de service distribué (DDoS). Utilisez la détection des menaces intégrée d’Azure Security Center afin de détecter les communications avec des adresses IP Internet inutilisées ou connues comme étant malveillantes.

Déployez le Pare-feu Azure aux limites réseau de l’organisation en activant le filtrage basé sur le renseignement sur les menaces et en le configurant sur « Alerter et refuser » afin de vous protéger de tout trafic réseau malveillant.

- [Guide pratique pour configurer la protection DDoS](../ddos-protection/manage-ddos-protection.md)

- [Utiliser l’espace de travail derrière le Pare-feu Azure pour Azure Machine Learning](how-to-access-azureml-behind-firewall.md)

- [Informations supplémentaires sur la détection des menaces d’Azure Security Center](../security-center/azure-defender.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Conseils** : pour toute machine virtuelle de vos services Azure Machine Learning sur laquelle est installée l’extension appropriée, vous pouvez activer la capture de paquets du service Network Watcher afin d’analyser les activités anormales. 

- [Créer une instance d’Azure Network Watcher](../network-watcher/network-watcher-create.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention d’intrusion (IDS/IPS) basés sur le réseau

**Conseils** : déployez la solution de pare-feu de votre choix dans les limites réseau de votre organisation pour détecter et/ou bloquer le trafic malveillant.

Sélectionnez une offre de la place de marché Azure qui prend en charge les fonctionnalités IDS/IPS avec des fonctionnalités d’inspection de charge utile.  Lorsque l’inspection de la charge utile n’est pas obligatoire, vous pouvez utiliser le renseignement sur les menaces du Pare-feu Azure. Le filtrage basé sur le renseignement sur les menaces du Pare-feu Azure génère des alertes et bloque le trafic en provenance ou à destination d’adresses IP et de domaines malveillants connus. Ces adresses IP et domaines proviennent du flux Microsoft Threat Intelligence.

- [Guide pratique pour déployer le Pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md)

- [Guide pratique pour configurer des alertes avec le Pare-feu Azure](../firewall/threat-intel.md)

- [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : pour les ressources qui doivent accéder à votre compte Azure Machine Learning, utilisez des étiquettes de service de réseau virtuel afin de définir des contrôles d’accès réseau sur des groupes de sécurité réseau ou le Pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de l’étiquette de service (par exemple, AzureMachineLearning) dans le champ Source ou Destination approprié d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Le service Azure Machine Learning propose une liste de balises de service pour ses cibles de calcul au sein d’un réseau virtuel, qui permet de minimiser la complexité. Vous pouvez vous en servir pour gérer votre réseau.

- [Pour plus d’informations sur l’utilisation des étiquettes de service](../virtual-network/service-tags-overview.md)

- [Vue d’ensemble de l’isolement et de la confidentialité des réseaux virtuels](how-to-network-security-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : définissez et implémentez des configurations de sécurité standard pour les ressources réseau associées à vos espaces de noms Azure Event Hubs à l’aide d’Azure Policy. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.MachineLearning » et « Microsoft.Network » afin de créer des stratégies personnalisées d’audit ou d’application de la configuration réseau de vos espaces de noms Azure Machine Learning. 

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : utilisez des balises pour les ressources réseau associées à votre déploiement Azure Machine Learning afin de les organiser logiquement dans une taxonomie.

Si une ressource de votre réseau virtuel Azure Machine Learning prend en charge le champ Description, utilisez-la pour documenter les règles qui autorisent le trafic à destination ou à partir d’un réseau.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : utilisez le journal des activités Azure pour superviser les configurations des ressources réseau et détecter les changements des ressources réseau associées à Azure Machine Learning. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Conseils** : Microsoft conserve la source de temps utilisée pour les ressources Azure, telles qu’Azure Machine Learning, en lien avec les horodatages dans les journaux.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : ingérez les journaux via Azure Monitor pour agréger les données de sécurité générées par Azure Machine Learning. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, et utilisez des comptes Stockage Azure pour le stockage à long terme et l’archivage. Vous pouvez également activer et intégrer des données dans Azure Sentinel ou une solution SIEM (Security Information and Event Management) tierce.

- [Configuration des journaux de diagnostic pour Azure Machine Learning](monitor-azure-machine-learning.md#configuration)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : activez les paramètres de diagnostic sur des ressources Azure pour accéder aux journaux d’audit, de sécurité et de diagnostic. Les journaux d’activité, automatiquement disponibles, incluent la source de l’événement, la date, l’utilisateur, l’horodatage, les adresses sources, les adresses de destination et d’autres éléments utiles.

Vous pouvez également mettre en corrélation des journaux d’opérations de service Azure Machine Learning à des fins de sécurité et de conformité.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md)

- [Présentation de la journalisation et des différents types de journaux dans Azure](../azure-monitor/platform/platform-logs-overview.md)

- [Activer la journalisation dans Azure Machine Learning](./how-to-track-experiments.md)

- [Supervision d’Azure Machine Learning](monitor-azure-machine-learning.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : si la ressource de calcul appartient à Microsoft, Microsoft est responsable de sa collecte et de sa supervision. 

Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour toutes les ressources de calcul appartenant à votre organisation, utilisez Azure Security Center pour surveiller le système d’exploitation. 

- [Guide pratique pour collecter les journaux des hôtes internes des machines virtuelles Azure avec Azure Monitor](../azure-monitor/learn/quick-collect-azurevm.md)

- [Présentation de la collecte de données Azure Security Center](../security-center/security-center-enable-data-collection.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Dans Azure Monitor, définissez la période de rétention des journaux pour les espaces de travail Log Analytics associés à vos instances Azure Machine Learning conformément aux réglementations de conformité de votre organisation.

- [Guide pratique pour définir les paramètres de conservation des journaux](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : analysez et supervisez les journaux afin de détecter tout comportement anormal, et examinez régulièrement les résultats pour votre solution Azure Machine Learning. Utilisez Azure Monitor et un espace de travail Log Analytics pour examiner les journaux et effectuer des requêtes sur leurs données.

Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce. 

- [Guide pratique pour exécuter des requêtes pour Azure Machine Learning dans des espaces de travail Log Analytics](monitor-azure-machine-learning.md#analyzing-log-data)

- [Activer la journalisation dans Azure Machine Learning](./how-to-track-experiments.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Bien démarrer avec les requêtes Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md)

- [Guide pratique pour effectuer des requêtes personnalisées dans Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils** : dans Azure Monitor, configurez les journaux liés à Azure Machine Learning dans les paramètres de diagnostic du journal d’activité et de Machine Learning pour envoyer les journaux vers un espace de travail Log Analytics à des fins d’interrogation, ou vers un compte de stockage en vue d’un stockage d’archivage à long terme. Utilisez l’espace de travail Log Analytics afin de créer des alertes pour les activités anormales détectées dans les événements et journaux de sécurité.

Vous pouvez également activer et intégrer les données dans Azure Sentinel.

- [Pour plus d’informations sur les alertes d’Azure Machine Learning](monitor-azure-machine-learning.md#alerts)

- [Guide pratique pour générer une alerte sur des données de journal de l’espace de travail Log Analytics](../azure-monitor/learn/tutorial-response.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Conseils** : si la ressource de calcul appartient à Microsoft, Microsoft est responsable du déploiement du logiciel anti-programme malveillant du service Azure Machine Learning. 

Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour les ressources de calcul appartenant à votre organisation, activez la collecte d’événements du logiciel anti-programme malveillant pour l’extension Microsoft Antimalware pour les services cloud et machines virtuelles Azure.

- [Guide pratique pour configurer l’extension Microsoft Antimalware pour une machine virtuelle](/powershell/module/servicemanagement/azure.service/set-azurevmmicrosoftantimalwareextension)

- [Guide pratique pour configurer l’extension Microsoft Antimalware pour les services cloud](/powershell/module/servicemanagement/azure.service/set-azureserviceantimalwareextension?view=azuresmps-4.0.0)

- [Présentation de Microsoft Antimalware](../security/fundamentals/antimalware.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Conseils** : non applicable. Azure Machine Learning ne traite pas et ne produit pas de journaux liés à DNS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Conseils** : Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour les ressources de calcul appartenant à votre organisation, utilisez Azure Security Center pour activer la surveillance du journal des événements de sécurité pour les machines virtuelles Azure. Azure Security Center approvisionne l’agent Log Analytics sur toutes les machines virtuelles Azure prises en charge, et toutes celles qui sont créées si l’approvisionnement automatique est activé. Vous pouvez sinon installer l’agent manuellement. Il active l’événement de création de processus 4688 et le champ de ligne de commande qui s’y trouve. Les nouveaux processus créés sur la machine virtuelle sont enregistrés par le journal des événements et analysés par les services de détection Security Center.

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : Vous pouvez utiliser l’onglet Gestion des identités et des accès (IAM) d’une ressource dans le portail Azure pour configurer le contrôle d’accès en fonction du rôle Azure (Azure RBAC) et conserver l’inventaire des ressources Azure Machine Learning. Les rôles sont appliqués aux utilisateurs, aux groupes, aux principaux de service et aux identités gérées dans Active Directory. Vous pouvez utiliser des rôles intégrés ou des rôles personnalisés pour les individus et les groupes.

Azure Machine Learning fournit des rôles intégrés pour les scénarios de gestion courants dans Azure Machine Learning. Une personne disposant d’un profil dans Azure Active Directory (Azure AD) peut attribuer ces rôles à des utilisateurs, des groupes, des principaux de service ou des identités managées pour accorder ou refuser l’accès à des ressources et opérations sur des ressources Azure Machine Learning.

Vous pouvez également utiliser le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes qui sont membres de groupes d’administration.

- [Comprendre le contrôle d’accès Azure dans Azure Machine Learning](how-to-assign-roles.md)

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure Active Directory avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : la gestion de l’accès aux ressources Machine Learning est contrôlée via Azure Active Directory (Azure AD). Azure AD n’intègre pas le concept des mots de passe par défaut.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Aide** : Azure Machine Learning est fourni avec trois rôles par défaut lors de la création d’un espace de travail, créant ainsi des procédures d’exploitation standard autour de l’utilisation de comptes de propriétaire.

Vous pouvez également activer l’accès juste-à-temps aux comptes d’administration en utilisant Azure Active Directory Privileged Identity Management et Azure Resource Manager. 

- [Pour en savoir plus sur les rôles par défaut de la solution Machine Learning](how-to-assign-roles.md#default-roles)

- [En savoir plus sur Privileged Identity Management](../active-directory/privileged-identity-management/index.yml)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : la solution Machine Learning étant intégrée avec Azure Active Directory, utilisez Azure Active Directory SSO au lieu de configurer des informations d’identification autonomes individuelles par service. Suivez les suggestions d’Azure Security Center liées à la gestion des identités et des accès.  

- [Présentation de l’authentification SSO avec Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : activer l’authentification multifacteur Azure Active Directory et suivez les recommandations liées aux identités et aux accès dans Azure Security Center.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Aide** : Utilisez une station de travail sécurisée et managée par Azure (également appelée station de travail à accès privilégié) pour les tâches administratives qui requièrent des privilèges élevés.

- [Comprendre les stations de travail sécurisées gérées par Azure](../active-directory/devices/concept-azure-managed-workstation.md)

- [Procédure d’activation d’Azure AD MFA](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils** : Utilisez les rapports de sécurité Azure Active Directory et la supervision pour détecter les activités suspectes ou potentiellement dangereuses qui se produisent dans l’environnement. Utiliser Azure Security Center pour superviser les activités liées aux identités et aux accès.

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-only-from-approved-locations"></a>3.8 : Gérer les ressources Azure uniquement à partir d’emplacements approuvés

**Aide** : Utilisez des emplacements nommés Azure AD pour n’autoriser l’accès qu’à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.
 
 
 
- [Procédure de configuration des emplacements nommés Azure AD](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Utiliser Azure Active Directory (Azure AD) comme système d’authentification et d’autorisation central. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.
 
L'accès en fonction du rôle peut être limité à plusieurs niveaux dans Azure. Pour Machine Learning, les rôles peuvent être gérés au niveau de l’espace de travail. Par exemple, vous pouvez disposer d’un accès de propriétaire à un espace de travail, mais au groupe de ressources contenant l’espace de travail. Cela fournit des contrôles d’accès plus précis pour séparer les rôles au sein du même groupe de ressources. 

- [Gérer l'accès à un espace de travail Azure Machine Learning](how-to-assign-roles.md) 
 
- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Conseils** : Azure AD fournit des journaux pour vous aider à découvrir les comptes obsolètes. Par ailleurs, utilisez les révisions d’accès et des identités Azure AD pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seuls les utilisateurs appropriés continuent de bénéficier d’un accès. 
 
Utilisez Azure Active Directory (Azure AD) Privileged Identity Management (PIM) pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.

- [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml)

- [Utilisation des révisions d’accès et des identités Azure AD](../active-directory/governance/access-reviews-overview.md)

- [Déployer Azure AD Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Aide** : Vous avez accès aux activités de connexion Azure AD, aux sources des journaux d’événements à risque et d’audit, ce qui vous permet de les intégrer à un outil SIEM/de supervision.

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure AD et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes souhaitées dans un espace de travail Log Analytics.
 
 
- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez les fonctionnalités d’Azure AD Identity Protection pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. Vous pouvez également ingérer des données dans Azure Sentinel pour approfondir votre examen.
 
- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)
 
- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)
 
- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : non applicable ; le service Azure Machine Learning ne prend pas en charge Customer Lockbox.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-protection"></a>Protection de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.
 
- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Aide** : Implémentez une isolation en utilisant des abonnements et groupes d’administration distincts selon les domaines de sécurité (par exemple, le type d’environnement et le niveau de confidentialité des données). Vous pouvez limiter le niveau d’accès à vos ressources Azure demandées par vos applications et environnements d’entreprise. Vous pouvez contrôler l’accès aux ressources Azure via Azure RBAC.
 
- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)
 
- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : utilisez une solution tierce de la Place de marché Azure dans les périmètres du réseau, qui surveille et bloque les transferts non autorisés d’informations sensibles tout en alertant les professionnels de la sécurité des informations. 

Pour la plateforme sous-jacente gérée par Microsoft, Microsoft traite tout le contenu client en tant que contenu sensible, et assure une protection contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes. 

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : les services web déployés via Azure Machine Learning prennent en charge uniquement le protocole TLS version 1.2 qui effectue le chiffrement des données en transit.

- [Utiliser TLS pour sécuriser un service web par le biais d’Azure Machine Learning](how-to-secure-web-service.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Conseils** : les fonctionnalités d’identification, de classification et de protection contre la perte des données ne sont pas encore disponibles pour Azure Machine Learning. Implémentez une solution tierce si nécessaire pour la conformité.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="46-use-azure-rbac-to-manage-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : Azure Machine Learning prend en charge l’utilisation d’Azure Active Directory (Azure AD) pour autoriser les demandes d’accès aux ressources Machine Learning. Azure AD vous permet d’utiliser le contrôle d’accès en fonction du rôle (Azure RBAC) pour accorder des autorisations à un principal de sécurité, qui peut être un utilisateur ou un principal de service d’application.

- [Gérer l'accès à un espace de travail Azure Machine Learning](how-to-assign-roles.md)
- [Utiliser Azure RBAC pour l’autorisation Kubernetes](../aks/manage-azure-rbac.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

Microsoft gère l’infrastructure sous-jacente de la solution Machine Learning, et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : Azure Machine Learning stocke les captures instantanées, les sorties et les journaux dans le compte Stockage Blob Azure qui est lié à l’espace de travail Azure Machine Learning et à votre abonnement. Toutes les données stockées dans Stockage Blob Azure sont chiffrées au repos à l’aide de clés gérées par Microsoft. Vous pouvez également chiffrer les données stockées dans un Stockage Blob Azure avec vos propres clés dans le service Machine Learning. 

- [Chiffrement des données au repos dans la solution Azure Machine Learning](concept-enterprise-security.md#encryption-at-rest)

- [Présentation du chiffrement au repos dans Azure](../security/fundamentals/encryption-atrest.md)

- [Guide pratique pour configurer des clés de chiffrement gérées par le client](../storage/common/customer-managed-keys-configure-key-vault.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes en cas de modifications d’instances de production d’Azure Machine Learning et d’autres ressources critiques ou associées.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : si la ressource de calcul appartient à Microsoft, Microsoft est responsable de la gestion des vulnérabilités du service Azure Machine Learning. 

Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour les ressources de calcul appartenant à votre organisation, suivez les recommandations d’Azure Security Center pour effectuer des évaluations de vulnérabilité sur vos machines virtuelles Azure, les images de conteneur et les serveurs SQL.

- [Implémenter les recommandations d'évaluation des vulnérabilités d'Azure Security Center](../security-center/deploy-vulnerability-assessment-vm.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : si la ressource de calcul appartient à Microsoft, Microsoft est responsable de la gestion des correctifs du service Azure Machine Learning. 

Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour toutes les ressources de calcul appartenant à votre organisation, utilisez Azure Automation Update Management pour vous assurer que les mises à jour de sécurité les plus récentes sont installées sur vos machines virtuelles Windows et Linux. Pour les machines virtuelles Windows, assurez-vous que Windows Update a été activé et configuré pour être mis à jour automatiquement.

- [Configurer Update Management pour les machines virtuelles dans Azure](../automation/update-management/overview.md)

- [Comprendre les stratégies de sécurité Azure analysées par Security Center](../security-center/policy-reference.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="53-deploy-an-automated-patch-management-solution-for-third-party-software-titles"></a>5.3 : Déployer une solution de gestion automatisée des correctifs de logiciels tiers

**Conseils** : Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour les ressources de calcul appartenant à votre organisation, utilisez une solution tierce de gestion des correctifs. Les clients qui utilisent déjà Configuration Manager dans leur environnement peuvent également tirer parti de l’éditeur de mise à jour de System Center, qui leur permet de publier des mises à jour personnalisées dans le service Windows Server Update. Cela permet à la solution Update Management d’appliquer des correctifs aux machines qui utilisent Configuration Manager en tant que référentiel de mise à jour avec des logiciels tiers.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils** : Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour les ressources de calcul appartenant à votre organisation, suivez les recommandations d’Azure Security Center pour effectuer des évaluations de vulnérabilité de vos machines virtuelles Azure, images de conteneur et serveurs SQL. Exportez les résultats de l’analyse à intervalles réguliers, et comparez les résultats pour vérifier que les vulnérabilités ont été corrigées. Lorsque vous suivez les recommandations de gestion des vulnérabilités proposées par Azure Security Center, vous pouvez pivoter vers le portail de la solution sélectionnée pour afficher les données d’analyse historiques.

- [Implémenter les recommandations d'évaluation des vulnérabilités d'Azure Security Center](../security-center/deploy-vulnerability-assessment-vm.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : le service Azure Resource Graph permet d’interroger et de découvrir toutes les ressources (telles que le calcul, le stockage, le réseau, les ports, etc.) dans vos abonnements.  Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements.

Même s’il est possible de découvrir les ressources Azure classiques via l’Explorateur Azure Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à partir de maintenant.

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : appliquez des balises aux ressources Azure en y ajoutant des métadonnées afin de les organiser de façon logique dans une taxonomie.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Aide** : Le cas échéant, utilisez des étiquettes, des groupes d’administration et des abonnements séparés pour organiser et suivre les ressources. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.
 
 
 
- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)
 
 
 
- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)
 
 
 
- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4 : Dresser et tenir un inventaire des ressources Azure approuvées

**Conseils** : Créez un inventaire des ressources Azure et logiciels approuvés pour les ressources de calcul en fonction des besoins de votre organisation.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients selon les définitions de stratégies intégrées suivantes :

* Types de ressources non autorisés
* Types de ressources autorisés

Utilisez également Azure Resource Graph pour interroger/découvrir des ressources dans les abonnements.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour les ressources de calcul appartenant à votre organisation, utilisez la fonctionnalité Suivi des modifications et inventaire d’Azure Automation Change pour automatiser la collecte d’informations d’inventaire à partir de vos machines virtuelles Windows et Linux. Le nom, la version, l’éditeur et l’heure d’actualisation du logiciel sont disponibles sur le portail Azure. Pour obtenir la date d’installation du logiciel, ainsi que d’autres informations, activez les diagnostics au niveau de l’invité, et acheminez les journaux les événements Windows vers un espace de travail Log Analytics.

- [Guide pratique pour activer le regroupement d’inventaires Azure Automation pour une machine virtuelle](../automation/automation-tutorial-installed-software.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour les ressources de calcul appartenant à votre organisation, utilisez la solution de Supervision de l’intégrité des fichiers (File Integrity Monitoring, FIM) d’Azure Security Center pour identifier tous les logiciels installés sur les machines virtuelles. Une autre option utilisable à la place de la solution FIM ou conjointement avec celle-ci est la fonctionnalité Suivi des modifications et inventaire d’Azure Automation pour dresser l’inventaire de vos machines virtuelles Linux et Windows. 

Vous pouvez implémenter votre propre processus de suppression des logiciels non autorisés. Vous pouvez également utiliser une solution tierce pour identifier les logiciels non approuvés.

Supprimez les ressources Azure qui ne sont plus nécessaires.

- [Guide pratique pour utiliser le monitoring d’intégrité des fichiers](../security-center/security-center-file-integrity-monitoring.md)

- [Comprendre la fonctionnalité Suivi des modifications et inventaire d’Azure Automation](../automation/change-tracking/overview.md)

- [Guide pratique pour activer l’inventaire des machines virtuelles Azure](../automation/automation-tutorial-installed-software.md)

- [Suppression de ressources et de groupes de ressources Azure](../azure-resource-manager/management/delete-resource-group.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour des ressources de calcul appartenant à votre organisation, utilisez des contrôles d’application adaptatifs d’Azure Security Center pour que seuls les logiciels autorisés s’exécutent et que l’exécution de tous les logiciels non autorisés soit bloquée sur les machines virtuelles Azure.

- [Guide pratique pour utiliser les contrôles d’application adaptatifs d’Azure Security Center](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients selon les définitions de stratégies intégrées suivantes :

* Types de ressources non autorisés
* Types de ressources autorisés

Utilisez également Azure Resource Graph pour interroger et découvrir des ressources dans les abonnements.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés

**Conseils** : Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour toutes les ressources de calcul appartenant à votre organisation, utilisez les contrôles d’application adaptatifs d’Azure Security Center pour spécifier les types de fichiers auxquels une règle peut ou non s’appliquer.

Implémentez une solution tierce si les contrôles d’application adaptatifs ne satisfont pas à l’exigence.

- [Guide pratique pour utiliser les contrôles d’application adaptatifs d’Azure Security Center](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Conseils** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».
 
- [Guide pratique pour configurer l’accès conditionnel de façon à bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-in-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour les ressources de calcul appartenant à votre organisation, selon le type de script, vous pouvez utiliser des configurations spécifiques du système d’exploitation ou des ressources tierces pour limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul Azure.  Vous avez également la possibilité d’utiliser des contrôles d’application adaptatifs d’Azure Security Center pour faire en sorte que seuls les logiciels autorisés s’exécutent et que l’exécution de tous les logiciels non autorisés soit bloquée sur les machines virtuelles Azure.

- [Guide pratique pour contrôler l’exécution des scripts PowerShell dans des environnements Windows](/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-6)

- [Guide pratique pour utiliser les contrôles d’application adaptatifs d’Azure Security Center](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Conseils** : définissez et implémentez des configurations de sécurité standard pour votre service Azure Machine Learning avec Azure Policy. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.ApiManagement » pour créer des stratégies personnalisées en vue d’auditer ou d’appliquer la configuration de vos services Azure Machine Learning.

Azure Resource Manager est capable d’exporter le modèle au format JSON (JavaScript Object Notation), qui doit être examiné pour vérifier que les configurations répondent aux exigences de sécurité de votre organisation.

Vous pouvez aussi utiliser les recommandations d’Azure Security Center comme ligne de base de configuration sécurisée pour vos ressources Azure.

Azure Machine Learning prend entièrement en charge les dépôts Git pour le suivi du travail. Vous pouvez cloner des dépôts directement dans le système de fichiers de votre espace de travail partagé, ou utiliser Git sur votre station de travail locale, et vous assurer que les configurations sécurisées s’appliquent aux ressources de code dans le cadre de votre environnement de Machine Learning.

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

- [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

- [Exportation monoressource ou multiressource vers un modèle sur le portail Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Recommandations de sécurité - Guide de référence](../security-center/recommendations-reference.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : si la ressource de calcul appartient à Microsoft, Microsoft est responsable des configurations sécurisées du système d’exploitation du service Azure Machine Learning.

Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour les ressources de calcul appartenant à votre organisation, utilisez les recommandations d’Azure Security Center pour préserver leurs configurations de sécurité.  Par ailleurs, vous pouvez utiliser des images de système d’exploitation personnalisées ou Azure Automation State Configuration pour établir la configuration de sécurité du système d’exploitation requise par votre organisation.

- [Suivi des recommandations d'Azure Security Center](../security-center/security-center-recommendations.md)

- [Recommandations de sécurité - Guide de référence](../security-center/recommendations-reference.md)

- [Présentation d'Azure Automation State Configuration](../automation/automation-dsc-overview.md)

- [Charger un disque dur virtuel et l'utiliser pour créer des machines virtuelles Windows dans Azure](../virtual-machines/windows/upload-generalized-managed.md)

- [Créer une machine virtuelle Linux à partir d'un disque personnalisé avec Azure CLI](../virtual-machines/linux/upload-vhd.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure. En outre, vous pouvez utiliser des modèles Azure Resource Manager pour appliquer la configuration de sécurité des ressources Azure requise par votre organisation. 
 
 
 
- [Comprendre les effets d'Azure Policy](../governance/policy/concepts/effects.md)
 
- [Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)
 
- [Présentation des modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : si la ressource de calcul appartient à Microsoft, Microsoft est responsable des configurations sécurisées du système d’exploitation du service Azure Machine Learning.

Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour les ressources de calcul appartenant à votre organisation, suivez les recommandations d’Azure Security Center concernant les évaluations de vulnérabilité de vos ressources de calcul Azure.  En outre, vous pouvez utiliser des modèles Azure Resource Manager, des images de système d’exploitation personnalisées ou Azure Automation State Configuration pour tenir à jour la configuration de sécurité du système d’exploitation que votre organisation requiert.   Combinés avec Azure Automation State Configuration, les modèles de machine virtuelle Microsoft peuvent vous aider à satisfaire aux exigences de sécurité, ainsi qu’à les tenir à jour. 

Notez également que les images de machines virtuelles de la Place de marché Azure publiées par Microsoft sont gérées et tenues à jour par Microsoft. 

- [Implémenter les recommandations d'évaluation des vulnérabilités d'Azure Security Center](../security-center/deploy-vulnerability-assessment-vm.md)

- [Guide pratique pour créer une machine virtuelle Azure à partir d’un modèle ARM](../virtual-machines/windows/ps-template.md)

- [Présentation d'Azure Automation State Configuration](../automation/automation-dsc-overview.md)

- [Créer une machine virtuelle Windows sur le portail Azure](../virtual-machines/windows/quick-create-portal.md)

- [Informations sur le téléchargement du modèle de machine virtuelle](../virtual-machines/windows/download-template.md)

- [Exemple de script permettant de charger un disque dur virtuel sur Azure et de créer une machine virtuelle](../virtual-machines/scripts/virtual-machines-windows-powershell-upload-generalized-script.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Conseils** : si vous utilisez des définitions d’Azure Policy personnalisées pour vos ressources Machine Learning ou associées, utilisez Azure Repos pour stocker et gérer votre code de façon sécurisée.

Azure Machine Learning prend entièrement en charge les dépôts Git pour le suivi du travail. Vous pouvez cloner des dépôts directement dans le système de fichiers de votre espace de travail partagé, ou utiliser Git sur votre station de travail locale, et vous assurer que les configurations sécurisées s’appliquent aux ressources de code dans le cadre de votre environnement de Machine Learning.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [Documentation Azure Repos](/azure/devops/repos/index?view=azure-devops)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour les ressources de calcul appartenant à votre organisation, utilisez un contrôle d’accès en fonction du rôle (Azure RBAC) Azure pour vous assurer que seuls des utilisateurs autorisés puissent accéder à vos images personnalisées. Une galerie d’images partagées vous permet de partager vos images avec différents utilisateurs, principaux de service ou groupes Active Directory au sein de votre organisation. Stockez les images conteneur dans Azure Container Registry, et utilisez un contrôle d’accès en fonction du rôle Azure pour vous assurer que seuls des utilisateurs autorisés puissent y accéder.

- [Présentation d’Azure RBAC](../role-based-access-control/rbac-and-directory-admin-roles.md)

- [Comprendre Azure RBAC pour Container Registry](../container-registry/container-registry-roles.md)

- [Comment configurer Azure RBAC](../role-based-access-control/quickstart-assign-role-user-portal.md)

- [Présentation de Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md)

- [Utiliser Azure RBAC pour l’autorisation Kubernetes](../aks/manage-azure-rbac.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : utilisez des alias Azure Policy dans l’espace de noms « Microsoft.MachineLearning » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Utiliser des alias](../governance/policy/concepts/definition-structure.md#aliases)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer des outils de gestion de la configuration pour les systèmes d'exploitation

**Conseils** : si la ressource de calcul appartient à Microsoft, Microsoft est responsable du déploiement de la configuration sécurisée du service Azure Machine Learning.

Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour des ressources de calcul appartenant à votre organisation, utilisez Azure Automation State Configuration pour les nœuds Desired State Configuration (DSC) dans n’importe quel centre de ressources cloud ou local. Vous pouvez facilement intégrer des machines, leur affecter des configurations déclaratives et afficher des rapports montrant la conformité de chaque machine à l’état souhaité que vous avez spécifié. 

- [Guide pratique pour activer Azure Automation State Configuration](../automation/automation-dsc-onboarding.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Aide** : Utiliser Azure Security Center pour effectuer des analyses de ligne de base pour vos ressources Azure. En outre, utilisez Azure Policy pour alerter et auditer les configurations des ressources Azure.
 
 
 
- [Guide pratique pour Corriger les recommandations dans Azure Security Center](../security-center/security-center-remediate-recommendations.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : si la ressource de calcul appartient à Microsoft, Microsoft est responsable de la supervision de la configuration sécurisée automatisée du service Azure Machine Learning.

Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour les ressources de calcul appartenant à votre organisation, utilisez le calcul et les applications d’Azure Security Center, et suivez les recommandations relatives aux machines virtuelles, aux serveurs et aux conteneurs.

- [Comprendre les recommandations concernant les conteneurs dans Azure Security Center](../security-center/container-security.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : utilisez les identités managées conjointement avec Azure Key Vault afin de simplifier la gestion des secrets pour vos applications cloud.

La solution Azure Machine Learning prenant en charge le chiffrement du magasin de données avec des clés gérées par le client, vous devez gérer les exigences de conformité et de sécurité de l’organisation en lien avec la rotation et la révocation des clés. 

Utilisez Azure Key Vault pour transmettre des secrets à des exécutions à distance en toute sécurité plutôt qu’un texte clair dans vos scripts d’apprentissage.

- [Clés gérées par le client de la solution Azure Machine Learning](concept-enterprise-security.md#azure-blob-storage)

- [Utiliser les secrets d’authentification dans les exécutions d’apprentissage Azure Machine Learning](how-to-use-secrets-in-runs.md)

- [Guide pratique pour utiliser des identités managées pour des ressources Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

- [Créer un coffre de clés](../key-vault/general/quick-create-portal.md)

- [Comment s’authentifier auprès de Key Vault](../key-vault/general/authentication.md)

- [Comment attribuer une stratégie d’accès Key Vault](../key-vault/general/assign-access-policy-portal.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils** : la solution Azure Machine Learning prend en charge les rôles intégrés et la possibilité de créer des rôles personnalisés. Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure AD. Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Key Vault, sans informations d’identification dans votre code.

 
- [Gérer l'accès à un espace de travail Azure Machine Learning](how-to-assign-roles.md)

- [Guide pratique pour configurer des identités managées pour des ressources Azure](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault. 

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-antimalware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Conseils** : le logiciel anti-programme malveillant de Microsoft est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure Machine Learning), mais il ne s’exécute pas sur le contenu client.

Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour les ressources de calcul appartenant à votre organisation, utilisez l’extension Microsoft Antimalware pour Azure pour superviser et protéger vos ressources en permanence. Pour Linux, utilisez une solution logicielle anti-programme malveillant tierce. Utilisez également la détection des menaces pour les services de données d’Azure Security Center afin de détecter les programmes malveillants chargés sur des comptes de stockage.

- [Guide pratique pour configurer l’extension Microsoft Antimalware pour Azure](../security/fundamentals/antimalware.md)

- [Protection contre les menaces dans Azure Security Center](../security-center/azure-defender.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Conseils** : le logiciel anti-programme malveillant de Microsoft est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure Machine Learning), mais il ne s’exécute pas sur le contenu client.

Il vous incombe de pré-analyser tout contenu chargé vers des ressources Azure autres que de calcul. Microsoft ne peut pas accéder aux données client et ne peut donc pas effectuer d’analyses anti-programme malveillant du contenu client en votre nom.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-antimalware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont à jour

**Conseils** : le logiciel anti-programme malveillant de Microsoft est activé et géré sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure Machine Learning), mais il ne s’exécute pas sur le contenu client.

Azure Machine Learning prend en charge les différentes ressources de calcul, y compris vos propres ressources de calcul, de manière variable. Pour toutes les ressources de calcul appartenant à votre organisation, suivez les recommandations relatives au calcul et aux applications d’Azure Security Center pour vous assurer que tous les points de terminaison sont à jour avec les signatures les plus récentes. Pour Linux, utilisez une solution logicielle anti-programme malveillant tierce.

- [Guide pratique pour déployer l’extension Microsoft Antimalware pour Azure](../security/fundamentals/antimalware.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="data-recovery"></a>Récupération de données

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Conseils** : la gestion de la récupération des données dans le service Machine Learning consiste à gérer les données des magasins de données connectés. Veillez à suivre les instructions relatives à la récupération des données pour les magasins connectés afin de sauvegarder les données conformément aux stratégies de l’organisation cliente.

- [Guide pratique pour récupérer des fichiers à partir d’une sauvegarde de machines virtuelles Azure](../backup/backup-azure-restore-files-from-vm.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Conseils** : la gestion de la sauvegarde des données dans le service Machine Learning consiste à gérer les données des magasins de données connectés. Activez le service Sauvegarde Azure pour les machines virtuelles, et configurez la fréquence souhaitée ainsi que les périodes de rétention. Sauvegardez les clés gérées par le client dans Azure Key Vault.

- [Guide pratique pour récupérer des fichiers à partir d’une sauvegarde de machines virtuelles Azure](../backup/backup-azure-restore-files-from-vm.md)
- [Guide pratique pour restaurer des clés du coffre de clés dans Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : la validation de la sauvegarde des données dans le service Machine Learning consiste à gérer les données des magasins de données connectés. Effectuez régulièrement une restauration des données dans le service Sauvegarde Azure. Assurez-vous que vous pouvez restaurer les clés gérées par le client sauvegardées.

- [Guide pratique pour récupérer des fichiers à partir d’une sauvegarde de machine virtuelle Azure](../backup/backup-azure-restore-files-from-vm.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés gérées par le client

**Conseils** : pour une sauvegarde locale, le chiffrement au repos est assuré à l’aide de la phrase secrète que vous fournissez lorsque vous sauvegardez sur Azure. Utilisez un contrôle d’accès en fonction du rôle Azure pour protéger les sauvegardes et les clés gérées par le client. 

Activez la suppression réversible et la protection contre la purge dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante. Si le Stockage Azure est utilisé pour stocker les sauvegardes, la suppression réversible vous permet d’enregistrer et de récupérer vos données en cas de suppression d’objets blob ou d’instantanés d’objets blob.
 
 
- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

- [Guide pratique pour activer la suppression réversible et la protection contre la purge dans Key Vault](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

- [Suppression réversible pour le service Stockage Blob Azure](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Aide** : Développez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé. 

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

- [Utiliser le guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de votre propre plan de réponse aux incidents](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Aide** : Azure Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou à l’analytique utilisées pour émettre l’alerte, ainsi qu’à l’intention malveillante estimée de l’activité à l’origine du déclenchement de l’alerte.

En outre, marquez les abonnements à l’aide d’étiquettes et créez un système de nommage pour identifier et classer les ressources Azure, en particulier celles qui traitent des données sensibles. Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

- [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md)

- [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Effectuez des exercices pour tester les capacités de réponse aux incidents de vos systèmes à intervalles réguliers, afin de protéger vos ressources Azure. Identifiez les points faibles et les lacunes, puis révisez votre plan de réponse en fonction des besoins.

- [Publication du NIST--Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities(Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Instructions** : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations Azure Security Center en utilisant la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel.

- [Comment configurer l’exportation continue](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation de workflow d’Azure Security Center pour déclencher automatiquement des réponses aux alertes et aux recommandations de sécurité afin de protéger vos ressources Azure.

- [Comment configurer l’automatisation du workflow dans Security Center](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de pénétration du cloud Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft.

- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)