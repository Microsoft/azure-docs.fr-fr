---
title: Vue d’ensemble des comptes d’utilisateur dans Azure Active Directory B2C
description: Découvrez les types de comptes d’utilisateur utilisables dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 11/05/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 0a8e08d0045d0520241341cc08fb800468ed6897
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96928611"
---
# <a name="overview-of-user-accounts-in-azure-active-directory-b2c"></a>Vue d’ensemble des comptes d’utilisateur dans Azure Active Directory B2C

Dans Azure Active Directory B2C (Azure AD B2C), plusieurs types de comptes peuvent être créés. Azure Active Directory, Active Directory B2B et Active Directory B2C partagent les types de comptes d’utilisateur utilisables.

Les types de comptes suivants sont disponibles :

- **Compte professionnel** – Un compte professionnel peut accéder aux ressources dans un locataire et, avec un rôle d’administrateur, peut gérer des locataires.
- **Compte invité** – Un compte invité peut uniquement être un compte Microsoft ou un utilisateur Azure Active Directory qui peut être utilisé pour accéder à des applications ou gérer des locataires.
- **Compte de consommateur** – Un compte de consommateur est utilisé par un utilisateur des applications inscrites auprès d’Azure AD B2C. Il existe plusieurs moyens de créer des comptes de consommateur :
  - L’utilisateur suit un flux utilisateur d’inscription dans une application Azure AD B2C
  - Utilisation de l’API Microsoft Graph
  - Utilisation du portail Azure

## <a name="work-account"></a>Compte professionnel

Un compte professionnel est créé de la même manière pour tous les locataires basés sur Azure AD. Pour créer un compte professionnel, vous pouvez utiliser les informations contenues dans [Démarrage rapide : Ajouter de nouveaux utilisateurs à Azure Active Directory](../active-directory/fundamentals/add-users-azure-active-directory.md). Un compte professionnel est créé en choisissant **Nouvel utilisateur** dans le portail Azure.

Lorsque vous ajoutez un nouveau compte professionnel, vous devez prendre en compte les paramètres de configuration suivants :

- **Nom** et **Nom d’utilisateur** – La propriété **Nom** contient le prénom et le nom de l’utilisateur. Le **nom d’utilisateur** est l’identificateur que l’utilisateur utilise pour se connecter. Le nom d’utilisateur inclut le domaine complet. La partie du nom de domaine du nom d’utilisateur doit être le nom de domaine initial par défaut, *nom de domaine.onmicrosoft.com*, ou un nom de [domaine personnalisé](../active-directory/fundamentals/add-custom-domain.md) vérifié, non fédéré, tel que *contoso.com*.
- **Profil** – Le compte est configuré avec un profil de données utilisateur. Vous avez la possibilité d’entrer un prénom, un nom, un poste et un nom de service. Vous pouvez modifier le profil une fois que le compte a été créé.
- **Groupes** – Un groupe vous permet d’effectuer différentes tâches de gestion, comme attribuer des licences ou des autorisations à plusieurs utilisateurs ou appareils simultanément. Vous pouvez placer le nouveau compte dans un [groupe](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md) existant dans votre locataire.
- **Rôle d’annuaire** – Vous devez spécifier le niveau d’accès que le compte d’utilisateur possède pour les ressources dans votre locataire. Les niveaux d’autorisation suivants sont disponibles :

    - **Utilisateur** – Les utilisateurs peuvent accéder aux ressources affectées, mais ne peuvent pas gérer la plupart des ressources de locataire.
    - **Administrateur général** – Les administrateurs généraux contrôlent totalement toutes les ressources de locataire.
    - **Administrateur limité** – Sélectionnez le ou les rôles d’administration pour l’utilisateur. Pour plus d’informations sur les rôles qui peuvent être sélectionnés, consultez [Attribution de rôles d’administrateur dans Azure Active Directory](../active-directory/roles/permissions-reference.md).

### <a name="create-a-work-account"></a>Créer un compte professionnel

Vous pouvez utiliser les informations suivantes pour créer un nouveau compte professionnel :

- [Azure portal](../active-directory/fundamentals/add-users-azure-active-directory.md)
- [Microsoft Graph](/graph/api/user-post-users)

### <a name="update-a-user-profile"></a>Mettre à jour un profil utilisateur

Vous pouvez utiliser les informations suivantes pour mettre à jour le profil d’un utilisateur :

- [Azure portal](../active-directory/fundamentals/active-directory-users-profile-azure-portal.md)
- [Microsoft Graph](/graph/api/user-update)

### <a name="reset-a-password-for-a-user"></a>Réinitialiser un mot de passe pour un utilisateur

Vous pouvez utiliser les informations suivantes pour réinitialiser le mot de passe d’un utilisateur :

- [Azure portal](../active-directory/fundamentals/active-directory-users-reset-password-azure-portal.md)
- [Microsoft Graph](/graph/api/user-update)

## <a name="guest-user"></a>Utilisateur invité

Vous pouvez inviter des utilisateurs externes dans votre locataire en tant qu’utilisateur invité. Un scénario standard pour inviter un utilisateur dans votre locataire Azure AD B2C consiste à partager les responsabilités d’administration. Pour obtenir un exemple d’utilisation d’un compte invité, consultez [Propriétés d’un utilisateur Azure Active Directory B2B Collaboration](../active-directory/external-identities/user-properties.md).

Lorsque vous invitez un utilisateur dans votre locataire, vous fournissez l’adresse e-mail du destinataire et un message décrivant l’invitation. Le lien d’invitation dirige l’utilisateur vers la page de consentement où le bouton **Prise en main** est sélectionné et la révision des autorisations est acceptée. Si une boîte de réception n’est pas attachée à l’adresse de messagerie, l’utilisateur peut accéder à la page de consentement en accédant à une page de Microsoft en utilisant les informations d’identification d’invité. L’utilisateur est alors obligé d’utiliser l’invitation de la même façon qu’en cliquant sur le lien dans l’e-mail. Par exemple : `https://myapps.microsoft.com/B2CTENANTNAME`.

Vous pouvez également utiliser l’[API Microsoft Graph](/graph/api/invitation-post?view=graph-rest-beta) pour inviter un utilisateur.

## <a name="consumer-user"></a>Utilisateur consommateur

L’utilisateur consommateur peut se connecter aux applications sécurisées par Azure AD B2C, mais il ne peut pas accéder aux ressources Azure telles que le portail Azure. L’utilisateur consommateur peut utiliser un compte local ou des comptes fédérés, tels que Facebook ou Twitter. Un compte de consommateur est créé avec un [flux utilisateur d’inscription ou de connexion](user-flow-overview.md), avec l’API Microsoft Graph ou avec le portail Azure.

Vous pouvez spécifier les données qui sont collectées lorsqu’un compte d’utilisateur consommateur est créé à l’aide des attributs d’utilisateur personnalisés. Pour plus d’informations, consultez [Définir des attributs personnalisés dans Azure Active Directory B2C](user-flow-custom-attributes.md).

Pour plus d’informations sur la gestion des comptes de consommateur, consultez [Gérer des comptes d’utilisateur Azure AD B2C avec Microsoft Graph](manage-user-accounts-graph-api.md).

### <a name="migrate-consumer-user-accounts"></a>Migrer des comptes d’utilisateurs consommateurs

Vous pouvez avoir besoin de migrer des comptes d’utilisateur existants de n’importe quel fournisseur d’identité vers Azure AD B2C. Pour plus d’informations, consultez [Migrer des utilisateurs vers Azure AD B2C](user-migration.md).
