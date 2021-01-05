---
title: Répertorier les attribution de rôle Azure à l’aide du portail Azure – Azure RBAC
description: Découvrez comment déterminer les ressources, utilisateurs, groupes, principaux de service ou identités managées qui ont accès au portail Azure et au contrôle d’accès en fonction du rôle Azure (Azure RBAC).
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.topic: how-to
ms.workload: identity
ms.date: 10/16/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: a5b019068368f1c0c076f5a1ed157c288bb7169c
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92479916"
---
# <a name="list-azure-role-assignments-using-the-azure-portal"></a>Répertorier les attributions de rôles Azure à l’aide du portail Azure

[!INCLUDE [Azure RBAC definition list access](../../includes/role-based-access-control-definition-list.md)] Cet article explique comment lister les attributions de rôles à l’aide du portail Azure.

> [!NOTE]
> Si votre organisation possède des fonctions de gestion externalisées pour un fournisseur de services qui utilise la [gestion des ressources déléguées Azure](../lighthouse/concepts/azure-delegated-resource-management.md), les attributions de rôles autorisées par ce fournisseur de services ne seront pas affichées ici.

## <a name="list-role-assignments-for-a-user-or-group"></a>Lister les attributions de rôles pour un utilisateur ou un groupe

La façon la plus simple de voir les rôles attribués à un utilisateur ou à un groupe dans un abonnement consiste à utiliser le volet **Attributions de rôles Azure** .

1. Dans le Portail Azure, sélectionnez **Tous les services** dans le menu Portail Azure.

1. Sélectionnez **Azure Active Directory** , puis **Utilisateurs** ou **Groupes** .

1. Cliquez sur l’utilisateur ou le groupe pour lequel vous voulez lister les attributions de rôles.

1. Cliquez sur **Attributions de rôles Azure** .

    Vous voyez une liste de rôles attribués à l’utilisateur ou au groupe sélectionné dans différentes étendues, comme un groupe d’administration, un abonnement, un groupe de ressources ou une ressource. Cette liste inclut toutes les attributions de rôles que vous êtes autorisé à lire.

    ![Attributions de rôles pour un utilisateur](./media/role-assignments-list-portal/azure-role-assignments-user.png)    

1. Pour changer d’abonnement, cliquez sur la liste **Abonnements** .

## <a name="list-owners-of-a-subscription"></a>Répertorier les propriétaires d’un abonnement

Les utilisateurs qui se sont vu attribuer le rôle [Propriétaire](built-in-roles.md#owner) pour un abonnement peuvent gérer tous les éléments de l’abonnement. Procédez comme suit pour répertorier les propriétaires d’un abonnement.

1. Dans le portail Azure, cliquez sur **Tous les services** , puis sur **Abonnements** .

1. Cliquez sur l’abonnement dont vous souhaitez répertorier les propriétaires.

1. Cliquez sur **Contrôle d’accès (IAM)** .

1. Cliquez sur l’onglet **Attributions de rôles** afin d’afficher toutes les attributions de rôles pour cet abonnement.

1. Faites défiler la page jusqu’à la section **Propriétaires** pour voir tous les utilisateurs auxquels le rôle Propriétaire a été attribué pour cet abonnement.

   ![Contrôle d’accès à l’abonnement – Onglet Attributions de rôles](./media/role-assignments-list-portal/access-control-role-assignments-subscription.png)

## <a name="list-role-assignments-at-a-scope"></a>Lister les attributions de rôles dans une étendue

1. Dans le portail Azure, cliquez sur **Tous les services** , puis sélectionnez l’étendue. Par exemple, vous pouvez sélectionner **Groupes d’administration** , **Abonnements** , **Groupes de ressources** , ou une ressource.

1. Cliquez sur la ressource spécifique.

1. Cliquez sur **Contrôle d’accès (IAM)** .

1. Cliquez sur l’onglet **Attributions de rôles** afin d’afficher toutes les attributions de rôles pour cette étendue.

   ![Contrôle d’accès - Onglet Attributions de rôles](./media/role-assignments-list-portal/access-control-role-assignments.png)

   L’onglet Attributions de rôles indique les personnes qui ont accès à cette étendue. Notez que certains rôles sont inclus dans l’étendue de **cette ressource** , tandis que d’autres sont **hérités** à partir d’une autre étendue. L’accès est attribué spécifiquement à cette ressource ou hérité d’une affectation à l’étendue parente.

## <a name="list-role-assignments-for-a-user-at-a-scope"></a>Lister les attributions de rôles pour un utilisateur dans une étendue

Pour lister l’accès pour un utilisateur, un groupe, un principal de service ou une identité managée, vous listez leurs attributions de rôles. Suivez ces étapes pour lister les attributions de rôles pour un utilisateur, un groupe, un principal de service ou une identité managée dans une étendue particulière.

1. Dans le portail Azure, cliquez sur **Tous les services** , puis sélectionnez l’étendue. Par exemple, vous pouvez sélectionner **Groupes d’administration** , **Abonnements** , **Groupes de ressources** , ou une ressource.

1. Cliquez sur la ressource spécifique.

1. Cliquez sur **Contrôle d’accès (IAM)** .

1. Cliquez sur l’onglet **Vérifier l’accès** .

    ![Contrôle d’accès - onglet Vérifier l’accès](./media/role-assignments-list-portal/access-control-check-access.png)

1. Dans la liste **Rechercher** , sélectionnez le type principal de sécurité dont vous souhaitez contrôler l’accès.

1. Dans la zone de recherche, entrez une chaîne afin de rechercher, dans le répertoire, des noms d’affichage, des adresses e-mail ou des identificateurs d’objet.

    ![Liste de sélection Vérifier l’accès](./media/role-assignments-list-portal/check-access-select.png)

1. Cliquez sur le principal de sécurité pour ouvrir le volet **Affectations** .

    ![Volet Affectations](./media/role-assignments-list-portal/check-access-assignments.png)

    Dans ce volet, vous pouvez voir les rôles attribués au principal de sécurité sélectionné et la portée. S’il existe des affectations de refus dans cette étendue, ou héritées par cette étendue, elles sont listées.

## <a name="list-role-assignments-for-a-managed-identity"></a>Lister les attributions de rôles pour une identité managée

Vous pouvez répertorier les attributions de rôles pour les identités gérées attribuées par le système et affectées par l’utilisateur au niveau d’une étendue particulière à l’aide du panneau de **Contrôle d’accès (IAM)** , comme décrit précédemment. Cette section décrit comment répertorier les attributions de rôles uniquement pour l’identité managée.

### <a name="system-assigned-managed-identity"></a>Identité managée affectée par le système

1. Dans le Portail Azure, ouvrez une identité gérée affectée par le système.

1. Dans le menu de gauche, cliquez sur **identité** .

    ![Identité managée affectée par le système](./media/shared/identity-system-assigned.png)

1. Sous **Autorisations** , cliquez sur **Attributions de rôles Azure** .

    Vous voyez une liste de rôles affectés à l’identité gérée affectée par le système sélectionnée à différentes étendues, telles que groupe d’administration, abonnement, groupe de ressources ou ressource. Cette liste inclut toutes les attributions de rôles que vous êtes autorisé à lire.

    ![Attributions de rôles pour une identité gérée affectée par le système](./media/shared/role-assignments-system-assigned.png)

1. Pour changer d’abonnement, cliquez sur la liste **Abonnement** .

### <a name="user-assigned-managed-identity"></a>Identité managée affectée par l’utilisateur

1. Dans le Portail Azure, ouvrez une identité gérée affectée par l’utilisateur.

1. Cliquez sur **Attributions de rôles Azure** .

    Vous voyez une liste de rôles affectés à l’identité gérée affectée à l’utilisateur sélectionnée à différentes étendues, telles que groupe d’administration, abonnement, groupe de ressources ou ressource. Cette liste inclut toutes les attributions de rôles que vous êtes autorisé à lire.

    ![Capture d’écran montrant les attributions de rôles pour une identité gérée affectée par l’utilisateur.](./media/shared/role-assignments-user-assigned.png)

1. Pour changer d’abonnement, cliquez sur la liste **Abonnement** .

## <a name="list-number-of-role-assignments"></a>Répertorier le nombre d’attributions de rôles

Vous pouvez avoir jusqu’à **2 000**  attributions de rôles dans chaque abonnement. Cette limite comprend les attributions de rôles au niveau de l’abonnement, du groupe de ressources et des étendues de ressources. Pour vous aider à suivre cette limite, l’onglet **Attributions de rôles** comprend un graphe indiquant le nombre d’attributions de rôles de l’abonnement actif.

![Contrôle d’accès : graphique du nombre d’attributions de rôles](./media/role-assignments-list-portal/access-control-role-assignments-chart.png)

Si vous vous approchez du nombre maximal et que vous essayez d’ajouter d’autres attributions de rôles, un avertissement s’affiche dans le volet **Ajouter une attribution de rôle** . Pour savoir comment réduire le nombre d’attributions de rôles, consultez [Résolution des problèmes de contrôle d’accès en fonction du rôle Azure](troubleshooting.md#azure-role-assignments-limit).

![Contrôle d’accès : avertissement Ajouter une attribution de rôle](./media/role-assignments-list-portal/add-role-assignment-warning.png)

## <a name="download-role-assignments"></a>Télécharger les attributions de rôle

Vous pouvez télécharger des attributions de rôles au niveau d’une étendue au format CSV ou JSON. Cela peut être utile si vous avez besoin d’inspecter la liste dans une feuille de calcul ou d’effectuer un inventaire lors de la migration d’un abonnement.

Lorsque vous téléchargez des attributions de rôles, vous devez garder à l’esprit les critères suivants :

- Si vous ne disposez pas des autorisations nécessaires pour lire le répertoire, par exemple le rôle Lecteur de répertoires, les colonnes DisplayName, SignInName et ObjectType sont vides.
- Les attributions de rôles dont l’entité de sécurité a été supprimée ne sont pas incluses.
- L’accès accordé aux administrateurs classiques n’est pas inclus.

Procédez comme suit pour télécharger les attributions de rôles sur une étendue.

1. Dans le portail Azure, cliquez sur **Tous les services** , puis sélectionnez l’étendue sur laquelle vous souhaitez télécharger les attributions de rôles. Par exemple, vous pouvez sélectionner **Groupes d’administration** , **Abonnements** , **Groupes de ressources** , ou une ressource.

1. Cliquez sur la ressource spécifique.

1. Cliquez sur **Contrôle d’accès (IAM)** .

1. Cliquez sur **Télécharger les attributions de rôles** pour ouvrir le volet Télécharger les attributions de rôles.

    ![Contrôle d’accès : Télécharger les attributions de rôles](./media/role-assignments-list-portal/download-role-assignments.png)

1. Utilisez les cases à cocher pour sélectionner les attributions de rôles que vous souhaitez inclure dans le fichier téléchargé.

    - **Héritées**  : inclut les attributions de rôles héritées pour l'étendue actuelle.
    - **Sur l’étendue actuelle**  : inclut les attributions de rôles pour l'étendue actuelle.
    - **Enfants**  : inclut les attributions de rôles aux niveaux inférieurs à l'étendue actuelle. Cette case à cocher est désactivée pour l’étendue du groupe d’administration.

1. Sélectionnez le format de fichier CSV (fichier de valeurs séparées par des virgules) ou le format JSON (JavaScript Object Notation).

1. Spécifiez le nom du fichier.

1. Cliquez sur **Démarrer** pour lancer le téléchargement.

    Les éléments suivants présentent des exemples de sortie pour chaque format de fichier.

    ![Télécharger les attributions de rôles au format CSV](./media/role-assignments-list-portal/download-role-assignments-csv.png)

    ![Capture d’écran des attributions de rôle téléchargées au format JSON.](./media/role-assignments-list-portal/download-role-assignments-json.png)

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter ou supprimer des attributions de rôles Azure avec le portail Azure](role-assignments-portal.md)
- [Résoudre les problèmes liés à Azure RBAC](troubleshooting.md)
