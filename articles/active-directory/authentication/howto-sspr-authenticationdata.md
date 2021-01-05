---
title: Préremplir les informations de contact pour la réinitialisation de mot de passe en libre-service - Azure Active Directory
description: Découvrez comment préremplir les informations de contact pour les utilisateurs de la fonctionnalité SSPR (réinitialisation de mot de passe en libre-service) d’Azure Active Directory afin qu’ils puissent utiliser la fonctionnalité sans effectuer de processus d’inscription.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 10/05/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ed93944c2a94370250fb75a679c1ac37a2a8418c
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861151"
---
# <a name="pre-populate-user-authentication-contact-information-for-azure-active-directory-self-service-password-reset-sspr"></a>Préremplir les informations de contact relatives à l’authentification utilisateur pour la réinitialisation de mot de passe en libre-service Azure Active Directory (SSPR)

Pour permettre l’utilisation de la fonctionnalité SSPR (réinitialisation de mot de passe en libre-service) d’Azure Active Directory (Azure AD), les informations de contact relatives à l’authentification d’un utilisateur doivent être présentes. Certaines organisations demandent aux utilisateurs d’inscrire eux-mêmes leurs données d’authentification. D’autres organisations préfèrent se synchroniser à partir des données d’authentification déjà présentes dans les services AD DS (Active Directory Domain Services). Ces données synchronisées sont accessibles à Azure AD et SSPR sans nécessiter d’interaction de l’utilisateur. Quand les utilisateurs doivent changer ou réinitialiser leur mot de passe, ils peuvent le faire même s’ils n’ont pas inscrit leurs informations de contact.

Vous pouvez préremplir les informations de contact relatives à l’authentification si vous respectez les exigences suivantes :

* Vous avez correctement préparé les données au format approprié dans votre annuaire local.
* Vous avez configuré [Azure AD Connect](../hybrid/how-to-connect-install-express.md) pour votre locataire Azure AD.

Les numéros de téléphone doivent être au format *+IndicatifTéléphoniqueInternational NuméroTéléphone*, par exemple *+1 4251234567*.

> [!NOTE]
> Un espace est obligatoire entre l’indicatif téléphonique international et le numéro de téléphone.
>
> La réinitialisation du mot de passe ne prend pas en charge les extensions de téléphone. Même au format *+1 4251234567X12345*, les extensions sont supprimées avant l’appel.

## <a name="fields-populated"></a>Champs renseignés

Si vous utilisez les paramètres par défaut dans Azure AD Connect, les mappages suivants sont effectués afin que les informations de contact relatives à l’authentification soient renseignées pour SSPR :

| Active Directory local | Azure AD     |
|------------------------------|--------------|
| telephoneNumber              | Téléphone de bureau |
| mobile                       | Téléphone mobile |

Une fois qu’un utilisateur confirme son numéro de téléphone mobile, le champ *Téléphone* situé sous **Informations de contact d’authentification** dans Azure AD est également renseigné avec ce numéro.

## <a name="authentication-contact-info"></a>Informations de contact d’authentification

Sur la page **Méthodes d’authentification** d’un utilisateur Azure AD dans le Portail Azure, un administrateur général peut définir manuellement les informations de contact d’authentification. Vous pouvez passer en revue les méthodes existantes sous la section *Méthodes d’authentification utilisables* ou **Ajouter des méthodes d’authentification**, comme indiqué dans la capture d’écran suivante :

:::image type="content" source="media/howto-sspr-authenticationdata/user-authentication-contact-info.png" alt-text="Gérer les méthodes d’authentification à partir du portail Azure":::

Les considérations suivantes s’appliquent pour ces informations de contact relatives à l’authentification :

* Si le champ *Téléphone* est renseigné et si l’option *Téléphone mobile* est activée dans la stratégie SSPR, l’utilisateur voit ce numéro sur la page d’inscription de réinitialisation du mot de passe et lors du workflow de réinitialisation du mot de passe.
* Si le champ *Adresse e-mail* est renseigné et si l’option *Adresse e-mail* est activée dans la stratégie SSPR, l’utilisateur voit cette adresse e-mail sur la page d’inscription de réinitialisation du mot de passe et lors du workflow de réinitialisation du mot de passe.

## <a name="security-questions-and-answers"></a>Questions et réponses de sécurité

Les questions et les réponses de sécurité sont stockées de manière sécurisée dans votre locataire Azure AD et sont uniquement accessibles aux utilisateurs par le biais du [portail d’inscription SSPR](https://aka.ms/ssprsetup). Les administrateurs ne peuvent pas voir, définir ou modifier le contenu des questions et des réponses des autres utilisateurs.

## <a name="what-happens-when-a-user-registers"></a>Ce qu’il se passe lorsqu'un utilisateur s'inscrit

Lorsqu’un utilisateur s'inscrit, la page d’inscription définit les champs suivants :

* **Téléphone d’authentification**
* **E-mail d’authentification**
* **Questions et réponses de sécurité**

Si vous avez fourni une valeur pour *Téléphone mobile* ou *Adresse e-mail de secours*, les utilisateurs peuvent immédiatement s’en servir pour réinitialiser leur mot de passe, même s’ils ne se sont pas inscrits au service.

Les utilisateurs voient également ces valeurs quand ils s’inscrivent pour la première fois, et peuvent les modifier s’ils le souhaitent. Une fois l’inscription des utilisateurs correctement effectuée, ces valeurs sont conservées dans les champs *Téléphone d’authentification* et *E-mail d’authentification*, respectivement.

## <a name="set-and-read-the-authentication-data-through-powershell"></a>Définir et lire les données d’authentification par le biais de PowerShell

Vous pouvez définir les champs suivants par le biais de PowerShell :

* *Adresse électronique secondaire*
* *Téléphone mobile*
* *Téléphone de bureau*
    * À définir uniquement en l’absence de synchronisation avec un annuaire local.

> [!IMPORTANT]
> Il existe un manque de parité connu dans les fonctionnalités de commande entre PowerShell v1 et PowerShell v2. L’[API REST Microsoft Graph (bêta) pour les méthodes d’authentification](/graph/api/resources/authenticationmethods-overview) est l’objectif d’ingénierie actuel qui permet de fournir une interaction moderne.

### <a name="use-powershell-version-1"></a>Utiliser PowerShell version 1

Pour commencer, [téléchargez et installez le module Azure AD PowerShell](/previous-versions/azure/jj151815(v=azure.100)#bkmk_installmodule). Une fois son installation effectuée, effectuez les étapes suivantes pour configurer chaque champ.

#### <a name="set-the-authentication-data-with-powershell-version-1"></a>Définir les données d’authentification avec PowerShell version 1

```PowerShell
Connect-MsolService

Set-MsolUser -UserPrincipalName user@domain.com -AlternateEmailAddresses @("email@domain.com")
Set-MsolUser -UserPrincipalName user@domain.com -MobilePhone "+1 4251234567"
Set-MsolUser -UserPrincipalName user@domain.com -PhoneNumber "+1 4252345678"

Set-MsolUser -UserPrincipalName user@domain.com -AlternateEmailAddresses @("email@domain.com") -MobilePhone "+1 4251234567" -PhoneNumber "+1 4252345678"
```

#### <a name="read-the-authentication-data-with-powershell-version-1"></a>Lire les données d’authentification avec PowerShell version 1

```PowerShell
Connect-MsolService

Get-MsolUser -UserPrincipalName user@domain.com | select AlternateEmailAddresses
Get-MsolUser -UserPrincipalName user@domain.com | select MobilePhone
Get-MsolUser -UserPrincipalName user@domain.com | select PhoneNumber

Get-MsolUser | select DisplayName,UserPrincipalName,AlternateEmailAddresses,MobilePhone,PhoneNumber | Format-Table
```

#### <a name="read-the-authentication-phone-and-authentication-email-options"></a>Lire les options Téléphone d’authentification et E-mail d’authentification

Pour lire le **Téléphone d’authentification** et **l’E-mail d’authentification** quand vous utilisez PowerShell version 1, exécutez les commandes suivantes :

```PowerShell
Connect-MsolService
Get-MsolUser -UserPrincipalName user@domain.com | select -Expand StrongAuthenticationUserDetails | select PhoneNumber
Get-MsolUser -UserPrincipalName user@domain.com | select -Expand StrongAuthenticationUserDetails | select Email
```

### <a name="use-powershell-version-2"></a>Utiliser PowerShell version 2

Pour commencer, [téléchargez et installez le module PowerShell version 2 d’Azure AD](/powershell/module/azuread/).

Pour effectuer une installation rapide à partir de versions récentes de PowerShell qui prennent en charge `Install-Module`, exécutez les commandes suivantes. La première ligne vérifie si le module est déjà installé :

```PowerShell
Get-Module AzureADPreview
Install-Module AzureADPreview
Connect-AzureAD
```

Une fois le module installé, suivez les étapes suivantes pour configurer chaque champ.

#### <a name="set-the-authentication-data-with-powershell-version-2"></a>Définir les données d’authentification avec PowerShell version 2

```PowerShell
Connect-AzureAD

Set-AzureADUser -ObjectId user@domain.com -OtherMails @("email@domain.com")
Set-AzureADUser -ObjectId user@domain.com -Mobile "+1 4251234567"
Set-AzureADUser -ObjectId user@domain.com -TelephoneNumber "+1 4252345678"

Set-AzureADUser -ObjectId user@domain.com -OtherMails @("emails@domain.com") -Mobile "+1 4251234567" -TelephoneNumber "+1 4252345678"
```

#### <a name="read-the-authentication-data-with-powershell-version-2"></a>Lire les données d’authentification avec PowerShell version 2

```PowerShell
Connect-AzureAD

Get-AzureADUser -ObjectID user@domain.com | select otherMails
Get-AzureADUser -ObjectID user@domain.com | select Mobile
Get-AzureADUser -ObjectID user@domain.com | select TelephoneNumber

Get-AzureADUser | select DisplayName,UserPrincipalName,otherMails,Mobile,TelephoneNumber | Format-Table
```

## <a name="next-steps"></a>Étapes suivantes

Une fois les informations de contact relatives à l’authentification prérenseignées pour les utilisateurs, suivez le tutoriel ci-après pour activer la réinitialisation de mot de passe en libre-service :

> [!div class="nextstepaction"]
> [Activer la réinitialisation de mot de passe en libre-service Azure AD](tutorial-enable-sspr.md)
