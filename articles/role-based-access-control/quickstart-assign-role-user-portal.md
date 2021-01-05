---
title: 'Tutoriel : Accorder un accès utilisateur aux ressources Azure à l’aide du portail Azure - RBAC Azure'
description: Découvrez dans ce tutoriel comment accorder un accès utilisateur aux ressources Azure à l’aide du portail Azure et du contrôle d’accès en fonction du rôle Azure (RBAC Azure).
services: role-based-access-control
documentationCenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: tutorial
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 02/22/2019
ms.author: rolyon
ms.openlocfilehash: c6736bb2642ff08d0aa3cda536ef1599324efded
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85362010"
---
# <a name="tutorial-grant-a-user-access-to-azure-resources-using-the-azure-portal"></a>Tutoriel : Accorder un accès utilisateur aux ressources Azure à l’aide du portail Azure

Le [contrôle d’accès en fonction du rôle Azure (RBAC Azure)](overview.md) vous permet de gérer l’accès aux ressources Azure. Dans ce tutoriel, vous allez autoriser un utilisateur à créer et à gérer des machines virtuelles dans un groupe de ressources.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Accorder l’accès à un utilisateur au niveau du groupe de ressources
> * Suppression d'accès

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

1. Dans la liste de navigation, cliquez sur **Groupes de ressources**.

1. Cliquez sur **Ajouter** pour ouvrir le panneau **Groupe de ressources**.

   ![Ajouter un nouveau groupe de ressources](./media/quickstart-assign-role-user-portal/resource-group.png)

1. Dans **Nom du groupe de ressources**, entrez **rbac-resource-group**.

1. Sélectionnez un abonnement et un emplacement.

1. Cliquez sur **Créer** pour créer le groupe de ressources.

1. Cliquez sur **Actualiser** pour actualiser la liste des groupes de ressources.

   Le nouveau groupe de ressources apparaît dans la liste de vos groupes de ressources.

   ![Liste de groupes de ressources](./media/quickstart-assign-role-user-portal/resource-group-list.png)

## <a name="grant-access"></a>Accorder l'accès

Dans le contrôle d’accès en fonction du rôle Azure, vous créez une attribution de rôle pour accorder l’accès.

1. Dans la liste **Groupes de ressources**, cliquez sur le nouveau groupe de ressources **rbac-resource-group**.

1. Cliquez sur **Contrôle d’accès (IAM)** .

1. Cliquez sur l’onglet **Attributions de rôles** pour afficher la liste actuelle des attributions de rôles.

   ![Panneau Contrôle d’accès (IAM) pour le groupe de ressources](./media/quickstart-assign-role-user-portal/access-control.png)

1. Cliquez sur **Ajouter** > **Ajouter une attribution de rôle** pour ouvrir le volet Ajouter une attribution de rôle.

   Si vous n’avez pas les autorisations pour attribuer des rôles, l’option Ajouter une attribution de rôle sera désactivée.

   ![Menu Ajouter une attribution de rôle](./media/shared/add-role-assignment-menu.png)

    Le volet Ajouter une attribution de rôle s’ouvre.

   ![Volet Ajouter une attribution de rôle](./media/quickstart-assign-role-user-portal/add-role-assignment.png)

1. Dans la liste déroulante **Rôle**, sélectionnez **Contributeur de machines virtuelles**.

1. Dans la liste **Sélectionner**, sélectionnez un utilisateur : vous-même ou un autre.

1. Cliquez sur **Enregistrer** pour créer l’attribution de rôle.

   Après quelques instants, l’utilisateur se voit attribuer le rôle Contributeur de machines virtuelles dans l’étendue du groupe de ressources rbac-resource-group.

   ![Attribution du rôle Contributeur de machines virtuelles](./media/quickstart-assign-role-user-portal/vm-contributor-assignment.png)

## <a name="remove-access"></a>Suppression d'accès

Dans le RBAC Azure, pour supprimer un accès, vous supprimez une attribution de rôle.

1. Dans la liste des attributions de rôles, ajoutez une coche en regard de l’utilisateur titulaire du rôle Contributeur de machines virtuelles.

1. Cliquez sur **Supprimer**.

   ![Supprimer le message d’attribution de rôle](./media/quickstart-assign-role-user-portal/remove-role-assignment.png)

1. Dans le message d’attribution de rôle qui s’affiche, cliquez sur **Oui**.

## <a name="clean-up"></a>Nettoyer

1. Dans la liste de navigation, cliquez sur **Groupes de ressources**.

1. Cliquez sur **rbac-resource-group** pour ouvrir le groupe de ressources.

1. Cliquez sur **Supprimer le groupe de ressources** pour supprimer le groupe de ressources.

   ![Supprimer un groupe de ressources](./media/quickstart-assign-role-user-portal/delete-resource-group.png)

1. Dans le panneau **Voulez-vous vraiment supprimer**, tapez le nom du groupe de ressources : **rbac-resource-group**.

1. Cliquez sur **Supprimer** pour supprimer le groupe de ressources.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Accorder un accès utilisateur aux ressources Azure à l’aide d’Azure PowerShell](tutorial-role-assignments-user-powershell.md)
