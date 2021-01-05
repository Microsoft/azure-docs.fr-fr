---
title: Ajouter ou supprimer un groupe dans un autre groupe - Azure AD
description: Instructions pour ajouter ou supprimer un groupe dans un autre groupe à l’aide d’Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 10/19/2018
ms.author: ajburnle
ms.custom: it-pro, seodec18
ms.reviewer: krbain
ms.collection: M365-identity-device-management
ms.openlocfilehash: cbd7eecd80b528b1d20d016d51c985bc24d1301d
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92371798"
---
# <a name="add-or-remove-a-group-from-another-group-using-azure-active-directory"></a>Ajouter ou supprimer un groupe dans un autre groupe à l’aide d’Azure Active Directory
Cet article vous aide à ajouter et supprimer un groupe d’un autre groupe à l’aide d’Azure Active Directory.

>[!Note]
>Si vous essayez de supprimer le groupe parent, consultez [How to update or delete a group and its members](active-directory-groups-delete-group.md) (Comment mettre à jour ou supprimer un groupe et ses membres).

## <a name="add-a-group-to-another-group"></a>Ajouter à un autre groupe
Vous pouvez ajouter un groupe de sécurité existant à un autre groupe de sécurité (aussi appelé groupe imbriqué), créer un groupe (ou sous-groupe) de membres et un groupe parent. Le groupe de membres hérite des attributs et des propriétés du groupe parent, vous permettant ainsi de gagner du temps de configuration.

>[!Important]
>Nous ne prenons actuellement pas en charge :<ul><li>L’ajout de groupes à un groupe synchronisé avec Active Directory en local.</li><li>L’ajout de groupes de sécurité aux groupes Microsoft 365.</li><li>L’ajout de groupes Microsoft 365 à des groupes de sécurité ou à d’autres groupes Microsoft 365.</li><li>L’affectation d’applications à des groupes imbriqués.</li><li>L’application de licences à des groupes imbriqués.</li><li>Ajout de groupes de distribution dans des scénarios d’imbrication.</li></ul>

### <a name="to-add-a-group-as-a-member-of-another-group"></a>Pour ajouter un groupe en tant que membre d’un autre groupe

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte d’administrateur général pour le répertoire.

2. Sélectionnez **Azure Active Directory** , puis **Groupes** .

3. Dans la page **Groupes - Tous les groupes** , recherchez et sélectionnez le groupe qui doit devenir un membre d’un autre groupe. Dans cet exercice, nous utilisons le groupe **MDM policy - West** (Stratégie GPM - Ouest).

    >[!Note]
    >Vous pouvez ajouter votre groupe en tant que membre à un seul groupe à la fois. La zone **Sélectionner un groupe** filtre également l’affichage en fonction de la correspondance de votre entrée avec une partie ou l’intégralité d’un nom d’utilisateur ou d’appareil. Les caractères génériques ne sont toutefois pas pris en charge.

    ![Page Groupes - Tous les groupes avec le groupe MDM policy - West (Stratégie GPM - Ouest) sélectionné](media/active-directory-groups-membership-azure-portal/group-all-groups-screen.png)

4. Dans la page **MDM policy - West - Group memberships** (Stratégie GPM - Ouest - Appartenances aux groupes), sélectionnez **Group memberships** (Appartenances aux groupes), sélectionnez **Ajouter** , recherchez le groupe dont votre groupe doit être membre, puis choisissez **Sélectionner** . Dans cet exercice, nous utilisons le groupe **MDM policy - All org** (Stratégie GPM - Toutes les org).

    Le groupe **MDM policy - West** (Stratégie GPM - Ouest) est maintenant un membre du groupe **MDM policy - All org** (Stratégie GPM - Toutes les org), héritant ainsi de toutes les propriétés et de la configuration du groupe Stratégie GPM - Toutes les org.

    ![Créer une appartenance de groupe en ajoutant un groupe à un autre groupe](media/active-directory-groups-membership-azure-portal/group-add-group-membership.png)

5. Consultez la page **MDM policy - West - Group memberships** (Stratégie GPM - Ouest - Appartenances aux groupes) pour voir la relation de groupe et de membre.

6. Pour une vue plus détaillée de la relation de groupe et de membre, sélectionnez le nom du groupe ( **MDM policy - All org** (Stratégie GPM - Toutes les org)) et observez les détails de la page **MDM policy - West** (Stratégie GPM - Ouest).

## <a name="remove-a-group-from-another-group"></a>Supprimer un groupe d’un autre groupe
Vous pouvez supprimer un groupe de sécurité existant d’un autre groupe de sécurité. Toutefois, la suppression du groupe supprime également les attributs et propriétés hérités pour ses membres.

### <a name="to-remove-a-member-group-from-another-group"></a>Pour supprimer un groupe de membres d’un autre groupe
1. Dans la page **Groupes - Tous les groupes** , recherchez et sélectionnez le groupe qui doit être supprimé en tant que membre d’un autre groupe. Dans cet exercice, nous utilisons une nouvelle fois le groupe **MDM policy - West** (Stratégie GPM - Ouest).

2. Dans la page **MDM policy - West overview** (Vue d’ensemble de la stratégie GPM - Ouest), sélectionnez **Group memberships** (Appartenances aux groupes).

3. Sélectionnez le groupe **MDM policy - All org** (Stratégie GPM - Toutes les org) dans la page **MDM policy - West - Group memberships** (Stratégie GPM - Ouest - Appartenances aux groupes), puis sélectionnez **Supprimer** dans les détails de la page **MDM policy - West** (Stratégie GPM - Ouest).

    ![Page d’appartenance au groupe affichant les détails du membre et du groupe](media/active-directory-groups-membership-azure-portal/group-membership-remove.png)

## <a name="additional-information"></a>Informations supplémentaires
Ces articles fournissent des informations supplémentaires sur Azure Active Directory.

- [Afficher vos groupes et vos membres](active-directory-groups-view-azure-portal.md)

- [Créer un groupe de base et ajouter des membres](active-directory-groups-create-azure-portal.md)

- [Ajouter ou supprimer des membres d’un groupe](active-directory-groups-members-azure-portal.md)

- [Modifier vos paramètres de groupe](active-directory-groups-settings-azure-portal.md)

- [Utilisation d’un groupe pour gérer l’accès aux applications SaaS](../enterprise-users/groups-saasapps.md)

- [Scénarios, limitations et problèmes connus liés à l’utilisation de groupes dans le cadre de la gestion des licences dans Azure Active Directory](../enterprise-users/licensing-group-advanced.md#limitations-and-known-issues)
