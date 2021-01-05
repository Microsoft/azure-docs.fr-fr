---
title: Modifier les plans de licence pour les utilisateurs et les groupes – Azure AD | Microsoft Docs
description: Migration des utilisateurs d’un groupe vers différents plans de service à l’aide de la gestion des licences de groupe dans Azure Active Directory
services: active-directory
keywords: Gestion des licences Azure AD
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: enterprise-users
ms.topic: how-to
ms.workload: identity
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: sumitp
ms.custom: it-pro;seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 050ae95c79e7ecb98f8508c2fdb41b90fc1b1da0
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96546537"
---
# <a name="change-license-assignments-for-a-user-or-group-in-azure-active-directory"></a>Modifier des affectations de licence pour un utilisateur ou un groupe dans Azure Active Directory

Cet article explique comment déplacer des utilisateurs et des groupes entre des plans de licences de services dans Azure Active Directory (Azure AD). L’objectif d’Azure AD est de veiller à ce qu’il n’y ait aucune perte de service ou données pendant le changement de licence. Les utilisateurs doivent passer d’un service à l’autre sans interruption. Les étapes d’affectation d’un plan de licence décrites dans cet article expliquent comment faire passer un utilisateur ou groupe d’Office 365 E1 à Office 365 E3, mais elles s’appliquent à tous les plans de licences. Quand vous mettez à jour des affectations de licence pour un utilisateur ou groupe, des suppressions et des ajouts sont effectués simultanément afin que les utilisateurs ne perdent pas l’accès à leurs services pendant les modifications de licences ou n’obtiennent pas de conflits de licence entre les plans.

## <a name="before-you-begin"></a>Avant de commencer

Avant de mettre à jour les affectations de licence, il est important de vérifier que certaines hypothèses sont vraies pour tous les utilisateurs ou groupes à mettre à jour. Si les hypothèses ne se vérifient pas pour tous les utilisateurs dans un groupe, la migration risque d’échouer pour certains d’entre eux. Certains utilisateurs peuvent alors perdre leur accès aux services ou aux données. Assurez-vous que :

- Les utilisateurs disposent du plan de licence (dans ce cas, Office 365 E1) affecté à un groupe et hérité par l’utilisateur, mais pas affecté directement.

- Vous avez suffisamment de licences disponibles pour le plan de licence que vous affectez. Si vous n’avez pas assez de licences, certains utilisateurs risquent de ne pas se voir affecter le nouveau plan de licence. Vous pouvez vérifier le nombre de licences disponibles.

- Les utilisateurs n’ont pas d’autres licences de services affectées pouvant entrer en conflit avec la licence voulue ou empêcher la suppression de la licence actuelle. Par exemple, une licence d’un service comme Workplace Analytics ou Project Online, qui possède une dépendance vis-à-vis d’autres services.

- Si vous gérez des groupes en local et les synchronisez dans Azure AD par le biais d’Azure AD Connect, alors vous ajoutez ou supprimer des utilisateurs à l’aide de votre système local. La synchronisation des modifications avec Azure AD peut prendre un certain temps pour apparaître dans les licences de groupe.

- Si vous utilisez des appartenances de groupes dynamiques Azure AD, vous ajoutez ou supprimez des utilisateurs en modifiant leurs attributs, mais le processus de mise à jour des affectations de licence reste le même.

## <a name="change-user-license-assignments"></a>Modifier des affectations de licence utilisateur

Dans la page **Mettre à jour des affectations de licence**, si vous voyez que certaines cases à cocher ne sont pas disponibles, cela signifie que les services ne sont pas modifiables, car ils sont hérités à partir d’une licence de groupe.

1. Connectez-vous au [Portail Azure](https://portal.azure.com/) à l’aide d’un compte Administrateur de licence dans votre organisation Azure AD.
1. Sélectionnez **Azure Active Directory** > **Utilisateurs**, puis ouvrez la page **Profil** d’un utilisateur.
1. Sélectionnez **Licences**.
1. Sélectionnez **Affectations** pour modifier l’affectation de licence de l’utilisateur ou du groupe. La page **Affectations** vous permet de résoudre les conflits d’affectation de licence.
1. Cochez la case Office 365 E3 et vérifiez que tous les services E1 affectés à l’utilisateur sont au minimum sélectionnés.
1. Décochez la case Office 365 E1.

    ![Page Affectations de licence d’un utilisateur avec Office 365 E1 décoché et Office 365 E3 coché](./media/licensing-groups-change-licenses/update-user-license-assignments.png)

1. Sélectionnez **Enregistrer**.

Azure AD applique les nouvelles licences et supprime les anciennes simultanément pour assurer la continuité du service.

## <a name="change-group-license-assignments"></a>Modifier des affectations de licence de groupe

1. Connectez-vous au [Portail Azure](https://portal.azure.com/) à l’aide d’un compte Administrateur de licence dans votre organisation Azure AD.
1. Sélectionnez **Azure Active Directory** > **Groupes**, puis ouvrez la page **Vue d’ensemble** d’un groupe.
1. Sélectionnez **Licences**.
1. Sélectionnez la commande **Affectations** pour modifier l’affectation de licence de l’utilisateur ou du groupe.
1. Cochez la case Office 365 E3. Pour assurer la continuité du service, veillez à sélectionner tous les services E1 déjà affectés à l’utilisateur.
1. Décochez la case Office 365 E1.

    ![Sélection de la commande Affectations dans une page de licences d’utilisateur ou de groupe](./media/licensing-groups-change-licenses/update-group-license-assignments.png)

1. Sélectionnez **Enregistrer**.

Pour assurer la continuité du service, Azure AD applique les nouvelles licences et supprime les anciennes simultanément.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur d’autres scénarios de gestion des licences par le biais des groupes, consultez les articles suivants :

- [Affectation de licences à un groupe dans Azure Active Directory](licensing-groups-assign.md)
- [Identification et résolution des problèmes de licence pour un groupe dans Azure Active Directory](licensing-groups-resolve-problems.md)
- [Guide pratique pour migrer des utilisateurs individuels sous licence vers des licences de groupe dans Azure Active Directory](licensing-groups-migrate-users.md)
- [Autres scénarios de gestion des licences de groupe Azure Active Directory](licensing-group-advanced.md)
- [Exemples PowerShell de gestion des licences de groupe dans Azure Active Directory](licensing-ps-examples.md)
