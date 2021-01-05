---
title: Base de référence de sécurité Azure pour Batch
description: La base de référence de sécurité Batch fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: batch
ms.topic: conceptual
ms.date: 12/01/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: e7be42b2a6e9f2cdc1aa0258f218fea9fd963093
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532027"
---
# <a name="azure-security-baseline-for-batch"></a>Base de référence de sécurité Azure pour Batch

Cette base de référence de sécurité applique les conseils du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview-v1.md) à Batch. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure.
Le contenu est regroupé par les **contrôles de sécurité** définis par le benchmark de sécurité Azure et les conseils associés applicables à Batch. Les **contrôles** non applicables à Batch ont été exclus.

 
Pour voir en quoi Batch est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité Batch](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Conseils** : Déployez le ou les pools Azure Batch dans un réseau virtuel. Pour permettre aux nœuds de calcul d’un pool de communiquer en toute sécurité avec d’autres machines virtuelles, ou avec un réseau local, vous pouvez provisionner le pool dans un sous-réseau d’un réseau virtuel Azure. En outre, en déployant le pool au sein d’un réseau virtuel, vous pouvez ensuite contrôler le groupe de sécurité réseau (NSG) utilisé pour sécuriser les interfaces réseau (NIC) des différents nœuds ainsi que le sous-réseau. Configurez le groupe de sécurité réseau pour autoriser le trafic uniquement à partir d’adresses IP/emplacements approuvés sur Internet.

Le cas échéant, désactivez l’accès au réseau public à l’aide d’Azure Private Link pour vous connecter au compte Azure Batch via un point de terminaison privé. Le service Azure Private Link est sécurisé et accepte uniquement les connexions à partir de points de terminaison privés authentifiés et autorisés. De plus, vous pouvez limiter la connectivité et la détectabilité en désactivant les points de terminaison RDP/SSH exposés publiquement pour les nœuds de calcul d’un pool Batch.

- [Guide pratique pour créer un pool Azure Batch dans un réseau virtuel](batch-virtual-network.md)

- [Guide pratique pour créer un compte Azure Batch avec l’accès réseau désactivé](private-connectivity.md)

- [Guide pratique pour créer un point de terminaison privé](../private-link/create-private-endpoint-portal.md)

- [Guide pratique pour refuser l’accès au trafic RDP/SSH](pool-endpoint-configuration.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des interfaces réseau

**Conseils** : Utilisez Azure Security Center et suivez les recommandations de protection du réseau applicables au réseau virtuel/groupe de sécurité réseau associé à votre pool Batch. Activez les journaux de flux sur le groupe de sécurité réseau utilisé pour protéger votre pool Batch et envoyez les journaux vers un compte de stockage Azure en vue de l’audit du trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Azure Log Analytics et utiliser Azure Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Azure Traffic Analytics offre certains avantages, parmi lesquels la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, de détecter les menaces de sécurité, de bien comprendre les modèles de flux de trafic et enfin de repérer les configurations réseau incorrectes.

- [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Guide pratique pour activer et utiliser Traffic Analytics](../network-watcher/traffic-analytics.md)

- [Présentation de la sécurité réseau assurée par Azure Security Center](../security-center/security-center-network-recommendations.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications avec des adresses IP connues comme étant malveillantes

**Conseils** : Activez le service Protection DDoS (déni de service distribué) Standard Azure sur le réseau virtuel protégeant votre pool Azure Batch afin d’assurer la protection contre les attaques DDoS. Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

- [Guide pratique pour configurer la protection DDoS](/azure/virtual-network/manage-ddos-protection)

- [Présentation de la fonctionnalité Threat Intelligence intégrée à Azure Security Center](/azure/security-center/security-center-alerts-service-layer)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Conseils** : Activez les journaux de flux sur le groupe de sécurité réseau (NSG) utilisé pour protéger votre pool Azure Batch et envoyez les journaux vers un compte de stockage Azure en vue de l’audit du trafic.

- [Guide pratique pour activer les journaux de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions (IDS/IPS) basés sur le réseau

**Conseils** : Si cela est nécessaire à des fins de conformité, sélectionnez une appliance virtuelle réseau dans la Place de marché Azure qui prend en charge les systèmes de détection des intrusions (IDS) et les systèmes de prévention des intrusions (IPS) avec des fonctionnalités d’inspection de la charge utile.

Si la détection et/ou la prévention des intrusions basées sur l’inspection de la charge utile ne font pas partie des exigences, vous pouvez utiliser le Pare-feu Azure avec la Threat Intelligence. Le filtrage basé sur la Threat Intelligence du Pare-feu Azure peut générer des alertes et refuser le trafic depuis ou vers des adresses IP et des domaines malveillants connus. Ces adresses IP et domaines proviennent du flux Microsoft Threat Intelligence.

Déployez le Pare-feu Azure avec une adresse IP publique dans le même réseau virtuel que vos nœuds de pool Azure Batch. Configurez des règles de traduction d’adresses réseau (NAT) entre des emplacements approuvés sur Internet et les adresses IP privées de vos nœuds de pool individuels. Dans le Pare-feu Azure, sous Threat Intelligence, configurez « Alerter et refuser » pour recevoir des alertes et bloquer le trafic en provenance ou à destination d’adresses IP et de domaines malveillants connus. Ces adresses IP et domaines proviennent du flux Microsoft Threat Intelligence ; seuls les enregistrements au niveau de confiance le plus élevé sont inclus. 

- [Guide pratique pour créer un pool Azure Batch dans un réseau virtuel](batch-virtual-network.md)

- [Guide pratique pour déployer le Pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md)

- [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : Utilisez des étiquettes de service de réseau virtuel pour définir des contrôles d’accès réseau sur les groupes de sécurité réseau ou les Pare-feu Azure associés à vos pools Azure Batch. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de la balise de service (par exemple, ApiManagement) dans le champ Source ou Destination approprié d'une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

- [Comprendre et utiliser les balises de service](../virtual-network/service-tags-overview.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour les ressources réseau associées à vos pools Azure Batch à l’aide d’Azure Policy. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.Batch » et « Microsoft.Network » pour créer des stratégies personnalisées d’audit ou d’application de la configuration réseau de vos pools Azure Batch.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : Utilisez des étiquettes pour les groupes de sécurité réseau et autres ressources liées à la sécurité réseau et au flux de trafic qui sont associés à vos pools Azure Batch. Concernant les règles NSG individuelles, utilisez le champ « Description » afin de spécifier le besoin métier et/ou la durée (etc.) pour toutes les règles qui autorisent le trafic vers/depuis un réseau.

Utilisez l’une des définitions Azure Policy intégrées en lien avec l’étiquetage, comme « Exiger une étiquette et sa valeur », pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

- [Guide pratique pour créer et utiliser des étiquettes](/azure/azure-resource-manager/resource-group-using-tags)

- [Guide pratique pour créer un réseau virtuel](../virtual-network/quick-create-portal.md)

- [Guide pratique pour créer un groupe de sécurité réseau](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : Utilisez le journal des activités Azure pour superviser les configurations des ressources réseau et détecter les changements des ressources réseau associées à vos pools Azure Batch. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](/azure/azure-monitor/platform/activity-log-view) 

- [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : Intégrez le compte Azure Batch à Azure Monitor pour agréger les données de sécurité générées par les appareils de cluster. Utilisez des requêtes personnalisées pour détecter les menaces dans l’environnement et y répondre.  Dans le cadre d’une supervision au niveau des ressources Azure Batch, utilisez les API Batch pour surveiller ou interroger l’état de toutes vos ressources, y compris les travaux, les tâches, les nœuds et les pools.

- [Guide pratique pour intégrer un compte Azure Batch à Azure Monitor](batch-diagnostics.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Dans le cadre d’une supervision au niveau du compte Azure Batch, supervisez chaque compte Batch à l’aide des fonctionnalités fournies dans Azure Monitor. Azure Monitor collecte des métriques et éventuellement des journaux de diagnostic pour les ressources au niveau du compte Batch, comme les pools, les travaux et les tâches. Collectez et utilisez ces données manuellement ou par programmation pour surveiller les activités dans votre compte Batch et diagnostiquer les problèmes.

Dans le cadre d’une supervision au niveau des ressources Azure Batch, utilisez les API Azure Batch pour surveiller ou interroger l’état de toutes vos ressources, y compris les travaux, les tâches, les nœuds et les pools.

- [Guide pratique pour configurer la supervision et la journalisation au niveau du compte Azure Batch](monitoring-overview.md)

- [Comprendre la supervision au niveau des ressources Batch](monitoring-overview.md#batch-resource-monitoring)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

#### <a name="azure-policy-built-in-definitions"></a>Définitions intégrées à Azure Policy

[!INCLUDE [microsoft.batch-2-3](../../includes/policy/standards/asb/rp-controls/microsoft.batch-2-3.md)]

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : Azure Monitor collecte des métriques et des journaux de diagnostic pour les ressources de votre compte Azure Batch. Collectez et utilisez ces données de plusieurs façons pour superviser votre compte Azure Batch et diagnostiquer les problèmes. Vous pouvez également configurer des alertes de métrique afin de recevoir des notifications lorsqu’une métrique atteint une valeur spécifiée.

Si nécessaire, vous devrez vous connecter à vos nœuds de pool individuels par le biais du protocole SSH (Secure Shell) ou du protocole RDP (Remote Desktop Protocol) pour accéder aux journaux du système d’exploitation local.

- [Guide pratique pour collecter les journaux de diagnostic à partir de votre compte Azure Batch](batch-diagnostics.md#batch-diagnostics)

- [Guide pratique pour se connecter à distance aux nœuds de votre pool Azure Batch](/azure/batch/batch-api-basics#error-handling)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Intégrez le compte Azure Batch à Azure Monitor. Vérifiez que la période de conservation des journaux définie dans l’espace de travail Azure Log Analytics utilisé est conforme aux obligations réglementaires de votre organisation.

- [Guide pratique pour configurer la supervision et la journalisation dans Azure Batch](monitoring-overview.md)

- [Guide pratique pour configurer la période de rétention dans un espace de travail Azure Log Analytics](../azure-monitor/platform/manage-cost-storage.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : Créez des alertes de métrique Azure Batch qui se déclenchent quand la valeur d’une métrique spécifiée dépasse un seuil donné.

- [Guide pratique pour configurer des alertes de métriques Azure Batch](batch-diagnostics.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils** : Créez des alertes de métrique Azure Batch qui se déclenchent quand la valeur d’une métrique spécifiée dépasse un seuil donné.

- [Guide pratique pour configurer des alertes de métriques Azure Batch](batch-diagnostics.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Conseils** : Exécutez Windows Defender sur chacun de vos nœuds Batch si vous utilisez un système d’exploitation Windows, ou fournissez votre propre solution anti-programme malveillant si vous travaillez sur Linux.

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Conseils** : Implémentez une solution tierce pour la journalisation DNS.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Conseils** : Configurez manuellement la journalisation de la console et la transcription PowerShell au niveau de chaque nœud.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : Conservez l’enregistrement du compte d’administrateur local créé lors du provisionnement du pool Azure Batch ainsi que des autres comptes que vous créez. De plus, si l’intégration Azure Active Directory est utilisée, Azure AD comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent donc être interrogés. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes qui sont membres de groupes d’administration.

Vous pouvez également suivre les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0&amp;preserve-view=true)

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0&amp;preserve-view=true)

- [Comment superviser les identités et les accès avec Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : Quand vous provisionnez un pool Azure Batch, vous avez l’option de créer des comptes de machine locale. Il n’y a pas de mot de passe par défaut à changer. Toutefois, vous pouvez spécifier des mots de passe différents pour l’accès SSH (Secure Shell) et l’accès RDP (Remote Desktop Protocol). Une fois que vous avez configuré le pool Azure Batch, vous pouvez générer un utilisateur aléatoire pour chaque nœud à partir du portail Azure ou de l’API Azure Resource Manager.

- [Guide pratique pour ajouter un utilisateur à un nœud de calcul spécifique](/rest/api/batchservice/computenode/adduser)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Intégrez l’authentification des applications Azure Batch avec Azure Active Directory. Créez des stratégies et des procédures qui contrôlent l’utilisation de comptes d’administration dédiés.

Vous pouvez également suivre les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

- [Guide pratique pour authentifier des applications Batch avec Azure Active Directory](batch-aad-auth.md)

- [Comment superviser les identités et les accès avec Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : Intégrez l’authentification des applications Azure Batch avec Azure Active Directory. Activez l’authentification multifacteur Azure AD et suivez les recommandations de gestion des identités et des accès Azure Security Center.

 

- [Guide pratique pour activer l’authentification multifacteur dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3.6 : Utiliser des stations de travail sécurisées et gérées par Azure pour les tâches administratives

**Conseils** : Utilisez des stations de travail à accès privilégié avec l’authentification multifacteur configurée pour vous connecter à vos ressources Azure Batch et les configurer.

- [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Guide pratique pour activer l’authentification multifacteur dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils** : Si vous avez intégré l’authentification pour des applications Azure Batch avec Azure Active Directory, utilisez les rapports de sécurité Azure Active Directory pour générer des journaux et des alertes lorsque des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement. Utiliser Azure Security Center pour superviser les activités liées aux identités et aux accès.

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](/azure/active-directory/reports-monitoring/concept-user-at-risk)

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Si vous avez intégré l’authentification des applications Azure Batch avec Azure Active Directory, vous pouvez utiliser les emplacements nommés de l’accès conditionnel pour autoriser l’accès uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

- [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Conseils** : Utilisez Azure Active Directory comme système central d’authentification et d’autorisation et intégrez l’authentification pour les applications Azure Batch avec Azure AD. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

- [Création et configuration d’une instance Azure AD](../active-directory-domain-services/tutorial-create-instance.md)

- [Guide pratique pour authentifier des applications Batch avec Azure AD](batch-aad-auth.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory fournit des journaux pour vous aider à découvrir les comptes obsolètes. De plus, vous pouvez utiliser les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. Les accès des utilisateurs peuvent être revus régulièrement pour vérifier que seuls les utilisateurs appropriés continuent de bénéficier d’un accès.

- [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Conseils** : Créez des paramètres de diagnostic pour les comptes d’utilisateur Azure Active Directory, en envoyant les journaux d’audit et les journaux de connexion vers un espace de travail Azure Log Analytics. Configurez les alertes souhaitées dans un espace de travail Azure Log Analytics.

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12 : Alerter en cas d’écart de comportement de connexion à un compte

**Conseils** : Utilisez la détection des risques et la fonctionnalité Identity Risk d’Azure Active Directory pour configurer des réponses automatisées aux actions suspectes détectées en rapport avec les identités d’utilisateurs. De plus, vous pouvez ingérer des données dans Azure Sentinel pour approfondir votre examen.

- [Guide pratique pour afficher les connexions risquées Azure AD](/azure/active-directory/reports-monitoring/concept-risky-sign-ins)

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support  

**Conseils** : Non disponible. Customer Lockbox n’est pas actuellement pris en charge pour Azure Batch.
 
- [Liste des services pris en charge pour Customer Lockbox](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

- [Guide pratique pour créer et utiliser des étiquettes](/azure/azure-resource-manager/resource-group-using-tags)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Les pools Azure Batch doivent être séparés par un réseau/sous-réseau virtuel étiqueté de manière appropriée, mais aussi sécurisés à l’aide de groupes de sécurité réseau (NSG). Les données Azure Batch doivent être stockées dans un compte de stockage Azure sécurisé.

- [Guide pratique pour créer un pool Azure Batch dans un réseau virtuel](batch-virtual-network.md)

- [Comment sécuriser des comptes de Stockage Azure](/azure/storage/common/storage-security-guide)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : Si vous avez des comptes de stockage Azure associés à votre ou vos pools Azure Batch qui contiennent des informations sensibles, marquez-les comme sensibles à l’aide d’étiquettes et sécurisez-les en suivant les bonnes pratiques Azure en la matière.

Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour le Stockage Azure ou les ressources de calcul. Implémentez une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

- [Comment sécuriser des comptes de Stockage Azure](/azure/storage/common/storage-security-guide)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Instructions** : Chiffrer toutes les informations sensibles en transit. Par défaut, les ressources Microsoft Azure négocient TLS 1.2. Assurez-vous que les clients qui se connectent à vos pools Azure Batch ou magasins de données (comptes Stockage Azure) sont en mesure de négocier TLS 1.2 ou une version ultérieure.

Assurez-vous aussi que le protocole HTTPS est obligatoire pour accéder au compte de stockage contenant vos données Azure Batch.

- [Comprendre le chiffrement en transit pour un compte Stockage Azure](../storage/blobs/security-recommendations.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Conseils** : Si vous avez des comptes de stockage Azure associés à votre ou vos pools Azure Batch qui contiennent des informations sensibles, marquez-les comme sensibles à l’aide d’étiquettes et sécurisez-les en suivant les bonnes pratiques Azure en la matière.

Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour le Stockage Azure ou les ressources de calcul. Implémentez une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

- [Comment sécuriser des comptes de Stockage Azure](/azure/storage/common/storage-security-guide)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4.6 : Utiliser le contrôle d’accès en fonction du rôle pour contrôler l’accès aux ressources

**Aide** : Utilisez le contrôle d’accès en fonction du rôle Azure (RBAC Azure) pour contrôler l’accès au plan de gestion des ressources Azure telles que le compte Batch, les pools Batch et les comptes de stockage.

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

- [Comment configurer Azure RBAC](../role-based-access-control/role-assignments-portal.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Aide** : Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour le Stockage Azure ou les ressources de calcul. Implémentez une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : Pour les comptes de stockage associés à votre compte Azure Batch, nous vous recommandons d’autoriser Microsoft à gérer les clés de chiffrement. Toutefois, vous avez la possibilité de gérer vos propres clés si nécessaire.

Vous pouvez utiliser le chiffrement de disque Azure pour protéger et préserver vos données et ainsi répondre aux engagements de l’entreprise en matière de sécurité et de conformité. L’ensemble des disques managés, des captures instantanées, des images et des données écrites sur des disques existants sont automatiquement chiffrés au repos avec des clés gérées par la plateforme.

- [Guide pratique pour gérer les clés de chiffrement pour les comptes Stockage Azure](/azure/storage/common/storage-encryption-keys-portal)

- [Guide pratique pour configurer des clés de chiffrement gérées par le client](/azure/storage/common/storage-encryption-keys-portal)

- [Guide pratique pour créer un pool avec le chiffrement de disque activé](disk-encryption.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : Utilisez Azure Monitor avec le journal des activités Azure pour créer des alertes quand des modifications sont apportées à des ressources Azure critiques liées ou associées à vos comptes/pools Azure Batch.

Configurez les paramètres de diagnostic des comptes de stockage associés à des pools Azure Batch pour superviser et journaliser toutes les opérations CRUD sur les données de pool.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

- [Guide pratique pour activer une journalisation/un audit supplémentaires pour un compte Stockage Azure](../storage/common/storage-monitor-storage-account.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : Pour les nœuds de pool Azure Batch, la gestion de la solution de gestion des vulnérabilités vous incombe.

Éventuellement, si vous avez un abonnement à Rapid7, à Qualys ou à toute autre plateforme de gestion des vulnérabilités, vous pouvez installer manuellement des agents d’évaluation des vulnérabilités sur les nœuds de pool Batch et gérer ces nœuds par le biais du portail en question.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : Les images des nœuds de pool Azure Batch de base sont gérées et tenues à jour par Microsoft. Assurez-vous de garder le système d’exploitation des nœuds de pool Azure Batch à jour avec les correctifs pendant toute la durée de vie du cluster, ce qui peut nécessiter l’activation des mises à jour automatiques, la supervision des nœuds ou des redémarrages périodiques.

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="53-deploy-automated-patch-management-solution-for-third-party-software-titles"></a>5.3 : Déployer une solution de gestion automatisée des correctifs des logiciels tiers

**Conseils** : Assurez-vous de garder les applications tierces installées sur les nœuds de pool Azure Batch à jour avec les correctifs pendant toute la durée de vie du cluster, ce qui peut nécessiter l’activation des mises à jour automatiques, la supervision des nœuds ou des redémarrages périodiques.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils** : Si vous avez un abonnement à Rapid7, à Qualys ou à toute autre plateforme de gestion des vulnérabilités, vous pouvez utiliser le portail de ce fournisseur pour consulter et comparer les analyses de vulnérabilités dos à dos.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Instructions** : Utilisez un programme de scoring des risques courants (par exemple, Common Vulnerability Scoring System) ou bien les évaluations des risques par défaut fournies par votre outil d’analyse tiers.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (calcul, stockage, réseau, etc.) dans vos abonnements. Vérifiez que vous avez les autorisations (lecture) appropriées dans votre locataire et que vous êtes en mesure d’établir une liste de tous les abonnements Azure et de toutes les ressources dans vos abonnements.

Même si les ressources Azure classiques peuvent être découvertes via l’Explorateur Azure Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à partir de maintenant.

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-4.8.0&amp;preserve-view=true)

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

- [Guide pratique pour créer et utiliser des étiquettes](/azure/azure-resource-manager/resource-group-using-tags)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des étiquettes, des groupes d’administration et diviser des abonnements, le cas échéant, pour organiser et suivre les ressources. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

- [Guide pratique pour créer des abonnements Azure supplémentaires](/azure/billing/billing-create-subscription)

- [Guide pratique pour créer des groupes d’administration](/azure/governance/management-groups/create)

- [Guide pratique pour créer et utiliser des étiquettes](/azure/azure-resource-manager/resource-group-using-tags)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Conseils** : Établissez la liste des ressources Azure approuvées et des logiciels approuvés pour les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :

- Types de ressources non autorisés
- Types de ressources autorisés

Utilisez Azure Resource Graph pour interroger/découvrir les ressources dans vos abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : Pour les nœuds de pool Azure Batch, implémentez une solution tierce qui détecte la présence d’applications logicielles non approuvées sur les nœuds de cluster.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : Pour les nœuds de pool Azure Batch, implémentez une solution tierce qui détecte la présence d’applications logicielles non approuvées sur les nœuds de cluster.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : Pour les nœuds de pool Azure Batch, implémentez une solution tierce qui empêche l’exécution de logiciels non autorisés.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes : 
- Types de ressources non autorisés 
- Types de ressources autorisés 

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/built-in-policies.md#general)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés

**Conseils** : Pour les nœuds de pool Azure Batch, implémentez une solution tierce qui empêche l’exécution de types de fichiers non autorisés.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

- [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Conseils** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.Batch » pour créer des stratégies personnalisées d’audit ou d’application de la configuration de vos comptes et pools Azure Batch.

- [Affichage des alias Azure Policy disponibles](https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-4.8.0&amp;preserve-view=true)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Établissez des configurations sécurisées pour le système d’exploitation de vos nœuds Batch Pool.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés aux ressources Azure associées à votre compte et vos pools Batch (tels que les réseaux virtuels, les sous-réseaux, les Pare-feu Azure, les comptes de stockage Azure, etc.). Vous pouvez utiliser les alias Azure Policy des espaces de noms suivants pour créer des stratégies personnalisées :

- Microsoft.Batch

- Microsoft.Storage

- Microsoft.Network

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Présentation des effets d’Azure Policy](../governance/policy/concepts/effects.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Les images des systèmes d’exploitation des pools Azure Batch sont gérées et tenues à jour par Microsoft. L’implémentation de la configuration d’état au niveau du système d’exploitation vous incombe.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous utilisez des définitions Azure Policy personnalisées pour vos comptes ou pools Azure Batch (ou ressources associées), utilisez Azure Repos pour stocker et gérer votre code en toute sécurité.

- [Stocker du code dans Azure DevOps](https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops&amp;preserve-view=true)

- [Documentation Azure Repos](https://docs.microsoft.com/azure/devops/repos/?view=azure-devops&amp;preserve-view=true)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Si vous utilisez des images personnalisées pour vos pools Azure Batch, utilisez le contrôle d’accès en fonction du rôle (RBAC) pour garantir que seuls les utilisateurs autorisés pourront accéder aux images.

- [Contrôle d'accès en fonction du rôle dans Azure](../role-based-access-control/rbac-and-directory-admin-roles.md)

- [Guide pratique pour configurer le contrôle RBAC dans Azure](../role-based-access-control/quickstart-assign-role-user-portal.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : Utilisez des définitions Azure Policy intégrées pour générer des alertes ainsi qu’auditer et appliquer les configurations des ressources liées à Azure Batch.  Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.Batch » pour créer des stratégies personnalisées pour vos comptes et pools Azure Batch. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer des outils de gestion de la configuration pour les systèmes d'exploitation

**Conseils** : Implémentez une solution tierce qui maintient l’état souhaité pour les systèmes d’exploitation de vos nœuds de pool Azure Batch.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Conseils** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.Batch » pour créer des stratégies personnalisées d’audit ou d’application de la configuration de votre instance Azure Batch. Vous pouvez également employer des stratégies intégrées créées spécifiquement pour Azure Batch ou les ressources utilisées par Azure Batch, par exemple :
- Les sous-réseaux doivent être associés à un groupe de sécurité réseau - Les comptes de stockage doivent utiliser un point de terminaison de service de réseau virtuel
- Les journaux de diagnostic doivent être activés dans les comptes Batch

- [Affichage des alias Azure Policy disponibles](https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-4.8.0&amp;preserve-view=true)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : Implémentez une solution tierce pour superviser l’état des systèmes d’exploitation de vos nœuds de pool Azure Batch.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : Azure Key Vault peut être utilisé avec des déploiements Azure Batch pour gérer les clés du stockage de pools dans des comptes de stockage Azure.

- [Intégration aux identités managées Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md)

- [Guide pratique pour créer un coffre Azure Key Vault](../key-vault/general/quick-create-portal.md)

- [Comment s’authentifier auprès de Key Vault](../key-vault/general/authentication.md)
- [Comment attribuer une stratégie d’accès Key Vault](../key-vault/general/assign-access-policy-portal.md)

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

**Conseils** : Exécutez Windows Defender sur vos nœuds de pool Azure Batch individuels si vous utilisez un système d’exploitation Windows, ou fournissez votre propre solution anti-programme malveillant si vous travaillez sur Linux.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Conseils** : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure Batch), mais il ne s’exécute pas sur du contenu client.

Pré-analysez les fichiers chargés sur des ressources Azure non liées au calcul, comme App Service, des Data Lake Storage, Stockage Blob, etc. Microsoft ne peut pas accéder aux données client dans ces instances.

- [Présentation de Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles](../security/fundamentals/antimalware.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Conseils** : Exécutez Windows Defender sur vos nœuds de pool Azure Batch individuels si vous utilisez un système d’exploitation Windows et vérifiez que la mise à jour automatique est activée. Fournissez votre propre solution anti-programme malveillant si vous êtes sur Linux.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="data-recovery"></a>Récupération des données

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : Garantir des sauvegardes automatiques régulières

**Conseils** : Si vous utilisez un compte de stockage Azure comme magasin de données de pool Azure Batch, choisissez l’option de redondance appropriée (stockage localement redondant, stockage redondant interzone, stockage géoredondant, stockage géographiquement redondant avec accès en lecture). 

- [Guide pratique pour configurer la redondance du stockage pour les comptes Stockage Azure](../storage/common/storage-redundancy.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés managées par le client

**Conseils** : Si vous utilisez un compte de stockage Azure comme magasin de données de pool Azure Batch, choisissez l’option de redondance appropriée (stockage localement redondant, stockage redondant interzone, stockage géoredondant, stockage géographiquement redondant avec accès en lecture). Si vous utilisez Azure Key Vault pour n’importe quelle partie de votre déploiement Azure Batch, vérifiez que vos clés sont sauvegardées.

- [Guide pratique pour configurer la redondance du stockage pour les comptes Stockage Azure](../storage/common/storage-redundancy.md)

- [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](https://docs.microsoft.com/powershell/module/az.keyvault/backup-azkeyvaultkey?view=azps-4.8.0&amp;preserve-view=true)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : Si vous gérez vos propres clés pour des comptes de stockage Azure ou toute autre ressource associée à votre implémentation d’Azure Batch, effectuez régulièrement des tests de restauration des clés sauvegardées.

- [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](https://docs.microsoft.com/powershell/module/az.keyvault/backup-azkeyvaultkey?view=azps-4.8.0&amp;preserve-view=true)

- [Guide pratique pour restaurer une clé gérée par le client avec PowerShell](https://docs.microsoft.com/powershell/module/az.keyvault/restore-azkeyvaultkey?view=azps-4.8.0&amp;preserve-view=true)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés managées par le client

**Conseils** : Si Azure Key Vault est utilisé pour conserver les clés associées aux comptes de stockage de pool Azure Batch, activez la suppression réversible dans Azure Key Vault pour protéger les clés contre toute suppression accidentelle ou malveillante.

- [Guide pratique pour activer la suppression réversible dans Azure Key Vault](/azure/key-vault/key-vault-soft-delete-powershell)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Vérifiez qu’il existe des plans de réponse aux incidents écrits qui définissent les rôles du personnel ainsi que les phases de gestion des incidents.

- [Comment configurer des automatisations de workflow dans Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center affecte un degré de gravité aux alertes pour vous aider à hiérarchiser l’ordre dans lequel vous remédiez à chaque alerte. Ainsi, quand une ressource est compromise, vous pouvez vous en occuper immédiatement. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

- [Reportez-vous à la publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Conseils** : Les informations de contact pour les incidents de sécurité seront utilisées par Microsoft afin de vous contacter si Microsoft Security Response Center (MSRC) découvre que vos données ont été utilisées de manière illégale ou par un tiers non autorisé.

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel.

- [Comment configurer l’exportation continue](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

- [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Conseil** : – [Respectez les règles d’engagement de Microsoft pour garantir la conformité de vos tests d’intrusion aux stratégies Microsoft](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1.)

Vous trouverez ici un complément d’informations sur la stratégie de Microsoft, l’exécution de Red Teaming et les tests d’intrusion de site actif sur l’infrastructure, les services et les applications cloud managés par Microsoft suivants : 

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Vue d’ensemble d’Azure Security Benchmark V2](/azure/security/benchmarks/overview)
- En savoir plus sur les [bases de référence de la sécurité Azure](/azure/security/benchmarks/security-baselines-overview)
