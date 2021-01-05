---
title: Authentification par certificat sur iOS - Azure Active Directory
description: En savoir plus sur les scénarios pris en charge et la configuration requise pour la configuration de l’authentification par certificat pour Azure Active Directory dans les solutions avec les appareils iOS
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 04/17/2020
ms.author: justinha
author: justinha
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: a5f9b96fe9ee0781803bbbd86316e8783b60a6f1
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861321"
---
# <a name="azure-active-directory-certificate-based-authentication-on-ios"></a>Authentification par certificat Azure Active Directory sur iOS

Afin d’améliorer la sécurité, les appareils iOS peuvent utiliser l’authentification par certificat pour s’authentifier auprès d’Azure Active Directory (Azure AD) à l’aide d’un certificat client lorsqu’ils se connectent aux applications ou services suivants :

* Des applications mobiles Office, telles que Microsoft Outlook et Microsoft Word ;
* Des clients Exchange ActiveSync (EAS).

L’utilisation de certificats élimine le besoin d’entrer un nom d’utilisateur et un mot de passe dans certaines applications de messagerie et Microsoft Office sur votre appareil mobile.

Cet article détaille les conditions requises et les scénarios pris en charge pour la configuration de l’authentification par certificat sur un appareil iOS. L’authentification par certificat pour iOS est disponible sur les clouds publics Azure, Microsoft Government Cloud, Microsoft Cloud Allemagne et Microsoft Azure Chine.

## <a name="microsoft-mobile-applications-support"></a>Prise en charge des applications mobiles Microsoft

| Applications | Support |
| --- | --- |
| Application Azure Information Protection |![La coche indique la prise en charge de cette application][1] |
| Portail d’entreprise Intune |![La coche indique la prise en charge de cette application][1] |
| Microsoft Teams |![La coche indique la prise en charge de cette application][1] |
| Office (mobile) |![La coche indique la prise en charge de cette application][1] |
| OneNote |![La coche indique la prise en charge de cette application][1] |
| OneDrive |![La coche indique la prise en charge de cette application][1] |
| Outlook |![La coche indique la prise en charge de cette application][1] |
| Power BI |![La coche indique la prise en charge de cette application][1] |
| Skype Entreprise |![La coche indique la prise en charge de cette application][1] |
| Word / Excel / PowerPoint |![La coche indique la prise en charge de cette application][1] |
| Yammer |![La coche indique la prise en charge de cette application][1] |

## <a name="requirements"></a>Spécifications

Pour utiliser l’authentification par certificat avec iOS, les conditions requises et considérations suivantes s’appliquent :

* La version du système d’exploitation de l’appareil doit être iOS 9 ou toute version ultérieure.
* Microsoft Authenticator est requis pour les applications Office sur iOS.
* Une préférence d’identité doit être créée dans les trousseaux macOS qui inclut l’URL d’authentification du serveur AD FS. Pour plus d’informations, consultez [Créer une préférence d’identité dans Trousseaux d’accès sur Mac](https://support.apple.com/guide/keychain-access/create-an-identity-preference-kyca6343b6c9/mac).

Ces conditions requises et considérations relatives aux services de fédération Active Directory (AD FS) s’appliquent :

* Le serveur AD FS doit être activé pour l’authentification par certificat et utiliser l’authentification fédérée.
* Le certificat doit avoir recours à l’utilisation améliorée de la clé et contenir l’UPN de l’utilisateur dans l’*autre nom de l’objet (nom du principal NT)* .

## <a name="configure-adfs"></a>Configurer AD FS

Pour qu’Azure AD révoque un certificat client, le jeton AD FS doit posséder les revendications suivantes. Azure AD ajoute ces revendications au jeton d’actualisation si elles sont disponibles dans le jeton AD FS (ou n’importe quel autre jeton SAML). Lorsque le jeton d’actualisation doit être validé, ces informations sont utilisées pour vérifier la révocation :

* `http://schemas.microsoft.com/ws/2008/06/identity/claims/<serialnumber>` : ajoute le numéro de série de votre certificat client
* `http://schemas.microsoft.com/2012/12/certificatecontext/field/<issuer>` : ajoute la chaîne de l’émetteur de votre certificat client

Une meilleure pratique consiste également à mettre à jour les pages d’erreurs AD FS de votre organisation à l’aide des informations suivantes :

* La configuration requise pour l’installation de Microsoft Authenticator sur iOS.
* Instructions sur l’obtention d’un certificat utilisateur.

Pour plus d’informations, consultez [Personnalisation de la page de connexion AD FS](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn280950(v=ws.11)).

## <a name="use-modern-authentication-with-office-apps"></a>Utiliser l’authentification moderne avec les applications Office

Certaines applications Office pour lesquelles l’authentification moderne est activée envoient `prompt=login` à Azure AD dans leur requête. Par défaut, Azure AD traduit `prompt=login` dans la requête envoyée à AD FS en `wauth=usernamepassworduri` (demande à AD FS d’effectuer l’authentification U/P) et `wfresh=0` (demande à AD FS d’ignorer l’état d’authentification unique et d’effectuer une nouvelle authentification). Si vous souhaitez activer l’authentification par certificat pour ces applications, modifiez le comportement par défaut d’Azure AD.

Pour mettre à jour le comportement par défaut, définissez « *PromptLoginBehavior* » dans vos paramètres de domaine fédéré sur *Désactivé*. Vous pouvez utiliser la cmdlet [MSOLDomainFederationSettings](/powershell/module/msonline/set-msoldomainfederationsettings) pour effectuer cette tâche, comme illustré dans l’exemple suivant :

```powershell
Set-MSOLDomainFederationSettings -domainname <domain> -PromptLoginBehavior Disabled
```

## <a name="support-for-exchange-activesync-clients"></a>Prise en charge pour les clients Exchange ActiveSync

Dans iOS 9 ou version ultérieure, le client de messagerie iOS natif est pris en charge. Pour savoir si cette fonctionnalité est prise en charge pour toutes les autres applications Exchange ActiveSync, contactez le développeur de votre application.

## <a name="next-steps"></a>Étapes suivantes

Pour configurer l’authentification par certificat dans votre environnement, consultez [Bien démarrer avec l’authentification par certificat](active-directory-certificate-based-authentication-get-started.md) pour obtenir des instructions.

<!--Image references-->
[1]: ./media/active-directory-certificate-based-authentication-ios/ic195031.png
