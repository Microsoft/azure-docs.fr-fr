---
title: Stratégies de réinitialisation de mot de passe libre-service - Azure Active Directory
description: En savoir plus sur les différentes options de stratégie de réinitialisation de mot de passe en libre-service Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 10/05/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.custom: contperf-fy20q4
ms.openlocfilehash: 980e830716b66b63c8f2dc6e641a88af26d44871
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97027062"
---
# <a name="password-policies-and-account-restrictions-in-azure-active-directory"></a>Stratégies de mot de passe et restrictions de compte dans Azure Active Directory

Dans Azure Active Directory (Azure AD), vous trouverez une stratégie de mot de passe qui définit des paramètres tels que la complexité, la longueur ou l’âge du mot de passe. Vous trouverez également une stratégie qui définit la longueur et les caractères acceptables pour les noms d’utilisateur.

Si la réinitialisation de mot de passe en libre-service (SSPR) est utilisée pour changer ou réinitialiser un mot de passe dans Azure AD, la stratégie de mot de passe est vérifiée. Si le mot de passe ne répond pas aux exigences de la stratégie, l’utilisateur est invité à réessayer. Les administrateurs Azure ont des restrictions sur l’utilisation de SSPR qui diffèrent de celles des comptes d’utilisateur standard.

Cet article décrit les paramètres de stratégie de mot de passe et les exigences de complexité associés aux comptes d’utilisateur dans votre locataire Azure AD, ainsi que la façon dont vous pouvez utiliser PowerShell pour vérifier ou définir des paramètres d’expiration du mot de passe.

## <a name="username-policies"></a><a name="userprincipalname-policies-that-apply-to-all-user-accounts"></a>Stratégies de nom d’utilisateur

Chaque compte se connectant à Azure AD doit être doté d’une valeur d’attribut unique de nom d’utilisateur principal (UPN) associée à son compte. Dans les environnements hybrides où un environnement Active Directory Domain Services (AD DS) local est synchronisé avec Azure AD à l’aide d’Azure Connect, l’UPN Azure AD est défini par défaut avec l’UPN local.

Le tableau suivant décrit les stratégies de nom d’utilisateur qui s’appliquent à la fois aux comptes AD DS locaux synchronisés avec Azure AD et aux comptes d’utilisateur uniquement dans le cloud créés directement dans Azure AD :

| Propriété | Conditions requises pour UserPrincipalName |
| --- | --- |
| Caractères autorisés |<ul> <li>A – Z</li> <li>a - z</li><li>0 – 9</li> <li> ' \. - \_ ! \# ^ \~</li></ul> |
| Caractères non autorisés |<ul> <li>Tout caractère « \@ \"» qui ne sépare pas le nom d’utilisateur du domaine.</li> <li>Ne peut pas contenir un point « . » précédant immédiatement le symbole « \@\" ».</li></ul> |
| Contraintes de longueur |<ul> <li>La longueur totale ne doit pas dépasser 113 caractères</li><li>Jusqu’à 64 caractères avant le symbole « \@\" ».</li><li>Jusqu’à 48 caractères après le symbole « \@\" ».</li></ul> |

## <a name="azure-ad-password-policies"></a><a name="password-policies-that-only-apply-to-cloud-user-accounts"></a>Stratégies de mot de passe Azure AD

Une stratégie de mot de passe est appliquée à tous les comptes d’utilisateur qui sont créés et gérés directement dans Azure AD. Certains de ces paramètres de stratégie de mot de passe ne peuvent pas être modifiés, bien que vous puissiez [configurer des mots de passe interdits personnalisés pour la protection par mot de passe Azure AD](tutorial-configure-custom-password-protection.md) ou des paramètres de verrouillage de compte.

Par défaut, un compte est bloqué au bout de 10 tentatives de connexion infructueuses avec un mot de passe incorrect. L’utilisateur est verrouillé pendant une minute. La durée de blocage de l’utilisateur augmente au fil des nouvelles tentatives de connexion incorrectes. Le [verrouillage intelligent](howto-password-smart-lockout.md) suit les trois derniers hachages de mots de passe incorrects afin d'éviter d'incrémenter le compteur de verrouillages pour le même mot de passe. Si un utilisateur entre plusieurs fois le même mot de passe incorrect, le compte n'est pas verrouillé. Vous pouvez définir le seuil de verrouillage intelligent et la durée.

La stratégie de mot de passe Azure AD ne s’applique pas aux comptes d’utilisateur synchronisés à partir d’un environnement AD DS local à l’aide d’Azure AD Connect, sauf si vous activez *EnforceCloudPasswordPolicyForPasswordSyncedUsers*.

Les options de stratégie de mot de passe Azure AD suivantes sont définies. Sauf indication contraire, vous ne pouvez pas modifier ces paramètres :

| Propriété | Spécifications |
| --- | --- |
| Caractères autorisés |<ul><li>A – Z</li><li>a - z</li><li>0 – 9</li> <li>@ # $ % ^ & * - _ ! + = [ ] { } &#124; \ : ' , . ? / \` ~ " ( ) ;</li> <li>espace vide</li></ul> |
| Caractères non autorisés | Caractères Unicode. |
| Restrictions de mot de passe |<ul><li>8 caractères minimum et 256 caractères maximum.</li><li>trois des quatre éléments suivants sont requis :<ul><li>Caractères minuscules.</li><li>Caractères majuscules.</li><li>Nombres (0-9).</li><li>Symboles (voir les restrictions de mot de passe précédentes).</li></ul></li></ul> |
| Durée d’expiration du mot de passe (âge maximum du mot de passe) |<ul><li>Valeur par défaut : **90** jours.</li><li>La valeur peut être configurée à l’aide de l’applet de commande `Set-MsolPasswordPolicy` à partir du module Azure Active Directory pour Windows PowerShell.</li></ul> |
| Notification d’expiration du mot de passe (lorsque les utilisateurs sont avertis de l’expiration du mot de passe) |<ul><li>Valeur par défaut : **14** jours (avant l’expiration du mot de passe).</li><li>La valeur peut être configurée à l’aide de l’applet de commande `Set-MsolPasswordPolicy`.</li></ul> |
| Expiration du mot de passe (empêche le mot de passe d’expirer) |<ul><li>Valeur par défaut : **false** (indique que le mot de passe a une date d’expiration).</li><li>La valeur peut être configurée pour des comptes d’utilisateur individuels à l’aide de l’applet de commande `Set-MsolUser`.</li></ul> |
| Historique de modification du mot de passe | Le dernier mot de passe *ne peut pas* être réutilisé lorsque l’utilisateur modifie un mot de passe. |
| Historique de réinitialisation du mot de passe | Le dernier mot de passe *peut* être réutilisé lorsque l’utilisateur réinitialise un mot de passe oublié. |

## <a name="administrator-reset-policy-differences"></a>Différences en matière de stratégie de réinitialisation par l’administrateur

Par défaut, les comptes d’administrateur sont activés pour la réinitialisation de mot de passe en libre-service, et une stratégie de réinitialisation de mot de passe *à deux portes* renforcée par défaut est appliquée. Cette stratégie peut être différente de celle que vous avez définie pour vos utilisateurs et ne peut pas être modifiée. Vous devez toujours tester la fonctionnalité de réinitialisation de mot de passe en tant qu’utilisateur, sans qu’un rôle d’administrateur Azure vous soit affecté.

Avec une stratégie à deux verrous, les administrateurs n’ont pas la possibilité d’utiliser des questions de sécurité.

La stratégie à deux verrous nécessite deux éléments de données d’authentification, par exemple une adresse e-mail, une application d’authentification ou un numéro de téléphone. Une stratégie à deux verrous s’applique dans les conditions suivantes :

* Tous les rôles d’administrateur suivants sont concernés :
  * Administrateur du support technique
  * Administrateur de support de service
  * Administrateur de facturation
  * Prise en charge de niveau 1 de partenaire
  * Prise en charge de niveau 2 de partenaire
  * Administrateur Exchange
  * Administrateur Skype Entreprise
  * Administrateur d’utilisateurs
  * Enregistreurs de répertoire
  * Administrateur général ou administrateur d’entreprise
  * Administrateur SharePoint
  * Administrateur de conformité
  * Administrateur d’application
  * Administrateur de sécurité
  * Administrateur de rôle privilégié
  * Administrateur Intune
  * Administrateur de services de proxy d’application
  * Administrateur Dynamics 365
  * Administrateur de services Power BI
  * Administrateur d’authentification
  * Administrateur d’authentification privilégié

* Si 30 jours se sont écoulés dans un abonnement d’essai ; ou
* Un domaine personnalisé a été configuré pour votre locataire Azure AD, par exemple *contoso.com* ; ou
* Azure AD Connect synchronise les identités à partir de votre répertoire local

Vous pouvez désactiver l’utilisation de SSPR pour les comptes d’administrateur à l’aide de l’applet de commande PowerShell [Set-MsolCompanySettings](/powershell/module/msonline/set-msolcompanysettings). Le paramètre `-SelfServePasswordResetEnabled $False` désactive SSPR pour les administrateurs.

### <a name="exceptions"></a>Exceptions

Une stratégie à un verrou nécessite un élément de données d’authentification, par exemple une adresse de messagerie ou un numéro de téléphone. Une stratégie à un verrou s’applique dans les conditions suivantes :

* Pendant les 30 premiers jours d’un abonnement d’essai ; ou
* Un domaine personnalisé n’a pas été configuré pour votre locataire Azure AD. Celui-ci utilise donc le domaine par défaut * *.onmicrosoft.com*. L’utilisation du domaine par défaut * *.onmicrosoft.com* n’est pas recommandée dans un environnement de production ; et
* Azure AD Connect ne synchronise pas les identités

## <a name="password-expiration-policies"></a><a name="set-password-expiration-policies-in-azure-ad"></a>Stratégies d’expiration de mot de passe

Un *administrateur général* ou un *administrateur d’utilisateurs* peut utiliser le [module Microsoft Azure AD pour Windows PowerShell](/powershell/module/Azuread/) afin de définir des mots de passe utilisateur qui n’expirent pas.

Vous pouvez également utiliser des applets de commande PowerShell pour supprimer la configuration de non-expiration ou pour voir quels mots de passe utilisateur sont définis pour ne jamais expirer.

Ces conseils s’appliquent à d’autres fournisseurs, tels que Intune et Microsoft 365, qui s’appuient également sur Azure AD pour les services d’identité et d’annuaire. L’expiration du mot de passe est la seule partie de la stratégie qui peut être modifiée.

> [!NOTE]
> Seuls les mots de passe de comptes d’utilisateurs non synchronisés par le biais d’Azure AD Connect peuvent être configurés pour ne pas expirer. Pour plus d’informations sur la synchronisation d’annuaires, consultez [Connecter AD à Azure AD](../hybrid/whatis-hybrid-identity.md).

### <a name="set-or-check-the-password-policies-by-using-powershell"></a>Définir ou vérifier les stratégies de mot de passe à l’aide de PowerShell

Pour commencer, [téléchargez et installez le module Azure AD PowerShell](/powershell/module/Azuread/), puis [connectez-le à votre locataire Azure AD](/powershell/module/azuread/connect-azuread#examples).

Une fois le module installé, suivez les étapes ci-après pour effectuer chaque tâche en fonction des besoins.

### <a name="check-the-expiration-policy-for-a-password"></a>Vérifier la stratégie d’expiration d’un mot de passe

1. Ouvrez une invite PowerShell et [connectez-vous à votre locataire Azure AD](/powershell/module/azuread/connect-azuread#examples) à l’aide d’un compte d’*administrateur général* ou d’*administrateur d’utilisateurs*.
1. Exécutez l’une des commandes suivantes pour un utilisateur individuel ou tous les utilisateurs :

   * Pour voir si le mot de passe d’un seul utilisateur est défini pour ne jamais expirer, exécutez l’applet de commande suivante. Remplacez `<user ID>` par l’identifiant de l’utilisateur que vous souhaitez vérifier, par exemple *driley\@contoso.onmicrosoft.com* :

       ```powershell
       Get-AzureADUser -ObjectId <user ID> | Select-Object @{N="PasswordNeverExpires";E={$_.PasswordPolicies -contains "DisablePasswordExpiration"}}
       ```

   * Pour afficher le paramètre **Le mot de passe n’expire jamais** pour tous les utilisateurs, exécutez la cmdlet suivante :

       ```powershell
       Get-AzureADUser -All $true | Select-Object UserPrincipalName, @{N="PasswordNeverExpires";E={$_.PasswordPolicies -contains "DisablePasswordExpiration"}}
       ```

### <a name="set-a-password-to-expire"></a>Définir un mot de passe pour qu’il expire

1. Ouvrez une invite PowerShell et [connectez-vous à votre locataire Azure AD](/powershell/module/azuread/connect-azuread#examples) à l’aide d’un compte d’*administrateur général* ou d’*administrateur d’utilisateurs*.
1. Exécutez l’une des commandes suivantes pour un utilisateur individuel ou tous les utilisateurs :

   * Pour définir le mot de passe d’un utilisateur afin qu’il expire, exécutez l’applet de commande suivante. Remplacez `<user ID>` par l’identifiant de l’utilisateur que vous souhaitez vérifier, par exemple *driley\@contoso.onmicrosoft.com* :

       ```powershell
       Set-AzureADUser -ObjectId <user ID> -PasswordPolicies None
       ```

   * Pour définir les mots de passe de tous les utilisateurs de l’organisation afin qu’ils expirent, utilisez la cmdlet suivante :

       ```powershell
       Get-AzureADUser -All $true | Set-AzureADUser -PasswordPolicies None
       ```

### <a name="set-a-password-to-never-expire"></a>Définir un mot de passe pour qu’il n’expire jamais

1. Ouvrez une invite PowerShell et [connectez-vous à votre locataire Azure AD](/powershell/module/azuread/connect-azuread#examples) à l’aide d’un compte d’*administrateur général* ou d’*administrateur d’utilisateurs*.
1. Exécutez l’une des commandes suivantes pour un utilisateur individuel ou tous les utilisateurs :

   * Pour définir le mot de passe d’un utilisateur afin qu’il n’expire jamais, exécutez l’applet de commande suivante : Remplacez `<user ID>` par l’identifiant de l’utilisateur que vous souhaitez vérifier, par exemple *driley\@contoso.onmicrosoft.com* :

       ```powershell
       Set-AzureADUser -ObjectId <user ID> -PasswordPolicies DisablePasswordExpiration
       ```

   * Pour définir les mots de passe de tous les utilisateurs de l’organisation afin qu’ils n’expirent jamais, utilisez la cmdlet suivante :

       ```powershell
       Get-AzureADUser -All $true | Set-AzureADUser -PasswordPolicies DisablePasswordExpiration
       ```

   > [!WARNING]
   > Les mots de passe définis sur `-PasswordPolicies DisablePasswordExpiration` le restent en fonction de l’attribut `pwdLastSet`. En fonction de l’attribut `pwdLastSet`, si vous définissez l’expiration sur `-PasswordPolicies None`, tous les mots dont `pwdLastSet` est supérieur à 90 jours doivent être modifiés par l’utilisateur lors de sa connexion suivante. Cette modification peut affecter un grand nombre d’utilisateurs.

## <a name="next-steps"></a>Étapes suivantes

Pour bien démarrer avec SSPR, consultez [Tutoriel : Permettre aux utilisateurs de déverrouiller leur compte ou de réinitialiser des mots de passe à l’aide de la réinitialisation de mot de passe en libre-service Azure Active Directory](tutorial-enable-sspr.md).

Si vos utilisateurs ou vous-même rencontrez des problèmes liés à SSPR, consultez [Résoudre les problèmes de réinitialisation de mot de passe en libre-service](./troubleshoot-sspr.md).
