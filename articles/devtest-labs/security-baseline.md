---
title: Base de sécurité Azure pour Azure DevTest Labs
description: Base de sécurité Azure pour Azure DevTest Labs
ms.topic: conceptual
ms.date: 07/23/2020
ms.openlocfilehash: 4ae3886e1c47b1d6835dc348b474e3d9e82a8f41
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348821"
---
# <a name="azure-security-baseline-for-azure-devtest-labs"></a>Base de sécurité Azure pour Azure DevTest Labs

La base de sécurité Azure pour Azure DevTest Labs contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Contrôle de sécurité : journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées
**Conseils :** Microsoft gère les sources de temps pour les ressources Azure. Toutefois, vous pouvez gérer les paramètres de synchronisation temporelle de vos ressources de calcul.

Pour en savoir plus sur la configuration de la synchronisation temporelle des ressources de calcul Azure, consultez l’article suivant : [Synchronisation de l’heure pour les machines virtuelles Windows dans Azure](../virtual-machines/windows/time-sync.md). 

**Surveillance d’Azure Security Center :** actuellement non disponible

**Responsabilité :** Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale
**Conseils :** activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux vers un espace de travail Log Analytics, Event Hub ou un compte de stockage Azure pour archivage. Les journaux d’activité fournissent des informations concernant les opérations effectuées sur vos instances Azure DevTest Labs au niveau du plan de gestion. À l’aide des données du journal d’activité Azure, vous pouvez déterminer le « quoi, qui et quand » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de gestion pour vos instances DevTest Labs.

Pour plus d’informations, consultez [Créer des paramètres de diagnostic pour envoyer des journaux et des métriques de plateforme à différentes destinations](../azure-monitor/platform/diagnostic-settings.md).

**Surveillance d’Azure Security Center :** actuellement non disponible

**Responsabilité :** Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure
**Conseils :** activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux vers un espace de travail Log Analytics, Event Hub ou un compte de stockage Azure pour archivage. Les journaux d’activité fournissent des informations concernant les opérations effectuées sur vos instances Azure DevTest Labs au niveau du plan de gestion. À l’aide des données du journal d’activité Azure, vous pouvez déterminer le « quoi, qui et quand » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de gestion pour vos instances DevTest Labs.

Pour plus d’informations, consultez [Créer des paramètres de diagnostic pour envoyer des journaux et des métriques de plateforme à différentes destinations](../azure-monitor/platform/diagnostic-settings.md).

**Surveillance d’Azure Security Center :** actuellement non disponible

**Responsabilité :** Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation
**Conseils :** Les machines virtuelles Azure DevTest Labs sont créées et détenues par le client. Il incombe donc à l’organisation de les surveiller. Vous pouvez utiliser Azure Security Center pour surveiller le système d’exploitation. Les données collectées par Security Center auprès du système d’exploitation incluent le type et la version du système d’exploitation, le système d’exploitation (journaux des événements Windows), les processus en cours d’exécution, le nom de l’ordinateur, les adresses IP et l’utilisateur connecté. L’agent Log Analytics collecte également les fichiers de vidage sur incident.

Pour plus d’informations, consultez les articles suivants : 

- [Guide pratique pour collecter les journaux des hôtes internes des machines virtuelles Azure avec Azure Monitor](../azure-monitor/learn/quick-collect-azurevm.md)
- [Présentation de la collecte de données Azure Security Center](../security-center/security-center-enable-data-collection.md)

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité
**_Conseils :_* Dans Azure Monitor, définissez la période de rétention des journaux pour les espaces de travail Log Analytics associés à vos instances Azure DevTest Labs conformément aux réglementations de conformité de votre organisation.

Pour plus d’informations, consultez l’article suivant : [Guide pratique pour définir les paramètres de conservation des journaux](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux
**Conseils :** Activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux à un espace de travail Log Analytics. Exécutez des requêtes dans Log Analytics pour rechercher des termes, identifier des tendances, analyser des modèles et fournir de nombreuses autres informations basées sur les données du journal d’activité qui ont pu être collectées pour Azure DevTest Labs.

Pour plus d’informations, consultez les articles suivants :

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/diagnostic-settings.md)
- [Collecte et analyse des journaux d’activité Azure dans l’espace de travail Log Analytics dans Azure Monitor](../azure-monitor/platform/activity-log.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer des alertes en cas d’activité anormale
**Conseils :** Utilisez l’espace de travail Azure Log Analytics pour surveiller les activités anormales dans les événements et les journaux de sécurité liés à votre Azure DevTest Labs, et alerter à leur sujet.

Pour plus d’informations, consultez l’article suivant : [Guide pratique pour générer une alerte sur des données de journal Log Analytics](../azure-monitor/learn/tutorial-response.md)

**Surveillance d’Azure Security Center :** actuellement non disponible

**Responsabilité :** Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant
**Conseils :** Non applicable. Azure DevTest Labs ne traite ou ne produit pas de journaux liés aux logiciels anti-programmes malveillants.

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS
**Conseils :** Non applicable. Azure DevTest Labs ne traite ou ne produit pas de journaux liés à DNS.

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande
**Conseils :** Azure DevTest Labs crée des machines Azure Compute qui sont détenues et gérées par le client. Utilisez Microsoft Monitoring Agent sur toutes les machines virtuelles Windows Azure prises en charge pour journaliser l’événement de création de processus et le champ `CommandLine`. En ce qui concerne les machines virtuelles Linux Azure prises en charge, vous pouvez configurer manuellement la journalisation de la console au niveau de chaque nœud et utiliser Syslog pour stocker les données. Exploitez également l’espace de travail Log Analytics d’Azure Monitor pour examiner les journaux et exécuter des requêtes sur les données journalisées à partir de machines virtuelles Azure.

- [Collecte de données dans Azure Security Center](../security-center/security-center-enable-data-collection.md#data-collection-tier)
- [Guide pratique pour exécuter des requêtes personnalisées dans Azure Monitor](../azure-monitor/log-query/get-started-queries.md)
- [Sources de données Syslog dans Azure Monitor](../azure-monitor/platform/data-sources-syslog.md)

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités
*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration
**Conseils :** Azure Active Directory (Azure AD) comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent être interrogés. Utilisez le module Azure AD PowerShell pour exécuter des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration.

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)
- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)
- [Rôles Azure DevTest Labs](devtest-lab-add-devtest-user.md)  

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible
**Conseils :** Azure Active Directory (Azure AD) n’intègre pas le concept des mots de passe par défaut. Selon le service, d’autres ressources Azure qui exigent un mot de passe forcent la création d’un mot de passe conforme à des exigences de complexité et d’une longueur minimale. Vous êtes responsable des applications tierces et des services de la Place de marché susceptibles d’utiliser des mots de passe par défaut.

DevTest Labs n’intègre pas le concept des mots de passe par défaut. 

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés
**Conseils :** Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

En outre, pour vous aider à suivre les comptes d’administration dédiés, vous pouvez utiliser des recommandations d’Azure Security Center ou des stratégies Azure intégrées, telles que les suivantes :

- Plusieurs propriétaires doivent être affectés à votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

- [Utilisation d’Azure Security Center pour superviser l’identité et l’accès (préversion)](../security-center/security-center-identity-access.md)  
- [Utilisation d’Azure Policy](../governance/policy/tutorials/create-and-manage.md)
- [Rôles Azure DevTest Labs](devtest-lab-add-devtest-user.md)  

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory
**Conseils :** DevTest Labs utilise le service Azure AD pour la gestion des identités. Tenez compte de ces deux aspects clés lorsque vous autorisez les utilisateurs à accéder à un environnement basé sur DevTest Labs :

- **Gestion des ressources** : l’accès au portail Azure permet d’effectuer les opérations de gestion des ressources (créer des machines virtuelles, créer des environnements, démarrer, arrêter, redémarrer, supprimer, appliquer des artefacts, etc.). La gestion des ressources s’effectue dans Azure en utilisant le contrôle d’accès en fonction du rôle Azure (Azure RBAC). Vous affectez des rôles aux utilisateurs et définissez les autorisations au niveau des ressources et de l’accès.
- **Machines virtuelles (au niveau du réseau)**  : dans la configuration par défaut, les machines virtuelles utilisent un compte d’administrateur local. S’il existe un domaine disponible (Azure AD Domain Services, un domaine local ou un domaine cloud), il est possible de lui adjoindre des machines. Les utilisateurs peuvent ensuite utiliser leurs identités basées sur le domaine en utilisant l’artefact de jonction de domaine pour se connecter aux machines. 

- [Architecture de référence pour DevTest Labs](devtest-lab-reference-architecture.md#architecture)
- [Présentation de l’authentification SSO avec Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory
**Conseils :** Activez l’authentification multifacteur (MFA) Azure Active Directory (AAD) et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)  
- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Surveillance d’Azure Security Center :** _ Oui

_ *Responsabilité :* * Client


### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives
**Conseils :** Utilisez des stations de travail à accès privilégié (PAW) avec l’authentification multifacteur (MFA) configurée pour la connexion aux ressources Azure et leur configuration.

- [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)  
- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)  

**Surveillance d’Azure Security Center :** N/A

**Responsabilité :** Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration
**Conseils :** Utilisez les rapports de sécurité Azure Active Directory (Azure AD) pour générer des journaux et des alertes quand une activité suspecte ou potentiellement dangereuse se produit dans l’environnement. Utiliser Azure Security Center pour superviser les activités liées aux identités et aux accès.

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../active-directory/identity-protection/overview-identity-protection.md)  
- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../security-center/security-center-identity-access.md)  

**Surveillance d’Azure Security Center :** actuellement non disponible

**Responsabilité :** Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement
**Conseils :** Utilisez des emplacements nommés à accès conditionnel pour autoriser l’accès uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

- [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)  

**Surveillance d’Azure Security Center :** actuellement non disponible

**Responsabilité :** Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory
**Conseils :** Utiliser Azure Active Directory (Azure AD) comme système d’authentification et d’autorisation central. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)  

**Surveillance d’Azure Security Center :** actuellement non disponible

**Responsabilité :** Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur
**Conseils :** Azure Active Directory (Azure AD) fournit des journaux pour vous aider à découvrir les comptes obsolètes. Par ailleurs, utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seules les utilisateurs appropriés continuent de bénéficier d’un accès.

- [Présentation des rapports Azure AD](../active-directory/reports-monitoring/overview-reports.md)  
- [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)  

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés
**Conseils :** Vous pouvez accéder aux sources de journal de l’activité de connexion, de l’audit et des événements à risque d’Azure Active Directory (Azure AD), ce qui vous permet d’intégrer n’importe quel outil SIEM (Security Information and Event Management) ou de supervision.

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure Active Directory et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer des alertes dans un espace de travail Log Analytics.

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)  

**Surveillance d’Azure Security Center :** actuellement non disponible

**Responsabilité :** Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte
**Conseils :** Utilisez les détections de risque et la fonctionnalité Risk and Identity Protection d’Azure Active Directory (Azure AD) pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur.

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)  
- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)  

**Surveillance d’Azure Security Center :** actuellement non disponible

**Responsabilité :** Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support
**Conseils :** Customer Lockbox n’est actuellement pas pris en charge pour Azure DevTest Labs.

- [Liste des services pris en charge pour Customer Lockbox](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability) 

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

## <a name="data-protection"></a>Protection des données
*Pour plus d’informations, consultez [Contrôle de sécurité : protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Tenir un inventaire des informations sensibles
**Conseils :** Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles
**Conseils :** Implémentez des abonnements ou des groupes d’administration distincts pour le développement, les tests et la production. Les instances Azure DevTest Labs doivent être séparées par un réseau virtuel/sous-réseau et balisées de manière appropriée. 

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)
- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)
- [Comment configurer un réseau virtuel pour DevTest Labs](devtest-lab-configure-vnet.md)
- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)
- [Comment créer et utiliser des balises pour DevTest Labs](devtest-lab-add-tag.md)

**Surveillance d’Azure Security Center :** actuellement non disponible

**Responsabilité :** Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles
**Conseils :** Pas encore disponible. Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour Azure DevTest Labs.

Microsoft gère l’infrastructure sous-jacente d’Azure DevTest Labs et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Surveillance d’Azure Security Center :** actuellement non disponible

**Responsabilité :** Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit
**Conseils :** Azure DevTest Labs nécessite des communications chiffrées avec le protocole TLS par défaut. La version 1.2 TLS est actuellement prise en charge. Si votre outil ou bibliothèque de client ne prend pas en charge TLS, vous pouvez activer des connexions non chiffrées par le biais du portail Azure ou des API de gestion. Dans les cas où il est impossible d’établir des connexions chiffrées, nous vous recommandons de placer votre lab et votre application cliente dans un réseau virtuel.

[Comprendre le scénario de chiffrement en transit pour DevTest Labs](https://techcommunity.microsoft.com/t5/azure-developer-community-blog/azure-devtest-labs-enforcing-tls-1-2-starting-may-01-2020/ba-p/1236279)

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles
**Conseils :** Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour Azure DevTest Labs. Baliser des instances contenant des informations sensibles en tant que telles, et implémenter une solution tierce à des fins de conformité si nécessaire.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Surveillance d’Azure Security Center :** actuellement non disponible

**Responsabilité :** Customer

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources
**Conseils :** Utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour contrôler l’accès aux labs dans Azure DevTest Labs.

- [Comment configurer Azure RBAC](../role-based-access-control/role-assignments-portal.md)
- [Comprendre les rôles dans DevTest Labs](devtest-lab-add-devtest-user.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès
**Conseils :** Si nécessaire, à des fins de conformité sur les ressources de calcul créées en tant que partie du DevTest Labs, implémentez un outil tiers par exemple, une solution automatisée de protection contre la perte de données basée sur un hôte pour appliquer des contrôles d’accès aux données même lorsque celles-ci sont copiées à partir d’un système.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Non applicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos
**Conseils :** Azure DevTest Labs stocke les données client suivantes :

- [Résultats de l’artefact](add-artifact-vm.md) qui incluent le déploiement et les journaux d’extension générés à partir de l’application des artefacts
- [Documents de formules](devtest-lab-manage-formulas.md) utilisés pour créer des machines virtuelles à partir de formules
- Disques de système d’exploitation et de données pour les machines virtuelles Lab 

Les résultats d’artefact et les documents de formule sont envoyés à un compte de stockage Azure créé dans le cadre de chaque déploiement de lab. Les données dans Stockage Azure sont chiffrées et déchiffrées en toute transparence à l’aide du chiffrement AES 256 bits, un des chiffrements par blocs les plus puissants actuellement disponibles, et sont conformes à la norme FIPS 140-2. Le chiffrement de Stockage Azure ne peut pas être désactivé. Vous pouvez vous reposer sur les clés managées par Microsoft pour le chiffrement de votre compte de stockage, ou vous pouvez gérer le chiffrement avec vos propres clés. Pour plus d’informations, consultez l’article [Chiffrement d’un compte de stockage lab](encrypt-storage.md).

Par défaut, tous les disques de données et de système d’exploitation Lab sont chiffrés avec une clé gérée par la plateforme. L’ensemble des disques managés, des instantanés et des images ainsi que les données écrites sur des disques managés existants sont automatiquement chiffrés au repos avec des clés gérées par les plateformes. En tant que propriétaire de lab, vous pouvez configurer le chiffrement des disques de système d’exploitation lab à l’aide d’une clé gérée par le client. Le chiffrement à l’aide d’une clé gérée par le client pour les disques de données Lab n’est actuellement pas configurable par le biais du lab lui-même. Toutefois, un administrateur d’abonnement peut configurer ce paramètre pour les disques lab dans un abonnement pour le moment. Pour plus d’informations, consultez [Chiffrer des disques de système d’exploitation Lab DevTest Labs à l’aide de clés gérées par le client](encrypt-disks-customer-managed-keys.md).

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Partagé

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques
**Conseils :** Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes quand des modifications sont apportées aux instances DevTest Labs ainsi qu’à d’autres ressources critiques ou connexes.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)
- [Guide pratique pour créer des alertes sur les événements du journal d’activité DevTest](create-alerts.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer



## <a name="vulnerability-management"></a>Gestion des vulnérabilités
*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés
**Conseils :** Suivez les recommandations d’Azure Security Center relatives à la sécurisation de vos instances Azure DevTest Labs et des ressources associées.

Microsoft assure la gestion des vulnérabilités sur les ressources sous-jacents prenant en charge Azure DevTest Labs.

- [Comprendre les recommandations d’Azure Security Center](../security-center/recommendations-reference.md) 

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Partagé

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée
**Conseils :** Utilisez Azure Update Management pour vous assurer que les mises à jour de sécurité les plus récentes sont installées sur vos machines virtuelles Windows et Linux hébergées dans DevTest Labs. Pour les machines virtuelles Windows, assurez-vous que Windows Update a été activé et configuré pour être mis à jour automatiquement. Ce paramètre n’est pas disponible actuellement pour la configuration via DevTest Labs. Toutefois, l’administrateur de laboratoire/d’abonnement peut configurer ce paramètre sur les machines virtuelles de calcul sous-jacentes dans leur abonnement. 

- [Configurer Update Management pour des machines virtuelles dans Azure](../automation/update-management/overview.md)
- [Comprendre les stratégies de sécurité Azure analysées par Security Center](../security-center/policy-reference.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3 : Déployer une solution de gestion de correctif logiciel tiers automatisée
**_Conseils :_* _ En tant qu’administrateur de laboratoire, vous pouvez utiliser des [artefacts DevTest Labs](add-artifact-vm.md) pour automatiser les mises à jour des images personnalisées du labo, notamment des correctifs de sécurité et d’autres mises à jour. 

Apprenez-en davantage sur la [Fabrique d’images DevTest Labs](image-factory-create.md), solution de configuration en tant que code qui crée et distribue automatiquement des images à intervalles réguliers avec toutes les configurations souhaitées. 

En tant qu’administrateur d’abonnements, vous pouvez également utiliser la solution Azure Update Management pour gérer les mises à jour et les correctifs pour les machines virtuelles DevTest Labs. Update Management s’appuie sur le référentiel de mise à jour configuré localement pour corriger les systèmes Windows pris en charge. Des outils comme System Center Updates Publisher (Updates Publisher) permettent de publier des mises à jour personnalisées dans Windows Server Update Services (WSUS). Ce scénario permet à Update Management de corriger des machines qui utilisent Configuration Manager comme référentiel de mise à jour avec des logiciels tiers.

- [Solution Update Management dans Azure](../automation/update-management/overview.md)
- [Gérer les mises à jour et les correctifs de vos machines virtuelles](../automation/update-management/overview.md)

_ *Surveillance d’Azure Security Center :* * Non applicable

**Responsabilité :** Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos
**Conseils :** Exportez les résultats de l’analyse à intervalles réguliers et comparez les résultats pour vérifier que les vulnérabilités ont été corrigées. Lorsqu’ils utilisent les recommandations de gestion des vulnérabilités suggérées par Azure Security Center, les clients peuvent basculer vers le portail de la solution sélectionnée pour afficher les données d’analyse historiques.

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.
**Conseils :** Utilisez les évaluations de risques par défaut (degré de sécurisation) fournies par Azure Security Center.

- [Présentation du degré de sécurisation Azure Security Center](../security-center/secure-score-security-controls.md)

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources
*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources
**Conseils :** Utilisez Azure Resource Graph pour interroger et découvrir toutes les ressources (y compris les ressources DevTest Labs) au sein de vos abonnements. Vérifiez que vous disposez des autorisations (en lecture) appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent.

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)
- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)
- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources
**Conseils :** Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)
- [Configurer des balises pour DevTest Labs](devtest-lab-add-tag.md)

**Surveillance d’Azure Security Center :** Non disponible

**Responsabilité :** Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées
**Conseils :** Utilisez un balisage, des groupes d’administration et des abonnements distincts, ainsi que des laboratoires distincts le cas échéant, pour organiser et suivre les laboratoires et les ressources associées. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement rapidement.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)
- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)
- [Guide pratique pour créer un laboratoire à l’aide de DevTest Labs](devtest-lab-create-lab.md)
- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)
- [Guide pratique pour configurer des balises pour un laboratoire](devtest-lab-add-tag.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées
**Conseils :** Créez un inventaire des ressources Azure et logiciels approuvés pour les ressources de calcul en fonction des besoins de votre organisation. En tant qu’administrateur d’abonnements, vous pouvez également utiliser les contrôles d’application adaptatifs, une fonctionnalité d’Azure Security Center, pour définir un ensemble d’applications autorisées à s’exécuter sur des groupes configurés de machines de labo. Cette fonctionnalité est disponible pour les machines et les serveurs Azure et non-Azure Windows (toutes versions, classique ou Azure Resource Manager), mais aussi Linux.

- [Guide pratique pour activer les contrôles d’application adaptatifs](../security-center/security-center-adaptive-application.md)
 
**Surveillance d’Azure Security Center :** Non applicable **responsabilité :** Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées
**Conseils :** Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant une stratégie Azure avec les définitions intégrées suivantes :

- Types de ressources non autorisés
- Types de ressources autorisés

Par ailleurs, utilisez Azure Resource Graph pour interroger/découvrir des ressources dans les abonnements. Cela peut être utile dans les environnements de haute sécurité, tels que ceux dotés de comptes de stockage.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)
- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul
**Conseils :** Azure Automation permet de contrôler totalement le déploiement, les opérations et la désaffectation des charges de travail et des ressources. En tant qu’administrateur d’abonnements, vous pouvez tirer parti de l’inventaire des machines virtuelles Azure pour automatiser la collecte d’informations sur tous les logiciels installés sur les machines virtuelles DevTest Labs dans votre abonnement. Les propriétés de nom, de version, d’éditeur et d’heure d’actualisation du logiciel sont disponibles sur le portail Azure. Pour avoir accès à la date d’installation et à d’autres informations, le client a demandé d’activer les diagnostics au niveau de l’invité et de rassembler les journaux des événements Windows dans un espace de travail Log Analytics.

Outre l’utilisation de Change Tracking pour la surveillance d’applications logicielles, les contrôles d’application adaptatifs d’Azure Security Center utilisent l’apprentissage automatique pour analyser les applications en cours d’exécution sur vos machines et créer une liste verte à partir de cette intelligence. Cette capacité simplifie considérablement le processus de configuration et de gestion des stratégies de mise en liste verte des applications, ce qui vous permet d’éviter l’utilisation de logiciels indésirables dans votre environnement. Vous pouvez configurer le mode audit ou le mode application. Le mode audit vérifie uniquement l’activité sur les machines virtuelles protégées. Le mode application applique les règles, puis s’assure que les applications n’ayant pas d’autorisation sont bloquées. 

- [Présentation d’Azure Automation](../automation/automation-intro.md)
- [Guide pratique pour activer l’inventaire des machines virtuelles Azure](../automation/automation-tutorial-installed-software.md)
- [Présentation des contrôles d’application adaptatifs](../security-center/security-center-adaptive-application.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer


### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées
**Conseils :** Azure Automation permet de contrôler totalement le déploiement, les opérations et la désaffectation des charges de travail et des ressources. En tant qu’administrateur d’abonnements, vous pouvez utiliser Change Tracking pour identifier tous les logiciels installés sur les machines virtuelles hébergées dans DevTest Labs. Vous pouvez implémenter votre propre processus ou utiliser la configuration de l’état Azure Automation pour supprimer les logiciels non autorisés.

- [Présentation d’Azure Automation](../automation/automation-intro.md)
- [Suivre les changements dans votre environnement avec la solution Change Tracking](../automation/change-tracking/overview.md)
- [Vue d’ensemble d’Azure Automation State Configuration](../automation/automation-dsc-overview.md)

**Surveillance d’Azure Security Center :** Non disponible

**Responsabilité :** Customer

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement
**Conseils :** En tant qu’administrateur d’abonnements, vous avez également la possibilité d’exploiter les contrôles d’application adaptatifs Azure Security Center pour faire en sorte que seuls les logiciels autorisés s’exécutent et que l’exécution de tous les logiciels non autorisés soit bloquée sur les machines virtuelles Azure hébergées dans DevTest Labs.

- [Guide pratique pour utiliser les contrôles d’application adaptatifs Azure Security Center](../security-center/security-center-adaptive-application.md)

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement
**Conseils :** Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant une stratégie Azure avec les définitions intégrées suivantes : 

- Types de ressources non autorisés
- Types de ressources autorisés

Voir les articles suivants : 
- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)
- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer


### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés
**Conseils :** Le contrôle d’application adaptatif est une solution de bout en bout intelligente et automatisée d’Azure Security Center qui vous permet de contrôler les applications qui peuvent s’exécuter sur vos machines Azure et non-Azure (Windows et Linux) hébergées dans DevTest Labs. Notez que vous devez être administrateur des abonnements pour pouvoir configurer ce paramètre pour les ressources de calcul sous-jacentes hébergées dans DevTest Labs. Implémentez une solution tierce si ce paramètre ne répond pas aux exigences de votre organisation.

- [Guide pratique pour utiliser les contrôles d’application adaptatifs Azure Security Center](../security-center/security-center-adaptive-application.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager
**Conseils :** Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant **Bloquer l’accès**** pour l’application **Gestion Microsoft Azure**.

- [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul
**Conseils :** Selon le type de script, vous pouvez utiliser des configurations de système d’exploitation spécifiques ou des ressources tierces pour limiter la capacité des utilisateurs à exécuter des scripts dans les machines virtuelles hébergées dans DevTest Labs. Vous avez également la possibilité d’utiliser les contrôles d’application adaptatifs Azure Security Center pour faire en sorte que seuls les logiciels autorisés s’exécutent et que l’exécution de tous les logiciels non autorisés soit bloquée sur les machines virtuelles Azure sous-jacentes.

- [Guide pratique pour contrôler l’exécution des scripts PowerShell dans des environnements Windows](/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-6)
- [Guide pratique pour utiliser les contrôles d’application adaptatifs Azure Security Center](../security-center/security-center-adaptive-application.md)

**Surveillance d’Azure Security Center :** Non disponible

**Responsabilité :** Customer


### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé
**Conseils :** Les applications à risque élevé déployées dans votre environnement Azure peuvent être isolées à l’aide d’un réseau virtuel, d’un sous-réseau, d’abonnements, de groupes d’administration, etc., et suffisamment sécurisées avec un pare-feu Azure, un pare-feu d’applications Web (WAF) ou un groupe de sécurité réseau (NSG).

- [Configurer un réseau virtuel pour DevTest Labs](devtest-lab-configure-vnet.md)
- [Présentation du Pare-feu Azure](../firewall/overview.md)
- [Présentation du pare-feu d’applications web](../web-application-firewall/overview.md)
- [Vue d’ensemble de la sécurité réseau](../virtual-network/network-security-groups-overview.md)
- [Vue d’ensemble de Réseau virtuel Microsoft Azure]()
- [Organiser vos ressources avec des groupes d’administration Azure](../governance/management-groups/overview.md)
- [Guide de décision concernant les abonnements](/azure/cloud-adoption-framework/decision-guides/subscriptions/)

**Surveillance d’Azure Security Center :** Non disponible

**Responsabilité :** Customer

## <a name="secure-configuration"></a>Configuration sécurisée
**Pour plus d’informations, consultez Contrôle de sécurité : Configuration sécurisée.**

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure
**Conseils :** Utilisez des alias Azure Policy pour créer des stratégies personnalisées afin d’auditer ou d’appliquer la configuration de vos ressources Azure créées en tant que partie de DevTest Labs. Vous pouvez aussi utiliser des définitions Azure Policy intégrées.

Par ailleurs, Azure Resource Manager a la possibilité d’exporter le modèle au format JSON (JavaScript Object Notation), qui doit être examiné pour vérifier que les configurations répondent/dépassent les exigences de sécurité de votre organisation.

Vous pouvez aussi utiliser les recommandations d’Azure Security Center comme base de référence de configuration sécurisée pour vos ressources Azure.

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)
- [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)
- [Exportation monoressource ou multiressource vers un modèle sur le portail Azure](../azure-resource-manager/templates/export-template-portal.md)
- [Recommandations de sécurité - Guide de référence](../security-center/recommendations-reference.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation
**Conseils :** Utilisez les recommandations d'Azure Security Center pour préserver les configurations de sécurité sur toutes les ressources de calcul sous-jacentes créées en tant que partie de DevTest Labs. En outre, vous pouvez utiliser des images de système d'exploitation personnalisées ou Azure Automation State Configuration pour établir la configuration de sécurité du système d'exploitation requise par votre organisation.

- [Suivi des recommandations d'Azure Security Center](../security-center/security-center-recommendations.md)
- [Recommandations de sécurité - Guide de référence](../security-center/recommendations-reference.md)
- [Vue d’ensemble d’Azure Automation State Configuration](../automation/automation-dsc-overview.md)
- [Charger un disque dur virtuel et l'utiliser pour créer des machines virtuelles Windows dans Azure](../virtual-machines/windows/upload-generalized-managed.md)
- [Créer une machine virtuelle Linux à partir d'un disque personnalisé avec Azure CLI](../virtual-machines/linux/upload-vhd.md)
- [Créer et distribuer des images personnalisées à plusieurs laboratoires DevTest](image-factory-save-distribute-custom-images.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Non applicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées
**Conseils :** Utilisez les commandes Azure Policy **refuser** et **déployer s’il n’existe pas** pour appliquer des paramètres sécurisés à vos ressources Azure créées en tant que partie de DevTest Labs. En outre, vous pouvez utiliser des modèles Azure Resource Manager pour appliquer la configuration de sécurité des ressources Azure requise par votre organisation.

- [Comprendre les effets d'Azure Policy](../governance/policy/concepts/effects.md)
- [Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)
- [Présentation des modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation
**Conseils :** Suivez les recommandations d’Azure Security Center en matière d'évaluation des vulnérabilités sur vos ressources de calcul Azure sous-jacentes créées en tant que partie d’un lab. En outre, vous pouvez utiliser des modèles Azure Resource Manager, des images de système d'exploitation personnalisées ou Azure Automation State Configuration pour appliquer la configuration de sécurité du système d'exploitation requise par votre organisation. Vous pouvez également utiliser une fabrique d’images, qui est une solution avec une configuration sous forme de code qui crée et distribue régulièrement des images, automatiquement et avec toutes les configurations nécessaires.

Notez également que les images de machines virtuelles de la Place de marché Azure publiées par Microsoft sont gérées et appliquées par Microsoft.

- [Implémenter les recommandations d'évaluation des vulnérabilités d'Azure Security Center](../security-center/deploy-vulnerability-assessment-vm.md)
- [Vue d’ensemble d’Azure Automation State Configuration](../automation/automation-dsc-overview.md)
- [Exemple de script permettant de charger un disque dur virtuel sur Azure et de créer une machine virtuelle](../virtual-machines/scripts/virtual-machines-windows-powershell-upload-generalized-script.md)
- [Comment créer une fabrique d’images dans DevTest Labs](image-factory-create.md)

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Partagé

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure
**Conseils :** Utilisez Azure DevOps pour stocker et gérer en toute sécurité votre code, comme les stratégies Azure personnalisées, les modèles Azure Resource Manager et les scripts Desired State Configuration. Pour accéder aux ressources que vous gérez dans Azure DevOps, vous pouvez accorder ou refuser des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps.

- [Tutoriel Git Azure Repos](/azure/devops/repos/git/gitworkflow)
- [À propos des autorisations et des groupes](/azure/devops/organizations/security/about-permissions?tabs=preview-page&view=azure-devops)
- [Intégration du flux de travail d’Azure DevTest Labs et d’Azure DevOps](devtest-lab-dev-ops.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées
**Conseils :** Si vous avez recours à des images personnalisées, utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour garantir que seuls les utilisateurs autorisés ont accès à ces images. À l’aide d’une galerie Shared Image Gallery, vous pouvez partager vos images avec des laboratoires spécifiques qui en ont besoin. Pour les images conteneur, stockez-les dans Azure Container Registry et tirez parti d’Azure RBAC pour vous assurer que seuls les utilisateurs autorisés peuvent accéder aux images.

- [Présentation d’Azure RBAC](../role-based-access-control/rbac-and-directory-admin-roles.md)
- [Comment configurer Azure RBAC](../role-based-access-control/quickstart-assign-role-user-portal.md)
- [Configurer une galerie Shared Image Gallery dans DevTest Labs](configure-shared-image-gallery.md)
- [Comprendre Azure RBAC pour Container Registry](../container-registry/container-registry-roles.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="77-deploy-system-configuration-management-tools"></a>7.7 : Déployer les outils de gestion de configuration système
**Conseils :** Définissez et implémentez des configurations de sécurité standard pour les ressources Azure à l’aide d’Azure Policy. Utilisez des alias Azure Policy pour créer des stratégies personnalisées afin d’auditer ou d’appliquer la configuration réseau de vos ressources Azure créées en tant que partie de DevTest Labs. Vous pouvez également utiliser des définitions de stratégie intégrées en lien avec vos ressources spécifiques. Par ailleurs, vous pouvez utiliser Azure Automation pour déployer les changements de configuration.

- [Configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)
- [Utiliser des alias](../governance/policy/concepts/definition-structure.md#aliases)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation
**Conseils :** Azure Automation State Configuration est un service de gestion de la configuration pour les nœuds DSC dans n’importe quel cloud ou centre de données local. Vous pouvez facilement intégrer des machines, leur affecter des configurations déclaratives et afficher des rapports montrant la conformité de chaque machine à l’état souhaité que vous avez spécifié. Vous pouvez également écrire un artefact personnalisé qui peut être installé sur chaque machine Lab pour s’assurer qu’il respecte les stratégies organisationnelles. 

- [Intégration des machines pour la gestion avec Azure Automation State Configuration](../automation/automation-dsc-onboarding.md)
- [Créer des artefacts personnalisés pour vos machines virtuelles DevTest Labs](devtest-lab-artifact-author.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la surveillance de la configuration automatique pour les services Azure
**Conseils :** Utiliser Azure Security Center pour effectuer des analyses de ligne de base pour vos ressources Azure créées sous DevTest Labs. En outre, utilisez Azure Policy pour alerter et auditer les configurations des ressources Azure.

- [Corriger les recommandations dans Azure Security Center](../security-center/security-center-remediate-recommendations.md)
 
**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation
**Conseils :** Utilisez Azure Security Center pour effectuer des analyses de base pour les paramètres de système d’exploitation et d’ancrage des conteneurs.

- [Comprendre les recommandations concernant les conteneurs dans Azure Security Center](../security-center/container-security.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité
**Conseils :** Utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets pour vos applications Cloud.

- [Configurer l’identité managée pour déployer des environnements Azure Resource Manager dans DevTest Labs](use-managed-identities-environments.md)
- [Configurer l’identité gérée pour déployer des machines virtuelles dans DevTest Labs](enable-managed-identities-lab-vms.md)
- [Créer un coffre de clés](../key-vault/general/quick-create-portal.md)
- [Comment s’authentifier auprès de Key Vault](../key-vault/general/authentication.md)
- [Comment attribuer une stratégie d’accès Key Vault](../key-vault/general/assign-access-policy-portal.md)

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique
**Conseils :** Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure AD. Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Key Vault, sans informations d’identification dans votre code.

- [Configurer l’identité managée pour déployer des environnements Azure Resource Manager dans DevTest Labs](use-managed-identities-environments.md)
- [Configurer l’identité gérée pour déployer des machines virtuelles dans DevTest Labs](enable-managed-identities-lab-vms.md)
 
**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification
**Conseils :** Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

- Comment configurer Credential Scanner

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer


## <a name="malware-defense"></a>Défense contre les programmes malveillants
*Pour plus d’informations, consultez Contrôle de sécurité : Défense contre les programmes malveillants.*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée
**Conseils :** Utilisez Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles afin de superviser et défendre en continu vos ressources. Pour Linux, utilisez une solution logicielle anti-programme malveillant tierce. Utilisez également la détection des menaces pour les services de données d’Azure Security Center afin de détecter les programmes malveillants chargés sur des comptes de stockage.

- Guide pratique pour configurer Microsoft Antimalware pour les services cloud et les machines virtuelles
- Protection contre les menaces dans Azure Security Center

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul
**Conseils :** Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure App Service hébergé dans un labo), mais il ne s’exécute pas sur votre contenu.
Pré-analysez les fichiers chargés sur des ressources Azure non liées au calcul, comme App Service, des Data Lake Storage, Stockage Blob, etc.

Utilisez la détection des menaces pour les services de données dans Azure Security Center pour détecter les programmes malveillants chargés sur les comptes de stockage.

- Présentation de Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles
- Présentation de la détection des menaces pour les services de données d’Azure Security Center

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Non applicable


### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour
**Conseils :** Lorsqu’il est déployé, Microsoft Antimalware pour Azure installe automatiquement les dernières signatures, plateformes et mises à jour du moteur. Suivez les recommandations faites dans Azure Security Center : « Calcul et applications » pour vous assurer que tous les points de terminaison des ressources de calcul sous-jacentes de DevTest Labs sont à jour avec les dernières signatures. La protection du système d’exploitation Windows peut encore être renforcée par une sécurité supplémentaire afin de limiter le risque d’attaques basées sur les virus ou les logiciels malveillants avec le service Microsoft Defender - Protection avancée contre les menaces, qui s’intègre à Azure Security Center.

- Guide pratique pour déployer Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles
- Microsoft Defender - Protection avancée contre les menaces

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

## <a name="data-recovery"></a>Récupération de données
*Pour plus d’informations, consultez [Contrôle de sécurité : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : Garantir des sauvegardes automatiques régulières
**Conseils :** Actuellement, Azure DevTest Labs ne prend pas en charge les sauvegardes et les captures instantanées de machine virtuelle. Toutefois, vous pouvez activer et configurer le service Sauvegarde Azure sur les machines virtuelles Azure sous-jacentes hébergées dans DevTest Labs. Vous pouvez également configurer la fréquence souhaitée et la période de rétention pour les sauvegardes automatiques, à condition que vous disposiez de l’accès approprié aux ressources de calcul sous-jacentes. 

- [Vue d’ensemble de la sauvegarde de machines virtuelles Azure](../backup/backup-azure-vms-introduction.md)
- [Sauvegarder une machine virtuelle Azure à partir des paramètres de la machine virtuelle](../backup/backup-azure-vms-first-look-arm.md)

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client
**Conseils :** Actuellement, Azure DevTest Labs ne prend pas en charge les sauvegardes et les captures instantanées de machine virtuelle. Toutefois, vous pouvez créer des captures instantanées de vos machines virtuelles Azure sous-jacentes hébergées dans DevTest Labs, ou des disques managés attachés à ces instances à l’aide de PowerShell ou d’API REST, à condition que vous disposiez de l’accès approprié aux ressources de calcul sous-jacentes. Vous pouvez également sauvegarder toutes les clés gérées par le client dans Azure Key Vault.

Activez le service Sauvegarde Azure sur les machines virtuelles Azure cibles, et définissez la fréquence et les périodes de rétention souhaitées. Cela inclut une sauvegarde complète de l’état du système. Si vous utilisez le chiffrement de disques Azure, la sauvegarde de machine virtuelle Azure gère automatiquement la sauvegarde des clés gérées par le client.

- [Sauvegarde sur des machines virtuelles Azure utilisant le chiffrement](../backup/backup-azure-vms-encryption.md)
- [Vue d’ensemble de la sauvegarde de machines virtuelles Azure](../backup/backup-azure-vms-introduction.md)
- [Vue d’ensemble de la sauvegarde de machines virtuelles Azure](../backup/backup-azure-vms-introduction.md)
- [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey?view=azurermps-6.13.0)

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client
**Conseils :** Assurez-vous que la restauration des données du contenu s’effectue régulièrement dans Sauvegarde Azure. Si nécessaire, testez la restauration du contenu sur un réseau virtuel ou abonnement isolé. Testez également la restauration des clés gérées par le client sauvegardées.

Si vous utilisez Azure Disk Encryption, vous pouvez restaurer la machine virtuelle Azure à l’aide des clés de chiffrement du disque. Lorsque vous utilisez le chiffrement de disque, vous pouvez restaurer la machine virtuelle Azure à l’aide des clés de chiffrement du disque.

- [Sauvegarde sur des machines virtuelles Azure utilisant le chiffrement](../backup/backup-azure-vms-encryption.md)
- [Guide pratique pour récupérer des fichiers à partir d’une sauvegarde de machine virtuelle Azure](../backup/backup-azure-restore-files-from-vm.md)
- [Guide pratique pour restaurer des clés de coffre de clés dans Azure](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)
- [Guide pratique pour sauvegarder et restaurer une machine virtuelle chiffrée](../backup/backup-azure-vms-encryption.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés gérées par le client
**Conseils :** Quand vous sauvegardez des disques managés avec Sauvegarde Azure, les machines sont chiffrées au repos avec SSE (Storage Service Encryption). Sauvegarde Azure peut également sauvegarder des machines virtuelles Azure chiffrées avec Azure Disk Encryption. Azure Disk Encryption s’intègre avec les clés de chiffrement de lecteur BitLocker qui sont sauvegardées dans un coffre de clés en tant que secrets. Azure Disk Encryption s’intègre également avec les clés de chiffrement Azure Key Vault. Activez la suppression réversible dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.

- [Suppression réversible pour les machines virtuelles](../backup/soft-delete-virtual-machines.md)
- [Vue d’ensemble de la suppression réversible d’Azure Key Vault](../key-vault/general/soft-delete-overview.md)

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

## <a name="incident-response"></a>Réponse aux incidents
*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents
**Conseils :** Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)
- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)
- [Utiliser le guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de votre propre plan de réponse aux incidents](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents
**Conseils :** Azure Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou à l’analytique utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez clairement les abonnements (par ex. production, non-production) à l’aide d’étiquettes et créez un système de nommage pour identifier et classer clairement les ressources Azure, en particulier celles qui traitent des données sensibles. Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

- [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md)
- [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md)

**Surveillance d’Azure Security Center :** Oui

**Responsabilité :** Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité
**Conseils :** Effectuez des exercices pour tester les capacités de réponse aux incidents de vos systèmes à intervalles réguliers, afin de protéger vos ressources Azure. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

- [Publication du NIST – Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité
**Conseils :** Microsoft utilisera les informations de contact pour le signalement d’incidents de sécurité pour vous contacter si le centre de réponse aux incidents de sécurité Microsoft (MSRC, Microsoft Security Response Center) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents
**Conseils :** Exportez vos alertes et recommandations Azure Security Center en utilisant la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel.

- [Comment configurer l’exportation continue](../security-center/continuous-export.md)
- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Customer

#### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité
**Conseils :** Utilisez la fonctionnalité d’automatisation de workflow d’Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » aux alertes et aux recommandations de sécurité afin de protéger vos ressources Azure.

- [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)
 
****Surveillance d’Azure Security Center :**** non applicable

**Responsabilité :** Customer


## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team
*Pour plus d’informations, consultez Contrôle de sécurité : [Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*


### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1 : Procédez régulièrement à des tests d’intrusion de vos ressources Azure et veillez à résoudre tous les problèmes de sécurité critiques détectés dans un délai de 60 jours
**Conseils :** Suivez les règles d’engagement de Microsoft pour garantir que vos tests d’intrusion sont conformes aux stratégies de Microsoft Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft.

- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)
- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Surveillance d’Azure Security Center :** Non applicable

**Responsabilité :** Partagé

## <a name="next-steps"></a>Étapes suivantes
Consultez l’article suivant :

- [Alertes de sécurité concernant les environnements dans Azure DevTest Labs](environment-security-alerts.md)