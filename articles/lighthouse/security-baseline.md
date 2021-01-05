---
title: Base de référence de sécurité Azure pour Azure Lighthouse
description: La base de référence de sécurité Azure Lighthouse fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le Benchmark de sécurité Azure.
author: msmbaldwin
ms.service: lighthouse
ms.topic: conceptual
ms.date: 11/19/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: a6a1b41dd4ede055ba81c069c2164bb36da99876
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602306"
---
# <a name="azure-security-baseline-for-azure-lighthouse"></a>Base de référence de sécurité Azure pour Azure Lighthouse

Cette base de référence de sécurité applique les conseils du [Benchmark de sécurité Azure version 2.0](../security/benchmarks/overview.md) à Azure Lighthouse. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Le contenu est regroupé par les **contrôles de sécurité** définis par le Benchmark de sécurité Azure et les conseils associés applicables à Azure Lighthouse. **Les contrôles** non applicables à Azure Lighthouse ont été exclus.

Pour voir comment Azure Lighthouse est entièrement mappé au Benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité Azure Lighthouse](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="identity-management"></a>Gestion des identités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des identités](../security/benchmarks/security-controls-v2-identity-management.md).*

### <a name="im-1-standardize-azure-active-directory-as-the-central-identity-and-authentication-system"></a>IM-1 : Normaliser Azure Active Directory comme système d’authentification et d’identité central

**Conseils** : Azure Lighthouse utilise Azure Active Directory (Azure AD) en tant que service de gestion des identités et des accès par défaut. Normalisez Azure AD pour régir la gestion des identités et des accès de votre organisation dans :
- Les ressources cloud Microsoft, comme le portail Azure, le stockage Azure, les machines virtuelles Azure (Linux et Windows), les applications Azure Key Vault, PaaS et SaaS.
- Les ressources de votre organisation, comme les applications sur Azure ou les ressources réseau de votre entreprise.

Avec Azure Lighthouse, les utilisateurs désignés d’un locataire gestionnaire disposent d’un rôle intégré Azure qui leur permet d’accéder à des abonnements délégués et/ou à des groupes de ressources dans le locataire d’un client. Tous les rôles intégrés sont actuellement pris en charge, à l’exception du propriétaire et des rôles intégrés avec l’autorisation DataActions. Le rôle Administrateur de l’accès utilisateur est pris en charge uniquement pour une utilisation limitée dans l’affectation de rôles à des identités gérées. Les rôles personnalisés et les Rôles Administrateur classique de l’abonnement ne sont pas pris en charge.

- [Locataires dans Azure Active Directory](../active-directory/develop/single-and-multi-tenant-apps.md) 

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md) 

- [Utilisez des fournisseurs d'identité externes pour l’application](../active-directory/external-identities/identity-providers.md) 

- [Qu’est-ce que le degré de sécurisation Identity Secure Score dans Azure Active Directory](../active-directory/fundamentals/identity-secure-score.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="im-2-manage-application-identities-securely-and-automatically"></a>IM-2 : Gérer les identités d’application de façon sécurisée et automatique

**Conseils** : Les identités gérées par Azure peuvent s’authentifier auprès des services et ressources Azure qui prennent en charge l’authentification Azure AD. L’authentification est activée à l’aide de règles d’octroi d’accès prédéfinies, évitant les informations d’identification codées en dur dans le code source ou les fichiers de configuration. Avec Azure Lighthouse, les utilisateurs dotés du rôle d’administrateur de l’accès utilisateur sur l’abonnement d’un client peuvent créer une identité managée dans le locataire de ce client. Bien que ce rôle ne soit généralement pas pris en charge avec Azure Lighthouse, il peut être utilisé dans ce scénario spécifique, ce qui permet aux utilisateurs disposant de cette autorisation d’affecter un ou plusieurs rôles intégrés spécifiques à des identités managées.

Pour les services qui ne prennent pas en charge les identités gérées, utilisez Azure AD pour créer un principal de service avec des autorisations restreintes au niveau de la ressource à la place. Azure Lighthouse permet aux principaux de service d’accéder aux ressources de client en fonction des rôles qu’ils ont accordés au cours du processus d’intégration. Il est recommandé de configurer des principaux de service avec des informations d’identification de certificat et de se replier sur les secrets des clients. Dans les deux cas, Azure Key Vault peut être utilisé conjointement avec des identités gérées par Azure, afin que l’environnement d’exécution (par exemple, une fonction Azure) puisse récupérer les informations d’identification du coffre de clés.

- [Identités managées Azure](../active-directory/managed-identities-azure-resources/overview.md)

- [Principal de service Azure](/powershell/azure/create-azure-service-principal-azureps)

- [Utiliser Azure Key Vault pour l’inscription du principal de sécurité](../key-vault/general/authentication.md#authorize-a-security-principal-to-access-key-vault)

- [Créer un utilisateur qui peut attribuer des rôles à une identité managée dans un locataire client](how-to/deploy-policy-remediation.md#create-a-user-who-can-assign-roles-to-a-managed-identity-in-the-customer-tenant)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-4-use-strong-authentication-controls-for-all-azure-active-directory-based-access"></a>IM-4 : Utiliser des contrôles d’authentification renforcés pour tous les accès basés sur Azure Active Directory

**Conseils** : exigez l’application de la solution Microsoft Azure Multi-Factor Authentication (MFA) à tous les utilisateurs de votre locataire gérant, notamment aux utilisateurs qui auront accès aux ressources déléguées du client. Nous vous suggérons de demander à vos clients d’implémenter également MFA dans leurs locataires.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-5-monitor-and-alert-on-account-anomalies"></a>IM-5 : Surveiller et alerter en cas d’anomalies de compte

**Conseils** : Azure AD fournit les sources de données suivantes : 

-   Connexions – le rapport de connexions fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.

-   Journaux d’audit : traçabilité proposée via des journaux d’activité pour toutes les modifications effectuées par diverses fonctionnalités au sein d’Azure AD. Les exemples de modifications journalisées dans les journaux d’audit incluent l’ajout ou la suppression d’utilisateurs, d’applications, de groupes, de rôles et de stratégies.

-   Connexions risquées : une connexion risquée est une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.

-   Utilisateurs avec indicateur de risque : un utilisateur à risque correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Ces sources de données peuvent être intégrées à Azure Monitor, à Azure Sentinel ou à des systèmes SIEM tiers.

Les fournisseurs de services qui utilisent Azure Lighthouse peuvent transférer les journaux Azure AD vers Azure Sentinel et afficher/définir des alertes sur les locataires afin de surveiller et de signaler les anomalies de compte.

- [Rapports d’activité d’audit dans Azure AD](../active-directory/reports-monitoring/concept-audit-logs.md)

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Gérer les espaces de travail Azure Sentinel à grande échelle](how-to/manage-sentinel-workspaces.md)

- [Connecter des données entre Azure AD et Azure Sentinel](../sentinel/connect-azure-active-directory.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-6-restrict-azure-resource-access-based-on-conditions"></a>IM-6 : Restreindre l’accès aux ressources Azure en fonction des conditions

**Conseils** : Azure Lighthouse ne prend pas en charge la fonctionnalité d’accès conditionnel pour les ressources de client déléguées. Dans le locataire gestionnaire, utilisez l’accès conditionnel Azure AD pour un contrôle d’accès plus granulaire en fonction des conditions définies par l’utilisateur, comme la nécessité de connecter des utilisateurs à partir de certaines plages d’adresses IP pour utiliser l’authentification multifacteur (MFA). Une gestion granulaire des sessions d’authentification peut également être utilisée via une stratégie d’accès conditionnel Azure AD pour différents cas d’usage. 

Vous devez exiger l’application de la solution MFA à tous les utilisateurs de votre locataire gérant, notamment aux utilisateurs qui auront accès aux ressources déléguées du client. Nous vous suggérons de demander à vos clients d’implémenter également MFA dans leurs locataires.

- [Présentation de l’accès conditionnel Azure](../active-directory/conditional-access/overview.md)

- [Stratégies d’accès conditionnel courantes](../active-directory/conditional-access/concept-conditional-access-policy-common.md)

- [Configurer la gestion de session d’authentification avec l’accès conditionnel](../active-directory/conditional-access/howto-conditional-access-session-lifetime.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="privileged-access"></a>Accès privilégié

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Accès privilégié](../security/benchmarks/security-controls-v2-privileged-access.md).*

### <a name="pa-1-protect-and-limit-highly-privileged-users"></a>PA-1 : Protéger et limiter les utilisateurs disposant de privilèges élevés

**Conseils** : Limitez le nombre de comptes d’utilisateur dotés de privilèges élevés et protégez ces comptes à un niveau élevé. Un compte d’administrateur général n’est pas nécessaire pour activer et utiliser Azure Lighthouse.

Pour accéder aux données du journal d’activité au niveau du locataire, un compte doit se voir attribuer le rôle intégré Lecteur d’analyse Azure au niveau de l’étendue racine (/). S’agissant d’un niveau d’accès étendu, nous vous recommandons d’attribuer ce rôle à un compte de principal de service plutôt qu’à un utilisateur individuel ou à un groupe. Cette affectation doit être effectuée par un utilisateur qui a le rôle Administrateur général avec accès élevé supplémentaire. Cet accès élevé doit être ajouté immédiatement avant de procéder à l’attribution de rôle, puis supprimé au terme de celle-ci.

- [Autorisations de rôle Administrateur dans Azure AD](../active-directory/roles/permissions-reference.md)

- [Attribution d’accès pour surveiller les données du journal d’activité au niveau du locataire](how-to/monitor-delegation-changes.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="pa-2-restrict-administrative-access-to-business-critical-systems"></a>PA-2 : Limiter l’accès administratif aux systèmes critiques de l’entreprise

**Conseils** : Azure Lighthouse utilise les contrôles d’accès en fonction du rôle Azure (RBAC Azure) pour isoler l’accès aux systèmes vitaux pour l’entreprise en restreignant les comptes disposant d’un accès privilégié aux abonnements et aux groupes d’administration dans lesquels ils se trouvent.

Veillez à suivre le principe du privilège minimum afin que les utilisateurs disposent uniquement des autorisations nécessaires pour effectuer leurs propres tâches.

Veillez également à limiter l'accès aux systèmes d'administration, d'identité et de sécurité qui ont un accès administratif à vos ressources critiques, comme les contrôleurs de domaine Active Directory, les outils de sécurité et les outils de gestion du système avec des agents installés sur les systèmes vitaux de l'entreprise. Les attaquants qui compromettent ces systèmes de gestion et de sécurité peuvent immédiatement les armer pour compromettre les ressources stratégiques de l’entreprise.

Pour assurer un contrôle d’accès cohérent, tous les types de contrôle d’accès doivent être alignés sur la stratégie de segmentation de votre entreprise.

- [Composants Azure et modèle de référence](/security/compass/microsoft-security-compass-introduction#azure-components-and-reference-model-2151)

- [Accès aux groupes d’administration](../governance/management-groups/overview.md#management-group-access)

- [Administrateurs d’abonnements Azure](../cost-management-billing/manage/add-change-subscription-administrator.md)

- [Meilleures pratiques pour la définition des utilisateurs et des rôles pour Azure Lighthouse](concepts/tenants-users-roles.md#best-practices-for-defining-users-and-roles)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="pa-3-review-and-reconcile-user-access-regularly"></a>PA-3 : Examiner et rapprocher régulièrement les accès utilisateur

**Conseils** : Azure Lighthouse utilise des comptes Azure Active Directory (Azure AD) pour gérer ses ressources, ainsi qu’examiner régulièrement les comptes utilisateur et l’attribution d’accès pour s’assurer que les comptes et leur accès sont valides. Vous pouvez utiliser les révisions d’accès Azure AD pour réviser les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. Les rapports Azure AD peuvent fournir des journaux pour vous aider à découvrir les comptes obsolètes. Vous pouvez également utiliser Azure AD Privileged Identity Management pour créer un workflow de rapport de révision d’accès afin de faciliter le processus de révision.

Les clients peuvent examiner le niveau d’accès accordé aux utilisateurs dans le locataire gestionnaire via Azure Lighthouse dans le portail Azure. Ils peuvent supprimer cet accès à tout moment.

En outre, Azure Privileged Identity Management peut être configuré pour déclencher des alertes lorsqu'un nombre excessif de comptes d'administrateur est créé, et pour identifier les comptes d'administrateur obsolètes ou mal configurés.

Notez que certains services Azure prennent en charge des utilisateurs et rôles locaux non gérés par le biais d’Azure AD. Vous devrez gérer ces utilisateurs séparément.

- [Créer une révision d’accès des rôles de ressources Azure dans Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-resource-roles-start-access-review.md) 

- [Supprimer l’accès à une délégation](how-to/remove-delegation.md)

- [Utilisation des révisions d’accès et des identités Azure AD](../active-directory/governance/access-reviews-overview.md)

- [Afficher la page des fournisseurs de services dans le portail Azure](how-to/view-manage-service-providers.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="pa-4-set-up-emergency-access-in-azure-ad"></a>PA-4 : Configurer l’accès d’urgence dans Azure AD

**Conseils** : Azure Lighthouse s’intègre à Azure Active Directory pour gérer ses ressources. Pour empêcher le verrouillage accidentel de votre organisation Azure AD, configurez un compte d’accès d’urgence pour conserver un accès lorsque les comptes administratifs normaux ne peuvent pas être utilisés. Les comptes d’accès d’urgence sont généralement hautement privilégiés et ne doivent pas être attribués à des personnes spécifiques. Les comptes d’accès d’urgence sont limités à des cas d’urgence ou à des scénarios « de secours » où il est impossible d’utiliser des comptes d’administration normaux.

Vous devez vous assurer que les informations d’identification (telles que le mot de passe, le certificat ou la carte à puce) des comptes d’accès d’urgence restent sécurisées et connues des seules personnes autorisées à les utiliser en cas d’urgence.

- [Gérer des comptes d’accès d’urgence dans Azure AD](../active-directory/roles/security-emergency-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="pa-5-automate-entitlement-management"></a>PA-5 : Automatiser la gestion des droits d'utilisation 

**Conseils** : Azure Lighthouse s’intègre à Azure Active Directory (Azure AD) pour gérer ses ressources. Utilisez les fonctionnalités de gestion des droits d’utilisation d’Azure AD pour automatiser les workflows de demandes d’accès, notamment les attributions, les révisions et l’expiration des accès. L’approbation en deux ou plusieurs étapes est également prise en charge.

- [Présentation des révisions d’accès Azure AD](../active-directory/governance/access-reviews-overview.md) 

- [Présentation de la gestion des droits d’utilisation Azure AD](../active-directory/governance/entitlement-management-overview.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="pa-6-use-privileged-access-workstations"></a>PA-6 : Utiliser des stations de travail d’accès privilégié

**Conseils** : Les stations de travail sécurisées et isolées sont extrêmement importantes pour la sécurité des rôles sensibles comme les administrateurs, développeurs et opérateurs de service critique. En fonction de vos besoins, vous pouvez utiliser des stations de travail utilisateur hautement sécurisées et/ou Azure Bastion pour effectuer des tâches de gestion administrative avec Azure Lighthouse dans des environnements de production. Utilisez Azure Active Directory, Microsoft Defender Advanced Threat Protection (MDATP) et/ou Microsoft Intune pour déployer une station de travail utilisateur sécurisée et gérée pour les tâches d’administration. Les stations de travail sécurisées peuvent être gérées de manière centralisée pour appliquer une configuration sécurisée, notamment une authentification forte, des lignes de base logicielles et matérielles et un accès réseau et logique restreint. 

- [Comprendre les stations de travail d’accès privilégié](../active-directory/devices/concept-azure-managed-workstation.md)

- [Déployer une station de travail d’accès privilégié](../active-directory/devices/howto-azure-managed-workstation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="pa-7-follow-just-enough-administration-least-privilege-principle"></a>PA-7 : Suivre le principe Just Enough Administration (privilèges minimum) 

**Conseils** : Azure Lighthouse est intégré au contrôle d'accès en fonction du rôle Azure (Azure RBAC) pour gérer ses ressources. Azure RBAC vous permet de gérer l’accès aux ressources Azure via des attributions de rôles. Vous pouvez affecter ces rôles intégrés à des utilisateurs, à des groupes, à des principaux de service et à des identités managées. Les rôles intégrés prédéfinis pour certaines ressources peuvent être inventoriés ou interrogés par le biais d’outils tels que Azure CLI, Azure PowerShell ou le Portail Azure. Les privilèges que vous affectez aux ressources via Azure RBAC doivent toujours être limités à ce qui est requis par les rôles. Ils complètent l’approche juste-à-temps (JIT) d’Azure AD Privileged Identity Management (PIM) et doivent être révisés régulièrement. Utilisez les rôles intégrés pour allouer des autorisations et ne créez des rôles personnalisés que si nécessaire.

Azure Lighthouse permet d’accéder aux ressources de client déléguées à l’aide des rôles intégrés Azure. Dans la plupart des cas, vous attribuez ces rôles à un groupe ou à un principal de service, plutôt qu’à de nombreux comptes d’utilisateur individuels. Cela vous permet d’ajouter ou de supprimer l’accès d’utilisateurs individuels sans devoir mettre à jour et republier le plan lorsque vos conditions d’accès changent.

Pour déléguer des ressources de client à un locataire gestionnaire, un déploiement doit être effectué par un compte non invité du locataire du client qui dispose du rôle intégré Propriétaire pour l'abonnement en cours d'intégration (ou qui contient les groupes de ressources en cours d'intégration).

- [Comprendre les locataires, utilisateurs et rôles dans Azure Lighthouse](concepts/tenants-users-roles.md)

- [Appliquer le principe du privilège minimum à Azure Lighthouse](concepts/recommended-security-practices.md#assign-permissions-to-groups-using-the-principle-of-least-privilege)

- [Présentation du contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md) 

- [Utilisation des révisions d’accès et des identités Azure AD](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="asset-management"></a>Gestion des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des ressources](../security/benchmarks/security-controls-v2-asset-management.md).*

### <a name="am-1-ensure-security-team-has-visibility-into-risks-for-assets"></a>AM-1 : S’assurer que l’équipe de sécurité a une visibilité sur les risques pour les ressources

**Conseils** : Assurez-vous que les équipes de sécurité reçoivent des autorisations de lecteur de sécurité dans votre locataire et vos abonnements Azure afin qu’elles puissent surveiller les risques de sécurité à l’aide d’Azure Security Center. 

Selon la façon dont les responsabilités de l’équipe de sécurité sont structurées, la surveillance des risques de sécurité peut tenir de la responsabilité d’une équipe de sécurité centrale ou d’une équipe locale. Cela dit, les insights et les risques liés à la sécurité doivent toujours être agrégés de manière centralisée au sein d’une organisation. 

Les autorisations de lecteur de sécurité peuvent être appliquées globalement à un locataire entier (groupe d’administration racine) ou étendues à des groupes d’administration ou à des abonnements spécifiques. 

Remarque : Des autorisations supplémentaires peuvent être nécessaires pour obtenir une visibilité sur les charges de travail et services. 

- [Vue d’ensemble du rôle lecteur de sécurité](../role-based-access-control/built-in-roles.md#security-reader)

- [Vue d'ensemble des groupes d'administration Azure](../governance/management-groups/overview.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="am-2-ensure-security-team-has-access-to-asset-inventory-and-metadata"></a>AM-2 : S’assurer que l’équipe de sécurité a accès à l’inventaire des ressources et aux métadonnées

**Conseils** : Les équipes de sécurité des clients peuvent consulter les journaux d’activité pour voir les activités effectuées par les fournisseurs de services utilisant Azure Lighthouse. 

Si un fournisseur de services souhaite autoriser son équipe de sécurité à passer en revue les ressources de client déléguées, les autorisations de l’équipe de sécurité doivent inclure le rôle intégré Lecteur.

- [Comment un client peut passer en revue l’activité d’un fournisseur de services](how-to/view-service-provider-activity.md)

- [Comment spécifier des rôles lors de l’intégration d’un client à Azure Lighthouse](how-to/onboard-customer.md#define-roles-and-permissions)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="am-3-use-only-approved-azure-services"></a>AM-3 : Utiliser des services Azure approuvés uniquement

**Conseils** : Utilisez Azure Policy pour auditer et limiter les services que les utilisateurs peuvent approvisionner dans votre environnement. Utilisez Azure Resource Graph pour interroger et découvrir des ressources dans leurs abonnements. Vous pouvez également utiliser Azure Monitor pour créer des règles afin de déclencher des alertes lorsqu’un service non approuvé est détecté.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/built-in-policies.md#general) 

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="am-4-ensure-security-of-asset-lifecycle-management"></a>AM-4 : Garantir la sécurité de la gestion du cycle de vie des actifs

**Conseils** : Supprimez l’accès aux ressources déléguées via le Azure Lighthouse lorsqu’elles ne sont plus nécessaires, de sorte que les fournisseurs de services n’y aient plus accès. Cet accès peut être supprimé par le client ou par le fournisseur de services. 

- [Supprimer l’accès à une délégation](how-to/remove-delegation.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="am-5-limit-users-ability-to-interact-with-azure-resource-manager"></a>AM-5 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Conseils** : Azure Lighthouse est compatible avec Azure Active Directory (Azure AD) pour l’identité et l’authentification. Vous pouvez utiliser l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

- [Guide pratique pour configurer l’accès conditionnel de façon à bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="logging-and-threat-detection"></a>Journalisation et détection des menaces

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Journalisation et détection des menaces](/azure/security/benchmarks/security-controls-v2-logging-threat-detection).*

### <a name="lt-1-enable-threat-detection-for-azure-resources"></a>LT-1 : Activer la détection des menaces pour les ressources Azure

**Conseils** : Azure Lighthouse vous permet de surveiller les ressources Azure de vos clients afin d’y détecter d’éventuelles menaces et anomalies. Concentrez-vous sur l’obtention d’alertes de haute qualité afin de réduire les faux positifs que doivent trier les analystes. Les alertes peuvent provenir de données de journal, d’agents ou d’autres données.

Utilisez la capacité de détection des menaces intégrée à Azure Security Center, qui repose sur la surveillance de la télémétrie du service Azure et sur l’analyse des journaux de service. Les données sont collectées à l’aide de l’agent Log Analytics, qui lit divers journaux d’événements et configurations liées à la sécurité du système et copie les données dans votre espace de travail à des fins d’analyse. 

En outre, utilisez Azure Sentinel pour créer des règles d’analyse, qui repèrent des menaces correspondant à des critères spécifiques dans l’environnement de votre client. Les règles génèrent des incidents lorsque les critères sont respectés, de sorte que vous pouvez enquêter sur chaque incident. Azure Sentinel peut également importer le renseignement sur les menaces provenant de tiers pour améliorer la capacité de détection des menaces. 

- [Protection contre les menaces dans Azure Security Center](../security-center/azure-defender.md)

- [Guide de référence des alertes de sécurité Azure Security Center](../security-center/alerts-reference.md)

- [Créer des règles d’analytique personnalisées pour détecter des menaces](../sentinel/tutorial-detect-threats-custom.md)

- [Gérer les espaces de travail Azure Sentinel à grande échelle](how-to/manage-sentinel-workspaces.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="lt-2-enable-threat-detection-for-azure-identity-and-access-management"></a>LT-2 : Activer la détection des menaces pour la gestion des identités et des accès Azure

**Conseils** : Azure Lighthouse vous permet d’utiliser Azure Security Center pour alerter en cas d’activités suspectes dans les locataires client que vous gérez, comme un nombre excessif de tentatives d’authentification ayant échoué et la présence de comptes dépréciés dans l’abonnement.

Azure Active Directory (Azure AD) fournit les journaux d’utilisateur suivants, qui peuvent être consultés dans les rapports Azure AD ou intégrés à Azure Monitor, Azure Sentinel ou autres outils de surveillance/SIEM pour des cas d’usage plus sophistiqués de surveillance et d’analyse :
- Connexion : le rapport de connexions fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.
- Journaux d’audit : traçabilité proposée via des journaux d’activité pour toutes les modifications effectuées par diverses fonctionnalités au sein d’Azure AD. Par exemple, les journaux d’audit peuvent inclure les modifications apportées à des ressources dans Azure AD comme l’ajout ou la suppression d’utilisateurs, d’applications, de groupes, de rôles, de stratégies, etc.
- Connexion risquée : une connexion risquée indique une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.
- Utilisateurs avec indicateur de risque : un utilisateur à risque correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Azure Security Center peut également alerter dans le cas de certaines activités suspectes, comme un nombre excessif de tentatives d’authentification ayant échoué et la présence de comptes dépréciés dans l’abonnement. En plus de la surveillance de base de l’hygiène de sécurité, le module Protection contre les menaces d’Azure Security Center peut également collecter des alertes de sécurité plus approfondies à partir de ressources de calcul Azure individuelles (machines virtuelles, conteneurs ou service d’application), de ressources de données (base de données SQL et stockage) et de couches de service Azure. Cette capacité offre une bonne visibilité sur les anomalies des comptes à l’intérieur des différentes ressources. 

- [Services et scénarios améliorés avec Azure Lighthouse](concepts/cross-tenant-management-experience.md#enhanced-services-and-scenarios)

- [Rapports d’activité d’audit dans Azure Active Directory](../active-directory/reports-monitoring/concept-audit-logs.md) 

- [Activer Azure AD Identity Protection](../active-directory/identity-protection/overview-identity-protection.md) 

- [Protection contre les menaces dans Azure Security Center](../security-center/azure-defender.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="lt-4-enable-logging-for-azure-resources"></a>LT-4 : Activer la journalisation pour les ressources Azure

**Conseils** : Les journaux d'activité, disponibles automatiquement, contiennent toutes les opérations d'écriture (PUT, POST, DELETE) pour vos ressources Azure Lighthouse, à l'exception des opérations de lecture (GET). Les journaux d’activité peuvent être utilisés pour rechercher une erreur lors de la résolution de problèmes ou pour surveiller la manière dont un utilisateur de votre organisation a modifié une ressource.

Azure Lighthouse vous permet d’utiliser les journaux Azure Monitor de manière évolutive sur les locataires client que vous gérez. Créez des espaces de travail Log Analytics directement dans les locataires client de sorte que les données client restent dans leurs locataires et ne soient pas exportés dans les vôtres. Cela permet également d’effectuer une supervision centralisée de l’ensemble des ressources ou services pris en charge par Log Analytics. Ainsi, vous disposez d’une plus grande flexibilité par rapport aux types de données que vous supervisez.

Les clients disposant d’abonnements délégués pour Azure Lighthouse peuvent consulter les données du journal d’activité Azure pour voir toutes les actions entreprises. Ils bénéficient ainsi d’une visibilité complète sur les opérations effectuées par les fournisseurs de services, ainsi que par les utilisateurs dans le locataire Azure Active Directory (Azure AD) du client.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md) 

- [Présentation de la journalisation et des différents types de journaux dans Azure](../azure-monitor/platform/platform-logs-overview.md)

- [Superviser les ressources déléguées à grande échelle](how-to/monitor-at-scale.md)

- [Voir l’activité des fournisseurs de services](how-to/view-service-provider-activity.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="lt-5-centralize-security-log-management-and-analysis"></a>LT-5 : Centraliser la gestion et l’analyse des journaux de sécurité

**Conseils** : Centralisez le stockage et l’analyse de la journalisation pour activer la corrélation. Pour chaque source de journal, assurez-vous d’avoir attribué un propriétaire de données, des conseils d’accès, un emplacement de stockage, les outils utilisés pour traiter les données et y accéder, ainsi que les exigences de conservation des données.

Veillez à intégrer les journaux d’activité Azure dans votre journalisation centralisée. Ingérez des journaux par le biais d’Azure Monitor pour agréger les données de sécurité générées par les appareils de point de terminaison, les ressources réseau et d’autres systèmes de sécurité. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, et utilisez des comptes Stockage Azure pour le stockage à long terme et l’archivage.

En outre, activez et intégrez les données dans Azure Sentinel ou une solution SIEM tierce.

Azure Lighthouse vous permet d’utiliser les journaux Azure Monitor de manière évolutive sur les locataires client que vous gérez. Créez des espaces de travail Log Analytics directement dans les locataires client de sorte que les données client restent dans leurs locataires et ne soient pas exportés dans les vôtres. Cela permet également d’effectuer une supervision centralisée de l’ensemble des ressources ou services pris en charge par Log Analytics. Ainsi, vous disposez d’une plus grande flexibilité par rapport aux types de données que vous supervisez.

Les clients disposant d’abonnements délégués pour Azure Lighthouse peuvent consulter les données du journal d’activité Azure pour voir toutes les actions entreprises. Ils bénéficient ainsi d’une visibilité complète sur les opérations effectuées par les fournisseurs de services, ainsi que par les utilisateurs dans le locataire Azure Active Directory (Azure AD) du client.

De nombreuses organisations choisissent d’utiliser Azure Sentinel pour les données « chaudes » qui sont utilisées fréquemment et Stockage Azure pour les données « froides » qui sont utilisées moins fréquemment.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md)

- [Superviser les ressources déléguées à grande échelle](how-to/monitor-at-scale.md)

- [Comment voir l’activité des fournisseurs de services](how-to/view-service-provider-activity.md)

- [Gérer les espaces de travail Azure Sentinel à grande échelle](how-to/manage-sentinel-workspaces.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="lt-6-configure-log-storage-retention"></a>LT-6 : Configurer la rétention du stockage des journaux

**Conseils** : Azure Lighthouse ne génère actuellement aucun journal lié à la sécurité. Les clients qui souhaitent afficher l’activité du fournisseur de services peuvent configurer la rétention des journaux en fonction des exigences de conformité, de réglementation et d’entreprise. 

Dans Azure Monitor, vous pouvez définir la période de rétention de votre espace de travail Log Analytics en fonction des règles de conformité de votre organisation. Utilisez des comptes Stockage Azure, Data Lake ou d’espace de travail Log Analytics pour le stockage à long terme et l’archivage.

- [Modification de la période de conservation des données dans Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

- [Guide pratique pour configurer la stratégie de conservation des journaux de compte de Stockage Azure](../storage/common/storage-monitor-storage-account.md#configure-logging)

- Configurer l’[exportation des alertes et recommandations Azure Security Center](../security-center/continuous-export.md) - Le client n’est pas en mesure de définir la rétention des journaux.

- [Comment un client peut passer en revue les données de journal d’activité des fournisseurs de services](how-to/view-service-provider-activity.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-controls-v2-incident-response.md).*

### <a name="ir-1-preparation--update-incident-response-process-for-azure"></a>IR-1 : Préparation – mettre à jour le processus de réponse aux incidents pour Azure

**Conseils** : Assurez-vous que votre organisation dispose de processus pour répondre aux incidents de sécurité, qu’elle a mis à jour ces processus pour Azure et qu’elle les exerce régulièrement pour garantir la préparation.

- [Implémenter la sécurité dans l’environnement de l’entreprise](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud)

- [Guide de référence sur les réponses aux incidents](/microsoft-365/downloads/IR-Reference-Guide.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="ir-2-preparation--setup-incident-notification"></a>IR-2 : Préparation – configurer la notification d’incident

**Conseils** : Configurez les coordonnées des personnes à contacter en cas d’incident de sécurité dans Azure Security Center. Microsoft utilisera ces coordonnées afin de vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Vous avez également la possibilité de personnaliser les alertes et les notifications d’incidents dans différents services Azure en fonction de vos besoins en matière de réponse aux incidents. 

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="ir-3-detection-and-analysis--create-incidents-based-on-high-quality-alerts"></a>IR-3 : Détection et analyse – créer des incidents en fonction d’alertes de haute qualité

**Instructions** : Veillez à avoir un processus de création d’alertes de bonne qualité et de mesure de la qualité des alertes. Cela vous permet de tirer les leçons des incidents passés et de classer par ordre de priorité les alertes pour les analystes, afin qu’ils ne perdent pas de temps sur les faux positifs. 

Vous pouvez créer des alertes de bonne qualité en vous basant sur l’expérience des incidents passés, sur les sources validées par la communauté, et sur des outils conçus pour générer et nettoyer les alertes en fusionnant et en mettant en corrélation différentes sources de signaux. 

Azure Security Center fournit des alertes de haute qualité sur de nombreuses ressources Azure. Vous pouvez utiliser le connecteur de données ASC pour diffuser en continu les alertes vers Azure Sentinel. Azure Sentinel vous permet de créer des règles d’alerte avancées pour générer automatiquement des incidents à des fins d’enquête. 

Exportez vos alertes et recommandations Azure Security Center en utilisant la fonctionnalité d’exportation pour identifier les risques pesant sur les ressources Azure. Exportez les alertes et les recommandations manuellement ou automatiquement de manière continue.

- [Procédure de configuration de l’exportation](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="ir-4-detection-and-analysis--investigate-an-incident"></a>IR-4 : Détection et analyse – enquêter sur un incident

**Conseils** : Veiller à ce que les analystes puissent interroger et utiliser diverses sources de données lorsqu’ils enquêtent sur des incidents potentiels, afin d’obtenir une vision complète de ce qui s’est passé. Différents journaux doivent être collectés pour suivre les activités d’un attaquant potentiel tout au long de la chaîne de destruction afin d’éviter les angles morts.  Vous devez également vous assurer que les insights et les enseignements sont capturés pour d’autres analystes et pour une référence historique future.  

Les sources de données à examiner comprennent les sources de journalisation centralisées qui sont déjà collectées auprès des services et des systèmes en fonctionnement concernés, mais elles peuvent également inclure les éléments suivants :

- Données réseau : utilisez les journaux de flux des groupes de sécurité réseau, Azure Network Watcher et Azure Monitor pour capturer des journaux de flux réseau et d’autres informations analytiques. 

- Captures instantanées des systèmes en fonctionnement : 

    - Utilisez la capacité de capture instantanée de la machine virtuelle Azure pour créer un instantané du disque du système en fonctionnement. 

    - Utilisez la capacité native de sauvegarde de la mémoire du système d’exploitation pour créer un instantané de la mémoire du système en fonctionnement.

    - Utilisez la capacité de capture instantanée des services Azure ou celle de votre logiciel pour créer des instantanés des systèmes en fonctionnement.

Azure Sentinel fournit des analyses de données approfondies sur pratiquement toutes les sources de journal et un portail de gestion des cas pour gérer le cycle de vie complet des incidents. Les renseignements obtenus au cours d’une enquête peuvent être associés à un incident à des fins de suivi et de rapport. 

- [Capture instantanée du disque d’un ordinateur Windows](../virtual-machines/windows/snapshot-copy-managed-disk.md)

- [Capture instantanée du disque d’un ordinateur Linux](../virtual-machines/linux/snapshot-copy-managed-disk.md)

- [Collecte de l’image mémoire et des informations de diagnostic par le support Microsoft Azure](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) 

- [Examiner les incidents avec Azure Sentinel](../sentinel/tutorial-investigate-cases.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="ir-5-detection-and-analysis--prioritize-incidents"></a>IR-5 : Détection et analyse – classer par ordre de priorité les incidents

**Conseils** : Donnez aux analystes le contexte sur lequel les incidents doivent se concentrer en premier lieu en fonction de la gravité de l’alerte et de la sensibilité des ressources. 

Azure Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez les ressources à l’aide d’étiquettes et créez un système de nommage pour identifier et classer les ressources Azure, en particulier celles qui traitent des données sensibles.  Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

- [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md)

- [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="ir-6-containment-eradication-and-recovery--automate-the-incident-handling"></a>IR-6 : Confinement, éradication et récupération – automatiser la gestion des incidents

**Conseils** : Automatisez les tâches manuelles répétitives pour accélérer le temps de réponse et réduire la charge de travail des analystes. Les tâches manuelles sont plus longues à exécuter, ce qui ralentit chaque incident et réduit le nombre d’incidents qu’un analyste peut traiter. Les tâches manuelles augmentent également la fatigue des analystes, ce qui accroît le risque d’erreur humaine entraînant des retards et dégrade la capacité des analystes à se concentrer efficacement sur des tâches complexes. Utilisez les fonctionnalités d’automatisation des workflows dans Azure Security Center et Azure Sentinel pour déclencher automatiquement des actions ou exécuter un playbook pour répondre aux alertes de sécurité entrantes. Le playbook prend des mesures, telles que l’envoi de notifications, la désactivation de comptes et l’isolement des réseaux problématiques. 

- [Configurer l’automatisation du workflow dans Security Center](../security-center/workflow-automation.md)

- [Configurer des réponses automatisées aux menaces dans Azure Security Center](../security-center/tutorial-security-incident.md#triage-security-alerts)

- [Configurer des réponses automatisées aux menaces dans Azure Sentinel](../sentinel/tutorial-respond-threats-playbook.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="posture-and-vulnerability-management"></a>Gestion de la posture et des vulnérabilités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion de la posture et des vulnérabilités](/azure/security/benchmarks/security-controls-v2-posture-vulnerability-management).*

### <a name="pv-1-establish-secure-configurations-for-azure-services"></a>PV-1 : Établir des configurations sécurisées pour les services Azure 

**Conseils** : Azure Lighthouse prend en charge les stratégies propres aux services ci-dessous, disponibles dans Azure Security Center pour auditer et appliquer les configurations de vos ressources Azure. Cet aspect peut être configuré dans les initiatives Azure Security Center ou Azure Policy.

- Autoriser l’intégration des ID de locataires gérants dans Azure Lighthouse

- Auditer la délégation d’étendues sur un locataire gérant

Vous pouvez utiliser Azure Blueprints pour automatiser le déploiement et la configuration de services et d’environnements d’application, notamment des modèles Resource Manager, des contrôles Azure RBAC et des stratégies, dans une même définition de blueprint.

- [Stratégies Azure Lighthouse](samples/policy-reference.md)

- [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md) 

- [Azure Blueprints](../governance/blueprints/overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="pv-2-sustain-secure-configurations-for-azure-services"></a>PV-2 : Supporter des configurations sécurisées pour les services Azure

**Conseils** : Azure Lighthouse prend en charge les stratégies propres aux services ci-dessous, disponibles dans Azure Security Center pour auditer et appliquer les configurations de vos ressources Azure. Cet aspect peut être configuré dans les initiatives Azure Security Center ou Azure Policy.

- [Stratégies Azure Lighthouse](samples/policy-reference.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="pv-3-establish-secure-configurations-for-compute-resources"></a>PV-3 : Établir des configurations sécurisées pour des ressources de calcul

**Conseils** : Utilisez Azure Security Center et Azure Policy pour établir des configurations sécurisées sur toutes les ressources de calcul, notamment les machines virtuelles, les conteneurs, etc.

- [Suivi des recommandations d'Azure Security Center](../security-center/security-center-recommendations.md) 

- [Recommandations de sécurité - Guide de référence](../security-center/recommendations-reference.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="pv-8-conduct-regular-attack-simulation"></a>PV-8 : Effectuer une simulation d’attaque régulière

**Conseils** : Selon les besoins, effectuez un test d’intrusion ou des activités Red Team sur vos ressources Azure et résolvez tous les problèmes de sécurité critiques détectés.
Suivez les règles d’engagement de pénétration du cloud Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft.

- [Test d’intrusion dans Azure](../security/fundamentals/pen-testing.md)

- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="governance-and-strategy"></a>Gouvernance et stratégie

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gouvernance et stratégie](../security/benchmarks/security-controls-v2-governance-strategy.md).*

### <a name="gs-1-define-asset-management-and-data-protection-strategy"></a>GS-1 : Définir la stratégie de gestion des ressources et de protection des données 

**Conseils** : Veillez à documenter et à communiquer une stratégie claire pour la surveillance et la protection continues des systèmes et des données. Hiérarchisez la découverte, l’évaluation, la protection et la surveillance des données et systèmes critiques de l’entreprise. 

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

-   Norme de classification des données en fonction des risques pour l’entreprise

-   Visibilité de l’organisation de sécurité sur les risques et l’inventaire des actifs 

-   Approbation de l’organisation de sécurité pour les services Azure en vue de leur utilisation 

-   Sécurité des ressources tout au long de leur cycle de vie

-   Stratégie de contrôle d’accès requise conformément à la classification des données organisationnelles

-   Utilisation des fonctionnalités de protection des données Azure natives et de tiers

-   Exigences de chiffrement des données pour les cas d’utilisation en transit et au repos

-   Normes de chiffrement appropriées

Pour plus d’informations, consultez les références suivantes :
- [Recommandation d’architecture de sécurité Azure - Stockage, données et chiffrement](/azure/architecture/framework/security/storage-data-encryption?bc=%2fsecurity%2fcompass%2fbreadcrumb%2ftoc.json&toc=%2fsecurity%2fcompass%2ftoc.json)

- [Notions de base de la sécurité Azure - Sécurité, chiffrement et stockage des données Azure](../security/fundamentals/encryption-overview.md)

- [Cloud Adoption Framework - Meilleures pratiques en matière de chiffrement et de sécurité des données Azure](../security/fundamentals/data-encryption-best-practices.md?bc=%2fazure%2fcloud-adoption-framework%2f_bread%2ftoc.json&toc=%2fazure%2fcloud-adoption-framework%2ftoc.json)

- [Benchmark de sécurité Azure - Gestion des ressources](/azure/security/benchmarks/security-controls-v2-incident-response)

- [Benchmark de sécurité Azure - Protection des données](/azure/security/benchmarks/security-controls-v2-data-protection)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-2-define-enterprise-segmentation-strategy"></a>GS-2 : Définir la stratégie de segmentation d'entreprise 

**Conseils** : Établissez une stratégie à l'échelle de l'entreprise pour segmenter l'accès aux ressources à l'aide d'une combinaison de contrôles d'identité, de réseau, d'application, d'abonnement, de groupe de gestion et autres.

Trouvez le bon équilibre entre la nécessité de séparation sur le plan de la sécurité et la nécessité d'exécuter quotidiennement les systèmes qui doivent communiquer entre eux et accéder aux données.

Veillez à ce que la stratégie de segmentation soit implémentée de manière cohérente pour tous les types de contrôle, y compris pour les modèles d'identité, d'accès et de sécurité du réseau, les modèles d'autorisation/d'accès aux applications et les contrôles des processus humains.

- [Aide relative à la stratégie de segmentation dans Azure (vidéo)](/security/compass/microsoft-security-compass-introduction#azure-components-and-reference-model-2151)

- [Aide relative à la stratégie de segmentation dans Azure (document)](/security/compass/governance#enterprise-segmentation-strategy)

- [Aligner la segmentation du réseau avec la stratégie de segmentation d’entreprise](/security/compass/network-security-containment#align-network-segmentation-with-enterprise-segmentation-strategy)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-3-define-security-posture-management-strategy"></a>GS-3 : Définir la stratégie de gestion de la posture de la sécurité

**Conseils** : Mesurez et atténuez en permanence les risques liés à vos ressources individuelles et à l’environnement dans lequel elles sont hébergées. Priorisez les ressources à valeur élevée et les surfaces d’attaque hautement exposées, comme les applications publiées, les points d’entrée et de sortie du réseau, les points de terminaison utilisateur et administrateur, etc.

- [Benchmark de sécurité Azure - Gestion de la posture et des vulnérabilités](/azure/security/benchmarks/security-controls-v2-posture-vulnerability-management)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-4-align-organization-roles-responsibilities-and-accountabilities"></a>GS-4 : Aligner les rôles et les responsabilités de l’organisation

**Conseils** : Veillez à documenter et à communiquer une stratégie claire pour les rôles et les responsabilités de votre organisation de sécurité. Veillez à définir clairement les responsabilités pour les décisions relatives à la sécurité, à former tout le monde au modèle de responsabilité partagée et à former les équipes techniques à la technologie permettant de sécuriser le cloud.

- [Meilleures pratiques pour la sécurité Azure 1 – Personnes : Former les équipes pour le parcours vers la sécurité dans le cloud](/azure/cloud-adoption-framework/security/security-top-10#1-people-educate-teams-about-the-cloud-security-journey)

- [Meilleures pratiques pour la sécurité Azure 2 – Personnes : Former les équipes pour les technologies de sécurité dans le cloud](/azure/cloud-adoption-framework/security/security-top-10#2-people-educate-teams-on-cloud-security-technology)

- [Meilleures pratiques pour la sécurité Azure 3 – Processus : Affecter les responsabilités pour les décisions de sécurité dans le cloud](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-5-define-network-security-strategy"></a>GS-5 : Définir la stratégie de sécurité réseau

**Conseils** : Établissez une approche de sécurité réseau Azure dans le cadre de la stratégie de contrôle d’accès de sécurité globale de votre organisation.  

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

-   Responsabilité centralisée pour la gestion et la sécurité du réseau

-   Modèle de segmentation de réseau virtuel aligné avec la stratégie de segmentation de l’entreprise

-   Stratégie de correction dans différents scénarios de menaces et d’attaques

-   Stratégie de périphérie d’Internet et d’entrée et de sortie /azure/security/benchmarks/security-controls-v2-logging-threat-detection
-   Stratégie de cloud hybride et d’interconnexion locale

-   Artefacts de sécurité réseau à jour (par exemple diagrammes réseau, architecture de réseau de référence)

Pour plus d’informations, consultez les références suivantes :
- [Meilleures pratiques pour la sécurité Azure 11 – Architecture. Stratégie de sécurité unifiée unique](/azure/cloud-adoption-framework/security/security-top-10#11-architecture-establish-a-single-unified-security-strategy)

- [Benchmark de sécurité Azure – Sécurité réseau](/azure/security/benchmarks/security-controls-v2-logging-threat-detection)

- [Vue d’ensemble de la sécurité réseau d’Azure](../security/fundamentals/network-overview.md)

- [Stratégie d’architecture de réseau d’entreprise](/azure/cloud-adoption-framework/ready/enterprise-scale/architecture)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-6-define-identity-and-privileged-access-strategy"></a>GS-6 : Définir une stratégie d’accès privilégié et d’identité

**Conseils** : Établissez une approche d’identité Azure et d’accès privilégié dans le cadre de la stratégie de contrôle d’accès de sécurité globale de votre organisation.  

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

-   Un système centralisé d’identité et d’authentification et son interconnexion avec d’autres systèmes d’identité internes et externes

-   Méthodes d’authentification fortes dans différents cas d’usage et différentes conditions

-   Protection des utilisateurs disposant de privilèges élevés

-   Surveillance et gestion des activités anormales des utilisateurs  

-   Vérification de l’identité et de l’accès des utilisateurs et processus de rapprochement

Pour plus d’informations, consultez les références suivantes :

- [Benchmark de sécurité Azure - Gestion des identités](/azure/security/benchmarks/security-controls-v2-identity-management)

- [Benchmark de sécurité Azure - Accès privilégié](/azure/security/benchmarks/security-controls-v2-privileged-access)

- [Meilleures pratiques pour la sécurité Azure 11 – Architecture. Stratégie de sécurité unifiée unique](/azure/cloud-adoption-framework/security/security-top-10#11-architecture-establish-a-single-unified-security-strategy)

- [Vue d’ensemble de la sécurité et de la gestion des identités Azure](../security/fundamentals/identity-management-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-7-define-logging-and-threat-response-strategy"></a>GS-7 : Définir la stratégie de journalisation et de réponse aux menaces

**Conseils** : Établissez une stratégie de journalisation et de réponse aux menaces pour détecter et corriger rapidement les menaces tout en répondant aux exigences de conformité. En priorité, fournissez aux analystes des alertes de bonne qualité et des expériences homogènes afin qu’ils puissent se concentrer sur les menaces plutôt que sur l’intégration et les étapes manuelles. 

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

-   Rôle et responsabilités de l’organisation d’opérations de sécurité (SecOP) 

-   Un processus de réponse aux incidents bien défini, aligné avec NIST ou autre cadre réglementaire du secteur 

-   Capture et rétention des journaux pour prendre en charge la détection des menaces, la réponse aux incidents et les besoins de conformité

-   Visibilité centralisée des informations de corrélation sur les menaces, avec SIEM, les fonctionnalités Azure natives et d’autres sources 

-   Plan de communication et de notification avec vos clients, fournisseurs et les parties publiques pertinentes

-   Utilisation de plateformes Azure natives et tierces pour la gestion des incidents, comme la journalisation et la détection des menaces, les investigations et la correction et l’éradication des attaques

-   Processus de gestion des incidents et des activités postérieures aux incidents, comme les leçons apprises et la rétention des preuves

Pour plus d’informations, consultez les références suivantes :

- [Benchmark de sécurité Azure - Journalisation et détection des menaces](/azure/security/benchmarks/security-controls-v2-logging-threat-detection)

- [Benchmark de sécurité Azure - Réponse aux incidents](/azure/security/benchmarks/security-controls-v2-incident-response)

- [Meilleures pratiques pour la sécurité Azure 4 - Processus. Mise à jour des processus de réponse aux incidents pour le cloud](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud)

- [Guide pour le cadre d’adoption d’Azure, la journalisation et la prise de décision pour les rapports](/azure/cloud-adoption-framework/decision-guides/logging-and-reporting/)

- [Mise à l’échelle, gestion et surveillance d’entreprise Azure](/azure/cloud-adoption-framework/ready/enterprise-scale/management-and-monitoring)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Vue d’ensemble d’Azure Security Benchmark V2](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)