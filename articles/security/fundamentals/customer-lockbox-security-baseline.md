---
title: Base de la sécurité Azure pour Customer Lockbox pour Microsoft Azure
description: Base de la sécurité Azure pour Customer Lockbox pour Microsoft Azure
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 06/05/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 6cf106f6d7aef4324eeeaf232a28cdd91a8a774b
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94843126"
---
# <a name="azure-security-baseline-for-customer-lockbox-for-microsoft-azure"></a>Base de la sécurité Azure pour Customer Lockbox pour Microsoft Azure

La base de la sécurité Azure pour Customer Lockbox pour Microsoft Azure contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez la [Vue d’ensemble des bases de référence de sécurité Azure](../benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : Sécurité réseau](../benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1 : Protéger les ressources à l'aide de groupes de sécurité réseau ou du Pare-feu Azure sur votre réseau virtuel

**Conseils** : non applicable. Vous ne pouvez pas associer un réseau virtuel, un sous-réseau ou un groupe de sécurité réseau avec Customer Lockbox.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2 : Surveiller et consigner la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Conseils** : non applicable. Vous ne pouvez pas associer un réseau virtuel, un sous-réseau ou un groupe de sécurité réseau avec Customer Lockbox.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Conseils** : non applicable. Vous ne pouvez pas associer un réseau virtuel, un sous-réseau ou un groupe de sécurité réseau avec Customer Lockbox.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="15-record-network-packets-and-flow-logs"></a>1.5 : Consigner les paquets réseau et les journaux de flux

**Conseils** : non applicable. Vous ne pouvez pas associer un réseau virtuel, un sous-réseau ou un groupe de sécurité réseau avec Customer Lockbox.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Conseils** : non applicable. Vous ne pouvez pas associer un réseau virtuel, un sous-réseau ou un groupe de sécurité réseau avec Customer Lockbox.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : non applicable. Vous ne pouvez pas associer un réseau virtuel, un sous-réseau ou un groupe de sécurité réseau avec Customer Lockbox.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : non applicable. Vous ne pouvez pas associer un réseau virtuel, un sous-réseau ou un groupe de sécurité réseau avec Azure Customer Lockbox.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : non applicable. Vous ne pouvez pas associer un réseau virtuel, un sous-réseau ou un groupe de sécurité réseau avec Customer Lockbox.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : non applicable. Vous ne pouvez pas associer un réseau virtuel, un sous-réseau ou un groupe de sécurité réseau avec Customer Lockbox. Il n’y a aucune configuration réseau à effectuer ou à superviser.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Contrôle de sécurité : Journalisation et supervision](../benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Conseils** : non applicable. Microsoft gère la source de temps utilisée pour des ressources telles que Customer Lockbox.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : Les journaux d’audit de Customer Lockbox sont automatiquement activés et gérés dans les Journaux d’activité Azure. Vous pouvez afficher ces données en les transmettant en continu du journal d’activité Azure vers un espace de travail Log Analytics, où vous pouvez ensuite effectuer des recherches et des analyses à partir de ces données.

Intégrez les journaux d’activité générés par Customer Lockbox à Azure Sentinel ou une autre solution SIEM pour activer l’agrégation et la gestion des journaux centraux.

* [Journaux d’audit pour Customer Lockbox](./customer-lockbox-overview.md#auditing-logs)

* [Guide pratique pour intégrer Azure Sentinel](../../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Les journaux d’audit de Customer Lockbox sont automatiquement activés et gérés dans les Journaux d’activité Azure. Vous pouvez afficher ces données en les transmettant en continu du journal d’activité Azure vers un espace de travail Log Analytics, où vous pouvez ensuite effectuer des recherches et des analyses à partir de ces données.

* [Journaux d’audit pour Customer Lockbox](./customer-lockbox-overview.md#auditing-logs)

* [Guide pratique pour intégrer Azure Sentinel](../../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Dans Azure Monitor, définissez la période de conservation des journaux pour les espaces de travail Log Analytics associés à votre Customer Lockbox, conformément aux réglementations de conformité de votre organisation.

* [Guide pratique pour définir les paramètres de conservation des journaux](../../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : Les journaux d’audit de Customer Lockbox sont automatiquement activés et gérés dans les Journaux d’activité Azure. Vous pouvez afficher ces données en les transmettant en continu du journal d’activité Azure vers un espace de travail Log Analytics, où vous pouvez ensuite effectuer des recherches et des analyses à partir de ces données. Analysez et supervisez les journaux de vos demandes de vos demandes Customer Lockbox pour détecter tout comportement anormal. Utilisez la section « journaux » de votre espace de travail Azure Sentinel pour exécuter des requêtes, ou créez des alertes basées sur vos journaux de Customer Lockbox.

* [Journaux d’audit dans Customer Lockbox](./customer-lockbox-overview.md#auditing-logs)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer des alertes en cas d’activité anormale

**Conseils** : Les journaux d’audit de Customer Lockbox sont automatiquement activés et gérés dans les Journaux d’activité Azure. Vous pouvez afficher ces données en les transmettant en continu du journal d’activité Azure vers un espace de travail Log Analytics, où vous pouvez ensuite effectuer des recherches et des analyses à partir de ces données. Analysez et supervisez les journaux de vos demandes de vos demandes Customer Lockbox pour détecter tout comportement anormal. Utilisez la section « journaux » de votre espace de travail Azure Sentinel pour exécuter des requêtes, ou créez des alertes basées sur vos journaux de Customer Lockbox.

* [Journaux d’audit dans Customer Lockbox](./customer-lockbox-overview.md#auditing-logs)

* [Guide pratique pour générer une alerte sur des données de journal Log Analytics](../../azure-monitor/learn/tutorial-response.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Conseils** : non applicable. Customer Lockbox ne traite pas et ne produit pas de journaux liés aux logiciels anti-programmes malveillants.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Conseils** : non applicable. Customer Lockbox ne traite pas et ne produit pas de journaux liés au DNS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : conservez un inventaire des comptes d’utilisateur qui ont un accès administratif à vos demandes Customer Lockbox. Vous pouvez utiliser le volet de gestion des identités et des accès (IAM) dans le portail Azure de votre abonnement afin de configurer le contrôle d’accès en fonction du rôle d’Azure (Azure RBAC). Les rôles sont appliqués aux utilisateurs, aux groupes, aux principaux de service et aux identités gérées dans Azure Active Directory.

Dans l’organisation du client, l’utilisateur qui a le rôle de Propriétaire pour l’abonnement Azure reçoit un e-mail de la part de Microsoft l’informant de tout demande d’accès en attente. Pour les demandes Customer Lockbox, cette personne est l’approbateur désigné.

* [Comprendre les rôles personnalisés](../../role-based-access-control/custom-roles.md)

* [Comment configurer Azure RBAC pour les classeurs](../../sentinel/quickstart-get-visibility.md)

* [Comprendre les autorisations de demande d’accès dans Customer Lockbox](./customer-lockbox-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : Azure Active Directory n’intègre pas le concept des mots de passe par défaut. Selon le service, d’autres ressources Azure qui exigent un mot de passe forcent la création d’un mot de passe conforme à des exigences de complexité et d’une longueur minimale. Vous êtes responsable des applications tierces et des services de la place de marché susceptibles d’utiliser des mots de passe par défaut.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

En outre, pour vous aider à suivre les comptes d’administration dédiés, vous pouvez utiliser des recommandations d’Azure Security Center ou des stratégies Azure intégrées, telles que les suivantes :
- Plusieurs propriétaires doivent être affectés à votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

* [Utilisation d’Azure Security Center pour superviser l’identité et l’accès (préversion)](../../security-center/security-center-identity-access.md)

* [Utilisation d’Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : non applicable. L’accès à Customer Lockbox s’effectue via le portail Azure et est réservé aux comptes ayant le rôle de locataire Propriétaire. L’authentification unique n’est pas prise en charge.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : Activer l’authentification multifacteur Azure Active Directory et suivre les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

* [Guide pratique pour activer l’authentification MFA dans Azure](../../active-directory/authentication/howto-mfa-getstarted.md)

* [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils** : Utilisez une station de travail à accès privilégié sur laquelle Azure AD Multi-Factor Authentication (MFA) est activé pour vous connecter à vos demandes Customer Lockbox et les configurer.

* [Stations de travail d’accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Planifier un déploiement informatique d'Azure AD Multi-Factor Authentication](../../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration

**Conseils** : Utilisez Azure Active Directory Privileged Identity Management pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.

Utilisez également les détections de risque Azure Active Directory pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs.

* [Déploiement de Privileged Identity Management (PIM)](../../active-directory/privileged-identity-management/pim-deployment-plan.md)

* [Présentation des détections de risques Azure AD](../../active-directory/identity-protection/overview-identity-protection.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Utilisez des emplacements nommés avec accès conditionnel pour autoriser l’accès au Portail Azure uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

* [Guide pratique pour configurer des emplacements nommés dans Azure](../../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Conseils** : le cas échéant, utiliser Azure Active Directory comme système d’authentification et d’autorisation central. Azure Active Directory protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, Azure Active Directory sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

* [Comment créer et configurer une instance Azure Active Directory](../../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Conseils** : Azure Active Directory fournit des journaux pour vous aider à découvrir les comptes obsolètes. Par ailleurs, utilisez les révisions d’accès des identités Azure Active Directory pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seules les utilisateurs appropriés continuent de bénéficier d’un accès.

* [Comprendre les rapports Azure Active Directory](../../active-directory/reports-monitoring/index.yml)

* [Comment utiliser les révisions d’accès Azure Active Directory](../../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés

**Conseils** : le cas échéant, utiliser Azure Active Directory comme système d’authentification et d’autorisation central. Azure Active Directory protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, Azure Active Directory sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

Vous avez accès aux activités de connexion Azure Active Directory, aux sources des journaux d’événements à risque et d’audit, ce qui vous permet de les intégrer à Azure Sentinel ou à un outil SIEM tiers.

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure Active Directory et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes de journal souhaitées dans un espace de travail Log Analytics.

* [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

* [Procédure d’intégration d’Azure Sentinel](../../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Conseils** : Pour les écarts de comportement de connexion au compte sur le plan de contrôle (le portail Azure), utilisez Azure Active Directory Identity Protection et les fonctionnalités de détection de risques afin de configurer des réponses automatiques aux actions suspectes détectées liées aux identités des utilisateurs. Vous pouvez également ingérer des données dans Azure Sentinel pour approfondir votre examen.

* [Comment afficher les connexions risquées Azure Active Directory](../../active-directory/identity-protection/overview-identity-protection.md)

* [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

* [Guide pratique pour intégrer Azure Sentinel](../../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : cette recommandation ne s’applique pas à Customer Lockbox.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-protection"></a>Protection de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Protection des données](../benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : cette recommandation n’est pas applicable. Les balises ne sont pas prises en charge pour Customer Lockbox.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : non applicable. Customer Lockbox sera approvisionné dans le même abonnement que les ressources auxquelles vous accordez l’accès. Il n’existe aucun point de terminaison public à protéger ou à isoler. Un accès de demande Customer Lockbox est accordé à l’utilisateur qui détient le rôle Propriétaire au niveau du locataire.

* [Comprendre le flux de travail de Customer Lockbox](./customer-lockbox-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : Microsoft gère l’infrastructure sous-jacente de Customer Lockbox, et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

* [Présentation de la protection des données client dans Azure](./protection-customer-data.md)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Par défaut, Microsoft utilise le protocole Transport Layer Security (TLS) pour protéger les données lorsqu’elles sont en déplacement entre les services cloud et les clients. Les centres de données Microsoft négocient une connexion TLS avec les systèmes clients qui se connectent aux services Azure. TLS fournit une authentification forte, la confidentialité et l’intégrité des messages (activation de la détection de falsification et d’interception des messages), l’interopérabilité, la flexibilité des algorithmes, ainsi que la facilité de déploiement et d’utilisation.

* [Présentation du chiffrement en transit avec Azure](./encryption-overview.md#encryption-of-data-in-transit)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Microsoft

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Conseils** : non applicable. Customer Lockbox ne contient pas de données client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : Une approbation de demande Customer Lockbox est accordée à l’utilisateur qui détient le rôle Propriétaire au niveau du locataire.

* [Comprendre le flux de travail de Customer Lockbox](./customer-lockbox-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul. Microsoft gère l’infrastructure sous-jacente de Customer Lockbox, et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

* [Protection des données client Azure](./protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : non applicable. Customer Lockbox ne contient pas de données client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : les journaux d’audit pour Customer Lockbox sont automatiquement activés et gérés dans les Journaux d’activité Azure. Utilisez le journal d’activité Azure pour surveiller et détecter les modifications apportées aux ressources Azure Customer Lockbox. Créez des alertes dans Azure Monitor, qui se déclenchent en cas de modification des ressources critiques.

* [Comment activer l’audit dans Customer Lockbox](./customer-lockbox-overview.md)

* [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../../azure-monitor/platform/activity-log.md#view-the-activity-log)

* [Guide pratique pour créer des alertes dans Azure Monitor](../../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : Non applicable ; Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Customer Lockbox.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3 : Déployer une solution de gestion de correctif logiciel tiers automatisée

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils** : Non applicable ; Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Customer Lockbox.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : Non applicable ; Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Customer Lockbox.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1 : Utiliser la découverte de ressources Azure

**Conseils** : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (telles que calcul, stockage, réseau, ports et protocoles) dans vos abonnements. Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements.

Bien que les ressources Azure classiques puissent être découvertes via Azure Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager.

* [Guide pratique pour créer des requêtes avec Azure Resource Graph](../../governance/resource-graph/first-query-portal.md)

* [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

* [Comprendre le contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : les balises ne sont pas prises en charge pour Customer Lockbox.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des étiquettes, des groupes d’administration et des abonnements distincts, le cas échéant, pour organiser et suivre les ressources Azure. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

Appliquez également des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions de stratégie intégrées suivantes :
- Types de ressources non autorisés
- Types de ressources autorisés

* [Guide pratique pour créer des abonnements Azure supplémentaires](../../cost-management-billing/manage/create-subscription.md)

* [Guide pratique pour créer des groupes d’administration](../../governance/management-groups/create-management-group-portal.md)

* [Guide pratique pour créer et utiliser des étiquettes](../../azure-resource-manager/management/tag-resources.md)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4 : Tenir un inventaire des ressources Azure approuvées et titres des logiciels

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements.

Utilisez Azure Resource Graph pour interroger/découvrir des ressources dans leurs abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

* [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

* [Guide pratique pour créer des requêtes avec Azure Resource Graph](../../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements, en utilisant Azure Policy avec les définitions intégrées suivantes :
- Types de ressources non autorisés
- Types de ressources autorisés

* [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

* [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-implement-approved-application-list"></a>6.10 : Implémenter une liste d’applications approuvées

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="611-limit-users-ability-to-interact-with-azureresources-manager-via-scripts"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager via des scripts

**Conseils** : Configurez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

* [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Conseils** : non applicable. Customer Lockbox ne dispose pas de paramètres de sécurité configurables.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : non applicable. Customer Lockbox ne dispose pas de paramètres de sécurité configurables.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Conseils** : Non applicable. Customer Lockbox n’a pas de paramètres de sécurité configurables.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7 : Déployer les outils de gestion de configuration système

**Conseils** : Non applicable. Customer Lockbox n’a pas de paramètres de sécurité configurables.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la surveillance de la configuration automatique pour les services Azure

**Conseils** : Non applicable. Customer Lockbox n’a pas de paramètres de sécurité configurables.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : Non applicable. L’accès aux demandes de Customer Lockbox est limité au propriétaire de l’abonnement Azure qui héberge la ressource. Aucun mot de passe, secret ou clé ne sont requis pour accéder à Customer Lockbox en dehors de la connexion en tant que propriétaire de locataire.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils** : non applicable. Customer Lockbox n’utilise pas les identités managées.

* [Services Azure qui prennent en charge les identités managées](../../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

* [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Conseils** : Non applicable. Ces conseils concernent les ressources de calcul. Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge la solution Customer Lockbox.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Conseils** : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure tels que Customer Lockbox.

Il vous incombe de pré-analyser tous le contenu téléchargé vers des ressources Azure sans calcul. Microsoft ne peut pas accéder aux données client et ne peut donc pas effectuer d’analyses anti-programme malveillant du contenu client en votre nom.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul. Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure, mais il ne s’exécute pas sur du contenu client.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-recovery"></a>Récupération de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Récupération de données](../benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Conseils** : non applicable. Customer Lockbox ne stocke pas de données client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Conseils** : non applicable. Customer Lockbox ne stocke pas de données client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : non applicable. Customer Lockbox ne stocke pas de données client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : garantir la protection des sauvegardes et des clés gérées par le client

**Conseils** : non applicable. Customer Lockbox ne stocke pas de données client, et n’utilise pas de clés ou de mots de passe pour l’accès.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

* [Comment configurer des automatisations de workflow dans Azure Security Center](../../security-center/security-center-planning-and-operations-guide.md)

* [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez clairement les abonnements (par ex. production, non production) et créez un système d’attribution de noms pour identifier et classer les ressources Azure de façon claire.

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

* [Reportez-vous à la publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Conseils** : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

* [Comment définir le contact de sécurité d’Azure Security Center](../../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser la sentinelle des alertes.

* [Comment configurer l’exportation continue](../../security-center/continuous-export.md)

* [Comment envoyer des alertes à Azure Sentinel](../../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

* [Comment configurer l’automatisation des workflows et Logic Apps](../../security-center/workflow-automation.md)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : Tests d’intrusion et exercices Red Team](../benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1 : Procédez régulièrement à des tests d’intrusion de vos ressources Azure et veillez à résoudre tous les problèmes de sécurité critiques détectés dans un délai de 60 jours

**Conseils** : 

* [Suivez les règles d’engagement de Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies Microsoft](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

* [Vous trouverez ici plus d’informations sur la stratégie de Microsoft, sur l’exécution de Red Teaming et sur les tests d’intrusion de site actif dans l’infrastructure, les services et les applications cloud gérés par Microsoft.](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../benchmarks/security-baselines-overview.md)