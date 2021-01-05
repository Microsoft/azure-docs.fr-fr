---
title: Répondre rapidement aux identités sécurisées avec Azure Active Directory
description: Répondez rapidement aux menaces avec les identités Azure AD basées sur le cloud.
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 04/27/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: davidspo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5c4301a61a79ab2351c18af0c76cccc3d07dd202
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94836680"
---
# <a name="rapidly-respond-to-secure-identities-with-azure-ad"></a>Répondre rapidement aux identités sécurisées avec Azure AD

Dans le monde actuel, il peut sembler très difficile de sécuriser les employés, en particulier lorsque vous devez fournir rapidement l’accès à de nombreux services. Cet article fournit une liste concise de toutes les actions à entreprendre en vue de déterminer et de hiérarchiser l’ordre de déploiement des fonctionnalités Azure AD, selon le type de licence dont vous disposez. Azure AD propose un grand nombre de fonctionnalités et fournit de nombreuses couches de sécurité pour vos identités. Cependant, la recherche de la fonctionnalité la mieux adaptée à vos besoins peut prendre du temps. De nombreuses organisations sont déjà passées au cloud ou sont en phase de transition. Ce document est conçu pour vous aider à déployer des services rapidement, en donnant la priorité à la sécurisation de vos identités. 

Chaque table fournit une recommandation de sécurité qui s’applique aussi bien aux identités administrateurs qu’aux identités utilisateurs. Ces recommandations ont pour but de vous protéger des principales attaques de sécurité (attaques par rejeu, hameçonnage, attaques par pulvérisation de mots de passe) tout en réduisant l’impact utilisateur et en améliorant l’expérience utilisateur. 

Les instructions permettront aux administrateurs de configurer l’accès aux applications SaaS et locales de manière sécurisée. En outre, elles s’appliquent également aux identités cloud, aux identités hybrides (synchronisées), ainsi qu’aux utilisateurs travaillant à distance ou au bureau.

Cette liste de vérification vous aidera à déployer rapidement les actions recommandées critiques pour protéger votre organisation immédiatement et explique comment :

- Renforcer vos informations d’identification.
- Réduire votre surface d’attaque.
- Automatiser la réponse aux menaces.
- Utiliser l’intelligence cloud.
- Activer le libre-service pour l’utilisateur final.

## <a name="prerequisites"></a>Prérequis

Ce guide part du principe que vos identités cloud ou hybrides ont déjà été créées dans Azure AD. Pour savoir quel type d’identité choisir, consultez [Choisir la méthode d’authentification adaptée à votre solution d’identité hybride Azure Active Directory](../hybrid/choose-ad-authn.md). 

## <a name="summary"></a>Résumé

Les éléments qui assurent la sécurisation d’une infrastructure d’identité sont nombreux. Cependant, la liste de vérification qui suit se concentre sur la sécurisation des utilisateurs qui travaillent à distance. La sécurisation de votre identité ne constitue qu’une partie de votre sécurité globale. Vous devez également penser à la protection des données, des applications et des appareils.

### <a name="guidance-for-azure-ad-free-office-365-or-microsoft-365-customers"></a>Instructions pour les clients Azure AD Free, Office 365 ou Microsoft 365.

Les utilisateurs de l’application Azure AD Free, Office 365 ou Microsoft 365 doivent appliquer un certain nombre de recommandations pour protéger leurs identités utilisateur. Le tableau fournit les principales actions à entreprendre pour les abonnements de licence suivants :

- Office 365 (Office 365 E1, E3, E5, F1, A1, A3, A5)
- Microsoft 365 (Business Basic, Applications pour les entreprises, Business standard, Business Premium, A1)
- Azure AD Free (inclus avec Azure, Dynamics 365, Intune et Power Platform)

| Action recommandée | Detail |
| --- | --- |
| [Activer les paramètres de sécurité par défaut](concept-fundamentals-security-defaults.md) | Protéger toutes les identités et les applications des utilisateurs en activant MFA et en bloquant l’authentification héritée |
| [Activer la synchronisation du hachage de mot de passe](../hybrid/how-to-connect-password-hash-synchronization.md) (si vous utilisez des identités hybrides) | Fournir une redondance pour l’authentification et améliorer la sécurité (verrouillage intelligent, verrouillage IP et possibilité de détecter les fuites d’informations d’identification). |
| [Activer le verrouillage intelligent ADFS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-smart-lockout-protection) (le cas échéant) | Protège les utilisateurs contre le verrouillage de leur compte extranet dû à une activité malveillante. |
| [Activer le verrouillage intelligent Azure Active Directory](../authentication/howto-password-smart-lockout.md) (si vous utilisez des identités managées) | Le verrouillage intelligent empêche les personnes malveillantes de deviner vos mots de passe ou d’utiliser des méthodes de force brute pour rentrer dans vos systèmes. |
| [Désactiver le consentement utilisateur pour une application](../manage-apps/configure-user-consent.md) | Le workflow de consentement administrateur offre aux administrateurs une méthode sécurisée pour accorder l’accès aux applications qui nécessitent l’approbation de l’administrateur, ce qui permet d’éviter que les utilisateurs finaux n’exposent les données de l’entreprise. Microsoft vous recommande de désactiver toutes les opérations futures de consentement d’utilisateurs pour aider à réduire votre surface d’exposition et à atténuer ce risque. |
| [Intégrer des applications SaaS prises en charge à Azure AD à partir de la galerie et activer l’authentification unique](../manage-apps/add-application-portal.md) | Azure AD a une galerie qui contient des milliers d’applications préintégrées. Certaines applications utilisées par votre organisation sont probablement dans la galerie, accessible directement à partir du portail Azure. Fournir un accès à distance aux applications SaaS d’entreprise de façon sécurisée avec une expérience utilisateur améliorée (SSO) |
| [Automatiser l’attribution et la désattribution des utilisateurs dans les applications SaaS](../app-provisioning/user-provisioning.md) (le cas échéant) | Créez automatiquement des identités et des rôles d’utilisateur dans les applications cloud (SaaS) auxquelles les utilisateurs doivent accéder. En plus de créer des identités utilisateur, l’attribution automatique d’utilisateurs comprend la maintenance et la suppression des identités utilisateur en cas de modification de l’état ou des rôles, ce qui accroît le niveau de sécurité de votre organisation. |
| [Activer l’accès hybride sécurisé : Sécuriser les applications héritées avec les réseaux et les contrôleurs de livraison d’applications existants](../manage-apps/secure-hybrid-access.md) (le cas échéant) | Publiez et protégez vos applications d’authentification locales et cloud héritées en les connectant à Azure AD avec votre contrôleur ou votre réseau de livraison d’application. |
| [Activer la réinitialisation de mot de passe en libre-service](../authentication/tutorial-enable-sspr.md) (applicable aux comptes cloud uniquement) | Cette fonctionnalité réduit le nombre d’appels au support technique, ainsi que la perte de productivité, quand l’utilisateur ne parvient pas à se connecter à son appareil ou à une application. |
| [Utiliser des rôles d’administrateur non généraux si possible](../roles/permissions-reference.md) | Donnez à vos administrateurs uniquement l’accès dont ils ont besoin et uniquement aux zones auxquelles ils doivent accéder. Les administrateurs ne doivent pas nécessairement tous être des administrateurs généraux. |
| [Activer l’aide de Microsoft sur les mots de passe](https://www.microsoft.com/research/publication/password-guidance/) | Cessez de demander aux utilisateurs de changer leur mot de passe selon un calendrier défini et désactivez les exigences de complexité pour que vos utilisateurs soient plus aptes à retenir leur mot de passe et le garde en lieu sûr. |


### <a name="guidance-for-azure-ad-premium-plan-1-customers"></a>Instructions pour les clients Azure AD Premium Plan 1.

Le tableau fournit les principales actions à entreprendre pour les abonnements de licence suivants :

- Azure Active Directory Premium P1 (Azure AD P1)
- Enterprise Mobility + Security (EMS E3)
- Microsoft 365 (M365 E3, A3, F1, F3)

| Action recommandée | Detail |
| --- | --- |
| [Activer l’expérience d’inscription combinée pour Azure AD MFA et SSPR afin de simplifier l’inscription des utilisateurs](../authentication/howto-registration-mfa-sspr-combined.md) | Autorisez vos utilisateurs à s’inscrire à partir d’une même expérience à Azure AD Multi-Factor Authentication et à la réinitialisation de mot de passe en libre-service. |
| [Configurer les paramètres MFA pour votre organisation](../authentication/howto-mfa-getstarted.md) | Garantissez la protection des comptes contre toute compromission avec l’authentification multifacteur. |
| [Activer la réinitialisation du mot de passe libre-service](../authentication/tutorial-enable-sspr.md) | Cette fonctionnalité réduit le nombre d’appels au support technique, ainsi que la perte de productivité, quand l’utilisateur ne parvient pas à se connecter à son appareil ou à une application. |
| [Implémenter la réécriture du mot de passe](../authentication/tutorial-enable-sspr-writeback.md) (si vous utilisez des identités hybrides) | Autorisez la réécriture des changements de mot de passe dans le cloud vers un environnement local Windows Server Active Directory. |
| Créer et activer des stratégies d’accès conditionnel | [MFA pour les administrateurs en vue de protéger les comptes auxquels sont attribués des droits d’administration.](../conditional-access/howto-conditional-access-policy-admin-mfa.md) <br><br> [Bloquez les protocoles d’authentification hérités en raison des risques accrus associés aux protocoles d’authentification hérités.](../conditional-access/howto-conditional-access-policy-block-legacy.md) <br><br> [MFA pour l’ensemble des utilisateurs et des applications en vue de créer une stratégie d’authentification multifacteur équilibrée pour votre environnement, en sécurisant vos utilisateurs et vos applications.](../conditional-access/howto-conditional-access-policy-all-users-mfa.md) <br><br> [Exigez MFA pour la gestion Azure afin de protéger vos ressources privilégiées en exigeant une authentification multifacteur pour tous les utilisateurs qui accèdent aux ressources Azure.](../conditional-access/howto-conditional-access-policy-azure-management.md) |
| [Activer la synchronisation du hachage de mot de passe](../hybrid/how-to-connect-password-hash-synchronization.md) (si vous utilisez des identités hybrides) | Fournir une redondance pour l’authentification et améliorer la sécurité (verrouillage intelligent, verrouillage IP et possibilité de détecter les fuites d’informations d’identification). |
| [Activer le verrouillage intelligent ADFS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-smart-lockout-protection) (le cas échéant) | Protège les utilisateurs contre le verrouillage de leur compte extranet dû à une activité malveillante. |
| [Activer le verrouillage intelligent Azure Active Directory](../authentication/howto-password-smart-lockout.md) (si vous utilisez des identités managées) | Le verrouillage intelligent empêche les personnes malveillantes de deviner vos mots de passe ou d’utiliser des méthodes de force brute pour rentrer dans vos systèmes. |
| [Désactiver le consentement utilisateur pour une application](../manage-apps/configure-user-consent.md) | Le workflow de consentement administrateur offre aux administrateurs une méthode sécurisée pour accorder l’accès aux applications qui nécessitent l’approbation de l’administrateur, ce qui permet d’éviter que les utilisateurs finaux n’exposent les données de l’entreprise. Microsoft vous recommande de désactiver toutes les opérations futures de consentement d’utilisateurs pour aider à réduire votre surface d’exposition et à atténuer ce risque. |
| [Offrir un accès à distance aux applications locales héritées avec le proxy d’application](../manage-apps/application-proxy-add-on-premises-application.md) | Activez le proxy d’application Azure AD et intégrez-le aux applications héritées pour permettre aux utilisateurs d’accéder de façon sécurisée aux applications locales en se connectant avec leur compte Azure AD. |
| [Activer l’accès hybride sécurisé : Sécuriser les applications héritées avec les réseaux et les contrôleurs de livraison d’applications existants](../manage-apps/secure-hybrid-access.md) (le cas échéant) | Publiez et protégez vos applications d’authentification locales et cloud héritées en les connectant à Azure AD avec votre contrôleur ou votre réseau de livraison d’application. |
| [Intégrer des applications SaaS prises en charge à Azure AD à partir de la galerie et activer l’authentification unique](../manage-apps/add-application-portal.md) | Azure AD a une galerie qui contient des milliers d’applications préintégrées. Certaines applications utilisées par votre organisation sont probablement dans la galerie, accessible directement à partir du portail Azure. Fournir un accès à distance aux applications SaaS d’entreprise de façon sécurisée avec une expérience utilisateur améliorée (SSO) |
| [Automatiser l’attribution et la désattribution des utilisateurs dans les applications SaaS](../app-provisioning/user-provisioning.md) (le cas échéant) | Créez automatiquement des identités et des rôles d’utilisateur dans les applications cloud (SaaS) auxquelles les utilisateurs doivent accéder. En plus de créer des identités utilisateur, l’attribution automatique d’utilisateurs comprend la maintenance et la suppression des identités utilisateur en cas de modification de l’état ou des rôles, ce qui accroît le niveau de sécurité de votre organisation. |
| [Activer l’accès conditionnel - Basé sur l’appareil](../conditional-access/require-managed-devices.md) | Améliorez la sécurité et l’expérience utilisateur avec l’accès conditionnel basé sur l’appareil. Cette étape garantit que les utilisateurs ne pourront accéder aux ressources qu’à partir d’appareils qui répondent à vos standards de sécurité et de conformité. Ces appareils sont également appelés « appareils gérés ». Les appareils gérés peuvent être des appareils conformes à Intune ou joints à Azure AD Hybride. |
| [Activer la Protection par mot de passe](../authentication/howto-password-ban-bad-on-premises-deploy.md) | Empêchez les utilisateurs d’utiliser des mots de passe faibles et faciles à deviner. |
| [Désigner plusieurs administrateurs généraux](../roles/security-emergency-access.md) | Attribuez au moins deux comptes d’administrateur général permanents exclusivement cloud à utiliser en cas d’urgence. Ces comptes ne sont pas utilisés quotidiennement et doivent avoir des mots de passe longs et complexes. Les comptes « Break Glass » vous permettent d’accéder au service en cas d’urgence. |
| [Utiliser des rôles d’administrateur non généraux si possible](../roles/permissions-reference.md) | Donnez à vos administrateurs uniquement l’accès dont ils ont besoin et uniquement aux zones auxquelles ils doivent accéder. Les administrateurs ne doivent pas nécessairement tous être des administrateurs généraux. |
| [Activer l’aide de Microsoft sur les mots de passe](https://www.microsoft.com/research/publication/password-guidance/) | Cessez de demander aux utilisateurs de changer leur mot de passe selon un calendrier défini et désactivez les exigences de complexité pour que vos utilisateurs soient plus aptes à retenir leur mot de passe et le garde en lieu sûr. |
| [Créer un plan pour l’accès des utilisateurs invités](../external-identities/what-is-b2b.md) | Collaborez avec les utilisateurs invités en les autorisant à se connecter à vos applications et services avec leur propre identité professionnelle, scolaire ou sociale. |

### <a name="guidance-for-azure-ad-premium-plan-2-customers"></a>Instructions pour les clients Azure AD Premium Plan 2.

Le tableau fournit les principales actions à entreprendre pour les abonnements de licence suivants :

- Azure Active Directory Premium P2 (Azure AD P2)
- Enterprise Mobility + Security (EMS E5)
- Microsoft 365 (M365 E5, A5)

| Action recommandée | Detail |
| --- | --- |
| [Activer l’expérience d’inscription combinée pour Azure AD MFA et SSPR afin de simplifier l’inscription des utilisateurs](../authentication/howto-registration-mfa-sspr-combined.md) | Autorisez vos utilisateurs à s’inscrire à partir d’une même expérience à Azure AD Multi-Factor Authentication et à la réinitialisation de mot de passe en libre-service. |
| [Configurer les paramètres MFA pour votre organisation](../authentication/howto-mfa-getstarted.md) | Garantissez la protection des comptes contre toute compromission avec l’authentification multifacteur. |
| [Activer la réinitialisation du mot de passe libre-service](../authentication/tutorial-enable-sspr.md) | Cette fonctionnalité réduit le nombre d’appels au support technique, ainsi que la perte de productivité, quand l’utilisateur ne parvient pas à se connecter à son appareil ou à une application. |
| [Implémenter la réécriture du mot de passe](../authentication/tutorial-enable-sspr-writeback.md) (si vous utilisez des identités hybrides) | Autorisez la réécriture des changements de mot de passe dans le cloud vers un environnement local Windows Server Active Directory. |
| [Activer les stratégies de protection des identités pour appliquer l’inscription MFA](../identity-protection/howto-identity-protection-configure-mfa-policy.md) | Gérez le déploiement d’Azure AD Multi-Factor Authentication (MFA). |
| [Activer les stratégies de risques liés aux utilisateurs et aux connexions d’Identity Protection](../identity-protection/howto-identity-protection-configure-risk-policies.md) | Activez les stratégies relatives aux utilisateurs et aux connexions d’Identity Protection. La stratégie de connexion recommandée consiste à cibler des connexions à risque moyen et à exiger l’authentification MFA. Pour les stratégies d’utilisateur, il faut cibler les utilisateurs à risque élevé qui nécessitent un changement de mot de passe. |
| Créer et activer des stratégies d’accès conditionnel | [MFA pour les administrateurs en vue de protéger les comptes auxquels sont attribués des droits d’administration.](../conditional-access/howto-conditional-access-policy-admin-mfa.md) <br><br> [Bloquez les protocoles d’authentification hérités en raison des risques accrus associés aux protocoles d’authentification hérités.](../conditional-access/howto-conditional-access-policy-block-legacy.md) <br><br> [Exigez MFA pour la gestion Azure afin de protéger vos ressources privilégiées en exigeant une authentification multifacteur pour tous les utilisateurs qui accèdent aux ressources Azure.](../conditional-access/howto-conditional-access-policy-azure-management.md) |
| [Activer la synchronisation du hachage de mot de passe](../hybrid/how-to-connect-password-hash-synchronization.md) (si vous utilisez des identités hybrides) | Fournir une redondance pour l’authentification et améliorer la sécurité (verrouillage intelligent, verrouillage IP et possibilité de détecter les fuites d’informations d’identification). |
| [Activer le verrouillage intelligent ADFS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-smart-lockout-protection) (le cas échéant) | Protège les utilisateurs contre le verrouillage de leur compte extranet dû à une activité malveillante. |
| [Activer le verrouillage intelligent Azure Active Directory](../authentication/howto-password-smart-lockout.md) (si vous utilisez des identités managées) | Le verrouillage intelligent empêche les personnes malveillantes de deviner vos mots de passe ou d’utiliser des méthodes de force brute pour rentrer dans vos systèmes. |
| [Désactiver le consentement utilisateur pour une application](../manage-apps/configure-user-consent.md) | Le workflow de consentement administrateur offre aux administrateurs une méthode sécurisée pour accorder l’accès aux applications qui nécessitent l’approbation de l’administrateur, ce qui permet d’éviter que les utilisateurs finaux n’exposent les données de l’entreprise. Microsoft vous recommande de désactiver toutes les opérations futures de consentement d’utilisateurs pour aider à réduire votre surface d’exposition et à atténuer ce risque. |
| [Offrir un accès à distance aux applications locales héritées avec le proxy d’application](../manage-apps/application-proxy-add-on-premises-application.md) | Activez le proxy d’application Azure AD et intégrez-le aux applications héritées pour permettre aux utilisateurs d’accéder de façon sécurisée aux applications locales en se connectant avec leur compte Azure AD. |
| [Activer l’accès hybride sécurisé : Sécuriser les applications héritées avec les réseaux et les contrôleurs de livraison d’applications existants](../manage-apps/secure-hybrid-access.md) (le cas échéant) | Publiez et protégez vos applications d’authentification locales et cloud héritées en les connectant à Azure AD avec votre contrôleur ou votre réseau de livraison d’application. |
| [Intégrer des applications SaaS prises en charge à Azure AD à partir de la galerie et activer l’authentification unique](../manage-apps/add-application-portal.md) | Azure AD a une galerie qui contient des milliers d’applications préintégrées. Certaines applications utilisées par votre organisation sont probablement dans la galerie, accessible directement à partir du portail Azure. Fournir un accès à distance aux applications SaaS d’entreprise de façon sécurisée avec une expérience utilisateur améliorée (SSO) |
| [Automatiser l’attribution et la désattribution des utilisateurs dans les applications SaaS](../app-provisioning/user-provisioning.md) (le cas échéant) | Créez automatiquement des identités et des rôles d’utilisateur dans les applications cloud (SaaS) auxquelles les utilisateurs doivent accéder. En plus de créer des identités utilisateur, l’attribution automatique d’utilisateurs comprend la maintenance et la suppression des identités utilisateur en cas de modification de l’état ou des rôles, ce qui accroît le niveau de sécurité de votre organisation. |
| [Activer l’accès conditionnel - Basé sur l’appareil](../conditional-access/require-managed-devices.md) | Améliorez la sécurité et l’expérience utilisateur avec l’accès conditionnel basé sur l’appareil. Cette étape garantit que les utilisateurs ne pourront accéder aux ressources qu’à partir d’appareils qui répondent à vos standards de sécurité et de conformité. Ces appareils sont également appelés « appareils gérés ». Les appareils gérés peuvent être des appareils conformes à Intune ou joints à Azure AD Hybride. |
| [Activer la Protection par mot de passe](../authentication/howto-password-ban-bad-on-premises-deploy.md) | Empêchez les utilisateurs d’utiliser des mots de passe faibles et faciles à deviner. |
| [Désigner plusieurs administrateurs généraux](../roles/security-emergency-access.md) | Attribuez au moins deux comptes d’administrateur général permanents exclusivement cloud à utiliser en cas d’urgence. Ces comptes ne sont pas utilisés quotidiennement et doivent avoir des mots de passe longs et complexes. Les comptes « Break Glass » vous permettent d’accéder au service en cas d’urgence. |
| [Utiliser des rôles d’administrateur non généraux si possible](../roles/permissions-reference.md) | Donnez à vos administrateurs uniquement l’accès dont ils ont besoin et uniquement aux zones auxquelles ils doivent accéder. Les administrateurs ne doivent pas nécessairement tous être des administrateurs généraux. |
| [Activer l’aide de Microsoft sur les mots de passe](https://www.microsoft.com/research/publication/password-guidance/) | Cessez de demander aux utilisateurs de changer leur mot de passe selon un calendrier défini et désactivez les exigences de complexité pour que vos utilisateurs soient plus aptes à retenir leur mot de passe et le garde en lieu sûr. |
| [Créer un plan pour l’accès des utilisateurs invités](../external-identities/what-is-b2b.md) | Collaborez avec les utilisateurs invités en les autorisant à se connecter à vos applications et services avec leur propre identité professionnelle, scolaire ou sociale. |
| [Activer Privileged Identity Management](../privileged-identity-management/pim-configure.md) | Permet de gérer, de contrôler et de superviser l’accès aux ressources importantes de votre organisation, en faisant en sorte que les administrateurs y auront accès uniquement si nécessaire et après approbation. |

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir des instructions détaillées sur le déploiement de chaque fonctionnalité Azure AD, passez en revue les [plans de déploiement de projet Azure AD](active-directory-deployment-plans.md).

- Pour obtenir une liste de vérification pour un déploiement Azure AD de bout en bout, consultez [Guide de déploiement des fonctionnalités Azure Active Directory](active-directory-deployment-checklist-p2.md).