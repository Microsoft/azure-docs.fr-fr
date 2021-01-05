---
title: Ajouter, supprimer et répertorier des groupes dans une unité administrative – Azure Active Directory | Microsoft Docs
description: Gérez les groupes et leurs autorisations de rôles dans une unité administrative dans Azure Active Directory.
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
ms.openlocfilehash: 092604429cf5a527d7ee62b412e879ad9991eace
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93394745"
---
# <a name="add-and-manage-groups-in-an-administrative-unit-in-azure-active-directory"></a>Ajouter et gérer des groupes dans une unité administrative dans Azure Active Directory

Dans Azure Active Directory (Azure AD), vous pouvez ajouter des groupes à une unité administrative pour obtenir une étendue administrative de contrôle plus précise.

Pour vous préparer à utiliser PowerShell et Microsoft Graph pour la gestion des unités administratives, consultez [Prise en main](admin-units-manage.md#get-started).

## <a name="add-groups-to-an-administrative-unit"></a>Ajouter des groupes à une unité administrative

Vous pouvez ajouter des groupes à une unité administrative à l’aide du portail Azure, de PowerShell ou de Microsoft Graph.

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure

Vous pouvez uniquement affecter des groupes individuels à une unité administrative. Il n’existe aucune option pour assigner des groupes dans une opération en bloc. Dans le portail Azure, vous pouvez affecter un groupe à une unité administrative de deux manières :

* Dans le volet **Groupes**  :

  1. Dans le portail Azure, accédez à **Azure AD**.
  1. Sélectionnez **Groupes** , puis sélectionnez le groupe que vous souhaitez affecter à l’unité administrative. 
  1. Dans le volet gauche, sélectionnez **Unités administratives** pour afficher la liste des unités administratives auxquelles le groupe est affecté. 

     ![Capture d’écran du lien « Affecter à une unité administrative » dans le volet « Unités administratives ».](./media/admin-units-add-manage-groups/assign-to-group-1.png)

  1. Sélectionnez **Affecter à l’unité administrative**.
  1. Dans le volet droit, sélectionnez l’unité administrative.

* À partir du volet **Unités administratives** > **Tous les groupes**  :

  1. Dans le portail Azure, accédez à **Azure AD**.
  
  1. Dans le volet gauche, sélectionnez **Unités administratives** , puis sélectionnez **Tous les groupes**. 
     Tous les groupes qui sont déjà affectés à l’unité administrative sont affichés dans le volet droit. 

  1. Dans le volet **Groupes** , sélectionnez **Ajouter**.
    Le volet droit répertorie tous les groupes disponibles dans votre organisation Azure AD. 

     ![Capture d’écran du bouton « Ajouter » pour ajouter un groupe à une unité administrative.](./media/admin-units-add-manage-groups/assign-to-admin-unit.png)

  1. Sélectionnez un ou plusieurs groupes à affecter à l’unité administrative, puis sélectionnez le bouton **Sélectionner**.

### <a name="use-powershell"></a>Utiliser PowerShell

Dans l’exemple suivant, utilisez la cmdlet `Add-AzureADMSAdministrativeUnitMember` pour ajouter le groupe à l’unité administrative. L’ID d’objet de l’unité administrative et l’ID d’objet du groupe à ajouter sont utilisés en tant qu’arguments. Modifiez la section en surbrillance selon les besoins de votre environnement spécifique.


```powershell
$administrative unitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'Test administrative unit 2'"
$GroupObj = Get-AzureADGroup -Filter "displayname eq 'TestGroup'"
Add-AzureADMSAdministrativeUnitMember -ObjectId $administrative unitObj.ObjectId -RefObjectId $GroupObj.ObjectId
```

### <a name="use-microsoft-graph"></a>Utiliser Microsoft Graph

Exécutez les commandes suivantes :

```http
Http request
POST /administrativeUnits/{Admin Unit id}/members/$ref

Request body
{
"@odata.id":"https://graph.microsoft.com/v1.0/groups/{id}"
}
```

Exemple :

```http
{
"@odata.id":"https://graph.microsoft.com/v1.0/groups/ 871d21ab-6b4e-4d56-b257-ba27827628f3"
}
```

## <a name="view-a-list-of-groups-in-an-administrative-unit"></a>Afficher une liste de groupes dans une unité administrative

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure

1. Dans le portail Azure, accédez à **Azure AD**.

1. Dans le volet gauche, sélectionnez **Unités administratives** , puis sélectionnez l’unité administrative dont vous souhaitez afficher les groupes. Par défaut, **Tous les utilisateurs** est sélectionné dans le volet gauche. 

1. Dans le volet gauche, sélectionnez **Groupes**. Le volet droit affiche la liste des groupes qui sont membres de l’unité administrative sélectionnée.

   ![Capture d’écran du volet « Groupes » affichant une liste de groupes dans une unité administrative.](./media/admin-units-add-manage-groups/list-groups-in-admin-units.png)

### <a name="use-powershell"></a>Utiliser PowerShell

Pour afficher la liste de tous les membres de l’unité administrative, exécutez la commande suivante : 

```powershell
$administrative unitObj = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'Test administrative unit 2'"
Get-AzureADMSAdministrativeUnitMember -ObjectId $administrative unitObj.ObjectId
```

Pour afficher tous les groupes qui sont membres de l’unité administrative, utilisez l’extrait de code suivant :

```http
foreach ($member in (Get-AzureADMSAdministrativeUnitMember -ObjectId $administrative unitObj.ObjectId)) 
{
if($member.ObjectType -eq "Group")
{
Get-AzureADGroup -ObjectId $member.ObjectId
}
}
```

### <a name="use-microsoft-graph"></a>Utiliser Microsoft Graph

Exécutez la commande suivante :

```http
HTTP request
GET /directory/administrativeUnits/{Admin id}/members/$/microsoft.graph.group
Request body
{}
```

## <a name="view-a-list-of-administrative-units-for-a-group"></a>Afficher une liste d’unités administratives pour un groupe

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure

1. Dans le portail Azure, accédez à **Azure AD**.

1. Dans le volet gauche, sélectionnez **Groupes** pour afficher une liste de groupes.

1. Sélectionnez un groupe pour ouvrir son profil. 

1. Dans le volet gauche, sélectionnez **Unités administratives** pour répertorier toutes les unités administratives dont le groupe est membre.

   ![Capture d’écran du volet « Unités administratives », affichant une liste des unités administratives auxquelles un groupe est affecté.](./media/admin-units-add-manage-groups/list-group-au.png)

### <a name="use-powershell"></a>Utiliser PowerShell

Exécutez la commande suivante :

```powershell
Get-AzureADMSAdministrativeUnit | where { Get-AzureADMSAdministrativeUnitMember -ObjectId $_.ObjectId | where {$_.ObjectId -eq $groupObjId} }
```

### <a name="use-microsoft-graph"></a>Utiliser Microsoft Graph

Exécutez la commande suivante :

```http
https://graph.microsoft.com/v1.0/groups/<group-id>/memberOf/$/Microsoft.Graph.AdministrativeUnit
```

## <a name="remove-a-group-from-an-administrative-unit"></a>Supprimer un groupe d’une unité administrative

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure

Dans le portail Azure, vous pouvez supprimer un groupe d’une unité administrative de deux manières :

- Supprimez-le à partir d’une vue d’ensemble du groupe :

  1. Dans le portail Azure, accédez à **Azure AD**.
  1. Dans le volet gauche, sélectionnez **Groupes** , puis ouvrez le profil du groupe que vous souhaitez supprimer d’une unité administrative.
  1. Dans le volet gauche, sélectionnez **Unités administratives** pour répertorier toutes les unités administratives auxquelles le groupe est affecté. 
  1. Sélectionnez l’unité administrative dont vous souhaitez supprimer le groupe, puis sélectionnez **Supprimer de l’unité administrative**.

     ![Capture d’écran du volet « Unités administratives » affichant la liste des groupes affectés à l’unité administrative sélectionnée.](./media/admin-units-add-manage-groups/group-au-remove.png)

- Supprimez-le à partir d’une unité administrative :

  1. Dans le portail Azure, accédez à **Azure AD**.
  1. Dans le volet gauche, sélectionnez **Unités administratives** , puis sélectionnez l’unité administrative à laquelle le groupe est affecté.
  1. Dans le volet gauche, sélectionnez **Groupes** pour répertorier tous les groupes qui sont affectés à l’unité administrative.
  1. Sélectionnez le groupe que vous souhaitez supprimer, puis choisissez **Supprimer les groupes**.

    ![Capture d’écran du volet « Groupes » affichant une liste de groupes dans une unité administrative.](./media/admin-units-add-manage-groups/list-groups-in-admin-units.png)

### <a name="use-powershell"></a>Utiliser PowerShell

Exécutez la commande suivante :

```powershell
Remove-AzureADMSAdministrativeUnitMember -ObjectId $auId -MemberId $memberGroupObjId
```

### <a name="use-microsoft-graph"></a>Utiliser Microsoft Graph

Exécutez la commande suivante :

```http
https://graph.microsoft.com/v1.0/directory/AdministrativeUnits/<adminunit-id>/members/<group-id>/$ref
```

## <a name="next-steps"></a>Étapes suivantes

- [Affecter un rôle à une unité administrative](admin-units-assign-roles.md)
- [Gérer les utilisateurs dans une unité administrative](admin-units-add-manage-users.md)
