---
title: Ajouter ou supprimer les propriétaires d’un groupe – Azure Active Directory | Microsoft Docs
description: Ces instructions expliquent comment ajouter ou supprimer les propriétaires de groupes avec Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 09/11/2018
ms.author: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 358a551908a7263f3258f47dfe1cceeffe2307b1
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860488"
---
# <a name="add-or-remove-group-owners-in-azure-active-directory"></a>Ajouter ou supprimer les propriétaires de groupes dans Azure Active Directory
Les groupes Azure Active Directory (Azure AD) sont détenus et gérés par des propriétaires de groupes. Les propriétaires de groupe peuvent être des utilisateurs ou des principaux de service et sont en mesure de gérer le groupe, notamment l’appartenance. Seuls les propriétaires de groupe existant ou les administrateurs gérant un groupe peuvent attribuer des propriétaires de groupe. Ils ne sont pas nécessairement membres du groupe.

Lorsqu’un groupe n’a aucun propriétaire, les administrateurs gérant le groupe sont toujours en mesure de gérer le groupe. Il est recommandé à chaque groupe d'avoir au moins un propriétaire. Une fois les propriétaires attribués à un groupe, il est impossible de retirer le dernier propriétaire. Veillez à sélectionner un autre propriétaire avant de retirer le dernier propriétaire du groupe.

## <a name="add-an-owner-to-a-group"></a>Ajouter un propriétaire à un groupe
Vous trouverez ci-dessous des instructions pour l’ajout d’un utilisateur en tant que propriétaire d’un groupe à l’aide du Portail Azure Active Directory. Pour ajouter un principal de service en tant que propriétaire d’un groupe, suivez les instructions pour effectuer cette opération avec [PowerShell](/powershell/module/Azuread/Add-AzureADGroupOwner).

### <a name="to-add-a-group-owner"></a>Pour ajouter un propriétaire de groupe
1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte d’administrateur général pour le répertoire.

2. Sélectionnez successivement **Azure Active Directory**, **Groupes** et le groupe pour lequel vous souhaitez ajouter un propriétaire (pour cet exemple, *MDM policy - West* [Stratégie GPM - Ouest]).

3. Dans la page **MDM policy - West Overview** (Vue d’ensemble de la stratégie GPM - Ouest), sélectionnez **Propriétaires**.

    ![Page MDM policy - West Overview (Vue d’ensemble de la stratégie GPM - Ouest) avec l’option Propriétaires mise en surbrillance](media/active-directory-accessmanagement-managing-group-owners/add-owners-option-overview-blade.png)

4. Dans la page **MDM policy - West - Owners** (Stratégie GPM - Ouest - Propriétaires), sélectionnez **Ajouter des propriétaires**, recherchez et sélectionnez l’utilisateur qui sera le nouveau propriétaire de groupe, puis choisissez **Sélectionner**.

    ![Page MDM policy - West - Owners (Stratégie GPM - Ouest - Propriétaires) avec l’option Propriétaires mise en surbrillance](media/active-directory-accessmanagement-managing-group-owners/add-owners-owners-blade.png)

    Une fois que vous avez sélectionné le nouveau propriétaire, vous pouvez actualiser la page **Propriétaires** et observer que le nom a été ajouté à la liste des propriétaires.

## <a name="remove-an-owner-from-a-group"></a>Supprimer un propriétaire d’un groupe
Supprimez un propriétaire d’un groupe à l’aide d’Azure AD.

### <a name="to-remove-an-owner"></a>Pour supprimer un propriétaire
1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte d’administrateur général pour le répertoire.

2. Sélectionnez successivement **Azure Active Directory**, **Groupes** et le groupe pour lequel vous souhaitez supprimer un propriétaire (pour cet exemple, *MDM policy - West* [Stratégie GPM - Ouest]).

3. Dans la page **MDM policy - West Overview** (Vue d’ensemble de la stratégie GPM - Ouest), sélectionnez **Propriétaires**.

    ![Page MDM policy - West Overview (Vue d’ensemble de la stratégie GPM - Ouest) avec l’option Supprimer les propriétaires mise en surbrillance](media/active-directory-accessmanagement-managing-group-owners/remove-owners-option-overview-blade.png)

4. Dans la page **MDM policy - West - Owners** (Stratégie GPM - Ouest - Propriétaires), sélectionnez l’utilisateur que vous souhaitez supprimer en tant que propriétaire de groupe, choisissez **Supprimer** dans la page d’informations de l’utilisateur, puis sélectionnez **Oui** pour confirmer votre décision.

    ![Page d’informations d’utilisateur avec l’option Supprimer mise en surbrillance](media/active-directory-accessmanagement-managing-group-owners/remove-owner-info-blade.png)

    Après que vous avez supprimé le propriétaire, vous pouvez revenir à la page **Propriétaires** et observer que le nom a été supprimé de la liste des propriétaires.

## <a name="next-steps"></a>Étapes suivantes
- [Gestion de l’accès aux ressources avec les groupes Azure Active Directory](active-directory-manage-groups.md)

- [Configuration des paramètres de groupe avec les applets de commande Azure Active Directory](../enterprise-users/groups-settings-cmdlets.md)

- [Utilisation d’un groupe pour gérer l’accès aux applications SaaS](../enterprise-users/groups-saasapps.md)

- [Intégration des identités locales dans Azure Active Directory](../hybrid/whatis-hybrid-identity.md)

- [Configuration des paramètres de groupe avec les applets de commande Azure Active Directory](../enterprise-users/groups-settings-v2-cmdlets.md)
