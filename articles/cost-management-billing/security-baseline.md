---
title: Base de référence de la sécurité Azure pour Cost Management
description: La base de référence de la sécurité Cost Management fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: cost-management-billing
ms.topic: conceptual
ms.date: 11/16/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 0155ed4df9ee2c22d1461633ac9efe225b53f9ca
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96530508"
---
# <a name="azure-security-baseline-for-cost-management"></a>Base de référence de la sécurité Azure pour Cost Management

Cette base de référence de sécurité applique les conseils du [benchmark de sécurité Azure version 2.0](../security/benchmarks/overview.md) à Azure Cost Management. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Le contenu est regroupé en fonction des **contrôles de sécurité** définis par le benchmark de sécurité Azure et les conseils associés applicables à Cost Management. Les **contrôles** non applicables à Cost Management ont été exclus.

Pour voir comment Cost Management est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité de Cost Management](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="identity-management"></a>Gestion des identités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des identités](../security/benchmarks/security-controls-v2-identity-management.md).*

### <a name="im-1-standardize-azure-active-directory-as-the-central-identity-and-authentication-system"></a>IM-1 : Normaliser Azure Active Directory comme système d’authentification et d’identité central

**Conseils** : Azure Cost Management est intégré à Azure Active Directory (Azure AD), qui est le service de gestion des identités et des accès par défaut d’Azure. Vous devez normaliser Azure AD pour régir la gestion des identités et des accès de votre organisation dans :

- Les ressources cloud Microsoft, comme le Portail Azure, le stockage Azure, les machines virtuelles Azure (Linux et Windows), les applications Azure Key Vault, PaaS et SaaS.

- Les ressources de votre organisation, comme les applications sur Azure ou les ressources réseau de votre entreprise.

La sécurisation d’Azure AD doit être d’une priorité élevée dans les pratiques de sécurité cloud de votre organisation. Azure AD fournit un score d'identité sécurisée pour vous aider à évaluer votre posture de sécurité des identités par rapport aux recommandations de Microsoft en matière de meilleures pratiques. Utilisez le score pour évaluer avec précision votre configuration par rapport aux meilleures pratiques recommandées et apporter des améliorations à votre posture de sécurité.

Remarque : Azure AD prend en charge les fournisseurs d’identité externes, qui permettent aux utilisateurs sans compte Microsoft de se connecter à leurs applications et ressources avec leur identité externe.

- [Location dans Azure AD](../active-directory/develop/single-and-multi-tenant-apps.md)

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

- [Définir des locataires Azure AD](https://azure.microsoft.com/resources/securing-azure-environments-with-azure-active-directory/)  

- [Utiliser des fournisseurs d’identité externes pour une application](../active-directory/external-identities/identity-providers.md)

- [Qu’est-ce que le degré de sécurisation Identity Secure Score dans Azure AD](../active-directory/fundamentals/identity-secure-score.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-3-use-azure-ad-single-sign-on-sso-for-application-access"></a>IM-3 : Utiliser l’authentification unique Azure AD pour l’accès aux applications

**Conseils** : Azure Cost Management utilise Azure Active Directory pour fournir la gestion des identités et des accès aux ressources Azure, aux applications cloud et aux applications locales. Cela inclut les identités d’entreprise, comme les employés, ainsi que les identités externes, comme les partenaires et les fournisseurs. Cela permet à l’authentification unique de gérer et sécuriser l’accès aux données et ressources de votre organisation localement et dans le cloud. Connectez l’ensemble de vos utilisateurs, applications et appareils à Azure AD pour un accès transparent et sécurisé, ainsi qu’une visibilité et un contrôle accrus.

- [Comprendre l’authentification unique des applications avec Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-4-use-strong-authentication-controls-for-all-azure-active-directory-based-access"></a>IM-4 : Utiliser des contrôles d’authentification renforcés pour tous les accès basés sur Azure Active Directory

**Conseils** : Azure Cost Management est intégré à Azure AD, qui prend en charge des contrôles d’authentification renforcés via l’authentification multifacteur (MFA) et des méthodes fortes sans mot de passe.

- Authentification multifacteur – Activez l’authentification multifacteur Azure AD et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center pour appliquer quelques bonnes pratiques dans votre configuration MFA. L’authentification multifacteur peut être appliquée à tous les utilisateurs, à des utilisateurs sélectionnés ou au niveau de chaque utilisateur en fonction des conditions de connexion et des facteurs de risque.

- Authentification sans mot de passe – Trois options d’authentification sans mot de passe sont disponibles : Windows Hello Entreprise, l’application Microsoft Authenticator et les méthodes d’authentification locales, comme les cartes à puce.

Pour les utilisateurs administrateurs et privilégiés, assurez-vous que le niveau le plus élevé de méthode d’authentification forte est utilisé, puis déployez la stratégie d’authentification forte appropriée pour les autres utilisateurs.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md) 

- [Introduction aux options d’authentification sans mot de passe pour Azure Active Directory](../active-directory/authentication/concept-authentication-passwordless.md) 

- [Stratégie de mot de passe par défaut d’Azure AD](../active-directory/authentication/concept-sspr-policy.md#password-policies-that-only-apply-to-cloud-user-accounts)

- [Éliminer les mauvais mots de passe à l’aide de Protection de mots de passe d’Azure AD](../active-directory/authentication/concept-password-ban-bad.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="im-5-monitor-and-alert-on-account-anomalies"></a>IM-5 : Surveiller et alerter en cas d’anomalies de compte

**Conseils** : Azure Cost Management est intégré à Azure Active Directory, qui fournit les sources de données suivantes :

- Connexions – Le rapport de connexions fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.

- Journaux d’audit : traçabilité proposée via des journaux d’activité pour toutes les modifications effectuées par diverses fonctionnalités au sein d’Azure AD. Par exemple, les journaux d’audit peuvent inclure les modifications apportées à des ressources dans Azure AD comme l’ajout ou la suppression d’utilisateurs, d’applications, de groupes, de rôles, de stratégies, etc.

- Connexions risquées : une connexion risquée est une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.

- Utilisateurs avec indicateur de risque : un utilisateur à risque correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Ces sources de données peuvent être intégrées à Azure Monitor, Azure Sentinel ou des systèmes SIEM tiers.

Azure Security Center peut également alerter dans le cas de certaines activités suspectes, comme un nombre excessif de tentatives d’authentification ayant échoué et la présence de comptes dépréciés dans l’abonnement.

Azure Advanced Threat Protection (ATP) est une solution de sécurité qui peut utiliser les signaux Active Directory pour identifier, détecter et examiner les menaces avancées, les identités compromises et les actions malveillantes internes.

- [Rapports d’activité d’audit dans Azure Active Directory](../active-directory/reports-monitoring/concept-audit-logs.md) 

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../active-directory/identity-protection/overview-identity-protection.md) 

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../security-center/security-center-identity-access.md) 

- [Alertes dans le module de protection de renseignement sur les menaces d’Azure Security Center](../security-center/alerts-reference.md) 

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="privileged-access"></a>Accès privilégié

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Accès privilégié](../security/benchmarks/security-controls-v2-privileged-access.md).*

### <a name="pa-1-protect-and-limit-highly-privileged-users"></a>PA-1 : Protéger et limiter les utilisateurs disposant de privilèges élevés

**Conseils** : Cost Management pour les Contrats Entreprise (EA) dispose du compte doté de privilèges élevés ci-dessous.

- Administrateur d’entreprise

Il est recommandé de vérifier régulièrement les utilisateurs affectés aux rôles au sein de votre Contrat Entreprise (EA).

Il est également recommandé, en tant que règle, de limiter le nombre de comptes ou rôles dotés de privilèges élevés et de protéger ces comptes à un niveau élevé, car les utilisateurs disposant de ces privilèges peuvent lire et modifier directement ou indirectement chaque ressource figurant dans votre environnement Azure.

Vous pouvez activer l’accès privilégié juste-à-temps (JAT) aux ressources Azure et Azure AD en utilisant Azure AD Privileged Identity Management (PIM). JAT accorde des autorisations temporaires pour effectuer des tâches privilégiées uniquement lorsque les utilisateurs en ont besoin. PIM peut également générer des alertes de sécurité en cas d’activité suspecte ou non sécurisée dans votre organisation Azure AD.

- [Gestion des rôles Azure Enterprise](manage/understand-ea-roles.md) 

- [Autorisations de rôle Administrateur dans Azure AD](../active-directory/roles/permissions-reference.md) 

- [Utiliser les alertes de sécurité Azure Privileged Identity Management](../active-directory/privileged-identity-management/pim-how-to-configure-security-alerts.md) 

- [Sécurisation de l’accès privilégié pour les déploiements hybrides et cloud dans Azure AD](../active-directory/roles/security-planning.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="pa-3-review-and-reconcile-user-access-regularly"></a>PA-3 : Examiner et rapprocher régulièrement les accès utilisateur

**Conseils** : Azure Cost Management s’appuie sur l’accès approprié pour afficher et gérer les données de coût organisationnel. L’accès est contrôlé via le contrôle d’accès en fonction du rôle Azure (Azure RBAC) au niveau de l’abonnement et par le biais de rôles d’administration avec des Contrats Entreprise (EA) ou un Contrat client Microsoft (MCA) pour les étendues de facturation. Auditez et examinez régulièrement l’accès accordé pour vous assurer que les utilisateurs ou les groupes disposent de l’accès requis.

- [Gérer l’accès aux informations de facturation pour Azure](manage/manage-billing-access.md)

- [Autoriser l’accès aux données Gestion des coûts](costs/assign-access-acm-data.md)

- [Gestion des rôles Azure Enterprise](manage/understand-ea-roles.md)

- [Présentation des rôles d'administrateur Azure dans le cadre des Contrats client Microsoft](manage/understand-mca-roles.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="pa-7-follow-just-enough-administration-least-privilege-principle"></a>PA-7 : Suivre le principe Just Enough Administration (privilèges minimum) 

**Conseils** : Azure Cost Management s’intègre au contrôle d’accès en fonction du rôle (RBAC) Azure pour gérer les ressources (par exemple, les budgets, les rapports enregistrés, etc.). Azure RBAC vous permet de gérer l’accès aux ressources Azure via des attributions de rôles. Vous pouvez affecter ces rôles à des utilisateurs, des groupes et des principaux de service. Les rôles intégrés prédéfinis pour certaines ressources peuvent être inventoriés ou interrogés par le biais d’outils tels que Azure CLI, Azure PowerShell ou le Portail Azure. Les privilèges que vous affectez aux ressources via Azure RBAC doivent toujours être limités à ce qui est requis par les rôles. Ils complètent l’approche juste-à-temps (JIT) d’Azure AD Privileged Identity Management (PIM) et doivent être révisés régulièrement.

Utilisez des rôles intégrés pour allouer les autorisations et créez un rôle personnalisé uniquement lorsque cela est nécessaire.

Azure Cost Management offre des rôles intégrés, des lecteurs et des contributeurs.

- [Lecteur Azure Cost Management](../role-based-access-control/built-in-roles.md#cost-management-reader)

- [Contributeur Azure Cost Management](../role-based-access-control/built-in-roles.md#cost-management-contributor)

Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (RBAC Azure) ? ../role-based-access-control/overview.md 

- [Guide pratique pour configurer le contrôle RBAC dans Azure](../role-based-access-control/role-assignments-portal.md) 

- [Utilisation des révisions d’accès et des identités Azure AD](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : protection des données](../security/benchmarks/security-controls-v2-data-protection.md).*

### <a name="dp-1-discovery-classify-and-label-sensitive-data"></a>DP-1 : Découvrir, classifier et étiqueter des données sensibles

**Conseils** : Nous vous recommandons de découvrir, classifier et étiqueter vos données sensibles de façon à concevoir les contrôles appropriés pour faire en sorte que le stockage, le traitement et la transmission sécurisées des informations sensibles soient assurés par les systèmes technologiques de l’organisation.

Utilisez Azure Information Protection (et son outil d’analyse associé) pour les informations sensibles présentes dans les documents Office au niveau d’Azure, de l’environnement local, d’Office 365 et ailleurs.

Vous pouvez utiliser Azure SQL Information Protection pour faciliter la classification et l’étiquetage des informations stockées dans les bases de données Azure SQL Database.

- [Étiqueter des informations sensibles à l’aide d’Azure Information Protection](/azure/information-protection/what-is-information-protection) 

- [Guide pratique pour implémenter la recherche de données Azure SQL](../azure-sql/database/data-discovery-and-classification-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="dp-2-protect-sensitive-data"></a>DP-2 : Protection des données sensibles

**Conseils** : Nous vous recommandons de protéger les données sensibles en limitant l’accès avec le contrôle d’accès en fonction du rôle Azure (Azure RBAC), des contrôles d’accès basés sur le réseau et des contrôles spécifiques dans les services Azure (comme le chiffrement dans les bases de données SQL et autres).

Pour assurer un contrôle d’accès cohérent, tous les types de contrôle d’accès doivent être alignés sur la stratégie de segmentation de votre entreprise. La stratégie de segmentation de l’entreprise doit aussi être informée de l’emplacement des systèmes et données sensibles ou vitaux de l’entreprise.

Pour la plateforme sous-jacente gérée par Microsoft, Microsoft traite tout le contenu client en tant que contenu sensible, et assure une protection contre la perte et l’exposition des données client. Pour assurer la sécurité des données des clients dans Azure, Microsoft a implémenté certains contrôles et fonctionnalités de protection des données par défaut.

- [Autoriser l’accès aux données Gestion des coûts](costs/assign-access-acm-data.md)

- [Contrôle d’accès en fonction du rôle (RBAC) Azure](../role-based-access-control/overview.md) 

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="dp-3-monitor-for-unauthorized-transfer-of-sensitive-data"></a>DP-3 : Détection des transferts non autorisés de données sensibles

**Conseils** : Détectez les transferts non autorisés de données vers des emplacements situés hors du champ de visibilité ou de contrôle de l’entreprise. Cela implique généralement de surveiller des activités anormales (transferts volumineux ou inhabituels) qui pourraient être le signe d’une exfiltration de données non autorisée.

Dans Stockage Azure et Azure SQL, la protection avancée contre les menaces (ou Advanced Threat Protection, ATP) peut générer une alerte en cas de transfert d’informations anormal qui pourrait être le signe d’un transfert non autorisé d’informations sensibles.

Azure Information Protection (AIP) offre des fonctionnalités de supervision pour les informations qui ont été classifiées et étiquetées.

Si cela est nécessaire pour la conformité de la protection contre la perte de données (DLP), vous pouvez utiliser une solution DLP basée sur l’hôte pour appliquer des contrôles de détection et/ou de prévention de façon à empêcher l’exfiltration de données.

- [Activer ATP pour Azure SQL](../azure-sql/database/threat-detection-overview.md) 

- [Activer ATP pour Stockage Azure](../storage/common/azure-defender-storage-configure.md?tabs=azure-security-center)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="dp-5-encrypt-sensitive-data-at-rest"></a>DP-5 : Chiffrement des données sensibles au repos

**Conseils** : Pour compléter les contrôles d’accès, la fonctionnalité d’exportation d’Azure Cost Management prend en charge le chiffrement par le Stockage Azure des données au repos afin d’assurer une protection contre les attaques « hors bande » (telles que l’accès au stockage sous-jacent) via le chiffrement à l’aide des clés gérées par Microsoft. Ce paramètre par défaut vise à empêcher que les attaquants puissent facilement lire ou modifier les données.

Pour plus d’informations, consultez les pages suivantes :

- [Chiffrement du stockage Azure pour les données au repos](../storage/common/storage-service-encryption.md)

- [Présentation du chiffrement au repos dans Azure](../security/fundamentals/encryption-atrest.md#encryption-at-rest-in-microsoft-cloud-services)

**Supervision d’Azure Security Center** : Non applicable

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

### <a name="am-4-ensure-security-of-asset-lifecycle-management"></a>AM-4 : Garantir la sécurité de la gestion du cycle de vie des actifs

**Conseils** : Établissez ou mettez à jour des stratégies de sécurité qui traitent les processus de gestion du cycle de vie pour les modifications à fort impact potentiel. Ces modifications incluent les modifications apportées aux fournisseurs d’identité et aux accès, à la sensibilité des données, à la configuration du réseau et à l’attribution des privilèges d’administration.

Supprimez les ressources Azure qui ne sont plus nécessaires.

- [Supprimer un groupe de ressources et des ressources Azure](../azure-resource-manager/management/delete-resource-group.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-threat-detection"></a>Journalisation et détection des menaces

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Journalisation et détection des menaces](../security/benchmarks/security-controls-v2-logging-threat-detection.md).*

### <a name="lt-2-enable-threat-detection-for-azure-identity-and-access-management"></a>LT-2 : Activer la détection des menaces pour la gestion des identités et des accès Azure

**Conseils** : Azure AD fournit les journaux d’utilisateur suivants qui peuvent être consultés dans les rapports Azure AD ou intégrés à Azure Monitor, Azure Sentinel ou d’autres outils de surveillance/SIEM pour des cas d’usage plus sophistiqués de surveillance et d’analytique :

- Connexions – le rapport de connexions fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.

- Journaux d’audit : traçabilité proposée via des journaux d’activité pour toutes les modifications effectuées par diverses fonctionnalités au sein d’Azure AD. Par exemple, les journaux d’audit peuvent inclure les modifications apportées à des ressources dans Azure AD comme l’ajout ou la suppression d’utilisateurs, d’applications, de groupes, de rôles, de stratégies, etc.

- Connexions risquées : une connexion risquée est une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.

- Utilisateurs avec indicateur de risque : un utilisateur à risque correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Azure Security Center peut également alerter dans le cas de certaines activités suspectes, comme un nombre excessif de tentatives d’authentification ayant échoué et la présence de comptes dépréciés dans l’abonnement. En plus de la surveillance de base de l’hygiène de sécurité, le module Protection contre les menaces d’Azure Security Center peut également collecter des alertes de sécurité plus approfondies à partir de ressources de calcul Azure individuelles (machines virtuelles, conteneurs ou service d’application), de ressources de données (base de données SQL et stockage) et de couches de service Azure. Cette capacité vous permet d’avoir une visibilité sur les anomalies de compte à l’intérieur des ressources individuelles.

Il est également recommandé de vérifier régulièrement les utilisateurs affectés aux rôles au sein de votre Contrat Entreprise (EA). 

- [Gestion des rôles Azure Enterprise](manage/understand-ea-roles.md)

- [Rapports d’activité d’audit dans Azure Active Directory](../active-directory/reports-monitoring/concept-audit-logs.md) 

- [Activer Azure AD Identity Protection](../active-directory/identity-protection/overview-identity-protection.md)

- [Protection contre les menaces dans Azure Security Center](../security-center/azure-defender.md)

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

**Conseils** : Assurez-vous de disposer d’un processus permettant de créer des alertes de haute qualité et de mesurer la qualité des alertes. Cela vous permet de tirer les leçons des incidents passés et de classer par ordre de priorité les alertes pour les analystes, afin qu’ils ne perdent pas de temps sur les faux positifs. 

Les alertes de haute qualité peuvent être créées en fonction de l’expérience acquise lors des incidents passés, des sources validées de la communauté et des outils conçus pour générer et nettoyer les alertes en fusionnant et en mettant en corrélation diverses sources de signaux. 

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

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion de la posture et des vulnérabilités](../security/benchmarks/security-controls-v2-posture-vulnerability-management.md).*

### <a name="pv-7-rapidly-and-automatically-remediate-software-vulnerabilities"></a>PV-7 : Corriger rapidement et automatiquement des vulnérabilités logicielles

**Conseils** : Déployez rapidement des mises à jour logicielles pour corriger les vulnérabilités logicielles dans les systèmes d’exploitation et les applications.

Privilégiez l’utilisation d’un programme de scoring des risques courants (par exemple, Common Vulnerability Scoring System) ou les niveaux de risque par défaut fournis par votre outil d’analyse tiers, et adaptez-les à votre environnement, en tenant compte des applications présentant un risque élevé pour la sécurité et de celles qui nécessitent une durée de fonctionnement élevée.

**Supervision d’Azure Security Center** : Non applicable

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

-   Utilisation des fonctionnalités de protection des données natives et tierces d’Azure

-   Exigences de chiffrement des données pour les cas d’utilisation en transit et au repos

-   Normes de chiffrement appropriées

Pour plus d’informations, consultez les références suivantes :
- [Recommandation d’architecture de sécurité Azure - Stockage, données et chiffrement](/azure/architecture/framework/security/storage-data-encryption?bc=%2fsecurity%2fcompass%2fbreadcrumb%2ftoc.json&toc=%2fsecurity%2fcompass%2ftoc.json)

- [Notions de base de la sécurité Azure - Sécurité, chiffrement et stockage des données Azure](../security/fundamentals/encryption-overview.md)

- [Cloud Adoption Framework - Meilleures pratiques en matière de chiffrement et de sécurité des données Azure](../security/fundamentals/data-encryption-best-practices.md?bc=%2fazure%2fcloud-adoption-framework%2f_bread%2ftoc.json&toc=%2fazure%2fcloud-adoption-framework%2ftoc.json)

- [Benchmark de sécurité Azure - Gestion des ressources](../security/benchmarks/security-controls-v2-asset-management.md)

- [Benchmark de sécurité Azure - Protection des données](../security/benchmarks/security-controls-v2-data-protection.md)

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

- [Benchmark de sécurité Azure - Gestion de la posture et des vulnérabilités](../security/benchmarks/security-controls-v2-posture-vulnerability-management.md)

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

-   Stratégie de périphérie d’Internet et d’entrée et de sortie

-   Stratégie de cloud hybride et d’interconnexion locale

-   Artefacts de sécurité réseau à jour (par exemple diagrammes réseau, architecture de réseau de référence)

Pour plus d’informations, consultez les références suivantes :
- [Meilleures pratiques pour la sécurité Azure 11 – Architecture. Stratégie de sécurité unifiée unique](/azure/cloud-adoption-framework/security/security-top-10#11-architecture-establish-a-single-unified-security-strategy)

- [Benchmark de sécurité Azure – Sécurité réseau](../security/benchmarks/security-controls-v2-network-security.md)

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

- [Benchmark de sécurité Azure - Gestion des identités](../security/benchmarks/security-controls-v2-identity-management.md)

- [Benchmark de sécurité Azure - Accès privilégié](../security/benchmarks/security-controls-v2-privileged-access.md)

- [Meilleures pratiques pour la sécurité Azure 11 – Architecture. Stratégie de sécurité unifiée unique](/azure/cloud-adoption-framework/security/security-top-10#11-architecture-establish-a-single-unified-security-strategy)

- [Vue d’ensemble de la sécurité et de la gestion des identités Azure](../security/fundamentals/identity-management-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-7-define-logging-and-threat-response-strategy"></a>GS-7 : Définir la stratégie de journalisation et de réponse aux menaces

**Conseils** : Établissez une stratégie de journalisation et de réponse aux menaces pour détecter et corriger rapidement les menaces tout en répondant aux exigences de conformité. Veillez à ce que les analystes reçoivent des alertes de haute qualité et des expériences homogènes afin qu’ils puissent se concentrer sur les menaces plutôt que sur l’intégration et les étapes manuelles. 

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

-   Rôle et responsabilités de l’organisation d’opérations de sécurité (SecOP) 

-   Un processus de réponse aux incidents bien défini, aligné avec NIST ou autre cadre réglementaire du secteur 

-   Capture et rétention des journaux pour prendre en charge la détection des menaces, la réponse aux incidents et les besoins de conformité

-   Visibilité centralisée des informations de corrélation sur les menaces, avec SIEM, les fonctionnalités Azure natives et d’autres sources 

-   Plan de communication et de notification avec vos clients, fournisseurs et les parties publiques pertinentes

-   Utilisation de plateformes Azure natives et tierces pour la gestion des incidents, comme la journalisation et la détection des menaces, les investigations et la correction et l’éradication des attaques

-   Processus de gestion des incidents et des activités postérieures aux incidents, comme les leçons apprises et la rétention des preuves

Pour plus d’informations, consultez les références suivantes :

- [Benchmark de sécurité Azure - Journalisation et détection des menaces](../security/benchmarks/security-controls-v2-logging-threat-detection.md)

- [Benchmark de sécurité Azure - Réponse aux incidents](../security/benchmarks/security-controls-v2-incident-response.md)

- [Meilleures pratiques pour la sécurité Azure 4 - Processus. Mise à jour des processus de réponse aux incidents pour le cloud](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud)

- [Guide pour le cadre d’adoption d’Azure, la journalisation et la prise de décision pour les rapports](/azure/cloud-adoption-framework/decision-guides/logging-and-reporting/)

- [Mise à l’échelle, gestion et surveillance d’entreprise Azure](/azure/cloud-adoption-framework/ready/enterprise-scale/management-and-monitoring)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Vue d’ensemble d’Azure Security Benchmark V2](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)