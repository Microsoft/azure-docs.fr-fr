---
title: Ligne de base de sécurité Azure pour Azure Container Registry
description: Ligne de base de sécurité Azure pour Azure Container Registry
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 03/16/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 3fcbc386c60611493912fdfb17226490549cdc53
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89396809"
---
# <a name="azure-security-baseline-for-azure-container-registry"></a>Ligne de base de sécurité Azure pour Azure Container Registry

La ligne de base de sécurité Azure pour Azure Container Registry contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1 : Protéger les ressources à l'aide de groupes de sécurité réseau ou du Pare-feu Azure sur votre réseau virtuel

**Aide** : Un réseau virtuel Azure fournit un accès réseau privé et sécurisé à vos ressources Azure et locales. En limitant l’accès à votre registre de conteneurs Azure privé à partir d’un réseau virtuel Azure, vous vous assurez que seules les ressources dans le réseau virtuel accèdent au registre. Pour les scénarios intersites, vous pouvez également configurer des règles de pare-feu pour autoriser l’accès au registre uniquement à partir d’adresses IP spécifiques. Derrière un pare-feu, configurez les règles d’accès du pare-feu et les étiquettes de service pour accéder à votre registre de conteneurs.

Restreindre l’accès à un registre de conteneurs Azure à l’aide d’un réseau virtuel Azure ou de règles de pare-feu : https://docs.microsoft.com/azure/container-registry/container-registry-vnet 

Configurer des règles pour accéder à un registre de conteneurs Azure derrière un pare-feu : https://docs.microsoft.com/azure/container-registry/container-registry-firewall-access-rules


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2 : Surveiller et consigner la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Aide** : Utilisez Azure Security Center et suivez les recommandations de protection du réseau pour protéger vos ressources réseau dans Azure. Activez les journaux de flux NSG et transférez-les vers un compte de stockage pour l'audit du trafic.

Comment activer les journaux de flux NSG : https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Protéger vos ressources réseau : https://docs.microsoft.com/azure/security-center/security-center-network-recommendations



**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Aide** : Non applicable. Le test d’évaluation est destiné à Azure App Service ou aux ressources de calcul hébergeant des applications web.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Aide** : Activez la protection DDoS Standard sur vos réseaux virtuels Azure pour vous protéger des attaques DDoS. Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.  Déployez le Pare-feu Azure à chaque extrémité du réseau de l'organisation en activant la fonctionnalité de renseignement sur les menaces et en la configurant sur « Alerter et refuser » pour vous protéger de tout trafic réseau malveillant.

Vous pouvez utiliser la fonctionnalité d'accès réseau juste-à-temps d'Azure Security Center pour configurer les groupes de sécurité réseau (NSG) et limiter l'exposition des points de terminaison aux adresses IP approuvées pendant une période limitée. De plus, utilisez la fonctionnalité de renforcement du réseau adaptatif d'Azure Security Center pour recommander des configurations NSG qui limitent les ports et les adresses IP sources en fonction du trafic réel et du renseignement sur les menaces.

Comment configurer la protection DDoS : https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection

Déployer le Pare-feu Azure : https://docs.microsoft.com/azure/firewall/tutorial-firewall-deploy-portal

Comprendre la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-alerts-service-layer

Comprendre la fonctionnalité de renforcement du réseau adaptatif d’Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-adaptive-network-hardening

Fonctionnalité de contrôle d’accès réseau juste-à-temps d’Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-just-in-time


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5 : Consigner les paquets réseau et les journaux de flux

**Aide** : Activez les journaux de flux de groupe de sécurité réseau (NSG) pour le groupe de sécurité réseau attaché au sous-réseau utilisé pour protéger votre registre de conteneurs Azure. Vous pouvez enregistrer les journaux de flux NSG dans un compte de Stockage Azure pour générer des enregistrements de flux. Si cela s’avère nécessaire pour analyser une activité anormale, activez la capture de paquets Azure Network Watcher.

Comment activer les journaux de flux NSG : https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Activer Network Watcher : https://docs.microsoft.com/azure/network-watcher/network-watcher-create


**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Aide** : Sélectionnez une offre de la place de marché Azure qui prend en charge les fonctionnalités IDS/IPS avec des fonctionnalités d’inspection de charge utile. Si la détection et/ou la prévention des intrusions basées sur l’inspection de la charge utile ne font pas partie des exigences, vous pouvez utiliser le Pare-feu Azure avec Threat Intelligence. Le filtrage basé sur le renseignement sur les menaces du Pare-feu Azure peut générer des alertes et refuser le trafic depuis ou vers des adresses IP et des domaines malveillants connus. Ces adresses IP et domaines proviennent du flux Microsoft Threat Intelligence.

Déployez la solution de pare-feu de votre choix dans les limites réseau de votre organisation pour détecter et/ou refuser le trafic malveillant.

Place de marché Azure :  https://azuremarketplace.microsoft.com/marketplace/?term=Firewall 

Déployer le Pare-feu Azure : https://docs.microsoft.com/azure/firewall/tutorial-firewall-deploy-portal

Configurer des alertes avec le Pare-feu Azure : https://docs.microsoft.com/azure/firewall/threat-intel


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Aide** : Non applicable. Le benchmark concerne les applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Aide** : Pour les ressources qui doivent accéder à votre registre de conteneurs, utilisez des étiquettes de service de réseau virtuel pour le service Azure Container Registry afin de définir des contrôles d’accès réseau sur des groupes de sécurité réseau ou le pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de l’étiquette de service « AzureContainerRegistry » dans le champ Source ou Destination approprié d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Autoriser l’accès par étiquette de service : https://docs.microsoft.com/azure/container-registry/container-registry-firewall-access-rules#allow-access-by-service-tag


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Aide** : Définissez et implémentez des configurations de sécurité standard pour les ressources réseau associées à vos registres de conteneurs Azure à l’aide d’Azure Policy. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.ContainerRegistry » et « Microsoft.Network » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de vos registres de conteneurs. 

Vous pouvez utiliser Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les artefacts d’environnement clés, tels que les modèles Resource Manager, les contrôles Azure RBAC et les stratégies, au sein d’une seule définition de blueprint. Appliquez aisément le blueprint aux nouveaux abonnements et affiner le contrôle et la gestion via le contrôle de version.

Auditer la conformité des registres de conteneurs Azure à l’aide d’Azure Policy :  https://docs.microsoft.com/azure/container-registry/container-registry-azure-policy

Créer une instance Azure Blueprint : https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Aide** : Le client peut utiliser Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les artefacts d’environnement clés, tels que les modèles Resource Manager, les contrôles Azure RBAC et les stratégies, au sein d’une seule définition de blueprint. Appliquez aisément le blueprint aux nouveaux abonnements et affiner le contrôle et la gestion via le contrôle de version.

Créer une instance Azure Blueprint : https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : Utilisez le journal des activités Azure pour superviser les configurations des ressources réseau et détecter les changements des ressources réseau associées à vos registres de conteneurs. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.

Comment consulter et récupérer les événements du journal d’activité Azure :  https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view

Comment créer des alertes dans Azure Monitor :  https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log



**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Contrôle de sécurité : journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Aide** : Microsoft gère les sources de temps pour les ressources Azure. Toutefois, vous avez la possibilité de gérer les paramètres de synchronisation date/heure pour vos ressources de calcul.

Comment configurer la synchronisation de l’heure pour les ressources de calcul Azure : https://docs.microsoft.com/azure/virtual-machines/windows/time-sync


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Ingérez les journaux via Azure Monitor pour agréger les données de sécurité générées par un registre de conteneurs Azure. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, puis utilisez les comptes de stockage Azure pour le stockage à long terme/d’archivage.

Journaux d’Azure Container Registry pour l’évaluation et l’audit de diagnostics :  https://docs.microsoft.com/azure/container-registry/container-registry-diagnostics-audit-logs



**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Aide** : Azure Monitor collecte les journaux de ressources (anciennement appelés journaux de diagnostic) pour les événements pilotés par l’utilisateur dans votre registre. Collectez et consommez ces données pour auditer les événements d’authentification de registre et fournir une trace d’activité complète sur les artefacts du registre, tels que les événements de tirage (pull) et d’envoi (push), pour vous permettre de diagnostiquer les problèmes de sécurité avec votre registre.

Journaux d’Azure Container Registry pour l’évaluation et l’audit de diagnostics : https://docs.microsoft.com/azure/container-registry/container-registry-diagnostics-audit-logs


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Aide** : Non applicable. Ce benchmark a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Aide** : Dans Azure Monitor, définissez la période de conservation de votre espace de travail Log Analytics en fonction des obligations réglementaires de votre organisation. Utilisez les comptes de stockage Azure pour le stockage à long terme/d’archivage.

Comment définir les paramètres de conservation des journaux pour les espaces de travail Log Analytics : https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Analysez et supervisez les journaux Azure Container Registry pour détecter les comportements anormaux et examinez régulièrement les résultats. Utilisez l’espace de travail Log Analytics d’Azure Monitor pour examiner les journaux et effectuer des requêtes sur leurs données.

Journaux d’Azure Container Registry pour l’évaluation et l’audit de diagnostics :  https://docs.microsoft.com/azure/container-registry/container-registry-diagnostics-audit-logs

Comprendre l’espace de travail Log Analytics : https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal

Comment exécuter des requêtes personnalisées dans Azure Monitor : https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer des alertes en cas d’activité anormale

**Aide** : Utilisez l’espace de travail Azure Log Analytics pour superviser les activités anormales dans les événements et les journaux de sécurité liés à votre registre de conteneurs Azure, et générer des alertes s’y rapportant.

Journaux d’Azure Container Registry pour l’évaluation et l’audit de diagnostics : https://docs.microsoft.com/azure/container-registry/container-registry-diagnostics-audit-logs

Comment générer une alerte sur des données de journal Log Analytics :  https://docs.microsoft.com/azure/azure-monitor/learn/tutorial-response


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide** : Non applicable. Azure Container Registry ne traite pas et ne produit pas de journaux liés aux logiciels anti-programmes malveillants.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Aide** : Non applicable. Azure Container Registry est un point de terminaison qui n’initie pas de communication, et le service n’interroge pas DNS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Aide** : Non applicable. Ce benchmark a trait aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Azure Active Directory (Azure AD) comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent être interrogés. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration.

Pour chaque registre de conteneurs Azure, assurez-vous que le compte d’administrateur intégré est activé ou désactivé. Désactivez le compte lorsqu’il n’est pas utilisé.

Comment obtenir un rôle d’annuaire dans Azure AD avec PowerShell : https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0

Comment obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell : https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0

Compte d’administrateur Azure Container Registry :  https://docs.microsoft.com/azure/container-registry/container-registry-authentication#admin-account


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Aide** : Azure Active Directory (Azure AD) n’intègre pas le concept des mots de passe par défaut. Selon le service, d’autres ressources Azure qui exigent un mot de passe forcent la création d’un mot de passe conforme à des exigences de complexité et d’une longueur minimale. Vous êtes responsable des applications tierces et des services de la place de marché susceptibles d’utiliser des mots de passe par défaut.

Si le compte d’administrateur par défaut d’un registre de conteneurs Azure est activé, les mots de passe complexes sont automatiquement créés et doivent faire l’objet d’une rotation. Désactivez le compte lorsqu’il n’est pas utilisé.

Compte d’administrateur Azure Container Registry : https://docs.microsoft.com/azure/container-registry/container-registry-authentication#admin-account



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

Créez également des procédures pour activer le compte administrateur intégré d’un registre de conteneurs. Désactivez le compte lorsqu’il n’est pas utilisé.

Comprendre l’identité et l’accès dans Azure Security Center :  https://docs.microsoft.com/azure/security-center/security-center-identity-access

Compte d’administrateur Azure Container Registry :  https://docs.microsoft.com/azure/container-registry/container-registry-authentication#admin-account



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : Dans la mesure du possible, utilisez l’authentification unique (SSO) d’Azure Active Directory au lieu de configurer des informations d’identification autonomes individuelles par service. Suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

Pour un accès individuel au registre de conteneurs, utilisez une connexion individuelle intégrée avec Azure Active Directory.

Comprendre l’authentification unique (SSO) avec Azure AD :  https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on

Connexion individuelle à un registre de conteneurs :  https://docs.microsoft.com/azure/container-registry/container-registry-authentication#individual-login-with-azure-ad


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Aide** : Activez l’authentification multifacteur (MFA) Azure Active Directory (Azure AD) et suivez les recommandations de gestion des identités et des accès dans Azure Security Center.

Comment activer l’authentification multifacteur dans Azure : https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Comment superviser les identités et les accès dans Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-identity-access


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Aide** : Utilisez des stations de travail avec accès privilégié avec l’authentification multifacteur (MFA) configurée pour la connexion aux ressources Azure et leur configuration.

En savoir plus sur les stations de travail avec accès privilégié :  https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations

Comment activer l’authentification multifacteur dans Azure : https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted


**Supervision d’Azure Security Center** : N/A

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration

**Aide** : Utilisez les rapports de sécurité Azure Active Directory (Azure AD) pour générer des journaux et des alertes quand une activité suspecte ou potentiellement dangereuse se produit dans l’environnement. Utiliser Azure Security Center pour superviser les activités liées aux identités et aux accès.

Comment identifier les utilisateurs Azure AD signalés pour une activité à risque : https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-user-at-risk

Comment superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-identity-access


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Aide** : Utilisez des emplacements nommés à accès conditionnel pour autoriser l’accès uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

Comment configurer des emplacements nommés dans Azure : https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Utiliser Azure Active Directory (Azure AD) comme système d’authentification et d’autorisation central. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

Comment créer et configurer une instance Azure AD : https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory (Azure AD) fournit des journaux pour vous aider à découvrir les comptes obsolètes. De plus, utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seules les utilisateurs appropriés continuent de bénéficier d’un accès.

Comprendre la génération de rapports Azure AD : https://docs.microsoft.com/azure/active-directory/reports-monitoring/

Comment utiliser les révisions d’accès des identités Azure : https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés

**Aide** : Vous pouvez accéder aux sources de journal de l’activité de connexion, de l’audit et des événements à risque d’Azure Active Directory (Azure AD), ce qui vous permet d’intégrer n’importe quel outil SIEM (Security Information and Event Management) ou de supervision.

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure Active Directory et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes souhaitées dans un espace de travail Log Analytics.

Comment intégrer des journaux d’activité Azure dans Azure Monitor : https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez les détections de risque et la fonctionnalité Risk and Identity Protection d’Azure Active Directory (Azure AD) pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. 

Comment afficher les connexions risquées à Azure AD : https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins

Comment configurer et activer des stratégies de protection des identités : https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Aide** : Customer Lockbox n’est actuellement pas pris en charge pour Azure Container Registry.

Liste des services pris en charge pour Customer Lockbox : https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability



**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Contrôle de sécurité : protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Aide** : Utilisez des étiquettes de ressource pour faciliter le suivi des registres de conteneurs Azure qui stockent ou traitent des informations sensibles.

Associez des étiquettes et des versions aux images de conteneur ou d’autres artefacts dans un registre, et verrouillez des images ou des référentiels pour faciliter le suivi des images qui stockent ou traitent des informations sensibles.

Comment créer et utiliser des étiquettes :  https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

Suggestions pour la création d’étiquettes et de versions pour les images de conteneurs :  https://docs.microsoft.com/azure/container-registry/container-registry-image-tag-version

Verrouiller une image dans un registre de conteneurs Azure :  https://docs.microsoft.com/azure/container-registry/container-registry-image-lock



**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Aide** : Implémentez des registres de conteneurs, des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Les ressources de stockage ou de traitement des données sensibles doivent être suffisamment isolées.

Les ressources doivent être séparées par un réseau virtuel ou un sous-réseau, étiquetés de manière appropriée et sécurisés par un groupe de sécurité réseau (NSG) ou un Pare-feu Azure.

Comment créer des abonnements Azure supplémentaires :  https://docs.microsoft.com/azure/billing/billing-create-subscription

Comment créer des groupes d’administration :  https://docs.microsoft.com/azure/governance/management-groups/create

Comment créer et utiliser des étiquettes : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

Restreindre l’accès à un registre de conteneurs Azure à l’aide d’un réseau virtuel Azure ou de règles de pare-feu : https://docs.microsoft.com/azure/container-registry/container-registry-vnet

Comment créer un NSG avec une configuration de sécurité : https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic

Déployer le Pare-feu Azure :

https://docs.microsoft.com/azure/firewall/tutorial-firewall-deploy-portal

Comment configurer Alerter ou une Alerte et refuser avec le pare-feu Azure :

https://docs.microsoft.com/azure/firewall/threat-intel



**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Aide** : Déployez un outil automatisé sur les périmètres du réseau qui surveille le transfert non autorisé d’informations sensibles et bloque ces transferts tout en alertant les professionnels de la sécurité des informations.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

Comprendre la protection des données client dans Azure :  https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Aide** : Assurez-vous que les clients qui se connectent à votre registre de conteneurs Azure peuvent négocier TLS 1.2 ou une version ultérieure. Par défaut, les ressources Microsoft Azure négocient TLS 1.2.

Suivez les recommandations d’Azure Security Center relatives au chiffrement au repos et au chiffrement en transit, le cas échéant.

Comprendre le chiffrement en transit avec Azure :  https://docs.microsoft.com/azure/security/fundamentals/encryption-overview#encryption-of-data-in-transit



**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Aide** : Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour Azure Container Registry. Implémentez une solution tierce si nécessaire à des fins de conformité.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

Comprendre la protection des données client dans Azure : https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Aide** : Utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour contrôler l’accès aux données et aux ressources dans un registre de conteneurs Azure. 

Comment configurer Azure RBAC : https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal

Autorisations et rôles Azure Container Registry :  https://docs.microsoft.com/azure/container-registry/container-registry-roles



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Conseils** : Si nécessaire à des fins de conformité sur les ressources de calcul, implémentez un outil tiers, tel qu’une solution automatisée de prévention contre la perte de données basée sur l’hôte, pour appliquer des contrôles d’accès aux données même lorsque des données sont copiées à partir d’un système.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

Comprendre la protection des données client dans Azure :  https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide** : Utilisez le chiffrement au repos sur toutes les ressources Azure. Par défaut, toutes les données d’un registre de conteneurs Azure sont chiffrées au repos à l’aide de clés gérées par Microsoft.

Comprendre le chiffrement au repos dans Azure : https://docs.microsoft.com/azure/security/fundamentals/encryption-atrest

Clés gérées par le client dans Azure Container Registry :  https://aka.ms/acr/cmk



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Azure Monitor collecte les journaux de ressources (anciennement appelés journaux de diagnostic) pour les événements pilotés par l’utilisateur dans votre registre. Collectez et consommez ces données pour auditer les événements d’authentification de registre et fournir une trace d’activité complète sur les artefacts du registre, tels que les événements d’envoi (push) et de tirage (pull), pour vous permettre de diagnostiquer les problèmes opérationnels avec votre registre.

Journaux d’Azure Container Registry pour l’évaluation et l’audit de diagnostics : https://docs.microsoft.com/azure/container-registry/container-registry-diagnostics-audit-logs


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Aide** : Suivez les recommandations d’Azure Security Center relatives à l’évaluation des vulnérabilités sur vos images de conteneur. Déployez éventuellement des solutions tierces à partir de la Place de marché Azure pour évaluer les vulnérabilités des images.

Comment implémenter les recommandations d’évaluation de la vulnérabilité d’Azure Security Center :  https://docs.microsoft.com/azure/security-center/security-center-vulnerability-assessment-recommendations

Intégration d’Azure Container Registry à Security Center (préversion) :  https://docs.microsoft.com/azure/security-center/azure-container-registry-integration



**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Aide** : Microsoft assure la gestion des correctifs sur les systèmes sous-jacents prenant en charge Azure Container Registry.

Automatisez les mises à jour des images de conteneur lorsque des mises à jour des images de base du système d’exploitation et d’autres correctifs sont détectées.

À propos des mises à jour des images de base pour les tâches Azure Container Registry :  https://docs.microsoft.com/azure/container-registry/container-registry-tasks-base-images


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3 : Déployer une solution de gestion de correctif logiciel tiers automatisée

**Aide** : Vous pouvez utiliser une solution tierce pour corriger les images d’application.  En outre, vous pouvez exécuter des tâches de Azure Container Registry pour automatiser les mises à jour des images d’application dans un registre de conteneurs en fonction des correctifs de sécurité ou d’autres mises à jour dans les images de base.

À propos des mises à jour des images de base pour ACR Tasks :  https://docs.microsoft.com/azure/container-registry/container-registry-tasks-base-images



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Aide** : Intégrez Azure Container Registry (ACR) à Azure Security Center pour activer l’analyse périodique des images de conteneur à la recherche de vulnérabilités. Déployez éventuellement des solutions tierces à partir de la Place de marché Azure pour effectuer des analyses périodiques de vulnérabilités des images.

Intégration d’Azure Container Registry à Security Center (préversion) :  https://docs.microsoft.com/azure/security-center/azure-container-registry-integration


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Aide** : Intégrez Azure Container Registry (ACR) à Azure Security Center pour activer l’analyse périodique des images de conteneur à la recherche de vulnérabilités et pour classifier les risques. Déployez éventuellement des solutions tierces à partir de la Place de marché Azure pour effectuer des analyses périodiques de vulnérabilités des images et la classification des risques.

Intégration d’Azure Container Registry à Security Center (préversion) :  https://docs.microsoft.com/azure/security-center/azure-container-registry-integration



**Supervision d’Azure Security Center** : N/A

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1 : Utiliser la découverte de ressources Azure

**Aide** : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (telles que calcul, stockage, réseau, ports et protocoles) dans vos abonnements.  Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

Comment créer des requêtes avec Azure Resource Graph : https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Comment voir vos abonnements Azure : https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0 

Comprendre Azure RBAC : https://docs.microsoft.com/azure/role-based-access-control/overview



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Aide** : Azure Container Registry gère les métadonnées, notamment les étiquettes et les manifestes pour les images dans un registre. Suivez les pratiques recommandées pour étiqueter les artefacts.

À propos des registres, des dépôts et des images : https://docs.microsoft.com/azure/container-registry/container-registry-concepts

Suggestions pour la création d’étiquettes et de versions pour les images de conteneurs : https://docs.microsoft.com/azure/container-registry/container-registry-image-tag-version


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Aide** : Azure Container Registry gère les métadonnées, notamment les étiquettes et les manifestes pour les images dans un registre. Suivez les pratiques recommandées pour étiqueter les artefacts.

À propos des registres, des dépôts et des images : https://docs.microsoft.com/azure/container-registry/container-registry-concepts

Suggestions pour la création d’étiquettes et de versions pour les images de conteneurs :  https://docs.microsoft.com/azure/container-registry/container-registry-image-tag-version



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4 : Tenir un inventaire des ressources Azure approuvées et titres des logiciels

**Aide** : Vous devez créer un inventaire des ressources Azure approuvées et des logiciels approuvés en fonction des besoins de votre organisation.  

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Aide** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements.

Utilisez Azure Resource Graph pour interroger/découvrir des ressources dans leurs abonnements.  Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

Auditer la conformité des registres de conteneurs Azure à l’aide d’Azure Policy :  https://docs.microsoft.com/azure/container-registry/container-registry-azure-policy

Comment configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Créer des requêtes avec Azure Graph :  https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : Analysez et supervisez les journaux Azure Container Registry pour détecter les comportements anormaux et examinez régulièrement les résultats. Utilisez l’espace de travail Log Analytics d’Azure Monitor pour examiner les journaux et effectuer des requêtes sur leurs données.

Journaux d’Azure Container Registry pour l’évaluation et l’audit de diagnostics :  https://docs.microsoft.com/azure/container-registry/container-registry-diagnostics-audit-logs

Comprendre l’espace de travail Log Analytics :  https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-portal

Comment exécuter des requêtes personnalisées dans Azure Monitor : https://docs.microsoft.com/azure/azure-monitor/log-query/get-started-queries


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Aide** : Azure Automation permet de contrôler totalement le déploiement, les opérations et la désaffectation des charges de travail et des ressources.  Vous pouvez implémenter votre propre solution  de suppression des ressources Azure non autorisées. Présentation d’Azure Automation :  https://docs.microsoft.com/azure/automation/automation-intro


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Aide** : Non applicable. Ce benchmark est destiné aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Aide** : Utilisez Azure Policy pour limiter les services que vous pouvez approvisionner dans votre environnement.

Auditer la conformité des registres de conteneurs Azure à l’aide d’Azure Policy :  https://docs.microsoft.com/azure/container-registry/container-registry-azure-policy

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Comment refuser un type de ressource spécifique avec Azure Policy :  https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-implement-approved-application-list"></a>6.10 : Implémenter une liste d’applications approuvées

**Aide** : Non applicable. Ce benchmark est destiné aux ressources de calcul.



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azureresources-manager-via-scripts"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager via des scripts

**Aide** : Utilisez des configurations de système d’exploitation spécifiques ou des ressources tierces pour limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul Azure.

Configurer l’accès conditionnel pour bloquer l’accès à Azure Resource Manager :  https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : Utilisez des configurations de système d’exploitation spécifiques ou des ressources tierces pour limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul Azure.

Par exemple, contrôler l’exécution d’un script PowerShell dans des environnements Windows :  https://docs.microsoft.com/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-6


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Aide** : Les logiciels nécessaires à des opérations métier, mais qui peuvent poser un risque élevé pour l’organisation, doivent être isolés sur leur propre machine virtuelle et/ou réseau virtuel et être suffisamment sécurisés à l’aide d’un Pare-feu Azure ou d’un groupe de sécurité réseau.

Créer un réseau virtuel :  https://docs.microsoft.com/azure/virtual-network/quick-create-portal

Comment créer un NSG avec une configuration de sécurité :  https://docs.microsoft.com/azure/virtual-network/tutorial-filter-network-traffic


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Utilisez Azure Policy ou Azure Security Center pour gérer les configurations de sécurité pour toutes les ressources Azure.

Comment configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Auditer la conformité des registres de conteneurs Azure à l’aide d’Azure Policy :  https://docs.microsoft.com/azure/container-registry/container-registry-azure-policy


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Aide** : Utilisez la recommandation « Corriger les vulnérabilités dans les configurations de sécurité sur vos machines virtuelles » Azure Security Center pour gérer les configurations de sécurité sur toutes les ressources de calcul.

Gérer les recommandations relatives à Azure Security Center :  https://docs.microsoft.com/azure/security-center/security-center-recommendations

Comment corriger les recommandations Azure Security Center :  https://docs.microsoft.com/azure/security-center/security-center-remediate-recommendations


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

Auditer la conformité des registres de conteneurs Azure à l’aide d’Azure Policy :  https://docs.microsoft.com/azure/container-registry/container-registry-azure-policy

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Comprendre les effets d’Azure Policy :  https://docs.microsoft.com/azure/governance/policy/concepts/effects



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Aide** : Non applicable. Ce benchmark a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous utilisez des définitions Azure Policy personnalisées, utilisez Azure Repos pour stocker et gérer votre code en toute sécurité.

Comment stocker du code dans Azure DevOps :  https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops

Documentation Azure Repos :  https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Aide** : Non applicable. Le test d’évaluation s’applique aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="77-deploy-system-configuration-management-tools"></a>7.7 : Déployer les outils de gestion de configuration système

**Aide** : Utilisez Azure Policy pour alerter, auditer et appliquer des configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

Auditer la conformité des registres de conteneurs Azure à l’aide d’Azure Policy :  https://docs.microsoft.com/azure/container-registry/container-registry-azure-policy

Comment configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation

**Aide** : Non applicable. Le test d’évaluation s’applique aux ressources de calcul.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la surveillance de la configuration automatique pour les services Azure

**Aide** : Utiliser Azure Security Center pour effectuer des analyses de ligne de base pour vos ressources Azure.

Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements.

Comment corriger les recommandations dans Azure Security Center :  https://docs.microsoft.com/azure/security-center/security-center-remediate-recommendations

Auditer la conformité des registres de conteneurs Azure à l’aide d’Azure Policy :  https://docs.microsoft.com/azure/container-registry/container-registry-azure-policy



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Aide** : Non applicable. Le test d’évaluation s’applique aux ressources de calcul.


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : Utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets pour vos applications Cloud.

Intégration aux identités managées Azure :  https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

Pour créer un coffre de clés : https://docs.microsoft.com/azure/key-vault/general/quick-create-portal

Comment s’authentifier auprès de Key Vault : https://docs.microsoft.com/azure/key-vault/general/authentication

Comment attribuer une stratégie d’accès Key Vault : https://docs.microsoft.com/azure/key-vault/general/assign-access-policy-portal

Utiliser une identité managée Azure dans Azure Container Registry Tasks :  https://docs.microsoft.com/azure/container-registry/container-registry-tasks-authentication-managed-identity


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Aide** : Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure AD. Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Key Vault, sans informations d’identification dans votre code.

Configuration d’identités managées :  https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm

Utiliser une identité managée pour s’authentifier auprès d’un registre de conteneurs Azure :  https://docs.microsoft.com/azure/container-registry/container-registry-authentication-managed-identity


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

Configuration de Credential Scanner :  https://secdevtools.azurewebsites.net/helpcredscan.html


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Aide** : Utilisez Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles afin de superviser et défendre en continu vos ressources. Pour Linux, utilisez une solution logicielle anti-programme malveillant tierce.

Comment configurer Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles :  https://docs.microsoft.com/azure/security/fundamentals/antimalware


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Aide** : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure Container Registry), mais il ne s’exécute pas sur du contenu client.

Pré-analysez les fichiers chargés sur des ressources Azure non liées au calcul, comme App Service, des Data Lake Storage, Stockage Blob, etc.


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Aide** : Non applicable. Ce benchmark a trait aux ressources de calcul. Microsoft gère les logiciels anti-programme malveillant pour la plateforme sous-jacente.


**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-recovery"></a>Récupération des données

*Pour plus d’informations, consultez [Contrôle de sécurité : récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Aide** : Les données de votre registre de conteneurs Microsoft Azure sont toujours répliquées pour en garantir la durabilité et la haute disponibilité. Azure Container Registry copie vos données afin qu’elles soient protégées contre les événements planifiés et les événements non planifiés

Vous pouvez également géo-répliquer un registre de conteneurs pour conserver les réplicas du Registre dans plusieurs régions Azure. 

Géoréplication dans Azure Container Registry :  https://docs.microsoft.com/azure/container-registry/container-registry-geo-replication



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Aide** : Sauvegardez éventuellement les images conteneur en les important d’un registre vers un autre.

Sauvegardez les clés gérées par le client dans Azure Key Vault à l’aide d’outils en ligne de commande ou de kits de développement logiciel (SDK) Azure.

Importer des images conteneur dans un registre de conteneurs :  https://docs.microsoft.com/azure/container-registry/container-registry-import-images

Comment sauvegarder des clés Key Vault dans Azure :  https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0


**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : valider toutes les sauvegardes, y compris les clés gérées par le client

**Aide** : Testez la restauration des clés gérées par le client sauvegardées dans Azure Key Vault à l’aide d’outils en ligne de commande ou de kits de développement logiciel (SDK) Azure.

Comment restaurer des clés Azure Key Vault dans Azure :  https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Vous pouvez activer la suppression réversible dans Azure Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.

Comment activer la suppression réversible dans Key Vault : https://docs.microsoft.com/azure/storage/blobs/storage-blob-soft-delete?tabs=azure-portal


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

Comment configurer des automatisations de workflow dans Azure Security Center :  https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide

Aide sur la création de votre propre processus de réponse aux incidents de sécurité :  https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/

Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft :  https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/

Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents :  https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Aide** : Azure Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez clairement les abonnements (par ex. production, non production) et créez un système d’attribution de noms pour identifier et classer les ressources Azure de façon claire.


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Aide** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

Reportez-vous à la publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques) :  https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Aide** : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé.  Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

Comment définir le contact de sécurité d’Azure Security Center :  https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details


**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser la sentinelle des alertes.

Comment configurer l’exportation continue :  https://docs.microsoft.com/azure/security-center/continuous-export

Comment diffuser en continu des alertes dans Azure Sentinel :  https://docs.microsoft.com/azure/sentinel/connect-azure-security-center


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

Comment configurer l’automatisation des workflows et Logic Apps :  https://docs.microsoft.com/azure/security-center/workflow-automation


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1 : Procédez régulièrement à des tests d’intrusion de vos ressources Azure et veillez à résoudre tous les problèmes de sécurité critiques détectés dans un délai de 60 jours

**Aide** : Suivre les règles d’engagement de Microsoft pour garantir que vos tests d’intrusion sont conformes aux stratégies de Microsoft :  https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

Vous trouverez plus d’informations sur la stratégie de Microsoft, sur l’exécution de Red Teaming, et sur les tests d’intrusion de site actif sur l’infrastructure, les services et les applications cloud managés par Microsoft ici :  https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consultez le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- Apprenez-en davantage sur les [Bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)
