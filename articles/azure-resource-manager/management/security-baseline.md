---
title: Base de référence de la sécurité Azure pour Azure Resource Manager
description: La base de référence de sécurité Azure Resource Manager fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 10/30/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: d841d3b922a5edd0257934566207187ae031c1b5
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532174"
---
# <a name="azure-security-baseline-for-azure-resource-manager"></a>Base de référence de la sécurité Azure pour Azure Resource Manager

Cette ligne de base de sécurité applique les instructions du [benchmark de sécurité Azure version 1.0](../../security/benchmarks/overview-v1.md) à Azure Resource Manager. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure.
Le contenu est regroupé par les **contrôles de sécurité** définis par le benchmark de sécurité Azure et les conseils associés applicables à Azure Resource Manager. Les **contrôles** non applicables à Azure Resource Manager ont été exclus.

 Pour voir comment Azure Resource Manager est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de la sécurité Azure Resource Manager](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Journalisation et supervision](../../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : Ingérez des journaux d’activité Azure Policy via Azure Monitor. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, puis utilisez des comptes Stockage Azure pour le stockage à long terme ou d’archivage. Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce.

- [Guide pratique pour intégrer Azure Sentinel](../../sentinel/quickstart-onboard.md) 

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../../azure-monitor/platform/diagnostic-settings.md) 

- [Guide pratique pour collecter les journaux des hôtes internes des machines virtuelles Azure avec Azure Monitor](../../azure-monitor/learn/quick-collect-azurevm.md) 

- [Guide pratique pour bien démarrer avec Azure Monitor et l’intégration d’une solution SIEM tierce](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Azure Resource Manager utilise les journaux d’activité, qui sont automatiquement activés, pour inclure la source de l’événement, la date, l’utilisateur, le timestamp, les adresses sources, les adresses de destination et d’autres éléments utiles.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../../azure-monitor/platform/diagnostic-settings.md) 

- [Présentation de la journalisation et des différents types de journaux dans Azure](../../azure-monitor/platform/platform-logs-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide** : Analysez et supervisez les journaux pour détecter les comportements anormaux et examinez régulièrement les résultats. Utilisez Azure Monitor et un espace de travail Log Analytics pour examiner les journaux et effectuer des requêtes sur leurs données. 

Vous pouvez également activer et intégrer les données dans Azure Sentinel ou une solution SIEM tierce. 

- [Guide pratique pour intégrer Azure Sentinel](../../sentinel/quickstart-onboard.md) 

- [Bien démarrer avec les requêtes Log Analytics](../../azure-monitor/log-query/log-analytics-tutorial.md) 

- [Guide pratique pour effectuer des requêtes personnalisées dans Azure Monitor](../../azure-monitor/log-query/get-started-queries.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils** : Utilisez Azure Security Center avec Log Analytics pour superviser les activités anormales détectées dans les journaux d’activité et générer des alertes s’y rapportant. Vous pouvez également activer et intégrer les données dans Azure Sentinel. 

- [Guide pratique pour intégrer Azure Sentinel](../../sentinel/quickstart-onboard.md) 

- [Guide pratique pour gérer les alertes dans Azure Security Center](../../security-center/security-center-managing-and-responding-alerts.md) 

- [Guide pratique pour générer une alerte sur des données de journal Log Analytics](../../azure-monitor/learn/tutorial-response.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : Le contrôle d’accès en fonction du rôle Azure (RBAC) vous permet de gérer l’accès aux ressources Azure par le biais d’attributions de rôles. Vous pouvez affecter ces rôles à des utilisateurs et regrouper des principaux de service et des identités managées. Les rôles intégrés prédéfinis pour certaines ressources peuvent être inventoriés ou interrogés par le biais d’outils tels qu’Azure CLI, Azure PowerShell ou le portail Azure.

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?preserve-view=true&view=azureadps-2.0)

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?preserve-view=true&view=azureadps-2.0)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

En outre, pour vous aider à suivre les comptes d’administration dédiés, vous pouvez utiliser des recommandations d’Azure Security Center ou des stratégies Azure intégrées, telles que :

- Plusieurs propriétaires doivent être affectés à votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

Vous pouvez également activer l’accès juste-à-temps en utilisant Azure AD Privileged Identity Management et Azure Resource Manager. 

- [En savoir plus sur Privileged Identity Management](../../active-directory/privileged-identity-management/index.yml)

- [Utilisation d’Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : Dans la mesure du possible, utilisez l’authentification unique (SSO) d’Azure Active Directory au lieu de configurer des informations d’identification autonomes individuelles par service. Suivez les suggestions d’Azure Security Center liées à la gestion des identités et des accès. 

- [Présentation de l’authentification SSO avec Azure AD](../../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Aide** : Activer l’authentification multifacteur Azure Active Directory et suivre les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

- [Guide pratique pour activer l’authentification MFA dans Azure](../../active-directory/authentication/howto-mfa-getstarted.md)

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Aide** : Utilisez une station de travail sécurisée et managée par Azure (également appelée station de travail à accès privilégié) pour les tâches administratives qui requièrent des privilèges élevés.

- [Comprendre les stations de travail sécurisées gérées par Azure](../../active-directory/devices/concept-azure-managed-workstation.md)

- [Procédure d’activation d’Azure AD MFA](../../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils** : Utilisez les rapports de sécurité Azure Active Directory et la supervision pour détecter les activités suspectes ou potentiellement dangereuses qui se produisent dans l’environnement. Utiliser Azure Security Center pour superviser les activités liées aux identités et aux accès.

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-only-from-approved-locations"></a>3.8 : Gérer les ressources Azure uniquement à partir d’emplacements approuvés

**Aide** : Utilisez des emplacements nommés Azure AD pour n’autoriser l’accès qu’à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

- [Procédure de configuration des emplacements nommés Azure AD](../../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Utilisez des emplacements nommés Azure AD pour n’autoriser l’accès qu’à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

- [Procédure de configuration des emplacements nommés Azure AD](../../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Conseils** : Azure AD fournit des journaux pour vous aider à découvrir les comptes obsolètes. Par ailleurs, utilisez les révisions d’accès et des identités Azure AD pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seuls les utilisateurs appropriés continuent de bénéficier d’un accès. 

- [Présentation des rapports Azure AD](../../active-directory/reports-monitoring/index.yml)

- [Utilisation des révisions d’accès et des identités Azure AD](../../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Aide** : Vous avez accès aux activités de connexion Azure AD, aux sources des journaux d’événements à risque et d’audit, ce qui vous permet de les intégrer à un outil SIEM/de supervision.

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure AD et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes souhaitées dans un espace de travail Log Analytics.

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Aide** : Utilisez les fonctionnalités d’Azure AD Identity Protection pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. Vous pouvez également ingérer des données dans Azure Sentinel pour approfondir votre examen.

- [Guide pratique pour afficher les connexions risquées Azure AD](../../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Guide pratique pour intégrer Azure Sentinel](../../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Protection des données](../../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

- [Guide pratique pour créer et utiliser des étiquettes](./tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="46-use-azure-rbac-to-manage-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : Utilisez Azure AD RBAC pour contrôler l’accès aux données et aux ressources, sinon utilisez des méthodes de contrôle d’accès spécifiques au service.

- [Guide pratique pour configurer le contrôle RBAC dans Azure](../../role-based-access-control/role-assignments-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : Pour le chiffrement au repos côté serveur, Azure Resource Manager prend en charge les clés gérées par Microsoft.

- [Présentation de la protection des données dans Azure Resource Manager](azure-resource-manager-security-controls.md#data-protection)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes lorsque des modifications sont apportées à des ressources Azure critiques.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : Azure Resource Graph permet d’interroger et de découvrir toutes les ressources (telles que le calcul, le stockage, le réseau, les ports, les protocoles et ainsi de suite) dans vos abonnements. Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements.

Même si les ressources Azure classiques peuvent être découvertes via l’explorateur Azure Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager.

- [Guide pratique pour créer des requêtes avec Azure Resource Graph](../../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?preserve-view=true&view=azps-4.8.0)

- [Présentation d’Azure RBAC](../../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Utilisez les paramètres Nom de la stratégie, Description et Catégorie pour organiser logiquement les ressources en fonction d’une taxonomie.

- [Pour plus d’informations sur l’étiquetage des ressources, consultez Guides de décision concernant le nommage et l’étiquetage des ressources](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=%2fazure%2fazure-resource-manager%2fmanagement%2ftoc.json)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Aide** : Utilisez des étiquettes, des groupes d’administration et des abonnements distincts, le cas échéant, pour organiser et suivre les ressources Azure. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

En outre, utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients à l’aide des définitions de stratégie intégrées suivantes :

- Types de ressources non autorisés
- Types de ressources autorisés

Des informations complémentaires sont fournies ci-dessous.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](./tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4 : Dresser et tenir un inventaire des ressources Azure approuvées

**Conseils** : Créez un inventaire des ressources Azure et logiciels approuvés pour les ressources de calcul en fonction des besoins de votre organisation.

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Aide** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements.

Utilisez Azure Resource Graph pour interroger et découvrir des ressources dans leurs abonnements.  Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Resource Graph](../../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients selon les définitions de stratégies intégrées suivantes :

- Types de ressources non autorisés
- Types de ressources autorisés

Des informations complémentaires sont fournies ci-dessous.

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../../governance/policy/samples/built-in-policies.md#general)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Conseils** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

- [Guide pratique pour configurer l’accès conditionnel de façon à bloquer l’accès à Azure Resource Manager](../../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-in-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide** : Utilisez des alias Azure Policy pour créer des stratégies personnalisées afin d’auditer ou d’appliquer la configuration de vos ressources Azure. Vous pouvez aussi utiliser des définitions Azure Policy intégrées.

Azure Resource Manager est capable d’exporter le modèle au format JSON (JavaScript Object Notation), qui doit être examiné pour vérifier que les configurations répondent aux exigences de sécurité de votre organisation.

Vous pouvez aussi utiliser les recommandations d’Azure Security Center comme ligne de base de configuration sécurisée pour vos ressources Azure.

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?preserve-view=true&view=azps-4.8.0)

- [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../../governance/policy/tutorials/create-and-manage.md)

- [Exportation monoressource ou multiressource vers un modèle sur le portail Azure](../templates/export-template-portal.md)

- [Recommandations de sécurité - Guide de référence](../../security-center/recommendations-reference.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide** : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.  En outre, vous pouvez utiliser des modèles Azure Resource Manager pour appliquer la configuration de sécurité des ressources Azure requise par votre organisation. 

En outre, en tant qu’administrateur, vous pouvez avoir besoin de verrouiller un abonnement, une ressource ou un groupe de ressources afin d’empêcher d’autres utilisateurs de votre organisation de supprimer ou modifier de manière accidentelle des ressources critiques. Vous pouvez définir le niveau de verrouillage sur CanNotDelete ou ReadOnly.

- [Comprendre les effets d'Azure Policy](../../governance/policy/concepts/effects.md)

- [Créer et gérer des stratégies pour assurer la conformité](../../governance/policy/tutorials/create-and-manage.md)

- [Présentation des modèles Azure Resource Manager](../templates/overview.md)

- [Guide pratique pour verrouiller des ressources pour empêcher des modifications inattendues](lock-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Conseils** : Utilisez Azure DevOps pour stocker et gérer de manière sécurisée votre code, comme les définitions de stratégies Azure personnalisées, les modèles Azure Resource Manager et les scripts Desired State Configuration. Pour accéder aux ressources que vous gérez dans Azure DevOps, vous pouvez accorder ou refuser des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps, ou à Active Directory s’il est intégré à TFS.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?preserve-view=true&view=azure-devops)

- [À propos des autorisations et des groupes dans Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour les ressources Azure à l’aide d’Azure Policy. Utilisez des alias Azure Policy pour créer des stratégies personnalisées afin d’auditer ou d’appliquer la configuration réseau de vos ressources Azure. Vous pouvez également utiliser des définitions de stratégie intégrées en lien avec vos ressources spécifiques.  Par ailleurs, vous pouvez utiliser Azure Automation pour déployer les changements de configuration.

- [Configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Utiliser des alias](../../governance/policy/concepts/definition-structure.md#aliases)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Conseils** : Utilisez des définitions Azure Policy intégrées ainsi que des stratégies personnalisées pour créer des alertes, auditer et appliquer des configurations système. Utilisez une stratégie Azure Policy [auditer], [refuser] et [déployer s’il n’existe pas] afin d’appliquer automatiquement des configurations pour vos ressources Azure.

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Utilisez les pratiques recommandées lors de la construction de votre modèle Resource Manager. Celles-ci vous aident à éviter des problèmes qui se produisent couramment en cas d’utilisation d’un modèle Resource Manager pour déployer une solution.

Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault. 

- [Implémenter les meilleures pratiques de sécurité des modèles](../templates/template-best-practices.md)

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-recovery"></a>Récupération de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Récupération de données](../../security/benchmarks/security-control-data-recovery.md).*

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : Veillez à pouvoir effectuer régulièrement le déploiement des modèles Azure Resource Manager dans un abonnement isolé, le cas échéant.

- [Déploiement de ressources avec des modèles ARM et le Portail Azure](../templates/deploy-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés managées par le client

**Conseils** : Utilisez Azure DevOps pour stocker et gérer de manière sécurisée votre code, comme les définitions de stratégies Azure personnalisées, les modèles Azure Resource Manager et les scripts Desired State Configuration. Pour accéder aux ressources que vous gérez dans Azure DevOps, vous pouvez accorder ou refuser des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps, ou à Active Directory s’il est intégré à TFS.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?preserve-view=true&view=azure-devops)

- [À propos des autorisations et des groupes dans Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Aide** : Développez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé. 

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/)

- [Utiliser le guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de votre propre plan de réponse aux incidents](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Effectuez des exercices pour tester les capacités de réponse aux incidents de vos systèmes à intervalles réguliers, afin de protéger vos ressources Azure. Identifiez les points faibles et les lacunes, puis révisez votre plan de réponse en fonction des besoins.

- [Publication du NIST--Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities(Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://csrc.nist.gov/publications/detail/sp/800-84/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations Azure Security Center en utilisant la fonctionnalité d’exportation continue pour identifier les risques pesant sur les ressources Azure. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel.

- [Comment configurer l’exportation continue](../../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation de workflow d’Azure Security Center pour déclencher automatiquement des réponses aux alertes et aux recommandations de sécurité afin de protéger vos ressources Azure.

- [Comment configurer l’automatisation du workflow dans Security Center](../../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Tests d’intrusion et exercices Red Team](../../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de pénétration du cloud Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft.

- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1)

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../../security/benchmarks/security-baselines-overview.md)