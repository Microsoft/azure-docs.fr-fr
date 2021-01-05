---
title: Guide de référence des opérations de gestion de l’authentification Azure Active Directory
description: Ce guide de référence des opérations décrit les vérifications et les actions à entreprendre pour sécuriser la gestion de l’authentification.
services: active-directory
author: martincoetzer
manager: daveba
tags: azuread
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: fundamentals
ms.date: 10/31/2019
ms.author: martinco
ms.openlocfilehash: 90e215ea445c8c700e351149e9c7a91d9a595252
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96859519"
---
# <a name="azure-active-directory-authentication-management-operations-reference-guide"></a>Guide de référence des opérations de gestion de l’authentification Azure Active Directory

Cette section du [Guide de référence des opérations Azure AD](active-directory-ops-guide-intro.md) décrit les vérifications et les actions à entreprendre pour sécuriser et gérer les informations d’identification, définir l’expérience d’authentification, déléguer l’affectation, mesurer l’utilisation et définir des stratégies d’accès basées sur l’attitude de l’entreprise en matière de sécurité.

> [!NOTE]
> Ces suggestions sont valables à la date de publication mais peuvent évoluer. Les organisations doivent évaluer en permanence leurs pratiques de gestion des identités à mesure que les produits et services Microsoft évoluent.

## <a name="key-operational-processes"></a>Processus opérationnels clés

### <a name="assign-owners-to-key-tasks"></a>Affecter les propriétaires à des tâches clés

La gestion d’Azure Active Directory nécessite l’exécution continue de tâches et de processus opérationnels clés, qui peuvent ne pas faire partie d’un projet de déploiement. Il est cependant important de configurer ces tâches pour optimiser votre environnement. Les tâches clés et leurs propriétaires recommandés sont listés ci-après :

| Tâche | Propriétaire |
| :- | :- |
| Gérer le cycle de vie de la configuration de l’authentification unique (SSO) dans Azure AD | Équipe des opérations IAM |
| Concevoir des stratégies d’accès conditionnel pour les applications Azure AD | Équipe d’architecture InfoSec |
| Archiver l’activité de connexion dans un système SIEM | Équipe des opérations InfoSec |
| Archiver les événements à risque dans un système SIEM | Équipe des opérations InfoSec |
| Trier et examiner les rapports de sécurité | Équipe des opérations InfoSec |
| Trier et examiner les événements à risque | Équipe des opérations InfoSec |
| Trier et examiner les utilisateurs avec indicateur de risque et les rapports de vulnérabilité d’Azure AD Identity Protection | Équipe des opérations InfoSec |

> [!NOTE]
> Azure AD Identity Protection nécessite une licence Azure AD Premium P2. Pour trouver la licence adaptée à vos besoins, consultez [Comparaison des fonctionnalités généralement disponibles des éditions Azure AD Free et Azure AD Premium](https://azure.microsoft.com/pricing/details/active-directory/).

Quand vous passerez votre liste en revue, vous devrez peut-être affecter un propriétaire à des tâches qui en sont dépourvues, ou modifier la propriété des tâches avec propriétaires qui ne sont pas conformes aux suggestions ci-dessus.

#### <a name="owner-recommended-reading"></a>Lectures recommandées pour les propriétaires

- [Attribution de rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md)
- [Gouvernance dans Azure](../../governance/index.yml)

## <a name="credentials-management"></a>Gestion des informations d’identification

### <a name="password-policies"></a>Stratégies de mot de passe

La gestion sécurisée des mots de passe est un des aspects les plus critiques de la gestion des identités et des accès, et ils sont souvent la cible principale des attaques. Azure AD prend en charge plusieurs fonctionnalités qui permettent d’éviter la réussite d’une attaque.

Utilisez le tableau ci-dessous pour trouver la solution recommandée permettant de limiter le problème à résoudre :

| Problème | Recommandation |
| :- | :- |
| Pas de mécanisme de protection contre les mots de passe faibles | Activez [la réinitialisation du mot de passe en libre-service (SSPR)](../authentication/concept-sspr-howitworks.md) et [la protection par mot de passe](../authentication/concept-password-ban-bad-on-premises.md) d’Azure AD |
| Pas de mécanisme de détection des mots de passe ayant fuité | Activez la [synchronisation du hachage de mot de passe](../hybrid/how-to-connect-password-hash-synchronization.md) pour obtenir des insights |
| Utilisation d’AD FS et impossibilité de passer à l’authentification managée | Activez le [verrouillage intelligent d’extranet AD FS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-smart-lockout-protection) et/ou le [verrouillage intelligent Azure AD](../authentication/howto-password-smart-lockout.md) |
| La stratégie de mot de passe utilise des règles de complexité telles que la longueur, les jeux de caractères multiples ou l’expiration | Reconsidérez-la conformément aux [pratiques recommandées par Microsoft](https://www.microsoft.com/research/publication/password-guidance/?from=http%3A%2F%2Fresearch.microsoft.com%2Fpubs%2F265143%2Fmicrosoft_password_guidance.pdf), passez à la gestion des mots de passe et déployez la [Protection par mot de passe Azure AD](../authentication/concept-password-ban-bad.md). |
| Les utilisateurs ne sont pas inscrits pour l’utilisation de l’authentification multifacteur (MFA) | [Inscrivez toutes les informations de sécurité de l’utilisateur](../identity-protection/howto-identity-protection-configure-mfa-policy.md) pour qu’elles puissent être utilisées comme un mécanisme permettant de vérifier l’identité de l’utilisateur avec son mot de passe |
| Il n’y a pas de révocation des mots de passe en fonction du risque de l’utilisateur | Déployez des [stratégies de risque utilisateur d’Azure AD Identity Protection](../identity-protection/howto-identity-protection-configure-risk-policies.md) pour forcer avec SSPR le changement des mots de passe sur les informations d’identification divulguées |
| Il n’existe pas de mécanisme de verrouillage intelligent pour protéger contre l’authentification malveillante d’acteurs malveillants provenant d’adresses IP identifiées | Déployez l’authentification managée par le cloud avec la synchronisation du hachage de mot de passe ou l’[authentification directe](../hybrid/how-to-connect-pta-quick-start.md) |

#### <a name="password-policies-recommended-reading"></a>Lectures recommandées sur les stratégies de mot de passe

- [Bonnes pratiques d’Azure AD et d’AD FS : Défense contre les attaques par pulvérisation de mots de passe - Enterprise Mobility + Security](https://cloudblogs.microsoft.com/enterprisemobility/2018/03/05/azure-ad-and-adfs-best-practices-defending-against-password-spray-attacks/)

### <a name="enable-self-service-password-reset-and-password-protection"></a>Activez la réinitialisation du mot de passe en libre-service et la protection par mot de passe

Les utilisateurs qui ont besoin de changer ou de réinitialiser leur mot de passe sont une des plus grandes sources de volume et de coût des appels au support technique. En plus du coût, le changement de mot de passe vu comme outil pour réduire le risque utilisateur est une étape fondamentale dans l’amélioration de l’attitude de votre organisation en matière de sécurité.

Au minimum, il est recommandé de déployer la [réinitialisation de mot de passe en libre-service](../authentication/concept-sspr-howitworks.md) et de la [protection par mot de passe](../authentication/howto-password-ban-bad-on-premises-deploy.md) en local d’Azure AD pour accomplir les opérations suivantes :

- Rediriger les appels au support technique.
- Remplacer l’utilisation de mots de passe temporaires.
- Remplacer les solutions existantes de gestion des mots de passe en libre-service qui s’appuient sur une solution locale.
- [Éliminer les mots de passe faibles](../authentication/concept-password-ban-bad.md) dans votre organisation.

> [!NOTE]
> Pour les organisations disposant d’un abonnement Azure AD Premium P2, il est recommandé de déployer SSPR et de l’utiliser dans le cadre d’une [stratégie de risque utilisateur Identity Protection](../identity-protection/howto-identity-protection-configure-risk-policies.md).

### <a name="strong-credential-management"></a>Gestion renforcée des informations d’identification

Les mots de passe en eux-mêmes ne sont pas suffisamment sécurisés pour empêcher les acteurs malveillants d’obtenir l’accès à votre environnement. Au minimum, tout utilisateur disposant d’un compte privilégié doit être activé pour l’authentification multifacteur (MFA). Dans l’idéal, vous devez activer l’[inscription combinée](../authentication/concept-registration-mfa-sspr-combined.md) et exiger que tous les utilisateurs s’inscrivent à MFA et à SSPR en utilisant l’[expérience d’inscription combinée](../user-help/security-info-setup-signin.md). Enfin, nous vous recommandons d’adopter une stratégie pour [fournir la résilience](../authentication/concept-resilient-controls.md) de façon à réduire le risque de verrouillage en raison de circonstances imprévues.

![Flux de l’expérience utilisateur combinée](./media/active-directory-ops-guide/active-directory-ops-img4.png)

### <a name="on-premises-outage-authentication-resiliency"></a>Résilience de l’authentification pour des pannes locales

Outre les avantages de la simplicité et de l’activation de la détection des informations d’identification divulguées, la synchronisation du hachage de mot de passe (PHS) d’Azure AD et Azure AD MFA permettent aux utilisateurs d’accéder aux applications SaaS et à Microsoft 365 en dépit des pannes locales dues à des cyberattaques, comme [NotPetya](https://www.microsoft.com/security/blog/2018/02/05/overview-of-petya-a-rapid-cyberattack/). Il est également possible d’activer la synchronisation du hachage de mot de passe en combinaison avec la fédération. L’activation de la synchronisation du hachage de mot de passe permet un basculement de secours de l’authentification quand les services de fédération ne sont pas disponibles.

Si votre organisation locale ne dispose pas d’une stratégie de résilience en cas de panne ou si elle en a une qui n’est pas intégrée à Azure AD, vous devez déployer la synchronisation du hachage de mot de passe Azure AD et définir un plan de reprise d’activité incluant la synchronisation du hachage de mot de passe. L’activation de la synchronisation du hachage de mot de passe d’Azure AD permettra aux utilisateurs de s’authentifier auprès d’Azure AD si votre annuaire Active Directory local n’est pas disponible.

![Flux de la synchronisation du hachage de mot de passe](./media/active-directory-ops-guide/active-directory-ops-img5.png)

Pour mieux comprendre les options d’authentification disponibles, consultez [Choisir la méthode d’authentification adaptée à votre solution d’identité hybride Azure Active Directory](../hybrid/choose-ad-authn.md).

### <a name="programmatic-usage-of-credentials"></a>Utilisation par programmation des informations d’identification

Les scripts Azure AD utilisant PowerShell ou les applications utilisant l’API Microsoft Graph nécessitent une authentification sécurisée. Une mauvaise gestion des informations d’identification pour l’exécution de ces scripts et outils augmente le risque de vol des informations d’identification. Si vous utilisez des scripts ou des applications qui s’appuient sur des mots de passe ou des invites de mot de passe codés en dur, vous devez d’abord examiner les mots de passe dans les fichiers de configuration ou le code source, puis remplacer ces dépendances et utiliser les identités managées par Azure, l’authentification Windows intégrée ou des [certificats](../reports-monitoring/tutorial-access-api-with-certificates.md) quand c’est possible. Pour les applications où les solutions qui précèdent ne sont pas possibles, envisagez d’utiliser [Azure Key Vault](https://azure.microsoft.com/services/key-vault/).

Si vous déterminez qu’il existe des principaux de service avec des informations d’identification de mot de passe, et que vous ne savez pas dans quelle mesure ces informations d’identification de mot de passe sont sécurisées par des scripts ou des applications, contactez le propriétaire de l’application pour mieux comprendre les modèles d’utilisation.

Microsoft vous recommande également de contacter les propriétaires d’application pour comprendre les modèles d’utilisation s’il existe des principaux de service avec des informations d’identification de mot de passe.

## <a name="authentication-experience"></a>Expérience d’authentification

### <a name="on-premises-authentication"></a>Authentification locale

L’authentification fédérée avec l’authentification Windows intégrée ou l’authentification unique (SSO) fluide avec la synchronisation de hachage de mot de passe ou l’authentification directe est la meilleure expérience utilisateur au sein du réseau d’entreprise avec une visibilité sur les contrôleurs de domaine locaux. Elle réduit le nombre de demandes d’informations d’identification et le risque des utilisateurs victimes d’attaques par hameçonnage. Si vous utilisez déjà l’authentification managée par le cloud avec la synchronisation de hachage de mot de passe ou l’authentification directe, mais que les utilisateurs doivent néanmoins toujours de taper leur mot de passe lors de l’authentification locale, vous devez immédiatement [déployer l’authentification unique fluide](../hybrid/how-to-connect-sso.md). D’autre part, si vous êtes actuellement fédéré avec des plans de migration vers l’authentification managée par le cloud, vous devez implémenter l’authentification unique fluide dans le cadre du projet de migration.

### <a name="device-trust-access-policies"></a>Stratégies d’accès approuvé aux appareils

Comme n’importe quel utilisateur de votre organisation, un appareil est une identité fondamentale que vous souhaitez protéger. Vous pouvez utiliser l’identité d’un appareil pour protéger vos ressources à tout moment et à partir de n’importe quel emplacement.  L’authentification de l’appareil et la prise en compte de son type d’approbation améliorent votre attitude et l’utilisabilité en matière de sécurité :

- En évitant les conflits, par exemple avec MFA, quand l’appareil est approuvé
- En bloquant l’accès depuis des appareils non approuvés
- Pour les appareils Windows 10, fournissez l’[authentification unique fluide aux ressources locales](../devices/azuread-join-sso.md).

Pour atteindre cet objectif, vous devez intégrer et gérer les identités des appareils dans Azure AD en utilisant une des méthodes suivantes :

- Les organisations peuvent utiliser [Microsoft Intune](/intune/what-is-intune) pour gérer l’appareil et appliquer des stratégies de conformité, attester l’intégrité de l’appareil et définir des stratégies d’accès conditionnel selon que l’appareil est ou non conforme. Microsoft Endpoint Configuration Manager peut gérer des appareils iOS, des bureaux Mac (via l’intégration JAMF), des bureaux Windows (en mode natif à l’aide de la gestion des appareils mobiles pour Windows 10, et la cogestion avec Microsoft Endpoint Configuration Manager) et les appareils mobiles Android.
- [La jointure Azure AD Hybride](../devices/hybrid-azuread-join-managed-domains.md) permet une gestion avec des stratégies de groupe ou de Microsoft Endpoint Configuration Manager dans un environnement avec des ordinateurs appartenant à un domaine Active Directory. Les organisations peuvent déployer un environnement managé via la synchronisation de hachage de mot de passe ou l’authentification directe avec l’authentification unique fluide. L’intégration de vos appareils à Azure AD optimise la productivité des utilisateurs via l’authentification unique pour vos ressources cloud et locales, tout en vous permettant de sécuriser en même temps l’accès à vos ressources cloud et locales avec l’[accès conditionnel](../conditional-access/overview.md).

Si vous avez des appareils Windows joints à un domaine qui ne sont pas inscrits dans le cloud, ou des appareils Windows joints à un domaine qui sont inscrits dans le cloud mais sans stratégies d’accès conditionnel, vous devez inscrire les appareils non inscrits et, dans les deux cas, [utiliser la jonction Azure AD Hybride en tant que contrôle](../conditional-access/require-managed-devices.md) dans vos stratégies d’accès conditionnel.

![Capture d’écran de l’octroi d’une autorisation dans une stratégie d’accès conditionnel nécessitant un appareil hybride](./media/active-directory-ops-guide/active-directory-ops-img6.png)

Si vous gérez des appareils avec MDM ou Microsoft Intune, mais vous n’utilisez pas de contrôles d’appareils dans vos stratégies d’accès conditionnel, nous vous recommandons d’utiliser [Exiger que l’appareil soit marqué comme conforme](../conditional-access/require-managed-devices.md#require-device-to-be-marked-as-compliant) comme contrôle dans ces stratégies.

![Capture d’écran de l’octroi d’une autorisation dans une stratégie d’accès conditionnel nécessitant la conformité des appareils](./media/active-directory-ops-guide/active-directory-ops-img7.png)

#### <a name="device-trust-access-policies-recommended-reading"></a>Lectures recommandées pour les stratégies d’accès des appareils approuvés

- [Guide pratique pour Planifier l’implémentation de la jonction Azure Active Directory Hybride](../devices/hybrid-azuread-join-plan.md)
- [Configurations des identités et de l’accès aux appareils](/microsoft-365/enterprise/microsoft-365-policies-configurations)

### <a name="windows-hello-for-business"></a>Windows Hello Entreprise

Dans Windows 10, [Windows Hello Entreprise](/windows/security/identity-protection/hello-for-business/hello-identity-verification) remplace les mots de passe par une authentification à deux facteurs forte sur les PC. Windows Hello Entreprise permet une expérience plus fluide de MFA pour les utilisateurs et réduit votre dépendance vis-à-vis des mots de passe. Si vous n’avez pas commencé à déployer des appareils Windows 10 ou si vous ne les avez que partiellement déployés, nous vous recommandons de procéder à une mise à niveau vers Windows 10 et d’[activer Windows Hello Entreprise](/windows/security/identity-protection/hello-for-business/hello-manage-in-organization) sur tous les appareils.

Si vous voulez en savoir plus sur l’authentification sans mot de passe, consultez [Un monde sans mot de passe avec Azure Active Directory](../authentication/concept-authentication-passwordless.md).

## <a name="application-authentication-and-assignment"></a>Authentification et affectation des applications

### <a name="single-sign-on-for-apps"></a>Authentification unique pour les applications

La fourniture d’un mécanisme d’authentification unique standardisé à l’échelle de l’entreprise est essentielle pour une meilleure expérience utilisateur, réduire les risques, la possibilité de signaler et la gouvernance. Si vous utilisez des applications qui prennent en charge l’authentification unique avec Azure AD mais qui sont actuellement configurées pour utiliser des comptes locaux, vous devez reconfigurer ces applications pour qu’elles utilisent l’authentification unique avec Azure AD. De même, si vous utilisez des applications qui prennent en charge l’authentification unique avec Azure AD mais qui utilisent actuellement un autre fournisseur d’identité, vous devez aussi reconfigurer ces applications pour qu’elles utilisent l’authentification unique avec Azure AD. Pour les applications qui ne prennent pas en charge les protocoles de fédération mais qui prennent en charge l’authentification basée sur des formulaires, nous vous recommandons de configurer ces applications pour qu’elles utilisent des [coffres de mots de passe](../manage-apps/application-proxy-configure-single-sign-on-password-vaulting.md) avec le proxy d’application Azure AD.

![Connexion au proxy d’application basée sur des mots de passe](./media/active-directory-ops-guide/active-directory-ops-img8.png)

> [!NOTE]
> Si vous n’avez pas de mécanisme pour découvrir des applications non managées dans votre organisation, nous vous recommandons d’implémenter un processus de découverte en utilisant une solution CASB (Cloud Access Security Broker), comme [Microsoft Cloud App Security](https://www.microsoft.com/enterprise-mobility-security/cloud-app-security).

Enfin, si vous disposez d’une galerie d’applications Azure AD et que vous utilisez des applications qui prennent en charge l’authentification unique avec Azure AD, nous vous recommandons de [placer les applications dans la galerie d’applications](../develop/v2-howto-app-gallery-listing.md).

#### <a name="single-sign-on-recommended-reading"></a>Lectures recommandées pour l’authentification unique

- [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

### <a name="migration-of-ad-fs-applications-to-azure-ad"></a>Migration des applications AD FS vers Azure AD

La [migration des applications depuis AD FS vers Azure AD](../manage-apps/migrate-adfs-apps-to-azure.md) active des fonctionnalités supplémentaires sur la sécurité, une gestion plus cohérente et une meilleure expérience de collaboration. Si vous avez des applications configurées dans AD FS qui prennent en charge l’authentification unique, vous devez les reconfigurer pour qu’elles utilisent l’authentification unique avec Azure AD. Si vous avez des applications configurées dans AD FS avec des configurations peu courantes qui ne sont pas prises en charge par Azure AD, vous devez contacter les propriétaires des applications pour déterminer si la configuration spéciale est une exigence absolue de l’application. Si ce n’est pas le cas, vous devez reconfigurer l’application pour qu’elle utilise l’authentification unique avec Azure AD.

![Azure AD comme fournisseur d’identité principal](./media/active-directory-ops-guide/active-directory-ops-img9.png)

> [!NOTE]
> [Azure AD Connect Health pour ADFS](../hybrid/how-to-connect-health-adfs.md) peut être utilisé pour collecter les détails de configuration de chaque application qui peut potentiellement être migrée vers Azure AD.

### <a name="assign-users-to-applications"></a>Affecter des utilisateurs à des applications

L’[affectation d’utilisateurs à des applications](../manage-apps/assign-user-or-group-access-portal.md) est plus pratique quand des groupes sont utilisés, car ils permettent une plus grande flexibilité et une gestion à grande échelle. Les avantages liés à l’utilisation de groupes sont [l’appartenance dynamique à des groupes en fonction d’attributs](../enterprise-users/groups-dynamic-membership.md) et la [délégation aux propriétaires d’application](../fundamentals/active-directory-accessmanagement-managing-group-owners.md). Par conséquent, si vous utilisez et que vous gérez déjà des groupes, nous vous recommandons d’effectuer les actions suivantes pour améliorer la gestion à grande échelle :

- Déléguez la gestion des groupes et la gouvernance aux propriétaires d’application.
- Autorisez l’accès en libre-service à l’application.
- Définissez des groupes dynamiques si les attributs utilisateur peuvent déterminer de façon cohérente l’accès aux applications.
- Implémentez l’attestation sur les groupes utilisés pour l’accès aux applications en utilisant des [révisions d’accès Azure AD](../governance/access-reviews-overview.md).

En revanche, si vous trouvez des applications auxquelles des utilisateurs individuels sont affectés, veillez à implémenter une [gouvernance](../governance/index.yml) pour ces applications.

#### <a name="assign-users-to-applications-recommended-reading"></a>Lectures recommandées pour l’affectation d’utilisateurs à des applications

- [Affecter des utilisateurs et des groupes à une application dans Azure Active Directory](../manage-apps/assign-user-or-group-access-portal.md)
- [Déléguer des autorisations d’inscription d’application dans Azure Active Directory](../roles/delegate-app-roles.md)
- [Règles d’appartenance dynamique pour les groupes dans Azure Active Directory](../enterprise-users/groups-dynamic-membership.md)

## <a name="access-policies"></a>Stratégies d’accès

### <a name="named-locations"></a>Emplacements nommés

Avec les [emplacements nommés](../reports-monitoring/quickstart-configure-named-locations.md) dans Azure AD, vous pouvez étiqueter des plages d’adresses IP approuvées dans votre organisation. Azure AD utilise des emplacements nommés pour effectuer les actions suivantes :

- Éviter les faux positifs dans les événements à risque. La connexion à partir d’un emplacement réseau approuvé réduit les risques liés à la connexion d’un utilisateur.
- Configurer l’[accès conditionnel en fonction des emplacements](../reports-monitoring/quickstart-configure-named-locations.md).

![Emplacements nommés](./media/active-directory-ops-guide/active-directory-ops-img10.png)

Selon vos priorités, utilisez le tableau ci-dessous pour trouver la solution recommandée qui correspond le mieux aux besoins de votre organisation :

| **Priorité** | **Scénario** | **Recommandation** |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| 1 | Si vous utilisez la synchronisation de hachage de mot de passe ou l’authentification directe, et que des emplacements nommés n’ont pas été définis | Définissez des emplacements nommés pour améliorer la détection des événements à risque |
| 2 | Si vous êtes fédéré et que vous n’utilisez pas la revendication « insideCorporateNetwork » et que des emplacements nommés n’ont pas été définis | Définissez des emplacements nommés pour améliorer la détection des événements à risque |
| 3 | Si vous n’utilisez pas d’emplacements nommés dans les stratégies d’accès conditionnel, et qu’il n’y a pas risque ou de contrôles des appareils dans les stratégies d’accès conditionnel | Configurez la stratégie d’accès conditionnel pour y inclure des emplacements nommés |
| 4 | Si vous êtes fédéré et que vous utilisez la revendication « insideCorporateNetwork », et que des emplacements nommés n’ont pas été définis | Définissez des emplacements nommés pour améliorer la détection des événements à risque |
| 5 | Si vous utilisez des adresses IP approuvées avec MFA au lieu d’emplacements nommés et que vous les marquez comme approuvés | Définissez des emplacements nommés et marquez-les comme approuvés pour améliorer la détection des événements à risque |

### <a name="risk-based-access-policies"></a>Stratégies d’accès basées sur les risques

Azure AD peut calculer le risque pour chaque connexion et pour chaque utilisateur. L’utilisation du risque comme critère dans les stratégies d’accès peut offrir une meilleure expérience utilisateur, par exemple moins d’invites d’authentification, et une meilleure sécurité, par exemple ne demander des informations aux utilisateurs que quand elles sont nécessaires, et automatiser la réponse et la correction.

![Stratégie en matière de risque à la connexion](./media/active-directory-ops-guide/active-directory-ops-img11.png)

Si vous avez déjà des licences Azure AD Premium P2 qui prennent en charge l’utilisation du risque dans les stratégies d’accès, mais qu’elles ne sont pas utilisées, nous vous recommandons fortement d’ajouter le risque à votre gestion de la sécurité.

#### <a name="risk-based-access-policies-recommended-reading"></a>Lectures recommandées pour les stratégies d’accès basées sur le risque

- [Guide pratique pour configurer la stratégie en matière de risque à la connexion](../identity-protection/howto-identity-protection-configure-risk-policies.md)
- [Guide pratique pour configurer la stratégie en matière de risque utilisateur](../identity-protection/howto-identity-protection-configure-risk-policies.md)

### <a name="client-application-access-policies"></a>Stratégies d’accès des applications clientes

La gestion des applications Microsoft Intune (GAM) offre la possibilité d’envoyer des contrôles de protection des données, comme le chiffrement du stockage, un code PIN, le nettoyage du stockage à distance, etc., vers des applications mobiles clientes compatibles, comme Outlook Mobile. En outre, des stratégies d’accès conditionnel peuvent être créées pour [limiter l’accès](../conditional-access/app-based-conditional-access.md) aux services cloud, comme Exchange Online, à partir d’applications approuvées ou compatibles.

Si vos employés installent des applications compatibles GAM, comme des applications mobiles Office, pour accéder à des ressources d’entreprise comme Exchange Online ou SharePoint Online, et que vous prenez également en charge BYOD, nous vous recommandons de déployer des stratégies GAM d’application pour gérer la configuration des applications dans les appareils personnels sans inscription MDM, puis de mettre à jour vos stratégies d’accès conditionnel pour autoriser l’accès seulement à partir de clients compatibles GAM.

![Contrôle de l’octroi de l’accès conditionnel](./media/active-directory-ops-guide/active-directory-ops-img12.png)

Si les employés installent des applications compatibles GAM sur des ressources d’entreprise et que l’accès est limité sur les appareils gérés par Intune, vous devez envisager de déployer des stratégies GAM d’application afin de gérer la configuration des applications pour les appareils personnels, et de mettre à jour les stratégies d’accès conditionnel pour autoriser l’accès seulement à partir de clients compatibles GAM.

### <a name="conditional-access-implementation"></a>Implémentation de l’accès conditionnel

L’accès conditionnel est un outil essentiel pour améliorer l’attitude de votre organisation quand à la sécurité. Par conséquent, il est important de suivre ces bonnes pratiques :

- Vérifier qu’au moins une stratégie est appliquée à chacune des applications SaaS
- Éviter de combiner le filtre **Toutes les applications** avec le contrôle **Bloquer** pour éviter les risques de verrouillage
- Éviter d’utiliser le filtre **Tous les utilisateurs** et d’ajouter par inadvertance **Invités**
- **Migrer toutes les stratégies « héritées » vers le Portail Azure**
- Définir tous les critères pour les utilisateurs, les appareils et les applications
- Utiliser des stratégies d’accès conditionnel pour [implémenter l’authentification MFA](../conditional-access/plan-conditional-access.md) au lieu d’utiliser une **authentification MFA par utilisateur**
- Disposer un petit ensemble de stratégies principales qui peuvent s’appliquer à plusieurs applications
- Définir des groupes d’exceptions vides et les ajouter aux stratégies pour disposer d’une stratégie d’exception
- Prévoir des comptes [d’urgence](../roles/security-planning.md#break-glass-what-to-do-in-an-emergency) sans contrôles MFA
- Veiller à garantir une expérience cohérente entre les applications clientes Microsoft 365 (par exemple Teams, OneDrive, Outlook, etc.) en implémentant le même ensemble de contrôles pour des services comme Exchange Online et SharePoint Online
- L’affectation aux stratégies doit être implémentée via des groupes et non pas via des personnes individuelles
- Effectuez des revues régulières des groupes d’exceptions utilisés dans les stratégies de façon à limiter le temps pendant lequel les utilisateurs sont en dehors de l’attitude de sécurité. Si vous avez Azure AD P2, vous pouvez utiliser les révisions d’accès pour automatiser le processus.

#### <a name="conditional-access-recommended-reading"></a>Lectures recommandées pour l’accès conditionnel

- [Bonnes pratiques relatives à l’accès conditionnel dans Azure Active Directory](../conditional-access/overview.md)
- [Configurations des identités et de l’accès aux appareils](/microsoft-365/enterprise/microsoft-365-policies-configurations)
- [Informations de référence sur les paramètres d’accès conditionnel d’Azure Active Directory](../conditional-access/concept-conditional-access-conditions.md)
- [Stratégies d’accès conditionnel courantes](../conditional-access/concept-conditional-access-policy-common.md)

## <a name="access-surface-area"></a>Surface d’exposition des accès

### <a name="legacy-authentication"></a>Authentification héritée

Des informations d’identification fortes comme MFA ne peuvent pas protéger les applications utilisant des protocoles d’authentification hérités, ce qui en fait le vecteur d’attaque préféré par des acteurs malveillants. Le verrouillage de l’authentification héritée est essentiel pour améliorer la sécurité des accès.

L’authentification héritée est un terme qui fait référence aux protocoles d’authentification utilisés par des applications comme :

- D’anciennes versions de clients Office qui n’utilisent pas l’authentification moderne (par exemple le client Office 2010)
- Des clients qui utilisent des protocoles de messagerie comme IMAP/SMTP/POP

Les attaquants préfèrent nettement ces protocoles : en fait, presque [100 % des attaques par pulvérisation de mot de passe](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Your-Pa-word-doesn-t-matter/ba-p/731984) utilisent des protocoles d’authentification hérités ! Les pirates utilisent des protocoles d’authentification hérités, car ils ne prennent pas en charge la connexion interactive, qui est nécessaire pour les demandes de sécurité supplémentaires, comme l’authentification multifacteur et l’authentification des appareils.

Si l’authentification héritée est largement utilisée dans votre environnement, vous devez planifier la migration des clients hérités vers des clients qui prennent en charge l’[authentification moderne](/office365/enterprise/modern-auth-for-office-2013-and-2016) dès que possible. Dans le même ordre d’idées, si vous avez des utilisateurs qui utilisent déjà une authentification moderne, mais que d’autres qui utilisent encore une authentification héritée, vous devez effectuer les étapes suivantes pour verrouiller les clients avec une authentification héritée :

1. Utilisez les [rapports d’activité de connexion](../reports-monitoring/concept-sign-ins.md) pour identifier les utilisateurs qui utilisent encore une authentification héritée et planifiez une remédiation :

   a. Effectuez une mise à niveau vers des clients compatibles avec une authentification moderne pour les utilisateurs concernés.
   
   b. Planifiez une période de basculement pour effectuer un verrouillage selon les étapes ci-dessous.
   
   c. Identifiez les applications héritées qui ont une dépendance forte vis-à-vis de l’authentification héritée. Consultez l’étape 3 ci-dessous.

2. Désactivez les protocoles hérités à la source (par exemple la boîte aux lettres Exchange) pour les utilisateurs qui n’utilisent pas l’authentification héritée de façon à éviter une plus grande exposition.
3. Pour les comptes restants (dans l’idéal, des identités non humaines comme des comptes de service), utilisez l’[accès conditionnel pour restreindre les protocoles hérités](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-Conditional-Access-support-for-blocking-legacy-auth-is/ba-p/245417) après l’authentification.

#### <a name="legacy-authentication-recommended-reading"></a>Lectures recommandées pour l’authentification héritée

- [Activer ou désactiver l’accès POP3 ou IMAP4 aux boîtes aux lettres dans Exchange Server](/exchange/clients/pop3-and-imap4/configure-mailbox-access)

### <a name="consent-grants"></a>Octrois de consentement

Dans le cadre d’une attaque par octroi de consentement illicite, l’attaquant crée une application inscrite auprès d’Azure AD qui demande l’accès à des données comme des informations de contact, des e-mails ou des documents. Les utilisateurs peuvent octroyer leur consentement à des applications malveillantes via des attaques par hameçonnage en cas d’accès à des sites web malveillants.

Vous trouverez ci-dessous une liste d’applications avec des permissions qu’il peut être nécessaire d’examiner pour les services de cloud computing Microsoft :

- Les applications avec des permissions \*.ReadWrite accordées à l’application ou déléguées
- Les applications avec des permissions déléguées peuvent lire, envoyer ou gérer des e-mails pour le compte de l’utilisateur
- Les applications disposant des permissions suivantes :

| Ressource | Autorisation |
| :- | :- |
| Exchange Online | EAS.AccessAsUser.All |
| | EWS.AccessAsUser.All |
| | Mail.Read |
| API Microsoft Graph | Mail.Read |
| | Mail.Read.Shared |
| | Mail.ReadWrite |

- Les applications ont accordé un emprunt d’identité utilisateur complet de l’utilisateur connecté. Par exemple :

|Ressource | Autorisation |
| :- | :- |
| API Microsoft Graph| Directory.AccessAsUser.All |
| API REST Azure | user_impersonation |

Pour éviter ce scénario, vous devez vous référer à [Détecter et remédier aux octrois de consentement illicites dans Office 365](/office365/securitycompliance/detect-and-remediate-illicit-consent-grants) pour identifier et corriger les applications avec des consentements illicites ou les applications qui ont plus d’autorisations que nécessaire. Ensuite, [supprimez entièrement le libre-service](../manage-apps/configure-user-consent.md) et [établissez des procédures de gouvernance](../manage-apps/configure-admin-consent-workflow.md). Enfin, planifiez des révisions régulières des permissions d’application et supprimez-les lorsqu’elles ne sont pas nécessaires.

#### <a name="consent-grants-recommended-reading"></a>Lectures recommandées pour les octrois de consentement

- [Autorisations pour l’API Microsoft Graph](/graph/permissions-reference)

### <a name="user-and-group-settings"></a>Paramètres des utilisateurs et des groupes

Vous trouverez ci-dessous les paramètres des utilisateurs et des groupes qui peuvent être verrouillés s’ils ne correspondent pas à un besoin métier explicite :

#### <a name="user-settings"></a>Paramètres utilisateur

- **Utilisateurs externes** : une collaboration externe peut se produire de façon naturelle dans l’entreprise avec des services comme Teams, Power BI, SharePoint Online et Azure Information Protection. Si vous avez des contraintes explicites pour contrôler les collaborations externes à l’initiative des utilisateurs, il est recommandé d’activer les utilisateurs externes en utilisant la [gestion des droits d’utilisation d’Azure AD](../governance/entitlement-management-overview.md) ou un fonctionnement contrôlé, par exemple via votre support technique. Si vous ne voulez pas autoriser la collaboration externe naturelle pour les services, vous pouvez [empêcher les membres d’inviter des utilisateurs externes](../external-identities/delegate-invitations.md). Vous pouvez aussi [autoriser ou bloquer des domaines spécifiques](../external-identities/allow-deny-list.md) dans les invitations d’utilisateurs externes.
- **Inscriptions des applications** : quand les inscriptions des applications sont activées, les utilisateurs finaux peuvent intégrer eux-mêmes des applications et accorder l’accès à leurs données. Un exemple classique d’inscription des applications est celui d’utilisateurs autorisant les plug-ins Outlook ou des assistants vocaux, comme Alexa et Siri, à lire leurs e-mails et leur calendrier, ou à envoyer des e-mails en leur nom. Si le client décide de désactiver l’inscription des applications, les équipes InfoSec et IAM doivent être impliquées dans la gestion des exceptions (les inscriptions des applications qui sont nécessaires en fonction des besoins de l’entreprise), car elles doivent inscrire les applications avec un compte d’administrateur, et exigent très probablement la conception d’un processus opérationnel.
- **Portail d’administration** : les organisations peuvent verrouiller le panneau Azure AD dans le portail Azure, afin que les non-administrateurs ne puissent pas accéder à la gestion d’Azure AD dans le portail Azure et ne soient pas perturbés. Accédez aux paramètres utilisateur dans le portail de gestion d’Azure AD pour restreindre l’accès :

![Accès restreint au portail d’administration](./media/active-directory-ops-guide/active-directory-ops-img13.png)

> [!NOTE]
> Les non-administrateurs peuvent néanmoins toujours accéder aux interfaces de gestion d’Azure AD via la ligne de commande et d’autres interfaces de programmation.

#### <a name="group-settings"></a>Suppression de groupe

**Gestion des groupes en libre-service / Les utilisateurs peuvent créer des groupes de sécurité / Groupes Microsoft 365.** S’il n’y a actuellement pas d’initiative en libre-service pour les groupes dans le cloud, les clients peuvent décider de la désactiver jusqu’à ce qu’ils soient prêts à utiliser cette fonctionnalité.

#### <a name="groups-recommended-reading"></a>Lectures recommandées pour les groupes

- [Qu’est-ce qu’Azure Active Directory B2B Collaboration ?](../external-identities/what-is-b2b.md)
- [Intégration d’applications dans Azure Active Directory](../develop/quickstart-register-app.md)
- [Applications, autorisations et consentement dans Azure Active Directory](../develop/quickstart-register-app.md)
- [Utiliser des groupes pour gérer l’accès aux ressources dans Azure Active Directory](./active-directory-manage-groups.md)
- [Configuration de la gestion de l’accès aux applications en libre-service dans Azure Active Directory](../enterprise-users/groups-self-service-management.md)

### <a name="traffic-from-unexpected-locations"></a>Trafic provenant d’emplacements inattendus

Les attaquants proviennent de différentes parties du monde. Gérez ce risque en utilisant des stratégies d’accès conditionnel avec l’emplacement comme condition. La [condition d’emplacement](../conditional-access/location-condition.md) d’une stratégie d’accès conditionnel vous permet de bloquer l’accès pour des emplacements à partir desquels il n’y a aucune raison de se connecter.

![Créer un nouvel emplacement nommé](./media/active-directory-ops-guide/active-directory-ops-img14.png)

Si elle est disponible, utilisez une solution SIEM (Security information and Event Management) pour analyser et rechercher des modèles d’accès entre les régions. Si vous n’utilisez pas un produit SIEM ou si celui-ci n’ingère pas les informations d’authentification provenant d’Azure AD, nous vous recommandons d’utiliser [Azure Monitor](../../azure-monitor/overview.md) pour identifier les modèles d’accès entre les régions.

## <a name="access-usage"></a>Utilisation des accès

### <a name="azure-ad-logs-archived-and-integrated-with-incident-response-plans"></a>Journaux Azure AD archivés et intégrés aux plans de réponse aux incidents

Avoir accès à l’activité de connexion, aux audits et aux événements à risque pour Azure AD est essentiel pour la résolution des problèmes, pour l’analyse de l’utilisation et pour les enquêtes judiciaires. Azure AD permet d’accéder à ces sources via des API REST qui ont une période de conservation limitée. Un système SIEM (Security information and Event Management), ou une technologie d’archivage équivalente, est essentiel pour le stockage à long terme des audits et de la prise en charge. Pour permettre un stockage à long terme des journaux Azure AD, vous devez les ajouter à votre solution SIEM existante ou utiliser [Azure Monitor](../reports-monitoring/concept-activity-logs-azure-monitor.md). Archivez les journaux qui peuvent être utilisés dans le cadre de vos plans de réponse aux incidents et des enquêtes.

#### <a name="logs-recommended-reading"></a>Lectures recommandées pour les journaux

- [Informations de référence sur l’API d’audit Azure Active Directory](/graph/api/resources/directoryaudit?view=graph-rest-beta%3fview%3dgraph-rest-beta)
- [Informations de référence sur l’API de création de rapports sur l’activité de connexion Azure Active Directory](/graph/api/resources/signin?view=graph-rest-beta%3fview%3dgraph-rest-beta)
- [Obtenir des données à l’aide de l’API Génération de rapports Azure AD avec des certificats](../reports-monitoring/tutorial-access-api-with-certificates.md)
- [Microsoft Graph pour Azure Active Directory Identity Protection](../identity-protection/howto-identity-protection-graph-api.md)
- [Informations de référence de l’API Activité de gestion Office 365](/office/office-365-management-api/office-365-management-activity-api-reference)
- [Utilisation du pack de contenu Power BI Azure Active Directory](../reports-monitoring/howto-use-azure-monitor-workbooks.md)

## <a name="summary"></a>Résumé

Une infrastructure d’identité sécurisée a douze aspects. Cette liste vous aide à sécuriser davantage et à gérer les informations d’identification, à définir l’expérience d’authentification, à déléguer l’affectation, à mesurer l’utilisation et à définir des stratégies d’accès en fonction de l’attitude de l’entreprise en matière de sécurité.

- Affectez des propriétaires aux tâches clés.
- Implémentez des solutions pour détecter les mots de passe faibles ou divulgués, pour améliorer la gestion et la protection des mots de passe, et pour sécuriser davantage l’accès des utilisateurs aux ressources.
- Gérez les identités des appareils pour protéger vos ressources à tout moment et à partir de n’importe quel emplacement.
- Implémentez l’authentification sans mot de passe.
- Fournissez un mécanisme d’authentification unique standardisé dans toute l’organisation.
- Migrez les applications de AD FS vers Azure AD pour bénéficier d’une meilleure sécurité et d’une gestion plus cohérente.
- Affectez les utilisateurs à des applications en utilisant des groupes pour permettre une plus grande flexibilité et une gestion à grande échelle.
- Configurez des stratégies d’accès basées sur les risques.
- Verrouillez les protocoles d’authentification hérités.
- Détectez et remédiez aux octrois de consentement illicites.
- Verrouillez les paramètres des utilisateurs et des groupes.
- Activez le stockage à long terme des journaux Azure AD pour le dépannage, l’analyse de l’utilisation et les enquêtes judiciaires.

## <a name="next-steps"></a>Étapes suivantes

Bien démarrer avec les [Vérifications et actions opérationnelles de gouvernance des identités](active-directory-ops-guide-govern.md).
