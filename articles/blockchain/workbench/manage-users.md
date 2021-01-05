---
title: Gérer les utilisateurs dans Azure Blockchain Workbench
description: Guide de gestion des utilisateurs dans Azure Blockchain Workbench
ms.date: 07/15/2020
ms.topic: how-to
ms.reviewer: ravastra
ms.openlocfilehash: be078e7149ba008ab8b7ad44a016a0d1cf82df0e
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91263009"
---
# <a name="manage-users-in-azure-blockchain-workbench"></a>Gérer les utilisateurs dans Azure Blockchain Workbench

Azure Blockchain Workbench intègre la gestion des utilisateurs dédiée aux individus et aux organisations qui font partie de votre consortium.

## <a name="prerequisites"></a>Conditions préalables requises

Un déploiement Blockchain Workbench est requis. Consultez [Azure Blockchain Workbench deployment](deploy.md) (Déploiement d’Azure Blockchain Workbench) pour plus de détails sur le déploiement.

## <a name="add-azure-ad-users"></a>Ajouter des utilisateurs Azure AD

Azure Blockchain Workbench utilise Azure Active Directory (Azure AD) pour l’authentification, le contrôle des accès et les rôles. Les utilisateurs du client Blockchain Workbench Azure AD peuvent s’authentifier et utiliser Blockchain Workbench. Ajoutez des utilisateurs au rôle Administrateur d’application afin d’interagir et d’effectuer des actions.

Les utilisateurs Blockchain Workbench doivent exister dans le client Azure AD avant que vous ne puissiez les attribuer à des applications et à des rôles. Pour ajouter des utilisateurs à Azure AD, exécutez la procédure suivante :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez votre compte en haut à droite, puis basculez vers le client Azure AD associé à Blockchain Workbench.
1. Sélectionnez **Azure Active Directory > Utilisateurs**. Une liste des utilisateurs de votre répertoire s’affiche.
1. Pour ajouter des utilisateurs au répertoire, sélectionnez **Nouvel utilisateur**. Pour les utilisateurs externes, sélectionnez **Nouvel utilisateur invité**.
1. Renseignez les champs requis pour le nouvel utilisateur. Sélectionnez **Create** (Créer).

Consultez la documentation relative à [Azure AD](../../active-directory/fundamentals/add-users-azure-active-directory.md) pour en savoir plus sur la gestion des utilisateurs dans Azure AD.

## <a name="manage-blockchain-workbench-administrators"></a>Gérer les administrateurs Blockchain Workbench

Une fois que les utilisateurs ont été ajoutés au répertoire, il est temps de choisir ceux qui seront administrateurs Blockchain Workbench. Les utilisateurs du groupe **Administrateur** sont associés au **rôle Administrateur d’application** dans Blockchain Workbench. Les administrateurs peuvent ajouter ou supprimer des utilisateurs, affecter des utilisateurs à des scénarios spécifiques et créer des applications.

Pour ajouter des utilisateurs dans le groupe **Administrateur** du répertoire Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Vérifiez que vous vous trouvez dans le client Azure AD associé à Blockchain Workbench en sélectionnant votre compte en haut à droite.
1. Select **Azure Active Directory > Applications d’entreprise**.
1. Modifiez le filtre déroulant **Type d’application** en **Toutes les applications**, puis sélectionnez **Appliquer**.
1. Sélectionner l’application cliente Azure AD pour Azure Blockchain Workbench

    ![Ensemble des inscriptions à l’application d’entreprise](./media/manage-users/select-blockchain-client-app.png)

1. Sélectionnez **Utilisateurs et groupes > Ajouter un utilisateur**.
1. Sous **Ajouter une attribution**, sélectionnez **Utilisateurs**. Choisissez ou recherchez l’utilisateur que vous souhaitez ajouter comme administrateur. Cliquez sur **Sélectionner** à l’issue de la sélection.

    ![Ajouter une attribution](./media/manage-users/add-user-assignment.png)

1. Vérifiez que le **Rôle** est défini sur **Administrateur**.
1. Sélectionnez **Attribuer**. Les utilisateurs ajoutés sont affichés dans la liste, avec le rôle d’administrateur affecté.

    ![Utilisateurs de l’application cliente Blockchain](./media/manage-users/blockchain-admin-list.png)

## <a name="managing-blockchain-workbench-members"></a>Gestion des membres Blockchain Workbench

Utilisez l’application Blockchain Workbench pour gérer les utilisateurs et les organisations qui font partie de votre consortium. Vous pouvez ajouter ou supprimer des utilisateurs dans les applications et les rôles.

1. [Ouvrez Blockchain Workbench](deploy.md#blockchain-workbench-web-url) dans votre navigateur, puis connectez-vous en tant qu’administrateur.

    ![Blockchain Workbench](./media/manage-users/blockchain-workbench-applications.png)

    Les membres sont ajoutés à chaque application. Les membres peuvent posséder un ou plusieurs rôles d’application pour lancer les contrats ou prendre des mesures.

1. Pour gérer les membres d’une application, sélectionnez une vignette d’applications dans le volet **Applications**.

    Le nombre de membres associés à l’application sélectionnée est répercuté dans la vignette des membres.

    ![Sélectionner une application](./media/manage-users/blockchain-workbench-select-application.png)


#### <a name="add-member-to-application"></a>Ajouter un membre à l’application

1. Sélectionnez la vignette des membres afin d’afficher une liste des membres actuels.
1. Sélectionnez **Ajouter des membres**.

    ![La capture d’écran montre la fenêtre d’appartenance à l’application avec le bouton Ajouter un membre mis en évidence.](./media/manage-users/application-add-members.png)

1. Recherchez le nom d’utilisateur.  Seuls les utilisateurs Azure AD existants dans le client Blockchain Workbench sont répertoriés. Si l’utilisateur est introuvable, vous devez [ajouter des utilisateurs Azure AD](#add-azure-ad-users).

    ![Ajouter des membres](./media/manage-users/find-user.png)

1. Sélectionnez un **Rôle** dans la liste déroulante.

    ![Sélectionner des membres de rôle](./media/manage-users/application-select-role.png)

1. Sélectionnez **Ajouter** pour ajouter le membre avec le rôle associé à l’application.

#### <a name="remove-member-from-application"></a>Supprimer un membre de l’application

1. Sélectionnez la vignette des membres afin d’afficher une liste des membres actuels.
1. Pour l’utilisateur que vous souhaitez supprimer, sélectionnez **Supprimer**  dans la liste déroulante du rôle.

    ![Supprimer un membre](./media/manage-users/application-remove-member.png)

#### <a name="change-or-add-role"></a>Modifier ou ajouter un rôle

1. Sélectionnez la vignette des membres afin d’afficher une liste des membres actuels.
1. Pour l’utilisateur que vous souhaitez modifier, cliquez sur la liste déroulante et sélectionnez le nouveau rôle.

    ![Modifier le rôle](./media/manage-users/application-change-role.png)

## <a name="next-steps"></a>Étapes suivantes

Dans cet article sur les procédures, vous avez appris à gérer des utilisateurs pour Azure Blockchain Workbench. Pour apprendre à créer une application blockchain, passez à l’article suivant.

> [!div class="nextstepaction"]
> [Créer une application blockchain dans Azure Blockchain Workbench](create-app.md)
