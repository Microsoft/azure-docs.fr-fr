---
title: Ajouter, supprimer et répertorier des utilisateurs dans une unité administrative – Azure Active Directory | Microsoft Docs
description: Gérer les utilisateurs et leurs autorisations de rôles dans une unité administrative dans Azure Active Directory
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.subservice: roles
ms.workload: identity
ms.date: 11/04/2020
ms.author: curtand
ms.reviewer: anandy
ms.custom: oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: b490e03ea7ac0a3bc780fa731629217126b6828a
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93395527"
---
# <a name="add-and-manage-users-in-an-administrative-unit-in-azure-active-directory"></a>Ajouter et gérer des utilisateurs dans une unité administrative dans Azure Active Directory

Dans Azure Active Directory (Azure AD), vous pouvez ajouter des utilisateurs à une unité administrative pour obtenir une étendue administrative de contrôle plus précise.

Pour vous préparer à utiliser PowerShell et Microsoft Graph pour la gestion des unités administratives, consultez [Prise en main](admin-units-manage.md#get-started).

## <a name="add-users-to-an-administrative-unit"></a>Ajouter des utilisateurs à une unité administrative

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure

Vous pouvez affecter des utilisateurs à des unités administratives individuellement ou dans une opération en bloc.

- Affectez des utilisateurs individuels à partir d’un profil utilisateur :

   1. Connectez-vous au [centre d’administration Azure AD](https://portal.azure.com) avec les autorisations Administrateur de rôle privilégié.

   1. Sélectionnez **Utilisateurs** et, afin d’ouvrir le profil de l’utilisateur, sélectionnez l’utilisateur à affecter à une unité administrative.
   
   1. Sélectionnez **Unités administratives**. 
   
   1. Pour affecter l’utilisateur à une ou plusieurs unités administratives, sélectionnez **Affecter à une unité administrative** puis, dans le volet droit, sélectionnez les unités administratives auxquelles vous souhaitez affecter l’utilisateur.

       ![Capture d’écran du volet « Unités administratives » pour l’affectation d’un utilisateur à une unité administrative.](./media/admin-units-add-manage-users/assign-users-individually.png)

- Affectez des utilisateurs individuels à partir d’une unité administrative :

   1. Connectez-vous au [centre d’administration Azure AD](https://portal.azure.com) avec les autorisations Administrateur de rôle privilégié.
   1. Sélectionnez **Unités administratives** puis sélectionnez l’unité administrative à laquelle l’utilisateur doit être affecté.
   1. Sélectionnez **Tous les utilisateurs** , puis sélectionnez **Ajouter un membre** et enfin, à partir du volet **Ajouter un membre** , choisissez un ou plusieurs utilisateurs à affecter à l’unité administrative.

        ![Capture d’écran du volet « Utilisateurs » de l’unité administrative permettant d’affecter un utilisateur à une unité administrative.](./media/admin-units-add-manage-users/assign-to-admin-unit.png)

- Affectez des utilisateurs dans une opération en bloc :

   1. Connectez-vous au [centre d’administration Azure AD](https://portal.azure.com) avec les autorisations Administrateur de rôle privilégié.

   1. Sélectionnez **Unités administratives**.

   1. Sélectionnez l’unité administrative à laquelle vous souhaitez ajouter des utilisateurs.

   1. Sélectionnez **Utilisateurs** > **Activités en bloc** > **Ajouter des membres en bloc**. Vous pouvez ensuite télécharger le modèle CSV (valeurs séparées par des virgules) et modifier le fichier. Le format est simple et nécessite l’ajout d’un UPN (nom d’utilisateur principal) unique sur chaque ligne. Une fois le fichier prêt, enregistrez-le à un emplacement approprié, puis chargez-le.

      ![Capture d’écran du volet « Utilisateurs » permettant d’affecter des utilisateurs à une unité administrative en tant qu’opération en bloc.](./media/admin-units-add-manage-users/bulk-assign-to-admin-unit.png)

### <a name="use-powershell"></a>Utiliser PowerShell

Dans PowerShell, utilisez la cmdlet `Add-AzureADAdministrativeUnitMember` de l’exemple suivant pour ajouter l’utilisateur à l’unité administrative. L’ID d’objet de l’unité administrative à laquelle vous souhaitez ajouter l’utilisateur et l’ID d’objet de l’utilisateur que vous souhaitez ajouter sont utilisés en tant qu’arguments. Modifiez la section en surbrillance selon les besoins de votre environnement spécifique.

```powershell
$administrativeunitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'Test administrative unit 2'"
$UserObj = Get-AzureADUser -Filter "UserPrincipalName eq 'billjohn@fabidentity.onmicrosoft.com'"
Add-AzureADMSAdministrativeUnitMember -Id $administrativeunitObj.ObjectId -RefObjectId $UserObj.ObjectId
```


### <a name="use-microsoft-graph"></a>Utiliser Microsoft Graph

Remplacez l’espace réservé par les informations de test et exécutez la commande suivante :

```http
Http request
POST /administrativeUnits/{Admin Unit id}/members/$ref
Request body
{
  "@odata.id":"https://graph.microsoft.com/v1.0/users/{id}"
}
```

Exemple :

```http
{
  "@odata.id":"https://graph.microsoft.com/v1.0/users/johndoe@fabidentity.com"
}
```

## <a name="view-a-list-of-administrative-units-for-a-user"></a>Afficher une liste d’unités administratives pour un utilisateur

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure

Dans le portail Azure, vous pouvez ouvrir le profil d’un utilisateur en procédant comme suit :

1. Accédez à **Azure AD** , puis sélectionnez **Utilisateurs**.

1. Sélectionnez l’utilisateur dont vous souhaitez afficher le profil.

1. Sélectionnez **Unités administratives** pour afficher la liste des unités administratives auxquelles l’utilisateur a été affecté.

   ![Capture d’écran des unités administratives auxquelles un utilisateur a été affecté.](./media/admin-units-add-manage-users/list-user-admin-units.png)

### <a name="use-powershell"></a>Utiliser PowerShell

Exécutez la commande suivante :

```powershell
Get-AzureADMSAdministrativeUnit | where { Get-AzureADMSAdministrativeUnitMember -Id $_.ObjectId | where {$_.RefObjectId -eq $userObjId} }
```
> [!NOTE]
> Par défaut, `Get-AzureADAdministrativeUnitMember` retourne uniquement 100 membres d’une unité administrative. Pour récupérer d’autres membres, vous pouvez ajouter `"-All $true"`.

### <a name="use-microsoft-graph"></a>Utiliser Microsoft Graph

Remplacez l’espace réservé par les informations de test et exécutez la commande suivante :

```http
https://graph.microsoft.com/v1.0/users/{id}/memberOf/$/Microsoft.Graph.AdministrativeUnit
```

## <a name="remove-a-single-user-from-an-administrative-unit"></a>Supprimer un seul utilisateur d’une unité administrative

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure

Vous pouvez supprimer un utilisateur d’une unité administrative de l’une des deux manières suivantes : 

* Dans le portail Azure, accédez à **Azure AD** , puis sélectionnez **Utilisateurs**. 
  1. Sélectionnez l’utilisateur pour ouvrir son profil. 
  1. Sélectionnez l’unité administrative dont vous souhaitez supprimer l’utilisateur, puis sélectionnez **Supprimer de l’unité administrative**.

     ![Capture d’écran montrant comment supprimer un utilisateur d’une unité administrative à partir du volet Profil de l’utilisateur.](./media/admin-units-add-manage-users/user-remove-admin-units.png)

* Dans le portail Azure, accédez à **Azure AD** puis, sélectionnez **Unités administratives**.
  1. Sélectionnez l’unité administrative dont vous souhaitez supprimer l’utilisateur. 
  1. Sélectionnez l’utilisateur, puis choisissez **Supprimer un membre**.
  
     ![Capture d’écran montrant comment supprimer un utilisateur au niveau de l’unité administrative.](./media/admin-units-add-manage-users/admin-units-remove-user.png)

### <a name="use-powershell"></a>Utiliser PowerShell

Exécutez la commande suivante :

```powershell
Remove-AzureADMSAdministrativeUnitMember -Id $auId -MemberId $memberUserObjId
```

### <a name="use-microsoft-graph"></a>Utiliser Microsoft Graph

Remplacez les espaces réservés par les informations de test et exécutez la commande suivante :

`https://graph.microsoft.com/v1.0/directory/administrativeUnits/{adminunit-id}/members/{user-id}/$ref`

## <a name="remove-multiple-users-as-a-bulk-operation"></a>Supprimer plusieurs utilisateurs dans une opération en bloc

Pour supprimer plusieurs utilisateurs d’une unité administrative, procédez comme suit :

1. Dans le portail Azure, accédez à **Azure AD**.

1. Sélectionnez **Unités administratives** , puis sélectionnez l’unité administrative dont vous souhaitez supprimer des utilisateurs. 

1. Sélectionnez **Supprimer des membres en bloc** , puis téléchargez le modèle CSV que vous allez utiliser pour répertorier les utilisateurs que vous souhaitez supprimer.

   ![Capture d’écran montrant le lien « Supprimer des membres en bloc » dans le volet « Utilisateurs ».](./media/admin-units-add-manage-users/bulk-user-remove.png)

1. Modifiez le modèle CSV téléchargé avec les entrées utilisateur appropriées. Ne supprimez pas les deux premières lignes du modèle. Ajoutez un nom d’utilisateur principal (UPN) dans chaque ligne.

   ![Capture d’écran d’un fichier CSV modifié pour la suppression en bloc d’utilisateurs d’une unité administrative.](./media/admin-units-add-manage-users/bulk-user-entries.png)

1. Enregistrez vos modifications, chargez le fichier, puis sélectionnez **Envoyer**.

## <a name="next-steps"></a>Étapes suivantes

- [Affecter un rôle à une unité administrative](admin-units-assign-roles.md)
- [Ajouter des groupes à une unité administrative](admin-units-add-manage-groups.md)
