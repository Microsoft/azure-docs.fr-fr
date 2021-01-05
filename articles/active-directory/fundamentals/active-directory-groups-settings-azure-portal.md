---
title: Modifier les informations de votre groupe - Azure Active Directory | Microsoft Docs
description: Instructions pour modifier les informations de votre groupe avec Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 08/27/2018
ms.author: ajburnle
ms.reviewer: krbain
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: c346877d29c5fef03b45c895cbf8d462f90d0979
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92369129"
---
# <a name="edit-your-group-information-using-azure-active-directory"></a>Modifier les informations de votre groupe avec Azure Active Directory

Avec Azure Active Directory (Azure AD), vous pouvez modifier les paramètres d’un groupe, y compris son nom, sa description ou son type d’appartenance.

## <a name="to-edit-your-group-settings"></a>Modifier vos paramètres de groupe
1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte d’administrateur général pour le répertoire.

2. Sélectionnez **Azure Active Directory** , puis **Groupes** .

    La page **Groups - All groups** (Groupes - Tous les groupes) s’affiche, présentant tous vos groupes actifs.

3. Depuis la page **Groupes - Tous les groupes** , saisissez autant que possible le nom du groupe dans la zone **Recherche** . Dans le cadre de cet article, nous recherchons le groupe **MDM policy - West** .

    Les résultats de la recherche apparaissent dans la zone **Recherche** , qui se met à jour au fur et à mesure que vous tapez des caractères.

    ![Page Tous les groupes, avec texte de recherche dans la zone Recherche](media/active-directory-groups-settings-azure-portal/search-for-specific-group.png)

4. Sélectionnez le groupe **MDM policy - West** , puis **Propriétés** depuis la zone **Gérer** .

    ![Page Présentation du groupe avec options de membre et informations en surbrillance](media/active-directory-groups-settings-azure-portal/group-overview-blade.png)

5. En cas de besoin, mettez à jour les informations dans **Paramètres généraux** , y compris :

    ![Les paramètres des propriétés d’un groupe](media/active-directory-groups-settings-azure-portal/group-properties-settings.png)

    - **Nom du groupe** . Modifiez le nom du groupe existant.
    
    - **Description du groupe** . Modifiez la description du groupe existant.

    - **Type du groupe** . Vous ne pouvez pas changer le type du groupe après sa création. Pour changer le **Type du groupe** , vous devez le supprimer et en recréer un.
    
    - **Type d’appartenance.** Changez le type d’appartenance. Pour plus d’informations sur les différents types d’appartenance disponibles, consultez le [Guide pratique pour créer un groupe de base et ajouter des membres avec le portail Azure Active Directory](active-directory-groups-create-azure-portal.md).
    
    - **ID d’objet** . Vous ne pouvez pas changer l’ID d’objet, mais vous pouvez le copier pour vous en servir dans vos commandes PowerShell pour le groupe. Pour plus d’informations sur l’utilisation des cmdlets PowerShell, consultez [Configuration des paramètres de groupe avec les cmdlets Azure Active Directory](../enterprise-users/groups-settings-v2-cmdlets.md).

## <a name="next-steps"></a>Étapes suivantes
Ces articles fournissent des informations supplémentaires sur Azure Active Directory.

- [Afficher vos groupes et vos membres](active-directory-groups-view-azure-portal.md)

- [Créer un groupe de base et ajouter des membres](active-directory-groups-create-azure-portal.md)

- [Ajouter ou supprimer des membres d’un groupe](active-directory-groups-members-azure-portal.md)

- [Gérer les règles dynamiques pour les utilisateurs dans un groupe](../enterprise-users/groups-create-rule.md)

- [Gérer l’appartenance à un groupe](active-directory-groups-membership-azure-portal.md)

- [Gérer l’accès aux ressources à l’aide des groupes](active-directory-manage-groups.md)

- [Associer ou ajouter un abonnement Azure à Azure Active Directory](active-directory-how-subscriptions-associated-directory.md)
