---
title: Autorisations utilisateur par défaut - Azure Active Directory | Microsoft Docs
description: Apprenez-en plus sur les différentes autorisations d’utilisateur disponibles dans Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 08/17/2020
ms.author: ajburnle
ms.reviewer: vincesm
ms.custom: it-pro, seodec18, contperf-fy21q1
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2c556940ace02f2e7bf0354e67d32df7c5c6714e
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032760"
---
# <a name="what-are-the-default-user-permissions-in-azure-active-directory"></a>Quelles sont les autorisations d’utilisateur par défaut dans Azure Active Directory ?
Dans Azure Active Directory (Azure AD), tous les utilisateurs bénéficient d’un jeu d’autorisations par défaut. L’accès d’un utilisateur se compose du type d’utilisateur, de ses [attributions de rôles](active-directory-users-assign-role-azure-portal.md) et de sa possession d’objets individuels. Cet article décrit ces autorisations par défaut et compare celles des utilisateurs membres et celles des utilisateurs invités. Les autorisations d’utilisateur par défaut ne peuvent être modifiées que dans les paramètres utilisateur dans Azure AD.

## <a name="member-and-guest-users"></a>Utilisateurs membres et utilisateurs invités
Le jeu d'autorisations par défaut reçu varie selon que l'utilisateur est membre natif du locataire (utilisateur membre) ou transféré depuis un autre répertoire en tant qu'invité de collaboration B2B (utilisateur invité). Pour plus d'informations sur l'ajout d'utilisateurs invités, consultez [Qu'est-ce que la collaboration B2B d'Azure AD](../external-identities/what-is-b2b.md).
* Les utilisateurs membres peuvent inscrire des applications, gérer leurs numéro de téléphone mobile et photo de profil, changer leur mot de passe et inviter des invités B2B. En outre, les utilisateurs peuvent lire toutes les informations d’annuaire (à quelques exceptions près). 
* Les utilisateurs invités disposent d'autorisations d'annuaire limitées. Ils peuvent gérer leur propre profil, modifier leur propre mot de passe et récupérer des informations sur d’autres utilisateurs, groupes et applications, mais ils ne peuvent pas lire toutes les informations de l’annuaire. Par exemple, des utilisateurs invités ne peuvent pas énumérer la liste des utilisateurs, des groupes et d’autres objets d’annuaire. Les invités peuvent être ajoutés à un rôle d’administrateur, bénéficiant ainsi des autorisations de lecture et d’écriture complètes contenues dans le rôle. Les invités peuvent également inviter d’autres invités.

## <a name="compare-member-and-guest-default-permissions"></a>Comparer les autorisations par défaut des invités et des membres

**Zone** | **Autorisations d’un utilisateur membre** | **Autorisations d’utilisateur invité par défaut** | **Autorisations d’utilisateur invité restreintes (préversion)**
------------ | --------- | ---------- | ----------
Utilisateurs et contacts | <ul><li>Énumérer la liste de tous les utilisateurs et contacts<li>Lire toutes les propriétés publiques des utilisateurs et des contacts</li><li>Inviter des invités<li>Changer son propre mot de passe<li>Gérer son propre numéro de téléphone mobile<li>Gérer sa propre photo<li>Invalider ses propres jetons d’actualisation</li></ul> | <ul><li>Lire ses propres propriétés<li>Lire le nom d’affichage, l’e-mail, le nom de connexion, la photo, le nom d’utilisateur principal et les propriétés de type d’utilisateur des autres utilisateurs et contacts<li>Changer son propre mot de passe<li>Rechercher un autre utilisateur par ObjectId (si autorisé)<li>Lire les informations sur le gestionnaire et le rapport direct d’autres utilisateurs</li></ul> | <ul><li>Lire ses propres propriétés<li>Changer son propre mot de passe</li></ul>
Groupes | <ul><li>Créer des groupes de sécurité<li>Créer des groupes Microsoft 365<li>Énumérer la liste de tous les groupes<li>Lire toutes les propriétés des groupes<li>Lire les appartenances aux groupes non masquées<li>Lire les appartenances aux groupes Microsoft 365 masquées pour un groupe rejoint<li>Gérer les propriétés, l’appartenance et l’adhésion des groupes que possède l’utilisateur<li>Ajouter des invités aux groupes acquis<li>Gérer les paramètres d’appartenance dynamique<li>Supprimer des groupes acquis<li>Restaurer les groupes Microsoft 365 détenus</li></ul> | <ul><li>Lire les propriétés des groupes non masqués, y compris l’appartenance et la propriété (même de groupes non joints)<li>Lire les appartenances aux groupes Microsoft 365 masquées pour les groupes rejoints<li>Rechercher des groupes par nom d’affichage ou ObjectId (si autorisé)</li></ul> | <ul><li>Lire des ID d’objet pour des groupes joints<li>Lire l’appartenance et la propriété des groupes joints dans certaines applications Microsoft 365 (si autorisé)</li></ul>
Applications | <ul><li>Inscrire (créer) une application<li>Énumérer la liste de toutes les applications<li>Lire les propriétés des applications inscrites et d’entreprise<li>Gérer les propriétés d’application, les affectations et les informations d’identification des applications acquises<li>Créer ou supprimer le mot de passe d’application pour un utilisateur<li>Supprimer des applications acquises<li>Restaurer des applications acquises</li></ul> | <ul><li>Lire les propriétés des applications inscrites et d’entreprise</li></ul> | <ul><li>Lire les propriétés des applications inscrites et d’entreprise
Périphériques</li></ul> | <ul><li>Énumérer la liste de tous les appareils<li>Lire toutes les propriétés des appareils<li>Gérer toutes les propriétés des appareils acquis</li></ul> | Aucune autorisation | Aucune autorisation
Répertoire | <ul><li>Lire toutes les informations de l’entreprise<li>Lire tous les domaines<li>Lire tous les contrats de partenaire</li></ul> | <ul><li>Lire le nom complet de l’entreprise<li>Lire tous les domaines</li></ul> | <ul><li>Lire le nom complet de l’entreprise<li>Lire tous les domaines</li></ul>
Rôles et étendues | <ul><li>Lire tous les rôles d’administrateur et toutes les appartenances<li>Lire toutes les propriétés et l’appartenance des unités administratives</li></ul> | Aucune autorisation | Aucune autorisation
Abonnements | <ul><li>Lire tous les abonnements<li>Activer le membre de plan de service</li></ul> | Aucune autorisation | Aucune autorisation
Stratégies | <ul><li>Lire toutes les propriétés des stratégies<li>Gérer toutes les propriétés d’une stratégie acquise</li></ul> | Aucune autorisation | Aucune autorisation

## <a name="restrict-member-users-default-permissions"></a>Restreindre les autorisations par défaut des utilisateurs membres 

Vous pouvez restreindre les autorisations par défaut des utilisateurs membres comme suit :

Autorisation | Explication du paramètre
---------- | ------------
Les utilisateurs peuvent inscrire l’application | Définir cette option sur Non empêche les utilisateurs de créer des inscriptions d’applications. La capacité peut ensuite être redonnée à des personnes spécifiques en les ajoutant au rôle Développeur d’applications.
Autoriser les utilisateurs à connecter un compte professionnel ou scolaire avec LinkedIn | Définir cette option sur Non empêche les utilisateurs de connecter leur compte professionnel ou scolaire avec leur compte LinkedIn. Pour plus d’informations, voir [Consentement et partage de données connexions de compte LinkedIn](../enterprise-users/linkedin-user-consent.md).
Possibilité de créer des groupes de sécurité | Définir cette option sur Non empêche les utilisateurs de créer des groupes de sécurité. Les administrateurs généraux et les administrateurs d’utilisateurs peuvent toujours créer des groupes de sécurité. Pour connaître la marche à suivre, consultez [Configuration des paramètres de groupe avec les applets de commande Azure Active Directory](../enterprise-users/groups-settings-cmdlets.md).
Possibilité de créer des groupes Microsoft 365 | Définir cette option sur Non empêche les utilisateurs de créer des groupes Microsoft 365. Définir cette option sur Certain(e)s permet à un ensemble d’utilisateurs spécifique de créer des groupes Microsoft 365. Les administrateurs généraux et les administrateurs d’utilisateurs peuvent toujours créer des groupes Microsoft 365. Pour connaître la marche à suivre, consultez [Configuration des paramètres de groupe avec les applets de commande Azure Active Directory](../enterprise-users/groups-settings-cmdlets.md).
Limiter l’accès au portail d’administration Azure AD | L’attribution de la valeur Non à cette option permet aux non-administrateurs d’utiliser le portail d’administration Azure AD pour lire et gérer les ressources Azure AD. Oui, interdit à tous les non-administrateurs l’accès aux données Azure AD sur le portail d’administration.<p>**Remarque**: ce paramètre ne limite pas l’accès aux données d’Azure AD à l’aide de PowerShell ou d’autres clients tels que Visual Studio. Quand la valeur est définie sur Oui, pour accorder à un utilisateur non-administrateur spécifique la possibilité d’utiliser le portail d’administration Azure AD, attribuez un rôle d’administrateur tel que le rôle Lecteurs de répertoire.<p>Ce rôle permet de lire les informations de base relatives aux annuaires, ce dont les utilisateurs membres disposent par défaut (les invités et les principaux du service n’en bénéficient pas).
Possibilité de lire d’autres utilisateurs | Ce paramètre est uniquement disponible dans PowerShell. Définir cet indicateur sur $false empêche tous les utilisateurs non administrateurs de lire les informations utilisateur dans le répertoire. Cet indicateur n’empêche pas de lire les informations utilisateur dans d’autres services Microsoft comme Exchange Online. Ce paramètre est destiné à des circonstances particulières et définir cet indicateur sur $false n’est pas recommandé.

## <a name="restrict-guest-users-default-permissions"></a>Restreindre les autorisations par défaut des utilisateurs invités

Vous pouvez restreindre les autorisations par défaut d’utilisateurs invités comme suit.

>[!NOTE]
>Le paramètre de restrictions d’accès d’utilisateur invités a remplacé le paramètre **Les autorisations d’utilisateurs invités sont limitées**. Pour obtenir des conseils sur l’utilisation de cette fonctionnalité, consultez [Restriction des autorisations d’accès invité (préversion) dans Azure Active Directory](../enterprise-users/users-restrict-guest-permissions.md).

Autorisation | Explication du paramètre
---------- | ------------
Restrictions d’accès d’utilisateurs invités (préversion) | La définition de cette option de façon à attribuer aux utilisateurs invités **le même accès que celui des membres** a pour effet d’accorder par défaut toutes les autorisations des utilisateurs membres aux utilisateurs invités.<p>La définition de cette option de façon à ce que **l’accès des utilisateurs invités soit limité aux propriétés et aux appartenances de leurs propres objets d’annuaire** a pour effet de limiter par défaut l’accès des invités à leur seul propre profil utilisateur. L’accès à d’autres utilisateurs n’est plus autorisé, même lors d’une recherche par nom d’utilisateur principal, ObjectId ou nom d'affichage. L’accès aux informations des groupes, y compris aux appartenances aux groupes, n’est plus autorisé.<p>**Remarque** : Ce paramètre n’empêche pas l’accès aux groupes joints dans certains services Microsoft 365 tels que Microsoft Teams. Pour en savoir plus, consultez [Accès invité à Microsoft Teams](https://docs.microsoft.com/MicrosoftTeams/guest-access).<p>Les utilisateurs invités peuvent toujours être ajoutés aux rôles d’administrateur, quels que soient les paramètres de cette autorisation.
Les invités peuvent inviter | Si cette option est définie sur Oui, les invités sont autorisés à inviter d’autres invités. Pour en savoir plus, consultez [Déléguer des invitations pour B2B Collaboration](../external-identities/delegate-invitations.md#configure-b2b-external-collaboration-settings).
Les membres peuvent inviter | Définir cette option sur Oui permet aux membres non-administrateurs de votre annuaire de convier des invités. Pour en savoir plus, consultez [Déléguer des invitations pour B2B Collaboration](../external-identities/delegate-invitations.md#configure-b2b-external-collaboration-settings).
Les administrateurs et utilisateurs ayant le rôle d’inviteur invité peuvent inviter | La définition de cette option sur Oui permet aux administrateurs et aux utilisateurs dont le rôle est « Inviteur d’invités » de convier des invités. Lorsque la valeur est Oui, les utilisateurs dont le rôle est « Inviteur d’invité » peuvent toujours convier des invités, quels que soient les membres pouvant utiliser le paramètre d’invitation. Pour en savoir plus, consultez [Déléguer des invitations pour B2B Collaboration](../external-identities/delegate-invitations.md#assign-the-guest-inviter-role-to-a-user).

## <a name="object-ownership"></a>Propriété des objets

### <a name="application-registration-owner-permissions"></a>Autorisations des propriétaires liées à l’inscription d’une application
Quand un utilisateur inscrit une application, il est automatiquement ajouté en tant que propriétaire de l’application. En tant que tel, il peut gérer les métadonnées de l’application, telles que le nom et les autorisations que demande l’application. Il peut également gérer la configuration spécifique du locataire de l’application, telle que les affectations d’utilisateurs et la configuration de l’authentification unique. Un propriétaire peut également ajouter ou supprimer des propriétaires. Contrairement aux administrateurs généraux, les propriétaires ne peuvent gérer que les applications qu’ils possèdent.

### <a name="enterprise-application-owner-permissions"></a>Autorisations de propriétaire d’applications d’entreprise
Quand un utilisateur ajoute une application d’entreprise, il est automatiquement ajouté en tant que propriétaire. En cette qualité, il peut gérer la configuration spécifique du locataire de l’application, telle que la configuration de l’authentification unique, l’approvisionnement et les affectations d’utilisateurs. Un propriétaire peut également ajouter ou supprimer des propriétaires. Contrairement aux administrateurs généraux, les propriétaires ne peuvent gérer que les applications qu’ils possèdent.

### <a name="group-owner-permissions"></a>Autorisations de propriétaire de groupe
Quand un utilisateur crée un groupe, il est automatiquement ajouté en tant que propriétaire de ce groupe. En tant que tel, il peut gérer les propriétés du groupe telles que le nom, ainsi que gérer l’appartenance au groupe. Un propriétaire peut également ajouter ou supprimer des propriétaires. Contrairement aux administrateurs globaux et administrateurs d’utilisateur, les propriétaires ne peuvent gérer que les groupes qu’ils possèdent. Pour assigner un propriétaire de groupe, consultez [Gestion des propriétaires d’un groupe](active-directory-accessmanagement-managing-group-owners.md).

### <a name="ownership-permissions"></a>Autorisations de propriété
Les tableaux suivants décrivent les autorisations spécifiques dans Azure Active Directory que les utilisateurs ont sur les objets qu’ils possèdent. L’utilisateur ne dispose de ces autorisations que sur les objets qu’il possède.

#### <a name="owned-application-registrations"></a>Inscriptions d’application possédées
Les utilisateurs peuvent effectuer les actions suivantes sur des inscriptions d’application possédées.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/applications/audience/update | Mettez à jour la propriété applications.audience dans Azure Active Directory. |
| microsoft.directory/applications/authentication/update | Mettez à jour la propriété applications.authentication dans Azure Active Directory. |
| microsoft.directory/applications/basic/update | Mettez à jour des propriétés de base sur des applications dans Azure Active Directory. |
| microsoft.directory/applications/credentials/update | Mettez à jour la propriété applications.credentials dans Azure Active Directory. |
| microsoft.directory/applications/delete | Supprimez des applications dans Azure Active Directory. |
| microsoft.directory/applications/owners/update | Mettez à jour la propriété applications.owners dans Azure Active Directory. |
| microsoft.directory/applications/permissions/update | Mettez à jour la propriété applications.permissions dans Azure Active Directory. |
| microsoft.directory/applications/policies/update | Mettez à jour la propriété applications.policies dans Azure Active Directory. |
| microsoft.directory/applications/restore | Restaurez des applications dans Azure Active Directory. |

#### <a name="owned-enterprise-applications"></a>Applications d’entreprise possédées
Les utilisateurs peuvent effectuer les actions suivantes sur les applications d’entreprise qu’ils possèdent. Une application d’entreprise se compose d’un principal du service, d’une ou plusieurs stratégies d’application et parfois d’un objet d’application dans le même locataire que le principal du service.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/auditLogs/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur auditLogs dans Azure Active Directory. |
| microsoft.directory/policies/basic/update | Mettez à jour des propriétés de base sur des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/delete | Supprimez des stratégies dans Azure Active Directory. |
| microsoft.directory/policies/owners/update | Mettez à jour la propriété policies.owners dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Mettez à jour la propriété servicePrincipals.appRoleAssignedTo dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/appRoleAssignments/update | Mettez à jour la propriété users.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/audience/update | Mettez à jour la propriété servicePrincipals.audience dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/authentication/update | Mettez à jour la propriété servicePrincipals.authentication dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/basic/update | Mettez à jour des propriétés de base sur des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/credentials/update | Mettez à jour la propriété servicePrincipals.credentials dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/delete | Supprimez des principaux de service dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/owners/update | Mettez à jour la propriété servicePrincipals.owners dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/permissions/update | Mettez à jour la propriété servicePrincipals.permissions dans Azure Active Directory. |
| microsoft.directory/servicePrincipals/policies/update | Mettez à jour la propriété servicePrincipals.policies dans Azure Active Directory. |
| microsoft.directory/signInReports/allProperties/read | Lisez toutes les propriétés (y compris les propriétés privilégiées) sur signInReports dans Azure Active Directory. |

#### <a name="owned-devices"></a>Appareils joints possédés
Les utilisateurs peuvent effectuer les actions suivantes sur les appareils qu’ils possèdent.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/devices/bitLockerRecoveryKeys/read | Lisez la propriété devices.bitLockerRecoveryKeys dans Azure Active Directory. |
| microsoft.directory/devices/disable | Désactivez des appareils dans Azure Active Directory. |

#### <a name="owned-groups"></a>Groupes possédés
Les utilisateurs peuvent effectuer les actions suivantes sur les groupes qu’ils possèdent.

| **Actions** | **Description** |
| --- | --- |
| microsoft.directory/groups/appRoleAssignments/update | Mettez à jour la propriété groups.appRoleAssignments dans Azure Active Directory. |
| microsoft.directory/groups/basic/update | Mettez à jour des propriétés de base sur des groupes dans Azure Active Directory. |
| microsoft.directory/groups/delete | Supprimez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/dynamicMembershipRule/update | Mettez à jour la propriété groups.dynamicMembershipRule dans Azure Active Directory. |
| microsoft.directory/groups/members/update | Mettez à jour la propriété groups.members dans Azure Active Directory. |
| microsoft.directory/groups/owners/update | Mettez à jour la propriété groups.owners dans Azure Active Directory. |
| microsoft.directory/groups/restore | Restaurez des groupes dans Azure Active Directory. |
| microsoft.directory/groups/settings/update | Mettez à jour la propriété groups.settings dans Azure Active Directory. |

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur le paramètre des restrictions d’accès des utilisateurs invités, consultez [Restriction des autorisations d’accès invité (préversion) dans Azure Active Directory](../enterprise-users/users-restrict-guest-permissions.md).
* Pour en savoir plus sur l’attribution des rôles d’administrateur Azure AD, consultez [Assign a user to administrator roles in Azure Active Directory](active-directory-users-assign-role-azure-portal.md) (Attribution d’un utilisateur aux rôles d’administrateur dans Azure Active Directory).
* Pour plus d’informations sur la façon dont l’accès aux ressources est contrôlé dans Microsoft Azure, voir [Présentation de l’accès aux ressources dans Azure](../../role-based-access-control/rbac-and-directory-admin-roles.md)
* Pour plus d’informations sur l’association entre Azure Active Directory et votre abonnement Azure, consultez [Association des abonnements Azure avec Azure Active Directory](active-directory-how-subscriptions-associated-directory.md)
* [Gestion des utilisateurs](add-users-azure-active-directory.md)
