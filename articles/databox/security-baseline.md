---
title: Base de référence de sécurité Azure pour Azure Data Box
description: Base de référence de sécurité Azure pour Azure Data Box
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 06/22/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 6c2a15ac8d0863539ca878a048940b19794e920d
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94842905"
---
# <a name="azure-security-baseline-for-azure-data-box"></a>Base de référence de sécurité Azure pour Azure Data Box

La Base de référence de sécurité Azure pour Azure Data Box contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez la [Vue d’ensemble des bases de référence de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Conseils** : non applicable. Votre Azure Data Box ne peut pas être associé à un réseau virtuel. Vous contrôlez le trafic du service Data Box vers le stockage hébergé par Azure via le portail Azure. Lorsque vous utilisez le service Data Box, les données sont transférées sur la dorsale principale Azure.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Conseils** : non applicable. Votre Azure Data Box ne peut pas être associé à un réseau virtuel. Vous contrôlez le trafic du service Data Box vers le stockage hébergé par Azure via le portail Azure. Lorsque vous utilisez le service Data Box, les données sont transférées sur la dorsale principale Azure.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Conseils** : non applicable. Votre Azure Data Box ne peut pas être associé à un réseau virtuel. Vous contrôlez le trafic du service Data Box vers le stockage hébergé par Azure via le portail Azure. Lorsque vous utilisez le service Data Box, les données sont transférées sur la dorsale principale Azure.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Conseils** : non applicable. Votre Azure Data Box ne peut pas être associé à un réseau virtuel. Vous contrôlez le trafic du service Data Box vers le stockage hébergé par Azure via le portail Azure. Lorsque vous utilisez le service Data Box, les données sont transférées sur la dorsale principale Azure.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions (IDS/IPS) basés sur le réseau

**Conseils** : Conseils : les points de terminaison qu’Azure Data Box utilise sont tous gérés par Microsoft. Vous êtes responsable de tous les contrôles supplémentaires que vous souhaitez déployer sur vos systèmes locaux.

* [Comprendre la sécurité d’Azure Data Box](./data-box-security.md)

* [Informations de port pour Azure Data Box](./data-box-system-requirements.md#port-requirements)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : non applicable. Votre Azure Data Box ne peut pas être associé à un réseau virtuel.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : non applicable. Votre Azure Data Box ne peut pas être associé à un réseau virtuel.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : non applicable. Votre Azure Data Box ne peut pas être associé à un réseau virtuel.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : non applicable. Votre Azure Data Box ne peut pas être associé à un réseau virtuel.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Contrôle de sécurité : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Conseils** : Microsoft conserve la source de temps utilisée pour les ressources Azure, par exemple, pour les horodatages dans les journaux. L’heure d’Azure Data Box peut dériver en l’absence de connexion au réseau qui peut accéder au serveur NTP sur le site du client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : à chaque étape de votre commande Data Box, vous pouvez effectuer plusieurs actions pour contrôler l’accès à la commande, auditer les événements, suivre la commande et interpréter les différents journaux générés.

* [Comprendre le suivi et la journalisation des événements pour votre Azure Data Box](./data-box-logs.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : à chaque étape de votre commande Data Box, vous pouvez effectuer plusieurs actions pour contrôler l’accès à la commande, auditer les événements, suivre la commande et interpréter les différents journaux générés.

* [Comprendre le suivi et la journalisation des événements pour votre Azure Data Box](./data-box-logs.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : non applicable. Cette recommandation a trait aux ressources de calcul. Data Box possède des journaux d’audit et un package de support contenant des journaux des événements de sécurité.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Non applicable

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : à chaque étape de votre commande Data Box, vous pouvez effectuer plusieurs actions pour contrôler l’accès à la commande, auditer les événements, suivre la commande et interpréter les différents journaux générés.

* [Comprendre le suivi et la journalisation des événements pour votre Azure Data Box](./data-box-logs.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils** : à chaque étape de votre commande Data Box, vous pouvez effectuer plusieurs actions pour contrôler l’accès à la commande, auditer les événements, suivre la commande et interpréter les différents journaux générés.

* [Comprendre le suivi et la journalisation des événements pour votre Azure Data Box](./data-box-logs.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Conseils** : non applicable. Azure Data Box ne traite et ne produit pas de journaux liés aux logiciels anti-programmes malveillants.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Conseils** : non applicable. Azure Data Box ne traite et ne produit pas de journaux liés au DNS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : conservez un inventaire des comptes d’utilisateur qui ont un accès administratif à votre Azure Data Box. Vous pouvez utiliser le volet de gestion des identités et des accès (IAM) dans le portail Azure de votre abonnement afin de configurer le contrôle d’accès en fonction du rôle d’Azure (Azure RBAC). Les rôles sont appliqués aux utilisateurs, aux groupes, aux principaux de service et aux identités managées dans Active Directory. Vous pouvez contrôler qui peut accéder à votre commande lors de la création de celle-ci pour la première fois. Configurez des rôles Azure d’étendues différentes pour contrôler l’accès à la commande Data Box. Un rôle Azure détermine le type d’accès accordé : lecture-écriture, lecture seule, lecture-écriture sur une partie des opérations.

* [Comprendre les rôles personnalisés](../role-based-access-control/custom-roles.md)

* [Comment configurer Azure RBAC pour les classeurs](../sentinel/quickstart-get-visibility.md)

* [Comprendre comment configurer le contrôle d’accès sur la commande](./data-box-logs.md#set-up-access-control-on-the-order)

**Supervision Azure Security Center** : Non

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : Azure AD n’intègre pas le concept des mots de passe par défaut. Selon le service, d’autres ressources Azure qui exigent un mot de passe forcent la création d’un mot de passe conforme à des exigences de complexité et d’une longueur minimale. Vous êtes responsable des applications tierces et des services de la place de marché susceptibles d’utiliser des mots de passe par défaut.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

En outre, pour vous aider à suivre les comptes d’administration dédiés, vous pouvez utiliser des recommandations d’Azure Security Center ou des stratégies Azure intégrées, telles que les suivantes :
- Plusieurs propriétaires doivent être affectés à votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

* [Utilisation d’Azure Security Center pour superviser l’identité et l’accès (préversion)](../security-center/security-center-identity-access.md)

* [Utilisation d’Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : non applicable. L’accès à votre commande Azure Data Box s’effectue via le portail Azure, et est réservé aux comptes ayant le rôle locataire de propriétaire ou contributeur.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : Non applicable

**Supervision Azure Security Center** : Non

**Responsabilité** : Non applicable

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils** : Utilisez une station de travail disposant d’un accès privilégié avec le service Azure AD Multi-Factor Authentication (MFA) activé pour vous connecter à vos commandes Azure Data Box et les configurer.

* [Stations de travail d’accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Planifier un déploiement informatique d’Azure AD Multi-Factor Authentication](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils** : non applicable. Azure Data Box n’a pas de compte administratif propre. Vous y accédez via le portail Azure. Vous pouvez cependant configurer votre abonnement Azure afin d’utiliser Azure Active Directory (AD) Privileged Identity Management pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.

Utilisez également les détections de risque Azure AD pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs.

* [Déploiement de Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

* [Présentation des détections de risques Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Utilisez des emplacements nommés avec accès conditionnel pour autoriser l’accès au Portail Azure uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

* [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Conseils** : le cas échéant, utilisez Azure Active Directory comme système central d’authentification et d’autorisation. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

* [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory (AD) fournit des journaux pour vous aider à découvrir les comptes obsolètes. De plus, utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seules les utilisateurs appropriés continuent de bénéficier d’un accès.

Pour l’appliance Data Box, cette solution n’est pas prise en charge en temps réel. Vous pouvez consulter les journaux à la fin du travail.

* [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml)

* [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Conseils** : le cas échéant, utilisez Azure Active Directory comme système central d’authentification et d’autorisation. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

Vous avez accès aux activités de connexion Azure AD et aux sources des journaux des événements à risque et des audits, ce qui vous permet de les intégrer à Azure Sentinel ou à un outil SIEM tiers.

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure AD et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes de journal souhaitées dans un espace de travail Log Analytics.

Les journaux du service Azure Data Box ne sont pas écrits dans l’espace de travail Log Analytics.

* [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

* [Procédure d’intégration d’Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Aide** : Pour les écarts du comportement de connexion au compte sur le plan de contrôle (le Portail Azure par exemple), utilisez Azure AD Identity Protection et les fonctionnalités de détections des risques pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. Vous pouvez également ingérer des données dans Azure Sentinel pour approfondir votre examen.

* [Guide pratique pour afficher une connexion risquée Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

* [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

* [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : Customer Lockbox n’est actuellement pas pris en charge pour Azure Data Box.

* [Liste des services pris en charge pour Customer Lockbox](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : non applicable pour Azure Data Box.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Azure Data Box sera approvisionné dans l’abonnement dans lequel résident les ressources auxquelles vous octroyez l’accès. Il n’existe aucun point de terminaison public à protéger ou à isoler. L’accès à Data Box est possible pour les utilisateurs disposant d’un accès de propriétaire ou de contributeur à l’abonnement.

Lors du chargement des données dans Azure, l’appliance Data Box et le service utilisé pour charger les données sont isolés.

* [Comment bien démarrer avec Azure Data Box](./data-box-quickstart-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : Microsoft gère l’infrastructure sous-jacente d’Azure Data Box et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client. Lorsque Data Box se trouve sur le site du client, suivez les meilleures pratiques pour vous assurer que les données sensibles transférées sont protégées.

* [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Microsoft gère l’infrastructure sous-jacente d’Azure Data Box et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client. Lorsque Data Box se trouve sur le site du client, suivez les meilleures pratiques pour vous assurer que les données sensibles transférées sont protégées.

* [Comprendre la migration des données dans Azure Data Box](./data-box-faq.md)

* [Vue d’ensemble de la sécurité de Data Box](./data-box-security.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Conseils** : actuellement indisponible. Les fonctionnalités d’identification, de classification et de prévention contre la perte de données ne sont pas encore disponibles pour Azure Data Box. Microsoft gère l’infrastructure sous-jacente d’Azure Data Box et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

* [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Non applicable

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : Vous disposez d’un accès propriétaire ou contributeur à l’abonnement pour créer une commande Data Box. Vous pouvez également définir des rôles Lecteur Data Box et Contributeur Data Box au niveau de la ressource.

* [Comprendre comment prendre en main Azure Data Box](./data-box-quickstart-portal.md)

* [Comprendre comment configurer le contrôle d’accès](./data-box-logs.md#set-up-access-control-on-the-order)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul. Microsoft gère l’infrastructure sous-jacente d’Azure Data Box et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

* [Protection des données client Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : Azure Data Box implémente le chiffrement AES 256 bits pour les données au repos.

Azure Data Box implémente le chiffrement AES 256 bits pour les données au repos. En outre, Azure Data Box protège la clé de déverrouillage de l’appareil (également appelée mot de passe de l’appareil) utilisée pour verrouiller l’appareil via une clé de chiffrement. Par défaut, la clé de déverrouillage de l’appareil pour un ordre Data Box est chiffrée avec une clé gérée par Microsoft. Pour un contrôle supplémentaire sur la clé de déverrouillage de l’appareil, vous pouvez également fournir une clé gérée par le client. Les clés gérées par le client doivent être créées et stockées dans Azure Key Vault.

* [Comprendre la protection des données Data Box](./data-box-security.md)

* [Utiliser des clés gérées par le client dans Azure Key Vault pour Azure Data Box](./data-box-customer-managed-encryption-key-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes quand des modifications sont apportées à Azure Data Box ainsi qu’à d’autres ressources critiques ou connexes.

* [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Azure Data Box.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : une fois Data Box expédié, il est installé avec les dernières mises à jour. Nous n’effectuons pas de mises à jour du champ.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="53-deploy-automated-patch-management-solution-for-third-party-software-titles"></a>5.3 : Déployer une solution de gestion automatisée des correctifs des logiciels tiers

**Conseils** : une fois Data Box expédié, il est installé avec les dernières mises à jour. Nous n’effectuons pas de mises à jour du champ.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils** : Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Azure Data Box.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Azure Data Box.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : non applicable. Il n’y a pas de ressources Data Box à découvrir.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : non applicable. Il n’y a pas de métadonnées de ressource pour Data Box.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : non applicable. Le service Data Box service veille à ce qu’aucune ressource Azure non autorisée ne soit utilisée.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Conseils** : non applicable. Il n’y en a pas au niveau de service Data Box.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : non applicable. Il n’y en a pas au niveau de service Data Box.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : non applicable. Il n’y en a pas au niveau de service Data Box.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : non applicable. Il n’y en a pas au niveau de service Data Box.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : non applicable. Il n’y en a pas au niveau de service Data Box.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : non applicable. Le service Data Box utilise uniquement des services Azure approuvés.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés

**Conseils** : non applicable. Le service Data Box utilise uniquement des titres de logiciels approuvés.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Configurez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

* [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : non applicable. Le service Data Box service ne prend pas en charge l’exécution de scripts.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : non applicable. Le service Data Box n’a pas d’applications web s’exécutant sur le service Azure App.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Conseils** : Azure Data Box est fourni avec les paramètres de sécurité conformes aux meilleures pratiques préconfigurés.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Azure Data Box est fourni avec les paramètres de sécurité conformes aux meilleures pratiques préconfigurés.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : Azure Data Box est fourni avec les paramètres de sécurité conformes aux meilleures pratiques préconfigurés pour les ressources, qu’il n’est pas possible de modifier.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Azure Data Box est fourni avec les paramètres de sécurité conformes aux meilleures pratiques préconfigurés pour les ressources, qu’il n’est pas possible de modifier.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Conseils** : Toutes les configurations de Data Box sont stockées en toute sécurité.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Toutes les images de système d’exploitation de Data Box sont stockées en toute sécurité.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Microsoft

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : non applicable. Il n’est pas possible de modifier les configurations d’Azure Data Box.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer des outils de gestion de la configuration pour les systèmes d'exploitation

**Conseils** : non applicable. Il n’est pas possible de modifier les configurations d’Azure Data Box.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Conseils** : non applicable. Il n’est pas possible de modifier les configurations d’Azure Data Box.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : non applicable. Il n’est pas possible de modifier les configurations d’Azure Data Box.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : Les clés gérées par le client doivent être créées et stockées dans Azure Key Vault.

* [Comment utiliser des clés gérées par le client dans Azure Key Vault pour Azure Data Box](./data-box-customer-managed-encryption-key-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils** : non applicable. Azure Data Box n’utilise pas d’identités managées.

* [Services Azure qui prennent en charge les identités managées](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Microsoft exécute un analyseur d’informations d’identification sur du code Data Box. En outre, Microsoft protège également les informations d’identification de manière sécurisée. Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

* [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : N/A

**Responsabilité** : Partagé

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Aide** : Non applicable. Ces conseils concernent les ressources de calcul. Le logiciel anti-programme malveillant de Microsoft est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure App Service), mais il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Conseils** : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure Customer Lockbox), mais ne s’exécute pas sur du contenu client.

Il vous incombe de pré-analyser tous contenu chargé vers des ressources Azure autres que de calcul. Microsoft ne peut pas accéder aux données client et ne peut donc pas effectuer d’analyses anti-programme malveillant du contenu client en votre nom.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul. Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure, mais il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-recovery"></a>Récupération de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Conseils** : non applicable. Le service Data Box ne nécessite pas de sauvegardes.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Conseils** : Veillez à sauvegarder toutes les données et la clé gérée par le client. Data Box n’effectue aucune sauvegarde.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : Vous devez vérifier que toutes vos données se trouvent dans le compte Stockage Azure avant de les supprimer localement.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés gérées par le client

**Conseils** : Assurez-vous que les sauvegardes ou les clés gérées par le client que vous avez sont protégées conformément aux meilleures pratiques.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

* [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

* [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

* [Tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de votre propre plan de réponse aux incidents](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez clairement les abonnements (par ex. production, non-production) à l’aide d’étiquettes et créez un système de nommage pour identifier et classer clairement les ressources Azure, en particulier celles qui traitent des données sensibles. Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

* [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md)

* [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Effectuez des exercices pour tester les capacités de réponse aux incidents de vos systèmes à intervalles réguliers, afin de protéger vos ressources Azure. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

* [Publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Instructions** : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

* [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Aide** : Exportez vos alertes et recommandations Azure Security Center en utilisant la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel.

* [Comment configurer l’exportation continue](../security-center/continuous-export.md)

* [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Aide** : Utilisez la fonctionnalité d’automatisation de workflow d’Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » aux alertes et aux recommandations de sécurité afin de protéger vos ressources Azure.

* [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Conseils** : Microsoft effectue des tests d’intrusion et une analyse des vulnérabilités sur les appareils Data Box. Vous pouvez effectuer vos propres tests de pénétration et analyse des vulnérabilités. Pour ce faire, vous devez suivre les règles d’engagement de Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft.

* [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

* [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)