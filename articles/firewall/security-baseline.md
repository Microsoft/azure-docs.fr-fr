---
title: Base de référence de sécurité Azure pour Pare-feu Azure
description: La base de référence de sécurité Pare-feu Azure fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: firewall
ms.topic: conceptual
ms.date: 08/05/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 764b10d1af2a037c4d5285ae42669ea2345779e0
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94660506"
---
# <a name="azure-security-baseline-for-azure-firewall"></a>Base de référence de sécurité Azure pour Pare-feu Azure

Cette base de référence de sécurité applique les conseils du [benchmark de sécurité Azure](../security/benchmarks/overview.md) à Pare-feu Azure. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Le contenu est regroupé par les **contrôles de sécurité** définis par le benchmark de sécurité Azure et les conseils associés applicables à Pare-feu Azure. Les **contrôles** non applicables à Pare-feu Azure ont été exclus. Pour voir comment Pare-feu Azure est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité Pare-feu Azure](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des interfaces réseau

**Aide** : Pare-feu Azure est intégré à Azure Monitor pour la journalisation du trafic traité par le pare-feu.

Par ailleurs, Utilisez Azure Security Center et suivez les recommandations de protection du réseau pour sécuriser vos ressources réseau associées à Pare-feu Azure.

- [Présentation de la sécurité réseau assurée par Azure Security Center](../security-center/security-center-network-recommendations.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications avec des adresses IP connues comme étant malveillantes

**Aide** : Activez le filtrage basé sur Threat Intelligence pour donner l’alerte et rejeter le trafic depuis ou vers des adresses IP et des domaines malveillants connus. Le filtrage basé sur Threat Intelligence peut être activé pour votre pare-feu pour donner l’alerte et rejeter le trafic depuis ou vers des adresses IP et des domaines malveillants connus.

- [Fonctionnalité de filtrage basé sur la Threat Intelligence du Pare-feu Azure](threat-intel.md)

- [Présentation de la fonctionnalité Threat Intelligence intégrée à Azure Security Center](../security-center/azure-defender.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Aide** : Dans Pare-feu Azure, une balise de service représente un groupe de préfixes d’adresses IP qui permet de simplifier la création de règles de sécurité.

Les balises de service Pare-feu Azure peuvent être utilisées dans le champ de destination des règles réseau et définissent des contrôles d’accès réseau sur Pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité.

En outre, les balises définies par le client, par exemple les groupes IP, sont également prises en charge et peuvent être utilisées dans une règle de réseau ou une règle d’application. Les balises FQDN dans les règles d’application sont également prises en charge pour autoriser le trafic sortant requis via votre pare-feu.

Notez que vous ne peut pas créer votre propre balise de service, ni spécifier les adresses IP incluses dans une balise. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

 

- [Balises de service de Pare-feu Azure](service-tags.md)

- [Balises de service disponibles](../virtual-network/service-tags-overview.md#available-service-tags)

- [Groupes IP dans Pare-feu Azure](ip-groups.md)

- [Présentation des balises FQDN](fqdn-tags.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : La stratégie Azure n’est pas encore entièrement prise en charge pour Pare-feu Azure. Azure Firewall Manager peut être utilisé pour garantir la standardisation des configurations de sécurité.

Vous pouvez également utiliser Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les principaux artefacts d’environnement (par exemple, les modèles Resource Manager, les contrôles Azure RBAC et les stratégies) dans une même définition de blueprint. Vous pouvez appliquer le blueprint aux nouveaux abonnements et ajuster le contrôle et la gestion grâce au contrôle de version.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Exemples Azure Policy pour le réseau](../governance/policy/samples/built-in-policies.md#network)

- [Guide pratique pour créer un blueprint Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : Utilisez le journal d’activité Azure pour effectuer le monitoring des configurations de ressources et détecter les changements apportés à vos ressources Pare-feu Azure. Créez des alertes dans Azure Monitor, qui se déclenchent en cas de modification des ressources critiques.

- [Surveiller les journaux et les indicateurs de performance du pare-feu Azure](./firewall-diagnostics.md)

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md) 

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Aide** : Microsoft gère les sources de temps pour les ressources Azure pour Pare-feu Azure. Les clients doivent créer une règle de réseau pour autoriser cet accès ou pour un serveur de temps que vous utilisez dans leur environnement.

- [Accès au serveur NTP](protect-windows-virtual-desktop.md#additional-considerations)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Vous pouvez activer des données de journal et les intégrer à Azure Sentinel ou un tiers SIEM pour la gestion centralisée de différents journaux de sécurité.

Les journaux d’activité peuvent être utilisés pour auditer des opérations sur Pare-feu Azure et surveiller des actions sur les ressources. Le journal d’activité contient toutes les opérations d’écriture (PUT, POST, DELETE) de vos ressources, à l’exception des opérations de lecture (GET). Les journaux d’activité peuvent être utilisés pour rechercher une erreur lors de la résolution de problèmes ou pour surveiller la manière dont un utilisateur de votre organisation a modifié une ressource.

Pare-feu Azure fournit également les journaux de diagnostic suivants pour obtenir des informations sur les applications clientes et les règles de réseau.

Journal de règles d’application : Chaque nouvelle connexion qui correspond à l’une de vos règles d’application configurées entraîne un journal pour la connexion acceptée/refusée.

Journal de règles de réseau : Chaque nouvelle connexion qui correspond à l’une de vos règles de réseau configurées entraîne un journal pour la connexion acceptée/refusée.

Remarque : Ces deux journaux peuvent être enregistrés dans un compte de stockage, transmis en continu au service Event Hubs et/ou envoyés vers les journaux Azure Monitor uniquement si vous les avez activés pour chaque Pare-feu Azure dans un environnement.

- [Journaux d’activité de Pare-feu Azure](logs-and-metrics.md)

Liste des actions de ressource dans les journaux d’activité : Opérations du fournisseur de ressources Azure Resource Manager

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Guide pratique pour bien démarrer avec Azure Monitor et l’intégration d’une solution SIEM tierce](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Aide** : Les journaux d’activité peuvent être utilisés pour auditer des opérations sur Pare-feu Azure et surveiller les actions sur les ressources. Le journal d’activité contient toutes les opérations d’écriture (PUT, POST, DELETE) de vos ressources Azure, à l’exception des opérations de lecture (GET). Pare-feu Azure fournit également les journaux de diagnostic suivants pour obtenir des informations sur les applications clientes et les règles de réseau. 

Journal de règles d’application : Chaque nouvelle connexion qui correspond à l’une de vos règles d’application configurées entraîne un journal pour la connexion acceptée/refusée.

Journal de règles de réseau : Chaque nouvelle connexion qui correspond à l’une de vos règles de réseau configurées entraîne un journal pour la connexion acceptée/refusée.

Notez que les deux journaux peuvent être enregistrés dans un compte de stockage, transmis en continu au service Event Hubs et/ou envoyés vers les journaux Azure Monitor uniquement si vous les avez activés pour chaque Pare-feu Azure dans un environnement.

- [Journaux d’activité de Pare-feu Azure](logs-and-metrics.md)

- [Liste des actions de ressource dans les journaux d’activité](../role-based-access-control/resource-provider-operations.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Aide** : La période de rétention de l’espace de travail Log Analytics peut être définie dans Azure Monitor en fonction des réglementations de conformité de votre organisation. La rétention des données peut être configurée entre 30 et 730 jours (2 ans) pour tous les espaces de travail en fonction du niveau tarifaire choisi.

Il existe 3 options pour stocker la rétention du stockage des journaux :

les comptes de stockage conviennent parfaitement aux journaux d’activité quand ils sont stockés pour une durée plus longue et consultés quand cela est nécessaire.

les hubs d’événements constituent une excellente solution pour l’intégration à d’autres outils SEIM (Security Information and Event Management) afin de recevoir des alertes sur vos ressources.

Les journaux d’activité Azure Monitor conviennent parfaitement pour la supervision en temps réel générale de votre application ou la recherche de tendances.

- [Journaux et métriques du pare-feu Azure](logs-and-metrics.md)

- [Modification de la période de conservation des données dans Log Analytics](../azure-monitor/platform/manage-cost-storage.md)

- [Guide pratique pour configurer la stratégie de conservation des journaux de compte de Stockage Azure](../storage/common/storage-monitor-storage-account.md#configure-logging)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : Pare-feu Azure est intégré à Azure Monitor pour la consultation et l’analyse des journaux d’activité de Pare-feu. Les journaux d’activité peuvent être envoyés à Log Analytics, Stockage Azure ou Event Hubs. Ils peuvent être analysés dans Log Analytics ou par d’autres outils comme Excel et Power BI. Il existe différents types de journaux Pare-feu Azure.

Les journaux d’activité peuvent être utilisés pour auditer des opérations sur Pare-feu Azure et surveiller des actions sur les ressources. Le journal d’activité contient toutes les opérations d’écriture (PUT, POST, DELETE) des ressources, à l’exception des opérations de lecture (GET). Les journaux d’activité peuvent être utilisés pour rechercher une erreur lors de la résolution de problèmes ou pour surveiller la manière dont un utilisateur de votre organisation a modifié une ressource.

Pare-feu Azure fournit également des journaux de diagnostic pour obtenir des informations sur les applications clientes et les règles de réseau.

Des journaux de règles d’application sont créés quand chaque nouvelle connexion qui correspond à l’une de vos règles d’application configurées entraîne un journal pour la connexion acceptée/refusée. 

Des journaux de règles de réseau sont créés quand chaque nouvelle connexion qui correspond à l’une de vos règles de réseau configurées entraîne un journal pour la connexion acceptée/refusée

Notez que les deux journaux peuvent être enregistrés dans un compte de stockage, transmis en continu au service Event Hubs et/ou envoyés vers les journaux Azure Monitor uniquement si vous les avez activés pour chaque Pare-feu Azure dans un environnement.

Les journaux d’activité Azure Monitor conviennent parfaitement pour la supervision en temps réel générale de votre application ou la recherche de tendances.

- [Journaux et métriques du pare-feu Azure](logs-and-metrics.md)

- [Journaux de diagnostic](logs-and-metrics.md#diagnostic-logs)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Aide** : Utilisez Azure Security Center avec un espace de travail Log Analytics pour superviser les activités anormales détectées dans les journaux de sécurité et les événements et générer des alertes s’y rapportant. 

Vous pouvez également activer et intégrer les données dans Azure Sentinel. 

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Guide pratique pour gérer les alertes dans Azure Security Center](../security-center/security-center-managing-and-responding-alerts.md)

- [Guide pratique pour générer une alerte sur des données de journal Log Analytics](../azure-monitor/learn/tutorial-response.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Azure AD comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent être interrogés. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration.

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

Vous pouvez également activer JIT (Juste-à-temps)/JEA (Just-Enough-Access) à l’aide de rôles Azure AD Privileged Identity Management pour les services Microsoft et d’Azure Resource Manager.

- [En savoir plus sur Privileged Identity Management](../active-directory/privileged-identity-management/index.yml)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4 : Utiliser l’authentification unique (SSO) Azure Active Directory

**Conseils** : Dans la mesure du possible, utilisez l’authentification unique (SSO) d’Azure Active Directory au lieu de configurer des informations d’identification autonomes individuelles par service. Suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

- [Présentation de l’authentification SSO avec Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Aide** : Activez l’authentification multifacteur (MFA) Azure Active Directory et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Aide** : Utilisez des PAW (stations de travail avec accès privilégié) avec l’authentification multifacteur (MFA) configurée pour se connecter aux ressources Pare-feu Azure et ressources associées et les configurer. 

- [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations) 

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils** : Utilisez les rapports de sécurité Azure Active Directory pour générer des journaux et des alertes lorsque des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement. Utiliser Azure Security Center pour superviser les activités liées aux identités et aux accès. 

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../active-directory/identity-protection/overview-identity-protection.md) 

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Aide** : Utilisez des emplacements nommés à accès conditionnel pour autoriser l’accès uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions. 

- [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Utiliser Azure Active Directory (Azure AD) comme système d’authentification et d’autorisation central. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur. 

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Conseils** : Azure AD fournit des journaux pour vous aider à découvrir les comptes obsolètes. De plus, utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seules les utilisateurs appropriés continuent de bénéficier d’un accès. 

- [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml)

- [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Conseils** : Vous avez accès aux activités de connexion Azure AD, aux sources des journaux d’événements à risque et d’audit, ce qui vous permet de les intégrer à un outil SIEM/de supervision. 

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure Active Directory et en envoyant les journaux d’audit et les journaux de connexion dans un espace de travail Log Analytics. Vous pouvez configurer les alertes souhaitées dans un espace de travail Log Analytics. 

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12 : Alerter en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez les fonctionnalités d’Azure AD Identity Risk and Identity Protection pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. Vous pouvez également ingérer des données dans Azure Sentinel pour approfondir votre examen. 

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Aide** : Utilisez des étiquettes pour faciliter le suivi des ressources Pare-feu Azure et ressources associées qui stockent ou traitent des informations sensibles. 

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez une isolation en utilisant des abonnements et groupes d’administration distincts selon les domaines de sécurité (par exemple, le type d’environnement et le niveau de confidentialité des données). Vous pouvez limiter le niveau d’accès à vos ressources Pare-feu Azure demandées par vos applications et environnements d’entreprise. Vous pouvez contrôler l’accès aux ressources Azure via le contrôle d’accès en fonction du rôle d’Azure. 

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Aide** : Exploitez une solution tierce issue de la Place de marché Azure sur les périmètres du réseau, qui surveille et bloque les transferts non autorisés d’informations sensibles tout en alertant les professionnels de la sécurité de l’information. 

Pour la plateforme sous-jacente, gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et assure une protection contre la perte et l’exposition de données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes. 

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Instructions** : Chiffrer toutes les informations sensibles en transit. Assurez-vous que les clients qui se connectent à vos ressources Pare-feu Azure et ressources associées peuvent négocier TLS 1.2 ou une version ultérieure. 

Suivez les recommandations d’Azure Security Center relatives au chiffrement au repos et au chiffrement en transit, le cas échéant. 

- [Présentation du chiffrement en transit avec Azure](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Aide** : Utilisez un outil de découverte actif tiers pour identifier toutes les informations sensibles stockées dans les ressources Azure à l’aide des ressources Pare-feu Azure et ressources associées, et mettez à jour l’inventaire des informations sensibles de l’organisation.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : Utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour contrôler l’accès à Pare-feu Azure et aux ressources associées.

- [Comment configurer Azure RBAC](../role-based-access-control/role-assignments-portal.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide** : Utilisez le chiffrement au repos sur toutes les ressources Azure à l’aide des ressources Pare-feu Azure et ressources associées. Microsoft recommande d’autoriser Azure à gérer vos clés de chiffrement, mais il existe une option vous permettant de gérer vos propres clés dans certains cas. 

- [Présentation du chiffrement au repos dans Azure](../security/fundamentals/encryption-atrest.md)

- [Guide pratique pour configurer des clés de chiffrement gérées par le client](../storage/common/customer-managed-keys-configure-key-vault.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor avec le journal des activités Azure pour créer des alertes lorsque des modifications sont apportées à Pare-feu Azure.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Aide** : Appliquez des étiquettes aux ressources Pare-feu Azure et ressources associées en ajoutant des métadonnées pour les organiser logiquement dans une taxonomie. 

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des balises, des groupes d’administration, voire des abonnements distincts, pour organiser et suivre les ressources Pare-feu Azure et associées. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Aide** : Créez un inventaire des ressources Pare-feu Azure approuvées, notamment de la configuration, en fonction des besoins de votre organisation.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Aide** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements.

Utilisez Azure Resource Graph pour interroger/découvrir des ressources Pare-feu Azure dans leurs abonnements. Vérifiez que toutes les ressources Pare-feu Azure et ressources associées présentes dans l’environnement sont approuvées.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Aide** : Implémentez votre propre processus de suppression des ressources Pare-feu Azure et ressources associées non autorisées. Vous pouvez également utiliser une solution tierce pour identifier les ressources Pare-feu Azure et ressources associées non autorisées

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Aide** : Utilisez Azure Policy pour limiter les services que vous pouvez approvisionner dans votre environnement.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ». 

- [Guide pratique pour configurer l’accès conditionnel de façon à bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Aide** : Les applications qui peuvent être nécessaires pour les opérations de l’entreprise, ou les environnements avec des profils de risque différents pour l’organisation, doivent être isolées et séparées à l’aide d’instances Pare-feu Azure distinctes.

- [Déployer et configurer un pare-feu Azure à l’aide du portail Azure](deploy-cli.md)

- [Guide pratique pour créer un réseau virtuel](../virtual-network/quick-create-portal.md)

- [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Azure Resource Manager a la possibilité d’exporter le modèle au format JSON (JavaScript Object Notation), qui doit être examiné pour vérifier que les configurations répondent/dépassent les exigences de sécurité de votre organisation.

Vous pouvez aussi utiliser les recommandations d’Azure Security Center comme base de référence de configuration sécurisée pour vos ressources Azure.

La stratégie Azure n’est pas entièrement prise en charge pour le moment. 

- [Exportation monoressource ou multiressource vers un modèle sur le portail Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Recommandations de sécurité - Guide de référence](../security-center/recommendations-reference.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Pare-feu Azure et ressources associées.  En outre, vous pouvez utiliser des modèles Azure Resource Manager pour appliquer la configuration de sécurité des ressources Pare-feu Azure et ressources associées, requise par votre organisation.

- [Comprendre les effets d'Azure Policy](../governance/policy/concepts/effects.md)

- [Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

- [Présentation des modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Utilisez Azure DevOps pour stocker et gérer de manière sécurisée votre code, comme les stratégies Azure personnalisées et les modèles Azure Resource Manager. Pour accéder aux ressources que vous gérez dans Azure DevOps, vous pouvez accorder ou refuser des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps, ou à Active Directory s’il est intégré à TFS.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [À propos des autorisations et des groupes dans Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Aide** : Définissez et implémentez des configurations de sécurité standard pour les ressources Pare-feu Azure et ressources associées à l’aide d’Azure Policy. Utilisez des alias Azure Policy pour créer des stratégies personnalisées afin d’auditer ou d’appliquer la configuration réseau de vos ressources Pare-feu Azure. Vous pouvez également utiliser des définitions de stratégie intégrées en lien avec vos ressources spécifiques.  

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Utiliser des alias](../governance/policy/concepts/definition-structure.md#aliases)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Aide** : Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure AD. Les identités gérées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD pour Azure Resource Manager et qui peut être utilisé avec l’API/le portail Azure/l’interface CLI/PowerShell.

- [Configurer des identités managées](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault. 

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-recovery"></a>Récupération de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : Garantir des sauvegardes automatiques régulières

**Aide** : Utilisez Azure Resource Manager pour exporter les ressources Pare-feu Azure et ressources associées dans un modèle JavaScript Object Notation (JSON) qui peut être utilisé comme sauvegarde pour Pare-feu Azure et les configurations associées.  Vous pouvez également exporter la configuration Pare-feu Azure à l’aide de la fonctionnalité Exporter le modèle de Pare-feu Azure à partir de portail Azure.  Utilisez Azure Automation pour exécuter automatiquement les scripts de sauvegarde.

- [Exportation monoressource ou multiressource vers un modèle sur le portail Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Déployer Pare-feu Azure à l’aide d’un modèle](deploy-template.md)

- [Référence de modèle des pare-feu Azure Microsoft Network](/azure/templates/microsoft.network/azurefirewalls)

- [À propos d’Azure Automation](../automation/automation-intro.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés managées par le client

**Aide** : Utilisez Azure Resource Manager pour exporter les ressources Pare-feu Azure et ressources associées dans un modèle JavaScript Object Notation (JSON) qui peut être utilisé comme sauvegarde pour Pare-feu Azure et les configurations associées.  Vous pouvez également exporter la configuration Pare-feu Azure à l’aide de la fonctionnalité Exporter le modèle de Pare-feu Azure à partir de portail Azure.

- [Déployer Pare-feu Azure à l’aide d’un modèle](deploy-template.md)

- [Référence de modèle des pare-feu Azure Microsoft Network](/azure/templates/microsoft.network/azurefirewalls)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Aide** : Assurez-vous que la restauration s’effectue régulièrement à l’aide des fichiers de modèle Azure Resource Manager sauvegardés.  

- [Déployer Pare-feu Azure à l’aide d’un modèle](deploy-template.md)

- [Référence de modèle des pare-feu Azure Microsoft Network](/azure/templates/microsoft.network/azurefirewalls)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés managées par le client

**Aide** : Utilisez Azure DevOps pour stocker et gérer de manière sécurisée votre code, comme les stratégies Azure personnalisées et les modèles Azure Resource Manager. Pour protéger les ressources que vous gérez dans Azure DevOps, vous pouvez octroyer ou refuser des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps ou à Active Directory s’il est intégré à TFS.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [À propos des autorisations et des groupes dans Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé. 

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

- [Tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de votre propre plan de réponse aux incidents](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte. 

En outre, marquez clairement les abonnements (par ex. production, non-production) à l’aide d’étiquettes et créez un système de nommage pour identifier et classer clairement les ressources Azure, en particulier celles qui traitent des données sensibles.  Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit. 

- [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md) 

- [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Effectuez des exercices pour tester les capacités de réponse aux incidents de vos systèmes à intervalles réguliers, afin de protéger vos ressources Azure. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins. 

- [Publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Instructions** : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus. 

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Aide** : Exportez vos alertes et recommandations Azure Security Center en utilisant la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure.

L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel.

- [Comment configurer l’exportation continue](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Aide** : Utilisez la fonctionnalité d’automatisation de workflow d’Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » aux alertes et aux recommandations de sécurité afin de protéger vos ressources Azure. 

- [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de Microsoft pour que vos tests d’intrusion soient conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft. 

- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)