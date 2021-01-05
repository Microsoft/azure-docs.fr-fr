---
title: Ajouter ou supprimer des attributions de rôle Azure avec le portail Azure - Azure RBAC
description: Découvrez comment accorder l’accès aux ressources Azure pour les utilisateurs, groupes, principaux de service ou identités managées à l’aide du portail Azure et du contrôle d’accès en fonction du rôle Azure (Azure RBAC).
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 09/30/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 1aebb80fcba6c21ccce63772195af80b94092bde
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92368316"
---
# <a name="add-or-remove-azure-role-assignments-using-the-azure-portal"></a>Ajouter ou supprimer des attributions de rôle Azure à l’aide du portail Azure

[!INCLUDE [Azure RBAC definition grant access](../../includes/role-based-access-control-definition-grant.md)] Cet article explique comment attribuer des rôles à l’aide du portail Azure.

Si vous devez attribuer des rôles d’administrateur dans Azure Active Directory, consultez [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](../active-directory/roles/manage-roles-portal.md).

## <a name="prerequisites"></a>Prérequis

Pour ajouter ou supprimer des attributions de rôles, vous devez disposer :

- d’autorisations `Microsoft.Authorization/roleAssignments/write` et `Microsoft.Authorization/roleAssignments/delete`, telles que [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator) ou [Propriétaire de l’accès utilisateur](built-in-roles.md#owner)

## <a name="access-control-iam"></a>Contrôle d’accès (IAM)

La page **Contrôle d’accès (IAM)** vous permet généralement d’attribuer des rôles afin d’accorder l’accès aux ressources Azure. Il permet de gérer les identités et les accès et apparaît dans plusieurs emplacements du portail Azure. Voici un exemple de page Contrôle d'accès (IAM) pour un abonnement.

![Page Contrôle d’accès (IAM) pour un abonnement](./media/role-assignments-portal/access-control-subscription.png)

Pour être le plus efficace possible avec la page Contrôle d’accès (IAM), vous pouvez suivre ces étapes pour attribuer un rôle.

1. Déterminer qui a besoin d’un accès. Vous pouvez attribuer un rôle à un utilisateur, à un groupe, à un principal de service ou à une identité managée.

1. Rechercher le rôle approprié. Les autorisations sont regroupées dans des rôles. Vous pouvez choisir parmi une liste de plusieurs [rôles intégrés Azure](built-in-roles.md) ou utiliser vos propres rôles personnalisés.

1. Identifier l’étendue nécessaire. Azure fournit quatre niveaux d’étendue : [groupe d’administration](../governance/management-groups/overview.md), abonnement, [groupe de ressource](../azure-resource-manager/management/overview.md#resource-groups) et ressource. Pour plus d’informations sur l’étendue, consultez [Comprendre l’étendue](scope-overview.md).

1. Suivez la procédure décrite dans l’une des sections suivantes pour attribuer un rôle.

## <a name="add-a-role-assignment"></a>Ajouter une attribution de rôle

Dans Azure RBAC, vous devez ajouter une attribution de rôle pour accorder l’accès à une ressource Azure. Suivez ces étapes pour attribuer un rôle.

1. Dans le Portail Azure, cliquez sur **Tous les services** , puis sélectionnez l’étendue à laquelle vous souhaitez accorder l’accès. Par exemple, vous pouvez sélectionner **Groupes d’administration** , **Abonnements** , **Groupes de ressources** , ou une ressource.

1. Cliquez sur la ressource spécifique pour cette étendue.

1. Cliquez sur **Contrôle d’accès (IAM)** .

1. Cliquez sur l’onglet **Attributions de rôles** afin d’afficher les attributions de rôles pour cette étendue.

    ![Contrôle d’accès (IAM) et onglet Attributions de rôles](./media/role-assignments-portal/role-assignments.png)

1. Cliquez sur **Ajouter** > **Ajouter une attribution de rôle** .

   Si vous n’avez pas les autorisations pour attribuer des rôles, l’option Ajouter une attribution de rôle sera désactivée.

   ![Menu Ajouter une attribution de rôle](./media/shared/add-role-assignment-menu.png)

    Le volet Ajouter une attribution de rôle s’ouvre.

   ![Volet Ajouter une attribution de rôle](./media/role-assignments-portal/add-role-assignment.png)

1. Dans la liste déroulante **Rôle** , sélectionnez un rôle, tel que **Contributeur de machines virtuelles** .

1. Dans la liste **Sélectionner** , sélectionnez un utilisateur, un groupe, un principal de service ou une identité gérée. Si vous ne voyez pas le principal de sécurité dans la liste, vous pouvez saisir du texte dans la zone **Sélectionner** pour rechercher des noms d’affichage, des adresses de messagerie et des identificateurs d’objet dans le répertoire.

1. Cliquez sur **Enregistrer** pour attribuer le rôle.

   Après quelques instants, le principal de sécurité est attribué au rôle dans l’étendue sélectionnée.

    ![Enregistrement Ajouter une attribution de rôle](./media/role-assignments-portal/add-role-assignment-save.png)

## <a name="assign-a-user-as-an-administrator-of-a-subscription"></a>Attribuer à un utilisateur en tant qu’administrateur d’un abonnement

Pour faire d’un utilisateur un administrateur d’un abonnement Azure, attribuez-lui le rôle [Propriétaire](built-in-roles.md#owner) au niveau de l’abonnement. Le rôle Propriétaire donne à l’utilisateur un accès total à toutes les ressources de l’abonnement, notamment l’autorisation d’accorder l’accès à d’autres personnes. Ces étapes sont les mêmes que celles de n’importe quelle autre attribution de rôle.

1. Dans le portail Azure, cliquez sur **Tous les services** , puis sur **Abonnements** .

1. Cliquez sur l’abonnement pour lequel vous souhaitez accorder l’accès.

1. Cliquez sur **Contrôle d’accès (IAM)** .

1. Cliquez sur l’onglet **Attributions de rôles** afin d’afficher les attributions de rôles pour cet abonnement.

    ![Contrôle d’accès (IAM) et onglet Attributions de rôles](./media/role-assignments-portal/role-assignments.png)

1. Cliquez sur **Ajouter** > **Ajouter une attribution de rôle** .

   Si vous n’avez pas les autorisations pour attribuer des rôles, l’option Ajouter une attribution de rôle sera désactivée.

   ![Menu Ajouter une attribution de rôle pour un abonnement](./media/shared/add-role-assignment-menu.png)

    Le volet Ajouter une attribution de rôle s’ouvre.

   ![Volet Ajouter une attribution de rôle pour un abonnement](./media/role-assignments-portal/add-role-assignment.png)

1. Dans la liste déroulante **Rôle** , sélectionnez le rôle **Propriétaire** .

1. Dans la liste **Sélectionner** , sélectionnez un utilisateur. Si vous ne voyez pas l’utilisateur dans la liste, vous pouvez taper dans la zone **Sélectionner** pour rechercher des noms d’affichage et des adresses e-mail dans l’annuaire.

1. Cliquez sur **Enregistrer** pour attribuer le rôle.

   Après quelques instants, le rôle Propriétaire est attribué à l’utilisateur au niveau de l’abonnement.

## <a name="add-a-role-assignment-for-a-managed-identity-preview"></a>Ajouter une attribution de rôle pour une identité managée (préversion)

Vous pouvez ajouter des attributions de rôles pour une identité managée à l’aide de la page **Contrôle d’accès (IAM)** , conformément à la description plus haut dans cet article. Lorsque vous utilisez la page Contrôle d’accès (IAM), vous démarrez avec l’étendue, puis vous sélectionnez l’identité managée et le rôle. Cette section décrit une autre façon d’ajouter des attributions de rôles pour une identité managée. En suivant ces étapes, vous commencez par l’identité managée, puis vous sélectionnez l’étendue et le rôle.

> [!IMPORTANT]
> L’ajout d’une attribution de rôle pour une identité managée à l’aide en suivant ces autres étapes est actuellement en préversion.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### <a name="system-assigned-managed-identity"></a>Identité managée affectée par le système

Procédez comme suit pour attribuer un rôle à une identité managée attribuée par le système en commençant par l’identité managée.

1. Dans le Portail Azure, ouvrez une identité gérée affectée par le système.

1. Dans le menu de gauche, cliquez sur **identité** .

    ![Identité managée affectée par le système](./media/shared/identity-system-assigned.png)

1. Sous **Autorisations** , cliquez sur **Attributions de rôles Azure** .

    Si des rôles sont déjà attribués à l’identité managée attribuée par le système sélectionnée, la liste des attributions de rôles s’affiche. Cette liste inclut toutes les attributions de rôles que vous êtes autorisé à lire.

    ![Attributions de rôles pour une identité gérée affectée par le système](./media/shared/role-assignments-system-assigned.png)

1. Pour changer d’abonnement, cliquez sur la liste **Abonnement** .

1. Cliquez sur **Ajouter une attribution de rôle (préversion)** .

1. Utilisez les listes déroulantes pour sélectionner l’ensemble de ressources auquel s’applique l’attribution de rôle, par exemple **Abonnement** , **Groupe de ressources** ou ressource.

    Si vous n’avez pas d’autorisations d’accès en écriture pour l’étendue sélectionnée, un message inline s’affiche. 

1. Dans la liste déroulante **Rôle** , sélectionnez un rôle, tel que **Contributeur de machines virtuelles** .

   ![Volet Ajouter une attribution de rôle pour une identité managée affectée par le système](./media/role-assignments-portal/add-role-assignment-with-scope.png)

1. Cliquez sur **Enregistrer** pour attribuer le rôle.

   Après quelques instants, le rôle est attribué à l’identité managée dans l’étendue sélectionnée.

### <a name="user-assigned-managed-identity"></a>Identité managée affectée par l’utilisateur

Procédez comme suit pour attribuer un rôle à une identité managée attribuée par l’utilisateur en commençant par l’identité managée.

1. Dans le Portail Azure, ouvrez une identité gérée affectée par l’utilisateur.

1. Dans le menu gauche, cliquez sur **Attributions de rôles Azure** .

    Si des rôles sont déjà attribués à l’identité managée attribuée par l’utilisateur sélectionnée, la liste des attributions de rôles s’affiche. Cette liste inclut toutes les attributions de rôles que vous êtes autorisé à lire.

    ![Attributions de rôles pour une identité managée affectée par l’utilisateur](./media/shared/role-assignments-user-assigned.png)

1. Pour changer d’abonnement, cliquez sur la liste **Abonnement** .

1. Cliquez sur **Ajouter une attribution de rôle (préversion)** .

1. Utilisez les listes déroulantes pour sélectionner l’ensemble de ressources auquel s’applique l’attribution de rôle, par exemple **Abonnement** , **Groupe de ressources** ou ressource.

    Si vous n’avez pas d’autorisations d’accès en écriture pour l’étendue sélectionnée, un message inline s’affiche. 

1. Dans la liste déroulante **Rôle** , sélectionnez un rôle, tel que **Contributeur de machines virtuelles** .

   ![Volet Ajouter une attribution de rôle pour une identité managée affectée par l’utilisateur](./media/role-assignments-portal/add-role-assignment-with-scope.png)

1. Cliquez sur **Enregistrer** pour attribuer le rôle.

   Après quelques instants, le rôle est attribué à l’identité managée dans l’étendue sélectionnée.

## <a name="remove-a-role-assignment"></a>Supprimer une attribution de rôle

Dans Azure RBAC, vous devez supprimer une attribution de rôle pour supprimer l’accès à une ressource Azure. Suivez ces étapes pour supprimer une attribution de rôle.

1. Ouvrez **Contrôle d’accès (IAM)** dans une étendue, par exemple un groupe d’administration, un abonnement, un groupe de ressources ou une ressource pour lesquels vous souhaitez supprimer l’accès.

1. Cliquez sur l’onglet **Attributions de rôles** afin d’afficher toutes les attributions de rôles pour cet abonnement.

1. Dans la liste des attributions de rôle, ajoutez une coche à côté du principal de sécurité comportant l’attribution de rôle que vous souhaitez supprimer.

   ![Attribution de rôle sélectionnée pour être supprimée](./media/role-assignments-portal/remove-role-assignment-select.png)

1. Cliquez sur **Supprimer** .

   ![Supprimer le message d’attribution de rôle](./media/role-assignments-portal/remove-role-assignment.png)

1. Dans le message d’attribution de rôle qui s’affiche, cliquez sur **Oui** .

    Si vous voyez un message indiquant que les attributions de rôles héritées ne peuvent pas être supprimées, cela signifie que vous tentez de supprimer une attribution de rôle dans une étendue enfant. Vous devez ouvrir le contrôle d’accès (IAM) dans l’étendue où le rôle a été attribué, puis réessayer. Un moyen rapide d’ouvrir le contrôle d’accès (IAM) à l’étendue correcte consiste à examiner la colonne **Étendue** , puis à cliquer sur le lien à côté de **(Héritée)** .

   ![Message Supprimer l’attribution de rôle pour les attributions de rôles héritées](./media/role-assignments-portal/remove-role-assignment-inherited.png)

## <a name="next-steps"></a>Étapes suivantes

- [Répertorier les attributions de rôle Azure à l’aide du portail Azure](role-assignments-list-portal.md)
- [Tutoriel : Accorder un accès utilisateur aux ressources Azure à l’aide du portail Azure](quickstart-assign-role-user-portal.md)
- [Résoudre les problèmes liés à Azure RBAC](troubleshooting.md)
- [Organiser vos ressources avec des groupes d’administration Azure](../governance/management-groups/overview.md)
