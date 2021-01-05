---
title: Afficher, ajouter et supprimer des affectations pour un package d’accès dans la gestion des droits d’utilisation d’Azure Active Directory - Azure Active Directory
description: Découvrez comment afficher, ajouter et supprimer des affectations pour un package d’accès dans la gestion des droits d’utilisation d’Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 18bc7ca9e108a35fd27f7b3155f186221a8caae3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90980056"
---
# <a name="view-add-and-remove-assignments-for-an-access-package-in-azure-ad-entitlement-management"></a>Afficher, ajouter et supprimer des affectations pour un package d’accès dans la gestion des droits d’utilisation d’Azure Active Directory

Dans la gestion des droits d'utilisation Azure AD, vous pouvez voir qui a été affecté aux packages d’accès, ainsi que leur stratégie et leur état. Si un package d’accès a une stratégie appropriée, vous pouvez également affecter directement l’utilisateur à un package d’accès. Cet article explique comment afficher, ajouter et supprimer des affectations pour des packages d’accès.

## <a name="prerequisites"></a>Prérequis

Pour utiliser la gestion des droits d’utilisation Azure AD et affecter des utilisateurs aux packages d’accès, vous devez disposer d’une des licences suivantes :

- Azure AD Premium P2
- Licence Enterprise Mobility + Security (EMS) E5

## <a name="view-who-has-an-assignment"></a>Afficher les utilisateurs qui ont une affectation

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs, gestionnaire de package d'accès ou gestionnaire d’affectation de package d’accès

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Packages d’accès**, puis ouvrez le package d'accès.

1. Cliquez sur **Affectations** pour afficher une liste des affectations actives.

    ![Liste des affectations à un package d’accès](./media/entitlement-management-access-package-assignments/assignments-list.png)

1. Cliquez sur une affectation spécifique pour voir plus de détails.

1. Pour afficher une liste des affectations dont les rôles de ressources n’étaient pas correctement provisionnés, cliquez sur filtre d’état et sélectionnez **Livraison en cours**.

    Vous pouvez voir des détails supplémentaires sur les erreurs de remise en recherchant la requête correspondante de l'utilisateur sur la page **Requêtes**.

1. Pour voir les affectations expirées, cliquez sur le filtre d’état et sélectionnez **Expiré**.

1. Pour télécharger un fichier CSV contenant la liste filtrée, cliquez sur **Télécharger**.

### <a name="viewing-assignments-programmatically"></a>Affichage d’affectations par programmation

Vous pouvez également récupérer des affectations dans un package d’accès à l’aide de Microsoft Graph.  Un utilisateur doté d’un rôle approprié avec une application disposant de l’autorisation `EntitlementManagement.ReadWrite.All` peut appeler l’API pour [répertorier les affectations accessPackageAssignments](/graph/api/accesspackageassignment-list?view=graph-rest-beta).

## <a name="directly-assign-a-user"></a>Affecter directement un utilisateur

Dans certains cas, vous pouvez affecter directement des utilisateurs spécifiques à un package d'accès pour leur éviter d’avoir à passer par le processus de demande du package d'accès. Pour affecter directement des utilisateurs, le package d'accès doit avoir une stratégie qui autorise les affectations directes par l'administrateur.

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs, gestionnaire de package d'accès ou gestionnaire d’affectation de package d’accès

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Packages d’accès**, puis ouvrez le package d'accès.

1. Dans le menu gauche, cliquez sur **Affectations**.

1. Cliquez sur **Nouvelle affectation** pour ouvrir la fenêtre Ajouter un utilisateur au package d’accès.

    ![Affectations - Ajouter un utilisateur au package d’accès](./media/entitlement-management-access-package-assignments/assignments-add-user.png)

1. Cliquez sur **Ajouter des utilisateurs** pour sélectionner les utilisateurs auxquels vous voulez affecter ce package d’accès.

1. Dans la liste **Sélectionner une stratégie**, sélectionnez une stratégie selon laquelle les requêtes et le cycle de vie des utilisateurs seront régis et suivis. Si vous souhaitez que les utilisateurs sélectionnés aient des paramètres de stratégie différents, vous pouvez cliquer sur **Créer une nouvelle stratégie** pour ajouter une nouvelle stratégie.

1. Définissez la date et l'heure de début et de fin de l'affectation des utilisateurs sélectionnés. Si aucune date de fin n’est fournie, les paramètres du cycle de vie de la stratégie seront utilisés.

1. Si vous le souhaitez, fournissez une justification de votre affectation directe à des fins d’archivage.

1. Cliquez sur **Ajouter** pour affecter directement les utilisateurs sélectionnés au package d'accès.

    Après quelques instants, cliquez sur **Actualiser** pour voir les utilisateurs dans la liste des affectations.

### <a name="directly-assigning-users-programmatically"></a>Affectation d’utilisateurs directement par programmation

Vous pouvez également affecter directement un utilisateur à un package d’accès à l’aide de Microsoft Graph.  Un utilisateur doté d’un rôle approprié avec une application disposant de l’autorisation déléguée `EntitlementManagement.ReadWrite.All` peut appeler l’API pour [créer une demande accessPackageAssignmentRequest](/graph/api/accesspackageassignmentrequest-post?view=graph-rest-beta).

## <a name="remove-an-assignment"></a>Supprimer une affectation

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs, gestionnaire de package d'accès ou gestionnaire d’affectation de package d’accès

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Packages d’accès**, puis ouvrez le package d'accès.

1. Dans le menu gauche, cliquez sur **Affectations**.
 
1. Cochez la case en regard de l’utilisateur dont vous souhaitez supprimer l’affectation du package d’accès. 

1. Cliquez sur le bouton **Supprimer** en haut du volet gauche. 
 
    ![Affectations - Supprimer un utilisateur du package d’accès](./media/entitlement-management-access-package-assignments/remove-assignment-select-remove-assignment.png)

    Une notification s’affiche pour vous informer que l’affectation a été supprimée. 

## <a name="next-steps"></a>Étapes suivantes

- [Changer la demande et les paramètres pour un package d’accès](entitlement-management-access-package-request-policy.md)
- [Afficher les rapports et les journaux](entitlement-management-reports.md)