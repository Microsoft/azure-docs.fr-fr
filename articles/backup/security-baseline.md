---
title: Ligne de base de sécurité Azure pour Sauvegarde Azure
description: Ligne de base de sécurité Azure pour Sauvegarde Azure
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/23/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: a2cbd2dd4dd32be137e461985e9f10fd07ed8a60
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96325636"
---
# <a name="azure-security-baseline-for-backup"></a>Ligne de base de sécurité Azure pour Sauvegarde Azure

La ligne de base de sécurité Azure pour Sauvegarde Azure contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1 : Protéger les ressources à l'aide de groupes de sécurité réseau ou du Pare-feu Azure sur votre réseau virtuel

**Conseils** : Non applicable ; vous ne pouvez pas associer un réseau virtuel, un sous-réseau ou un groupe de sécurité réseau à un coffre Recovery Services. Lors de la sauvegarde d’une machine virtuelle Azure, les données sont transférées sur le réseau principal Azure. Lors de la sauvegarde à partir d’un ordinateur local, un tunnel chiffré est créé avec un point de terminaison spécifique dans Azure, et les informations d’identification sont utilisées pour préchiffrer les données avant leur envoi via le tunnel chiffré.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2 : Surveiller et consigner la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Conseils** : Non applicable ; vous ne pouvez pas associer un réseau virtuel, un sous-réseau ou un groupe de sécurité réseau à un coffre Recovery Services. Lors de la sauvegarde d’une machine virtuelle Azure, les données sont transférées sur le réseau principal Azure. Lors de la sauvegarde à partir d’un ordinateur local, un tunnel chiffré est créé avec un point de terminaison spécifique dans Azure, et les informations d’identification sont utilisées pour préchiffrer les données avant leur envoi via le tunnel chiffré.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Aide** : Les points de terminaison utilisés par Sauvegarde Azure (y compris l’agent Microsoft Azure Recovery Services) sont tous gérés par Microsoft. Vous êtes responsable de tous les contrôles supplémentaires que vous souhaitez déployer sur vos systèmes locaux.

- [Présentation de la prise en charge de la mise en réseau et de l’accès pour l’agent MARS](./backup-support-matrix-mars-agent.md#networking-and-access-support)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="15-record-network-packets-and-flow-logs"></a>1.5 : Consigner les paquets réseau et les journaux de flux

**Conseils** : Non applicable ; vous ne pouvez pas associer un réseau virtuel, un sous-réseau ou un groupe de sécurité réseau à un coffre Recovery Services. Lors de la sauvegarde d’une machine virtuelle Azure, les données sont transférées sur le réseau principal Azure. Lors de la sauvegarde à partir d’ordinateurs locaux, un tunnel chiffré est créé avec un point de terminaison spécifique dans Azure, et les informations d’identification sont utilisées pour préchiffrer les données avant leur envoi via le tunnel chiffré.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Aide** : Les points de terminaison utilisés par Sauvegarde Azure (y compris l’agent Microsoft Azure Recovery Services) sont tous gérés par Microsoft. Vous êtes responsable de tous les contrôles supplémentaires que vous souhaitez déployer sur vos systèmes locaux.

- [Présentation de la prise en charge de la mise en réseau et de l’accès pour l’agent MARS](./backup-support-matrix-mars-agent.md#networking-and-access-support)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : Si vous utilisez l’agent MARS sur une machine virtuelle Azure, utilisez la balise de service AzureBackup sur votre groupe de sécurité réseau ou votre Pare-feu Azure pour autoriser l’accès sortant vers Sauvegarde Azure.

- [Sauvegarder des bases de données SQL Server sur des machines virtuelles Azure](./backup-sql-server-database-azure-vms.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : Non applicable ; les points de terminaison utilisés par Sauvegarde Azure (y compris l’agent Microsoft Azure Recovery Services) sont tous gérés par Microsoft.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Aide** : Si vous utilisez l’agent MARS sur une machine virtuelle Azure, associez cette machine virtuelle à un groupe de sécurité réseau. Utilisez la description pour spécifier le besoin professionnel de la règle.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Aide** : Si vous utilisez l’agent MARS sur une machine virtuelle Azure protégée par un NSG ou un Pare-feu Azure, utilisez le journal d’activité Azure pour surveiller la configuration du groupe de sécurité réseau ou du Pare-feu. Vous pouvez créer des alertes dans Azure Monitor qui se déclenchent lorsque des modifications sont apportées à ces ressources.

- [Afficher et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Créer, afficher et gérer des alertes de journal d’activité à l’aide d’Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Contrôle de sécurité : journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Aide** : Non applicable ; Microsoft conserve la source de temps utilisée pour les ressources Azure, par exemple, Sauvegarde Azure, pour les timestamps dans les journaux.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Pour la journalisation d’audit du plan de contrôle, activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux vers un espace de travail Log Analytics, Event Hub ou un compte de stockage Azure pour archivage. À l’aide des données des journaux d’activité Azure, vous pouvez déterminer les éléments « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de contrôle pour vos ressources Azure.

Ingérez également les journaux via Azure Monitor pour agréger les données de sécurité générées par Sauvegarde Azure. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, puis utilisez les comptes de stockage pour le stockage à long terme/d’archivage. Vous pouvez également activer et intégrer des données dans Azure Sentinel ou une solution SIEM (Security Information and Event Management) tierce.

- [Activation des paramètres de diagnostic pour le journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Utilisation des paramètres de diagnostic pour les coffres Recovery Services](./backup-azure-diagnostic-events.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Aide** : Pour la journalisation d’audit du plan de contrôle, activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux vers un espace de travail Log Analytics, Event Hub ou un compte de stockage Azure pour archivage. À l’aide des données des journaux d’activité Azure, vous pouvez déterminer les éléments « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de contrôle pour vos ressources Azure.

En outre, Sauvegarde Azure envoie des événements de diagnostic qui peuvent être collectés et utilisés à des fins d’analyse, d’alerte et de création de rapports. Vous pouvez configurer les paramètres de diagnostic d’un coffre Recovery Services par le biais du portail Azure. Vous pouvez envoyer un ou plusieurs événements de diagnostic à un compte de stockage, un Event Hub ou un espace de travail Log Analytics.

- [Activation des paramètres de diagnostic pour le journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Utilisation des paramètres de diagnostic pour les coffres Recovery Services](./backup-azure-diagnostic-events.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Aide** : Dans Azure Monitor, définissez la période de rétention des journaux pour les espaces de travail Log Analytics associés à vos coffres Recovery Services conformément aux réglementations de conformité de votre organisation.

- [Définition des paramètres de rétention des journaux](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Sauvegarde Azure fournit des capacités intégrées de supervision et d’alerte dans un coffre Recovery Services. Ces fonctionnalités sont disponibles sans infrastructure de gestion supplémentaire. Vous pouvez également augmenter l’échelle de votre supervision et de vos rapports à l’aide d’Azure Monitor.

Activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux à un espace de travail Log Analytics. Exécutez des requêtes dans Log Analytics pour rechercher des termes, identifier des tendances, analyser des modèles et fournir de nombreuses autres informations basées sur les données du journal d’activité qui ont pu être collectées pour les coffres Recovery Services.

- [Surveillance des charges de travail de Sauvegarde Azure](./backup-azure-monitoring-built-in-monitor.md)

- [Activation des paramètres de diagnostic pour le journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Collecte et analyse des journaux d’activité Azure dans l’espace de travail Log Analytics dans Azure Monitor](../azure-monitor/platform/activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer des alertes en cas d’activité anormale

**Aide** : Sauvegarde Azure fournit des capacités intégrées de supervision et d’alerte dans un coffre Recovery Services. Ces fonctionnalités sont disponibles sans infrastructure de gestion supplémentaire. Vous pouvez également augmenter l’échelle de votre supervision et de vos rapports à l’aide d’Azure Monitor.

Les alertes sont principalement des scénarios dans le cadre desquels les utilisateurs reçoivent des notifications leur permettant de prendre les mesures nécessaires. La section Alertes de sauvegarde affiche les alertes générées par le service Sauvegarde Azure. Ces alertes sont définies par le service et vous ne pouvez pas créer d’alerte de façon personnalisée.

Vous pouvez également intégrer un espace de travail Log Analytics à Azure Sentinel, car cela fournit une solution d’orchestration, d’automatisation et de réponse aux incidents de sécurité informatique (Security Orchestration Automated Response, SOAR). Cela permet de créer des playbooks (solutions automatisées) utilisables pour corriger des problèmes de sécurité. En outre, vous pouvez créer des alertes de journal personnalisées dans votre espace de travail Log Analytics à l’aide d’Azure Monitor.

- [Surveillance des charges de travail de Sauvegarde Azure](./backup-azure-monitoring-built-in-monitor.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Créer, afficher et gérer des alertes de journal à l’aide d’Azure Monitor](../azure-monitor/platform/alerts-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide** : Non applicable ; Sauvegarde Azure ne traite pas ni ne produit de journaux liés aux logiciels anti-programmes malveillants.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Aide** : Non applicable ; Sauvegarde Azure ne traite pas ni ne produit de journaux liés au DNS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Azure Active Directory (AD) comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent être interrogés. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration.

Documents associés :

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole)

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Aide** : Azure AD n’intègre pas le concept des mots de passe par défaut. Selon le service, d’autres ressources Azure qui exigent un mot de passe forcent la création d’un mot de passe conforme à des exigences de complexité et d’une longueur minimale. Vous êtes responsable des applications tierces et des services de la place de marché susceptibles d’utiliser des mots de passe par défaut.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

En outre, pour vous aider à suivre les comptes d’administration dédiés, vous pouvez utiliser des recommandations d’Azure Security Center ou des stratégies Azure intégrées, telles que les suivantes : Plus d’un propriétaire doit être assigné à votre abonnement. Les comptes déconseillés possédant des autorisations de propriétaire doivent être supprimés de votre abonnement. Les comptes externes possédant des autorisations de propriétaire doivent être supprimés de votre abonnement.

- [Utilisation d’Azure Security Center pour superviser l’identité et l’accès (préversion)](../security-center/security-center-identity-access.md)

- [Utilisation d’Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Aide** : Utilisez une inscription d’application Azure (principal du service) afin de récupérer un jeton qui peut être utilisé pour interagir avec vos coffres Recovery Services via des appels d’API.

- [Appels d’API REST Azure](/rest/api/azure/#how-to-call-azure-rest-apis-with-postman)

- [Inscription de votre application cliente (principal du service) à l’aide d’Azure AD](/rest/api/azure/#register-your-client-application-with-azure-ad)

- [Informations sur l’API Azure Recovery Services](/rest/api/recoveryservices/)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : Lorsque vous effectuez des opérations critiques dans Sauvegarde Azure, vous devez entrer un code PIN de sécurité, disponible sur le Portail Azure. L'activation de la fonctionnalité Azure AD Multi-Factor Authentication offre une couche de sécurité supplémentaire. Seuls les utilisateurs autorisés, disposant d’informations d’identification Azure valides et authentifiés à partir d’un deuxième appareil peuvent accéder au portail Azure.

- [Multi-Factor Authentication dans Sauvegarde Azure](./backup-azure-security-feature.md)

- [Planifier un déploiement informatique d'Azure AD Multi-Factor Authentication](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils** : Utilisez une station de travail à accès privilégié sur laquelle le service Azure AD Multi-Factor Authentication (MFA) est configuré pour vous connecter à vos ressources compatibles avec Sauvegarde Azure et les configurer.

- [Stations de travail d’accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Planifier un déploiement informatique d'Azure AD Multi-Factor Authentication](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration

**Aide** : Utilisez Azure Active Directory (AD) Privileged Identity Management pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.

Utilisez également les détections de risque Azure AD pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs.

- [Déploiement de Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

- [Présentation des détections de risques Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Utilisez des emplacements nommés avec accès conditionnel pour autoriser l’accès au Portail Azure uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

- [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Utilisez Azure Active Directory (AD) comme système central d’authentification et d’autorisation pour vos instances Sauvegarde Azure. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

- [Configuration de Sauvegarde Azure pour utiliser la connexion Azure AD](../app-service/configure-authentication-provider-aad.md)

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory (AD) fournit des journaux pour vous aider à découvrir les comptes obsolètes. De plus, utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seules les utilisateurs appropriés continuent de bénéficier d’un accès.

- [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml)

- [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés

**Aide** : Utilisez Azure Active Directory (AD) comme système central d’authentification et d’autorisation pour vos instances Sauvegarde Azure. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

Vous avez accès aux activités de connexion Azure AD et aux sources des journaux des événements à risque et des audits, ce qui vous permet de les intégrer à Azure Sentinel ou à un outil SIEM tiers.

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure AD et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes de journal souhaitées dans un espace de travail Log Analytics.

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Procédure d’intégration d’Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez Azure Active Directory (AD) comme système central d’authentification et d’autorisation pour vos coffres Recovery Services. Pour les écarts du comportement de connexion au compte sur le plan de contrôle (le Portail Azure), utilisez Azure AD Identity Protection et les fonctionnalités de détections de risques pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. Vous pouvez également ingérer des données dans Azure Sentinel pour approfondir votre examen.

- [Configuration de Sauvegarde Azure pour utiliser la connexion Azure AD](../app-service/configure-authentication-provider-aad.md)

- [Guide pratique pour afficher une connexion risquée Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Aide** : Actuellement non disponible ; Customer Lockbox n’est pas encore pris en charge pour Sauvegarde Azure.

- [Liste des services pris en charge pour Customer Lockbox](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Contrôle de sécurité : protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Lors de la sauvegarde de machines virtuelles IaaS Azure, Sauvegarde Azure fournit des sauvegardes indépendantes et isolées pour éviter une destruction accidentelle des données d’origine. Les sauvegardes sont stockées dans un coffre Recovery Services avec gestion intégrée des points de récupération.

Implémentez des abonnements ou des groupes d’administration distincts pour le développement, les tests et la production de coffres Recovery Services. Les ressources doivent être séparées par un réseau virtuel/sous-réseau, marquées correctement et sécurisées par un groupe de sécurité réseau ou un pare-feu Azure. Les ressources de stockage ou de traitement des données sensibles doivent être suffisamment isolées. Pour les machines virtuelles qui stockent ou traitent des données sensibles, implémentez la stratégie et les procédures pour les désactiver lorsqu’elles ne sont pas utilisées.

Documents associés :

- [Vue d’ensemble d’Azure Backup](./backup-overview.md)

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Aide** : Actuellement non disponible ; les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour Sauvegarde Azure.

Microsoft gère l’infrastructure sous-jacente de Sauvegarde Azure et a implémenté des contrôles stricts pour empêcher la perte ou la divulgation des données des clients.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Le trafic de sauvegarde des serveurs vers le coffre Recovery Services est transféré via une liaison HTTPS sécurisée et chiffré à l’aide de l’algorithme Advanced Encryption Standard (AES) 256 lorsqu’il est stocké dans le coffre.

- [Présentation du chiffrement au repos dans Sauvegarde Azure](./backup-encryption.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Aide** : Actuellement non disponible ; les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour Sauvegarde Azure.

Microsoft gère l’infrastructure sous-jacente de Sauvegarde Azure et a implémenté des contrôles stricts pour empêcher la perte ou la divulgation des données des clients.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : actuellement non disponible

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Aide** : Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) permet une gestion des accès affinée pour Azure. Avec Azure RBAC, vous pouvez séparer les tâches au sein de votre équipe et accorder aux utilisateurs uniquement les accès nécessaires pour accomplir leur travail.

Le service Sauvegarde Azure fournit trois rôles intégrés pour contrôler les opérations de gestion des sauvegardes : Contributeur de sauvegarde, Opérateur de sauvegarde et Lecteur de sauvegarde. Vous pouvez mapper les rôles intégrés de Sauvegarde à différentes actions de gestion des sauvegardes.

- [Comment configurer Azure RBAC](../role-based-access-control/role-assignments-portal.md)

- [Utilisation du contrôle d’accès en fonction du rôle Azure pour gérer les points de récupération Sauvegarde Azure](./backup-rbac-rs-vault.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Aide** : Non applicable. Cette recommandation a trait aux ressources de calcul. Microsoft gère l’infrastructure sous-jacente de Sauvegarde Azure et a implémenté des contrôles stricts pour empêcher la perte ou la divulgation des données des clients.

- [Protection des données client Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide** : Sauvegarde Azure prend en charge le chiffrement des données au repos. Pour la sauvegarde sur site, le chiffrement au repos est assuré à l’aide de la phrase secrète que vous fournissez lorsque vous sauvegardez sur Azure. Pour les charges de travail cloud, les données sont chiffrées au repos à l’aide de Storage Service Encryption (SSE). Microsoft ne déchiffre les données de sauvegarde à aucun moment.

Lorsque vous sauvegardez avec l’agent MARS ou à l’aide d’un coffre Recovery Services chiffré avec une clé gérée par le client, vous seul avez accès à la clé de chiffrement. Microsoft ne conserve jamais de copie de la clé de chiffrement et n’y a pas accès. Si la clé est égarée, Microsoft ne peut pas récupérer les données de sauvegarde.

- [Présentation du chiffrement au repos dans Sauvegarde Azure](./backup-encryption.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes en cas de modifications sur des coffres Azure Recovery Services en production et d’autres ressources cruciales ou associées.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : Pas encore disponible ; l’évaluation des vulnérabilités dans Azure Security Center n’est pas encore disponible pour Sauvegarde Azure.

La plateforme sous-jacente est analysée et corrigée par Microsoft. Examinez les contrôles de sécurité disponibles pour Sauvegarde Azure afin de réduire les vulnérabilités liées à la configuration du service.

- [Comprendre les contrôles de sécurité disponibles pour Sauvegarde Azure]()

**Supervision Azure Security Center** : actuellement non disponible

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

**Aide** : Actuellement non disponible ; les configurations de sécurité pour Sauvegarde Azure ne sont pas encore prises en charge dans Azure Security Center.

- [Liste des services PaaS pris en charge par Azure Security Center](../security-center/features-paas.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1 : Utiliser la découverte de ressources Azure

**Aide** : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (telles que calcul, stockage, réseau, ports et protocoles) dans vos abonnements.  Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

- [Guide pratique pour créer des requêtes avec Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription)

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Aide** : Utilisez des étiquettes, des groupes d’administration et des abonnements distincts, le cas échéant, pour organiser et suivre les ressources Azure. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

En outre, utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients à l’aide des définitions de stratégie intégrées suivantes : Types de ressources non autorisés et Types de ressources autorisés.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4 : Tenir un inventaire des ressources Azure approuvées et titres des logiciels

**Aide** : Définissez des ressources Azure approuvées et des logiciels approuvés pour les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements.

Utilisez Azure Resource Graph pour interroger/découvrir des ressources dans leurs abonnements.  Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Aide** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes : Types de ressources non autorisés et Types de ressources autorisés.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-implement-approved-application-list"></a>6.10 : Implémenter une liste d’applications approuvées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="611-divlimit-users-ability-to-interact-with-azure-resource-manager-via-scriptsdiv"></a>6.11 : <div>Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager via des scripts</div>

**Aide** : Configurez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

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

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Définissez et implémentez des configurations de sécurité standard pour votre coffre Recovery Services à l’aide d’Azure Policy. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.RecoveryServices » pour créer des stratégies personnalisées permettant d’auditer vos coffres Recovery Services ou d’appliquer leur configuration.

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Présentation des effets d’Azure Policy](../governance/policy/concepts/effects.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous utilisez des définitions Azure Policy personnalisées, utilisez Azure DevOps ou Azure Repos pour stocker et gérer votre code en toute sécurité.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow)

- [Documentation Azure Repos](/azure/devops/repos/index)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7 : Déployer les outils de gestion de configuration système

**Conseils** : Utilisez les définitions Azure Policy intégrées ainsi que des alias Azure Policy dans l’espace de noms « Microsoft.RecoveryServices » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la surveillance de la configuration automatique pour les services Azure

**Conseils** : Utilisez les définitions Azure Policy intégrées ainsi que des alias Azure Policy dans l’espace de noms « Microsoft.RecoveryServices » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. Utilisez une stratégie Azure Policy [auditer], [refuser] et [déployer s’il n’existe pas] afin d’appliquer automatiquement des configurations pour vos ressources Azure.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Aide** : Lors de la configuration de l’agent MARS, stockez votre phrase secrète de chiffrement dans Azure Key Vault.

- [Créer un coffre de clés](../key-vault/secrets/quick-create-portal.md)

* [Comment s’authentifier auprès de Key Vault](../key-vault/general/authentication.md)

* [Comment attribuer une stratégie d’accès Key Vault](../key-vault/general/assign-access-policy-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils** : Non applicable ; les identités managées ne sont pas prises en charge pour Sauvegarde Azure.

- [Services prenant en charge les identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Aide** : Non applicable ; cette recommandation est destinée aux ressources de calcul. Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Sauvegarde Azure), mais il ne s’exécute pas sur le contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Aide** : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Sauvegarde Azure), mais il ne s’exécute pas sur votre contenu.

Pré-analysez les fichiers chargés sur des ressources Azure non liées au calcul, comme App Service, Data Lake Storage et Stockage Blob.

Utilisez la détection des menaces pour les services de données dans Azure Security Center pour détecter les programmes malveillants chargés sur les comptes de stockage.

- [Présentation de Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles](../security/fundamentals/antimalware.md)

- [Présentation de la détection des menaces pour les services de données d’Azure Security Center](../security-center/azure-defender.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-recovery"></a>Récupération des données

*Pour plus d’informations, consultez [Contrôle de sécurité : récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Aide** : Non applicable ; cette recommandation est destinée aux ressources en cours de sauvegarde et non au service Sauvegarde Azure lui-même.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Aide** : Le stockage localement redondant (LRS) réplique vos données trois fois (il crée trois copies de vos données) dans une unité d’échelle de stockage dans un centre de données. Toutes les copies des données existent dans la même région. Le stockage LRS est une option à faible coût qui protège vos données contre les défaillances matérielles locales. Le stockage géoredondant (GRS) est l’option de réplication recommandée par défaut. Le stockage géo-redondant réplique vos données vers une région secondaire, distante de plusieurs centaines de kilomètres de l’emplacement principal des données sources. Le stockage GRS est plus onéreux que le stockage LRS, mais il offre une durabilité des données supérieure, même en cas de panne au niveau régional.

Clés de sauvegarde gérées par le client dans Azure Key Vault.

- [Vue d’ensemble d’Azure Backup](./backup-overview.md)

- [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

- [Présentation du chiffrement dans Sauvegarde Azure](./backup-encryption.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : valider toutes les sauvegardes, y compris les clés gérées par le client

**Aide** : Testez la restauration des clés gérées par le client sauvegardées.

- [Guide pratique pour restaurer des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Pour la sauvegarde sur site, le chiffrement au repos est assuré à l’aide de la phrase secrète que vous fournissez lorsque vous sauvegardez sur Azure. Pour les machines virtuelles Azure, les données sont chiffrées au repos à l’aide de Storage Service Encryption (SSE). Vous pouvez activer la suppression réversible dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.

- [Activation de la suppression réversible dans Key Vault](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

- [Comment configurer des automatisations de workflow dans Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Vous pouvez également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de votre propre plan de réponse aux incidents](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez clairement les abonnements (par ex. production, non production) et créez un système d’attribution de noms pour identifier et classer les ressources Azure de façon claire.

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

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

*Pour plus d’informations, consultez [Contrôle de sécurité : tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1 : Procédez régulièrement à des tests d’intrusion de vos ressources Azure et veillez à résoudre tous les problèmes de sécurité critiques détectés dans un délai de 60 jours

**Aide** : [Respectez les règles d’engagement de Microsoft pour garantir que vos tests d’intrusion sont conformes aux stratégies Microsoft.](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Vous trouverez plus d’informations sur la stratégie de Microsoft, sur l’exécution de Red Teaming et sur les tests d’intrusion de site actif sur l’infrastructure, les services et les applications cloud managés par Microsoft ici.](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consultez le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- Apprenez-en davantage sur les [Bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)