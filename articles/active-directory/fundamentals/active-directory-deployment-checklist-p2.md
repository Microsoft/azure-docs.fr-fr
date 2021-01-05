---
title: Checklist du déploiement d’Azure AD
description: Checklist du déploiement des fonctionnalités d’Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 10/29/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: martinco
ms.collection: M365-identity-device-management
ms.openlocfilehash: 92d1e5b8ac6492b0b1d819431e4616d32a092cc8
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94836918"
---
# <a name="azure-active-directory-feature-deployment-guide"></a>Guide de déploiement des fonctionnalités d’Azure Active Directory

Il peut être décourageant de déployer Azure Active Directory (Azure AD) pour votre entreprise et en assurer la sécurité. Cet article identifie les tâches courantes que les clients préfèrent exécuter par étapes, sur une durée de 30, 60, 90 jours ou plus, pour améliorer leur sécurité. Même les organisations qui ont déjà déployé Azure AD peuvent utiliser ce guide pour s’assurer de tirer le meilleur parti de leur investissement.

Une infrastructure d’identités correctement planifiée et exécutée offre un accès sécurisé à vos charges de travail et à vos données de productivité seulement aux utilisateurs et appareils connus.

Par ailleurs, les clients peuvent vérifier leur [degré de sécurisation d’identité](identity-secure-score.md) pour voir s’ils sont conformes aux bonnes pratiques de Microsoft. Vérifiez votre degré de sécurisation avant et après l’implémentation de ces recommandations, pour voir si vous faites mieux (ou moins bien) que les autres organisations de votre secteur ou de même taille que la vôtre.

## <a name="prerequisites"></a>Prérequis

La plupart des suggestions de ce guide peuvent être implémentées avec Azure AD Free ou Basic, ou sans licence. Quand une licence est obligatoire, nous indiquons l’édition minimum nécessaire pour accomplir la tâche.

Des informations de licence supplémentaires sont disponibles dans les pages suivantes :

* [Gestion des licences Azure AD](https://azure.microsoft.com/pricing/details/active-directory/)
* [Microsoft 365 Entreprise](https://www.microsoft.com/en-us/licensing/product-licensing/microsoft-365-enterprise)
* [Enterprise Mobility + Security](https://www.microsoft.com/en-us/licensing/product-licensing/enterprise-mobility-security)
* [Tarification d’Azure AD External Identities](../external-identities/external-identities-pricing.md)

## <a name="phase-1-build-a-foundation-of-security"></a>Phase 1 : Créer la base de la sécurité

Dans cette étape, les administrateurs activent des fonctionnalités de sécurité de référence pour créer une base plus sécurisée et facile à utiliser dans Azure AD avant l’importation ou la création de comptes d’utilisateur normaux. Cette étape de base garantit une meilleure sécurité dès le départ et vous permet de présenter les nouveaux concepts une seule fois à vos utilisateurs finaux.

| Tâche | Detail | Licence obligatoire |
| ---- | ------ | ---------------- |
| [Désigner plusieurs administrateurs généraux](../roles/security-emergency-access.md) | Attribuez au moins deux comptes d’administrateur général permanents exclusivement cloud à utiliser en cas d’urgence. Ces comptes ne sont pas utilisés quotidiennement et doivent avoir des mots de passe longs et complexes. | Azure AD Gratuit |
| [Utiliser des rôles d’administrateur non généraux si possible](../roles/permissions-reference.md) | Donnez à vos administrateurs uniquement l’accès dont ils ont besoin et uniquement aux zones auxquelles ils doivent accéder. Les administrateurs ne doivent pas nécessairement tous être des administrateurs généraux. | Azure AD Gratuit |
| [Activer Privileged Identity Management pour suivre l’utilisation du rôle d’administrateur](../privileged-identity-management/pim-getting-started.md) | Activez Privileged Identity Management pour commencer à suivre l’utilisation du rôle d’administrateur. | Azure AD Premium P2 |
| [Déployer la réinitialisation du mot de passe en libre-service](../authentication/howto-sspr-deployment.md) | Réduisez les appels au support technique pour les réinitialisations de mot de passe en autorisant le personnel à réinitialiser leur propre mot de passe à l’aide de stratégies que vous contrôlez en tant qu’administrateur. | |
| [Créer une liste de mots de passe interdits personnalisée propre à l’organisation](../authentication/tutorial-configure-custom-password-protection.md) | Empêchez les utilisateurs de créer des mots de passe comprenant des mots ou des expressions courantes de votre organisation ou de votre zone. | |
| [Activer l’intégration locale à la protection de mot de passe Azure AD](../authentication/concept-password-ban-bad-on-premises.md) | Étendez la liste de mots de passe interdits à votre répertoire local, pour garantir que les mots de passe définis localement sont également conformes à la liste de mots de passe interdits générale et à celle de chaque locataire. | Azure AD Premium P1 |
| [Activer l’aide de Microsoft sur les mots de passe](https://www.microsoft.com/research/publication/password-guidance/) | Cessez de demander aux utilisateurs de changer leur mot de passe selon un calendrier défini et désactivez les exigences de complexité pour que vos utilisateurs soient plus aptes à retenir leur mot de passe et le garde en lieu sûr. | Azure AD Gratuit |
| [Désactiver les réinitialisations de mot de passe périodiques pour les comptes d’utilisateur dans le cloud](../authentication/concept-sspr-policy.md#set-a-password-to-never-expire) | Les réinitialisations de mot de passe périodiques encouragent vos utilisateurs à incrémenter leur mot de passe existant. Utilisez les instructions de la documentation d’aide de Microsoft sur les mots de passe et appliquez votre stratégie locale aux utilisateurs exclusivement dans le cloud. | Azure AD Gratuit |
| [Personnaliser le verrouillage intelligent d’Azure Active Directory](../authentication/howto-password-smart-lockout.md) | Arrêtez la réplication des verrouillages d’utilisateurs cloud sur les utilisateurs Active Directory locaux | |
| [Activer le verrouillage intelligent extranet pour AD FS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-smart-lockout-protection) | Le verrouillage extranet AD FS protège contre les attaques par force brute visant à deviner les mots de passe, tout en permettant aux utilisateurs AD FS valides de continuer à utiliser leur compte. | |
| [Bloquer l’authentification héritée à Microsoft Azure AD avec l’accès conditionnel](../conditional-access/block-legacy-authentication.md) | Bloquez les protocoles d’authentification hérités, tels que POP, SMTP, IMAP et MAPI, qui ne peuvent pas appliquer Multi-Factor Authentication, ce qui en fait un point d’entrée privilégié pour les adversaires. | Azure AD Premium P1 |
| [Déployer l’authentification multifacteur Azure AD à l’aide de stratégies d’accès conditionnel](../authentication/howto-mfa-getstarted.md) | Demandez aux utilisateurs d’utiliser la vérification en deux étapes quand ils accèdent à des applications sensibles à l’aide de stratégies d’accès conditionnel. | Azure AD Premium P1 |
| [Activer Azure Active Directory Identity Protection](../identity-protection/overview-identity-protection.md) | Activez le suivi des connexions risquées et des informations d’identification compromises pour les utilisateurs de votre organisation. | Azure AD Premium P2 |
| [Utiliser les détections de risques pour déclencher l’authentification multifacteur et le changement du mot de passe](../authentication/tutorial-risk-based-sspr-mfa.md) | Activez une automatisation pour déclencher des événements comme l’authentification multifacteur, la réinitialisation du mot de passe et le blocage des connexions en fonction du risque. | Azure AD Premium P2 |
| [Activer l’inscription combinée pour la réinitialisation de mot de passe en libre-service et l’authentification multifacteur Azure AD](../authentication/concept-registration-mfa-sspr-combined.md) | Autorisez vos utilisateurs à s’inscrire à partir d’une même expérience à Azure AD Multi-Factor Authentication et à la réinitialisation de mot de passe en libre-service. | Azure AD Premium P1 |

## <a name="phase-2-import-users-enable-synchronization-and-manage-devices"></a>Phase 2 : Importer des utilisateurs, activer la synchronisation et gérer des appareils

À présent, nous renforçons les fondations posées à l’étape 1 en effectuant les actions suivantes : importer nos utilisateurs et activer la synchronisation, planifier l’accès invité et préparer la prise en charge de fonctionnalités supplémentaires.

| Tâche | Detail | Licence obligatoire |
| ---- | ------ | ---------------- |
| [Installer Azure AD Connect](../hybrid/how-to-connect-install-select-installation.md) | Préparez la synchronisation des utilisateurs de votre annuaire local existant dans le cloud. | Azure AD Gratuit |
| [Implémenter la synchronisation de hachage du mot de passe](../hybrid/how-to-connect-password-hash-synchronization.md) | Synchronisez les hachages de mot de passe pour autoriser la réplication des changements de mot de passe, la détection et la correction de mot de passe incorrect et le signalement des informations d’identification ayant fait l’objet d’une fuite. | Azure AD Gratuit |
| [Implémenter l’écriture différée du mot de passe](../authentication/tutorial-enable-sspr-writeback.md) | Autorisez la réécriture des changements de mot de passe dans le cloud vers un environnement local Windows Server Active Directory. | Azure AD Premium P1 |
| [Implémenter Azure AD Connect Health](../hybrid/whatis-azure-ad-connect.md#what-is-azure-ad-connect-health) | Activez la supervision des principales statistiques d’intégrité pour vos serveurs Azure AD Connect, vos serveurs AD FS et vos contrôleurs de domaine. | Azure AD Premium P1 |
| [Affecter des licences aux utilisateurs par appartenance aux groupes dans Azure Active Directory](../enterprise-users/licensing-groups-assign.md) | Économisez du temps et des efforts en créant des groupes de licences qui activent ou désactivent des fonctionnalités par groupe et non par utilisateur. | |
| [Créer un plan pour l’accès des utilisateurs invités](../external-identities/what-is-b2b.md) | Collaborez avec les utilisateurs invités en les autorisant à se connecter à vos applications et services avec leur propre identité professionnelle, scolaire ou de réseau social. | [Tarification d’Azure AD External Identities](../external-identities/external-identities-pricing.md) |
| [Décider de la stratégie de gestion des appareils](../devices/overview.md) | Choisissez la stratégie de votre organisation concernant les appareils. Inscription ou jointure, BYOD (apportez votre propre appareil) ou appareil fourni par l’entreprise. | |
| [Déployer Windows Hello Entreprise dans votre organisation](/windows/security/identity-protection/hello-for-business/hello-manage-in-organization) | Utilisez Windows Hello pour une authentification sans mot de passe | |
| [Déployer de nouvelles méthodes d’authentification sans mot de passe pour vos utilisateurs](../authentication/concept-authentication-passwordless.md) | Fournir aux utilisateurs des méthodes d’authentification pratiques, sans mot de passe | Azure AD Premium P1 |

## <a name="phase-3-manage-applications"></a>Phase 3 : Gérer les applications

Selon notre approche de renforcement des étapes précédentes, nous identifions les applications candidates pour la migration et l’intégration à Azure AD, et installons ces applications.

| Tâche | Detail | Licence obligatoire |
| ---- | ------ | ---------------- |
| Identifier vos applications | Identifiez les applications utilisées dans votre organisation : les applications locales, les applications SaaS dans le cloud et les autres applications métier. Déterminez si ces applications peuvent et doivent être gérées avec Azure AD. | Aucune licence obligatoire |
| [Intégrer les applications SaaS prises en charge dans la galerie](../manage-apps/add-application-portal.md) | Azure AD a une galerie qui contient des milliers d’applications préintégrées. Certaines applications utilisées par votre organisation sont probablement dans la galerie, accessible directement à partir du portail Azure. | Azure AD Gratuit |
| [Utiliser le proxy d’application pour intégrer les applications locales](../manage-apps/application-proxy-add-on-premises-application.md) | Le proxy d’application permet aux utilisateurs d’accéder aux applications locales en se connectant avec leur compte Azure AD. | |

## <a name="phase-4-audit-privileged-identities-complete-an-access-review-and-manage-user-lifecycle"></a>Phase 4 : Auditer les identités privilégiées, effectuer une révision d’accès et gérer le cycle de vie utilisateur

Dans l’étape 4, les administrateurs appliquent le principe du moindre privilège pour l’administration, les révisions de premier accès et l’automatisation des tâches courantes du cycle de vie utilisateur.

| Tâche | Detail | Licence obligatoire |
| ---- | ------ | ---------------- |
| [Appliquer l’utilisation de Privileged Identity Management](../privileged-identity-management/pim-security-wizard.md) | Supprimez les rôles d’administrateur des comptes d’utilisateur normaux de tous les jours. Autorisez les utilisateurs à utiliser leur rôle d’administrateur s’ils réussissent un contrôle d’authentification multifacteur, s’ils fournissent une justification professionnelle ou s’ils reçoivent l’approbation des approbateurs désignés. | Azure AD Premium P2 |
| [Terminer une révision d’accès des rôles d’annuaire Azure AD dans PIM](../privileged-identity-management/pim-how-to-start-security-review.md) | Consultez vos équipes de sécurité et de direction pour créer une stratégie de révision d’accès afin d’examiner les accès administratifs en fonction des stratégies de votre organisation. | Azure AD Premium P2 |
| [Implémenter des stratégies d’appartenance de groupe dynamique](../enterprise-users/groups-dynamic-membership.md) | Utilisez des groupes dynamiques pour attribuer automatiquement des utilisateurs à des groupes en fonction de leurs attributs dans HR (ou votre source de confiance), comme le service, le poste, la région, et autres. |  |
| [Implémenter le provisionnement d’applications en fonction du groupe](../manage-apps/what-is-access-management.md) | Utilisez le provisionnement de gestion d’accès en fonction du groupe pour provisionner automatiquement des utilisateurs pour les applications SaaS. |  |
| [Automatiser le provisionnement et le déprovisionnement](../app-provisioning/user-provisioning.md) | Supprimez les étapes manuelles du cycle de vie des comptes de vos employés pour empêcher les accès non autorisés. Synchronisez des identités depuis votre source fidèle (système HR) vers Azure AD. |  |

## <a name="next-steps"></a>Étapes suivantes

[Détails des prix et des licences AD Azure](https://azure.microsoft.com/pricing/details/active-directory/)

[Configurations des identités et de l’accès aux appareils](/microsoft-365/enterprise/microsoft-365-policies-configurations)

[Stratégies d’accès courantes et recommandées pour les appareils et identités](/microsoft-365/enterprise/identity-access-policies)