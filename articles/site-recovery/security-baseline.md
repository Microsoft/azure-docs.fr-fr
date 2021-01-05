---
title: Ligne de base de la sécurité Azure pour Site Recovery
description: La ligne de base de la sécurité pour Site Recovery fournit de l’aide et des ressources pour l’implémentation des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/26/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 860bc9440547a4c62336ccc573859ec56b3400b5
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96533857"
---
# <a name="azure-security-baseline-for-site-recovery"></a>Ligne de base de la sécurité Azure pour Site Recovery

Cette ligne de base de la sécurité applique les conseils du [ Benchmark de sécurité Azure version 1.0](../security/benchmarks/overview-v1.md) au service Site Recovery. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Le contenu est regroupé sur la base des **contrôles de sécurité** définis par le benchmark de sécurité Azure et l’aide associée applicables au service Site Recovery. Les **contrôles** non applicables au service Site Recovery ont été exclus. 

Pour voir comment le service Site Recovery est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la ligne de base de sécurité de Site Recovery](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Conseils** : le service Microsoft Azure Site Recovery ne prend pas en charge le déploiement dans un réseau virtuel Azure. Configurez le service Site Recovery avec un point de terminaison privé Azure pour appliquer des communications sécurisées à votre réseau.

- [Prise en charge de liaison privée Azure Site Recovery](azure-to-azure-how-to-enable-replication-private-endpoints.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : le service Site Recovery prend en charge les étiquettes de service, qui permettent aux clients de n’ouvrir le trafic qu’à des services et ports spécifiques. Les clients doivent autoriser l’étiquette de service « AzureSiteRecovery » sur leur pare-feu ou groupe de sécurité réseau pour autoriser l’accès sortant au service Site Recovery.

- [Connectivité sortante à l’aide d’étiquettes de service](azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : Utilisez des étiquettes de ressource pour les groupes de sécurité réseau (NSG) et autres ressources liées à la sécurité réseau et au trafic. Pour les règles de groupe de sécurité réseau individuelles, utilisez le champ « Description » pour documenter les règles autorisant le trafic en direction et à partir d’un réseau. 

Incorporez n’importe laquelle des définitions Azure Policy intégrées en lien avec l’étiquetage, comme « Exiger une étiquette et sa valeur », pour vous assurer que toutes les ressources sont créées avec étiquettes, et être informé de l’existence de ressources non étiquetées. 

Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes. 

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md) 

- [Comment créer un réseau virtuel Azure](../virtual-network/quick-create-portal.md) 

- [Comment filtrer le trafic réseau avec les règles de groupes de sécurité réseau](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : surveillez les modifications apportées aux configurations de ressources réseau en lien avec le service Site Recovery à l’aide des journaux des activités Azure. Créez des alertes dans Azure Monitor pour être informé de la modification de ressources réseau critiques du service Site Recovery.

- [Afficher et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Créer, afficher et gérer des alertes de journal d’activité à l’aide d’Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : activez les paramètres de diagnostic de journal des activités Azure pour la journalisation d’audit, et envoyez les journaux à un espace de travail Log Analytics, un compte de stockage Azure ou un Event Hub Azure à des fins d’archivage.

Utilisez les données du journal des activités Azure pour déterminer « le quoi, le qui et le quand » de toutes les opérations d’écriture (PUT, POST, DELETE) effectuées sur vos ressources Azure.

Ingérez les journaux du service Site Recovery dans Azure Monitor afin d’agréger les données de sécurité générées. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, et utilisez des comptes de stockage pour le stockage à long terme ou l’archivage. Vous pouvez également activer et intégrer des données dans Azure Sentinel ou une tierce solution de gestion des informations de sécurité et des événements (Security Information and Event Management, SIEM).

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Superviser Site Recovery avec les journaux Azure Monitor](monitor-log-analytics.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : activez les paramètres de diagnostic de journal des activités Azure pour la journalisation d’audit, et envoyez les journaux à un espace de travail Log Analytics, un compte de stockage Azure ou un Event Hub Azure à des fins d’archivage. 

Utilisez les données du journal des activités Azure pour déterminer « le quoi, le qui et le quand » de toutes les opérations d’écriture (PUT, POST, DELETE) effectuées sur vos ressources Azure.

Ingérez les journaux du service Site Recovery avec Azure Monitor afin d’agréger les données de sécurité générées. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, et utilisez des comptes de stockage pour le stockage à long terme et l’archivage. Activez et intégrez des données à Azure Sentinel ou à une solution SIEM tierce.

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Superviser Site Recovery avec les journaux Azure Monitor](monitor-log-analytics.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : définissez une période de rétention des journaux pour les espaces de travail Log Analytics associés à vos coffres Recovery Services, en utilisant Azure Monitor conformément aux réglementations de conformité de votre organisation. 

- [Guide pratique pour définir les paramètres de conservation des journaux](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux à un espace de travail Log Analytics. 

Exécuter des requêtes dans Log Analytics pour rechercher des termes, identifier des tendances, ainsi qu’analyser des modèles et des insights sur les données de journal des activités collectées à partir de coffres Recovery Services.

- [Surveiller Site Recovery](site-recovery-monitor-and-troubleshoot.md)

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Collecte et analyse des journaux d’activité Azure dans l’espace de travail Log Analytics dans Azure Monitor](../azure-monitor/platform/activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils** : supervisez les machines répliquées par Azure Site Recovery à l’aide des journaux Azure Monitor et de Log Analytics. Utilisez Log Analytics dans Azure Monitor pour écrire et tester des requêtes de journal, ainsi que pour analyser de façon interactive les données de journal. Azure Monitor collecte les journaux des activités et des ressources, ainsi que d’autres données de surveillance. 

Visualisez et interrogez les résultats du journal, et configurez des alertes pour prendre des mesures basées sur des données surveillées. Configurez des alertes sur un espace de travail Log Analytics pour Azure Sentinel, car cela fournit une solution de réponse automatisée d’orchestration de sécurité (Security Orchestration Automated Response, SOAR). Cela permet de créer des solutions automatisées telles que des playbooks, et d’utiliser celles-ci pour corriger des problèmes de sécurité. Créez des alertes de journal personnalisées dans votre espace de travail Log Analytics à l’aide d’Azure Monitor. 

- [Surveiller Site Recovery](site-recovery-monitor-and-troubleshoot.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Créer, afficher et gérer des alertes de journal à l’aide d’Azure Monitor](../azure-monitor/platform/alerts-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : aucun rôle n’est attribué par défaut. Les rôles doivent être explicitement attribués en fonction des besoins de l’entreprise. Toutes les attributions de rôles peuvent être vérifiées à l’aide de l’interface de ligne de commande PowerShell ou d’Azure Active Directory (Azure AD) pour découvrir les comptes membres de groupes d’administration.

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?preserve-view=true&view=azureadps-2.0)

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?preserve-view=true&view=azureadps-2.0)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez les fonctionnalités de gestion des identités et des accès de Security Center pour superviser le nombre de comptes d’administration.

En outre, pour vous aider à suivre les comptes d’administration dédiés, suivez les recommandations d’Azure Security Center ou des stratégies Azure intégrées, par exemple : 

- Plusieurs propriétaires doivent être affectés à votre abonnement 
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement 

- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

Créez un processus pour effectuer le suivi des identités et du contrôle d’accès pour les comptes d’administration, et les examiner régulièrement.

- [Utilisation d’Azure Security Center pour surveiller l’identité et l’accès](../security-center/security-center-identity-access.md)

- [Utilisation d’Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : utilisez une inscription d’application Azure avec un principal de service afin de récupérer un jeton à utiliser pour interagir avec vos coffres Recovery Services via des appels d’API.

- [Appels d’API REST Azure](/rest/api/azure/#how-to-call-azure-rest-apis-with-postman)

- [Inscription de votre application cliente (principal du service) à l’aide d’Azure AD](/rest/api/azure/#register-your-client-application-with-azure-ad)

- [Informations sur l’API Azure Recovery Services](/rest/api/recoveryservices)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : activez une authentification multifacteur (MFA) Azure AD, et suivez les recommandations de gestion des identités et des accès du Security Center. 
- [Planifier un déploiement d'Azure AD Multi-Factor Authentication](../active-directory/authentication/howto-mfa-getstarted.md)

- [Surveiller l’identité et l’accès](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils** : utilisez une station de travail sécurisée et gérée par Azure, également appelée station de travail à accès privilégié (Privileged Access Workstation, PAW) avec une authentification multifacteur Azure pour accomplir des tâches d’administration, ainsi que des actions privilégiées sur des ressources Site Recovery.

- [Stations de travail d’accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Planifier un déploiement informatique d'Azure AD Multi-Factor Authentication](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils** : utilisez la technologie Privileged Identity Management (PIM) d’Azure AD pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.
Visualisez les alertes et rapports sur les comportements des utilisateurs à risque avec la fonctionnalité de détection de risque d’Azure AD.

- [Déploiement de Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

- [Présentation des détections de risques Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-only-from-approved-locations"></a>3.8 : Gérer les ressources Azure uniquement à partir d’emplacements approuvés

**Conseils** : utilisez des emplacements nommés avec accès conditionnel pour autoriser l’accès au portail Azure uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP, de régions ou de pays.
- [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Conseils** : utilisez Azure AD comme système central d’authentification et d’autorisation pour le service Site Recovery. Azure AD protège les données à l’aide d’un chiffrement fort des données au repos et en transit, et sale, hache et stocke de manière sécurisée les informations d’identification utilisateur. 

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Conseils** : utilisez les journaux Azure AD pour découvrir plus facilement les comptes obsolètes. 

Gérez efficacement les appartenances de groupe, l’accès aux applications d’entreprise et les attributions de rôles avec les révisions des identités et des accès d’Azure AD. 

Créez un processus pour réviser régulièrement les accès utilisateur afin de vous assurer que seuls les utilisateurs dont les accès ont été révisés continuent d’avoir accès. 

- [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml)

- [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Conseils** : utilisez Azure AD comme système central d’authentification et d’autorisation pour les ressources du service Site Recovery. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit, mais aussi en salant, en hachant et en stockant de manière sécurisée les informations d’identification des utilisateurs.

Vous avez accès aux sources des journaux d’activités de connexion, d’audit et d’événements à risque d’Azure AD, ce qui vous permet d’intégrer celles-ci avec Azure Sentinel ou tout SIEM ou outil de surveillance disponible sur la Place de marché Azure.

Simplifiez davantage ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure AD et en envoyant les journaux d’audit et de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes souhaitées dans un espace de travail Log Analytics.

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Procédure d’intégration d’Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Conseils** : utilisez Azure AD comme système central d’authentification et d’autorisation pour vos coffres Recovery Services. 

Utilisez les fonctionnalités de protection des identités d’Azure AD pour la détection de comportement de connexion au compte, ainsi que pour configurer des réponses automatiques aux actions suspectes détectées, en lien avec les identités des utilisateurs. Ingérez également des données dans Azure Sentinel en vue d’examen plus approfondi.

- [Guide pratique pour afficher une connexion risquée Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : implémentez des abonnements ou groupes d’administration distincts pour les coffres Recovery Services de développement, de test et de production. Séparez les ressources à l’aide d’un réseau virtuel ou d’un sous-réseau, étiquetés de manière appropriée, et sécurisés par un groupe de sécurité réseau (NSG) ou un Pare-feu Azure. 

Désactivez les machines virtuelles qui stockent ou traitent les données sensibles quand celles-ci ne sont pas utilisées. Implémentez une stratégie et des procédures pour rendre ce processus récurrent. 

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Vue d’ensemble de Site Recovery](site-recovery-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : utilisez une liaison ou un point de terminaison privés, des groupes de sécurité réseau et des étiquettes de service pour atténuer les opportunités d’exfiltration de données des machines virtuelles compatibles Site Recovery.

Microsoft gère la plateforme sous-jacente qu’utilise Site Recovery, traite tous les contenus des clients comme s’ils étaient sensibles, et protège les données client contre la perte et l’exposition. Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection robustes pour garantir la sécurité des données client dans Azure. 

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

- [Répliquer des machines virtuelles avec des points de terminaison privés Azure](azure-to-azure-how-to-enable-replication-private-endpoints.md)

- [Répliquer des machines virtuelles avec des étiquettes de service Azure Site Recovery](azure-to-azure-about-networking.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Site Recovery utilise un canal https sécurisé, chiffré à l’aide d’un algorithme Advanced Encryption Standard (AES 256), entre les serveurs de charge de travail Azure et les services Site Recovery hébergés derrière un coffre Recovery Services.

Les versions du protocole TLS actuelles prises en charge pour Site Recovery sont TLS 1.0, TLS 1.1 et TLS 1.2 dans les régions qui étaient actives à la fin de l’année 2019. TLS 1.2 est la seule version du protocole TLS prise en charge pour les nouvelles régions.

- [Compréhension du chiffrement en transit pour Azure Site Recovery](physical-azure-set-up-source.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Conseils** : les fonctionnalités d’identification, de classification et de protection contre la perte des données ne sont pas encore disponibles pour Site Recovery. 

Implémentez une solution tierce si nécessaire pour la conformité.

Microsoft gère la plateforme sous-jacente qu’utilise Site Recovery, traite tous les contenus des clients comme s’ils étaient sensibles, et protège les données client contre la perte et l’exposition. Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection robustes pour garantir la sécurité des données client dans Azure. 

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="46-use-azure-rbac-to-manage-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : utilisez un contrôle d’accès en fonction du rôle Azure (RBAC Azure) pour gérer l’accès aux données et ressources du service Site Recovery. 

Séparez les tâches de travail avec un RBAC Azure, et accordez un accès approprié à celles-ci. Utilisez les rôles Site Recovery intégrés pour contrôler les opérations de gestion du service Site Recovery.

- [Comment configurer Azure RBAC](../role-based-access-control/role-assignments-portal.md)

- [Utiliser un contrôle d’accès en fonction du rôle pour gérer le service Azure Site Recovery](site-recovery-role-based-linked-access-control.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : activez un double chiffrement avec des clés gérées par la plateforme et par le client. Cette fonctionnalité est disponible dans le service Site Recovery. 

Celui-ci prend en charge le chiffrement au repos des données. Pour les charges de travail IaaS Azure, les données sont chiffrées au repos à l’aide de la fonctionnalité Storage Service Encryption (SSE). 

Seul le client a accès à la clé de chiffrement lors de l’utilisation d’un coffre Recovery Services chiffré avec une clé gérée par le client. Microsoft ne conserve jamais de copie, n’a pas accès à la clé et ne déchiffre à aucun moment les données transférées de l’emplacement principal vers l’emplacement de récupération d’urgence. 

- [Prise en charge des clés gérées par le client pour le service Azure Site Recovery](azure-to-azure-how-to-enable-replication-cmk-disks.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : utilisez Azure Monitor avec les journaux des activités Azure pour créer des alertes lorsque des modifications sont apportées à des ressources critiques. Ces ressources peuvent inclure des instances de production de coffres Recovery Services, des ressources du service Site Recovery et des ressources associées.
- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : utilisez le service Azure Resource Graph pour interroger ou découvrir toutes les ressources, dont les coffres Recovery Services, au sein de vos abonnements. Vérifiez la disponibilité des autorisations de lecture appropriées dans votre locataire, et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?preserve-view=true&view=azps-4.8.0)

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : appliquez aux coffres Recovery Services ainsi qu’à d’autres ressources associées des étiquettes que le Site Recovery utilise avec les métadonnées, afin de les organiser logiquement dans une taxonomie.
- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : utilisez un étiquetage, des groupes d’administration et des abonnements distincts, selon le besoin, pour organiser et suivre les ressources (coffres) Recovery Services et d’autres ressources associées. 

En outre, utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients à l’aide des définitions de stratégie intégrées suivantes : 

- Types de ressources non autorisés
- Types de ressources autorisés

Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4 : Dresser et tenir un inventaire des ressources Azure approuvées

**Conseils** : dressez un inventaire des ressources Azure approuvées, ainsi que des logiciels approuvés pour les ressources de calcul en fonction des exigences organisationnelles du client.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients selon les définitions de stratégies intégrées suivantes : 

- Types de ressources non autorisés 
- Types de ressources autorisés

Utilisez le service Azure Resource Graph pour interroger et découvrir les ressources dans les abonnements.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients selon les définitions de stratégies intégrées suivantes :

- Types de ressources non autorisés 
- Types de ressources autorisés

Il est important de comprendre comment créer et gérer des stratégies dans Azure pour rester conforme aux normes de votre entreprise et aux contrats de niveau de service.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ». Cela peut empêcher la création et la modification de ressources dans un environnement de haute sécurité.

- [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Définissez et implémentez des configurations de sécurité standard pour votre coffre Recovery Services à l’aide d’Azure Policy. 

Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.RecoveryServices » pour créer des stratégies personnalisées afin d’auditer ou d’appliquer la configuration des ressources de coffre Recovery Services du service Site Recovery.
- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?preserve-view=true&view=azps-4.8.0)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : Utilisez les effets Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.
- [Présentation des effets d’Azure Policy](../governance/policy/concepts/effects.md)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Conseils** : choisissez des Azure Repos pour stocker et gérer en toute sécurité votre code si vous utilisez des définitions Azure Policy personnalisées pour vos coffres Recovery Services et les ressources associées.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?preserve-view=true&view=azure-devops)

- [Documentation Azure Repos](/azure/devops/repos/?preserve-view=true&view=azure-devops)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : Utilisez les définitions Azure Policy intégrées ainsi que des alias Azure Policy dans l’espace de noms « Microsoft.RecoveryServices » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. 

En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Conseils** : Utilisez les définitions Azure Policy intégrées ainsi que des alias Azure Policy dans l’espace de noms « Microsoft.RecoveryServices » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. 

Utilisez les effets Azure Policy [auditer], [refuser] et [déployer s’il n’existe pas] afin d’appliquer automatiquement des configurations pour vos ressources Azure.
- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : le client doit gérer les secrets du service Site Recovery intégrés avec Azure Key Vault, tout en permettant la récupération d’urgence des machines virtuelles compatibles Azure Disk Encryption. 

- [Comment créer un coffre de clés](../key-vault/general/quick-create-portal.md)

- [Comment s’authentifier auprès du coffre de clés](../key-vault/general/authentication.md)

- [Comment attribuer une stratégie d’accès au coffre de clés](../key-vault/general/assign-access-policy-portal.md)

- [Comment activer la récupération d’urgence pour les machines virtuelles compatibles Azure Disk Encryption à l’aide du service Site Recovery](azure-to-azure-how-to-enable-replication-ade-vms.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils** : le service Site Recovery prend en charge l’identité gérée par le système uniquement quand un client peut activer celle-ci sur un coffre Recovery Services. La même méthodologie s’applique aux ressources utilisées dans l’offre de récupération d’urgence pour définir la limite d’accès. 

Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure AD. 

Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Key Vault, sans informations d’identification dans votre code.

- [Intégration aux identités managées Azure](../azure-app-configuration/howto-integrate-azure-managed-service-identity.md?tabs=core2x)

- [Comment activer l’identité gérée par le système sur un coffre Recovery Services](azure-to-azure-how-to-enable-replication-private-endpoints.md#enable-the-managed-identity-for-the-vault)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Conseils** : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Site Recovery), mais il ne s’exécute pas sur votre contenu. Pré-analysez les fichiers chargés sur des ressources Azure non liées au calcul, comme App Service, Data Lake Storage et Stockage Blob.

Utilisez la détection des menaces de Security Center pour les services de données afin de détecter les programmes malveillants chargés sur les comptes de stockage.

- [Présentation de Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles](../security/fundamentals/antimalware.md)

- [Présentation de la détection des menaces pour les services de données d’Azure Security Center](../security-center/azure-defender.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-recovery"></a>Récupération des données

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés managées par le client

**Conseils** : Site Recovery utilise en interne un compte de stockage Azure pour conserver l’état de la solution de récupération d’urgence, tel que configuré par les clients sur leurs charges de travail.

Toutes les ressources de stockage qu’utilisent les services Site Recovery comportent des métadonnées avec une configuration de type : Stockage géo-redondant avec accès en lecture (RA-GRS). Les comptes de stockage de type supérieur à GRS (comme RAGRS, RAG-ZRS) répliquent vos données dans une région secondaire (à des centaines de kilomètres de l’emplacement principal des données sources) pour continuer à servir la récupération d’urgence pour les clients en cas de panne.

Cela est hors de portée du client et l’équipe du service Site Recovery s’en charge en interne. Le client peut sauvegarder les clés du Key Vault dans Azure.

- [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](/powershell/module/az.keyvault/backup-azkeyvaultkey?preserve-view=true&view=azps-4.8.0)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : testez régulièrement la restauration des clés gérées par le client sauvegardées.

- [Guide pratique pour restaurer des clés de coffre de clés dans Azure](/powershell/module/az.keyvault/restore-azkeyvaultkey?preserve-view=true&view=azps-4.8.0)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés managées par le client

**Conseils** : les données sont chiffrées au repos à l’aide de la fonctionnalité Storage Service Encryption (SSE) avec des machines virtuelles basées sur l’infrastructure en tant que service (Infrastructure as a Service, IaaS) d’Azure. Activez la suppression réversible dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.

- [Activation de la suppression réversible dans Key Vault](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. 

Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion ou d’administration des incidents, de leur détection à leur examen a posteriori.

- [Comment configurer des automatisations de workflow dans Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process)

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process)

- [Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : hiérarchisez les alertes à examiner en premier, en fonction de la gravité qui leur est attribuée dans le Security Center. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

Marquez clairement les abonnements (par exemple, production, non-production) et créez un système de nommage pour identifier et classer les ressources Azure de façon claire.

- [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md) 

- [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifier les points faibles et les lacunes, et révisez le plan en fonction des besoins

- [Consultez la publication du NIST, Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Aide** : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé. 

Créez un processus pour examiner les incidents a posteriori pour vous assurer que les problèmes sont résolus.

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Aide** : Exportez vos alertes et recommandations de Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. 

Utilisez le connecteur de données du Security Center pour diffuser en continu les alertes vers Azure Sentinel en fonction des besoins.
- [Comment configurer l’exportation continue](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans le Centre de sécurité pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.
- [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de Microsoft pour garantir que vos tests d’intrusion sont conformes aux stratégies de Microsoft : https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

- [Vous trouverez ici plus d’informations sur la stratégie de Microsoft, sur l’exécution de Red Teaming et sur les tests d’intrusion de site actif dans l’infrastructure, les services et les applications cloud gérés par Microsoft.](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Vue d’ensemble d’Azure Security Benchmark V2](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)