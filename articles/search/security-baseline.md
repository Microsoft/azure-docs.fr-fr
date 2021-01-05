---
title: Ligne de base de la sécurité Azure pour Recherche cognitive Azure
description: La base de référence de sécurité Recherche cognitive Azure fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: search
ms.topic: conceptual
ms.date: 09/25/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 07ef3da9f67de92c3526b83222c1c8088660d7c7
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499864"
---
# <a name="azure-security-baseline-for-azure-cognitive-search"></a>Ligne de base de la sécurité Azure pour Recherche cognitive Azure

Cette base de référence de sécurité applique les conseils d’[ version 1.0](../security/benchmarks/overview.md) à la Recherche cognitive Azure. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Le contenu est regroupé par les **contrôles de sécurité** définis par Azure Security Benchmark et les conseils associés applicables à la Recherche cognitive Azure. Les **contrôles** ne s’appliquent pas à la Recherche cognitive Azure, ou le client a été exclu.

Pour voir comment Recherche cognitive Azure est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité de Recherche cognitive Azure](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Conseils** : Assurez-vous que tous les déploiements de sous-réseaux virtuels Microsoft Azure ont un groupe de sécurité réseau appliqué avec des règles pour implémenter un schéma d’accès reposant sur le principe du « moindre privilège ». Autorisez l’accès uniquement aux ports et plages d’adresses IP de confiance de votre application. Déployez Recherche cognitive Azure avec un point de terminaison privé Azure, dans la mesure du possible, pour activer l’accès privé à vos services à partir de votre réseau virtuel.

La Recherche cognitive prend également en charge des fonctionnalités de sécurité réseau supplémentaires pour la gestion des listes de contrôle d’accès réseau. Configurez votre service de recherche pour autoriser uniquement la communication avec des sources approuvées en limitant l’accès de plages d’adresses IP publiques spécifiques à l’aide de sa fonctionnalité de pare-feu.

- [Guide pratique pour configurer des points de terminaison privés pour Recherche cognitive Azure](./service-create-private-endpoint.md)

- [Comment configurer le pare-feu Recherche cognitive Azure](./service-configure-firewall.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Conseils** : La Recherche cognitive ne peut pas être déployée directement dans un réseau virtuel. Toutefois, si votre application cliente ou vos sources de données se trouvent dans un réseau virtuel, vous pouvez surveiller et journaliser le trafic pour ces composants dans le réseau, y compris les demandes envoyées à un service de recherche dans le cloud. Les recommandations standard incluent l’activation d’un journal de flux de groupe de sécurité réseau et l’envoi des journaux vers le stockage Azure ou un espace de travail Log Analytics. Vous pouvez éventuellement utiliser Traffic Analytics pour obtenir des informations sur les modèles de trafic.

- [Tutoriel : journaliser le trafic réseau à destination et en provenance d’une machine virtuelle à l’aide du portail Azure](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Traffic Analytics](../network-watcher/traffic-analytics.md)

- [Protéger vos ressources réseau](../security-center/security-center-network-recommendations.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Non applicable à Recherche cognitive. Cette recommandation concerne les applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Conseils** : La Recherche cognitive ne fournit pas de fonctionnalité spécifique pour combattre une attaque par déni de service distribuée, mais vous pouvez activer la protection DDoS standard sur les réseaux virtuels associés à votre service Recherche cognitive à des fins de protection générale.

- [Guide pratique pour configurer la protection DDoS](../ddos-protection/manage-ddos-protection.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Conseils** : Activez les journaux de flux pour les groupes de sécurité réseau protégeant les machines virtuelles Azure qui se connecteront à votre service Recherche cognitive. Envoyez les journaux dans un compte de stockage Azure pour l’audit du trafic. 

Si cela s'avère nécessaire pour analyser une activité anormale, activez la capture de paquets Network Watcher.

- [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Guide pratique pour activer Network Watcher](../network-watcher/network-watcher-create.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions (IDS/IPS) basés sur le réseau

**Conseils** : La Recherche cognitive ne prend pas en charge la détection d’intrusion réseau, mais à des fins d’atténuation des intrusions, vous pouvez configurer des règles de pare-feu pour spécifier les adresses IP acceptées par le service Recherche cognitive. Configurez un point de terminaison privé pour maintenir le trafic de recherche hors de l’Internet public.

- [Comment configurer des clés gérées par le client pour le chiffrement des données](./search-security-manage-encryption-keys.md)

- [Obtenir des informations de clé gérée par le client à partir d’index et de synonym maps (cartes de synonymes)](./search-security-get-encryption-keys.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Non applicable à Recherche cognitive. Cette recommandation concerne les applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : Utilisez des étiquettes de service, si vous utilisez des indexeurs et ensembles de compétences dans Recherche cognitive, pour représenter une plage d’adresses IP qui ont l’autorisation de se connecter à des ressources externes. 

Autorisez ou refusez le trafic vers les ressources en spécifiant le nom d’étiquette de service (par exemple, AzureCognitiveSearch) dans le champ source ou de destination approprié d’une règle. 

- [Balises de service du réseau virtuel](../virtual-network/service-tags-overview.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : Recherche cognitive n’a pas et ne dépend pas des ressources réseau par nature. Les applications clientes et les sources de données associées à votre application de recherche peuvent se trouver sur un réseau virtuel, mais le service de recherche n’est pas déployé sur le réseau. 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : Vous pouvez configurer Recherche cognitive avec un point de terminaison privé Azure pour intégrer votre service de recherche à un réseau virtuel.  Utilisez des étiquettes de ressource pour les groupes de sécurité réseau (NSG) et autres ressources liées à la sécurité réseau et au trafic. Pour les règles de groupe de sécurité réseau, utilisez le champ « Description » pour documenter les règles qui autorisent le trafic vers/à partir d’un réseau. 

Utilisez l’une des définitions Azure Policy intégrée en lien avec l’étiquetage, comme « Exiger une étiquette et sa valeur », pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées. 

Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes. 

- [Comment créer un point de terminaison privé pour Recherche cognitive](./service-create-private-endpoint.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

- [Comment créer un réseau virtuel Azure](../virtual-network/quick-create-portal.md)

- [Comment filtrer le trafic réseau avec les règles de groupes de sécurité réseau](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : Recherche cognitive n’a pas de composants de mise en réseau et n’en dépend d’aucun. Les configurations de ces ressources ne peuvent donc pas être surveillées.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Conseils** : Recherche cognitive ne prend pas en charge la configuration de vos propres sources de synchronisation de l’heure. Le service de recherche s’appuie sur les sources de synchronisation de l’heure de Microsoft et n’est pas exposé aux clients pour la configuration.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : Ingérez des journaux liés à Recherche cognitive par le biais d’Azure Monitor pour agréger les données de sécurité générées par les appareils de point de terminaison, les ressources réseau et d’autres systèmes de sécurité. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, et utilisez des comptes Stockage Azure pour le stockage à long terme et l’archivage.
Vous pouvez également activer et intégrer ces données dans Azure Sentinel ou une solution SIEM tierce.

- [Guide pratique pour bien démarrer avec Azure Monitor et l’intégration d’une solution SIEM tierce](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Les journaux de diagnostic et opérationnels fournissent des informations sur les opérations détaillées de Recherche cognitive et sont utiles pour surveiller le service et les charges de travail qui y accèdent.  Pour capturer les données de diagnostic, activez la journalisation en spécifiant où sont stockées les informations de journalisation.

- [Collecter et analyser des données de journal pour Recherche cognitive Azure](./search-monitor-logs.md)

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md) 

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : Non applicable à Recherche cognitive. non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Par défaut, les données d’historique qui alimentent les métriques de diagnostic sont conservées par Recherche cognitive pendant 30 jours. Pour une conservation plus longue activez le paramètre spécifiant l'option de stockage relative aux métriques et événements consignés persistants.

Dans Azure Monitor, définissez la période de rétention de votre espace de travail Log Analytics en fonction des réglementations de conformité de votre organisation. Utilisez les comptes de stockage Azure pour le stockage à long terme et l’archivage. 

- [Modification de la période de conservation des données dans Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

- [Guide pratique pour configurer la stratégie de conservation des journaux de compte de Stockage Azure](../storage/common/storage-monitor-storage-account.md#configure-logging)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : Analysez et supervisez les journaux de vos demandes de votre service Recherche cognitive pour détecter tout comportement anormal. Utilisez Log Analytics d’Azure Monitor pour examiner les journaux et effectuer des requêtes sur leurs données. Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.

- [Collecter et analyser des données de journal pour Recherche cognitive](./search-monitor-logs.md)

- [Comment visualiser les données du journal de recherche dans Power BI](./search-monitor-logs-powerbi.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [En savoir plus sur Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md)

- [Guide pratique pour effectuer des requêtes personnalisées dans Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils** : Utilisez Security Center avec un espace de travail Log Analytics pour superviser les activités anormales détectées dans les journaux de sécurité et les événements et générer des alertes s’y rapportant. Vous pouvez également activer et intégrer les données dans Azure Sentinel.

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Guide pratique pour gérer les alertes dans Azure Security Center](../security-center/security-center-managing-and-responding-alerts.md)

- [Guide pratique pour générer une alerte sur des données de journal Log Analytics](../azure-monitor/learn/tutorial-response.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Conseils** : Non applicable à Recherche cognitive. Microsoft gère la solution anti-programme malveillant pour la plateforme sous-jacente.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Conseils** : Non applicable à Recherche cognitive. Elle ne produit pas et ne consomme pas les journaux DNS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Conseils** : Non applicable à Recherche cognitive. L’audit de ligne de commande n’est pas disponible pour Recherche cognitive.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) vous permet de gérer l’accès aux ressources Azure par le biais d’attributions de rôles. Vous pouvez affecter ces rôles à des utilisateurs, regrouper des principaux de service et des identités managées. Les rôles intégrés prédéfinis pour certaines ressources peuvent être inventoriés ou interrogés par le biais d’outils tels que Azure CLI, Azure PowerShell ou le Portail Azure.

Les rôles Recherche cognitive sont associés à des autorisations qui prennent en charge les tâches de gestion des niveaux de service.  Ces rôles n’accordent pas d’accès au point de terminaison de service. Pour l’accès aux opérations sur le point de terminaison (par exemple, la gestion d’index, le remplissage d’index et les requêtes sur les données de recherche), utilisez des clés API pour authentifier la demande.

- [Définir des rôles pour l’accès des administrateurs à Recherche cognitive Azure](./search-security-rbac.md)

- [Créer et gérer des clés API pour un service Recherche cognitive Azure](./search-security-api-keys.md)

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole)
- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : Non applicable à Recherche cognitive. Elle n’a pas de concept de mot de passe par défaut.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Recherche cognitive n’a pas le concept de compte d’administrateur de niveau local ou Azure Active Directory (Azure AD) qui peut être utilisé pour gérer les index et les opérations. 

Pour les opérations de gestion, utilisez les rôles Azure AD intégrés qui doivent être attribués explicitement. Invoquez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration.

- [Comment utiliser les rôles pour l’accès administratif dans Recherche cognitive](./search-security-rbac.md)

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : Utilisez l’authentification unique avec Azure Active Directory (Azure AD) pour accéder aux informations de service de recherche pour les opérations de gestion prises en charge par Azure Resource Manager. 

Établissez un processus pour réduire le nombre d’identités et d’informations d’identification en activant l’authentification unique pour le service avec les identités existantes de votre organisation.

- [Présentation de l’authentification SSO avec Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : Activez la fonctionnalité d’authentification multifacteur (MFA) Azure Active Directory (Azure AD) et suivez les recommandations relatives à la gestion des identités et des accès dans Security Center.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md) 

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md) 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils** : Utilisez une station de travail disposant d’un accès privilégié avec l’authentification multifacteur (MFA) configurée pour vous connecter aux ressources Azure et y accéder.

- [Comprendre les stations de travail sécurisées gérées par Azure](../active-directory/devices/concept-azure-managed-workstation.md)
 

- [Procédure d’activation d’Azure AD MFA](../active-directory/authentication/howto-mfa-getstarted.md)
 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils** : Utilisez les rapports de sécurité Azure Active Directory (Azure AD) et la supervision pour détecter les activités suspectes ou potentiellement dangereuses qui se produisent dans l’environnement. Utilisez Security Center pour superviser l’activité liée aux identités et aux accès.

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-only-from-approved-locations"></a>3.8 : Gérer les ressources Azure uniquement à partir d’emplacements approuvés

**Conseils** : Non applicable à Recherche cognitive. Elle ne prend pas en charge l’utilisation d’un emplacement approuvé comme condition d’accès.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Conseils** : Utilisez Azure Active Directory (Azure AD) comme système central d’authentification et d’autorisation pour les tâches de gestion des niveaux de service dans Recherche cognitive Azure. Les identités Azure AD n’accordent pas l’accès au point de terminaison du service de recherche.  L’accès à des opérations comme la gestion d’index, le remplissage d’index et les requêtes sur les données de recherche est disponible via les clés d’API.

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

- [Créer et gérer des clés API pour un service Recherche cognitive Azure](./search-security-api-keys.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory (Azure AD) fournit des journaux pour vous aider à découvrir les comptes obsolètes. Utilisez les révisions d’accès et des identités Azure AD pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seuls les utilisateurs appropriés continuent de bénéficier d’un accès. 

Examinez les journaux de diagnostic de Recherche cognitive pour étudier l’activité dans le point de terminaison du service de recherche, comme la gestion des index, le remplissage d’index et les requêtes.

- [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml)

- [Utilisation des révisions d’accès et des identités Azure AD](../active-directory/governance/access-reviews-overview.md)

- [Superviser les opérations et l’activité de Recherche cognitive Azure](./search-monitor-usage.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Conseils** : Accédez aux activités de connexion Azure Active Directory (Azure AD), aux sources des journaux d’événements à risque et d’audit, ce qui vous permet de les intégrer à un outil SIEM ou de supervision.

Simplifiez ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure AD et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Configurez les alertes souhaitées dans un espace de travail Log Analytics.

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md) 

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez les fonctionnalités d’Azure Active Directory (Azure AD) Identity Protection pour configurer des réponses automatisées aux actions suspectes détectées, liées aux identités d’utilisateur. Ingérez des données dans Azure Sentinel pour approfondir votre examen, comme requis.

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md) 

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md) 

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md) 

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : Non applicable à Recherche cognitive. Customer Lockbox ne prend pas en charge Recherche cognitive.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-protection"></a>Protection de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Les ressources doivent être séparées par un réseau virtuel ou un sous-réseau, étiquetés de manière appropriée et sécurisés au sein d’un groupe de sécurité réseau ou de Pare-feu Azure. Les ressources de stockage ou de traitement des données sensibles doivent être isolées. Utilisez une liaison privée pour configurer un point de terminaison privé sur Recherche cognitive.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md) 

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

- [Comment créer un point de terminaison privé pour Recherche cognitive](./service-create-private-endpoint.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : utilisez une solution tierce de la Place de marché Azure dans les périmètres du réseau, qui surveille et bloque les transferts non autorisés d’informations sensibles tout en alertant les professionnels de la sécurité des informations.

Microsoft gère la plateforme sous-jacente et traite tout le contenu des clients en tant que contenu sensible et assure une protection contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md) 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Recherche cognitive chiffre les données en transit à l’aide de Transport Layer Security 1.2 et applique le chiffrement (SSL/TLS) à tout moment pour toutes les connexions. Cela garantit que toutes les données sont chiffrées « en transit » entre le client et le service.

- [Présentation du chiffrement en transit avec Azure](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit) 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Conseils** : Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour la Recherche cognitive. Implémentez une solution tierce si nécessaire pour la conformité. 

Microsoft gère la plateforme sous-jacente et traite tout le contenu des clients en tant que contenu sensible et assure une protection contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="46-use-azure-rbac-to-manage-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : Pour l’administration des services, utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour gérer l’accès aux clés et à la configuration. Pour les opérations de contenu, comme l’indexation et les requêtes, Recherche cognitive utilise des clés à la place d’un modèle de contrôle d’accès basé sur l’identité. Utilisez Azure RBAC pour contrôler l’accès aux clés.
- [Configurer le contrôle d'accès en fonction du rôle dans Azure](../role-based-access-control/role-assignments-portal.md) 

 
- [Comment utiliser les rôles pour l’accès administratif à Recherche cognitive](./search-security-rbac.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Conseils** : Non applicable à Recherche cognitive. Ces instructions sont destinées aux ressources de calcul. 

Microsoft gère l’infrastructure sous-jacente de la Recherche cognitive Azure, et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : Par défaut, le service Recherche cognitive chiffre automatiquement le contenu indexé au repos avec des clés gérées par Microsoft. Si vous avez besoin de davantage de protection, vous pouvez compléter le chiffrement par défaut avec une deuxième couche de chiffrement à l’aide de clés que vous créez et gérez dans Azure Key Vault.

- [Configurer des clés gérées par le client pour le chiffrement des données dans le service Recherche cognitive Azure](./search-security-manage-encryption-keys.md)

- [Présentation du chiffrement au repos dans Azure](../security/fundamentals/encryption-atrest.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes en cas de modifications d’instances de production Recherche cognitive Azure et d’autres ressources critiques ou associées.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

- [Comment créer des alertes pour les activités de Recherche cognitive](./search-monitor-logs.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : Actuellement non disponible pour Recherche cognitive.  Pour les clusters qui stockent le contenu du service de recherche, Microsoft est responsable de la gestion des vulnérabilités de ces clusters.

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Microsoft

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : Non applicable à Recherche cognitive. non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="53-deploy-an-automated-patch-management-solution-for-third-party-software-titles"></a>5.3 : Déployer une solution de gestion automatisée des correctifs de logiciels tiers

**Conseils** : Non applicable à Recherche cognitive. non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils** : Non applicable à Recherche cognitive. Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Recherche cognitive Azure.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : Non applicable à Recherche cognitive. Elle n’a pas de système de notation ou de scoring de risque standard en place pour les résultats d’analyse des vulnérabilités.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : Azure Resource Graph permet d’interroger/de découvrir toutes les ressources (telles que le calcul, le stockage, le réseau, les ports, les protocoles et ainsi de suite) dans vos abonnements.  

Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements.  

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md) 

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription) 

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en utilisant des métadonnées pour les organiser de façon logique dans une taxonomie.

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

**Conseils** : Définissez une liste de ressources Azure approuvées liées à l’indexation et au traitement des ensembles de compétences dans Recherche cognitive.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Nous vous recommandons de définir un inventaire des ressources Azure qui ont été approuvées pour utilisation conformément aux stratégies et normes de votre organisation au préalable, puis de surveiller les ressources Azure non approuvées avec Azure Policy ou Azure Resource Graph.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md) 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : Non applicable à Recherche cognitive. Ce guide est destiné aux ressources de calcul.

Nous vous recommandons de disposer d’un inventaire des applications logicielles qui ont été considérées comme approuvées conformément aux stratégies et normes de sécurité de votre organisation, et de surveiller les logiciels non approuvés installés sur vos ressources de calcul Azure.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : Non applicable à Recherche cognitive. non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : Non applicable à Recherche cognitive. Elle n’expose pas de ressources de calcul et ne permet pas l’installation d’applications logicielles sur ses ressources.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :

- Types de ressources non autorisés
- Types de ressources autorisés

Utilisez Azure Resource Graph pour interroger ou découvrir les ressources dans vos abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md) 

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés

**Conseils** : Non applicable à Recherche cognitive. Cette recommandation a trait aux applications s’exécutant sur des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Conseils** : Pour la gestion des services, utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ». 

Contrôlez l’accès aux clés utilisées pour authentifier les requêtes pour toutes les autres opérations, en particulier celles relatives au contenu avec Recherche cognitive.

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-in-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : Non applicable à Recherche cognitive. non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : Non applicable à Recherche cognitive. Cette recommandation concerne les applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Conseils** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.Search » pour créer des stratégies personnalisées d’audit ou d’application de la configuration de vos ressources Recherche cognitive Azure. Vous pouvez également utiliser des définitions Azure Policy intégrées pour vos services Recherche cognitive, comme :

- Activer la journalisation d’audit pour les ressources Azure

Azure Resource Manager est capable d’exporter le modèle au format JSON (JavaScript Object Notation), qui doit être examiné pour vérifier que les configurations répondent aux exigences de sécurité de votre organisation. 

Vous pouvez aussi utiliser les recommandations d’Azure Security Center comme ligne de base de configuration sécurisée pour vos ressources Azure. 

- [Contrôles de conformité réglementaire d’Azure Policy pour Recherche cognitive Azure](./security-controls-policy.md)

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Non applicable à Recherche cognitive. non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : Utilisez les effets Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Recherche cognitive Azure. 

Vous pouvez utiliser des modèles Azure Resource Manager pour appliquer la configuration de sécurité des ressources Azure requise par votre organisation. 

- [Comprendre les effets d'Azure Policy](../governance/policy/concepts/effects.md)

- [Contrôles de conformité réglementaire d’Azure Policy pour Recherche cognitive Azure](./security-controls-policy.md)

- [Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

- [Présentation des modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Non applicable à Recherche cognitive. non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous utilisez des définitions Azure Policy personnalisées, utilisez Azure DevOps ou Azure Repos pour stocker et gérer votre code en toute sécurité.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [Documentation Azure Repos](/azure/devops/repos/index)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Non applicable à Recherche cognitive. non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour les ressources de votre service Recherche cognitive Azure à l’aide d’Azure Policy. 

Utilisez des alias pour créer des stratégies personnalisées afin d’auditer ou d’appliquer la configuration réseau. Vous pouvez également utiliser des définitions de stratégie intégrées en lien avec vos ressources spécifiques. 

Par ailleurs, vous pouvez utiliser Azure Automation pour déployer les changements de configuration et gérer les exceptions de stratégie. 

- [Contrôles de conformité réglementaire d’Azure Policy pour Recherche cognitive Azure](./security-controls-policy.md)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer des outils de gestion de la configuration pour les systèmes d'exploitation

**Conseils** : Non applicable à Recherche cognitive. non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Conseils** : Utilisez Security Center pour effectuer des analyses de base de vos ressources de service Recherche cognitive.  En outre, utilisez Azure Policy pour alerter et auditer vos configurations des ressources. 

- [Corriger les recommandations dans Azure Security Center](../security-center/security-center-remediate-recommendations.md)

- [Contrôles de conformité réglementaire d’Azure Policy pour Recherche cognitive Azure](./security-controls-policy.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : Non applicable à Recherche cognitive. non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : Utilisez les identités managées Azure conjointement avec Azure Key Vault afin de simplifier la gestion des secrets pour vos applications cloud.
- [Guide pratique pour utiliser des identités managées pour des ressources Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md) 
- [Créer un coffre de clés](../key-vault/general/quick-create-portal.md) 

- [Fournir une authentification Key Vault avec une identité managée](../key-vault/general/assign-access-policy-portal.md) 

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils** : Utilisez une identité gérée Azure pour accorder à Recherche cognitive l’accès à d’autres services Azure, comme les sources de données d’indexeur et Key Vault, à l’aide d’une identité gérée automatiquement dans Azure Active Directory (Azure AD). Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Azure Key Vault, sans informations d’identification dans votre code. 

- [Configurer une connexion d’indexeur à une source de données à l’aide d’une identité managée](./search-howto-managed-identities-data-sources.md)

- [Configurer des clés gérées par le client pour le chiffrement des données à l’aide d’une identité managée](./search-security-manage-encryption-keys.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Non applicable à Recherche cognitive. Elle n’héberge pas de code et ne dispose d’aucune information d’identification.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-antimalware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Conseils** : Non applicable à Recherche cognitive. non applicable. Cette recommandation a trait aux ressources de calcul.

Le logiciel anti-programme malveillant Microsoft est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Recherche cognitive Azure), mais il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Conseils** : Pré-analysez tout contenu chargé sur des ressources Azure non liées au calcul, comme Recherche cognitive Azure, Stockage Blob, Azure SQL Database et ainsi de suite. 

Il vous incombe de pré-analyser tout contenu chargé vers des ressources Azure autres que de calcul. Microsoft ne peut pas accéder aux données client et ne peut donc pas effectuer d’analyses anti-programme malveillant du contenu client en votre nom.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-antimalware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont à jour

**Conseils** : Non applicable à Recherche cognitive. Elle n’autorise pas l’installation de solutions anti-programme malveillant sur ses ressources. Pour la plateforme sous-jacente, Microsoft gère la mise à jour des logiciels et signatures anti-programmes malveillants. 

Pour toutes les ressources de calcul appartenant à votre organisation et utilisées dans votre solution de recherche, suivez les recommandations relatives au calcul &amp; aux applications de Security Center pour vous assurer que tous les points de terminaison sont à jour avec les signatures les plus récentes. Pour Linux, utilisez une solution logicielle anti-programme malveillant tierce.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="data-recovery"></a>Récupération de données

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Conseils** : Le contenu stocké dans un service de recherche ne peut pas être sauvegardé via la sauvegarde Azure ou tout autre mécanisme intégré, mais vous pouvez reconstruire un index à partir du code source de l’application et des sources de données principales, ou créer un outil personnalisé pour récupérer et stocker le contenu indexé.

- [Exemple de sauvegarde-restauration d’index GitHub](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/index-backup-restore)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Conseils** : Recherche cognitive ne prend actuellement pas en charge la sauvegarde automatisée des données dans un service de recherche et la sauvegarde doit être effectuée via un processus manuel.  Vous pouvez également sauvegarder les clés gérées par le client dans Azure Key Vault. 

- [Sauvegarder et restaurer un index Recherche cognitive Azure](/samples/azure-samples/azure-search-dotnet-samples/azure-search-backup-restore-index/)

- [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : Recherche cognitive ne prend actuellement pas en charge la sauvegarde automatisée des données dans un service de recherche et la sauvegarde/restauration doit être effectuée via un processus manuel.  Effectuez régulièrement une restauration des données du contenu que vous avez sauvegardé manuellement pour garantir l’intégrité de bout en bout de votre processus de sauvegarde.

- [Sauvegarder et restaurer un index Recherche cognitive Azure](/samples/azure-samples/azure-search-dotnet-samples/azure-search-backup-restore-index/)

- [Guide pratique pour restaurer des clés du coffre de clés dans Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés gérées par le client

**Conseils** : Recherche cognitive ne prend actuellement pas en charge la sauvegarde automatisée des données dans un service de recherche et la sauvegarde doit être effectuée via un processus manuel.  Vous pouvez également sauvegarder les clés gérées par le client dans Azure Key Vault. 

Activez la suppression réversible et la protection contre la purge dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante. Si le Stockage Azure est utilisé pour stocker les sauvegardes manuelles, la suppression réversible vous permet d’enregistrer et de récupérer vos données en cas de suppression d’objets blob ou d’instantanés d’objets blob. 

- [Sauvegarder et restaurer un index Recherche cognitive Azure](/samples/azure-samples/azure-search-dotnet-samples/azure-search-backup-restore-index/)

- [Guide pratique pour activer la suppression réversible et la protection contre la purge dans Key Vault](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

- [Suppression réversible pour le service Stockage Blob Azure](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Aide** : Développez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou à l’analytique utilisées pour émettre l’alerte, ainsi qu’à l’intention malveillante estimée de l’activité à l’origine du déclenchement de l’alerte.

En outre, marquez les abonnements à l’aide d’étiquettes et créez un système de nommage pour identifier et classer les ressources Azure, en particulier celles qui traitent des données sensibles. Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

- [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md)

- [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

- [Consultez la publication du NIST – Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Instructions** : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Aide** : Exportez vos alertes et recommandations de Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Security Center pour diffuser en continu les alertes vers Azure Sentinel.

- [Comment configurer l’exportation continue](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

- [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : actuellement non disponible

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