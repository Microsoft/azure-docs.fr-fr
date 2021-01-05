---
title: Base de référence de la sécurité pour ASecurity Center
description: La base de référence de sécurité Security Center fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: security-center
ms.topic: conceptual
ms.date: 08/05/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 3b648168cd80920b8042f1edeacca893c21b98e2
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498946"
---
# <a name="azure-security-baseline-for-security-center"></a>Base de référence de la sécurité pour ASecurity Center

Cette base de référence de sécurité applique les conseils du [benchmark de sécurité Azure](../security/benchmarks/overview.md) à Azure Security Center. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Le contenu est regroupé par les **contrôles de sécurité** définis par le benchmark de sécurité Azure et les conseils associés applicables à Azure Security Center. Les **contrôles** non applicables à Azure Security Center ont été exclus. Pour voir comment Azure Security Center est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité Azure Security Center](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Aide** : Azure Security Center est une offre Azure de base. Vous ne pouvez pas associer un réseau virtuel, un sous-réseau ou un groupe de sécurité réseau directement à Security Center. Si vous activez la collecte de données pour vos ressources informatiques et Security Center stocke les données qu’il collecte via un espace de travail Log Analytics, vous pouvez configurer cet espace de travail pour qu’il utilise Azure Private Link afin d’accéder aux données de votre espace de travail via un point de terminaison privé dans votre réseau virtuel. En outre, si l’utilisation de la collecte de données Security Center s’appuie sur l’agent Log Analytics déployé sur vos serveurs pour collecter des données de sécurité et fournir une protection à ces ressources de calcul. L’agent Log Analytics nécessite l’ouverture de ports et de protocoles spécifiques en vue d’un fonctionnement correct avec Security Center. Verrouillez vos réseaux pour n’autoriser que les ports et protocoles requis et n’ajoutez que les règles supplémentaires dont votre application a besoin pour fonctionner via les groupes de sécurité réseau.

- [Collecte de données dans Azure Security Center](security-center-enable-data-collection.md)

- [Filtrer le trafic réseau avec les groupes de sécurité réseau](../virtual-network/tutorial-filter-network-traffic.md)

- [Configuration de pare-feu requise pour l’utilisation de l’agent Log Analytics](../azure-monitor/platform/log-analytics-agent.md#firewall-requirements)

- [Présentation d’Azure Private Link](../private-link/private-link-overview.md) 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Aide** : L’offre Azure Security Center ne s’intègre pas directement à un réseau virtuel, mais elle peut collecter des données à partir de serveurs configurés avec l’agent Log Analytics qui sont déployés sur vos réseaux. Les serveurs qui sont configurés pour envoyer des données à Security Center requièrent que certains ports et protocoles soient autorisés à communiquer correctement. Définissez et appliquez des configurations de sécurité standard pour ces ressources réseau à l’aide d’Azure Policy.

Vous pouvez également utiliser Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les artefacts d’environnement clés, tels que les modèles Resource Manager, les attributions de rôle et les affectations Azure Policy, au sein d’une seule définition de blueprint. Vous pouvez appliquer le blueprint aux nouveaux abonnements pour déployer des configurations Security Center et des ressources réseau associées de manière cohérente et sécurisée.

- [Collecte de données dans Azure Security Center](security-center-enable-data-collection.md)

- [Configuration de pare-feu requise pour l’utilisation de l’agent Log Analytics](../azure-monitor/platform/log-analytics-agent.md#firewall-requirements)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Exemples Azure Policy pour le réseau](../governance/policy/samples/built-in-policies.md#network)

- [Guide pratique pour créer un blueprint Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Aide** : L’offre Azure Security Center ne s’intègre pas directement à un réseau virtuel, mais elle peut collecter des données à partir de serveurs configurés avec l’agent Log Analytics qui sont déployés sur vos réseaux. Les serveurs qui sont configurés pour envoyer des données à Security Center requièrent que certains ports et protocoles soient autorisés à communiquer correctement. Définissez et appliquez des configurations de sécurité standard pour ces ressources réseau à l’aide d’Azure Policy.

Utilisez des étiquettes de ressources pour les groupes de sécurité réseau et d’autres ressources, comme les serveurs sur vos réseaux, qui sont configurés pour envoyer des journaux de sécurité à Azure Security Center. Pour les règles de groupe de sécurité réseau, utilisez le champ « Description » pour documenter les règles qui autorisent le trafic vers/à partir d’un réseau. 

Utilisez l’une des définitions Azure Policy intégrées en lien avec l’étiquetage, comme « Exiger une étiquette et sa valeur », pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes. 

- [Collecte de données dans Azure Security Center](security-center-enable-data-collection.md)

- [Configuration de pare-feu requise pour l’utilisation de l’agent Log Analytics](../azure-monitor/platform/log-analytics-agent.md#firewall-requirements)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md) 

- [Comment créer un réseau virtuel Azure](../virtual-network/quick-create-portal.md) 

- [Comment filtrer le trafic réseau avec les règles de groupes de sécurité réseau](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Aide** : Utilisez le journal d’activité Azure pour superviser les configurations des ressources et détecter les changements des ressources réseau associées à Azure Security Center. Créez des alertes dans Azure Monitor pour être averti en cas de modification des ressources critiques.

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log) 

- [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Regroupez les données de sécurité générées par Azure Security Center et ses sources connectées en utilisant un espace de travail Log Analytics central. 

Configurez la collecte des données de Security Center pour envoyer des données et des événements de sécurité à partir de vos ressources de calcul Azure connectées vers un espace de travail Log Analytics central. Outre la collecte de données, utilisez la fonctionnalité d’exportation continue pour diffuser en continu les alertes de sécurité et les recommandations générées par Security Center à votre espace de travail Log Analytics central. Dans Azure Monitor, vous pouvez interroger et effectuer des analyses sur les données de sécurité générées à partir de Security Center et de vos ressources Azure connectées. 

Vous pouvez également envoyer des données générées par Security Center à Azure Sentinel ou à une solution SIEM tierce.

- [Exporter en continu les données Security Center](continuous-export.md)

- [Collecte de données dans Azure Security Center](security-center-enable-data-collection.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md) 

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md) 

- [Guide pratique pour collecter les journaux des hôtes internes des machines virtuelles Azure avec Azure Monitor](../azure-monitor/learn/quick-collect-azurevm.md)

- [Guide pratique pour bien démarrer avec Azure Monitor et l’intégration d’une solution SIEM tierce](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Les journaux d’activité Azure Monitor sont automatiquement disponibles ; ils contiennent toutes les opérations d’écriture pour votre ressource comme Azure Security Center, notamment quelles opérations ont été effectuées, qui a lancé l’opération, et quand elles ont eu lieu. Envoyez vos journaux d’activité Azure à un espace de travail Log Analytics pour la consolidation des journaux et une rétention accrue.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md) 

- [Présentation de la journalisation et des différents types de journaux dans Azure](../azure-monitor/platform/platform-logs-overview.md)

- [Envoyer les journaux d’activité à un espace de travail Log Analytics](../azure-monitor/platform/activity-log.md#send-to-log-analytics-workspace)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Dans Azure Monitor, définissez la période de rétention de votre espace de travail Log Analytics en fonction des réglementations de conformité de votre organisation. Utilisez les comptes de stockage Azure pour le stockage à long terme et l’archivage. 

- [Modification de la période de conservation des données dans Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period) 

- [Guide pratique pour configurer la stratégie de conservation des journaux de compte de Stockage Azure](../storage/common/storage-monitor-storage-account.md#configure-logging)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Analysez et surveillez les journaux générés par Azure Security Center et ses sources connectées à la recherche d’un comportement anormal et passez régulièrement en revue les résultats. Utilisez Azure Monitor et un espace de travail Log Analytics pour examiner les journaux et effectuer des requêtes sur leurs données.

Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce. 

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md) 

- [Bien démarrer avec les requêtes Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md) 

- [Guide pratique pour effectuer des requêtes personnalisées dans Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Aide** : Configurez les alertes du journal Azure Monitor pour rechercher des activités indésirables ou anormales qui sont enregistrées par votre journal d’activité ou les données produites par Azure Security Center. Configurez des groupes d’actions afin que votre organisation soit informée et puisse agir si une alerte de journal est lancée pour une activité anormale. Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher des applications logiques sur les alertes et recommandations de sécurité. Les workflows Security Center permettent d’informer les utilisateurs sur la réponse aux incidents, ou de prendre des mesures pour corriger les ressources en fonction des informations d’alerte.

Vous pouvez également activer et intégrer dans Azure Sentinel, des données relatives à et produites par Azure Security Center. Azure Sentinel prend en charge les règles de réponse automatisée aux problèmes liés à la sécurité.

- [Automatisation des workflows Azure Security Center](workflow-automation.md)

- [Guide pratique pour gérer les alertes dans Azure Security Center](security-center-managing-and-responding-alerts.md) 

- [Guide pratique pour générer une alerte sur des données de journal Log Analytics](../azure-monitor/learn/tutorial-response.md)

- [Configurer des réponses automatisées aux menaces dans Azure Sentinel](../sentinel/tutorial-respond-threats-playbook.md)

- [Alertes de journal dans Azure Monitor](../azure-monitor/platform/alerts-unified-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) vous permet de gérer l’accès aux ressources Azure par le biais d’attributions de rôles. Vous pouvez affecter ces rôles à des utilisateurs, regrouper des principaux de service et des identités managées. Les rôles intégrés prédéfinis pour certaines ressources peuvent être inventoriés ou interrogés par le biais d’outils tels que Azure CLI, Azure PowerShell ou le Portail Azure. Azure Security Center possède des rôles intégrés qui peuvent être affectés comme « lecteur de sécurité » et « administrateur de sécurité », ce qui permet aux utilisateurs de lire ou mettre à jour les stratégies de sécurité et d’ignorer les alertes et les recommandations.

- [Autorisations dans Azure Security Center](security-center-permissions.md)

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0) 

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Aide** : Créez des procédures d’exploitation standard autour de l’utilisation de comptes d’administration dédiés pour la plateforme Azure ou spécifique à l’offre Azure Security Center. Utilisez la gestion des identités et des accès Azure Security Center pour superviser le nombre de comptes d’administration d’Azure Active Directory. Security Center possède également des rôles intégrés pour la sécurité « admin » qui permet aux utilisateurs de mettre à jour les stratégies de sécurité et d’ignorer les alertes et les recommandations, de vérifier et de rapprocher les utilisateurs qui ont cette attribution de rôle régulièrement.

En outre, pour vous aider à suivre les comptes d’administration dédiés, vous pouvez utiliser des recommandations d’Azure Security Center ou des stratégies Azure intégrées, telles que les suivantes :

- Plusieurs propriétaires doivent être affectés à votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

- [Autorisations dans Azure Security Center](security-center-permissions.md)

- [Utilisation d’Azure Security Center pour superviser l’identité et l’accès (préversion)](security-center-identity-access.md)

- [Utilisation d’Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : Dans la mesure du possible, utilisez l’authentification unique (SSO) d’Azure Active Directory au lieu de configurer des informations d’identification autonomes individuelles par service. Suivez les suggestions d’Azure Security Center liées à la gestion des identités et des accès.

- [Présentation de l’authentification SSO avec Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Aide** : Activez Azure Active Directory MFA pour accéder à Azure Security Center et au Portail Azure, suivez les recommandations relatives aux identités et aux accès Security Center. 

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md) 

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](security-center-identity-access.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Aide** : Utilisez une station de travail sécurisée et managée par Azure (également appelée station de travail à accès privilégié) pour les tâches administratives qui requièrent des privilèges élevés.

- [Comprendre les stations de travail sécurisées gérées par Azure](../active-directory/devices/concept-azure-managed-workstation.md)

- [Procédure d’activation d’Azure AD MFA](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils** : Utilisez les rapports de sécurité Azure Active Directory et la supervision pour détecter les activités suspectes ou potentiellement dangereuses qui se produisent dans l’environnement. Utiliser Azure Security Center pour superviser les activités liées aux identités et aux accès.

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../active-directory/identity-protection/overview-identity-protection.md) 

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](security-center-identity-access.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Aide** : Utilisez des emplacements nommés Azure AD pour n’autoriser l’accès qu’à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions. 

- [Procédure de configuration des emplacements nommés Azure AD](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Utilisez Azure Active Directory (Azure AD) comme système central d’authentification et d’autorisation avec Azure Security Center. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur. Azure Security Center possède des rôles intégrés qui peuvent être affectés comme « administrateur de sécurité », ce qui permet aux utilisateurs de mettre à jour les stratégies de sécurité et d’ignorer les alertes et les recommandations.

- [Autorisations dans Azure Security Center](security-center-permissions.md)

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory fournit des journaux pour vous aider à découvrir les comptes obsolètes. Par ailleurs, utilisez les révisions d’accès et des identités Azure AD pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès utilisateur lié à Azure Security Center peut être vérifié régulièrement de sorte que seuls les utilisateurs appropriés continuent de bénéficier d’un accès. 

- [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml) 

- [Utilisation des révisions d’accès et des identités Azure AD](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Aide** : Vous avez accès aux activités de connexion Azure AD, aux sources des journaux d’événements à risque et d’audit, ce qui vous permet de les intégrer à un outil SIEM/de supervision. 

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure AD et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes souhaitées dans un espace de travail Log Analytics.  

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte 

**Aide** : Utilisez les fonctionnalités d’Azure AD Identity Protection pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. Vous pouvez également ingérer des données dans Azure Sentinel pour approfondir votre examen. 

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md) 

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md) 

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Aide** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure telles que l’espace de travail Log Analytics qui stocke des informations de sécurité confidentielles à partir d’Azure Security Center.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Aide** : Implémentez une isolation en utilisant des abonnements et groupes d’administration distincts selon les domaines de sécurité (par exemple, le type d’environnement et le niveau de confidentialité des données). Vous pouvez limiter le niveau d’accès à vos ressources Azure demandées par vos applications et environnements d’entreprise. Vous pouvez contrôler l’accès aux ressources Azure via Azure RBAC.

Par défaut, les données Azure Security Center sont stockées dans le service principal Security Center. Si votre organisation a ajouté des conditions requises pour stocker ces données dans vos ressources personnelles, vous pouvez configurer un espace de travail Log Analytics pour stocker des données, des alertes et des recommandations Security Center. Lorsque vous utilisez votre propre espace de travail, vous pouvez ajouter une séparation supplémentaire en configurant différents espaces de travail en fonction de l’environnement d’où proviennent les données.

- [Exporter en continu les données Security Center](continuous-export.md)

- [Collecte de données dans Azure Security Center](security-center-enable-data-collection.md)

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md) 

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md) 

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Instructions** : Chiffrer toutes les informations sensibles en transit. Assurez-vous que les clients qui se connectent à vos ressources Azure peuvent négocier TLS 1.2 ou une version ultérieure. Toutes les machines virtuelles configurées avec l’agent Log Analytics et pour envoyer des données à Azure Security Center doivent être configurées pour utiliser TLS 1.2.

Suivez les recommandations d’Azure Security Center relatives au chiffrement au repos et au chiffrement en transit, le cas échéant. 

- [Envoi de données en toute sécurité à Log Analytics](../azure-monitor/platform/data-security.md#sending-data-securely-using-tls-12)

- [Présentation du chiffrement en transit avec Azure](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources 

**Aide** : Utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour gérer l’accès aux données et ressources associées à Azure Security Center. Azure Security Center possède des rôles intégrés qui peuvent être affectés comme « lecteur de sécurité » et « administrateur de sécurité », ce qui permet aux utilisateurs de lire ou mettre à jour les stratégies de sécurité et d’ignorer les alertes et les recommandations. L’espace de travail Log Analytics qui stocke les données collectées par Security Center possède également des rôles intégrés que vous pouvez attribuer comme « Lecteur Log Analytics », « Contributeur Log Analytics », entre autres. Attribuez le rôle le moins permissif permettant aux utilisateurs d’effectuer leurs tâches. Par exemple, affectez le rôle Lecteur aux utilisateurs qui n’ont besoin que de consulter des informations sur l’intégrité de la sécurité d’une ressource sans effectuer aucune action, telles que l’application des recommandations ou la modification des stratégies.

- [Autorisations pour l’espace de travail Azure Log Analytics](../role-based-access-control/built-in-roles.md#log-analytics-reader)

- [Autorisations dans Azure Security Center](security-center-permissions.md)

- [Comment configurer Azure RBAC](../role-based-access-control/role-assignments-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide** : Azure Security Center utilise un espace de travail Log Analytics configuré pour stocker les données, les alertes et les recommandations qu’il génère. Configurez une clé managée par le client (CMK) pour l’espace de travail que vous avez configuré pour la collecte de données Security Center. CMK permet de chiffrer toutes les données enregistrées ou envoyées à l’espace de travail à l’aide d’une clé de Azure Key Vault créée et dont vous êtes propriétaire. 

- [Clé gérée par le client dans Azure Monitor](../azure-monitor/platform/customer-managed-keys.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor pour créer des alertes lorsque des modifications sont apportées à des ressources Azure critiques relatives à Azure Security Center. Ces modifications peuvent inclure toutes les actions qui modifient les configurations relatives à Security Center, telles que la désactivation des alertes ou des recommandations, ou la mise à jour ou la suppression des magasins de données.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Aide** : Utilisez un programme de score de risque commun (par exemple, le système de notation des vulnérabilités courantes) ou les niveaux de risque par défaut fournis par votre outil d’analyse tiers.

- [Publication du NIST--Système de notation des vulnérabilités courantes](https://www.nist.gov/publications/common-vulnerability-scoring-system)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Aide** : Utilisez Azure Resource Graph pour interroger et découvrir toutes les ressources associées à Azure Security Center au sein de vos abonnements. Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure pour découvrir les ressources Security Center. 

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md) 

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0) 

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Aide** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure telles que l’espace de travail Log Analytics qui stocke des informations de sécurité confidentielles à partir d’Azure Security Center.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Aide** : le cas échéant, utilisez un étiquetage, des groupes d’administration et des abonnements distincts pour organiser et suivre les ressources Azure Security. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

En outre, utilisez la stratégie Azure pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements client à l’aide des définitions de stratégie intégrées suivantes :

- Types de ressources non autorisés
- Types de ressources autorisés

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

**Aide** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements. 

Utilisez Azure Resource Graph pour interroger et découvrir des ressources dans leurs abonnements.  Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées. 

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Aide** : Supprimez les ressources Azure associées à Azure Security Center lorsqu’elles ne sont plus nécessaires dans le cadre de l’inventaire et du processus de vérification de votre organisation.

- [Suppression de ressources et de groupes de ressources Azure](../azure-resource-manager/management/delete-resource-group.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements, en utilisant Azure Policy avec les définitions intégrées suivantes :

- Types de ressources non autorisés
- Types de ressources autorisés

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Configurez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

- [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Définissez et implémentez des configurations de sécurité standard pour Azure Security Center et son espace de travail connecté via Azure Policy. Utilisez les alias Azure Policy dans les espaces de noms « Microsoft.OperationalInsights » et « Microsoft.Security » pour créer des définitions Azure Policy personnalisées afin d’auditer ou d’appliquer la configuration de Security Center et de son espace de travail Log Analytics.

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : Utilisez les effets Azure Policy « refuser » et « déployer s’il n’existe pas » pour appliquer des paramètres sécurisés à vos ressources Azure. En outre, vous pouvez utiliser des modèles Azure Resource Manager pour appliquer la configuration de sécurité des ressources Azure requise par votre organisation. 

- [Comprendre les effets d'Azure Policy](../governance/policy/concepts/effects.md) 

- [Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md) 

- [Présentation des modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Utilisez Azure DevOps pour stocker et gérer de manière sécurisée votre code, comme les définitions de stratégies Azure personnalisées, les modèles Azure Resource Manager et les scripts Desired State Configuration. Pour accéder aux ressources que vous gérez dans Azure DevOps, vous pouvez accorder ou refuser des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps, ou à Active Directory s’il est intégré à TFS. 

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops) 

- [À propos des autorisations et des groupes dans Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour les ressources Azure à l’aide d’Azure Policy. Utilisez des alias Azure Policy pour créer des stratégies personnalisées afin d’auditer ou d’appliquer la configuration de vos ressources Azure Security Center associées. Par ailleurs, vous pouvez utiliser Azure Automation pour déployer les changements de configuration. 

- [Configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Utiliser des alias](../governance/policy/concepts/definition-structure.md#aliases)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Aide** : Utilisez les définitions de Azure Policy intégrées, ainsi que les alias Azure Policy dans les espaces de noms « Microsoft. OperationalInsights » et « Microsoft. Security » afin de créer des stratégies personnalisées pour alerter, auditer et appliquer des configurations de ressources Azure. Utilisez les effets de stratégie Azure Policy « auditer », « refuser » et « déployer s’il n’existe pas » afin d’appliquer automatiquement des configurations pour vos ressources Azure.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Aide** : Azure Security Center utilise un espace de travail Log Analytics configuré pour stocker les données, les alertes et les recommandations qu’il génère. Configurez une clé managée par le client (CMK) pour l’espace de travail que vous avez configuré pour la collecte de données Security Center. CMK permet de chiffrer toutes les données enregistrées ou envoyées à l’espace de travail à l’aide d’une clé de Azure Key Vault créée et dont vous êtes propriétaire. 

- [Clé gérée par le client dans Azure Monitor](../azure-monitor/platform/customer-managed-keys.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

- [Comment configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Conseils** : Azure Security Center n’est pas destiné au stockage ou au traitement des fichiers. Il vous incombe de pré-analyser tout contenu chargé vers des ressources Azure autres que de calcul, notamment l’espace de travail Log Analytics.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-recovery"></a>Récupération de données

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières 

**Aide** : Suivez une approche infrastructure en tant que code (IAC) et utilisez Azure Resource Manager pour déployer vos ressources Azure Security Center associées dans un modèle JavaScript Object Notation (JSON) qui peut être utilisé comme sauvegarde pour les configurations liées aux ressources.

- [Exportation monoressource ou multiressource vers un modèle sur le portail Azure](../azure-resource-manager/templates/export-template-portal.md)

- [Modèles Azure Resource Manager pour la ressource de sécurité](/azure/templates/microsoft.security/allversions)

- [À propos d’Azure Automation](../automation/automation-intro.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés managées par le client

**Aide** : Azure Security Center utilise un espace de travail Log Analytics pour stocker les données, les alertes et les recommandations qu’il génère. Vous pouvez configurer Azure Monitor et l’espace de travail que le Security Center utilise pour activer une clé managée par le client. Si vous utilisez Azure Key Vault pour stocker vos clés managées par le client, effectuez régulièrement des sauvegardes automatiques de vos clés.

- [Comment sauvegarder des clés Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseil** : assurez-vous que la restauration s’effectue régulièrement à l’aide des fichiers de modèle sauvegardés Azure Resource Manager. Testez la restauration des clés gérées par le client sauvegardées.

- [Gérer un espace de travail Log Analytics à l’aide de modèles Azure Resource Manager](../azure-monitor/samples/resource-manager-workspace.md)

- [Guide pratique pour restaurer des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés managées par le client

**Aide** : Utilisez Azure DevOps pour stocker et gérer de manière sécurisée votre code, comme les définitions de stratégies Azure personnalisées et les modèles Azure Resource Manager. Pour protéger les ressources que vous gérez dans Azure DevOps, vous pouvez octroyer ou refuser des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps ou à Active Directory s’il est intégré à TFS. Utilisez le contrôle d’accès en fonction du rôle Azure pour protéger les clés managées par le client.

En outre, activez la suppression réversible et la protection contre la purge dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.  Si le Stockage Azure est utilisé pour stocker les sauvegardes de modèle Azure Resource Manager, la suppression réversible vous permet d’enregistrer et de récupérer vos données en cas de suppression d’objets blob ou d’instantanés d’objets blob. 

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [À propos des autorisations et des groupes dans Azure DevOps](/azure/devops/organizations/security/about-permissions)

- [Guide pratique pour activer la suppression réversible et la protection contre la purge dans Key Vault](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal) 

- [Suppression réversible pour les objets blob de Stockage Azure](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

**Supervision d’Azure Security Center** : Non applicable

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

En outre, marquez les abonnements à l’aide d’étiquettes et créez un système de nommage pour identifier et classer les ressources Azure, en particulier celles qui traitent des données sensibles.  Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit. 

- [Alertes de sécurité dans le Centre de sécurité Azure](security-center-alerts-overview.md) 

- [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Effectuez des exercices pour tester les capacités de réponse aux incidents de vos systèmes à intervalles réguliers, afin de protéger vos ressources Azure. Identifiez les points faibles et les lacunes, puis révisez votre plan de réponse en fonction des besoins. 

- [Publication du NIST--Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities(Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Instructions** : Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus. 

- [Comment définir le contact de sécurité d’Azure Security Center](security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations Azure Security Center en utilisant la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel. 

- [Comment configurer l’exportation continue](continuous-export.md) 

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation de workflow d’Azure Security Center pour déclencher automatiquement des réponses aux alertes et aux recommandations de sécurité afin de protéger vos ressources Azure. 

- [Comment configurer l’automatisation du workflow dans Security Center](workflow-automation.md)

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