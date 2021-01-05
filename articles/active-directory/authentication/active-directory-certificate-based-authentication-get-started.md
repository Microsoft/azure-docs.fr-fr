---
title: Authentification par certificat - Azure Active Directory
description: Découvrez comment configurer l’authentification par certificat dans votre environnement
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: annaba
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: 3ba84bb3ee38981217e72f8372a836b03647083d
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861338"
---
# <a name="get-started-with-certificate-based-authentication-in-azure-active-directory"></a>Bien démarrer avec l’authentification par certificat dans Azure Active Directory

L’authentification basée sur les certificats vous permet d’être authentifié par Azure Active Directory avec un certificat client sur un appareil Windows, Android ou iOS, lors de la connexion de votre compte Exchange Online à :

- Des applications mobiles Microsoft, telles que Microsoft Outlook et Microsoft Word
- Des clients Exchange ActiveSync (EAS).

La configuration de cette fonctionnalité élimine le besoin d’entrer un nom d’utilisateur et un mot de passe dans certaines applications de messagerie et Microsoft Office sur votre appareil mobile.

Cette rubrique :

- Vous indique la procédure pour configurer et utiliser l’authentification par certificat pour les utilisateurs de clients dans les plans Office 365 Enterprise, Business et Education et US Government. Cette fonctionnalité est disponible en version préliminaire dans les plans Office 365 China, US Government Defense et US Government Federal.
- Suppose que vous avez déjà une [infrastructure de clé publique (PKI)](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831740(v=ws.11)) et [AD FS](../hybrid/how-to-connect-fed-whatis.md) configurés.

## <a name="requirements"></a>Spécifications

Pour configurer l’authentification basée sur les certificats, les conditions suivantes doivent être remplies :

- L’authentification basée sur les certificats est uniquement prise en charge dans les environnements fédérés pour les applications de navigateur, les clients natifs qui utilisent l’authentification moderne (ADAL) ou les bibliothèques MSAL. La seule exception est Exchange Active Sync (EAS) pour Exchange Online (EXO), qui peut être utilisé à la fois pour les comptes fédérés et les comptes managés.
- L’autorité de certification racine et les autorités de certification intermédiaires doivent être configurées dans Azure Active Directory.
- Chaque autorité de certification doit avoir une liste de révocation de certificat (CRL) qui peut être référencée via une URL accessible sur Internet.
- Vous devez disposer d’au moins une autorité de certification configurée dans Azure Active Directory. Vous trouverez les étapes associées dans la section [Configurer les autorités de certification](#step-2-configure-the-certificate-authorities).
- Pour les clients Exchange ActiveSync, le certificat client doit avoir l’adresse de messagerie routable de l’utilisateur dans Exchange Online, dans la valeur Nom du principal ou la valeur Nom RFC822 du champ Autre nom de l’objet. Azure Active Directory mappe la valeur RFC822 à l’attribut Adresse proxy dans le répertoire.
- Votre appareil client doit avoir accès à au moins une autorité de certification qui émet des certificats clients.
- Un certificat client pour l’authentification du client doit avoir été émis pour votre client.

>[!IMPORTANT]
>La taille maximale d’une liste de révocation de certificats à télécharger et à mettre en cache par Azure Active Directory est de 20 Mo, et le temps nécessaire au téléchargement de cette liste ne doit pas dépasser 10 secondes.  Si Azure Active Directory ne parvient pas à télécharger une liste de révocation de certificats, les authentifications basées sur les certificats émis par l’autorité de certification correspondante échouent. Les bonnes pratiques permettant de s’assurer que les fichiers CRL se trouvent dans des limites de taille acceptables consistent à conserver les durées de vie des certificats dans des limites raisonnables, et à nettoyer les certificats expirés.

## <a name="step-1-select-your-device-platform"></a>Étape 1 : Sélectionner la plateforme de votre appareil

Dans un premier temps, pour la plateforme d’appareil qui vous intéresse, vous devez passer en revue les éléments suivants :

- La prise en charge des applications mobiles Office
- Les conditions requises spécifiques pour la mise en œuvre

Les informations connexes existent pour les plateformes d’appareils suivantes :

- [Android](active-directory-certificate-based-authentication-android.md)
- [iOS](active-directory-certificate-based-authentication-ios.md)

## <a name="step-2-configure-the-certificate-authorities"></a>Étape 2 : Configurer les autorités de certification

Pour configurer vos autorités de certification dans Azure Active Directory, pour chaque autorité de certification, vous devez télécharger les éléments suivants :

* La partie publique du certificat, au format *.cer*
* Les URL accessibles sur Internet où résident les listes de révocation de certificat (CRL).

Le schéma d’une autorité de certification se présente comme suit :

```csharp
    class TrustedCAsForPasswordlessAuth
    {
       CertificateAuthorityInformation[] certificateAuthorities;
    }

    class CertificateAuthorityInformation

    {
        CertAuthorityType authorityType;
        X509Certificate trustedCertificate;
        string crlDistributionPoint;
        string deltaCrlDistributionPoint;
        string trustedIssuer;
        string trustedIssuerSKI;
    }

    enum CertAuthorityType
    {
        RootAuthority = 0,
        IntermediateAuthority = 1
    }
```

Pour la configuration, vous pouvez utiliser [Azure Active Directory PowerShell Version 2](/powershell/azure/active-directory/install-adv2) :

1. Démarrez Windows PowerShell avec les privilèges administrateur.
2. Installez le module Azure AD, version [2.0.0.33](https://www.powershellgallery.com/packages/AzureAD/2.0.0.33) ou ultérieure.

```powershell
    Install-Module -Name AzureAD –RequiredVersion 2.0.0.33
```

Comme première étape de configuration, vous devez établir une connexion avec votre client. Dès qu’une connexion est établie avec votre locataire, vous pouvez examiner, ajouter, supprimer et modifier les autorités de certification approuvées qui sont définies dans votre annuaire.

### <a name="connect"></a>Se connecter

Pour établir une connexion avec votre client, utilisez l’applet de commande [Connect-AzureAD](/powershell/module/azuread/connect-azuread) :

```azurepowershell
    Connect-AzureAD
```

### <a name="retrieve"></a>Récupération

Pour récupérer les autorités de certification approuvées qui sont définies dans votre répertoire, utilisez l’applet de commande [Get-AzureADTrustedCertificateAuthority](/powershell/module/azuread/get-azureadtrustedcertificateauthority).

```azurepowershell
    Get-AzureADTrustedCertificateAuthority
```

### <a name="add"></a>Ajouter

Pour créer une autorité de certification approuvée, utilisez l’applet de commande [New-AzureADTrustedCertificateAuthority](/powershell/module/azuread/new-azureadtrustedcertificateauthority) et définissez l’attribut **crlDistributionPoint** à une valeur correcte :

```azurepowershell
    $cert=Get-Content -Encoding byte "[LOCATION OF THE CER FILE]"
    $new_ca=New-Object -TypeName Microsoft.Open.AzureAD.Model.CertificateAuthorityInformation
    $new_ca.AuthorityType=0
    $new_ca.TrustedCertificate=$cert
    $new_ca.crlDistributionPoint="<CRL Distribution URL>"
    New-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $new_ca
```

### <a name="remove"></a>Supprimer

Pour supprimer une autorité de certification approuvée, utilisez l’applet de commande [Remove-AzureADTrustedCertificateAuthority](/powershell/module/azuread/remove-azureadtrustedcertificateauthority) :

```azurepowershell
    $c=Get-AzureADTrustedCertificateAuthority
    Remove-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $c[2]
```

### <a name="modify"></a>Modifier

Pour modifier une autorité de certification approuvée, utilisez l’applet de commande [Set-AzureADTrustedCertificateAuthority](/powershell/module/azuread/set-azureadtrustedcertificateauthority) :

```azurepowershell
    $c=Get-AzureADTrustedCertificateAuthority
    $c[0].AuthorityType=1
    Set-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $c[0]
```

## <a name="step-3-configure-revocation"></a>Étape 3 : Configurer la révocation

Pour révoquer un certificat client, Azure Active Directory extrait la liste de révocation de certificat (CRL) à partir des URL téléchargées dans le cadre des informations sur l’autorité de certification et la met en cache. L’horodateur de la dernière publication (propriété **Effective Date** ) dans la liste de révocation de certificat permet de vérifier si la CRL est toujours valide. La CRL est référencée périodiquement pour révoquer l’accès à des certificats qui font partie de la liste.

Si une révocation plus instantanée est requise (par exemple, si un utilisateur perd un appareil), le jeton d’autorisation de l’utilisateur peut être invalidé. Pour invalider le jeton d’autorisation, définissez le champ **StsRefreshTokenValidFrom** pour cet utilisateur à l’aide de Windows PowerShell. Vous devez mettre à jour le champ **StsRefreshTokenValidFrom** pour chaque utilisateur pour lequel vous souhaitez révoquer l’accès.

Pour garantir que la révocation persiste, vous devez définir la propriété **Effective Date** de la CRL sur une date postérieure à la valeur définie par **StsRefreshTokenValidFrom** et vérifiez que le certificat en question est inclus dans la CRL.

Les étapes suivantes décrivent le processus de mise à jour et d’invalidation du jeton d’autorisation avec la définition du champ **StsRefreshTokenValidFrom** .

**Pour configurer la révocation :**

1. Connectez-vous au service MSOL avec les informations d’identification administrateur :

```powershell
        $msolcred = get-credential
        connect-msolservice -credential $msolcred
```

2. Récupérez la valeur StsRefreshTokensValidFrom actuelle pour un utilisateur :

```powershell
        $user = Get-MsolUser -UserPrincipalName test@yourdomain.com`
        $user.StsRefreshTokensValidFrom
```

3. Configurez une nouvelle valeur StsRefreshTokensValidFrom pour l’utilisateur. Elle doit être égale à l’horodateur actuel :

```powershell
        Set-MsolUser -UserPrincipalName test@yourdomain.com -StsRefreshTokensValidFrom ("03/05/2016")
```

La date que vous définissez doit être dans le futur. Si la date n’est pas dans le futur, la propriété **StsRefreshTokensValidFrom** n’est pas définie. Si la date est dans le futur, la propriété **StsRefreshTokensValidFrom** est définie sur l’heure actuelle (et non la date indiquée par la commande Set-MsolUser).

## <a name="step-4-test-your-configuration"></a>Étape 4 : Tester votre configuration

### <a name="testing-your-certificate"></a>Test de votre certificat

Comme premier test de configuration, vous devez essayer de vous connecter à [Outlook Web Access](https://outlook.office365.com) ou [SharePoint Online](https://microsoft.sharepoint.com) à l’aide du **navigateur sur l’appareil**.

Si votre connexion est réussie, vous savez que :

- Le certificat utilisateur a été configuré sur votre appareil de test
- AD FS est correctement configuré

### <a name="testing-office-mobile-applications"></a>Test des applications Office mobiles

**Pour tester l’authentification par certificat sur votre application Office mobile :**

1. Sur votre appareil de test, installez une application mobile Office (par exemple, OneDrive).
3. Lancez l’application.
4. Entrez votre nom d’utilisateur et sélectionnez le certificat utilisateur que vous souhaitez utiliser.

Vous devez être connecté.

### <a name="testing-exchange-activesync-client-applications"></a>Test des applications clientes Exchange ActiveSync

Pour accéder à Exchange ActiveSync (EAS) via l’authentification par certificat, un profil EAS contenant le certificat client doit être disponible pour l’application.

Le profil EAS doit contenir les informations suivantes :

- Le certificat utilisateur à utiliser pour l’authentification

- Le point de terminaison EAS (par exemple, outlook.office365.com)

Un profil EAS peut être configuré et placé sur l’appareil via l’utilisation de la gestion des appareils mobiles (MDM), comme Intune, ou en plaçant manuellement le certificat dans le profil EAS sur l’appareil.

### <a name="testing-eas-client-applications-on-android"></a>Test des applications clientes EAS sur Android

**Pour tester l’authentification par certificat :**

1. Configurez un profil EAS dans l’application, qui respecte les spécifications citées dans la section précédente.
2. Ouvrez l’application et vérifiez la synchronisation de la messagerie.

## <a name="next-steps"></a>Étapes suivantes

[Obtenir des informations supplémentaires sur l’authentification basée sur les certificats pour les appareils Android.](active-directory-certificate-based-authentication-android.md)

[Obtenir des informations supplémentaires sur l’authentification basée sur les certificats pour les appareils iOS.](active-directory-certificate-based-authentication-ios.md)
