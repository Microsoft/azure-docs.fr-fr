---
title: Inviter des utilisateurs internes sur B2B Collaboration - Azure AD
description: Si vous disposez de comptes d’utilisateurs internes pour les partenaires, les distributeurs, les fournisseurs, les prestataires et autres invités, vous pouvez passer à Azure AD B2B Collaboration en les invitant à se connecter avec leurs propres informations d’identification ou informations de connexion externes. Utilisez PowerShell ou l’API d’invitation Microsoft Graph.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 04/12/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: 75839893d0048841440f2c790df7d23ad825d962
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97093466"
---
# <a name="invite-internal-users-to-b2b-collaboration"></a>Inviter des utilisateurs internes sur B2B Collaboration

> [!NOTE]
> L’invitation d’utilisateurs internes sur B2B Collaboration est une fonctionnalité en préversion publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Avant la mise à disposition d’Azure AD B2B Collaboration, les organisations pouvaient collaborer avec les distributeurs, les fournisseurs, les prestataires et autres utilisateurs invités en définissant leurs informations d’identification internes. Si vous avez des utilisateurs invités internes de ce type, vous pouvez les inviter à utiliser B2B Collaboration pour bénéficier des avantages d’Azure AD B2B. Vos utilisateurs invités B2B peuvent utiliser leurs propres identités et informations d’identification pour se connecter. Vous n’avez pas besoin de conserver les mots de passe ni de gérer les cycles de vie des comptes.

L’envoi d’une invitation à un compte interne existant vous permet de conserver l’ID d’objet, l’UPN, les appartenances aux groupes ainsi que les affectations d’applications de l’utilisateur. Vous n’avez pas besoin de supprimer et de réinviter manuellement l’utilisateur ou de réaffecter des ressources. Pour inviter l’utilisateur, vous pouvez vous servir de l’API d’invitation afin de passer à la fois l’objet utilisateur interne et l’adresse e-mail de l’utilisateur invité avec l’invitation. Quand l’utilisateur accepte l’invitation, le service B2B remplace l’objet utilisateur interne existant par un utilisateur B2B. Par la suite, l’utilisateur doit se connecter aux services de ressources cloud à l’aide de ses informations d’identification B2B. Il peut toujours utiliser ses informations d’identification internes pour accéder aux ressources locales, mais vous pouvez l’éviter en réinitialisant ou en changeant le mot de passe du compte interne.

> [!NOTE]
> L’invitation est unidirectionnelle. Vous pouvez inviter des utilisateurs internes à utiliser B2B Collaboration, mais vous ne pouvez pas supprimer les informations d’identification B2B une fois qu’elles ont été ajoutées. Pour rétablir l’utilisateur en tant qu’utilisateur interne uniquement, vous devez supprimer l’objet utilisateur et en créer un autre.

En préversion publique, la méthode décrite dans cet article pour l’invitation d’utilisateurs internes sur B2B Collaboration ne peut pas être utilisée dans les cas suivants :

- Une licence Exchange a été attribuée à l’utilisateur interne.
- L’utilisateur provient d’un domaine configuré pour la fédération directe dans votre annuaire.
- L’utilisateur interne est un compte cloud uniquement, et son compte principal n’est pas dans Azure AD.

Dans les cas de ce genre, si l’utilisateur interne doit être changé en utilisateur B2B, vous devez supprimer le compte interne et envoyer à l’utilisateur une invitation permettant d’accéder à B2B Collaboration.

**Utilisateurs synchronisés localement** : Pour les comptes d’utilisateurs synchronisés localement et dans le cloud, l’annuaire local reste la source d’autorité une fois que les utilisateurs ont été invités à accéder à B2B Collaboration. Tous les changements que vous apportez au compte local sont synchronisés avec le compte cloud, notamment la désactivation ou la suppression du compte. Vous ne pouvez donc pas empêcher l’utilisateur de se connecter à son compte local tout en conservant son compte cloud, en supprimant simplement le compte local. À la place, vous pouvez affecter un GUID aléatoire ou toute autre valeur inconnue au mot de passe du compte local.

## <a name="how-to-invite-internal-users-to-b2b-collaboration"></a>Guide pratique pour inviter des utilisateurs internes sur B2B Collaboration

Vous pouvez utiliser PowerShell ou l’API d’invitation pour envoyer une invitation B2B à l’utilisateur interne. Vérifiez que l’adresse e-mail de l’invitation est définie en tant qu’adresse e-mail externe pour l’objet utilisateur interne.

- Dans le cas d’un utilisateur cloud uniquement, utilisez l’adresse e-mail de la propriété User.OtherMails pour l’invitation.
- Dans le cas d’un utilisateur synchronisé localement, vous devez utiliser la valeur de la propriété User.Mail pour l’invitation.
- Le domaine figurant dans la propriété Mail de l’utilisateur doit correspondre au compte dont il se sert pour se connecter. Sinon, certains services tels que Teams ne pourront pas authentifier l’utilisateur.

Par défaut, l’invitation envoie à l’utilisateur un e-mail l’informant qu’il a été invité, mais vous pouvez supprimer cet e-mail et envoyer le vôtre à la place.

> [!NOTE]
> Pour envoyer votre propre e-mail ou toute autre communication, utilisez New-AzureADMSInvitation avec -SendInvitationMessage:$false afin d’inviter les utilisateurs sans assistance, puis envoyez votre propre e-mail à l’utilisateur converti. Consultez [API et personnalisation d’Azure AD B2B Collaboration](customize-invitation-api.md).

## <a name="use-powershell-to-send-a-b2b-invitation"></a>Utiliser PowerShell pour envoyer une invitation B2B

Utilisez la commande suivante pour inviter l’utilisateur sur B2B Collaboration :

```powershell
Uninstall-Module AzureADPreview
Install-Module AzureADPreview
$ADGraphUser = Get-AzureADUser -objectID "UPN of Internal User"
$msGraphUser = New-Object Microsoft.Open.MSGraph.Model.User -ArgumentList $ADGraphUser.ObjectId
New-AzureADMSInvitation -InvitedUserEmailAddress <<external email>> -SendInvitationMessage $True -InviteRedirectUrl "http://myapps.microsoft.com" -InvitedUser $msGraphUser
```

## <a name="use-the-invitation-api-to-send-a-b2b-invitation"></a>Utiliser l’API d’invitation pour envoyer une invitation B2B

L’exemple ci-dessous montre comment appeler l’API d’invitation pour inviter un utilisateur interne en tant qu’utilisateur B2B.

```json
POST https://graph.microsoft.com/v1.0/invitations
Authorization: Bearer eyJ0eX...
ContentType: application/json
{
    "invitedUserEmailAddress": "<<external email>>",
    "sendInvitationMessage": true,
    "invitedUserMessageInfo": {
        "messageLanguage": "en-US",
        "ccRecipients": [
            {
                "emailAddress": {
                    "name": null,
                    "address": "<<optional additional notification email>>"
                }
            }
        ],
        "customizedMessageBody": "<<custom message>>"
    },
    "inviteRedirectUrl": "https://myapps.microsoft.com?tenantId=",
    "invitedUser": {
        "id": "<<ID for the user you want to convert>>"
    }
}
```

La réponse à l’API est la même que celle obtenue quand vous invitez un nouvel utilisateur invité à accéder à l’annuaire.

## <a name="next-steps"></a>Étapes suivantes

- [Utilisation d’une invitation B2B Collaboration](redemption-experience.md)
