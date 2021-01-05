---
title: Révoquer les accès utilisateur lors d’une urgence dans Azure Active Directory | Microsoft Docs
description: Comment révoquer tous les accès d’un utilisateur dans Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: how-to
author: curtand
ms.author: curtand
manager: daveba
ms.reviewer: krbain
ms.date: 12/02/2020
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 826ca9fc20d8bbcf9a5f90ccc895b9f9867a6be1
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860573"
---
# <a name="revoke-user-access-in-azure-active-directory"></a>Révoquer les accès utilisateur dans Azure Active Directory

Parmi les scénarios susceptibles d’amener un administrateur à révoquer tous les accès d’un utilisateur, citons la compromission des comptes, le départ des employés et d’autres menaces internes. Selon la complexité de l’environnement, les administrateurs peuvent s’assurer de la révocation des accès de plusieurs manières. Dans certains scénarios, on peut observer un délai entre le début de la révocation des accès et le moment où les accès sont réellement révoqués.

Pour atténuer les risques, il est essentiel de bien comprendre le fonctionnement des jetons. Il existe de nombreux types de jetons, qui appartiennent à l’un des modèles mentionnés dans les sections ci-dessous.

## <a name="access-tokens-and-refresh-tokens"></a>Jetons d’accès et jetons d’actualisation

Les jetons d’accès et les jetons d’actualisation sont fréquemment utilisés avec les applications clientes lourdes, mais ils le sont aussi dans les applications basées sur un navigateur, telles que les applications monopage.

- Quand un utilisateur s’authentifie auprès d’Azure AD, les stratégies d’autorisation sont évaluées pour déterminer si l’utilisateur peut être autorisé à accéder à une ressource spécifique.  

- Si l’utilisateur est autorisé, Azure AD émet un jeton d’accès et un jeton d’actualisation pour la ressource.  

- Les jetons d’accès émis par Azure AD restent valides pendant une heure par défaut. Si le protocole d’authentification l’autorise, l’application peut réauthentifier l’utilisateur de manière transparente pour lui, en passant le jeton d’actualisation à Azure AD après l’expiration du jeton d’accès.

Azure AD réévalue ensuite ses stratégies d’autorisation. Si l’utilisateur est toujours autorisé, Azure AD émet un nouveau jeton d’accès et un nouveau jeton d’actualisation.

Les jetons d’accès peuvent poser un problème de sécurité si l’accès doit être révoqué avant la fin de la durée de vie du jeton, qui est généralement d’une heure environ. C’est la raison pour laquelle Microsoft travaille activement à l’intégration de l’[évaluation continue de l’accès](../conditional-access/concept-continuous-access-evaluation.md) aux applications Microsoft 365, le but étant d’assurer l’invalidation des jetons d’accès en quasi-temps réel.  

## <a name="session-tokens-cookies"></a>Jetons de session (cookies)

La plupart des applications basées sur un navigateur utilisent des jetons de session à la place des jetons d’accès et d’actualisation.  

- Quand un utilisateur ouvre un navigateur et s’authentifie auprès d’une application par le biais d’Azure AD, il reçoit deux jetons de session : un d’Azure AD et un autre de l’application.  

- Une fois que l’application a émis son propre jeton de session, l’accès à l’application est régi par la session de l’application. À ce stade, seules les stratégies d’autorisation connues de l’application sont appliquées à l’utilisateur.

- Les stratégies d’autorisation d’Azure AD sont réévaluées aussi souvent que l’application renvoie l’utilisateur vers Azure AD. La réévaluation se produit généralement de manière transparente pour l’utilisateur, à une fréquence variable selon la configuration de l’application. Il est possible que l’application ne renvoie jamais l’utilisateur vers Azure AD tant que le jeton de session est valide.

- Pour qu’un jeton de session puisse être révoqué, l’application doit révoquer l’accès sur la base de ses propres stratégies d’autorisation. Azure AD ne peut pas directement révoquer un jeton de session émis par une application.  

## <a name="revoke-access-for-a-user-in-the-hybrid-environment"></a>Révoquer l’accès d’un utilisateur dans l’environnement hybride

Dans un environnement hybride où un Active Directory local est synchronisé avec Azure Active Directory, Microsoft recommande aux administrateurs informatiques de suivre les procédures ci-après.  

### <a name="on-premises-active-directory-environment"></a>Environnement Active Directory local

En tant qu’administrateur dans l’environnement Active Directory, connectez-vous à votre réseau local, ouvrez PowerShell et effectuez les étapes suivantes :

1. Désactivez l’utilisateur dans Active Directory. Consultez [Disable-ADAccount](/powershell/module/addsadministration/disable-adaccount).

    ```PowerShell
    Disable-ADAccount -Identity johndoe  
    ```

1. Réinitialisez le mot de passe de l’utilisateur à deux reprises dans Active Directory. Consultez [Set-ADAccountPassword](/powershell/module/addsadministration/set-adaccountpassword).

    > [!NOTE]
    > Changer deux fois de suite le mot de passe d’un utilisateur contribue à atténuer le risque d’attaque de type pass-the-hash, surtout quand des délais sont observés durant la réplication du mot de passe local. Si vous avez l’assurance que ce compte n’est pas compromis, vous pouvez vous contenter de réinitialiser le mot de passe une seule fois.

    > [!IMPORTANT] 
    > N’employez pas les exemples de mots de passe donnés dans les applets de commande suivantes. Vous devez les remplacer par une chaîne aléatoire.

    ```PowerShell
    Set-ADAccountPassword -Identity johndoe -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "p@ssw0rd1" -Force)
    Set-ADAccountPassword -Identity johndoe -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "p@ssw0rd2" -Force)
    ```

### <a name="azure-active-directory-environment"></a>Environnement Azure Active Directory

En tant qu’administrateur dans Azure Active Directory, ouvrez PowerShell, exécutez ``Connect-AzureAD`` et effectuez les actions suivantes :

1. Désactivez l’utilisateur dans Azure AD. Consultez [Set-AzureADUser](/powershell/module/azuread/Set-AzureADUser).

    ```PowerShell
    Set-AzureADUser -ObjectId johndoe@contoso.com -AccountEnabled $false
    ```
1. Révoquez les jetons d’actualisation Azure AD de l’utilisateur. Consultez [Revoke-AzureADUserAllRefreshToken](/powershell/module/azuread/revoke-azureaduserallrefreshtoken).

    ```PowerShell
    Revoke-AzureADUserAllRefreshToken -ObjectId johndoe@contoso.com
    ```

1. Désactivez les appareils de l’utilisateur. Consultez [Get-AzureADUserRegisteredDevice](/powershell/module/azuread/get-azureaduserregistereddevice).

    ```PowerShell
    Get-AzureADUserRegisteredDevice -ObjectId johndoe@contoso.com | Set-AzureADDevice -AccountEnabled $false
    ```

## <a name="optional-steps"></a>Étapes facultatives

- [Effacez les données d’entreprise des applications gérées par Intune](/mem/intune/apps/apps-selective-wipe).

- [Réinitialisez les appareils d’entreprise pour rétablir leurs paramètres d’usine](/mem/intune/remote-actions/devices-wipe).

> [!NOTE]
> Après une réinitialisation, les données de l’appareil ne peuvent plus être récupérées.

## <a name="when-access-is-revoked"></a>Après la révocation des accès

Une fois que les administrateurs ont effectué les étapes ci-dessus, l’utilisateur ne peut pas obtenir de nouveaux jetons pour aucune des applications liées à Azure Active Directory. Le temps écoulé entre la révocation des accès de l’utilisateur et la perte effective des accès dépend de la manière dont l’application accorde les accès :

- Pour les **applications utilisant des jetons d’accès**, l’utilisateur perd ses accès dès que le jeton d’accès arrive à expiration.

- Pour les **applications utilisant des jetons de session**, les sessions actives prennent fin dès que le jeton arrive à expiration. Si l’état désactivé de l’utilisateur est synchronisé avec l’application, l’application peut révoquer automatiquement les sessions actives de l’utilisateur si elle est configurée pour cela.  Le délai de révocation dépend de la fréquence de synchronisation entre l’application et Azure AD.

## <a name="next-steps"></a>Étapes suivantes

- [Pratiques de sécurisation des accès pour les administrateurs d’Azure AD](../roles/security-planning.md)
- [Ajouter ou mettre à jour les informations de profil utilisateur](../fundamentals/active-directory-users-profile-azure-portal.md)
