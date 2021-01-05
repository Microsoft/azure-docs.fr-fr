---
title: Se connecter en utilisant un e-mail en guise d’ID de connexion alternatif pour Azure Active Directory
description: Découvrez comment configurer de façon à permettre aux utilisateurs de se connecter à Azure Active Directory en utilisant leur adresse e-mail en tant qu’ID de connexion alternatif (préversion)
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 10/01/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: calui
ms.openlocfilehash: 0ca5f6a853852acbb4ef97adfce2364592bae270
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97559838"
---
# <a name="sign-in-to-azure-active-directory-using-email-as-an-alternate-login-id-preview"></a>Se connecter à Azure Active Directory en utilisant un e-mail en guise d’ID de connexion alternatif (préversion)

> [!NOTE]
> La connexion à Azure AD avec une adresse e-mail en guise d’ID de connexion alternatif est une fonctionnalité en préversion publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

De nombreuses organisations souhaitent permettre aux utilisateurs de se connecter à Azure Active Directory (Azure AD) en utilisant les informations d’identification de leur environnement d’annuaire local. En vertu de cette approche, dite d’authentification hybride, les utilisateurs ne doivent se souvenir que d’un seul ensemble d’informations d’identification.

Certaines organisations n’ont pas adopté l’authentification hybride pour les raisons suivantes :

* Par défaut, le nom d’utilisateur principal (UPN, User Principal Name) Azure AD est celui figurant dans l’annuaire local.
* La modification de l’UPN Azure AD crée une discordance entre les environnements local et Azure AD, ce qui peut entraîner des problèmes avec certains services et applications.
* Pour des raisons commerciales ou de conformité, l’organisation ne souhaite pas utiliser l’UPN local pour se connecter à Azure AD.

Pour faciliter l’adoption de l’authentification hybride, vous pouvez désormais configurer Azure AD de façon à permettre aux utilisateurs de se connecter en utilisant une adresse e-mail de votre domaine vérifié en guise d’ID de connexion alternatif. Par exemple, si *Contoso* a été rebaptisé *Fabrikam*, au lieu de continuer à se connecter avec l’UPN `balas@contoso.com` hérité, il est maintenant possible d’utiliser une adresse e-mail en tant qu’ID de connexion alternatif. Pour accéder à une application ou à des services, les utilisateurs se connectent à Azure AD à l’aide de l’e-mail qui leur est attribué, par exemple, `balas@fabrikam.com`.

Cet article explique comment activer et utiliser une adresse e-mail comme ID de connexion de substitution. Cette fonctionnalité est disponible dans l’édition Azure AD Free et les versions ultérieures.

> [!NOTE]
> Cette fonctionnalité est destinée uniquement aux utilisateurs Azure AD authentifiés dans le cloud.

> [!NOTE]
> Actuellement, cette fonctionnalité n’est pas prise en charge sur les appareils Windows 10 joints à Azure AD pour les locataires dotés de l’authentification cloud. Cette fonctionnalité ne s’applique pas aux appareils joints à Azure AD Hybride.

## <a name="overview-of-azure-ad-sign-in-approaches"></a>Vue d’ensemble des approches de la connexion à Azure AD

Pour se connecter à Azure AD, les utilisateurs entrent un nom qui identifie leur compte de manière unique. Historiquement, vous pouviez uniquement utiliser l’UPN Azure AD en tant que nom de connexion.

Pour les organisations dont l’UPN local correspond à l’adresse e-mail de connexion préférée de l’utilisateur, cette approche est intéressante. Ces organisations définissent l’UPN Azure AD sur la même valeur que l’UPN local, et les utilisateurs bénéficient d’une expérience de connexion cohérente.

Cela étant, dans certaines organisations, l’UPN local n’est pas utilisé en tant que nom de connexion. Dans les environnements locaux, vous configurez l’instance AD DS locale de façon à autoriser la connexion avec un ID de connexion de substitution. La définition de l’UPN Azure AD sur la même valeur que l’UPN local n’est pas possible car Azure AD imposerait aux utilisateurs de se connecter avec cette valeur.

La solution de contournement à ce problème consiste à définir l’UPN Azure AD sur l’adresse e-mail que l’utilisateur envisage d’utiliser pour se connecter. Cette approche fonctionne, même si elle donne lieu à des UPN différents entre AD local et Azure AD. Qui plus est, cette configuration n’est pas compatible avec toutes les charges de travail Microsoft 365.

Une autre approche consiste à synchroniser les UPN Azure AD et locaux avec la même valeur, puis à configurer Azure AD pour permettre aux utilisateurs de se connecter à Azure AD à l’aide d’une adresse e-mail vérifiée. Pour ce faire, vous définissez une ou plusieurs adresses e-mail dans l’attribut *ProxyAddresses* de l’utilisateur dans le répertoire local. Les valeurs *ProxyAddresses* sont ensuite synchronisées automatiquement avec Azure AD à l’aide d’Azure AD Connect.

## <a name="preview-limitations"></a>Limitations de la version préliminaire

La connexion à Azure AD avec une adresse e-mail en tant qu’ID de connexion de substitution est disponible dans l’édition Azure AD Free et les versions ultérieures.

Pendant la période de la préversion, les limitations suivantes s’appliquent lorsqu’un utilisateur se connecte en utilisant un e-mail non-UPN comme ID de connexion alternatif :

* Les utilisateurs peuvent voir leur UPN, même quand ils sont connectés avec leur e-mail non-UPN. L’exemple de comportement suivant peut être observé :
    * L’utilisateur est invité à se connecter avec l’UPN lorsqu’il est dirigé vers la connexion Azure AD avec `login_hint=<non-UPN email>`.
    * Lorsqu’un utilisateur se connecte avec un e-mail non-UPN et qu’il entre un mot de passe incorrect, la page *Entrez votre mot de passe* change pour afficher l’UPN.
    * Sur certains sites et applications Microsoft, tels que [https://portal.azure.com](https://portal.azure.com) et Microsoft Office, le contrôle du **gestionnaire de comptes** généralement affiché en haut à droite peut afficher l’UPN de l’utilisateur au lieu de l’e-mail non-UPN utilisé pour la connexion.

* Certains flux ne sont actuellement pas compatibles avec l’e-mail non-UPN, notamment :
    * Identity Protection ne fait pas correspondre actuellement les ID de connexion alternatifs avec la détection de risque *Informations d’identification fuitées*. Cette détection de risque utilise l’UPN pour faire correspondre les informations d’identification qui ont été divulguées. Pour plus d’informations, consultez [Détection et correction des riques Azure AD Identity Protection][identity-protection].
    * Les invitations B2B envoyées à un ID de connexion alternatif ne sont pas entièrement prises en charge. Après avoir accepté une invitation envoyée à un e-mail en tant qu’ID de connexion alternatif, la connexion avec l’e-mail alternatif peut ne pas fonctionner pour l’utilisateur sur le point de terminaison du locataire.

## <a name="synchronize-sign-in-email-addresses-to-azure-ad"></a>Synchroniser les adresses e-mail de connexion sur Azure AD

L’authentification traditionnelle d’Active Directory Domain Services (AD DS) ou des services de fédération Active Directory (AD FS) s’effectue directement sur votre réseau et est gérée par votre infrastructure AD DS. Avec une authentification hybride, les utilisateurs peuvent se connecter directement à Azure AD.

Pour permettre cette approche d’authentification hybride, vous synchronisez votre environnement AD DS local sur Azure AD à l’aide d’[Azure AD Connect][azure-ad-connect], et le configurez pour utiliser la synchronisation du hachage de mot de passe ou l’authentification directe (PTA).

Dans les deux options de configuration, l’utilisateur envoie son nom d’utilisateur et son mot de passe à Azure AD qui valide les informations d’identification et émet un ticket. Lorsque les utilisateurs se connectent à Azure AD, cela évite à votre organisation de devoir héberger et gérer une infrastructure AD FS.

![Diagramme représentant une identité hybride Azure AD avec une synchronisation de hachage du mot de passe](media/howto-authentication-use-email-signin/hybrid-password-hash-sync.png)

![Diagramme représentant une identité hybride Azure AD avec une authentification directe](media/howto-authentication-use-email-signin/hybrid-pass-through-authentication.png)

L’un des attributs utilisateur automatiquement synchronisés par Azure AD Connect est *ProxyAddresses*. Si un utilisateur a une adresse électronique définie dans l’environnement AD DS local comme faisant partie de l’attribut *ProxyAddresses*, cette adresse est automatiquement synchronisée avec Azure AD. Cette adresse e-mail peut ensuite être utilisée directement dans le processus de connexion Azure AD en tant qu’ID de connexion alternatif.

> [!IMPORTANT]
> Seuls les adresses e-mail de domaines vérifiés pour le locataire sont synchronisées sur Azure AD. Chaque locataire Azure AD dispose d’un ou plusieurs domaines vérifiés dont vous avez prouvé la propriété, et qui sont liés de manière unique à votre locataire.
>
> Pour plus d’informations, consultez [Ajouter et vérifier un nom de domaine personnalisé dans Azure AD][verify-domain].

Pour plus d’informations, consultez [Choisir la méthode d’authentification adaptée à votre solution d’identité hybride Azure AD][hybrid-auth-methods].

## <a name="enable-user-sign-in-with-an-email-address"></a>Permettre à un utilisateur de se connecter avec une adresse e-mail

Une fois que les utilisateurs auxquels l’attribut *ProxyAddresses* est appliqué sont synchronisés avec Azure AD à l’aide d’Azure AD Connect, vous devez activer la fonctionnalité pour que les utilisateurs se connectent avec leur adresse e-mail en guise d’ID de connexion alternatif pour votre locataire. Cette fonctionnalité indique aux serveurs de connexion Azure AD de vérifier le nom de connexion non seulement par rapport aux valeurs d’UPN, mais aussi par rapport aux valeurs de *ProxyAddresses* pour l’adresse e-mail.

Actuellement, pendant la période de préversion, vous pouvez activer la fonctionnalité de connexion avec une adresse e-mail en guise d’ID de connexion alternatif uniquement en utilisant PowerShell. Pour les suivre étapes suivantes, vous devez disposer d’autorisations d’*administrateur client* :

1. Ouvrez une session PowerShell en tant qu’administrateur, puis installez le module *AzureADPreview* à l’aide de la cmdlet [Install-Module][Install-Module] :

    ```powershell
    Install-Module AzureADPreview
    ```

    Si vous y êtes invité, sélectionnez **Y** pour installer NuGet ou pour installer à partir d’un dépôt non approuvé.

1. Connectez-vous à votre locataire Azure AD en tant qu’*administrateur client* en utilisant la cmdlet [Connect-AzureAD][Connect-AzureAD] :

    ```powershell
    Connect-AzureAD
    ```

    La commande retourne des informations sur votre compte, votre environnement et votre ID de locataire.

1. Vérifiez si la stratégie *HomeRealmDiscoveryPolicy* existe déjà dans votre locataire en utilisant la cmdlet [Get-AzureADPolicy][Get-AzureADPolicy] comme suit :

    ```powershell
    Get-AzureADPolicy | where-object {$_.Type -eq "HomeRealmDiscoveryPolicy"} | fl *
    ```

1. Si aucune stratégie n’est actuellement configurée, la commande ne retourne rien. Si une stratégie est retournée, ignorez cette étape et passez à l’étape suivante pour mettre à jour une stratégie existante.

    Pour ajouter la stratégie *HomeRealmDiscoveryPolicy* au locataire, utilisez la cmdlet [New-AzureADPolicy][New-AzureADPolicy] et définissez l’attribut *AlternateIdLogin* sur *"Enabled": true* comme dans l’exemple suivant :

    ```powershell
    New-AzureADPolicy -Definition @('{"HomeRealmDiscoveryPolicy" :{"AlternateIdLogin":{"Enabled": true}}}') `
        -DisplayName "BasicAutoAccelerationPolicy" `
        -IsOrganizationDefault $true `
        -Type "HomeRealmDiscoveryPolicy"
    ```

    Une fois la stratégie créée, la commande retourne l’ID de stratégie, comme illustré dans l’exemple de sortie suivant :

    ```powershell
    Id                                   DisplayName                 Type                     IsOrganizationDefault
    --                                   -----------                 ----                     ---------------------
    5de3afbe-4b7a-4b33-86b0-7bbe308db7f7 BasicAutoAccelerationPolicy HomeRealmDiscoveryPolicy True
    ```

1. Si une stratégie est déjà configurée, vérifiez si l’attribut *AlternateIdLogin* est activé, comme dans l’exemple de sortie de stratégie suivant :

    ```powershell
    Id : 5de3afbe-4b7a-4b33-86b0-7bbe308db7f7
    OdataType :
    AlternativeIdentifier :
    Definition : {{"HomeRealmDiscoveryPolicy" :{"AlternateIdLogin":{"Enabled": true}}}}
    DisplayName : BasicAutoAccelerationPolicy
    IsOrganizationDefault : True
    KeyCredentials : {}
    Type : HomeRealmDiscoveryPolicy
    ```

    Si la stratégie existe mais que l’attribut *AlternateIdLogin* n’est pas présent ou activé, ou s’il existe sur la stratégie d’autres attributs que vous souhaitez conserver, mettez à jour la stratégie existante à l’aide de la cmdlet [Set-AzureADPolicy][Set-AzureADPolicy].

    > [!IMPORTANT]
    > Lorsque vous mettez à jour la stratégie, veillez à inclure les anciens paramètres et le nouvel attribut  *AlternateIdLogin*.

    L’exemple suivant ajoute l’attribut  *AlternateIdLogin* et conserve l’attribut  *AllowCloudPasswordValidation* qui peut avoir déjà été défini :

    ```powershell
    Set-AzureADPolicy -id b581c39c-8fe3-4bb5-b53d-ea3de05abb4b `
        -Definition @('{"HomeRealmDiscoveryPolicy" :{"AllowCloudPasswordValidation":true,"AlternateIdLogin":{"Enabled": true}}}') `
        -DisplayName "BasicAutoAccelerationPolicy" `
        -IsOrganizationDefault $true `
        -Type "HomeRealmDiscoveryPolicy"
    ```

    Vérifiez que la stratégie mise à jour affiche vos modifications et que l’attribut *AlternateIdLogin* est désormais activé :

    ```powershell
    Get-AzureADPolicy | where-object {$_.Type -eq "HomeRealmDiscoveryPolicy"} | fl *
    ```

Une fois la stratégie appliquée, il peut falloir jusqu’à une heure pour qu’elle se propage et que les utilisateurs puissent se connecter avec leur ID de connexion de substitution.

## <a name="test-user-sign-in-with-email"></a>Tester la connexion de l’utilisateur avec une adresse e-mail

Pour tester le fait que les utilisateurs peuvent se connecter avec une adresse e-mail, accédez à [https://myprofile.microsoft.com][my-profile] et connectez-vous à l’aide d’un compte d’utilisateur basé sur son adresse e-mail, par exemple `balas@fabrikam.com`, et non sur son UPN, par exemple `balas@contoso.com`. L’expérience de connexion doit ressembler à celle d’une connexion basée sur un UPN.

## <a name="enable-staged-rollout-to-test-user-sign-in-with-an-email-address"></a>Activer le lancement intermédiaire pour tester la connexion de l’utilisateur avec une adresse e-mail  

Le[lancement intermédiaire][staged-rollout] permet aux administrateurs du locataire d’activer des fonctionnalités pour des groupes spécifiques. Il est recommandé aux administrateurs du locataire d’utiliser le lancement intermédiaire pour tester la connexion de l’utilisateur avec une adresse e-mail. Lorsque les administrateurs sont prêts à déployer cette fonctionnalité sur la totalité de leur locataire, ils doivent utiliser une stratégie de découverte du domaine d’accueil.  


Pour les suivre étapes suivantes, vous devez disposer d’autorisations d’*administrateur client* :

1. Ouvrez une session PowerShell en tant qu’administrateur, puis installez le module *AzureADPreview* à l’aide de la cmdlet [Install-Module][Install-Module] :

    ```powershell
    Install-Module AzureADPreview
    ```

    Si vous y êtes invité, sélectionnez **Y** pour installer NuGet ou pour installer à partir d’un dépôt non approuvé.

2. Connectez-vous à votre locataire Azure AD en tant qu’*administrateur client* en utilisant la cmdlet [Connect-AzureAD][Connect-AzureAD] :

    ```powershell
    Connect-AzureAD
    ```

    La commande retourne des informations sur votre compte, votre environnement et votre ID de locataire.

3. Répertoriez toutes les stratégies de lancement intermédiaires existantes à l’aide de la cmdlet suivante :
   
   ```powershell
   Get-AzureADMSFeatureRolloutPolicy
   ``` 

4. S’il n’existe aucune stratégie de lancement intermédiaire pour cette fonctionnalité, créez une stratégie de lancement intermédiaire et prenez note de l’ID de stratégie :

   ```powershell
   New-AzureADMSFeatureRolloutPolicy -Feature EmailAsAlternateId -DisplayName "EmailAsAlternateId Rollout Policy" -IsEnabled $true
   ```

5. Recherchez l’ID directoryObject du groupe à ajouter à la stratégie de lancement intermédiaire. Notez la valeur retournée pour le paramètre *ID*, car elle sera utilisée à l’étape suivante.
   
   ```powershell
   Get-AzureADMSGroup -SearchString "Name of group to be added to the staged rollout policy"
   ```

6. Ajoutez le groupe à la stratégie de lancement intermédiaire, comme indiqué dans l’exemple suivant. Remplacez la valeur du paramètre *-Id* par la valeur retournée pour l’ID de stratégie à l’étape 4 et remplacez la valeur du paramètre *-RefObjectId* par l’*ID* noté à l’étape 5. Il peut falloir jusqu’à 1 heure avant que les utilisateurs du groupe puissent utiliser leurs adresses proxy pour se connecter.

   ```powershell
   Add-AzureADMSFeatureRolloutPolicyDirectoryObject -Id "ROLLOUT_POLICY_ID" -RefObjectId "GROUP_OBJECT_ID"
   ```
   
Pour les nouveaux membres ajoutés au groupe, il peut falloir jusqu’à 24 heures avant de pouvoir utiliser leurs adresses proxy pour se connecter.

### <a name="removing-groups"></a>Suppression de groupes

Pour supprimer un groupe d’une stratégie de lancement intermédiaire, exécutez la commande suivante :

```powershell
Remove-AzureADMSFeatureRolloutPolicyDirectoryObject -Id "ROLLOUT_POLICY_ID" -ObjectId "GROUP_OBJECT_ID" 
```

### <a name="removing-policies"></a>Suppression de stratégies

Pour supprimer une stratégie de lancement intermédiaire, commencez par désactiver la stratégie, puis supprimez-la du système :

```powershell
Set-AzureADMSFeatureRolloutPolicy -Id "ROLLOUT_POLICY_ID" -IsEnabled $false 
Remove-AzureADMSFeatureRolloutPolicy -Id "ROLLOUT_POLICY_ID"
```

## <a name="troubleshoot"></a>Dépanner

Si des utilisateurs éprouvent des difficultés à se connecter avec leur adresse e-mail, passez en revue les étapes de résolution des problèmes suivantes :

1. Assurez-vous que l’adresse e-mail du compte d’utilisateur est définie pour l’attribut *ProxyAddresses* dans l’environnement AD DS local.
1. Vérifiez qu’Azure AD Connect est configuré et synchronise correctement les comptes d’utilisateur à partir de l’environnement de AD DS local dans Azure AD.
1. Vérifiez que l’attribut *AlternateIdLogin* de la stratégie *HomeRealmDiscoveryPolicy* d’Azure AD est défini sur *"Enabled": true* :

    ```powershell
    Get-AzureADPolicy | where-object {$_.Type -eq "HomeRealmDiscoveryPolicy"} | fl *
    ```

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur une identité hybride, par exemple, un proxy Azure AD App ou Azure AD Domain Services, consultez [Identité hybride Azure AD pour l’accès aux charges de travail locales et leur gestion][hybrid-overview].

Pour plus d’informations sur les opérations d’identité hybrides, voyez comment fonctionnent les synchronisations par [hachage de mot de passe][phs-overview] et par [authentification directe][pta-overview].

<!-- INTERNAL LINKS -->
[verify-domain]: ../fundamentals/add-custom-domain.md
[hybrid-auth-methods]: ../hybrid/choose-ad-authn.md
[azure-ad-connect]: ../hybrid/whatis-azure-ad-connect.md
[hybrid-overview]: ../hybrid/cloud-governed-management-for-on-premises.md
[phs-overview]: ../hybrid/how-to-connect-password-hash-synchronization.md
[pta-overview]: ../hybrid/how-to-connect-pta-how-it-works.md
[identity-protection]: ../identity-protection/overview-identity-protection.md#risk-detection-and-remediation

<!-- EXTERNAL LINKS -->
[Install-Module]: /powershell/module/powershellget/install-module
[Connect-AzureAD]: /powershell/module/azuread/connect-azuread
[Get-AzureADPolicy]: /powershell/module/azuread/get-azureadpolicy
[New-AzureADPolicy]: /powershell/module/azuread/new-azureadpolicy
[Set-AzureADPolicy]: /powershell/module/azuread/set-azureadpolicy
[staged-rollout]: /powershell/module/azuread/?view=azureadps-2.0-preview&preserve-view=true#staged-rollout
[my-profile]: https://myprofile.microsoft.com
