---
title: Base de référence de sécurité Azure pour Stockage Azure
description: Base de référence de sécurité Azure pour Stockage Azure
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/23/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 29e72f8358591614489be2731d16e89428ef388b
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96301361"
---
# <a name="azure-security-baseline-for-azure-storage"></a>Base de référence de sécurité Azure pour Stockage Azure

La base de référence de sécurité Azure pour Stockage Azure contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : sécurité réseau](../../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1 : Protéger les ressources à l'aide de groupes de sécurité réseau ou du Pare-feu Azure sur votre réseau virtuel

**Aide** : Configurez le Pare-feu de Stockage Azure en restreignant l’accès aux clients de plages d’IP publiques spécifiques, de certains réseaux virtuels sur Azure ou de ressources Azure spécifiques. Vous pouvez également configurer des points de terminaison privés afin que le trafic entre votre entreprise et le service de stockage passe exclusivement par des réseaux privés.

Remarque : Les comptes de stockage Classic ne prennent pas en charge les pare-feux et les réseaux virtuels.

- [Guide pratique pour configurer le Pare-feu de Stockage Azure](./storage-network-security.md#change-the-default-network-access-rule)

- [Guide pratique pour configurer des points de terminaison privés pour Stockage Azure](./storage-private-endpoints.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-vnet-subnet-and-nic-configuration-and-traffic"></a>1.2 : Surveiller et journaliser la configuration et le trafic de réseau virtuel, de sous-réseau et de carte réseau

**Aide** : Stockage Azure fournit un modèle de sécurité en couche. Vous pouvez limiter l’accès à votre compte de stockage aux demandes provenant d’adresses IP ou de plages d’adresses IP spécifiées, ou d’une liste de sous-réseaux dans un réseau virtuel Azure (VNet). Vous pouvez utiliser Azure Security Center et suivre les recommandations de protection du réseau pour sécuriser vos ressources réseau dans Azure. En outre, activez les journaux de flux NSG pour les réseaux virtuels/sous-réseaux configurés pour les comptes de stockage via le pare-feu des comptes de stockage et envoyez les journaux dans un compte de stockage pour l’audit du trafic. 

Notez que si vous avez des points de terminaison privés attachés à votre compte de stockage, vous ne pouvez pas configurer de règles de groupe de sécurité réseau (NSG) pour les sous-réseaux. 

- [Configurer des pare-feu et des réseaux virtuels dans Stockage Azure](./storage-network-security.md)

- [Guide pratique pour activer les journaux de flux NSG](../../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Présentation de la sécurité réseau fournie par Azure Security Center](../../security-center/security-center-network-recommendations.md)

- [Présentation des points de terminaison privés pour Stockage Azure](./storage-private-endpoints.md#known-issues)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Aide** : Non applicable. La recommandation concerne les applications web s’exécutant sur Azure App Service ou les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications avec des adresses IP connues comme étant malveillantes

**Aide** : Activez Advanced Threat Protection pour votre compte Stockage Azure. La protection avancée contre les menaces pour Stockage Azure offre une couche supplémentaire de veille de sécurité qui détecte les tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses des comptes de stockage. Ces alertes intégrées à Azure Security Center sont basées sur les activités pour lesquelles une communication réseau a été associée à une adresse IP qui a été résolue, afin de déterminer si l’adresse IP est une adresse IP à risque connue (par exemple, un cryptomineur connu) ou une adresse IP qui n’est pas reconnue comme étant risquée. Les alertes de sécurité sont déclenchées lorsque des anomalies se produisent dans l’activité. 

- [Activation d’Advanced Threat Protection](./azure-defender-storage-configure.md?tabs=azure-portal)

- [Présentation de la fonctionnalité Threat Intelligence intégrée à Azure Security Center](../../security-center/azure-defender.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5 : Journaliser les paquets et les flux réseau

**Aide** : La fonctionnalité de capture des paquets Network Watcher vous permet de créer des sessions de capture afin d’effectuer le suivi du trafic entre le compte de stockage et une machine virtuelle. Les filtres sont fournis pour la session de capture afin de vous garantir que vous capturez uniquement le trafic souhaité. La capture des paquets permet de diagnostiquer des anomalies du réseau de manières proactive et réactive. Elle permet aussi de collecter des statistiques réseau, d’obtenir des informations sur les intrusions, de déboguer des communications client-serveur, et bien plus. La possibilité de déclencher des captures de paquets à distance allège le fardeau de l’exécution manuelle de captures de paquets sur des machines virtuelles spécifiques, permettant ainsi d’économiser un temps précieux. 

- [Gestion des captures de paquets avec Azure Network Watcher à l’aide du portail](../../network-watcher/network-watcher-packet-capture-manage-portal.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions sur le réseau

**Conseils** : La protection avancée contre les menaces pour Stockage Azure offre une couche supplémentaire de veille de sécurité qui détecte les tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses des comptes de stockage. Les alertes de sécurité sont déclenchées lorsque des anomalies se produisent dans l’activité. Ces alertes de sécurité sont intégrées à Azure Security Center et envoyées par e-mail aux administrateurs d’abonnement avec les détails des activités suspectes et des recommandations sur la façon d’examiner et de corriger les menaces. 

- [Configuration de la protection avancée contre les menaces pour Stockage Azure](./azure-defender-storage-configure.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-your-web-applications"></a>1.7 : Gérer le trafic à destination de vos applications web

**Conseils** : Non applicable. La recommandation concerne les applications web s’exécutant sur Azure App Service ou les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Aide** : Pour les ressources des réseaux virtuels qui doivent accéder à votre compte de stockage, utilisez des étiquettes de service de réseau virtuel pour le réseau virtuel configuré afin de définir des contrôles d’accès réseau sur des groupes de sécurité réseau ou le Pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de l’étiquette de service (par exemple, Stockage) dans le champ de source ou de destination approprié d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent. 

Lorsque l’accès réseau doit être limité à des comptes de stockage spécifiques, utilisez des stratégies de point de terminaison de service de réseau virtuel.

- [Pour plus d’informations sur l’utilisation des étiquettes de service](../../virtual-network/service-tags-overview.md)

- [Pour plus d’informations sur les stratégies de point de terminaison de service de réseau virtuel pour Stockage Azure](../../virtual-network/virtual-network-service-endpoint-policies-overview.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Service

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les appareils réseau

**Aide** : Définissez et implémentez des configurations de sécurité standard pour les ressources réseau associées à votre compte de stockage Azure à l’aide d’Azure Policy. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.Storage » et « Microsoft.Network » pour créer des stratégies personnalisées d’audit ou d’application de la configuration réseau de vos ressources de compte de stockage. 

Vous pouvez également utiliser des définitions de stratégie intégrées associées au compte de stockage, par exemple : Les comptes de stockage doivent utiliser un point de terminaison de service de réseau virtuel 

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Exemples Azure Policy pour le stockage](../../governance/policy/samples/built-in-policies.md#storage)

- [Exemples Azure Policy pour le réseau](../../governance/policy/samples/built-in-policies.md#network)

- [Guide pratique pour créer un blueprint Azure](../../governance/blueprints/create-blueprint-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Documenter les règles de configuration du trafic

**Aide** : Utilisez des balises pour les groupes de sécurité réseau (NSG) et autres ressources liées à la sécurité réseau et au trafic. Concernant les règles NSG individuelles, utilisez le champ « Description » afin de spécifier le besoin métier et/ou la durée (etc.) pour toutes les règles qui autorisent le trafic vers/depuis un réseau. Utilisez l’une des définitions de stratégie Azure intégrée en lien avec l’étiquetage comme « Exiger une étiquette et sa valeur » pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées. Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes. 

- [Guide pratique pour créer et utiliser des étiquettes](../../azure-resource-manager/management/tag-resources.md)

- [Guide pratique pour créer un réseau virtuel](../../virtual-network/quick-create-portal.md)

- [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../../virtual-network/tutorial-filter-network-traffic.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les changements

**Aide** : Utilisez Azure Policy pour consigner les modifications de configuration des ressources réseau. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques. 

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des alertes dans Azure Monitor](../../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Contrôle de sécurité : journalisation et supervision](../../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-resource"></a>2.1 : Utiliser la ressource Synchronisation date/heure approuvée

**Aide** : Non applicable ; Microsoft gère les sources temporelles pour les comptes de stockage Azure.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Ingérez des journaux par le biais d’Azure Monitor pour agréger les données de sécurité générées par les appareils de point de terminaison, les ressources réseau et d’autres systèmes de sécurité. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, puis utilisez les comptes de stockage Azure pour le stockage à long terme/d’archivage, éventuellement avec des fonctionnalités de sécurité telles que le stockage immuable et les conservations de rétention appliquées.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../../azure-monitor/platform/diagnostic-settings.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Aide** : Azure Storage Analytics fournit des journaux pour les blobs, les files d’attente et les tables. Vous pouvez utiliser le Portail Azure afin de configurer les journaux à enregistrer pour votre compte. 

- [Configuration de la surveillance de votre compte de stockage Azure](./storage-monitor-storage-account.md#configure-monitoring-for-a-storage-account)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Aide** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Aide** : Lors du stockage de journaux des événements de sécurité dans le compte de stockage Azure ou dans l’espace de travail Log Analytics, vous pouvez définir la stratégie de rétention en fonction des besoins de votre organisation. 

- [Guide pratique pour configurer la stratégie de conservation des journaux de compte de Stockage Azure](./storage-monitor-storage-account.md#configure-logging)

- [Modification de la période de conservation des données dans Log Analytics](../../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Pour passer en revue les journaux de Stockage Azure, il existe les options habituelles, telles que les requêtes par le biais de l’offre Log Analytics, ainsi qu’une option unique permettant d’afficher les fichiers journaux directement. Dans Stockage Azure, les journaux sont stockés dans des blobs auxquels il faut accéder directement sur `http://accountname.blob.core.windows.net/$logs`. Le dossier de journalisation est masqué par défaut ; vous devez donc naviguer directement. Il ne s’affiche pas dans les commandes de liste.) 

En outre, activez Advanced Threat Protection pour votre compte Stockage Azure. La protection avancée contre les menaces pour Stockage Azure offre une couche supplémentaire de veille de sécurité qui détecte les tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses des comptes de stockage. Les alertes de sécurité sont déclenchées lorsque des anomalies se produisent dans l’activité. Ces alertes de sécurité sont intégrées à Azure Security Center et envoyées par e-mail aux administrateurs d’abonnement avec les détails des activités suspectes et des recommandations sur la façon d’examiner et de corriger les menaces. 

- [Journalisation et examen des données](./storage-analytics-logging.md#how-logs-are-stored)

- [Activation d’Advanced Threat Protection](./azure-defender-storage-configure.md?tabs=azure-portal)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer les alertes d’activité anormale

**Aide** : Dans Azure Security Center, activez Advanced Threat Protection pour le compte de stockage. Activez les paramètres de diagnostic le compte de stockage et envoyez les journaux à un espace de travail Log Analytics. Intégrez votre espace de travail Log Analytics à Azure Sentinel, car cela fournit une solution SOAR (Security Orchestration Automated Response). Cela permet de créer des playbooks (solutions automatisées) utilisables pour corriger des problèmes de sécurité. 

- [Guide pratique pour intégrer Azure Sentinel](../../sentinel/quickstart-onboard.md)

- [Guide pratique pour gérer les alertes dans Azure Security Center](../../security-center/security-center-managing-and-responding-alerts.md)

- [Guide pratique pour générer une alerte sur des données de journal Log Analytics](../../azure-monitor/learn/tutorial-response.md)

- [Journalisation Azure Storage Analytics](./storage-analytics-logging.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide** : Utilisez Azure Security Center et activez la protection contre les menaces pour Stockage Azure afin de détecter les chargements de programmes malveillants sur Stockage Azure à l’aide de l’analyse de réputation de hachage et les accès suspects à partir d’un nœud de sortie Tor actif (proxy d’anonymisation). 

- [Configuration de la protection avancée contre les menaces pour Stockage Azure](./azure-defender-storage-configure.md?tabs=azure-portal)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Aide** : La solution Azure DNS Analytics (préversion) dans Azure Monitor rassemble des informations sur une infrastructure DNS concernant la sécurité, les performances et les opérations. Actuellement, elle ne prend pas en charge les comptes de stockage Azure. Vous pouvez toutefois utiliser une solution de journalisation DNS tierce. 

- [Rassemblement d’informations sur votre infrastructure DNS avec la solution DNS Analytics (préversion)](../../azure-monitor/insights/dns-analytics.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer la journalisation d’audit en ligne de commande

**Aide** : Non applicable. Le benchmark a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Azure AD comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent être interrogés. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration. 

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole)

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Changer les mots de passe par défaut quand cela est possible

**Conseils** : Ni les comptes de stockage Azure ni Azure Active Directory ne suivent le concept de mots de passe vierges ou par défaut. Stockage Azure implémente un modèle de contrôle d’accès qui prend en charge le contrôle d’accès en fonction du rôle (Azure RBAC), ainsi que les clés partagées et les signatures d’accès partagé (SAP). Une caractéristique de cette authentification SAP et par clé partagée est qu’aucune identité n’est associée à l’appelant. Par conséquent, aucune permission basée sur une autorisation de principal de sécurité ne peut être accordée. 

- [Autorisation de l’accès aux données dans Stockage Azure](./storage-auth.md)

- [Présentation des principaux de sécurité et du contrôle d’accès pour le compte Stockage Azure](./storage-introduction.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Aide** : Créez des procédures de fonctionnement standard autour de l’utilisation de comptes d’administration dédiés ayant accès à votre compte de stockage. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration. 

Vous pouvez également activer JIT (Juste-à-temps)/JEA (Just-Enough-Access) à l’aide de rôles privilégiés Azure AD Privileged Identity Management pour les services Microsoft et Azure ARM. 

- [Présentation de l’identité et de l’accès Azure Security Center](../../security-center/security-center-identity-access.md)

- [Présentation de Privileged Identity Management](../../active-directory/privileged-identity-management/index.yml)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4 : Utiliser l’authentification unique (SSO) Azure Active Directory

**Aide** : Dans la mesure du possible, utilisez l’authentification unique (SSO) d’Azure Active Directory au lieu de configurer des informations d’identification autonomes individuelles par service. Suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center. 

- [Présentation de l’authentification unique avec Azure AD](../../active-directory/manage-apps/what-is-single-sign-on.md)

- [Autorisation de l’accès aux données dans Stockage Azure](./storage-auth.md)

- [Autorisation de l’accès aux blobs et aux files d’attente à l’aide d’Azure Active Directory](./storage-auth-aad.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="35-use-multifactor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : Activez l’authentification multifacteur Azure Active Directory et suivez les recommandations de gestion des identités et des accès dans Azure Security Center pour protéger vos ressources de compte de stockage. 

- [Guide pratique pour activer l’authentification MFA dans Azure](../../active-directory/authentication/howto-mfa-getstarted.md)

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../../security-center/security-center-identity-access.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des machines dédiées (stations de travail à accès privilégié) pour toutes les tâches administratives

**Aide** : Utilisez des stations de travail avec accès privilégié avec l’authentification multifacteur paramétrée pour vous connecter aux ressources de compte de stockage et les configurer. 

- [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Guide pratique pour activer l’authentification MFA dans Azure](../../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration

**Conseils** : Envoyez les alertes de détection des risques d’Azure Security Center à Azure Monitor et configurez des alertes/notifications personnalisées à l’aide de groupes d’actions. Activez Advanced Threat Protection pour le compte de stockage Azure de façon à générer des alertes en cas d’activité suspecte. Utilisez également les détections de risque Azure AD pour visualiser les alertes et les rapports sur les comportements des utilisateurs à risque. 

- [Configuration d’Advanced Threat Protection pour le compte de stockage Azure](./azure-defender-storage-configure.md)

- [Présentation des détections de risques Azure AD](../../active-directory/identity-protection/overview-identity-protection.md)

- [Configuration des groupes d’actions pour générer des alertes et des notifications personnalisées](../../azure-monitor/platform/action-groups.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resource-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir d’emplacements approuvés uniquement

**Aide** : Utilisez des emplacements nommés à accès conditionnel pour autoriser l’accès uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions. 

- [Guide pratique pour configurer des emplacements nommés dans Azure](../../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Utiliser Azure Active Directory (Azure AD) comme système d’authentification et d’autorisation central. Azure fournit un contrôle d’accès basé sur un rôle Azure (Azure RBAC) pour contrôler l’accès d’un client aux ressources dans un compte de stockage.  Comme meilleure pratique, utilisez si possible les informations d’identification Azure AD plutôt que la clé de compte, qui peut être plus facilement compromise. Lorsque la conception de votre application nécessite des signatures d’accès partagé pour être en mesure d’accéder au Stockage Blob, utilisez les informations d’identification Azure AD pour créer, si possible, des signatures d’accès partagé (SAP) de délégation d’utilisateur pour profiter d’une sécurité supérieure.

- [Création et configuration d’une instance Azure AD](../../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

- [Utilisation du fournisseur de ressources Stockage Azure pour accéder aux ressources de gestion](./authorization-resource-provider.md)

- [Configuration de l’accès aux données Blob et File d’attente Azure avec Azure RBAC dans le portail Azure](./storage-auth-aad-rbac-portal.md)

- [Autorisation de l’accès aux données dans Stockage Azure](./storage-auth.md)

- [Accorder un accès limité aux ressources du Stockage Azure à l’aide des signatures d’accès partagé (SAP)](./storage-sas-overview.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Revoir et harmoniser régulièrement les accès utilisateur

**Aide** : Passez en revue les journaux Azure Active Directory pour identifier les comptes obsolètes, qui peuvent inclure ceux ayant des rôles d’administration de compte de stockage. Utilisez également des révisions d’accès d’identité Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise susceptibles d’être utilisées pour accéder aux ressources de compte de stockage et les attributions de rôles. L’accès des utilisateurs doit être passé en revue régulièrement pour vérifier que seuls les utilisateurs appropriés bénéficient d’un accès permanent.  

Vous pouvez également utiliser une signature d’accès partagé (SAP) pour fournir un accès délégué sécurisé aux ressources de votre compte de stockage sans compromettre la sécurité de vos données. Vous pouvez contrôler, entre autres paramètres, les ressources auxquelles le client peut accéder, les autorisations dont il dispose pour ces ressources ainsi que la durée de validité de la SAP.

Passez également en revue l’accès en lecture anonyme aux conteneurs et aux blobs. Par défaut, un conteneur et les objets blob qu’il contient sont accessibles uniquement par un utilisateur qui dispose des autorisations appropriées. Vous pouvez utiliser Azure Monitor pour vous alerter des accès anonymes aux comptes de stockage à l’aide de la condition d’authentification anonyme.

Une façon efficace de réduire le risque d’un accès insoupçonné à un compte d’utilisateur consiste à limiter la durée d’accès que vous accordez aux utilisateurs. Les URI SAP limités dans le temps sont un moyen efficace de faire expirer automatiquement l’accès d’un utilisateur à un compte de stockage. De plus, la rotation fréquente des clés de compte de stockage est un moyen de s’assurer que les accès inattendus via des clés de compte de stockage sont d’une durée limitée.

- [Présentation des rapports Azure AD](../../active-directory/reports-monitoring/index.yml)

- [Affichage et modification de l’accès au niveau du compte de stockage Azure](./storage-auth-aad-rbac-portal.md)

- [Accorder un accès limité aux ressources du Stockage Azure à l’aide des signatures d’accès partagé (SAP)](./storage-sas-overview.md)

- [Gestion de l’accès en lecture anonyme aux conteneurs et aux objets blob](../blobs/anonymous-read-access-configure.md)

- [Surveiller un compte de stockage dans le portail Azure](./storage-monitor-storage-account.md)

- [Gestion des clés d’accès au compte de stockage](./storage-account-keys-manage.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés

**Conseils** : Utilisez Storage Analytics pour consigner des informations détaillées sur les requêtes ayant réussi ou échoué pour un service de stockage. Tous les journaux d’activité sont stockés dans des objets blob de blocs dans un conteneur nommé $logs, qui est automatiquement créé lorsque Storage Analytics est activé pour un compte de stockage.

Créez des paramètres de diagnostic pour les comptes d’utilisateur Azure Active Directory, en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes souhaitées dans un espace de travail Log Analytics. Pour surveiller les échecs d’authentification sur les comptes de stockage Azure, vous pouvez créer des alertes pour vous avertir lorsque certains seuils ont été atteints pour les mesures de ressources de stockage. Utilisez également Azure Monitor pour vous alerter des accès anonymes aux comptes de stockage à l’aide de la condition d’authentification anonyme.

- [Journalisation Azure Storage Analytics](./storage-analytics-logging.md)

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Configuration des alertes de mesures pour les comptes de stockage Azure](./storage-monitor-storage-account.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerter en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez les fonctionnalités de protection des identités et contre les risques d’Azure Active Directory pour configurer des réponses automatiques aux actions suspectes détectées liées à vos ressources de compte de stockage. Vous devez activer des réponses automatisées via Azure Sentinel pour implémenter les réponses de sécurité de votre organisation. 

- [Guide pratique pour afficher les connexions risquées Azure AD](../../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Guide pratique pour intégrer Azure Sentinel](../../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Aide** : Dans le cadre des scénarios de support où Microsoft a besoin d’accéder aux données client, Customer Lockbox (préversion pour le compte de stockage) fournit une interface qui permet aux clients de passer en revue et d’approuver ou de refuser les demandes d’accès aux données client. Microsoft ne nécessite pas ni ne demande l’accès aux secrets de votre organisation stockés dans le compte de stockage.

- [Présentation de Customer Lockbox](../../security/fundamentals/customer-lockbox-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Contrôle de sécurité : protection des données](../../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources de compte de stockage qui stockent ou traitent des informations sensibles. 

- [Guide pratique pour créer et utiliser des étiquettes](../../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Aide** : Implémentez une isolation en utilisant des abonnements, groupes d’administration et comptes de stockage distincts selon les domaines de sécurité (par exemple, l’environnement et la sensibilité des données).  Vous pouvez limiter votre compte de stockage pour contrôler le niveau d’accès à vos comptes de stockage selon les exigences de vos applications et environnements d’entreprise, en fonction du type et du sous-ensemble de réseaux utilisés. Quand des règles de réseau sont configurées, seules les applications demandant des données sur l’ensemble de réseaux spécifié peuvent accéder à un compte de stockage. Vous pouvez contrôler l’accès à Stockage Azure via Azure RBAC. Vous pouvez également configurer des points de terminaison privés pour améliorer la sécurité, car le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public. 

- [Guide pratique pour créer des abonnements Azure supplémentaires](../../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../../azure-resource-manager/management/tag-resources.md)

- [Configurer des pare-feu et des réseaux virtuels dans Stockage Azure](./storage-network-security.md)

- [Points de terminaison de service de réseau virtuel](../../virtual-network/virtual-network-service-endpoints-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Superviser et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : Pour les ressources de compte de stockage qui stockent ou traitent des informations sensibles, marquez les ressources comme sensibles à l’aide de balises. Pour réduire le risque de perte de données par exfiltration, limitez le trafic sortant pour les comptes de stockage Azure à l’aide du Pare-feu Azure. 

Utilisez également les stratégies de point de terminaison de service de réseau virtuel pour filtrer le trafic du réseau virtuel sortant vers les comptes de stockage Azure sur le point de terminaison de service et exfiltrer des données vers des comptes de stockage Azure spécifiques uniquement.

- [Configurer des pare-feu et des réseaux virtuels dans Stockage Azure](../../virtual-network/virtual-network-service-endpoint-policies-overview.md)

- [Stratégies de point de terminaison de service de réseau virtuel pour Stockage Azure](../../private-link/tutorial-private-endpoint-storage-portal.md)

- [Présentation de la protection des données client dans Azure](../../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Vous pouvez appliquer l’utilisation du protocole HTTPS en activant l’option Transfert sécurisé requis pour le compte de stockage. Les connexions utilisant le protocole HTTP seront refusées une fois cette option activée. En outre, utilisez Azure Security Center et Azure Policy pour appliquer le transfert sécurisé pour votre compte de stockage.

- [Transfert sécurisé requis dans Stockage Azure](./storage-require-secure-transfer.md)

- [Stratégies de sécurité Azure supervisées par Azure Security Center](../../security-center/policy-reference.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Aide** : Les fonctionnalités d’identification des données ne sont pas encore disponibles pour le compte de stockage Azure et les ressources associées. Implémentez une solution tierce si nécessaire à des fins de conformité. 

- [Présentation de la protection des données client dans Azure](../../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : Azure Active Directory (Azure AD) autorise les droits d’accès aux ressources sécurisées par le biais du contrôle d’accès en fonction du rôle Azure (Azure RBAC). Stockage Azure définit un ensemble de rôles intégrés Azure qui englobent les ensembles communs d’autorisations permettant d’accéder aux données blob ou de file d’attente. 

- [Attribution des rôles Azure pour un compte de stockage Azure](./storage-auth-aad-rbac-portal.md#assign-azure-roles-using-the-azure-portal)

- [Utilisation du fournisseur de ressources Stockage Azure pour accéder aux ressources de gestion](./authorization-resource-provider.md)

- [Configuration de l’accès aux données Blob et File d’attente Azure avec Azure RBAC dans le portail Azure](./storage-auth-aad-rbac-portal.md)

- [Guide pratique pour créer et configurer une instance AAD](../../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

- [Autorisation de l’accès aux données dans Stockage Azure](./storage-auth.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Aide** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide** : Le chiffrement de Stockage Azure est activé pour tous les comptes de stockage et ne peut pas être désactivé. Stockage Azure chiffre automatiquement vos données lors de leur conservation dans le cloud. Lorsque vous lisez des données depuis le stockage Azure, elles sont déchiffrées par le stockage Azure avant d’être retournées. Le chiffrement de Stockage Azure vous permet de sécuriser vos données au repos sans avoir à modifier le code existant ni à ajouter du code dans les applications. 

- [Présentation du chiffrement Stockage Azure au repos](./storage-service-encryption.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes qui se déclenchent lorsque des modifications sont apportées à des ressources de compte de stockage. Vous pouvez également activer la journalisation de Stockage Azure pour suivre la façon dont chaque demande effectuée auprès de Stockage Azure a été autorisée. Les journaux indiquent si une demande a été faite anonymement, en utilisant un jeton OAuth 2.0, avec une clé partagée ou avec une signature d’accès partagé. Utilisez également Azure Monitor pour vous alerter des accès anonymes aux comptes de stockage à l’aide de la condition d’authentification anonyme.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../../azure-monitor/platform/alerts-activity-log.md)

- [Journalisation Azure Storage Analytics](./storage-analytics-logging.md)

- [Configuration des alertes de mesures pour les comptes de stockage Azure](./storage-monitor-storage-account.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : Suivez les recommandations d’Azure Security Center pour auditer et surveiller en permanence la configuration de vos comptes de stockage. 

- [Recommandations de sécurité - Guide de référence](../../security-center/recommendations-reference.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Aide** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3 : Déployer une solution de gestion automatisée des correctifs logiciels tiers

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Aide** : Non applicable ; Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge les comptes de stockage.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classification des risques pour hiérarchiser la correction des vulnérabilités découvertes.

**Aide** : Utilisez les évaluations de risques par défaut (degré de sécurisation) fournies par Azure Security Center. 

- [Présentation du degré de sécurisation Azure Security Center](../../security-center/secure-score-security-controls.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1 : Utiliser la découverte de ressources Azure

**Aide** : Utilisez Azure Resource Graph pour interroger et découvrir toutes les ressources (y compris les comptes de stockage) au sein de vos abonnements. Vérifiez que vous disposez des autorisations (en lecture) appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent. 

- [Guide pratique pour créer des requêtes avec Azure Graph](../../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription)

- [Présentation d’Azure RBAC](../../role-based-access-control/overview.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des étiquettes aux ressources de compte de stockage en fournissant des métadonnées pour les organiser de façon logique par catégories. 

- [Guide pratique pour créer et utiliser des étiquettes](../../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des étiquettes, des groupes d’administration, voire des abonnements séparés, pour organiser et suivre les comptes de stockage et les ressources associées. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile. 

En outre, utilisez Advanced Threat Protection pour Stockage Azure afin de détecter les ressources Azure non autorisées. 

- [Guide pratique pour créer des abonnements Azure supplémentaires](../../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../../azure-resource-manager/management/tag-resources.md)

- [Configuration de la protection avancée contre les menaces pour Stockage Azure](./azure-defender-storage-configure.md?tabs=azure-portal)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-maintain-inventory-of-approved-azure-resources-and-software-titles"></a>6.4 : Tenir un inventaire des ressources Azure et logiciels approuvés

**Aide** : Vous devez créer un inventaire des ressources Azure approuvées et des logiciels approuvés en fonction des besoins de votre organisation. 


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Superviser les ressources Azure non approuvées

**Aide** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes : 

 - Types de ressources non autorisés 
 - Types de ressources autorisés 

Utilisez également Azure Resource Graph pour interroger/découvrir des ressources dans les abonnements. Cela peut être utile dans les environnements de haute sécurité, tels que ceux dotés de comptes de stockage. 

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Graph](../../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Détecter les applications logicielles non approuvées dans les ressources de calcul

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer les ressources Azure et les applications logicielles non approuvées

**Aide** : Le client peut empêcher la création ou l’utilisation de ressources à l’aide d’Azure Policy comme l’exigent les stratégies d’entreprise du client. 

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Aide** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes : 

- Types de ressources non autorisés 
- Types de ressources autorisés 

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-implement-approved-application-list"></a>6.10 : Implémenter une liste d’applications approuvées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager-via-scripts"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager via des scripts

**Aide** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ». Cela peut empêcher la création et la modification de ressources dans un environnement de haute sécurité, telles que celles dotées de comptes de stockage. 

- [Configuration de l’accès conditionnel pour bloquer l’accès à ARM](../../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Aide** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.Storage » pour créer des stratégies personnalisées d’audit ou d’application de la configuration de vos comptes de stockage. Vous pouvez également utiliser des définitions Azure Policy intégrées pour le compte de stockage Azure telles que : 

Auditer l'accès réseau non restreint aux comptes de stockage  
Déployer Advanced Threat Protection sur les comptes de stockage  
Les comptes de stockage doivent être migrés vers de nouvelles ressources Azure Resource Manager  
La sécurisation du transfert vers des comptes de stockage doit être activée  

Utilisez les recommandations d’Azure Security Center comme base de référence d’une configuration sécurisée pour vos comptes de stockage. 

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias)

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="72-establish-secure-configurations-for-your-operating-system"></a>7.2 : Établir des configurations sécurisées pour votre système d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="73-maintain-secure-configurations-for-all-azure-resources"></a>7.3 : Maintenir des configurations sécurisées pour toutes les ressources Azure

**Conseils** : Utilisez les commandes Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources de compte de stockage. 

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Présentation des effets d’Azure Policy](../../governance/policy/concepts/effects.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-configurations-for-operating-systems"></a>7.4 : Maintenir des configurations sécurisées pour les systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Utilisez Azure Repos pour stocker et gérer votre code de manière sécurisée, par exemple, les stratégies Azure personnalisées, les modèles Azure Resource Manager, les scripts Desired State Configuration, etc. Pour accéder aux ressources que vous gérez dans Azure DevOps, vous pouvez accorder ou refuser des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps, ou à Active Directory s’il est intégré à TFS.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [À propos des autorisations et des groupes dans Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7 : Déployer les outils de gestion de la configuration système

**Aide** : Tirez parti d’Azure Policy pour alerter, auditer et appliquer des configurations système pour le compte de stockage. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie. 

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation

**Aide** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la supervision de la configuration automatique pour les services Azure

**Conseils** : Tirez parti d’Azure Security Center pour effectuer des analyses de ligne de base pour vos ressources de compte de stockage Azure. 

- [Corriger les recommandations dans Azure Security Center](../../security-center/security-center-remediate-recommendations.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la supervision de la configuration automatique pour les systèmes d’exploitation

**Aide** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="711-securely-manage-azure-secrets"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Aide** : Le Stockage Azure chiffre automatiquement vos données lors de leur conservation dans le cloud. Vous pouvez utiliser des clés gérées par Microsoft pour le chiffrement du compte de stockage, ou vous pouvez gérer le chiffrement avec vos propres clés. Si vous utilisez des clés fournies par le client, vous pouvez tirer parti d’Azure Key Vault pour stocker les clés en toute sécurité. 

En outre, faites pivoter les clés de compte de stockage régulièrement pour limiter l’impact de la perte ou de la divulgation des clés de compte de stockage.

- [Chiffrement du stockage Azure pour les données au repos](./storage-service-encryption.md)

- [Gestion des clés d’accès au compte de stockage](./storage-account-keys-manage.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="712-securely-and-automatically-manage-identities"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Aide** : Autorisez l’accès aux blobs et files d’attente dans des comptes de stockage Azure à l’aide d’Azure Active Directory et des identités managées. Stockage File d’attente et Blob Azure prennent en charge l’authentification Azure Active Directory (Azure AD) avec des identités managées pour les ressources Azure. Les identités managées pour ressources Azure autorisent l’accès à des objets blob et à des données de files d’attente en utilisant les informations d’identification Azure AD d’applications s’exécutant dans Machines virtuelles Azure, d’applications de fonction, de groupe de machines virtuelles identiques, et d’autres services. En utilisant des identités managées pour ressources Azure et Azure AD Authentication, vous pouvez éviter de stocker des informations d’identification avec les applications qui s’exécutent dans le cloud. 

- [Guide pratique pour octroyer l’accès aux données de blob et de file d’attente Azure à l’aide d’une identité managée](./storage-auth-aad-rbac-portal.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault. 

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Aide** : Non applicable. Cette recommandation a trait aux ressources de calcul. Microsoft gère les logiciels anti-programme malveillant pour la plateforme sous-jacente.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Aide** : Utilisez la protection contre les menaces pour Stockage Azure afin de détecter les chargements de programmes malveillants sur Stockage Azure à l’aide de l’analyse de réputation de hachage et les accès suspects à partir d’un nœud de sortie Tor actif (proxy d’anonymisation). 

Vous pouvez également pré-analyser le contenu des programmes malveillants avant de les charger dans des ressources Azure sans calcul, telles qu’App Service, Data Lake Storage, Stockage Blob, etc., pour répondre aux besoins de votre organisation.

- [Configuration de la protection avancée contre les menaces pour Stockage Azure](./azure-defender-storage-configure.md?tabs=azure-portal)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont à jour

**Aide** : Non applicable. Cette recommandation a trait aux ressources de calcul. Microsoft gère les logiciels anti-programme malveillant pour la plateforme sous-jacente.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-recovery"></a>Récupération des données

*Pour plus d’informations, consultez [Contrôle de sécurité : récupération de données](../../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : Mettre en place des sauvegardes automatiques régulières

**Aide** : Les données de votre compte de stockage Microsoft Azure sont toujours répliquées automatiquement pour en garantir la durabilité et la haute disponibilité. Stockage Azure copie vos données afin qu’elles soient protégées contre les événements planifiés ou non, notamment les défaillances matérielles temporaires, les pannes de réseau ou de courant et les catastrophes naturelles massives. Vous pouvez choisir de répliquer vos données dans le même centre de données, dans des centres de données zonaux d’une même région ou entre des régions géographiques différentes. 

Vous pouvez également activer Azure Automation pour prendre régulièrement des instantanés de blobs.

- [Présentation de la redondance et des contrats de niveau de service Stockage Azure](./storage-redundancy.md)

- [Création d’un instantané de blob](/rest/api/storageservices/creating-a-snapshot-of-a-blob)

- [Vue d’ensemble d’Azure Automation](../../automation/automation-intro.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Aide** : Pour sauvegarder des données à partir de services de compte de stockage pris en charge, plusieurs méthodes sont disponibles, notamment l’utilisation d’azcopy ou d’outils tiers. Le stockage immuable pour le Stockage Blob Azure permet aux utilisateurs de stocker des objets de données critiques pour l’entreprise dans un état WORM (disque optique non réinscriptible). Cet état les rend non effaçables et immuables pour une durée spécifiée par l’utilisateur.

- [Bien démarrer avec AzCopy](./storage-use-azcopy-v10.md)

- [Définir et gérer des stratégies d’immuabilité pour le stockage Blob](../blobs/storage-blob-immutability-policies-manage.md?tabs=azure-portal)

Les clés gérées ou fournies par le client peuvent être sauvegardées dans Azure Key Vault à l’aide d’Azure CLI ou de PowerShell. 

- [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : Effectuez régulièrement une restauration de données de vos certificats, clés, comptes de stockage managés et secrets Key Vault avec les commandes PowerShell suivantes : 

Restore-AzKeyVaultCertificate ; Restore-AzKeyVaultKey ; Restore-AzKeyVaultManagedStorageAccount ; Restore-AzKeyVaultSecret 

- [Restauration de certificats Key Vault](/powershell/module/azurerm.keyvault/restore-azurekeyvaultcertificate)

- [Restauration de clés Key Vault](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey)

- [Restauration des comptes de stockage gérés par Key Vault](/powershell/module/az.keyvault/backup-azkeyvaultmanagedstorageaccount)

- [Restauration de secrets Key Vault](/powershell/module/azurerm.keyvault/restore-azurekeyvaultsecret)

- [AzCopy est un utilitaire de ligne de commande que vous pouvez utiliser pour copier des blobs, des fichiers ou des données de table vers ou depuis un compte de stockage](./storage-use-azcopy-v10.md)

Remarque : Si vous voulez copier des données depuis et vers le service de stockage Table Azure, installez AzCopy version 7.3.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Pour activer les clés gérées par le client sur un compte de stockage, vous devez utiliser un coffre de clés Azure pour stocker vos clés. Vous devez activer les propriétés Suppression réversible et Ne pas vider sur le coffre de clés. La fonctionnalité de suppression réversible de Key Vault permet de récupérer des coffres et des objets de coffre supprimés, tels que des clés, des secrets et autres certificats. Si vous sauvegardez des données de compte de stockage dans des blobs Stockage Azure, activez la suppression réversible pour enregistrer et récupérer vos données en cas de suppression de blobs ou d’instantanés de blobs. Vous devez traiter vos sauvegardes comme des données sensibles et appliquer les contrôles d’accès et de protection des données appropriés dans le cadre de cette base de référence. En outre, pour améliorer la protection, vous pouvez stocker les objets de données vitaux pour l’entreprise dans un état WORM (écriture unique et lecture multiple).

- [Utilisation de la suppression réversible d’Azure Key Vault](../../key-vault/general/key-vault-recovery.md)

- [Suppression réversible pour les objets blob de Stockage Azure](../blobs/soft-delete-blob-overview.md?tabs=azure-portal)

- [Stocker des données blob critiques pour l’entreprise avec un stockage immuable](../blobs/storage-blob-immutable-storage.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

- [Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de scoring et de hiérarchisation des incidents

**Aide** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte. 

En outre, marquez clairement les abonnements (par ex. production, non-production) à l’aide d’étiquettes et créez un système de nommage pour identifier et classer clairement les ressources Azure, en particulier celles qui traitent des données sensibles. Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

- [Alertes de sécurité dans le Centre de sécurité Azure](../../security-center/security-center-alerts-overview.md)

- [Organisation des ressources Azure à l’aide de catégories](../../azure-resource-manager/management/tag-resources.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Effectuez des exercices pour tester les capacités de réponse aux incidents de vos systèmes à intervalles réguliers, afin de protéger vos ressources Azure. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

- [Publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir les informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Conseils** : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

- [Comment définir le contact de sécurité d’Azure Security Center](../../security-center/security-center-provide-security-contact-details.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Aide** : Exportez vos alertes et recommandations Azure Security Center en utilisant la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel.

- [Comment configurer l’exportation continue](../../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../../sentinel/connect-azure-security-center.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Aide** : Utilisez la fonctionnalité d’automatisation de workflow d’Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » aux alertes et aux recommandations de sécurité afin de protéger vos ressources Azure.

- [Comment configurer l’automatisation des workflows et Logic Apps](../../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : tests d’intrusion et exercices Red Team](../../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources"></a>11.1 : Procéder à des tests d’intrusion réguliers de vos ressources Azure

**Aide** : Suivez les règles d’engagement de Microsoft pour que vos tests d’intrusion soient conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft.

- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consultez le [benchmark de sécurité Azure](../../security/benchmarks/overview.md)
- Apprenez-en davantage sur les [Bases de référence de la sécurité Azure](../../security/benchmarks/security-baselines-overview.md)