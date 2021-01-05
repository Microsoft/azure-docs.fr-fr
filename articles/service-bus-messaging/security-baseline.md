---
title: Base de référence de sécurité Azure pour Service Bus
description: La base de référence de sécurité Service Bus fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: service-bus-messaging
ms.topic: conceptual
ms.date: 09/25/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 9dda02e483f3437237837a9b3b8bdcbcf84fcd1d
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96489596"
---
# <a name="azure-security-baseline-for-service-bus"></a>Base de référence de sécurité Azure pour Service Bus

La base de référence de sécurité Service Bus contient des recommandations qui vous aideront à améliorer l’état de la sécurité de votre déploiement. La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview-v1.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques. Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

Pour voir comment Service Bus est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité Service Bus](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels 

**Conseils** : L’intégration de Service Bus avec le service Azure Private Link permet un accès privé sécurisé aux fonctionnalités de messagerie à partir de charges de travail telles que les machines virtuelles liées à des réseaux virtuels. Créez une connexion de point de terminaison privé à votre espace de noms Service Bus. Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel. Sachant que l’ensemble du trafic à destination du service peut être routé via le point de terminaison privé, il n’y a aucun besoin de passerelles, d’appareils NAT, de connexions ExpressRoute ou VPN ou d’adresses IP publiques.

Vous pouvez également sécuriser votre espace de noms Azure Service Bus à l’aide de pare-feu. Azure Service Bus prend en charge les contrôles d’accès basés sur IP pour le pare-feu entrant. Vous pouvez définir des règles de pare-feu à l’aide du portail Azure, de modèles Azure Resource Manager ou par le biais d’Azure CLI ou d’Azure PowerShell.

- [Autoriser l’accès aux espaces de noms Azure Service Bus via des points de terminaison privés](private-link-service.md)

- [Autoriser l’accès à un espace de noms Azure Service Bus à partir d’adresses ou de plages d’adresses IP spécifiques](service-bus-ip-filtering.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des interfaces réseau

**Conseils** : Si vous utilisez des machines virtuelles Azure pour accéder à vos entités Service Bus, activez les journaux de flux de groupe de sécurité réseau (NSG) et envoyez les journaux vers un compte de stockage pour auditer le trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau. 

Utilisez Azure Security Center et suivez les recommandations de protection du réseau pour sécuriser vos ressources Service Bus dans Azure.

- [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Guide pratique pour activer et utiliser Traffic Analytics](../network-watcher/traffic-analytics.md)

- [Guide pratique pour activer Network Watcher](../network-watcher/network-watcher-create.md)

- [Présentation de la sécurité réseau fournie par Azure Security Center](../security-center/security-center-network-recommendations.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications avec des adresses IP connues comme étant malveillantes

**Conseils** : Activez le service Protection DDoS Standard sur les réseaux virtuels associés à vos espaces de noms Service Bus à des fins de protection contre les attaques par déni de service distribué (DDoS). Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

- [Guide pratique pour configurer la protection DDoS](../ddos-protection/manage-ddos-protection.md)

- [Fonctionnalité Threat Intelligence intégrée à Azure Security Center](../security-center/azure-defender.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Conseils** : Si vous utilisez des machines virtuelles Azure pour accéder à vos entités Service Bus, vous pouvez utiliser la capture de paquets Network Watcher pour analyser les activités anormales.

- [Guide pratique pour activer Network Watcher](../network-watcher/network-watcher-create.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions (IDS/IPS) basés sur le réseau

**Conseils** : Si vous utilisez des machines virtuelles Azure pour accéder à vos entités Service Bus, sélectionnez une offre de la Place de marché Azure qui prend en charge les fonctionnalités IDS/IPS avec des fonctionnalités d’inspection de charge utile. Si la détection et/ou la prévention des intrusions basées sur l’inspection de la charge utile ne sont pas nécessaires pour votre organisation, vous pouvez utiliser la fonctionnalité de pare-feu Azure intégrée d’Azure Service Bus. Vous pouvez limiter l’accès à votre espace de noms Service Bus pour une plage limitée d’adresses IP ou pour une adresse IP spécifique à l’aide de règles de pare-feu.

- [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall)

- [Comment ajouter une règle de pare-feu dans des espaces de noms Service Bus pour une adresse IP spécifiée](service-bus-ip-filtering.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : Utilisez les étiquettes de service de réseau virtuel pour définir les contrôles d’accès réseau sur les groupes de sécurité réseau ou le pare-feu Azure qui filtrent le trafic vers et depuis les ressources Service Bus. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de l’étiquette de service (par exemple ServiceBus) dans le champ de source ou de destination approprié d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent. 

- [Présentation et utilisation des balises de service](../virtual-network/service-tags-overview.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour les ressources réseau associées à vos espaces de noms Azure Service Bus à l’aide d’Azure Policy. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.ServiceBus » et « Microsoft.Network » pour créer des stratégies personnalisées d’audit ou d’application de la configuration réseau de vos espaces de noms Service Bus. Vous pouvez également utiliser des définitions de stratégie intégrée en lien avec Azure Service Bus, par exemple :

- Service Bus doit utiliser un point de terminaison de service de réseau virtuel
- Les journaux de diagnostic dans Service Bus doivent être activés

Vous pouvez également créer des définitions de stratégie personnalisées si les définitions intégrées ne répondent pas aux besoins de votre organisation.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Stratégie intégrée Azure pour l’espace de noms Service Bus](./policy-reference.md#azure-service-bus-messaging)

- [Exemples Azure Policy pour le réseau](../governance/policy/samples/built-in-policies.md#network)

- [Guide pratique pour créer un blueprint Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : Utilisez des étiquettes pour les réseaux virtuels et autres ressources liées à la sécurité réseau et au flux de trafic qui sont associés à vos espaces de noms Service Bus. Pour les règles de groupe de sécurité réseau, utilisez le champ « Description » pour spécifier les besoins de l’entreprise, la durée et d’autres informations descriptives pour toutes les règles qui autorisent le trafic vers ou à partir d’un réseau associé à vos espaces de noms Service Bus. 

Utilisez l’une des définitions de stratégie Azure intégrée en lien avec l’étiquetage, telles que « Exiger une étiquette et sa valeur » pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées. 

Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes. 

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md) 

- [Guide pratique pour créer un réseau virtuel](../virtual-network/quick-create-portal.md) 

- [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : Utilisez le journal d’activité Azure pour superviser les configurations des ressources réseau et détecter les changements des ressources réseau associées à Azure Service Bus. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : Ingérer les journaux via Azure Monitor pour agréger les données de sécurité générées par des ressources Service Bus. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, et configurez des comptes Stockage Azure pour le stockage à long terme ou l’archivage. Vous pouvez également configurer les journaux relatifs à Service Bus pour les envoyer à Azure Sentinel ou à un SIEM tiers.

- [Procédure de configuration des paramètres de diagnostic pour Azure Service Bus](service-bus-diagnostic-logs.md)

- [Présentation du journal d’activité Azure](../azure-monitor/platform/platform-logs-overview.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md) 

- [Guide pratique pour bien démarrer avec Azure Monitor et l’intégration d’une solution SIEM tierce](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Activez les paramètres de diagnostic pour votre espace de noms Azure Service Bus. Azure Service Bus prend actuellement en charge l’activité et les journaux opérationnels ou de diagnostic. Les journaux d’activité comportent des informations sur les opérations effectuées sur un travail. Les journaux de diagnostic fournissent des informations plus détaillées sur les opérations et les actions qui sont effectuées en relation avec votre espace de noms à l’aide de l’API ou via les clients de gestion sur le Kit de développement logiciel (SDK) du langage. Plus précisément, ces journaux d’activité capturent le type d’opération, notamment la création de la file d’attente, les ressources utilisées et l’état de l’opération.

- [Guide pratique pour activer les paramètres de diagnostic pour Azure Service Bus](service-bus-diagnostic-logs.md)

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/activity-log.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Dans Azure Monitor, définissez la période de conservation de votre espace de travail Log Analytics en fonction des obligations réglementaires de votre organisation pour capturer et examiner les incidents liés à Service Bus.

- [Définir les paramètres de conservation des journaux pour les espaces de travail Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : Analysez et supervisez les journaux pour détecter les comportements anormaux et examinez régulièrement les résultats liés à vos entités Service Bus. Utilisez Azure Monitor pour examiner les journaux et effectuer des requêtes sur leurs données liées à Service Bus.

- [En savoir plus sur l’espace de travail Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md)

- [Guide pratique pour effectuer des requêtes personnalisées dans Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseil** : utilisez Azure Security Center avec un espace de travail Log Analytics pour superviser les activités anormales détectées dans les journaux de sécurité et les événements et générer des alertes s’y rapportant. Vous pouvez également activer et intégrer les données dans Azure Sentinel.

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Guide pratique pour gérer les alertes dans Azure Security Center](../security-center/security-center-managing-and-responding-alerts.md)

- [Guide pratique pour générer une alerte sur des données de journal Log Analytics](../azure-monitor/learn/tutorial-response.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) vous permet de gérer l’accès aux ressources Azure par le biais d’attributions de rôles. Vous pouvez affecter ces rôles à des utilisateurs, regrouper des principaux de service et des identités managées. Les rôles intégrés prédéfinis pour Service Bus peuvent être inventoriés ou interrogés par le biais d’outils tels qu’Azure CLI, Azure PowerShell ou le portail Azure.

- [Rôles Azure intégrés pour Service Bus](authenticate-application.md#azure-built-in-roles-for-azure-service-bus)

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0) 

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : L’accès au plan de contrôle par Service Bus est contrôlé via Azure Active Directory (Azure AD). Azure AD n’intègre pas le concept des mots de passe par défaut.

L’accès au plan de données pour Service Bus est contrôlé par le biais d’Azure AD avec des identités managées ou des inscriptions d’applications ainsi que des signatures d’accès partagé. Les signatures d’accès partagé sont utilisées par les clients qui se connectent à vos espaces de noms Service Bus et peuvent être regénérées à tout moment.

- [Comprendre les signatures d’accès partagé pour Service Bus](service-bus-sas.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

En outre, pour vous aider à suivre les comptes d’administration dédiés, vous pouvez utiliser des recommandations d’Azure Security Center ou des stratégies Azure intégrées, telles que les suivantes :

- Plusieurs propriétaires doivent être affectés à votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

Vous pouvez également créer des définitions de stratégie personnalisées si les définitions intégrées ne répondent pas aux besoins de votre organisation.

- [Utilisation d’Azure Security Center pour surveiller l’identité et l’accès](../security-center/security-center-identity-access.md)

- [Utilisation d’Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4 : Utiliser l’authentification unique (SSO) Azure Active Directory

**Conseils** : Microsoft Azure offre la gestion du contrôle d’accès intégré pour les ressources et les applications basées sur Azure Active Directory (Azure AD). L’un des principaux avantages de l’utilisation d’Azure AD avec Azure Service Bus est que vous n’avez plus besoin de stocker vos informations d’identification dans le code. Au lieu de cela, vous pouvez demander un jeton d’accès OAuth 2.0 à partir de la Plateforme d’identité Microsoft. Le nom de ressource à utiliser pour demander un jeton est https://azure.microsoft.com/services/service-bus/. Azure AD authentifie le principal de sécurité (un utilisateur, un groupe ou un principal de service) qui exécute l’application. Si l’authentification réussit, Azure AD retourne un jeton d’accès à l’application et l’application peut ensuite l’utiliser pour autoriser les demandes vers les ressources Azure Service Bus.

- [Comment authentifier une application avec Azure AD pour accéder aux ressources Service Bus](authenticate-application.md)

- [Présentation de l’authentification unique avec Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : Activez Azure Active Directory Multi-Factor Authentication (MFA) et suivez les recommandations Azure Security Center sur la gestion des identités et des accès pour protéger vos ressources Service Bus.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3.6 : Utiliser des stations de travail sécurisées et gérées par Azure pour les tâches administratives

**Conseils** : Utilisez des stations de travail disposant d’un accès privilégié avec Multi-Factor Authentication (MFA) configuré pour se connecter aux ressources Service Bus et les configurer.

- [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils** : Utilisez les rapports de sécurité Azure Active Directory et la supervision pour détecter les activités suspectes ou potentiellement dangereuses qui se produisent dans l’environnement. Utiliser Azure Security Center pour superviser les activités liées aux identités et aux accès.

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Aide** : Utilisez des emplacements nommés Azure AD pour n’autoriser l’accès qu’à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions. 

- [Procédure de configuration des emplacements nommés Azure AD](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Conseils** : Utilisez Azure Active Directory (AD) comme système central d’authentification et d’autorisation pour des ressources Azure telles que Service Bus. Ainsi, le mécanisme de contrôle d’accès en fonction du rôle Azure (Azure RBAC) est applicable aux ressources administratives sensibles.

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

- [Autoriser l’accès aux ressources Service Bus à l’aide d’Azure Active Directory](authenticate-application.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory (Azure AD) fournit des journaux pour vous aider à découvrir les comptes obsolètes. De plus, utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seules les utilisateurs appropriés continuent de bénéficier d’un accès.

En outre, effectuez une rotation régulière de la signature d’accès partagé de vos espaces de noms Service Bus.

- [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml)

- [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

- [Comprendre les signatures d’accès partagé pour les espaces de noms Service Bus](service-bus-sas.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Conseils** : Vous avez accès aux activités de connexion Azure Active Directory (Azure AD), aux sources des journaux d’événements à risque et d’audit, ce qui vous permet de les intégrer à Azure Sentinel ou à un outil SIEM tiers.

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure AD et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Ensuite, dans Azure Monitor, vous pouvez configurer les alertes de journal souhaitées pour certaines actions qui se produisent dans les journaux.

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Autoriser l’accès aux ressources Service Bus à l’aide d’Azure Active Directory](authenticate-application.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12 : Alerter en cas d’écart de comportement de connexion à un compte

**Conseils** : Utilisez les fonctionnalités de protection des identités et de détection des risques d’Azure Active Directory pour configurer des réponses automatiques aux actions suspectes détectées en lien avec vos ressources Service Bus. Vous devez activer des réponses automatisées via Azure Sentinel pour implémenter les réponses de sécurité de votre organisation.

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : Non disponible ; Customer Lockbox n’est pas encore pris en charge pour Service Bus.

- [Liste des services pris en charge pour Customer Lockbox](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes sur des ressources associées à Service Bus pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et des groupes d’administration distincts pour le développement, les tests et la production. Les espaces de noms Service Bus doivent être séparés par un réseau virtuel avec des points de terminaison privés configurés et correctement étiquetés.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

- [Guide pratique pour créer un réseau virtuel](../virtual-network/quick-create-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : Quand vous utilisez des machines virtuelles pour accéder à vos entités Service Bus, utilisez des réseaux virtuels, des points de terminaison privés, un pare-feu Service Bus, des groupes de sécurité réseau et des étiquettes de service pour limiter les risques d’exfiltration de données.

Microsoft gère l’infrastructure sous-jacente d’Azure Service Bus et a implémenté des contrôles stricts pour empêcher la perte ou la divulgation des données des clients.

- [Configurer des règles de pare-feu IP pour des espaces de noms Azure Service Bus](service-bus-ip-filtering.md)

- [Autoriser l’accès à un espace de noms Azure Service Bus à partir de réseaux virtuels spécifiques](private-link-service.md)

- [Autoriser l’accès aux espaces de noms Azure Service Bus via des points de terminaison privés](private-link-service.md)

- [Comprendre les groupes de sécurité réseau et les étiquettes de service](../virtual-network/network-security-groups-overview.md)

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Par défaut, Azure Service Bus impose des communications chiffrées par le protocole TLS. Les versions 1.0, 1.1 et 1.2 de TLS sont actuellement prises en charge. Toutefois, TLS 1.0 et 1.1 étant en passe de dépréciation dans l’ensemble du secteur, utilisez TLS 1.2 ou une version ultérieure dans la mesure du possible.

- [Pour comprendre les fonctionnalités de sécurité de Service Bus, consultez Sécurité réseau](network-security.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Conseils** : Les fonctionnalités d’identification, de classification et de protection contre la perte des données ne sont pas encore disponibles pour Azure Service Bus. Implémentez une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4.6 : Utiliser le contrôle d’accès en fonction du rôle pour contrôler l’accès aux ressources

**Conseils** : Azure Service Bus prend en charge l’utilisation d’Azure Active Directory (Azure AD) pour autoriser les requêtes d’accès aux entités Service Bus. Azure AD vous permet d’utiliser le contrôle d’accès en fonction du rôle (Azure RBAC) pour accorder des autorisations à un principal de sécurité, qui peut être un utilisateur ou un principal de service d’application.

- [Comprendre les rôles Azure RBAC et disponibles pour Azure Service Bus](authenticate-application.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : Azure Service Bus prend en charge le chiffrement des données au repos à l’aide de clés gérées par Microsoft ou de clés gérées par le client. Cette fonctionnalité vous permet de créer, de faire pivoter, de désactiver et de révoquer l’accès aux clés gérées par le client qui sont utilisées pour chiffrer les données Azure Service Bus au repos.

- [Comment configurer les clés gérées par le client pour le chiffrement des données Azure Service Bus](configure-customer-managed-key.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes en cas de modifications sur des instances de production Azure Service Bus et autres ressources critiques ou associées.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : Utilisez Azure Resource Graph pour interroger et découvrir toutes les ressources (y compris les espaces de noms Azure Service Bus) au sein de vos abonnements. Vérifiez que vous disposez des autorisations (en lecture) appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent.

- [Guide pratique pour créer des requêtes avec Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des étiquettes, des groupes d’administration, voire des abonnements séparés, pour organiser et suivre les espaces de noms Azure Service Bus et les ressources associées. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Conseils** : Créez un inventaire des ressources Azure et logiciels approuvés pour les ressources de calcul en fonction des besoins de votre organisation.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients selon les définitions de stratégies intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

Utilisez également Azure Resource Graph pour interroger et découvrir des ressources dans les abonnements.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients selon les définitions de stratégies intégrées suivantes :

- Types de ressources non autorisés
- Types de ressources autorisés

Vous pouvez également créer des définitions de stratégie personnalisées si les définitions intégrées ne répondent pas aux besoins de votre organisation.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Configurez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

- [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour vos déploiements Azure Service Bus. Vous pouvez également utiliser des définitions de stratégie intégrées pour Azure Service Bus, par exemple :

- Les journaux de diagnostic dans Service Bus doivent être activés
- Service Bus doit utiliser un point de terminaison de service de réseau virtuel pour limiter le trafic réseau vers vos réseaux privés.

Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.ServiceBus » pour créer des stratégies personnalisées d’audit ou d’application de configurations.

- [Stratégies intégrées Azure pour Service Bus ](./policy-reference.md)

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources ou applications Service Bus.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Pour plus d’informations sur les effets d’Azure Policy](../governance/policy/concepts/effects.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.ServiceBus » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Conseils** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.ServiceBus » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. Utilisez les stratégies Azure Policy [auditer], [refuser] et [déployer s’il n’existe pas] pour appliquer automatiquement des configurations pour vos déploiements Azure Service Bus et les ressources associées.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** :  Pour les machines virtuelles Azure ou les applications web s’exécutant sur Azure App Service utilisées pour accéder à vos entités Service Bus, utilisez Managed Service Identity conjointement avec Azure Key Vault afin de simplifier et sécuriser la gestion des signatures d’accès partagé pour vos déploiements Azure Service Bus. Vérifiez que la suppression réversible est activée dans Key Vault.

- [Authentifier une identité managée avec Azure Active Directory pour accéder aux ressources Service Bus](service-bus-managed-service-identity.md)

- [Configurer les clés gérées par le client pour Service Bus](configure-customer-managed-key.md)

- [Créer un coffre de clés](../key-vault/general/quick-create-portal.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils** : Pour les machines virtuelles Azure ou les applications web s’exécutant sur Azure App Service utilisées pour accéder à vos entités Service Bus, utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser Azure Service Bus. Vérifiez que la suppression réversible est activée dans Key Vault.

Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure Active Directory (Azure AD). Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Azure Key Vault, sans informations d’identification dans votre code.

- [Authentifier une identité managée avec Azure Active Directory pour accéder aux ressources Service Bus](service-bus-managed-service-identity.md)

- [Configurer les clés gérées par le client pour Service Bus](configure-customer-managed-key.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Conseils** : Pré-analysez tout contenu chargé sur des ressources Azure non liées au calcul, comme Azure Service Bus, App Service, Data Lake Storage, Stockage Blob, Azure Database pour PostgreSQL, etc. Microsoft ne peut pas accéder à vos données dans ces instances.

Le logiciel anti-programme malveillant de Microsoft est activé sur l’hôte sous-jacent qui prend en charge les services Azure, mais il ne s’exécute pas sur le contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="data-recovery"></a>Récupération de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : Garantir des sauvegardes automatiques régulières

**Conseils** : Configurez la géo-reprise d’activité après sinistre pour Azure Service Bus. Si tout un centre de données ou une région Azure complète (si aucune zone de disponibilité n’est utilisée) connaît un temps d’arrêt, il est essentiel que le traitement des données puisse continuer dans les autres régions ou centres de données. Pour cette raison, la géo-reprise d’activité après sinistre et la géoréplication sont des fonctionnalités importantes pour les entreprises. Azure Service Bus prend en charge la géorécupération d’urgence et la géoréplication au niveau de l’espace de noms.

- [Comprendre la géo-reprise d'activité après sinistre pour Azure Service Bus](service-bus-geo-dr.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Conseils** : Azure Service Bus fournit une fonctionnalité de chiffrement des données au repos à l’aide d’Azure Storage Service Encryption (Azure SSE). Service Bus utilise le service Stockage Azure pour stocker les données. Par défaut, toutes les données stockées avec ce service sont chiffrées à l'aide de clés gérées par Microsoft. Si vous utilisez Azure Key Vault pour stocker des clés gérées par le client, veillez à effectuer des sauvegardes automatisées régulières de vos clés.

Veillez à effectuer des sauvegardes automatisées régulières de vos secrets Key Vault avec la commande PowerShell suivante : Backup-AzKeyVaultSecret

- [Comment configurer les clés gérées par le client pour le chiffrement des données Azure Service Bus au repos](configure-customer-managed-key.md)

- [Comment sauvegarder des secrets Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultsecret)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : Testez la restauration des clés gérées par le client sauvegardées utilisées pour chiffrer les données Service Bus.

- [Comment configurer les clés gérées par le client pour le chiffrement des données Azure Service Bus au repos](configure-customer-managed-key.md)

- [Guide pratique pour restaurer des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Activez la suppression réversible dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante. Azure Service Bus nécessite que les fonctionnalités Suppression réversible et Ne pas vider soient configurées pour les clés gérées par le client.

- [Activation de la suppression réversible dans Key Vault](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

- [Configurer un coffre de clés avec des clés](../event-hubs/configure-customer-managed-key.md)

**Supervision d’Azure Security Center** : Oui

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

**Aide** : Azure Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez les abonnements à l’aide d’étiquettes et créez un système de nommage pour identifier et classer les ressources Azure, en particulier celles qui traitent des données sensibles. Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit. 

- [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md) 

- [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Aide** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

- [Publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Instructions** : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus. 

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations Azure Security Center en utilisant la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel. 

- [Comment configurer l’exportation continue](../security-center/continuous-export.md) 

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Aide** : 

Utilisez la fonctionnalité d’automatisation de workflow d’Azure Security Center pour déclencher automatiquement des réponses aux alertes et aux recommandations de sécurité afin de protéger vos ressources Azure. 

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