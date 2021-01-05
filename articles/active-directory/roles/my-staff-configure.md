---
title: Délégation de la gestion des utilisateurs avec Mon personnel (préversion) – Azure AD | Microsoft Docs
description: Déléguez la gestion des utilisateurs avec Mon personnel et les unités administratives.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.topic: how-to
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.date: 05/08/2020
ms.author: curtand
ms.reviewer: sahenry
ms.custom: oldportal;it-pro;
ms.openlocfilehash: 927d1f6dff9a24fb8427bb9c353e6cc61ac6a1f0
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92372939"
---
# <a name="manage-your-users-with-my-staff-preview"></a>Gestion des utilisateurs avec Mon personnel (préversion)

Mon personnel permet de déléguer à une figure d’autorité (par exemple, un responsable de magasin ou un chef d’équipe) les autorisations nécessaires pour que les membres de son personnel puissent accéder à leur compte Azure AD. Au lieu de s’appuyer sur un support technique central, les organisations peuvent déléguer à un chef d’équipe des tâches courantes comme la réinitialisation de mot de passe ou la modification des numéros de téléphone. Avec Mon personnel, l’utilisateur qui ne parvient pas à accéder à son compte peut récupérer l’accès en quelques clics seulement, sans intervention du support technique ou du personnel informatique.

Avant de configurer Mon personnel pour votre organisation, nous vous recommandons de consulter cette documentation ainsi que la [documentation utilisateur](../user-help/my-staff-team-manager.md) pour bien comprendre ses fonctionnalités et son impact sur vos utilisateurs. Vous pouvez utiliser la documentation utilisateur pour former vos utilisateurs et les préparer à la nouvelle expérience, afin de faciliter le déploiement.

L’authentification par SMS des utilisateurs est une fonctionnalité en préversion publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="how-my-staff-works"></a>Fonctionnement de Mon personnel

Mon personnel se base sur des unités administratives, c’est-à-dire des conteneurs de ressources permettant de limiter l’étendue du contrôle administratif d’une attribution de rôle. Dans Mon personnel, les unités administratives sont utilisées pour définir un sous-ensemble des utilisateurs d’une organisation, par exemple un magasin ou un service. Un rôle étendu à une ou plusieurs unités administratives pourrait alors être attribué à un chef d’équipe. Dans l’exemple ci-dessous, l’utilisateur s’est vu accorder le rôle d’administrateur d’authentification, dont l’étendue couvre les trois unités administratives. Pour plus d’informations sur les unités administratives, consultez [Gestion des unités administratives dans Azure Active Directory](administrative-units.md).

## <a name="before-you-begin"></a>Avant de commencer

Pour faire ce qui est décrit dans cet article, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.

  * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement.

  * Si nécessaire, [créez un locataire Azure Active Directory](../fundamentals/sign-up-organization.md) ou [associez un abonnement Azure à votre compte](../fundamentals/active-directory-how-subscriptions-associated-directory.md).
* Vous devez disposer de privilèges d’ *administrateur d’entreprise* dans votre locataire Azure AD pour activer l’authentification par SMS.
* Chaque utilisateur activé dans la stratégie de méthode d’authentification par SMS doit disposer d’une licence, même s’il ne l’utilise pas. Chaque utilisateur activé doit avoir l’une des licences Azure AD ou Microsoft 365 suivantes :

  * [Azure AD Premium P1 ou P2](https://azure.microsoft.com/pricing/details/active-directory/)
  * [Microsoft 365 (M365) F1 ou F3](https://www.microsoft.com/licensing/news/m365-firstline-workers)
  * [Enterprise Mobility + Security (EMS) E3 ou E5](https://www.microsoft.com/microsoft-365/enterprise-mobility-security/compare-plans-and-pricing) ou [Microsoft 365 (M365) E3 ou E5](https://www.microsoft.com/microsoft-365/compare-microsoft-365-enterprise-plans)

## <a name="how-to-enable-my-staff"></a>Activation de Mon personnel

Une fois que vous avez configuré les unités administratives, vous pouvez appliquer cette étendue aux utilisateurs qui accèdent à Mon personnel. Seuls les utilisateurs disposant d’un rôle d’administrateur ont accès à Mon personnel. Pour activer Mon personnel, procédez de la façon suivante :

1. Connectez-vous au Portail Azure en tant qu’Administrateur d’utilisateurs.
2. Accédez à **Azure Active Directory** > **Paramètres utilisateur** > **Préversion des fonctionnalités utilisateur** > **Gérer les paramètres de préversion des fonctionnalités utilisateur** .
3. Sous **Les administrateurs peuvent accéder à Mon personnel** , vous pouvez choisir d’activer l’accès pour tous les utilisateurs, les utilisateurs sélectionnés ou aucun utilisateur.

> [!Note]
> Seuls les utilisateurs disposant d’un rôle d’administrateur ont accès à Mon personnel. Si vous activez Mon personnel pour un utilisateur à qui aucun rôle d’administrateur n’est attribué, il ne pourra pas accéder à Mon personnel.

## <a name="conditional-access"></a>Accès conditionnel

Vous pouvez protéger le portail Mon personnel à l’aide d’une stratégie d’accès conditionnel Azure AD. Utilisez celle-ci pour des tâches telles que l’authentification multifacteur avant d’accéder au portail Mon personnel.

Nous vous recommandons vivement de protéger le portail Mon personnel à l’aide de [stratégies d’accès conditionnel Azure AD](../conditional-access/index.yml). Pour appliquer une stratégie d’accès conditionnel au portail Mon personnel, vous devez créer manuellement le principal du service Mon personnel à l’aide de PowerShell.

### <a name="apply-a-conditional-access-policy-to-my-staff"></a>Appliquer une stratégie d’accès conditionnel au portail Mon personnel

1. Installez les [cmdlets PowerShell Microsoft Graph Beta](https://github.com/microsoftgraph/msgraph-sdk-powershell/blob/dev/samples/0-InstallModule.ps1).
1. Exécutez les commandes suivantes :

   ```powershell
   Connect-Graph -Scopes "Directory.AccessAsUser.All"
   New-MgServicePrincipal -DisplayName "My Staff" -AppId "ba9ff945-a723-4ab5-a977-bd8c9044fe61"
   ```
1. Créez une stratégie d’accès conditionnel qui s’applique à l’application cloud Mon personnel.

    ![Créer une stratégie d’accès conditionnel pour l’application Mon personnel](./media/my-staff-configure/conditional-access.png)

## <a name="using-my-staff"></a>Utilisation de Mon personnel

Quand un utilisateur accède à Mon personnel, il voit apparaître le nom des [unités administratives](administrative-units.md) sur lesquelles il dispose d’autorisations d’administration. Dans la [documentation utilisateur de Mon personnel](../user-help/my-staff-team-manager.md), nous utilisons le terme « emplacement » pour faire référence aux unités administratives. Les autorisations d’un administrateur qui ne comportent pas d’étendue couvrant des unités administratives s’appliquent à l’ensemble de l’organisation. Après l’activation de Mon personnel, les utilisateurs activés et disposant d’un rôle d’administrateur peuvent y accéder à l’adresse [https://mystaff.microsoft.com](https://mystaff.microsoft.com). Ils ont la possibilité de sélectionner une unité administrative pour afficher les utilisateurs qu’elle comporte, et de sélectionner un utilisateur pour ouvrir son profil.

## <a name="reset-a-users-password"></a>Réinitialiser le mot de passe de l’utilisateur

Les rôles suivants sont autorisés à réinitialiser le mot de passe d’un utilisateur :

- [Administrateur d’authentification](permissions-reference.md#authentication-administrator)
- [Administrateur d’authentification privilégié](permissions-reference.md#privileged-authentication-administrator)
- [Administrateur général](permissions-reference.md#global-administrator--company-administrator)
- [Administrateur du support technique](permissions-reference.md#helpdesk-administrator)
- [Administrateur d’utilisateurs](permissions-reference.md#user-administrator)
- [Administrateur de mots de passe](permissions-reference.md#password-administrator)

Dans **Mon personnel** , ouvrez le profil d’un utilisateur. Sélectionnez **Réinitialiser le mot de passe** .

- Si l’utilisateur se trouve uniquement sur le cloud, un mot de passe temporaire s’affiche. Vous pouvez le donner à l’utilisateur.
- Si l’utilisateur est synchronisé à partir d’Active Directory en local, vous pouvez entrer un mot de passe qui respecte vos stratégies AD locales. Vous pouvez ensuite le donner à l’utilisateur.

    ![Indicateur de progression de la réinitialisation de mot de passe et notification de réussite](./media/my-staff-configure/reset-password.png)

L’utilisateur devra modifier son mot de passe à la prochaine connexion.

## <a name="manage-a-phone-number"></a>Gestion d’un numéro de téléphone

Dans **Mon personnel** , ouvrez le profil d’un utilisateur.

- Sélectionnez la section **Ajouter un numéro de téléphone** pour ajouter un numéro de téléphone à l’utilisateur.
- Sélectionnez **Modifier le numéro de téléphone** pour modifier le numéro de téléphone.
- Sélectionnez **Supprimer le numéro de téléphone** pour supprimer le numéro de téléphone de l’utilisateur.

Selon vos paramètres, l’utilisateur peut ensuite utiliser le numéro de téléphone que vous avez configuré pour se connecter par SMS, effectuer une authentification multifacteur et opérer une réinitialisation du mot de passe en libre-service.

Pour gérer le numéro de téléphone d’un utilisateur, vous devez disposer de l’un des rôles suivants :

- [Administrateur d’authentification](permissions-reference.md#authentication-administrator)
- [Administrateur d’authentification privilégié](permissions-reference.md#privileged-authentication-administrator)
- [Administrateur général](permissions-reference.md#global-administrator--company-administrator)

## <a name="search"></a>Recherche

Pour rechercher des unités administratives et des utilisateurs de votre organisation, vous pouvez utiliser la barre de recherche de Mon personnel. La recherche peut porter sur toutes les unités administratives et tous les utilisateurs de votre organisation, mais vous ne pouvez apporter des modifications qu’aux utilisateurs se trouvant dans une unité administrative sur laquelle des autorisations d’administrateur vous ont été accordées.

Il est également possible de rechercher un utilisateur au sein d’une unité administrative. Pour cela, utilisez la barre de recherche qui se trouve en haut de la liste d’utilisateurs.

## <a name="audit-logs"></a>Journaux d’audit

Vous pouvez afficher les journaux d’audit des actions effectuées dans Mon personnel sur le portail Azure Active Directory. Si un journal d’audit a été généré par une action effectuée dans Mon personnel, cette information est indiquée sous DÉTAILS SUPPLÉMENTAIRES dans l’événement d’audit.

## <a name="next-steps"></a>Étapes suivantes

[Documentation utilisateur de Mon personnel](../user-help/my-staff-team-manager.md)
[Documentation sur les unités administratives](administrative-units.md)