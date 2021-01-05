---
title: Gérer les méthodes d'authentification d'Azure AD Multi-Factor Authentication - Azure Active Directory
description: Apprenez à configurer les paramètres utilisateur d'Azure Active Directory pour Azure AD Multi-Factor Authentication
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/04/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla, dawoo
ms.collection: M365-identity-device-management
ms.openlocfilehash: a96f01d38fb41f64336d0a2b1d2aa33a96ca9f1a
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742746"
---
# <a name="manage-user-authentication-methods-for-azure-ad-multi-factor-authentication"></a>Gérer les méthodes d'authentification utilisateur d'Azure AD Multi-Factor Authentication

Les utilisateurs d’Azure AD ont deux ensembles distincts de coordonnées :  

- Les coordonnées du profil public, qui sont gérées dans le profil utilisateur et visibles par les membres de votre organisation. Pour les utilisateurs synchronisés depuis Windows Server Active Directory, ces informations sont gérées localement dans Windows Server Active Directory Domain Services.
- Les méthodes d’authentification, qui sont toujours privées et utilisées uniquement pour l’authentification, notamment l’authentification multifacteur (MFA). Les administrateurs peuvent gérer ces méthodes dans le panneau de méthode d’authentification d’un utilisateur, et les utilisateurs peuvent gérer leurs méthodes dans la page Informations de sécurité de Mon compte.

Lors de la gestion des méthodes Azure AD Multi-Factor Authentication de vos utilisateurs, les Administrateurs d'authentification peuvent : 

1. Ajouter des méthodes d’authentification pour un utilisateur spécifique, notamment des numéros de téléphone utilisés pour l’authentification multifacteur.
1. Réinitialisez le mot de passe d’un utilisateur.
1. Demander à un utilisateur de se réinscrire pour l’authentification multifacteur.
1. Révoquer des sessions MFA existantes.
1. Supprimer les mots de passe d’application existants d’un utilisateur.  

## <a name="add-authentication-methods-for-a-user"></a>Ajouter des méthodes d’authentification pour un utilisateur 

Vous pouvez ajouter des méthodes d’authentification pour un utilisateur via le portail Azure ou Microsoft Graph.  

> [!NOTE]
> Pour des raisons de sécurité, les champs publics de coordonnées des utilisateurs ne doivent pas être utilisés pour effectuer l’authentification multifacteur. Au lieu de cela, les utilisateurs doivent renseigner leur numéro de méthode d’authentification à utiliser pour l’authentification multifacteur.  

:::image type="content" source="media/howto-mfa-userdevicesettings/add-authentication-method-detail.png" alt-text="Ajouter des méthodes d’authentification à partir du portail Azure":::

Pour ajouter des méthodes d’authentification pour un utilisateur via le portail Azure :  

1. Connectez-vous au **portail Azure**. 
1. Accédez à **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs**. 
1. Choisissez l’utilisateur auquel vous souhaitez ajouter une méthode d’authentification et sélectionnez **Méthodes d’authentification**.  
1. En haut de la fenêtre, sélectionnez **+ Ajouter une méthode d’authentification**.
   1. Sélectionnez une méthode (numéro de téléphone ou e-mail). L’e-mail peut être utilisé pour la réinitialisation de mot de passe, mais pas pour l’authentification. Lorsque vous ajoutez un numéro de téléphone, sélectionnez un type de téléphone et entrez un numéro de téléphone dans un format valide (par exemple, +1 4255551234).
   1. Sélectionnez **Ajouter**.

> [!NOTE]
> L’expérience de préversion permet aux administrateurs d’ajouter toutes les méthodes d’authentification disponibles pour les utilisateurs, tandis que l’expérience d’origine autorise uniquement la mise à jour des méthodes utilisant un numéro téléphone et un autre de remplacement.

### <a name="manage-methods-using-powershell"></a>Gérer les méthodes à l’aide de PowerShell :  

Installez le module Microsoft.Graph.Identity.Signins PowerShell à l’aide des commandes suivantes. 

```powershell
Install-module Microsoft.Graph.Identity.Signins
Connect-MgGraph -Scopes UserAuthenticationMethod.ReadWrite.All
Select-MgProfile -Name beta
```

Répertoriez les méthodes d’authentification par téléphone pour un utilisateur spécifique.

```powershell
Get-MgUserAuthenticationPhoneMethod -UserId balas@contoso.com
```

Créez une méthode d’authentification par téléphone mobile pour un utilisateur spécifique.

```powershell
New-MgUserAuthenticationPhoneMethod -UserId balas@contoso.com -phoneType “mobile” -phoneNumber "+1 7748933135"
```

Supprimez une méthode par téléphone spécifique pour un utilisateur.

```powershell
Remove-MgUserAuthenticationPhoneMethod -UserId balas@contoso.com -PhoneAuthenticationMethodId 3179e48a-750b-4051-897c-87b9720928f7
```

Les méthodes d’authentification peuvent également être gérées à l’aide d’API Microsoft Graph. Pour plus d’informations, consultez le document [Vue d’ensemble de l’API des méthodes d’authentification Azure AD](/graph/api/resources/authenticationmethods-overview?view=graph-rest-beta&preserve-view=true).

## <a name="manage-user-authentication-options"></a>Gérer les options d’authentification des utilisateurs

Si le rôle *Administrateur d’authentification* vous est attribué, vous pouvez imposer aux utilisateurs de réinitialiser leur mot de passe, de se réinscrire auprès de MFA ou de révoquer les sessions MFA existantes de leur objet utilisateur. Pour gérer les paramètres utilisateur, effectuez les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur la gauche, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs**.
1. Choisissez l’utilisateur sur lequel vous souhaitez effectuer une action et sélectionnez **Méthodes d’authentification**. En haut de la fenêtre, choisissez l’une des options suivantes pour l’utilisateur :
   - **Réinitialiser le mot de passe** permet de réinitialiser le mot de passe de l’utilisateur et d’affecter un mot de passe temporaire qui doit être changé à la prochaine connexion.
   - **Exiger le réenregistrement de la MFA** permet d’imposer à l’utilisateur, à sa prochaine connexion, de configurer une nouvelle méthode d’authentification MFA.
   
      > [!NOTE]
      > Les méthodes d’authentification actuellement inscrites de l’utilisateur ne sont pas supprimées lorsqu’un administrateur requiert une nouvelle inscription pour MFA. Une fois qu’un utilisateur a effectué une nouvelle inscription pour MFA, nous lui recommandons de consulter ses informations de sécurité et de supprimer toutes les méthodes d’authentification précédemment inscrites qui ne sont plus utilisables.
   
   - **Révoquer les sessions MFA** permet d’effacer les sessions MFA mémorisées de l’utilisateur et d’obliger ce dernier à s’authentifier via MFA la prochaine fois que cette méthode est imposée par la stratégie de l’appareil.
   
    :::image type="content" source="media/howto-mfa-userdevicesettings/manage-authentication-methods-in-azure.png" alt-text="Gérer les méthodes d’authentification à partir du portail Azure":::

## <a name="delete-users-existing-app-passwords"></a>Supprimer les mots de passe d’application existants des utilisateurs

Pour les utilisateurs qui ont défini des mots de passe d’application, les administrateurs peuvent également choisir de supprimer ces mots de passe, ce qui entraînera l’échec de l’authentification héritée dans ces applications. Ces actions peuvent être nécessaires si vous devez fournir une assistance à un utilisateur ou si vous souhaitez réinitialiser ses méthodes d’authentification. Les applications sans navigateur qui ont été associées à ces mots de passe d’application cesseront de fonctionner jusqu’à la création d’un nouveau mot de passe d’application. 

Pour supprimer les mots de passe d’application d’un utilisateur, effectuez les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sur la gauche, sélectionnez **Azure Active Directory** > **Utilisateurs** > **Tous les utilisateurs**.
1. Sélectionnez **Multi-Factor Authentication**. Il se peut que vous deviez faire défiler vers la droite pour afficher cette option de menu. Sélectionnez l’exemple de capture d’écran ci-dessous pour afficher entièrement la fenêtre du portail Azure et l’emplacement du menu : [![Select Multi-Factor Authentication from the Users window in Azure AD.](media/howto-mfa-userstates/selectmfa-cropped.png)](media/howto-mfa-userstates/selectmfa.png#lightbox)
1. Cochez la case en regard du ou des utilisateurs que vous souhaitez gérer. Une liste d’options rapides s’affiche à droite.
1. Sélectionnez **Gérer les paramètres utilisateur**, puis cochez la case **Supprimer tous les mots de passe d’application existants qui ont été générés par les utilisateurs sélectionnés**, comme illustré dans l’exemple suivant : ![Supprimer tous les mots de passe d’application existants](./media/howto-mfa-userdevicesettings/deleteapppasswords.png)
1. Sélectionnez **Enregistrer**, puis **Fermer**.

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a montré comment configurer des paramètres utilisateur individuels. Pour configurer l'ensemble des paramètres du service Azure AD Multi-Factor Authentication, consultez [Configurer les paramètres d'Azure AD Multi-Factor Authentication](howto-mfa-mfasettings.md).

Si vos utilisateurs ont besoin d'aide, consultez le [Guide de l'utilisateur d'Azure AD Multi-Factor Authentication](../user-help/multi-factor-authentication-end-user-first-time.md).
