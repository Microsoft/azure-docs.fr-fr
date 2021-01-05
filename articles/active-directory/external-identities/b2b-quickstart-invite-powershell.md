---
title: 'Démarrage rapide : Ajouter un utilisateur invité avec PowerShell - Azure AD'
description: Dans ce guide de démarrage rapide, vous allez apprendre à utiliser PowerShell pour envoyer une invitation à un utilisateur externe d’Azure AD B2B Collaboration.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: b0faaf0394bddc2e443afc194bbd0ecef72625f9
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "87910511"
---
# <a name="quickstart-add-a-guest-user-with-powershell"></a>Démarrage rapide : Ajouter un utilisateur invité avec PowerShell

Avec Azure Active Directory B2B Collaboration, vous disposez de plusieurs méthodes pour inviter des partenaires externes à utiliser vos applications et vos services. Dans le guide de démarrage rapide précédent, vous avez vu comment ajouter des utilisateurs invités directement dans le portail d’administration Azure Active Directory. Vous pouvez également utiliser PowerShell pour ajouter des utilisateurs invités, un par un, ou en bloc. Dans ce guide de démarrage rapide, vous allez utiliser la commande New-AzureADMSInvitation pour ajouter un utilisateur invité à votre locataire Azure.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer. 

## <a name="prerequisites"></a>Prérequis

### <a name="install-the-latest-azureadpreview-module"></a>Installez la dernière version du module AzureADPreview.
Vérifiez que vous installez la dernière version du module Azure AD PowerShell for Graph (AzureADPreview). 

Vérifiez d’abord les modules que vous avez installés. Ouvrez Windows PowerShell en tant qu’utilisateur avec privilèges élevés (Exécuter en tant qu’administrateur) et exécutez la commande suivante :
 
```powershell  
Get-Module -ListAvailable AzureAD*
```

Si le module AzureADPreview s’affiche sans un message indiquant qu’une version plus récente existe, tout est prêt. Sinon, en fonction de la sortie, effectuez l’une des actions suivantes :

- Si aucun résultat n’est retourné, exécutez la commande suivante pour installer le module AzureADPreview :
  
   ```powershell  
   Install-Module AzureADPreview
   ```
- Si seul le module AzureAD apparaît dans les résultats, exécutez les commandes suivantes pour installer le module AzureADPreview : 

   ```powershell 
   Uninstall-Module AzureAD 
   Install-Module AzureADPreview 
   ```
- Si seul le module AzureADPreview apparaît dans les résultats, mais que vous recevez un message indiquant qu’il existe une version plus récente, exécutez les commandes suivantes pour mettre à jour le module : 

   ```powershell 
   Uninstall-Module AzureADPreview 
   Install-Module AzureADPreview 
  ```

Vous pouvez recevoir une invite indiquant que vous installez le module à partir d’un dépôt non approuvé. Ce cas se produit si vous n’aviez pas configuré le dépôt PSGallery comme dépôt fiable. Appuyez sur **O** pour installer le module.

### <a name="get-a-test-email-account"></a>Obtenir un compte e-mail de test

Vous avez besoin d’un compte e-mail de test auquel vous pouvez envoyer l’invitation. Le compte doit être externe à votre organisation. Vous pouvez utiliser n’importe quel type de compte e-mail, y compris un compte de réseaux sociaux, comme gmail.com ou outlook.com.

## <a name="sign-in-to-your-tenant"></a>Connectez-vous à votre locataire

Exécutez la commande suivante pour vous connecter au domaine du locataire :

```powershell
Connect-AzureAD -TenantDomain "<Tenant_Domain_Name>"
```
Par exemple : `Connect-AzureAD -TenantDomain "contoso.onmicrosoft.com"`.

À l’invite, entrez vos informations d’identification.

## <a name="send-an-invitation"></a>Envoyer une invitation

1. Pour envoyer une invitation à votre compte e-mail de test, exécutez la commande PowerShell suivante (remplacez **"Sanda"** et **sanda\@fabrikam.com** par le nom de votre compte e-mail de test et par votre adresse e-mail) : 

   ```powershell
   New-AzureADMSInvitation -InvitedUserDisplayName "Sanda" -InvitedUserEmailAddress sanda@fabrikam.com -InviteRedirectURL https://myapps.microsoft.com -SendInvitationMessage $true
   ```
2. La commande envoie une invitation à l’adresse e-mail spécifiée. Regardez la sortie, qui doit être semblable à l’exemple qui suit :

   ![Sortie de PowerShell montrant l’acceptation d’un utilisateur en attente](media/quickstart-invite-powershell/powershell-azureadmsinvitation-result.png)

## <a name="verify-the-user-exists-in-the-directory"></a>Vérifier que l’utilisateur se trouve dans l’annuaire

1. Pour vérifier que l’utilisateur invité a bien été ajouté à Azure AD, exécutez la commande suivante :
 
   ```powershell
   Get-AzureADUser -Filter "UserType eq 'Guest'"
   ```
3. Examinez la sortie pour voir si l’utilisateur que vous avez invité y figure, avec un nom d’utilisateur principal (UPN) au format suivant : *adresse_e-mail*#EXT#\@*domaine*. Par exemple, *sanda_fabrikam.com#EXT#\@contoso.onmicrosoft.com*, où contoso.onmicrosoft.com est l’organisation à partir de laquelle vous avez envoyé les invitations.

   ![Sortie PowerShell montrant l’ajout d’un utilisateur invité](media/quickstart-invite-powershell/powershell-guest-user-added.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez supprimer le compte d’utilisateur de test de l’annuaire. Exécutez la commande suivante pour supprimer un compte d’utilisateur :

```powershell
 Remove-AzureADUser -ObjectId "<UPN>"
```
Par exemple : `Remove-AzureADUser -ObjectId "sanda_fabrikam.com#EXT#@contoso.onmicrosoft.com"`


## <a name="next-steps"></a>Étapes suivantes
Dans ce guide de démarrage rapide, vous avez invité, puis ajouté un utilisateur invité à votre annuaire à l’aide de PowerShell. Ensuite, vous avez vu comment inviter des utilisateurs en bloc à l’aide de PowerShell.

> [!div class="nextstepaction"]
> [Tutoriel : Inviter en bloc des utilisateurs Azure AD B2B Collaboration](tutorial-bulk-invite.md)
