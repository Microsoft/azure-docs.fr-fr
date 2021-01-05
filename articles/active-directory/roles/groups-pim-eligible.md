---
title: Attribuer un rôle à un groupe à l’aide de Privileged Identity Management dans Azure AD | Microsoft Docs
description: Découvrez comment attribuer un rôle Azure Active Directory (Azure AD) à un groupe à l’aide d’Azure AD Privileged Identity Management (PIM).
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: article
ms.date: 11/05/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 74ad503374e0148e9813508b6c7f8b21e2dca7a3
ms.sourcegitcommit: 0d171fe7fc0893dcc5f6202e73038a91be58da03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93379160"
---
# <a name="assign-a-role-to-a-group-using-privileged-identity-management"></a>Attribuer un rôle à un groupe à l’aide de Privileged Identity Management

Cet article décrit la procédure d’attribution d’un rôle Azure Active Directory (Azure AD) à un groupe à l’aide d’Azure AD Privileged Identity Management (PIM).

> [!NOTE]
> Vous devez utiliser une version mise à jour de Privileged Identity Management pour pouvoir attribuer au groupe un rôle Azure AD via PIM. Vous êtes peut-être sur une version antérieure de PIM si votre organisation Azure AD exploite l'API Privileged Identity Management. Le cas échéant, contactez l’alias pim_preview@microsoft.com pour déplacer votre organisation et mettre à jour votre API. Pour en savoir plus, consultez [Fonctionnalités et rôles Azure AD dans PIM](../privileged-identity-management/azure-ad-roles-features.md).

## <a name="using-azure-ad-admin-center"></a>Utilisation du Centre d’administration Azure AD

1. Connectez-vous à [Azure AD Privileged Identity Management](https://ms.portal.azure.com/?Microsoft_AAD_IAM_GroupRoles=true&Microsoft_AAD_IAM_userRolesV2=true&Microsoft_AAD_IAM_enablePimIntegration=true#blade/Microsoft_Azure_PIMCommon/CommonMenuBlade/quickStart) en tant qu’administrateur de rôle privilégié ou administrateur général dans votre organisation.

1. Sélectionnez **Privileged Identity Management** > **Rôles Azure AD**  > **Rôles** > **Ajouter des attributions**

1. Sélectionnez un rôle, puis un groupe. Seuls les groupes éligibles à l’attribution de rôle (groupes à attribution de rôle) sont affichés, et non tous les groupes.

    ![Capture d'écran représentant la page « Ajouter des attributions » dans laquelle les sections « Sélectionner un rôle » et « Sélectionner des membres » sont mises en surbrillance.](./media/groups-pim-eligible/select-member.png)

1. Sélectionnez le paramètre d’appartenance souhaité. Pour les rôles nécessitant une activation, choisissez **éligible**. Par défaut, l’utilisateur est éligible de façon permanente, mais vous pouvez également définir une heure de début et de fin de l’éligibilité de l’utilisateur. Une fois que vous avez terminé, cliquez sur Enregistrer et Ajouter pour terminer l’attribution de rôle.

    ![sélectionner l’utilisateur auquel vous attribuez le rôle](./media/groups-pim-eligible/set-assignment-settings.png)

## <a name="using-powershell"></a>Utilisation de PowerShell

### <a name="download-the-azure-ad-preview-powershell-module"></a>Télécharger le module de préversion PowerShell Azure AD

Pour installer le module PowerShell Azure AD, utilisez les cmdlets suivantes :

```powershell
install-module azureadpreview
import-module azureadpreview
```

Pour vérifier que le module est prêt à l’emploi, utilisez la cmdlet suivante :

```powershell
get-module azureadpreview
```

### <a name="assign-a-group-as-an-eligible-member-of-a-role"></a>Marquer un groupe comme membre éligible d'un rôle

```powershell
$schedule = New-Object Microsoft.Open.MSGraph.Model.AzureADMSPrivilegedSchedule
$schedule.Type = "Once"
$schedule.StartDateTime = "2019-04-26T20:49:11.770Z"
$schedule.endDateTime = "2019-07-25T20:49:11.770Z"
Open-AzureADMSPrivilegedRoleAssignmentRequest -ProviderId aadRoles -Schedule $schedule -ResourceId "[YOUR TENANT ID]" -RoleDefinitionId "9f8c1837-f885-4dfd-9a75-990f9222b21d" -SubjectId "[YOUR GROUP ID]" -AssignmentState "Eligible" -Type "AdminAdd"
```

## <a name="using-microsoft-graph-api"></a>Utilisation de l’API Microsoft Graph

```powershell
POST
https://graph.microsoft.com/beta/privilegedAccess/aadroles/roleAssignmentRequests  

{

 "roleDefinitionId": {roleDefinitionId},

 "resourceId": {tenantId},

 "subjectId": {GroupId},

 "assignmentState": "Eligible",

 "type": "AdminAdd",

 "reason": "reason string",

 "schedule": {

   "startDateTime": {DateTime},

   "endDateTime": {DateTime},

   "type": "Once"

 }

}
```

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser des groupes cloud pour gérer les attributions de rôles](groups-concept.md)
- [Résolution des problèmes des rôles attribués aux groupes cloud](groups-faq-troubleshooting.md)
- [Configurer les paramètres des rôles d’administrateur Azure AD dans Privileged Identity Management](../privileged-identity-management/pim-how-to-change-default-settings.md)
- [Attribuer des rôles de ressources Azure dans Privileged Identity Management](../privileged-identity-management/pim-resource-roles-assign-roles.md)
