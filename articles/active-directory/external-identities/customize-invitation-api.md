---
title: API et personnalisation de collaboration B2B - Azure Active Directory
description: Azure Active Directory B2B Collaboration prend en charge les relations interentreprises en permettant aux partenaires commerciaux d’accéder de façon sélective à vos applications d’entreprise.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 04/11/2017
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.collection: M365-identity-device-management
ms.openlocfilehash: b7cbcdb4b947e4b45a5473dc0f9f0252b5ad1d5c
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92442046"
---
# <a name="azure-active-directory-b2b-collaboration-api-and-customization"></a>API et personnalisation d’Azure Active Directory B2B Collaboration

De nombreux clients nous ont indiqué qu’ils souhaitent personnaliser le processus d’invitation d’une manière qui convient mieux à leurs organisations. Avec notre API, cela est possible. [https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation](/graph/api/resources/invitation)

## <a name="capabilities-of-the-invitation-api"></a>Fonctionnalités de l’API d’invitation

L’API offre les fonctionnalités suivantes :

1. Invitez un utilisateur externe avec *n’importe quelle* adresse e-mail.

    ```
    "invitedUserDisplayName": "Sam"
    "invitedUserEmailAddress": "gsamoogle@gmail.com"
    ```

2. Personnalisez l’emplacement vers lequel vous souhaitez diriger vos utilisateurs une fois qu’ils ont accepté l’invitation.

    ```
    "inviteRedirectUrl": "https://myapps.microsoft.com/"
    ```

3. Choisissez d’envoyer l’e-mail d’invitation standard par notre intermédiaire

    ```
    "sendInvitationMessage": true
    ```

   accompagné d’un message au destinataire que vous pouvez personnaliser

    ```
    "customizedMessageBody": "Hello Sam, let's collaborate!"
    ```

4. Et choisissez de mettre en copie des personnes que vous souhaitez informer de votre invitation de ce collaborateur.

5. Ou personnalisez entièrement votre flux de travail d’invitation et d’accueil en choisissant de ne pas envoyer des notifications via Azure AD.

    ```
    "sendInvitationMessage": false
    ```

   Dans ce cas, vous obtenez une URL d’échange de la part de l’API que vous pouvez incorporer dans un modèle d’e-mail, un message instantané ou toute autre méthode de distribution de votre choix.

6. Enfin, si vous êtes un administrateur, vous pouvez choisir d’inviter l’utilisateur en tant que membre.

    ```
    "invitedUserType": "Member"
    ```


## <a name="authorization-model"></a>Modèle d’autorisation

L’API peut être exécutée dans les modes d’autorisation suivants :

### <a name="app--user-mode"></a>Mode Application + Utilisateur

Dans ce mode, toute personne utilisant l’API doit disposer des autorisations permettant de créer des invitations B2B.

### <a name="app-only-mode"></a>Mode Application uniquement

Dans le contexte Application uniquement, l’application a besoin de l’étendue User.Invite.All pour que l’invitation réussisse.

Pour plus d'informations, consultez : https://developer.microsoft.com/graph/docs/authorization/permission_scopes


## <a name="powershell"></a>PowerShell

Vous pouvez utiliser PowerShell pour ajouter et inviter facilement des utilisateurs externes au sein d’une organisation. Créez une invitation à l’aide de l’applet de commande suivante :

```powershell
New-AzureADMSInvitation
```

Vous pouvez utiliser les options suivantes :

* -InvitedUserDisplayName
* -InvitedUserEmailAddress
* -SendInvitationMessage
* -InvitedUserMessageInfo

### <a name="invitation-status"></a>Statut de l’invitation

Une fois que vous avez envoyé une invitation à un utilisateur externe, vous pouvez utiliser l’applet de commande **Get-AzureADUser** pour voir s’il l’a acceptée. Les propriétés suivantes de l’applet de commande Get-AzureADUser sont remplies quand une invitation est envoyée à un utilisateur externe :

* **UserState** indique si le statut de l’invitation est **PendingAcceptance** (En attente d’acceptation) ou **Accepted** (Acceptée).
* **UserStateChangedOn** affiche l’horodatage de la modification la plus récente de la propriété **UserState** .

Vous pouvez utiliser l’option **Filter** pour filtrer les résultats en fonction de la propriété **UserState** . L’exemple ci-dessous montre comment filtrer les résultats pour afficher uniquement les utilisateurs qui ont une invitation en attente. L’exemple illustre également l’option **Format-List** , qui vous permet de spécifier les propriétés à afficher. 
 

```powershell
Get-AzureADUser -Filter "UserState eq 'PendingAcceptance'" | Format-List -Property DisplayName,UserPrincipalName,UserState,UserStateChangedOn
```

> [!NOTE]
> Vérifiez que vous disposez de la dernière version du module AzureAD PowerShell ou du module AzureADPreview PowerShell. 

## <a name="see-also"></a>Voir aussi

Consultez les informations de référence sur les API d’invitation dans [https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation](/graph/api/resources/invitation).

## <a name="next-steps"></a>Étapes suivantes

- [Qu'est-ce que la collaboration B2B d'Azure AD ?](what-is-b2b.md)
- [Éléments de l’e-mail d’invitation de B2B Collaboration](invitation-email-elements.md)
- [Utilisation d’une invitation B2B Collaboration](redemption-experience.md)
- [Ajouter des utilisateurs B2B Collaboration sans invitation](add-user-without-invite.md)