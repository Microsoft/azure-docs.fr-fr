---
title: Ajouter et supprimer des unités administratives – Azure Active Directory | Microsoft Docs
description: Utiliser des unités administratives pour limiter l’étendue des autorisations de rôle dans Azure Active Directory.
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
ms.openlocfilehash: bbe76d21d948525421790f574830da7b3a163216
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93395391"
---
# <a name="manage-administrative-units-in-azure-active-directory"></a>Gérer des unités administratives dans Azure Active Directory

Pour un contrôle administratif plus précis dans Azure Active Directory (Azure AD), vous pouvez assigner des utilisateurs à un rôle Azure AD dont l’étendue est limitée à une ou plusieurs unités administratives.

## <a name="get-started"></a>Bien démarrer

1. Pour exécuter des requêtes à partir des instructions suivantes via l'[Afficheur Graph](https://aka.ms/ge), procédez comme suit :

    a. Dans le portail Azure, allez à Azure AD. 
    
    b. Dans la liste des applications, sélectionnez **Afficheur Graph**.
    
    c. Dans le volet **Autorisations** , sélectionnez **Accorder un consentement administrateur pour Afficheur Graph**.

    ![Capture d’écran montrant le lien « Accorder un consentement administrateur pour Afficheur Graph ».](./media/admin-units-manage/select-graph-explorer.png)


1. Utilisez la préversion d'Azure AD PowerShell.

## <a name="add-an-administrative-unit"></a>Ajouter une unité administrative

Vous pouvez ajouter une unité administrative à l’aide du portail Azure ou de PowerShell.

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure

1. Dans le portail Azure, allez à Azure AD. Ensuite, dans le volet gauche, sélectionnez **Unités administratives**.

    ![Capture d’écran du lien « Unités administratives » dans Azure AD.](./media/admin-units-manage/nav-to-admin-units.png)

1. Sélectionnez le bouton **Ajouter** dans la partie supérieure du volet, puis, dans la zone **Nom** , entrez le nom de l’unité administrative. Si vous le souhaitez, ajoutez une description de l’unité administrative.

    ![Capture d’écran montrant le bouton Ajouter et la zone Nom permettant d’entrer le nom de l’unité administrative.](./media/admin-units-manage/add-new-admin-unit.png)

1. Sélectionnez le bouton bleu **Ajouter** pour finaliser l’unité administrative.

### <a name="use-powershell"></a>Utiliser PowerShell

Installez Azure AD PowerShell (préversion) avant d’essayer d’exécuter les commandes suivantes :

```powershell
Connect-AzureAD
New-AzureADMSAdministrativeUnit -Description "West Coast region" -DisplayName "West Coast"
```

Vous pouvez modifier les valeurs entre guillemets si nécessaire.

### <a name="use-microsoft-graph"></a>Utiliser Microsoft Graph

```http
Http Request
POST /administrativeUnits
Request body
{
  "displayName": "North America Operations",
  "description": "North America Operations administration"
}
```

## <a name="remove-an-administrative-unit"></a>Supprimer une unité administrative

Dans Azure AD, vous pouvez supprimer une unité administrative dont vous n’avez plus besoin comme une unité d'étendue pour les rôles d'administration.

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure

1. Dans le portail Azure, accédez à **Azure AD** puis, sélectionnez **Unités administratives**. 
1. Sélectionnez l’unité administrative à supprimer, puis **Supprimer**. 
1. Pour confirmer que vous souhaitez supprimer l’unité administrative, sélectionnez **Oui**. L’unité administrative est supprimée.

![Capture d’écran du bouton Supprimer de l’unité administrative et de la fenêtre de confirmation.](./media/admin-units-manage/select-admin-unit-to-delete.png)

### <a name="use-powershell"></a>Utiliser PowerShell

```powershell
$delau = Get-AzureADMSAdministrativeUnit -Filter "displayname eq 'DeleteMe Admin Unit'"
Remove-AzureADMSAdministrativeUnit -ObjectId $delau.ObjectId
```

Vous pouvez modifier les valeurs entre guillemets si nécessaire, en fonction de l’environnement spécifique.

### <a name="use-the-graph-api"></a>Utiliser l’API Graph

```http
HTTP request
DELETE /administrativeUnits/{Admin id}
Request body
{}
```

## <a name="next-steps"></a>Étapes suivantes

* [Gérer les utilisateurs dans une unité administrative](admin-units-add-manage-users.md)
* [Gérer les groupes dans une unité administrative](admin-units-add-manage-groups.md)
