---
title: Activer Multi-Factor Authentication - Azure Active Directory
description: Apprenez à activer Azure AD Multi-Factor Authentication par utilisateur en modifiant l'état de l'utilisateur
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 08/17/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 749829f641119273813d3c8ca826daf8b4dc4d11
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742661"
---
# <a name="enable-per-user-azure-ad-multi-factor-authentication-to-secure-sign-in-events"></a>Activer Azure AD Multi-Factor Authentication par utilisateur pour sécuriser les événements de connexion

Pour sécuriser les événements de connexion de l’utilisateur dans Azure AD, vous pouvez exiger l’utilisation de l’authentification multifacteur (MFA). Il est recommandé d'activer Azure AD Multi-Factor Authentication à l'aide de stratégies d'accès conditionnel pour protéger les utilisateurs. L’accès conditionnel est une fonctionnalité Azure AD Premium P1 ou P2 qui vous permet d’appliquer des règles pour exiger l’authentification multifacteur selon les besoins dans certains scénarios. Pour commencer à utiliser l’accès conditionnel, consultez [Tutoriel : Événements de connexion utilisateur sécurisée avec Azure AD Multi-Factor Authentication](tutorial-enable-azure-mfa.md).

Pour les locataires libres Azure AD sans accès conditionnel, vous pouvez [utiliser les paramètres de sécurité par défaut pour protéger les utilisateurs](../fundamentals/concept-fundamentals-security-defaults.md). Les utilisateurs sont invités à utiliser l’authentification multifacteur le cas échéant, mais vous ne pouvez pas définir vos propres règles pour contrôler le comportement.

Si nécessaire, vous pouvez activer Azure AD Multi-Factor Authentication par utilisateur sur chaque compte. S’il est activé individuellement, l’utilisateur effectue l’authentification multifacteur chaque fois qu’il se connecte (à quelques exceptions près, notamment lorsqu’il se connecte à partir d’adresses IP approuvées ou que la fonctionnalité de _mémorisation de l’authentification multifacteur sur les appareils approuvés_ est activée).

Il n’est pas recommandé de changer d’état d’utilisateur, sauf si vos licences Azure AD n’incluent pas l’accès conditionnel et que vous ne souhaitez pas utiliser les paramètres de sécurité par défaut. Pour plus d'informations sur les différentes façons d'activer l'authentification multifacteur, consultez [Fonctionnalités et licences Azure AD Multi-Factor Authentication](concept-mfa-licensing.md).

> [!IMPORTANT]
>
> Cet article explique comment afficher et modifier l'état d'Azure AD Multi-Factor Authentication par utilisateur. Si vous utilisez l’accès conditionnel ou les paramètres de sécurité par défaut, vous ne passez pas en revue ni n’activez les comptes d’utilisateur en suivant ces étapes.
>
> L'activation d'Azure AD Multi-Factor Authentication via une stratégie d'accès conditionnel ne modifie pas l'état de l'utilisateur. Ne soyez pas surpris si les utilisateurs sont signalés comme étant désactivés. L’accès conditionnel ne change pas l’état.
>
> **N'activez pas ou n'appliquez pas Azure AD Multi-Factor Authentication par utilisateur si vous utilisez des stratégies d'accès conditionnel.**

## <a name="azure-ad-multi-factor-authentication-user-states"></a>Azure AD Multi-Factor Authentication - État de l'utilisateur

L'état d'un utilisateur indique si un administrateur l'a inscrit à Azure AD Multi-Factor Authentication par utilisateur. Les comptes d'utilisateur d'Azure AD Multi-Factor Authentication peuvent présenter les trois états suivants :

| État | Description | Authentification héritée affectée | Applications du navigateur affectées | Authentification moderne affectée |
|:---:| --- |:---:|:--:|:--:|
| Désactivé | État par défaut d'un utilisateur non inscrit à Azure AD Multi-Factor Authentication par utilisateur. | Non | Non | Non |
| activé | L'utilisateur est inscrit à Azure AD Multi-Factor Authentication par utilisateur, mais peut toujours utiliser son mot de passe pour une authentification héritée. Si l’utilisateur n’a pas encore inscrit de méthode d’authentification multifacteur, il reçoit une invite à s’y inscrire la prochaine fois qu’il se connecte à l’aide de l’authentification moderne (par exemple, via un navigateur web). | Non. L’authentification héritée continue de fonctionner jusqu’à ce que le processus d’inscription soit terminé. | Oui. Après expiration de la session, l'inscription à Azure AD Multi-Factor Authentication est nécessaire.| Oui. Après expiration du jeton d'accès, l'inscription à Azure AD Multi-Factor Authentication est nécessaire. |
| Appliquée | L'utilisateur est inscrit à Azure AD Multi-Factor Authentication par utilisateur. Si l’utilisateur n’a pas encore inscrit de méthode d’authentification, il reçoit une invite à s’y inscrire la prochaine fois qu’il se connecte à l’aide de l’authentification moderne (par exemple, via un navigateur web). Les utilisateurs qui s’inscrive alors qu’ils sont à l’état *Activé* passent automatiquement à l’état *Appliqué*. | Oui. Les applications requièrent des mots de passe d'application. | Oui. Une authentification Azure AD MFA est requise à la connexion. | Oui. Une authentification Azure AD MFA est requise à la connexion. |

Tous les utilisateurs commencent avec l’état *Désactivé*. Dès lors qu'ils sont inscrits à Azure AD Multi-Factor Authentication par utilisateur, leur état devient *Activé*. Lorsque les utilisateurs activés se connectent et suivent le processus d’inscription, leur état passe à *Appliqué*. Les administrateurs peuvent passer des utilisateurs d’un état à un autre, notamment d’*Appliqué* à *Activé* ou *Désactivé*.

> [!NOTE]
> Si l’authentification multifacteur par utilisateur est réactivée sur un utilisateur et que celui-ci ne se réinscrit pas, son état d’authentification multifacteur ne passe pas d’*Activé* à *Appliqué* dans l’interface utilisateur de la gestion de l’authentification multifacteur. L’administrateur doit passer l’utilisateur directement à l’état *Appliqué*.

## <a name="view-the-status-for-a-user"></a>Afficher l’état d’un utilisateur

Pour afficher et gérer les états des utilisateurs, procédez comme suit pour accéder à la page Portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur.
1. Recherchez et sélectionnez *Azure Active Directory*, puis choisissez **Utilisateurs** > **Tous les utilisateurs**.
1. Sélectionnez **Multi-Factor Authentication**. Il se peut que vous deviez faire défiler vers la droite pour afficher cette option de menu. Sélectionnez l’exemple de capture d’écran ci-dessous pour voir l’ensemble de la fenêtre du portail Azure et l’emplacement du menu : [![Sélectionnez Authentification multifacteur dans la fenêtre Utilisateurs d’Azure AD.](media/howto-mfa-userstates/selectmfa-cropped.png)](media/howto-mfa-userstates/selectmfa.png#lightbox)
1. Une nouvelle page s’ouvre et affiche l’état de l’utilisateur, comme illustré dans l’exemple suivant.
   ![Capture d'écran présentant des exemples d'informations sur l'état de l'utilisateur pour Azure AD Multi-Factor Authentication](./media/howto-mfa-userstates/userstate1.png)

## <a name="change-the-status-for-a-user"></a>Modifier l’état d’un utilisateur

Pour modifier l'état d'un utilisateur dans Azure AD Multi-Factor Authentication par utilisateur, procédez comme suit :

1. Suivez les étapes précédentes pour [afficher l'état d'un utilisateur](#view-the-status-for-a-user) et accéder à la page **Utilisateurs** d'Azure AD Multi-Factor Authentication.
1. Recherchez l'utilisateur que vous souhaitez activer pour Azure AD Multi-Factor Authentication par utilisateur. Vous devrez peut-être modifier l’affichage en haut de la page afin d’afficher les **utilisateurs**.
   ![Sélectionner l’utilisateur dont l’état doit être modifié dans l’onglet Utilisateurs](./media/howto-mfa-userstates/enable1.png)
1. Activez la case à cocher en regard des noms des utilisateurs dont vous souhaitez modifier l’état.
1. Sur la partie droite, sous **étapes rapides**, cliquez sur **Activer** ou **Désactiver**. Dans l’exemple suivant, l’utilisateur *John Smith* affiche une coche en regard de son nom et il est autorisé à utiliser : ![Activer l’utilisateur sélectionné en cliquant sur Activer dans le menu étapes rapides](./media/howto-mfa-userstates/user1.png)

   > [!TIP]
   > Les utilisateurs *activés* basculent automatiquement vers l'état *Appliqué* lorsqu'ils s'inscrivent à Azure AD Multi-Factor Authentication. Ne modifiez pas manuellement l’état de l’utilisateur en *Appliqué*, sauf si l’utilisateur est déjà inscrit ou s’il est acceptable que l’utilisateur subisse une interruption de connexion aux protocoles d’authentification hérités.

1. Confirmez votre sélection dans la fenêtre contextuelle qui s’ouvre.

Dès que vous avez activé les utilisateurs, informez-les-en par e-mail. Informez les utilisateurs qu’une invite s’affiche et leur demande de s’inscrire la prochaine fois qu’ils se connectent. Par ailleurs, si votre organisation utilise des applications sans navigateur qui ne prennent pas en charge l’authentification moderne, vos utilisateurs devront créer des mots de passe d’application. Pour plus d'informations, consultez le [Guide de l'utilisateur final d'Azure AD Multi-Factor Authentication](../user-help/multi-factor-authentication-end-user-first-time.md) pour les aider à commencer.

## <a name="change-state-using-powershell"></a>Modifier l’état à l’aide de PowerShell

Pour modifier l’état utilisateur avec [Azure AD PowerShell](/powershell/azure/), modifiez le paramètre `$st.State` pour un compte d’utilisateur. Il existe trois états possibles pour un compte d'utilisateur :

* *Activé*
* *Appliqué*
* *Désactivé*  

En général, ne passez pas les utilisateurs directement à l’état *Appliqué*, sauf s’ils sont déjà inscrits pour l’authentification multifacteur. En effet, dans ce cas, les applications d'authentification héritées cesseront de fonctionner, car l'utilisateur ne s'est pas inscrit à Azure AD Multi-Factor Authentication et n'a pas obtenu de [mot de passe d'application](howto-mfa-app-passwords.md). Dans certains cas, ce comportement peut être souhaité, mais il a un impact sur l’expérience de l’utilisateur jusqu’à ce que celui-ci s’enregistre.

Pour commencer, installez le module *MSOnline* à l’aide de [Install-Module](/powershell/module/powershellget/install-module), comme suit :

```PowerShell
Install-Module MSOnline
```

Puis connectez-vous à l’aide de [Connect-MsolService](/powershell/module/msonline/connect-msolservice) :

```PowerShell
Connect-MsolService
```

L’exemple de script PowerShell suivant active l’authentification multifacteur pour un utilisateur individuel nommé *bsimon@contoso.com* :

```PowerShell
$st = New-Object -TypeName Microsoft.Online.Administration.StrongAuthenticationRequirement
$st.RelyingParty = "*"
$st.State = "Enabled"
$sta = @($st)

# Change the following UserPrincipalName to the user you wish to change state
Set-MsolUser -UserPrincipalName bsimon@contoso.com -StrongAuthenticationRequirements $sta
```

PowerShell est une bonne option si vous devez activer de nombreux utilisateurs à la fois. Le script suivant parcourt une liste d’utilisateurs et active l’authentification multifacteur sur leurs comptes. Définissez les comptes d’utilisateur sur la première ligne de `$users`, comme suit :

   ```PowerShell
   # Define your list of users to update state in bulk
   $users = "bsimon@contoso.com","jsmith@contoso.com","ljacobson@contoso.com"

   foreach ($user in $users)
   {
       $st = New-Object -TypeName Microsoft.Online.Administration.StrongAuthenticationRequirement
       $st.RelyingParty = "*"
       $st.State = "Enabled"
       $sta = @($st)
       Set-MsolUser -UserPrincipalName $user -StrongAuthenticationRequirements $sta
   }
   ```

Pour désactiver l’authentification multifacteur, l’exemple suivant obtient un utilisateur avec [Get-MsolUser](/powershell/module/msonline/get-msoluser), puis supprime toute valeur *StrongAuthenticationRequirements* définie pour l’utilisateur à l’aide de [Set-MsolUser](/powershell/module/msonline/set-msoluser) :

```PowerShell
Get-MsolUser -UserPrincipalName bsimon@contoso.com | Set-MsolUser -StrongAuthenticationRequirements @()
```

Vous pouvez également désactiver directement l’authentification multifacteur pour un utilisateur à l’aide de [Set-MsolUser](/powershell/module/msonline/set-msoluser), comme suit :

```PowerShell
Set-MsolUser -UserPrincipalName bsimon@contoso.com -StrongAuthenticationRequirements @()
```

## <a name="convert-users-from-per-user-mfa-to-conditional-access"></a>Convertir des utilisateurs de l’authentification multifacteur par utilisateur à l’accès conditionnel

La commande PowerShell suivante peut vous aider à effectuer cette conversion.

```PowerShell
# Sets the MFA requirement state
function Set-MfaState {

    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipelineByPropertyName=$True)]
        $ObjectId,
        [Parameter(ValueFromPipelineByPropertyName=$True)]
        $UserPrincipalName,
        [ValidateSet("Disabled","Enabled","Enforced")]
        $State
    )

    Process {
        Write-Verbose ("Setting MFA state for user '{0}' to '{1}'." -f $ObjectId, $State)
        $Requirements = @()
        if ($State -ne "Disabled") {
            $Requirement =
                [Microsoft.Online.Administration.StrongAuthenticationRequirement]::new()
            $Requirement.RelyingParty = "*"
            $Requirement.State = $State
            $Requirements += $Requirement
        }

        Set-MsolUser -ObjectId $ObjectId -UserPrincipalName $UserPrincipalName `
                     -StrongAuthenticationRequirements $Requirements
    }
}

# Disable MFA for all users
Get-MsolUser -All | Set-MfaState -State Disabled
```

> [!NOTE]
> Si l’authentification multifacteur est réactivée sur un utilisateur et que celui-ci ne se réinscrit pas, son état d’authentification multifacteur ne passe pas d’*Activé* à *Appliqué* dans l’interface utilisateur de la gestion de l’authentification multifacteur. Dans ce cas, l’administrateur doit passer l’utilisateur directement à l’état *Appliqué*.

## <a name="next-steps"></a>Étapes suivantes

Pour configurer les paramètres d'Azure AD Multi-Factor Authentication, consultez [Configurer les paramètres d'Azure AD Multi-Factor Authentication](howto-mfa-mfasettings.md).

Pour gérer les paramètres utilisateur pour Azure AD Multi-Factor Authentication, consultez [Gérer les paramètres utilisateur avec Azure AD Multi-Factor Authentication](howto-mfa-userdevicesettings.md).

Afin de comprendre la raison pour laquelle un utilisateur est invité ou non à effectuer une authentification multifacteur, consultez [Rapports Azure AD Multi-Factor Authentication](howto-mfa-reporting.md).
