---
title: Autoriser ou bloquer des invitations à des organisations spécifiques - Azure AD
description: Indique comment un administrateur peut utiliser le Portail Azure ou PowerShell pour définir une liste verte ou d’exclusion pour autoriser ou bloquer des utilisateurs B2B à partir de certains domaines.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 07/15/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: sasubram
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: b87650f364f8ccfd3a531d710bfbdc4715f0ac5a
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92442182"
---
# <a name="allow-or-block-invitations-to-b2b-users-from-specific-organizations"></a>Autoriser ou bloquer des invitations aux utilisateurs B2B à partir d’organisations spécifiques

Vous pouvez utiliser une liste verte ou d’exclusion pour autoriser ou bloquer des invitations aux utilisateurs B2B à partir d’organisations spécifiques. Par exemple, si vous souhaitez bloquer des domaines d’adresse e-mail personnels, vous pouvez configurer une liste d’exclusion qui contient des domaines, tels que Gmail.com et Outlook.com. Ou, si votre entreprise dispose d’un partenariat avec d’autres entreprises comme Contoso.com, Fabrikam.com et Litware.com et que vous souhaitez restreindre les invitations à ces seules organisations, vous pouvez ajouter Contoso.com, Fabrikam.com et Litware.com à votre liste verte.
  
## <a name="important-considerations"></a>Points importants à prendre en compte

- Vous pouvez créer une liste verte ou une liste d’exclusion. Vous ne pouvez pas configurer les deux types de listes. Par défaut, les domaines qui ne sont pas dans la liste verte sont dans la liste d’exclusion et vice versa. 
- Vous ne pouvez créer qu’une seule stratégie par organisation. Vous pouvez mettre à jour la stratégie pour inclure plusieurs domaines ou vous pouvez supprimer la stratégie pour en créer une nouvelle. 
- Le nombre de domaines que vous pouvez ajouter à une liste verte ou à une liste d’exclusion n’est limité que par la taille de la stratégie. La taille maximale de la stratégie entière est de 25 Ko (25 000 caractères). Elle comprend la liste verte ou la liste d’exclusion et tous les autres paramètres configurés pour d’autres fonctionnalités.
- Cette liste fonctionne indépendamment à partir des listes vertes/rouges OneDrive Entreprise et SharePoint Online. Si vous souhaitez restreindre le partage des fichiers individuels dans SharePoint Online, vous devez configurer une liste verte ou d’exclusion pour OneDrive Entreprise et SharePoint Online. Pour plus d’informations, consultez [Partage des domaines restreints dans SharePoint Online et OneDrive Entreprise](https://support.office.com/article/restricted-domains-sharing-in-sharepoint-online-and-onedrive-for-business-5d7589cd-0997-4a00-a2ba-2320ec49c4e9).
- La liste ne s’applique pas aux utilisateurs externes qui ont déjà utilisé l’invitation. La liste est appliquée une fois configurée. Si une invitation de l’utilisateur est en attente et que vous définissez une stratégie qui bloque leur domaine, la tentative de l’utilisateur d’utiliser l’invitation échoue.

## <a name="set-the-allow-or-deny-list-policy-in-the-portal"></a>Définir la stratégie des listes vertes ou d’exclusion dans le portail

Par défaut, le paramètre **Autoriser l’envoi des invitations à un domaine (plus inclusif)** est activé. Dans ce cas, vous pouvez inviter des utilisateurs B2B à partir de toute organisation.

### <a name="add-a-deny-list"></a>Ajouter une liste d’exclusion

Il s’agit du scénario le plus courant, dans lequel votre organisation souhaite travailler avec presque toutes les organisations, mais souhaite empêcher invitation des utilisateurs de domaines spécifiques en tant qu’utilisateurs B2B.

Pour ajouter une liste d’exclusion :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Azure Active Directory** > **Utilisateurs** > **Paramètres d’utilisateur** .
3. Sous **Utilisateurs externes** , sélectionnez **Gérer les paramètres de collaboration externe** .
4. Sous **Restrictions de la collaboration** , sélectionnez **Refuser les invitations aux domaines spécifiés** .
5. Sous **DOMAINES CIBLES** , entrez le nom de l’un des domaines que vous souhaitez bloquer. Pour plusieurs domaines, entrez chaque domaine sur une nouvelle ligne. Par exemple :

   ![Affiche l’option d’exclusion avec les domaines ajoutés](./media/allow-deny-list/DenyListSettings.png)
 
6. Quand vous avez terminé, cliquez sur **Enregistrer** .

Après avoir défini la stratégie, si vous essayez d’inviter un utilisateur à partir d’un domaine bloqué, vous recevez un message indiquant que le domaine de l’utilisateur est actuellement bloqué par votre stratégie d’invitation.
 
### <a name="add-an-allow-list"></a>Ajouter une liste verte

Il s’agit d’une configuration plus restrictive, dans laquelle vous pouvez définir des domaines spécifiques dans la liste verte et restreindre les invitations à d’autres organisations ou domaines qui ne sont pas mentionnés. 

Si vous souhaitez utiliser une liste verte, veillez à consacrer du temps à évaluer entièrement les besoins de votre entreprise. Si vous rendez cette stratégie trop restrictive, vos utilisateurs peuvent choisir d’envoyer des documents par e-mail ou trouver d’autres modes de collaboration approuvés non informatiques.


Pour ajouter un élément à une liste verte :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Azure Active Directory** > **Utilisateurs** > **Paramètres d’utilisateur** .
3. Sous **Utilisateurs externes** , sélectionnez **Gérer les paramètres de collaboration externe** .
4. Sous **Restrictions de la collaboration** , sélectionnez **Allow invitations only to the specified domains (most restrictive)** (Autoriser les invitations uniquement vers les domaines spécifiés (le plus restrictif)).
5. Sous **DOMAINES CIBLES** , entrez le nom de l’un des domaines que vous souhaitez autoriser. Pour plusieurs domaines, entrez chaque domaine sur une nouvelle ligne. Par exemple :

   ![Affiche l’option d’autorisation avec les domaines ajoutés](./media/allow-deny-list/AllowListSettings.png)
 
6. Quand vous avez terminé, cliquez sur **Enregistrer** .

Après avoir défini la stratégie, si vous essayez d’inviter un utilisateur à partir d’un domaine ne figurant pas sur la liste verte, vous recevez un message indiquant que le domaine de l’utilisateur est actuellement bloqué par votre stratégie d’invitation.

### <a name="switch-from-allow-to-deny-list-and-vice-versa"></a>Passer de la liste verte à la liste d’exclusion et vice versa 

Si vous passez d’une stratégie à l’autre, la configuration de la stratégie existante est alors abandonnée. Veillez à sauvegarder les détails de votre configuration avant d’effectuer le changement. 

## <a name="set-the-allow-or-deny-list-policy-using-powershell"></a>Définir la stratégie des listes vertes ou d’exclusion à l’aide de PowerShell

### <a name="prerequisite"></a>Configuration requise

> [!Note]
> Le module AzureADPreview n’est pas un module entièrement pris en charge, car il s’agit d’une version préliminaire. 

Pour définir les listes vertes ou d’exclusion à l’aide de PowerShell, vous devez installer la préversion du Module Azure Active Directory pour Windows PowerShell. Plus précisément, installez le module AzureADPreview version 2.0.0.98 ou version ultérieure.

Pour vérifier la version du module (et vérifier qu’elle est installée) :
 
1. Ouvrez Windows PowerShell en tant qu’utilisateur avec des privilèges élevés (exécuter en tant qu’Administrateur). 
2. Exécutez la commande suivante pour vérifier si des versions du Module Azure Active Directory pour Windows PowerShell sont installées sur votre ordinateur :

   ```powershell  
   Get-Module -ListAvailable AzureAD*
   ```

Si le module n’est pas installé ou si vous n’avez pas de version requise, effectuez l’une des opérations suivantes :

- Si aucun résultat n’est retourné, exécutez la commande suivante pour installer la dernière version du module AzureADPreview :
  
   ```powershell  
   Install-Module AzureADPreview
   ```
- Si seul le module AzureAD est affiché dans les résultats, exécutez les commandes suivantes pour installer le module AzureADPreview : 

   ```powershell 
   Uninstall-Module AzureAD 
   Install-Module AzureADPreview 
   ```
- Si seul le module AzureADPreview s’affiche dans les résultats, mais que la version est antérieure à 2.0.0.98, exécutez les commandes suivantes pour le mettre à jour : 

   ```powershell 
   Uninstall-Module AzureADPreview 
   Install-Module AzureADPreview 
   ```

- Si les deux modules AzureAD et AzureADPreview s’affichent dans les résultats, mais que la version du module AzureADPreview est antérieure à 2.0.0.98, exécutez les commandes suivantes pour le mettre à jour : 

   ```powershell 
   Uninstall-Module AzureAD 
   Uninstall-Module AzureADPreview 
   Install-Module AzureADPreview 
    ```

### <a name="use-the-azureadpolicy-cmdlets-to-configure-the-policy"></a>Utiliser les cmdlets AzureADPolicy pour configurer la stratégie

Pour créer une liste verte ou d’exclusion, utilisez la cmdlet [New-AzureADPolicy](/powershell/module/azuread/new-azureadpolicy?view=azureadps-2.0-preview). L’exemple suivant montre comment définir une liste d’exclusion qui bloque le domaine « live.com ».

```powershell 
$policyValue = @("{`"B2BManagementPolicy`":{`"InvitationsAllowedAndBlockedDomainsPolicy`":{`"AllowedDomains`": [],`"BlockedDomains`": [`"live.com`"]}}}")

New-AzureADPolicy -Definition $policyValue -DisplayName B2BManagementPolicy -Type B2BManagementPolicy -IsOrganizationDefault $true 
```

L’exemple suivant est identique, mais avec la définition de la stratégie incluse.

```powershell  
New-AzureADPolicy -Definition @("{`"B2BManagementPolicy`":{`"InvitationsAllowedAndBlockedDomainsPolicy`":{`"AllowedDomains`": [],`"BlockedDomains`": [`"live.com`"]}}}") -DisplayName B2BManagementPolicy -Type B2BManagementPolicy -IsOrganizationDefault $true 
```

Pour définir la stratégie des listes vertes ou d’exclusion, utilisez la cmdlet [Set-AzureADPolicy](/powershell/module/azuread/set-azureadpolicy?view=azureadps-2.0-preview). Par exemple :

```powershell   
Set-AzureADPolicy -Definition $policyValue -Id $currentpolicy.Id 
```

Pour obtenir la stratégie, utilisez la cmdlet [Get-AzureADPolicy](/powershell/module/azuread/get-azureadpolicy?view=azureadps-2.0-preview). Par exemple :

```powershell
$currentpolicy = Get-AzureADPolicy | ?{$_.Type -eq 'B2BManagementPolicy'} | select -First 1 
```

Pour supprimer la stratégie, utilisez la cmdlet [Remove-AzureADPolicy](/powershell/module/azuread/remove-azureadpolicy?view=azureadps-2.0-preview). Par exemple :

```powershell
Remove-AzureADPolicy -Id $currentpolicy.Id 
```

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir une vue d’ensemble d’Azure AD B2B , consultez [Qu’est-ce que la collaboration Azure AD B2B ?](what-is-b2b.md)
- Pour en savoir plus sur l’accès conditionnel et B2B Collaboration, voir [Accès conditionnel pour les utilisateurs de B2B Collaboration](conditional-access.md).