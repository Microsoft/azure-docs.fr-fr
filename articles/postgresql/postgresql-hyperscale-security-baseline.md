---
title: Base de référence de sécurité Azure pour Azure Database pour PostgreSQL – Hyperscale (Citus)
description: La base de référence de sécurité Azure Database pour PostgreSQL – Hyperscale (Citus) propose des instructions procédurales et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: postgresql
ms.topic: conceptual
ms.date: 08/04/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 836cf4600e353da61673ec8ee5df8f30fb4b9562
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97629848"
---
# <a name="azure-security-baseline-for-azure-database-for-postgresql---hyperscale-citus"></a>Base de référence de sécurité Azure pour Azure Database pour PostgreSQL – Hyperscale (Citus)

La base de référence de sécurité Azure pour Azure Database pour PostgreSQL – Hyperscale (Citus) contient des recommandations visant à améliorer l’état de la sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Aide** : Le pare-feu du serveur Azure Database pour PostgreSQL empêche tout accès à votre nœud coordinateur Hyperscale (Citus) tant que vous n’avez pas spécifié les ordinateurs qui disposent d’autorisations. Le pare-feu octroie l’accès au serveur en fonction de l’adresse IP d’origine de chaque demande. Pour configurer votre pare-feu, vous créez des règles de pare-feu qui spécifient les plages d’adresses IP acceptables. Vous pouvez créer des règles de pare-feu au niveau du serveur.

- [Guide pratique pour configurer les règles de pare-feu dans Azure Database pour PostgreSQL – Hyperscale (Citus)](./concepts-hyperscale-firewall-rules.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour les paramètres réseau et les ressources réseau associées à vos instances Azure Database pour PostgreSQL avec Azure Policy. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.Network » pour créer des stratégies personnalisées permettant d’auditer ou d’appliquer la configuration réseau de vos instances Azure Database pour PostgreSQL.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Exemples Azure Policy pour le réseau](../governance/policy/samples/built-in-policies.md#network)

- [Guide pratique pour créer un blueprint Azure](../governance/blueprints/create-blueprint-portal.md)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Contrôle de sécurité : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide** : Pour la journalisation d’audit du plan de contrôle, activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux vers un espace de travail Log Analytics, Event Hub ou un compte de stockage Azure pour archivage. À l’aide des données des journaux d’activité Azure, vous pouvez déterminer les éléments « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de contrôle pour vos ressources Azure.

Ingérez également les journaux au moyen d’Azure Monitor pour agréger les données de sécurité générées par Hyperscale (Citus). Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, puis utilisez les comptes de stockage pour le stockage à long terme/d’archivage. Vous pouvez également activer et intégrer des données dans Azure Sentinel ou une solution SIEM (Security Information and Event Management) tierce. 

- [Activation des paramètres de diagnostic pour le journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Métriques disponibles dans Hyperscale (Citus)](./concepts-hyperscale-monitoring.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Aide** : Hyperscale (Citus) fournit des mesures pour chaque nœud dans un groupe de serveurs. Les mesures permettent de mieux comprendre le comportement des ressources de prise en charge. Chaque métrique est émise selon une fréquence d’une minute et est conservée jusqu’à 30 jours dans l’historique.

Pour la journalisation d’audit du plan de contrôle, activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux vers un espace de travail Log Analytics, Event Hub ou un compte de stockage Azure pour archivage. À l’aide des données des journaux d’activité Azure, vous pouvez déterminer les éléments « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de contrôle pour vos ressources Azure.

Ingérez également les journaux au moyen d’Azure Monitor pour agréger les données de sécurité générées par Hyperscale (Citus). Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, puis utilisez les comptes de stockage pour le stockage à long terme/d’archivage. Vous pouvez également activer et intégrer des données dans Azure Sentinel ou une solution SIEM (Security Information and Event Management) tierce. 

- [Métriques disponibles dans Hyperscale (Citus)](./concepts-hyperscale-monitoring.md)

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Dans Azure Monitor, concernant l’espace de travail Log Analytics utilisé pour stocker vos journaux Hyperscale (Citus), définissez la période de rétention en fonction des réglementations de conformité de votre organisation. Utilisez les comptes de stockage Azure pour le stockage à long terme/d’archivage.

- [Définir les paramètres de conservation des journaux pour les espaces de travail Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

- [Stockage des journaux des ressources dans un compte de stockage Azure](../azure-monitor/platform/resource-logs.md#send-to-azure-storage)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Analysez et supervisez les journaux de vos instances Hyperscale (Citus) pour détecter tout comportement anormal. Utilisez Log Analytics d’Azure Monitor pour examiner les journaux et effectuer des requêtes sur leurs données. Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [En savoir plus sur Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md)

- [Guide pratique pour effectuer des requêtes personnalisées dans Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Aide** : Il est possible d’activer les paramètres de diagnostic pour Hyperscale (Citus) et d’envoyer les journaux à un espace de travail Log Analytics. Vous pouvez configurer et recevoir une alerte basée sur les métriques de supervision de vos services Azure. Utilisez Log Analytics d’Azure Monitor pour examiner les journaux et effectuer des requêtes sur leurs données. Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.

Intégrez votre espace de travail Log Analytics à Azure Sentinel, car cela fournit une solution de réponse automatisée d’orchestration de sécurité. Cela permet de créer des playbooks (solutions automatisées) utilisables pour corriger des problèmes de sécurité.

- [Métriques disponibles dans Hyperscale (Citus)](./howto-hyperscale-alert-on-metric.md)

- [Guide pratique pour configurer les paramètres de diagnostic pour le journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide** : Conservez un inventaire des comptes d’utilisateur possédant un accès d’administration au plan de contrôle (par exemple, le Portail Azure) de vos ressources Hyperscale (Citus), ainsi que des comptes d’administration ayant accès au plan de données (dans la base de données elle-même) de vos instances Hyperscale (Citus).

Hyperscale (Citus) ne prend pas en charge le contrôle d’accès en fonction du rôle intégré, mais il est possible de créer des rôles personnalisés basés sur des opérations précises du fournisseur de ressources.

Par ailleurs, le moteur PostgreSQL utilise les rôles pour contrôler l’accès aux objets de base de données. Or, un nouveau groupe de serveurs Hyperscale (Citus) s’accompagne de plusieurs rôles prédéfinis. Pour modifier des privilèges d’utilisateur, utilisez des commandes PostgreSQL standard, à l’aide d’un outil tel que PgAdmin ou psql.

- [Présentation des rôles personnalisés pour un abonnement Azure](../role-based-access-control/custom-roles.md) 

- [Présentation des opérations du fournisseur de ressources Azure Database pour PostgreSQL](../role-based-access-control/resource-provider-operations.md#microsoftdbforpostgresql) 

- [Présentation de la gestion des accès avec Azure Database pour PostgreSQL](./concepts-security.md#access-management)

- [Guide pratique pour créer des utilisateurs dans Azure Database pour PostgreSQL – Hyperscale (Citus)](./howto-hyperscale-create-users.md)

- [Guide pratique pour se connecter à PostgreSQL – Hyperscale (Citus) avec psql](./quickstart-create-hyperscale-portal.md#connect-to-the-database-using-psql)


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : Azure AD n’intègre pas le concept des mots de passe par défaut. Selon le service, d’autres ressources Azure qui exigent un mot de passe forcent la création d’un mot de passe conforme à des exigences de complexité et d’une longueur minimale. Vous êtes responsable des applications tierces et des services de la place de marché susceptibles d’utiliser des mots de passe par défaut.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures opérationnelles standard autour de l’utilisation des comptes d’administration dédiés utilisés pour accéder à vos instances Hyperscale (Citus). Les comptes administrateur de gestion de la ressource Azure sont liés à Azure Active Directory. Il existe également des comptes administrateur du serveur local au sein du groupe de serveurs Hyperscale (Citus) pour la gestion des autorisations d’accès à la base de données. Utilisez la gestion des identités et des accès Azure Security Center pour superviser le nombre de comptes d’administration d’Azure Active Directory.

- [Présentation de l’identité et de l’accès Azure Security Center](../security-center/security-center-identity-access.md) 

- [Guide pratique pour créer des utilisateurs dans Azure Database pour PostgreSQL – Hyperscale (Citus)](./howto-hyperscale-create-users.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Aide** : Pour accéder au portail Azure, activez l’authentification multifacteur (MFA) Azure Active Directory et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Aide** : Utilisez des stations de travail disposant d’un accès privilégié avec Multi-Factor Authentication (MFA) configuré pour se connecter aux ressources Azure et les configurer.

- [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-alert-on-account-login-behavior-deviation"></a>3.7 : Alerte en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez Azure Active Directory (AD) Privileged Identity Management pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.

Utilisez les détections de risque Azure AD pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs.

- [Déploiement de Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

- [Présentation des détections de risques Azure AD](../active-directory/identity-protection/overview-identity-protection.md)


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Utilisez des emplacements nommés à accès conditionnel pour autoriser l’accès au portail et à Azure Resource Manager seulement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

- [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Utilisez Azure Active Directory (AD) en tant que système central d’authentification et d’autorisation pour gérer les ressources PostgreSQL. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

Les utilisateurs au sein d’un groupe de serveurs Hyperscale (Citus) ne peuvent pas être liés directement à des comptes Azure Active Directory. Pour modifier les privilèges des utilisateurs relatifs à l’accès aux objets de la base de données, utilisez des commandes PostgreSQL standard à l’aide d’outils tels que PgAdmin ou psql.

- [Modifier les privilèges des rôles utilisateur](./howto-hyperscale-create-users.md#how-to-modify-privileges-for-user-role)

- [Guide pratique pour créer et configurer une instance AAD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Examinez et rapprochez l’accès des utilisateurs qui ont accès à la base de données locale et de ceux qui ont accès à Azure Active Directory pour gérer les ressources PostgreSQL.

Pour les utilisateurs ayant accès à la gestion des ressources Azure de la base de données, examinez les journaux d’Azure Active Directory (AD) pour identifier les comptes obsolètes. Utilisez également des révisions d’accès d’identité Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise susceptibles de servir à accéder à Hyperscale (Citus) et les attributions de rôles. Il convient d’examiner régulièrement les accès des utilisateurs, par exemple, tous les 90 jours, pour vérifier que seuls les utilisateurs appropriés sont autorisés à accéder.

- [Examen des utilisateurs PostgreSQL et des rôles attribués](https://www.postgresql.org/docs/current/database-roles.html)

- [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml)

- [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Aide** : Dans Azure Active Directory (AD), vous avez accès aux sources des journaux Activité de connexion, Audit et Événement à risque Azure AD, qui permettent l’intégration à n’importe quel outil SIEM/de supervision. 

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure Active Directory et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes souhaitées dans un espace de travail Log Analytics. 

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12 : Alerter en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez la détection de risque et la fonctionnalité Identity Protection d’Azure Active Directory pour configurer des réponses automatiques aux actions suspectes détectées au niveau d’Azure Active Directory (AD). Vous pouvez activer des réponses automatisées par le biais d’Azure Sentinel pour implémenter les réponses de sécurité de votre organisation.

Vous pouvez aussi ingérer des journaux dans Azure Sentinel pour approfondir votre examen.

- [Vue d’ensemble d’Azure AD Identity Protection](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Aide** : Non disponible à l’heure actuelle ; Customer Lockbox n’est pas encore pris en charge pour Hyperscale (Citus).

- [Liste des services pris en charge pour Customer Lockbox](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : actuellement non disponible

## <a name="data-protection"></a>Protection de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Aide** : Utilisez des étiquettes pour faciliter le suivi des instances Hyperscale (Citus) et des ressources associées qui stockent ou traitent des informations sensibles.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Utilisez une combinaison de rôles d’administrateur et de règles de pare-feu pour isoler et limiter l’accès réseau à vos instances Azure Database pour PostgreSQL.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Présentation des règles de pare-feu dans Azure Database pour PostgreSQL – Hyperscale (Citus)](./concepts-hyperscale-firewall-rules.md)

- [Présentation des rôles dans Hyperscale (Citus)](./howto-hyperscale-create-users.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Aide** : La version 1.2 du protocole TLS (Transport Layer Security) est nécessaire pour les connexions des applications clientes au nœud coordinateur Hyperscale (Citus). L’application de connexions TLS entre votre serveur de base de données et vos applications clientes vous protège contre les « attaques de l’intercepteur » en chiffrant le flux de données entre le serveur et votre application

Pour tous les serveurs Azure Database pour PostgreSQL provisionnés par le biais du portail Azure, l’application de connexions TLS est activée par défaut.

Dans certains cas, les applications tierces ont besoin d’un fichier de certificat local généré à partir du fichier de certificat (.cer) d’une autorité de certification de confiance pour se connecter en toute sécurité.

- [Guide pratique pour configurer TLS dans Azure Database pour PostgreSQL – Hyperscale (Citus)](./concepts-hyperscale-ssl-connection-security.md)

- [Applications imposant la vérification du certificat pour la connectivité TLS](./concepts-hyperscale-ssl-connection-security.md)



**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Aide** : Utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour contrôler l’accès au plan de contrôle Hyperscale (Citus) [par exemple, Portail Azure]. Azure RBAC ne modifie pas les autorisations utilisateur dans la base de données.

Pour modifier les privilèges des utilisateurs au niveau de la base de données, utilisez des commandes PostgreSQL standard à l’aide d’un outil comme PgAdmin ou psql.

- [Comment configurer Azure RBAC](../role-based-access-control/role-assignments-portal.md)

- [Guide pratique pour configurer l’accès utilisateur avec SQL pour Azure Database pour PostgreSQL](./howto-hyperscale-create-users.md)


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide** :  
Au moins une fois par jour, Azure Database pour PostgreSQL – Hyperscale (Citus) effectue des sauvegardes instantanées des fichiers de données et du journal des transactions de base de données. Les sauvegardes vous permettent de restaurer un serveur à n’importe quel point dans le temps au sein de la période de conservation. (La période de conservation est de 35 jours pour tous les clusters.) Toutes les sauvegardes sont chiffrées à l’aide du chiffrement AES de 256 bits. L’offre PostgreSQL Hyperscale (Citus) utilise des clés gérées par Microsoft pour le chiffrement.

- [Présentation du chiffrement pour les sauvegardes Azure PostgreSQL – Hyperscale (Citus)](./concepts-hyperscale-backup.md)



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes au cas où des modifications se produisent sur des instances de production d’Hyperscale (Citus) et d’autres ressources critiques ou associées.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Aide** : Non disponible à l’heure actuelle ; Azure Security Center ne prend pas encore en charge l’évaluation des vulnérabilités avec Azure Database pour PostgreSQL – Hyperscale (Citus).

- [Couverture des fonctionnalités pour les services PaaS Azure dans Azure Security Center](../security-center/features-paas.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : actuellement non disponible

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Aide** : Utilisez Azure Resource Graph pour interroger et découvrir toutes les ressources (y compris les instances Hyperscale (Citus)) au sein de vos abonnements. Vérifiez que vous disposez des autorisations (en lecture) appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent.

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription)

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Aide** : Appliquez des étiquettes aux instances Hyperscale (Citus) et à d’autres ressources associées en ajoutant des métadonnées pour les organiser logiquement en une taxonomie.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Aide** : Utilisez des étiquettes, des groupes d’administration et des abonnements distincts selon les besoins pour organiser et effectuer le suivi des instances Hyperscale (Citus) et les ressources associées. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Instructions** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant une stratégie Azure avec les définitions intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

Utilisez également Azure Resource Graph pour interroger/découvrir des ressources dans les abonnements.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Instructions** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant une stratégie Azure avec les définitions intégrées suivantes :

- Types de ressources non autorisés
- Types de ressources autorisés

Utilisez également Azure Resource Graph pour interroger/découvrir des ressources dans les abonnements.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Instructions** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant une stratégie Azure avec les définitions intégrées suivantes :

- Types de ressources non autorisés
- Types de ressources autorisés

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ». Cette configuration est susceptible d’empêcher la création et la modification des ressources dans un environnement haute sécurité, comme des instances Hyperscale (Citus) contenant des informations sensibles.

- [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Définissez et implémentez des configurations standard de la sécurité pour vos instances Hyperscale (Citus) avec Azure Policy. Utilisez Azure Policy pour créer des stratégies personnalisées permettant d’auditer ou d’appliquer la configuration réseau de vos instances Azure Database pour PostgreSQL.

Par ailleurs, Azure Resource Manager a la possibilité d’exporter le modèle au format JSON (JavaScript Object Notation), qui doit être examiné pour vérifier que les configurations répondent/dépassent les exigences de sécurité de votre organisation. 

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Exportation monoressource ou multiressource vers un modèle sur le portail Azure](../azure-resource-manager/templates/export-template-portal.md) 



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : Utilisez Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.  En outre, vous pouvez utiliser des modèles Azure Resource Manager pour appliquer la configuration de sécurité des ressources Azure requise par votre organisation. 

- [Comprendre les effets d'Azure Policy](../governance/policy/concepts/effects.md)

- [Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

- [Présentation des modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md)



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide** : Si vous avez recours à des définitions Azure Policy personnalisées pour vos instances Hyperscale (Citus) et les ressources associées, utilisez Azure Repos pour stocker et gérer votre code de façon sécurisée.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops&preserve-view=true)

- [Documentation Azure Repos](/azure/devops/repos/index?view=azure-devops&preserve-view=true)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : Utilisez Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.  En outre, vous pouvez utiliser des modèles Azure Resource Manager pour appliquer la configuration de sécurité des ressources Azure requise par votre organisation. 

- [Comprendre les effets d'Azure Policy](../governance/policy/concepts/effects.md)

- [Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

- [Présentation des modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md)



**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Aide** : Utiliser des alias Azure Policy dans l’espace de noms « Microsoft.DBforPostgreSQL » pour créer des stratégies personnalisées afin d’alerter, d’auditer et d’appliquer des configurations système. Utilisez une stratégie Azure [auditer], [refuser] et [déployer si elle n’existe pas] pour appliquer automatiquement des configurations pour vos instances Azure Database pour PostgreSQL et les ressources associées.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Aide** : Azure Database pour PostgreSQL – Hyperscale (Citus) ne prend actuellement pas directement en charge les identités managées. Quand vous créez un serveur Azure Database pour PostgreSQL, vous devez fournir les informations d’identification d’un utilisateur administrateur. Vous pouvez créer des rôles utilisateur supplémentaires dans l’interface du portail Azure.

- [Créer un serveur Azure Database pour PostgreSQL – Hyperscale (Citus)](./quickstart-create-hyperscale-portal.md#create-a-hyperscale-citus-server-group)

- [Créer des rôles utilisateur supplémentaires](./howto-hyperscale-create-users.md#how-to-create-additional-user-roles)


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : actuellement non disponible

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Aide** : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Hyperscale (Citus)), mais ne s’exécute pas sur du contenu client.

Pré-analysez tout contenu chargé sur des ressources Azure non liées au calcul, comme App Service, Data Lake Storage, Stockage Blob, Azure Database pour PostgreSQL, etc. Microsoft ne peut pas accéder à vos données dans ces instances.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-recovery"></a>Récupération de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Aide** : Azure Database pour PostgreSQL – Hyperscale (Citus) crée automatiquement des sauvegardes de chaque nœud et les conserve dans un stockage localement redondant. Les sauvegardes peuvent être utilisées pour restaurer votre cluster Hyperscale (Citus) à une heure spécifiée.

- [Guide pratique pour effectuer une sauvegarde et restauration dans Azure Database pour PostgreSQL – Hyperscale (Citus)](./concepts-hyperscale-backup.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Aide** : Au moins une fois par jour, Azure Database pour PostgreSQL effectue des sauvegardes instantanées des fichiers de données et du journal des transactions de base de données. Les sauvegardes vous permettent de restaurer un serveur à n’importe quel point dans le temps au sein de la période de conservation. La période de rétention est actuellement de 35 jours pour tous les clusters. Toutes les sauvegardes sont chiffrées à l’aide du chiffrement AES de 256 bits.

Dans les régions Azure qui prennent en charge les zones de disponibilité, les instantanés de sauvegarde sont stockés dans trois zones de disponibilité. Tant qu’au moins une zone de disponibilité est en ligne, le cluster Hyperscale (Citus) peut être restauré.

- [Guide pratique pour effectuer une sauvegarde et restauration dans Azure Database pour PostgreSQL – Hyperscale (Citus)](./concepts-hyperscale-backup.md)


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Aide** : Dans Azure Database pour PostgreSQL, la restauration d’un cluster Hyperscale (Citus) crée un cluster à partir des sauvegardes des nœuds d’origine. Vous pouvez restaurer un cluster à n’importe quel point dans le temps au cours des 35 derniers jours. Le processus de restauration crée un cluster dans les mêmes région, abonnement et groupe de ressources Azure que le cluster d’origine. La configuration du nouveau cluster est identique à celle du cluster d’origine : même nombre de nœuds, même nombre de vCore, même taille de stockage et mêmes rôles utilisateur.

Ni les paramètres de pare-feu ni les paramètres du serveur PostgreSQL ne sont conservés à partir du groupe de serveurs d’origine ; ils sont réinitialisés aux valeurs par défaut. Le pare-feu empêche toute connexion. Vous devez ajuster manuellement ces paramètres après la restauration.

- [Guide pratique pour effectuer une sauvegarde et restauration dans Azure Database pour PostgreSQL – Hyperscale (Citus)](./concepts-hyperscale-backup.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés gérées par le client

**Aide** : Les clusters Hyperscale (Citus) supprimés ne peuvent pas être restaurés. Si vous supprimez le cluster, tous les nœuds qui appartiennent au cluster sont supprimés et ne peuvent pas être récupérés. Pour protéger les ressources du cluster contre une suppression accidentelle ou des modifications inattendues à l’issue du déploiement, les administrateurs peuvent utiliser des verrous de gestion.

- [Guide pratique pour effectuer une sauvegarde et restauration dans Azure Database pour PostgreSQL – Hyperscale (Citus)](./concepts-hyperscale-backup.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé. 

- [Comment configurer des automatisations de workflow dans Azure Security Center](../security-center/security-center-planning-and-operations-guide.md) 

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/) 

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/) 

- [Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte. 

En outre, marquez clairement les abonnements (par ex. production, non production) et créez un système d’attribution de noms pour identifier et classer les ressources Azure de façon claire.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins. 

- [Reportez-vous à la publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Aide** : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé.  Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus. 

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Oui

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

*Pour plus d’informations, consultez [Contrôle de sécurité : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de Microsoft pour garantir que vos tests d’intrusion sont conformes aux stratégies de Microsoft : https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1 

- [Vous trouverez ici plus d’informations sur la stratégie de Microsoft, sur l’exécution de Red Teaming et sur les tests d’intrusion de site actif dans l’infrastructure, les services et les applications cloud gérés par Microsoft.](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)