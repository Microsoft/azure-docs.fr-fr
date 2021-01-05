---
title: Considérations relatives à UWP (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: En savoir plus sur les considérations relatives à l’utilisation de la plateforme Windows universelle (UWP) avec la bibliothèque d’authentification Microsoft pour .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/16/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: c71c55adbf79af780169380269b5081322f94baf
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93083437"
---
# <a name="considerations-for-using-universal-windows-platform-with-msalnet"></a>Considérations relatives à l’utilisation de la plateforme Windows universelle avec MSAL.NET
Les développeurs d’applications qui utilisent la plateforme Windows universelle (UWP) avec MSAL.NET doivent prendre en compte les concepts présentés dans cet article.

## <a name="the-usecorporatenetwork-property"></a>La propriété UseCorporateNetwork
Sur la plateforme Windows Runtime (WinRT), `PublicClientApplication` a la propriété booléenne `UseCorporateNetwork`. Cette propriété permet aux applications Windows 8.1 et aux applications UWP de tirer parti de l’Authentification Windows intégrée (IWA) si l’utilisateur est connecté à un compte disposant d’un locataire Azure Active Directory (Azure AD) fédéré. Les utilisateurs qui sont connectés au système d’exploitation peuvent également utiliser l’authentification unique (SSO). Lorsque vous définissez la propriété `UseCorporateNetwork`, MSAL.NET utilise un répartiteur d’authentification web (WAB).

> [!IMPORTANT]
> La définition de la propriété `UseCorporateNetwork` sur true suppose que le développeur d’applications a activé l’IWA dans l’application. Pour activer l’IWA :
> - Dans `Package.appxmanifest` de votre application UWP, cliquez sur l’onglet **Fonctionnalités** et activez les fonctionnalités suivantes :
>   - **Authentification en entreprise**
>   - **Réseaux privés (client et serveur)**
>   - **Certificat utilisateur partagé**

L’IWA n’est pas activé par défaut, car Microsoft Store nécessite un haut niveau de vérification avant d’accepter les applications qui demandent les fonctionnalités d’authentification d’entreprise ou de certificats utilisateur partagés. Tous les développeurs ne souhaitent pas passer ce niveau de vérification.

Sur la plateforme UWP, l’implémentation WAB sous-jacente ne fonctionne pas correctement dans les scénarios d’entreprise où l’accès conditionnel est activé. Les utilisateurs voient les symptômes de ce problème lorsqu’ils essaient de se connecter à l’aide de Windows Hello. Lorsque l’utilisateur est invité à choisir un certificat :

- Le certificat correspondant au code confidentiel est introuvable.
- Une fois que l’utilisateur a choisi un certificat, il n’est pas invité à entrer le code confidentiel.

Vous pouvez essayer d’éviter ce problème à l’aide d’une autre méthode, telle que l’authentification par nom d’utilisateur-mot de passe et par téléphone, mais l’expérience n’est pas bonne.

## <a name="troubleshooting"></a>Dépannage

Certains clients ont signalé l’erreur de connexion suivante : dans des environnements d’entreprise spécifiques dans lesquels ils savent qu’ils disposent d’une connexion Internet, la connexion fonctionne avec un réseau public.

```Text
We can't connect to the service you need right now. Check your network connection or try this again later.
```

Vous pouvez éviter ce problème en vous assurant que WAB (le composant Windows sous-jacent) autorise un réseau privé. Pour ce faire, vous pouvez définir une clé de Registre :

```Text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\authhost.exe\EnablePrivateNetwork = 00000001
```

Pour plus d’informations, consultez [Répartiteur d’authentification web – Fiddler](/windows/uwp/security/web-authentication-broker#fiddler).

## <a name="next-steps"></a>Étapes suivantes
Les exemples suivants fournissent des informations complémentaires.

Exemple | Plateforme | Description 
|------ | -------- | -----------|
|[active-directory-dotnet-native-uwp-v2](https://github.com/azure-samples/active-directory-dotnet-native-uwp-v2) | UWP | Application cliente UWP qui utilise MSAL.NET. Elle accède à Microsoft Graph pour un utilisateur qui s’authentifie à l’aide d’un point de terminaison Azure AD 2.0. <br>![Topologie](media/msal-net-uwp-considerations/topology-native-uwp.png)|
|[active-directory-xamarin-native-v2](https://github.com/Azure-Samples/active-directory-xamarin-native-v2) | Xamarin iOS, Android, UWP | Application Xamarin Forms qui montre comment utiliser MSAL pour authentifier des comptes personnels Microsoft et Azure AD par le biais de la plateforme d’identités Microsoft. Elle montre également comment accéder à Microsoft Graph et affiche le jeton résultant. <br>![Diagramme qui montre comment utiliser MSAL pour authentifier des comptes personnels Microsoft et Azure AD par le biais de la plateforme d’identités Microsoft.](media/msal-net-uwp-considerations/topology-xamarin-native.png)|
