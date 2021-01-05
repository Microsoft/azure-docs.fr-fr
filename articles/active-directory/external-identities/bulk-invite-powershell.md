---
title: 'Tutoriel : Inviter en bloc des utilisateurs B2B Collaboration - Azure Active Directory | Microsoft Docs'
description: Dans ce tutoriel, vous découvrez comment utiliser PowerShell et un fichier CSV pour envoyer des invitations en bloc à des utilisateurs invités Azure AD B2B Collaboration externes.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: tutorial
ms.date: 02/11/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5e047f11cc243ab1a36a8c61dd1b229d9e115115
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92892483"
---
# <a name="tutorial-use-powershell-to-bulk-invite-azure-ad-b2b-collaboration-users"></a>Tutoriel : Utiliser PowerShell pour inviter en bloc des utilisateurs Azure AD B2B Collaboration

Si vous utilisez Azure Active Directory (Azure AD) B2B Collaboration pour travailler avec des partenaires externes, vous pouvez inviter en même temps plusieurs utilisateurs invités pour votre organisation. Dans ce tutoriel, vous découvrez comment utiliser PowerShell pour envoyer des invitations en bloc à des utilisateurs externes. Vous effectuez les étapes suivantes :

> [!div class="checklist"]
> * Préparer un fichier de valeurs séparées par des virgules (.csv) avec les informations des utilisateurs
> * Exécuter un script PowerShell pour envoyer les invitations
> * Vérifier que les utilisateurs ont été ajoutés à l’annuaire

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer. 

## <a name="prerequisites"></a>Prérequis

### <a name="install-the-latest-azureadpreview-module"></a>Installez la dernière version du module AzureADPreview.

Vérifiez que vous installez la dernière version du module Azure AD PowerShell for Graph (AzureADPreview). 

Vérifiez d’abord les modules que vous avez installés. Ouvrez Windows PowerShell en tant qu’utilisateur avec privilèges élevés (Exécuter en tant qu’administrateur) et exécutez la commande suivante :

```powershell
Get-Module -ListAvailable AzureAD*
```

En fonction de la sortie, effectuez une des actions suivantes :

- Si aucun résultat n’est retourné, exécutez la commande suivante pour installer le module AzureADPreview :
  
   ```powershell
   Install-Module AzureADPreview
   ```

- Si seul le module AzureAD apparaît dans les résultats, exécutez les commandes suivantes pour installer le module AzureADPreview : 

   ```powershell
   Uninstall-Module AzureAD
   Install-Module AzureADPreview
   ```

- Si seul le module AzureADPreview apparaît dans les résultats mais que vous recevez un message indiquant qu’il existe une version plus récente, exécutez les commandes suivantes pour mettre à jour le module :

   ```powershell
   Uninstall-Module AzureADPreview
   Install-Module AzureADPreview
   ```

Vous pouvez recevoir une invite indiquant que vous installez le module à partir d’un dépôt non approuvé. Ce cas se produit si vous n’aviez pas configuré le dépôt PSGallery comme dépôt fiable. Appuyez sur **O** pour installer le module.

### <a name="get-test-email-accounts"></a>Obtenir des comptes de messagerie de test

Vous avez besoin d’au moins deux comptes de messagerie de test auxquels vous pouvez envoyer les invitations. Les comptes doivent être externes à votre organisation. Vous pouvez utiliser n’importe quel type de compte, notamment des comptes de réseaux sociaux, comme des adresses gmail.com ou outlook.com.

## <a name="prepare-the-csv-file"></a>Préparer le fichier CSV

Dans Microsoft Excel, créez un fichier CSV avec la liste des noms et des adresses e-mail des utilisateurs invités. Veillez à inclure les en-têtes de colonne **Name** et **InvitedUserEmailAddress**.

Par exemple, créez une feuille de calcul dans le format suivant :

![Sortie de PowerShell montrant l’acceptation d’un utilisateur en attente](media/tutorial-bulk-invite/AddUsersExcel.png)

Enregistrez le fichier sous **C:\BulkInvite\Invitations.csv**. 

Si vous n’avez pas Excel, vous pouvez créer un fichier CSV dans n’importe quel éditeur de texte, comme le Bloc-notes. Séparez chaque valeur avec une virgule et chaque ligne avec une nouvelle ligne. 

## <a name="sign-in-to-your-tenant"></a>Connectez-vous à votre locataire

Exécutez la commande suivante pour vous connecter au domaine du locataire :

```powershell
Connect-AzureAD -TenantDomain "<Tenant_Domain_Name>"
```

Par exemple : `Connect-AzureAD -TenantDomain "contoso.onmicrosoft.com"`.

À l’invite, entrez vos informations d’identification.

## <a name="send-bulk-invitations"></a>Envoyer des invitations en bloc

Pour envoyer les invitations, exécutez le script PowerShell suivant (où **c:\bulkinvite\invitations.csv** est le chemin du fichier CSV) :

```powershell
$invitations = import-csv c:\bulkinvite\invitations.csv

$messageInfo = New-Object Microsoft.Open.MSGraph.Model.InvitedUserMessageInfo

$messageInfo.customizedMessageBody = "Hello. You are invited to the Contoso organization."

foreach ($email in $invitations)
   {New-AzureADMSInvitation `
      -InvitedUserEmailAddress $email.InvitedUserEmailAddress `
      -InvitedUserDisplayName $email.Name `
      -InviteRedirectUrl https://myapps.microsoft.com `
      -InvitedUserMessageInfo $messageInfo `
      -SendInvitationMessage $true
   }
```

Le script envoie une invitation aux adresses e-mail du fichier Invitations.csv. Vous devez voir une sortie similaire à ceci pour chaque utilisateur :

![Capture d’écran montrant la sortie PowerShell qui comprend l’acceptation d’un utilisateur en attente.](media/tutorial-bulk-invite/B2BBulkImport.png)

## <a name="verify-users-exist-in-the-directory"></a>Vérifier que les utilisateurs existent dans l’annuaire

Pour vérifier que les utilisateurs invités ont été ajoutés à Azure AD, exécutez la commande suivante :

```powershell
 Get-AzureADUser -Filter "UserType eq 'Guest'"
```

Vous devez y voir figurer les utilisateurs que vous avez invités, avec un nom d’utilisateur principal (UPN) au format *adresse_e-mail* #EXT#\@*domaine*. Par exemple, *lstokes_fabrikam.com#EXT#\@contoso.onmicrosoft.com* , où contoso.onmicrosoft.com est l’organisation à partir de laquelle vous avez envoyé les invitations.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, vous pouvez supprimer les comptes d’utilisateur de test dans l’annuaire. Exécutez la commande suivante pour supprimer un compte d’utilisateur :

```powershell
 Remove-AzureADUser -ObjectId "<UPN>"
```

Par exemple : `Remove-AzureADUser -ObjectId "lstokes_fabrikam.com#EXT#@contoso.onmicrosoft.com"`

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez envoyé des invitations en bloc à des utilisateurs invités en dehors de votre organisation. Découvrez ensuite comment fonctionne le processus d’échange d’invitation.

> [!div class="nextstepaction"]
> [Découvrir plus d’informations sur le processus d’échange d’invitation d’Azure AD B2B Collaboration](redemption-experience.md)
