---
title: Base de référence de sécurité Azure pour Azure Data Factory
description: Base de référence de sécurité Azure pour Azure Data Factory
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 06/05/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: e48b1357b8576baca0e8defca94f4c5248c6f74e
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461286"
---
# <a name="azure-security-baseline-for-azure-data-factory"></a>Base de référence de sécurité Azure pour Azure Data Factory

La base de référence de sécurité Azure pour Azure Data Factory contient des suggestions qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez la [Vue d’ensemble des bases de référence de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Aide** : Lorsque vous créez un Azure-SSIS Integration Runtime (IR), vous avez la possibilité de le joindre à un réseau virtuel. Cela permet à Azure Data Factory de créer certaines ressources réseau, telles qu’un groupe de sécurité réseau (NSG) et un équilibreur de charge. Vous avez également la possibilité de fournir votre propre adresse IP publique statique ou de demander à Azure Data Factory d’en créer une pour vous. Sur le groupe de sécurité réseau qui est automatiquement créé par Azure Data Factory, le port 3389 est ouvert par défaut à tout le trafic. Verrouillez-le pour vous assurer que seuls vos administrateurs y aient accès.

Les IR auto-hébergés peuvent être déployés sur un ordinateur local ou une machine virtuelle Azure à l’intérieur d’un réseau virtuel. Assurez-vous que votre déploiement de sous-réseaux de réseau virtuel dispose d’un groupe de sécurité réseau configuré de manière à autoriser uniquement l’accès administratif. Azure-SSIS IR a interdit le port 3389 sortant par défaut au niveau de la règle de pare-feu Windows sur chaque nœud IR pour la protection. Vous pouvez sécuriser vos ressources configurées sur un réseau virtuel en associant un groupe de sécurité réseau au sous-réseau et en définissant des règles strictes.

Lorsque le lien privé est disponible, utilisez des points de terminaison privés pour sécuriser les ressources liées à votre pipeline Azure Data Factory, par exemple Azure SQL Server. Avec Private Link, le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public.

* [Procédure de création d’un Azure-SSIS Integration Runtime](./create-azure-ssis-integration-runtime.md)

* [Procédure de création et de configuration d’un IR auto-hébergé](./create-self-hosted-integration-runtime.md)

* [Guide pratique pour créer un réseau virtuel](../virtual-network/quick-create-portal.md)

* [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

* [Joindre un Azure-SSIS Integration Runtime à un réseau virtuel](./join-azure-ssis-integration-runtime-virtual-network.md#virtual-network-configuration)

* [Présentation d’Azure Private Link](../private-link/private-link-overview.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Aide** : Utilisez Azure Security Center et suivez les suggestions de protection du réseau applicables au réseau virtuel et au groupe de sécurité réseau associé au déploiement de votre Integration Runtime.

En outre, activez les journaux de flux du groupe de sécurité réseau (NSG) pour le NSG qui protège le déploiement de votre Integration Runtime et envoyez les journaux dans un compte de stockage Azure pour l’audit du trafic.

Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

* [Présentation de la sécurité réseau assurée par Azure Security Center](../security-center/security-center-network-recommendations.md)

* [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Présentation de la sécurité réseau assurée par Azure Security Center](../security-center/security-center-network-recommendations.md)

* [Guide pratique pour activer et utiliser Traffic Analytics](../network-watcher/traffic-analytics.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Non applicable. Cette recommandation a trait à Azure App Service ou aux ressources de calcul hébergeant des applications web.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Aide** : Activez le Service Protection DDoS Standard sur les réseaux virtuels associés au déploiement de votre Integration Runtime à des fins de protection contre les attaques par déni de service distribué. Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

* [Guide pratique pour configurer la protection DDoS](../ddos-protection/manage-ddos-protection.md)

* [Présentation de la fonctionnalité Threat Intelligence intégrée à Azure Security Center](../security-center/azure-defender.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Aide** : Activez les journaux de flux du groupe de sécurité réseau (NSG) pour le NSG qui protège le déploiement de votre Integration Runtime et envoyez les journaux dans un compte de stockage Azure pour l’audit du trafic.

Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

* [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

* [Présentation de la sécurité réseau assurée par Azure Security Center](../security-center/security-center-network-recommendations.md)

* [Guide pratique pour activer et utiliser Traffic Analytics](../network-watcher/traffic-analytics.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions (IDS/IPS) basés sur le réseau

**Aide** : Si vous souhaitez inspecter le trafic sortant d’Azure-SSIS IR, vous pouvez acheminer le trafic partant d’Azure-SSIS IR vers une appliance de pare-feu local via un tunneling forcé Azure ExpressRoute ou vers une appliance virtuelle réseau de la Place de marché Azure qui prend en charge les fonctionnalités IDS/IPS. Si la détection et/ou la prévention des intrusions basées sur l’inspection de la charge utile ne font pas partie des exigences, vous pouvez utiliser le Pare-feu Azure avec Threat Intelligence.

* [Joindre un Azure-SSIS Integration Runtime à un réseau virtuel](./join-azure-ssis-integration-runtime-virtual-network.md)

* [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall)

* [Guide pratique pour déployer le Pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md)

* [Guide pratique pour configurer des alertes avec le Pare-feu Azure](../firewall/threat-intel.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Non applicable. Cette recommandation a trait à Azure App Service ou aux ressources de calcul hébergeant des applications web.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Aide** : Utilisez des balises de service de réseau virtuel pour définir des contrôles d’accès réseau sur un groupe de sécurité réseau (NSG) ou le pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de la balise de service (par exemple, DataFactoryManagement) dans le champ source ou de destination approprié d'une règle, vous pouvez autoriser ou refuser le trafic entrant pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

* [Présentation et utilisation des balises de service](../virtual-network/service-tags-overview.md)

* [Comprendre les balises de service spécifiques à Azure Data Factory](./join-azure-ssis-integration-runtime-virtual-network.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Aide** : Définissez et implémentez des configurations de sécurité standard pour les paramètres réseau et les ressources réseau associées à vos instances Azure Data Factory avec Azure Policy. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.DataFactory » et « Microsoft.Network » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de vos instances Azure Data Factory. Vous pouvez aussi utiliser des définitions de stratégie intégrées relatives au réseau ou à vos instances Azure Data Factory comme :
- DDoS Protection Standard doit être activé

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Exemples Azure Policy pour le réseau](../governance/policy/samples/built-in-policies.md#network) 

* [Guide pratique pour créer un blueprint Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Aide** : Utilisez des étiquettes pour les ressources liées à la sécurité réseau et au flux de trafic de vos instances Azure Data Factory afin de fournir des métadonnées et une organisation logique.

Utilisez l’une des définitions Azure Policy intégrées en lien avec l’étiquetage comme « Exiger une étiquette et sa valeur » pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Aide** : Utilisez le journal d’activité Azure pour superviser les configurations des ressources réseau et détecter les modifications de ces ressources associées à vos instances Azure Data Factory. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.

* [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

* [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Contrôle de sécurité : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Aide** : Microsoft conserve la source de temps utilisée pour les ressources Azure, par exemple Azure Data Factory, dans le cadre des horodatages des journaux.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : Ingérez les journaux via Azure Monitor pour agréger les données de sécurité générées par Azure Data Factory. Dans Azure Monitor, vous pouvez interroger l’espace de travail Log Analytics qui est configuré de manière à recevoir vos journaux d’activité Azure Data Factory. Utilisez les comptes de stockage Azure pour les hubs d’événements ou le stockage de journaux d’archivage/à long terme, qui permettent d’exporter des données vers d’autres systèmes.

Vous pouvez également activer et intégrer des données à Azure Sentinel ou à des outils tiers d’Informations de sécurité et gestion d'événements (SIEM). Vous pouvez également intégrer Azure Data Factory à Git pour tirer parti de plusieurs avantages du contrôle de code source, tels que la possibilité de suivre/d’auditer les modifications et la possibilité de restaurer des modifications qui introduisent des bogues.

* [Comment configurer les paramètres de diagnostic](../azure-monitor/platform/diagnostic-settings.md)

* [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

* [Guide pratique pour bien démarrer avec Azure Monitor et l’intégration d’une solution SIEM tierce](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

* [Contrôle de code source dans Azure Data Factory](./source-control.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Pour la journalisation d’audit du plan de contrôle, activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux vers un espace de travail Log Analytics, Azure Event Hubs ou un compte de stockage Azure pour archivage. À l’aide des données des journaux d’activité Azure, vous pouvez déterminer les éléments « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de contrôle pour vos ressources Azure.

Utilisez les paramètres de diagnostic pour configurer les journaux de diagnostic des ressources non dédiées au calcul dans Azure Data Factory, telles que les mesures et les données d’exécution de pipeline. Azure Data Factory stocke les données d’exécution du pipeline pendant 45 jours. Pour conserver ces données pendant une période plus longue, enregistrez vos journaux de diagnostic dans un compte de stockage à des fins d’audit ou d’inspection manuelle et spécifiez la durée de rétention en jours. Vous pouvez également diffuser les journaux dans Azure Event Hubs ou les envoyer dans un espace de travail Log Analytics pour analyse.

* [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/activity-log.md)

* [Comprendre les journaux de diagnostic Azure Data Factory](./monitor-using-azure-monitor.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Aide** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, vous pouvez utiliser Azure Monitor pour collecter des données à partir de la machine virtuelle. L’installation de l’extension de machine virtuelle Log Analytics permet à Azure Monitor de collecter des données à partir de vos machines virtuelles Azure. Azure Security Center peut alors fournir une analyse du journal des événements de sécurité pour les machines virtuelles. Compte tenu du volume de données généré par le journal des événements de sécurité, elles ne sont pas stockées par défaut.

Si votre organisation souhaite conserver les données du journal des événements de sécurité, elles peuvent être stockées dans une couche de collecte des données, à partir de laquelle elles peuvent être interrogées dans Log Analytics.

* [Procédure de collecte de données à partir de machines virtuelles Azure avec Azure Monitor](../azure-monitor/learn/quick-collect-azurevm.md)

* [Collecte de données dans Azure Security Center](../security-center/security-center-enable-data-collection.md#data-collection-tier)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Aide** : Activez les paramètres de diagnostic pour Azure Data Factory. Si vous choisissez de stocker les journaux dans un espace de travail Log Analytics, définissez la période de conservation de votre espace de travail Log Analytics en fonction des obligations réglementaires de votre organisation. Utilisez les comptes de stockage Azure pour le stockage à long terme/d’archivage.

* [Procédure d’activation des journaux de diagnostic dans Azure Data Factory](./monitor-using-azure-monitor.md)

* [Définir les paramètres de conservation des journaux pour les espaces de travail Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Activez les paramètres de diagnostic pour Azure Data Factory et envoyez les journaux dans un espace de travail Log Analytics. Log Analytics permet d’analyser et de superviser vos journaux pour détecter les comportements anormaux. Examinez régulièrement les résultats. Veillez également à activer les paramètres de diagnostic pour les magasins de données liés à vos déploiements Azure Data Factory. Reportez-vous à la ligne de base de sécurité de chaque service pour obtenir des conseils sur l’activation des paramètres de diagnostic.

Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, activez également les paramètres de diagnostic pour la machine virtuelle.

Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.

* [Schéma Log Analytics](./monitor-using-azure-monitor.md#schema-of-logs-and-events)

* [Procédure de collecte de données à partir d’une machine virtuelle Azure avec Azure Monitor](../azure-monitor/learn/quick-collect-azurevm.md)

* [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Aide** : Vous pouvez déclencher des alertes sur les indicateurs de performance pris en charge dans Data Factory en accédant à la section Alertes et métriques dans Azure Monitor.

Configurez les paramètres de diagnostic pour Azure Data Factory et envoyez les journaux dans un espace de travail Log Analytics. Au sein de votre espace de travail Log Analytics, configurez les alertes pour qu’elles se déclenchent lorsqu’un ensemble prédéfini de conditions se produisent. Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.

En outre, veillez à activer les paramètres de diagnostic pour les services liés à vos magasins de données. Vous pouvez vous référer à la ligne de base de sécurité de chaque service pour obtenir des conseils.

* [Alertes dans Azure Data Factory](./monitor-visually.md#alerts)

* [Page de toutes les mesures prises en charge](../azure-monitor/platform/metrics-supported.md)

* [Comment configurer les alertes dans un espace de travail Log Analytics](../azure-monitor/platform/alerts-log.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, vous pouvez utiliser Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles, et configurer vos machines virtuelles de manière à consigner les événements dans un compte de stockage Azure. Configurez un espace de travail Log Analytics de manière à ingérer les événements des comptes de stockage et créer des alertes, le cas échéant. Suivez les recommandations faites dans Azure Security Center : « Calcul et applications ».

* [Guide pratique pour configurer Microsoft Antimalware pour les services cloud et les machines virtuelles](../security/fundamentals/antimalware.md)

* [Procédure d’activation de l’analyse au niveau de l’invité pour les machines virtuelles](../cost-management-billing/cloudyn/azure-vm-extended-metrics.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Aide** : Non applicable ; Azure Data Factory ne traite pas et ne génère pas de journaux liés au DNS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Conseils** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, vous pouvez activer les journaux d'audit de ligne de commande. Azure Security Center fournit une analyse du journal des événements de sécurité pour les machines virtuelles Azure. Security Center approvisionne Microsoft Monitoring Agent pour toutes les machines virtuelles Azure prises en charge et toutes celles nouvellement créées lorsque l’approvisionnement automatique est activé ou vous pouvez installer l’agent manuellement. L’agent active l’événement de création de processus 4688 et le champ CommandLine à l’intérieur de l’événement 4688. Les nouveaux processus créés sur la machine virtuelle sont enregistrés par le journal des événements et analysés par les services de détection Security Center.

* [Collecte de données dans Azure Security Center](../security-center/security-center-enable-data-collection.md#data-collection-tier)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Dans Azure Data Factory, assurez-vous de suivre et de rapprocher régulièrement l’accès utilisateur. Pour créer des instances Data Factory, le compte d’utilisateur que vous utilisez pour vous connecter à Azure doit être membre des rôles Contributeur ou Propriétaire, ou administrateur de l’abonnement Azure.

En outre, au niveau du locataire, Azure Active Directory (AD) comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent être interrogés. Le module Azure AD PowerShell permet d’effectuer des requêtes ad hoc afin de découvrir les comptes membres des groupes d’administration qui disposent d’un accès administratif au plan de contrôle de vos instances Azure Data Factory.

Bien qu’Azure AD soit la méthode recommandée pour administrer l’accès utilisateur, gardez à l’esprit que si vous exécutez Integration Runtime sur une machine virtuelle Azure, votre machine virtuelle peut également posséder des comptes locaux. Les comptes locaux et de domaine doivent être revus et gérés, normalement avec un encombrement minimal. En outre, nous vous conseillons de consulter le gestionnaire d’identité privilégié pour la fonctionnalité Juste à temps afin de réduire la disponibilité des autorisations d’administration.

* [Roles and permissions for Azure Data Factory](./concepts-roles-permissions.md) (Rôles et autorisations pour Azure Data Factory)

* [Informations sur Privileged Identity Manager](../active-directory/privileged-identity-management/pim-deployment-plan.md)

* [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

* [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

* [Informations pour les comptes locaux](../active-directory/devices/assign-local-admin.md#manage-the-device-administrator-role)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Aide** : Azure Active Directory utilise Azure Active Directory (AD) pour fournir l’accès au portail Azure et à la console Azure Data Factory. Azure AD n’utilise pas le concept des mots de passe par défaut, cependant vous êtes chargé de modifier ou de ne pas autoriser les mots de passe par défaut pour les applications personnalisées ou tierces.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures d’exploitation standard autour de l’utilisation de comptes d’administration dédiés pour accéder au plan de contrôle Azure (portail Azure), ainsi qu’à la console Azure Data Factory. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration dans Azure AD.

En outre, pour vous aider à suivre les comptes d’administration dédiés, vous pouvez utiliser des recommandations d’Azure Security Center ou des stratégies Azure intégrées, telles que les suivantes :
- Plusieurs propriétaires doivent être affectés à votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, les comptes administrateur sur les machines virtuelles Azure peuvent également être configurés avec Azure Privileged Identity Manager (PIM). Azure Privileged Identity Manager fournit plusieurs options, telles que l’élévation Juste à temps, l’authentification multifacteur et les options de délégation pour que les autorisations ne soient disponibles que pour des périodes spécifiques et nécessitent l’approbation d’une deuxième personne.

* [Présentation de l’identité et de l’accès Azure Security Center](../security-center/security-center-identity-access.md)

* [Utilisation d’Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Informations sur Privileged Identity Manager](../active-directory/privileged-identity-management/pim-deployment-plan.md)

* [Roles and permissions for Azure Data Factory](./concepts-roles-permissions.md) (Rôles et autorisations pour Azure Data Factory)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Aide** : L’inscription d’une application Azure (principal de service) permet de récupérer un jeton que votre application ou fonction peut utiliser pour interagir avec vos coffres Recovery Services et y accéder.

* [Appels d’API REST Azure](/rest/api/azure/#how-to-call-azure-rest-apis-with-postman)

* [Inscription de votre application cliente (principal du service) à l’aide d’Azure AD](/rest/api/azure/#register-your-client-application-with-azure-ad)

* [Informations sur l’API Azure Recovery Services](/rest/api/recoveryservices/)

* [Informations sur l’API REST pour Azure Data Factory](/rest/api/datafactory/)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Aide** : Activez l’authentification multifacteur (MFA) Azure Active Directory et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

* [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

* [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils** : Utilisez des stations de travail disposant d’un accès privilégié avec Multi-Factor Authentication (MFA) configuré pour vous connecter aux ressources Azure et les configurer.

* [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils** : Utilisez les rapports de sécurité Azure Active Directory pour générer des journaux et des alertes lorsque des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement. Utiliser Azure Security Center pour superviser les activités liées aux identités et aux accès.

Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, vous pouvez, en outre, intégrer votre machine virtuelle à Azure Sentinel. Microsoft Azure Sentinel est une solution native cloud et évolutive de type SIEM (Security Information and Event Management) et SOAR (Security Orchestrated Automated Response). Azure Sentinel assure une analyse de sécurité intelligente et fournit des informations sur les menaces dans l’ensemble de l’entreprise. Elle constitue une solution unique pour la détection des alertes, la visibilité des menaces, la chasse proactive et la réponse face aux menaces.

* [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../active-directory/identity-protection/overview-identity-protection.md)

* [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../security-center/security-center-identity-access.md)

* [Procédure d’intégration d’Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Aide** : Utilisez des emplacements nommés à accès conditionnel pour autoriser l’accès uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

* [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Une fabrique de données peut être associée à une identité managée pour les ressources Azure, laquelle représente cette même fabrique de données. Vous pouvez utiliser cette identité managée pour l’authentification Azure SQL Database. La fabrique en question peut accéder à votre base de données et copier des données depuis ou vers celle-ci à l’aide de cette identité.

Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, vous pouvez utiliser des identités managées pour vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, notamment Key Vault, sans informations d’identification dans votre code. Votre code, qui s’exécute sur une machine virtuelle, peut utiliser une identité managée pour faire une demande de jetons d’accès pour les services qui prennent en charge l’authentification Azure AD.

* [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

* [Que sont les identités managées pour les ressources Azure ?](../active-directory/managed-identities-azure-resources/overview.md)

* [Copier et transformer des données dans Azure SQL Database à l’aide d’Azure Data Factory](./connector-azure-sql-database.md)

* [Procédure de configuration et de gestion de l’authentification Azure Active Directory avec Azure SQL Database](../azure-sql/database/authentication-aad-configure.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory (AD) fournit des journaux pour vous aider à découvrir les comptes obsolètes. De plus, utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seules les utilisateurs appropriés continuent de bénéficier d’un accès.

Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, vous devez examiner les groupes de sécurité locaux et les utilisateurs pour vous assurer qu’il n’existe pas de comptes inattendus susceptibles de compromettre le système.

* [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

* [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml)

* [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Aide** : Vous avez accès aux activités de connexion Azure Active Directory (AD), aux sources des journaux d’événements à risque et d’audit, ce qui vous permet de les intégrer à un outil SIEM/de supervision. Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure AD et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes de journal souhaitées dans un espace de travail Log Analytics.

Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, intégrez cette machine virtuelle à Azure Sentinel. Microsoft Azure Sentinel est une solution native cloud et évolutive de type SIEM (Security Information and Event Management) et SOAR (Security Orchestrated Automated Response). Azure Sentinel assure une analyse de sécurité intelligente et fournit des informations sur les menaces dans l’ensemble de l’entreprise. Elle constitue une solution unique pour la détection des alertes, la visibilité des menaces, la chasse proactive et la réponse face aux menaces.

* [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

* [Autoriser l’accès aux ressources Event Hubs à l’aide d’Azure Active Directory](../event-hubs/authorize-access-azure-active-directory.md)

* [Procédure d’intégration d’Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez Azure Active Directory (AD) comme système central d’authentification et d’autorisation pour vos ressources Azure Data Factory, telles qu’Azure SQL Database ou des machines Virtuelles Azure. Pour les écarts du comportement de connexion au compte sur le plan de contrôle (le Portail Azure), utilisez Azure AD Identity Protection et les fonctionnalités de détections de risques pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. Vous pouvez également ingérer des données dans Azure Sentinel pour approfondir votre examen.

* [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

* [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

* [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

* [Configurer et gérer l’authentification Azure Active Directory avec SQL](../azure-sql/database/authentication-aad-configure.md?tabs=azure-powershell)

* [Activer l'authentification Azure Active Directory pour Azure-SSIS Integration Runtime](./enable-aad-authentication-azure-ssis-ir.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : Dans le cadre des scénarios de support où Microsoft a besoin d’accéder aux données client, Azure Customer Lockbox fournit une interface qui permet aux clients de consulter et d’approuver ou refuser les demandes d’accès aux données client. Notez que même si Azure Lockbox n’est pas disponible pour Azure Data Factory proprement dit, Azure Lockbox prend en charge Azure SQL Database et les Machines virtuelles Azure.

* [Présentation de Customer Lockbox](../security/fundamentals/customer-lockbox-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

Utilisez la fonctionnalité de découverte et classification des données Azure SQL Database. Découverte et classification des données offre des fonctionnalités avancées intégrées à Azure SQL Database pour la découverte, la classification, l’étiquetage &amp; protégeant les données sensibles dans vos bases de données.

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

* [Utilisation de la découverte et de la classification pour Azure SQL Server](../azure-sql/database/data-discovery-and-classification-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Les Integration Runtime doivent être séparés par un réseau/sous-réseau virtuel et étiquetés de manière appropriée.

 Vous pouvez également utiliser des points de terminaison privés pour effectuer l’isolement réseau. Un point de terminaison privé Azure est une interface réseau qui vous connecte de façon privée et sécurisée à un service basé sur la technologie Azure Private Link. Private Endpoint utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel.

* [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

* [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

* [Présentation de Private Link](../private-link/private-endpoint-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : Pour les sources de données (telles qu’Azure SQL Database) qui stockent ou traitent des informations sensibles pour votre déploiement d’Azure Data Factory, marquez les ressources associées comme sensibles à l’aide d’étiquettes.

Lorsque Private Link est disponible, utilisez des points de terminaison privés pour sécuriser les ressources liées à votre pipeline Azure Data Factory. Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public. Vous pouvez également réduire le risque d’exfiltration de données en configurant un ensemble strict de règles de trafic sortant sur un groupe de sécurité réseau (NSG) et en associant ce groupe de sécurité réseau à votre sous-réseau.

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

* [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

* [Présentation d’Azure Private Link](../private-link/private-link-overview.md)

* [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Aide** : Tous les transferts de données entre les services de déplacement des données dans Data Factory et une banque de données cloud s’effectuent via un canal HTTPS ou TLS sécurisé, si la banque de données cloud prend en charge HTTPS ou TLS. La version du protocole TLS utilisée est 1.2.

Toutes les connexions à Azure SQL Database et à Azure Synapse Analytics doivent être chiffrées (via SSL/TLS) lorsque les données sont en transit depuis et vers la base de données. Lorsque vous créez un pipeline à l’aide de JSON, ajoutez la propriété de chiffrement et définissez sa valeur sur True dans la chaîne de connexion. Pour le stockage Azure, vous pouvez utiliser HTTPS dans la chaîne de connexion.

* [Comprendre le chiffrement en transit avec Azure Data Factory](./data-movement-security-considerations.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Aide** : Si vous utilisez Azure Data Factory pour copier et transformer vos instances Azure SQL Database, utilisez la fonctionnalité de découverte et de classification des données d’Azure SQL Database. Découverte et classification des données offre des fonctionnalités avancées intégrées à Azure SQL Database pour la découverte, la classification, l’étiquetage &amp; protégeant les données sensibles dans vos bases de données.

Les fonctionnalités de découverte et de classification des données ne sont pas encore disponibles pour les autres services Azure.

* [Utilisation de la découverte et de la classification des données pour Azure SQL Server](../azure-sql/database/data-discovery-and-classification-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : utilisez le contrôle d’accès en fonction du rôle Azure (RBAC Azure) pour contrôler l’accès au plan de contrôle Azure Data Factory (portail Azure).

Pour créer des instances Data Factory, le compte d’utilisateur que vous utilisez pour vous connecter à Azure doit être membre des rôles Contributeur ou Propriétaire, ou administrateur de l’abonnement Azure.

Pour vos sources de données Data Factory, telles qu’Azure SQL Database, reportez-vous à la référence de base de sécurité de ce service pour plus d’informations sur le RBAC Azure.

* [Comment configurer Azure RBAC](../role-based-access-control/role-assignments-portal.md)

* [Roles and permissions for Azure Data Factory](./concepts-roles-permissions.md) (Rôles et autorisations pour Azure Data Factory)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Aide** : Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour le Stockage Azure ou les ressources de calcul. Implémentez une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

* [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide** : Nous vous recommandons d’activer le mécanisme de chiffrement des données pour les magasins de données liés à vos déploiements Azure Data Factory. Vous pouvez vous référer à la ligne de base de sécurité de ce service pour plus d’informations sur le chiffrement des données au repos.

Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, les disques virtuels sur les machines virtuelles Windows sont chiffrés au repos à l’aide d’un chiffrement côté serveur ou d’Azure Disk Encryption (ADE). Azure Disk Encryption utilise la fonctionnalité BitLocker de Windows pour chiffrer des disques managés avec des clés gérées par le client au sein de la machine virtuelle invitée. Le chiffrement côté serveur avec des clés gérées par le client améliore l’utilisation de Azure Disk Encryption en vous permettant d’utiliser des types et des images de système d’exploitation pour vos machines virtuelles en chiffrant les données dans le service de stockage.

Vous pouvez stocker des informations d’identification ou des valeurs secrètes dans Azure Key Vault et les utiliser pendant l’exécution du pipeline pour les transmettre à vos activités. Vous pouvez également stocker les informations d’identification des magasins de données et calculs dans un coffre de clés Azure Key Vault. Azure Data Factory récupère les informations d’identification lors de l’exécution d’une activité qui utilise le magasin de données/calcul.

* [Comprendre le chiffrement au repos dans Azure Data Factory](./data-movement-security-considerations.md)

* [Chiffrement côté serveur de disques managés Azure](../virtual-machines/disk-encryption.md)

* [Azure Disk Encryption pour les machines virtuelles Windows](../virtual-machines/windows/disk-encryption-overview.md)

* [Utilisation des secrets Azure Key Vault dans les activités de pipeline](./how-to-use-azure-key-vault-secrets-pipeline-activities.md)

* [Procédure de stockage des informations d’identification dans Azure Key Vault](./store-credentials-in-key-vault.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor avec le journal des activités Azure pour créer des alertes lorsque des modifications sont apportées à Azure Data Factory et aux ressources associées.

* [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

* [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

* [Journalisation Azure Storage Analytics](../storage/common/storage-analytics-logging.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : Si vous utilisez Azure SQL Database comme magasin de données, activez Advanced Data Security pour Azure SQL Database et suivez les suggestions d’Azure Security Center relatives à l’évaluation des vulnérabilités sur vos instances Azure SQL Server.

Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, suivez les suggestions d’Azure Security Center relatives à l’évaluation des vulnérabilités sur vos machines virtuelles. Utilisez la solution de sécurité Azure recommandée ou tierce pour effectuer des évaluations de vulnérabilités pour vos machines virtuelles.

* [Procédure d’exécution de l’évaluation des vulnérabilités sur Azure SQL Database](../azure-sql/database/sql-vulnerability-assessment.md)

* [Procédure d’activation d’Advanced Data Security](../azure-sql/database/azure-defender-for-sql.md)

* [Implémenter les recommandations d'évaluation des vulnérabilités d'Azure Security Center](../security-center/deploy-vulnerability-assessment-vm.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Aide** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, utilisez la solution Azure Update Management pour gérer les mises à jour et les correctifs pour vos machines virtuelles. Update Management s’appuie sur le référentiel de mise à jour configuré localement pour corriger les systèmes Windows pris en charge. Des outils comme System Center Updates Publisher (Updates Publisher) permettent de publier des mises à jour personnalisées dans Windows Server Update Services (WSUS). Ce scénario permet à Update Management de corriger des machines qui utilisent Configuration Manager comme référentiel de mise à jour avec des logiciels tiers.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

* [Solution Update Management dans Azure](../automation/update-management/overview.md)

* [Gérer les mises à jour et les correctifs pour vos machines virtuelles Azure](../automation/update-management/manage-updates-for-vm.md)

* [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="53-deploy-automated-patch-management-solution-for-third-party-software-titles"></a>5.3 : Déployer une solution de gestion automatisée des correctifs des logiciels tiers

**Conseils** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, vous pouvez utiliser une solution de gestion des correctifs tierce. Vous pouvez utiliser la solution Azure Update Management pour gérer les mises à jour et les correctifs pour vos machines virtuelles. Update Management s’appuie sur le référentiel de mise à jour configuré localement pour corriger les systèmes Windows pris en charge. Des outils comme System Center Updates Publisher (Updates Publisher) permettent de publier des mises à jour personnalisées dans Windows Server Update Services (WSUS). Ce scénario permet à Update Management de corriger des machines qui utilisent Configuration Manager comme référentiel de mise à jour avec des logiciels tiers.

* [Solution Update Management dans Azure](../automation/update-management/overview.md)

* [Gérer les mises à jour et les correctifs pour vos machines virtuelles Azure](../automation/update-management/manage-updates-for-vm.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Aide** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, exportez les résultats de l’analyse à intervalles réguliers et comparez les résultats pour vérifier que les vulnérabilités ont été corrigées. Lorsque vous utilisez les suggestions de gestion des vulnérabilités proposées par Azure Security Center, vous pouvez faire pivoter le portail de la solution sélectionnée pour afficher les données d’analyse historiques.

* [Comprendre l’analyseur de vulnérabilité intégré pour machines virtuelles](../security-center/deploy-vulnerability-assessment-vm.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Aide** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, vous pouvez utiliser l’analyseur de vulnérabilité natif. L’analyseur de vulnérabilité inclus avec Azure Security Center est alimenté par Qualys. L’analyseur de Qualys est l’outil de référence pour identifier en temps réel les vulnérabilités dans vos machines virtuelles Azure.

Après avoir identifié les vulnérabilités, Security Center présente les résultats et informations connexes sous forme de recommandations. Les informations connexes incluent les étapes de correction, les CVE connexes, les scores CVSS et bien plus encore. Vous pouvez afficher les vulnérabilités identifiées pour un ou plusieurs abonnements, ou pour une machine virtuelle spécifique.

* [Analyseur de vulnérabilité intégré pour machines virtuelles](../security-center/deploy-vulnerability-assessment-vm.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (telles que calcul, stockage, réseau, ports et protocoles) dans vos abonnements. Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

* [Guide pratique pour créer des requêtes avec Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

* [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

* [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Aide** : Utilisez des étiquettes, des groupes d’administration et des abonnements distincts, le cas échéant, pour organiser et suivre les ressources Azure. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

Appliquez également des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions de stratégie intégrées suivantes :
- Types de ressources non autorisés
- Types de ressources autorisés

* [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

* [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

* [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Aide** : Définissez des ressources Azure approuvées et des logiciels approuvés pour les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements.

Utilisez Azure Resource Graph pour interroger/découvrir des ressources dans leurs abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Aide** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, tirez parti de l’inventaire des machines virtuelles Azure pour automatiser la collecte d’informations sur tous les logiciels des machines virtuelles. Azure Automation permet de contrôler totalement le déploiement, les opérations et la désaffectation des charges de travail et des ressources.

Remarque : Le nom, la version, l’éditeur et l’heure d’actualisation du logiciel sont disponibles sur le portail Azure. Pour avoir accès à la date d’installation et à d’autres informations, le client a demandé d’activer les diagnostics au niveau de l’invité et de placer les journaux des événements Windows dans un espace de travail Log Analytics.

* [Présentation d’Azure Automation](../automation/automation-intro.md)

* [Procédure d’activation de l’inventaire des machines virtuelles Azure](../automation/automation-tutorial-installed-software.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Aide** : Si vous utilisez votre Integration Runtime sur une machine virtuelle Azure, Azure Automation permet de contrôler totalement le déploiement, les opérations et la désaffectation des charges de travail et des ressources. Vous pouvez utiliser Change Tracking pour identifier tous les logiciels installés sur des machines virtuelles. Vous pouvez implémenter votre propre processus ou utiliser la configuration de l’état Azure Automation pour supprimer les logiciels non autorisés.

* [Présentation d’Azure Automation](../automation/automation-intro.md)

* [Suivre les changements dans votre environnement avec la solution Change Tracking](../automation/change-tracking/overview.md)

* [Présentation d'Azure Automation State Configuration](../automation/automation-dsc-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Aide** : Si vous utilisez votre Integration Runtime sur une machine virtuelle Azure, utilisez les contrôles d’application adaptatifs d’Azure Security Center pour vous assurer que seuls les logiciels autorisés s’exécutent et que l’exécution de tous les logiciels non autorisés est bloquée sur les machines virtuelles.

* [Guide pratique pour utiliser les contrôles d’application adaptatifs Azure Security Center](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :
- Types de ressources non autorisés
- Types de ressources autorisés

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés

**Aide** : Le contrôle d’application adaptatif est une solution de bout en bout intelligente et automatisée d’Azure Security Center qui vous permet de contrôler les applications qui peuvent s’exécuter sur vos machines Azure et non-Azure (Windows et Linux). Implémentez une solution tierce si cela ne répond pas aux exigences de votre organisation.

Notez que cela ne s’applique que si votre Integration Runtime s’exécute sur une machine virtuelle Azure.

* [Guide pratique pour utiliser les contrôles d’application adaptatifs Azure Security Center](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Configurez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

* [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Aide** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, selon le type de script, vous pouvez utiliser des configurations de système d’exploitation spécifiques ou des ressources tierces pour limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul Azure. Vous avez également la possibilité d’exploiter les contrôles d’application adaptatifs Azure Security Center pour faire en sorte que seuls les logiciels autorisés s’exécutent et que l’exécution de tous les logiciels non autorisés soit bloquée sur les machines virtuelles Azure.

* [Guide pratique pour contrôler l’exécution des scripts PowerShell dans des environnements Windows](/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-6)

* [Guide pratique pour utiliser les contrôles d’application adaptatifs Azure Security Center](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Aide** : Les applications à haut risque déployées dans votre environnement Azure peuvent être isolées à l’aide d’un réseau virtuel, d’un sous-réseau, d’abonnements, de groupes d’administration, etc. et suffisamment sécurisées avec un pare-feu Azure, un pare-feu d’applications web (WAF) ou un groupe de sécurité réseau.

* [Réseaux virtuels et machines virtuelles dans Azure](../virtual-machines/network-overview.md)

* [Qu’est-ce qu’un pare-feu Azure ?](../firewall/overview.md)

* [Qu’est-ce que le pare-feu d’applications web Azure ?](../web-application-firewall/overview.md)

* [Groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md)

* [Qu’est-ce que le réseau virtuel Azure ?](../virtual-network/virtual-networks-overview.md)

* [Organiser vos ressources avec des groupes d’administration Azure](../governance/management-groups/overview.md)

* [Guide de décision concernant les abonnements](/azure/cloud-adoption-framework/decision-guides/subscriptions/)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Définissez et implémentez des configurations de sécurité standard pour Azure Data Factory avec Azure Policy. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.DataFactory » pour créer des stratégies personnalisées d’audit ou d’application de la configuration de vos instances Azure Data Factory.

* [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Aide** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, utilisez la suggestion [Corriger les vulnérabilités dans les configurations de sécurité sur vos machines virtuelles] d’Azure Security Center pour gérer les configurations de sécurité sur toutes les ressources de calcul.

* [Suivi des recommandations d'Azure Security Center](../security-center/security-center-recommendations.md)

* [Procédure de correction des recommandations d’Azure Security Center](../security-center/security-center-remediate-recommendations.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

* [Présentation des effets d’Azure Policy](../governance/policy/concepts/effects.md)

* [Informations sur la création de modèles Azure Resource Manager](../virtual-machines/windows/ps-template.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, notez qu’il existe plusieurs options pour maintenir une configuration sécurisée pour le déploiement des machines virtuelles :
- via les modèles Azure Resource Manager : Il s’agit de fichiers JSON utilisés pour déployer une machine virtuelle à partir du portail Azure et un modèle personnalisé doit être conservé. Microsoft effectue la maintenance sur les modèles de base.
- Disque dur virtuel (VHD) personnalisé : Dans certains cas, il peut être nécessaire d’avoir des fichiers VHD personnalisés utilisés, par exemple, pour traiter des environnements complexes qui ne peuvent pas être gérés par d’autres moyens. - Azure Automation - Configuration de l’état : Une fois le système d’exploitation de base déployé, vous pouvez l’utiliser pour un contrôle plus détaillé des paramètres et l’appliquer à l’aide de l’infrastructure d’automatisation.

Pour la plupart des scénarios, les modèles de machine virtuelle de base de Microsoft combinés à Azure Automation Desired State Configuration peuvent permettre de satisfaire les exigences de sécurité.

* [Informations sur le téléchargement du modèle de machine virtuelle](../virtual-machines/windows/download-template.md)

* [Informations sur la création de modèles Azure Resource Manager](../virtual-machines/windows/ps-template.md)

* [Procédure de chargement d’un disque dur virtuel de machine virtuelle personnalisé dans Azure](/azure-stack/operator/azure-stack-add-vm-image?view=azs-1910)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous utilisez des définitions Azure Policy personnalisées, utilisez Azure DevOps ou Azure Repos pour stocker et gérer votre code en toute sécurité.

* [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

* [Documentation Azure Repos](/azure/devops/repos/index?view=azure-devops)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Aide** : Si vous avez recours à des images personnalisées, utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour garantir que seuls les utilisateurs autorisés ont accès à ces images. Pour les images conteneur, stockez-les dans Azure Container Registry et utilisez Azure RBAC pour vous assurer que seuls des utilisateurs autorisés peuvent y accéder.

Le rôle de contributeur de Data Factory ne peut pas être utilisé pour créer et gérer des fabriques de données, ni des ressources enfant à l’intérieur de celles-ci.

* [Présentation d’Azure RBAC](../role-based-access-control/rbac-and-directory-admin-roles.md)

* [Comprendre Azure RBAC pour Container Registry](../container-registry/container-registry-roles.md)

* [Comment configurer Azure RBAC](../role-based-access-control/quickstart-assign-role-user-portal.md)

* [Roles and permissions for Azure Data Factory](./concepts-roles-permissions.md) (Rôles et autorisations pour Azure Data Factory)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Aide** : Utilisez les définitions Azure Policy intégrées ainsi que les alias Azure Policy dans l’espace de noms « Microsoft.DataFactory » pour créer des stratégies personnalisées d’alerte, d’audit et d’application de configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer des outils de gestion de la configuration pour les systèmes d'exploitation

**Aide** : Cette suggestion peut s’appliquer si votre Integration Runtime s’exécute sur une machine virtuelle Azure. Azure Automation State Configuration est un service de gestion de la configuration pour les nœuds DSC dans n’importe quel cloud ou centre de données local. Il permet de faire évoluer des milliers d’ordinateurs rapidement et facilement à partir d’un emplacement central et sécurisé. Vous pouvez facilement intégrer des machines, leur affecter des configurations déclaratives et afficher des rapports montrant la conformité de chaque machine à l’état souhaité que vous avez spécifié.

* [Intégration des machines pour la gestion avec Azure Automation State Configuration](../automation/automation-dsc-onboarding.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Conseils** : Utilisez les définitions Azure Policy intégrées ainsi que les alias Azure Policy dans l’espace de noms « Microsoft.DataFactory » pour créer des stratégies personnalisées d’alerte, d’audit et d’application de configurations système. Utilisez une stratégie Azure Policy [auditer], [refuser] et [déployer s’il n’existe pas] afin d’appliquer automatiquement des configurations pour vos ressources Azure.

* [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Aide** : Cette suggestion peut s’appliquer si votre Integration Runtime s’exécute sur une machine virtuelle Azure. Azure Automation State Configuration est un service de gestion de la configuration pour les nœuds DSC dans n’importe quel cloud ou centre de données local. Il permet de faire évoluer des milliers d’ordinateurs rapidement et facilement à partir d’un emplacement central et sécurisé. Vous pouvez facilement intégrer des machines, leur affecter des configurations déclaratives et afficher des rapports montrant la conformité de chaque machine à l’état souhaité que vous avez spécifié.

* [Intégration des machines pour la gestion avec Azure Automation State Configuration](../automation/automation-dsc-onboarding.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : Utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets pour vos applications Cloud.

Vous pouvez également stocker des informations d’identification ou des valeurs secrètes dans Azure Key Vault et les utiliser pendant l’exécution du pipeline pour les transmettre à vos activités. Vérifiez que la suppression réversible est activée.

* [Intégration aux identités managées Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

* [Créer un coffre de clés](../key-vault/secrets/quick-create-portal.md)

* [Comment s’authentifier auprès de Key Vault](../key-vault/general/authentication.md)

* [Comment attribuer une stratégie d’accès Key Vault](../key-vault/general/assign-access-policy-portal.md)

* [Utiliser des secrets Azure Key Vault dans les activités de pipeline](./how-to-use-azure-key-vault-secrets-pipeline-activities.md)

* [Suppression réversible dans Azure Key Vault](../key-vault/general/soft-delete-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Aide** : Lors de la création d’une fabrique de données, une identité managée est créée en même temps que la fabrique. L’identité managée est une application managée qui est inscrite auprès d’Azure Active Directory et qui représente cette fabrique de données.

* [Identité managée pour Azure Data Factory](./data-factory-service-identity.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

* [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Aide** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, vous pouvez utiliser Microsoft Antimalware pour machines virtuelles Azure Windows afin de surveiller et défendre vos ressources en permanence.

* [Guide pratique pour configurer Microsoft Antimalware pour les services cloud et les machines virtuelles](../security/fundamentals/antimalware.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Aide** : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure App Service), mais il ne s’exécute pas sur votre contenu.

Pré-analysez les fichiers chargés sur des ressources Azure non liées au calcul, comme App Service, des Data Lake Storage, Stockage Blob, etc.

Utilisez la détection des menaces pour les services de données dans Azure Security Center pour détecter les programmes malveillants chargés sur les comptes de stockage.

* [Présentation de Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles](../security/fundamentals/antimalware.md)

* [Présentation de la détection des menaces pour les services de données d’Azure Security Center](../security-center/azure-defender.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Aide** : Lorsqu’il est déployé, Microsoft Antimalware pour Azure installe automatiquement les dernières signatures, plateformes et mises à jour du moteur. Suivez les recommandations faites dans Azure Security Center : « Calcul et applications » pour mettre à jour les signatures de tous les points de terminaison. La protection du système d’exploitation Windows peut encore être renforcée par une sécurité supplémentaire afin de limiter le risque d’attaques basées sur les virus ou les logiciels malveillants avec le service Microsoft Defender - Protection avancée contre les menaces, qui s’intègre à Azure Security Center.

* [Guide pratique pour déployer Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles](../security/fundamentals/antimalware.md)

* [Microsoft Defender - Protection avancée contre les menaces](/windows/security/threat-protection/microsoft-defender-atp/onboard-configure)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="data-recovery"></a>Récupération de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Conseils** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, activez la sauvegarde Azure et configurez la machine virtuelle, ainsi que la fréquence souhaitée et la période de rétention pour les sauvegardes automatiques.

Pour tous vos magasins de données, reportez-vous à la ligne de base de sécurité de ce service pour obtenir des suggestions sur la procédure d’exécution de sauvegardes régulières et automatisées.

* [Vue d’ensemble de la sauvegarde de machines virtuelles Azure](../backup/backup-azure-vms-introduction.md)

* [Sauvegarder une machine virtuelle Azure à partir des paramètres de la machine virtuelle](../backup/backup-azure-vms-first-look-arm.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Aide** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, activez la sauvegarde Azure et ciblez les machines virtuelles Azure, ainsi que la fréquence souhaitée et les périodes de rétention. Clés de sauvegarde gérées par le client dans Azure Key Vault.

Pour tous vos magasins de données, reportez-vous à la ligne de base de sécurité de ce service pour obtenir des suggestions sur la procédure d’exécution de sauvegardes régulières et automatisées.

* [Vue d’ensemble de la sauvegarde de machines virtuelles Azure](../backup/backup-azure-vms-introduction.md)

* [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure, assurez-vous d’effectuer régulièrement la restauration des données du contenu au sein de la sauvegarde Azure. Si nécessaire, testez le contenu de restauration vers un réseau local virtuel isolé. Testez régulièrement la restauration des clés sauvegardées gérées par le client.

Pour tous vos magasins de données, reportez-vous à la ligne de base de sécurité de ce service pour obtenir des instructions sur la validation des sauvegardes.

* [Guide pratique pour récupérer des fichiers à partir d’une sauvegarde de machines virtuelles Azure](../backup/backup-azure-restore-files-from-vm.md)

* [Guide pratique pour restaurer des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Si vous exécutez votre Integration Runtime sur une machine virtuelle Azure et que vous sauvegardez cette machine virtuelle avec Sauvegarde Azure, votre machine virtuelle est chiffrée au repos avec Storage Service Encryption (SSE). Sauvegarde Azure peut également sauvegarder des machines virtuelles Azure chiffrées avec Azure Disk Encryption. Azure Disk Encryption s’intègre avec les clés de chiffrement de lecteur BitLocker qui sont sauvegardées dans un coffre de clés en tant que secrets. Azure Disk Encryption s’intègre également avec les clés de chiffrement Azure Key Vault. Activez la suppression réversible dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.

* [Suppression réversible pour les machines virtuelles](../backup/backup-azure-security-feature-cloud.md)

* [Vue d’ensemble de la suppression réversible d’Azure Key Vault](../key-vault/general/soft-delete-overview.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

* [Comment configurer des automatisations de workflow dans Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

* [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez clairement les abonnements (par ex. production, non production) et créez un système d’attribution de noms pour identifier et classer les ressources Azure de façon claire.

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

* [Reportez-vous à la publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Conseils** : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

* [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser la sentinelle des alertes.

* [Comment configurer l’exportation continue](../security-center/continuous-export.md)

* [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

* [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Conseils** : 

* [Suivez les règles d’engagement de Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies Microsoft](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

* [Vous trouverez ici plus d’informations sur la stratégie de Microsoft, sur l’exécution de Red Teaming et sur les tests d’intrusion de site actif dans l’infrastructure, les services et les applications cloud gérés par Microsoft.](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)