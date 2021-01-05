---
title: Gagner en visibilité au niveau locataire dans Azure Security Center | Microsoft Docs
description: Cet article explique comment gérer votre position de sécurité à grande échelle en appliquant des stratégies à tous les abonnements liés à votre locataire Azure Active Directory.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: b85c0e93-9982-48ad-b23f-53b367f22b10
ms.service: security-center
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/19/2018
ms.author: memildin
ms.openlocfilehash: 7252a6ccd77212f75f5db54e5f3fcad7aa2df50a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013800"
---
# <a name="gain-tenant-wide-visibility-for-azure-security-center"></a>Gagner en visibilité au niveau locataire dans Azure Security Center
Cet article explique comment gérer la position de sécurité de votre organisation à grande échelle en appliquant des stratégies de sécurité à tous les abonnements Azure liés à votre locataire Azure Active Directory.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="management-groups"></a>Groupes d’administration
Avec les groupes d’administration Azure, vous avez la possibilité de gérer efficacement non seulement l’accès, les stratégies et les rapports de groupes d’abonnements, mais aussi l’intégralité du domaine Azure en intervenant sur le groupe d’administration racine. Chaque locataire Azure AD reçoit un seul groupe d’administration de niveau supérieur appelé groupe d’administration racine. Ce groupe d’administration racine est intégré à la hiérarchie et contient tous les groupes d’administration et abonnements. Il permet d’appliquer des stratégies globales et des attributions de rôles Azure au niveau du répertoire. 

Le groupe d’administration racine est créé automatiquement quand vous effectuez l’une des actions suivantes : 
1. Choisir d’utiliser des groupes d’administration Azure en accédant à **Groupes d’administration** dans le [portail Azure](https://portal.azure.com).
2. Créer un groupe d’administration via un appel d’API.
3. Créer un groupe d’administration avec PowerShell.

Pour une présentation détaillée des groupes d’administration, consultez l’article [Organiser vos ressources avec des groupes d’administration Azure](../governance/management-groups/overview.md).

## <a name="create-a-management-group-in-the-azure-portal"></a>Créer un groupe d’administration dans le portail Azure
Vous pouvez organiser les abonnements en groupes d’administration et y appliquer vos stratégies de gouvernance. Tous les abonnements d’un groupe d’administration héritent automatiquement des stratégies appliquées à ce groupe d’administration. Même si les groupes d’administration ne sont pas obligatoires pour intégrer Security Center, il est vivement recommandé d’en créer au moins un pour que le groupe d’administration racine soit créé. Une fois que le groupe est créé, tous les abonnements sous votre locataire Azure AD y sont liés. Pour des instructions sur PowerShell et d’autres informations, consultez [Créer des groupes d’administration pour gérer les ressources et l’organisation](../governance/management-groups/create-management-group-portal.md).

 
1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Tous les services** > **Groupes d’administration**.
3. Dans la page principale, sélectionnez **Nouveau groupe d’administration**. 

    ![Groupe principal](./media/security-center-management-groups/main.png) 
4.  Renseignez le champ ID du groupe d’administration. 
    - L’**ID du groupe d’administration** est l’identificateur unique de l’annuaire utilisé pour envoyer des commandes sur ce groupe d’administration. Cet identificateur n’est pas modifiable après sa création car il est utilisé dans tout le système Azure pour identifier ce groupe. 
    - Le champ du nom d’affichage correspond au nom qui s’affiche dans le portail Azure. Un nom d’affichage distinct est un champ facultatif lors de la création du groupe d’administration. Il peut être modifié à tout moment.  

      ![Créer](./media/security-center-management-groups/create_context_menu.png)  
5.  Sélectionnez **Enregistrer**.

### <a name="view-management-groups-in-the-azure-portal"></a>Afficher les groupes d’administration dans le portail Azure
1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Pour voir les groupes d’administration, sélectionnez **Tous les services** sous le menu principal d’Azure.
3. Sous **Général**, sélectionnez **Groupes d’administration**.

    ![Créer un groupe d’administration](./media/security-center-management-groups/all-services.png)

## <a name="grant-tenant-level-visibility-and-the-ability-to-assign-policies"></a>Donner une visibilité au niveau locataire et affecter des stratégies

Pour avoir une visibilité de la sécurité de tous les abonnements inscrits dans le locataire Azure AD, un rôle Azure doté des autorisations de lecture suffisantes doit être attribué au groupe d’administration racine.

### <a name="elevate-access-for-a-global-administrator-in-azure-active-directory"></a>Élever l’accès d’un administrateur général dans Azure Active Directory
Les administrateurs de locataires Azure Active Directory n’ont pas d’accès direct aux abonnements Azure. Toutefois, en tant qu’administrateur d’annuaire, ils ont le droit de s’élever à un rôle qui bénéficie d’un accès. Un administrateur client Azure AD doit s’élever à la fonction d’administrateur des accès utilisateur au niveau du groupe d’administration racine pour pouvoir attribuer des rôles Azure. Pour des instructions sur PowerShell et d’autres informations, consultez [Élever l’accès d’un administrateur général dans Azure Active Directory](../role-based-access-control/elevate-access-global-admin.md). 


1. Connectez-vous au [portail Azure](https://portal.azure.com) ou au [Centre d’administration Azure Active Directory](https://aad.portal.azure.com).

2. Dans la liste de navigation, cliquez sur **Azure Active Directory**, puis sur **Propriétés**.

   ![Propriétés Azure AD - capture d’écran](./media/security-center-management-groups/aad-properties.png)

3. Sous **Gestion des accès aux ressources Azure**, définissez le commutateur sur **Oui**.

   ![Gestion des accès aux ressources Azure - capture d’écran](./media/security-center-management-groups/aad-properties-global-admin-setting.png)

   - Quand vous définissez le commutateur sur Oui, le rôle Administrateur de l’accès utilisateur vous est attribué dans Azure RBAC au niveau de l’étendue racine (/). Ceci vous accorde l’autorisation d’attribuer des rôles dans tous les abonnements et groupes d’administration Azure associés à cet annuaire Azure AD. Ce commutateur est disponible seulement pour les utilisateurs auxquels le rôle Administrateur général a été attribué dans Azure AD.

   - Quand vous définissez le commutateur sur Non, le rôle Administrateur de l’accès utilisateur dans Azure RBAC est supprimé de votre compte d’utilisateur. Vous ne pouvez plus attribuer des rôles dans tous les abonnements et groupes d’administration Azure associés à cet annuaire Azure AD. Vous pouvez voir et gérer seulement les abonnements et groupes d’administration Azure auxquels l’accès vous a été accordé.

4. Cliquez sur **Enregistrer** pour enregistrer votre paramètre.

    - Ce paramètre n’est pas une propriété globale et s’applique uniquement à l’utilisateur actuellement connecté.

5. Effectuez les tâches que vous devez accomplir via un accès avec élévation de privilèges. Lorsque vous avez terminé, repositionnez le commutateur sur **Non**.


### <a name="assign-azure-roles-to-users"></a>Attribuer des rôles Azure aux utilisateurs
Pour gagner en visibilité sur tous les abonnements, les administrateurs clients doivent attribuer le rôle Azure approprié à tous les utilisateurs à qui ils souhaitent accorder une visibilité sur l’ensemble du locataire, y compris eux-mêmes, au niveau du groupe d’administration racine. Les rôles recommandés à affecter sont **Administrateur de sécurité** ou **Lecteur Sécurité**. En règle générale, le rôle Administrateur de sécurité est nécessaire pour appliquer des stratégies au niveau racine, tandis que Lecteur de Sécurité est suffisant pour fournir une visibilité au niveau locataire. Pour plus d’informations sur les autorisations accordées par ces rôles, consultez la [description du rôle intégré Administrateur de sécurité](../role-based-access-control/built-in-roles.md#security-admin) ou la [description du rôle intégré de Lecteur Sécurité](../role-based-access-control/built-in-roles.md#security-reader).


#### <a name="assign-azure-roles-to-users-through-the-azure-portal"></a>Attribuer des rôles Azure aux utilisateurs via le portail Azure : 

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
1. Pour voir les groupes d’administration, sélectionnez **Tous les services** sous le menu principal d’Azure puis sélectionnez **Groupes d'administration**.
1.  Sélectionnez un groupe d’administration et cliquez sur **Détails**.

    ![Capture d’écran Détails des groupes d'administration](./media/security-center-management-groups/management-group-details.PNG)
 
1. Cliquez sur **Contrôle d’accès (IAM)** , puis **Attributions de rôles**.

1. Cliquez sur **Ajouter une attribution de rôle**.

1. Sélectionnez le rôle à affecter et l’utilisateur, puis cliquez sur **Enregistrer**.  
   
   ![Capture d’écran de l’ajout du rôle Lecteur Sécurité](./media/security-center-management-groups/asc-security-reader.png)


#### <a name="assign-azure-roles-to-users-with-powershell"></a>Attribuer des rôles Azure aux utilisateurs à l’aide de PowerShell : 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

1. Installez [Azure PowerShell](/powershell/azure/install-az-ps).
2. Exécutez les commandes suivantes : 

    ```azurepowershell
    # Login to Azure as a Global Administrator user
    Connect-AzAccount
    ```

3. Quand vous y êtes invité, connectez-vous avec des informations d’identification d’administrateur général. 

    ![Capture d’écran de l’invite de connexion](./media/security-center-management-groups/azurerm-sign-in.PNG)

4. Accordez les autorisations du rôle de lecteur en exécutant la commande suivante :

    ```azurepowershell
    # Add Reader role to the required user on the Root Management Group
    # Replace "user@domian.com” with the user to grant access to
    New-AzRoleAssignment -SignInName "user@domain.com" -RoleDefinitionName "Reader" -Scope "/"
    ```
5. Pour supprimer le rôle, utilisez la commande suivante : 

    ```azurepowershell
    Remove-AzRoleAssignment -SignInName "user@domain.com" -RoleDefinitionName "Reader" -Scope "/" 
    ```

### <a name="open-or-refresh-security-center"></a>Ouvrir ou actualiser Security Center
Une fois que vous disposez d’un accès élevé, ouvrez ou actualisez Azure Security Center pour vérifier que vous voyez tous les abonnements sous votre locataire Azure AD. 

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
2. Veillez à sélectionner tous les abonnements dans le sélecteur d’abonnements que vous souhaitez afficher dans Security Center.

    ![Capture d’écran du sélecteur d’abonnements](./media/security-center-management-groups/subscription-selector.png)

1. Sélectionnez **Tous les services** sous le menu principal d’Azure, puis sélectionnez **Security Center**.
2. Dans la **Vue d’ensemble**, se trouve un graphique des abonnements couverts.

    ![Capture d’écran du graphique des abonnements couverts](./media/security-center-management-groups/security-center-subscription-coverage.png)

3. Cliquez sur **Couverture** pour voir la liste des abonnements couverts. 

    ![Capture d’écran de la liste des abonnements couverts](./media/security-center-management-groups/security-center-coverage.png)

### <a name="remove-elevated-access"></a>Supprimer l’accès élevé 
Une fois que les rôles Azure ont été attribués aux utilisateurs, l’administrateur client doit se supprimer du rôle d’administrateur des accès utilisateur.

1. Connectez-vous au [portail Azure](https://portal.azure.com) ou au [Centre d’administration Azure Active Directory](https://aad.portal.azure.com).

2. Dans la liste de navigation, cliquez sur **Azure Active Directory**, puis sur **Propriétés**.

3. Sous **Gestion des accès aux ressources Azure**, définissez le commutateur sur **Non**.

4. Cliquez sur **Enregistrer** pour enregistrer votre paramètre.



## <a name="adding-subscriptions-to-a-management-group"></a>Ajout d’abonnements à un groupe d’administration
Vous pouvez ajouter des abonnements au groupe d’administration que vous avez créé. Ces étapes ne sont pas obligatoires pour obtenir une visibilité au niveau locataire ni pour gérer les accès et les stratégies globales.

1. Sous **Groupes d’administration**, sélectionnez un groupe d’administration pour y ajouter votre abonnement.

    ![Sélectionner un groupe d’administration pour y ajouter un abonnement](./media/security-center-management-groups/management-group-subscriptions.png)

2. Sélectionnez **Ajouter existant**.

    ![Ajouter existant](./media/security-center-management-groups/add-existing.png)

3. Entrez un abonnement sous **Ajouter une ressource existante** et cliquez sur **Enregistrer**.

4. Répétez les étapes 1 à 3 pour tous les abonnements concernés.

   > [!NOTE]
   > Les groupes d’administration peuvent contenir à la fois des abonnements et des groupes d’administration enfants. Lorsque vous attribuez un rôle Azure à un utilisateur dans le groupe d’administration parent, l’accès est hérité par les abonnements du groupe d’administration enfant. Les stratégies définies au niveau du groupe d’administration parent sont aussi héritées par les enfants. 

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez appris à gagner en visibilité au niveau locataire dans Azure Security Center. Pour plus d’informations sur Security Center, consultez les articles suivants :

> [!div class="nextstepaction"]
> [Surveillance de l’intégrité de la sécurité dans Azure Security Center](security-center-monitoring.md)

> [!div class="nextstepaction"]
> [Gérer et répondre aux alertes de sécurité dans Azure Security Center](security-center-managing-and-responding-alerts.md)